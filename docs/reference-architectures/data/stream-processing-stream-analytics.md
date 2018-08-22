---
title: Procesamiento de flujos de datos con Azure Stream Analytics
description: Creación de una canalización de procesamiento de flujos de datos de un extremo a otro en Azure
author: MikeWasson
ms.date: 08/09/2018
ms.openlocfilehash: 82887bdd45f811ac733ead18c1f256098e575253
ms.sourcegitcommit: c4106b58ad08f490e170e461009a4693578294ea
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2018
ms.locfileid: "40025522"
---
# <a name="stream-processing-with-azure-stream-analytics"></a><span data-ttu-id="6bf74-103">Procesamiento de flujos de datos con Azure Stream Analytics</span><span class="sxs-lookup"><span data-stu-id="6bf74-103">Stream processing with Azure Stream Analytics</span></span>

<span data-ttu-id="6bf74-104">Esta arquitectura de referencia muestra una canalización de procesamiento de flujos de datos de un extremo a otro.</span><span class="sxs-lookup"><span data-stu-id="6bf74-104">This reference architecture shows an end-to-end stream processing pipeline.</span></span> <span data-ttu-id="6bf74-105">La canalización ingiere los datos de dos orígenes, correlaciona los registros de los dos flujos de datos y calcula una media acumulada en un intervalo de tiempo.</span><span class="sxs-lookup"><span data-stu-id="6bf74-105">The pipeline ingests data from two sources, correlates records in the two streams, and calculates a rolling average across a time window.</span></span> <span data-ttu-id="6bf74-106">Los resultados se almacenan para su posterior análisis.</span><span class="sxs-lookup"><span data-stu-id="6bf74-106">The results are stored for further analysis.</span></span> [<span data-ttu-id="6bf74-107">**Implemente esta solución**.</span><span class="sxs-lookup"><span data-stu-id="6bf74-107">**Deploy this solution**.</span></span>](#deploy-the-solution)

![](./images/stream-processing-asa/stream-processing-asa.png)

<span data-ttu-id="6bf74-108">**Escenario**: una empresa de taxi recopila los datos acerca de cada carrera de taxi.</span><span class="sxs-lookup"><span data-stu-id="6bf74-108">**Scenario**: A taxi company collects data about each taxi trip.</span></span> <span data-ttu-id="6bf74-109">En este escenario, se supone que hay dos dispositivos independientes que envían datos.</span><span class="sxs-lookup"><span data-stu-id="6bf74-109">For this scenario, we assume there are two separate devices sending data.</span></span> <span data-ttu-id="6bf74-110">El taxi tienen un medidor que envía la información acerca de cada carrera: duración, distancia y ubicaciones de recogida y destino.</span><span class="sxs-lookup"><span data-stu-id="6bf74-110">The taxi has a meter that sends information about each ride &mdash; the duration, distance, and pickup and dropoff locations.</span></span> <span data-ttu-id="6bf74-111">Un dispositivo independiente acepta los pagos de clientes y envía los datos sobre las tarifas.</span><span class="sxs-lookup"><span data-stu-id="6bf74-111">A separate device accepts payments from customers and sends data about fares.</span></span> <span data-ttu-id="6bf74-112">La empresa de taxi desea calcular el promedio de propinas por milla conducida, en tiempo real, con el fin de identificar las tendencias.</span><span class="sxs-lookup"><span data-stu-id="6bf74-112">The taxi company wants to calculate the average tip per mile driven, in real time, in order to spot trends.</span></span>

## <a name="architecture"></a><span data-ttu-id="6bf74-113">Arquitectura</span><span class="sxs-lookup"><span data-stu-id="6bf74-113">Architecture</span></span>

<span data-ttu-id="6bf74-114">La arquitectura consta de los siguientes componentes:</span><span class="sxs-lookup"><span data-stu-id="6bf74-114">The architecture consists of the following components.</span></span>

<span data-ttu-id="6bf74-115">**Orígenes de datos**.</span><span class="sxs-lookup"><span data-stu-id="6bf74-115">**Data sources**.</span></span> <span data-ttu-id="6bf74-116">En esta arquitectura, hay dos orígenes de datos que generan flujos de datos en tiempo real.</span><span class="sxs-lookup"><span data-stu-id="6bf74-116">In this architecture, there are two data sources that generate data streams in real time.</span></span> <span data-ttu-id="6bf74-117">El primer flujo de datos contiene información sobre la carrera y, el segundo, contiene información sobre las tarifas.</span><span class="sxs-lookup"><span data-stu-id="6bf74-117">The first stream contains ride information, and the second contains fare information.</span></span> <span data-ttu-id="6bf74-118">La arquitectura de referencia incluye un generador de datos simulados que lee un conjunto de archivos estáticos e inserta los datos en Event Hubs.</span><span class="sxs-lookup"><span data-stu-id="6bf74-118">The reference architecture includes a simulated data generator that reads from a set of static files and pushes the data to Event Hubs.</span></span> <span data-ttu-id="6bf74-119">En una aplicación real, los orígenes de datos serían los dispositivos instalados en el taxi.</span><span class="sxs-lookup"><span data-stu-id="6bf74-119">In a real application, the data sources would be devices installed in the taxi cabs.</span></span>

<span data-ttu-id="6bf74-120">**Azure Event Hubs**.</span><span class="sxs-lookup"><span data-stu-id="6bf74-120">**Azure Event Hubs**.</span></span> <span data-ttu-id="6bf74-121">[Event Hubs](/azure/event-hubs/) es un servicio de ingesta de eventos.</span><span class="sxs-lookup"><span data-stu-id="6bf74-121">[Event Hubs](/azure/event-hubs/) is an event ingestion service.</span></span> <span data-ttu-id="6bf74-122">Esta arquitectura emplea dos instancias de centro de eventos, uno para cada origen de datos.</span><span class="sxs-lookup"><span data-stu-id="6bf74-122">This architecture uses two event hub instances, one for each data source.</span></span> <span data-ttu-id="6bf74-123">Cada origen de datos envía un flujo de datos al centro de eventos asociado.</span><span class="sxs-lookup"><span data-stu-id="6bf74-123">Each data source sends a stream of data to the associated event hub.</span></span>

<span data-ttu-id="6bf74-124">**Azure Stream Analytics**.</span><span class="sxs-lookup"><span data-stu-id="6bf74-124">**Azure Stream Analytics**.</span></span> <span data-ttu-id="6bf74-125">[Stream Analytics](/azure/stream-analytics/) es un motor de procesamiento de eventos.</span><span class="sxs-lookup"><span data-stu-id="6bf74-125">[Stream Analytics](/azure/stream-analytics/) is an event-processing engine.</span></span> <span data-ttu-id="6bf74-126">Un trabajo de Stream Analytics lee los flujos de datos desde los dos centros de eventos y realiza el procesamiento de los flujos.</span><span class="sxs-lookup"><span data-stu-id="6bf74-126">A Stream Analytics job reads the data streams from the two event hubs and performs stream processing.</span></span>

<span data-ttu-id="6bf74-127">**Cosmos DB**.</span><span class="sxs-lookup"><span data-stu-id="6bf74-127">**Cosmos DB**.</span></span> <span data-ttu-id="6bf74-128">La salida del trabajo de Stream Analytics es una serie de registros que se escriben como documentos JSON en una base de datos de documentos de Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="6bf74-128">The output from the Stream Analytics job is a series of records, which are written as JSON documents to a Cosmos DB document database.</span></span>

<span data-ttu-id="6bf74-129">**Microsoft Power BI**.</span><span class="sxs-lookup"><span data-stu-id="6bf74-129">**Microsoft Power BI**.</span></span> <span data-ttu-id="6bf74-130">Power BI es un conjunto de herramientas de análisis de negocios que sirve para analizar datos con el fin de obtener perspectivas empresariales.</span><span class="sxs-lookup"><span data-stu-id="6bf74-130">Power BI is a suite of business analytics tools to analyze data for business insights.</span></span> <span data-ttu-id="6bf74-131">En esta arquitectura, carga los datos desde Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="6bf74-131">In this architecture, it loads the data from Cosmos DB.</span></span> <span data-ttu-id="6bf74-132">Esto permite a los usuarios analizar el conjunto completo de los datos históricos que se han recopilado.</span><span class="sxs-lookup"><span data-stu-id="6bf74-132">This allows users to analyze the complete set of historical data that's been collected.</span></span> <span data-ttu-id="6bf74-133">También podría transmitir los resultados directamente desde Stream Analytics a Power BI para obtener una vista en tiempo real de los datos.</span><span class="sxs-lookup"><span data-stu-id="6bf74-133">You could also stream the results directly from Stream Analytics to Power BI for a real-time view of the data.</span></span> <span data-ttu-id="6bf74-134">Para más información, consulte [Streaming en tiempo real en Power BI](/power-bi/service-real-time-streaming).</span><span class="sxs-lookup"><span data-stu-id="6bf74-134">For more information, see [Real-time streaming in Power BI](/power-bi/service-real-time-streaming).</span></span>

<span data-ttu-id="6bf74-135">**Azure Monitor**.</span><span class="sxs-lookup"><span data-stu-id="6bf74-135">**Azure Monitor**.</span></span> <span data-ttu-id="6bf74-136">[Azure Monitor](/azure/monitoring-and-diagnostics/) recopila métricas de rendimiento sobre los servicios de Azure implementados en la solución.</span><span class="sxs-lookup"><span data-stu-id="6bf74-136">[Azure Monitor](/azure/monitoring-and-diagnostics/) collects performance metrics about the Azure services deployed in the solution.</span></span> <span data-ttu-id="6bf74-137">Puede visualizarlos en un panel para obtener información acerca del estado de la solución.</span><span class="sxs-lookup"><span data-stu-id="6bf74-137">By visualizing these in a dashboard, you can get insights into the health of the solution.</span></span> 

## <a name="data-ingestion"></a><span data-ttu-id="6bf74-138">Ingesta de datos</span><span class="sxs-lookup"><span data-stu-id="6bf74-138">Data ingestion</span></span>

<span data-ttu-id="6bf74-139">Para simular un origen de datos, esta arquitectura de referencia usa el conjunto de datos [New York City Taxi Data](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797)<sup>[[1]](#note1)</sup>.</span><span class="sxs-lookup"><span data-stu-id="6bf74-139">To simulate a data source, this reference architecture uses the [New York City Taxi Data](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797) dataset<sup>[[1]](#note1)</sup>.</span></span> <span data-ttu-id="6bf74-140">Este conjunto de datos contiene datos acerca de carreras de taxi en la ciudad de Nueva York durante un período de 4 años (de 2010 a 2013).</span><span class="sxs-lookup"><span data-stu-id="6bf74-140">This dataset contains data about taxi trips in New York City over a 4-year period (2010 &ndash; 2013).</span></span> <span data-ttu-id="6bf74-141">Contiene dos tipos de registros: datos de carreras y datos de tarifas.</span><span class="sxs-lookup"><span data-stu-id="6bf74-141">It contains two types of record: Ride data and fare data.</span></span> <span data-ttu-id="6bf74-142">Los datos de carreras incluyen la duración del viaje, la distancia de viaje y la ubicación de recogida y destino.</span><span class="sxs-lookup"><span data-stu-id="6bf74-142">Ride data includes trip duration, trip distance, and pickup and dropoff location.</span></span> <span data-ttu-id="6bf74-143">Los datos de tarifas incluyen las tarifas, los impuestos y las propinas.</span><span class="sxs-lookup"><span data-stu-id="6bf74-143">Fare data includes fare, tax, and tip amounts.</span></span> <span data-ttu-id="6bf74-144">Los campos comunes en ambos tipos de registro son la placa y el número de licencia, y el identificador del proveedor.</span><span class="sxs-lookup"><span data-stu-id="6bf74-144">Common fields in both record types include medallion number, hack license, and vendor ID.</span></span> <span data-ttu-id="6bf74-145">Juntos, estos tres campos identifican un taxi además del conductor.</span><span class="sxs-lookup"><span data-stu-id="6bf74-145">Together these three fields uniquely identify a taxi plus a driver.</span></span> <span data-ttu-id="6bf74-146">Los datos se almacenan en formato CSV.</span><span class="sxs-lookup"><span data-stu-id="6bf74-146">The data is stored in CSV format.</span></span> 

<span data-ttu-id="6bf74-147">El generador de datos es una aplicación de .NET Core que lee los registros y los envía a Azure Event Hubs.</span><span class="sxs-lookup"><span data-stu-id="6bf74-147">The data generator is a .NET Core application that reads the records and sends them to Azure Event Hubs.</span></span> <span data-ttu-id="6bf74-148">El generador envía los datos de carreras en formato JSON y los datos de tarifas en formato CSV.</span><span class="sxs-lookup"><span data-stu-id="6bf74-148">The generator sends ride data in JSON format and fare data in CSV format.</span></span> 

<span data-ttu-id="6bf74-149">Event Hubs usa [particiones](/azure/event-hubs/event-hubs-features#partitions) para segmentar los datos.</span><span class="sxs-lookup"><span data-stu-id="6bf74-149">Event Hubs uses [partitions](/azure/event-hubs/event-hubs-features#partitions) to segment the data.</span></span> <span data-ttu-id="6bf74-150">Las particiones permiten a los consumidores leer cada partición en paralelo.</span><span class="sxs-lookup"><span data-stu-id="6bf74-150">Partitions allow a consumer to read each partition in parallel.</span></span> <span data-ttu-id="6bf74-151">Cuando se envían datos a Event Hubs, puede especificar explícitamente la clave de partición.</span><span class="sxs-lookup"><span data-stu-id="6bf74-151">When you send data to Event Hubs, you can specify the partition key explicitly.</span></span> <span data-ttu-id="6bf74-152">En caso contrario, los registros se asignan a las particiones en modo round-robin.</span><span class="sxs-lookup"><span data-stu-id="6bf74-152">Otherwise, records are assigned to partitions in round-robin fashion.</span></span> 

<span data-ttu-id="6bf74-153">En este escenario en particular, los datos de carreras y los datos de tarifas deben terminar con el mismo identificador de partición para un taxi determinado.</span><span class="sxs-lookup"><span data-stu-id="6bf74-153">In this particular scenario, ride data and fare data should end up with the same partition ID for a given taxi cab.</span></span> <span data-ttu-id="6bf74-154">Esto permite a Stream Analytics aplicar un cierto paralelismo cuando se establece una correlación entre los dos flujos.</span><span class="sxs-lookup"><span data-stu-id="6bf74-154">This enables Stream Analytics to apply a degree of parallelism when it correlates the two streams.</span></span> <span data-ttu-id="6bf74-155">Un registro en la partición *n* de los datos de carreras coincidirá con un registro en la partición de datos *n* de los datos de tarifas.</span><span class="sxs-lookup"><span data-stu-id="6bf74-155">A record in partition *n* of the ride data will match a record in partition *n* of the fare data.</span></span>

![](./images/stream-processing-asa/stream-processing-eh.png)

<span data-ttu-id="6bf74-156">En el generador de datos, el modelo de datos común para ambos tipos de registro tiene una propiedad `PartitionKey` que es la concatenación de `Medallion`, `HackLicense` y `VendorId`.</span><span class="sxs-lookup"><span data-stu-id="6bf74-156">In the data generator, the common data model for both record types has a `PartitionKey` property which is the concatenation of `Medallion`, `HackLicense`, and `VendorId`.</span></span>

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

<span data-ttu-id="6bf74-157">Esta propiedad se utiliza para proporcionar una clave de partición explícita cuando se realizan envíos a Event Hubs:</span><span class="sxs-lookup"><span data-stu-id="6bf74-157">This property is used to provide an explicit partition key when sending to Event Hubs:</span></span>

```csharp
using (var client = pool.GetObject())
{
    return client.Value.SendAsync(new EventData(Encoding.UTF8.GetBytes(
        t.GetData(dataFormat))), t.PartitionKey);
}
```

## <a name="stream-processing"></a><span data-ttu-id="6bf74-158">Procesamiento de flujos</span><span class="sxs-lookup"><span data-stu-id="6bf74-158">Stream processing</span></span>

<span data-ttu-id="6bf74-159">El trabajo de procesamiento de flujos se define mediante una consulta SQL con diferentes pasos.</span><span class="sxs-lookup"><span data-stu-id="6bf74-159">The stream processing job is defined using a SQL query with several distinct steps.</span></span> <span data-ttu-id="6bf74-160">Los dos primeros pasos simplemente seleccionan los registros de los dos flujos de entrada.</span><span class="sxs-lookup"><span data-stu-id="6bf74-160">The first two steps simply select records from the two input streams.</span></span>

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

<span data-ttu-id="6bf74-161">El paso siguiente combina los dos flujos de entrada para seleccionar los registros coincidentes de cada flujo.</span><span class="sxs-lookup"><span data-stu-id="6bf74-161">The next step joins the two input streams to select matching records from each stream.</span></span>

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

<span data-ttu-id="6bf74-162">Esta consulta combina los registros en un conjunto de campos que identifican los registros coincidentes de forma única (Medallion, HackLicense, VendorId y PickupTime).</span><span class="sxs-lookup"><span data-stu-id="6bf74-162">This query joins records on a set of fields that uniquely identify matching records (Medallion, HackLicense, VendorId, and PickupTime).</span></span> <span data-ttu-id="6bf74-163">La instrucción `JOIN` también incluye el identificador de partición.</span><span class="sxs-lookup"><span data-stu-id="6bf74-163">The `JOIN` statement also includes the partition ID.</span></span> <span data-ttu-id="6bf74-164">Como ya se mencionó, esto aprovecha el hecho de que los registros coincidentes siempre tienen el mismo identificador de partición en este escenario.</span><span class="sxs-lookup"><span data-stu-id="6bf74-164">As mentioned, this takes advantage of the fact that matching records always have the same partition ID in this scenario.</span></span>

<span data-ttu-id="6bf74-165">En Stream Analytics, las combinaciones son *temporales*, lo que significa que los registros se combinan dentro de un intervalo de tiempo determinado.</span><span class="sxs-lookup"><span data-stu-id="6bf74-165">In Stream Analytics, joins are *temporal*, meaning records are joined within a particular window of time.</span></span> <span data-ttu-id="6bf74-166">De caso contrario, el trabajo podría tener que esperar indefinidamente a una coincidencia.</span><span class="sxs-lookup"><span data-stu-id="6bf74-166">Otherwise, the job might need to wait indefinitely for a match.</span></span> <span data-ttu-id="6bf74-167">La función [DATEDIFF](https://msdn.microsoft.com/azure/stream-analytics/reference/join-azure-stream-analytics) especifica la separación máxima en el tiempo de dos registros coincidentes para considerarlo una coincidencia.</span><span class="sxs-lookup"><span data-stu-id="6bf74-167">The [DATEDIFF](https://msdn.microsoft.com/azure/stream-analytics/reference/join-azure-stream-analytics) function specifies how far two matching records can be separated in time for a match.</span></span> 

<span data-ttu-id="6bf74-168">El último paso del trabajo calcula la media de propinas por milla, agrupada por una ventana de salto de 5 minutos.</span><span class="sxs-lookup"><span data-stu-id="6bf74-168">The last step in the job computes the average tip per mile, grouped by a hopping window of 5 minutes.</span></span>

```sql
SELECT System.Timestamp AS WindowTime,
       SUM(tr.TipAmount) / SUM(tr.TripDistanceInMiles) AS AverageTipPerMile
  INTO [TaxiDrain]
  FROM [Step3] tr
  GROUP BY HoppingWindow(Duration(minute, 5), Hop(minute, 1))
```

<span data-ttu-id="6bf74-169">Stream Analytics proporciona varias [funciones de intervalos de tiempo](/azure/stream-analytics/stream-analytics-window-functions).</span><span class="sxs-lookup"><span data-stu-id="6bf74-169">Stream Analytics provides several [windowing functions](/azure/stream-analytics/stream-analytics-window-functions).</span></span> <span data-ttu-id="6bf74-170">Una ventana de salto avanza en el tiempo un período fijo; en este caso, 1 minuto por salto.</span><span class="sxs-lookup"><span data-stu-id="6bf74-170">A hopping window moves forward in time by a fixed period, in this case 1 minute per hop.</span></span> <span data-ttu-id="6bf74-171">El objetivo es calcular una media móvil en los últimos cinco minutos.</span><span class="sxs-lookup"><span data-stu-id="6bf74-171">The result is to calculate a moving average over the past 5 minutes.</span></span>

<span data-ttu-id="6bf74-172">En la arquitectura que se muestra a continuación, solo se guardan los resultados del trabajo de Stream Analytics en Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="6bf74-172">In the architecture shown here, only the results of the Stream Analytics job are saved to Cosmos DB.</span></span> <span data-ttu-id="6bf74-173">En un escenario de macrodatos, considere la posibilidad de usar también [Event Hubs Capture](/azure/event-hubs/event-hubs-capture-overview) para guardar los datos de eventos sin procesar en Azure Blob Storage.</span><span class="sxs-lookup"><span data-stu-id="6bf74-173">For a big data scenario, consider also using [Event Hubs Capture](/azure/event-hubs/event-hubs-capture-overview) to save the raw event data into Azure Blob storage.</span></span> <span data-ttu-id="6bf74-174">Mantener los datos sin procesar le permitirá ejecutar consultas por lotes en los datos históricos en el futuro, con el fin de obtener información nueva a partir de los datos.</span><span class="sxs-lookup"><span data-stu-id="6bf74-174">Keeping the raw data will allow you to run batch queries over your historical data at later time, in order to derive new insights from the data.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="6bf74-175">Consideraciones sobre escalabilidad</span><span class="sxs-lookup"><span data-stu-id="6bf74-175">Scalability considerations</span></span>

### <a name="event-hubs"></a><span data-ttu-id="6bf74-176">Event Hubs</span><span class="sxs-lookup"><span data-stu-id="6bf74-176">Event Hubs</span></span>

<span data-ttu-id="6bf74-177">La capacidad de procesamiento de Event Hubs se mide en [unidades de procesamiento](/azure/event-hubs/event-hubs-features#throughput-units).</span><span class="sxs-lookup"><span data-stu-id="6bf74-177">The throughput capacity of Event Hubs is measured in [throughput units](/azure/event-hubs/event-hubs-features#throughput-units).</span></span> <span data-ttu-id="6bf74-178">Para escalar un centro de eventos automáticamente, puede habilitar el [inflado automático](/azure/event-hubs/event-hubs-auto-inflate), que permite escalar las unidades de procesamiento en función del tráfico, hasta un máximo configurado.</span><span class="sxs-lookup"><span data-stu-id="6bf74-178">You can autoscale an event hub by enabling [auto-inflate](/azure/event-hubs/event-hubs-auto-inflate), which automatically scales the throughput units based on traffic, up to a configured maximum.</span></span> 

### <a name="stream-analytics"></a><span data-ttu-id="6bf74-179">Stream Analytics</span><span class="sxs-lookup"><span data-stu-id="6bf74-179">Stream Analytics</span></span>

<span data-ttu-id="6bf74-180">Para Stream Analytics, los recursos de proceso que se asignan a un trabajo se miden en unidades de streaming.</span><span class="sxs-lookup"><span data-stu-id="6bf74-180">For Stream Analytics, the computing resources allocated to a job are measured in Streaming Units.</span></span> <span data-ttu-id="6bf74-181">Los trabajos de Stream Analytics escalan mejor si el trabajo se puede ejecutar en paralelo.</span><span class="sxs-lookup"><span data-stu-id="6bf74-181">Stream Analytics jobs scale best if the job can be parallelized.</span></span> <span data-ttu-id="6bf74-182">De este modo, Stream Analytics puede distribuir el trabajo entre varios nodos de proceso.</span><span class="sxs-lookup"><span data-stu-id="6bf74-182">That way, Stream Analytics can distribute the job across multiple compute nodes.</span></span>

<span data-ttu-id="6bf74-183">Para los datos de entrada de Event Hubs, utilice la palabra clave `PARTITION BY` para dividir el trabajo de Stream Analytics en particiones.</span><span class="sxs-lookup"><span data-stu-id="6bf74-183">For Event Hubs input, use the `PARTITION BY` keyword to partition the Stream Analytics job.</span></span> <span data-ttu-id="6bf74-184">Los datos se dividirán en subconjuntos en función de las particiones de Event Hubs.</span><span class="sxs-lookup"><span data-stu-id="6bf74-184">The data will be divided into subsets based on the Event Hubs partitions.</span></span> 

<span data-ttu-id="6bf74-185">Las funciones de intervalos de tiempo y las combinaciones temporales requieren unidades de streaming adicionales.</span><span class="sxs-lookup"><span data-stu-id="6bf74-185">Windowing functions and temporal joins require additional SU.</span></span> <span data-ttu-id="6bf74-186">Cuando sea posible, use `PARTITION BY` para que cada partición se procese por separado.</span><span class="sxs-lookup"><span data-stu-id="6bf74-186">When possible, use `PARTITION BY` so that each partition is processed separately.</span></span> <span data-ttu-id="6bf74-187">Para más información, consulte [Descripción y ajuste de las unidades de streaming](/azure/stream-analytics/stream-analytics-streaming-unit-consumption#windowed-aggregates).</span><span class="sxs-lookup"><span data-stu-id="6bf74-187">For more information, see [Understand and adjust Streaming Units](/azure/stream-analytics/stream-analytics-streaming-unit-consumption#windowed-aggregates).</span></span>

<span data-ttu-id="6bf74-188">Si no es posible ejecutar todo el trabajo de Stream Analytics en paralelo, intente dividirlo en varios pasos, empezando con uno o varios pasos en paralelo.</span><span class="sxs-lookup"><span data-stu-id="6bf74-188">If it's not possible to parallelize the entire Stream Analytics job, try to break the job into multiple steps, starting with one or more parallel steps.</span></span> <span data-ttu-id="6bf74-189">De este modo, los primeros pasos que pueden ejecutar en paralelo.</span><span class="sxs-lookup"><span data-stu-id="6bf74-189">That way, the first steps can run in parallel.</span></span> <span data-ttu-id="6bf74-190">Por ejemplo, en esta arquitectura de referencia:</span><span class="sxs-lookup"><span data-stu-id="6bf74-190">For example, in this reference architecture:</span></span>

- <span data-ttu-id="6bf74-191">Los pasos 1 y 2 son instrucciones `SELECT` simples que seleccionan registros dentro de una sola partición.</span><span class="sxs-lookup"><span data-stu-id="6bf74-191">Steps 1 and 2 are simple `SELECT` statements that select records within a single partition.</span></span> 
- <span data-ttu-id="6bf74-192">El paso 3 realiza una combinación con particiones en los dos flujos de entrada.</span><span class="sxs-lookup"><span data-stu-id="6bf74-192">Step 3 performs a partitioned join across two input streams.</span></span> <span data-ttu-id="6bf74-193">Este paso aprovecha del hecho de que los registros que coinciden comparten la misma clave de partición y, por lo tanto, garantiza que tendrán el mismo identificador de partición en cada flujo de entrada.</span><span class="sxs-lookup"><span data-stu-id="6bf74-193">This step takes advantage of the fact that matching records share the same partition key, and so are guaranteed to have the same partition ID in each input stream.</span></span>
- <span data-ttu-id="6bf74-194">El paso 4 realiza la agregación en todas las particiones.</span><span class="sxs-lookup"><span data-stu-id="6bf74-194">Step 4 aggregates across all of the partitions.</span></span> <span data-ttu-id="6bf74-195">Este paso no se puede ejecutar en paralelo.</span><span class="sxs-lookup"><span data-stu-id="6bf74-195">This step cannot be parallelized.</span></span>

<span data-ttu-id="6bf74-196">Use el [diagrama de trabajo](/azure/stream-analytics/stream-analytics-job-diagram-with-metrics) de Stream Analytics para ver cuántas particiones se asignan a cada paso del trabajo.</span><span class="sxs-lookup"><span data-stu-id="6bf74-196">Use the Stream Analytics [job diagram](/azure/stream-analytics/stream-analytics-job-diagram-with-metrics) to see how many partitions are assigned to each step in the job.</span></span> <span data-ttu-id="6bf74-197">El siguiente diagrama muestra el diagrama de trabajo de esta arquitectura de referencia:</span><span class="sxs-lookup"><span data-stu-id="6bf74-197">The following diagram shows the job diagram for this reference architecture:</span></span>

![](./images/stream-processing-asa/job-diagram.png)

### <a name="cosmos-db"></a><span data-ttu-id="6bf74-198">Cosmos DB</span><span class="sxs-lookup"><span data-stu-id="6bf74-198">Cosmos DB</span></span>

<span data-ttu-id="6bf74-199">La capacidad de rendimiento de Cosmos DB se mide en [unidades de solicitud](/azure/cosmos-db/request-units) (RU).</span><span class="sxs-lookup"><span data-stu-id="6bf74-199">Throughput capacity for Cosmos DB is measured in [Request Units](/azure/cosmos-db/request-units) (RU).</span></span> <span data-ttu-id="6bf74-200">Para escalar un contenedor de Cosmos DB más allá de 10 000 RU, debe especificar una [clave de partición](/azure/cosmos-db/partition-data) al crear el contenedor, e incluir la clave de partición en todos los documentos.</span><span class="sxs-lookup"><span data-stu-id="6bf74-200">In order to scale a Cosmos DB container past 10,000 RU, you must specify a [partition key](/azure/cosmos-db/partition-data) when you create the container, and include the partition key in every document.</span></span> 

<span data-ttu-id="6bf74-201">En esta arquitectura de referencia, se crean nuevos documentos solo una vez por minuto (el intervalo de la ventana salto), por lo que los requisitos de procesamiento son bastante bajos.</span><span class="sxs-lookup"><span data-stu-id="6bf74-201">In this reference architecture, new documents are created only once per minute (the hopping window interval), so the throughput requirements are quite low.</span></span> <span data-ttu-id="6bf74-202">Por ese motivo, no es necesario asignar una clave de partición en este escenario.</span><span class="sxs-lookup"><span data-stu-id="6bf74-202">For that reason, there's no need to assign a partition key in this scenario.</span></span>

## <a name="monitoring-considerations"></a><span data-ttu-id="6bf74-203">Consideraciones sobre supervisión</span><span class="sxs-lookup"><span data-stu-id="6bf74-203">Monitoring considerations</span></span>

<span data-ttu-id="6bf74-204">Con cualquier solución de procesamiento de flujos, es importante supervisar el rendimiento y el estado del sistema.</span><span class="sxs-lookup"><span data-stu-id="6bf74-204">With any stream processing solution, it's important to monitor the performance and health of the system.</span></span> <span data-ttu-id="6bf74-205">[Azure Monitor](/azure/monitoring-and-diagnostics/) recopila métricas y registros de diagnóstico para los servicios de Azure que se utilizan en la arquitectura.</span><span class="sxs-lookup"><span data-stu-id="6bf74-205">[Azure Monitor](/azure/monitoring-and-diagnostics/) collects metrics and diagnostics logs for the Azure services used in the architecture.</span></span> <span data-ttu-id="6bf74-206">Azure Monitor se integra en la plataforma de Azure y no requiere ningún código adicional en la aplicación.</span><span class="sxs-lookup"><span data-stu-id="6bf74-206">Azure Monitor is built into the Azure platform and does not require any additional code in your application.</span></span>

<span data-ttu-id="6bf74-207">Cualquiera de las siguientes señales de advertencia indica que debe escalar horizontalmente el recurso de Azure correspondiente:</span><span class="sxs-lookup"><span data-stu-id="6bf74-207">Any of the following warning signals indicate that you should scale out the relevant Azure resource:</span></span>

- <span data-ttu-id="6bf74-208">Event Hubs limita las solicitudes o está cerca de la cuota diaria de mensajes.</span><span class="sxs-lookup"><span data-stu-id="6bf74-208">Event Hubs throttles requests or is close to the daily message quota.</span></span>
- <span data-ttu-id="6bf74-209">El trabajo de Stream Analytics utiliza sistemáticamente más del 80 % de las unidades de streaming (SU) asignadas.</span><span class="sxs-lookup"><span data-stu-id="6bf74-209">The Stream Analytics job consistently uses more than 80% of allocated Streaming Units (SU).</span></span>
- <span data-ttu-id="6bf74-210">Cosmos DB comienza a limitar las solicitudes.</span><span class="sxs-lookup"><span data-stu-id="6bf74-210">Cosmos DB begins to throttle requests.</span></span>

<span data-ttu-id="6bf74-211">La arquitectura de referencia incluye un panel personalizado, que se implementa en Azure Portal.</span><span class="sxs-lookup"><span data-stu-id="6bf74-211">The reference architecture includes a custom dashboard, which is deployed to the Azure portal.</span></span> <span data-ttu-id="6bf74-212">Después de implementar la arquitectura, abra [Azure Portal](https://portal.azure.com) y seleccione `TaxiRidesDashboard` en la lista de paneles para ver el panel.</span><span class="sxs-lookup"><span data-stu-id="6bf74-212">After you deploy the architecture, you can view the dashboard by opening the [Azure Portal](https://portal.azure.com) and selecting `TaxiRidesDashboard` from list of dashboards.</span></span> <span data-ttu-id="6bf74-213">Para más información sobre cómo crear e implementar paneles personalizados en Azure Portal, consulte [Creación mediante programación de paneles de Azure](/azure/azure-portal/azure-portal-dashboards-create-programmatically).</span><span class="sxs-lookup"><span data-stu-id="6bf74-213">For more information about creating and deploying custom dashboards in the Azure portal, see [Programmatically create Azure Dashboards](/azure/azure-portal/azure-portal-dashboards-create-programmatically).</span></span>

<span data-ttu-id="6bf74-214">La siguiente imagen muestra el panel después de que el trabajo de Stream Analytics se ejecutara durante una hora aproximadamente.</span><span class="sxs-lookup"><span data-stu-id="6bf74-214">The following image shows the dashboard after the Stream Analytics job ran for about an hour.</span></span>

![](./images/stream-processing-asa/asa-dashboard.png)

<span data-ttu-id="6bf74-215">El panel de la parte inferior izquierda muestra que el consumo de unidades de streaming para el trabajo de Stream Analytics aumenta durante los primeros 15 minutos y después se nivela.</span><span class="sxs-lookup"><span data-stu-id="6bf74-215">The panel on the lower left shows that the SU consumption for the Stream Analytics job climbs during the first 15 minutes and then levels off.</span></span> <span data-ttu-id="6bf74-216">Este es un patrón típico mientras el trabajo alcanza un estado estable.</span><span class="sxs-lookup"><span data-stu-id="6bf74-216">This is a typical pattern as the job reaches a steady state.</span></span> 

<span data-ttu-id="6bf74-217">Tenga en cuenta que Event Hubs limita las solicitudes, tal y como se muestra en el panel superior derecho.</span><span class="sxs-lookup"><span data-stu-id="6bf74-217">Notice that Event Hubs is throttling requests, shown in the upper right panel.</span></span> <span data-ttu-id="6bf74-218">Un solicitud limitada de vez en cuanto no es un problema, porque el SDK de cliente de Event Hubs realiza reintentos automáticamente cuando recibe un error de limitación.</span><span class="sxs-lookup"><span data-stu-id="6bf74-218">An occasional throttled request is not a problem, because the Event Hubs client SDK automatically retries when it receives a throttling error.</span></span> <span data-ttu-id="6bf74-219">Sin embargo, si se producen errores de limitación sistemáticamente, significa que el centro de eventos necesita más unidades de procesamiento.</span><span class="sxs-lookup"><span data-stu-id="6bf74-219">However, if you see consistent throttling errors, it means the event hub needs more throughput units.</span></span> <span data-ttu-id="6bf74-220">El gráfico siguiente muestra una ejecución de prueba con la característica de inflado automático de Event Hubs, que escala horizontalmente las unidades de procesamiento automáticamente cuando es necesario.</span><span class="sxs-lookup"><span data-stu-id="6bf74-220">The following graph shows a test run using the Event Hubs auto-inflate feature, which automatically scales out the throughput units as needed.</span></span> 

![](./images/stream-processing-asa/stream-processing-eh-autoscale.png)

<span data-ttu-id="6bf74-221">El inflado automático se habilitó en la marca de 06:35 aproximadamente.</span><span class="sxs-lookup"><span data-stu-id="6bf74-221">Auto-inflate was enabled at about the 06:35 mark.</span></span> <span data-ttu-id="6bf74-222">Puede ver la caída de solicitudes limitadas, porque Event Hubs escaló automáticamente a 3 unidades de procesamiento.</span><span class="sxs-lookup"><span data-stu-id="6bf74-222">You can see the p drop in throttled requests, as Event Hubs automatically scaled up to 3 throughput units.</span></span>

<span data-ttu-id="6bf74-223">Curiosamente, esto tuvo el efecto secundario de aumentar el uso de las unidades de streaming en el trabajo de Stream Analytics.</span><span class="sxs-lookup"><span data-stu-id="6bf74-223">Interestingly, this had the side effect of increasing the SU utilization in the Stream Analytics job.</span></span> <span data-ttu-id="6bf74-224">Con la limitación, Event Hubs redujo artificialmente la tasa de ingesta del trabajo de Stream Analytics.</span><span class="sxs-lookup"><span data-stu-id="6bf74-224">By throttling, Event Hubs was artificially reducing the ingestion rate for the Stream Analytics job.</span></span> <span data-ttu-id="6bf74-225">Es bastante habitual que, al resolver un cuello de botella de rendimiento, se produzca otro.</span><span class="sxs-lookup"><span data-stu-id="6bf74-225">It's actually common that resolving one performance bottleneck reveals another.</span></span> <span data-ttu-id="6bf74-226">En este caso, asignar unidades de streaming adicionales para el trabajo de Stream Analytics resolvió el problema.</span><span class="sxs-lookup"><span data-stu-id="6bf74-226">In this case, allocating additional SU for the Stream Analytics job resolved the issue.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="6bf74-227">Implementación de la solución</span><span class="sxs-lookup"><span data-stu-id="6bf74-227">Deploy the solution</span></span>

<span data-ttu-id="6bf74-228">Hay disponible una implementación de esta arquitectura de referencia en [GitHub](https://github.com/mspnp/reference-architectures/tree/master/data).</span><span class="sxs-lookup"><span data-stu-id="6bf74-228">A deployment for this reference architecture is available on [GitHub](https://github.com/mspnp/reference-architectures/tree/master/data).</span></span> 

### <a name="prerequisites"></a><span data-ttu-id="6bf74-229">Requisitos previos</span><span class="sxs-lookup"><span data-stu-id="6bf74-229">Prerequisites</span></span>

1. <span data-ttu-id="6bf74-230">Clone, bifurque o descargue el archivo zip del repositorio de GitHub de [arquitecturas de referencia](https://github.com/mspnp/reference-architectures).</span><span class="sxs-lookup"><span data-stu-id="6bf74-230">Clone, fork, or download the zip file for the [reference architectures](https://github.com/mspnp/reference-architectures) GitHub repository.</span></span>

2. <span data-ttu-id="6bf74-231">Instale [Docker](https://www.docker.com/) para ejecutar el generador de datos.</span><span class="sxs-lookup"><span data-stu-id="6bf74-231">Install [Docker](https://www.docker.com/) to run the data generator.</span></span>

3. <span data-ttu-id="6bf74-232">Instale la [CLI de Azure 2.0](/cli/azure/install-azure-cli?view=azure-cli-latest).</span><span class="sxs-lookup"><span data-stu-id="6bf74-232">Install [Azure CLI 2.0](/cli/azure/install-azure-cli?view=azure-cli-latest).</span></span>

4. <span data-ttu-id="6bf74-233">Desde un símbolo del sistema, un símbolo del sistema de Bash o un símbolo del sistema de PowerShell, inicie sesión en su cuenta de Azure como se indica a continuación:</span><span class="sxs-lookup"><span data-stu-id="6bf74-233">From a command prompt, bash prompt, or PowerShell prompt, sign into your Azure account as follows:</span></span>

    ```
    az login
    ```

### <a name="download-the-source-data-files"></a><span data-ttu-id="6bf74-234">Descarga de archivos de datos de origen</span><span class="sxs-lookup"><span data-stu-id="6bf74-234">Download the source data files</span></span>

1. <span data-ttu-id="6bf74-235">Cree un directorio llamado `DataFile` en el directorio `data/streaming_asa`, en el repositorio de GitHub.</span><span class="sxs-lookup"><span data-stu-id="6bf74-235">Create a directory named `DataFile` under the `data/streaming_asa` directory in the GitHub repo.</span></span>

2. <span data-ttu-id="6bf74-236">Abra un explorador web y vaya a https://uofi.app.box.com/v/NYCtaxidata/folder/2332219935.</span><span class="sxs-lookup"><span data-stu-id="6bf74-236">Open a web browser and navigate to https://uofi.app.box.com/v/NYCtaxidata/folder/2332219935.</span></span>

3. <span data-ttu-id="6bf74-237">Haga clic en el botón **Descargar** en esta página para descargar un archivo zip con todos los datos de taxi de ese año.</span><span class="sxs-lookup"><span data-stu-id="6bf74-237">Click the **Download** button on this page to download a zip file of all the taxi data for that year.</span></span>

4. <span data-ttu-id="6bf74-238">Extraiga el archivo zip en el directorio `DataFile` local.</span><span class="sxs-lookup"><span data-stu-id="6bf74-238">Extract the zip file to the `DataFile` directory.</span></span>

    > [!NOTE]
    > <span data-ttu-id="6bf74-239">Este archivo zip contiene otros archivos zip.</span><span class="sxs-lookup"><span data-stu-id="6bf74-239">This zip file contains other zip files.</span></span> <span data-ttu-id="6bf74-240">No extraiga los archivos zip secundarios.</span><span class="sxs-lookup"><span data-stu-id="6bf74-240">Don't extract the child zip files.</span></span>

<span data-ttu-id="6bf74-241">La estructura de directorios debe tener el siguiente aspecto:</span><span class="sxs-lookup"><span data-stu-id="6bf74-241">The directory structure should look like the following:</span></span>

```
/data
    /streaming_asa
        /DataFile
            /FOIL2013
                trip_data_1.zip
                trip_data_2.zip
                trip_data_3.zip
                ...
```

### <a name="deploy-the-azure-resources"></a><span data-ttu-id="6bf74-242">Implementación de los recursos de Azure</span><span class="sxs-lookup"><span data-stu-id="6bf74-242">Deploy the Azure resources</span></span>

1. <span data-ttu-id="6bf74-243">Desde un shell o símbolo del sistema de Windows, ejecute el siguiente comando y siga las indicaciones de inicio de sesión:</span><span class="sxs-lookup"><span data-stu-id="6bf74-243">From a shell or Windows Command Prompt, run the following command and follow the sign-in prompt:</span></span>

    ```bash
    az login
    ```

2. <span data-ttu-id="6bf74-244">Vaya a la carpeta `data/streaming_asa` del repositorio de GitHub.</span><span class="sxs-lookup"><span data-stu-id="6bf74-244">Navigate to the folder `data/streaming_asa` in the GitHub repository</span></span>

    ```bash
    cd data/streaming_asa
    ```

2. <span data-ttu-id="6bf74-245">Ejecute los siguientes comandos para implementar los recursos de Azure:</span><span class="sxs-lookup"><span data-stu-id="6bf74-245">Run the following commands to deploy the Azure resources:</span></span>

    ```bash
    export resourceGroup='[Resource group name]'
    export resourceLocation='[Location]'
    export cosmosDatabaseAccount='[Cosmos DB account name]'
    export cosmosDatabase='[Cosmod DB database name]'
    export cosmosDataBaseCollection='[Cosmos DB collection name]'
    export eventHubNamespace='[Event Hubs namespace name]'

    # Create a resource group
    az group create --name $resourceGroup --location $resourceLocation

    # Deploy resources
    az group deployment create --resource-group $resourceGroup \
      --template-file ./azure/deployresources.json --parameters \
      eventHubNamespace=$eventHubNamespace \
      outputCosmosDatabaseAccount=$cosmosDatabaseAccount \
      outputCosmosDatabase=$cosmosDatabase \
      outputCosmosDatabaseCollection=$cosmosDataBaseCollection

    # Create a database 
    az cosmosdb database create --name $cosmosDatabaseAccount \
        --db-name $cosmosDatabase --resource-group $resourceGroup

    # Create a collection
    az cosmosdb collection create --collection-name $cosmosDataBaseCollection \
        --name $cosmosDatabaseAccount --db-name $cosmosDatabase \
        --resource-group $resourceGroup
    ```

3. <span data-ttu-id="6bf74-246">En Azure Portal, vaya al grupo de recursos creado.</span><span class="sxs-lookup"><span data-stu-id="6bf74-246">In the Azure portal, navigate to the resource group that was created.</span></span>

4. <span data-ttu-id="6bf74-247">Abra la hoja del trabajo de Stream Analytics.</span><span class="sxs-lookup"><span data-stu-id="6bf74-247">Open the blade for the Stream Analytics job.</span></span>

5. <span data-ttu-id="6bf74-248">Haga clic en **Iniciar** para iniciar el trabajo.</span><span class="sxs-lookup"><span data-stu-id="6bf74-248">Click **Start** to start the job.</span></span> <span data-ttu-id="6bf74-249">Seleccione **Ahora** como hora de inicio de la salida.</span><span class="sxs-lookup"><span data-stu-id="6bf74-249">Select **Now** as the output start time.</span></span> <span data-ttu-id="6bf74-250">Espere a que el trabajo se inicie.</span><span class="sxs-lookup"><span data-stu-id="6bf74-250">Wait for the job to start.</span></span>

### <a name="run-the-data-generator"></a><span data-ttu-id="6bf74-251">Ejecución del generador de datos</span><span class="sxs-lookup"><span data-stu-id="6bf74-251">Run the data generator</span></span>

1. <span data-ttu-id="6bf74-252">Obtenga la cadena de conexión del centro de eventos.</span><span class="sxs-lookup"><span data-stu-id="6bf74-252">Get the Event Hub connection strings.</span></span> <span data-ttu-id="6bf74-253">Puede obtener esta información en Azure Portal o con los siguientes comandos de la CLI:</span><span class="sxs-lookup"><span data-stu-id="6bf74-253">You can get these from the Azure portal, or by running the following CLI commands:</span></span>

    ```bash
    # RIDE_EVENT_HUB
    az eventhubs eventhub authorization-rule keys list \
        --eventhub-name taxi-ride \
        --name taxi-ride-asa-access-policy \
        --namespace-name $eventHubNamespace \
        --resource-group $resourceGroup \
        --query primaryConnectionString

    # FARE_EVENT_HUB
    az eventhubs eventhub authorization-rule keys list \
        --eventhub-name taxi-fare \
        --name taxi-fare-asa-access-policy \
        --namespace-name $eventHubNamespace \
        --resource-group $resourceGroup \
        --query primaryConnectionString
    ```

2. <span data-ttu-id="6bf74-254">Vaya al directorio `data/streaming_asa/onprem` del repositorio de GitHub.</span><span class="sxs-lookup"><span data-stu-id="6bf74-254">Navigate to the directory `data/streaming_asa/onprem` in the GitHub repository</span></span>

3. <span data-ttu-id="6bf74-255">Actualice los valores en el archivo `main.env` de la siguiente manera:</span><span class="sxs-lookup"><span data-stu-id="6bf74-255">Update the values in the file `main.env` as follows:</span></span>

    ```
    RIDE_EVENT_HUB=[Connection string for taxi-ride event hub]
    FARE_EVENT_HUB=[Connection string for taxi-fare event hub]
    RIDE_DATA_FILE_PATH=/DataFile/FOIL2013
    MINUTES_TO_LEAD=0
    PUSH_RIDE_DATA_FIRST=false
    ```

4. <span data-ttu-id="6bf74-256">Ejecute el siguiente comando para compilar la imagen de Docker.</span><span class="sxs-lookup"><span data-stu-id="6bf74-256">Run the following command to build the Docker image.</span></span>

    ```bash
    docker build --no-cache -t dataloader .
    ```

5. <span data-ttu-id="6bf74-257">Vuelva al directorio principal, `data/stream_asa`.</span><span class="sxs-lookup"><span data-stu-id="6bf74-257">Navigate back to the parent directory, `data/stream_asa`.</span></span>

    ```bash
    cd ..
    ```

6. <span data-ttu-id="6bf74-258">Ejecute el siguiente comando para ejecutar la imagen de Docker.</span><span class="sxs-lookup"><span data-stu-id="6bf74-258">Run the following command to run the Docker image.</span></span>

    ```bash
    docker run -v `pwd`/DataFile:/DataFile --env-file=onprem/main.env dataloader:latest
    ```

<span data-ttu-id="6bf74-259">La salida debe tener un aspecto similar al siguiente:</span><span class="sxs-lookup"><span data-stu-id="6bf74-259">The output should look like the following:</span></span>

```
Created 10000 records for TaxiFare
Created 10000 records for TaxiRide
Created 20000 records for TaxiFare
Created 20000 records for TaxiRide
Created 30000 records for TaxiFare
...
```

<span data-ttu-id="6bf74-260">Deje que el programa se ejecute durante al menos 5 minutos, que es el intervalo definido en la consulta de Stream Analytics.</span><span class="sxs-lookup"><span data-stu-id="6bf74-260">Let the program run for at least 5 minutes, which is the window defined in the Stream Analytics query.</span></span> <span data-ttu-id="6bf74-261">Para comprobar que el trabajo de Stream Analytics se ejecuta correctamente, abra Azure Portal y vaya a la base de datos de Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="6bf74-261">To verify the Stream Analytics job is running correctly, open the Azure portal and navigate to the Cosmos DB database.</span></span> <span data-ttu-id="6bf74-262">Abra la hoja **Explorador de datos** y vea los documentos.</span><span class="sxs-lookup"><span data-stu-id="6bf74-262">Open the **Data Explorer** blade and view the documents.</span></span> 

<span data-ttu-id="6bf74-263">[1] <span id="note1">Donovan, Brian; Work, Dan (2016): New York City Taxi Trip Data (2010-2013).</span><span class="sxs-lookup"><span data-stu-id="6bf74-263">[1] <span id="note1">Donovan, Brian; Work, Dan (2016): New York City Taxi Trip Data (2010-2013).</span></span> <span data-ttu-id="6bf74-264">Universidad de Illinois en Urbana-Champaign.</span><span class="sxs-lookup"><span data-stu-id="6bf74-264">University of Illinois at Urbana-Champaign.</span></span> <span data-ttu-id="6bf74-265">https://doi.org/10.13012/J8PN93H8</span><span class="sxs-lookup"><span data-stu-id="6bf74-265">https://doi.org/10.13012/J8PN93H8</span></span>
