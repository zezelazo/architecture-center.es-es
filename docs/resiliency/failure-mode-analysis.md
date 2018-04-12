---
title: Análisis del modo de error
description: Directrices para llevar a cabo un análisis del modo de error para soluciones en la nube basadas en Azure.
author: MikeWasson
ms.date: 03/24/2017
ms.custom: resiliency
pnp.series.title: Design for Resiliency
ms.openlocfilehash: 8786c411249267e502003a90d5f2ff5e4c786803
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2018
---
# <a name="failure-mode-analysis"></a>Análisis del modo de error
[!INCLUDE [header](../_includes/header.md)]

El análisis del modo de error (FMA) es un proceso de creación de resistencia en un sistema mediante la identificación de posibles puntos de error en el sistema. El FMA debe formar parte de las fases de diseño y arquitectura, de modo que pueda generar una recuperación de errores en el sistema desde el principio.

Este es el proceso general para realizar un FMA:

1. Identifique todos los componentes del sistema. Incluya dependencias externas, como proveedores de identidades, servicios de terceros, etc.   
2. Para cada componente, identifique los posibles errores que se puedan producir. Un único componente puede tener más de un modo de error. Por ejemplo, debería considerar los errores de lectura y escritura por separado porque el impacto y las posibles mitigaciones serán diferentes.
3. Clasifique por orden de prioridad cada modo de error según su riesgo general. Tenga en cuenta estos factores:  

   * Cuál es la probabilidad de que se produzca el error. ¿Es relativamente frecuente? ¿Sumamente raro? No necesita números exactos, ya que la finalidad es realizar una clasificación por orden de prioridad.
   * ¿Cuál es el impacto en la aplicación, en términos de disponibilidad, pérdida de datos, costo económico e interrupción de la actividad comercial?
4. Para cada modo de error, determine cómo responderá y se recuperará la aplicación. Tenga en cuenta los inconvenientes en lo relativo al costo y a la complejidad de la aplicación.   

Como punto de partida para el proceso FMA, este artículo contiene un catálogo de los posibles modos de error y sus mitigaciones. El catálogo está organizado por tecnología o servicio de Azure, además de una categoría general para el diseño en el nivel de aplicación. El catálogo no es exhaustivo pero abarca muchos de los principales servicios de Azure.

## <a name="app-service"></a>App Service
### <a name="app-service-app-shuts-down"></a>La aplicación de App Service se cierra.
**Detección**. Causas posibles:

* Cierre esperado

  * Un operador cierra la aplicación; por ejemplo, mediante Azure Portal.
  * La aplicación se ha descargado porque estaba inactiva. (Solo si el valor `Always On` está deshabilitado).
* Cierre inesperado

  * La aplicación se bloquea.
  * Una instancia de la máquina virtual de App Service deja de estar disponible.

El registro Application_End detectará el cierre del dominio de aplicación (bloqueo del proceso de software) que es la única manera de detectar los cierres del dominio de aplicación.

**Recuperación**

* Si se trata de un cierre esperado, utilice el evento de cierre de la aplicación para cerrarla correctamente. Por ejemplo, en ASP.NET, utilice el método `Application_End`.
* Si la aplicación se ha descargado mientras estaba inactiva, se reiniciará automáticamente en la siguiente solicitud. No obstante, incurrirá en el costo de "arranque en frío".
* Para evitar que la aplicación se descargue mientras está inactiva, habilite el valor `Always On` en la aplicación web. Consulte [Configuración de aplicaciones web en Azure App Service][app-service-configure].
* Para evitar que un operador cierre la aplicación, establezca un bloqueo de recurso con el nivel `ReadOnly`. Consulte [Bloqueo de recursos con Azure Resource Manager][rm-locks].
* Si la aplicación se bloquea o una máquina virtual de App Service deja de estar disponible, App Service reinicia automáticamente la aplicación.

**Diagnóstico**. Registros de aplicaciones y registros de servidor web. Consulte [Habilitación del registro de diagnóstico para aplicaciones web en Azure App Service][app-service-logging].

### <a name="a-particular-user-repeatedly-makes-bad-requests-or-overloads-the-system"></a>Un usuario determinado realiza repetidamente solicitudes incorrectas o sobrecarga el sistema.
**Detección**. Autentique los usuarios e incluya los identificadores de usuario en los registros de aplicaciones.

**Recuperación**

* Use [Azure API Management][api-management] para limitar las solicitudes del usuario. Consulte [Limitación avanzada de solicitudes con Azure API Management][api-management-throttling]
* Bloquee el usuario.

**Diagnóstico**. Registre todas las solicitudes de autenticación.

### <a name="a-bad-update-was-deployed"></a>Se implementó una actualización incorrecta.
**Detección**. Supervise el mantenimiento de la aplicación mediante Azure Portal (consulte [Supervisión del rendimiento de Azure Web App][app-insights-web-apps]) o bien implemente el [patrón de supervisión de puntos de conexión de mantenimiento][health-endpoint-monitoring-pattern].

**Recuperación**. Utilice varias [ranuras de implementación][app-service-slots] y revierta a la última implementación correcta conocida. Para más información, consulte [Basic web application][ra-web-apps-basic] (Aplicación web básica).///

## <a name="azure-active-directory"></a>Azure Active Directory
### <a name="openid-connect-oidc-authentication-fails"></a>Se produce un error en la autenticación de OpenID Connect (OIDC).
**Detección**. Los posibles modos de error son:

1. Azure AD no está disponible o no está accesible debido a un problema de red. Se produce un error en el redireccionamiento al punto de conexión de autenticación y el middleware OIDC genera una excepción.
2. El inquilino de Azure AD no existe. El redireccionamiento al punto de conexión de autenticación devuelve un código de error HTTP y el middleware OIDC genera una excepción.
3. El usuario no se puede autenticar. No es necesaria ninguna estrategia de detección; Azure AD controla los errores de inicio de sesión.

**Recuperación**

1. Detecte las excepciones no controladas en el middleware.
2. Controle eventos `AuthenticationFailed`.
3. Redireccione al usuario a una página de error.
4. El usuario lo reintenta.

## <a name="azure-search"></a>Azure Search
### <a name="writing-data-to-azure-search-fails"></a>Se produce un error al escribir datos en Azure Search.
**Detección**. Detecte errores `Microsoft.Rest.Azure.CloudException`.

**Recuperación**

El [SDK de .NET de Search][search-sdk] realiza reintentos automáticamente después de los errores transitorios. Las excepciones producidas por el cliente SDK deben tratarse como errores no transitorios.

La directiva de reintentos predeterminada usa la interrupción exponencial. Para utilizar una directiva de reintentos diferente, llame a `SetRetryPolicy` en la clase `SearchIndexClient` o `SearchServiceClient`. Para más información, consulte el artículo sobre [reintentos automáticos][auto-rest-client-retry].

**Diagnóstico**. Utilice [Search Traffic Analytics][search-analytics] (Análisis del tráfico de búsqueda).///

### <a name="reading-data-from-azure-search-fails"></a>Se produce un error al leer datos de Azure Search.
**Detección**. Detecte errores `Microsoft.Rest.Azure.CloudException`.

**Recuperación**

El [SDK de .NET de Search][search-sdk] realiza reintentos automáticamente después de los errores transitorios. Las excepciones producidas por el cliente SDK deben tratarse como errores no transitorios.

La directiva de reintentos predeterminada usa la interrupción exponencial. Para utilizar una directiva de reintentos diferente, llame a `SetRetryPolicy` en la clase `SearchIndexClient` o `SearchServiceClient`. Para más información, consulte el artículo sobre [reintentos automáticos][auto-rest-client-retry].

**Diagnóstico**. Utilice [Search Traffic Analytics][search-analytics] (Análisis del tráfico de búsqueda).///

## <a name="cassandra"></a>Cassandra
### <a name="reading-or-writing-to-a-node-fails"></a>Se produce un error al leer o escribir en un nodo.
**Detección**. Detecte la excepción. Para clientes. NET, suele ser `System.Web.HttpException`. Otros clientes pueden tener otros tipos de excepción.  Para más información, consulte [Cassandra error handling done right](http://www.datastax.com/dev/blog/cassandra-error-handling-done-right) (Correcto control de errores en Cassandra).

**Recuperación**

* Cada [cliente de Cassandra](https://wiki.apache.org/cassandra/ClientOptions) tiene sus propias directivas y funcionalidades de reintento. Para más información, consulte [Cassandra error handling done right][cassandra-error-handling] (Correcto control de errores en Cassandra).
* Use una implementación en modo compatible con bastidor, con nodos de datos distribuidos a través de los dominios de error.
* Implemente en varias regiones con coherencia de cuórum local. Si se produce un error no transitorio, conmute por error a otra región.

**Diagnóstico**. Registros de aplicación

## <a name="cloud-service"></a>Servicio en la nube
### <a name="web-or-worker-roles-are-unexpectedlybeing-shut-down"></a>Los roles web o de trabajo se cierran inesperadamente.
**Detección**. Se desencadena el evento [RoleEnvironment.Stopping][RoleEnvironment.Stopping].

<strong>Recuperación</strong>. Invalide el método [RoleEntryPoint.OnStop][RoleEntryPoint.OnStop] para limpiar correctamente. Para más información, consulte [The Right Way to Handle Azure OnStop Events][onstop-events] (Modo correcto de controlar eventos OnStop de Azure)/// (blog).

## <a name="cosmos-db"></a>Cosmos DB 
### <a name="reading-data-fails"></a>Se produce un error en la lectura de datos.
**Detección**. Detecte `System.Net.Http.HttpRequestException` o `Microsoft.Azure.Documents.DocumentClientException`.

**Recuperación**

* El SDK vuelve a ejecutar automáticamente los intentos erróneos. Para establecer el número de reintentos y el tiempo de espera máximo, configure `ConnectionPolicy.RetryOptions`. Las excepciones que genera el cliente se encuentran más allá de la directiva de reintentos o no son errores transitorios.
* Si Cosmos DB limita el cliente, devuelve un error HTTP 429. Compruebe el código de estado en `DocumentClientException`. Si recibe el error 429 constantemente, considere la posibilidad de aumentar el valor de rendimiento de la colección.
    * Si está utilizando la API de MongoDB, el servicio devuelve el código de error 16500 durante la limitación.
* Replique la base de datos de Cosmos DB en dos o más regiones. Todas las réplicas son legibles. Con los SDK de cliente, especifique el parámetro `PreferredLocations`. Se trata de una lista ordenada de las regiones de Azure. Todas las lecturas se enviarán a la primera región disponible en la lista. Si se produce un error en la solicitud, el cliente intentará las demás regiones en la lista, en orden. Para más información, consulte [Configuración de la distribución global de Azure Cosmos DB con la API de SQL][cosmosdb-multi-region].

**Diagnóstico**. Registre todos los errores en el lado cliente.

### <a name="writing-data-fails"></a>Se produce un error en la escritura de datos.
**Detección**. Detecte `System.Net.Http.HttpRequestException` o `Microsoft.Azure.Documents.DocumentClientException`.

**Recuperación**

* El SDK vuelve a ejecutar automáticamente los intentos erróneos. Para establecer el número de reintentos y el tiempo de espera máximo, configure `ConnectionPolicy.RetryOptions`. Las excepciones que genera el cliente se encuentran más allá de la directiva de reintentos o no son errores transitorios.
* Si Cosmos DB limita el cliente, devuelve un error HTTP 429. Compruebe el código de estado en `DocumentClientException`. Si recibe el error 429 constantemente, considere la posibilidad de aumentar el valor de rendimiento de la colección.
* Replique la base de datos de Cosmos DB en dos o más regiones. Si se produce un error en la región primaria, se promoverá otra región para la escritura. También puede desencadenar una conmutación por error manualmente. El SDK realiza la detección y el enrutamiento automáticos, por lo que el código de la aplicación continúa funcionando después de una conmutación por error. Durante el período de conmutación por error (normalmente minutos), las operaciones de escritura tendrán una latencia superior, según va encontrando el SDK la nueva región de escritura.
  Para más información, consulte [Configuración de la distribución global de Azure Cosmos DB con la API de SQL][cosmosdb-multi-region].
* Como acción de reserva, conserve el documento en una cola de copia de seguridad y procese la cola más adelante.

**Diagnóstico**. Registre todos los errores en el lado cliente.

## <a name="elasticsearch"></a>Elasticsearch
### <a name="reading-data-from-elasticsearch-fails"></a>Se produce un error al leer datos de Elasticsearch.
**Detección**. Detecte la excepción adecuada para el [cliente de Elasticsearch][elasticsearch-client] que se va a usar.

**Recuperación**

* Use un mecanismo de reintento. Cada cliente tiene sus propias directivas de reintento.
* Implemente varios nodos de Elasticsearch y use la replicación para lograr una alta disponibilidad.

Para más información, consulte [Running Elasticsearch on Azure][elasticsearch-azure] (Ejecución de Elasticsearch en Azure)///.

**Diagnóstico**. Puede usar herramientas de supervisión para Elasticsearch o registrar todos los errores en el lado cliente con la carga. Consulte la sección "Supervisión" en [Running Elasticsearch on Azure][elasticsearch-azure] (Ejecución de Elasticsearch en Azure)///.

### <a name="writing-data-to-elasticsearch-fails"></a>Se produce un error al escribir datos en Elasticsearch.
**Detección**. Detecte la excepción adecuada para el [cliente de Elasticsearch][elasticsearch-client] que se va a usar.  

**Recuperación**

* Use un mecanismo de reintento. Cada cliente tiene sus propias directivas de reintento.
* Si la aplicación puede tolerar un nivel de coherencia reducido, considere la posibilidad de escribir con el valor `write_consistency` de `quorum`.

Para más información, consulte [Running Elasticsearch on Azure][elasticsearch-azure] (Ejecución de Elasticsearch en Azure)///.

**Diagnóstico**. Puede usar herramientas de supervisión para Elasticsearch o registrar todos los errores en el lado cliente con la carga. Consulte la sección "Supervisión" en [Running Elasticsearch on Azure][elasticsearch-azure] (Ejecución de Elasticsearch en Azure)///.

## <a name="queue-storage"></a>Queue Storage
### <a name="writing-a-message-to-azure-queue-storage-fails-consistently"></a>Se produce un error al escribir un mensaje en Azure Queue Storage constantemente.
**Detección**. Después de *N* reintentos, todavía se produce un error en la operación de escritura.

**Recuperación**

* Almacene los datos en una memoria caché local y reenvíe las escrituras al almacenamiento más adelante, cuando el servicio esté disponible.
* Cree una cola secundaria y escriba en esa cola si la cola principal no está disponible.

**Diagnóstico**. Use [métricas de almacenamiento][storage-metrics].

### <a name="the-application-cannot-process-a-particular-message-from-the-queue"></a>La aplicación no puede procesar un mensaje concreto de la cola.
**Detección**. Específica de la aplicación. Por ejemplo, el mensaje contiene datos no válidos o se produce un error en la lógica de negocios por algún motivo.

**Recuperación**

Mueva el mensaje a una cola independiente. Ejecute un proceso independiente para examinar los mensajes de dicha cola.

Considere la posibilidad de usar colas de mensajería de Azure Service Bus, lo que proporciona una funcionalidad de [cola de mensajes fallidos][sb-dead-letter-queue] para este propósito.

> [!NOTE]
> Si usa colas de Storage con WebJobs, el SDK de WebJobs proporciona control de mensajes dudosos integrado. Consulte [How to use Azure queue storage with the WebJobs SDK][sb-poison-message] (Uso del almacenamiento de colas de Azure con el SDK de WebJobs)///.

**Diagnóstico**. Utilice el registro de aplicaciones.

## <a name="redis-cache"></a>Caché en Redis
### <a name="reading-from-the-cache-fails"></a>Se produce un error de lectura en la memoria caché.
**Detección**. Detecte `StackExchange.Redis.RedisConnectionException`.

**Recuperación**

1. Reinténtelo con los errores transitorios. Azure Redis Cache admite el reintento integrado. Consulte [Directrices de reintento de Azure Redis Cache][redis-retry].
2. Trate los errores no transitorios como un error de caché y recurra al origen de datos original.

**Diagnóstico**. Utilice [diagnósticos de Redis Cache][redis-monitor].

### <a name="writing-to-the-cache-fails"></a>Se produce un error de escritura en la memoria caché.
**Detección**. Detecte `StackExchange.Redis.RedisConnectionException`.

**Recuperación**

1. Reinténtelo con los errores transitorios. Azure Redis Cache admite el reintento integrado. Consulte [Directrices de reintento de Azure Redis Cache][redis-retry].
2. Si el error no es transitorio, podrá omitirlo y permitir que otras transacciones escriban en la memoria caché más adelante.

**Diagnóstico**. Utilice [diagnósticos de Redis Cache][redis-monitor].

## <a name="sql-database"></a>SQL Database
### <a name="cannot-connect-to-the-database-in-the-primary-region"></a>No se puede conectar a la base de datos en la región primaria.
**Detección**. Se produce un error de conexión.

**Recuperación**

Requisito previo: la base de datos debe configurarse para una replicación geográfica activa. Consulte [Replicación geográfica activa para SQL Database][sql-db-replication].

* Para consultas, lea desde una réplica secundaria.
* Para inserciones y actualizaciones, conmute por error manualmente a una réplica secundaria. Consulte [Inicio de una conmutación por error planeada o no planeada para Azure SQL Database][sql-db-failover].

La réplica usa una cadena de conexión diferente, por lo que deberá actualizar la cadena de conexión en la aplicación.

### <a name="client-runs-out-of-connections-in-the-connection-pool"></a>El cliente se ha quedado sin conexiones en el grupo de conexiones.
**Detección**. Detecte errores `System.InvalidOperationException`.

**Recuperación**

* Vuelva a intentar la operación.
* Como plan de mitigación, aísle los grupos de conexiones para cada caso de uso, a fin de que un solo caso de uso no pueda dominar todas las conexiones.
* Aumente el número máximo de conexiones.

**Diagnóstico**. Registros de aplicaciones.

### <a name="database-connection-limit-is-reached"></a>Se alcanzó el límite de conexiones de base de datos.
**Detección**. Azure SQL Database limita el número de trabajos, inicios de sesión y sesiones simultáneos. Los límites dependen del nivel de servicio. Para más información, consulte [Límites de recursos de Azure SQL Database][sql-db-limits].

Para detectar estos errores, detecte `System.Data.SqlClient.SqlException` y compruebe el valor de `SqlException.Number` para el código de error SQL. Para ver una lista de códigos de error relevantes, consulte [Códigos de error para las aplicaciones cliente de SQL Database: errores de conexión de bases de datos y otros problemas][sql-db-errors].

**Recuperación**. Estos errores se consideran transitorios por lo que al reintentarlo, es posible que se resuelva el problema. Si se producen constantemente estos errores, considere la posibilidad de escalar la base de datos.

**Diagnóstico**. - La consulta [sys.event_log][sys.event_log] devuelve las conexiones realizadas correctamente, los errores de conexión y los interbloqueos.

* Cree una [regla de alertas][azure-alerts] para conexiones erróneas.
* Habilite la [auditoría de SQL Database][sql-db-audit] y compruebe si hay inicios de sesión erróneos.

## <a name="service-bus-messaging"></a>Mensajería de Service Bus
### <a name="reading-a-message-from-a-service-bus-queue-fails"></a>Se produce un error al leer un mensaje de una cola de Service Bus.
**Detección**. Detecte excepciones desde el cliente SDK. La clase base para las excepciones de Service Bus es [MessagingException][sb-messagingexception-class]. Si el error es transitorio, la propiedad `IsTransient` es true.

Para más información, consulte [Excepciones de mensajería de Service Bus][sb-messaging-exceptions].

**Recuperación**

1. Reinténtelo con los errores transitorios. Consulte [Directrices de reintento de Service Bus][sb-retry].
2. Los mensajes que no se pueden entregar a cualquier receptor se colocan en una *cola de mensajes fallidos*. Use esta cola para ver qué mensajes no se pudieron recibir. No hay limpieza automática de la cola de mensajes fallidos. Los mensajes permanecen allí hasta que los recupere explícitamente. Consulte [Información general de colas de mensajes fallidos de Service Bus][sb-dead-letter-queue].

### <a name="writing-a-message-to-a-service-bus-queue-fails"></a>Se produce un error al escribir un mensaje en una cola de Service Bus.
**Detección**. Detecte excepciones desde el cliente SDK. La clase base para las excepciones de Service Bus es [MessagingException][sb-messagingexception-class]. Si el error es transitorio, la propiedad `IsTransient` es true.

Para más información, consulte [Excepciones de mensajería de Service Bus][sb-messaging-exceptions].

**Recuperación**

1. El cliente de Service Bus realiza reintentos automáticamente tras los errores transitorios. De forma predeterminada, usa la interrupción exponencial. Tras el período de tiempo de expiración máximo o el número máximo de reintentos, el cliente genera una excepción. Para más información, consulte [Directrices de reintento de Service Bus][sb-retry].
2. Si se supera la cuota de cola, el cliente genera [QuotaExceededException][QuotaExceededException]. El mensaje de excepción proporciona más detalles. Purgue algunos mensajes de la cola antes de reintentarlo y considere la posibilidad de usar el patrón Circuit Breaker para evitar reintentos continuos mientras se supera la cuota. Además, asegúrese de que la propiedad [BrokeredMessage.TimeToLive] no esté establecida demasiado alta.
3. Dentro de una región, se puede mejorar la resistencia mediante [temas o colas con particiones][sb-partition]. Se asigna una cola o un tema sin particiones a un almacén de mensajería. Si este almacén de mensajería no está disponible, se producirá un error en todas las operaciones de esa cola o tema. Una cola o tema con particiones está particionado entre varios almacenes de mensajería.
4. Para proporcionar una resistencia adicional, cree dos espacios de nombres de Service Bus en diferentes regiones y replique los mensajes. Puede usar una replicación activa o pasiva.

   * Replicación activa: el cliente envía todos los mensajes a ambas colas. El receptor escucha en ambas colas. Etiquete los mensajes con un identificador único, para que el cliente pueda descartar los mensajes duplicados.
   * Replicación pasiva: el cliente envía el mensaje a una cola. Si se produce un error, el cliente recurre a la otra cola. El receptor escucha en ambas colas. Este enfoque reduce el número de mensajes duplicados que se envían. Sin embargo, el receptor todavía debe controlar los mensajes duplicados.

     Para más información, consulte [GeoReplication sample][sb-georeplication-sample] (Muestra de georeplicación) y [Procedimientos recomendados para aislar aplicaciones ante desastres e interrupciones de Service Bus](/azure/service-bus-messaging/service-bus-outages-disasters/).

### <a name="duplicate-message"></a>Mensaje duplicado.
**Detección**. Examine las propiedades `MessageId` y `DeliveryCount` del mensaje.

**Recuperación**

* Si es posible, diseñe las operaciones de procesamiento de mensajes para que sean idempotentes. En caso contrario, almacene identificadores de mensaje de los mensajes que ya se han procesado y compruebe el identificador antes de procesar un mensaje.
* Habilite la detección de duplicados, mediante la creación de la cola con `RequiresDuplicateDetection` establecido en true. Con este valor, Service Bus elimina automáticamente cualquier mensaje que se envíe con el mismo `MessageId` que un mensaje anterior.  Tenga en cuenta lo siguiente:

  * Este valor impide que se pongan en la cola mensajes duplicados. No impide que un receptor procese el mismo mensaje más de una vez.
  * La detección de duplicados tiene un período de tiempo. Si se envía un duplicado más allá de este período, no se detectará.

**Diagnóstico**. Registre mensajes duplicados.

### <a name="the-application-cannot-process-a-particular-message-from-the-queue"></a>La aplicación no puede procesar un mensaje concreto de la cola.
**Detección**. Específica de la aplicación. Por ejemplo, el mensaje contiene datos no válidos o se produce un error en la lógica de negocios por algún motivo.

**Recuperación**

Hay dos modos de error que se deben tener en cuenta.

* El receptor detecta el error. En este caso, mueva el mensaje a la cola de mensajes fallidos. Posteriormente, ejecute un proceso independiente para examinar los mensajes de la cola de mensajes fallidos.
* Se produce un error en el receptor en medio del procesamiento del mensaje &mdash; por ejemplo, debido a una excepción no controlada. Para controlar este caso, utilice el modo `PeekLock`. En este modo, si expira el bloqueo, el mensaje pasa a estar disponible para otros receptores. Si el mensaje supera el número máximo de entregas o el período de vida, el mensaje se mueve automáticamente a la cola de mensajes fallidos.

Para más información, consulte [Información general de colas de mensajes fallidos de Service Bus][sb-dead-letter-queue].

**Diagnóstico**. Siempre que la aplicación mueva un mensaje a la cola de mensajes fallidos, escribe un evento en los registros de aplicaciones.

## <a name="service-fabric"></a>Service Fabric
### <a name="a-request-to-a-service-fails"></a>Se produce un error en una solicitud a un servicio.
**Detección**. El servicio devuelve un error.

**Recuperación**

* Busque de nuevo un proxy (`ServiceProxy` o `ActorProxy`) y vuelva a llamar al método de servicio/actor.
* **Servicio con estado**. Encapsule las operaciones en colecciones confiables en una transacción. Si se produce un error, se revertirá la transacción. La solicitud, si se extrae de una cola, se procesará de nuevo.
* **Servicio sin estado**. Si el servicio conserva los datos en un almacén externo, todas las operaciones deben ser idempotentes.

**Diagnóstico**. Registro de aplicaciones

### <a name="service-fabric-node-is-shut-down"></a>El nodo de Service Fabric se cierra.
**Detección**. Se pasa un token de cancelación para el método `RunAsync` del servicio. Service Fabric cancela la tarea antes de cerrar el nodo.

**Recuperación**. Use el token de cancelación para detectar el cierre. Cuando Service Fabric solicita la cancelación, finalice cualquier trabajo y salga de  `RunAsync` tan pronto como sea posible.

**Diagnóstico**. Registros de aplicación

## <a name="storage"></a>Storage
### <a name="writing-data-to-azure-storage-fails"></a>Se produce un error al escribir datos en Azure Storage.
**Detección**. El cliente recibe mensajes de error al escribir.

**Recuperación**

1. Reintente la operación para recuperarse de errores transitorios. La [directiva de reintentos][Storage.RetryPolicies] en el SDK de cliente trata esto automáticamente.
2. Implemente el patrón Circuit Breaker para evitar sobrecargar el almacenamiento.
3. Si se produce un error en N reintentos, realice una acción de reserva correcta. Por ejemplo: 

   * Almacene los datos en una memoria caché local y reenvíe las escrituras al almacenamiento más adelante, cuando el servicio esté disponible.
   * Si la acción de escritura se encontraba en un ámbito transaccional, compense la transacción.

**Diagnóstico**. Use [métricas de almacenamiento][storage-metrics].

### <a name="reading-data-from-azure-storage-fails"></a>Se produce un error al leer datos de Azure Storage.
**Detección**. El cliente recibe mensajes de error al leer.

**Recuperación**

1. Reintente la operación para recuperarse de errores transitorios. La [directiva de reintentos][Storage.RetryPolicies] en el SDK de cliente trata esto automáticamente.
2. Para el almacenamiento de RA-GRS, si se produce un error en la lectura desde el punto de conexión principal, intente leer desde el punto de conexión secundario. El SDK de cliente trata esto automáticamente. Consulte el artículo sobre [replicación de Azure Storage][storage-replication].
3. Si se produce un error en *N* reintentos, realice una acción de reserva para ejecutar una degradación correcta. Por ejemplo, si una imagen de producto no se puede recuperar desde el almacenamiento, muestre una imagen de marcador de posición genérica.

**Diagnóstico**. Use [métricas de almacenamiento][storage-metrics].

## <a name="virtual-machine"></a>Máquina virtual
### <a name="connection-to-a-backend-vm-fails"></a>Se produce un error en la conexión a una máquina virtual de back-end.
**Detección**. Errores de conexión de red.

**Recuperación**

* Implemente al menos dos máquinas virtuales de back-end en un conjunto de disponibilidad, detrás de un equilibrador de carga.
* Si el error de conexión es transitorio, a veces TCP reintentará enviar correctamente el mensaje.
* Implemente una directiva de reintentos en la aplicación.
* Para errores persistentes o no transitorios, implemente el patrón [Circuit Breaker][circuit-breaker].
* Si la máquina virtual que realiza la llamada supera el límite de salida de la red, se llenará la cola de salida. Si la cola de salida está constantemente llena, considere la posibilidad de un escalado horizontal.

**Diagnóstico**. Registre eventos en los límites del servicio.

### <a name="vm-instance-becomes-unavailable-or-unhealthy"></a>Una instancia de la máquina virtual de App Service ha dejado de estar disponible o no tiene mantenimiento.
**Detección**. Configure un [sondeo de mantenimiento][lb-probe] de una instancia de Load Balancer que indique si la instancia de la máquina virtual tiene mantenimiento. El sondeo debe comprobar si las funciones críticas están respondiendo correctamente.

**Recuperación**. Para cada capa de aplicación, colocar varias instancias de máquina virtual en el mismo conjunto de disponibilidad y colocar un equilibrador de carga delante de las máquinas virtuales. Cuando un sondeo de mantenimiento no responde, la instancia de Load Balancer deja de enviar nuevas conexiones a la instancia sin mantenimiento.

**Diagnóstico**. - Use [análisis de registros][lb-monitor] de Load Balancer.

* Configure el sistema de supervisión para controlar todos los puntos de conexión de supervisión de mantenimiento.

### <a name="operator-accidentally-shuts-down-a-vm"></a>Un operador cierra accidentalmente una máquina virtual.
**Detección**. N/D

**Recuperación**. Establezca un bloqueo de recurso con el nivel `ReadOnly`. Consulte [Bloqueo de recursos con Azure Resource Manager][rm-locks].

**Diagnóstico**. Utilice [Registros de actividad de Azure][azure-activity-logs].

## <a name="webjobs"></a>Trabajos web
### <a name="continuous-job-stops-running-when-the-scm-host-is-idle"></a>El trabajo continuo deja de ejecutarse cuando el host SCM está inactivo.
**Detección**. Pase un token de cancelación a la función de WebJob. Para más información, consulte el [cierre correcto][web-jobs-shutdown].

**Recuperación**. Habilite el valor `Always On` en la aplicación web. Para más información, consulte [Ejecución de tareas en segundo plano con WebJobs][web-jobs].

## <a name="application-design"></a>Diseño de aplicación
### <a name="application-cant-handle-a-spike-in-incoming-requests"></a>La aplicación no puede controlar un pico en las solicitudes entrantes.
**Detección**. Depende de la aplicación. Síntomas típicos:

* El sitio web comienza a devolver códigos de error HTTP 5xx.
* Los servicios dependientes, como base de datos o almacenamiento, comienzan a limitar las solicitudes. Busque errores HTTP, como HTTP 429 (demasiadas solicitudes), en función del servicio.
* La longitud de la cola HTTP aumenta.

**Recuperación**

* Escale horizontalmente para controlar el aumento de la carga.
* Mitigue los errores para evitar que unos errores en cascada interrumpan toda la aplicación. Entre las estrategias de mitigación se incluyen:

  * Implemente el [patrón Throttling][throttling-pattern] para evitar sobrecargar los sistemas back-end.
  * Use [Queue-Based Load Leveling][queue-based-load-leveling] para almacenar en búfer las solicitudes y procesarlas a un ritmo adecuado.
  * Dé prioridad a determinados clientes. Por ejemplo, si la aplicación tiene planes de tarifa gratuitos y de pago, limite los clientes del plan de tarifa gratuito, pero no los de pago. Consulte [Patrón de cola de prioridad][priority-queue-pattern].

**Diagnóstico**. Use el [registro de diagnóstico de App Service][app-service-logging]. Utilice un servicio como [Azure Log Analytics][azure-log-analytics], [Application Insights][app-insights] o [New Relic][new-relic] para que le ayuden a comprender los registros de diagnóstico.

### <a name="one-of-the-operations-in-a-workflow-or-distributed-transaction-fails"></a>Se produce un error en una de las operaciones en un flujo de trabajo o una transacción distribuida.
**Detección**. Después de *N* reintentos, sigue produciendo un error.

**Recuperación**

* Como plan de mitigación, implemente el patrón [Scheduler Agent Supervisor][scheduler-agent-supervisor] para administrar todo el flujo de trabajo.
* No lo intente de nuevo en los tiempos de expiración. Hay una tasa de éxito baja para este error.
* Ponga el trabajo en la cola para volver a intentarlo más tarde.

**Diagnóstico**. Inicie sesión en todas las operaciones (correctas y erróneas), incluidas las acciones de compensación. Utilice los identificadores de correlación para que pueda realizar un seguimiento de todas las operaciones dentro de la misma transacción.

### <a name="a-call-to-a-remote-service-fails"></a>Se produce un error en una llamada a un servicio remoto.
**Detección**. Código de error HTTP.

**Recuperación**

1. Reinténtelo con los errores transitorios.
2. Si se produce un error en la llamada después de *N* reintentos, realice una acción de reserva. (Específica de la aplicación).
3. Implemente el [patrón Circuit Breaker][circuit-breaker] para evitar errores en cascada.

**Diagnóstico**. Registre todos los errores de llamadas remotas.

## <a name="next-steps"></a>Pasos siguientes
Para más información sobre el proceso FMA, consulte [Resilience by design for cloud services][resilience-by-design-pdf] [Resistencia mediante diseño para servicios en la nube] (descarga de PDF).

<!-- links -->

[api-management]: https://azure.microsoft.com/documentation/services/api-management/
[api-management-throttling]: /azure/api-management/api-management-sample-flexible-throttling/
[app-insights]: /azure/application-insights/app-insights-overview/
[app-insights-web-apps]: /azure/application-insights/app-insights-azure-web-apps/
[app-service-configure]: /azure/app-service-web/web-sites-configure/
[app-service-logging]: /azure/app-service-web/web-sites-enable-diagnostic-log/
[app-service-slots]: /azure/app-service-web/web-sites-staged-publishing/
[auto-rest-client-retry]: https://github.com/Azure/autorest/tree/master/docs
[azure-activity-logs]: /azure/monitoring-and-diagnostics/monitoring-overview-activity-logs/
[azure-alerts]: /azure/monitoring-and-diagnostics/insights-alerts-portal/
[azure-log-analytics]: /azure/log-analytics/log-analytics-overview/
[BrokeredMessage.TimeToLive]: https://msdn.microsoft.com/library/microsoft.servicebus.messaging.brokeredmessage.timetolive.aspx
[cassandra-error-handling]: http://www.datastax.com/dev/blog/cassandra-error-handling-done-right
[circuit-breaker]: https://msdn.microsoft.com/library/dn589784.aspx
[cosmosdb-multi-region]: /azure/cosmos-db/tutorial-global-distribution-sql-api
[elasticsearch-azure]: ../elasticsearch/index.md
[elasticsearch-client]: https://www.elastic.co/guide/en/elasticsearch/client/index.html
[health-endpoint-monitoring-pattern]: https://msdn.microsoft.com/library/dn589789.aspx
[onstop-events]: https://azure.microsoft.com/blog/the-right-way-to-handle-azure-onstop-events/
[lb-monitor]: /azure/load-balancer/load-balancer-monitor-log/
[lb-probe]: /azure/load-balancer/load-balancer-custom-probe-overview/#learn-about-the-types-of-probes
[new-relic]: https://newrelic.com/
[priority-queue-pattern]: https://msdn.microsoft.com/library/dn589794.aspx
[queue-based-load-leveling]: https://msdn.microsoft.com/library/dn589783.aspx
[QuotaExceededException]: https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.quotaexceededexception.aspx
[ra-web-apps-basic]: ../reference-architectures/app-service-web-app/basic-web-app.md
[redis-monitor]: /azure/redis-cache/cache-how-to-monitor/
[redis-retry]: ../best-practices/retry-service-specific.md#azure-redis-cache-retry-guidelines
[resilience-by-design-pdf]: http://download.microsoft.com/download/D/8/C/D8C599A4-4E8A-49BF-80EE-FE35F49B914D/Resilience_by_Design_for_Cloud_Services_White_Paper.pdf
[RoleEntryPoint.OnStop]: https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleentrypoint.onstop.aspx
[RoleEnvironment.Stopping]: https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleenvironment.stopping.aspx
[rm-locks]: /azure/azure-resource-manager/resource-group-lock-resources/
[sb-dead-letter-queue]: /azure/service-bus-messaging/service-bus-dead-letter-queues/
[sb-georeplication-sample]: https://github.com/Azure-Samples/azure-servicebus-messaging-samples/tree/master/GeoReplication
[sb-messagingexception-class]: https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.messagingexception.aspx
[sb-messaging-exceptions]: /azure/service-bus-messaging/service-bus-messaging-exceptions/
[sb-outages]: /azure/service-bus-messaging/service-bus-outages-disasters/#protecting-queues-and-topics-against-datacenter-outages-or-disasters
[sb-partition]: /azure/service-bus-messaging/service-bus-partitioning/
[sb-poison-message]: /azure/app-service-web/websites-dotnet-webjobs-sdk-storage-queues-how-to/#poison
[sb-retry]: ../best-practices/retry-service-specific.md#service-bus-retry-guidelines
[search-sdk]: https://msdn.microsoft.com/library/dn951165.aspx
[scheduler-agent-supervisor]: https://msdn.microsoft.com/library/dn589780.aspx
[search-analytics]: /azure/search/search-traffic-analytics/
[sql-db-audit]: /azure/sql-database/sql-database-auditing-get-started/
[sql-db-errors]: /azure/sql-database/sql-database-develop-error-messages/#resource-governance-errors
[sql-db-failover]: /azure/sql-database/sql-database-geo-replication-failover-portal/
[sql-db-limits]: /azure/sql-database/sql-database-resource-limits/
[sql-db-replication]: /azure/sql-database/sql-database-geo-replication-overview/
[storage-metrics]: https://msdn.microsoft.com/library/dn782843.aspx
[storage-replication]: /azure/storage/storage-redundancy/
[Storage.RetryPolicies]: https://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.aspx
[sys.event_log]: https://msdn.microsoft.com/library/dn270018.aspx
[throttling-pattern]: https://msdn.microsoft.com/library/dn589798.aspx
[web-jobs]: /azure/app-service-web/web-sites-create-web-jobs/
[web-jobs-shutdown]: /azure/app-service-web/websites-dotnet-webjobs-sdk-storage-queues-how-to/#graceful
