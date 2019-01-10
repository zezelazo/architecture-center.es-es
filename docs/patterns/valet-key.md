---
title: Patrón Valet Key
titleSuffix: Cloud Design Patterns
description: Usa un token o clave que proporciona a los clientes acceso directo restringido a un recurso o servicio específico.
keywords: Patrón de diseño
author: dragon119
ms.date: 06/23/2017
ms.custom: seodec18
ms.openlocfilehash: 09173717d499d524d4d5dad2c1202c1bf361b1e5
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/04/2019
ms.locfileid: "54009872"
---
# <a name="valet-key-pattern"></a>Patrón Valet Key

[!INCLUDE [header](../_includes/header.md)]

Usa un token que proporciona a los clientes acceso directo restringido a un recurso específico, con el fin de descargar la transferencia de datos desde la aplicación. Esto es especialmente útil en aplicaciones que usan sistemas o colas de almacenamiento hospedado en la nube, ya que puede minimizar los costes y maximizar la escalabilidad y el rendimiento.

## <a name="context-and-problem"></a>Contexto y problema

Los programas cliente y los exploradores web necesitan a menudo leer y escribir flujos de datos a y desde el almacenamiento de una aplicación. Normalmente, la aplicación administra el movimiento de los datos &mdash; o bien los captura del almacenamiento y los transmite al cliente o lee el flujo cargado del cliente y lo almacena en el almacén de datos. Sin embargo, este enfoque consume recursos valiosos, como proceso, memoria y ancho de banda.

Los almacenes de datos disponen de la posibilidad de cargar y descargar los datos directamente, sin necesidad de que la aplicación realice ningún procesamiento para mover estos datos. Sin embargo, para lograr esto es necesario normalmente que el cliente tenga acceso a las credenciales de seguridad del almacén. Esta técnica puede ser útil para reducir los costes de transferencia de datos y el requisito de escalado horizontal de la aplicación, y para aumentar el rendimiento. Sin embargo, significa que la aplicación ya no es capaz de administrar la seguridad de los datos. Una vez que el cliente tiene una conexión al almacén de datos para el acceso directo, la aplicación no puede actuar como equipo selector. Ya no tiene el control del proceso y no puede impedir las sucesivas cargas o descargas del almacén de datos.

Este no es un enfoque realista en sistemas distribuidos que deben atender a clientes en los que no se confía. Por el contrario, las aplicaciones deben poder controlar con seguridad el acceso a los datos de forma granular, y aun así reducir la carga en el servidor, para lo cual configuran esta conexión y luego permiten que el cliente se comunique directamente con el almacén de datos para realizar las operaciones de lectura y escritura necesarias.

## <a name="solution"></a>Solución

Debe resolver el problema de controlar el acceso a un almacén de datos cuando el almacén no pueda administrar la autenticación y la autorización de los clientes. Una solución típica consiste en restringir el acceso a la conexión pública del almacén de datos y proporcionar al cliente una clave o un token que pueda validar dicho almacén.

Esta clave o token se conoce normalmente como clave auxiliar. Esta clave proporciona acceso limitado a recursos específicos y permite únicamente operaciones predefinidas, como lectura y escritura en almacenamiento o colas, o carga y descarga en un explorador web. Las aplicaciones pueden crear y emitir claves auxiliares a dispositivos cliente y exploradores web de manera rápida y fácil, lo que permite que los clientes realicen las operaciones necesarias sin necesidad de que la aplicación administre directamente la transferencia de datos. De esta manera, se elimina la sobrecarga de procesamiento, y el impacto sobre el rendimiento y la escalabilidad, de la aplicación y el servidor.

El cliente usa este token para acceder a un recurso concreto en el almacén de datos durante un período de tiempo específico, y con restricciones específicas sobre los permisos de acceso, como se muestra en la ilustración. Tras el período especificado, la clave deja de ser válida y no permite el acceso al recurso.

![Figura 1: Información general del patrón](./_images/valet-key-pattern.png)

También es posible configurar una clave que tenga otras dependencias, como el ámbito de los datos. Por ejemplo, según las capacidades del almacén de datos, la clave puede especificar una tabla completa de un almacén de datos o solo filas específicas de una tabla. En sistemas de almacenamiento de nube, la clave puede especificar un contenedor o simplemente un elemento específico dentro de un contenedor.

La clave también se puede invalidar mediante la aplicación. Este enfoque es útil si el cliente notifica al servidor que la operación de transferencia de datos ha finalizado. Luego, el servidor puede invalidar dicha clave para impedir que se vuelva a acceder.

Mediante este patrón, puede simplificar la administración del acceso a los recursos porque no hay ningún requisito para crear y autenticar a un usuario, conceder permisos y luego quitar de nuevo el usuario. También permite limitar de forma fácil la ubicación, el permiso y el período de validez&mdash;todo ello mediante la simple generación de una clave en tiempo de ejecución. Los factores importantes son limitar el período de validez, y especialmente la ubicación del recurso, lo más estrictamente posible para que el destinatario solo pueda usarse para el fin que se ha previsto.

## <a name="issues-and-considerations"></a>Problemas y consideraciones

Tenga en cuenta los puntos siguientes al decidir cómo implementar este patrón:

**Administrar el estado de validez y el período de la clave**. Si se ha filtrado o puesto en peligro, la clave desbloquea de manera efectiva el elemento de destino y permite que esté disponible para su uso malintencionado durante el período de validez. Una clave normalmente se puede revocar o deshabilitar, dependiendo de cómo se emitiera. Las directivas del lado servidor se pueden cambiar, o la clave de servidor con la que se han firmado se puede invalidar. Especifique un período de validez corto para minimizar el riesgo de permitir que tengan lugar operaciones no autorizadas contra el almacén de datos. Sin embargo, si el período de validez es demasiado corto, el cliente no podrá completar la operación antes de que expire la clave. Permita que los usuarios autorizados renueven la clave antes de que expire el período de validez si se requieren varios accesos al recurso protegido.

**Controlar el nivel de acceso que proporcionará la clave**. Normalmente, la clave debe permitir al usuario realizar únicamente las acciones necesarias para completar la operación, como el acceso de solo lectura si el cliente no debería poder cargar los datos en el almacén de datos. Con las cargas de archivos, es habitual especificar una clave que proporcione permisos de solo escritura, así como la ubicación y el período de validez. Es fundamental especificar con precisión el recurso o el conjunto de recursos a los que se aplica la clave.

**Considerar cómo controlar el comportamiento de los usuarios**. Implementar este patrón significa cierta pérdida de control sobre los recursos a los que se concede acceso a los usuarios. El nivel de control que se puede ejercer está limitado por las funcionalidades de las directivas y los permisos disponibles para el servicio o el almacén de datos de destino. Por ejemplo, lo habitual es que no sea posible crear una clave que limite el tamaño de los datos que se van a escribir en el almacenamiento, o el número de veces que se puede usar la clave para acceder a un archivo. Esta situación puede dar lugar a que aumenten los costes de transferencia de datos de forma inesperada, incluso cuando esta clave la usa el cliente previsto, y podría deberse a un error en el código que provoca cargas y descargas repetidas. Para limitar el número de veces que se puede cargar un archivo, siempre que sea posible, obligue al cliente a que notifique a la aplicación cuando una operación finalice. Por ejemplo, algunos almacenes de datos generan eventos que el código de aplicación puede usar para supervisar las operaciones y controlar el comportamiento del usuario. Sin embargo, resulta difícil exigir cuotas para usuarios individuales en un escenario multiinquilino donde todos los usuarios de un mismo inquilino usan la misma clave.

**Validar y, opcionalmente, corregir, todos los datos cargados**. Un usuario malintencionado que consiga acceso a la clave podría cargar datos diseñados para poner en peligro el sistema. También, los usuarios autorizados podrían cargar datos que no son válidos que, cuando se procesan, dan lugar a un error o un error del sistema. Para evitar este problema, asegúrese de que todos los datos cargados se validen y se comprueben en busca de contenido malintencionado antes de su uso.

**Auditar todas las operaciones**. Numerosos mecanismos basados en claves pueden registrar operaciones como cargas, descargas y errores. Estos registros se pueden incorporar normalmente a un proceso de auditoría y también se usan para facturación si el usuario paga según el volumen de datos o el tamaño de los archivos. Use los registros para detectar errores de autenticación que podrían deberse a problemas con el proveedor de claves o a la eliminación accidental de una directiva de acceso almacenada.

**Entregar la clave de forma segura**. Se puede insertar en una dirección URL que el usuario activa en una página web, o se puede usar en una operación de redireccionamiento del servidor para que la descarga se produzca automáticamente. Use siempre HTTPS para entregar la clave por un canal seguro.

**Proteger los datos confidenciales en tránsito**. La entrega de los datos confidenciales a través de la aplicación tiene lugar normalmente mediante SSL o TLS, y es así como debe ser para los clientes que acceden al almacén de datos directamente.

Otros aspectos que se deben tener en cuenta al implementar este patrón son:

- Si el cliente no notifica, o no puede notificar, al servidor el fin de la operación y el único límite es el período de expiración de la clave, la aplicación no podrá realizar operaciones de auditoría, como contar el número de cargas o descargas o impedir varias cargas o descargas.

- La flexibilidad de las directivas de clave que se pueden generar podría ser limitada. Por ejemplo, algunos mecanismos solo permiten el uso de un período de expiración con un tiempo fijado. Otros usuarios no pueden especificar una granularidad suficiente de permisos de lectura y escritura.

- Si se especifica la hora de inicio del período de validez del token o de la clave, asegúrese de que sea un poco anterior a la hora actual del servidor para permitir que se sincronicen los relojes del cliente que podrían estar ligeramente fuera de sincronización. Si no se especifica, el valor predeterminado es normalmente la hora actual del servidor.

- La dirección URL que contiene la clave se registrará en los archivos de registro del servidor. Si bien la clave habrá expirado normalmente antes de que los archivos de registro se utilicen para el análisis, asegúrese de limitar el acceso a ellos. Si los datos de registro se trasmiten a un sistema de supervisión o se almacenan en otra ubicación, considere la posibilidad de implementar un retraso para impedir la fuga de claves hasta después de que haya expirado el periodo de validez.

- Si el código de cliente se ejecuta en un explorador web, es posible que el explorador tenga que ser compatible con el uso compartido de recursos entre orígenes (CORS) para permitir que el código que se ejecuta en él acceda a los datos de un dominio diferente a aquel que entregó la página. Algunos exploradores más antiguos y algunos almacenes de datos no son compatibles con CORS, y el código que se ejecuta en estos exploradores podría usar una clave auxiliar para proporcionar acceso a los datos de un dominio diferente, como una cuenta de almacenamiento en la nube.

## <a name="when-to-use-this-pattern"></a>Cuándo usar este patrón

Este patrón es útil en las situaciones siguientes:

- Para minimizar la carga de recursos y maximizar el rendimiento y la escalabilidad. Con una clave auxiliar, no es necesario bloquear el recurso, no se requiere ninguna llamada al servidor remoto, no hay ningún límite en el número de claves auxiliares que se pueden emitir y se evita que exista un único punto de error derivado de realizar la transferencia de datos a través del código de aplicación. La creación de una clave auxiliar es normalmente una operación criptográfica sencilla que consiste en firmar una cadena con una clave.

- Para minimizar el coste operativo. Habilitar el acceso directo a los almacenes y las colas resulta rentable y permite hacer un uso eficiente de los recursos, puede dar lugar a menos recorridos de ida y vuelta en la red y podría permitir una reducción en el número de recursos de proceso necesarios.

- Cuando los clientes cargan o descargan datos de forma periódica, en especial en caso de grandes volúmenes o cuando en cada operación intervienen archivos de gran tamaño.

- Cuando la aplicación tiene recursos de proceso limitados disponibles, debido a las limitaciones de hospedaje o a aspectos relacionados con el coste. En este escenario, el patrón es incluso más útil si hay muchas cargas o descargas de datos simultáneas, ya que libera a la aplicación de tener que administrar la transferencia de datos.

- Cuando los datos se almacenan en un almacén de datos remoto o en otro centro de datos. Si uno de los requisitos fuera que la aplicación funcionara como equipo selector, podría haber un cargo por el uso de ancho de banda adicional que supone transferir los datos entre los centros de datos o, en redes públicas y privadas, entre el cliente y la aplicación y luego entre la aplicación y el almacén de datos.

Este patrón podría no ser útil en las siguientes situaciones:

- Si la aplicación debe realizar alguna tarea en los datos antes de que se almacenen o antes de enviarlos al cliente. Por ejemplo, si la aplicación necesita realizar la validación, registrar el éxito de acceso o ejecutar una transformación en los datos. Sin embargo, algunos almacenes de datos y clientes pueden negociar y llevar a cabo transformaciones sencillas, como la compresión y descompresión (por ejemplo, un explorador web normalmente puede administrar formatos GZip).

- Si el diseño de una aplicación existente dificulta incorporar el patrón. El uso de este patrón normalmente requiere un enfoque de arquitectura diferente en la entrega y recepción de los datos.

- Si es necesario mantener registros de auditoría o controlar la cantidad de veces que se ejecuta una operación de transferencia de datos, y el mecanismo de clave auxiliar en uso no admite notificaciones que el servidor pueda usar para administrar estas operaciones.

- Si es necesario limitar el tamaño de los datos, especialmente durante las operaciones de carga. La única solución a ello es que la aplicación compruebe el tamaño de los datos después de que la operación finaliza, o comprobar el tamaño de las cargas tras un período especificado o según una programación.

## <a name="example"></a>Ejemplo

Azure admite firmas de acceso compartido en Azure Storage que permiten un mejor control del acceso a los datos de blobs, tablas y colas, y de colas y temas de Service Bus. Un token de firma de acceso compartido se puede configurar para proporcionar derechos de acceso específicos, como lectura, escritura, actualización y eliminación para una tabla específica; un intervalo de claves dentro de una tabla; una cola; un blob; o un contenedor de blobs. La validez puede ser un período de tiempo especificado o sin límite de tiempo.

Las firmas de acceso compartido de Azure también admiten directivas de acceso almacenadas por el servidor que se pueden asociar a un recurso específico, como una tabla o un blob. Esta característica proporciona mayor control y flexibilidad en comparación con los tokens de firma de acceso compartido generados por la aplicación, y debe utilizarse siempre que sea posible. La configuración definida en una directiva almacenada por el servidor se puede cambiar y se refleja en el token sin necesidad de emitir uno nuevo, pero la configuración definida en el token no se puede cambiar sin emitir uno nuevo. Este enfoque también permite revocar un token de firma de acceso compartido válido antes de que haya expirado.

> Para más información, consulte [Introducing Table SAS (Shared Access Signature), Queue SAS and update to Blob SAS](https://blogs.msdn.microsoft.com/windowsazurestorage/2012/06/12/introducing-table-sas-shared-access-signature-queue-sas-and-update-to-blob-sas/) (Introducción a SAS [firma de acceso compartido] de Table, SAS de Queue y actualización a SAS de Blob) y [Uso de firmas de acceso compartido](/azure/storage/common/storage-dotnet-shared-access-signature-part-1) en MSDN.

El código siguiente muestra cómo crear un token de firma de acceso compartido que sea válido durante cinco minutos. El método `GetSharedAccessReferenceForUpload` devuelve un token de firmas de acceso compartido que puede usarse para cargar un archivo en Azure Blob Storage.

```csharp
public class ValuesController : ApiController
{
  private readonly CloudStorageAccount account;
  private readonly string blobContainer;
  ...
  /// <summary>
  /// Return a limited access key that allows the caller to upload a file
  /// to this specific destination for a defined period of time.
  /// </summary>
  private StorageEntitySas GetSharedAccessReferenceForUpload(string blobName)
  {
    var blobClient = this.account.CreateCloudBlobClient();
    var container = blobClient.GetContainerReference(this.blobContainer);

    var blob = container.GetBlockBlobReference(blobName);

    var policy = new SharedAccessBlobPolicy
    {
      Permissions = SharedAccessBlobPermissions.Write,

      // Specify a start time five minutes earlier to allow for client clock skew.
      SharedAccessStartTime = DateTime.UtcNow.AddMinutes(-5),

      // Specify a validity period of five minutes starting from now.
      SharedAccessExpiryTime = DateTime.UtcNow.AddMinutes(5)
    };

    // Create the signature.
    var sas = blob.GetSharedAccessSignature(policy);

    return new StorageEntitySas
    {
      BlobUri = blob.Uri,
      Credentials = sas,
      Name = blobName
    };
  }

  public struct StorageEntitySas
  {
    public string Credentials;
    public Uri BlobUri;
    public string Name;
  }
}
```

> El ejemplo completo está disponible en la solución de ValetKey, que se puede descargar en [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/valet-key). El proyecto ValetKey.Web de esta solución contiene una aplicación web que incluye la clase `ValuesController` mostrada anteriormente. Hay una aplicación cliente de ejemplo que usa esta aplicación web para recuperar una clave de firmas de acceso compartido y cargar un archivo en el almacenamiento de blobs, que se puede encontrar en el proyecto ValetKey.Client.

## <a name="next-steps"></a>Pasos siguientes

Los patrones y las directrices siguientes también pueden ser importantes a la hora de implementar este modelo:

- Se encuentra disponible un ejemplo que demuestra este patrón en [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/valet-key).
- [Patrón Gatekeeper](./gatekeeper.md). Este patrón se puede usar junto con el patrón Valet Key para proteger las aplicaciones y los servicios mediante una instancia de host dedicada que actúa como intermediario entre los clientes y la aplicación o el servicio. El equipo selector valida y corrige las solicitudes y pasa las solicitudes y los datos entre el cliente y la aplicación. Esto puede proporcionar una capa de seguridad adicional y reducir la superficie expuesta a ataques del sistema.
- [Patrón Static Content Hosting](./static-content-hosting.md). Describe cómo implementar recursos estáticos en un servicio de almacenamiento basado en la nube que puede proporcionar estos recursos directamente al cliente para reducir la necesidad de instancias de proceso costosas. Cuando la finalidad de los recursos no es que estén públicamente disponibles, el patrón Vale Key se puede usar para protegerlos.
- [Introducing Table SAS (Shared Access Signature), Queue SAS and update to Blob SAS](https://blogs.msdn.microsoft.com/windowsazurestorage/2012/06/12/introducing-table-sas-shared-access-signature-queue-sas-and-update-to-blob-sas/) (Introducción a SAS [firma de acceso compartido] de Table, SAS de Queue y actualización a SAS de Blob)
- [Uso de firmas de acceso compartido](/azure/storage/common/storage-dotnet-shared-access-signature-part-1)
- [Autenticación con firma de acceso compartido en Service Bus](/azure/service-bus-messaging/service-bus-sas)
