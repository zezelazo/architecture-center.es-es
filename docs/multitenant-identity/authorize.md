---
title: Autorización en aplicaciones multiinquilino
description: Realización de la autorización en una aplicación multiinquilino
author: MikeWasson
ms.date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: app-roles
pnp.series.next: web-api
ms.openlocfilehash: 8ff2317eb85197ed93e048b6a2d836405436cc17
ms.sourcegitcommit: 4ba3304eebaa8c493c3e5307bdd9d723cd90b655
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/12/2018
ms.locfileid: "53307170"
---
# <a name="role-based-and-resource-based-authorization"></a><span data-ttu-id="21414-103">Autorización basada en roles y en recursos</span><span class="sxs-lookup"><span data-stu-id="21414-103">Role-based and resource-based authorization</span></span>

<span data-ttu-id="21414-104">[![GitHub](../_images/github.png) Código de ejemplo][sample application]</span><span class="sxs-lookup"><span data-stu-id="21414-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="21414-105">Nuestra [implementación de referencia] es una aplicación ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="21414-105">Our [reference implementation] is an ASP.NET Core application.</span></span> <span data-ttu-id="21414-106">En este artículo se van a ver dos enfoques generales para la autorización, mediante las API de autorización provistas en ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="21414-106">In this article we'll look at two general approaches to authorization, using the authorization APIs provided in ASP.NET Core.</span></span>

* <span data-ttu-id="21414-107">**Autorización basada en roles**.</span><span class="sxs-lookup"><span data-stu-id="21414-107">**Role-based authorization**.</span></span> <span data-ttu-id="21414-108">Autorizar una acción basándose en los roles asignados a un usuario.</span><span class="sxs-lookup"><span data-stu-id="21414-108">Authorizing an action based on the roles assigned to a user.</span></span> <span data-ttu-id="21414-109">Por ejemplo, algunas acciones requieren un rol de administrador.</span><span class="sxs-lookup"><span data-stu-id="21414-109">For example, some actions require an administrator role.</span></span>
* <span data-ttu-id="21414-110">**Autorización basada en recursos**.</span><span class="sxs-lookup"><span data-stu-id="21414-110">**Resource-based authorization**.</span></span> <span data-ttu-id="21414-111">Autorizar una acción basándose en un recurso determinado.</span><span class="sxs-lookup"><span data-stu-id="21414-111">Authorizing an action based on a particular resource.</span></span> <span data-ttu-id="21414-112">Por ejemplo, cada recurso tiene un propietario.</span><span class="sxs-lookup"><span data-stu-id="21414-112">For example, every resource has an owner.</span></span> <span data-ttu-id="21414-113">El propietario puede eliminar el recurso; otros usuarios no.</span><span class="sxs-lookup"><span data-stu-id="21414-113">The owner can delete the resource; other users cannot.</span></span>

<span data-ttu-id="21414-114">Una aplicación típica empleará una combinación de ambos.</span><span class="sxs-lookup"><span data-stu-id="21414-114">A typical app will employ a mix of both.</span></span> <span data-ttu-id="21414-115">Por ejemplo, para eliminar un recurso, el usuario debe ser el propietario del mismo *o* administrador.</span><span class="sxs-lookup"><span data-stu-id="21414-115">For example, to delete a resource, the user must be the resource owner *or* an admin.</span></span>

## <a name="role-based-authorization"></a><span data-ttu-id="21414-116">Autorización basada en roles</span><span class="sxs-lookup"><span data-stu-id="21414-116">Role-Based Authorization</span></span>
<span data-ttu-id="21414-117">La aplicación [Tailspin Surveys][Tailspin] define los siguientes roles:</span><span class="sxs-lookup"><span data-stu-id="21414-117">The [Tailspin Surveys][Tailspin] application defines the following roles:</span></span>

* <span data-ttu-id="21414-118">Administrador.</span><span class="sxs-lookup"><span data-stu-id="21414-118">Administrator.</span></span> <span data-ttu-id="21414-119">Puede realizar todas las operaciones CRUD en cualquier encuesta que pertenezca a ese inquilino.</span><span class="sxs-lookup"><span data-stu-id="21414-119">Can perform all CRUD operations on any survey that belongs to that tenant.</span></span>
* <span data-ttu-id="21414-120">Creador.</span><span class="sxs-lookup"><span data-stu-id="21414-120">Creator.</span></span> <span data-ttu-id="21414-121">Puede crear nuevas encuestas.</span><span class="sxs-lookup"><span data-stu-id="21414-121">Can create new surveys</span></span>
* <span data-ttu-id="21414-122">Lector.</span><span class="sxs-lookup"><span data-stu-id="21414-122">Reader.</span></span> <span data-ttu-id="21414-123">Puede leer cualquier encuesta que pertenezca a ese inquilino.</span><span class="sxs-lookup"><span data-stu-id="21414-123">Can read any surveys that belong to that tenant</span></span>

<span data-ttu-id="21414-124">Los roles se aplican a los *usuarios* de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="21414-124">Roles apply to *users* of the application.</span></span> <span data-ttu-id="21414-125">En la aplicación Surveys, un usuario es administrador, creador o lector.</span><span class="sxs-lookup"><span data-stu-id="21414-125">In the Surveys application, a user is either an administrator, creator, or reader.</span></span>

<span data-ttu-id="21414-126">Para ver una discusión de cómo definir y administrar roles, consulte [Roles de la aplicación].</span><span class="sxs-lookup"><span data-stu-id="21414-126">For a discussion of how to define and manage roles, see [Application roles].</span></span>

<span data-ttu-id="21414-127">Independientemente de cómo administre los roles, el código de autorización tendrá un aspecto similar.</span><span class="sxs-lookup"><span data-stu-id="21414-127">Regardless of how you manage the roles, your authorization code will look similar.</span></span> <span data-ttu-id="21414-128">ASP.NET Core cuenta con una abstracción llamada [directivas de autorización][policies].</span><span class="sxs-lookup"><span data-stu-id="21414-128">ASP.NET Core has an abstraction called [authorization policies][policies].</span></span> <span data-ttu-id="21414-129">Con esta característica, el usuario define directivas de autorización en el código y después las aplica a las acciones de controlador.</span><span class="sxs-lookup"><span data-stu-id="21414-129">With this feature, you define authorization policies in code, and then apply those policies to controller actions.</span></span> <span data-ttu-id="21414-130">La directiva se separa del controlador.</span><span class="sxs-lookup"><span data-stu-id="21414-130">The policy is decoupled from the controller.</span></span>

### <a name="create-policies"></a><span data-ttu-id="21414-131">Creación de directivas</span><span class="sxs-lookup"><span data-stu-id="21414-131">Create policies</span></span>
<span data-ttu-id="21414-132">Para definir una directiva, primero cree una clase que implemente `IAuthorizationRequirement`.</span><span class="sxs-lookup"><span data-stu-id="21414-132">To define a policy, first create a class that implements `IAuthorizationRequirement`.</span></span> <span data-ttu-id="21414-133">Es lo más fácil para derivar de `AuthorizationHandler`.</span><span class="sxs-lookup"><span data-stu-id="21414-133">It's easiest to derive from `AuthorizationHandler`.</span></span> <span data-ttu-id="21414-134">En el método `Handle` , examine las notificaciones pertinentes.</span><span class="sxs-lookup"><span data-stu-id="21414-134">In the `Handle` method, examine the relevant claim(s).</span></span>

<span data-ttu-id="21414-135">A continuación se muestra un ejemplo de la aplicación Tailspin Surveys:</span><span class="sxs-lookup"><span data-stu-id="21414-135">Here is an example from the Tailspin Surveys application:</span></span>

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

<span data-ttu-id="21414-136">Esta clase define el requisito para un usuario para crear una nueva encuesta.</span><span class="sxs-lookup"><span data-stu-id="21414-136">This class defines the requirement for a user to create a new survey.</span></span> <span data-ttu-id="21414-137">El usuario debe estar en el rol SurveyAdmin o SurveyCreator.</span><span class="sxs-lookup"><span data-stu-id="21414-137">The user must be in the SurveyAdmin or SurveyCreator role.</span></span>

<span data-ttu-id="21414-138">En la clase de inicio, defina una directiva con nombre que incluya uno o varios requisitos.</span><span class="sxs-lookup"><span data-stu-id="21414-138">In your startup class, define a named policy that includes one or more requirements.</span></span> <span data-ttu-id="21414-139">Si hay varios requisitos, el usuario debe cumplir *todos* ellos para que se le conceda autorización.</span><span class="sxs-lookup"><span data-stu-id="21414-139">If there are multiple requirements, the user must meet *every* requirement to be authorized.</span></span> <span data-ttu-id="21414-140">El código siguiente define dos directivas:</span><span class="sxs-lookup"><span data-stu-id="21414-140">The following code defines two policies:</span></span>

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

<span data-ttu-id="21414-141">Este código también establece el esquema de autenticación, que indica a ASP.NET qué middleware de autenticación se debe ejecutar si se produce un error en la autorización.</span><span class="sxs-lookup"><span data-stu-id="21414-141">This code also sets the authentication scheme, which tells ASP.NET which authentication middleware should run if authorization fails.</span></span> <span data-ttu-id="21414-142">En este caso, especificamos el middleware de autenticación de cookies porque dicho middleware puede redirigir al usuario a una página "Prohibido".</span><span class="sxs-lookup"><span data-stu-id="21414-142">In this case, we specify the cookie authentication middleware, because the cookie authentication middleware can redirect the user to a "Forbidden" page.</span></span> <span data-ttu-id="21414-143">La ubicación de la página Prohibido se establece en la opción `AccessDeniedPath` para el middleware de cookies; consulte [Configuración del middleware de autenticación].</span><span class="sxs-lookup"><span data-stu-id="21414-143">The location of the Forbidden page is set in the `AccessDeniedPath` option for the cookie middleware; see [Configuring the authentication middleware].</span></span>

### <a name="authorize-controller-actions"></a><span data-ttu-id="21414-144">Autorización de acciones de controlador</span><span class="sxs-lookup"><span data-stu-id="21414-144">Authorize controller actions</span></span>
<span data-ttu-id="21414-145">Por último, para autorizar una acción en un controlador MVC, establezca la directiva en el atributo `Authorize` :</span><span class="sxs-lookup"><span data-stu-id="21414-145">Finally, to authorize an action in an MVC controller, set the policy in the `Authorize` attribute:</span></span>

```csharp
[Authorize(Policy = PolicyNames.RequireSurveyCreator)]
public IActionResult Create()
{
    var survey = new SurveyDTO();
    return View(survey);
}
```

<span data-ttu-id="21414-146">En versiones anteriores de ASP.NET, establecería la propiedad **Roles** en el atributo:</span><span class="sxs-lookup"><span data-stu-id="21414-146">In earlier versions of ASP.NET, you would set the **Roles** property on the attribute:</span></span>

```csharp
// old way
[Authorize(Roles = "SurveyCreator")]
```

<span data-ttu-id="21414-147">Esto todavía se admite en ASP.NET Core, pero tiene algunos inconvenientes en comparación con las directivas de autorización:</span><span class="sxs-lookup"><span data-stu-id="21414-147">This is still supported in ASP.NET Core, but it has some drawbacks compared with authorization policies:</span></span>

* <span data-ttu-id="21414-148">Supone un tipo de notificación determinado.</span><span class="sxs-lookup"><span data-stu-id="21414-148">It assumes a particular claim type.</span></span> <span data-ttu-id="21414-149">Las directivas pueden comprobar cualquier tipo de notificación.</span><span class="sxs-lookup"><span data-stu-id="21414-149">Policies can check for any claim type.</span></span> <span data-ttu-id="21414-150">Los roles son simplemente un tipo de notificación.</span><span class="sxs-lookup"><span data-stu-id="21414-150">Roles are just a type of claim.</span></span>
* <span data-ttu-id="21414-151">El nombre de rol está codificado de forma rígida en el atributo.</span><span class="sxs-lookup"><span data-stu-id="21414-151">The role name is hard-coded into the attribute.</span></span> <span data-ttu-id="21414-152">Con las directivas, la lógica de autorización está toda en un mismo lugar, lo que facilita la actualización o incluso la carga de la configuración.</span><span class="sxs-lookup"><span data-stu-id="21414-152">With policies, the authorization logic is all in one place, making it easier to update or even load from configuration settings.</span></span>
* <span data-ttu-id="21414-153">Las directivas permiten decisiones de autorización más complejas (por ejemplo, edad > = 21) que no se pueden expresar mediante la pertenencia de rol simple.</span><span class="sxs-lookup"><span data-stu-id="21414-153">Policies enable more complex authorization decisions (e.g., age >= 21) that can't be expressed by simple role membership.</span></span>

## <a name="resource-based-authorization"></a><span data-ttu-id="21414-154">Autorización basada en recursos</span><span class="sxs-lookup"><span data-stu-id="21414-154">Resource based authorization</span></span>
<span data-ttu-id="21414-155">*Autorización basada en recursos* tiene lugar siempre que la autorización depende de un recurso específico que se verá afectado por una operación.</span><span class="sxs-lookup"><span data-stu-id="21414-155">*Resource based authorization* occurs whenever the authorization depends on a specific resource that will be affected by an operation.</span></span> <span data-ttu-id="21414-156">En la aplicación Tailspin Surveys , cada encuesta tiene un propietario y de cero a muchos colaboradores.</span><span class="sxs-lookup"><span data-stu-id="21414-156">In the Tailspin Surveys application, every survey has an owner and zero-to-many contributors.</span></span>

* <span data-ttu-id="21414-157">El propietario puede leer, actualizar, eliminar, publicar y cancelar la publicación de la encuesta.</span><span class="sxs-lookup"><span data-stu-id="21414-157">The owner can read, update, delete, publish, and unpublish the survey.</span></span>
* <span data-ttu-id="21414-158">El propietario puede asignar colaboradores a la encuesta.</span><span class="sxs-lookup"><span data-stu-id="21414-158">The owner can assign contributors to the survey.</span></span>
* <span data-ttu-id="21414-159">Los colaboradores pueden leer y actualizar la encuesta.</span><span class="sxs-lookup"><span data-stu-id="21414-159">Contributors can read and update the survey.</span></span>

<span data-ttu-id="21414-160">Tenga en cuenta que "propietario" y "colaborador" no son roles de aplicación; se almacenan por encuesta, en la base de datos de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="21414-160">Note that "owner" and "contributor" are not application roles; they are stored per survey, in the application database.</span></span> <span data-ttu-id="21414-161">Por ejemplo, para comprobar si un usuario puede eliminar una encuesta, la aplicación comprueba si dicho usuario es el propietario de esa encuesta.</span><span class="sxs-lookup"><span data-stu-id="21414-161">To check whether a user can delete a survey, for example, the app checks whether the user is the owner for that survey.</span></span>

<span data-ttu-id="21414-162">En ASP.NET Core, implemente la autorización basada en recursos derivando de **AuthorizationHandler** y sustituyendo el método **Handle**.</span><span class="sxs-lookup"><span data-stu-id="21414-162">In ASP.NET Core, implement resource-based authorization by deriving from **AuthorizationHandler** and overriding the **Handle** method.</span></span>

```csharp
public class SurveyAuthorizationHandler : AuthorizationHandler<OperationAuthorizationRequirement, Survey>
{
    protected override void HandleRequirementAsync(AuthorizationHandlerContext context, OperationAuthorizationRequirement operation, Survey resource)
    {
    }
}
```

<span data-ttu-id="21414-163">Observe que esta clase está fuertemente tipada para objetos Survey.</span><span class="sxs-lookup"><span data-stu-id="21414-163">Notice that this class is strongly typed for Survey objects.</span></span>  <span data-ttu-id="21414-164">Registre la clase para DI en el inicio:</span><span class="sxs-lookup"><span data-stu-id="21414-164">Register the class for DI on startup:</span></span>

```csharp
services.AddSingleton<IAuthorizationHandler>(factory =>
{
    return new SurveyAuthorizationHandler();
});
```

<span data-ttu-id="21414-165">Para realizar comprobaciones de autorización, use la interfaz **IAuthorizationService** , que puede inyectar insertar en los controladores.</span><span class="sxs-lookup"><span data-stu-id="21414-165">To perform authorization checks, use the **IAuthorizationService** interface, which you can inject into your controllers.</span></span> <span data-ttu-id="21414-166">El código siguiente comprueba si un usuario puede leer una encuesta:</span><span class="sxs-lookup"><span data-stu-id="21414-166">The following code checks whether a user can read a survey:</span></span>

```csharp
if (await _authorizationService.AuthorizeAsync(User, survey, Operations.Read) == false)
{
    return StatusCode(403);
}
```

<span data-ttu-id="21414-167">Como se pasa en un objeto `Survey`, esta llamada invocará a `SurveyAuthorizationHandler`.</span><span class="sxs-lookup"><span data-stu-id="21414-167">Because we pass in a `Survey` object, this call will invoke the `SurveyAuthorizationHandler`.</span></span>

<span data-ttu-id="21414-168">En el código de autorización, un buen enfoque es agregar todos los permisos basados en roles y en recursos del usuario, y después comprobar el conjunto total respecto a la operación deseada.</span><span class="sxs-lookup"><span data-stu-id="21414-168">In your authorization code, a good approach is to aggregate all of the user's role-based and resource-based permissions, then check the aggregate set against the desired operation.</span></span>
<span data-ttu-id="21414-169">A continuación se muestra un ejemplo de la aplicación Surveys.</span><span class="sxs-lookup"><span data-stu-id="21414-169">Here is an example from the Surveys app.</span></span> <span data-ttu-id="21414-170">La aplicación define varios tipos de permiso:</span><span class="sxs-lookup"><span data-stu-id="21414-170">The application defines several permission types:</span></span>

* <span data-ttu-id="21414-171">Administración</span><span class="sxs-lookup"><span data-stu-id="21414-171">Admin</span></span>
* <span data-ttu-id="21414-172">Colaborador</span><span class="sxs-lookup"><span data-stu-id="21414-172">Contributor</span></span>
* <span data-ttu-id="21414-173">Creador</span><span class="sxs-lookup"><span data-stu-id="21414-173">Creator</span></span>
* <span data-ttu-id="21414-174">Propietario</span><span class="sxs-lookup"><span data-stu-id="21414-174">Owner</span></span>
* <span data-ttu-id="21414-175">Lector</span><span class="sxs-lookup"><span data-stu-id="21414-175">Reader</span></span>

<span data-ttu-id="21414-176">La aplicación también define un conjunto de operaciones posibles en encuestas:</span><span class="sxs-lookup"><span data-stu-id="21414-176">The application also defines a set of possible operations on surveys:</span></span>

* <span data-ttu-id="21414-177">Crear</span><span class="sxs-lookup"><span data-stu-id="21414-177">Create</span></span>
* <span data-ttu-id="21414-178">Lectura</span><span class="sxs-lookup"><span data-stu-id="21414-178">Read</span></span>
* <span data-ttu-id="21414-179">Actualizar</span><span class="sxs-lookup"><span data-stu-id="21414-179">Update</span></span>
* <span data-ttu-id="21414-180">Eliminar</span><span class="sxs-lookup"><span data-stu-id="21414-180">Delete</span></span>
* <span data-ttu-id="21414-181">Publicar</span><span class="sxs-lookup"><span data-stu-id="21414-181">Publish</span></span>
* <span data-ttu-id="21414-182">Cancelar publicación</span><span class="sxs-lookup"><span data-stu-id="21414-182">Unpublish</span></span>

<span data-ttu-id="21414-183">El código siguiente crea una lista de permisos para un usuario y encuesta determinados.</span><span class="sxs-lookup"><span data-stu-id="21414-183">The following code creates a list of permissions for a particular user and survey.</span></span> <span data-ttu-id="21414-184">Observe que este código busca tanto en los roles de aplicación del usuario como en los campos de propietario y colaborador de la encuesta.</span><span class="sxs-lookup"><span data-stu-id="21414-184">Notice that this code looks at both the user's app roles, and the owner/contributor fields in the survey.</span></span>

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

<span data-ttu-id="21414-185">En una aplicación de varios inquilinos, debe asegurarse de que los permisos no se "fugan" a los datos de otro inquilino.</span><span class="sxs-lookup"><span data-stu-id="21414-185">In a multi-tenant application, you must ensure that permissions don't "leak" to another tenant's data.</span></span> <span data-ttu-id="21414-186">En la aplicación Surveys, se permite el permiso de colaborador entre los inquilinos &mdash; puede asignar un usuario de otro inquilino como un colaborador.</span><span class="sxs-lookup"><span data-stu-id="21414-186">In the Surveys app, the Contributor permission is allowed across tenants &mdash; you can assign someone from another tenant as a contributor.</span></span> <span data-ttu-id="21414-187">Los otros tipos de permisos están restringidos a los recursos que pertenecen al inquilino del usuario.</span><span class="sxs-lookup"><span data-stu-id="21414-187">The other permission types are restricted to resources that belong to that user's tenant.</span></span> <span data-ttu-id="21414-188">Para aplicar este requisito, el código comprueba el identificador del inquilino antes de conceder el permiso.</span><span class="sxs-lookup"><span data-stu-id="21414-188">To enforce this requirement, the code checks the tenant ID before granting the permission.</span></span> <span data-ttu-id="21414-189">(El campo `TenantId` asignado cuando se crea la encuesta.)</span><span class="sxs-lookup"><span data-stu-id="21414-189">(The `TenantId` field as assigned when the survey is created.)</span></span>

<span data-ttu-id="21414-190">El paso siguiente es comprobar la operación (lectura, actualización, eliminación, etc.) en relación a los permisos.</span><span class="sxs-lookup"><span data-stu-id="21414-190">The next step is to check the operation (read, update, delete, etc) against the permissions.</span></span> <span data-ttu-id="21414-191">La aplicación Surveys implementa este paso mediante el uso de una tabla de búsqueda de funciones:</span><span class="sxs-lookup"><span data-stu-id="21414-191">The Surveys app implements this step by using a lookup table of functions:</span></span>

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

<span data-ttu-id="21414-192">[**Siguiente**][web-api]</span><span class="sxs-lookup"><span data-stu-id="21414-192">[**Next**][web-api]</span></span>

<!-- Links -->
[Tailspin]: tailspin.md

[Roles de la aplicación]: app-roles.md
[Application roles]: app-roles.md
[policies]: /aspnet/core/security/authorization/policies
[Implementación de referencia]: tailspin.md
[reference implementation]: tailspin.md
[Configuración del middleware de autenticación]: authenticate.md#configure-the-auth-middleware
[Configuring the authentication middleware]: authenticate.md#configure-the-auth-middleware
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
[web-api]: web-api.md
