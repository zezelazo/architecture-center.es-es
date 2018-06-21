---
title: Creación de un bosque de recursos AD DS en Azure
description: >-
  Cómo crear un dominio de confianza de Active Directory en Azure.

  guidance,vpn-gateway,expressroute,load-balancer,virtual-network,active-directory
author: telmosampaio
ms.date: 05/02/2018
pnp.series.title: Identity management
pnp.series.prev: adds-extend-domain
pnp.series.next: adfs
cardTitle: Create an AD DS forest in Azure
ms.openlocfilehash: 047ecea41ba30ce4cccf17b8c4964a37ae60150f
ms.sourcegitcommit: 0de300b6570e9990e5c25efc060946cb9d079954
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/03/2018
ms.locfileid: "32323914"
---
# <a name="create-an-active-directory-domain-services-ad-ds-resource-forest-in-azure"></a>Creación de un bosque de recursos de Active Directory Domain Services (AD DS) en Azure

Esta arquitectura de referencia muestra cómo crear un dominio de Active Directory independiente en Azure en el que confíen los dominios del bosque de AD local. [**Implemente esta solución**.](#deploy-the-solution)

[![0]][0] 

*Descargue un [archivo Visio][visio-download] de esta arquitectura.*

Active Directory Domain Services (AD DS) almacena información de identidad en una estructura jerárquica. El nodo superior de la estructura jerárquica se conoce como bosque. Un bosque contiene dominios y los dominios contienen otros tipos de objetos. Esta arquitectura de referencia crea un bosque de AD DS en Azure con una relación de confianza saliente unidireccional con un dominio local. El bosque en Azure contiene un dominio que no existe en el entorno local. Debido a la relación de confianza, se puede confiar en los inicios de sesión realizados en los dominios locales para acceder a los recursos del dominio de Azure independiente. 

Los usos habituales de esta arquitectura incluyen el mantenimiento de la separación de seguridad de objetos e identidades mantenida en la nube y la migración de dominios individuales del entorno local a la nube. 

Para consideraciones adicionales, consulte [Selección de una solución para la integración de Active Directory local con Azure][considerations]. 

## <a name="architecture"></a>Architecture

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

Hay disponible una implementación de esta arquitectura en [GitHub][github]. Tenga en cuenta que la implementación completa puede tardar hasta dos horas, lo que incluye la creación de una instancia de VPN Gateway y ejecutar los scripts que configuran AD DS.

### <a name="prerequisites"></a>requisitos previos

1. Clone, bifurque o descargue el archivo ZIP del repositorio de GitHub de [arquitecturas de referencia][github].

2. Instale la [CLI de Azure 2.0][azure-cli-2].

3. Instale el paquete de NPM de [Azure Building Blocks][azbb].

4. Desde un símbolo del sistema, un símbolo del sistema de bash o un símbolo del sistema de PowerShell, inicie sesión en la cuenta de Azure mediante el siguiente comando.

   ```bash
   az login
   ```

### <a name="deploy-the-simulated-on-premises-datacenter"></a>Implementación del centro de datos local simulado

1. Vaya a la carpeta `identity/adds-forest` del repositorio de GitHub.

2. Abra el archivo `onprem.json` . Busque instancias de `adminPassword` y `Password` y agregue valores para las contraseñas.

3. Ejecute el siguiente comando y espere a que finalice la implementación:

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p onprem.json --deploy
    ```

### <a name="deploy-the-azure-vnet"></a>Implementación de la red virtual de Azure

1. Abra el archivo `azure.json` . Busque instancias de `adminPassword` y `Password` y agregue valores para las contraseñas.

2. En el mismo archivo, busque instancias de `sharedKey` y escriba las claves compartidas de la conexión VPN. 

    ```bash
    "sharedKey": "",
    ```

3. Ejecute el siguiente comando y espere a que finalice la implementación.

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p onoprem.json --deploy
    ```

   Realice la implementación en el mismo grupo de recursos que la red virtual local.


### <a name="test-the-ad-trust-relation"></a>Comprobación de la relación de confianza de AD

1. Use Azure Portal y vaya al grupo de recursos que ha creado.

2. Para la máquina virtual denominada `ra-adt-mgmt-vm1` use Azure Portal.

2. Haga clic en `Connect` para abrir una sesión de escritorio remoto en la máquina virtual. El nombre de usuario es `contoso\testuser` y la contraseña es la que especificó en el archivo de parámetros `onprem.json`.

3. Desde dentro de la sesión de escritorio remoto, abra otra sesión de escritorio remoto en 192.168.0.4, que es la dirección IP de la máquina virtual denominada `ra-adtrust-onpremise-ad-vm1`. El nombre de usuario es `contoso\testuser` y la contraseña es la que especificó en el archivo de parámetros `azure.json`.

4. Desde dentro de la sesión de escritorio remoto para `ra-adtrust-onpremise-ad-vm1`, vaya al **administrador del servidor** y haga clic en **Herramientas** > **Dominios y confianzas de Active Directory**. 

5. En el panel izquierdo, haga clic con el botón derecho en contoso.com y seleccione **Propiedades**.

6. Haga clic en la pestaña **Trusts** (Confianzas). Debería ver que treyresearch.net aparece como una confianza entrante.

![](./images/ad-forest-trust.png)


## <a name="next-steps"></a>Pasos siguientes

* Conozca los procedimientos recomendados para [extender el dominio de AD DS local a Azure][adds-extend-domain]
* Conozca los procedimientos recomendados para [crear una infraestructura de AD FS][adfs] en Azure.

<!-- links -->
[adds-extend-domain]: adds-extend-domain.md
[adfs]: adfs.md
[azure-cli-2]: /azure/install-azure-cli
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks

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
[visio-download]: https://archcenter.blob.core.windows.net/cdn/identity-architectures.vsdx
[0]: ./images/adds-forest.png "Protección de la arquitectura de red híbrida con dominios independientes de Active Directory"