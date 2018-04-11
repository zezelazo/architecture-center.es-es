---
title: Antipatrón Chatty I/O
description: Un gran número de solicitudes de E/S puede afectar al rendimiento y la capacidad de respuesta.
author: dragon119
ms.openlocfilehash: 4f0e0e455ceb58317d3029d8ab4631d476802499
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/23/2018
---
# <a name="chatty-io-antipattern"></a><span data-ttu-id="5beb1-103">Antipatrón Chatty I/O</span><span class="sxs-lookup"><span data-stu-id="5beb1-103">Chatty I/O antipattern</span></span>

<span data-ttu-id="5beb1-104">El efecto acumulado de un gran número de solicitudes de E/S puede afectar notablemente al rendimiento y la capacidad de respuesta.</span><span class="sxs-lookup"><span data-stu-id="5beb1-104">The cumulative effect of a large number of I/O requests can have a significant impact on performance and responsiveness.</span></span>

## <a name="problem-description"></a><span data-ttu-id="5beb1-105">Descripción del problema</span><span class="sxs-lookup"><span data-stu-id="5beb1-105">Problem description</span></span>

<span data-ttu-id="5beb1-106">Las llamadas de red y otras operaciones de E/S son inherentemente lentas en comparación con las tareas de proceso.</span><span class="sxs-lookup"><span data-stu-id="5beb1-106">Network calls and other I/O operations are inherently slow compared to compute tasks.</span></span> <span data-ttu-id="5beb1-107">Cada solicitud de E/S normalmente tiene una sobrecarga importante y el efecto acumulado de numerosas operaciones de E/S puede ralentizar el sistema.</span><span class="sxs-lookup"><span data-stu-id="5beb1-107">Each I/O request typically has significant overhead, and the cumulative effect of numerous I/O operations can slow down the system.</span></span> <span data-ttu-id="5beb1-108">Estas son algunas causas comunes de Chatty I/O.</span><span class="sxs-lookup"><span data-stu-id="5beb1-108">Here are some common causes of chatty I/O.</span></span>

### <a name="reading-and-writing-individual-records-to-a-database-as-distinct-requests"></a><span data-ttu-id="5beb1-109">Lectura y escritura de registros individuales en una base de datos como solicitudes distintas</span><span class="sxs-lookup"><span data-stu-id="5beb1-109">Reading and writing individual records to a database as distinct requests</span></span>

<span data-ttu-id="5beb1-110">En el ejemplo siguiente se lee de una base de datos de productos.</span><span class="sxs-lookup"><span data-stu-id="5beb1-110">The following example reads from a database of products.</span></span> <span data-ttu-id="5beb1-111">Hay tres tablas, `Product`, `ProductSubcategory` y `ProductPriceListHistory`.</span><span class="sxs-lookup"><span data-stu-id="5beb1-111">There are three tables, `Product`, `ProductSubcategory`, and `ProductPriceListHistory`.</span></span> <span data-ttu-id="5beb1-112">El código recupera todos los productos en una subcategoría, junto con la información sobre los precios, mediante la ejecución de una serie de consultas:</span><span class="sxs-lookup"><span data-stu-id="5beb1-112">The code retrieves all of the products in a subcategory, along with the pricing information, by executing a series of queries:</span></span>  

1. <span data-ttu-id="5beb1-113">Consulte la subcategoría en la tabla `ProductSubcategory`.</span><span class="sxs-lookup"><span data-stu-id="5beb1-113">Query the subcategory from the `ProductSubcategory` table.</span></span>
2. <span data-ttu-id="5beb1-114">Consulte la tabla `Product` para buscar todos los productos de esa subcategoría.</span><span class="sxs-lookup"><span data-stu-id="5beb1-114">Find all products in that subcategory by querying the `Product` table.</span></span>
3. <span data-ttu-id="5beb1-115">Para cada producto, consulte los datos de precio en la tabla `ProductPriceListHistory`.</span><span class="sxs-lookup"><span data-stu-id="5beb1-115">For each product, query the pricing data from the `ProductPriceListHistory` table.</span></span>

<span data-ttu-id="5beb1-116">La aplicación usa [Entity Framework][ef] para consultar la base de datos.</span><span class="sxs-lookup"><span data-stu-id="5beb1-116">The application uses [Entity Framework][ef] to query the database.</span></span> <span data-ttu-id="5beb1-117">Puede encontrar el ejemplo completo [aquí][code-sample].</span><span class="sxs-lookup"><span data-stu-id="5beb1-117">You can find the complete sample [here][code-sample].</span></span> 

```csharp
public async Task<IHttpActionResult> GetProductsInSubCategoryAsync(int subcategoryId)
{
    using (var context = GetContext())
    {
        // Get product subcategory.
        var productSubcategory = await context.ProductSubcategories
                .Where(psc => psc.ProductSubcategoryId == subcategoryId)
                .FirstOrDefaultAsync();

        // Find products in that category.
        productSubcategory.Product = await context.Products
            .Where(p => subcategoryId == p.ProductSubcategoryId)
            .ToListAsync();

        // Find price history for each product.
        foreach (var prod in productSubcategory.Product)
        {
            int productId = prod.ProductId;
            var productListPriceHistory = await context.ProductListPriceHistory
                .Where(pl => pl.ProductId == productId)
                .ToListAsync();
            prod.ProductListPriceHistory = productListPriceHistory;
        }
        return Ok(productSubcategory);
    }
}
```

<span data-ttu-id="5beb1-118">En este ejemplo se muestra el problema de forma explícita, pero, en ocasiones, si captura implícitamente los registros secundarios uno a uno, el asignador relacional de objetos puede enmascarar el problema.</span><span class="sxs-lookup"><span data-stu-id="5beb1-118">This example shows the problem explicitly, but sometimes an O/RM can mask the problem, if it implicitly fetches child records one at a time.</span></span> <span data-ttu-id="5beb1-119">Esto se conoce como el "problema N+1".</span><span class="sxs-lookup"><span data-stu-id="5beb1-119">This is known as the "N+1 problem".</span></span> 

### <a name="implementing-a-single-logical-operation-as-a-series-of-http-requests"></a><span data-ttu-id="5beb1-120">Implementación de una única operación lógica como serie de solicitudes HTTP</span><span class="sxs-lookup"><span data-stu-id="5beb1-120">Implementing a single logical operation as a series of HTTP requests</span></span>

<span data-ttu-id="5beb1-121">Esto suele ocurrir cuando los desarrolladores intentan seguir un paradigma basado en los objetos y tratan los objetos remotos como si fueran objetos locales en la memoria.</span><span class="sxs-lookup"><span data-stu-id="5beb1-121">This often happens when developers try to follow an object-oriented paradigm, and treat remote objects as if they were local objects in memory.</span></span> <span data-ttu-id="5beb1-122">Esto puede producir demasiado recorrido de ida y vuelta en la red.</span><span class="sxs-lookup"><span data-stu-id="5beb1-122">This can result in too many network round trips.</span></span> <span data-ttu-id="5beb1-123">Por ejemplo, la siguiente API de web expone las propiedades individuales de los objetos `User` a través de métodos HTTP GET individuales.</span><span class="sxs-lookup"><span data-stu-id="5beb1-123">For example, the following web API exposes the individual properties of `User` objects through individual HTTP GET methods.</span></span> 

```csharp
public class UserController : ApiController
{
    [HttpGet]
    [Route("users/{id:int}/username")]
    public HttpResponseMessage GetUserName(int id)
    {
        ...
    }

    [HttpGet]
    [Route("users/{id:int}/gender")]
    public HttpResponseMessage GetGender(int id)
    {
        ...
    }

    [HttpGet]
    [Route("users/{id:int}/dateofbirth")]
    public HttpResponseMessage GetDateOfBirth(int id)
    {
        ...
    }
}
```

<span data-ttu-id="5beb1-124">Aunque técnicamente no hay ningún problema con este enfoque, la mayoría de los clientes probablemente tendrá que obtener varias propiedades para cada `User`, lo cual dará lugar a código de cliente similar al siguiente.</span><span class="sxs-lookup"><span data-stu-id="5beb1-124">While there's nothing technically wrong with this approach, most clients will probably need to get several properties for each `User`, resulting in client code like the following.</span></span> 

```csharp
HttpResponseMessage response = await client.GetAsync("users/1/username");
response.EnsureSuccessStatusCode();
var userName = await response.Content.ReadAsStringAsync();

response = await client.GetAsync("users/1/gender");
response.EnsureSuccessStatusCode();
var gender = await response.Content.ReadAsStringAsync();

response = await client.GetAsync("users/1/dateofbirth");
response.EnsureSuccessStatusCode();
var dob = await response.Content.ReadAsStringAsync();
```

### <a name="reading-and-writing-to-a-file-on-disk"></a><span data-ttu-id="5beb1-125">Lectura y escritura en un archivo de disco</span><span class="sxs-lookup"><span data-stu-id="5beb1-125">Reading and writing to a file on disk</span></span>

<span data-ttu-id="5beb1-126">Las operaciones de E/S de archivos implican la apertura de un archivo y su desplazamiento al punto adecuado para leer o escribir datos.</span><span class="sxs-lookup"><span data-stu-id="5beb1-126">File I/O involves opening a file and moving to the appropriate point before reading or writing data.</span></span> <span data-ttu-id="5beb1-127">Una vez completada la operación, el archivo podría cerrarse para ahorrar recursos del sistema operativo.</span><span class="sxs-lookup"><span data-stu-id="5beb1-127">When the operation is complete, the file might be closed to save operating system resources.</span></span> <span data-ttu-id="5beb1-128">Una aplicación que lee y escribe constantemente pequeñas cantidades de información en un archivo generará una sobrecarga de E/S considerable.</span><span class="sxs-lookup"><span data-stu-id="5beb1-128">An application that continually reads and writes small amounts of information to a file will generate significant I/O overhead.</span></span> <span data-ttu-id="5beb1-129">Las solicitudes de escritura pequeñas también pueden provocar la fragmentación del archivo, lo cual mermará aún más las operaciones de E/S posteriores.</span><span class="sxs-lookup"><span data-stu-id="5beb1-129">Small write requests can also lead to file fragmentation, slowing subsequent I/O operations still further.</span></span> 

<span data-ttu-id="5beb1-130">En el siguiente ejemplo se usa un objeto `FileStream` para escribir un objeto `Customer` en un archivo.</span><span class="sxs-lookup"><span data-stu-id="5beb1-130">The following example uses a `FileStream` to write a `Customer` object to a file.</span></span> <span data-ttu-id="5beb1-131">Al crear `FileStream`, el archivo se abre y al eliminarlo, se cierra.</span><span class="sxs-lookup"><span data-stu-id="5beb1-131">Creating the `FileStream` opens the file, and disposing it closes the file.</span></span> <span data-ttu-id="5beb1-132">(La instrucción `using` elimina automáticamente el objeto `FileStream`). Si la aplicación llama a este método varias veces cuando se agregan nuevos clientes, la sobrecarga de E/S puede acumularse rápidamente.</span><span class="sxs-lookup"><span data-stu-id="5beb1-132">(The `using` statement automatically disposes the `FileStream` object.) If the application calls this method repeatedly as new customers are added, the I/O overhead can accumulate quickly.</span></span>

```csharp
private async Task SaveCustomerToFileAsync(Customer cust)
{
    using (Stream fileStream = new FileStream(CustomersFileName, FileMode.Append))
    {
        BinaryFormatter formatter = new BinaryFormatter();
        byte [] data = null;
        using (MemoryStream memStream = new MemoryStream())
        {
            formatter.Serialize(memStream, cust);
            data = memStream.ToArray();
        }
        await fileStream.WriteAsync(data, 0, data.Length);
    }
}
```

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="5beb1-133">Procedimiento para corregir el problema</span><span class="sxs-lookup"><span data-stu-id="5beb1-133">How to fix the problem</span></span>

<span data-ttu-id="5beb1-134">Reduzca el número de solicitudes de E/S al empaquetar los datos en menos solicitudes, pero mayores.</span><span class="sxs-lookup"><span data-stu-id="5beb1-134">Reduce the number of I/O requests by packaging the data into larger, fewer requests.</span></span>

<span data-ttu-id="5beb1-135">Capture los datos de una base de datos como una sola consulta, en lugar de como varias consultas menores.</span><span class="sxs-lookup"><span data-stu-id="5beb1-135">Fetch data from a database as a single query, instead of several smaller queries.</span></span> <span data-ttu-id="5beb1-136">Esta es una versión revisada del código que recupera información del producto.</span><span class="sxs-lookup"><span data-stu-id="5beb1-136">Here's a revised version of the code that retrieves product information.</span></span>

```csharp
public async Task<IHttpActionResult> GetProductCategoryDetailsAsync(int subCategoryId)
{
    using (var context = GetContext())
    {
        var subCategory = await context.ProductSubcategories
                .Where(psc => psc.ProductSubcategoryId == subCategoryId)
                .Include("Product.ProductListPriceHistory")
                .FirstOrDefaultAsync();

        if (subCategory == null)
            return NotFound();

        return Ok(subCategory);
    }
}
```

<span data-ttu-id="5beb1-137">Siga los principios de diseño de REST para las API web.</span><span class="sxs-lookup"><span data-stu-id="5beb1-137">Follow REST design principles for web APIs.</span></span> <span data-ttu-id="5beb1-138">Esta es una versión revisada de la API web del ejemplo anterior.</span><span class="sxs-lookup"><span data-stu-id="5beb1-138">Here's a revised version of the web API from the earlier example.</span></span> <span data-ttu-id="5beb1-139">En lugar de métodos GET independientes para cada propiedad, hay un único método GET que devuelve el objeto `User`.</span><span class="sxs-lookup"><span data-stu-id="5beb1-139">Instead of separate GET methods for each property, there is a single GET method that returns the `User`.</span></span> <span data-ttu-id="5beb1-140">Esto da como resultado un cuerpo de respuesta mayor por solicitud, pero es probable que los clientes realicen menos llamadas API.</span><span class="sxs-lookup"><span data-stu-id="5beb1-140">This results in a larger response body per request, but each client is likely to make fewer API calls.</span></span>

```csharp
public class UserController : ApiController
{
    [HttpGet]
    [Route("users/{id:int}")]
    public HttpResponseMessage GetUser(int id)
    {
        ...
    }
}

// Client code
HttpResponseMessage response = await client.GetAsync("users/1");
response.EnsureSuccessStatusCode();
var user = await response.Content.ReadAsStringAsync();
```

<span data-ttu-id="5beb1-141">Para las operaciones de E/S de archivos, considere la posibilidad de almacenar en búfer los datos en memoria y de escribirlos en un archivo como una única operación.</span><span class="sxs-lookup"><span data-stu-id="5beb1-141">For file I/O, consider buffering data in memory and then writing the buffered data to a file as a single operation.</span></span> <span data-ttu-id="5beb1-142">Este enfoque reduce la sobrecarga de al abrir y cerrar el archivo repetidamente y ayuda a reducir la fragmentación del archivo en el disco.</span><span class="sxs-lookup"><span data-stu-id="5beb1-142">This approach reduces the overhead from repeatedly opening and closing the file, and helps to reduce fragmentation of the file on disk.</span></span>

```csharp
// Save a list of customer objects to a file
private async Task SaveCustomerListToFileAsync(List<Customer> customers)
{
    using (Stream fileStream = new FileStream(CustomersFileName, FileMode.Append))
    {
        BinaryFormatter formatter = new BinaryFormatter();
        foreach (var cust in customers)
        {
            byte[] data = null;
            using (MemoryStream memStream = new MemoryStream())
            {
                formatter.Serialize(memStream, cust);
                data = memStream.ToArray();
            }
            await fileStream.WriteAsync(data, 0, data.Length);
        }
    }
}

// In-memory buffer for customers.
List<Customer> customers = new List<Customers>();

// Create a new customer and add it to the buffer
var cust = new Customer(...);
customers.Add(cust);

// Add more customers to the list as they are created
...

// Save the contents of the list, writing all customers in a single operation
await SaveCustomerListToFileAsync(customers);
```

## <a name="considerations"></a><span data-ttu-id="5beb1-143">Consideraciones</span><span class="sxs-lookup"><span data-stu-id="5beb1-143">Considerations</span></span>

- <span data-ttu-id="5beb1-144">En los dos primeros ejemplos se realizan *menos* llamadas de E/S, pero cada una de ellas recupera *más* información.</span><span class="sxs-lookup"><span data-stu-id="5beb1-144">The first two examples make *fewer* I/O calls, but each one retrieves *more* information.</span></span> <span data-ttu-id="5beb1-145">Debe tener en cuenta el equilibrio entre estos dos factores.</span><span class="sxs-lookup"><span data-stu-id="5beb1-145">You must consider the tradeoff between these two factors.</span></span> <span data-ttu-id="5beb1-146">La respuesta correcta dependerá de los patrones de uso reales.</span><span class="sxs-lookup"><span data-stu-id="5beb1-146">The right answer will depend on the actual usage patterns.</span></span> <span data-ttu-id="5beb1-147">Por ejemplo, en el ejemplo de API web, puede que los clientes a menudo solo necesiten el nombre de usuario.</span><span class="sxs-lookup"><span data-stu-id="5beb1-147">For example, in the web API example, it might turn out that clients often need just the user name.</span></span> <span data-ttu-id="5beb1-148">En ese caso, tendría sentido exponerlo como una llamada API independiente.</span><span class="sxs-lookup"><span data-stu-id="5beb1-148">In that case, it might make sense to expose it as a separate API call.</span></span> <span data-ttu-id="5beb1-149">Para más información, consulte el [antipatrón Extraneous Fetching][extraneous-fetching].</span><span class="sxs-lookup"><span data-stu-id="5beb1-149">For more information, see the [Extraneous Fetching][extraneous-fetching] antipattern.</span></span>

- <span data-ttu-id="5beb1-150">Al leer datos, no realice solicitudes de E/S demasiado grandes.</span><span class="sxs-lookup"><span data-stu-id="5beb1-150">When reading data, do not make your I/O requests too large.</span></span> <span data-ttu-id="5beb1-151">Las aplicaciones solo deben recuperar la información que sea probable que usen.</span><span class="sxs-lookup"><span data-stu-id="5beb1-151">An application should only retrieve the information that it is likely to use.</span></span> 

- <span data-ttu-id="5beb1-152">En ocasiones resulta útil para la partición de la información de un objeto en dos fragmentos, los *datos de acceso frecuente* en la mayoría de las solicitudes y los *datos de acceso menos frecuente* que casi no se usan.</span><span class="sxs-lookup"><span data-stu-id="5beb1-152">Sometimes it helps to partition the information for an object into two chunks, *frequently accessed data* that accounts for most requests, and *less frequently accessed data* that is used rarely.</span></span> <span data-ttu-id="5beb1-153">A menudo, los datos a los que se accede con más frecuencia son una cantidad relativamente pequeña del total de datos de un objeto, por lo que devolver solo esa parte puede prevenir notablemente la sobrecarga de E/S.</span><span class="sxs-lookup"><span data-stu-id="5beb1-153">Often the most frequently accessed data is a relatively small portion of the total data for an object, so returning just that portion can save significant I/O overhead.</span></span>

- <span data-ttu-id="5beb1-154">Al escribir los datos, evite bloquear los recursos durante más tiempo del necesario para reducir las posibilidades de contención durante las operaciones largas.</span><span class="sxs-lookup"><span data-stu-id="5beb1-154">When writing data, avoid locking resources for longer than necessary, to reduce the chances of contention during a lengthy operation.</span></span> <span data-ttu-id="5beb1-155">Si una operación de escritura abarca varios almacenes de datos, archivos o servicios, adopte un enfoque coherente.</span><span class="sxs-lookup"><span data-stu-id="5beb1-155">If a write operation spans multiple data stores, files, or services, then adopt an eventually consistent approach.</span></span> <span data-ttu-id="5beb1-156">Consulte la [guía de coherencia de los datos][data-consistency-guidance].</span><span class="sxs-lookup"><span data-stu-id="5beb1-156">See [Data Consistency guidance][data-consistency-guidance].</span></span>

- <span data-ttu-id="5beb1-157">Si almacena los datos en búfer en memoria antes de escribir, estos son vulnerables si se bloquea el proceso.</span><span class="sxs-lookup"><span data-stu-id="5beb1-157">If you buffer data in memory before writing it, the data is vulnerable if the process crashes.</span></span> <span data-ttu-id="5beb1-158">Si la velocidad de los datos normalmente tiene ráfagas o es relativamente dispersa, puede ser más seguro para almacenarlos en búfer en una cola durable externa como [Event Hubs](http://azure.microsoft.com/services/event-hubs/).</span><span class="sxs-lookup"><span data-stu-id="5beb1-158">If the data rate typically has bursts or is relatively sparse, it may be safer to buffer the data in an external durable queue such as [Event Hubs](http://azure.microsoft.com/services/event-hubs/).</span></span>

- <span data-ttu-id="5beb1-159">Considere la posibilidad de almacenar en caché los datos que recupere de un servicio o una base de datos.</span><span class="sxs-lookup"><span data-stu-id="5beb1-159">Consider caching data that you retrieve from a service or a database.</span></span> <span data-ttu-id="5beb1-160">Esto puede ayudar a reducir el volumen de E/S, al evitar repetir las solicitudes de los mismos datos.</span><span class="sxs-lookup"><span data-stu-id="5beb1-160">This can help to reduce the volume of I/O by avoiding repeated requests for the same data.</span></span> <span data-ttu-id="5beb1-161">Para más información, consulte los [procedimientos recomendados para el almacenamiento en caché][caching-guidance].</span><span class="sxs-lookup"><span data-stu-id="5beb1-161">For more information, see [Caching best practices][caching-guidance].</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="5beb1-162">Procedimiento para detectar el problema</span><span class="sxs-lookup"><span data-stu-id="5beb1-162">How to detect the problem</span></span>

<span data-ttu-id="5beb1-163">Algunos síntomas de Chatty I/O son una latencia elevada y un rendimiento bajo.</span><span class="sxs-lookup"><span data-stu-id="5beb1-163">Symptoms of chatty I/O include high latency and low throughput.</span></span> <span data-ttu-id="5beb1-164">Es probable que los usuarios finales informen de tiempos de respuesta prolongados o errores provocados por el agotamiento del tiempo de espera de los servicios, debido al aumento de contención de los recursos de E/S.</span><span class="sxs-lookup"><span data-stu-id="5beb1-164">End users are likely to report extended response times or failures caused by services timing out, due to increased contention for I/O resources.</span></span>

<span data-ttu-id="5beb1-165">Puede realizar los pasos siguientes para ayudar a identificar la causa de cualquier problema:</span><span class="sxs-lookup"><span data-stu-id="5beb1-165">You can perform the following steps to help identify the causes of any problems:</span></span>

1. <span data-ttu-id="5beb1-166">Supervise los procesos del sistema de producción para identificar las operaciones con tiempos de respuesta prolongados.</span><span class="sxs-lookup"><span data-stu-id="5beb1-166">Perform process monitoring of the production system to identify operations with poor response times.</span></span>
2. <span data-ttu-id="5beb1-167">Realice pruebas de carga de cada operación identificada en el paso anterior.</span><span class="sxs-lookup"><span data-stu-id="5beb1-167">Perform load testing of each operation identified in the previous step.</span></span>
3. <span data-ttu-id="5beb1-168">Durante las pruebas de carga, recopile datos de telemetría acerca de las solicitudes de acceso de datos realizadas por cada operación.</span><span class="sxs-lookup"><span data-stu-id="5beb1-168">During the load tests, gather telemetry data about the data access requests made by each operation.</span></span>
4. <span data-ttu-id="5beb1-169">Recopile estadísticas detalladas de cada solicitud enviada a un almacén de datos.</span><span class="sxs-lookup"><span data-stu-id="5beb1-169">Gather detailed statistics for each request sent to a data store.</span></span>
5. <span data-ttu-id="5beb1-170">Genere perfiles de aplicación en el entorno de prueba para establecer dónde se pueden estar produciendo cuellos de botella de E/S.</span><span class="sxs-lookup"><span data-stu-id="5beb1-170">Profile the application in the test environment to establish where possible I/O bottlenecks might be occurring.</span></span> 

<span data-ttu-id="5beb1-171">Busque alguno de estos síntomas:</span><span class="sxs-lookup"><span data-stu-id="5beb1-171">Look for any of these symptoms:</span></span>

- <span data-ttu-id="5beb1-172">Un gran número de solicitudes de E/S pequeñas en el mismo archivo.</span><span class="sxs-lookup"><span data-stu-id="5beb1-172">A large number of small I/O requests made to the same file.</span></span>
- <span data-ttu-id="5beb1-173">Un gran número de solicitudes de red pequeñas realizadas por una instancia de la aplicación al mismo servicio.</span><span class="sxs-lookup"><span data-stu-id="5beb1-173">A large number of small network requests made by an application instance to the same service.</span></span>
- <span data-ttu-id="5beb1-174">Un gran número de solicitudes pequeñas realizadas por una instancia de la aplicación al mismo almacén de datos.</span><span class="sxs-lookup"><span data-stu-id="5beb1-174">A large number of small requests made by an application instance to the same data store.</span></span>
- <span data-ttu-id="5beb1-175">Aplicaciones y servicios dependientes de las operaciones de E/S.</span><span class="sxs-lookup"><span data-stu-id="5beb1-175">Applications and services becoming I/O bound.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="5beb1-176">Diagnóstico de ejemplo</span><span class="sxs-lookup"><span data-stu-id="5beb1-176">Example diagnosis</span></span>

<span data-ttu-id="5beb1-177">En las siguientes secciones se aplican estos pasos al ejemplo anterior donde se consulta una base de datos.</span><span class="sxs-lookup"><span data-stu-id="5beb1-177">The following sections apply these steps to the example shown earlier that queries a database.</span></span>

### <a name="load-test-the-application"></a><span data-ttu-id="5beb1-178">Prueba de carga de la aplicación</span><span class="sxs-lookup"><span data-stu-id="5beb1-178">Load test the application</span></span>

<span data-ttu-id="5beb1-179">En este gráfico se muestran los resultados de las pruebas de carga.</span><span class="sxs-lookup"><span data-stu-id="5beb1-179">This graph shows the results of load testing.</span></span> <span data-ttu-id="5beb1-180">La mediana de tiempo de respuesta es de 10 segundos por solicitud.</span><span class="sxs-lookup"><span data-stu-id="5beb1-180">Median response time is measured in 10s of seconds per request.</span></span> <span data-ttu-id="5beb1-181">En el gráfico se muestra una latencia muy alta.</span><span class="sxs-lookup"><span data-stu-id="5beb1-181">The graph shows very high latency.</span></span> <span data-ttu-id="5beb1-182">Con una carga de 1000 usuarios, uno de ellos podría tener que esperar casi un minuto ver los resultados de una consulta.</span><span class="sxs-lookup"><span data-stu-id="5beb1-182">With a load of 1000 users, a user might have to wait for nearly a minute to see the results of a query.</span></span> 

![Resultados de la prueba de carga de los indicadores clave para la aplicación de ejemplo de Chatty I/O][key-indicators-chatty-io]

> [!NOTE]
> <span data-ttu-id="5beb1-184">La aplicación se implementó como aplicación web de Azure App Service, con Azure SQL Database.</span><span class="sxs-lookup"><span data-stu-id="5beb1-184">The application was deployed as an Azure App Service web app, using Azure SQL Database.</span></span> <span data-ttu-id="5beb1-185">Para la prueba de carga se utilizó una carga de trabajo de pasos simulada de hasta 1000 usuarios simultáneos.</span><span class="sxs-lookup"><span data-stu-id="5beb1-185">The load test used a simulated step workload of up to 1000 concurrent users.</span></span> <span data-ttu-id="5beb1-186">La base de datos se configuró con un grupo de conexiones que admitía hasta 1000 conexiones simultáneas, para reducir la posibilidad de que la contención para las conexiones afectara a los resultados.</span><span class="sxs-lookup"><span data-stu-id="5beb1-186">The database was configured with a connection pool supporting up to 1000 concurrent connections, to reduce the chance that contention for connections would affect the results.</span></span> 

### <a name="monitor-the-application"></a><span data-ttu-id="5beb1-187">Supervisión de la aplicación</span><span class="sxs-lookup"><span data-stu-id="5beb1-187">Monitor the application</span></span>

<span data-ttu-id="5beb1-188">Puede usar un paquete de supervisión del rendimiento de la aplicación (APM) para capturar y analizar las métricas clave que pueden identificar Chatty I/O.</span><span class="sxs-lookup"><span data-stu-id="5beb1-188">You can use an application performance monitoring (APM) package to capture and analyze the key metrics that might identify chatty I/O.</span></span> <span data-ttu-id="5beb1-189">La importancia de las distintas métricas dependerá de la carga de trabajo de E/S.</span><span class="sxs-lookup"><span data-stu-id="5beb1-189">Which metrics are important will depend on the I/O workload.</span></span> <span data-ttu-id="5beb1-190">En este ejemplo, las solicitudes de E/S interesantes eran las consultas de base de datos.</span><span class="sxs-lookup"><span data-stu-id="5beb1-190">For this example, the interesting I/O requests were the database queries.</span></span> 

<span data-ttu-id="5beb1-191">En la siguiente imagen se muestran los resultados generados con [New APM Relic][new-relic].</span><span class="sxs-lookup"><span data-stu-id="5beb1-191">The following image shows results generated using [New Relic APM][new-relic].</span></span> <span data-ttu-id="5beb1-192">El tiempo de respuesta promedio de la base de datos fue de unos 5,6 segundos por solicitud como máximo durante la carga de trabajo máxima.</span><span class="sxs-lookup"><span data-stu-id="5beb1-192">The average database response time peaked at approximately 5.6 seconds per request during the maximum workload.</span></span> <span data-ttu-id="5beb1-193">El sistema admitió un promedio de 410 solicitudes por minuto durante la prueba.</span><span class="sxs-lookup"><span data-stu-id="5beb1-193">The system was able to support an average of 410 requests per minute throughout the test.</span></span>

![Información general del tráfico de visita de la base de datos AdventureWorks2012][databasetraffic]

### <a name="gather-detailed-data-access-information"></a><span data-ttu-id="5beb1-195">Recopilación de la información de acceso a los datos detallados</span><span class="sxs-lookup"><span data-stu-id="5beb1-195">Gather detailed data access information</span></span>

<span data-ttu-id="5beb1-196">Al profundizar más en los datos de supervisión se observa que la aplicación ejecuta tres instrucciones SQL SELECT diferentes.</span><span class="sxs-lookup"><span data-stu-id="5beb1-196">Digging deeper into the monitoring data shows the application executes three different SQL SELECT statements.</span></span> <span data-ttu-id="5beb1-197">Estas corresponden a las solicitudes generadas por Entity Framework para capturar los datos de las tablas `ProductListPriceHistory`, `Product` y `ProductSubcategory`.</span><span class="sxs-lookup"><span data-stu-id="5beb1-197">These correspond to the requests generated by Entity Framework to fetch data from the `ProductListPriceHistory`, `Product`, and `ProductSubcategory` tables.</span></span>
<span data-ttu-id="5beb1-198">Además, la consulta que recupera los datos de la tabla `ProductListPriceHistory` es la instrucción SELECT ejecutada con mucha más frecuencia, en un orden de magnitud.</span><span class="sxs-lookup"><span data-stu-id="5beb1-198">Furthermore, the query that retrieves data from the `ProductListPriceHistory` table is by far the most frequently executed SELECT statement, by an order of magnitude.</span></span>

![Consultas realizadas por la aplicación de ejemplo sometida a prueba][queries]

<span data-ttu-id="5beb1-200">Parece que el método `GetProductsInSubCategoryAsync` anterior realiza 45 consultas SELECT.</span><span class="sxs-lookup"><span data-stu-id="5beb1-200">It turns out that the `GetProductsInSubCategoryAsync` method, shown earlier, performs 45 SELECT queries.</span></span> <span data-ttu-id="5beb1-201">Cada consulta hace que la aplicación abra una nueva conexión de SQL.</span><span class="sxs-lookup"><span data-stu-id="5beb1-201">Each query causes the application to open a new SQL connection.</span></span>

![Estadísticas de consulta para la aplicación de ejemplo sometida a prueba][queries2]

> [!NOTE]
> <span data-ttu-id="5beb1-203">En esta imagen se muestra información de seguimiento de la instancia más lenta de la operación `GetProductsInSubCategoryAsync` en la prueba de carga.</span><span class="sxs-lookup"><span data-stu-id="5beb1-203">This image shows trace information for the slowest instance of the `GetProductsInSubCategoryAsync` operation in the load test.</span></span> <span data-ttu-id="5beb1-204">En un entorno de producción, es útil examinar el seguimiento de las instancias más lentas para ver si hay un patrón que indique un problema.</span><span class="sxs-lookup"><span data-stu-id="5beb1-204">In a production environment, it's useful to examine traces of the slowest instances, to see if there is a pattern that suggests a problem.</span></span> <span data-ttu-id="5beb1-205">Si simplemente observa los valores promedio, podría pasar por alto los problemas que provoquen la disminución considerable del rendimiento con carga.</span><span class="sxs-lookup"><span data-stu-id="5beb1-205">If you just look at the average values, you might overlook problems that will get dramatically worse under load.</span></span>

<span data-ttu-id="5beb1-206">En la imagen siguiente se muestran las instrucciones SQL reales que se han emitido.</span><span class="sxs-lookup"><span data-stu-id="5beb1-206">The next image shows the actual SQL statements that were issued.</span></span> <span data-ttu-id="5beb1-207">La consulta que captura la información de precios se ejecuta para cada producto de la subcategoría de producto.</span><span class="sxs-lookup"><span data-stu-id="5beb1-207">The query that fetches price information is run for each individual product in the product subcategory.</span></span> <span data-ttu-id="5beb1-208">El uso de una combinación de reduciría considerablemente el número de llamadas a la base de datos.</span><span class="sxs-lookup"><span data-stu-id="5beb1-208">Using a join would considerably reduce the number of database calls.</span></span>

![Detalles de consulta para la aplicación de ejemplo sometida a prueba][queries3]

<span data-ttu-id="5beb1-210">Si usa un asignador relacional de objetos como Entity Framework, el seguimiento de las consultas SQL puede proporcionar una visión general de cómo este convierte las llamadas con programación en instrucciones SQL e indicar las áreas donde se podría optimizar el acceso a los datos.</span><span class="sxs-lookup"><span data-stu-id="5beb1-210">If you are using an O/RM, such as Entity Framework, tracing the SQL queries can provide insight into how the O/RM translates programmatic calls into SQL statements, and indicate areas where data access might be optimized.</span></span> 

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="5beb1-211">Implementación de la solución y comprobación del resultado</span><span class="sxs-lookup"><span data-stu-id="5beb1-211">Implement the solution and verify the result</span></span>

<span data-ttu-id="5beb1-212">Al volver a escribir la llamada a Entity Framework se produjeron los siguientes resultados.</span><span class="sxs-lookup"><span data-stu-id="5beb1-212">Rewriting the call to Entity Framework produced the following results.</span></span>

![Resultados de la prueba de carga de los indicadores clave para la API fragmentada de la aplicación de ejemplo de Chatty I/O][key-indicators-chunky-io]

<span data-ttu-id="5beb1-214">Esta prueba de carga se realizó en la misma implementación, con el mismo perfil de carga.</span><span class="sxs-lookup"><span data-stu-id="5beb1-214">This load test was performed on the same deployment, using the same load profile.</span></span> <span data-ttu-id="5beb1-215">Esta vez, en el gráfico se muestra una latencia mucho menor.</span><span class="sxs-lookup"><span data-stu-id="5beb1-215">This time the graph shows much lower latency.</span></span> <span data-ttu-id="5beb1-216">El tiempo medio de solicitud con 1000 usuarios es de 5-6 segundos, ha descendido casi un minuto.</span><span class="sxs-lookup"><span data-stu-id="5beb1-216">The average request time at 1000 users is between 5 and 6 seconds, down from nearly a minute.</span></span>

<span data-ttu-id="5beb1-217">Ahora el sistema admite un promedio de 3970 solicitudes por minuto, en comparación con las 410 de la prueba anterior.</span><span class="sxs-lookup"><span data-stu-id="5beb1-217">This time the system supported an average of 3,970 requests per minute, compared to 410 for the earlier test.</span></span>

![Información general de las transacciones de la API fragmentada][databasetraffic2]

<span data-ttu-id="5beb1-219">El seguimiento de la instrucción SQL muestra que todos los datos se capturan en una única instrucción SELECT.</span><span class="sxs-lookup"><span data-stu-id="5beb1-219">Tracing the SQL statement shows that all the data is fetched in a single SELECT statement.</span></span> <span data-ttu-id="5beb1-220">Aunque esta consulta es considerablemente más compleja, se realiza solo una vez por operación.</span><span class="sxs-lookup"><span data-stu-id="5beb1-220">Although this query is considerably more complex, it is performed only once per operation.</span></span> <span data-ttu-id="5beb1-221">Y mientras las combinaciones complejas pueden resultar caras, los sistemas de bases de datos relacionales están optimizados para este tipo de consulta.</span><span class="sxs-lookup"><span data-stu-id="5beb1-221">And while complex joins can become expensive, relational database systems are optimized for this type of query.</span></span>  

![Detalles de consulta de la API fragmentada][queries4]

## <a name="related-resources"></a><span data-ttu-id="5beb1-223">Recursos relacionados</span><span class="sxs-lookup"><span data-stu-id="5beb1-223">Related resources</span></span>

- <span data-ttu-id="5beb1-224">[Procedimientos recomendados para el diseño de API][api-design]</span><span class="sxs-lookup"><span data-stu-id="5beb1-224">[API Design best practices][api-design]</span></span>
- <span data-ttu-id="5beb1-225">[Procedimientos recomendados para el almacenamiento en caché][caching-guidance]</span><span class="sxs-lookup"><span data-stu-id="5beb1-225">[Caching best practices][caching-guidance]</span></span>
- <span data-ttu-id="5beb1-226">[Manual básico de coherencia de datos][data-consistency-guidance]</span><span class="sxs-lookup"><span data-stu-id="5beb1-226">[Data Consistency Primer][data-consistency-guidance]</span></span>
- <span data-ttu-id="5beb1-227">[Antipatrón Extraneous Fetching][extraneous-fetching]</span><span class="sxs-lookup"><span data-stu-id="5beb1-227">[Extraneous Fetching antipattern][extraneous-fetching]</span></span>
- <span data-ttu-id="5beb1-228">[Antipatrón No Caching][no-cache]</span><span class="sxs-lookup"><span data-stu-id="5beb1-228">[No Caching antipattern][no-cache]</span></span>

[api-design]: ../../best-practices/api-design.md
[caching-guidance]: ../../best-practices/caching.md
[code-sample]:  https://github.com/mspnp/performance-optimization/tree/master/ChattyIO
[data-consistency-guidance]: http://https://msdn.microsoft.com/library/dn589800.aspx
[ef]: /ef/
[extraneous-fetching]: ../extraneous-fetching/index.md
[new-relic]: https://newrelic.com/application-monitoring
[no-cache]: ../no-caching/index.md

[key-indicators-chatty-io]: _images/ChattyIO.jpg
[key-indicators-chunky-io]: _images/ChunkyIO.jpg
[databasetraffic]: _images/DatabaseTraffic.jpg
[databasetraffic2]: _images/DatabaseTraffic2.jpg
[queries]: _images/DatabaseQueries.jpg
[queries2]: _images/DatabaseQueries2.jpg
[queries3]: _images/DatabaseQueries3.jpg
[queries4]: _images/DatabaseQueries4.jpg

