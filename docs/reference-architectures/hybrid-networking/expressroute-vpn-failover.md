---
title: Implementación de una arquitectura de red híbrida de alta disponibilidad
description: Cómo implementar una arquitectura de red de sitio a sitio segura que abarque una instancia de Azure Virtual Network y una red local conectada mediante ExpressRoute con conmutación por error de VPN Gateway.
author: telmosampaio
ms.date: 11/28/2016
pnp.series.title: Connect an on-premises network to Azure
pnp.series.prev: expressroute
cardTitle: Improving availability
ms.openlocfilehash: 81298215c814cee805eff57fdc28f7c127148b5f
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2018
---
# <a name="connect-an-on-premises-network-to-azure-using-expressroute-with-vpn-failover"></a><span data-ttu-id="85183-103">Conexión de una red local a Azure mediante ExpressRoute con conmutación por error de VPN</span><span class="sxs-lookup"><span data-stu-id="85183-103">Connect an on-premises network to Azure using ExpressRoute with VPN failover</span></span>

<span data-ttu-id="85183-104">Esta arquitectura de referencia muestra cómo conectar una red local a una red Azure Virtual Network (VNet) mediante ExpressRoute, con una red privada virtual (VPN) de sitio a sitio como conexión de conmutación por error.</span><span class="sxs-lookup"><span data-stu-id="85183-104">This reference architecture shows how to connect an on-premises network to an Azure virtual network (VNet) using ExpressRoute, with a site-to-site virtual private network (VPN) as a failover connection.</span></span> <span data-ttu-id="85183-105">El tráfico fluye entre la red local y la red virtual de Azure a través de una conexión de ExpressRoute.</span><span class="sxs-lookup"><span data-stu-id="85183-105">Traffic flows between the on-premises network and the Azure VNet through an ExpressRoute connection.</span></span> <span data-ttu-id="85183-106">Si se produce una pérdida de conectividad en el circuito de ExpressRoute, el tráfico se enruta a través de un túnel de VPN con IPSec.</span><span class="sxs-lookup"><span data-stu-id="85183-106">If there is a loss of connectivity in the ExpressRoute circuit, traffic is routed through an IPSec VPN tunnel.</span></span> [<span data-ttu-id="85183-107">**Implemente esta solución**.</span><span class="sxs-lookup"><span data-stu-id="85183-107">**Deploy this solution**.</span></span>](#deploy-the-solution)

<span data-ttu-id="85183-108">Tenga en cuenta que si el circuito de ExpressRoute no está disponible, la ruta VPN solo se ocupará de conexiones entre pares privados.</span><span class="sxs-lookup"><span data-stu-id="85183-108">Note that if the ExpressRoute circuit is unavailable, the VPN route will only handle private peering connections.</span></span> <span data-ttu-id="85183-109">Las conexiones entre pares públicos y de Microsoft pasarán por Internet.</span><span class="sxs-lookup"><span data-stu-id="85183-109">Public peering and Microsoft peering connections will pass over the Internet.</span></span> 

<span data-ttu-id="85183-110">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="85183-110">![[0]][0]</span></span>

<span data-ttu-id="85183-111">*Descargue un [archivo Visio][visio-download] de esta arquitectura.*</span><span class="sxs-lookup"><span data-stu-id="85183-111">*Download a [Visio file][visio-download] of this architecture.*</span></span>

## <a name="architecture"></a><span data-ttu-id="85183-112">Architecture</span><span class="sxs-lookup"><span data-stu-id="85183-112">Architecture</span></span> 

<span data-ttu-id="85183-113">La arquitectura consta de los siguientes componentes:</span><span class="sxs-lookup"><span data-stu-id="85183-113">The architecture consists of the following components.</span></span>

* <span data-ttu-id="85183-114">**Red local**.</span><span class="sxs-lookup"><span data-stu-id="85183-114">**On-premises network**.</span></span> <span data-ttu-id="85183-115">Una red de área local privada que se ejecuta dentro de una organización.</span><span class="sxs-lookup"><span data-stu-id="85183-115">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="85183-116">**Dispositivo VPN**.</span><span class="sxs-lookup"><span data-stu-id="85183-116">**VPN appliance**.</span></span> <span data-ttu-id="85183-117">Un dispositivo o servicio que proporciona conectividad externa a la red local.</span><span class="sxs-lookup"><span data-stu-id="85183-117">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="85183-118">El dispositivo VPN puede ser un dispositivo de hardware, o puede ser una solución de software como el servicio de Enrutamiento y acceso remoto (RRAS) en Windows Server 2012.</span><span class="sxs-lookup"><span data-stu-id="85183-118">The VPN appliance may be a hardware device, or it can be a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="85183-119">Para obtener una lista de dispositivos VPN compatibles e información acerca de cómo configurar dispositivos VPN seleccionados para conectarse a Azure, consulte [Acerca de los dispositivos VPN y los parámetros de IPsec/IKE para conexiones de VPN Gateway de sitio a sitio][vpn-appliance].</span><span class="sxs-lookup"><span data-stu-id="85183-119">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="85183-120">**Circuito ExpressRoute**.</span><span class="sxs-lookup"><span data-stu-id="85183-120">**ExpressRoute circuit**.</span></span> <span data-ttu-id="85183-121">Un circuito de capa 2 o capa 3 suministrado por el proveedor de conectividad que se une a la red local con Azure a través de los enrutadores perimetrales.</span><span class="sxs-lookup"><span data-stu-id="85183-121">A layer 2 or layer 3 circuit supplied by the connectivity provider that joins the on-premises network with Azure through the edge routers.</span></span> <span data-ttu-id="85183-122">El circuito usa la infraestructura de hardware administrada por el proveedor de conectividad.</span><span class="sxs-lookup"><span data-stu-id="85183-122">The circuit uses the hardware infrastructure managed by the connectivity provider.</span></span>

* <span data-ttu-id="85183-123">**Puerta de enlace de red virtual de ExpressRoute**.</span><span class="sxs-lookup"><span data-stu-id="85183-123">**ExpressRoute virtual network gateway**.</span></span> <span data-ttu-id="85183-124">La puerta de enlace de red virtual de ExpressRoute permite que la red virtual se conecte al circuito de ExpressRoute que se usa para la conectividad con la red local.</span><span class="sxs-lookup"><span data-stu-id="85183-124">The ExpressRoute virtual network gateway enables the VNet to connect to the ExpressRoute circuit used for connectivity with your on-premises network.</span></span>

* <span data-ttu-id="85183-125">**Puerta de enlace de red virtual de VPN**</span><span class="sxs-lookup"><span data-stu-id="85183-125">**VPN virtual network gateway**.</span></span> <span data-ttu-id="85183-126">La puerta de enlace de red virtual de VPN permite que la red virtual se conecte al dispositivo VPN en la red local.</span><span class="sxs-lookup"><span data-stu-id="85183-126">The VPN virtual network gateway enables the VNet to connect to the VPN appliance in the on-premises network.</span></span> <span data-ttu-id="85183-127">La puerta de enlace de red virtual de VPN está configurada para aceptar las solicitudes procedentes de la red local solo a través del dispositivo VPN.</span><span class="sxs-lookup"><span data-stu-id="85183-127">The VPN virtual network gateway is configured to accept requests from the on-premises network only through the VPN appliance.</span></span> <span data-ttu-id="85183-128">Para más información, consulte [Conectar una red local con una red virtual de Microsoft Azure][connect-to-an-Azure-vnet].</span><span class="sxs-lookup"><span data-stu-id="85183-128">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

* <span data-ttu-id="85183-129">**Conexión VPN**.</span><span class="sxs-lookup"><span data-stu-id="85183-129">**VPN connection**.</span></span> <span data-ttu-id="85183-130">La conexión tiene propiedades que especifican el tipo de conexión (IPSec) y la clave compartida con el dispositivo VPN local para cifrar el tráfico.</span><span class="sxs-lookup"><span data-stu-id="85183-130">The connection has properties that specify the connection type (IPSec) and the key shared with the on-premises VPN appliance to encrypt traffic.</span></span>

* <span data-ttu-id="85183-131">**Azure Virtual Network (VNet)**.</span><span class="sxs-lookup"><span data-stu-id="85183-131">**Azure Virtual Network (VNet)**.</span></span> <span data-ttu-id="85183-132">Cada red virtual se encuentra en una sola región de Azure y puede hospedar varios niveles de aplicación.</span><span class="sxs-lookup"><span data-stu-id="85183-132">Each VNet resides in a single Azure region, and can host multiple application tiers.</span></span> <span data-ttu-id="85183-133">Los niveles de aplicación se pueden segmentar con subredes en cada red virtual.</span><span class="sxs-lookup"><span data-stu-id="85183-133">Application tiers can be segmented using subnets in each VNet.</span></span>

* <span data-ttu-id="85183-134">**Subred de puerta de enlace**.</span><span class="sxs-lookup"><span data-stu-id="85183-134">**Gateway subnet**.</span></span> <span data-ttu-id="85183-135">Las puertas de enlace de red virtual se conservan en la misma subred.</span><span class="sxs-lookup"><span data-stu-id="85183-135">The virtual network gateways are held in the same subnet.</span></span>

* <span data-ttu-id="85183-136">**Aplicación en la nube**.</span><span class="sxs-lookup"><span data-stu-id="85183-136">**Cloud application**.</span></span> <span data-ttu-id="85183-137">La aplicación hospedada en Azure.</span><span class="sxs-lookup"><span data-stu-id="85183-137">The application hosted in Azure.</span></span> <span data-ttu-id="85183-138">Puede incluir varios niveles, con varias subredes que se conectan a través de equilibradores de carga de Azure.</span><span class="sxs-lookup"><span data-stu-id="85183-138">It might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="85183-139">Para obtener más información acerca de la infraestructura de aplicaciones, consulte [Ejecutar cargas de trabajo de máquinas virtuales Windows][windows-vm-ra] y [Ejecutar cargas de trabajo de máquinas virtuales Linux][linux-vm-ra].</span><span class="sxs-lookup"><span data-stu-id="85183-139">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

## <a name="recommendations"></a><span data-ttu-id="85183-140">Recomendaciones</span><span class="sxs-lookup"><span data-stu-id="85183-140">Recommendations</span></span>

<span data-ttu-id="85183-141">Las siguientes recomendaciones sirven para la mayoría de los escenarios.</span><span class="sxs-lookup"><span data-stu-id="85183-141">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="85183-142">Sígalas a menos que tenga un requisito concreto que las invalide.</span><span class="sxs-lookup"><span data-stu-id="85183-142">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="vnet-and-gatewaysubnet"></a><span data-ttu-id="85183-143">VNet y GatewaySubnet</span><span class="sxs-lookup"><span data-stu-id="85183-143">VNet and GatewaySubnet</span></span>

<span data-ttu-id="85183-144">Cree la puerta de enlace de red virtual de ExpressRoute y la puerta de enlace de red virtual de VPN en la misma red virtual.</span><span class="sxs-lookup"><span data-stu-id="85183-144">Create the ExpressRoute virtual network gateway and the VPN virtual network gateway in the same VNet.</span></span> <span data-ttu-id="85183-145">Esto significa que deben compartir la misma subred denominada *GatewaySubnet*.</span><span class="sxs-lookup"><span data-stu-id="85183-145">This means that they should share the same subnet named *GatewaySubnet*.</span></span>

<span data-ttu-id="85183-146">Si la red virtual ya incluye una subred denominada *GatewaySubnet*, asegúrese de que tiene un espacio de direcciones de /27 o más grande.</span><span class="sxs-lookup"><span data-stu-id="85183-146">If the VNet already includes a subnet named *GatewaySubnet*, ensure that it has a /27 or larger address space.</span></span> <span data-ttu-id="85183-147">Si la subred es demasiado pequeña, use el siguiente comando de PowerShell para quitarla:</span><span class="sxs-lookup"><span data-stu-id="85183-147">If the existing subnet is too small, use the following PowerShell command to remove the subnet:</span></span> 

```powershell
$vnet = Get-AzureRmVirtualNetworkGateway -Name <yourvnetname> -ResourceGroupName <yourresourcegroup>
Remove-AzureRmVirtualNetworkSubnetConfig -Name GatewaySubnet -VirtualNetwork $vnet
```

<span data-ttu-id="85183-148">Si la red virtual no contiene una subred denominada **GatewaySubnet**, cree una nueva con el siguiente comando de Powershell:</span><span class="sxs-lookup"><span data-stu-id="85183-148">If the VNet does not contain a subnet named **GatewaySubnet**, create a new one using the following Powershell command:</span></span>

```powershell
$vnet = Get-AzureRmVirtualNetworkGateway -Name <yourvnetname> -ResourceGroupName <yourresourcegroup>
Add-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet -AddressPrefix "10.200.255.224/27"
$vnet = Set-AzureRmVirtualNetwork -VirtualNetwork $vnet
```

### <a name="vpn-and-expressroute-gateways"></a><span data-ttu-id="85183-149">VPN y puertas de enlace de ExpressRoute</span><span class="sxs-lookup"><span data-stu-id="85183-149">VPN and ExpressRoute gateways</span></span>

<span data-ttu-id="85183-150">Compruebe que su organización cumple los [requisitos previos de ExpressRoute][expressroute-prereq] para conectarse a Azure.</span><span class="sxs-lookup"><span data-stu-id="85183-150">Verify that your organization meets the [ExpressRoute prerequisite requirements][expressroute-prereq] for connecting to Azure.</span></span>

<span data-ttu-id="85183-151">Si ya tiene una puerta de enlace de red virtual de VPN en la red virtual de Azure, use el siguiente comando de Powershell para quitarla:</span><span class="sxs-lookup"><span data-stu-id="85183-151">If you already have a VPN virtual network gateway in your Azure VNet, use the following  Powershell command to remove it:</span></span>

```powershell
Remove-AzureRmVirtualNetworkGateway -Name <yourgatewayname> -ResourceGroupName <yourresourcegroup>
```

<span data-ttu-id="85183-152">Siga las instrucciones de [Implementing a hybrid network architecture with Azure ExpressRoute][implementing-expressroute] (Implementación de una arquitectura de red híbrida con Azure ExpressRoute) para establecer la conexión de ExpressRoute.</span><span class="sxs-lookup"><span data-stu-id="85183-152">Follow the instructions in [Implementing a hybrid network architecture with Azure ExpressRoute][implementing-expressroute] to establish your ExpressRoute connection.</span></span>

<span data-ttu-id="85183-153">Siga las instrucciones de [Implementing a hybrid network architecture with Azure and On-premises VPN][implementing-vpn] (Implementación de una arquitectura de red híbrida con Azure y VPN local) para establecer la conexión de puerta de enlace de red virtual de VPN.</span><span class="sxs-lookup"><span data-stu-id="85183-153">Follow the instructions in [Implementing a hybrid network architecture with Azure and On-premises VPN][implementing-vpn] to establish your VPN virtual network gateway connection.</span></span>

<span data-ttu-id="85183-154">Después de haber establecido las conexiones de puerta de enlace de red virtual, pruebe el entorno como sigue:</span><span class="sxs-lookup"><span data-stu-id="85183-154">After you have established the virtual network gateway connections, test the environment as follows:</span></span>

1. <span data-ttu-id="85183-155">Asegúrese de que puede conectarse desde su red local a la red virtual de Azure.</span><span class="sxs-lookup"><span data-stu-id="85183-155">Make sure you can connect from your on-premises network to your Azure VNet.</span></span>
2. <span data-ttu-id="85183-156">Póngase en contacto con su proveedor para detener la conectividad de ExpressRoute para la prueba.</span><span class="sxs-lookup"><span data-stu-id="85183-156">Contact your provider to stop ExpressRoute connectivity for testing.</span></span>
3. <span data-ttu-id="85183-157">Compruebe que todavía puede conectarse desde su red local a la red virtual de Azure mediante la conexión de puerta de enlace de red virtual de VPN.</span><span class="sxs-lookup"><span data-stu-id="85183-157">Verify that you can still connect from your on-premises network to your Azure VNet using the VPN virtual network gateway connection.</span></span>
4. <span data-ttu-id="85183-158">Póngase en contacto con su proveedor para restablecer la conectividad de ExpressRoute.</span><span class="sxs-lookup"><span data-stu-id="85183-158">Contact your provider to reestablish ExpressRoute connectivity.</span></span>

## <a name="considerations"></a><span data-ttu-id="85183-159">Consideraciones</span><span class="sxs-lookup"><span data-stu-id="85183-159">Considerations</span></span>

<span data-ttu-id="85183-160">Para conocer las consideraciones de ExpressRoute, consulte las instrucciones de [Implementing a hybrid network architecture with Azure ExpressRoute][guidance-expressroute] (Implementación de una arquitectura de red híbrida con Azure ExpressRoute).</span><span class="sxs-lookup"><span data-stu-id="85183-160">For ExpressRoute considerations, see the [Implementing a Hybrid Network Architecture with Azure ExpressRoute][guidance-expressroute] guidance.</span></span>

<span data-ttu-id="85183-161">Para conocer las consideraciones de ExpressRoute, consulte las instrucciones de [Implementing a hybrid network architecture with Azure and On-premises VPN][guidance-vpn] (Implementación de una arquitectura de red híbrida con Azure y VPN local).</span><span class="sxs-lookup"><span data-stu-id="85183-161">For site-to-site VPN considerations, see the [Implementing a Hybrid Network Architecture with Azure and On-premises VPN][guidance-vpn] guidance.</span></span>

<span data-ttu-id="85183-162">Para obtener consideraciones generales de seguridad de Azure, consulte [Servicios en la nube de Microsoft y seguridad de red][best-practices-security].</span><span class="sxs-lookup"><span data-stu-id="85183-162">For general Azure security considerations, see [Microsoft cloud services and network security][best-practices-security].</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="85183-163">Implementación de la solución</span><span class="sxs-lookup"><span data-stu-id="85183-163">Deploy the solution</span></span>

<span data-ttu-id="85183-164">**Requisitos previos**.</span><span class="sxs-lookup"><span data-stu-id="85183-164">**Prequisites.**</span></span> <span data-ttu-id="85183-165">Debe tener una infraestructura local existente ya configurada con un dispositivo de red adecuado.</span><span class="sxs-lookup"><span data-stu-id="85183-165">You must have an existing on-premises infrastructure already configured with a suitable network appliance.</span></span>

<span data-ttu-id="85183-166">Para implementar la solución, siga estos pasos:</span><span class="sxs-lookup"><span data-stu-id="85183-166">To deploy the solution, perform the following steps.</span></span>

1. <span data-ttu-id="85183-167">Haga clic en el botón a continuación:</span><span class="sxs-lookup"><span data-stu-id="85183-167">Click the button below:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute-vpn-failover%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. <span data-ttu-id="85183-168">Espere a que el vínculo abra Azure Portal y, a continuación, siga estos pasos:</span><span class="sxs-lookup"><span data-stu-id="85183-168">Wait for the link to open in the Azure portal, then follow these steps:</span></span>   
   * <span data-ttu-id="85183-169">El nombre del **Grupo de recursos** ya está definido en el archivo de parámetros, así que seleccione **Crear nuevo** y escriba `ra-hybrid-vpn-er-rg` en el cuadro de texto.</span><span class="sxs-lookup"><span data-stu-id="85183-169">The **Resource group** name is already defined in the parameter file, so select **Create New** and enter `ra-hybrid-vpn-er-rg` in the text box.</span></span>
   * <span data-ttu-id="85183-170">Seleccione la región en el cuadro de lista desplegable **Ubicación**.</span><span class="sxs-lookup"><span data-stu-id="85183-170">Select the region from the **Location** drop down box.</span></span>
   * <span data-ttu-id="85183-171">No modifique los cuadros de texto **URI raíz de plantilla** o **URI raíz de parámetro**.</span><span class="sxs-lookup"><span data-stu-id="85183-171">Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.</span></span>
   * <span data-ttu-id="85183-172">Revise los términos y condiciones, y haga clic en la casilla **Acepto los términos y condiciones indicados anteriormente**.</span><span class="sxs-lookup"><span data-stu-id="85183-172">Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.</span></span>
   * <span data-ttu-id="85183-173">Haga clic en el botón **Comprar**.</span><span class="sxs-lookup"><span data-stu-id="85183-173">Click the **Purchase** button.</span></span>
3. <span data-ttu-id="85183-174">Espere a que la implementación se complete.</span><span class="sxs-lookup"><span data-stu-id="85183-174">Wait for the deployment to complete.</span></span>
4. <span data-ttu-id="85183-175">Haga clic en el botón a continuación:</span><span class="sxs-lookup"><span data-stu-id="85183-175">Click the button below:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute-vpn-failover%2Fazuredeploy-expressRouteCircuit.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
5. <span data-ttu-id="85183-176">Espere a que el vínculo abra Azure Portal y, a continuación, siga estos pasos:</span><span class="sxs-lookup"><span data-stu-id="85183-176">Wait for the link to open in the Azure portal, then enter then follow these steps:</span></span>
   * <span data-ttu-id="85183-177">Seleccione **Usar existente** en la sección **Grupo de recursos** y escriba `ra-hybrid-vpn-er-rg` en el cuadro de texto.</span><span class="sxs-lookup"><span data-stu-id="85183-177">Select **Use existing** in the **Resource group** section and enter `ra-hybrid-vpn-er-rg` in the text box.</span></span>
   * <span data-ttu-id="85183-178">Seleccione la región en el cuadro de lista desplegable **Ubicación**.</span><span class="sxs-lookup"><span data-stu-id="85183-178">Select the region from the **Location** drop down box.</span></span>
   * <span data-ttu-id="85183-179">No modifique los cuadros de texto **URI raíz de plantilla** o **URI raíz de parámetro**.</span><span class="sxs-lookup"><span data-stu-id="85183-179">Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.</span></span>
   * <span data-ttu-id="85183-180">Revise los términos y condiciones, y haga clic en la casilla **Acepto los términos y condiciones indicados anteriormente**.</span><span class="sxs-lookup"><span data-stu-id="85183-180">Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.</span></span>
   * <span data-ttu-id="85183-181">Haga clic en el botón **Comprar**.</span><span class="sxs-lookup"><span data-stu-id="85183-181">Click the **Purchase** button.</span></span>

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
