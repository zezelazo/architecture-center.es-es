---
title: Antipatrón Busy Front End
description: El trabajo asincrónico en un gran número de subprocesos en segundo plano puede privar de recursos a otras tareas de primer plano.
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: 89a2d6c41af1e19ca1b9b6a0a5dceac615afd60a
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/28/2018
ms.locfileid: "47428302"
---
# <a name="busy-front-end-antipattern"></a><span data-ttu-id="891dd-103">Antipatrón Busy Front End</span><span class="sxs-lookup"><span data-stu-id="891dd-103">Busy Front End antipattern</span></span>

<span data-ttu-id="891dd-104">Realizar trabajo asincrónico en un gran número de subprocesos en segundo plano puede privar de recursos a otras tareas simultáneas en primer plano y, por tanto, reducir los tiempos de respuesta a niveles inaceptables.</span><span class="sxs-lookup"><span data-stu-id="891dd-104">Performing asynchronous work on a large number of background threads can starve other concurrent foreground tasks of resources, decreasing response times to unacceptable levels.</span></span>

## <a name="problem-description"></a><span data-ttu-id="891dd-105">Descripción del problema</span><span class="sxs-lookup"><span data-stu-id="891dd-105">Problem description</span></span>

<span data-ttu-id="891dd-106">Las tareas que consumen muchos recursos pueden aumentar los tiempos de respuesta para las solicitudes de usuario y producir latencia alta.</span><span class="sxs-lookup"><span data-stu-id="891dd-106">Resource-intensive tasks can increase the response times for user requests and cause high latency.</span></span> <span data-ttu-id="891dd-107">Una manera de mejorar los tiempos de respuesta es descargar las tareas que consumen muchos recursos en un subproceso independiente.</span><span class="sxs-lookup"><span data-stu-id="891dd-107">One way to improve response times is to offload a resource-intensive task to a separate thread.</span></span> <span data-ttu-id="891dd-108">Este enfoque permite a la aplicación responder mientras se produce el procesamiento en segundo plano.</span><span class="sxs-lookup"><span data-stu-id="891dd-108">This approach lets the application stay responsive while processing happens in the background.</span></span> <span data-ttu-id="891dd-109">Sin embargo, las tareas que se ejecutan en un subproceso en segundo plano siguen consumiendo recursos.</span><span class="sxs-lookup"><span data-stu-id="891dd-109">However, tasks that run on a background thread still consume resources.</span></span> <span data-ttu-id="891dd-110">Si hay muchos, pueden privar a los subprocesos que se encargan de las solicitudes.</span><span class="sxs-lookup"><span data-stu-id="891dd-110">If there are too many of them, they can starve the threads that are handling requests.</span></span>

> [!NOTE]
> <span data-ttu-id="891dd-111">El término *recurso* puede abarcar muchas cosas, por ejemplo, uso de CPU, ocupación de la memoria y E/O de red o de disco.</span><span class="sxs-lookup"><span data-stu-id="891dd-111">The term *resource* can encompass many things, such as CPU utilization, memory occupancy, and network or disk I/O.</span></span>

<span data-ttu-id="891dd-112">Este problema se suele producir cuando se desarrolla una aplicación como fragmento de código monolítico, con toda la lógica empresarial combinada en un solo nivel compartido con la capa de presentación.</span><span class="sxs-lookup"><span data-stu-id="891dd-112">This problem typically occurs when an application is developed as monolithic piece of code, with all of the business logic combined into a single tier shared with the presentation layer.</span></span>

<span data-ttu-id="891dd-113">Este es un ejemplo del uso de ASP.NET que ilustra el problema.</span><span class="sxs-lookup"><span data-stu-id="891dd-113">Here’s an example using ASP.NET that demonstrates the problem.</span></span> <span data-ttu-id="891dd-114">Puede encontrar el ejemplo completo [aquí][code-sample].</span><span class="sxs-lookup"><span data-stu-id="891dd-114">You can find the complete sample [here][code-sample].</span></span>

```csharp
public class WorkInFrontEndController : ApiController
{
    [HttpPost]
    [Route("api/workinfrontend")]
    public HttpResponseMessage Post()
    {
        new Thread(() =>
        {
            //Simulate processing
            Thread.SpinWait(Int32.MaxValue / 100);
        }).Start();

        return Request.CreateResponse(HttpStatusCode.Accepted);
    }
}

public class UserProfileController : ApiController
{
    [HttpGet]
    [Route("api/userprofile/{id}")]
    public UserProfile Get(int id)
    {
        //Simulate processing
        return new UserProfile() { FirstName = "Alton", LastName = "Hudgens" };
    }
}
```

- <span data-ttu-id="891dd-115">El método `Post` del controlador `WorkInFrontEnd` implementa una operación HTTP POST.</span><span class="sxs-lookup"><span data-stu-id="891dd-115">The `Post` method in the `WorkInFrontEnd` controller implements an HTTP POST operation.</span></span> <span data-ttu-id="891dd-116">Esta operación simula una tarea de ejecución prolongada que consume mucha CPU.</span><span class="sxs-lookup"><span data-stu-id="891dd-116">This operation simulates a long-running, CPU-intensive task.</span></span> <span data-ttu-id="891dd-117">El trabajo se realiza en un subproceso independiente para intentar que la operación POST se complete rápidamente.</span><span class="sxs-lookup"><span data-stu-id="891dd-117">The work is performed on a separate thread, in an attempt to enable the POST operation to complete quickly.</span></span>

- <span data-ttu-id="891dd-118">El método `Get` del controlador `UserProfile` implementa una operación HTTP GET.</span><span class="sxs-lookup"><span data-stu-id="891dd-118">The `Get` method in the `UserProfile` controller implements an HTTP GET operation.</span></span> <span data-ttu-id="891dd-119">Este método consume mucha menos CPU.</span><span class="sxs-lookup"><span data-stu-id="891dd-119">This method is much less CPU intensive.</span></span>

<span data-ttu-id="891dd-120">La principal preocupación es la necesidad de recursos del método `Post`.</span><span class="sxs-lookup"><span data-stu-id="891dd-120">The primary concern is the resource requirements of the `Post` method.</span></span> <span data-ttu-id="891dd-121">Aunque coloca el trabajo en un subproceso en segundo plano, puede seguir consumiendo importantes recursos de CPU.</span><span class="sxs-lookup"><span data-stu-id="891dd-121">Although it puts the work onto a background thread, the work can still consume considerable CPU resources.</span></span> <span data-ttu-id="891dd-122">Estos recursos se comparten con otras operaciones que lleven a cabo otros usuarios a la vez.</span><span class="sxs-lookup"><span data-stu-id="891dd-122">These resources are shared with other operations being performed by other concurrent users.</span></span> <span data-ttu-id="891dd-123">Si un número moderado de usuarios envía esta solicitud al mismo tiempo, es probable que el rendimiento general se vea afectado y, por tanto, se ralenticen todas las operaciones.</span><span class="sxs-lookup"><span data-stu-id="891dd-123">If a moderate number of users send this request at the same time, overall performance is likely to suffer, slowing down all operations.</span></span> <span data-ttu-id="891dd-124">Los usuarios pueden experimentar una notable latencia con el método `Get`, por ejemplo.</span><span class="sxs-lookup"><span data-stu-id="891dd-124">Users might experience significant latency in the `Get` method, for example.</span></span>

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="891dd-125">Procedimiento para corregir el problema</span><span class="sxs-lookup"><span data-stu-id="891dd-125">How to fix the problem</span></span>

<span data-ttu-id="891dd-126">Mueva los procesos que consumen muchos recursos a un back-end independiente.</span><span class="sxs-lookup"><span data-stu-id="891dd-126">Move processes that consume significant resources to a separate back end.</span></span> 

<span data-ttu-id="891dd-127">Con este enfoque, el front-end coloca las tareas que consumen muchos recursos en una cola de mensajes.</span><span class="sxs-lookup"><span data-stu-id="891dd-127">With this approach, the front end puts resource-intensive tasks onto a message queue.</span></span> <span data-ttu-id="891dd-128">El back-end recoge las tareas para el procesamiento asincrónico.</span><span class="sxs-lookup"><span data-stu-id="891dd-128">The back end picks up the tasks for asynchronous processing.</span></span> <span data-ttu-id="891dd-129">La cola también actúa como un nivelador de la carga, al almacenar en búfer las solicitudes para el back-end.</span><span class="sxs-lookup"><span data-stu-id="891dd-129">The queue also acts as a load leveler, buffering requests for the back end.</span></span> <span data-ttu-id="891dd-130">Si la longitud de cola es demasiado larga, puede configurar el escalado automático para escalar horizontalmente el back-end.</span><span class="sxs-lookup"><span data-stu-id="891dd-130">If the queue length becomes too long, you can configure autoscaling to scale out the back end.</span></span>

<span data-ttu-id="891dd-131">Esta es una versión revisada del código anterior.</span><span class="sxs-lookup"><span data-stu-id="891dd-131">Here is a revised version of the previous code.</span></span> <span data-ttu-id="891dd-132">En esta versión, el método `Post` coloca un mensaje en una cola de Service Bus.</span><span class="sxs-lookup"><span data-stu-id="891dd-132">In this version, the `Post` method puts a message on a Service Bus queue.</span></span> 

```csharp
public class WorkInBackgroundController : ApiController
{
    private static readonly QueueClient QueueClient;
    private static readonly string QueueName;
    private static readonly ServiceBusQueueHandler ServiceBusQueueHandler;

    public WorkInBackgroundController()
    {
        var serviceBusConnectionString = ...;
        QueueName = ...;
        ServiceBusQueueHandler = new ServiceBusQueueHandler(serviceBusConnectionString);
        QueueClient = ServiceBusQueueHandler.GetQueueClientAsync(QueueName).Result;
    }

    [HttpPost]
    [Route("api/workinbackground")]
    public async Task<long> Post()
    {
        return await ServiceBusQueuehandler.AddWorkLoadToQueueAsync(QueueClient, QueueName, 0);
    }
}
```

<span data-ttu-id="891dd-133">El back-end extrae los mensajes de la cola de Service Bus y lleva a cabo el procesamiento.</span><span class="sxs-lookup"><span data-stu-id="891dd-133">The back end pulls messages from the Service Bus queue and does the processing.</span></span>

```csharp
public async Task RunAsync(CancellationToken cancellationToken)
{
    this._queueClient.OnMessageAsync(
        // This lambda is invoked for each message received.
        async (receivedMessage) =>
        {
            try
            {
                // Simulate processing of message
                Thread.SpinWait(Int32.Maxvalue / 1000);

                await receivedMessage.CompleteAsync();
            }
            catch
            {
                receivedMessage.Abandon();
            }
        });
}
```

## <a name="considerations"></a><span data-ttu-id="891dd-134">Consideraciones</span><span class="sxs-lookup"><span data-stu-id="891dd-134">Considerations</span></span>

- <span data-ttu-id="891dd-135">Este enfoque agrega cierta complejidad adicional a la aplicación.</span><span class="sxs-lookup"><span data-stu-id="891dd-135">This approach adds some additional complexity to the application.</span></span> <span data-ttu-id="891dd-136">Incorpore los mensajes a la cola y sáquelos con cuidado para evitar la pérdida de solicitudes en caso de fallo.</span><span class="sxs-lookup"><span data-stu-id="891dd-136">You must handle queuing and dequeuing safely to avoid losing requests in the event of a failure.</span></span>
- <span data-ttu-id="891dd-137">La aplicación depende de un servicio adicional para la cola de mensajes.</span><span class="sxs-lookup"><span data-stu-id="891dd-137">The application takes a dependency on an additional service for the message queue.</span></span>
- <span data-ttu-id="891dd-138">El entorno de procesamiento debe ser lo suficientemente escalable para controlar la carga de trabajo esperada y cumplir los objetivos de rendimiento necesarios.</span><span class="sxs-lookup"><span data-stu-id="891dd-138">The processing environment must be sufficiently scalable to handle the expected workload and meet the required throughput targets.</span></span>
- <span data-ttu-id="891dd-139">Aunque este enfoque debe mejorar la capacidad de respuesta general, las tareas que se han movido al back-end pueden tardar más en completarse.</span><span class="sxs-lookup"><span data-stu-id="891dd-139">While this approach should improve overall responsiveness, the tasks that are moved to the back end may take longer to complete.</span></span> 

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="891dd-140">Procedimiento para detectar el problema</span><span class="sxs-lookup"><span data-stu-id="891dd-140">How to detect the problem</span></span>

<span data-ttu-id="891dd-141">Los síntomas de Busy Front End incluyen latencia alta mientras se realizan tareas que consumen muchos recursos.</span><span class="sxs-lookup"><span data-stu-id="891dd-141">Symptoms of a busy front end include high latency when resource-intensive tasks are being performed.</span></span> <span data-ttu-id="891dd-142">Es probable que los usuarios finales informen de tiempos de respuesta prolongados o errores provocados por el agotamiento del tiempo de espera de los servicios. Estos errores también podrían devolver errores HTTP 500 (servidor interno) o errores HTTP 503 (servicio no disponible).</span><span class="sxs-lookup"><span data-stu-id="891dd-142">End users are likely to report extended response times or failures caused by services timing out. These failures could also return HTTP 500 (Internal Server) errors or HTTP 503 (Service Unavailable) errors.</span></span> <span data-ttu-id="891dd-143">Examine los registros de eventos del servidor web, probablemente contengan información más detallada sobre las causas y las circunstancias de los errores.</span><span class="sxs-lookup"><span data-stu-id="891dd-143">Examine the event logs for the web server, which are likely to contain more detailed information about the causes and circumstances of the errors.</span></span>

<span data-ttu-id="891dd-144">Puede realizar los pasos siguientes para ayudar a identificar este problema:</span><span class="sxs-lookup"><span data-stu-id="891dd-144">You can perform the following steps to help identify this problem:</span></span>

1. <span data-ttu-id="891dd-145">Supervise los procesos del sistema de producción para identificar los puntos en los que se ralentizan los tiempos de respuesta.</span><span class="sxs-lookup"><span data-stu-id="891dd-145">Perform process monitoring of the production system, to identify points when response times slow down.</span></span>
2. <span data-ttu-id="891dd-146">Examine los datos de telemetría capturados en estos puntos para determinar la combinación de operaciones que se están realizando y los recursos que se utilizan.</span><span class="sxs-lookup"><span data-stu-id="891dd-146">Examine the telemetry data captured at these points to determine the mix of operations being performed and the resources being used.</span></span> 
3. <span data-ttu-id="891dd-147">Busque correlaciones entre los tiempos de respuesta lentos y los volúmenes, y las combinaciones de operaciones que se producen en esos momentos.</span><span class="sxs-lookup"><span data-stu-id="891dd-147">Find any correlations between poor response times and the volumes and combinations of operations that were happening at those times.</span></span>
4. <span data-ttu-id="891dd-148">Realice una prueba de carga con cada operación sospechosa para identificar las operaciones que consumen recursos y privan de estos a otras operaciones.</span><span class="sxs-lookup"><span data-stu-id="891dd-148">Load test each suspected operation to identify which operations are consuming resources and starving other operations.</span></span> 
5. <span data-ttu-id="891dd-149">Revise el código fuente de esas operaciones para determinar por qué provocan el exceso de consumo de recursos.</span><span class="sxs-lookup"><span data-stu-id="891dd-149">Review the source code for those operations to determine why they might cause excessive resource consumption.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="891dd-150">Diagnóstico de ejemplo</span><span class="sxs-lookup"><span data-stu-id="891dd-150">Example diagnosis</span></span> 

<span data-ttu-id="891dd-151">En las secciones siguientes se aplican estos pasos para la aplicación de ejemplo descrita anteriormente.</span><span class="sxs-lookup"><span data-stu-id="891dd-151">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="identify-points-of-slowdown"></a><span data-ttu-id="891dd-152">Identificación de los puntos de ralentización</span><span class="sxs-lookup"><span data-stu-id="891dd-152">Identify points of slowdown</span></span>

<span data-ttu-id="891dd-153">Instrumente los métodos para realizar el seguimiento de la duración y los recursos consumidos por cada solicitud.</span><span class="sxs-lookup"><span data-stu-id="891dd-153">Instrument each method to track the duration and resources consumed by each request.</span></span> <span data-ttu-id="891dd-154">Después, supervise la aplicación en producción.</span><span class="sxs-lookup"><span data-stu-id="891dd-154">Then monitor the application in production.</span></span> <span data-ttu-id="891dd-155">Esto puede proporcionar una vista general de la competencia de las solicitudes entre sí.</span><span class="sxs-lookup"><span data-stu-id="891dd-155">This can provide an overall view of how requests compete with each other.</span></span> <span data-ttu-id="891dd-156">Durante los períodos de esfuerzo, las solicitudes de ejecución lenta que consumen muchos recursos probablemente afectarán a otras operaciones; este comportamiento se puede observar al supervisar el sistema y ver la disminución en el rendimiento.</span><span class="sxs-lookup"><span data-stu-id="891dd-156">During periods of stress, slow-running resource-hungry requests will likely impact other operations, and this behavior can be observed by monitoring the system and noting the drop off in performance.</span></span>

<span data-ttu-id="891dd-157">La siguiente imagen muestra un panel de supervisión.</span><span class="sxs-lookup"><span data-stu-id="891dd-157">The following image shows a monitoring dashboard.</span></span> <span data-ttu-id="891dd-158">(Se usa [AppDynamics] para las pruebas). Inicialmente, el sistema tiene poca carga.</span><span class="sxs-lookup"><span data-stu-id="891dd-158">(We used [AppDynamics] for our tests.) Initially, the system has light load.</span></span> <span data-ttu-id="891dd-159">A continuación, los usuarios inician la solicitud del método GET `UserProfile`.</span><span class="sxs-lookup"><span data-stu-id="891dd-159">Then users start requesting the `UserProfile` GET method.</span></span> <span data-ttu-id="891dd-160">El rendimiento es razonablemente bueno hasta que otros usuarios inician la emisión de solicitudes del método POST `WorkInFrontEnd`.</span><span class="sxs-lookup"><span data-stu-id="891dd-160">The performance is reasonably good until other users start issuing requests to the `WorkInFrontEnd` POST method.</span></span> <span data-ttu-id="891dd-161">En ese momento, aumentan considerablemente los tiempos de respuesta (primera flecha).</span><span class="sxs-lookup"><span data-stu-id="891dd-161">At that point, response times increase dramatically (first arrow).</span></span> <span data-ttu-id="891dd-162">Los tiempos de respuesta solo mejoran cuando el volumen de solicitudes realizadas al controlador `WorkInFrontEnd` disminuye (segunda flecha).</span><span class="sxs-lookup"><span data-stu-id="891dd-162">Response times only improve after the volume of requests to the `WorkInFrontEnd` controller diminishes (second arrow).</span></span>

![Panel de las transacciones comerciales de AppDynamics que muestra los efectos en los tiempos de respuesta de todas las solicitudes cuando se utiliza el controlador WorkInFrontEnd][AppDynamics-Transactions-Front-End-Requests]

### <a name="examine-telemetry-data-and-find-correlations"></a><span data-ttu-id="891dd-164">Examen de los datos de telemetría y búsqueda de correlaciones</span><span class="sxs-lookup"><span data-stu-id="891dd-164">Examine telemetry data and find correlations</span></span>

<span data-ttu-id="891dd-165">La imagen siguiente muestra algunas de las métricas que se recopilan para supervisar la utilización de recursos durante el mismo intervalo.</span><span class="sxs-lookup"><span data-stu-id="891dd-165">The next image shows some of the metrics gathered to monitor resource utilization during the same interval.</span></span> <span data-ttu-id="891dd-166">En primer lugar, pocos usuarios acceden al sistema.</span><span class="sxs-lookup"><span data-stu-id="891dd-166">At first, few users are accessing the system.</span></span> <span data-ttu-id="891dd-167">Según se van conectando más usuarios, el uso de CPU se vuelve muy alto (el 100 %).</span><span class="sxs-lookup"><span data-stu-id="891dd-167">As more users connect, CPU utilization becomes very high (100%).</span></span> <span data-ttu-id="891dd-168">Observe también que la velocidad de E/S de red aumenta inicialmente con el uso de CPU.</span><span class="sxs-lookup"><span data-stu-id="891dd-168">Also notice that the network I/O rate initially goes up as CPU usage rises.</span></span> <span data-ttu-id="891dd-169">Sin embargo, una vez que se alcanza el uso de CPU máximo, el flujo de E/S de red realmente disminuye.</span><span class="sxs-lookup"><span data-stu-id="891dd-169">But once CPU usage peaks, network I/O actually goes down.</span></span> <span data-ttu-id="891dd-170">Eso se debe a que el sistema solo puede controlar un número relativamente pequeño de solicitudes con la capacidad de CPU al máximo.</span><span class="sxs-lookup"><span data-stu-id="891dd-170">That's because the system can only handle a relatively small number of requests once the CPU is at capacity.</span></span> <span data-ttu-id="891dd-171">Según se desconectan los usuarios, la carga de CPU disminuye.</span><span class="sxs-lookup"><span data-stu-id="891dd-171">As users disconnect, the CPU load tails off.</span></span>

![Métricas de AppDynamics que muestran el uso de CPU y de red][AppDynamics-Metrics-Front-End-Requests]

<span data-ttu-id="891dd-173">En este punto, parece que el método `Post` del controlador `WorkInFrontEnd` es un candidato ideal para un examen más minucioso.</span><span class="sxs-lookup"><span data-stu-id="891dd-173">At this point, it appears the `Post` method in the `WorkInFrontEnd` controller is a prime candidate for closer examination.</span></span> <span data-ttu-id="891dd-174">Es necesario seguir trabajando en un entorno controlado para confirmar la hipótesis.</span><span class="sxs-lookup"><span data-stu-id="891dd-174">Further work in a controlled environment is needed to confirm the hypothesis.</span></span>

### <a name="perform-load-testing"></a><span data-ttu-id="891dd-175">Pruebas de carga</span><span class="sxs-lookup"><span data-stu-id="891dd-175">Perform load testing</span></span> 

<span data-ttu-id="891dd-176">El paso siguiente es realizar pruebas en un entorno controlado.</span><span class="sxs-lookup"><span data-stu-id="891dd-176">The next step is to perform tests in a controlled environment.</span></span> <span data-ttu-id="891dd-177">Por ejemplo, la ejecución de una serie de pruebas de carga que incluya y luego omita las solicitudes una a una, para ver el resultado.</span><span class="sxs-lookup"><span data-stu-id="891dd-177">For example, run a series of load tests that include and then omit each request in turn to see the effects.</span></span>

<span data-ttu-id="891dd-178">El gráfico siguiente muestra los resultados de una prueba de carga en una implementación idéntica a la del servicio en la nube utilizado en las pruebas anteriores.</span><span class="sxs-lookup"><span data-stu-id="891dd-178">The graph below shows the results of a load test performed against an identical deployment of the cloud service used in the previous tests.</span></span> <span data-ttu-id="891dd-179">Para la prueba se ha utilizado una carga constante de 500 usuarios que realizan la operación `Get` en el controlador `UserProfile`, junto con una carga por pasos de usuarios que realizan la operación `Post` en el controlador `WorkInFrontEnd`.</span><span class="sxs-lookup"><span data-stu-id="891dd-179">The test used a constant load of 500 users performing the `Get` operation in the `UserProfile` controller, along with a step load of users performing the `Post` operation in the `WorkInFrontEnd` controller.</span></span> 

![Resultados de las pruebas de carga iniciales para el controlador WorkInFrontEnd][Initial-Load-Test-Results-Front-End]

<span data-ttu-id="891dd-181">Inicialmente, la carga por pasos es 0, por lo que los únicos usuarios activos realizan solicitudes `UserProfile`.</span><span class="sxs-lookup"><span data-stu-id="891dd-181">Initially, the step load is 0, so the only active users are performing the `UserProfile` requests.</span></span> <span data-ttu-id="891dd-182">El sistema responde aproximadamente 500 solicitudes por segundo.</span><span class="sxs-lookup"><span data-stu-id="891dd-182">The system is able to respond to approximately 500 requests per second.</span></span> <span data-ttu-id="891dd-183">A los 60 segundos, una carga de 100 usuarios adicionales comienza a enviar solicitudes POST al controlador `WorkInFrontEnd`.</span><span class="sxs-lookup"><span data-stu-id="891dd-183">After 60 seconds, a load of 100 additional users starts sending POST requests to the `WorkInFrontEnd` controller.</span></span> <span data-ttu-id="891dd-184">Casi de inmediato, la carga de trabajo enviada al controlador `UserProfile` desciende a unas 150 solicitudes por segundo.</span><span class="sxs-lookup"><span data-stu-id="891dd-184">Almost immediately, the workload sent to the `UserProfile` controller drops to about 150 requests per second.</span></span> <span data-ttu-id="891dd-185">Esto es debido a la manera en que funciona el ejecutor de pruebas de carga.</span><span class="sxs-lookup"><span data-stu-id="891dd-185">This is due to the way the load-test runner functions.</span></span> <span data-ttu-id="891dd-186">Para enviar la solicitud siguiente espera una respuesta, por lo que cuanto más tarde en recibirla, menor será la tasa de la solicitud.</span><span class="sxs-lookup"><span data-stu-id="891dd-186">It waits for a response before sending the next request, so the longer it takes to receive a response, the lower the request rate.</span></span>

<span data-ttu-id="891dd-187">Cuantos más usuarios envíen solicitudes POST al controlador `WorkInFrontEnd`, menor será la tasa de respuesta del controlador `UserProfile`.</span><span class="sxs-lookup"><span data-stu-id="891dd-187">As more users send POST requests to the `WorkInFrontEnd` controller, the response rate of the `UserProfile` controller continues to drop.</span></span> <span data-ttu-id="891dd-188">Pero tenga en cuenta que el volumen de solicitudes de las que se ocupa el controlador `WorkInFrontEnd` se mantiene relativamente constante.</span><span class="sxs-lookup"><span data-stu-id="891dd-188">But note that the volume of requests handled by the `WorkInFrontEnd`controller remains relatively constant.</span></span> <span data-ttu-id="891dd-189">La saturación del sistema se vuelve evidente a medida que la velocidad global de ambas solicitudes tiende hacia un límite fijo pero bajo.</span><span class="sxs-lookup"><span data-stu-id="891dd-189">The saturation of the system becomes apparent as the overall rate of both requests tends towards a steady but low limit.</span></span>


### <a name="review-the-source-code"></a><span data-ttu-id="891dd-190">Revisión del código fuente</span><span class="sxs-lookup"><span data-stu-id="891dd-190">Review the source code</span></span>

<span data-ttu-id="891dd-191">El último paso es observar el código fuente.</span><span class="sxs-lookup"><span data-stu-id="891dd-191">The final step is to look at the source code.</span></span> <span data-ttu-id="891dd-192">El equipo de desarrollo tenía constancia que el método `Post` puede tardar un tiempo considerable, por lo cual, en la implementación original se utiliza un subproceso independiente.</span><span class="sxs-lookup"><span data-stu-id="891dd-192">The development team was aware that the `Post` method could take a considerable amount of time, which is why the original implementation used a separate thread.</span></span> <span data-ttu-id="891dd-193">El problema inmediato se ha resuelto, ya que el método `Post` no se bloqueaba mientras esperaba que terminaran las tareas prolongadas.</span><span class="sxs-lookup"><span data-stu-id="891dd-193">That solved the immediate problem, because the `Post` method did not block waiting for a long-running task to complete.</span></span>

<span data-ttu-id="891dd-194">Sin embargo, el trabajo que realiza este método sigue consumiendo CPU, memoria y otros recursos.</span><span class="sxs-lookup"><span data-stu-id="891dd-194">However, the work performed by this method still consumes CPU, memory, and other resources.</span></span> <span data-ttu-id="891dd-195">Habilitar este proceso para que se ejecute de forma asincrónica en realidad puede perjudicar al rendimiento, ya que los usuarios pueden desencadenar numerosas operaciones de estas al mismo tiempo, sin control.</span><span class="sxs-lookup"><span data-stu-id="891dd-195">Enabling this process to run asynchronously might actually damage performance, as users can trigger a large number of these operations simultaneously, in an uncontrolled manner.</span></span> <span data-ttu-id="891dd-196">El número de subprocesos que puede ejecutar un servidor está limitado.</span><span class="sxs-lookup"><span data-stu-id="891dd-196">There is a limit to the number of threads that a server can run.</span></span> <span data-ttu-id="891dd-197">Más allá de este límite, es probable que la aplicación reciba una excepción al intentar iniciar un nuevo subproceso.</span><span class="sxs-lookup"><span data-stu-id="891dd-197">Past this limit, the application is likely to get an exception when it tries to start a new thread.</span></span>

> [!NOTE]
> <span data-ttu-id="891dd-198">Esto no significa que deba evitar las operaciones asincrónicas.</span><span class="sxs-lookup"><span data-stu-id="891dd-198">This doesn't mean you should avoid asynchronous operations.</span></span> <span data-ttu-id="891dd-199">Realizar una asincrónica await en una llamada de red es un procedimiento recomendado.</span><span class="sxs-lookup"><span data-stu-id="891dd-199">Performing an asynchronous await on a network call is a recommended practice.</span></span> <span data-ttu-id="891dd-200">(Consulte el antipatrón [de E/S sincrónico][sync-io]). Este problema es que el trabajo que consume mucha CPU se generó en otro subproceso.</span><span class="sxs-lookup"><span data-stu-id="891dd-200">(See the [Synchronous I/O][sync-io] antipattern.) The problem here is that CPU-intensive work was spawned on another thread.</span></span> 

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="891dd-201">Implementación de la solución y comprobación del resultado</span><span class="sxs-lookup"><span data-stu-id="891dd-201">Implement the solution and verify the result</span></span>

<span data-ttu-id="891dd-202">En la siguiente imagen se ilustra la supervisión del rendimiento una vez implementada la solución.</span><span class="sxs-lookup"><span data-stu-id="891dd-202">The following image shows performance monitoring after the solution was implemented.</span></span> <span data-ttu-id="891dd-203">La carga era similar a la que anterior, pero los tiempos de respuesta para el controlador `UserProfile` ahora son mucho más rápidos.</span><span class="sxs-lookup"><span data-stu-id="891dd-203">The load was similar to that shown earlier, but the response times for the `UserProfile` controller are now much faster.</span></span> <span data-ttu-id="891dd-204">El volumen de solicitudes aumentó de 2759 a 23 565 en el mismo periodo.</span><span class="sxs-lookup"><span data-stu-id="891dd-204">The volume of requests increased over the same duration, from 2,759 to 23,565.</span></span> 

![Panel de las transacciones comerciales de AppDynamics que muestra los efectos en los tiempos de respuesta de todas las solicitudes cuando se utiliza el controlador WorkInBackground][AppDynamics-Transactions-Background-Requests]

<span data-ttu-id="891dd-206">Tenga en cuenta que el controlador `WorkInBackground` también controlaba un volumen de solicitudes mucho mayor.</span><span class="sxs-lookup"><span data-stu-id="891dd-206">Note that the `WorkInBackground` controller also handled a much larger volume of requests.</span></span> <span data-ttu-id="891dd-207">Sin embargo, no se puede realizar una comparación directa en este caso, puesto que el trabajo que se realizar en este controlador es muy diferente del código original.</span><span class="sxs-lookup"><span data-stu-id="891dd-207">However, you can't make a direct comparison in this case, because the work being performed in this controller is very different from the original code.</span></span> <span data-ttu-id="891dd-208">La nueva versión simplemente pone en cola una solicitud, en lugar de realizar un cálculo lento.</span><span class="sxs-lookup"><span data-stu-id="891dd-208">The new version simply queues a request, rather than performing a time consuming calculation.</span></span> <span data-ttu-id="891dd-209">La cuestión principal es que este método ya no arrastra todo el sistema con la carga.</span><span class="sxs-lookup"><span data-stu-id="891dd-209">The main point is that this method no longer drags down the entire system under load.</span></span>

<span data-ttu-id="891dd-210">El uso de CPU y de red también muestran mejor rendimiento.</span><span class="sxs-lookup"><span data-stu-id="891dd-210">CPU and network utilization also show the improved performance.</span></span> <span data-ttu-id="891dd-211">El uso de CPU nunca alcanza el 100 %, y el volumen de solicitudes de red administrado es mucho mayor que el anterior y no disminuye hasta que lo haga la carga de trabajo.</span><span class="sxs-lookup"><span data-stu-id="891dd-211">The CPU utilization never reached 100%, and the volume of handled network requests was far greater than earlier, and did not tail off until the workload dropped.</span></span>

![Métricas de AppDynamics que muestran el uso de CPU y de red del controlador WorkInBackground][AppDynamics-Metrics-Background-Requests]

<span data-ttu-id="891dd-213">En el siguiente gráfico se muestran los resultados de una prueba de carga.</span><span class="sxs-lookup"><span data-stu-id="891dd-213">The following graph shows the results of a load test.</span></span> <span data-ttu-id="891dd-214">El volumen total de solicitudes atendidas ha mejorado notablemente en comparación con las pruebas anteriores.</span><span class="sxs-lookup"><span data-stu-id="891dd-214">The overall volume of requests serviced is greatly improved compared to the the earlier tests.</span></span>

![Resultados de las pruebas de carga del controlador BackgroundImageProcessing][Load-Test-Results-Background]

## <a name="related-guidance"></a><span data-ttu-id="891dd-216">Instrucciones relacionadas</span><span class="sxs-lookup"><span data-stu-id="891dd-216">Related guidance</span></span>

- <span data-ttu-id="891dd-217">[Procedimientos recomendados de escalado automático][autoscaling]</span><span class="sxs-lookup"><span data-stu-id="891dd-217">[Autoscaling best practices][autoscaling]</span></span>
- <span data-ttu-id="891dd-218">[Procedimientos recomendados para los trabajos en segundo plano][background-jobs]</span><span class="sxs-lookup"><span data-stu-id="891dd-218">[Background jobs best practices][background-jobs]</span></span>
- <span data-ttu-id="891dd-219">[Queue-Based Load Leveling pattern][load-leveling] (Patrón de equilibrio de carga basado en colas)</span><span class="sxs-lookup"><span data-stu-id="891dd-219">[Queue-Based Load Leveling pattern][load-leveling]</span></span>
- <span data-ttu-id="891dd-220">[Web Queue Worker architecture style][web-queue-worker] (Estilo de arquitectura web-cola-trabajo)</span><span class="sxs-lookup"><span data-stu-id="891dd-220">[Web Queue Worker architecture style][web-queue-worker]</span></span>

[AppDyanamics]: https://www.appdynamics.com/
[autoscaling]: /azure/architecture/best-practices/auto-scaling
[background-jobs]: /azure/architecture/best-practices/background-jobs
[code-sample]: https://github.com/mspnp/performance-optimization/tree/master/BusyFrontEnd
[fullDemonstrationOfSolution]: https://github.com/mspnp/performance-optimization/tree/master/BusyFrontEnd
[load-leveling]: /azure/architecture/patterns/queue-based-load-leveling
[sync-io]: ../synchronous-io/index.md
[web-queue-worker]: /azure/architecture/guide/architecture-styles/web-queue-worker

[WebJobs]: https://www.hanselman.com/blog/IntroducingWindowsAzureWebJobs.aspx
[ComputePartitioning]: https://msdn.microsoft.com/library/dn589773.aspx
[ServiceBusQueues]: https://msdn.microsoft.com/library/azure/hh367516.aspx
[AppDynamics-Transactions-Front-End-Requests]: ./_images/AppDynamicsPerformanceStats.jpg
[AppDynamics-Metrics-Front-End-Requests]: ./_images/AppDynamicsFrontEndMetrics.jpg
[Initial-Load-Test-Results-Front-End]: ./_images/InitialLoadTestResultsFrontEnd.jpg
[AppDynamics-Transactions-Background-Requests]: ./_images/AppDynamicsBackgroundPerformanceStats.jpg
[AppDynamics-Metrics-Background-Requests]: ./_images/AppDynamicsBackgroundMetrics.jpg
[Load-Test-Results-Background]: ./_images/LoadTestResultsBackground.jpg


