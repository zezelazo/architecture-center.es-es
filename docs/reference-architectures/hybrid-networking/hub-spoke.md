---
title: "Implementación de una topología de red en estrella tipo hub-and-spoke en Azure"
description: "Cómo implementar una topología de red en estrella tipo hub-and-spoke en Azure."
author: telmosampaio
ms.date: 02/14/2018
pnp.series.title: Implement a hub-spoke network topology in Azure
pnp.series.prev: expressroute
ms.openlocfilehash: c03ecd4ba5ddbe50cfb17e56d75c18102b751cfb
ms.sourcegitcommit: 475064f0a3c2fac23e1286ba159aaded287eec86
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/19/2018
---
# <a name="implement-a-hub-spoke-network-topology-in-azure"></a><span data-ttu-id="32770-103">Implementación de una topología de red en estrella tipo hub-and-spoke en Azure</span><span class="sxs-lookup"><span data-stu-id="32770-103">Implement a hub-spoke network topology in Azure</span></span>

<span data-ttu-id="32770-104">En esta arquitectura de referencia se muestra cómo implementar una topología en estrella tipo hub-and-spoke en Azure.</span><span class="sxs-lookup"><span data-stu-id="32770-104">This reference architecture shows how to implement a hub-spoke topology in Azure.</span></span> <span data-ttu-id="32770-105">El *concentrador* (hub) es una red virtual (VNet) en Azure que actúa como un punto central de conectividad para la red local.</span><span class="sxs-lookup"><span data-stu-id="32770-105">The *hub* is a virtual network (VNet) in Azure that acts as a central point of connectivity to your on-premises network.</span></span> <span data-ttu-id="32770-106">Los *radios* (spoke) son redes virtuales (VNet) que se emparejan con el concentrador y que se pueden usar para aislar cargas de trabajo.</span><span class="sxs-lookup"><span data-stu-id="32770-106">The *spokes* are VNets that peer with the hub, and can be used to isolate workloads.</span></span> <span data-ttu-id="32770-107">El tráfico fluye entre el centro de datos local y el concentrador a través de una conexión a ExpressRoute o a VPN Gateway.</span><span class="sxs-lookup"><span data-stu-id="32770-107">Traffic flows between the on-premises datacenter and the hub through an ExpressRoute or VPN gateway connection.</span></span>  <span data-ttu-id="32770-108">[**Implemente esta solución**](#deploy-the-solution).</span><span class="sxs-lookup"><span data-stu-id="32770-108">[**Deploy this solution**](#deploy-the-solution).</span></span>

<span data-ttu-id="32770-109">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="32770-109">![[0]][0]</span></span>

<span data-ttu-id="32770-110">*Descargue un [archivo Visio][visio-download] de esta arquitectura.*</span><span class="sxs-lookup"><span data-stu-id="32770-110">*Download a [Visio file][visio-download] of this architecture*</span></span>


<span data-ttu-id="32770-111">Las ventajas de esta topología son:</span><span class="sxs-lookup"><span data-stu-id="32770-111">The benefits of this toplogy include:</span></span>

* <span data-ttu-id="32770-112">**Ahorros en costos** gracias a la centralización de los servicios que se pueden compartir entre varias cargas de trabajo, como las aplicaciones virtuales de red (NVA) y los servidores DNS, en una única ubicación.</span><span class="sxs-lookup"><span data-stu-id="32770-112">**Cost savings** by centralizing services that can be shared by multiple workloads, such as network virtual appliances (NVAs) and DNS servers, in a single location.</span></span>
* <span data-ttu-id="32770-113">**Superar los límites de las suscripciones** gracias al emparejamiento de las redes virtuales de diferentes suscripciones con el concentrador central.</span><span class="sxs-lookup"><span data-stu-id="32770-113">**Overcome subscriptions limits** by peering VNets from different subscriptions to the central hub.</span></span>
* <span data-ttu-id="32770-114">**Separación de intereses** entre la TI central (SecOps e InfraOps) y las cargas de trabajo (DevOps).</span><span class="sxs-lookup"><span data-stu-id="32770-114">**Separation of concerns** between central IT (SecOps, InfraOps) and workloads (DevOps).</span></span>

<span data-ttu-id="32770-115">Los usos habituales de esta arquitectura incluyen:</span><span class="sxs-lookup"><span data-stu-id="32770-115">Typical uses for this architecture include:</span></span>

* <span data-ttu-id="32770-116">Cargas de trabajo implementadas en distintos entornos, como desarrollo, pruebas y producción, que requieren servicios compartidos como DNS, IDS, NTP o AD DS.</span><span class="sxs-lookup"><span data-stu-id="32770-116">Workloads deployed in different environments, such as development, testing, and production, that require shared services such as DNS, IDS, NTP, or AD DS.</span></span> <span data-ttu-id="32770-117">Los servicios compartidos se colocan en la red virtual del concentrador, mientras que cada entorno se implementa en un radio para mantener el aislamiento.</span><span class="sxs-lookup"><span data-stu-id="32770-117">Shared services are placed in the hub VNet, while each environment is deployed to a spoke to maintain isolation.</span></span>
* <span data-ttu-id="32770-118">Cargas de trabajo que no requieren conectividad entre sí, pero requieren acceso a los servicios compartidos.</span><span class="sxs-lookup"><span data-stu-id="32770-118">Workloads that do not require connectivity to each other, but require access to shared services.</span></span>
* <span data-ttu-id="32770-119">Empresas que requieren el control centralizado sobre aspectos de seguridad, como un firewall en el concentrador como una red perimetral, así como la administración segregada de las cargas de trabajo en cada radio.</span><span class="sxs-lookup"><span data-stu-id="32770-119">Enterprises that require central control over security aspects, such as a firewall in the hub as a DMZ, and segregated management for the workloads in each spoke.</span></span>

## <a name="architecture"></a><span data-ttu-id="32770-120">Architecture</span><span class="sxs-lookup"><span data-stu-id="32770-120">Architecture</span></span>

<span data-ttu-id="32770-121">La arquitectura consta de los siguientes componentes:</span><span class="sxs-lookup"><span data-stu-id="32770-121">The architecture consists of the following components.</span></span>

* <span data-ttu-id="32770-122">**Red local**.</span><span class="sxs-lookup"><span data-stu-id="32770-122">**On-premises network**.</span></span> <span data-ttu-id="32770-123">Una red de área local privada que se ejecuta dentro de una organización.</span><span class="sxs-lookup"><span data-stu-id="32770-123">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="32770-124">**Dispositivo VPN**.</span><span class="sxs-lookup"><span data-stu-id="32770-124">**VPN device**.</span></span> <span data-ttu-id="32770-125">Un dispositivo o servicio que proporciona conectividad externa a la red local.</span><span class="sxs-lookup"><span data-stu-id="32770-125">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="32770-126">El dispositivo VPN puede ser un dispositivo de hardware, o puede ser una solución de software como el servicio de Enrutamiento y acceso remoto (RRAS) en Windows Server 2012.</span><span class="sxs-lookup"><span data-stu-id="32770-126">The VPN device may be a hardware device, or a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="32770-127">Para obtener una lista de dispositivos VPN compatibles e información acerca de cómo configurar dispositivos VPN seleccionados para conectarse a Azure, consulte [Acerca de los dispositivos VPN y los parámetros de IPsec/IKE para conexiones de VPN Gateway de sitio a sitio][vpn-appliance].</span><span class="sxs-lookup"><span data-stu-id="32770-127">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="32770-128">**Puerta de enlace de red virtual de VPN o puerta de enlace de ExpressRoute**.</span><span class="sxs-lookup"><span data-stu-id="32770-128">**VPN virtual network gateway or ExpressRoute gateway**.</span></span> <span data-ttu-id="32770-129">La puerta de enlace de red virtual permite que la red virtual se conecte al dispositivo VPN o al circuito de ExpressRoute que se usa para la conectividad con la red local.</span><span class="sxs-lookup"><span data-stu-id="32770-129">The virtual network gateway enables the VNet to connect to the VPN device, or ExpressRoute circuit, used for connectivity with your on-premises network.</span></span> <span data-ttu-id="32770-130">Para más información, consulte [Conectar una red local con una red virtual de Microsoft Azure][connect-to-an-Azure-vnet].</span><span class="sxs-lookup"><span data-stu-id="32770-130">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

> [!NOTE]
> <span data-ttu-id="32770-131">Los scripts de implementación para esta arquitectura de referencia usan una instancia de VPN Gateway para la conectividad y una red virtual en Azure para simular la red local.</span><span class="sxs-lookup"><span data-stu-id="32770-131">The deployment scripts for this reference architecture use a VPN gateway for connectivity, and a VNet in Azure to simulate your on-premises network.</span></span>

* <span data-ttu-id="32770-132">**Red virtual del concentrador**.</span><span class="sxs-lookup"><span data-stu-id="32770-132">**Hub VNet**.</span></span> <span data-ttu-id="32770-133">La red virtual de Azure utilizada como el concentrador en la topología en estrella tipo hub-and-spoke.</span><span class="sxs-lookup"><span data-stu-id="32770-133">Azure VNet used as the hub in the hub-spoke topology.</span></span> <span data-ttu-id="32770-134">El concentrador es el punto central de conectividad a la red local y un lugar para hospedar los servicios que se pueden usar en las diferentes cargas de trabajo hospedadas en las redes virtuales de los radios.</span><span class="sxs-lookup"><span data-stu-id="32770-134">The hub is the central point of connectivity to your on-premises network, and a place to host services that can be consumed by the different workloads hosted in the spoke VNets.</span></span>

* <span data-ttu-id="32770-135">**Subred de puerta de enlace**.</span><span class="sxs-lookup"><span data-stu-id="32770-135">**Gateway subnet**.</span></span> <span data-ttu-id="32770-136">Las puertas de enlace de red virtual se conservan en la misma subred.</span><span class="sxs-lookup"><span data-stu-id="32770-136">The virtual network gateways are held in the same subnet.</span></span>

* <span data-ttu-id="32770-137">**Subred de servicios compartidos**.</span><span class="sxs-lookup"><span data-stu-id="32770-137">**Shared services subnet**.</span></span> <span data-ttu-id="32770-138">Una subred de la red virtual del concentrador utilizada para hospedar servicios que se pueden compartir entre todos los radios, como DNS o AD DS.</span><span class="sxs-lookup"><span data-stu-id="32770-138">A subnet in the hub VNet used to host services that can be shared among all spokes, such as DNS or AD DS.</span></span>

* <span data-ttu-id="32770-139">**Redes virtuales de radios**.</span><span class="sxs-lookup"><span data-stu-id="32770-139">**Spoke VNets**.</span></span> <span data-ttu-id="32770-140">Una o varias redes virtuales de Azure que se usan como radios en la topología en estrella tipo hub-and-spoke.</span><span class="sxs-lookup"><span data-stu-id="32770-140">One or more Azure VNets that are used as spokes in the hub-spoke topology.</span></span> <span data-ttu-id="32770-141">Los radios pueden utilizarse para aislar las cargas de trabajo en sus propias redes virtuales, administradas por separado desde otros radios.</span><span class="sxs-lookup"><span data-stu-id="32770-141">Spokes can be used to isolate workloads in their own VNets, managed separately from other spokes.</span></span> <span data-ttu-id="32770-142">Cada carga de trabajo puede incluir varios niveles, con varias subredes que se conectan a través de equilibradores de carga de Azure.</span><span class="sxs-lookup"><span data-stu-id="32770-142">Each workload might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="32770-143">Para obtener más información acerca de la infraestructura de aplicaciones, consulte [Ejecutar cargas de trabajo de máquinas virtuales Windows][windows-vm-ra] y [Ejecutar cargas de trabajo de máquinas virtuales Linux][linux-vm-ra].</span><span class="sxs-lookup"><span data-stu-id="32770-143">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

* <span data-ttu-id="32770-144">**Emparejamiento de VNET**.</span><span class="sxs-lookup"><span data-stu-id="32770-144">**VNet peering**.</span></span> <span data-ttu-id="32770-145">Se pueden conectar dos redes virtuales en la misma región de Azure mediante una [conexión de emparejamiento][vnet-peering].</span><span class="sxs-lookup"><span data-stu-id="32770-145">Two VNets in the same Azure region can be connected using a [peering connection][vnet-peering].</span></span> <span data-ttu-id="32770-146">Las conexiones de emparejamiento son conexiones no transitivas de baja latencia entre las redes virtuales.</span><span class="sxs-lookup"><span data-stu-id="32770-146">Peering connections are non-transitive, low latency connections between VNets.</span></span> <span data-ttu-id="32770-147">Una vez establecido el emparejamiento, las redes virtuales intercambian el tráfico mediante la red troncal de Azure, sin necesidad de un enrutador.</span><span class="sxs-lookup"><span data-stu-id="32770-147">Once peered, the VNets exchange traffic by using the Azure backbone, without the need for a router.</span></span> <span data-ttu-id="32770-148">En una topología de red en estrella tipo hub-and-spoke, el emparejamiento de VNET se usa para conectar el concentrador a cada radio.</span><span class="sxs-lookup"><span data-stu-id="32770-148">In a hub-spoke network topology, you use VNet peering to connect the hub to each spoke.</span></span>

> [!NOTE]
> <span data-ttu-id="32770-149">En este artículo solo se tratan las implementaciones de [Resource Manager](/azure/azure-resource-manager/resource-group-overview), pero también puede conectar una red virtual clásica a una red virtual de Resource Manager en la misma suscripción.</span><span class="sxs-lookup"><span data-stu-id="32770-149">This article only covers [Resource Manager](/azure/azure-resource-manager/resource-group-overview) deployments, but you can also connect a classic VNet to a Resource Manager VNet in the same subscription.</span></span> <span data-ttu-id="32770-150">De este modo, los radios pueden hospedar implementaciones clásicas y seguir beneficiándose de los servicios compartidos en el concentrador.</span><span class="sxs-lookup"><span data-stu-id="32770-150">That way, your spokes can host classic deployments and still benefit from services shared in the hub.</span></span>


## <a name="recommendations"></a><span data-ttu-id="32770-151">Recomendaciones</span><span class="sxs-lookup"><span data-stu-id="32770-151">Recommendations</span></span>

<span data-ttu-id="32770-152">Las siguientes recomendaciones sirven para la mayoría de los escenarios.</span><span class="sxs-lookup"><span data-stu-id="32770-152">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="32770-153">Sígalas a menos que tenga un requisito concreto que las invalide.</span><span class="sxs-lookup"><span data-stu-id="32770-153">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="resource-groups"></a><span data-ttu-id="32770-154">Grupos de recursos</span><span class="sxs-lookup"><span data-stu-id="32770-154">Resource groups</span></span>

<span data-ttu-id="32770-155">La red virtual del concentrador y la red virtual de cada radio se pueden implementar en diferentes grupos de recursos e incluso en distintas suscripciones, siempre y cuando pertenezcan al mismo inquilino de Azure Active Directory (Azure AD) en la misma región.</span><span class="sxs-lookup"><span data-stu-id="32770-155">The hub VNet, and each spoke VNet, can be implemented in different resource groups, and even different subscriptions, as long as they belong to the same Azure Active Directory (Azure AD) tenant in the same Azure region.</span></span> <span data-ttu-id="32770-156">Esto permite realizar una administración descentralizada de cada carga de trabajo, mientras se comparten los servicios que se mantienen en la red virtual del concentrador.</span><span class="sxs-lookup"><span data-stu-id="32770-156">This allows for a decentralized management of each workload, while sharing services maintained in the hub VNet.</span></span>

### <a name="vnet-and-gatewaysubnet"></a><span data-ttu-id="32770-157">VNet y GatewaySubnet</span><span class="sxs-lookup"><span data-stu-id="32770-157">VNet and GatewaySubnet</span></span>

<span data-ttu-id="32770-158">Cree una subred denominada *GatewaySubnet*, con un intervalo de direcciones de /27.</span><span class="sxs-lookup"><span data-stu-id="32770-158">Create a subnet named *GatewaySubnet*, with an address range of /27.</span></span> <span data-ttu-id="32770-159">Esta subred es necesaria para la puerta de enlace de red virtual.</span><span class="sxs-lookup"><span data-stu-id="32770-159">This subnet is required by the virtual network gateway.</span></span> <span data-ttu-id="32770-160">La asignación de treinta y dos direcciones a esta subred ayudará a evitar que se alcancen las limitaciones de tamaño de la puerta de enlace en el futuro.</span><span class="sxs-lookup"><span data-stu-id="32770-160">Allocating 32 addresses to this subnet will help to prevent reaching gateway size limitations in the future.</span></span>

<span data-ttu-id="32770-161">Para más información sobre la configuración de la puerta de enlace, consulte las siguientes arquitecturas de referencia, en función del tipo de conexión:</span><span class="sxs-lookup"><span data-stu-id="32770-161">For more information about setting up the gateway, see the following reference architectures, depending on your connection type:</span></span>

- <span data-ttu-id="32770-162">[Red híbrida mediante ExpressRoute][guidance-expressroute]</span><span class="sxs-lookup"><span data-stu-id="32770-162">[Hybrid network using ExpressRoute][guidance-expressroute]</span></span>
- <span data-ttu-id="32770-163">[Red híbrida con una instancia de VPN Gateway][guidance-vpn]</span><span class="sxs-lookup"><span data-stu-id="32770-163">[Hybrid network using a VPN gateway][guidance-vpn]</span></span>

<span data-ttu-id="32770-164">Para una mayor disponibilidad, puede utilizar ExpressRoute y una VPN para la conmutación por error.</span><span class="sxs-lookup"><span data-stu-id="32770-164">For higher availability, you can use ExpressRoute plus a VPN for failover.</span></span> <span data-ttu-id="32770-165">Vea [Conexión de una red local a Azure mediante ExpressRoute con conmutación por error de VPN][hybrid-ha].</span><span class="sxs-lookup"><span data-stu-id="32770-165">See [Connect an on-premises network to Azure using ExpressRoute with VPN failover][hybrid-ha].</span></span>

<span data-ttu-id="32770-166">También se puede utilizar una topología en estrella tipo hub-and-spoke sin una puerta de enlace, en caso de que no se necesite la conectividad con la red local.</span><span class="sxs-lookup"><span data-stu-id="32770-166">A hub-spoke topology can also be used without a gateway, if you don't need connectivity with your on-premises network.</span></span> 

### <a name="vnet-peering"></a><span data-ttu-id="32770-167">Emparejamiento de VNET</span><span class="sxs-lookup"><span data-stu-id="32770-167">VNet peering</span></span>

<span data-ttu-id="32770-168">El emparejamiento de VNET es una relación no transitiva entre dos redes virtuales.</span><span class="sxs-lookup"><span data-stu-id="32770-168">VNet peering is a non-transitive relationship between two VNets.</span></span> <span data-ttu-id="32770-169">Si necesita que los radios se conecten entre sí, considere la posibilidad de agregar una conexión de emparejamiento independiente entre dichos radios.</span><span class="sxs-lookup"><span data-stu-id="32770-169">If you require spokes to connect to each other, consider adding a separate peering connection between those spokes.</span></span>

<span data-ttu-id="32770-170">Sin embargo, si tiene varios radios que necesitan conectarse entre sí, se quedará sin posibles conexiones de emparejamiento muy rápido debido a la [limitación del número de emparejamientos de VNET por cada red virtual][vnet-peering-limit].</span><span class="sxs-lookup"><span data-stu-id="32770-170">However, if you have several spokes that need to connect with each other, you will run out of possible peering connections very quickly due to the [limitation on number of VNets peerings per VNet][vnet-peering-limit].</span></span> <span data-ttu-id="32770-171">En este escenario, considere la posibilidad de usar rutas definidas por el usuario (UDR) para forzar que el tráfico destinado a un radio se envíe a una NVA que actúa como un enrutador en la red virtual del concentrador.</span><span class="sxs-lookup"><span data-stu-id="32770-171">In this scenario, consider using user defined routes (UDRs) to force traffic destined to a spoke to be sent to an NVA acting as a router at the hub VNet.</span></span> <span data-ttu-id="32770-172">Esto permitirá que los radios se conecten entre sí.</span><span class="sxs-lookup"><span data-stu-id="32770-172">This will allow the spokes to connect to each other.</span></span>

<span data-ttu-id="32770-173">También puede configurar los radios para que usen la puerta de enlace de la red virtual del concentrador para comunicarse con las redes remotas.</span><span class="sxs-lookup"><span data-stu-id="32770-173">You can also configure spokes to use the hub VNet gateway to communicate with remote networks.</span></span> <span data-ttu-id="32770-174">Para permitir que el tráfico de la puerta de enlace fluya del radio al concentrador y para conectarse a las redes remotas, debe:</span><span class="sxs-lookup"><span data-stu-id="32770-174">To allow gateway traffic to flow from spoke to hub, and connect to remote networks, you must:</span></span>

  - <span data-ttu-id="32770-175">Configurar la conexión de emparejamiento de VNET en el concentrador para **permitir el tránsito de la puerta de enlace**.</span><span class="sxs-lookup"><span data-stu-id="32770-175">Configure the VNet peering connection in the hub to **allow gateway transit**.</span></span>
  - <span data-ttu-id="32770-176">Configurar la conexión de emparejamiento de VNET en cada radio para **usar las puertas de enlace remotas**.</span><span class="sxs-lookup"><span data-stu-id="32770-176">Configure the VNet peering connection in each spoke to **use remote gateways**.</span></span>
  - <span data-ttu-id="32770-177">Configurar todas las conexiones de emparejamiento de VNET para **permitir el tráfico reenviado**.</span><span class="sxs-lookup"><span data-stu-id="32770-177">Configure all VNet peering connections to **allow forwarded traffic**.</span></span>

## <a name="considerations"></a><span data-ttu-id="32770-178">Consideraciones</span><span class="sxs-lookup"><span data-stu-id="32770-178">Considerations</span></span>

### <a name="spoke-connectivity"></a><span data-ttu-id="32770-179">Conectividad de los radios</span><span class="sxs-lookup"><span data-stu-id="32770-179">Spoke connectivity</span></span>

<span data-ttu-id="32770-180">Si se necesita una conectividad entre los radios, considere la posibilidad de implementar una NVA para el enrutamiento en el concentrador y de usar las UDR en el radio para reenviar el tráfico al concentrador.</span><span class="sxs-lookup"><span data-stu-id="32770-180">If you require connectivity between spokes, consider implementing an NVA for routing in the hub, and using UDRs in the spoke to forward traffic to the hub.</span></span>

<span data-ttu-id="32770-181">![[2]][2]</span><span class="sxs-lookup"><span data-stu-id="32770-181">![[2]][2]</span></span>

<span data-ttu-id="32770-182">En este escenario, debe configurar las conexiones de emparejamiento para que **permitan el tráfico reenviado**.</span><span class="sxs-lookup"><span data-stu-id="32770-182">In this scenario, you must configure the peering connections to **allow forwarded traffic**.</span></span>

### <a name="overcoming-vnet-peering-limits"></a><span data-ttu-id="32770-183">Superación de los límites de emparejamiento de VNET</span><span class="sxs-lookup"><span data-stu-id="32770-183">Overcoming VNet peering limits</span></span>

<span data-ttu-id="32770-184">Asegúrese de tener en cuenta la [limitación del número de emparejamientos de VNET por cada red virtual][vnet-peering-limit] en Azure.</span><span class="sxs-lookup"><span data-stu-id="32770-184">Make sure you consider the [limitation on number of VNets peerings per VNet][vnet-peering-limit] in Azure.</span></span> <span data-ttu-id="32770-185">Si decide que necesita más radios de los que el límite permitirá, considere la posibilidad de crear una topología en estrella tipo hub-hub-and-spoke-spoke, donde el primer nivel de radios también actúa como concentradores.</span><span class="sxs-lookup"><span data-stu-id="32770-185">If you decide you need more spokes than the limit will allow, consider creating a hub-spoke-hub-spoke topology, where the first level of spokes also act as hubs.</span></span> <span data-ttu-id="32770-186">En el siguiente diagrama se ilustra este enfoque.</span><span class="sxs-lookup"><span data-stu-id="32770-186">The following diagram shows this approach.</span></span>

<span data-ttu-id="32770-187">![[3]][3]</span><span class="sxs-lookup"><span data-stu-id="32770-187">![[3]][3]</span></span>

<span data-ttu-id="32770-188">También tenga en cuenta qué servicios se comparten en el concentrador, para asegurarse de que el concentrador se escala para tener un número mayor de radios.</span><span class="sxs-lookup"><span data-stu-id="32770-188">Also consider what services are shared in the hub, to ensure the hub scales for a larger number of spokes.</span></span> <span data-ttu-id="32770-189">Por ejemplo, si el concentrador proporciona servicios de firewall, tenga en cuenta los límites de ancho de banda de la solución de firewall al agregar varios radios.</span><span class="sxs-lookup"><span data-stu-id="32770-189">For instance, if your hub provides firewall services, consider the bandwidth limits of your firewall solution when adding multiple spokes.</span></span> <span data-ttu-id="32770-190">Puede mover algunos de estos servicios compartidos a un segundo nivel de concentradores.</span><span class="sxs-lookup"><span data-stu-id="32770-190">You might want to move some of these shared services to a second level of hubs.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="32770-191">Implementación de la solución</span><span class="sxs-lookup"><span data-stu-id="32770-191">Deploy the solution</span></span>

<span data-ttu-id="32770-192">Hay disponible una implementación de esta arquitectura en [GitHub][ref-arch-repo].</span><span class="sxs-lookup"><span data-stu-id="32770-192">A deployment for this architecture is available on [GitHub][ref-arch-repo].</span></span> <span data-ttu-id="32770-193">Usa máquinas virtuales Ubuntu en cada red virtual para probar la conectividad.</span><span class="sxs-lookup"><span data-stu-id="32770-193">It uses Ubuntu VMs in each VNet to test connectivity.</span></span> <span data-ttu-id="32770-194">No hay ningún servicio real hospedado en la subred de **servicios compartidos** de la **red virtual del concentrador**.</span><span class="sxs-lookup"><span data-stu-id="32770-194">There are no actual services hosted in the **shared-services** subnet in the **hub VNet**.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="32770-195">requisitos previos</span><span class="sxs-lookup"><span data-stu-id="32770-195">Prerequisites</span></span>

<span data-ttu-id="32770-196">Antes de poder implementar la arquitectura de referencia en su propia suscripción, debe realizar los pasos siguientes.</span><span class="sxs-lookup"><span data-stu-id="32770-196">Before you can deploy the reference architecture to your own subscription, you must perform the following steps.</span></span>

1. <span data-ttu-id="32770-197">Clone, bifurque o descargue el archivo ZIP para el repositorio de GitHub de [arquitecturas de referencia de AzureCAT][ref-arch-repo].</span><span class="sxs-lookup"><span data-stu-id="32770-197">Clone, fork, or download the zip file for the [AzureCAT reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="32770-198">Si prefiere usar la CLI de Azure, asegúrese de que tiene la CLI de Azure 2.0 instalada en el equipo.</span><span class="sxs-lookup"><span data-stu-id="32770-198">If you prefer to use the Azure CLI, make sure you have the Azure CLI 2.0 installed on your computer.</span></span> <span data-ttu-id="32770-199">Para instalar la CLI, siga las instrucciones de [Instalación de la CLI de Azure 2.0][azure-cli-2].</span><span class="sxs-lookup"><span data-stu-id="32770-199">To install the CLI, follow the instructions in [Install Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="32770-200">Si prefiere usar PowerShell, asegúrese de que tiene el último módulo de PowerShell para Azure instalado en el equipo.</span><span class="sxs-lookup"><span data-stu-id="32770-200">If you prefer to use PowerShell, make sure you have the latest PowerShell module for Azure installed on you computer.</span></span> <span data-ttu-id="32770-201">Para instalar el último módulo de Azure PowerShell, siga las instrucciones descritas en [Instalación de PowerShell para Azure][azure-powershell].</span><span class="sxs-lookup"><span data-stu-id="32770-201">To install the latest Azure PowerShell module, follow the instructions in [Install PowerShell for Azure][azure-powershell].</span></span>

4. <span data-ttu-id="32770-202">Desde un símbolo del sistema, un símbolo del sistema de Bash o un símbolo del sistema de PowerShell, inicie sesión en la cuenta de Azure con alguno de los comandos siguientes y siga las indicaciones.</span><span class="sxs-lookup"><span data-stu-id="32770-202">From a command prompt, bash prompt, or PowerShell prompt, login to your Azure account by using one of the commands below, and follow the prompts.</span></span>

  ```bash
  az login
  ```

  ```powershell
  Login-AzureRmAccount
  ```

### <a name="deploy-the-simulated-on-premises-datacenter"></a><span data-ttu-id="32770-203">Implementación del centro de datos local simulado</span><span class="sxs-lookup"><span data-stu-id="32770-203">Deploy the simulated on-premises datacenter</span></span>

<span data-ttu-id="32770-204">Para implementar el centro de datos local simulado como una red virtual de Azure, realice los pasos siguientes.</span><span class="sxs-lookup"><span data-stu-id="32770-204">To deploy the simulated on-premises datacenter as an Azure VNet, perform the following steps.</span></span>

1. <span data-ttu-id="32770-205">Navegue hasta la carpeta `hybrid-networking\hub-spoke\onprem` del repositorio que descargó en el paso de requisitos previos anterior.</span><span class="sxs-lookup"><span data-stu-id="32770-205">Navigate to the `hybrid-networking\hub-spoke\onprem` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="32770-206">Abra el archivo `onprem.vm.parameters.json`, escriba un nombre de usuario y la contraseña entre comillas en las líneas 11 y 12, tal y como se muestra a continuación, y después guarde el archivo.</span><span class="sxs-lookup"><span data-stu-id="32770-206">Open the `onprem.vm.parameters.json` file and enter a username and password between the quotes in line 11 and 12, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. <span data-ttu-id="32770-207">Ejecute el comando de Bash o de PowerShell siguiente para implementar el entorno local simulado como una red virtual en Azure.</span><span class="sxs-lookup"><span data-stu-id="32770-207">Run the bash or PowerShell command below to deploy the simulated on-premises environment as a VNet in Azure.</span></span> <span data-ttu-id="32770-208">Sustituya los valores con su suscripción, el nombre del grupo de recursos y la región de Azure.</span><span class="sxs-lookup"><span data-stu-id="32770-208">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

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
  > <span data-ttu-id="32770-209">Si decide usar un nombre del grupo de recursos distinto (que no sea `ra-onprem-rg`), asegúrese de que busca todos los archivos de parámetros que usan ese nombre y de que los modifica para usar el nombre de su propio grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="32770-209">If you decide to use a different resource group name (other than `ra-onprem-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

4. <span data-ttu-id="32770-210">Espere a que finalice la implementación.</span><span class="sxs-lookup"><span data-stu-id="32770-210">Wait for the deployment to finish.</span></span> <span data-ttu-id="32770-211">Esta implementación crea una red virtual, una máquina virtual con Ubuntu y una instancia de VPN Gateway.</span><span class="sxs-lookup"><span data-stu-id="32770-211">This deployment creates a virtual network, a virtual machine running Ubuntu, and a VPN gateway.</span></span> <span data-ttu-id="32770-212">La creación de la instancia de VPN Gateway puede durar más de cuarenta minutos.</span><span class="sxs-lookup"><span data-stu-id="32770-212">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="azure-hub-vnet"></a><span data-ttu-id="32770-213">Red virtual del concentrador de Azure</span><span class="sxs-lookup"><span data-stu-id="32770-213">Azure hub VNet</span></span>

<span data-ttu-id="32770-214">Para implementar la red virtual del concentrador y conectarla a la red virtual local simulada creada anteriormente, realice los pasos siguientes.</span><span class="sxs-lookup"><span data-stu-id="32770-214">To deploy the hub VNet, and connect to the simulated on-premises VNet created above, perform the following steps.</span></span>

1. <span data-ttu-id="32770-215">Navegue hasta la carpeta `hybrid-networking\hub-spoke\hub` del repositorio que descargó en el paso de requisitos previos anterior.</span><span class="sxs-lookup"><span data-stu-id="32770-215">Navigate to the `hybrid-networking\hub-spoke\hub` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="32770-216">Abra el archivo `hub.vm.parameters.json`, escriba un nombre de usuario y la contraseña entre comillas en las líneas 11 y 12, tal y como se muestra a continuación, y después guarde el archivo.</span><span class="sxs-lookup"><span data-stu-id="32770-216">Open the `hub.vm.parameters.json` file and enter a username and password between the quotes in line 11 and 12, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. <span data-ttu-id="32770-217">Abra el archivo `hub.gateway.parameters.json` y escriba una clave compartida entre comillas en la línea 23, tal y como se muestra a continuación, y después guarde el archivo.</span><span class="sxs-lookup"><span data-stu-id="32770-217">Open the `hub.gateway.parameters.json` file and enter a shared key between the quotes in line 23, as shown below, then save the file.</span></span> <span data-ttu-id="32770-218">Anote este valor, ya que deberá usarlo más adelante en la implementación.</span><span class="sxs-lookup"><span data-stu-id="32770-218">Keep a note of this value, you will need to use it later in the deployment.</span></span>

  ```bash
  "sharedKey": "",
  ```

4. <span data-ttu-id="32770-219">Ejecute el comando de Bash o de PowerShell siguiente para implementar el entorno local simulado como una red virtual en Azure.</span><span class="sxs-lookup"><span data-stu-id="32770-219">Run the bash or PowerShell command below to deploy the simulated on-premises environment as a VNet in Azure.</span></span> <span data-ttu-id="32770-220">Sustituya los valores con su suscripción, el nombre del grupo de recursos y la región de Azure.</span><span class="sxs-lookup"><span data-stu-id="32770-220">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

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
  > <span data-ttu-id="32770-221">Si decide usar un nombre del grupo de recursos distinto (que no sea `ra-hub-rg`), asegúrese de que busca todos los archivos de parámetros que usan ese nombre y de que los modifica para usar el nombre de su propio grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="32770-221">If you decide to use a different resource group name (other than `ra-hub-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="32770-222">Espere a que finalice la implementación.</span><span class="sxs-lookup"><span data-stu-id="32770-222">Wait for the deployment to finish.</span></span> <span data-ttu-id="32770-223">Esta implementación crea una red virtual, una máquina virtual con Ubuntu, una instancia de VPN Gateway y una conexión a la puerta de enlace creada en la sección anterior.</span><span class="sxs-lookup"><span data-stu-id="32770-223">This deployment creates a virtual network, a virtual machine running Ubuntu, a VPN gateway, and a connection to the gateway created in the previous section.</span></span> <span data-ttu-id="32770-224">La creación de la instancia de VPN Gateway puede durar más de cuarenta minutos.</span><span class="sxs-lookup"><span data-stu-id="32770-224">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="connection-from-on-premises-to-the-hub"></a><span data-ttu-id="32770-225">Conexión del entorno local al concentrador</span><span class="sxs-lookup"><span data-stu-id="32770-225">Connection from on-premises to the hub</span></span>

<span data-ttu-id="32770-226">Para establecer una conexión del centro de datos local simulado a la red virtual del concentrador, realice los pasos siguientes.</span><span class="sxs-lookup"><span data-stu-id="32770-226">To connect from the simulated on-premises datacenter to the hub VNet, perform the following steps.</span></span>

1. <span data-ttu-id="32770-227">Navegue hasta la carpeta `hybrid-networking\hub-spoke\onprem` del repositorio que descargó en el paso de requisitos previos anterior.</span><span class="sxs-lookup"><span data-stu-id="32770-227">Navigate to the `hybrid-networking\hub-spoke\onprem` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="32770-228">Abra el archivo `onprem.connection.parameters.json` y escriba una clave compartida entre comillas en la línea 9, tal y como se muestra a continuación, y después guarde el archivo.</span><span class="sxs-lookup"><span data-stu-id="32770-228">Open the `onprem.connection.parameters.json` file and enter a shared key between the quotes in line 9, as shown below, then save the file.</span></span> <span data-ttu-id="32770-229">Este valor de clave compartida debe ser el mismo que se usa en la puerta de enlace local implementada previamente.</span><span class="sxs-lookup"><span data-stu-id="32770-229">This shared key value must be the same used in the on-premises gateway you deployed previously.</span></span>

  ```bash
  "sharedKey": "",
  ```

3. <span data-ttu-id="32770-230">Ejecute el comando de Bash o de PowerShell siguiente para implementar el entorno local simulado como una red virtual en Azure.</span><span class="sxs-lookup"><span data-stu-id="32770-230">Run the bash or PowerShell command below to deploy the simulated on-premises environment as a VNet in Azure.</span></span> <span data-ttu-id="32770-231">Sustituya los valores con su suscripción, el nombre del grupo de recursos y la región de Azure.</span><span class="sxs-lookup"><span data-stu-id="32770-231">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

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
  > <span data-ttu-id="32770-232">Si decide usar un nombre del grupo de recursos distinto (que no sea `ra-onprem-rg`), asegúrese de que busca todos los archivos de parámetros que usan ese nombre y de que los modifica para usar el nombre de su propio grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="32770-232">If you decide to use a different resource group name (other than `ra-onprem-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

4. <span data-ttu-id="32770-233">Espere a que finalice la implementación.</span><span class="sxs-lookup"><span data-stu-id="32770-233">Wait for the deployment to finish.</span></span> <span data-ttu-id="32770-234">Esta implementación crea una conexión entre la red virtual que se utiliza para simular un centro de datos local y la red virtual del concentrador.</span><span class="sxs-lookup"><span data-stu-id="32770-234">This deployment creates a connection between the VNet used to simulate an on-premises datacenter, and the hub VNet.</span></span>

### <a name="azure-spoke-vnets"></a><span data-ttu-id="32770-235">Redes virtuales de radios de Azure</span><span class="sxs-lookup"><span data-stu-id="32770-235">Azure spoke VNets</span></span>

<span data-ttu-id="32770-236">Para implementar las redes virtuales de los radios y conectarlas a la red virtual del concentrador creada anteriormente, realice los pasos siguientes.</span><span class="sxs-lookup"><span data-stu-id="32770-236">To deploy the spoke VNets, and connect to the hub VNet created above, perform the following steps.</span></span>

1. <span data-ttu-id="32770-237">Cambie a la carpeta `hybrid-networking\hub-spoke\spokes` del repositorio que descargó en el paso de requisitos previos anterior.</span><span class="sxs-lookup"><span data-stu-id="32770-237">Switch to the `hybrid-networking\hub-spoke\spokes` folder for the repository you download in the pre-requisites step above.</span></span>

2. <span data-ttu-id="32770-238">Abra el archivo `spoke1.web.parameters.json` y escriba un nombre de usuario y la contraseña entre comillas en las líneas 53 y 54, tal y como se muestra a continuación, y después guarde el archivo.</span><span class="sxs-lookup"><span data-stu-id="32770-238">Open the `spoke1.web.parameters.json` file and enter a username and password between the quotes in line 53 and 54, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. <span data-ttu-id="32770-239">Repita el paso anterior para el archivo `spoke2.web.parameters.json`.</span><span class="sxs-lookup"><span data-stu-id="32770-239">Repeat the previous step for file `spoke2.web.parameters.json`.</span></span>

4. <span data-ttu-id="32770-240">Ejecute el comando de Bash o de PowerShell siguiente para implementar el primer radio y conectarlo al concentrador.</span><span class="sxs-lookup"><span data-stu-id="32770-240">Run the bash or PowerShell command below to deploy the first spoke and connect it to the hub.</span></span> <span data-ttu-id="32770-241">Sustituya los valores con su suscripción, el nombre del grupo de recursos y la región de Azure.</span><span class="sxs-lookup"><span data-stu-id="32770-241">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

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
  > <span data-ttu-id="32770-242">Si decide usar un nombre del grupo de recursos distinto (que no sea `ra-spoke1-rg`), asegúrese de que busca todos los archivos de parámetros que usan ese nombre y de que los modifica para usar el nombre de su propio grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="32770-242">If you decide to use a different resource group name (other than `ra-spoke1-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="32770-243">Espere a que finalice la implementación.</span><span class="sxs-lookup"><span data-stu-id="32770-243">Wait for the deployment to finish.</span></span> <span data-ttu-id="32770-244">Esta implementación crea una red virtual, un equilibrador de carga con tres máquinas virtuales que ejecutan Ubuntu y Apache y una conexión de emparejamiento de VNET a la red virtual del concentrador creada en la sección anterior.</span><span class="sxs-lookup"><span data-stu-id="32770-244">This deployment creates a virtual network, a load balancer with three virtual machine running Ubuntu and Apache, and a VNet peering connection to the hub VNet created in the previous section.</span></span> <span data-ttu-id="32770-245">Esta implementación puede tardar más de veinte minutos.</span><span class="sxs-lookup"><span data-stu-id="32770-245">This deployment may take over 20 minutes.</span></span>

6. <span data-ttu-id="32770-246">Ejecute el comando de Bash o de PowerShell siguiente para implementar el primer radio y conectarlo al concentrador.</span><span class="sxs-lookup"><span data-stu-id="32770-246">Run the bash or PowerShell command below to deploy the first spoke and connect it to the hub.</span></span> <span data-ttu-id="32770-247">Sustituya los valores con su suscripción, el nombre del grupo de recursos y la región de Azure.</span><span class="sxs-lookup"><span data-stu-id="32770-247">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

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
  > <span data-ttu-id="32770-248">Si decide usar un nombre del grupo de recursos distinto (que no sea `ra-spoke2-rg`), asegúrese de que busca todos los archivos de parámetros que usan ese nombre y de que los modifica para usar el nombre de su propio grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="32770-248">If you decide to use a different resource group name (other than `ra-spoke2-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="32770-249">Espere a que finalice la implementación.</span><span class="sxs-lookup"><span data-stu-id="32770-249">Wait for the deployment to finish.</span></span> <span data-ttu-id="32770-250">Esta implementación crea una red virtual, un equilibrador de carga con tres máquinas virtuales que ejecutan Ubuntu y Apache y una conexión de emparejamiento de VNET a la red virtual del concentrador creada en la sección anterior.</span><span class="sxs-lookup"><span data-stu-id="32770-250">This deployment creates a virtual network, a load balancer with three virtual machine running Ubuntu and Apache, and a VNet peering connection to the hub VNet created in the previous section.</span></span> <span data-ttu-id="32770-251">Esta implementación puede tardar más de veinte minutos.</span><span class="sxs-lookup"><span data-stu-id="32770-251">This deployment may take over 20 minutes.</span></span>

### <a name="azure-hub-vnet-peering-to-spoke-vnets"></a><span data-ttu-id="32770-252">Emparejamiento de VNET del concentrador de Azure a las redes virtuales de los radios</span><span class="sxs-lookup"><span data-stu-id="32770-252">Azure hub VNet peering to spoke VNets</span></span>

<span data-ttu-id="32770-253">Para implementar las conexiones de emparejamiento de VNET para la red virtual del concentrador, realice los pasos siguientes.</span><span class="sxs-lookup"><span data-stu-id="32770-253">To deploy the VNet peering connections for the hub VNet, perform the following steps.</span></span>

1. <span data-ttu-id="32770-254">Cambie a la carpeta `hybrid-networking\hub-spoke\hub` del repositorio que descargó en el paso de requisitos previos anterior.</span><span class="sxs-lookup"><span data-stu-id="32770-254">Switch to the `hybrid-networking\hub-spoke\hub` folder for the repository you download in the pre-requisites step above.</span></span>

2. <span data-ttu-id="32770-255">Ejecute el comando de Bash o de PowerShell siguiente para implementar la conexión de emparejamiento al primer radio.</span><span class="sxs-lookup"><span data-stu-id="32770-255">Run the bash or PowerShell command below to deploy the peering connection to the first spoke.</span></span> <span data-ttu-id="32770-256">Sustituya los valores con su suscripción, el nombre del grupo de recursos y la región de Azure.</span><span class="sxs-lookup"><span data-stu-id="32770-256">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

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

2. <span data-ttu-id="32770-257">Ejecute el comando de Bash o de PowerShell siguiente para implementar la conexión de emparejamiento al segundo radio.</span><span class="sxs-lookup"><span data-stu-id="32770-257">Run the bash or PowerShell command below to deploy the peering connection to the second spoke.</span></span> <span data-ttu-id="32770-258">Sustituya los valores con su suscripción, el nombre del grupo de recursos y la región de Azure.</span><span class="sxs-lookup"><span data-stu-id="32770-258">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

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

### <a name="test-connectivity"></a><span data-ttu-id="32770-259">Comprobación de la conectividad</span><span class="sxs-lookup"><span data-stu-id="32770-259">Test connectivity</span></span>

<span data-ttu-id="32770-260">Para verificar si funciona la topología en estrella tipo hub-and-spoke conectada a una implementación del centro de datos local, siga estos pasos.</span><span class="sxs-lookup"><span data-stu-id="32770-260">To verify that the hub-spoke topology connected to an on-premises datacenter deployment worked, follow these steps.</span></span>

1. <span data-ttu-id="32770-261">Desde [Azure Portal][portal], conéctese a su suscripción y vaya a la máquina virtual `ra-onprem-vm1` del grupo de recursos `ra-onprem-rg`.</span><span class="sxs-lookup"><span data-stu-id="32770-261">From the [Azure portal][portal], connect to your subscription, and navigate to the `ra-onprem-vm1` virtual machine in the `ra-onprem-rg` resource group.</span></span>

2. <span data-ttu-id="32770-262">En la hoja `Overview`, tenga en cuenta `Public IP address` para la máquina virtual.</span><span class="sxs-lookup"><span data-stu-id="32770-262">In the `Overview` blade, note the `Public IP address` for the VM.</span></span>

3. <span data-ttu-id="32770-263">Use un cliente SSH para conectarse a la dirección IP anotada anteriormente con el uso del nombre de usuario y la contraseña especificados durante la implementación.</span><span class="sxs-lookup"><span data-stu-id="32770-263">Use an SSH client to connect to the IP address you noted above using the user name and password you specified during deployment.</span></span>

4. <span data-ttu-id="32770-264">Desde el símbolo del sistema en la máquina virtual a la que está conectado, ejecute el comando siguiente para probar la conectividad desde la red virtual local a la red virtual Spoke1.</span><span class="sxs-lookup"><span data-stu-id="32770-264">From the command prompt on the VM you connected to, run the command below to test connectivity from the on-premises VNet to the Spoke1 VNet.</span></span>

  ```bash
  ping 10.1.1.37
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
