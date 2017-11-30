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
# <a name="availability-patterns"></a><span data-ttu-id="44d55-107">Patrones de disponibilidad</span><span class="sxs-lookup"><span data-stu-id="44d55-107">Availability patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="44d55-108">La disponibilidad define la proporción de tiempo que un sistema es funcional y está en funcionamiento.</span><span class="sxs-lookup"><span data-stu-id="44d55-108">Availability defines the proportion of time that the system is functional and working.</span></span> <span data-ttu-id="44d55-109">Los errores del sistema, los problemas de infraestructura, los ataques malintencionados y la carga del sistema pueden afectar a la disponibilidad.</span><span class="sxs-lookup"><span data-stu-id="44d55-109">It will be affected by system errors, infrastructure problems, malicious attacks, and system load.</span></span> <span data-ttu-id="44d55-110">Normalmente se mide como porcentaje del tiempo de actividad.</span><span class="sxs-lookup"><span data-stu-id="44d55-110">It is usually measured as a percentage of uptime.</span></span> <span data-ttu-id="44d55-111">Las aplicaciones en la nube suelen proporcionar a los usuarios un Acuerdo de Nivel de Servicio, lo cual conlleva que las aplicaciones se deben diseñar e implementar de tal forma que se maximice la disponibilidad.</span><span class="sxs-lookup"><span data-stu-id="44d55-111">Cloud applications typically provide users with a service level agreement (SLA), which means that applications must be designed and implemented in a way that maximizes availability.</span></span>

| <span data-ttu-id="44d55-112">Patrón</span><span class="sxs-lookup"><span data-stu-id="44d55-112">Pattern</span></span> | <span data-ttu-id="44d55-113">Resumen</span><span class="sxs-lookup"><span data-stu-id="44d55-113">Summary</span></span> |
| ------- | ------- |
| [<span data-ttu-id="44d55-114">Health Endpoint Monitoring</span><span class="sxs-lookup"><span data-stu-id="44d55-114">Health Endpoint Monitoring</span></span>](../health-endpoint-monitoring.md) | <span data-ttu-id="44d55-115">Implementa comprobaciones funcionales en una aplicación a la que pueden acceder herramientas externas a través de los puntos de conexión expuestos en intervalos regulares.</span><span class="sxs-lookup"><span data-stu-id="44d55-115">Implement functional checks in an application that external tools can access through exposed endpoints at regular intervals.</span></span> |
| [<span data-ttu-id="44d55-116">Queue-Based Load Leveling</span><span class="sxs-lookup"><span data-stu-id="44d55-116">Queue-Based Load Leveling</span></span>](../queue-based-load-leveling.md) | <span data-ttu-id="44d55-117">Use una cola que actúe como búfer entre una tarea y un servicio que invoca para equilibrar cargas pesadas intermitentes.</span><span class="sxs-lookup"><span data-stu-id="44d55-117">Use a queue that acts as a buffer between a task and a service that it invokes in order to smooth intermittent heavy loads.</span></span> |
| [<span data-ttu-id="44d55-118">Limitaciones</span><span class="sxs-lookup"><span data-stu-id="44d55-118">Throttling</span></span>](../throttling.md) | <span data-ttu-id="44d55-119">Controlan el consumo de recursos que usa una instancia de una aplicación, un inquilino individual o un servicio completo.</span><span class="sxs-lookup"><span data-stu-id="44d55-119">Control the consumption of resources used by an instance of an application, an individual tenant, or an entire service.</span></span> |