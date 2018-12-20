---
title: Implementación de una topología de red en estrella tipo hub-and-spoke
titleSuffix: Azure Reference Architectures
description: Implemente una topología de red en estrella tipo hub-and-spoke con servicios compartidos en Azure.
author: telmosampaio
ms.date: 10/09/2018
ms.custom: seodec18
ms.openlocfilehash: 37ae02d8ef02f64329d5e5215e5a32df9f0f9491
ms.sourcegitcommit: 88a68c7e9b6b772172b7faa4b9fd9c061a9f7e9d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/08/2018
ms.locfileid: "53120465"
---
# <a name="implement-a-hub-spoke-network-topology-with-shared-services-in-azure"></a><span data-ttu-id="0f5ba-103">Implementación de una topología de red en estrella tipo hub-and-spoke con servicios compartidos en Azure</span><span class="sxs-lookup"><span data-stu-id="0f5ba-103">Implement a hub-spoke network topology with shared services in Azure</span></span>

<span data-ttu-id="0f5ba-104">Esta arquitectura de referencia se basa en la arquitectura de referencia tipo [hub-and-spoke][guidance-hub-spoke] para incluir servicios compartidos en el centro que puedan consumir todos los radios.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-104">This reference architecture builds on the [hub-spoke][guidance-hub-spoke] reference architecture to include shared services in the hub that can be consumed by all spokes.</span></span> <span data-ttu-id="0f5ba-105">Como primer paso para migrar un centro de datos a la nube y generar un [centro de datos virtual], los primeros servicios que necesitará compartir son los de identidad y seguridad.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-105">As a first step toward migrating a datacenter to the cloud, and building a [virtual datacenter], the first services you need to share are identity and security.</span></span> <span data-ttu-id="0f5ba-106">Esta arquitectura de referencia muestra cómo ampliar los servicios de Active Directory desde el centro de datos local a Azure y cómo agregar una aplicación virtual de red (NVA) que pueda actuar como un firewall, en una topología en estrella tipo hub-and-spoke.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-106">This reference architecture shows you how to extend your Active Directory services from your on-premises datacenter to Azure, and how to add a network virtual appliance (NVA) that can act as a firewall, in a hub-spoke topology.</span></span>  <span data-ttu-id="0f5ba-107">[**Implemente esta solución**](#deploy-the-solution).</span><span class="sxs-lookup"><span data-stu-id="0f5ba-107">[**Deploy this solution**](#deploy-the-solution).</span></span>

![Topología de servicios compartidos en Azure](./images/shared-services.png)

<span data-ttu-id="0f5ba-109">*Descargue un [archivo Visio][visio-download] de esta arquitectura.*</span><span class="sxs-lookup"><span data-stu-id="0f5ba-109">*Download a [Visio file][visio-download] of this architecture*</span></span>

<span data-ttu-id="0f5ba-110">Las ventajas de esta topología incluyen:</span><span class="sxs-lookup"><span data-stu-id="0f5ba-110">The benefits of this topology include:</span></span>

- <span data-ttu-id="0f5ba-111">**Ahorros en costos** gracias a la centralización de los servicios que se pueden compartir entre varias cargas de trabajo, como las aplicaciones virtuales de red (NVA) y los servidores DNS, en una única ubicación.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-111">**Cost savings** by centralizing services that can be shared by multiple workloads, such as network virtual appliances (NVAs) and DNS servers, in a single location.</span></span>
- <span data-ttu-id="0f5ba-112">**Superar los límites de las suscripciones** gracias al emparejamiento de las redes virtuales de diferentes suscripciones con el concentrador central.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-112">**Overcome subscriptions limits** by peering VNets from different subscriptions to the central hub.</span></span>
- <span data-ttu-id="0f5ba-113">**Separación de intereses** entre la TI central (SecOps e InfraOps) y las cargas de trabajo (DevOps).</span><span class="sxs-lookup"><span data-stu-id="0f5ba-113">**Separation of concerns** between central IT (SecOps, InfraOps) and workloads (DevOps).</span></span>

<span data-ttu-id="0f5ba-114">Los usos habituales de esta arquitectura incluyen:</span><span class="sxs-lookup"><span data-stu-id="0f5ba-114">Typical uses for this architecture include:</span></span>

- <span data-ttu-id="0f5ba-115">Cargas de trabajo implementadas en distintos entornos, como desarrollo, pruebas y producción, que requieren servicios compartidos como DNS, IDS, NTP o AD DS.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-115">Workloads deployed in different environments, such as development, testing, and production, that require shared services such as DNS, IDS, NTP, or AD DS.</span></span> <span data-ttu-id="0f5ba-116">Los servicios compartidos se colocan en la red virtual del concentrador, mientras que cada entorno se implementa en un radio para mantener el aislamiento.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-116">Shared services are placed in the hub VNet, while each environment is deployed to a spoke to maintain isolation.</span></span>
- <span data-ttu-id="0f5ba-117">Cargas de trabajo que no requieren conectividad entre sí, pero requieren acceso a los servicios compartidos.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-117">Workloads that do not require connectivity to each other, but require access to shared services.</span></span>
- <span data-ttu-id="0f5ba-118">Empresas que requieren el control centralizado sobre aspectos de seguridad, como un firewall en el concentrador como una red perimetral, así como la administración segregada de las cargas de trabajo en cada radio.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-118">Enterprises that require central control over security aspects, such as a firewall in the hub as a DMZ, and segregated management for the workloads in each spoke.</span></span>

## <a name="architecture"></a><span data-ttu-id="0f5ba-119">Arquitectura</span><span class="sxs-lookup"><span data-stu-id="0f5ba-119">Architecture</span></span>

<span data-ttu-id="0f5ba-120">La arquitectura consta de los siguientes componentes:</span><span class="sxs-lookup"><span data-stu-id="0f5ba-120">The architecture consists of the following components.</span></span>

- <span data-ttu-id="0f5ba-121">**Red local**.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-121">**On-premises network**.</span></span> <span data-ttu-id="0f5ba-122">Una red de área local privada que se ejecuta dentro de una organización.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-122">A private local-area network running within an organization.</span></span>

- <span data-ttu-id="0f5ba-123">**Dispositivo VPN**.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-123">**VPN device**.</span></span> <span data-ttu-id="0f5ba-124">Un dispositivo o servicio que proporciona conectividad externa a la red local.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-124">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="0f5ba-125">El dispositivo VPN puede ser un dispositivo de hardware, o puede ser una solución de software como el servicio de Enrutamiento y acceso remoto (RRAS) en Windows Server 2012.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-125">The VPN device may be a hardware device, or a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="0f5ba-126">Para obtener una lista de dispositivos VPN compatibles e información acerca de cómo configurar dispositivos VPN seleccionados para conectarse a Azure, consulte [Acerca de los dispositivos VPN y los parámetros de IPsec/IKE para conexiones de VPN Gateway de sitio a sitio][vpn-appliance].</span><span class="sxs-lookup"><span data-stu-id="0f5ba-126">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

- <span data-ttu-id="0f5ba-127">**Puerta de enlace de red virtual de VPN o puerta de enlace de ExpressRoute**.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-127">**VPN virtual network gateway or ExpressRoute gateway**.</span></span> <span data-ttu-id="0f5ba-128">La puerta de enlace de red virtual permite que la red virtual se conecte al dispositivo VPN o al circuito de ExpressRoute que se usa para la conectividad con la red local.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-128">The virtual network gateway enables the VNet to connect to the VPN device, or ExpressRoute circuit, used for connectivity with your on-premises network.</span></span> <span data-ttu-id="0f5ba-129">Para más información, consulte [Conectar una red local con una red virtual de Microsoft Azure][connect-to-an-Azure-vnet].</span><span class="sxs-lookup"><span data-stu-id="0f5ba-129">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

> [!NOTE]
> <span data-ttu-id="0f5ba-130">Los scripts de implementación para esta arquitectura de referencia usan una instancia de VPN Gateway para la conectividad y una red virtual en Azure para simular la red local.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-130">The deployment scripts for this reference architecture use a VPN gateway for connectivity, and a VNet in Azure to simulate your on-premises network.</span></span>

- <span data-ttu-id="0f5ba-131">**Red virtual del concentrador**.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-131">**Hub VNet**.</span></span> <span data-ttu-id="0f5ba-132">La red virtual de Azure utilizada como el concentrador en la topología en estrella tipo hub-and-spoke.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-132">Azure VNet used as the hub in the hub-spoke topology.</span></span> <span data-ttu-id="0f5ba-133">El concentrador es el punto central de conectividad a la red local y un lugar para hospedar los servicios que se pueden usar en las diferentes cargas de trabajo hospedadas en las redes virtuales de los radios.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-133">The hub is the central point of connectivity to your on-premises network, and a place to host services that can be consumed by the different workloads hosted in the spoke VNets.</span></span>

- <span data-ttu-id="0f5ba-134">**Subred de puerta de enlace**.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-134">**Gateway subnet**.</span></span> <span data-ttu-id="0f5ba-135">Las puertas de enlace de red virtual se conservan en la misma subred.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-135">The virtual network gateways are held in the same subnet.</span></span>

- <span data-ttu-id="0f5ba-136">**Subred de servicios compartidos**.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-136">**Shared services subnet**.</span></span> <span data-ttu-id="0f5ba-137">Una subred de la red virtual del concentrador utilizada para hospedar servicios que se pueden compartir entre todos los radios, como DNS o AD DS.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-137">A subnet in the hub VNet used to host services that can be shared among all spokes, such as DNS or AD DS.</span></span>

- <span data-ttu-id="0f5ba-138">**Subred DMZ**.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-138">**DMZ subnet**.</span></span> <span data-ttu-id="0f5ba-139">Una subred de la red virtual del centro que se usa para hospedar aplicaciones virtuales de red (NVA) que pueden actuar como aplicaciones de seguridad como, por ejemplo, firewalls.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-139">A subnet in the hub VNet used to host NVAs that can act as security appliances, such as firewalls.</span></span>

- <span data-ttu-id="0f5ba-140">**Redes virtuales de radios**.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-140">**Spoke VNets**.</span></span> <span data-ttu-id="0f5ba-141">Una o varias redes virtuales de Azure que se usan como radios en la topología en estrella tipo hub-and-spoke.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-141">One or more Azure VNets that are used as spokes in the hub-spoke topology.</span></span> <span data-ttu-id="0f5ba-142">Los radios pueden utilizarse para aislar las cargas de trabajo en sus propias redes virtuales, administradas por separado desde otros radios.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-142">Spokes can be used to isolate workloads in their own VNets, managed separately from other spokes.</span></span> <span data-ttu-id="0f5ba-143">Cada carga de trabajo puede incluir varios niveles, con varias subredes que se conectan a través de equilibradores de carga de Azure.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-143">Each workload might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="0f5ba-144">Para obtener más información acerca de la infraestructura de aplicaciones, consulte [Ejecutar cargas de trabajo de máquinas virtuales Windows][windows-vm-ra] y [Ejecutar cargas de trabajo de máquinas virtuales Linux][linux-vm-ra].</span><span class="sxs-lookup"><span data-stu-id="0f5ba-144">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

- <span data-ttu-id="0f5ba-145">**Emparejamiento de VNET**.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-145">**VNet peering**.</span></span> <span data-ttu-id="0f5ba-146">Se pueden conectar dos redes virtuales en la misma región de Azure mediante una [conexión de emparejamiento][vnet-peering].</span><span class="sxs-lookup"><span data-stu-id="0f5ba-146">Two VNets in the same Azure region can be connected using a [peering connection][vnet-peering].</span></span> <span data-ttu-id="0f5ba-147">Las conexiones de emparejamiento son conexiones no transitivas de baja latencia entre las redes virtuales.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-147">Peering connections are non-transitive, low latency connections between VNets.</span></span> <span data-ttu-id="0f5ba-148">Una vez establecido el emparejamiento, las redes virtuales intercambian el tráfico mediante la red troncal de Azure, sin necesidad de un enrutador.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-148">Once peered, the VNets exchange traffic by using the Azure backbone, without the need for a router.</span></span> <span data-ttu-id="0f5ba-149">En una topología de red en estrella tipo hub-and-spoke, el emparejamiento de VNET se usa para conectar el concentrador a cada radio.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-149">In a hub-spoke network topology, you use VNet peering to connect the hub to each spoke.</span></span>

> [!NOTE]
> <span data-ttu-id="0f5ba-150">En este artículo solo se tratan las implementaciones de [Resource Manager](/azure/azure-resource-manager/resource-group-overview), pero también puede conectar una red virtual clásica a una red virtual de Resource Manager en la misma suscripción.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-150">This article only covers [Resource Manager](/azure/azure-resource-manager/resource-group-overview) deployments, but you can also connect a classic VNet to a Resource Manager VNet in the same subscription.</span></span> <span data-ttu-id="0f5ba-151">De este modo, los radios pueden hospedar implementaciones clásicas y seguir beneficiándose de los servicios compartidos en el concentrador.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-151">That way, your spokes can host classic deployments and still benefit from services shared in the hub.</span></span>

## <a name="recommendations"></a><span data-ttu-id="0f5ba-152">Recomendaciones</span><span class="sxs-lookup"><span data-stu-id="0f5ba-152">Recommendations</span></span>

<span data-ttu-id="0f5ba-153">Todas las recomendaciones de la arquitectura de referencia de tipo [hub-and-spoke][guidance-hub-spoke] también se aplican a la arquitectura de referencia de los servicios compartidos.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-153">All the recommendations for the [hub-spoke][guidance-hub-spoke] reference architecture also apply to the shared services reference architecture.</span></span>

<span data-ttu-id="0f5ba-154">Igualmente, las siguientes recomendaciones sirven para la mayoría de escenarios de servicios compartidos.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-154">Also, the following recommendations apply for most scenarios under shared services.</span></span> <span data-ttu-id="0f5ba-155">Sígalas a menos que tenga un requisito concreto que las invalide.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-155">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="identity"></a><span data-ttu-id="0f5ba-156">Identidad</span><span class="sxs-lookup"><span data-stu-id="0f5ba-156">Identity</span></span>

<span data-ttu-id="0f5ba-157">La mayoría de las organizaciones empresariales tienen un entorno de servicios de directorio de Active Directory (ADDS) en su centro de datos local.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-157">Most enterprise organizations have an Active Directory Directory Services (ADDS) environment in their on-premises datacenter.</span></span> <span data-ttu-id="0f5ba-158">Para facilitar la administración de los recursos trasladados a Azure desde la red local que depende de ADDS, se recomienda hospedar los controladores de dominio de ADDS en Azure.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-158">To facilitate management of assets moved to Azure from your on-premises network that depend on ADDS, it is recommended to host ADDS domain controllers in Azure.</span></span>

<span data-ttu-id="0f5ba-159">Si desea utilizar los objetos de la directiva de grupo que desea controlar de forma independiente para Azure y para el entorno local, use un sitio de AD diferente para cada región de Azure.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-159">If you make use of Group Policy Objects, that you want to control separately for Azure and your on-premises environment, use a different AD site for each Azure region.</span></span> <span data-ttu-id="0f5ba-160">Sitúe los controladores de dominio en una red virtual central (centro) al que puedan acceder las cargas de trabajo dependientes.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-160">Place your domain controllers in a central VNet (hub) that dependent workloads can access.</span></span>

### <a name="security"></a><span data-ttu-id="0f5ba-161">Seguridad</span><span class="sxs-lookup"><span data-stu-id="0f5ba-161">Security</span></span>

<span data-ttu-id="0f5ba-162">Al trasladar cargas de trabajo desde el entorno local a Azure, algunas de estas cargas necesitarán hospedarse en máquinas virtuales.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-162">As you move workloads from your on-premises environment to Azure, some of these workloads will require to be hosted in VMs.</span></span> <span data-ttu-id="0f5ba-163">Por motivos de cumplimiento normativo, puede que tenga que aplicar restricciones al tráfico que recorre esas cargas de trabajo.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-163">For compliance reasons, you may need to enforce restrictions on traffic traversing those workloads.</span></span>

<span data-ttu-id="0f5ba-164">Puede usar aplicaciones virtuales de red (NVA) en Azure para hospedar diferentes tipos de servicios de seguridad y rendimiento.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-164">You can use network virtual appliances (NVAs) in Azure to host different types of security and performance services.</span></span> <span data-ttu-id="0f5ba-165">Si está familiarizado con un conjunto determinado de aplicaciones locales actualmente, se recomienda que use las mismas aplicaciones virtualizadas en Azure, si es posible.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-165">If you are familiar with a given set of appliances on-premises today, it is recommended to use the same virtualized appliances in Azure, where applicable.</span></span>

> [!NOTE]
> <span data-ttu-id="0f5ba-166">Los scripts de implementación de esta arquitectura de referencia usan una máquina virtual Ubuntu con reenvío IP habilitado para imitar una aplicación virtual de red.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-166">The deployment scripts for this reference architecture use an Ubuntu VM with IP forwarding enabled to mimic a network virtual appliance.</span></span>

## <a name="considerations"></a><span data-ttu-id="0f5ba-167">Consideraciones</span><span class="sxs-lookup"><span data-stu-id="0f5ba-167">Considerations</span></span>

### <a name="overcoming-vnet-peering-limits"></a><span data-ttu-id="0f5ba-168">Superación de los límites de emparejamiento de VNET</span><span class="sxs-lookup"><span data-stu-id="0f5ba-168">Overcoming VNet peering limits</span></span>

<span data-ttu-id="0f5ba-169">Asegúrese de tener en cuenta la [limitación del número de emparejamientos de VNET por cada red virtual][vnet-peering-limit] en Azure.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-169">Make sure you consider the [limitation on number of VNets peerings per VNet][vnet-peering-limit] in Azure.</span></span> <span data-ttu-id="0f5ba-170">Si decide que necesita más radios de los que el límite permitirá, considere la posibilidad de crear una topología en estrella tipo hub-hub-and-spoke-spoke, donde el primer nivel de radios también actúa como concentradores.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-170">If you decide you need more spokes than the limit will allow, consider creating a hub-spoke-hub-spoke topology, where the first level of spokes also act as hubs.</span></span> <span data-ttu-id="0f5ba-171">En el siguiente diagrama se ilustra este enfoque.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-171">The following diagram shows this approach.</span></span>

![Topología en estrella tipo hub-and-spoke en Azure](./images/hub-spokehub-spoke.svg)

<span data-ttu-id="0f5ba-173">También tenga en cuenta qué servicios se comparten en el concentrador, para asegurarse de que el concentrador se escala para tener un número mayor de radios.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-173">Also consider what services are shared in the hub, to ensure the hub scales for a larger number of spokes.</span></span> <span data-ttu-id="0f5ba-174">Por ejemplo, si el concentrador proporciona servicios de firewall, tenga en cuenta los límites de ancho de banda de la solución de firewall al agregar varios radios.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-174">For instance, if your hub provides firewall services, consider the bandwidth limits of your firewall solution when adding multiple spokes.</span></span> <span data-ttu-id="0f5ba-175">Puede mover algunos de estos servicios compartidos a un segundo nivel de concentradores.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-175">You might want to move some of these shared services to a second level of hubs.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="0f5ba-176">Implementación de la solución</span><span class="sxs-lookup"><span data-stu-id="0f5ba-176">Deploy the solution</span></span>

<span data-ttu-id="0f5ba-177">Hay disponible una implementación de esta arquitectura en [GitHub][ref-arch-repo].</span><span class="sxs-lookup"><span data-stu-id="0f5ba-177">A deployment for this architecture is available on [GitHub][ref-arch-repo].</span></span> <span data-ttu-id="0f5ba-178">La implementación crea los siguientes grupos de recursos en su suscripción:</span><span class="sxs-lookup"><span data-stu-id="0f5ba-178">The deployment creates the following resource groups in your subscription:</span></span>

- <span data-ttu-id="0f5ba-179">hub-adds-rg</span><span class="sxs-lookup"><span data-stu-id="0f5ba-179">hub-adds-rg</span></span>
- <span data-ttu-id="0f5ba-180">hub-nva-rg</span><span class="sxs-lookup"><span data-stu-id="0f5ba-180">hub-nva-rg</span></span>
- <span data-ttu-id="0f5ba-181">hub-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="0f5ba-181">hub-vnet-rg</span></span>
- <span data-ttu-id="0f5ba-182">onprem-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="0f5ba-182">onprem-vnet-rg</span></span>
- <span data-ttu-id="0f5ba-183">spoke1-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="0f5ba-183">spoke1-vnet-rg</span></span>
- <span data-ttu-id="0f5ba-184">spoke2-vent-rg</span><span class="sxs-lookup"><span data-stu-id="0f5ba-184">spoke2-vent-rg</span></span>

<span data-ttu-id="0f5ba-185">Los archivos de parámetro de plantilla hacen referencia a estos nombres, por lo que si se cambian es necesario actualizar los archivos de parámetro para que coincidan.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-185">The template parameter files refer to these names, so if you change them, update the parameter files to match.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="0f5ba-186">Requisitos previos</span><span class="sxs-lookup"><span data-stu-id="0f5ba-186">Prerequisites</span></span>

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-the-simulated-on-premises-datacenter-using-azbb"></a><span data-ttu-id="0f5ba-187">Implementación del centro de datos local simulado mediante azbb</span><span class="sxs-lookup"><span data-stu-id="0f5ba-187">Deploy the simulated on-premises datacenter using azbb</span></span>

<span data-ttu-id="0f5ba-188">En este paso se implementa el centro de datos local simulado como una red virtual de Azure.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-188">This step deploys the simulated on-premises datacenter as an Azure VNet.</span></span>

1. <span data-ttu-id="0f5ba-189">Vaya a la carpeta `hybrid-networking\shared-services-stack\` del repositorio de GitHub.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-189">Navigate to the `hybrid-networking\shared-services-stack\` folder of the GitHub repository.</span></span>

2. <span data-ttu-id="0f5ba-190">Abra el archivo `onprem.json` .</span><span class="sxs-lookup"><span data-stu-id="0f5ba-190">Open the `onprem.json` file.</span></span>

3. <span data-ttu-id="0f5ba-191">Busque todas las instancias de `UserName`, `adminUserName`,`Password` y `adminPassword`.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-191">Search for all instances of `UserName`, `adminUserName`,`Password`, and `adminPassword`.</span></span> <span data-ttu-id="0f5ba-192">Escriba los valores del nombre de usuario y la contraseña en los parámetros y guarde el archivo.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-192">Enter values for the user name and password in the parameters and save the file.</span></span>

4. <span data-ttu-id="0f5ba-193">Ejecute el siguiente comando:</span><span class="sxs-lookup"><span data-stu-id="0f5ba-193">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g onprem-vnet-rg -l <location> -p onprem.json --deploy
   ```

5. <span data-ttu-id="0f5ba-194">Espere a que finalice la implementación.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-194">Wait for the deployment to finish.</span></span> <span data-ttu-id="0f5ba-195">Esta implementación crea una red virtual, una máquina virtual Windows y una instancia de VPN Gateway.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-195">This deployment creates a virtual network, a virtual machine running Windows, and a VPN gateway.</span></span> <span data-ttu-id="0f5ba-196">La creación de la instancia de VPN Gateway puede durar más de cuarenta minutos.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-196">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="deploy-the-hub-vnet"></a><span data-ttu-id="0f5ba-197">Implementación de la red virtual del concentrador</span><span class="sxs-lookup"><span data-stu-id="0f5ba-197">Deploy the hub VNet</span></span>

<span data-ttu-id="0f5ba-198">En este paso se implementa la red virtual de concentrador y la conecta con la red virtual local simulada.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-198">This step deploys the hub VNet and connects it to the simulated on-premises VNet.</span></span>

1. <span data-ttu-id="0f5ba-199">Abra el archivo `hub-vnet.json` .</span><span class="sxs-lookup"><span data-stu-id="0f5ba-199">Open the `hub-vnet.json` file.</span></span>

2. <span data-ttu-id="0f5ba-200">Busque `adminPassword` y escriba un nombre de usuario y una contraseña en los parámetros.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-200">Search for `adminPassword` and enter a user name and password in the parameters.</span></span>

3. <span data-ttu-id="0f5ba-201">Busque todas las instancias de `sharedKey` y escriba un valor de clave compartida.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-201">Search for all instances of `sharedKey` and enter a value for a shared key.</span></span> <span data-ttu-id="0f5ba-202">Guarde el archivo.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-202">Save the file.</span></span>

   ```json
   "sharedKey": "abc123",
   ```

4. <span data-ttu-id="0f5ba-203">Ejecute el siguiente comando:</span><span class="sxs-lookup"><span data-stu-id="0f5ba-203">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet.json --deploy
   ```

5. <span data-ttu-id="0f5ba-204">Espere a que finalice la implementación.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-204">Wait for the deployment to finish.</span></span> <span data-ttu-id="0f5ba-205">Esta implementación crea una red virtual, una máquina virtual, una instancia de VPN Gateway y una conexión a la puerta de enlace creada en la sección anterior.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-205">This deployment creates a virtual network, a virtual machine, a VPN gateway, and a connection to the gateway created in the previous section.</span></span> <span data-ttu-id="0f5ba-206">La puerta de enlace de VPN puede tardar más de 40 minutos en completarse.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-206">The VPN gateway can take more than 40 minutes to complete.</span></span>

### <a name="deploy-ad-ds-in-azure"></a><span data-ttu-id="0f5ba-207">Implementación de AD DS en Azure</span><span class="sxs-lookup"><span data-stu-id="0f5ba-207">Deploy AD DS in Azure</span></span>

<span data-ttu-id="0f5ba-208">En este paso se implementan los controladores de dominio de AD DS en Azure.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-208">This step deploys AD DS domain controllers in Azure.</span></span>

1. <span data-ttu-id="0f5ba-209">Abra el archivo `hub-adds.json` .</span><span class="sxs-lookup"><span data-stu-id="0f5ba-209">Open the `hub-adds.json` file.</span></span>

2. <span data-ttu-id="0f5ba-210">Busque todas las instancias de `Password` y `adminPassword`.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-210">Search for all instances of `Password` and `adminPassword`.</span></span> <span data-ttu-id="0f5ba-211">Escriba los valores del nombre de usuario y la contraseña en los parámetros y guarde el archivo.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-211">Enter values for the user name and password in the parameters and save the file.</span></span>

3. <span data-ttu-id="0f5ba-212">Ejecute el siguiente comando:</span><span class="sxs-lookup"><span data-stu-id="0f5ba-212">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-adds-rg -l <location> -p hub-adds.json --deploy
   ```

<span data-ttu-id="0f5ba-213">Este paso de implementación puede tardar varios minutos, porque combina las dos máquinas virtuales en el dominio hospedado en el centro de datos simulados del entorno local, e instala AD DS en ellas.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-213">This deployment step may take several minutes, because it joins the two VMs to the domain hosted in the simulated on-premises datacenter, and installs AD DS on them.</span></span>

### <a name="deploy-the-spoke-vnets"></a><span data-ttu-id="0f5ba-214">Implementación de redes virtuales de radios</span><span class="sxs-lookup"><span data-stu-id="0f5ba-214">Deploy the spoke VNets</span></span>

<span data-ttu-id="0f5ba-215">Este paso implementa las redes virtuales de radio.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-215">This step deploys the spoke VNets.</span></span>

1. <span data-ttu-id="0f5ba-216">Abra el archivo `spoke1.json` .</span><span class="sxs-lookup"><span data-stu-id="0f5ba-216">Open the `spoke1.json` file.</span></span>

2. <span data-ttu-id="0f5ba-217">Busque `adminPassword` y escriba un nombre de usuario y una contraseña en los parámetros.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-217">Search for `adminPassword` and enter a user name and password in the parameters.</span></span>

3. <span data-ttu-id="0f5ba-218">Ejecute el siguiente comando:</span><span class="sxs-lookup"><span data-stu-id="0f5ba-218">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke1-vnet-rg -l <location> -p spoke1.json --deploy
   ```

4. <span data-ttu-id="0f5ba-219">Repita los pasos 1 y 2 para el archivo `spoke2.json`.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-219">Repeat steps 1 and 2 for the file `spoke2.json`.</span></span>

5. <span data-ttu-id="0f5ba-220">Ejecute el siguiente comando:</span><span class="sxs-lookup"><span data-stu-id="0f5ba-220">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke2-vnet-rg -l <location> -p spoke2.json --deploy
   ```

### <a name="peer-the-hub-vnet-to-the-spoke-vnets"></a><span data-ttu-id="0f5ba-221">Emparejamiento de la red virtual de concentrador con las redes virtuales de radio</span><span class="sxs-lookup"><span data-stu-id="0f5ba-221">Peer the hub VNet to the spoke VNets</span></span>

<span data-ttu-id="0f5ba-222">Para crear una conexión de emparejamiento desde la red virtual de concentrador con las redes virtuales de radio, ejecute el comando siguiente:</span><span class="sxs-lookup"><span data-stu-id="0f5ba-222">To create a peering connection from the hub VNet to the spoke VNets, run the following command:</span></span>

```bash
azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet-peering.json --deploy
```

### <a name="deploy-the-nva"></a><span data-ttu-id="0f5ba-223">Implementación de la aplicación virtual de red</span><span class="sxs-lookup"><span data-stu-id="0f5ba-223">Deploy the NVA</span></span>

<span data-ttu-id="0f5ba-224">Este paso implementa una aplicación virtual de red en la subred `dmz`.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-224">This step deploys an NVA in the `dmz` subnet.</span></span>

1. <span data-ttu-id="0f5ba-225">Abra el archivo `hub-nva.json` .</span><span class="sxs-lookup"><span data-stu-id="0f5ba-225">Open the `hub-nva.json` file.</span></span>

2. <span data-ttu-id="0f5ba-226">Busque `adminPassword` y escriba un nombre de usuario y una contraseña en los parámetros.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-226">Search for `adminPassword` and enter a user name and password in the parameters.</span></span>

3. <span data-ttu-id="0f5ba-227">Ejecute el siguiente comando:</span><span class="sxs-lookup"><span data-stu-id="0f5ba-227">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-nva-rg -l <location> -p hub-nva.json --deploy
   ```

### <a name="test-connectivity"></a><span data-ttu-id="0f5ba-228">Comprobación de la conectividad</span><span class="sxs-lookup"><span data-stu-id="0f5ba-228">Test connectivity</span></span>

<span data-ttu-id="0f5ba-229">Pruebe la conectividad desde el entorno local simulado a la red virtual del concentrador.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-229">Test conectivity from the simulated on-premises environment to the hub VNet.</span></span>

1. <span data-ttu-id="0f5ba-230">Use Azure Portal para encontrar la máquina virtual denominada `jb-vm1` en el grupo de recursos `onprem-jb-rg`.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-230">Use the Azure portal to find the VM named `jb-vm1` in the `onprem-jb-rg` resource group.</span></span>

2. <span data-ttu-id="0f5ba-231">Haga clic en `Connect` para abrir una sesión de escritorio remoto en la máquina virtual.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-231">Click `Connect` to open a remote desktop session to the VM.</span></span> <span data-ttu-id="0f5ba-232">Usar la contraseña que especificó en el `onprem.json` archivo de parámetros.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-232">Use the password that you specified in the `onprem.json` parameter file.</span></span>

3. <span data-ttu-id="0f5ba-233">Abra una consola de PowerShell en la máquina virtual y utilice el cmdlet `Test-NetConnection` para comprobar que puede conectarse a la máquina virtual de JumpBox en la red virtual del concentrador.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-233">Open a PowerShell console in the VM, and use the `Test-NetConnection` cmdlet to verify that you can connect to the jumpbox VM in the hub VNet.</span></span>

   ```powershell
   Test-NetConnection 10.0.0.68 -CommonTCPPort RDP
   ```

<span data-ttu-id="0f5ba-234">La salida debe tener una apariencia similar a la siguiente:</span><span class="sxs-lookup"><span data-stu-id="0f5ba-234">The output should look similar to the following:</span></span>

```powershell
ComputerName     : 10.0.0.68
RemoteAddress    : 10.0.0.68
RemotePort       : 3389
InterfaceAlias   : Ethernet 2
SourceAddress    : 192.168.1.000
TcpTestSucceeded : True
```

> [!NOTE]
> <span data-ttu-id="0f5ba-235">De forma predeterminada, las máquinas virtuales de Windows Server no permiten respuestas ICMP en Azure.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-235">By default, Windows Server VMs do not allow ICMP responses in Azure.</span></span> <span data-ttu-id="0f5ba-236">Si desea usar `ping` para probar la conectividad, debe habilitar el tráfico ICMP en el firewall de Windows con seguridad avanzada para cada máquina virtual.</span><span class="sxs-lookup"><span data-stu-id="0f5ba-236">If you want to use `ping` to test connectivity, you need to enable ICMP traffic in the Windows Advanced Firewall for each VM.</span></span>

<span data-ttu-id="0f5ba-237">Repita los mismos pasos para probar la conectividad con las redes virtuales de radio:</span><span class="sxs-lookup"><span data-stu-id="0f5ba-237">Repeat the sames steps to test connectivity to the spoke VNets:</span></span>

```powershell
Test-NetConnection 10.1.0.68 -CommonTCPPort RDP
Test-NetConnection 10.2.0.68 -CommonTCPPort RDP
```

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
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
