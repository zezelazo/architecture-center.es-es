---
title: Aplicación web de varios niveles creada para lograr alta disponibilidad y recuperación ante desastres en Azure
description: Cree una aplicación web de varios niveles para lograr una alta disponibilidad y recuperación ante desastres en Azure mediante máquinas virtuales, conjuntos de disponibilidad y zonas de disponibilidad de Azure, Azure Site Recovery y Azure Traffic Manager
author: sujayt
ms.date: 11/16/2018
ms.custom: product-team
ms.openlocfilehash: 71534dc095d5fba137a0e610d4e725c2efc6b432
ms.sourcegitcommit: a0e8d11543751d681953717f6e78173e597ae207
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/06/2018
ms.locfileid: "53004595"
---
# <a name="multitier-web-application-built-for-high-availability-and-disaster-recovery-on-azure"></a>Aplicación web de varios niveles creada para lograr alta disponibilidad y recuperación ante desastres en Azure

Este escenario de ejemplo es aplicable a cualquier sector que necesite implementar aplicaciones resistentes de varios niveles creadas para obtener una alta disponibilidad y recuperación ante desastres. En este escenario, la aplicación consta de tres niveles.

- Capa web: el nivel superior que incluye la interfaz de usuario. Este nivel analiza las interacciones del usuario y pasa las acciones al siguiente nivel para su procesamiento.
- Capa de negocio: procesa las interacciones de usuario y toma decisiones lógicas sobre los pasos siguientes. Este nivel conecta el nivel web con el nivel de datos.
- Capa de datos: almacena los datos de la aplicación. Normalmente se usa una base de datos, un almacenamiento de objetos o un almacenamiento de archivos.

Entre los escenarios de aplicación se incluye cualquier aplicación crítica que se ejecute en Windows o Linux. Puede tratarse de una aplicación estandarizada, como SAP y SharePoint o una aplicación de línea de negocio personalizada.

## <a name="relevant-use-cases"></a>Casos de uso pertinentes

Otros casos de uso pertinentes incluyen:

* Implementación de aplicaciones altamente resistentes, como SAP y SharePoint
* Diseño de un plan de continuidad empresarial y recuperación ante desastres para aplicaciones de línea de negocio
* Configuración de la recuperación ante desastres y realización de maniobras relacionadas con fines de cumplimiento normativo

## <a name="architecture"></a>Arquitectura

Este escenario muestra una aplicación de varios niveles que usa ASP.NET y Microsoft SQL Server. En las [regiones de Azure que admiten zonas de disponibilidad](/azure/availability-zones/az-overview#regions-that-support-availability-zones) puede implementar las máquinas virtuales (VM) en una región de origen en distintas zonas de disponibilidad y replicarlas en la región de destino que se usa para la recuperación ante desastres. En las regiones de Azure que no admiten zonas de disponibilidad, puede implementar las máquinas virtuales dentro de un conjunto de disponibilidad y replicarlas en la región de destino.

![Introducción a la arquitectura de una aplicación web de varios niveles muy resistente][architecture]

- Distribuya las máquinas virtuales de cada nivel en dos zonas de disponibilidad de regiones que admitan zonas. En otras regiones, implemente las máquinas virtuales de cada nivel dentro de un conjunto de disponibilidad.
- El nivel de base de datos se puede configurar para usar grupos de disponibilidad Always On. Con esta configuración de SQL Server, se configura una base de datos principal dentro de un clúster con un máximo de ocho bases de datos secundarias. Si se produce un problema con la base de datos principal, el clúster conmuta por error a una de las bases de datos secundarias, lo cual permite que la aplicación siga estando disponible. Para más información, consulte [Información general de los grupos de disponibilidad AlwaysOn (SQL Server)][docs-sql-always-on].
- Para escenarios de recuperación ante desastres, puede configurar la replicación nativa asincrónica Always On de SQL en la región de destino que se usa para la recuperación ante desastres. También puede configurar la replicación de Azure Site Recovery en la región de destino si la frecuencia de cambio de datos está dentro de los límites que admite Azure Site Recovery.
- Los usuarios acceden al nivel web de ASP.NET de front-end a través del punto de conexión de Traffic Manager.
- Traffic Manager redirige el tráfico al punto de conexión de la dirección IP pública principal de la región de origen primaria.
- La dirección IP pública redirige la llamada a una de las instancias de máquina virtual del nivel web a través de un equilibrador de carga público. Todas las instancias de máquina virtual de nivel web están en una subred.
- Desde la máquina virtual del nivel web, cada llamada se enruta a una de las instancias de máquina virtual del nivel empresarial mediante un equilibrador de carga interno para su procesamiento. Todas las máquinas virtuales de nivel empresarial están en una subred independiente.
- La operación se procesa en el nivel empresarial y la aplicación ASP.NET se conecta al clúster de Microsoft SQL Server en un nivel de back-end a través de un equilibrador de carga interno de Azure. Estas instancias de SQL Server de back-end están en una subred independiente.
- El punto de conexión secundario de Traffic Manager se configura como la dirección IP pública en la región de destino que se usa para la recuperación ante desastres.
- Si se produce una interrupción en la región primaria, puede invocar la conmutación por error de Azure Site Recovery y la aplicación se activará en la región de destino.
- El punto de conexión de Traffic Manager redirige automáticamente el tráfico de cliente a la dirección IP pública en la región de destino.

### <a name="components"></a>Componentes

* Los [conjuntos de disponibilidad][docs-availability-sets] garantizan que las máquinas virtuales implementadas en Azure se distribuyan entre varios nodos de hardware aislados en un clúster. Si se produce un error de hardware o software en Azure, solo un subconjunto de las máquinas virtuales se verá afectado y toda la solución seguirá disponible y en funcionamiento.
* Las [zonas de disponibilidad][docs-availability-zones] protegen las aplicaciones y los datos de errores del centro de datos. Las zonas de disponibilidad son ubicaciones físicas independientes dentro de una región de Azure. Cada zona de disponibilidad consta de uno o varios centros de datos equipados con alimentación, refrigeración y redes independientes. 
* [Azure Site Recovery (ASR)][docs-azure-site-recovery] le permite replicar las máquinas virtuales en otra región de Azure para satisfacer sus necesidades de continuidad empresarial y recuperación ante desastres. Puede realizar pruebas de recuperación ante desastres periódicas para asegurarse de que satisface los requisitos de cumplimiento. La máquina virtual se replicará en la región seleccionada con la configuración que especifique, de forma que podrá recuperar las aplicaciones en caso de interrupciones del servicio en la región de origen.
* [Azure Traffic Manager][docs-traffic-manager] es un equilibrador de carga de tráfico basado en DNS que distribuye el tráfico de forma óptima a servicios de regiones de Azure globales, al tiempo que proporciona una alta disponibilidad y capacidad de respuesta.
* [Azure Load Balancer][docs-load-balancer] distribuye el tráfico entrante según reglas y sondeos de mantenimiento definidos. Una instancia de Load Balancer proporciona baja latencia y alto rendimiento, y puede escalar hasta millones de flujos para todas las aplicaciones TCP y UDP. Se utiliza un equilibrador de carga público en este escenario para distribuir el tráfico entrante del cliente en el nivel web. Se usa un equilibrador de carga interno en este escenario para distribuir el tráfico desde el nivel empresarial al clúster de SQL Server de back-end.

### <a name="alternatives"></a>Alternativas

* Windows se puede reemplazar fácilmente por otros sistemas operativos ya que nada en la infraestructura depende del sistema operativo.
* [SQL Server para Linux][docs-sql-server-linux] puede reemplazar el almacén de datos de back-end.
* La base de datos puede sustituirse por cualquier aplicación estándar de base de datos disponible.

## <a name="other-considerations"></a>Otras consideraciones

### <a name="scalability"></a>Escalabilidad

Puede agregar o quitar máquinas virtuales en cada nivel según sus requisitos de escalado. Dado que este escenario utiliza equilibradores de carga, puede agregar más máquinas virtuales a un nivel sin que afecte al tiempo de actividad de la aplicación.

Para ver otros temas de escalabilidad, consulte la [lista de comprobación de escalabilidad][scalability] que encontrará en el centro de arquitectura de Azure.

### <a name="security"></a>Seguridad

Los grupos de seguridad de red protegen todo el tráfico de la red virtual hacia la capa de aplicación de front-end. Las reglas limitan el flujo de tráfico de forma que solo las instancias de máquinas virtuales de la capa de aplicación de front-end pueden acceder al nivel de la base de datos de back-end. No se permite ningún tráfico de Internet saliente procedente del nivel empresarial o el nivel de base de datos. Para reducir la superficie de ataque, no hay ningún puerto de administración remota directa abierto. Para más información, consulte [Grupos de seguridad de red de Azure][docs-nsg].

Para obtener instrucciones generales sobre el diseño de escenarios seguros, consulte la [documentación de seguridad de Azure][security].

## <a name="pricing"></a>Precios

La configuración de la recuperación ante desastres para las máquinas virtuales de Azure mediante Azure Site Recovery hará que se incurra en los siguientes gastos de forma continuada.

- Costo de las licencias de Azure Site Recovery por máquina virtual.
- Costos de salida de red para replicar los cambios de datos de los discos de máquina virtual de origen en otra región de Azure. Azure Site Recovery utiliza la compresión integrada para reducir los requisitos de transferencia de datos en un 50 % aproximadamente.
- Costos de almacenamiento en el sitio de recuperación. Estos suelen ser iguales a los del almacenamiento en la región de origen más el almacenamiento adicional necesario para mantener los puntos de recuperación como instantáneas para la recuperación.

Se ha proporcionado una [calculadora de costos de ejemplo][calculator] para configurar la recuperación ante desastres de una aplicación de tres niveles que usa seis máquinas virtuales. Se han preconfigurado todos los servicios en la calculadora de costos. Para ver cómo cambiarían los precios en su caso concreto, cambie las variables adecuadas para estimar el costo.

<!-- links -->
[architecture]: ./media/arhitecture-disaster-recovery-multi-tier-app.png
[autoscaling]: /azure/architecture/best-practices/auto-scaling
[availability]: ../../checklist/availability.md
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[docs-availability-zones]: /azure/availability-zones/az-overview
[docs-load-balancer]: /azure/load-balancer/load-balancer-overview
[docs-nsg]: /azure/virtual-network/security-overview
[docs-vmss]: /azure/virtual-machine-scale-sets/overview
[docs-sql-always-on]: /sql/database-engine/availability-groups/windows/overview-of-always-on-availability-groups-sql-server
[docs-vmss-autoscale]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview
[docs-vnet]: /azure/virtual-network/virtual-networks-overview
[docs-sql-server-linux]: /sql/linux/sql-server-linux-overview?view=sql-server-linux-2017
[docs-traffic-manager]: /azure/traffic-manager/
[docs-azure-site-recovery]: /azure/site-recovery/azure-to-azure-quickstart/
[docs-availability-sets]: /azure/virtual-machines/windows/manage-availability/
[calculator]: https://azure.com/e/6835332265044d6d931d68c917979e6d/
