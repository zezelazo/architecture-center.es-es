---
title: Almacenamiento en caché de tokens de acceso en una aplicación multiinquilino
description: Almacenamiento en caché de tokens de acceso utilizados para invocar una API web de back-end
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: web-api
pnp.series.next: adfs
ms.openlocfilehash: cffc15686ef9d77fafb40982efdbcd4a79f5aaf2
ms.sourcegitcommit: 8ab30776e0c4cdc16ca0dcc881960e3108ad3e94
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/08/2017
ms.locfileid: "26359245"
---
# <a name="cache-access-tokens"></a><span data-ttu-id="f1412-103">Almacenamiento en caché de los tokens de acceso</span><span class="sxs-lookup"><span data-stu-id="f1412-103">Cache access tokens</span></span>

<span data-ttu-id="f1412-104">[![GitHub](../_images/github.png) Código de ejemplo][sample application]</span><span class="sxs-lookup"><span data-stu-id="f1412-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="f1412-105">Es relativamente costoso obtener un token acceso de OAuth, porque requiere una solicitud HTTP al punto de conexión del token.</span><span class="sxs-lookup"><span data-stu-id="f1412-105">It's relatively expensive to get an OAuth access token, because it requires an HTTP request to the token endpoint.</span></span> <span data-ttu-id="f1412-106">Por lo tanto, es bueno almacenar tokens en caché siempre que sea posible.</span><span class="sxs-lookup"><span data-stu-id="f1412-106">Therefore, it's good to cache tokens whenever possible.</span></span> <span data-ttu-id="f1412-107">La [biblioteca de autenticación de Azure AD][ADAL] almacena automáticamente en caché los tokens obtenidos de Azure AD, incluidos los tokens de actualización.</span><span class="sxs-lookup"><span data-stu-id="f1412-107">The [Azure AD Authentication Library][ADAL] (ADAL)  automatically caches tokens obtained from Azure AD, including refresh tokens.</span></span>

<span data-ttu-id="f1412-108">ADAL proporciona una implementación de caché de tokens predeterminada.</span><span class="sxs-lookup"><span data-stu-id="f1412-108">ADAL provides a default token cache implementation.</span></span> <span data-ttu-id="f1412-109">Sin embargo, esta caché de tokens está diseñada para aplicaciones cliente nativas y **no** es adecuada para aplicaciones web:</span><span class="sxs-lookup"><span data-stu-id="f1412-109">However, this token cache is intended for native client apps, and is **not** suitable for web apps:</span></span>

* <span data-ttu-id="f1412-110">Es una instancia estática y no es segura para subprocesos.</span><span class="sxs-lookup"><span data-stu-id="f1412-110">It is a static instance, and not thread safe.</span></span>
* <span data-ttu-id="f1412-111">No se escala a un gran número de usuarios, ya que los tokens de todos los usuarios van en el mismo diccionario.</span><span class="sxs-lookup"><span data-stu-id="f1412-111">It doesn't scale to large numbers of users, because tokens from all users go into the same dictionary.</span></span>
* <span data-ttu-id="f1412-112">No se puede compartir entre servidores web en una granja.</span><span class="sxs-lookup"><span data-stu-id="f1412-112">It can't be shared across web servers in a farm.</span></span>

<span data-ttu-id="f1412-113">En su lugar, debe implementar una caché de tokens personalizada que se derive de la clase `TokenCache` de ADAL pero que sea adecuada para un entorno de servidor y proporcione el nivel de aislamiento deseable entre los tokens de los diferentes usuarios.</span><span class="sxs-lookup"><span data-stu-id="f1412-113">Instead, you should implement a custom token cache that derives from the ADAL `TokenCache` class but is suitable for a server environment and provides the desirable level of isolation between tokens for different users.</span></span>

<span data-ttu-id="f1412-114">La clase `TokenCache` almacena un diccionario de tokens, indexado por emisor, recurso, identificador de cliente y usuario.</span><span class="sxs-lookup"><span data-stu-id="f1412-114">The `TokenCache` class stores a dictionary of tokens, indexed by issuer, resource, client ID, and user.</span></span> <span data-ttu-id="f1412-115">Una caché de tokens personalizada debe escribir este diccionario en una memoria auxiliar, como una caché en Redis.</span><span class="sxs-lookup"><span data-stu-id="f1412-115">A custom token cache should write this dictionary to a backing store, such as a Redis cache.</span></span>

<span data-ttu-id="f1412-116">En la aplicación Tailspin Survey, la clase `DistributedTokenCache` implementa la caché del token.</span><span class="sxs-lookup"><span data-stu-id="f1412-116">In the Tailspin Surveys application, the `DistributedTokenCache` class implements the token cache.</span></span> <span data-ttu-id="f1412-117">Esta implementación usa la abstracción [IDistributedCache][distributed-cache] de ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="f1412-117">This implementation uses the [IDistributedCache][distributed-cache] abstraction from ASP.NET Core.</span></span> <span data-ttu-id="f1412-118">De este modo, cualquier implementación `IDistributedCache` se puede usar como memoria auxiliar.</span><span class="sxs-lookup"><span data-stu-id="f1412-118">That way, any `IDistributedCache` implementation can be used as a backing store.</span></span>

* <span data-ttu-id="f1412-119">De forma predeterminada, la aplicación Surveys utiliza una caché en Redis.</span><span class="sxs-lookup"><span data-stu-id="f1412-119">By default, the Surveys app uses a Redis cache.</span></span>
* <span data-ttu-id="f1412-120">Para un servidor web de instancia única, podría utilizar ASP.NET Core [en la caché en memoria][in-memory-cache].</span><span class="sxs-lookup"><span data-stu-id="f1412-120">For a single-instance web server, you could use the ASP.NET Core [in-memory cache][in-memory-cache].</span></span> <span data-ttu-id="f1412-121">(Esto también es una buena opción para ejecutar la aplicación de forma local durante el desarrollo).</span><span class="sxs-lookup"><span data-stu-id="f1412-121">(This is also a good option for running the app locally during development.)</span></span>

<span data-ttu-id="f1412-122">`DistributedTokenCache` almacena los datos de la memoria caché como pares de clave/valor en la memoria auxiliar.</span><span class="sxs-lookup"><span data-stu-id="f1412-122">`DistributedTokenCache` stores the cache data as key/value pairs in the backing store.</span></span> <span data-ttu-id="f1412-123">La clave es el identificador de usuario más el identificador de cliente, por lo que la memoria auxiliar contiene datos de caché independientes para cada combinación única de usuario/cliente.</span><span class="sxs-lookup"><span data-stu-id="f1412-123">The key is the user ID plus client ID, so the backing store holds separate cache data for each unique combination of user/client.</span></span>

![Memoria caché de tokens](./images/token-cache.png)

<span data-ttu-id="f1412-125">Las particiones de la memoria auxiliar las realiza el usuario.</span><span class="sxs-lookup"><span data-stu-id="f1412-125">The backing store is partitioned by user.</span></span> <span data-ttu-id="f1412-126">Para cada solicitud HTTP, los tokens de ese usuario se leen desde la memoria auxiliar y se cargan en el diccionario `TokenCache` .</span><span class="sxs-lookup"><span data-stu-id="f1412-126">For each HTTP request, the tokens for that user are read from the backing store and loaded into the `TokenCache` dictionary.</span></span> <span data-ttu-id="f1412-127">Si Redis se utiliza como memoria auxiliar, cada una de las instancias de servidor de una granja de servidores lee/escribe en la misma caché, y este enfoque se escala a muchos usuarios.</span><span class="sxs-lookup"><span data-stu-id="f1412-127">If Redis is used as the backing store, every server instance in a server farm reads/writes to the same cache, and this approach scales to many users.</span></span>

## <a name="encrypting-cached-tokens"></a><span data-ttu-id="f1412-128">Cifrado de tokens almacenados en caché</span><span class="sxs-lookup"><span data-stu-id="f1412-128">Encrypting cached tokens</span></span>
<span data-ttu-id="f1412-129">Los tokens son datos confidenciales, ya que conceden acceso a los recursos de un usuario.</span><span class="sxs-lookup"><span data-stu-id="f1412-129">Tokens are sensitive data, because they grant access to a user's resources.</span></span> <span data-ttu-id="f1412-130">(Además, a diferencia de la contraseña del usuario, no puede almacenar simplemente un hash del token). Por lo tanto, es fundamental proteger los tokens para que no se vean comprometidos.</span><span class="sxs-lookup"><span data-stu-id="f1412-130">(Moreover, unlike a user's password, you can't just store a hash of the token.) Therefore, it's critical to protect tokens from being compromised.</span></span> <span data-ttu-id="f1412-131">La memoria caché con respaldo de Redis está protegida por una contraseña, pero si un usuario la obtiene, podría conseguir todos los tokens de acceso almacenados en caché.</span><span class="sxs-lookup"><span data-stu-id="f1412-131">The Redis-backed cache is protected by a password, but if someone obtains the password, they could get all of the cached access tokens.</span></span> <span data-ttu-id="f1412-132">Por esta razón, `DistributedTokenCache` cifra todo lo que escribe en la memoria auxiliar.</span><span class="sxs-lookup"><span data-stu-id="f1412-132">For that reason, the `DistributedTokenCache` encrypts everything that it writes to the backing store.</span></span> <span data-ttu-id="f1412-133">El cifrado se realiza mediante las API de [protección de datos][data-protection] de ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="f1412-133">Encryption is done using the ASP.NET Core [data protection][data-protection] APIs.</span></span>

> [!NOTE]
> <span data-ttu-id="f1412-134">Si implementa en sitios web de Azure, se hace una copia de seguridad de las claves de cifrado en el almacenamiento de red y se sincronizan en todos los equipos (vea [Duración y administración de claves][key-management]).</span><span class="sxs-lookup"><span data-stu-id="f1412-134">If you deploy to Azure Web Sites, the encryption keys are backed up to network storage and synchronized across all machines (see [Key management and lifetime][key-management]).</span></span> <span data-ttu-id="f1412-135">De forma predeterminada, las claves no se cifran cuando se ejecutan en sitios web de Azure, pero puede [habilitar el cifrado mediante un certificado X.509][x509-cert-encryption].</span><span class="sxs-lookup"><span data-stu-id="f1412-135">By default, keys are not encrypted when running in Azure Web Sites, but you can [enable encryption using an X.509 certificate][x509-cert-encryption].</span></span>
> 
> 

## <a name="distributedtokencache-implementation"></a><span data-ttu-id="f1412-136">Implementación de DistributedTokenCache</span><span class="sxs-lookup"><span data-stu-id="f1412-136">DistributedTokenCache implementation</span></span>
<span data-ttu-id="f1412-137">La clase `DistributedTokenCache` se deriva de la clase [TokenCache][tokencache-class] de la biblioteca de autenticación de Azure AD.</span><span class="sxs-lookup"><span data-stu-id="f1412-137">The `DistributedTokenCache` class derives from the ADAL [TokenCache][tokencache-class] class.</span></span>

<span data-ttu-id="f1412-138">En el constructor, la clase `DistributedTokenCache` crea una clave para el usuario actual y la carga la memoria caché desde la memoria auxiliar:</span><span class="sxs-lookup"><span data-stu-id="f1412-138">In the constructor, the `DistributedTokenCache` class creates a key for the current user and loads the cache from the backing store:</span></span>

```csharp
public DistributedTokenCache(
    ClaimsPrincipal claimsPrincipal,
    IDistributedCache distributedCache,
    ILoggerFactory loggerFactory,
    IDataProtectionProvider dataProtectionProvider)
    : base()
{
    _claimsPrincipal = claimsPrincipal;
    _cacheKey = BuildCacheKey(_claimsPrincipal);
    _distributedCache = distributedCache;
    _logger = loggerFactory.CreateLogger<DistributedTokenCache>();
    _protector = dataProtectionProvider.CreateProtector(typeof(DistributedTokenCache).FullName);
    AfterAccess = AfterAccessNotification;
    LoadFromCache();
}
```

<span data-ttu-id="f1412-139">La clave se crea concatenando el identificador de usuario y el identificador de cliente.</span><span class="sxs-lookup"><span data-stu-id="f1412-139">The key is created by concatenating the user ID and client ID.</span></span> <span data-ttu-id="f1412-140">Ambos se toman de las notificaciones encontradas en `ClaimsPrincipal`del usuario:</span><span class="sxs-lookup"><span data-stu-id="f1412-140">Both of these are taken from claims found in the user's `ClaimsPrincipal`:</span></span>

```csharp
private static string BuildCacheKey(ClaimsPrincipal claimsPrincipal)
{
    string clientId = claimsPrincipal.FindFirstValue("aud", true);
    return string.Format(
        "UserId:{0}::ClientId:{1}",
        claimsPrincipal.GetObjectIdentifierValue(),
        clientId);
}
```

<span data-ttu-id="f1412-141">Para cargar los datos de la memoria caché, lea el blob serializado desde la memoria auxiliar y llame a `TokenCache.Deserialize` para convertir el blob en datos de memoria caché.</span><span class="sxs-lookup"><span data-stu-id="f1412-141">To load the cache data, read the serialized blob from the backing store, and call `TokenCache.Deserialize` to convert the blob into cache data.</span></span>

```csharp
private void LoadFromCache()
{
    byte[] cacheData = _distributedCache.Get(_cacheKey);
    if (cacheData != null)
    {
        this.Deserialize(_protector.Unprotect(cacheData));
    }
}
```

<span data-ttu-id="f1412-142">Cada vez que ADAL acceda a la memoria caché, se activará un evento `AfterAccess` .</span><span class="sxs-lookup"><span data-stu-id="f1412-142">Whenever ADAL access the cache, it fires an `AfterAccess` event.</span></span> <span data-ttu-id="f1412-143">Si han cambiado los datos de la memoria caché, la propiedad `HasStateChanged` es true.</span><span class="sxs-lookup"><span data-stu-id="f1412-143">If the cache data has changed, the `HasStateChanged` property is true.</span></span> <span data-ttu-id="f1412-144">En ese caso, actualice la memoria auxiliar para reflejar el cambio y, a continuación, establezca `HasStateChanged` en false.</span><span class="sxs-lookup"><span data-stu-id="f1412-144">In that case, update the backing store to reflect the change, and then set `HasStateChanged` to false.</span></span>

```csharp
public void AfterAccessNotification(TokenCacheNotificationArgs args)
{
    if (this.HasStateChanged)
    {
        try
        {
            if (this.Count > 0)
            {
                _distributedCache.Set(_cacheKey, _protector.Protect(this.Serialize()));
            }
            else
            {
                // There are no tokens for this user/client, so remove the item from the cache.
                _distributedCache.Remove(_cacheKey);
            }
            this.HasStateChanged = false;
        }
        catch (Exception exp)
        {
            _logger.WriteToCacheFailed(exp);
            throw;
        }
    }
}
```

<span data-ttu-id="f1412-145">TokenCache envía otros dos eventos:</span><span class="sxs-lookup"><span data-stu-id="f1412-145">TokenCache sends two other events:</span></span>

* <span data-ttu-id="f1412-146">`BeforeWrite`.</span><span class="sxs-lookup"><span data-stu-id="f1412-146">`BeforeWrite`.</span></span> <span data-ttu-id="f1412-147">Se le llama inmediatamente antes de que ADAL escriba en la memoria caché.</span><span class="sxs-lookup"><span data-stu-id="f1412-147">Called immediately before ADAL writes to the cache.</span></span> <span data-ttu-id="f1412-148">Puede utilizar esto para implementar una estrategia de simultaneidad</span><span class="sxs-lookup"><span data-stu-id="f1412-148">You can use this to implement a concurrency strategy</span></span>
* <span data-ttu-id="f1412-149">`BeforeAccess`.</span><span class="sxs-lookup"><span data-stu-id="f1412-149">`BeforeAccess`.</span></span> <span data-ttu-id="f1412-150">Se le llama inmediatamente antes de que ADAL lea desde la memoria caché.</span><span class="sxs-lookup"><span data-stu-id="f1412-150">Called immediately before ADAL reads from the cache.</span></span> <span data-ttu-id="f1412-151">Aquí puede volver a cargar la memoria caché para obtener la versión más reciente.</span><span class="sxs-lookup"><span data-stu-id="f1412-151">Here you can reload the cache to get the latest version.</span></span>

<span data-ttu-id="f1412-152">En nuestro caso, decidimos no administrar estos dos eventos.</span><span class="sxs-lookup"><span data-stu-id="f1412-152">In our case, we decided not to handle these two events.</span></span>

* <span data-ttu-id="f1412-153">Para la simultaneidad, la última escritura tiene prioridad.</span><span class="sxs-lookup"><span data-stu-id="f1412-153">For concurrency, last write wins.</span></span> <span data-ttu-id="f1412-154">Eso está bien, porque los tokens se almacenan por separado para cada usuario + cliente, por lo que solo podría ocurrir un conflicto si el mismo usuario tuviera dos inicios de sesión abiertos simultáneamente.</span><span class="sxs-lookup"><span data-stu-id="f1412-154">That's OK, because tokens are stored independently for each user + client, so a conflict would only happen if the same user had two concurrent login sessions.</span></span>
* <span data-ttu-id="f1412-155">En el caso de la lectura, cargamos la memoria caché en cada solicitud.</span><span class="sxs-lookup"><span data-stu-id="f1412-155">For reading, we load the cache on every request.</span></span> <span data-ttu-id="f1412-156">Las solicitudes tienen poca vigencia.</span><span class="sxs-lookup"><span data-stu-id="f1412-156">Requests are short lived.</span></span> <span data-ttu-id="f1412-157">Si se modifica la memoria caché en ese tiempo, la siguiente solicitud recogerá el nuevo valor.</span><span class="sxs-lookup"><span data-stu-id="f1412-157">If the cache gets modified in that time, the next request will pick up the new value.</span></span>

<span data-ttu-id="f1412-158">[**Siguiente**][client-assertion]</span><span class="sxs-lookup"><span data-stu-id="f1412-158">[**Next**][client-assertion]</span></span>

<!-- links -->
[ADAL]: https://msdn.microsoft.com/library/azure/jj573266.aspx
[client-assertion]: ./client-assertion.md
[data-protection]: /aspnet/core/security/data-protection/
[distributed-cache]: /aspnet/core/performance/caching/distributed
[key-management]: /aspnet/core/security/data-protection/configuration/default-settings
[in-memory-cache]: /aspnet/core/performance/caching/memory
[tokencache-class]: https://msdn.microsoft.com/library/azure/microsoft.identitymodel.clients.activedirectory.tokencache.aspx
[x509-cert-encryption]: /aspnet/core/security/data-protection/implementation/key-encryption-at-rest#x509-certificate
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
