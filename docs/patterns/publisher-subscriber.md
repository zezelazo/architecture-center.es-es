---
title: Patrón de publicador y suscriptor
description: Permita que una aplicación anuncie eventos de forma asincrónica a varios consumidores interesados.
keywords: Patrón de diseño
author: alexbuckgit
ms.date: 12/07/2018
ms.openlocfilehash: f47792dd60e65cac1ff06928d9ea100ed452933a
ms.sourcegitcommit: a0a9981e7586bed8d876a54e055dea1e392118f8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/11/2018
ms.locfileid: "53233943"
---
# <a name="publisher-subscriber-pattern"></a>Patrón de publicador y suscriptor

Permita que una aplicación anuncie eventos de forma asincrónica a varios consumidores interesados, sin necesidad de emparejar los remitentes con los receptores.

**También se denomina**: Mensajería de publicación y suscripción

## <a name="context-and-problem"></a>Contexto y problema

En las aplicaciones distribuidas y basadas en la nube, los componentes del sistema a menudo necesitan proporcionar información a otros componentes a medida que suceden los eventos.

La mensajería asincrónica es una forma eficaz de desacoplar a los remitentes de los consumidores y evitar bloquear al remitente para que espere una respuesta. Sin embargo, el uso de una cola de mensajes dedicada para cada consumidor no es efectivo para muchos consumidores. Además, algunos de los consumidores podrían estar interesados solo en un subconjunto de la información. ¿Cómo puede el remitente anunciar eventos a todos los consumidores interesados sin conocer su identidad?

## <a name="solution"></a>Solución

Introduzca un subsistema de mensajería asincrónica que incluya lo siguiente:

- Un canal de mensajería de entrada utilizado por el remitente. El remitente empaqueta los eventos en mensajes, mediante un formato de mensaje conocido, y envía estos mensajes a través del canal de entrada. El remitente en este patrón también se denomina *publicador*.

  > [!NOTE]
  > Un *mensaje* es un paquete de datos. Un *evento* es un mensaje que notifica a otros componentes sobre un cambio o una acción que ha tenido lugar.

- Un canal de mensajería de salida por consumidor. Los consumidores se conocen como *suscriptores*.

- Un mecanismo para copiar cada mensaje del canal de entrada a los canales de salida para todos los suscriptores interesados en ese mensaje. Esta operación la controla típicamente un intermediario tal como un agente de mensajes o un bus de eventos.

En el siguiente diagrama se muestran los componentes lógicos de este patrón:

![Patrón de publicador y suscriptor mediante un agente de mensajes](./_images/publish-subscribe.png)
 
La mensajería de publicación y suscripción tiene las siguientes ventajas:

- Desacopla los subsistemas que aún necesitan comunicarse. Los subsistemas se pueden administrar de forma independiente y los mensajes se pueden administrar correctamente incluso si uno o más receptores están desconectados.

- Aumenta la escalabilidad y mejora la capacidad de respuesta del remitente. El remitente puede enviar rápidamente un solo mensaje al canal de entrada y luego volver a sus responsabilidades de procesamiento principales. La infraestructura de mensajería es responsable de garantizar que los mensajes se entreguen a los suscriptores interesados.

- Mejora la confiabilidad. La mensajería asincrónica ayuda a que las aplicaciones continúen funcionando sin problemas bajo cargas cada vez mayores y controlen los errores intermitentes con mayor eficacia.

- Permite el procesamiento diferido o programado. Los suscriptores pueden esperar para recoger los mensajes hasta las horas de menor consumo, o los mensajes pueden enrutarse o procesarse de acuerdo a un horario específico.

- Permite una integración más sencilla entre sistemas que utilizan diferentes plataformas, lenguajes de programación o protocolos de comunicación, así como entre sistemas locales y aplicaciones que se ejecutan en la nube.

- Facilita flujos de trabajo asincrónicos en toda la empresa.

- Mejora la capacidad de prueba. Los canales pueden supervisarse y los mensajes se pueden inspeccionar o registrar como parte de una estrategia general de pruebas de integración.

- Proporciona separación de preocupaciones para sus aplicaciones. Cada aplicación puede centrarse en sus funcionalidades principales, mientras que la infraestructura de mensajería controla todo lo necesario para enrutar de forma fiable los mensajes a múltiples consumidores. 

## <a name="issues-and-considerations"></a>Problemas y consideraciones

Tenga en cuenta los puntos siguientes al decidir cómo implementar este patrón:

- **Tecnologías existentes.** Se recomienda encarecidamente utilizar los productos y servicios de mensajería disponibles que admiten un modelo de publicación y suscripción, en lugar de crear uno propio. En Azure, considere el uso de [Service Bus](/azure/service-bus-messaging/) o [Event Grid](/azure/event-grid/). Otras tecnologías que se pueden utilizar para la mensajería de publicación y suscripción incluyen Redis, RabbitMQ y Apache Kafka.

- **Control de suscripciones.** La infraestructura de mensajería debe proporcionar mecanismos que los consumidores puedan utilizar para suscribirse o darse de baja de los canales disponibles.

- **Seguridad.** La conexión a cualquier canal de mensajes debe estar restringida por una directiva de seguridad para evitar que usuarios o aplicaciones no autorizados resulten interceptados.

- **Subconjuntos de mensajes.** Por lo general, los suscriptores solo están interesados en el subconjunto de los mensajes distribuidos por un publicador. Los servicios de mensajería a menudo permiten a los suscriptores reducir el conjunto de mensajes recibidos por:

  - **Temas.** Cada tema tiene un canal de salida dedicado, y cada consumidor puede suscribirse a todos los temas pertinentes.
  - **Filtrado de contenido.** Los mensajes se inspeccionan y se distribuyen según el contenido de cada mensaje. Cada suscriptor puede especificar el contenido que le interesa.

- **Suscriptores con caracteres comodín.** Considere la posibilidad de permitir que los suscriptores se suscriban a varios temas mediante caracteres comodín.

- **Comunicación bidireccional.** Los canales en un sistema de publicación y suscripción se tratan como unidireccionales. Si un suscriptor específico necesita enviar una confirmación o comunicar el estado al editor, considere la posibilidad de utilizar el [patrón de solicitud o respuesta](http://www.enterpriseintegrationpatterns.com/patterns/messaging/RequestReply.html). Este patrón utiliza un canal para enviar un mensaje al suscriptor y un canal de respuesta separado para comunicarse con el publicador.

- **Orden de los mensajes.** El orden en que las instancias de consumidor reciben los mensajes no está garantizado y no refleja necesariamente el orden en que se crearon los mensajes. Diseñe el sistema para asegurarse de que el procesamiento de mensajes sea idempotente para eliminar cualquier dependencia en el orden en el que se administran los mensajes.

- **Prioridad del mensaje.** Algunas soluciones pueden requerir que los mensajes se procesen en un orden específico. El [patrón de cola de prioridad](priority-queue.md) proporciona un mecanismo para garantizar que determinados mensajes se entreguen antes que otros.

- **Mensajes dudosos.** Un mensaje con formato incorrecto, o una tarea que requiere acceso a recursos que no están disponibles, puede hacer que una instancia de servicio produzca un error. El sistema debe impedir que dichos mensajes vuelvan a la cola. En su lugar, capture y almacene los detalles de estos mensajes en otro lugar para que se puedan analizar si es necesario.

- **Mensajes repetidos.** El mismo mensaje se puede enviar varias veces. Por ejemplo, puede producirse un error después de que el remitente publique un mensaje. A continuación, una nueva instancia del remitente podría iniciarse y repetir el mensaje. La infraestructura de mensajería debe implementar la detección y eliminación de mensajes duplicados (también conocida como deduplicación) basada en identificadores de mensajes para proporcionar la entrega de mensajes de una sola vez.

- **Expiración de mensajes.** Un mensaje puede tener una duración limitada. Si no se procesa dentro de este período, es posible que ya no sea pertinente y se debe descartar. Un remitente puede especificar un tiempo de experimentación como parte de los datos del mensaje. Un receptor puede examinar esta información antes de decidir si desea realizar la lógica de negocios asociada con el mensaje.

- **Programación de mensajes.** Un mensaje puede estar prohibido temporalmente y no debe procesarse hasta una fecha y hora específicas. El mensaje no debe estar disponible para un receptor hasta ese momento.

## <a name="when-to-use-this-pattern"></a>Cuándo usar este patrón

Use este patrón en los siguientes supuestos:

- Una aplicación necesita difundir información a un número significativo de consumidores.

- Una aplicación necesita comunicarse con una o más aplicaciones o servicios desarrollados de forma independiente, que pueden utilizar diferentes plataformas, lenguajes de programación y protocolos de comunicación.

- Una aplicación puede enviar información a los consumidores sin requerir respuestas en tiempo real de los consumidores.

- Los sistemas que se están integrando están diseñados para admitir un eventual modelo de consistencia para sus datos.

- Una aplicación necesita comunicar información a varios consumidores, que pueden tener diferentes requisitos de disponibilidad o programaciones de tiempo de actividad que el remitente.

Este modelo podría no ser útil en las situaciones siguientes:

- Una aplicación tiene solo unos pocos consumidores que necesitan información significativamente diferente de la aplicación que la produce.

- Una aplicación requiere una interacción casi en tiempo real con los consumidores.

## <a name="example"></a>Ejemplo

El siguiente diagrama muestra una arquitectura de integración empresarial que utiliza Service Bus para coordinar los flujos de trabajo, y Event Grid notifica a los subsistemas de los eventos que ocurren. Para más información, consulte [Integración empresarial en Azure mediante colas de mensajes y eventos](../reference-architectures/enterprise-integration/queues-events.md).

![Arquitectura de integración empresarial](../reference-architectures/enterprise-integration/_images/enterprise-integration-queues-events.png)

## <a name="related-patterns-and-guidance"></a>Orientación y patrones relacionados

Los patrones y las directrices siguientes podrían ser importantes a la hora de implementar este patrón:

- [Elija entre los servicios de Azure de entrega de mensajes](/azure/event-grid/compare-messaging-services).

- El [estilo de arquitectura basada en eventos](../guide/architecture-styles/event-driven.md) es un estilo de arquitectura que utiliza la mensajería de publicación y suscripción.

- [Manual de mensajería asincrónica](https://msdn.microsoft.com/library/dn589781.aspx). Las colas de mensajes son un mecanismo de comunicaciones asincrónico. Si un servicio de consumidor debe enviar una respuesta a una aplicación, podría ser necesario implementar alguna forma de mensajería de respuesta. En Asynchronous Messaging Primer se proporciona información sobre cómo implementar la mensajería de solicitud/respuesta con colas de mensajes.

- [Patrón de observador](https://en.wikipedia.org/wiki/Observer_pattern). El patrón de publicador y suscriptor se basa en el patrón de observador al desacoplar los sujetos de los observadores a través de la mensajería asincrónica.

- [Patrón de agente de mensajes](https://en.wikipedia.org/wiki/Message_broker). Muchos subsistemas de mensajería que admiten un modelo de publicación y suscripción se implementan mediante un servidor de mensajes.
