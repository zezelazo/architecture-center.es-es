---
title: Administración de identidades en aplicaciones multiinquilino
description: Procedimientos recomendados para autenticación, autorización y administración de identidades en aplicaciones multiinquilino.
author: MikeWasson
ms.date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.next: tailspin
ms.openlocfilehash: 24e09720d3257cbfae350995fa5238663da1d26e
ms.sourcegitcommit: e7e0e0282fa93f0063da3b57128ade395a9c1ef9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/05/2018
ms.locfileid: "52902058"
---
# <a name="manage-identity-in-multitenant-applications"></a><span data-ttu-id="4e084-103">Administración de identidades en aplicaciones multiinquilino</span><span class="sxs-lookup"><span data-stu-id="4e084-103">Manage Identity in Multitenant Applications</span></span>

<span data-ttu-id="4e084-104">En esta serie se artículos se describen los procedimientos recomendados para la arquitectura multiinquilino durante el uso de Azure AD para la autenticación y la administración de identidades.</span><span class="sxs-lookup"><span data-stu-id="4e084-104">This series of articles describes best practices for multitenancy, when using Azure AD for authentication and identity management.</span></span>

<span data-ttu-id="4e084-105">[![GitHub](../_images/github.png) Código de ejemplo][sample application]</span><span class="sxs-lookup"><span data-stu-id="4e084-105">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="4e084-106">Cuando está creando una aplicación multiinquilino, uno de los primeros desafíos es administrar las identidades de usuario, ya que ahora todos los usuarios pertenecen a un inquilino.</span><span class="sxs-lookup"><span data-stu-id="4e084-106">When you're building a multitenant application, one of the first challenges is managing user identities, because now every user belongs to a tenant.</span></span> <span data-ttu-id="4e084-107">Por ejemplo: </span><span class="sxs-lookup"><span data-stu-id="4e084-107">For example:</span></span>

* <span data-ttu-id="4e084-108">Los usuarios inician sesión con las credenciales de la organización.</span><span class="sxs-lookup"><span data-stu-id="4e084-108">Users sign in with their organizational credentials.</span></span>
* <span data-ttu-id="4e084-109">Los usuarios deben tener acceso a datos de su organización, pero no a los datos que pertenecen a otros inquilinos.</span><span class="sxs-lookup"><span data-stu-id="4e084-109">Users should have access to their organization's data, but not data that belongs to other tenants.</span></span>
* <span data-ttu-id="4e084-110">Una organización puede suscribirse a la aplicación y, a continuación, asignar roles de aplicación a sus miembros.</span><span class="sxs-lookup"><span data-stu-id="4e084-110">An organization can sign up for the application, and then assign application roles to its members.</span></span>

<span data-ttu-id="4e084-111">Azure Active Directory (Azure AD) tiene algunas características excelentes que admiten todos estos escenarios.</span><span class="sxs-lookup"><span data-stu-id="4e084-111">Azure Active Directory (Azure AD) has some great features that support all of these scenarios.</span></span>

<span data-ttu-id="4e084-112">Para acompañar esta serie de artículos, se ha creado una [implementación de un extremo a otro][sample application] completa de una aplicación multiinquilino.</span><span class="sxs-lookup"><span data-stu-id="4e084-112">To accompany this series of articles, we created a complete [end-to-end implementation][sample application] of a multitenant application.</span></span> <span data-ttu-id="4e084-113">Los artículos reflejan lo que hemos aprendido en el proceso de creación de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="4e084-113">The articles reflect what we learned in the process of building the application.</span></span> <span data-ttu-id="4e084-114">Para empezar a trabajar con la aplicación, consulte [Running the Surveys application ][running-the-app] (Ejecución de la aplicación Surveys).</span><span class="sxs-lookup"><span data-stu-id="4e084-114">To get started with the application, see [Run the Surveys application][running-the-app].</span></span>

## <a name="introduction"></a><span data-ttu-id="4e084-115">Introducción</span><span class="sxs-lookup"><span data-stu-id="4e084-115">Introduction</span></span>

<span data-ttu-id="4e084-116">Supongamos que está escribiendo una aplicación de SaaS empresarial que se hospedará en la nube.</span><span class="sxs-lookup"><span data-stu-id="4e084-116">Let's say you're writing an enterprise SaaS application to be hosted in the cloud.</span></span> <span data-ttu-id="4e084-117">Por supuesto, la aplicación tendrá usuarios:</span><span class="sxs-lookup"><span data-stu-id="4e084-117">Of course, the application will have users:</span></span>

![Usuarios](./images/users.png)

<span data-ttu-id="4e084-119">Sin embargo, los usuarios pertenecen a las organizaciones:</span><span class="sxs-lookup"><span data-stu-id="4e084-119">But those users belong to organizations:</span></span>

![Usuarios de organización](./images/org-users.png)

<span data-ttu-id="4e084-121">Ejemplo: Tailspin oferta suscripciones a su aplicación de SaaS.</span><span class="sxs-lookup"><span data-stu-id="4e084-121">Example: Tailspin sells subscriptions to its SaaS application.</span></span> <span data-ttu-id="4e084-122">Contoso y Fabrikam se registran en la aplicación.</span><span class="sxs-lookup"><span data-stu-id="4e084-122">Contoso and Fabrikam sign up for the app.</span></span> <span data-ttu-id="4e084-123">Cuando Alice (`alice@contoso`) inicia sesión, la aplicación debe saber que esta usuaria forma parte de Contoso.</span><span class="sxs-lookup"><span data-stu-id="4e084-123">When Alice (`alice@contoso`) signs in, the application should know that Alice is part of Contoso.</span></span>

* <span data-ttu-id="4e084-124">Alice *debería* tener acceso a datos de Contoso.</span><span class="sxs-lookup"><span data-stu-id="4e084-124">Alice *should* have access to Contoso data.</span></span>
* <span data-ttu-id="4e084-125">Alice *no debería* tener acceso a los datos de Fabrikam.</span><span class="sxs-lookup"><span data-stu-id="4e084-125">Alice *should not* have access to Fabrikam data.</span></span>

<span data-ttu-id="4e084-126">Esta guía muestra cómo administrar identidades de usuario en una aplicación multiinquilino, usando [Azure Active Directory][AzureAD] (Azure AD) para controlar el inicio de sesión y la autenticación.</span><span class="sxs-lookup"><span data-stu-id="4e084-126">This guidance will show you how to manage user identities in a multitenant application, using [Azure Active Directory][AzureAD] (Azure AD) to handle sign-in and authentication.</span></span>

## <a name="what-is-multitenancy"></a><span data-ttu-id="4e084-127">¿Qué significa multiinquilino?</span><span class="sxs-lookup"><span data-stu-id="4e084-127">What is multitenancy?</span></span>
<span data-ttu-id="4e084-128">Un *inquilino* es un grupo de usuarios.</span><span class="sxs-lookup"><span data-stu-id="4e084-128">A *tenant* is a group of users.</span></span> <span data-ttu-id="4e084-129">En una aplicación SaaS, el inquilino es un suscriptor o un usuario de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="4e084-129">In a SaaS application, the tenant is a subscriber or customer of the application.</span></span> <span data-ttu-id="4e084-130">*arquitectura multiinquilino* es una arquitectura en la que varios inquilinos comparten la misma instancia física de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="4e084-130">*Multitenancy* is an architecture where multiple tenants share the same physical instance of the app.</span></span> <span data-ttu-id="4e084-131">Aunque los inquilinos comparten los recursos físicos (como las máquinas virtuales o el almacenamiento), cada inquilino obtiene su propia instancia lógica de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="4e084-131">Although tenants share physical resources (such as VMs or storage), each tenant gets its own logical instance of the app.</span></span>

<span data-ttu-id="4e084-132">Normalmente, los datos de la aplicación se comparten entre los usuarios de un mismo inquilino, pero no con otros inquilinos.</span><span class="sxs-lookup"><span data-stu-id="4e084-132">Typically, application data is shared among the users within a tenant, but not with other tenants.</span></span>

![Multiinquilino](./images/multitenant.png)

<span data-ttu-id="4e084-134">Compare esta arquitectura con otra de un solo inquilino, donde cada inquilino tiene una instancia física dedicada.</span><span class="sxs-lookup"><span data-stu-id="4e084-134">Compare this architecture with a single-tenant architecture, where each tenant has a dedicated physical instance.</span></span> <span data-ttu-id="4e084-135">En una arquitectura de un solo inquilino, los nuevos inquilinos se agregan mediante la puesta en marcha de nuevas instancias de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="4e084-135">In a single-tenant architecture, you add tenants by spinning up new instances of the app.</span></span>

![Un solo inquilino](./images/single-tenant.png)

### <a name="multitenancy-and-horizontal-scaling"></a><span data-ttu-id="4e084-137">Arquitectura multiinquilino y escalado horizontal</span><span class="sxs-lookup"><span data-stu-id="4e084-137">Multitenancy and horizontal scaling</span></span>
<span data-ttu-id="4e084-138">Para lograr la reducción horizontal en la nube, es habitual agregar más instancias físicas.</span><span class="sxs-lookup"><span data-stu-id="4e084-138">To achieve scale in the cloud, it’s common to add more physical instances.</span></span> <span data-ttu-id="4e084-139">Esto se conoce como *ampliación horizontal* o *escalado horizontal*. Piense en una aplicación web.</span><span class="sxs-lookup"><span data-stu-id="4e084-139">This is known as *horizontal scaling* or *scaling out*. Consider a web app.</span></span> <span data-ttu-id="4e084-140">Para administrar más tráfico, puede agregar más máquinas virtuales de servidor y colocarlas detrás de un equilibrador de carga.</span><span class="sxs-lookup"><span data-stu-id="4e084-140">To handle more traffic, you can add more server VMs and put them behind a load balancer.</span></span> <span data-ttu-id="4e084-141">Cada máquina virtual ejecuta una instancia física independiente de la aplicación web.</span><span class="sxs-lookup"><span data-stu-id="4e084-141">Each VM runs a separate physical instance of the web app.</span></span>

![Equilibrio de carga de un sitio web](./images/load-balancing.png)

<span data-ttu-id="4e084-143">Cualquier solicitud puede enrutarse a cualquier instancia.</span><span class="sxs-lookup"><span data-stu-id="4e084-143">Any request can be routed to any instance.</span></span> <span data-ttu-id="4e084-144">En conjunto, el sistema funciona como una única instancia lógica.</span><span class="sxs-lookup"><span data-stu-id="4e084-144">Together, the system functions as a single logical instance.</span></span> <span data-ttu-id="4e084-145">Puede anular una máquina virtual o poner en marcha una nueva sin que los usuarios resulten afectados.</span><span class="sxs-lookup"><span data-stu-id="4e084-145">You can tear down a VM or spin up a new VM, without affecting users.</span></span> <span data-ttu-id="4e084-146">En esta arquitectura, cada instancia física es multiinquilino, y se escala mediante la incorporación de más instancias.</span><span class="sxs-lookup"><span data-stu-id="4e084-146">In this architecture, each physical instance is multi-tenant, and you scale by adding more instances.</span></span> <span data-ttu-id="4e084-147">Si una instancia deja de funcionar, ningún inquilino debería resultar afectado.</span><span class="sxs-lookup"><span data-stu-id="4e084-147">If one instance goes down, it should not affect any tenant.</span></span>

## <a name="identity-in-a-multitenant-app"></a><span data-ttu-id="4e084-148">Identidad en una aplicación multiinquilino</span><span class="sxs-lookup"><span data-stu-id="4e084-148">Identity in a multitenant app</span></span>
<span data-ttu-id="4e084-149">En una aplicación multiinquilino, debe tener en cuenta a los usuarios en el contexto de los inquilinos.</span><span class="sxs-lookup"><span data-stu-id="4e084-149">In a multitenant app, you must consider users in the context of tenants.</span></span>

<span data-ttu-id="4e084-150">**Autenticación**</span><span class="sxs-lookup"><span data-stu-id="4e084-150">**Authentication**</span></span>

* <span data-ttu-id="4e084-151">Los usuarios inician sesión en la aplicación con sus credenciales de la organización.</span><span class="sxs-lookup"><span data-stu-id="4e084-151">Users sign into the app with their organization credentials.</span></span> <span data-ttu-id="4e084-152">No es necesario que creen nuevos perfiles de usuario para la aplicación.</span><span class="sxs-lookup"><span data-stu-id="4e084-152">They don't have to create new user profiles for the app.</span></span>
* <span data-ttu-id="4e084-153">Los usuarios de la misma organización forman parte del mismo inquilino.</span><span class="sxs-lookup"><span data-stu-id="4e084-153">Users within the same organization are part of the same tenant.</span></span>
* <span data-ttu-id="4e084-154">Cuando un usuario inicia sesión, la aplicación sabe a qué inquilino pertenece el usuario.</span><span class="sxs-lookup"><span data-stu-id="4e084-154">When a user signs in, the application knows which tenant the user belongs to.</span></span>

<span data-ttu-id="4e084-155">**Autorización**</span><span class="sxs-lookup"><span data-stu-id="4e084-155">**Authorization**</span></span>

* <span data-ttu-id="4e084-156">Al autorizar las acciones de un usuario (por ejemplo, ver un recurso), la aplicación debe tener en cuenta el inquilino del usuario.</span><span class="sxs-lookup"><span data-stu-id="4e084-156">When authorizing a user's actions (say, viewing a resource), the app must take into account the user's tenant.</span></span>
* <span data-ttu-id="4e084-157">Es necesario asignar roles a los usuarios dentro de la aplicación, como "Administrador" o "Usuario estándar".</span><span class="sxs-lookup"><span data-stu-id="4e084-157">Users might be assigned roles within the application, such as "Admin" or "Standard User".</span></span> <span data-ttu-id="4e084-158">Las asignaciones de roles deben administrarse por el cliente, no por el proveedor de SaaS.</span><span class="sxs-lookup"><span data-stu-id="4e084-158">Role assignments should be managed by the customer, not by the SaaS provider.</span></span>

<span data-ttu-id="4e084-159">**Ejemplo:**</span><span class="sxs-lookup"><span data-stu-id="4e084-159">**Example.**</span></span> <span data-ttu-id="4e084-160">Alice, una empleada de Contoso, navega a la aplicación en su explorador y hace clic en el botón "Iniciar sesión".</span><span class="sxs-lookup"><span data-stu-id="4e084-160">Alice, an employee at Contoso, navigates to the application in her browser and clicks the “Log in” button.</span></span> <span data-ttu-id="4e084-161">Se le redirige a una pantalla de inicio de sesión donde escribe sus credenciales corporativas (nombre de usuario y contraseña).</span><span class="sxs-lookup"><span data-stu-id="4e084-161">She is redirected to a login screen where she enters her corporate credentials (username and password).</span></span> <span data-ttu-id="4e084-162">En este momento, ha iniciado sesión en la aplicación como `alice@contoso.com`.</span><span class="sxs-lookup"><span data-stu-id="4e084-162">At this point, she is logged into the app as `alice@contoso.com`.</span></span> <span data-ttu-id="4e084-163">La aplicación también sabe que Alice es un usuario administrador en esta aplicación.</span><span class="sxs-lookup"><span data-stu-id="4e084-163">The application also knows that Alice is an admin user for this application.</span></span> <span data-ttu-id="4e084-164">Puesto que es administrador, puede ver una lista de todos los recursos que pertenecen a Contoso.</span><span class="sxs-lookup"><span data-stu-id="4e084-164">Because she is an admin, she can see a list of all the resources that belong to Contoso.</span></span> <span data-ttu-id="4e084-165">Sin embargo, no puede ver recursos de Fabrikam, puesto que solo es administrador dentro de su inquilino.</span><span class="sxs-lookup"><span data-stu-id="4e084-165">However, she cannot view Fabrikam's resources, because she is an admin only within her tenant.</span></span>

<span data-ttu-id="4e084-166">En esta guía, examinaremos específicamente el uso de Azure AD para la administración de identidades.</span><span class="sxs-lookup"><span data-stu-id="4e084-166">In this guidance, we'll look specifically at using Azure AD for identity management.</span></span>

* <span data-ttu-id="4e084-167">Se supone que el cliente almacena sus perfiles de usuario en Azure AD (incluidos los inquilinos de Office 365 y Dynamics CRM).</span><span class="sxs-lookup"><span data-stu-id="4e084-167">We assume the customer stores their user profiles in Azure AD (including Office365 and Dynamics CRM tenants)</span></span>
* <span data-ttu-id="4e084-168">Los clientes con Active Directory (AD) local pueden usar [Azure AD Connect][ADConnect] para sincronizar su instancia de AD local con Azure AD.</span><span class="sxs-lookup"><span data-stu-id="4e084-168">Customers with on-premise Active Directory (AD) can use [Azure AD Connect][ADConnect] to sync their on-premise AD with Azure AD.</span></span>

<span data-ttu-id="4e084-169">Si un cliente con AD local no puede usar Azure AD Connect (debido a la directiva de TI corporativa u otras razones), el proveedor de SaaS puede federarse con el AD del cliente a través de los Servicios de federación de Active Directory (AD FS).</span><span class="sxs-lookup"><span data-stu-id="4e084-169">If a customer with on-premise AD cannot use Azure AD Connect (due to corporate IT policy or other reasons), the SaaS provider can federate with the customer's AD through Active Directory Federation Services (AD FS).</span></span> <span data-ttu-id="4e084-170">Esta opción se describe en [Federación con AD FS de un cliente].</span><span class="sxs-lookup"><span data-stu-id="4e084-170">This option is described in [Federating with a customer's AD FS].</span></span>

<span data-ttu-id="4e084-171">Esta guía no tiene en cuenta otros aspectos de la arquitectura multiinquilino como la partición de datos, la configuración de cada inquilino, etc.</span><span class="sxs-lookup"><span data-stu-id="4e084-171">This guidance does not consider other aspects of multitenancy such as data partitioning, per-tenant configuration, and so forth.</span></span>

<span data-ttu-id="4e084-172">[**Siguiente**][tailpin]</span><span class="sxs-lookup"><span data-stu-id="4e084-172">[**Next**][tailpin]</span></span>



<!-- Links -->
[ADConnect]: /azure/active-directory/hybrid/whatis-hybrid-identity
[AzureAD]: /azure/active-directory

[Federación con AD FS de un cliente]: adfs.md
[Federating with a customer's AD FS]: adfs.md
[tailpin]: tailspin.md

[running-the-app]: ./run-the-app.md
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
