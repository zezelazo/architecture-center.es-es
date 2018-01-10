---
title: "Análisis de dominios para microservicios"
description: "Análisis de dominios para microservicios"
author: MikeWasson
ms.date: 12/08/2017
ms.openlocfilehash: dc07c5195299c88a946accbe4e13a997afaaff90
ms.sourcegitcommit: a8453c4bc7c870fa1a12bb3c02e3b310db87530c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/29/2017
---
# <a name="designing-microservices-domain-analysis"></a>Diseño de microservicios: análisis de dominios 

Uno de los mayores desafíos de los microservicios es definir los límites de los servicios individuales. La regla general es que un servicio debe hacer "algo"; sin embargo, llevarla a la práctica requiere una minuciosa reflexión. No hay ningún proceso mecánico que cree el diseño "correcto". Debe meditar detenidamente sobre el dominio de su empresa, los requisitos y los objetivos. En caso contrario, puede terminar con un diseño incoherente que exhiba algunas características no deseables, como son dependencias ocultas entre servicios, un acoplamiento rígido o interfaces mal diseñadas. En este capítulo, se adopta un enfoque basado en dominios para el diseño de los microservicios. 

Los microservicios deben diseñarse alrededor de las funcionalidades de la empresa, no en capas horizontales como el acceso a datos o la mensajería. Además, deben tener un acoplamiento flexible y una alta cohesión funcional. Los microservicios exhiben un *acoplamiento flexible* si puede cambiar un servicio sin necesidad de que los demás se actualicen al mismo tiempo. Un microservicio es *coherente* si tiene un propósito único y bien definido, como administrar las cuentas de usuario o realizar el seguimiento del historial de entregas. Un servicio debe encapsular el conocimiento del dominio y abstraer ese conocimiento ante los clientes. Por ejemplo, un cliente debe poder programar un dron sin conocer los detalles sobre el algoritmo de programación y el modo en que la flota de drones se administra.

El diseño basado en dominios (DDD) proporciona una plataforma que puede ayudarle en gran medida a diseñar bien los microservicios. El diseño basado en dominios tiene dos fases distintas, tácticas y estratégicas. En el diseño basado en dominios estratégico, se define la estructura a gran escala del sistema. Ayuda a garantizar que la arquitectura permanece centrada en las funcionalidades del negocio. El diseño basado en dominios táctico proporciona un conjunto de modelos de diseño que puede usar para crear el modelo de dominio. Estos modelos incluyen entidades, agregados y servicios de dominio. Los modelos tácticos le ayudarán a diseñar microservicios coherentes y con acoplamiento flexible.

![](./images/ddd-process.png)

En este capítulo y en el siguiente, usaremos los siguientes pasos con la aplicación Drone Delivery: 

1. Empiece por analizar el dominio de la empresa para conocer los requisitos funcionales de la aplicación. El resultado de este paso es una descripción informal del dominio, que puede perfilarse en un conjunto más formal de modelos de dominio. 

2. A continuación, defina los *contextos delimitados* del dominio. Cada contexto delimitado contiene un modelo de dominio que representa un subdominio concreto de la aplicación mayor. 

3. Dentro de un contexto delimitado, se aplican los modelos tácticos de diseño basado en dominios para definir las entidades, los agregados y los servicios de dominio. 
 
4. Use los resultados del paso anterior para identificar los microservicios en la aplicación.

En este capítulo, trataremos los tres primeros pasos, relacionados principalmente con el diseño basado en dominios. En el próximo capítulo, se identificarán los microservicios. Sin embargo, es importante recordar que el diseño basado en dominios es un proceso iterativo y en curso. Los límites del servicio no son fijos. A medida que una aplicación evolucione, puede decidir dividir un servicio en varios menores.

> [!NOTE]
> Este capítulo no está pensado para mostrar un análisis de dominio completo y minucioso. Hemos hecho deliberadamente que el ejemplo sea breve para ilustrar los puntos principales. Para obtener más información sobre el diseño basado en dominios, se recomienda el libro de Eric Evans *Domain-Driven Design* (Diseño basado en dominios), en el que se usó el término por primera vez. Otra buena referencia es *Implementing Domain-Driven Design* (Implementación del diseño basado en dominios), de Vaughn Vernon. 

## <a name="analyze-the-domain"></a>Análisis del dominio

El empleo de un método de diseño basado en dominios le ayudará a diseñar los microservicios de modo que cada servicio constituya una solución natural para un requisito empresarial funcional. Puede servirle de ayuda para evitar la trampa que supone permitir que los límites de la organización o las elecciones de tecnología dicten el diseño.

Antes de escribir ningún código, necesita una visión general del sistema que se va a crear. Con el diseño basado en dominios, se empieza por modelar el dominio empresarial y se crea un *modelo de dominio*. El modelo de dominio es un modelo abstracto del ámbito empresarial. Extrae y organiza el conocimiento del dominio, y proporciona un lenguaje común para los desarrolladores y los expertos en los dominios. 

Empiece por asignar todas las funciones empresariales y sus conexiones. Probablemente, esto requerirá la colaboración de los expertos en dominios, los arquitectos de software y otros actores implicados. No es necesario usar ningún formalismo concreto.  Esboce un diagrama o dibuje en una pizarra.

A medida que rellene el diagrama, puede empezar a identificar subdominios discretos. ¿Qué funciones están estrechamente relacionadas? ¿Cuáles son fundamentales para la empresa y cuáles proporcionan servicios auxiliares? ¿Cuál es el gráfico de dependencias? Durante esta fase inicial, no se preocupe de las tecnologías ni de los detalles de implementación. Es decir, debe tener en cuenta el lugar donde la aplicación tendrá que integrarse con sistemas externos, como CRM o sistemas de facturación o de procesamiento de pagos. 

## <a name="drone-delivery-analyzing-the-business-domain"></a>Drone Delivery: análisis del dominio empresarial.

Después del análisis inicial del dominio, el equipo de Fabrikam ha propuesto un borrador que representa el dominio para Drone Delivery.

![](./images/ddd1.svg) 

- El elemento **Shipping** (envío) se coloca en el centro del diagrama, ya que es importante para el negocio. Todo lo demás dentro del diagrama existe para habilitar esta funcionalidad.
- El elemento **Drone management** (administración de drones) también es esencial para la empresa. La funcionalidad que está estrechamente relacionada con la anterior es **Drone repair** (reparación de drones) y el uso de **predictive analysis**  (análisis predictivos) que permiten predecir el momento en que los drones tienen que someterse a tareas de mantenimiento. 
- **ETA analysis** (análisis del tiempo estimado de llegada) proporciona estimaciones de tiempo para la recogida y la entrega. 
- La funcionalidad **Third-party transportation** (transporte de terceros) permitirá que la aplicación programe métodos de transporte alternativo, si un dron no puede entregar un paquete completo.
- La funcionalidad **Drone sharing** (uso compartido de drones) es una posible extensión del negocio principal. La empresa puede tener drones de sobra durante ciertas horas y alquilar el excedente que, de otro modo, permanecería inactivo. Esta característica no estará en la versión inicial.
- La funcionalidad **Video surveillance** (vigilancia por vídeo) es otra área que la empresa podría expandir en versiones posteriores.
- **User accounts** (cuentas de usuario), **Invoicing** (facturación) y **Call center** (centro de llamadas) son subdominios que contribuyen al negocio principal.
 
Tenga en cuenta que, en este punto del proceso, no hemos tomado decisiones sobre la implementación o las tecnologías. Algunos de los subsistemas pueden implicar a sistemas de software externos o servicios de terceros. Aun así, la aplicación requiere interactuar con estos sistemas y servicios, por lo que es importante incluirlos en el modelo de dominio. 

> [!NOTE]
> Cuando una aplicación depende de un sistema externo, existe el riesgo de que el esquema de datos o la API del sistema externo se infiltren en la aplicación, poniendo en peligro a la larga el diseño de la arquitectura. Esto ocurre especialmente con sistemas heredados que pueden no seguir procedimientos recomendados modernos y podrían usar esquemas de datos complejos o API obsoletas. En ese caso, es importante contar con un límite bien definido entre dichos sistemas externos y la aplicación. Considere el uso del [patrón Strangler](../patterns/strangler.md) o del [patrón Anti-Corruption Layer](../patterns/anti-corruption-layer.md) para este propósito.

## <a name="define-bounded-contexts"></a>Definición de contextos delimitados

El modelo de dominio incluirá representaciones de cosas reales del mundo, como son usuarios, drones, paquetes y otras. Pero eso no significa que todas las partes del sistema deban usar las mismas representaciones para las mismas cosas. 

Por ejemplo, los subsistemas que controlan la reparación de drones y el análisis predictivo tendrán que representar los drones con muchas características físicas, como su historial de mantenimiento, kilometraje, antigüedad, número de modelo, características de rendimiento, etcétera. Pero, cuando llega el momento de programar una entrega, no importan esos elementos. El subsistema de programación solo necesita saber si un dron está disponible y el tiempo estimado para la recogida y la entrega. 

Si se ha intentado crear un modelo único para ambos subsistemas, sería innecesariamente complejo. También resultaría más difícil para el modelo evolucionar con el tiempo, porque los cambios deberán satisfacer a varios equipos que trabajen en subsistemas independientes. Por lo tanto, a menudo es mejor diseñar modelos independientes que representen la misma entidad del mundo real (en este caso, un dron) en dos contextos diferentes. Cada modelo contiene solo las características y los atributos que sean pertinentes en su contexto determinado.

Aquí es donde entra en juego el concepto del diseño basado en dominios referente a los *contextos limitados*. Un contexto delimitado es simplemente el límite dentro de un dominio donde se aplica un modelo de dominio en particular. Si examinamos el diagrama anterior, podemos agrupar la funcionalidad teniendo en cuenta si varias funciones compartirán un único modelo de dominio. 

![](./images/ddd2.svg) 
 
Los contextos delimitados no están necesariamente aislados entre sí. En este diagrama, las líneas continuas que conectan los contextos delimitados representan los lugares donde dos de ellos interactúan. Por ejemplo, el envío depende de las cuentas de usuario (Accouns) para obtener información sobre los clientes y de la administración de drones (Drone management) para programar los de la flota.

En el libro *Domain Driven Design* de Eric Evans, se describen varios patrones para mantener la integridad de un modelo de dominio cuando interactúa con otro contexto delimitado. Uno de los principios fundamentales de los microservicios es que los servicios se comunican a través de API bien definidas. Este método se corresponde con dos patrones que Evans llama Open Host Service (servicio de host abierto) y Published Language (lenguaje publicado). La idea subyacente en Open Host Service es que un subsistema define un protocolo formal (API) para que otros se comuniquen con él. Published Language amplía esta idea publicando la API de forma que otros equipos puedan usarla para escribir clientes. En el capítulo sobre el [diseño de API](./api-design.md), se trata sobre el uso de [OpenAPI Specification](https://www.openapis.org/specification/repo) (Especificación de OpenAPI), conocida anteriormente como Swagger, para definir las descripciones de la interfaz independiente del lenguaje para las API de REST, expresadas en formato JSON o YAML.

En el resto de este viaje, nos centraremos en el contexto delimitado del envío. 

## <a name="tactical-ddd"></a>Diseño basado en dominios táctico

Durante la fase estratégica del diseño basado en dominios (DDD), se asigna el dominio empresarial y se definen los contextos delimitados para los modelos de dominio. El diseño basado en dominios táctico consiste en definir los modelos de dominio con más precisión. Los patrones tácticos se aplican dentro de un único contexto delimitado. En una arquitectura de microservicios, interesan especialmente los patrones de agregados y entidades. Aplicar estos patrones nos ayudará a identificar los límites naturales de los servicios de nuestra aplicación (consulte el [siguiente capítulo](./microservice-boundaries.md)). Como principio general, un microservicio no debe ser menor que un agregado ni mayor que un contexto delimitado. En primer lugar, revisaremos los patrones tácticos. A continuación, se podrán aplicar al contexto delimitado de envío, en la aplicación Drone Delivery. 

### <a name="overview-of-the-tactical-patterns"></a>Información general sobre los patrones tácticos

En esta sección se proporciona un breve resumen de los patrones tácticos del diseño basado en dominios, por lo que, si ya está familiarizado con él, podría omitir esta sección. Los patrones se describen con más detalle en los capítulos 5 y 6 del libro de Eric Evans, y en el libro *Implementing Domain-Driven Design* de Vaughn Vernon. 

![](./images/ddd-patterns.png)

**Entidades**. Una entidad es un objeto con una identidad única que persiste en el tiempo. Por ejemplo, en una aplicación bancaria, las cuentas y los clientes serían entidades. 

- Una entidad tiene un identificador único en el sistema, que se puede usar para buscar la entidad o para recuperarla. Eso no significa que el identificador siempre se exponga directamente a los usuarios. Podría ser un identificado único (GUID) o una clave principal de una base de datos. 
- Una identidad puede abarcar varios contextos delimitados y puede durar más que la aplicación. Por ejemplo, los números de cuentas bancarias o los identificadores emitidos por entidades gubernamentales no están asociados a la duración de una aplicación en particular.
- Los atributos de una entidad pueden cambiar con el tiempo. Por ejemplo, el nombre o la dirección de una persona pueden variar, pero ella sigue siendo la misma. 
- Una entidad puede contener referencias a otras entidades.
 
**Objetos de valor**. Un objeto de valor no tiene identidad. Se define únicamente mediante los valores de sus atributos. Los objetos de valor también son inmutables. Para actualizar un objeto de valor, siempre hay que crear una nueva instancia que reemplace a la anterior. Los objetos de valor pueden tener métodos que encapsulen la lógica del dominio, pero esos métodos no deben afectar al estado del objeto. Ejemplos típicos de objetos de valor son los colores, las fechas y horas, y los valores de divisa. 

**Agregados**. Un agregado define un límite de coherencia alrededor de una o varias entidades. Una entidad exacta en un agregado es la raíz. La búsqueda se realiza con el identificador de la entidad raíz. Cualquier otra entidad en el agregado es secundaria de la raíz y se hace referencia a ella siguiendo punteros desde esta. 

El propósito de un agregado es modelar las invariantes transaccionales. Las cosas en el mundo real tienen redes complejas de relaciones. Los clientes crean pedidos, los pedidos contienen productos, los productos tienen proveedores y así sucesivamente. Si la aplicación modifica varios objetos relacionados, ¿cómo podemos garantizar la coherencia? ¿Cómo se realiza el seguimiento de los elementos invariables y cómo se aplican?  

A menudo, las aplicaciones tradicionales han usado las transacciones de base de datos para asegurar la coherencia. En una aplicación distribuida, sin embargo, eso a menudo no es factible. Una sola transacción comercial puede abarcar varios almacenes de datos, puede ser de ejecución prolongada o implicar a servicios de terceros. En última instancia, depende de la aplicación, no del nivel de datos, el aplicar los valores invariables necesarios para el dominio. Eso es lo que los agregados se supone que modelan.

> [!NOTE]
> Un agregado puede constar de una sola entidad, sin entidades secundarias. Lo que lo convierte en agregado es el límite transaccional.

**Servicios de aplicación y de dominio**. En la terminología del diseño basado en dominios, un servicio es un objeto que implementa alguna lógica sin mantener ningún estado. Evans distingue entre *servicios de dominio*, que encapsulan la lógica del dominio, y *servicios de aplicación*, que proporcionan la funcionalidad técnica, como la autenticación del usuario o el envío de un mensaje SMS. Los servicios de dominio a menudo se utilizan para modelar el comportamiento que abarca varias entidades. 

> [!NOTE]
> El término *servicio* está saturado en el desarrollo de software. La definición aquí no está relacionada directamente con los microservicios.

**Eventos de dominio**. Los eventos de dominio se pueden utilizar para notificar a otras partes del sistema cuando sucede algo. Como sugiere su nombre, los eventos de dominio deben significar algo dentro del dominio. Por ejemplo, "se inserta un registro en una tabla" no es un evento de dominio. "Se canceló una entrega" es un evento de dominio. Los eventos de dominio son especialmente importantes en una arquitectura de microservicios. Dado que los microservicios se distribuyen y no comparten los almacenes de datos, los eventos de dominio proporcionan una manera de que los microservicios se coordinen entre sí. En el capítulo [Comunicación entre servicios](./interservice-communication.md) se describe la mensajería asincrónica con más detalle.
 
Hay otros patrones del diseño basado en dominios que no se mencionan aquí, como son los generadores, los repositorios y los módulos. Pueden ser patrones útiles cuando se vaya a implementar un microservicio, pero son menos importantes al diseñar los límites entre microservicios.

## <a name="drone-delivery-applying-the-patterns"></a>Drone Delivery: aplicación de patrones

Empezaremos con los escenarios que el contexto delimitado de envío debe controlar.

- Un cliente puede solicitar un dron para recoger las mercancías de una empresa que se registre en el servicio de entrega de drones.
- El remitente genera una etiqueta (código de barras o RFID) para colocar en el paquete. 
- Un dron recogerá el paquete en la ubicación de origen y lo entregará en la ubicación de destino.
- Cuando un cliente programa una entrega, el sistema proporciona un tiempo estimado de llegada (ETA) según la información de la ruta, las condiciones meteorológicas y los datos históricos. 
- Cuando el dron está en vuelo, el usuario puede realizar el seguimiento de la ubicación actual y del tiempo estimado de llegada más reciente. 
- Hasta que un dron recoja el paquete, el cliente puede cancelar una entrega.
- Se notifica al cliente cuándo se completa la entrega.
- El remitente puede solicitar al cliente la confirmación de la entrega, en forma de una firma o de una huella digital.
- Los usuarios pueden ver el historial de una entrega completada.

En estos casos, el equipo de desarrollo identificó las siguientes **entidades**.

- Entrega
- Paquete
- Dron
- Cuenta
- Confirmación
- Notificación
- Etiqueta

Las cuatro primeras (entrega, paquete, dron y cuenta) son todos **agregados** que representan los límites de la coherencia transaccional. Las confirmaciones y las notificaciones son las entidades secundarias de las entregas y las etiquetas son las entidades secundarias de los paquetes. 

Los **objetos de valor** en este diseño incluyen la ubicación (Location), el tiempo estimado (ETA), el peso del paquete (PackageWeight) y su tamaño (PackageSize). 

Para ilustrar esto, se proporciona un diagrama UML del agregado de entrega. Tenga en cuenta que contiene referencias a otros agregados, como son la cuenta, el paquete y el dron.

![](./images/delivery-entity.png)

Hay dos eventos de dominio:

- Mientras un dron está volando, la entidad Dron envía eventos DroneStatus que describen la ubicación y el estado del dron (en vuelo, en tierra).

- La entidad de entrega (Delivery) envía eventos de seguimiento de la entrega (DeliveryTracking) cada vez que cambia la fase de una entrega. Entre estos, se incluyen los de creación, reprogramación, preparada y completada (DeliveryCreated, DeliveryRescheduled, DeliveryHeadedToDropoff y DeliveryCompleted, respectivamente). 

Tenga en cuenta que estos eventos describen aspectos significativos dentro del modelo de dominio. Describen algo sobre este y no están vinculados a una construcción de un lenguaje de programación determinado.

El equipo de desarrollo identificó un área más de funcionalidad, que no encaja claramente en ninguna de las entidades descritas hasta ahora. Una parte del sistema debe coordinar todos los pasos implicados en la programación o actualización de una entrega. Por lo tanto, el equipo de desarrollo agrega dos **servicios de dominio** al diseño: un *programador* (Scheduler) que coordina los pasos y un *supervisor* que supervisa el estado de cada paso, con el fin de detectar si en alguno se generó un error o se agotó su tiempo asignado. Esta es una variación del patrón [Scheduler Agent Supervisor](../patterns/scheduler-agent-supervisor.md) (supervisor del agente de programación).

![](./images/drone-ddd.png)

> [!div class="nextstepaction"]
> [Identificación de los límites de los microservicios](./microservice-boundaries.md)
