---
title: Árbol de decisión para los servicios de proceso de Azure
description: Un diagrama de flujo para la selección de un servicio de proceso
author: MikeWasson
ms.date: 10/23/2018
ms.openlocfilehash: 35002b4840b80bcc35b5baf36ec8e414ed8f20be
ms.sourcegitcommit: 2ae794de13c45cf24ad60d4f4dbb193c25944eff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/25/2018
ms.locfileid: "50001905"
---
# <a name="decision-tree-for-azure-compute-services"></a>Árbol de decisión para los servicios de proceso de Azure

Azure ofrece una serie de formas de hospedar el código de aplicación. El término *proceso* hace referencia al modelo de hospedaje para los recursos informáticos donde se ejecutan las aplicaciones. El diagrama de flujo siguiente le ayudará a elegir un servicio de proceso para la aplicación. El diagrama de flujo le guía a través de un conjunto de criterios de decisión clave para alcanzar una recomendación. 

**Tratar este diagrama de flujo como un punto de partida.** Cada aplicación tiene requisitos únicos, por lo que debe usar la recomendación como un punto de partida. A partir de ahí, realice una evaluación más detallada y examine aspectos como:
 
- Conjunto de características
- [Límites de servicio](/azure/azure-subscription-service-limits)
- [Costee](https://azure.microsoft.com/pricing/)
- [Acuerdo de Nivel de Servicio](https://azure.microsoft.com/support/legal/sla/)
- [Disponibilidad regional](https://azure.microsoft.com/global-infrastructure/services/)
- Ecosistema del desarrollador y habilidades del equipo
- [Tablas de comparación de procesos](./compute-comparison.md)

Si la aplicación consta de varias cargas de trabajo, evalúe cada carga de trabajo por separado. Una solución completa puede incluir dos o más servicios de proceso.

Para más información acerca de las opciones de hospedaje de contenedores en Azure, consulte https://azure.microsoft.com/overview/containers/.

## <a name="flowchart"></a>Diagrama de flujo

![](../images/compute-decision-tree.svg)

## <a name="definitions"></a>Definiciones

- **Lift-and-shift** es una estrategia de migración de una carga de trabajo a la nube sin volver a diseñar la aplicación ni realizar cambios en el código. También se denomina *rehospedaje*. Para más información, consulte [Azure Migration Center](https://azure.microsoft.com/migration/).

- **Optimizado para la nube** es una estrategia de migración a la nube mediante la refactorización de una aplicación para aprovechar las funcionalidades y características nativas de la nube.

## <a name="next-steps"></a>Pasos siguientes

Para que ver más criterios hay que tener en cuenta, consulte [Criterios para elegir un servicio de proceso de Azure](./compute-comparison.md).
