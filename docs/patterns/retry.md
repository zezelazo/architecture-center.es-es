---
title: Retry
description: "Permite que una aplicación trate los errores temporales anticipados cuando intenta conectarse a un servicio o un recurso de red, mediante el reintento de forma transparente de una operación que anteriormente produjo error."
keywords: "Patrón de diseño"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- resiliency
ms.openlocfilehash: 73fdcbcc2bd75593a4c8e33dc2259c90593e14db
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/23/2018
---
# <a name="retry-pattern"></a>Patrón Retry

[!INCLUDE [header](../_includes/header.md)]

Permite que una aplicación trate los errores transitorios cuando intenta conectarse a un servicio o un recurso de red, mediante el reintento de forma transparente de una operación con error. Esto puede mejorar la estabilidad de la aplicación.

## <a name="context-and-problem"></a>Contexto y problema

Una aplicación que se comunica con elementos que se ejecutan en la nube tiene que ser vulnerable a los errores transitorios que puedan producirse en este entorno. Entre los errores transitorios cabe mencionar la pérdida momentánea de conectividad de red en componentes y servicios, la falta de disponibilidad temporal de un servicio o los tiempos de espera que surgen cuando un servicio está ocupado.

Estos errores normalmente se corrigen automáticamente, y si se repite la acción que desencadena un error tras un retraso adecuado, es probable que tenga éxito. Por ejemplo, un servicio de base de datos que procesa un número elevado de solicitudes simultáneas puede implementar una estrategia de limitación que temporalmente rechace las sucesivas solicitudes hasta que su carga de trabajo haya disminuido. Una aplicación que intenta acceder a la base de datos podría no conectarse, pero si lo intenta de nuevo después de un retraso podría tener éxito.

## <a name="solution"></a>Solución

En la nube, los errores transitorios son algo habitual y una aplicación debería diseñarse para tratarlos con elegancia y transparencia. De esta forma, se reducen los efectos que pueden tener los errores sobre las tareas empresariales que realiza la aplicación.

Si una aplicación detecta un error al intentar enviar una solicitud a un servicio remoto, puede tratar el error mediante las estrategias siguientes:

- **Cancelar**. Si el error indica que el error no es transitorio o que no es probable que tenga éxito si se repite, la aplicación debe cancelar la operación y notificar una excepción. Por ejemplo, un error de autenticación ocasionado al proporcionarse credenciales no válidas no es probable que tenga éxito sin importar cuántas veces se intente.

- **Reintentar**. Si el error específico notificado es inusual o poco frecuente, se podría deber a circunstancias poco habituales, como un paquete de red que se daña mientras se está transmitiendo. En este caso, la aplicación podría reintentar la solicitud con error inmediatamente porque es improbable que se repita el mismo error y lo más probable es que la solicitud tenga éxito.

- **Reintentar después de un retraso.** Si el error es debido a uno de los muchos errores habituales de conectividad o disponibilidad, puede que la red o el servicio necesiten un corto período de tiempo mientras se corrigen los problemas de conectividad o se borra el trabajo pendiente. La aplicación debe esperar el momento adecuado antes de reintentar la solicitud.

Para los errores transitorios más comunes, se debe elegir el período entre reintentos a fin distribuir las solicitudes de varias instancias de la aplicación lo más uniformemente posible. De esta forma se reduce la posibilidad de que un servicio ocupado siga sobrecargado. Si muchas instancias de una aplicación sobrecargan continuamente un servicio con solicitudes de reintento, el servicio tardará más en recuperarse.

Si la solicitud sigue sin funcionar, la aplicación puede esperar y realizar otro intento. Si es necesario, este proceso puede repetirse aumentando los retrasos entre reintentos, hasta que se haya intentado un número máximo de solicitudes. El retraso se puede aumentar de manera exponencial o incremental, según el tipo de error y la probabilidad de que se corrija durante este tiempo.

En el siguiente diagrama se ilustra la invocación de una operación en un servicio hospedado mediante este patrón. Si la solicitud no tiene éxito después de un número predefinido de intentos, la aplicación debe tratar el error como una excepción y tratarlo como corresponda.

![Figura 1: Invocación de una operación en un servicio hospedado con el patrón de reintento](./_images/retry-pattern.png)

La aplicación debe encapsular todos los intentos de acceso a un servicio remoto en un código que implemente una directiva de reintento que coincida con una de las estrategias enumeradas anteriormente. Las solicitudes enviadas a distintos servicios pueden estar sujetas a diferentes directivas. Algunos proveedores proporcionan bibliotecas que implementan directivas de reintento, donde la aplicación puede especificar el número máximo de reintentos, el tiempo entre reintentos y otros parámetros.

Una aplicación debe registrar los detalles de los errores y las operaciones con errores. Esta información es útil para los operadores. Si un servicio está con frecuencia no disponible o ocupado, suele ser porque el servicio ha agotado sus recursos. Se puede reducir la frecuencia de estos errores mediante el escalado horizontal del servicio. Por ejemplo, si un servicio de base de datos está continuamente sobrecargado, podría ser beneficioso particionar la base de datos y repartir la carga entre varios servidores.

> [Microsoft Entity Framework](https://docs.microsoft.com/ef/) proporciona los medios para volver a intentar las operaciones de base de datos. Además, la mayoría de los SDK de cliente y de servicios de Azure incluye un mecanismo de reintento. Para más información, consulte [Retry guidance for specific services](https://docs.microsoft.com/azure/architecture/best-practices/retry-service-specific) (Guía de reintentos para servicios específicos).

## <a name="issues-and-considerations"></a>Problemas y consideraciones

A la hora de decidir cómo implementar este patrón, debe considerar los siguientes puntos:

La directiva de reintentos se debe optimizar para que coincida con los requisitos empresariales de la aplicación y la naturaleza del error. Si las operaciones no son críticas, es mejor provocar el fracaso y responder rápido a los errores en lugar de reintentarlas varias veces y que afecten al rendimiento de la aplicación. Por ejemplo, en una aplicación web interactiva que accede a un servicio remoto, es mejor que el error se produzca rápido tras un pequeño número de reintentos con solo un corto retraso entre ellos y mostrar un mensaje adecuado al usuario (por ejemplo, "inténtelo de nuevo más tarde"). En una aplicación por lotes, puede ser más adecuado aumentar el número de reintentos con un retraso que aumente exponencialmente entre intentos.

Una directiva de reintentos agresiva con un retraso mínimo entre intentos, y un gran número de reintentos, podría degradar aún más un servicio ocupado que se ejecuta en su capacidad o próximo a esta. Esta directiva de reintentos también podría afectar a la capacidad de respuesta de la aplicación si continuamente intenta realizar una operación con error.

Si una solicitud sigue sin funcionar después de un número considerable de reintentos, lo mejor para la aplicación es impedir que las solicitudes adicionales vayan al mismo recurso y simplemente notificar un error inmediatamente. Cuando expira el período, la aplicación puede permitir que pasen una o varias solicitudes de manera provisional para ver si se procesan correctamente. Para más información sobre esta estrategia, consulte [Circuit Breaker pattern](circuit-breaker.md) (Patrón Circuit Breaker).

Considere si la operación es idempotente. Si es así, es intrínsecamente seguro volver a intentarla. En caso contrario, los reintentos podrían provocar que la operación se ejecutara más de una vez, con efectos secundarios imprevistos. Por ejemplo, un servicio podría recibir la solicitud, procesarla correctamente, pero no poder enviar una respuesta. En ese momento, la lógica de reintento podría volver a enviar la solicitud, suponiendo que no se reciba la primera solicitud.

Una solicitud a un servicio puede dar error por diversos motivos y generar diferentes excepciones según la naturaleza del error. Algunas excepciones indican un error que puede resolverse rápidamente, mientras que otras indican que el error está durando mucho. Para la directiva de reintentos resulta útil ajustar el tiempo entre reintentos en función del tipo de la excepción.

Considere de qué modo reintentar una operación que forma parte de una transacción afectará a la coherencia de la transacción en general. Ajuste la directiva de reintentos en las operaciones transaccionales para maximizar la probabilidad de éxito y reducir la necesidad de deshacer todos los pasos de transacción.

Asegúrese de que todo el código de reintento esté sobradamente probado en diversas condiciones de error. Compruebe que no afecte gravemente al rendimiento o la confiabilidad de la aplicación, que no provoque una carga excesiva en los servicios y recursos ni que genere condiciones de carrera o cuellos de botella.

Implemente la lógica de reintento solo donde se entienda el contexto completo de una operación con error. Por ejemplo, si una tarea que contiene una directiva de reintentos invoca otra tarea que también contiene una directiva de reintentos, este nivel adicional de reintentos puede agregar retrasos prolongados al procesamiento. Lo mejor sería configurar la tarea de nivel inferior para que fracase y responda a rápido a los errores, y notificar el motivo del error a la tarea que lo ha invocado. Esta tarea de nivel superior puede luego tratar el error según su propia directiva.

Es importante registrar todos los errores de conectividad que provocan un reintento de modo que se puedan identificar los problemas subyacentes con la aplicación, los servicios o los recursos.

Investigue los errores que es más probables que se produzcan con un servicio o un recurso a fin de detectar si es probable que sean de larga duración o terminales. Si lo son, es mejor tratar el error como una excepción. La aplicación puede notificar o registrar la excepción y, a continuación, intentar continuar bien mediante la invocando de un servicio alternativo (si está disponible) o la oferta de una funcionalidad degradada. Para más información sobre cómo detectar y tratar los errores de larga duración, consulte [Circuit Breaker pattern](circuit-breaker.md) (Patrón Circuit Breaker).

## <a name="when-to-use-this-pattern"></a>Cuándo usar este patrón

Use este patrón cuando una aplicación pueda experimentar errores transitorios mientras interactúa con un servicio remoto o accede a un recurso remoto. Cabe esperar que estos errores sean de corta duración, y repetir una solicitud que anteriormente ha dado error podría tener éxito en un intento posterior.

Este patrón podría no ser útil:

- Cuando es probable que un error sea de larga duración, ya que esto puede afectar a la capacidad de respuesta de una aplicación. La aplicación podría estar desperdiciando tiempo y recursos intentando repetir la solicitud que es probable que produzca error.
- Para tratar errores que no son debidos a errores transitorios, como excepciones internas debidas a errores de la lógica de negocios de una aplicación.
- Como alternativa para resolver problemas de escalabilidad en un sistema. Si una aplicación experimenta errores frecuentes de disponibilidad, suele ser un indicio de que el servicio o recurso al que se accede se debe escalar verticalmente.

## <a name="example"></a>Ejemplo

Este ejemplo en C# muestra una implementación del patrón Retry. El método `OperationWithBasicRetryAsync`, que se muestra a continuación, invoca un servicio externo de manera asincrónica mediante el método `TransientOperationAsync`. Los detalles del método `TransientOperationAsync` serán específicos del servicio y se omitirán del código de ejemplo.

```csharp
private int retryCount = 3;
private readonly TimeSpan delay = TimeSpan.FromSeconds(5);

public async Task OperationWithBasicRetryAsync()
{
  int currentRetry = 0;

  for (;;)
  {
    try
    {
      // Call external service.
      await TransientOperationAsync();

      // Return or break.
      break;
    }
    catch (Exception ex)
    {
      Trace.TraceError("Operation Exception");

      currentRetry++;

      // Check if the exception thrown was a transient exception
      // based on the logic in the error detection strategy.
      // Determine whether to retry the operation, as well as how
      // long to wait, based on the retry strategy.
      if (currentRetry > this.retryCount || !IsTransient(ex))
      {
        // If this isn't a transient error or we shouldn't retry, 
        // rethrow the exception.
        throw;
      }
    }

    // Wait to retry the operation.
    // Consider calculating an exponential delay here and
    // using a strategy best suited for the operation and fault.
    await Task.Delay(delay);
  }
}

// Async method that wraps a call to a remote service (details not shown).
private async Task TransientOperationAsync()
{
  ...
}
```

La instrucción que invoca este método está contenida en un bloque try/catch encapsulado en un bucle for. El bucle for se cierra si la llamada al método `TransientOperationAsync` tiene éxito sin producir una excepción. Si el método `TransientOperationAsync` produce un error, el bloque catch examina el motivo del error. Si se considera un error transitorio, el código espera un breve retraso antes de volver a intentar la operación.

El bucle for también hace un seguimiento del número de veces que se ha intentado la operación y, si el código produce error tres veces, se supone que es de larga duración. Si la excepción no es transitoria o es de larga duración, el controlador catch produce una excepción. Esta excepción cierra el bucle for y debe capturarla el código que invoca el método `OperationWithBasicRetryAsync`.

El método `IsTransient`, que se muestra a continuación, busca un conjunto específico de excepciones que son pertinentes para el entorno en el que se ejecuta el código. La definición de una excepción transitoria varía de acuerdo con los recursos a los que se accede y el entorno en el que se realiza la operación.

```csharp
private bool IsTransient(Exception ex)
{
  // Determine if the exception is transient.
  // In some cases this is as simple as checking the exception type, in other
  // cases it might be necessary to inspect other properties of the exception.
  if (ex is OperationTransientException)
    return true;

  var webException = ex as WebException;
  if (webException != null)
  {
    // If the web exception contains one of the following status values
    // it might be transient.
    return new[] {WebExceptionStatus.ConnectionClosed,
                  WebExceptionStatus.Timeout,
                  WebExceptionStatus.RequestCanceled }.
            Contains(webException.Status);
  }

  // Additional exception checking logic goes here.
  return false;
}
```

## <a name="related-patterns-and-guidance"></a>Orientación y patrones relacionados

- [Patrón Circuit Breaker](circuit-breaker.md). El patrón Retry es útil para tratar los errores transitorios. Si se espera que un error sea de larga duración, puede que sea más adecuado implementar el patrón Circuit Breaker. El patrón Retry también puede utilizarse junto con un interruptor para proporcionar un enfoque integral para el tratamiento de errores.
- [Retry guidance for specific services](https://docs.microsoft.com/azure/architecture/best-practices/retry-service-specific) (Guía de reintentos para servicios específicos)
- [Connection Resiliency](https://docs.microsoft.com/ef/core/miscellaneous/connection-resiliency) (Resistencia de la conexión)
