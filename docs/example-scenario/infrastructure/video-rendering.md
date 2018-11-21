---
title: Representación de vídeo en 3D en Azure
description: Ejecute cargas de trabajo de HPC nativas en Azure mediante el servicio Azure Batch.
author: adamboeglin
ms.date: 07/13/2018
ms.openlocfilehash: 1ffdaa5467fec73a01b8caa18b71c2bc4e49abbe
ms.sourcegitcommit: 0a31fad9b68d54e2858314ca5fe6cba6c6b95ae4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/13/2018
ms.locfileid: "51610692"
---
# <a name="3d-video-rendering-on-azure"></a><span data-ttu-id="620c8-103">Representación de vídeo en 3D en Azure</span><span class="sxs-lookup"><span data-stu-id="620c8-103">3D video rendering on Azure</span></span>

<span data-ttu-id="620c8-104">La representación de vídeo en 3D es un proceso lento que requiere una cantidad de tiempo de la CPU importante para completarse.</span><span class="sxs-lookup"><span data-stu-id="620c8-104">3D video rendering is a time consuming process that requires a significant amount of CPU time to complete.</span></span> <span data-ttu-id="620c8-105">En una sola máquina, el proceso de generar un archivo de vídeo a partir de recursos estáticos puede tardar horas o incluso días según la longitud y complejidad del vídeo que se va a producir.</span><span class="sxs-lookup"><span data-stu-id="620c8-105">On a single machine, the process of generating a video file from static assets can take hours or even days depending on the length and complexity of the video you are producing.</span></span> <span data-ttu-id="620c8-106">Muchas compañías se decidirán por adquirir equipos de escritorio muy costosos para realizar estas tareas o por invertir en granjas de representación a las que puedan enviar los trabajos.</span><span class="sxs-lookup"><span data-stu-id="620c8-106">Many companies will purchase either expensive high end desktop computers to perform these tasks, or invest in large render farms that they can submit jobs to.</span></span> <span data-ttu-id="620c8-107">Sin embargo, si aprovecha las ventajas de Azure Batch, esta tecnología estará disponible cuando la necesite y se cerrará en caso contrario y todo ello sin ninguna inversión de capital.</span><span class="sxs-lookup"><span data-stu-id="620c8-107">However, by taking advantage of Azure Batch, that power is available to you when you need it and shuts itself down when you don't, all without any capital investment.</span></span>

<span data-ttu-id="620c8-108">El servicio Batch ofrece una experiencia coherente de administración y programación de trabajos, tanto si selecciona nodos de proceso de Windows Server como de Linux.</span><span class="sxs-lookup"><span data-stu-id="620c8-108">Batch gives you a consistent management experience and job scheduling, whether you select Windows Server or Linux compute nodes.</span></span> <span data-ttu-id="620c8-109">Con Batch, puede usar las aplicaciones de Windows o de Linux existentes, como AutoDesk Maya y Blender, para ejecutar trabajos de representación a gran escala en Azure.</span><span class="sxs-lookup"><span data-stu-id="620c8-109">With Batch, you can use your existing Windows or Linux applications, including AutoDesk Maya and Blender, to run large-scale render jobs in Azure.</span></span>

## <a name="relevant-use-cases"></a><span data-ttu-id="620c8-110">Casos de uso pertinentes</span><span class="sxs-lookup"><span data-stu-id="620c8-110">Relevant use cases</span></span>

<span data-ttu-id="620c8-111">Otros casos de uso pertinentes incluyen:</span><span class="sxs-lookup"><span data-stu-id="620c8-111">Other relevant use cases include:</span></span>

* <span data-ttu-id="620c8-112">Modelado 3D</span><span class="sxs-lookup"><span data-stu-id="620c8-112">3D modeling</span></span>
* <span data-ttu-id="620c8-113">Representación en Visual FX (VFX)</span><span class="sxs-lookup"><span data-stu-id="620c8-113">Visual FX (VFX) rendering</span></span>
* <span data-ttu-id="620c8-114">Transcodificación de vídeo</span><span class="sxs-lookup"><span data-stu-id="620c8-114">Video transcoding</span></span>
* <span data-ttu-id="620c8-115">Procesamiento de imágenes, corrección del color y cambio de tamaño</span><span class="sxs-lookup"><span data-stu-id="620c8-115">Image processing, color correction, and resizing</span></span>

## <a name="architecture"></a><span data-ttu-id="620c8-116">Arquitectura</span><span class="sxs-lookup"><span data-stu-id="620c8-116">Architecture</span></span>

![Introducción a la arquitectura de los componentes implicados en una solución de HPC nativa en la nube mediante Azure Batch][architecture]

<span data-ttu-id="620c8-118">En este escenario se muestra un flujo de trabajo que usa Azure Batch.</span><span class="sxs-lookup"><span data-stu-id="620c8-118">This scenario shows a workflow that uses Azure Batch.</span></span> <span data-ttu-id="620c8-119">El flujo de datos es el siguiente:</span><span class="sxs-lookup"><span data-stu-id="620c8-119">The data flows as follows:</span></span>

1. <span data-ttu-id="620c8-120">Cargue los archivos de entrada y las aplicaciones que los procesarán en su cuenta de Azure Storage.</span><span class="sxs-lookup"><span data-stu-id="620c8-120">Upload input files and the applications to process those files to your Azure Storage account.</span></span>
2. <span data-ttu-id="620c8-121">Cree un grupo de Batch de nodos de proceso en la cuenta de Batch, un trabajo para que ejecute la carga de trabajo en el grupo y tareas para ese trabajo.</span><span class="sxs-lookup"><span data-stu-id="620c8-121">Create a Batch pool of compute nodes in your Batch account, a job to run the workload on the pool, and tasks in the job.</span></span>
3. <span data-ttu-id="620c8-122">Descargue los archivos de entrada y las aplicaciones en Batch.</span><span class="sxs-lookup"><span data-stu-id="620c8-122">Download input files and the applications to Batch.</span></span>
4. <span data-ttu-id="620c8-123">Supervise la ejecución de las tareas.</span><span class="sxs-lookup"><span data-stu-id="620c8-123">Monitor task execution.</span></span>
5. <span data-ttu-id="620c8-124">Cargue la salida de la tarea.</span><span class="sxs-lookup"><span data-stu-id="620c8-124">Upload task output.</span></span>
6. <span data-ttu-id="620c8-125">Descargue los archivos de salida.</span><span class="sxs-lookup"><span data-stu-id="620c8-125">Download output files.</span></span>

<span data-ttu-id="620c8-126">Para simplificar este proceso, también puede usar los [complementos de Batch para Maya y 3ds Max][batch-plugins]</span><span class="sxs-lookup"><span data-stu-id="620c8-126">To simplify this process, you could also use the [Batch Plugins for Maya and 3ds Max][batch-plugins]</span></span>

### <a name="components"></a><span data-ttu-id="620c8-127">Componentes</span><span class="sxs-lookup"><span data-stu-id="620c8-127">Components</span></span>

<span data-ttu-id="620c8-128">Azure Batch se basa en las siguientes tecnologías de Azure:</span><span class="sxs-lookup"><span data-stu-id="620c8-128">Azure Batch builds upon the following Azure technologies:</span></span>

* <span data-ttu-id="620c8-129">Las [redes virtuales](/azure/virtual-network/virtual-networks-overview) se usan para el nodo principal y los recursos de proceso.</span><span class="sxs-lookup"><span data-stu-id="620c8-129">[Virtual Networks](/azure/virtual-network/virtual-networks-overview) are used for both the head node and the compute resources.</span></span>
* <span data-ttu-id="620c8-130">[Cuentas de Azure Storage](/azure/storage/common/storage-introduction): se utilizan para la sincronización y la retención de datos.</span><span class="sxs-lookup"><span data-stu-id="620c8-130">[Azure Storage accounts](/azure/storage/common/storage-introduction) are used for synchronization and data retention.</span></span>
* <span data-ttu-id="620c8-131">CycleCloud usa [conjuntos de escalado de máquinas virtuales][vmss] para los recursos de proceso.</span><span class="sxs-lookup"><span data-stu-id="620c8-131">[Virtual Machine Scale Sets][vmss] are used by CycleCloud for compute resources.</span></span>

## <a name="considerations"></a><span data-ttu-id="620c8-132">Consideraciones</span><span class="sxs-lookup"><span data-stu-id="620c8-132">Considerations</span></span>

### <a name="machine-sizes-available-for-azure-batch"></a><span data-ttu-id="620c8-133">Tamaños de máquina disponibles para Azure Batch</span><span class="sxs-lookup"><span data-stu-id="620c8-133">Machine Sizes available for Azure Batch</span></span>

<span data-ttu-id="620c8-134">Aunque la mayoría de los usuarios de representación elegirán recursos con una alta potencia de CPU, otras cargas de trabajo que usan conjuntos de escalado de máquinas virtuales pueden elegir las máquinas virtuales de forma diferente en función de diversos factores:</span><span class="sxs-lookup"><span data-stu-id="620c8-134">While most rendering customers will choose resources with high CPU power, other workloads using virtual machine scale sets may choose VMs differently and will depend on a number of factors:</span></span>

* <span data-ttu-id="620c8-135">¿Tiene la aplicación que se va a ejecutar un límite de memoria?</span><span class="sxs-lookup"><span data-stu-id="620c8-135">Is the application being run memory bound?</span></span>
* <span data-ttu-id="620c8-136">¿Necesita la aplicación usar GPU?</span><span class="sxs-lookup"><span data-stu-id="620c8-136">Does the application need to use GPUs?</span></span> 
* <span data-ttu-id="620c8-137">¿Son los tipos de trabajo lamentablemente paralelos o requieren conectividad InfiniBand para trabajos estrechamente acoplados?</span><span class="sxs-lookup"><span data-stu-id="620c8-137">Are the job types embarrassingly parallel or require infiniband connectivity for tightly coupled jobs?</span></span>
* <span data-ttu-id="620c8-138">Requieren un acceso de E/S rápido al almacenamiento de los nodos de proceso.</span><span class="sxs-lookup"><span data-stu-id="620c8-138">Require fast I/O to access storage on the compute Nodes.</span></span>

<span data-ttu-id="620c8-139">Azure tiene una amplia gama de tamaños de máquina virtual que pueden dar respuesta a los requisitos anteriores de las aplicaciones, algunos son específicos de HPC, pero incluso los tamaños más pequeños se pueden usar para proporcionar una implementación en malla eficaz:</span><span class="sxs-lookup"><span data-stu-id="620c8-139">Azure has a wide range of VM sizes that can address each and every one of the above application requirements, some are specific to HPC, but even the smallest sizes can be utilized to provide an effective grid implementation:</span></span>

* <span data-ttu-id="620c8-140">[Tamaños de máquina virtual de HPC][compute-hpc]: debido a la limitación de CPU característica de la representación, Microsoft normalmente recomienda máquinas virtuales de la serie H de Azure.</span><span class="sxs-lookup"><span data-stu-id="620c8-140">[HPC VM sizes][compute-hpc] Due to the CPU bound nature of rendering, Microsoft typically suggests the Azure H-Series VMs.</span></span> <span data-ttu-id="620c8-141">Estas se crean específicamente para necesidades informáticas de alto nivel, tienen tamaños de vCPU de 8 y 16 núcleos disponibles y ofrecen memoria DDR4, almacenamiento temporal SSD y tecnología Haswell E5 Intel.</span><span class="sxs-lookup"><span data-stu-id="620c8-141">This type of VM is built specifically for high end computational needs, they have 8 and 16 core vCPU sizes available, and features DDR4 memory, SSD temporary storage, and Haswell E5 Intel technology.</span></span>
* <span data-ttu-id="620c8-142">[Tamaños de máquina virtual de GPU][compute-gpu]: los tamaños de máquina virtual optimizada para GPU son máquinas virtuales especializadas con GPU de NVIDIA.</span><span class="sxs-lookup"><span data-stu-id="620c8-142">[GPU VM sizes][compute-gpu] GPU optimized VM sizes are specialized virtual machines available with single or multiple NVIDIA GPUs.</span></span> <span data-ttu-id="620c8-143">Estos tamaños están diseñados para cargas de trabajo de proceso intensivo, uso intensivo de gráficos y visualización.</span><span class="sxs-lookup"><span data-stu-id="620c8-143">These sizes are designed for compute-intensive, graphics-intensive, and visualization workloads.</span></span>
* <span data-ttu-id="620c8-144">Los tamaños NC, NCv2, NCv3 y ND están optimizados para las aplicaciones de uso intensivo de procesos y red, así como algoritmos, incluidas aplicaciones basadas en CUDA y OpenCL y simulaciones de inteligencia artificial y aprendizaje profundo.</span><span class="sxs-lookup"><span data-stu-id="620c8-144">NC, NCv2, NCv3, and ND sizes are optimized for compute-intensive and network-intensive applications and algorithms, including CUDA and OpenCL-based applications and simulations, AI, and Deep Learning.</span></span> <span data-ttu-id="620c8-145">NV, los tamaños están optimizados y diseñados para la visualización remota, streaming, juegos, codificación y escenarios VDI mediante marcos como OpenGL y DirectX.</span><span class="sxs-lookup"><span data-stu-id="620c8-145">NV sizes are optimized and designed for remote visualization, streaming, gaming, encoding, and VDI scenarios utilizing frameworks such as OpenGL and DirectX.</span></span>
* <span data-ttu-id="620c8-146">[Tamaños de máquina virtual optimizados para memoria][compute-memory]: cuando se necesita más memoria, los tamaños de máquina virtual optimizados para memoria ofrecen una mayor proporción de memoria en relación con la CPU.</span><span class="sxs-lookup"><span data-stu-id="620c8-146">[Memory optimized VM sizes][compute-memory] When more memory is required, the memory optimized VM sizes offer a higher memory-to-CPU ratio.</span></span>
* <span data-ttu-id="620c8-147">[Tamaños de máquinas virtuales de uso general][compute-general]: también hay disponibles tamaños de máquinas virtuales de uso general que ofrecen una relación más equilibrada entre memoria y CPU.</span><span class="sxs-lookup"><span data-stu-id="620c8-147">[General purposes VM sizes][compute-general] General-purpose VM sizes are also available and provide balanced CPU-to-memory ratio.</span></span>

### <a name="alternatives"></a><span data-ttu-id="620c8-148">Alternativas</span><span class="sxs-lookup"><span data-stu-id="620c8-148">Alternatives</span></span>

<span data-ttu-id="620c8-149">Si necesita más control sobre su entorno de representación en Azure o necesita una implementación híbrida, CycleCloud Computing puede ayudar a orquestar una cuadrícula de IaaS en la nube.</span><span class="sxs-lookup"><span data-stu-id="620c8-149">If you require more control over your rendering environment in Azure or need a hybrid implementation, then CycleCloud computing can help orchestrate an IaaS grid in the cloud.</span></span> <span data-ttu-id="620c8-150">El uso de las mismas tecnologías de Azure subyacentes igual que Azure Batch hace que la compilación y mantenimiento de una cuadrícula de IaaS sea un proceso eficiente.</span><span class="sxs-lookup"><span data-stu-id="620c8-150">Using the same underlying Azure technologies as Azure Batch, it makes building and maintaining an IaaS grid an efficient process.</span></span> <span data-ttu-id="620c8-151">Para más información y para conocer los principios de diseño, use el siguiente vínculo:</span><span class="sxs-lookup"><span data-stu-id="620c8-151">To find out more and learn about the design principles use the following link:</span></span>

<span data-ttu-id="620c8-152">Para obtener información general completa de todas las soluciones de HPC que están disponibles en Azure, consulte el artículo [Soluciones de Big Compute, HPC y Batch mediante máquinas virtuales de Azure][hpc-alt-solutions].</span><span class="sxs-lookup"><span data-stu-id="620c8-152">For a complete overview of all the HPC solutions that are available to you in Azure, see the article [HPC, Batch, and Big Compute solutions using Azure VMs][hpc-alt-solutions]</span></span>

### <a name="availability"></a><span data-ttu-id="620c8-153">Disponibilidad</span><span class="sxs-lookup"><span data-stu-id="620c8-153">Availability</span></span>

<span data-ttu-id="620c8-154">La supervisión de los componentes de Azure Batch se puede realizar con diferentes servicios, herramientas y API.</span><span class="sxs-lookup"><span data-stu-id="620c8-154">Monitoring of the Azure Batch components is available through a range of services, tools, and APIs.</span></span> <span data-ttu-id="620c8-155">Esto se explica con más detalle en el artículo [Supervisión de soluciones de Batch][batch-monitor].</span><span class="sxs-lookup"><span data-stu-id="620c8-155">Monitoring is discussed further in the [Monitor Batch solutions][batch-monitor] article.</span></span>

### <a name="scalability"></a><span data-ttu-id="620c8-156">Escalabilidad</span><span class="sxs-lookup"><span data-stu-id="620c8-156">Scalability</span></span>

<span data-ttu-id="620c8-157">Los grupos de una cuenta de Azure Batch se pueden escalar de forma manual o automática mediante una fórmula basada en métricas de Azure Batch.</span><span class="sxs-lookup"><span data-stu-id="620c8-157">Pools within an Azure Batch account can either scale through manual intervention or, by using a formula based on Azure Batch metrics, be scaled automatically.</span></span> <span data-ttu-id="620c8-158">Para más información sobre escalabilidad, consulte el artículo [Creación de una fórmula de escalado automática para escalar nodos de proceso en un grupo de Batch][batch-scaling].</span><span class="sxs-lookup"><span data-stu-id="620c8-158">For more information on scalability, see the article [Create an automatic scaling formula for scaling nodes in a Batch pool][batch-scaling].</span></span>

### <a name="security"></a><span data-ttu-id="620c8-159">Seguridad</span><span class="sxs-lookup"><span data-stu-id="620c8-159">Security</span></span>

<span data-ttu-id="620c8-160">Para obtener instrucciones generales sobre el diseño de soluciones seguras, consulte la [documentación de seguridad de Azure][security].</span><span class="sxs-lookup"><span data-stu-id="620c8-160">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="620c8-161">Resistencia</span><span class="sxs-lookup"><span data-stu-id="620c8-161">Resiliency</span></span>

<span data-ttu-id="620c8-162">Aunque actualmente no hay ninguna funcionalidad de conmutación por error en Azure Batch, se recomienda usar los siguientes pasos para garantizar la disponibilidad en el caso de una interrupción imprevista:</span><span class="sxs-lookup"><span data-stu-id="620c8-162">While there is currently no failover capability in Azure Batch, we recommend using the following steps to ensure availability if there is an unplanned outage:</span></span>

* <span data-ttu-id="620c8-163">Cree una cuenta de Azure Batch en una ubicación alternativa de Azure con una cuenta de almacenamiento alternativa</span><span class="sxs-lookup"><span data-stu-id="620c8-163">Create an Azure Batch account in an alternate Azure location with an alternate Storage Account</span></span>
* <span data-ttu-id="620c8-164">Cree los mismos grupos de nodos con el mismo nombre, con cero nodos asignados</span><span class="sxs-lookup"><span data-stu-id="620c8-164">Create the same node pools with the same name, with zero nodes allocated</span></span>
* <span data-ttu-id="620c8-165">Asegúrese de que se han creado y actualizado las aplicaciones en la cuenta de almacenamiento alternativa</span><span class="sxs-lookup"><span data-stu-id="620c8-165">Ensure Applications are created and updated to the alternate Storage Account</span></span>
* <span data-ttu-id="620c8-166">Cargue los archivos de entrada y envíe trabajos a la cuenta de Azure Batch alternativa</span><span class="sxs-lookup"><span data-stu-id="620c8-166">Upload input files and submit jobs to the alternate Azure Batch account</span></span>

## <a name="deploy-this-scenario"></a><span data-ttu-id="620c8-167">Implementación de este escenario</span><span class="sxs-lookup"><span data-stu-id="620c8-167">Deploy this scenario</span></span>

### <a name="creating-an-azure-batch-account-and-pools-manually"></a><span data-ttu-id="620c8-168">Creación manual de una cuenta y de grupos de Azure Batch</span><span class="sxs-lookup"><span data-stu-id="620c8-168">Creating an Azure Batch account and pools manually</span></span>

<span data-ttu-id="620c8-169">En este escenario se muestra cómo funciona Azure Batch y un ejemplo de Azure Batch Labs como solución de SaaS que se puede desarrollar para los clientes propios:</span><span class="sxs-lookup"><span data-stu-id="620c8-169">This scenario demonstrates how Azure Batch works while showcasing Azure Batch Labs as an example SaaS solution that can be developed for your own customers:</span></span>

<span data-ttu-id="620c8-170">[Azure Batch Masterclass][batch-labs-masterclass]</span><span class="sxs-lookup"><span data-stu-id="620c8-170">[Azure Batch Masterclass][batch-labs-masterclass]</span></span>

### <a name="deploying-the-example-scenario-using-an-azure-resource-manager-template"></a><span data-ttu-id="620c8-171">Implementación del escenario de ejemplo mediante una plantilla de Azure Resource Manager</span><span class="sxs-lookup"><span data-stu-id="620c8-171">Deploying the example scenario using an Azure Resource Manager template</span></span>

<span data-ttu-id="620c8-172">La plantilla implementará:</span><span class="sxs-lookup"><span data-stu-id="620c8-172">The template will deploy:</span></span>

* <span data-ttu-id="620c8-173">Una nueva cuenta de Azure Batch</span><span class="sxs-lookup"><span data-stu-id="620c8-173">A new Azure Batch account</span></span>
* <span data-ttu-id="620c8-174">Una cuenta de almacenamiento</span><span class="sxs-lookup"><span data-stu-id="620c8-174">A storage account</span></span>
* <span data-ttu-id="620c8-175">Un grupo de nodos asociado con la cuenta de Batch</span><span class="sxs-lookup"><span data-stu-id="620c8-175">A node pool associated with the batch account</span></span>
* <span data-ttu-id="620c8-176">El grupo de nodos se configurará para usar máquinas virtuales A2 v2 con imágenes de Canonical Ubuntu</span><span class="sxs-lookup"><span data-stu-id="620c8-176">The node pool will be configured to use A2 v2 VMs with Canonical Ubuntu images</span></span>
* <span data-ttu-id="620c8-177">El grupo de nodos contendrá inicialmente cero máquinas virtuales y necesitará escalado manual para agregar máquinas virtuales</span><span class="sxs-lookup"><span data-stu-id="620c8-177">The node pool will contain zero VMs initially and will require you to manually scale to add VMs</span></span>

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fhpc%2Fbatchcreatewithpools.json" target="_blank">
    <img src="https://azuredeploy.net/deploybutton.png"/>
</a>

<span data-ttu-id="620c8-178">Obtenga más información sobre las [plantillas de Resource Manager][azure-arm-templates].</span><span class="sxs-lookup"><span data-stu-id="620c8-178">[Learn more about Resource Manager templates][azure-arm-templates]</span></span>

## <a name="pricing"></a><span data-ttu-id="620c8-179">Precios</span><span class="sxs-lookup"><span data-stu-id="620c8-179">Pricing</span></span>

<span data-ttu-id="620c8-180">El costo de usar Azure Batch dependerá de los tamaños de máquina virtual que se usen para los grupos y de cuánto tiempo están estas máquinas virtuales asignadas y en ejecución; no hay ningún costo asociado con la creación de una cuenta de Azure Batch.</span><span class="sxs-lookup"><span data-stu-id="620c8-180">The cost of using Azure Batch will depend on the VM sizes that are used for the pools and how long these VMs are allocated and running, there is no cost associated with an Azure Batch account creation.</span></span> <span data-ttu-id="620c8-181">También se deben tener en cuenta el almacenamiento y la salida de datos, ya que estos supondrán costos adicionales.</span><span class="sxs-lookup"><span data-stu-id="620c8-181">Storage and data egress should be taken into account as these will apply additional costs.</span></span>

<span data-ttu-id="620c8-182">Los siguientes son ejemplos de costos en los que podría incurrir un trabajo que se completa en 8 horas y que utiliza un número variable de servidores:</span><span class="sxs-lookup"><span data-stu-id="620c8-182">The following are examples of costs that could be incurred for a job that completes in 8 hours using a different number of servers:</span></span>

* <span data-ttu-id="620c8-183">100 máquinas virtuales de alto rendimiento de CPU: [Estimación de costos][hpc-est-high]</span><span class="sxs-lookup"><span data-stu-id="620c8-183">100 High-Performance CPU VMs: [Cost Estimate][hpc-est-high]</span></span>

  <span data-ttu-id="620c8-184">100 x H16m (16 núcleos, 225 GB de RAM, Premium Storage de 512 GB), almacenamiento de blobs de 2 TB, salida de 1 TB</span><span class="sxs-lookup"><span data-stu-id="620c8-184">100 x H16m (16 cores, 225 GB RAM, Premium Storage 512 GB), 2 TB Blob Storage, 1-TB egress</span></span>

* <span data-ttu-id="620c8-185">50 máquinas virtuales de alto rendimiento de CPU: [Estimación de costos][hpc-est-med]</span><span class="sxs-lookup"><span data-stu-id="620c8-185">50 High-Performance CPU VMs: [Cost Estimate][hpc-est-med]</span></span>

  <span data-ttu-id="620c8-186">50 x H16m (16 núcleos, 225 GB de RAM, Premium Storage de 512 GB), almacenamiento de blobs de 2 TB, salida de 1 TB</span><span class="sxs-lookup"><span data-stu-id="620c8-186">50 x H16m (16 cores, 225 GB RAM, Premium Storage 512 GB), 2 TB Blob Storage, 1-TB egress</span></span>

* <span data-ttu-id="620c8-187">10 máquinas virtuales de alto rendimiento de CPU: [Estimación de costos][hpc-est-low]</span><span class="sxs-lookup"><span data-stu-id="620c8-187">10 High-Performance CPU VMs: [Cost Estimate][hpc-est-low]</span></span>

  <span data-ttu-id="620c8-188">10 x H16m (16 núcleos, 225 GB de RAM, Premium Storage de 512 GB), almacenamiento de blobs de 2 TB, salida de 1 TB</span><span class="sxs-lookup"><span data-stu-id="620c8-188">10 x H16m (16 cores, 225 GB RAM, Premium Storage 512 GB), 2 TB Blob Storage, 1-TB egress</span></span>

### <a name="pricing-for-low-priority-vms"></a><span data-ttu-id="620c8-189">Precios de las máquinas virtuales de prioridad baja</span><span class="sxs-lookup"><span data-stu-id="620c8-189">Pricing for low-priority VMs</span></span>

<span data-ttu-id="620c8-190">Azure Batch también admite el uso de máquinas virtuales de prioridad baja en los grupos de nodos, lo que podría suponer un ahorro de costos considerable.</span><span class="sxs-lookup"><span data-stu-id="620c8-190">Azure Batch also supports the use of low-priority VMs in the node pools, which can potentially provide a substantial cost saving.</span></span> <span data-ttu-id="620c8-191">Para más información, una comparación de precios entre las máquinas virtuales estándar y las máquinas virtuales de prioridad baja incluida, consulte [Precios de Batch][batch-pricing].</span><span class="sxs-lookup"><span data-stu-id="620c8-191">For more information, including a price comparison between standard VMs and low-priority VMs, see [Azure Batch Pricing][batch-pricing].</span></span>

> [!NOTE] 
> <span data-ttu-id="620c8-192">Las máquinas virtuales de prioridad baja solo son adecuadas para determinadas aplicaciones y cargas de trabajo.</span><span class="sxs-lookup"><span data-stu-id="620c8-192">Low-priority VMs are only suitable for certain applications and workloads.</span></span>

## <a name="related-resources"></a><span data-ttu-id="620c8-193">Recursos relacionados</span><span class="sxs-lookup"><span data-stu-id="620c8-193">Related resources</span></span>

<span data-ttu-id="620c8-194">[Introducción a Azure Batch][batch-overview]</span><span class="sxs-lookup"><span data-stu-id="620c8-194">[Azure Batch Overview][batch-overview]</span></span>

<span data-ttu-id="620c8-195">[Documentación de Azure Batch][batch-doc]</span><span class="sxs-lookup"><span data-stu-id="620c8-195">[Azure Batch Documentation][batch-doc]</span></span>

<span data-ttu-id="620c8-196">[Uso de contenedores en Azure Batch][batch-containers]</span><span class="sxs-lookup"><span data-stu-id="620c8-196">[Using containers on Azure Batch][batch-containers]</span></span>

<!-- links -->
[architecture]: ./media/architecture-video-rendering.png
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[security]: /azure/security/
[resiliency]: /azure/architecture/resiliency/
[scalability]: /azure/architecture/checklist/scalability
[vmss]: /azure/virtual-machine-scale-sets/overview
[storage]: https://azure.microsoft.com/services/storage/
[batch]: https://azure.microsoft.com/services/batch/
[batch-arch]: https://azure.microsoft.com/solutions/architecture/big-compute-with-azure-batch/
[compute-hpc]: /azure/virtual-machines/windows/sizes-hpc
[compute-gpu]: /azure/virtual-machines/windows/sizes-gpu
[compute-compute]: /azure/virtual-machines/windows/sizes-compute
[compute-memory]: /azure/virtual-machines/windows/sizes-memory
[compute-general]: /azure/virtual-machines/windows/sizes-general
[compute-storage]: /azure/virtual-machines/windows/sizes-storage
[compute-acu]: /azure/virtual-machines/windows/acu
[compute=benchmark]: /azure/virtual-machines/windows/compute-benchmark-scores
[hpc-est-high]: https://azure.com/e/9ac25baf44ef49c3a6b156935ee9544c
[hpc-est-med]: https://azure.com/e/0286f1d6f6784310af4dcda5aec8c893
[hpc-est-low]: https://azure.com/e/e39afab4e71949f9bbabed99b428ba4a
[batch-labs-masterclass]: https://github.com/azurebigcompute/BigComputeLabs/tree/master/Azure%20Batch%20Masterclass%20Labs
[batch-scaling]: /azure/batch/batch-automatic-scaling
[hpc-alt-solutions]: /azure/virtual-machines/linux/high-performance-computing?toc=%2fazure%2fbatch%2ftoc.json
[batch-monitor]: /azure/batch/monitoring-overview
[batch-pricing]: https://azure.microsoft.com/pricing/details/batch/
[batch-doc]: /azure/batch/
[batch-overview]: https://azure.microsoft.com/services/batch/
[batch-containers]: https://github.com/Azure/batch-shipyard
[azure-arm-templates]: /azure/azure-resource-manager/resource-group-overview#template-deployment
[batch-plugins]: /azure/batch/batch-rendering-service#options-for-submitting-a-render-job