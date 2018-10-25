---
title: Selección de una tecnología de procesamiento de flujos
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: b78ebe253e927eece38930ba7f6186f485a956f7
ms.sourcegitcommit: fdcacbfdc77370532a4dde776c5d9b82227dff2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49962929"
---
# <a name="choosing-a-stream-processing-technology-in-azure"></a>Selección de una tecnología de procesamiento de flujos

Este artículo compara las opciones de tecnología para procesamiento de flujos en tiempo real en Azure.

El procesamiento de flujos en tiempo real usa mensajes de una cola o almacenamiento basado en archivos, procesa los mensajes y reenvía el resultado a otra cola de mensajes, almacén de archivos o base de datos. El procesamiento puede incluir la consulta, el filtrado y la agregación de mensajes. Los motores de procesamiento de flujos deben poder utilizar un flujo ilimitado de datos y generar resultados con una latencia mínima. Para más información, consulte [Procesamiento en tiempo real](../big-data/real-time-processing.md).

## <a name="what-are-your-options-when-choosing-a-technology-for-real-time-processing"></a>¿De qué opciones dispone a la hora de elegir una tecnología de procesamiento en tiempo real?
En Azure, los almacenes de datos siguientes cumplirán los requisitos principales para el procesamiento en tiempo real:
- [Azure Stream Analytics](/azure/stream-analytics/)
- [HDInsight con Spark Streaming](/azure/hdinsight/spark/apache-spark-streaming-overview)
- [Apache Spark en Azure Databricks](/azure/azure-databricks/)
- [HDInsight con Storm](/azure/hdinsight/storm/apache-storm-overview)
- [Azure Functions](/azure/azure-functions/functions-overview)
- [Azure App Service](/azure/app-service/web-sites-create-web-jobs)

## <a name="key-selection-criteria"></a>Principales criterios de selección

En escenarios de procesamiento en tiempo real, seleccione primero el servicio adecuado para sus necesidades respondiendo a estas preguntas:

- ¿Prefiere un enfoque declarativo o imperativo para crear una lógica de procesamiento de flujos?

- ¿Necesita compatibilidad integrada para procesamiento temporal o basado en ventanas?

- ¿Recibe datos en formatos distintos a Avro, JSON o CSV? Si es así, considere la posibilidad de utilizar opciones que admitan cualquier formato que use código personalizado.

- ¿Necesita escalar el procesamiento más allá de 1 GB/s? En caso afirmativo, considere la posibilidad de usar opciones que se escalan con el tamaño del clúster. 

## <a name="capability-matrix"></a>Matriz de funcionalidades

En las tablas siguientes se resumen las diferencias clave en cuanto a funcionalidades. 

### <a name="general-capabilities"></a>Funcionalidades generales

| | Azure Stream Analytics | HDInsight con Spark Streaming | Apache Spark en Azure Databricks | HDInsight con Storm | Azure Functions | Azure App Service WebJobs |
| --- | --- | --- | --- | --- | --- | --- | 
| Capacidad de programación | Lenguaje de consulta de Stream Analytics, JavaScript | Scala, Python, Java | Scala, Python, Java, R | Java, C# | C#, F#, Node.js | C#, Node.js, PHP, Java, Python |
| Paradigma de programación | Declarativa | Mezcla de declarativa e imperativa | Mezcla de declarativa e imperativa | Imperativa | Imperativa | Imperativa |    
| Modelo de precios | [Unidades de streaming](https://azure.microsoft.com/pricing/details/stream-analytics/) | Por hora de clúster | [Unidades de Databricks](https://azure.microsoft.com/pricing/details/databricks/) | Por hora de clúster | Por ejecución de funciones y consumo de recursos | Por hora de plan de App Service |  

### <a name="integration-capabilities"></a>Funcionalidades de integración

| | Azure Stream Analytics | HDInsight con Spark Streaming | Apache Spark en Azure Databricks | HDInsight con Storm | Azure Functions | Azure App Service WebJobs |
| --- | --- | --- | --- | --- | --- | --- | 
| Entradas | Azure Event Hubs, Azure IoT Hub, Azure Blob Storage  | Event Hubs, IoT Hub, Kafka, HDFS, blobs de Storage, Azure Data Lake Store  | Event Hubs, IoT Hub, Kafka, HDFS, blobs de Storage, Azure Data Lake Store  | Event Hubs, IoT Hub, blobs de Storage, Azure Data Lake Store  | [Enlaces admitidos](/azure/azure-functions/functions-triggers-bindings#supported-bindings) | Service Bus, colas de Storage, blobs de Storage, Event Hubs, WebHooks, Cosmos DB, Files |
| Receptores |  Azure Data Lake Store, Azure SQL Database, Storage Blobs, Event Hubs, Power BI, Table Storage, Service Bus Queues, Service Bus Topics, Cosmos DB, Azure Functions  | HDFS, Kafka, blobs de Storage, Azure Data Lake Store, Cosmos DB | HDFS, Kafka, blobs de Storage, Azure Data Lake Store, Cosmos DB | Event Hubs, Service Bus, Kafka | [Enlaces admitidos](/azure/azure-functions/functions-triggers-bindings#supported-bindings) | Service Bus, colas de Storage, blobs de Storage, Event Hubs, WebHooks, Cosmos DB, Files | 

### <a name="processing-capabilities"></a>Funcionalidades de procesamiento

| | Azure Stream Analytics | HDInsight con Spark Streaming | Apache Spark en Azure Databricks | HDInsight con Storm | Azure Functions | Azure App Service WebJobs |
| --- | --- | --- | --- | --- | --- | --- | 
| Compatibilidad integrada con almacenamiento temporal o basado en ventanas | SÍ | Sí | Sí | SÍ | No | Sin  |
| Formatos de datos de entrada | Avro, JSON o CSV, con codificación UTF-8 | Cualquier formato que use código personalizado | Cualquier formato que use código personalizado | Cualquier formato que use código personalizado | Cualquier formato que use código personalizado | Cualquier formato que use código personalizado |
| Escalabilidad | [Particiones de consulta](/azure/stream-analytics/stream-analytics-parallelization) | Limitada por el tamaño del clúster | Limitado por la configuración de escalado del clúster de Databricks | Limitada por el tamaño del clúster | Hasta 200 instancias de aplicación de función procesándose en paralelo | Limitada por la capacidad del plan de App Service | 
| Compatibilidad con control de eventos desordenados y llegada tardía | SÍ | Sí | Sí | SÍ | No | Sin  |

Consulte también:

- [Elección de una tecnología de ingesta de mensajes en tiempo real](./real-time-ingestion.md)
- [Procesamiento en tiempo real](../big-data/real-time-processing.md)
