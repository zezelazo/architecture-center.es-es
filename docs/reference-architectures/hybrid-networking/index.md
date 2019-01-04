---
title: Conexión de una red local a Azure
titleSuffix: Azure Reference Architectures
description: Compare las arquitecturas de referencia para conectar una red local a Azure.
author: telmosampaio
ms.date: 07/02/2018
ms.openlocfilehash: f13249f225ad7ab5072de2b2175cdc2ffb6d0074
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/04/2019
ms.locfileid: "54011062"
---
# <a name="choose-a-solution-for-connecting-an-on-premises-network-to-azure"></a><span data-ttu-id="4fbe3-103">Elección de una solución para conectar una red local a Azure</span><span class="sxs-lookup"><span data-stu-id="4fbe3-103">Choose a solution for connecting an on-premises network to Azure</span></span>

<span data-ttu-id="4fbe3-104">En este artículo se comparan las opciones para conectar una red local a una instancia de Azure Virtual Network (VNet).</span><span class="sxs-lookup"><span data-stu-id="4fbe3-104">This article compares options for connecting an on-premises network to an Azure Virtual Network (VNet).</span></span> <span data-ttu-id="4fbe3-105">Para cada opción, hay disponible una arquitectura de referencia más detallada.</span><span class="sxs-lookup"><span data-stu-id="4fbe3-105">For each option, a more detailed reference architecture is available.</span></span>

## <a name="vpn-connection"></a><span data-ttu-id="4fbe3-106">Conexión VPN</span><span class="sxs-lookup"><span data-stu-id="4fbe3-106">VPN connection</span></span>

<span data-ttu-id="4fbe3-107">Una [puerta de enlace de VPN](/azure/vpn-gateway/vpn-gateway-about-vpngateways) es un tipo de puerta de enlace de red virtual que envía tráfico cifrado entre una instancia de Azure Virtual Network y una ubicación local.</span><span class="sxs-lookup"><span data-stu-id="4fbe3-107">A [VPN gateway](/azure/vpn-gateway/vpn-gateway-about-vpngateways) is a type of virtual network gateway that sends encrypted traffic between an Azure virtual network and an on-premises location.</span></span> <span data-ttu-id="4fbe3-108">El tráfico cifrado pasa a través de Internet público.</span><span class="sxs-lookup"><span data-stu-id="4fbe3-108">The encrypted traffic goes over the public Internet.</span></span>

<span data-ttu-id="4fbe3-109">Esta arquitectura es adecuada para las aplicaciones híbridas en las que es probable que el tráfico entre el hardware local y la nube sea ligero, o en la que esté dispuesto a compensar una latencia ligeramente mayor con la flexibilidad y el poder de procesamiento de la nube.</span><span class="sxs-lookup"><span data-stu-id="4fbe3-109">This architecture is suitable for hybrid applications where the traffic between on-premises hardware and the cloud is likely to be light, or you are willing to trade slightly extended latency for the flexibility and processing power of the cloud.</span></span>

### <a name="benefits"></a><span data-ttu-id="4fbe3-110">Ventajas</span><span class="sxs-lookup"><span data-stu-id="4fbe3-110">Benefits</span></span>

- <span data-ttu-id="4fbe3-111">Fácil de configurar.</span><span class="sxs-lookup"><span data-stu-id="4fbe3-111">Simple to configure.</span></span>

### <a name="challenges"></a><span data-ttu-id="4fbe3-112">Desafíos</span><span class="sxs-lookup"><span data-stu-id="4fbe3-112">Challenges</span></span>

- <span data-ttu-id="4fbe3-113">Requiere un dispositivo VPN local.</span><span class="sxs-lookup"><span data-stu-id="4fbe3-113">Requires an on-premises VPN device.</span></span>
- <span data-ttu-id="4fbe3-114">Aunque Microsoft garantiza la disponibilidad del 99,9 % para cada VPN Gateway, este [Acuerdo de Nivel de Servicio](https://azure.microsoft.com/support/legal/sla/vpn-gateway/) solo cubre VPN Gateway y no la conexión de red a la puerta de enlace.</span><span class="sxs-lookup"><span data-stu-id="4fbe3-114">Although Microsoft guarantees 99.9% availability for each VPN Gateway, this [SLA](https://azure.microsoft.com/support/legal/sla/vpn-gateway/) only covers the VPN gateway, and not your network connection to the gateway.</span></span>
- <span data-ttu-id="4fbe3-115">Una conexión VPN a través de Azure VPN Gateway admite actualmente un ancho de banda máximo de 1,25 Gbps.</span><span class="sxs-lookup"><span data-stu-id="4fbe3-115">A VPN connection over Azure VPN Gateway currently supports a maximum of 1.25 Gbps bandwidth.</span></span> <span data-ttu-id="4fbe3-116">Quizá tenga que dividir su red Azure Virtual Network entre varias conexiones VPN si espera que se supere este rendimiento.</span><span class="sxs-lookup"><span data-stu-id="4fbe3-116">You may need to partition your Azure virtual network across multiple VPN connections if you expect to exceed this throughput.</span></span>

### <a name="reference-architecture"></a><span data-ttu-id="4fbe3-117">Arquitectura de referencia</span><span class="sxs-lookup"><span data-stu-id="4fbe3-117">Reference architecture</span></span>

- [<span data-ttu-id="4fbe3-118">Red híbrida con VPN Gateway</span><span class="sxs-lookup"><span data-stu-id="4fbe3-118">Hybrid network with VPN gateway</span></span>](./vpn.md)

<!-- markdownlint-disable MD024 -->

## <a name="azure-expressroute-connection"></a><span data-ttu-id="4fbe3-119">Conexión de Azure ExpressRoute</span><span class="sxs-lookup"><span data-stu-id="4fbe3-119">Azure ExpressRoute connection</span></span>

<span data-ttu-id="4fbe3-120">Las conexiones de [ExpressRoute](/azure/expressroute/) utilizan una conexión privada y dedicada a través de un proveedor de conectividad de terceros.</span><span class="sxs-lookup"><span data-stu-id="4fbe3-120">[ExpressRoute](/azure/expressroute/) connections use a private, dedicated connection through a third-party connectivity provider.</span></span> <span data-ttu-id="4fbe3-121">La conexión privada extiende la red local en Azure.</span><span class="sxs-lookup"><span data-stu-id="4fbe3-121">The private connection extends your on-premises network into Azure.</span></span>

<span data-ttu-id="4fbe3-122">Esta arquitectura es adecuada para aplicaciones híbridas que ejecutan cargas de trabajo críticas a gran escala que requieren un alto grado de escalabilidad.</span><span class="sxs-lookup"><span data-stu-id="4fbe3-122">This architecture is suitable for hybrid applications running large-scale, mission-critical workloads that require a high degree of scalability.</span></span>

### <a name="benefits"></a><span data-ttu-id="4fbe3-123">Ventajas</span><span class="sxs-lookup"><span data-stu-id="4fbe3-123">Benefits</span></span>

- <span data-ttu-id="4fbe3-124">Ancho de banda mucho más alto disponible; hasta 10 Gbps en función del proveedor de conectividad.</span><span class="sxs-lookup"><span data-stu-id="4fbe3-124">Much higher bandwidth available; up to 10 Gbps depending on the connectivity provider.</span></span>
- <span data-ttu-id="4fbe3-125">Admite el escalado dinámico de ancho de banda para ayudar a reducir los costos durante los períodos de menor actividad.</span><span class="sxs-lookup"><span data-stu-id="4fbe3-125">Supports dynamic scaling of bandwidth to help reduce costs during periods of lower demand.</span></span> <span data-ttu-id="4fbe3-126">Sin embargo, no todos los proveedores de conectividad tienen esta opción.</span><span class="sxs-lookup"><span data-stu-id="4fbe3-126">However, not all connectivity providers have this option.</span></span>
- <span data-ttu-id="4fbe3-127">Puede permitir que su organización tenga acceso directo a las nubes nacionales, en función del proveedor de conectividad.</span><span class="sxs-lookup"><span data-stu-id="4fbe3-127">May allow your organization direct access to national clouds, depending on the connectivity provider.</span></span>
- <span data-ttu-id="4fbe3-128">Acuerdo de Nivel de Servicio del 99,9 % de disponibilidad en toda la conexión.</span><span class="sxs-lookup"><span data-stu-id="4fbe3-128">99.9% availability SLA across the entire connection.</span></span>

### <a name="challenges"></a><span data-ttu-id="4fbe3-129">Desafíos</span><span class="sxs-lookup"><span data-stu-id="4fbe3-129">Challenges</span></span>

- <span data-ttu-id="4fbe3-130">Puede ser difícil de configurar.</span><span class="sxs-lookup"><span data-stu-id="4fbe3-130">Can be complex to set up.</span></span> <span data-ttu-id="4fbe3-131">La creación de una conexión ExpressRoute requiere trabajar con un proveedor de conectividad de terceros.</span><span class="sxs-lookup"><span data-stu-id="4fbe3-131">Creating an ExpressRoute connection requires working with a third-party connectivity provider.</span></span> <span data-ttu-id="4fbe3-132">El proveedor es responsable del aprovisionamiento de la conexión de red.</span><span class="sxs-lookup"><span data-stu-id="4fbe3-132">The provider is responsible for provisioning the network connection.</span></span>
- <span data-ttu-id="4fbe3-133">Requiere que enrutadores de un alto ancho de banda a nivel local.</span><span class="sxs-lookup"><span data-stu-id="4fbe3-133">Requires high-bandwidth routers on-premises.</span></span>

### <a name="reference-architecture"></a><span data-ttu-id="4fbe3-134">Arquitectura de referencia</span><span class="sxs-lookup"><span data-stu-id="4fbe3-134">Reference architecture</span></span>

- [<span data-ttu-id="4fbe3-135">Red híbrida con ExpressRoute</span><span class="sxs-lookup"><span data-stu-id="4fbe3-135">Hybrid network with ExpressRoute</span></span>](./expressroute.md)

## <a name="expressroute-with-vpn-failover"></a><span data-ttu-id="4fbe3-136">ExpressRoute con conmutación por error de VPN</span><span class="sxs-lookup"><span data-stu-id="4fbe3-136">ExpressRoute with VPN failover</span></span>

<span data-ttu-id="4fbe3-137">Esta opción combina los dos anteriores: uso de ExpressRoute en condiciones normales, pero conmutación por error a una conexión VPN si se produce una pérdida de conectividad en el circuito de ExpressRoute.</span><span class="sxs-lookup"><span data-stu-id="4fbe3-137">This options combines the previous two, using ExpressRoute in normal conditions, but failing over to a VPN connection if there is a loss of connectivity in the ExpressRoute circuit.</span></span>

<span data-ttu-id="4fbe3-138">Esta arquitectura es adecuada para aplicaciones híbridas que necesitan un mayor ancho de banda que ExpressRoute y también requieren conectividad de red de alta disponibilidad.</span><span class="sxs-lookup"><span data-stu-id="4fbe3-138">This architecture is suitable for hybrid applications that need the higher bandwidth of ExpressRoute, and also require highly available network connectivity.</span></span>

### <a name="benefits"></a><span data-ttu-id="4fbe3-139">Ventajas</span><span class="sxs-lookup"><span data-stu-id="4fbe3-139">Benefits</span></span>

- <span data-ttu-id="4fbe3-140">Alta disponibilidad si se produce un error en el circuito de ExpressRoute, aunque la conexión de reserva está en una red de ancho de banda inferior.</span><span class="sxs-lookup"><span data-stu-id="4fbe3-140">High availability if the ExpressRoute circuit fails, although the fallback connection is on a lower bandwidth network.</span></span>

### <a name="challenges"></a><span data-ttu-id="4fbe3-141">Desafíos</span><span class="sxs-lookup"><span data-stu-id="4fbe3-141">Challenges</span></span>

- <span data-ttu-id="4fbe3-142">Configuración compleja.</span><span class="sxs-lookup"><span data-stu-id="4fbe3-142">Complex to configure.</span></span> <span data-ttu-id="4fbe3-143">Debe configurar tanto una conexión VPN como un circuito de ExpressRoute.</span><span class="sxs-lookup"><span data-stu-id="4fbe3-143">You need to set up both a VPN connection and an ExpressRoute circuit.</span></span>
- <span data-ttu-id="4fbe3-144">Requiere hardware redundante (dispositivos VPN) y una conexión redundante de Azure VPN Gateway que tiene un coste.</span><span class="sxs-lookup"><span data-stu-id="4fbe3-144">Requires redundant hardware (VPN appliances), and a redundant Azure VPN Gateway connection for which you pay charges.</span></span>

### <a name="reference-architecture"></a><span data-ttu-id="4fbe3-145">Arquitectura de referencia</span><span class="sxs-lookup"><span data-stu-id="4fbe3-145">Reference architecture</span></span>

- [<span data-ttu-id="4fbe3-146">Red híbrida con ExpressRoute y conmutación por error de VPN</span><span class="sxs-lookup"><span data-stu-id="4fbe3-146">Hybrid network with ExpressRoute and VPN failover</span></span>](./expressroute-vpn-failover.md)

<!-- markdownlint-disable MD024 -->

## <a name="hub-spoke-network-topology"></a><span data-ttu-id="4fbe3-147">Topología de red en estrella tipo hub-and-spoke</span><span class="sxs-lookup"><span data-stu-id="4fbe3-147">Hub-spoke network topology</span></span>

<span data-ttu-id="4fbe3-148">Una topología de red en estrella de tipo hub-and-spoke constituye una forma de aislar cargas de trabajo mientras se comparten servicios como los de identidad y seguridad.</span><span class="sxs-lookup"><span data-stu-id="4fbe3-148">A hub-spoke network topology is a way to isolate workloads while sharing services such as identity and security.</span></span> <span data-ttu-id="4fbe3-149">El centro (hub) es una red virtual (VNet) en Azure que actúa como un punto central de conectividad para la red local.</span><span class="sxs-lookup"><span data-stu-id="4fbe3-149">The hub is a virtual network (VNet) in Azure that acts as a central point of connectivity to your on-premises network.</span></span> <span data-ttu-id="4fbe3-150">Los radios son redes virtuales que se emparejan con el concentrador.</span><span class="sxs-lookup"><span data-stu-id="4fbe3-150">The spokes are VNets that peer with the hub.</span></span> <span data-ttu-id="4fbe3-151">Los servicios compartidos se implementan en el centro, mientras que las cargas de trabajo individuales se implementan como radios.</span><span class="sxs-lookup"><span data-stu-id="4fbe3-151">Shared services are deployed in the hub, while individual workloads are deployed as spokes.</span></span>

### <a name="reference-architectures"></a><span data-ttu-id="4fbe3-152">Arquitecturas de referencia</span><span class="sxs-lookup"><span data-stu-id="4fbe3-152">Reference architectures</span></span>

- [<span data-ttu-id="4fbe3-153">Topología en estrella tipo hub-and-spoke</span><span class="sxs-lookup"><span data-stu-id="4fbe3-153">Hub-spoke topology</span></span>](./hub-spoke.md)
- [<span data-ttu-id="4fbe3-154">Topología en estrella tipo hub-and-spoke con servicios compartidos</span><span class="sxs-lookup"><span data-stu-id="4fbe3-154">Hub-spoke with shared services</span></span>](./shared-services.md)
