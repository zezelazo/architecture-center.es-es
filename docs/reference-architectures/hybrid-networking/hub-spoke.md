---
title: "Implementación de una topología de red en estrella tipo hub-and-spoke en Azure"
description: "Cómo implementar una topología de red en estrella tipo hub-and-spoke en Azure."
author: telmosampaio
ms.date: 05/05/2017
pnp.series.title: Implement a hub-spoke network topology in Azure
pnp.series.prev: expressroute
ms.openlocfilehash: e6f07a7962dd5728226b023700268340590d97a3
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2017
---
# <a name="implement-a-hub-spoke-network-topology-in-azure"></a>Implementación de una topología de red en estrella tipo hub-and-spoke en Azure

En esta arquitectura de referencia se muestra cómo implementar una topología en estrella tipo hub-and-spoke en Azure. El *concentrador* (hub) es una red virtual (VNet) en Azure que actúa como un punto central de conectividad para la red local. Los *radios* (spoke) son redes virtuales (VNet) que se emparejan con el concentrador y que se pueden usar para aislar cargas de trabajo. El tráfico fluye entre el centro de datos local y el concentrador a través de una conexión a ExpressRoute o a VPN Gateway.  [**Implemente esta solución**](#deploy-the-solution).

![[0]][0]

*Descargue un [archivo Visio][visio-download] de esta arquitectura.*


Las ventajas de esta topología son:

* **Ahorros en costos** gracias a la centralización de los servicios que se pueden compartir entre varias cargas de trabajo, como las aplicaciones virtuales de red (NVA) y los servidores DNS, en una única ubicación.
* **Superar los límites de las suscripciones** gracias al emparejamiento de las redes virtuales de diferentes suscripciones con el concentrador central.
* **Separación de intereses** entre la TI central (SecOps e InfraOps) y las cargas de trabajo (DevOps).

Los usos habituales de esta arquitectura incluyen:

* Cargas de trabajo implementadas en distintos entornos, como desarrollo, pruebas y producción, que requieren servicios compartidos como DNS, IDS, NTP o AD DS. Los servicios compartidos se colocan en la red virtual del concentrador, mientras que cada entorno se implementa en un radio para mantener el aislamiento.
* Cargas de trabajo que no requieren conectividad entre sí, pero requieren acceso a los servicios compartidos.
* Empresas que requieren el control centralizado sobre aspectos de seguridad, como un firewall en el concentrador como una red perimetral, así como la administración segregada de las cargas de trabajo en cada radio.

## <a name="architecture"></a>Arquitectura

La arquitectura consta de los siguientes componentes:

* **Red local**. Una red de área local privada que se ejecuta dentro de una organización.

* **Dispositivo VPN**. Un dispositivo o servicio que proporciona conectividad externa a la red local. El dispositivo VPN puede ser un dispositivo de hardware, o puede ser una solución de software como el servicio de Enrutamiento y acceso remoto (RRAS) en Windows Server 2012. Para obtener una lista de dispositivos VPN compatibles e información acerca de cómo configurar dispositivos VPN seleccionados para conectarse a Azure, consulte [Acerca de los dispositivos VPN y los parámetros de IPsec/IKE para conexiones de VPN Gateway de sitio a sitio][vpn-appliance].

* **Puerta de enlace de red virtual de VPN o puerta de enlace de ExpressRoute**. La puerta de enlace de red virtual permite que la red virtual se conecte al dispositivo VPN o al circuito de ExpressRoute que se usa para la conectividad con la red local. Para más información, consulte [Conectar una red local con una red virtual de Microsoft Azure][connect-to-an-Azure-vnet].

> [!NOTE]
> Los scripts de implementación para esta arquitectura de referencia usan una instancia de VPN Gateway para la conectividad y una red virtual en Azure para simular la red local.

* **Red virtual del concentrador**. La red virtual de Azure utilizada como el concentrador en la topología en estrella tipo hub-and-spoke. El concentrador es el punto central de conectividad a la red local y un lugar para hospedar los servicios que se pueden usar en las diferentes cargas de trabajo hospedadas en las redes virtuales de los radios.

* **Subred de puerta de enlace**. Las puertas de enlace de red virtual se conservan en la misma subred.

* **Subred de servicios compartidos**. Una subred de la red virtual del concentrador utilizada para hospedar servicios que se pueden compartir entre todos los radios, como DNS o AD DS.

* **Redes virtuales de radios**. Una o varias redes virtuales de Azure que se usan como radios en la topología en estrella tipo hub-and-spoke. Los radios pueden utilizarse para aislar las cargas de trabajo en sus propias redes virtuales, administradas por separado desde otros radios. Cada carga de trabajo puede incluir varios niveles, con varias subredes que se conectan a través de equilibradores de carga de Azure. Para obtener más información acerca de la infraestructura de aplicaciones, consulte [Ejecutar cargas de trabajo de máquinas virtuales Windows][windows-vm-ra] y [Ejecutar cargas de trabajo de máquinas virtuales Linux][linux-vm-ra].

* **Emparejamiento de VNET**. Se pueden conectar dos redes virtuales en la misma región de Azure mediante una [conexión de emparejamiento][vnet-peering]. Las conexiones de emparejamiento son conexiones no transitivas de baja latencia entre las redes virtuales. Una vez establecido el emparejamiento, las redes virtuales intercambian el tráfico mediante la red troncal de Azure, sin necesidad de un enrutador. En una topología de red en estrella tipo hub-and-spoke, el emparejamiento de VNET se usa para conectar el concentrador a cada radio.

> [!NOTE]
> En este artículo solo se tratan las implementaciones de [Resource Manager](/azure/azure-resource-manager/resource-group-overview), pero también puede conectar una red virtual clásica a una red virtual de Resource Manager en la misma suscripción. De este modo, los radios pueden hospedar implementaciones clásicas y seguir beneficiándose de los servicios compartidos en el concentrador.


## <a name="recommendations"></a>Recomendaciones

Las siguientes recomendaciones sirven para la mayoría de los escenarios. Sígalas a menos que tenga un requisito concreto que las invalide.

### <a name="resource-groups"></a>Grupos de recursos

La red virtual del concentrador y la red virtual de cada radio se pueden implementar en diferentes grupos de recursos e incluso en distintas suscripciones, siempre y cuando pertenezcan al mismo inquilino de Azure Active Directory (Azure AD) en la misma región. Esto permite realizar una administración descentralizada de cada carga de trabajo, mientras se comparten los servicios que se mantienen en la red virtual del concentrador.

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

Hay disponible una implementación de esta arquitectura en [GitHub][ref-arch-repo]. Usa máquinas virtuales Ubuntu en cada red virtual para probar la conectividad. No hay ningún servicio real hospedado en la subred de **servicios compartidos** de la **red virtual del concentrador**.

### <a name="prerequisites"></a>Requisitos previos

Antes de poder implementar la arquitectura de referencia en su propia suscripción, debe realizar los pasos siguientes.

1. Clone, bifurque o descargue el archivo ZIP para el repositorio de GitHub de [arquitecturas de referencia de AzureCAT][ref-arch-repo].

2. Si prefiere usar la CLI de Azure, asegúrese de que tiene la CLI de Azure 2.0 instalada en el equipo. Para instalar la CLI, siga las instrucciones de [Instalación de la CLI de Azure 2.0][azure-cli-2].

3. Si prefiere usar PowerShell, asegúrese de que tiene el último módulo de PowerShell para Azure instalado en el equipo. Para instalar el último módulo de Azure PowerShell, siga las instrucciones descritas en [Instalación de PowerShell para Azure][azure-powershell].

4. Desde un símbolo del sistema, un símbolo del sistema de Bash o un símbolo del sistema de PowerShell, inicie sesión en la cuenta de Azure con alguno de los comandos siguientes y siga las indicaciones.

  ```bash
  az login
  ```

  ```powershell
  Login-AzureRmAccount
  ```

### <a name="deploy-the-simulated-on-premises-datacenter"></a>Implementación del centro de datos local simulado

Para implementar el centro de datos local simulado como una red virtual de Azure, realice los pasos siguientes.

1. Navegue hasta la carpeta `hybrid-networking\hub-spoke\onprem` del repositorio que descargó en el paso de requisitos previos anterior.

2. Abra el archivo `onprem.vm.parameters.json`, escriba un nombre de usuario y la contraseña entre comillas en las líneas 11 y 12, tal y como se muestra a continuación, y después guarde el archivo.

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. Ejecute el comando de Bash o de PowerShell siguiente para implementar el entorno local simulado como una red virtual en Azure. Sustituya los valores con su suscripción, el nombre del grupo de recursos y la región de Azure.

  ```bash
  sh ./onprem.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-onprem-rg \
    --location westus
  ```

  ```powershell
  ./onprem.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-onprem-rg `
    -Location westus
  ```
  > [!NOTE]
  > Si decide usar un nombre del grupo de recursos distinto (que no sea `ra-onprem-rg`), asegúrese de que busca todos los archivos de parámetros que usan ese nombre y de que los modifica para usar el nombre de su propio grupo de recursos.

4. Espere a que finalice la implementación. Esta implementación crea una red virtual, una máquina virtual con Ubuntu y una instancia de VPN Gateway. La creación de la instancia de VPN Gateway puede durar más de cuarenta minutos.

### <a name="azure-hub-vnet"></a>Red virtual del concentrador de Azure

Para implementar la red virtual del concentrador y conectarla a la red virtual local simulada creada anteriormente, realice los pasos siguientes.

1. Navegue hasta la carpeta `hybrid-networking\hub-spoke\hub` del repositorio que descargó en el paso de requisitos previos anterior.

2. Abra el archivo `hub.vm.parameters.json`, escriba un nombre de usuario y la contraseña entre comillas en las líneas 11 y 12, tal y como se muestra a continuación, y después guarde el archivo.

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. Abra el archivo `hub.gateway.parameters.json` y escriba una clave compartida entre comillas en la línea 23, tal y como se muestra a continuación, y después guarde el archivo. Anote este valor, ya que deberá usarlo más adelante en la implementación.

  ```bash
  "sharedKey": "",
  ```

4. Ejecute el comando de Bash o de PowerShell siguiente para implementar el entorno local simulado como una red virtual en Azure. Sustituya los valores con su suscripción, el nombre del grupo de recursos y la región de Azure.

  ```bash
  sh ./hub.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-hub-rg \
    --location westus
  ```

  ```powershell
  ./hub.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-hub-rg `
    -Location westus
  ```
  > [!NOTE]
  > Si decide usar un nombre del grupo de recursos distinto (que no sea `ra-hub-rg`), asegúrese de que busca todos los archivos de parámetros que usan ese nombre y de que los modifica para usar el nombre de su propio grupo de recursos.

5. Espere a que finalice la implementación. Esta implementación crea una red virtual, una máquina virtual con Ubuntu, una instancia de VPN Gateway y una conexión a la puerta de enlace creada en la sección anterior. La creación de la instancia de VPN Gateway puede durar más de cuarenta minutos.

### <a name="connection-from-on-premises-to-the-hub"></a>Conexión del entorno local al concentrador

Para establecer una conexión del centro de datos local simulado a la red virtual del concentrador, realice los pasos siguientes.

1. Navegue hasta la carpeta `hybrid-networking\hub-spoke\onprem` del repositorio que descargó en el paso de requisitos previos anterior.

2. Abra el archivo `onprem.connection.parameters.json` y escriba una clave compartida entre comillas en la línea 9, tal y como se muestra a continuación, y después guarde el archivo. Este valor de clave compartida debe ser el mismo que se usa en la puerta de enlace local implementada previamente.

  ```bash
  "sharedKey": "",
  ```

3. Ejecute el comando de Bash o de PowerShell siguiente para implementar el entorno local simulado como una red virtual en Azure. Sustituya los valores con su suscripción, el nombre del grupo de recursos y la región de Azure.

  ```bash
  sh ./onprem.connection.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-onprem-rg \
    --location westus
  ```

  ```powershell
  ./onprem.connection.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-onprem-rg `
    -Location westus
  ```
  > [!NOTE]
  > Si decide usar un nombre del grupo de recursos distinto (que no sea `ra-onprem-rg`), asegúrese de que busca todos los archivos de parámetros que usan ese nombre y de que los modifica para usar el nombre de su propio grupo de recursos.

4. Espere a que finalice la implementación. Esta implementación crea una conexión entre la red virtual que se utiliza para simular un centro de datos local y la red virtual del concentrador.

### <a name="azure-spoke-vnets"></a>Redes virtuales de radios de Azure

Para implementar las redes virtuales de los radios y conectarlas a la red virtual del concentrador creada anteriormente, realice los pasos siguientes.

1. Cambie a la carpeta `hybrid-networking\hub-spoke\spokes` del repositorio que descargó en el paso de requisitos previos anterior.

2. Abra el archivo `spoke1.web.parameters.json` y escriba un nombre de usuario y la contraseña entre comillas en las líneas 53 y 54, tal y como se muestra a continuación, y después guarde el archivo.

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. Repita el paso anterior para el archivo `spoke2.web.parameters.json`.

4. Ejecute el comando de Bash o de PowerShell siguiente para implementar el primer radio y conectarlo al concentrador. Sustituya los valores con su suscripción, el nombre del grupo de recursos y la región de Azure.

  ```bash
  sh ./spoke.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke1-rg \
    --location westus \
    --spoke 1
  ```

  ```powershell
  ./spoke.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke1-rg `
    -Location westus `
    -Spoke 1
  ```
  > [!NOTE]
  > Si decide usar un nombre del grupo de recursos distinto (que no sea `ra-spoke1-rg`), asegúrese de que busca todos los archivos de parámetros que usan ese nombre y de que los modifica para usar el nombre de su propio grupo de recursos.

5. Espere a que finalice la implementación. Esta implementación crea una red virtual, un equilibrador de carga con tres máquinas virtuales que ejecutan Ubuntu y Apache y una conexión de emparejamiento de VNET a la red virtual del concentrador creada en la sección anterior. Esta implementación puede tardar más de veinte minutos.

6. Ejecute el comando de Bash o de PowerShell siguiente para implementar el primer radio y conectarlo al concentrador. Sustituya los valores con su suscripción, el nombre del grupo de recursos y la región de Azure.

  ```bash
  sh ./spoke.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke2-rg \
    --location westus \
    --spoke 2
  ```

  ```powershell
  ./spoke.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke2-rg `
    -Location westus `
    -Spoke 2
  ```
  > [!NOTE]
  > Si decide usar un nombre del grupo de recursos distinto (que no sea `ra-spoke2-rg`), asegúrese de que busca todos los archivos de parámetros que usan ese nombre y de que los modifica para usar el nombre de su propio grupo de recursos.

5. Espere a que finalice la implementación. Esta implementación crea una red virtual, un equilibrador de carga con tres máquinas virtuales que ejecutan Ubuntu y Apache y una conexión de emparejamiento de VNET a la red virtual del concentrador creada en la sección anterior. Esta implementación puede tardar más de veinte minutos.

### <a name="azure-hub-vnet-peering-to-spoke-vnets"></a>Emparejamiento de VNET del concentrador de Azure a las redes virtuales de los radios

Para implementar las conexiones de emparejamiento de VNET para la red virtual del concentrador, realice los pasos siguientes.

1. Cambie a la carpeta `hybrid-networking\hub-spoke\hub` del repositorio que descargó en el paso de requisitos previos anterior.

2. Ejecute el comando de Bash o de PowerShell siguiente para implementar la conexión de emparejamiento al primer radio. Sustituya los valores con su suscripción, el nombre del grupo de recursos y la región de Azure.

  ```bash
  sh ./hub.peering.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-hub-rg \
    --location westus \
    --spoke 1
  ```

  ```powershell
  ./hub.peering.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-hub-rg `
    -Location westus `
    -Spoke 1
  ```

2. Ejecute el comando de Bash o de PowerShell siguiente para implementar la conexión de emparejamiento al segundo radio. Sustituya los valores con su suscripción, el nombre del grupo de recursos y la región de Azure.

  ```bash
  sh ./hub.peering.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-hub-rg \
    --location westus \
    --spoke 2
  ```

  ```powershell
  ./hub.peering.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-hub-rg `
    -Location westus `
    -Spoke 2
  ```

### <a name="test-connectivity"></a>Comprobación de la conectividad

Para verificar si funciona la topología en estrella tipo hub-and-spoke conectada a una implementación del centro de datos local, siga estos pasos.

1. Desde [Azure Portal][portal], conéctese a su suscripción y vaya a la máquina virtual `ra-onprem-vm1` del grupo de recursos `ra-onprem-rg`.

2. En la hoja `Overview`, tenga en cuenta `Public IP address` para la máquina virtual.

3. Use un cliente SSH para conectarse a la dirección IP anotada anteriormente con el uso del nombre de usuario y la contraseña especificados durante la implementación.

4. Desde el símbolo del sistema en la máquina virtual a la que está conectado, ejecute el comando siguiente para probar la conectividad desde la red virtual local a la red virtual Spoke1.

  ```bash
  ping 10.1.1.37
  ```

### <a name="add-connectivity-between-spokes"></a>Adición de conectividad entre radios

Si desea permitir que los radios se conecten entre sí, debe implementar UDR en cada radio que reenvía tráfico destinado a otros radios a la puerta de enlace de la red virtual del concentrador. Realice los pasos siguientes para verificar que actualmente no puede conectarse de un radio a otro y después implemente UDR y vuelva a probar la conectividad.

1. Repita los pasos 1 a 4 anteriores, si ya no está conectado a la máquina virtual del JumpBox.

2. Conéctese a uno de los servidores web del radio 1.

  ```bash
  ssh 10.1.1.37
  ```

3. Pruebe la conectividad entre el radio 1 y el radio 2. Debe producirse un error.

  ```bash
  ping 10.1.2.37
  ```

4. Vuelva al símbolo del sistema del equipo.

5. Cambie a la carpeta `hybrid-networking\hub-spoke\spokes` del repositorio que descargó en el paso de requisitos previos anterior.

6. Ejecute el comando de Bash o de PowerShell siguiente para implementar una UDR en el primer radio. Sustituya los valores con su suscripción, el nombre del grupo de recursos y la región de Azure.

  ```bash
  sh ./spoke.udr.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke1-rg \
    --location westus \
    --spoke 1
  ```

  ```powershell
  ./spoke.udr.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke1-rg `
    -Location westus `
    -Spoke 1
  ```

7. Ejecute el comando de Bash o de PowerShell siguiente para implementar una UDR en el segundo radio. Sustituya los valores con su suscripción, el nombre del grupo de recursos y la región de Azure.

  ```bash
  sh ./spoke.udr.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke2-rg \
    --location westus \
    --spoke 2
  ```

  ```powershell
  ./spoke.udr.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke2-rg `
    -Location westus `
    -Spoke 2
  ```

8. Vuelva a cambiar al terminal SSH.

9. Pruebe la conectividad entre el radio 1 y el radio 2. Debe ser correcta.

  ```bash
  ping 10.1.2.37
  ```

<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azure-powershell]: /powershell/azure/install-azure-ps?view=azuresmps-3.7.0
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
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[windows-vm-ra]: ../virtual-machines-windows/index.md

[visio-download]: https://archcenter.azureedge.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/hub-spoke.png "Topología en estrella tipo hub-and-spoke en Azure"
[1]: ./images/hub-spoke-gateway-routing.svg "Topología en estrella tipo hub-and-spoke en Azure con enrutamiento transitivo"
[2]: ./images/hub-spoke-no-gateway-routing.svg "Topología en estrella tipo hub-and-spoke en Azure con enrutamiento transitivo mediante una NVA"
[3]: ./images/hub-spokehub-spoke.svg "Topología en estrella tipo hub-hub-and-spoke-spoke en Azure"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
