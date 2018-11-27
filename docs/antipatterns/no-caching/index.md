---
title: Antipatrón No Caching
description: Si se capturan los mismos datos varias veces, puede reducirse el rendimiento la y escalabilidad.
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: ec19cde567fb63248c121328322e834d99c841e8
ms.sourcegitcommit: 19a517a2fb70768b3edb9a7c3c37197baa61d9b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/26/2018
ms.locfileid: "52295589"
---
# <a name="no-caching-antipattern"></a><span data-ttu-id="86ca3-103">Antipatrón No Caching</span><span class="sxs-lookup"><span data-stu-id="86ca3-103">No Caching antipattern</span></span>

<span data-ttu-id="86ca3-104">En una aplicación en la nube que controla muchas solicitudes simultáneas, capturar repetidamente los mismos datos puede reducir el rendimiento y la escalabilidad.</span><span class="sxs-lookup"><span data-stu-id="86ca3-104">In a cloud application that handles many concurrent requests, repeatedly fetching the same data can reduce performance and scalability.</span></span> 

## <a name="problem-description"></a><span data-ttu-id="86ca3-105">Descripción del problema</span><span class="sxs-lookup"><span data-stu-id="86ca3-105">Problem description</span></span>

<span data-ttu-id="86ca3-106">Cuando los datos no están almacenados en caché, puede ocasionar diversos comportamientos no deseados, como son:</span><span class="sxs-lookup"><span data-stu-id="86ca3-106">When data is not cached, it can cause a number of undesirable behaviors, including:</span></span>

- <span data-ttu-id="86ca3-107">Capturar repetidamente la misma información de un recurso cuyo acceso resulta costoso, en cuanto a la sobrecarga de E/S o la latencia.</span><span class="sxs-lookup"><span data-stu-id="86ca3-107">Repeatedly fetching the same information from a resource that is expensive to access, in terms of I/O overhead or latency.</span></span>
- <span data-ttu-id="86ca3-108">Construir varias veces los mismos objetos o estructuras de datos para varias solicitudes.</span><span class="sxs-lookup"><span data-stu-id="86ca3-108">Repeatedly constructing the same objects or data structures for multiple requests.</span></span>
- <span data-ttu-id="86ca3-109">Un exceso de llamadas a un servicio remoto que tiene una cuota de servicio y que limita a los clientes más allá de un determinado período.</span><span class="sxs-lookup"><span data-stu-id="86ca3-109">Making excessive calls to a remote service that has a service quota and throttles clients past a certain limit.</span></span>

<span data-ttu-id="86ca3-110">A su vez, estos problemas pueden provocar tiempos de respuesta insuficientes, mayor contención en el almacén de datos y una escalabilidad escasa.</span><span class="sxs-lookup"><span data-stu-id="86ca3-110">In turn, these problems can lead to poor response times, increased contention in the data store, and poor scalability.</span></span>

<span data-ttu-id="86ca3-111">En el ejemplo siguiente se utiliza Entity Framework para conectarse a una base de datos.</span><span class="sxs-lookup"><span data-stu-id="86ca3-111">The following example uses Entity Framework to connect to a database.</span></span> <span data-ttu-id="86ca3-112">Cada solicitud de cliente produce una llamada a la base de datos, incluso si varias solicitudes recuperan exactamente los mismos datos.</span><span class="sxs-lookup"><span data-stu-id="86ca3-112">Every client request results in a call to the database, even if multiple requests are fetching exactly the same data.</span></span> <span data-ttu-id="86ca3-113">El costo de las solicitudes repetidas, en cuanto a los cargos por el acceso a los datos y la sobrecarga de E/S, puede acumularse rápidamente.</span><span class="sxs-lookup"><span data-stu-id="86ca3-113">The cost of repeated requests, in terms of I/O overhead and data access charges, can accumulate quickly.</span></span>

```csharp
public class PersonRepository : IPersonRepository
{
    public async Task<Person> GetAsync(int id)
    {
        using (var context = new AdventureWorksContext())
        {
            return await context.People
                .Where(p => p.Id == id)
                .FirstOrDefaultAsync()
                .ConfigureAwait(false);
        }
    }
}
```

<span data-ttu-id="86ca3-114">Puede encontrar el ejemplo completo [aquí][sample-app].</span><span class="sxs-lookup"><span data-stu-id="86ca3-114">You can find the complete sample [here][sample-app].</span></span>

<span data-ttu-id="86ca3-115">Este antipatrón suele ocurrir porque:</span><span class="sxs-lookup"><span data-stu-id="86ca3-115">This antipattern typically occurs because:</span></span>

- <span data-ttu-id="86ca3-116">No usar una memoria caché es más fácil de implementar y funciona bien con cargas bajas.</span><span class="sxs-lookup"><span data-stu-id="86ca3-116">Not using a cache is simpler to implement, and it works fine under low loads.</span></span> <span data-ttu-id="86ca3-117">El almacenamiento en caché hace que el código sea más complicado.</span><span class="sxs-lookup"><span data-stu-id="86ca3-117">Caching makes the code more complicated.</span></span> 
- <span data-ttu-id="86ca3-118">No se entienden con claridad las ventajas y desventajas del uso del almacenamiento en caché.</span><span class="sxs-lookup"><span data-stu-id="86ca3-118">The benefits and drawbacks of using a cache are not clearly understood.</span></span>
- <span data-ttu-id="86ca3-119">Preocupa la sobrecarga que supone mantener la precisión y la actualización de los datos almacenados en caché.</span><span class="sxs-lookup"><span data-stu-id="86ca3-119">There is concern about the overhead of maintaining the accuracy and freshness of cached data.</span></span>
- <span data-ttu-id="86ca3-120">Una aplicación se migró desde un sistema local, en el que la latencia de red no era un problema, y el sistema se ejecutaba en un costoso hardware de alto rendimiento, por lo que el almacenamiento en caché no se consideró en el diseño original.</span><span class="sxs-lookup"><span data-stu-id="86ca3-120">An application was migrated from an on-premises system, where network latency was not an issue, and the system ran on expensive high-performance hardware, so caching wasn't considered in the original design.</span></span>
- <span data-ttu-id="86ca3-121">Los desarrolladores no son conscientes de que el almacenamiento en caché es una posibilidad en un escenario determinado.</span><span class="sxs-lookup"><span data-stu-id="86ca3-121">Developers aren't aware that caching is a possibility in a given scenario.</span></span> <span data-ttu-id="86ca3-122">Por ejemplo, no pueden considerar usar ETags al implementar una API de web.</span><span class="sxs-lookup"><span data-stu-id="86ca3-122">For example, developers may not think of using ETags when implementing a web API.</span></span>

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="86ca3-123">Procedimiento para corregir el problema</span><span class="sxs-lookup"><span data-stu-id="86ca3-123">How to fix the problem</span></span>

<span data-ttu-id="86ca3-124">La estrategia de almacenamiento en caché más popular es el sistema *a petición* o el modelo *cache-aside*.</span><span class="sxs-lookup"><span data-stu-id="86ca3-124">The most popular caching strategy is the *on-demand* or *cache-aside* strategy.</span></span>

- <span data-ttu-id="86ca3-125">En las lecturas, la aplicación intenta leer los datos de la memoria caché.</span><span class="sxs-lookup"><span data-stu-id="86ca3-125">On read, the application tries to read the data from the cache.</span></span> <span data-ttu-id="86ca3-126">Si los datos no están allí, la aplicación los recupera del origen de datos y los agrega.</span><span class="sxs-lookup"><span data-stu-id="86ca3-126">If the data isn't in the cache, the application retrieves it from the data source and adds it to the cache.</span></span>
- <span data-ttu-id="86ca3-127">Durante las escrituras, la aplicación escribe el cambio directamente en el origen de datos y quita el valor antiguo de la memoria caché.</span><span class="sxs-lookup"><span data-stu-id="86ca3-127">On write, the application writes the change directly to the data source and removes the old value from the cache.</span></span> <span data-ttu-id="86ca3-128">Se recuperará y se agregará a la memoria caché la próxima vez que se requiera.</span><span class="sxs-lookup"><span data-stu-id="86ca3-128">It will be retrieved and added to the cache the next time it is required.</span></span>

<span data-ttu-id="86ca3-129">Este enfoque es adecuado para los datos que cambian con frecuencia.</span><span class="sxs-lookup"><span data-stu-id="86ca3-129">This approach is suitable for data that changes frequently.</span></span> <span data-ttu-id="86ca3-130">Aquí se muestra el ejemplo anterior actualizado para utilizar el modelo [Cache-Aside][cache-aside-pattern].</span><span class="sxs-lookup"><span data-stu-id="86ca3-130">Here is the previous example updated to use the [Cache-Aside][cache-aside-pattern] pattern.</span></span>  

```csharp
public class CachedPersonRepository : IPersonRepository
{
    private readonly PersonRepository _innerRepository;

    public CachedPersonRepository(PersonRepository innerRepository)
    {
        _innerRepository = innerRepository;
    }

    public async Task<Person> GetAsync(int id)
    {
        return await CacheService.GetAsync<Person>("p:" + id, () => _innerRepository.GetAsync(id)).ConfigureAwait(false);
    }
}

public class CacheService
{
    private static ConnectionMultiplexer _connection;

    public static async Task<T> GetAsync<T>(string key, Func<Task<T>> loadCache, double expirationTimeInMinutes)
    {
        IDatabase cache = Connection.GetDatabase();
        T value = await GetAsync<T>(cache, key).ConfigureAwait(false);
        if (value == null)
        {
            // Value was not found in the cache. Call the lambda to get the value from the database.
            value = await loadCache().ConfigureAwait(false);
            if (value != null)
            {
                // Add the value to the cache.
                await SetAsync(cache, key, value, expirationTimeInMinutes).ConfigureAwait(false);
            }
        }
        return value;
    }
}
```

<span data-ttu-id="86ca3-131">Tenga en cuenta que el método `GetAsync` llama ahora a la clase `CacheService`, en lugar de llamar directamente a la base de datos.</span><span class="sxs-lookup"><span data-stu-id="86ca3-131">Notice that the `GetAsync` method now calls the `CacheService` class, rather than calling the database directly.</span></span> <span data-ttu-id="86ca3-132">La clase `CacheService` primero intenta obtener el elemento de Azure Redis Cache.</span><span class="sxs-lookup"><span data-stu-id="86ca3-132">The `CacheService` class first tries to get the item from Azure Redis Cache.</span></span> <span data-ttu-id="86ca3-133">Si el valor no se encuentra allí, la clase `CacheService` invoca una función lambda que le pasó el responsable de realizar la llamada.</span><span class="sxs-lookup"><span data-stu-id="86ca3-133">If the value isn't found in Redis Cache, the `CacheService` invokes a lambda function that was passed to it by the caller.</span></span> <span data-ttu-id="86ca3-134">La función lambda debe recuperar los datos de la base de datos.</span><span class="sxs-lookup"><span data-stu-id="86ca3-134">The lambda function is responsible for fetching the data from the database.</span></span> <span data-ttu-id="86ca3-135">Esta implementación desvincula el repositorio de la solución de almacenamiento en caché concreta y `CacheService` de la base de datos.</span><span class="sxs-lookup"><span data-stu-id="86ca3-135">This implementation decouples the repository from the particular caching solution, and decouples the `CacheService` from the database.</span></span> 

## <a name="considerations"></a><span data-ttu-id="86ca3-136">Consideraciones</span><span class="sxs-lookup"><span data-stu-id="86ca3-136">Considerations</span></span>

- <span data-ttu-id="86ca3-137">Si la memoria caché no está disponible, quizás debido a un error transitorio, no devuelva un error al cliente.</span><span class="sxs-lookup"><span data-stu-id="86ca3-137">If the cache is unavailable, perhaps because of a transient failure, don't return an error to the client.</span></span> <span data-ttu-id="86ca3-138">En su lugar, capture los datos del origen de datos original.</span><span class="sxs-lookup"><span data-stu-id="86ca3-138">Instead, fetch the data from the original data source.</span></span> <span data-ttu-id="86ca3-139">Sin embargo, tenga en cuenta que, mientras se recupera la memoria caché, el almacén de datos original podría inundarse de solicitudes, dando lugar a tiempos de espera y errores de conexión.</span><span class="sxs-lookup"><span data-stu-id="86ca3-139">However, be aware that while the cache is being recovered, the original data store could be swamped with requests, resulting in timeouts and failed connections.</span></span> <span data-ttu-id="86ca3-140">(Después de todo, esta es una de las motivaciones para utilizar una memoria caché en primer lugar). Use una técnica como [Circuit Breaker pattern][circuit-breaker] (patrón Circuit Breaker) para evitar sobrecargar el origen de datos.</span><span class="sxs-lookup"><span data-stu-id="86ca3-140">(After all, this is one of the motivations for using a cache in the first place.) Use a technique such as the [Circuit Breaker pattern][circuit-breaker] to avoid overwhelming the data source.</span></span>

- <span data-ttu-id="86ca3-141">Las aplicaciones que almacenan en caché datos no estáticos se deben diseñar para admitir una posible coherencia.</span><span class="sxs-lookup"><span data-stu-id="86ca3-141">Applications that cache nonstatic data should be designed to support eventual consistency.</span></span>

- <span data-ttu-id="86ca3-142">Para las API de web, puede admitir el almacenamiento en caché de cliente incluyendo un encabezado Cache-Control en los mensajes de solicitud y respuesta, y usar ETags para identificar las versiones de los objetos.</span><span class="sxs-lookup"><span data-stu-id="86ca3-142">For web APIs, you can support client-side caching by including a Cache-Control header in request and response messages, and using ETags to identify versions of objects.</span></span> <span data-ttu-id="86ca3-143">Para más información, consulte [Implementación de la API][api-implementation].</span><span class="sxs-lookup"><span data-stu-id="86ca3-143">For more information, see [API implementation][api-implementation].</span></span>

- <span data-ttu-id="86ca3-144">No tiene que almacenar en caché las entidades completas.</span><span class="sxs-lookup"><span data-stu-id="86ca3-144">You don't have to cache entire entities.</span></span> <span data-ttu-id="86ca3-145">Si la mayor parte de una entidad es estática, pero solo una pequeña parte cambia con frecuencia, almacene en caché los elementos estáticos y recupere los dinámicos del origen de datos.</span><span class="sxs-lookup"><span data-stu-id="86ca3-145">If most of an entity is static but only a small piece changes frequently, cache the static elements and retrieve the dynamic elements from the data source.</span></span> <span data-ttu-id="86ca3-146">Este enfoque puede ayudar a reducir el volumen de operaciones de E/S que se realizan en el origen de datos.</span><span class="sxs-lookup"><span data-stu-id="86ca3-146">This approach can help to reduce the volume of I/O being performed against the data source.</span></span>

- <span data-ttu-id="86ca3-147">En algunos casos, si los datos volátiles son de corta duración, puede ser útil almacenarlos en caché.</span><span class="sxs-lookup"><span data-stu-id="86ca3-147">In some cases, if volatile data is short-lived, it can be useful to cache it.</span></span> <span data-ttu-id="86ca3-148">Por ejemplo, considere un dispositivo que envía continuamente actualizaciones de estado.</span><span class="sxs-lookup"><span data-stu-id="86ca3-148">For example, consider a device that continually sends status updates.</span></span> <span data-ttu-id="86ca3-149">Podría tener sentido almacenar en memoria caché esta información cuando llega y no escribirla en un almacén persistente.</span><span class="sxs-lookup"><span data-stu-id="86ca3-149">It might make sense to cache this information as it arrives, and not write it to a persistent store at all.</span></span>  

- <span data-ttu-id="86ca3-150">Para evitar que los datos se queden obsoletos, muchas soluciones de almacenamiento en caché admiten períodos de expiración configurables, de modo que se quitan automáticamente de la memoria caché después de un intervalo especificado.</span><span class="sxs-lookup"><span data-stu-id="86ca3-150">To prevent data from becoming stale, many caching solutions support configurable expiration periods, so that data is automatically removed from the cache after a specified interval.</span></span> <span data-ttu-id="86ca3-151">Debe ajustar la hora de expiración para su escenario.</span><span class="sxs-lookup"><span data-stu-id="86ca3-151">You may need to tune the expiration time for your scenario.</span></span> <span data-ttu-id="86ca3-152">Los datos muy estáticos pueden permanecer en la memoria caché durante períodos más largos que los volátiles, que pueden quedar obsoletos con rapidez.</span><span class="sxs-lookup"><span data-stu-id="86ca3-152">Data that is highly static can stay in the cache for longer periods than volatile data that may become stale quickly.</span></span>

- <span data-ttu-id="86ca3-153">Si la solución de almacenamiento en caché no proporciona una expiración integrada, debe implementar un proceso en segundo plano que limpie en ocasiones la memoria caché, para evitar que crezca de forma ilimitada.</span><span class="sxs-lookup"><span data-stu-id="86ca3-153">If the caching solution doesn't provide built-in expiration, you may need to implement a background process that occasionally sweeps the cache, to prevent it from growing without limits.</span></span> 

- <span data-ttu-id="86ca3-154">Además de almacenar en caché datos de un origen de datos externo, puede usar el almacenamiento en caché para guardar los resultados de cálculos complejos.</span><span class="sxs-lookup"><span data-stu-id="86ca3-154">Besides caching data from an external data source, you can use caching to save the results of complex computations.</span></span> <span data-ttu-id="86ca3-155">Sin embargo, antes debe instrumentar la aplicación para determinar si realmente está limitada por la CPU.</span><span class="sxs-lookup"><span data-stu-id="86ca3-155">Before you do that, however, instrument the application to determine whether the application is really CPU bound.</span></span>

- <span data-ttu-id="86ca3-156">Puede ser útil preparar la memoria caché cuando la aplicación se inicia.</span><span class="sxs-lookup"><span data-stu-id="86ca3-156">It might be useful to prime the cache when the application starts.</span></span> <span data-ttu-id="86ca3-157">Rellene la memoria caché con los datos que tengan más probabilidad de usarse.</span><span class="sxs-lookup"><span data-stu-id="86ca3-157">Populate the cache with the data that is most likely to be used.</span></span>

- <span data-ttu-id="86ca3-158">Incluya siempre instrumental que detecte los aciertos y los errores de caché.</span><span class="sxs-lookup"><span data-stu-id="86ca3-158">Always include instrumentation that detects cache hits and cache misses.</span></span> <span data-ttu-id="86ca3-159">Utilice esta información para ajustar las directivas de almacenamiento en caché, por ejemplo, qué datos almacenar en ella y cuánto tiempo mantenerlos antes de que expiren.</span><span class="sxs-lookup"><span data-stu-id="86ca3-159">Use this information to tune caching policies, such what data to cache, and how long to hold data in the cache before it expires.</span></span>

- <span data-ttu-id="86ca3-160">Si la falta de almacenamiento en caché constituye un cuello de botella, agregarlo puede aumentar tanto el volumen de solicitudes que el front-end web llegue a sobrecargarse.</span><span class="sxs-lookup"><span data-stu-id="86ca3-160">If the lack of caching is a bottleneck, then adding caching may increase the volume of requests so much that the web front end becomes overloaded.</span></span> <span data-ttu-id="86ca3-161">Los clientes pueden empezar a recibir errores HTTP 503 (servicio no disponible).</span><span class="sxs-lookup"><span data-stu-id="86ca3-161">Clients may start to receive HTTP 503 (Service Unavailable) errors.</span></span> <span data-ttu-id="86ca3-162">Se trata de una indicación de que debe escalar horizontalmente el front-end.</span><span class="sxs-lookup"><span data-stu-id="86ca3-162">These are an indication that you should scale out the front end.</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="86ca3-163">Procedimiento para detectar el problema</span><span class="sxs-lookup"><span data-stu-id="86ca3-163">How to detect the problem</span></span>

<span data-ttu-id="86ca3-164">Puede realizar los pasos siguientes para ayudar a identificar si la falta de almacenamiento en caché está causando problemas de rendimiento:</span><span class="sxs-lookup"><span data-stu-id="86ca3-164">You can perform the following steps to help identify whether lack of caching is causing performance problems:</span></span>

1. <span data-ttu-id="86ca3-165">Revise el código de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="86ca3-165">Review the application design.</span></span> <span data-ttu-id="86ca3-166">Realice un inventario de todos los almacenes de datos que la aplicación usa.</span><span class="sxs-lookup"><span data-stu-id="86ca3-166">Take an inventory of all the data stores that the application uses.</span></span> <span data-ttu-id="86ca3-167">Para cada uno, determine si la aplicación usa una memoria caché.</span><span class="sxs-lookup"><span data-stu-id="86ca3-167">For each, determine whether the application is using a cache.</span></span> <span data-ttu-id="86ca3-168">Si es posible, determine con qué frecuencia cambian los datos.</span><span class="sxs-lookup"><span data-stu-id="86ca3-168">If possible, determine how frequently the data changes.</span></span> <span data-ttu-id="86ca3-169">Buenos candidatos iniciales para el almacenamiento en caché son los datos que cambian lentamente y los datos estáticos de referencia que se lean con frecuencia.</span><span class="sxs-lookup"><span data-stu-id="86ca3-169">Good initial candidates for caching include data that changes slowly, and static reference data that is read frequently.</span></span> 

2. <span data-ttu-id="86ca3-170">Instrumente la aplicación y supervise el sistema real para averiguar la frecuencia con que la aplicación recupera datos o calcula la información.</span><span class="sxs-lookup"><span data-stu-id="86ca3-170">Instrument the application and monitor the live system to find out how frequently the application retrieves data or calculates information.</span></span>

3. <span data-ttu-id="86ca3-171">Genere perfiles de la aplicación en un entorno de prueba para capturar las métricas de bajo nivel sobre la sobrecarga asociada a operaciones de acceso a datos u otros cálculos realizados con frecuencia.</span><span class="sxs-lookup"><span data-stu-id="86ca3-171">Profile the application in a test environment to capture low-level metrics about the overhead associated with data access operations or other frequently performed calculations.</span></span>

4. <span data-ttu-id="86ca3-172">Realice pruebas de carga de un entorno de prueba para identificar cómo responde el sistema con una carga de trabajo normal y con una carga intensa.</span><span class="sxs-lookup"><span data-stu-id="86ca3-172">Perform load testing in a test environment to identify how the system responds under a normal workload and under heavy load.</span></span> <span data-ttu-id="86ca3-173">Las pruebas de carga deben simular el patrón de acceso a datos observado en el entorno de producción con cargas de trabajo realistas.</span><span class="sxs-lookup"><span data-stu-id="86ca3-173">Load testing should simulate the pattern of data access observed in the production environment using realistic workloads.</span></span> 

5. <span data-ttu-id="86ca3-174">Examine las estadísticas de acceso a los datos de los almacenes de datos subyacentes y revise con qué frecuencia se repiten las mismas solicitudes de datos.</span><span class="sxs-lookup"><span data-stu-id="86ca3-174">Examine the data access statistics for the underlying data stores and review how often the same data requests are repeated.</span></span> 


## <a name="example-diagnosis"></a><span data-ttu-id="86ca3-175">Diagnóstico de ejemplo</span><span class="sxs-lookup"><span data-stu-id="86ca3-175">Example diagnosis</span></span>

<span data-ttu-id="86ca3-176">En las secciones siguientes se aplican estos pasos para la aplicación de ejemplo descrita anteriormente.</span><span class="sxs-lookup"><span data-stu-id="86ca3-176">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="instrument-the-application-and-monitor-the-live-system"></a><span data-ttu-id="86ca3-177">Instrumentación de la aplicación y supervisión de los sistemas en vivo</span><span class="sxs-lookup"><span data-stu-id="86ca3-177">Instrument the application and monitor the live system</span></span>

<span data-ttu-id="86ca3-178">Instrumente la aplicación y supervísela para obtener información acerca de las solicitudes específicas que realizan los usuarios mientras está en producción.</span><span class="sxs-lookup"><span data-stu-id="86ca3-178">Instrument the application and monitor it to get information about the specific requests that users make while the application is in production.</span></span> 

<span data-ttu-id="86ca3-179">La siguiente imagen muestra la supervisión de los datos capturados por [New Relic][NewRelic] durante una prueba de carga.</span><span class="sxs-lookup"><span data-stu-id="86ca3-179">The following image shows monitoring data captured by [New Relic][NewRelic] during a load test.</span></span> <span data-ttu-id="86ca3-180">En este caso, la única operación HTTP GET realizada es `Person/GetAsync`.</span><span class="sxs-lookup"><span data-stu-id="86ca3-180">In this case, the only HTTP GET operation performed is `Person/GetAsync`.</span></span> <span data-ttu-id="86ca3-181">Pero, en un entorno real de producción, conocer la frecuencia relativa con que se realiza cada solicitud puede ofrecerle una visión general de los recursos que se deben almacenar en caché.</span><span class="sxs-lookup"><span data-stu-id="86ca3-181">But in a live production environment, knowing the relative frequency that each request is performed can give you insight into which resources should be cached.</span></span>

![New Relic que muestra las solicitudes de servidor para la aplicación CachingDemo][NewRelic-server-requests]

<span data-ttu-id="86ca3-183">Si necesita realizar un análisis más profundo, puede usar un generador de perfiles para capturar los datos de rendimiento de bajo nivel en un entorno de prueba (no en el sistema de producción).</span><span class="sxs-lookup"><span data-stu-id="86ca3-183">If you need a deeper analysis, you can use a profiler to capture low-level performance data in a test environment (not the production system).</span></span> <span data-ttu-id="86ca3-184">Observe las métricas, como la velocidad de las solicitudes de E/S, el uso de la memoria y el uso de la CPU.</span><span class="sxs-lookup"><span data-stu-id="86ca3-184">Look at metrics such as I/O request rates, memory usage, and CPU utilization.</span></span> <span data-ttu-id="86ca3-185">Estas métricas pueden mostrar un gran número de solicitudes realizadas a un almacén de datos o un servicio, o que se repite un proceso que efectúa el mismo cálculo.</span><span class="sxs-lookup"><span data-stu-id="86ca3-185">These metrics may show a large number of requests to a data store or service, or repeated processing that performs the same calculation.</span></span> 

### <a name="load-test-the-application"></a><span data-ttu-id="86ca3-186">Prueba de carga de la aplicación</span><span class="sxs-lookup"><span data-stu-id="86ca3-186">Load test the application</span></span>

<span data-ttu-id="86ca3-187">El siguiente gráfico muestra los resultados de la aplicación de ejemplo de la prueba de carga.</span><span class="sxs-lookup"><span data-stu-id="86ca3-187">The following graph shows the results of load testing the sample application.</span></span> <span data-ttu-id="86ca3-188">La prueba de carga simula una carga por pasos de hasta 800 usuarios que realizan una serie de operaciones habituales.</span><span class="sxs-lookup"><span data-stu-id="86ca3-188">The load test simulates a step load of up to 800 users performing a typical series of operations.</span></span> 

![Resultados de la prueba de carga del rendimiento para el escenario sin almacenamiento en caché][Performance-Load-Test-Results-Uncached]

<span data-ttu-id="86ca3-190">El número de pruebas superadas realizadas cada segundo alcanza un nivel estable y las solicitudes adicionales se ralentizan como resultado.</span><span class="sxs-lookup"><span data-stu-id="86ca3-190">The number of successful tests performed each second reaches a plateau, and additional requests are slowed as a result.</span></span> <span data-ttu-id="86ca3-191">El tiempo promedio de prueba aumenta regularmente con la carga de trabajo.</span><span class="sxs-lookup"><span data-stu-id="86ca3-191">The average test time steadily increases with the workload.</span></span> <span data-ttu-id="86ca3-192">El tiempo de respuesta se estabiliza una vez que la carga de usuarios alcanza el máximo.</span><span class="sxs-lookup"><span data-stu-id="86ca3-192">The response time levels off once the user load peaks.</span></span>

### <a name="examine-data-access-statistics"></a><span data-ttu-id="86ca3-193">Examen de las estadísticas de acceso a los datos</span><span class="sxs-lookup"><span data-stu-id="86ca3-193">Examine data access statistics</span></span>

<span data-ttu-id="86ca3-194">Las estadísticas de acceso a los datos y otra información que un almacén proporciona pueden ofrecerle información útil, como las consultas que se repiten con más frecuencia.</span><span class="sxs-lookup"><span data-stu-id="86ca3-194">Data access statistics and other information provided by a data store can give useful information, such as which queries are repeated most frequently.</span></span> <span data-ttu-id="86ca3-195">Por ejemplo, en Microsoft SQL Server, la vista de administración `sys.dm_exec_query_stats` tiene información estadística sobre las consultas ejecutadas recientemente.</span><span class="sxs-lookup"><span data-stu-id="86ca3-195">For example, in Microsoft SQL Server, the `sys.dm_exec_query_stats` management view has statistical information for recently executed queries.</span></span> <span data-ttu-id="86ca3-196">El texto de cada consulta está disponible en la vista `sys.dm_exec-query_plan`.</span><span class="sxs-lookup"><span data-stu-id="86ca3-196">The text for each query is available in the `sys.dm_exec-query_plan` view.</span></span> <span data-ttu-id="86ca3-197">Puede utilizar una herramienta como SQL Server Management Studio para ejecutar la siguiente consulta SQL y determinar la frecuencia con la que se realizan las consultas.</span><span class="sxs-lookup"><span data-stu-id="86ca3-197">You can use a tool such as SQL Server Management Studio to run the following SQL query and determine how frequently queries are performed.</span></span>

```SQL
SELECT UseCounts, Text, Query_Plan
FROM sys.dm_exec_cached_plans
CROSS APPLY sys.dm_exec_sql_text(plan_handle)
CROSS APPLY sys.dm_exec_query_plan(plan_handle)
```

<span data-ttu-id="86ca3-198">La columna `UseCount` en los resultados indica la frecuencia con la que se ejecuta cada consulta.</span><span class="sxs-lookup"><span data-stu-id="86ca3-198">The `UseCount` column in the results indicates how frequently each query is run.</span></span> <span data-ttu-id="86ca3-199">La siguiente imagen muestra que la tercera consulta se ejecuta más de 250 000 veces, significativamente más que cualquier otra.</span><span class="sxs-lookup"><span data-stu-id="86ca3-199">The following image shows that the third query was run more than 250,000 times, significantly more than any other query.</span></span>

![Resultados de la consulta de las vistas de administración dinámica en SQL Server Management Server][Dynamic-Management-Views]

<span data-ttu-id="86ca3-201">Esta es la consulta SQL que está causando tantas solicitudes de base de datos:</span><span class="sxs-lookup"><span data-stu-id="86ca3-201">Here is the SQL query that is causing so many database requests:</span></span> 

```SQL
(@p__linq__0 int)SELECT TOP (2)
[Extent1].[BusinessEntityId] AS [BusinessEntityId],
[Extent1].[FirstName] AS [FirstName],
[Extent1].[LastName] AS [LastName]
FROM [Person].[Person] AS [Extent1]
WHERE [Extent1].[BusinessEntityId] = @p__linq__0
```

<span data-ttu-id="86ca3-202">Esta es la consulta que Entity Framework genera en el método `GetByIdAsync` mostrado anteriormente.</span><span class="sxs-lookup"><span data-stu-id="86ca3-202">This is the query that Entity Framework generates in `GetByIdAsync` method shown earlier.</span></span>

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="86ca3-203">Implementación de la solución y comprobación del resultado</span><span class="sxs-lookup"><span data-stu-id="86ca3-203">Implement the solution and verify the result</span></span>

<span data-ttu-id="86ca3-204">Después de incorporar una memoria caché, repita las pruebas de carga y compare los resultados de las anteriores sin memoria caché.</span><span class="sxs-lookup"><span data-stu-id="86ca3-204">After you incorporate a cache, repeat the load tests and compare the results to the earlier load tests without a cache.</span></span> <span data-ttu-id="86ca3-205">Estos son los resultados de las pruebas de carga después de agregar una memoria caché a la aplicación de ejemplo.</span><span class="sxs-lookup"><span data-stu-id="86ca3-205">Here are the load test results after adding a cache to the sample application.</span></span>

![Resultados de la prueba de carga del rendimiento para el escenario con almacenamiento en caché][Performance-Load-Test-Results-Cached]

<span data-ttu-id="86ca3-207">El volumen de pruebas superadas todavía alcanza un nivel estable, pero con una carga de usuarios mayor.</span><span class="sxs-lookup"><span data-stu-id="86ca3-207">The volume of successful tests still reaches a plateau, but at a higher user load.</span></span> <span data-ttu-id="86ca3-208">La tasa de solicitudes en esta carga es significativamente más alta que la anterior.</span><span class="sxs-lookup"><span data-stu-id="86ca3-208">The request rate at this load is significantly higher than earlier.</span></span> <span data-ttu-id="86ca3-209">El tiempo promedio de prueba sigue aumentando con la carga, pero el tiempo de respuesta máximo es de 0,05 ms, en comparación con el valor de 1 ms anterior &mdash;una mejora del 20&times;.</span><span class="sxs-lookup"><span data-stu-id="86ca3-209">Average test time still increases with load, but the maximum response time is 0.05 ms, compared with 1ms earlier &mdash; a 20&times; improvement.</span></span> 

## <a name="related-resources"></a><span data-ttu-id="86ca3-210">Recursos relacionados</span><span class="sxs-lookup"><span data-stu-id="86ca3-210">Related resources</span></span>

- <span data-ttu-id="86ca3-211">[Procedimientos recomendados para la implementación de API][api-implementation]</span><span class="sxs-lookup"><span data-stu-id="86ca3-211">[API implementation best practices][api-implementation]</span></span>
- <span data-ttu-id="86ca3-212">[Modelo Cache-Aside][cache-aside-pattern]</span><span class="sxs-lookup"><span data-stu-id="86ca3-212">[Cache-Aside Pattern][cache-aside-pattern]</span></span>
- <span data-ttu-id="86ca3-213">[Procedimientos recomendados para el almacenamiento en caché][caching-guidance]</span><span class="sxs-lookup"><span data-stu-id="86ca3-213">[Caching best practices][caching-guidance]</span></span>
- <span data-ttu-id="86ca3-214">[Patrón Circuit Breaker][circuit-breaker]</span><span class="sxs-lookup"><span data-stu-id="86ca3-214">[Circuit Breaker pattern][circuit-breaker]</span></span>

[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/NoCaching
[cache-aside-pattern]: /azure/architecture/patterns/cache-aside
[caching-guidance]: ../../best-practices/caching.md
[circuit-breaker]: ../../patterns/circuit-breaker.md
[api-implementation]: ../../best-practices/api-implementation.md#optimizing-client-side-data-access
[NewRelic]: https://newrelic.com/partner/azure
[NewRelic-server-requests]: _images/New-Relic.jpg
[Performance-Load-Test-Results-Uncached]:_images/InitialLoadTestResults.jpg
[Dynamic-Management-Views]: _images/SQLServerManagementStudio.jpg
[Performance-Load-Test-Results-Cached]: _images/CachedLoadTestResults.jpg
