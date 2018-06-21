---
title: Patrones de disponibilidad
description: La disponibilidad define la proporción de tiempo que un sistema es funcional y está en funcionamiento. Los errores del sistema, los problemas de infraestructura, los ataques malintencionados y la carga del sistema pueden afectar a la disponibilidad. Normalmente se mide como porcentaje del tiempo de actividad. Las aplicaciones en la nube suelen proporcionar a los usuarios un Acuerdo de Nivel de Servicio, lo cual conlleva que las aplicaciones se deben diseñar e implementar de tal forma que se maximice la disponibilidad.
keywords: Patrón de diseño
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: eac1d192fe83f8501d11dce02d9bec16e939ba55
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2018
ms.locfileid: "30847284"
---
# <a name="availability-patterns"></a>Patrones de disponibilidad

[!INCLUDE [header](../../_includes/header.md)]

La disponibilidad define la proporción de tiempo que un sistema es funcional y está en funcionamiento. Los errores del sistema, los problemas de infraestructura, los ataques malintencionados y la carga del sistema pueden afectar a la disponibilidad. Normalmente se mide como porcentaje del tiempo de actividad. Las aplicaciones en la nube suelen proporcionar a los usuarios un Acuerdo de Nivel de Servicio, lo cual conlleva que las aplicaciones se deben diseñar e implementar de tal forma que se maximice la disponibilidad.


|                            Patrón                             |                                                           Resumen                                                            |
|----------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------|
| [Health Endpoint Monitoring](../health-endpoint-monitoring.md) | Implementa comprobaciones funcionales en una aplicación a la que pueden acceder herramientas externas a través de los puntos de conexión expuestos en intervalos regulares. |
|  [Queue-Based Load Leveling](../queue-based-load-leveling.md)  | Use una cola que actúe como búfer entre una tarea y un servicio que invoca para equilibrar cargas pesadas intermitentes.  |
|                 [Limitaciones](../throttling.md)                 |   Controlan el consumo de recursos que usa una instancia de una aplicación, un inquilino individual o un servicio completo.    |

