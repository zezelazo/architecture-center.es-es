---
title: Leader Election
description: "Coordina las acciones realizadas por una colección de instancias de tareas de colaboración de una aplicación distribuida mediante la elección de una instancia como líder que asume la responsabilidad de administrar las demás instancias."
keywords: "Patrón de diseño"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- design-implementation
- resiliency
ms.openlocfilehash: ddb61097ed3229ed0ed517b94c280d3ef892c999
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2017
---
# <a name="leader-election-pattern"></a>Patrón Leader Election

[!INCLUDE [header](../_includes/header.md)]

Coordina las acciones realizadas por una colección de instancias de colaboración de una aplicación distribuida mediante la elección de una instancia como líder que asume la responsabilidad de administrar las otras. Esto puede ayudar a garantizar que las instancias no entren en conflicto, provoquen la contención de recursos compartidos o interfieran por accidente con el trabajo que realizan otras instancias.

## <a name="context-and-problem"></a>Contexto y problema

Una aplicación en la nube típica tiene muchas tareas que actúan de manera coordinada. Estas instancias podrían ser todas instancias que ejecutan el mismo código y que requieren acceso a los mismos recursos, o podría estar funcionando juntas en paralelo para realizar las partes individuales de un cálculo complejo.

Las instancias de tarea podrían ejecutarse por separado la mayor parte del tiempo, pero también podría ser necesario coordinar las acciones de cada una para garantizar que no entren en conflicto, provoquen la contención de recursos compartidos o interfieran por accidente con el trabajo que realizan otras instancias de tarea.

Por ejemplo:

- En un sistema basado en la nube que implementa escalado horizontal, varias instancias de la misma tarea podrían estar ejecutándose al mismo tiempo y cada instancia atender a un usuario diferente. Si estas instancias escriben en un recurso compartido, es necesario coordinar sus acciones para evitar que cada instancia sobrescriba los cambios realizados por las demás.
- Si las tareas ejecutan elementos individuales de un cálculo complejo en paralelo, los resultados deben agregarse cuando dichos cálculos finalicen.

Las instancia de tarea son todas del mismo nivel, por lo que no hay un líder natural que actúe como coordinador o agregador.

## <a name="solution"></a>Solución

Se debe elegir una única instancia de tarea para que actúe como líder, y esta instancia debe coordinar las acciones de las demás instancias de tarea subordinadas. Si todas las instancias de tarea ejecutan el mismo código, cada una de ellas tiene la capacidad para actuar como líder. Por lo tanto, el proceso de elección debe administrarse con cuidado para evitar que dos o más instancias asuman el rol de líder al mismo tiempo.

El sistema debe proporcionar un mecanismo eficaz para seleccionar el líder. Este método tiene que hacer frente a eventos, como interrupciones de red o errores de proceso. En muchas de las soluciones, las instancias de tarea subordinadas supervisan el líder a través de algún tipo de método de latido o mediante sondeo. Si el líder designado termina de forma inesperada, o un error de red hace que el líder no esté disponible para las instancias de tarea subordinadas, será necesario elegir un nuevo líder.

Existen varias estrategias para elegir un líder entre un conjunto de tareas en un entorno distribuido, como son:
- Seleccionar la instancia de tarea con el identificador de proceso o de instancia peor clasificados.
- Competir para adquirir una exclusión mutua compartida y distribuida. La primera instancia de tarea que adquiere la exclusión mutua es el líder. Sin embargo, el sistema debe asegurarse de que, si el líder finaliza o se desconecta del resto del sistema, se libere la exclusión mutua para permitir que otra instancia de tarea se convierta en el líder.
- Implementar uno de los algoritmos de Leader Electrion comunes como el [algoritmo Bully](http://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/BullyExample.html) o el [algoritmo Ring](http://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/RingElectExample.html). Estos algoritmos dan por supuesto que cada candidato de la elección tiene un identificador único y que puede comunicarse con los otros candidatos de manera confiable.

## <a name="issues-and-considerations"></a>Problemas y consideraciones

Tenga en cuenta los puntos siguientes al decidir cómo implementar este patrón:
- El proceso de elegir un líder debería ser resistente a errores transitorios y persistentes.
- Tiene que ser posible detectar cuándo el líder ha generado un error o ha dejado de estar disponible (por ejemplo, debido a un error de comunicación). La rapidez con que se necesite la detección depende del sistema. Algunos sistemas podrían funcionar durante un corto espacio de tiempo sin un líder, en cuyo transcurso un error transitorio podría corregirse. En otros casos, podría ser necesario detectar el error del líder inmediatamente y desencadenar una nueva elección.
- En un sistema que implementa el escalado automático horizontal, el líder podría finalizarse si el sistema se reduce y cierra algunos de los recursos de computación.
- El uso de una exclusión mutua compartida distribuida introduce una dependencia en el servicio externo que proporciona la exclusión mutua. El servicio constituye un punto único de error. Si por algún motivo deja de estar disponible, el sistema no podrá elegir un líder.
- El uso de un único proceso dedicado como líder es un enfoque sencillo. Sin embargo, si el proceso produce un error, podría haber un considerable retraso mientras se reinicia. La latencia resultante puede afectar al rendimiento y a los tiempos de respuesta de otros procesos si están a la espera de que el líder coordine una operación.
- La implementación de uno de los algoritmos de Leader Election proporciona manualmente la máxima flexibilidad para optimizar y ajustar el código.

## <a name="when-to-use-this-pattern"></a>Cuándo usar este patrón

Use este patrón cuando las tareas de una aplicación distribuida, como una solución hospedada en las nube, necesiten una cuidadosa coordinación y no haya un líder natural.

>  Evite convertir el líder en un cuello de botella en el sistema. La finalidad del líder es coordinar el trabajo de las tareas subordinadas, y no tiene que participar necesariamente en este trabajo&mdash;aunque debería poder hacerlo si la tarea no se elige como líder.

Este patrón puede ser útil en los siguientes casos:
- Hay un líder natural o un proceso dedicado que puede actuar siempre como líder. Por ejemplo, podría implementarse un proceso de singleton que coordine las instancias de tarea. Si se produce un error en este proceso o su estado no es correcto, el sistema puede cerrarlo y reiniciarlo.
- La coordinación entre las tareas se puede lograr con un método más ligero. Por ejemplo, si varias instancias de tarea solo necesitan acceso coordinado a un recurso compartido, una solución mejor es usar bloqueo optimista o pesimista para controlar el acceso.
- Una solución de terceros es más adecuada. Por ejemplo, el servicio Microsoft Azure HDInsight (basado en Apache Hadoop) usa los servicios que proporciona Apache Zookeeper para coordinar las tareas de asignación y reducción que recopilan y resumen los datos.

## <a name="example"></a>Ejemplo

El proyecto DistributedMutex de la solución LeaderElection (se puede encontrar un ejemplo que demuestra este patrón en [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/leader-election)) muestra cómo usar una concesión en un blob de Azure Storage para proporcionar un mecanismo a fin de implementar una exclusión mutua distribuida compartida. Esta exclusión mutua puede usarse para elegir un líder entre un grupo de instancias de rol de un servicio en la nube de Azure. La primera instancia de rol en adquirir la concesión se elige como líder y lo sigue siendo hasta que libera la concesión o es incapaz de renovarla. Otras instancias de rol pueden continuar con la supervisión de la concesión del blob en caso de que el líder ya no esté disponible.

>  Una concesión de blob es un bloqueo exclusivo de escritura sobre un blob. Un único blob solo puede ser el sujeto de una concesión en cualquier momento en el tiempo. Una instancia de rol puede solicitar una concesión sobre un blob especificado y se le concederá la concesión si ninguna otra instancia de rol mantiene una concesión sobre el mismo blob. En caso contrario, la solicitud inicia una excepción.

> Para evitar que una instancia de rol con errores conserve la concesión de forma indefinida, especifique una duración para la concesión. Cuando expire, la concesión estará disponible. Sin embargo, mientras una instancia de rol mantenga la concesión puede solicitar que ésta se renueve, así que se le otorgará durante un periodo de tiempo adicional. La instancia de rol puede repetir continuamente este proceso si desea conservar la concesión.
Para más información sobre cómo conceder un blob, consulte [Lease Blob (API de REST)](https://msdn.microsoft.com/library/azure/ee691972.aspx).

La clase `BlobDistributedMutex` del ejemplo de C# contiene el método `RunTaskWhenMutexAquired` que permite que una instancia de rol intente adquirir una concesión sobre un blob especificado. Los detalles del blog (el nombre, el contenedor y la cuenta de almacenamiento) se pasan al constructor en un objeto `BlobSettings` cuando se crea el objeto `BlobDistributedMutex` (este objeto es una estructura sencilla que se incluye el código de ejemplo). El constructor también acepta un elemento `Task` que hace referencia al código que debe ejecutar la instancia de rol si adquiere correctamente la concesión sobre el blob y se elige como líder. Tenga en cuenta que el código que administra los detalles de bajo nivel de adquisición de la concesión se implementa en una clase auxiliar independiente llamada `BlobLeaseManager`.

```csharp
public class BlobDistributedMutex
{
  ...
  private readonly BlobSettings blobSettings;
  private readonly Func<CancellationToken, Task> taskToRunWhenLeaseAcquired;
  ...

  public BlobDistributedMutex(BlobSettings blobSettings,
           Func<CancellationToken, Task> taskToRunWhenLeaseAquired)
  {
    this.blobSettings = blobSettings;
    this.taskToRunWhenLeaseAquired = taskToRunWhenLeaseAquired;
  }

  public async Task RunTaskWhenMutexAcquired(CancellationToken token)
  {
    var leaseManager = new BlobLeaseManager(blobSettings);
    await this.RunTaskWhenBlobLeaseAcquired(leaseManager, token);
  }
  ...
```

El método `RunTaskWhenMutexAquired` del código de ejemplo anterior invoca el método `RunTaskWhenBlobLeaseAcquired` mostrado en el código de ejemplo siguiente para adquirir realmente la concesión. El método `RunTaskWhenBlobLeaseAcquired` se ejecuta de forma asincrónica. Si la concesión se adquirió correctamente, la instancia de rol se elige como líder. El propósito del delegado `taskToRunWhenLeaseAcquired` es realizar el trabajo que coordina las otras instancias de rol. Si la concesión no se adquiere, otra instancia de rol se elije como líder y la instancia de rol actual queda como la subordinada. Tenga en cuenta que el método `TryAcquireLeaseOrWait` es un método auxiliar que utiliza el objeto `BlobLeaseManager` para adquirir la concesión.

```csharp
  private async Task RunTaskWhenBlobLeaseAcquired(
    BlobLeaseManager leaseManager, CancellationToken token)
  {
    while (!token.IsCancellationRequested)
    {
      // Try to acquire the blob lease.
      // Otherwise wait for a short time before trying again.
      string leaseId = await this.TryAquireLeaseOrWait(leaseManager, token);

      if (!string.IsNullOrEmpty(leaseId))
      {
        // Create a new linked cancellation token source so that if either the
        // original token is canceled or the lease can't be renewed, the
        // leader task can be canceled.
        using (var leaseCts =
          CancellationTokenSource.CreateLinkedTokenSource(new[] { token }))
        {
          // Run the leader task.
          var leaderTask = this.taskToRunWhenLeaseAquired.Invoke(leaseCts.Token);
          ...
        }
      }
    }
    ...
  }
```

La tarea iniciada por el líder también se ejecuta de manera asincrónica. Mientras se ejecuta esta tarea, el método `RunTaskWhenBlobLeaseAquired` mostrado en el siguiente código de ejemplo intenta periódicamente renovar la concesión. Esto ayuda a garantizar que la instancia de rol sigue siendo el líder. En la solución de ejemplo, el retraso entre solicitudes de renovación es menor que el tiempo especificado para la duración de la concesión a fin de evitar que otra instancia de rol se elija como líder. Si se produce un error en la renovación por cualquier motivo, se cancela la tarea.

Si no se renueva la concesión o se cancela la tarea (probablemente debido al cierre de la instancia de rol), se libera la concesión. En este momento, esta u otra instancia de rol podrían elegirse como líder. El extracto de código siguiente muestra esta parte del proceso.

```csharp
  private async Task RunTaskWhenBlobLeaseAcquired(
    BlobLeaseManager leaseManager, CancellationToken token)
  {
    while (...)
    {
      ...
      if (...)
      {
        ...
        using (var leaseCts = ...)
        {
          ...
          // Keep renewing the lease in regular intervals.
          // If the lease can't be renewed, then the task completes.
          var renewLeaseTask =
            this.KeepRenewingLease(leaseManager, leaseId, leaseCts.Token);

          // When any task completes (either the leader task itself or when it
          // couldn't renew the lease) then cancel the other task.
          await CancelAllWhenAnyCompletes(leaderTask, renewLeaseTask, leaseCts);
        }
      }
    }
  }
  ...
}
```

El método `KeepRenewingLease` es otro método auxiliar que usa el objeto `BlobLeaseManager` para renovar la concesión. El método `CancelAllWhenAnyCompletes` cancela las tareas especificadas como los dos primeros parámetros. En el diagrama siguiente se ilustra el uso de la clase `BlobDistributedMutex` para elegir un líder y ejecutar una tarea que coordina las operaciones.

![La figura 1 muestra las funciones de la clase BlobDistributedMutex.](./_images/leader-election-diagram.png)


En el código de ejemplo siguiente se muestra cómo usar la clase `BlobDistributedMutex` en un rol de trabajo. Este código adquiere una concesión sobre un blob denominado `MyLeaderCoordinatorTask` en el contenedor de la concesión en el almacenamiento de desarrollo y especifica que el código definido en el método `MyLeaderCoordinatorTask` debe ejecutarse si la instancia de rol se elige como líder.

```csharp
var settings = new BlobSettings(CloudStorageAccount.DevelopmentStorageAccount,
  "leases", "MyLeaderCoordinatorTask");
var cts = new CancellationTokenSource();
var mutex = new BlobDistributedMutex(settings, MyLeaderCoordinatorTask);
mutex.RunTaskWhenMutexAcquired(this.cts.Token);
...

// Method that runs if the role instance is elected the leader
private static async Task MyLeaderCoordinatorTask(CancellationToken token)
{
  ...
}
```

Tenga en cuenta los siguientes puntos acerca de la solución de ejemplo:
- El blob es un posible único punto de error. Si el servicio de blob deja de estar disponible, o no es accesible, el líder no podrá renovar la concesión y ninguna otra instancia de rol podrá adquirirla. En este caso, ninguna instancia de rol podrá funcionar como líder. Sin embargo, el servicio de blob está diseñado para ser resistente, por lo que se considera improbable el error completo de dicho servicio.
- Si la tarea que realiza el líder se detiene, el líder puede continuar y renovar la concesión, lo que impide que cualquier otra instancia de rol pueda adquirirla y asuma el rol de líder para coordinar las tareas. En el mundo real, se debe comprobar el mantenimiento del líder a intervalos frecuentes.
- El proceso de elección es no determinista. No se puede realizar ninguna suposición acerca de qué instancia de rol adquirirá la concesión del blob y se convertirá en líder.
- El blob que se usa como el destino de la concesión no debe usarse para otros fines. Si una instancia de rol intenta almacenar datos en este blob, estos datos no podrán estar accesibles a menos que la instancia de rol sea el líder y mantenga la concesión del blob.

## <a name="related-patterns-and-guidance"></a>Orientación y patrones relacionados

Las directrices siguientes también pueden ser importantes a la hora de implementar este patrón:
- Este patrón tiene una [aplicación de ejemplo](https://github.com/mspnp/cloud-design-patterns/tree/master/leader-election) descargable.
- [Guía de escalado automático](https://msdn.microsoft.com/library/dn589774.aspx). Es posible iniciar y detener instancias de los hosts de las tareas a medida que varía la carga de la aplicación. El escalado automático puede ayudar a mantener el rendimiento y la capacidad de proceso durante los períodos de procesamiento máximo.
- [Compute Partitioning Guidance](https://msdn.microsoft.com/library/dn589773.aspx) (Orientación sobre la creación de particiones de proceso). En esta guía se describe cómo asignar tareas a los hosts de un servicio en la nube de tal forma que ayude a reducir los costes de ejecución mientras se mantiene la escalabilidad, el rendimiento, la disponibilidad y la seguridad del servicio.
- El [patrón Task-based Asynchronous](https://msdn.microsoft.com/library/hh873175.aspx).
- Un ejemplo que ilustra el [algoritmo Bully](http://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/BullyExample.html).
- Un ejemplo que ilustra el [algoritmo Ring](http://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/RingElectExample.html).
- El artículo [Apache Zookeeper on Microsoft Azure](https://msopentech.com/opentech-projects/apache-zookeeper-on-windows-azure-2/) (Apache Zookeeper en Microsoft Azure) en el sitio web de Microsoft Open Technologies.
- [Apache Curator](http://curator.apache.org/) una biblioteca cliente de Apache ZooKeeper.
- El artículo [Lease Blob (API de REST)](https://msdn.microsoft.com/library/azure/ee691972.aspx) en MSDN.
