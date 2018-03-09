---
title: "Implementación de una topología de red en estrella tipo hub-and-spoke en Azure"
description: "Cómo implementar una topología de red en estrella tipo hub-and-spoke en Azure."
author: telmosampaio
ms.date: 02/23/2018
pnp.series.title: Implement a hub-spoke network topology in Azure
pnp.series.prev: expressroute
ms.openlocfilehash: 1a2855f0d4a903fc4d7a022aef20ea73fe763e2c
ms.sourcegitcommit: 2123c25b1a0b5501ff1887f98030787191cf6994
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/08/2018
---
# <a name="implement-a-hub-spoke-network-topology-in-azure"></a><span data-ttu-id="4e75a-103">Implementación de una topología de red en estrella tipo hub-and-spoke en Azure</span><span class="sxs-lookup"><span data-stu-id="4e75a-103">Implement a hub-spoke network topology in Azure</span></span>

<span data-ttu-id="4e75a-104">En esta arquitectura de referencia se muestra cómo implementar una topología en estrella tipo hub-and-spoke en Azure.</span><span class="sxs-lookup"><span data-stu-id="4e75a-104">This reference architecture shows how to implement a hub-spoke topology in Azure.</span></span> <span data-ttu-id="4e75a-105">El *concentrador* (hub) es una red virtual (VNet) en Azure que actúa como un punto central de conectividad para la red local.</span><span class="sxs-lookup"><span data-stu-id="4e75a-105">The *hub* is a virtual network (VNet) in Azure that acts as a central point of connectivity to your on-premises network.</span></span> <span data-ttu-id="4e75a-106">Los *radios* (spoke) son redes virtuales (VNet) que se emparejan con el concentrador y que se pueden usar para aislar cargas de trabajo.</span><span class="sxs-lookup"><span data-stu-id="4e75a-106">The *spokes* are VNets that peer with the hub, and can be used to isolate workloads.</span></span> <span data-ttu-id="4e75a-107">El tráfico fluye entre el centro de datos local y el concentrador a través de una conexión a ExpressRoute o a VPN Gateway.</span><span class="sxs-lookup"><span data-stu-id="4e75a-107">Traffic flows between the on-premises datacenter and the hub through an ExpressRoute or VPN gateway connection.</span></span>  <span data-ttu-id="4e75a-108">[**Implemente esta solución**](#deploy-the-solution).</span><span class="sxs-lookup"><span data-stu-id="4e75a-108">[**Deploy this solution**](#deploy-the-solution).</span></span>

<span data-ttu-id="4e75a-109">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="4e75a-109">![[0]][0]</span></span>

<span data-ttu-id="4e75a-110">*Descargue un [archivo Visio][visio-download] de esta arquitectura.*</span><span class="sxs-lookup"><span data-stu-id="4e75a-110">*Download a [Visio file][visio-download] of this architecture*</span></span>


<span data-ttu-id="4e75a-111">Las ventajas de esta topología son:</span><span class="sxs-lookup"><span data-stu-id="4e75a-111">The benefits of this toplogy include:</span></span>

* <span data-ttu-id="4e75a-112">**Ahorros en costos** gracias a la centralización de los servicios que se pueden compartir entre varias cargas de trabajo, como las aplicaciones virtuales de red (NVA) y los servidores DNS, en una única ubicación.</span><span class="sxs-lookup"><span data-stu-id="4e75a-112">**Cost savings** by centralizing services that can be shared by multiple workloads, such as network virtual appliances (NVAs) and DNS servers, in a single location.</span></span>
* <span data-ttu-id="4e75a-113">**Superar los límites de las suscripciones** gracias al emparejamiento de las redes virtuales de diferentes suscripciones con el concentrador central.</span><span class="sxs-lookup"><span data-stu-id="4e75a-113">**Overcome subscriptions limits** by peering VNets from different subscriptions to the central hub.</span></span>
* <span data-ttu-id="4e75a-114">**Separación de intereses** entre la TI central (SecOps e InfraOps) y las cargas de trabajo (DevOps).</span><span class="sxs-lookup"><span data-stu-id="4e75a-114">**Separation of concerns** between central IT (SecOps, InfraOps) and workloads (DevOps).</span></span>

<span data-ttu-id="4e75a-115">Los usos habituales de esta arquitectura incluyen:</span><span class="sxs-lookup"><span data-stu-id="4e75a-115">Typical uses for this architecture include:</span></span>

* <span data-ttu-id="4e75a-116">Cargas de trabajo implementadas en distintos entornos, como desarrollo, pruebas y producción, que requieren servicios compartidos como DNS, IDS, NTP o AD DS.</span><span class="sxs-lookup"><span data-stu-id="4e75a-116">Workloads deployed in different environments, such as development, testing, and production, that require shared services such as DNS, IDS, NTP, or AD DS.</span></span> <span data-ttu-id="4e75a-117">Los servicios compartidos se colocan en la red virtual del concentrador, mientras que cada entorno se implementa en un radio para mantener el aislamiento.</span><span class="sxs-lookup"><span data-stu-id="4e75a-117">Shared services are placed in the hub VNet, while each environment is deployed to a spoke to maintain isolation.</span></span>
* <span data-ttu-id="4e75a-118">Cargas de trabajo que no requieren conectividad entre sí, pero requieren acceso a los servicios compartidos.</span><span class="sxs-lookup"><span data-stu-id="4e75a-118">Workloads that do not require connectivity to each other, but require access to shared services.</span></span>
* <span data-ttu-id="4e75a-119">Empresas que requieren el control centralizado sobre aspectos de seguridad, como un firewall en el concentrador como una red perimetral, así como la administración segregada de las cargas de trabajo en cada radio.</span><span class="sxs-lookup"><span data-stu-id="4e75a-119">Enterprises that require central control over security aspects, such as a firewall in the hub as a DMZ, and segregated management for the workloads in each spoke.</span></span>

## <a name="architecture"></a><span data-ttu-id="4e75a-120">Architecture</span><span class="sxs-lookup"><span data-stu-id="4e75a-120">Architecture</span></span>

<span data-ttu-id="4e75a-121">La arquitectura consta de los siguientes componentes:</span><span class="sxs-lookup"><span data-stu-id="4e75a-121">The architecture consists of the following components.</span></span>

* <span data-ttu-id="4e75a-122">**Red local**.</span><span class="sxs-lookup"><span data-stu-id="4e75a-122">**On-premises network**.</span></span> <span data-ttu-id="4e75a-123">Una red de área local privada que se ejecuta dentro de una organización.</span><span class="sxs-lookup"><span data-stu-id="4e75a-123">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="4e75a-124">**Dispositivo VPN**.</span><span class="sxs-lookup"><span data-stu-id="4e75a-124">**VPN device**.</span></span> <span data-ttu-id="4e75a-125">Un dispositivo o servicio que proporciona conectividad externa a la red local.</span><span class="sxs-lookup"><span data-stu-id="4e75a-125">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="4e75a-126">El dispositivo VPN puede ser un dispositivo de hardware, o puede ser una solución de software como el servicio de Enrutamiento y acceso remoto (RRAS) en Windows Server 2012.</span><span class="sxs-lookup"><span data-stu-id="4e75a-126">The VPN device may be a hardware device, or a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="4e75a-127">Para obtener una lista de dispositivos VPN compatibles e información acerca de cómo configurar dispositivos VPN seleccionados para conectarse a Azure, consulte [Acerca de los dispositivos VPN y los parámetros de IPsec/IKE para conexiones de VPN Gateway de sitio a sitio][vpn-appliance].</span><span class="sxs-lookup"><span data-stu-id="4e75a-127">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="4e75a-128">**Puerta de enlace de red virtual de VPN o puerta de enlace de ExpressRoute**.</span><span class="sxs-lookup"><span data-stu-id="4e75a-128">**VPN virtual network gateway or ExpressRoute gateway**.</span></span> <span data-ttu-id="4e75a-129">La puerta de enlace de red virtual permite que la red virtual se conecte al dispositivo VPN o al circuito de ExpressRoute que se usa para la conectividad con la red local.</span><span class="sxs-lookup"><span data-stu-id="4e75a-129">The virtual network gateway enables the VNet to connect to the VPN device, or ExpressRoute circuit, used for connectivity with your on-premises network.</span></span> <span data-ttu-id="4e75a-130">Para más información, consulte [Conectar una red local con una red virtual de Microsoft Azure][connect-to-an-Azure-vnet].</span><span class="sxs-lookup"><span data-stu-id="4e75a-130">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

> [!NOTE]
> <span data-ttu-id="4e75a-131">Los scripts de implementación para esta arquitectura de referencia usan una instancia de VPN Gateway para la conectividad y una red virtual en Azure para simular la red local.</span><span class="sxs-lookup"><span data-stu-id="4e75a-131">The deployment scripts for this reference architecture use a VPN gateway for connectivity, and a VNet in Azure to simulate your on-premises network.</span></span>

* <span data-ttu-id="4e75a-132">**Red virtual del concentrador**.</span><span class="sxs-lookup"><span data-stu-id="4e75a-132">**Hub VNet**.</span></span> <span data-ttu-id="4e75a-133">La red virtual de Azure utilizada como el concentrador en la topología en estrella tipo hub-and-spoke.</span><span class="sxs-lookup"><span data-stu-id="4e75a-133">Azure VNet used as the hub in the hub-spoke topology.</span></span> <span data-ttu-id="4e75a-134">El concentrador es el punto central de conectividad a la red local y un lugar para hospedar los servicios que se pueden usar en las diferentes cargas de trabajo hospedadas en las redes virtuales de los radios.</span><span class="sxs-lookup"><span data-stu-id="4e75a-134">The hub is the central point of connectivity to your on-premises network, and a place to host services that can be consumed by the different workloads hosted in the spoke VNets.</span></span>

* <span data-ttu-id="4e75a-135">**Subred de puerta de enlace**.</span><span class="sxs-lookup"><span data-stu-id="4e75a-135">**Gateway subnet**.</span></span> <span data-ttu-id="4e75a-136">Las puertas de enlace de red virtual se conservan en la misma subred.</span><span class="sxs-lookup"><span data-stu-id="4e75a-136">The virtual network gateways are held in the same subnet.</span></span>

* <span data-ttu-id="4e75a-137">**Redes virtuales de radios**.</span><span class="sxs-lookup"><span data-stu-id="4e75a-137">**Spoke VNets**.</span></span> <span data-ttu-id="4e75a-138">Una o varias redes virtuales de Azure que se usan como radios en la topología en estrella tipo hub-and-spoke.</span><span class="sxs-lookup"><span data-stu-id="4e75a-138">One or more Azure VNets that are used as spokes in the hub-spoke topology.</span></span> <span data-ttu-id="4e75a-139">Los radios pueden utilizarse para aislar las cargas de trabajo en sus propias redes virtuales, administradas por separado desde otros radios.</span><span class="sxs-lookup"><span data-stu-id="4e75a-139">Spokes can be used to isolate workloads in their own VNets, managed separately from other spokes.</span></span> <span data-ttu-id="4e75a-140">Cada carga de trabajo puede incluir varios niveles, con varias subredes que se conectan a través de equilibradores de carga de Azure.</span><span class="sxs-lookup"><span data-stu-id="4e75a-140">Each workload might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="4e75a-141">Para obtener más información acerca de la infraestructura de aplicaciones, consulte [Ejecutar cargas de trabajo de máquinas virtuales Windows][windows-vm-ra] y [Ejecutar cargas de trabajo de máquinas virtuales Linux][linux-vm-ra].</span><span class="sxs-lookup"><span data-stu-id="4e75a-141">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

* <span data-ttu-id="4e75a-142">**Emparejamiento de VNET**.</span><span class="sxs-lookup"><span data-stu-id="4e75a-142">**VNet peering**.</span></span> <span data-ttu-id="4e75a-143">Se pueden conectar dos redes virtuales en la misma región de Azure mediante una [conexión de emparejamiento][vnet-peering].</span><span class="sxs-lookup"><span data-stu-id="4e75a-143">Two VNets in the same Azure region can be connected using a [peering connection][vnet-peering].</span></span> <span data-ttu-id="4e75a-144">Las conexiones de emparejamiento son conexiones no transitivas de baja latencia entre las redes virtuales.</span><span class="sxs-lookup"><span data-stu-id="4e75a-144">Peering connections are non-transitive, low latency connections between VNets.</span></span> <span data-ttu-id="4e75a-145">Una vez establecido el emparejamiento, las redes virtuales intercambian el tráfico mediante la red troncal de Azure, sin necesidad de un enrutador.</span><span class="sxs-lookup"><span data-stu-id="4e75a-145">Once peered, the VNets exchange traffic by using the Azure backbone, without the need for a router.</span></span> <span data-ttu-id="4e75a-146">En una topología de red en estrella tipo hub-and-spoke, el emparejamiento de VNET se usa para conectar el concentrador a cada radio.</span><span class="sxs-lookup"><span data-stu-id="4e75a-146">In a hub-spoke network topology, you use VNet peering to connect the hub to each spoke.</span></span>

> [!NOTE]
> <span data-ttu-id="4e75a-147">En este artículo solo se tratan las implementaciones de [Resource Manager](/azure/azure-resource-manager/resource-group-overview), pero también puede conectar una red virtual clásica a una red virtual de Resource Manager en la misma suscripción.</span><span class="sxs-lookup"><span data-stu-id="4e75a-147">This article only covers [Resource Manager](/azure/azure-resource-manager/resource-group-overview) deployments, but you can also connect a classic VNet to a Resource Manager VNet in the same subscription.</span></span> <span data-ttu-id="4e75a-148">De este modo, los radios pueden hospedar implementaciones clásicas y seguir beneficiándose de los servicios compartidos en el concentrador.</span><span class="sxs-lookup"><span data-stu-id="4e75a-148">That way, your spokes can host classic deployments and still benefit from services shared in the hub.</span></span>

## <a name="recommendations"></a><span data-ttu-id="4e75a-149">Recomendaciones</span><span class="sxs-lookup"><span data-stu-id="4e75a-149">Recommendations</span></span>

<span data-ttu-id="4e75a-150">Las siguientes recomendaciones sirven para la mayoría de los escenarios.</span><span class="sxs-lookup"><span data-stu-id="4e75a-150">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="4e75a-151">Sígalas a menos que tenga un requisito concreto que las invalide.</span><span class="sxs-lookup"><span data-stu-id="4e75a-151">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="resource-groups"></a><span data-ttu-id="4e75a-152">Grupos de recursos</span><span class="sxs-lookup"><span data-stu-id="4e75a-152">Resource groups</span></span>

<span data-ttu-id="4e75a-153">La red virtual del concentrador y la red virtual de cada radio se pueden implementar en diferentes grupos de recursos e incluso en distintas suscripciones, siempre y cuando pertenezcan al mismo inquilino de Azure Active Directory (Azure AD) en la misma región.</span><span class="sxs-lookup"><span data-stu-id="4e75a-153">The hub VNet, and each spoke VNet, can be implemented in different resource groups, and even different subscriptions, as long as they belong to the same Azure Active Directory (Azure AD) tenant in the same Azure region.</span></span> <span data-ttu-id="4e75a-154">Esto permite realizar una administración descentralizada de cada carga de trabajo, mientras se comparten los servicios que se mantienen en la red virtual del concentrador.</span><span class="sxs-lookup"><span data-stu-id="4e75a-154">This allows for a decentralized management of each workload, while sharing services maintained in the hub VNet.</span></span>

### <a name="vnet-and-gatewaysubnet"></a><span data-ttu-id="4e75a-155">VNet y GatewaySubnet</span><span class="sxs-lookup"><span data-stu-id="4e75a-155">VNet and GatewaySubnet</span></span>

<span data-ttu-id="4e75a-156">Cree una subred denominada *GatewaySubnet*, con un intervalo de direcciones de /27.</span><span class="sxs-lookup"><span data-stu-id="4e75a-156">Create a subnet named *GatewaySubnet*, with an address range of /27.</span></span> <span data-ttu-id="4e75a-157">Esta subred es necesaria para la puerta de enlace de red virtual.</span><span class="sxs-lookup"><span data-stu-id="4e75a-157">This subnet is required by the virtual network gateway.</span></span> <span data-ttu-id="4e75a-158">La asignación de treinta y dos direcciones a esta subred ayudará a evitar que se alcancen las limitaciones de tamaño de la puerta de enlace en el futuro.</span><span class="sxs-lookup"><span data-stu-id="4e75a-158">Allocating 32 addresses to this subnet will help to prevent reaching gateway size limitations in the future.</span></span>

<span data-ttu-id="4e75a-159">Para más información sobre la configuración de la puerta de enlace, consulte las siguientes arquitecturas de referencia, en función del tipo de conexión:</span><span class="sxs-lookup"><span data-stu-id="4e75a-159">For more information about setting up the gateway, see the following reference architectures, depending on your connection type:</span></span>

- <span data-ttu-id="4e75a-160">[Red híbrida mediante ExpressRoute][guidance-expressroute]</span><span class="sxs-lookup"><span data-stu-id="4e75a-160">[Hybrid network using ExpressRoute][guidance-expressroute]</span></span>
- <span data-ttu-id="4e75a-161">[Red híbrida con una instancia de VPN Gateway][guidance-vpn]</span><span class="sxs-lookup"><span data-stu-id="4e75a-161">[Hybrid network using a VPN gateway][guidance-vpn]</span></span>

<span data-ttu-id="4e75a-162">Para una mayor disponibilidad, puede utilizar ExpressRoute y una VPN para la conmutación por error.</span><span class="sxs-lookup"><span data-stu-id="4e75a-162">For higher availability, you can use ExpressRoute plus a VPN for failover.</span></span> <span data-ttu-id="4e75a-163">Vea [Conexión de una red local a Azure mediante ExpressRoute con conmutación por error de VPN][hybrid-ha].</span><span class="sxs-lookup"><span data-stu-id="4e75a-163">See [Connect an on-premises network to Azure using ExpressRoute with VPN failover][hybrid-ha].</span></span>

<span data-ttu-id="4e75a-164">También se puede utilizar una topología en estrella tipo hub-and-spoke sin una puerta de enlace, en caso de que no se necesite la conectividad con la red local.</span><span class="sxs-lookup"><span data-stu-id="4e75a-164">A hub-spoke topology can also be used without a gateway, if you don't need connectivity with your on-premises network.</span></span> 

### <a name="vnet-peering"></a><span data-ttu-id="4e75a-165">Emparejamiento de VNET</span><span class="sxs-lookup"><span data-stu-id="4e75a-165">VNet peering</span></span>

<span data-ttu-id="4e75a-166">El emparejamiento de VNET es una relación no transitiva entre dos redes virtuales.</span><span class="sxs-lookup"><span data-stu-id="4e75a-166">VNet peering is a non-transitive relationship between two VNets.</span></span> <span data-ttu-id="4e75a-167">Si necesita que los radios se conecten entre sí, considere la posibilidad de agregar una conexión de emparejamiento independiente entre dichos radios.</span><span class="sxs-lookup"><span data-stu-id="4e75a-167">If you require spokes to connect to each other, consider adding a separate peering connection between those spokes.</span></span>

<span data-ttu-id="4e75a-168">Sin embargo, si tiene varios radios que necesitan conectarse entre sí, se quedará sin posibles conexiones de emparejamiento muy rápido debido a la [limitación del número de emparejamientos de VNET por cada red virtual][vnet-peering-limit].</span><span class="sxs-lookup"><span data-stu-id="4e75a-168">However, if you have several spokes that need to connect with each other, you will run out of possible peering connections very quickly due to the [limitation on number of VNets peerings per VNet][vnet-peering-limit].</span></span> <span data-ttu-id="4e75a-169">En este escenario, considere la posibilidad de usar rutas definidas por el usuario (UDR) para forzar que el tráfico destinado a un radio se envíe a una NVA que actúa como un enrutador en la red virtual del concentrador.</span><span class="sxs-lookup"><span data-stu-id="4e75a-169">In this scenario, consider using user defined routes (UDRs) to force traffic destined to a spoke to be sent to an NVA acting as a router at the hub VNet.</span></span> <span data-ttu-id="4e75a-170">Esto permitirá que los radios se conecten entre sí.</span><span class="sxs-lookup"><span data-stu-id="4e75a-170">This will allow the spokes to connect to each other.</span></span>

<span data-ttu-id="4e75a-171">También puede configurar los radios para que usen la puerta de enlace de la red virtual del concentrador para comunicarse con las redes remotas.</span><span class="sxs-lookup"><span data-stu-id="4e75a-171">You can also configure spokes to use the hub VNet gateway to communicate with remote networks.</span></span> <span data-ttu-id="4e75a-172">Para permitir que el tráfico de la puerta de enlace fluya del radio al concentrador y para conectarse a las redes remotas, debe:</span><span class="sxs-lookup"><span data-stu-id="4e75a-172">To allow gateway traffic to flow from spoke to hub, and connect to remote networks, you must:</span></span>

  - <span data-ttu-id="4e75a-173">Configurar la conexión de emparejamiento de VNET en el concentrador para **permitir el tránsito de la puerta de enlace**.</span><span class="sxs-lookup"><span data-stu-id="4e75a-173">Configure the VNet peering connection in the hub to **allow gateway transit**.</span></span>
  - <span data-ttu-id="4e75a-174">Configurar la conexión de emparejamiento de VNET en cada radio para **usar las puertas de enlace remotas**.</span><span class="sxs-lookup"><span data-stu-id="4e75a-174">Configure the VNet peering connection in each spoke to **use remote gateways**.</span></span>
  - <span data-ttu-id="4e75a-175">Configurar todas las conexiones de emparejamiento de VNET para **permitir el tráfico reenviado**.</span><span class="sxs-lookup"><span data-stu-id="4e75a-175">Configure all VNet peering connections to **allow forwarded traffic**.</span></span>

## <a name="considerations"></a><span data-ttu-id="4e75a-176">Consideraciones</span><span class="sxs-lookup"><span data-stu-id="4e75a-176">Considerations</span></span>

### <a name="spoke-connectivity"></a><span data-ttu-id="4e75a-177">Conectividad de los radios</span><span class="sxs-lookup"><span data-stu-id="4e75a-177">Spoke connectivity</span></span>

<span data-ttu-id="4e75a-178">Si se necesita una conectividad entre los radios, considere la posibilidad de implementar una NVA para el enrutamiento en el concentrador y de usar las UDR en el radio para reenviar el tráfico al concentrador.</span><span class="sxs-lookup"><span data-stu-id="4e75a-178">If you require connectivity between spokes, consider implementing an NVA for routing in the hub, and using UDRs in the spoke to forward traffic to the hub.</span></span>

<span data-ttu-id="4e75a-179">![[2]][2]</span><span class="sxs-lookup"><span data-stu-id="4e75a-179">![[2]][2]</span></span>

<span data-ttu-id="4e75a-180">En este escenario, debe configurar las conexiones de emparejamiento para que **permitan el tráfico reenviado**.</span><span class="sxs-lookup"><span data-stu-id="4e75a-180">In this scenario, you must configure the peering connections to **allow forwarded traffic**.</span></span>

### <a name="overcoming-vnet-peering-limits"></a><span data-ttu-id="4e75a-181">Superación de los límites de emparejamiento de VNET</span><span class="sxs-lookup"><span data-stu-id="4e75a-181">Overcoming VNet peering limits</span></span>

<span data-ttu-id="4e75a-182">Asegúrese de tener en cuenta la [limitación del número de emparejamientos de VNET por cada red virtual][vnet-peering-limit] en Azure.</span><span class="sxs-lookup"><span data-stu-id="4e75a-182">Make sure you consider the [limitation on number of VNets peerings per VNet][vnet-peering-limit] in Azure.</span></span> <span data-ttu-id="4e75a-183">Si decide que necesita más radios de los que el límite permitirá, considere la posibilidad de crear una topología en estrella tipo hub-hub-and-spoke-spoke, donde el primer nivel de radios también actúa como concentradores.</span><span class="sxs-lookup"><span data-stu-id="4e75a-183">If you decide you need more spokes than the limit will allow, consider creating a hub-spoke-hub-spoke topology, where the first level of spokes also act as hubs.</span></span> <span data-ttu-id="4e75a-184">En el siguiente diagrama se ilustra este enfoque.</span><span class="sxs-lookup"><span data-stu-id="4e75a-184">The following diagram shows this approach.</span></span>

<span data-ttu-id="4e75a-185">![[3]][3]</span><span class="sxs-lookup"><span data-stu-id="4e75a-185">![[3]][3]</span></span>

<span data-ttu-id="4e75a-186">También tenga en cuenta qué servicios se comparten en el concentrador, para asegurarse de que el concentrador se escala para tener un número mayor de radios.</span><span class="sxs-lookup"><span data-stu-id="4e75a-186">Also consider what services are shared in the hub, to ensure the hub scales for a larger number of spokes.</span></span> <span data-ttu-id="4e75a-187">Por ejemplo, si el concentrador proporciona servicios de firewall, tenga en cuenta los límites de ancho de banda de la solución de firewall al agregar varios radios.</span><span class="sxs-lookup"><span data-stu-id="4e75a-187">For instance, if your hub provides firewall services, consider the bandwidth limits of your firewall solution when adding multiple spokes.</span></span> <span data-ttu-id="4e75a-188">Puede mover algunos de estos servicios compartidos a un segundo nivel de concentradores.</span><span class="sxs-lookup"><span data-stu-id="4e75a-188">You might want to move some of these shared services to a second level of hubs.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="4e75a-189">Implementación de la solución</span><span class="sxs-lookup"><span data-stu-id="4e75a-189">Deploy the solution</span></span>

<span data-ttu-id="4e75a-190">Hay disponible una implementación de esta arquitectura en [GitHub][ref-arch-repo].</span><span class="sxs-lookup"><span data-stu-id="4e75a-190">A deployment for this architecture is available on [GitHub][ref-arch-repo].</span></span> <span data-ttu-id="4e75a-191">Usa máquinas virtuales Ubuntu en cada red virtual para probar la conectividad.</span><span class="sxs-lookup"><span data-stu-id="4e75a-191">It uses Ubuntu VMs in each VNet to test connectivity.</span></span> <span data-ttu-id="4e75a-192">No hay ningún servicio real hospedado en la subred de **servicios compartidos** de la **red virtual del concentrador**.</span><span class="sxs-lookup"><span data-stu-id="4e75a-192">There are no actual services hosted in the **shared-services** subnet in the **hub VNet**.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="4e75a-193">requisitos previos</span><span class="sxs-lookup"><span data-stu-id="4e75a-193">Prerequisites</span></span>

<span data-ttu-id="4e75a-194">Antes de poder implementar la arquitectura de referencia en su propia suscripción, debe realizar los pasos siguientes.</span><span class="sxs-lookup"><span data-stu-id="4e75a-194">Before you can deploy the reference architecture to your own subscription, you must perform the following steps.</span></span>

1. <span data-ttu-id="4e75a-195">Clone, bifurque o descargue el archivo ZIP para el repositorio de GitHub de [arquitecturas de referencia de AzureCAT][ref-arch-repo].</span><span class="sxs-lookup"><span data-stu-id="4e75a-195">Clone, fork, or download the zip file for the [AzureCAT reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="4e75a-196">Asegúrese de que tiene la CLI de Azure 2.0 instalada en el equipo.</span><span class="sxs-lookup"><span data-stu-id="4e75a-196">Make sure you have the Azure CLI 2.0 installed on your computer.</span></span> <span data-ttu-id="4e75a-197">Para obtener instrucciones sobre la instalación de la CLI, consulte [Instalación de la CLI de Azure 2.0][azure-cli-2].</span><span class="sxs-lookup"><span data-stu-id="4e75a-197">For CLI installation instructions, see [Install Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="4e75a-198">Instale el paquete de NPM de [Azure Building Blocks][azbb].</span><span class="sxs-lookup"><span data-stu-id="4e75a-198">Install the [Azure buulding blocks][azbb] npm package.</span></span>

4. <span data-ttu-id="4e75a-199">Desde un símbolo del sistema, un símbolo del sistema de Bash o un símbolo del sistema de PowerShell, inicie sesión en la cuenta de Azure mediante el siguiente comando y siga las indicaciones.</span><span class="sxs-lookup"><span data-stu-id="4e75a-199">From a command prompt, bash prompt, or PowerShell prompt, login to your Azure account by using the command below, and follow the prompts.</span></span>

  ```bash
  az login
  ```

### <a name="deploy-the-simulated-on-premises-datacenter-using-azbb"></a><span data-ttu-id="4e75a-200">Implementación del centro de datos local simulado mediante azbb</span><span class="sxs-lookup"><span data-stu-id="4e75a-200">Deploy the simulated on-premises datacenter using azbb</span></span>

<span data-ttu-id="4e75a-201">Para implementar el centro de datos local simulado como una red virtual de Azure, siga estos pasos:</span><span class="sxs-lookup"><span data-stu-id="4e75a-201">To deploy the simulated on-premises datacenter as an Azure VNet, follow these steps:</span></span>

1. <span data-ttu-id="4e75a-202">Navegue hasta la carpeta `hybrid-networking\hub-spoke\` del repositorio que descargó en el paso de requisitos previos anterior.</span><span class="sxs-lookup"><span data-stu-id="4e75a-202">Navigate to the `hybrid-networking\hub-spoke\` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="4e75a-203">Abra el archivo `onprem.json` y escriba un nombre de usuario y la contraseña entre comillas en las líneas 36 y 37, tal y como se muestra a continuación, y después guarde el archivo.</span><span class="sxs-lookup"><span data-stu-id="4e75a-203">Open the `onprem.json` file and enter a username and password between the quotes in line 36 and 37, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. <span data-ttu-id="4e75a-204">En la línea 38, para `osType`, escriba `Windows` o `Linux` para instalar Windows Server 2016 Datacenter o Ubuntu 16.04 como sistema operativo de JumpBox.</span><span class="sxs-lookup"><span data-stu-id="4e75a-204">On line 38, for `osType`, type `Windows` or `Linux` to install either Windows Server 2016 Datacenter, or Ubuntu 16.04 as the operating system for the jumpbox.</span></span>

4. <span data-ttu-id="4e75a-205">Ejecute `azbb` para implementar el entorno simulado local tal y como se muestra a continuación.</span><span class="sxs-lookup"><span data-stu-id="4e75a-205">Run `azbb` to deploy the simulated onprem environment as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g onprem-vnet-rg - l <location> -p onoprem.json --deploy
  ```
  > [!NOTE]
  > <span data-ttu-id="4e75a-206">Si decide usar un nombre del grupo de recursos distinto (que no sea `onprem-vnet-rg`), asegúrese de que busca todos los archivos de parámetros que usan ese nombre y de que los modifica para usar el nombre de su propio grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="4e75a-206">If you decide to use a different resource group name (other than `onprem-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="4e75a-207">Espere a que finalice la implementación.</span><span class="sxs-lookup"><span data-stu-id="4e75a-207">Wait for the deployment to finish.</span></span> <span data-ttu-id="4e75a-208">Esta implementación crea una red virtual, una máquina virtual y una instancia de VPN Gateway.</span><span class="sxs-lookup"><span data-stu-id="4e75a-208">This deployment creates a virtual network, a virtual machine, and a VPN gateway.</span></span> <span data-ttu-id="4e75a-209">La creación de la instancia de VPN Gateway puede durar más de cuarenta minutos.</span><span class="sxs-lookup"><span data-stu-id="4e75a-209">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="azure-hub-vnet"></a><span data-ttu-id="4e75a-210">Red virtual del concentrador de Azure</span><span class="sxs-lookup"><span data-stu-id="4e75a-210">Azure hub VNet</span></span>

<span data-ttu-id="4e75a-211">Para implementar la red virtual del concentrador y conectarla a la red virtual local simulada creada anteriormente, realice los pasos siguientes.</span><span class="sxs-lookup"><span data-stu-id="4e75a-211">To deploy the hub VNet, and connect to the simulated on-premises VNet created above, perform the following steps.</span></span>

1. <span data-ttu-id="4e75a-212">Abra el archivo `hub-vnet.json` y escriba un nombre de usuario y la contraseña entre comillas en las líneas 39 y 40, tal y como se muestra a continuación.</span><span class="sxs-lookup"><span data-stu-id="4e75a-212">Open the `hub-vnet.json` file and enter a username and password between the quotes in line 39 and 40, as shown below.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

2. <span data-ttu-id="4e75a-213">En la línea 41, para `osType`, escriba `Windows` o `Linux` para instalar Windows Server 2016 Datacenter o Ubuntu 16.04 como sistema operativo de JumpBox.</span><span class="sxs-lookup"><span data-stu-id="4e75a-213">On line 41, for `osType`, type `Windows` or `Linux` to install either Windows Server 2016 Datacenter, or Ubuntu 16.04 as the operating system for the jumpbox.</span></span>

3. <span data-ttu-id="4e75a-214">Escriba una clave compartida entre comillas en la línea 72, tal y como se muestra a continuación, y después guarde el archivo.</span><span class="sxs-lookup"><span data-stu-id="4e75a-214">Enter a shared key between the quotes in line 72, as shown below, then save the file.</span></span>

  ```bash
  "sharedKey": "",
  ```

4. <span data-ttu-id="4e75a-215">Ejecute `azbb` para implementar el entorno simulado local tal y como se muestra a continuación.</span><span class="sxs-lookup"><span data-stu-id="4e75a-215">Run `azbb` to deploy the simulated onprem environment as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g hub-vnet-rg - l <location> -p hub-vnet.json --deploy
  ```
  > [!NOTE]
  > <span data-ttu-id="4e75a-216">Si decide usar un nombre del grupo de recursos distinto (que no sea `hub-vnet-rg`), asegúrese de que busca todos los archivos de parámetros que usan ese nombre y de que los modifica para usar el nombre de su propio grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="4e75a-216">If you decide to use a different resource group name (other than `hub-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="4e75a-217">Espere a que finalice la implementación.</span><span class="sxs-lookup"><span data-stu-id="4e75a-217">Wait for the deployment to finish.</span></span> <span data-ttu-id="4e75a-218">Esta implementación crea una red virtual, una máquina virtual, una instancia de VPN Gateway y una conexión a la puerta de enlace creada en la sección anterior.</span><span class="sxs-lookup"><span data-stu-id="4e75a-218">This deployment creates a virtual network, a virtual machine, a VPN gateway, and a connection to the gateway created in the previous section.</span></span> <span data-ttu-id="4e75a-219">La creación de la instancia de VPN Gateway puede durar más de cuarenta minutos.</span><span class="sxs-lookup"><span data-stu-id="4e75a-219">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="optional-test-connectivity-from-onprem-to-hub"></a><span data-ttu-id="4e75a-220">(Opcional) Comprobación de la conectividad desde el entorno local al concentrador</span><span class="sxs-lookup"><span data-stu-id="4e75a-220">(Optional) Test connectivity from onprem to hub</span></span>

<span data-ttu-id="4e75a-221">Para probar la conectividad desde el entorno local simulado a la red virtual de concentrador mediante máquinas virtuales Windows, realice los pasos siguientes.</span><span class="sxs-lookup"><span data-stu-id="4e75a-221">To test conectivity from the simulated on-premises environment to the hub VNet using Windows VMs, perform the following steps.</span></span>

1. <span data-ttu-id="4e75a-222">En Azure Portal, vaya al grupo de recursos `onprem-jb-rg` y, a continuación, haga clic en el recurso de máquina virtual `jb-vm1`.</span><span class="sxs-lookup"><span data-stu-id="4e75a-222">From the Azure portal, navigate to the `onprem-jb-rg` resource group, then click on the `jb-vm1` virtual machine resource.</span></span>

2.  <span data-ttu-id="4e75a-223">En la esquina superior izquierda de la hoja VM del portal, haga clic en `Connect` y siga las indicaciones para utilizar Escritorio remoto para conectarse a la máquina virtual.</span><span class="sxs-lookup"><span data-stu-id="4e75a-223">On the top left hand corner of your VM blade in the portal, click `Connect`, and follow the prompts to use remote desktop to connect to the VM.</span></span> <span data-ttu-id="4e75a-224">Asegúrese de usar el nombre de usuario y la contraseña que especificó en las líneas 36 y 37 del archivo `onprem.json`.</span><span class="sxs-lookup"><span data-stu-id="4e75a-224">Make sure to use the username and password you specified in lines 36 and 37 in the `onprem.json` file.</span></span>

3. <span data-ttu-id="4e75a-225">Abra una consola de PowerShell en la máquina virtual y utilice el cmdlet `Test-NetConnection` para comprobar que puede conectarse a la máquina virtual de JumpBox del concentrador tal y como se muestra a continuación.</span><span class="sxs-lookup"><span data-stu-id="4e75a-225">Open a PowerShell console in the VM, and use the `Test-NetConnection` cmdlet to verify that you can connect to the hub jumpbox VM as shown below.</span></span>

  ```powershell
  Test-NetConnection 10.0.0.68 -CommonTCPPort RDP
  ```
  > [!NOTE]
  > <span data-ttu-id="4e75a-226">De forma predeterminada, las máquinas virtuales de Windows Server no permiten respuestas ICMP en Azure.</span><span class="sxs-lookup"><span data-stu-id="4e75a-226">By default, Windows Server VMs do not allow ICMP responses in Azure.</span></span> <span data-ttu-id="4e75a-227">Si desea usar `ping` para probar la conectividad, debe habilitar el tráfico ICMP en el firewall de Windows con seguridad avanzada para cada máquina virtual.</span><span class="sxs-lookup"><span data-stu-id="4e75a-227">If you want to use `ping` to test connectivity, you need to enable ICMP traffic in the Windows Advanced Firewall for each VM.</span></span>

<span data-ttu-id="4e75a-228">Para probar la conectividad desde el entorno local simulado a la red virtual de concentrador mediante máquinas virtuales Linux, realice los pasos siguientes:</span><span class="sxs-lookup"><span data-stu-id="4e75a-228">To test conectivity from the simulated on-premises environment to the hub VNet using Linux VMs, perform the following steps:</span></span>

1. <span data-ttu-id="4e75a-229">En Azure Portal, vaya al grupo de recursos `onprem-jb-rg` y, a continuación, haga clic en el recurso de máquina virtual `jb-vm1`.</span><span class="sxs-lookup"><span data-stu-id="4e75a-229">From the Azure portal, navigate to the `onprem-jb-rg` resource group, then click on the `jb-vm1` virtual machine resource.</span></span>

2. <span data-ttu-id="4e75a-230">En la esquina superior izquierda de la hoja VM del portal, haga clic en `Connect` y, a continuación, copie el comando `ssh` que aparece en el portal.</span><span class="sxs-lookup"><span data-stu-id="4e75a-230">On the top left hand corner of your VM blade in the portal, click `Connect`, and then copy the `ssh` command shown on the portal.</span></span> 

3. <span data-ttu-id="4e75a-231">Desde un símbolo del sistema de Linux, ejecute `ssh` para conectarse a la instancia de JumpBox del entorno local simulado con la información que copió en el paso 2 anterior, como se indica a continuación.</span><span class="sxs-lookup"><span data-stu-id="4e75a-231">From a Linux prompt, run `ssh` to connect to the simulated on-premises environment jumpbox witht the information you copied in step 2 above, as shown below.</span></span>

  ```bash
  ssh <your_user>@<public_ip_address>
  ```

4. <span data-ttu-id="4e75a-232">Use la contraseña que especificó en la línea 37 del archivo `onprem.json` para conectarse a la máquina virtual.</span><span class="sxs-lookup"><span data-stu-id="4e75a-232">Use the password you specified in line 37 in the `onprem.json` file to the connect to the VM.</span></span>

5. <span data-ttu-id="4e75a-233">Use el comando `ping` para probar la conectividad con JumpBox del concentrador tal y como se muestra a continuación.</span><span class="sxs-lookup"><span data-stu-id="4e75a-233">Use the `ping` command to test connectivity to the hub jumpbox, as shown below.</span></span>

  ```bash
  ping 10.0.0.68
  ```

### <a name="azure-spoke-vnets"></a><span data-ttu-id="4e75a-234">Redes virtuales de radios de Azure</span><span class="sxs-lookup"><span data-stu-id="4e75a-234">Azure spoke VNets</span></span>

<span data-ttu-id="4e75a-235">Para implementar las redes virtuales de radios, siga estos pasos.</span><span class="sxs-lookup"><span data-stu-id="4e75a-235">To deploy the spoke VNets, perform the following steps.</span></span>

1. <span data-ttu-id="4e75a-236">Abra el archivo `spoke1.json` y escriba un nombre de usuario y la contraseña entre comillas en las líneas 47 y 48, tal y como se muestra a continuación, y después guarde el archivo.</span><span class="sxs-lookup"><span data-stu-id="4e75a-236">Open the `spoke1.json` file and enter a username and password between the quotes in lines 47 and 48, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

2. <span data-ttu-id="4e75a-237">En la línea 49, para `osType`, escriba `Windows` o `Linux` para instalar Windows Server 2016 Datacenter o Ubuntu 16.04 como sistema operativo de JumpBox.</span><span class="sxs-lookup"><span data-stu-id="4e75a-237">On line 49, for `osType`, type `Windows` or `Linux` to install either Windows Server 2016 Datacenter, or Ubuntu 16.04 as the operating system for the jumpbox.</span></span>

3. <span data-ttu-id="4e75a-238">Ejecute `azbb` para implementar el entorno de red virtual del primer radio tal y como se muestra a continuación.</span><span class="sxs-lookup"><span data-stu-id="4e75a-238">Run `azbb` to deploy the first spoke VNet environment as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g spoke1-vnet-rg - l <location> -p spoke1.json --deploy
  ```
  
  > [!NOTE]
  > <span data-ttu-id="4e75a-239">Si decide usar un nombre del grupo de recursos distinto (que no sea `spoke1-vnet-rg`), asegúrese de que busca todos los archivos de parámetros que usan ese nombre y de que los modifica para usar el nombre de su propio grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="4e75a-239">If you decide to use a different resource group name (other than `spoke1-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

3. <span data-ttu-id="4e75a-240">Repita el paso 1 anterior para el archivo `spoke2.json`.</span><span class="sxs-lookup"><span data-stu-id="4e75a-240">Repeat step 1 above for file `spoke2.json`.</span></span>

4. <span data-ttu-id="4e75a-241">Ejecute `azbb` para implementar el entorno de red virtual del segundo radio tal y como se muestra a continuación.</span><span class="sxs-lookup"><span data-stu-id="4e75a-241">Run `azbb` to deploy the second spoke VNet environment as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g spoke2-vnet-rg - l <location> -p spoke2.json --deploy
  ```
  > [!NOTE]
  > <span data-ttu-id="4e75a-242">Si decide usar un nombre del grupo de recursos distinto (que no sea `spoke2-vnet-rg`), asegúrese de que busca todos los archivos de parámetros que usan ese nombre y de que los modifica para usar el nombre de su propio grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="4e75a-242">If you decide to use a different resource group name (other than `spoke2-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

### <a name="azure-hub-vnet-peering-to-spoke-vnets"></a><span data-ttu-id="4e75a-243">Emparejamiento de VNET del concentrador de Azure a las redes virtuales de los radios</span><span class="sxs-lookup"><span data-stu-id="4e75a-243">Azure hub VNet peering to spoke VNets</span></span>

<span data-ttu-id="4e75a-244">Para crear una conexión de emparejamiento desde la red virtual del concentrador a las redes virtuales de radios, realice los siguientes pasos.</span><span class="sxs-lookup"><span data-stu-id="4e75a-244">To create a peering connection from the hub VNet to the spoke VNets, perform the following steps.</span></span>

1. <span data-ttu-id="4e75a-245">Abra el archivo `hub-vnet-peering.json` y compruebe que el nombre del grupo de recursos y el nombre de la red virtual para cada uno de los emparejamientos de la red virtual que empiezan en la línea 29 son correctos.</span><span class="sxs-lookup"><span data-stu-id="4e75a-245">Open the `hub-vnet-peering.json` file and verify that the resource group name, and virtual network name for each of the virtual network peerings starting in line 29 are correct.</span></span>

2. <span data-ttu-id="4e75a-246">Ejecute `azbb` para implementar el entorno de red virtual del primer radio tal y como se muestra a continuación.</span><span class="sxs-lookup"><span data-stu-id="4e75a-246">Run `azbb` to deploy the first spoke VNet environment as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g hub-vnet-rg - l <location> -p hub-vnet-peering.json --deploy
  ```

  > [!NOTE]
  > <span data-ttu-id="4e75a-247">Si decide usar un nombre del grupo de recursos distinto (que no sea `hub-vnet-rg`), asegúrese de que busca todos los archivos de parámetros que usan ese nombre y de que los modifica para usar el nombre de su propio grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="4e75a-247">If you decide to use a different resource group name (other than `hub-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

### <a name="test-connectivity"></a><span data-ttu-id="4e75a-248">Comprobación de la conectividad</span><span class="sxs-lookup"><span data-stu-id="4e75a-248">Test connectivity</span></span>

<span data-ttu-id="4e75a-249">Para probar la conectividad desde el entorno local simulado a las redes virtuales de radios mediante máquinas virtuales Windows, realice los pasos siguientes.</span><span class="sxs-lookup"><span data-stu-id="4e75a-249">To test conectivity from the simulated on-premises environment to the spoke VNets using Windows VMs, perform the following steps.</span></span>

1. <span data-ttu-id="4e75a-250">En Azure Portal, vaya al grupo de recursos `onprem-jb-rg` y, a continuación, haga clic en el recurso de máquina virtual `jb-vm1`.</span><span class="sxs-lookup"><span data-stu-id="4e75a-250">From the Azure portal, navigate to the `onprem-jb-rg` resource group, then click on the `jb-vm1` virtual machine resource.</span></span>

2.  <span data-ttu-id="4e75a-251">En la esquina superior izquierda de la hoja VM del portal, haga clic en `Connect` y siga las indicaciones para utilizar Escritorio remoto para conectarse a la máquina virtual.</span><span class="sxs-lookup"><span data-stu-id="4e75a-251">On the top left hand corner of your VM blade in the portal, click `Connect`, and follow the prompts to use remote desktop to connect to the VM.</span></span> <span data-ttu-id="4e75a-252">Asegúrese de usar el nombre de usuario y la contraseña que especificó en las líneas 36 y 37 del archivo `onprem.json`.</span><span class="sxs-lookup"><span data-stu-id="4e75a-252">Make sure to use the username and password you specified in lines 36 and 37 in the `onprem.json` file.</span></span>

3. <span data-ttu-id="4e75a-253">Abra una consola de PowerShell en la máquina virtual y utilice el cmdlet `Test-NetConnection` para comprobar que puede conectarse a la máquina virtual de JumpBox del concentrador tal y como se muestra a continuación.</span><span class="sxs-lookup"><span data-stu-id="4e75a-253">Open a PowerShell console in the VM, and use the `Test-NetConnection` cmdlet to verify that you can connect to the hub jumpbox VM as shown below.</span></span>

  ```powershell
  Test-NetConnection 10.1.0.68 -CommonTCPPort RDP
  Test-NetConnection 10.2.0.68 -CommonTCPPort RDP
  ```

<span data-ttu-id="4e75a-254">Para probar la conectividad desde el entorno local simulado a las redes virtuales de radios mediante máquinas virtuales Linux, realice los pasos siguientes:</span><span class="sxs-lookup"><span data-stu-id="4e75a-254">To test conectivity from the simulated on-premises environment to the spoke VNets using Linux VMs, perform the following steps:</span></span>

1. <span data-ttu-id="4e75a-255">En Azure Portal, vaya al grupo de recursos `onprem-jb-rg` y, a continuación, haga clic en el recurso de máquina virtual `jb-vm1`.</span><span class="sxs-lookup"><span data-stu-id="4e75a-255">From the Azure portal, navigate to the `onprem-jb-rg` resource group, then click on the `jb-vm1` virtual machine resource.</span></span>

2. <span data-ttu-id="4e75a-256">En la esquina superior izquierda de la hoja VM del portal, haga clic en `Connect` y, a continuación, copie el comando `ssh` que aparece en el portal.</span><span class="sxs-lookup"><span data-stu-id="4e75a-256">On the top left hand corner of your VM blade in the portal, click `Connect`, and then copy the `ssh` command shown on the portal.</span></span> 

3. <span data-ttu-id="4e75a-257">Desde un símbolo del sistema de Linux, ejecute `ssh` para conectarse a la instancia de JumpBox del entorno local simulado con la información que copió en el paso 2 anterior, como se indica a continuación.</span><span class="sxs-lookup"><span data-stu-id="4e75a-257">From a Linux prompt, run `ssh` to connect to the simulated on-premises environment jumpbox witht the information you copied in step 2 above, as shown below.</span></span>

  ```bash
  ssh <your_user>@<public_ip_address>
  ```

5. <span data-ttu-id="4e75a-258">Use la contraseña que especificó en la línea 37 del archivo `onprem.json` para conectarse a la máquina virtual.</span><span class="sxs-lookup"><span data-stu-id="4e75a-258">Use the password you specified in line 37 in the `onprem.json` file to the connect to the VM.</span></span>

6. <span data-ttu-id="4e75a-259">Use el comando `ping` para probar la conectividad con las máquinas virtuales de JumpBox en cada radio tal y como se muestra a continuación.</span><span class="sxs-lookup"><span data-stu-id="4e75a-259">Use the `ping` command to test connectivity to the jumpbox VMs in each spoke, as shown below.</span></span>

  ```bash
  ping 10.1.0.68
  ping 10.2.0.68
  ```

### <a name="add-connectivity-between-spokes"></a><span data-ttu-id="4e75a-260">Adición de conectividad entre radios</span><span class="sxs-lookup"><span data-stu-id="4e75a-260">Add connectivity between spokes</span></span>

<span data-ttu-id="4e75a-261">Si desea permitir que los radios se conecten unos con otros, deberá usar una aplicación virtual de red (NVA) como enrutador de la red virtual de concentrador y forzar el tráfico desde los radios al enrutador al intentar conectarse a otro radio.</span><span class="sxs-lookup"><span data-stu-id="4e75a-261">If you want to allow spokes to connect to each other, you need to use a newtwork virtual appliance (NVA) as a router in the hub virtual netowrk, and force traffic from spokes to the router when trying to connect to another spoke.</span></span> <span data-ttu-id="4e75a-262">Para implementar una aplicación virtual de red de ejemplo básica como máquina virtual y las rutas necesarias definidas por el usuario para permitir que las dos redes virtuales de radios se conecten, realice los pasos siguientes:</span><span class="sxs-lookup"><span data-stu-id="4e75a-262">To deploy a basic sample NVA as a single VM, and the necessary uder defined routes to allow the two spoke VNets to connect, perform the following steps:</span></span>

1. <span data-ttu-id="4e75a-263">Abra el archivo `hub-nva.json` y escriba un nombre de usuario y la contraseña entre comillas en las líneas 13 y 14, tal y como se muestra a continuación, y después guarde el archivo.</span><span class="sxs-lookup"><span data-stu-id="4e75a-263">Open the `hub-nva.json` file and enter a username and password between the quotes in lines 13 and 14, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```
2. <span data-ttu-id="4e75a-264">Ejecute `azbb` para implementar la máquina virtual de la aplicación virtual de red y las rutas definidas por el usuario.</span><span class="sxs-lookup"><span data-stu-id="4e75a-264">Run `azbb` to deploy the NVA VM and user defined routes.</span></span>

  ```bash
  azbb -s <subscription_id> -g hub-nva-rg - l <location> -p hub-nva.json --deploy
  ```
  > [!NOTE]
  > <span data-ttu-id="4e75a-265">Si decide usar un nombre del grupo de recursos distinto (que no sea `hub-nva-rg`), asegúrese de que busca todos los archivos de parámetros que usan ese nombre y de que los modifica para usar el nombre de su propio grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="4e75a-265">If you decide to use a different resource group name (other than `hub-nva-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

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
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[windows-vm-ra]: ../virtual-machines-windows/index.md

[visio-download]: https://archcenter.azureedge.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/hub-spoke.png "Topología en estrella tipo hub-and-spoke en Azure"
[1]: ./images/hub-spoke-gateway-routing.svg "Topología en estrella tipo hub-and-spoke en Azure con enrutamiento transitivo"
[2]: ./images/hub-spoke-no-gateway-routing.svg "Topología en estrella tipo hub-and-spoke en Azure con enrutamiento transitivo mediante una NVA"
[3]: ./images/hub-spokehub-spoke.svg "Topología en estrella tipo hub-hub-and-spoke-spoke en Azure"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
