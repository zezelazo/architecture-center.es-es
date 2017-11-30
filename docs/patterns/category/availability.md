---
title: Patrones de disponibilidad
description: "La disponibilidad define la proporción de tiempo que un sistema es funcional y está en funcionamiento. Los errores del sistema, los problemas de infraestructura, los ataques malintencionados y la carga del sistema pueden afectar a la disponibilidad. Normalmente se mide como porcentaje del tiempo de actividad. Las aplicaciones en la nube suelen proporcionar a los usuarios un Acuerdo de Nivel de Servicio, lo cual conlleva que las aplicaciones se deben diseñar e implementar de tal forma que se maximice la disponibilidad."
keywords: "Patrón de diseño"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: f7eb6b0df388b2f1dab83e64ab540cc22f368e19
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2017
---
# <a name="availability-patterns"></a>Patrones de disponibilidad

[!INCLUDE [header](../../_includes/header.md)]

La disponibilidad define la proporción de tiempo que un sistema es funcional y está en funcionamiento. Los errores del sistema, los problemas de infraestructura, los ataques malintencionados y la carga del sistema pueden afectar a la disponibilidad. Normalmente se mide como porcentaje del tiempo de actividad. Las aplicaciones en la nube suelen proporcionar a los usuarios un Acuerdo de Nivel de Servicio, lo cual conlleva que las aplicaciones se deben diseñar e implementar de tal forma que se maximice la disponibilidad.

| Patrón | Resumen |
| ------- | ------- |
| [Health Endpoint Monitoring](../health-endpoint-monitoring.md) | Implementa comprobaciones funcionales en una aplicación a la que pueden acceder herramientas externas a través de los puntos de conexión expuestos en intervalos regulares. |
| [Queue-Based Load Leveling](../queue-based-load-leveling.md) | Use una cola que actúe como búfer entre una tarea y un servicio que invoca para equilibrar cargas pesadas intermitentes. |
| [Limitaciones](../throttling.md) | Controlan el consumo de recursos que usa una instancia de una aplicación, un inquilino individual o un servicio completo. |