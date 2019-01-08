---
title: Un servicio de ingeniería asistida por PC
titleSuffix: Azure Example Scenarios
description: Proporcione una plataforma de software como servicio (SaaS) de ingeniería asistida por PC (CAE) en Azure.
author: alexbuckgit
ms.date: 08/22/2018
ms.openlocfilehash: 0e8ce29639e4e189acef633585191b178c6c8721
ms.sourcegitcommit: bb7fcffbb41e2c26a26f8781df32825eb60df70c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/20/2018
ms.locfileid: "53643873"
---
# <a name="a-computer-aided-engineering-service-on-azure"></a>Un servicio de ingeniería asistida por PC en Azure

Este escenario de ejemplo muestra la entrega de una plataforma de software como servicio (SaaS) basada en las capacidades de informática de alto rendimiento (HPC) de Azure. Este escenario se basa en una solución de ingeniería de software. De todas formas, la arquitectura es significativa para otros sectores que requieren los recursos HPC como la representación en imágenes, los modelos complejos y el cálculo de riesgos financieros.

En este ejemplo se muestra un proveedor de software de ingeniería, que ofrece aplicaciones de ingeniería asistida por PC (CAE) a compañías de ingeniería y empresas de fabricación. Las soluciones CAE posibilitan la innovación, reducen los tiempos de desarrollo y los costos durante la duración del diseño de un producto. Estas soluciones requieren recursos de proceso considerables y generalmente procesan grandes volúmenes de datos. El elevado costo de una aplicación de HPC local o de estaciones de trabajo de tecnología avanzada, a menudo mantienen a estas tecnologías fuera del alcance de pequeñas compañías y empresarios de ingeniería y de los estudiantes.

La empresa quiere expandir el mercado para sus aplicaciones mediante la creación de una plataforma de SaaS respaldada por tecnologías HPC basadas en la nube. Sus clientes deben poder pagar por los recursos de proceso según los necesiten y tener acceso a una enorme capacidad informática que de otra forma resultaría inasequible.

Los objetivos de la empresa incluyen:

- Sacar provecho de las funcionalidades de HPC en Azure para acelerar el diseño del producto y el proceso de pruebas.
- Usar las últimas innovaciones de hardware para ejecutar simulaciones complejas, mientras se minimizan los costos para simulaciones más sencillas.
- Habilitar la visualización de gran realismo y la representación en un explorador web, sin necesidad de una estación de trabajo de ingeniería de gama alta.

## <a name="relevant-use-cases"></a>Casos de uso pertinentes

Otros casos de uso pertinentes incluyen:

- Investigación genómica
- Simulación de tiempo metereológico
- Aplicaciones de química computacional

## <a name="architecture"></a>Arquitectura

![Arquitectura de una solución de SaaS que habilita las funcionalidades de HPC][architecture]

- Los usuarios pueden acceder a máquinas virtuales de serie NV mediante un explorador con una conexión RDP basada en HTML5 usando el [servicio Apache Guacamole](https://guacamole.apache.org/). Estas instancias de máquina virtual proporcionan GPU eficaces para las tareas de representación y colaboración. Los usuarios pueden modificar los diseños y ver los resultados sin necesidad de acceso a equipos portátiles o dispositivos móviles de alta gama. El programador pone en marcha máquinas virtuales adicionales en función de la heurística definida por el usuario.
- Desde una sesión de escritorio CAD, los usuarios pueden enviar cargas de trabajo para su ejecución en los nodos de clúster HPC disponibles. Estas cargas de trabajo realizan tareas tales como el análisis de tensión o cálculos de dinámica de fluidos computacional, lo que elimina la necesidad de clústeres de proceso locales dedicados. Estos nodos del clúster se pueden configurar para la escalabilidad automática en función de la carga o de la profundidad de cola según la demanda del usuario activo para los recursos de proceso.
- Azure Kubernetes Service (AKS) se utiliza para hospedar los recursos web disponibles para los usuarios finales.

### <a name="components"></a>Componentes

- [Máquinas virtuales de serie H](/azure/virtual-machines/linux/sizes-hpc) se utilizan para ejecutar simulaciones de proceso intensivo, como el modelado molecular y la dinámica de fluidos computacional. La solución también aprovecha tecnologías como la conectividad de acceso directo a memoria remota (RDMA) y la red InfiniBand.
- [Las máquinas virtuales de serie NV](/azure/virtual-machines/windows/sizes-gpu) proporciona a los ingenieros la funcionalidad de una estación de trabajo de alta gama desde un explorador web estándar. Estas máquinas virtuales tienen GPU NVIDIA Tesla M60 que admiten la representación avanzada y pueden ejecutar cargas de trabajo de precisión sencilla.
- [Las máquinas virtuales de uso general](/azure/virtual-machines/linux/sizes-general) que ejecutan CentOS controlan cargas de trabajo más tradicionales como las aplicaciones web.
- [Application Gateway](/azure/application-gateway/overview) equilibra la carga de las solicitudes que entran en los servidores web.
- [Azure Kubernetes Service (AKS)](/azure/aks/intro-kubernetes) se usa para ejecutar cargas de trabajo escalables con un costo menor para simulaciones que no requieren las funcionalidades de gama alta de máquinas virtuales HPC o GPU.
- [Altair PBS Works Suite](https://www.pbsworks.com/PBSProduct.aspx?n=PBS-Works-Suite&c=Overview-and-Capabilities) organiza el flujo de trabajo HPC, lo que garantiza que hay suficientes instancias de máquina virtual disponibles para controlar la carga actual en cada momento. También desasigna máquinas virtuales cuando la demanda es inferior para reducir los costos.
- [Blob Storage](/azure/storage/blobs/storage-blobs-introduction) almacena los archivos que admiten los trabajos programados.

### <a name="alternatives"></a>Alternativas

- [Azure CycleCloud](/azure/cyclecloud/overview) simplifica la creación, administración, funcionamiento y optimización de los clústeres de HPC. Ofrece una directiva avanzada y características de gobierno. CycleCloud admite cualquier programador de trabajos o pila de software.
- [HPC Pack](/azure/virtual-machines/windows/hpcpack-cluster-options) puede crear y administrar un clúster de HPC de Azure para cargas de trabajo basadas en Windows Server. HPC Pack no es una opción para cargas de trabajo basadas en Linux.
- [Azure Automation State Configuration](/azure/automation/automation-dsc-overview) proporciona un enfoque de infraestructura como código para definir las máquinas virtuales y software que se van a implementar. Las máquinas virtuales se pueden implementar como parte de un conjunto de escalado de máquinas virtuales con reglas de escalabilidad automática para los nodos de proceso en función del número de trabajos enviados a la cola de trabajos. Cuando se necesita una nueva máquina virtual, se aprovisiona con la imagen de revisión más reciente desde la Galería de imágenes de Azure y, a continuación, se instala y configura el software necesario mediante un script de configuración de DSC de PowerShell.
- [Azure Functions](/azure/azure-functions/functions-overview)

## <a name="considerations"></a>Consideraciones

- Si bien el uso de un enfoque de infraestructura como código es una excelente manera de administrar las definiciones de compilación de la máquina virtual, aprovisionar una nueva máquina virtual mediante un script puede llevar mucho tiempo. Esta solución ha encontrado un buen término medio con el uso del script de DSC para crear periódicamente una imagen maestra, que a su vez puede utilizarse para aprovisionar una nueva máquina virtual en menos tiempo que el que lleva crear una máquina virtual a petición mediante DSC. Los servicios Azure DevOps u otras herramientas de CI/CD pueden actualizar periódicamente las imágenes maestras mediante scripts de DSC.
- El equilibrio entre los costos de la solución global y una rápida disponibilidad de recursos de proceso es un factor clave a considerar. El aprovisionamiento de un grupo de instancias de máquina virtual de la serie N y su colocación en un estado desasignado reduce los costos operativos. Cuando se necesita una máquina virtual adicional, la reasignación de una instancia existente implicará encender la máquina virtual en un host diferente, pero el tiempo de detección del bus PCI que necesita el sistema operativo para identificar e instalar controladores de la GPU se elimina, porque una máquina virtual desaprovisionada que se vuelve a aprovisionar conserva el mismo bus PCI para la GPU tras el reinicio.
- La arquitectura original se basaba completamente en máquinas virtuales de Azure para ejecutar simulaciones. Con el fin de reducir los costos de cargas de trabajo que no requieren todas las funcionalidades de una máquina virtual, estas cargas de trabajo se agruparon en contenedores y se implementaron en Azure Kubernetes Service (AKS).
- Los trabajadores de la compañía tenían las habilidades y conocimientos existentes en tecnologías de código abierto. Pueden aprovechar las ventajas de estas habilidades y conocimientos trabajando sobre las bases de tecnologías, como Linux y Kubernetes.

## <a name="pricing"></a>Precios

Para ayudarle a explorar el costo de ejecución de este escenario, muchos de los servicios necesarios están preconfigurados en un [ejemplo de la calculadora de costos][calculator]. Los costos de la solución son dependientes del número y la escala de los servicios necesarios para satisfacer sus necesidades.

Las siguientes consideraciones supondrán una parte considerable de los costos de esta solución:

- Los costos de máquina virtual de Azure aumentarán linealmente a medida que se aprovisionen instancias adicionales. Las máquinas virtuales que se desasignen supondrán solo costos de almacenamiento y no de proceso. Estas máquinas desasignadas se pueden reasignar cuando la demanda sea alta.
- Los costos de Azure Kubernetes Service se basan en el tipo de máquina virtual elegido para soportar la carga de trabajo. Los costos, aumentará linealmente en función del número de máquinas virtuales del clúster.

## <a name="next-steps"></a>Pasos siguientes

- Lea la [historia cliente de Altair][source-document]. El escenario de este ejemplo se basa en una versión de su arquitectura.
- Revise otras [soluciones Big Compute](https://azure.microsoft.com/solutions/big-compute) disponibles en Azure.

<!-- links -->
[architecture]: ./media/architecture-hpc-saas.png
[source-document]: https://customers.microsoft.com/story/altair-manufacturing-azure
[calculator]: https://azure.com/e/3cb9ccdc893f41ffbcdb00c328178ccf
