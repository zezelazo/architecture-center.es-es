---
title: Procesamiento de flujos de datos con Azure Databricks
description: Creación de una canalización de procesamiento de flujos de datos de un extremo a otro en Azure mediante Azure Databricks
author: petertaylor9999
ms.date: 11/30/2018
ms.openlocfilehash: 0640e900c212d2b75cc9cdd5bec3a4f7c050490d
ms.sourcegitcommit: e7e0e0282fa93f0063da3b57128ade395a9c1ef9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/05/2018
ms.locfileid: "52902840"
---
# <a name="stream-processing-with-azure-databricks"></a><span data-ttu-id="9bcb7-103">Procesamiento de flujos de datos con Azure Databricks</span><span class="sxs-lookup"><span data-stu-id="9bcb7-103">Stream processing with Azure Databricks</span></span>

<span data-ttu-id="9bcb7-104">Esta arquitectura de referencia muestra una canalización de [procesamiento de flujos de datos](/azure/architecture/data-guide/big-data/real-time-processing) de un extremo a otro.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-104">This reference architecture shows an end-to-end [stream processing](/azure/architecture/data-guide/big-data/real-time-processing) pipeline.</span></span> <span data-ttu-id="9bcb7-105">Este tipo de canalización tiene cuatro fases: ingesta, proceso, almacenamiento y análisis e informes.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-105">This type of pipeline has four stages: ingest, process, store, and analysis and reporting.</span></span> <span data-ttu-id="9bcb7-106">Para esta arquitectura de referencia, la canalización ingiere datos de dos orígenes, realiza una combinación de los registros relacionados de cada flujo, enriquece el resultado y calcula un promedio en tiempo real.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-106">For this reference architecture, the pipeline ingests data from two sources, performs a join on related records from each stream, enriches the result, and calculates an average in real time.</span></span> <span data-ttu-id="9bcb7-107">Los resultados se almacenan para su posterior análisis.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-107">The results are stored for further analysis.</span></span> [<span data-ttu-id="9bcb7-108">**Implemente esta solución**.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-108">**Deploy this solution**.</span></span>](#deploy-the-solution)

![](./images/stream-processing-databricks.png)

<span data-ttu-id="9bcb7-109">**Escenario**: una empresa de taxi recopila los datos acerca de cada carrera de taxi.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-109">**Scenario**: A taxi company collects data about each taxi trip.</span></span> <span data-ttu-id="9bcb7-110">En este escenario, se supone que hay dos dispositivos independientes que envían datos.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-110">For this scenario, we assume there are two separate devices sending data.</span></span> <span data-ttu-id="9bcb7-111">El taxi tienen un medidor que envía la información acerca de cada carrera: duración, distancia y ubicaciones de recogida y destino.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-111">The taxi has a meter that sends information about each ride &mdash; the duration, distance, and pickup and dropoff locations.</span></span> <span data-ttu-id="9bcb7-112">Un dispositivo independiente acepta los pagos de clientes y envía los datos sobre las tarifas.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-112">A separate device accepts payments from customers and sends data about fares.</span></span> <span data-ttu-id="9bcb7-113">Con el fin de identificar las tendencias, la empresa de taxi desea calcular el promedio de propinas por milla conducida, en tiempo real, en cada barrio.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-113">To spot ridership trends, the taxi company wants to calculate the average tip per mile driven, in real time, for each neighborhood.</span></span>

## <a name="architecture"></a><span data-ttu-id="9bcb7-114">Arquitectura</span><span class="sxs-lookup"><span data-stu-id="9bcb7-114">Architecture</span></span>

<span data-ttu-id="9bcb7-115">La arquitectura consta de los siguientes componentes:</span><span class="sxs-lookup"><span data-stu-id="9bcb7-115">The architecture consists of the following components.</span></span>

<span data-ttu-id="9bcb7-116">**Orígenes de datos**.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-116">**Data sources**.</span></span> <span data-ttu-id="9bcb7-117">En esta arquitectura, hay dos orígenes de datos que generan flujos de datos en tiempo real.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-117">In this architecture, there are two data sources that generate data streams in real time.</span></span> <span data-ttu-id="9bcb7-118">El primer flujo de datos contiene información sobre la carrera y, el segundo, contiene información sobre las tarifas.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-118">The first stream contains ride information, and the second contains fare information.</span></span> <span data-ttu-id="9bcb7-119">La arquitectura de referencia incluye un generador de datos simulados que lee un conjunto de archivos estáticos e inserta los datos en Event Hubs.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-119">The reference architecture includes a simulated data generator that reads from a set of static files and pushes the data to Event Hubs.</span></span> <span data-ttu-id="9bcb7-120">En una aplicación real, los orígenes de datos serían los dispositivos instalados en el taxi.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-120">The data sources in a real application would be devices installed in the taxi cabs.</span></span>

<span data-ttu-id="9bcb7-121">**Azure Event Hubs**.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-121">**Azure Event Hubs**.</span></span> <span data-ttu-id="9bcb7-122">[Event Hubs](/azure/event-hubs/) es un servicio de ingesta de eventos.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-122">[Event Hubs](/azure/event-hubs/) is an event ingestion service.</span></span> <span data-ttu-id="9bcb7-123">Esta arquitectura emplea dos instancias de centro de eventos, uno para cada origen de datos.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-123">This architecture uses two event hub instances, one for each data source.</span></span> <span data-ttu-id="9bcb7-124">Cada origen de datos envía un flujo de datos al centro de eventos asociado.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-124">Each data source sends a stream of data to the associated event hub.</span></span>

<span data-ttu-id="9bcb7-125">**Azure Databricks**.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-125">**Azure Databricks**.</span></span> <span data-ttu-id="9bcb7-126">[Azure Databricks](/azure/azure-databricks/) es una plataforma de análisis basada en Apache Spark optimizada para la plataforma de servicios en la nube de Microsoft Azure.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-126">[Databricks](/azure/azure-databricks/) is an Apache Spark-based analytics platform optimized for the Microsoft Azure cloud services platform.</span></span> <span data-ttu-id="9bcb7-127">Databricks se utiliza para correlacionar los datos de la carrera de taxi y las tarifas, y también para enriquecer los datos correlacionados con los datos de los barrios almacenados en el sistema de archivos de Databricks.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-127">Databricks is used to correlate of the taxi ride and fare data, and also to enrich the correlated data with neighborhood data stored in the Databricks file system.</span></span>

<span data-ttu-id="9bcb7-128">**Cosmos DB**.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-128">**Cosmos DB**.</span></span> <span data-ttu-id="9bcb7-129">La salida del trabajo de Azure Databricks es una serie de registros, que se escriben en [Cosmos DB](/azure/cosmos-db/) mediante Cassandra API.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-129">The output from Azure Databricks job is a series of records, which are written to [Cosmos DB](/azure/cosmos-db/) using the Cassandra API.</span></span> <span data-ttu-id="9bcb7-130">Se usa Cassandra API porque admite el modelado de datos de series temporales.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-130">The Cassandra API is used because it supports time series data modeling.</span></span>

<span data-ttu-id="9bcb7-131">**Azure Log Analytics**.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-131">**Azure Log Analytics**.</span></span> <span data-ttu-id="9bcb7-132">Los datos de registro de aplicaciones que recopila [Azure Monitor](/azure/monitoring-and-diagnostics/) se almacenan en un [área de trabajo de Log Analytics](/azure/log-analytics).</span><span class="sxs-lookup"><span data-stu-id="9bcb7-132">Application log data collected by [Azure Monitor](/azure/monitoring-and-diagnostics/) is stored in a [Log Analytics workspace](/azure/log-analytics).</span></span> <span data-ttu-id="9bcb7-133">Las consultas de Log Analytics pueden usarse para analizar y ver las métricas, e inspeccionar los mensajes de registro para identificar problemas dentro de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-133">Log Analytics queries can be used to analyze and visualize metrics and inspect log messages to identify issues within the application.</span></span>

## <a name="data-ingestion"></a><span data-ttu-id="9bcb7-134">Ingesta de datos</span><span class="sxs-lookup"><span data-stu-id="9bcb7-134">Data ingestion</span></span>

<span data-ttu-id="9bcb7-135">Para simular un origen de datos, esta arquitectura de referencia usa el conjunto de datos [New York City Taxi Data](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797)<sup>[[1]](#note1)</sup>.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-135">To simulate a data source, this reference architecture uses the [New York City Taxi Data](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797) dataset<sup>[[1]](#note1)</sup>.</span></span> <span data-ttu-id="9bcb7-136">Este conjunto de datos contiene datos acerca de carreras de taxi en la ciudad de Nueva York durante un período de cuatro años (de 2010 a 2013).</span><span class="sxs-lookup"><span data-stu-id="9bcb7-136">This dataset contains data about taxi trips in New York City over a four-year period (2010 &ndash; 2013).</span></span> <span data-ttu-id="9bcb7-137">Contiene dos tipos de registros: datos de carreras y datos de tarifas.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-137">It contains two types of record: Ride data and fare data.</span></span> <span data-ttu-id="9bcb7-138">Los datos de carreras incluyen la duración del viaje, la distancia de viaje y la ubicación de recogida y destino.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-138">Ride data includes trip duration, trip distance, and pickup and dropoff location.</span></span> <span data-ttu-id="9bcb7-139">Los datos de tarifas incluyen las tarifas, los impuestos y las propinas.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-139">Fare data includes fare, tax, and tip amounts.</span></span> <span data-ttu-id="9bcb7-140">Los campos comunes en ambos tipos de registro son la placa y el número de licencia, y el identificador del proveedor.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-140">Common fields in both record types include medallion number, hack license, and vendor ID.</span></span> <span data-ttu-id="9bcb7-141">Juntos, estos tres campos identifican un taxi además del conductor.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-141">Together these three fields uniquely identify a taxi plus a driver.</span></span> <span data-ttu-id="9bcb7-142">Los datos se almacenan en formato CSV.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-142">The data is stored in CSV format.</span></span> 

<span data-ttu-id="9bcb7-143">El generador de datos es una aplicación de .NET Core que lee los registros y los envía a Azure Event Hubs.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-143">The data generator is a .NET Core application that reads the records and sends them to Azure Event Hubs.</span></span> <span data-ttu-id="9bcb7-144">El generador envía los datos de carreras en formato JSON y los datos de tarifas en formato CSV.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-144">The generator sends ride data in JSON format and fare data in CSV format.</span></span> 

<span data-ttu-id="9bcb7-145">Event Hubs usa [particiones](/azure/event-hubs/event-hubs-features#partitions) para segmentar los datos.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-145">Event Hubs uses [partitions](/azure/event-hubs/event-hubs-features#partitions) to segment the data.</span></span> <span data-ttu-id="9bcb7-146">Las particiones permiten a los consumidores leer cada partición en paralelo.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-146">Partitions allow a consumer to read each partition in parallel.</span></span> <span data-ttu-id="9bcb7-147">Cuando se envían datos a Event Hubs, puede especificar explícitamente la clave de partición.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-147">When you send data to Event Hubs, you can specify the partition key explicitly.</span></span> <span data-ttu-id="9bcb7-148">En caso contrario, los registros se asignan a las particiones en modo round-robin.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-148">Otherwise, records are assigned to partitions in round-robin fashion.</span></span> 

<span data-ttu-id="9bcb7-149">En este escenario, los datos de carreras y los datos de tarifas deben terminar con el mismo identificador de partición para un taxi determinado.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-149">In this scenario, ride data and fare data should end up with the same partition ID for a given taxi cab.</span></span> <span data-ttu-id="9bcb7-150">Esto permite a Databricks aplicar un cierto paralelismo cuando se establece una correlación entre los dos flujos.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-150">This enables Databricks to apply a degree of parallelism when it correlates the two streams.</span></span> <span data-ttu-id="9bcb7-151">Un registro en la partición *n* de los datos de carreras coincidirá con un registro en la partición de datos *n* de los datos de tarifas.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-151">A record in partition *n* of the ride data will match a record in partition *n* of the fare data.</span></span>

![](./images/stream-processing-databricks-eh.png)

<span data-ttu-id="9bcb7-152">En el generador de datos, el modelo de datos común para ambos tipos de registro tiene una propiedad `PartitionKey` que es la concatenación de `Medallion`, `HackLicense` y `VendorId`.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-152">In the data generator, the common data model for both record types has a `PartitionKey` property that is the concatenation of `Medallion`, `HackLicense`, and `VendorId`.</span></span>

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

<span data-ttu-id="9bcb7-153">Esta propiedad se utiliza para proporcionar una clave de partición explícita cuando se realizan envíos a Event Hubs:</span><span class="sxs-lookup"><span data-stu-id="9bcb7-153">This property is used to provide an explicit partition key when sending to Event Hubs:</span></span>

```csharp
using (var client = pool.GetObject())
{
    return client.Value.SendAsync(new EventData(Encoding.UTF8.GetBytes(
        t.GetData(dataFormat))), t.PartitionKey);
}
```

### <a name="event-hubs"></a><span data-ttu-id="9bcb7-154">Event Hubs</span><span class="sxs-lookup"><span data-stu-id="9bcb7-154">Event Hubs</span></span>

<span data-ttu-id="9bcb7-155">La capacidad de procesamiento de Event Hubs se mide en [unidades de procesamiento](/azure/event-hubs/event-hubs-features#throughput-units).</span><span class="sxs-lookup"><span data-stu-id="9bcb7-155">The throughput capacity of Event Hubs is measured in [throughput units](/azure/event-hubs/event-hubs-features#throughput-units).</span></span> <span data-ttu-id="9bcb7-156">Para escalar un centro de eventos automáticamente, puede habilitar el [inflado automático](/azure/event-hubs/event-hubs-auto-inflate), que permite escalar las unidades de procesamiento en función del tráfico, hasta un máximo configurado.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-156">You can autoscale an event hub by enabling [auto-inflate](/azure/event-hubs/event-hubs-auto-inflate), which automatically scales the throughput units based on traffic, up to a configured maximum.</span></span> 

## <a name="stream-processing"></a><span data-ttu-id="9bcb7-157">Procesamiento de flujos</span><span class="sxs-lookup"><span data-stu-id="9bcb7-157">Stream processing</span></span>

<span data-ttu-id="9bcb7-158">En Azure Databricks, el procesamiento de datos se realiza mediante un trabajo.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-158">In Azure Databricks, data processing is performed by a job.</span></span> <span data-ttu-id="9bcb7-159">El trabajo está asignado a un clúster y se ejecuta en él.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-159">The job is assigned to and runs on a cluster.</span></span> <span data-ttu-id="9bcb7-160">El trabajo puede ser código personalizado escrito en Java o un [cuaderno](https://docs.databricks.com/user-guide/notebooks/index.html) de Spark.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-160">The job can either be custom code written in Java, or a Spark [notebook](https://docs.databricks.com/user-guide/notebooks/index.html).</span></span>

<span data-ttu-id="9bcb7-161">En esta arquitectura de referencia, el trabajo es un archivo de Java con clases escritas tanto en Scala como en Java.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-161">In this reference architecture, the job is a Java archive with classes written in both Java and Scala.</span></span> <span data-ttu-id="9bcb7-162">Al especificar el archivo de Java para un trabajo de Databricks, se especifica la clase para que el clúster de Databricks lo ejecute.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-162">When specifying the Java archive for a Databricks job, the class is specified for execution by the Databricks cluster.</span></span> <span data-ttu-id="9bcb7-163">En este caso, el método **main** de la clase **com.microsoft.pnp.TaxiCabReader** contiene la lógica de procesamiento de datos.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-163">Here, the **main** method of the **com.microsoft.pnp.TaxiCabReader** class contains the data processing logic.</span></span> 

### <a name="reading-the-stream-from-the-two-event-hub-instances"></a><span data-ttu-id="9bcb7-164">Lectura del flujo de datos de las dos instancias del centro de eventos</span><span class="sxs-lookup"><span data-stu-id="9bcb7-164">Reading the stream from the two event hub instances</span></span>

<span data-ttu-id="9bcb7-165">La lógica de procesamiento de datos usa [streaming estructurado de Spark](https://spark.apache.org/docs/2.1.2/structured-streaming-programming-guide.html) para leer de las dos instancias de Azure Event Hub:</span><span class="sxs-lookup"><span data-stu-id="9bcb7-165">The data processing logic uses [Spark structured streaming](https://spark.apache.org/docs/2.1.2/structured-streaming-programming-guide.html) to read from the two Azure event hub instances:</span></span>

```scala
val rideEventHubOptions = EventHubsConf(rideEventHubConnectionString)
      .setConsumerGroup(conf.taxiRideConsumerGroup())
      .setStartingPosition(EventPosition.fromStartOfStream)
    val rideEvents = spark.readStream
      .format("eventhubs")
      .options(rideEventHubOptions.toMap)
      .load

    val fareEventHubOptions = EventHubsConf(fareEventHubConnectionString)
      .setConsumerGroup(conf.taxiFareConsumerGroup())
      .setStartingPosition(EventPosition.fromStartOfStream)
    val fareEvents = spark.readStream
      .format("eventhubs")
      .options(fareEventHubOptions.toMap)
      .load
```

### <a name="enriching-the-data-with-the-neighborhood-information"></a><span data-ttu-id="9bcb7-166">Enriquecimiento de los datos con la información de los barrios</span><span class="sxs-lookup"><span data-stu-id="9bcb7-166">Enriching the data with the neighborhood information</span></span>

<span data-ttu-id="9bcb7-167">Los datos de las carreras incluyen las coordenadas de latitud y longitud de las ubicaciones de origen y destino.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-167">The ride data includes the latitude and longitude coordinates of the pick up and drop off locations.</span></span> <span data-ttu-id="9bcb7-168">Aunque estas coordenadas son útiles, no se consumen fácilmente en el análisis.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-168">While these coordinates are useful, they are not easily consumed for analysis.</span></span> <span data-ttu-id="9bcb7-169">Por lo tanto, estos datos se enriquecen con los datos de los barrios, que se leen desde un [archivo de forma](https://en.wikipedia.org/wiki/Shapefile).</span><span class="sxs-lookup"><span data-stu-id="9bcb7-169">Therefore, this data is enriched with neighborhood data that is read from a [shapefile](https://en.wikipedia.org/wiki/Shapefile).</span></span> 

<span data-ttu-id="9bcb7-170">El formato del archivo de forma es binario y no se analiza fácilmente, pero la biblioteca [GeoTools](http://geotools.org/) proporciona herramientas para que los datos geoespaciales usen el archivo de forma.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-170">The shapefile format is binary and not easily parsed, but the [GeoTools](http://geotools.org/) library provides tools for geospatial data that use the shapefile format.</span></span> <span data-ttu-id="9bcb7-171">Esta biblioteca se utiliza en la clase **com.microsoft.pnp.GeoFinder** para determinar el nombre del barrio en función de las coordenadas de origen y destino.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-171">This library is used in the **com.microsoft.pnp.GeoFinder** class to determine the neighborhood name based on the pick up and drop off coordinates.</span></span> 

```scala
val neighborhoodFinder = (lon: Double, lat: Double) => {
      NeighborhoodFinder.getNeighborhood(lon, lat).get()
    }
```

### <a name="joining-the-ride-and-fare-data"></a><span data-ttu-id="9bcb7-172">Combinación de los datos de carreras y tarifas</span><span class="sxs-lookup"><span data-stu-id="9bcb7-172">Joining the ride and fare data</span></span>

<span data-ttu-id="9bcb7-173">En primer lugar, se transforman los datos de carreras y tarifas:</span><span class="sxs-lookup"><span data-stu-id="9bcb7-173">First the ride and fare data is transformed:</span></span>

```scala
    val rides = transformedRides
      .filter(r => {
        if (r.isNullAt(r.fieldIndex("errorMessage"))) {
          true
        }
        else {
          malformedRides.add(1)
          false
        }
      })
      .select(
        $"ride.*",
        to_neighborhood($"ride.pickupLon", $"ride.pickupLat")
          .as("pickupNeighborhood"),
        to_neighborhood($"ride.dropoffLon", $"ride.dropoffLat")
          .as("dropoffNeighborhood")
      )
      .withWatermark("pickupTime", conf.taxiRideWatermarkInterval())

    val fares = transformedFares
      .filter(r => {
        if (r.isNullAt(r.fieldIndex("errorMessage"))) {
          true
        }
        else {
          malformedFares.add(1)
          false
        }
      })
      .select(
        $"fare.*",
        $"pickupTime"
      )
      .withWatermark("pickupTime", conf.taxiFareWatermarkInterval())
```

<span data-ttu-id="9bcb7-174">Y, a continuación, los datos de las carreras se combinan con los datos de las tarifas:</span><span class="sxs-lookup"><span data-stu-id="9bcb7-174">And then the ride data is joined with the fare data:</span></span>

```scala
val mergedTaxiTrip = rides.join(fares, Seq("medallion", "hackLicense", "vendorId", "pickupTime"))
```

### <a name="processing-the-data-and-inserting-into-cosmos-db"></a><span data-ttu-id="9bcb7-175">Procesamiento de los datos e inserción en Cosmos DB</span><span class="sxs-lookup"><span data-stu-id="9bcb7-175">Processing the data and inserting into Cosmos DB</span></span>

<span data-ttu-id="9bcb7-176">La tarifa media para cada barrio se calcula para un intervalo de tiempo determinado:</span><span class="sxs-lookup"><span data-stu-id="9bcb7-176">The average fare amount for each neighborhood is calculated for a given time interval:</span></span>

```scala
val maxAvgFarePerNeighborhood = mergedTaxiTrip.selectExpr("medallion", "hackLicense", "vendorId", "pickupTime", "rateCode", "storeAndForwardFlag", "dropoffTime", "passengerCount", "tripTimeInSeconds", "tripDistanceInMiles", "pickupLon", "pickupLat", "dropoffLon", "dropoffLat", "paymentType", "fareAmount", "surcharge", "mtaTax", "tipAmount", "tollsAmount", "totalAmount", "pickupNeighborhood", "dropoffNeighborhood")
      .groupBy(window($"pickupTime", conf.windowInterval()), $"pickupNeighborhood")
      .agg(
        count("*").as("rideCount"),
        sum($"fareAmount").as("totalFareAmount"),
        sum($"tipAmount").as("totalTipAmount")
      )
      .select($"window.start", $"window.end", $"pickupNeighborhood", $"rideCount", $"totalFareAmount", $"totalTipAmount")
```

<span data-ttu-id="9bcb7-177">Después, se inserta en Cosmos DB:</span><span class="sxs-lookup"><span data-stu-id="9bcb7-177">Which is then inserted into Cosmos DB:</span></span>

```scala
maxAvgFarePerNeighborhood
      .writeStream
      .queryName("maxAvgFarePerNeighborhood_cassandra_insert")
      .outputMode(OutputMode.Append())
      .foreach(new CassandraSinkForeach(connector))
      .start()
      .awaitTermination()
```

## <a name="security-considerations"></a><span data-ttu-id="9bcb7-178">Consideraciones sobre la seguridad</span><span class="sxs-lookup"><span data-stu-id="9bcb7-178">Security considerations</span></span>

<span data-ttu-id="9bcb7-179">El acceso al área de trabajo de la base de datos de Azure se controla mediante la [consola de administrador](https://docs.databricks.com/administration-guide/admin-settings/index.html).</span><span class="sxs-lookup"><span data-stu-id="9bcb7-179">Access to the Azure Database workspace is controlled using the [administrator console](https://docs.databricks.com/administration-guide/admin-settings/index.html).</span></span> <span data-ttu-id="9bcb7-180">La consola de administrador incluye funcionalidad para agregar usuarios, administrar permisos de usuario y configurar el inicio de sesión único.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-180">The administrator console includes functionality to add users, manage user permissions, and set up single sign-on.</span></span> <span data-ttu-id="9bcb7-181">El control del acceso para las áreas de trabajo, los clústeres, los trabajos y las tablas también se puede establecer en la consola de administrador.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-181">Access control for workspaces, clusters, jobs, and tables can also be set through the administrator console.</span></span>

### <a name="managing-secrets"></a><span data-ttu-id="9bcb7-182">Administración de secretos</span><span class="sxs-lookup"><span data-stu-id="9bcb7-182">Managing secrets</span></span>

<span data-ttu-id="9bcb7-183">Azure Databricks incluye un [almacén de secretos](https://docs.azuredatabricks.net/user-guide/secrets/index.html) que se usa para almacenar secretos, incluidas las cadenas de conexión, las claves de acceso, los nombres de usuario y las contraseñas.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-183">Azure Databricks includes a [secret store](https://docs.azuredatabricks.net/user-guide/secrets/index.html) that is used to store secrets, including connection strings, access keys, user names, and passwords.</span></span> <span data-ttu-id="9bcb7-184">Los secretos del almacén de secretos de Azure Databricks se particionan por **ámbitos**:</span><span class="sxs-lookup"><span data-stu-id="9bcb7-184">Secrets within the Azure Databricks secret store are partitioned by **scopes**:</span></span>

```bash
databricks secrets create-scope --scope "azure-databricks-job"
```

<span data-ttu-id="9bcb7-185">Los secretos se agregan en el nivel de ámbito:</span><span class="sxs-lookup"><span data-stu-id="9bcb7-185">Secrets are added at the scope level:</span></span>

```bash
databricks secrets put --scope "azure-databricks-job" --key "taxi-ride"
```

> [!NOTE]
> <span data-ttu-id="9bcb7-186">Se puede utilizar un ámbito respaldado por Azure Key Vault en lugar del ámbito nativo de Azure Databricks.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-186">An Azure Key Vault-backed scope can be used instead of the native Azure Databricks scope.</span></span> <span data-ttu-id="9bcb7-187">Para más información, consulte el artículo sobre los [ámbitos respaldados por Azure Key Vault](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html#azure-key-vault-backed-scopes).</span><span class="sxs-lookup"><span data-stu-id="9bcb7-187">To learn more, see [Azure Key Vault-backed scopes](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html#azure-key-vault-backed-scopes).</span></span>

<span data-ttu-id="9bcb7-188">En el código, se obtiene acceso a los secretos mediante las [utilidades de secretos](https://docs.databricks.com/user-guide/dev-tools/dbutils.html#secrets-utilities) de Azure Databricks.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-188">In code, secrets are accessed via the Azure Databricks [secrets utilities](https://docs.databricks.com/user-guide/dev-tools/dbutils.html#secrets-utilities).</span></span>


## <a name="monitoring-considerations"></a><span data-ttu-id="9bcb7-189">Consideraciones sobre supervisión</span><span class="sxs-lookup"><span data-stu-id="9bcb7-189">Monitoring considerations</span></span>

<span data-ttu-id="9bcb7-190">Azure Databricks se basa en Apache Spark, y ambos usan [log4j](https://logging.apache.org/log4j/2.x/) como la biblioteca estándar para el registro.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-190">Azure Databricks is based on Apache Spark, and both use [log4j](https://logging.apache.org/log4j/2.x/) as the standard library for logging.</span></span> <span data-ttu-id="9bcb7-191">Además del registro predeterminado proporcionado por Apache Spark, esta arquitectura de referencia envía registros y métricas a [Azure Log Analytics](/azure/log-analytics/).</span><span class="sxs-lookup"><span data-stu-id="9bcb7-191">In addition to the default logging provided by Apache Spark, this reference architecture sends logs and metrics to [Azure Log Analytics](/azure/log-analytics/).</span></span>

<span data-ttu-id="9bcb7-192">La clase **com.microsoft.pnp.TaxiCabReader** configura el sistema de registro de Apache Spark para enviar sus registros a Azure Log Analytics usando los valores del archivo **log4j.properties**.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-192">The **com.microsoft.pnp.TaxiCabReader** class configures the Apache Spark logging system to send its logs to Azure Log Analytics using the values in the **log4j.properties** file.</span></span> <span data-ttu-id="9bcb7-193">Mientras que los mensajes del registrador de Apache Spark son cadenas, Azure Log Analytics requiere que los mensajes de registro tengan el formato JSON.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-193">While the Apache Spark logger messages are strings, Azure Log Analytics requires log messages to be formatted as JSON.</span></span> <span data-ttu-id="9bcb7-194">La clase **com.microsoft.pnp.log4j.LogAnalyticsAppender** transforma estos mensajes al formato JSON:</span><span class="sxs-lookup"><span data-stu-id="9bcb7-194">The **com.microsoft.pnp.log4j.LogAnalyticsAppender** class transforms these messages to JSON:</span></span>

```scala

    @Override
    protected void append(LoggingEvent loggingEvent) {
        if (this.layout == null) {
            this.setLayout(new JSONLayout());
        }

        String json = this.getLayout().format(loggingEvent);
        try {
            this.client.send(json, this.logType);
        } catch(IOException ioe) {
            LogLog.warn("Error sending LoggingEvent to Log Analytics", ioe);
        }
    }

```

<span data-ttu-id="9bcb7-195">Como la clase **com.microsoft.pnp.TaxiCabReader** procesa los mensajes de carreras y tarifas, es posible que alguno sea incorrecto y, por tanto, no sea válido.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-195">As the **com.microsoft.pnp.TaxiCabReader** class processes ride and fare messages, it's possible that either one may be malformed and therefore not valid.</span></span> <span data-ttu-id="9bcb7-196">En un entorno de producción, es importante analizar estos mensajes incorrectamente formados para identificar un problema con los orígenes de datos, para poder corregirlos rápidamente y evitar la pérdida de datos.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-196">In a production environment, it's important to analyze these malformed messages to identify a problem with the data sources so it can be fixed quickly to prevent data loss.</span></span> <span data-ttu-id="9bcb7-197">La clase **com.microsoft.pnp.TaxiCabReader** registra un acumulador de Apache Spark que controla el número de registros de carreras y tarifas con formato incorrecto:</span><span class="sxs-lookup"><span data-stu-id="9bcb7-197">The **com.microsoft.pnp.TaxiCabReader** class registers an Apache Spark Accumulator that keeps track of the number of malformed fare and ride records:</span></span>

```scala
    @transient val appMetrics = new AppMetrics(spark.sparkContext)
    appMetrics.registerGauge("metrics.malformedrides", AppAccumulators.getRideInstance(spark.sparkContext))
    appMetrics.registerGauge("metrics.malformedfares", AppAccumulators.getFareInstance(spark.sparkContext))
    SparkEnv.get.metricsSystem.registerSource(appMetrics)
```

<span data-ttu-id="9bcb7-198">Apache Spark utiliza la biblioteca Dropwizard para enviar métricas, y algunos de los campos de métricas nativos de Dropwizard no son compatibles con Azure Log Analytics.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-198">Apache Spark uses the Dropwizard library to send metrics, and some of the native Dropwizard metrics fields are incompatible with Azure Log Analytics.</span></span> <span data-ttu-id="9bcb7-199">Por lo tanto, esta arquitectura de referencia incluye un receptor y un notificador de Dropwizard personalizados.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-199">Therefore, this reference architecture includes a custom Dropwizard sink and reporter.</span></span> <span data-ttu-id="9bcb7-200">Da formato a las métricas con el formato que Azure Log Analytics espera.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-200">It formats the metrics in the format expected by Azure Log Analytics.</span></span> <span data-ttu-id="9bcb7-201">Cuando Apache Spark informa de las métricas, también se envían las métricas personalizadas de los datos de carreras y tarifas con formato incorrecto</span><span class="sxs-lookup"><span data-stu-id="9bcb7-201">When Apache Spark reports metrics, the custom metrics for the malformed ride and fare data are also sent.</span></span>

<span data-ttu-id="9bcb7-202">La última métrica que se registra en el área de trabajo de Azure Log Analytics es el progreso acumulado del trabajo de flujo estructurado de Spark.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-202">The last metric to be logged to the Azure Log Analytics workspace is the cumulative progress of the Spark Structured Streaming job progress.</span></span> <span data-ttu-id="9bcb7-203">Esto se realiza mediante un agente de escucha StreamingQuery personalizado que se implementa en la clase **com.microsoft.pnp.StreamingMetricsListener**.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-203">This is done using a custom StreamingQuery listener implemented in the **com.microsoft.pnp.StreamingMetricsListener** class.</span></span> <span data-ttu-id="9bcb7-204">Esta clase se registra en la sesión de Apache Spark cuando se ejecuta el trabajo:</span><span class="sxs-lookup"><span data-stu-id="9bcb7-204">This class is registered to the Apache Spark Session when the job runs:</span></span>

```scala
spark.streams.addListener(new StreamingMetricsListener())
```

<span data-ttu-id="9bcb7-205">El entorno de ejecución de Apache Spark llama a los métodos en StreamingMetricsListener cada vez que se produce un evento de streaming estructurado, y envía los mensajes de registro y las métricas al área de trabajo de Azure Log Analytics.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-205">The methods in the StreamingMetricsListener are called by the Apache Spark runtime whenever a structured steaming event occurs, sending log messages and metrics to the Azure Log Analytics workspace.</span></span> <span data-ttu-id="9bcb7-206">Puede usar las siguientes consultas en el área de trabajo para supervisar la aplicación:</span><span class="sxs-lookup"><span data-stu-id="9bcb7-206">You can use the following queries in your workspace to monitor the application:</span></span>

### <a name="latency-and-throughput-for-streaming-queries"></a><span data-ttu-id="9bcb7-207">Latencia y rendimiento de las consultas de streaming</span><span class="sxs-lookup"><span data-stu-id="9bcb7-207">Latency and throughput for streaming queries</span></span> 

```shell
taxijob_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| project  mdc_inputRowsPerSecond_d, mdc_durationms_triggerExecution_d  
| render timechart
``` 
### <a name="exceptions-logged-during-stream-query-execution"></a><span data-ttu-id="9bcb7-208">Excepciones registradas durante la ejecución de consultas de flujos de datos</span><span class="sxs-lookup"><span data-stu-id="9bcb7-208">Exceptions logged during stream query execution</span></span>

```shell
taxijob_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| where Level contains "Error" 
```

### <a name="accumulation-of-malformed-fare-and-ride-data"></a><span data-ttu-id="9bcb7-209">Acumulación de datos de carrera y tarifas con formato incorrecto</span><span class="sxs-lookup"><span data-stu-id="9bcb7-209">Accumulation of malformed fare and ride data</span></span>

```shell
SparkMetric_CL 
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| render timechart 
| where name_s contains "metrics.malformedrides"

SparkMetric_CL 
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| render timechart 
| where name_s contains "metrics.malformedfares" 
```

### <a name="job-execution-to-trace-resiliency"></a><span data-ttu-id="9bcb7-210">Ejecución de trabajos para la resistencia del seguimiento</span><span class="sxs-lookup"><span data-stu-id="9bcb7-210">Job execution to trace resiliency</span></span>

```shell
SparkMetric_CL 
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| render timechart 
| where name_s contains "driver.DAGScheduler.job.allJobs" 
```

## <a name="deploy-the-solution"></a><span data-ttu-id="9bcb7-211">Implementación de la solución</span><span class="sxs-lookup"><span data-stu-id="9bcb7-211">Deploy the solution</span></span>

<span data-ttu-id="9bcb7-212">Hay disponible una implementación de esta arquitectura de referencia en [GitHub](https://github.com/mspnp/azure-databricks-streaming-analytics).</span><span class="sxs-lookup"><span data-stu-id="9bcb7-212">A deployment for this reference architecture is available on [GitHub](https://github.com/mspnp/azure-databricks-streaming-analytics).</span></span> 

### <a name="prerequisites"></a><span data-ttu-id="9bcb7-213">Requisitos previos</span><span class="sxs-lookup"><span data-stu-id="9bcb7-213">Prerequisites</span></span>

1. <span data-ttu-id="9bcb7-214">Clone, bifurque o descargue el [procesamiento de flujos de datos con el repositorio de GitrHub de Azure Databricks](https://github.com/mspnp/azure-databricks-streaming-analytics).</span><span class="sxs-lookup"><span data-stu-id="9bcb7-214">Clone, fork, or download the [stream processing with Azure Databricks](https://github.com/mspnp/azure-databricks-streaming-analytics) GitHub repository.</span></span>

2. <span data-ttu-id="9bcb7-215">Instale [Docker](https://www.docker.com/) para ejecutar el generador de datos.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-215">Install [Docker](https://www.docker.com/) to run the data generator.</span></span>

3. <span data-ttu-id="9bcb7-216">Instale la [CLI de Azure 2.0](/cli/azure/install-azure-cli?view=azure-cli-latest).</span><span class="sxs-lookup"><span data-stu-id="9bcb7-216">Install [Azure CLI 2.0](/cli/azure/install-azure-cli?view=azure-cli-latest).</span></span>

4. <span data-ttu-id="9bcb7-217">Instale la [CLI de Databricks](https://docs.databricks.com/user-guide/dev-tools/databricks-cli.html).</span><span class="sxs-lookup"><span data-stu-id="9bcb7-217">Install [Databricks CLI](https://docs.databricks.com/user-guide/dev-tools/databricks-cli.html).</span></span>

5. <span data-ttu-id="9bcb7-218">Desde un símbolo del sistema, un símbolo del sistema de Bash o un símbolo del sistema de PowerShell, inicie sesión en su cuenta de Azure como se indica a continuación:</span><span class="sxs-lookup"><span data-stu-id="9bcb7-218">From a command prompt, bash prompt, or PowerShell prompt, sign into your Azure account as follows:</span></span>
    ```shell
    az login
    ```
6. <span data-ttu-id="9bcb7-219">Instale un IDE de Java, con los siguientes recursos:</span><span class="sxs-lookup"><span data-stu-id="9bcb7-219">Install a Java IDE, with the following resources:</span></span>
    - <span data-ttu-id="9bcb7-220">JDK 1.8</span><span class="sxs-lookup"><span data-stu-id="9bcb7-220">JDK 1.8</span></span>
    - <span data-ttu-id="9bcb7-221">Scala SDK 2.11</span><span class="sxs-lookup"><span data-stu-id="9bcb7-221">Scala SDK 2.11</span></span>
    - <span data-ttu-id="9bcb7-222">Maven 3.5.4</span><span class="sxs-lookup"><span data-stu-id="9bcb7-222">Maven 3.5.4</span></span>

### <a name="download-the-new-york-city-taxi-and-neighborhood-data-files"></a><span data-ttu-id="9bcb7-223">Descargue los archivos de datos de taxis y barrios la ciudad de Nueva York</span><span class="sxs-lookup"><span data-stu-id="9bcb7-223">Download the New York City taxi and neighborhood data files</span></span>

1. <span data-ttu-id="9bcb7-224">Cree un directorio denominado `DataFile` en la raíz del repositorio de Github clonado en el sistema de archivos local.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-224">Create a directory named `DataFile` in the root of the cloned Github repository in your local file system.</span></span>

2. <span data-ttu-id="9bcb7-225">Abra un explorador web y vaya a https://uofi.app.box.com/v/NYCtaxidata/folder/2332219935.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-225">Open a web browser and navigate to https://uofi.app.box.com/v/NYCtaxidata/folder/2332219935.</span></span>

3. <span data-ttu-id="9bcb7-226">Haga clic en el botón **Descargar** en esta página para descargar un archivo zip con todos los datos de taxi de ese año.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-226">Click the **Download** button on this page to download a zip file of all the taxi data for that year.</span></span>

4. <span data-ttu-id="9bcb7-227">Extraiga el archivo zip en el directorio `DataFile` local.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-227">Extract the zip file to the `DataFile` directory.</span></span>

    > [!NOTE]
    > <span data-ttu-id="9bcb7-228">Este archivo zip contiene otros archivos zip.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-228">This zip file contains other zip files.</span></span> <span data-ttu-id="9bcb7-229">No extraiga los archivos zip secundarios.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-229">Don't extract the child zip files.</span></span>

    <span data-ttu-id="9bcb7-230">La estructura de directorios debe ser como la siguiente:</span><span class="sxs-lookup"><span data-stu-id="9bcb7-230">The directory structure must look like the following:</span></span>

    ```shell
    /DataFile
        /FOIL2013
            trip_data_1.zip
            trip_data_2.zip
            trip_data_3.zip
            ...
    ```

5. <span data-ttu-id="9bcb7-231">Abra un explorador web y vaya a https://www.zillow.com/howto/api/neighborhood-boundaries.htm.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-231">Open a web browser and navigate to https://www.zillow.com/howto/api/neighborhood-boundaries.htm.</span></span> 

6. <span data-ttu-id="9bcb7-232">Haga clic en **New York Neighborhood Boundaries** para descargar el archivo.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-232">Click on **New York Neighborhood Boundaries** to download the file.</span></span>

7. <span data-ttu-id="9bcb7-233">Copie el archivo **ZillowNeighborhoods NY.zip** desde el directorio de **descargas** de su explorador al directorio `DataFile`.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-233">Copy the **ZillowNeighborhoods-NY.zip** file from your browser's **downloads** directory to the `DataFile` directory.</span></span>

### <a name="deploy-the-azure-resources"></a><span data-ttu-id="9bcb7-234">Implementación de los recursos de Azure</span><span class="sxs-lookup"><span data-stu-id="9bcb7-234">Deploy the Azure resources</span></span>

1. <span data-ttu-id="9bcb7-235">Desde un shell o símbolo del sistema de Windows, ejecute el siguiente comando y siga las indicaciones de inicio de sesión:</span><span class="sxs-lookup"><span data-stu-id="9bcb7-235">From a shell or Windows Command Prompt, run the following command and follow the sign-in prompt:</span></span>

    ```bash
    az login
    ```

2. <span data-ttu-id="9bcb7-236">Vaya a la carpeta denominada `azure` del repositorio de GitHub:</span><span class="sxs-lookup"><span data-stu-id="9bcb7-236">Navigate to the folder named `azure` in the GitHub repository:</span></span>

    ```bash
    cd azure
    ```

3. <span data-ttu-id="9bcb7-237">Ejecute los siguientes comandos para implementar los recursos de Azure:</span><span class="sxs-lookup"><span data-stu-id="9bcb7-237">Run the following commands to deploy the Azure resources:</span></span>

    ```bash
    export resourceGroup='[Resource group name]'
    export resourceLocation='[Region]'
    export eventHubNamespace='[Event Hubs namespace name]'
    export databricksWorkspaceName='[Azure Databricks workspace name]'
    export cosmosDatabaseAccount='[Cosmos DB database name]'
    export logAnalyticsWorkspaceName='[Log Analytics workspace name]'
    export logAnalyticsWorkspaceRegion='[Log Analytics region]'

    # Create a resource group
    az group create --name $resourceGroup --location $resourceLocation

    # Deploy resources
    az group deployment create --resource-group $resourceGroup \
        --template-file deployresources.json --parameters \
        eventHubNamespace=$eventHubNamespace \
        databricksWorkspaceName=$databricksWorkspaceName \
        cosmosDatabaseAccount=$cosmosDatabaseAccount \
        logAnalyticsWorkspaceName=$logAnalyticsWorkspaceName \
        logAnalyticsWorkspaceRegion=$logAnalyticsWorkspaceRegion
    ```

4. <span data-ttu-id="9bcb7-238">La salida de la implementación se escribe en la consola una vez completada.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-238">The output of the deployment is written to the console once complete.</span></span> <span data-ttu-id="9bcb7-239">Busque el siguiente código JSON en la salida:</span><span class="sxs-lookup"><span data-stu-id="9bcb7-239">Search the output for the following JSON:</span></span>

```JSON
"outputs": {
        "cosmosDb": {
          "type": "Object",
          "value": {
            "hostName": <value>,
            "secret": <value>,
            "username": <value>
          }
        },
        "eventHubs": {
          "type": "Object",
          "value": {
            "taxi-fare-eh": <value>,
            "taxi-ride-eh": <value>
          }
        },
        "logAnalytics": {
          "type": "Object",
          "value": {
            "secret": <value>,
            "workspaceId": <value>
          }
        }
},
```
<span data-ttu-id="9bcb7-240">Estos valores son los secretos que se agregarán a los secretos de Databricks en las próximas secciones.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-240">These values are the secrets that will be added to Databricks secrets in upcoming sections.</span></span> <span data-ttu-id="9bcb7-241">Guárdelos de forma segura hasta que los agregue a esas secciones.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-241">Keep them secure until you add them in those sections.</span></span>

### <a name="add-a-cassandra-table-to-the-cosmos-db-account"></a><span data-ttu-id="9bcb7-242">Adición de una tabla de Cassandra a la cuenta de Cosmos DB</span><span class="sxs-lookup"><span data-stu-id="9bcb7-242">Add a Cassandra table to the Cosmos DB Account</span></span>

1. <span data-ttu-id="9bcb7-243">En Azure Portal, vaya al grupo de recursos creado en la sección **Implementación de los recursos de Azure**.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-243">In the Azure portal, navigate to the resource group created in the **deploy the Azure resources** section above.</span></span> <span data-ttu-id="9bcb7-244">Haga clic en **Cuenta de Azure Cosmos DB**.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-244">Click on **Azure Cosmos DB Account**.</span></span> <span data-ttu-id="9bcb7-245">Cree una tabla con Cassandra API.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-245">Create a table with the Cassandra API.</span></span>

2. <span data-ttu-id="9bcb7-246">En la hoja **Información general**, haga clic en **agregar tabla**.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-246">In the **overview** blade, click **add table**.</span></span>

3. <span data-ttu-id="9bcb7-247">Cuando se abra la hoja **Agregar tabla**, escriba `newyorktaxi` en el cuadro de texto **Nombre de Keyspace**.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-247">When the **add table** blade opens, enter `newyorktaxi` in the **Keyspace name** text box.</span></span> 

4. <span data-ttu-id="9bcb7-248">En la sección **escriba los comandos CQL para crear la tabla**, escriba `neighborhoodstats` en el cuadro de texto junto a `newyorktaxi`.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-248">In the **enter CQL command to create the table** section, enter `neighborhoodstats` in the text box beside `newyorktaxi`.</span></span>

5. <span data-ttu-id="9bcb7-249">En el cuadro de texto que aparece a continuación, escriba lo siguiente:</span><span class="sxs-lookup"><span data-stu-id="9bcb7-249">In the text box below, enter the following::</span></span>
```shell
(neighborhood text, window_end timestamp, number_of_rides bigint,total_fare_amount double, primary key(neighborhood, window_end))
```
6. <span data-ttu-id="9bcb7-250">En el cuadro de texto **Rendimiento (1 000 - 1 000 000 RU/s)**, escriba el valor `4000`.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-250">In the **Throughput (1,000 - 1,000,000 RU/s)** text box enter the value `4000`.</span></span>

7. <span data-ttu-id="9bcb7-251">Haga clic en **OK**.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-251">Click **OK**.</span></span>

### <a name="add-the-databricks-secrets-using-the-databricks-cli"></a><span data-ttu-id="9bcb7-252">Agregue los secretos de Databricks con la CLI de Databricks</span><span class="sxs-lookup"><span data-stu-id="9bcb7-252">Add the Databricks secrets using the Databricks CLI</span></span>

<span data-ttu-id="9bcb7-253">En primer lugar, especifique los secretos para el centro de eventos:</span><span class="sxs-lookup"><span data-stu-id="9bcb7-253">First, enter the secrets for EventHub:</span></span>

1. <span data-ttu-id="9bcb7-254">Use la **CLI de Azure Databricks** instalada en el paso 2 de los requisitos previos para crear el ámbito de los secretos de Azure Databricks:</span><span class="sxs-lookup"><span data-stu-id="9bcb7-254">Using the **Azure Databricks CLI** installed in step 2 of the prerequisites, create the Azure Databricks secret scope:</span></span>
    ```shell
    databricks secrets create-scope --scope "azure-databricks-job"
    ```
2. <span data-ttu-id="9bcb7-255">Agregue el secreto para el centro de eventos de carreras de taxi:</span><span class="sxs-lookup"><span data-stu-id="9bcb7-255">Add the secret for the taxi ride EventHub:</span></span>
    ```shell
    databricks secrets put --scope "azure-databricks-job" --key "taxi-ride"
    ```
    <span data-ttu-id="9bcb7-256">Una vez ejecutado, este comando abre el editor vi.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-256">Once executed, this command opens the vi editor.</span></span> <span data-ttu-id="9bcb7-257">Escriba el valor de **taxi-ride-eh** de la sección de salida **eventHubs** del paso 4 en la sección *implementar los recursos de Azure*.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-257">Enter the **taxi-ride-eh** value from the **eventHubs** output section in step 4 of the *deploy the Azure resources* section.</span></span> <span data-ttu-id="9bcb7-258">Guarde y salga de vi.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-258">Save and exit vi.</span></span>

3. <span data-ttu-id="9bcb7-259">Agregue el secreto para el centro de eventos de tarifas de taxi:</span><span class="sxs-lookup"><span data-stu-id="9bcb7-259">Add the secret for the taxi fare EventHub:</span></span>
    ```shell
    databricks secrets put --scope "azure-databricks-job" --key "taxi-fare"
    ```
    <span data-ttu-id="9bcb7-260">Una vez ejecutado, este comando abre el editor vi.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-260">Once executed, this command opens the vi editor.</span></span> <span data-ttu-id="9bcb7-261">Escriba el valor de **taxi-fare-eh** de la sección de salida **eventHubs** del paso 4 en la sección *implementación de los recursos de Azure*.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-261">Enter the **taxi-fare-eh** value from the **eventHubs** output section in step 4 of the *deploy the Azure resources* section.</span></span> <span data-ttu-id="9bcb7-262">Guarde y salga de vi.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-262">Save and exit vi.</span></span>

<span data-ttu-id="9bcb7-263">A continuación, especifique los secretos para Cosmos DB:</span><span class="sxs-lookup"><span data-stu-id="9bcb7-263">Next, enter the secrets for Cosmos DB:</span></span>

1. <span data-ttu-id="9bcb7-264">Abra Azure Portal y vaya al grupo de recursos especificado en el paso 3 de la sección **implementación de los recursos de Azure**.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-264">Open the Azure portal, and navigate to the resource group specified in step 3 of the **deploy the Azure resources** section.</span></span> <span data-ttu-id="9bcb7-265">Haga clic en la cuenta de Azure Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-265">Click on the Azure Cosmos DB Account.</span></span>

2. <span data-ttu-id="9bcb7-266">Use la **CLI de Azure Databricks** para agregar el secreto para el nombre de usuario de Cosmos DB:</span><span class="sxs-lookup"><span data-stu-id="9bcb7-266">Using the **Azure Databricks CLI**, add the secret for the Cosmos DB user name:</span></span>
    ```shell
    databricks secrets put --scope azure-databricks-job --key "cassandra-username"
    ```
<span data-ttu-id="9bcb7-267">Una vez ejecutado, este comando abre el editor vi.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-267">Once executed, this command opens the vi editor.</span></span> <span data-ttu-id="9bcb7-268">Escriba el valor de **username** de la sección de salida **CosmosDb** del paso 4 en la sección *implementación de los recursos de Azure*.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-268">Enter the **username** value from the **CosmosDb** output section in step 4 of the *deploy the Azure resources* section.</span></span> <span data-ttu-id="9bcb7-269">Guarde y salga de vi.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-269">Save and exit vi.</span></span>

3. <span data-ttu-id="9bcb7-270">A continuación, agregue el secreto para la contraseña de Cosmos DB:</span><span class="sxs-lookup"><span data-stu-id="9bcb7-270">Next, add the secret for the Cosmos DB password:</span></span>
    ```shell
    databricks secrets put --scope azure-databricks-job --key "cassandra-password"
    ```

<span data-ttu-id="9bcb7-271">Una vez ejecutado, este comando abre el editor vi.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-271">Once executed, this command opens the vi editor.</span></span> <span data-ttu-id="9bcb7-272">Escriba el valor de **secret** de la sección de salida **CosmosDb** del paso 4 en la sección *implementación de los recursos de Azure*.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-272">Enter the **secret** value from the **CosmosDb** output section in step 4 of the *deploy the Azure resources* section.</span></span> <span data-ttu-id="9bcb7-273">Guarde y salga de vi.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-273">Save and exit vi.</span></span>

> [!NOTE]
> <span data-ttu-id="9bcb7-274">Si usa un [ámbito de secretos respaldado por Azure Key Vault](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html#azure-key-vault-backed-scopes), el ámbito debe llamarse **azure-databricks-job** y los secretos deben tener exactamente los mismos nombres que los mencionados.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-274">If using an [Azure Key Vault-backed secret scope](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html#azure-key-vault-backed-scopes), the scope must be named **azure-databricks-job** and the secrets must have the exact same names as those above.</span></span>

### <a name="add-the-zillow-neighborhoods-data-file-to-the-databricks-file-system"></a><span data-ttu-id="9bcb7-275">Agregue el archivo de datos Zillow Neighborhoods al sistema de archivos de Databricks</span><span class="sxs-lookup"><span data-stu-id="9bcb7-275">Add the Zillow Neighborhoods data file to the Databricks file system</span></span>

1. <span data-ttu-id="9bcb7-276">Cree un directorio en el sistema de archivos de Databricks:</span><span class="sxs-lookup"><span data-stu-id="9bcb7-276">Create a directory in the Databricks file system:</span></span>
    ```bash
    dbfs mkdirs dbfs:/azure-databricks-jobs
    ```

2. <span data-ttu-id="9bcb7-277">Vaya al directorio `DataFile` y escriba lo siguiente:</span><span class="sxs-lookup"><span data-stu-id="9bcb7-277">Navigate to the `DataFile` directory and enter the following:</span></span>
    ```bash
    dbfs cp ZillowNeighborhoods-NY.zip dbfs:/azure-databricks-jobs
    ```

### <a name="add-the-azure-log-analytics-workspace-id-and-primary-key-to-configuration-files"></a><span data-ttu-id="9bcb7-278">Adición del identificador de área de trabajo y la clave principal de Azure Log Analytics a los archivos de configuración</span><span class="sxs-lookup"><span data-stu-id="9bcb7-278">Add the Azure Log Analytics workspace ID and primary key to configuration files</span></span>

<span data-ttu-id="9bcb7-279">Para esta sección, necesita el identificador del área de trabajo y la clave principal de Log Analytics.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-279">For this section, you require the Log Analytics workspace ID and primary key.</span></span> <span data-ttu-id="9bcb7-280">El identificador de área de trabajo es el valor de **workspaceId** de la sección de salida **logAnalytics**, en el paso 4 de la sección *implementación de los recursos de Azure*.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-280">The workspace ID is the **workspaceId** value from the **logAnalytics** output section in step 4 of the *deploy the Azure resources* section.</span></span> <span data-ttu-id="9bcb7-281">La clave principal es el valor de **secret** de la sección de salida.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-281">The primary key is the **secret** from the output section.</span></span> 

1. <span data-ttu-id="9bcb7-282">Para configurar el registro de log4j, abra `\azure\AzureDataBricksJob\src\main\resources\com\microsoft\pnp\azuredatabricksjob\log4j.properties`.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-282">To configure log4j logging, open `\azure\AzureDataBricksJob\src\main\resources\com\microsoft\pnp\azuredatabricksjob\log4j.properties`.</span></span> <span data-ttu-id="9bcb7-283">Edite los dos valores siguientes:</span><span class="sxs-lookup"><span data-stu-id="9bcb7-283">Edit the following two values:</span></span>
    ```shell
    log4j.appender.A1.workspaceId=<Log Analytics workspace ID>
    log4j.appender.A1.secret=<Log Analytics primary key>
    ```

2. <span data-ttu-id="9bcb7-284">Para configurar el registro personalizado, abra `\azure\azure-databricks-monitoring\scripts\metrics.properties`.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-284">To configure custom logging, open `\azure\azure-databricks-monitoring\scripts\metrics.properties`.</span></span> <span data-ttu-id="9bcb7-285">Edite los dos valores siguientes:</span><span class="sxs-lookup"><span data-stu-id="9bcb7-285">Edit the following two values:</span></span>
    ```shell
    *.sink.loganalytics.workspaceId=<Log Analytics workspace ID>
    *.sink.loganalytics.secret=<Log Analytics primary key>
    ```

### <a name="build-the-jar-files-for-the-databricks-job-and-databricks-monitoring"></a><span data-ttu-id="9bcb7-286">Compilación de los archivos .jar para la supervisión de Databricks y los trabajos de Databricks</span><span class="sxs-lookup"><span data-stu-id="9bcb7-286">Build the .jar files for the Databricks job and Databricks monitoring</span></span>

1. <span data-ttu-id="9bcb7-287">Use su entorno de desarrollo de Java para importar el archivo de proyecto de Maven llamado **pom.xml**, que se encuentra en el directorio raíz.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-287">Use your Java IDE to import the Maven project file named **pom.xml** located in the root directory.</span></span> 

2. <span data-ttu-id="9bcb7-288">Realice una compilación limpia.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-288">Perform a clean build.</span></span> <span data-ttu-id="9bcb7-289">La salida de esta compilación es dos archivos llamados **azure-databricks-trabajo-1.0-SNAPSHOT.jar** y **azure-databricks-monitoring-0.9.jar**.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-289">The output of this build is files named **azure-databricks-job-1.0-SNAPSHOT.jar** and **azure-databricks-monitoring-0.9.jar**.</span></span> 

### <a name="configure-custom-logging-for-the-databricks-job"></a><span data-ttu-id="9bcb7-290">Configuración del registro personalizado para el trabajo de Databricks</span><span class="sxs-lookup"><span data-stu-id="9bcb7-290">Configure custom logging for the Databricks job</span></span>

1. <span data-ttu-id="9bcb7-291">Copie el archivo **azure-databricks-monitoring-0.9.jar** al sistema de archivos de Databricks; para ello, escriba el comando siguiente en la **CLI de Databricks**:</span><span class="sxs-lookup"><span data-stu-id="9bcb7-291">Copy the **azure-databricks-monitoring-0.9.jar** file to the Databricks file system by entering the following command in the **Databricks CLI**:</span></span>
    ```shell
    databricks fs cp --overwrite azure-databricks-monitoring-0.9.jar dbfs:/azure-databricks-job/azure-databricks-monitoring-0.9.jar
    ```

2. <span data-ttu-id="9bcb7-292">Copie las propiedades del registro personalizado desde `\azure\azure-databricks-monitoring\scripts\metrics.properties` al sistema de archivos de Databricks, para lo que debe escribir el siguiente comando:</span><span class="sxs-lookup"><span data-stu-id="9bcb7-292">Copy the custom logging properties from `\azure\azure-databricks-monitoring\scripts\metrics.properties` to the Databricks file system by entering the following command:</span></span>
    ```shell
    databricks fs cp --overwrite metrics.properties dbfs:/azure-databricks-job/metrics.properties
    ```

3. <span data-ttu-id="9bcb7-293">Aunque aún no haya decidido un nombre para el clúster de Databricks, seleccione uno ahora.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-293">While you haven't yet decided on a name for your Databricks cluster, select one now.</span></span> <span data-ttu-id="9bcb7-294">Deberá escribir el nombre debajo, en la ruta de acceso del sistema de archivos de Databricks para el clúster.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-294">You'll enter the name below in the Databricks file system path for your cluster.</span></span> <span data-ttu-id="9bcb7-295">Copie el script de inicialización desde `\azure\azure-databricks-monitoring\scripts\spark.metrics` al sistema de archivos de Databricks, para lo que debe escribir el siguiente comando:</span><span class="sxs-lookup"><span data-stu-id="9bcb7-295">Copy the initialization script from `\azure\azure-databricks-monitoring\scripts\spark.metrics` to the Databricks file system by entering the following command:</span></span>
    ```
    databricks fs cp --overwrite spark-metrics.sh dbfs:/databricks/init/<cluster-name>/spark-metrics.sh
    ```

### <a name="create-a-databricks-cluster"></a><span data-ttu-id="9bcb7-296">Creación de un clúster de Databricks</span><span class="sxs-lookup"><span data-stu-id="9bcb7-296">Create a Databricks cluster</span></span>

1. <span data-ttu-id="9bcb7-297">En el área de trabajo de Databricks, haga clic en "Clústeres" y en "Crear clúster".</span><span class="sxs-lookup"><span data-stu-id="9bcb7-297">In the Databricks workspace, click "Clusters", then click "create cluster".</span></span> <span data-ttu-id="9bcb7-298">Escriba el nombre de clúster que creó en el paso 3 de la sección **configuración del registro personalizado para el trabajo de Databricks**.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-298">Enter the cluster name you created in step 3 of the **configure custom logging for the Databricks job** section above.</span></span> 

2. <span data-ttu-id="9bcb7-299">Seleccione un modo de clúster **standard**.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-299">Select a **standard** cluster mode.</span></span>

3. <span data-ttu-id="9bcb7-300">Establezca **Databricks runtime version** (Versión del entorno de ejecución de Databricks) en **4.3 (includes Apache Spark 2.3.1, Scala 2.11)** [4.3 (incluye Apache Spark 2.3.1, Scala 2.11)]</span><span class="sxs-lookup"><span data-stu-id="9bcb7-300">Set **Databricks runtime version** to **4.3 (includes Apache Spark 2.3.1, Scala 2.11)**</span></span>

4. <span data-ttu-id="9bcb7-301">Establezca **Python version** (Versión de Python) en **2**.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-301">Set **Python version** to **2**.</span></span>

5. <span data-ttu-id="9bcb7-302">Establezca **Driver Type** (Tipo de controlador) en **Same as worker** (Igual que el trabajo).</span><span class="sxs-lookup"><span data-stu-id="9bcb7-302">Set **Driver Type** to **Same as worker**</span></span>

6. <span data-ttu-id="9bcb7-303">Establezca **Worker Type** (Tipo de trabajo) en **Standard_DS3_v2**.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-303">Set **Worker Type** to **Standard_DS3_v2**.</span></span>

7. <span data-ttu-id="9bcb7-304">Establezca **Min Workers** (Trabajos mínimos) en **2**.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-304">Set **Min Workers** to **2**.</span></span>

8. <span data-ttu-id="9bcb7-305">Anule la selección de **Enable autoscaling** (Habilitar escalado automático).</span><span class="sxs-lookup"><span data-stu-id="9bcb7-305">Deselect **Enable autoscaling**.</span></span> 

9. <span data-ttu-id="9bcb7-306">En el cuadro de diálogo **Auto Termination** (Finalización automática), haga clic en **Scripts Init**.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-306">Below the **Auto Termination** dialog box, click on **Init Scripts**.</span></span> 

10. <span data-ttu-id="9bcb7-307">Escriba **dbfs:/databricks/init/<cluster-name>/spark-metrics.sh** y sustituya el nombre del clúster creado en el paso 1 para <cluster-name>.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-307">Enter **dbfs:/databricks/init/<cluster-name>/spark-metrics.sh**, substituting the cluster name created in step 1 for <cluster-name>.</span></span>

11. <span data-ttu-id="9bcb7-308">Haga clic en el botón **Agregar**.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-308">Click the **Add** button.</span></span>

12. <span data-ttu-id="9bcb7-309">Haga clic en el botón **Create Cluster** (Crear clúster).</span><span class="sxs-lookup"><span data-stu-id="9bcb7-309">Click the **Create Cluster** button.</span></span>

### <a name="create-a-databricks-job"></a><span data-ttu-id="9bcb7-310">Creación de un trabajo de Databricks</span><span class="sxs-lookup"><span data-stu-id="9bcb7-310">Create a Databricks job</span></span>

1. <span data-ttu-id="9bcb7-311">En el área de trabajo de Databricks, haga clic en "Trabajos", "crear trabajo".</span><span class="sxs-lookup"><span data-stu-id="9bcb7-311">In the Databricks workspace, click "Jobs", "create job".</span></span>

2. <span data-ttu-id="9bcb7-312">Escriba el nombre del trabajo.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-312">Enter a job name.</span></span>

3. <span data-ttu-id="9bcb7-313">Haga clic en "set jar" y se abrirá el cuadro de diálogo "Cargar JAR para ejecución".</span><span class="sxs-lookup"><span data-stu-id="9bcb7-313">Click "set jar", this opens the "Upload JAR to Run" dialog box.</span></span>

4. <span data-ttu-id="9bcb7-314">Arrastre el archivo **azure-databricks-job-1.0-SNAPSHOT.jar** que creó en la sección **compilación del archivo .jar para el trabajo de Databricks** hasta el cuadro **Soltar JAR aquí para cargar**.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-314">Drag the **azure-databricks-job-1.0-SNAPSHOT.jar** file you created in the **build the .jar for the Databricks job** section to the **Drop JAR here to upload** box.</span></span>

5. <span data-ttu-id="9bcb7-315">Escriba **com.microsoft.pnp.TaxiCabReader** en el campo **Clase Main**.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-315">Enter **com.microsoft.pnp.TaxiCabReader** in the **Main Class** field.</span></span>

6. <span data-ttu-id="9bcb7-316">En el campo Argumentos, escriba lo siguiente:</span><span class="sxs-lookup"><span data-stu-id="9bcb7-316">In the arguments field, enter the following:</span></span>
    ```shell
    -n jar:file:/dbfs/azure-databricks-jobs/ZillowNeighborhoods-NY.zip!/ZillowNeighborhoods-NY.shp --taxi-ride-consumer-group taxi-ride-eh-cg --taxi-fare-consumer-group taxi-fare-eh-cg --window-interval "1 minute" --cassandra-host <Cosmos DB Cassandra host name from above> 
    ``` 

7. <span data-ttu-id="9bcb7-317">Instale las bibliotecas dependientes siguiendo estos pasos:</span><span class="sxs-lookup"><span data-stu-id="9bcb7-317">Install the dependent libraries by following these steps:</span></span>
    
    1. <span data-ttu-id="9bcb7-318">En la interfaz de usuario de Databricks, haga clic en el botón **inicio**.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-318">In the Databricks user interface, click on the **home** button.</span></span>
    
    2. <span data-ttu-id="9bcb7-319">En la lista desplegable **Users** (Usuarios), haga clic en el nombre de la cuenta de usuario para abrir la configuración del área de trabajo de su cuenta.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-319">In the **Users** drop-down, click on your user account name to open your account workspace settings.</span></span>
    
    3. <span data-ttu-id="9bcb7-320">Haga clic en la flecha desplegable situada junto al nombre de la cuenta, haga clic en **create** (crear) y haga clic en **Library** (Biblioteca) para abrir el cuadro de diálogo **New Library** (Nueva biblioteca).</span><span class="sxs-lookup"><span data-stu-id="9bcb7-320">Click on the drop-down arrow beside your account name, click on **create**, and click on **Library** to open the **New Library** dialog.</span></span>
    
    4. <span data-ttu-id="9bcb7-321">En la lista desplegable **Source** (Origen), seleccione **Maven Coordinate** (Coordenada de Maven).</span><span class="sxs-lookup"><span data-stu-id="9bcb7-321">In the **Source** drop-down control, select **Maven Coordinate**.</span></span>
    
    5. <span data-ttu-id="9bcb7-322">Bajo el encabezado **Install Maven Artifacts** (Instalar artefactos de Maven), escriba `com.microsoft.azure:azure-eventhubs-spark_2.11:2.3.5` en el cuadro de texto **Coordinate** (Coordenada).</span><span class="sxs-lookup"><span data-stu-id="9bcb7-322">Under the **Install Maven Artifacts** heading, enter `com.microsoft.azure:azure-eventhubs-spark_2.11:2.3.5` in the **Coordinate** text box.</span></span> 
    
    6. <span data-ttu-id="9bcb7-323">Haga clic en **Create Library** (Crear biblioteca) para abrir la ventana **Artifacts** (Artefactos).</span><span class="sxs-lookup"><span data-stu-id="9bcb7-323">Click on **Create Library** to open the **Artifacts** window.</span></span>
    
    7. <span data-ttu-id="9bcb7-324">En **Status on running clusters** (Estados en clústeres en ejecución), active la casilla **Attach automatically to all clusters** (Adjuntar automáticamente a todos los clústeres).</span><span class="sxs-lookup"><span data-stu-id="9bcb7-324">Under **Status on running clusters** check the **Attach automatically to all clusters** checkbox.</span></span>
    
    8. <span data-ttu-id="9bcb7-325">Repita los pasos del 1 al 7 para la coordenada de Maven `com.microsoft.azure.cosmosdb:azure-cosmos-cassandra-spark-helper:1.0.0`.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-325">Repeat steps 1 - 7 for the `com.microsoft.azure.cosmosdb:azure-cosmos-cassandra-spark-helper:1.0.0` Maven coordinate.</span></span>
    
    9. <span data-ttu-id="9bcb7-326">Repita los pasos del 1 al 6 para la coordenada de Maven `org.geotools:gt-shapefile:19.2`.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-326">Repeat steps 1 - 6 for the `org.geotools:gt-shapefile:19.2` Maven coordinate.</span></span>
    
    10. <span data-ttu-id="9bcb7-327">Haga clic en **Advanced Options** (Opciones avanzadas).</span><span class="sxs-lookup"><span data-stu-id="9bcb7-327">Click on **Advanced Options**.</span></span>
    
    11. <span data-ttu-id="9bcb7-328">Escriba `http://download.osgeo.org/webdav/geotools/` en el cuadro de texto **Repository** (Repositorio).</span><span class="sxs-lookup"><span data-stu-id="9bcb7-328">Enter `http://download.osgeo.org/webdav/geotools/` in the **Repository** text box.</span></span> 
    
    12. <span data-ttu-id="9bcb7-329">Haga clic en **Create Library** (Crear biblioteca) para abrir la ventana **Artifacts** (Artefactos).</span><span class="sxs-lookup"><span data-stu-id="9bcb7-329">Click **Create Library** to open the **Artifacts** window.</span></span> 
    
    13. <span data-ttu-id="9bcb7-330">En **Status on running clusters** (Estados en clústeres en ejecución), active la casilla **Attach automatically to all clusters** (Adjuntar automáticamente a todos los clústeres).</span><span class="sxs-lookup"><span data-stu-id="9bcb7-330">Under **Status on running clusters** check the **Attach automatically to all clusters** checkbox.</span></span>

8. <span data-ttu-id="9bcb7-331">Agregue las bibliotecas dependientes que agregó en el paso 7 al trabajo creado al final del paso 6:</span><span class="sxs-lookup"><span data-stu-id="9bcb7-331">Add the dependent libraries added in step 7 to the job created at the end of step 6:</span></span>
    1. <span data-ttu-id="9bcb7-332">En el área de trabajo de Azure Databricks, haga clic en **Jobs** (Trabajos).</span><span class="sxs-lookup"><span data-stu-id="9bcb7-332">In the Azure Databricks workspace, click on **Jobs**.</span></span>

    2. <span data-ttu-id="9bcb7-333">Haga clic en el nombre del trabajo creado en el paso 2 de la sección **creación de un trabajo de Databricks**.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-333">Click on the job name created in step 2 of the **create a Databricks job** section.</span></span> 
    
    3. <span data-ttu-id="9bcb7-334">Junto a la sección **Dependent Libraries** (Bibliotecas dependientes), haga clic en **Add** (Agregar) para abrir el cuadro de diálogo **Add Dependent Library** (Agregar biblioteca dependiente).</span><span class="sxs-lookup"><span data-stu-id="9bcb7-334">Beside the **Dependent Libraries** section, click on **Add** to open the **Add Dependent Library** dialog.</span></span> 
    
    4. <span data-ttu-id="9bcb7-335">En **Library From** (Desde biblioteca), seleccione **Workspace** (Área de trabajo).</span><span class="sxs-lookup"><span data-stu-id="9bcb7-335">Under **Library From** select **Workspace**.</span></span>
    
    5. <span data-ttu-id="9bcb7-336">Haga clic en **Users** (Usuarios), en su nombre de usuario y, a continuación, en `azure-eventhubs-spark_2.11:2.3.5`.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-336">Click on **users**, then your username, then click on `azure-eventhubs-spark_2.11:2.3.5`.</span></span> 
    
    6. <span data-ttu-id="9bcb7-337">Haga clic en **OK**.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-337">Click **OK**.</span></span>
    
    7. <span data-ttu-id="9bcb7-338">Repita los pasos del 1 al 6 para `spark-cassandra-connector_2.11:2.3.1` y `gt-shapefile:19.2`.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-338">Repeat steps 1 - 6 for `spark-cassandra-connector_2.11:2.3.1` and `gt-shapefile:19.2`.</span></span>

9. <span data-ttu-id="9bcb7-339">Junto a **Cluster:**, haga clic en **Edit** (Editar).</span><span class="sxs-lookup"><span data-stu-id="9bcb7-339">Beside **Cluster:**, click on **Edit**.</span></span> <span data-ttu-id="9bcb7-340">Se abrirá el cuadro de diálogo **Configure Cluster** (Configurar clúster).</span><span class="sxs-lookup"><span data-stu-id="9bcb7-340">This opens the **Configure Cluster** dialog.</span></span> <span data-ttu-id="9bcb7-341">En la lista desplegable **Cluster Type** (Tipo de clúster), seleccione **Existing Cluster** (Clúster existente).</span><span class="sxs-lookup"><span data-stu-id="9bcb7-341">In the **Cluster Type** drop-down, select **Existing Cluster**.</span></span> <span data-ttu-id="9bcb7-342">En la lista desplegable **Select Cluster** (Seleccionar clúster), seleccione el clúster creado en la sección **creación de un clúster de Databricks**.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-342">In the **Select Cluster** drop-down, select the cluster created the **create a Databricks cluster** section.</span></span> <span data-ttu-id="9bcb7-343">Haga clic en **Confirm** (Confirmar).</span><span class="sxs-lookup"><span data-stu-id="9bcb7-343">Click **confirm**.</span></span>

10. <span data-ttu-id="9bcb7-344">Haga clic en **Run now** (Ejecutar ahora).</span><span class="sxs-lookup"><span data-stu-id="9bcb7-344">Click **run now**.</span></span>

### <a name="run-the-data-generator"></a><span data-ttu-id="9bcb7-345">Ejecución del generador de datos</span><span class="sxs-lookup"><span data-stu-id="9bcb7-345">Run the data generator</span></span>

1. <span data-ttu-id="9bcb7-346">Vaya al directorio denominado `onprem` del repositorio de GitHub.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-346">Navigate to the directory named `onprem` in the GitHub repository.</span></span>

2. <span data-ttu-id="9bcb7-347">Actualice los valores en el archivo **main.env** de la siguiente manera:</span><span class="sxs-lookup"><span data-stu-id="9bcb7-347">Update the values in the file **main.env** as follows:</span></span>

    ```shell
    RIDE_EVENT_HUB=[Connection string for the taxi-ride event hub]
    FARE_EVENT_HUB=[Connection string for the taxi-fare event hub]
    RIDE_DATA_FILE_PATH=/DataFile/FOIL2013
    MINUTES_TO_LEAD=0
    PUSH_RIDE_DATA_FIRST=false
    ```
    <span data-ttu-id="9bcb7-348">La cadena de conexión para el centro de eventos taxi-ride es el valor de **taxi-ride-eh** de la sección de salida **eventHubs** del paso 4 en la sección *implementar los recursos de Azure*.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-348">The connection string for the taxi-ride event hub is the **taxi-ride-eh** value from the **eventHubs** output section in step 4 of the *deploy the Azure resources* section.</span></span> <span data-ttu-id="9bcb7-349">La cadena de conexión para el centro de eventos taxi-fare es el valor de **taxi-fare-eh** de la sección de salida **eventHubs** del paso 4 en la sección *implementar los recursos de Azure*.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-349">The connection string for the taxi-fare event hub the **taxi-fare-eh** value from the **eventHubs** output section in step 4 of the *deploy the Azure resources* section.</span></span>

3. <span data-ttu-id="9bcb7-350">Ejecute el siguiente comando para compilar la imagen de Docker.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-350">Run the following command to build the Docker image.</span></span>

    ```bash
    docker build --no-cache -t dataloader .
    ```

4. <span data-ttu-id="9bcb7-351">Vuelva al directorio principal.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-351">Navigate back to the parent directory.</span></span>

    ```bash
    cd ..
    ```

5. <span data-ttu-id="9bcb7-352">Ejecute el siguiente comando para ejecutar la imagen de Docker.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-352">Run the following command to run the Docker image.</span></span>

    ```bash
    docker run -v `pwd`/DataFile:/DataFile --env-file=onprem/main.env dataloader:latest
    ```

<span data-ttu-id="9bcb7-353">La salida debe tener un aspecto similar al siguiente:</span><span class="sxs-lookup"><span data-stu-id="9bcb7-353">The output should look like the following:</span></span>

```
Created 10000 records for TaxiFare
Created 10000 records for TaxiRide
Created 20000 records for TaxiFare
Created 20000 records for TaxiRide
Created 30000 records for TaxiFare
...
```

<span data-ttu-id="9bcb7-354">Para comprobar que el trabajo de Databricks se ejecuta correctamente, abra Azure Portal y vaya a la base de datos de Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-354">To verify the Databricks job is running correctly, open the Azure portal and navigate to the Cosmos DB database.</span></span> <span data-ttu-id="9bcb7-355">Abra la hoja **Explorador de datos** y examine los datos en la tabla **taxi records**.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-355">Open the **Data Explorer** blade and examine the data in the **taxi records** table.</span></span> 

<span data-ttu-id="9bcb7-356">[1] <span id="note1">Donovan, Brian; Work, Dan (2016): New York City Taxi Trip Data (2010-2013).</span><span class="sxs-lookup"><span data-stu-id="9bcb7-356">[1] <span id="note1">Donovan, Brian; Work, Dan (2016): New York City Taxi Trip Data (2010-2013).</span></span> <span data-ttu-id="9bcb7-357">Universidad de Illinois en Urbana-Champaign.</span><span class="sxs-lookup"><span data-stu-id="9bcb7-357">University of Illinois at Urbana-Champaign.</span></span> <span data-ttu-id="9bcb7-358">https://doi.org/10.13012/J8PN93H8</span><span class="sxs-lookup"><span data-stu-id="9bcb7-358">https://doi.org/10.13012/J8PN93H8</span></span>
