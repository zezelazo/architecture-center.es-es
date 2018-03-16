---
title: "Selección de una tecnología de procesamiento de flujos"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 23d9849c14964b0905300f191a41084b589fd127
ms.sourcegitcommit: 943e671a8d522cef5ddc8c6e04848134b03c2de4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/05/2018
---
# <a name="choosing-a-stream-processing-technology-in-azure"></a><span data-ttu-id="873eb-102">Selección de una tecnología de procesamiento de flujos</span><span class="sxs-lookup"><span data-stu-id="873eb-102">Choosing a stream processing technology in Azure</span></span>

<span data-ttu-id="873eb-103">Este artículo compara las opciones de tecnología para procesamiento de flujos en tiempo real en Azure.</span><span class="sxs-lookup"><span data-stu-id="873eb-103">This article compares technology choices for real-time stream processing in Azure.</span></span>

<span data-ttu-id="873eb-104">El procesamiento de flujos en tiempo real usa mensajes de una cola o almacenamiento basado en archivos, procesa los mensajes y reenvía el resultado a otra cola de mensajes, almacén de archivos o base de datos.</span><span class="sxs-lookup"><span data-stu-id="873eb-104">Real-time stream processing consumes messages from either queue or file-based storage, process the messages, and forward the result to another message queue, file store, or database.</span></span> <span data-ttu-id="873eb-105">El procesamiento puede incluir la consulta, el filtrado y la agregación de mensajes.</span><span class="sxs-lookup"><span data-stu-id="873eb-105">Processing may include querying, filtering, and aggregating messages.</span></span> <span data-ttu-id="873eb-106">Los motores de procesamiento de flujos deben poder utilizar un flujo ilimitado de datos y generar resultados con una latencia mínima.</span><span class="sxs-lookup"><span data-stu-id="873eb-106">Stream processing engines must be able to consume an endless streams of data and produce results with minimal latency.</span></span> <span data-ttu-id="873eb-107">Para más información, consulte [Procesamiento en tiempo real](../scenarios/real-time-processing.md).</span><span class="sxs-lookup"><span data-stu-id="873eb-107">For more information, see [Real time processing](../scenarios/real-time-processing.md).</span></span>

## <a name="what-are-your-options-when-choosing-a-technology-for-real-time-processing"></a><span data-ttu-id="873eb-108">¿De qué opciones dispone a la hora de elegir una tecnología de procesamiento en tiempo real?</span><span class="sxs-lookup"><span data-stu-id="873eb-108">What are your options when choosing a technology for real-time processing?</span></span>
<span data-ttu-id="873eb-109">En Azure, los almacenes de datos siguientes cumplirán los requisitos principales para el procesamiento en tiempo real:</span><span class="sxs-lookup"><span data-stu-id="873eb-109">In Azure, all of the following data stores will meet the core requirements supporting real-time processing:</span></span>
- [<span data-ttu-id="873eb-110">Azure Stream Analytics</span><span class="sxs-lookup"><span data-stu-id="873eb-110">Azure Stream Analytics</span></span>](/azure/stream-analytics/)
- [<span data-ttu-id="873eb-111">HDInsight con Spark Streaming</span><span class="sxs-lookup"><span data-stu-id="873eb-111">HDInsight with Spark Streaming</span></span>](/azure/hdinsight/spark/apache-spark-streaming-overview)
- [<span data-ttu-id="873eb-112">Apache Spark en Azure Databricks</span><span class="sxs-lookup"><span data-stu-id="873eb-112">Apache Spark in Azure Databricks</span></span>](/azure/azure-databricks/)
- [<span data-ttu-id="873eb-113">HDInsight con Storm</span><span class="sxs-lookup"><span data-stu-id="873eb-113">HDInsight with Storm</span></span>](/azure/hdinsight/storm/apache-storm-overview)
- [<span data-ttu-id="873eb-114">Azure Functions</span><span class="sxs-lookup"><span data-stu-id="873eb-114">Azure Functions</span></span>](/azure/azure-functions/functions-overview)
- [<span data-ttu-id="873eb-115">Azure App Service</span><span class="sxs-lookup"><span data-stu-id="873eb-115">Azure App Service WebJobs</span></span>](/azure/app-service/web-sites-create-web-jobs)

## <a name="key-selection-criteria"></a><span data-ttu-id="873eb-116">Principales criterios de selección</span><span class="sxs-lookup"><span data-stu-id="873eb-116">Key Selection Criteria</span></span>

<span data-ttu-id="873eb-117">En escenarios de procesamiento en tiempo real, seleccione primero el servicio adecuado para sus necesidades respondiendo a estas preguntas:</span><span class="sxs-lookup"><span data-stu-id="873eb-117">For real-time processing scenarios, begin choosing the appropriate service for your needs by answering these questions:</span></span>

- <span data-ttu-id="873eb-118">¿Prefiere un enfoque declarativo o imperativo para crear una lógica de procesamiento de flujos?</span><span class="sxs-lookup"><span data-stu-id="873eb-118">Do you prefer a declarative or imperative approach to authoring stream processing logic?</span></span>

- <span data-ttu-id="873eb-119">¿Necesita compatibilidad integrada para procesamiento temporal o basado en ventanas?</span><span class="sxs-lookup"><span data-stu-id="873eb-119">Do you need built-in support for temporal processing or windowing?</span></span>

- <span data-ttu-id="873eb-120">¿Recibe datos en formatos distintos a Avro, JSON o CSV?</span><span class="sxs-lookup"><span data-stu-id="873eb-120">Does your data arrive in formats besides Avro, JSON, or CSV?</span></span> <span data-ttu-id="873eb-121">Si es así, considere la posibilidad de utilizar opciones que admitan cualquier formato que use código personalizado.</span><span class="sxs-lookup"><span data-stu-id="873eb-121">If yes, consider options support any format using custom code.</span></span>

- <span data-ttu-id="873eb-122">¿Necesita escalar el procesamiento más allá de 1 GB/s?</span><span class="sxs-lookup"><span data-stu-id="873eb-122">Do you need to scale your processing beyond 1 GB/s?</span></span> <span data-ttu-id="873eb-123">En caso afirmativo, considere la posibilidad de usar opciones que se escalan con el tamaño del clúster.</span><span class="sxs-lookup"><span data-stu-id="873eb-123">If yes, consider the options that scale with the cluster size.</span></span> 

## <a name="capability-matrix"></a><span data-ttu-id="873eb-124">Matriz de funcionalidades</span><span class="sxs-lookup"><span data-stu-id="873eb-124">Capability matrix</span></span>

<span data-ttu-id="873eb-125">En las tablas siguientes se resumen las diferencias clave en cuanto a funcionalidades.</span><span class="sxs-lookup"><span data-stu-id="873eb-125">The following tables summarize the key differences in capabilities.</span></span> 

### <a name="general-capabilities"></a><span data-ttu-id="873eb-126">Funcionalidades generales</span><span class="sxs-lookup"><span data-stu-id="873eb-126">General capabilities</span></span>
| | <span data-ttu-id="873eb-127">Azure Stream Analytics</span><span class="sxs-lookup"><span data-stu-id="873eb-127">Azure Stream Analytics</span></span> | <span data-ttu-id="873eb-128">HDInsight con Spark Streaming</span><span class="sxs-lookup"><span data-stu-id="873eb-128">HDInsight with Spark Streaming</span></span> | <span data-ttu-id="873eb-129">Apache Spark en Azure Databricks</span><span class="sxs-lookup"><span data-stu-id="873eb-129">Apache Spark in Azure Databricks</span></span> | <span data-ttu-id="873eb-130">HDInsight con Storm</span><span class="sxs-lookup"><span data-stu-id="873eb-130">HDInsight with Storm</span></span> | <span data-ttu-id="873eb-131">Azure Functions</span><span class="sxs-lookup"><span data-stu-id="873eb-131">Azure Functions</span></span> | <span data-ttu-id="873eb-132">Azure App Service WebJobs</span><span class="sxs-lookup"><span data-stu-id="873eb-132">Azure App Service WebJobs</span></span> |
| --- | --- | --- | --- | --- | --- | --- | 
| <span data-ttu-id="873eb-133">Capacidad de programación</span><span class="sxs-lookup"><span data-stu-id="873eb-133">Programmability</span></span> | <span data-ttu-id="873eb-134">Lenguaje de consulta de Stream Analytics, JavaScript</span><span class="sxs-lookup"><span data-stu-id="873eb-134">Stream analytics query language, JavaScript</span></span> | <span data-ttu-id="873eb-135">Scala, Python, Java</span><span class="sxs-lookup"><span data-stu-id="873eb-135">Scala, Python, Java</span></span> | <span data-ttu-id="873eb-136">Scala, Python, Java, R</span><span class="sxs-lookup"><span data-stu-id="873eb-136">Scala, Python, Java, R</span></span> | <span data-ttu-id="873eb-137">Java, C#</span><span class="sxs-lookup"><span data-stu-id="873eb-137">Java, C#</span></span> | <span data-ttu-id="873eb-138">C#, F#, Node.js</span><span class="sxs-lookup"><span data-stu-id="873eb-138">C#, F#, Node.js</span></span> | <span data-ttu-id="873eb-139">C#, Node.js, PHP, Java, Python</span><span class="sxs-lookup"><span data-stu-id="873eb-139">C#, Node.js, PHP, Java, Python</span></span> |
| <span data-ttu-id="873eb-140">Paradigma de programación</span><span class="sxs-lookup"><span data-stu-id="873eb-140">Programming paradigm</span></span> | <span data-ttu-id="873eb-141">Declarativa</span><span class="sxs-lookup"><span data-stu-id="873eb-141">Declarative</span></span> | <span data-ttu-id="873eb-142">Mezcla de declarativa e imperativa</span><span class="sxs-lookup"><span data-stu-id="873eb-142">Mixture of declarative and imperative</span></span> | <span data-ttu-id="873eb-143">Mezcla de declarativa e imperativa</span><span class="sxs-lookup"><span data-stu-id="873eb-143">Mixture of declarative and imperative</span></span> | <span data-ttu-id="873eb-144">Imperativa</span><span class="sxs-lookup"><span data-stu-id="873eb-144">Imperative</span></span> | <span data-ttu-id="873eb-145">Imperativa</span><span class="sxs-lookup"><span data-stu-id="873eb-145">Imperative</span></span> | <span data-ttu-id="873eb-146">Imperativa</span><span class="sxs-lookup"><span data-stu-id="873eb-146">Imperative</span></span> |    
| <span data-ttu-id="873eb-147">Modelo de precios</span><span class="sxs-lookup"><span data-stu-id="873eb-147">Pricing model</span></span> | [<span data-ttu-id="873eb-148">Unidades de streaming</span><span class="sxs-lookup"><span data-stu-id="873eb-148">Streaming units</span></span>](https://azure.microsoft.com/pricing/details/stream-analytics/) | <span data-ttu-id="873eb-149">Por hora de clúster</span><span class="sxs-lookup"><span data-stu-id="873eb-149">Per cluster hour</span></span> | [<span data-ttu-id="873eb-150">Unidades de Databricks</span><span class="sxs-lookup"><span data-stu-id="873eb-150">Databricks units</span></span>](https://azure.microsoft.com/pricing/details/databricks/) | <span data-ttu-id="873eb-151">Por hora de clúster</span><span class="sxs-lookup"><span data-stu-id="873eb-151">Per cluster hour</span></span> | <span data-ttu-id="873eb-152">Por ejecución de funciones y consumo de recursos</span><span class="sxs-lookup"><span data-stu-id="873eb-152">Per function execution and resource consumption</span></span> | <span data-ttu-id="873eb-153">Por hora de plan de App Service</span><span class="sxs-lookup"><span data-stu-id="873eb-153">Per app service plan hour</span></span> |  

### <a name="integration-capabilities"></a><span data-ttu-id="873eb-154">Funcionalidades de integración</span><span class="sxs-lookup"><span data-stu-id="873eb-154">Integration capabilities</span></span>
| | <span data-ttu-id="873eb-155">Azure Stream Analytics</span><span class="sxs-lookup"><span data-stu-id="873eb-155">Azure Stream Analytics</span></span> | <span data-ttu-id="873eb-156">HDInsight con Spark Streaming</span><span class="sxs-lookup"><span data-stu-id="873eb-156">HDInsight with Spark Streaming</span></span> | <span data-ttu-id="873eb-157">Apache Spark en Azure Databricks</span><span class="sxs-lookup"><span data-stu-id="873eb-157">Apache Spark in Azure Databricks</span></span> | <span data-ttu-id="873eb-158">HDInsight con Storm</span><span class="sxs-lookup"><span data-stu-id="873eb-158">HDInsight with Storm</span></span> | <span data-ttu-id="873eb-159">Azure Functions</span><span class="sxs-lookup"><span data-stu-id="873eb-159">Azure Functions</span></span> | <span data-ttu-id="873eb-160">Azure App Service WebJobs</span><span class="sxs-lookup"><span data-stu-id="873eb-160">Azure App Service WebJobs</span></span> |
| --- | --- | --- | --- | --- | --- | --- | 
| <span data-ttu-id="873eb-161">Entradas</span><span class="sxs-lookup"><span data-stu-id="873eb-161">Inputs</span></span> | [<span data-ttu-id="873eb-162">Entradas de Stream Analytics</span><span class="sxs-lookup"><span data-stu-id="873eb-162">Stream Analytics inputs</span></span>](/azure/stream-analytics/stream-analytics-define-inputs)  | <span data-ttu-id="873eb-163">Event Hubs, IoT Hub, Kafka, HDFS, blobs de Storage, Azure Data Lake Store</span><span class="sxs-lookup"><span data-stu-id="873eb-163">Event Hubs, IoT Hub, Kafka, HDFS, Storage Blobs, Azure Data Lake Store</span></span>  | <span data-ttu-id="873eb-164">Event Hubs, IoT Hub, Kafka, HDFS, blobs de Storage, Azure Data Lake Store</span><span class="sxs-lookup"><span data-stu-id="873eb-164">Event Hubs, IoT Hub, Kafka, HDFS, Storage Blobs, Azure Data Lake Store</span></span>  | <span data-ttu-id="873eb-165">Event Hubs, IoT Hub, blobs de Storage, Azure Data Lake Store</span><span class="sxs-lookup"><span data-stu-id="873eb-165">Event Hubs, IoT Hub, Storage Blobs, Azure Data Lake Store</span></span>  | [<span data-ttu-id="873eb-166">Enlaces admitidos</span><span class="sxs-lookup"><span data-stu-id="873eb-166">Supported bindings</span></span>](/azure/azure-functions/functions-triggers-bindings#supported-bindings) | <span data-ttu-id="873eb-167">Service Bus, colas de Storage, blobs de Storage, Event Hubs, WebHooks, Cosmos DB, Files</span><span class="sxs-lookup"><span data-stu-id="873eb-167">Service Bus, Storage Queues, Storage Blobs, Event Hubs, WebHooks, Cosmos DB, Files</span></span> |
| <span data-ttu-id="873eb-168">Receptores</span><span class="sxs-lookup"><span data-stu-id="873eb-168">Sinks</span></span> |  [<span data-ttu-id="873eb-169">Salidas de Stream Analytics</span><span class="sxs-lookup"><span data-stu-id="873eb-169">Stream Analytics outputs</span></span>](/azure/stream-analytics/stream-analytics-define-outputs) | <span data-ttu-id="873eb-170">HDFS, Kafka, blobs de Storage, Azure Data Lake Store, Cosmos DB</span><span class="sxs-lookup"><span data-stu-id="873eb-170">HDFS, Kafka, Storage Blobs, Azure Data Lake Store, Cosmos DB</span></span> | <span data-ttu-id="873eb-171">HDFS, Kafka, blobs de Storage, Azure Data Lake Store, Cosmos DB</span><span class="sxs-lookup"><span data-stu-id="873eb-171">HDFS, Kafka, Storage Blobs, Azure Data Lake Store, Cosmos DB</span></span> | <span data-ttu-id="873eb-172">Event Hubs, Service Bus, Kafka</span><span class="sxs-lookup"><span data-stu-id="873eb-172">Event Hubs, Service Bus, Kafka</span></span> | [<span data-ttu-id="873eb-173">Enlaces admitidos</span><span class="sxs-lookup"><span data-stu-id="873eb-173">Supported bindings</span></span>](/azure/azure-functions/functions-triggers-bindings#supported-bindings) | <span data-ttu-id="873eb-174">Service Bus, colas de Storage, blobs de Storage, Event Hubs, WebHooks, Cosmos DB, Files</span><span class="sxs-lookup"><span data-stu-id="873eb-174">Service Bus, Storage Queues, Storage Blobs, Event Hubs, WebHooks, Cosmos DB, Files</span></span> | 

### <a name="processing-capabilities"></a><span data-ttu-id="873eb-175">Funcionalidades de procesamiento</span><span class="sxs-lookup"><span data-stu-id="873eb-175">Processing capabilities</span></span>
| | <span data-ttu-id="873eb-176">Azure Stream Analytics</span><span class="sxs-lookup"><span data-stu-id="873eb-176">Azure Stream Analytics</span></span> | <span data-ttu-id="873eb-177">HDInsight con Spark Streaming</span><span class="sxs-lookup"><span data-stu-id="873eb-177">HDInsight with Spark Streaming</span></span> | <span data-ttu-id="873eb-178">Apache Spark en Azure Databricks</span><span class="sxs-lookup"><span data-stu-id="873eb-178">Apache Spark in Azure Databricks</span></span> | <span data-ttu-id="873eb-179">HDInsight con Storm</span><span class="sxs-lookup"><span data-stu-id="873eb-179">HDInsight with Storm</span></span> | <span data-ttu-id="873eb-180">Azure Functions</span><span class="sxs-lookup"><span data-stu-id="873eb-180">Azure Functions</span></span> | <span data-ttu-id="873eb-181">Azure App Service WebJobs</span><span class="sxs-lookup"><span data-stu-id="873eb-181">Azure App Service WebJobs</span></span> |
| --- | --- | --- | --- | --- | --- | --- | 
| <span data-ttu-id="873eb-182">Compatibilidad integrada con almacenamiento temporal o basado en ventanas</span><span class="sxs-lookup"><span data-stu-id="873eb-182">Built-in temporal/windowing support</span></span> | <span data-ttu-id="873eb-183">Sí</span><span class="sxs-lookup"><span data-stu-id="873eb-183">Yes</span></span> | <span data-ttu-id="873eb-184">Sí</span><span class="sxs-lookup"><span data-stu-id="873eb-184">Yes</span></span> | <span data-ttu-id="873eb-185">Sí</span><span class="sxs-lookup"><span data-stu-id="873eb-185">Yes</span></span> | <span data-ttu-id="873eb-186">Sí</span><span class="sxs-lookup"><span data-stu-id="873eb-186">Yes</span></span> | <span data-ttu-id="873eb-187">Sin </span><span class="sxs-lookup"><span data-stu-id="873eb-187">No</span></span> | <span data-ttu-id="873eb-188">Sin </span><span class="sxs-lookup"><span data-stu-id="873eb-188">No</span></span> |
| <span data-ttu-id="873eb-189">Formatos de datos de entrada</span><span class="sxs-lookup"><span data-stu-id="873eb-189">Input data formats</span></span> | <span data-ttu-id="873eb-190">Avro, JSON o CSV, con codificación UTF-8</span><span class="sxs-lookup"><span data-stu-id="873eb-190">Avro, JSON or CSV, UTF-8 encoded</span></span> | <span data-ttu-id="873eb-191">Cualquier formato que use código personalizado</span><span class="sxs-lookup"><span data-stu-id="873eb-191">Any format using custom code</span></span> | <span data-ttu-id="873eb-192">Cualquier formato que use código personalizado</span><span class="sxs-lookup"><span data-stu-id="873eb-192">Any format using custom code</span></span> | <span data-ttu-id="873eb-193">Cualquier formato que use código personalizado</span><span class="sxs-lookup"><span data-stu-id="873eb-193">Any format using custom code</span></span> | <span data-ttu-id="873eb-194">Cualquier formato que use código personalizado</span><span class="sxs-lookup"><span data-stu-id="873eb-194">Any format using custom code</span></span> | <span data-ttu-id="873eb-195">Cualquier formato que use código personalizado</span><span class="sxs-lookup"><span data-stu-id="873eb-195">Any format using custom code</span></span> |
| <span data-ttu-id="873eb-196">Escalabilidad</span><span class="sxs-lookup"><span data-stu-id="873eb-196">Scalability</span></span> | [<span data-ttu-id="873eb-197">Particiones de consulta</span><span class="sxs-lookup"><span data-stu-id="873eb-197">Query partitions</span></span>](/azure/stream-analytics/stream-analytics-parallelization) | <span data-ttu-id="873eb-198">Limitada por el tamaño del clúster</span><span class="sxs-lookup"><span data-stu-id="873eb-198">Bounded by cluster size</span></span> | <span data-ttu-id="873eb-199">Limitado por la configuración de escalado del clúster de Databricks</span><span class="sxs-lookup"><span data-stu-id="873eb-199">Bounded by Databricks cluster scale configuration</span></span> | <span data-ttu-id="873eb-200">Limitada por el tamaño del clúster</span><span class="sxs-lookup"><span data-stu-id="873eb-200">Bounded by cluster size</span></span> | <span data-ttu-id="873eb-201">Hasta 200 instancias de aplicación de función procesándose en paralelo</span><span class="sxs-lookup"><span data-stu-id="873eb-201">Up to 200 function app instances processing in parallel</span></span> | <span data-ttu-id="873eb-202">Limitada por la capacidad del plan de App Service</span><span class="sxs-lookup"><span data-stu-id="873eb-202">Bounded by app service plan capacity</span></span> | 
| <span data-ttu-id="873eb-203">Compatibilidad con control de eventos desordenados y llegada tardía</span><span class="sxs-lookup"><span data-stu-id="873eb-203">Late arrival and out of order event handling support</span></span> | <span data-ttu-id="873eb-204">Sí</span><span class="sxs-lookup"><span data-stu-id="873eb-204">Yes</span></span> | <span data-ttu-id="873eb-205">Sí</span><span class="sxs-lookup"><span data-stu-id="873eb-205">Yes</span></span> | <span data-ttu-id="873eb-206">Sí</span><span class="sxs-lookup"><span data-stu-id="873eb-206">Yes</span></span> | <span data-ttu-id="873eb-207">Sí</span><span class="sxs-lookup"><span data-stu-id="873eb-207">Yes</span></span> | <span data-ttu-id="873eb-208">Sin </span><span class="sxs-lookup"><span data-stu-id="873eb-208">No</span></span> | <span data-ttu-id="873eb-209">Sin </span><span class="sxs-lookup"><span data-stu-id="873eb-209">No</span></span> |

<span data-ttu-id="873eb-210">Consulte también:</span><span class="sxs-lookup"><span data-stu-id="873eb-210">See also:</span></span>

- [<span data-ttu-id="873eb-211">Elección de una tecnología de ingesta de mensajes en tiempo real</span><span class="sxs-lookup"><span data-stu-id="873eb-211">Choosing a real-time message ingestion technology</span></span>](./real-time-ingestion.md)
- [<span data-ttu-id="873eb-212">Comparación de Apache Storm y Azure Stream Analytics</span><span class="sxs-lookup"><span data-stu-id="873eb-212">Comparing Apache Storm and Azure Stream Analytics</span></span>](/azure/stream-analytics/stream-analytics-comparison-storm)
- [<span data-ttu-id="873eb-213">Procesamiento en tiempo real</span><span class="sxs-lookup"><span data-stu-id="873eb-213">Real time processing</span></span>](../scenarios/real-time-processing.md)