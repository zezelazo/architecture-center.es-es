---
title: Procesamiento de flujos de datos con Azure Stream Analytics
titleSuffix: Azure Reference Architectures
description: Cree una canalización de procesamiento de flujos de datos de un extremo a otro en Azure.
author: MikeWasson
ms.date: 11/06/2018
ms.custom: seodec18
ms.openlocfilehash: 130f297d3cfdeb1900ada79f1e9c65ec542dc2b7
ms.sourcegitcommit: bb7fcffbb41e2c26a26f8781df32825eb60df70c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/20/2018
ms.locfileid: "53643754"
---
# <a name="create-a-stream-processing-pipeline-with-azure-stream-analytics"></a>Creación de una canalización de procesamiento de flujos de datos con Azure Stream Analytics

Esta arquitectura de referencia muestra una canalización de [procesamiento de flujos de datos](/azure/architecture/data-guide/big-data/real-time-processing) de un extremo a otro. La canalización ingiere los datos de dos orígenes, correlaciona los registros de los dos flujos de datos y calcula una media acumulada en un intervalo de tiempo. Los resultados se almacenan para su posterior análisis.

Hay disponible una implementación de referencia de esta arquitectura en [GitHub][github].

![Arquitectura de referencia para la creación de una canalización de procesamiento de flujos de datos con Azure Stream Analytics](./images/stream-processing-asa/stream-processing-asa.png)

**Escenario**: Una empresa de taxi recopila los datos acerca de cada carrera de taxi. En este escenario, se supone que hay dos dispositivos independientes que envían datos. El taxi tienen un medidor que envía la información acerca de cada carrera: duración, distancia y ubicaciones de recogida y destino. Un dispositivo independiente acepta los pagos de clientes y envía los datos sobre las tarifas. La empresa de taxi desea calcular el promedio de propinas por milla conducida, en tiempo real, con el fin de identificar las tendencias.

## <a name="architecture"></a>Arquitectura

La arquitectura consta de los siguientes componentes:

**Orígenes de datos**. En esta arquitectura, hay dos orígenes de datos que generan flujos de datos en tiempo real. El primer flujo de datos contiene información sobre la carrera y, el segundo, contiene información sobre las tarifas. La arquitectura de referencia incluye un generador de datos simulados que lee un conjunto de archivos estáticos e inserta los datos en Event Hubs. En una aplicación real, los orígenes de datos serían los dispositivos instalados en el taxi.

**Azure Event Hubs**. [Event Hubs](/azure/event-hubs/) es un servicio de ingesta de eventos. Esta arquitectura emplea dos instancias de centro de eventos, uno para cada origen de datos. Cada origen de datos envía un flujo de datos al centro de eventos asociado.

**Azure Stream Analytics**. [Stream Analytics](/azure/stream-analytics/) es un motor de procesamiento de eventos. Un trabajo de Stream Analytics lee los flujos de datos desde los dos centros de eventos y realiza el procesamiento de los flujos.

**Cosmos DB**. La salida del trabajo de Stream Analytics es una serie de registros que se escriben como documentos JSON en una base de datos de documentos de Cosmos DB.

**Microsoft Power BI**. Power BI es un conjunto de herramientas de análisis de negocios que sirve para analizar datos con el fin de obtener perspectivas empresariales. En esta arquitectura, carga los datos desde Cosmos DB. Esto permite a los usuarios analizar el conjunto completo de los datos históricos que se han recopilado. También podría transmitir los resultados directamente desde Stream Analytics a Power BI para obtener una vista en tiempo real de los datos. Para más información, consulte [Streaming en tiempo real en Power BI](/power-bi/service-real-time-streaming).

**Azure Monitor**. [Azure Monitor](/azure/monitoring-and-diagnostics/) recopila métricas de rendimiento sobre los servicios de Azure implementados en la solución. Puede visualizarlos en un panel para obtener información acerca del estado de la solución.

## <a name="data-ingestion"></a>Ingesta de datos

<!-- markdownlint-disable MD033 MD034 -->

Para simular un origen de datos, esta arquitectura de referencia usa el conjunto de datos [New York City Taxi Data](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797)<sup>[[1]](#note1)</sup>. Este conjunto de datos contiene datos acerca de carreras de taxi en la ciudad de Nueva York durante un período de 4 años (de 2010 a 2013). Contiene dos tipos de registros: datos de carreras y datos de tarifas. Los datos de carreras incluyen la duración del viaje, la distancia de viaje y la ubicación de recogida y destino. Los datos de tarifas incluyen las tarifas, los impuestos y las propinas. Los campos comunes en ambos tipos de registro son la placa y el número de licencia, y el identificador del proveedor. Juntos, estos tres campos identifican un taxi además del conductor. Los datos se almacenan en formato CSV.

[1] <span id="note1">Donovan, Brian; Trabajo, Dan (2016): New York City Taxi Trip Data (2010-2013). Universidad de Illinois en Urbana-Champaign. https://doi.org/10.13012/J8PN93H8

<!-- markdownlint-enable MD033 MD034 -->

El generador de datos es una aplicación de .NET Core que lee los registros y los envía a Azure Event Hubs. El generador envía los datos de carreras en formato JSON y los datos de tarifas en formato CSV.

Event Hubs usa [particiones](/azure/event-hubs/event-hubs-features#partitions) para segmentar los datos. Las particiones permiten a los consumidores leer cada partición en paralelo. Cuando se envían datos a Event Hubs, puede especificar explícitamente la clave de partición. En caso contrario, los registros se asignan a las particiones en modo round-robin.

En este escenario en particular, los datos de carreras y los datos de tarifas deben terminar con el mismo identificador de partición para un taxi determinado. Esto permite a Stream Analytics aplicar un cierto paralelismo cuando se establece una correlación entre los dos flujos. Un registro en la partición *n* de los datos de carreras coincidirá con un registro en la partición de datos *n* de los datos de tarifas.

![Diagrama de procesamiento de flujos de datos con Azure Stream Analytics y Event Hubs](./images/stream-processing-asa/stream-processing-eh.png)

En el generador de datos, el modelo de datos común para ambos tipos de registro tiene una propiedad `PartitionKey` que es la concatenación de `Medallion`, `HackLicense` y `VendorId`.

```csharp
public abstract class TaxiData
{
    public TaxiData()
    {
    }

    [JsonProperty]
    public long Medallion { get; set; }

    [JsonProperty]
    public long HackLicense { get; set; }

    [JsonProperty]
    public string VendorId { get; set; }

    [JsonProperty]
    public DateTimeOffset PickupTime { get; set; }

    [JsonIgnore]
    public string PartitionKey
    {
        get => $"{Medallion}_{HackLicense}_{VendorId}";
    }
```

Esta propiedad se utiliza para proporcionar una clave de partición explícita cuando se realizan envíos a Event Hubs:

```csharp
using (var client = pool.GetObject())
{
    return client.Value.SendAsync(new EventData(Encoding.UTF8.GetBytes(
        t.GetData(dataFormat))), t.PartitionKey);
}
```

## <a name="stream-processing"></a>Procesamiento de flujos

El trabajo de procesamiento de flujos se define mediante una consulta SQL con diferentes pasos. Los dos primeros pasos simplemente seleccionan los registros de los dos flujos de entrada.

```sql
WITH
Step1 AS (
    SELECT PartitionId,
           TRY_CAST(Medallion AS nvarchar(max)) AS Medallion,
           TRY_CAST(HackLicense AS nvarchar(max)) AS HackLicense,
           VendorId,
           TRY_CAST(PickupTime AS datetime) AS PickupTime,
           TripDistanceInMiles
    FROM [TaxiRide] PARTITION BY PartitionId
),
Step2 AS (
    SELECT PartitionId,
           medallion AS Medallion,
           hack_license AS HackLicense,
           vendor_id AS VendorId,
           TRY_CAST(pickup_datetime AS datetime) AS PickupTime,
           tip_amount AS TipAmount
    FROM [TaxiFare] PARTITION BY PartitionId
),
```

El paso siguiente combina los dos flujos de entrada para seleccionar los registros coincidentes de cada flujo.

```sql
Step3 AS (
  SELECT
         tr.Medallion,
         tr.HackLicense,
         tr.VendorId,
         tr.PickupTime,
         tr.TripDistanceInMiles,
         tf.TipAmount
    FROM [Step1] tr
    PARTITION BY PartitionId
    JOIN [Step2] tf PARTITION BY PartitionId
      ON tr.Medallion = tf.Medallion
     AND tr.HackLicense = tf.HackLicense
     AND tr.VendorId = tf.VendorId
     AND tr.PickupTime = tf.PickupTime
     AND tr.PartitionId = tf.PartitionId
     AND DATEDIFF(minute, tr, tf) BETWEEN 0 AND 15
)
```

Esta consulta combina los registros en un conjunto de campos que identifican los registros coincidentes de forma única (Medallion, HackLicense, VendorId y PickupTime). La instrucción `JOIN` también incluye el identificador de partición. Como ya se mencionó, esto aprovecha el hecho de que los registros coincidentes siempre tienen el mismo identificador de partición en este escenario.

En Stream Analytics, las combinaciones son *temporales*, lo que significa que los registros se combinan dentro de un intervalo de tiempo determinado. De caso contrario, el trabajo podría tener que esperar indefinidamente a una coincidencia. La función [DATEDIFF](https://msdn.microsoft.com/azure/stream-analytics/reference/join-azure-stream-analytics) especifica la separación máxima en el tiempo de dos registros coincidentes para considerarlo una coincidencia.

El último paso del trabajo calcula la media de propinas por milla, agrupada por una ventana de salto de 5 minutos.

```sql
SELECT System.Timestamp AS WindowTime,
       SUM(tr.TipAmount) / SUM(tr.TripDistanceInMiles) AS AverageTipPerMile
  INTO [TaxiDrain]
  FROM [Step3] tr
  GROUP BY HoppingWindow(Duration(minute, 5), Hop(minute, 1))
```

Stream Analytics proporciona varias [funciones de intervalos de tiempo](/azure/stream-analytics/stream-analytics-window-functions). Una ventana de salto avanza en el tiempo un período fijo; en este caso, 1 minuto por salto. El objetivo es calcular una media móvil en los últimos cinco minutos.

En la arquitectura que se muestra a continuación, solo se guardan los resultados del trabajo de Stream Analytics en Cosmos DB. En un escenario de macrodatos, considere la posibilidad de usar también [Event Hubs Capture](/azure/event-hubs/event-hubs-capture-overview) para guardar los datos de eventos sin procesar en Azure Blob Storage. Mantener los datos sin procesar le permitirá ejecutar consultas por lotes en los datos históricos en el futuro, con el fin de obtener información nueva a partir de los datos.

## <a name="scalability-considerations"></a>Consideraciones sobre escalabilidad

### <a name="event-hubs"></a>Event Hubs

La capacidad de procesamiento de Event Hubs se mide en [unidades de procesamiento](/azure/event-hubs/event-hubs-features#throughput-units). Para escalar un centro de eventos automáticamente, puede habilitar el [inflado automático](/azure/event-hubs/event-hubs-auto-inflate), que permite escalar las unidades de procesamiento en función del tráfico, hasta un máximo configurado.

### <a name="stream-analytics"></a>Stream Analytics

Para Stream Analytics, los recursos de proceso que se asignan a un trabajo se miden en unidades de streaming. Los trabajos de Stream Analytics escalan mejor si el trabajo se puede ejecutar en paralelo. De este modo, Stream Analytics puede distribuir el trabajo entre varios nodos de proceso.

Para los datos de entrada de Event Hubs, utilice la palabra clave `PARTITION BY` para dividir el trabajo de Stream Analytics en particiones. Los datos se dividirán en subconjuntos en función de las particiones de Event Hubs.

Las funciones de intervalos de tiempo y las combinaciones temporales requieren unidades de streaming adicionales. Cuando sea posible, use `PARTITION BY` para que cada partición se procese por separado. Para más información, consulte [Descripción y ajuste de las unidades de streaming](/azure/stream-analytics/stream-analytics-streaming-unit-consumption#windowed-aggregates).

Si no es posible ejecutar todo el trabajo de Stream Analytics en paralelo, intente dividirlo en varios pasos, empezando con uno o varios pasos en paralelo. De este modo, los primeros pasos que pueden ejecutar en paralelo. Por ejemplo, en esta arquitectura de referencia:

- Los pasos 1 y 2 son instrucciones `SELECT` simples que seleccionan registros dentro de una sola partición.
- El paso 3 realiza una combinación con particiones en los dos flujos de entrada. Este paso aprovecha del hecho de que los registros que coinciden comparten la misma clave de partición y, por lo tanto, garantiza que tendrán el mismo identificador de partición en cada flujo de entrada.
- El paso 4 realiza la agregación en todas las particiones. Este paso no se puede ejecutar en paralelo.

Use el [diagrama de trabajo](/azure/stream-analytics/stream-analytics-job-diagram-with-metrics) de Stream Analytics para ver cuántas particiones se asignan a cada paso del trabajo. El siguiente diagrama muestra el diagrama de trabajo de esta arquitectura de referencia:

![Diagrama de trabajo](./images/stream-processing-asa/job-diagram.png)

### <a name="cosmos-db"></a>Cosmos DB

La capacidad de rendimiento de Cosmos DB se mide en [unidades de solicitud](/azure/cosmos-db/request-units) (RU). Para escalar un contenedor de Cosmos DB más allá de 10 000 RU, debe especificar una [clave de partición](/azure/cosmos-db/partition-data) al crear el contenedor, e incluir la clave de partición en todos los documentos.

En esta arquitectura de referencia, se crean nuevos documentos solo una vez por minuto (el intervalo de la ventana salto), por lo que los requisitos de procesamiento son bastante bajos. Por ese motivo, no es necesario asignar una clave de partición en este escenario.

## <a name="monitoring-considerations"></a>Consideraciones sobre supervisión

Con cualquier solución de procesamiento de flujos, es importante supervisar el rendimiento y el estado del sistema. [Azure Monitor](/azure/monitoring-and-diagnostics/) recopila métricas y registros de diagnóstico para los servicios de Azure que se utilizan en la arquitectura. Azure Monitor se integra en la plataforma de Azure y no requiere ningún código adicional en la aplicación.

Cualquiera de las siguientes señales de advertencia indica que debe escalar horizontalmente el recurso de Azure correspondiente:

- Event Hubs limita las solicitudes o está cerca de la cuota diaria de mensajes.
- El trabajo de Stream Analytics utiliza sistemáticamente más del 80 % de las unidades de streaming (SU) asignadas.
- Cosmos DB comienza a limitar las solicitudes.

La arquitectura de referencia incluye un panel personalizado, que se implementa en Azure Portal. Después de implementar la arquitectura, abra [Azure Portal](https://portal.azure.com) y seleccione `TaxiRidesDashboard` en la lista de paneles para ver el panel. Para más información sobre cómo crear e implementar paneles personalizados en Azure Portal, consulte [Creación mediante programación de paneles de Azure](/azure/azure-portal/azure-portal-dashboards-create-programmatically).

La siguiente imagen muestra el panel después de que el trabajo de Stream Analytics se ejecutara durante una hora aproximadamente.

![Captura de pantalla del panel de carreras de taxi](./images/stream-processing-asa/asa-dashboard.png)

El panel de la parte inferior izquierda muestra que el consumo de unidades de streaming para el trabajo de Stream Analytics aumenta durante los primeros 15 minutos y después se nivela. Este es un patrón típico mientras el trabajo alcanza un estado estable.

Tenga en cuenta que Event Hubs limita las solicitudes, tal y como se muestra en el panel superior derecho. Un solicitud limitada de vez en cuanto no es un problema, porque el SDK de cliente de Event Hubs realiza reintentos automáticamente cuando recibe un error de limitación. Sin embargo, si se producen errores de limitación sistemáticamente, significa que el centro de eventos necesita más unidades de procesamiento. El gráfico siguiente muestra una ejecución de prueba con la característica de inflado automático de Event Hubs, que escala horizontalmente las unidades de procesamiento automáticamente cuando es necesario.

![Captura de pantalla del escalado automático de Event Hubs](./images/stream-processing-asa/stream-processing-eh-autoscale.png)

El inflado automático se habilitó en la marca de 06:35 aproximadamente. Puede ver la caída de solicitudes limitadas, porque Event Hubs escaló automáticamente a 3 unidades de procesamiento.

Curiosamente, esto tuvo el efecto secundario de aumentar el uso de las unidades de streaming en el trabajo de Stream Analytics. Con la limitación, Event Hubs redujo artificialmente la tasa de ingesta del trabajo de Stream Analytics. Es bastante habitual que, al resolver un cuello de botella de rendimiento, se produzca otro. En este caso, asignar unidades de streaming adicionales para el trabajo de Stream Analytics resolvió el problema.

## <a name="deploy-the-solution"></a>Implementación de la solución

Para la implementación y ejecución de la implementación de referencia, siga los pasos del [Léame de GitHub][github].

## <a name="related-resources"></a>Recursos relacionados

Puede examinar los siguientes [escenarios de ejemplo de Azure](/azure/architecture/example-scenario) que muestran soluciones específicas que usan algunas tecnologías similares:

- [IoT y el análisis de datos en el sector de la construcción](/azure/architecture/example-scenario/data/big-data-with-iot)
- [Detección de fraudes en tiempo real](/azure/architecture/example-scenario/data/fraud-detection)

<!-- links -->

[github]: https://github.com/mspnp/azure-stream-analytics-data-pipeline

