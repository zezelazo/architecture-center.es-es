---
title: "Ejecución de máquinas virtuales de carga equilibrada en Azure para conseguir escalabilidad y disponibilidad"
description: "Cómo ejecutar varias máquinas virtuales con Windows en Azure para escalabilidad y disponibilidad."
author: telmosampaio
ms.date: 09/07/2017
pnp.series.title: Windows VM workloads
pnp.series.next: n-tier
pnp.series.prev: single-vm
ms.openlocfilehash: d38cfb41255c547f1f1e87ef289c7a79033df778
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2017
---
# <a name="run-load-balanced-vms-for-scalability-and-availability"></a>Ejecución de máquinas virtuales de carga equilibrada para escalabilidad y disponibilidad

En esta arquitectura de referencia se muestra un conjunto de prácticas demostradas para ejecutar varias máquinas virtuales con Windows en un conjunto de escalado detrás de un equilibrador de carga, para mejorar la disponibilidad y escalabilidad. Esta arquitectura puede usarse para cualquier carga de trabajo sin estado, como un servidor web, y es un bloque de creación para implementar aplicaciones de n niveles. [**Implemente esta solución**.](#deploy-the-solution)

![[0]][0]

*Descargue un [archivo Visio][visio-download] de esta arquitectura.*

## <a name="architecture"></a>Arquitectura

Esta arquitectura se basa en la que se muestra en [Ejecución de una VM con Windows en Azure][single vm]. Las recomendaciones que ahí se encuentran se aplican también a esta arquitectura.

En esta arquitectura, una carga de trabajo se distribuye entre varias instancias de máquina virtual. Hay una única dirección IP pública, y el tráfico de Internet se distribuye a las máquinas virtuales con un equilibrador de carga. Esta arquitectura puede usarse para una aplicación de nivel único, como una aplicación web sin estado.

La arquitectura consta de los siguientes componentes:

* **Grupo de recursos.** Los [*grupos de recursos*][resource-manager-overview] se utilizan para agrupar los recursos, para que puedan administrarse según su duración, su propietario y otros criterios.
* **Red virtual y subred.** Cada máquina virtual de Azure se implementa en una red virtual, que se divide a su vez en subredes.
* **Azure Load Balancer**. El [equilibrador de carga] distribuye las solicitudes entrantes de Internet a las instancias de máquina virtual. 
* **Dirección IP pública**. Se necesita una dirección IP pública para que el equilibrador de carga reciba tráfico de Internet.
* **Conjunto de escalado de VM**. Un [conjunto de escalado de VM][vm-scaleset] es un conjunto de máquinas virtuales idénticas que se utiliza para hospedar una carga de trabajo. Los conjuntos de escalado de permiten reducir o escalar horizontalmente el número de máquinas virtuales de forma manual, o bien en función de reglas predefinidas.
* **Conjunto de disponibilidad**. El [conjunto de disponibilidad][availability set] contiene las máquinas virtuales, por lo que estas son aptas para un [Acuerdo de Nivel de Servicio (SLA)][vm-sla] superior. Para que el SLA superior se aplique, el conjunto de disponibilidad debe incluir un mínimo de dos máquinas virtuales. Los conjuntos de disponibilidad están implícitos en los conjuntos de escalado. Si crea máquinas virtuales fuera de un conjunto de escaldo, debe crear un conjunto de disponibilidad independiente.
* **Managed Disks**. Azure Managed Disks administra los archivos de disco duro virtual (VHD) de los discos de máquina virtual. 
* **Storage**. Cree una cuenta de Azure Storage para almacenar registros de diagnóstico de las máquinas virtuales.

## <a name="recommendations"></a>Recomendaciones

Los requisitos pueden diferir de los de la arquitectura que se describe aquí. Use estas recomendaciones como punto inicial. 

### <a name="availability-and-scalability-recommendations"></a>Recomendaciones de disponibilidad y escalabilidad

Una opción para la disponibilidad y escalabilidad es el uso del [conjunto de escalado de máquinas virtuales][vmss]. Los conjuntos de escalado de máquinas virtuales permiten implementar y administrar un conjunto de máquinas virtuales idénticas. Los conjuntos de escalado admiten el escalado automático en función de las métricas de rendimiento. A medida que aumenta la carga en las máquinas virtuales, se agregan más máquinas virtuales automáticamente al equilibrador de carga. Considere la posibilidad de usar conjuntos de escalado si necesita escalar horizontalmente las máquinas virtuales de inmediato o si necesita realizar el escalado automático.

De forma predeterminada, los conjuntos de escalado usan el "sobreaprovisionamiento", lo que significa que el conjunto de escalado inicialmente aprovisiona más máquinas virtuales de las solicitadas y después elimina las máquinas virtuales adicionales. Esto mejora la tasa de éxito general al aprovisionar las máquinas virtuales. Si no usa [Managed Disks](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-managed-disks), se recomienda no disponer de más de veinte máquinas virtuales por cuenta de almacenamiento con el sobreaprovisionamiento habilitado ni más de cuarenta máquinas virtuales con el sobreaprovisionamiento deshabilitado.

Hay dos maneras básicas de configurar máquinas virtuales implementadas en un conjunto de escalado:

- Use extensiones para configurar la máquina virtual después de aprovisionarla. Con este método, las nuevas instancias de máquina virtual pueden tardar más en iniciarse que una máquina virtual sin extensiones.

- Implemente un [disco administrado](/azure/storage/storage-managed-disks-overview) con una imagen de disco personalizada. Esta opción puede ser más rápida de implementar. Sin embargo, requiere mantener la imagen actualizada.

Para obtener consideraciones adicionales, vea [Consideraciones de diseño para conjuntos de escalado][vmss-design].

> [!TIP]
> Cuando utilice cualquier solución de escalado automático, pruébela con antelación con cargas de trabajo de nivel de producción.

Si no utiliza un conjunto de escalado, considere la posibilidad de usar al menos un conjunto de disponibilidad. Cree al menos dos máquinas virtuales en el conjunto de disponibilidad, para dar soporte al [SLA de disponibilidad para máquinas virtuales de Azure][vm-sla]. Azure Load Balancer también requiere que las VM de carga equilibrada pertenezcan al mismo conjunto de disponibilidad.

Cada suscripción de Azure tiene límites predeterminados establecidos, incluido un número máximo de máquinas virtuales por región. Puede aumentar el límite si rellena una solicitud de soporte técnico. Para más información, vea [Límites, cuotas y restricciones de suscripción y servicios de Microsoft Azure][subscription-limits].

### <a name="network-recommendations"></a>Recomendaciones de red

Coloque las máquinas virtuales en la misma subred. No exponga las máquinas virtuales directamente a Internet, pero, en su lugar, asigne una dirección IP privada a cada máquina virtual. Los clientes se conectan con la dirección IP pública del equilibrador de carga.

Si tiene que iniciar sesión en las máquinas virtuales detrás del equilibrador de carga, considere la posibilidad de agregar una sola máquina virtual como un host bastión/JumpBox con una dirección IP pública en la que pueda iniciar sesión. Después, inicie sesión en las máquinas virtuales detrás del equilibrador de carga desde el JumpBox. Otra alternativa consiste en configurar reglas NAT de entrada en el equilibrador de carga para el mismo propósito. Sin embargo, tener un JumpBox es una solución mejor si hospeda cargas de trabajo de n niveles o varias cargas de trabajo.

### <a name="load-balancer-recommendations"></a>Recomendaciones para el equilibrador de carga

Agregue todas las máquinas virtuales del conjunto de disponibilidad al grupo de direcciones de back-end del equilibrador de carga.

Defina reglas del equilibrador de carga para dirigir el tráfico de red a las máquinas virtuales. Por ejemplo, para habilitar el tráfico HTTP, cree una regla que asigne el puerto 80 de la configuración de front-end al puerto 80 del grupo de direcciones de back-end. Cuando un cliente envía una solicitud HTTP al puerto 80, el equilibrador de carga selecciona una dirección IP de back-end mediante un [algoritmo hash][load balancer hashing] que incluye la dirección IP de origen. De ese modo, las solicitudes de cliente se distribuyen entre todas las máquinas virtuales.

Para enrutar el tráfico a una máquina virtual específica, use las reglas NAT. Por ejemplo, para habilitar RDP en las máquinas virtuales, cree una regla NAT distinta para cada máquina virtual. Cada regla debe asignar un número de puerto distinto al puerto 3389, el puerto predeterminado para RDP. Por ejemplo, use el puerto 50001 para "VM1," el puerto 50002 para "VM2", y así sucesivamente. Asigne las reglas NAT a las NIC en las máquinas virtuales.

### <a name="storage-account-recommendations"></a>Recomendaciones sobre las cuentas de almacenamiento

Cree cuentas de almacenamiento de Azure distintas para cada máquina virtual para almacenar los discos duros virtuales (VHD), con el fin de evitar alcanzar los límites de operaciones de entrada/salida por segundo [(IOPS)][vm-disk-limits] para cuentas de almacenamiento.

Se recomienda usar [Managed Disks](/azure/storage/storage-managed-disks-overview) con [Premium Storage][premium]. Managed Disks no requiere una cuenta de almacenamiento. Solo debe especificar el tamaño y el tipo de disco y se implementa en un modo de alta disponibilidad.

Cree una cuenta de almacenamiento para los registros de diagnóstico. Todas las máquinas virtuales pueden compartir esta cuenta de almacenamiento. Puede tratarse de una cuenta de almacenamiento no administrada con discos estándar.

## <a name="availability-considerations"></a>Consideraciones sobre disponibilidad

El conjunto de disponibilidad hace que la aplicación sea más resistente a eventos de mantenimiento planeado y no planeado.

* El *mantenimiento planeado* se produce cuando Microsoft actualiza la plataforma subyacente, lo que, en ocasiones, provoca el reinicio de las máquinas virtuales. Azure se asegura de que no todas las máquinas virtuales de un conjunto de disponibilidad se reinicien al mismo tiempo. Al menos una se mantiene en ejecución mientras las demás se reinician.
* El *mantenimiento no planeado* ocurre si se produce un error de hardware. Azure se asegura de que las máquinas virtuales de un conjunto de disponibilidad se aprovisionen en más de un bastidor del servidor. Esto ayuda a reducir el impacto de los errores de hardware, las interrupciones de red y las interrupciones de energía, entre otros.

Para más información, vea [Administración de la disponibilidad de las máquinas virtuales][availability set]. En el vídeo siguiente también se ofrece información general útil sobre los conjuntos de disponibilidad: [How Do I Configure an Availability Set to Scale VMs][availability set ch9] (Configuración de un conjunto de disponibilidad para escalar máquinas virtuales).

> [!WARNING]
> Asegúrese de configurar el conjunto de disponibilidad cuando aprovisione la máquina virtual. Actualmente, no hay ninguna manera de agregar una máquina virtual de Resource Manager a un conjunto de disponibilidad después de aprovisionar la máquina virtual.

El equilibrador de carga utiliza [sondeos de mantenimiento] para supervisar la disponibilidad de las instancias de máquina virtual. Si un sondeo no puede conectar con una instancia durante un período de tiempo de espera, el equilibrador de carga deja de enviar tráfico a esa máquina virtual. Sin embargo, el equilibrador de carga continuará con el sondeo y, si la máquina virtual vuelve a estar disponible, el equilibrador de carga reanudará el envío del tráfico a esa máquina virtual.

A continuación, se presentan algunas recomendaciones sobre los sondeos de mantenimiento del equilibrador de carga:

* Los sondeos pueden probar los protocolos HTTP o TCP. Si las máquinas virtuales ejecutan un servidor HTTP, cree un sondeo HTTP. De lo contrario, cree un sondeo TCP.
* Para un sondeo HTTP, especifique la ruta de acceso a un punto de conexión HTTP. El sondeo comprueba si hay una respuesta HTTP 200 desde esta ruta de acceso. Puede ser la ruta de acceso raíz ("/") o un punto de conexión de supervisión de mantenimiento que implementa alguna lógica personalizada para comprobar el mantenimiento de la aplicación. El punto de conexión debe permitir solicitudes HTTP anónimas.
* El sondeo se envía desde una dirección IP [conocida][health-probe-ip], 168.63.129.16. Asegúrese de que no bloquear el tráfico hacia o desde esta dirección IP en las directivas de firewall o en las reglas de grupo de seguridad de red (NSG).
* Use [registros de sondeo de mantenimiento][health probe log] para ver el estado de los sondeos de mantenimiento. Habilite el registro en Azure Portal para cada equilibrador de carga. Los registros se escriben en Azure Blob Storage. Los registros muestran cuántas máquinas virtuales del back-end no reciben tráfico de red debido a las respuestas de sondeo con error.

## <a name="manageability-considerations"></a>Consideraciones sobre la manejabilidad

Con varias máquinas virtuales, es importante automatizar los procesos, para que sean confiables y repetibles. Puede usar [Azure Automation][azure-automation] para automatizar la implementación, la aplicación de revisiones al SO y otras tareas. [Azure Automation][azure-automation] es un servicio de automatización basado en Windows Powershell que puede usarse para este fin. Los scripts de automatización de ejemplo están disponibles en la [Galería de runbook] de TechNet.

## <a name="security-considerations"></a>Consideraciones sobre la seguridad

Las redes virtuales son un límite de aislamiento del tráfico de Azure. Las máquinas virtuales de una red virtual no se pueden comunicar directamente con las máquinas virtuales de una red virtual diferente. Las máquinas virtuales que se encuentran en la misma red virtual se pueden comunicar entre sí, a menos que se creen [grupos de seguridad de red][nsg] (NSG) para restringir el tráfico. Para más información, vea [Servicios en la nube de Microsoft y seguridad de red][network-security].

Para el tráfico entrante de Internet, las reglas del equilibrador de carga definen qué tráfico puede alcanzar el back-end. Sin embargo, las reglas del equilibrador de carga no son compatibles con las listas seguras de IP, por lo que si desea agregar determinadas direcciones IP públicas a una lista segura, agregue un NSG a la subred.

## <a name="deploy-the-solution"></a>Implementación de la solución

Hay disponible una implementación de esta arquitectura en [GitHub][github-folder]. Implementa lo siguiente:

  * Una red virtual con una sola subred denominada **web** utilizada para hospedar la máquina virtual.
  * Un conjunto de escalado de máquinas virtuales que contiene máquinas virtuales que ejecutan la última versión de Windows Server 2016 Datacenter Edition. El escalado automático está habilitado.
  * Un equilibrador de carga que se encuentra delante del conjunto de escalado de máquinas virtuales.
  * Un NSG con reglas de entrada para permitir el tráfico HTTP al conjunto de escalado de máquinas virtuales.

### <a name="prerequisites"></a>Requisitos previos

Antes de poder implementar la arquitectura de referencia en su propia suscripción, debe realizar los pasos siguientes.

1. Clone, bifurque o descargue el archivo ZIP para el repositorio de GitHub de [arquitecturas de referencia de AzureCAT][ref-arch-repo].

2. Asegúrese de que tiene la CLI de Azure 2.0 instalada en el equipo. Para instalar la CLI, siga las instrucciones de [Instalación de la CLI de Azure 2.0][azure-cli-2].

3. Instale el paquete de NPM de [Azure Building Blocks][azbb].

4. Desde un símbolo del sistema, un símbolo del sistema de Bash o un símbolo del sistema de PowerShell, inicie sesión en la cuenta de Azure con alguno de los comandos siguientes y siga las indicaciones.

  ```bash
  az login
  ```

### <a name="deploy-the-solution-using-azbb"></a>Implementación de la solución con AZBB

Para implementar el ejemplo de carga de trabajo con una única máquina virtual, siga estos pasos:

1. Navegue hasta la carpeta `virtual-machines\multi-vm\parameters\windows` del repositorio que descargó en el paso de requisitos previos anterior.

2. Abra el archivo `multi-vm-v2.json`, escriba un nombre de usuario y la contraseña entre comillas, tal y como se muestra a continuación, y después guarde el archivo.

  ```bash
  "adminUsername": "",
  "adminPassword": "",
  ```

3. Ejecute `azbb` para implementar las máquinas virtuales, tal y como se muestra a continuación.

  ```bash
  azbb -s <subscription_id> -g <resource_group_name> -l <location> -p multi-vm-v2.json --deploy
  ```

Para más información sobre la implementación de esta arquitectura de referencia de ejemplo, visite el [repositorio de GitHub][git].

<!-- Links -->
[github-folder]: http://github.com/mspnp/reference-architectures/tree/master/virtual-machines/multi-vm
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[n-tier-linux]: ../virtual-machines-linux/n-tier.md
[n-tier-windows]: n-tier.md
[single vm]: single-vm.md
[premium]: /azure/storage/common/storage-premium-storage
[naming conventions]: /azure/guidance/guidance-naming-conventions
[vm-scaleset]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
[availability set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[availability set ch9]: https://channel9.msdn.com/Series/Microsoft-Azure-Fundamentals-Virtual-Machines/08
[azure-automation]: https://azure.microsoft.com/documentation/services/automation/
[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-automation]: /azure/automation/automation-intro
[bastion host]: https://en.wikipedia.org/wiki/Bastion_host
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/multi-vm
[health probe log]: /azure/load-balancer/load-balancer-monitor-log
[sondeos de mantenimiento]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[health-probe-ip]: /azure/virtual-network/virtual-networks-nsg#special-rules
[equilibrador de carga]: /azure/load-balancer/load-balancer-get-started-internet-arm-cli
[load balancer hashing]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[network-security]: /azure/best-practices-network-security
[nsg]: /azure/virtual-network/virtual-networks-nsg
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview 
[Galería de runbook]: /azure/automation/automation-runbook-gallery#runbooks-in-runbook-gallery
[subscription-limits]: /azure/azure-subscription-service-limits
[visio-download]: https://archcenter.azureedge.net/cdn/vm-reference-architectures.vsdx
[vm-disk-limits]: /azure/azure-subscription-service-limits#virtual-machine-disk-limits
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_2/
[vmss]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
[vmss-design]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-design-overview
[vmss-quickstart]: https://azure.microsoft.com/documentation/templates/?term=scale+set
[VM-sizes]: https://azure.microsoft.com/documentation/articles/virtual-machines-windows-sizes/
[0]: ./images/multi-vm-diagram.png "Arquitectura de una solución de varias máquinas virtuales en Azure que componen un conjunto de disponibilidad con dos máquinas virtuales y un equilibrador de carga"
[azure-cli-2]: /azure/install-azure-cli?view=azure-cli-latest
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Template-Building-Blocks-Version-2-(Windows)
[git]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/multi-vm