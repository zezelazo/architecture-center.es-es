---
title: Administración de identidades en aplicaciones multiinquilino
description: Procedimientos recomendados para autenticación, autorización y administración de identidades en aplicaciones multiinquilino.
author: MikeWasson
ms.date: 07/21/2017
ms.openlocfilehash: 864317cc98ee0211d4f4274253eda12b72beceed
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/08/2019
ms.locfileid: "54114154"
---
# <a name="manage-identity-in-multitenant-applications"></a>Administración de identidades en aplicaciones multiinquilino

En esta serie se artículos se describen los procedimientos recomendados para la arquitectura multiinquilino durante el uso de Azure AD para la autenticación y la administración de identidades.

[![GitHub](../_images/github.png) Código de ejemplo][sample-application]

Cuando está creando una aplicación multiinquilino, uno de los primeros desafíos es administrar las identidades de usuario, ya que ahora todos los usuarios pertenecen a un inquilino. Por ejemplo: 

- Los usuarios inician sesión con las credenciales de la organización.
- Los usuarios deben tener acceso a datos de su organización, pero no a los datos que pertenecen a otros inquilinos.
- Una organización puede suscribirse a la aplicación y, a continuación, asignar roles de aplicación a sus miembros.

Azure Active Directory (Azure AD) tiene algunas características excelentes que admiten todos estos escenarios.

Para acompañar esta serie de artículos, se ha creado una [implementación de un extremo a otro][sample-application] completa de una aplicación multiinquilino. Los artículos reflejan lo que hemos aprendido en el proceso de creación de la aplicación. Para empezar a trabajar con la aplicación, consulte [Running the Surveys application ][running-the-app] (Ejecución de la aplicación Surveys).

## <a name="introduction"></a>Introducción

Supongamos que está escribiendo una aplicación de SaaS empresarial que se hospedará en la nube. Por supuesto, la aplicación tendrá usuarios:

![Usuarios](./images/users.png)

Sin embargo, los usuarios pertenecen a las organizaciones:

![Usuarios de organización](./images/org-users.png)

Ejemplo: Tailspin oferta suscripciones a su aplicación de SaaS. Contoso y Fabrikam se registran en la aplicación. Cuando Alice (`alice@contoso`) inicia sesión, la aplicación debe saber que esta usuaria forma parte de Contoso.

- Alice *debería* tener acceso a datos de Contoso.
- Alice *no debería* tener acceso a los datos de Fabrikam.

Esta guía muestra cómo administrar identidades de usuario en una aplicación multiinquilino, usando [Azure Active Directory](/azure/active-directory) (Azure AD) para controlar el inicio de sesión y la autenticación.

<!-- markdownlint-disable MD026 -->

## <a name="what-is-multitenancy"></a>¿Qué significa multiinquilino?

<!-- markdownlint-enable MD026 -->

Un *inquilino* es un grupo de usuarios. En una aplicación SaaS, el inquilino es un suscriptor o un usuario de la aplicación. *arquitectura multiinquilino* es una arquitectura en la que varios inquilinos comparten la misma instancia física de la aplicación. Aunque los inquilinos comparten los recursos físicos (como las máquinas virtuales o el almacenamiento), cada inquilino obtiene su propia instancia lógica de la aplicación.

Normalmente, los datos de la aplicación se comparten entre los usuarios de un mismo inquilino, pero no con otros inquilinos.

![Multiinquilino](./images/multitenant.png)

Compare esta arquitectura con otra de un solo inquilino, donde cada inquilino tiene una instancia física dedicada. En una arquitectura de un solo inquilino, los nuevos inquilinos se agregan mediante la puesta en marcha de nuevas instancias de la aplicación.

![Un solo inquilino](./images/single-tenant.png)

### <a name="multitenancy-and-horizontal-scaling"></a>Arquitectura multiinquilino y escalado horizontal

Para lograr la reducción horizontal en la nube, es habitual agregar más instancias físicas. Esto se conoce como *ampliación horizontal* o *escalado horizontal*. Piense en una aplicación web. Para administrar más tráfico, puede agregar más máquinas virtuales de servidor y colocarlas detrás de un equilibrador de carga. Cada máquina virtual ejecuta una instancia física independiente de la aplicación web.

![Equilibrio de carga de un sitio web](./images/load-balancing.png)

Cualquier solicitud puede enrutarse a cualquier instancia. En conjunto, el sistema funciona como una única instancia lógica. Puede anular una máquina virtual o poner en marcha una nueva sin que los usuarios resulten afectados. En esta arquitectura, cada instancia física es multiinquilino, y se escala mediante la incorporación de más instancias. Si una instancia deja de funcionar, ningún inquilino debería resultar afectado.

## <a name="identity-in-a-multitenant-app"></a>Identidad en una aplicación multiinquilino

En una aplicación multiinquilino, debe tener en cuenta a los usuarios en el contexto de los inquilinos.

### <a name="authentication"></a>Autenticación

- Los usuarios inician sesión en la aplicación con sus credenciales de la organización. No es necesario que creen nuevos perfiles de usuario para la aplicación.
- Los usuarios de la misma organización forman parte del mismo inquilino.
- Cuando un usuario inicia sesión, la aplicación sabe a qué inquilino pertenece el usuario.

### <a name="authorization"></a>Autorización

- Al autorizar las acciones de un usuario (por ejemplo, ver un recurso), la aplicación debe tener en cuenta el inquilino del usuario.
- Es necesario asignar roles a los usuarios dentro de la aplicación, como "Administrador" o "Usuario estándar". Las asignaciones de roles deben administrarse por el cliente, no por el proveedor de SaaS.

**Ejemplo:** Alice, una empleada de Contoso, navega a la aplicación en su explorador y hace clic en el botón "Iniciar sesión". Se le redirige a una pantalla de inicio de sesión donde escribe sus credenciales corporativas (nombre de usuario y contraseña). En este momento, ha iniciado sesión en la aplicación como `alice@contoso.com`. La aplicación también sabe que Alice es un usuario administrador en esta aplicación. Puesto que es administrador, puede ver una lista de todos los recursos que pertenecen a Contoso. Sin embargo, no puede ver recursos de Fabrikam, puesto que solo es administrador dentro de su inquilino.

En esta guía, examinaremos específicamente el uso de Azure AD para la administración de identidades.

- Se supone que el cliente almacena sus perfiles de usuario en Azure AD (incluidos los inquilinos de Office 365 y Dynamics CRM).
- Los clientes con Active Directory (AD) local pueden usar [Azure AD Connect](/azure/active-directory/hybrid/whatis-hybrid-identity) para sincronizar su instancia de AD local con Azure AD.

Si un cliente con AD local no puede usar Azure AD Connect (debido a la directiva de TI corporativa u otras razones), el proveedor de SaaS puede federarse con el AD del cliente a través de los Servicios de federación de Active Directory (AD FS). Esta opción se describe en [Federación con AD FS de un cliente](adfs.md).

Esta guía no tiene en cuenta otros aspectos de la arquitectura multiinquilino como la partición de datos, la configuración de cada inquilino, etc.

[**Next**](./tailspin.md)

<!-- links -->

[sample-application]: https://github.com/mspnp/multitenant-saas-guidance
[running-the-app]: ./run-the-app.md
