---
title: Identidad federada
description: La autenticación se delega a un proveedor de identidad externo.
keywords: Patrón de diseño
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- security
ms.openlocfilehash: a1edbdd080309383201d33e73602e2f18928c080
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2017
---
# <a name="federated-identity-pattern"></a>Patrón Federated Identity

[!INCLUDE [header](../_includes/header.md)]

La autenticación se delega a un proveedor de identidad externo. Esto puede simplificar el desarrollo, minimizar los requisitos de administración de usuarios y mejorar la experiencia del usuario de la aplicación.

## <a name="context-and-problem"></a>Contexto y problema

Normalmente, los usuarios deben trabajar con varias aplicaciones que proporcionan y hospedan diversas organizaciones con las que mantienen una relación de negocios. A estos usuarios se les puede exigir que usen cada uno de ellos credenciales específicas (y diferentes). Esta situación puede dar lugar a:

- **Una experiencia de usuario desestructurada**. Con frecuencia, los usuarios olvidan las credenciales de inicio de sesión cuando si tienen muchas diferentes.

- **Exposición de las vulnerabilidades de seguridad**. Cuando un usuario abandona la empresa, la cuenta se debe desaprovisionar inmediatamente. En organizaciones de gran tamaño, es fácil pasarlo por alto.

- **Administración de usuarios complicada**. Los administradores deben administrar las credenciales de todos los usuarios y realizar tareas adicionales, como proporcionar recordatorios de contraseña.

Los usuarios prefieren normalmente usar las mismas credenciales en todas estas aplicaciones.

## <a name="solution"></a>Solución

Implemente un mecanismo de autenticación que pueda usar la identidad federada. Separe la autenticación de usuarios del código de aplicación, y delegue la autentificación a un proveedor de identidades de confianza. Esto puede simplificar el desarrollo y permitir que los usuarios se autentiquen mediante una variedad más amplia de proveedores de identidades (IdP) al tiempo que se reduce la sobrecarga administrativa. También permite separar claramente la autenticación de la autorización.

Entre los proveedores de identidades de confianza se incluyen directorios corporativos, servicios de federación locales, otros servicios de token de seguridad (STS) que proporcionan los asociados comerciales o proveedores de identidades sociales que pueden autenticar a los usuarios que tienen, por ejemplo, una cuenta de Microsoft, Google, Yahoo! o Facebook.

En la siguiente figura se ilustra el patrón Federated Identity (Identidad federada) cuando una aplicación cliente necesita acceder a un servicio que requiere autentificación. La autenticación se realiza mediante un IdP que trabaja en combinación con un STS. El IdP emite tokens de seguridad que proporcionan información sobre el usuario autenticado. Esta información, conocida como notificaciones, incluye la identidad del usuario y podría incluir también otra información como la pertenencia a roles y derechos de acceso más específicos.

![Introducción a la autenticación federada](./_images/federated-identity-overview.png)


Este modelo se conoce a menudo como control de acceso basado en notificaciones. Las aplicaciones y los servicios autorizarán el acceso a las características y funcionalidades en función de las notificaciones contenidas en el token. El servicio que requiere autenticación debe confiar en el IdP. La aplicación cliente se pone en contacto el IdP que realiza la autenticación. Si la autenticación es correcta, el IdP devuelve un token que contiene las notificaciones que identifican al usuario en el STS (tenga en cuenta que el IdP y el STS pueden ser el mismo servicio). El STS puede transformar y aumentar las notificaciones en el token según reglas predefinidas, antes de devolverlo al cliente. La aplicación cliente puede luego pasar este token al servicio como prueba de su identidad.

> Podría haber STS adicionales en la cadena de confianza. Por ejemplo, en el escenario descrito más adelante, un STS local confía en otro STS que es responsable de acceder a un proveedor de identidades para autenticar al usuario. Este enfoque es común en escenarios empresariales donde hay un STS y un directorio locales.

La autenticación federada proporciona una solución basada en estándares al problema de confiar en identidades de dominios diferentes y puede admitir el inicio de sesión único. Cada vez es más común en todos los tipos de aplicaciones, especialmente en las aplicaciones hospedadas en la nube, ya que admite el inicio de sesión único sin necesidad de una conexión de red directa a los proveedores de identidades. El usuario no tiene que especificar credenciales en cada aplicación. La seguridad aumenta porque impide la creación de las credenciales necesarias para acceder a muchas aplicaciones diferentes, y también oculta las credenciales del usuario de todos excepto del proveedor de identidades original. Las aplicaciones ven solo la información de la identidad autenticada contenida en el token.

La identidad federada tiene la ventaja principal de que la administración de identidades y credenciales es responsabilidad del proveedor de identidades. La aplicación o el servicio no necesitan proporcionar características de administración de identidades. Además, en escenarios corporativos, el directorio corporativo no necesita saber nada del usuario si confía en el proveedor de identidades. Como consecuencia, se elimina la sobrecarga administrativa de la administración de la identidad del usuario dentro del directorio.

## <a name="issues-and-considerations"></a>Problemas y consideraciones

Al diseñar aplicaciones que implementan la autenticación federada, considere los siguientes puntos:

- La autenticación puede ser un único punto de error. Si implementa la aplicación en varios centros de datos, considere la posibilidad de implementar el mecanismo de administración de identidades en los mismos centros de datos para mantener la disponibilidad y confiabilidad de la aplicación.

- Las herramientas de autenticación le permite configurar el control de acceso en función de las notificaciones de rol contenidas en el token de autenticación. Con frecuencia a esto se le conoce como control de acceso basado en roles (RBAC) y puede permitir un nivel de control más específico sobre el acceso a las características y recursos.

- A diferencia de los directorios corporativos, autenticación basada en notificaciones mediante proveedores de identidades sociales normalmente no proporciona información sobre el usuario autenticado, a excepción de una dirección de correo electrónico y quizás un nombre. Algunos proveedores de identidades sociales, como una cuenta de Microsoft, proporcionan solo un identificador único. Normalmente, la aplicación debe mantener cierta información de los usuarios registrados y poder relacionar esta información con el identificador contenido en las notificaciones del token. Por lo general, esto se consigue mediante el registro cuando el usuario accede a la aplicación por primera vez, y la información se introduce en el token como notificaciones adicionales después de cada autenticación.

- Si hay más de un proveedor de identidades configurado para el STS, este debe detectar el proveedor de identidades al que se debe redirigir al usuario para la autenticación. Este proceso se denomina detección de dominios de inicio. El STS podría realizar este proceso automáticamente en función de una dirección de correo electrónico o un nombre de usuario que proporcione el usuario, un subdominio de la aplicación a la que accede el usuario, el ámbito de la dirección IP del usuario o el contenido de una cookie almacenada en el explorador del usuario. Por ejemplo, si el usuario escribió una dirección de correo electrónico en el dominio de Microsoft, como user@live.com, el STS redirigirá al usuario a la página de inicio de sesión de cuenta de Microsoft. En visitas posteriores, el STS podría usar una cookie para indicar que el último inicio de sesión fue con una cuenta de Microsoft. Si la detección automática no puede determinar el dominio de inicio, el STS mostrará una página de detección de dominios de inicio con los proveedores de identidades de confianza, entre los cuales el usuario debe seleccionar el que quiera usar.

## <a name="when-to-use-this-pattern"></a>Cuándo usar este patrón

Este patrón es útil en escenarios como:

- **Inicio de sesión único en la empresa**. En este escenario debe autenticar a los empleados en las aplicaciones corporativas que se hospedan en la nube fuera del límite de seguridad corporativo, sin exigirles que inicien sesión cada vez que visitan una aplicación. La experiencia de los usuarios es la misma que cuando usan aplicaciones locales donde se autentican al iniciar sesión en una red corporativa y desde ese momento tienen acceso a todas las aplicaciones pertinentes sin necesidad de iniciar de nuevo la sesión.

- **Identidad federada con varios asociados**. En este escenario, debe autenticar a los usuarios corporativos y a los asociados comerciales que no tienen cuentas en el directorio corporativo. Esta situación es común en aplicaciones de negocio a negocio, aplicaciones que se integran con servicios de terceros y en aquellos casos en que las empresas con diferentes sistemas de TI han combinado o compartido los recursos.

- **Identidad federada en aplicaciones SaaS**. En este escenario, fabricantes independientes de software proporcionan un servicio listo para usar para varios clientes o inquilinos. Cada inquilino se autentica mediante un proveedor de identidades adecuado. Por ejemplo, los usuarios empresariales usarán sus credenciales corporativas, mientras que los consumidores y clientes del inquilino utilizarán sus credenciales de identidades sociales.

Este patrón podría no ser útil en las siguientes situaciones:

- Todos los usuarios de la aplicación se pueden autenticar mediante un proveedor de identidades, y no hay ningún requisito para autenticarlos con cualquier otro proveedor de identidades. Esto es habitual en aplicaciones empresariales que utilizan un directorio corporativo (accesible dentro de la aplicación) para la autenticación, mediante una VPN o (en un escenario hospedado en la nube) a través de una conexión de red virtual entre el directorio local y la aplicación.

- La aplicación se creó originalmente mediante un mecanismo de autenticación diferente, quizás con almacenes de usuario personalizados, o no tiene la funcionalidad para gestionar los estándares de negociación utilizados por tecnologías basadas en notificaciones. Retroadaptar la autenticación basada en notificaciones y el control de acceso en las aplicaciones existentes puede ser complejo y, probablemente, poco rentable.

## <a name="example"></a>Ejemplo

Una organización hospeda una aplicación de software como servicio (SaaS) multiinquilinos en Microsoft Azure. La aplicación incluye un sitio web que los inquilinos pueden usar para administrar la aplicación para sus propios usuarios. La aplicación permite a los inquilinos acceder al sitio web mediante una identidad federada que genera Active Directory Federation Services (ADFS) cuando un usuario se autentica con la instancia de Active Directory propiedad de esa organización.

![Cómo los usuarios de un suscriptor de una gran empresa acceden a la aplicación](./_images/federated-identity-multitenat.png)


En la ilustración se muestra cómo los inquilinos se autentican con su propio proveedor de identidades (paso 1), en este caso ADFS. Después de autenticar correctamente un inquilino, ADFS emite un token. El explorador cliente reenvía este token al proveedor de federación de la aplicación de SaaS, que confía en los tokens emitidos por el ADFS del inquilino, con el fin de devolver un token que sea válido para el proveedor de federación de SaaS (paso 2). Si es necesario, el proveedor de federación de SaaS realiza una transformación de las notificaciones del token en notificaciones que la aplicación reconoce (paso 3) antes de devolver el nuevo token al explorador cliente. La aplicación confía en los tokens emitidos por el proveedor de federación de SaaS y usa las notificaciones del token para aplicar las reglas de autorización (paso 4).

Los inquilinos no tienen que recordar distintas credenciales para acceder a la aplicación, y un administrador de la empresa del inquilino puede configurar en su propio ADFS la lista de usuarios que pueden acceder a la aplicación.

## <a name="related-guidance"></a>Instrucciones relacionadas

- [Microsoft Azure Active Directory](https://azure.microsoft.com/services/active-directory/)
- [Active Directory Domain Services](https://msdn.microsoft.com/library/bb897402.aspx)
- [Servicios de federación de Active Directory](https://msdn.microsoft.com/library/bb897402.aspx)
- [Administración de identidades para aplicaciones multiinquilino en Microsoft Azure](https://azure.microsoft.com/documentation/articles/guidance-multitenant-identity/)
- [Aplicaciones multiempresa en Azure](https://azure.microsoft.com/documentation/articles/dotnet-develop-multitenant-applications/)
