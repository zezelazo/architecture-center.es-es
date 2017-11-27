---
title: "Implementación de SAP NetWeaver y SAP HANA en Azure"
description: "Prácticas probadas para ejecutar SAP HANA en un entorno de alta disponibilidad en Azure."
author: njray
ms.date: 06/29/2017
ms.openlocfilehash: 27a97103c0c6f305cb8e830d670c8d0ba7e22aa5
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2017
---
# <a name="deploy-sap-netweaver-and-sap-hana-on-azure"></a>Implementación de SAP NetWeaver y SAP HANA en Azure

Esta arquitectura de referencia muestra un conjunto de prácticas probadas para ejecutar SAP HANA en un entorno de alta disponibilidad en Azure. [**Implemente esta solución**.](#deploy-the-solution)

![0][0]

*Descargue un [archivo Visio][visio-download] de esta arquitectura.*

> [!NOTE]
> La implementación de esta arquitectura de referencia requiere licencias adecuadas de los productos de SAP y de otras tecnologías que no son de Microsoft. Para obtener información acerca de la asociación entre Microsoft y SAP, consulte [SAP HANA en Azure][sap-hana-on-azure].

## <a name="architecture"></a>Arquitectura

La arquitectura consta de los siguientes componentes.

- **Red virtual**. Una red virtual es una representación de una red aislada lógicamente en Azure. Todas las máquinas virtuales de esta arquitectura de referencia se implementan en la misma red virtual. La red virtual se subdivide en subredes. Cree una subred independiente para cada nivel, lo que incluye aplicación (SAP NetWeaver), base de datos (SAP HANA), administración (el Jumpbox) y Active Directory.

- **Máquinas virtuales (VM)**. Las máquinas virtuales de esta arquitectura se agrupan en varios niveles distintos.

    - **SAP NetWeaver**. Incluye SAP ASCS, SAP Web Dispatcher y los servidores de aplicaciones SAP. 
    
    - **SAP HANA**. Esta arquitectura de referencia usa SAP HANA para el nivel de base de datos, que se ejecuta en una sola instancia de [GS5][vm-sizes-mem]. SAP HANA está certificado para cargas de trabajo de OLAP de producción en GS5 o [SAP HANA en instancias grandes de Azure][azure-large-instances]. Esta arquitectura de referencia es para máquinas virtuales de Azure de las series G y M. Para obtener información acerca de SAP HANA en instancias grandes de Azure, consulte [Introducción y arquitectura de SAP HANA en Azure (instancias grandes)][azure-large-instances].
   
    - **Jumpbox**. También se denomina bastion host. Se trata de una máquina virtual segura en la red que los administradores utilizan para conectarse a las restantes máquinas virtuales. 
     
    - **Controladores de dominio de Windows Server Active Directory (AD)** Los controladores de dominio se utilizan para configurar el clúster de conmutación por error de Windows Server (véase a continuación).
 
- **Conjuntos de disponibilidad**. Coloque las máquinas virtuales para los roles de SAP ACSC, el servidor de aplicaciones de SAP y SAP Web Dispatcher en conjuntos de disponibilidad independientes y aprovisione al menos dos máquinas virtuales para cada rol. Esto hace que las máquinas virtuales sean aptas para un Acuerdo de Nivel de Servicio (SLA) mayor.
    
- **NIC**. Las máquinas virtuales que ejecutan SAP NetWeaver y SAP HANA requieren dos interfaces de red (NIC). Cada NIC se asigna a una subred diferente, con el fin de separar los distintos tipos de tráfico. Para más información, consulte la sección [Recomendaciones](#recommendations) a continuación.

- **Clúster de conmutación por error de Windows Server**. Las máquinas virtuales que ejecutan SAP ACSC están configuradas como clúster de conmutación por error para lograr una alta disponibilidad. Para admitir el clúster de conmutación por error, SIOS DataKeeper Cluster Edition realiza la función de volumen compartido de clúster (CSV) mediante la replicación de discos independientes que pertenecen a los nodos del clúster. Para más información, consulte [Running SAP applications on the Microsoft platform][running-sap] (Ejecución de aplicaciones SAP en la plataforma de Microsoft).
    
- **Equilibradores de carga.** Se usan dos instancias de [Azure Load Balancer][azure-lb]. La primera, que se muestra a la izquierda del diagrama, distribuye el tráfico a las máquinas virtuales de SAP Web Dispatcher. Esta configuración implementa la opción de distribuidor web en paralelo que se describe en [High Availability of the SAP Web Dispatcher (Alta disponibilidad de SAP Web Dispatcher)][sap-dispatcher-ha]. El segundo equilibrador de carga, que se muestra a la derecha, habilita la conmutación por error en el clúster de conmutación por error de Windows Server, para lo que dirige las conexiones entrantes al nodo activo/correcto.

- **VPN Gateway.** VPN Gateway extiende su red local a la red virtual de Azure. También puede usar ExpressRoute, que utiliza una conexión privada dedicada que no pasa por la red Internet pública. La solución de ejemplo no implementa la puerta de enlace. Para más información, consulte [Connect an on-premises network to Azure][hybrid-networking] (Conexión de una red local a Azure).

## <a name="recommendations"></a>Recomendaciones

Los requisitos pueden diferir de los de la arquitectura que se describe aquí. Use estas recomendaciones como punto de partida.

### <a name="load-balancers"></a>Equilibradores de carga

[SAP Web Dispatcher][sap-dispatcher] controla el equilibrio de carga del tráfico HTTP(S) en los servidores de doble pila (ABAP y Java). Durante años SAP ha propugnado servidores de aplicaciones de una sola pila, por lo que en la actualidad muy pocas aplicaciones se ejecutan en un modelo de implementación de pila doble. La instancia de Azure Load Balancer que se muestra en el diagrama de la arquitectura implementa el clúster de alta disponibilidad para SAP Web Dispatcher.

El equilibrio de carga del tráfico que llega a los servidores de aplicaciones se controla en SAP. En el caso del tráfico de los clientes SAPGUI que se conectan a un servidor de SAP a través de DIAG y llamadas a funciones remotas (RFC), el servidor de mensajes SCS equilibra la carga mediante la creación de [grupos de inicio de sesión][logon-groups] del servidor de aplicaciones de SAP. 

SMLG es una transacción de SAP ABAP que se usa para administrar la funcionalidad de equilibrio de carga del inicio de sesión de los servicios centrales de SAP. El grupo de back-end del grupo de inicio de sesión tiene más de un servidor de aplicaciones de ABAP. Los clientes que acceden a los servicios de clúster de ASCS se conectan a Azure Load Balancer a través de una dirección IP de front-end. El nombre de red virtual del clúster de ASCS también tiene una dirección IP. Si lo desea, esta dirección puede asociarse a una dirección IP adicional en Azure Load Balancer, con el fin de que el clúster se puede administrar de forma remota.  

### <a name="nics"></a>Tarjetas de red

Las funciones de administración del entorno de SAP requieren separación del tráfico del servidor en diferentes tarjetas de red. Por ejemplo, los datos empresariales se deben separar tanto del tráfico administrativo como del de copias de seguridad. La asignación de varias tarjetas de red a diferentes subredes permite esta separación de datos. Para más información, consulte "Red" en el documento [Building High Availability for SAP NetWeaver and SAP HANA][sap-ha] (Creación de alta disponibilidad para SAP NetWeaver y SAP HANA) (en PDF).

Asigne la tarjeta de red de administración a la subred de administración y la tarjeta de red de comunicación de datos a otra subred. Para ver los detalles de la configuración, consulte [Creación y administración de una máquina virtual Windows que tiene varias tarjetas de red][multiple-vm-nics].

### <a name="azure-storage"></a>Azure Storage

Con todas las máquinas virtuales del servidor de bases de datos, se recomienda utilizar Azure Premium Storage para que la latencia de lectura y escritura sea consistente. En el caso de los servidores de aplicaciones de SAP, incluidas las máquinas virtuales (A)SCS, puede usar Azure Standard Storage, ya que la ejecución de la aplicación tiene lugar en la memoria y solo utiliza discos para el registro.

Para lograr la máxima confiabilidad, se recomienda usar [Azure Managed Disks][managed-disks]. Los discos administrados garantizan que los discos de las máquinas virtuales de un conjunto de disponibilidad estén aislados, con el fin de evitar únicos puntos de error.

> [!NOTE]
> Actualmente la plantilla de Resource Manager para esta arquitectura de referencia no utiliza discos administrados, pero está previsto actualizarla para que los use.

Para lograr un rendimiento del ancho de banda de disco y un IOPS altos, las prácticas habituales de optimización del rendimiento del volumen de almacenamiento se aplican al diseño de Azure Storage. Por ejemplo, la unión de varios discos para crear un volumen de disco mayor mejora el rendimiento de E/S. La habilitación de la caché de lectura en el contenido de almacenamiento que cambia con poca frecuencia mejora la velocidad de recuperación de datos. Para más información acerca de los requisitos de rendimiento, consulte [SAP note 1943937 - Hardware Configuration Check Tool][sap-1943937] (SAP nota 1943937: Herramienta de comprobación de la configuración del hardware).

Para el almacén de datos de copia de seguridad, se recomienda utilizar el [nivel de almacenamiento de acceso esporádico][cool-blob-storage] de Azure Blob Storage. El nivel de almacenamiento de acceso esporádico es una forma rentable de almacenar los datos a los que se accede con menos frecuencia y son de larga duración.

## <a name="scalability-considerations"></a>Consideraciones sobre escalabilidad

En el nivel de aplicación de SAP, Azure ofrece una amplia variedad de tamaños de máquina virtual para el escalado vertical. Para ver una lista inclusiva, consulte [SAP note 1928533 - Applications on Azure: Supported Products and Azure VM types][sap-1928533] (SAP nota 1928533: Aplicaciones de SAP en Azure: tipos de máquina virtual de Azure y productos compatibles). Realice un escalado horizontal mediante la adición de más máquinas virtuales al conjunto de disponibilidad.

En el caso de SAP HANA en máquinas virtuales de Azure con aplicaciones OLTP y OLAP SAP, el tamaño de la máquina virtual con certificación SAP es GS5 con una sola instancia de máquina virtual. Para cargas de trabajo mayores, Microsoft también ofrece [Azure Large Instances][azure-large-instances] para SAP HANA en servidores físicos que están en la misma ubicación que un centro de datos con certificación de Microsoft Azure, que proporciona un máximo de 4 TB de capacidad de memoria para una sola instancia en este momento. La configuración con varios nodos también es posible con una capacidad de memoria total de hasta 32 TB.

## <a name="availability-considerations"></a>Consideraciones sobre disponibilidad

En esta instalación distribuida de la aplicación de SAP en una base de datos centralizada, la instalación base se replica para lograr una alta disponibilidad. Para cada capa de la arquitectura, el diseño de la alta disponibilidad varía:

- **Web Dispatcher**. La alta disponibilidad se logra con instancias de SAP Web Dispatcher redundantes con tráfico de la aplicación de SAP. Vea [SAP Web Dispatcher][swd] en la documentación de SAP.

- **ASCS.** Para lograr alta disponibilidad de ASCS en máquinas virtuales Windows de Azure, se usa Windows Sever Failover Clustering con SIOS DataKeeper para implementar el volumen compartido del clúster. Para obtener detalles acerca de la implementación, consulte [Clustering SAP ASCS on Azure][clustering] (Agrupación de SAP ASCS en clústeres en Azure).

- **Servidores de aplicaciones.** La alta disponibilidad se logra mediante el equilibrio de la carga de tráfico dentro de un grupo de servidores de aplicaciones.

- **Nivel de base de datos.** Esta arquitectura de referencia implementa una única instancia de base de datos de SAP HANA. Para lograr alta disponibilidad, implemente más de una instancia y utilice la replicación del de HANA (HSR) para implementar la conmutación por error manual. Para habilitar la conmutación automática por error, se requiere una extensión HA para la distribución específica de Linux.

### <a name="disaster-recovery-considerations"></a>Consideraciones acerca de la recuperación ante desastres

Cada nivel utiliza una estrategia diferente para proporcionar protección mediante la recuperación ante desastres (DR).

- **Servidores de aplicaciones.** Los servidores de aplicaciones de SAP no contienen datos empresariales. En Azure, una estrategia de recuperación ante desastres simple es crear servidores de aplicaciones SAP en otra región. Tras cualquier cambio de configuración o actualización del kernel en el servidor de aplicaciones principal, los mismos cambios se deben copiar a las máquinas virtuales de la región de recuperación ante desastres. Por ejemplo, los archivos ejecutables de kernel se copian en las máquinas virtuales de recuperación ante desastres.

- **Servicios centrales de SAP.** Este componente de la pila de aplicaciones SAP tampoco conserva los datos empresariales. Puede crear una máquina virtual en la región de la DR para que ejecute el rol de SCS. El único contenido del nodo de SCS principal que se sincroniza es el del recurso compartido **/sapmnt**. Además, si los cambios de configuración o las actualizaciones del kernel tienen lugar en los servidores de SCS principales, se deben repetir en el SCS de recuperación ante desastres. Para sincronizar los dos servidores, no hay más que usar un trabajo de copia programado regularmente para copiar **/sapmnt** a la parte de recuperación ante desastres. Para más información acerca del proceso de compilación, copia y conmutación por error de prueba, descargue [SAP NetWeaver: Building a Hyper-V and Microsoft Azure–based Disaster Recovery Solution][sap-netweaver-dr] (SAP NetWeaver: creación de una solución de recuperación ante desastres basada en Hyper-V y Microsoft Azure) y consulte "4.3. Capa de SAP SPOF (ASCS)."

- **Nivel de base de datos.** Use soluciones de replicación compatibles con HANA como HSR o Replicación de Storage. 

## <a name="manageability-considerations"></a>Consideraciones sobre la manejabilidad

SAP HANA tiene una característica de copia de seguridad que usa la infraestructura subyacente de Azure. Para hacer una copia de seguridad de la base de datos de SAP HANA que se ejecuta en las máquinas virtuales de Azure, tanto la instantánea de SAP HANA como la de Azure Storage se usan para asegurar la coherencia de los archivos de copia de seguridad. Para más información, consulte [Guía de copia de seguridad de SAP HANA en Azure Virtual Machines][hana-backup] y [Preguntas sobre el servicio Azure Backup][backup-faq].

Azure proporciona varias funciones para la [supervisión y el diagnóstico][monitoring] de la infraestructura global. Además, la supervisión mejorada de las máquinas virtuales de Azure (Linux o Windows) la controla Azure Operations Management Suite (OMS).

Para proporcionar una supervisión basada en SAP de los recursos y un rendimiento del servicio de la infraestructura de SAP, utilice la extensión de Azure SAP Enhanced Monitoring. Esta extensión introduce las estadísticas de supervisión de Azure en la aplicación de SAP para la supervisión de sistema operativo y las funciones de DBA Cockpit. 

## <a name="security-considerations"></a>Consideraciones sobre la seguridad

SAP tiene su propio motor de administración de usuarios (UME) para controlar el acceso basado en rol y la autorización en la aplicación de SAP. Para más información, consulte [SAP HANA Security - An Overview][sap-security] (Introducción a la seguridad de SAP HANA) (para acceder se requiere una cuenta de SAP Service Marketplace).

Por la seguridad de la infraestructura, los datos se protegen tanto en tránsito como en reposo. La sección "Security considerations" (Consideraciones de seguridad) de [SAP NetWeaver on Azure Virtual Machines (VMs) – Planning and Implementation Guide][netweaver-on-azure] (empieza a afrontar la seguridad de la red). La guía también especifica los puertos de red que se deben abrir en los firewalls para permitir la comunicación de la aplicación. 

Para cifrar discos de una máquina virtual IaaS Windows y Linux, puede usar [Azure Disk Encryption][disk-encryption]. Azure Disk Encryption usa la característica BitLocker de Windows y la característica DM-Crypt de Linux para ofrecer cifrado de volúmenes al sistema operativo y a los discos de datos. La solución también funciona con Azure Key Vault para ayudarle a controlar y administrar las claves y los secretos del cifrado de discos en la suscripción de Key Vault. Los datos de los discos de máquinas virtuales se cifran en reposo en Azure Storage.

Para el cifrado de datos en reposo de SAP HANA, se recomienda utilizar la tecnología de cifrado nativa de SAP HANA.

> [!NOTE]
> No utilice el cifrado de datos en reposo de HANA con el cifrado de disco de Azure en el mismo servidor.

Considere la posibilidad de usar [grupos de seguridad de red][nsg] (NSG) para restringir el tráfico entre las distintas subredes de la red virtual.

## <a name="deploy-the-solution"></a>Implementación de la solución 

Los scripts de implementación de esta arquitectura de referencia están disponibles en [GitHub][github].


### <a name="prerequisites"></a>Requisitos previos

- Para completar la instalación, es preciso tener acceso al centro de descarga de software de SAP.
 
- Instale la versión más reciente de [Azure PowerShell][azure-ps]. 

- Esta implementación requiere 51 núcleos. Antes de realizar la implementación, compruebe que la suscripción tiene cuota suficiente para los núcleos de la máquina virtual. Si no la tiene, use Azure Portal para enviar una solicitud de más cuota al soporte técnico.
 
- Esta implementación utiliza una máquina virtual de la serie GS. Compruebe la disponibilidad de la serie GS por región [aquí][region-availability].

- Para calcular el costo de esta implementación, consulte la [Calculadora de precios de Azure][azure-pricing]. 
 
Esta arquitectura de referencia implementa las siguientes máquinas virtuales:

| Nombre del recurso | Tamaño de VM | Propósito  |
|---------------|---------|----------|
| `ra-sapApps-scs-vm1` ... `ra-sapApps-scs-vmN` | DS11v2 | Servicios centrales de SAP |
| `ra-sapApps-vm1` ... `ra-sapApps-vmN` | DS11v2 | Aplicación SAP NetWeaver |
| `ra-sap-wdp-vm1` ... `ra-sap-wdp-vmN` | DS11v2 | SAP Web Dispatcher |
| `ra-sap-data-vm1` | GS5 | Instancia de base de datos de SAP HANA |
| `ra-sap-jumpbox-vm1` | DS1V2 | Jumpbox |

Se implementa una única instancia de SAP HANA. En el caso de las máquinas virtuales de aplicación, el número de instancias que se van implementar se especifica en los parámetros de la plantilla.

### <a name="deploy-sap-infrastructure"></a>Implementación de la infraestructura de SAP

Esta arquitectura se puede implementar de forma incremental o toda de una vez. La primera vez se recomienda realizar una implementación incremental, con el fin de poder ver lo que hace cada implementación. Especifique el incremento mediante uno de los siguientes parámetros de *modo*

| Modo           | Qué hace                                                                                                            |
|----------------|-----------------------------------------------------|
| infrastructure | Implementa la infraestructura de red en Azure.        |
| carga de trabajo       | Implementa los servidores de SAP en la red.             |
| todo            | Implementa todas las implementaciones anteriores.              |

Para implementar la solución, siga estos pasos:

1. Descargue o clone el [repositorio de GitHub][github] en el equipo local.

2. Abra una ventana de PowerShell y navegue hasta la carpeta `/sap/sap-hana/`.

3. Ejecute el siguiente cmdlet de PowerShell. Para `<subscription id>`, use el identificador de su suscripción a Azure. Para `<location>`, especifique una región de Azure, como `eastus` o `westus`. Para `<mode>`, especifique uno de los modos enumerados anteriormente.

    ```powershell
     .\Deploy-ReferenceArchitecture -SubscriptionId <subscription id> -Location <location> -ResourceGroupName <resource group> <mode>
    ```

4.  Cuando se le solicite, inicie sesión en su cuenta de Azure. 

Los scripts de implementación pueden tardar varias horas en completarse, según el modo seleccionado.

> [!WARNING]
> Los archivos de parámetros incluyen una contraseña codificada de forma rígida (`AweS0me@PW`) en varios lugares. Cambie estos valores antes de realizar la implementación.
 
### <a name="configure-sap-applications-and-database"></a>Configurar la base de datos y aplicaciones de SAP

Después de implementar la infraestructura de SAP, instale y configure las aplicaciones de SAP y la base de datos de HANA en las máquinas virtuales como se indica a continuación.

> [!NOTE]
> Para obtener instrucciones de instalación de SAP, debe tener el nombre de usuario y la contraseña del Portal de SAP Support para descargar las [guías de instalación de SAP][sap-guide].

1. Inicie sesión en el Jumpbox (`ra-sap-jumpbox-vm1`). El Jumpbox lo usará para iniciar sesión en las otras máquinas virtuales. 

2.  En todas las máquinas virtuales denominadas `ra-sap-wdp-vm1` ... `ra-sap-wdp-vmN`, inicie sesión en la máquina virtual e instale y configure la instancia de SAP Web Dispatcher siguiendo los pasos que se describen en la wiki [Web Dispatcher Installation][sap-dispatcher-install].

3.  Inicie sesión en la máquina virtual denominada `ra-sap-data-vm1`. Instale y configure la instancia de SAP Hana Database mediante la guía [SAP HANA Server Installation and Update Guide][hana-guide].

4. En todas las máquinas virtuales denominadas `ra-sapApps-scs-vm1` ... `ra-sapApps-scs-vmN`, inicie sesión en la máquina virtual e instale y configure los servicios centrales de SAP mediante la [guías de instalación de SAP][sap-guide].

5.  En todas las máquinas virtuales denominadas `ra-sapApps-vm1` ... `ra-sapApps-vmN`, inicie sesión en la máquina virtual e instale y configure la aplicación SAP Netweaver mediante la [guías de instalación de SAP][sap-guide].

**_Colaboradores de esta arquitectura de referencia_**  &mdash; Rick Rainey, Ross Sponholtz, Ben Trinh

[azure-large-instances]: /azure/virtual-machines/workloads/sap/hana-overview-architecture
[azure-lb]: /azure/load-balancer/load-balancer-overview
[azure-pricing]: https://azure.microsoft.com/pricing/calculator/
[azure-ps]: /powershell/azure/overview
[backup-faq]: /azure/backup/backup-azure-backup-faq
[clustering]: https://blogs.msdn.microsoft.com/saponsqlserver/2015/05/20/clustering-sap-ascs-instance-using-windows-server-failover-cluster-on-microsoft-azure-with-sios-datakeeper-and-azure-internal-load-balancer/
[cool-blob-storage]: /azure/storage/storage-blob-storage-tiers
[disk-encryption]: /azure/security/azure-security-disk-encryption
[github]: https://github.com/mspnp/reference-architectures/tree/master/sap/sap-hana
[hana-backup]: /azure/virtual-machines/workloads/sap/sap-hana-backup-guide
[hana-guide]: https://help.sap.com/viewer/2c1988d620e04368aa4103bf26f17727/2.0.01/en-US/7eb0167eb35e4e2885415205b8383584.html
[hybrid-networking]: ../hybrid-networking/index.md
[logon-groups]: https://wiki.scn.sap.com/wiki/display/SI/ABAP+Logon+Group+based+Load+Balancing
[managed-disks]: /azure/storage/storage-managed-disks-overview
[monitoring]: /azure/architecture/best-practices/monitoring
[multiple-vm-nics]: /azure/virtual-machines/windows/multiple-nics
[netweaver-on-azure]: /azure/virtual-machines/workloads/sap/planning-guide
[nsg]: /azure/virtual-network/virtual-networks-nsg
[region-availability]: https://azure.microsoft.com/regions/services/
[running-SAP]: https://blogs.msdn.microsoft.com/saponsqlserver/2016/06/07/sap-on-sql-general-update-for-customers-partners-june-2016/
[sap-1943937]: https://launchpad.support.sap.com/#/notes/1943937
[sap-1928533]: https://launchpad.support.sap.com/#/notes/1928533
[sap-dispatcher]: https://help.sap.com/doc/saphelp_nw73ehp1/7.31.19/en-US/48/8fe37933114e6fe10000000a421937/frameset.htm
[sap-dispatcher-ha]: https://help.sap.com/doc/saphelp_nw73ehp1/7.31.19/en-US/48/9a9a6b48c673e8e10000000a42189b/frameset.htm
[sap-dispatcher-install]: https://wiki.scn.sap.com/wiki/display/SI/Web+Dispatcher+Installation
[sap-guide]: https://service.sap.com/instguides
[sap-ha]: https://support.sap.com/content/dam/SAAP/SAP_Activate/AGS_70.pdf
[sap-hana-on-azure]: https://azure.microsoft.com/services/virtual-machines/sap-hana/
[sap-netweaver-dr]: http://download.microsoft.com/download/9/5/6/956FEDC3-702D-4EFB-A7D3-2DB7505566B6/SAP%20NetWeaver%20-%20Building%20an%20Azure%20based%20Disaster%20Recovery%20Solution%20V1_5%20.docx
[sap-security]: https://archive.sap.com/documents/docs/DOC-62943
[visio-download]: https://archcenter.azureedge.net/cdn/SAP-HANA-architecture.vsdx
[vm-sizes-mem]: /azure/virtual-machines/windows/sizes-memory
[swd]: https://help.sap.com/doc/saphelp_nw70ehp2/7.02.16/en-us/48/8fe37933114e6fe10000000a421937/frameset.htm
[0]: ./images/sap-hana.png "Arquitectura de SAP HANA mediante Microsoft Azure"
