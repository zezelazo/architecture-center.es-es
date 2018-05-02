---
title: Árbol de decisión para los servicios de proceso de Azure
description: Un diagrama de flujo para la selección de un servicio de proceso
author: MikeWasson
ms.date: 04/21/2018
ms.openlocfilehash: f5703f4906ca2ea6f825b383710eb4bd335f5043
ms.sourcegitcommit: d702b4d27e96e7a5a248dc4f2f0e25cf6e82c134
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/23/2018
---
# <a name="decision-tree-for-azure-compute-services"></a>Árbol de decisión para los servicios de proceso de Azure

Azure ofrece una serie de formas de hospedar el código de aplicación. El término *proceso* hace referencia al modelo de hospedaje para los recursos informáticos donde se ejecutan las aplicaciones. El diagrama de flujo siguiente le ayudará a elegir un servicio de proceso para la aplicación.
 
![](../images/compute-decision-tree.svg)

El diagrama de flujo le guía a través de un conjunto de criterios de decisión clave para alcanzar una recomendación. Cada aplicación tiene requisitos únicos, por lo que debe tratar la recomendación como un punto inicial. A partir de ahí realice un análisis más detallado, examinando aspectos como:
 
- Conjunto de características
- [Límites de servicio](/azure/azure-subscription-service-limits)
- [Costee](https://azure.microsoft.com/pricing/)
- [Acuerdo de Nivel de Servicio](https://azure.microsoft.com/support/legal/sla/)
- [Disponibilidad regional](https://azure.microsoft.com/global-infrastructure/services/)
- Ecosistema del desarrollador y habilidades del equipo
- [Tablas de comparación de procesos](./compute-comparison.md)

Si la aplicación consta de varias cargas de trabajo, evalúe cada carga de trabajo por separado. Una solución completa puede incluir dos o más servicios de proceso.

