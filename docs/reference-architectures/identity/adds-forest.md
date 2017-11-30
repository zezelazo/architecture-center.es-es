---
title: "Creación de un bosque de recursos AD DS en Azure"
description: "Cómo crear un dominio de confianza de Active Directory en Azure.\nguidance,vpn-gateway,expressroute,load-balancer,virtual-network,active-directory"
author: telmosampaio
ms.date: 11/28/2016
pnp.series.title: Identity management
pnp.series.prev: adds-extend-domain
pnp.series.next: adfs
cardTitle: Create an AD DS forest in Azure
ms.openlocfilehash: bb7e57af2afacf1faa7679c854bf49217918eba8
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2017
---
# <a name="create-an-active-directory-domain-services-ad-ds-resource-forest-in-azure"></a>Creación de un bosque de recursos de Active Directory Domain Services (AD DS) en Azure

Esta arquitectura de referencia muestra cómo crear un dominio de Active Directory independiente en Azure en el que confíen los dominios del bosque de AD local. [**Implemente esta solución**.](#deploy-the-solution)

[![0]][0] 

*Descargue un [archivo Visio][visio-download] de esta arquitectura.*

Active Directory Domain Services (AD DS) almacena información de identidad en una estructura jerárquica. El nodo superior de la estructura jerárquica se conoce como bosque. Un bosque contiene dominios y los dominios contienen otros tipos de objetos. Esta arquitectura de referencia crea un bosque de AD DS en Azure con una relación de confianza saliente unidireccional con un dominio local. El bosque en Azure contiene un dominio que no existe en el entorno local. Debido a la relación de confianza, se puede confiar en los inicios de sesión realizados en los dominios locales para acceder a los recursos del dominio de Azure independiente. 

Los usos habituales de esta arquitectura incluyen el mantenimiento de la separación de seguridad de objetos e identidades mantenida en la nube y la migración de dominios individuales del entorno local a la nube. 

Para obtener consideraciones adicionales, consulte [Selección de una solución para la integración de Active Directory local con Azure][considerations]. 

## <a name="architecture"></a>Arquitectura

La arquitectura tiene los siguientes componentes.

* **Red local** La red local contiene su propio bosque y dominios de Active Directory.
* **Servidores de Active Directory**. Son controladores de dominio que implementan servicios de dominio que se ejecutan como máquinas virtuales en la nube. Estos servidores hospedan un bosque que contiene uno o varios dominios, que están separados de aquellos ubicados en el entorno local.
* **Relación de confianza unidireccional**. El ejemplo del diagrama muestra una confianza unidireccional entre el dominio de Azure y el dominio local. Esta relación permite que los usuarios locales accedan a los recursos que hay en el dominio de Azure, pero no al revés. Es posible crear una confianza bidireccional si los usuarios de la nube también necesitan acceso a los recursos locales.
* **Subred de Active Directory**. Los servidores de AD DS se hospedan en una subred independiente. Las reglas de grupo de seguridad de red (NSG) protegen los servidores de AD DS y proporcionan un firewall contra el tráfico de orígenes inesperados.
* **Puerta de enlace de Azure**. La puerta de enlace de Azure proporciona una conexión entre la red local y la red virtual de Azure. Puede ser una [conexión VPN] [ azure-vpn-gateway] o de [Azure ExpressRoute][azure-expressroute]. Para más información, consulte [Implementación de una arquitectura de red híbrida segura en Azure][implementing-a-secure-hybrid-network-architecture].

## <a name="recommendations"></a>Recomendaciones

Si necesita recomendaciones específicas sobre cómo implementar Active Directory en Azure, consulte los siguientes artículos:

- [Extending Active Directory Domain Services (AD DS) to Azure][adds-extend-domain] (Extensión de Active Directory Domain Services (AD DS) a Azure). 
- [Directrices para implementar Windows Server Active Directory en Azure Virtual Machines][ad-azure-guidelines].

### <a name="trust"></a>Trust

Los dominios locales están dentro de un bosque diferente de los dominios en la nube. Para permitir la autenticación de usuarios locales en la nube, los dominios de Azure deben confiar en el dominio de inicio de sesión del bosque local. De forma similar, si la nube proporciona un dominio de inicio de sesión para los usuarios externos, puede ser necesario que el bosque local confíe en el dominio en la nube.

Puede establecer relaciones de confianza en el nivel de bosque mediante la [creación de confianzas de bosque][creating-forest-trusts], o en el nivel de dominio mediante la [creación de confianzas externas][creating-external-trusts]. Una nivel de confianza de bosque crea una relación entre todos los dominios de los dos bosques. Una confianza de nivel de dominio externo solo crea una relación entre dos dominios especificados. Solo se deben crear confianzas de nivel de dominio externo entre dominios de diferentes bosques.

Las relaciones de confianza pueden ser en un solo sentido (unidireccionales) o en ambos sentidos (bidireccionales):

* Una confianza unidireccional permite a los usuarios de un dominio o bosque (conocido como el dominio o bosque *entrante* dominio o bosque) acceder a los recursos mantenidos en otro (el dominio o bosque *saliente*).
* Una confianza bidireccional permite a los usuarios del dominio o bosque acceder a los recursos mantenidos en los otros.

En la tabla siguiente se resumen las configuraciones de confianza para algunos escenarios sencillos:

| Escenario | Confianza local | Confianza en la nube |
| --- | --- | --- |
| Los usuarios locales necesitan acceso a los recursos de la nube, pero no al revés. |Unidireccional, entrante |Unidireccional, saliente |
| Los usuarios de la nube necesitan acceso a los recursos ubicados en el entorno local, pero no al revés. |Unidireccional, saliente |Unidireccional, entrante |
| Los usuarios de la nube y locales necesitan acceso a los recursos mantenidos en la nube y locales. |Bidireccional, entrante y saliente |Bidireccional, entrante y saliente |

## <a name="scalability-considerations"></a>Consideraciones sobre escalabilidad

Active Directory es escalable automáticamente para los controladores de dominio que forman parte del mismo dominio. Las solicitudes se distribuyen entre todos los controladores de un dominio. Si agrega otro controlador de dominio, este se sincroniza automáticamente con el dominio. No configure un equilibrador de carga independiente para dirigir el tráfico a los controladores dentro del dominio. Asegúrese de que todos los controladores de dominio tengan suficientes recursos de memoria y almacenamiento para administrar la base de datos de dominio. Iguale el tamaño de todas las máquinas virtuales de controlador de dominio.

## <a name="availability-considerations"></a>Consideraciones sobre disponibilidad

Aprovisione al menos dos controladores de dominio para cada dominio. Esto permite la replicación automática entre servidores. Cree un conjunto de disponibilidad para las máquinas virtuales que actúan como servidores de Active Directory que administran cada dominio. Coloque al menos dos servidores en este conjunto de disponibilidad.

Además, considere la posibilidad de designar uno o más servidores de cada dominio como [maestro de operaciones en espera][standby-operations-masters] para los casos en que se produzca un error de conectividad a un servidor que actúa con el rol de operación de maestro único flexible (FSMO).

## <a name="manageability-considerations"></a>Consideraciones sobre la manejabilidad

Para conocer las consideraciones sobre administración y supervisión, consulte [Extensión de Active Directory a Azure][adds-extend-domain]. 
 
Para más información, consulte [Supervisión de Active Directory][monitoring_ad]. Puede instalar herramientas como [Microsoft Systems Center][microsoft_systems_center] en un servidor de supervisión en la subred de administración para ayudar a realizar estas tareas.

## <a name="security-considerations"></a>Consideraciones sobre la seguridad

Las confianzas de nivel de bosque son transitivas. Si establece una confianza de nivel de bosque entre un bosque local y un bosque en la nube, esta confianza se extiende a otros dominios nuevos creados en cualquiera de los bosques. Si usa dominios para proporcionar separación por motivos de seguridad, considere la posibilidad de crear confianzas solo en el nivel de dominio. Las confianzas de nivel de dominio son no transitivas.

Para conocer las consideraciones sobre seguridad específicas de Active Directory, consulte la sección [Extensión de Active Directory a Azure][adds-extend-domain].

## <a name="deploy-the-solution"></a>Implementación de la solución

Hay disponible una solución en [Github][github] para implementar esta arquitectura de refeerencia. Necesitará la versión más reciente de la CLI de Azure para ejecutar el script de Powershell que implementa la solución. Para implementar la arquitectura de referencia, siga estos pasos:

1. Descargue o clone la carpeta de soluciones de [Github][github] en la máquina local.

2. Abra la CLI de Azure y navegue hasta la carpeta local de la solución.

3. Ejecute el siguiente comando:
   
    ```Powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> <mode>
    ```
   
    Reemplace `<subscription id>` con la identificación de su suscripción de Azure.
   
    Para `<location>`, especifique una región de Azure, como `eastus` o `westus`.
   
    El parámetro `<mode>` controla la granularidad de la implementación y puede ser uno de los siguientes valores:
   
   * `Onpremise`: implementa el entorno local simulado.
   * `Infrastructure`: implementa la infraestructura de VNet y el JumBox en Azure.
   * `CreateVpn`: implementa la puerta de enlace de red virtual de Azure y la conecta a la red local simulada.
   * `AzureADDS`: implementa las máquinas virtuales que actúan como servidores de Active Directory DS, implementa Active Directory en estas máquinas virtuales e implementa el dominio en Azure.
   * `WebTier`: implementa las máquinas virtuales de nivel web y el equilibrador de carga.
   * `Prepare`: implementa todas las implementaciones anteriores. **Esta es la opción recomendada si no tiene una red local existente pero quiere implementar la arquitectura de referencia completa que se ha descrito anteriormente para pruebas o evaluación.** 
   * `Workload`: implementa las máquinas virtuales de capa de datos y empresa y los equilibradores de carga. Tenga en cuenta que estas máquinas virtuales no se incluyen en la implementación `Prepare`.

4. Espere a que la implementación se complete. Si va a realizar la implementación `Prepare`, tardará varias horas.
     
5. Si va a usar la configuración local simulada, configure la relación de confianza entrante:
   
   1. Conéctese al JumpBox (*ra-adtrust-mgmt-vm1* en el grupo de recursos *ra-adtrust-security-rg*). Inicie sesión como *testuser* con la contraseña *AweS0me@PW*.
   2. En el JumpBox, abra una sesión RDP en la primera máquina virtual del dominio *contoso.com* (el dominio local). Esta máquina virtual tiene la dirección IP 192.168.0.4. El nombre de usuario es *contoso\testuser* y la contraseña *AweS0me@PW*.
   3. Descargue el script [incoming-trust.ps1][incoming-trust] y ejecútelo para crear la confianza entrante desde el dominio *treyresearch.com*.

6. Si usa su propia infraestructura local:
   
   1. Descargue el script [incoming-trust.ps1][incoming-trust].
   2. Edítelo y reemplace el valor de la variable `$TrustedDomainName` por el nombre de su propio dominio.
   3. Ejecute el script.

7. En el JumpBox, conéctese a la primera máquina virtual del dominio *treyresearch.com* (el dominio en la nube). Esta máquina virtual tiene la dirección IP 10.0.4.4. El nombre de usuario es *treyresearch\testuser* y la contraseña *AweS0me@PW*.

8. Descargue el script [outgoing-trust.ps1][outgoing-trust] y ejecútelo para crear la confianza saliente desde el dominio *treyresearch.com*. Si va a usar sus propias máquinas locales, edite primero el script. Establezca la variable `$TrustedDomainName` en el nombre del dominio local y especifique las direcciones IP de los servidores de Active Directory DS para este dominio en la variable `$TrustedDomainDnsIpAddresses`.

9. Espere unos minutos a que finalicen los pasos anteriores y luego conéctese a una máquina virtual local y siga los pasos descritos en el artículo [Comprobación de una confianza][verify-a-trust] para determinar si la relación de confianza entre los dominios *contoso.com* y *treyresearch.com* está configurada correctamente.

## <a name="next-steps"></a>Pasos siguientes

* Conozca los procedimientos recomendados para [extender el dominio de AD DS local a Azure][adds-extend-domain]
* Conozca los procedimientos recomendados para [crear una infraestructura de AD FS][adfs] en Azure.

<!-- links -->
[adds-extend-domain]: adds-extend-domain.md
[adfs]: adfs.md

[implementing-a-secure-hybrid-network-architecture]: ../dmz/secure-vnet-hybrid.md
[implementing-a-secure-hybrid-network-architecture-with-internet-access]: ../dmz/secure-vnet-dmz.md

[running-VMs-for-an-N-tier-architecture-on-Azure]: ../virtual-machines-windows/n-tier.md

[ad-azure-guidelines]: https://msdn.microsoft.com/library/azure/jj156090.aspx
[azure-expressroute]: https://azure.microsoft.com/documentation/articles/expressroute-introduction/
[azure-vpn-gateway]: https://azure.microsoft.com/documentation/articles/vpn-gateway-about-vpngateways/
[considerations]: ./considerations.md
[creating-external-trusts]: https://technet.microsoft.com/library/cc816837(v=ws.10).aspx
[creating-forest-trusts]: https://technet.microsoft.com/library/cc816810(v=ws.10).aspx
[github]: https://github.com/mspnp/reference-architectures/tree/master/identity/adds-forest
[incoming-trust]: https://raw.githubusercontent.com/mspnp/reference-architectures/master/identity/adds-forest/extensions/incoming-trust.ps1
[microsoft_systems_center]: https://www.microsoft.com/server-cloud/products/system-center-2016/
[monitoring_ad]: https://msdn.microsoft.com/library/bb727046.aspx
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[solution-script]: https://raw.githubusercontent.com/mspnp/reference-architectures/master/identity/adds-forest/Deploy-ReferenceArchitecture.ps1
[standby-operations-masters]: https://technet.microsoft.com/library/cc794737(v=ws.10).aspx
[outgoing-trust]: https://raw.githubusercontent.com/mspnp/reference-architectures/master/identity/adds-forest/extensions/outgoing-trust.ps1
[verify-a-trust]: https://technet.microsoft.com/library/cc753821.aspx
[visio-download]: https://archcenter.azureedge.net/cdn/identity-architectures.vsdx
[0]: ./images/adds-forest.png "Protección de la arquitectura de red híbrida con dominios independientes de Active Directory"