---
title: Antipatrón Improper Instantiation
titleSuffix: Performance antipatterns for cloud apps
description: Evite la creación continua de nuevas instancias de un objeto que pretenda crearse una vez y después compartirse.
author: dragon119
ms.date: 06/05/2017
ms.custom: seodec18
ms.openlocfilehash: b92ae5f5e79a0ababf44d7aa2d771d4d72900cae
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/04/2019
ms.locfileid: "54009925"
---
# <a name="improper-instantiation-antipattern"></a><span data-ttu-id="fec9f-103">Antipatrón Improper Instantiation</span><span class="sxs-lookup"><span data-stu-id="fec9f-103">Improper Instantiation antipattern</span></span>

<span data-ttu-id="fec9f-104">La creación continua de nuevas instancias de un objeto que pretenda crearse una vez y después compartirse puede perjudicar el rendimiento.</span><span class="sxs-lookup"><span data-stu-id="fec9f-104">It can hurt performance to continually create new instances of an object that is meant to be created once and then shared.</span></span>

## <a name="problem-description"></a><span data-ttu-id="fec9f-105">Descripción del problema</span><span class="sxs-lookup"><span data-stu-id="fec9f-105">Problem description</span></span>

<span data-ttu-id="fec9f-106">Muchas bibliotecas proporcionan abstracciones de recursos externos.</span><span class="sxs-lookup"><span data-stu-id="fec9f-106">Many libraries provide abstractions of external resources.</span></span> <span data-ttu-id="fec9f-107">Internamente, estas clases suelen administrar sus propias conexiones a los recursos, que actúan como agentes que los clientes pueden utilizar para tener acceso al recurso.</span><span class="sxs-lookup"><span data-stu-id="fec9f-107">Internally, these classes typically manage their own connections to the resource, acting as brokers that clients can use to access the resource.</span></span> <span data-ttu-id="fec9f-108">Estos son algunos ejemplos de clases de agente que son pertinentes para las aplicaciones de Azure:</span><span class="sxs-lookup"><span data-stu-id="fec9f-108">Here are some examples of broker classes that are relevant to Azure applications:</span></span>

- <span data-ttu-id="fec9f-109">`System.Net.Http.HttpClient`.</span><span class="sxs-lookup"><span data-stu-id="fec9f-109">`System.Net.Http.HttpClient`.</span></span> <span data-ttu-id="fec9f-110">Se comunica con un servicio web mediante HTTP.</span><span class="sxs-lookup"><span data-stu-id="fec9f-110">Communicates with a web service using HTTP.</span></span>
- <span data-ttu-id="fec9f-111">`Microsoft.ServiceBus.Messaging.QueueClient`.</span><span class="sxs-lookup"><span data-stu-id="fec9f-111">`Microsoft.ServiceBus.Messaging.QueueClient`.</span></span> <span data-ttu-id="fec9f-112">Envía y recibe mensajes a una cola de Service Bus.</span><span class="sxs-lookup"><span data-stu-id="fec9f-112">Posts and receives messages to a Service Bus queue.</span></span>
- <span data-ttu-id="fec9f-113">`Microsoft.Azure.Documents.Client.DocumentClient`.</span><span class="sxs-lookup"><span data-stu-id="fec9f-113">`Microsoft.Azure.Documents.Client.DocumentClient`.</span></span> <span data-ttu-id="fec9f-114">Se conecta a una instancia de Cosmos DB</span><span class="sxs-lookup"><span data-stu-id="fec9f-114">Connects to a Cosmos DB instance</span></span>
- <span data-ttu-id="fec9f-115">`StackExchange.Redis.ConnectionMultiplexer`.</span><span class="sxs-lookup"><span data-stu-id="fec9f-115">`StackExchange.Redis.ConnectionMultiplexer`.</span></span> <span data-ttu-id="fec9f-116">Se conecta a Redis, incluida Azure Redis Cache.</span><span class="sxs-lookup"><span data-stu-id="fec9f-116">Connects to Redis, including Azure Redis Cache.</span></span>

<span data-ttu-id="fec9f-117">Estas clases se diseñaron para que su instancia se creara una vez y se reutilizase durante todo el ciclo de vida de una aplicación.</span><span class="sxs-lookup"><span data-stu-id="fec9f-117">These classes are intended to be instantiated once and reused throughout the lifetime of an application.</span></span> <span data-ttu-id="fec9f-118">Sin embargo, se suele malentender que estas clases se deben adquirir solo cuando sea necesario y liberarse rápidamente.</span><span class="sxs-lookup"><span data-stu-id="fec9f-118">However, it's a common misunderstanding that these classes should be acquired only as necessary and released quickly.</span></span> <span data-ttu-id="fec9f-119">(Las mencionadas aquí son bibliotecas de. NET, pero el patrón no es único para. NET). El siguiente ejemplo de ASP.NET crea una instancia de `HttpClient` para comunicarse con un servicio remoto.</span><span class="sxs-lookup"><span data-stu-id="fec9f-119">(The ones listed here happen to be .NET libraries, but the pattern is not unique to .NET.) The following ASP.NET example creates an instance of `HttpClient` to communicate with a remote service.</span></span> <span data-ttu-id="fec9f-120">Puede encontrar el ejemplo completo [aquí][sample-app].</span><span class="sxs-lookup"><span data-stu-id="fec9f-120">You can find the complete sample [here][sample-app].</span></span>

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

<span data-ttu-id="fec9f-121">En una aplicación web, esta técnica no es escalable.</span><span class="sxs-lookup"><span data-stu-id="fec9f-121">In a web application, this technique is not scalable.</span></span> <span data-ttu-id="fec9f-122">Se crea un nuevo objeto `HttpClient` para cada solicitud del usuario.</span><span class="sxs-lookup"><span data-stu-id="fec9f-122">A new `HttpClient` object is created for each user request.</span></span> <span data-ttu-id="fec9f-123">Con mucha carga, el servidor web puede agotar el número de sockets disponibles, lo que genera errores `SocketException`.</span><span class="sxs-lookup"><span data-stu-id="fec9f-123">Under heavy load, the web server may exhaust the number of available sockets, resulting in `SocketException` errors.</span></span>

<span data-ttu-id="fec9f-124">Este problema no se limita a la clase `HttpClient`.</span><span class="sxs-lookup"><span data-stu-id="fec9f-124">This problem is not restricted to the `HttpClient` class.</span></span> <span data-ttu-id="fec9f-125">Otras clases que encapsulan recursos o que son caros de crear pueden causar problemas similares.</span><span class="sxs-lookup"><span data-stu-id="fec9f-125">Other classes that wrap resources or are expensive to create might cause similar issues.</span></span> <span data-ttu-id="fec9f-126">En el ejemplo siguiente se crea una instancia de la clase `ExpensiveToCreateService`.</span><span class="sxs-lookup"><span data-stu-id="fec9f-126">The following example creates an instances of the `ExpensiveToCreateService` class.</span></span> <span data-ttu-id="fec9f-127">Aquí el problema no es necesariamente el agotamiento de sockets, sino simplemente cuánto se tarda en crear cada instancia.</span><span class="sxs-lookup"><span data-stu-id="fec9f-127">Here the issue is not necessarily socket exhaustion, but simply how long it takes to create each instance.</span></span> <span data-ttu-id="fec9f-128">Crear y destruir instancias de esta clase continuamente puede afectar de forma desfavorable a la escalabilidad del sistema.</span><span class="sxs-lookup"><span data-stu-id="fec9f-128">Continually creating and destroying instances of this class might adversely affect the scalability of the system.</span></span>

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

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="fec9f-129">Procedimiento para corregir el problema</span><span class="sxs-lookup"><span data-stu-id="fec9f-129">How to fix the problem</span></span>

<span data-ttu-id="fec9f-130">Si la clase que encapsula el recurso externo se puede compartir y es segura para subprocesos, cree una instancia singleton compartida o un grupo de instancias reutilizables de la clase.</span><span class="sxs-lookup"><span data-stu-id="fec9f-130">If the class that wraps the external resource is shareable and thread-safe, create a shared singleton instance or a pool of reusable instances of the class.</span></span>

<span data-ttu-id="fec9f-131">El siguiente ejemplo utiliza una instancia estática de `HttpClient`, que por tanto comparte la conexión a través de todas las solicitudes.</span><span class="sxs-lookup"><span data-stu-id="fec9f-131">The following example uses a static `HttpClient` instance, thus sharing the connection across all requests.</span></span>

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

## <a name="considerations"></a><span data-ttu-id="fec9f-132">Consideraciones</span><span class="sxs-lookup"><span data-stu-id="fec9f-132">Considerations</span></span>

- <span data-ttu-id="fec9f-133">El elemento clave de este antipatrón es la creación y destrucción reiterada de instancias de un objeto *compartible*.</span><span class="sxs-lookup"><span data-stu-id="fec9f-133">The key element of this antipattern is repeatedly creating and destroying instances of a *shareable* object.</span></span> <span data-ttu-id="fec9f-134">Si una clase no puede compartirse (no es segura para subprocesos), este antipatrón no se aplica.</span><span class="sxs-lookup"><span data-stu-id="fec9f-134">If a class is not shareable (not thread-safe), then this antipattern does not apply.</span></span>

- <span data-ttu-id="fec9f-135">El tipo de recurso compartido podría dictar si debe utilizar un singleton o crear un grupo.</span><span class="sxs-lookup"><span data-stu-id="fec9f-135">The type of shared resource might dictate whether you should use a singleton or create a pool.</span></span> <span data-ttu-id="fec9f-136">La clase `HttpClient` está diseñada para ser compartida en lugar de agrupada.</span><span class="sxs-lookup"><span data-stu-id="fec9f-136">The `HttpClient` class is designed to be shared rather than pooled.</span></span> <span data-ttu-id="fec9f-137">Otros objetos pueden admitir la agrupación, lo que permite al sistema distribuir la carga de trabajo en varias instancias.</span><span class="sxs-lookup"><span data-stu-id="fec9f-137">Other objects might support pooling, enabling the system to spread the workload across multiple instances.</span></span>

- <span data-ttu-id="fec9f-138">Los objetos que se comparten entre varias solicitudes *deben* ser seguros para subprocesos.</span><span class="sxs-lookup"><span data-stu-id="fec9f-138">Objects that you share across multiple requests *must* be thread-safe.</span></span> <span data-ttu-id="fec9f-139">La clase `HttpClient` está diseñada para usarse de esta manera, pero otras quizás no admitan solicitudes simultáneas, de modo que debe revisar la documentación disponible.</span><span class="sxs-lookup"><span data-stu-id="fec9f-139">The `HttpClient` class is designed to be used in this manner, but other classes might not support concurrent requests, so check the available documentation.</span></span>

- <span data-ttu-id="fec9f-140">Tenga cuidado al establecer las propiedades de configuración de objetos compartidos ya que esto puede producir condiciones de carrera.</span><span class="sxs-lookup"><span data-stu-id="fec9f-140">Be careful about setting properties on shared objects, as this can lead to race conditions.</span></span> <span data-ttu-id="fec9f-141">Por ejemplo, si se establece `DefaultRequestHeaders` en la clase `HttpClient` antes de cada solicitud puede crear una condición de carrera.</span><span class="sxs-lookup"><span data-stu-id="fec9f-141">For example, setting `DefaultRequestHeaders` on the `HttpClient` class before each request can create a race condition.</span></span> <span data-ttu-id="fec9f-142">Establezca esas propiedades una vez (por ejemplo, durante el inicio) y cree instancias independientes si tiene que configurar valores diferentes.</span><span class="sxs-lookup"><span data-stu-id="fec9f-142">Set such properties once (for example, during startup), and create separate instances if you need to configure different settings.</span></span>

- <span data-ttu-id="fec9f-143">Algunos tipos de recursos son escasos y no deben retenerse.</span><span class="sxs-lookup"><span data-stu-id="fec9f-143">Some resource types are scarce and should not be held onto.</span></span> <span data-ttu-id="fec9f-144">Las conexiones de base de datos son un ejemplo.</span><span class="sxs-lookup"><span data-stu-id="fec9f-144">Database connections are an example.</span></span> <span data-ttu-id="fec9f-145">Al mantener abierta una conexión de base de datos que no es necesaria, puede impedir que otros usuarios simultáneos obtengan acceso a la base de datos.</span><span class="sxs-lookup"><span data-stu-id="fec9f-145">Holding an open database connection that is not required may prevent other concurrent users from gaining access to the database.</span></span>

- <span data-ttu-id="fec9f-146">En .NET Framework, muchos objetos que establecen conexiones a recursos externos se crean con métodos de factoría estáticos de otras clases que administran estas conexiones.</span><span class="sxs-lookup"><span data-stu-id="fec9f-146">In the .NET Framework, many objects that establish connections to external resources are created by using static factory methods of other classes that manage these connections.</span></span> <span data-ttu-id="fec9f-147">Estos objetos están pensados para guardarse y volverse a utilizar, en lugar de desecharse y volverse a crear.</span><span class="sxs-lookup"><span data-stu-id="fec9f-147">These objects are intended to be saved and reused, rather than disposed and recreated.</span></span> <span data-ttu-id="fec9f-148">Por ejemplo, en Azure Service Bus, el objeto `QueueClient` se crea a través de un objeto `MessagingFactory`.</span><span class="sxs-lookup"><span data-stu-id="fec9f-148">For example, in Azure Service Bus, the `QueueClient` object is created through a `MessagingFactory` object.</span></span> <span data-ttu-id="fec9f-149">Internamente, el objeto `MessagingFactory` administra las conexiones.</span><span class="sxs-lookup"><span data-stu-id="fec9f-149">Internally, the `MessagingFactory` manages connections.</span></span> <span data-ttu-id="fec9f-150">Para más información, consulte [Procedimientos recomendados para mejorar el rendimiento mediante la mensajería de Service Bus][service-bus-messaging].</span><span class="sxs-lookup"><span data-stu-id="fec9f-150">For more information, see [Best Practices for performance improvements using Service Bus Messaging][service-bus-messaging].</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="fec9f-151">Procedimiento para detectar el problema</span><span class="sxs-lookup"><span data-stu-id="fec9f-151">How to detect the problem</span></span>

<span data-ttu-id="fec9f-152">Entre los síntomas de este problema se incluyen una disminución del rendimiento o una tasa mayor de errores, además de los siguientes:</span><span class="sxs-lookup"><span data-stu-id="fec9f-152">Symptoms of this problem include a drop in throughput or an increased error rate, along with one or more of the following:</span></span>

- <span data-ttu-id="fec9f-153">Un aumento en las excepciones que indican el agotamiento de los recursos, como sockets, conexiones de base de datos, identificadores de archivos, etc.</span><span class="sxs-lookup"><span data-stu-id="fec9f-153">An increase in exceptions that indicate exhaustion of resources such as sockets, database connections, file handles, and so on.</span></span>
- <span data-ttu-id="fec9f-154">Un aumento del uso de la memoria y de la recolección de elementos no utilizados.</span><span class="sxs-lookup"><span data-stu-id="fec9f-154">Increased memory use and garbage collection.</span></span>
- <span data-ttu-id="fec9f-155">Un aumento de la actividad de red, del disco o de las bases de datos.</span><span class="sxs-lookup"><span data-stu-id="fec9f-155">An increase in network, disk, or database activity.</span></span>

<span data-ttu-id="fec9f-156">Puede realizar los pasos siguientes para ayudar a identificar este problema:</span><span class="sxs-lookup"><span data-stu-id="fec9f-156">You can perform the following steps to help identify this problem:</span></span>

1. <span data-ttu-id="fec9f-157">Supervise los procesos del sistema de producción para identificar los puntos en los que se ralentizan los tiempos de respuesta o el sistema produce errores debido a la falta de recursos.</span><span class="sxs-lookup"><span data-stu-id="fec9f-157">Performing process monitoring of the production system, to identify points when response times slow down or the system fails due to lack of resources.</span></span>
2. <span data-ttu-id="fec9f-158">Examine los datos de telemetría capturados en estos momentos para determinar qué operaciones podrían crear y destruir objetos que consuman recursos.</span><span class="sxs-lookup"><span data-stu-id="fec9f-158">Examine the telemetry data captured at these points to determine which operations might be creating and destroying resource-consuming objects.</span></span>
3. <span data-ttu-id="fec9f-159">Realice una prueba de carga de cada operación sospechosa, en un entorno de pruebas controlado en lugar de en el sistema de producción.</span><span class="sxs-lookup"><span data-stu-id="fec9f-159">Load test each suspected operation, in a controlled test environment rather than the production system.</span></span>
4. <span data-ttu-id="fec9f-160">Revise el código fuente y examine cómo se administran los objetos de agente.</span><span class="sxs-lookup"><span data-stu-id="fec9f-160">Review the source code and examine the how broker objects are managed.</span></span>

<span data-ttu-id="fec9f-161">Observe en los seguimientos de la pila las operaciones que se ejecutan con lentitud o que generan excepciones cuando el sistema está sometido a carga.</span><span class="sxs-lookup"><span data-stu-id="fec9f-161">Look at stack traces for operations that are slow-running or that generate exceptions when the system is under load.</span></span> <span data-ttu-id="fec9f-162">Esta información puede ayudar a identificar el modo en que estas operaciones usan los recursos.</span><span class="sxs-lookup"><span data-stu-id="fec9f-162">This information can help to identify how these operations are utilizing resources.</span></span> <span data-ttu-id="fec9f-163">Las excepciones pueden ayudarle a determinar si los errores están provocados porque los recursos compartidos se agotan.</span><span class="sxs-lookup"><span data-stu-id="fec9f-163">Exceptions can help to determine whether errors are caused by shared resources being exhausted.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="fec9f-164">Diagnóstico de ejemplo</span><span class="sxs-lookup"><span data-stu-id="fec9f-164">Example diagnosis</span></span>

<span data-ttu-id="fec9f-165">En las secciones siguientes se aplican estos pasos para la aplicación de ejemplo descrita anteriormente.</span><span class="sxs-lookup"><span data-stu-id="fec9f-165">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="identify-points-of-slow-down-or-failure"></a><span data-ttu-id="fec9f-166">Identificación de los puntos de ralentización o de error</span><span class="sxs-lookup"><span data-stu-id="fec9f-166">Identify points of slow down or failure</span></span>

<span data-ttu-id="fec9f-167">La siguiente imagen muestra los resultados que se generan mediante [APM de New Relic][new-relic], con las operaciones que tienen un tiempo de respuesta insuficiente.</span><span class="sxs-lookup"><span data-stu-id="fec9f-167">The following image shows results generated using [New Relic APM][new-relic], showing operations that have a poor response time.</span></span> <span data-ttu-id="fec9f-168">En este caso, merece la pena seguir investigando el método `GetProductAsync` del controlador `NewHttpClientInstancePerRequest`.</span><span class="sxs-lookup"><span data-stu-id="fec9f-168">In this case, the `GetProductAsync` method in the `NewHttpClientInstancePerRequest` controller is worth investigating further.</span></span> <span data-ttu-id="fec9f-169">Tenga en cuenta que la tasa de errores también aumenta cuando estas operaciones se están ejecutando.</span><span class="sxs-lookup"><span data-stu-id="fec9f-169">Notice that the error rate also increases when these operations are running.</span></span>

![El panel del monitor de New Relic que muestra la aplicación de ejemplo creando una nueva instancia de un objeto HttpClient para cada solicitud][dashboard-new-HTTPClient-instance]

### <a name="examine-telemetry-data-and-find-correlations"></a><span data-ttu-id="fec9f-171">Examen de los datos de telemetría y búsqueda de correlaciones</span><span class="sxs-lookup"><span data-stu-id="fec9f-171">Examine telemetry data and find correlations</span></span>

<span data-ttu-id="fec9f-172">La imagen siguiente muestra los datos capturados con subprocesos de generación de perfiles, durante el mismo período correspondiente en la imagen anterior.</span><span class="sxs-lookup"><span data-stu-id="fec9f-172">The next image shows data captured using thread profiling, over the same period corresponding as the previous image.</span></span> <span data-ttu-id="fec9f-173">El sistema emplea mucho tiempo en abrir conexiones de socket y aún más tiempo en cerrarlas y controlar las excepciones de socket.</span><span class="sxs-lookup"><span data-stu-id="fec9f-173">The system spends a significant time opening socket connections, and even more time closing them and handling socket exceptions.</span></span>

![El generador de perfiles de subproceso de New Relic que muestra la aplicación de ejemplo creando una nueva instancia de un objeto HttpClient para cada solicitud][thread-profiler-new-HTTPClient-instance]

### <a name="performing-load-testing"></a><span data-ttu-id="fec9f-175">Pruebas de carga</span><span class="sxs-lookup"><span data-stu-id="fec9f-175">Performing load testing</span></span>

<span data-ttu-id="fec9f-176">Use pruebas de carga para simular las operaciones típicas que los usuarios pueden realizar.</span><span class="sxs-lookup"><span data-stu-id="fec9f-176">Use load testing to simulate the typical operations that users might perform.</span></span> <span data-ttu-id="fec9f-177">Así puede ayudar a identificar qué partes de un sistema sufren de agotamiento de recursos en cargas variables.</span><span class="sxs-lookup"><span data-stu-id="fec9f-177">This can help to identify which parts of a system suffer from resource exhaustion under varying loads.</span></span> <span data-ttu-id="fec9f-178">Realice estas pruebas en un entorno controlado en lugar de en el sistema de producción.</span><span class="sxs-lookup"><span data-stu-id="fec9f-178">Perform these tests in a controlled environment rather than the production system.</span></span> <span data-ttu-id="fec9f-179">El siguiente gráfico muestra el rendimiento de las solicitudes administradas por el controlador `NewHttpClientInstancePerRequest` a medida que la carga de usuarios aumenta hasta 100 simultáneos.</span><span class="sxs-lookup"><span data-stu-id="fec9f-179">The following graph shows the throughput of requests handled by the `NewHttpClientInstancePerRequest` controller as the user load increases to 100 concurrent users.</span></span>

![Rendimiento de la aplicación de ejemplo que crea una nueva instancia de un objeto HttpClient para cada solicitud][throughput-new-HTTPClient-instance]

<span data-ttu-id="fec9f-181">En primer lugar, el volumen de solicitudes administradas por segundo aumenta al mismo tiempo que la carga de trabajo.</span><span class="sxs-lookup"><span data-stu-id="fec9f-181">At first, the volume of requests handled per second increases as the workload increases.</span></span> <span data-ttu-id="fec9f-182">Sin embargo, con aproximadamente 30 usuarios, el volumen de solicitudes correctas alcanza el límite y el sistema empieza a generar excepciones.</span><span class="sxs-lookup"><span data-stu-id="fec9f-182">At about 30 users, however, the volume of successful requests reaches a limit, and the system starts to generate exceptions.</span></span> <span data-ttu-id="fec9f-183">Desde ese momento, el volumen de excepciones aumenta gradualmente con la carga de usuarios.</span><span class="sxs-lookup"><span data-stu-id="fec9f-183">From then on, the volume of exceptions gradually increases with the user load.</span></span>

<span data-ttu-id="fec9f-184">La prueba de carga notificó estos errores como errores HTTP 500 (servidor interno).</span><span class="sxs-lookup"><span data-stu-id="fec9f-184">The load test reported these failures as HTTP 500 (Internal Server) errors.</span></span> <span data-ttu-id="fec9f-185">Al revisar la telemetría, se evidenció que estos errores eran causados porque el sistema se quedó sin recursos de socket, cada vez más a medida que se creaban más objetos `HttpClient`.</span><span class="sxs-lookup"><span data-stu-id="fec9f-185">Reviewing the telemetry showed that these errors were caused by the system running out of socket resources, as more and more `HttpClient` objects were created.</span></span>

<span data-ttu-id="fec9f-186">El gráfico siguiente muestra una prueba similar para un controlador que crea el objeto `ExpensiveToCreateService` personalizado.</span><span class="sxs-lookup"><span data-stu-id="fec9f-186">The next graph shows a similar test for a controller that creates the custom `ExpensiveToCreateService` object.</span></span>

![Rendimiento de la aplicación de ejemplo que crea una nueva instancia de un objeto ExpensiveToCreateService para cada solicitud][throughput-new-ExpensiveToCreateService-instance]

<span data-ttu-id="fec9f-188">En esta ocasión, el controlador no genera ninguna excepción, pero el rendimiento se estabiliza, mientras que el tiempo promedio de respuesta aumenta en un factor de 20.</span><span class="sxs-lookup"><span data-stu-id="fec9f-188">This time, the controller does not generate any exceptions, but throughput still reaches a plateau, while the average response time increases by a factor of 20.</span></span> <span data-ttu-id="fec9f-189">(El gráfico usa una escala logarítmica para el tiempo de respuesta y el rendimiento). La telemetría mostró que la creación de nuevas instancias de `ExpensiveToCreateService` fue la causa principal del problema.</span><span class="sxs-lookup"><span data-stu-id="fec9f-189">(The graph uses a logarithmic scale for response time and throughput.) Telemetry showed that creating new instances of the `ExpensiveToCreateService` was the main cause of the problem.</span></span>

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="fec9f-190">Implementación de la solución y comprobación del resultado</span><span class="sxs-lookup"><span data-stu-id="fec9f-190">Implement the solution and verify the result</span></span>

<span data-ttu-id="fec9f-191">Después de cambiar el método `GetProductAsync` para compartir una sola instancia de `HttpClient`, una segunda prueba de carga mostró que el rendimiento había mejorado.</span><span class="sxs-lookup"><span data-stu-id="fec9f-191">After switching the `GetProductAsync` method to share a single `HttpClient` instance, a second load test showed improved performance.</span></span> <span data-ttu-id="fec9f-192">No se notificaron errores y el sistema fue capaz de controlar una carga creciente de hasta 500 solicitudes por segundo.</span><span class="sxs-lookup"><span data-stu-id="fec9f-192">No errors were reported, and the system was able to handle an increasing load of up to 500 requests per second.</span></span> <span data-ttu-id="fec9f-193">El tiempo promedio de respuesta se redujo a la mitad en comparación con la prueba anterior.</span><span class="sxs-lookup"><span data-stu-id="fec9f-193">The average response time was cut in half, compared with the previous test.</span></span>

![Rendimiento de la aplicación de ejemplo que reutiliza la misma instancia de un objeto HttpClient para cada solicitud][throughput-single-HTTPClient-instance]

<span data-ttu-id="fec9f-195">Para la comparación, la siguiente imagen muestra la telemetría de seguimiento de la pila.</span><span class="sxs-lookup"><span data-stu-id="fec9f-195">For comparison, the following image shows the stack trace telemetry.</span></span> <span data-ttu-id="fec9f-196">En esta ocasión, el sistema emplea la mayor parte del tiempo en realizar el trabajo real, en lugar de abrir y cerrar sockets.</span><span class="sxs-lookup"><span data-stu-id="fec9f-196">This time, the system spends most of its time performing real work, rather than opening and closing sockets.</span></span>

![El generador de perfiles de subproceso de New Relic que muestra la aplicación de ejemplo creando una nueva instancia de un objeto HttpClient para cada solicitud][thread-profiler-single-HTTPClient-instance]

<span data-ttu-id="fec9f-198">El gráfico siguiente muestra una prueba de carga similar con una instancia compartida del objeto `ExpensiveToCreateService`.</span><span class="sxs-lookup"><span data-stu-id="fec9f-198">The next graph shows a similar load test using a shared instance of the `ExpensiveToCreateService` object.</span></span> <span data-ttu-id="fec9f-199">Una vez más, el volumen de solicitudes tratadas aumenta en consonancia con la carga de usuarios, mientras que el tiempo promedio de respuesta sigue siendo bajo.</span><span class="sxs-lookup"><span data-stu-id="fec9f-199">Again, the volume of handled requests increases in line with the user load, while the average response time remains low.</span></span>

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
