---
title: Cache-Aside
description: Carga datos a petición en una memoria caché desde un almacén de datos
keywords: Patrón de diseño
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- data-management
- performance-scalability
ms.openlocfilehash: 1536a33884c9c9faa1e3702c951067249e691bf8
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/23/2018
---
# <a name="cache-aside-pattern"></a>Patrón Cache-Aside

[!INCLUDE [header](../_includes/header.md)]

Carga datos a petición en una caché desde un almacén de datos Este patrón puede mejorar el rendimiento y también ayuda a mantener la coherencia entre los datos contenidos en la caché y los datos del almacén de datos subyacente.

## <a name="context-and-problem"></a>Contexto y problema

Las aplicaciones usan una caché para mejorar el acceso repetido a la información contenida en un almacén de datos. Sin embargo, esperar que los datos almacenados en caché sean siempre coherentes con los datos del almacén de datos no resulta práctico. Las aplicaciones deben implementar una estrategia que ayude a garantizar que los datos de la caché estén lo más actualizados posible, pero que también pueda detectar y tratar situaciones que surgen cuando los datos de la caché se han vuelto obsoletos.

## <a name="solution"></a>Solución

Muchos sistemas comerciales de almacenamiento en caché proporcionan operaciones de lectura simultánea, escritura simultánea y escritura por detrás. En estos sistemas, una aplicación recupera datos mediante la referencia a la caché. Si los datos no se están en la caché, se recuperan del almacén de datos y se agregan a la caché. Las modificaciones en los datos contenidos en la caché se reescriben automáticamente en el almacén de datos.

Si hay memorias caché que no proporcionen esta funcionalidad, es responsabilidad de las aplicaciones que usan la caché mantener los datos.

Una aplicación puede emular la funcionalidad de almacenamiento en caché de lectura simultánea mediante la implementación de la estrategia de reserva de caché. Esta estrategia carga datos en la caché a petición. En la ilustración se muestra el uso del patrón Cache-Aside para almacenar datos en la caché.

![Uso del patrón Cache-Aside para almacenar los datos en la caché](./_images/cache-aside-diagram.png)


Si una aplicación actualiza la información, puede seguir la estrategia de escritura simultánea mediante la modificación del almacén de datos y la invalidación del elemento correspondiente en la caché.

Cuando luego se necesite el elemento, el uso de la estrategia de reserva de caché provocará que los datos actualizados se recuperen del almacén de datos y se agreguen de nuevo a la caché.

## <a name="issues-and-considerations"></a>Problemas y consideraciones

Tenga en cuenta los puntos siguientes al decidir cómo implementar este patrón: 

**Duración de los datos almacenados en caché**. Muchas de las memorias caché implementan una directiva de expiración que invalida los datos y los quita de la caché si no se accede a ellos durante un período especificado. Para que la reserva de caché sea eficaz, asegúrese de que la directiva de expiración coincida con el patrón de acceso en las aplicaciones que usan los datos. No conviene que el período de expiración sea demasiado corto, ya que puede provocar que las aplicaciones recuperen los datos continuamente del almacén de datos y los agreguen a la caché. De igual forma, tampoco debe ser tan largo que los datos almacenados en caché puedan volverse obsoletos. Recuerde que el almacenamiento en caché es más eficaz con datos relativamente estáticos o datos que se leen frecuentemente.

**Expulsión de los datos**. La mayoría de las memorias caché tienen un tamaño limitado en comparación con el almacén de datos donde se originan los datos, por lo que si es necesario expulsarán los datos. La mayoría de las memorias caché adoptan una directiva de uso reciente para seleccionar los elementos que se expulsarán, si bien podría ser personalizable. Configure la propiedad de expiración global y otras propiedades de la caché, y la propiedad de expiración de cada elemento almacenado en caché, para asegurarse de que la caché le resulte rentable. No siempre es adecuado aplicar una directiva de expulsión global para todos los elementos de la caché. Por ejemplo, si un elemento almacenado en caché es caro de recuperar del almacén de datos, puede ser ventajoso mantener este elemento en la caché a costa de elementos de acceso más frecuente pero menos costosos.

**Desbloqueo de la memoria caché**. Muchas soluciones rellenan previamente la caché con los datos que es probable que necesite una aplicación como parte del procesamiento de inicio. El patrón Cache-Aside puede resultarle útil si algunos de estos datos expiran o se expulsan.

**Coherencia** La implementación del patrón Cache-Aside no garantiza la coherencia entre el almacén de datos y la caché. En cualquier momento, un proceso externo puede cambiar un elemento del almacén de datos, y este cambio podría no reflejarse en la caché hasta la próxima vez que se cargue el elemento. En un sistema que replica datos entre almacenes de datos, este problema puede convertirse en grave si la sincronización se produce con frecuencia.

**Almacenamiento en caché local (en memoria)**. Un caché podría ser local para una instancia de la aplicación y almacenarse en memoria. La reserva de memoria puede ser útil en este entorno si una aplicación accede de forma repetida a los mismos datos. Sin embargo, una caché local es privada y, por lo tanto, diferentes instancias de aplicación podrían tener una copia cada una de los mismos datos en caché. Estos datos podrían volverse enseguida incoherentes entre las memorias caché, por lo que podría ser necesario hacer que expiren los datos mantenidos en la caché privada y actualizarlos con una mayor frecuencia. En estos casos, considere la posibilidad de investigar el uso de un mecanismo de almacenamiento en caché compartido o distribuido.

## <a name="when-to-use-this-pattern"></a>Cuándo usar este patrón

Use este patrón en los siguientes supuestos:

- Una caché no proporcione operaciones nativas de lectura y escritura simultáneas.
- La demanda de recursos sea impredecible. Este patrón permite que las aplicaciones carguen datos a petición. No realiza ninguna suposición de qué datos necesitará una aplicación de antemano.

Este patrón podría no ser útil en los siguientes casos:

- Cuando el conjunto de datos almacenado en caché sea estático. Si los datos caben en el espacio disponible en la caché, desbloquee la caché con los datos al inicio y aplique una directiva que impida que los datos expiren.
- Para almacenar en caché información del estado de sesión de una aplicación web hospedada en una granja de servidores web. En este entorno, debe evitar introducir dependencias basadas en la afinidad cliente-servidor.

## <a name="example"></a>Ejemplo

En Microsoft Azure puede usar Azure Redis Cache para crear una caché distribuida que se pueda compartir entre varias instancias de una aplicación. 

Para conectarse a una instancia de Azure Redis Cache, llame a método estático `Connect` y pase la cadena de conexión. El método devuelve un elemento `ConnectionMultiplexer` que representa la conexión. Un enfoque para compartir una instancia de `ConnectionMultiplexer` en su aplicación es tener una propiedad estática que devuelva una instancia conectada, como en el ejemplo siguiente. Este enfoque proporciona una manera segura para subprocesos de inicializar una sola instancia conectada.

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

El método `GetMyEntityAsync` del código de ejemplo siguiente muestra una implementación del patrón Cache-Aside basada en Azure Redis Cache. Este método recupera un objeto de la caché mediante el enfoque de lectura simultánea.

Un objeto se identifica mediante un identificador de número entero como clave. El método `GetMyEntityAsync` intenta recuperar un elemento con esta clave de la caché. Si se encuentra un elemento coincidente, se devuelve. Si no hay ninguna coincidencia en la caché, el método `GetMyEntityAsync` recupera el objeto de un almacén de datos, lo agrega a la caché y luego lo devuelve. El código que realmente lee los datos del almacén de datos no se muestra aquí, ya que depende del almacén de datos. Tenga en cuenta que el elemento en caché está configurado para que expire a fin de evitar que se vuelva obsoleto si se actualiza en otro lugar.


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

>  En los ejemplos se usa la API de Azure Redis Cache para acceder al almacén y recuperar información de la caché. Para más información, consulte [Uso de Microsoft Azure Redis Cache](https://docs.microsoft.com/azure/redis-cache/cache-dotnet-how-to-use-azure-redis-cache) y [Creación de una aplicación web con Caché en Redis](https://docs.microsoft.com/azure/redis-cache/cache-web-app-howto).

El método `UpdateEntityAsync` que se muestra a continuación ilustra cómo invalidar un objeto en la caché cuando la aplicación cambia su valor. El código actualiza el almacén de datos original y, a continuación, quita el elemento en caché de la caché.

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
> Es importante el orden de los pasos. Actualice el almacén de datos *antes* de quitar el elemento de la caché. Si quita primero el elemento en caché, hay una pequeña ventana de tiempo en que un cliente podría recuperar el elemento antes de que se actualice el almacén de datos. Como resultado, se producirá un error de la caché (porque el elemento se ha quitado de ella), que hará que la versión anterior del elemento se recupere del almacén de datos y se vuelva a agregar a la caché. Los datos de la caché, por lo tanto, estarán obsoletos.


## <a name="related-guidance"></a>Instrucciones relacionadas 

La siguiente información puede resultarle de interés al implementar este patrón:

- [Guía sobre el almacenamiento en caché](https://docs.microsoft.com/azure/architecture/best-practices/caching). Proporciona información adicional sobre cómo se pueden almacenar en caché los datos de una solución de nube y los problemas que se deben considerar al implementar una caché.

- [Data Consistency Primer](https://msdn.microsoft.com/library/dn589800.aspx) (Manual básico de coherencia de datos). Las aplicaciones de nube usan normalmente datos que se distribuyen entre los almacenes de datos. Administrar y mantener la coherencia de los datos en este entorno son un aspecto fundamental del sistema, en especial por los problemas de simultaneidad y disponibilidad que puedan surgir. En este manual básico se describen los problemas de coherencia entre los datos distribuidos y se resume cómo una aplicación puede implementar coherencia definitiva para mantener la disponibilidad de datos.
