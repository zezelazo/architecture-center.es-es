---
title: Antipatrón Busy Front End
titleSuffix: Performance antipatterns for cloud apps
description: El trabajo asincrónico en un gran número de subprocesos en segundo plano puede privar de recursos a otras tareas de primer plano.
author: dragon119
ms.date: 06/05/2017
ms.custom: seodec18
ms.openlocfilehash: f52cedde5a17f098fb9218c48479fae981a2c7df
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/04/2019
ms.locfileid: "54011521"
---
# <a name="busy-front-end-antipattern"></a>Antipatrón Busy Front End

Realizar trabajo asincrónico en un gran número de subprocesos en segundo plano puede privar de recursos a otras tareas simultáneas en primer plano y, por tanto, reducir los tiempos de respuesta a niveles inaceptables.

## <a name="problem-description"></a>Descripción del problema

Las tareas que consumen muchos recursos pueden aumentar los tiempos de respuesta para las solicitudes de usuario y producir latencia alta. Una manera de mejorar los tiempos de respuesta es descargar las tareas que consumen muchos recursos en un subproceso independiente. Este enfoque permite a la aplicación responder mientras se produce el procesamiento en segundo plano. Sin embargo, las tareas que se ejecutan en un subproceso en segundo plano siguen consumiendo recursos. Si hay muchos, pueden privar a los subprocesos que se encargan de las solicitudes.

> [!NOTE]
> El término *recurso* puede abarcar muchas cosas, por ejemplo, uso de CPU, ocupación de la memoria y E/O de red o de disco.

Este problema se suele producir cuando se desarrolla una aplicación como fragmento de código monolítico, con toda la lógica empresarial combinada en un solo nivel compartido con la capa de presentación.

Este es un ejemplo del uso de ASP.NET que ilustra el problema. Puede encontrar el ejemplo completo [aquí][code-sample].

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

- El método `Post` del controlador `WorkInFrontEnd` implementa una operación HTTP POST. Esta operación simula una tarea de ejecución prolongada que consume mucha CPU. El trabajo se realiza en un subproceso independiente para intentar que la operación POST se complete rápidamente.

- El método `Get` del controlador `UserProfile` implementa una operación HTTP GET. Este método consume mucha menos CPU.

La principal preocupación es la necesidad de recursos del método `Post`. Aunque coloca el trabajo en un subproceso en segundo plano, puede seguir consumiendo importantes recursos de CPU. Estos recursos se comparten con otras operaciones que lleven a cabo otros usuarios a la vez. Si un número moderado de usuarios envía esta solicitud al mismo tiempo, es probable que el rendimiento general se vea afectado y, por tanto, se ralenticen todas las operaciones. Los usuarios pueden experimentar una notable latencia con el método `Get`, por ejemplo.

## <a name="how-to-fix-the-problem"></a>Procedimiento para corregir el problema

Mueva los procesos que consumen muchos recursos a un back-end independiente.

Con este enfoque, el front-end coloca las tareas que consumen muchos recursos en una cola de mensajes. El back-end recoge las tareas para el procesamiento asincrónico. La cola también actúa como un nivelador de la carga, al almacenar en búfer las solicitudes para el back-end. Si la longitud de cola es demasiado larga, puede configurar el escalado automático para escalar horizontalmente el back-end.

Esta es una versión revisada del código anterior. En esta versión, el método `Post` coloca un mensaje en una cola de Service Bus.

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

El back-end extrae los mensajes de la cola de Service Bus y lleva a cabo el procesamiento.

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

## <a name="considerations"></a>Consideraciones

- Este enfoque agrega cierta complejidad adicional a la aplicación. Incorpore los mensajes a la cola y sáquelos con cuidado para evitar la pérdida de solicitudes en caso de fallo.
- La aplicación depende de un servicio adicional para la cola de mensajes.
- El entorno de procesamiento debe ser lo suficientemente escalable para controlar la carga de trabajo esperada y cumplir los objetivos de rendimiento necesarios.
- Aunque este enfoque debe mejorar la capacidad de respuesta general, las tareas que se han movido al back-end pueden tardar más en completarse.

## <a name="how-to-detect-the-problem"></a>Procedimiento para detectar el problema

Los síntomas de Busy Front End incluyen latencia alta mientras se realizan tareas que consumen muchos recursos. Es probable que los usuarios finales informen de tiempos de respuesta prolongados o errores provocados por el agotamiento del tiempo de espera de los servicios. Estos errores también podrían devolver errores HTTP 500 (servidor interno) o errores HTTP 503 (servicio no disponible). Examine los registros de eventos del servidor web, probablemente contengan información más detallada sobre las causas y las circunstancias de los errores.

Puede realizar los pasos siguientes para ayudar a identificar este problema:

1. Supervise los procesos del sistema de producción para identificar los puntos en los que se ralentizan los tiempos de respuesta.
2. Examine los datos de telemetría capturados en estos puntos para determinar la combinación de operaciones que se están realizando y los recursos que se utilizan.
3. Busque correlaciones entre los tiempos de respuesta lentos y los volúmenes, y las combinaciones de operaciones que se producen en esos momentos.
4. Realice una prueba de carga con cada operación sospechosa para identificar las operaciones que consumen recursos y privan de estos a otras operaciones.
5. Revise el código fuente de esas operaciones para determinar por qué provocan el exceso de consumo de recursos.

## <a name="example-diagnosis"></a>Diagnóstico de ejemplo

En las secciones siguientes se aplican estos pasos para la aplicación de ejemplo descrita anteriormente.

### <a name="identify-points-of-slowdown"></a>Identificación de los puntos de ralentización

Instrumente los métodos para realizar el seguimiento de la duración y los recursos consumidos por cada solicitud. Después, supervise la aplicación en producción. Esto puede proporcionar una vista general de la competencia de las solicitudes entre sí. Durante los períodos de esfuerzo, las solicitudes de ejecución lenta que consumen muchos recursos probablemente afectarán a otras operaciones; este comportamiento se puede observar al supervisar el sistema y ver la disminución en el rendimiento.

La siguiente imagen muestra un panel de supervisión. (Se usa [AppDynamics] para las pruebas). Inicialmente, el sistema tiene poca carga. A continuación, los usuarios inician la solicitud del método GET `UserProfile`. El rendimiento es razonablemente bueno hasta que otros usuarios inician la emisión de solicitudes del método POST `WorkInFrontEnd`. En ese momento, aumentan considerablemente los tiempos de respuesta (primera flecha). Los tiempos de respuesta solo mejoran cuando el volumen de solicitudes realizadas al controlador `WorkInFrontEnd` disminuye (segunda flecha).

![Panel de las transacciones comerciales de AppDynamics que muestra los efectos en los tiempos de respuesta de todas las solicitudes cuando se utiliza el controlador WorkInFrontEnd][AppDynamics-Transactions-Front-End-Requests]

### <a name="examine-telemetry-data-and-find-correlations"></a>Examen de los datos de telemetría y búsqueda de correlaciones

La imagen siguiente muestra algunas de las métricas que se recopilan para supervisar la utilización de recursos durante el mismo intervalo. En primer lugar, pocos usuarios acceden al sistema. Según se van conectando más usuarios, el uso de CPU se vuelve muy alto (el 100 %). Observe también que la velocidad de E/S de red aumenta inicialmente con el uso de CPU. Sin embargo, una vez que se alcanza el uso de CPU máximo, el flujo de E/S de red realmente disminuye. Eso se debe a que el sistema solo puede controlar un número relativamente pequeño de solicitudes con la capacidad de CPU al máximo. Según se desconectan los usuarios, la carga de CPU disminuye.

![Métricas de AppDynamics que muestran el uso de CPU y de red][AppDynamics-Metrics-Front-End-Requests]

En este punto, parece que el método `Post` del controlador `WorkInFrontEnd` es un candidato ideal para un examen más minucioso. Es necesario seguir trabajando en un entorno controlado para confirmar la hipótesis.

### <a name="perform-load-testing"></a>Pruebas de carga

El paso siguiente es realizar pruebas en un entorno controlado. Por ejemplo, la ejecución de una serie de pruebas de carga que incluya y luego omita las solicitudes una a una, para ver el resultado.

El gráfico siguiente muestra los resultados de una prueba de carga en una implementación idéntica a la del servicio en la nube utilizado en las pruebas anteriores. Para la prueba se ha utilizado una carga constante de 500 usuarios que realizan la operación `Get` en el controlador `UserProfile`, junto con una carga por pasos de usuarios que realizan la operación `Post` en el controlador `WorkInFrontEnd`.

![Resultados de las pruebas de carga iniciales para el controlador WorkInFrontEnd][Initial-Load-Test-Results-Front-End]

Inicialmente, la carga por pasos es 0, por lo que los únicos usuarios activos realizan solicitudes `UserProfile`. El sistema responde aproximadamente 500 solicitudes por segundo. A los 60 segundos, una carga de 100 usuarios adicionales comienza a enviar solicitudes POST al controlador `WorkInFrontEnd`. Casi de inmediato, la carga de trabajo enviada al controlador `UserProfile` desciende a unas 150 solicitudes por segundo. Esto es debido a la manera en que funciona el ejecutor de pruebas de carga. Para enviar la solicitud siguiente espera una respuesta, por lo que cuanto más tarde en recibirla, menor será la tasa de la solicitud.

Cuantos más usuarios envíen solicitudes POST al controlador `WorkInFrontEnd`, menor será la tasa de respuesta del controlador `UserProfile`. Pero tenga en cuenta que el volumen de solicitudes de las que se ocupa el controlador `WorkInFrontEnd` se mantiene relativamente constante. La saturación del sistema se vuelve evidente a medida que la velocidad global de ambas solicitudes tiende hacia un límite fijo pero bajo.

### <a name="review-the-source-code"></a>Revisión del código fuente

El último paso es observar el código fuente. El equipo de desarrollo tenía constancia que el método `Post` puede tardar un tiempo considerable, por lo cual, en la implementación original se utiliza un subproceso independiente. El problema inmediato se ha resuelto, ya que el método `Post` no se bloqueaba mientras esperaba que terminaran las tareas prolongadas.

Sin embargo, el trabajo que realiza este método sigue consumiendo CPU, memoria y otros recursos. Habilitar este proceso para que se ejecute de forma asincrónica en realidad puede perjudicar al rendimiento, ya que los usuarios pueden desencadenar numerosas operaciones de estas al mismo tiempo, sin control. El número de subprocesos que puede ejecutar un servidor está limitado. Más allá de este límite, es probable que la aplicación reciba una excepción al intentar iniciar un nuevo subproceso.

> [!NOTE]
> Esto no significa que deba evitar las operaciones asincrónicas. Realizar una asincrónica await en una llamada de red es un procedimiento recomendado. (Consulte el antipatrón [de E/S sincrónico][sync-io]). Este problema es que el trabajo que consume mucha CPU se generó en otro subproceso.

### <a name="implement-the-solution-and-verify-the-result"></a>Implementación de la solución y comprobación del resultado

En la siguiente imagen se ilustra la supervisión del rendimiento una vez implementada la solución. La carga era similar a la que anterior, pero los tiempos de respuesta para el controlador `UserProfile` ahora son mucho más rápidos. El volumen de solicitudes aumentó de 2759 a 23 565 en el mismo periodo.

![Panel de las transacciones comerciales de AppDynamics que muestra los efectos en los tiempos de respuesta de todas las solicitudes cuando se utiliza el controlador WorkInBackground][AppDynamics-Transactions-Background-Requests]

Tenga en cuenta que el controlador `WorkInBackground` también controlaba un volumen de solicitudes mucho mayor. Sin embargo, no se puede realizar una comparación directa en este caso, puesto que el trabajo que se realizar en este controlador es muy diferente del código original. La nueva versión simplemente pone en cola una solicitud, en lugar de realizar un cálculo lento. La cuestión principal es que este método ya no arrastra todo el sistema con la carga.

El uso de CPU y de red también muestran mejor rendimiento. El uso de CPU nunca alcanza el 100 %, y el volumen de solicitudes de red administrado es mucho mayor que el anterior y no disminuye hasta que lo haga la carga de trabajo.

![Métricas de AppDynamics que muestran el uso de CPU y de red del controlador WorkInBackground][AppDynamics-Metrics-Background-Requests]

En el siguiente gráfico se muestran los resultados de una prueba de carga. El volumen total de solicitudes atendidas ha mejorado notablemente en comparación con las pruebas anteriores.

![Resultados de las pruebas de carga del controlador BackgroundImageProcessing][Load-Test-Results-Background]

## <a name="related-guidance"></a>Instrucciones relacionadas

- [Procedimientos recomendados de escalado automático][autoscaling]
- [Procedimientos recomendados para los trabajos en segundo plano][background-jobs]
- [Queue-Based Load Leveling pattern][load-leveling] (Patrón de equilibrio de carga basado en colas)
- [Web Queue Worker architecture style][web-queue-worker] (Estilo de arquitectura web-cola-trabajo)

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
