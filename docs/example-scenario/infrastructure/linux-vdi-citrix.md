---
title: Escritorios virtuales Linux con Citrix
description: Cree un entorno VDI para escritorios de Linux mediante Citrix en Azure.
author: miguelangelopereira
ms.date: 09/12/2018
ms.openlocfilehash: 383642b05926c5a09abf0b2f95fef10539d95aec
ms.sourcegitcommit: 0a31fad9b68d54e2858314ca5fe6cba6c6b95ae4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/13/2018
ms.locfileid: "51610658"
---
# <a name="linux-virtual-desktops-with-citrix"></a>Escritorios virtuales Linux con Citrix

Este escenario de ejemplo es aplicable a cualquier sector que necesite una infraestructura de escritorio virtual (VDI) para equipos de escritorio Linux. VDI se refiere al proceso de ejecución de un escritorio de usuario en una máquina virtual que se encuentra en un servidor en el centro de datos. El cliente en este escenario ha optado por utilizar una solución basada en Citrix para sus necesidades de VDI.

A menudo, las organizaciones tienen entornos heterogéneos con empleados que usan varios dispositivos y sistemas operativos. Proporcionar acceso constante a las aplicaciones mientras se mantiene un entorno seguro puede resultar un desafío. Una solución de VDI para escritorios Linux permitirá a la organización proporcionar acceso, independientemente del dispositivo o el sistema operativo utilizado por el usuario final.

Algunas de las ventajas de esta solución de ejemplo incluyen lo siguiente:
* El retorno de la inversión será superior con escritorios virtuales Linux compartidos al proporcionar a más usuarios acceso a la misma infraestructura. Mediante la consolidación de recursos en un entorno de VDI centralizado, los dispositivos de usuario final no necesitan ser tan potentes.
* El rendimiento será coherente independientemente del dispositivo del usuario final.
* Los usuarios pueden acceder a las aplicaciones de Linux desde cualquier dispositivo (incluidos los dispositivos que no son Linux).
* Los datos confidenciales se pueden proteger en el centro de datos de Azure para todos los empleados distribuidos.

## <a name="relevant-use-cases"></a>Casos de uso pertinentes

Tenga en cuenta este escenario para el caso de uso siguiente:

* Proporcionar acceso seguro a escritorios Linux VDI especializados críticos desde dispositivos Linux y que no son Linux

## <a name="architecture"></a>Arquitectura

[![](./media/azure-citrix-sample-diagram.png "Diagrama de arquitectura")](./media/azure-citrix-sample-diagram.png#lightbox)

Este escenario de ejemplo muestra cómo permitir el acceso a los escritorios virtuales Linux desde la red corporativa:

* Se establece un circuito de ExpressRoute entre el entorno local y Azure para una conectividad rápida y confiable a la nube.
* La solución XenDesktop de Citrix implementada para VDI.
* CitrixVDA se ejecuta en Ubuntu (u otra distribución admitida).
* Los grupos de seguridad de red de Azure aplicarán las listas de control de acceso de red correctas.
* Citrix ADC (NetScaler) publicará y equilibrará la carga de todos los servicios de Citrix.
* Se usará Active Directory Domain Services para unir el dominio a los servidores Citrix. Los servidores VDA no estarán unidos al dominio.
* Azure File Sync híbrido permitirá el almacenamiento compartido en la solución. Por ejemplo, se puede utilizar en soluciones remotas y domésticas.

En este escenario, se utilizan las SKU siguientes:

- Citrix ADC (NetScaler): 2 x D4sv3 con la [imagen NetScaler 12.0 VPX Standard Edition 200 MBPS PAYG](https://azuremarketplace.microsoft.com/pt-br/marketplace/apps/citrix.netscalervpx-120?tab=PlansAndPrice)
- Servidor de licencias de Citrix: 1 x D2s v3
- Citrix VDA: 4 x D8s v3
- Citrix Storefront: 2 x D2s v3
- Controlador de entrega de Citrix: 2 x D2s v3
- Controladores de dominio: 2 x D2sv3
- Servidores de archivos de Azure: 2 x D2sv3

> [!NOTE]
> Todas las licencias (con la excepción de NetScaler) son de tipo Traiga su propia licencia (BYOL)

### <a name="components"></a>Componentes

- [Azure Virtual Network](/azure/virtual-network/virtual-networks-overview) permite que recursos como, por ejemplo, máquinas virtuales, se puedan comunicar de forma segura entre ellos, con Internet y con redes locales. Las redes virtuales proporcionan aislamiento y segmentación, filtrado y enrutamiento de tráfico, y permiten la conexión entre ubicaciones. Se usará una red virtual para todos los recursos de este escenario.
- Los [grupos de seguridad de red de Azure](/azure/virtual-network/security-overview) contienen una lista de reglas de seguridad que permiten o deniegan el tráfico entrante o saliente en función de las direcciones IP de origen o destino, el puerto y el protocolo. Las redes virtuales de este escenario se protegen con reglas de grupo de seguridad de red que restringen el flujo de tráfico entre los componentes de la aplicación.
- [Azure Load Balancer](/azure/application-gateway/overview) distribuye el tráfico entrante según reglas y sondeos de mantenimiento. Una instancia de Load Balancer proporciona baja latencia y alto rendimiento, y puede escalar hasta millones de flujos para todas las aplicaciones TCP y UDP. En este escenario se usa un equilibrador de carga interno para distribuir el tráfico en Citrix NetScaler.
- Se utilizará [Azure File Sync híbrido](https://github.com/MicrosoftDocs/azure-docs/edit/master/articles/storage/files/storage-sync-files-planning.md) para todo el almacenamiento compartido. El almacenamiento se replicará en dos servidores de archivos que usan File Sync híbrido.
- [Azure SQL Database](/azure/sql-database/sql-database-technical-overview) es un servicio de base de datos relacional (DBaaS) basado en la última versión estable del Motor de base de datos de Microsoft SQL Server. Se usará para hospedar las bases de datos de Citrix.
- [ExpressRoute](/azure/expressroute/expressroute-introduction) permite ampliar las redes locales en la nube de Microsoft a través de una conexión privada que facilita un proveedor de conectividad. 
- [Active Directory Domain Services se utiliza para la autenticación de usuarios y los servicios de directorio
- Los [conjuntos de disponibilidad de Azure](/azure/virtual-machines/windows/tutorial-availability-sets) garantizan que las máquinas virtuales implementadas en Azure se distribuyan entre varios nodos de hardware aislados en un clúster. De este modo, se asegura de que, si se produce un error de hardware o software en Azure, solo un subconjunto de las máquinas virtuales se verá afectado y que la solución seguirá disponible y en funcionamiento. 
- [Citrix ADC (NetScaler)](https://www.citrix.com/products/citrix-adc) es un controlador de entrega de aplicaciones que realiza un análisis de tráfico específico de la aplicación para distribuir, optimizar y proteger de forma inteligente el tráfico de red de la capa 4 a la capa 7 (L4 – L7) para aplicaciones web. 
- [Citrix Storefront](https://www.citrix.com/products/citrix-virtual-apps-and-desktops/citrix-storefront.html) es una tienda de aplicaciones empresariales que mejora la seguridad, simplifica las implementaciones y entrega una moderna experiencia de usuario casi nativa de Citrix Receiver en cualquier plataforma. StoreFront facilita la administración de aplicaciones virtuales de Citrix y entornos de escritorio con varios sitios y varias versiones. 
- El [Servidor de licencias de Citrix](https://www.citrix.com/buy/licensing/overview.html) administrará las licencias de los productos de Citrix.
- [Citrix XenDesktops VDA](https://docs.citrix.com/en-us/citrix-virtual-apps-desktops-service) habilita las conexiones a aplicaciones y escritorios. VDA se instala en la máquina que ejecuta las aplicaciones o escritorios virtuales para el usuario. Permite que las máquinas se registren en los controladores de entrega y administra la conexión de la experiencia de alta definición (HDX) con un dispositivo de usuario.
- El [Controlador de entrega de Citrix](https://docs.citrix.com/en-us/xenapp-and-xendesktop/7-15-ltsr/manage-deployment/delivery-controllers) es el componente de servidor responsable de administrar el acceso de usuario, además de la intermediación y optimización de las conexiones. Los controladores también proporcionan los servicios de creación de máquinas que crean las imágenes de escritorio y del servidor.

### <a name="alternatives"></a>Alternativas

- Existen varios asociados con soluciones de VDI que se admiten en Azure, como VMware, Workspot y otros. Esta arquitectura de ejemplo específica se basa en un proyecto implementado que usa Citrix.
- Citrix ofrece un servicio en la nube que abstrae parte de esta arquitectura. Podría ser una alternativa a esta solución. Para más información, consulte [Citrix Cloud](https://www.citrix.com/products/citrix-cloud).

## <a name="considerations"></a>Consideraciones

- Compruebe los [Requisitos de Linux en Citrix](https://docs.citrix.com/en-us/linux-virtual-delivery-agent/current-release/system-requirements).
- La latencia puede tener impacto en la solución general. Para un entorno de producción, realice pruebas según corresponda.
- Según el escenario, la solución puede necesitar máquinas virtuales con GPU para VDA. Para esta solución, se supone que la GPU no es un requisito.

### <a name="availability-scalability-and-security"></a>Disponibilidad, escalabilidad y seguridad

- Esta solución de ejemplo está diseñada para lograr alta disponibilidad para todos los roles excepto el servidor de licencias. Dado que el entorno sigue funcionando durante un período de gracia de 30 días si el servidor de licencias está sin conexión, no se requiere ninguna redundancia adicional en ese servidor.
- Todos los servidores que proporcionan roles similares se deben implementar en [conjuntos de disponibilidad](/azure/virtual-machines/windows/manage-availability#configure-multiple-virtual-machines-in-an-availability-set-for-redundancy).
- Esta solución de ejemplo no incluye funcionalidades de recuperación ante desastres. [Azure Site Recovery](/azure/site-recovery/site-recovery-overview) podría ser un buen complemento a este diseño.
- Para una implementación de producción, se debe implementar una solución de administración para la [copia de seguridad](/azure/backup/backup-introduction-to-azure-backup), [supervisión](/azure/monitoring-and-diagnostics/monitoring-overview) y [administración de actualizaciones](/azure/automation/automation-update-management).
- Esta solución de ejemplo debería funcionar para aproximadamente 250 usuarios simultáneos (aproximadamente 50-60 por cada servidor de VDA) con un uso mixto. Esto depende en gran medida del tipo de aplicaciones que se van a usar. Para su uso en producción, se deben realizar las pruebas de carga más exigentes.

## <a name="deploy-this-scenario"></a>Implementación de este escenario

Para información sobre la implementación, consulte la [documentación de Citrix](https://docs.citrix.com/en-us/citrix-virtual-apps-desktops/install-configure.html) oficial.

## <a name="pricing"></a>Precios

- Las licencias de Citrix XenDesktop no se incluyen los costos de los servicios de Azure.
- La licencia de Citrix NetScaler se incluye en un modelo de pago por uso.
- El uso de instancias reservadas reducirá notablemente el costo de proceso de la solución.
- No se incluye el costo de ExpressRoute.

## <a name="next-steps"></a>Pasos siguientes

- Compruebe la documentación de Citrix para el planeamiento y la implementación [aquí](https://docs.citrix.com/en-us/citrix-virtual-apps-desktops/install-configure).
- Para implementar Citrix ADC (NetScaler) en Azure, revise las plantillas de Resource Manager suministradas por Citrix [aquí](https://github.com/citrix/netscaler-azure-templates).
