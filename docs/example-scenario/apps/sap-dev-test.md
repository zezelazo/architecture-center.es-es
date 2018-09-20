---
title: SAP para cargas de trabajo de desarrollo/pruebas
description: Escenario SAP para un entorno de desarrollo/pruebas
author: AndrewDibbins
ms.date: 7/11/18
ms.openlocfilehash: d0f266e40969cf4782e69041889a686387499722
ms.sourcegitcommit: c49aeef818d7dfe271bc4128b230cfc676f05230
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/11/2018
ms.locfileid: "44389186"
---
# <a name="sap-for-devtest-workloads"></a>SAP para cargas de trabajo de desarrollo/pruebas

En este ejemplo se proporciona orientación sobre cómo ejecutar una implementación de desarrollo/pruebas de SAP NetWeaver en un entorno de Windows o Linux en Azure. La base de datos empleada es AnyDB, el término de SAP para todos los sistemas de administración de bases de datos compatibles (que no sean SAP HANA). Dado que esta arquitectura está diseñada para entornos que no son de producción, se implementa con una sola máquina virtual (VM) y se puede cambiar su tamaño para adaptarse a las necesidades de su organización.

Para casos de uso de producción, revise las arquitecturas de referencia de SAP que están disponibles a continuación:

* [SAP NetWeaver para AnyDB][sap-netweaver]
* [SAP S/4Hana][sap-hana]
* [SAP en Azure (instancias grandes)][sap-large]

## <a name="related-use-cases"></a>Casos de uso relacionados

Tenga en cuenta este escenario para los casos de uso siguientes:

* Cargas de trabajo no productivas de instancias de SAP no importantes (espacio aislado, desarrollo, prueba, control de calidad)
* Cargas de trabajo de SAP Business One no importantes

## <a name="architecture"></a>Arquitectura

![Diagrama](media/sap-2tier/SAP-Infra-2Tier_finalversion.png)

Este escenario trata sobre el aprovisionamiento de una única base de datos del sistema SAP y del servidor de aplicaciones de SAP en una sola máquina virtual. Los datos fluyen a través del escenario como se indica a continuación:

1. Los clientes del nivel de presentación usan la GUI de SAP, u otras interfaces de usuario (Internet Explorer, Excel o cualquier otra aplicación web) de forma local para acceder al sistema SAP basado en Azure.
2. La conectividad se proporciona mediante una conexión de ExpressRoute establecida. La conexión ExpressRoute finaliza en Azure, en la puerta de enlace de ExpressRoute. El tráfico de red se enruta a través de la puerta de enlace de ExpressRoute hacia la subred de puerta de enlace y desde esta a la subred de aplicaciones de nivel spoke (consulte el patrón [hub-and-spoke][hub-spoke]) y a través de una puerta de enlace de seguridad de red hacia la máquina virtual de la aplicación SAP.
3. Los servidores de administración de identidades proporcionan servicios de autenticación.
4. JumpBox proporciona funcionalidades de administración local.

### <a name="components"></a>Componentes

* Un [grupo de recursos](/azure/azure-resource-manager/resource-group-overview#resource-groups) es un contenedor lógico de recursos de Azure.
* [Las redes virtuales](/azure/virtual-network/virtual-networks-overview) son la base de las comunicaciones de red dentro de Azure.
* [Máquina virtual](/azure/virtual-machines/windows/overview): Azure Virtual Machines proporciona una infraestructura bajo demanda, a gran escala, segura y virtualizada con Windows o Linux Server.
* [ExpressRoute](/azure/expressroute/expressroute-introduction) le permite ampliar sus redes locales en la nube de Microsoft a través de una conexión privada que facilita un proveedor de conectividad.
* Los [grupos de seguridad de red](/azure/virtual-network/security-overview) le permiten limitar el tráfico a los recursos de una red virtual. Un grupo de seguridad de red contiene una lista de reglas de seguridad que permiten o deniegan el tráfico de red entrante o saliente en función de las direcciones IP de origen o destino, el puerto y el protocolo. 

## <a name="considerations"></a>Consideraciones

### <a name="availability"></a>Disponibilidad

 Microsoft ofrece un Acuerdo de Nivel de Servicio (SLA) para instancias únicas de máquina virtual. Para más información, consulte el [Acuerdo de Nivel de Servicio de Microsoft Azure para máquinas virtuales](https://azure.microsoft.com/support/legal/sla/virtual-machines)

### <a name="scalability"></a>Escalabilidad

Para obtener instrucciones generales sobre cómo diseñar soluciones escalables, consulte la [lista de comprobación de escalabilidad][scalability] en el centro de arquitectura de Azure.

### <a name="security"></a>Seguridad

Para obtener instrucciones generales sobre el diseño de soluciones seguras, consulte la [documentación de seguridad de Azure][security].

### <a name="resiliency"></a>Resistencia

Para obtener instrucciones generales sobre el diseño de soluciones resistentes, consulte [Diseño de aplicaciones resistentes de Azure][resiliency].

## <a name="pricing"></a>Precios

Para explorar el costo de ejecutar este escenario, todos los servicios están preconfigurados en la calculadora de costos.  Para ver cómo cambiarían los precios en su caso concreto, cambie las variables pertinentes para que coincidan con el tráfico esperado.

Hemos proporcionado cuatro ejemplos de perfiles de costo según la cantidad de tráfico que se espera obtener:

|Tamaño|SAP|Tipo de máquina virtual|Storage|Calculadora de precios de Azure|
|----|----|-------|-------|---------------|
|Pequeña|8000|D8s_v3|2xP20, 1xP10|[Pequeño](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1)|
|Mediano|16000|D16s_v3|3xP20, 1xP10|[Mediano](https://azure.com/e/465bd07047d148baab032b2f461550cd)|
grande|32000|E32s_v3|3xP20, 1xP10|[Grande](https://azure.com/e/ada2e849d68b41c3839cc976000c6931)|
Extragrande|64000|M64s|4xP20, 1xP10|[Extragrande](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef)|

Nota: La información sobre precios es una orientación y solo indica los costos de máquinas virtuales y de almacenamiento (no incluye los gastos de redes, almacenamiento de copia de seguridad y entrada o salida de datos).

* [Pequeño](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1): un sistema pequeño que consta de una máquina virtual del tipo D8s_v3 con 8 vCPU, 32 GB de RAM y almacenamiento temporal de 200 GB, además de dos discos de almacenamiento premium de 512 GB y uno de 128 GB.
* [Mediano](https://azure.com/e/465bd07047d148baab032b2f461550cd): un sistema mediano que consta de una máquina virtual del tipo D16s_v3 con 16 vCPU, 64 GB de RAM y almacenamiento temporal de 400 GB, además de tres discos de almacenamiento premium de 512 GB y uno de 128 GB.
* [Grande](https://azure.com/e/ada2e849d68b41c3839cc976000c6931): un sistema grande que consta de una máquina virtual del tipo E32s_v3 con 32 vCPU, 256 GB de RAM y almacenamiento temporal de 512 GB, además de tres discos de almacenamiento premium de 512 GB y uno de 128 GB.
* [Extragrande](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef): un sistema extra grande que consta de una máquina virtual del tipo M64s con 64 vCPU, 1024 GB de RAM y almacenamiento temporal de 2000 GB, además de cuatro discos de almacenamiento premium de 512 GB y uno de 128 GB.

## <a name="deployment"></a>Implementación

Para implementar una infraestructura subyacente parecida a la del escenario anterior, use el botón Implementar.

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fsap-2tier%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

\* SAP no se instala automáticamente; debe instalarlo manualmente una vez creada la infraestructura.

<!-- links -->
[reference architecture]:  /azure/architecture/reference-architectures/sap
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[sap-netweaver]: /azure/architecture/reference-architectures/sap/sap-netweaver
[sap-hana]: /azure/architecture/reference-architectures/sap/sap-s4hana
[sap-large]: /azure/architecture/reference-architectures/sap/hana-large-instances
[hub-spoke]: /azure/architecture/reference-architectures/hybrid-networking/hub-spoke