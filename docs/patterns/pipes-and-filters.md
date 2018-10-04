---
title: Canalizaciones y filtros
description: Desglosa una tarea que realiza un procesamiento complejo en una serie de elementos independientes que se pueden volver a utilizar.
keywords: Patrón de diseño
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- design-implementation
- messaging
ms.openlocfilehash: fd616676f9487bdfe1bf23b3d0fec6c65b97a8f4
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/28/2018
ms.locfileid: "47429577"
---
# <a name="pipes-and-filters-pattern"></a>Patrón Pipes and Filters

[!INCLUDE [header](../_includes/header.md)]

Descompone una tarea que realiza un procesamiento complejo en una serie de elementos independientes que se pueden volver a utilizar. Este patrón puede mejorar el rendimiento, la escalabilidad y la capacidad de reutilización al permitir que los elementos de tarea que realizan el procesamiento se implementen y escalen por separado.

## <a name="context-and-problem"></a>Contexto y problema

Una aplicación debe realizar diversas tareas de complejidad variable sobre la información que procesa. Un método sencillo pero inflexible de implementar una aplicación es realizar este procesamiento como un módulo monolítico. Sin embargo, este método probablemente reducirá las posibilidades de refactorización del código, su optimización o su reutilización si partes del mismo procesamiento se necesitan en otro lugar dentro de la aplicación.

En la ilustración se muestran los problemas con el procesamiento de datos mediante el enfoque monolítico. Una aplicación recibe y procesa datos de dos orígenes. Los datos de cada origen se procesan mediante un módulo independiente que lleva a cabo una serie de tareas para transformar estos datos, antes de pasar el resultado a la lógica de negocios de la aplicación.

![Figura 1: Una solución implementada mediante módulos monolíticos](./_images/pipes-and-filters-modules.png)

Algunas de las tareas que realizan los módulos monolíticos son funcionalmente muy similares, pero los módulos se han diseñado por separado. El código que implementa las tareas se acopla estrechamente en un módulo y se ha desarrollado sin apenas considerar su reutilización o escalabilidad.

Sin embargo, las tareas de procesamiento realizadas por cada módulo, o los requisitos de implementación de cada tarea, podrían cambiar a medida que se actualizan los requisitos empresariales. Es posible que algunas tareas sean de proceso intensivo y puedan beneficiarse de su ejecución en hardware de gran potencia, mientras que otras no necesiten recursos tan caros. Además, puede que en el futuro se necesite procesamiento adicional o que cambie el orden en el que el procesamiento realiza las tareas. Se necesita una solución que aborde estos problemas y aumente las posibilidades de reutilización del código.

## <a name="solution"></a>Solución

Desglose el procesamiento que requiere cada flujo en un conjunto de componentes (o filtros) independientes, que realice cada uno una única tarea. Al estandarizar el formato de los datos que recibe y envía cada componente, estos filtros se pueden combinar en una canalización. Esto contribuye a evitar la duplicación de código y facilita la eliminación, la sustitución o la integración de componentes adicionales en caso de que cambien los requisitos de procesamiento. En la ilustración siguiente se muestra una solución implementada mediante canalizaciones y filtros.

![Figura 2: Una solución implementada mediante canalizaciones y filtros](./_images/pipes-and-filters-solution.png)


El tiempo que tarda en procesarse una única solicitud depende de la velocidad del filtro más lento de la canalización. Uno o más filtros podrían formar un cuello de botella, en especial si un gran número de solicitudes aparece en un flujo de un origen de datos en particular. Una de las principales ventajas de la estructura de canalizaciones es que ofrece oportunidades para ejecutar instancias en paralelo de filtros lentos, que permite que el sistema reparta la carga y mejore el rendimiento.

Los filtros que forman una canalización se pueden ejecutar en equipos diferentes, de forma que se pueden escalar de manera independiente y aprovechar la elasticidad que proporcionan muchos entornos de nube. Un filtro que consuma muchos recursos informáticos se puede ejecutar en hardware de alto rendimiento, mientras que otros filtros menos exigentes se pueden hospedar en hardware básico menos costoso. Los filtros no tienen que estar siquiera en el mismo centro de datos o la misma ubicación geográfica, lo que permite que cada elemento de una canalización se ejecute en un entorno próximo a los recursos que necesita.  En la ilustración siguiente se muestra un ejemplo aplicado a la canalización de los datos desde el origen 1.

![La figura 3 muestra un ejemplo aplicado a la canalización de los datos desde el origen 1](./_images/pipes-and-filters-load-balancing.png)

Si la entrada y la salida de un filtro se estructuran como un flujo, se puede realizar el procesamiento de cada filtro en paralelo. El primer filtro de la canalización puede iniciar su trabajo y generar sus resultados, que se pasan directamente al filtro siguiente de la secuencia antes de que el primer filtro haya completado su trabajo.

Otra ventaja es la resistencia que puede proporcionar este modelo. Si se produce un error en un filtro o la máquina en la que se ejecuta ya no está disponible, la canalización puede volver a programar el trabajo que estaba realizando el filtro y dirigir este trabajo a otra instancia del componente. El error de un único filtro no da lugar necesariamente a un error de la canalización entera.

El uso del patrón Pipes and Filters en combinación con el [patrón Compensating Transaction](compensating-transaction.md) es un enfoque alternativo a la implementación de transacciones distribuidas. Una transacción distribuida se puede dividir en tareas independientes compensables, cada una de los cuales se puede implementar mediante un filtro que también implementa el patrón Compensating Transaction. Los filtros de una canalización se pueden implementar como tareas hospedadas diferentes que se ejecutan cerca de los datos que mantienen.

## <a name="issues-and-considerations"></a>Problemas y consideraciones

A la hora de decidir cómo implementar este patrón, debe considerar los siguientes puntos:
- **Complejidad**. La mayor flexibilidad que proporciona este patrón también puede presentar complejidad, especialmente si los filtros de una canalización se distribuyen entre diferentes servidores.

- **Confiabilidad**. Use una infraestructura que garantice que el flujo de datos entre los filtros de una canalización no se pierda.

- **Idempotencia**. Si se produce un error en un filtro de una canalización después de recibir un mensaje y el trabajo se vuelve a programar para otra instancia del filtro, puede que parte del trabajo ya se haya completado. Si este trabajo actualiza algún aspecto del estado global (por ejemplo, la información almacenada en una base de datos), se podría repetir la misma actualización. Un problema similar puede surgir si se produce un error en un filtro después de publicar sus resultados en el filtro siguiente de la canalización, pero antes de que indique que ha completado su trabajo correctamente. En estos casos, otra instancia del filtro podría repetir el mismo trabajo, lo que provoca que los mismos resultados se publiquen dos veces. Esto podría dar lugar a que los sucesivos filtros de la canalización procesaran los mismos datos dos veces. Por lo tanto, los filtros de una canalización se deben diseñar para que sean idempotentes. Para más información, consulte los [patrones de idempotencia](https://blog.jonathanoliver.com/idempotency-patterns/) en el blog de Jonathan Oliver.

- **Mensajes repetidos**. Si se produce un error en un filtro de una canalización después de publicarse un mensaje en la siguiente fase de esta, podría ejecutarse otra instancia del filtro y publicarse una copia del mismo mensaje en la canalización. Como consecuencia, dos instancias del mismo mensaje pasarían al siguiente filtro. Para evitar esto, la canalización debe detectar y eliminar los mensajes duplicados.

    >  Si va a implementar la canalización mediante colas de mensajes (como colas de Microsoft Azure Service Bus), la infraestructura de puesta en cola de mensajes podría proporcionar la detección y eliminación automáticas de los mensajes duplicados.

- **Contexto y estado**. En una canalización, cada filtro se ejecuta básicamente de forma aislada y no se debe hacer ninguna suposición sobre cómo se invocó. Esto significa que cada filtro debe proporcionarse con suficiente contexto para realizar su trabajo. Este contexto puede incluir una gran cantidad de información de estado.

## <a name="when-to-use-this-pattern"></a>Cuándo usar este patrón

Use este patrón en los siguientes supuestos:
- El procesamiento que requiera una aplicación se pueda desglosar fácilmente en un conjunto de pasos independientes.

- Los pasos de procesamiento que realiza una aplicación tengan requisitos de escalabilidad diferentes.

    >  Los filtros que se van a escalar juntos se pueden agrupar en el mismo proceso. Para más información, consulte [Compute Resource Consolidation pattern](compute-resource-consolidation.md) (Patrón Compute Resource Consolidation).

- Se requiere flexibilidad para permitir la reordenación de los pasos de procesamiento que realiza una aplicación o la funcionalidad para agregar y quitar pasos.

- El sistema puede beneficiarse de la distribución de los pasos de procesamiento entre diferentes servidores.

- Se requiere una solución confiable que minimice los efectos de los errores en un paso durante el procesamiento de los datos.

Este modelo podría no ser útil en las situaciones siguientes:
- Los pasos de procesamiento que realiza una aplicación no son independientes, o se deben realizar juntos como parte de la misma transacción.

- La cantidad de información de contexto o estado que requiera un paso convierte a este enfoque en ineficaz. Podría ser posible conservar la información de estado en una base de datos, pero no use esta estrategia si la carga adicional en la base de datos provoca una contención excesiva.

## <a name="example"></a>Ejemplo

Puede usar una secuencia de colas de mensajes para proporcionar la infraestructura necesaria para implementar una canalización. Una cola de mensajes inicial recibe mensajes no procesados. Un componente que se implementa como una tarea de filtro escucha un mensaje en esta cola, realiza su trabajo y, a continuación, publica el mensaje transformado en la cola siguiente de la secuencia. Otra tarea de filtro puede escuchar mensajes en esta cola, procesarlos, publicar los resultados en otra cola, y así sucesivamente hasta que los datos completamente transformados aparezcan en el mensaje final de la cola. En la ilustración siguiente se muestra cómo implementar una canalización mediante colas de mensajes.

![Figura 4: Implementación de una canalización mediante colas de mensajes](./_images/pipes-and-filters-message-queues.png)


Si va a compilar una solución en Azure, puede usar colas de Service Bus para proporcionar un mecanismo de puesta en cola confiable y escalable. La clase `ServiceBusPipeFilter` que se muestra a continuación en C# ilustra cómo puede implementar un filtro que recibe mensajes de entrada de una cola, procesa estos mensajes y publica los resultados en otra cola.

>  La clase `ServiceBusPipeFilter` se define en el proyecto PipesAndFilters.Shared disponible en [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/pipes-and-filters).

```csharp
public class ServiceBusPipeFilter
{
  ...
  private readonly string inQueuePath;
  private readonly string outQueuePath;
  ...
  private QueueClient inQueue;
  private QueueClient outQueue;
  ...

  public ServiceBusPipeFilter(..., string inQueuePath, string outQueuePath = null)
  {
     ...
     this.inQueuePath = inQueuePath;
     this.outQueuePath = outQueuePath;
  }

  public void Start()
  {
    ...
    // Create the outbound filter queue if it doesn't exist.
    ...
    this.outQueue = QueueClient.CreateFromConnectionString(...);

    ...
    // Create the inbound and outbound queue clients.
    this.inQueue = QueueClient.CreateFromConnectionString(...);
  }

  public void OnPipeFilterMessageAsync(
    Func<BrokeredMessage, Task<BrokeredMessage>> asyncFilterTask, ...)
  {
    ...

    this.inQueue.OnMessageAsync(
      async (msg) =>
    {
      ...
      // Process the filter and send the output to the
      // next queue in the pipeline.
      var outMessage = await asyncFilterTask(msg);

      // Send the message from the filter processor
      // to the next queue in the pipeline.
      if (outQueue != null)
      {
        await outQueue.SendAsync(outMessage);
      }

      // Note: There's a chance that the same message could be sent twice
      // or that a message gets processed by an upstream or downstream
      // filter at the same time.
      // This would happen in a situation where processing of a message was
      // completed, it was sent to the next pipe/queue, and then failed
      // to complete when using the PeekLock method.
      // Idempotent message processing and concurrency should be considered
      // in a real-world implementation.
    },
    options);
  }

  public async Task Close(TimeSpan timespan)
  {
    // Pause the processing threads.
    this.pauseProcessingEvent.Reset();

    // There's no clean approach for waiting for the threads to complete
    // the processing. This example simply stops any new processing, waits
    // for the existing thread to complete, then closes the message pump
    // and finally returns.
    Thread.Sleep(timespan);

    this.inQueue.Close();
    ...
  }

  ...
}
```

El método `Start` de la clase `ServiceBusPipeFilter` conecta a un par de colas de entrada y salida y el método `Close` desconecta de la cola de entrada. El método `OnPipeFilterMessageAsync` realiza el procesamiento real de mensajes y el parámetro `asyncFilterTask` para este método especifica el procesamiento que se va a realizar. El método `OnPipeFilterMessageAsync` espera los mensajes entrantes en la cola de entrada, ejecuta el código especificado por el parámetro `asyncFilterTask` sobre cada mensaje que llega y publica los resultados en la cola de salida. Las colas propiamente dichas se especifican mediante el constructor.

La solución de ejemplo implementa filtros en un conjunto de roles de trabajo. Cada rol de trabajo se puede escalar de forma independiente, según la complejidad del procesamiento de negocio que realiza o los recursos necesarios para su procesamiento. Además, se pueden ejecutar varias instancias de cada rol de trabajo en paralelo para mejorar el rendimiento.

El código siguiente muestra un rol de trabajo de Azure denominado `PipeFilterARoleEntry`, definido en el proyecto PipeFilterA de la solución de ejemplo.

```csharp
public class PipeFilterARoleEntry : RoleEntryPoint
{
  ...
  private ServiceBusPipeFilter pipeFilterA;

  public override bool OnStart()
  {
    ...
    this.pipeFilterA = new ServiceBusPipeFilter(
      ...,
      Constants.QueueAPath,
      Constants.QueueBPath);

    this.pipeFilterA.Start();
    ...
  }

  public override void Run()
  {
    this.pipeFilterA.OnPipeFilterMessageAsync(async (msg) =>
    {
      // Clone the message and update it.
      // Properties set by the broker (Deliver count, enqueue time, ...)
      // aren't cloned and must be copied over if required.
      var newMsg = msg.Clone();

      await Task.Delay(500); // DOING WORK

      Trace.TraceInformation("Filter A processed message:{0} at {1}",
        msg.MessageId, DateTime.UtcNow);

      newMsg.Properties.Add(Constants.FilterAMessageKey, "Complete");

      return newMsg;
    });

    ...
  }

  ...
}
```

Este rol contiene un objeto `ServiceBusPipeFilter`. El método `OnStart` del rol conecta a las colas para recibir mensajes de entrada y publicar mensajes de salida (los nombres de las colas se definen en la clase `Constants`). El método `Run` invoca el método `OnPipeFilterMessagesAsync` para realizar algún procesamiento en cada mensaje recibido (en este ejemplo, el procesamiento se simula esperando un breve espacio de tiempo). Una vez completado el procesamiento, se construye un nuevo mensaje que contiene los resultados (en este caso, al mensaje de entrada se le ha agregado una propiedad personalizada), y este mensaje se publica en la cola de salida.

El código de ejemplo contiene otro rol de trabajo denominado `PipeFilterBRoleEntry` en el proyecto PipeFilterB. Este rol es similar a `PipeFilterARoleEntry`, excepto por el hecho de que realiza un procesamiento diferente en el método `Run`. En la solución de ejemplo, estos dos roles se combinan para construir una canalización; la cola de salida del rol `PipeFilterARoleEntry` es la cola de entrada del rol `PipeFilterBRoleEntry`.

La solución de ejemplo también proporciona dos roles adicionales llamados `InitialSenderRoleEntry` (en el proyecto InitialSender) y `FinalReceiverRoleEntry` (en el proyecto FinalReceiver). El rol `InitialSenderRoleEntry` proporciona el mensaje inicial de la canalización. El método `OnStart` conecta a una única cola y el método `Run` publica un método en esta cola. Esta cola es la cola de entrada que usa el rol `PipeFilterARoleEntry`, así que al publicarse un mensaje, será este `PipeFilterARoleEntry` rol el que lo reciba y procese. El mensaje procesado pasa entones por el rol `PipeFilterBRoleEntry`.

La cola de entrada del rol `FinalReceiveRoleEntry` es la cola de salida del rol `PipeFilterBRoleEntry`. El método `Run` del rol `FinalReceiveRoleEntry`, que se muestra a continuación, recibe el mensaje y realiza algún procesamiento final. Luego escribe los valores de las propiedades personalizadas que agregan los filtros de la canalización en la salida de seguimiento.

```csharp
public class FinalReceiverRoleEntry : RoleEntryPoint
{
  ...
  // Final queue/pipe in the pipeline to process data from.
  private ServiceBusPipeFilter queueFinal;

  public override bool OnStart()
  {
    ...
    // Set up the queue.
    this.queueFinal = new ServiceBusPipeFilter(...,Constants.QueueFinalPath);
    this.queueFinal.Start();
    ...
  }

  public override void Run()
  {
    this.queueFinal.OnPipeFilterMessageAsync(
      async (msg) =>
      {
        await Task.Delay(500); // DOING WORK

        // The pipeline message was received.
        Trace.TraceInformation(
          "Pipeline Message Complete - FilterA:{0} FilterB:{1}",
          msg.Properties[Constants.FilterAMessageKey],
          msg.Properties[Constants.FilterBMessageKey]);

        return null;
      });
    ...
  }

  ...
}
```

## <a name="related-patterns-and-guidance"></a>Orientación y patrones relacionados

Los patrones y las directrices siguientes también pueden ser importantes a la hora de implementar este modelo:
- Se encuentra disponible un ejemplo que demuestra este patrón en [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/pipes-and-filters).
- [Patrón de consumidores de la competencia](competing-consumers.md). Una canalización puede contener varias instancias de uno o varios filtros. Este enfoque es útil para la ejecución en paralelo de instancias de filtros lentos, que permiten que el sistema reparta la carga y mejore el rendimiento. Cada instancia de un filtro competirá por la entrada con las demás instancias; dos instancias de un filtro nunca deberían poder procesar los mismos datos. Proporciona una explicación de este enfoque.
- [Patrón Compute Resource Consolidation](compute-resource-consolidation.md). Los filtros que podrían escalarse juntos, se pueden agrupar en el mismo proceso. Proporciona más información sobre las ventajas e inconvenientes de esta estrategia.
- [Patrón Compensating Transaction](compensating-transaction.md). Un filtro puede implementarse como una operación que se puede invertir o que tiene una operación de compensación que restaura el estado a una versión anterior en caso de error. Explica cómo se puede implementar para mantener o lograr coherencia definitiva.
- [Patrones de idempotencia](https://blog.jonathanoliver.com/idempotency-patterns/) en el blog de Jonathan Oliver.
