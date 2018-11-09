---
title: Implementación de una arquitectura de red híbrida de alta disponibilidad
description: Cómo implementar una arquitectura de red de sitio a sitio segura que abarque una instancia de Azure Virtual Network y una red local conectada mediante ExpressRoute con conmutación por error de VPN Gateway.
author: telmosampaio
ms.date: 10/22/2017
pnp.series.title: Connect an on-premises network to Azure
pnp.series.prev: expressroute
cardTitle: Improving availability
ms.openlocfilehash: 31bf471dbff3661face7d94fbec0973d81541ec7
ms.sourcegitcommit: dbbf914757b03cdee7a274204f9579fa63d7eed2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/02/2018
ms.locfileid: "50916420"
---
# <a name="connect-an-on-premises-network-to-azure-using-expressroute-with-vpn-failover"></a>Conexión de una red local a Azure mediante ExpressRoute con conmutación por error de VPN

Esta arquitectura de referencia muestra cómo conectar una red local a una red Azure Virtual Network (VNet) mediante ExpressRoute, con una red privada virtual (VPN) de sitio a sitio como conexión de conmutación por error. El tráfico fluye entre la red local y la red virtual de Azure a través de una conexión de ExpressRoute. Si se produce una pérdida de conectividad en el circuito de ExpressRoute, el tráfico se enruta a través de un túnel de VPN con IPSec. [**Implemente esta solución**.](#deploy-the-solution)

Tenga en cuenta que si el circuito de ExpressRoute no está disponible, la ruta VPN solo se ocupará de conexiones entre pares privados. Las conexiones entre pares públicos y de Microsoft pasarán por Internet. 

![[0]][0]

*Descargue un [archivo Visio][visio-download] de esta arquitectura.*

## <a name="architecture"></a>Arquitectura 

La arquitectura consta de los siguientes componentes:

* **Red local**. Una red de área local privada que se ejecuta dentro de una organización.

* **Dispositivo VPN**. Un dispositivo o servicio que proporciona conectividad externa a la red local. El dispositivo VPN puede ser un dispositivo de hardware, o puede ser una solución de software como el servicio de Enrutamiento y acceso remoto (RRAS) en Windows Server 2012. Para obtener una lista de dispositivos VPN compatibles e información acerca de cómo configurar dispositivos VPN seleccionados para conectarse a Azure, consulte [Acerca de los dispositivos VPN y los parámetros de IPsec/IKE para conexiones de VPN Gateway de sitio a sitio][vpn-appliance].

* **Circuito ExpressRoute**. Un circuito de capa 2 o capa 3 suministrado por el proveedor de conectividad que se une a la red local con Azure a través de los enrutadores perimetrales. El circuito usa la infraestructura de hardware administrada por el proveedor de conectividad.

* **Puerta de enlace de red virtual de ExpressRoute**. La puerta de enlace de red virtual de ExpressRoute permite que la red virtual se conecte al circuito de ExpressRoute que se usa para la conectividad con la red local.

* **Puerta de enlace de red virtual de VPN** La puerta de enlace de red virtual de VPN permite que la red virtual se conecte al dispositivo VPN en la red local. La puerta de enlace de red virtual de VPN está configurada para aceptar las solicitudes procedentes de la red local solo a través del dispositivo VPN. Para más información, consulte [Conectar una red local con una red virtual de Microsoft Azure][connect-to-an-Azure-vnet].

* **Conexión VPN**. La conexión tiene propiedades que especifican el tipo de conexión (IPSec) y la clave compartida con el dispositivo VPN local para cifrar el tráfico.

* **Azure Virtual Network (VNet)**. Cada red virtual se encuentra en una sola región de Azure y puede hospedar varios niveles de aplicación. Los niveles de aplicación se pueden segmentar con subredes en cada red virtual.

* **Subred de puerta de enlace**. Las puertas de enlace de red virtual se conservan en la misma subred.

* **Aplicación en la nube**. La aplicación hospedada en Azure. Puede incluir varios niveles, con varias subredes que se conectan a través de equilibradores de carga de Azure. Para obtener más información acerca de la infraestructura de aplicaciones, consulte [Ejecutar cargas de trabajo de máquinas virtuales Windows][windows-vm-ra] y [Ejecutar cargas de trabajo de máquinas virtuales Linux][linux-vm-ra].

## <a name="recommendations"></a>Recomendaciones

Las siguientes recomendaciones sirven para la mayoría de los escenarios. Sígalas a menos que tenga un requisito concreto que las invalide.

### <a name="vnet-and-gatewaysubnet"></a>VNet y GatewaySubnet

Cree la puerta de enlace de red virtual de ExpressRoute y la puerta de enlace de red virtual de VPN en la misma red virtual. Esto significa que deben compartir la misma subred denominada *GatewaySubnet*.

Si la red virtual ya incluye una subred denominada *GatewaySubnet*, asegúrese de que tiene un espacio de direcciones de /27 o más grande. Si la subred es demasiado pequeña, use el siguiente comando de PowerShell para quitarla: 

```powershell
$vnet = Get-AzureRmVirtualNetworkGateway -Name <yourvnetname> -ResourceGroupName <yourresourcegroup>
Remove-AzureRmVirtualNetworkSubnetConfig -Name GatewaySubnet -VirtualNetwork $vnet
```

Si la red virtual no contiene una subred denominada **GatewaySubnet**, cree una nueva con el siguiente comando de Powershell:

```powershell
$vnet = Get-AzureRmVirtualNetworkGateway -Name <yourvnetname> -ResourceGroupName <yourresourcegroup>
Add-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet -AddressPrefix "10.200.255.224/27"
$vnet = Set-AzureRmVirtualNetwork -VirtualNetwork $vnet
```

### <a name="vpn-and-expressroute-gateways"></a>VPN y puertas de enlace de ExpressRoute

Compruebe que su organización cumple los [requisitos previos de ExpressRoute][expressroute-prereq] para conectarse a Azure.

Si ya tiene una puerta de enlace de red virtual de VPN en la red virtual de Azure, use el siguiente comando de Powershell para quitarla:

```powershell
Remove-AzureRmVirtualNetworkGateway -Name <yourgatewayname> -ResourceGroupName <yourresourcegroup>
```

Siga las instrucciones de [Implementing a hybrid network architecture with Azure ExpressRoute][implementing-expressroute] (Implementación de una arquitectura de red híbrida con Azure ExpressRoute) para establecer la conexión de ExpressRoute.

Siga las instrucciones de [Implementing a hybrid network architecture with Azure and On-premises VPN][implementing-vpn] (Implementación de una arquitectura de red híbrida con Azure y VPN local) para establecer la conexión de puerta de enlace de red virtual de VPN.

Después de haber establecido las conexiones de puerta de enlace de red virtual, pruebe el entorno como sigue:

1. Asegúrese de que puede conectarse desde su red local a la red virtual de Azure.
2. Póngase en contacto con su proveedor para detener la conectividad de ExpressRoute para la prueba.
3. Compruebe que todavía puede conectarse desde su red local a la red virtual de Azure mediante la conexión de puerta de enlace de red virtual de VPN.
4. Póngase en contacto con su proveedor para restablecer la conectividad de ExpressRoute.

## <a name="considerations"></a>Consideraciones

Para conocer las consideraciones de ExpressRoute, consulte las instrucciones de [Implementing a hybrid network architecture with Azure ExpressRoute][guidance-expressroute] (Implementación de una arquitectura de red híbrida con Azure ExpressRoute).

Para conocer las consideraciones de ExpressRoute, consulte las instrucciones de [Implementing a hybrid network architecture with Azure and On-premises VPN][guidance-vpn] (Implementación de una arquitectura de red híbrida con Azure y VPN local).

Para obtener consideraciones generales de seguridad de Azure, consulte [Servicios en la nube de Microsoft y seguridad de red][best-practices-security].

## <a name="deploy-the-solution"></a>Implementación de la solución

**Requisitos previos**. Debe tener una infraestructura local existente ya configurada con un dispositivo de red adecuado.

Para implementar la solución, siga estos pasos:

1. Haga clic en el botón a continuación:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute-vpn-failover%2Fazuredeploy.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. Espere a que el vínculo abra Azure Portal y, a continuación, siga estos pasos:   
   * El nombre del **Grupo de recursos** ya está definido en el archivo de parámetros, así que seleccione **Crear nuevo** y escriba `ra-hybrid-vpn-er-rg` en el cuadro de texto.
   * Seleccione la región en el cuadro de lista desplegable **Ubicación**.
   * No modifique los cuadros de texto **URI raíz de plantilla** o **URI raíz de parámetro**.
   * Revise los términos y condiciones, y haga clic en la casilla **Acepto los términos y condiciones indicados anteriormente**.
   * Haga clic en el botón **Comprar**.
3. Espere a que la implementación se complete.
4. Haga clic en el botón a continuación:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute-vpn-failover%2Fazuredeploy-expressRouteCircuit.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
5. Espere a que el vínculo abra Azure Portal y, a continuación, siga estos pasos:
   * Seleccione **Usar existente** en la sección **Grupo de recursos** y escriba `ra-hybrid-vpn-er-rg` en el cuadro de texto.
   * Seleccione la región en el cuadro de lista desplegable **Ubicación**.
   * No modifique los cuadros de texto **URI raíz de plantilla** o **URI raíz de parámetro**.
   * Revise los términos y condiciones, y haga clic en la casilla **Acepto los términos y condiciones indicados anteriormente**.
   * Haga clic en el botón **Comprar**.

<!-- links -->

[windows-vm-ra]: ../virtual-machines-windows/index.md
[linux-vm-ra]: ../virtual-machines-linux/index.md


[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[azure-vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[expressroute-prereq]: /azure/expressroute/expressroute-prerequisites
[implementing-expressroute]: ./expressroute.md
[implementing-vpn]: ./vpn.md
[guidance-expressroute]: ./expressroute.md
[guidance-vpn]: ./vpn.md
[best-practices-security]: /azure/best-practices-network-security
[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-architectures.vsdx
[0]: ./images/expressroute-vpn-failover.png "Arquitectura de una arquitectura de red híbrida de alta disponibilidad mediante ExpressRoute y VPN gateway"
