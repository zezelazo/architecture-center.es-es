---
title: Motor de búsqueda inteligente de productos para comercio electrónico
description: Proporcione una experiencia de búsqueda de alcance mundial en una aplicación de comercio electrónico.
author: jelledruyts
ms.date: 09/14/2018
ms.custom: fasttrack
ms.openlocfilehash: 5eabdb94b9345e73b21526681e0dbd6ae859d7be
ms.sourcegitcommit: a0e8d11543751d681953717f6e78173e597ae207
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/06/2018
ms.locfileid: "53004892"
---
# <a name="intelligent-product-search-engine-for-e-commerce"></a>Motor de búsqueda inteligente de productos para comercio electrónico

Este escenario de ejemplo muestra cómo el uso de un servicio de búsqueda dedicada puede aumentar considerablemente el nivel de pertinencia en los resultados de búsqueda para los clientes de comercio electrónico.

La búsqueda es el mecanismo principal que los clientes usan para encontrar y, en último término, comprar productos, y por ello es esencial que los resultados de las búsquedas sean pertinentes en relación a la _intención_ de la consulta realizada mediante la búsqueda, y que la experiencia de búsqueda de principio a fin coincida con la que ofrecen los gigantes de la búsqueda, proporcionando resultados casi instantáneos, análisis lingüístico, coincidencias de ubicación geográfica, filtrado, búsqueda por facetas, autocompletado, resaltado de referencias, etc.

Imagine una aplicación web de comercio electrónico típica con datos de producto almacenados en una base de datos relacional como SQL Server o Azure SQL Database. Generalmente las consultas de búsqueda se controlan dentro de la base de datos usando las características consultas `LIKE` o [búsqueda de texto completo][docs-sql-fts]. Al usar [Azure Search][docs-search] en su lugar, se libera la base de datos operativa del procesamiento de consultas, y se pueden empezar a aprovechar esas características de difícil implementación que proporcionan a los clientes la mejor experiencia de búsqueda posible. Además, dado que Azure Search es un componente de plataforma como servicio (PaaS), no tiene que preocuparse por administrar infraestructura o convertirse en una experto de búsqueda.

## <a name="relevant-use-cases"></a>Casos de uso pertinentes

Otros casos de uso pertinentes incluyen:

* Búsqueda de anuncios inmobiliarias o tiendas cerca de la ubicación física del usuario.
* Búsqueda de artículos en un sitio de noticias o búsqueda de resultados deportivos, con una preferencia más alta por la información más _reciente_.
* Búsqueda en repositorios de gran tamaño de organizaciones _centradas en documentos_ tales como organismos legisladores y notarías.

En última instancia _todas_ las aplicaciones que tenga algún tipo de funcionalidad de búsqueda pueden beneficiarse de un servicio de búsqueda dedicado.

## <a name="architecture"></a>Arquitectura

![Introducción a la arquitectura de los componentes de Azure implicados en un motor de búsqueda inteligente de productos para el comercio electrónico][architecture]

Este escenario incluye una solución de comercio electrónico donde los clientes pueden buscar a través de un catálogo de productos.
1. Los clientes van a la **aplicación web de comercio electrónico** desde cualquier dispositivo.
2. El catálogo de productos se mantiene en una instancia de **Azure SQL Database** para el procesamiento de transacciones.
3. Azure Search usa un **indexador de búsqueda** para mantener automáticamente actualizado su índice de búsqueda mediante el seguimiento de cambios integrado.
4. Las consultas de búsqueda del cliente se descargan en el servicio **Azure Search**, que procesa la consulta y devuelve los resultados más pertinentes.
5. Como alternativa a una experiencia de búsqueda basada en web, los clientes también pueden usar un **bot de conversación** en medios sociales o directamente desde asistentes digitales para buscar productos y refinar de manera incremental su consulta de búsqueda y resultados.
6. Opcionalmente, la característica de **búsqueda cognitiva** puede usarse para aplicar inteligencia artificial para un procesamiento todavía más inteligente.

### <a name="components"></a>Componentes

* [App Services: Web Apps][docs-webapps] hospeda aplicaciones web que permiten la escalabilidad automática y la alta disponibilidad sin tener que administrar la infraestructura.
* [SQL Database][docs-sql-database] es un servicio administrado de base de datos relacional de uso general de Microsoft Azure que admite estructuras como datos relacionales, JSON, espacial y XML.
* [Azure Search][docs-search] es una solución de búsqueda como servicio en la nube que ofrece una experiencia de búsqueda de datos enriquecida en un contenido privado y heterogéneo en las aplicaciones web, para dispositivos móviles y empresariales.
* [Bot Service][docs-botservice] proporciona herramientas para crear, probar, implementar y administrar bots inteligentes.
* [Cognitive Services][docs-cognitive] permite usar algoritmos inteligentes para ver, oír, hablar, comprender e interpretar las necesidades de los usuarios con formas de comunicación naturales.

### <a name="alternatives"></a>Alternativas

* Puede usar las funcionalidades de **búsqueda en base de datos**, por ejemplo, mediante la búsqueda de texto completo de SQL Server, pero el almacén transaccional también procesa consultas (lo que aumenta la necesidad de capacidad de procesamiento) y las capacidades de búsqueda dentro de la base de datos son más limitadas.
* Puede hospedar el código abierto [Apache Lucene][apache-lucene] (en el que se basa Azure Search) en Azure Virtual Machines, pero de esta forma, vuelve a la administración de una infraestructura como-servicio (IaaS) y no se beneficia de las muchas características que Azure Search proporciona por encima de Lucene.
* También puede implementar [Elasticsearch][elastic-marketplace] desde Azure Marketplace, se trata de un producto de búsqueda alternativo y capaz que proporciona otro proveedor, pero en este caso también se ejecuta como una carga de trabajo de IaaS.

Otras opciones de la capa de datos incluyen:

* [Cosmos DB](/azure/cosmos-db/introduction): es la base de datos multimodelo de distribución global de Microsoft. Cosmos DB proporciona una plataforma para ejecutar otros modelos de datos, como Mongo DB, Cassandra, datos de grafos o almacenamiento de tablas simple. Azure Search también admite la indexación de los datos directamente de Cosmos DB.

## <a name="considerations"></a>Consideraciones

### <a name="scalability"></a>Escalabilidad

El [plan de tarifa][search-tier] del servicio Azure Search no determina las características disponibles, pero se utiliza principalmente para el [planeamiento de capacidad][search-capacity], ya que define el almacenamiento máximo que obtiene y el número de particiones y réplicas que puede aprovisionar. Las **particiones** permiten indexar más documentos y obtener un mayor rendimiento de escritura, mientras que las **réplicas** proporcionar más consultas por segundo (QPS) y alta disponibilidad.

Puede cambiar dinámicamente el número de particiones y réplicas pero no puede cambiar el plan de tarifa, por lo que debe considerar detenidamente el plan correcto para la carga de trabajo de destino. Si de todos modos necesita cambiar el plan, tendrá que aprovisionar un nuevo servicio en paralelo y volver a cargar en él los índices, momento que puede aprovechar para señalar a sus aplicaciones el nuevo servicio.

### <a name="availability"></a>Disponibilidad

Azure Search proporciona un [contrato de nivel de servicio con disponibilidad del 99,9 %][search-sla] para _lecturas_ (es decir, para consultar) si tiene al menos dos réplicas y para _actualizaciones_(es decir, actualizando los índices de búsqueda) si tiene al menos tres réplicas. Por lo tanto, debe aprovisionar al menos dos réplicas si desea que los clientes puedan realizar _búsquedas_ de forma confiable, y tres si los _cambios en el índice_ también se consideran operaciones de alta disponibilidad.

Si necesita realizar cambios importantes en el índice sin tiempo de inactividad (por ejemplo, cambiar tipos de datos, eliminar o cambiar el nombre de campos), tiene que volver a generar el índice. Al igual que en el cambio de nivel de servicio, esto significa crear un nuevo índice, volver a rellenarlo con los datos y después, actualizar sus aplicaciones para que apunten al nuevo índice.

### <a name="security"></a>Seguridad

Azure Search es compatible con muchos [estándares de privacidad de datos y seguridad][search-security], lo que hace posible que se pueda usar en la mayoría de las industrias.

Para proteger el acceso al servicio, Azure Search usa dos tipos de claves: **las claves de administración**, que permiten realizar _cualquier_ tarea en el servicio, y las **claves de consulta**, que se pueden utilizar unicamente para operaciones de solo lectura, como las consultas. Normalmente, la aplicación que realiza la búsqueda no actualiza el índice, por lo que debe configurarse solo con una clave de consulta y no con una clave de administración (especialmente si la búsqueda se realiza desde un dispositivo del usuario final como script que se ejecuta en un explorador web).

### <a name="search-relevance"></a>Nivel de pertinencia de la búsqueda

El grado de éxito de la aplicación de comercio electrónico depende en gran medida de lo pertinente que sean los resultados de búsqueda para sus clientes. Ajuste minuciosamente el servicio de búsqueda para conseguir los mejores resultados basados en la investigación del usuario, o confíe en las características integradas, como el [análisis del tráfico de búsqueda][search-analysis] para comprender los patrones de búsqueda de su cliente que le permitan tomar decisiones basadas en datos.

Formas habituales para optimizar el servicio de búsqueda incluyen:

* Uso de [perfiles de puntuación][search-scoring] para influir en la pertinencia de los resultados de búsqueda, por ejemplo, basándose en qué campo se hizo la coincidencia con la consulta, cómo es de reciente la consulta, distancia geográfica al usuario,...
* Uso de [analizadores de idioma proporcionados por Microsoft][search-languages] que utilizan una pila de Procesamiento de lenguaje natural (NLP) avanzada para interpretar mejor las consultas
* Uso de [analizadores personalizados][search-analyzers] para garantizar que sus productos se pueden encontrar correctamente, especialmente para búsquedas de información no basada en el lenguaje, como la marca de un producto y el modelo.

## <a name="deploy-this-scenario"></a>Implementación de este escenario

Para implementar una versión de comercio electrónico más completa de este escenario, puede seguir este [tutorial paso a paso][end-to-end-walkthrough] que proporciona una aplicación de ejemplo de .NET que se ejecuta una aplicación simple para la compra de entradas. También incluye Azure Search y usa muchas de las características mencionadas aquí. Además, hay una plantilla de Resource Manager para automatizar la implementación de la mayoría de los recursos de Azure.

## <a name="pricing"></a>Precios

Para explorar el costo de ejecutar este escenario, todos los servicios que se menciona anteriormente están preconfigurados en la calculadora de costos. Para ver cómo cambiarían los precios en su caso concreto, cambie las variables pertinentes para que coincidan con el uso esperado.

Hemos proporcionado tres ejemplos de perfiles de costo según la cantidad de tráfico que se espera obtener:

* [Pequeño][small-pricing]: En este perfil, usamos una sola aplicación web `Standard S1` para hospedar el sitio web, el nivel gratis de Azure Bot Service, un único servicio `Basic` de Azure Search y una instancia `Standard S2` de SQL Database.
* [Mediano][medium-pricing]: En este caso escalamos la aplicación web a dos instancias del nivel `Standard S3`, actualizando Search Service a un nivel `Standard S1` y con una instancia `Standard S6` de SQL Database.
* [Grande][large-pricing]: En el perfil más grande, usamos cuatro instancias de una aplicación web `Premium P2V2`, actualizamos Azure Bot Service al nivel `Standard S1` (con 1.000.000 de mensajes en canales Premium), usamos 2 unidades del servicio `Standard S3` de Azure Search y una instancia `Premium P6` de SQL Database.

## <a name="related-resources"></a>Recursos relacionados

Para más información sobre Azure Search, visite el [centro de documentación][docs-search], consulte los [ejemplos][search-samples], o vea un [sitio de demostración][search-demo] completo en acción.

<!-- links -->
[architecture]: ./media/architecture-ecommerce-search.png
[docs-sql-fts]: /sql/relational-databases/search/query-with-full-text-search
[docs-search]: /azure/search/search-what-is-azure-search
[docs-sql-database]: /azure/sql-database/sql-database-technical-overview
[docs-webapps]: /azure/app-service/app-service-web-overview
[docs-botservice]: /azure/bot-service/
[docs-cognitive]: /azure/cognitive-services/
[apache-lucene]: https://lucene.apache.org/
[elastic-marketplace]: https://azuremarketplace.microsoft.com/marketplace/apps/elastic.elasticsearch
[end-to-end-walkthrough]: https://github.com/Azure/fta-customerfacingapps/tree/master/ecommerce/articles
[search-sla]: https://go.microsoft.com/fwlink/?LinkId=716855
[search-tier]: /azure/search/search-sku-tier
[search-capacity]: /azure/search/search-capacity-planning
[search-security]: /azure/search/search-security-overview
[search-analysis]: /azure/search/search-traffic-analytics
[search-languages]: /rest/api/searchservice/language-support
[search-analyzers]: /rest/api/searchservice/custom-analyzers-in-azure-search
[search-scoring]: /rest/api/searchservice/add-scoring-profiles-to-a-search-index
[search-samples]: https://azure.microsoft.com/resources/samples/?service=search&sort=0
[search-demo]: https://azjobsdemo.azurewebsites.net/
[small-pricing]: https://azure.com/e/db2672a55b6b4d768ef0060a8d9759bd
[medium-pricing]: https://azure.com/e/a5ad0706c9e74add811e83ef83766a1c
[large-pricing]: https://azure.com/e/57f95a898daa487795bd305599973ee6
