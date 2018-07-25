---
title: Representación de vídeo en 3D en Azure
description: Ejecución de cargas de trabajo de HPC nativas en Azure mediante el servicio Azure Batch
author: adamboeglin
ms.date: 07/13/2018
ms.openlocfilehash: b3af0641642d7ec4b022e8c96f51693eeb0adee4
ms.sourcegitcommit: 71cbef121c40ef36e2d6e3a088cb85c4260599b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/14/2018
ms.locfileid: "39061006"
---
# <a name="3d-video-rendering-on-azure"></a>Representación de vídeo en 3D en Azure

La representación en 3D es un proceso lento que requiere una cantidad significativa de tiempo de la CPU para completarse.  En una sola máquina, el proceso de generar un archivo de vídeo a partir de recursos estáticos puede tardar horas o incluso días según la longitud y complejidad del vídeo que se va a producir.  Muchas compañías se decidirán por adquirir equipos de escritorio muy costosos para realizar estas tareas o por invertir en granjas de representación a las que puedan enviar los trabajos.  Sin embargo, si aprovecha las ventajas de Azure Batch, esta tecnología estará disponible cuando la necesite y se cerrará en caso contrario y todo ello sin ninguna inversión de capital.

El servicio Batch ofrece una experiencia coherente de administración y programación de trabajos, tanto si selecciona nodos de proceso de Windows Server como de Linux. Con Batch, puede usar las aplicaciones de Windows o de Linux existentes, como AutoDesk Maya y Blender, para ejecutar trabajos de representación a gran escala en Azure.

## <a name="related-use-cases"></a>Casos de uso relacionados

Tenga en cuenta este escenario para otros casos de uso parecidos:

* Modelado 3D
* Representación en Visual FX (VFX)
* Transcodificación de vídeo
* Procesamiento de imágenes, corrección del color y cambio de tamaño

## <a name="architecture"></a>Arquitectura

![Introducción a la arquitectura de los componentes implicados en una solución de HPC nativa en la nube mediante Azure Batch][architecture]

Este escenario de ejemplo incluye el flujo de trabajo cuando se utiliza Azure Batch y los datos fluyen de la siguiente manera:

1. Cargue los archivos de entrada y las aplicaciones que los procesarán en su cuenta de Azure Storage.
2. Cree un grupo de Batch de nodos de proceso en la cuenta de Batch, un trabajo para que ejecute la carga de trabajo en el grupo y tareas para ese trabajo.
3. Descargue los archivos de entrada y las aplicaciones en Batch.
4. Supervisar la ejecución de las tareas
5. Cargue la salida de la tarea.
6. Descargue los archivos de salida.

Para simplificar este proceso, también puede usar los [complementos de Batch para Maya y 3ds Max][batch-plugins]

### <a name="components"></a>Componentes

Azure Batch se basa en las siguientes tecnologías de Azure:

* Un [grupo de recursos][resource-groups] es un contenedor lógico de recursos de Azure.
* Las [redes virtuales][vnet] se usan para el nodo principal y los recursos de proceso
* Las cuentas de [almacenamiento][storage] se utilizan para la sincronización y la retención de datos
* CycleCloud usa [conjuntos de escalado de máquinas virtuales][vmss] para los recursos de proceso

## <a name="considerations"></a>Consideraciones

### <a name="machine-sizes-available-for-azure-batch"></a>Tamaños de máquina disponibles para Azure Batch
Aunque la mayoría de los clientes de representación elegirán recursos con una alta potencia de CPU, otras cargas de trabajo que usan conjuntos de escalado de máquinas virtuales pueden elegir las máquinas virtuales de forma diferente en función de diversos factores:
  - ¿Tiene la aplicación que se va a ejecutar un límite de memoria?
  - ¿Necesita la aplicación usar GPU? 
  - ¿Son los tipos de trabajo embarazosamente paralelos o requieren conectividad InfiniBand para trabajos estrechamente acoplados?
  - ¿Requieren una E/S rápida en Storage en los nodos de proceso?

Azure tiene una amplia gama de tamaños de máquina virtual que pueden dar respuesta a los requisitos anteriores de las aplicaciones, algunos son específicos de HPC, pero incluso los tamaños más pequeños se pueden usar para proporcionar una implementación de cuadrícula eficaz:

  - [Tamaños de máquina virtual de HPC][compute-hpc]: debido a la limitación de CPU característica de la representación, Microsoft sugiere normalmente máquinas virtuales de la serie H de Azure.  Estas se crean específicamente con disponibilidad para necesidades informáticas de alto nivel, tienen tamaños de vCPU de 8 y 16 núcleos disponibles y ofrecen memoria DDR4, almacenamiento temporal SSD y tecnología Haswell E5 Intel.
  - [Tamaños de máquina virtual de GPU][compute-gpu]: los tamaños de máquina virtual optimizada para GPU son máquinas virtuales especializadas con GPU de NVIDIA. Estos tamaños están diseñados para cargas de trabajo de proceso intensivo, uso intensivo de gráficos y visualización.
    - Los tamaños NC, NCv2, NCv3 y ND están optimizados para las aplicaciones de uso intensivo de procesos y red, así como algoritmos, incluidas aplicaciones basadas en CUDA y OpenCL y simulaciones de inteligencia artificial y aprendizaje profundo. NV, los tamaños están optimizados y diseñados para la visualización remota, streaming, juegos, codificación y escenarios VDI mediante marcos como OpenGL y DirectX.
  - [Tamaños de máquina virtual optimizados para memoria][compute-memory]: cuando se necesita más memoria, los tamaños de máquina virtual optimizados para memoria ofrecen una mayor proporción de memoria en relación con la CPU.
  - [Tamaños de máquinas virtuales de uso general][compute-general]: también hay disponibles tamaños de máquinas virtuales de uso general que ofrecen una relación más equilibrada entre memoria y CPU.

### <a name="alternatives"></a>Alternativas

Si necesita más control sobre su entorno de representación en Azure o necesita una implementación híbrida, CycleCloud Computing puede ayudar a orquestar una cuadrícula de IaaS en la nube. El uso de las mismas tecnologías de Azure subyacentes igual que Azure Batch hace que la compilación y mantenimiento de una cuadrícula de IaaS sea un proceso eficiente. Para más información y para conocer los principios de diseño, use el siguiente vínculo:

Para obtener una información general completa de todas las soluciones de HPC que están disponibles en Azure, consulte el artículo [Soluciones de Big Compute, HPC y Batch mediante máquinas virtuales de Azure][hpc-alt-solutions]

### <a name="availability"></a>Disponibilidad

La supervisión de los componentes de Azure Batch está disponible a través de una gama de servicios, herramientas y API. Esto se explica con más detalle en el artículo [Supervisión de soluciones de Batch][batch-monitor].

### <a name="scalability"></a>Escalabilidad

Los grupos de una cuenta de Azure Batch se pueden escalar manualmente, o de forma automática mediante una fórmula basada en métricas de Azure Batch. Para más información, consulte el artículo [Creación de una fórmula de escala automática para escalar nodos de proceso en un grupo de Batch][batch-scaling].

### <a name="security"></a>Seguridad

Para obtener instrucciones generales sobre el diseño de soluciones seguras, consulte la [documentación de seguridad de Azure][security].

### <a name="resiliency"></a>Resistencia

Aunque actualmente no hay ninguna funcionalidad de conmutación por error en Azure Batch, se recomienda usar los siguientes pasos para garantizar la disponibilidad en el caso de una interrupción imprevista:

* Cree una cuenta de Azure Batch en una ubicación alternativa de Azure con una cuenta de almacenamiento alternativa
* Cree los mismos grupos de nodos con el mismo nombre, con 0 nodos asignados
* Asegúrese de que se han creado y actualizado las aplicaciones en la cuenta de almacenamiento alternativa
* Cargue los archivos de entrada y envíe trabajos a la cuenta de Azure Batch alternativa

## <a name="deploy-this-scenario"></a>Implementación de este escenario

### <a name="creating-an-azure-batch-account-and-pools-manually"></a>Creación manual de una cuenta y de grupos de Azure Batch

Este escenario de ejemplo le ayudará a comprender cómo funciona Azure Batch al tiempo que muestra Azure Batch Labs como ejemplo de solución de SaaS que se puede desarrollar para sus propios clientes:

[Azure Batch Masterclass][batch-labs-masterclass]

### <a name="deploying-the-sample-scenario-using-an-azure-resource-manager-arm-template"></a>Implementación del escenario de ejemplo mediante una plantilla de Azure Resource Manager (ARM)

La plantilla implementará:
  - Una nueva cuenta de Azure Batch
  - Una cuenta de almacenamiento
  - Un grupo de nodos asociado con la cuenta de Batch
  - El grupo de nodos se configurará para usar máquinas virtuales A2 v2 con imágenes de Canonical Ubuntu
  - El grupo de nodos contendrá inicialmente 0 máquinas virtuales y necesitará escalado manual para agregar máquinas virtuales

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fhpc%2Fbatchcreatewithpools.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

[Más información sobre plantillas de ARM][azure-arm-templates]

## <a name="pricing"></a>Precios

El costo de usar Azure Batch dependerá de los tamaños de máquina virtual que se usen para los grupos y de cuánto tiempo están estos asignados y en ejecución, no hay ningún costo asociado con la creación de una cuenta de Azure Batch. También se deben tener en cuenta el almacenamiento y la salida de datos ya que estos supondrán costos adicionales.

Los siguientes son ejemplos de costos en los que podría incurrir un trabajo que se completa en 8 horas y que utiliza un número variable de servidores:


- 100 máquinas virtuales de alto rendimiento de CPU: [Estimación de costos][hpc-est-high]

  100 x H16m (16 núcleos, 225 GB de RAM, Premium Storage de 512 GB), almacenamiento de blobs de 2 TB, salida de 1 TB

- 50 máquinas virtuales de alto rendimiento de CPU: [Estimación de costos][hpc-est-med]

  50 x H16m (16 núcleos, 225 GB de RAM, Premium Storage de 512 GB), almacenamiento de blobs de 2 TB, salida de 1 TB

- 10 máquinas virtuales de alto rendimiento de CPU: [Estimación de costos][hpc-est-low]
  
  10 x H16m (16 núcleos, 225 GB de RAM, Premium Storage de 512 GB), almacenamiento de blobs de 2 TB, salida de 1 TB

### <a name="low-priority-vm-pricing"></a>Precios de máquina virtual de prioridad baja

Azure Batch también admite el uso de máquinas virtuales de prioridad baja* en los grupos de nodos, lo que posiblemente podría proporcionar un ahorro de costos considerable. Para ver una comparación de precios entre máquinas virtuales estándar y máquinas virtuales de prioridad baja y para más información sobre estas últimas, consulte [Precios de Batch][batch-pricing].

\* Tenga en cuenta que solo determinadas aplicaciones y cargas de trabajo son adecuadas para ejecutarse en máquinas virtuales de prioridad baja.

## <a name="related-resources"></a>Recursos relacionados

[Introducción a Azure Batch][batch-overview]

[Documentación de Azure Batch][batch-doc]

[Uso de contenedores en Azure Batch][batch-containers]

<!-- links -->
[architecture]: ./media/native-hpc-ref-arch.png
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[security]: /azure/security/
[resiliency]: /azure/architecture/resiliency/
[scalability]: /azure/architecture/checklist/scalability
[vmss]: /azure/virtual-machine-scale-sets/overview
[vnet]: /azure/virtual-network/virtual-networks-overview
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
[batch-pricing]: https://azure.microsoft.com/en-gb/pricing/details/batch/
[batch-doc]: /azure/batch/
[batch-overview]: https://azure.microsoft.com/services/batch/
[batch-containers]: https://github.com/Azure/batch-shipyard
[azure-arm-templates]: /azure/azure-resource-manager/resource-group-overview#template-deployment
[batch-plugins]: /azure/batch/batch-rendering-service#options-for-submitting-a-render-job