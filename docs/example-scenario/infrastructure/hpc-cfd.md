---
title: Simulaciones de dinámicas de fluido computacionales (CFD) en Azure
description: Ejecute simulaciones de dinámicas de fluido computacionales (CFD) en Azure.
author: mikewarr
ms.date: 09/20/2018
ms.custom: fasttrack
ms.openlocfilehash: 42921122d74d07bf890f55be61b04c7e9a4f4e87
ms.sourcegitcommit: a0e8d11543751d681953717f6e78173e597ae207
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/06/2018
ms.locfileid: "53004652"
---
# <a name="running-computational-fluid-dynamics-cfd-simulations-on-azure"></a>Simulaciones de dinámicas de fluido computacionales (CFD) en Azure

Las simulaciones de dinámicas de fluidos computacionales (CFD) requieren un tiempo de proceso considerable y hardware especializado. A medida que aumenta el uso de clúster, lo hacen también los tiempos de simulación y el uso de cuadrículas en general, por lo que se producen problemas de capacidad de reserva y tiempos prolongados en cola. El hardware físico puede resultar costoso y no adecuarse al cambio de afluencia de uso que atraviesan las empresas. Gracias a Azure, muchos de estos desafíos se pueden resolver sin costo alguno.

Azure proporciona el hardware necesario para ejecutar los trabajos de CFD en máquinas virtuales con GPU y CPU. Los tamaños de máquina virtual con acceso directo a memoria remota (RDMA) tienen redes basadas en FDR InfiniBand que permiten la comunicación MPI (interfaz de paso de mensajes) de baja latencia. La combinación con Avere vFXT, que ofrece un sistema de archivos con clústeres a nivel empresarial, garantiza el máximo rendimiento de lectura en Azure para los clientes.

Se puede usar Azure CycleCloud para aprovisionar los clústeres y orquestar los datos tanto en escenarios híbridos como en la nube para simplificar la creación, la administración y la optimización de los clústeres de informática de alto rendimiento. Mediante la supervisión de los trabajos pendientes, CycleCloud inicia automáticamente el proceso a petición, en el que solo paga por uso, que está conectado al programador de cargas de trabajo de su elección.

## <a name="relevant-use-cases"></a>Casos de uso pertinentes

Otros sectores pertinentes para las aplicaciones de CFD incluyen:

* Aeronáutica
* Automoción
* Compilación de HVAC
* Petróleo y gas
* Ciencias biológicas

## <a name="architecture"></a>Arquitectura

![Diagrama de la arquitectura][architecture]

En este diagrama se muestra una descripción general de un diseño híbrido típico que proporciona supervisión de los trabajos en los nodos de Azure a petición:

1. Conexión al servidor de Azure CycleCloud para configurar el clúster.
2. Configuración y creación del nodo principal del clúster, mediante máquinas con RDMA para comunicación MPI.
3. Incorporación y configuración del nodo principal local.
4. Si no hay recursos suficientes, Azure CycleCloud escala (o reduce) los recursos de proceso de Azure. Para evitar la sobreasignación, se puede definir un límite predeterminado.
5. Tareas asignadas a los nodos de ejecución.
6. Datos almacenados en caché en Azure desde el servidor NFS local.
7. Datos leídos de Avere vFXT para la caché de Azure.
8. Información de los trabajos y las tareas retransmitida al servidor de Azure CycleCloud.

### <a name="components"></a>Componentes

* [Azure CycleCloud][cyclecloud], una herramienta para crear, administrar, operar y optimizar los clústeres de informática de alto rendimiento y Big Compute en Azure.
* [Avere vFXT en Azure][avere] se usa para proporcionar un sistema de archivos con clústeres a nivel empresarial compilado para la nube.
* [Máquinas virtuales de Azure][vms]: se utilizan para crear un conjunto estático de instancias de proceso.
* [Virtual Machine Scale Sets (conjunto de escalado de máquinas virtuales)][vmss]: proporcionan un grupo de máquinas virtuales idénticas que Azure CycleCloud puede escalar o reducir verticalmente.
* [Cuentas de Azure Storage](/azure/storage/common/storage-introduction): se utilizan para la sincronización y la retención de datos.
* [Virtual Network](/azure/virtual-network/virtual-networks-overview): permite a muchos tipos de recursos de Azure, como máquinas virtuales de Azure, comunicarse de forma segura entre ellos, con Internet y con las redes locales.

### <a name="alternatives"></a>Alternativas

Los clientes pueden usar también Azure CycleCloud para crear una cuadrícula completamente en Azure. En esta configuración, el servidor de Azure CycleCloud se ejecuta en la suscripción de Azure.

[Azure Batch][batch] facilita un enfoque moderno de aplicación donde no es necesaria la administración de un programador de la carga de trabajo. Azure Batch puede ejecutar aplicaciones de informática de alto rendimiento (HPC) en paralelo y a gran escala de manera eficaz en la nube. Azure Batch le permite definir los recursos de proceso de Azure para ejecutar las aplicaciones en paralelo o a escala sin tener que configurar ni administrar la infraestructura de forma manual. Azure Batch programa tareas de proceso intensivo y agrega o quita dinámicamente recursos de proceso según sus necesidades.

### <a name="scalability-and-security"></a>Escalabilidad y seguridad

El escalado de los nodos de ejecución en Azure CycleCloud los se realiza manualmente o con la escalabilidad automática. Para más información, consulte el artículo sobre [Escalabilidad automática de CycleCloud][cycle-scale].

Para instrucciones generales de diseño de soluciones seguras, consulte la [documentación de Azure Security Center][security].

## <a name="deploy-this-scenario"></a>Implementación de este escenario

Para la implementación en Azure existen ciertos requisitos previos. Antes de implementar la plantilla de Resource Manager, siga estos pasos:
1. Cree una [entidad de servicio][cycle-svcprin] para recuperar los valores de appId y displayName, el nombre, la contraseña y el inquilino.
2. Genere un [par de claves SSH][cycle-ssh] para iniciar sesión en el servidor de CycleCloud de forma segura.

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FCycleCloudCommunity%2Fcyclecloud_arm%2Fmaster%2Fazuredeploy.json" target="_blank">
    <img src="https://azuredeploy.net/deploybutton.png"/>
</a>

3. [Inicie sesión en el servidor CycleCloud][cycle-login] para configurar y crear un clúster.
4. [Cree un clúster][cycle-create].

La caché de Avere es una solución opcional que aumenta drásticamente el rendimiento de lectura de los datos de trabajo de una aplicación. Avere vFXT para Azure soluciona el problema de la ejecución de estas aplicaciones empresariales de informática de alto rendimiento en la nube y aprovecha a la vez los datos almacenados localmente o en Azure Blob Storage.

Si la organización planea una infraestructura híbrida con almacenamiento local e informática en la nube, las aplicaciones con informática de alto rendimiento pueden aparecer en Azure mediante dispositivos con almacenamiento conectado a la red con datos almacenados y acelerar las CPU virtuales según proceda. El conjunto de datos nunca se mueve por completo a la nube. Durante el procesamiento, los bytes solicitados se almacenan temporalmente en caché mediante un clúster de Avere.

Para instalar y configurar Avere vFXT, siga la [Guía de instalación y configuración de Avere][avere].

## <a name="pricing"></a>Precios

El costo de ejecutar una implementación de informática de alto rendimiento con el servidor de CycleCloud varía en función de distintos factores. Por ejemplo, CycleCloud se paga según el tiempo de proceso utilizado; en general, con el servidor maestro y de CycleCloud asignados y en ejecución permanentemente. El costo de ejecutar los nodos de ejecución dependerá del tiempo que estén en funcionamiento y del tamaño. También se aplican los cargos de Azure normales por el almacenamiento y las redes.

Este escenario muestra cómo se ejecutan las aplicaciones de CFD en Azure, por lo que las máquinas necesitarán acceso directo a memoria remota, que solo está disponible en tamaños específicos de máquina virtual. Los siguientes son ejemplos de costos en los que se incurre con un conjunto de escalado asignado continuamente durante ocho horas al día en un mes, con una salida de datos de 1 TB. También incluye los precios del servidor de Azure CycleCloud y de la instalación de Avere vFXT para Azure:

* Región: Europa del Norte
* Servidor de Azure CycleCloud: 1 x D3 estándar (4 x CPU, 14 GB de memoria, HDD estándar de 32 GB)
* Servidor maestro de Azure CycleCloud: 1 x D12 estándar (4 x CPU, 28 GB de memoria, HDD estándar de 32 GB)
* Matriz de nodos de Azure CycleCloud: 10 x H16r estándar (16 x CPU, 112 GB de memoria)
* Avere vFXT en clúster de Azure: 3 x D16s v3 (SO de 200 GB, disco de datos de 1 TB SSD Premium)
* Salida de datos: 1 TB

Revise el [precio estimado][pricing] del hardware enumerado anteriormente.

## <a name="next-steps"></a>Pasos siguientes

Una vez implementado el ejemplo, obtenga más información acerca de [Azure CycleCloud][cyclecloud].

## <a name="related-resources"></a>Recursos relacionados

* [Máquinas compatibles con RDMA][rdma]
* [Personalización de una máquina virtual con RDMA][rdma-custom]

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
