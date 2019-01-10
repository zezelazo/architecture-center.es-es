---
title: Patrones de disponibilidad
titleSuffix: Cloud Design Patterns
description: La disponibilidad define la proporción de tiempo que un sistema es funcional y está en funcionamiento. Los errores del sistema, los problemas de infraestructura, los ataques malintencionados y la carga del sistema pueden afectar a la disponibilidad. Normalmente se mide como porcentaje del tiempo de actividad. Las aplicaciones en la nube suelen proporcionar a los usuarios un Acuerdo de Nivel de Servicio, lo cual conlleva que las aplicaciones se deben diseñar e implementar de tal forma que se maximice la disponibilidad.
keywords: Patrón de diseño
author: dragon119
ms.date: 06/23/2017
ms.custom: seodec18
ms.openlocfilehash: d256ddd39ca3e8a452162b7b75d67f26f7bb22c2
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/04/2019
ms.locfileid: "54009821"
---
# <a name="availability-patterns"></a><span data-ttu-id="84191-107">Patrones de disponibilidad</span><span class="sxs-lookup"><span data-stu-id="84191-107">Availability patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="84191-108">La disponibilidad define la proporción de tiempo que un sistema es funcional y está en funcionamiento.</span><span class="sxs-lookup"><span data-stu-id="84191-108">Availability defines the proportion of time that the system is functional and working.</span></span> <span data-ttu-id="84191-109">Los errores del sistema, los problemas de infraestructura, los ataques malintencionados y la carga del sistema pueden afectar a la disponibilidad.</span><span class="sxs-lookup"><span data-stu-id="84191-109">It will be affected by system errors, infrastructure problems, malicious attacks, and system load.</span></span> <span data-ttu-id="84191-110">Normalmente se mide como porcentaje del tiempo de actividad.</span><span class="sxs-lookup"><span data-stu-id="84191-110">It is usually measured as a percentage of uptime.</span></span> <span data-ttu-id="84191-111">Las aplicaciones en la nube suelen proporcionar a los usuarios un Acuerdo de Nivel de Servicio, lo cual conlleva que las aplicaciones se deben diseñar e implementar de tal forma que se maximice la disponibilidad.</span><span class="sxs-lookup"><span data-stu-id="84191-111">Cloud applications typically provide users with a service level agreement (SLA), which means that applications must be designed and implemented in a way that maximizes availability.</span></span>

|                            <span data-ttu-id="84191-112">Patrón</span><span class="sxs-lookup"><span data-stu-id="84191-112">Pattern</span></span>                             |                                                           <span data-ttu-id="84191-113">Resumen</span><span class="sxs-lookup"><span data-stu-id="84191-113">Summary</span></span>                                                            |
|----------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------|
| [<span data-ttu-id="84191-114">Health Endpoint Monitoring</span><span class="sxs-lookup"><span data-stu-id="84191-114">Health Endpoint Monitoring</span></span>](../health-endpoint-monitoring.md) | <span data-ttu-id="84191-115">Implementa comprobaciones funcionales en una aplicación a la que pueden acceder herramientas externas a través de los puntos de conexión expuestos en intervalos regulares.</span><span class="sxs-lookup"><span data-stu-id="84191-115">Implement functional checks in an application that external tools can access through exposed endpoints at regular intervals.</span></span> |
|  [<span data-ttu-id="84191-116">Queue-Based Load Leveling</span><span class="sxs-lookup"><span data-stu-id="84191-116">Queue-Based Load Leveling</span></span>](../queue-based-load-leveling.md)  | <span data-ttu-id="84191-117">Usa una cola que actúa como búfer entre una tarea y un servicio que invoca para equilibrar cargas pesadas intermitentes.</span><span class="sxs-lookup"><span data-stu-id="84191-117">Use a queue that acts as a buffer between a task and a service that it invokes in order to smooth intermittent heavy loads.</span></span>  |
|                 [<span data-ttu-id="84191-118">Limitaciones</span><span class="sxs-lookup"><span data-stu-id="84191-118">Throttling</span></span>](../throttling.md)                 |   <span data-ttu-id="84191-119">Controlan el consumo de recursos que usa una instancia de una aplicación, un inquilino individual o un servicio completo.</span><span class="sxs-lookup"><span data-stu-id="84191-119">Control the consumption of resources used by an instance of an application, an individual tenant, or an entire service.</span></span>    |
