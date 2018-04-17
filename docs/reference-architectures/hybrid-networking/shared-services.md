---
title: Implementación de una topología de red en estrella tipo hub-and-spoke con servicios compartidos en Azure
description: Implementación de una topología de red en estrella tipo hub-and-spoke con servicios compartidos en Azure.
author: telmosampaio
ms.date: 02/25/2018
pnp.series.title: Implement a hub-spoke network topology with shared services in Azure
pnp.series.prev: hub-spoke
ms.openlocfilehash: b492427f12e026be97629ccdc2b8d19c8c66f47d
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2018
---
# <a name="implement-a-hub-spoke-network-topology-with-shared-services-in-azure"></a><span data-ttu-id="f0669-103">Implementación de una topología de red en estrella tipo hub-and-spoke con servicios compartidos en Azure</span><span class="sxs-lookup"><span data-stu-id="f0669-103">Implement a hub-spoke network topology with shared services in Azure</span></span>

<span data-ttu-id="f0669-104">Esta arquitectura de referencia se basa en la arquitectura de referencia tipo [hub-and-spoke][guidance-hub-spoke] para incluir servicios compartidos en el centro que puedan consumir todos los radios.</span><span class="sxs-lookup"><span data-stu-id="f0669-104">This reference architecture builds on the [hub-spoke][guidance-hub-spoke] reference architecture to include shared services in the hub that can be consumed by all spokes.</span></span> <span data-ttu-id="f0669-105">Como primer paso para migrar un centro de datos a la nube y generar un [centro de datos virtual], los primeros servicios que necesitará compartir son los de identidad y seguridad.</span><span class="sxs-lookup"><span data-stu-id="f0669-105">As a first step toward migrating a datacenter to the cloud, and building a [virtual datacenter], the first services you need to share are identity and security.</span></span> <span data-ttu-id="f0669-106">Esta arquitectura de referencia muestra cómo ampliar los servicios de Active Directory desde el centro de datos local a Azure y cómo agregar una aplicación virtual de red (NVA) que pueda actuar como un firewall, en una topología en estrella tipo hub-and-spoke.</span><span class="sxs-lookup"><span data-stu-id="f0669-106">This reference archiecture shows you how to extend your Active Directory services from your on-premises datacenter to Azure, and how to add a network virtual appliance (NVA) that can act as a firewall, in a hub-spoke topology.</span></span>  <span data-ttu-id="f0669-107">[**Implemente esta solución**](#deploy-the-solution).</span><span class="sxs-lookup"><span data-stu-id="f0669-107">[**Deploy this solution**](#deploy-the-solution).</span></span>

<span data-ttu-id="f0669-108">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="f0669-108">![[0]][0]</span></span>

<span data-ttu-id="f0669-109">*Descargue un [archivo Visio][visio-download] de esta arquitectura.*</span><span class="sxs-lookup"><span data-stu-id="f0669-109">*Download a [Visio file][visio-download] of this architecture*</span></span>

<span data-ttu-id="f0669-110">Las ventajas de esta topología son:</span><span class="sxs-lookup"><span data-stu-id="f0669-110">The benefits of this toplogy include:</span></span>

* <span data-ttu-id="f0669-111">**Ahorros en costos** gracias a la centralización de los servicios que se pueden compartir entre varias cargas de trabajo, como las aplicaciones virtuales de red (NVA) y los servidores DNS, en una única ubicación.</span><span class="sxs-lookup"><span data-stu-id="f0669-111">**Cost savings** by centralizing services that can be shared by multiple workloads, such as network virtual appliances (NVAs) and DNS servers, in a single location.</span></span>
* <span data-ttu-id="f0669-112">**Superar los límites de las suscripciones** gracias al emparejamiento de las redes virtuales de diferentes suscripciones con el concentrador central.</span><span class="sxs-lookup"><span data-stu-id="f0669-112">**Overcome subscriptions limits** by peering VNets from different subscriptions to the central hub.</span></span>
* <span data-ttu-id="f0669-113">**Separación de intereses** entre la TI central (SecOps e InfraOps) y las cargas de trabajo (DevOps).</span><span class="sxs-lookup"><span data-stu-id="f0669-113">**Separation of concerns** between central IT (SecOps, InfraOps) and workloads (DevOps).</span></span>

<span data-ttu-id="f0669-114">Los usos habituales de esta arquitectura incluyen:</span><span class="sxs-lookup"><span data-stu-id="f0669-114">Typical uses for this architecture include:</span></span>

* <span data-ttu-id="f0669-115">Cargas de trabajo implementadas en distintos entornos, como desarrollo, pruebas y producción, que requieren servicios compartidos como DNS, IDS, NTP o AD DS.</span><span class="sxs-lookup"><span data-stu-id="f0669-115">Workloads deployed in different environments, such as development, testing, and production, that require shared services such as DNS, IDS, NTP, or AD DS.</span></span> <span data-ttu-id="f0669-116">Los servicios compartidos se colocan en la red virtual del concentrador, mientras que cada entorno se implementa en un radio para mantener el aislamiento.</span><span class="sxs-lookup"><span data-stu-id="f0669-116">Shared services are placed in the hub VNet, while each environment is deployed to a spoke to maintain isolation.</span></span>
* <span data-ttu-id="f0669-117">Cargas de trabajo que no requieren conectividad entre sí, pero requieren acceso a los servicios compartidos.</span><span class="sxs-lookup"><span data-stu-id="f0669-117">Workloads that do not require connectivity to each other, but require access to shared services.</span></span>
* <span data-ttu-id="f0669-118">Empresas que requieren el control centralizado sobre aspectos de seguridad, como un firewall en el concentrador como una red perimetral, así como la administración segregada de las cargas de trabajo en cada radio.</span><span class="sxs-lookup"><span data-stu-id="f0669-118">Enterprises that require central control over security aspects, such as a firewall in the hub as a DMZ, and segregated management for the workloads in each spoke.</span></span>

## <a name="architecture"></a><span data-ttu-id="f0669-119">Architecture</span><span class="sxs-lookup"><span data-stu-id="f0669-119">Architecture</span></span>

<span data-ttu-id="f0669-120">La arquitectura consta de los siguientes componentes:</span><span class="sxs-lookup"><span data-stu-id="f0669-120">The architecture consists of the following components.</span></span>

* <span data-ttu-id="f0669-121">**Red local**.</span><span class="sxs-lookup"><span data-stu-id="f0669-121">**On-premises network**.</span></span> <span data-ttu-id="f0669-122">Una red de área local privada que se ejecuta dentro de una organización.</span><span class="sxs-lookup"><span data-stu-id="f0669-122">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="f0669-123">**Dispositivo VPN**.</span><span class="sxs-lookup"><span data-stu-id="f0669-123">**VPN device**.</span></span> <span data-ttu-id="f0669-124">Un dispositivo o servicio que proporciona conectividad externa a la red local.</span><span class="sxs-lookup"><span data-stu-id="f0669-124">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="f0669-125">El dispositivo VPN puede ser un dispositivo de hardware, o puede ser una solución de software como el servicio de Enrutamiento y acceso remoto (RRAS) en Windows Server 2012.</span><span class="sxs-lookup"><span data-stu-id="f0669-125">The VPN device may be a hardware device, or a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="f0669-126">Para obtener una lista de dispositivos VPN compatibles e información acerca de cómo configurar dispositivos VPN seleccionados para conectarse a Azure, consulte [Acerca de los dispositivos VPN y los parámetros de IPsec/IKE para conexiones de VPN Gateway de sitio a sitio][vpn-appliance].</span><span class="sxs-lookup"><span data-stu-id="f0669-126">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="f0669-127">**Puerta de enlace de red virtual de VPN o puerta de enlace de ExpressRoute**.</span><span class="sxs-lookup"><span data-stu-id="f0669-127">**VPN virtual network gateway or ExpressRoute gateway**.</span></span> <span data-ttu-id="f0669-128">La puerta de enlace de red virtual permite que la red virtual se conecte al dispositivo VPN o al circuito de ExpressRoute que se usa para la conectividad con la red local.</span><span class="sxs-lookup"><span data-stu-id="f0669-128">The virtual network gateway enables the VNet to connect to the VPN device, or ExpressRoute circuit, used for connectivity with your on-premises network.</span></span> <span data-ttu-id="f0669-129">Para más información, consulte [Conectar una red local con una red virtual de Microsoft Azure][connect-to-an-Azure-vnet].</span><span class="sxs-lookup"><span data-stu-id="f0669-129">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

> [!NOTE]
> <span data-ttu-id="f0669-130">Los scripts de implementación para esta arquitectura de referencia usan una instancia de VPN Gateway para la conectividad y una red virtual en Azure para simular la red local.</span><span class="sxs-lookup"><span data-stu-id="f0669-130">The deployment scripts for this reference architecture use a VPN gateway for connectivity, and a VNet in Azure to simulate your on-premises network.</span></span>

* <span data-ttu-id="f0669-131">**Red virtual del concentrador**.</span><span class="sxs-lookup"><span data-stu-id="f0669-131">**Hub VNet**.</span></span> <span data-ttu-id="f0669-132">La red virtual de Azure utilizada como el concentrador en la topología en estrella tipo hub-and-spoke.</span><span class="sxs-lookup"><span data-stu-id="f0669-132">Azure VNet used as the hub in the hub-spoke topology.</span></span> <span data-ttu-id="f0669-133">El concentrador es el punto central de conectividad a la red local y un lugar para hospedar los servicios que se pueden usar en las diferentes cargas de trabajo hospedadas en las redes virtuales de los radios.</span><span class="sxs-lookup"><span data-stu-id="f0669-133">The hub is the central point of connectivity to your on-premises network, and a place to host services that can be consumed by the different workloads hosted in the spoke VNets.</span></span>

* <span data-ttu-id="f0669-134">**Subred de puerta de enlace**.</span><span class="sxs-lookup"><span data-stu-id="f0669-134">**Gateway subnet**.</span></span> <span data-ttu-id="f0669-135">Las puertas de enlace de red virtual se conservan en la misma subred.</span><span class="sxs-lookup"><span data-stu-id="f0669-135">The virtual network gateways are held in the same subnet.</span></span>

* <span data-ttu-id="f0669-136">**Subred de servicios compartidos**.</span><span class="sxs-lookup"><span data-stu-id="f0669-136">**Shared services subnet**.</span></span> <span data-ttu-id="f0669-137">Una subred de la red virtual del concentrador utilizada para hospedar servicios que se pueden compartir entre todos los radios, como DNS o AD DS.</span><span class="sxs-lookup"><span data-stu-id="f0669-137">A subnet in the hub VNet used to host services that can be shared among all spokes, such as DNS or AD DS.</span></span>

* <span data-ttu-id="f0669-138">**Subred DMZ**.</span><span class="sxs-lookup"><span data-stu-id="f0669-138">**DMZ subnet**.</span></span> <span data-ttu-id="f0669-139">Una subred de la red virtual del centro que se usa para hospedar aplicaciones virtuales de red (NVA) que pueden actuar como aplicaciones de seguridad como, por ejemplo, firewalls.</span><span class="sxs-lookup"><span data-stu-id="f0669-139">A subnet in the hub VNet used to host NVAs that can act as security appliances, such as firewalls.</span></span>

* <span data-ttu-id="f0669-140">**Redes virtuales de radios**.</span><span class="sxs-lookup"><span data-stu-id="f0669-140">**Spoke VNets**.</span></span> <span data-ttu-id="f0669-141">Una o varias redes virtuales de Azure que se usan como radios en la topología en estrella tipo hub-and-spoke.</span><span class="sxs-lookup"><span data-stu-id="f0669-141">One or more Azure VNets that are used as spokes in the hub-spoke topology.</span></span> <span data-ttu-id="f0669-142">Los radios pueden utilizarse para aislar las cargas de trabajo en sus propias redes virtuales, administradas por separado desde otros radios.</span><span class="sxs-lookup"><span data-stu-id="f0669-142">Spokes can be used to isolate workloads in their own VNets, managed separately from other spokes.</span></span> <span data-ttu-id="f0669-143">Cada carga de trabajo puede incluir varios niveles, con varias subredes que se conectan a través de equilibradores de carga de Azure.</span><span class="sxs-lookup"><span data-stu-id="f0669-143">Each workload might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="f0669-144">Para obtener más información acerca de la infraestructura de aplicaciones, consulte [Ejecutar cargas de trabajo de máquinas virtuales Windows][windows-vm-ra] y [Ejecutar cargas de trabajo de máquinas virtuales Linux][linux-vm-ra].</span><span class="sxs-lookup"><span data-stu-id="f0669-144">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

* <span data-ttu-id="f0669-145">**Emparejamiento de VNET**.</span><span class="sxs-lookup"><span data-stu-id="f0669-145">**VNet peering**.</span></span> <span data-ttu-id="f0669-146">Se pueden conectar dos redes virtuales en la misma región de Azure mediante una [conexión de emparejamiento][vnet-peering].</span><span class="sxs-lookup"><span data-stu-id="f0669-146">Two VNets in the same Azure region can be connected using a [peering connection][vnet-peering].</span></span> <span data-ttu-id="f0669-147">Las conexiones de emparejamiento son conexiones no transitivas de baja latencia entre las redes virtuales.</span><span class="sxs-lookup"><span data-stu-id="f0669-147">Peering connections are non-transitive, low latency connections between VNets.</span></span> <span data-ttu-id="f0669-148">Una vez establecido el emparejamiento, las redes virtuales intercambian el tráfico mediante la red troncal de Azure, sin necesidad de un enrutador.</span><span class="sxs-lookup"><span data-stu-id="f0669-148">Once peered, the VNets exchange traffic by using the Azure backbone, without the need for a router.</span></span> <span data-ttu-id="f0669-149">En una topología de red en estrella tipo hub-and-spoke, el emparejamiento de VNET se usa para conectar el concentrador a cada radio.</span><span class="sxs-lookup"><span data-stu-id="f0669-149">In a hub-spoke network topology, you use VNet peering to connect the hub to each spoke.</span></span>

> [!NOTE]
> <span data-ttu-id="f0669-150">En este artículo solo se tratan las implementaciones de [Resource Manager](/azure/azure-resource-manager/resource-group-overview), pero también puede conectar una red virtual clásica a una red virtual de Resource Manager en la misma suscripción.</span><span class="sxs-lookup"><span data-stu-id="f0669-150">This article only covers [Resource Manager](/azure/azure-resource-manager/resource-group-overview) deployments, but you can also connect a classic VNet to a Resource Manager VNet in the same subscription.</span></span> <span data-ttu-id="f0669-151">De este modo, los radios pueden hospedar implementaciones clásicas y seguir beneficiándose de los servicios compartidos en el concentrador.</span><span class="sxs-lookup"><span data-stu-id="f0669-151">That way, your spokes can host classic deployments and still benefit from services shared in the hub.</span></span>

## <a name="recommendations"></a><span data-ttu-id="f0669-152">Recomendaciones</span><span class="sxs-lookup"><span data-stu-id="f0669-152">Recommendations</span></span>

<span data-ttu-id="f0669-153">Todas las recomendaciones de la arquitectura de referencia de tipo [hub-and-spoke][guidance-hub-spoke] también se aplican a la arquitectura de referencia de los servicios compartidos.</span><span class="sxs-lookup"><span data-stu-id="f0669-153">All the recommendations for the [hub-spoke][guidance-hub-spoke] reference architecture also apply to the shared services reference architecture.</span></span> 

<span data-ttu-id="f0669-154">Igualmente, las siguientes recomendaciones sirven para la mayoría de escenarios de servicios compartidos.</span><span class="sxs-lookup"><span data-stu-id="f0669-154">ALso, the following recommendations apply for most scenarios under shared services.</span></span> <span data-ttu-id="f0669-155">Sígalas a menos que tenga un requisito concreto que las invalide.</span><span class="sxs-lookup"><span data-stu-id="f0669-155">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="identity"></a><span data-ttu-id="f0669-156">Identidad</span><span class="sxs-lookup"><span data-stu-id="f0669-156">Identity</span></span>

<span data-ttu-id="f0669-157">La mayoría de las organizaciones empresariales tienen un entorno de servicios de directorio de Active Directory (ADDS) en su centro de datos local.</span><span class="sxs-lookup"><span data-stu-id="f0669-157">Most enterprise organizations have an Active Directory Directory Services (ADDS) environment in their on-premises datacenter.</span></span> <span data-ttu-id="f0669-158">Para facilitar la administración de los recursos trasladados a Azure desde la red local que depende de ADDS, se recomienda hospedar los controladores de dominio de ADDS en Azure.</span><span class="sxs-lookup"><span data-stu-id="f0669-158">To facilitate management of assets moved to Azure from your on-premises network that depend on ADDS, it is recommended to host ADDS domain controllers in Azure.</span></span>

<span data-ttu-id="f0669-159">Si desea utilizar los objetos de la directiva de grupo que desea controlar de forma independiente para Azure y para el entorno local, use un sitio de AD diferente para cada región de Azure.</span><span class="sxs-lookup"><span data-stu-id="f0669-159">If you make use of Group Policy Objects, that you want to control separately for Azure and your on-premises environment, use a different AD site for each Azure region.</span></span> <span data-ttu-id="f0669-160">Sitúe los controladores de dominio en una red virtual central (centro) al que puedan acceder las cargas de trabajo dependientes.</span><span class="sxs-lookup"><span data-stu-id="f0669-160">Place your domain controllers in a central VNet (hub) that dependent workloads can access.</span></span>

### <a name="security"></a><span data-ttu-id="f0669-161">Seguridad</span><span class="sxs-lookup"><span data-stu-id="f0669-161">Security</span></span>

<span data-ttu-id="f0669-162">Al trasladar cargas de trabajo desde el entorno local a Azure, algunas de estas cargas necesitarán hospedarse en máquinas virtuales.</span><span class="sxs-lookup"><span data-stu-id="f0669-162">As you move workloads from your on-premises environment to Azure, some of these workloads will require to be hosted in VMs.</span></span> <span data-ttu-id="f0669-163">Por motivos de cumplimiento normativo, puede que tenga que aplicar restricciones al tráfico que recorre esas cargas de trabajo.</span><span class="sxs-lookup"><span data-stu-id="f0669-163">For compliance reasons, you may need to enforce restrictions on traffic traversing those workloads.</span></span> 

<span data-ttu-id="f0669-164">Puede usar aplicaciones virtuales de red (NVA) en Azure para hospedar diferentes tipos de servicios de seguridad y rendimiento.</span><span class="sxs-lookup"><span data-stu-id="f0669-164">You can use network virtual appliances (NVAs) in Azure to host different types of security and performance services.</span></span> <span data-ttu-id="f0669-165">Si está familiarizado con un conjunto determinado de aplicaciones locales actualmente, se recomienda que use las mismas aplicaciones virtualizadas en Azure, si es posible.</span><span class="sxs-lookup"><span data-stu-id="f0669-165">If you are familiar with a given set of appliances on-premises today, it is recommended to use the same virtualized appliances in Azure, where applicable.</span></span>

> [!NOTE]
> <span data-ttu-id="f0669-166">Los scripts de implementación de esta arquitectura de referencia usan una máquina virtual Ubuntu con reenvío IP habilitado para imitar una aplicación virtual de red.</span><span class="sxs-lookup"><span data-stu-id="f0669-166">The deployment scripts for this reference architecture use an Ubuntu VM with IP forwarding enbaled to mimic a network virtual appliance.</span></span>

## <a name="considerations"></a><span data-ttu-id="f0669-167">Consideraciones</span><span class="sxs-lookup"><span data-stu-id="f0669-167">Considerations</span></span>

### <a name="overcoming-vnet-peering-limits"></a><span data-ttu-id="f0669-168">Superación de los límites de emparejamiento de VNET</span><span class="sxs-lookup"><span data-stu-id="f0669-168">Overcoming VNet peering limits</span></span>

<span data-ttu-id="f0669-169">Asegúrese de tener en cuenta la [limitación del número de emparejamientos de VNET por cada red virtual][vnet-peering-limit] en Azure.</span><span class="sxs-lookup"><span data-stu-id="f0669-169">Make sure you consider the [limitation on number of VNets peerings per VNet][vnet-peering-limit] in Azure.</span></span> <span data-ttu-id="f0669-170">Si decide que necesita más radios de los que el límite permitirá, considere la posibilidad de crear una topología en estrella tipo hub-hub-and-spoke-spoke, donde el primer nivel de radios también actúa como concentradores.</span><span class="sxs-lookup"><span data-stu-id="f0669-170">If you decide you need more spokes than the limit will allow, consider creating a hub-spoke-hub-spoke topology, where the first level of spokes also act as hubs.</span></span> <span data-ttu-id="f0669-171">En el siguiente diagrama se ilustra este enfoque.</span><span class="sxs-lookup"><span data-stu-id="f0669-171">The following diagram shows this approach.</span></span>

<span data-ttu-id="f0669-172">![[3]][3]</span><span class="sxs-lookup"><span data-stu-id="f0669-172">![[3]][3]</span></span>

<span data-ttu-id="f0669-173">También tenga en cuenta qué servicios se comparten en el concentrador, para asegurarse de que el concentrador se escala para tener un número mayor de radios.</span><span class="sxs-lookup"><span data-stu-id="f0669-173">Also consider what services are shared in the hub, to ensure the hub scales for a larger number of spokes.</span></span> <span data-ttu-id="f0669-174">Por ejemplo, si el concentrador proporciona servicios de firewall, tenga en cuenta los límites de ancho de banda de la solución de firewall al agregar varios radios.</span><span class="sxs-lookup"><span data-stu-id="f0669-174">For instance, if your hub provides firewall services, consider the bandwidth limits of your firewall solution when adding multiple spokes.</span></span> <span data-ttu-id="f0669-175">Puede mover algunos de estos servicios compartidos a un segundo nivel de concentradores.</span><span class="sxs-lookup"><span data-stu-id="f0669-175">You might want to move some of these shared services to a second level of hubs.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="f0669-176">Implementación de la solución</span><span class="sxs-lookup"><span data-stu-id="f0669-176">Deploy the solution</span></span>

<span data-ttu-id="f0669-177">Hay disponible una implementación de esta arquitectura en [GitHub][ref-arch-repo].</span><span class="sxs-lookup"><span data-stu-id="f0669-177">A deployment for this architecture is available on [GitHub][ref-arch-repo].</span></span> <span data-ttu-id="f0669-178">Usa máquinas virtuales Ubuntu en cada red virtual para probar la conectividad.</span><span class="sxs-lookup"><span data-stu-id="f0669-178">It uses Ubuntu VMs in each VNet to test connectivity.</span></span> <span data-ttu-id="f0669-179">No hay ningún servicio real hospedado en la subred de **servicios compartidos** de la **red virtual del concentrador**.</span><span class="sxs-lookup"><span data-stu-id="f0669-179">There are no actual services hosted in the **shared-services** subnet in the **hub VNet**.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="f0669-180">requisitos previos</span><span class="sxs-lookup"><span data-stu-id="f0669-180">Prerequisites</span></span>

<span data-ttu-id="f0669-181">Antes de poder implementar la arquitectura de referencia en su propia suscripción, debe realizar los pasos siguientes.</span><span class="sxs-lookup"><span data-stu-id="f0669-181">Before you can deploy the reference architecture to your own subscription, you must perform the following steps.</span></span>

1. <span data-ttu-id="f0669-182">Clone, bifurque o descargue el archivo ZIP del repositorio de GitHub de [arquitecturas de referencia][ref-arch-repo].</span><span class="sxs-lookup"><span data-stu-id="f0669-182">Clone, fork, or download the zip file for the [reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="f0669-183">Asegúrese de que tiene la CLI de Azure 2.0 instalada en el equipo.</span><span class="sxs-lookup"><span data-stu-id="f0669-183">Make sure you have the Azure CLI 2.0 installed on your computer.</span></span> <span data-ttu-id="f0669-184">Para obtener instrucciones sobre la instalación de la CLI, consulte [Instalación de la CLI de Azure 2.0][azure-cli-2].</span><span class="sxs-lookup"><span data-stu-id="f0669-184">For CLI installation instructions, see [Install Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="f0669-185">Instale el paquete de NPM de [Azure Building Blocks][azbb].</span><span class="sxs-lookup"><span data-stu-id="f0669-185">Install the [Azure building blocks][azbb] npm package.</span></span>

4. <span data-ttu-id="f0669-186">Desde un símbolo del sistema, un símbolo del sistema de Bash o un símbolo del sistema de PowerShell, inicie sesión en la cuenta de Azure mediante el siguiente comando y siga las indicaciones.</span><span class="sxs-lookup"><span data-stu-id="f0669-186">From a command prompt, bash prompt, or PowerShell prompt, login to your Azure account by using the command below, and follow the prompts.</span></span>

   ```bash
   az login
   ```

### <a name="deploy-the-simulated-on-premises-datacenter-using-azbb"></a><span data-ttu-id="f0669-187">Implementación del centro de datos local simulado mediante azbb</span><span class="sxs-lookup"><span data-stu-id="f0669-187">Deploy the simulated on-premises datacenter using azbb</span></span>

<span data-ttu-id="f0669-188">Para implementar el centro de datos local simulado como una red virtual de Azure, siga estos pasos:</span><span class="sxs-lookup"><span data-stu-id="f0669-188">To deploy the simulated on-premises datacenter as an Azure VNet, follow these steps:</span></span>

1. <span data-ttu-id="f0669-189">Navegue hasta la carpeta `hybrid-networking\shared-services-stack\` del repositorio que descargó en el paso de requisitos previos anterior.</span><span class="sxs-lookup"><span data-stu-id="f0669-189">Navigate to the `hybrid-networking\shared-services-stack\` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="f0669-190">Abra el archivo `onprem.json` y escriba un nombre de usuario y la contraseña entre comillas en las líneas 45 y 46, tal y como se muestra a continuación, y después guarde el archivo.</span><span class="sxs-lookup"><span data-stu-id="f0669-190">Open the `onprem.json` file and enter a username and password between the quotes in line 45 and 46, as shown below, then save the file.</span></span>

   ```bash
   "adminUsername": "XXX",
   "adminPassword": "YYY",
   ```

3. <span data-ttu-id="f0669-191">Ejecute `azbb` para implementar el entorno simulado local tal y como se muestra a continuación.</span><span class="sxs-lookup"><span data-stu-id="f0669-191">Run `azbb` to deploy the simulated onprem environment as shown below.</span></span>

   ```bash
   azbb -s <subscription_id> -g onprem-vnet-rg - l <location> -p onoprem.json --deploy
   ```
   > [!NOTE]
   > <span data-ttu-id="f0669-192">Si decide usar un nombre del grupo de recursos distinto (que no sea `onprem-vnet-rg`), asegúrese de que busca todos los archivos de parámetros que usan ese nombre y de que los modifica para usar el nombre de su propio grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="f0669-192">If you decide to use a different resource group name (other than `onprem-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

4. <span data-ttu-id="f0669-193">Espere a que finalice la implementación.</span><span class="sxs-lookup"><span data-stu-id="f0669-193">Wait for the deployment to finish.</span></span> <span data-ttu-id="f0669-194">Esta implementación crea una red virtual, una máquina virtual Windows y una instancia de VPN Gateway.</span><span class="sxs-lookup"><span data-stu-id="f0669-194">This deployment creates a virtual network, a virtual machine running Windows, and a VPN gateway.</span></span> <span data-ttu-id="f0669-195">La creación de la instancia de VPN Gateway puede durar más de cuarenta minutos.</span><span class="sxs-lookup"><span data-stu-id="f0669-195">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="azure-hub-vnet"></a><span data-ttu-id="f0669-196">Red virtual del concentrador de Azure</span><span class="sxs-lookup"><span data-stu-id="f0669-196">Azure hub VNet</span></span>

<span data-ttu-id="f0669-197">Para implementar la red virtual del concentrador y conectarla a la red virtual local simulada creada anteriormente, realice los pasos siguientes.</span><span class="sxs-lookup"><span data-stu-id="f0669-197">To deploy the hub VNet, and connect to the simulated on-premises VNet created above, perform the following steps.</span></span>

1. <span data-ttu-id="f0669-198">Abra el archivo `hub-vnet.json` y escriba un nombre de usuario y la contraseña entre comillas en las líneas 50 y 51, tal y como se muestra a continuación.</span><span class="sxs-lookup"><span data-stu-id="f0669-198">Open the `hub-vnet.json` file and enter a username and password between the quotes in line 50 and 51, as shown below.</span></span>

   ```bash
   "adminUsername": "XXX",
   "adminPassword": "YYY",
   ```

2. <span data-ttu-id="f0669-199">En la línea 52, para `osType`, escriba `Windows` o `Linux` para instalar Windows Server 2016 Datacenter o Ubuntu 16.04 como sistema operativo de JumpBox.</span><span class="sxs-lookup"><span data-stu-id="f0669-199">On line 52, for `osType`, type `Windows` or `Linux` to install either Windows Server 2016 Datacenter, or Ubuntu 16.04 as the operating system for the jumpbox.</span></span>

3. <span data-ttu-id="f0669-200">Escriba una clave compartida entre comillas en la línea 83, tal y como se muestra a continuación, y después guarde el archivo.</span><span class="sxs-lookup"><span data-stu-id="f0669-200">Enter a shared key between the quotes in line 83, as shown below, then save the file.</span></span>

   ```bash
   "sharedKey": "",
   ```

4. <span data-ttu-id="f0669-201">Ejecute `azbb` para implementar el entorno simulado local tal y como se muestra a continuación.</span><span class="sxs-lookup"><span data-stu-id="f0669-201">Run `azbb` to deploy the simulated onprem environment as shown below.</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg - l <location> -p hub-vnet.json --deploy
   ```
   > [!NOTE]
   > <span data-ttu-id="f0669-202">Si decide usar un nombre del grupo de recursos distinto (que no sea `hub-vnet-rg`), asegúrese de que busca todos los archivos de parámetros que usan ese nombre y de que los modifica para usar el nombre de su propio grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="f0669-202">If you decide to use a different resource group name (other than `hub-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="f0669-203">Espere a que finalice la implementación.</span><span class="sxs-lookup"><span data-stu-id="f0669-203">Wait for the deployment to finish.</span></span> <span data-ttu-id="f0669-204">Esta implementación crea una red virtual, una máquina virtual, una instancia de VPN Gateway y una conexión a la puerta de enlace creada en la sección anterior.</span><span class="sxs-lookup"><span data-stu-id="f0669-204">This deployment creates a virtual network, a virtual machine, a VPN gateway, and a connection to the gateway created in the previous section.</span></span> <span data-ttu-id="f0669-205">La creación de la instancia de VPN Gateway puede durar más de cuarenta minutos.</span><span class="sxs-lookup"><span data-stu-id="f0669-205">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="adds-in-azure"></a><span data-ttu-id="f0669-206">ADDS en Azure</span><span class="sxs-lookup"><span data-stu-id="f0669-206">ADDS in Azure</span></span>

<span data-ttu-id="f0669-207">Para implementar los controladores de dominio de ADDS en Azure, realice los pasos siguientes.</span><span class="sxs-lookup"><span data-stu-id="f0669-207">To deploy the ADDS domain controllers in Azure, perform the following steps.</span></span>

1. <span data-ttu-id="f0669-208">Abra el archivo `hub-adds.json` y escriba un nombre de usuario y la contraseña entre comillas en las líneas 14 y 15, tal y como se muestra a continuación, y después guarde el archivo.</span><span class="sxs-lookup"><span data-stu-id="f0669-208">Open the `hub-adds.json` file and enter a username and password between the quotes in lines 14 and 15, as shown below, then save the file.</span></span>

   ```bash
   "adminUsername": "XXX",
   "adminPassword": "YYY",
   ```

2. <span data-ttu-id="f0669-209">Ejecute `azbb` para implementar los controladores de dominio de ADDS tal y como se muestra a continuación.</span><span class="sxs-lookup"><span data-stu-id="f0669-209">Run `azbb` to deploy the ADDS domain controllers as shown below.</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-adds-rg - l <location> -p hub-adds.json --deploy
   ```
  
   > [!NOTE]
   > <span data-ttu-id="f0669-210">Si decide usar un nombre del grupo de recursos distinto (que no sea `hub-adds-rg`), asegúrese de que busca todos los archivos de parámetros que usan ese nombre y de que los modifica para usar el nombre de su propio grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="f0669-210">If you decide to use a different resource group name (other than `hub-adds-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

   > [!NOTE]
   > <span data-ttu-id="f0669-211">Esta parte de la implementación puede tardar varios minutos, ya que requiere unir las dos máquinas virtuales al dominio hospedado en el centro de datos local simulado y, a continuación, instalar AD DS en ellas.</span><span class="sxs-lookup"><span data-stu-id="f0669-211">This part of the deployment may take several minutes, since it requires joining the two VMs to the domain hosted int he simulated on-premises datacenter, then installing AD DS on them.</span></span>

### <a name="nva"></a><span data-ttu-id="f0669-212">Aplicación virtual de red</span><span class="sxs-lookup"><span data-stu-id="f0669-212">NVA</span></span>

<span data-ttu-id="f0669-213">Para implementar una aplicación virtual de red en la subred `dmz`, realice los pasos siguientes:</span><span class="sxs-lookup"><span data-stu-id="f0669-213">To deploy an NVA in the `dmz` subnet, perform the following steps:</span></span>

1. <span data-ttu-id="f0669-214">Abra el archivo `hub-nva.json` y escriba un nombre de usuario y la contraseña entre comillas en las líneas 13 y 14, tal y como se muestra a continuación, y después guarde el archivo.</span><span class="sxs-lookup"><span data-stu-id="f0669-214">Open the `hub-nva.json` file and enter a username and password between the quotes in lines 13 and 14, as shown below, then save the file.</span></span>

   ```bash
   "adminUsername": "XXX",
   "adminPassword": "YYY",
   ```
2. <span data-ttu-id="f0669-215">Ejecute `azbb` para implementar la máquina virtual de la aplicación virtual de red y las rutas definidas por el usuario.</span><span class="sxs-lookup"><span data-stu-id="f0669-215">Run `azbb` to deploy the NVA VM and user defined routes.</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-nva-rg - l <location> -p hub-nva.json --deploy
   ```
   > [!NOTE]
   > <span data-ttu-id="f0669-216">Si decide usar un nombre del grupo de recursos distinto (que no sea `hub-nva-rg`), asegúrese de que busca todos los archivos de parámetros que usan ese nombre y de que los modifica para usar el nombre de su propio grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="f0669-216">If you decide to use a different resource group name (other than `hub-nva-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

### <a name="azure-spoke-vnets"></a><span data-ttu-id="f0669-217">Redes virtuales de radios de Azure</span><span class="sxs-lookup"><span data-stu-id="f0669-217">Azure spoke VNets</span></span>

<span data-ttu-id="f0669-218">Para implementar las redes virtuales de radios, siga estos pasos.</span><span class="sxs-lookup"><span data-stu-id="f0669-218">To deploy the spoke VNets, perform the following steps.</span></span>

1. <span data-ttu-id="f0669-219">Abra el archivo `spoke1.json` y escriba un nombre de usuario y la contraseña entre comillas en las líneas 52 y 53, tal y como se muestra a continuación, y después guarde el archivo.</span><span class="sxs-lookup"><span data-stu-id="f0669-219">Open the `spoke1.json` file and enter a username and password between the quotes in lines 52 and 53, as shown below, then save the file.</span></span>

   ```bash
   "adminUsername": "XXX",
   "adminPassword": "YYY",
   ```

2. <span data-ttu-id="f0669-220">En la línea 54, para `osType`, escriba `Windows` o `Linux` para instalar Windows Server 2016 Datacenter o Ubuntu 16.04 como sistema operativo de JumpBox.</span><span class="sxs-lookup"><span data-stu-id="f0669-220">On line 54, for `osType`, type `Windows` or `Linux` to install either Windows Server 2016 Datacenter, or Ubuntu 16.04 as the operating system for the jumpbox.</span></span>

3. <span data-ttu-id="f0669-221">Ejecute `azbb` para implementar el entorno de red virtual del primer radio tal y como se muestra a continuación.</span><span class="sxs-lookup"><span data-stu-id="f0669-221">Run `azbb` to deploy the first spoke VNet environment as shown below.</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke1-vnet-rg - l <location> -p spoke1.json --deploy
   ```
  
   > [!NOTE]
   > <span data-ttu-id="f0669-222">Si decide usar un nombre del grupo de recursos distinto (que no sea `spoke1-vnet-rg`), asegúrese de que busca todos los archivos de parámetros que usan ese nombre y de que los modifica para usar el nombre de su propio grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="f0669-222">If you decide to use a different resource group name (other than `spoke1-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

4. <span data-ttu-id="f0669-223">Repita el paso 1 anterior para el archivo `spoke2.json`.</span><span class="sxs-lookup"><span data-stu-id="f0669-223">Repeat step 1 above for file `spoke2.json`.</span></span>

5. <span data-ttu-id="f0669-224">Ejecute `azbb` para implementar el entorno de red virtual del segundo radio tal y como se muestra a continuación.</span><span class="sxs-lookup"><span data-stu-id="f0669-224">Run `azbb` to deploy the second spoke VNet environment as shown below.</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke2-vnet-rg - l <location> -p spoke2.json --deploy
   ```
   > [!NOTE]
   > <span data-ttu-id="f0669-225">Si decide usar un nombre del grupo de recursos distinto (que no sea `spoke2-vnet-rg`), asegúrese de que busca todos los archivos de parámetros que usan ese nombre y de que los modifica para usar el nombre de su propio grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="f0669-225">If you decide to use a different resource group name (other than `spoke2-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

### <a name="azure-hub-vnet-peering-to-spoke-vnets"></a><span data-ttu-id="f0669-226">Emparejamiento de VNET del concentrador de Azure a las redes virtuales de los radios</span><span class="sxs-lookup"><span data-stu-id="f0669-226">Azure hub VNet peering to spoke VNets</span></span>

<span data-ttu-id="f0669-227">Para crear una conexión de emparejamiento desde la red virtual del concentrador a las redes virtuales de radios, realice los siguientes pasos.</span><span class="sxs-lookup"><span data-stu-id="f0669-227">To create a peering connection from the hub VNet to the spoke VNets, perform the following steps.</span></span>

1. <span data-ttu-id="f0669-228">Abra el archivo `hub-vnet-peering.json` y compruebe que el nombre del grupo de recursos y el nombre de la red virtual para cada uno de los emparejamientos de la red virtual que empiezan en la línea 29 son correctos.</span><span class="sxs-lookup"><span data-stu-id="f0669-228">Open the `hub-vnet-peering.json` file and verify that the resource group name, and virtual network name for each of the virtual network peerings starting in line 29 are correct.</span></span>

2. <span data-ttu-id="f0669-229">Ejecute `azbb` para implementar el entorno de red virtual del primer radio tal y como se muestra a continuación.</span><span class="sxs-lookup"><span data-stu-id="f0669-229">Run `azbb` to deploy the first spoke VNet environment as shown below.</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg - l <location> -p hub-vnet-peering.json --deploy
   ```

   > [!NOTE]
   > <span data-ttu-id="f0669-230">Si decide usar un nombre del grupo de recursos distinto (que no sea `hub-vnet-rg`), asegúrese de que busca todos los archivos de parámetros que usan ese nombre y de que los modifica para usar el nombre de su propio grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="f0669-230">If you decide to use a different resource group name (other than `hub-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

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
[virtual datacenter]: https://aka.ms/vdc
[vnet-peering]: /azure/virtual-network/virtual-network-peering-overview
[vnet-peering-limit]: /azure/azure-subscription-service-limits#networking-limits
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[windows-vm-ra]: ../virtual-machines-windows/index.md

[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/shared-services.png "Topología de servicios compartidos en Azure"
[3]: ./images/hub-spokehub-spoke.svg "Topología en estrella tipo hub-hub-and-spoke-spoke en Azure"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
