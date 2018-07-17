---
title: Ejecución de una granja de servidores de SharePoint Server 2016 de alta disponibilidad en Azure
description: Procedimientos de demostrada eficacia para configurar una granja de servidores de SharePoint Server 2016 de alta disponibilidad en Azure.
author: njray
ms.date: 07/14/2018
ms.openlocfilehash: ff690300cb5f4af301bcfac58ac10b9b3c47f96d
ms.sourcegitcommit: 71cbef121c40ef36e2d6e3a088cb85c4260599b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/14/2018
ms.locfileid: "39060904"
---
# <a name="run-a-high-availability-sharepoint-server-2016-farm-in-azure"></a>Ejecución de una granja de servidores de SharePoint Server 2016 de alta disponibilidad en Azure

Esta arquitectura de referencia muestra un conjunto de procedimientos de demostrada eficacia para configurar una granja de servidores de SharePoint Server 2016 de alta disponibilidad en Azure, mediante la topología MinRole y los grupos de disponibilidad AlwaysOn de SQL Server. La granja de servidores de SharePoint se implementa en una red virtual protegida sin puntos de conexión accesibles desde Internet ni presencia en la susodicha red. [**Implementación de la solución**.](#deploy-the-solution) 

![](./images/sharepoint-ha.png)

*Descargue un [archivo Visio][visio-download] de esta arquitectura.*

## <a name="architecture"></a>Arquitectura

Esta arquitectura se basa en la que se muestra en [Run Windows VMs for an N-tier application][windows-n-tier] (Ejecución de máquinas virtuales Windows para una aplicación de n niveles). Implementa una granja de servidores de SharePoint Server 2016 con alta disponibilidad dentro de una red virtual (VNet) de Azure. Esta arquitectura es adecuada para un entorno de pruebas o producción, una infraestructura híbrida de SharePoint con Office 365 o como base para un escenario de recuperación ante desastres.

La arquitectura consta de los siguientes componentes:

- **Grupos de recursos**. Un [grupo de recursos][resource-group] es un contenedor que incluye recursos de Azure relacionados. Un grupo de recursos se usa para los servidores de SharePoint y otro grupo de recursos para los componentes de infraestructura que son independientes de las máquinas virtuales, como la red virtual y los equilibradores de carga.

- **Red virtual**. Las máquinas virtuales se implementan en una red virtual con un espacio de direcciones de intranet único. La red virtual se subdivide además en subredes. 

- **Máquinas virtuales (VM)**. Las máquinas virtuales se implementan en la red virtual y se les asignan direcciones IP estáticas privadas a todas ellas. Se recomiendan las direcciones IP estáticas para las máquinas virtuales que ejecutan SQL Server y SharePoint Server 2016, a fin de evitar problemas con el almacenamiento en caché de direcciones IP y los cambios de direcciones después de un reinicio.

- **Conjuntos de disponibilidad**. Coloque las máquinas virtuales de cada rol de SharePoint en distintos [conjuntos de disponibilidad][availability-set] y aprovisione al menos dos máquinas virtuales (VM) para cada rol. Esto hace que las máquinas virtuales sean aptas para un Acuerdo de Nivel de Servicio (SLA) mayor. 

- **Equilibrador de carga interno**. El [equilibrador de carga][load-balancer] distribuye el tráfico de solicitudes de SharePoint desde la red local hasta los servidores web front-end de la granja de servidores de SharePoint. 

- **Grupos de seguridad de red (NSG)**. Para cada subred que contiene máquinas virtuales, se crea un [grupo de seguridad de red][nsg]. Use NSG para restringir el tráfico de red dentro de la red virtual, con el fin de aislar las subredes. 

- **Puerta de enlace**. La puerta de enlace proporciona una conexión entre la red local y la red virtual de Azure. La conexión puede usar ExpressRoute o VPN de sitio a sitio. Para más información, consulte [Connect an on-premises network to Azure][hybrid-ra] (Conexión de una red local a Azure).

- **Controladores de dominio de Windows Server Active Directory (AD)**. Esta arquitectura de referencia implementa controladores de dominio de Windows Server AD. Estos controladores de dominio se ejecutan en la red virtual de Azure y tienen una relación de confianza con el bosque de Windows Server AD local. Las solicitudes web de cliente de los recursos de la granja de servidores de SharePoint se autentican en la red virtual en lugar de enviarse ese tráfico de autenticación a la red local a través de la conexión de puerta de enlace. En DNS, se crean registros A o CNAME de intranet para que los usuarios de intranet puedan resolver el nombre de la granja de servidores de SharePoint en la dirección IP privada del equilibrador de carga interno.

  SharePoint Server 2016 también admite el uso de [Azure Active Directory Domain Services](/azure/active-directory-domain-services/). Azure AD Domain Services proporciona servicios de dominio administrado, por lo que no necesitará implementar y administrar controladores de dominio en Azure.

- **Grupo de disponibilidad AlwaysOn de SQL Server** Para lograr una alta disponibilidad de la base de datos de SQL Server, se recomiendan los [grupos de disponibilidad AlwaysOn de SQL Server][sql-always-on]. Se usan dos máquinas virtuales para SQL Server. Una contiene la réplica de base de datos principal y la otra la réplica secundaria. 

- **Máquina virtual de mayoría de nodos**. Esta máquina virtual permite que el clúster de conmutación por error establezca el cuórum. Para más información, consulte [Descripción de las configuraciones de cuórum en un clúster de conmutación por error][sql-quorum].

- **Servidores de SharePoint**. Los servidores de SharePoint desempeñan funciones de front-end web, almacenamiento en caché, aplicación y búsqueda. 

- **JumpBox**. También se le conoce como [bastion host][bastion-host]. Se trata de una máquina virtual segura en la red que usan los administradores para conectarse al resto de máquinas virtuales. El JumpBox tiene un NSG que solo permite el tráfico remoto que procede de direcciones IP públicas de una lista segura. El NSG debe permitir el tráfico de escritorio remoto (RDP).

## <a name="recommendations"></a>Recomendaciones

Los requisitos pueden diferir de los de la arquitectura que se describe aquí. Use estas recomendaciones como punto de partida.

### <a name="resource-group-recommendations"></a>Recomendaciones para grupos de recursos

Se recomienda separar los grupos de recursos de acuerdo con el rol del servidor y tener un grupo de recursos independiente para los componentes de infraestructura que sean recursos globales. En esta arquitectura, los recursos de SharePoint forman un grupo, mientras que SQL Server y otros recursos de utilidad forman otro.

### <a name="virtual-network-and-subnet-recommendations"></a>Recomendaciones para red virtual y subred

Use una subred para cada rol de SharePoint, más una subred para la puerta de enlace y otra para el JumpBox. 

La subred de puerta de enlace debe tener el nombre *GatewaySubnet*. Asigne el espacio de direcciones de subred de puerta de enlace desde la última parte del espacio de direcciones de red virtual. Para más información, consulte la sección [Connect an on-premises network to Azure using a VPN gateway][hybrid-vpn-ra] (Conexión de una red local a Azure mediante una puerta de enlace de VPN).

### <a name="vm-recommendations"></a>Recomendaciones de VM

En función de los tamaños de máquina virtual DSv2 estándar, esta arquitectura requiere como mínimo 38 núcleos:

- 8 servidores de SharePoint en Standard_DS3_v2 (4 núcleos cada uno) = 32 núcleos
- 2 controladores de dominio de Active Directory en Standard_DS1_v2 (1 núcleo cada uno) = 2 núcleos
- 2 máquinas virtuales de SQL Server en Standard_DS1_v2 = 2 núcleos
- 1 mayoría de nodos en Standard_DS1_v2 = 1 núcleo
- 1 servidor de administración en Standard_DS1_v2 = 1 núcleo

El número total de núcleos dependerá de los tamaños de máquina virtual que seleccione. Para más información, consulte [Recomendaciones para el servidor de SharePoint](#sharepoint-server-recommendations) a continuación.

Asegúrese de que su suscripción de Azure tenga suficiente cuota de núcleos de máquina virtual para la implementación, o esta no se realizará correctamente. Consulte [Límites, cuotas y restricciones de suscripción y servicios de Azure][quotas]. 
 
### <a name="nsg-recommendations"></a>Recomendaciones para NSG

Se recomienda tener un NSG para cada subred que contenga máquinas virtuales, para permitir el aislamiento de la subred. Si quiere configurar el aislamiento de la subred, agregue reglas NSG que definan el tráfico entrante o saliente permitido o denegado para cada subred. Para más información, consulte [Filtrado del tráfico de red con grupos de seguridad de red][virtual-networks-nsg]. 

No asigne un NSG a la subred de puerta de enlace o la puerta de enlace dejará de funcionar. 

### <a name="storage-recommendations"></a>Recomendaciones para almacenamiento

La configuración del almacenamiento de las máquinas virtuales de la granja de servidores debe coincidir con los procedimientos recomendados usados en las implementaciones locales. Los servidores de SharePoint deben tener un disco aparte para los registros. Los servidores de SharePoint que hospedan roles de índice de búsqueda requieren espacio en disco adicional para que se almacene el índice de búsqueda. En SQL Server, el procedimiento habitual consiste en separar datos y registros. Agregue más discos para el almacenamiento de copia de seguridad de base de datos y use un disco independiente para [tempdb][tempdb].

Para lograr la máxima confiabilidad, se recomienda usar [Azure Managed Disks][managed-disks]. Los discos administrados garantizan que los discos de las máquinas virtuales de un conjunto de disponibilidad estén aislados, con el fin de evitar únicos puntos de error. 

> [!NOTE]
> Actualmente la plantilla de Resource Manager para esta arquitectura de referencia no usa discos administrados, pero está previsto actualizarla para que los use.

Use discos administrados Premium en todas las máquinas virtuales de SQL Server y SharePoint. Puede usar discos administrados Estándar para el servidor de mayoría de nodos, los controladores de dominio y el servidor de administración. 

### <a name="sharepoint-server-recommendations"></a>Recomendaciones para el servidor de SharePoint

Antes de configurar la granja de servidores de SharePoint, asegúrese de que tiene una cuenta de servicio de Windows Server Active Directory por cada servicio. Para esta arquitectura, se necesita como mínimo las siguientes cuentas de nivel de dominio para aislar los privilegios por rol:

- Cuenta de servicio de SQL Server
- Cuenta de usuario de instalación
- Cuenta de granja de servidores
- Cuenta de Search Service
- Cuenta de acceso al contenido
- Cuentas de grupo de aplicaciones web
- Cuentas de grupo de aplicaciones de servicio
- Cuenta de superusuario de caché
- Cuenta de lector avanzado de caché

Para todos los roles, excepto indexador de búsqueda, se recomienda usar el tamaño de máquina virtual [Standard_DS3_v2][vm-sizes-general]. El indexador de búsqueda debe tener como mínimo el tamaño [Standard_DS13_v2][vm-sizes-memory]. 

> [!NOTE]
> La plantilla de Resource Manager de esta arquitectura de referencia usa el tamaño DS3 más pequeño para el indexador de búsqueda, con el fin de probar la implementación. Si la implementación es para producción, use el tamaño DS13 o mayor. 

Si las cargas de trabajo son de producción, consulte [Requisitos de hardware y software para SharePoint Server 2016][sharepoint-reqs]. 

Para satisfacer el requisito de compatibilidad con el rendimiento del disco de 200 MB por segundo como mínimo, asegúrese de planear la arquitectura de búsqueda. Consulte [Planear la arquitectura de búsqueda empresarial en SharePoint Server 2013][sharepoint-search]. Además, siga las directrices descritas en [Best practices for crawling in SharePoint Server 2016][sharepoint-crawling] (Procedimientos recomendados para rastreo en SharePoint Server 2016).

Asimismo, almacene los datos del componente de búsqueda en un volumen de almacenamiento independiente o en una partición con alto rendimiento. Para reducir la carga y mejorar el rendimiento, configure las cuentas de usuario de caché de objeto, que son necesarias en esta arquitectura. Divida los archivos del sistema operativo Windows Server, los archivos de programa de SharePoint Server 2016 y los registros de diagnóstico en tres volúmenes de almacenamiento independientes o particiones con un rendimiento normal. 

Para más información sobre estas recomendaciones, consulte [Cuentas de servicio y administrativas de implementación inicial en SharePoint Server 2016][sharepoint-accounts].

### <a name="hybrid-workloads"></a>Cargas de trabajo híbridas

Esta arquitectura de referencia implementa una granja de servidores de SharePoint Server 2016 que se puede usar como un [entorno de SharePoint híbrido][ sharepoint-hybrid] &mdash; es decir, que extiende SharePoint Server 2016 a Office 365 SharePoint Online. Si tiene Office Online Server, consulte [Compatibilidad de Office Web Apps y Office Online Server en Azure][office-web-apps].

Las aplicaciones de servicio predeterminadas de esta implementación están diseñadas para admitir cargas de trabajo híbridas. Todas las cargas de trabajo híbridas de SharePoint Server 2016 y Office 365 se pueden implementar en esta granja de servidores sin cambios en la infraestructura de SharePoint, con una excepción: la aplicación híbrida de nube de Search Service no se debe implementar en servidores que hospeden una topología de búsqueda existente. Por lo tanto, es necesario agregar a la granja una o varias máquinas virtuales basadas en roles de búsqueda para admitir este escenario híbrido.

### <a name="sql-server-always-on-availability-groups"></a>Grupos de disponibilidad AlwaysOn de SQL Server

En esta arquitectura se usan máquinas virtuales de SQL Server porque SharePoint Server 2016 no puede usar Azure SQL Database. Para lograr alta disponibilidad en SQL Server, se recomienda usar grupos de disponibilidad AlwaysOn, que especifican un conjunto de bases de datos que conmutan por error juntas, lo que hace que tengan una alta capacidad de disponibilidad y recuperación. En esta arquitectura de referencia, las bases de datos se crean durante la implementación, pero es necesario habilitar manualmente los grupos de disponibilidad AlwaysOn y agregar las bases de datos de SharePoint a uno de estos grupos. Para más información, consulte [Crear el grupo de disponibilidad y agregar las bases de datos de SharePoint][create-availability-group].

También se recomienda agregar una dirección IP de escucha al clúster, que es la dirección IP privada del equilibrador de carga interno para las máquinas virtuales de SQL Server.

Para conocer los tamaños de máquina virtual recomendados y otras recomendaciones sobre rendimiento para SQL Server en Azure, consulte [Procedimientos recomendados para SQL Server en Azure Virtual Machines][sql-performance]. Además, siga las recomendaciones de [Procedimientos recomendados para SQL Server en una granja de servidores de SharePoint 2016][sql-sharepoint-best-practices].

Se recomienda que el servidor de mayoría de nodos resida en un equipo independiente de los asociados de replicación. El servidor permite que el servidor asociado de replicación secundaria en una sesión en modo de alta seguridad reconozca si debe iniciar una conmutación por error automática. A diferencia de los dos asociados, los servidores de mayoría de nodos no atienden la base de datos, sino que admiten la conmutación por error automática. 

## <a name="scalability-considerations"></a>Consideraciones sobre escalabilidad

Para escalar verticalmente los servidores existentes, simplemente cambie el tamaño de la máquina virtual. 

Con la funcionalidad [MinRoles][minroles] de SharePoint Server 2016, puede escalar horizontalmente los servidores según el rol del servidor y también quitar servidores de un rol. Cuando agregue servidores a un rol, puede especificar cualquiera de los roles únicos o uno de los roles combinados. Sin embargo, si agrega servidores al rol de búsqueda, también debe reconfigurar la topología de búsqueda mediante PowerShell. Además, puede convertir roles mediante MinRoles. Para más información, consulte [Administración de una granja de servidores de MinRole en SharePoint Server 2016][sharepoint-minrole].

Tenga en cuenta que SharePoint Server 2016 no admite el uso de conjuntos de escalado de máquinas virtuales para el escalado automático.

## <a name="availability-considerations"></a>Consideraciones sobre disponibilidad

Esta arquitectura de referencia admite alta disponibilidad dentro de una región de Azure, porque cada rol tiene al menos dos máquinas virtuales implementadas en un conjunto de disponibilidad.

Para protegerse frente a un error regional, cree una granja de servidores de recuperación ante desastres independiente en una región distinta de Azure. Los objetivos de tiempo de recuperación (RTO) y los objetivos de punto de recuperación (RPO) determinarán los requisitos de instalación. Para más información, consulte [Seleccionar una estrategia de recuperación ante desastres para SharePoint 2016][sharepoint-dr]. La región secundaria debe ser una *región emparejada* con la región primaria. En el caso de una interrupción amplia, tiene prioridad la recuperación de una región de cada pareja. Para más información, consulte [Continuidad empresarial y recuperación ante desastres (BCDR): regiones emparejadas de Azure][paired-regions].

## <a name="manageability-considerations"></a>Consideraciones sobre la manejabilidad

Para operar y mantener servidores, granjas de servidores y sitios, siga los procedimientos recomendados para las operaciones de SharePoint. Para más información, consulte [Operaciones para SharePoint Server 2016][sharepoint-ops].

Las tareas que debe tener en cuenta al administrar SQL Server en un entorno de SharePoint pueden diferir de aquellas que se suelen considerar para una aplicación de base de datos. Un procedimiento recomendado es realizar una copia de seguridad semanal completa de todas las bases de datos SQL con copias de seguridad incrementales durante la noche. Realice copia de seguridad de los registros de transacciones cada 15 minutos. Otro procedimiento consiste en implementar tareas de mantenimiento de SQL Server en las bases de datos y deshabilitar las integradas de SharePoint. Para más información, consulte [Almacenamiento y configuración y planeamiento de capacidad de SQL Server][sql-server-capacity-planning]. 

## <a name="security-considerations"></a>Consideraciones sobre la seguridad

Las cuentas de servicio de nivel de dominio usadas para ejecutar SharePoint Server 2016 requieren controladores de dominio de Windows Server AD para los procesos de autenticación y unión a dominios. Azure Active Directory Domain Services no se puede usar con esta finalidad. Para ampliar la infraestructura de identidad de Windows Server AD ya implantada en la intranet, esta arquitectura emplea dos controladores de dominio de réplica de Windows Server AD de un bosque existente de Windows Server AD local.

Además, siempre es conveniente planear la protección de seguridad. Otras recomendaciones son:

- Agregar reglas a los NSG para aislar las subredes y los roles.
- No asignar direcciones IP públicas a las máquinas virtuales.
- Para la detección de intrusiones y el análisis de cargas útiles, considere la posibilidad de usar una aplicación virtual de red delante de los servidores web front-end en lugar de un equilibrador de carga interno de Azure.
- Como opción, use directivas IPsec para el cifrado del tráfico de texto no cifrado entre servidores. Si también va a realizar el aislamiento de la subred, actualice las reglas del grupo de seguridad de red para permitir el tráfico IPsec.
- Instalar agentes antimalware para las máquinas virtuales.

## <a name="deploy-the-solution"></a>Implementación de la solución

Hay disponible una implementación de esta arquitectura de referencia en [GitHub][github]. La implementación puede tardar varias horas en completarse.

La implementación crea los siguientes grupos de recursos en su suscripción:

- ra-onprem-sp2016-rg
- ra-sp2016-network-rg

Los archivos de parámetro de plantilla hacen referencia a estos nombres, por lo que si se cambian es necesario actualizar los archivos de parámetro para que coincidan. 

Los archivos de parámetros incluyen una contraseña codificada de forma rígida en varios lugares. Cambie estos valores antes de realizar la implementación.

### <a name="prerequisites"></a>requisitos previos

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-the-solution"></a>Implementación de la solución 

1. Ejecute el siguiente comando para implementar una red local simulada.

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p onprem.json --deploy
    ```

2. Ejecute el siguiente comando para implementar la red virtual de Azure y la puerta de enlace de VPN.

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p connections.json --deploy
    ```

3. Ejecute el siguiente comando para implementar Jumpbox, los controladores de dominio de AD y las máquinas virtuales de SQL Server.

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p azure1.json --deploy
    ```

4. Ejecute el siguiente comando para crear el clúster de conmutación por error y el grupo de disponibilidad. 

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p azure2-cluster.json --deploy

5. Run the following command to deploy the remaining VMs.

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p azure3.json --deploy
    ```

En este punto, compruebe que puede realizar una conexión TCP desde el front-end web al equilibrador de carga de los Grupos de disponibilidad AlwaysOn de SQL Server. Para ello, siga los pasos que se indican a continuación:

1. Use Azure Portal para encontrar la máquina virtual denominada `ra-sp-jb-vm1` en el grupo de recursos `ra-sp2016-network-rg`. Esta es la máquina virtual de JumpBox.

2. Haga clic en `Connect` para abrir una sesión de escritorio remoto en la máquina virtual. Usar la contraseña que especificó en el `azure1.json` archivo de parámetros.

3. En la sesión de escritorio remoto, inicie sesión en 10.0.5.4. Esta es la dirección IP de la máquina virtual denominada `ra-sp-app-vm1`.

4. Abra una consola de PowerShell en la máquina virtual y utilice el cmdlet `Test-NetConnection` para comprobar que puede conectarse al equilibrador de carga.

    ```powershell
    Test-NetConnection 10.0.3.100 -Port 1433
    ```

La salida debe tener una apariencia similar a la siguiente:

```powershell
ComputerName     : 10.0.3.100
RemoteAddress    : 10.0.3.100
RemotePort       : 1433
InterfaceAlias   : Ethernet 3
SourceAddress    : 10.0.0.132
TcpTestSucceeded : True
```

Si se produce un error, use Azure Portal para reiniciar la máquina virtual denominada `ra-sp-sql-vm2`. Después de que se reinicia la máquina virtual, ejecute el comando `Test-NetConnection` de nuevo. Puede que tenga que esperar alrededor de un minuto después de reiniciar la máquina virtual para que la conexión se realice correctamente. 

Ahora, finalice la implementación como se indica a continuación.

1. Ejecute el siguiente comando para implementar el nodo principal de la granja de SharePoint.

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p azure4-sharepoint-server.json --deploy
    ```

2. Ejecute el siguiente comando para implementar la memoria caché, la búsqueda y la web de SharePoint.

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p azure5-sharepoint-farm.json --deploy
    ```

3. Ejecute el siguiente comando para crear las reglas del grupo de seguridad de red.

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p azure6-security.json --deploy
    ```

### <a name="validate-the-deployment"></a>Validación de la implementación

1. En [Azure Portal][azure-portal], vaya al grupo de recursos `ra-onprem-sp2016-rg`.

2. En la lista de recursos, seleccione el recurso de máquina virtual denominado `ra-onpr-u-vm1`. 

3. Conéctese a la máquina virtual como se describe en [Conexión a la máquina virtual][connect-to-vm]. El nombre de usuario es `\onpremuser`.

5.  Cuando se establezca la conexión remota a la máquina virtual, abra un explorador en la máquina virtual y vaya a `http://portal.contoso.local`.

6.  En el cuadro **Seguridad de Windows**, inicie sesión en el portal de SharePoint con `contoso.local\testuser` como nombre de usuario.

Este inicio de sesión crea un túnel desde el dominio de Fabrikam.com que usa la red local hasta el dominio de contoso.local que usa el portal de SharePoint. Cuando se abra el sitio de SharePoint, verá el sitio de demostración raíz.

**_Personas que han contribuido a esta arquitectura de referencia_**  &mdash; Joe Davies, Bob Fox, Neil Hodgkinson, Paul Stork

<!-- links -->

[availability-set]: /azure/virtual-machines/windows/manage-availability
[azure-portal]: https://portal.azure.com
[azure-ps]: /powershell/azure/overview
[azure-pricing]: https://azure.microsoft.com/pricing/calculator/
[bastion-host]: https://en.wikipedia.org/wiki/Bastion_host
[create-availability-group]: https://technet.microsoft.com/library/mt793548(v=office.16).aspx
[connect-to-vm]: /azure/virtual-machines/windows/quick-create-portal#connect-to-virtual-machine
[github]: https://github.com/mspnp/reference-architectures
[hybrid-ra]: ../hybrid-networking/index.md
[hybrid-vpn-ra]: ../hybrid-networking/vpn.md
[load-balancer]: /azure/load-balancer/load-balancer-internal-overview
[managed-disks]: /azure/storage/storage-managed-disks-overview
[minroles]: https://technet.microsoft.com/library/mt346114(v=office.16).aspx
[nsg]: /azure/virtual-network/virtual-networks-nsg
[office-web-apps]: https://support.microsoft.com/help/3199955/office-web-apps-and-office-online-server-supportability-in-azure
[paired-regions]: /azure/best-practices-availability-paired-regions
[readme]: https://github.com/mspnp/reference-architectures/tree/master/sharepoint/sharepoint-2016
[resource-group]: /azure/azure-resource-manager/resource-group-overview
[quotas]: /azure/azure-subscription-service-limits
[sharepoint-accounts]: https://technet.microsoft.com/library/ee662513(v=office.16).aspx
[sharepoint-crawling]: https://technet.microsoft.com/library/dn535606(v=office.16).aspx
[sharepoint-dr]: https://technet.microsoft.com/library/ff628971(v=office.16).aspx
[sharepoint-hybrid]: https://aka.ms/sphybrid
[sharepoint-minrole]: https://technet.microsoft.com/library/mt743705(v=office.16).aspx
[sharepoint-ops]: https://technet.microsoft.com/library/cc262289(v=office.16).aspx
[sharepoint-reqs]: https://technet.microsoft.com/library/cc262485(v=office.16).aspx
[sharepoint-search]: https://technet.microsoft.com/library/dn342836.aspx
[sql-always-on]: /sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server
[sql-performance]: /virtual-machines/windows/sql/virtual-machines-windows-sql-performance
[sql-server-capacity-planning]: https://technet.microsoft.com/library/cc298801(v=office.16).aspx
[sql-quorum]: https://technet.microsoft.com/library/cc731739(v=ws.11).aspx
[sql-sharepoint-best-practices]: https://technet.microsoft.com/library/hh292622(v=office.16).aspx
[tempdb]: /sql/relational-databases/databases/tempdb-database
[virtual-networks-nsg]: /azure/virtual-network/virtual-networks-nsg
[visio-download]: https://archcenter.blob.core.windows.net/cdn/Sharepoint-2016.vsdx
[vm-sizes-general]: /azure/virtual-machines/windows/sizes-general
[vm-sizes-memory]: /azure/virtual-machines/windows/sizes-memory
[windows-n-tier]: ../virtual-machines-windows/n-tier.md
