---
title: "Implementación de una topología de red en estrella tipo hub-and-spoke con servicios compartidos en Azure"
description: "Implementación de una topología de red en estrella tipo hub-and-spoke con servicios compartidos en Azure."
author: telmosampaio
ms.date: 02/25/2018
pnp.series.title: Implement a hub-spoke network topology with shared services in Azure
pnp.series.prev: hub-spoke
ms.openlocfilehash: c0fb1d1ddd7c70ed914d58e7c73b10475b91aedf
ms.sourcegitcommit: 2123c25b1a0b5501ff1887f98030787191cf6994
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/08/2018
---
# <a name="implement-a-hub-spoke-network-topology-with-shared-services-in-azure"></a>Implementación de una topología de red en estrella tipo hub-and-spoke con servicios compartidos en Azure

Esta arquitectura de referencia se basa en la arquitectura de referencia tipo [hub-and-spoke][guidance-hub-spoke] para incluir servicios compartidos en el centro que puedan consumir todos los radios. Como primer paso para migrar un centro de datos a la nube y generar un [centro de datos virtual], los primeros servicios que necesitará compartir son los de identidad y seguridad. Esta arquitectura de referencia muestra cómo ampliar los servicios de Active Directory desde el centro de datos local a Azure y cómo agregar una aplicación virtual de red (NVA) que pueda actuar como un firewall, en una topología en estrella tipo hub-and-spoke.  [**Implemente esta solución**](#deploy-the-solution).

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

## <a name="architecture"></a>Architecture

La arquitectura consta de los siguientes componentes:

* **Red local**. Una red de área local privada que se ejecuta dentro de una organización.

* **Dispositivo VPN**. Un dispositivo o servicio que proporciona conectividad externa a la red local. El dispositivo VPN puede ser un dispositivo de hardware, o puede ser una solución de software como el servicio de Enrutamiento y acceso remoto (RRAS) en Windows Server 2012. Para obtener una lista de dispositivos VPN compatibles e información acerca de cómo configurar dispositivos VPN seleccionados para conectarse a Azure, consulte [Acerca de los dispositivos VPN y los parámetros de IPsec/IKE para conexiones de VPN Gateway de sitio a sitio][vpn-appliance].

* **Puerta de enlace de red virtual de VPN o puerta de enlace de ExpressRoute**. La puerta de enlace de red virtual permite que la red virtual se conecte al dispositivo VPN o al circuito de ExpressRoute que se usa para la conectividad con la red local. Para más información, consulte [Conectar una red local con una red virtual de Microsoft Azure][connect-to-an-Azure-vnet].

> [!NOTE]
> Los scripts de implementación para esta arquitectura de referencia usan una instancia de VPN Gateway para la conectividad y una red virtual en Azure para simular la red local.

* **Red virtual del concentrador**. La red virtual de Azure utilizada como el concentrador en la topología en estrella tipo hub-and-spoke. El concentrador es el punto central de conectividad a la red local y un lugar para hospedar los servicios que se pueden usar en las diferentes cargas de trabajo hospedadas en las redes virtuales de los radios.

* **Subred de puerta de enlace**. Las puertas de enlace de red virtual se conservan en la misma subred.

* **Subred de servicios compartidos**. Una subred de la red virtual del concentrador utilizada para hospedar servicios que se pueden compartir entre todos los radios, como DNS o AD DS.

* **Subred DMZ**. Una subred de la red virtual del centro que se usa para hospedar aplicaciones virtuales de red (NVA) que pueden actuar como aplicaciones de seguridad como, por ejemplo, firewalls.

* **Redes virtuales de radios**. Una o varias redes virtuales de Azure que se usan como radios en la topología en estrella tipo hub-and-spoke. Los radios pueden utilizarse para aislar las cargas de trabajo en sus propias redes virtuales, administradas por separado desde otros radios. Cada carga de trabajo puede incluir varios niveles, con varias subredes que se conectan a través de equilibradores de carga de Azure. Para obtener más información acerca de la infraestructura de aplicaciones, consulte [Ejecutar cargas de trabajo de máquinas virtuales Windows][windows-vm-ra] y [Ejecutar cargas de trabajo de máquinas virtuales Linux][linux-vm-ra].

* **Emparejamiento de VNET**. Se pueden conectar dos redes virtuales en la misma región de Azure mediante una [conexión de emparejamiento][vnet-peering]. Las conexiones de emparejamiento son conexiones no transitivas de baja latencia entre las redes virtuales. Una vez establecido el emparejamiento, las redes virtuales intercambian el tráfico mediante la red troncal de Azure, sin necesidad de un enrutador. En una topología de red en estrella tipo hub-and-spoke, el emparejamiento de VNET se usa para conectar el concentrador a cada radio.

> [!NOTE]
> En este artículo solo se tratan las implementaciones de [Resource Manager](/azure/azure-resource-manager/resource-group-overview), pero también puede conectar una red virtual clásica a una red virtual de Resource Manager en la misma suscripción. De este modo, los radios pueden hospedar implementaciones clásicas y seguir beneficiándose de los servicios compartidos en el concentrador.

## <a name="recommendations"></a>Recomendaciones

Todas las recomendaciones de la arquitectura de referencia de tipo [hub-and-spoke][guidance-hub-spoke] también se aplican a la arquitectura de referencia de los servicios compartidos. 

Igualmente, las siguientes recomendaciones sirven para la mayoría de escenarios de servicios compartidos. Sígalas a menos que tenga un requisito concreto que las invalide.

### <a name="identity"></a>Identidad

La mayoría de las organizaciones empresariales tienen un entorno de servicios de directorio de Active Directory (ADDS) en su centro de datos local. Para facilitar la administración de los recursos trasladados a Azure desde la red local que depende de ADDS, se recomienda hospedar los controladores de dominio de ADDS en Azure.

Si desea utilizar los objetos de la directiva de grupo que desea controlar de forma independiente para Azure y para el entorno local, use un sitio de AD diferente para cada región de Azure. Sitúe los controladores de dominio en una red virtual central (centro) al que puedan acceder las cargas de trabajo dependientes.

### <a name="security"></a>Seguridad

Al trasladar cargas de trabajo desde el entorno local a Azure, algunas de estas cargas necesitarán hospedarse en máquinas virtuales. Por motivos de cumplimiento normativo, puede que tenga que aplicar restricciones al tráfico que recorre esas cargas de trabajo. 

Puede usar aplicaciones virtuales de red (NVA) en Azure para hospedar diferentes tipos de servicios de seguridad y rendimiento. Si está familiarizado con un conjunto determinado de aplicaciones locales actualmente, se recomienda que use las mismas aplicaciones virtualizadas en Azure, si es posible.

> [!NOTE]
> Los scripts de implementación de esta arquitectura de referencia usan una máquina virtual Ubuntu con reenvío IP habilitado para imitar una aplicación virtual de red.

## <a name="considerations"></a>Consideraciones

### <a name="overcoming-vnet-peering-limits"></a>Superación de los límites de emparejamiento de VNET

Asegúrese de tener en cuenta la [limitación del número de emparejamientos de VNET por cada red virtual][vnet-peering-limit] en Azure. Si decide que necesita más radios de los que el límite permitirá, considere la posibilidad de crear una topología en estrella tipo hub-hub-and-spoke-spoke, donde el primer nivel de radios también actúa como concentradores. En el siguiente diagrama se ilustra este enfoque.

![[3]][3]

También tenga en cuenta qué servicios se comparten en el concentrador, para asegurarse de que el concentrador se escala para tener un número mayor de radios. Por ejemplo, si el concentrador proporciona servicios de firewall, tenga en cuenta los límites de ancho de banda de la solución de firewall al agregar varios radios. Puede mover algunos de estos servicios compartidos a un segundo nivel de concentradores.

## <a name="deploy-the-solution"></a>Implementación de la solución

Hay disponible una implementación de esta arquitectura en [GitHub][ref-arch-repo]. Usa máquinas virtuales Ubuntu en cada red virtual para probar la conectividad. No hay ningún servicio real hospedado en la subred de **servicios compartidos** de la **red virtual del concentrador**.

### <a name="prerequisites"></a>requisitos previos

Antes de poder implementar la arquitectura de referencia en su propia suscripción, debe realizar los pasos siguientes.

1. Clone, bifurque o descargue el archivo ZIP para el repositorio de GitHub de [arquitecturas de referencia de AzureCAT][ref-arch-repo].

2. Asegúrese de que tiene la CLI de Azure 2.0 instalada en el equipo. Para obtener instrucciones sobre la instalación de la CLI, consulte [Instalación de la CLI de Azure 2.0][azure-cli-2].

3. Instale el paquete de NPM de [Azure Building Blocks][azbb].

4. Desde un símbolo del sistema, un símbolo del sistema de Bash o un símbolo del sistema de PowerShell, inicie sesión en la cuenta de Azure mediante el siguiente comando y siga las indicaciones.

  ```bash
  az login
  ```

### <a name="deploy-the-simulated-on-premises-datacenter-using-azbb"></a>Implementación del centro de datos local simulado mediante azbb

Para implementar el centro de datos local simulado como una red virtual de Azure, siga estos pasos:

1. Navegue hasta la carpeta `hybrid-networking\shared-services-stack\` del repositorio que descargó en el paso de requisitos previos anterior.

2. Abra el archivo `onprem.json` y escriba un nombre de usuario y la contraseña entre comillas en las líneas 45 y 46, tal y como se muestra a continuación, y después guarde el archivo.

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. Ejecute `azbb` para implementar el entorno simulado local tal y como se muestra a continuación.

  ```bash
  azbb -s <subscription_id> -g onprem-vnet-rg - l <location> -p onoprem.json --deploy
  ```
  > [!NOTE]
  > Si decide usar un nombre del grupo de recursos distinto (que no sea `onprem-vnet-rg`), asegúrese de que busca todos los archivos de parámetros que usan ese nombre y de que los modifica para usar el nombre de su propio grupo de recursos.

4. Espere a que finalice la implementación. Esta implementación crea una red virtual, una máquina virtual Windows y una instancia de VPN Gateway. La creación de la instancia de VPN Gateway puede durar más de cuarenta minutos.

### <a name="azure-hub-vnet"></a>Red virtual del concentrador de Azure

Para implementar la red virtual del concentrador y conectarla a la red virtual local simulada creada anteriormente, realice los pasos siguientes.

1. Abra el archivo `hub-vnet.json` y escriba un nombre de usuario y la contraseña entre comillas en las líneas 50 y 51, tal y como se muestra a continuación.

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

2. En la línea 52, para `osType`, escriba `Windows` o `Linux` para instalar Windows Server 2016 Datacenter o Ubuntu 16.04 como sistema operativo de JumpBox.

3. Escriba una clave compartida entre comillas en la línea 83, tal y como se muestra a continuación, y después guarde el archivo.

  ```bash
  "sharedKey": "",
  ```

4. Ejecute `azbb` para implementar el entorno simulado local tal y como se muestra a continuación.

  ```bash
  azbb -s <subscription_id> -g hub-vnet-rg - l <location> -p hub-vnet.json --deploy
  ```
  > [!NOTE]
  > Si decide usar un nombre del grupo de recursos distinto (que no sea `hub-vnet-rg`), asegúrese de que busca todos los archivos de parámetros que usan ese nombre y de que los modifica para usar el nombre de su propio grupo de recursos.

5. Espere a que finalice la implementación. Esta implementación crea una red virtual, una máquina virtual, una instancia de VPN Gateway y una conexión a la puerta de enlace creada en la sección anterior. La creación de la instancia de VPN Gateway puede durar más de cuarenta minutos.

### <a name="adds-in-azure"></a>ADDS en Azure

Para implementar los controladores de dominio de ADDS en Azure, realice los pasos siguientes.

1. Abra el archivo `hub-adds.json` y escriba un nombre de usuario y la contraseña entre comillas en las líneas 14 y 15, tal y como se muestra a continuación, y después guarde el archivo.

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

2. Ejecute `azbb` para implementar los controladores de dominio de ADDS tal y como se muestra a continuación.

  ```bash
  azbb -s <subscription_id> -g hub-adds-rg - l <location> -p hub-adds.json --deploy
  ```
  
  > [!NOTE]
  > Si decide usar un nombre del grupo de recursos distinto (que no sea `hub-adds-rg`), asegúrese de que busca todos los archivos de parámetros que usan ese nombre y de que los modifica para usar el nombre de su propio grupo de recursos.

  > [!NOTE]
  > Esta parte de la implementación puede tardar varios minutos, ya que requiere unir las dos máquinas virtuales al dominio hospedado en el centro de datos local simulado y, a continuación, instalar AD DS en ellas.

### <a name="nva"></a>Aplicación virtual de red

Para implementar una aplicación virtual de red en la subred `dmz`, realice los pasos siguientes:

1. Abra el archivo `hub-nva.json` y escriba un nombre de usuario y la contraseña entre comillas en las líneas 13 y 14, tal y como se muestra a continuación, y después guarde el archivo.

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```
2. Ejecute `azbb` para implementar la máquina virtual de la aplicación virtual de red y las rutas definidas por el usuario.

  ```bash
  azbb -s <subscription_id> -g hub-nva-rg - l <location> -p hub-nva.json --deploy
  ```
  > [!NOTE]
  > Si decide usar un nombre del grupo de recursos distinto (que no sea `hub-nva-rg`), asegúrese de que busca todos los archivos de parámetros que usan ese nombre y de que los modifica para usar el nombre de su propio grupo de recursos.

### <a name="azure-spoke-vnets"></a>Redes virtuales de radios de Azure

Para implementar las redes virtuales de radios, siga estos pasos.

1. Abra el archivo `spoke1.json` y escriba un nombre de usuario y la contraseña entre comillas en las líneas 52 y 53, tal y como se muestra a continuación, y después guarde el archivo.

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

2. En la línea 54, para `osType`, escriba `Windows` o `Linux` para instalar Windows Server 2016 Datacenter o Ubuntu 16.04 como sistema operativo de JumpBox.

3. Ejecute `azbb` para implementar el entorno de red virtual del primer radio tal y como se muestra a continuación.

  ```bash
  azbb -s <subscription_id> -g spoke1-vnet-rg - l <location> -p spoke1.json --deploy
  ```
  
  > [!NOTE]
  > Si decide usar un nombre del grupo de recursos distinto (que no sea `spoke1-vnet-rg`), asegúrese de que busca todos los archivos de parámetros que usan ese nombre y de que los modifica para usar el nombre de su propio grupo de recursos.

3. Repita el paso 1 anterior para el archivo `spoke2.json`.

4. Ejecute `azbb` para implementar el entorno de red virtual del segundo radio tal y como se muestra a continuación.

  ```bash
  azbb -s <subscription_id> -g spoke2-vnet-rg - l <location> -p spoke2.json --deploy
  ```
  > [!NOTE]
  > Si decide usar un nombre del grupo de recursos distinto (que no sea `spoke2-vnet-rg`), asegúrese de que busca todos los archivos de parámetros que usan ese nombre y de que los modifica para usar el nombre de su propio grupo de recursos.

### <a name="azure-hub-vnet-peering-to-spoke-vnets"></a>Emparejamiento de VNET del concentrador de Azure a las redes virtuales de los radios

Para crear una conexión de emparejamiento desde la red virtual del concentrador a las redes virtuales de radios, realice los siguientes pasos.

1. Abra el archivo `hub-vnet-peering.json` y compruebe que el nombre del grupo de recursos y el nombre de la red virtual para cada uno de los emparejamientos de la red virtual que empiezan en la línea 29 son correctos.

2. Ejecute `azbb` para implementar el entorno de red virtual del primer radio tal y como se muestra a continuación.

  ```bash
  azbb -s <subscription_id> -g hub-vnet-rg - l <location> -p hub-vnet-peering.json --deploy
  ```

  > [!NOTE]
  > Si decide usar un nombre del grupo de recursos distinto (que no sea `hub-vnet-rg`), asegúrese de que busca todos los archivos de parámetros que usan ese nombre y de que los modifica para usar el nombre de su propio grupo de recursos.

<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[guidance-hub-spoke]: ./hub-spoke.md
[azure-vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[best-practices-security]: /azure/best-practices-network-securit
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[guidance-expressroute]: ./expressroute.md
[guidance-vpn]: ./vpn.md
[linux-vm-ra]: ../virtual-machines-linux/index.md
[hybrid-ha]: ./expressroute-vpn-failover.md
[naming conventions]: /azure/guidance/guidance-naming-conventions
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[centro de datos virtual]: https://aka.ms/vdc
[vnet-peering]: /azure/virtual-network/virtual-network-peering-overview
[vnet-peering-limit]: /azure/azure-subscription-service-limits#networking-limits
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[windows-vm-ra]: ../virtual-machines-windows/index.md

[visio-download]: https://archcenter.azureedge.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/shared-services.png "Topología de servicios compartidos en Azure"
[3]: ./images/hub-spokehub-spoke.svg "Topología en estrella tipo hub-hub-and-spoke-spoke en Azure"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
