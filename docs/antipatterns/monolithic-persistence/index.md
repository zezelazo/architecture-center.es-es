---
title: "Antipatrón Monolithic Persistence"
description: "Colocar todos los datos de una aplicación en un único almacén de datos puede perjudicar el rendimiento."
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: 7f04b9f0805c281068b6b2edaf040683773e6f6e
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2017
---
# <a name="monolithic-persistence-antipattern"></a><span data-ttu-id="dc772-103">Antipatrón Monolithic Persistence</span><span class="sxs-lookup"><span data-stu-id="dc772-103">Monolithic Persistence antipattern</span></span>

<span data-ttu-id="dc772-104">Colocar todos los datos de una aplicación en un único almacén de datos puede perjudicar al rendimiento, ya sea porque da lugar a la contención de recursos o porque el almacén de datos no sea una buena opción para algunos de los datos.</span><span class="sxs-lookup"><span data-stu-id="dc772-104">Putting all of an application's data into a single data store can hurt performance, either because it leads to resource contention, or because the data store is not a good fit for some of the data.</span></span>

## <a name="problem-description"></a><span data-ttu-id="dc772-105">Descripción del problema</span><span class="sxs-lookup"><span data-stu-id="dc772-105">Problem description</span></span>

<span data-ttu-id="dc772-106">Históricamente, las aplicaciones a menudo usaban un único almacén de datos, independientemente de los distintos tipos de datos que la aplicación pudiera tener que almacenar.</span><span class="sxs-lookup"><span data-stu-id="dc772-106">Historically, applications have often used a single data store, regardless of the different types of data that the application might need to store.</span></span> <span data-ttu-id="dc772-107">Normalmente, esto se hace para simplificar el diseño de la aplicación, o bien para que se corresponda con las capacidades actuales del equipo de desarrollo.</span><span class="sxs-lookup"><span data-stu-id="dc772-107">Usually this was done to simplify the application design, or else to match the existing skill set of the development team.</span></span> 

<span data-ttu-id="dc772-108">Los sistemas modernos basados en la nube a menudo tienen requisitos no funcionales y funcionales adicionales, y necesitan almacenar muchos tipos de datos heterogéneos, como documentos, imágenes, datos almacenados en caché, mensajes en cola, registros de aplicación y telemetría.</span><span class="sxs-lookup"><span data-stu-id="dc772-108">Modern cloud-based systems often have additional functional and nonfunctional requirements, and need to store many heterogenous types of data, such as documents, images, cached data, queued messages, application logs, and telemetry.</span></span> <span data-ttu-id="dc772-109">Si se sigue el enfoque tradicional y se coloca toda esta información en el mismo almacén de datos, puede perjudicar el rendimiento, por dos motivos principales:</span><span class="sxs-lookup"><span data-stu-id="dc772-109">Following the traditional approach and putting all of this information into the same data store can hurt performance, for two main reasons:</span></span>

- <span data-ttu-id="dc772-110">Almacenar y recuperar grandes cantidades de datos no relacionados en el mismo almacén de datos puede ocasionar una contención, lo que a su vez conduce a tiempos de respuesta mayores y a errores de conexión.</span><span class="sxs-lookup"><span data-stu-id="dc772-110">Storing and retrieving large amounts of unrelated data in the same data store can cause contention, which in turn leads to slow response times and connection failures.</span></span>
- <span data-ttu-id="dc772-111">Con independencia del almacén de datos elegido, podría no ser la mejor opción para todos los distintos tipos de datos o quizá no se puedan optimizar las operaciones que la aplicación realiza.</span><span class="sxs-lookup"><span data-stu-id="dc772-111">Whichever data store is chosen, it might not be the best fit for all of the different types of data, or it might not be optimized for the operations that the application performs.</span></span> 

<span data-ttu-id="dc772-112">En el ejemplo siguiente se muestra un controlador de ASP.NET Web API que agrega un nuevo registro a una base de datos y también registra el resultado en un registro.</span><span class="sxs-lookup"><span data-stu-id="dc772-112">The following example shows an ASP.NET Web API controller that adds a new record to a database and also records the result to a log.</span></span> <span data-ttu-id="dc772-113">El registro se mantiene en la misma base de datos que los datos empresariales.</span><span class="sxs-lookup"><span data-stu-id="dc772-113">The log is held in the same database as the business data.</span></span> <span data-ttu-id="dc772-114">Puede encontrar el ejemplo completo [aquí][sample-app].</span><span class="sxs-lookup"><span data-stu-id="dc772-114">You can find the complete sample [here][sample-app].</span></span>

```csharp
public class MonoController : ApiController
{
    private static readonly string ProductionDb = ...;

    public async Task<IHttpActionResult> PostAsync([FromBody]string value)
    {
        await DataAccess.InsertPurchaseOrderHeaderAsync(ProductionDb);
        await DataAccess.LogAsync(ProductionDb, LogTableName);
        return Ok();
    }
}
```

<span data-ttu-id="dc772-115">La velocidad a la que se generan registros probablemente afectará al rendimiento de las operaciones empresariales.</span><span class="sxs-lookup"><span data-stu-id="dc772-115">The rate at which log records are generated will probably affect the performance of the business operations.</span></span> <span data-ttu-id="dc772-116">Y, si otro componente, como un monitor de proceso de aplicación, lee y procesa los datos del registro con regularidad, también puede afectar a las operaciones empresariales.</span><span class="sxs-lookup"><span data-stu-id="dc772-116">And if another component, such as an application process monitor, regularly reads and processes the log data, that can also affect the business operations.</span></span>

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="dc772-117">Procedimiento para corregir el problema</span><span class="sxs-lookup"><span data-stu-id="dc772-117">How to fix the problem</span></span>

<span data-ttu-id="dc772-118">Separe los datos según su uso.</span><span class="sxs-lookup"><span data-stu-id="dc772-118">Separate data according to its use.</span></span> <span data-ttu-id="dc772-119">Para cada conjunto de datos, seleccione el almacén de datos que mejor coincida con el que se va a usar.</span><span class="sxs-lookup"><span data-stu-id="dc772-119">For each data set, select a data store that best matches how that data set will be used.</span></span> <span data-ttu-id="dc772-120">En el ejemplo anterior, la aplicación debe registrarse en un almacén independiente de la base de datos que contenga los datos empresariales:</span><span class="sxs-lookup"><span data-stu-id="dc772-120">In the previous example, the application should be logging to a separate store from the database that holds business data:</span></span> 

```csharp
public class PolyController : ApiController
{
    private static readonly string ProductionDb = ...;
    private static readonly string LogDb = ...;

    public async Task<IHttpActionResult> PostAsync([FromBody]string value)
    {
        await DataAccess.InsertPurchaseOrderHeaderAsync(ProductionDb);
        // Log to a different data store.
        await DataAccess.LogAsync(LogDb, LogTableName);
        return Ok();
    }
}
```

## <a name="considerations"></a><span data-ttu-id="dc772-121">Consideraciones</span><span class="sxs-lookup"><span data-stu-id="dc772-121">Considerations</span></span>

- <span data-ttu-id="dc772-122">Separe los datos por la forma en que se usan y cómo se accede a ellos.</span><span class="sxs-lookup"><span data-stu-id="dc772-122">Separate data by the way it is used and how it is accessed.</span></span> <span data-ttu-id="dc772-123">Por ejemplo, no almacene información de registro y datos empresariales en el mismo almacén de datos.</span><span class="sxs-lookup"><span data-stu-id="dc772-123">For example, don't store log information and business data in the same data store.</span></span> <span data-ttu-id="dc772-124">Estos tipos de datos tienen requisitos y patrones de acceso significativamente diferentes.</span><span class="sxs-lookup"><span data-stu-id="dc772-124">These types of data have significantly different requirements and patterns of access.</span></span> <span data-ttu-id="dc772-125">Las entradas del registro son intrínsecamente secuenciales, mientras que es más probable que los datos empresariales requieran acceso aleatorio y, a menudo, relacional.</span><span class="sxs-lookup"><span data-stu-id="dc772-125">Log records are inherently sequential, while business data is more likely to require random access, and is often relational.</span></span>

- <span data-ttu-id="dc772-126">Considere el patrón de acceso a los datos para cada tipo de datos.</span><span class="sxs-lookup"><span data-stu-id="dc772-126">Consider the data access pattern for each type of data.</span></span> <span data-ttu-id="dc772-127">Por ejemplo, almacene los documentos e informes con formato en una base de datos de documentos como [Cosmos DB][CosmosDB], pero use [Azure Redis Cache][ Azure-cache] para almacenar los datos en caché temporalmente.</span><span class="sxs-lookup"><span data-stu-id="dc772-127">For example, store formatted reports and documents in a document database such as [Cosmos DB][CosmosDB], but use [Azure Redis Cache][Azure-cache] to cache temporary data.</span></span>

- <span data-ttu-id="dc772-128">Si sigue esta guía pero todavía alcanza los límites de la base de datos, puede que deba ampliar la base de datos.</span><span class="sxs-lookup"><span data-stu-id="dc772-128">If you follow this guidance but still reach the limits of the database, you may need to scale up the database.</span></span> <span data-ttu-id="dc772-129">También considere la posibilidad de escalar horizontalmente y dividir la carga entre servidores de base de datos.</span><span class="sxs-lookup"><span data-stu-id="dc772-129">Also consider scaling horizontally and partitioning the load across database servers.</span></span> <span data-ttu-id="dc772-130">Sin embargo, esto puede requerir volver a diseñar la aplicación.</span><span class="sxs-lookup"><span data-stu-id="dc772-130">However, partitioning may require redesigning the application.</span></span> <span data-ttu-id="dc772-131">Para más información, consulte [Partitioning guidance][DataPartitioningGuidance] (Creación de particiones de datos).</span><span class="sxs-lookup"><span data-stu-id="dc772-131">For more information, see [Data partitioning][DataPartitioningGuidance].</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="dc772-132">Procedimiento para detectar el problema</span><span class="sxs-lookup"><span data-stu-id="dc772-132">How to detect the problem</span></span>

<span data-ttu-id="dc772-133">El sistema probablemente se ralentizará de forma considerable y terminará produciendo un error, ya que se queda sin recursos como las conexiones de base de datos.</span><span class="sxs-lookup"><span data-stu-id="dc772-133">The system will likely slow down dramatically and eventually fail, as the system runs out of resources such as database connections.</span></span>

<span data-ttu-id="dc772-134">Puede realizar los pasos siguientes para ayudar a identificar la causa:</span><span class="sxs-lookup"><span data-stu-id="dc772-134">You can perform the following steps to help identify the cause.</span></span>

1. <span data-ttu-id="dc772-135">Instrumente el sistema para registrar las estadísticas esenciales de rendimiento.</span><span class="sxs-lookup"><span data-stu-id="dc772-135">Instrument the system to record the key performance statistics.</span></span> <span data-ttu-id="dc772-136">Capture información de temporización para cada operación, así como los lugares en los que la aplicación lee y escribe datos.</span><span class="sxs-lookup"><span data-stu-id="dc772-136">Capture timing information for each operation, as well as the points where the application reads and writes data.</span></span>
1. <span data-ttu-id="dc772-137">Si es posible, supervise el sistema en ejecución durante unos días, en un entorno de producción, para obtener una visión real de cómo se utiliza.</span><span class="sxs-lookup"><span data-stu-id="dc772-137">If possible, monitor the system running for a few days in a production environment to get a real-world view of how the system is used.</span></span> <span data-ttu-id="dc772-138">Si esto no es posible, ejecute pruebas de carga generadas por script con un volumen realista de usuarios virtuales que realizan una serie de operaciones habituales.</span><span class="sxs-lookup"><span data-stu-id="dc772-138">If this is not possible, run scripted load tests with a realistic volume of virtual users performing a typical series of operations.</span></span>
2. <span data-ttu-id="dc772-139">Utilice los datos de telemetría para identificar los períodos con un rendimiento bajo.</span><span class="sxs-lookup"><span data-stu-id="dc772-139">Use the telemetry data to identify periods of poor performance.</span></span>
3. <span data-ttu-id="dc772-140">Identifique a qué almacenes de datos se accedió durante esos períodos.</span><span class="sxs-lookup"><span data-stu-id="dc772-140">Identify which data stores were accessed during those periods.</span></span>
4. <span data-ttu-id="dc772-141">Identifique los recursos de almacenamiento de datos que podrían estar experimentando alguna contención.</span><span class="sxs-lookup"><span data-stu-id="dc772-141">Identify data storage resources that might be experiencing contention.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="dc772-142">Diagnóstico de ejemplo</span><span class="sxs-lookup"><span data-stu-id="dc772-142">Example diagnosis</span></span>

<span data-ttu-id="dc772-143">En las secciones siguientes se aplican estos pasos para la aplicación de ejemplo descrita anteriormente.</span><span class="sxs-lookup"><span data-stu-id="dc772-143">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="instrument-and-monitor-the-system"></a><span data-ttu-id="dc772-144">Instrumentación y supervisión del sistema</span><span class="sxs-lookup"><span data-stu-id="dc772-144">Instrument and monitor the system</span></span>

<span data-ttu-id="dc772-145">El siguiente gráfico muestra los resultados de la aplicación de ejemplo de la prueba de carga descrita antes.</span><span class="sxs-lookup"><span data-stu-id="dc772-145">The following graph shows the results of load testing the sample application described earlier.</span></span> <span data-ttu-id="dc772-146">La prueba usaba una carga por pasos de hasta 1 000 usuarios simultáneos.</span><span class="sxs-lookup"><span data-stu-id="dc772-146">The test used a step load of up to 1000 concurrent users.</span></span>

![Resultados de rendimiento de la prueba de carga para el controlador basado en SQL][MonolithicScenarioLoadTest]

<span data-ttu-id="dc772-148">A medida que aumenta la carga a 700 usuarios, aumenta el rendimiento.</span><span class="sxs-lookup"><span data-stu-id="dc772-148">As the load increases to 700 users, so does the throughput.</span></span> <span data-ttu-id="dc772-149">En ese momento, el rendimiento se estabiliza y parece que el sistema se esté ejecutando a su capacidad máxima.</span><span class="sxs-lookup"><span data-stu-id="dc772-149">But at that point, throughput levels off, and the system appears to be running at its maximum capacity.</span></span> <span data-ttu-id="dc772-150">El promedio de respuesta aumenta gradualmente con la carga de usuarios, lo que evidencia que el sistema no puede hacer frente a la demanda.</span><span class="sxs-lookup"><span data-stu-id="dc772-150">The average response gradually increases with user load, showing that the system can't keep up with demand.</span></span>

### <a name="identify-periods-of-poor-performance"></a><span data-ttu-id="dc772-151">Identificación de períodos con un rendimiento bajo</span><span class="sxs-lookup"><span data-stu-id="dc772-151">Identify periods of poor performance</span></span>

<span data-ttu-id="dc772-152">Si va a supervisar el sistema de producción, podría observar patrones.</span><span class="sxs-lookup"><span data-stu-id="dc772-152">If you are monitoring the production system, you might notice patterns.</span></span> <span data-ttu-id="dc772-153">Por ejemplo, los tiempos de respuesta podrían caer significativamente a la misma hora cada día.</span><span class="sxs-lookup"><span data-stu-id="dc772-153">For example, response times might drop off significantly at the same time each day.</span></span> <span data-ttu-id="dc772-154">Podría deberse a una carga de trabajo normal o a un trabajo por lotes programado, o simplemente a que el sistema tenga más usuarios en determinados momentos.</span><span class="sxs-lookup"><span data-stu-id="dc772-154">This could be caused by a regular workload or scheduled batch job, or just because the system has more users at certain times.</span></span> <span data-ttu-id="dc772-155">Debe centrarse en los datos de telemetría en estos casos.</span><span class="sxs-lookup"><span data-stu-id="dc772-155">You should focus on the telemetry data for these events.</span></span>

<span data-ttu-id="dc772-156">Busque correlaciones entre mayores tiempos de respuesta y la mayor actividad de las bases de datos, o en la E/S en los recursos compartidos.</span><span class="sxs-lookup"><span data-stu-id="dc772-156">Look for correlations between increased response times and increased database activity or I/O to shared resources.</span></span> <span data-ttu-id="dc772-157">Si hay correlaciones, significa que la base de datos podría ser un cuello de botella.</span><span class="sxs-lookup"><span data-stu-id="dc772-157">If there are correlations, it means the database might be a bottleneck.</span></span>

### <a name="identify-which-data-stores-are-accessed-during-those-periods"></a><span data-ttu-id="dc772-158">Identifique a qué almacenes de datos se accedió durante esos períodos.</span><span class="sxs-lookup"><span data-stu-id="dc772-158">Identify which data stores are accessed during those periods</span></span>

<span data-ttu-id="dc772-159">El gráfico siguiente muestra la utilización de las unidades de rendimiento de la base de datos (DTU) durante la prueba de carga.</span><span class="sxs-lookup"><span data-stu-id="dc772-159">The next graph shows the utilization of database throughput units (DTU) during the load test.</span></span> <span data-ttu-id="dc772-160">(Una DTU es una medida de la capacidad disponible y combina el uso de CPU, la asignación de memoria y la velocidad de E/S). El uso de DTU alcanzó rápidamente el 100 %.</span><span class="sxs-lookup"><span data-stu-id="dc772-160">(A DTU is a measure of available capacity, and is a combination of CPU utilization, memory allocation, I/O rate.) Utilization of DTUs quickly reached 100%.</span></span> <span data-ttu-id="dc772-161">Es, aproximadamente, el punto en el que el rendimiento alcanzó su máximo en el gráfico anterior.</span><span class="sxs-lookup"><span data-stu-id="dc772-161">This is roughly the point where throughput peaked in the previous graph.</span></span> <span data-ttu-id="dc772-162">El uso de las bases de datos permaneció muy alto hasta que la prueba finalizó.</span><span class="sxs-lookup"><span data-stu-id="dc772-162">Database utilization remained very high until the test finished.</span></span> <span data-ttu-id="dc772-163">Hay una ligera reducción hacia el final, que podría estar provocada por la limitación, la competencia por las conexiones de base de datos u otros factores.</span><span class="sxs-lookup"><span data-stu-id="dc772-163">There is a slight drop toward the end, which could be caused by throttling, competition for database connections, or other factors.</span></span>

![El monitor de base de datos en el Portal de Azure clásico que muestra el uso de recursos de la base de datos][MonolithicDatabaseUtilization]

### <a name="examine-the-telemetry-for-the-data-stores"></a><span data-ttu-id="dc772-165">Examine la telemetría para los almacenes de datos</span><span class="sxs-lookup"><span data-stu-id="dc772-165">Examine the telemetry for the data stores</span></span>

<span data-ttu-id="dc772-166">Instrumente los almacenes de datos para capturar los detalles de bajo nivel de la actividad.</span><span class="sxs-lookup"><span data-stu-id="dc772-166">Instrument the data stores to capture the low-level details of the activity.</span></span> <span data-ttu-id="dc772-167">En la aplicación de ejemplo, la estadísticas de acceso a los datos mostraban un alto volumen de operaciones de inserción realizadas en ambas tablas: `PurchaseOrderHeader` y `MonoLog`.</span><span class="sxs-lookup"><span data-stu-id="dc772-167">In the sample application, the data access statistics showed a high volume of insert operations performed against both the `PurchaseOrderHeader` table and the `MonoLog` table.</span></span> 

![Estadísticas de acceso a los datos para la aplicación de ejemplo][MonolithicDataAccessStats]

### <a name="identify-resource-contention"></a><span data-ttu-id="dc772-169">Identificación de la contención de recursos</span><span class="sxs-lookup"><span data-stu-id="dc772-169">Identify resource contention</span></span>

<span data-ttu-id="dc772-170">En este momento, puede revisar el código fuente, centrándose en los puntos donde la aplicación accede a los recursos que sufren la contención.</span><span class="sxs-lookup"><span data-stu-id="dc772-170">At this point, you can review the source code, focusing on the points where contended resources are accessed by the application.</span></span> <span data-ttu-id="dc772-171">Busque situaciones como:</span><span class="sxs-lookup"><span data-stu-id="dc772-171">Look for situations such as:</span></span>

- <span data-ttu-id="dc772-172">Datos que están separados lógicamente y se escriban en el mismo almacén.</span><span class="sxs-lookup"><span data-stu-id="dc772-172">Data that is logically separate being written to the same store.</span></span> <span data-ttu-id="dc772-173">Datos como los registros, los informes y los mensajes en cola no deben mantenerse en la misma base de datos que la información empresarial.</span><span class="sxs-lookup"><span data-stu-id="dc772-173">Data such as logs, reports, and queued messages should not be held in the same database as business information.</span></span>
- <span data-ttu-id="dc772-174">Una discrepancia entre la elección del almacén de datos y el tipo de datos, como blobs grandes o documentos XML en una base de datos relacional.</span><span class="sxs-lookup"><span data-stu-id="dc772-174">A mismatch between the choice of data store and the type of data, such as large blobs or XML documents in a relational database.</span></span>
- <span data-ttu-id="dc772-175">Datos con patrones de uso significativamente diferentes que comparten el mismo almacén, como los datos con muchas escrituras y pocas lecturas que se almacenan con datos con pocas escrituras y muchas lecturas.</span><span class="sxs-lookup"><span data-stu-id="dc772-175">Data with significantly different usage patterns that share the same store, such as high-write/low-read data being stored with low-write/high-read data.</span></span>

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="dc772-176">Implementación de la solución y comprobación del resultado</span><span class="sxs-lookup"><span data-stu-id="dc772-176">Implement the solution and verify the result</span></span>

<span data-ttu-id="dc772-177">La aplicación se cambió para escribir los registros en un almacén de datos independiente.</span><span class="sxs-lookup"><span data-stu-id="dc772-177">The application was changed to write logs to a separate data store.</span></span> <span data-ttu-id="dc772-178">Estos son los resultados de la prueba de carga:</span><span class="sxs-lookup"><span data-stu-id="dc772-178">Here are the load test results:</span></span>

![Resultados de rendimiento de la prueba de carga con el controlador Polyglot][PolyglotScenarioLoadTest]

<span data-ttu-id="dc772-180">El patrón de rendimiento es similar al del gráfico anterior, pero el punto en el que alcanza el rendimiento máximo es aproximadamente unas 500 solicitudes por segundo mayor.</span><span class="sxs-lookup"><span data-stu-id="dc772-180">The pattern of throughput is similar to the earlier graph, but the point at which performance peaks is approximately 500 requests per second higher.</span></span> <span data-ttu-id="dc772-181">El tiempo de respuesta promedio es ligeramente inferior.</span><span class="sxs-lookup"><span data-stu-id="dc772-181">The average response time is marginally lower.</span></span> <span data-ttu-id="dc772-182">Sin embargo, estas estadísticas no muestran todo el panorama.</span><span class="sxs-lookup"><span data-stu-id="dc772-182">However, these statistics don't tell the full story.</span></span> <span data-ttu-id="dc772-183">La telemetría para la base de datos empresarial muestra que el uso de DTU alcanza el máximo aproximadamente en un 75 %, en lugar del 100 %.</span><span class="sxs-lookup"><span data-stu-id="dc772-183">Telemetry for the business database shows that DTU utilization peaks at around 75%, rather than 100%.</span></span>

![Monitor de base de datos en el Portal de Azure clásico que muestra el uso de recursos de la base de datos en el escenario con Polyglot][PolyglotDatabaseUtilization]

<span data-ttu-id="dc772-185">De forma similar, la utilización de DTU máxima de la base de datos de registro solo alcanza aproximadamente el 70 %.</span><span class="sxs-lookup"><span data-stu-id="dc772-185">Similarly, the maximum DTU utilization of the log database only reaches about 70%.</span></span> <span data-ttu-id="dc772-186">Las bases de datos ya no son el factor que limita el rendimiento del sistema.</span><span class="sxs-lookup"><span data-stu-id="dc772-186">The databases are no longer the limiting factor in the performance of the system.</span></span>

![Monitor de base de datos en el Portal de Azure clásico que muestra el uso de recursos de la base de datos de registro en el escenario con Polyglot][LogDatabaseUtilization]


## <a name="related-resources"></a><span data-ttu-id="dc772-188">Recursos relacionados</span><span class="sxs-lookup"><span data-stu-id="dc772-188">Related resources</span></span>

- <span data-ttu-id="dc772-189">[Elección del almacén de datos apropiado][data-store-overview]</span><span class="sxs-lookup"><span data-stu-id="dc772-189">[Choose the right data store][data-store-overview]</span></span>
- <span data-ttu-id="dc772-190">[Criterios para elegir un almacén de datos][data-store-comparison]</span><span class="sxs-lookup"><span data-stu-id="dc772-190">[Criteria for choosing a data store][data-store-comparison]</span></span>
- <span data-ttu-id="dc772-191">[Data Access for Highly-Scalable Solutions: Using SQL, NoSQL, and Polyglot Persistence][Data-Access-Guide] (Acceso a datos para soluciones muy escalables: uso de la persistencia de Polyglot, SQL y NoSQL)</span><span class="sxs-lookup"><span data-stu-id="dc772-191">[Data Access for Highly-Scalable Solutions: Using SQL, NoSQL, and Polyglot Persistence][Data-Access-Guide]</span></span>
- <span data-ttu-id="dc772-192">[Creación de particiones de datos][DataPartitioningGuidance]</span><span class="sxs-lookup"><span data-stu-id="dc772-192">[Data partitioning][DataPartitioningGuidance]</span></span>

[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/MonolithicPersistence
[CosmosDB]: http://azure.microsoft.com/services/cosmos-db/
[Azure-cache]: /azure/redis-cache/
[Data-Access-Guide]: https://msdn.microsoft.com/library/dn271399.aspx
[DataPartitioningGuidance]: ../../best-practices/data-partitioning.md
[data-store-overview]: ../../guide/technology-choices/data-store-overview.md
[data-store-comparison]: ../../guide/technology-choices/data-store-comparison.md

[MonolithicScenarioLoadTest]: _images/MonolithicScenarioLoadTest.jpg
[MonolithicDatabaseUtilization]: _images/MonolithicDatabaseUtilization.jpg
[MonolithicDataAccessStats]: _images/MonolithicDataAccessStats.jpg
[PolyglotScenarioLoadTest]: _images/PolyglotScenarioLoadTest.jpg
[PolyglotDatabaseUtilization]: _images/PolyglotDatabaseUtilization.jpg
[LogDatabaseUtilization]: _images/LogDatabaseUtilization.jpg
