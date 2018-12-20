---
title: Entornos de desarrollo y pruebas para cargas de trabajo SAP en Azure
description: Cree un entorno de desarrollo y pruebas para las cargas de trabajo SAP.
author: AndrewDibbins
ms.date: 7/11/18
ms.custom: fasttrack
ms.openlocfilehash: 84665bfeb6ada568c631e1db72b97269d79f2e60
ms.sourcegitcommit: a0e8d11543751d681953717f6e78173e597ae207
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/06/2018
ms.locfileid: "53004680"
---
# <a name="devtest-environments-for-sap-workloads-on-azure"></a><span data-ttu-id="0bd6f-103">Entornos de desarrollo y pruebas para cargas de trabajo SAP en Azure</span><span class="sxs-lookup"><span data-stu-id="0bd6f-103">Dev/test environments for SAP workloads on Azure</span></span>

<span data-ttu-id="0bd6f-104">En este ejemplo se muestra cómo establecer un entorno de desarrollo y pruebas para SAP NetWeaver en un entorno Windows o Linux en Azure.</span><span class="sxs-lookup"><span data-stu-id="0bd6f-104">This example shows how to establish a dev/test environment for SAP NetWeaver in a Windows or Linux environment on Azure.</span></span> <span data-ttu-id="0bd6f-105">La base de datos empleada es AnyDB, el término de SAP para todos los sistemas de administración de bases de datos compatibles (que no sean SAP HANA).</span><span class="sxs-lookup"><span data-stu-id="0bd6f-105">The database used is AnyDB, the SAP term for any supported DBMS (that isn't SAP HANA).</span></span> <span data-ttu-id="0bd6f-106">Dado que esta arquitectura está diseñada para entornos que no son de producción, se implementa con una sola máquina virtual (VM) y se puede cambiar su tamaño para adaptarse a las necesidades de su organización.</span><span class="sxs-lookup"><span data-stu-id="0bd6f-106">Because this architecture is designed for non-production environments, it's deployed with just a single virtual machine (VM) and it's size can be changed to accommodate your organization's needs.</span></span>

<span data-ttu-id="0bd6f-107">Para casos de uso de producción, revise las arquitecturas de referencia de SAP que están disponibles a continuación:</span><span class="sxs-lookup"><span data-stu-id="0bd6f-107">For production use cases review the SAP reference architectures available below:</span></span>

* <span data-ttu-id="0bd6f-108">[SAP NetWeaver para AnyDB][sap-netweaver]</span><span class="sxs-lookup"><span data-stu-id="0bd6f-108">[SAP NetWeaver for AnyDB][sap-netweaver]</span></span>
* <span data-ttu-id="0bd6f-109">[SAP S/4Hana][sap-hana]</span><span class="sxs-lookup"><span data-stu-id="0bd6f-109">[SAP S/4HANA][sap-hana]</span></span>
* <span data-ttu-id="0bd6f-110">[SAP en Azure (instancias grandes)][sap-large]</span><span class="sxs-lookup"><span data-stu-id="0bd6f-110">[SAP on Azure large instances][sap-large]</span></span>

## <a name="relevant-use-cases"></a><span data-ttu-id="0bd6f-111">Casos de uso pertinentes</span><span class="sxs-lookup"><span data-stu-id="0bd6f-111">Relevant use cases</span></span>

<span data-ttu-id="0bd6f-112">Otros casos de uso pertinentes incluyen:</span><span class="sxs-lookup"><span data-stu-id="0bd6f-112">Other relevant use cases include:</span></span>

* <span data-ttu-id="0bd6f-113">Cargas de trabajo no productivas de instancias de SAP no importantes (espacio aislado, desarrollo, prueba, control de calidad)</span><span class="sxs-lookup"><span data-stu-id="0bd6f-113">Non-critical SAP non-productive workloads (sandbox, development, test, quality assurance)</span></span>
* <span data-ttu-id="0bd6f-114">Cargas de trabajo empresariales de SAP no críticas</span><span class="sxs-lookup"><span data-stu-id="0bd6f-114">Non-critical SAP business workloads</span></span>

## <a name="architecture"></a><span data-ttu-id="0bd6f-115">Arquitectura</span><span class="sxs-lookup"><span data-stu-id="0bd6f-115">Architecture</span></span>

![Diagrama de la arquitectura para entornos de desarrollo y pruebas para cargas de trabajo SAP](media/architecture-sap-dev-test.png)

<span data-ttu-id="0bd6f-117">Este escenario muestra el aprovisionamiento de una sola base de datos del sistema SAP y del servidor de aplicaciones de SAP en una sola máquina virtual.</span><span class="sxs-lookup"><span data-stu-id="0bd6f-117">This scenario demonstrates provisioning a single SAP system database and SAP application server on a single virtual machine.</span></span> <span data-ttu-id="0bd6f-118">Los datos fluyen por el escenario de la siguiente manera:</span><span class="sxs-lookup"><span data-stu-id="0bd6f-118">The data flows through the scenario as follows:</span></span>

1. <span data-ttu-id="0bd6f-119">Los clientes usan la interfaz de usuario de SAP u otras herramientas de cliente (Excel, un explorador web u otra aplicación web) para tener acceso al sistema SAP basado en Azure.</span><span class="sxs-lookup"><span data-stu-id="0bd6f-119">Customers use the SAP user interface or other client tools (Excel, a web browser, or other web application) to access the Azure-based SAP system.</span></span>
2. <span data-ttu-id="0bd6f-120">La conectividad se proporciona mediante una conexión de ExpressRoute establecida.</span><span class="sxs-lookup"><span data-stu-id="0bd6f-120">Connectivity is provided through the use of an established ExpressRoute.</span></span> <span data-ttu-id="0bd6f-121">La conexión de ExpressRoute finaliza en Azure, en la puerta de enlace de ExpressRoute.</span><span class="sxs-lookup"><span data-stu-id="0bd6f-121">The ExpressRoute connection is terminated in Azure at the ExpressRoute gateway.</span></span> <span data-ttu-id="0bd6f-122">El tráfico de red se enruta a través de la puerta de enlace de ExpressRoute hacia la subred de la puerta de enlace y desde esta a la subred de aplicaciones de nivel spoke (consulte el patrón [hub-and-spoke][hub-spoke]) y a través de una puerta de enlace de seguridad de red hacia la máquina virtual de la aplicación SAP.</span><span class="sxs-lookup"><span data-stu-id="0bd6f-122">Network traffic routes through the ExpressRoute gateway to the gateway subnet, and from the gateway subnet to the application-tier spoke subnet (see the [hub-spoke][hub-spoke] pattern) and via a Network Security Gateway to the SAP application virtual machine.</span></span>
3. <span data-ttu-id="0bd6f-123">Los servidores de administración de identidades proporcionan servicios de autenticación.</span><span class="sxs-lookup"><span data-stu-id="0bd6f-123">The identity management servers provide authentication services.</span></span>
4. <span data-ttu-id="0bd6f-124">JumpBox proporciona funcionalidades de administración local.</span><span class="sxs-lookup"><span data-stu-id="0bd6f-124">The jump box provides local management capabilities.</span></span>

### <a name="components"></a><span data-ttu-id="0bd6f-125">Componentes</span><span class="sxs-lookup"><span data-stu-id="0bd6f-125">Components</span></span>

* <span data-ttu-id="0bd6f-126">Las [redes virtuales](/azure/virtual-network/virtual-networks-overview) son la base de las comunicaciones de red dentro de Azure.</span><span class="sxs-lookup"><span data-stu-id="0bd6f-126">[Virtual Networks](/azure/virtual-network/virtual-networks-overview) are the basis of network communication within Azure.</span></span>
* <span data-ttu-id="0bd6f-127">[Máquina virtual](/azure/virtual-machines/windows/overview): Azure Virtual Machines proporciona una infraestructura bajo demanda, a gran escala, segura y virtualizada con servidores Windows o Linux.</span><span class="sxs-lookup"><span data-stu-id="0bd6f-127">[Virtual Machine](/azure/virtual-machines/windows/overview) Azure Virtual Machines provides on-demand, high-scale, secure, virtualized infrastructure using Windows or Linux Server.</span></span>
* <span data-ttu-id="0bd6f-128">[ExpressRoute](/azure/expressroute/expressroute-introduction) permite ampliar las redes locales en la nube de Microsoft a través de una conexión privada que facilita un proveedor de conectividad.</span><span class="sxs-lookup"><span data-stu-id="0bd6f-128">[ExpressRoute](/azure/expressroute/expressroute-introduction) lets you extend your on-premises networks into the Microsoft cloud over a private connection facilitated by a connectivity provider.</span></span>
* <span data-ttu-id="0bd6f-129">Los [grupos de seguridad de red](/azure/virtual-network/security-overview) le permiten limitar el tráfico a los recursos de una red virtual.</span><span class="sxs-lookup"><span data-stu-id="0bd6f-129">[Network Security Group](/azure/virtual-network/security-overview) lets you limit network traffic to resources in a virtual network.</span></span> <span data-ttu-id="0bd6f-130">Un grupo de seguridad de red contiene una lista de reglas de seguridad que permiten o deniegan el tráfico de red entrante o saliente en función de las direcciones IP de origen o destino, el puerto y el protocolo.</span><span class="sxs-lookup"><span data-stu-id="0bd6f-130">A network security group contains a list of security rules that allow or deny inbound or outbound network traffic based on source or destination IP address, port, and protocol.</span></span> 
* <span data-ttu-id="0bd6f-131">Los [grupos de recursos](/azure/azure-resource-manager/resource-group-overview#resource-groups) actúan como contenedores lógicos de recursos de Azure.</span><span class="sxs-lookup"><span data-stu-id="0bd6f-131">[Resource Groups](/azure/azure-resource-manager/resource-group-overview#resource-groups) act as logical containers for Azure resources.</span></span>

## <a name="considerations"></a><span data-ttu-id="0bd6f-132">Consideraciones</span><span class="sxs-lookup"><span data-stu-id="0bd6f-132">Considerations</span></span>

### <a name="availability"></a><span data-ttu-id="0bd6f-133">Disponibilidad</span><span class="sxs-lookup"><span data-stu-id="0bd6f-133">Availability</span></span>

 <span data-ttu-id="0bd6f-134">Microsoft ofrece un Acuerdo de Nivel de Servicio (SLA) para instancias únicas de máquina virtual.</span><span class="sxs-lookup"><span data-stu-id="0bd6f-134">Microsoft offers a service level agreement (SLA) for single VM instances.</span></span> <span data-ttu-id="0bd6f-135">Para más información, consulte el [Acuerdo de Nivel de Servicio de Microsoft Azure para máquinas virtuales](https://azure.microsoft.com/support/legal/sla/virtual-machines)</span><span class="sxs-lookup"><span data-stu-id="0bd6f-135">For more information on Microsoft Azure Service Level Agreement for Virtual Machines [SLA For Virtual Machines](https://azure.microsoft.com/support/legal/sla/virtual-machines)</span></span>

### <a name="scalability"></a><span data-ttu-id="0bd6f-136">Escalabilidad</span><span class="sxs-lookup"><span data-stu-id="0bd6f-136">Scalability</span></span>

<span data-ttu-id="0bd6f-137">Para obtener instrucciones generales sobre cómo diseñar soluciones escalables, consulte la [lista de comprobación de escalabilidad][scalability] en el centro de arquitectura de Azure.</span><span class="sxs-lookup"><span data-stu-id="0bd6f-137">For general guidance on designing scalable solutions, see the [scalability checklist][scalability] in the Azure Architecture Center.</span></span>

### <a name="security"></a><span data-ttu-id="0bd6f-138">Seguridad</span><span class="sxs-lookup"><span data-stu-id="0bd6f-138">Security</span></span>

<span data-ttu-id="0bd6f-139">Para obtener instrucciones generales sobre el diseño de soluciones seguras, consulte la [documentación de seguridad de Azure][security].</span><span class="sxs-lookup"><span data-stu-id="0bd6f-139">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="0bd6f-140">Resistencia</span><span class="sxs-lookup"><span data-stu-id="0bd6f-140">Resiliency</span></span>

<span data-ttu-id="0bd6f-141">Para obtener instrucciones generales sobre el diseño de soluciones resistentes, consulte [Diseño de aplicaciones resistentes de Azure][resiliency].</span><span class="sxs-lookup"><span data-stu-id="0bd6f-141">For general guidance on designing resilient solutions, see [Designing resilient applications for Azure][resiliency].</span></span>

## <a name="pricing"></a><span data-ttu-id="0bd6f-142">Precios</span><span class="sxs-lookup"><span data-stu-id="0bd6f-142">Pricing</span></span>

<span data-ttu-id="0bd6f-143">Para ayudarle a explorar el costo de ejecución de este escenario, todos los servicios están preconfigurados en los siguientes ejemplos de la calculadora de costos.</span><span class="sxs-lookup"><span data-stu-id="0bd6f-143">To help you explore the cost of running this scenario, all of the services are pre-configured in the cost calculator examples below.</span></span> <span data-ttu-id="0bd6f-144">Para ver cómo cambiarían los precios en su caso concreto, cambie las variables pertinentes para que coincidan con el tráfico esperado.</span><span class="sxs-lookup"><span data-stu-id="0bd6f-144">To see how the pricing would change for your particular use case, change the appropriate variables to match your expected traffic.</span></span>

<span data-ttu-id="0bd6f-145">Se proporcionan cuatro ejemplos de perfiles de costo según la cantidad de tráfico que se espera recibir:</span><span class="sxs-lookup"><span data-stu-id="0bd6f-145">We have provided four sample cost profiles based on amount of traffic you expect to receive:</span></span>

|<span data-ttu-id="0bd6f-146">Tamaño</span><span class="sxs-lookup"><span data-stu-id="0bd6f-146">Size</span></span>|<span data-ttu-id="0bd6f-147">SAP</span><span class="sxs-lookup"><span data-stu-id="0bd6f-147">SAPs</span></span>|<span data-ttu-id="0bd6f-148">Tipo de máquina virtual</span><span class="sxs-lookup"><span data-stu-id="0bd6f-148">VM Type</span></span>|<span data-ttu-id="0bd6f-149">Storage</span><span class="sxs-lookup"><span data-stu-id="0bd6f-149">Storage</span></span>|<span data-ttu-id="0bd6f-150">Calculadora de precios de Azure</span><span class="sxs-lookup"><span data-stu-id="0bd6f-150">Azure Pricing Calculator</span></span>|
|----|----|-------|-------|---------------|
|<span data-ttu-id="0bd6f-151">Pequeña</span><span class="sxs-lookup"><span data-stu-id="0bd6f-151">Small</span></span>|<span data-ttu-id="0bd6f-152">8000</span><span class="sxs-lookup"><span data-stu-id="0bd6f-152">8000</span></span>|<span data-ttu-id="0bd6f-153">D8s_v3</span><span class="sxs-lookup"><span data-stu-id="0bd6f-153">D8s_v3</span></span>|<span data-ttu-id="0bd6f-154">2xP20, 1xP10</span><span class="sxs-lookup"><span data-stu-id="0bd6f-154">2xP20, 1xP10</span></span>|[<span data-ttu-id="0bd6f-155">Pequeño</span><span class="sxs-lookup"><span data-stu-id="0bd6f-155">Small</span></span>](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1)|
|<span data-ttu-id="0bd6f-156">Mediano</span><span class="sxs-lookup"><span data-stu-id="0bd6f-156">Medium</span></span>|<span data-ttu-id="0bd6f-157">16000</span><span class="sxs-lookup"><span data-stu-id="0bd6f-157">16000</span></span>|<span data-ttu-id="0bd6f-158">D16s_v3</span><span class="sxs-lookup"><span data-stu-id="0bd6f-158">D16s_v3</span></span>|<span data-ttu-id="0bd6f-159">3xP20, 1xP10</span><span class="sxs-lookup"><span data-stu-id="0bd6f-159">3xP20, 1xP10</span></span>|[<span data-ttu-id="0bd6f-160">Mediano</span><span class="sxs-lookup"><span data-stu-id="0bd6f-160">Medium</span></span>](https://azure.com/e/465bd07047d148baab032b2f461550cd)|
<span data-ttu-id="0bd6f-161">grande</span><span class="sxs-lookup"><span data-stu-id="0bd6f-161">Large</span></span>|<span data-ttu-id="0bd6f-162">32000</span><span class="sxs-lookup"><span data-stu-id="0bd6f-162">32000</span></span>|<span data-ttu-id="0bd6f-163">E32s_v3</span><span class="sxs-lookup"><span data-stu-id="0bd6f-163">E32s_v3</span></span>|<span data-ttu-id="0bd6f-164">3xP20, 1xP10</span><span class="sxs-lookup"><span data-stu-id="0bd6f-164">3xP20, 1xP10</span></span>|[<span data-ttu-id="0bd6f-165">Grande</span><span class="sxs-lookup"><span data-stu-id="0bd6f-165">Large</span></span>](https://azure.com/e/ada2e849d68b41c3839cc976000c6931)|
<span data-ttu-id="0bd6f-166">Extragrande</span><span class="sxs-lookup"><span data-stu-id="0bd6f-166">Extra Large</span></span>|<span data-ttu-id="0bd6f-167">64000</span><span class="sxs-lookup"><span data-stu-id="0bd6f-167">64000</span></span>|<span data-ttu-id="0bd6f-168">M64s</span><span class="sxs-lookup"><span data-stu-id="0bd6f-168">M64s</span></span>|<span data-ttu-id="0bd6f-169">4xP20, 1xP10</span><span class="sxs-lookup"><span data-stu-id="0bd6f-169">4xP20, 1xP10</span></span>|[<span data-ttu-id="0bd6f-170">Extragrande</span><span class="sxs-lookup"><span data-stu-id="0bd6f-170">Extra Large</span></span>](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef)|

> [!NOTE]
> <span data-ttu-id="0bd6f-171">Estos precios son una guía que solo indica los costos de las máquinas virtuales y el almacenamiento.</span><span class="sxs-lookup"><span data-stu-id="0bd6f-171">This pricing is a guide that only indicates the VMs and storage costs.</span></span> <span data-ttu-id="0bd6f-172">Se excluyen las redes, el almacenamiento de copia de seguridad y los cargos de entrada y salida de datos.</span><span class="sxs-lookup"><span data-stu-id="0bd6f-172">It excludes networking, backup storage, and data ingress/egress charges.</span></span>

* <span data-ttu-id="0bd6f-173">[Pequeño](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1): Un sistema pequeño consta de una máquina virtual del tipo D8s_v3 con 8 vCPU, 32 GB de RAM y almacenamiento temporal de 200 GB, además de dos discos de almacenamiento de 512 GB y uno de almacenamiento Premium de 128 GB.</span><span class="sxs-lookup"><span data-stu-id="0bd6f-173">[Small](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1): A small system consists of VM type D8s_v3 with 8x vCPUs, 32 GB RAM and 200 GB temp storage, additionally two 512 GB and one 128 GB premium storage disks.</span></span>
* <span data-ttu-id="0bd6f-174">[Mediano](https://azure.com/e/465bd07047d148baab032b2f461550cd): Un sistema mediano consta de una máquina virtual del tipo D16s_v3 con 16 vCPU, 64 GB de RAM y almacenamiento temporal de 400 GB, además de tres discos de almacenamiento de 512 GB y uno de almacenamiento Premium de 128 GB.</span><span class="sxs-lookup"><span data-stu-id="0bd6f-174">[Medium](https://azure.com/e/465bd07047d148baab032b2f461550cd): A medium system consists of VM type D16s_v3 with 16x vCPUs, 64 GB RAM and 400 GB temp storage, additionally three 512 GB and one 128 GB premium storage disks.</span></span>
* <span data-ttu-id="0bd6f-175">[Grande](https://azure.com/e/ada2e849d68b41c3839cc976000c6931): Un sistema grande consta de una máquina virtual del tipo E32s_v3 con 32 vCPU, 256 GB de RAM y almacenamiento temporal de 512 GB, además de tres discos de almacenamiento Premium de 512 GB y uno de 128 GB.</span><span class="sxs-lookup"><span data-stu-id="0bd6f-175">[Large](https://azure.com/e/ada2e849d68b41c3839cc976000c6931): A large system consists of VM type E32s_v3 with 32x vCPUs, 256 GB RAM and 512 GB temp storage, additionally three 512GB and one 128GB premium storage disks.</span></span>
* <span data-ttu-id="0bd6f-176">[Extra-grande](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef): Un sistema extra-grande consta de una máquina virtual del tipo M64s con 64 vCPU, 1024 GB de RAM y almacenamiento temporal de 2000 GB, además de cuatro discos de almacenamiento premium de 512 GB y uno de 128 GB.</span><span class="sxs-lookup"><span data-stu-id="0bd6f-176">[Extra Large](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef): An extra large system consists of a VM type M64s with 64x vCPUs, 1024 GB RAM and 2000 GB temp storage, additionally four 512 GB and one 128 GB premium storage disks.</span></span>

## <a name="deployment"></a><span data-ttu-id="0bd6f-177">Implementación</span><span class="sxs-lookup"><span data-stu-id="0bd6f-177">Deployment</span></span>

<span data-ttu-id="0bd6f-178">Haga clic aquí para implementar la infraestructura subyacente para este escenario.</span><span class="sxs-lookup"><span data-stu-id="0bd6f-178">Click here to deploy the underlying infrastructure for this scenario.</span></span>

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fsap-2tier%2Fazuredeploy.json" target="_blank">
    <img src="https://azuredeploy.net/deploybutton.png"/>
</a>

> [!NOTE]
> <span data-ttu-id="0bd6f-179">SAP y Oracle no se instalan durante esta implementación.</span><span class="sxs-lookup"><span data-stu-id="0bd6f-179">SAP and Oracle are not installed during this deployment.</span></span> <span data-ttu-id="0bd6f-180">Debe implementar estos componentes por separado.</span><span class="sxs-lookup"><span data-stu-id="0bd6f-180">You will need to deploy these components separately.</span></span>

<!-- links -->
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[sap-netweaver]: /azure/architecture/reference-architectures/sap/sap-netweaver
[sap-hana]: /azure/architecture/reference-architectures/sap/sap-s4hana
[sap-large]: /azure/architecture/reference-architectures/sap/hana-large-instances
[hub-spoke]: /azure/architecture/reference-architectures/hybrid-networking/hub-spoke
