---
title: "Integración de dominios locales de AD con Azure Active Directory"
description: "Procedimiento para implementar una arquitectura de red híbrida segura con Azure Active Directory."
author: telmosampaio
pnp.series.title: Identity management
ms.date: 11/28/2016
pnp.series.next: adds-extend-domain
pnp.series.prev: ./index
cardTitle: Integrate on-premises AD with Azure AD
ms.openlocfilehash: dd4cf0369974ea68d240ed294b1c50972d361d74
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2017
---
# <a name="integrate-on-premises-active-directory-domains-with-azure-active-directory"></a>Integración de dominios locales de Active Directory con Azure Active Directory

Azure Active Directory (Azure AD) es un directorio multiinquilino basado en la nube y un servicio de identidades. Esta arquitectura de referencia muestra procedimientos recomendados para la integración de dominios de Active Directory locales con Azure AD a fin de proporcionar la autenticación de identidades basada en la nube. [**Implemente esta solución**.](#deploy-the-solution)

[![0]][0] 

*Descargue un [archivo Visio][visio-download] de esta arquitectura.*

> [!NOTE]
> Para simplificar, este diagrama solo muestra las conexiones relacionadas directamente con Azure AD y el tráfico no relacionado con el protocolo que se puede producir como parte de la federación de la identidad y la autenticación. Por ejemplo, una aplicación web puede redirigir el explorador web para autenticar la solicitud a través de Azure AD. Una vez autenticada, la solicitud puede pasarse a la aplicación web, con la información de identidad adecuada.
> 

Los usos habituales de esta arquitectura de referencia incluyen:

* Aplicaciones web implementadas en Azure que proporcionan acceso a los usuarios remotos que pertenecen a la organización.
* La implementación de funcionalidades de autoservicio para usuarios finales, como restablecer sus contraseñas y delegar la administración de grupos. Tenga en cuenta que esto requiere la edición Premium de Azure AD.
* Arquitecturas en las que la red local y la red virtual de Azure de la aplicación no están conectadas mediante un túnel VPN o un circuito ExpressRoute.

> [!NOTE]
> Actualmente, Azure AD solo admite la autenticación de usuario. Algunas aplicaciones y servicios, como SQL Server, pueden requerir la autenticación de equipo, en cuyo caso esta solución no es adecuada.
> 

Para consideraciones adicionales, consulte [Selección de una solución para la integración de Active Directory local con Azure][considerations]. 

## <a name="architecture"></a>Arquitectura

La arquitectura tiene los siguientes componentes.

* **Inquilino de Azure AD**. Una instancia de [Azure AD][azure-active-directory] creada por su organización. Actúa como un servicio de directorio de aplicaciones en la nube mediante el almacenamiento de objetos copiados desde la instancia local de Active Directory y proporciona servicios de identidad.
* **Subred de nivel web**. Esta subred contiene las máquinas virtuales que ejecutan una aplicación web. Azure AD puede actuar como un intermediario de identidad para esta aplicación.
* **Servidor local AD DS**. Un servicio local de identidades y directorios. El directorio de AD DS se puede sincronizar con Azure AD para permitirle autenticar a los usuarios locales.
* **Servidor de Azure AD Connect Sync**. Un equipo local que ejecuta el servicio de sincronización [Azure AD Connect][ azure-ad-connect]. Este servicio sincroniza la información contenida en Active Directory local para Azure AD. Por ejemplo, si aprovisiona o desaprovisiona usuarios y grupos locales, estos cambios se propagan a Azure AD. 
  
  > [!NOTE]
  > Por motivos de seguridad, Azure AD almacena las contraseñas de usuario como un valor hash. Si un usuario requiere el restablecimiento de una contraseña, debe hacerse de forma local y el nuevo hash debe enviarse a Azure AD. Las ediciones Premium de Azure AD incluyen características que pueden automatizar esta tarea para permitir a los usuarios restablecer sus propias contraseñas.
  > 

* **Máquinas virtuales para aplicación de N niveles**. La implementación incluye la infraestructura para una aplicación con n niveles. Para más información sobre estos recursos, consulte [Ejecución de máquinas virtuales Windows para una arquitectura de n niveles][implementing-a-multi-tier-architecture-on-Azure].

## <a name="recommendations"></a>Recomendaciones

Las siguientes recomendaciones sirven para la mayoría de los escenarios. Sígalas a menos que tenga un requisito concreto que las invalide. 

### <a name="azure-ad-connect-sync-service"></a>Cuenta del servicio de sincronización Azure AD Connect

El servicio de sincronización de Azure AD Connect garantiza que la información de identidad almacenada en la nube sea coherente con la que se mantiene en local. Este servicio se instala con el software de Azure AD Connect. 

Antes de implementar la sincronización de Azure AD Connect, determine los requisitos de sincronización de la organización. Por ejemplo, lo que se debe sincronizar, de qué dominios y con qué frecuencia. Para más información, vea [Determinación de los requisitos de sincronización de directorios][aad-sync-requirements].

Puede ejecutar el servicio de sincronización de Azure AD Connect en una máquina virtual o un equipo hospedado en local. Según la volatilidad de la información en el directorio de Active Directory, es probable que la carga en el servicio de sincronización de Azure AD Connect sea alta después de la sincronización inicial con Azure AD. La ejecución del servicio en una máquina virtual facilita escalar el servidor, si es necesario. Supervise la actividad en la máquina virtual tal como se describe en la sección de consideraciones de supervisión, para determinar si el escalado es necesario.

Si tiene varios dominios locales en un bosque, se recomienda almacenar y sincronizar la información para todo el bosque en un único inquilino de Azure AD. Filtre la información de identidades que se produce en más de un dominio, por lo que cada identidad aparece una sola vez en Azure AD, en lugar de duplicarse. La duplicación puede provocar incoherencias cuando se sincronizan los datos. Para más información, consulte la sección Topología, a continuación. 

Use un filtro para que solo se almacenen en Azure AD los datos necesarios. Por ejemplo, a la organización podría no convenirle almacenar información sobre las cuentas inactivas en Azure AD. El filtrado puede basarse en grupos, en dominios, en unidades organizativa (OU) o en atributos. Puede combinar filtros para generar reglas más complejas. Por ejemplo, puede sincronizar objetos que se encuentren en un dominio y que tengan un valor específico en un atributo seleccionado. Para obtener información detallada, vea [Sincronización de Azure AD Connect: configurar el filtrado][aad-filtering].

Para implementar la alta disponibilidad para el servicio de sincronización de AD Connect, ejecute un servidor provisional secundario. Para más información, consulte la sección Recomendaciones de topología.

### <a name="security-recommendations"></a>Recomendaciones de seguridad

**Administración de contraseñas de usuario.** Las ediciones Premium de Azure AD admiten la escritura diferida de contraseñas, permitiendo a los usuarios locales realizar restablecimientos de contraseñas de autoservicio desde dentro de Azure Portal. Esta característica solo debería habilitarse después de revisar la directiva de seguridad de las contraseñas de su organización. Por ejemplo, puede restringir qué usuarios pueden cambiar sus contraseñas y personalizar la experiencia de su administración. Para más información, vea [Personalización de la administración de contraseñas para ajustarse a las necesidades de su organización][aad-password-management]. 

**Proteja las aplicaciones locales que sean accesibles externamente.** Utilice el Proxy de aplicación de Azure AD para proporcionar acceso controlado a las aplicaciones web locales para los usuarios externos a través de Azure AD. Solo los usuarios que tienen credenciales válidas en su directorio de Azure tienen permiso para usar la aplicación. Para obtener más información, consulte el artículo [Habilitación del Proxy de aplicación en Azure Portal][aad-application-proxy].

**Supervise activamente Azure AD en busca de indicios de actividad sospechosa.**    Considere el uso de la edición Premium P2 de Azure AD, que incluye Azure AD Identity Protection. Identity Protection utiliza algoritmos de aprendizaje automático adaptables y heurísticos para detectar anomalías y eventos de riesgo que pueden indicar que se ha puesto en peligro una identidad. Por ejemplo, puede detectar actividad inusual como actividades irregulares de inicio de sesión, inicios de sesión de orígenes desconocidos o de direcciones IP con actividad sospechosa, o inicios de sesión desde dispositivos que pueden estar infectados. Con estos datos, Identity Protection genera informes y alertas que permiten investigar estos eventos de riesgo y tomar las acciones de corrección adecuadas. Para más información, consulte [Azure Active Directory Identity Protection][aad-identity-protection].
  
Puede usar la característica de informes de Azure AD en Azure Portal para supervisar las actividades relacionadas con la seguridad que se producen en el sistema. Para más información sobre estos informes, consulte [Azure Active Directory Reporting Guide][aad-reporting-guide] (Guía de informes de Azure Active Directory).

### <a name="topology-recommendations"></a>Recomendaciones de topología

Configure Azure AD Connect para implementar la topología que mejor se ajuste a los requisitos de su organización. Las topologías que admite Azure AD Connect son los siguientes:

* **Un único bosque, un único directorio de Azure AD**. En esta topología, Azure AD Connect sincroniza los objetos y la información de identidad de uno o varios dominios de un único bosque local en un solo inquilino de Azure AD. Se trata de la topología predeterminada implementada por la instalación rápida de Azure AD Connect.
  
  > [!NOTE]
  > No use varios servidores de sincronización de Azure AD Connect para conectarse a dominios diferentes en el mismo bosque local para el mismo inquilino de Azure AD, a menos que esté ejecutando un servidor en modo de almacenamiento provisional, según se describe a continuación.
  > 
  > 

* **Varios bosques, un único directorio de Azure AD**. En esta topología, Azure AD Connect sincroniza la información de identidad y los objetos desde varios bosques en un solo inquilino de Azure AD. Use esta topología si su organización tiene más de un bosque local. Puede consolidar la información de identidad para que cada usuario único se represente una vez en el directorio de Azure AD, aunque exista el mismo usuario en más de un bosque. Todos los bosques usar el mismo servidor de sincronización de Azure AD Connect. El servidor de sincronización de Azure AD Connect no tiene que formar parte de ningún dominio, pero debe ser accesible desde todos los bosques.
  
  > [!NOTE]
  > En esta topología, no utilice servidores independientes de sincronización de Azure AD Connect para conectar cada bosque local a un único inquilino de Azure AD. Esto puede dar lugar a información de identidad duplicada en Azure AD si los usuarios están presentes en más de un bosque.
  > 
  > 

* **Varios bosques: topologías independientes**. Esta topología combina la información de identidad de bosques independientes en un solo inquilino de Azure AD, tratando a todos los bosques como entidades independientes. Esta topología es útil si se combinan bosques de organizaciones diferentes y se mantiene en un solo bosque la información de identidad para cada usuario.
  
  > [!NOTE]
  > Si se sincronizan las listas globales de direcciones (GAL) de cada bosque, un usuario de un bosque puede estar presente en otro como un contacto. Esto puede ocurrir si su organización ha implementado GALSync con Forefront Identity Manager 2010 o Microsoft Identity Manager 2016. En este escenario, puede especificar que los usuarios deben identificarse por su atributo *Mail*. También puede hacer coincidir las identidades mediante los atributos *ObjectSID* y *msExchMasterAccountSID*. Resulta de utilidad si tiene uno o varios bosques de recursos con las cuentas deshabilitadas.
  > 
  > 

* **Servidor provisional**. En esta configuración, ejecuta una segunda instancia del servidor de sincronización de Azure AD Connect en paralelo con la primera. Esta estructura es compatible con escenarios como:
  
  * Alta disponibilidad.
  * Prueba e implementación de una nueva configuración del servidor de sincronización de Azure AD Connect.
  * Introducción a un nuevo servidor y retirada de una configuración anterior. 
    
    En estos casos, la segunda instancia se ejecuta en *modo de almacenamiento provisional*. El servidor registra los objetos importados y los datos de sincronización en su base de datos, pero no pasa los datos a Azure AD. Si deshabilita el modo de almacenamiento provisional, el servidor comienza a escribir datos en Azure AD y también realiza escrituras diferidas de contraseñas en los directorios locales cuando corresponda. Para más información, vea [Sincronización de Azure AD Connect: tareas y consideraciones operativas][aad-connect-sync-operational-tasks].

* **Varios directorios de Azure AD**. Se recomienda crear un solo directorio de Azure AD para una organización, pero puede haber situaciones en las que necesite dividir la información en directorios de Azure AD diferentes. En este caso, evite los problemas de la escritura diferida de las contraseñas y la sincronización asegurándose de que cada objeto del bosque local aparece en un único directorio de Azure AD. Para implementar este escenario, configure servidores independientes de sincronización de Azure AD Connect y use el filtrado para que cada servidor de sincronización de Azure AD Connect funcione en un conjunto de objetos mutuamente excluyentes. 

Para más información acerca de estas topologías, consulte [Topologías para Azure AD Connect][aad-topologies].

### <a name="user-authentication"></a>Autenticación de usuarios

De forma predeterminada, el servidor de sincronización de Azure AD Connect configura la sincronización de contraseñas entre el dominio local y Azure AD, y el servicio de Azure AD da por supuesto que los usuarios se autentican proporcionando la misma contraseña que usan en local. Para muchas organizaciones, esto es apropiado, pero debe tener en cuenta las directivas existentes y la infraestructura de su organización. Por ejemplo:

* La directiva de seguridad de una organización podría prohibir la sincronización de hash de contraseñas en la nube.
* Podría requerir que los usuarios experimenten un inicio de sesión único (SSO) fluido al tener acceso a recursos en la nube desde máquinas unidas a un dominio en la red corporativa.
* Su organización podría disponer ya de Active Directory Federation Services (AD FS) o de un proveedor de federación implementado de terceros. Puede configurar Azure AD para usar esta infraestructura e implementar la autenticación y SSO, en lugar de utilizar la información de contraseñas que se mantiene en la nube.

Para más información, consulte [Opciones para el inicio de sesión de los usuarios en Azure AD Connect][aad-user-sign-in].

### <a name="azure-ad-application-proxy"></a>Proxy de aplicación de Azure AD 

Use Azure AD para permitir el acceso a aplicaciones locales.

Exponga las aplicaciones web locales mediante los conectores de proxy de aplicación administrados por el componente de proxy de aplicación de Azure AD. El conector de proxy de aplicación abre una conexión de red de salida para el proxy de aplicación de Azure AD y las solicitudes de los usuarios remotos se enrutan desde Azure AD a través de esta conexión a las aplicaciones web. Así se evita tener que abrir los puertos entrantes en el firewall local y se reduce la superficie expuesta a ataques de la organización.

Para más información, consulte [Publicación de aplicaciones mediante el proxy de aplicación de Azure AD][aad-application-proxy].

### <a name="object-synchronization"></a>Sincronización de objetos 

La configuración predeterminada de Azure AD Connect sincroniza los objetos desde el directorio local de Active Directory en función de las reglas especificadas en el artículo [Sincronización de Azure AD Connect: descripción de la configuración predeterminada][aad-connect-sync-default-rules]. Los objetos que cumplen estas reglas se sincronizan mientras que se pasan por alto todos los demás. Algunas reglas de ejemplo:

* Los objetos de usuario deben tener un único atributo *sourceAnchor* y el atributo *accountEnabled* debe estar relleno.
* Los objetos de usuario deben tener un atributo *sAMAccountName* y no se puede iniciar con el texto *Azure AD_* ni *MSOL_*.

Azure AD Connect aplica varias reglas a los objetos User, Contact, Group, ForeignSecurityPrincipal y Computer. Utilice el Editor de reglas de sincronización instalado con Azure AD Connect, si tiene que modificar el conjunto predeterminado de reglas. Para más información, vea [Sincronización de Azure AD Connect: descripción de la configuración predeterminada][aad-connect-sync-default-rules]).

También puede definir sus propios filtros para limitar los objetos que se van a sincronizar por dominio o unidad organizativa. Como alternativa, puede implementar un filtrado personalizado más complejo como el que se describe en [Azure AD Connect: configuración del filtrado][aad-filtering].

### <a name="monitoring"></a>Supervisión 

La supervisión de estado se realiza mediante los siguientes agentes instalados de forma local:

* Azure AD Connect instala un agente que captura información acerca de las operaciones de sincronización. Utilice la hoja Azure AD Connect Health en Azure Portal para supervisar su estado y rendimiento. Para más información, vea [Uso de Azure AD Connect Health con AD DS][aad-health].
* Para supervisar el estado de los dominios de AD DS y los directorios de Azure, instale el agente de Azure AD Connect Health para AD DS en una máquina, dentro del dominio local. Utilice la hoja de Azure Active Directory Connect Health en Azure Portal para la supervisión de estado. Para más información, vea [Uso de Azure AD Connect Health con AD DS][aad-health-adds]. 
* Instale Azure AD Connect Health para el agente de AD FS a fin de supervisar el estado de los servicios que se ejecutan en local y use la hoja de Azure Active Directory Connect Health en Azure Portal para supervisar AD FS. Para más información, vea [Uso de Azure AD Connect Health con AD FS][aad-health-adfs].

Para más información acerca de cómo instalar los agentes de Azure AD Connect Health y sus requisitos, consulte [Instalación del agente de Azure AD Connect Health][aad-agent-installation].

## <a name="scalability-considerations"></a>Consideraciones sobre escalabilidad

El servicio de Azure AD permite la escalabilidad en función de las réplicas, con una única réplica principal que controla las operaciones de escritura y varias réplicas secundarias de solo lectura. Azure AD redirige de forma transparente los reintentos de escritura realizados en las réplicas secundarias a la réplica principal, y proporciona la coherencia final. Todos los cambios realizados en la réplica principal se propagan a las secundarias. Esta arquitectura se escala bien porque la mayoría de las operaciones en Azure AD son lecturas en lugar de escrituras. Para más información, consulte [Azure AD: Under the hood of our geo-redundant, highly available, distributed cloud directory][aad-scalability] (Azure AD: bajo el paraguas de nuestro directorio en la nube con redundancia geográfica, alta disponibilidad y distribuido).

Para el servidor de sincronización de Azure AD Connect, determine la cantidad de objetos que es probable que se sincronicen desde su directorio local. Si tiene menos de 100.000 objetos, puede usar el software LocalDB de SQL Server Express predeterminado proporcionado con Azure AD Connect. Si tiene un mayor número de objetos, debe instalar una versión de producción de SQL Server y realizar una instalación personalizada de Azure AD Connect, especificando que debería usar una instancia existente de SQL Server.

## <a name="availability-considerations"></a>Consideraciones sobre disponibilidad

El servicio de Azure AD se distribuye geográficamente y se ejecuta en varios centros de datos propagados por el mundo con conmutación por error automática. Si un centro de datos deja de estar disponible, Azure AD garantiza que los datos del directorio están disponibles para el acceso de una instancia en al menos dos centros de datos más dispersos regionalmente.

> [!NOTE]
> El Acuerdo de Nivel de Servicio (SLA) para los servicios de Azure AD Basic y Premium garantiza al menos un 99,9 % de disponibilidad. No hay ningún SLA para el nivel gratuito de Azure AD. Para más información, consulte [SLA de Azure Active Directory][sla-aad].
> 
> 

Considere la posibilidad de aprovisionar una segunda instancia del servidor de sincronización de Azure AD Connect en modo provisional para aumentar la disponibilidad, como se describe en la sección de recomendaciones de topología. 

Si no utiliza la instancia de LocalDB de SQL Server Express que viene con Azure AD Connect, considere el uso de clústeres SQL para lograr una alta disponibilidad. En Azure AD Connect, no se admiten soluciones como la creación de reflejo y Always On.

Para conocer consideraciones adicionales acerca de cómo lograr una alta disponibilidad del servidor de sincronización de Azure AD Connect y también de cómo recuperarse después de un error, consulte [Sincronización de Azure AD Connect: tareas y consideraciones operativas y recuperación ante desastres][aad-sync-disaster-recovery].

## <a name="manageability-considerations"></a>Consideraciones sobre la manejabilidad

Hay dos aspectos importantes para la administración de Azure AD:

* La administración de Azure AD en la nube.
* El mantenimiento de los servidores de sincronización de Azure AD Connect.

Azure AD proporciona las siguientes opciones para la administración de dominios y directorios en la nube: 

* **Módulo PowerShell de Azure Active Directory**. Use este [módulo] [ aad-powershell] si tiene que programar tareas administrativas comunes de Azure AD, como la administración de usuarios, la administración de dominios y la configuración del inicio de sesión único.
* **Hoja de administración de Azure AD en Azure Portal**. Esta hoja proporciona una vista de administración interactiva del directorio y le permite controlar y configurar la mayoría de los aspectos de Azure AD. 

Azure AD Connect instala las siguientes herramientas para mantener los servicios de sincronización de Azure AD Connect de las máquinas locales:
  
* **Consola de Microsoft Azure Active Directory Connect**. Esta herramienta permite modificar la configuración del servidor de sincronización de Azure AD, personalizar el modo en que se produce la sincronización, habilitar o deshabilitar el modo de almacenamiento provisional y cambiar el modo de inicio de sesión del usuario. Tenga en cuenta que puede habilitar el inicio de sesión de Active Directory FS con su infraestructura local.
* **Synchronization Service Manager**. Use la pestaña *Operaciones* de esta herramienta para administrar el proceso de sincronización y detectar si alguna parte del proceso ha dado error. Puede desencadenar las sincronizaciones manualmente mediante esta herramienta. La pestaña *Conectores* le permite controlar las conexiones para los dominios a los que se asocia el motor de sincronización.
* **Editor de reglas de sincronización**. Utilice esta herramienta para personalizar la forma en que se transforman los objetos cuando se copian entre un directorio local y Azure AD. Le permite especificar atributos y objetos adicionales para la sincronización y, a continuación, ejecutar los filtros para determinar qué objetos deben o no sincronizarse. Para más información, vea la sección Editor de reglas de sincronización en el documento [Sincronización de Azure AD Connect: descripción de la configuración predeterminada][aad-connect-sync-default-rules].

Para más información y sugerencias para administrar Azure AD Connect, consulte [Azure AD Connect Sync: procedimientos recomendados de cambio de la configuración predeterminada][aad-sync-best-practices].

## <a name="security-considerations"></a>Consideraciones sobre la seguridad

Use el control de acceso condicional para denegar las solicitudes de autenticación de orígenes inesperados:

- Desencadene [Azure Multi-Factor Authentication (MFA)][azure-multifactor-authentication], si un usuario intenta conectarse desde una ubicación que no sea de confianza, por ejemplo, a través de Internet, en lugar de usar una red de confianza.

- Utilice el tipo de plataforma de dispositivo del usuario (iOS, Android, Windows Mobile, Windows) para determinar la directiva de acceso a las aplicaciones y las características.

- Registre el estado habilitado o deshabilitado de los dispositivos de los usuarios e incorpore dicha información en las comprobaciones de la directiva de acceso. Por ejemplo, si la pérdida o el robo del teléfono de un usuario se debe registrar como deshabilitado para evitar que se use para obtener acceso.

- Controle el acceso de los usuarios a los recursos según la pertenencia al grupo. Use [reglas de pertenencia dinámica de Azure AD][ aad-dynamic-membership-rules] para simplificar la administración del grupo. Para una breve descripción de cómo funciona esto, consulte [Introducción a la pertenencia dinámica a grupos][aad-dynamic-memberships].

- Use directivas de riesgo del acceso condicional con Azure AD Identity Protection, a fin de proporcionar protección avanzada en función de actividades de inicio de sesión inusuales u otros eventos.

Para más información, consulte [Acceso condicional en Azure Active Directory][aad-conditional-access].

## <a name="deploy-the-solution"></a>Implementación de la solución

En GitHub, se puede encontrar una implementación de una arquitectura de referencia que implementa estas recomendaciones y consideraciones. Esta arquitectura de referencia implementa una red local simulada en Azure que puede usar para probar y experimentar. La arquitectura de referencia se puede implementar con máquinas virtuales Windows o Linux siguiendo estas instrucciones: 

1. Haga clic en el botón a continuación:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fidentity%2Fazure-ad%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. Una vez abierto el vínculo en Azure Portal, debe especificar los valores de algunas de las opciones: 
   * El nombre del **Grupo de recursos** ya está definido en el archivo de parámetros, así que seleccione **Crear nuevo** y escriba `ra-aad-onpremise-rg` en el cuadro de texto.
   * Seleccione la región en el cuadro de lista desplegable **Ubicación**.
   * No modifique los cuadros de texto **URI raíz de plantilla** o **URI raíz de parámetro**.
   * Seleccione **Windows** o **Linux** en el cuadro de lista desplegable **Tipo de SO**.
   * Revise los términos y condiciones, y haga clic en la casilla **Acepto los términos y condiciones indicados anteriormente**.
   * Haga clic en el botón **Comprar**.
3. Espere a que la implementación se complete.
4. Los archivos de parámetros incluyen nombres de usuario y contraseñas de administrador codificados de forma rígida, y es muy recomendable que cambie ambos inmediatamente en todas las máquinas virtuales. Haga clic en cada máquina virtual en Azure Portal y después en **Restablecer contraseña**, en la hoja **Soporte técnico y solución de problemas**. Seleccione **Restablecer contraseña** en el cuadro de lista desplegable **Modo**, seleccione un valor de **Nombre de usuario** y un valor de **Contraseña**. Haga clic en el botón **Actualizar** para conservar el nuevo nombre de usuario y contraseña.

<!-- links -->

[implementing-a-multi-tier-architecture-on-Azure]: ../virtual-machines-windows/n-tier.md

[aad-agent-installation]: /azure/active-directory/active-directory-aadconnect-health-agent-install
[aad-application-proxy]: /azure/active-directory/active-directory-application-proxy-enable
[aad-conditional-access]: /azure/active-directory//active-directory-conditional-access
[aad-connect-sync-default-rules]: /azure/active-directory/active-directory-aadconnectsync-understanding-default-configuration
[aad-connect-sync-operational-tasks]: /azure/active-directory/active-directory-aadconnectsync-operations#staging-mode
[aad-dynamic-memberships]: https://youtu.be/Tdiz2JqCl9Q
[aad-dynamic-membership-rules]: /azure/active-directory/active-directory-accessmanagement-groups-with-advanced-rules
[aad-editions]: /azure/active-directory/active-directory-editions
[aad-filtering]: /azure/active-directory/active-directory-aadconnectsync-configure-filtering
[aad-health]: /azure/active-directory/active-directory-aadconnect-health-sync
[aad-health-adds]: /azure/active-directory/active-directory-aadconnect-health-adds
[aad-health-adfs]: /azure/active-directory/active-directory-aadconnect-health-adfs
[aad-identity-protection]: /azure/active-directory/active-directory-identityprotection
[aad-password-management]: /azure/active-directory/active-directory-passwords-customize
[aad-powershell]: https://msdn.microsoft.com/library/azure/mt757189.aspx
[aad-reporting-guide]: /azure/active-directory/active-directory-reporting-guide
[aad-scalability]: https://blogs.technet.microsoft.com/enterprisemobility/2014/09/02/azure-ad-under-the-hood-of-our-geo-redundant-highly-available-distributed-cloud-directory/
[aad-sync-best-practices]: /azure/active-directory/active-directory-aadconnectsync-best-practices-changing-default-configuration
[aad-sync-disaster-recovery]: /azure/active-directory/active-directory-aadconnectsync-operations#disaster-recovery
[aad-sync-requirements]: /azure/active-directory/active-directory-hybrid-identity-design-considerations-directory-sync-requirements
[aad-topologies]: /azure/active-directory/active-directory-aadconnect-topologies
[aad-user-sign-in]: /azure/active-directory/active-directory-aadconnect-user-signin
[azure-active-directory]: /azure/active-directory-domain-services/active-directory-ds-overview
[azure-ad-connect]: /azure/active-directory/active-directory-aadconnect
[azure-multifactor-authentication]: /azure/multi-factor-authentication/multi-factor-authentication
[considerations]: ./considerations.md
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[sla-aad]: https://azure.microsoft.com/support/legal/sla/active-directory/v1_0/
[visio-download]: https://archcenter.azureedge.net/cdn/identity-architectures.vsdx


[0]: ./images/azure-ad.png "Arquitectura de identidad en la nube con Azure Active Directory"