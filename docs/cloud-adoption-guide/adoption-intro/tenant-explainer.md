---
title: 'Explicación: ¿Qué es un inquilino de Azure Active Directory?'
description: Explica el funcionamiento interno de Azure Active Directory para proporcionar una identidad como servicio (IDaaS) en Azure
author: petertay
ms.openlocfilehash: ce5a33b92047e1f360eee8fcbc7a726bcf8cd19f
ms.sourcegitcommit: 2e8b06e9c07875d65b91d5431bfd4bc465a7a242
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/09/2018
ms.locfileid: "29062043"
---
# <a name="explainer-what-is-an-azure-active-directory-tenant"></a>Explicación: ¿Qué es un inquilino de Azure Active Directory?

En el artículo de la explicación [¿Cómo funciona Azure?](azure-explainer.md), ha aprendido que Azure es una colección de servidores y de hardware de red que ejecutan software y hardware virtualizado en nombre de los usuarios. También ha descubierto que algunos de estos servidores ejecutan una aplicación distribuida de orquestación para administrar, crear, leer, actualizar y eliminar recursos de Azure.

Sin embargo, tal como podría esperar, Azure no permite a nadie realizar ninguna de estas operaciones en un recurso. Azure restringe el acceso a estas operaciones mediante un servicio de identidades digitales de confianza denominado **Azure Active Directory** (Azure AD). Azure AD almacena el nombre de usuario, contraseñas, datos de perfil y otra información. Los usuarios de Azure AD se segmentan en **inquilinos**. Un inquilino es una construcción lógica que representa una instancia segura y dedicada de Azure AD, normalmente asociada a una organización.

Para crear un inquilino, Azure requiere una **cuenta con privilegios**. Esta cuenta con privilegios está asociada a una cuenta de Azure o a un contrato Enterprise. Ambas son construcciones de facturación y no se almacenan en Azure AD: estas cuentas se almacenan en una base de datos de facturación altamente segura. 

Una vez creado el inquilino, se genera un **identificador de inquilino** para el inquilino y se guarda en una base de datos de Azure AD interna altamente segura. Un propietario de una cuenta con privilegios puede luego iniciar sesión en Azure Portal y agregar usuarios a un inquilino de Azure AD recién creado. 

La mayoría de las empresas ya tienen al menos un servicio de administración de identidades, normalmente Active Directory Domain Services (AD DS). Azure AD es capaz de sincronizar o federar identidades de usuario de AD DS, por lo que no es necesario que las empresas administren las identidades por separado en los dos entornos. Esto se va a tratar más detalladamente en los artículos de las fases de adopción intermedia y avanzada para identidades digitales.

## <a name="next-steps"></a>pasos siguientes

* Ahora que ha aprendido acerca de los inquilinos de Azure AD, el primer paso en la fase de adopción básica es aprender a [obtener un inquilino de Azure Active Directory][how-to-get-aad-tenant]. Después, revise la [guía de diseño para inquilinos de Azure AD](tenant.md).

<!-- Links -->
[how-to-get-aad-tenant]: /azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json