---
title: Antipatrón Extraneous Fetching
description: Recuperar más datos de los necesarios para una operación comercial puede dar lugar a una sobrecarga de E/S innecesarias y reducir la capacidad de respuesta.
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: 7a72bfd3e4b2e206f3266a046fac2083224ecb4f
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2018
---
# <a name="extraneous-fetching-antipattern"></a><span data-ttu-id="29c39-103">Antipatrón Extraneous Fetching</span><span class="sxs-lookup"><span data-stu-id="29c39-103">Extraneous Fetching antipattern</span></span>

<span data-ttu-id="29c39-104">Recuperar más datos de los necesarios para una operación comercial puede dar lugar a una sobrecarga de E/S innecesarias y reducir la capacidad de respuesta.</span><span class="sxs-lookup"><span data-stu-id="29c39-104">Retrieving more data than needed for a business operation can result in unnecessary I/O overhead and reduce responsiveness.</span></span> 

## <a name="problem-description"></a><span data-ttu-id="29c39-105">Descripción del problema</span><span class="sxs-lookup"><span data-stu-id="29c39-105">Problem description</span></span>

<span data-ttu-id="29c39-106">Este antipatrón puede producirse si la aplicación intenta minimizar las solicitudes de E/S recuperando todos los datos que *podría* necesitar.</span><span class="sxs-lookup"><span data-stu-id="29c39-106">This antipattern can occur if the application tries to minimize I/O requests by retrieving all of the data that it *might* need.</span></span> <span data-ttu-id="29c39-107">A menudo, es el resultado de compensar en exceso el antipatrón [Chatty I/O][chatty-io].</span><span class="sxs-lookup"><span data-stu-id="29c39-107">This is often a result of overcompensating for the [Chatty I/O][chatty-io] antipattern.</span></span> <span data-ttu-id="29c39-108">Por ejemplo, una aplicación podría capturar los detalles de todos los productos de una base de datos.</span><span class="sxs-lookup"><span data-stu-id="29c39-108">For example, an application might fetch the details for every product in a database.</span></span> <span data-ttu-id="29c39-109">Pero puede que el usuario solo tenga un subconjunto de los detalles (algunos pueden no ser pertinentes para los clientes) y probablemente no necesite ver *todos* los productos a la vez.</span><span class="sxs-lookup"><span data-stu-id="29c39-109">But the user may need just a subset of the details (some may not be relevant to customers), and probably doesn't need to see *all* of the products at once.</span></span> <span data-ttu-id="29c39-110">Incluso si el usuario está explorando el catálogo completo, tendría sentido paginar los resultados &mdash;mostrando 20 a la vez, por ejemplo.</span><span class="sxs-lookup"><span data-stu-id="29c39-110">Even if the user is browsing the entire catalog, it would make sense to paginate the results &mdash; showing 20 at a time, for example.</span></span>

<span data-ttu-id="29c39-111">Otro origen de este problema es seguir prácticas de programación o diseño deficientes.</span><span class="sxs-lookup"><span data-stu-id="29c39-111">Another source of this problem is following poor programming or design practices.</span></span> <span data-ttu-id="29c39-112">Por ejemplo, el código siguiente utiliza Entity Framework para capturar los detalles completos para todos los productos.</span><span class="sxs-lookup"><span data-stu-id="29c39-112">For example, the following code uses Entity Framework to fetch the complete details for every product.</span></span> <span data-ttu-id="29c39-113">A continuación, filtra los resultados para devolver únicamente un subconjunto de los campos y descarta el resto.</span><span class="sxs-lookup"><span data-stu-id="29c39-113">Then it filters the results to return only a subset of the fields, discarding the rest.</span></span> <span data-ttu-id="29c39-114">Puede encontrar el ejemplo completo [aquí][sample-app].</span><span class="sxs-lookup"><span data-stu-id="29c39-114">You can find the complete sample [here][sample-app].</span></span>

```csharp
public async Task<IHttpActionResult> GetAllFieldsAsync()
{
    using (var context = new AdventureWorksContext())
    {
        // Execute the query. This happens at the database.
        var products = await context.Products.ToListAsync();

        // Project fields from the query results. This happens in application memory.
        var result = products.Select(p => new ProductInfo { Id = p.ProductId, Name = p.Name });
        return Ok(result);
    }
}
```

<span data-ttu-id="29c39-115">En el ejemplo siguiente, la aplicación recupera los datos para llevar a cabo una agregación que puede realizar la base de datos en su lugar.</span><span class="sxs-lookup"><span data-stu-id="29c39-115">In the next example, the application retrieves data to perform an aggregation that could be done by the database instead.</span></span> <span data-ttu-id="29c39-116">La aplicación calcula el total de ventas obteniendo todos los registros de todos los pedidos vendidos y, a continuación, calculando la suma de esos registros.</span><span class="sxs-lookup"><span data-stu-id="29c39-116">The application calculates total sales by getting every record for all orders sold, and then computing the sum over those records.</span></span> <span data-ttu-id="29c39-117">Puede encontrar el ejemplo completo [aquí][sample-app].</span><span class="sxs-lookup"><span data-stu-id="29c39-117">You can find the complete sample [here][sample-app].</span></span>

```csharp
public async Task<IHttpActionResult> AggregateOnClientAsync()
{
    using (var context = new AdventureWorksContext())
    {
        // Fetch all order totals from the database.
        var orderAmounts = await context.SalesOrderHeaders.Select(soh => soh.TotalDue).ToListAsync();

        // Sum the order totals in memory.
        var total = orderAmounts.Sum();
        return Ok(total);
    }
}
```

<span data-ttu-id="29c39-118">En el ejemplo siguiente se muestra un pequeño problema debido a la forma en que Entity Framework usa LINQ to Entities.</span><span class="sxs-lookup"><span data-stu-id="29c39-118">The next example shows a subtle problem caused by the way Entity Framework uses LINQ to Entities.</span></span> 

```csharp
var query = from p in context.Products.AsEnumerable()
            where p.SellStartDate < DateTime.Now.AddDays(-7) // AddDays cannot be mapped by LINQ to Entities
            select ...;

List<Product> products = query.ToList();
```

<span data-ttu-id="29c39-119">La aplicación está intentando buscar productos con un valor de `SellStartDate` de más de una semana.</span><span class="sxs-lookup"><span data-stu-id="29c39-119">The application is trying to find products with a `SellStartDate` more than a week old.</span></span> <span data-ttu-id="29c39-120">En la mayoría de los casos, LINQ to Entities traduciría una cláusula `where` en una instrucción SQL que la base de datos ejecuta.</span><span class="sxs-lookup"><span data-stu-id="29c39-120">In most cases, LINQ to Entities would translate a `where` clause to a SQL statement that is executed by the database.</span></span> <span data-ttu-id="29c39-121">No obstante, en este caso, LINQ to Entities no puede asignar el método `AddDays` a SQL.</span><span class="sxs-lookup"><span data-stu-id="29c39-121">In this case, however, LINQ to Entities cannot map the `AddDays` method to SQL.</span></span> <span data-ttu-id="29c39-122">En su lugar, se devuelve cada fila de la tabla `Product` y los resultados se filtran en la memoria.</span><span class="sxs-lookup"><span data-stu-id="29c39-122">Instead, every row from the `Product` table is returned, and the results are filtered in memory.</span></span> 

<span data-ttu-id="29c39-123">La llamada a `AsEnumerable` es una sugerencia de que hay un problema.</span><span class="sxs-lookup"><span data-stu-id="29c39-123">The call to `AsEnumerable` is a hint that there is a problem.</span></span> <span data-ttu-id="29c39-124">Este método convierte los resultados a una interfaz `IEnumerable`.</span><span class="sxs-lookup"><span data-stu-id="29c39-124">This method converts the results to an `IEnumerable` interface.</span></span> <span data-ttu-id="29c39-125">Aunque `IEnumerable` admite el filtrado, este se realiza en el *cliente*, no en la base de datos.</span><span class="sxs-lookup"><span data-stu-id="29c39-125">Although `IEnumerable` supports filtering, the filtering is done on the *client* side, not the database.</span></span> <span data-ttu-id="29c39-126">De forma predeterminada, LINQ to Entities usa `IQueryable`, que pasa la responsabilidad del filtrado al origen de datos.</span><span class="sxs-lookup"><span data-stu-id="29c39-126">By default, LINQ to Entities uses `IQueryable`, which passes the responsibility for filtering to the data source.</span></span> 

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="29c39-127">Procedimiento para corregir el problema</span><span class="sxs-lookup"><span data-stu-id="29c39-127">How to fix the problem</span></span>

<span data-ttu-id="29c39-128">Evite capturar grandes volúmenes de datos que puedan quedarse obsoletos rápidamente o podrían descartarse, y capture solo los necesarios para realizar la operación.</span><span class="sxs-lookup"><span data-stu-id="29c39-128">Avoid fetching large volumes of data that may quickly become outdated or might be discarded, and only fetch the data needed for the operation being performed.</span></span> 

<span data-ttu-id="29c39-129">En lugar de obtener todas las columnas de una tabla y, a continuación, filtrarlas después, seleccione las que necesite de la base de datos.</span><span class="sxs-lookup"><span data-stu-id="29c39-129">Instead of getting every column from a table and then filtering them, select the columns that you need from the database.</span></span>

```csharp
public async Task<IHttpActionResult> GetRequiredFieldsAsync()
{
    using (var context = new AdventureWorksContext())
    {
        // Project fields as part of the query itself
        var result = await context.Products
            .Select(p => new ProductInfo {Id = p.ProductId, Name = p.Name})
            .ToListAsync();
        return Ok(result);
    }
}
```

<span data-ttu-id="29c39-130">De igual forma, realice las agregaciones en la base de datos y no en memoria de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="29c39-130">Similarly, perform aggregation in the database and not in application memory.</span></span>

```csharp
public async Task<IHttpActionResult> AggregateOnDatabaseAsync()
{
    using (var context = new AdventureWorksContext())
    {
        // Sum the order totals as part of the database query.
        var total = await context.SalesOrderHeaders.SumAsync(soh => soh.TotalDue);
        return Ok(total);
    }
}
```

<span data-ttu-id="29c39-131">Al utilizar Entity Framework, asegúrese de que las consultas LINQ se resuelven mediante la interfaz `IQueryable` y no `IEnumerable`.</span><span class="sxs-lookup"><span data-stu-id="29c39-131">When using Entity Framework, ensure that LINQ queries are resolved using the `IQueryable`interface and not `IEnumerable`.</span></span> <span data-ttu-id="29c39-132">Puede que tenga que ajustar la consulta para utilizar solo las funciones que se puedan asignar al origen de datos.</span><span class="sxs-lookup"><span data-stu-id="29c39-132">You may need to adjust the query to use only functions that can be mapped to the data source.</span></span> <span data-ttu-id="29c39-133">El ejemplo anterior se puede refactorizar para quitar el método `AddDays` de la consulta, lo que permite que la base de datos realice el filtrado.</span><span class="sxs-lookup"><span data-stu-id="29c39-133">The earlier example can be refactored to remove the `AddDays` method from the query, allowing filtering to be done by the database.</span></span>

```csharp
DateTime dateSince = DateTime.Now.AddDays(-7); // AddDays has been factored out. 
var query = from p in context.Products
            where p.SellStartDate < dateSince // This criterion can be passed to the database by LINQ to Entities
            select ...;

List<Product> products = query.ToList();
```

## <a name="considerations"></a><span data-ttu-id="29c39-134">Consideraciones</span><span class="sxs-lookup"><span data-stu-id="29c39-134">Considerations</span></span>

- <span data-ttu-id="29c39-135">En algunos casos, puede mejorar el rendimiento mediante la partición horizontal de los datos.</span><span class="sxs-lookup"><span data-stu-id="29c39-135">In some cases, you can improve performance by partitioning data horizontally.</span></span> <span data-ttu-id="29c39-136">Si diferentes operaciones acceden a atributos distintos de los datos, crear particiones horizontales puede reducir la contención.</span><span class="sxs-lookup"><span data-stu-id="29c39-136">If different operations access different attributes of the data, horizontal partitioning may reduce contention.</span></span> <span data-ttu-id="29c39-137">A menudo, la mayoría de las operaciones se ejecutan en un pequeño subconjunto de los datos, de modo que diseminar esta carga puede mejorar el rendimiento.</span><span class="sxs-lookup"><span data-stu-id="29c39-137">Often, most operations are run against a small subset of the data, so spreading this load may improve performance.</span></span> <span data-ttu-id="29c39-138">Vea [Creación de particiones de datos][data-partitioning].</span><span class="sxs-lookup"><span data-stu-id="29c39-138">See [Data partitioning][data-partitioning].</span></span>

- <span data-ttu-id="29c39-139">Para las operaciones que tienen que admitir consultas sin limitar, implemente la paginación y capture solo un número limitado de entidades a la vez.</span><span class="sxs-lookup"><span data-stu-id="29c39-139">For operations that have to support unbounded queries, implement pagination and only fetch a limited number of entities at a time.</span></span> <span data-ttu-id="29c39-140">Por ejemplo, si un cliente está explorando un catálogo de productos, puede mostrar una página de resultados al mismo tiempo.</span><span class="sxs-lookup"><span data-stu-id="29c39-140">For example, if a customer is browsing a product catalog, you can show one page of results at a time.</span></span>

- <span data-ttu-id="29c39-141">Cuando sea posible, aproveche las características integradas en el almacén de datos.</span><span class="sxs-lookup"><span data-stu-id="29c39-141">When possible, take advantage of features built into the data store.</span></span> <span data-ttu-id="29c39-142">Por ejemplo, las bases de datos SQL suelen proporcionar funciones de agregado.</span><span class="sxs-lookup"><span data-stu-id="29c39-142">For example, SQL databases typically provide aggregate functions.</span></span> 

- <span data-ttu-id="29c39-143">Si utiliza un almacén de datos que no es compatible con una función determinada, como la agregación, podría almacenar el resultado calculado en otra parte, actualizar el valor a medida que los registros se agregan o se actualizan, de forma que la aplicación no tenga que volver a calcular el valor cada vez que sea necesario.</span><span class="sxs-lookup"><span data-stu-id="29c39-143">If you're using a data store that doesn't support a particular function, such as aggregration, you could store the calculated result elsewhere, updating the value as records are added or updated, so the application doesn't have to recalculate the value each time it's needed.</span></span>

- <span data-ttu-id="29c39-144">Si ve que las solicitudes recuperan un gran número de campos, examine el código fuente para determinar si todos ellos son realmente necesarios.</span><span class="sxs-lookup"><span data-stu-id="29c39-144">If you see that requests are retrieving a large number of fields, examine the source code to determine whether all of these fields are actually necessary.</span></span> <span data-ttu-id="29c39-145">A veces, estas solicitudes son el resultado de una consulta `SELECT *` con un diseño inadecuado.</span><span class="sxs-lookup"><span data-stu-id="29c39-145">Sometimes these requests are the result of poorly designed `SELECT *` query.</span></span> 

- <span data-ttu-id="29c39-146">De forma similar, las solicitudes que recuperan un gran número de entidades pueden ser un indicio de que la aplicación no filtra los datos correctamente.</span><span class="sxs-lookup"><span data-stu-id="29c39-146">Similarly, requests that retrieve a large number of entities may be sign that the application is not filtering data correctly.</span></span> <span data-ttu-id="29c39-147">Compruebe que todas estas entidades sean realmente necesarias.</span><span class="sxs-lookup"><span data-stu-id="29c39-147">Verify that all of these entities are actually needed.</span></span> <span data-ttu-id="29c39-148">Use el filtrado de la base de datos si es posible, por ejemplo, mediante cláusulas `WHERE` de SQL.</span><span class="sxs-lookup"><span data-stu-id="29c39-148">Use database-side filtering if possible, for example, by using `WHERE` clauses in SQL.</span></span> 

- <span data-ttu-id="29c39-149">Descargar el procesamiento en la base de datos no siempre es la mejor opción.</span><span class="sxs-lookup"><span data-stu-id="29c39-149">Offloading processing to the database is not always the best option.</span></span> <span data-ttu-id="29c39-150">Solo puede usar esta estrategia cuando la base de datos se haya diseñado u optimizado para realizar esta acción.</span><span class="sxs-lookup"><span data-stu-id="29c39-150">Only use this strategy when the database is designed or optimized to do so.</span></span> <span data-ttu-id="29c39-151">La mayor parte de sistemas de base de datos están muy optimizados para determinadas funciones, pero no están diseñados para actuar como motores de aplicaciones de uso general.</span><span class="sxs-lookup"><span data-stu-id="29c39-151">Most database systems are highly optimized for certain functions, but are not designed to act as general-purpose application engines.</span></span> <span data-ttu-id="29c39-152">Para más información, consulte el [antipatrón Busy Database][BusyDatabase].</span><span class="sxs-lookup"><span data-stu-id="29c39-152">For more information, see the [Busy Database antipattern][BusyDatabase].</span></span>


## <a name="how-to-detect-the-problem"></a><span data-ttu-id="29c39-153">Procedimiento para detectar el problema</span><span class="sxs-lookup"><span data-stu-id="29c39-153">How to detect the problem</span></span>

<span data-ttu-id="29c39-154">Algunos síntomas del antipatrón Extraneous Fetching son una latencia elevada y un rendimiento bajo.</span><span class="sxs-lookup"><span data-stu-id="29c39-154">Symptoms of extraneous fetching include high latency and low throughput.</span></span> <span data-ttu-id="29c39-155">Si los datos se recuperan de un almacén de datos, también es probable que se dé una mayor contención.</span><span class="sxs-lookup"><span data-stu-id="29c39-155">If the data is retrieved from a data store, increased contention is also probable.</span></span> <span data-ttu-id="29c39-156">Es probable que los usuarios finales informen de tiempos de respuesta prolongados o errores provocados por el agotamiento del tiempo de espera de los servicios. Estos errores podrían devolver errores HTTP 500 (servidor interno) o errores HTTP 503 (servicio no disponible).</span><span class="sxs-lookup"><span data-stu-id="29c39-156">End users are likely to report extended response times or failures caused by services timing out. These failures could return HTTP 500 (Internal Server) errors or HTTP 503 (Service Unavailable) errors.</span></span> <span data-ttu-id="29c39-157">Examine los registros de eventos del servidor web, que probablemente contengan información más detallada sobre las causas y las circunstancias de los errores.</span><span class="sxs-lookup"><span data-stu-id="29c39-157">Examine the event logs for the web server, which are likely to contain more detailed information about the causes and circumstances of the errors.</span></span>

<span data-ttu-id="29c39-158">Los síntomas de este antipatrón y algunos de la telemetría obtenida podrían ser muy similares a los del [antipatrón Monolithic Persistence][MonolithicPersistence].</span><span class="sxs-lookup"><span data-stu-id="29c39-158">The symptoms of this antipattern and some of the telemetry obtained might be very similar to those of the [Monolithic Persistence antipattern][MonolithicPersistence].</span></span> 

<span data-ttu-id="29c39-159">Puede realizar los pasos siguientes para ayudar a identificar la causa:</span><span class="sxs-lookup"><span data-stu-id="29c39-159">You can perform the following steps to help identify the cause:</span></span>

1. <span data-ttu-id="29c39-160">Identifique las cargas de trabajo o transacciones lentas realizando pruebas de carga, supervisando los procesos o con otros métodos de captura de datos de instrumentación.</span><span class="sxs-lookup"><span data-stu-id="29c39-160">Identify slow workloads or transactions by performing load-testing, process monitoring, or other methods of capturing instrumentation data.</span></span>
2. <span data-ttu-id="29c39-161">Observe los patrones de comportamiento mostrados por el sistema.</span><span class="sxs-lookup"><span data-stu-id="29c39-161">Observe any behavioral patterns exhibited by the system.</span></span> <span data-ttu-id="29c39-162">¿Hay límites determinados en cuanto a transacciones por segundo o en el volumen de los usuarios?</span><span class="sxs-lookup"><span data-stu-id="29c39-162">Are there particular limits in terms of transactions per second or volume of users?</span></span>
3. <span data-ttu-id="29c39-163">Ponga en correlación las instancias de las cargas de trabajo de baja velocidad con los patrones de comportamiento.</span><span class="sxs-lookup"><span data-stu-id="29c39-163">Correlate the instances of slow workloads with behavioral patterns.</span></span>
4. <span data-ttu-id="29c39-164">Identifique los almacenes de datos que se van a usar.</span><span class="sxs-lookup"><span data-stu-id="29c39-164">Identify the data stores being used.</span></span> <span data-ttu-id="29c39-165">Para cada origen de datos, ejecute una telemetría de nivel inferior para observar el comportamiento de las operaciones.</span><span class="sxs-lookup"><span data-stu-id="29c39-165">For each data source, run lower level telemetry to observe the behavior of operations.</span></span>
6. <span data-ttu-id="29c39-166">Identifique las consultas de ejecución lenta que hacen referencia a estos orígenes de datos.</span><span class="sxs-lookup"><span data-stu-id="29c39-166">Identify any slow-running queries that reference these data sources.</span></span>
7. <span data-ttu-id="29c39-167">Realice un análisis específico de los recursos de las consultas de ejecución lenta y determine cómo se usan y consumen los datos.</span><span class="sxs-lookup"><span data-stu-id="29c39-167">Perform a resource-specific analysis of the slow-running queries and ascertain how the data is used and consumed.</span></span>

<span data-ttu-id="29c39-168">Busque alguno de estos síntomas:</span><span class="sxs-lookup"><span data-stu-id="29c39-168">Look for any of these symptoms:</span></span>

- <span data-ttu-id="29c39-169">Solicitudes de E/S frecuentes y grandes realizadas en el mismo almacén de datos o de recursos.</span><span class="sxs-lookup"><span data-stu-id="29c39-169">Frequent, large I/O requests made to the same resource or data store.</span></span>
- <span data-ttu-id="29c39-170">Contención en un almacén de datos o un recurso compartido.</span><span class="sxs-lookup"><span data-stu-id="29c39-170">Contention in a shared resource or data store.</span></span>
- <span data-ttu-id="29c39-171">Una operación que reciba con frecuencia grandes volúmenes de datos a través de la red.</span><span class="sxs-lookup"><span data-stu-id="29c39-171">An operation that frequently receives large volumes of data over the network.</span></span>
- <span data-ttu-id="29c39-172">Aplicaciones y servicios que empleen mucho tiempo esperando a que las operaciones de E/S se completen.</span><span class="sxs-lookup"><span data-stu-id="29c39-172">Applications and services spending significant time waiting for I/O to complete.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="29c39-173">Diagnóstico de ejemplo</span><span class="sxs-lookup"><span data-stu-id="29c39-173">Example diagnosis</span></span>    

<span data-ttu-id="29c39-174">En las secciones siguientes se aplican estos pasos a los ejemplos anteriores.</span><span class="sxs-lookup"><span data-stu-id="29c39-174">The following sections apply these steps to the previous examples.</span></span>

### <a name="identify-slow-workloads"></a><span data-ttu-id="29c39-175">Identificación de las cargas de trabajo de baja velocidad</span><span class="sxs-lookup"><span data-stu-id="29c39-175">Identify slow workloads</span></span>

<span data-ttu-id="29c39-176">Este gráfico muestra los resultados de rendimiento de una prueba de carga que simula hasta 400 usuarios que ejecutan simultáneamente el método `GetAllFieldsAsync` mostrado anteriormente.</span><span class="sxs-lookup"><span data-stu-id="29c39-176">This graph shows performance results from a load test that simulated up to 400 concurrent users running the `GetAllFieldsAsync` method shown earlier.</span></span> <span data-ttu-id="29c39-177">El rendimiento disminuye lentamente a medida que aumenta la carga.</span><span class="sxs-lookup"><span data-stu-id="29c39-177">Throughput diminishes slowly as the load increases.</span></span> <span data-ttu-id="29c39-178">El tiempo promedio de respuesta aumenta cuando aumenta la carga de trabajo.</span><span class="sxs-lookup"><span data-stu-id="29c39-178">Average response time goes up as the workload increases.</span></span> 

![Resultados de la prueba de carga para el método GetAllFieldsAsync][Load-Test-Results-Client-Side1]

<span data-ttu-id="29c39-180">Una prueba de carga para la operación `AggregateOnClientAsync` muestra un patrón similar.</span><span class="sxs-lookup"><span data-stu-id="29c39-180">A load test for the `AggregateOnClientAsync` operation shows a similar pattern.</span></span> <span data-ttu-id="29c39-181">El volumen de solicitudes es razonablemente estable.</span><span class="sxs-lookup"><span data-stu-id="29c39-181">The volume of requests is reasonably stable.</span></span> <span data-ttu-id="29c39-182">El tiempo promedio de respuesta aumenta la carga de trabajo, aunque más lentamente que en el gráfico anterior.</span><span class="sxs-lookup"><span data-stu-id="29c39-182">The average response time increases with the workload, although more slowly than the previous graph.</span></span>

![Resultados de la prueba de carga para el método AggregateOnClientAsync][Load-Test-Results-Client-Side2]

### <a name="correlate-slow-workloads-with-behavioral-patterns"></a><span data-ttu-id="29c39-184">Ponga en correlación de las cargas de trabajo de baja velocidad con los patrones de comportamiento</span><span class="sxs-lookup"><span data-stu-id="29c39-184">Correlate slow workloads with behavioral patterns</span></span>

<span data-ttu-id="29c39-185">La correlación entre períodos regulares de uso elevado y una disminución del rendimiento pueden indicar áreas problemáticas.</span><span class="sxs-lookup"><span data-stu-id="29c39-185">Any correlation between regular periods of high usage and slowing performance can indicate areas of concern.</span></span> <span data-ttu-id="29c39-186">Examine detenidamente el perfil de rendimiento de la funcionalidad que parece ejecutarse con lentitud, para determinar si coincide con las pruebas de carga realizadas anteriormente.</span><span class="sxs-lookup"><span data-stu-id="29c39-186">Closely examine the performance profile of functionality that is suspected to be slow running, to determine whether it matches the load testing performed earlier.</span></span>

<span data-ttu-id="29c39-187">Realice la prueba de carga de la misma funcionalidad con cargas de usuario paso a paso, para buscar el punto donde el rendimiento disminuye de forma significativa o se colapsa por completo.</span><span class="sxs-lookup"><span data-stu-id="29c39-187">Load test the same functionality using step-based user loads, to find the point where performance drops significantly or fails completely.</span></span> <span data-ttu-id="29c39-188">Si ese punto cae dentro de los límites de su uso real esperado, examine cómo se implementa la funcionalidad.</span><span class="sxs-lookup"><span data-stu-id="29c39-188">If that point falls within the bounds of your expected real-world usage, examine how the functionality is implemented.</span></span>

<span data-ttu-id="29c39-189">Un funcionamiento lento no es necesariamente un problema, si no ocurre cuando el sistema está bajo presión, en un momento crítico y no afecta negativamente al rendimiento de otras operaciones importantes.</span><span class="sxs-lookup"><span data-stu-id="29c39-189">A slow operation is not necessarily a problem, if it is not being performed when the system is under stress, is not time critical, and does not negatively affect the performance of other important operations.</span></span> <span data-ttu-id="29c39-190">Por ejemplo, generar estadísticas de funcionamiento mensuales podría ser una operación de ejecución prolongada, pero probablemente se pueda realizar como un proceso por lotes y ejecutarse como un trabajo de prioridad baja.</span><span class="sxs-lookup"><span data-stu-id="29c39-190">For example, generating monthly operational statistics might be a long-running operation, but it can probably be performed as a batch process and run as a low priority job.</span></span> <span data-ttu-id="29c39-191">Por otro lado, que los clientes consulten el catálogo de productos es una operación esencial para el negocio.</span><span class="sxs-lookup"><span data-stu-id="29c39-191">On the other hand, customers querying the product catalog is a critical business operation.</span></span> <span data-ttu-id="29c39-192">Céntrese en la telemetría generada por estas operaciones críticas para ver cómo varía el rendimiento durante los períodos de uso elevado.</span><span class="sxs-lookup"><span data-stu-id="29c39-192">Focus on the telemetry generated by these critical operations to see how the performance varies during periods of high usage.</span></span>

### <a name="identify-data-sources-in-slow-workloads"></a><span data-ttu-id="29c39-193">Identificación de los orígenes de datos en las cargas de trabajo de baja velocidad</span><span class="sxs-lookup"><span data-stu-id="29c39-193">Identify data sources in slow workloads</span></span>

<span data-ttu-id="29c39-194">Si sospecha que un servicio está teniendo un rendimiento bajo debido al modo en que recupera los datos, investigue cómo interactúa la aplicación con los repositorios que utiliza.</span><span class="sxs-lookup"><span data-stu-id="29c39-194">If you suspect that a service is performing poorly because of the way it retrieves data, investigate how the application interacts with the repositories it uses.</span></span> <span data-ttu-id="29c39-195">Supervise el sistema real para ver a qué orígenes se tiene acceso durante los períodos de bajo rendimiento.</span><span class="sxs-lookup"><span data-stu-id="29c39-195">Monitor the live system to see which sources are accessed during periods of poor performance.</span></span> 

<span data-ttu-id="29c39-196">Para cada origen de datos, instrumente el sistema para capturar lo siguiente:</span><span class="sxs-lookup"><span data-stu-id="29c39-196">For each data source, instrument the system to capture the following:</span></span>

- <span data-ttu-id="29c39-197">La frecuencia con que se tiene acceso a cada almacén de datos.</span><span class="sxs-lookup"><span data-stu-id="29c39-197">The frequency that each data store is accessed.</span></span>
- <span data-ttu-id="29c39-198">El volumen de datos que entran y salen en el almacén de datos.</span><span class="sxs-lookup"><span data-stu-id="29c39-198">The volume of data entering and exiting the data store.</span></span>
- <span data-ttu-id="29c39-199">La temporalización de estas operaciones, especialmente la latencia de las solicitudes.</span><span class="sxs-lookup"><span data-stu-id="29c39-199">The timing of these operations, especially the latency of requests.</span></span>
- <span data-ttu-id="29c39-200">La naturaleza y la frecuencia de los errores que se producen al tener acceso a cada almacén de datos bajo una carga típica.</span><span class="sxs-lookup"><span data-stu-id="29c39-200">The nature and rate of any errors that occur while accessing each data store under typical load.</span></span>

<span data-ttu-id="29c39-201">Compare esta información con el volumen de datos devueltos por la aplicación al cliente.</span><span class="sxs-lookup"><span data-stu-id="29c39-201">Compare this information against the volume of data being returned by the application to the client.</span></span> <span data-ttu-id="29c39-202">Realice un seguimiento de la relación entre el volumen de datos devueltos por el almacén de datos y el volumen devuelto al cliente.</span><span class="sxs-lookup"><span data-stu-id="29c39-202">Track the ratio of the volume of data returned by the data store against the volume of data returned to the client.</span></span> <span data-ttu-id="29c39-203">Si hay grandes diferencias, investigue para determinar si la aplicación está recuperando datos que no sean necesarios.</span><span class="sxs-lookup"><span data-stu-id="29c39-203">If there is any large disparity, investigate to determine whether the application is fetching data that it doesn't need.</span></span>

<span data-ttu-id="29c39-204">Es posible que pueda capturar estos datos observando el sistema real y siguiendo el ciclo de vida de cada solicitud de usuario, o que pueda modelar una serie de cargas de trabajo sintéticas y ejecutarlas en un sistema de prueba.</span><span class="sxs-lookup"><span data-stu-id="29c39-204">You may be able to capture this data by observing the live system and tracing the lifecycle of each user request, or you can model a series of synthetic workloads and run them against a test system.</span></span>

<span data-ttu-id="29c39-205">Los gráficos siguientes muestran la telemetría capturada mediante [APM de New Relic] [ new-relic] durante una prueba de carga del método `GetAllFieldsAsync`.</span><span class="sxs-lookup"><span data-stu-id="29c39-205">The following graphs show telemetry captured using [New Relic APM][new-relic] during a load test of the `GetAllFieldsAsync` method.</span></span> <span data-ttu-id="29c39-206">Tenga en cuenta la diferencia entre los volúmenes de los datos recibidos de la base de datos y las respuestas HTTP correspondientes.</span><span class="sxs-lookup"><span data-stu-id="29c39-206">Note the difference between the volumes of data received from the database and the corresponding HTTP responses.</span></span>

![Telemetría para el método “GetAllFieldsAsync“][TelemetryAllFields]

<span data-ttu-id="29c39-208">Para cada solicitud, la base de datos devolvió 80 503 bytes, pero la respuesta al cliente solo contenía 19 855 bytes, aproximadamente un 25 % del tamaño de la respuesta de la base de datos.</span><span class="sxs-lookup"><span data-stu-id="29c39-208">For each request, the database returned 80,503 bytes, but the response to the client only contained 19,855 bytes, about 25% of the size of the database response.</span></span> <span data-ttu-id="29c39-209">El tamaño de los datos devueltos al cliente puede variar según el formato.</span><span class="sxs-lookup"><span data-stu-id="29c39-209">The size of the data returned to the client can vary depending on the format.</span></span> <span data-ttu-id="29c39-210">Para esta prueba de carga, el cliente solicitó datos JSON.</span><span class="sxs-lookup"><span data-stu-id="29c39-210">For this load test, the client requested JSON data.</span></span> <span data-ttu-id="29c39-211">La prueba independiente con XML (que no se muestra) tenía un tamaño de respuesta de 35 655 bytes o el 44 % del tamaño de la respuesta de la base de datos.</span><span class="sxs-lookup"><span data-stu-id="29c39-211">Separate testing using XML (not shown) had a response size of 35,655 bytes, or 44% of the size of the database response.</span></span>

<span data-ttu-id="29c39-212">La prueba de carga para el método `AggregateOnClientAsync` muestra resultados más extremos.</span><span class="sxs-lookup"><span data-stu-id="29c39-212">The load test for the `AggregateOnClientAsync` method shows more extreme results.</span></span> <span data-ttu-id="29c39-213">En este caso, cada prueba realizó una consulta que recuperó más de 280 Kb de datos de la base de datos, pero la respuesta JSON fue solo de 14 bytes.</span><span class="sxs-lookup"><span data-stu-id="29c39-213">In this case, each test performed a query that retrieved over 280Kb of data from the database, but the JSON response was a mere 14 bytes.</span></span> <span data-ttu-id="29c39-214">La amplia disparidad se debe a que el método calcula un resultado total a partir de un gran volumen de datos.</span><span class="sxs-lookup"><span data-stu-id="29c39-214">The wide disparity is because the method calculates an aggregated result from a large volume of data.</span></span>

![Telemetría para el método “AggregateOnClientAsync“][TelemetryAggregateOnClient]

### <a name="identify-and-analyze-slow-queries"></a><span data-ttu-id="29c39-216">Identificación y análisis de las consultas de baja velocidad</span><span class="sxs-lookup"><span data-stu-id="29c39-216">Identify and analyze slow queries</span></span>

<span data-ttu-id="29c39-217">Busque las consultas de base de datos que consuman la mayoría de los recursos y tarden más tiempo en ejecutarse.</span><span class="sxs-lookup"><span data-stu-id="29c39-217">Look for database queries that consume the most resources and take the most time to execute.</span></span> <span data-ttu-id="29c39-218">Puede agregar instrumentación para buscar las horas de inicio y finalización de muchas operaciones de base de datos.</span><span class="sxs-lookup"><span data-stu-id="29c39-218">You can add instrumentation to find the start and completion times for many database operations.</span></span> <span data-ttu-id="29c39-219">Muchos de los almacenes de datos también proporcionan información detallada sobre cómo se realizan y optimizan las consultas.</span><span class="sxs-lookup"><span data-stu-id="29c39-219">Many data stores also provide in-depth information on how queries are performed and optimized.</span></span> <span data-ttu-id="29c39-220">Por ejemplo, el panel Rendimiento de las consultas en el portal de administración de Azure SQL Database permite seleccionar una consulta y ver la información detallada del rendimiento en tiempo de ejecución.</span><span class="sxs-lookup"><span data-stu-id="29c39-220">For example, the Query Performance pane in the Azure SQL Database management portal lets you select a query and view detailed runtime performance information.</span></span> <span data-ttu-id="29c39-221">Esta es la consulta generada por la operación `GetAllFieldsAsync`:</span><span class="sxs-lookup"><span data-stu-id="29c39-221">Here is the query generated by the `GetAllFieldsAsync` operation:</span></span>

![El panel Detalles de la consulta del portal de administración de Windows Azure SQL Database][QueryDetails]


## <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="29c39-223">Implementación de la solución y comprobación del resultado</span><span class="sxs-lookup"><span data-stu-id="29c39-223">Implement the solution and verify the result</span></span>

<span data-ttu-id="29c39-224">Después de cambiar el método `GetRequiredFieldsAsync` para utilizar una instrucción SELECT en la base de datos, las pruebas de carga mostraban los resultados siguientes.</span><span class="sxs-lookup"><span data-stu-id="29c39-224">After changing the `GetRequiredFieldsAsync` method to use a SELECT statement on the database side, load testing showed the following results.</span></span>

![Resultados de las pruebas de carga para el método GetRequiredFieldsAsyn][Load-Test-Results-Database-Side1]

<span data-ttu-id="29c39-226">Esta prueba de carga usaba la misma implementación y la misma carga de trabajo simulada de 400 usuarios simultáneos anterior.</span><span class="sxs-lookup"><span data-stu-id="29c39-226">This load test used the same deployment and the same simulated workload of 400 concurrent users as before.</span></span> <span data-ttu-id="29c39-227">El gráfico muestra una latencia mucho menor.</span><span class="sxs-lookup"><span data-stu-id="29c39-227">The graph shows much lower latency.</span></span> <span data-ttu-id="29c39-228">El tiempo de respuesta aumenta con la carga a aproximadamente 1,3 segundos, en comparación con los 4 segundos del caso anterior.</span><span class="sxs-lookup"><span data-stu-id="29c39-228">Response time rises with load to approximately 1.3 seconds, compared to 4 seconds in the previous case.</span></span> <span data-ttu-id="29c39-229">El rendimiento también es superior a 350 solicitudes por segundo, en comparación con los de 100 de antes.</span><span class="sxs-lookup"><span data-stu-id="29c39-229">The throughput is also higher at 350 requests per second compared to 100 earlier.</span></span> <span data-ttu-id="29c39-230">El volumen de datos recuperados de la base de datos ahora coincide más con el tamaño de los mensajes de respuesta HTTP.</span><span class="sxs-lookup"><span data-stu-id="29c39-230">The volume of data retrieved from the database now closely matches the size of the HTTP response messages.</span></span>

![Telemetría para el método “GetRequiredFieldsAsync“][TelemetryRequiredFields]

<span data-ttu-id="29c39-232">Las pruebas de carga mediante el método `AggregateOnDatabaseAsync` generan los siguientes resultados:</span><span class="sxs-lookup"><span data-stu-id="29c39-232">Load testing using the `AggregateOnDatabaseAsync` method generates the following results:</span></span>

![Resultados de la prueba de carga para el método AggregateOnDatabaseAsync][Load-Test-Results-Database-Side2]

<span data-ttu-id="29c39-234">El tiempo de respuesta promedio ahora es mínimo.</span><span class="sxs-lookup"><span data-stu-id="29c39-234">The average response time is now minimal.</span></span> <span data-ttu-id="29c39-235">Se trata de una mejora del orden de magnitud en el rendimiento, provocada principalmente por la gran reducción de las operaciones de E/S desde la base de datos.</span><span class="sxs-lookup"><span data-stu-id="29c39-235">This is an order of magnitude improvement in performance, caused primarily by the large reduction in I/O from the database.</span></span> 

<span data-ttu-id="29c39-236">Aquí está la telemetría correspondiente para el método `AggregateOnDatabaseAsync`.</span><span class="sxs-lookup"><span data-stu-id="29c39-236">Here is the corresponding telemetry for the `AggregateOnDatabaseAsync` method.</span></span> <span data-ttu-id="29c39-237">La cantidad de datos recuperados de la base de datos se ha reducido considerablemente desde más de 280 Kb por transacción a 53 bytes.</span><span class="sxs-lookup"><span data-stu-id="29c39-237">The amount of data retrieved from the database was vastly reduced, from over 280Kb per transaction to 53 bytes.</span></span> <span data-ttu-id="29c39-238">Como resultado, la cantidad máxima sostenida de solicitudes por minuto se ha ampliado de alrededor de 2 000 a más de 25 000.</span><span class="sxs-lookup"><span data-stu-id="29c39-238">As a result, the maximum sustained number of requests per minute was raised from around 2,000 to over 25,000.</span></span>

![Telemetría para el método “AggregateOnDatabaseAsync“][TelemetryAggregateInDatabaseAsync]


## <a name="related-resources"></a><span data-ttu-id="29c39-240">Recursos relacionados</span><span class="sxs-lookup"><span data-stu-id="29c39-240">Related resources</span></span>

- <span data-ttu-id="29c39-241">[Antipatrón Busy Database][BusyDatabase]</span><span class="sxs-lookup"><span data-stu-id="29c39-241">[Busy Database antipattern][BusyDatabase]</span></span>
- <span data-ttu-id="29c39-242">[Antipatrón Chatty I/O][chatty-io]</span><span class="sxs-lookup"><span data-stu-id="29c39-242">[Chatty I/O antipattern][chatty-io]</span></span>
- <span data-ttu-id="29c39-243">[Procedimientos recomendados para las particiones de datos][data-partitioning]</span><span class="sxs-lookup"><span data-stu-id="29c39-243">[Data partitioning best practices][data-partitioning]</span></span>


[BusyDatabase]: ../busy-database/index.md
[data-partitioning]: ../../best-practices/data-partitioning.md
[new-relic]: https://newrelic.com/application-monitoring

[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/ExtraneousFetching

[chatty-io]: ../chatty-io/index.md
[MonolithicPersistence]: ../monolithic-persistence/index.md
[Load-Test-Results-Client-Side1]:_images/LoadTestResultsClientSide1.jpg
[Load-Test-Results-Client-Side2]:_images/LoadTestResultsClientSide2.jpg
[Load-Test-Results-Database-Side1]:_images/LoadTestResultsDatabaseSide1.jpg
[Load-Test-Results-Database-Side2]:_images/LoadTestResultsDatabaseSide2.jpg
[QueryPerformanceZoomed]: _images/QueryPerformanceZoomed.jpg
[QueryDetails]: _images/QueryDetails.jpg
[TelemetryAllFields]: _images/TelemetryAllFields.jpg
[TelemetryAggregateOnClient]: _images/TelemetryAggregateOnClient.jpg
[TelemetryRequiredFields]: _images/TelemetryRequiredFields.jpg
[TelemetryAggregateInDatabaseAsync]: _images/TelemetryAggregateInDatabase.jpg
