---
title: Antipatrón Synchronous I/O
description: Bloquear el subproceso que realiza la llamada mientras se completan las operaciones de E/S puede reducir el rendimiento y afectar a la escalabilidad vertical.
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: d5b3635565c6b71ef7716f54ee8cccc76093c3a3
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2017
ms.locfileid: "24538559"
---
# <a name="synchronous-io-antipattern"></a><span data-ttu-id="b6c3f-103">Antipatrón Synchronous I/O</span><span class="sxs-lookup"><span data-stu-id="b6c3f-103">Synchronous I/O antipattern</span></span>

<span data-ttu-id="b6c3f-104">Bloquear el subproceso que realiza la llamada mientras se completan las operaciones de E/S puede reducir el rendimiento y afectar a la escalabilidad vertical.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-104">Blocking the calling thread while I/O completes can reduce performance and affect vertical scalability.</span></span>

## <a name="problem-description"></a><span data-ttu-id="b6c3f-105">Descripción del problema</span><span class="sxs-lookup"><span data-stu-id="b6c3f-105">Problem description</span></span>

<span data-ttu-id="b6c3f-106">Una operación de E/S sincrónica bloquea el subproceso que realiza la llamada mientras se completa la E/S.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-106">A synchronous I/O operation blocks the calling thread while the I/O completes.</span></span> <span data-ttu-id="b6c3f-107">El subproceso que realiza la llamada entra en un estado de espera y no puede realizar trabajo útil durante este intervalo, malgastando recursos de procesamiento.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-107">The calling thread enters a wait state and is unable to perform useful work during this interval, wasting processing resources.</span></span>

<span data-ttu-id="b6c3f-108">Algunos ejemplos comunes de E/S son:</span><span class="sxs-lookup"><span data-stu-id="b6c3f-108">Common examples of I/O include:</span></span>

- <span data-ttu-id="b6c3f-109">Recuperar o guardar datos en una base de datos o en cualquier tipo de almacenamiento persistente.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-109">Retrieving or persisting data to a database or any type of persistent storage.</span></span>
- <span data-ttu-id="b6c3f-110">Enviar una solicitud a un servicio web.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-110">Sending a request to a web service.</span></span>
- <span data-ttu-id="b6c3f-111">Publicar un mensaje o recuperarlo de una cola.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-111">Posting a message or retrieving a message from a queue.</span></span>
- <span data-ttu-id="b6c3f-112">Escribir o leer en un archivo local.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-112">Writing to or reading from a local file.</span></span>

<span data-ttu-id="b6c3f-113">Este antipatrón suele ocurrir porque:</span><span class="sxs-lookup"><span data-stu-id="b6c3f-113">This antipattern typically occurs because:</span></span>

- <span data-ttu-id="b6c3f-114">Parece ser la manera más intuitiva de realizar una operación.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-114">It appears to be the most intuitive way to perform an operation.</span></span> 
- <span data-ttu-id="b6c3f-115">La aplicación requiere una respuesta de una solicitud.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-115">The application requires a response from a request.</span></span>
- <span data-ttu-id="b6c3f-116">La aplicación utiliza una biblioteca que solo proporciona métodos sincrónicos de E/S.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-116">The application uses a library that only provides synchronous methods for I/O.</span></span> 
- <span data-ttu-id="b6c3f-117">Una biblioteca externa lleva a cabo internamente operaciones de E/S sincrónicas.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-117">An external library performs synchronous I/O operations internally.</span></span> <span data-ttu-id="b6c3f-118">Una sola llamada sincrónica de E/S puede bloquear una cadena de llamadas completa.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-118">A single synchronous I/O call can block an entire call chain.</span></span>

<span data-ttu-id="b6c3f-119">El siguiente código carga un archivo en Azure Blob Storage.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-119">The following code uploads a file to Azure blob storage.</span></span> <span data-ttu-id="b6c3f-120">Hay dos lugares donde el código se bloquea en espera de la E/S sincrónica: los métodos `CreateIfNotExists` y `UploadFromStream`.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-120">There are two places where the code blocks waiting for synchronous I/O, the `CreateIfNotExists` method and the `UploadFromStream` method.</span></span>

```csharp
var blobClient = storageAccount.CreateCloudBlobClient();
var container = blobClient.GetContainerReference("uploadedfiles");

container.CreateIfNotExists();
var blockBlob = container.GetBlockBlobReference("myblob");

// Create or overwrite the "myblob" blob with contents from a local file.
using (var fileStream = File.OpenRead(HostingEnvironment.MapPath("~/FileToUpload.txt")))
{
    blockBlob.UploadFromStream(fileStream);
}
```

<span data-ttu-id="b6c3f-121">Este es un ejemplo de espera de una respuesta de un servicio externo.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-121">Here's an example of waiting for a response from an external service.</span></span> <span data-ttu-id="b6c3f-122">El método `GetUserProfile` llama a un servicio remoto que devuelve un `UserProfile`.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-122">The `GetUserProfile` method calls a remote service that returns a `UserProfile`.</span></span>

```csharp
public interface IUserProfileService
{
    UserProfile GetUserProfile();
}

public class SyncController : ApiController
{
    private readonly IUserProfileService _userProfileService;

    public SyncController()
    {
        _userProfileService = new FakeUserProfileService();
    }

    // This is a synchronous method that calls the synchronous GetUserProfile method.
    public UserProfile GetUserProfile()
    {
        return _userProfileService.GetUserProfile();
    }
}
```

<span data-ttu-id="b6c3f-123">Puede encontrar el código completo para ambos ejemplos [aquí][sample-app].</span><span class="sxs-lookup"><span data-stu-id="b6c3f-123">You can find the complete code for both of these examples [here][sample-app].</span></span>

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="b6c3f-124">Procedimiento para corregir el problema</span><span class="sxs-lookup"><span data-stu-id="b6c3f-124">How to fix the problem</span></span>

<span data-ttu-id="b6c3f-125">Reemplace las operaciones de E/S sincrónicas con operaciones asincrónicas.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-125">Replace synchronous I/O operations with asynchronous operations.</span></span> <span data-ttu-id="b6c3f-126">Así se libera el subproceso actual para continuar realizando el trabajo significativo en lugar de bloquearse y se ayuda a mejorar la utilización de recursos de proceso.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-126">This frees the current thread to continue performing meaningful work rather than blocking, and helps improve the utilization of compute resources.</span></span> <span data-ttu-id="b6c3f-127">Realizar las operaciones de E/S de forma asincrónica es especialmente eficaz para controlar una sobrecarga inesperada en las solicitudes de las aplicaciones cliente.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-127">Performing I/O asynchronously is particularly efficient for handling an unexpected surge in requests from client applications.</span></span> 

<span data-ttu-id="b6c3f-128">Muchas bibliotecas proporcionan versiones sincrónicas y asincrónicas de los métodos.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-128">Many libraries provide both synchronous and asynchronous versions of methods.</span></span> <span data-ttu-id="b6c3f-129">Siempre que sea posible, utilice las asincrónicas.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-129">Whenever possible, use the asynchronous versions.</span></span> <span data-ttu-id="b6c3f-130">Esta es la versión asincrónica del ejemplo anterior que carga un archivo en Azure Blob Storage.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-130">Here is the asynchronous version of the previous example that uploads a file to Azure blob storage.</span></span>

```csharp
var blobClient = storageAccount.CreateCloudBlobClient();
var container = blobClient.GetContainerReference("uploadedfiles");

await container.CreateIfNotExistsAsync();

var blockBlob = container.GetBlockBlobReference("myblob");

// Create or overwrite the "myblob" blob with contents from a local file.
using (var fileStream = File.OpenRead(HostingEnvironment.MapPath("~/FileToUpload.txt")))
{
    await blockBlob.UploadFromStreamAsync(fileStream);
}
```

<span data-ttu-id="b6c3f-131">El operador `await` devuelve el control al entorno de llamada mientras se realiza la operación asincrónica.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-131">The `await` operator returns control to the calling environment while the asynchronous operation is performed.</span></span> <span data-ttu-id="b6c3f-132">El código después de esta instrucción actúa como una continuación que se ejecuta cuando se completa la operación asincrónica.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-132">The code after this statement acts as a continuation that runs when the asynchronous operation has completed.</span></span>

<span data-ttu-id="b6c3f-133">Un servicio bien diseñado también debe proporcionar las operaciones asincrónicas.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-133">A well designed service should also provide asynchronous operations.</span></span> <span data-ttu-id="b6c3f-134">Esta es una versión asincrónica del servicio web que ejecuta los perfiles de usuario.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-134">Here is an asynchronous version of the web service that returns user profiles.</span></span> <span data-ttu-id="b6c3f-135">El método `GetUserProfileAsync` depende de si se dispone de una versión asincrónica del servicio User Profile.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-135">The `GetUserProfileAsync` method depends on having an asynchronous version of the User Profile service.</span></span>

```csharp
public interface IUserProfileService
{
    Task<UserProfile> GetUserProfileAsync();
}

public class AsyncController : ApiController
{
    private readonly IUserProfileService _userProfileService;

    public AsyncController()
    {
        _userProfileService = new FakeUserProfileService();
    }

    // This is an synchronous method that calls the Task based GetUserProfileAsync method.
    public Task<UserProfile> GetUserProfileAsync()
    {
        return _userProfileService.GetUserProfileAsync();
    }
}
```

<span data-ttu-id="b6c3f-136">Para las bibliotecas que no proporcionan versiones asincrónicas de las operaciones, es posible crear contenedores asincrónicos en torno a métodos sincrónicos seleccionados.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-136">For libraries that don't provide asynchronous versions of operations, it may be possible to create asynchronous wrappers around selected synchronous methods.</span></span> <span data-ttu-id="b6c3f-137">Siga este enfoque con precaución.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-137">Follow this approach with caution.</span></span> <span data-ttu-id="b6c3f-138">Aunque puede mejorar la capacidad de respuesta en el subproceso que invoca el contenedor asincrónico, realmente consume más recursos.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-138">While it may improve responsiveness on the thread that invokes the asynchronous wrapper, it actually consumes more resources.</span></span> <span data-ttu-id="b6c3f-139">Puede crear un subproceso adicional y no se asocia ninguna sobrecarga a la sincronización del trabajo realizado por él.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-139">An extra thread may be created, and there is overhead associated with synchronizing the work done by this thread.</span></span> <span data-ttu-id="b6c3f-140">Algunos inconvenientes se analizan en esta entrada del blog: [¿Debería exponer los contenedores asincrónicos para los métodos sincrónicos?][async-wrappers]</span><span class="sxs-lookup"><span data-stu-id="b6c3f-140">Some tradeoffs are discussed in this blog post: [Should I expose asynchronous wrappers for synchronous methods?][async-wrappers]</span></span>

<span data-ttu-id="b6c3f-141">Este es un ejemplo de un contenedor asincrónico alrededor de un método sincrónico.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-141">Here is an example of an asynchronous wrapper around a synchronous method.</span></span>

```csharp
// Asynchronous wrapper around synchronous library method
private async Task<int> LibraryIOOperationAsync()
{
    return await Task.Run(() => LibraryIOOperation());
}
```

<span data-ttu-id="b6c3f-142">Ahora el código de llamada puede esperar en el contenedor:</span><span class="sxs-lookup"><span data-stu-id="b6c3f-142">Now the calling code can await on the wrapper:</span></span>

```csharp
// Invoke the asynchronous wrapper using a task
await LibraryIOOperationAsync();
```

## <a name="considerations"></a><span data-ttu-id="b6c3f-143">Consideraciones</span><span class="sxs-lookup"><span data-stu-id="b6c3f-143">Considerations</span></span>

- <span data-ttu-id="b6c3f-144">Las operaciones de E/S que se espera que sean de muy corta duración y es poco probable que ocasionen la contención podrían tener un rendimiento mejor que las operaciones sincrónicas.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-144">I/O operations that are expected to be very short lived and are unlikely to cause contention might be more performant as synchronous operations.</span></span> <span data-ttu-id="b6c3f-145">Un ejemplo podría ser leer archivos pequeños en una unidad SSD.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-145">An example might be reading small files on an SSD drive.</span></span> <span data-ttu-id="b6c3f-146">La sobrecarga que supone enviar una tarea a otro subproceso y sincronizar con ese subproceso cuando se completa la tarea, puede descompensar las ventajas de la E/S asincrónica.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-146">The overhead of dispatching a task to another thread, and synchronizing with that thread when the task completes, might outweigh the benefits of asynchronous I/O.</span></span> <span data-ttu-id="b6c3f-147">Sin embargo, estos casos son relativamente poco frecuentes y la mayoría de las operaciones de E/S deben realizarse de forma asincrónica.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-147">However, these cases are relatively rare, and most I/O operations should be done asynchronously.</span></span>

- <span data-ttu-id="b6c3f-148">Mejorar el rendimiento de E/S puede hacer que otras partes del sistema se conviertan en cuellos de botella.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-148">Improving I/O performance may cause other parts of the system to become bottlenecks.</span></span> <span data-ttu-id="b6c3f-149">Por ejemplo, los subprocesos de desbloqueo podrían producir un volumen mayor de solicitudes simultáneas a los recursos compartidos, lo que, a su vez, conduciría al agotamiento de los recursos o a su limitación.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-149">For example, unblocking threads might result in a higher volume of concurrent requests to shared resources, leading in turn to resource starvation or throttling.</span></span> <span data-ttu-id="b6c3f-150">Si esto se convierte en un problema, puede que tenga que escalar horizontalmente el número de servidores web o crear particiones de los almacenes de datos para reducir la contención.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-150">If that becomes a problem, you might need to scale out the number of web servers or partition data stores to reduce contention.</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="b6c3f-151">Procedimiento para detectar el problema</span><span class="sxs-lookup"><span data-stu-id="b6c3f-151">How to detect the problem</span></span>

<span data-ttu-id="b6c3f-152">Para los usuarios, puede parecer que la aplicación no responde o que deja de responder periódicamente.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-152">For users, the application may seem unresponsive or appear to hang periodically.</span></span> <span data-ttu-id="b6c3f-153">La aplicación podría dar error con excepciones de tiempo de espera.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-153">The application might fail with timeout exceptions.</span></span> <span data-ttu-id="b6c3f-154">Estos errores también podrían devolver errores HTTP 500 (servidor interno).</span><span class="sxs-lookup"><span data-stu-id="b6c3f-154">These failures could also return HTTP 500 (Internal Server) errors.</span></span> <span data-ttu-id="b6c3f-155">En el servidor, las solicitudes de cliente entrantes podrían bloquearse hasta que un subproceso esté disponible, lo que produce longitudes de cola de solicitudes excesivas, que se manifiestan como errores HTTP 503 (servicio no disponible).</span><span class="sxs-lookup"><span data-stu-id="b6c3f-155">On the server, incoming client requests might be blocked until a thread becomes available, resulting in excessive request queue lengths, manifested as HTTP 503 (Service Unavailable) errors.</span></span>

<span data-ttu-id="b6c3f-156">Puede realizar los pasos siguientes para ayudar a identificar este problema:</span><span class="sxs-lookup"><span data-stu-id="b6c3f-156">You can perform the following steps to help identify the problem:</span></span>

1. <span data-ttu-id="b6c3f-157">Supervisar el sistema de producción y determinar si los subprocesos de trabajo bloqueados están restringiendo el rendimiento.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-157">Monitor the production system and determine whether blocked worker threads are constraining throughput.</span></span>

2. <span data-ttu-id="b6c3f-158">Si se están bloqueando las solicitudes debido a la falta de subprocesos, revise la aplicación para determinar qué operaciones pueden realizar la E/S sincrónicamente.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-158">If requests are being blocked due to lack of threads, review the application to determine which operations may be performing I/O synchronously.</span></span>

3. <span data-ttu-id="b6c3f-159">Realizar pruebas de carga controlada de cada operación que está realizando la E/S sincrónica, para averiguar si estas operaciones afectan desfavorablemente al rendimiento del sistema.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-159">Perform controlled load testing of each operation that is performing synchronous I/O, to find out whether those operations are affecting system performance.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="b6c3f-160">Diagnóstico de ejemplo</span><span class="sxs-lookup"><span data-stu-id="b6c3f-160">Example diagnosis</span></span>

<span data-ttu-id="b6c3f-161">En las secciones siguientes se aplican estos pasos para la aplicación de ejemplo descrita anteriormente.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-161">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="monitor-web-server-performance"></a><span data-ttu-id="b6c3f-162">Supervisión del rendimiento del servidor web</span><span class="sxs-lookup"><span data-stu-id="b6c3f-162">Monitor web server performance</span></span>

<span data-ttu-id="b6c3f-163">Para los roles web y las aplicaciones web de Azure, merece la pena supervisar el rendimiento del servidor web IIS.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-163">For Azure web applications and web roles, it's worth monitoring the performance of the IIS web server.</span></span> <span data-ttu-id="b6c3f-164">En concreto, preste atención a la longitud de la cola de solicitudes para establecer si las solicitudes se bloquean a la espera de los subprocesos disponibles durante los períodos de gran actividad.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-164">In particular, pay attention to the request queue length to establish whether requests are being blocked waiting for available threads during periods of high activity.</span></span> <span data-ttu-id="b6c3f-165">Puede recopilar esta información habilitando los diagnósticos de Azure.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-165">You can gather this information by enabling Azure diagnostics.</span></span> <span data-ttu-id="b6c3f-166">Para más información, consulte:</span><span class="sxs-lookup"><span data-stu-id="b6c3f-166">For more information, see:</span></span>

- <span data-ttu-id="b6c3f-167">[Supervisión de aplicaciones en Azure App Service][web-sites-monitor]</span><span class="sxs-lookup"><span data-stu-id="b6c3f-167">[Monitor Apps in Azure App Service][web-sites-monitor]</span></span>
- <span data-ttu-id="b6c3f-168">[Creación y uso de contadores de rendimiento en una aplicación de Azure][performance-counters]</span><span class="sxs-lookup"><span data-stu-id="b6c3f-168">[Create and use performance counters in an Azure application][performance-counters]</span></span>

<span data-ttu-id="b6c3f-169">Instrumente la aplicación para ver cómo se administran las solicitudes una vez que se han aceptado.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-169">Instrument the application to see how requests are handled once they have been accepted.</span></span> <span data-ttu-id="b6c3f-170">El seguimiento del flujo de una solicitud puede ayudar a identificar si está realizando llamadas de ejecución lenta y bloqueando el subproceso actual.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-170">Tracing the flow of a request can help to identify whether it is performing slow-running calls and blocking the current thread.</span></span> <span data-ttu-id="b6c3f-171">La generación de perfiles de los subprocesos también puede resaltar las solicitudes que se bloquean.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-171">Thread profiling can also highlight requests that are being blocked.</span></span>

### <a name="load-test-the-application"></a><span data-ttu-id="b6c3f-172">Prueba de carga de la aplicación</span><span class="sxs-lookup"><span data-stu-id="b6c3f-172">Load test the application</span></span>

<span data-ttu-id="b6c3f-173">El siguiente gráfico muestra el rendimiento del método sincrónico `GetUserProfile` mostrado anteriormente, con diferentes cargas de hasta 4 000 usuarios simultáneos.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-173">The following graph shows the performance of the synchronous `GetUserProfile` method shown earlier, under varying loads of up to 4000 concurrent users.</span></span> <span data-ttu-id="b6c3f-174">La aplicación es de ASP.NET y se ejecuta en un rol web de Azure Cloud Service.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-174">The application is an ASP.NET application running in an Azure Cloud Service web role.</span></span>

![Gráfico del rendimiento de la aplicación de ejemplo que realiza operaciones de E/S sincrónicas][sync-performance]

<span data-ttu-id="b6c3f-176">La operación sincrónica está codificada para que se suspenda durante 2 segundos, a fin de simular la E/S sincrónica, por lo que el tiempo de respuesta mínimo es ligeramente superior a 2 segundos.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-176">The synchronous operation is hard-coded to sleep for 2 seconds, to simulate synchronous I/O, so the minimum response time is slightly over 2 seconds.</span></span> <span data-ttu-id="b6c3f-177">Cuando la carga alcanza aproximadamente 2 500 usuarios simultáneos, el tiempo de respuesta promedio alcanza un nivel estable, aunque el volumen de solicitudes por segundo continúa aumentando.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-177">When the load reaches approximately 2500 concurrent users, the average response time reaches a plateau, although the volume of requests per second continues to increase.</span></span> <span data-ttu-id="b6c3f-178">Tenga en cuenta que la escala de estas dos medidas es logarítmica.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-178">Note that the scale for these two measures is logarithmic.</span></span> <span data-ttu-id="b6c3f-179">El número de solicitudes por segundo se duplica entre este punto y el final de la prueba.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-179">The number of requests per second doubles between this point and the end of the test.</span></span>

<span data-ttu-id="b6c3f-180">Por sí sola, a partir de esta prueba no queda demostrado necesariamente si la E/S sincrónica es un problema.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-180">In isolation, it's not necessarily clear from this test whether the synchronous I/O is a problem.</span></span> <span data-ttu-id="b6c3f-181">Bajo una carga más compleja, la aplicación puede llegar a un punto clave en el que el servidor web ya no pueda procesar las solicitudes de manera oportuna, haciendo que las aplicaciones de cliente reciban excepciones de tiempo de espera.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-181">Under heavier load, the application may reach a tipping point where the web server can no longer process requests in a timely manner, causing client applications to receive time-out exceptions.</span></span>

<span data-ttu-id="b6c3f-182">El servidor web IIS pone en la cola las solicitudes entrantes y se pasan a un subproceso que se ejecuta en el grupo de subprocesos de ASP.NET.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-182">Incoming requests are queued by the IIS web server and handed to a thread running in the ASP.NET thread pool.</span></span> <span data-ttu-id="b6c3f-183">Dado que cada operación realiza la E/S de forma sincrónica, el subproceso se bloquea hasta que se completa la operación.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-183">Because each operation performs I/O synchronously, the thread is blocked until the operation completes.</span></span> <span data-ttu-id="b6c3f-184">A medida que aumenta la carga de trabajo, finalmente todos los subprocesos de ASP.NET en el grupo de subprocesos se asignan y bloquean.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-184">As the workload increases, eventually all of the ASP.NET threads in the thread pool are allocated and blocked.</span></span> <span data-ttu-id="b6c3f-185">En ese momento, todas las solicitudes entrantes adicionales deben esperar en la cola a un subproceso disponible.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-185">At that point, any further incoming requests must wait in the queue for an available thread.</span></span> <span data-ttu-id="b6c3f-186">A medida que aumenta la longitud de la cola, las solicitudes empiezan a agotar el tiempo de espera.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-186">As the queue length grows, requests start to time out.</span></span>

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="b6c3f-187">Implementación de la solución y comprobación del resultado</span><span class="sxs-lookup"><span data-stu-id="b6c3f-187">Implement the solution and verify the result</span></span>

<span data-ttu-id="b6c3f-188">El gráfico siguiente muestra los resultados de la prueba de carga de la versión asincrónica del código.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-188">The next graph shows the results from load testing the asynchronous version of the code.</span></span>

![Gráfico del rendimiento de la aplicación de ejemplo que realiza operaciones de E/S sincrónicas][async-performance]

<span data-ttu-id="b6c3f-190">El rendimiento es mucho más alto.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-190">Throughput is far higher.</span></span> <span data-ttu-id="b6c3f-191">A lo largo del mismo período que la prueba anterior, el sistema controla correctamente un rendimiento casi diez veces mayor, medido en solicitudes por segundo.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-191">Over the same duration as the previous test, the system successfully handles a nearly tenfold increase in throughput, as measured in requests per second.</span></span> <span data-ttu-id="b6c3f-192">Además, el tiempo promedio de respuesta es relativamente constante y permanece unas 25 veces por debajo que la prueba anterior.</span><span class="sxs-lookup"><span data-stu-id="b6c3f-192">Moreover, the average response time is relatively constant and remains approximately 25 times smaller than the previous test.</span></span>


[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/SynchronousIO


[async-wrappers]: http://blogs.msdn.com/b/pfxteam/archive/2012/03/24/10287244.aspx
[performance-counters]: /azure/cloud-services/cloud-services-dotnet-diagnostics-performance-counters
[web-sites-monitor]: /azure/app-service-web/web-sites-monitor

[sync-performance]: _images/SyncPerformance.jpg
[async-performance]: _images/AsyncPerformance.jpg



