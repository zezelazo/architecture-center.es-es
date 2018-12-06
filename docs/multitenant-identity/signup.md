---
title: Registro e incorporación de inquilinos en aplicaciones multiinquilino
description: Incorporación de inquilinos en una aplicación multiinquilino
author: MikeWasson
ms.date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: claims
pnp.series.next: app-roles
ms.openlocfilehash: 541a4dd9abb2168eef4a60a0ec99e1e7c06049b5
ms.sourcegitcommit: e7e0e0282fa93f0063da3b57128ade395a9c1ef9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/05/2018
ms.locfileid: "52902483"
---
# <a name="tenant-sign-up-and-onboarding"></a>Registro e incorporación de inquilinos

[![GitHub](../_images/github.png) Código de ejemplo][sample application]

En este artículo se describe cómo implementar un proceso de *registro* en una aplicación multiinquilino, lo que permite a un cliente registrar su propia organización en la aplicación.
Hay varias razones que justifican la implementación de un proceso de registro:

* Permitir que un administrador de AD dé su consentimiento para que la toda la organización del cliente utilice la aplicación.
* Recopilar del cliente información de pago con tarjeta de crédito o de otra índole.
* Realizar las instalaciones únicas por inquilino necesarias para la aplicación.

## <a name="admin-consent-and-azure-ad-permissions"></a>Consentimiento de administrador y permisos de Azure AD
Para realizar la autenticación con Azure AD, la aplicación necesita acceso al directorio del usuario. Como mínimo, la aplicación necesita permiso para leer el perfil del usuario. La primera vez que un usuario inicie sesión, Azure AD mostrará una página de consentimiento en la que figuran los permisos que se solicitan. Al hacer clic en **Aceptar**, el usuario concede el permiso a la aplicación.

De forma predeterminada, se concede consentimiento para cada usuario de manera individualizada. Todo usuario que inicie sesión verá la página de consentimiento. Sin embargo, Azure AD admite también el *consentimiento del administrador*, que permite a un administrador de AD dar su consentimiento para toda la organización.

Cuando se usa el flujo de consentimiento del administrador, la página de consentimiento indica que el administrador de AD concede permiso en nombre de todo el inquilino:

![Petición de consentimiento del administrador](./images/admin-consent.png)

Una vez que el administrador haga clic en **Aceptar**, otros usuarios dentro del mismo inquilino podrán iniciar sesión y Azure AD omitirá la pantalla de consentimiento.

Únicamente un administrador de AD puede dar consentimiento de administrador, puesto que concede permiso en nombre de toda la organización. Si un usuario no administrador intenta autenticarse con el flujo del consentimiento de administrador, Azure AD mostrará un error:

![Error de consentimiento](./images/consent-error.png)

Si la aplicación requiere permisos adicionales en un momento posterior, el cliente deberá registrarse de nuevo y dar su consentimiento a los permisos actualizados.  

## <a name="implementing-tenant-sign-up"></a>Implementación del registro de inquilino
En la aplicación [Surveys de Tailspin][Tailspin],  se han definido varios requisitos para el proceso de suscripción:

* Un inquilino debe registrarse antes de que los usuarios puedan iniciar sesión.
* El registro emplea el flujo del consentimiento de administrador.
* El registro agrega al inquilino del usuario a la base de datos de la aplicación.
* Una vez que se registre un inquilino, la página mostrará una página de incorporación.

En esta sección, le guiaremos a través de la implementación del proceso de registro.
Es importante comprender que "registrarse" e "iniciar sesión" son conceptos de la aplicación. Durante el flujo de autenticación, Azure AD no puede saber si el usuario está en un proceso de registro. Corresponde a la aplicación controlar el contexto.

Cuando un usuario anónimo visita la aplicación Surveys, se le presentan dos botones: uno para iniciar sesión y otro para inscribir a su empresa (registrarla).

![Página de registro de la aplicación](./images/sign-up-page.png)

Estos botones invocan a acciones de la clase `AccountController`.

La acción `SignIn` devuelve un valor **ChallegeResult**que hace que el software intermedio OpenID Connect redirija al punto de conexión de autenticación. Esta es la manera predeterminada de desencadenar la autenticación en ASP.NET Core.  

```csharp
[AllowAnonymous]
public IActionResult SignIn()
{
    return new ChallengeResult(
        OpenIdConnectDefaults.AuthenticationScheme,
        new AuthenticationProperties
        {
            IsPersistent = true,
            RedirectUri = Url.Action("SignInCallback", "Account")
        });
}
```

Ahora compare la acción `SignUp` :

```csharp
[AllowAnonymous]
public IActionResult SignUp()
{
    var state = new Dictionary<string, string> { { "signup", "true" }};
    return new ChallengeResult(
        OpenIdConnectDefaults.AuthenticationScheme,
        new AuthenticationProperties(state)
        {
            RedirectUri = Url.Action(nameof(SignUpCallback), "Account")
        });
}
```

Al igual que `SignIn`, la acción `SignUp` también devuelve `ChallengeResult`. Pero esta vez, agregamos una parte de la información de estado para `AuthenticationProperties` en `ChallengeResult`:

* signup: una marca booleana que indica que el usuario ha iniciado el proceso de registro.

La información de estado de `AuthenticationProperties` se agregará al parámetro [state] de OpenID Connect, que va y viene durante el flujo de autenticación.

![Parámetro de estado](./images/state-parameter.png)

Cuando el usuario se autentica en Azure AD y se le redirige de nuevo a la aplicación, el vale de autenticación contiene el estado. Estamos usando este hecho para asegurarnos de que el valor "signup" se conserva en todo el flujo de autenticación.

## <a name="adding-the-admin-consent-prompt"></a>Incorporación de la petición de consentimiento del administrador
En Azure AD, el flujo del consentimiento de administración se activa mediante la adición de un parámetro "prompt" a la cadena de consulta de la solicitud de autenticación:

```
/authorize?prompt=admin_consent&...
```

La aplicación Surveys agrega la solicitud durante el evento `RedirectToAuthenticationEndpoint` . A este evento se le llama justo antes de que el software intermedio redirija al punto de conexión de autenticación.

```csharp
public override Task RedirectToAuthenticationEndpoint(RedirectContext context)
{
    if (context.IsSigningUp())
    {
        context.ProtocolMessage.Prompt = "admin_consent";
    }

    _logger.RedirectToIdentityProvider();
    return Task.FromResult(0);
}
```

Al establecer` ProtocolMessage.Prompt` , se indica al software intermedio que agregue el parámetro "prompt" a la solicitud de autenticación.

Tenga en cuenta que este valor de solicitud solo es necesario durante el registro. El inicio de sesión normal no debe incluirlo. Para distinguir entre ellos, se comprueba el valor `signup` en el estado de autenticación. El siguiente método de extensión comprueba esta condición:

```csharp
internal static bool IsSigningUp(this BaseControlContext context)
{
    Guard.ArgumentNotNull(context, nameof(context));

    string signupValue;
    // Check the HTTP context and convert to string
    if ((context.Ticket == null) ||
        (!context.Ticket.Properties.Items.TryGetValue("signup", out signupValue)))
    {
        return false;
    }

    // We have found the value, so see if it's valid
    bool isSigningUp;
    if (!bool.TryParse(signupValue, out isSigningUp))
    {
        // The value for signup is not a valid boolean, throw                
        throw new InvalidOperationException($"'{signupValue}' is an invalid boolean value");
    }

    return isSigningUp;
}
```

## <a name="registering-a-tenant"></a>Registro de un inquilino
La aplicación Surveys almacena parte de la información sobre cada inquilino y usuario en la base de datos de la aplicación.

![Tabla de inquilinos](./images/tenant-table.png)

En la tabla Tenant, IssuerValue es el valor de la notificación del emisor del inquilino. En el caso de Azure AD, se trata de `https://sts.windows.net/<tentantID>` y proporciona un valor único por inquilino.

Cuando se suscribe un nuevo inquilino, la aplicación Surveys escribe un registro del inquilino en la base de datos. Esto sucede dentro del evento `AuthenticationValidated`. (no lo haga antes de este evento, porque el token del identificador aún no estará validado, por lo que no podrá confiar en los valores de notificación. Consulte [Autenticación].

Este es el código relevante de la aplicación Surveys:

```csharp
public override async Task TokenValidated(TokenValidatedContext context)
{
    var principal = context.AuthenticationTicket.Principal;
    var userId = principal.GetObjectIdentifierValue();
    var tenantManager = context.HttpContext.RequestServices.GetService<TenantManager>();
    var userManager = context.HttpContext.RequestServices.GetService<UserManager>();
    var issuerValue = principal.GetIssuerValue();
    _logger.AuthenticationValidated(userId, issuerValue);

    // Normalize the claims first.
    NormalizeClaims(principal);
    var tenant = await tenantManager.FindByIssuerValueAsync(issuerValue)
        .ConfigureAwait(false);

    if (context.IsSigningUp())
    {
        if (tenant == null)
        {
            tenant = await SignUpTenantAsync(context, tenantManager)
                .ConfigureAwait(false);
        }

        // In this case, we need to go ahead and set up the user signing us up.
        await CreateOrUpdateUserAsync(context.Ticket, userManager, tenant)
            .ConfigureAwait(false);
    }
    else
    {
        if (tenant == null)
        {
            _logger.UnregisteredUserSignInAttempted(userId, issuerValue);
            throw new SecurityTokenValidationException($"Tenant {issuerValue} is not registered");
        }

        await CreateOrUpdateUserAsync(context.Ticket, userManager, tenant)
            .ConfigureAwait(false);
    }
}
```

Este código hace lo siguiente:

1. Compruebe si el valor de emisor del inquilino ya está en la base de datos. Si no se ha registrado el inquilino, `FindByIssuerValueAsync` se devuelve null.
2. Si el usuario se está registrando:
   1. Agregue el inquilino a la base de datos (`SignUpTenantAsync`).
   2. Agregue el usuario autenticado a la base de datos (`CreateOrUpdateUserAsync`).
3. En caso contrario, complete el flujo normal de inicio de sesión:
   1. Si no se encontró el emisor del inquilino en la base de datos, el inquilino no está registrado y el cliente debe registrarse. En ese caso, se produce una excepción para provocar un error en la autenticación.
   2. De no ser así, cree un registro de base de datos para este usuario, si no hay todavía ninguno (`CreateOrUpdateUserAsync`).

Este es el método `SignUpTenantAsync` que agrega el inquilino a la base de datos.

```csharp
private async Task<Tenant> SignUpTenantAsync(BaseControlContext context, TenantManager tenantManager)
{
    Guard.ArgumentNotNull(context, nameof(context));
    Guard.ArgumentNotNull(tenantManager, nameof(tenantManager));

    var principal = context.Ticket.Principal;
    var issuerValue = principal.GetIssuerValue();
    var tenant = new Tenant
    {
        IssuerValue = issuerValue,
        Created = DateTimeOffset.UtcNow
    };

    try
    {
        await tenantManager.CreateAsync(tenant)
            .ConfigureAwait(false);
    }
    catch(Exception ex)
    {
        _logger.SignUpTenantFailed(principal.GetObjectIdentifierValue(), issuerValue, ex);
        throw;
    }

    return tenant;
}
```

Este es un resumen del flujo de registro completo en la aplicación Surveys:

1. El usuario hace clic en el **Sign Up** (Registrarse).
2. La acción `AccountController.SignUp` devuelve un resultado de desafío.  El estado de autenticación incluye el valor "signup".
3. En el evento `RedirectToAuthenticationEndpoint`, agregue el mensaje `admin_consent`.
4. El software intermedio OpenID Connect se redirige a Azure AD y el usuario se autentica.
5. En el evento `AuthenticationValidated` , busque el estado de "signup".
6. Agregue el inquilino a la base de datos.

[**Siguiente**][app roles]

<!-- Links -->
[app roles]: app-roles.md
[Tailspin]: tailspin.md

[state]: https://openid.net/specs/openid-connect-core-1_0.html#AuthRequest
[Autenticación]: authenticate.md
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
