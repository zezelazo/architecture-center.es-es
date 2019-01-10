---
title: Patrón External Configuration Store
titleSuffix: Cloud Design Patterns
description: Extrae la información de configuración del paquete de implementación de la aplicación y la lleva a una ubicación centralizada.
keywords: Patrón de diseño
author: dragon119
ms.date: 06/23/2017
ms.custom: seodec18
ms.openlocfilehash: 7e37e5bc052a9d8e8747a3a4ac3d79a311185ea4
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/04/2019
ms.locfileid: "54011317"
---
# <a name="external-configuration-store-pattern"></a>Patrón External Configuration Store

[!INCLUDE [header](../_includes/header.md)]

Extrae la información de configuración del paquete de implementación de la aplicación y la lleva a una ubicación centralizada. Este patrón puede proporcionar oportunidades para facilitar la administración y el control de los datos de configuración y para compartir los datos de configuración entre aplicaciones e instancias de aplicación.

## <a name="context-and-problem"></a>Contexto y problema

La mayoría de los entornos de tiempo de ejecución de aplicaciones incluyen información de configuración que está contenida en los archivos implementados con la aplicación. En algunos casos, es posible editar estos archivos para cambiar el comportamiento de la aplicación después de que se ha implementado. Sin embargo, los cambios en la configuración requieren volver a implementar la aplicación, lo que suele dar lugar a un tiempo de inactividad inaceptable y más sobrecarga administrativa.

Los archivos de configuración local también limitan la configuración a una única aplicación, pero en ocasiones sería útil compartir los valores de configuración entre varias aplicaciones. Algunos ejemplos incluyen cadenas de conexión de base de datos, información de temas de interfaz de usuario o las direcciones URL de las colas y el almacenamiento que e usan en un conjunto relacionado de aplicaciones.

Administrar los cambios en las configuraciones locales entre varias instancias en ejecución de la aplicación puede resultar difícil, en especial en escenarios hospedados en la nube. Puede dar lugar a instancias que usan distintos valores de configuración mientras se implementa la actualización.

Además, las actualizaciones de aplicaciones y componentes pueden requerir cambios en los esquemas de configuración. Muchos sistemas de configuración no admiten diferentes versiones de la información de configuración.

## <a name="solution"></a>Solución

Almacene la información de configuración en almacenamiento externo y proporcione una interfaz que puede usarse para leer y actualizar de manera rápida y eficaz los valores de configuración. El tipo de almacén externo depende del entorno de hospedaje y de tiempo de ejecución de la aplicación. En un escenario hospedado en la nube, suele ser un servicio de almacenamiento basado en la nube, pero podría ser una base de datos hospedada u otro sistema.

La memoria auxiliar que elija para la información de configuración debe tener una interfaz que proporcione acceso coherente y fácil de usar. Debe exponer la información en un formato estructurado y correctamente escrito. Puede que también la implementación deba autorizar el acceso a los usuarios con el fin de proteger los datos de configuración y ser lo suficientemente flexible como para permitir el almacenamiento de varias versiones de la configuración (por ejemplo, desarrollo, ensayo o producción, incluidas varias versiones de cada una).

> Muchos sistemas de configuración integrados leen los datos cuando se inicia la aplicación y los almacenan en caché en memoria para proporcionar acceso rápido y minimizar el impacto en el rendimiento de la aplicación. Según el tipo de memoria auxiliar usado y la latencia de este almacén, podría resultar útil implementar un mecanismo de almacenamiento en caché en el almacén de configuración externo. Para obtener más información, consulte [Caching Guidance](https://msdn.microsoft.com/library/dn589802.aspx)(Guía de almacenamiento en caché). La ilustración muestra información general sobre el patrón de almacén de configuración externo con caché local opcional.

![Información general sobre el patrón External Configuration Store con caché local opcional](./_images/external-configuration-store-overview.png)

## <a name="issues-and-considerations"></a>Problemas y consideraciones

Tenga en cuenta los puntos siguientes al decidir cómo implementar este patrón:

Elija una memoria auxiliar que ofrezca unos niveles aceptables de rendimiento, alta disponibilidad y solidez y que se pueda copiar como parte del proceso de administración y mantenimiento de la aplicación. En una aplicación hospedada en la nube, el uso de un mecanismo de almacenamiento en la nube suele ser una buena elección para satisfacer estos requisitos.

Diseñe el esquema de la memoria auxiliar que permita flexibilidad en los tipos de información que puede contener. Asegúrese de que proporciona todos los requisitos de configuración, como datos con tipo, colecciones de valores de configuración, varias versiones de configuración y otras características que requieren las aplicaciones que lo usan. El esquema debe ser fácil de extender para admitir valores de configuración adicionales a medida que cambien los requisitos.

Tenga en cuenta las funcionalidades físicas de la memoria auxiliar, cómo se relaciona con la manera en que se almacena información de configuración y los efectos en el rendimiento. Por ejemplo, para almacenar un documento XML que contiene información de configuración, será necesario que la interfaz de configuración o la aplicación analicen el documento con el fin de leer los valores de configuración individuales. Como resultado, la actualización de un valor de configuración será más complicada, aunque el almacenamiento en caché de la configuración puede ayudar a compensar el rendimiento más lento de lectura.

Tenga en cuenta cómo la interfaz de configuración permitirá controlar el ámbito y la herencia de los valores de configuración. Por ejemplo, podría ser un requisito extender el ámbito de la configuración a la organización, la aplicación y el nivel de máquina. Podría ser necesario permitir la delegación del control sobre el acceso a diferentes ámbitos, e impedir o permitir que aplicaciones individuales invaliden la configuración.

Asegúrese de que la interfaz de configuración pueda exponer los datos de configuración en el formato necesario, como valores con tipo, colecciones, pares de claves-valor o contenedores de propiedades.

Considere como se comportará la interfaz del almacén de configuración cuando los valores de configuración contengan errores o no existan en la memoria auxiliar. Puede que sea adecuado devolver los valores de configuración predeterminados y registrar los errores. Tenga en cuenta también aspectos como la distinción entre mayúsculas y minúsculas de las claves o los nombres de los valores de configuración, el almacenamiento y la administración de datos binarios y las maneras en que se administran los valores nulos o vacíos.

Considere como proteger los datos de configuración para permitir el acceso solo a las aplicaciones y los usuarios adecuados. Aunque esta es probablemente una característica de la interfaz del almacén de configuración, también es necesaria para garantizar que no se puede tener acceso directo a los datos de la memoria auxiliar sin el permiso adecuado. Asegúrese de que exista una separación estricta entre los permisos necesarios para leer y escribir datos de configuración. Considere también si debe cifrar algunos o todos los valores de configuración y cómo se implementará esto en la interfaz del almacén de configuración.

Las configuraciones almacenadas centralmente, que cambian el comportamiento de la aplicación en tiempo de ejecución, son muy importantes y se deben implementar, actualizar y administrar con los mismos mecanismos que se usan para implementar el código de la aplicación. Por ejemplo, los cambios que puedan afectar a más de una aplicación deben llevarse a cabo mediante un enfoque de prueba completa e implementación de ensayo para garantizar que el cambio es adecuado para todas las aplicaciones que usan esta configuración. La modificación por un administrador de una opción para actualizar una aplicación, podría afectar negativamente a otras aplicaciones que usen la misma configuración.

Si una aplicación almacena en caché la información de configuración, será necesario alertarla si cambia la configuración. Se podría implementar una directiva de caducidad sobre los datos de configuración almacenados en caché para que esta información se actualice automáticamente de forma periódica y se recojan los cambios (y se actúe en función de ellos).

## <a name="when-to-use-this-pattern"></a>Cuándo usar este patrón

Este patrón es útil para:

- Valores de configuración que se comparten entre varias aplicaciones e instancias de aplicación, o donde se deba aplicar una configuración estándar entre varias aplicaciones e instancias de aplicación.

- Un sistema de configuración estándar que no admita todos los valores de configuración necesarios, como el almacenamiento de imágenes o tipos de datos complejos.

- Como almacén complementario para algunos de los valores de las aplicaciones, que permita quizás que las aplicaciones reemplacen algunos o todos los valores almacenados a nivel central.

- Como forma de simplificar la administración de varias aplicaciones y, opcionalmente, de supervisar el uso de valores de configuración mediante el registro de algunos o todos los tipos de acceso en el almacén de configuración.

## <a name="example"></a>Ejemplo

En una aplicación hospedada de Microsoft Azure, una opción típica para almacenar información de configuración externamente es usar Azure Storage. Esta solución es resistente, ofrece un alto rendimiento y se replica tres veces con conmutación automática por error para ofrecer alta disponibilidad. Azure Table Storage proporciona un almacén de clave-valor con la posibilidad de usar un esquema flexible para los valores. Azure Blob Storage proporciona un almacén jerárquico basado en contenedores que puede contener cualquier tipo de datos en blobs nombrados de forma individual.

En el ejemplo siguiente se muestra cómo se puede implementar un almacén de configuración sobre Blob Storage para almacenar y exponer la información de configuración. La clase `BlobSettingsStore` abstrae el almacenamiento de blobs para contener la información de configuración e implementa la interfaz `ISettingsStore` que se muestra en el código siguiente.

> Este código se proporciona en el proyecto _ExternalConfigurationStore.Cloud_ en la solución _ExternalConfigurationStore_, disponible en [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/external-configuration-store).

```csharp
public interface ISettingsStore
{
    Task<string> GetVersionAsync();

    Task<Dictionary<string, string>> FindAllAsync();
}
```

Esta interfaz define métodos para recuperar y actualizar los valores de configuración contenidos en el almacén de configuración e incluye un número de versión que puede usarse para detectar si los parámetros de configuración se han modificado recientemente. La clase `BlobSettingsStore` usa la propiedad `ETag` del blob para implementar el control de versiones. La propiedad `ETag` se actualiza automáticamente cada vez que se escribe el blob.

> De forma predeterminada, esta solución simple expone todos los valores de configuración como valores de cadena en lugar de valores con tipo.

La clase `ExternalConfigurationManager` proporciona un contenedor alrededor de un objeto `BlobSettingsStore`. Una aplicación puede usar esta clase para almacenar y recuperar información de configuración. Esta clase usa la biblioteca [Reactive Extensions](https://msdn.microsoft.com/library/hh242985.aspx) de Microsoft para exponer los cambios realizados en la configuración a través de una implementación de la interfaz `IObservable`. Si se modifica un valor de configuración mediante la llamada al método `SetAppSetting`, se genera el evento `Changed` y todos los suscriptores a este evento reciben una notificación.

Tenga en cuenta que toda la configuración también se almacena en caché en un objeto `Dictionary` dentro de la clase `ExternalConfigurationManager` para un fácil acceso. El método `GetSetting` usado para recuperar un valor de configuración lee los datos de la caché. Si el valor de configuración no se encuentra en la caché, se recupera en su lugar del objeto `BlobSettingsStore`.

El método `GetSettings` invoca el método `CheckForConfigurationChanges` para detectar si se ha cambiado la información de configuración en el almacenamiento de blobs. Para ello, examina el número de versión y lo compara con el número de versión actual que contiene el objeto `ExternalConfigurationManager`. Si se han producido uno o más cambios, se genera el evento `Changed` y se actualizan los valores de configuración almacenados en caché en el objeto `Dictionary`. Esta es una aplicación del [patrón Cache-Aside](./cache-aside.md).

El siguiente código de ejemplo se muestra cómo se implementa el evento `Changed` eventos, el método `GetSettings` y el método `CheckForConfigurationChanges`:

```csharp
public class ExternalConfigurationManager : IDisposable
{
  // An abstraction of the configuration store.
  private readonly ISettingsStore settings;
  private readonly ISubject<KeyValuePair<string, string>> changed;
  ...
  private readonly ReaderWriterLockSlim settingsCacheLock = new ReaderWriterLockSlim();
  private readonly SemaphoreSlim syncCacheSemaphore = new SemaphoreSlim(1);  
  ...
  private Dictionary<string, string> settingsCache;
  private string currentVersion;
  ...
  public ExternalConfigurationManager(ISettingsStore settings, ...)
  {
    this.settings = settings;
    ...
  }
  ...
  public IObservable<KeyValuePair<string, string>> Changed => this.changed.AsObservable();
  ...

  public string GetAppSetting(string key)
  {
    ...
    // Try to get the value from the settings cache.
    // If there's a cache miss, get the setting from the settings store and refresh the settings cache.

    string value;
    try
    {
        this.settingsCacheLock.EnterReadLock();

        this.settingsCache.TryGetValue(key, out value);
    }
    finally
    {
        this.settingsCacheLock.ExitReadLock();
    }

    return value;
  }
  ...
  private void CheckForConfigurationChanges()
  {
    try
    {
        // It is assumed that updates are infrequent.
        // To avoid race conditions in refreshing the cache, synchronize access to the in-memory cache.
        await this.syncCacheSemaphore.WaitAsync();

        var latestVersion = await this.settings.GetVersionAsync();

        // If the versions are the same, nothing has changed in the configuration.
        if (this.currentVersion == latestVersion) return;

        // Get the latest settings from the settings store and publish changes.
        var latestSettings = await this.settings.FindAllAsync();

        // Refresh the settings cache.
        try
        {
            this.settingsCacheLock.EnterWriteLock();

            if (this.settingsCache != null)
            {
                //Notify settings changed
                latestSettings.Except(this.settingsCache).ToList().ForEach(kv => this.changed.OnNext(kv));
            }
            this.settingsCache = latestSettings;
        }
        finally
        {
            this.settingsCacheLock.ExitWriteLock();
        }

        // Update the current version.
        this.currentVersion = latestVersion;
    }
    catch (Exception ex)
    {
        this.changed.OnError(ex);
    }
    finally
    {
        this.syncCacheSemaphore.Release();
    }
  }
}
```

> La clase `ExternalConfigurationManager` también proporciona una propiedad denominada `Environment`. Esta propiedad admite distintas configuraciones para una aplicación que se ejecuta en entornos diferentes, como ensayo y producción.

Un objeto `ExternalConfigurationManager` también puede consultar el objeto `BlobSettingsStore` periódicamente para comprobar los posibles cambios. En el código siguiente, el método `StartMonitor` llama a `CheckForConfigurationChanges` en un intervalo para detectar los cambios y generar el evento `Changed`, como se describió anteriormente.

```csharp
public class ExternalConfigurationManager : IDisposable
{
  ...
  private readonly ISubject<KeyValuePair<string, string>> changed;
  private Dictionary<string, string> settingsCache;
  private readonly CancellationTokenSource cts = new CancellationTokenSource();
  private Task monitoringTask;
  private readonly TimeSpan interval;

  private readonly SemaphoreSlim timerSemaphore = new SemaphoreSlim(1);
  ...
  public ExternalConfigurationManager(string environment) : this(new BlobSettingsStore(environment), TimeSpan.FromSeconds(15), environment)
  {
  }
  
  public ExternalConfigurationManager(ISettingsStore settings, TimeSpan interval, string environment)
  {
      this.settings = settings;
      this.interval = interval;
      this.CheckForConfigurationChangesAsync().Wait();
      this.changed = new Subject<KeyValuePair<string, string>>();
      this.Environment = environment;
  }
  ...
  /// <summary>
  /// Check to see if the current instance is monitoring for changes
  /// </summary>
  public bool IsMonitoring => this.monitoringTask != null && !this.monitoringTask.IsCompleted;

  /// <summary>
  /// Start the background monitoring for configuration changes in the central store
  /// </summary>
  public void StartMonitor()
  {
      if (this.IsMonitoring)
          return;

      try
      {
          this.timerSemaphore.Wait();

          // Check again to make sure we are not already running.
          if (this.IsMonitoring)
              return;

          // Start running our task loop.
          this.monitoringTask = ConfigChangeMonitor();
      }
      finally
      {
          this.timerSemaphore.Release();
      }
  }

  /// <summary>
  /// Loop that monitors for configuration changes
  /// </summary>
  /// <returns></returns>
  public async Task ConfigChangeMonitor()
  {
      while (!cts.Token.IsCancellationRequested)
      {
          await this.CheckForConfigurationChangesAsync();
          await Task.Delay(this.interval, cts.Token);
      }
  }

  /// <summary>
  /// Stop monitoring for configuration changes
  /// </summary>
  public void StopMonitor()
  {
      try
      {
          this.timerSemaphore.Wait();

          // Signal the task to stop.
          this.cts.Cancel();

          // Wait for the loop to stop.
          this.monitoringTask.Wait();

          this.monitoringTask = null;
      }
      finally
      {
          this.timerSemaphore.Release();
      }
  }

  public void Dispose()
  {
      this.cts.Cancel();
  }
  ...
}
```

Se crea una instancia de la clase `ExternalConfigurationManager` como una instancia de singleton mediante la clase `ExternalConfiguration` mostrada a continuación.

```csharp
public static class ExternalConfiguration
{
    private static readonly Lazy<ExternalConfigurationManager> configuredInstance = new Lazy<ExternalConfigurationManager>(
        () =>
        {
            var environment = CloudConfigurationManager.GetSetting("environment");
            return new ExternalConfigurationManager(environment);
        });

    public static ExternalConfigurationManager Instance => configuredInstance.Value;
}
```

El código siguiente se toma de la clase `WorkerRole` en el proyecto _ExternalConfigurationStore.Cloud_. Muestra cómo la aplicación usa la clase `ExternalConfiguration` para leer un valor de configuración.

```csharp
public override void Run()
{
  // Start monitoring configuration changes.
  ExternalConfiguration.Instance.StartMonitor();

  // Get a setting.
  var setting = ExternalConfiguration.Instance.GetAppSetting("setting1");
  Trace.TraceInformation("Worker Role: Get setting1, value: " + setting);

  this.completeEvent.WaitOne();
}
```

El siguiente código, también de la clase `WorkerRole`, muestra cómo la aplicación se suscribe a eventos de configuración.

```csharp
public override bool OnStart()
{
  ...
  // Subscribe to the event.
  ExternalConfiguration.Instance.Changed.Subscribe(
     m => Trace.TraceInformation("Configuration has changed. Key:{0} Value:{1}",
          m.Key, m.Value),
     ex => Trace.TraceError("Error detected: " + ex.Message));
  ...
}
```

## <a name="related-patterns-and-guidance"></a>Orientación y patrones relacionados

- Se encuentra disponible un ejemplo que demuestra este patrón en [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/external-configuration-store).
