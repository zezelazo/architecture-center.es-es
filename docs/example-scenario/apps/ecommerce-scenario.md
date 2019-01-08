---
title: Front-end de comercio electrónico
titleSuffix: Azure Example Scenarios
description: Hospede un sitio de comercio electrónico en Azure.
author: masonch
ms.date: 7/13/18
ms.custom: fasttrack
ms.openlocfilehash: d6587218813fa450b284f3a300c7254a3c9fe41f
ms.sourcegitcommit: bb7fcffbb41e2c26a26f8781df32825eb60df70c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/20/2018
ms.locfileid: "53643958"
---
# <a name="an-e-commerce-front-end-on-azure"></a>Front-end de comercio electrónico en Azure

Este escenario de ejemplo le guiará por la implementación de un front-end de comercio electrónico mediante herramientas de plataforma como servicio (PaaS) de Azure. Muchos sitios web de comercio electrónico se enfrentan a la estacionalidad y la variabilidad del tráfico a lo largo del tiempo. Cuando la demanda de sus productos o servicios despegue, de forma predecible o impredecible, el uso de herramientas de PaaS le permitirá manejar más clientes y más transacciones automáticamente. Además, en este escenario puede aprovechar la economía de la nube pagando solo por la capacidad que usa.

Este documento le ayudará a obtener información sobre los distintos componentes y consideraciones de PaaS de Azure que se utilizan a la hora de implementar una aplicación de comercio electrónico de ejemplo llamada *Relecloud Concerts*, una plataforma de venta de entradas para conciertos en línea.

## <a name="relevant-use-cases"></a>Casos de uso pertinentes

Otros casos de uso pertinentes incluyen:

- Compilación de una aplicación que necesita escalado elástico para controlar ráfagas de clientes a diferentes horas.
- Compilación de una aplicación diseñada para funcionar con alta disponibilidad en diferentes regiones de Azure de todo el mundo.

## <a name="architecture"></a>Arquitectura

![Arquitectura del escenario de ejemplo para una aplicación de comercio electrónico][architecture]

En este escenario se incluye la compra de entradas desde un sitio de comercio electrónico y los datos fluyen a través del escenario de la siguiente manera:

1. Azure Traffic Manager enruta una solicitud de usuario al sitio de comercio electrónico hospedado en Azure App Service.
2. Azure CDN proporciona imágenes estáticas y contenido para el usuario.
3. El usuario inicia sesión en la aplicación a través de un inquilino de Azure Active Directory B2C.
4. El usuario busca conciertos mediante Azure Search.
5. El sitio web extrae los detalles del concierto de Azure SQL Database.
6. El sitio web hace referencia a imágenes de entradas adquiridas de Blob Storage.
7. Los resultados de la consulta de la base de datos se almacenan en caché en Azure Redis Cache para mejorar el rendimiento.
8. El usuario envía pedidos de entradas y reseñas de conciertos que se colocan en la cola.
9. Azure Functions procesa el pago del pedido y las reseñas de conciertos.
10. Cognitive Services proporciona un análisis de la reseña del concierto para determinar la opinión (positiva o negativa).
11. Application Insights proporciona métricas de rendimiento para supervisar el estado de la aplicación web.

### <a name="components"></a>Componentes

- [Azure CDN][docs-cdn] ofrece contenido estático almacenado en caché desde ubicaciones cercanas a los usuarios para reducir la latencia.
- [Azure Traffic Manager][docs-traffic-manager] controla la distribución del tráfico de usuario en los puntos de conexión de servicio de las diferentes regiones de Azure.
- [App Services: Web Apps][docs-webapps] hospeda aplicaciones web que permiten la escalabilidad automática y la alta disponibilidad sin tener que administrar la infraestructura.
- [Azure Active Directory B2C][docs-b2c] es un servicio de administración de identidades que le permite personalizar y controlar la manera en que los clientes se registran, inician sesión y administran sus perfiles de una aplicación.
- [Las colas de Storage][docs-storage-queues] almacenan grandes cantidades de mensajes en cola a los que una aplicación puede acceder.
- Las [funciones][docs-functions] son opciones de proceso sin servidor que permiten ejecutar aplicaciones a petición sin tener que administrar la infraestructura.
- [Cognitive Services: Análisis de sentimiento][docs-sentiment-analysis] usa las API de aprendizaje automático y permite a los desarrolladores agregar fácilmente características inteligentes (como detección de vídeo y emociones; reconocimiento facial, de voz y de visión; y comprensión de voz y del lenguaje) en sus aplicaciones.
- [Azure Search][docs-search] es una solución de búsqueda como servicio en la nube que ofrece una experiencia de búsqueda de datos enriquecida en un contenido privado y heterogéneo en las aplicaciones web, para dispositivos móviles y empresariales.
- [Blob Storage][docs-storage-blobs] está optimizado para el almacenamiento de grandes cantidades de datos no estructurados, como texto o datos binarios.
- [Redis Cache][docs-redis-cache] mejora el rendimiento y la escalabilidad de los sistemas que dependen en gran medida de almacenes de datos de back-end mediante la copia temporal de los datos a los que se accede frecuentemente en un almacenamiento rápido ubicado cerca de la aplicación.
- [SQL Database][docs-sql-database] es un servicio administrado de base de datos relacional de uso general de Microsoft Azure que admite estructuras como datos relacionales, JSON, espacial y XML.
- [Application Insights][docs-application-insights] está diseñado para ayudarle a mejorar continuamente el rendimiento y facilidad de uso detectando automáticamente las anomalías de rendimiento a través de herramientas de análisis integradas que le ayudarán a comprender qué hacen los usuarios con una aplicación.

### <a name="alternatives"></a>Alternativas

Hay otras muchas tecnologías disponibles para la compilación de una aplicación orientada al cliente y centrada en el comercio electrónico a escala. Estas incluyen el front-end de la aplicación así como la capa de datos.

Otras opciones para el nivel y las funciones web incluyen:

- [Service Fabric][docs-service-fabric]: una plataforma centrada en torno a componentes distribuidos que se benefician de su implementación y ejecución en un clúster con un alto grado de control. Service Fabric también se puede utilizar para hospedar contenedores.
- [Azure Kubernetes Service][docs-kubernetes-service]: una plataforma para compilar e implementar soluciones basadas en contenedores que se pueden usar como una implementación de una arquitectura de microservicios. Esto permite la agilidad de los distintos componentes de la aplicación para poder escalarse de forma independiente a petición.
- [Azure Container Instances][docs-container-instances]: una manera de implementar y ejecutar rápidamente contenedores con un ciclo de vida corto. En este caso, los contenedores se implementan para ejecutar un trabajo de procesamiento rápido como, por ejemplo, el procesamiento de un mensaje o la realización de un cálculo y, posteriormente, se desaprovisionan tan pronto como la tarea ha finalizado.
- [Service Bus] [service bus] se podría usar en lugar de Queue Storage.

Otras opciones de la capa de datos incluyen:

- [Cosmos DB](/azure/cosmos-db/introduction): Base de datos multimodelo de distribución global de Microsoft. Este servicio proporciona una plataforma para ejecutar otros modelos de datos, como Mongo DB, Cassandra, Graph o almacenamiento de tablas simple.

## <a name="considerations"></a>Consideraciones

### <a name="availability"></a>Disponibilidad

- Considere la posibilidad de aprovechar los [patrones de diseño típicos de disponibilidad][design-patterns-availability] al compilar la aplicación en la nube.
- Revise las consideraciones sobre disponibilidad en la correspondiente [arquitectura de referencia de aplicación web de App Service][app-service-reference-architecture]
- Para ver otras consideraciones sobre disponibilidad, consulte la [lista de comprobación de disponibilidad][availability] en el Centro de arquitectura de Azure.

### <a name="scalability"></a>Escalabilidad

- Al compilar una aplicación en la nube debe tener en cuenta los [patrones de diseño típicos de escalabilidad][design-patterns-scalability].
- Revise las consideraciones sobre escalabilidad en la correspondiente [arquitectura de referencia de aplicación web de App Service][app-service-reference-architecture]
- Para ver otros temas sobre escalabilidad, consulte la [lista de comprobación de escalabilidad][scalability] que encontrará en el Centro de arquitectura de Azure.

### <a name="security"></a>Seguridad

- Considere la posibilidad de aprovechar los [patrones de diseño típicos de seguridad][design-patterns-security] donde corresponda.
- Revise las consideraciones sobre seguridad en la correspondiente [arquitectura de referencia de aplicación web de App Service][app-service-reference-architecture].
- Considere la posibilidad de seguir un proceso de [ciclo de vida de desarrollo seguro][secure-development] para ayudar a los desarrolladores a compilar software más seguro y a afrontar los requisitos de cumplimiento de seguridad al tiempo que se reduce el costo del desarrollo.
- Revise la arquitectura del plano técnico para más información sobre el [cumplimiento de Azure PCI DSS][pci-dss-blueprint].

### <a name="resiliency"></a>Resistencia

- Piense en la posibilidad de aprovechar un [patrón de interruptores][circuit-breaker] para proporcionar un control de errores sencillo en caso de que una parte de la aplicación no esté disponible.
- Revise los [patrones de diseño típicos de resistencia][design-patterns-resiliency] y considere la posibilidad de implementar estos cuando corresponda.
- Encontrará varios [procedimientos recomendados para App Service][resiliency-app-service] en el Centro de arquitectura de Azure.
- Considere la posibilidad de usar una [replicación geográfica][sql-geo-replication] activa para la capa de datos y un almacenamiento con [redundancia geográfica][storage-geo-redudancy] para imágenes y colas.
- Para un análisis más en profundidad sobre [resistencia][resiliency], consulte el artículo correspondiente en el Centro de arquitectura de Azure.

## <a name="deploy-the-scenario"></a>Implementación del escenario

Para implementar este escenario, puede seguir este [tutorial detallado][end-to-end-walkthrough] que muestra cómo implementar manualmente cada componente. Este tutorial también proporciona una aplicación de ejemplo de .NET que ejecuta una aplicación simple de compra de entradas. Además, hay una plantilla de Resource Manager para automatizar la implementación de la mayoría de los recursos de Azure.

## <a name="pricing"></a>Precios

Para explorar el costo de ejecutar este escenario, todos los servicios están preconfigurados en la calculadora de costos. Para ver cómo cambiarían los precios en su caso concreto, cambie las variables pertinentes para que coincidan con el tráfico esperado.

Hemos proporcionado tres ejemplos de perfiles de costo según la cantidad de tráfico que se espera obtener:

- [Pequeño][small-pricing]: Este ejemplo de precios representa los componentes necesarios para crear una instancia con un nivel de producción mínimo. En este perfil damos por hecho que hay una pequeña cantidad de usuarios, de solo unos miles al mes. La aplicación usa una sola instancia de una aplicación web estándar que será suficiente para habilitar el escalado automático. Cada uno de los demás componentes se escala a un nivel básico que permitirá un costo mínimo pero garantizará un soporte según el Acuerdo de Nivel de Servicio y suficiente capacidad para controlar una carga de trabajo de nivel de producción.
- [Mediano][medium-pricing]: Este ejemplo de precios representa los componentes necesarios para una implementación de tamaño moderado. En este caso, se calculan aproximadamente 100 000 usuarios utilizando el sistema en el transcurso de un mes. El tráfico esperado se controla con una instancia de App Service simple con un nivel estándar moderado. También se agregan a la calculadora los niveles moderados de Cognitive Services y Search Service.
- [Grande][large-pricing]: Este ejemplo de precios representa una aplicación diseñada para operaciones a gran escala, con millones de usuarios al mes y movimientos de terabytes de datos. En este nivel de uso, se requieren aplicaciones web de alto rendimiento y de nivel premium implementadas en varias regiones y dirigidas por un administrador de tráfico. Los datos se conservan en diversas opciones: almacenamiento, bases de datos y CDN, y están configuradas para alcanzar terabytes de tamaño.

## <a name="related-resources"></a>Recursos relacionados

- [Arquitectura de referencia de aplicaciones web para varias regiones][multi-region-web-app]
- [eShop en el ejemplo de referencia de contenedores][microservices-ecommerce]

<!-- links -->
[architecture]: ./media/architecture-ecommerce-scenario.png
[small-pricing]: https://azure.com/e/90fbb6a661a04888a57322985f9b34ac
[medium-pricing]: https://azure.com/e/38d5d387e3234537b6859660db1c9973
[large-pricing]: https://azure.com/e/f07f99b6c3134803a14c9b43fcba3e2f
[app-service-reference-architecture]: ../../reference-architectures/app-service-web-app/basic-web-app.md
[availability]: /azure/architecture/checklist/availability
[circuit-breaker]: /azure/architecture/patterns/circuit-breaker
[design-patterns-availability]: /azure/architecture/patterns/category/availability
[design-patterns-resiliency]: /azure/architecture/patterns/category/resiliency
[design-patterns-scalability]: /azure/architecture/patterns/category/performance-scalability
[design-patterns-security]: /azure/architecture/patterns/category/security
[docs-application-insights]: /azure/application-insights/app-insights-overview
[docs-b2c]: /azure/active-directory-b2c/active-directory-b2c-overview
[docs-cdn]: /azure/cdn/cdn-overview
[docs-container-instances]: /azure/container-instances/
[docs-kubernetes-service]: /azure/aks/
[docs-functions]: /azure/azure-functions/functions-overview
[docs-redis-cache]: /azure/redis-cache/cache-overview
[docs-search]: /azure/search/search-what-is-azure-search
[docs-service-fabric]: /azure/service-fabric/
[docs-sentiment-analysis]: /azure/cognitive-services/welcome
[docs-sql-database]: /azure/sql-database/sql-database-technical-overview
[docs-storage-blobs]: /azure/storage/blobs/storage-blobs-introduction
[docs-storage-queues]: /azure/storage/queues/storage-queues-introduction
[docs-traffic-manager]: /azure/traffic-manager/traffic-manager-overview
[docs-webapps]: /azure/app-service/app-service-web-overview
[end-to-end-walkthrough]: https://github.com/Azure/fta-customerfacingapps/tree/master/ecommerce/articles
[microservices-ecommerce]: https://github.com/dotnet-architecture/eShopOnContainers
[multi-region-web-app]: /azure/architecture/reference-architectures/app-service-web-app/multi-region
[pci-dss-blueprint]: /azure/security/blueprints/payment-processing-blueprint
[resiliency-app-service]: /azure/architecture/checklist/resiliency-per-service#app-service
[resiliency]: /azure/architecture/checklist/resiliency
[scalability]: /azure/architecture/checklist/scalability
[secure-development]: https://www.microsoft.com/SDL/process/design.aspx
[sql-geo-replication]: /azure/sql-database/sql-database-geo-replication-overview
[storage-geo-redudancy]: /azure/storage/common/storage-redundancy-grs
