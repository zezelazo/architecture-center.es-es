---
title: Procesamiento de eventos sin servidor con Azure Functions
description: Arquitectura de referencia que muestra la ingesta y el procesamiento de eventos sin servidor
author: MikeWasson
ms.date: 10/16/2018
ms.openlocfilehash: 76c8b9c1244c987c96e38e50ecad7814cc49cd88
ms.sourcegitcommit: 19a517a2fb70768b3edb9a7c3c37197baa61d9b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/26/2018
ms.locfileid: "52295657"
---
# <a name="serverless-event-processing-using-azure-functions"></a>Procesamiento de eventos sin servidor con Azure Functions

Esta arquitectura de referencia muestra una arquitectura [sin servidor](https://azure.microsoft.com/solutions/serverless/) orientada a eventos que ingiere un flujo de datos, procesa los datos y escribe los resultados en una base de datos de back-end. Hay disponible una implementación de referencia de esta arquitectura en [GitHub][github].

![](./_images/serverless-event-processing.png)

## <a name="architecture"></a>Arquitectura

**Event Hubs** ingiere el flujo de datos. [Event Hubs][eh] está diseñado para escenarios de transmisión de datos de alto rendimiento.

> [!NOTE]
> Para escenarios de IoT, se recomienda IoT Hub. IoT Hub tiene un punto de conexión integrado que es compatible con la API de Azure Event Hubs, por lo que puede usar cualquiera de los servicios en esta arquitectura sin realizar ningún cambio importante en el procesamiento de back-end. Para más información, consulte [Conexión de dispositivos de IoT a Azure: IoT Hub y Event Hubs][iot].

**Aplicación de función**. [Azure Functions][functions] es una opción de proceso sin servidor. Utiliza un modelo orientado a eventos, en el que un desencadenador invoca un fragmento de código (una "función"). En esta arquitectura, cuando los eventos llegan a Event Hubs, desencadenan una función que procesa los eventos y escribe los resultados en el almacenamiento.

Las aplicaciones de función son convenientes para procesar registros individuales procedentes de Event Hubs. Para escenarios de procesamiento de secuencias más complejas, considere la posibilidad de usar Apache Spark con Azure Databricks o Azure Stream Analytics.

**Cosmos DB**. [Cosmos DB][cosmosdb] es un servicio de base de datos multi-modelo. En este escenario, la función de procesamiento de eventos almacena registros JSON, con [SQL API][cosmosdb-sql] de Cosmos DB.

**Queue Storage**. [Queue Storage][queue] se usa para los mensajes fallidos. Si se produce un error al procesar un evento, la función almacena los datos del evento en una cola de mensajes fallidos para su procesamiento posterior. Para más información, consulte [Consideraciones de resistencia](#resiliency-considerations).

**Azure Monitor**. [Monitor][monitor] recopila métricas de rendimiento sobre los servicios de Azure implementados en la solución. Puede visualizarlas en un panel para obtener visibilidad del mantenimiento de la solución.

**Azure Pipelines**. [Pipelines][pipelines] es un servicio de integración continua (CI) y entrega continua (CD) que compila, prueba e implementa la aplicación.

## <a name="scalability-considerations"></a>Consideraciones sobre escalabilidad

### <a name="event-hubs"></a>Event Hubs

La capacidad de procesamiento de Event Hubs se mide en [unidades de procesamiento][eh-throughput]. Para escalar un centro de eventos automáticamente, puede habilitar el [inflado automático][eh-autoscale], que permite escalar las unidades de procesamiento en función del tráfico, hasta un máximo configurado.

El [desencadenador de Event Hubs][eh-trigger] de la aplicación de función se escala según el número de particiones del centro de eventos. Se asigna una instancia de la función a cada partición cada vez. Para maximizar el rendimiento, es posible recibir los eventos en un lote, en lugar de uno cada vez.

### <a name="cosmos-db"></a>Cosmos DB

La capacidad de rendimiento de Cosmos DB se mide en [unidades de solicitud][ru] (RU). Para escalar un contenedor de Cosmos DB más allá de 10 000 RU, debe especificar una [clave de partición][partition-key] al crear el contenedor e incluir la clave de partición en todos los documentos que cree.

Estas son algunas características de una buena clave de partición:

- El espacio de valores de la clave es grande. 
- Habrá una distribución uniforme de lecturas y escrituras por valor de clave, evitando las claves frecuentes.
- El máximo de datos almacenados para cualquier valor de clave único no superará el tamaño de partición física máximo (10 GB). 
- La clave de partición de un documento no cambiará. No se puede actualizar la clave de partición de un documento existente. 

En el escenario de esta arquitectura de referencia, la función almacena exactamente un documento por cada dispositivo que envía datos. La función actualiza continuamente los documentos con el estado del dispositivo más reciente mediante una operación de actualización e inserción de datos. El identificador de dispositivo es una buena clave de partición para este escenario, ya que las escrituras se distribuirán uniformemente entre las claves y el tamaño de cada partición estará enlazado estrictamente porque hay un documento único por cada valor de clave. Para más información sobre las claves de partición, consulte [Particionado y escalado en Azure Cosmos DB][cosmosdb-scale].

## <a name="resiliency-considerations"></a>Consideraciones de resistencia

Al usar el desencadenador de Event Hubs con Azure Functions, debe detectar las excepciones dentro del bucle de procesamiento. Si se produce una excepción no controlada, el runtime de Azure Functions no vuelve a intentar el envío de los mensajes. Si no se puede procesar un mensaje, coloque el mensaje en una cola de mensajes fallidos. Utilice un proceso fuera de banda para examinar los mensajes y determinar la acción correctiva. 

El código siguiente muestra cómo se detectan las excepciones en la función de ingesta y cómo se colocan los mensajes no procesados en una cola de mensajes fallidos.

```csharp
[FunctionName("RawTelemetryFunction")]
[StorageAccount("DeadLetterStorage")]
public static async Task RunAsync(
    [EventHubTrigger("%EventHubName%", Connection = "EventHubConnection", ConsumerGroup ="%EventHubConsumerGroup%")]EventData[] messages,
    [Queue("deadletterqueue")] IAsyncCollector<DeadLetterMessage> deadLetterMessages,
    ILogger logger)
{
    foreach (var message in messages)
    {
        DeviceState deviceState = null;

        try
        {
            deviceState = telemetryProcessor.Deserialize(message.Body.Array, logger);
        }
        catch (Exception ex)
        {
            logger.LogError(ex, "Error deserializing message", message.SystemProperties.PartitionKey, message.SystemProperties.SequenceNumber);
            await deadLetterMessages.AddAsync(new DeadLetterMessage { Issue = ex.Message, EventData = message });
        }

        try
        {
            await stateChangeProcessor.UpdateState(deviceState, logger);
        }
        catch (Exception ex)
        {
            logger.LogError(ex, "Error updating status document", deviceState);
            await deadLetterMessages.AddAsync(new DeadLetterMessage { Issue = ex.Message, EventData = message, DeviceState = deviceState });
        }
    }
}
```

Observe que la función utiliza el [enlace de salida de Queue Storage][queue-binding] para colocar los elementos en la cola.

El código mostrado anteriormente también registra las excepciones en Application Insights. Puede usar el número de secuencia y la clave de partición para correlacionar los mensajes fallidos con las excepciones en los registros. 

Los mensajes de la cola de mensajes fallidos deben tener suficiente información para que pueda entender el contexto del error. En este ejemplo, la clase `DeadLetterMessage` contiene el mensaje con la excepción, los datos del evento original y el mensaje del evento deserializado (si está disponible). 

```csharp
public class DeadLetterMessage
{
    public string Issue { get; set; }
    public EventData EventData { get; set; }
    public DeviceState DeviceState { get; set; }
}
```

Use [Azure Monitor][monitor] para supervisar el centro de eventos. Si ve que hay entrada pero no hay salida, significa que no se están procesando los mensajes. En ese caso, vaya a [Log Analytics][log-analytics] y busque excepciones u otros errores.

## <a name="disaster-recovery-considerations"></a>Consideraciones acerca de la recuperación ante desastres

La implementación que se muestra aquí reside en una sola región de Azure. Para un enfoque más resistente a la recuperación ante desastres, aproveche las ventajas de las características de distribución geográfica de los distintos servicios:

- **Event Hubs**. Cree dos espacios de nombres de Event Hubs, un espacio de nombres principal (activo) y un espacio de nombres secundario (pasivo). Los mensajes se enrutan automáticamente al espacio de nombres activo a menos que realice una conmutación por error al espacio de nombres secundario. Para más información, consulte [Recuperación ante desastres con redundancia geográfica de Azure Event Hubs][eh-dr].

- **Aplicación de función**. Implemente una segunda aplicación de función que está a la espera para leer desde el espacio de nombres secundario de Event Hubs. Esta función escribe en una cuenta de almacenamiento secundaria para la cola de mensajes fallidos.

- **Cosmos DB**. Cosmos DB admite [varias regiones maestras][cosmosdb-geo], lo que permite operaciones de escritura en cualquier región que se agregue a la cuenta de Cosmos DB. Si no habilita la funcionalidad de arquitectura multimaestro, todavía puede conmutar por error a la región de escritura principal. Los SDK de cliente de Cosmos DB y los enlaces de la función de Azure controlan automáticamente la conmutación por error, por lo que no es necesario actualizar los parámetros de configuración de la aplicación.

- **Azure Storage**. Use almacenamiento [RA-GRS][ra-grs] para la cola de mensajes fallidos. Esto crea una réplica de solo lectura en otra región. Si la región primaria deja de estar disponible, puede leer los elementos que hay actualmente en la cola. Además, aprovisione otra cuenta de almacenamiento en la región secundaria en la que la función pueda escribir después de una conmutación por error.

## <a name="deploy-the-solution"></a>Implementación de la solución

Para implementar esta arquitectura de referencia, consulte el [Léame de GitHub][readme]. 

<!-- links -->

[cosmosdb]: /azure/cosmos-db/introduction
[cosmosdb-geo]: /azure/cosmos-db/distribute-data-globally
[cosmosdb-scale]: /azure/cosmos-db/partition-data
[cosmosdb-sql]: /azure/cosmos-db/sql-api-introduction
[eh]: /azure/event-hubs/
[eh-autoscale]: /azure/event-hubs/event-hubs-auto-inflate
[eh-dr]: /azure/event-hubs/event-hubs-geo-dr
[eh-throughput]: /azure/event-hubs/event-hubs-features#throughput-units
[eh-trigger]: /azure/azure-functions/functions-bindings-event-hubs
[functions]: /azure/azure-functions/functions-overview
[iot]: /azure/iot-hub/iot-hub-compare-event-hubs
[log-analytics]: /azure/log-analytics/log-analytics-queries
[monitor]: /azure/azure-monitor/overview
[partition-key]: /azure/cosmos-db/partition-data
[pipelines]: /azure/devops/pipelines/index
[queue]: /azure/storage/queues/storage-queues-introduction
[queue-binding]: /azure/azure-functions/functions-bindings-storage-queue#output
[ra-grs]: /azure/storage/common/storage-redundancy-grs
[ru]: /azure/cosmos-db/request-units

[github]: https://github.com/mspnp/serverless-reference-implementation
[readme]: https://github.com/mspnp/serverless-reference-implementation/blob/master/README.md
