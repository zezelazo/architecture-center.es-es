---
title: Antipatrón No Caching
titleSuffix: Performance antipatterns for cloud apps
description: Si se capturan los mismos datos varias veces, puede reducirse el rendimiento la y escalabilidad.
author: dragon119
ms.date: 06/05/2017
ms.custom: seodec18
ms.openlocfilehash: c2a5cdbb8863f87b8928558c8237e8659032ac32
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/04/2019
ms.locfileid: "54011606"
---
# <a name="no-caching-antipattern"></a>Antipatrón No Caching

En una aplicación en la nube que controla muchas solicitudes simultáneas, capturar repetidamente los mismos datos puede reducir el rendimiento y la escalabilidad.

## <a name="problem-description"></a>Descripción del problema

Cuando los datos no están almacenados en caché, puede ocasionar diversos comportamientos no deseados, como son:

- Capturar repetidamente la misma información de un recurso cuyo acceso resulta costoso, en cuanto a la sobrecarga de E/S o la latencia.
- Construir varias veces los mismos objetos o estructuras de datos para varias solicitudes.
- Un exceso de llamadas a un servicio remoto que tiene una cuota de servicio y que limita a los clientes más allá de un determinado período.

A su vez, estos problemas pueden provocar tiempos de respuesta insuficientes, mayor contención en el almacén de datos y una escalabilidad escasa.

En el ejemplo siguiente se utiliza Entity Framework para conectarse a una base de datos. Cada solicitud de cliente produce una llamada a la base de datos, incluso si varias solicitudes recuperan exactamente los mismos datos. El costo de las solicitudes repetidas, en cuanto a los cargos por el acceso a los datos y la sobrecarga de E/S, puede acumularse rápidamente.

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

Puede encontrar el ejemplo completo [aquí][sample-app].

Este antipatrón suele ocurrir porque:

- No usar una memoria caché es más fácil de implementar y funciona bien con cargas bajas. El almacenamiento en caché hace que el código sea más complicado.
- No se entienden con claridad las ventajas y desventajas del uso del almacenamiento en caché.
- Preocupa la sobrecarga que supone mantener la precisión y la actualización de los datos almacenados en caché.
- Una aplicación se migró desde un sistema local, en el que la latencia de red no era un problema, y el sistema se ejecutaba en un costoso hardware de alto rendimiento, por lo que el almacenamiento en caché no se consideró en el diseño original.
- Los desarrolladores no son conscientes de que el almacenamiento en caché es una posibilidad en un escenario determinado. Por ejemplo, no pueden considerar usar ETags al implementar una API de web.

## <a name="how-to-fix-the-problem"></a>Procedimiento para corregir el problema

La estrategia de almacenamiento en caché más popular es el sistema *a petición* o el modelo *cache-aside*.

- En las lecturas, la aplicación intenta leer los datos de la memoria caché. Si los datos no están allí, la aplicación los recupera del origen de datos y los agrega.
- Durante las escrituras, la aplicación escribe el cambio directamente en el origen de datos y quita el valor antiguo de la memoria caché. Se recuperará y se agregará a la memoria caché la próxima vez que se requiera.

Este enfoque es adecuado para los datos que cambian con frecuencia. Aquí se muestra el ejemplo anterior actualizado para utilizar el modelo [Cache-Aside][cache-aside-pattern].

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

Tenga en cuenta que el método `GetAsync` llama ahora a la clase `CacheService`, en lugar de llamar directamente a la base de datos. La clase `CacheService` primero intenta obtener el elemento de Azure Redis Cache. Si el valor no se encuentra allí, la clase `CacheService` invoca una función lambda que le pasó el responsable de realizar la llamada. La función lambda debe recuperar los datos de la base de datos. Esta implementación desvincula el repositorio de la solución de almacenamiento en caché concreta y `CacheService` de la base de datos.

## <a name="considerations"></a>Consideraciones

- Si la memoria caché no está disponible, quizás debido a un error transitorio, no devuelva un error al cliente. En su lugar, capture los datos del origen de datos original. Sin embargo, tenga en cuenta que, mientras se recupera la memoria caché, el almacén de datos original podría inundarse de solicitudes, dando lugar a tiempos de espera y errores de conexión. (Después de todo, esta es una de las motivaciones para utilizar una memoria caché en primer lugar). Use una técnica como [Circuit Breaker pattern][circuit-breaker] (patrón Circuit Breaker) para evitar sobrecargar el origen de datos.

- Las aplicaciones que almacenan en caché datos no estáticos se deben diseñar para admitir una posible coherencia.

- Para las API de web, puede admitir el almacenamiento en caché de cliente incluyendo un encabezado Cache-Control en los mensajes de solicitud y respuesta, y usar ETags para identificar las versiones de los objetos. Para más información, consulte [Implementación de la API][api-implementation].

- No tiene que almacenar en caché las entidades completas. Si la mayor parte de una entidad es estática, pero solo una pequeña parte cambia con frecuencia, almacene en caché los elementos estáticos y recupere los dinámicos del origen de datos. Este enfoque puede ayudar a reducir el volumen de operaciones de E/S que se realizan en el origen de datos.

- En algunos casos, si los datos volátiles son de corta duración, puede ser útil almacenarlos en caché. Por ejemplo, considere un dispositivo que envía continuamente actualizaciones de estado. Podría tener sentido almacenar en memoria caché esta información cuando llega y no escribirla en un almacén persistente.

- Para evitar que los datos se queden obsoletos, muchas soluciones de almacenamiento en caché admiten períodos de expiración configurables, de modo que se quitan automáticamente de la memoria caché después de un intervalo especificado. Debe ajustar la hora de expiración para su escenario. Los datos muy estáticos pueden permanecer en la memoria caché durante períodos más largos que los volátiles, que pueden quedar obsoletos con rapidez.

- Si la solución de almacenamiento en caché no proporciona una expiración integrada, debe implementar un proceso en segundo plano que limpie en ocasiones la memoria caché, para evitar que crezca de forma ilimitada.

- Además de almacenar en caché datos de un origen de datos externo, puede usar el almacenamiento en caché para guardar los resultados de cálculos complejos. Sin embargo, antes debe instrumentar la aplicación para determinar si realmente está limitada por la CPU.

- Puede ser útil preparar la memoria caché cuando la aplicación se inicia. Rellene la memoria caché con los datos que tengan más probabilidad de usarse.

- Incluya siempre instrumental que detecte los aciertos y los errores de caché. Utilice esta información para ajustar las directivas de almacenamiento en caché, por ejemplo, qué datos almacenar en ella y cuánto tiempo mantenerlos antes de que expiren.

- Si la falta de almacenamiento en caché constituye un cuello de botella, agregarlo puede aumentar tanto el volumen de solicitudes que el front-end web llegue a sobrecargarse. Los clientes pueden empezar a recibir errores HTTP 503 (servicio no disponible). Se trata de una indicación de que debe escalar horizontalmente el front-end.

## <a name="how-to-detect-the-problem"></a>Procedimiento para detectar el problema

Puede realizar los pasos siguientes para ayudar a identificar si la falta de almacenamiento en caché está causando problemas de rendimiento:

1. Revise el código de la aplicación. Realice un inventario de todos los almacenes de datos que la aplicación usa. Para cada uno, determine si la aplicación usa una memoria caché. Si es posible, determine con qué frecuencia cambian los datos. Buenos candidatos iniciales para el almacenamiento en caché son los datos que cambian lentamente y los datos estáticos de referencia que se lean con frecuencia.

2. Instrumente la aplicación y supervise el sistema real para averiguar la frecuencia con que la aplicación recupera datos o calcula la información.

3. Genere perfiles de la aplicación en un entorno de prueba para capturar las métricas de bajo nivel sobre la sobrecarga asociada a operaciones de acceso a datos u otros cálculos realizados con frecuencia.

4. Realice pruebas de carga de un entorno de prueba para identificar cómo responde el sistema con una carga de trabajo normal y con una carga intensa. Las pruebas de carga deben simular el patrón de acceso a datos observado en el entorno de producción con cargas de trabajo realistas.

5. Examine las estadísticas de acceso a los datos de los almacenes de datos subyacentes y revise con qué frecuencia se repiten las mismas solicitudes de datos.

## <a name="example-diagnosis"></a>Diagnóstico de ejemplo

En las secciones siguientes se aplican estos pasos para la aplicación de ejemplo descrita anteriormente.

### <a name="instrument-the-application-and-monitor-the-live-system"></a>Instrumentación de la aplicación y supervisión de los sistemas en vivo

Instrumente la aplicación y supervísela para obtener información acerca de las solicitudes específicas que realizan los usuarios mientras está en producción.

La siguiente imagen muestra la supervisión de los datos capturados por [New Relic][NewRelic] durante una prueba de carga. En este caso, la única operación HTTP GET realizada es `Person/GetAsync`. Pero, en un entorno real de producción, conocer la frecuencia relativa con que se realiza cada solicitud puede ofrecerle una visión general de los recursos que se deben almacenar en caché.

![New Relic que muestra las solicitudes de servidor para la aplicación CachingDemo][NewRelic-server-requests]

Si necesita realizar un análisis más profundo, puede usar un generador de perfiles para capturar los datos de rendimiento de bajo nivel en un entorno de prueba (no en el sistema de producción). Observe las métricas, como la velocidad de las solicitudes de E/S, el uso de la memoria y el uso de la CPU. Estas métricas pueden mostrar un gran número de solicitudes realizadas a un almacén de datos o un servicio, o que se repite un proceso que efectúa el mismo cálculo.

### <a name="load-test-the-application"></a>Prueba de carga de la aplicación

El siguiente gráfico muestra los resultados de la aplicación de ejemplo de la prueba de carga. La prueba de carga simula una carga por pasos de hasta 800 usuarios que realizan una serie de operaciones habituales.

![Resultados de la prueba de carga del rendimiento para el escenario sin almacenamiento en caché][Performance-Load-Test-Results-Uncached]

El número de pruebas superadas realizadas cada segundo alcanza un nivel estable y las solicitudes adicionales se ralentizan como resultado. El tiempo promedio de prueba aumenta regularmente con la carga de trabajo. El tiempo de respuesta se estabiliza una vez que la carga de usuarios alcanza el máximo.

### <a name="examine-data-access-statistics"></a>Examen de las estadísticas de acceso a los datos

Las estadísticas de acceso a los datos y otra información que un almacén proporciona pueden ofrecerle información útil, como las consultas que se repiten con más frecuencia. Por ejemplo, en Microsoft SQL Server, la vista de administración `sys.dm_exec_query_stats` tiene información estadística sobre las consultas ejecutadas recientemente. El texto de cada consulta está disponible en la vista `sys.dm_exec-query_plan`. Puede utilizar una herramienta como SQL Server Management Studio para ejecutar la siguiente consulta SQL y determinar la frecuencia con la que se realizan las consultas.

```SQL
SELECT UseCounts, Text, Query_Plan
FROM sys.dm_exec_cached_plans
CROSS APPLY sys.dm_exec_sql_text(plan_handle)
CROSS APPLY sys.dm_exec_query_plan(plan_handle)
```

La columna `UseCount` en los resultados indica la frecuencia con la que se ejecuta cada consulta. La siguiente imagen muestra que la tercera consulta se ejecuta más de 250 000 veces, significativamente más que cualquier otra.

![Resultados de la consulta de las vistas de administración dinámica en SQL Server Management Server][Dynamic-Management-Views]

Esta es la consulta SQL que está causando tantas solicitudes de base de datos:

```SQL
(@p__linq__0 int)SELECT TOP (2)
[Extent1].[BusinessEntityId] AS [BusinessEntityId],
[Extent1].[FirstName] AS [FirstName],
[Extent1].[LastName] AS [LastName]
FROM [Person].[Person] AS [Extent1]
WHERE [Extent1].[BusinessEntityId] = @p__linq__0
```

Esta es la consulta que Entity Framework genera en el método `GetByIdAsync` mostrado anteriormente.

### <a name="implement-the-solution-and-verify-the-result"></a>Implementación de la solución y comprobación del resultado

Después de incorporar una memoria caché, repita las pruebas de carga y compare los resultados de las anteriores sin memoria caché. Estos son los resultados de las pruebas de carga después de agregar una memoria caché a la aplicación de ejemplo.

![Resultados de la prueba de carga del rendimiento para el escenario con almacenamiento en caché][Performance-Load-Test-Results-Cached]

El volumen de pruebas superadas todavía alcanza un nivel estable, pero con una carga de usuarios mayor. La tasa de solicitudes en esta carga es significativamente más alta que la anterior. El tiempo promedio de prueba sigue aumentando con la carga, pero el tiempo de respuesta máximo es de 0,05 ms, en comparación con el valor de 1 ms anterior &mdash;una mejora del 20&times;.

## <a name="related-resources"></a>Recursos relacionados

- [Procedimientos recomendados para la implementación de API][api-implementation]
- [Patrón reservado en caché][cache-aside-pattern]
- [Procedimientos recomendados para el almacenamiento en caché][caching-guidance]
- [Patrón Circuit Breaker][circuit-breaker]

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
