---
title: "Hospedaje de contenido estático"
description: "Implemente contenido estático en un servicio de almacenamiento basado en la nube que pueda entregarlo directamente al cliente."
keywords: "Patrón de diseño"
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
---
# <a name="static-content-hosting-pattern"></a>Patrón Static Content Hosting

[!INCLUDE [header](../_includes/header.md)]

Implemente contenido estático en un servicio de almacenamiento basado en la nube que pueda entregarlo directamente al cliente. Así, puede reducir la necesidad de instancias de proceso potencialmente costosas.

## <a name="context-and-problem"></a>Contexto y problema

Las aplicaciones web suelen incluyen algunos elementos de contenido estático. Este contenido estático puede incluir páginas HTML y otros recursos, como imágenes y documentos que están disponibles para el cliente, bien como parte de una página HTML (por ejemplo, imágenes en línea, hojas de estilos y archivos de JavaScript del lado cliente) o como descargas independientes (como documentos PDF).

Aunque los servidores web están bien ajustados para optimizar las solicitudes gracias a la ejecución de código de página eficiente y dinámico y el almacenamiento en caché de la salida, aún tienen que tratar las solicitudes para descargar contenido estático. Como consecuencia, se consumen ciclos de procesamiento que con frecuencia se podrían aprovecharse mejor.

## <a name="solution"></a>Solución

En la mayoría de los entornos de hospedaje en la nube, es posible reducir la necesidad de instancias de proceso (por ejemplo, usar una instancia más pequeña o menos instancias), mediante la ubicación de algunos de los recursos y páginas estáticas de una aplicación en un servicio de almacenamiento. El coste de almacenamiento hospedado en la nube es normalmente mucho menor que para las instancias de proceso.

Al hospedar algunas partes de una aplicación en un servicio de almacenamiento, los principales aspectos que hay que tener en cuenta están relacionados con la implementación de la aplicación y la protección de los recursos que no están destinados a estar disponibles para usuarios anónimos.

## <a name="issues-and-considerations"></a>Problemas y consideraciones

Tenga en cuenta los puntos siguientes al decidir cómo implementar este patrón:

- El servicio de almacenamiento hospedado debe exponer un punto de conexión HTTP al que los usuarios puedan acceder para descargar los recursos estáticos. Algunos servicios de almacenamiento también admiten HTTPS, por lo que es posible hospedar los recursos en servicios de almacenamiento que requieran SSL.

- Para obtener el máximo rendimiento y disponibilidad, considere la posibilidad de usar una red de entrega de contenido (CDN) para almacenar en caché el contenido del contenedor de almacenamiento en varios centros de datos por todo el mundo. Sin embargo, probablemente deba pagar por el uso de la red CDN.

- Las cuentas de almacenamiento a menudo se replican geográficamente de forma predeterminada para proporcionar resistencia frente a eventos que podrían afectar a un centro de datos. Esto significa que la dirección IP podría cambiar, pero la dirección URL seguirá siendo la misma.

- Cuando parte del contenido está ubicado en una cuenta de almacenamiento y otra parte en una instancia de proceso hospedada, se vuelve más difícil implementar una aplicación y actualizarla. Es posible que deba realizar implementaciones independientes y controlar las versiones de la aplicación y el contenido para administrarlo de forma más fácil&mdash;en especial, cuando el contenido estático incluye archivos de script o componentes de interfaz de usuario. Sin embargo, si solo se tienen que actualizar recursos estáticos, pueden cargarse simplemente en la cuenta de almacenamiento sin necesidad de volver a implementar el paquete de aplicación.

- Puede que los servicios de almacenamiento no admitan el uso de nombres de dominio personalizados. En este caso, es necesario especificar la dirección URL completa de los recursos en los vínculos ya que estarán en un dominio diferente al del contenido generado dinámicamente que incluye los vínculos.

- Los contenedores de almacenamiento deben estar configurados para el acceso de lectura público, pero es esencial asegurarse de que no estén configurados para el acceso de escritura público para impedir que los usuarios puedan cargar contenido. Considere el uso de una clave de valet o token para controlar el acceso a los recursos que no deben estar disponibles de forma anónima&mdash;consulte [Valet Key pattern](valet-key.md) (Patrón Valet Key) para más información.

## <a name="when-to-use-this-pattern"></a>Cuándo usar este patrón

Este patrón es útil para:

- Minimizar el coste de hospedaje de sitios web y aplicaciones que contienen algunos recursos estáticos.

- Minimizar el coste de hospedaje de sitios web que solo constan de contenido y recursos estáticos. Según las funcionalidades del sistema de almacenamiento del proveedor de hospedaje, puede que exista la posibilidad de hospedar por completo un sitio web totalmente estático en una cuenta de almacenamiento.

- Exponer recursos y contenido estáticos para aplicaciones que se ejecutan en otros entornos de hospedaje o en servidores locales.

- Ubicar contenido en más de un área geográfica mediante una red de entrega de contenido que almacena en caché el contenido de la cuenta de almacenamiento en varios centros de datos por todo el mundo.

- Supervisar los costes y el uso de ancho de banda. Usar una cuenta de almacenamiento independiente para todo el contenido estático o parte del mismo permite separar más fácilmente los costes de los costes de hospedaje y tiempo de ejecución.

Este patrón podría no ser útil en las siguientes situaciones:

- La aplicación debe realizar algún procesamiento en el contenido estático antes de entregarlo al cliente. Por ejemplo, podría ser necesario agregar una marca de tiempo a un documento.

- El volumen del contenido estático es muy pequeño. La sobrecarga que supone recuperar este contenido del almacenamiento independiente puede sobrepasar las ventajas en los costes de separarlo del recurso de proceso.

## <a name="example"></a>Ejemplo

Se puede acceder directamente al contenido estático ubicado en Azure Blob Storage mediante un explorador web. Azure proporciona una interfaz basada en HTTP encima del almacenamiento que se pueden exponer públicamente a los clientes. Por ejemplo, el contenido de un contenedor de Azure Blob Storage se expone mediante una dirección URL con el formato siguiente:

`http://[ storage-account-name ].blob.core.windows.net/[ container-name ]/[ file-name ]`


Al cargar el contenido, es necesario crear uno o varios contenedores de blobs para almacenar los archivos y documentos. Tenga en cuenta que el permiso predeterminado para un nuevo contenedor es privado y debe cambiarlo a público para permitir que los clientes accedan al contenido. Si es necesario proteger el contenido contra el acceso anónimo, puede implementar el [patrón Vale Key](valet-key.md), donde los usuarios deben presentar un token válido para descargar los recursos.

> En [Blob Service Concepts](https://msdn.microsoft.com/library/azure/dd179376.aspx) (Conceptos de Blob Service) hay información sobre el almacenamiento de blobs y las formas de acceder a él y usarlo.

Los vínculos de cada página especifican la dirección URL del recurso y el cliente accede a él directamente desde el servicio de almacenamiento. La ilustración muestra la entrega de las partes estáticas de una aplicación directamente desde un servicio de almacenamiento.

![Figura 1: Entrega de las partes estáticas de una aplicación directamente desde un servicio de almacenamiento](./_images/static-content-hosting-pattern.png)


Los vínculos de las páginas entregadas al cliente deben especificar la dirección URL completa del contenedor de blobs y los recursos. Por ejemplo, una página que contenga un vínculo a una imagen en un contenedor público podría contener el siguiente código HTML.

```html
<img src="http://mystorageaccount.blob.core.windows.net/myresources/image1.png"
     alt="My image" />
```

> Si los recursos están protegidos mediante una clave de valet, como una firma de acceso compartido de Azure, esta firma debe incluirse en las direcciones URL de los vínculos.

Existe una solución denominada StaticContentHosting que demuestra el uso de almacenamiento externo para recursos estáticos que se puede encontrar en [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting). El proyecto StaticContentHosting.Cloud contiene los archivos de configuración que especifican la cuenta de almacenamiento y el contenedor que incluye el contenido estático.

```xml
<Setting name="StaticContent.StorageConnectionString"
         value="UseDevelopmentStorage=true" />
<Setting name="StaticContent.Container" value="static-content" />
```

La clase `Settings` en el archivo Settings.cs del proyecto StaticContentHosting.Web contiene métodos para extraer estos valores y crear un valor de cadena que contiene la dirección URL del contenedor de la cuenta de almacenamiento en la nube.

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

La clase `StaticContentUrlHtmlHelper` en el archivo StaticContentUrlHtmlHelper.cs expone un método denominado `StaticContentUrl` que genera una dirección URL que contiene la ruta de acceso a la cuenta de almacenamiento en la nube si la dirección URL pasada a esta comienza con el carácter de ruta de acceso raíz ASP.NET (~).

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

El archivo Index.cshtml de la carpeta Views\Home contiene un elemento de imagen que usa el método `StaticContentUrl` para crear la dirección URL de su atributo `src`.

```html
<img src="@Html.StaticContentUrl("~/media/orderedList1.png")" alt="Test Image" />
```

## <a name="related-patterns-and-guidance"></a>Orientación y patrones relacionados

- Se encuentra disponible un ejemplo que demuestra este patrón en [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting).
- [Patrón Valet Key](valet-key.md). Si se supone que los recursos de destino no van a estar disponibles para los usuarios anónimos, es necesario implementar la seguridad sobre el almacén que alberga el contenido estático. En este artículo se describe el uso de un token o una clave que proporciona a los clientes acceso directo restringido a un recurso o servicio específico, como un servicio de almacenamiento hospedado en la nube.
- [Una manera eficaz de implementar un sitio web estático en Azure](http://www.infosysblogs.com/microsoft/2010/06/an_efficient_way_of_deploying.html) en el blog de Infosys.
- [Conceptos de Blob service](https://msdn.microsoft.com/library/azure/dd179376.aspx)
