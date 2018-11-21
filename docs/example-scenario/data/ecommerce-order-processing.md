---
title: Procesamiento de pedidos escalable en Azure
description: Cree una canalización de procesamiento de pedidos altamente escalable con Azure Cosmos DB.
author: alexbuckgit
ms.date: 07/10/2018
ms.openlocfilehash: 1c3bb2cc33be74f5ff8ee0513de4c3f7df70aa37
ms.sourcegitcommit: 0a31fad9b68d54e2858314ca5fe6cba6c6b95ae4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/13/2018
ms.locfileid: "51610862"
---
# <a name="scalable-order-processing-on-azure"></a>Procesamiento de pedidos escalable en Azure

Este escenario de ejemplo es útil para aquellas organizaciones que necesitan una arquitectura resistente y altamente escalable para el procesamiento de pedidos en línea. Entre las posibles aplicaciones se incluyen aplicaciones de comercio electrónico, puntos de venta, realización de pedidos y reserva y seguimiento de inventario. 

Este escenario emplea un enfoque de aprovisionamiento de eventos mediante un modelo de programación funcional que se implementa a través de microservicios. Cada microservicio se trata como un procesador de flujos de datos y toda la lógica de negocios se implementa mediante microservicios. Este enfoque permite obtener alta disponibilidad y resistencia, replicación geográfica y un rendimiento rápido.

El uso de servicios de Azure administrados como Cosmos DB y HDInsight puede ayudarle a reducir los costos aprovechando la experiencia de Microsoft en almacenamiento y recuperación de datos globalmente distribuidos de escala en la nube. Este escenario es adecuado específicamente para un escenario de comercio electrónico o de venta. Si tiene otras necesidades de servicios de datos, revise la lista de [servicios de bases de datos inteligentes totalmente administrados de Azure][product-category].

## <a name="relevant-use-cases"></a>Casos de uso pertinentes

Otros casos de uso pertinentes incluyen:

* Sistemas back-end de comercio electrónico o punto de venta.
* Sistemas de administración de inventario.
* Sistemas de realización de pedidos.
* Otros escenarios de integración pertinentes para una canalización de procesamiento de pedidos.

## <a name="architecture"></a>Arquitectura

![Arquitectura de ejemplo de una canalización de procesamiento de pedidos escalable][architecture]

Esta arquitectura detalla los componentes clave de una canalización de procesamiento de pedidos. Los datos fluyen por el escenario de la siguiente manera:

1. Los mensajes de eventos entran en el sistema a través de aplicaciones orientadas al cliente (sincrónicamente a través de HTTP) y de varios sistemas back-end (asincrónicamente a través de Apache Kafka). Estos mensajes se pasan a una canalización de procesamiento de comandos.
2. Un microservicio de procesador de comandos ingiere y asigna cada mensaje de evento a un comando de un conjunto definido. El procesador de comandos recupera cualquier información relacionada con el estado actual para ejecutar el comando desde una base de datos de instantáneas de secuencias de eventos. A continuación, se ejecuta el comando y la salida de este se emite como un nuevo evento.
3. Cada evento que se emite como salida de un comando se confirma en una base de datos de secuencias de eventos mediante Cosmos DB.
4. Con cada inserción o actualización de base de datos que se confirma en la base de datos de secuencias de eventos, la fuente de cambios de Cosmos DB crea un evento. Los sistemas de nivel final se pueden suscribir a todos los temas de eventos que sean pertinentes para ese sistema.
5. Todos los eventos de la fuente de cambios de Cosmos DB se envían también a un microservicio de secuencia de eventos de instantáneas que calcula los cambios de estado provocados por los eventos que se han producido. Posteriormente, el nuevo estado se confirma en la base de datos de instantáneas de secuencias de eventos que está almacenada en Cosmos DB. Esta base de datos de instantáneas proporciona un origen de datos distribuido globalmente y con baja latencia para el estado actual de todos los elementos de datos. La base de datos de secuencias de eventos proporciona un registro completo de todos los mensajes de eventos que han pasado a través de la arquitectura, lo cual permite escenarios eficaces de pruebas, solución de problemas y recuperación ante desastres.

### <a name="components"></a>Componentes

* [Cosmos DB](/azure/cosmos-db/introduction) es un servicio de Microsoft, de bases de datos multimodelo y de distribución global que permite a las soluciones escalar de forma elástica e independiente el rendimiento y el almacenamiento en cualquier número de regiones geográficas. Ofrece garantía de rendimiento, latencia, disponibilidad y coherencia con Acuerdos de Nivel de Servicio (SLA) integrales. En este escenario se emplea Cosmos DB para el almacenamiento de flujos de eventos y de instantáneas, y aprovecha las características de la [fuente de cambios de Cosmos DB][docs-cosmos-db-change-feed] para proporcionar coherencia de datos y recuperación de errores.
* [Apache Kafka en HDInsight](/azure/hdinsight/kafka/apache-kafka-introduction) es una implementación de servicio administrado de Apache Kafka, una plataforma de streaming distribuida de código abierto para compilar canalizaciones y aplicaciones de streaming de datos en tiempo real. Kafka también proporciona una funcionalidad de agente de mensajería similar a una cola de mensajes, donde puede publicar y suscribirse a los flujos de datos con nombre. Este escenario utiliza Kafka para procesar los eventos entrantes así como los eventos de nivel inferior en la canalización de procesamiento de pedidos. 

## <a name="considerations"></a>Consideraciones

Existen muchas opciones tecnológicas de ingesta de mensajes, almacenamiento de datos, procesamiento de secuencias, almacenamiento de datos analíticos en tiempo real, y de análisis e informes. Para obtener información general sobre estas opciones, sus funcionalidades y los principales criterios de selección, consulte [Arquitecturas de macrodatos: Procesamiento en tiempo real](/azure/architecture/data-guide/technology-choices/real-time-ingestion) en la [Guía de arquitectura de datos de Azure](/azure/architecture/data-guide).

Los microservicios se han convertido en un estilo arquitectónico popular para la compilación de aplicaciones en la nube resistentes, altamente escalables, que se pueden implementar de forma independiente y que evolucionan rápidamente. Los microservicios requieren un enfoque diferente para diseñar y compilar aplicaciones. Herramientas como Docker, Kubernetes, Azure Service Fabric y Nomad permiten el desarrollo de arquitecturas basadas en microservicios. Para obtener instrucciones sobre la creación y ejecución de una arquitectura basada en microservicios, consulte [Diseño de microservicios en Azure] (/azure/architecture/microservices) en Azure Architecture Center.

### <a name="availability"></a>Disponibilidad

El enfoque de aprovisionamiento de eventos de este escenario permite a los componentes del sistema acoplarse de forma flexible e implementarse independientemente unos de otros. Cosmos DB ofrece [alta disponibilidad][docs-cosmos-db-regional-failover] y ayuda a las organizaciones a administrar los inconvenientes asociados con la coherencia, disponibilidad y rendimiento, todo ello con [las correspondientes garantías][docs-cosmos-db-guarantees]. Apache Kafka en HDInsight está también diseñado para [alta disponibilidad][docs-kafka-high-availability].

Azure Monitor proporciona interfaces de usuario unificadas para la supervisión de distintos servicios de Azure. Para más información, consulte [Supervisión en Microsoft Azure](/azure/monitoring-and-diagnostics/monitoring-overview). Event Hubs y Stream Analytics se integran ambos con Azure Monitor. 

Para otras consideraciones sobre disponibilidad, consulte la [lista de comprobación de disponibilidad][availability].

### <a name="scalability"></a>Escalabilidad

Kafka en HDInsight permite la [configuración del almacenamiento y la escalabilidad](/azure/hdinsight/kafka/apache-kafka-scalability) para los clústeres de Kafka. Cosmos DB proporciona un rendimiento rápido predecible y se [escala sin problemas](/azure/cosmos-db/partition-data) a medida que crece su aplicación.
La arquitectura de aprovisionamiento de eventos basada en microservicios de este escenario también facilita el escalado del sistema y la expansión de sus funcionalidades.

Para ver otras consideraciones sobre escalabilidad, consulte la [lista de comprobación de escalabilidad][scalability] que encontrará en Azure Architecture Center.

### <a name="security"></a>Seguridad

El [modelo de seguridad de Cosmos DB](/azure/cosmos-db/secure-access-to-data) autentica a los usuarios y proporciona acceso a sus datos y recursos. Para más información, consulte [Seguridad de base de datos de Cosmos DB](/azure/cosmos-db/database-security).

Para obtener instrucciones generales sobre el diseño de soluciones seguras, consulte la [documentación de seguridad de Azure][security].

### <a name="resiliency"></a>Resistencia

La arquitectura de aprovisionamiento de eventos y las tecnologías asociadas de este ejemplo, hacen que este escenario sea altamente resistente cuando se producen errores. Para obtener instrucciones generales sobre el diseño de soluciones resistentes, consulte [Diseño de aplicaciones resistentes de Azure][resiliency].

## <a name="pricing"></a>Precios

Para examinar el costo de ejecutar este escenario, todos los servicios están preconfigurados en la calculadora de costos. Para ver cómo cambiarían los precios en su escenario concreto, cambie las variables pertinentes para que coincidan con el volumen de datos esperado. En este escenario, los precios de ejemplo solo incluyen Cosmos DB y un clúster de Kafka para procesar los eventos generados desde la fuente de cambios de Cosmos DB. No se incluyen procesadores ni microservicios para sistemas originales ni para sistemas de nivel final, y su costo depende en gran medida de la cantidad y escalabilidad de estos servicios así como de las tecnologías elegidas para implementarlos.

La moneda de Azure Cosmos DB es la unidad de solicitud (RU). Con las unidades de solicitud, no es necesario reservar capacidades de lectura/escritura ni aprovisionar CPU, memoria e IOPS. Azure Cosmos DB admite varias API con distintas operaciones, que van desde lecturas y escrituras sencillas a consultas de grafos complejas. Puesto que no todas las solicitudes son iguales, se les asigna una cantidad regularizada de unidades de solicitud según la cantidad de procesamientos necesario para atender la solicitud. El número de unidades de solicitud que requiere la solución depende del tamaño del elemento de datos y del número de operaciones de lectura y escritura en la base de datos por segundo. Para más información, consulte [Unidades de solicitud en Azure Cosmos DB](/azure/cosmos-db/request-units). Estos precios estimados se basan en una instancia de Cosmos DB que se ejecuta en dos regiones de Azure.

Hemos proporcionado tres ejemplos de perfiles de costo según la cantidad de actividad que se espera:

* [Pequeño][small-pricing]: este ejemplo de plan de tarifa corresponde a 5 unidades de solicitud reservadas con un almacén de datos de 1 TB en Cosmos DB y un pequeño clúster de Kafka (D3 v2).
* [Mediano][medium-pricing]: este ejemplo de plan de tarifa corresponde a 50 unidades de solicitud reservadas con un almacén de datos de 10 TB en Cosmos DB y un clúster mediano de Kafka (D4 v2).
* [Grande][large-pricing]: este ejemplo de plan de tarifa se corresponde con 500 unidades de solicitud reservadas con un almacén de datos de 30 TB en Cosmos DB y un clúster grande de Kafka (D5 v2).

## <a name="related-resources"></a>Recursos relacionados

Este escenario de ejemplo se basa en una versión más amplia de esta arquitectura creada por [Jet.com](https://jet.com) para su canalización global de procesamiento de pedidos. Para más información, consulte el [perfil de cliente técnico de jet.com][source-document] y [la presentación de jet.com en Build 2018][source-presentation].

Otros recursos relacionados son los siguientes:
* _[Designing Data-Intensive Applications](https://dataintensive.net)_ (Diseño de aplicaciones de uso intensivo de datos) por Martin Kleppmann (O'Reilly Media, 2017).
* _[Domain Modeling Made Functional: Tackle Software Complexity with Domain-Driven Design and F#](https://pragprog.com/book/swdddf/domain-modeling-made-functional)_ (Modelo de dominio funcional: Solución de la complejidad del software con un diseño basado en dominios y F#) por Scott Wlaschin (Pragmatic Programmers LLC, 2018).
* Otros [casos de uso de Cosmos DB][docs-cosmos-db-use-cases]
* [Arquitectura de procesamiento en tiempo real](/azure/architecture/data-guide/big-data/real-time-processing) en la [Guía de arquitectura de datos de Azure](/azure/architecture/data-guide).

<!-- links -->
[architecture]: ./media/architecture-ecommerce-order-processing.png
[product-category]: https://azure.microsoft.com/product-categories/databases/
[source-document]: https://customers.microsoft.com/story/jet-com-powers-innovative-e-commerce-engine-on-azure-in-less-than-12-months
[source-presentation]: https://channel9.msdn.com/events/Build/2018/BRK3602
[small-pricing]: https://azure.com/e/3d43949ffbb945a88cc0a126dc3a0e6e
[medium-pricing]: https://azure.com/e/1f1e7bf2a6ad4f7799581211f4369b9b
[large-pricing]: https://azure.com/e/75207172ece94cf6b5fb354a2252b333
[docs-cosmos-db-change-feed]: /azure/cosmos-db/change-feed
[docs-cosmos-db-regional-failover]: /azure/cosmos-db/regional-failover
[docs-cosmos-db-guarantees]: /azure/cosmos-db/distribute-data-globally#AvailabilityGuarantees
[docs-cosmos-db-use-cases]: /azure/cosmos-db/use-cases
[docs-kafka-high-availability]: /azure/hdinsight/kafka/apache-kafka-high-availability
[docs-event-hubs]: /azure/event-hubs/event-hubs-what-is-event-hubs
[docs-stream-analytics]: /azure/stream-analytics/stream-analytics-introduction
[availability]: /azure/architecture/checklist/availability
[scalability]: /azure/architecture/checklist/scalability
[resiliency]: /azure/architecture/patterns/category/resiliency/
[security]: /azure/security/
