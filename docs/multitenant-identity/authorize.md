---
title: "Autorización en aplicaciones multiinquilino"
description: "Realización de la autorización en una aplicación multiinquilino"
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: app-roles
pnp.series.next: web-api
ms.openlocfilehash: 86c308d21f19bb3ac2a4a2240a9a03a504de5cf4
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2017
---
# <a name="role-based-and-resource-based-authorization"></a>Autorización basada en roles y en recursos

[![GitHub](../_images/github.png) Código de ejemplo][sample application]

Nuestra [implementación de referencia] es una aplicación ASP.NET Core. En este artículo se van a ver dos enfoques generales para la autorización, mediante las API de autorización provistas en ASP.NET Core.

* **Autorización basada en roles**. Autorizar una acción basándose en los roles asignados a un usuario. Por ejemplo, algunas acciones requieren un rol de administrador.
* **Autorización basada en recursos**. Autorizar una acción basándose en un recurso determinado. Por ejemplo, cada recurso tiene un propietario. El propietario puede eliminar el recurso; otros usuarios no.

Una aplicación típica empleará una combinación de ambos. Por ejemplo, para eliminar un recurso, el usuario debe ser el propietario del mismo *o* administrador.

## <a name="role-based-authorization"></a>Autorización basada en roles
La aplicación [Tailspin Surveys][Tailspin] define los siguientes roles:

* Administrador. Puede realizar todas las operaciones CRUD en cualquier encuesta que pertenezca a ese inquilino.
* Creador. Puede crear nuevas encuestas.
* Lector. Puede leer cualquier encuesta que pertenezca a ese inquilino.

Los roles se aplican a los *usuarios* de la aplicación. En la aplicación Surveys, un usuario es administrador, creador o lector.

Para ver una discusión de cómo definir y administrar roles, consulte [Application roles (Roles de aplicación)].

Independientemente de cómo administre los roles, el código de autorización tendrá un aspecto similar. ASP.NET Core cuenta con una abstracción llamada [directivas de autorización][policies]. Con esta característica, el usuario define directivas de autorización en el código y después las aplica a las acciones de controlador. La directiva se separa del controlador.

### <a name="create-policies"></a>Creación de directivas
Para definir una directiva, primero cree una clase que implemente `IAuthorizationRequirement`. Es lo más fácil para derivar de `AuthorizationHandler`. En el método `Handle` , examine las notificaciones pertinentes.

A continuación se muestra un ejemplo de la aplicación Tailspin Surveys:

```csharp
public class SurveyCreatorRequirement : AuthorizationHandler<SurveyCreatorRequirement>, IAuthorizationRequirement
{
    protected override Task HandleRequirementAsync(AuthorizationHandlerContext context, SurveyCreatorRequirement requirement)
    {
        if (context.User.HasClaim(ClaimTypes.Role, Roles.SurveyAdmin) || 
            context.User.HasClaim(ClaimTypes.Role, Roles.SurveyCreator))
        {
            context.Succeed(requirement);
        }
        return Task.FromResult(0);
    }
}
```

Esta clase define el requisito para un usuario para crear una nueva encuesta. El usuario debe estar en el rol SurveyAdmin o SurveyCreator.

En la clase de inicio, defina una directiva con nombre que incluya uno o varios requisitos. Si hay varios requisitos, el usuario debe cumplir *todos* ellos para que se le conceda autorización. El código siguiente define dos directivas:

```csharp
services.AddAuthorization(options =>
{
    options.AddPolicy(PolicyNames.RequireSurveyCreator,
        policy =>
        {
            policy.AddRequirements(new SurveyCreatorRequirement());
            policy.RequireAuthenticatedUser(); // Adds DenyAnonymousAuthorizationRequirement 
            // By adding the CookieAuthenticationDefaults.AuthenticationScheme, if an authenticated
            // user is not in the appropriate role, they will be redirected to a "forbidden" page.
            policy.AddAuthenticationSchemes(CookieAuthenticationDefaults.AuthenticationScheme);
        });

    options.AddPolicy(PolicyNames.RequireSurveyAdmin,
        policy =>
        {
            policy.AddRequirements(new SurveyAdminRequirement());
            policy.RequireAuthenticatedUser();  
            policy.AddAuthenticationSchemes(CookieAuthenticationDefaults.AuthenticationScheme);
        });
});
```

Este código también establece el esquema de autenticación, que indica a ASP.NET qué middleware de autenticación se debe ejecutar si se produce un error en la autorización. En este caso, especificamos el middleware de autenticación de cookies porque dicho middleware puede redirigir al usuario a una página "Prohibido". La ubicación de la página Prohibido se establece en la opción `AccessDeniedPath` para el middleware de cookies; consulte [Configuring the authentication middleware] (Configuración del middleware de autenticación).

### <a name="authorize-controller-actions"></a>Autorización de acciones de controlador
Por último, para autorizar una acción en un controlador MVC, establezca la directiva en el atributo `Authorize` :

```csharp
[Authorize(Policy = PolicyNames.RequireSurveyCreator)]
public IActionResult Create()
{
    var survey = new SurveyDTO();
    return View(survey);
}
```

En versiones anteriores de ASP.NET, establecería la propiedad **Roles** en el atributo:

```csharp
// old way
[Authorize(Roles = "SurveyCreator")]

```

Esto todavía se admite en ASP.NET Core, pero tiene algunos inconvenientes en comparación con las directivas de autorización:

* Supone un tipo de notificación determinado. Las directivas pueden comprobar cualquier tipo de notificación. Los roles son simplemente un tipo de notificación.
* El nombre de rol está codificado de forma rígida en el atributo. Con las directivas, la lógica de autorización está toda en un mismo lugar, lo que facilita la actualización o incluso la carga de la configuración.
* Las directivas permiten decisiones de autorización más complejas (por ejemplo, edad > = 21) que no se pueden expresar mediante la pertenencia de rol simple.

## <a name="resource-based-authorization"></a>Autorización basada en recursos
La *autorización basada en recursos* tiene lugar siempre que la autorización depende de un recurso específico que se verá afectado por una operación. En la aplicación Tailspin Surveys , cada encuesta tiene un propietario y de cero a muchos colaboradores.

* El propietario puede leer, actualizar, eliminar, publicar y cancelar la publicación de la encuesta.
* El propietario puede asignar colaboradores a la encuesta.
* Los colaboradores pueden leer y actualizar la encuesta.

Tenga en cuenta que "propietario" y "colaborador" no son roles de aplicación; se almacenan por encuesta, en la base de datos de la aplicación. Por ejemplo, para comprobar si un usuario puede eliminar una encuesta, la aplicación comprueba si dicho usuario es el propietario de esa encuesta.

En ASP.NET Core, implemente la autorización basada en recursos derivando de **AuthorizationHandler** y sustituyendo el método **Handle**.

```csharp
public class SurveyAuthorizationHandler : AuthorizationHandler<OperationAuthorizationRequirement, Survey>
{
    protected override void HandleRequirementAsync(AuthorizationHandlerContext context, OperationAuthorizationRequirement operation, Survey resource)
    {
    }
}
```

Observe que esta clase está fuertemente tipada para objetos Survey.  Registre la clase para DI en el inicio:

```csharp
services.AddSingleton<IAuthorizationHandler>(factory =>
{
    return new SurveyAuthorizationHandler();
});
```

Para realizar comprobaciones de autorización, use la interfaz **IAuthorizationService** , que puede inyectar insertar en los controladores. El código siguiente comprueba si un usuario puede leer una encuesta:

```csharp
if (await _authorizationService.AuthorizeAsync(User, survey, Operations.Read) == false)
{
    return StatusCode(403);
}
```

Como se pasa en un objeto `Survey`, esta llamada invocará a `SurveyAuthorizationHandler`.

En el código de autorización, un buen enfoque es agregar todos los permisos basados en roles y en recursos del usuario, y después comprobar el conjunto total respecto a la operación deseada.
A continuación se muestra un ejemplo de la aplicación Surveys. La aplicación define varios tipos de permiso:

* Administrador
* Colaborador
* Creador
* Propietario
* Lector

La aplicación también define un conjunto de operaciones posibles en encuestas:

* Crear
* Lectura
* Actualizar
* Eliminar
* Publicar
* Cancelar publicación

El código siguiente crea una lista de permisos para un usuario y encuesta determinados. Observe que este código busca tanto en los roles de aplicación del usuario como en los campos de propietario y colaborador de la encuesta.

```csharp
public class SurveyAuthorizationHandler : AuthorizationHandler<OperationAuthorizationRequirement, Survey>
{
    protected override Task HandleRequirementAsync(AuthorizationHandlerContext context, OperationAuthorizationRequirement requirement, Survey resource)
    {
        var permissions = new List<UserPermissionType>();
        int surveyTenantId = context.User.GetSurveyTenantIdValue();
        int userId = context.User.GetSurveyUserIdValue();
        string user = context.User.GetUserName();

        if (resource.TenantId == surveyTenantId)
        {
            // Admin can do anything, as long as the resource belongs to the admin's tenant.
            if (context.User.HasClaim(ClaimTypes.Role, Roles.SurveyAdmin))
            {
                context.Succeed(requirement);
                return Task.FromResult(0);
            }

            if (context.User.HasClaim(ClaimTypes.Role, Roles.SurveyCreator))
            {
                permissions.Add(UserPermissionType.Creator);
            }
            else
            {
                permissions.Add(UserPermissionType.Reader);
            }

            if (resource.OwnerId == userId)
            {
                permissions.Add(UserPermissionType.Owner);
            }
        }
        if (resource.Contributors != null && resource.Contributors.Any(x => x.UserId == userId))
        {
            permissions.Add(UserPermissionType.Contributor);
        }

        if (ValidateUserPermissions[requirement](permissions))
        {
            context.Succeed(requirement);
        }
        return Task.FromResult(0);
    }
}
```

En una aplicación de varios inquilinos, debe asegurarse de que los permisos no se "fugan" a los datos de otro inquilino. En la aplicación Surveys, se permite el permiso de colaborador entre los inquilinos; puede asignar un usuario de otro inquilino como un colaborador. Los otros tipos de permisos están restringidos a los recursos que pertenecen al inquilino del usuario. Para aplicar este requisito, el código comprueba el identificador del inquilino antes de conceder el permiso. (El campo `TenantId` asignado cuando se crea la encuesta.)

El paso siguiente es comprobar la operación (lectura, actualización, eliminación, etc.) en relación a los permisos. La aplicación Surveys implementa este paso mediante el uso de una tabla de búsqueda de funciones:

```csharp
static readonly Dictionary<OperationAuthorizationRequirement, Func<List<UserPermissionType>, bool>> ValidateUserPermissions
    = new Dictionary<OperationAuthorizationRequirement, Func<List<UserPermissionType>, bool>>

    {
        { Operations.Create, x => x.Contains(UserPermissionType.Creator) },

        { Operations.Read, x => x.Contains(UserPermissionType.Creator) ||
                                x.Contains(UserPermissionType.Reader) ||
                                x.Contains(UserPermissionType.Contributor) ||
                                x.Contains(UserPermissionType.Owner) },

        { Operations.Update, x => x.Contains(UserPermissionType.Contributor) ||
                                x.Contains(UserPermissionType.Owner) },

        { Operations.Delete, x => x.Contains(UserPermissionType.Owner) },

        { Operations.Publish, x => x.Contains(UserPermissionType.Owner) },

        { Operations.UnPublish, x => x.Contains(UserPermissionType.Owner) }
    };
```

[**Siguiente**][web-api]

<!-- Links -->
[Tailspin]: tailspin.md

[Application roles (Roles de aplicación)]: app-roles.md
[policies]: /aspnet/core/security/authorization/policies
[implementación de referencia]: tailspin.md
[Configuring the authentication middleware]: authenticate.md#configure-the-auth-middleware
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
[web-api]: web-api.md
