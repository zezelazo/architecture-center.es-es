---
title: Hospedaje de contenido estático
description: Implemente contenido estático en un servicio de almacenamiento basado en la nube que pueda entregarlo directamente al cliente.
keywords: Patrón de diseño
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- data-management
- design-implementation
- performance-scalability
ms.openlocfilehash: deb15001bea2598d56a2793be78bbc3e7473bdf3
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2017
ms.locfileid: "24541695"
---
# <a name="static-content-hosting-pattern"></a><span data-ttu-id="ee26a-104">Patrón Static Content Hosting</span><span class="sxs-lookup"><span data-stu-id="ee26a-104">Static Content Hosting pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="ee26a-105">Implemente contenido estático en un servicio de almacenamiento basado en la nube que pueda entregarlo directamente al cliente.</span><span class="sxs-lookup"><span data-stu-id="ee26a-105">Deploy static content to a cloud-based storage service that can deliver them directly to the client.</span></span> <span data-ttu-id="ee26a-106">Así, puede reducir la necesidad de instancias de proceso potencialmente costosas.</span><span class="sxs-lookup"><span data-stu-id="ee26a-106">This can reduce the need for potentially expensive compute instances.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="ee26a-107">Contexto y problema</span><span class="sxs-lookup"><span data-stu-id="ee26a-107">Context and problem</span></span>

<span data-ttu-id="ee26a-108">Las aplicaciones web suelen incluyen algunos elementos de contenido estático.</span><span class="sxs-lookup"><span data-stu-id="ee26a-108">Web applications typically include some elements of static content.</span></span> <span data-ttu-id="ee26a-109">Este contenido estático puede incluir páginas HTML y otros recursos, como imágenes y documentos que están disponibles para el cliente, bien como parte de una página HTML (por ejemplo, imágenes en línea, hojas de estilos y archivos de JavaScript del lado cliente) o como descargas independientes (como documentos PDF).</span><span class="sxs-lookup"><span data-stu-id="ee26a-109">This static content might include HTML pages and other resources such as images and documents that are available to the client, either as part of an HTML page (such as inline images, style sheets, and client-side JavaScript files) or as separate downloads (such as PDF documents).</span></span>

<span data-ttu-id="ee26a-110">Aunque los servidores web están bien ajustados para optimizar las solicitudes gracias a la ejecución de código de página eficiente y dinámico y el almacenamiento en caché de la salida, aún tienen que tratar las solicitudes para descargar contenido estático.</span><span class="sxs-lookup"><span data-stu-id="ee26a-110">Although web servers are well tuned to optimize requests through efficient dynamic page code execution and output caching, they still have to handle requests to download static content.</span></span> <span data-ttu-id="ee26a-111">Como consecuencia, se consumen ciclos de procesamiento que con frecuencia se podrían aprovecharse mejor.</span><span class="sxs-lookup"><span data-stu-id="ee26a-111">This consumes processing cycles that could often be put to better use.</span></span>

## <a name="solution"></a><span data-ttu-id="ee26a-112">Solución</span><span class="sxs-lookup"><span data-stu-id="ee26a-112">Solution</span></span>

<span data-ttu-id="ee26a-113">En la mayoría de los entornos de hospedaje en la nube, es posible reducir la necesidad de instancias de proceso (por ejemplo, usar una instancia más pequeña o menos instancias), mediante la ubicación de algunos de los recursos y páginas estáticas de una aplicación en un servicio de almacenamiento.</span><span class="sxs-lookup"><span data-stu-id="ee26a-113">In most cloud hosting environments it's possible to minimize the need for compute instances (for example, use a smaller instance or fewer instances), by locating some of an application’s resources and static pages in a storage service.</span></span> <span data-ttu-id="ee26a-114">El coste de almacenamiento hospedado en la nube es normalmente mucho menor que para las instancias de proceso.</span><span class="sxs-lookup"><span data-stu-id="ee26a-114">The cost for cloud-hosted storage is typically much less than for compute instances.</span></span>

<span data-ttu-id="ee26a-115">Al hospedar algunas partes de una aplicación en un servicio de almacenamiento, los principales aspectos que hay que tener en cuenta están relacionados con la implementación de la aplicación y la protección de los recursos que no están destinados a estar disponibles para usuarios anónimos.</span><span class="sxs-lookup"><span data-stu-id="ee26a-115">When hosting some parts of an application in a storage service, the main considerations are related to deployment of the application and to securing resources that aren't intended to be available to anonymous users.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="ee26a-116">Problemas y consideraciones</span><span class="sxs-lookup"><span data-stu-id="ee26a-116">Issues and considerations</span></span>

<span data-ttu-id="ee26a-117">Tenga en cuenta los puntos siguientes al decidir cómo implementar este patrón:</span><span class="sxs-lookup"><span data-stu-id="ee26a-117">Consider the following points when deciding how to implement this pattern:</span></span>

- <span data-ttu-id="ee26a-118">El servicio de almacenamiento hospedado debe exponer un punto de conexión HTTP al que los usuarios puedan acceder para descargar los recursos estáticos.</span><span class="sxs-lookup"><span data-stu-id="ee26a-118">The hosted storage service must expose an HTTP endpoint that users can access to download the static resources.</span></span> <span data-ttu-id="ee26a-119">Algunos servicios de almacenamiento también admiten HTTPS, por lo que es posible hospedar los recursos en servicios de almacenamiento que requieran SSL.</span><span class="sxs-lookup"><span data-stu-id="ee26a-119">Some storage services also support HTTPS, so it's possible to host resources in storage services that require SSL.</span></span>

- <span data-ttu-id="ee26a-120">Para obtener el máximo rendimiento y disponibilidad, considere la posibilidad de usar una red de entrega de contenido (CDN) para almacenar en caché el contenido del contenedor de almacenamiento en varios centros de datos por todo el mundo.</span><span class="sxs-lookup"><span data-stu-id="ee26a-120">For maximum performance and availability, consider using a content delivery network (CDN) to cache the contents of the storage container in multiple datacenters around the world.</span></span> <span data-ttu-id="ee26a-121">Sin embargo, probablemente deba pagar por el uso de la red CDN.</span><span class="sxs-lookup"><span data-stu-id="ee26a-121">However, you'll likely have to pay for using the CDN.</span></span>

- <span data-ttu-id="ee26a-122">Las cuentas de almacenamiento a menudo se replican geográficamente de forma predeterminada para proporcionar resistencia frente a eventos que podrían afectar a un centro de datos.</span><span class="sxs-lookup"><span data-stu-id="ee26a-122">Storage accounts are often geo-replicated by default to provide resiliency against events that might affect a datacenter.</span></span> <span data-ttu-id="ee26a-123">Esto significa que la dirección IP podría cambiar, pero la dirección URL seguirá siendo la misma.</span><span class="sxs-lookup"><span data-stu-id="ee26a-123">This means that the IP address might change, but the URL will remain the same.</span></span>

- <span data-ttu-id="ee26a-124">Cuando parte del contenido está ubicado en una cuenta de almacenamiento y otra parte en una instancia de proceso hospedada, se vuelve más difícil implementar una aplicación y actualizarla.</span><span class="sxs-lookup"><span data-stu-id="ee26a-124">When some content is located in a storage account and other content is in a hosted compute instance it becomes more challenging to deploy an application and to update it.</span></span> <span data-ttu-id="ee26a-125">Es posible que deba realizar implementaciones independientes y controlar las versiones de la aplicación y el contenido para administrarlo de forma más fácil&mdash;en especial, cuando el contenido estático incluye archivos de script o componentes de interfaz de usuario.</span><span class="sxs-lookup"><span data-stu-id="ee26a-125">You might have to perform separate deployments, and version the application and content to manage it more easily&mdash;especially when the static content includes script files or UI components.</span></span> <span data-ttu-id="ee26a-126">Sin embargo, si solo se tienen que actualizar recursos estáticos, pueden cargarse simplemente en la cuenta de almacenamiento sin necesidad de volver a implementar el paquete de aplicación.</span><span class="sxs-lookup"><span data-stu-id="ee26a-126">However, if only static resources have to be updated, they can simply be uploaded to the storage account without needing to redeploy the application package.</span></span>

- <span data-ttu-id="ee26a-127">Puede que los servicios de almacenamiento no admitan el uso de nombres de dominio personalizados.</span><span class="sxs-lookup"><span data-stu-id="ee26a-127">Storage services might not support the use of custom domain names.</span></span> <span data-ttu-id="ee26a-128">En este caso, es necesario especificar la dirección URL completa de los recursos en los vínculos ya que estarán en un dominio diferente al del contenido generado dinámicamente que incluye los vínculos.</span><span class="sxs-lookup"><span data-stu-id="ee26a-128">In this case it's necessary to specify the full URL of the resources in links because they'll be in a different domain from the dynamically-generated content containing the links.</span></span>

- <span data-ttu-id="ee26a-129">Los contenedores de almacenamiento deben estar configurados para el acceso de lectura público, pero es esencial asegurarse de que no estén configurados para el acceso de escritura público para impedir que los usuarios puedan cargar contenido.</span><span class="sxs-lookup"><span data-stu-id="ee26a-129">The storage containers must be configured for public read access, but it's vital to ensure that they aren't configured for public write access to prevent users being able to upload content.</span></span> <span data-ttu-id="ee26a-130">Considere el uso de una clave de valet o token para controlar el acceso a los recursos que no deben estar disponibles de forma anónima&mdash;consulte [Valet Key pattern](valet-key.md) (Patrón Valet Key) para más información.</span><span class="sxs-lookup"><span data-stu-id="ee26a-130">Consider using a valet key or token to control access to resources that shouldn't be available anonymously&mdash;see the [Valet Key pattern](valet-key.md) for more information.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="ee26a-131">Cuándo usar este patrón</span><span class="sxs-lookup"><span data-stu-id="ee26a-131">When to use this pattern</span></span>

<span data-ttu-id="ee26a-132">Este patrón es útil para:</span><span class="sxs-lookup"><span data-stu-id="ee26a-132">This pattern is useful for:</span></span>

- <span data-ttu-id="ee26a-133">Minimizar el coste de hospedaje de sitios web y aplicaciones que contienen algunos recursos estáticos.</span><span class="sxs-lookup"><span data-stu-id="ee26a-133">Minimizing the hosting cost for websites and applications that contain some static resources.</span></span>

- <span data-ttu-id="ee26a-134">Minimizar el coste de hospedaje de sitios web que solo constan de contenido y recursos estáticos.</span><span class="sxs-lookup"><span data-stu-id="ee26a-134">Minimizing the hosting cost for websites that consist of only static content and resources.</span></span> <span data-ttu-id="ee26a-135">Según las funcionalidades del sistema de almacenamiento del proveedor de hospedaje, puede que exista la posibilidad de hospedar por completo un sitio web totalmente estático en una cuenta de almacenamiento.</span><span class="sxs-lookup"><span data-stu-id="ee26a-135">Depending on the capabilities of the hosting provider’s storage system, it might be possible to entirely host a fully static website in a storage account.</span></span>

- <span data-ttu-id="ee26a-136">Exponer recursos y contenido estáticos para aplicaciones que se ejecutan en otros entornos de hospedaje o en servidores locales.</span><span class="sxs-lookup"><span data-stu-id="ee26a-136">Exposing static resources and content for applications running in other hosting environments or on-premises servers.</span></span>

- <span data-ttu-id="ee26a-137">Ubicar contenido en más de un área geográfica mediante una red de entrega de contenido que almacena en caché el contenido de la cuenta de almacenamiento en varios centros de datos por todo el mundo.</span><span class="sxs-lookup"><span data-stu-id="ee26a-137">Locating content in more than one geographical area using a content delivery network that caches the contents of the storage account in multiple datacenters around the world.</span></span>

- <span data-ttu-id="ee26a-138">Supervisar los costes y el uso de ancho de banda.</span><span class="sxs-lookup"><span data-stu-id="ee26a-138">Monitoring costs and bandwidth usage.</span></span> <span data-ttu-id="ee26a-139">Usar una cuenta de almacenamiento independiente para todo el contenido estático o parte del mismo permite separar más fácilmente los costes de los costes de hospedaje y tiempo de ejecución.</span><span class="sxs-lookup"><span data-stu-id="ee26a-139">Using a separate storage account for some or all of the static content allows the costs to be more easily separated from hosting and runtime costs.</span></span>

<span data-ttu-id="ee26a-140">Este patrón podría no ser útil en las siguientes situaciones:</span><span class="sxs-lookup"><span data-stu-id="ee26a-140">This pattern might not be useful in the following situations:</span></span>

- <span data-ttu-id="ee26a-141">La aplicación debe realizar algún procesamiento en el contenido estático antes de entregarlo al cliente.</span><span class="sxs-lookup"><span data-stu-id="ee26a-141">The application needs to perform some processing on the static content before delivering it to the client.</span></span> <span data-ttu-id="ee26a-142">Por ejemplo, podría ser necesario agregar una marca de tiempo a un documento.</span><span class="sxs-lookup"><span data-stu-id="ee26a-142">For example, it might be necessary to add a timestamp to a document.</span></span>

- <span data-ttu-id="ee26a-143">El volumen del contenido estático es muy pequeño.</span><span class="sxs-lookup"><span data-stu-id="ee26a-143">The volume of static content is very small.</span></span> <span data-ttu-id="ee26a-144">La sobrecarga que supone recuperar este contenido del almacenamiento independiente puede sobrepasar las ventajas en los costes de separarlo del recurso de proceso.</span><span class="sxs-lookup"><span data-stu-id="ee26a-144">The overhead of retrieving this content from separate storage can outweigh the cost benefit of separating it out from the compute resource.</span></span>

## <a name="example"></a><span data-ttu-id="ee26a-145">Ejemplo</span><span class="sxs-lookup"><span data-stu-id="ee26a-145">Example</span></span>

<span data-ttu-id="ee26a-146">Se puede acceder directamente al contenido estático ubicado en Azure Blob Storage mediante un explorador web.</span><span class="sxs-lookup"><span data-stu-id="ee26a-146">Static content located in Azure Blob storage can be accessed directly by a web browser.</span></span> <span data-ttu-id="ee26a-147">Azure proporciona una interfaz basada en HTTP encima del almacenamiento que se pueden exponer públicamente a los clientes.</span><span class="sxs-lookup"><span data-stu-id="ee26a-147">Azure provides an HTTP-based interface over storage that can be publicly exposed to clients.</span></span> <span data-ttu-id="ee26a-148">Por ejemplo, el contenido de un contenedor de Azure Blob Storage se expone mediante una dirección URL con el formato siguiente:</span><span class="sxs-lookup"><span data-stu-id="ee26a-148">For example, content in an Azure Blob storage container is exposed using a URL with the following form:</span></span>

`http://[ storage-account-name ].blob.core.windows.net/[ container-name ]/[ file-name ]`


<span data-ttu-id="ee26a-149">Al cargar el contenido, es necesario crear uno o varios contenedores de blobs para almacenar los archivos y documentos.</span><span class="sxs-lookup"><span data-stu-id="ee26a-149">When uploading the content it's necessary to create one or more blob containers to hold the files and documents.</span></span> <span data-ttu-id="ee26a-150">Tenga en cuenta que el permiso predeterminado para un nuevo contenedor es privado y debe cambiarlo a público para permitir que los clientes accedan al contenido.</span><span class="sxs-lookup"><span data-stu-id="ee26a-150">Note that the default permission for a new container is Private, and you must change this to Public to allow clients to access the contents.</span></span> <span data-ttu-id="ee26a-151">Si es necesario proteger el contenido contra el acceso anónimo, puede implementar el [patrón Vale Key](valet-key.md), donde los usuarios deben presentar un token válido para descargar los recursos.</span><span class="sxs-lookup"><span data-stu-id="ee26a-151">If it's necessary to protect the content from anonymous access, you can implement the [Valet Key pattern](valet-key.md) so users must present a valid token to download the resources.</span></span>

> <span data-ttu-id="ee26a-152">En [Blob Service Concepts](https://msdn.microsoft.com/library/azure/dd179376.aspx) (Conceptos de Blob Service) hay información sobre el almacenamiento de blobs y las formas de acceder a él y usarlo.</span><span class="sxs-lookup"><span data-stu-id="ee26a-152">[Blob Service Concepts](https://msdn.microsoft.com/library/azure/dd179376.aspx) has information about blob storage, and the ways that you can access and use it.</span></span>

<span data-ttu-id="ee26a-153">Los vínculos de cada página especifican la dirección URL del recurso y el cliente accede a él directamente desde el servicio de almacenamiento.</span><span class="sxs-lookup"><span data-stu-id="ee26a-153">The links in each page will specify the URL of the resource and the client will access it directly from the storage service.</span></span> <span data-ttu-id="ee26a-154">La ilustración muestra la entrega de las partes estáticas de una aplicación directamente desde un servicio de almacenamiento.</span><span class="sxs-lookup"><span data-stu-id="ee26a-154">The figure illustrates delivering static parts of an application directly from a storage service.</span></span>

![Figura 1: Entrega de las partes estáticas de una aplicación directamente desde un servicio de almacenamiento](./_images/static-content-hosting-pattern.png)


<span data-ttu-id="ee26a-156">Los vínculos de las páginas entregadas al cliente deben especificar la dirección URL completa del contenedor de blobs y los recursos.</span><span class="sxs-lookup"><span data-stu-id="ee26a-156">The links in the pages delivered to the client must specify the full URL of the blob container and resource.</span></span> <span data-ttu-id="ee26a-157">Por ejemplo, una página que contenga un vínculo a una imagen en un contenedor público podría contener el siguiente código HTML.</span><span class="sxs-lookup"><span data-stu-id="ee26a-157">For example, a page that contains a link to an image in a public container might contain the following HTML.</span></span>

```html
<img src="http://mystorageaccount.blob.core.windows.net/myresources/image1.png"
     alt="My image" />
```

> <span data-ttu-id="ee26a-158">Si los recursos están protegidos mediante una clave de valet, como una firma de acceso compartido de Azure, esta firma debe incluirse en las direcciones URL de los vínculos.</span><span class="sxs-lookup"><span data-stu-id="ee26a-158">If the resources are protected by using a valet key, such as an Azure shared access signature, this signature must be included in the URLs in the links.</span></span>

<span data-ttu-id="ee26a-159">Existe una solución denominada StaticContentHosting que demuestra el uso de almacenamiento externo para recursos estáticos que se puede encontrar en [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting).</span><span class="sxs-lookup"><span data-stu-id="ee26a-159">A solution named StaticContentHosting that demonstrates using external storage for static resources is available from [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting).</span></span> <span data-ttu-id="ee26a-160">El proyecto StaticContentHosting.Cloud contiene los archivos de configuración que especifican la cuenta de almacenamiento y el contenedor que incluye el contenido estático.</span><span class="sxs-lookup"><span data-stu-id="ee26a-160">The StaticContentHosting.Cloud project contains configuration files that specify the storage account and container that holds the static content.</span></span>

```xml
<Setting name="StaticContent.StorageConnectionString"
         value="UseDevelopmentStorage=true" />
<Setting name="StaticContent.Container" value="static-content" />
```

<span data-ttu-id="ee26a-161">La clase `Settings` en el archivo Settings.cs del proyecto StaticContentHosting.Web contiene métodos para extraer estos valores y crear un valor de cadena que contiene la dirección URL del contenedor de la cuenta de almacenamiento en la nube.</span><span class="sxs-lookup"><span data-stu-id="ee26a-161">The `Settings` class in the file Settings.cs of the StaticContentHosting.Web project contains methods to extract these values and build a string value containing the cloud storage account container URL.</span></span>

```csharp
public class Settings
{
  public static string StaticContentStorageConnectionString {
    get
    {
      return RoleEnvironment.GetConfigurationSettingValue(
                              "StaticContent.StorageConnectionString");
    }
  }

  public static string StaticContentContainer
  {
    get
    {
      return RoleEnvironment.GetConfigurationSettingValue("StaticContent.Container");
    }
  }

  public static string StaticContentBaseUrl
  {
    get
    {
      var account = CloudStorageAccount.Parse(StaticContentStorageConnectionString);

      return string.Format("{0}/{1}", account.BlobEndpoint.ToString().TrimEnd('/'),
                                      StaticContentContainer.TrimStart('/'));
    }
  }
}
```

<span data-ttu-id="ee26a-162">La clase `StaticContentUrlHtmlHelper` en el archivo StaticContentUrlHtmlHelper.cs expone un método denominado `StaticContentUrl` que genera una dirección URL que contiene la ruta de acceso a la cuenta de almacenamiento en la nube si la dirección URL pasada a esta comienza con el carácter de ruta de acceso raíz ASP.NET (~).</span><span class="sxs-lookup"><span data-stu-id="ee26a-162">The `StaticContentUrlHtmlHelper` class in the file StaticContentUrlHtmlHelper.cs exposes a method named `StaticContentUrl` that generates a URL containing the path to the cloud storage account if the URL passed to it starts with the ASP.NET root path character (~).</span></span>

```csharp
public static class StaticContentUrlHtmlHelper
{
  public static string StaticContentUrl(this HtmlHelper helper, string contentPath)
  {
    if (contentPath.StartsWith("~"))
    {
      contentPath = contentPath.Substring(1);
    }

    contentPath = string.Format("{0}/{1}", Settings.StaticContentBaseUrl.TrimEnd('/'),
                                contentPath.TrimStart('/'));

    var url = new UrlHelper(helper.ViewContext.RequestContext);

    return url.Content(contentPath);
  }
}
```

<span data-ttu-id="ee26a-163">El archivo Index.cshtml de la carpeta Views\Home contiene un elemento de imagen que usa el método `StaticContentUrl` para crear la dirección URL de su atributo `src`.</span><span class="sxs-lookup"><span data-stu-id="ee26a-163">The file Index.cshtml in the Views\Home folder contains an image element that uses the `StaticContentUrl` method to create the URL for its `src` attribute.</span></span>

```html
<img src="@Html.StaticContentUrl("~/media/orderedList1.png")" alt="Test Image" />
```

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="ee26a-164">Orientación y patrones relacionados</span><span class="sxs-lookup"><span data-stu-id="ee26a-164">Related patterns and guidance</span></span>

- <span data-ttu-id="ee26a-165">Se encuentra disponible un ejemplo que demuestra este patrón en [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting).</span><span class="sxs-lookup"><span data-stu-id="ee26a-165">A sample that demonstrates this pattern is available on [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting).</span></span>
- <span data-ttu-id="ee26a-166">[Patrón Valet Key](valet-key.md).</span><span class="sxs-lookup"><span data-stu-id="ee26a-166">[Valet Key pattern](valet-key.md).</span></span> <span data-ttu-id="ee26a-167">Si se supone que los recursos de destino no van a estar disponibles para los usuarios anónimos, es necesario implementar la seguridad sobre el almacén que alberga el contenido estático.</span><span class="sxs-lookup"><span data-stu-id="ee26a-167">If the target resources aren't supposed to be available to anonymous users it's necessary to implement security over the store that holds the static content.</span></span> <span data-ttu-id="ee26a-168">En este artículo se describe el uso de un token o una clave que proporciona a los clientes acceso directo restringido a un recurso o servicio específico, como un servicio de almacenamiento hospedado en la nube.</span><span class="sxs-lookup"><span data-stu-id="ee26a-168">Describes how to use a token or key that provides clients with restricted direct access to a specific resource or service such as a cloud-hosted storage service.</span></span>
- <span data-ttu-id="ee26a-169">[Una manera eficaz de implementar un sitio web estático en Azure](http://www.infosysblogs.com/microsoft/2010/06/an_efficient_way_of_deploying.html) en el blog de Infosys.</span><span class="sxs-lookup"><span data-stu-id="ee26a-169">[An efficient way of deploying a static web site on Azure](http://www.infosysblogs.com/microsoft/2010/06/an_efficient_way_of_deploying.html) on the Infosys blog.</span></span>
- [<span data-ttu-id="ee26a-170">Conceptos de Blob service</span><span class="sxs-lookup"><span data-stu-id="ee26a-170">Blob Service Concepts</span></span>](https://msdn.microsoft.com/library/azure/dd179376.aspx)
