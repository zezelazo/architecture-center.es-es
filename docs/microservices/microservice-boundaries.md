---
title: "Identificación de los límites de los microservicios"
description: "Identificación de los límites de los microservicios"
author: MikeWasson
ms.date: 12/08/2017
ms.openlocfilehash: 046749191bd565813218b3834cb4674c4c5100e2
ms.sourcegitcommit: a8453c4bc7c870fa1a12bb3c02e3b310db87530c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/29/2017
---
# <a name="designing-microservices-identifying-microservice-boundaries"></a>Diseño de microservicios: identificación de los límites de los microservicios

¿Cuál es el tamaño correcto para un microservicio? Seguramente oirá afirmaciones del tipo "ni demasiado grande ni demasiado pequeño", y aunque esto es ciertamente correcto, no resulta muy útil en la práctica. Pero si se empieza desde un modelo de dominio cuidadosamente diseñado, es mucho más fácil razonar acerca de los microservicios.

![](./images/bounded-contexts.png)

## <a name="from-domain-model-to-microservices"></a>Desde un modelo de dominio a los microservicios

En el [capítulo anterior](./domain-analysis.md), definimos un conjunto de contextos delimitados para la aplicación Drone Delivery. Luego observamos más de cerca uno de estos contextos delimitados, el contexto delimitado Shipping, e identificamos un conjunto de entidades, agregados, y servicios de dominio para ese contexto delimitado.

Ahora estamos preparados para pasar del modelo de dominio al diseño de la aplicación. Este es un enfoque que puede usar para derivar microservicios desde el modelo de dominio.

1. Comience con un contexto delimitado. En general, la funcionalidad en un microservicio no debe abarcar más de un contexto delimitado. Por definición, un contexto delimitado marca el límite de un modelo de dominio en particular. Si encuentra que un microservicio mezcla modelos de dominio diferentes, es un signo de que puede que tenga que volver atrás para refinar el análisis de dominio.

2. A continuación, mire los agregados en el modelo de dominio. Los agregados suelen ser buenos candidatos para microservicios. Un agregado bien diseñado tiene muchas de las características de un microservicio bien diseñado, como:

    - Un agregado se deriva de los requisitos empresariales más que de las preocupaciones técnicas, como el acceso a datos o la mensajería.  
    - Un agregado debe tener alta cohesión funcional.
    - Un agregado es un límite de persistencia.
    - Los agregados deben tener un acoplamiento flexible. 
    
3. Los servicios de dominio también son buenos candidatos para microservicios. Los servicios de dominio son operaciones sin estado a través de varios agregados. Un ejemplo típico es un flujo de trabajo que implica varios microservicios. Veremos un ejemplo de esto en la aplicación Drone Delivery.

4. Por último, tenga en cuenta los requisitos no funcionales. Observe factores como el tamaño del equipo, los tipos de datos, tecnologías, requisitos de escalabilidad, requisitos de disponibilidad y requisitos de seguridad. Estos factores pueden llevarle a volver a dividir un microservicio en dos o más servicios más pequeños, o a hacer lo contrario combinando varios microservicios en uno. 

Después de identificar los microservicios en su aplicación, valide el diseño con los siguientes criterios:

- Cada servicio tiene una única responsabilidad.
- No hay llamadas que generen mucha conversación entre servicios. Si dividir la funcionalidad en dos servicios hace que estos generen demasiada conversación, esto puede ser una indicación de que estas funciones deben estar en el mismo servicio.
- Cada servicio es lo suficientemente pequeño como para que un equipo pequeño lo pueda generar trabajando de forma independiente.
- No hay interdependencias que requieran la implementación de dos o más servicios en sincronía. Siempre debe ser posible implementar un servicio sin tener que volver a implementar ninguno de los demás servicios.
- Los servicios no están estrechamente acoplados y pueden evolucionar independientemente.
- Los límites de servicio no causarán problemas con la coherencia de datos o la integridad. A veces es importante mantener la coherencia de datos colocando la funcionalidad en un único microservicio. Dicho esto, considere si realmente necesita una fuerte coherencia. Hay estrategias para resolver la coherencia final en un sistema distribuido, y las ventajas de descomponer servicios generalmente superan los desafíos que supone administrar la coherencia final.

Sobre todo, es importante ser práctico y recordar que el diseño basado en dominio es un proceso iterativo. En caso de duda, empiece con microservicios más generales. Dividir un microservicio en dos servicios más pequeños es más sencillo que la refactorización de funcionalidad entre varios microservicios existentes.
  
## <a name="drone-delivery-defining-the-microservices"></a>Drone Delivery: definición de los microservicios

Recuerde que el equipo de desarrollo había identificado los cuatro agregados &mdash;Delivery, Package, Drone, y Account&mdash; y dos servicios de dominio, Scheduler y Supervisor. 

Delivery y Package son candidatos obvios para los microservicios. Scheduler y Supervisor coordinan las actividades realizadas por otros microservicios, por lo que tiene sentido implementar estos servicios de dominio como microservicios.  

Drone y Account son interesantes porque pertenecen a otros contextos delimitados. Una opción es que Scheduler llame a los contextos delimitados Drone y Account directamente. Otra opción es crear microservicios Drone y Account dentro del contexto delimitado Shipping. Estos microservicios mediarán entre los contextos delimitados a través de la exposición de las API o los esquemas de datos que son más adecuados para el contexto Shipping.

Los detalles de los contextos delimitados Drone y Account quedan fuera del ámbito de esta guía, por lo que hemos creado servicios ficticios para ellos en nuestra implementación de referencia. Pero hay algunos factores a tener en cuenta en esta situación:

- ¿Cuál es la sobrecarga de la red al llamar directamente al otro contexto delimitado? 

- ¿Es el esquema de datos para el otro contexto delimitado adecuado para este contexto, o es mejor tener un esquema que se adapte a este contexto delimitado? 

- ¿Es el otro contexto delimitado un sistema heredado? Si es así, podría crear un servicio que actúe como una [capa para evitar daños](../patterns/anti-corruption-layer.md) para traducir entre el sistema heredado y la aplicación moderna. 

- ¿Cuál es la estructura del equipo? ¿Es fácil comunicarse con el equipo que se encarga del otro contexto delimitado? De lo contrario, la creación de un servicio que medie entre los dos contextos puede ayudar a mitigar el costo de la comunicación entre equipos.

Hasta ahora, no hemos considerado ningún requisito no funcional. Pensando en los requisitos de rendimiento de la aplicación, el equipo de desarrollo decidió crear un microservicio Ingestion independiente, que se encarga de la ingesta de las solicitudes de cliente. Este microservicio implementará la [redistribución de la carga](../patterns/queue-based-load-leveling.md) colocando las solicitudes entrantes en un búfer para su procesamiento. Scheduler leerá las solicitudes desde el búfer y ejecutará el flujo de trabajo. 

Los requisitos no funcionales llevaron al equipo a crear un servicio adicional. Todos los servicios hasta ahora se han dedicado al proceso de programación y entrega de paquetes en tiempo real. Pero el sistema también necesita almacenar el historial de cada entrega en un almacenamiento a largo plazo para el análisis de datos. El equipo consideró hacer que esto fuera responsabilidad del servicio Delivery. Sin embargo, los requisitos de almacenamiento de datos para el análisis histórico son muy distintos a los de las operaciones en vuelo (consulte [Consideraciones de datos](./data-considerations.md)). Por lo tanto, el equipo decidió crear un servicio Delivery History independiente que realizará escuchas de eventos DeliveryTracking del servicio Delivery, y escribirá los eventos en un almacenamiento a largo plazo.

En el siguiente diagrama se muestra el proceso en este punto:
 
![](./images/microservices.png)

## <a name="choosing-a-compute-option"></a>Selección de una opción de proceso

El término *proceso* hace referencia al modelo de hospedaje para los recursos informáticos donde se ejecutan las aplicaciones. Para una arquitectura de microservicios, hay dos enfoques especialmente populares:

- Un orquestador de servicios que administra los servicios que se ejecutan en nodos dedicados (máquinas virtuales).
- Una arquitectura sin servidor mediante funciones como un servicio (FaaS). 

Aunque estas no son las únicas opciones, ambas son métodos probados para generar microservicios. Una aplicación puede incluir ambos enfoques.

### <a name="service-orchestrators"></a>Orquestadores de servicios

Un orquestador controla las tareas relacionadas con la implementación y administración de un conjunto de servicios. Estas tareas incluyen colocar servicios en los nodos, supervisar el estado de los servicios, reiniciar los servicios en mal estado, equilibrar la carga del tráfico de red en todas las instancias de servicio, detectar servicios, escalar el número de instancias de un servicio y aplicar actualizaciones de configuración. Entre los orquestadores más populares se encuentran Kubernetes, DC/OS, Docker Swarm y Service Fabric. 

- [Azure Container Service](/azure/container-service/) (ACS) es un servicio de Azure que le permite implementar un clúster de Kubernetes, DC/OS o Docker Swarm listo para producción.

- [AKS (Azure Container Service)](/azure/aks/) es un servicio administrado de Kubernetes. AKS aprovisiona Kubernetes y expone los puntos de conexión de la API de Kubernetes, pero hospeda y administra el plano de control de Kubernetes, realizando actualizaciones automatizadas, correcciones automatizadas, autoescala y otras tareas de administración. Se puede considerar AKS como "API de Kubernetes como servicio". En el momento de redactar este texto, AKS sigue aún en versión preliminar. De todas formas, se espera que AKS se convertirá en la mejor manera de ejecutar Kubernetes en Azure. 

- [Service Fabric](/azure/service-fabric/) es una plataforma de sistemas distribuidos para empaquetar, implementar y administrar microservicios. Los microservicios pueden implementarse en Service Fabric como contenedores, como archivos ejecutables binarios o como [Reliable Services](/azure/service-fabric/service-fabric-reliable-services-introduction). Usando el modelo de programación de Reliable Services, los servicios pueden utilizar directamente las API de programación de Service Fabric para consultar el sistema, informar del estado, recibir notificaciones acerca de la configuración y los cambios de código y detectar otros servicios. Una diferencia clave con Service Fabric es el enfoque prioritario dado a la creación de servicios con estado mediante [Reliable Collections](/azure/service-fabric/service-fabric-reliable-services-reliable-collections).

### <a name="containers"></a>Contenedores

A veces se habla de contenedores y microservicios como si fueran lo mismo. Aunque esto no es cierto (los contenedores no son necesarios para generar microservicios), los contenedores tienen algunas de las ventajas que son especialmente apropiadas para los microservicios, como:

- **Portabilidad**. Una imagen de contenedor es un paquete independiente que se ejecuta sin necesidad de instalar las bibliotecas u otras dependencias. Esto hace que sean fáciles de implementar. Los contenedores pueden iniciarse y detenerse rápidamente, por lo que puede poner en marcha nuevas instancias para controlar más carga o para recuperarse de errores de nodo. 

- **Densidad**. Los contenedores son ligeros en comparación con la ejecución de una máquina virtual, ya que comparten recursos de sistema operativo. Esto permite empaquetar varios contenedores en un nodo único, lo que es especialmente útil cuando la aplicación consta de muchos servicios pequeños.

- **Aislamiento de recursos**. Puede limitar la cantidad de memoria y CPU que están disponibles para un contenedor, lo que puede ayudar a garantizar que un proceso descontrolado no agotará los recursos del host. Consulte el [patrón Bulkhead](../patterns/bulkhead.md) para más información.

### <a name="serverless-functions-as-a-service"></a>Arquitectura sin servidor (funciones como un servicio)

Con una arquitectura sin servidor, no se administran las máquinas virtuales ni la infraestructura de red virtual. En su lugar, se implementa código y el servicio de hospedaje controla la colocación de ese código en una máquina virtual y lo ejecuta. Este enfoque tiende a favorecer a las pequeñas funciones pormenorizadas que se coordinan mediante el uso de los desencadenadores basados en eventos. Por ejemplo, un mensaje que se colocan en una cola puede desencadenar una función que lee de la cola y procesa el mensaje.

[Azure Functions][functions] es un servicio de proceso sin servidor que admite varios desencadenadores de función, incluidas las solicitudes HTTP, las colas de Service Bus y los eventos de Event Hubs. Si desea una lista completa consulte [Conceptos básicos sobre los enlaces y desencadenadores de Azure Functions][functions-triggers]. Tenga en cuenta también a [Azure Event Grid][event-grid], que es un servicio de enrutamiento de evento administrado de Azure.

### <a name="orchestrator-or-serverless"></a>¿Orquestador o sin servidor?

Estos son algunos de los factores a considerar a la hora de elegir entre un enfoque con orquestador y un enfoque sin servidor.

**Capacidad de administración** una aplicación sin servidor es fácil de administrar, porque la plataforma administra todos los recursos de proceso. Un orquestador aunque abstrae algunos aspectos de la administración y la configuración de un clúster, no oculta por completo las máquinas virtuales subyacentes. Con un orquestador, tendrá que pensar en problemas, como el equilibrio de carga, uso de CPU y memoria y funciones de red.

**Flexibilidad y control**. Un orquestador le ofrece un gran control sobre la configuración y administración de los servicios y del clúster. Como contrapartida tiene una complejidad adicional. Con una arquitectura sin servidor, se renuncia a un cierto grado de control porque se extraen estos detalles.

**Portabilidad**. Todos los orquestadores enumerados aquí (Kubernetes, DC/OS, Docker Swarm y Service Fabric) se pueden ejecutar de forma local o en varias nubes públicas. 

**Integración de aplicaciones**. Puede resultar complicado crear una aplicación compleja con una arquitectura sin servidor. Una opción en Azure consiste en usar [Azure Logic Apps](/azure/logic-apps/) para coordinar un conjunto de Azure Functions. Para encontrar un ejemplo de este enfoque consulte [Creación de una función que se integre con Azure Logic Apps](/azure/azure-functions/functions-twitter-email.)

**Costo**. Con un orquestador, se paga por las máquinas virtuales que se ejecutan en el clúster. Con una aplicación sin servidor, solo se paga por el consumo real de los recursos de proceso. En ambos casos, debe tener en cuenta el costo de los servicios adicionales, como almacenamiento, bases de datos y servicios de mensajería.

**Escalabilidad**. Azure Functions se escala automáticamente para satisfacer la demanda en función del número de eventos de entrada. Con un orquestador, puede escalar horizontalmente aumentando el número de instancias de servicio que se ejecutan en el clúster. También puede escalar agregando máquinas virtuales adicionales al clúster.

Nuestra implementación de referencia, usa principalmente Kubernetes, pero hemos usado Azure Functions para un servicio: el servicio Delivery History. Azure Functions era una buena opción para este servicio en concreto porque es una carga de trabajo basada en eventos. Mediante el uso de un desencadenador de Event Hubs para invocar la función, el servicio necesitó una cantidad mínima de código. Además, el servicio Delivery History no forma parte del flujo de trabajo principal, por lo que ejecutarlo fuera del clúster Kubernetes no afecta a la latencia de un extremo a otro de las operaciones iniciadas por el usuario. 

> [!div class="nextstepaction"]
> [Consideraciones sobre los datos](./data-considerations.md)

<!-- links -->

[acs-engine]: https://github.com/Azure/acs-engine
[acs-faq]: /azure/container-service/dcos-swarm/container-service-faq
[event-grid]: /azure/event-grid/
[functions]: /azure/azure-functions/functions-overview
[functions-triggers]: /azure/azure-functions/functions-triggers-bindings
