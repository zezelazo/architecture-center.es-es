---
title: 'Guía: Diseño de inquilinos de Azure AD'
description: Guía para el diseño de inquilinos de Azure como parte de una estrategia de adopción básica en la nube
author: telmosampaio
ms.openlocfilehash: 9ac52e9fd44bd8b9c777625002d5960f4f269be2
ms.sourcegitcommit: 29fbcb1eec44802d2c01b6d3bcf7d7bd0bae65fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/27/2018
---
# <a name="guidance-azure-ad-tenant-design"></a><span data-ttu-id="57cad-103">Guía: Diseño de inquilinos de Azure AD</span><span class="sxs-lookup"><span data-stu-id="57cad-103">Guidance: Azure AD tenant design</span></span>

<span data-ttu-id="57cad-104">Un inquilino de Azure AD proporciona servicios de identidad digital y espacios de nombres que se utilizan para una o varias [suscripciones de Azure](subscription-explainer.md).</span><span class="sxs-lookup"><span data-stu-id="57cad-104">An Azure AD tenant provides digital identity services and namespaces used for one or more [Azure subscriptions](subscription-explainer.md).</span></span> <span data-ttu-id="57cad-105">Si está siguiendo el esquema de adopción básico, ya sabe [cómo obtener un inquilino de Azure AD][how-to-get-aad-tenant].</span><span class="sxs-lookup"><span data-stu-id="57cad-105">If you are following the foundational adoption outline, you have already learned [how to get an Azure AD tenant][how-to-get-aad-tenant].</span></span> 

## <a name="design-considerations"></a><span data-ttu-id="57cad-106">Consideraciones de diseño</span><span class="sxs-lookup"><span data-stu-id="57cad-106">Design considerations</span></span>

- <span data-ttu-id="57cad-107">En la fase de adopción básica, puede comenzar con un único inquilino de Azure AD.</span><span class="sxs-lookup"><span data-stu-id="57cad-107">At the foundational adoption stage, you can begin with a single Azure AD tenant.</span></span> <span data-ttu-id="57cad-108">Si su organización tiene una suscripción de Office 365 o una suscripción de Azure existentes, ya tiene un inquilino de Azure AD que puede usar.</span><span class="sxs-lookup"><span data-stu-id="57cad-108">If your organization has an existing Office 365 subscription or an Azure subscription, you already have an Azure AD tenant that you can use.</span></span> <span data-ttu-id="57cad-109">Si no dispone de ninguna, puede obtener más información acerca de [cómo obtener un inquilino de Azure AD][how-to-get-aad-tenant].</span><span class="sxs-lookup"><span data-stu-id="57cad-109">If you do not have either or of these, you can learn more about [how to get an Azure AD tenant][how-to-get-aad-tenant].</span></span> 
- <span data-ttu-id="57cad-110">En las fases de adopción intermedia y avanzada, aprenderá a sincronizar o federar directorios locales con Azure AD.</span><span class="sxs-lookup"><span data-stu-id="57cad-110">In the intermediate and advanced adoption stages, you will learn how to synchronize or federate on-premises directories with Azure AD.</span></span> <span data-ttu-id="57cad-111">Esto le permitirá utilizar identidades digitales locales en Azure AD.</span><span class="sxs-lookup"><span data-stu-id="57cad-111">This will allow you to use on-premises digital identity in Azure AD.</span></span> <span data-ttu-id="57cad-112">Sin embargo, en la fase básica, agregará nuevos usuarios que solo tienen identidades en su único inquilino de Azure AD.</span><span class="sxs-lookup"><span data-stu-id="57cad-112">However, at the foundational stage, you will be adding new users that only have identity your single Azure AD tenant.</span></span> <span data-ttu-id="57cad-113">Será responsable de administrar estas identidades.</span><span class="sxs-lookup"><span data-stu-id="57cad-113">You will be responsible for managing those identities.</span></span> <span data-ttu-id="57cad-114">Por ejemplo, tendrá que incorporar a nuevos usuarios de Azure AD, desactivar a usuarios de Azure AD que ya no deseen tener acceso a los recursos de Azure y realizar otros cambios en los permisos de usuario.</span><span class="sxs-lookup"><span data-stu-id="57cad-114">For example, you will have to on-board new Azure AD users, off-board Azure AD users that you no longer wish to have access to Azure resources, and other changes to user permissions.</span></span>

## <a name="next-steps"></a><span data-ttu-id="57cad-115">Pasos siguientes</span><span class="sxs-lookup"><span data-stu-id="57cad-115">Next Steps</span></span>

* <span data-ttu-id="57cad-116">Ahora que tiene un inquilino de Azure AD, aprenda a [agregar un usuario][azure-ad-add-user].</span><span class="sxs-lookup"><span data-stu-id="57cad-116">Now that you have an Azure AD tenant, learn [how to add a user][azure-ad-add-user].</span></span> <span data-ttu-id="57cad-117">Después de haber agregado uno o más usuarios nuevos a su inquilino de Azure AD, el siguiente paso es conocer las [suscripciones de Azure](subscription-explainer.md).</span><span class="sxs-lookup"><span data-stu-id="57cad-117">After you have added one or more new users to your Azure AD tenant, your next step is learning about [Azure subscriptions](subscription-explainer.md).</span></span>

<!-- Links -->

[azure-ad-add-user]: /azure/active-directory/add-users-azure-active-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-manage-azure-ad]: /azure/active-directory/active-directory-administer?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-tenant]: /azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-associate-subscription]: /azure/active-directory/active-directory-how-subscriptions-associated-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
[how-to-get-aad-tenant]: /azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json
