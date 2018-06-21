---
title: External Configuration Store
description: Extrae la información de configuración del paquete de implementación de la aplicación y la lleva a una ubicación centralizada.
keywords: Patrón de diseño
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- design-implementation
- management-monitoring
ms.openlocfilehash: 733ca979903d1526d3a1a6b281a8903893e19fda
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2017
ms.locfileid: "24542287"
---
# <a name="external-configuration-store-pattern"></a><span data-ttu-id="53426-104">Patrón External Configuration Store</span><span class="sxs-lookup"><span data-stu-id="53426-104">External Configuration Store pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="53426-105">Extrae la información de configuración del paquete de implementación de la aplicación y la lleva a una ubicación centralizada.</span><span class="sxs-lookup"><span data-stu-id="53426-105">Move configuration information out of the application deployment package to a centralized location.</span></span> <span data-ttu-id="53426-106">Este patrón puede proporcionar oportunidades para facilitar la administración y el control de los datos de configuración y para compartir los datos de configuración entre aplicaciones e instancias de aplicación.</span><span class="sxs-lookup"><span data-stu-id="53426-106">This can provide opportunities for easier management and control of configuration data, and for sharing configuration data across applications and application instances.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="53426-107">Contexto y problema</span><span class="sxs-lookup"><span data-stu-id="53426-107">Context and problem</span></span>

<span data-ttu-id="53426-108">La mayoría de los entornos de tiempo de ejecución de aplicaciones incluyen información de configuración que está contenida en los archivos implementados con la aplicación.</span><span class="sxs-lookup"><span data-stu-id="53426-108">The majority of application runtime environments include configuration information that's held in files deployed with the application.</span></span> <span data-ttu-id="53426-109">En algunos casos, es posible editar estos archivos para cambiar el comportamiento de la aplicación después de que se ha implementado.</span><span class="sxs-lookup"><span data-stu-id="53426-109">In some cases, it's possible to edit these files to change the application behavior after it's been deployed.</span></span> <span data-ttu-id="53426-110">Sin embargo, los cambios en la configuración requieren volver a implementar la aplicación, lo que suele dar lugar a un tiempo de inactividad inaceptable y más sobrecarga administrativa.</span><span class="sxs-lookup"><span data-stu-id="53426-110">However, changes to the configuration require the application be redeployed, often resulting in unacceptable downtime and other administrative overhead.</span></span>

<span data-ttu-id="53426-111">Los archivos de configuración local también limitan la configuración a una única aplicación, pero en ocasiones sería útil compartir los valores de configuración entre varias aplicaciones.</span><span class="sxs-lookup"><span data-stu-id="53426-111">Local configuration files also limit the configuration to a single application, but sometimes it would be useful to share configuration settings across multiple applications.</span></span> <span data-ttu-id="53426-112">Algunos ejemplos incluyen cadenas de conexión de base de datos, información de temas de interfaz de usuario o las direcciones URL de las colas y el almacenamiento que e usan en un conjunto relacionado de aplicaciones.</span><span class="sxs-lookup"><span data-stu-id="53426-112">Examples include database connection strings, UI theme information, or the URLs of queues and storage used by a related set of applications.</span></span>

<span data-ttu-id="53426-113">Administrar los cambios en las configuraciones locales entre varias instancias en ejecución de la aplicación puede resultar difícil, en especial en escenarios hospedados en la nube.</span><span class="sxs-lookup"><span data-stu-id="53426-113">It's challenging to manage changes to local configurations across multiple running instances of the application, especially in a cloud-hosted scenario.</span></span> <span data-ttu-id="53426-114">Puede dar lugar a instancias que usan distintos valores de configuración mientras se implementa la actualización.</span><span class="sxs-lookup"><span data-stu-id="53426-114">It can result in instances using different configuration settings while the update is being deployed.</span></span>

<span data-ttu-id="53426-115">Además, las actualizaciones de aplicaciones y componentes pueden requerir cambios en los esquemas de configuración.</span><span class="sxs-lookup"><span data-stu-id="53426-115">In addition, updates to applications and components might require changes to configuration schemas.</span></span> <span data-ttu-id="53426-116">Muchos sistemas de configuración no admiten diferentes versiones de la información de configuración.</span><span class="sxs-lookup"><span data-stu-id="53426-116">Many configuration systems don't support different versions of configuration information.</span></span>

## <a name="solution"></a><span data-ttu-id="53426-117">Solución</span><span class="sxs-lookup"><span data-stu-id="53426-117">Solution</span></span>

<span data-ttu-id="53426-118">Almacene la información de configuración en almacenamiento externo y proporcione una interfaz que puede usarse para leer y actualizar de manera rápida y eficaz los valores de configuración.</span><span class="sxs-lookup"><span data-stu-id="53426-118">Store the configuration information in external storage, and provide an interface that can be used to quickly and efficiently read and update configuration settings.</span></span> <span data-ttu-id="53426-119">El tipo de almacén externo depende del entorno de hospedaje y de tiempo de ejecución de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="53426-119">The type of external store depends on the hosting and runtime environment of the application.</span></span> <span data-ttu-id="53426-120">En un escenario hospedado en la nube, suele ser un servicio de almacenamiento basado en la nube, pero podría ser una base de datos hospedada u otro sistema.</span><span class="sxs-lookup"><span data-stu-id="53426-120">In a cloud-hosted scenario it's typically a cloud-based storage service, but could be a hosted database or other system.</span></span>

<span data-ttu-id="53426-121">La memoria auxiliar que elija para la información de configuración debe tener una interfaz que proporcione acceso coherente y fácil de usar.</span><span class="sxs-lookup"><span data-stu-id="53426-121">The backing store you choose for configuration information should have an interface that provides consistent and easy-to-use access.</span></span> <span data-ttu-id="53426-122">Debe exponer la información en un formato estructurado y correctamente escrito.</span><span class="sxs-lookup"><span data-stu-id="53426-122">It should expose the information in a correctly typed and structured format.</span></span> <span data-ttu-id="53426-123">Puede que también la implementación deba autorizar el acceso a los usuarios con el fin de proteger los datos de configuración y ser lo suficientemente flexible como para permitir el almacenamiento de varias versiones de la configuración (por ejemplo, desarrollo, ensayo o producción, incluidas varias versiones de cada una).</span><span class="sxs-lookup"><span data-stu-id="53426-123">The implementation might also need to authorize users’ access in order to protect configuration data, and be flexible enough to allow storage of multiple versions of the configuration (such as development, staging, or production, including multiple release versions of each one).</span></span>

> <span data-ttu-id="53426-124">Muchos sistemas de configuración integrados leen los datos cuando se inicia la aplicación y los almacenan en caché en memoria para proporcionar acceso rápido y minimizar el impacto en el rendimiento de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="53426-124">Many built-in configuration systems read the data when the application starts up, and cache the data in memory to provide fast access and minimize the impact on application performance.</span></span> <span data-ttu-id="53426-125">Según el tipo de memoria auxiliar usado y la latencia de este almacén, podría resultar útil implementar un mecanismo de almacenamiento en caché en el almacén de configuración externo.</span><span class="sxs-lookup"><span data-stu-id="53426-125">Depending on the type of backing store used, and the latency of this store, it might be helpful to implement a caching mechanism within the external configuration store.</span></span> <span data-ttu-id="53426-126">Para obtener más información, consulte [Caching Guidance](https://msdn.microsoft.com/library/dn589802.aspx)(Guía de almacenamiento en caché).</span><span class="sxs-lookup"><span data-stu-id="53426-126">For more information, see the [Caching Guidance](https://msdn.microsoft.com/library/dn589802.aspx).</span></span> <span data-ttu-id="53426-127">La ilustración muestra información general sobre el patrón de almacén de configuración externo con caché local opcional.</span><span class="sxs-lookup"><span data-stu-id="53426-127">The figure illustrates an overview of the External Configuration Store pattern with optional local cache.</span></span>

![Información general sobre el patrón External Configuration Store con caché local opcional](./_images/external-configuration-store-overview.png)


## <a name="issues-and-considerations"></a><span data-ttu-id="53426-129">Problemas y consideraciones</span><span class="sxs-lookup"><span data-stu-id="53426-129">Issues and considerations</span></span>

<span data-ttu-id="53426-130">Tenga en cuenta los puntos siguientes al decidir cómo implementar este patrón:</span><span class="sxs-lookup"><span data-stu-id="53426-130">Consider the following points when deciding how to implement this pattern:</span></span>

<span data-ttu-id="53426-131">Elija una memoria auxiliar que ofrezca unos niveles aceptables de rendimiento, alta disponibilidad y solidez y que se pueda copiar como parte del proceso de administración y mantenimiento de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="53426-131">Choose a backing store that offers acceptable performance, high availability, robustness, and can be backed up as part of the application maintenance and administration process.</span></span> <span data-ttu-id="53426-132">En una aplicación hospedada en la nube, el uso de un mecanismo de almacenamiento en la nube suele ser una buena elección para satisfacer estos requisitos.</span><span class="sxs-lookup"><span data-stu-id="53426-132">In a cloud-hosted application, using a cloud storage mechanism is usually a good choice to meet these requirements.</span></span>

<span data-ttu-id="53426-133">Diseñe el esquema de la memoria auxiliar que permita flexibilidad en los tipos de información que puede contener.</span><span class="sxs-lookup"><span data-stu-id="53426-133">Design the schema of the backing store to allow flexibility in the types of information it can hold.</span></span> <span data-ttu-id="53426-134">Asegúrese de que proporciona todos los requisitos de configuración, como datos con tipo, colecciones de valores de configuración, varias versiones de configuración y otras características que requieren las aplicaciones que lo usan.</span><span class="sxs-lookup"><span data-stu-id="53426-134">Ensure that it provides for all configuration requirements such as typed data, collections of settings, multiple versions of settings, and any other features that the applications using it require.</span></span> <span data-ttu-id="53426-135">El esquema debe ser fácil de extender para admitir valores de configuración adicionales a medida que cambien los requisitos.</span><span class="sxs-lookup"><span data-stu-id="53426-135">The schema should be easy to extend to support additional settings as requirements change.</span></span>

<span data-ttu-id="53426-136">Tenga en cuenta las funcionalidades físicas de la memoria auxiliar, cómo se relaciona con la manera en que se almacena información de configuración y los efectos en el rendimiento.</span><span class="sxs-lookup"><span data-stu-id="53426-136">Consider the physical capabilities of the backing store, how it relates to the way configuration information is stored, and the effects on performance.</span></span> <span data-ttu-id="53426-137">Por ejemplo, para almacenar un documento XML que contiene información de configuración, será necesario que la interfaz de configuración o la aplicación analicen el documento con el fin de leer los valores de configuración individuales.</span><span class="sxs-lookup"><span data-stu-id="53426-137">For example, storing an XML document containing configuration information will require either the configuration interface or the application to parse the document in order to read individual settings.</span></span> <span data-ttu-id="53426-138">Como resultado, la actualización de un valor de configuración será más complicada, aunque el almacenamiento en caché de la configuración puede ayudar a compensar el rendimiento más lento de lectura.</span><span class="sxs-lookup"><span data-stu-id="53426-138">It'll make updating a setting more complicated, though caching the settings can help to offset slower read performance.</span></span>

<span data-ttu-id="53426-139">Tenga en cuenta cómo la interfaz de configuración permitirá controlar el ámbito y la herencia de los valores de configuración.</span><span class="sxs-lookup"><span data-stu-id="53426-139">Consider how the configuration interface will permit control of the scope and inheritance of configuration settings.</span></span> <span data-ttu-id="53426-140">Por ejemplo, podría ser un requisito extender el ámbito de la configuración a la organización, la aplicación y el nivel de máquina.</span><span class="sxs-lookup"><span data-stu-id="53426-140">For example, it might be a requirement to scope configuration settings at the organization, application, and the machine level.</span></span> <span data-ttu-id="53426-141">Podría ser necesario permitir la delegación del control sobre el acceso a diferentes ámbitos, e impedir o permitir que aplicaciones individuales invaliden la configuración.</span><span class="sxs-lookup"><span data-stu-id="53426-141">It might need to support delegation of control over access to different scopes, and to prevent or allow individual applications to override settings.</span></span>

<span data-ttu-id="53426-142">Asegúrese de que la interfaz de configuración pueda exponer los datos de configuración en el formato necesario, como valores con tipo, colecciones, pares de claves-valor o contenedores de propiedades.</span><span class="sxs-lookup"><span data-stu-id="53426-142">Ensure that the configuration interface can expose the configuration data in the required formats such as typed values, collections, key/value pairs, or property bags.</span></span>

<span data-ttu-id="53426-143">Considere como se comportará la interfaz del almacén de configuración cuando los valores de configuración contengan errores o no existan en la memoria auxiliar.</span><span class="sxs-lookup"><span data-stu-id="53426-143">Consider how the configuration store interface will behave when settings contain errors, or don't exist in the backing store.</span></span> <span data-ttu-id="53426-144">Puede que sea adecuado devolver los valores de configuración predeterminados y registrar los errores.</span><span class="sxs-lookup"><span data-stu-id="53426-144">It might be appropriate to return default settings and log errors.</span></span> <span data-ttu-id="53426-145">Tenga en cuenta también aspectos como la distinción entre mayúsculas y minúsculas de las claves o los nombres de los valores de configuración, el almacenamiento y la administración de datos binarios y las maneras en que se administran los valores nulos o vacíos.</span><span class="sxs-lookup"><span data-stu-id="53426-145">Also consider aspects such as the case sensitivity of configuration setting keys or names, the storage and handling of binary data, and the ways that null or empty values are handled.</span></span>

<span data-ttu-id="53426-146">Considere como proteger los datos de configuración para permitir el acceso solo a las aplicaciones y los usuarios adecuados.</span><span class="sxs-lookup"><span data-stu-id="53426-146">Consider how to protect the configuration data to allow access to only the appropriate users and applications.</span></span> <span data-ttu-id="53426-147">Aunque esta es probablemente una característica de la interfaz del almacén de configuración, también es necesaria para garantizar que no se puede tener acceso directo a los datos de la memoria auxiliar sin el permiso adecuado.</span><span class="sxs-lookup"><span data-stu-id="53426-147">This is likely a feature of the configuration store interface, but it's also necessary to ensure that the data in the backing store can't be accessed directly without the appropriate permission.</span></span> <span data-ttu-id="53426-148">Asegúrese de que exista una separación estricta entre los permisos necesarios para leer y escribir datos de configuración.</span><span class="sxs-lookup"><span data-stu-id="53426-148">Ensure strict separation between the permissions required to read and to write configuration data.</span></span> <span data-ttu-id="53426-149">Considere también si debe cifrar algunos o todos los valores de configuración y cómo se implementará esto en la interfaz del almacén de configuración.</span><span class="sxs-lookup"><span data-stu-id="53426-149">Also consider whether you need to encrypt some or all of the configuration settings, and how this'll be implemented in the configuration store interface.</span></span>

<span data-ttu-id="53426-150">Las configuraciones almacenadas centralmente, que cambian el comportamiento de la aplicación en tiempo de ejecución, son muy importantes y se deben implementar, actualizar y administrar con los mismos mecanismos que se usan para implementar el código de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="53426-150">Centrally stored configurations, which change application behavior during runtime, are critically important and should be deployed, updated, and managed using the same mechanisms as deploying application code.</span></span> <span data-ttu-id="53426-151">Por ejemplo, los cambios que puedan afectar a más de una aplicación deben llevarse a cabo mediante un enfoque de prueba completa e implementación de ensayo para garantizar que el cambio es adecuado para todas las aplicaciones que usan esta configuración.</span><span class="sxs-lookup"><span data-stu-id="53426-151">For example, changes that can affect more than one application must be carried out using a full test and staged deployment approach to ensure that the change is appropriate for all applications that use this configuration.</span></span> <span data-ttu-id="53426-152">La modificación por un administrador de una opción para actualizar una aplicación, podría afectar negativamente a otras aplicaciones que usen la misma configuración.</span><span class="sxs-lookup"><span data-stu-id="53426-152">If an administrator edits a setting to update one application, it could adversely impact other applications that use the same setting.</span></span>

<span data-ttu-id="53426-153">Si una aplicación almacena en caché la información de configuración, será necesario alertarla si cambia la configuración.</span><span class="sxs-lookup"><span data-stu-id="53426-153">If an application caches configuration information, the application needs to be alerted if the configuration changes.</span></span> <span data-ttu-id="53426-154">Se podría implementar una directiva de caducidad sobre los datos de configuración almacenados en caché para que esta información se actualice automáticamente de forma periódica y se recojan los cambios (y se actúe en función de ellos).</span><span class="sxs-lookup"><span data-stu-id="53426-154">It might be possible to implement an expiration policy over cached configuration data so that this information is automatically refreshed periodically and any changes picked up (and acted on).</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="53426-155">Cuándo usar este patrón</span><span class="sxs-lookup"><span data-stu-id="53426-155">When to use this pattern</span></span>

<span data-ttu-id="53426-156">Este patrón es útil para:</span><span class="sxs-lookup"><span data-stu-id="53426-156">This pattern is useful for:</span></span>

- <span data-ttu-id="53426-157">Valores de configuración que se comparten entre varias aplicaciones e instancias de aplicación, o donde se deba aplicar una configuración estándar entre varias aplicaciones e instancias de aplicación.</span><span class="sxs-lookup"><span data-stu-id="53426-157">Configuration settings that are shared between multiple applications and application instances, or where a standard configuration must be enforced across multiple applications and application instances.</span></span>

- <span data-ttu-id="53426-158">Un sistema de configuración estándar que no admita todos los valores de configuración necesarios, como el almacenamiento de imágenes o tipos de datos complejos.</span><span class="sxs-lookup"><span data-stu-id="53426-158">A standard configuration system that doesn't support all of the required configuration settings, such as storing images or complex data types.</span></span>

- <span data-ttu-id="53426-159">Como almacén complementario para algunos de los valores de las aplicaciones, que permita quizás que las aplicaciones reemplacen algunos o todos los valores almacenados a nivel central.</span><span class="sxs-lookup"><span data-stu-id="53426-159">As a complementary store for some of the settings for applications, perhaps allowing applications to override some or all of the centrally-stored settings.</span></span>

- <span data-ttu-id="53426-160">Como forma de simplificar la administración de varias aplicaciones y, opcionalmente, de supervisar el uso de valores de configuración mediante el registro de algunos o todos los tipos de acceso en el almacén de configuración.</span><span class="sxs-lookup"><span data-stu-id="53426-160">As a way to simplify administration of multiple applications, and optionally for monitoring use of configuration settings by logging some or all types of access to the configuration store.</span></span>

## <a name="example"></a><span data-ttu-id="53426-161">Ejemplo</span><span class="sxs-lookup"><span data-stu-id="53426-161">Example</span></span>

<span data-ttu-id="53426-162">En una aplicación hospedada de Microsoft Azure, una opción típica para almacenar información de configuración externamente es usar Azure Storage.</span><span class="sxs-lookup"><span data-stu-id="53426-162">In a Microsoft Azure hosted application, a typical choice for storing configuration information externally is to use Azure Storage.</span></span> <span data-ttu-id="53426-163">Esta solución es resistente, ofrece un alto rendimiento y se replica tres veces con conmutación automática por error para ofrecer alta disponibilidad.</span><span class="sxs-lookup"><span data-stu-id="53426-163">This is resilient, offers high performance, and is replicated three times with automatic failover to offer high availability.</span></span> <span data-ttu-id="53426-164">Azure Table Storage proporciona un almacén de clave-valor con la posibilidad de usar un esquema flexible para los valores.</span><span class="sxs-lookup"><span data-stu-id="53426-164">Azure Table storage provides a key/value store with the ability to use a flexible schema for the values.</span></span> <span data-ttu-id="53426-165">Azure Blob Storage proporciona un almacén jerárquico basado en contenedores que puede contener cualquier tipo de datos en blobs nombrados de forma individual.</span><span class="sxs-lookup"><span data-stu-id="53426-165">Azure Blob storage provides a hierarchical, container-based store that can hold any type of data in individually named blobs.</span></span>

<span data-ttu-id="53426-166">En el ejemplo siguiente se muestra cómo se puede implementar un almacén de configuración sobre Blob Storage para almacenar y exponer la información de configuración.</span><span class="sxs-lookup"><span data-stu-id="53426-166">The following example shows how a configuration store can be implemented over Blob storage to store and expose configuration information.</span></span> <span data-ttu-id="53426-167">La clase `BlobSettingsStore` abstrae el almacenamiento de blobs para contener la información de configuración e implementa la interfaz `ISettingsStore` que se muestra en el código siguiente.</span><span class="sxs-lookup"><span data-stu-id="53426-167">The `BlobSettingsStore` class abstracts Blob storage for holding configuration information, and implements the `ISettingsStore` interface shown in the following code.</span></span>

> <span data-ttu-id="53426-168">Este código se proporciona en el proyecto _ExternalConfigurationStore.Cloud_ en la solución _ExternalConfigurationStore_, disponible en [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/external-configuration-store).</span><span class="sxs-lookup"><span data-stu-id="53426-168">This code is provided in the _ExternalConfigurationStore.Cloud_ project in the _ExternalConfigurationStore_ solution, available from [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/external-configuration-store).</span></span>

```csharp
public interface ISettingsStore
{
    Task<string> GetVersionAsync();

    Task<Dictionary<string, string>> FindAllAsync();
}
```

<span data-ttu-id="53426-169">Esta interfaz define métodos para recuperar y actualizar los valores de configuración contenidos en el almacén de configuración e incluye un número de versión que puede usarse para detectar si los parámetros de configuración se han modificado recientemente.</span><span class="sxs-lookup"><span data-stu-id="53426-169">This interface defines methods for retrieving and updating configuration settings held in the configuration store, and includes a version number that can be used to detect whether any configuration settings have been modified recently.</span></span> <span data-ttu-id="53426-170">La clase `BlobSettingsStore` usa la propiedad `ETag` del blob para implementar el control de versiones.</span><span class="sxs-lookup"><span data-stu-id="53426-170">The `BlobSettingsStore` class uses the `ETag` property of the blob to implement versioning.</span></span> <span data-ttu-id="53426-171">La propiedad `ETag` se actualiza automáticamente cada vez que se escribe el blob.</span><span class="sxs-lookup"><span data-stu-id="53426-171">The `ETag` property is updated automatically each time the blob is written.</span></span>

> <span data-ttu-id="53426-172">De forma predeterminada, esta solución simple expone todos los valores de configuración como valores de cadena en lugar de valores con tipo.</span><span class="sxs-lookup"><span data-stu-id="53426-172">By design, this simple solution exposes all configuration settings as string values rather than typed values.</span></span>

<span data-ttu-id="53426-173">La clase `ExternalConfigurationManager` proporciona un contenedor alrededor de un objeto `BlobSettingsStore`.</span><span class="sxs-lookup"><span data-stu-id="53426-173">The `ExternalConfigurationManager` class provides a wrapper around a `BlobSettingsStore` object.</span></span> <span data-ttu-id="53426-174">Una aplicación puede usar esta clase para almacenar y recuperar información de configuración.</span><span class="sxs-lookup"><span data-stu-id="53426-174">An application can use this class to store and retrieve configuration information.</span></span> <span data-ttu-id="53426-175">Esta clase usa la biblioteca [Reactive Extensions](https://msdn.microsoft.com/library/hh242985.aspx) de Microsoft para exponer los cambios realizados en la configuración a través de una implementación de la interfaz `IObservable`.</span><span class="sxs-lookup"><span data-stu-id="53426-175">This class uses the Microsoft [Reactive Extensions](https://msdn.microsoft.com/library/hh242985.aspx) library to expose any changes made to the configuration through an implementation of the `IObservable` interface.</span></span> <span data-ttu-id="53426-176">Si se modifica un valor de configuración mediante la llamada al método `SetAppSetting`, se genera el evento `Changed` y todos los suscriptores a este evento reciben una notificación.</span><span class="sxs-lookup"><span data-stu-id="53426-176">If a setting is modified by calling the `SetAppSetting` method, the `Changed` event is raised and all subscribers to this event will be notified.</span></span>

<span data-ttu-id="53426-177">Tenga en cuenta que toda la configuración también se almacena en caché en un objeto `Dictionary` dentro de la clase `ExternalConfigurationManager` para un fácil acceso.</span><span class="sxs-lookup"><span data-stu-id="53426-177">Note that all settings are also cached in a `Dictionary` object inside the `ExternalConfigurationManager` class for fast access.</span></span> <span data-ttu-id="53426-178">El método `GetSetting` usado para recuperar un valor de configuración lee los datos de la caché.</span><span class="sxs-lookup"><span data-stu-id="53426-178">The `GetSetting` method used to retrieve a configuration setting reads the data from the cache.</span></span> <span data-ttu-id="53426-179">Si el valor de configuración no se encuentra en la caché, se recupera en su lugar del objeto `BlobSettingsStore`.</span><span class="sxs-lookup"><span data-stu-id="53426-179">If the setting isn't found in the cache, it's retrieved from the `BlobSettingsStore` object instead.</span></span>

<span data-ttu-id="53426-180">El método `GetSettings` invoca el método `CheckForConfigurationChanges` para detectar si se ha cambiado la información de configuración en el almacenamiento de blobs.</span><span class="sxs-lookup"><span data-stu-id="53426-180">The `GetSettings` method invokes the `CheckForConfigurationChanges` method to detect whether the configuration information in blob storage has changed.</span></span> <span data-ttu-id="53426-181">Para ello, examina el número de versión y lo compara con el número de versión actual que contiene el objeto `ExternalConfigurationManager`.</span><span class="sxs-lookup"><span data-stu-id="53426-181">It does this by examining the version number and comparing it with the current version number held by the `ExternalConfigurationManager` object.</span></span> <span data-ttu-id="53426-182">Si se han producido uno o más cambios, se genera el evento `Changed` y se actualizan los valores de configuración almacenados en caché en el objeto `Dictionary`.</span><span class="sxs-lookup"><span data-stu-id="53426-182">If one or more changes have occurred, the `Changed` event is raised and the configuration settings cached in the `Dictionary` object are refreshed.</span></span> <span data-ttu-id="53426-183">Esta es una aplicación del [patrón Cache-Aside](cache-aside.md).</span><span class="sxs-lookup"><span data-stu-id="53426-183">This is an application of the [Cache-Aside pattern](cache-aside.md).</span></span>

<span data-ttu-id="53426-184">El siguiente código de ejemplo se muestra cómo se implementa el evento `Changed` eventos, el método `GetSettings` y el método `CheckForConfigurationChanges`:</span><span class="sxs-lookup"><span data-stu-id="53426-184">The following code sample shows how the `Changed` event, the `GetSettings` method, and the `CheckForConfigurationChanges` method are implemented:</span></span>

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

> <span data-ttu-id="53426-185">La clase `ExternalConfigurationManager` también proporciona una propiedad denominada `Environment`.</span><span class="sxs-lookup"><span data-stu-id="53426-185">The `ExternalConfigurationManager` class also provides a property named `Environment`.</span></span> <span data-ttu-id="53426-186">Esta propiedad admite distintas configuraciones para una aplicación que se ejecuta en entornos diferentes, como ensayo y producción.</span><span class="sxs-lookup"><span data-stu-id="53426-186">This property supports varying configurations for an application running in different environments, such as staging and production.</span></span>

<span data-ttu-id="53426-187">Un objeto `ExternalConfigurationManager` también puede consultar el objeto `BlobSettingsStore` periódicamente para comprobar los posibles cambios.</span><span class="sxs-lookup"><span data-stu-id="53426-187">An `ExternalConfigurationManager` object can also query the `BlobSettingsStore` object periodically for any changes.</span></span> <span data-ttu-id="53426-188">En el código siguiente, el método `StartMonitor` llama a `CheckForConfigurationChanges` en un intervalo para detectar los cambios y generar el evento `Changed`, como se describió anteriormente.</span><span class="sxs-lookup"><span data-stu-id="53426-188">In the following code, the `StartMonitor` method calls `CheckForConfigurationChanges` at an interval to detect any changes and raise the `Changed` event, as described earlier.</span></span>

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

<span data-ttu-id="53426-189">Se crea una instancia de la clase `ExternalConfigurationManager` como una instancia de singleton mediante la clase `ExternalConfiguration` mostrada a continuación.</span><span class="sxs-lookup"><span data-stu-id="53426-189">The `ExternalConfigurationManager` class is instantiated as a singleton instance by the `ExternalConfiguration` class shown below.</span></span>

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

<span data-ttu-id="53426-190">El código siguiente se toma de la clase `WorkerRole` en el proyecto _ExternalConfigurationStore.Cloud_.</span><span class="sxs-lookup"><span data-stu-id="53426-190">The following code is taken from the `WorkerRole` class in the _ExternalConfigurationStore.Cloud_ project.</span></span> <span data-ttu-id="53426-191">Muestra cómo la aplicación usa la clase `ExternalConfiguration` para leer un valor de configuración.</span><span class="sxs-lookup"><span data-stu-id="53426-191">It shows how the application uses the `ExternalConfiguration` class to read a setting.</span></span>

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

<span data-ttu-id="53426-192">El siguiente código, también de la clase `WorkerRole`, muestra cómo la aplicación se suscribe a eventos de configuración.</span><span class="sxs-lookup"><span data-stu-id="53426-192">The following code, also from the `WorkerRole` class, shows how the application subscribes to configuration events.</span></span>

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

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="53426-193">Orientación y patrones relacionados</span><span class="sxs-lookup"><span data-stu-id="53426-193">Related patterns and guidance</span></span>

- <span data-ttu-id="53426-194">Se encuentra disponible un ejemplo que demuestra este patrón en [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/external-configuration-store).</span><span class="sxs-lookup"><span data-stu-id="53426-194">A sample that demonstrates this pattern is available on [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/external-configuration-store).</span></span>
