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
# <a name="create-a-stream-processing-pipeline-with-azure-databricks"></a>Creación de una canalización de procesamiento de flujos de datos con Azure Databricks

Esta arquitectura de referencia muestra una canalización de [procesamiento de flujos de datos](/azure/architecture/data-guide/big-data/real-time-processing) de un extremo a otro. Este tipo de canalización tiene cuatro fases: ingesta, proceso, almacenamiento y análisis e informes. Para esta arquitectura de referencia, la canalización ingiere datos de dos orígenes, realiza una combinación de los registros relacionados de cada flujo, enriquece el resultado y calcula un promedio en tiempo real. Los resultados se almacenan para su posterior análisis. [**Implemente esta solución**](#deploy-the-solution).

![Arquitectura de referencia para el procesamiento de flujos de datos con Azure Databricks](./images/stream-processing-databricks.png)

**Escenario**: Una empresa de taxi recopila los datos acerca de cada carrera de taxi. En este escenario, se supone que hay dos dispositivos independientes que envían datos. El taxi tienen un medidor que envía la información acerca de cada carrera: duración, distancia y ubicaciones de recogida y destino. Un dispositivo independiente acepta los pagos de clientes y envía los datos sobre las tarifas. Con el fin de identificar las tendencias, la empresa de taxi desea calcular el promedio de propinas por milla conducida, en tiempo real, en cada barrio.

## <a name="architecture"></a>Arquitectura

La arquitectura consta de los siguientes componentes:

**Orígenes de datos**. En esta arquitectura, hay dos orígenes de datos que generan flujos de datos en tiempo real. El primer flujo de datos contiene información sobre la carrera y, el segundo, contiene información sobre las tarifas. La arquitectura de referencia incluye un generador de datos simulados que lee un conjunto de archivos estáticos e inserta los datos en Event Hubs. En una aplicación real, los orígenes de datos serían los dispositivos instalados en el taxi.

**Azure Event Hubs**. [Event Hubs](/azure/event-hubs/) es un servicio de ingesta de eventos. Esta arquitectura emplea dos instancias de centro de eventos, uno para cada origen de datos. Cada origen de datos envía un flujo de datos al centro de eventos asociado.

**Azure Databricks**. [Azure Databricks](/azure/azure-databricks/) es una plataforma de análisis basada en Apache Spark optimizada para la plataforma de servicios en la nube de Microsoft Azure. Databricks se utiliza para correlacionar los datos de la carrera de taxi y las tarifas, y también para enriquecer los datos correlacionados con los datos de los barrios almacenados en el sistema de archivos de Databricks.

**Cosmos DB**. La salida del trabajo de Azure Databricks es una serie de registros, que se escriben en [Cosmos DB](/azure/cosmos-db/) mediante Cassandra API. Se usa Cassandra API porque admite el modelado de datos de series temporales.

**Azure Log Analytics**. Los datos de registro de aplicaciones que recopila [Azure Monitor](/azure/monitoring-and-diagnostics/) se almacenan en un [área de trabajo de Log Analytics](/azure/log-analytics). Las consultas de Log Analytics pueden usarse para analizar y ver las métricas, e inspeccionar los mensajes de registro para identificar problemas dentro de la aplicación.

## <a name="data-ingestion"></a>Ingesta de datos

<!-- markdownlint-disable MD033 -->

Para simular un origen de datos, esta arquitectura de referencia usa el conjunto de datos [New York City Taxi Data](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797)<sup>[[1]](#note1)</sup>. Este conjunto de datos contiene datos acerca de carreras de taxi en la ciudad de Nueva York durante un período de cuatro años (de 2010 a 2013). Contiene dos tipos de registros: datos de carreras y datos de tarifas. Los datos de carreras incluyen la duración del viaje, la distancia de viaje y la ubicación de recogida y destino. Los datos de tarifas incluyen las tarifas, los impuestos y las propinas. Los campos comunes en ambos tipos de registro son la placa y el número de licencia, y el identificador del proveedor. Juntos, estos tres campos identifican un taxi además del conductor. Los datos se almacenan en formato CSV.

> [1] <span id="note1">Donovan, Brian; Trabajo, Dan (2016): New York City Taxi Trip Data (2010-2013). Universidad de Illinois en Urbana-Champaign. <https://doi.org/10.13012/J8PN93H8>

<!-- markdownlint-enable MD033 -->

El generador de datos es una aplicación de .NET Core que lee los registros y los envía a Azure Event Hubs. El generador envía los datos de carreras en formato JSON y los datos de tarifas en formato CSV.

Event Hubs usa [particiones](/azure/event-hubs/event-hubs-features#partitions) para segmentar los datos. Las particiones permiten a los consumidores leer cada partición en paralelo. Cuando se envían datos a Event Hubs, puede especificar explícitamente la clave de partición. En caso contrario, los registros se asignan a las particiones en modo round-robin.

En este escenario, los datos de carreras y los datos de tarifas deben terminar con el mismo identificador de partición para un taxi determinado. Esto permite a Databricks aplicar un cierto paralelismo cuando se establece una correlación entre los dos flujos. Un registro en la partición *n* de los datos de carreras coincidirá con un registro en la partición de datos *n* de los datos de tarifas.

![Diagrama de procesamiento de flujos de datos con Azure Databricks y Event Hubs](./images/stream-processing-databricks-eh.png)

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

### <a name="event-hubs"></a>Event Hubs

La capacidad de procesamiento de Event Hubs se mide en [unidades de procesamiento](/azure/event-hubs/event-hubs-features#throughput-units). Para escalar un centro de eventos automáticamente, puede habilitar el [inflado automático](/azure/event-hubs/event-hubs-auto-inflate), que permite escalar las unidades de procesamiento en función del tráfico, hasta un máximo configurado.

## <a name="stream-processing"></a>Procesamiento de flujos

En Azure Databricks, el procesamiento de datos se realiza mediante un trabajo. El trabajo está asignado a un clúster y se ejecuta en él. El trabajo puede ser código personalizado escrito en Java o un [cuaderno](https://docs.databricks.com/user-guide/notebooks/index.html) de Spark.

En esta arquitectura de referencia, el trabajo es un archivo de Java con clases escritas tanto en Scala como en Java. Al especificar el archivo de Java para un trabajo de Databricks, se especifica la clase para que el clúster de Databricks lo ejecute. En este caso, el método **main** de la clase **com.microsoft.pnp.TaxiCabReader** contiene la lógica de procesamiento de datos.

### <a name="reading-the-stream-from-the-two-event-hub-instances"></a>Lectura del flujo de datos de las dos instancias del centro de eventos

La lógica de procesamiento de datos usa [streaming estructurado de Spark](https://spark.apache.org/docs/2.1.2/structured-streaming-programming-guide.html) para leer de las dos instancias de Azure Event Hub:

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

### <a name="enriching-the-data-with-the-neighborhood-information"></a>Enriquecimiento de los datos con la información de los barrios

Los datos de las carreras incluyen las coordenadas de latitud y longitud de las ubicaciones de origen y destino. Aunque estas coordenadas son útiles, no se consumen fácilmente en el análisis. Por lo tanto, estos datos se enriquecen con los datos de los barrios, que se leen desde un [archivo de forma](https://en.wikipedia.org/wiki/Shapefile).

El formato del archivo de forma es binario y no se analiza fácilmente, pero la biblioteca [GeoTools](http://geotools.org/) proporciona herramientas para que los datos geoespaciales usen el archivo de forma. Esta biblioteca se utiliza en la clase **com.microsoft.pnp.GeoFinder** para determinar el nombre del barrio en función de las coordenadas de origen y destino.

```scala
val neighborhoodFinder = (lon: Double, lat: Double) => {
      NeighborhoodFinder.getNeighborhood(lon, lat).get()
    }
```

### <a name="joining-the-ride-and-fare-data"></a>Combinación de los datos de carreras y tarifas

En primer lugar, se transforman los datos de carreras y tarifas:

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

Y, a continuación, los datos de las carreras se combinan con los datos de las tarifas:

```scala
val mergedTaxiTrip = rides.join(fares, Seq("medallion", "hackLicense", "vendorId", "pickupTime"))
```

### <a name="processing-the-data-and-inserting-into-cosmos-db"></a>Procesamiento de los datos e inserción en Cosmos DB

La tarifa media para cada barrio se calcula para un intervalo de tiempo determinado:

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

Después, se inserta en Cosmos DB:

```scala
maxAvgFarePerNeighborhood
      .writeStream
      .queryName("maxAvgFarePerNeighborhood_cassandra_insert")
      .outputMode(OutputMode.Append())
      .foreach(new CassandraSinkForeach(connector))
      .start()
      .awaitTermination()
```

## <a name="security-considerations"></a>Consideraciones sobre la seguridad

El acceso al área de trabajo de la base de datos de Azure se controla mediante la [consola de administrador](https://docs.databricks.com/administration-guide/admin-settings/index.html). La consola de administrador incluye funcionalidad para agregar usuarios, administrar permisos de usuario y configurar el inicio de sesión único. El control del acceso para las áreas de trabajo, los clústeres, los trabajos y las tablas también se puede establecer en la consola de administrador.

### <a name="managing-secrets"></a>Administración de secretos

Azure Databricks incluye un [almacén de secretos](https://docs.azuredatabricks.net/user-guide/secrets/index.html) que se usa para almacenar secretos, incluidas las cadenas de conexión, las claves de acceso, los nombres de usuario y las contraseñas. Los secretos del almacén de secretos de Azure Databricks se particionan por **ámbitos**:

```bash
databricks secrets create-scope --scope "azure-databricks-job"
```

Los secretos se agregan en el nivel de ámbito:

```bash
databricks secrets put --scope "azure-databricks-job" --key "taxi-ride"
```

> [!NOTE]
> Se puede utilizar un ámbito respaldado por Azure Key Vault en lugar del ámbito nativo de Azure Databricks. Para más información, consulte el artículo sobre los [ámbitos respaldados por Azure Key Vault](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html#azure-key-vault-backed-scopes).

En el código, se obtiene acceso a los secretos mediante las [utilidades de secretos](https://docs.databricks.com/user-guide/dev-tools/dbutils.html#secrets-utilities) de Azure Databricks.

## <a name="monitoring-considerations"></a>Consideraciones sobre supervisión

Azure Databricks se basa en Apache Spark, y ambos usan [log4j](https://logging.apache.org/log4j/2.x/) como la biblioteca estándar para el registro. Además del registro predeterminado proporcionado por Apache Spark, esta arquitectura de referencia envía registros y métricas a [Azure Log Analytics](/azure/log-analytics/).

La clase **com.microsoft.pnp.TaxiCabReader** configura el sistema de registro de Apache Spark para enviar sus registros a Azure Log Analytics usando los valores del archivo **log4j.properties**. Mientras que los mensajes del registrador de Apache Spark son cadenas, Azure Log Analytics requiere que los mensajes de registro tengan el formato JSON. La clase **com.microsoft.pnp.log4j.LogAnalyticsAppender** transforma estos mensajes al formato JSON:

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

Como la clase **com.microsoft.pnp.TaxiCabReader** procesa los mensajes de carreras y tarifas, es posible que alguno sea incorrecto y, por tanto, no sea válido. En un entorno de producción, es importante analizar estos mensajes incorrectamente formados para identificar un problema con los orígenes de datos, para poder corregirlos rápidamente y evitar la pérdida de datos. La clase **com.microsoft.pnp.TaxiCabReader** registra un acumulador de Apache Spark que controla el número de registros de carreras y tarifas con formato incorrecto:

```scala
    @transient val appMetrics = new AppMetrics(spark.sparkContext)
    appMetrics.registerGauge("metrics.malformedrides", AppAccumulators.getRideInstance(spark.sparkContext))
    appMetrics.registerGauge("metrics.malformedfares", AppAccumulators.getFareInstance(spark.sparkContext))
    SparkEnv.get.metricsSystem.registerSource(appMetrics)
```

Apache Spark utiliza la biblioteca Dropwizard para enviar métricas, y algunos de los campos de métricas nativos de Dropwizard no son compatibles con Azure Log Analytics. Por lo tanto, esta arquitectura de referencia incluye un receptor y un notificador de Dropwizard personalizados. Da formato a las métricas con el formato que Azure Log Analytics espera. Cuando Apache Spark informa de las métricas, también se envían las métricas personalizadas de los datos de carreras y tarifas con formato incorrecto

La última métrica que se registra en el área de trabajo de Azure Log Analytics es el progreso acumulado del trabajo de flujo estructurado de Spark. Esto se realiza mediante un agente de escucha StreamingQuery personalizado que se implementa en la clase **com.microsoft.pnp.StreamingMetricsListener**. Esta clase se registra en la sesión de Apache Spark cuando se ejecuta el trabajo:

```scala
spark.streams.addListener(new StreamingMetricsListener())
```

El entorno de ejecución de Apache Spark llama a los métodos en StreamingMetricsListener cada vez que se produce un evento de streaming estructurado, y envía los mensajes de registro y las métricas al área de trabajo de Azure Log Analytics. Puede usar las siguientes consultas en el área de trabajo para supervisar la aplicación:

### <a name="latency-and-throughput-for-streaming-queries"></a>Latencia y rendimiento de las consultas de streaming

```shell
taxijob_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| project  mdc_inputRowsPerSecond_d, mdc_durationms_triggerExecution_d
| render timechart
```

### <a name="exceptions-logged-during-stream-query-execution"></a>Excepciones registradas durante la ejecución de consultas de flujos de datos

```shell
taxijob_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| where Level contains "Error"
```

### <a name="accumulation-of-malformed-fare-and-ride-data"></a>Acumulación de datos de carrera y tarifas con formato incorrecto

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

### <a name="job-execution-to-trace-resiliency"></a>Ejecución de trabajos para la resistencia del seguimiento

```shell
SparkMetric_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| render timechart
| where name_s contains "driver.DAGScheduler.job.allJobs"
```

## <a name="deploy-the-solution"></a>Implementación de la solución

Para la implementación y la ejecución de la implementación de referencia, siga los pasos del archivo [Léame de GitHub](https://github.com/mspnp/azure-databricks-streaming-analytics).
