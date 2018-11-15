---
title: Cache-Aside
description: Carga datos a petición en una memoria caché desde un almacén de datos
keywords: Patrón de diseño
author: dragon119
ms.date: 11/01/2018
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- data-management
- performance-scalability
ms.openlocfilehash: 4c93ed02ff28e79cedc26f83364592baba96821d
ms.sourcegitcommit: dbbf914757b03cdee7a274204f9579fa63d7eed2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/02/2018
ms.locfileid: "50916391"
---
# <a name="cache-aside-pattern"></a><span data-ttu-id="be267-104">Patrón Cache-Aside</span><span class="sxs-lookup"><span data-stu-id="be267-104">Cache-Aside pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="be267-105">Carga datos a petición en una caché desde un almacén de datos</span><span class="sxs-lookup"><span data-stu-id="be267-105">Load data on demand into a cache from a data store.</span></span> <span data-ttu-id="be267-106">Este patrón puede mejorar el rendimiento y también ayuda a mantener la coherencia entre los datos contenidos en la caché y los datos del almacén de datos subyacente.</span><span class="sxs-lookup"><span data-stu-id="be267-106">This can improve performance and also helps to maintain consistency between data held in the cache and data in the underlying data store.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="be267-107">Contexto y problema</span><span class="sxs-lookup"><span data-stu-id="be267-107">Context and problem</span></span>

<span data-ttu-id="be267-108">Las aplicaciones usan una caché para mejorar el acceso repetido a la información contenida en un almacén de datos.</span><span class="sxs-lookup"><span data-stu-id="be267-108">Applications use a cache to improve repeated access to information held in a data store.</span></span> <span data-ttu-id="be267-109">Sin embargo, esperar que los datos almacenados en caché sean siempre coherentes con los datos del almacén de datos no resulta práctico.</span><span class="sxs-lookup"><span data-stu-id="be267-109">However, it's impractical to expect that cached data will always be completely consistent with the data in the data store.</span></span> <span data-ttu-id="be267-110">Las aplicaciones deben implementar una estrategia que ayude a garantizar que los datos de la caché estén lo más actualizados posible, pero que también pueda detectar y tratar situaciones que surgen cuando los datos de la caché se han vuelto obsoletos.</span><span class="sxs-lookup"><span data-stu-id="be267-110">Applications should implement a strategy that helps to ensure that the data in the cache is as up-to-date as possible, but can also detect and handle situations that arise when the data in the cache has become stale.</span></span>

## <a name="solution"></a><span data-ttu-id="be267-111">Solución</span><span class="sxs-lookup"><span data-stu-id="be267-111">Solution</span></span>

<span data-ttu-id="be267-112">Muchos sistemas comerciales de almacenamiento en caché proporcionan operaciones de lectura simultánea, escritura simultánea y escritura por detrás.</span><span class="sxs-lookup"><span data-stu-id="be267-112">Many commercial caching systems provide read-through and write-through/write-behind operations.</span></span> <span data-ttu-id="be267-113">En estos sistemas, una aplicación recupera datos mediante la referencia a la caché.</span><span class="sxs-lookup"><span data-stu-id="be267-113">In these systems, an application retrieves data by referencing the cache.</span></span> <span data-ttu-id="be267-114">Si los datos no se están en la caché, se recuperan del almacén de datos y se agregan a la caché.</span><span class="sxs-lookup"><span data-stu-id="be267-114">If the data isn't in the cache, it's retrieved from the data store and added to the cache.</span></span> <span data-ttu-id="be267-115">Las modificaciones en los datos contenidos en la caché se reescriben automáticamente en el almacén de datos.</span><span class="sxs-lookup"><span data-stu-id="be267-115">Any modifications to data held in the cache are automatically written back to the data store as well.</span></span>

<span data-ttu-id="be267-116">Si hay memorias caché que no proporcionen esta funcionalidad, es responsabilidad de las aplicaciones que usan la caché mantener los datos.</span><span class="sxs-lookup"><span data-stu-id="be267-116">For caches that don't provide this functionality, it's the responsibility of the applications that use the cache to maintain the data.</span></span>

<span data-ttu-id="be267-117">Una aplicación puede emular la funcionalidad de almacenamiento en caché de lectura simultánea mediante la implementación de la estrategia de reserva de caché.</span><span class="sxs-lookup"><span data-stu-id="be267-117">An application can emulate the functionality of read-through caching by implementing the cache-aside strategy.</span></span> <span data-ttu-id="be267-118">Esta estrategia carga datos en la caché a petición.</span><span class="sxs-lookup"><span data-stu-id="be267-118">This strategy loads data into the cache on demand.</span></span> <span data-ttu-id="be267-119">En la ilustración se muestra el uso del patrón Cache-Aside para almacenar datos en la caché.</span><span class="sxs-lookup"><span data-stu-id="be267-119">The figure illustrates using the Cache-Aside pattern to store data in the cache.</span></span>

![Uso del patrón Cache-Aside para almacenar los datos en la caché](./_images/cache-aside-diagram.png)


<span data-ttu-id="be267-121">Si una aplicación actualiza la información, puede seguir la estrategia de escritura simultánea mediante la modificación del almacén de datos y la invalidación del elemento correspondiente en la caché.</span><span class="sxs-lookup"><span data-stu-id="be267-121">If an application updates information, it can follow the write-through strategy by making the modification to the data store, and by invalidating the corresponding item in the cache.</span></span>

<span data-ttu-id="be267-122">Cuando luego se necesite el elemento, el uso de la estrategia de reserva de caché provocará que los datos actualizados se recuperen del almacén de datos y se agreguen de nuevo a la caché.</span><span class="sxs-lookup"><span data-stu-id="be267-122">When the item is next required, using the cache-aside strategy will cause the updated data to be retrieved from the data store and added back into the cache.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="be267-123">Problemas y consideraciones</span><span class="sxs-lookup"><span data-stu-id="be267-123">Issues and considerations</span></span>

<span data-ttu-id="be267-124">Tenga en cuenta los puntos siguientes al decidir cómo implementar este patrón:</span><span class="sxs-lookup"><span data-stu-id="be267-124">Consider the following points when deciding how to implement this pattern:</span></span> 

<span data-ttu-id="be267-125">**Duración de los datos almacenados en caché**.</span><span class="sxs-lookup"><span data-stu-id="be267-125">**Lifetime of cached data**.</span></span> <span data-ttu-id="be267-126">Muchas de las memorias caché implementan una directiva de expiración que invalida los datos y los quita de la caché si no se accede a ellos durante un período especificado.</span><span class="sxs-lookup"><span data-stu-id="be267-126">Many caches implement an expiration policy that invalidates data and removes it from the cache if it's not accessed for a specified period.</span></span> <span data-ttu-id="be267-127">Para que la reserva de caché sea eficaz, asegúrese de que la directiva de expiración coincida con el patrón de acceso en las aplicaciones que usan los datos.</span><span class="sxs-lookup"><span data-stu-id="be267-127">For cache-aside to be effective, ensure that the expiration policy matches the pattern of access for applications that use the data.</span></span> <span data-ttu-id="be267-128">No conviene que el período de expiración sea demasiado corto, ya que puede provocar que las aplicaciones recuperen los datos continuamente del almacén de datos y los agreguen a la caché.</span><span class="sxs-lookup"><span data-stu-id="be267-128">Don't make the expiration period too short because this can cause applications to continually retrieve data from the data store and add it to the cache.</span></span> <span data-ttu-id="be267-129">De igual forma, tampoco debe ser tan largo que los datos almacenados en caché puedan volverse obsoletos.</span><span class="sxs-lookup"><span data-stu-id="be267-129">Similarly, don't make the expiration period so long that the cached data is likely to become stale.</span></span> <span data-ttu-id="be267-130">Recuerde que el almacenamiento en caché es más eficaz con datos relativamente estáticos o datos que se leen frecuentemente.</span><span class="sxs-lookup"><span data-stu-id="be267-130">Remember that caching is most effective for relatively static data, or data that is read frequently.</span></span>

<span data-ttu-id="be267-131">**Expulsión de los datos**.</span><span class="sxs-lookup"><span data-stu-id="be267-131">**Evicting data**.</span></span> <span data-ttu-id="be267-132">La mayoría de las memorias caché tienen un tamaño limitado en comparación con el almacén de datos donde se originan los datos, por lo que si es necesario expulsarán los datos.</span><span class="sxs-lookup"><span data-stu-id="be267-132">Most caches have a limited size compared to the data store where the data originates, and they'll evict data if necessary.</span></span> <span data-ttu-id="be267-133">La mayoría de las memorias caché adoptan una directiva de uso reciente para seleccionar los elementos que se expulsarán, si bien podría ser personalizable.</span><span class="sxs-lookup"><span data-stu-id="be267-133">Most caches adopt a least-recently-used policy for selecting items to evict, but this might be customizable.</span></span> <span data-ttu-id="be267-134">Configure la propiedad de expiración global y otras propiedades de la caché, y la propiedad de expiración de cada elemento almacenado en caché, para asegurarse de que la caché le resulte rentable.</span><span class="sxs-lookup"><span data-stu-id="be267-134">Configure the global expiration property and other properties of the cache, and the expiration property of each cached item, to ensure that the cache is cost effective.</span></span> <span data-ttu-id="be267-135">No siempre es adecuado aplicar una directiva de expulsión global para todos los elementos de la caché.</span><span class="sxs-lookup"><span data-stu-id="be267-135">It isn't always appropriate to apply a global eviction policy to every item in the cache.</span></span> <span data-ttu-id="be267-136">Por ejemplo, si un elemento almacenado en caché es caro de recuperar del almacén de datos, puede ser ventajoso mantener este elemento en la caché a costa de elementos de acceso más frecuente pero menos costosos.</span><span class="sxs-lookup"><span data-stu-id="be267-136">For example, if a cached item is very expensive to retrieve from the data store, it can be beneficial to keep this item in the cache at the expense of more frequently accessed but less costly items.</span></span>

<span data-ttu-id="be267-137">**Desbloqueo de la memoria caché**.</span><span class="sxs-lookup"><span data-stu-id="be267-137">**Priming the cache**.</span></span> <span data-ttu-id="be267-138">Muchas soluciones rellenan previamente la caché con los datos que es probable que necesite una aplicación como parte del procesamiento de inicio.</span><span class="sxs-lookup"><span data-stu-id="be267-138">Many solutions prepopulate the cache with the data that an application is likely to need as part of the startup processing.</span></span> <span data-ttu-id="be267-139">El patrón Cache-Aside puede resultarle útil si algunos de estos datos expiran o se expulsan.</span><span class="sxs-lookup"><span data-stu-id="be267-139">The Cache-Aside pattern can still be useful if some of this data expires or is evicted.</span></span>

<span data-ttu-id="be267-140">**Coherencia**</span><span class="sxs-lookup"><span data-stu-id="be267-140">**Consistency**.</span></span> <span data-ttu-id="be267-141">La implementación del patrón Cache-Aside no garantiza la coherencia entre el almacén de datos y la caché.</span><span class="sxs-lookup"><span data-stu-id="be267-141">Implementing the Cache-Aside pattern doesn't guarantee consistency between the data store and the cache.</span></span> <span data-ttu-id="be267-142">En cualquier momento, un proceso externo puede cambiar un elemento del almacén de datos, y este cambio podría no reflejarse en la caché hasta la próxima vez que se cargue el elemento.</span><span class="sxs-lookup"><span data-stu-id="be267-142">An item in the data store can be changed at any time by an external process, and this change might not be reflected in the cache until the next time the item is loaded.</span></span> <span data-ttu-id="be267-143">En un sistema que replica datos entre almacenes de datos, este problema puede convertirse en grave si la sincronización se produce con frecuencia.</span><span class="sxs-lookup"><span data-stu-id="be267-143">In a system that replicates data across data stores, this problem can become serious if synchronization occurs frequently.</span></span>

<span data-ttu-id="be267-144">**Almacenamiento en caché local (en memoria)**.</span><span class="sxs-lookup"><span data-stu-id="be267-144">**Local (in-memory) caching**.</span></span> <span data-ttu-id="be267-145">Un caché podría ser local para una instancia de la aplicación y almacenarse en memoria.</span><span class="sxs-lookup"><span data-stu-id="be267-145">A cache could be local to an application instance and stored in-memory.</span></span> <span data-ttu-id="be267-146">La reserva de memoria puede ser útil en este entorno si una aplicación accede de forma repetida a los mismos datos.</span><span class="sxs-lookup"><span data-stu-id="be267-146">Cache-aside can be useful in this environment if an application repeatedly accesses the same data.</span></span> <span data-ttu-id="be267-147">Sin embargo, una caché local es privada y, por lo tanto, diferentes instancias de aplicación podrían tener una copia cada una de los mismos datos en caché.</span><span class="sxs-lookup"><span data-stu-id="be267-147">However, a local cache is private and so different application instances could each have a copy of the same cached data.</span></span> <span data-ttu-id="be267-148">Estos datos podrían volverse enseguida incoherentes entre las memorias caché, por lo que podría ser necesario hacer que expiren los datos mantenidos en la caché privada y actualizarlos con una mayor frecuencia.</span><span class="sxs-lookup"><span data-stu-id="be267-148">This data could quickly become inconsistent between caches, so it might be necessary to expire data held in a private cache and refresh it more frequently.</span></span> <span data-ttu-id="be267-149">En estos casos, considere la posibilidad de investigar el uso de un mecanismo de almacenamiento en caché compartido o distribuido.</span><span class="sxs-lookup"><span data-stu-id="be267-149">In these scenarios, consider investigating the use of a shared or a distributed caching mechanism.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="be267-150">Cuándo usar este patrón</span><span class="sxs-lookup"><span data-stu-id="be267-150">When to use this pattern</span></span>

<span data-ttu-id="be267-151">Use este patrón en los siguientes supuestos:</span><span class="sxs-lookup"><span data-stu-id="be267-151">Use this pattern when:</span></span>

- <span data-ttu-id="be267-152">Una caché no proporcione operaciones nativas de lectura y escritura simultáneas.</span><span class="sxs-lookup"><span data-stu-id="be267-152">A cache doesn't provide native read-through and write-through operations.</span></span>
- <span data-ttu-id="be267-153">La demanda de recursos sea impredecible.</span><span class="sxs-lookup"><span data-stu-id="be267-153">Resource demand is unpredictable.</span></span> <span data-ttu-id="be267-154">Este patrón permite que las aplicaciones carguen datos a petición.</span><span class="sxs-lookup"><span data-stu-id="be267-154">This pattern enables applications to load data on demand.</span></span> <span data-ttu-id="be267-155">No realiza ninguna suposición de qué datos necesitará una aplicación de antemano.</span><span class="sxs-lookup"><span data-stu-id="be267-155">It makes no assumptions about which data an application will require in advance.</span></span>

<span data-ttu-id="be267-156">Este patrón podría no ser útil en los siguientes casos:</span><span class="sxs-lookup"><span data-stu-id="be267-156">This pattern might not be suitable:</span></span>

- <span data-ttu-id="be267-157">Cuando el conjunto de datos almacenado en caché sea estático.</span><span class="sxs-lookup"><span data-stu-id="be267-157">When the cached data set is static.</span></span> <span data-ttu-id="be267-158">Si los datos caben en el espacio disponible en la caché, desbloquee la caché con los datos al inicio y aplique una directiva que impida que los datos expiren.</span><span class="sxs-lookup"><span data-stu-id="be267-158">If the data will fit into the available cache space, prime the cache with the data on startup and apply a policy that prevents the data from expiring.</span></span>
- <span data-ttu-id="be267-159">Para almacenar en caché información del estado de sesión de una aplicación web hospedada en una granja de servidores web.</span><span class="sxs-lookup"><span data-stu-id="be267-159">For caching session state information in a web application hosted in a web farm.</span></span> <span data-ttu-id="be267-160">En este entorno, debe evitar introducir dependencias basadas en la afinidad cliente-servidor.</span><span class="sxs-lookup"><span data-stu-id="be267-160">In this environment, you should avoid introducing dependencies based on client-server affinity.</span></span>

## <a name="example"></a><span data-ttu-id="be267-161">Ejemplo</span><span class="sxs-lookup"><span data-stu-id="be267-161">Example</span></span>

<span data-ttu-id="be267-162">En Microsoft Azure puede usar Azure Redis Cache para crear una caché distribuida que se pueda compartir entre varias instancias de una aplicación.</span><span class="sxs-lookup"><span data-stu-id="be267-162">In Microsoft Azure you can use Azure Redis Cache to create a distributed cache that can be shared by multiple instances of an application.</span></span> 

<span data-ttu-id="be267-163">Los siguientes ejemplos de código usan el cliente [StackExchange.Redis], que es una biblioteca cliente de Redis escrita para. NET.</span><span class="sxs-lookup"><span data-stu-id="be267-163">This following code examples use the [StackExchange.Redis] client, which is a Redis client library written for .NET.</span></span> <span data-ttu-id="be267-164">Para conectarse a una instancia de Azure Redis Cache, llame a método estático `ConnectionMultiplexer.Connect` y pase la cadena de conexión.</span><span class="sxs-lookup"><span data-stu-id="be267-164">To connect to an Azure Redis Cache instance, call the static `ConnectionMultiplexer.Connect` method and pass in the connection string.</span></span> <span data-ttu-id="be267-165">El método devuelve un elemento `ConnectionMultiplexer` que representa la conexión.</span><span class="sxs-lookup"><span data-stu-id="be267-165">The method returns a `ConnectionMultiplexer` that represents the connection.</span></span> <span data-ttu-id="be267-166">Un enfoque para compartir una instancia de `ConnectionMultiplexer` en su aplicación es tener una propiedad estática que devuelva una instancia conectada, como en el ejemplo siguiente.</span><span class="sxs-lookup"><span data-stu-id="be267-166">One approach to sharing a `ConnectionMultiplexer` instance in your application is to have a static property that returns a connected instance, similar to the following example.</span></span> <span data-ttu-id="be267-167">Este enfoque proporciona una manera segura para subprocesos de inicializar una sola instancia conectada.</span><span class="sxs-lookup"><span data-stu-id="be267-167">This approach provides a thread-safe way to initialize only a single connected instance.</span></span>

```csharp
private static ConnectionMultiplexer Connection;

// Redis Connection string info
private static Lazy<ConnectionMultiplexer> lazyConnection = new Lazy<ConnectionMultiplexer>(() =>
{
    string cacheConnection = ConfigurationManager.AppSettings["CacheConnection"].ToString();
    return ConnectionMultiplexer.Connect(cacheConnection);
});

public static ConnectionMultiplexer Connection => lazyConnection.Value;
```

<span data-ttu-id="be267-168">El método `GetMyEntityAsync` del código de ejemplo siguiente muestra una implementación del patrón Cache-Aside.</span><span class="sxs-lookup"><span data-stu-id="be267-168">The `GetMyEntityAsync` method in the following code example shows an implementation of the Cache-Aside pattern.</span></span> <span data-ttu-id="be267-169">Este método recupera un objeto de la caché mediante el enfoque de lectura simultánea.</span><span class="sxs-lookup"><span data-stu-id="be267-169">This method retrieves an object from the cache using the read-through approach.</span></span>

<span data-ttu-id="be267-170">Un objeto se identifica mediante un identificador de número entero como clave.</span><span class="sxs-lookup"><span data-stu-id="be267-170">An object is identified by using an integer ID as the key.</span></span> <span data-ttu-id="be267-171">El método `GetMyEntityAsync` intenta recuperar un elemento con esta clave de la caché.</span><span class="sxs-lookup"><span data-stu-id="be267-171">The `GetMyEntityAsync` method tries to retrieve an item with this key from the cache.</span></span> <span data-ttu-id="be267-172">Si se encuentra un elemento coincidente, se devuelve.</span><span class="sxs-lookup"><span data-stu-id="be267-172">If a matching item is found, it's returned.</span></span> <span data-ttu-id="be267-173">Si no hay ninguna coincidencia en la caché, el método `GetMyEntityAsync` recupera el objeto de un almacén de datos, lo agrega a la caché y luego lo devuelve.</span><span class="sxs-lookup"><span data-stu-id="be267-173">If there's no match in the cache, the `GetMyEntityAsync` method retrieves the object from a data store, adds it to the cache, and then returns it.</span></span> <span data-ttu-id="be267-174">El código que realmente lee los datos del almacén de datos no se muestra aquí, ya que depende del almacén de datos.</span><span class="sxs-lookup"><span data-stu-id="be267-174">The code that actually reads the data from the data store is not shown here, because it depends on the data store.</span></span> <span data-ttu-id="be267-175">Tenga en cuenta que el elemento en caché está configurado para que expire a fin de evitar que se vuelva obsoleto si se actualiza en otro lugar.</span><span class="sxs-lookup"><span data-stu-id="be267-175">Note that the cached item is configured to expire to prevent it from becoming stale if it's updated elsewhere.</span></span>


```csharp
// Set five minute expiration as a default
private const double DefaultExpirationTimeInMinutes = 5.0;

public async Task<MyEntity> GetMyEntityAsync(int id)
{
  // Define a unique key for this method and its parameters.
  var key = $"MyEntity:{id}";
  var cache = Connection.GetDatabase();
  
  // Try to get the entity from the cache.
  var json = await cache.StringGetAsync(key).ConfigureAwait(false);
  var value = string.IsNullOrWhiteSpace(json) 
                ? default(MyEntity) 
                : JsonConvert.DeserializeObject<MyEntity>(json);
  
  if (value == null) // Cache miss
  {
    // If there's a cache miss, get the entity from the original store and cache it.
    // Code has been omitted because it's data store dependent.  
    value = ...;

    // Avoid caching a null value.
    if (value != null)
    {
      // Put the item in the cache with a custom expiration time that 
      // depends on how critical it is to have stale data.
      await cache.StringSetAsync(key, JsonConvert.SerializeObject(value)).ConfigureAwait(false);
      await cache.KeyExpireAsync(key, TimeSpan.FromMinutes(DefaultExpirationTimeInMinutes)).ConfigureAwait(false);
    }
  }

  return value;
}
```

>  <span data-ttu-id="be267-176">En los ejemplos se usa Redis Cache para acceder al almacén y recuperar información de la caché.</span><span class="sxs-lookup"><span data-stu-id="be267-176">The examples use Redis Cache to access the store and retrieve information from the cache.</span></span> <span data-ttu-id="be267-177">Para más información, consulte [Uso de Microsoft Azure Redis Cache](https://docs.microsoft.com/azure/redis-cache/cache-dotnet-how-to-use-azure-redis-cache) y [Creación de una aplicación web con Caché en Redis](https://docs.microsoft.com/azure/redis-cache/cache-web-app-howto).</span><span class="sxs-lookup"><span data-stu-id="be267-177">For more information, see [Using Microsoft Azure Redis Cache](https://docs.microsoft.com/azure/redis-cache/cache-dotnet-how-to-use-azure-redis-cache) and [How to create a Web App with Redis Cache](https://docs.microsoft.com/azure/redis-cache/cache-web-app-howto)</span></span>

<span data-ttu-id="be267-178">El método `UpdateEntityAsync` que se muestra a continuación ilustra cómo invalidar un objeto en la caché cuando la aplicación cambia su valor.</span><span class="sxs-lookup"><span data-stu-id="be267-178">The `UpdateEntityAsync` method shown below demonstrates how to invalidate an object in the cache when the value is changed by the application.</span></span> <span data-ttu-id="be267-179">El código actualiza el almacén de datos original y, a continuación, quita el elemento en caché de la caché.</span><span class="sxs-lookup"><span data-stu-id="be267-179">The code updates the original data store and then removes the cached item from the cache.</span></span>

```csharp
public async Task UpdateEntityAsync(MyEntity entity)
{
    // Update the object in the original data store.
    await this.store.UpdateEntityAsync(entity).ConfigureAwait(false); 

    // Invalidate the current cache object.
    var cache = Connection.GetDatabase();
    var id = entity.Id;
    var key = $"MyEntity:{id}"; // The key for the cached object.
    await cache.KeyDeleteAsync(key).ConfigureAwait(false); // Delete this key from the cache.
}
```

> [!NOTE]
> <span data-ttu-id="be267-180">Es importante el orden de los pasos.</span><span class="sxs-lookup"><span data-stu-id="be267-180">The order of the steps is important.</span></span> <span data-ttu-id="be267-181">Actualice el almacén de datos *antes* de quitar el elemento de la caché.</span><span class="sxs-lookup"><span data-stu-id="be267-181">Update the data store *before* removing the item from the cache.</span></span> <span data-ttu-id="be267-182">Si quita primero el elemento en caché, hay una pequeña ventana de tiempo en que un cliente podría recuperar el elemento antes de que se actualice el almacén de datos.</span><span class="sxs-lookup"><span data-stu-id="be267-182">If you remove the cached item first, there is a small window of time when a client might fetch the item before the data store is updated.</span></span> <span data-ttu-id="be267-183">Como resultado, se producirá un error de la caché (porque el elemento se ha quitado de ella), que hará que la versión anterior del elemento se recupere del almacén de datos y se vuelva a agregar a la caché.</span><span class="sxs-lookup"><span data-stu-id="be267-183">That will result in a cache miss (because the item was removed from the cache), causing the earlier version of the item to be fetched from the data store and added back into the cache.</span></span> <span data-ttu-id="be267-184">Los datos de la caché, por lo tanto, estarán obsoletos.</span><span class="sxs-lookup"><span data-stu-id="be267-184">The result will be stale cache data.</span></span>


## <a name="related-guidance"></a><span data-ttu-id="be267-185">Instrucciones relacionadas</span><span class="sxs-lookup"><span data-stu-id="be267-185">Related guidance</span></span> 

<span data-ttu-id="be267-186">La siguiente información puede resultarle de interés al implementar este patrón:</span><span class="sxs-lookup"><span data-stu-id="be267-186">The following information may be relevant when implementing this pattern:</span></span>

- <span data-ttu-id="be267-187">[Guía sobre el almacenamiento en caché](https://docs.microsoft.com/azure/architecture/best-practices/caching).</span><span class="sxs-lookup"><span data-stu-id="be267-187">[Caching Guidance](https://docs.microsoft.com/azure/architecture/best-practices/caching).</span></span> <span data-ttu-id="be267-188">Proporciona información adicional sobre cómo se pueden almacenar en caché los datos de una solución de nube y los problemas que se deben considerar al implementar una caché.</span><span class="sxs-lookup"><span data-stu-id="be267-188">Provides additional information on how you can cache data in a cloud solution, and the issues that you should consider when you implement a cache.</span></span>

- <span data-ttu-id="be267-189">[Data Consistency Primer](https://msdn.microsoft.com/library/dn589800.aspx) (Manual básico de coherencia de datos).</span><span class="sxs-lookup"><span data-stu-id="be267-189">[Data Consistency Primer](https://msdn.microsoft.com/library/dn589800.aspx).</span></span> <span data-ttu-id="be267-190">Las aplicaciones de nube usan normalmente datos que se distribuyen entre los almacenes de datos.</span><span class="sxs-lookup"><span data-stu-id="be267-190">Cloud applications typically use data that's spread across data stores.</span></span> <span data-ttu-id="be267-191">Administrar y mantener la coherencia de los datos en este entorno son un aspecto fundamental del sistema, en especial por los problemas de simultaneidad y disponibilidad que puedan surgir.</span><span class="sxs-lookup"><span data-stu-id="be267-191">Managing and maintaining data consistency in this environment is a critical aspect of the system, particularly the concurrency and availability issues that can arise.</span></span> <span data-ttu-id="be267-192">En este manual básico se describen los problemas de coherencia entre los datos distribuidos y se resume cómo una aplicación puede implementar coherencia definitiva para mantener la disponibilidad de datos.</span><span class="sxs-lookup"><span data-stu-id="be267-192">This primer describes issues about consistency across distributed data, and summarizes how an application can implement eventual consistency to maintain the availability of data.</span></span>


[StackExchange.Redis]: https://github.com/StackExchange/StackExchange.Redis
