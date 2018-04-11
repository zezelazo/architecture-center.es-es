---
title: Antipatrón Improper Instantiation
description: Evite la creación continua de nuevas instancias de un objeto que pretenda crearse una vez y después compartirse.
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: 4d5ef9ad9e675b46df94b51e81d7a4bd4c1b25e9
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/23/2018
---
# <a name="improper-instantiation-antipattern"></a>Antipatrón Improper Instantiation

La creación continua de nuevas instancias de un objeto que pretenda crearse una vez y después compartirse puede perjudicar el rendimiento. 

## <a name="problem-description"></a>Descripción del problema

Muchas bibliotecas proporcionan abstracciones de recursos externos. Internamente, estas clases suelen administrar sus propias conexiones a los recursos, que actúan como agentes que los clientes pueden utilizar para tener acceso al recurso. Estos son algunos ejemplos de clases de agente que son pertinentes para las aplicaciones de Azure:

- `System.Net.Http.HttpClient`. Se comunica con un servicio web mediante HTTP.
- `Microsoft.ServiceBus.Messaging.QueueClient`. Envía y recibe mensajes a una cola de Service Bus. 
- `Microsoft.Azure.Documents.Client.DocumentClient`. Se conecta a una instancia de Cosmos DB
- `StackExchange.Redis.ConnectionMultiplexer`. Se conecta a Redis, incluida Azure Redis Cache.

Estas clases se diseñaron para que su instancia se creara una vez y se reutilizase durante todo el ciclo de vida de una aplicación. Sin embargo, se suele malentender que estas clases se deben adquirir solo cuando sea necesario y liberarse rápidamente. (Las mencionadas aquí son bibliotecas de. NET, pero el patrón no es único para. NET). El siguiente ejemplo de ASP.NET crea una instancia de `HttpClient` para comunicarse con un servicio remoto. Puede encontrar el ejemplo completo [aquí][sample-app].

```csharp
public class NewHttpClientInstancePerRequestController : ApiController
{
    // This method creates a new instance of HttpClient and disposes it for every call to GetProductAsync.
    public async Task<Product> GetProductAsync(string id)
    {
        using (var httpClient = new HttpClient())
        {
            var hostName = HttpContext.Current.Request.Url.Host;
            var result = await httpClient.GetStringAsync(string.Format("http://{0}:8080/api/...", hostName));
            return new Product { Name = result };
        }
    }
}
```

En una aplicación web, esta técnica no es escalable. Se crea un nuevo objeto `HttpClient` para cada solicitud del usuario. Con mucha carga, el servidor web puede agotar el número de sockets disponibles, lo que genera errores `SocketException`.

Este problema no se limita a la clase `HttpClient`. Otras clases que encapsulan recursos o que son caros de crear pueden causar problemas similares. En el ejemplo siguiente se crea una instancia de la clase `ExpensiveToCreateService`. Aquí el problema no es necesariamente el agotamiento de sockets, sino simplemente cuánto se tarda en crear cada instancia. Crear y destruir instancias de esta clase continuamente puede afectar de forma desfavorable a la escalabilidad del sistema.

```csharp
public class NewServiceInstancePerRequestController : ApiController
{
    public async Task<Product> GetProductAsync(string id)
    {
        var expensiveToCreateService = new ExpensiveToCreateService();
        return await expensiveToCreateService.GetProductByIdAsync(id);
    }
}

public class ExpensiveToCreateService
{
    public ExpensiveToCreateService()
    {
        // Simulate delay due to setup and configuration of ExpensiveToCreateService
        Thread.SpinWait(Int32.MaxValue / 100);
    }
    ...
}
```

## <a name="how-to-fix-the-problem"></a>Procedimiento para corregir el problema

Si la clase que encapsula el recurso externo se puede compartir y es segura para subprocesos, cree una instancia singleton compartida o un grupo de instancias reutilizables de la clase.

El siguiente ejemplo utiliza una instancia estática de `HttpClient`, que por tanto comparte la conexión a través de todas las solicitudes.

```csharp
public class SingleHttpClientInstanceController : ApiController
{
    private static readonly HttpClient httpClient;

    static SingleHttpClientInstanceController()
    {
        httpClient = new HttpClient();
    }

    // This method uses the shared instance of HttpClient for every call to GetProductAsync.
    public async Task<Product> GetProductAsync(string id)
    {
        var hostName = HttpContext.Current.Request.Url.Host;
        var result = await httpClient.GetStringAsync(string.Format("http://{0}:8080/api/...", hostName));
        return new Product { Name = result };
    }
}
```

## <a name="considerations"></a>Consideraciones

- El elemento clave de este antipatrón es la creación y destrucción reiterada de instancias de un objeto *compartible*. Si una clase no puede compartirse (no es segura para subprocesos), este antipatrón no se aplica.

- El tipo de recurso compartido podría dictar si debe utilizar un singleton o crear un grupo. La clase `HttpClient` está diseñada para ser compartida en lugar de agrupada. Otros objetos pueden admitir la agrupación, lo que permite al sistema distribuir la carga de trabajo en varias instancias.

- Los objetos que se comparten entre varias solicitudes *deben* ser seguros para subprocesos. La clase `HttpClient` está diseñada para usarse de esta manera, pero otras quizás no admitan solicitudes simultáneas, de modo que debe revisar la documentación disponible.

- Algunos tipos de recursos son escasos y no deben retenerse. Las conexiones de base de datos son un ejemplo. Al mantener abierta una conexión de base de datos que no es necesaria, puede impedir que otros usuarios simultáneos obtengan acceso a la base de datos.

- En .NET Framework, muchos objetos que establecen conexiones a recursos externos se crean con métodos de factoría estáticos de otras clases que administran estas conexiones. Estos objetos de factorías están pensados para guardarse y volverse a utilizar, en lugar de desecharse y volverse a crear. Por ejemplo, en Azure Service Bus, el objeto `QueueClient` se crea a través de un objeto `MessagingFactory`. Internamente, el objeto `MessagingFactory` administra las conexiones. Para más información, consulte [Procedimientos recomendados para mejorar el rendimiento mediante la mensajería de Service Bus][service-bus-messaging].

## <a name="how-to-detect-the-problem"></a>Procedimiento para detectar el problema

Entre los síntomas de este problema se incluyen una disminución del rendimiento o una tasa mayor de errores, además de los siguientes: 

- Un aumento en las excepciones que indican el agotamiento de los recursos, como sockets, conexiones de base de datos, identificadores de archivos, etc. 
- Un aumento del uso de la memoria y de la recolección de elementos no utilizados.
- Un aumento de la actividad de red, del disco o de las bases de datos.

Puede realizar los pasos siguientes para ayudar a identificar este problema:

1. Supervise los procesos del sistema de producción para identificar los puntos en los que se ralentizan los tiempos de respuesta o el sistema produce errores debido a la falta de recursos.
2. Examine los datos de telemetría capturados en estos momentos para determinar qué operaciones podrían crear y destruir objetos que consuman recursos.
3. Realice una prueba de carga de cada operación sospechosa, en un entorno de pruebas controlado en lugar de en el sistema de producción.
4. Revise el código fuente y examine cómo se administran los objetos de agente.

Observe en los seguimientos de la pila las operaciones que se ejecutan con lentitud o que generan excepciones cuando el sistema está sometido a carga. Esta información puede ayudar a identificar el modo en que estas operaciones usan los recursos. Las excepciones pueden ayudarle a determinar si los errores están provocados porque los recursos compartidos se agotan. 

## <a name="example-diagnosis"></a>Diagnóstico de ejemplo

En las secciones siguientes se aplican estos pasos para la aplicación de ejemplo descrita anteriormente.

### <a name="identify-points-of-slow-down-or-failure"></a>Identificación de los puntos de ralentización o de error

La siguiente imagen muestra los resultados que se generan mediante [APM de New Relic][new-relic], con las operaciones que tienen un tiempo de respuesta insuficiente. En este caso, merece la pena seguir investigando el método `GetProductAsync` del controlador `NewHttpClientInstancePerRequest`. Tenga en cuenta que la tasa de errores también aumenta cuando estas operaciones se están ejecutando. 

![El panel del monitor de New Relic que muestra la aplicación de ejemplo creando una nueva instancia de un objeto HttpClient para cada solicitud][dashboard-new-HTTPClient-instance]

### <a name="examine-telemetry-data-and-find-correlations"></a>Examen de los datos de telemetría y búsqueda de correlaciones

La imagen siguiente muestra los datos capturados con subprocesos de generación de perfiles, durante el mismo período correspondiente en la imagen anterior. El sistema emplea mucho tiempo en abrir conexiones de socket y aún más tiempo en cerrarlas y controlar las excepciones de socket.

![El generador de perfiles de subproceso de New Relic que muestra la aplicación de ejemplo creando una nueva instancia de un objeto HttpClient para cada solicitud][thread-profiler-new-HTTPClient-instance]

### <a name="performing-load-testing"></a>Pruebas de carga

Use pruebas de carga para simular las operaciones típicas que los usuarios pueden realizar. Así puede ayudar a identificar qué partes de un sistema sufren de agotamiento de recursos en cargas variables. Realice estas pruebas en un entorno controlado en lugar de en el sistema de producción. El siguiente gráfico muestra el rendimiento de las solicitudes administradas por el controlador `NewHttpClientInstancePerRequest` a medida que la carga de usuarios aumenta hasta 100 simultáneos.

![Rendimiento de la aplicación de ejemplo que crea una nueva instancia de un objeto HttpClient para cada solicitud][throughput-new-HTTPClient-instance]

En primer lugar, el volumen de solicitudes administradas por segundo aumenta al mismo tiempo que la carga de trabajo. Sin embargo, con aproximadamente 30 usuarios, el volumen de solicitudes correctas alcanza el límite y el sistema empieza a generar excepciones. Desde ese momento, el volumen de excepciones aumenta gradualmente con la carga de usuarios. 

La prueba de carga notificó estos errores como errores HTTP 500 (servidor interno). Al revisar la telemetría, se evidenció que estos errores eran causados porque el sistema se quedó sin recursos de socket, cada vez más a medida que se creaban más objetos `HttpClient`.

El gráfico siguiente muestra una prueba similar para un controlador que crea el objeto `ExpensiveToCreateService` personalizado.

![Rendimiento de la aplicación de ejemplo que crea una nueva instancia de un objeto ExpensiveToCreateService para cada solicitud][throughput-new-ExpensiveToCreateService-instance]

En esta ocasión, el controlador no genera ninguna excepción, pero el rendimiento se estabiliza, mientras que el tiempo promedio de respuesta aumenta en un factor de 20. (El gráfico usa una escala logarítmica para el tiempo de respuesta y el rendimiento). La telemetría mostró que la creación de nuevas instancias de `ExpensiveToCreateService` fue la causa principal del problema.

### <a name="implement-the-solution-and-verify-the-result"></a>Implementación de la solución y comprobación del resultado

Después de cambiar el método `GetProductAsync` para compartir una sola instancia de `HttpClient`, una segunda prueba de carga mostró que el rendimiento había mejorado. No se notificaron errores y el sistema fue capaz de controlar una carga creciente de hasta 500 solicitudes por segundo. El tiempo promedio de respuesta se redujo a la mitad en comparación con la prueba anterior.

![Rendimiento de la aplicación de ejemplo que reutiliza la misma instancia de un objeto HttpClient para cada solicitud][throughput-single-HTTPClient-instance]

Para la comparación, la siguiente imagen muestra la telemetría de seguimiento de la pila. En esta ocasión, el sistema emplea la mayor parte del tiempo en realizar el trabajo real, en lugar de abrir y cerrar sockets.

![El generador de perfiles de subproceso de New Relic que muestra la aplicación de ejemplo creando una nueva instancia de un objeto HttpClient para cada solicitud][thread-profiler-single-HTTPClient-instance]

El gráfico siguiente muestra una prueba de carga similar con una instancia compartida del objeto `ExpensiveToCreateService`. Una vez más, el volumen de solicitudes tratadas aumenta en consonancia con la carga de usuarios, mientras que el tiempo promedio de respuesta sigue siendo bajo. 

![Rendimiento de la aplicación de ejemplo que reutiliza la misma instancia de un objeto HttpClient para cada solicitud][throughput-single-ExpensiveToCreateService-instance]



[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/ImproperInstantiation
[service-bus-messaging]: /azure/service-bus-messaging/service-bus-performance-improvements
[new-relic]: https://newrelic.com/application-monitoring
[throughput-new-HTTPClient-instance]: _images/HttpClientInstancePerRequest.jpg
[dashboard-new-HTTPClient-instance]: _images/HttpClientInstancePerRequestWebTransactions.jpg
[thread-profiler-new-HTTPClient-instance]: _images/HttpClientInstancePerRequestThreadProfile.jpg
[throughput-new-ExpensiveToCreateService-instance]: _images/ServiceInstancePerRequest.jpg
[throughput-single-HTTPClient-instance]: _images/SingleHttpClientInstance.jpg
[throughput-single-ExpensiveToCreateService-instance]: _images/SingleServiceInstance.jpg
[thread-profiler-single-HTTPClient-instance]: _images/SingleHttpClientInstanceThreadProfile.jpg
