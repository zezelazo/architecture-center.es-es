---
title: Acerca de la aplicación Tailspin Surveys
description: Información general acerca de la aplicación Tailspin Surveys
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: index
pnp.series.next: authenticate
ms.openlocfilehash: 028f7940d2e3cd7e8e629554f8af290ec5fdd184
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2017
---
# <a name="the-tailspin-scenario"></a><span data-ttu-id="1dddb-103">El escenario de Tailspin</span><span class="sxs-lookup"><span data-stu-id="1dddb-103">The Tailspin scenario</span></span>

<span data-ttu-id="1dddb-104">[![GitHub](../_images/github.png) Código de ejemplo][sample application]</span><span class="sxs-lookup"><span data-stu-id="1dddb-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="1dddb-105">Tailspin es una compañía ficticia que está desarrollando una aplicación SaaS llamada Surveys.</span><span class="sxs-lookup"><span data-stu-id="1dddb-105">Tailspin is a fictitious company that is developing a SaaS application named Surveys.</span></span> <span data-ttu-id="1dddb-106">Esta aplicación permite a las organizaciones crear y publicar encuestas en línea.</span><span class="sxs-lookup"><span data-stu-id="1dddb-106">This application enables organizations to create and publish online surveys.</span></span>

* <span data-ttu-id="1dddb-107">Una organización puede suscribirse a la aplicación.</span><span class="sxs-lookup"><span data-stu-id="1dddb-107">An organization can sign up for the application.</span></span>
* <span data-ttu-id="1dddb-108">Después de que la organización se ha suscrito, los usuarios pueden iniciar sesión en la aplicación con las credenciales de la organización.</span><span class="sxs-lookup"><span data-stu-id="1dddb-108">After the organization is signed up, users can sign into the application with their organizational credentials.</span></span>
* <span data-ttu-id="1dddb-109">Los usuarios pueden crear, editar y publicar encuestas.</span><span class="sxs-lookup"><span data-stu-id="1dddb-109">Users can create, edit, and publish surveys.</span></span>

> [!NOTE]
> <span data-ttu-id="1dddb-110">Para empezar a trabajar con la aplicación, consulte [Running the Surveys application ] (Ejecución de la aplicación Surveys).</span><span class="sxs-lookup"><span data-stu-id="1dddb-110">To get started with the application, see [Run the Surveys application].</span></span>
> 
> 

## <a name="users-can-create-edit-and-view-surveys"></a><span data-ttu-id="1dddb-111">Los usuarios pueden crear, editar y visualizar encuestas.</span><span class="sxs-lookup"><span data-stu-id="1dddb-111">Users can create, edit, and view surveys</span></span>
<span data-ttu-id="1dddb-112">Un usuario no autenticado puede ver todas las encuestas que ha creado, o sobre las que tiene derechos de colaborador, y crear nuevas encuestas.</span><span class="sxs-lookup"><span data-stu-id="1dddb-112">An authenticated user can view all the surveys that he or she has created or has contributor rights to, and create new surveys.</span></span> <span data-ttu-id="1dddb-113">Observe que el usuario ha iniciado sesión con su identidad organizativa, `bob@contoso.com`.</span><span class="sxs-lookup"><span data-stu-id="1dddb-113">Notice that the user is signed in with his organizational identity, `bob@contoso.com`.</span></span>

![Aplicación Surveys](./images/surveys-screenshot.png)

<span data-ttu-id="1dddb-115">Esta captura de pantalla muestra la página Edit Survey (Editar encuesta):</span><span class="sxs-lookup"><span data-stu-id="1dddb-115">This screenshot shows the Edit Survey page:</span></span>

![Editar encuesta](./images/edit-survey.png)

<span data-ttu-id="1dddb-117">Los usuarios también pueden ver todas las encuestas creadas por otros usuarios del mismo inquilino.</span><span class="sxs-lookup"><span data-stu-id="1dddb-117">Users can also view any surveys created by other users within the same tenant.</span></span>

![Encuestas de inquilinos](./images/tenant-surveys.png)

## <a name="survey-owners-can-invite-contributors"></a><span data-ttu-id="1dddb-119">Los propietarios de encuestas pueden invitar a colaboradores</span><span class="sxs-lookup"><span data-stu-id="1dddb-119">Survey owners can invite contributors</span></span>
<span data-ttu-id="1dddb-120">Cuando un usuario crea una encuesta, puede invitar a otras personas a ser colaboradores en la misma.</span><span class="sxs-lookup"><span data-stu-id="1dddb-120">When a user creates a survey, he or she can invite other people to be contributors on the survey.</span></span> <span data-ttu-id="1dddb-121">Los colaboradores pueden modificar la encuesta, pero no pueden eliminarla ni publicarla.</span><span class="sxs-lookup"><span data-stu-id="1dddb-121">Contributors can edit the survey, but cannot delete or publish it.</span></span>  

![Agregar colaborador](./images/add-contributor.png)

<span data-ttu-id="1dddb-123">Un usuario puede agregar colaboradores de otros inquilinos, lo que permite compartir recursos entre inquilinos.</span><span class="sxs-lookup"><span data-stu-id="1dddb-123">A user can add contributors from other tenants, which enables cross-tenant sharing of resources.</span></span> <span data-ttu-id="1dddb-124">En esta captura de pantalla, Bob (`bob@contoso.com`) está agregando a Alice (`alice@fabrikam.com`) como colaboradora a una encuesta que Bob ha creado.</span><span class="sxs-lookup"><span data-stu-id="1dddb-124">In this screenshot, Bob (`bob@contoso.com`) is adding Alice (`alice@fabrikam.com`) as a contributor to a survey that Bob created.</span></span>

<span data-ttu-id="1dddb-125">Cuando Alice inicia sesión, ve la encuesta que aparece en la lista de "Surveys I can contribute to" (Encuestas en las que puedo colaborar).</span><span class="sxs-lookup"><span data-stu-id="1dddb-125">When Alice logs in, she sees the survey listed under "Surveys I can contribute to".</span></span>

![Colaborador de la encuesta](./images/contributor.png)

<span data-ttu-id="1dddb-127">Tenga en cuenta que Alice inicia sesión en su propio inquilino, no como un invitado del inquilino Contoso.</span><span class="sxs-lookup"><span data-stu-id="1dddb-127">Note that Alice signs into her own tenant, not as a guest of the Contoso tenant.</span></span> <span data-ttu-id="1dddb-128">Alice dispone de permisos de colaborador solo para esa encuesta pero no puede ver otras encuestas desde el inquilino Contoso.</span><span class="sxs-lookup"><span data-stu-id="1dddb-128">Alice has contributor permissions only for that survey &mdash; she cannot view other surveys from the Contoso tenant.</span></span>

## <a name="architecture"></a><span data-ttu-id="1dddb-129">Arquitectura</span><span class="sxs-lookup"><span data-stu-id="1dddb-129">Architecture</span></span>
<span data-ttu-id="1dddb-130">La aplicación Surveys consta de un front-end web y un back-end de API web.</span><span class="sxs-lookup"><span data-stu-id="1dddb-130">The Surveys application consists of a web front end and a web API backend.</span></span> <span data-ttu-id="1dddb-131">Ambos se implementan mediante [ASP.NET Core].</span><span class="sxs-lookup"><span data-stu-id="1dddb-131">Both are implemented using [ASP.NET Core].</span></span>

<span data-ttu-id="1dddb-132">La aplicación web utiliza Azure Active Directory (Azure AD) para autenticar a los usuarios.</span><span class="sxs-lookup"><span data-stu-id="1dddb-132">The web application uses Azure Active Directory (Azure AD) to authenticate users.</span></span> <span data-ttu-id="1dddb-133">La aplicación web también llama a Azure AD para obtener tokens de acceso de OAuth 2 para la API Web.</span><span class="sxs-lookup"><span data-stu-id="1dddb-133">The web application also calls Azure AD to get OAuth 2 access tokens for the Web API.</span></span> <span data-ttu-id="1dddb-134">Los tokens de acceso se almacenan en Azure Redis Cache.</span><span class="sxs-lookup"><span data-stu-id="1dddb-134">Access tokens are cached in Azure Redis Cache.</span></span> <span data-ttu-id="1dddb-135">La memoria caché habilita a varias instancias para compartir la misma caché de token (por ejemplo, en una granja de servidores).</span><span class="sxs-lookup"><span data-stu-id="1dddb-135">The cache enables multiple instances to share the same token cache (e.g., in a server farm).</span></span>

![Arquitectura](./images/architecture.png)

<span data-ttu-id="1dddb-137">[**Siguiente**][authentication]</span><span class="sxs-lookup"><span data-stu-id="1dddb-137">[**Next**][authentication]</span></span>

<!-- Links -->

[authentication]: authenticate.md

[Running the Surveys application ]: ./run-the-app.md
[Run the Surveys application]: ./run-the-app.md
[ASP.NET Core]: /aspnet/core
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
