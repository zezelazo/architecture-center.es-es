---
title: Antipatrón Synchronous I/O
titleSuffix: Performance antipatterns for cloud apps
description: Bloquear el subproceso que realiza la llamada mientras se completan las operaciones de E/S puede reducir el rendimiento y afectar a la escalabilidad vertical.
author: dragon119
ms.date: 06/05/2017
ms.custom: seodec18
ms.openlocfilehash: 209295cfc911ae168bca2f1c64dc930a27a9a4ba
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/04/2019
ms.locfileid: "54009345"
---
# <a name="synchronous-io-antipattern"></a>Antipatrón Synchronous I/O

Bloquear el subproceso que realiza la llamada mientras se completan las operaciones de E/S puede reducir el rendimiento y afectar a la escalabilidad vertical.

## <a name="problem-description"></a>Descripción del problema

Una operación de E/S sincrónica bloquea el subproceso que realiza la llamada mientras se completa la E/S. El subproceso que realiza la llamada entra en un estado de espera y no puede realizar trabajo útil durante este intervalo, malgastando recursos de procesamiento.

Algunos ejemplos comunes de E/S son:

- Recuperar o guardar datos en una base de datos o en cualquier tipo de almacenamiento persistente.
- Enviar una solicitud a un servicio web.
- Publicar un mensaje o recuperarlo de una cola.
- Escribir o leer en un archivo local.

Este antipatrón suele ocurrir porque:

- Parece ser la manera más intuitiva de realizar una operación.
- La aplicación requiere una respuesta de una solicitud.
- La aplicación utiliza una biblioteca que solo proporciona métodos sincrónicos de E/S.
- Una biblioteca externa lleva a cabo internamente operaciones de E/S sincrónicas. Una sola llamada sincrónica de E/S puede bloquear una cadena de llamadas completa.

El siguiente código carga un archivo en Azure Blob Storage. Hay dos lugares donde el código se bloquea en espera de la E/S sincrónica: los métodos `CreateIfNotExists` y `UploadFromStream`.

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

Este es un ejemplo de espera de una respuesta de un servicio externo. El método `GetUserProfile` llama a un servicio remoto que devuelve un `UserProfile`.

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

Puede encontrar el código completo para ambos ejemplos [aquí][sample-app].

## <a name="how-to-fix-the-problem"></a>Procedimiento para corregir el problema

Reemplace las operaciones de E/S sincrónicas con operaciones asincrónicas. Así se libera el subproceso actual para continuar realizando el trabajo significativo en lugar de bloquearse y se ayuda a mejorar la utilización de recursos de proceso. Realizar las operaciones de E/S de forma asincrónica es especialmente eficaz para controlar una sobrecarga inesperada en las solicitudes de las aplicaciones cliente.

Muchas bibliotecas proporcionan versiones sincrónicas y asincrónicas de los métodos. Siempre que sea posible, utilice las asincrónicas. Esta es la versión asincrónica del ejemplo anterior que carga un archivo en Azure Blob Storage.

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

El operador `await` devuelve el control al entorno de llamada mientras se realiza la operación asincrónica. El código después de esta instrucción actúa como una continuación que se ejecuta cuando se completa la operación asincrónica.

Un servicio bien diseñado también debe proporcionar las operaciones asincrónicas. Esta es una versión asincrónica del servicio web que ejecuta los perfiles de usuario. El método `GetUserProfileAsync` depende de si se dispone de una versión asincrónica del servicio User Profile.

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

Para las bibliotecas que no proporcionan versiones asincrónicas de las operaciones, es posible crear contenedores asincrónicos en torno a métodos sincrónicos seleccionados. Siga este enfoque con precaución. Aunque puede mejorar la capacidad de respuesta en el subproceso que invoca el contenedor asincrónico, realmente consume más recursos. Puede crear un subproceso adicional y no se asocia ninguna sobrecarga a la sincronización del trabajo realizado por él. Algunos de los inconvenientes se analizan en esta entrada del blog: [¿Debería exponer los contenedores asincrónicos para los métodos sincrónicos?][async-wrappers]

Este es un ejemplo de un contenedor asincrónico alrededor de un método sincrónico.

```csharp
// Asynchronous wrapper around synchronous library method
private async Task<int> LibraryIOOperationAsync()
{
    return await Task.Run(() => LibraryIOOperation());
}
```

Ahora el código de llamada puede esperar en el contenedor:

```csharp
// Invoke the asynchronous wrapper using a task
await LibraryIOOperationAsync();
```

## <a name="considerations"></a>Consideraciones

- Las operaciones de E/S que se espera que sean de muy corta duración y es poco probable que ocasionen la contención podrían tener un rendimiento mejor que las operaciones sincrónicas. Un ejemplo podría ser leer archivos pequeños en una unidad SSD. La sobrecarga que supone enviar una tarea a otro subproceso y sincronizar con ese subproceso cuando se completa la tarea, puede descompensar las ventajas de la E/S asincrónica. Sin embargo, estos casos son relativamente poco frecuentes y la mayoría de las operaciones de E/S deben realizarse de forma asincrónica.

- Mejorar el rendimiento de E/S puede hacer que otras partes del sistema se conviertan en cuellos de botella. Por ejemplo, los subprocesos de desbloqueo podrían producir un volumen mayor de solicitudes simultáneas a los recursos compartidos, lo que, a su vez, conduciría al agotamiento de los recursos o a su limitación. Si esto se convierte en un problema, puede que tenga que escalar horizontalmente el número de servidores web o crear particiones de los almacenes de datos para reducir la contención.

## <a name="how-to-detect-the-problem"></a>Procedimiento para detectar el problema

Para los usuarios, puede parecer que la aplicación no responde o que deja de responder periódicamente. La aplicación podría dar error con excepciones de tiempo de espera. Estos errores también podrían devolver errores HTTP 500 (servidor interno). En el servidor, las solicitudes de cliente entrantes podrían bloquearse hasta que un subproceso esté disponible, lo que produce longitudes de cola de solicitudes excesivas, que se manifiestan como errores HTTP 503 (servicio no disponible).

Puede realizar los pasos siguientes para ayudar a identificar este problema:

1. Supervisar el sistema de producción y determinar si los subprocesos de trabajo bloqueados están restringiendo el rendimiento.

2. Si se están bloqueando las solicitudes debido a la falta de subprocesos, revise la aplicación para determinar qué operaciones pueden realizar la E/S sincrónicamente.

3. Realizar pruebas de carga controlada de cada operación que está realizando la E/S sincrónica, para averiguar si estas operaciones afectan desfavorablemente al rendimiento del sistema.

## <a name="example-diagnosis"></a>Diagnóstico de ejemplo

En las secciones siguientes se aplican estos pasos para la aplicación de ejemplo descrita anteriormente.

### <a name="monitor-web-server-performance"></a>Supervisión del rendimiento del servidor web

Para los roles web y las aplicaciones web de Azure, merece la pena supervisar el rendimiento del servidor web IIS. En concreto, preste atención a la longitud de la cola de solicitudes para establecer si las solicitudes se bloquean a la espera de los subprocesos disponibles durante los períodos de gran actividad. Puede recopilar esta información habilitando los diagnósticos de Azure. Para más información, consulte:

- [Supervisión de aplicaciones en Azure App Service][web-sites-monitor]
- [Creación y uso de contadores de rendimiento en una aplicación de Azure][performance-counters]

Instrumente la aplicación para ver cómo se administran las solicitudes una vez que se han aceptado. El seguimiento del flujo de una solicitud puede ayudar a identificar si está realizando llamadas de ejecución lenta y bloqueando el subproceso actual. La generación de perfiles de los subprocesos también puede resaltar las solicitudes que se bloquean.

### <a name="load-test-the-application"></a>Prueba de carga de la aplicación

El siguiente gráfico muestra el rendimiento del método sincrónico `GetUserProfile` mostrado anteriormente, con diferentes cargas de hasta 4 000 usuarios simultáneos. La aplicación es de ASP.NET y se ejecuta en un rol web de Azure Cloud Service.

![Gráfico del rendimiento de la aplicación de ejemplo que realiza operaciones de E/S sincrónicas][sync-performance]

La operación sincrónica está codificada para que se suspenda durante 2 segundos, a fin de simular la E/S sincrónica, por lo que el tiempo de respuesta mínimo es ligeramente superior a 2 segundos. Cuando la carga alcanza aproximadamente 2 500 usuarios simultáneos, el tiempo de respuesta promedio alcanza un nivel estable, aunque el volumen de solicitudes por segundo continúa aumentando. Tenga en cuenta que la escala de estas dos medidas es logarítmica. El número de solicitudes por segundo se duplica entre este punto y el final de la prueba.

Por sí sola, a partir de esta prueba no queda demostrado necesariamente si la E/S sincrónica es un problema. Bajo una carga más compleja, la aplicación puede llegar a un punto clave en el que el servidor web ya no pueda procesar las solicitudes de manera oportuna, haciendo que las aplicaciones de cliente reciban excepciones de tiempo de espera.

El servidor web IIS pone en la cola las solicitudes entrantes y se pasan a un subproceso que se ejecuta en el grupo de subprocesos de ASP.NET. Dado que cada operación realiza la E/S de forma sincrónica, el subproceso se bloquea hasta que se completa la operación. A medida que aumenta la carga de trabajo, finalmente todos los subprocesos de ASP.NET en el grupo de subprocesos se asignan y bloquean. En ese momento, todas las solicitudes entrantes adicionales deben esperar en la cola a un subproceso disponible. A medida que aumenta la longitud de la cola, las solicitudes empiezan a agotar el tiempo de espera.

### <a name="implement-the-solution-and-verify-the-result"></a>Implementación de la solución y comprobación del resultado

El gráfico siguiente muestra los resultados de la prueba de carga de la versión asincrónica del código.

![Gráfico del rendimiento de la aplicación de ejemplo que realiza operaciones de E/S sincrónicas][async-performance]

El rendimiento es mucho más alto. A lo largo del mismo período que la prueba anterior, el sistema controla correctamente un rendimiento casi diez veces mayor, medido en solicitudes por segundo. Además, el tiempo promedio de respuesta es relativamente constante y permanece unas 25 veces por debajo que la prueba anterior.

[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/SynchronousIO
[async-wrappers]: https://blogs.msdn.microsoft.com/pfxteam/2012/03/24/should-i-expose-asynchronous-wrappers-for-synchronous-methods/
[performance-counters]: /azure/cloud-services/cloud-services-dotnet-diagnostics-performance-counters
[web-sites-monitor]: /azure/app-service-web/web-sites-monitor

[sync-performance]: _images/SyncPerformance.jpg
[async-performance]: _images/AsyncPerformance.jpg
