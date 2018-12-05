---
title: Procesamiento de flujos de datos con Azure Databricks
description: Creación de una canalización de procesamiento de flujos de datos de un extremo a otro en Azure mediante Azure Databricks
author: petertaylor9999
ms.date: 11/01/2018
ms.openlocfilehash: a7e9df57572c9b3a3b0e4f418f148449aa40b04c
ms.sourcegitcommit: 19a517a2fb70768b3edb9a7c3c37197baa61d9b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/26/2018
ms.locfileid: "52295746"
---
# <a name="stream-processing-with-azure-databricks"></a>Procesamiento de flujos de datos con Azure Databricks

Esta arquitectura de referencia muestra una canalización de [procesamiento de flujos de datos](/azure/architecture/data-guide/big-data/real-time-processing) de un extremo a otro. Este tipo de canalización tiene cuatro fases: ingesta, proceso, almacenamiento y análisis e informes. Para esta arquitectura de referencia, la canalización ingiere datos de dos orígenes, realiza una combinación de los registros relacionados de cada flujo, enriquece el resultado y calcula un promedio en tiempo real. Los resultados se almacenan para su posterior análisis. [**Implemente esta solución**.](#deploy-the-solution)

![](./images/stream-processing-databricks.png)

**Escenario**: una empresa de taxi recopila los datos acerca de cada carrera de taxi. En este escenario, se supone que hay dos dispositivos independientes que envían datos. El taxi tienen un medidor que envía la información acerca de cada carrera: duración, distancia y ubicaciones de recogida y destino. Un dispositivo independiente acepta los pagos de clientes y envía los datos sobre las tarifas. Con el fin de identificar las tendencias, la empresa de taxi desea calcular el promedio de propinas por milla conducida, en tiempo real, en cada barrio.

## <a name="architecture"></a>Arquitectura

La arquitectura consta de los siguientes componentes:

**Orígenes de datos**. En esta arquitectura, hay dos orígenes de datos que generan flujos de datos en tiempo real. El primer flujo de datos contiene información sobre la carrera y, el segundo, contiene información sobre las tarifas. La arquitectura de referencia incluye un generador de datos simulados que lee un conjunto de archivos estáticos e inserta los datos en Event Hubs. En una aplicación real, los orígenes de datos serían los dispositivos instalados en el taxi.

**Azure Event Hubs**. [Event Hubs](/azure/event-hubs/) es un servicio de ingesta de eventos. Esta arquitectura emplea dos instancias de centro de eventos, uno para cada origen de datos. Cada origen de datos envía un flujo de datos al centro de eventos asociado.

**Azure Databricks**. [Azure Databricks](/azure/azure-databricks/) es una plataforma de análisis basada en Apache Spark optimizada para la plataforma de servicios en la nube de Microsoft Azure. Databricks se utiliza para correlacionar los datos de la carrera de taxi y las tarifas, y también para enriquecer los datos correlacionados con los datos de los barrios almacenados en el sistema de archivos de Databricks.

**Cosmos DB**. La salida del trabajo de Azure Databricks es una serie de registros, que se escriben en [Cosmos DB](/azure/cosmos-db/) mediante Cassandra API. Se usa Cassandra API porque admite el modelado de datos de series temporales.

**Azure Log Analytics**. Los datos de registro de aplicaciones que recopila [Azure Monitor](/azure/monitoring-and-diagnostics/) se almacenan en un [área de trabajo de Log Analytics](/azure/log-analytics). Las consultas de Log Analytics pueden usarse para analizar y ver las métricas, e inspeccionar los mensajes de registro para identificar problemas dentro de la aplicación.

## <a name="data-ingestion"></a>Ingesta de datos

Para simular un origen de datos, esta arquitectura de referencia usa el conjunto de datos [New York City Taxi Data](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797)<sup>[[1]](#note1)</sup>. Este conjunto de datos contiene datos acerca de carreras de taxi en la ciudad de Nueva York durante un período de cuatro años (de 2010 a 2013). Contiene dos tipos de registros: datos de carreras y datos de tarifas. Los datos de carreras incluyen la duración del viaje, la distancia de viaje y la ubicación de recogida y destino. Los datos de tarifas incluyen las tarifas, los impuestos y las propinas. Los campos comunes en ambos tipos de registro son la placa y el número de licencia, y el identificador del proveedor. Juntos, estos tres campos identifican un taxi además del conductor. Los datos se almacenan en formato CSV. 

El generador de datos es una aplicación de .NET Core que lee los registros y los envía a Azure Event Hubs. El generador envía los datos de carreras en formato JSON y los datos de tarifas en formato CSV. 

Event Hubs usa [particiones](/azure/event-hubs/event-hubs-features#partitions) para segmentar los datos. Las particiones permiten a los consumidores leer cada partición en paralelo. Cuando se envían datos a Event Hubs, puede especificar explícitamente la clave de partición. En caso contrario, los registros se asignan a las particiones en modo round-robin. 

En este escenario, los datos de carreras y los datos de tarifas deben terminar con el mismo identificador de partición para un taxi determinado. Esto permite a Databricks aplicar un cierto paralelismo cuando se establece una correlación entre los dos flujos. Un registro en la partición *n* de los datos de carreras coincidirá con un registro en la partición de datos *n* de los datos de tarifas.

![](./images/stream-processing-databricks-eh.png)

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

```
taxijob_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| project  mdc_inputRowsPerSecond_d, mdc_durationms_triggerExecution_d  
| render timechart
``` 
### <a name="exceptions-logged-during-stream-query-execution"></a>Excepciones registradas durante la ejecución de consultas de flujos de datos

```
taxijob_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| where Level contains "Error" 
```

### <a name="accumulation-of-malformed-fare-and-ride-data"></a>Acumulación de datos de carrera y tarifas con formato incorrecto

```
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
```
SparkMetric_CL 
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| render timechart 
| where name_s contains "driver.DAGScheduler.job.allJobs" 
```

## <a name="deploy-the-solution"></a>Implementación de la solución

Hay disponible una implementación de esta arquitectura de referencia en [GitHub](https://github.com/mspnp/reference-architectures/tree/master/data). 

### <a name="prerequisites"></a>Requisitos previos

1. Clone, bifurque o descargue el archivo zip del repositorio de GitHub de [arquitecturas de referencia](https://github.com/mspnp/reference-architectures).

2. Instale [Docker](https://www.docker.com/) para ejecutar el generador de datos.

3. Instale la [CLI de Azure 2.0](/cli/azure/install-azure-cli?view=azure-cli-latest).

4. Instale la [CLI de Databricks](https://docs.databricks.com/user-guide/dev-tools/databricks-cli.html).

5. Desde un símbolo del sistema, un símbolo del sistema de Bash o un símbolo del sistema de PowerShell, inicie sesión en su cuenta de Azure como se indica a continuación:
    ```
    az login
    ```
6. Instale un IDE de Java, con los siguientes recursos:
    - JDK 1.8
    - Scala SDK 2.11
    - Maven 3.5.4

### <a name="download-the-new-york-city-taxi-and-neighborhood-data-files"></a>Descargue los archivos de datos de taxis y barrios la ciudad de Nueva York

1. Cree un directorio llamado `DataFile` en el directorio `data/streaming_azuredatabricks`, en el sistema de archivos local.

2. Abra un explorador web y vaya a https://uofi.app.box.com/v/NYCtaxidata/folder/2332219935.

3. Haga clic en el botón **Descargar** en esta página para descargar un archivo zip con todos los datos de taxi de ese año.

4. Extraiga el archivo zip en el directorio `DataFile` local.

    > [!NOTE]
    > Este archivo zip contiene otros archivos zip. No extraiga los archivos zip secundarios.

    La estructura de directorios debe tener el siguiente aspecto:

    ```
    /data
        /streaming_azuredatabricks
            /DataFile
                /FOIL2013
                    trip_data_1.zip
                    trip_data_2.zip
                    trip_data_3.zip
                    ...
    ```

5. Abra un explorador web y vaya a https://www.zillow.com/howto/api/neighborhood-boundaries.htm. 

6. Haga clic en **New York Neighborhood Boundaries** para descargar el archivo.

7. Copie el archivo **ZillowNeighborhoods NY.zip** desde el directorio de **descargas** de su explorador al directorio `DataFile`.

### <a name="deploy-the-azure-resources"></a>Implementación de los recursos de Azure

1. Desde un shell o símbolo del sistema de Windows, ejecute el siguiente comando y siga las indicaciones de inicio de sesión:

    ```bash
    az login
    ```

2. Vaya a la carpeta `data/streaming_azuredatabricks` del repositorio de GitHub.

    ```bash
    cd data/streaming_azuredatabricks
    ```

3. Ejecute los siguientes comandos para implementar los recursos de Azure:

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
        --template-file ./azure/deployresources.json --parameters \
        eventHubNamespace=$eventHubNamespace \
        databricksWorkspaceName=$databricksWorkspaceName \
        cosmosDatabaseAccount=$cosmosDatabaseAccount \
        logAnalyticsWorkspaceName=$logAnalyticsWorkspaceName \
        logAnalyticsWorkspaceRegion=$logAnalyticsWorkspaceRegion
    ```

4. La salida de la implementación se escribe en la consola una vez completada. Busque el siguiente código JSON en la salida:

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
Estos valores son los secretos que se agregarán a los secretos de Databricks en las próximas secciones. Guárdelos de forma segura hasta que los agregue a esas secciones.

### <a name="add-a-cassandra-table-to-the-cosmos-db-account"></a>Adición de una tabla de Cassandra a la cuenta de Cosmos DB

1. En Azure Portal, vaya al grupo de recursos creado en la sección **Implementación de los recursos de Azure**. Haga clic en **Cuenta de Azure Cosmos DB**. Cree una tabla con Cassandra API.

2. En la hoja **Información general**, haga clic en **agregar tabla**.

3. Cuando se abra la hoja **Agregar tabla**, escriba `newyorktaxi` en el cuadro de texto **Nombre de Keyspace**. 

4. En la sección **escriba los comandos CQL para crear la tabla**, escriba `neighborhoodstats` en el cuadro de texto junto a `newyorktaxi`.

5. En el cuadro de texto que aparece a continuación, escriba lo siguiente:
```
(neighborhood text, window_end timestamp, number_of_rides bigint,total_fare_amount double, primary key(neighborhood, window_end))
```
6. En el cuadro de texto **Rendimiento (1 000 - 1 000 000 RU/s)**, escriba el valor `4000`.

7. Haga clic en **OK**.

### <a name="add-the-databricks-secrets-using-the-databricks-cli"></a>Agregue los secretos de Databricks con la CLI de Databricks

En primer lugar, especifique los secretos para el centro de eventos:

1. Use la **CLI de Azure Databricks** instalada en el paso 2 de los requisitos previos para crear el ámbito de los secretos de Azure Databricks:
    ```
    databricks secrets create-scope --scope "azure-databricks-job"
    ```
2. Agregue el secreto para el centro de eventos de carreras de taxi:
    ```
    databricks secrets put --scope "azure-databricks-job" --key "taxi-ride"
    ```
    Una vez ejecutado, este comando abre el editor vi. Escriba el valor de **taxi-ride-eh** de la sección de salida **eventHubs** del paso 4 en la sección *implementar los recursos de Azure*. Guarde y salga de vi.

3. Agregue el secreto para el centro de eventos de tarifas de taxi:
    ```
    databricks secrets put --scope "azure-databricks-job" --key "taxi-fare"
    ```
    Una vez ejecutado, este comando abre el editor vi. Escriba el valor de **taxi-fare-eh** de la sección de salida **eventHubs** del paso 4 en la sección *implementación de los recursos de Azure*. Guarde y salga de vi.

A continuación, especifique los secretos para Cosmos DB:

1. Abra Azure Portal y vaya al grupo de recursos especificado en el paso 3 de la sección **implementación de los recursos de Azure**. Haga clic en la cuenta de Azure Cosmos DB.

2. Use la **CLI de Azure Databricks** para agregar el secreto para el nombre de usuario de Cosmos DB:
    ```
    databricks secrets put --scope azure-databricks-job --key "cassandra-username"
    ```
Una vez ejecutado, este comando abre el editor vi. Escriba el valor de **username** de la sección de salida **CosmosDb** del paso 4 en la sección *implementación de los recursos de Azure*. Guarde y salga de vi.

3. A continuación, agregue el secreto para la contraseña de Cosmos DB:
    ```
    databricks secrets put --scope azure-databricks-job --key "cassandra-password"
    ```

Una vez ejecutado, este comando abre el editor vi. Escriba el valor de **secret** de la sección de salida **CosmosDb** del paso 4 en la sección *implementación de los recursos de Azure*. Guarde y salga de vi.

> [!NOTE]
> Si usa un [ámbito de secretos respaldado por Azure Key Vault](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html#azure-key-vault-backed-scopes), el ámbito debe llamarse **azure-databricks-job** y los secretos deben tener exactamente los mismos nombres que los mencionados.

### <a name="add-the-zillow-neighborhoods-data-file-to-the-databricks-file-system"></a>Agregue el archivo de datos Zillow Neighborhoods al sistema de archivos de Databricks

1. Cree un directorio en el sistema de archivos de Databricks:
    ```bash
    dbfs mkdirs dbfs:/azure-databricks-jobs
    ```

2. Vaya a data/streaming_azuredatabricks/DataFile y escriba lo siguiente:
    ```bash
    dbfs cp ZillowNeighborhoods-NY.zip dbfs:/azure-databricks-jobs
    ```

### <a name="add-the-azure-log-analytics-workspace-id-and-primary-key-to-configuration-files"></a>Adición del identificador de área de trabajo y la clave principal de Azure Log Analytics a los archivos de configuración

Para esta sección, necesita el identificador del área de trabajo y la clave principal de Log Analytics. El identificador de área de trabajo es el valor de **workspaceId** de la sección de salida **logAnalytics**, en el paso 4 de la sección *implementación de los recursos de Azure*. La clave principal es el valor de **secret** de la sección de salida. 

1. Para configurar el registro de log4j, abra data\streaming_azuredatabricks\azure\AzureDataBricksJob\src\main\resources\com\microsoft\pnp\azuredatabricksjob\log4j.properties. Edite los dos valores siguientes:
    ```
    log4j.appender.A1.workspaceId=<Log Analytics workspace ID>
    log4j.appender.A1.secret=<Log Analytics primary key>
    ```

2. Para configurar el registro personalizado, abra data\streaming_azuredatabricks\azure\azure-databricks-monitoring\scripts\metrics.properties. Edite los dos valores siguientes:
    ``` 
    *.sink.loganalytics.workspaceId=<Log Analytics workspace ID>
    *.sink.loganalytics.secret=<Log Analytics primary key>
    ```

### <a name="build-the-jar-files-for-the-databricks-job-and-databricks-monitoring"></a>Compilación de los archivos .jar para la supervisión de Databricks y los trabajos de Databricks

1. Use su IDE de Java para importar el archivo de proyecto de Maven llamado **pom.xml**, ubicado en la raíz del directorio **datos/streaming_azuredatabricks**. 

2. Realice una compilación limpia. La salida de esta compilación es dos archivos llamados **azure-databricks-trabajo-1.0-SNAPSHOT.jar** y **azure-databricks-monitoring-0.9.jar**. 

### <a name="configure-custom-logging-for-the-databricks-job"></a>Configuración del registro personalizado para el trabajo de Databricks

1. Copie el archivo **azure-databricks-monitoring-0.9.jar** al sistema de archivos de Databricks; para ello, escriba el comando siguiente en la **CLI de Databricks**:
    ```
    databricks fs cp --overwrite azure-databricks-monitoring-0.9.jar dbfs:/azure-databricks-job/azure-databricks-monitoring-0.9.jar
    ```

2. Copie las propiedades de registro personalizado de data\streaming_azuredatabricks\azure\azure-databricks-monitoring\scripts\metrics.properties al sistema de archivos de Databricks con el comando siguiente:
    ```
    databricks fs cp --overwrite metrics.properties dbfs:/azure-databricks-job/metrics.properties
    ```

3. Aunque aún no haya decidido un nombre para el clúster de Databricks, seleccione uno ahora. Deberá escribir el nombre debajo, en la ruta de acceso del sistema de archivos de Databricks para el clúster. Copie el script de inicialización de data\streaming_azuredatabricks\azure\azure-databricks-monitoring\scripts\spark.metrics al sistema de archivos de Databricks con el comando siguiente:
    ```
    databricks fs cp --overwrite spark-metrics.sh dbfs:/databricks/init/<cluster-name>/spark-metrics.sh
    ```

### <a name="create-a-databricks-cluster"></a>Creación de un clúster de Databricks

1. En el área de trabajo de Databricks, haga clic en "Clústeres" y en "Crear clúster". Escriba el nombre de clúster que creó en el paso 3 de la sección **configuración del registro personalizado para el trabajo de Databricks**. 

2. Seleccione un modo de clúster **standard**.

3. Establezca **Databricks runtime version** (Versión del entorno de ejecución de Databricks) en **4.3 (includes Apache Spark 2.3.1, Scala 2.11)** [4.3 (incluye Apache Spark 2.3.1, Scala 2.11)]

4. Establezca **Python version** (Versión de Python) en **2**.

5. Establezca **Driver Type** (Tipo de controlador) en **Same as worker** (Igual que el trabajo).

6. Establezca **Worker Type** (Tipo de trabajo) en **Standard_DS3_v2**.

7. Establezca **Min Workers** (Trabajos mínimos) en **2**.

8. Anule la selección de **Enable autoscaling** (Habilitar escalado automático). 

9. En el cuadro de diálogo **Auto Termination** (Finalización automática), haga clic en **Scripts Init**. 

10. Escriba **dbfs:/databricks/init/<cluster-name>/spark-metrics.sh** y sustituya el nombre del clúster creado en el paso 1 para <cluster-name>.

11. Haga clic en el botón **Agregar**.

12. Haga clic en el botón **Create Cluster** (Crear clúster).

### <a name="create-a-databricks-job"></a>Creación de un trabajo de Databricks

1. En el área de trabajo de Databricks, haga clic en "Trabajos", "crear trabajo".

2. Escriba el nombre del trabajo.

3. Haga clic en "set jar" y se abrirá el cuadro de diálogo "Cargar JAR para ejecución".

4. Arrastre el archivo **azure-databricks-job-1.0-SNAPSHOT.jar** que creó en la sección **compilación del archivo .jar para el trabajo de Databricks** hasta el cuadro **Soltar JAR aquí para cargar**.

5. Escriba **com.microsoft.pnp.TaxiCabReader** en el campo **Clase Main**.

6. En el campo Argumentos, escriba lo siguiente:
    ```
    -n jar:file:/dbfs/azure-databricks-jobs/ZillowNeighborhoods-NY.zip!/ZillowNeighborhoods-NY.shp --taxi-ride-consumer-group taxi-ride-eh-cg --taxi-fare-consumer-group taxi-fare-eh-cg --window-interval "1 minute" --cassandra-host <Cosmos DB Cassandra host name from above> 
    ``` 

7. Instale las bibliotecas dependientes siguiendo estos pasos:
    
    1. En la interfaz de usuario de Databricks, haga clic en el botón **inicio**.
    
    2. En la lista desplegable **Users** (Usuarios), haga clic en el nombre de la cuenta de usuario para abrir la configuración del área de trabajo de su cuenta.
    
    3. Haga clic en la flecha desplegable situada junto al nombre de la cuenta, haga clic en **create** (crear) y haga clic en **Library** (Biblioteca) para abrir el cuadro de diálogo **New Library** (Nueva biblioteca).
    
    4. En la lista desplegable **Source** (Origen), seleccione **Maven Coordinate** (Coordenada de Maven).
    
    5. Bajo el encabezado **Install Maven Artifacts** (Instalar artefactos de Maven), escriba `com.microsoft.azure:azure-eventhubs-spark_2.11:2.3.5` en el cuadro de texto **Coordinate** (Coordenada). 
    
    6. Haga clic en **Create Library** (Crear biblioteca) para abrir la ventana **Artifacts** (Artefactos).
    
    7. En **Status on running clusters** (Estados en clústeres en ejecución), active la casilla **Attach automatically to all clusters** (Adjuntar automáticamente a todos los clústeres).
    
    8. Repita los pasos del 1 al 7 para la coordenada de Maven `com.microsoft.azure.cosmosdb:azure-cosmos-cassandra-spark-helper:1.0.0`.
    
    9. Repita los pasos del 1 al 6 para la coordenada de Maven `org.geotools:gt-shapefile:19.2`.
    
    10. Haga clic en **Advanced Options** (Opciones avanzadas).
    
    11. Escriba `http://download.osgeo.org/webdav/geotools/` en el cuadro de texto **Repository** (Repositorio). 
    
    12. Haga clic en **Create Library** (Crear biblioteca) para abrir la ventana **Artifacts** (Artefactos). 
    
    13. En **Status on running clusters** (Estados en clústeres en ejecución), active la casilla **Attach automatically to all clusters** (Adjuntar automáticamente a todos los clústeres).

8. Agregue las bibliotecas dependientes que agregó en el paso 7 al trabajo creado al final del paso 6:
    1. En el área de trabajo de Azure Databricks, haga clic en **Jobs** (Trabajos).

    2. Haga clic en el nombre del trabajo creado en el paso 2 de la sección **creación de un trabajo de Databricks**. 
    
    3. Junto a la sección **Dependent Libraries** (Bibliotecas dependientes), haga clic en **Add** (Agregar) para abrir el cuadro de diálogo **Add Dependent Library** (Agregar biblioteca dependiente). 
    
    4. En **Library From** (Desde biblioteca), seleccione **Workspace** (Área de trabajo).
    
    5. Haga clic en **Users** (Usuarios), en su nombre de usuario y, a continuación, en `azure-eventhubs-spark_2.11:2.3.5`. 
    
    6. Haga clic en **OK**.
    
    7. Repita los pasos del 1 al 6 para `spark-cassandra-connector_2.11:2.3.1` y `gt-shapefile:19.2`.

9. Junto a **Cluster:**, haga clic en **Edit** (Editar). Se abrirá el cuadro de diálogo **Configure Cluster** (Configurar clúster). En la lista desplegable **Cluster Type** (Tipo de clúster), seleccione **Existing Cluster** (Clúster existente). En la lista desplegable **Select Cluster** (Seleccionar clúster), seleccione el clúster creado en la sección **creación de un clúster de Databricks**. Haga clic en **Confirm** (Confirmar).

10. Haga clic en **Run now** (Ejecutar ahora).

### <a name="run-the-data-generator"></a>Ejecución del generador de datos

1. Vaya al directorio `data/streaming_azuredatabricks/onprem` del repositorio de GitHub.

2. Actualice los valores en el archivo **main.env** de la siguiente manera:

    ```
    RIDE_EVENT_HUB=[Connection string for the taxi-ride event hub]
    FARE_EVENT_HUB=[Connection string for the taxi-fare event hub]
    RIDE_DATA_FILE_PATH=/DataFile/FOIL2013
    MINUTES_TO_LEAD=0
    PUSH_RIDE_DATA_FIRST=false
    ```
    La cadena de conexión para el centro de eventos taxi-ride es el valor de **taxi-ride-eh** de la sección de salida **eventHubs** del paso 4 en la sección *implementar los recursos de Azure*. La cadena de conexión para el centro de eventos taxi-fare es el valor de **taxi-fare-eh** de la sección de salida **eventHubs** del paso 4 en la sección *implementar los recursos de Azure*.

3. Ejecute el siguiente comando para compilar la imagen de Docker.

    ```bash
    docker build --no-cache -t dataloader .
    ```

4. Vuelva al directorio principal, `data/stream_azuredatabricks`.

    ```bash
    cd ..
    ```

5. Ejecute el siguiente comando para ejecutar la imagen de Docker.

    ```bash
    docker run -v `pwd`/DataFile:/DataFile --env-file=onprem/main.env dataloader:latest
    ```

La salida debe tener un aspecto similar al siguiente:

```
Created 10000 records for TaxiFare
Created 10000 records for TaxiRide
Created 20000 records for TaxiFare
Created 20000 records for TaxiRide
Created 30000 records for TaxiFare
...
```

Para comprobar que el trabajo de Databricks se ejecuta correctamente, abra Azure Portal y vaya a la base de datos de Cosmos DB. Abra la hoja **Explorador de datos** y examine los datos en la tabla **taxi records**. 

[1] <span id="note1">Donovan, Brian; Work, Dan (2016): New York City Taxi Trip Data (2010-2013). Universidad de Illinois en Urbana-Champaign. https://doi.org/10.13012/J8PN93H8
