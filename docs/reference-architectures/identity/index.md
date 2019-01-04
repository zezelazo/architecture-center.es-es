---
title: Integración de Active Directory local con Azure
titleSuffix: Azure Reference Architectures
description: Compare las arquitecturas de referencia para la integración de Active Directory local con Azure.
ms.date: 07/02/2018
ms.custom: seodec18
ms.openlocfilehash: 99a64f0a5fbe5624aa8ad05bd3565ab2aef618b3
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/04/2019
ms.locfileid: "54011793"
---
# <a name="choose-a-solution-for-integrating-on-premises-active-directory-with-azure"></a>Selección de una solución para la integración de Active Directory local con Azure

En este artículo se comparan opciones para integrar el entorno local de Active Directory (AD) con una red de Azure. Para cada opción, hay disponible una arquitectura de referencia más detallada.

Muchas organizaciones usan Active Directory Domain Services (AD DS) para autenticar las identidades asociadas con usuarios, equipos, aplicaciones y otros recursos que se incluyen en un límite de seguridad. Los servicios de identidad y directorio se hospedan normalmente de forma local, pero si parte de su aplicación está hospedada en el entorno local y parte en Azure, puede que se produzca una latencia al enviar las solicitudes de autenticación desde Azure de vuelta al entorno local. La implementación de servicios de directorio e identidad en Azure puede reducir esta latencia.

Azure proporciona dos soluciones para implementar servicios de directorio e identidad en Azure:

- Use [Azure AD][azure-active-directory] para crear un dominio de Active Directory en la nube y conectarlo a su dominio local de Active Directory. [Azure AD Connect][azure-ad-connect] integra sus directorios locales con Azure AD.

- Amplíe la infraestructura existente de Active Directory local a Azure mediante la implementación de una máquina virtual en Azure que ejecute AD DS como un controlador de dominio. Esta arquitectura es más frecuente cuando la red local y la red virtual de Azure (VNet) están conectadas mediante una conexión VPN o ExpressRoute. Son posibles diversas variantes de esta arquitectura:

  - Cree un dominio en Azure y únalo a su bosque de AD local.
  - Cree un bosque independiente en Azure en el que confíen los dominios del bosque local.
  - Replique una implementación de Servicios de federación de Active Directory (AD FS) en Azure.

En las secciones siguientes se describe cada una de estas opciones con más detalle.

## <a name="integrate-your-on-premises-domains-with-azure-ad"></a>Integración de los dominios locales con Azure AD

Use Azure Active Directory (Azure AD) para crear un dominio en Azure y vincularlo a un dominio de AD local.

El directorio de Azure AD no es una extensión de un directorio local. Por el contrario, es una copia que contiene los mismos objetos e identidades. Los cambios realizados en estos elementos locales se copian en Azure AD, pero los realizados en Azure AD no se replican al dominio local.

También puede usar Azure AD sin utilizar un directorio local. En este caso, Azure AD actúa como el origen principal de toda la información de identidad, en lugar de contener los datos replicados desde un directorio local.

**Ventajas**

- No es necesario mantener una infraestructura de AD en la nube. Azure AD es un servicio que administra y mantiene completamente Microsoft.
- Azure AD proporciona la misma información de identidad que está disponible en el entorno local.
- La autenticación puede tener lugar en Azure, lo que reduce la necesidad de que las aplicaciones y los usuarios externos se pongan en contacto con el dominio local.

**Desafíos**

- Los servicios de identidad se limitan a usuarios y grupos. No existe la posibilidad de autenticar cuentas de servicio y de equipo.
- Debe configurar la conectividad con el dominio local para mantener sincronizado el directorio de Azure AD.
- Puede que sea necesario volver a escribir las aplicaciones para permitir la autenticación mediante Azure AD.

**Arquitectura de referencia**

- [Integración de dominios locales de Active Directory con Azure Active Directory][aad]

## <a name="ad-ds-in-azure-joined-to-an-on-premises-forest"></a>AD DS en Azure unido a un bosque local

Implemente servidores de Servicios de dominio de Active Directory (AD DS) en Azure. Cree un dominio en Azure y únalo a su bosque de AD local.

Tenga en cuenta esta opción si tiene que usar características de AD DS que no están implementadas actualmente por Azure AD.

**Ventajas**

- Proporciona acceso a la misma información de identidad que está disponible en el entorno local.
- Puede autenticar las cuentas de usuario, servicio y equipo de forma local y en Azure.
- No es necesario administrar un bosque de AD independiente. El dominio de Azure puede pertenecer al bosque local.
- Puede aplicar la directiva de grupo definida por objetos de directiva de grupo local al dominio de Azure.

**Desafíos**

- Debe implementar y administrar sus propios servidores y dominios de AD DS en la nube.
- Puede haber cierta latencia de sincronización entre los servidores de dominio en la nube y los servidores que se ejecutan en el entorno local.

**Arquitectura de referencia**

- [Extensión de Active Directory Domain Services (AD DS) a Azure][ad-ds]

## <a name="ad-ds-in-azure-with-a-separate-forest"></a>AD DS en Azure con un bosque independiente

Implemente servidores de AD Domain Services (AD DS) en Azure, pero cree un bosque [independiente][ad-forest-defn] de Azure AD que esté separado del bosque local. Todos los dominios del bosque local confían en este bosque.

Los usos habituales de esta arquitectura incluyen el mantenimiento de la separación de seguridad de objetos e identidades mantenida en la nube y la migración de dominios individuales del entorno local a la nube.

**Ventajas**

- Puede implementar identidades locales y separar las identidades que son solo de Azure.
- No es necesario realizar la replicación del bosque local de AD a Azure.

**Desafíos**

- La autenticación en Azure de las identidades locales requiere saltos de red adicionales a los servidores de AD local.
- Debe implementar sus propios servidores y bosques de AD DS en la nube y establecer las relaciones de confianza adecuadas entre los bosques.

**Arquitectura de referencia**

- [Creación de un bosque de recursos de Active Directory Domain Services (AD DS) en Azure][ad-ds-forest]

## <a name="extend-ad-fs-to-azure"></a>Extensión de AD FS a Azure

Replique una implementación de Servicios de federación de Active Directory (AD FS) a Azure, para realizar la autenticación y la autorización federadas de los componentes que se ejecutan en Azure.

Usos habituales de esta arquitectura:

- Autenticar y autorizar a los usuarios de organizaciones asociadas.
- Permitir a los usuarios autenticarse en exploradores web que se ejecutan fuera del firewall de la organización.
- Permitir a los usuarios conectarse desde dispositivos externos autorizados, como dispositivos móviles.

**Ventajas**

- Puede aprovechar las aplicaciones basadas en notificaciones.
- Ofrece la posibilidad de confiar la autenticación a asociados externos.
- Compatibilidad con un gran conjunto de protocolos de autenticación.

**Desafíos**

- Debe implementar sus propios servidores de AD DS, AD FS y de proxy de aplicación web de AD FS en Azure.
- Esta arquitectura puede ser difícil de configurar.

**Arquitectura de referencia**

- [Extensión de Servicios de federación de Active Directory (AD FS) a Azure][adfs]

<!-- links -->

[aad]: ./azure-ad.md
[ad-ds]: ./adds-extend-domain.md
[ad-ds-forest]: ./adds-forest.md
[ad-forest-defn]: /windows/desktop/AD/forests
[adfs]: ./adfs.md

[azure-active-directory]: /azure/active-directory-domain-services/active-directory-ds-overview
[azure-ad-connect]: /azure/active-directory/hybrid/whatis-hybrid-identity
