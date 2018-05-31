---
title: Ejecución de SAP HANA en Azure (instancias grandes)
description: Prácticas probadas para ejecutar SAP HANA en un entorno de alta disponibilidad en Azure (instancias grandes).
author: lbrader
ms.date: 05/16/2018
ms.openlocfilehash: 7605fa8a0012aaef3f7323c6f88614b640152e3b
ms.sourcegitcommit: bb348bd3a8a4e27ef61e8eee74b54b07b65dbf98
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/21/2018
ms.locfileid: "34423091"
---
# <a name="run-sap-hana-on-azure-large-instances"></a>Ejecución de SAP HANA en Azure (instancias grandes)

Esta arquitectura de referencia muestra un conjunto de prácticas probadas para ejecutar SAP HANA en Azure (instancias grandes) con alta disponibilidad y recuperación ante desastres. Esta oferta se llama HANA (instancias grandes) y se implementa en los servidores físicos de las regiones de Azure. 

![0][0]

> [!NOTE]
> La implementación de esta arquitectura de referencia requiere licencias adecuadas de los productos de SAP y de otras tecnologías que no son de Microsoft.

## <a name="architecture"></a>Architecture

La arquitectura consta de los siguientes componentes de infraestructura.

- **Red virtual**. El servicio [Azure Virtual Network][vnet] se conecta a recursos de Azure entre sí y de forma segura y se subdivide en [subredes][subnet] diferentes para cada capa. Los niveles de la aplicación SAP se implementan en máquinas virtuales de Azure (VM) para conectarse a la capa de base de datos HANA que reside en instancias de gran tamaño.

- **Máquinas virtuales**. Las máquinas virtuales se utilizan tanto en el nivel de aplicación de SAP como en la capa de servicios compartidos. La última incluye una JumpBox que usan los administradores para configurar HANA (instancias grandes) y para proporcionar acceso a otras máquinas virtuales. 

- **HANA (instancias grandes)**. A [servidor físico][physical] certificado que cumple los estándares de SAP HANA Tailored Datacenter Integration (TDI) es el que ejecuta SAP HANA. Esta arquitectura utiliza dos instancias de HANA (instancias grandes): una unidad de proceso principal y otra secundaria. La alta disponibilidad en la capa de datos se proporciona a través de HANA System Replication (HSR).

- **Par de alta disponibilidad**. Un grupo de hojas de HANA (instancias grandes) se administran de forma conjunta para proporcionar confiabilidad y redundancia a las aplicaciones. 

- **MSEE (Microsoft Enterprise Edge)**. MSEE es un punto de conexión de un proveedor de conectividad o del perímetro de la red que atraviesa un circuito de ExpressRoute. 

- **Tarjetas de interfaz de red (NIC)**. Para habilitar la comunicación, el servidor de HANA (instancias grandes) proporciona cuatro tarjetas de interfaz de red virtuales de forma predeterminada. Esta arquitectura requiere una para la comunicación del cliente, otra para la conectividad de nodo a nodo que necesita HSR, una tercera para el almacenamiento de HANA (instancias grandes) y una cuarta para el iSCSI que se usa en la agrupación en clústeres de alta disponibilidad.
    
- **Almacenamiento de Network File System (NFS)**. El servidor de [NFS][nfs] admite el recurso compartido de archivos de red que proporciona la persistencia de datos seguros en HANA (archivos grandes).

- **ExpressRoute.** [ExpressRoute][expressroute] es el servicio de red de Azure recomendado para crear conexiones privadas entre una red local y las redes virtuales de Azure que no pasan por Internet. Las máquinas virtuales de Azure se conectan a HANA (instancias grandes) mediante otra conexión de ExpressRoute. La conexión de ExpressRoute entre la red virtual de Azure y HANA (instancias grandes ) se configura como parte de la oferta de Microsoft.

- **Puerta de enlace**. La puerta de enlace de ExpressRoute se utiliza para conectar la red virtual Azure que se usa para el nivel de aplicación de SAP a la red de HANA (instancias grandes). Use las SKU [Alto rendimiento o Ultrarrendimiento][sku].

- **Recuperación ante desastres (DR)**. Bajo solicitud, se admite la replicación de almacenamiento y se configurará desde el sitio principal al [de recuperación ante desastres][DR-site] ubicado en otra región.  
 
## <a name="recommendations"></a>Recomendaciones
Las recomendaciones pueden variar, así que use estas como punto de partida.

### <a name="hana-large-instances-compute"></a>Proceso de HANA (instancias grandes)
Las [instancias grandes][physical] son servidores físicos que tienen la arquitectura de CPU de Intel EX E7 y están configurados en un sello de instancias grandes (es decir, un conjunto específico de hojas o servidores). Una unidad de proceso equivale a un servidor o a una hoja y una marca consta de varios servidores u hojas. En un sello de instancias grandes, los servidores no se comparten y se dedican a ejecutar la implementación de un cliente de SAP HANA.

Hay varias SKU disponibles para HANA (instancias grandes), que admiten instancias individuales de hasta 20 TB (60 TB de escalabilidad horizontal) de memoria para S/4HANA u otras cargas de trabajo de SAP HANA. Se ofrecen [dos clases][classes] de servidores:

- Clase Tipo I: S72, S72m, S144, S144m, S192 y S192m

- Clase Tipo II: S384, S384m, S384xm, S576m, S768m y S960m

Por ejemplo, la SKU S72 incluye 768 GB de RAM, 3 terabytes (TB) de almacenamiento y 2 procesadores Intel Xeon (E7-8890 v3) con 36 núcleos. Elija una SKU que cumpla los requisitos de tamaño que determinó en las sesiones de arquitectura y diseño. Asegúrese siempre de que el tamaño se aplica a la SKU correcta. Tanto las funcionalidades como los requisitos de implementación [varían en función del tipo][type] y la disponibilidad varía en función de la [región][region]. También puede pasar de una SKU a otra mayor.

Microsoft le ayuda a establecer la configuración de las instancias grandes, pero es responsabilidad suya comprobar la del sistema operativo. Asegúrese de revisar las últimas notas de SAP relativas a su versión exacta de Linux.

### <a name="storage"></a>Storage
El diseño del almacenamiento se implementa en función de la recomendación de TDI para SAP HANA. HANA (instancias grandes) incluye una configuración de almacenamiento específica para las especificaciones estándar de TDI. Sin embargo, se puede adquirir almacenamiento adicional en incrementos de 1 TB. 

Para admitir los requisitos de los entornos de misión crítica, lo que incluye la recuperación rápida, se usa NFS, no el almacenamiento conectado de forma directa. El servidor de almacenamiento de NFS para HANA (instancias grandes) está hospedado en un entorno de varios inquilinos, en el que los inquilinos se segregan y se protegen mediante el uso del aislamiento de los procesos, la red y el almacenamiento.

Para lograr una alta disponibilidad en el sitio principal, use diferentes diseños de almacenamiento. Por ejemplo, en una escalabilidad horizontal con varios hosts, el almacenamiento se comparte. Otra opción de alta disponibilidad es la replicación que usa aplicaciones como HSR. Sin embargo, en el caso de la recuperación ante desastres, se usa una replicación de almacenamiento que usa instantáneas.

### <a name="networking"></a>Redes
Esta arquitectura utiliza redes físicas y virtuales. La red virtual forma parte de IaaS de Azure y se conecta a una red física discreta de HANA (instancias grandes) a través de circuitos [ExpressRoute][expressroute]. Una puerta de enlace entre locales conecta las cargas de trabajo de la red virtual de Azure con los sitios locales.

Las redes de HANA (instancias grandes) están aisladas entre sí por motivos de seguridad. Las instancias que residen en diferentes regiones no se comunican entre sí, excepto para la replicación del almacenamiento dedicado. Sin embargo, para usar HSR, es necesario que haya comunicaciones entre las distintas regiones. Se pueden usar [tablas de enrutamiento de IP][ip] o servidores proxy para habilitar HSR entre regiones.

Todas las redes virtuales de Azure que se conectan HANA (instancias grandes) en una región se pueden [interconectar][cross-connected] a través de ExpressRoute a HANA (instancias grandes) en una región secundaria.

ExpressRoute para HANA (instancias grandes) se incluye de forma predeterminada durante el aprovisionamiento. Para la configuración se necesita un diseño de red específico, lo que incluye el enrutamiento de dominio y incluidos los intervalos de direcciones CIDR necesarios. Para más información, consulte [Infraestructura y conectividad con SAP HANA en Azure (instancias grandes)][HLI-infrastructure].

## <a name="scalability-considerations"></a>Consideraciones sobre escalabilidad
Para escalar o reducir verticalmente, puede elegir entre la gran cantidad tamaños de servidores que están disponibles para HANA (instancias grandes). Están clasificados por categorías como [Tipo I y Tipo II][classes] y personalizados para las diferentes cargas de trabajo. Elija un tamaño que puede aumentar al mismo ritmo que la carga de trabajo durante los tres años siguientes. También están disponibles los compromisos de un año.

Una implementación escalada de múltiples hosts suele utilizarse para implementaciones de BW/4HANA como un tipo de estrategia de creación de particiones en una base de datos. Para realizar escalados horizontales, planee la colocación de las tablas HANA antes de la instalación. Desde una perspectiva de la infraestructura, varios hosts están conectados a un volumen de almacenamiento compartido, lo que permite que los hosts en espera tomen el control rápidamente en caso de que se produzca un error en uno de los nodos de trabajo de proceso del sistema de HANA.

S/4HANA y SAP Business Suite en HANA en una sola hoja se pueden escalar hasta 20 TB con a una sola instancia de HANA (instancias grandes).

En los escenarios nuevos, [SAP Quick Sizer][quick-sizer] está disponible para calcular los requisitos de memoria de la implementación del software de SAP en HANA. Los requisitos de memoria de HANA aumentan a medida que aumenta el volumen de datos. Use el consumo de memoria actual del sistema como base para predecir el consumo futuro y, después, asigne la petición a uno de los tamaños de HANA (instancias grandes).

Si ya tiene implementaciones de SAP, SAP proporciona informes que puede usar para comprobar los datos que usan los sistemas existentes y para calcular los requisitos de memoria de una instancia de HANA. Por ejemplo, vea las siguientes notas de SAP: 

- SAP Note [1793345][sap-1793345] - Sizing for SAP Suite on HANA (Nota de SAP 1793345: Ajuste de tamaño para SAP Suite en HANA)
- SAP Note [1872170][sap-1872170] - Suite on HANA and S/4 HANA sizing report (Nota de SAP 1872170: Informe de ajuste de tamaño de Suite en HANA y S/4 HANA)
- SAP Note [2121330][sap-2121330] - FAQ: SAP BW on HANA Sizing Report (Nota de SAP 2121330: P+F: informe de ajuste de tamaño de SAP BW en HANA)
- SAP Note [1736976][sap-1736976] - Sizing Report for BW on HANA (Nota de SAP 1736976: Informe de ajuste de tamaño para BW en HANA)
- SAP Note [2296290][sap-2296290] - New Sizing Report for BW on HANA (Nota de SAP 2296290: Nuevo informe de ajuste de tamaño para BW en HANA)

## <a name="availability-considerations"></a>Consideraciones sobre disponibilidad

La redundancia de recursos es el tema general en las soluciones de infraestructura de alta disponibilidad. Para las empresas que tienen un Acuerdo de Nivel de Servicio menos estrictos, las máquinas virtuales de Azure de instancia única ofrecen un SLA de tiempo de actividad. Para más información, consulte [Contratos de nivel de servicio](https://azure.microsoft.com/support/legal/sla/).

Trabaje con SAP, un integrador de sistemas o Microsoft para diseñar e implementar correctamente una estrategia de [alta disponibilidad y recuperación ante desastres][hli-hadr]. Esta arquitectura se rige por el [Acuerdo de Nivel de Servicio][sla] (SLA) de Azure para HANA en Azure (instancias grandes). Para evaluar sus requisitos de disponibilidad, tenga en cuenta los únicos puntos de error, el nivel deseado de tiempo de actividad de los servicios y estas métricas comunes:

- Objetivo de tiempo de recuperación (RTO) significa el periodo de tiempo en que el servidor de HANA (instancias grandes) no está disponible.

- Objetivo de punto de recuperación (RPO) significa el período máximo tolerable en que los datos del cliente pueden perderse debido a un error.

Para lograr alta disponibilidad, implemente más de una instancia en un par de alta disponibilidad y use HSR en modo sincrónico para minimizar el tiempo de inactividad y la pérdida de datos. Además de una configuración local de dos nodos de alta disponibilidad, HSR admite la replicación multinivel, en la que un tercer nodo de una región de Azure independiente se registra en la réplica secundaria del par HSR con clústeres como su destino de replicación. De este modo se forma una cadena de margarita de replicación. La conmutación por error en el nodo de recuperación ante desastres es un proceso manual.

Al configurar el HSR de HANA (instancias grandes) con conmutación por error automática, puede solicitar el equipo de Microsoft Service Management que configure un [dispositivo STONITH][stonith] para los servidores existentes. 

## <a name="disaster-recovery-considerations"></a>Consideraciones acerca de la recuperación ante desastres
Esta arquitectura admite la [recuperación ante desastres][hli-dr] entre HANA (instancias grandes) en diferentes regiones de Azure. Hay dos formas de admitir la recuperación ante desastres con HANA (instancias grandes):

- Replicación de almacenamiento. El contenido de almacenamiento principal se replica constantemente en los sistemas remotos de almacenamiento de recuperación ante desastres que están disponibles en el servidor de HANA (instancias grandes) de recuperación ante desastres designado. En la replicación de almacenamiento, la base de datos de HANA no se carga en memoria. Esta opción de recuperación ante desastres es más sencilla desde una perspectiva de administración. Para determinar si se trata de una estrategia adecuada, considere el tiempo de carga de la base de datos con respecto al Acuerdo de Nivel de Servicio de disponibilidad. La replicación de almacenamiento también permite realizar la recuperación en un momento dado. Si se ha configurado una recuperación ante desastres con varios fines (optimización de costo), debe adquirir almacenamiento adicional del mismo tamaño en la ubicación de recuperación ante desastres. Microsoft proporciona [scripts de conmutación por error y de instantáneas de almacenamiento][scripts] de autoservicios para la conmutación por error de HANA como parte de la oferta de HANA (instancias grandes).

- HSR de varios nivel con una tercera réplica en la región de recuperación ante desastres (donde la base de datos HANA esté cargado en memoria). Esta opción admite un tiempo de recuperación menor, pero no admite la recuperación en un momento dado. HSR requiere un sistema secundario. La replicación del sistema de HANA en el sitio de recuperación ante desastres se controla a través de servidores proxy, como nginx o tablas de IP. 

> [!NOTE]
> Los costos de esta arquitectura de referencia se puede optimizar mediante la ejecución en un entorno de instancia única. Este [escenario con costos optimizados](https://blogs.sap.com/2016/07/19/new-whitepaper-for-high-availability-for-sap-hana-cost-optimized-scenario/) es apropiado para las cargas de trabajo de HANA que no sean de producción. 

## <a name="backup-considerations"></a>Consideraciones de copia de seguridad
En función de sus requisitos empresariales, elija cualesquiera de las distintas opciones disponibles para [copias de seguridad y recuperación][hli-backup].

| Opción de copia de seguridad                   | Ventajas                                                                                                   | Desventajas                                                       |
|---------------------------------|--------------------------------------------------------------------------------------------------------|------------------------------------------------------------|
| Copia de seguridad de HANA        | Nativo de SAP. Comprobación de coherencia integrada.                                                             | Tiempos de copia de seguridad y recuperación prolongados. Consumo de espacio de almacenamiento. |
| Instantánea de HANA      | Nativo de SAP. Copia de seguridad y restauración rápidas.                                                               |                                       |
| Instantánea de almacenamiento   | Se incluye en HANA (instancias grandes). Recuperación ante desastres optimizada para HANA (instancias grandes). Compatibilidad de copia de seguridad de volumen de arranque. | Un máximo de 254 instantáneas por volumen.                          |
| Copia de seguridad de registro         | Necesario para la recuperación a un momento dado.                                                                   |                                                            |
| Otras herramientas de copia de seguridad | Ubicación de copia de seguridad redundante.                                                                             | Costos de licencias adicionales.                                |

Además, en SapHanaTutorial.com se puede encontrar un artículo útil, [Comparison between HANA backup options][sap-hana-tutorial] (Comparación de las opciones de copia de seguridad de HANA).

## <a name="manageability-considerations"></a>Consideraciones sobre la manejabilidad
Supervise los recursos de HANA (instancias grandes ) como la CPU, la memoria, el ancho de banda de red y el espacio de almacenamiento con SAP HANA Studio, SAP HANA Cockpit, SAP Solution Manager y otras herramientas nativas de Linux. HANA (instancias grandes) no incluye herramientas de supervisión integradas. Microsoft ofrece recursos que ayudan a [solucionar problemas y realizar supervisión][hli-troubleshoot] según los requisitos de su organización y el equipo de soporte técnico de Microsoft puede ayudarle a solucionar problemas técnicos. 

Si necesita más funcionalidad de cálculo, debe obtener una SKU mayor. 

## <a name="security-considerations"></a>Consideraciones sobre la seguridad
- De forma predeterminada, HANA (instancias grandes) utiliza el cifrado de almacenamiento basado en TDE (cifrado de datos transparente) para los datos en reposo.

- Los datos en tránsito entre HANA (instancias grandes) y las máquinas virtuales no se cifran. Para cifrar la transferencia de datos, habilite el cifrado específico de la aplicación. Consulte SAP Note [2159014][sap-2159014] – FAQ: SAP HANA Security (Nota 2100040 de SAP: P+F; seguridad de SAP HANA).

- El aislamiento proporciona seguridad entre los inquilinos en el entorno multiinquilino de HANA (instancias grandes). Los inquilinos se aíslan mediante su propia VLAN.

- Los [procedimientos recomendados de seguridad de la red de Azure][network-best-practices] proporcionan una guía útil.

- Al igual que en cualquier otra implementación, se recomienda la [protección del sistema operativo][os-hardening].

- Por motivos de seguridad física, el acceso a los centros de datos de Azure está limitado solo a personal autorizado. Ningún cliente puede acceder a los servidores físicos.

Para más información, consulte [SAP HANA Security—An Overview][sap-security] (Seguridad de SAP HANA: descripción general).( Se requiere una cuenta de SAP Service Marketplace para acceder)

## <a name="communities"></a>Comunidades
Las comunidades pueden responder preguntas y ayudarle a configurar una implementación correcta. Tenga en cuenta lo siguiente.

* [Blog Ejecución de aplicaciones SAP en la plataforma de Microsoft][running-sap-blog]
* [Soporte técnico de la comunidad de Azure][azure-forum]
* [SAP Community][sap-community]
* [Stack Overflow SAP][stack-overflow]

[azure-forum]: https://azure.microsoft.com/support/forums/
[azure-large-instances]: /azure/virtual-machines/workloads/sap/hana-overview-architecture
[classes]: /azure/virtual-machines/workloads/sap/hana-overview-architecture
[cross-connected]: /azure/virtual-machines/workloads/sap/hana-overview-high-availability-disaster-recovery#network-considerations-for-disaster-recovery-with-hana-large-instances
[dr-site]: /azure/virtual-machines/workloads/sap/hana-overview-high-availability-disaster-recovery
[expressroute]: /azure/architecture/reference-architectures/hybrid-networking/expressroute
[filter-network]: https://azure.microsoft.com/blog/multiple-vm-nics-and-network-virtual-appliances-in-azure/
[hli-dr]: /azure/virtual-machines/workloads/sap/hana-overview-high-availability-disaster-recovery#network-considerations-for-disaster-recovery-with-hana-large-instances
[hli-backup]: /azure/virtual-machines/workloads/sap/hana-overview-high-availability-disaster-recovery#backup-and-restore
[hli-hadr]: /azure/virtual-machines/workloads/sap/hana-overview-high-availability-disaster-recovery?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json
[hli-infrastructure]: /azure/virtual-machines/workloads/sap/hana-overview-infrastructure-connectivity
[hli-overview]: /azure/virtual-machines/workloads/sap/hana-overview-architecture
[hli-troubleshoot]: /azure/virtual-machines/workloads/sap/troubleshooting-monitoring
[ip]: https://blogs.msdn.microsoft.com/saponsqlserver/2018/02/10/setting-up-hana-system-replication-on-azure-hana-large-instances/
[network-best-practices]: /azure/security/azure-security-network-security-best-practices
[nfs]: /azure/virtual-machines/workloads/sap/high-availability-guide-suse-nfs
[os-hardening]: /azure/security/azure-security-iaas
[physical]: /azure/virtual-machines/workloads/sap/hana-overview-architecture
[planning]: /azure/vpn-gateway/vpn-gateway-plan-design
[protecting-sap]: https://blogs.msdn.microsoft.com/saponsqlserver/2016/05/06/protecting-sap-systems-running-on-vmware-with-azure-site-recovery/
[ref-arch]: /azure/architecture/reference-architectures/
[running-SAP]: https://blogs.msdn.microsoft.com/saponsqlserver/2016/06/07/sap-on-sql-general-update-for-customers-partners-june-2016/
[region]: https://azure.microsoft.com/global-infrastructure/services/
[running-sap-blog]: https://blogs.msdn.microsoft.com/saponsqlserver/2017/05/04/sap-on-azure-general-update-for-customers-partners-april-2017/
[quick-sizer]: http://service.sap.com/quicksizing
[sap-1793345]: https://launchpad.support.sap.com/#/notes/1793345
[sap-1872170]: https://launchpad.support.sap.com/#/notes/1872170
[sap-2121330]: https://launchpad.support.sap.com/#/notes/2121330
[sap-2159014]: https://launchpad.support.sap.com/#/notes/2159014
[sap-1736976]: https://launchpad.support.sap.com/#/notes/1736976
[sap-2296290]: https://launchpad.support.sap.com/#/notes/2296290
[sap-community]: https://www.sap.com/community.html
[sap-hana-tutorial]: http://saphanatutorial.com/comparison-between-hana-backup-options/
[sap-security]: https://archive.sap.com/documents/docs/DOC-62943
[scripts]: /azure/virtual-machines/workloads/sap/hana-overview-high-availability-disaster-recovery
[sku]: /azure/expressroute/expressroute-about-virtual-network-gateways
[sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[stack-overflow]: http://stackoverflow.com/tags/sap/info
[stonith]: /azure/virtual-machines/workloads/sap/ha-setup-with-stonith
[subnet]: /azure/virtual-network/virtual-network-manage-subnet
[swd]: https://help.sap.com/doc/saphelp_nw70ehp2/7.02.16/en-us/48/8fe37933114e6fe10000000a421937/frameset.htm
[type]: /azure/virtual-machines/workloads/sap/hana-installation
[vnet]: /azure/virtual-network/virtual-networks-overview
[0]: ./images/sap-hana-large-instances.png "Arquitectura de SAP HANA mediante Azure (instancias grande)"
