---
title: Simulaciones de dinámicas de fluido computacionales (CFD) en Azure
description: Ejecute simulaciones de dinámicas de fluido computacionales (CFD) en Azure.
author: mikewarr
ms.date: 09/20/2018
ms.openlocfilehash: 5734e6fe707e3beb5e23f2ad2b4344ba289803bb
ms.sourcegitcommit: b2a4eb132857afa70201e28d662f18458865a48e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/05/2018
ms.locfileid: "48818588"
---
# <a name="running-computational-fluid-dynamics-cfd-simulations-on-azure"></a><span data-ttu-id="ee7f7-103">Simulaciones de dinámicas de fluido computacionales (CFD) en Azure</span><span class="sxs-lookup"><span data-stu-id="ee7f7-103">Running computational fluid dynamics (CFD) simulations on Azure</span></span>

<span data-ttu-id="ee7f7-104">Las simulaciones de dinámicas de fluidos computacionales (CFD) requieren un tiempo de proceso considerable y hardware especializado.</span><span class="sxs-lookup"><span data-stu-id="ee7f7-104">Computational Fluid Dynamics (CFD) simulations require significant compute time along with specialized hardware.</span></span> <span data-ttu-id="ee7f7-105">A medida que aumenta el uso de clúster, lo hacen también los tiempos de simulación y el uso de cuadrículas en general, por lo que se producen problemas de capacidad de reserva y tiempos prolongados en cola.</span><span class="sxs-lookup"><span data-stu-id="ee7f7-105">As cluster usage increases, simulation times and overall grid use grow, leading to issues with spare capacity and long queue times.</span></span> <span data-ttu-id="ee7f7-106">El hardware físico puede resultar costoso y no adecuarse al cambio de afluencia de uso que atraviesan las empresas.</span><span class="sxs-lookup"><span data-stu-id="ee7f7-106">Adding physical hardware can be expensive, and may not align to the usage peaks and valleys that a business goes through.</span></span> <span data-ttu-id="ee7f7-107">Gracias a Azure, muchos de estos desafíos se pueden resolver sin costo alguno.</span><span class="sxs-lookup"><span data-stu-id="ee7f7-107">By taking advantage of Azure, many of these challenges can be overcome with no capital expenditure.</span></span>

<span data-ttu-id="ee7f7-108">Azure proporciona el hardware necesario para ejecutar los trabajos de CFD en máquinas virtuales con GPU y CPU.</span><span class="sxs-lookup"><span data-stu-id="ee7f7-108">Azure provides the hardware you need to run your CFD jobs on both GPU and CPU virtual machines.</span></span> <span data-ttu-id="ee7f7-109">Los tamaños de máquina virtual con acceso directo a memoria remota (RDMA) tienen redes basadas en FDR InfiniBand que permiten la comunicación MPI (interfaz de paso de mensajes) de baja latencia.</span><span class="sxs-lookup"><span data-stu-id="ee7f7-109">RDMA (Remote Direct Memory Access) enabled VM sizes have FDR InfiniBand-based networking which allows for low latency MPI (Message Passing Interface) communication.</span></span> <span data-ttu-id="ee7f7-110">La combinación con Avere vFXT, que ofrece un sistema de archivos con clústeres a nivel empresarial, garantiza el máximo rendimiento de lectura en Azure para los clientes.</span><span class="sxs-lookup"><span data-stu-id="ee7f7-110">Combined with the Avere vFXT, which provides an enterprise-scale clustered file system, customers can ensure maximum throughput for read operations in Azure.</span></span>

<span data-ttu-id="ee7f7-111">Se puede usar Azure CycleCloud para aprovisionar los clústeres y orquestar los datos tanto en escenarios híbridos como en la nube para simplificar la creación, la administración y la optimización de los clústeres de informática de alto rendimiento.</span><span class="sxs-lookup"><span data-stu-id="ee7f7-111">To simplify the creation, management, and optimization of HPC clusters, Azure CycleCloud can be used to provision clusters and orchestrate data in both hybrid and cloud scenarios.</span></span> <span data-ttu-id="ee7f7-112">Mediante la supervisión de los trabajos pendientes, CycleCloud inicia automáticamente el proceso a petición, en el que solo paga por uso, que está conectado al programador de cargas de trabajo de su elección.</span><span class="sxs-lookup"><span data-stu-id="ee7f7-112">By monitoring the pending jobs, CycleCloud will automatically launch on-demand compute, where you only pay for what you use, connected to the workload scheduler of your choice.</span></span>

## <a name="relevant-use-cases"></a><span data-ttu-id="ee7f7-113">Casos de uso pertinentes</span><span class="sxs-lookup"><span data-stu-id="ee7f7-113">Relevant use cases</span></span>

<span data-ttu-id="ee7f7-114">Considere este escenario para estas industrias, donde podrían usarse aplicaciones CFD:</span><span class="sxs-lookup"><span data-stu-id="ee7f7-114">Consider this scenario for these industries where CFD applications could be used:</span></span>

* <span data-ttu-id="ee7f7-115">Aeronáutica</span><span class="sxs-lookup"><span data-stu-id="ee7f7-115">Aeronautics</span></span>
* <span data-ttu-id="ee7f7-116">Automoción</span><span class="sxs-lookup"><span data-stu-id="ee7f7-116">Automotive</span></span>
* <span data-ttu-id="ee7f7-117">Compilación de HVAC</span><span class="sxs-lookup"><span data-stu-id="ee7f7-117">Building HVAC</span></span>
* <span data-ttu-id="ee7f7-118">Petróleo y gas</span><span class="sxs-lookup"><span data-stu-id="ee7f7-118">Oil and gas</span></span>
* <span data-ttu-id="ee7f7-119">Ciencias biológicas</span><span class="sxs-lookup"><span data-stu-id="ee7f7-119">Life sciences</span></span>

## <a name="architecture"></a><span data-ttu-id="ee7f7-120">Arquitectura</span><span class="sxs-lookup"><span data-stu-id="ee7f7-120">Architecture</span></span>

![Diagrama de la arquitectura][architecture]

<span data-ttu-id="ee7f7-122">En este diagrama se muestra una descripción general de un diseño híbrido típico que proporciona supervisión de los trabajos en los nodos de Azure a petición:</span><span class="sxs-lookup"><span data-stu-id="ee7f7-122">This diagram shows a high-level overview of a typical hybrid design providing job monitoring of the on-demand nodes in Azure:</span></span>

1. <span data-ttu-id="ee7f7-123">Conexión al servidor de Azure CycleCloud para configurar el clúster.</span><span class="sxs-lookup"><span data-stu-id="ee7f7-123">Connect to the Azure CycleCloud server to configure the cluster.</span></span>
2. <span data-ttu-id="ee7f7-124">Configuración y creación del nodo principal del clúster, mediante máquinas con RDMA para comunicación MPI.</span><span class="sxs-lookup"><span data-stu-id="ee7f7-124">Configure and create the cluster head node, using RDMA enabled machines for MPI.</span></span>
3. <span data-ttu-id="ee7f7-125">Incorporación y configuración del nodo principal local.</span><span class="sxs-lookup"><span data-stu-id="ee7f7-125">Add and configure the on-premises head node.</span></span>
4. <span data-ttu-id="ee7f7-126">Si no hay recursos suficientes, Azure CycleCloud escala (o reduce) los recursos de proceso de Azure.</span><span class="sxs-lookup"><span data-stu-id="ee7f7-126">If there are insufficient resources, Azure CycleCloud will scale up (or down) compute resources in Azure.</span></span> <span data-ttu-id="ee7f7-127">Para evitar la sobreasignación, se puede definir un límite predeterminado.</span><span class="sxs-lookup"><span data-stu-id="ee7f7-127">A predetermined limit can be defined to prevent over allocation.</span></span>
5. <span data-ttu-id="ee7f7-128">Tareas asignadas a los nodos de ejecución.</span><span class="sxs-lookup"><span data-stu-id="ee7f7-128">Tasks allocated to the execute nodes.</span></span>
6. <span data-ttu-id="ee7f7-129">Datos almacenados en caché en Azure desde el servidor NFS local.</span><span class="sxs-lookup"><span data-stu-id="ee7f7-129">Data cached in Azure from on-premises NFS server.</span></span>
7. <span data-ttu-id="ee7f7-130">Datos leídos de Avere vFXT para la caché de Azure.</span><span class="sxs-lookup"><span data-stu-id="ee7f7-130">Data read in from the Avere vFXT for Azure cache.</span></span>
8. <span data-ttu-id="ee7f7-131">Información de los trabajos y las tareas retransmitida al servidor de Azure CycleCloud.</span><span class="sxs-lookup"><span data-stu-id="ee7f7-131">Job and task information relayed to the Azure CycleCloud server.</span></span>

### <a name="components"></a><span data-ttu-id="ee7f7-132">Componentes</span><span class="sxs-lookup"><span data-stu-id="ee7f7-132">Components</span></span>

* <span data-ttu-id="ee7f7-133">[Azure CycleCloud][cyclecloud], una herramienta para crear, administrar, operar y optimizar los clústeres de informática de alto rendimiento y Big Compute en Azure.</span><span class="sxs-lookup"><span data-stu-id="ee7f7-133">[Azure CycleCloud][cyclecloud] a tool for creating, managing, operating, and optimizing HPC and Big Compute clusters in Azure.</span></span>
* <span data-ttu-id="ee7f7-134">[Avere vFXT en Azure][avere] se usa para proporcionar un sistema de archivos con clústeres a nivel empresarial compilado para la nube.</span><span class="sxs-lookup"><span data-stu-id="ee7f7-134">[Avere vFXT on Azure][avere] is used to provide an enterprise-scale clustered file system built for the cloud.</span></span>
* <span data-ttu-id="ee7f7-135">[Máquinas virtuales de Azure][vms]: se utilizan para crear un conjunto estático de instancias de proceso.</span><span class="sxs-lookup"><span data-stu-id="ee7f7-135">[Azure Virtual Machines (VMs)][vms] are used to create a static set of compute instances.</span></span>
* <span data-ttu-id="ee7f7-136">[Virtual Machine Scale Sets (conjunto de escalado de máquinas virtuales)][vmss]: proporcionan un grupo de máquinas virtuales idénticas que Azure CycleCloud puede escalar o reducir verticalmente.</span><span class="sxs-lookup"><span data-stu-id="ee7f7-136">[Virtual Machine Scale Sets (virtual machine scale set)][vmss] provide a group of identical VMs capable of being scaled up or down by Azure CycleCloud.</span></span>
* <span data-ttu-id="ee7f7-137">[Cuentas de Azure Storage](/azure/storage/common/storage-introduction): se utilizan para la sincronización y la retención de datos.</span><span class="sxs-lookup"><span data-stu-id="ee7f7-137">[Azure Storage accounts](/azure/storage/common/storage-introduction) are used for synchronization and data retention.</span></span>
* <span data-ttu-id="ee7f7-138">[Virtual Network](/azure/virtual-network/virtual-networks-overview): permite a muchos tipos de recursos de Azure, como máquinas virtuales de Azure, comunicarse de forma segura entre ellos, con Internet y con las redes locales.</span><span class="sxs-lookup"><span data-stu-id="ee7f7-138">[Virtual Networks](/azure/virtual-network/virtual-networks-overview) enable many types of Azure resources, such as Azure Virtual Machines (VMs), to securely communicate with each other, the internet, and on-premises networks.</span></span>

### <a name="alternatives"></a><span data-ttu-id="ee7f7-139">Alternativas</span><span class="sxs-lookup"><span data-stu-id="ee7f7-139">Alternatives</span></span>

<span data-ttu-id="ee7f7-140">Los clientes pueden usar también Azure CycleCloud para crear una cuadrícula completamente en Azure.</span><span class="sxs-lookup"><span data-stu-id="ee7f7-140">Customers can also use Azure CycleCloud to create a grid entirely in Azure.</span></span> <span data-ttu-id="ee7f7-141">En esta configuración, el servidor de Azure CycleCloud se ejecuta en la suscripción de Azure.</span><span class="sxs-lookup"><span data-stu-id="ee7f7-141">In this setup, the Azure CycleCloud server is run within your Azure subscription.</span></span>

<span data-ttu-id="ee7f7-142">[Azure Batch][batch] facilita un enfoque moderno de aplicación donde no es necesaria la administración de un programador de la carga de trabajo.</span><span class="sxs-lookup"><span data-stu-id="ee7f7-142">For a modern application approach where management of a workload scheduler is not needed, [Azure Batch][batch] can help.</span></span> <span data-ttu-id="ee7f7-143">Azure Batch puede ejecutar aplicaciones de informática de alto rendimiento (HPC) en paralelo y a gran escala de manera eficaz en la nube.</span><span class="sxs-lookup"><span data-stu-id="ee7f7-143">Azure Batch can run large-scale parallel and high-performance computing (HPC) applications efficiently in the cloud.</span></span> <span data-ttu-id="ee7f7-144">Azure Batch le permite definir los recursos de proceso de Azure para ejecutar las aplicaciones en paralelo o a escala sin tener que configurar ni administrar la infraestructura de forma manual.</span><span class="sxs-lookup"><span data-stu-id="ee7f7-144">Azure Batch allows you to define the Azure compute resources to execute your applications in parallel or at scale without manually configuring or managing infrastructure.</span></span> <span data-ttu-id="ee7f7-145">Azure Batch programa tareas de proceso intensivo y agrega o quita dinámicamente recursos de proceso según sus necesidades.</span><span class="sxs-lookup"><span data-stu-id="ee7f7-145">Azure Batch schedules compute-intensive tasks and dynamically adds and removes compute resources based on your requirements.</span></span>

### <a name="scalability-and-security"></a><span data-ttu-id="ee7f7-146">Escalabilidad y seguridad</span><span class="sxs-lookup"><span data-stu-id="ee7f7-146">Scalability, and Security</span></span>

<span data-ttu-id="ee7f7-147">El escalado de los nodos de ejecución en Azure CycleCloud los se realiza manualmente o con la escalabilidad automática.</span><span class="sxs-lookup"><span data-stu-id="ee7f7-147">Scaling the execute nodes on Azure CycleCloud can be accomplished either manually or using autoscaling.</span></span> <span data-ttu-id="ee7f7-148">Para más información, consulte el artículo sobre [Escalabilidad automática de CycleCloud][cycle-scale].</span><span class="sxs-lookup"><span data-stu-id="ee7f7-148">For more information, see [CycleCloud Autoscaling][cycle-scale].</span></span>

<span data-ttu-id="ee7f7-149">Para instrucciones generales de diseño de soluciones seguras, consulte la [documentación de Azure Security Center][security].</span><span class="sxs-lookup"><span data-stu-id="ee7f7-149">For general guidance on designing secure solutions, see the [Azure security documentation][security].</span></span>

## <a name="deploy-this-scenario"></a><span data-ttu-id="ee7f7-150">Implementación de este escenario</span><span class="sxs-lookup"><span data-stu-id="ee7f7-150">Deploy this scenario</span></span>

<span data-ttu-id="ee7f7-151">Para la implementación en Azure existen ciertos requisitos previos.</span><span class="sxs-lookup"><span data-stu-id="ee7f7-151">Before deploying in Azure, some pre-requisites are required.</span></span> <span data-ttu-id="ee7f7-152">Antes de implementar la plantilla de Resource Manager, siga estos pasos:</span><span class="sxs-lookup"><span data-stu-id="ee7f7-152">Follow these steps before deploying the Resource Manager template:</span></span>
1. <span data-ttu-id="ee7f7-153">Cree una [entidad de servicio][cycle-svcprin] para recuperar los valores de appId y displayName, el nombre, la contraseña y el inquilino.</span><span class="sxs-lookup"><span data-stu-id="ee7f7-153">Create a [service principal][cycle-svcprin] for retrieving the appId, displayName, name, password, and tenant.</span></span>
2. <span data-ttu-id="ee7f7-154">Genere un [par de claves SSH][cycle-ssh] para iniciar sesión en el servidor de CycleCloud de forma segura.</span><span class="sxs-lookup"><span data-stu-id="ee7f7-154">Generate an [SSH key pair][cycle-ssh] to sign in securely to the CycleCloud server.</span></span>

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FCycleCloudCommunity%2Fcyclecloud_arm%2Fmaster%2Fazuredeploy.json" target="_blank">
    <img src="https://azuredeploy.net/deploybutton.png"/>
</a>

3. <span data-ttu-id="ee7f7-155">[Inicie sesión en el servidor CycleCloud][cycle-login] para configurar y crear un clúster.</span><span class="sxs-lookup"><span data-stu-id="ee7f7-155">[Log into the CycleCloud server][cycle-login] to configure and create a new cluster.</span></span>
4. <span data-ttu-id="ee7f7-156">[Cree un clúster][cycle-create].</span><span class="sxs-lookup"><span data-stu-id="ee7f7-156">[Create a cluster][cycle-create].</span></span>

<span data-ttu-id="ee7f7-157">La caché de Avere es una solución opcional que aumenta drásticamente el rendimiento de lectura de los datos de trabajo de una aplicación.</span><span class="sxs-lookup"><span data-stu-id="ee7f7-157">The Avere Cache is an optional solution that can drastically increase read throughput for the application job data.</span></span> <span data-ttu-id="ee7f7-158">Avere vFXT para Azure soluciona el problema de la ejecución de estas aplicaciones empresariales de informática de alto rendimiento en la nube y aprovecha a la vez los datos almacenados localmente o en Azure Blob Storage.</span><span class="sxs-lookup"><span data-stu-id="ee7f7-158">Avere vFXT for Azure solves the problem of running these enterprise HPC applications in the cloud while leveraging data stored on-premises or in Azure Blob storage.</span></span>

<span data-ttu-id="ee7f7-159">Si la organización planea una infraestructura híbrida con almacenamiento local e informática en la nube, las aplicaciones con informática de alto rendimiento pueden aparecer en Azure mediante dispositivos con almacenamiento conectado a la red con datos almacenados y acelerar las CPU virtuales según proceda.</span><span class="sxs-lookup"><span data-stu-id="ee7f7-159">For organizations that are planning for a hybrid infrastructure with both on-premises storage and cloud computing, HPC applications can “burst” into Azure using data stored in NAS devices and spin up virtual CPUs as needed.</span></span> <span data-ttu-id="ee7f7-160">El conjunto de datos nunca se mueve por completo a la nube.</span><span class="sxs-lookup"><span data-stu-id="ee7f7-160">The data set is never moved completely into the cloud.</span></span> <span data-ttu-id="ee7f7-161">Durante el procesamiento, los bytes solicitados se almacenan temporalmente en caché mediante un clúster de Avere.</span><span class="sxs-lookup"><span data-stu-id="ee7f7-161">The requested bytes are temporarily cached using an Avere cluster during processing.</span></span>

<span data-ttu-id="ee7f7-162">Para instalar y configurar Avere vFXT, siga la [Guía de instalación y configuración de Avere][avere].</span><span class="sxs-lookup"><span data-stu-id="ee7f7-162">To set up and configure an Avere vFXT installation, follow the [Avere Setup and Configuration guide][avere].</span></span>

## <a name="pricing"></a><span data-ttu-id="ee7f7-163">Precios</span><span class="sxs-lookup"><span data-stu-id="ee7f7-163">Pricing</span></span>

<span data-ttu-id="ee7f7-164">El costo de ejecutar una implementación de informática de alto rendimiento con el servidor de CycleCloud varía en función de distintos factores.</span><span class="sxs-lookup"><span data-stu-id="ee7f7-164">The cost of running an HPC implementation using CycleCloud server will vary depending on a number of factors.</span></span> <span data-ttu-id="ee7f7-165">Por ejemplo, CycleCloud se paga según el tiempo de proceso utilizado; en general, con el servidor maestro y de CycleCloud asignados y en ejecución permanentemente.</span><span class="sxs-lookup"><span data-stu-id="ee7f7-165">For example, CycleCloud is charged by the amount of compute time that is used, with the Master and CycleCloud server typically being constantly allocated and running.</span></span> <span data-ttu-id="ee7f7-166">El costo de ejecutar los nodos de ejecución dependerá del tiempo que estén en funcionamiento y del tamaño.</span><span class="sxs-lookup"><span data-stu-id="ee7f7-166">The cost of running the Execute nodes will depend on how long these are up and running as well as what size is used.</span></span> <span data-ttu-id="ee7f7-167">También se aplican los cargos de Azure normales por el almacenamiento y las redes.</span><span class="sxs-lookup"><span data-stu-id="ee7f7-167">The normal Azure charges for storage and networking also apply.</span></span>

<span data-ttu-id="ee7f7-168">Este escenario muestra cómo se ejecutan las aplicaciones de CFD en Azure, por lo que las máquinas necesitarán acceso directo a memoria remota, que solo está disponible en tamaños específicos de máquina virtual.</span><span class="sxs-lookup"><span data-stu-id="ee7f7-168">This scenario shows how CFD applications can be run in Azure, so the machines will require RDMA functionality, which is only available on specific VM sizes.</span></span> <span data-ttu-id="ee7f7-169">Los siguientes son ejemplos de costos en los que se incurre con un conjunto de escalado asignado continuamente durante ocho horas al día en un mes, con una salida de datos de 1 TB.</span><span class="sxs-lookup"><span data-stu-id="ee7f7-169">The following are examples of costs that could be incurred for a scale set that is allocated continuously for eight hours per day for one month, with data egress of 1 TB.</span></span> <span data-ttu-id="ee7f7-170">También incluye los precios del servidor de Azure CycleCloud y de la instalación de Avere vFXT para Azure:</span><span class="sxs-lookup"><span data-stu-id="ee7f7-170">It also includes pricing for the Azure CycleCloud server and the Avere vFXT for Azure install:</span></span>

* <span data-ttu-id="ee7f7-171">Región: Europa del Norte</span><span class="sxs-lookup"><span data-stu-id="ee7f7-171">Region: North Europe</span></span>
* <span data-ttu-id="ee7f7-172">Servidor de Azure CycleCloud: 1 x D3 estándar (4 x CPU, 14 GB de memoria, HDD estándar de 32 GB)</span><span class="sxs-lookup"><span data-stu-id="ee7f7-172">Azure CycleCloud Server: 1 x Standard D3 (4 x CPUs, 14 GB Memory, Standard HDD 32 GB)</span></span>
* <span data-ttu-id="ee7f7-173">Servidor maestro de Azure CycleCloud: 1 x D12 estándar (4 x CPU, 28 GB de memoria, HDD estándar de 32 GB)</span><span class="sxs-lookup"><span data-stu-id="ee7f7-173">Azure CycleCloud Master Server: 1 x Standard D12 v (4 x CPUs, 28 GB Memory, Standard HDD 32 GB)</span></span>
* <span data-ttu-id="ee7f7-174">Matriz de nodos de Azure CycleCloud: 10 x H16r estándar (16 x CPU, 112 GB de memoria)</span><span class="sxs-lookup"><span data-stu-id="ee7f7-174">Azure CycleCloud Node Array: 10 x Standard H16r (16 x CPUs, 112 GB Memory)</span></span>
* <span data-ttu-id="ee7f7-175">Avere vFXT en clúster de Azure: 3 x D16s v3 (sistema operativo de 200 GB, disco de datos SSD Premium de 1 TB)</span><span class="sxs-lookup"><span data-stu-id="ee7f7-175">Avere vFXT on Azure Cluster: 3 x D16s v3 (200 GB OS, Premium SSD 1-TB data disk)</span></span>
* <span data-ttu-id="ee7f7-176">Salida de datos: 1 TB</span><span class="sxs-lookup"><span data-stu-id="ee7f7-176">Data Egress: 1 TB</span></span>

<span data-ttu-id="ee7f7-177">Revise el [precio estimado][pricing] del hardware enumerado anteriormente.</span><span class="sxs-lookup"><span data-stu-id="ee7f7-177">Review this [price estimate][pricing] for the hardware listed above.</span></span>

## <a name="next-steps"></a><span data-ttu-id="ee7f7-178">Pasos siguientes</span><span class="sxs-lookup"><span data-stu-id="ee7f7-178">Next Steps</span></span>

<span data-ttu-id="ee7f7-179">Una vez implementado el ejemplo, obtenga más información acerca de [Azure CycleCloud][cyclecloud].</span><span class="sxs-lookup"><span data-stu-id="ee7f7-179">Once you've deployed the sample, learn more about [Azure CycleCloud][cyclecloud].</span></span>

## <a name="related-resources"></a><span data-ttu-id="ee7f7-180">Recursos relacionados</span><span class="sxs-lookup"><span data-stu-id="ee7f7-180">Related resources</span></span>

* <span data-ttu-id="ee7f7-181">[Máquinas compatibles con RDMA][rdma]</span><span class="sxs-lookup"><span data-stu-id="ee7f7-181">[RDMA Capable Machine Instances][rdma]</span></span>
* <span data-ttu-id="ee7f7-182">[Personalización de una máquina virtual con RDMA][rdma-custom]</span><span class="sxs-lookup"><span data-stu-id="ee7f7-182">[Customizing an RDMA Instance VM][rdma-custom]</span></span>

<!-- links -->
[architecture]: ./media/architecture-hpc-cfd.png
[calculator]: https://azure.com/e/
[availability]: /azure/architecture/checklist/availability
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[vmss]: /azure/virtual-machine-scale-sets/overview
[cyclecloud]: /azure/cyclecloud/
[rdma]: /azure/virtual-machines/windows/sizes-hpc#rdma-capable-instances
[gpu]: /azure/virtual-machines/windows/sizes-gpu
[hpcsizes]: /azure/virtual-machines/windows/sizes-hpc
[vms]: /azure/virtual-machines/
[low-pri]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-use-low-priority
[batch]: /azure/batch/
[avere]: https://github.com/Azure/Avere/blob/master/README.md
[cycle-prereq]: /azure/cyclecloud/quickstart-install-cyclecloud#prerequisites
[cycle-svcprin]: /azure/cyclecloud/quickstart-install-cyclecloud#service-principal
[cycle-ssh]: /azure/cyclecloud/quickstart-install-cyclecloud#ssh-keypair
[cycle-login]: /azure/cyclecloud/quickstart-install-cyclecloud#log-into-the-cyclecloud-application-server
[cycle-create]: /azure/cyclecloud/quickstart-create-and-run-cluster
[rdma]: /azure/virtual-machines/windows/sizes-hpc#rdma-capable-instances
[rdma-custom]: /azure/virtual-machines/linux/classic/rdma-cluster#customize-the-vm
[pricing]: https://azure.com/e/53030a04a2ab47a289156e2377a4247a
[cycle-scale]: /azure/cyclecloud/autoscale
