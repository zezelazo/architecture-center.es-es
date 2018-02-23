---
title: "Guía: Diseño de inquilinos de Azure AD"
description: "Guía para el diseño de inquilinos de Azure como parte de una estrategia de adopción básica en la nube"
author: telmosampaio
ms.openlocfilehash: 5bf710f74bde04e041f2896b4a638c3513e4aa44
ms.sourcegitcommit: 2e8b06e9c07875d65b91d5431bfd4bc465a7a242
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/09/2018
---
# <a name="guidance-azure-ad-tenant-design"></a>Guía: Diseño de inquilinos de Azure AD

Un inquilino de Azure AD proporciona servicios de identidad digital y espacios de nombres que se utilizan para una o varias [suscripciones de Azure](subscription-explainer.md). Si está siguiendo el esquema de adopción básico, ya sabe [cómo obtener un inquilino de Azure AD][how-to-get-aad-tenant]. 

## <a name="design-considerations"></a>Consideraciones de diseño

- En la fase de adopción básica, puede comenzar con un único inquilino de Azure AD. Si su organización tiene una suscripción de Office 365 o una suscripción de Azure existentes, ya tiene un inquilino de Azure AD que puede usar. Si no dispone de ninguna, puede obtener más información acerca de [cómo obtener un inquilino de Azure AD][how-to-get-aad-tenant]. 
- En las fases de adopción intermedia y avanzada, aprenderá a sincronizar o federar directorios locales con Azure AD. Esto le permitirá utilizar identidades digitales locales en Azure AD. Sin embargo, en la fase básica, agregará nuevos usuarios que solo tienen identidades en su único inquilino de Azure AD. Será responsable de administrar estas identidades. Por ejemplo, tendrá que incorporar a nuevos usuarios de Azure AD, desactivar a usuarios de Azure AD que ya no deseen tener acceso a los recursos de Azure y realizar otros cambios en los permisos de usuario.

## <a name="next-steps"></a>Pasos siguientes

* Ahora que tiene un inquilino de Azure AD, aprenda a [agregar un usuario][azure-ad-add-user]. Después de haber agregado uno o más usuarios nuevos a su inquilino de Azure AD, el siguiente paso es conocer las [suscripciones de Azure](subscription-explainer.md).

<!-- Links -->

[azure-ad-add-user]: /azure/active-directory/add-users-azure-active-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-manage-azure-ad]: /azure/active-directory/active-directory-administer?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-tenant]: /azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-associate-subscription]: /azure/active-directory/active-directory-how-subscriptions-associated-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
[how-to-get-aad-tenant]: /azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json