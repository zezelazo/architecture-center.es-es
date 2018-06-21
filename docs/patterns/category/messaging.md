---
title: Patrones de mensajería
description: La naturaleza distribuida de las aplicaciones en la nube requiere una infraestructura de mensajería que permite conectar los componentes y servicios, idealmente mediante un acoplamiento flexible, para maximizar la escalabilidad. La mensajería asincrónica se usa ampliamente y ofrece numerosas ventajas, pero también supone desafíos como la ordenación de los mensajes, la administración de mensajes dudosos, la idempotencia, etc.
keywords: Patrón de diseño
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: 8bf37903df3a6eb23f1581e0405358a7aee61f79
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2018
ms.locfileid: "30848587"
---
# <a name="messaging-patterns"></a><span data-ttu-id="8e623-105">Patrones de mensajería</span><span class="sxs-lookup"><span data-stu-id="8e623-105">Messaging patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="8e623-106">La naturaleza distribuida de las aplicaciones en la nube requiere una infraestructura de mensajería que permite conectar los componentes y servicios, idealmente mediante un acoplamiento flexible, para maximizar la escalabilidad.</span><span class="sxs-lookup"><span data-stu-id="8e623-106">The distributed nature of cloud applications requires a messaging infrastructure that connects the components and services, ideally in a loosely coupled manner in order to maximize scalability.</span></span> <span data-ttu-id="8e623-107">La mensajería asincrónica se usa ampliamente y ofrece numerosas ventajas, pero también supone desafíos como la ordenación de los mensajes, la administración de mensajes dudosos, la idempotencia, etc.</span><span class="sxs-lookup"><span data-stu-id="8e623-107">Asynchronous messaging is widely used, and provides many benefits, but also brings challenges such as the ordering of messages, poison message management, idempotency, and more.</span></span>


|                            <span data-ttu-id="8e623-108">Patrón</span><span class="sxs-lookup"><span data-stu-id="8e623-108">Pattern</span></span>                             |                                                                        <span data-ttu-id="8e623-109">Resumen</span><span class="sxs-lookup"><span data-stu-id="8e623-109">Summary</span></span>                                                                         |
|----------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------|
|        [<span data-ttu-id="8e623-110">Competing Consumers</span><span class="sxs-lookup"><span data-stu-id="8e623-110">Competing Consumers</span></span>](../competing-consumers.md)        |                            <span data-ttu-id="8e623-111">Permite que varios consumidores simultáneos procesen los mensajes recibidos en el mismo canal de mensajería.</span><span class="sxs-lookup"><span data-stu-id="8e623-111">Enable multiple concurrent consumers to process messages received on the same messaging channel.</span></span>                            |
|          [<span data-ttu-id="8e623-112">Pipes and Filters</span><span class="sxs-lookup"><span data-stu-id="8e623-112">Pipes and Filters</span></span>](../pipes-and-filters.md)          |                       <span data-ttu-id="8e623-113">Desglosa una tarea que realiza un procesamiento complejo en una serie de elementos independientes que se pueden volver a utilizar.</span><span class="sxs-lookup"><span data-stu-id="8e623-113">Break down a task that performs complex processing into a series of separate elements that can be reused.</span></span>                        |
|             [<span data-ttu-id="8e623-114">Priority Queue</span><span class="sxs-lookup"><span data-stu-id="8e623-114">Priority Queue</span></span>](../priority-queue.md)             | <span data-ttu-id="8e623-115">Clasifica por orden de prioridad las solicitudes enviadas a los servicios para que aquellas con una prioridad más alta se reciban y procesen más rápidamente que las que tienen una prioridad más baja.</span><span class="sxs-lookup"><span data-stu-id="8e623-115">Prioritize requests sent to services so that requests with a higher priority are received and processed more quickly than those with a lower priority.</span></span> |
|  [<span data-ttu-id="8e623-116">Queue-Based Load Leveling</span><span class="sxs-lookup"><span data-stu-id="8e623-116">Queue-Based Load Leveling</span></span>](../queue-based-load-leveling.md)  |              <span data-ttu-id="8e623-117">Use una cola que actúe como búfer entre una tarea y un servicio que invoca para equilibrar cargas pesadas intermitentes.</span><span class="sxs-lookup"><span data-stu-id="8e623-117">Use a queue that acts as a buffer between a task and a service that it invokes in order to smooth intermittent heavy loads.</span></span>               |
| [<span data-ttu-id="8e623-118">Scheduler Agent Supervisor</span><span class="sxs-lookup"><span data-stu-id="8e623-118">Scheduler Agent Supervisor</span></span>](../scheduler-agent-supervisor.md) |                              <span data-ttu-id="8e623-119">Coordina un conjunto de acciones en un conjunto distribuido de servicios y otros recursos remotos.</span><span class="sxs-lookup"><span data-stu-id="8e623-119">Coordinate a set of actions across a distributed set of services and other remote resources.</span></span>                              |

