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
# <a name="explainer-what-is-an-azure-subscription"></a>Explicación: ¿Qué es una suscripción de Azure?

En el artículo de la explicación [¿Qué es un inquilino de Azure Active Directory?](tenant-explainer.md), ha aprendido que las identidades digitales para su organización se almacenan en un inquilino de Azure Active Directory. También ha aprendido que Azure confía en Azure Active Directory a fin de autenticar las solicitudes de usuario para crear, leer, actualizar o eliminar un recurso. 

Comprendemos fundamentalmente por qué es necesario restringir el acceso a estas operaciones en un recurso: para evitar que los usuarios no autenticados y no autorizados tengan acceso a nuestros recursos. Sin embargo, estas operaciones de recursos tienen otras propiedades que una organización desea controlar, como el número de recursos que un usuario o grupo de usuarios tiene permitido crear y el costo para ejecutar estos recursos. 

Azure implementa este control, que se denomina **suscripción**. Una suscripción agrupa juntos a usuarios y recursos creados por estos usuarios. Cada uno de estos recursos contribuye a un [límite total][subscription-service-limits] en ese recurso en particular.

Las organizaciones pueden usar las suscripciones para administrar los costos y la creación de recursos por los usuarios, equipos, proyectos o mediante muchas otras estrategias. Estas estrategias se van a tratar en los artículos de las fases de adopción intermedia y avanzada. 

## <a name="next-steps"></a>pasos siguientes

* Ahora que ha aprendido acerca de las suscripciones de Azure, obtenga más información sobre cómo [crear una suscripción](subscription.md) antes de crear sus primeros recursos de Azure...

<!-- Links -->
[azure-get-started]: https://azure.microsoft.com/en-us/get-started/
[azure-offers]: https://azure.microsoft.com/en-us/support/legal/offer-details/
[azure-free-trial]: https://azure.microsoft.com/en-us/offers/ms-azr-0044p/
[azure-change-subscription-offer]: /azure/billing/billing-how-to-switch-azure-offer
[microsoft-account]: https://account.microsoft.com/account
[subscription-service-limits]: /azure/azure-subscription-service-limits
[docs-organizational-account]: https://docs.microsoft.com/en-us/azure/active-directory/sign-up-organization
