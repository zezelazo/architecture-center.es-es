---
title: Ejecución de una VM con Windows en Azure
titleSuffix: Azure Reference Architectures
description: Procedimientos recomendados para ejecutar una máquina virtual Windows en Azure.
author: telmosampaio
ms.date: 12/13/2018
ms.openlocfilehash: b874fd3958a7f5571e6b77a24b266b113af49331
ms.sourcegitcommit: 032f402482762f4e674aeebbc122ad18dfba11eb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/14/2018
ms.locfileid: "53396460"
---
# <a name="run-a-windows-virtual-machine-on-azure"></a>Ejecución de una máquina virtual Windows en Azure

El aprovisionamiento de una máquina virtual (VM) de Azure requiere componentes adicionales, además de la propia máquina virtual, que incluyen recursos de red y de almacenamiento. En este artículo se muestran procedimientos recomendados para ejecutar una máquina virtual Windows en Azure.

![Máquina virtual Windows en Azure](./images/single-vm-diagram.png)

## <a name="resource-group"></a>Grupos de recursos

Un [grupo de recursos][resource-manager-overview] es un contenedor lógico que incluye recursos de Azure relacionados. En general, los grupos de recursos se basan en su duración y quién los administra.

Coloque los recursos estrechamente asociados que comparten el mismo ciclo de vida en un mismo [grupo de recursos][resource-manager-overview]. Los grupos de recursos permiten implementar y supervisar los recursos como un grupo, y realizar un seguimiento de los costos de facturación por grupo de recursos. También se pueden eliminar recursos en conjunto, lo que resulta muy útil para implementaciones de prueba. Asigne nombres de recursos significativos para simplificar la ubicación de un recurso específico y comprender su rol. Para obtener más información, consulte las recomendaciones de [Convenciones de nomenclatura para los recursos de Azure][naming-conventions].

## <a name="virtual-machine"></a>Máquina virtual

Puede aprovisionar una máquina virtual desde una lista de imágenes publicadas, desde una imagen administrada personalizada o desde un archivo de disco duro virtual (VHD) cargado en Azure Blob Storage.

Azure ofrece numerosos tamaños diferentes de máquina virtual. Para obtener más información, consulte [Tamaños de máquinas virtuales en Azure][virtual-machine-sizes]. Si desplaza una carga de trabajo existente a Azure, comience con el tamaño de máquina virtual que más se parezca a los servidores locales. Luego, mida el rendimiento de la carga de trabajo real, en términos de CPU, memoria y operaciones de entrada/salida por segundo (IOPS) del disco, y ajuste el tamaño según sea necesario.

Por lo general, se recomienda elegir una región de Azure más cercana a los usuarios internos o clientes. No todos los tamaños de máquina virtual están disponibles en todas las regiones. Para más información, consulte [Productos disponibles por región][services-by-region]. Para obtener una lista de los tamaños de máquina virtual disponibles en una región específica, ejecute el siguiente comando en la interfaz de la línea de comandos (CLI) de Azure:

```azurecli
az vm list-sizes --location <location>
```

Para obtener información sobre cómo elegir una imagen de VM publicada, consulte [Búsqueda de imágenes de VM de Windows][select-vm-image].

## <a name="disks"></a>Discos

Para un mejor rendimiento de la E/S de disco, se recomienda [Premium Storage][premium-storage], que almacena los datos en unidades de estado sólido (SSD). El costo se basa en la capacidad del disco aprovisionado. Las E/S por segundo y el rendimiento también dependen del tamaño del disco, por lo que al aprovisionar un disco, debería tener en cuenta los tres factores (capacidad, E/S por segundo y rendimiento).

También se recomienda usar [Managed Disks][managed-disks]. Managed Disks simplifica la administración de discos y controla el almacenamiento automáticamente. Los discos administrados no requieren una cuenta de almacenamiento. Solo debe especificar el tamaño y el tipo de disco, y se implementará como un recurso de alta disponibilidad

El disco del sistema operativo es un disco duro virtual almacenado en [Azure Storage][azure-storage], por lo que se conserva incluso cuando la máquina host está inactiva. También se recomienda crear uno o varios [discos de datos][data-disk], que son discos duros virtuales persistentes que se usan para los datos de aplicación. Cuando sea posible, instale las aplicaciones en un disco de datos, no en el disco del sistema operativo. Es posible que algunas aplicaciones heredadas deban instalar componentes en la unidad C:. En ese caso, puede [cambiar el tamaño del disco del sistema operativo][resize-os-disk] mediante PowerShell.

La máquina virtual también se crea con un disco temporal (la unidad `D:` de Windows). Este disco se almacena en una unidad física del equipo host. *No* se guarda en Azure Storage y es posible que se elimine durante los reinicios y otros eventos del ciclo de vida de la máquina virtual. Use este disco solo para datos temporales, como archivos de paginación o de intercambio.

## <a name="network"></a>Red

Los componentes de red incluyen los siguientes recursos:

- **Red virtual**. Todas las máquinas virtuales se implementan en una red virtual que se puede dividir en varias subredes.

- **Interfaz de red (NIC)**. La NIC permite que la VM se comunique con la red virtual. Si necesita varias tarjetas de interfaz de red para la máquina virtual, tenga en cuenta que hay un número máximo definido para cada [tamaño de máquina virtual][vm-size-tables].

- **Dirección IP pública**. Es necesaria una dirección IP pública para comunicarse con la máquina virtual, por ejemplo, a través de Escritorio remoto (RDP). La dirección IP pública puede ser dinámica o estática. El valor predeterminado es dinámica.

- Reserve una [dirección IP estática][static-ip] si necesita una dirección IP fija que no cambie; por ejemplo, si tiene que crear un registro "A" en DNS o agregar la dirección IP a una lista segura.
- También puede crear un nombre de dominio completo (FQDN) para la dirección IP. Después, puede registrar un [registro CNAME][cname-record] en DNS que apunte al nombre de dominio completo. Para más información, consulte [Crear un nombre de dominio completo en Azure Portal][fqdn].

- **Grupo de seguridad de red (NSG)**. [Los grupos de seguridad de red][nsg] se utilizan para permitir o denegar el tráfico de red a las máquinas virtuales. Los grupos de seguridad de red se pueden asociar con subredes o con instancias de máquina virtual individuales.

Todos los NSG contienen un conjunto de [reglas predeterminadas][nsg-default-rules], incluida una que bloquea todo el tráfico de entrada de Internet. No se puede eliminar las reglas predeterminadas, pero otras reglas pueden reemplazarlas. Para permitir el tráfico de Internet, cree reglas que permitan el tráfico entrante a puertos específicos; por ejemplo, el puerto 80 para HTTP. Para habilitar RDP, agregue una regla de NSG que permita el tráfico entrante al puerto TCP 3389.

## <a name="operations"></a>Operaciones

**Diagnóstico**. Habilite la supervisión y el diagnóstico, como las métricas básicas de estado, los registros de infraestructura de diagnóstico y los [diagnósticos de arranque][boot-diagnostics]. Los diagnósticos de arranque pueden ayudarle a diagnosticar errores de arranque si la máquina virtual entra en un estado de imposibilidad de arranque. Cree una cuenta de Azure Storage para almacenar los registros. Una cuenta de almacenamiento con redundancia local (LRS) estándar es suficiente para este tipo de registros. Para más información, consulte [Habilitación de supervisión y diagnóstico][enable-monitoring].

**Disponibilidad**. La máquina virtual puede verse afectada por un [mantenimiento planeado][planned-maintenance] o un [tiempo de inactividad no planeado][manage-vm-availability]. Puede usar [registros de reinicio de máquina virtual][reboot-logs] para determinar si se produjo un reinicio de la máquina virtual por un mantenimiento planeado. Para aumentar la disponibilidad, implemente varias máquinas virtuales en un [conjunto de disponibilidad](/azure/virtual-machines/windows/manage-availability#configure-multiple-virtual-machines-in-an-availability-set-for-redundancy). Esta configuración también ofrece un [acuerdo de nivel de servicio (SLA)][vm-sla] superior.

**Copias de seguridad** Para crear una protección contra la pérdida accidental de datos, use el servicio [Azure Backup](/azure/backup/) para realizar una copia de seguridad de las máquinas virtuales en el almacenamiento con redundancia geográfica. Azure Backup proporciona copias de seguridad coherentes con la aplicación.

**Detención de una máquina virtual**. Azure hace una distinción entre los estados "Detenido" y "Desasignado". Se le cobra cuando el estado de la máquina virtual se detiene, pero no cuando se desasigna la máquina virtual. En Azure Portal, con el botón **Detener**, se desasigna la máquina virtual. Si apaga desde dentro del sistema operativo mientras tiene la sesión iniciada, la VM se detiene pero **no** se desasigna, por lo que se le seguirá cobrando.

**Eliminación de una máquina virtual**.  Si elimina una VM, no se eliminarán los discos duros virtuales. Esto significa que puede eliminar de forma segura la VM sin perder datos. Sin embargo, se le seguirá cobrando por el almacenamiento. Para eliminar el VHD, elimine el archivo de [Blob Storage][blob-storage]. Para evitar eliminaciones por error, use un [bloqueo de recurso][resource-lock] para bloquear el grupo de recursos completo o recursos individuales, como una máquina virtual.

## <a name="security-considerations"></a>Consideraciones sobre la seguridad

Use [Azure Security Center][security-center] para obtener una visión central del estado de la seguridad de sus recursos en Azure. Security Center supervisa los posibles problemas de seguridad y proporciona una imagen completa del estado de seguridad de su implementación. El Centro de seguridad se configura por cada suscripción de Azure. Habilite la colección de datos de seguridad que se describe en [Guía de inicio rápido: incorporación de su suscripción de Azure al nivel Estándar de Security Center][security-center-get-started]. Una vez que habilite la recolección, el Centro de seguridad busca automáticamente las VM creadas en esa suscripción.

**Administración de revisiones**. Si está habilitada esta opción, Security Center comprueba si falta alguna actualización crítica y de seguridad. Use la [configuración de directiva de grupo][group-policy] de la máquina virtual para habilitar las actualizaciones automáticas del sistema.

**Antimalware**.  Si está habilitada esta opción, el Centro de seguridad comprueba si está instalado software antimalware. También puede Security Center para instalar software antimalware desde el Portal de Azure.

**Control de acceso**. Use el [control de acceso basado en rol (RBAC)][rbac] para controlar el acceso a los recursos de Azure. RBAC le permite asignar roles de autorización a los miembros de su equipo de DevOps. Por ejemplo, el rol de lector puede ver recursos de Azure pero no crearlos, administrarlos o eliminarlos. Algunos permisos son específicos para un tipo de recurso de Azure. Por ejemplo, el rol Colaborador de la máquina virtual puede reiniciar o desasignar una máquina virtual, restablecer la contraseña de administrador, crear una nueva máquina virtual, etc. Otros [roles de RBAC integrados][rbac-roles] que pueden resultar útiles para esta arquitectura son, por ejemplo, el de [Usuario de DevTest Labs][rbac-devtest] y el de [Colaborador de la red][rbac-network]. 

> [!NOTE]
> RBAC no limita las acciones que puede realizar un usuario que ha iniciado sesión en una máquina virtual. Esos permisos están determinados por el tipo de cuenta en el sistema operativo invitado.

**Registros de auditoría**. Use los [registros de auditoría][audit-logs] para ver las acciones de aprovisionamiento y otros eventos de la máquina virtual.

**Cifrado de datos**. Use [Azure Disk Encryption][disk-encryption] si necesita cifrar los discos de datos y del sistema operativo.

## <a name="next-steps"></a>Pasos siguientes

- Para aprovisionar una máquina virtual Windows consulte [Tutorial: Creación y administración de máquinas virtuales Windows con Azure PowerShell](/azure/virtual-machines/windows/tutorial-manage-vm)
- Para una arquitectura de N niveles completa en las máquinas virtuales Windows, consulte [Aplicación Windows de n niveles en Azure con SQL Server](./n-tier-sql-server.md).

<!-- links -->
[audit-logs]: https://azure.microsoft.com/blog/analyze-azure-audit-logs-in-powerbi-more/
[azure-storage]: /azure/storage/storage-introduction
[blob-storage]: /azure/storage/storage-introduction
[boot-diagnostics]: https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/
[cname-record]: https://en.wikipedia.org/wiki/CNAME_record
[data-disk]: /azure/virtual-machines/virtual-machines-windows-about-disks-vhds
[disk-encryption]: /azure/security/azure-security-disk-encryption
[enable-monitoring]: /azure/monitoring-and-diagnostics/insights-how-to-use-diagnostics
[fqdn]: /azure/virtual-machines/virtual-machines-windows-portal-create-fqdn
[group-policy]: https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn595129(v=ws.11)
[manage-vm-availability]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[managed-disks]: /azure/storage/storage-managed-disks-overview
[naming-conventions]: ../../best-practices/naming-conventions.md
[nsg]: /azure/virtual-network/virtual-networks-nsg
[nsg-default-rules]: /azure/virtual-network/virtual-networks-nsg#default-rules
[planned-maintenance]: /azure/virtual-machines/virtual-machines-windows-planned-maintenance
[premium-storage]: /azure/virtual-machines/windows/premium-storage
[rbac]: /azure/active-directory/role-based-access-control-what-is
[rbac-roles]: /azure/active-directory/role-based-access-built-in-roles
[rbac-devtest]: /azure/active-directory/role-based-access-built-in-roles#devtest-labs-user
[rbac-network]: /azure/active-directory/role-based-access-built-in-roles#network-contributor
[reboot-logs]: https://azure.microsoft.com/blog/viewing-vm-reboot-logs/
[resize-os-disk]: /azure/virtual-machines/virtual-machines-windows-expand-os-disk
[resource-lock]: /azure/resource-group-lock-resources
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[security-center]: /azure/security-center/security-center-intro
[security-center-get-started]: /azure/security-center/security-center-get-started
[select-vm-image]: /azure/virtual-machines/virtual-machines-windows-cli-ps-findimage
[services-by-region]: https://azure.microsoft.com/regions/#services
[static-ip]: /azure/virtual-network/virtual-networks-reserved-public-ip
[virtual-machine-sizes]: /azure/virtual-machines/virtual-machines-windows-sizes
[visio-download]: https://archcenter.blob.core.windows.net/cdn/vm-reference-architectures.vsdx
[vm-size-tables]: /azure/virtual-machines/virtual-machines-windows-sizes
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
