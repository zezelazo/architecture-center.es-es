---
title: Implementación de una topología de red en estrella tipo hub-and-spoke en Azure
description: Cómo implementar una topología de red en estrella tipo hub-and-spoke en Azure.
author: telmosampaio
ms.date: 04/09/2018
pnp.series.title: Implement a hub-spoke network topology in Azure
pnp.series.prev: expressroute
ms.openlocfilehash: 9105748f434e5d655b09b1fe0775417f33a912b0
ms.sourcegitcommit: f7fa67e3bdbc57d368edb67bac0e1fdec63695d2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/05/2018
ms.locfileid: "37843599"
---
# <a name="implement-a-hub-spoke-network-topology-in-azure"></a><span data-ttu-id="9178f-103">Implementación de una topología de red en estrella tipo hub-and-spoke en Azure</span><span class="sxs-lookup"><span data-stu-id="9178f-103">Implement a hub-spoke network topology in Azure</span></span>

<span data-ttu-id="9178f-104">En esta arquitectura de referencia se muestra cómo implementar una topología en estrella tipo hub-and-spoke en Azure.</span><span class="sxs-lookup"><span data-stu-id="9178f-104">This reference architecture shows how to implement a hub-spoke topology in Azure.</span></span> <span data-ttu-id="9178f-105">El *concentrador* (hub) es una red virtual (VNet) en Azure que actúa como un punto central de conectividad para la red local.</span><span class="sxs-lookup"><span data-stu-id="9178f-105">The *hub* is a virtual network (VNet) in Azure that acts as a central point of connectivity to your on-premises network.</span></span> <span data-ttu-id="9178f-106">Los *radios* (spoke) son redes virtuales (VNet) que se emparejan con el concentrador y que se pueden usar para aislar cargas de trabajo.</span><span class="sxs-lookup"><span data-stu-id="9178f-106">The *spokes* are VNets that peer with the hub, and can be used to isolate workloads.</span></span> <span data-ttu-id="9178f-107">El tráfico fluye entre el centro de datos local y el concentrador a través de una conexión a ExpressRoute o a VPN Gateway.</span><span class="sxs-lookup"><span data-stu-id="9178f-107">Traffic flows between the on-premises datacenter and the hub through an ExpressRoute or VPN gateway connection.</span></span>  <span data-ttu-id="9178f-108">[**Implemente esta solución**](#deploy-the-solution).</span><span class="sxs-lookup"><span data-stu-id="9178f-108">[**Deploy this solution**](#deploy-the-solution).</span></span>

<span data-ttu-id="9178f-109">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="9178f-109">![[0]][0]</span></span>

<span data-ttu-id="9178f-110">*Descargue un [archivo Visio][visio-download] de esta arquitectura.*</span><span class="sxs-lookup"><span data-stu-id="9178f-110">*Download a [Visio file][visio-download] of this architecture*</span></span>


<span data-ttu-id="9178f-111">Las ventajas de esta topología son:</span><span class="sxs-lookup"><span data-stu-id="9178f-111">The benefits of this toplogy include:</span></span>

* <span data-ttu-id="9178f-112">**Ahorros en costos** gracias a la centralización de los servicios que se pueden compartir entre varias cargas de trabajo, como las aplicaciones virtuales de red (NVA) y los servidores DNS, en una única ubicación.</span><span class="sxs-lookup"><span data-stu-id="9178f-112">**Cost savings** by centralizing services that can be shared by multiple workloads, such as network virtual appliances (NVAs) and DNS servers, in a single location.</span></span>
* <span data-ttu-id="9178f-113">**Superar los límites de las suscripciones** gracias al emparejamiento de las redes virtuales de diferentes suscripciones con el concentrador central.</span><span class="sxs-lookup"><span data-stu-id="9178f-113">**Overcome subscriptions limits** by peering VNets from different subscriptions to the central hub.</span></span>
* <span data-ttu-id="9178f-114">**Separación de intereses** entre la TI central (SecOps e InfraOps) y las cargas de trabajo (DevOps).</span><span class="sxs-lookup"><span data-stu-id="9178f-114">**Separation of concerns** between central IT (SecOps, InfraOps) and workloads (DevOps).</span></span>

<span data-ttu-id="9178f-115">Los usos habituales de esta arquitectura incluyen:</span><span class="sxs-lookup"><span data-stu-id="9178f-115">Typical uses for this architecture include:</span></span>

* <span data-ttu-id="9178f-116">Cargas de trabajo implementadas en distintos entornos, como desarrollo, pruebas y producción, que requieren servicios compartidos como DNS, IDS, NTP o AD DS.</span><span class="sxs-lookup"><span data-stu-id="9178f-116">Workloads deployed in different environments, such as development, testing, and production, that require shared services such as DNS, IDS, NTP, or AD DS.</span></span> <span data-ttu-id="9178f-117">Los servicios compartidos se colocan en la red virtual del concentrador, mientras que cada entorno se implementa en un radio para mantener el aislamiento.</span><span class="sxs-lookup"><span data-stu-id="9178f-117">Shared services are placed in the hub VNet, while each environment is deployed to a spoke to maintain isolation.</span></span>
* <span data-ttu-id="9178f-118">Cargas de trabajo que no requieren conectividad entre sí, pero requieren acceso a los servicios compartidos.</span><span class="sxs-lookup"><span data-stu-id="9178f-118">Workloads that do not require connectivity to each other, but require access to shared services.</span></span>
* <span data-ttu-id="9178f-119">Empresas que requieren el control centralizado sobre aspectos de seguridad, como un firewall en el concentrador como una red perimetral, así como la administración segregada de las cargas de trabajo en cada radio.</span><span class="sxs-lookup"><span data-stu-id="9178f-119">Enterprises that require central control over security aspects, such as a firewall in the hub as a DMZ, and segregated management for the workloads in each spoke.</span></span>

## <a name="architecture"></a><span data-ttu-id="9178f-120">Arquitectura</span><span class="sxs-lookup"><span data-stu-id="9178f-120">Architecture</span></span>

<span data-ttu-id="9178f-121">La arquitectura consta de los siguientes componentes:</span><span class="sxs-lookup"><span data-stu-id="9178f-121">The architecture consists of the following components.</span></span>

* <span data-ttu-id="9178f-122">**Red local**.</span><span class="sxs-lookup"><span data-stu-id="9178f-122">**On-premises network**.</span></span> <span data-ttu-id="9178f-123">Una red de área local privada que se ejecuta dentro de una organización.</span><span class="sxs-lookup"><span data-stu-id="9178f-123">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="9178f-124">**Dispositivo VPN**.</span><span class="sxs-lookup"><span data-stu-id="9178f-124">**VPN device**.</span></span> <span data-ttu-id="9178f-125">Un dispositivo o servicio que proporciona conectividad externa a la red local.</span><span class="sxs-lookup"><span data-stu-id="9178f-125">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="9178f-126">El dispositivo VPN puede ser un dispositivo de hardware, o puede ser una solución de software como el servicio de Enrutamiento y acceso remoto (RRAS) en Windows Server 2012.</span><span class="sxs-lookup"><span data-stu-id="9178f-126">The VPN device may be a hardware device, or a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="9178f-127">Para obtener una lista de dispositivos VPN compatibles e información acerca de cómo configurar dispositivos VPN seleccionados para conectarse a Azure, consulte [Acerca de los dispositivos VPN y los parámetros de IPsec/IKE para conexiones de VPN Gateway de sitio a sitio][vpn-appliance].</span><span class="sxs-lookup"><span data-stu-id="9178f-127">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="9178f-128">**Puerta de enlace de red virtual de VPN o puerta de enlace de ExpressRoute**.</span><span class="sxs-lookup"><span data-stu-id="9178f-128">**VPN virtual network gateway or ExpressRoute gateway**.</span></span> <span data-ttu-id="9178f-129">La puerta de enlace de red virtual permite que la red virtual se conecte al dispositivo VPN o al circuito de ExpressRoute que se usa para la conectividad con la red local.</span><span class="sxs-lookup"><span data-stu-id="9178f-129">The virtual network gateway enables the VNet to connect to the VPN device, or ExpressRoute circuit, used for connectivity with your on-premises network.</span></span> <span data-ttu-id="9178f-130">Para más información, consulte [Conectar una red local con una red virtual de Microsoft Azure][connect-to-an-Azure-vnet].</span><span class="sxs-lookup"><span data-stu-id="9178f-130">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

> [!NOTE]
> <span data-ttu-id="9178f-131">Los scripts de implementación para esta arquitectura de referencia usan una instancia de VPN Gateway para la conectividad y una red virtual en Azure para simular la red local.</span><span class="sxs-lookup"><span data-stu-id="9178f-131">The deployment scripts for this reference architecture use a VPN gateway for connectivity, and a VNet in Azure to simulate your on-premises network.</span></span>

* <span data-ttu-id="9178f-132">**Red virtual del concentrador**.</span><span class="sxs-lookup"><span data-stu-id="9178f-132">**Hub VNet**.</span></span> <span data-ttu-id="9178f-133">La red virtual de Azure utilizada como el concentrador en la topología en estrella tipo hub-and-spoke.</span><span class="sxs-lookup"><span data-stu-id="9178f-133">Azure VNet used as the hub in the hub-spoke topology.</span></span> <span data-ttu-id="9178f-134">El concentrador es el punto central de conectividad a la red local y un lugar para hospedar los servicios que se pueden usar en las diferentes cargas de trabajo hospedadas en las redes virtuales de los radios.</span><span class="sxs-lookup"><span data-stu-id="9178f-134">The hub is the central point of connectivity to your on-premises network, and a place to host services that can be consumed by the different workloads hosted in the spoke VNets.</span></span>

* <span data-ttu-id="9178f-135">**Subred de puerta de enlace**.</span><span class="sxs-lookup"><span data-stu-id="9178f-135">**Gateway subnet**.</span></span> <span data-ttu-id="9178f-136">Las puertas de enlace de red virtual se conservan en la misma subred.</span><span class="sxs-lookup"><span data-stu-id="9178f-136">The virtual network gateways are held in the same subnet.</span></span>

* <span data-ttu-id="9178f-137">**Redes virtuales de radios**.</span><span class="sxs-lookup"><span data-stu-id="9178f-137">**Spoke VNets**.</span></span> <span data-ttu-id="9178f-138">Una o varias redes virtuales de Azure que se usan como radios en la topología en estrella tipo hub-and-spoke.</span><span class="sxs-lookup"><span data-stu-id="9178f-138">One or more Azure VNets that are used as spokes in the hub-spoke topology.</span></span> <span data-ttu-id="9178f-139">Los radios pueden utilizarse para aislar las cargas de trabajo en sus propias redes virtuales, administradas por separado desde otros radios.</span><span class="sxs-lookup"><span data-stu-id="9178f-139">Spokes can be used to isolate workloads in their own VNets, managed separately from other spokes.</span></span> <span data-ttu-id="9178f-140">Cada carga de trabajo puede incluir varios niveles, con varias subredes que se conectan a través de equilibradores de carga de Azure.</span><span class="sxs-lookup"><span data-stu-id="9178f-140">Each workload might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="9178f-141">Para obtener más información acerca de la infraestructura de aplicaciones, consulte [Ejecutar cargas de trabajo de máquinas virtuales Windows][windows-vm-ra] y [Ejecutar cargas de trabajo de máquinas virtuales Linux][linux-vm-ra].</span><span class="sxs-lookup"><span data-stu-id="9178f-141">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

* <span data-ttu-id="9178f-142">**Emparejamiento de VNET**.</span><span class="sxs-lookup"><span data-stu-id="9178f-142">**VNet peering**.</span></span> <span data-ttu-id="9178f-143">Se pueden conectar dos redes virtuales en la misma región de Azure mediante una [conexión de emparejamiento][vnet-peering].</span><span class="sxs-lookup"><span data-stu-id="9178f-143">Two VNets in the same Azure region can be connected using a [peering connection][vnet-peering].</span></span> <span data-ttu-id="9178f-144">Las conexiones de emparejamiento son conexiones no transitivas de baja latencia entre las redes virtuales.</span><span class="sxs-lookup"><span data-stu-id="9178f-144">Peering connections are non-transitive, low latency connections between VNets.</span></span> <span data-ttu-id="9178f-145">Una vez establecido el emparejamiento, las redes virtuales intercambian el tráfico mediante la red troncal de Azure, sin necesidad de un enrutador.</span><span class="sxs-lookup"><span data-stu-id="9178f-145">Once peered, the VNets exchange traffic by using the Azure backbone, without the need for a router.</span></span> <span data-ttu-id="9178f-146">En una topología de red en estrella tipo hub-and-spoke, el emparejamiento de VNET se usa para conectar el concentrador a cada radio.</span><span class="sxs-lookup"><span data-stu-id="9178f-146">In a hub-spoke network topology, you use VNet peering to connect the hub to each spoke.</span></span>

> [!NOTE]
> <span data-ttu-id="9178f-147">En este artículo solo se tratan las implementaciones de [Resource Manager](/azure/azure-resource-manager/resource-group-overview), pero también puede conectar una red virtual clásica a una red virtual de Resource Manager en la misma suscripción.</span><span class="sxs-lookup"><span data-stu-id="9178f-147">This article only covers [Resource Manager](/azure/azure-resource-manager/resource-group-overview) deployments, but you can also connect a classic VNet to a Resource Manager VNet in the same subscription.</span></span> <span data-ttu-id="9178f-148">De este modo, los radios pueden hospedar implementaciones clásicas y seguir beneficiándose de los servicios compartidos en el concentrador.</span><span class="sxs-lookup"><span data-stu-id="9178f-148">That way, your spokes can host classic deployments and still benefit from services shared in the hub.</span></span>

## <a name="recommendations"></a><span data-ttu-id="9178f-149">Recomendaciones</span><span class="sxs-lookup"><span data-stu-id="9178f-149">Recommendations</span></span>

<span data-ttu-id="9178f-150">Las siguientes recomendaciones sirven para la mayoría de los escenarios.</span><span class="sxs-lookup"><span data-stu-id="9178f-150">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="9178f-151">Sígalas a menos que tenga un requisito concreto que las invalide.</span><span class="sxs-lookup"><span data-stu-id="9178f-151">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="resource-groups"></a><span data-ttu-id="9178f-152">Grupos de recursos</span><span class="sxs-lookup"><span data-stu-id="9178f-152">Resource groups</span></span>

<span data-ttu-id="9178f-153">La red virtual del concentrador y la red virtual de cada radio se pueden implementar en diferentes grupos de recursos e incluso en distintas suscripciones, siempre y cuando pertenezcan al mismo inquilino de Azure Active Directory (Azure AD) en la misma región.</span><span class="sxs-lookup"><span data-stu-id="9178f-153">The hub VNet, and each spoke VNet, can be implemented in different resource groups, and even different subscriptions, as long as they belong to the same Azure Active Directory (Azure AD) tenant in the same Azure region.</span></span> <span data-ttu-id="9178f-154">Esto permite realizar una administración descentralizada de cada carga de trabajo, mientras se comparten los servicios que se mantienen en la red virtual del concentrador.</span><span class="sxs-lookup"><span data-stu-id="9178f-154">This allows for a decentralized management of each workload, while sharing services maintained in the hub VNet.</span></span>

### <a name="vnet-and-gatewaysubnet"></a><span data-ttu-id="9178f-155">VNet y GatewaySubnet</span><span class="sxs-lookup"><span data-stu-id="9178f-155">VNet and GatewaySubnet</span></span>

<span data-ttu-id="9178f-156">Cree una subred denominada *GatewaySubnet*, con un intervalo de direcciones de /27.</span><span class="sxs-lookup"><span data-stu-id="9178f-156">Create a subnet named *GatewaySubnet*, with an address range of /27.</span></span> <span data-ttu-id="9178f-157">Esta subred es necesaria para la puerta de enlace de red virtual.</span><span class="sxs-lookup"><span data-stu-id="9178f-157">This subnet is required by the virtual network gateway.</span></span> <span data-ttu-id="9178f-158">La asignación de treinta y dos direcciones a esta subred ayudará a evitar que se alcancen las limitaciones de tamaño de la puerta de enlace en el futuro.</span><span class="sxs-lookup"><span data-stu-id="9178f-158">Allocating 32 addresses to this subnet will help to prevent reaching gateway size limitations in the future.</span></span>

<span data-ttu-id="9178f-159">Para más información sobre la configuración de la puerta de enlace, consulte las siguientes arquitecturas de referencia, en función del tipo de conexión:</span><span class="sxs-lookup"><span data-stu-id="9178f-159">For more information about setting up the gateway, see the following reference architectures, depending on your connection type:</span></span>

- <span data-ttu-id="9178f-160">[Red híbrida mediante ExpressRoute][guidance-expressroute]</span><span class="sxs-lookup"><span data-stu-id="9178f-160">[Hybrid network using ExpressRoute][guidance-expressroute]</span></span>
- <span data-ttu-id="9178f-161">[Red híbrida con una instancia de VPN Gateway][guidance-vpn]</span><span class="sxs-lookup"><span data-stu-id="9178f-161">[Hybrid network using a VPN gateway][guidance-vpn]</span></span>

<span data-ttu-id="9178f-162">Para una mayor disponibilidad, puede utilizar ExpressRoute y una VPN para la conmutación por error.</span><span class="sxs-lookup"><span data-stu-id="9178f-162">For higher availability, you can use ExpressRoute plus a VPN for failover.</span></span> <span data-ttu-id="9178f-163">Vea [Conexión de una red local a Azure mediante ExpressRoute con conmutación por error de VPN][hybrid-ha].</span><span class="sxs-lookup"><span data-stu-id="9178f-163">See [Connect an on-premises network to Azure using ExpressRoute with VPN failover][hybrid-ha].</span></span>

<span data-ttu-id="9178f-164">También se puede utilizar una topología en estrella tipo hub-and-spoke sin una puerta de enlace, en caso de que no se necesite la conectividad con la red local.</span><span class="sxs-lookup"><span data-stu-id="9178f-164">A hub-spoke topology can also be used without a gateway, if you don't need connectivity with your on-premises network.</span></span> 

### <a name="vnet-peering"></a><span data-ttu-id="9178f-165">Emparejamiento de VNET</span><span class="sxs-lookup"><span data-stu-id="9178f-165">VNet peering</span></span>

<span data-ttu-id="9178f-166">El emparejamiento de VNET es una relación no transitiva entre dos redes virtuales.</span><span class="sxs-lookup"><span data-stu-id="9178f-166">VNet peering is a non-transitive relationship between two VNets.</span></span> <span data-ttu-id="9178f-167">Si necesita que los radios se conecten entre sí, considere la posibilidad de agregar una conexión de emparejamiento independiente entre dichos radios.</span><span class="sxs-lookup"><span data-stu-id="9178f-167">If you require spokes to connect to each other, consider adding a separate peering connection between those spokes.</span></span>

<span data-ttu-id="9178f-168">Sin embargo, si tiene varios radios que necesitan conectarse entre sí, se quedará sin posibles conexiones de emparejamiento muy rápido debido a la [limitación del número de emparejamientos de VNET por cada red virtual][vnet-peering-limit].</span><span class="sxs-lookup"><span data-stu-id="9178f-168">However, if you have several spokes that need to connect with each other, you will run out of possible peering connections very quickly due to the [limitation on number of VNets peerings per VNet][vnet-peering-limit].</span></span> <span data-ttu-id="9178f-169">En este escenario, considere la posibilidad de usar rutas definidas por el usuario (UDR) para forzar que el tráfico destinado a un radio se envíe a una NVA que actúa como un enrutador en la red virtual del concentrador.</span><span class="sxs-lookup"><span data-stu-id="9178f-169">In this scenario, consider using user defined routes (UDRs) to force traffic destined to a spoke to be sent to an NVA acting as a router at the hub VNet.</span></span> <span data-ttu-id="9178f-170">Esto permitirá que los radios se conecten entre sí.</span><span class="sxs-lookup"><span data-stu-id="9178f-170">This will allow the spokes to connect to each other.</span></span>

<span data-ttu-id="9178f-171">También puede configurar los radios para que usen la puerta de enlace de la red virtual del concentrador para comunicarse con las redes remotas.</span><span class="sxs-lookup"><span data-stu-id="9178f-171">You can also configure spokes to use the hub VNet gateway to communicate with remote networks.</span></span> <span data-ttu-id="9178f-172">Para permitir que el tráfico de la puerta de enlace fluya del radio al concentrador y para conectarse a las redes remotas, debe:</span><span class="sxs-lookup"><span data-stu-id="9178f-172">To allow gateway traffic to flow from spoke to hub, and connect to remote networks, you must:</span></span>

  - <span data-ttu-id="9178f-173">Configurar la conexión de emparejamiento de VNET en el concentrador para **permitir el tránsito de la puerta de enlace**.</span><span class="sxs-lookup"><span data-stu-id="9178f-173">Configure the VNet peering connection in the hub to **allow gateway transit**.</span></span>
  - <span data-ttu-id="9178f-174">Configurar la conexión de emparejamiento de VNET en cada radio para **usar las puertas de enlace remotas**.</span><span class="sxs-lookup"><span data-stu-id="9178f-174">Configure the VNet peering connection in each spoke to **use remote gateways**.</span></span>
  - <span data-ttu-id="9178f-175">Configurar todas las conexiones de emparejamiento de VNET para **permitir el tráfico reenviado**.</span><span class="sxs-lookup"><span data-stu-id="9178f-175">Configure all VNet peering connections to **allow forwarded traffic**.</span></span>

## <a name="considerations"></a><span data-ttu-id="9178f-176">Consideraciones</span><span class="sxs-lookup"><span data-stu-id="9178f-176">Considerations</span></span>

### <a name="spoke-connectivity"></a><span data-ttu-id="9178f-177">Conectividad de los radios</span><span class="sxs-lookup"><span data-stu-id="9178f-177">Spoke connectivity</span></span>

<span data-ttu-id="9178f-178">Si se necesita una conectividad entre los radios, considere la posibilidad de implementar una NVA para el enrutamiento en el concentrador y de usar las UDR en el radio para reenviar el tráfico al concentrador.</span><span class="sxs-lookup"><span data-stu-id="9178f-178">If you require connectivity between spokes, consider implementing an NVA for routing in the hub, and using UDRs in the spoke to forward traffic to the hub.</span></span>

<span data-ttu-id="9178f-179">![[2]][2]</span><span class="sxs-lookup"><span data-stu-id="9178f-179">![[2]][2]</span></span>

<span data-ttu-id="9178f-180">En este escenario, debe configurar las conexiones de emparejamiento para que **permitan el tráfico reenviado**.</span><span class="sxs-lookup"><span data-stu-id="9178f-180">In this scenario, you must configure the peering connections to **allow forwarded traffic**.</span></span>

### <a name="overcoming-vnet-peering-limits"></a><span data-ttu-id="9178f-181">Superación de los límites de emparejamiento de VNET</span><span class="sxs-lookup"><span data-stu-id="9178f-181">Overcoming VNet peering limits</span></span>

<span data-ttu-id="9178f-182">Asegúrese de tener en cuenta la [limitación del número de emparejamientos de VNET por cada red virtual][vnet-peering-limit] en Azure.</span><span class="sxs-lookup"><span data-stu-id="9178f-182">Make sure you consider the [limitation on number of VNets peerings per VNet][vnet-peering-limit] in Azure.</span></span> <span data-ttu-id="9178f-183">Si decide que necesita más radios de los que el límite permitirá, considere la posibilidad de crear una topología en estrella tipo hub-hub-and-spoke-spoke, donde el primer nivel de radios también actúa como concentradores.</span><span class="sxs-lookup"><span data-stu-id="9178f-183">If you decide you need more spokes than the limit will allow, consider creating a hub-spoke-hub-spoke topology, where the first level of spokes also act as hubs.</span></span> <span data-ttu-id="9178f-184">En el siguiente diagrama se ilustra este enfoque.</span><span class="sxs-lookup"><span data-stu-id="9178f-184">The following diagram shows this approach.</span></span>

<span data-ttu-id="9178f-185">![[3]][3]</span><span class="sxs-lookup"><span data-stu-id="9178f-185">![[3]][3]</span></span>

<span data-ttu-id="9178f-186">También tenga en cuenta qué servicios se comparten en el concentrador, para asegurarse de que el concentrador se escala para tener un número mayor de radios.</span><span class="sxs-lookup"><span data-stu-id="9178f-186">Also consider what services are shared in the hub, to ensure the hub scales for a larger number of spokes.</span></span> <span data-ttu-id="9178f-187">Por ejemplo, si el concentrador proporciona servicios de firewall, tenga en cuenta los límites de ancho de banda de la solución de firewall al agregar varios radios.</span><span class="sxs-lookup"><span data-stu-id="9178f-187">For instance, if your hub provides firewall services, consider the bandwidth limits of your firewall solution when adding multiple spokes.</span></span> <span data-ttu-id="9178f-188">Puede mover algunos de estos servicios compartidos a un segundo nivel de concentradores.</span><span class="sxs-lookup"><span data-stu-id="9178f-188">You might want to move some of these shared services to a second level of hubs.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="9178f-189">Implementación de la solución</span><span class="sxs-lookup"><span data-stu-id="9178f-189">Deploy the solution</span></span>

<span data-ttu-id="9178f-190">Hay disponible una implementación de esta arquitectura en [GitHub][ref-arch-repo].</span><span class="sxs-lookup"><span data-stu-id="9178f-190">A deployment for this architecture is available on [GitHub][ref-arch-repo].</span></span> <span data-ttu-id="9178f-191">Usa máquinas virtuales en cada red virtual para probar la conectividad.</span><span class="sxs-lookup"><span data-stu-id="9178f-191">It uses VMs in each VNet to test connectivity.</span></span> <span data-ttu-id="9178f-192">No hay ningún servicio real hospedado en la subred de **servicios compartidos** de la **red virtual del concentrador**.</span><span class="sxs-lookup"><span data-stu-id="9178f-192">There are no actual services hosted in the **shared-services** subnet in the **hub VNet**.</span></span>

<span data-ttu-id="9178f-193">La implementación crea los siguientes grupos de recursos en su suscripción:</span><span class="sxs-lookup"><span data-stu-id="9178f-193">The deployment creates the following resource groups in your subscription:</span></span>

- <span data-ttu-id="9178f-194">hub-nva-rg</span><span class="sxs-lookup"><span data-stu-id="9178f-194">hub-nva-rg</span></span>
- <span data-ttu-id="9178f-195">hub-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="9178f-195">hub-vnet-rg</span></span>
- <span data-ttu-id="9178f-196">onprem-jb-rg</span><span class="sxs-lookup"><span data-stu-id="9178f-196">onprem-jb-rg</span></span>
- <span data-ttu-id="9178f-197">onprem-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="9178f-197">onprem-vnet-rg</span></span>
- <span data-ttu-id="9178f-198">spoke1-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="9178f-198">spoke1-vnet-rg</span></span>
- <span data-ttu-id="9178f-199">spoke2-vent-rg</span><span class="sxs-lookup"><span data-stu-id="9178f-199">spoke2-vent-rg</span></span>

<span data-ttu-id="9178f-200">Los archivos de parámetro de plantilla hacen referencia a estos nombres, por lo que si se cambian es necesario actualizar los archivos de parámetro para que coincidan.</span><span class="sxs-lookup"><span data-stu-id="9178f-200">The template parameter files refer to these names, so if you change them, update the parameter files to match.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="9178f-201">requisitos previos</span><span class="sxs-lookup"><span data-stu-id="9178f-201">Prerequisites</span></span>

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-the-simulated-on-premises-datacenter"></a><span data-ttu-id="9178f-202">Implementación del centro de datos local simulado</span><span class="sxs-lookup"><span data-stu-id="9178f-202">Deploy the simulated on-premises datacenter</span></span>

<span data-ttu-id="9178f-203">Para implementar el centro de datos local simulado como una red virtual de Azure, siga estos pasos:</span><span class="sxs-lookup"><span data-stu-id="9178f-203">To deploy the simulated on-premises datacenter as an Azure VNet, follow these steps:</span></span>

1. <span data-ttu-id="9178f-204">Vaya a la carpeta `hybrid-networking/hub-spoke` del repositorio de arquitecturas de referencia.</span><span class="sxs-lookup"><span data-stu-id="9178f-204">Navigate to the `hybrid-networking/hub-spoke` folder of the reference architectures repository.</span></span>

2. <span data-ttu-id="9178f-205">Abra el archivo `onprem.json` .</span><span class="sxs-lookup"><span data-stu-id="9178f-205">Open the `onprem.json` file.</span></span> <span data-ttu-id="9178f-206">Reemplace los valores de `adminUsername` y `adminPassword`.</span><span class="sxs-lookup"><span data-stu-id="9178f-206">Replace the values for `adminUsername` and `adminPassword`.</span></span>

    ```bash
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

3. <span data-ttu-id="9178f-207">(Opcional) Para una implementación de Linux, establezca `osType` en `Linux`.</span><span class="sxs-lookup"><span data-stu-id="9178f-207">(Optional) For a Linux deployment, set `osType` to `Linux`.</span></span>

4. <span data-ttu-id="9178f-208">Ejecute el siguiente comando:</span><span class="sxs-lookup"><span data-stu-id="9178f-208">Run the following command:</span></span>

    ```bash
    azbb -s <subscription_id> -g onprem-vnet-rg -l <location> -p onprem.json --deploy
    ```

5. <span data-ttu-id="9178f-209">Espere a que finalice la implementación.</span><span class="sxs-lookup"><span data-stu-id="9178f-209">Wait for the deployment to finish.</span></span> <span data-ttu-id="9178f-210">Esta implementación crea una red virtual, una máquina virtual y una instancia de VPN Gateway.</span><span class="sxs-lookup"><span data-stu-id="9178f-210">This deployment creates a virtual network, a virtual machine, and a VPN gateway.</span></span> <span data-ttu-id="9178f-211">Se tardará aproximadamente unos 40 minutos en crear una instancia de VPN Gateway.</span><span class="sxs-lookup"><span data-stu-id="9178f-211">It can take about 40 minutes to create the VPN gateway.</span></span>

### <a name="deploy-the-hub-vnet"></a><span data-ttu-id="9178f-212">Implementación de la red virtual del concentrador</span><span class="sxs-lookup"><span data-stu-id="9178f-212">Deploy the hub VNet</span></span>

<span data-ttu-id="9178f-213">Para implementar red virtual del concentrador, siga los pasos a continuación.</span><span class="sxs-lookup"><span data-stu-id="9178f-213">To deploy the hub VNet, perform the following steps.</span></span>

1. <span data-ttu-id="9178f-214">Abra el archivo `hub-vnet.json` .</span><span class="sxs-lookup"><span data-stu-id="9178f-214">Open the `hub-vnet.json` file.</span></span> <span data-ttu-id="9178f-215">Reemplace los valores de `adminUsername` y `adminPassword`.</span><span class="sxs-lookup"><span data-stu-id="9178f-215">Replace the values for `adminUsername` and `adminPassword`.</span></span>

    ```bash
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. <span data-ttu-id="9178f-216">(Opcional) Para una implementación de Linux, establezca `osType` en `Linux`.</span><span class="sxs-lookup"><span data-stu-id="9178f-216">(Optional) For a Linux deployment, set `osType` to `Linux`.</span></span>

3. <span data-ttu-id="9178f-217">Busque ambas instancias de `sharedKey` y especifique una clave compartida para la conexión VPN.</span><span class="sxs-lookup"><span data-stu-id="9178f-217">Find both instances of `sharedKey` and enter a shared key for the VPN connection.</span></span> <span data-ttu-id="9178f-218">Los valores deben coincidir.</span><span class="sxs-lookup"><span data-stu-id="9178f-218">The values must match.</span></span>

    ```bash
    "sharedKey": "",
    ```

4. <span data-ttu-id="9178f-219">Ejecute el siguiente comando:</span><span class="sxs-lookup"><span data-stu-id="9178f-219">Run the following command:</span></span>

    ```bash
    azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet.json --deploy
    ```

5. <span data-ttu-id="9178f-220">Espere a que finalice la implementación.</span><span class="sxs-lookup"><span data-stu-id="9178f-220">Wait for the deployment to finish.</span></span> <span data-ttu-id="9178f-221">Esta implementación crea una red virtual, una máquina virtual, una instancia de VPN Gateway y una conexión a la puerta de enlace.</span><span class="sxs-lookup"><span data-stu-id="9178f-221">This deployment creates a virtual network, a virtual machine, a VPN gateway, and a connection to the gateway.</span></span>  <span data-ttu-id="9178f-222">Se tardará aproximadamente unos 40 minutos en crear una instancia de VPN Gateway.</span><span class="sxs-lookup"><span data-stu-id="9178f-222">It can take about 40 minutes to create the VPN gateway.</span></span>

### <a name="test-connectivity-with-the-hub"></a><span data-ttu-id="9178f-223">Prueba de conectividad con el concentrador</span><span class="sxs-lookup"><span data-stu-id="9178f-223">Test connectivity with the hub</span></span>

<span data-ttu-id="9178f-224">Pruebe la conectividad desde el entorno local simulado a la red virtual del concentrador.</span><span class="sxs-lookup"><span data-stu-id="9178f-224">Test conectivity from the simulated on-premises environment to the hub VNet.</span></span>

<span data-ttu-id="9178f-225">**Implementación de Windows**</span><span class="sxs-lookup"><span data-stu-id="9178f-225">**Windows deployment**</span></span>

1. <span data-ttu-id="9178f-226">Use Azure Portal para encontrar la máquina virtual denominada `jb-vm1` en el grupo de recursos `onprem-jb-rg`.</span><span class="sxs-lookup"><span data-stu-id="9178f-226">Use the Azure portal to find the VM named `jb-vm1` in the `onprem-jb-rg` resource group.</span></span>

2. <span data-ttu-id="9178f-227">Haga clic en `Connect` para abrir una sesión de escritorio remoto en la máquina virtual.</span><span class="sxs-lookup"><span data-stu-id="9178f-227">Click `Connect` to open a remote desktop session to the VM.</span></span> <span data-ttu-id="9178f-228">Usar la contraseña que especificó en el `onprem.json` archivo de parámetros.</span><span class="sxs-lookup"><span data-stu-id="9178f-228">Use the password that you specified in the `onprem.json` parameter file.</span></span>

3. <span data-ttu-id="9178f-229">Abra una consola de PowerShell en la máquina virtual y utilice el cmdlet `Test-NetConnection` para comprobar que puede conectarse a la máquina virtual de JumpBox en la red virtual del concentrador.</span><span class="sxs-lookup"><span data-stu-id="9178f-229">Open a PowerShell console in the VM, and use the `Test-NetConnection` cmdlet to verify that you can connect to the jumpbox VM in the hub VNet.</span></span>

   ```powershell
   Test-NetConnection 10.0.0.68 -CommonTCPPort RDP
   ```
<span data-ttu-id="9178f-230">La salida debe tener una apariencia similar a la siguiente:</span><span class="sxs-lookup"><span data-stu-id="9178f-230">The output should look similar to the following:</span></span>

```powershell
ComputerName     : 10.0.0.68
RemoteAddress    : 10.0.0.68
RemotePort       : 3389
InterfaceAlias   : Ethernet 2
SourceAddress    : 192.168.1.000
TcpTestSucceeded : True
```

> [!NOTE]
> <span data-ttu-id="9178f-231">De forma predeterminada, las máquinas virtuales de Windows Server no permiten respuestas ICMP en Azure.</span><span class="sxs-lookup"><span data-stu-id="9178f-231">By default, Windows Server VMs do not allow ICMP responses in Azure.</span></span> <span data-ttu-id="9178f-232">Si desea usar `ping` para probar la conectividad, debe habilitar el tráfico ICMP en el firewall de Windows con seguridad avanzada para cada máquina virtual.</span><span class="sxs-lookup"><span data-stu-id="9178f-232">If you want to use `ping` to test connectivity, you need to enable ICMP traffic in the Windows Advanced Firewall for each VM.</span></span>

<span data-ttu-id="9178f-233">**Implementación de Linux**</span><span class="sxs-lookup"><span data-stu-id="9178f-233">**Linux deployment**</span></span>

1. <span data-ttu-id="9178f-234">Use Azure Portal para encontrar la máquina virtual denominada `jb-vm1` en el grupo de recursos `onprem-jb-rg`.</span><span class="sxs-lookup"><span data-stu-id="9178f-234">Use the Azure portal to find the VM named `jb-vm1` in the `onprem-jb-rg` resource group.</span></span>

2. <span data-ttu-id="9178f-235">Haga clic en `Connect` y copie el comando `ssh`que se muestra en el portal.</span><span class="sxs-lookup"><span data-stu-id="9178f-235">Click `Connect` and copy the `ssh` command shown in the portal.</span></span> 

3. <span data-ttu-id="9178f-236">Desde un símbolo del sistema de Linux, ejecute `ssh` para conectar con el entorno local simulado.</span><span class="sxs-lookup"><span data-stu-id="9178f-236">From a Linux prompt, run `ssh` to connect to the simulated on-premises environment.</span></span> <span data-ttu-id="9178f-237">Usar la contraseña que especificó en el `onprem.json` archivo de parámetros.</span><span class="sxs-lookup"><span data-stu-id="9178f-237">Use the password that you specified in the `onprem.json` parameter file.</span></span>

4. <span data-ttu-id="9178f-238">Use el comando `ping` para probar la conectividad con la máquina virtual de JumpBox en la red virtual del concentrador:</span><span class="sxs-lookup"><span data-stu-id="9178f-238">Use the `ping` command to test connectivity to the jumpbox VM in the hub VNet:</span></span>

   ```bash
   ping 10.0.0.68
   ```

### <a name="deploy-the-spoke-vnets"></a><span data-ttu-id="9178f-239">Implementación de redes virtuales de radios</span><span class="sxs-lookup"><span data-stu-id="9178f-239">Deploy the spoke VNets</span></span>

<span data-ttu-id="9178f-240">Para implementar las redes virtuales de radios, siga estos pasos.</span><span class="sxs-lookup"><span data-stu-id="9178f-240">To deploy the spoke VNets, perform the following steps.</span></span>

1. <span data-ttu-id="9178f-241">Abra el archivo `spoke1.json` .</span><span class="sxs-lookup"><span data-stu-id="9178f-241">Open the `spoke1.json` file.</span></span> <span data-ttu-id="9178f-242">Reemplace los valores de `adminUsername` y `adminPassword`.</span><span class="sxs-lookup"><span data-stu-id="9178f-242">Replace the values for `adminUsername` and `adminPassword`.</span></span>

    ```bash
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. <span data-ttu-id="9178f-243">(Opcional) Para una implementación de Linux, establezca `osType` en `Linux`.</span><span class="sxs-lookup"><span data-stu-id="9178f-243">(Optional) For a Linux deployment, set `osType` to `Linux`.</span></span>

3. <span data-ttu-id="9178f-244">Ejecute el siguiente comando:</span><span class="sxs-lookup"><span data-stu-id="9178f-244">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke1-vnet-rg -l <location> -p spoke1.json --deploy
   ```
  
4. <span data-ttu-id="9178f-245">Repita los pasos 1 y 2 para el archivo `spoke2.json`.</span><span class="sxs-lookup"><span data-stu-id="9178f-245">Repeat steps 1-2 for the `spoke2.json` file.</span></span>

5. <span data-ttu-id="9178f-246">Ejecute el siguiente comando:</span><span class="sxs-lookup"><span data-stu-id="9178f-246">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke2-vnet-rg -l <location> -p spoke2.json --deploy
   ```

6. <span data-ttu-id="9178f-247">Ejecute el siguiente comando:</span><span class="sxs-lookup"><span data-stu-id="9178f-247">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet-peering.json --deploy
   ```

### <a name="test-connectivity"></a><span data-ttu-id="9178f-248">Comprobación de la conectividad</span><span class="sxs-lookup"><span data-stu-id="9178f-248">Test connectivity</span></span>

<span data-ttu-id="9178f-249">Pruebe la conectividad desde el entorno local simulado a la red virtual de radios.</span><span class="sxs-lookup"><span data-stu-id="9178f-249">Test conectivity from the simulated on-premises environment to the spoke VNets.</span></span>

<span data-ttu-id="9178f-250">**Implementación de Windows**</span><span class="sxs-lookup"><span data-stu-id="9178f-250">**Windows deployment**</span></span>

1. <span data-ttu-id="9178f-251">Use Azure Portal para encontrar la máquina virtual denominada `jb-vm1` en el grupo de recursos `onprem-jb-rg`.</span><span class="sxs-lookup"><span data-stu-id="9178f-251">Use the Azure portal to find the VM named `jb-vm1` in the `onprem-jb-rg` resource group.</span></span>

2. <span data-ttu-id="9178f-252">Haga clic en `Connect` para abrir una sesión de escritorio remoto en la máquina virtual.</span><span class="sxs-lookup"><span data-stu-id="9178f-252">Click `Connect` to open a remote desktop session to the VM.</span></span> <span data-ttu-id="9178f-253">Usar la contraseña que especificó en el `onprem.json` archivo de parámetros.</span><span class="sxs-lookup"><span data-stu-id="9178f-253">Use the password that you specified in the `onprem.json` parameter file.</span></span>

3. <span data-ttu-id="9178f-254">Abra una consola de PowerShell en la máquina virtual y utilice el cmdlet `Test-NetConnection` para comprobar que puede conectarse a las máquinas virtuales de JumpBox en las redes virtuales de radio.</span><span class="sxs-lookup"><span data-stu-id="9178f-254">Open a PowerShell console in the VM, and use the `Test-NetConnection` cmdlet to verify that you can connect to the jumpbox VMs in the spoke VNets.</span></span>

   ```powershell
   Test-NetConnection 10.1.0.68 -CommonTCPPort RDP
   Test-NetConnection 10.2.0.68 -CommonTCPPort RDP
   ```

<span data-ttu-id="9178f-255">**Implementación de Linux**</span><span class="sxs-lookup"><span data-stu-id="9178f-255">**Linux deployment**</span></span>

<span data-ttu-id="9178f-256">Para probar la conectividad desde el entorno local simulado a las redes virtuales de radios mediante máquinas virtuales Linux, realice los pasos siguientes:</span><span class="sxs-lookup"><span data-stu-id="9178f-256">To test conectivity from the simulated on-premises environment to the spoke VNets using Linux VMs, perform the following steps:</span></span>

1. <span data-ttu-id="9178f-257">Use Azure Portal para encontrar la máquina virtual denominada `jb-vm1` en el grupo de recursos `onprem-jb-rg`.</span><span class="sxs-lookup"><span data-stu-id="9178f-257">Use the Azure portal to find the VM named `jb-vm1` in the `onprem-jb-rg` resource group.</span></span>

2. <span data-ttu-id="9178f-258">Haga clic en `Connect` y copie el comando `ssh`que se muestra en el portal.</span><span class="sxs-lookup"><span data-stu-id="9178f-258">Click `Connect` and copy the `ssh` command shown in the portal.</span></span> 

3. <span data-ttu-id="9178f-259">Desde un símbolo del sistema de Linux, ejecute `ssh` para conectar con el entorno local simulado.</span><span class="sxs-lookup"><span data-stu-id="9178f-259">From a Linux prompt, run `ssh` to connect to the simulated on-premises environment.</span></span> <span data-ttu-id="9178f-260">Usar la contraseña que especificó en el `onprem.json` archivo de parámetros.</span><span class="sxs-lookup"><span data-stu-id="9178f-260">Use the password that you specified in the `onprem.json` parameter file.</span></span>

5. <span data-ttu-id="9178f-261">Use el comando `ping` para probar la conectividad con las máquinas virtuales de JumpBox en cada radio:</span><span class="sxs-lookup"><span data-stu-id="9178f-261">Use the `ping` command to test connectivity to the jumpbox VMs in each spoke:</span></span>

   ```bash
   ping 10.1.0.68
   ping 10.2.0.68
   ```

### <a name="add-connectivity-between-spokes"></a><span data-ttu-id="9178f-262">Adición de conectividad entre radios</span><span class="sxs-lookup"><span data-stu-id="9178f-262">Add connectivity between spokes</span></span>

<span data-ttu-id="9178f-263">Este paso es opcional.</span><span class="sxs-lookup"><span data-stu-id="9178f-263">This step is optional.</span></span> <span data-ttu-id="9178f-264">Si desea permitir que los radios se conecten unos con otros, tendrá que usar una aplicación virtual de red (NVA) como enrutador de la red virtual de concentrador y forzar el tráfico desde los radios al enrutador cuando intente conectar con otro radio.</span><span class="sxs-lookup"><span data-stu-id="9178f-264">If you want to allow spokes to connect to each other, you must use a newtwork virtual appliance (NVA) as a router in the hub VNet, and force traffic from spokes to the router when trying to connect to another spoke.</span></span> <span data-ttu-id="9178f-265">Para implementar una aplicación virtual de red de ejemplo básica como máquina virtual única, junto con las rutas definidas por el usuario (UDR) que son necesarias para que las dos redes virtuales de radios se conecten, realice los pasos siguientes:</span><span class="sxs-lookup"><span data-stu-id="9178f-265">To deploy a basic sample NVA as a single VM, along with user-defined routes (UDRs) to allow the two spoke VNets to connect, perform the following steps:</span></span>

1. <span data-ttu-id="9178f-266">Abra el archivo `hub-nva.json` .</span><span class="sxs-lookup"><span data-stu-id="9178f-266">Open the `hub-nva.json` file.</span></span> <span data-ttu-id="9178f-267">Reemplace los valores de `adminUsername` y `adminPassword`.</span><span class="sxs-lookup"><span data-stu-id="9178f-267">Replace the values for `adminUsername` and `adminPassword`.</span></span>

    ```bash
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. <span data-ttu-id="9178f-268">Ejecute el siguiente comando:</span><span class="sxs-lookup"><span data-stu-id="9178f-268">Run the following command:</span></span>

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
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[windows-vm-ra]: ../virtual-machines-windows/index.md

[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/hub-spoke.png "Topología en estrella tipo hub-and-spoke en Azure"
[1]: ./images/hub-spoke-gateway-routing.svg "Topología en estrella tipo hub-and-spoke en Azure con enrutamiento transitivo"
[2]: ./images/hub-spoke-no-gateway-routing.svg "Topología en estrella tipo hub-and-spoke en Azure con enrutamiento transitivo mediante una NVA"
[3]: ./images/hub-spokehub-spoke.svg "Topología en estrella tipo hub-hub-and-spoke-spoke en Azure"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
