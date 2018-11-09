---
title: Aplicación web escalable
description: Mejora de la escalabilidad en una aplicación web que se ejecuta en Microsoft Azure
author: MikeWasson
pnp.series.title: Azure App Service
pnp.series.prev: basic-web-app
pnp.series.next: multi-region-web-app
ms.date: 10/25/2018
cardTitle: Improve scalability
ms.openlocfilehash: 208413a49fe4a3f9ca308fa1a939ba426e7fa636
ms.sourcegitcommit: 065fa8ecb37c8be1827da861243ad6a33c75c99d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/26/2018
ms.locfileid: "50136699"
---
# <a name="improve-scalability-in-a-web-application"></a>Mejora de la escalabilidad en una aplicación web

Esta arquitectura de referencia muestra procedimientos de demostrada eficacia para mejorar la escalabilidad y el rendimiento en una aplicación web de Azure App Service.

![[0]][0]

*Descargue un [archivo Visio][visio-download] de esta arquitectura.*

## <a name="architecture"></a>Arquitectura

Esta arquitectura se basa en la que se muestra en [Aplicación web básica][basic-web-app]. Incluye los siguientes componentes:

* **Grupo de recursos**. Un [grupo de recursos][resource-group] es un contenedor lógico de recursos de Azure.
* **[Aplicación web][app-service-web-app]**. Una aplicación moderna típica podría incluir un sitio web y una o varias web API web de RESTful. Los clientes del explorador podrían consumir una API web mediante AJAX, y también las aplicaciones nativas o las aplicaciones del lado servidor podrían consumirla. Para conocer las consideraciones sobre el diseño de las API web, consulte la [guía de diseño de API][api-guidance].
* **Aplicación de función**. Use [Function Apps][functions] para ejecutar tareas en segundo plano. Las funciones se invocan mediante un desencadenador como, por ejemplo, un evento del temporizador o un mensaje que se coloca en cola. Para las tareas con estado de ejecución prolongada, use [Durable Functions][durable-functions].
* **Cola**. En la arquitectura que se muestra aquí, la aplicación pone en cola las tareas en segundo plano mediante la colocación de un mensaje en una cola de [Azure Queue Storage][queue-storage]. El mensaje desencadena una función de aplicación. Como alternativa, puede usar colas de Service Bus. Para ver una comparación, consulte [Colas de Storage y de Service Bus: comparación y diferencias][queues-compared].
* **Caché**. Almacene datos parcialmente estáticos en [Azure Redis Cache][azure-redis].  
* <strong>CDN</strong>. Use [Azure Content Delivery Network][ azure-cdn] (CDN) para almacenar en caché el contenido disponible públicamente y así reducir latencia y acelerar la entrega de contenido.
* **Almacenamiento de datos**. Use [Azure SQL Database][sql-db] con datos relacionales. Para datos no relacionales, considere la posibilidad de usar [Cosmos DB][cosmosdb].
* **Azure Search**. Use [Azure Search][azure-search] para agregar funcionalidad de búsqueda, como sugerencias de búsqueda, búsqueda aproximada y búsqueda específica del idioma. Azure Search se usa normalmente en combinación con otro almacén de datos, en especial si el almacén de datos principal requiere una coherencia estricta. En este enfoque, almacene los datos acreditado en el otro almacén de datos y el índice de búsqueda en Azure Search. Azure Search también se puede usar para consolidar un índice de búsqueda sencillo desde varios almacenes de datos.  
* **Azure DNS**. [Azure DNS][azure-dns] es un servicio de hospedaje para dominios DNS que permite resolver nombres mediante la infraestructura de Microsoft Azure. Al hospedar dominios en Azure, puede administrar los registros DNS con las mismas credenciales, API, herramientas y facturación que con los demás servicios de Azure.
* **Application Gateway**. [Application Gateway](/azure/application-gateway/) es un equilibrador de carga de nivel 7. En esta arquitectura, enruta las solicitudes HTTP al front-end web. Application Gateway proporciona también un [firewall de aplicaciones web](/azure/application-gateway/waf-overview) (WAF) que protege la aplicación contra puntos vulnerables de la seguridad comunes. 

## <a name="recommendations"></a>Recomendaciones

Los requisitos pueden diferir de los de la arquitectura que se describe aquí. Use las recomendaciones de esta sección como punto de partida.

### <a name="app-service-apps"></a>Aplicaciones de App Service
Se recomienda crear la aplicación web y la API web como aplicaciones de App Service independientes. Este diseño permite ejecutarlas en planes de App Service diferentes, por lo que se pueden escalar de forma independiente. Si inicialmente no necesita ese nivel de escalabilidad, puede implementar las aplicaciones en el mismo plan y moverlas más tarde a planes diferentes si es necesario.

> [!NOTE]
> Los planes Básico, Estándar y Premium se facturan por las instancias de máquina virtual, no por aplicación. Consulte [Precios de App Service][app-service-pricing].
> 
> 

### <a name="cache"></a>Memoria caché
Puede mejorar el rendimiento y la escalabilidad mediante [Azure Redis Cache][azure-redis] para almacenar en caché algunos datos. Considere el uso de Redis Cache para:

* Datos de transacción parcialmente estáticos
* Estado de sesión
* Salida HTML Esto puede ser útil en aplicaciones que representan la salida HTML compleja.

Para instrucciones detalladas sobre cómo diseñar una estrategia de almacenamiento en caché, consulte la [guía de almacenamiento en caché][caching-guidance].

### <a name="cdn"></a>CDN
Use [Azure CDN][azure-cdn] para almacenar en caché el contenido estático. La principal ventaja de una red CDN es reducir la latencia de los usuarios, ya que el contenido se almacena en caché en un servidor perimetral que está geográficamente próximo al usuario. CDN también puede reducir la carga sobre la aplicación, ya que el la aplicación no administra el tráfico.

Si la aplicación está compuesta mayormente por páginas estáticas, considere la posibilidad de usar [CDN para almacenar en caché la aplicación entera][cdn-app-service]. Si no es así, coloque el contenido estático, como imágenes, CSS y archivos HTML, en [Azure Storage y use CDN para almacenar en caché esos archivos][cdn-storage-account].

> [!NOTE]
> Azure CDN no puede servir contenido que requiera autenticación.
> 
> 

Para obtener instrucciones más detalladas, consulte [guía de la red de entrega de contenido (CDN)][cdn-guidance].

### <a name="storage"></a>Storage
Las aplicaciones modernas suelen procesan grandes cantidades de datos. Para escalarlos a la nube, es importante elegir el tipo de almacenamiento correcto. Estas son algunas recomendaciones básicas. 

| Qué desea almacenar | Ejemplo | Almacenamiento recomendado |
| --- | --- | --- |
| Archivos |Imágenes, documentos, archivos PDF |Azure Blob Storage |
| Pares clave-valor |Datos de perfil de usuario consultados por identificador de usuario |Almacenamiento de tablas de Azure |
| Mensajes cortos diseñados para desencadenar procesamiento adicional |Solicitudes de pedido |Azure Queue Storage, cola de Service Bus o tema de Service Bus |
| Datos no relacionales con un esquema flexible que requiere consulta básica |Catálogo de productos |Base de datos de documentos, como Azure Cosmos DB, MongoDB o Apache CouchDB |
| Datos relacionales que requieren compatibilidad más completa con consultas, un esquemas estricto o fuerte coherencia |Inventario de productos |Azure SQL Database |

 Consulte [Elección del almacén de datos apropiado][datastore].

## <a name="scalability-considerations"></a>Consideraciones sobre escalabilidad

Una de las ventajas principales de Azure App Service es la posibilidad de escalar la aplicación en función de la carga. Estas son algunas consideraciones que se deben tener en cuenta al planear el escalado de la aplicación.

### <a name="app-service-app"></a>Aplicación de App Service
Si la solución incluye varias aplicaciones de App Service, podría implementarlas en planes de App Service diferentes. Este enfoque permite escalarlas por separado porque se ejecutan en instancias independientes. 

De igual forma, considere la posibilidad de colocar una aplicación de función en su propio plan de modo que las tareas en segundo plano no se ejecuten en las mismas instancias que administran las solicitudes HTTP. Si las tareas en segundo plano se ejecutan de forma intermitente, considere la posibilidad de usar un [plan de consumo][functions-consumption-plan], que se facture según el número de ejecuciones en lugar de por horas. 

### <a name="sql-database"></a>SQL Database
Aumente la escalabilidad de una base de datos SQL mediante el *particionamiento* de la base de datos. El particionamiento hace referencia a la creación de particiones de la base de datos de manera horizontal. El particionamiento permite escalar la base de datos horizontalmente mediante [herramientas Elastic Database][sql-elastic]. Entre las posibles ventajas del particionamiento se incluyen:

- Mejor rendimiento de las transacciones.
- Las consultas pueden ejecutarse con mayor rapidez sobre un subconjunto de los datos.

### <a name="azure-search"></a>Azure Search
Azure Search quita la sobrecarga que supone realizar búsquedas de datos complejos desde el almacén de datos principal, y puede escalarse para administrar la carga. Consulte [Escalado de niveles de recursos para cargas de trabajo de indexación y consulta en Azure Search][azure-search-scaling].

## <a name="security-considerations"></a>Consideraciones sobre la seguridad
En esta sección se enumeran las consideraciones de seguridad que son específicas de los servicios de Azure descritos en este artículo. No es una lista completa de procedimientos de seguridad recomendados. Si desea conocer algunas otras consideraciones de seguridad, consulte [Protección de una aplicación en Azure App Service][app-service-security].

### <a name="cross-origin-resource-sharing-cors"></a>Uso compartido de recursos entre orígenes
Si crea un sitio web y una API web como aplicaciones independientes, el sitio web no podrá realizar llamadas AJAX a la API en el lado cliente a menos que habilite CORS.

> [!NOTE]
> La seguridad del explorador impide que una página web realice solicitudes AJAX a otro dominio. Esta restricción se conoce como directiva de mismo origen y evita que un sitio malintencionado lea datos confidenciales de otro sitio. CORS es una norma de W3C que permite que un servidor "se relaje" con la directiva del mismo origen de forma que permite algunas solicitudes entre orígenes y rechaza otras.
> 
> 

App Services tiene compatibilidad integrada con CORS, sin necesidad de escribir ningún código de aplicación. Consulte [Consume an API app from JavaScript using CORS][cors] (Consumo de una aplicación de API desde JavaScript con CORS). Agregue el sitio web a la lista de orígenes permitidos para la API.

### <a name="sql-database-encryption"></a>Cifrado de SQL Database
Use [Cifrado de datos transparente][sql-encryption] si necesita cifrar los datos en reposo en la base de datos. Esta característica realiza el cifrado y el descifrado en tiempo real de una base de datos completa (incluidas las copias de seguridad y los archivos de registros de transacciones) y no requiere realizar ningún cambio en la aplicación. El cifrado agrega alguna latencia, así que es conveniente separar los datos que deben estar protegidos en su propia base de datos y habilitar el cifrado únicamente para esa base de datos.  
  

<!-- links -->

[api-guidance]: ../../best-practices/api-design.md
[app-service-security]: /azure/app-service-web/web-sites-security
[app-service-web-app]: /azure/app-service-web/app-service-web-overview
[app-service-api-app]: /azure/app-service-api/app-service-api-apps-why-best-platform
[app-service-pricing]: https://azure.microsoft.com/pricing/details/app-service/
[azure-cdn]: https://azure.microsoft.com/services/cdn/
[azure-dns]: /azure/dns/dns-overview
[azure-redis]: https://azure.microsoft.com/services/cache/
[azure-search]: https://azure.microsoft.com/documentation/services/search/
[azure-search-scaling]: /azure/search/search-capacity-planning
[basic-web-app]: basic-web-app.md
[basic-web-app-scalability]: basic-web-app.md#scalability-considerations
[caching-guidance]: ../../best-practices/caching.md
[cdn-app-service]: /azure/app-service-web/cdn-websites-with-cdn
[cdn-storage-account]: /azure/cdn/cdn-create-a-storage-account-with-cdn
[cdn-guidance]: ../../best-practices/cdn.md
[cors]: /azure/app-service-api/app-service-api-cors-consume-javascript
[cosmosdb]: /azure/cosmos-db/
[datastore]: ../..//guide/technology-choices/data-store-overview.md
[durable-functions]: /azure/azure-functions/durable-functions-overview
[functions]: /azure/azure-functions/functions-overview
[functions-consumption-plan]: /azure/azure-functions/functions-scale#consumption-plan
[queue-storage]: /azure/storage/storage-dotnet-how-to-use-queues
[queues-compared]: /azure/service-bus-messaging/service-bus-azure-and-service-bus-queues-compared-contrasted
[resource-group]: /azure/azure-resource-manager/resource-group-overview#resource-groups
[sql-db]: https://azure.microsoft.com/documentation/services/sql-database/
[sql-elastic]: /azure/sql-database/sql-database-elastic-scale-introduction
[sql-encryption]: https://msdn.microsoft.com/library/dn948096.aspx
[tm]: https://azure.microsoft.com/services/traffic-manager/
[visio-download]: https://archcenter.blob.core.windows.net/cdn/app-service-reference-architectures.vsdx
[web-app-multi-region]: ./multi-region.md
[0]: ./images/scalable-web-app.png "Aplicación web en Azure con escalabilidad mejorada"
