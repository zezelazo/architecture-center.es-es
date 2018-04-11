---
title: Comunicación entre servicios en los microservicios
description: Comunicación entre servicios en los microservicios
author: MikeWasson
ms.date: 12/08/2017
ms.openlocfilehash: aff2fb7b2be25ca32d6224cee15363880cfb1488
ms.sourcegitcommit: a8453c4bc7c870fa1a12bb3c02e3b310db87530c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/29/2017
---
# <a name="designing-microservices-interservice-communication"></a>Diseño de microservicios: comunicación entre servicios

La comunicación entre microservicios tiene que ser eficiente y sólida. Con una gran cantidad de pequeños servicios interactuando para realizar una sola transacción, esto puede suponer un desafío. En este capítulo, veremos los inconvenientes entre la mensajería asincrónica en comparación con las API sincrónicas. A continuación, veremos algunos de los desafíos a la hora de diseñar una comunicación resistente entre servicios, y el rol que puede desempeñar una malla de servicio.

![](./images/interservice-communication.png)

## <a name="challenges"></a>Desafíos 

Estos son algunos de los principales desafíos que se derivan de la comunicación de servicio a servicio. Las mallas de servicio, que se describe más adelante en este capítulo, están diseñadas para controlar muchos de estos desafíos.

**Resistencia.** Puede haber decenas o incluso centenares de instancias de cualquier microservicio concreto. Se puede producir un error en una instancia por una serie de motivos. Puede haber un error de nivel de nodo, como un error de hardware o un reinicio de máquina virtual. Una instancia puede bloquearse o verse abrumada por las solicitudes, y dejar de procesar todas las nuevas solicitudes. Cualquiera de estos eventos puede provocar que una llamada de red genere un error. Hay dos patrones de diseño que pueden ayudar a que las llamadas de red de servicio a servicio ganen en resistencia:

- **[Retry](../patterns/retry.md)**. Una llamada de red puede producir un error debido a un error transitorio que desaparece por sí solo. En lugar de declarar un error directamente, el autor de la llamada debería reintentar la operación un determinado número de veces o hasta que transcurra un período de tiempo de espera configurado. Sin embargo, si una operación no es idempotente, los reintentos pueden producir efectos secundarios no deseados. La llamada original puede realizarse correctamente, pero el autor de la llamada nunca recibe una respuesta. Si el autor de la llamada vuelve a intentar realizarla, se puede invocar la operación dos veces. Por lo general, no es seguro volver a intentar los métodos POST o PATCH, porque no se garantiza que sean idempotentes.

- **[Circuit Breaker](../patterns/circuit-breaker.md)**. Demasiadas solicitudes con error pueden causar un cuello de botella, ya que las solicitudes pendientes se acumulan en la cola. Estas solicitudes bloqueadas pueden contener recursos críticos del sistema, tales como la memoria, subprocesos o conexiones de base de datos, entre otros, que pueden causar errores en cascada. El patrón Circuit Breaker puede evitar que un servicio intente repetidamente una operación que probablemente produzca errores. 

**Equilibrio de carga**. Cuando el servicio "A" llama servicio "B", la solicitud tiene que llegar a una instancia en ejecución del servicio "B". En Kubernetes, el tipo de recurso `Service` proporciona una dirección IP estable para un grupo de pods. El tráfico de red a la dirección IP del servicio se reenvía a un pod mediante reglas de iptable. De forma predeterminada, se elige un pod aleatorio. Una malla de servicio (ver abajo) puede proporcionar algoritmos de equilibrio de carga más inteligentes en función de la latencia observada o de otras métricas.

**Seguimiento distribuido**. Una sola transacción puede abarcar varios servicios. Esto puede dificultar la supervisión del rendimiento general y del estado del sistema. Incluso si todos los servicios generan registros y métricas, si no hay una manera de unirlos, son de uso limitado. En el capítulo [Registro y supervisión](./logging-monitoring.md) se habla en más detalle acerca del seguimiento distribuido, pero se menciona aquí como un desafío.

**Versiones del servicio**. Cuando un equipo implementa una nueva versión de un servicio, tiene que evitar romper otros servicios o clientes externos que dependen de él. Además, puede ejecutar varias versiones de un servicio en paralelo y enrutar las solicitudes a una versión determinada. Consulte [Control de versiones de la API](./api-design.md#api-versioning) para más información sobre este punto.

**Cifrado TLS y autenticación de TLS mutua**. Por motivos de seguridad, puede querer cifrar el tráfico entre los servicios con TLS y utilizar la autenticación de TLS mutua para autenticar a los autores de las llamadas.

## <a name="synchronous-versus-asynchronous-messaging"></a>Mensajería sincrónica frente a la asincrónica

Hay dos patrones de mensajería básicos que los microservicios pueden usar para comunicarse con otros microservicios. 

1. Comunicación sincrónica. En este patrón, un servicio llama a una API que otro servicio expone mediante un protocolo como HTTP o gRPC. Esta opción es un patrón de mensajería sincrónico porque el autor de la llamada espera a la respuesta del receptor. 

2. Paso de mensajes asincrónicos. En este patrón, un servicio envía el mensaje sin esperar por la respuesta, y uno o más servicios procesan el mensaje de forma asincrónica.

Es importante distinguir entre operaciones de E/S asincrónicas y un protocolo asincrónico. Una operación de E/S asincrónica significa que el subproceso de llamada no se bloquea mientras se completa la E/S. Esto es importante para el rendimiento, pero es un detalle de implementación en cuanto a la arquitectura. Un protocolo asincrónico significa que el remitente no espera ninguna respuesta. HTTP es un protocolo sincrónico, aunque un cliente HTTP puede utilizar operaciones de E/S asincrónicas cuando envía una solicitud. 

Cada patrón tiene sus inconvenientes. Solicitud/respuesta es un paradigma conocido, por lo que el diseño de una API puede resultar más natural que diseñar un sistema de mensajería. Sin embargo, la mensajería asincrónica tiene algunas ventajas que pueden ser muy útiles en una arquitectura de microservicios:

- **Acoplamiento reducido**. El remitente del mensaje no tiene que conocer al consumidor. 

- **Varios suscriptores**. Al utilizar un modelo de pub/sub, se pueden suscribir varios consumidores para recibir eventos. Consulte [Estilo de arquitectura basada en eventos](/azure/architecture/guide/architecture-styles/event-driven).

- **Aislamiento de errores**. Si se produce un error en el consumidor, el remitente puede seguir enviando mensajes. Los mensajes se recogerán cuando el consumidor se recupere. Esta capacidad es especialmente útil en una arquitectura de microservicios, dado que cada servicio tiene su propio ciclo de vida. Un servicio puede dejar de estar disponible o se puede sustituir por una versión más reciente en un momento dado. La mensajería asincrónica puede controlar los tiempos de inactividad intermitentes. Las API sincrónicas, por otro lado, requieren que el servicio de bajada esté disponible o se producirá un error en la operación. 
 
- **Capacidad de respuesta**. Un servicio ascendente puede responder con mayor rapidez si no espera por los servicios de bajada. Esto es especialmente útil en una arquitectura de microservicios. Si hay una cadena de dependencias de servicio (servicio A llama a B, que llama a C, y así sucesivamente), la espera de las llamadas sincrónicas puede agregar cantidades inaceptables de latencia.

- **Redistribución de la carga**. Una cola puede actuar como un búfer para redistribuir la carga de trabajo con el fin de que los receptores puedan procesar mensajes a su propio ritmo. 

- **Flujos de trabajo**. Las colas pueden usarse para administrar un flujo de trabajo, creando puntos de comprobación del mensaje después de cada paso del flujo de trabajo.

Sin embargo, el uso eficaz de la mensajería asincrónica también presenta algunos desafíos.

- **Acoplamiento con la infraestructura de mensajería**. El uso de una infraestructura de mensajería determinada puede producir el acoplamiento rígido con esa infraestructura. Esto dificultará el cambio a otra infraestructura de mensajería más adelante.

- **Latencia**. La latencia de un extremo a otro para una operación puede ser alta si las colas de mensajes se llenan.  

- **Costo**. Con un rendimiento elevado, el costo económico de la infraestructura de mensajería puede ser significativo.

- **Complejidad**. El control de mensajería asincrónica no es una tarea trivial. Por ejemplo, tiene que controlar los mensajes duplicados, mediante la desduplicación o haciendo que las operaciones sean idempotentes. También es difícil implementar la semántica de solicitud-respuesta usando mensajería asincrónica. Para enviar una respuesta, se necesitan otra cola, además de una forma de correlacionar mensajes de solicitud y respuesta.

- **Rendimiento**. Si los mensajes requieren *semántica de cola*, la cola puede convertirse en un cuello de botella en el sistema. Cada mensaje requiere por lo menos una operación de cola y una operación de eliminación de cola. Además, la semántica de cola suele necesitar algún tipo de bloqueo dentro de la infraestructura de mensajería. Si la cola es un servicio administrado, puede haber latencia adicional porque la cola es externa a la red virtual del clúster. Para mitigar estos problemas puede procesar mensajes por lotes, pero esto complica el código. Si los mensajes no requieren semántica de cola, es posible que pueda usar un *flujo* de eventos en lugar de una cola. Para más información, consulte [Estilo de arquitectura basada en eventos](../guide/architecture-styles/event-driven.md).  

## <a name="drone-delivery-choosing-the-messaging-patterns"></a>Drone Delivery: selección de los patrones de mensajería

Con estas consideraciones en mente, el equipo de desarrollo ha realizado las siguientes opciones de diseño para la aplicación Drone Delivery

- El servicio Ingestion expone una API de REST pública que las aplicaciones cliente usan para programar, actualizar o cancelar las entregas.

- El servicio Ingestion usa Event Hubs para enviar mensajes asincrónicos al servicio del Scheduler. Los mensajes asincrónicos son necesarios para implementar la nivelación de carga que se requiere para la ingesta. Para más información sobre cómo interactúan los servicios Ingestion y Scheduler, consulte [Ingesta y flujo de trabajo][ingestion-workflow].

- Todos los servicios Account, Delivery, Package, Drone y Third-party Transport exponen las API de REST internas. El servicio Scheduler llama a estas API para llevar a cabo una solicitud de usuario. Una razón para usar las API sincrónicas es que Scheduler necesita obtener una respuesta de cada uno de los servicios de bajada. Un error en cualquiera de los servicios de bajada significa un error en la operación completa. Sin embargo, un problema potencial es la cantidad de latencia que se introduce a través de una llamada a los servicios back-end. 

- Si cualquier servicio de bajada tiene un error no transitorio, toda la transacción debe marcarse como errónea. Para controlar este caso, el servicio Scheduler envía un mensaje asincrónico al Supervisor, para que el Supervisor puede programar transacciones de compensación, como se describe en el capítulo [Ingesta y flujo de trabajo][ingestion-workflow].   

- El servicio de entrega expone una API pública que los clientes pueden utilizar para obtener el estado de una entrega. En el capítulo [Puertas de enlace de API](./gateway.md), se describe cómo una puerta de enlace de API puede ocultar los servicios subyacentes al cliente, de forma que el cliente no necesita saber qué servicios exponen cada API. 

- Mientras un dron está en vuelo, el servicio Drone envía eventos que contienen la ubicación actual del dron y su estado. El servicio Delivery escucha a estos eventos para el seguimiento del estado de una entrega.

- Cuando el estado de una entrega cambia, el servicio Delivery envía un evento de estado de entrega, como `DeliveryCreated` o `DeliveryCompleted`. Cualquier servicio puede suscribirse a estos eventos. En el diseño actual, el servicio Delivery es el único suscriptor, pero más adelante puede haber otros suscriptores. Por ejemplo, los eventos pueden ir a un servicio de análisis en tiempo real. Y dado que Scheduler no tiene que esperar una respuesta, agregar más suscriptores no afecta a la ruta de acceso de flujo de trabajo principal.

![](./images/drone-communication.png)

Tenga en cuenta que los eventos de estado de entrega se derivan de los eventos de la ubicación de dron. Por ejemplo, cuando un dron llega a una ubicación de entrega y suelta un paquete, el servicio Delivery traduce esto en un evento DeliveryCompleted. Este es un ejemplo de pensamiento en términos de modelos de dominio. Como se describió anteriormente, Drone Management pertenece a un contexto independiente enlazado. Los eventos de dron proporcionan la ubicación física de un dron. Los eventos de entrega, por otro lado, representan los cambios en el estado de una entrega, que es una entidad empresarial diferente.

## <a name="using-a-service-mesh"></a>Uso de una malla de servicio

Una *malla de servicio* es una capa de software que controla la comunicación de servicio al servicio. Las mallas del servicio están diseñadas para tratar muchos de los problemas enumerados en la sección anterior, y para mover la responsabilidad de estas preocupaciones fuera de los microservicios a una capa compartida. La malla del servicio actúa como un proxy que intercepta la comunicación de red entre microservicios en el clúster. 

> [!NOTE]
> La malla de servicio es un ejemplo del [patrón Ambassador](../patterns/ambassador.md) &mdash;, un servicio de aplicación auxiliar que envía solicitudes de red en nombre de la aplicación. 

En estos momentos, las opciones principales para una malla de servicio en Kubernetes son [linkerd](https://linkerd.io/) y [Istio](https://istio.io/). Estas dos tecnologías están evolucionando rápidamente. En el momento de redactar esta guía, la versión más reciente de Istio es 0.2, osea que sigue siendo muy nuevo. De todas formas, algunas características que tienen en común linkerd y Istio incluyen: 

- El equilibrio de carga en el nivel de sesión, en función de las latencias observadas o el número de solicitudes pendientes. Esto puede mejorar el rendimiento sobre el equilibrio de carga de nivel 4 que proporciona Kubernetes. 

- Enrutamiento de nivel 7 basado en la ruta de acceso URL, encabezado de Host, versión de API u otras reglas de nivel de aplicación.

- Reintentos de solicitudes con error. Una malla de servicio entiende los códigos de error HTTP y puede volver a intentar automáticamente las solicitudes que han tenido un error. Puede configurar el número máximo de reintentos, junto con un período de tiempo de espera para limitar la latencia máxima. 

- Interrupción de circuito. Si una instancia produce errores en las solicitudes de forma consistente, la malla de servicio la marcará temporalmente como no disponible. Tras un período de interrupción, intentará la instancia de nuevo. Puede configurar el interruptor de circuito en función de diversos criterios, como el número de errores consecutivos,  

- La malla servicio captura métricas sobre llamadas entre servicios, como el volumen de solicitudes, la latencia, las tasas de error y de éxito y los tamaños de respuesta. La malla de servicio también habilita el seguimiento distribuido mediante la incorporación de información de correlación para cada salto en una solicitud.

- Autenticación mutua de TLS para las llamadas de servicio a servicio.

¿Necesita una malla de servicio? El valor que se agrega a un sistema distribuido es ciertamente atractivo. Si no dispone de una malla de servicio, debe tener en cuenta cada uno de los retos mencionados al principio del capítulo. Puede resolver problemas como reintentos, interruptores de circuito y seguimiento distribuido sin una malla de servicio, pero una malla de servicio mueve estos problemas fuera de los servicios individuales a una capa dedicada. Por otro lado, las mallas de servicio son una tecnología relativamente nueva que todavía está en proceso de maduración. La implementación de una malla de servicio agrega complejidad a la instalación y configuración del clúster. Puede haber implicaciones de rendimiento, porque las solicitudes se enrutan ahora mediante el proxy de la malla de servicio y porque ahora se ejecutan servicios adicionales en cada nodo del clúster. Antes de implementar una malla de servicio en producción, debe realizar pruebas exhaustivas de rendimiento y carga.

> [!div class="nextstepaction"]
> [Diseño de API](./api-design.md)

<!-- links -->

[ingestion-workflow]: ./ingestion-workflow.md
