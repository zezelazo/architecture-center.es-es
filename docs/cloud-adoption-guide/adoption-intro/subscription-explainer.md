---
title: "Explicación: ¿Qué es una suscripción de Azure?"
description: Explica las suscripciones, cuentas y ofertas de Azure
author: abuck
ms.openlocfilehash: 7b88e9489b40b100eecb76602b45901566b3f37f
ms.sourcegitcommit: 2e8b06e9c07875d65b91d5431bfd4bc465a7a242
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/09/2018
---
# <a name="explainer-what-is-an-azure-subscription"></a><span data-ttu-id="66b77-103">Explicación: ¿Qué es una suscripción de Azure?</span><span class="sxs-lookup"><span data-stu-id="66b77-103">Explainer: What is an Azure subscription?</span></span>

<span data-ttu-id="66b77-104">En el artículo de la explicación [¿Qué es un inquilino de Azure Active Directory?](tenant-explainer.md), ha aprendido que las identidades digitales para su organización se almacenan en un inquilino de Azure Active Directory.</span><span class="sxs-lookup"><span data-stu-id="66b77-104">In the [what is an Azure Active Directory tenant?](tenant-explainer.md) explainer article, you learned that digital identity for your organization is stored in an Azure Active Directory tenant.</span></span> <span data-ttu-id="66b77-105">También ha aprendido que Azure confía en Azure Active Directory a fin de autenticar las solicitudes de usuario para crear, leer, actualizar o eliminar un recurso.</span><span class="sxs-lookup"><span data-stu-id="66b77-105">You also learned that Azure trusts Azure Active Directory to authenticate user requests to create, read, update, or delete a resource.</span></span> 

<span data-ttu-id="66b77-106">Comprendemos fundamentalmente por qué es necesario restringir el acceso a estas operaciones en un recurso: para evitar que los usuarios no autenticados y no autorizados tengan acceso a nuestros recursos.</span><span class="sxs-lookup"><span data-stu-id="66b77-106">We fundamentally understand why it's necessary to restrict access to these operations on a resource - to prevent unauthenticated and unauthorized users from accessing our resources.</span></span> <span data-ttu-id="66b77-107">Sin embargo, estas operaciones de recursos tienen otras propiedades que una organización desea controlar, como el número de recursos que un usuario o grupo de usuarios tiene permitido crear y el costo para ejecutar estos recursos.</span><span class="sxs-lookup"><span data-stu-id="66b77-107">However, these resource operations have other properties that an organization would like to control, such as the number of resources a user or group of users is allowed to create, and, the cost to run those resources.</span></span> 

<span data-ttu-id="66b77-108">Azure implementa este control, que se denomina **suscripción**.</span><span class="sxs-lookup"><span data-stu-id="66b77-108">Azure implements this control, and it is named a **subscription**.</span></span> <span data-ttu-id="66b77-109">Una suscripción agrupa juntos a usuarios y recursos creados por estos usuarios.</span><span class="sxs-lookup"><span data-stu-id="66b77-109">A subscription groups together users and the resources that have been created by those users.</span></span> <span data-ttu-id="66b77-110">Cada uno de estos recursos contribuye a un [límite total][subscription-service-limits] en ese recurso en particular.</span><span class="sxs-lookup"><span data-stu-id="66b77-110">Each of those resources contributes to an [overall limit][subscription-service-limits] on that particular resource.</span></span>

<span data-ttu-id="66b77-111">Las organizaciones pueden usar las suscripciones para administrar los costos y la creación de recursos por los usuarios, equipos, proyectos o mediante muchas otras estrategias.</span><span class="sxs-lookup"><span data-stu-id="66b77-111">Organizations can use subscriptions to manage costs and creation of resource by users, teams, projects, or using many other strategies.</span></span> <span data-ttu-id="66b77-112">Estas estrategias se van a tratar en los artículos de las fases de adopción intermedia y avanzada.</span><span class="sxs-lookup"><span data-stu-id="66b77-112">These strategies will be discussed in the intermediate and advanced adoption stage articles.</span></span> 

## <a name="next-steps"></a><span data-ttu-id="66b77-113">pasos siguientes</span><span class="sxs-lookup"><span data-stu-id="66b77-113">Next steps</span></span>

* <span data-ttu-id="66b77-114">Ahora que ha aprendido acerca de las suscripciones de Azure, obtenga más información sobre cómo [crear una suscripción](subscription.md) antes de crear sus primeros recursos de Azure...</span><span class="sxs-lookup"><span data-stu-id="66b77-114">Now that you have learned about Azure subscriptions, learn more about [creating a subscription](subscription.md) before you create your first Azure resources..</span></span>

<!-- Links -->
[azure-get-started]: https://azure.microsoft.com/en-us/get-started/
[azure-offers]: https://azure.microsoft.com/en-us/support/legal/offer-details/
[azure-free-trial]: https://azure.microsoft.com/en-us/offers/ms-azr-0044p/
[azure-change-subscription-offer]: /azure/billing/billing-how-to-switch-azure-offer
[microsoft-account]: https://account.microsoft.com/account
[subscription-service-limits]: /azure/azure-subscription-service-limits
[docs-organizational-account]: https://docs.microsoft.com/en-us/azure/active-directory/sign-up-organization
