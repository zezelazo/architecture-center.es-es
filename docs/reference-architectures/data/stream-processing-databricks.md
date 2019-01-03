---
title: Procesamiento de flujos de datos con Azure Databricks
titleSuffix: Azure Reference Architectures
description: Cree una canalización de procesamiento de flujos de datos de un extremo a otro en Azure mediante Azure Databricks.
author: petertaylor9999
ms.date: 11/30/2018
ms.custom: seodec18
ms.openlocfilehash: f7364334f889388ad432efadd46362a9fa82fe8b
ms.sourcegitcommit: bb7fcffbb41e2c26a26f8781df32825eb60df70c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/20/2018
ms.locfileid: "53644128"
---
# <a name="create-a-stream-processing-pipeline-with-azure-databricks"></a><span data-ttu-id="07f39-103">Creación de una canalización de procesamiento de flujos de datos con Azure Databricks</span><span class="sxs-lookup"><span data-stu-id="07f39-103">Create a stream processing pipeline with Azure Databricks</span></span>

<span data-ttu-id="07f39-104">Esta arquitectura de referencia muestra una canalización de [procesamiento de flujos de datos](/azure/architecture/data-guide/big-data/real-time-processing) de un extremo a otro.</span><span class="sxs-lookup"><span data-stu-id="07f39-104">This reference architecture shows an end-to-end [stream processing](/azure/architecture/data-guide/big-data/real-time-processing) pipeline.</span></span> <span data-ttu-id="07f39-105">Este tipo de canalización tiene cuatro fases: ingesta, proceso, almacenamiento y análisis e informes.</span><span class="sxs-lookup"><span data-stu-id="07f39-105">This type of pipeline has four stages: ingest, process, store, and analysis and reporting.</span></span> <span data-ttu-id="07f39-106">Para esta arquitectura de referencia, la canalización ingiere datos de dos orígenes, realiza una combinación de los registros relacionados de cada flujo, enriquece el resultado y calcula un promedio en tiempo real.</span><span class="sxs-lookup"><span data-stu-id="07f39-106">For this reference architecture, the pipeline ingests data from two sources, performs a join on related records from each stream, enriches the result, and calculates an average in real time.</span></span> <span data-ttu-id="07f39-107">Los resultados se almacenan para su posterior análisis.</span><span class="sxs-lookup"><span data-stu-id="07f39-107">The results are stored for further analysis.</span></span> <span data-ttu-id="07f39-108">[**Implemente esta solución**](#deploy-the-solution).</span><span class="sxs-lookup"><span data-stu-id="07f39-108">[**Deploy this solution**](#deploy-the-solution).</span></span>

![Arquitectura de referencia para el procesamiento de flujos de datos con Azure Databricks](./images/stream-processing-databricks.png)

<span data-ttu-id="07f39-110">**Escenario**: Una empresa de taxi recopila los datos acerca de cada carrera de taxi.</span><span class="sxs-lookup"><span data-stu-id="07f39-110">**Scenario**: A taxi company collects data about each taxi trip.</span></span> <span data-ttu-id="07f39-111">En este escenario, se supone que hay dos dispositivos independientes que envían datos.</span><span class="sxs-lookup"><span data-stu-id="07f39-111">For this scenario, we assume there are two separate devices sending data.</span></span> <span data-ttu-id="07f39-112">El taxi tienen un medidor que envía la información acerca de cada carrera: duración, distancia y ubicaciones de recogida y destino.</span><span class="sxs-lookup"><span data-stu-id="07f39-112">The taxi has a meter that sends information about each ride &mdash; the duration, distance, and pickup and dropoff locations.</span></span> <span data-ttu-id="07f39-113">Un dispositivo independiente acepta los pagos de clientes y envía los datos sobre las tarifas.</span><span class="sxs-lookup"><span data-stu-id="07f39-113">A separate device accepts payments from customers and sends data about fares.</span></span> <span data-ttu-id="07f39-114">Con el fin de identificar las tendencias, la empresa de taxi desea calcular el promedio de propinas por milla conducida, en tiempo real, en cada barrio.</span><span class="sxs-lookup"><span data-stu-id="07f39-114">To spot ridership trends, the taxi company wants to calculate the average tip per mile driven, in real time, for each neighborhood.</span></span>

## <a name="architecture"></a><span data-ttu-id="07f39-115">Arquitectura</span><span class="sxs-lookup"><span data-stu-id="07f39-115">Architecture</span></span>

<span data-ttu-id="07f39-116">La arquitectura consta de los siguientes componentes:</span><span class="sxs-lookup"><span data-stu-id="07f39-116">The architecture consists of the following components.</span></span>

<span data-ttu-id="07f39-117">**Orígenes de datos**.</span><span class="sxs-lookup"><span data-stu-id="07f39-117">**Data sources**.</span></span> <span data-ttu-id="07f39-118">En esta arquitectura, hay dos orígenes de datos que generan flujos de datos en tiempo real.</span><span class="sxs-lookup"><span data-stu-id="07f39-118">In this architecture, there are two data sources that generate data streams in real time.</span></span> <span data-ttu-id="07f39-119">El primer flujo de datos contiene información sobre la carrera y, el segundo, contiene información sobre las tarifas.</span><span class="sxs-lookup"><span data-stu-id="07f39-119">The first stream contains ride information, and the second contains fare information.</span></span> <span data-ttu-id="07f39-120">La arquitectura de referencia incluye un generador de datos simulados que lee un conjunto de archivos estáticos e inserta los datos en Event Hubs.</span><span class="sxs-lookup"><span data-stu-id="07f39-120">The reference architecture includes a simulated data generator that reads from a set of static files and pushes the data to Event Hubs.</span></span> <span data-ttu-id="07f39-121">En una aplicación real, los orígenes de datos serían los dispositivos instalados en el taxi.</span><span class="sxs-lookup"><span data-stu-id="07f39-121">The data sources in a real application would be devices installed in the taxi cabs.</span></span>

<span data-ttu-id="07f39-122">**Azure Event Hubs**.</span><span class="sxs-lookup"><span data-stu-id="07f39-122">**Azure Event Hubs**.</span></span> <span data-ttu-id="07f39-123">[Event Hubs](/azure/event-hubs/) es un servicio de ingesta de eventos.</span><span class="sxs-lookup"><span data-stu-id="07f39-123">[Event Hubs](/azure/event-hubs/) is an event ingestion service.</span></span> <span data-ttu-id="07f39-124">Esta arquitectura emplea dos instancias de centro de eventos, uno para cada origen de datos.</span><span class="sxs-lookup"><span data-stu-id="07f39-124">This architecture uses two event hub instances, one for each data source.</span></span> <span data-ttu-id="07f39-125">Cada origen de datos envía un flujo de datos al centro de eventos asociado.</span><span class="sxs-lookup"><span data-stu-id="07f39-125">Each data source sends a stream of data to the associated event hub.</span></span>

<span data-ttu-id="07f39-126">**Azure Databricks**.</span><span class="sxs-lookup"><span data-stu-id="07f39-126">**Azure Databricks**.</span></span> <span data-ttu-id="07f39-127">[Azure Databricks](/azure/azure-databricks/) es una plataforma de análisis basada en Apache Spark optimizada para la plataforma de servicios en la nube de Microsoft Azure.</span><span class="sxs-lookup"><span data-stu-id="07f39-127">[Databricks](/azure/azure-databricks/) is an Apache Spark-based analytics platform optimized for the Microsoft Azure cloud services platform.</span></span> <span data-ttu-id="07f39-128">Databricks se utiliza para correlacionar los datos de la carrera de taxi y las tarifas, y también para enriquecer los datos correlacionados con los datos de los barrios almacenados en el sistema de archivos de Databricks.</span><span class="sxs-lookup"><span data-stu-id="07f39-128">Databricks is used to correlate of the taxi ride and fare data, and also to enrich the correlated data with neighborhood data stored in the Databricks file system.</span></span>

<span data-ttu-id="07f39-129">**Cosmos DB**.</span><span class="sxs-lookup"><span data-stu-id="07f39-129">**Cosmos DB**.</span></span> <span data-ttu-id="07f39-130">La salida del trabajo de Azure Databricks es una serie de registros, que se escriben en [Cosmos DB](/azure/cosmos-db/) mediante Cassandra API.</span><span class="sxs-lookup"><span data-stu-id="07f39-130">The output from Azure Databricks job is a series of records, which are written to [Cosmos DB](/azure/cosmos-db/) using the Cassandra API.</span></span> <span data-ttu-id="07f39-131">Se usa Cassandra API porque admite el modelado de datos de series temporales.</span><span class="sxs-lookup"><span data-stu-id="07f39-131">The Cassandra API is used because it supports time series data modeling.</span></span>

<span data-ttu-id="07f39-132">**Azure Log Analytics**.</span><span class="sxs-lookup"><span data-stu-id="07f39-132">**Azure Log Analytics**.</span></span> <span data-ttu-id="07f39-133">Los datos de registro de aplicaciones que recopila [Azure Monitor](/azure/monitoring-and-diagnostics/) se almacenan en un [área de trabajo de Log Analytics](/azure/log-analytics).</span><span class="sxs-lookup"><span data-stu-id="07f39-133">Application log data collected by [Azure Monitor](/azure/monitoring-and-diagnostics/) is stored in a [Log Analytics workspace](/azure/log-analytics).</span></span> <span data-ttu-id="07f39-134">Las consultas de Log Analytics pueden usarse para analizar y ver las métricas, e inspeccionar los mensajes de registro para identificar problemas dentro de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="07f39-134">Log Analytics queries can be used to analyze and visualize metrics and inspect log messages to identify issues within the application.</span></span>

## <a name="data-ingestion"></a><span data-ttu-id="07f39-135">Ingesta de datos</span><span class="sxs-lookup"><span data-stu-id="07f39-135">Data ingestion</span></span>

<!-- markdownlint-disable MD033 -->

<span data-ttu-id="07f39-136">Para simular un origen de datos, esta arquitectura de referencia usa el conjunto de datos [New York City Taxi Data](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797)<sup>[[1]](#note1)</sup>.</span><span class="sxs-lookup"><span data-stu-id="07f39-136">To simulate a data source, this reference architecture uses the [New York City Taxi Data](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797) dataset<sup>[[1]](#note1)</sup>.</span></span> <span data-ttu-id="07f39-137">Este conjunto de datos contiene datos acerca de carreras de taxi en la ciudad de Nueva York durante un período de cuatro años (de 2010 a 2013).</span><span class="sxs-lookup"><span data-stu-id="07f39-137">This dataset contains data about taxi trips in New York City over a four-year period (2010 &ndash; 2013).</span></span> <span data-ttu-id="07f39-138">Contiene dos tipos de registros: datos de carreras y datos de tarifas.</span><span class="sxs-lookup"><span data-stu-id="07f39-138">It contains two types of record: Ride data and fare data.</span></span> <span data-ttu-id="07f39-139">Los datos de carreras incluyen la duración del viaje, la distancia de viaje y la ubicación de recogida y destino.</span><span class="sxs-lookup"><span data-stu-id="07f39-139">Ride data includes trip duration, trip distance, and pickup and dropoff location.</span></span> <span data-ttu-id="07f39-140">Los datos de tarifas incluyen las tarifas, los impuestos y las propinas.</span><span class="sxs-lookup"><span data-stu-id="07f39-140">Fare data includes fare, tax, and tip amounts.</span></span> <span data-ttu-id="07f39-141">Los campos comunes en ambos tipos de registro son la placa y el número de licencia, y el identificador del proveedor.</span><span class="sxs-lookup"><span data-stu-id="07f39-141">Common fields in both record types include medallion number, hack license, and vendor ID.</span></span> <span data-ttu-id="07f39-142">Juntos, estos tres campos identifican un taxi además del conductor.</span><span class="sxs-lookup"><span data-stu-id="07f39-142">Together these three fields uniquely identify a taxi plus a driver.</span></span> <span data-ttu-id="07f39-143">Los datos se almacenan en formato CSV.</span><span class="sxs-lookup"><span data-stu-id="07f39-143">The data is stored in CSV format.</span></span>

> <span data-ttu-id="07f39-144">[1] <span id="note1">Donovan, Brian; Trabajo, Dan (2016): New York City Taxi Trip Data (2010-2013).</span><span class="sxs-lookup"><span data-stu-id="07f39-144">[1] <span id="note1">Donovan, Brian; Work, Dan (2016): New York City Taxi Trip Data (2010-2013).</span></span> <span data-ttu-id="07f39-145">Universidad de Illinois en Urbana-Champaign.</span><span class="sxs-lookup"><span data-stu-id="07f39-145">University of Illinois at Urbana-Champaign.</span></span> <https://doi.org/10.13012/J8PN93H8>

<!-- markdownlint-enable MD033 -->

<span data-ttu-id="07f39-146">El generador de datos es una aplicación de .NET Core que lee los registros y los envía a Azure Event Hubs.</span><span class="sxs-lookup"><span data-stu-id="07f39-146">The data generator is a .NET Core application that reads the records and sends them to Azure Event Hubs.</span></span> <span data-ttu-id="07f39-147">El generador envía los datos de carreras en formato JSON y los datos de tarifas en formato CSV.</span><span class="sxs-lookup"><span data-stu-id="07f39-147">The generator sends ride data in JSON format and fare data in CSV format.</span></span>

<span data-ttu-id="07f39-148">Event Hubs usa [particiones](/azure/event-hubs/event-hubs-features#partitions) para segmentar los datos.</span><span class="sxs-lookup"><span data-stu-id="07f39-148">Event Hubs uses [partitions](/azure/event-hubs/event-hubs-features#partitions) to segment the data.</span></span> <span data-ttu-id="07f39-149">Las particiones permiten a los consumidores leer cada partición en paralelo.</span><span class="sxs-lookup"><span data-stu-id="07f39-149">Partitions allow a consumer to read each partition in parallel.</span></span> <span data-ttu-id="07f39-150">Cuando se envían datos a Event Hubs, puede especificar explícitamente la clave de partición.</span><span class="sxs-lookup"><span data-stu-id="07f39-150">When you send data to Event Hubs, you can specify the partition key explicitly.</span></span> <span data-ttu-id="07f39-151">En caso contrario, los registros se asignan a las particiones en modo round-robin.</span><span class="sxs-lookup"><span data-stu-id="07f39-151">Otherwise, records are assigned to partitions in round-robin fashion.</span></span>

<span data-ttu-id="07f39-152">En este escenario, los datos de carreras y los datos de tarifas deben terminar con el mismo identificador de partición para un taxi determinado.</span><span class="sxs-lookup"><span data-stu-id="07f39-152">In this scenario, ride data and fare data should end up with the same partition ID for a given taxi cab.</span></span> <span data-ttu-id="07f39-153">Esto permite a Databricks aplicar un cierto paralelismo cuando se establece una correlación entre los dos flujos.</span><span class="sxs-lookup"><span data-stu-id="07f39-153">This enables Databricks to apply a degree of parallelism when it correlates the two streams.</span></span> <span data-ttu-id="07f39-154">Un registro en la partición *n* de los datos de carreras coincidirá con un registro en la partición de datos *n* de los datos de tarifas.</span><span class="sxs-lookup"><span data-stu-id="07f39-154">A record in partition *n* of the ride data will match a record in partition *n* of the fare data.</span></span>

![Diagrama de procesamiento de flujos de datos con Azure Databricks y Event Hubs](./images/stream-processing-databricks-eh.png)

<span data-ttu-id="07f39-156">En el generador de datos, el modelo de datos común para ambos tipos de registro tiene una propiedad `PartitionKey` que es la concatenación de `Medallion`, `HackLicense` y `VendorId`.</span><span class="sxs-lookup"><span data-stu-id="07f39-156">In the data generator, the common data model for both record types has a `PartitionKey` property that is the concatenation of `Medallion`, `HackLicense`, and `VendorId`.</span></span>

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

<span data-ttu-id="07f39-157">Esta propiedad se utiliza para proporcionar una clave de partición explícita cuando se realizan envíos a Event Hubs:</span><span class="sxs-lookup"><span data-stu-id="07f39-157">This property is used to provide an explicit partition key when sending to Event Hubs:</span></span>

```csharp
using (var client = pool.GetObject())
{
    return client.Value.SendAsync(new EventData(Encoding.UTF8.GetBytes(
        t.GetData(dataFormat))), t.PartitionKey);
}
```

### <a name="event-hubs"></a><span data-ttu-id="07f39-158">Event Hubs</span><span class="sxs-lookup"><span data-stu-id="07f39-158">Event Hubs</span></span>

<span data-ttu-id="07f39-159">La capacidad de procesamiento de Event Hubs se mide en [unidades de procesamiento](/azure/event-hubs/event-hubs-features#throughput-units).</span><span class="sxs-lookup"><span data-stu-id="07f39-159">The throughput capacity of Event Hubs is measured in [throughput units](/azure/event-hubs/event-hubs-features#throughput-units).</span></span> <span data-ttu-id="07f39-160">Para escalar un centro de eventos automáticamente, puede habilitar el [inflado automático](/azure/event-hubs/event-hubs-auto-inflate), que permite escalar las unidades de procesamiento en función del tráfico, hasta un máximo configurado.</span><span class="sxs-lookup"><span data-stu-id="07f39-160">You can autoscale an event hub by enabling [auto-inflate](/azure/event-hubs/event-hubs-auto-inflate), which automatically scales the throughput units based on traffic, up to a configured maximum.</span></span>

## <a name="stream-processing"></a><span data-ttu-id="07f39-161">Procesamiento de flujos</span><span class="sxs-lookup"><span data-stu-id="07f39-161">Stream processing</span></span>

<span data-ttu-id="07f39-162">En Azure Databricks, el procesamiento de datos se realiza mediante un trabajo.</span><span class="sxs-lookup"><span data-stu-id="07f39-162">In Azure Databricks, data processing is performed by a job.</span></span> <span data-ttu-id="07f39-163">El trabajo está asignado a un clúster y se ejecuta en él.</span><span class="sxs-lookup"><span data-stu-id="07f39-163">The job is assigned to and runs on a cluster.</span></span> <span data-ttu-id="07f39-164">El trabajo puede ser código personalizado escrito en Java o un [cuaderno](https://docs.databricks.com/user-guide/notebooks/index.html) de Spark.</span><span class="sxs-lookup"><span data-stu-id="07f39-164">The job can either be custom code written in Java, or a Spark [notebook](https://docs.databricks.com/user-guide/notebooks/index.html).</span></span>

<span data-ttu-id="07f39-165">En esta arquitectura de referencia, el trabajo es un archivo de Java con clases escritas tanto en Scala como en Java.</span><span class="sxs-lookup"><span data-stu-id="07f39-165">In this reference architecture, the job is a Java archive with classes written in both Java and Scala.</span></span> <span data-ttu-id="07f39-166">Al especificar el archivo de Java para un trabajo de Databricks, se especifica la clase para que el clúster de Databricks lo ejecute.</span><span class="sxs-lookup"><span data-stu-id="07f39-166">When specifying the Java archive for a Databricks job, the class is specified for execution by the Databricks cluster.</span></span> <span data-ttu-id="07f39-167">En este caso, el método **main** de la clase **com.microsoft.pnp.TaxiCabReader** contiene la lógica de procesamiento de datos.</span><span class="sxs-lookup"><span data-stu-id="07f39-167">Here, the **main** method of the **com.microsoft.pnp.TaxiCabReader** class contains the data processing logic.</span></span>

### <a name="reading-the-stream-from-the-two-event-hub-instances"></a><span data-ttu-id="07f39-168">Lectura del flujo de datos de las dos instancias del centro de eventos</span><span class="sxs-lookup"><span data-stu-id="07f39-168">Reading the stream from the two event hub instances</span></span>

<span data-ttu-id="07f39-169">La lógica de procesamiento de datos usa [streaming estructurado de Spark](https://spark.apache.org/docs/2.1.2/structured-streaming-programming-guide.html) para leer de las dos instancias de Azure Event Hub:</span><span class="sxs-lookup"><span data-stu-id="07f39-169">The data processing logic uses [Spark structured streaming](https://spark.apache.org/docs/2.1.2/structured-streaming-programming-guide.html) to read from the two Azure event hub instances:</span></span>

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

### <a name="enriching-the-data-with-the-neighborhood-information"></a><span data-ttu-id="07f39-170">Enriquecimiento de los datos con la información de los barrios</span><span class="sxs-lookup"><span data-stu-id="07f39-170">Enriching the data with the neighborhood information</span></span>

<span data-ttu-id="07f39-171">Los datos de las carreras incluyen las coordenadas de latitud y longitud de las ubicaciones de origen y destino.</span><span class="sxs-lookup"><span data-stu-id="07f39-171">The ride data includes the latitude and longitude coordinates of the pick up and drop off locations.</span></span> <span data-ttu-id="07f39-172">Aunque estas coordenadas son útiles, no se consumen fácilmente en el análisis.</span><span class="sxs-lookup"><span data-stu-id="07f39-172">While these coordinates are useful, they are not easily consumed for analysis.</span></span> <span data-ttu-id="07f39-173">Por lo tanto, estos datos se enriquecen con los datos de los barrios, que se leen desde un [archivo de forma](https://en.wikipedia.org/wiki/Shapefile).</span><span class="sxs-lookup"><span data-stu-id="07f39-173">Therefore, this data is enriched with neighborhood data that is read from a [shapefile](https://en.wikipedia.org/wiki/Shapefile).</span></span>

<span data-ttu-id="07f39-174">El formato del archivo de forma es binario y no se analiza fácilmente, pero la biblioteca [GeoTools](http://geotools.org/) proporciona herramientas para que los datos geoespaciales usen el archivo de forma.</span><span class="sxs-lookup"><span data-stu-id="07f39-174">The shapefile format is binary and not easily parsed, but the [GeoTools](http://geotools.org/) library provides tools for geospatial data that use the shapefile format.</span></span> <span data-ttu-id="07f39-175">Esta biblioteca se utiliza en la clase **com.microsoft.pnp.GeoFinder** para determinar el nombre del barrio en función de las coordenadas de origen y destino.</span><span class="sxs-lookup"><span data-stu-id="07f39-175">This library is used in the **com.microsoft.pnp.GeoFinder** class to determine the neighborhood name based on the pick up and drop off coordinates.</span></span>

```scala
val neighborhoodFinder = (lon: Double, lat: Double) => {
      NeighborhoodFinder.getNeighborhood(lon, lat).get()
    }
```

### <a name="joining-the-ride-and-fare-data"></a><span data-ttu-id="07f39-176">Combinación de los datos de carreras y tarifas</span><span class="sxs-lookup"><span data-stu-id="07f39-176">Joining the ride and fare data</span></span>

<span data-ttu-id="07f39-177">En primer lugar, se transforman los datos de carreras y tarifas:</span><span class="sxs-lookup"><span data-stu-id="07f39-177">First the ride and fare data is transformed:</span></span>

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

<span data-ttu-id="07f39-178">Y, a continuación, los datos de las carreras se combinan con los datos de las tarifas:</span><span class="sxs-lookup"><span data-stu-id="07f39-178">And then the ride data is joined with the fare data:</span></span>

```scala
val mergedTaxiTrip = rides.join(fares, Seq("medallion", "hackLicense", "vendorId", "pickupTime"))
```

### <a name="processing-the-data-and-inserting-into-cosmos-db"></a><span data-ttu-id="07f39-179">Procesamiento de los datos e inserción en Cosmos DB</span><span class="sxs-lookup"><span data-stu-id="07f39-179">Processing the data and inserting into Cosmos DB</span></span>

<span data-ttu-id="07f39-180">La tarifa media para cada barrio se calcula para un intervalo de tiempo determinado:</span><span class="sxs-lookup"><span data-stu-id="07f39-180">The average fare amount for each neighborhood is calculated for a given time interval:</span></span>

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

<span data-ttu-id="07f39-181">Después, se inserta en Cosmos DB:</span><span class="sxs-lookup"><span data-stu-id="07f39-181">Which is then inserted into Cosmos DB:</span></span>

```scala
maxAvgFarePerNeighborhood
      .writeStream
      .queryName("maxAvgFarePerNeighborhood_cassandra_insert")
      .outputMode(OutputMode.Append())
      .foreach(new CassandraSinkForeach(connector))
      .start()
      .awaitTermination()
```

## <a name="security-considerations"></a><span data-ttu-id="07f39-182">Consideraciones sobre la seguridad</span><span class="sxs-lookup"><span data-stu-id="07f39-182">Security considerations</span></span>

<span data-ttu-id="07f39-183">El acceso al área de trabajo de la base de datos de Azure se controla mediante la [consola de administrador](https://docs.databricks.com/administration-guide/admin-settings/index.html).</span><span class="sxs-lookup"><span data-stu-id="07f39-183">Access to the Azure Database workspace is controlled using the [administrator console](https://docs.databricks.com/administration-guide/admin-settings/index.html).</span></span> <span data-ttu-id="07f39-184">La consola de administrador incluye funcionalidad para agregar usuarios, administrar permisos de usuario y configurar el inicio de sesión único.</span><span class="sxs-lookup"><span data-stu-id="07f39-184">The administrator console includes functionality to add users, manage user permissions, and set up single sign-on.</span></span> <span data-ttu-id="07f39-185">El control del acceso para las áreas de trabajo, los clústeres, los trabajos y las tablas también se puede establecer en la consola de administrador.</span><span class="sxs-lookup"><span data-stu-id="07f39-185">Access control for workspaces, clusters, jobs, and tables can also be set through the administrator console.</span></span>

### <a name="managing-secrets"></a><span data-ttu-id="07f39-186">Administración de secretos</span><span class="sxs-lookup"><span data-stu-id="07f39-186">Managing secrets</span></span>

<span data-ttu-id="07f39-187">Azure Databricks incluye un [almacén de secretos](https://docs.azuredatabricks.net/user-guide/secrets/index.html) que se usa para almacenar secretos, incluidas las cadenas de conexión, las claves de acceso, los nombres de usuario y las contraseñas.</span><span class="sxs-lookup"><span data-stu-id="07f39-187">Azure Databricks includes a [secret store](https://docs.azuredatabricks.net/user-guide/secrets/index.html) that is used to store secrets, including connection strings, access keys, user names, and passwords.</span></span> <span data-ttu-id="07f39-188">Los secretos del almacén de secretos de Azure Databricks se particionan por **ámbitos**:</span><span class="sxs-lookup"><span data-stu-id="07f39-188">Secrets within the Azure Databricks secret store are partitioned by **scopes**:</span></span>

```bash
databricks secrets create-scope --scope "azure-databricks-job"
```

<span data-ttu-id="07f39-189">Los secretos se agregan en el nivel de ámbito:</span><span class="sxs-lookup"><span data-stu-id="07f39-189">Secrets are added at the scope level:</span></span>

```bash
databricks secrets put --scope "azure-databricks-job" --key "taxi-ride"
```

> [!NOTE]
> <span data-ttu-id="07f39-190">Se puede utilizar un ámbito respaldado por Azure Key Vault en lugar del ámbito nativo de Azure Databricks.</span><span class="sxs-lookup"><span data-stu-id="07f39-190">An Azure Key Vault-backed scope can be used instead of the native Azure Databricks scope.</span></span> <span data-ttu-id="07f39-191">Para más información, consulte el artículo sobre los [ámbitos respaldados por Azure Key Vault](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html#azure-key-vault-backed-scopes).</span><span class="sxs-lookup"><span data-stu-id="07f39-191">To learn more, see [Azure Key Vault-backed scopes](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html#azure-key-vault-backed-scopes).</span></span>

<span data-ttu-id="07f39-192">En el código, se obtiene acceso a los secretos mediante las [utilidades de secretos](https://docs.databricks.com/user-guide/dev-tools/dbutils.html#secrets-utilities) de Azure Databricks.</span><span class="sxs-lookup"><span data-stu-id="07f39-192">In code, secrets are accessed via the Azure Databricks [secrets utilities](https://docs.databricks.com/user-guide/dev-tools/dbutils.html#secrets-utilities).</span></span>

## <a name="monitoring-considerations"></a><span data-ttu-id="07f39-193">Consideraciones sobre supervisión</span><span class="sxs-lookup"><span data-stu-id="07f39-193">Monitoring considerations</span></span>

<span data-ttu-id="07f39-194">Azure Databricks se basa en Apache Spark, y ambos usan [log4j](https://logging.apache.org/log4j/2.x/) como la biblioteca estándar para el registro.</span><span class="sxs-lookup"><span data-stu-id="07f39-194">Azure Databricks is based on Apache Spark, and both use [log4j](https://logging.apache.org/log4j/2.x/) as the standard library for logging.</span></span> <span data-ttu-id="07f39-195">Además del registro predeterminado proporcionado por Apache Spark, esta arquitectura de referencia envía registros y métricas a [Azure Log Analytics](/azure/log-analytics/).</span><span class="sxs-lookup"><span data-stu-id="07f39-195">In addition to the default logging provided by Apache Spark, this reference architecture sends logs and metrics to [Azure Log Analytics](/azure/log-analytics/).</span></span>

<span data-ttu-id="07f39-196">La clase **com.microsoft.pnp.TaxiCabReader** configura el sistema de registro de Apache Spark para enviar sus registros a Azure Log Analytics usando los valores del archivo **log4j.properties**.</span><span class="sxs-lookup"><span data-stu-id="07f39-196">The **com.microsoft.pnp.TaxiCabReader** class configures the Apache Spark logging system to send its logs to Azure Log Analytics using the values in the **log4j.properties** file.</span></span> <span data-ttu-id="07f39-197">Mientras que los mensajes del registrador de Apache Spark son cadenas, Azure Log Analytics requiere que los mensajes de registro tengan el formato JSON.</span><span class="sxs-lookup"><span data-stu-id="07f39-197">While the Apache Spark logger messages are strings, Azure Log Analytics requires log messages to be formatted as JSON.</span></span> <span data-ttu-id="07f39-198">La clase **com.microsoft.pnp.log4j.LogAnalyticsAppender** transforma estos mensajes al formato JSON:</span><span class="sxs-lookup"><span data-stu-id="07f39-198">The **com.microsoft.pnp.log4j.LogAnalyticsAppender** class transforms these messages to JSON:</span></span>

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

<span data-ttu-id="07f39-199">Como la clase **com.microsoft.pnp.TaxiCabReader** procesa los mensajes de carreras y tarifas, es posible que alguno sea incorrecto y, por tanto, no sea válido.</span><span class="sxs-lookup"><span data-stu-id="07f39-199">As the **com.microsoft.pnp.TaxiCabReader** class processes ride and fare messages, it's possible that either one may be malformed and therefore not valid.</span></span> <span data-ttu-id="07f39-200">En un entorno de producción, es importante analizar estos mensajes incorrectamente formados para identificar un problema con los orígenes de datos, para poder corregirlos rápidamente y evitar la pérdida de datos.</span><span class="sxs-lookup"><span data-stu-id="07f39-200">In a production environment, it's important to analyze these malformed messages to identify a problem with the data sources so it can be fixed quickly to prevent data loss.</span></span> <span data-ttu-id="07f39-201">La clase **com.microsoft.pnp.TaxiCabReader** registra un acumulador de Apache Spark que controla el número de registros de carreras y tarifas con formato incorrecto:</span><span class="sxs-lookup"><span data-stu-id="07f39-201">The **com.microsoft.pnp.TaxiCabReader** class registers an Apache Spark Accumulator that keeps track of the number of malformed fare and ride records:</span></span>

```scala
    @transient val appMetrics = new AppMetrics(spark.sparkContext)
    appMetrics.registerGauge("metrics.malformedrides", AppAccumulators.getRideInstance(spark.sparkContext))
    appMetrics.registerGauge("metrics.malformedfares", AppAccumulators.getFareInstance(spark.sparkContext))
    SparkEnv.get.metricsSystem.registerSource(appMetrics)
```

<span data-ttu-id="07f39-202">Apache Spark utiliza la biblioteca Dropwizard para enviar métricas, y algunos de los campos de métricas nativos de Dropwizard no son compatibles con Azure Log Analytics.</span><span class="sxs-lookup"><span data-stu-id="07f39-202">Apache Spark uses the Dropwizard library to send metrics, and some of the native Dropwizard metrics fields are incompatible with Azure Log Analytics.</span></span> <span data-ttu-id="07f39-203">Por lo tanto, esta arquitectura de referencia incluye un receptor y un notificador de Dropwizard personalizados.</span><span class="sxs-lookup"><span data-stu-id="07f39-203">Therefore, this reference architecture includes a custom Dropwizard sink and reporter.</span></span> <span data-ttu-id="07f39-204">Da formato a las métricas con el formato que Azure Log Analytics espera.</span><span class="sxs-lookup"><span data-stu-id="07f39-204">It formats the metrics in the format expected by Azure Log Analytics.</span></span> <span data-ttu-id="07f39-205">Cuando Apache Spark informa de las métricas, también se envían las métricas personalizadas de los datos de carreras y tarifas con formato incorrecto</span><span class="sxs-lookup"><span data-stu-id="07f39-205">When Apache Spark reports metrics, the custom metrics for the malformed ride and fare data are also sent.</span></span>

<span data-ttu-id="07f39-206">La última métrica que se registra en el área de trabajo de Azure Log Analytics es el progreso acumulado del trabajo de flujo estructurado de Spark.</span><span class="sxs-lookup"><span data-stu-id="07f39-206">The last metric to be logged to the Azure Log Analytics workspace is the cumulative progress of the Spark Structured Streaming job progress.</span></span> <span data-ttu-id="07f39-207">Esto se realiza mediante un agente de escucha StreamingQuery personalizado que se implementa en la clase **com.microsoft.pnp.StreamingMetricsListener**.</span><span class="sxs-lookup"><span data-stu-id="07f39-207">This is done using a custom StreamingQuery listener implemented in the **com.microsoft.pnp.StreamingMetricsListener** class.</span></span> <span data-ttu-id="07f39-208">Esta clase se registra en la sesión de Apache Spark cuando se ejecuta el trabajo:</span><span class="sxs-lookup"><span data-stu-id="07f39-208">This class is registered to the Apache Spark Session when the job runs:</span></span>

```scala
spark.streams.addListener(new StreamingMetricsListener())
```

<span data-ttu-id="07f39-209">El entorno de ejecución de Apache Spark llama a los métodos en StreamingMetricsListener cada vez que se produce un evento de streaming estructurado, y envía los mensajes de registro y las métricas al área de trabajo de Azure Log Analytics.</span><span class="sxs-lookup"><span data-stu-id="07f39-209">The methods in the StreamingMetricsListener are called by the Apache Spark runtime whenever a structured steaming event occurs, sending log messages and metrics to the Azure Log Analytics workspace.</span></span> <span data-ttu-id="07f39-210">Puede usar las siguientes consultas en el área de trabajo para supervisar la aplicación:</span><span class="sxs-lookup"><span data-stu-id="07f39-210">You can use the following queries in your workspace to monitor the application:</span></span>

### <a name="latency-and-throughput-for-streaming-queries"></a><span data-ttu-id="07f39-211">Latencia y rendimiento de las consultas de streaming</span><span class="sxs-lookup"><span data-stu-id="07f39-211">Latency and throughput for streaming queries</span></span>

```shell
taxijob_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| project  mdc_inputRowsPerSecond_d, mdc_durationms_triggerExecution_d
| render timechart
```

### <a name="exceptions-logged-during-stream-query-execution"></a><span data-ttu-id="07f39-212">Excepciones registradas durante la ejecución de consultas de flujos de datos</span><span class="sxs-lookup"><span data-stu-id="07f39-212">Exceptions logged during stream query execution</span></span>

```shell
taxijob_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| where Level contains "Error"
```

### <a name="accumulation-of-malformed-fare-and-ride-data"></a><span data-ttu-id="07f39-213">Acumulación de datos de carrera y tarifas con formato incorrecto</span><span class="sxs-lookup"><span data-stu-id="07f39-213">Accumulation of malformed fare and ride data</span></span>

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

### <a name="job-execution-to-trace-resiliency"></a><span data-ttu-id="07f39-214">Ejecución de trabajos para la resistencia del seguimiento</span><span class="sxs-lookup"><span data-stu-id="07f39-214">Job execution to trace resiliency</span></span>

```shell
SparkMetric_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| render timechart
| where name_s contains "driver.DAGScheduler.job.allJobs"
```

## <a name="deploy-the-solution"></a><span data-ttu-id="07f39-215">Implementación de la solución</span><span class="sxs-lookup"><span data-stu-id="07f39-215">Deploy the solution</span></span>

<span data-ttu-id="07f39-216">Para la implementación y la ejecución de la implementación de referencia, siga los pasos del archivo [Léame de GitHub](https://github.com/mspnp/azure-databricks-streaming-analytics).</span><span class="sxs-lookup"><span data-stu-id="07f39-216">To the deploy and run the reference implementation, follow the steps in the [GitHub readme](https://github.com/mspnp/azure-databricks-streaming-analytics).</span></span>
