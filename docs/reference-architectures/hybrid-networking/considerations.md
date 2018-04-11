---
title: Elección de una solución para conectar una red local a Azure
description: Compara las arquitecturas de referencia para conectar una red local a Azure.
author: telmosampaio
ms.date: 04/06/2017
ms.openlocfilehash: 274b9df1817632a7f3eaafa8bf02e965fdc3feea
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2017
---
# <a name="choose-a-solution-for-connecting-an-on-premises-network-to-azure"></a><span data-ttu-id="509fc-103">Elección de una solución para conectar una red local a Azure</span><span class="sxs-lookup"><span data-stu-id="509fc-103">Choose a solution for connecting an on-premises network to Azure</span></span>

<span data-ttu-id="509fc-104">En este artículo se comparan las opciones para conectar una red local a una instancia de Azure Virtual Network (VNet).</span><span class="sxs-lookup"><span data-stu-id="509fc-104">This article compares options for connecting an on-premises network to an Azure Virtual Network (VNet).</span></span> <span data-ttu-id="509fc-105">Se proporciona una arquitectura de referencia y una solución implementable para cada opción.</span><span class="sxs-lookup"><span data-stu-id="509fc-105">We provide a reference architecture and a deployable solution for each option.</span></span>

## <a name="vpn-connection"></a><span data-ttu-id="509fc-106">Conexión VPN</span><span class="sxs-lookup"><span data-stu-id="509fc-106">VPN connection</span></span>

<span data-ttu-id="509fc-107">Use una red privada virtual (VPN) para conectar su red local con una Azure VNet a través de un túnel de VPN con IPSec.</span><span class="sxs-lookup"><span data-stu-id="509fc-107">Use a virtual private network (VPN) to connect your on-premises network with an Azure VNet through an IPSec VPN tunnel.</span></span>

<span data-ttu-id="509fc-108">Esta arquitectura es adecuada para las aplicaciones híbridas en las que es probable que el tráfico entre el hardware local y la nube sea ligero, o en la que esté dispuesto a compensar una latencia ligeramente mayor con la flexibilidad y el poder de procesamiento de la nube.</span><span class="sxs-lookup"><span data-stu-id="509fc-108">This architecture is suitable for hybrid applications where the traffic between on-premises hardware and the cloud is likely to be light, or you are willing to trade slightly extended latency for the flexibility and processing power of the cloud.</span></span>

<span data-ttu-id="509fc-109">**Ventajas**</span><span class="sxs-lookup"><span data-stu-id="509fc-109">**Benefits**</span></span>

- <span data-ttu-id="509fc-110">Fácil de configurar.</span><span class="sxs-lookup"><span data-stu-id="509fc-110">Simple to configure.</span></span>

<span data-ttu-id="509fc-111">**Desafíos**</span><span class="sxs-lookup"><span data-stu-id="509fc-111">**Challenges**</span></span>

- <span data-ttu-id="509fc-112">Requiere un dispositivo VPN local.</span><span class="sxs-lookup"><span data-stu-id="509fc-112">Requires an on-premises VPN device.</span></span>
- <span data-ttu-id="509fc-113">Aunque Microsoft garantiza la disponibilidad del 99,9 % para cada VPN Gateway, este Acuerdo de Nivel de Servicio solo cubre VPN Gateway y no la conexión de red para la puerta de enlace.</span><span class="sxs-lookup"><span data-stu-id="509fc-113">Although Microsoft guarantees 99.9% availability for each VPN Gateway, this SLA only covers the VPN gateway, and not your network connection to the gateway.</span></span>
- <span data-ttu-id="509fc-114">Una conexión VPN a través de Azure VPN Gateway admite actualmente un ancho de banda máximo de 200 Mbps.</span><span class="sxs-lookup"><span data-stu-id="509fc-114">A VPN connection over Azure VPN Gateway currently supports a maximum of 200 Mbps bandwidth.</span></span> <span data-ttu-id="509fc-115">Quizá tenga que dividir su red Azure Virtual Network entre varias conexiones VPN si espera que se supere este rendimiento.</span><span class="sxs-lookup"><span data-stu-id="509fc-115">You may need to partition your Azure virtual network across multiple VPN connections if you expect to exceed this throughput.</span></span>

<span data-ttu-id="509fc-116">**[Más información...][vpn]**</span><span class="sxs-lookup"><span data-stu-id="509fc-116">**[Read more...][vpn]**</span></span>

## <a name="azure-expressroute-connection"></a><span data-ttu-id="509fc-117">Conexión de Azure ExpressRoute</span><span class="sxs-lookup"><span data-stu-id="509fc-117">Azure ExpressRoute connection</span></span>

<span data-ttu-id="509fc-118">Las conexiones de ExpressRoute utilizan una conexión privada y dedicada a través de un proveedor de conectividad de terceros.</span><span class="sxs-lookup"><span data-stu-id="509fc-118">ExpressRoute connections use a private, dedicated connection through a third-party connectivity provider.</span></span> <span data-ttu-id="509fc-119">La conexión privada extiende la red local en Azure.</span><span class="sxs-lookup"><span data-stu-id="509fc-119">The private connection extends your on-premises network into Azure.</span></span> 

<span data-ttu-id="509fc-120">Esta arquitectura es adecuada para aplicaciones híbridas que ejecutan cargas de trabajo críticas a gran escala que requieren un alto grado de escalabilidad.</span><span class="sxs-lookup"><span data-stu-id="509fc-120">This architecture is suitable for hybrid applications running large-scale, mission-critical workloads that require a high degree of scalability.</span></span> 

<span data-ttu-id="509fc-121">**Ventajas**</span><span class="sxs-lookup"><span data-stu-id="509fc-121">**Benefits**</span></span>

- <span data-ttu-id="509fc-122">Ancho de banda mucho más alto disponible; hasta 10 Gbps en función del proveedor de conectividad.</span><span class="sxs-lookup"><span data-stu-id="509fc-122">Much higher bandwidth available; up to 10 Gbps depending on the connectivity provider.</span></span>
- <span data-ttu-id="509fc-123">Admite el escalado dinámico de ancho de banda para ayudar a reducir los costos durante los períodos de menor actividad.</span><span class="sxs-lookup"><span data-stu-id="509fc-123">Supports dynamic scaling of bandwidth to help reduce costs during periods of lower demand.</span></span> <span data-ttu-id="509fc-124">Sin embargo, no todos los proveedores de conectividad tienen esta opción.</span><span class="sxs-lookup"><span data-stu-id="509fc-124">However, not all connectivity providers have this option.</span></span>
- <span data-ttu-id="509fc-125">Puede permitir que su organización tenga acceso directo a las nubes nacionales, en función del proveedor de conectividad.</span><span class="sxs-lookup"><span data-stu-id="509fc-125">May allow your organization direct access to national clouds, depending on the connectivity provider.</span></span>
- <span data-ttu-id="509fc-126">Acuerdo de Nivel de Servicio del 99,9 % de disponibilidad en toda la conexión.</span><span class="sxs-lookup"><span data-stu-id="509fc-126">99.9% availability SLA across the entire connection.</span></span>

<span data-ttu-id="509fc-127">**Desafíos**</span><span class="sxs-lookup"><span data-stu-id="509fc-127">**Challenges**</span></span>

- <span data-ttu-id="509fc-128">Puede ser difícil de configurar.</span><span class="sxs-lookup"><span data-stu-id="509fc-128">Can be complex to set up.</span></span> <span data-ttu-id="509fc-129">La creación de una conexión ExpressRoute requiere trabajar con un proveedor de conectividad de terceros.</span><span class="sxs-lookup"><span data-stu-id="509fc-129">Creating an ExpressRoute connection requires working with a third-party connectivity provider.</span></span> <span data-ttu-id="509fc-130">El proveedor es responsable del aprovisionamiento de la conexión de red.</span><span class="sxs-lookup"><span data-stu-id="509fc-130">The provider is responsible for provisioning the network connection.</span></span>
- <span data-ttu-id="509fc-131">Requiere que enrutadores de un alto ancho de banda a nivel local.</span><span class="sxs-lookup"><span data-stu-id="509fc-131">Requires high-bandwidth routers on-premises.</span></span>

<span data-ttu-id="509fc-132">**[Más información...][expressroute]**</span><span class="sxs-lookup"><span data-stu-id="509fc-132">**[Read more...][expressroute]**</span></span>

## <a name="expressroute-with-vpn-failover"></a><span data-ttu-id="509fc-133">ExpressRoute con conmutación por error de VPN</span><span class="sxs-lookup"><span data-stu-id="509fc-133">ExpressRoute with VPN failover</span></span>

<span data-ttu-id="509fc-134">Esta opción combina los dos anteriores: uso de ExpressRoute en condiciones normales, pero conmutación por error a una conexión VPN si se produce una pérdida de conectividad en el circuito de ExpressRoute.</span><span class="sxs-lookup"><span data-stu-id="509fc-134">This options combines the previous two, using ExpressRoute in normal conditions, but failing over to a VPN connection if there is a loss of connectivity in the ExpressRoute circuit.</span></span>

<span data-ttu-id="509fc-135">Esta arquitectura es adecuada para aplicaciones híbridas que necesitan un mayor ancho de banda que ExpressRoute y también requieren conectividad de red de alta disponibilidad.</span><span class="sxs-lookup"><span data-stu-id="509fc-135">This architecture is suitable for hybrid applications that need the higher bandwidth of ExpressRoute, and also require highly available network connectivity.</span></span> 

<span data-ttu-id="509fc-136">**Ventajas**</span><span class="sxs-lookup"><span data-stu-id="509fc-136">**Benefits**</span></span>

- <span data-ttu-id="509fc-137">Alta disponibilidad si se produce un error en el circuito de ExpressRoute, aunque la conexión de reserva está en una red de ancho de banda inferior.</span><span class="sxs-lookup"><span data-stu-id="509fc-137">High availability if the ExpressRoute circuit fails, although the fallback connection is on a lower bandwidth network.</span></span>

<span data-ttu-id="509fc-138">**Desafíos**</span><span class="sxs-lookup"><span data-stu-id="509fc-138">**Challenges**</span></span>

- <span data-ttu-id="509fc-139">Configuración compleja.</span><span class="sxs-lookup"><span data-stu-id="509fc-139">Complex to configure.</span></span> <span data-ttu-id="509fc-140">Debe configurar tanto una conexión VPN como un circuito de ExpressRoute.</span><span class="sxs-lookup"><span data-stu-id="509fc-140">You need to set up both a VPN connection and an ExpressRoute circuit.</span></span>
- <span data-ttu-id="509fc-141">Requiere hardware redundante (dispositivos VPN) y una conexión redundante de Azure VPN Gateway que tiene un coste.</span><span class="sxs-lookup"><span data-stu-id="509fc-141">Requires redundant hardware (VPN appliances), and a redundant Azure VPN Gateway connection for which you pay charges.</span></span>

<span data-ttu-id="509fc-142">**[Más información...][expressroute-vpn-failover]**</span><span class="sxs-lookup"><span data-stu-id="509fc-142">**[Read more...][expressroute-vpn-failover]**</span></span>

<!-- links -->
[expressroute]: ./expressroute.md
[expressroute-vpn-failover]: ./expressroute-vpn-failover.md
[vpn]: ./vpn.md