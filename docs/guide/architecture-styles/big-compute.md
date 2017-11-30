---
title: Estilo de arquitectura Big Compute
description: Describe las ventajas, las dificultades y los procedimientos recomendados para las arquitecturas Big Compute en Azure.
author: MikeWasson
ms.openlocfilehash: b16be4133143d7d73062eeb280b44779c390f387
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2017
---
# <a name="big-compute-architecture-style"></a><span data-ttu-id="977a4-103">Estilo de arquitectura Big Compute</span><span class="sxs-lookup"><span data-stu-id="977a4-103">Big compute architecture style</span></span>

<span data-ttu-id="977a4-104">El término *Big Compute* describe cargas de trabajo a gran escala que requieren un gran número de núcleos, a menudo cientos o miles.</span><span class="sxs-lookup"><span data-stu-id="977a4-104">The term *big compute* describes large-scale workloads that require a large number of cores, often numbering in the hundreds or thousands.</span></span> <span data-ttu-id="977a4-105">Los escenarios incluyen representación de imágenes, dinámica de fluidos, modelado de riesgos financieros, exploración petrolífera, diseño de fármacos y análisis de esfuerzos en ingeniería, entre otros.</span><span class="sxs-lookup"><span data-stu-id="977a4-105">Scenarios include image rendering, fluid dynamics, financial risk modeling, oil exploration, drug design, and engineering stress analysis, among others.</span></span>

![](./images/big-compute-logical.png)

<span data-ttu-id="977a4-106">Estas son algunas características típicas de las aplicaciones Big Compute:</span><span class="sxs-lookup"><span data-stu-id="977a4-106">Here are some typical characteristics of big compute applications:</span></span>

- <span data-ttu-id="977a4-107">El trabajo se puede dividir en tareas discretas, que se pueden ejecutar simultáneamente en numerosos núcleos.</span><span class="sxs-lookup"><span data-stu-id="977a4-107">The work can be split into discrete tasks, which can be run across many cores simultaneously.</span></span>
- <span data-ttu-id="977a4-108">Cada tarea está limitada.</span><span class="sxs-lookup"><span data-stu-id="977a4-108">Each task is finite.</span></span> <span data-ttu-id="977a4-109">Toma una entrada, realiza un procesamiento y genera una salida.</span><span class="sxs-lookup"><span data-stu-id="977a4-109">It takes some input, does some processing, and produces output.</span></span> <span data-ttu-id="977a4-110">Toda la aplicación se ejecuta durante una cantidad limitada de tiempo (de minutos a días).</span><span class="sxs-lookup"><span data-stu-id="977a4-110">The entire application runs for a finite amount of time (minutes to days).</span></span> <span data-ttu-id="977a4-111">Un patrón común consiste en aprovisionar un gran número de núcleos en una ráfaga y reducir luego hasta cero una vez que se complete la aplicación.</span><span class="sxs-lookup"><span data-stu-id="977a4-111">A common pattern is to provision a large number of cores in a burst, and then spin down to zero once the application completes.</span></span> 
- <span data-ttu-id="977a4-112">La aplicación no debe permanecer activa de manera ininterrumpida.</span><span class="sxs-lookup"><span data-stu-id="977a4-112">The application does not need to stay up 24/7.</span></span> <span data-ttu-id="977a4-113">Sin embargo, el sistema debe controlar los errores de nodo o los bloqueos de aplicación.</span><span class="sxs-lookup"><span data-stu-id="977a4-113">However, the system must handle node failures or application crashes.</span></span>
- <span data-ttu-id="977a4-114">Para algunas aplicaciones, las tareas son independientes y se pueden ejecutar en paralelo.</span><span class="sxs-lookup"><span data-stu-id="977a4-114">For some applications, tasks are independent and can run in parallel.</span></span> <span data-ttu-id="977a4-115">En otros casos, las tareas están estrechamente acopladas, lo que significa que deben interactuar o intercambiar resultados intermedios.</span><span class="sxs-lookup"><span data-stu-id="977a4-115">In other cases, tasks are tightly coupled, meaning they must interact or exchange intermediate results.</span></span> <span data-ttu-id="977a4-116">En ese caso, considere la posibilidad de utilizar tecnologías de redes de alta velocidad como InfiniBand y acceso directo a memoria remota (RDMA).</span><span class="sxs-lookup"><span data-stu-id="977a4-116">In that case, consider using high-speed networking technologies such as InfiniBand and remote direct memory access (RDMA).</span></span> 
- <span data-ttu-id="977a4-117">En función de la carga de trabajo, puede usar tamaños de máquinas virtuales de proceso intensivo (H16r, H16mr y A9).</span><span class="sxs-lookup"><span data-stu-id="977a4-117">Depending on your workload, you might use compute-intensive VM sizes (H16r, H16mr, and A9).</span></span>

## <a name="when-to-use-this-architecture"></a><span data-ttu-id="977a4-118">Cuándo utilizar esta arquitectura</span><span class="sxs-lookup"><span data-stu-id="977a4-118">When to use this architecture</span></span>

- <span data-ttu-id="977a4-119">En operaciones de cálculo intensivas como simulación y procesamiento de números.</span><span class="sxs-lookup"><span data-stu-id="977a4-119">Computationally intensive operations such as simulation and number crunching.</span></span>
- <span data-ttu-id="977a4-120">En simulaciones de cálculo intensivas y que se deben dividir entre CPU de varios equipos (de decenas a miles).</span><span class="sxs-lookup"><span data-stu-id="977a4-120">Simulations that are computationally intensive and must be split across CPUs in multiple computers (10-1000s).</span></span>
- <span data-ttu-id="977a4-121">En simulaciones que requieren gran cantidad de memoria para un equipo y que se deben dividir en varios equipos.</span><span class="sxs-lookup"><span data-stu-id="977a4-121">Simulations that require too much memory for one computer, and must be split across multiple computers.</span></span>
- <span data-ttu-id="977a4-122">En cálculos de ejecución prolongada que tardarían demasiado tiempo en finalizar en un único equipo.</span><span class="sxs-lookup"><span data-stu-id="977a4-122">Long-running computations that would take too long to complete on a single computer.</span></span>
- <span data-ttu-id="977a4-123">En cálculos más pequeños que se deben ejecutar cientos o miles de veces, como simulaciones Monte Carlo.</span><span class="sxs-lookup"><span data-stu-id="977a4-123">Smaller computations that must be run 100s or 1000s of times, such as Monte Carlo simulations.</span></span>

## <a name="benefits"></a><span data-ttu-id="977a4-124">Ventajas</span><span class="sxs-lookup"><span data-stu-id="977a4-124">Benefits</span></span>

- <span data-ttu-id="977a4-125">Alto rendimiento con procesamiento "[embarazosamente paralelo][embarrassingly-parallel]".</span><span class="sxs-lookup"><span data-stu-id="977a4-125">High performance with "[embarrassingly parallel][embarrassingly-parallel]" processing.</span></span>
- <span data-ttu-id="977a4-126">Puede aprovechar cientos o miles de núcleos de equipos para resolver problemas de gran tamaño con mayor rapidez.</span><span class="sxs-lookup"><span data-stu-id="977a4-126">Can harness hundreds or thousands of computer cores to solve large problems faster.</span></span>
- <span data-ttu-id="977a4-127">Acceso a hardware especializado de alto rendimiento, con redes dedicadas de alta velocidad InfiniBand.</span><span class="sxs-lookup"><span data-stu-id="977a4-127">Access to specialized high-performance hardware, with dedicated high-speed InfiniBand networks.</span></span>
- <span data-ttu-id="977a4-128">Puede aprovisionar máquinas virtuales según sea necesario para trabajar y luego anularlas.</span><span class="sxs-lookup"><span data-stu-id="977a4-128">You can provision VMs as needed to do work, and then tear them down.</span></span> 

## <a name="challenges"></a><span data-ttu-id="977a4-129">Desafíos</span><span class="sxs-lookup"><span data-stu-id="977a4-129">Challenges</span></span>

- <span data-ttu-id="977a4-130">Administración de la infraestructura de la máquina virtual.</span><span class="sxs-lookup"><span data-stu-id="977a4-130">Managing the VM infrastructure.</span></span>
- <span data-ttu-id="977a4-131">Administración del volumen del procesamiento de números.</span><span class="sxs-lookup"><span data-stu-id="977a4-131">Managing the volume of number crunching.</span></span> 
- <span data-ttu-id="977a4-132">Aprovisionamiento de miles de núcleos de manera puntual.</span><span class="sxs-lookup"><span data-stu-id="977a4-132">Provisioning thousands of cores in a timely manner.</span></span>
- <span data-ttu-id="977a4-133">Para tareas estrechamente acopladas, agregar más núcleos puede provocar un rendimiento decreciente.</span><span class="sxs-lookup"><span data-stu-id="977a4-133">For tightly coupled tasks, adding more cores can have diminishing returns.</span></span> <span data-ttu-id="977a4-134">Es posible que deba experimentar para encontrar el número de núcleos óptimo.</span><span class="sxs-lookup"><span data-stu-id="977a4-134">You may need to experiment to find the optimum number of cores.</span></span>

## <a name="big-compute-using-azure-batch"></a><span data-ttu-id="977a4-135">Big compute con Azure Batch</span><span class="sxs-lookup"><span data-stu-id="977a4-135">Big compute using Azure Batch</span></span>

<span data-ttu-id="977a4-136">[Azure Batch][batch] es un servicio administrado para ejecutar aplicaciones a gran escala y de informática de alto rendimiento (HPC).</span><span class="sxs-lookup"><span data-stu-id="977a4-136">[Azure Batch][batch] is a managed service for running large-scale high-performance computing (HPC) applications.</span></span>

<span data-ttu-id="977a4-137">Con Azure Batch, configura un grupo de máquinas virtuales, y carga las aplicaciones y los archivos de datos.</span><span class="sxs-lookup"><span data-stu-id="977a4-137">Using Azure Batch, you configure a VM pool, and upload the applications and data files.</span></span> <span data-ttu-id="977a4-138">A continuación, el servicio Batch aprovisiona las máquinas virtuales, asigna tareas a estas, ejecuta las tareas y supervisa el progreso.</span><span class="sxs-lookup"><span data-stu-id="977a4-138">Then the Batch service provisions the VMs, assign tasks to the VMs, runs the tasks, and monitors the progress.</span></span> <span data-ttu-id="977a4-139">Batch puede escalar horizontalmente las máquinas virtuales de forma automática en respuesta a la carga de trabajo.</span><span class="sxs-lookup"><span data-stu-id="977a4-139">Batch can automatically scale out the VMs in response to the workload.</span></span> <span data-ttu-id="977a4-140">Batch también proporciona la programación de trabajos.</span><span class="sxs-lookup"><span data-stu-id="977a4-140">Batch also provides job scheduling.</span></span>

![](./images/big-compute-batch.png) 

## <a name="big-compute-running-on-virtual-machines"></a><span data-ttu-id="977a4-141">Big Compute en ejecución en máquinas virtuales</span><span class="sxs-lookup"><span data-stu-id="977a4-141">Big compute running on Virtual Machines</span></span>

<span data-ttu-id="977a4-142">Puede usar [Microsoft HPC Pack][hpc-pack] para administrar un clúster de máquinas virtuales, así como programar y supervisar trabajos de HPC.</span><span class="sxs-lookup"><span data-stu-id="977a4-142">You can use [Microsoft HPC Pack][hpc-pack] to administer a cluster of VMs, and schedule and monitor HPC jobs.</span></span> <span data-ttu-id="977a4-143">Con este enfoque, debe aprovisionar y administrar las máquinas virtuales y la infraestructura de red.</span><span class="sxs-lookup"><span data-stu-id="977a4-143">With this approach, you must provision and manage the VMs and network infrastructure.</span></span> <span data-ttu-id="977a4-144">Tenga en cuenta este enfoque si tienen cargas de trabajo de HPC existentes y desea mover algunas o todas a Azure.</span><span class="sxs-lookup"><span data-stu-id="977a4-144">Consider this approach if you have existing HPC workloads and want to move some or all it to Azure.</span></span> <span data-ttu-id="977a4-145">Puede mover todo el clúster de HPC a Azure o mantener el clúster de HPC en modo local pero usar Azure para la capacidad de ráfaga.</span><span class="sxs-lookup"><span data-stu-id="977a4-145">You can move the entire HPC cluster to Azure, or keep your HPC cluster on-premises but use Azure for burst capacity.</span></span> <span data-ttu-id="977a4-146">Para más información, consulte [Soluciones de Batch y HPC para cargas de trabajo de procesos a gran escala][batch-hpc-solutions].</span><span class="sxs-lookup"><span data-stu-id="977a4-146">For more information, see [Batch and HPC solutions for large-scale computing workloads][batch-hpc-solutions].</span></span>

### <a name="hpc-pack-deployed-to-azure"></a><span data-ttu-id="977a4-147">HPC Pack implementado en Azure</span><span class="sxs-lookup"><span data-stu-id="977a4-147">HPC Pack deployed to Azure</span></span>

<span data-ttu-id="977a4-148">En este escenario, el clúster de HPC se crea por completo dentro de Azure.</span><span class="sxs-lookup"><span data-stu-id="977a4-148">In this scenario, the HPC cluster is created entirely within Azure.</span></span>

![](./images/big-compute-iaas.png) 
 
<span data-ttu-id="977a4-149">El nodo principal proporciona servicios de programación de trabajos y de administración para el clúster.</span><span class="sxs-lookup"><span data-stu-id="977a4-149">The head node provides management and job scheduling services to the cluster.</span></span> <span data-ttu-id="977a4-150">Para tareas estrechamente acopladas, use una red RDMA que proporcione un ancho de banda muy alto y una comunicación de baja latencia entre las máquinas virtuales.</span><span class="sxs-lookup"><span data-stu-id="977a4-150">For tightly coupled tasks, use an RDMA network that provides very high bandwidth, low latency communication between VMs.</span></span> <span data-ttu-id="977a4-151">Para más información, consulte [Implementación de un clúster de HPC Pack 2016 en Azure][deploy-hpc-azure].</span><span class="sxs-lookup"><span data-stu-id="977a4-151">For more information see [Deploy an HPC Pack 2016 cluster in Azure][deploy-hpc-azure].</span></span>

### <a name="burst-an-hpc-cluster-to-azure"></a><span data-ttu-id="977a4-152">Expansión de clúster de HPC a Azure</span><span class="sxs-lookup"><span data-stu-id="977a4-152">Burst an HPC cluster to Azure</span></span>

<span data-ttu-id="977a4-153">En este escenario, una organización ejecuta HPC Pack local y usa máquinas virtuales de Azure para la capacidad de ráfaga.</span><span class="sxs-lookup"><span data-stu-id="977a4-153">In this scenario, an organization is running HPC Pack on-premises, and uses Azure VMs for burst capacity.</span></span> <span data-ttu-id="977a4-154">El nodo principal del clúster es local.</span><span class="sxs-lookup"><span data-stu-id="977a4-154">The cluster head node is on-premises.</span></span> <span data-ttu-id="977a4-155">ExpressRoute o VPN Gateway conecta la red local a la red virtual de Azure.</span><span class="sxs-lookup"><span data-stu-id="977a4-155">ExpressRoute or VPN Gateway connects the on-premises network to the Azure VNet.</span></span>

![](./images/big-compute-hybrid.png) 


[batch]: /azure/batch/
[batch-hpc-solutions]: /azure/batch/batch-hpc-solutions
[deploy-hpc-azure]: /azure/virtual-machines/windows/hpcpack-2016-cluster
[embarrassingly-parallel]: https://en.wikipedia.org/wiki/Embarrassingly_parallel
[hpc-pack]: https://technet.microsoft.com/library/cc514029

 
