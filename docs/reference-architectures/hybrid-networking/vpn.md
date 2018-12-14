---
title: Conexión de una red local a Azure mediante VPN
titleSuffix: Azure Reference Architectures
description: Implemente una arquitectura de red de sitio a sitio segura que abarque una red virtual de Azure y una red local conectada mediante VPN.
author: RohitSharma-pnp
ms.date: 10/22/2018
ms.openlocfilehash: a1bb2e250cb261e1a56abfb58b099fd078c068e5
ms.sourcegitcommit: 88a68c7e9b6b772172b7faa4b9fd9c061a9f7e9d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/08/2018
ms.locfileid: "53120448"
---
# <a name="connect-an-on-premises-network-to-azure-using-a-vpn-gateway"></a><span data-ttu-id="a1d8a-103">Conexión de una red local a Azure mediante VPN Gateway</span><span class="sxs-lookup"><span data-stu-id="a1d8a-103">Connect an on-premises network to Azure using a VPN gateway</span></span>

<span data-ttu-id="a1d8a-104">Esta arquitectura de referencia muestra cómo extender una red local a Azure mediante una red privada virtual (VPN) de sitio a sitio.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-104">This reference architecture shows how to extend an on-premises network to Azure, using a site-to-site virtual private network (VPN).</span></span> <span data-ttu-id="a1d8a-105">El tráfico fluye entre la red local y Azure Virtual Network (VNet) a través de un túnel VPN de IPSec.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-105">Traffic flows between the on-premises network and an Azure Virtual Network (VNet) through an IPSec VPN tunnel.</span></span> <span data-ttu-id="a1d8a-106">[**Implemente esta solución**](#deploy-the-solution).</span><span class="sxs-lookup"><span data-stu-id="a1d8a-106">[**Deploy this solution**](#deploy-the-solution).</span></span>

![Red híbrida que abarca las infraestructuras de Azure y local](./images/vpn.png)

<span data-ttu-id="a1d8a-108">*Descargue un [archivo Visio][visio-download] de esta arquitectura.*</span><span class="sxs-lookup"><span data-stu-id="a1d8a-108">*Download a [Visio file][visio-download] of this architecture.*</span></span>

## <a name="architecture"></a><span data-ttu-id="a1d8a-109">Arquitectura</span><span class="sxs-lookup"><span data-stu-id="a1d8a-109">Architecture</span></span>

<span data-ttu-id="a1d8a-110">La arquitectura consta de los siguientes componentes:</span><span class="sxs-lookup"><span data-stu-id="a1d8a-110">The architecture consists of the following components.</span></span>

- <span data-ttu-id="a1d8a-111">**Red local**.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-111">**On-premises network**.</span></span> <span data-ttu-id="a1d8a-112">Una red de área local privada que se ejecuta dentro de una organización.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-112">A private local-area network running within an organization.</span></span>

- <span data-ttu-id="a1d8a-113">**Dispositivo VPN**.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-113">**VPN appliance**.</span></span> <span data-ttu-id="a1d8a-114">Un dispositivo o servicio que proporciona conectividad externa a la red local.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-114">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="a1d8a-115">El dispositivo VPN puede ser un dispositivo de hardware, o puede ser una solución de software como el servicio de Enrutamiento y acceso remoto (RRAS) en Windows Server 2012.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-115">The VPN appliance may be a hardware device, or it can be a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="a1d8a-116">Para obtener una lista de dispositivos VPN admitidos e información sobre la configuración para conectarlos a Azure VPN Gateway, consulte las instrucciones para el dispositivo seleccionado en el artículo [Acerca de los dispositivos VPN y los parámetros de IPsec/IKE para conexiones de VPN Gateway de sitio a sitio][vpn-appliance].</span><span class="sxs-lookup"><span data-stu-id="a1d8a-116">For a list of supported VPN appliances and information on configuring them to connect to an Azure VPN gateway, see the instructions for the selected device in the article [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

- <span data-ttu-id="a1d8a-117">**Red virtual**.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-117">**Virtual network (VNet)**.</span></span> <span data-ttu-id="a1d8a-118">La aplicación en la nube y los componentes de Azure VPN Gateway se encuentran en la misma red virtual [(VNet)][azure-virtual-network].</span><span class="sxs-lookup"><span data-stu-id="a1d8a-118">The cloud application and the components for the Azure VPN gateway reside in the same [VNet][azure-virtual-network].</span></span>

- <span data-ttu-id="a1d8a-119">**Azure VPN Gateway**.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-119">**Azure VPN gateway**.</span></span> <span data-ttu-id="a1d8a-120">El servicio [VPN Gateway][azure-vpn-gateway] le permite conectar la red virtual a la red local mediante un dispositivo VPN.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-120">The [VPN gateway][azure-vpn-gateway] service enables you to connect the VNet to the on-premises network through a VPN appliance.</span></span> <span data-ttu-id="a1d8a-121">Para más información, consulte [Conectar una red local con una red virtual de Microsoft Azure][connect-to-an-Azure-vnet].</span><span class="sxs-lookup"><span data-stu-id="a1d8a-121">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span> <span data-ttu-id="a1d8a-122">VPN Gateway incluye los siguientes elementos:</span><span class="sxs-lookup"><span data-stu-id="a1d8a-122">The VPN gateway includes the following elements:</span></span>

  - <span data-ttu-id="a1d8a-123">**Puerta de enlace de red virtual**.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-123">**Virtual network gateway**.</span></span> <span data-ttu-id="a1d8a-124">Recurso que proporciona un dispositivo VPN virtual para la red virtual.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-124">A resource that provides a virtual VPN appliance for the VNet.</span></span> <span data-ttu-id="a1d8a-125">Es responsable de enrutar el tráfico de la red local a la red virtual.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-125">It is responsible for routing traffic from the on-premises network to the VNet.</span></span>
  - <span data-ttu-id="a1d8a-126">**Puerta de enlace de red local**.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-126">**Local network gateway**.</span></span> <span data-ttu-id="a1d8a-127">Abstracción del dispositivo VPN local.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-127">An abstraction of the on-premises VPN appliance.</span></span> <span data-ttu-id="a1d8a-128">El tráfico de red de la aplicación en la nube a la red local se enruta a través de esta puerta de enlace.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-128">Network traffic from the cloud application to the on-premises network is routed through this gateway.</span></span>
  - <span data-ttu-id="a1d8a-129">**Conexión**.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-129">**Connection**.</span></span> <span data-ttu-id="a1d8a-130">La conexión tiene propiedades que especifican el tipo de conexión (IPSec) y la clave compartida con el dispositivo VPN local para cifrar el tráfico.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-130">The connection has properties that specify the connection type (IPSec) and the key shared with the on-premises VPN appliance to encrypt traffic.</span></span>
  - <span data-ttu-id="a1d8a-131">**Subred de puerta de enlace**.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-131">**Gateway subnet**.</span></span> <span data-ttu-id="a1d8a-132">La puerta de enlace de red virtual se mantiene en su propia subred, que está sujeta a los distintos requisitos que se describen a continuación, en la sección Recomendaciones.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-132">The virtual network gateway is held in its own subnet, which is subject to various requirements, described in the Recommendations section below.</span></span>

- <span data-ttu-id="a1d8a-133">**Aplicación en la nube**.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-133">**Cloud application**.</span></span> <span data-ttu-id="a1d8a-134">La aplicación hospedada en Azure.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-134">The application hosted in Azure.</span></span> <span data-ttu-id="a1d8a-135">Puede incluir varios niveles, con varias subredes que se conectan a través de equilibradores de carga de Azure.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-135">It might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="a1d8a-136">Para obtener más información acerca de la infraestructura de aplicaciones, consulte [Ejecutar cargas de trabajo de máquinas virtuales Windows][windows-vm-ra] y [Ejecutar cargas de trabajo de máquinas virtuales Linux][linux-vm-ra].</span><span class="sxs-lookup"><span data-stu-id="a1d8a-136">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

- <span data-ttu-id="a1d8a-137">**Equilibrador de carga interno**.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-137">**Internal load balancer**.</span></span> <span data-ttu-id="a1d8a-138">El tráfico de red de VPN Gateway se enruta a la aplicación en la nube a través de un equilibrador de carga interno.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-138">Network traffic from the VPN gateway is routed to the cloud application through an internal load balancer.</span></span> <span data-ttu-id="a1d8a-139">El equilibrador de carga se encuentra en la subred front-end de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-139">The load balancer is located in the front-end subnet of the application.</span></span>

## <a name="recommendations"></a><span data-ttu-id="a1d8a-140">Recomendaciones</span><span class="sxs-lookup"><span data-stu-id="a1d8a-140">Recommendations</span></span>

<span data-ttu-id="a1d8a-141">Las siguientes recomendaciones sirven para la mayoría de los escenarios.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-141">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="a1d8a-142">Sígalas a menos que tenga un requisito concreto que las invalide.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-142">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="vnet-and-gateway-subnet"></a><span data-ttu-id="a1d8a-143">Red virtual y subred de puerta de enlace</span><span class="sxs-lookup"><span data-stu-id="a1d8a-143">VNet and gateway subnet</span></span>

<span data-ttu-id="a1d8a-144">Cree una red virtual de Azure con un espacio de direcciones lo suficientemente grande para todos los recursos necesarios.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-144">Create an Azure VNet with an address space large enough for all of your required resources.</span></span> <span data-ttu-id="a1d8a-145">Asegúrese de que el espacio de direcciones de la red virtual tiene suficiente espacio para crecer si es probable que se necesiten máquinas virtuales adicionales en el futuro.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-145">Ensure that the VNet address space has sufficient room for growth if additional VMs are likely to be needed in the future.</span></span> <span data-ttu-id="a1d8a-146">El espacio de direcciones de la red virtual no debe superponerse a la red local.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-146">The address space of the VNet must not overlap with the on-premises network.</span></span> <span data-ttu-id="a1d8a-147">Por ejemplo, en el diagrama anterior se usa el espacio de direcciones 10.20.0.0/16 para la red virtual.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-147">For example, the diagram above uses the address space 10.20.0.0/16 for the VNet.</span></span>

<span data-ttu-id="a1d8a-148">Cree una subred denominada *GatewaySubnet*, con un intervalo de direcciones de /27.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-148">Create a subnet named *GatewaySubnet*, with an address range of /27.</span></span> <span data-ttu-id="a1d8a-149">Esta subred es necesaria para la puerta de enlace de red virtual.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-149">This subnet is required by the virtual network gateway.</span></span> <span data-ttu-id="a1d8a-150">La asignación de treinta y dos direcciones a esta subred ayudará a evitar que se alcancen las limitaciones de tamaño de la puerta de enlace en el futuro.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-150">Allocating 32 addresses to this subnet will help to prevent reaching gateway size limitations in the future.</span></span> <span data-ttu-id="a1d8a-151">Evite también colocar esta subred en el centro del espacio de direcciones.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-151">Also, avoid placing this subnet in the middle of the address space.</span></span> <span data-ttu-id="a1d8a-152">Una práctica recomendada consiste en establecer el espacio de direcciones para la subred de la puerta de enlace en el extremo superior del espacio de direcciones de la red virtual.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-152">A good practice is to set the address space for the gateway subnet at the upper end of the VNet address space.</span></span> <span data-ttu-id="a1d8a-153">El ejemplo que aparece en el diagrama usa 10.20.255.224/27.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-153">The example shown in the diagram uses 10.20.255.224/27.</span></span>  <span data-ttu-id="a1d8a-154">Este es un procedimiento rápido para calcular el [CIDR]:</span><span class="sxs-lookup"><span data-stu-id="a1d8a-154">Here is a quick procedure to calculate the [CIDR]:</span></span>

1. <span data-ttu-id="a1d8a-155">Establezca los bits variables del espacio de direcciones de la red virtual en 1 hasta alcanzar los bits que usa la subred de puerta de enlace. A continuación, establezca los bits restantes en 0.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-155">Set the variable bits in the address space of the VNet to 1, up to the bits being used by the gateway subnet, then set the remaining bits to 0.</span></span>
2. <span data-ttu-id="a1d8a-156">Convierta los bits resultantes a decimales y expréselos como un espacio de direcciones con la longitud del prefijo establecida en el tamaño de la subred de puerta de enlace.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-156">Convert the resulting bits to decimal and express it as an address space with the prefix length set to the size of the gateway subnet.</span></span>

<span data-ttu-id="a1d8a-157">Por ejemplo, al aplicar el paso 1 anterior a una red virtual con un intervalo de direcciones IP de 10.20.0.0/16, se convierte en 10.20.0b11111111.0b11100000.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-157">For example, for a VNet with an IP address range of 10.20.0.0/16, applying step #1 above becomes 10.20.0b11111111.0b11100000.</span></span>  <span data-ttu-id="a1d8a-158">Si se convierte en decimales y se expresa como un espacio de direcciones, da como resultado 10.20.255.224/27.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-158">Converting that to decimal and expressing it as an address space yields 10.20.255.224/27.</span></span>

> [!WARNING]
> <span data-ttu-id="a1d8a-159">No implemente ninguna máquina virtual en la subred de puerta de enlace.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-159">Do not deploy any VMs to the gateway subnet.</span></span> <span data-ttu-id="a1d8a-160">Tampoco asigne ningún NSG a esta subred, ya que causaría que la puerta de enlace dejase de funcionar.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-160">Also, do not assign an NSG to this subnet, as it will cause the gateway to stop functioning.</span></span>
>

### <a name="virtual-network-gateway"></a><span data-ttu-id="a1d8a-161">Puerta de enlace de red virtual</span><span class="sxs-lookup"><span data-stu-id="a1d8a-161">Virtual network gateway</span></span>

<span data-ttu-id="a1d8a-162">Asigne una dirección IP pública para la puerta de enlace de la red virtual.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-162">Allocate a public IP address for the virtual network gateway.</span></span>

<span data-ttu-id="a1d8a-163">Cree la puerta de enlace de red virtual en la subred de puerta de enlace y asígnele la dirección IP pública recién asignada.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-163">Create the virtual network gateway in the gateway subnet and assign it the newly allocated public IP address.</span></span> <span data-ttu-id="a1d8a-164">Use el tipo de puerta de enlace que mejor se ajuste a sus requisitos y que su dispositivo VPN haya habilitado:</span><span class="sxs-lookup"><span data-stu-id="a1d8a-164">Use the gateway type that most closely matches your requirements and that is enabled by your VPN appliance:</span></span>

- <span data-ttu-id="a1d8a-165">Cree una [puerta de enlace basada en directivas][policy-based-routing] si necesita controlar estrechamente cómo se enrutan las solicitudes en función de los criterios de las directivas, tales como los prefijos de las direcciones.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-165">Create a [policy-based gateway][policy-based-routing] if you need to closely control how requests are routed based on policy criteria such as address prefixes.</span></span> <span data-ttu-id="a1d8a-166">Las puertas de enlace basadas en directivas utilizan el enrutamiento estático y solo funcionan con conexiones de sitio a sitio.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-166">Policy-based gateways use static routing, and only work with site-to-site connections.</span></span>

- <span data-ttu-id="a1d8a-167">Cree una [puerta de enlace basada en rutas][route-based-routing] si se conecta a la red local mediante RRAS, admite conexiones multisitio o entre regiones, o bien si implementa conexiones de red virtual a red virtual (incluidas las rutas que atraviesan varias redes virtuales).</span><span class="sxs-lookup"><span data-stu-id="a1d8a-167">Create a [route-based gateway][route-based-routing] if you connect to the on-premises network using RRAS, support multi-site or cross-region connections, or implement VNet-to-VNet connections (including routes that traverse multiple VNets).</span></span> <span data-ttu-id="a1d8a-168">Las puertas de enlace basadas en rutas utilizan el enrutamiento dinámico para dirigir el tráfico entre redes.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-168">Route-based gateways use dynamic routing to direct traffic between networks.</span></span> <span data-ttu-id="a1d8a-169">Dado que pueden intentar rutas alternativas, pueden tolerar mejor los errores en la ruta de acceso de red que las rutas estáticas.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-169">They can tolerate failures in the network path better than static routes because they can try alternative routes.</span></span> <span data-ttu-id="a1d8a-170">Las puertas de enlace basadas en rutas también pueden reducir la sobrecarga de administración porque es posible que las rutas no tengan que actualizarse manualmente cuando las direcciones de red cambien.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-170">Route-based gateways can also reduce the management overhead because routes might not need to be updated manually when network addresses change.</span></span>

<span data-ttu-id="a1d8a-171">Para obtener una lista de dispositivos VPN compatibles, consulte [Acerca de los dispositivos VPN y los parámetros de IPsec/IKE para conexiones de VPN Gateway de sitio a sitio][vpn-appliances].</span><span class="sxs-lookup"><span data-stu-id="a1d8a-171">For a list of supported VPN appliances, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliances].</span></span>

> [!NOTE]
> <span data-ttu-id="a1d8a-172">Tras crear la puerta de enlace, no se puede cambiar entre los tipos de puerta de enlace sin antes eliminarla y volverla a crear.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-172">After the gateway has been created, you cannot change between gateway types without deleting and re-creating the gateway.</span></span>
>

<span data-ttu-id="a1d8a-173">Seleccione la SKU de Azure VPN Gateway que mejor se ajuste a sus requisitos de rendimiento.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-173">Select the Azure VPN gateway SKU that most closely matches your throughput requirements.</span></span> <span data-ttu-id="a1d8a-174">Consulte [SKU de puertas de enlace][azure-gateway-skus] para más información</span><span class="sxs-lookup"><span data-stu-id="a1d8a-174">For more informayion, see [Gateway SKUs][azure-gateway-skus]</span></span>

> [!NOTE]
> <span data-ttu-id="a1d8a-175">La SKU Básica no es compatible con Azure ExpressRoute.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-175">The Basic SKU is not compatible with Azure ExpressRoute.</span></span> <span data-ttu-id="a1d8a-176">También puede [cambiar la SKU][changing-SKUs] tras crear la puerta de enlace.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-176">You can [change the SKU][changing-SKUs] after the gateway has been created.</span></span>
>

<span data-ttu-id="a1d8a-177">Se le aplicará un cargo dependiendo del tiempo de aprovisionamiento y de la disponibilidad de la puerta de enlace.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-177">You are charged based on the amount of time that the gateway is provisioned and available.</span></span> <span data-ttu-id="a1d8a-178">Vea [Precios de VPN Gateway][azure-gateway-charges].</span><span class="sxs-lookup"><span data-stu-id="a1d8a-178">See [VPN Gateway Pricing][azure-gateway-charges].</span></span>

<span data-ttu-id="a1d8a-179">En lugar de permitir que las solicitudes pasen directamente a las máquinas virtuales de la aplicación, cree reglas de enrutamiento para la subred de puerta de enlace que dirige el tráfico de aplicaciones entrante desde la puerta de enlace hacia el equilibrador de carga interno.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-179">Create routing rules for the gateway subnet that direct incoming application traffic from the gateway to the internal load balancer, rather than allowing requests to pass directly to the application VMs.</span></span>

### <a name="on-premises-network-connection"></a><span data-ttu-id="a1d8a-180">Conexión de red local</span><span class="sxs-lookup"><span data-stu-id="a1d8a-180">On-premises network connection</span></span>

<span data-ttu-id="a1d8a-181">Cree una puerta de enlace de red local.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-181">Create a local network gateway.</span></span> <span data-ttu-id="a1d8a-182">Especifique la dirección IP pública del dispositivo VPN local y el espacio de direcciones de la red local.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-182">Specify the public IP address of the on-premises VPN appliance, and the address space of the on-premises network.</span></span> <span data-ttu-id="a1d8a-183">Tenga en cuenta que el dispositivo VPN local debe tener una dirección IP pública a la cual pueda tener acceso la puerta de enlace de red local de Azure VPN Gateway.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-183">Note that the on-premises VPN appliance must have a public IP address that can be accessed by the local network gateway in Azure VPN Gateway.</span></span> <span data-ttu-id="a1d8a-184">El dispositivo VPN no puede ubicarse detrás de un dispositivo de traducción de direcciones de red (NAT).</span><span class="sxs-lookup"><span data-stu-id="a1d8a-184">The VPN device cannot be located behind a network address translation (NAT) device.</span></span>

<span data-ttu-id="a1d8a-185">Cree una conexión de sitio a sitio para la puerta de enlace de red virtual y la puerta de enlace de red local.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-185">Create a site-to-site connection for the virtual network gateway and the local network gateway.</span></span> <span data-ttu-id="a1d8a-186">Seleccione el tipo de conexión de sitio a sitio (IPSec) y especifique la clave compartida.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-186">Select the site-to-site (IPSec) connection type, and specify the shared key.</span></span> <span data-ttu-id="a1d8a-187">El cifrado de sitio a sitio con Azure VPN Gateway se basa en el protocolo IPSec y utiliza claves compartidas previamente para la autenticación.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-187">Site-to-site encryption with the Azure VPN gateway is based on the IPSec protocol, using preshared keys for authentication.</span></span> <span data-ttu-id="a1d8a-188">La clave se especifica al crear la instancia de Azure VPN Gateway.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-188">You specify the key when you create the Azure VPN gateway.</span></span> <span data-ttu-id="a1d8a-189">Debe configurar el dispositivo VPN que se ejecuta localmente con la misma clave.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-189">You must configure the VPN appliance running on-premises with the same key.</span></span> <span data-ttu-id="a1d8a-190">Actualmente, no se admiten otros mecanismos de autenticación.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-190">Other authentication mechanisms are not currently supported.</span></span>

<span data-ttu-id="a1d8a-191">Asegúrese de que la infraestructura de enrutamiento local esté configurada para reenviar al dispositivo VPN las solicitudes destinadas a direcciones de la red virtual de Azure.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-191">Ensure that the on-premises routing infrastructure is configured to forward requests intended for addresses in the Azure VNet to the VPN device.</span></span>

<span data-ttu-id="a1d8a-192">Abra los puertos de la red local que requiera la aplicación en la nube.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-192">Open any ports required by the cloud application in the on-premises network.</span></span>

<span data-ttu-id="a1d8a-193">Pruebe la conexión para comprobar que:</span><span class="sxs-lookup"><span data-stu-id="a1d8a-193">Test the connection to verify that:</span></span>

- <span data-ttu-id="a1d8a-194">El dispositivo VPN local enruta correctamente el tráfico a la aplicación en la nube a través de Azure VPN Gateway.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-194">The on-premises VPN appliance correctly routes traffic to the cloud application through the Azure VPN gateway.</span></span>
- <span data-ttu-id="a1d8a-195">La red virtual enruta correctamente el tráfico a la red local.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-195">The VNet correctly routes traffic back to the on-premises network.</span></span>
- <span data-ttu-id="a1d8a-196">El tráfico prohibido en ambas direcciones está bloqueado correctamente.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-196">Prohibited traffic in both directions is blocked correctly.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="a1d8a-197">Consideraciones sobre escalabilidad</span><span class="sxs-lookup"><span data-stu-id="a1d8a-197">Scalability considerations</span></span>

<span data-ttu-id="a1d8a-198">Puede lograr una escalabilidad vertical limitada si cambia de las SKU de VPN Gateway Estándar o Básica a la SKU Alto rendimiento de VPN.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-198">You can achieve limited vertical scalability by moving from the Basic or Standard VPN Gateway SKUs to the High Performance VPN SKU.</span></span>

<span data-ttu-id="a1d8a-199">Para las redes virtuales que esperan un gran volumen de tráfico VPN, considere la posibilidad de distribuir las diferentes cargas de trabajo en redes virtuales más pequeñas independientes y de configurar una puerta de enlace VPN para cada una de ellas.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-199">For VNets that expect a large volume of VPN traffic, consider distributing the different workloads into separate smaller VNets and configuring a VPN gateway for each of them.</span></span>

<span data-ttu-id="a1d8a-200">Puede dividir la red virtual de forma horizontal o vertical.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-200">You can partition the VNet either horizontally or vertically.</span></span> <span data-ttu-id="a1d8a-201">Para dividirla horizontalmente, mueva algunas instancias de máquina virtual de cada capa a subredes de la red virtual nueva.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-201">To partition horizontally, move some VM instances from each tier into subnets of the new VNet.</span></span> <span data-ttu-id="a1d8a-202">El resultado es que cada red virtual tiene la misma estructura y funcionalidad.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-202">The result is that each VNet has the same structure and functionality.</span></span> <span data-ttu-id="a1d8a-203">Para dividirla verticalmente, vuelva a diseñar cada capa para dividir la funcionalidad en diferentes áreas lógicas (por ejemplo, gestión de pedidos, facturación, administración de cuentas de clientes, etc.).</span><span class="sxs-lookup"><span data-stu-id="a1d8a-203">To partition vertically, redesign each tier to divide the functionality into different logical areas (such as handling orders, invoicing, customer account management, and so on).</span></span> <span data-ttu-id="a1d8a-204">Cada área funcional se puede colocar en su propia red virtual.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-204">Each functional area can then be placed in its own VNet.</span></span>

<span data-ttu-id="a1d8a-205">Con la replicación de un controlador de dominio de Active Directory local en la red virtual y la implementación de DNS en la red virtual, puede ayudar a reducir tráfico relacionado con la seguridad y la administración que fluye de la red local a la nube.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-205">Replicating an on-premises Active Directory domain controller in the VNet, and implementing DNS in the VNet, can help to reduce some of the security-related and administrative traffic flowing from on-premises to the cloud.</span></span> <span data-ttu-id="a1d8a-206">Para obtener más información, consulte [Extensión de Active Directory Domain Services (AD DS) a Azure][adds-extend-domain].</span><span class="sxs-lookup"><span data-stu-id="a1d8a-206">For more information, see [Extending Active Directory Domain Services (AD DS) to Azure][adds-extend-domain].</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="a1d8a-207">Consideraciones sobre disponibilidad</span><span class="sxs-lookup"><span data-stu-id="a1d8a-207">Availability considerations</span></span>

<span data-ttu-id="a1d8a-208">Si necesita asegurarse de que la red local sigue estando disponible en Azure VPN Gateway, implemente un clúster de conmutación por error para la instancia de VPN Gateway local.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-208">If you need to ensure that the on-premises network remains available to the Azure VPN gateway, implement a failover cluster for the on-premises VPN gateway.</span></span>

<span data-ttu-id="a1d8a-209">Si su organización tiene varios sitios locales, cree [conexiones multisitio][vpn-gateway-multi-site] a una o varias redes virtuales de Azure.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-209">If your organization has multiple on-premises sites, create [multi-site connections][vpn-gateway-multi-site] to one or more Azure VNets.</span></span> <span data-ttu-id="a1d8a-210">Este enfoque requiere enrutamiento dinámico (basado en rutas), así que debe asegurarse de que la instancia de VPN Gateway local admite esta característica.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-210">This approach requires dynamic (route-based) routing, so make sure that the on-premises VPN gateway supports this feature.</span></span>

<span data-ttu-id="a1d8a-211">Para obtener más información acerca de los acuerdos de nivel de servicio, consulte [Contrato de nivel de servicio para VPN Gateway][sla-for-vpn-gateway].</span><span class="sxs-lookup"><span data-stu-id="a1d8a-211">For details about service level agreements, see [SLA for VPN Gateway][sla-for-vpn-gateway].</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="a1d8a-212">Consideraciones sobre la manejabilidad</span><span class="sxs-lookup"><span data-stu-id="a1d8a-212">Manageability considerations</span></span>

<span data-ttu-id="a1d8a-213">Supervise la información de diagnóstico de los dispositivos VPN locales.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-213">Monitor diagnostic information from on-premises VPN appliances.</span></span> <span data-ttu-id="a1d8a-214">Este proceso depende de las características que proporcione el dispositivo VPN.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-214">This process depends on the features provided by the VPN appliance.</span></span> <span data-ttu-id="a1d8a-215">Por ejemplo, si usa el servicio de Enrutamiento y acceso remoto en Windows Server 2012, consulte [RRAS logging][rras-logging] (Registro de RRAS).</span><span class="sxs-lookup"><span data-stu-id="a1d8a-215">For example, if you are using the Routing and Remote Access Service on Windows Server 2012, [RRAS logging][rras-logging].</span></span>

<span data-ttu-id="a1d8a-216">Use los [diagnósticos de Azure VPN Gateway][ gateway-diagnostic-logs] para capturar información sobre problemas de conectividad.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-216">Use [Azure VPN gateway diagnostics][gateway-diagnostic-logs] to capture information about connectivity issues.</span></span> <span data-ttu-id="a1d8a-217">Estos registros se pueden utilizar para realizar el seguimiento de información, como el origen y los destinos de las solicitudes de conexión, el protocolo que se utilizó y cómo se estableció la conexión (o por qué el intento fue erróneo).</span><span class="sxs-lookup"><span data-stu-id="a1d8a-217">These logs can be used to track information such as the source and destinations of connection requests, which protocol was used, and how the connection was established (or why the attempt failed).</span></span>

<span data-ttu-id="a1d8a-218">Supervise los registros operativos de Azure VPN Gateway mediante los registros de auditoría disponibles en Azure Portal.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-218">Monitor the operational logs of the Azure VPN gateway using the audit logs available in the Azure portal.</span></span> <span data-ttu-id="a1d8a-219">Hay disponibles registros independientes para la puerta de enlace de red local, la puerta de enlace de red de Azure y la conexión.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-219">Separate logs are available for the local network gateway, the Azure network gateway, and the connection.</span></span> <span data-ttu-id="a1d8a-220">Esta información puede utilizarse para realizar un seguimiento de los cambios realizados en la puerta de enlace y puede resultar útil si una puerta de enlace que funcionaba previamente dejó de hacerlo por algún motivo.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-220">This information can be used to track any changes made to the gateway, and can be useful if a previously functioning gateway stops working for some reason.</span></span>

![Registros de auditoría en Azure Portal](../_images/guidance-hybrid-network-vpn/audit-logs.png)

<span data-ttu-id="a1d8a-222">Supervise la conectividad y realice un seguimiento de los eventos de error de conectividad.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-222">Monitor connectivity, and track connectivity failure events.</span></span> <span data-ttu-id="a1d8a-223">Puede usar un paquete de supervisión como [Nagios][nagios] para capturar y notificar esta información.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-223">You can use a monitoring package such as [Nagios][nagios] to capture and report this information.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="a1d8a-224">Consideraciones sobre la seguridad</span><span class="sxs-lookup"><span data-stu-id="a1d8a-224">Security considerations</span></span>

<span data-ttu-id="a1d8a-225">Genere una clave compartida diferente para cada instancia de VPN Gateway.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-225">Generate a different shared key for each VPN gateway.</span></span> <span data-ttu-id="a1d8a-226">Use una clave compartida segura para resistir los ataques de fuerza bruta.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-226">Use a strong shared key to help resist brute-force attacks.</span></span>

> [!NOTE]
> <span data-ttu-id="a1d8a-227">Actualmente, no se puede usar Azure Key Vault para la compartición de claves previa en Azure VPN Gateway.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-227">Currently, you cannot use Azure Key Vault to preshare keys for the Azure VPN gateway.</span></span>
>

<span data-ttu-id="a1d8a-228">Asegúrese de que el dispositivo VPN local use un método de cifrado que sea [compatible con Azure VPN Gateway][vpn-appliance-ipsec].</span><span class="sxs-lookup"><span data-stu-id="a1d8a-228">Ensure that the on-premises VPN appliance uses an encryption method that is [compatible with the Azure VPN gateway][vpn-appliance-ipsec].</span></span> <span data-ttu-id="a1d8a-229">Para el enrutamiento basado en directivas, Azure VPN Gateway admite los algoritmos de cifrado AES256, AES128 y 3DES.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-229">For policy-based routing, the Azure VPN gateway supports the AES256, AES128, and 3DES encryption algorithms.</span></span> <span data-ttu-id="a1d8a-230">Las puertas de enlace basadas en rutas admiten AES256 y 3DES.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-230">Route-based gateways support AES256 and 3DES.</span></span>

<span data-ttu-id="a1d8a-231">Si su dispositivo VPN local se encuentra en una red perimetral (DMZ) que tiene un firewall entre la red perimetral e Internet, es posible que deba configurar [reglas de firewall adicionales][additional-firewall-rules] para permitir la conexión VPN de sitio a sitio.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-231">If your on-premises VPN appliance is on a perimeter network (DMZ) that has a firewall between the perimeter network and the Internet, you might have to configure [additional firewall rules][additional-firewall-rules] to allow the site-to-site VPN connection.</span></span>

<span data-ttu-id="a1d8a-232">Si la aplicación en la red virtual envía datos a Internet, considere la posibilidad de [implementar la tunelización forzada][forced-tunneling] para enrutar todo el tráfico vinculado a Internet a través de la red local.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-232">If the application in the VNet sends data to the Internet, consider [implementing forced tunneling][forced-tunneling] to route all Internet-bound traffic through the on-premises network.</span></span> <span data-ttu-id="a1d8a-233">Este enfoque le permite auditar las solicitudes salientes que realiza la aplicación desde la infraestructura local.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-233">This approach enables you to audit outgoing requests made by the application from the on-premises infrastructure.</span></span>

> [!NOTE]
> <span data-ttu-id="a1d8a-234">La tunelización forzada puede afectar a la conectividad a los servicios de Azure (por ejemplo, el servicio de almacenamiento) y el administrador de licencias de Windows.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-234">Forced tunneling can impact connectivity to Azure services (the Storage Service, for example) and the Windows license manager.</span></span>
>

## <a name="troubleshooting"></a><span data-ttu-id="a1d8a-235">solución de problemas</span><span class="sxs-lookup"><span data-stu-id="a1d8a-235">Troubleshooting</span></span>

<span data-ttu-id="a1d8a-236">Para obtener información general sobre cómo solucionar errores comunes relacionados con la VPN, consulte [Troubleshooting common VPN related errors][troubleshooting-vpn-errors] (Solución de errores comunes relacionados con la VPN).</span><span class="sxs-lookup"><span data-stu-id="a1d8a-236">For general information on troubleshooting common VPN-related errors, see [Troubleshooting common VPN related errors][troubleshooting-vpn-errors].</span></span>

<span data-ttu-id="a1d8a-237">Las recomendaciones siguientes son útiles para determinar si su dispositivo VPN local funciona correctamente.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-237">The following recommendations are useful for determining if your on-premises VPN appliance is functioning correctly.</span></span>

- <span data-ttu-id="a1d8a-238">**Compruebe si hay errores en cualquier archivo de registro que generó el dispositivo VPN.**</span><span class="sxs-lookup"><span data-stu-id="a1d8a-238">**Check any log files generated by the VPN appliance for errors or failures.**</span></span>

    <span data-ttu-id="a1d8a-239">Esto le ayudará a determinar si el dispositivo VPN funciona correctamente.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-239">This will help you determine if the VPN appliance is functioning correctly.</span></span> <span data-ttu-id="a1d8a-240">La ubicación de esta información variará según el dispositivo.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-240">The location of this information will vary according to your appliance.</span></span> <span data-ttu-id="a1d8a-241">Por ejemplo, si está usando RRAS en Windows Server 2012, puede usar el siguiente comando de PowerShell para mostrar información de los eventos de error para el servicio RRAS:</span><span class="sxs-lookup"><span data-stu-id="a1d8a-241">For example, if you are using RRAS on Windows Server 2012, you can use the following PowerShell command to display error event information for the RRAS service:</span></span>

    ```PowerShell
    Get-EventLog -LogName System -EntryType Error -Source RemoteAccess | Format-List -Property *
    ```

    <span data-ttu-id="a1d8a-242">La propiedad *Message* de cada entrada proporciona una descripción del error.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-242">The *Message* property of each entry provides a description of the error.</span></span> <span data-ttu-id="a1d8a-243">Algunos ejemplos comunes:</span><span class="sxs-lookup"><span data-stu-id="a1d8a-243">Some common examples are:</span></span>

        - Inability to connect, possibly due to an incorrect IP address specified for the Azure VPN gateway in the RRAS VPN network interface configuration.

        ```console
        EventID            : 20111
        MachineName        : on-prem-vm
        Data               : {41, 3, 0, 0}
        Index              : 14231
        Category           : (0)
        CategoryNumber     : 0
        EntryType          : Error
        Message            : RoutingDomainID- {00000000-0000-0000-0000-000000000000}: A demand dial connection to the remote
                             interface AzureGateway on port VPN2-4 was successfully initiated but failed to complete
                             successfully because of the  following error: The network connection between your computer and
                             the VPN server could not be established because the remote server is not responding. This could
                             be because one of the network devices (for example, firewalls, NAT, routers, and so on) between your computer
                             and the remote server is not configured to allow VPN connections. Please contact your
                             Administrator or your service provider to determine which device may be causing the problem.
        Source             : RemoteAccess
        ReplacementStrings : {{00000000-0000-0000-0000-000000000000}, AzureGateway, VPN2-4, The network connection between
                             your computer and the VPN server could not be established because the remote server is not
                             responding. This could be because one of the network devices (for example, firewalls, NAT, routers, and so on)
                             between your computer and the remote server is not configured to allow VPN connections. Please
                             contact your Administrator or your service provider to determine which device may be causing the
                             problem.}
        InstanceId         : 20111
        TimeGenerated      : 3/18/2016 1:26:02 PM
        TimeWritten        : 3/18/2016 1:26:02 PM
        UserName           :
        Site               :
        Container          :
        ```

        - The wrong shared key being specified in the RRAS VPN network interface configuration.

        ```console
        EventID            : 20111
        MachineName        : on-prem-vm
        Data               : {233, 53, 0, 0}
        Index              : 14245
        Category           : (0)
        CategoryNumber     : 0
        EntryType          : Error
        Message            : RoutingDomainID- {00000000-0000-0000-0000-000000000000}: A demand dial connection to the remote
                             interface AzureGateway on port VPN2-4 was successfully initiated but failed to complete
                             successfully because of the  following error: Internet key exchange (IKE) authentication credentials are unacceptable.

        Source             : RemoteAccess
        ReplacementStrings : {{00000000-0000-0000-0000-000000000000}, AzureGateway, VPN2-4, IKE authentication credentials are
                             unacceptable.
                             }
        InstanceId         : 20111
        TimeGenerated      : 3/18/2016 1:34:22 PM
        TimeWritten        : 3/18/2016 1:34:22 PM
        UserName           :
        Site               :
        Container          :
        ```

    <span data-ttu-id="a1d8a-244">También puede obtener información de registro de eventos sobre los intentos de conexión a través del servicio RRAS mediante el siguiente comando de PowerShell:</span><span class="sxs-lookup"><span data-stu-id="a1d8a-244">You can also obtain event log information about attempts to connect through the RRAS service using the following PowerShell command:</span></span>

    ```powershell
    Get-EventLog -LogName Application -Source RasClient | Format-List -Property *
    ```

    <span data-ttu-id="a1d8a-245">Si se produce un error en la conexión, este registro contendrá errores con un aspecto similar al siguiente:</span><span class="sxs-lookup"><span data-stu-id="a1d8a-245">In the event of a failure to connect, this log will contain errors that look similar to the following:</span></span>

    ```console
    EventID            : 20227
    MachineName        : on-prem-vm
    Data               : {}
    Index              : 4203
    Category           : (0)
    CategoryNumber     : 0
    EntryType          : Error
    Message            : CoId={B4000371-A67F-452F-AA4C-3125AA9CFC78}: The user SYSTEM dialed a connection named
                         AzureGateway that has failed. The error code returned on failure is 809.
    Source             : RasClient
    ReplacementStrings : {{B4000371-A67F-452F-AA4C-3125AA9CFC78}, SYSTEM, AzureGateway, 809}
    InstanceId         : 20227
    TimeGenerated      : 3/18/2016 1:29:21 PM
    TimeWritten        : 3/18/2016 1:29:21 PM
    UserName           :
    Site               :
    Container          :
    ```

- <span data-ttu-id="a1d8a-246">**Compruebe la conectividad y el enrutamiento en VPN Gateway.**</span><span class="sxs-lookup"><span data-stu-id="a1d8a-246">**Verify connectivity and routing across the VPN gateway.**</span></span>

    <span data-ttu-id="a1d8a-247">Es posible que el dispositivo VPN no enrute correctamente el tráfico a través de Azure VPN Gateway.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-247">The VPN appliance may not be correctly routing traffic through the Azure VPN Gateway.</span></span> <span data-ttu-id="a1d8a-248">Use una herramienta como [PsPing][psping] para comprobar la conectividad y el enrutamiento en VPN Gateway.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-248">Use a tool such as [PsPing][psping] to verify connectivity and routing across the VPN gateway.</span></span> <span data-ttu-id="a1d8a-249">Por ejemplo, para probar la conectividad de una máquina local a un servidor web que se encuentre en la red virtual, ejecute el siguiente comando (reemplazando `<<web-server-address>>` por la dirección del servidor web):</span><span class="sxs-lookup"><span data-stu-id="a1d8a-249">For example, to test connectivity from an on-premises machine to a web server located on the VNet, run the following command (replacing `<<web-server-address>>` with the address of the web server):</span></span>

    ```console
    PsPing -t <<web-server-address>>:80
    ```

    <span data-ttu-id="a1d8a-250">Si la máquina local puede enrutar el tráfico al servidor web, debería ver un resultado similar al siguiente:</span><span class="sxs-lookup"><span data-stu-id="a1d8a-250">If the on-premises machine can route traffic to the web server, you should see output similar to the following:</span></span>

    ```console
    D:\PSTools>psping -t 10.20.0.5:80

    PsPing v2.01 - PsPing - ping, latency, bandwidth measurement utility
    Copyright (C) 2012-2014 Mark Russinovich
    Sysinternals - www.sysinternals.com

    TCP connect to 10.20.0.5:80:
    Infinite iterations (warmup 1) connecting test:
    Connecting to 10.20.0.5:80 (warmup): 6.21ms
    Connecting to 10.20.0.5:80: 3.79ms
    Connecting to 10.20.0.5:80: 3.44ms
    Connecting to 10.20.0.5:80: 4.81ms

      Sent = 3, Received = 3, Lost = 0 (0% loss),
      Minimum = 3.44ms, Maximum = 4.81ms, Average = 4.01ms
    ```

    <span data-ttu-id="a1d8a-251">Si la máquina local no puede comunicarse con el destino especificado, verá mensajes similares al siguiente:</span><span class="sxs-lookup"><span data-stu-id="a1d8a-251">If the on-premises machine cannot communicate with the specified destination, you will see messages like this:</span></span>

    ```console
    D:\PSTools>psping -t 10.20.1.6:80

    PsPing v2.01 - PsPing - ping, latency, bandwidth measurement utility
    Copyright (C) 2012-2014 Mark Russinovich
    Sysinternals - www.sysinternals.com

    TCP connect to 10.20.1.6:80:
    Infinite iterations (warmup 1) connecting test:
    Connecting to 10.20.1.6:80 (warmup): This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80: This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80: This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80: This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80:
      Sent = 3, Received = 0, Lost = 3 (100% loss),
      Minimum = 0.00ms, Maximum = 0.00ms, Average = 0.00ms
    ```

- <span data-ttu-id="a1d8a-252">**Compruebe que el firewall local permita que pase el tráfico VPN y que estén abiertos los puertos correctos.**</span><span class="sxs-lookup"><span data-stu-id="a1d8a-252">**Verify that the on-premises firewall allows VPN traffic to pass and that the correct ports are opened.**</span></span>

- <span data-ttu-id="a1d8a-253">**Compruebe que el dispositivo VPN local use un método de cifrado que sea [compatible con Azure VPN Gateway][vpn-appliance].**</span><span class="sxs-lookup"><span data-stu-id="a1d8a-253">**Verify that the on-premises VPN appliance uses an encryption method that is [compatible with the Azure VPN gateway][vpn-appliance].**</span></span> <span data-ttu-id="a1d8a-254">Para el enrutamiento basado en directivas, Azure VPN Gateway admite los algoritmos de cifrado AES256, AES128 y 3DES.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-254">For policy-based routing, the Azure VPN gateway supports the AES256, AES128, and 3DES encryption algorithms.</span></span> <span data-ttu-id="a1d8a-255">Las puertas de enlace basadas en rutas admiten AES256 y 3DES.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-255">Route-based gateways support AES256 and 3DES.</span></span>

<span data-ttu-id="a1d8a-256">Las recomendaciones siguientes son útiles para determinar si hay algún problema con Azure VPN Gateway:</span><span class="sxs-lookup"><span data-stu-id="a1d8a-256">The following recommendations are useful for determining if there is a problem with the Azure VPN gateway:</span></span>

- <span data-ttu-id="a1d8a-257">**Examine los [registros de diagnóstico de Azure VPN Gateway][gateway-diagnostic-logs] para identificar posibles problemas.**</span><span class="sxs-lookup"><span data-stu-id="a1d8a-257">**Examine [Azure VPN gateway diagnostic logs][gateway-diagnostic-logs] for potential issues.**</span></span>

- <span data-ttu-id="a1d8a-258">**Compruebe que Azure VPN Gateway y el dispositivo VPN local estén configurados con la misma clave de autenticación compartida.**</span><span class="sxs-lookup"><span data-stu-id="a1d8a-258">**Verify that the Azure VPN gateway and on-premises VPN appliance are configured with the same shared authentication key.**</span></span>

    <span data-ttu-id="a1d8a-259">Puede ver la clave compartida almacenada en Azure VPN Gateway mediante el siguiente comando de la CLI de Azure:</span><span class="sxs-lookup"><span data-stu-id="a1d8a-259">You can view the shared key stored by the Azure VPN gateway using the following Azure CLI command:</span></span>

    ```azurecli
    azure network vpn-connection shared-key show <<resource-group>> <<vpn-connection-name>>
    ```

    <span data-ttu-id="a1d8a-260">Use el comando adecuado para que su dispositivo VPN local muestre la clave compartida que se configuró.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-260">Use the command appropriate for your on-premises VPN appliance to show the shared key configured for that appliance.</span></span>

    <span data-ttu-id="a1d8a-261">Compruebe que la subred *GatewaySubnet* que contiene la Azure VPN Gateway no esté asociada con un NSG.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-261">Verify that the *GatewaySubnet* subnet holding the Azure VPN gateway is not associated with an NSG.</span></span>

    <span data-ttu-id="a1d8a-262">Puede ver los detalles de la subred con el siguiente comando de la CLI de Azure:</span><span class="sxs-lookup"><span data-stu-id="a1d8a-262">You can view the subnet details using the following Azure CLI command:</span></span>

    ```azurecli
    azure network vnet subnet show -g <<resource-group>> -e <<vnet-name>> -n GatewaySubnet
    ```

    <span data-ttu-id="a1d8a-263">Asegúrese de que no hay ningún campo de datos denominado *Network Security Group id*. En el ejemplo siguiente se muestran los resultados de una instancia de *GatewaySubnet* que tiene un NSG asignado (*VPN-Gateway-Group*).</span><span class="sxs-lookup"><span data-stu-id="a1d8a-263">Ensure there is no data field named *Network Security Group id*. The following example shows the results for an instance of the *GatewaySubnet* that has an assigned NSG (*VPN-Gateway-Group*).</span></span> <span data-ttu-id="a1d8a-264">Esto puede impedir que la puerta de enlace funcione correctamente si no hay ninguna regla definida para este NSG.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-264">This can prevent the gateway from working correctly if there are any rules defined for this NSG.</span></span>

    ```console
    C:\>azure network vnet subnet show -g profx-prod-rg -e profx-vnet -n GatewaySubnet
        info:    Executing command network vnet subnet show
        + Looking up virtual network "profx-vnet"
        + Looking up the subnet "GatewaySubnet"
        data:    Id                              : /subscriptions/########-####-####-####-############/resourceGroups/profx-prod-rg/providers/Microsoft.Network/virtualNetworks/profx-vnet/subnets/GatewaySubnet
        data:    Name                            : GatewaySubnet
        data:    Provisioning state              : Succeeded
        data:    Address prefix                  : 10.20.3.0/27
        data:    Network Security Group id       : /subscriptions/########-####-####-####-############/resourceGroups/profx-prod-rg/providers/Microsoft.Network/networkSecurityGroups/VPN-Gateway-Group
        info:    network vnet subnet show command OK
    ```

- <span data-ttu-id="a1d8a-265">**Compruebe que las máquinas virtuales en la red virtual de Azure estén configuradas para permitir el tráfico procedente de fuera de la red virtual.**</span><span class="sxs-lookup"><span data-stu-id="a1d8a-265">**Verify that the virtual machines in the Azure VNet are configured to permit traffic coming in from outside the VNet.**</span></span>

    <span data-ttu-id="a1d8a-266">Compruebe si hay alguna regla de NSG asociada con las subredes que contienen estas máquinas virtuales.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-266">Check any NSG rules associated with subnets containing these virtual machines.</span></span> <span data-ttu-id="a1d8a-267">Puede ver todas las reglas de NSG con el siguiente comando de la CLI de Azure:</span><span class="sxs-lookup"><span data-stu-id="a1d8a-267">You can view all NSG rules using the following Azure CLI command:</span></span>

    ```azurecli
    azure network nsg show -g <<resource-group>> -n <<nsg-name>>
    ```

- <span data-ttu-id="a1d8a-268">**Compruebe que Azure VPN Gateway esté conectado.**</span><span class="sxs-lookup"><span data-stu-id="a1d8a-268">**Verify that the Azure VPN gateway is connected.**</span></span>

    <span data-ttu-id="a1d8a-269">Puede usar el siguiente comando de Azure PowerShell para comprobar el estado actual de la conexión VPN de Azure.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-269">You can use the following Azure PowerShell command to check the current status of the Azure VPN connection.</span></span> <span data-ttu-id="a1d8a-270">El parámetro `<<connection-name>>` es el nombre de la conexión VPN de Azure que vincula la puerta de enlace de red virtual y la puerta de enlace local.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-270">The `<<connection-name>>` parameter is the name of the Azure VPN connection that links the virtual network gateway and the local gateway.</span></span>

    ```powershell
    Get-AzureRmVirtualNetworkGatewayConnection -Name <<connection-name>> - ResourceGroupName <<resource-group>>
    ```

    <span data-ttu-id="a1d8a-271">Los fragmentos de código siguientes resaltan la salida que se genera si la puerta de enlace está conectada (primer ejemplo) o desconectada (segundo ejemplo):</span><span class="sxs-lookup"><span data-stu-id="a1d8a-271">The following snippets highlight the output generated if the gateway is connected (the first example), and disconnected (the second example):</span></span>

    ```powershell
    PS C:\> Get-AzureRmVirtualNetworkGatewayConnection -Name profx-gateway-connection -ResourceGroupName profx-prod-rg

    AuthorizationKey           :
    VirtualNetworkGateway1     : Microsoft.Azure.Commands.Network.Models.PSVirtualNetworkGateway
    VirtualNetworkGateway2     :
    LocalNetworkGateway2       : Microsoft.Azure.Commands.Network.Models.PSLocalNetworkGateway
    Peer                       :
    ConnectionType             : IPsec
    RoutingWeight              : 0
    SharedKey                  : ####################################
    ConnectionStatus           : Connected
    EgressBytesTransferred     : 55254803
    IngressBytesTransferred    : 32227221
    ProvisioningState          : Succeeded
    ...
    ```

    ```powershell
    PS C:\> Get-AzureRmVirtualNetworkGatewayConnection -Name profx-gateway-connection2 -ResourceGroupName profx-prod-rg

    AuthorizationKey           :
    VirtualNetworkGateway1     : Microsoft.Azure.Commands.Network.Models.PSVirtualNetworkGateway
    VirtualNetworkGateway2     :
    LocalNetworkGateway2       : Microsoft.Azure.Commands.Network.Models.PSLocalNetworkGateway
    Peer                       :
    ConnectionType             : IPsec
    RoutingWeight              : 0
    SharedKey                  : ####################################
    ConnectionStatus           : NotConnected
    EgressBytesTransferred     : 0
    IngressBytesTransferred    : 0
    ProvisioningState          : Succeeded
    ...
    ```

<span data-ttu-id="a1d8a-272">Las recomendaciones siguientes son útiles para determinar si hay algún problema con el uso del ancho de banda, la configuración de la máquina virtual host o el rendimiento de la aplicación:</span><span class="sxs-lookup"><span data-stu-id="a1d8a-272">The following recommendations are useful for determining if there is an issue with Host VM configuration, network bandwidth utilization, or application performance:</span></span>

- <span data-ttu-id="a1d8a-273">**Compruebe que el firewall del sistema operativo invitado que se ejecuta en las máquinas virtuales de Azure en la subred esté configurado correctamente para permitir el tráfico de los intervalos de IP locales.**</span><span class="sxs-lookup"><span data-stu-id="a1d8a-273">**Verify that the firewall in the guest operating system running on the Azure VMs in the subnet is configured correctly to allow permitted traffic from the on-premises IP ranges.**</span></span>

- <span data-ttu-id="a1d8a-274">**Compruebe que el volumen de tráfico no se acerque al límite del ancho de banda disponible en Azure VPN Gateway.**</span><span class="sxs-lookup"><span data-stu-id="a1d8a-274">**Verify that the volume of traffic is not close to the limit of the bandwidth available to the Azure VPN gateway.**</span></span>

    <span data-ttu-id="a1d8a-275">La manera de comprobarlo depende del dispositivo VPN que se ejecute localmente.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-275">How to verify this depends on the VPN appliance running on-premises.</span></span> <span data-ttu-id="a1d8a-276">Por ejemplo, si está usando RRAS en Windows Server 2012, puede usar el Monitor de rendimiento para el seguimiento del volumen de datos que se recibe y se transmite a través de la conexión VPN.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-276">For example, if you are using RRAS on Windows Server 2012, you can use Performance Monitor to track the volume of data being received and transmitted over the VPN connection.</span></span> <span data-ttu-id="a1d8a-277">Mediante el objeto *Total de RAS*, seleccione los contadores *Bytes recibidos/s* y *Bytes transmitidos/s*:</span><span class="sxs-lookup"><span data-stu-id="a1d8a-277">Using the *RAS Total* object, select the *Bytes Received/Sec* and *Bytes Transmitted/Sec* counters:</span></span>

    ![Contadores de rendimiento para supervisar el tráfico de red VPN](../_images/guidance-hybrid-network-vpn/RRAS-perf-counters.png)

    <span data-ttu-id="a1d8a-279">Debería comparar los resultados con el ancho de banda disponible para VPN Gateway (desde 100 Mbps para la SKU Básica hasta 1,25 Mbps para la SKU VpnGw3):</span><span class="sxs-lookup"><span data-stu-id="a1d8a-279">You should compare the results with the bandwidth available to the VPN gateway (from 100 Mbps for the Basic SKU to 1.25 Gbps for VpnGw3 SKU):</span></span>

    ![Ejemplo de gráfico de rendimiento de red VPN](../_images/guidance-hybrid-network-vpn/RRAS-perf-graph.png)

- <span data-ttu-id="a1d8a-281">**Compruebe que implementó la cantidad y el tamaño correctos de máquinas virtuales para la carga de la aplicación.**</span><span class="sxs-lookup"><span data-stu-id="a1d8a-281">**Verify that you have deployed the right number and size of VMs for your application load.**</span></span>

    <span data-ttu-id="a1d8a-282">Determine si alguna de las máquinas virtuales de la red virtual de Azure se ejecuta con lentitud.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-282">Determine if any of the virtual machines in the Azure VNet are running slowly.</span></span> <span data-ttu-id="a1d8a-283">Si es así, es posible que estén sobrecargadas, que sean insuficientes para controlar la carga o que los equilibradores de carga no estén configurados correctamente.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-283">If so, they may be overloaded, there may be too few to handle the load, or the load-balancers may not be configured correctly.</span></span> <span data-ttu-id="a1d8a-284">Para averiguarlo, consulte [Microsoft Azure Virtual Machine Monitoring with Azure Diagnostics Extension][azure-vm-diagnostics] (Supervisión de máquinas virtuales de Microsoft Azure mediante la extensión de Azure Diagnostics).</span><span class="sxs-lookup"><span data-stu-id="a1d8a-284">To determine this, [capture and analyze diagnostic information][azure-vm-diagnostics].</span></span> <span data-ttu-id="a1d8a-285">Puede examinar los resultados mediante Azure Portal, aunque también existen muchas herramientas de terceros que pueden proporcionar información detallada sobre los datos de rendimiento.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-285">You can examine the results using the Azure portal, but many third-party tools are also available that can provide detailed insights into the performance data.</span></span>

- <span data-ttu-id="a1d8a-286">**Compruebe que la aplicación haga un uso eficaz de los recursos de nube.**</span><span class="sxs-lookup"><span data-stu-id="a1d8a-286">**Verify that the application is making efficient use of cloud resources.**</span></span>

    <span data-ttu-id="a1d8a-287">Instrumente el código de aplicación que se ejecuta en cada máquina virtual para determinar si las aplicaciones están sacando el máximo partido de los recursos.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-287">Instrument application code running on each VM to determine whether applications are making the best use of resources.</span></span> <span data-ttu-id="a1d8a-288">Puede usar herramientas como [Application Insights][application-insights].</span><span class="sxs-lookup"><span data-stu-id="a1d8a-288">You can use tools such as [Application Insights][application-insights].</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="a1d8a-289">Implementación de la solución</span><span class="sxs-lookup"><span data-stu-id="a1d8a-289">Deploy the solution</span></span>

<span data-ttu-id="a1d8a-290">**Requisitos previos**.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-290">**Prerequisites**.</span></span> <span data-ttu-id="a1d8a-291">Debe tener una infraestructura local existente ya configurada con un dispositivo de red adecuado.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-291">You must have an existing on-premises infrastructure already configured with a suitable network appliance.</span></span>

<span data-ttu-id="a1d8a-292">Para implementar la solución, siga estos pasos:</span><span class="sxs-lookup"><span data-stu-id="a1d8a-292">To deploy the solution, perform the following steps.</span></span>

<!-- markdownlint-disable MD033 -->

1. <span data-ttu-id="a1d8a-293">Haga clic en el botón a continuación:</span><span class="sxs-lookup"><span data-stu-id="a1d8a-293">Click the button below:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fvpn%2Fazuredeploy.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. <span data-ttu-id="a1d8a-294">Espere a que el vínculo abra Azure Portal y, a continuación, siga estos pasos:</span><span class="sxs-lookup"><span data-stu-id="a1d8a-294">Wait for the link to open in the Azure portal, then follow these steps:</span></span>
   - <span data-ttu-id="a1d8a-295">El nombre del **Grupo de recursos** ya está definido en el archivo de parámetros, así que seleccione **Crear nuevo** y escriba `ra-hybrid-vpn-rg` en el cuadro de texto.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-295">The **Resource group** name is already defined in the parameter file, so select **Create New** and enter `ra-hybrid-vpn-rg` in the text box.</span></span>
   - <span data-ttu-id="a1d8a-296">Seleccione la región en el cuadro de lista desplegable **Ubicación**.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-296">Select the region from the **Location** drop down box.</span></span>
   - <span data-ttu-id="a1d8a-297">No modifique los cuadros de texto **URI raíz de plantilla** o **URI raíz de parámetro**.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-297">Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.</span></span>
   - <span data-ttu-id="a1d8a-298">Revise los términos y condiciones, y haga clic en la casilla **Acepto los términos y condiciones indicados anteriormente**.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-298">Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.</span></span>
   - <span data-ttu-id="a1d8a-299">Haga clic en el botón **Comprar**.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-299">Click the **Purchase** button.</span></span>
3. <span data-ttu-id="a1d8a-300">Espere a que la implementación se complete.</span><span class="sxs-lookup"><span data-stu-id="a1d8a-300">Wait for the deployment to complete.</span></span>

<!-- markdownlint-enable MD033 -->

<!-- links -->

[adds-extend-domain]: ../identity/adds-extend-domain.md
[expressroute]: ../hybrid-networking/expressroute.md
[windows-vm-ra]: ../virtual-machines-windows/index.md
[linux-vm-ra]: ../virtual-machines-linux/index.md

[naming conventions]: /azure/guidance/guidance-naming-conventions

[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[arm-templates]: /azure/resource-group-authoring-templates
[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-portal]: /azure/azure-portal/resource-group-portal
[azure-powershell]: /azure/powershell-azure-resource-manager
[azure-virtual-network]: /azure/virtual-network/virtual-networks-overview
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[azure-vpn-gateway]: https://azure.microsoft.com/services/vpn-gateway/
[azure-gateway-charges]: https://azure.microsoft.com/pricing/details/vpn-gateway/
[azure-gateway-skus]: /azure/vpn-gateway/vpn-gateway-about-vpngateways#gwsku
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[vpn-gateway-multi-site]: /azure/vpn-gateway/vpn-gateway-multi-site
[policy-based-routing]: https://en.wikipedia.org/wiki/Policy-based_routing
[route-based-routing]: https://en.wikipedia.org/wiki/Static_routing
[network-security-group]: /azure/virtual-network/virtual-networks-nsg
[sla-for-vpn-gateway]: https://azure.microsoft.com/support/legal/sla/vpn-gateway/v1_2/
[additional-firewall-rules]: https://technet.microsoft.com/library/dn786406.aspx#firewall
[nagios]: https://www.nagios.org/
[azure-vpn-gateway-diagnostics]: https://blogs.technet.microsoft.com/keithmayer/2014/12/18/diagnose-azure-virtual-network-vpn-connectivity-issues-with-powershell/
[ping]: https://technet.microsoft.com/library/ff961503.aspx
[tracert]: https://technet.microsoft.com/library/ff961507.aspx
[psping]: https://technet.microsoft.com/sysinternals/jj729731.aspx
[nmap]: https://nmap.org
[changing-SKUs]: https://azure.microsoft.com/blog/azure-virtual-network-gateway-improvements/
[gateway-diagnostic-logs]: https://blogs.technet.microsoft.com/keithmayer/2016/10/12/step-by-step-capturing-azure-resource-manager-arm-vnet-gateway-diagnostic-logs/
[troubleshooting-vpn-errors]: https://blogs.technet.microsoft.com/rrasblog/2009/08/12/troubleshooting-common-vpn-related-errors/
[rras-logging]: https://www.petri.com/enable-diagnostic-logging-in-windows-server-2012-r2-routing-and-remote-access
[create-on-prem-network]: https://technet.microsoft.com/library/dn786406.aspx#routing
[azure-vm-diagnostics]: https://azure.microsoft.com/blog/windows-azure-virtual-machine-monitoring-with-wad-extension/
[application-insights]: /azure/application-insights/app-insights-overview-usage
[forced-tunneling]: https://azure.microsoft.com/documentation/articles/vpn-gateway-about-forced-tunneling/
[vpn-appliances]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-architectures.vsdx
[vpn-appliance-ipsec]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices#ipsec-parameters
[virtualNetworkGateway-parameters]: https://github.com/mspnp/hybrid-networking/vpn/parameters/virtualNetworkGateway.parameters.json
[azure-cli]: https://azure.microsoft.com/documentation/articles/xplat-cli-install/
[CIDR]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
