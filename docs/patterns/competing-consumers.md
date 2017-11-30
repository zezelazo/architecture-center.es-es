---
title: Consumidores de la competencia
description: "Permite que varios consumidores simultáneos procesen los mensajes recibidos en el mismo canal de mensajería."
keywords: "Patrón de diseño"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories: messaging
ms.openlocfilehash: d72a09ef7613bebe3701634e4eac0716400e471d
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2017
---
# <a name="competing-consumers-pattern"></a>Patrón de consumidores de la competencia

[!INCLUDE [header](../_includes/header.md)]

Permite que varios consumidores simultáneos procesen los mensajes recibidos en el mismo canal de mensajería. Este patrón permite que un sistema procese varios mensajes simultáneamente a fin de optimizar el rendimiento, mejorar la escalabilidad y disponibilidad, y equilibrar la carga de trabajo.

## <a name="context-and-problem"></a>Contexto y problema

Cabe esperar que una aplicación que se ejecuta en la nube administre un gran número de solicitudes. En lugar de procesar cada solicitud de forma sincrónica, una técnica común es que la aplicación las pase mediante un sistema de mensajería a otro servicio (un servicio de consumidor) que las administra de manera asincrónica. Esta estrategia ayuda a garantizar que la lógica de negocios de la aplicación no se bloquee mientras se procesan las solicitudes.

El número de solicitudes puede variar significativamente con el tiempo por diversos motivos. Un aumento repentino de la actividad del usuario o solicitudes agregadas procedentes de varios inquilinos pueden provocar una carga de trabajo imprevisible. En horas de máxima actividad, un sistema podría tener la necesidad de procesar muchos cientos de solicitudes por segundo, mientras que en otras ocasiones el número puede ser muy pequeño. Además, la naturaleza del trabajo realizado para administrar estas solicitudes puede ser muy variable. Usar una única instancia del servicio de consumidor puede provocar que la instancia se inunde de solicitudes, o el sistema de mensajería podría sobrecargarse por una afluencia de mensajes procedentes de la aplicación. Para administrar esta carga de trabajo cambiante, el sistema puede ejecutar varias instancias del servicio de consumidor. Sin embargo, estos consumidores debe coordinarse para garantizar que cada mensaje se entrega solo a un único consumidor. La carga de trabajo también debe equilibrarse entre los consumidores para impedir que una instancia se convierte en un cuello de botella.

## <a name="solution"></a>Solución

Use una cola de mensajes para implementar el canal de comunicación entre la aplicación y las instancias del servicio de consumidor. La aplicación publica las solicitudes en forma de mensajes en la cola, y las instancias de servicio de consumidor reciben los mensajes de la cola y los procesan. Este enfoque permite que el mismo grupo de instancias de servicio de consumidor administren los mensajes desde cualquier instancia de la aplicación. En la ilustración se muestra cómo usar una cola de mensajes para distribuir trabajo a las instancias de un servicio.

![Uso de una cola de mensajes para distribuir el trabajo a las instancias de un servicio](./_images/competing-consumers-diagram.png)

Esta solución tiene las siguientes ventajas:

- Proporciona un sistema de redistribución de la carga que puede administrar grandes variaciones en el volumen de solicitudes enviadas por las instancias de la aplicación. La cola actúa como búfer entre las instancias de la aplicación y las instancias de servicio de consumidor. Esto puede ayudar a reducir el impacto sobre la disponibilidad y la capacidad de respuesta de las instancias de aplicación y las de servicio, como se describe en [Queue-based Load Leveling pattern](queue-based-load-leveling.md) (Patrón Queue-based Load Leveling). Administrar un mensaje que requiere un procesamiento de ejecución prolongada no impide que otras instancias del servicio de consumidor administren simultáneamente otros mensajes.

- Mejora la confiabilidad. Si un productor se comunica directamente con un consumidor en lugar de usar este patrón, pero no supervisa el consumidor, existe una alta probabilidad de que los mensajes se pierdan o no puedan procesarse si se produce un error en el consumidor. En este patrón, no se envían mensajes a una instancia de servicio específica. Una instancia de servicio que ha dado error no bloqueará a un productor y cualquier instancia de servicio de trabajo podrá procesar los mensajes.

- No es necesaria una coordinación compleja entre los consumidores o entre el productor y las instancias de consumidor. La cola de mensajes garantiza que cada mensaje se entrega al menos una vez.

- Es escalable. El sistema puede aumentar o disminuir de forma dinámica el número de instancias del servicio de consumidor a medida que el volumen de mensajes varía.

- Puede mejorar la resistencia si la cola de mensajes proporciona operaciones de lectura transaccionales. Si una instancia de servicio de consumidor lee y procesa el mensaje como parte de una operación transaccional y se produce un error en la instancia de servicio de consumidor, este patrón puede garantizar que el mensaje se devuelva a la cola para que otra instancia de servicio de consumidor pueda recogerlo y administrarlo.

## <a name="issues-and-considerations"></a>Problemas y consideraciones

Tenga en cuenta los puntos siguientes al decidir cómo implementar este patrón:

- **Orden de los mensajes**. El orden en que las instancias de servicio de consumidor reciben loa mensajes no está garantizado y no refleja necesariamente el orden en que se crearon los mensajes. Diseñe el sistema para asegurarse de que el procesamiento de mensajes sea idempotente, ya que de esta forma podrá eliminar cualquier dependencia en el orden en el que se administran los mensajes. Para más información, consulte [Idempotency Patterns](http://blog.jonathanoliver.com/idempotency-patterns/) (Patrones de idempotencia) en el blog de Jonathon Oliver.

    > Las colas de Microsoft Azure Service Bus pueden implementar el orden de mensajes garantizado "primero en entrar, primero en salir" mediante sesiones de mensajes. Para más información, consulte [Sesiones de uso de patrones de mensajería](https://msdn.microsoft.com/magazine/jj863132.aspx).

- **Diseño de servicios para proporcionar resistencia**. Si el sistema está diseñado para detectar y reiniciar las instancias de servicio con error, podría ser necesario implementar el procesamiento realizado por las instancias de servicio como operaciones idempotente a fin de reducir los efectos de que un único mensaje se recupere y procese más de una vez.

- **Detección de mensajes dudosos**. Un mensaje con formato incorrecto, o una tarea que requiere acceso a recursos que no están disponibles, puede hacer que una instancia de servicio produzca un error. El sistema debe impedir que dichos mensajes se devuelvan a la cola y, en su lugar, capturar y almacenar los detalles de estos mensajes en otra parte, de modo que puedan analizarse si es necesario.

- **Administración de resultados**. La instancia de servicio que administra un mensaje se separa por completo de la lógica de aplicación que genera el mensaje, y es posible que no se puedan comunicar directamente. Si la instancia de servicio genera resultados que deben pasarse de vuelta a la lógica de aplicación, esta información debe almacenarse en una ubicación que sea accesible para ambas. Para evitar que la lógica de aplicación recupere datos incompletos, el sistema debe indicar cuándo el procesamiento ha finalizado.

     > Si usa Azure, un proceso de trabajo puede pasar los resultados de vuelta a la lógica de aplicación mediante una cola de respuesta a mensajes dedicada. La lógica de aplicación debe poder correlacionar estos resultados con el mensaje original. Este escenario se describe con más detalle en [Asynchronous Messaging Primer](https://msdn.microsoft.com/library/dn589781.aspx) (Manual básico de mensajería asincrónica).

- **Escalado del sistema de mensajería**. En una solución a gran escala, una única cola de mensajes podría verse desbordada por el número de mensajes y convertirse en un cuello de botella en el sistema. En esta situación, considere la posibilidad de crear particiones del sistema de mensajería para enviar mensajes de productores específicos a una cola determinada o usar el equilibrio de carga para distribuir los mensajes entre varias colas de mensajes.

- **Garantía de confiabilidad del sistema de mensajería**. Es necesario un sistema de mensajería confiable para garantizar que después de que la aplicación pone en cola un mensaje, este no se perderá. Esto es esencial para garantizar que todos los mensajes se entregan al menos una vez.

## <a name="when-to-use-this-pattern"></a>Cuándo usar este patrón

Use este patrón cuando:

- La carga de trabajo de una aplicación se divida en tareas que se pueden ejecutar de forma asincrónica.
- Las tareas sean independientes y se puedan ejecutar en paralelo.
- El volumen de trabajo sea tan variable que requiera una solución escalable.
- La solución debe proporcionar alta disponibilidad y ser resistente si se produce un error en el procesamiento de una tarea.

Este modelo podría no ser útil en las situaciones siguientes:

- No sea fácil dividir la carga de trabajo de la aplicación en tareas discretas, o haya un alto grado de dependencia entre las tareas.
- Las tareas deban realizarse de forma sincrónica y la lógica de la aplicación deba esperar a que una tarea se complete antes de continuar.
- Las tareas se deban realizar en una secuencia concreta.

> Algunos sistemas de mensajería admiten sesiones que permiten que un productor agrupe los mensajes y garantizan que el mismo consumidor los administre todos. Este mecanismo se puede usar con los mensajes con prioridad (si se admiten) para implementar una forma de ordenación de los mensajes que los entrega en secuencia desde un productor hasta un único consumidor.

## <a name="example"></a>Ejemplo

Azure proporciona colas de almacenamiento y colas de Service Bus que pueden funcionar como un mecanismo para implementar este patrón. La lógica de la aplicación puede publicar los mensajes en una cola, y los consumidores implementados como tareas en uno o varios roles pueden recuperar los mensajes de esta cola y procesarlos. Para lograr resistencia, una cola de Service Bus permite que un consumidor use el modo `PeekLock` cuando recupera un mensaje de la cola. Este modo no quita realmente el mensaje, sino que simplemente lo oculta a otros consumidores. El consumidor original puede eliminar el mensaje cuando haya terminado de procesarlo. Si se produce un error en el consumidor, el bloque de inspección agotará el tiempo de espera y el mensaje estará visible de nuevo, de forma que otro consumidor pueda recuperarlo.

> Para más información sobre el uso de colas de Azure Service Bus, consulte [Colas, temas y suscripciones de Service Bus](https://msdn.microsoft.com/library/windowsazure/hh367516.aspx).
Para más información sobre el uso de colas de almacenamiento de Azure, consulte [Introducción a Azure Queue Storage mediante .NET](https://azure.microsoft.com/documentation/articles/storage-dotnet-how-to-use-queues/).

El siguiente código de la clase `QueueManager` de la solución CompetingConsumers disponible en [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/competing-consumers) muestra cómo puede crear una cola mediante una instancia `QueueClient` en el controlador de eventos `Start` en un rol web o de trabajo.

```csharp
private string queueName = ...;
private string connectionString = ...;
...

public async Task Start()
{
  // Check if the queue already exists.
  var manager = NamespaceManager.CreateFromConnectionString(this.connectionString);
  if (!manager.QueueExists(this.queueName))
  {
    var queueDescription = new QueueDescription(this.queueName);

    // Set the maximum delivery count for messages in the queue. A message
    // is automatically dead-lettered after this number of deliveries. The
    // default value for dead letter count is 10.
    queueDescription.MaxDeliveryCount = 3;

    await manager.CreateQueueAsync(queueDescription);
  }
  ...

  // Create the queue client. By default the PeekLock method is used.
  this.client = QueueClient.CreateFromConnectionString(
    this.connectionString, this.queueName);
}
```

El fragmento de código siguiente muestra cómo una aplicación puede crear y enviar un lote de mensajes a la cola.

```csharp
public async Task SendMessagesAsync()
{
  // Simulate sending a batch of messages to the queue.
  var messages = new List<BrokeredMessage>();

  for (int i = 0; i < 10; i++)
  {
    var message = new BrokeredMessage() { MessageId = Guid.NewGuid().ToString() };
    messages.Add(message);
  }
  await this.client.SendBatchAsync(messages);
}
```

El código siguiente muestra cómo una instancia de servicio de consumidor puede recibir mensajes de la cola siguiendo un enfoque orientado a eventos. El parámetro `processMessageTask` para el método `ReceiveMessages` es un delegado que hace referencia al código que se ejecuta cuando se recibe un mensaje. Este código se ejecuta de forma asincrónica.

```csharp
private ManualResetEvent pauseProcessingEvent;
...

public void ReceiveMessages(Func<BrokeredMessage, Task> processMessageTask)
{
  // Set up the options for the message pump.
  var options = new OnMessageOptions();

  // When AutoComplete is disabled it's necessary to manually
  // complete or abandon the messages and handle any errors.
  options.AutoComplete = false;
  options.MaxConcurrentCalls = 10;
  options.ExceptionReceived += this.OptionsOnExceptionReceived;

  // Use of the Service Bus OnMessage message pump.
  // The OnMessage method must be called once, otherwise an exception will occur.
  this.client.OnMessageAsync(
    async (msg) =>
    {
      // Will block the current thread if Stop is called.
      this.pauseProcessingEvent.WaitOne();

      // Execute processing task here.
      await processMessageTask(msg);
    },
    options);
}
...

private void OptionsOnExceptionReceived(object sender,
  ExceptionReceivedEventArgs exceptionReceivedEventArgs)
{
  ...
}
```

Tenga en cuenta que las características de escalado automática, como los que están disponibles en Azure, pueden utilizarse para iniciar y detener instancias de rol a medida que varía la longitud de cola. Para más información, consulte [Guía de escalado automático](https://msdn.microsoft.com/library/dn589774.aspx). Además, no es necesario mantener una correspondencia uno a uno entre las instancias de rol y los procesos de trabajo&mdash;una única instancia de rol puede implementar varios procesos de trabajo. Para más información, consulte [Compute Resource Consolidation pattern](compute-resource-consolidation.md) (Patrón Compute Resource Consolidation).

## <a name="related-patterns-and-guidance"></a>Orientación y patrones relacionados

Los patrones y las directrices siguientes podrían ser importantes a la hora de implementar este patrón:

- [Manual de mensajería asincrónica](https://msdn.microsoft.com/library/dn589781.aspx). Las colas de mensajes son un mecanismo de comunicaciones asincrónico. Si un servicio de consumidor debe enviar una respuesta a una aplicación, podría ser necesario implementar alguna forma de mensajería de respuesta. En Asynchronous Messaging Primer se proporciona información sobre cómo implementar la mensajería de solicitud/respuesta con colas de mensajes.

- [Guía de escalado automático](https://msdn.microsoft.com/library/dn589774.aspx). Se podrían iniciar y detener instancias de un servicio de consumidor dado que la longitud de la cola en la que las aplicaciones publican los mensajes varía. El escalado automático puede ayudar a mantener el rendimiento durante los períodos de procesamiento de máximo.

- [Patrón Compute Resource Consolidation](compute-resource-consolidation.md). Se podrían consolidar varias instancias de un servicio de consumidor en un único proceso para reducir los costes y la sobrecarga de administración. El patrón de consolidación de los recursos de proceso describe las ventajas e inconvenientes de este enfoque.

- [Patrón Queue-based Load Leveling](queue-based-load-leveling.md). La introducción de una cola de mensajes puede agregar resistencia al sistema, al permitir que las instancias de servicio administren volúmenes muy diversos de solicitudes desde instancias de aplicación. La cola de mensajes actúa como búfer, que redistribuye la carga. El patrón de redistribución de carga basada en colas describe este escenario con mayor detalle.

- Este patrón lleva asociada una [aplicación de ejemplo](https://github.com/mspnp/cloud-design-patterns/tree/master/competing-consumers).
