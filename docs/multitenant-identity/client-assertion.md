---
title: "Uso de aserción de cliente para obtener tokens de acceso de Azure AD"
description: "Uso de aserción de cliente para obtener tokens de acceso de Azure AD."
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: adfs
pnp.series.next: key-vault
ms.openlocfilehash: 9fe1ee2ec5a540edc41c3a310476507f8d862f0c
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2017
---
# <a name="use-client-assertion-to-get-access-tokens-from-azure-ad"></a><span data-ttu-id="ce9aa-103">Uso de aserción de cliente para obtener tokens de acceso de Azure AD</span><span class="sxs-lookup"><span data-stu-id="ce9aa-103">Use client assertion to get access tokens from Azure AD</span></span>

<span data-ttu-id="ce9aa-104">[![GitHub](../_images/github.png) Código de ejemplo][sample application]</span><span class="sxs-lookup"><span data-stu-id="ce9aa-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

## <a name="background"></a><span data-ttu-id="ce9aa-105">Fondo</span><span class="sxs-lookup"><span data-stu-id="ce9aa-105">Background</span></span>
<span data-ttu-id="ce9aa-106">Al usar el flujo de código de autorización o flujo híbrido en OpenID Connect, el cliente intercambia un código de autorización para un token de acceso.</span><span class="sxs-lookup"><span data-stu-id="ce9aa-106">When using authorization code flow or hybrid flow in OpenID Connect, the client exchanges an authorization code for an access token.</span></span> <span data-ttu-id="ce9aa-107">Durante este paso, el cliente tiene que autenticarse en el servidor.</span><span class="sxs-lookup"><span data-stu-id="ce9aa-107">During this step, the client has to authenticate itself to the server.</span></span>

![Secreto del cliente](./images/client-secret.png)

<span data-ttu-id="ce9aa-109">Una manera de autenticar el cliente es mediante un secreto de cliente.</span><span class="sxs-lookup"><span data-stu-id="ce9aa-109">One way to authenticate the client is by using a client secret.</span></span> <span data-ttu-id="ce9aa-110">Así es cómo la aplicación [Surveys de Tailspin][Surveys] está configurada de forma predeterminada.</span><span class="sxs-lookup"><span data-stu-id="ce9aa-110">That's how the [Tailspin Surveys][Surveys] application is configured by default.</span></span>

<span data-ttu-id="ce9aa-111">Esta es una solicitud de ejemplo desde el cliente al IDP, que solicita un token de acceso.</span><span class="sxs-lookup"><span data-stu-id="ce9aa-111">Here is an example request from the client to the IDP, requesting an access token.</span></span> <span data-ttu-id="ce9aa-112">Tenga en cuenta el parámetro `client_secret` .</span><span class="sxs-lookup"><span data-stu-id="ce9aa-112">Note the `client_secret` parameter.</span></span>

```
POST https://login.microsoftonline.com/b9bd2162xxx/oauth2/token HTTP/1.1
Content-Type: application/x-www-form-urlencoded

resource=https://tailspin.onmicrosoft.com/surveys.webapi
  &client_id=87df91dc-63de-4765-8701-b59cc8bd9e11
  &client_secret=i3Bf12Dn...
  &grant_type=authorization_code
  &code=PG8wJG6Y...
```

<span data-ttu-id="ce9aa-113">El secreto es una cadena, por lo que debe asegurarse de no perder el valor.</span><span class="sxs-lookup"><span data-stu-id="ce9aa-113">The secret is just a string, so you have to make sure not to leak the value.</span></span> <span data-ttu-id="ce9aa-114">La práctica recomendada es mantener el secreto de cliente fuera del control de código fuente.</span><span class="sxs-lookup"><span data-stu-id="ce9aa-114">The best practice is to keep the client secret out of source control.</span></span> <span data-ttu-id="ce9aa-115">Cuando se implemente en Azure, almacene el secreto en un [valor de configuración de la aplicación][configure-web-app].</span><span class="sxs-lookup"><span data-stu-id="ce9aa-115">When you deploy to Azure, store the secret in an [app setting][configure-web-app].</span></span>

<span data-ttu-id="ce9aa-116">Sin embargo, cualquier persona con acceso a la suscripción de Azure puede ver la configuración de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="ce9aa-116">However, anyone with access to the Azure subscription can view the app settings.</span></span> <span data-ttu-id="ce9aa-117">Además, siempre existe la tentación de proteger los secretos en el control de código fuente (por ejemplo, en los scripts de implementación), compartirlos por correo electrónico, etc.</span><span class="sxs-lookup"><span data-stu-id="ce9aa-117">Further, there is always a temptation to check secrets into source control (e.g., in deployment scripts), share them by email, and so on.</span></span>

<span data-ttu-id="ce9aa-118">Para obtener seguridad adicional, puede usar [aserción de cliente] en lugar de un secreto de cliente.</span><span class="sxs-lookup"><span data-stu-id="ce9aa-118">For additional security, you can use [client assertion] instead of a client secret.</span></span> <span data-ttu-id="ce9aa-119">Con aserción de cliente, el cliente utiliza un certificado X.509 para demostrar que la solicitud de token procedía del cliente.</span><span class="sxs-lookup"><span data-stu-id="ce9aa-119">With client assertion, the client uses an X.509 certificate to prove the token request came from the client.</span></span> <span data-ttu-id="ce9aa-120">El certificado de cliente se instala en el servidor web.</span><span class="sxs-lookup"><span data-stu-id="ce9aa-120">The client certificate is installed on the web server.</span></span> <span data-ttu-id="ce9aa-121">Por lo general, será más fácil restringir el acceso al certificado que garantizar que nadie revela inadvertidamente un secreto de cliente.</span><span class="sxs-lookup"><span data-stu-id="ce9aa-121">Generally, it will be easier to restrict access to the certificate, than to ensure that nobody inadvertently reveals a client secret.</span></span> <span data-ttu-id="ce9aa-122">Para más información sobre cómo configurar certificados en una aplicación web, consulte [Using Certificates in Azure Websites Applications][using-certs-in-websites] (Uso de certificados en aplicaciones de Azure Websites).</span><span class="sxs-lookup"><span data-stu-id="ce9aa-122">For more information about configuring certificates in a web app, see [Using Certificates in Azure Websites Applications][using-certs-in-websites]</span></span>

<span data-ttu-id="ce9aa-123">A continuación figura una solicitud de token utilizando aserción de cliente:</span><span class="sxs-lookup"><span data-stu-id="ce9aa-123">Here is a token request using client assertion:</span></span>

```
POST https://login.microsoftonline.com/b9bd2162xxx/oauth2/token HTTP/1.1
Content-Type: application/x-www-form-urlencoded

resource=https://tailspin.onmicrosoft.com/surveys.webapi
  &client_id=87df91dc-63de-4765-8701-b59cc8bd9e11
  &client_assertion_type=urn:ietf:params:oauth:client-assertion-type:jwt-bearer
  &client_assertion=eyJhbGci...
  &grant_type=authorization_code
  &code= PG8wJG6Y...
```

<span data-ttu-id="ce9aa-124">Observe que el parámetro `client_secret` ya no se usa.</span><span class="sxs-lookup"><span data-stu-id="ce9aa-124">Notice that the `client_secret` parameter is no longer used.</span></span> <span data-ttu-id="ce9aa-125">En su lugar, el parámetro `client_assertion` contiene un token de JWT que se firmó con el certificado de cliente.</span><span class="sxs-lookup"><span data-stu-id="ce9aa-125">Instead, the `client_assertion` parameter contains a JWT token that was signed using the client certificate.</span></span> <span data-ttu-id="ce9aa-126">El parámetro `client_assertion_type` especifica el tipo de aserción &mdash; en este caso, token de JWT.</span><span class="sxs-lookup"><span data-stu-id="ce9aa-126">The `client_assertion_type` parameter specifies the type of assertion &mdash; in this case, JWT token.</span></span> <span data-ttu-id="ce9aa-127">El servidor valida el token de JWT.</span><span class="sxs-lookup"><span data-stu-id="ce9aa-127">The server validates the JWT token.</span></span> <span data-ttu-id="ce9aa-128">Si el token de JWT no es válido, la solicitud de token devuelve un error.</span><span class="sxs-lookup"><span data-stu-id="ce9aa-128">If the JWT token is invalid, the token request returns an error.</span></span>

> [!NOTE]
> <span data-ttu-id="ce9aa-129">Los certificados X.509 no son la única forma de aserción de cliente; nos centraremos en ella aquí porque es compatible con Azure AD.</span><span class="sxs-lookup"><span data-stu-id="ce9aa-129">X.509 certificates are not the only form of client assertion; we focus on it here because it is supported by Azure AD.</span></span>
> 
> 

<span data-ttu-id="ce9aa-130">En tiempo de ejecución, la aplicación web lee el certificado del almacén de certificados.</span><span class="sxs-lookup"><span data-stu-id="ce9aa-130">At run time, the web application reads the certificate from the certificate store.</span></span> <span data-ttu-id="ce9aa-131">El certificado debe estar instalado en la misma máquina que la aplicación web.</span><span class="sxs-lookup"><span data-stu-id="ce9aa-131">The certificate must be installed on the same machine as the web app.</span></span>

<span data-ttu-id="ce9aa-132">La aplicación Surveys incluye una clase auxiliar que crea un elemento [ClientAssertionCertificate](/dotnet/api/microsoft.identitymodel.clients.activedirectory.clientassertioncertificate) que puede pasar al método [AuthenticationContext.AcquireTokenSilentAsync](/dotnet/api/microsoft.identitymodel.clients.activedirectory.authenticationcontext.acquiretokensilentasync) para adquirir un token de Azure AD.</span><span class="sxs-lookup"><span data-stu-id="ce9aa-132">The Surveys application includes a helper class that creates a [ClientAssertionCertificate](/dotnet/api/microsoft.identitymodel.clients.activedirectory.clientassertioncertificate) that you can pass to the [AuthenticationContext.AcquireTokenSilentAsync](/dotnet/api/microsoft.identitymodel.clients.activedirectory.authenticationcontext.acquiretokensilentasync) method to acquire a token from Azure AD.</span></span>

```csharp
public class CertificateCredentialService : ICredentialService
{
    private Lazy<Task<AdalCredential>> _credential;

    public CertificateCredentialService(IOptions<ConfigurationOptions> options)
    {
        var aadOptions = options.Value?.AzureAd;
        _credential = new Lazy<Task<AdalCredential>>(() =>
        {
            X509Certificate2 cert = CertificateUtility.FindCertificateByThumbprint(
                aadOptions.Asymmetric.StoreName,
                aadOptions.Asymmetric.StoreLocation,
                aadOptions.Asymmetric.CertificateThumbprint,
                aadOptions.Asymmetric.ValidationRequired);
            string password = null;
            var certBytes = CertificateUtility.ExportCertificateWithPrivateKey(cert, out password);
            return Task.FromResult(new AdalCredential(new ClientAssertionCertificate(aadOptions.ClientId, new X509Certificate2(certBytes, password))));
        });
    }

    public async Task<AdalCredential> GetCredentialsAsync()
    {
        return await _credential.Value;
    }
}
```

<span data-ttu-id="ce9aa-133">Para información sobre cómo configurar la aserción de cliente en la aplicación Surveys, consulte [Use Azure Key Vault to protect application secrets ][key vault] (Uso de Azure Key Vault para proteger los secretos de aplicación).</span><span class="sxs-lookup"><span data-stu-id="ce9aa-133">For information about setting up client assertion in the Surveys application, see [Use Azure Key Vault to protect application secrets ][key vault].</span></span>

<span data-ttu-id="ce9aa-134">[**Siguiente**][key vault]</span><span class="sxs-lookup"><span data-stu-id="ce9aa-134">[**Next**][key vault]</span></span>

<!-- Links -->
[configure-web-app]: /azure/app-service-web/web-sites-configure/
[azure-management-portal]: https://portal.azure.com
[aserción de cliente]: https://tools.ietf.org/html/rfc7521
[key vault]: key-vault.md
[Setup-KeyVault]: https://github.com/mspnp/multitenant-saas-guidance/blob/master/scripts/Setup-KeyVault.ps1
[Surveys]: tailspin.md
[using-certs-in-websites]: https://azure.microsoft.com/blog/using-certificates-in-azure-websites-applications/

[sample application]: https://github.com/mspnp/multitenant-saas-guidance
