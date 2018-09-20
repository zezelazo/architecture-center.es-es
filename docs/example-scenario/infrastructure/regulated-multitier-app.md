---
title: Protección de aplicaciones web de Windows para sectores regulados
description: Escenario de eficacia probada a la hora de compilar una aplicación web segura, de varios niveles, con Windows Server en Azure que use conjuntos de escalado, Application Gateway y equilibradores de carga.
author: iainfoulds
ms.date: 07/11/2018
ms.openlocfilehash: 3572f215d9134a6650d76e1b14458226334c6f42
ms.sourcegitcommit: c49aeef818d7dfe271bc4128b230cfc676f05230
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/11/2018
ms.locfileid: "44389298"
---
# <a name="secure-windows-web-application-for-regulated-industries"></a>Protección de aplicaciones web de Windows para sectores regulados

Este escenario de ejemplo es aplicable a sectores regulados que necesitan proteger las aplicaciones de varios niveles. En este escenario, una aplicación front-end de ASP.NET se conecta de forma segura a un clúster back-end protegido de Microsoft SQL Server.

Entre los escenarios de aplicación se incluyen aplicaciones para quirófanos, citas de pacientes y conservación de historiales o renovación y pedido de recetas. Tradicionalmente, las organizaciones usaban aplicaciones y servicios locales heredados para estos escenarios. Con una forma segura y escalable de implementar estas aplicaciones de Windows Server en Azure, las organizaciones pueden modernizar sus implementaciones y reducir los costos operativos locales y la sobrecarga de administración.

## <a name="related-use-cases"></a>Casos de uso relacionados

Tenga en cuenta este escenario para los casos de uso siguientes:

* Modernización de implementaciones de aplicaciones en un entorno seguro en la nube.
* Reducción de la administración de aplicaciones y servicios locales heredados.
* Mejora de la asistencia sanitaria al paciente y experimentación con nuevas plataformas de aplicaciones.

## <a name="architecture"></a>Arquitectura

![Introducción a la arquitectura de los componentes de Azure implicados en aplicaciones de Windows Server de varios niveles para sectores regulados][architecture]

Este escenario incluye una aplicación de varios niveles para sectores regulados que usa ASP.NET y Microsoft SQL Server. Los datos fluyen por el escenario de la siguiente manera:

1. Los usuarios acceden a la aplicación front-end de ASP.NET para sectores regulados a través de Azure Application Gateway.
2. Application Gateway distribuye el tráfico a las instancias de máquinas virtuales de un conjunto de escalado de máquinas virtuales de Azure.
3. La aplicación ASP.NET se conecta al clúster de Microsoft SQL Server en un nivel de back-end a través de un equilibrador de carga de Azure. Estas instancias de back-end de SQL Server están en una red virtual de Azure independiente, protegidas mediante reglas de grupo de seguridad de red que limitan el flujo de tráfico.
4. El equilibrador de carga distribuye el tráfico de SQL Server en instancias de máquinas virtuales de otro conjunto de escalado de máquinas virtuales.
5. Azure Blob Storage actúa como un testigo en la nube para el clúster de SQL Server en el nivel de back-end.  La conexión desde dentro de la red virtual se habilita con un punto de conexión de servicio de red virtual para Azure Storage.

### <a name="components"></a>Componentes

* [Azure Application Gateway][appgateway-docs] es un equilibrador de carga de tráfico web de capa 7 que es compatible con aplicaciones y puede distribuir el tráfico según reglas de enrutamiento específicas. App Gateway puede también controlar la descarga de SSL para obtener un rendimiento de servidor web mejorado.
* [Azure Virtual Network][vnet-docs] permite que varios recursos como, por ejemplo, máquinas virtuales, se puedan comunicar de forma segura entre ellos, con Internet y con redes locales. Las redes virtuales proporcionan aislamiento y segmentación, filtrado y enrutamiento de tráfico, y permiten la conexión entre ubicaciones. Se usan dos redes virtuales combinadas con los grupos de seguridad de red adecuados en este escenario para proporcionar una [red perimetral][dmz] (DMZ) y aislamiento para los componentes de la aplicación. El emparejamiento de redes virtuales conecta las dos redes entre sí.
* El [conjunto de escalado de máquinas virtuales de Azure][scaleset-docs] permite crear y administrar un grupo de máquinas virtuales idénticas con equilibrio de carga. El número de instancias de máquina virtual puede aumentar o disminuir automáticamente según la demanda, o de acuerdo a una programación definida. Se usan dos conjuntos de escalado de máquinas virtuales en este escenario, uno para las instancias de la aplicación ASP.NET de front-end y otro para las instancias de máquinas virtuales de clústeres de SQL Server de back-end. Se puede usar Desired State Configuration (DSC) de PowerShell o la extensión de script personalizada de Azure para aprovisionar las instancias de máquinas virtuales con el software y los valores de configuración requeridos.
* Los [grupos de seguridad de red de Azure][nsg-docs] contienen una lista de reglas de seguridad que permiten o deniegan el tráfico entrante o saliente en función de las direcciones IP de origen o destino, el puerto y el protocolo. Las redes virtuales de este escenario se protegen con reglas de grupo de seguridad de red que restringen el flujo de tráfico entre los componentes de la aplicación.
* [Azure Load Balancer][loadbalancer-docs] distribuye el tráfico entrante según reglas y sondeos de mantenimiento. Una instancia de Load Balancer proporciona baja latencia y alto rendimiento, y puede escalar hasta millones de flujos para todas las aplicaciones TCP y UDP. Se usa un equilibrador de carga interno en este escenario para distribuir el tráfico desde la capa de aplicación front-end al clúster de SQL Server de back-end.
* [Azure Blob Storage][cloudwitness-docs] actúa como una ubicación de testigo en la nube para el clúster de SQL Server. Este testigo se usa para las operaciones del clúster y las decisiones que requieren un voto adicional para decidir el cuórum. El uso de un testigo en la nube elimina la necesidad de una máquina virtual adicional para que actúe como un testigo de recurso compartido de archivos tradicional.

### <a name="alternatives"></a>Alternativas

* *nix, Windows se puede reemplazar fácilmente por otros sistemas operativos ya que nada en la infraestructura depende del sistema operativo.

* [SQL Server para Linux][sql-linux] puede reemplazar el almacén de datos de back-end.

* [Cosmos DB][cosmos] es otra alternativa para el almacén de datos.

## <a name="considerations"></a>Consideraciones

### <a name="availability"></a>Disponibilidad

Las instancias de máquinas virtuales en este escenario se implementan en zonas de disponibilidad. Cada zona de disponibilidad consta de uno o varios centros de datos equipados con alimentación, refrigeración y redes independientes. Hay un mínimo de tres zonas disponibles en todas las regiones habilitadas. Esta distribución de instancias de máquinas virtuales en varias zonas proporciona alta disponibilidad para las capas de aplicación. Para más información, consulte [¿Qué son las zonas de disponibilidad en Azure?][azureaz-docs]

El nivel de base de datos se puede configurar para usar grupos de disponibilidad Always On. Con esta configuración de SQL Server, se configura una base de datos principal dentro de un clúster con un máximo de ocho bases de datos secundarias. Si se produce un problema con la base de datos principal, el clúster conmuta por error a una de las bases de datos secundarias, lo cual permite que la aplicación siga estando disponible. Para más información, consulte [Información general de los grupos de disponibilidad AlwaysOn (SQL Server)][sqlalwayson-docs].

Para ver otros temas de disponibilidad, consulte la [lista de comprobación de disponibilidad][availability] que encontrará en Azure Architecture Center.

### <a name="scalability"></a>Escalabilidad

Este escenario utiliza conjuntos de escalado de máquinas virtuales para los componentes de front-end y de back-end. Con los conjuntos de escalado, el número de instancias de máquina virtual que se ejecutan en la capa de aplicación de front-end se puede reducir horizontalmente de forma automática en respuesta a la demanda del cliente, o según una programación definida. Para más información, consulte [Introducción al escalado automático con conjuntos de escalado de máquinas virtuales][vmssautoscale-docs].

Para ver otros temas de escalabilidad, consulte la [lista de comprobación de escalabilidad][scalability] que encontrará en el centro de arquitectura de Azure.

### <a name="security"></a>Seguridad

Los grupos de seguridad de red protegen todo el tráfico de la red virtual hacia la capa de aplicación de front-end. Las reglas limitan el flujo de tráfico de forma que solo las instancias de máquinas virtuales de la capa de aplicación de front-end pueden acceder al nivel de la base de datos de back-end. No se permite ningún tráfico de Internet saliente procedente del nivel de la base de datos. Para reducir la superficie de ataque, no hay ningún puerto de administración remota directa abierto. Para más información, consulte [Grupos de seguridad de red de Azure][nsg-docs].

Para ver instrucciones sobre la implementación de Payment Card Industry Data Security Standard (PCI DSS 3.2), consulte [infraestructura compatible][pci-dss]. Para obtener instrucciones generales sobre el diseño de escenarios seguros, consulte la [documentación de seguridad de Azure][security].

### <a name="resiliency"></a>Resistencia

En combinación con el uso de zonas de disponibilidad y conjuntos de escalado de máquinas virtuales, este escenario utiliza una instancia de Azure Application Gateway y un equilibrador de carga. Estos dos componentes de redes distribuyen el tráfico a las instancias de máquinas virtuales conectadas e incluyen sondeos de estado que garantizan que el tráfico solo se distribuye a las máquinas virtuales correctas. Se configuran dos instancias de Application Gateway en una configuración activa/pasiva, y se usa un equilibrador de carga con redundancia de zona. Esta configuración hace que los recursos y aplicaciones de red sean resistentes frente a problemas que, de lo contrario, interrumpirían el tráfico y afectarían al acceso de los usuarios finales.

Para obtener instrucciones generales sobre el diseño de escenarios resistentes, consulte [Diseño de aplicaciones resistentes de Azure][resiliency].

## <a name="deploy-the-scenario"></a>Implementación del escenario

**Requisitos previos**

* Debe tener una cuenta de Azure. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.
* Para implementar un clúster de SQL Server en el conjunto de escalado de back-end, necesita un dominio en Active Directory Domain Services.

Para implementar la infraestructura principal de este escenario con una plantilla de Azure Resource Manager, realice los pasos siguientes.

1. Seleccione el botón **Implementar en Azure**:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Finfrastructure%2Fregulated-multitier-app%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. Espere a que la implementación de plantilla se abra en Azure Portal y complete los pasos siguientes:
   * Elija **Crear nuevo** para el grupo de recursos y proporcione un nombre como, por ejemplo, *myWindowsscenario*, en el cuadro de texto.
   * Seleccione una región en el cuadro de lista desplegable **Ubicación**.
   * Indique un nombre de usuario y una contraseña segura para las instancias de conjuntos de escalado de máquinas virtuales.
   * Revise los términos y condiciones, y seleccione **Acepto los términos y condiciones indicados anteriormente**.
   * Seleccione el botón **Comprar**.

La implementación puede tardar unos 15-20 minutos en completarse.

## <a name="pricing"></a>Precios

Para explorar el costo de ejecutar este escenario, todos los servicios están preconfigurados en la calculadora de costos.  Para ver cómo cambiarían los precios en su caso concreto, cambie las variables pertinentes para que coincidan con el tráfico esperado.

Hemos incluido tres perfiles de costos de ejemplo basados en el número de instancias de conjuntos de escalado de máquinas virtuales que ejecutan las aplicaciones.

* [Pequeño][small-pricing]: este ejemplo de plan de tarifa corresponde a dos instancias de máquina virtual de front-end y dos de back-end.
* [Mediano][medium-pricing]: este ejemplo de plan de tarifa corresponde a 20 instancias de máquina virtual de front-end y cinco de back-end.
* [Grande][large-pricing]: este ejemplo de plan de tarifa corresponde a 100 instancias de máquina virtual de front-end y 10 de back-end.

## <a name="related-resources"></a>Recursos relacionados

Este escenario usa un conjunto de escalado de máquinas virtuales de back-end que ejecuta un clúster de Microsoft SQL Server. También se puede usar Azure Cosmos DB como un nivel de base de datos escalable y seguro para los datos de aplicación. Un [punto de conexión de servicio de red virtual de Azure][vnetendpoint-docs] le permite proteger los recursos de los servicios de Azure fundamentales y los limita solo a sus redes virtuales. En este escenario, los puntos de conexión le permiten proteger el tráfico entre la capa de aplicación de front-end y Cosmos DB. Para más información sobre Cosmos DB, consulte [Introducción a Azure Cosmos DB][azurecosmosdb-docs].

También puede ver una completa [arquitectura de referencia para una aplicación de n niveles genérica con SQL Server][ntiersql-ra].

<!-- links -->
[appgateway-docs]: /azure/application-gateway/overview
[architecture]: ./media/regulated-multitier-app/architecture-regulated-multitier-app.png
[autoscaling]: /azure/architecture/best-practices/auto-scaling
[availability]: /architecture/checklist/availability
[azureaz-docs]: /azure/availability-zones/az-overview
[azurecosmosdb-docs]: /azure/cosmos-db/introduction
[cloudwitness-docs]: /windows-server/failover-clustering/deploy-cloud-witness
[loadbalancer-docs]: /azure/load-balancer/load-balancer-overview
[nsg-docs]: /azure/virtual-network/security-overview
[ntiersql-ra]: /azure/architecture/reference-architectures/n-tier/n-tier-sql-server
[resiliency]: /azure/architecture/resiliency/ 
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability 
[scaleset-docs]: /azure/virtual-machine-scale-sets/overview
[sqlalwayson-docs]: /sql/database-engine/availability-groups/windows/overview-of-always-on-availability-groups-sql-server
[vmssautoscale-docs]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview
[vnet-docs]: /azure/virtual-network/virtual-networks-overview
[vnetendpoint-docs]: /azure/virtual-network/virtual-network-service-endpoints-overview
[pci-dss]: /azure/security/blueprints/pcidss-iaaswa-overview
[dmz]: /azure/virtual-network/virtual-networks-dmz-nsg
[cosmos]: /azure/cosmos-db/
[sql-linux]: /sql/linux/sql-server-linux-overview?view=sql-server-linux-2017

[small-pricing]: https://azure.com/e/711bbfcbbc884ef8aa91cdf0f2caff72
[medium-pricing]: https://azure.com/e/b622d82d79b34b8398c4bce35477856f
[large-pricing]: https://azure.com/e/1d99d8b92f90496787abecffa1473a93