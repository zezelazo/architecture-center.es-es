---
title: 'Adopción de la nube empresarial: Administración de acceso a los recursos en Azure'
description: 'Explica los elementos de la administración de acceso a los recursos en Azure: Azure Resource Manager, suscripciones, grupos de recursos y recursos'
author: petertaylor9999
ms.openlocfilehash: cd26b73e0327fa15b6ae29492b45331a19b9d6c2
ms.sourcegitcommit: ae8a1de6f4af7a89a66a8339879843d945201f85
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/31/2018
ms.locfileid: "43327081"
---
# <a name="enterprise-cloud-adoption-resource-access-management-in-azure"></a>Adopción de la nube empresarial: Administración de acceso a los recursos en Azure

Como ya hemos aprendido en [¿Qué es el gobierno de los recursos en la nube?](what-is-governance.md), por gobierno se entiende el proceso continuo de administrar, supervisar y auditar el uso de los recursos de Azure para cumplir los objetivos y requisitos de su organización. Antes de continuar y aprender cómo se diseña un modelo de gobierno, es importante comprender los controles de administración de acceso a los recursos de Azure. La configuración de estos controles de administración de acceso a los recursos constituye la base de su modelo de gobierno.

Para empezar, veamos con más detenimiento cómo se implementan los recursos en Azure. 

## <a name="what-is-an-azure-resource"></a>¿Qué es un recurso de Azure?

En Azure, el término **recurso** hace referencia a una entidad administrada por Azure. Por ejemplo, las máquinas virtuales, las redes virtuales y las cuentas de almacenamiento se denominan recursos de Azure.

![](../_images/governance-1-9.png)   
*Ilustración 1. Un recurso.*

## <a name="what-is-an-azure-resource-group"></a>¿Qué es un grupo de recursos de Azure?

En Azure, todos los recursos deben pertenecer a un [grupo de recursos](/azure/azure-resource-manager/resource-group-overview#resource-groups). Un grupo de recursos es simplemente una construcción lógica que agrupa varios recursos para poder administrarlos como una sola entidad. Por ejemplo, los recursos que comparten un ciclo de vida similar, como los recursos para una [aplicación de n niveles](/azure/architecture/guide/architecture-styles/n-tier), pueden crearse o eliminarse como un grupo. 

![](../_images/governance-1-10.png)   
*Ilustración 2. Un grupo de recursos contiene un recurso.* 

Los grupos de recursos, y los recursos que contienen, están asociados a una **suscripción** de Azure. 

## <a name="what-is-an-azure-subscription"></a>¿Qué es una suscripción de Azure?

Una suscripción de Azure es similar a un grupo de recursos en cuanto que se trata de una construcción lógica que agrupa los grupos de recursos y sus recursos. Sin embargo, una suscripción de Azure también está asociada con los controles que utiliza Azure Resource Manager. ¿Qué significa? Veamos más de cerca Azure Resource Manager y cómo se relaciona con una suscripción de Azure.

![](../_images/governance-1-11.png)   
*Ilustración 3. Una suscripción de Azure.*

## <a name="what-is-azure-resource-manager"></a>¿Qué es Azure Resource Manager?

En [¿Cómo funciona Azure?](what-is-azure.md), aprendió que Azure incluye un "front-end" con muchos servicios que orquestan todas las funciones de Azure. Uno de estos servicios es [Azure Resource Manager](/azure/azure-resource-manager/), y hospeda la API RESTful que los clientes utilizan para administrar los recursos. 

![](../_images/governance-1-12.png)   
*Ilustración 4. Azure Resource Manager.*

La siguiente ilustración muestra tres clientes: [Powershell](/powershell/azure/overview), [Azure Portal](https://portal.azure.com) y la [interfaz de la línea de comandos (CLI) de Azure](/cli/azure):

![](../_images/governance-1-13.png)   
*Ilustración 5. Los clientes de Azure conectan con la API RESTful de Azure Resource Manager.*

Aunque estos clientes se conectan a Azure Resource Manager mediante la API RESTful, Azure Resource Manager no incluye una funcionalidad para administrar los recursos directamente. En su lugar, la mayoría de los tipos de recursos de Azure tienen sus propios [**proveedores de recursos**](/azure/azure-resource-manager/resource-group-overview#terminology). 

![](../_images/governance-1-14.png)   
*Ilustración 6. Proveedores de recursos de Azure.*

Cuando un cliente realiza una solicitud para administrar un recurso específico, Azure Resource Manager conecta con el proveedor de recursos para ese tipo de recurso para completar la solicitud. Por ejemplo, si un cliente realiza una solicitud para administrar un recurso de máquina virtual, Azure Resource Manager conecta con el proveedor de recursos **Microsoft.compute**. 

![](../_images/governance-1-15.png)   
*Ilustración 7. Azure Resource Manager conecta con el proveedor de recursos **Microsoft.compute** para administrar el recurso especificado en la solicitud del cliente.*

Azure Resource Manager requiere que el cliente especifique un identificador de suscripción y de grupo de recursos para poder administrar el recurso de máquina virtual. 

Ahora que ya comprende cómo funciona Azure Resource Manager, volvamos a nuestra explicación de cómo una suscripción de Azure está asociada con los controles utilizados por Azure Resource Manager. Antes de que Azure Resource Manager pueda ejecutar ninguna solicitud de administración de recursos, se comprueban un conjunto de controles. 

El primer control es que la solicitud debe realizarla un usuario validado y Azure Resource Manager tiene una relación de confianza con [Azure Active Directory (Azure AD)](/azure/active-directory/), que proporciona la funcionalidad de identidades de usuario.

![](../_images/governance-1-16.png)   
*Ilustración 8. Azure Active Directory.*

Los usuarios de Azure AD se segmentan en **inquilinos**. Un inquilino es una construcción lógica que representa una instancia segura y dedicada de Azure AD, normalmente asociada a una organización. Cada suscripción está asociada a un inquilino de Azure AD.

![](../_images/governance-1-17.png)   
*Ilustración 9. Inquilino de Azure AD asociado a una suscripción.*

Para poder procesar solicitudes de cliente para administrar un recurso en una determinada suscripción, el usuario debe tener una cuenta en el inquilino de Azure AD. 

El control siguiente es una comprobación de que el usuario tiene permisos suficientes para realizar la solicitud. Los permisos se asignan a los usuarios con el [control de acceso basado en rol (RBAC)](/azure/role-based-access-control/).

![](../_images/governance-1-18.png)   
*Ilustración 10. Cada usuario del inquilino se asigna a uno o varios roles de RBAC.*

Un rol de RBAC especifica un conjunto de permisos que un usuario puede tener en un recurso específico. Cuando se asigna el rol al usuario, se aplican estos permisos. Por ejemplo, el [rol integrado **propietario**](/azure/role-based-access-control/built-in-roles#owner) permite a un usuario realizar cualquier acción en un recurso.

El siguiente control es una comprobación de que la solicitud se permite de acuerdo con la configuración especificada para la [directiva de recursos de Azure](/azure/azure-policy/). Las directivas de recursos de Azure especifican las operaciones permitidas para un recurso específico. Por ejemplo, una directiva de recursos de Azure puede especificar que los usuarios solo pueden implementar un tipo específico de máquina virtual.

![](../_images/governance-1-19.png)   
*Ilustración 11. Directiva de recursos de Azure.*

El siguiente control es una comprobación de que la solicitud no supera un [límite de suscripción de Azure](/azure/azure-subscription-service-limits). Por ejemplo, cada suscripción tiene un límite de 980 grupos de recursos por suscripción. Si se recibe una solicitud para implementar un grupo de recursos adicional una vez alcanzado el límite, se deniega.

![](../_images/governance-1-20.png)   
*Ilustración 12. Límites de recursos de Azure.* 

El control final es una comprobación de que la solicitud está dentro del compromiso financiero asociado a la suscripción. Por ejemplo, si la solicitud es para implementar una máquina virtual, Azure Resource Manager comprueba que la suscripción tiene suficiente información de pago.

![](../_images/governance-1-21.png)   
*Ilustración 13. Una suscripción tiene asociado un compromiso financiero.*

## <a name="summary"></a>Resumen

En este artículo, ha aprendido cómo se administra el acceso a los recursos en Azure mediante Azure Resource Manager.

## <a name="next-steps"></a>Pasos siguientes

Ahora que comprende cómo se administra el acceso a los recursos en Azure, aprenda cómo diseñar un modelo de gobierno [para un único equipo](../governance/governance-single-team.md) o para [varios equipos](../governance/governance-multiple-teams.md) con estos servicios.

> [!div class="nextstepaction"]
> [Introducción al gobierno](../governance/overview.md)
