---
title: "Ejecución de una VM con Windows en Azure"
description: "Cómo ejecutar una única máquina virtual en Azure teniendo en cuenta la escalabilidad, resistencia, manejabilidad y seguridad."
author: telmosampaio
ms.date: 09/06/2017
pnp.series.title: Windows VM workloads
pnp.series.next: multi-vm
pnp.series.prev: ./index
ms.openlocfilehash: adedcd797208e52be58fc0ab0f37fc3da1723484
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2017
---
# <a name="run-a-windows-vm-on-azure"></a>Ejecución de una VM con Windows en Azure

En esta arquitectura de referencia se muestra un conjunto de prácticas demostradas para ejecutar una máquina virtual con Windows en Azure. Incluye recomendaciones para el aprovisionamiento de la máquina virtual junto con los componentes de red y almacenamiento. Esta arquitectura puede utilizarse para ejecutar una sola instancia, y es la base para arquitecturas más complejas, como aplicaciones de n niveles. [**Implemente esta solución**.](#deploy-the-solution)

![[0]][0]

*Descargue un [archivo Visio][visio-download] de esta arquitectura.*

## <a name="architecture"></a>Arquitectura

El aprovisionamiento de una máquina virtual en Azure implica más piezas en movimiento que la propia máquina virtual. Existen elementos de proceso, red y almacenamiento que debe tener en cuenta.

* **Grupo de recursos.** Un [*grupo de recursos*][resource-manager-overview] es un contenedor que incluye recursos relacionados. Normalmente se crean grupos de recursos para los distintos recursos de una solución en función de su duración y de quién vaya a administrar los recursos. Para una carga de trabajo de máquina virtual única, puede crear un único grupo de recursos para todos los recursos.
* **Máquina virtual**. Puede aprovisionar una máquina virtual desde una lista de imágenes publicadas o desde un archivo de disco duro virtual (VHD) cargado en Azure Blob Storage.
* **Disco del sistema operativo.** El disco del sistema operativo es un VHD almacenado en [Azure Storage][azure-storage]. Esto significa que persiste incluso si el equipo host deja de funcionar.
* **Disco temporal.** La VM se crea con un disco temporal (la unidad `D:` de Windows). Este disco se almacena en una unidad física del equipo host. *No* se guarda en Azure Storage y podría desaparecer durante los reinicios y otros eventos del ciclo de vida de la máquina virtual. Use este disco solo para datos temporales, como archivos de paginación o de intercambio.
* **Discos de datos.** Un [disco de datos][data-disk] es un VHD persistente usado para los datos de la aplicación. Los discos de datos se almacenan en Azure Storage, como el disco del sistema operativo.
* **Red virtual y subred.** Cada máquina virtual de Azure se implementa en una red virtual, que se divide a su vez en subredes.
* **Dirección IP pública.** Es necesaria una dirección IP pública para comunicarse con la máquina virtual&mdash;, por ejemplo, a través de Escritorio remoto (RDP).
* **Interfaz de red (NIC)**. La NIC permite que la VM se comunique con la red virtual.
* **Grupo de seguridad de red (NSG)**. El [NSG][nsg] se usa para permitir o denegar el tráfico de red a la subred. Puede asociar un NSG a una NIC individual o a una subred. Si se asocia con una subred, las reglas NSG se aplican a todas las máquinas virtuales de esa subred.
* **Diagnóstico.** El registro de diagnóstico es fundamental para administrar y solucionar problemas de la VM.

## <a name="recommendations"></a>Recomendaciones

En esta arquitectura se muestran las recomendaciones de línea base para ejecutar una máquina virtual con Windows en Azure. No obstante, no se recomienda usar una sola máquina virtual para cargas de trabajo críticas, ya que se crea un único punto de error. Para una mayor disponibilidad, implemente varias máquinas virtuales en un [conjunto de disponibilidad][availability-set]. Para más información, consulte el artículo sobre la [ejecución de varias máquinas virtuales en Azure][multi-vm]. 

### <a name="vm-recommendations"></a>Recomendaciones de VM

Azure ofrece muchos tamaños de máquinas virtuales, pero se recomiendan las series DS y GS, ya que estos tamaños admiten [Premium Storage][premium-storage]. Seleccione uno de estos tamaños de máquina a menos que tenga una carga de trabajo especializada, como puede ser el caso de la informática de alto rendimiento. Para más información, consulte el artículo sobre [tamaños de máquinas virtuales][virtual-machine-sizes]. 

Si desplaza una carga de trabajo existente a Azure, comience con el tamaño de máquina virtual que más se parezca a los servidores locales. Luego, mida el rendimiento de la carga de trabajo real con respecto a la CPU, la memoria y las operaciones de entrada/salida por segundo (IOPS) de disco, y ajuste el tamaño, si es necesario. Si necesita varias NIC para la máquina virtual, tenga en cuenta que su número máximo es una función del [tamaño de máquinas virtuales][vm-size-tables].   

Cuando aprovisiona la máquina virtual y otros recursos, debe especificar una región. Por lo general, se recomienda elegir una región más cercana a los usuarios internos o clientes. Sin embargo, no todos los tamaños de máquina virtual están disponibles en todas las regiones. Para más información, consulte los [servicios por región][services-by-region]. Para ver una lista de los tamaños de máquina virtual disponibles en una región específica, ejecute el siguiente comando de la interfaz de la línea de comandos (CLI) de Azure:

```
az vm list-sizes --location <location>
```

Para más información sobre cómo elegir una imagen de máquina virtual publicada, consulte [Navegación y selección de las imágenes de máquina virtual Windows en Azure con Powershell o CLI][select-vm-image].

Habilite la supervisión y el diagnóstico, como las métricas básicas de estado, los registros de infraestructura de diagnóstico y los [diagnósticos de arranque][boot-diagnostics]. Los diagnósticos de arranque pueden ayudarle a diagnosticar errores de arranque si la máquina virtual entra en un estado de imposibilidad de arranque. Para más información, consulte [Habilitación de supervisión y diagnóstico][enable-monitoring].  

### <a name="disk-and-storage-recommendations"></a>Recomendaciones de discos y almacenamiento

Para un mejor rendimiento de la E/S de disco, se recomienda [Premium Storage][premium-storage], que almacena los datos en unidades de estado sólido (SSD). El costo se basa en el tamaño del disco aprovisionado. Las E/S por segundo y el rendimiento también dependen del tamaño del disco, por lo que al aprovisionar un disco, debería tener en cuenta los tres factores (capacidad, E/S por segundo y rendimiento). 

También se recomienda usar [Managed Disks](/azure/storage/storage-managed-disks-overview). Managed Disks no requiere una cuenta de almacenamiento. Solo debe especificar el tamaño y el tipo de disco y se implementa en un modo de alta disponibilidad.

Si no usa Managed Disks, cree cuentas de almacenamiento de Azure distintas para cada máquina virtual para almacenar los discos duros virtuales (VHD) con el fin de evitar alcanzar los límites de IOPS para cuentas de almacenamiento. 

Agregue uno o más discos de datos. Cuando se crea un nuevo disco duro virtual, no tiene formato. Inicie sesión en la VM para dar formato al disco. Si no usa Managed Disks y tiene un gran número de discos de datos, tenga en cuenta los límites de E/S totales de la cuenta de almacenamiento. Para más información, consulte [Límites de discos de máquinas virtuales][vm-disk-limits].

Cuando sea posible, instale las aplicaciones en un disco de datos, no en el disco del sistema operativo. Sin embargo, en el caso de algunas aplicaciones heredadas, podría ser necesario instalar componentes en la unidad C:. En ese caso, puede [cambiar el tamaño de la unidad del sistema operativo][resize-os-disk] con PowerShell.

Para obtener el mejor rendimiento, cree una cuenta de almacenamiento independiente para contener los registros de diagnóstico. Una cuenta de almacenamiento con redundancia local (LRS) estándar es suficiente para este tipo de registros.

### <a name="network-recommendations"></a>Recomendaciones de red

La dirección IP pública puede ser dinámica o estática. El valor predeterminado es dinámica.

* Reserve una [dirección IP estática][static-ip] si necesita una dirección IP fija que no cambie; por ejemplo, si tiene que crear un registro D en DNS o necesita que la dirección IP se agregue a una lista segura.
* También puede crear un nombre de dominio completo (FQDN) para la dirección IP. Después, puede registrar un [registro CNAME][cname-record] en DNS que apunte al nombre de dominio completo. Para más información, consulte [Crear un nombre de dominio completo en Azure Portal][fqdn].

Todos los NSG contienen un conjunto de [reglas predeterminadas][nsg-default-rules], incluida una que bloquea todo el tráfico de entrada de Internet. No se puede eliminar las reglas predeterminadas, pero otras reglas pueden reemplazarlas. Para permitir el tráfico de Internet, cree reglas que permitan el tráfico entrante a puertos específicos; por ejemplo, el puerto 80 para HTTP.  

Para habilitar RDP, agregue una regla de NSG que permita el tráfico entrante al puerto TCP 3389.

## <a name="scalability-considerations"></a>Consideraciones sobre escalabilidad

Puede escalar o reducir verticalmente una máquina virtual [cambiando su tamaño][vm-resize]. Para escalar horizontalmente, coloque dos o más máquinas virtuales detrás de un equilibrador de carga. Para más detalles, consulte [Running multiple VMs on Azure for scalability and availability][multi-vm] (Ejecución de varias máquinas virtuales en Azure para escalabilidad y disponibilidad).

## <a name="availability-considerations"></a>Consideraciones sobre disponibilidad

Para una mayor disponibilidad, implemente varias máquinas virtuales en un conjunto de disponibilidad. Este procedimiento también ofrece un [Acuerdo de Nivel de Servicio][vm-sla] (SLA) superior. 

La máquina virtual puede verse afectada por un [mantenimiento planeado][planned-maintenance] o un [mantenimiento no planeado][manage-vm-availability]. Puede usar [registros de reinicio de máquina virtual][reboot-logs] para determinar si se produjo un reinicio de la máquina virtual por un mantenimiento planeado.

Los VHD se almacenan en [Azure Storage][azure-storage], que se replica para su disponibilidad y durabilidad. 

Para protegerse de la pérdida accidental de datos durante las operaciones normales (por ejemplo, debido a errores de usuario), debe implementar también copias de seguridad de un momento dado mediante [instantáneas de blobs][blob-snapshot] u otra herramienta.

## <a name="manageability-considerations"></a>Consideraciones sobre la manejabilidad

**Grupos de recursos.** Coloque los recursos estrechamente acoplados que comparten el mismo ciclo de vida en un mismo [grupo de recursos][resource-manager-overview]. Los grupos de recursos le permiten implementar y supervisar los recursos como un grupo, y acumular los costos de facturación por grupo de recursos. También se pueden eliminar recursos en conjunto, lo que resulta muy útil para implementaciones de prueba. Asigne a los recursos nombres descriptivos. De esta forma será más fácil encontrarlos y comprender su función. Consulte [Recommended Naming Conventions for Azure Resources][naming conventions] (Convenciones de nomenclatura recomendadas para los recursos de Azure).

**Detención de una máquina virtual.** Azure hace una distinción entre los estados "Detenido" y "Desasignado". Se le cobra cuando el estado de la máquina virtual se detiene, pero no cuando se desasigna la máquina virtual. En Azure Portal, con el botón **Detener**, se desasigna la máquina virtual. Si apaga desde dentro del sistema operativo mientras tiene la sesión iniciada, la VM se detiene pero *no* se desasigna, por lo que se le seguirá cobrando.

**Eliminación de una máquina virtual.** Si elimina una VM, no se eliminarán los discos duros virtuales. Esto significa que puede eliminar de forma segura la VM sin perder datos. Sin embargo, se le seguirá cobrando por el almacenamiento. Para eliminar el VHD, elimine el archivo de [Blob Storage][blob-storage].

Para evitar eliminaciones por error, use un [bloqueo de recurso][resource-lock] para bloquear el grupo de recursos completo o recursos individuales, como la máquina virtual. 

## <a name="security-considerations"></a>Consideraciones sobre la seguridad

Use [Azure Security Center][security-center] para obtener una visión central del estado de la seguridad de sus recursos en Azure. Security Center supervisa los posibles problemas de seguridad y proporciona una imagen completa del estado de seguridad de su implementación. El Centro de seguridad se configura por cada suscripción de Azure. Habilite la recolección de datos de seguridad tal como se describe en [Uso del Centro de seguridad]. Una vez que habilite la recolección, el Centro de seguridad busca automáticamente las VM creadas en esa suscripción.

**Administración de revisiones.** Si está habilitada esta opción, el Centro de seguridad comprueba si faltan actualizaciones críticas y de seguridad. Use la [configuración de directiva de grupo][group-policy] de la máquina virtual para habilitar las actualizaciones automáticas del sistema.

**Antimalware.** Si está habilitada esta opción, el Centro de seguridad comprueba si está instalado software antimalware. También puede Security Center para instalar software antimalware desde el Portal de Azure.

**Operaciones.** Use el [control de acceso basado en rol][rbac] (RBAC) para controlar el acceso a los recursos de Azure que implementa. RBAC le permite asignar roles de autorización a los miembros de su equipo de DevOps. Por ejemplo, el rol de lector puede ver recursos de Azure pero no crearlos, administrarlos o eliminarlos. Algunos roles son específicos de un tipo de recurso de Azure determinado. Por ejemplo, el rol Colaborador de máquina virtual puede reiniciar o desasignar una máquina virtual, restablecer la contraseña de administrador, crear una nueva máquina virtual, etc. Otros [roles de RBAC integrados][rbac-roles] que pueden resultar útiles para esta arquitectura son, por ejemplo, el de [Usuario de DevTest Lab][rbac-devtest] y el de [Colaborador de la red][rbac-network]. Un usuario puede asignarse a varios roles, y es posible crear roles personalizados para una especificación aún más detallada de los permisos.

> [!NOTE]
> RBAC no limita las acciones que puede realizar un usuario que ha iniciado sesión en una máquina virtual. Esos permisos están determinados por el tipo de cuenta en el sistema operativo invitado.   

Use los [registros de auditoría][audit-logs] para ver las acciones de aprovisionamiento y otros eventos de la máquina virtual.

**Cifrado de datos.** Considere la posibilidad de usar [Azure Disk Encryption][disk-encryption] si necesita cifrar los discos de datos y del sistema operativo. 

## <a name="deploy-the-solution"></a>Implementación de la solución

Hay disponible una implementación de esta arquitectura en [GitHub][github-folder]. Implementa lo siguiente:

  * Una red virtual con una sola subred denominada **web** utilizada para hospedar la máquina virtual.
  * Un NSG con dos reglas de entrada para permitir el tráfico RDP y HTTP a la máquina virtual.
  * Una VM en la que se ejecuta la última versión de Windows Server 2016 Datacenter Edition.
  * Una extensión de script personalizada de ejemplo que da formato a los dos discos de datos y un script DSC de PowerShell que implementa IIS.

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

1. Navegue hasta la carpeta `virtual-machines\single-vm\parameters\windows` del repositorio que descargó en el paso de requisitos previos anterior.

2. Abra el archivo `single-vm-v2.json`, escriba un nombre de usuario y la clave SSH entre comillas, tal y como se muestra a continuación, y después guarde el archivo.

  ```bash
  "adminUsername": "",
  "adminPassword": "",
  ```

3. Ejecute `azbb` para implementar la máquina virtual de ejemplo, tal y como se muestra a continuación.

  ```bash
  azbb -s <subscription_id> -g <resource_group_name> -l <location> -p single-vm-v2.json --deploy
  ```

Para más información sobre la implementación de esta arquitectura de referencia de ejemplo, visite el [repositorio de GitHub][git].

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre [Azure Building Blocks][azbbv2].
- Implemente [varias máquinas virtuales][multi-vm] en Azure.

<!-- links -->
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azbbv2]: https://github.com/mspnp/template-building-blocks
[audit-logs]: https://azure.microsoft.com/blog/analyze-azure-audit-logs-in-powerbi-more/
[availability-set]: /azure/virtual-machines/virtual-machines-windows-create-availability-set
[azure-storage]: /azure/storage/storage-introduction
[blob-snapshot]: /azure/storage/storage-blob-snapshots
[blob-storage]: /azure/storage/storage-introduction
[boot-diagnostics]: https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/
[cname-record]: https://en.wikipedia.org/wiki/CNAME_record
[data-disk]: /azure/virtual-machines/virtual-machines-windows-about-disks-vhds
[disk-encryption]: /azure/security/azure-security-disk-encryption
[enable-monitoring]: /azure/monitoring-and-diagnostics/insights-how-to-use-diagnostics
[fqdn]: /azure/virtual-machines/virtual-machines-windows-portal-create-fqdn
[github-folder]: http://github.com/mspnp/reference-architectures/tree/master/virtual-machines/single-vm
[group-policy]: https://technet.microsoft.com/en-us/library/dn595129.aspx
[log-collector]: https://azure.microsoft.com/blog/simplifying-virtual-machine-troubleshooting-using-azure-log-collector/
[manage-vm-availability]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[multi-vm]: multi-vm.md
[naming conventions]: ../../best-practices/naming-conventions.md
[nsg]: /azure/virtual-network/virtual-networks-nsg
[nsg-default-rules]: /azure/virtual-network/virtual-networks-nsg#default-rules
[planned-maintenance]: /azure/virtual-machines/virtual-machines-windows-planned-maintenance
[premium-storage]: /azure/storage/storage-premium-storage
[rbac]: /azure/active-directory/role-based-access-control-what-is
[rbac-roles]: /azure/active-directory/role-based-access-built-in-roles
[rbac-devtest]: /azure/active-directory/role-based-access-built-in-roles#devtest-labs-user
[rbac-network]: /azure/active-directory/role-based-access-built-in-roles#network-contributor
[reboot-logs]: https://azure.microsoft.com/blog/viewing-vm-reboot-logs/
[resize-os-disk]: /azure/virtual-machines/virtual-machines-windows-expand-os-disk
[resource-lock]: /azure/resource-group-lock-resources
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[security-center]: https://azure.microsoft.com/services/security-center/
[select-vm-image]: /azure/virtual-machines/virtual-machines-windows-cli-ps-findimage
[services-by-region]: https://azure.microsoft.com/regions/#services
[static-ip]: /azure/virtual-network/virtual-networks-reserved-public-ip
[storage-account-limits]: /azure/azure-subscription-service-limits#storage-limits
[storage-price]: https://azure.microsoft.com/pricing/details/storage/
[Uso del Centro de seguridad]: /azure/security-center/security-center-get-started#use-security-center
[virtual-machine-sizes]: /azure/virtual-machines/virtual-machines-windows-sizes
[visio-download]: https://archcenter.azureedge.net/cdn/vm-reference-architectures.vsdx
[vm-disk-limits]: /azure/azure-subscription-service-limits#virtual-machine-disk-limits
[vm-resize]: /azure/virtual-machines/virtual-machines-linux-change-vm-size
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[vm-size-tables]: /azure/virtual-machines/virtual-machines-windows-sizes#size-tables
[0]: ./images/single-vm-diagram.png "Arquitectura de una única máquina virtual Windows en Azure"
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[azure-cli-2]: /cli/azure/install-azure-cli?view=azure-cli-latest
[git]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/single-vm
