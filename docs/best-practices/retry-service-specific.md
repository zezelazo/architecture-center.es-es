---
title: Vuelva a intentar la orientación específica del servicio
titleSuffix: Best practices for cloud applications
description: Instrucciones específicas de servicios para establecer el mecanismo de reintento.
author: dragon119
ms.date: 08/13/2018
ms.custom: seodec18
ms.openlocfilehash: ad26b55625276ae95004652acfd745b2c4b53a8f
ms.sourcegitcommit: 4ba3304eebaa8c493c3e5307bdd9d723cd90b655
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/12/2018
ms.locfileid: "53307357"
---
# <a name="retry-guidance-for-specific-services"></a>Guía de reintentos para servicios específicos

La mayoría de SDK de cliente y de servicios de Azure incluye un mecanismo de reintento. Sin embargo, estos son diferentes porque cada servicio tiene requisitos y características diferentes, por lo que cada mecanismo de reintento se ajusta a un servicio específico. Esta guía resume las características de mecanismo de reintento de la mayoría de los servicios de Azure e incluye información para ayudarle a usar, adaptar o ampliar el mecanismo de reintento para ese servicio.

Para obtener instrucciones generales sobre el control de errores transitorios y reintentar conexiones y operaciones en servicios y recursos, consulte [Guía de reintentos](./transient-faults.md).

En la tabla siguiente se resumen las características de reintento de los servicios de Azure descritos en esta guía.

| **Servicio** | **Capacidades de reintento** | **Configuración de directivas** | **Ámbito** | **Características de telemetría** |
| --- | --- | --- | --- | --- |
| **[Azure Active Directory](#azure-active-directory)** |Nativo en la biblioteca ADAL |Insertado en la biblioteca ADAL |Interno |None |
| **[Cosmos DB](#cosmos-db)** |Nativo en servicio |No configurable |Global |TraceSource |
| **Data Lake Store** |Nativo en el cliente |No configurable |Operaciones individuales |None |
| **[Event Hubs](#event-hubs)** |Nativo en el cliente |Programático |Cliente |None |
| **[IoT Hub](#iot-hub)** |Nativo en el SDK de cliente |Programático |Cliente |None |
| **[Redis Cache](#azure-redis-cache)** |Nativo en el cliente |Programático |Cliente |TextWriter |
| **[Search](#azure-search)** |Nativo en el cliente |Programático |Cliente |ETW o personalizado |
| **[Service Bus](#service-bus)** |Nativo en el cliente |Programático |Administrador de espacio de nombres, fábrica de mensajería y cliente |ETW |
| **[Service Fabric](#service-fabric)** |Nativo en el cliente |Programático |Cliente |None |
| **[SQL Database con ADO.NET](#sql-database-using-adonet)** |[Polly](#transient-fault-handling-with-polly) |Declarativo y programático |Instrucciones únicas o bloques de código |Personalizado |
| **[SQL Database con Entity Framework](#sql-database-using-entity-framework-6)** |Nativo en el cliente |Programático |Global por AppDomain |None |
| **[SQL Database con Entity Framework Core](#sql-database-using-entity-framework-core)** |Nativo en el cliente |Programático |Global por AppDomain |None |
| **[Storage](#azure-storage)** |Nativo en el cliente |Programático |Operaciones de cliente e individuales |TraceSource |

> [!NOTE]
> Para la mayoría de los mecanismos de reintento integrados de Azure, actualmente no hay ninguna manera de aplicar una directiva de reintento diferente para distintos tipos de error o excepción. Debe configurar una directiva que proporcione un promedio de rendimiento y disponibilidad óptimo. Una forma de ajustar la directiva consiste en analizar archivos de registro para determinar el tipo de errores transitorios que se están produciendo.

<!-- markdownlint-disable MD024 MD033 -->

## <a name="azure-active-directory"></a>Azure Active Directory

Azure Active Directory (Azure AD) es una solución de nube de administración de identidades y accesos integral que combina servicios de directorio de núcleo, gobierno de identidades avanzado, seguridad y administración de acceso a aplicaciones. Azure AD también ofrece a los desarrolladores una plataforma de administración de identidades para proporcionar control de acceso a sus aplicaciones, según las reglas y directivas centralizadas.

> [!NOTE]
> Para obtener instrucciones de reintento en los puntos de conexión de Managed Service Identity, consulte [Uso de una identidad de servicio administrada de máquina virtual de Azure para obtener tokens](/azure/active-directory/managed-service-identity/how-to-use-vm-token#error-handling).

### <a name="retry-mechanism"></a>Mecanismo de reintento

Hay un mecanismo de reintento integrado para Azure Active Directory en la biblioteca de autenticación de Active Directory (ADAL). Para evitar bloqueos inesperados, se recomienda que las bibliotecas de terceros y el código de la aplicación **no** reintenten las conexiones con error y que dejen que ADAL controle los reintentos.

### <a name="retry-usage-guidance"></a>Instrucciones de uso del reintento

Tenga en cuenta las siguientes directrices cuando use Azure Active Directory:

- Cuando sea posible, utilice la biblioteca ADAL y la compatibilidad para reintentos integrada.
- Si va a usar la API REST para Azure Active Directory, vuelva a intentar la operación si el código de resultado es 429 (Demasiadas solicitudes) o un error en el intervalo 5xx. No lo intente de nuevo para cualquier otro error.
- Se recomienda una directiva de retroceso exponencial para su uso en escenarios de lotes con Azure Active Directory.

Considere la posibilidad de comenzar con la configuración siguiente para las operaciones de reintento. Esta es la configuración de propósito general, y debe supervisar las operaciones y ajustar los valores para adaptarlos a su propio escenario.

| **Contexto** | **Destino de ejemplo E2E<br />latencia máxima** | **Estrategia de reintento** | **Configuración** | **Valores** | **Cómo funciona** |
| --- | --- | --- | --- | --- | --- |
| Interactivo, interfaz de usuario<br />o primer plano |2 segundos |FixedInterval |Número de reintentos<br />Intervalo de reintento<br />Primer reintento rápido |3<br />500 ms<br />true |Intento 1 - retraso de 0 segundos<br />Intento 2 - retraso de 500 ms<br />Intento 3 – retraso de 500 ms |
| Segundo plano o<br />proceso por lotes |60 segundos |ExponentialBackoff |Número de reintentos<br />Interrupción mínima<br />Interrupción máxima<br />Interrupción delta<br />Primer reintento rápido |5<br />0 segundos<br />60 segundos<br />2 segundos<br />false |Intento 1 - retraso de 0 segundos<br />Intento 2 - retraso de ~2 segundos<br />Intento 3 - retraso de ~6 segundos<br />Intento 4 - retraso de ~14 segundos<br />Intento 5 - retraso de ~30 segundos |

### <a name="more-information"></a>Más información

- [Bibliotecas de autenticación de Azure Active Directory][adal]

## <a name="cosmos-db"></a>Cosmos DB

Cosmos DB es una base de datos multimodelo completamente administrada que admite datos JSON sin esquema. Ofrece un rendimiento confiable y configurable, procesamiento transaccional de JavaScript nativo y se ha creado para la nube con la escala elástica.

### <a name="retry-mechanism"></a>Mecanismo de reintento

La clase `DocumentClient` reintenta automáticamente los intentos con error. Para establecer el número de reintentos y el tiempo de espera máximo, configure [ConnectionPolicy.RetryOptions]. Las excepciones que genera el cliente se encuentran más allá de la directiva de reintentos o no son errores transitorios.

Si Cosmos DB limita el cliente, devuelve un error HTTP 429. Compruebe el código de estado en `DocumentClientException`.

### <a name="policy-configuration"></a>Configuración de directivas

La siguiente tabla muestra la configuración predeterminada de la clase `RetryOptions`.

| Configuración | Valor predeterminado | DESCRIPCIÓN |
| --- | --- | --- |
| MaxRetryAttemptsOnThrottledRequests |9 |Número máximo de reintentos si se produce un error en la solicitud porque Cosmos DB aplicó la limitación de velocidad en el cliente. |
| MaxRetryWaitTimeInSeconds |30 |El tiempo de reintentos máximo en segundos. |

### <a name="example"></a>Ejemplo

```csharp
DocumentClient client = new DocumentClient(new Uri(endpoint), authKey); ;
var options = client.ConnectionPolicy.RetryOptions;
options.MaxRetryAttemptsOnThrottledRequests = 5;
options.MaxRetryWaitTimeInSeconds = 15;
```

### <a name="telemetry"></a>Telemetría

Los reintentos se registran como mensajes de seguimiento no estructurados a través de .NET **TraceSource**. Debe configurar un **TraceListener** para capturar los eventos y escribirlos en un registro de destino adecuado.

Por ejemplo, si agrega lo siguiente al archivo App.config, se generarán seguimientos en un archivo de texto en la misma ubicación que el archivo ejecutable:

```xml
<configuration>
  <system.diagnostics>
    <switches>
      <add name="SourceSwitch" value="Verbose"/>
    </switches>
    <sources>
      <source name="DocDBTrace" switchName="SourceSwitch" switchType="System.Diagnostics.SourceSwitch" >
        <listeners>
          <add name="MyTextListener" type="System.Diagnostics.TextWriterTraceListener" traceOutputOptions="DateTime,ProcessId,ThreadId" initializeData="CosmosDBTrace.txt"></add>
        </listeners>
      </source>
    </sources>
  </system.diagnostics>
</configuration>
```

## <a name="event-hubs"></a>Event Hubs

Azure Event Hubs es un servicio de ingestión de datos de telemetría a hiperescala que recopila, transforma y almacena millones de eventos.

### <a name="retry-mechanism"></a>Mecanismo de reintento

El comportamiento de reintentos de la biblioteca de cliente de Azure Event Hubs se controla mediante la propiedad `RetryPolicy` de la clase `EventHubClient`. La directiva predeterminada realiza reintentos con retroceso exponencial cuando Azure Event Hubs devuelve una excepción transitoria `EventHubsException` o `OperationCanceledException`.

### <a name="example"></a>Ejemplo

```csharp
EventHubClient client = EventHubClient.CreateFromConnectionString("[event_hub_connection_string]");
client.RetryPolicy = RetryPolicy.Default;
```

### <a name="more-information"></a>Más información

[Biblioteca de cliente .NET estándar para Azure Event Hubs](https://github.com/Azure/azure-event-hubs-dotnet)

## <a name="iot-hub"></a>IoT Hub

Azure IoT Hub es un servicio para conectar, supervisar y administrar dispositivos para desarrollar aplicaciones de Internet de las cosas (IoT).

### <a name="retry-mechanism"></a>Mecanismo de reintento

El SDK de dispositivo IoT de Azure puede detectar errores de red, protocolo o aplicación. Según el tipo de error, el SDK de comprueba si debe realizarse un reintento. Si el error es *recuperable*, el SDK comienza a intentarlo de nuevo con la directiva de reintentos configurada.

La directiva de reintentos predeterminada es *retroceso exponencial con distorsión aleatoria*, pero se puede configurar.

### <a name="policy-configuration"></a>Configuración de directivas

La configuración de las directivas difiere según el lenguaje. Para más información, consulte cómo [configurar las directivas de reintentos de IoT Hub](/azure/iot-hub/iot-hub-reliability-features-in-sdks#retry-policy-apis).

### <a name="more-information"></a>Más información

- [Directiva de reintentos de IoT Hub](/azure/iot-hub/iot-hub-reliability-features-in-sdks)
- [Solución de problemas de desconexión de dispositivos de IoT Hub](/azure/iot-hub/iot-hub-troubleshoot-connectivity)

## <a name="azure-redis-cache"></a>Azure Redis Cache

Azure Redis Cache es un servicio de caché de acceso rápido a datos y de baja latencia basado en Redis Cache de código abierto popular. Es segura, está administrada por Microsoft y es accesible desde cualquier aplicación en Azure.

Las instrucciones de esta sección se basan en el uso del cliente StackExchange.Redis para tener acceso a la memoria caché. Puede encontrar una lista de otros clientes adecuados en el [sitio web de Redis](https://redis.io/clients), y estos pueden tener mecanismos de reintento diferentes.

Tenga en cuenta que el cliente StackExchange.Redis usa multiplexación a través de una sola conexión. El uso recomendado es crear una instancia del cliente al iniciar la aplicación y usar esta instancia para todas las operaciones en la memoria caché. Por este motivo, la conexión a la memoria caché se realiza solo una vez y todas las instrucciones de esta sección están relacionadas con la directiva de reintentos de esta conexión inicial y no para cada operación que tiene acceso a la memoria caché.

### <a name="retry-mechanism"></a>Mecanismo de reintento

El cliente StackExchange.Redis usa una clase de administrador de conexiones que se configura mediante un conjunto de opciones, entre ellas:

- **ConnectRetry**. Número de veces que se volverá a intentar una conexión con error a la memoria caché.
- **ReconnectRetryPolicy**. Estrategia de reintentos que se usará.
- **ConnectTimeout**. Tiempo de espera máximo en milisegundos.

### <a name="policy-configuration"></a>Configuración de directivas

Las directivas de reintento se configuran mediante programación, estableciendo las opciones para el cliente antes de conectarse a la memoria caché. Esto puede hacerse mediante la creación de una instancia de la clase **ConfigurationOptions**, rellenando sus propiedades y pasándola al método **Conectar**.

Las clases integradas admiten retrasos lineales (constantes) y retroceso exponencial con intervalos de reintento aleatorios. También puede implementar la interfaz **IReconnectRetryPolicy** para crear una directiva de reintentos personalizada.

En el ejemplo siguiente se configura una estrategia de reintentos con retroceso exponencial.

```csharp
var deltaBackOffInMilliseconds = TimeSpan.FromSeconds(5).Milliseconds;
var maxDeltaBackOffInMilliseconds = TimeSpan.FromSeconds(20).Milliseconds;
var options = new ConfigurationOptions
{
    EndPoints = {"localhost"},
    ConnectRetry = 3,
    ReconnectRetryPolicy = new ExponentialRetry(deltaBackOffInMilliseconds, maxDeltaBackOffInMilliseconds),
    ConnectTimeout = 2000
};
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

Como alternativa, puede especificar las opciones como una cadena y pasar esto al método **Conectar** . Tenga en cuenta que la propiedad **ReconnectRetryPolicy** no se puede establecer de esta manera, sino solo mediante código.

```csharp
var options = "localhost,connectRetry=3,connectTimeout=2000";
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

También es posible especificar las opciones directamente al conectarse a la memoria caché.

```csharp
var conn = ConnectionMultiplexer.Connect("redis0:6380,redis1:6380,connectRetry=3");
```

Para más información, consulte [Configuración de StackExchange.Redis](https://stackexchange.github.io/StackExchange.Redis/Configuration) en la documentación de StackExchange.Redis.

La siguiente tabla muestra la configuración predeterminada de la directiva de reintento integrada.

| **Contexto** | **Configuración** | **Valor predeterminado**<br />(v 1.2.2) | **Significado** |
| --- | --- | --- | --- |
| ConfigurationOptions |ConnectRetry<br /><br />ConnectTimeout<br /><br />SyncTimeout<br /><br />ReconnectRetryPolicy |3<br /><br />Máximo de 5000 ms más SyncTimeout<br />1000<br /><br />LinearRetry 5000 ms |El número de veces que se repiten los intentos de conexión durante la operación de conexión inicial.<br />Tiempo de espera (ms) para conectar las operaciones. No un retraso entre reintentos.<br />Tiempo (ms) para permitir las operaciones sincrónicas.<br /><br />Reintentar cada 5000 ms.|

> [!NOTE]
> Para las operaciones sincrónicas, `SyncTimeout` puede agregar latencia de extremo a extremo, pero si se establece un valor demasiado bajo, puede provocar tiempos de espera excesivos. Consulte [Solución de problemas de Azure Redis Cache][redis-cache-troubleshoot]. Por lo general, evite el uso de las operaciones sincrónicas y use operaciones asincrónicas en su lugar. Para obtener más información, consulte [Canalizaciones y multiplexores](https://github.com/StackExchange/StackExchange.Redis/blob/master/docs/PipelinesMultiplexers.md).

### <a name="retry-usage-guidance"></a>Instrucciones de uso del reintento

Tenga en cuenta las siguientes directrices cuando use Azure Redis Cache:

- El cliente Redis StackExchange administra sus propios reintentos, pero solo al establecer una conexión a la memoria caché cuando la aplicación se inicia por primera vez. Puede configurar el tiempo de espera de conexión, el número de reintentos y el tiempo entre reintentos para establecer esta conexión, pero la directiva de reintentos no se aplica a las operaciones en la memoria caché.
- En lugar de usar un gran número de reintentos, considere la posibilidad de recurrir a obtener acceso a un origen de datos original en su lugar.

### <a name="telemetry"></a>Telemetría

Puede recopilar información acerca de las conexiones (pero no otras operaciones) con un **TextWriter**.

```csharp
var writer = new StringWriter();
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

A continuación, se muestra un ejemplo del resultado que este genera.

```text
localhost:6379,connectTimeout=2000,connectRetry=3
1 unique nodes specified
Requesting tie-break from localhost:6379 > __Booksleeve_TieBreak...
Allowing endpoints 00:00:02 to respond...
localhost:6379 faulted: SocketFailure on PING
localhost:6379 failed to nominate (Faulted)
> UnableToResolvePhysicalConnection on GET
No masters detected
localhost:6379: Standalone v2.0.0, master; keep-alive: 00:01:00; int: Connecting; sub: Connecting; not in use: DidNotRespond
localhost:6379: int ops=0, qu=0, qs=0, qc=1, wr=0, sync=1, socks=2; sub ops=0, qu=0, qs=0, qc=0, wr=0, socks=2
Circular op-count snapshot; int: 0 (0.00 ops/s; spans 10s); sub: 0 (0.00 ops/s; spans 10s)
Sync timeouts: 0; fire and forget: 0; last heartbeat: -1s ago
resetting failing connections to retry...
retrying; attempts left: 2...
...
```

### <a name="examples"></a>Ejemplos

En el ejemplo de código siguiente se configura un retraso constante (lineal) entre los reintentos al inicializar el cliente StackExchange.Redis. Este ejemplo muestra cómo establecer la configuración mediante una instancia de **ConfigurationOptions**.

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using StackExchange.Redis;

namespace RetryCodeSamples
{
    class CacheRedisCodeSamples
    {
        public async static Task Samples()
        {
            var writer = new StringWriter();
            {
                try
                {
                    var retryTimeInMilliseconds = TimeSpan.FromSeconds(4).Milliseconds; // delay between retries

                    // Using object-based configuration.
                    var options = new ConfigurationOptions
                                        {
                                            EndPoints = { "localhost" },
                                            ConnectRetry = 3,
                                            ReconnectRetryPolicy = new LinearRetry(retryTimeInMilliseconds)
                                        };
                    ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);

                    // Store a reference to the multiplexer for use in the application.
                }
                catch
                {
                    Console.WriteLine(writer.ToString());
                    throw;
                }
            }
        }
    }
}
```

En este ejemplo se especifican las opciones como una cadena para establecer la configuración. El tiempo de espera de conexión es el período de tiempo máximo que se esperará a que se realice la conexión con la memoria caché; no es el retraso entre los reintentos. Tenga en cuenta que la propiedad **ReconnectRetryPolicy** solo puede establecerse mediante código.

```csharp
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using StackExchange.Redis;

namespace RetryCodeSamples
{
    class CacheRedisCodeSamples
    {
        public async static Task Samples()
        {
            var writer = new StringWriter();
            {
                try
                {
                    // Using string-based configuration.
                    var options = "localhost,connectRetry=3,connectTimeout=2000";
                    ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);

                    // Store a reference to the multiplexer for use in the application.
                }
                catch
                {
                    Console.WriteLine(writer.ToString());
                    throw;
                }
            }
        }
    }
}
```

Para obtener más ejemplos, consulte [Configuración](https://github.com/StackExchange/StackExchange.Redis/blob/master/docs/Configuration.md) en el sitio web del proyecto.

### <a name="more-information"></a>Más información

- [Sitio web de Redis](https://redis.io/)

## <a name="azure-search"></a>Azure Search

Azure Search puede usarse para agregar capacidades de búsqueda eficaces y sofisticadas a un sitio web o aplicación, ajustar de manera rápida y fácil los resultados de la búsqueda y construir modelos de clasificación enriquecidos y optimizados.

### <a name="retry-mechanism"></a>Mecanismo de reintento

El comportamiento de reintento en el SDK de Azure Search se controla mediante el método `SetRetryPolicy` en las clases [SearchServiceClient] y [SearchIndexClient]. Los reintentos de directiva predeterminada con interrupción exponencial cuando Azure Search devuelve una respuesta 5xx o 408 (tiempo de espera de solicitud).

### <a name="telemetry"></a>Telemetría

Realice un seguimiento con ETW o mediante el registro de un proveedor de seguimiento personalizado. Para más información, consulte la [documentación de AutoRest][autorest].

## <a name="service-bus"></a>Azure Service Bus

Service Bus es una plataforma de mensajería de nube que proporciona el intercambio de mensajes de acoplamiento flexible con mejor escala y resistencia para los componentes de una aplicación, ya esté hospedada en la nube o localmente.

### <a name="retry-mechanism"></a>Mecanismo de reintento

Service Bus implementa reintentos mediante implementaciones de la clase base [RetryPolicy](/dotnet/api/microsoft.servicebus.retrypolicy) . Todos los clientes de Service Bus exponen una propiedad **RetryPolicy** que puede establecerse en una de las implementaciones de la clase base **RetryPolicy**. Las implementaciones integradas son:

- La [clase RetryExponential](/dotnet/api/microsoft.servicebus.retryexponential). Esto expone las propiedades que controlan el intervalo de interrupción, el número de reintentos y la propiedad **TerminationTimeBuffer** que se utiliza para limitar el tiempo total para que se complete la operación.

- La [clase NoRetry](/dotnet/api/microsoft.servicebus.noretry). Se utiliza cuando los reintentos en el nivel de la API de Service Bus no son necesarios, como cuando otro proceso administra los reintentos como parte de una operación en lotes o de múltiples pasos.

Las acciones de Service Bus pueden devolver una amplia gama de excepciones, como se muestra en [Excepciones de mensajería de Service Bus](/azure/service-bus-messaging/service-bus-messaging-exceptions). La lista proporciona información sobre si estas indican que la operación de reintento es adecuada. Por ejemplo, un **ServerBusyException** indica que el cliente debe esperar durante un período de tiempo y, a continuación, volver a intentar la operación. La aparición de una **ServerBusyException** también hace que Service Bus cambie a un modo diferente, en el que se agrega un retraso adicional de 10 segundos a los retrasos de reintento calculados. Este modo se restablece tras un breve período.

Las excepciones devueltas de Service Bus exponen la propiedad **IsTransient** propiedad que indica si el cliente debería reintentar la operación. La directiva **RetryExponential** se basa en la propiedad **IsTransient** de la clase **MessagingException**, que es la clase base para todas las excepciones de Service Bus. Si crea implementaciones personalizadas de la clase de base **RetryPolicy**, podría usar una combinación del tipo de excepción y la propiedad **IsTransient** para proporcionar un control más fino sobre las acciones de reintento. Por ejemplo, se podría detectar una **QuotaExceededException** y tomar medidas para vaciar la cola antes de volver a intentar enviar un mensaje.

### <a name="policy-configuration"></a>Configuración de directivas

Las directivas de reintento se establecen mediante programación y se pueden establecer como una directiva predeterminada para un **NamespaceManager** y una **MessagingFactory**, o por separado para cada cliente de mensajería. Para establecer la directiva de reintentos predeterminada para una sesión de mensajería, establezca **RetryPolicy** de **NamespaceManager**.

```csharp
namespaceManager.Settings.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                                maxBackoff: TimeSpan.FromSeconds(30),
                                                                maxRetryCount: 3);
```

Para establecer la directiva de reintentos predeterminada para todos los clientes creada a partir de una fábrica de mensajería, establezca **RetryPolicy** de **MessagingFactory**.

```csharp
messagingFactory.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                    maxBackoff: TimeSpan.FromSeconds(30),
                                                    maxRetryCount: 3);
```

Para establecer la directiva de reintentos para un cliente de mensajería o para invalidar su directiva predeterminada, establezca su propiedad **RetryPolicy** mediante una instancia de la clase de directiva requerida:

```csharp
client.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                            maxBackoff: TimeSpan.FromSeconds(30),
                                            maxRetryCount: 3);
```

No se puede establecer la directiva de reintentos en el nivel de operación individual. Se aplica a todas las operaciones para el cliente de mensajería.
La siguiente tabla muestra la configuración predeterminada de la directiva de reintento integrada.

| Configuración | Valor predeterminado | Significado |
|---------|---------------|---------|
| Directiva | Exponencial | Retroceso exponencial. |
| MinimalBackoff | 0 | El intervalo de retroceso mínimo. Se agrega al intervalo de reintento calculado a partir de deltaBackoff. |
| MaximumBackoff | 30 segundos | El intervalo de retroceso máximo. MaximumBackoff se usa si el intervalo de reintento calculado es mayor que MaxBackoff. |
| DeltaBackoff | 3 segundos | Intervalo de espera entre reintentos. Se usará múltiplos de este período de tiempo para los intentos de reintentos posteriores. |
| TimeBuffer | 5 segundos | El búfer del tiempo de finalización asociado con el reintento. Si el tiempo restante es menor que TimeBuffer, se abandonan los intentos de reintento. |
| MaxRetryCount | 10 | El número máximo de reintentos. |
| ServerBusyBaseSleepTime | 10 segundos | Si la última excepción encontrada fue **ServerBusyException**, este valor se agregará al intervalo de reintento calculado. No se puede cambiar este valor. |

### <a name="retry-usage-guidance"></a>Instrucciones de uso del reintento

Cuando se usa Service Bus, tenga en cuenta las siguientes directrices:

- Al utilizar la implementación **RetryExponential** integrada, no implemente una operación de reserva, ya que la directiva reacciona a las excepciones de servidor ocupado y automáticamente cambia a un modo de reintentos adecuado.
- Service Bus admite una característica denominada espacios de nombres emparejados, que implementa la conmutación automática por error a una cola de copia de seguridad en un espacio de nombres independiente si se produce un error en la cola del espacio de nombres principal. Los mensajes de la cola secundaria pueden enviarse a la cola principal cuando se recupera. Esta característica ayuda a controlar los errores transitorios. Para obtener más información, consulte [Patrones de mensajería asincrónica y alta disponibilidad](/azure/service-bus-messaging/service-bus-async-messaging).

Considere la posibilidad de comenzar con la configuración siguiente para volver a intentar las operaciones. Esta es la configuración de propósito general, y debe supervisar las operaciones y ajustar los valores para adaptarlos a su propio escenario.

| Context | Latencia máxima de ejemplo | Directiva de reintentos | Configuración | Cómo funciona |
|---------|---------|---------|---------|---------|
| Interactivo, interfaz de usuario o primer plano | 2 segundos*  | Exponencial | MinimumBackoff = 0 <br/> MaximumBackoff = 30 s <br/> DeltaBackoff = 300 ms <br/> TimeBuffer = 300 ms <br/> MaxRetryCount = 2 | Intento 1: Retraso 0 s. <br/> Intento 2: Retraso ~300 ms <br/> Intento 3: Retraso ~900 ms |
| Segundo plano o lote | 30 segundos | Exponencial | MinimumBackoff = 1 <br/> MaximumBackoff = 30 s <br/> DeltaBackoff = 1,75 s <br/> TimeBuffer = 5 s <br/> MaxRetryCount = 3 | Intento 1: Retraso de ~1 segundo <br/> Intento 2: Retraso de ~3 segundos <br/> Intento 3: Retraso de ~6 ms <br/> Intento 4: Retraso ~13 ms |

\*Sin incluir el retraso adicional que se suma si se recibe una respuesta de servidor ocupado.

### <a name="telemetry"></a>Telemetría

Service Bus registra reintentos como eventos ETW mediante un **EventSource**. Debe asociar un **EventListener** al origen de eventos para capturar los eventos y verlos en el Visor de rendimiento o escribirlos en un registro de destino adecuado. Los eventos de reintento tienen la forma siguiente:

```text
Microsoft-ServiceBus-Client/RetryPolicyIteration
ThreadID="14,500"
FormattedMessage="[TrackingId:] RetryExponential: Operation Get:https://retry-tests.servicebus.windows.net/TestQueue/?api-version=2014-05 at iteration 0 is retrying after 00:00:00.1000000 sleep because of Microsoft.ServiceBus.Messaging.MessagingCommunicationException: The remote name could not be resolved: 'retry-tests.servicebus.windows.net'.TrackingId:6a26f99c-dc6d-422e-8565-f89fdd0d4fe3, TimeStamp:9/5/2014 10:00:13 PM."
trackingId=""
policyType="RetryExponential"
operation="Get:https://retry-tests.servicebus.windows.net/TestQueue/?api-version=2014-05"
iteration="0"
iterationSleep="00:00:00.1000000"
lastExceptionType="Microsoft.ServiceBus.Messaging.MessagingCommunicationException"
exceptionMessage="The remote name could not be resolved: 'retry-tests.servicebus.windows.net'.TrackingId:6a26f99c-dc6d-422e-8565-f89fdd0d4fe3,TimeStamp:9/5/2014 10:00:13 PM"
```

### <a name="examples"></a>Ejemplos

En el siguiente ejemplo de código se muestra cómo establecer la directiva de reintentos para:

- Un administrador de espacio de nombres. La directiva se aplica a todas las operaciones de ese administrador y no se puede reemplazar para las operaciones individuales.
- Una fábrica de mensajería. La directiva se aplica a todos los clientes que se crean a partir de ese generador y no se puede invalidar al crear clientes individuales.
- Un cliente de mensajería individual. Después de crear un cliente, puede establecer la directiva de reintentos para el cliente. La directiva se aplica a todas las operaciones de ese cliente.

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.ServiceBus;
using Microsoft.ServiceBus.Messaging;

namespace RetryCodeSamples
{
    class ServiceBusCodeSamples
    {
        private const string connectionString =
            @"Endpoint=sb://[my-namespace].servicebus.windows.net/;
                SharedAccessKeyName=RootManageSharedAccessKey;
                SharedAccessKey=C99..........Mk=";

        public async static Task Samples()
        {
            const string QueueName = "TestQueue";

            ServiceBusEnvironment.SystemConnectivity.Mode = ConnectivityMode.Http;

            var namespaceManager = NamespaceManager.CreateFromConnectionString(connectionString);

            // The namespace manager will have a default exponential policy with 10 retry attempts
            // and a 3 second delay delta.
            // Retry delays will be approximately 0 sec, 3 sec, 9 sec, 25 sec and the fixed 30 sec,
            // with an extra 10 sec added when receiving a ServiceBusyException.

            {
                // Set different values for the retry policy, used for all operations on the namespace manager.
                namespaceManager.Settings.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(0),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);

                // Policies cannot be specified on a per-operation basis.
                if (!await namespaceManager.QueueExistsAsync(QueueName))
                {
                    await namespaceManager.CreateQueueAsync(QueueName);
                }
            }

            var messagingFactory = MessagingFactory.Create(
                namespaceManager.Address, namespaceManager.Settings.TokenProvider);
            // The messaging factory will have a default exponential policy with 10 retry attempts
            // and a 3 second delay delta.
            // Retry delays will be approximately 0 sec, 3 sec, 9 sec, 25 sec and the fixed 30 sec,
            // with an extra 10 sec added when receiving a ServiceBusyException.

            {
                // Set different values for the retry policy, used for clients created from it.
                messagingFactory.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(1),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);


                // Policies cannot be specified on a per-operation basis.
                var session = await messagingFactory.AcceptMessageSessionAsync();
            }

            {
                var client = messagingFactory.CreateQueueClient(QueueName);
                // The client inherits the policy from the factory that created it.


                // Set different values for the retry policy on the client.
                client.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(0.1),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);

                // Policies cannot be specified on a per-operation basis.
                var session = await client.AcceptMessageSessionAsync();
            }
        }
    }
}
```

### <a name="more-information"></a>Más información

- [Patrones de mensajería asincrónica y alta disponibilidad](/azure/service-bus-messaging/service-bus-async-messaging)

## <a name="service-fabric"></a>Service Fabric

Distribuir servicios confiables en un clúster de Service Fabric protege frente a la mayoría de los errores transitorios posibles descritos en este artículo. Sin embargo, aún se pueden producir algunos errores transitorios. Por ejemplo, el servicio de nombres podría estar en medio de un cambio de enrutamiento cuando recibe una solicitud, lo que hace que se produzca una excepción. Si la misma solicitud llega 100 milisegundos después, probablemente se realizará correctamente.

Internamente, Service Fabric administra este tipo de errores transitorios. Puede configurar algunas opciones mediante la clase `OperationRetrySettings` al configurar los servicios. El código siguiente muestra un ejemplo. En la mayoría de los casos, esto no será necesario y la configuración predeterminada funcionará bien.

```csharp
FabricTransportRemotingSettings transportSettings = new FabricTransportRemotingSettings
{
    OperationTimeout = TimeSpan.FromSeconds(30)
};

var retrySettings = new OperationRetrySettings(TimeSpan.FromSeconds(15), TimeSpan.FromSeconds(1), 5);

var clientFactory = new FabricTransportServiceRemotingClientFactory(transportSettings);

var serviceProxyFactory = new ServiceProxyFactory((c) => clientFactory, retrySettings);

var client = serviceProxyFactory.CreateServiceProxy<ISomeService>(
    new Uri("fabric:/SomeApp/SomeStatefulReliableService"),
    new ServicePartitionKey(0));
```

### <a name="more-information"></a>Más información

- [Control remoto de excepciones](/azure/service-fabric/service-fabric-reliable-services-communication-remoting#remoting-exception-handling)

## <a name="sql-database-using-adonet"></a>SQL Database mediante ADO.NET

SQL Database es una instancia de SQL Database hospedada que está disponible en una amplia variedad de tamaños y como un servicio estándar (compartido) y premium (no compartido).

### <a name="retry-mechanism"></a>Mecanismo de reintento

SQL Database no tiene soporte integrado para reintentos cuando se tiene acceso mediante ADO.NET. Sin embargo, los códigos de retorno de solicitudes pueden usarse para determinar el motivo del error de una solicitud. Para más información acerca de la limitación de SQL Database, consulte [Límites de recursos de Azure SQL Database](/azure/sql-database/sql-database-resource-limits). Para obtener una lista de los códigos de error pertinentes, consulte [Códigos de error SQL para las aplicaciones de cliente de SQL Database](/azure/sql-database/sql-database-develop-error-messages).

Puede usar la biblioteca de Polly implementar reintentos para SQL Database. Consulte [Control de errores transitorios con Polly](#transient-fault-handling-with-polly).

### <a name="retry-usage-guidance"></a>Instrucciones de uso del reintento

Al obtener acceso a la SQL Database mediante ADO.NET, tenga en cuenta las siguientes directrices:

- Elija la opción de servicio adecuada (compartida o premium). Una instancia compartida puede sufrir retrasos en la conexión más largos de lo habitual y limitaciones debido al uso de otros inquilinos del servidor compartido. Si se requieren más operaciones de rendimiento predecible y de latencia baja confiable, considere la posibilidad de elegir la opción premium.
- Asegúrese de realizar reintentos al nivel o ámbito adecuado para evitar las operaciones no idempotentes que provocan incoherencias en los datos. Lo ideal es que todas las operaciones sean idempotentes para que puedan repetirse sin causar incoherencias. Si no es el caso, el reintento deberá realizarse en un nivel o ámbito que permita que todos los cambios relacionados se deshagan si se produce un error en una operación; por ejemplo, desde dentro de un ámbito transaccional. Para obtener más información, consulte [Capa de acceso a datos de fundamentos del servicio de nube: tratamiento de errores transitorios](https://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx#Idempotent_Guarantee).
- No se recomienda una estrategia de intervalo fijo para su uso con Azure SQL Database excepto para los escenarios interactivos donde hay solo unos cuantos reintentos a intervalos muy cortos. En su lugar, considere el uso de una estrategia de retroceso exponencial para la mayoría de escenarios.
- Elija un valor adecuado para los tiempos de espera de conexión y comando a la hora de definir las conexiones. Un tiempo de espera demasiado corto puede provocar errores prematuros en las conexiones cuando la base de datos está ocupada. Un tiempo de espera demasiado largo puede impedir que la lógica de reintento funcione correctamente esperando demasiado tiempo antes de detectar un error en la conexión. El valor de tiempo de espera es un componente de la latencia de extremo a extremo; se agrega eficazmente al intervalo entre reintentos especificado en la directiva de reintento para cada reintento.
- Cierre la conexión después de un cierto número de reintentos, incluso cuando se usa una lógica de reintentos de interrupción exponencial y vuelva a intentar la operación en una conexión nueva. Reintentar la misma operación varias veces en la misma conexión puede ser un factor que contribuya a ocasionar problemas de conexión. Para obtener un ejemplo de esta técnica, consulte [Capa de acceso a datos de fundamentos del servicio de nube: tratamiento de errores transitorios](https://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx).
- Cuando la agrupación de conexiones está en uso (valor predeterminado) es probable que se elija la misma conexión de la agrupación, incluso después de cerrar y volver a abrir una conexión. Si este es el caso, una técnica para resolverlo es llamar al método **ClearPool** de la clase **SqlConnection** para marcar la conexión como no reutilizable. Sin embargo, debería hacerlo solo después de que fallen varios intentos de conexión y solo al encontrar la clase específica de errores transitorios como tiempos de espera SQL (código de error -2) relacionados con conexiones erróneas.
- Si el código de acceso de datos usa las transacciones iniciadas como instancias **TransactionScope** , la lógica de reintento debe volver a abrir la conexión e iniciar un nuevo ámbito de transacción. Por este motivo, el bloque de código que se puede reintentar debe abarcar todo el ámbito de la transacción.

Considere la posibilidad de comenzar con la configuración siguiente para volver a intentar las operaciones. Esta es la configuración de propósito general, y debe supervisar las operaciones y ajustar los valores para adaptarlos a su propio escenario.

| **Contexto** | **Destino de ejemplo E2E<br />latencia máxima** | **Estrategia de reintento** | **Configuración** | **Valores** | **Cómo funciona** |
| --- | --- | --- | --- | --- | --- |
| Interactivo, interfaz de usuario<br />o primer plano |2 segundos |FixedInterval |Número de reintentos<br />Intervalo de reintento<br />Primer reintento rápido |3<br />500 ms<br />true |Intento 1 - retraso de 0 segundos<br />Intento 2 - retraso de 500 ms<br />Intento 3 – retraso de 500 ms |
| Fondo<br />o proceso por lotes |30 segundos |ExponentialBackoff |Número de reintentos<br />Interrupción mínima<br />Interrupción máxima<br />Interrupción delta<br />Primer reintento rápido |5<br />0 segundos<br />60 segundos<br />2 segundos<br />false |Intento 1 - retraso de 0 segundos<br />Intento 2 - retraso de ~2 segundos<br />Intento 3 - retraso de ~6 segundos<br />Intento 4 - retraso de ~14 segundos<br />Intento 5 - retraso de ~30 segundos |

> [!NOTE]
> Los destinos de latencia de extremo a extremo suponen el tiempo de espera predeterminado para las conexiones con el servicio. Si especifica tiempos de espera de conexión más largos, la latencia de extremo a extremo se extenderá este tiempo adicional en cada reintento.

### <a name="examples"></a>Ejemplos

En esta sección se muestra cómo usar Polly para acceder a Azure SQL Database mediante un conjunto de directivas de reintento configurado en la clase `Policy`.

El código siguiente muestra un método de extensión en la clase `SqlCommand` que llama a `ExecuteAsync` con retroceso exponencial.

```csharp
public async static Task<SqlDataReader> ExecuteReaderWithRetryAsync(this SqlCommand command)
{
    GuardConnectionIsNotNull(command);

    var policy = Policy.Handle<Exception>().WaitAndRetryAsync(
        retryCount: 3, // Retry 3 times
        sleepDurationProvider: attempt => TimeSpan.FromMilliseconds(200 * Math.Pow(2, attempt - 1)), // Exponential backoff based on an initial 200ms delay.
        onRetry: (exception, attempt) =>
        {
            // Capture some info for logging/telemetry.
            logger.LogWarn($"ExecuteReaderWithRetryAsync: Retry {attempt} due to {exception}.");
        });

    // Retry the following call according to the policy.
    await policy.ExecuteAsync<SqlDataReader>(async token =>
    {
        // This code is executed within the Policy

        if (conn.State != System.Data.ConnectionState.Open) await conn.OpenAsync(token);
        return await command.ExecuteReaderAsync(System.Data.CommandBehavior.Default, token);

    }, cancellationToken);
}
```

Este método de extensión asincrónico puede utilizarse como se indica a continuación.

```csharp
var sqlCommand = sqlConnection.CreateCommand();
sqlCommand.CommandText = "[some query]";

using (var reader = await sqlCommand.ExecuteReaderWithRetryAsync())
{
    // Do something with the values
}
```

### <a name="more-information"></a>Más información

- [Capa de acceso a datos de fundamentos del servicio de nube: tratamiento de errores transitorios](https://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx)

Para obtener instrucciones generales sobre cómo sacar el máximo partido de SQL Database, consulte [Azure SQL Database Performance and Elasticity Guide](https://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx) (Guía de elasticidad y rendimiento de base de datos de SQL Azure).

## <a name="sql-database-using-entity-framework-6"></a>SQL Database mediante Entity Framework 6

SQL Database es una instancia de SQL Database hospedada que está disponible en una amplia variedad de tamaños y como un servicio estándar (compartido) y premium (no compartido). Entity Framework es un mapeador relacional de objetos que permite a los desarrolladores de .NET trabajar con datos relacionales usando objetos específicos del dominio. Elimina la necesidad de usar la mayoría del código de acceso a datos que los programadores suelen tener que escribir.

### <a name="retry-mechanism"></a>Mecanismo de reintento

Se proporciona compatibilidad con los reintentos al obtener acceso a la instancia de SQL Database mediante Entity Framework 6.0 y versiones posteriores mediante un mecanismo denominado [resistencia de la conexión y lógica de reintento](/ef/ef6/fundamentals/connection-resiliency/retry-logic). Las características principales del mecanismo de reintento son:

- La abstracción principal es la interfaz **IDbExecutionStrategy** . Esta interfaz:
  - Define métodos **Execute*** sincrónicos y asincrónicos.
  - Define las clases que pueden usarse o configurarse directamente en un contexto de base de datos como una estrategia predeterminada, asignada al nombre de un proveedor o asignada a un nombre de proveedor y de servidor. Cuando se configura en un contexto, los reintentos se producen en el nivel de operaciones de base de datos individual, de las que podría haber varias para una operación de contexto especificada.
  - Define cuándo se debe reintentar una conexión fallida y cómo.
- Incluye varias implementaciones integradas de la interfaz **IDbExecutionStrategy** :
  - Predeterminado: ningún reintento.
  - Valor predeterminado para la SQL Database (automático): sin reintento, pero inspecciona las excepciones y las ajusta con la recomendación de usar la estrategia de SQL Database.
  - Predeterminado para la SQL Database: exponencial (heredado de la clase base) más la lógica de detección de la SQL Database.
- Implementa una estrategia de retroceso exponencial que incluye la selección aleatoria.
- Las clases de reintento integradas tienen estado y no están protegidas para subprocesos. Sin embargo, se pueden volver a usar una vez completada la operación actual.
- Si se supera el número de reintentos especificado, los resultados se encapsulan en una nueva excepción. No se propaga la excepción actual.

### <a name="policy-configuration"></a>Configuración de directivas

Al obtener acceso a SQL Database mediante Entity Framework 6.0 y versiones posteriores, se proporciona compatibilidad con los reintentos. Las directivas de reintento se configuran mediante programación. No se puede cambiar la configuración en cada operación.

Al configurar una estrategia en el contexto como valor predeterminado, especifique una función que cree una nueva estrategia a petición. El código siguiente muestra cómo puede crear una clase de configuración de reintento que amplíe la clase base **DbConfiguration** .

```csharp
public class BloggingContextConfiguration : DbConfiguration
{
  public BlogConfiguration()
  {
    // Set up the execution strategy for SQL Database (exponential) with 5 retries and 4 sec delay
    this.SetExecutionStrategy(
         "System.Data.SqlClient", () => new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(4)));
  }
}
```

A continuación, puede especificar esto como estrategia de reintento predeterminada para todas las operaciones mediante el método **SetConfiguration** de la instancia **DbConfiguration** cuando se inicia la aplicación. De forma predeterminada, EF detectará y usará automáticamente la clase de configuración.

```csharp
DbConfiguration.SetConfiguration(new BloggingContextConfiguration());
```

Puede especificar la clase de configuración de reintento para un contexto anotando la clase de contexto con un atributo **DbConfigurationType** . Sin embargo, si solo tiene una clase de configuración, EF la usará sin necesidad de anotar el contexto.

```csharp
[DbConfigurationType(typeof(BloggingContextConfiguration))]
public class BloggingContext : DbContext
```

Si necesita usar estrategias de reintento diferentes para operaciones específicas o deshabilitar reintentos para operaciones específicas, puede crear una clase de configuración que permita suspender o intercambiar estrategias estableciendo un indicador en **CallContext**. La clase de configuración puede usar este indicador para cambiar las estrategias o deshabilitar la estrategia proporcionada por usted y usar una estrategia predeterminada. Para más información, vea [Suspenda la estrategia de ejecución](/ef/ef6/fundamentals/connection-resiliency/retry-logic#workaround-suspend-execution-strategy) (EF6 y versiones posteriores).

Otra técnica para el uso de estrategias de reintento específica para las operaciones individuales es crear una instancia de la clase de estrategia necesaria y proporcionar la configuración deseada a través de parámetros. A continuación, invoque su método **ExecuteAsync** .

```csharp
var executionStrategy = new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(4));
var blogs = await executionStrategy.ExecuteAsync(
    async () =>
    {
        using (var db = new BloggingContext("Blogs"))
        {
            // Acquire some values asynchronously and return them
        }
    },
    new CancellationToken()
);
```

La manera más sencilla de usar una clase **DbConfiguration** es buscarla en el mismo ensamblado que la clase **DbContext**. Sin embargo, esto no es adecuado cuando se requiere el mismo contexto en diferentes escenarios, como diferentes estrategias de reintento interactivo y en segundo plano. Si los contextos diferentes se ejecutan en dominios de aplicación independientes, puede usar la compatibilidad integrada para especificar clases de configuración en el archivo de configuración o establecerla explícitamente mediante código. Si se deben ejecutar los contextos diferentes en el mismo dominio de aplicación, se requerirá una solución personalizada.

Para más información, vea [Configuración basada en código](/ef/ef6/fundamentals/configuring/code-based) (EF6 y versiones posteriores).

La siguiente tabla muestra la configuración predeterminada de las directivas de reintento integradas cuando se usa EF6.

| Configuración | Valor predeterminado | Significado |
|---------|---------------|---------|
| Directiva | Exponencial | Retroceso exponencial. |
| MaxRetryCount | 5 | El número máximo de reintentos. |
| MaxDelay | 30 segundos | El retraso máximo entre los reintentos. Este valor no afecta al modo en que se calcula la serie de retrasos. Solo define un límite superior. |
| DefaultCoefficient | 1 segundo | El coeficiente para el cálculo del retroceso exponencial. No se puede cambiar este valor. |
| DefaultRandomFactor | 1.1 | El multiplicador utilizado para agregar un retraso aleatorio para cada entrada. No se puede cambiar este valor. |
| DefaultExponentialBase | 2 | El multiplicador utilizado para calcular el retraso siguiente. No se puede cambiar este valor. |

### <a name="retry-usage-guidance"></a>Instrucciones de uso del reintento

Al obtener acceso a la SQL Database mediante EF6, tenga en cuenta las siguientes directrices:

- Elija la opción de servicio adecuada (compartida o premium). Una instancia compartida puede sufrir retrasos en la conexión más largos de lo habitual y limitaciones debido al uso de otros inquilinos del servidor compartido. Si se requieren un rendimiento predecible y operaciones de latencia baja confiable, considere la posibilidad de elegir la opción premium.

- No se recomienda una estrategia de intervalo fijo para su uso con Azure SQL Database. En su lugar, use una estrategia de retroceso exponencial debido a que el servicio esté sobrecargado, y retrasos más prolongados permiten un mayor tiempo para la recuperación.

- Elija un valor adecuado para los tiempos de espera de conexión y comando a la hora de definir las conexiones. Base el tiempo de espera en el diseño de la lógica de negocios y a través de pruebas. Puede que necesite modificar este valor con el tiempo a medida que los volúmenes de datos o los procesos empresariales cambien. Un tiempo de espera demasiado corto puede provocar errores prematuros en las conexiones cuando la base de datos está ocupada. Un tiempo de espera demasiado largo puede impedir que la lógica de reintento funcione correctamente esperando demasiado tiempo antes de detectar un error en la conexión. El valor de tiempo de espera es un componente de la latencia de extremo a extremo, aunque no podrá determinar fácilmente cuántos comandos se ejecutarán cuando se guarde el contexto. Puede cambiar el tiempo de espera predeterminado estableciendo la propiedad **CommandTimeout** de la instancia **DbContext**.

- Entity Framework admite configuraciones de reintento definidas en archivos de configuración. Sin embargo, para obtener la máxima flexibilidad en Azure puede crear la configuración mediante programación dentro de la aplicación. Los parámetros específicos para las directivas de reintento, como el número de reintentos y los intervalos de reintento, se pueden almacenar en el archivo de configuración del servicio y usar en tiempo de ejecución para crear las directivas apropiadas. Esto permite cambiar la configuración sin necesidad de que la aplicación se reinicie.

Considere la posibilidad de comenzar con la configuración siguiente para las operaciones de reintento. No se puede especificar el retraso entre reintentos (es fijo y se genera como una secuencia exponencial). Puede especificar solo los valores máximos, como se muestra aquí, a menos que cree una estrategia de reintento personalizada. Esta es la configuración de propósito general, y debe supervisar las operaciones y ajustar los valores para adaptarlos a su propio escenario.

| **Contexto** | **Destino de ejemplo E2E<br />latencia máxima** | **Directiva de reintentos** | **Configuración** | **Valores** | **Cómo funciona** |
| --- | --- | --- | --- | --- | --- |
| Interactivo, interfaz de usuario<br />o primer plano |2 segundos |Exponencial |MaxRetryCount<br />MaxDelay |3<br />750 ms |Intento 1 - retraso de 0 segundos<br />Intento 2 - retraso de 750 ms<br />Intento 3 – retraso de 750 ms |
| Fondo<br /> o proceso por lotes |30 segundos |Exponencial |MaxRetryCount<br />MaxDelay |5<br />12 segundos |Intento 1 - retraso de 0 segundos<br />Intento 2 - retraso de ~1 segundos<br />Intento 3 - retraso de ~3 segundos<br />Intento 4 - retraso de ~7 segundos<br />Intento 5 - retraso de 12 segundos |

> [!NOTE]
> Los destinos de latencia de extremo a extremo suponen el tiempo de espera predeterminado para las conexiones con el servicio. Si especifica tiempos de espera de conexión más largos, la latencia de extremo a extremo se extenderá este tiempo adicional en cada reintento.

### <a name="examples"></a>Ejemplos

En el ejemplo de código siguiente se define una solución de acceso de datos simple que utiliza Entity Framework. Establece una estrategia de reintento específica mediante la definición de una instancia de una clase denominada **BlogConfiguration** que extiende **DbConfiguration**.

```csharp
using System;
using System.Collections.Generic;
using System.Data.Entity;
using System.Data.Entity.SqlServer;
using System.Threading.Tasks;

namespace RetryCodeSamples
{
    public class BlogConfiguration : DbConfiguration
    {
        public BlogConfiguration()
        {
            // Set up the execution strategy for SQL Database (exponential) with 5 retries and 12 sec delay.
            // These values could be loaded from configuration rather than being hard-coded.
            this.SetExecutionStrategy(
                    "System.Data.SqlClient", () => new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(12)));
        }
    }

    // Specify the configuration type if more than one has been defined.
    // [DbConfigurationType(typeof(BlogConfiguration))]
    public class BloggingContext : DbContext
    {
        // Definition of content goes here.
    }

    class EF6CodeSamples
    {
        public async static Task Samples()
        {
            // Execution strategy configured by DbConfiguration subclass, discovered automatically or
            // or explicitly indicated through configuration or with an attribute. Default is no retries.
            using (var db = new BloggingContext("Blogs"))
            {
                // Add, edit, delete blog items here, then:
                await db.SaveChangesAsync();
            }
        }
    }
}
```

Más ejemplos de uso del mecanismo de reintento de Entity Framework se pueden encontrar en [Resistencia de la conexión/lógica de reintento](/ef/ef6/fundamentals/connection-resiliency/retry-logic).

### <a name="more-information"></a>Más información

- [Azure SQL Database Performance and Elasticity Guide](https://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx) (Guía de elasticidad y rendimiento de base de datos de SQL Azure)

## <a name="sql-database-using-entity-framework-core"></a>SQL Database mediante Entity Framework Core

[Entity Framework Core](/ef/core/) es un mapeador relacional de objetos que permite a los desarrolladores de .NET trabajar con datos usando objetos específicos del dominio. Elimina la necesidad de usar la mayoría del código de acceso a datos que los programadores suelen tener que escribir. Esta versión de Entity Framework se escribió desde el principio y no hereda automáticamente todas las características de EF6.x.

### <a name="retry-mechanism"></a>Mecanismo de reintento

La compatibilidad con los reintentos se proporciona al obtener acceso a SQL Database mediante Entity Framework Core mediante un mecanismo denominado [resistencia de la conexión](/ef/core/miscellaneous/connection-resiliency). La resistencia de la conexión se incorporó por primera vez en EF Core 1.1.0.

La abstracción principal es la interfaz `IExecutionStrategy`. La estrategia de ejecución para SQL Server, incluido SQL Azure, conoce los tipos de excepción que se pueden recuperar y tiene valores predeterminados razonables para el número máximo de reintentos o el retraso entre los reintentos, entre otros.

### <a name="examples"></a>Ejemplos

El código siguiente permite reintentos automáticos cuando se configura el objeto DbContext, que representa una sesión con la base de datos.

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
        .UseSqlServer(
            @"Server=(localdb)\mssqllocaldb;Database=EFMiscellanous.ConnectionResiliency;Trusted_Connection=True;",
            options => options.EnableRetryOnFailure());
}
```

El código siguiente muestra cómo ejecutar una transacción con reintentos automáticos, mediante el uso de una estrategia de ejecución. La transacción se define en un delegado. Si se produce un error transitorio, la estrategia de ejecución invoca al delegado de nuevo.

```csharp
using (var db = new BloggingContext())
{
    var strategy = db.Database.CreateExecutionStrategy();

    strategy.Execute(() =>
    {
        using (var transaction = db.Database.BeginTransaction())
        {
            db.Blogs.Add(new Blog { Url = "https://blogs.msdn.com/dotnet" });
            db.SaveChanges();

            db.Blogs.Add(new Blog { Url = "https://blogs.msdn.com/visualstudio" });
            db.SaveChanges();

            transaction.Commit();
        }
    });
}
```

## <a name="azure-storage"></a>Azure Storage

Los servicios de Azure Storage incluyen almacenamiento de tablas y blobs, archivos y colas de almacenamiento.

### <a name="retry-mechanism"></a>Mecanismo de reintento

Los reintentos se producen en el nivel de operación REST individual y son parte integral de la implementación de la API de cliente. El SDK de almacenamiento de cliente usa clases que implementan la [Interfaz IExtendedRetryPolicy](/dotnet/api/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy).

Hay diferentes implementaciones de la interfaz. Los clientes de almacenamiento pueden elegir entre directivas diseñadas específicamente para el acceso a tablas, blobs y colas. Cada implementación utiliza una estrategia de reintento diferente que define el intervalo de reintento y otros detalles.

Las clases integradas proporcionan compatibilidad para intervalos de reintento lineales (retraso constante) y exponenciales de selección aleatoria. También hay una directiva de no realización de reintentos a usar cuando otro proceso está controlando los reintentos a un nivel superior. Sin embargo, puede implementar sus propias clases de reintentos si tiene requisitos específicos no proporcionados por las clases integradas.

Los reintentos alternativos cambian entre ubicación del servicio de almacenamiento principal y secundario si usa almacenamiento con redundancia geográfica con acceso de lectura (RA-GRS) y el resultado de la solicitud es un error que se puede reproducir. Para obtener más información, consulte [Opciones de redundancia de Azure Storage](/azure/storage/common/storage-redundancy) .

### <a name="policy-configuration"></a>Configuración de directivas

Las directivas de reintento se configuran mediante programación. Un procedimiento típico consiste en crear y rellenar una instancia **TableRequestOptions**, **BlobRequestOptions**, **FileRequestOptions** o **QueueRequestOptions** instancia.

```csharp
TableRequestOptions interactiveRequestOption = new TableRequestOptions()
{
  RetryPolicy = new LinearRetry(TimeSpan.FromMilliseconds(500), 3),
  // For Read-access geo-redundant storage, use PrimaryThenSecondary.
  // Otherwise set this to PrimaryOnly.
  LocationMode = LocationMode.PrimaryThenSecondary,
  // Maximum execution time based on the business use case.
  MaximumExecutionTime = TimeSpan.FromSeconds(2)
};
```

A continuación, la instancia de opciones de solicitud se puede establecer en el cliente y todas las operaciones con el cliente usarán las opciones de solicitud especificadas.

```csharp
client.DefaultRequestOptions = interactiveRequestOption;
var stats = await client.GetServiceStatsAsync();
```

Puede anular las opciones de solicitud de cliente pasando una instancia completada de la clase de opciones de solicitud como parámetro a los métodos de operación.

```csharp
var stats = await client.GetServiceStatsAsync(interactiveRequestOption, operationContext: null);
```

Puede usar una instancia **OperationContext** para especificar el código para ejecutar cuando se produce un reintento y cuando se ha completado una operación. Este código puede recopilar información acerca de la operación para su uso en registros y telemetría.

```csharp
// Set up notifications for an operation
var context = new OperationContext();
context.ClientRequestID = "some request id";
context.Retrying += (sender, args) =>
{
    /* Collect retry information */
};
context.RequestCompleted += (sender, args) =>
{
    /* Collect operation completion information */
};
var stats = await client.GetServiceStatsAsync(null, context);
```

Además de indicar si se puede volver a intentar un error, las directivas de reintento ampliadas devuelven un objeto **RetryContext** que indica el número de reintentos, los resultados de la última solicitud y si se realizará el siguiente reintento en la ubicación principal o secundaria (consulte la tabla siguiente para obtener más información). Las propiedades del objeto **RetryContext** pueden usarse para decidir cuándo y si realizar un reintento. Para obtener más información, consulte [Método IExtendedRetryPolicy.Evaluate](/dotnet/api/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.evaluate).

La siguiente tabla muestra la configuración predeterminada de las directivas de reintento integradas.

**Opciones de solicitud:**

| **Configuración** | **Valor predeterminado** | **Significado** |
| --- | --- | --- |
| MaximumExecutionTime | None | Tiempo de ejecución máximo para la solicitud, incluidos todos los posibles reintentos. Si no se especifica, el periodo que se permite a una solicitud es ilimitado. En otras palabras, la solicitud podría dejar de responder. |
| ServerTimeout | None | Intervalo de tiempo de espera del servidor para la solicitud (el valor se redondea en segundos). Si no se especifica, usará el valor predeterminado para todas las solicitudes al servidor. Normalmente, la mejor opción es omitir este valor para que se utilice la configuración predeterminada del servidor. |
| LocationMode | None | Si la cuenta de almacenamiento se crea con la opción de replicación del almacenamiento con redundancia geográfica de acceso de lectura (RA-GRS), puede usar el modo de ubicación para indicar qué ubicación debe recibir la solicitud. Por ejemplo, si se especifica **PrimaryThenSecondary** , las solicitudes siempre se envían a la ubicación principal en primer lugar. Si se produce un error en una solicitud, se envía a la ubicación secundaria. |
| RetryPolicy | ExponentialPolicy | Consulte a continuación los detalles de cada opción. |

**Directiva exponencial:**

| **Configuración** | **Valor predeterminado** | **Significado** |
| --- | --- | --- |
| maxAttempt | 3 | Número de reintentos. |
| deltaBackoff | 4 segundos | Intervalo de espera entre reintentos. Se usarán múltiplos de este período de tiempo, incluyendo un elemento aleatorio, para los reintentos posteriores. |
| MinBackoff | 3 segundos | Agregado a todos los intervalos de reintento calculados a partir de deltaBackoff. No se puede cambiar este valor.
| MaxBackoff | 120 segundos | MaxBackoff se usa si el intervalo de reintento calculado es mayor que MaxBackoff. No se puede cambiar este valor. |

**Directiva lineal:**

| **Configuración** | **Valor predeterminado** | **Significado** |
| --- | --- | --- |
| maxAttempt | 3 | Número de reintentos. |
| deltaBackoff | 30 segundos | Intervalo de espera entre reintentos. |

### <a name="retry-usage-guidance"></a>Instrucciones de uso del reintento

Al obtener acceso a los servicios de almacenamiento de Azure mediante la API de cliente de almacenamiento, tenga en cuenta las siguientes directrices:

- Use las directivas de reintento integradas del espacio de nombres Microsoft.WindowsAzure.Storage.RetryPolicies donde son adecuadas para sus requisitos. En la mayoría de los casos, estas directivas serán suficientes.

- Use la directiva **ExponentialRetry** en operaciones por lotes, tareas en segundo plano o escenarios no interactivos. En estos escenarios, normalmente puede permitir más tiempo para recuperar el servicio (por consiguiente, con unas mayores posibilidades de que la operación se efectúe correctamente).

- Considere la posibilidad de especificar la propiedad **MaximumExecutionTime** del parámetro **RequestOptions** para limitar el tiempo de ejecución total, pero tenga en cuenta el tipo y tamaño de la operación al elegir un valor de tiempo de espera.

- Si necesita implementar un reintento personalizado, evite crear contenedores en torno a las clases de cliente de almacenamiento. En su lugar, use las capacidades para ampliar las directivas existentes a través de la interfaz **IExtendedRetryPolicy** .

- Si utiliza almacenamiento con redundancia geográfica con acceso de lectura (RA-GRS) puede usar el **LocationMode** para especificar que los reintentos tengan acceso a la copia de solo lectura secundaria de la tienda en caso de error en el acceso principal. Sin embargo, al utilizar esta opción debe asegurarse de que la aplicación pueda trabajar correctamente con datos que pueden ser obsoletos si todavía no ha completado la replicación desde el almacén principal.

Considere la posibilidad de comenzar con la configuración siguiente para volver a intentar las operaciones. Esta es la configuración de propósito general, y debe supervisar las operaciones y ajustar los valores para adaptarlos a su propio escenario.

| **Contexto** | **Destino de ejemplo E2E<br />latencia máxima** | **Directiva de reintentos** | **Configuración** | **Valores** | **Cómo funciona** |
| --- | --- | --- | --- | --- | --- |
| Interactivo, interfaz de usuario<br />o primer plano |2 segundos |Lineal |maxAttempt<br />deltaBackoff |3<br />500 ms |Intento 1 - retraso de 500 segundos<br />Intento 2 - retraso de 500 ms<br />Intento 3 – retraso de 500 ms |
| Fondo<br />o proceso por lotes |30 segundos |Exponencial |maxAttempt<br />deltaBackoff |5<br />4 segundos |Intento 1 - retraso de ~3 segundos<br />Intento 2 - retraso de ~7 segundos<br />Intento 3 - retraso de ~15 segundos |

### <a name="telemetry"></a>Telemetría

Los reintentos se registran en un **TraceSource**. Debe configurar un **TraceListener** para capturar los eventos y escribirlos en un registro de destino adecuado. Puede usar el **TextWriterTraceListener** o **XmlWriterTraceListener** para escribir los datos en un archivo de registro, el **EventLogTraceListener** para escribir en el registro de eventos de Windows o el **EventProviderTraceListener** para escribir los datos de seguimiento en el subsistema ETW. También puede configurar automáticamente el vaciado del búfer y el nivel de detalle de los eventos que se registrarán (por ejemplo, Error, Advertencia, Informativo y Detallado). Para obtener más información, consulte [Registro de cliente con biblioteca de cliente de almacenamiento .NET](/rest/api/storageservices/Client-side-Logging-with-the-.NET-Storage-Client-Library).

Las operaciones pueden recibir una instancia **OperationContext**, que expone un evento de **Reintento** que puede usarse para adjuntar la lógica personalizada de telemetría. Para obtener más información, consulte [Evento OperationContext.Retrying](/dotnet/api/microsoft.windowsazure.storage.operationcontext.retrying).

### <a name="examples"></a>Ejemplos

En el ejemplo de código siguiente se muestra cómo crear dos instancias **TableRequestOptions** con diferentes configuraciones de reintento; una para solicitudes interactivas y otra para solicitudes en segundo plano. A continuación, el ejemplo establece estas dos directivas de reintento en el cliente para que puedan aplicarse a todas las solicitudes y también establece la estrategia interactiva en una solicitud concreta para que reemplace la configuración predeterminada que se aplica al cliente.

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.WindowsAzure.Storage;
using Microsoft.WindowsAzure.Storage.RetryPolicies;
using Microsoft.WindowsAzure.Storage.Table;

namespace RetryCodeSamples
{
    class AzureStorageCodeSamples
    {
        private const string connectionString = "UseDevelopmentStorage=true";

        public async static Task Samples()
        {
            var storageAccount = CloudStorageAccount.Parse(connectionString);

            TableRequestOptions interactiveRequestOption = new TableRequestOptions()
            {
                RetryPolicy = new LinearRetry(TimeSpan.FromMilliseconds(500), 3),
                // For Read-access geo-redundant storage, use PrimaryThenSecondary.
                // Otherwise set this to PrimaryOnly.
                LocationMode = LocationMode.PrimaryThenSecondary,
                // Maximum execution time based on the business use case.
                MaximumExecutionTime = TimeSpan.FromSeconds(2)
            };

            TableRequestOptions backgroundRequestOption = new TableRequestOptions()
            {
                // Client has a default exponential retry policy with 4 sec delay and 3 retry attempts
                // Retry delays will be approximately 3 sec, 7 sec, and 15 sec
                MaximumExecutionTime = TimeSpan.FromSeconds(30),
                // PrimaryThenSecondary in case of Read-access geo-redundant storage, else set this to PrimaryOnly
                LocationMode = LocationMode.PrimaryThenSecondary
            };

            var client = storageAccount.CreateCloudTableClient();
            // Client has a default exponential retry policy with 4 sec delay and 3 retry attempts
            // Retry delays will be approximately 3 sec, 7 sec, and 15 sec
            // ServerTimeout and MaximumExecutionTime are not set

            {
                // Set properties for the client (used on all requests unless overridden)
                // Different exponential policy parameters for background scenarios
                client.DefaultRequestOptions = backgroundRequestOption;
                // Linear policy for interactive scenarios
                client.DefaultRequestOptions = interactiveRequestOption;
            }

            {
                // set properties for a specific request
                var stats = await client.GetServiceStatsAsync(interactiveRequestOption, operationContext: null);
            }

            {
                // Set up notifications for an operation
                var context = new OperationContext();
                context.ClientRequestID = "some request id";
                context.Retrying += (sender, args) =>
                {
                    /* Collect retry information */
                };
                context.RequestCompleted += (sender, args) =>
                {
                    /* Collect operation completion information */
                };
                var stats = await client.GetServiceStatsAsync(null, context);
            }
        }
    }
}
```

### <a name="more-information"></a>Más información

- [Recomendaciones de la directiva de reintento de biblioteca cliente de Azure Storage](https://azure.microsoft.com/blog/2014/05/22/azure-storage-client-library-retry-policy-recommendations/)

- [Biblioteca cliente de almacenamiento 2.0: implementación de directivas de reintento](https://gauravmantri.com/2012/12/30/storage-client-library-2-0-implementing-retry-policies/)

## <a name="general-rest-and-retry-guidelines"></a>Directrices generales de REST y de reintento

Al obtener acceso a los servicios de Azure o de terceros, tenga en cuenta lo siguiente:

- Use un enfoque sistemático para administrar reintentos, quizás como código reutilizable, para que pueda aplicar una metodología coherente en todos los clientes y todas las soluciones.

- Considere la posibilidad de usar una plataforma de reintentos, como [Polly][polly] para administrar los reintentos si el servicio de destino o el cliente no tiene ningún mecanismo de reintentos integrado. Esto le ayudará a implementar un comportamiento de reintento coherente y puede proporcionar una estrategia de reintento predeterminada adecuados para el servicio de destino. Sin embargo, puede que necesite crear código de reintento personalizado para los servicios que tienen un comportamiento no estándar, que no dependen de excepciones para indicar errores transitorios o si desea usar una respuesta de **respuesta de reintento** para administrar el comportamiento de reintento.

- La lógica de detección transitoria dependerá de la API de cliente real que use para invocar las llamadas REST. Algunos clientes, como los de la clase **HttpClient** más reciente no producirán excepciones para las solicitudes completadas con un código de estado HTTP no correcto.

- El código de estado HTTP devuelto desde el servicio puede ayudar a indicar si el error es transitorio. Puede que necesite examinar las excepciones generadas por un cliente o el marco de trabajo de reintento para obtener acceso al código de estado o para determinar el tipo de excepción equivalente. Los siguientes códigos HTTP normalmente indican que un reintento es adecuado:
  
  - Tiempo de espera de solicitud 408
  - 429 Demasiadas solicitudes
  - Error de servidor interno 500
  - Puerta de enlace incorrecta 502
  - Servicio no disponible 503
  - Tiempo de espera de puerta de enlace 504

- Si la lógica de reintento se basa en excepciones, las siguientes suelen indican un error transitorio donde no se pudo establecer ninguna conexión:

  - WebExceptionStatus.ConnectionClosed
  - WebExceptionStatus.ConnectFailure
  - WebExceptionStatus.Timeout
  - WebExceptionStatus.RequestCanceled

- En el caso de un estado no disponible del servicio, el servicio podría indicar el retraso adecuado antes de reintentar en el encabezado de respuesta **Retry-After** o un encabezado personalizado diferente. Los servicios también pueden enviar información adicional como encabezados personalizados o incrustados en el contenido de la respuesta.

- No intente de nuevo los códigos de estado que representan errores de cliente (errores en el intervalo 4xx) excepto un Tiempo de espera de solicitud 408.

- Pruebe las estrategias y mecanismos de reintento a fondo en una amplia variedad de condiciones, como diferentes estados de red diferente y diferentes cargas de sistema.

### <a name="retry-strategies"></a>Estrategia de reintento

Estos son los tipos típicos de intervalos de estrategia de reintento:

- **Exponencial**. Una directiva de reintentos que realiza un número especificado de reintentos, mediante un enfoque de retroceso exponencial aleatorio para determinar el intervalo entre reintentos. Por ejemplo: 

    ```csharp
    var random = new Random();

    var delta = (int)((Math.Pow(2.0, currentRetryCount) - 1.0) *
                random.Next((int)(this.deltaBackoff.TotalMilliseconds * 0.8),
                (int)(this.deltaBackoff.TotalMilliseconds * 1.2)));
    var interval = (int)Math.Min(checked(this.minBackoff.TotalMilliseconds + delta),
                    this.maxBackoff.TotalMilliseconds);
    retryInterval = TimeSpan.FromMilliseconds(interval);
    ```

- **Incremental**. Una estrategia de reintentos con un número especificado de reintentos y un intervalo de tiempo incremental entre los reintentos. Por ejemplo: 

    ```csharp
    retryInterval = TimeSpan.FromMilliseconds(this.initialInterval.TotalMilliseconds +
                    (this.increment.TotalMilliseconds * currentRetryCount));
    ```

- **LinearRetry**. Una directiva de reintentos que realiza un número especificado de reintentos, usando un intervalo de tiempo fijo especificado entre los reintentos. Por ejemplo: 

    ```csharp
    retryInterval = this.deltaBackoff;
    ```

### <a name="transient-fault-handling-with-polly"></a>Consulte Control de errores transitorios con Polly.

[Polly][polly] es una biblioteca para controlar mediante programación los reintentos y las estrategias de [disyuntor](../patterns/circuit-breaker.md). El proyecto Polly es miembro de [.NET Foundation][dotnet-foundation]. Para los servicios en los que el cliente no admite los reintentos de forma nativa, Polly es una alternativa válida y evita tener que escribir código de reintentos personalizado, que puede ser difícil de implementar correctamente. Polly también permite el seguimiento de los errores cuando se produzcan, para poder registrar los reintentos.

### <a name="more-information"></a>Más información

- [Resistencia de la conexión](/ef/core/miscellaneous/connection-resiliency)
- [Puntos de datos - EF Core 1.1](https://msdn.microsoft.com/magazine/mt745093.aspx)

<!-- links -->

[adal]: /azure/active-directory/develop/active-directory-authentication-libraries
[autorest]: https://github.com/Azure/autorest/tree/master/docs
[ConnectionPolicy.RetryOptions]: https://msdn.microsoft.com/library/azure/microsoft.azure.documents.client.connectionpolicy.retryoptions.aspx
[dotnet-foundation]: https://dotnetfoundation.org/
[polly]: http://www.thepollyproject.org
[redis-cache-troubleshoot]: /azure/redis-cache/cache-how-to-troubleshoot
[SearchIndexClient]: https://msdn.microsoft.com/library/azure/microsoft.azure.search.searchindexclient.aspx
[SearchServiceClient]: https://msdn.microsoft.com/library/microsoft.azure.search.searchserviceclient.aspx
