---
title: Implementación de una topología de red en estrella tipo hub-and-spoke en Azure
titleSuffix: Azure Reference Architectures
description: Implemente una topología de red en estrella tipo hub-and-spoke en Azure.
author: telmosampaio
ms.date: 10/08/2018
ms.custom: seodec18
ms.openlocfilehash: fe56630b621f02fe71b864642b75688ba1965862
ms.sourcegitcommit: 8d951fd7e9534054b160be48a1881ae0857561ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/13/2018
ms.locfileid: "53329439"
---
# <a name="implement-a-hub-spoke-network-topology-in-azure"></a>Implementación de una topología de red en estrella tipo hub-and-spoke en Azure

En esta arquitectura de referencia se muestra cómo implementar una topología en estrella tipo hub-and-spoke en Azure. El *concentrador* (hub) es una red virtual (VNet) en Azure que actúa como un punto central de conectividad para la red local. Los *radios* (spoke) son redes virtuales (VNet) que se emparejan con el concentrador y que se pueden usar para aislar cargas de trabajo. El tráfico fluye entre el centro de datos local y el concentrador a través de una conexión a ExpressRoute o a VPN Gateway. [**Implemente esta solución**](#deploy-the-solution).

![[0]][0]

*Descargue un [archivo Visio][visio-download] de esta arquitectura.*

Las ventajas de esta topología son:

- **Ahorros en costos** gracias a la centralización de los servicios que se pueden compartir entre varias cargas de trabajo, como las aplicaciones virtuales de red (NVA) y los servidores DNS, en una única ubicación.
- **Superar los límites de las suscripciones** gracias al emparejamiento de las redes virtuales de diferentes suscripciones con el concentrador central.
- **Separación de intereses** entre la TI central (SecOps e InfraOps) y las cargas de trabajo (DevOps).

Los usos habituales de esta arquitectura incluyen:

- Cargas de trabajo implementadas en distintos entornos, como desarrollo, pruebas y producción, que requieren servicios compartidos como DNS, IDS, NTP o AD DS. Los servicios compartidos se colocan en la red virtual del concentrador, mientras que cada entorno se implementa en un radio para mantener el aislamiento.
- Cargas de trabajo que no requieren conectividad entre sí, pero requieren acceso a los servicios compartidos.
- Empresas que requieren el control centralizado sobre aspectos de seguridad, como un firewall en el concentrador como una red perimetral, así como la administración segregada de las cargas de trabajo en cada radio.

## <a name="architecture"></a>Arquitectura

La arquitectura consta de los siguientes componentes:

- **Red local**. Una red de área local privada que se ejecuta dentro de una organización.

- **Dispositivo VPN**. Un dispositivo o servicio que proporciona conectividad externa a la red local. El dispositivo VPN puede ser un dispositivo de hardware, o puede ser una solución de software como el servicio de Enrutamiento y acceso remoto (RRAS) en Windows Server 2012. Para obtener una lista de dispositivos VPN compatibles e información acerca de cómo configurar dispositivos VPN seleccionados para conectarse a Azure, consulte [Acerca de los dispositivos VPN y los parámetros de IPsec/IKE para conexiones de VPN Gateway de sitio a sitio][vpn-appliance].

- **Puerta de enlace de red virtual de VPN o puerta de enlace de ExpressRoute**. La puerta de enlace de red virtual permite que la red virtual se conecte al dispositivo VPN o al circuito de ExpressRoute que se usa para la conectividad con la red local. Para más información, consulte [Conectar una red local con una red virtual de Microsoft Azure][connect-to-an-Azure-vnet].

> [!NOTE]
> Los scripts de implementación para esta arquitectura de referencia usan una instancia de VPN Gateway para la conectividad y una red virtual en Azure para simular la red local.

- **Red virtual del concentrador**. La red virtual de Azure utilizada como el concentrador en la topología en estrella tipo hub-and-spoke. El concentrador es el punto central de conectividad a la red local y un lugar para hospedar los servicios que se pueden usar en las diferentes cargas de trabajo hospedadas en las redes virtuales de los radios.

- **Subred de puerta de enlace**. Las puertas de enlace de red virtual se conservan en la misma subred.

- **Redes virtuales de radios**. Una o varias redes virtuales de Azure que se usan como radios en la topología en estrella tipo hub-and-spoke. Los radios pueden utilizarse para aislar las cargas de trabajo en sus propias redes virtuales, administradas por separado desde otros radios. Cada carga de trabajo puede incluir varios niveles, con varias subredes que se conectan a través de equilibradores de carga de Azure. Para obtener más información acerca de la infraestructura de aplicaciones, consulte [Ejecutar cargas de trabajo de máquinas virtuales Windows][windows-vm-ra] y [Ejecutar cargas de trabajo de máquinas virtuales Linux][linux-vm-ra].

- **Emparejamiento de VNET**. Se pueden conectar dos redes virtuales mediante una [conexión de emparejamiento][vnet-peering]. Las conexiones de emparejamiento son conexiones no transitivas de baja latencia entre las redes virtuales. Una vez establecido el emparejamiento, las redes virtuales intercambian el tráfico mediante la red troncal de Azure, sin necesidad de un enrutador. En una topología de red en estrella tipo hub-and-spoke, el emparejamiento de VNET se usa para conectar el concentrador a cada radio. Puede emparejar redes virtuales de la misma región o de regiones diferentes. Para más información, consulte [Requisitos y restricciones][vnet-peering-requirements].

> [!NOTE]
> En este artículo solo se tratan las implementaciones de [Resource Manager](/azure/azure-resource-manager/resource-group-overview), pero también puede conectar una red virtual clásica a una red virtual de Resource Manager en la misma suscripción. De este modo, los radios pueden hospedar implementaciones clásicas y seguir beneficiándose de los servicios compartidos en el concentrador.

## <a name="recommendations"></a>Recomendaciones

Las siguientes recomendaciones sirven para la mayoría de los escenarios. Sígalas a menos que tenga un requisito concreto que las invalide.

### <a name="resource-groups"></a>Grupos de recursos

La red virtual del concentrador y la red virtual de cada radio se pueden implementar en distintos grupos de recursos e incluso en distintas suscripciones. Cuando empareja redes virtuales en distintas suscripciones, ambas suscripciones pueden estar asociadas al mismo inquilino de Azure Active Directory o a uno diferente. Esto permite realizar una administración descentralizada de cada carga de trabajo, mientras se comparten los servicios que se mantienen en la red virtual del concentrador.

### <a name="vnet-and-gatewaysubnet"></a>VNet y GatewaySubnet

Cree una subred denominada *GatewaySubnet*, con un intervalo de direcciones de /27. Esta subred es necesaria para la puerta de enlace de red virtual. La asignación de treinta y dos direcciones a esta subred ayudará a evitar que se alcancen las limitaciones de tamaño de la puerta de enlace en el futuro.

Para más información sobre la configuración de la puerta de enlace, consulte las siguientes arquitecturas de referencia, en función del tipo de conexión:

- [Red híbrida mediante ExpressRoute][guidance-expressroute]
- [Red híbrida con una instancia de VPN Gateway][guidance-vpn]

Para una mayor disponibilidad, puede utilizar ExpressRoute y una VPN para la conmutación por error. Vea [Conexión de una red local a Azure mediante ExpressRoute con conmutación por error de VPN][hybrid-ha].

También se puede utilizar una topología en estrella tipo hub-and-spoke sin una puerta de enlace, en caso de que no se necesite la conectividad con la red local.

### <a name="vnet-peering"></a>Emparejamiento de VNET

El emparejamiento de VNET es una relación no transitiva entre dos redes virtuales. Si necesita que los radios se conecten entre sí, considere la posibilidad de agregar una conexión de emparejamiento independiente entre dichos radios.

Sin embargo, si tiene varios radios que necesitan conectarse entre sí, se quedará sin posibles conexiones de emparejamiento muy rápido debido a la [limitación del número de emparejamientos de VNET por cada red virtual][vnet-peering-limit]. En este escenario, considere la posibilidad de usar rutas definidas por el usuario (UDR) para forzar que el tráfico destinado a un radio se envíe a una NVA que actúa como un enrutador en la red virtual del concentrador. Esto permitirá que los radios se conecten entre sí.

También puede configurar los radios para que usen la puerta de enlace de la red virtual del concentrador para comunicarse con las redes remotas. Para permitir que el tráfico de la puerta de enlace fluya del radio al concentrador y para conectarse a las redes remotas, debe:

- Configurar la conexión de emparejamiento de VNET en el concentrador para **permitir el tránsito de la puerta de enlace**.
- Configurar la conexión de emparejamiento de VNET en cada radio para **usar las puertas de enlace remotas**.
- Configurar todas las conexiones de emparejamiento de VNET para **permitir el tráfico reenviado**.

## <a name="considerations"></a>Consideraciones

### <a name="spoke-connectivity"></a>Conectividad de los radios

Si se necesita una conectividad entre los radios, considere la posibilidad de implementar una NVA para el enrutamiento en el concentrador y de usar las UDR en el radio para reenviar el tráfico al concentrador.

![[2]][2]

En este escenario, debe configurar las conexiones de emparejamiento para que **permitan el tráfico reenviado**.

### <a name="overcoming-vnet-peering-limits"></a>Superación de los límites de emparejamiento de VNET

Asegúrese de tener en cuenta la [limitación del número de emparejamientos de VNET por cada red virtual][vnet-peering-limit] en Azure. Si decide que necesita más radios de los que el límite permitirá, considere la posibilidad de crear una topología en estrella tipo hub-hub-and-spoke-spoke, donde el primer nivel de radios también actúa como concentradores. En el siguiente diagrama se ilustra este enfoque.

![[3]][3]

También tenga en cuenta qué servicios se comparten en el concentrador, para asegurarse de que el concentrador se escala para tener un número mayor de radios. Por ejemplo, si el concentrador proporciona servicios de firewall, tenga en cuenta los límites de ancho de banda de la solución de firewall al agregar varios radios. Puede mover algunos de estos servicios compartidos a un segundo nivel de concentradores.

## <a name="deploy-the-solution"></a>Implementación de la solución

Hay disponible una implementación de esta arquitectura en [GitHub][ref-arch-repo]. Usa máquinas virtuales en cada red virtual para probar la conectividad. No hay ningún servicio real hospedado en la subred de **servicios compartidos** de la **red virtual del concentrador**.

La implementación crea los siguientes grupos de recursos en su suscripción:

- hub-nva-rg
- hub-vnet-rg
- onprem-jb-rg
- onprem-vnet-rg
- spoke1-vnet-rg
- spoke2-vent-rg

Los archivos de parámetro de plantilla hacen referencia a estos nombres, por lo que si se cambian es necesario actualizar los archivos de parámetro para que coincidan.

### <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-the-simulated-on-premises-datacenter"></a>Implementación del centro de datos local simulado

Para implementar el centro de datos local simulado como una red virtual de Azure, siga estos pasos:

1. Vaya a la carpeta `hybrid-networking/hub-spoke` del repositorio de arquitecturas de referencia.

2. Abra el archivo `onprem.json` . Reemplace los valores de `adminUsername` y `adminPassword`.

    ```json
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

3. (Opcional) Para una implementación de Linux, establezca `osType` en `Linux`.

4. Ejecute el siguiente comando:

    ```bash
    azbb -s <subscription_id> -g onprem-vnet-rg -l <location> -p onprem.json --deploy
    ```

5. Espere a que finalice la implementación. Esta implementación crea una red virtual, una máquina virtual y una instancia de VPN Gateway. Se tardará aproximadamente unos 40 minutos en crear una instancia de VPN Gateway.

### <a name="deploy-the-hub-vnet"></a>Implementación de la red virtual del concentrador

Para implementar red virtual del concentrador, siga los pasos a continuación.

1. Abra el archivo `hub-vnet.json` . Reemplace los valores de `adminUsername` y `adminPassword`.

    ```json
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. (Opcional) Para una implementación de Linux, establezca `osType` en `Linux`.

3. Busque ambas instancias de `sharedKey` y especifique una clave compartida para la conexión VPN. Los valores deben coincidir.

    ```json
    "sharedKey": "",
    ```

4. Ejecute el siguiente comando:

    ```bash
    azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet.json --deploy
    ```

5. Espere a que finalice la implementación. Esta implementación crea una red virtual, una máquina virtual, una instancia de VPN Gateway y una conexión a la puerta de enlace.  Se tardará aproximadamente unos 40 minutos en crear una instancia de VPN Gateway.

### <a name="test-connectivity-with-the-hub"></a>Prueba de conectividad con el concentrador

Pruebe la conectividad desde el entorno local simulado a la red virtual del concentrador.

**Implementación de Windows**

1. Use Azure Portal para encontrar la máquina virtual denominada `jb-vm1` en el grupo de recursos `onprem-jb-rg`.

2. Haga clic en `Connect` para abrir una sesión de escritorio remoto en la máquina virtual. Usar la contraseña que especificó en el `onprem.json` archivo de parámetros.

3. Abra una consola de PowerShell en la máquina virtual y utilice el cmdlet `Test-NetConnection` para comprobar que puede conectarse a la máquina virtual de JumpBox en la red virtual del concentrador.

   ```powershell
   Test-NetConnection 10.0.0.68 -CommonTCPPort RDP
   ```

La salida debe tener una apariencia similar a la siguiente:

```powershell
ComputerName     : 10.0.0.68
RemoteAddress    : 10.0.0.68
RemotePort       : 3389
InterfaceAlias   : Ethernet 2
SourceAddress    : 192.168.1.000
TcpTestSucceeded : True
```

> [!NOTE]
> De forma predeterminada, las máquinas virtuales de Windows Server no permiten respuestas ICMP en Azure. Si desea usar `ping` para probar la conectividad, debe habilitar el tráfico ICMP en el firewall de Windows con seguridad avanzada para cada máquina virtual.

**Implementación de Linux**

1. Use Azure Portal para encontrar la máquina virtual denominada `jb-vm1` en el grupo de recursos `onprem-jb-rg`.

2. Haga clic en `Connect` y copie el comando `ssh`que se muestra en el portal. 

3. Desde un símbolo del sistema de Linux, ejecute `ssh` para conectar con el entorno local simulado. Usar la contraseña que especificó en el `onprem.json` archivo de parámetros.

4. Use el comando `ping` para probar la conectividad con la máquina virtual de JumpBox en la red virtual del concentrador:

   ```shell
   ping 10.0.0.68
   ```

### <a name="deploy-the-spoke-vnets"></a>Implementación de redes virtuales de radios

Para implementar las redes virtuales de radios, siga estos pasos.

1. Abra el archivo `spoke1.json` . Reemplace los valores de `adminUsername` y `adminPassword`.

    ```json
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. (Opcional) Para una implementación de Linux, establezca `osType` en `Linux`.

3. Ejecute el siguiente comando:

   ```bash
   azbb -s <subscription_id> -g spoke1-vnet-rg -l <location> -p spoke1.json --deploy
   ```

4. Repita los pasos 1 y 2 para el archivo `spoke2.json`.

5. Ejecute el siguiente comando:

   ```bash
   azbb -s <subscription_id> -g spoke2-vnet-rg -l <location> -p spoke2.json --deploy
   ```

6. Ejecute el siguiente comando:

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet-peering.json --deploy
   ```

### <a name="test-connectivity"></a>Comprobación de la conectividad

Pruebe la conectividad desde el entorno local simulado a la red virtual de radios.

**Implementación de Windows**

1. Use Azure Portal para encontrar la máquina virtual denominada `jb-vm1` en el grupo de recursos `onprem-jb-rg`.

2. Haga clic en `Connect` para abrir una sesión de escritorio remoto en la máquina virtual. Usar la contraseña que especificó en el `onprem.json` archivo de parámetros.

3. Abra una consola de PowerShell en la máquina virtual y utilice el cmdlet `Test-NetConnection` para comprobar que puede conectarse a las máquinas virtuales de JumpBox en las redes virtuales de radio.

   ```powershell
   Test-NetConnection 10.1.0.68 -CommonTCPPort RDP
   Test-NetConnection 10.2.0.68 -CommonTCPPort RDP
   ```

**Implementación de Linux**

Para probar la conectividad desde el entorno local simulado a las redes virtuales de radios mediante máquinas virtuales Linux, realice los pasos siguientes:

1. Use Azure Portal para encontrar la máquina virtual denominada `jb-vm1` en el grupo de recursos `onprem-jb-rg`.

2. Haga clic en `Connect` y copie el comando `ssh`que se muestra en el portal.

3. Desde un símbolo del sistema de Linux, ejecute `ssh` para conectar con el entorno local simulado. Usar la contraseña que especificó en el `onprem.json` archivo de parámetros.

4. Use el comando `ping` para probar la conectividad con las máquinas virtuales de JumpBox en cada radio:

   ```bash
   ping 10.1.0.68
   ping 10.2.0.68
   ```

### <a name="add-connectivity-between-spokes"></a>Adición de conectividad entre radios

Este paso es opcional. Si desea permitir que los radios se conecten unos con otros, tendrá que usar una aplicación virtual de red (NVA) como enrutador de la red virtual del concentrador y forzar el tráfico desde los radios al enrutador cuando intente conectar con otro radio. Para implementar una aplicación virtual de red de ejemplo básica como máquina virtual única, junto con las rutas definidas por el usuario (UDR) que son necesarias para que las dos redes virtuales de radios se conecten, realice los pasos siguientes:

1. Abra el archivo `hub-nva.json` . Reemplace los valores de `adminUsername` y `adminPassword`.

    ```json
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. Ejecute el siguiente comando:

   ```bash
   azbb -s <subscription_id> -g hub-nva-rg -l <location> -p hub-nva.json --deploy
   ```

<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azure-vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[best-practices-security]: /azure/best-practices-network-securit
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[guidance-expressroute]: ./expressroute.md
[guidance-vpn]: ./vpn.md
[linux-vm-ra]: ../virtual-machines-linux/index.md
[hybrid-ha]: ./expressroute-vpn-failover.md
[naming conventions]: /azure/guidance/guidance-naming-conventions
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[vnet-peering]: /azure/virtual-network/virtual-network-peering-overview
[vnet-peering-limit]: /azure/azure-subscription-service-limits#networking-limits
[vnet-peering-requirements]: /azure/virtual-network/virtual-network-manage-peering#requirements-and-constraints
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[windows-vm-ra]: ../virtual-machines-windows/index.md
[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures

[0]: ./images/hub-spoke.png "Topología en estrella tipo hub-and-spoke en Azure"
[1]: ./images/hub-spoke-gateway-routing.svg "Topología en estrella tipo hub-and-spoke en Azure con enrutamiento transitivo"
[2]: ./images/hub-spoke-no-gateway-routing.svg "Topología en estrella tipo hub-and-spoke en Azure con enrutamiento transitivo mediante una NVA"
[3]: ./images/hub-spokehub-spoke.svg "Topología en estrella tipo hub-hub-and-spoke-spoke en Azure"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
