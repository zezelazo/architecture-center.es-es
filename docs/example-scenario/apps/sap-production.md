---
title: Ejecución de cargas de trabajo SAP de producción mediante una base de datos de Oracle en Azure
description: Ejecute una implementación de producción de SAP en Azure mediante una base de datos de Oracle.
author: DharmeshBhagat
ms.date: 9/12/2018
ms.openlocfilehash: 75942b4d9b18b7bbe7a162826bcf4fe9ece22dce
ms.sourcegitcommit: 0a31fad9b68d54e2858314ca5fe6cba6c6b95ae4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/13/2018
ms.locfileid: "51610913"
---
# <a name="running-sap-production-workloads-using-an-oracle-database-on-azure"></a>Ejecución de cargas de trabajo SAP de producción mediante una instancia de Oracle Database en Azure

Los sistemas SAP se usan para ejecutar aplicaciones empresariales críticas. Cualquier interrupción paraliza procesos clave y puede provocar un aumento de los gastos o una pérdida de ingresos. Para evitar estas consecuencias, se necesita una infraestructura SAP que sea resistente y de alta disponibilidad en caso de errores.

A fin de crear un entorno de SAP de alta disponibilidad, es necesario eliminar los únicos puntos de error en la arquitectura y los procesos del sistema. Los únicos puntos de error pueden estar provocados por errores del sitio, errores en los componentes del sistema o incluso errores humanos.

Este escenario de ejemplo muestra una implementación de SAP en máquinas virtuales Windows o Linux en Azure, junto con una base de datos de Oracle de alta disponibilidad (HA). Para la implementación de SAP, puede usar máquinas virtuales de diferentes tamaños según sus requisitos.

## <a name="relevant-use-cases"></a>Casos de uso pertinentes

Otros casos de uso pertinentes incluyen:

* Cargas de trabajo críticas en ejecución en SAP.
* Cargas de trabajo SAP no críticas.
* Entornos de prueba de SAP que simulan un entorno de alta disponibilidad.

## <a name="architecture"></a>Arquitectura

![Introducción a la arquitectura de un entorno de producción de SAP en Azure][architecture]

Este ejemplo incluye una configuración de alta disponibilidad para una base de datos de Oracle, los servicios centrales de SAP y varios servidores de aplicaciones de SAP que se ejecutan en distintas máquinas virtuales. La red de Azure usa una [topología de hub-and-spoke](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke) por motivos de seguridad. Los datos fluyen por la solución de la siguiente manera:

1. Los usuarios acceden al sistema SAP mediante la interfaz de usuario SAP, un explorador web u otras herramientas de cliente como Microsoft Excel. Una conexión de ExpressRoute proporciona acceso desde la red local de la organización a los recursos que se ejecutan en Azure.
2. La conexión de ExpressRoute finaliza en Azure en la puerta de enlace de red virtual (VNet) de ExpressRoute. El tráfico de red se enruta a una subred de puerta de enlace a través de la puerta de enlace de ExpressRoute creada en la red virtual del concentrador.
3. La red virtual del concentrador se empareja con una red virtual de radio. La subred de la capa de aplicación hospeda las máquinas virtuales que ejecutan SAP en un conjunto de disponibilidad.
4. Los servidores de administración de identidades proporcionan servicios de autenticación para la solución.
5. Los administradores del sistema utilizan JumpBox para administrar de forma segura los recursos implementados en Azure.

### <a name="components"></a>Componentes

* Las [redes virtuales](/azure/virtual-network/virtual-networks-overview) se usan en este escenario para crear una topología de tipo hub-and-spoke virtual en Azure.
* Las [máquinas virtuales](/azure/virtual-machines/windows/overview) proporcionan los recursos de proceso para cada nivel de la solución. Cada clúster de máquinas virtuales está configurado como un [conjunto de disponibilidad](/azure/virtual-machines/windows/regions-and-availability#availability-sets).
* [ExpressRoute](/azure/expressroute/expressroute-introduction) amplía su red local en la nube de Microsoft mediante una conexión privada que establece un proveedor de conectividad.
* Los [grupos de seguridad de red](/azure/virtual-network/security-overview) limita el acceso a los recursos en una red virtual. Un grupo de seguridad de red contiene una lista de reglas de seguridad que permiten o deniegan el tráfico de red en función de las direcciones IP de origen o destino, el puerto y el protocolo. 
* Los [grupos de recursos](/azure/azure-resource-manager/resource-group-overview#resource-groups) actúan como contenedores lógicos de recursos de Azure.

### <a name="alternatives"></a>Alternativas

SAP ofrece opciones flexibles para diferentes combinaciones de sistema operativo, sistema de administración de bases de datos y tipos de máquinas virtuales en un entorno de Azure. Para más información, consulte [SAP note 1928533](https://launchpad.support.sap.com/#/notes/1928533) "SAP Applications on Azure: Supported Products and Azure VM Types" (Nota de SAP 1928533: Aplicaciones de SAP en Azure: tipos de máquina virtual de Azure y productos compatibles).

## <a name="considerations"></a>Consideraciones

Hay definidos procedimientos recomendados para la creación de entornos de SAP de alta disponibilidad en Azure. Para más información, consulte [Escenarios y arquitectura de alta disponibilidad para SAP NetWeaver](/azure/virtual-machines/workloads/sap/sap-high-availability-architecture-scenarios).
Para más información, consulte [Alta disponibilidad de aplicaciones de SAP en máquinas virtuales de Azure](/azure/virtual-machines/workloads/sap/high-availability-guide).
* Las bases de datos de Oracle también tienen procedimientos recomendados para Azure. Para más información, consulte [Diseño e implementación de una base de datos de Oracle en Azure](/azure/virtual-machines/workloads/oracle/oracle-design). 
* Oracle Data Guard se usa para eliminar únicos puntos de error de bases de datos de Oracle críticas. Para más información, consulte [Implementación de Oracle Data Guard en una máquina virtual Linux en Azure](/azure/virtual-machines/workloads/oracle/configure-oracle-dataguard).

Microsoft Azure ofrece servicios de infraestructura que pueden usarse para implementar productos de SAP con una base de datos de Oracle. Para más información, consulte [Implementación de Oracle DBMS en Azure para una carga de trabajo de SAP](/azure/virtual-machines/workloads/sap/dbms_guide_oracle).

## <a name="pricing"></a>Precios

Para ayudarle a explorar el costo de ejecución de este escenario, todos los servicios están preconfigurados en los siguientes ejemplos de la calculadora de costos. Para ver cómo cambiarían los precios en su caso concreto, cambie las variables pertinentes para que coincidan con el tráfico esperado.

Se proporcionan cuatro ejemplos de perfiles de costo según la cantidad de tráfico que se espera recibir:

|Tamaño|SAP|Tipo de MV de BD|Almacenamiento de BD|MV de (A)SCS|Almacenamiento de (A)SCS|Tipo de MV de aplicaciones|Almacenamiento de aplicaciones|Calculadora de precios de Azure|
|----|----|-------|-------|-----|---|---|--------|---------------|
|Pequeña|30000|DS13_v2|4xP20, 1xP20|DS11_v2|1x P10|DS13_v2|1x P10|[Pequeño](https://azure.com/e/45880ba0bfdf47d497851a7cf2650c7c)|
|Mediano|70000|DS14_v2|6xP20, 1xP20|DS11_v2|1x P10|4x DS13_v2|1x P10|[Mediano](https://azure.com/e/9a523f79591347ca9a48c3aaa1406f8a)|
grande|180000|E32s_v3|5xP30, 1xP20|DS11_v2|1x P10|6x DS14_v2|1x P10|[Grande](https://azure.com/e/f70fccf571e948c4b37d4fecc07cbf42)|
Extragrande|250000|M64s|6xP30, 1xP30|DS11_v2|1x P10|10x DS14_v2|1x P10|[Extragrande](https://azure.com/e/58c636922cf94faf9650f583ff35e97b)|

> [!NOTE]
> Estos precios son una guía que solo indica los costos de las máquinas virtuales y el almacenamiento. Se excluyen las redes, el almacenamiento de copia de seguridad y los cargos de entrada y salida de datos.

* [Pequeño](https://azure.com/e/45880ba0bfdf47d497851a7cf2650c7c): un sistema pequeño que consta de una máquina virtual del tipo DS13_v2 para el servidor de bases de datos con 8 vCPU, 56 GB de RAM y almacenamiento temporal de 112 GB, además de cinco discos de almacenamiento premium de 512 GB. Un servidor de instancia central de SAP que usa una máquina virtual del tipo DS11_v2 con 2 vCPU, 14 GB de RAM y almacenamiento temporal de 28 GB. Una única máquina virtual del tipo DS13_v2 para el servidor de aplicaciones de SAP con 8 vCPU, 56 GB de RAM y almacenamiento temporal de 400 GB, además de un disco de almacenamiento premium de 128 GB.

* [Mediano](https://azure.com/e/9a523f79591347ca9a48c3aaa1406f8a): un sistema mediano que consta de una máquina virtual del tipo DS14_v2 para el servidor de bases de datos con 16 vCPU, 112 GB de RAM y almacenamiento temporal de 800 GB, además de siete discos de almacenamiento premium de 512 GB. Un servidor de instancia central de SAP que usa una máquina virtual del tipo DS11_v2 con 2 vCPU, 14 GB de RAM y almacenamiento temporal de 28 GB. Cuatro máquinas virtuales del tipo DS13_v2 para el servidor de aplicaciones de SAP con 8 vCPU, 56 GB de RAM y almacenamiento temporal de 400 GB, además de un disco de almacenamiento premium de 128 GB.

* [Grande](https://azure.com/e/f70fccf571e948c4b37d4fecc07cbf42): un sistema grande que consta de una máquina virtual del tipo E32s_v3 para el servidor de bases de datos con 32 vCPU, 256 GB de RAM y almacenamiento temporal de 800 GB, además de tres discos de almacenamiento premium de 512 GB y uno de 128 GB. Un servidor de instancia central de SAP que usa una máquina virtual del tipo DS11_v2 con 2 vCPU, 14 GB de RAM y almacenamiento temporal de 28 GB. Seis máquinas virtuales del tipo DS14_v2 para el servidor de aplicaciones de SAP con 16 vCPU, 112 GB de RAM y almacenamiento temporal de 224 GB, además de seis discos de almacenamiento premium de 128 GB.

* [Extragrande](https://azure.com/e/58c636922cf94faf9650f583ff35e97b): un sistema extragrande que consta de una máquina virtual del tipo M64s para el servidor de bases de datos con 64 vCPU, 1024 GB de RAM y almacenamiento temporal de 2000 GB, además de siete discos de almacenamiento premium de 1024 GB. Un servidor de instancia central de SAP que usa una máquina virtual del tipo DS11_v2 con 2 vCPU, 14 GB de RAM y almacenamiento temporal de 28 GB. 10 máquinas virtuales del tipo DS14_v2 para el servidor de aplicaciones de SAP con 16 vCPU, 112 GB de RAM y almacenamiento temporal de 224 GB, además de 10 discos de almacenamiento premium de 128 GB.

## <a name="deployment"></a>Implementación

Use el vínculo siguiente para implementar la infraestructura subyacente para este escenario.

<a
href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fsap-3tier-distributed-ora%2Fazuredeploy.json" target="_blank">
    <img src="https://azuredeploy.net/deploybutton.png"/>
</a>

> [!NOTE]
> SAP y Oracle no se instalan durante esta implementación. Debe implementar estos componentes por separado.

## <a name="related-resources"></a>Recursos relacionados

Para más información sobre la ejecución de cargas de trabajo de producción de SAP en Azure, revise las arquitecturas de referencia siguientes:
* [SAP NetWeaver para AnyDB](/azure/architecture/reference-architectures/sap/sap-netweaver) 
* [SAP S/4HANA](/azure/architecture/reference-architectures/sap/sap-s4hana)
* [Instancias grandes de SAP HANA](/azure/architecture/reference-architectures/sap/hana-large-instances)

<!-- links -->
[architecture]: media/architecture-sap-production.png
