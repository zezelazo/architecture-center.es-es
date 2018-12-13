---
title: Patrones de mensajería
description: La naturaleza distribuida de las aplicaciones en la nube requiere una infraestructura de mensajería que permite conectar los componentes y servicios, idealmente mediante un acoplamiento flexible, para maximizar la escalabilidad. La mensajería asincrónica se usa ampliamente y ofrece numerosas ventajas, pero también supone desafíos como la ordenación de los mensajes, la administración de mensajes dudosos, la idempotencia, etc.
keywords: Patrón de diseño
author: dragon119
ms.date: 12/07/2018
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: 754e9a7dcad20dc1c4471af00f3f142f18022d62
ms.sourcegitcommit: a0a9981e7586bed8d876a54e055dea1e392118f8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/11/2018
ms.locfileid: "53233887"
---
# <a name="messaging-patterns"></a><span data-ttu-id="b84b1-105">Patrones de mensajería</span><span class="sxs-lookup"><span data-stu-id="b84b1-105">Messaging patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="b84b1-106">La naturaleza distribuida de las aplicaciones en la nube requiere una infraestructura de mensajería que permite conectar los componentes y servicios, idealmente mediante un acoplamiento flexible, para maximizar la escalabilidad.</span><span class="sxs-lookup"><span data-stu-id="b84b1-106">The distributed nature of cloud applications requires a messaging infrastructure that connects the components and services, ideally in a loosely coupled manner in order to maximize scalability.</span></span> <span data-ttu-id="b84b1-107">La mensajería asincrónica se usa ampliamente y ofrece numerosas ventajas, pero también supone desafíos como la ordenación de los mensajes, la administración de mensajes dudosos, la idempotencia, etc.</span><span class="sxs-lookup"><span data-stu-id="b84b1-107">Asynchronous messaging is widely used, and provides many benefits, but also brings challenges such as the ordering of messages, poison message management, idempotency, and more.</span></span>

| <span data-ttu-id="b84b1-108">Patrón</span><span class="sxs-lookup"><span data-stu-id="b84b1-108">Pattern</span></span> | <span data-ttu-id="b84b1-109">Resumen</span><span class="sxs-lookup"><span data-stu-id="b84b1-109">Summary</span></span> |
| ------- | ------- |
| [<span data-ttu-id="b84b1-110">Competing Consumers</span><span class="sxs-lookup"><span data-stu-id="b84b1-110">Competing Consumers</span></span>](../competing-consumers.md) | <span data-ttu-id="b84b1-111">Permite que varios consumidores simultáneos procesen los mensajes recibidos en el mismo canal de mensajería.</span><span class="sxs-lookup"><span data-stu-id="b84b1-111">Enable multiple concurrent consumers to process messages received on the same messaging channel.</span></span> |
| [<span data-ttu-id="b84b1-112">Pipes and Filters</span><span class="sxs-lookup"><span data-stu-id="b84b1-112">Pipes and Filters</span></span>](../pipes-and-filters.md) | <span data-ttu-id="b84b1-113">Desglosa una tarea que realiza un procesamiento complejo en una serie de elementos independientes que se pueden volver a utilizar.</span><span class="sxs-lookup"><span data-stu-id="b84b1-113">Break down a task that performs complex processing into a series of separate elements that can be reused.</span></span> |
| [<span data-ttu-id="b84b1-114">Priority Queue</span><span class="sxs-lookup"><span data-stu-id="b84b1-114">Priority Queue</span></span>](../priority-queue.md) | <span data-ttu-id="b84b1-115">Clasifica por orden de prioridad las solicitudes enviadas a los servicios para que aquellas con una prioridad más alta se reciban y procesen más rápidamente que las que tienen una prioridad más baja.</span><span class="sxs-lookup"><span data-stu-id="b84b1-115">Prioritize requests sent to services so that requests with a higher priority are received and processed more quickly than those with a lower priority.</span></span> |
| [<span data-ttu-id="b84b1-116">Publisher-Subscriber</span><span class="sxs-lookup"><span data-stu-id="b84b1-116">Publisher-Subscriber</span></span>](../publisher-subscriber.md) | <span data-ttu-id="b84b1-117">Permita que una aplicación anuncie eventos de forma asincrónica a varios consumidores interesados, sin necesidad de emparejar los remitentes con los receptores.</span><span class="sxs-lookup"><span data-stu-id="b84b1-117">Enable an application to announce events to multiple interested consumers aynchronously, without coupling the senders to the receivers.</span></span> |
| [<span data-ttu-id="b84b1-118">Queue-Based Load Leveling</span><span class="sxs-lookup"><span data-stu-id="b84b1-118">Queue-Based Load Leveling</span></span>](../queue-based-load-leveling.md) | <span data-ttu-id="b84b1-119">Usa una cola que actúa como búfer entre una tarea y un servicio que invoca para equilibrar cargas pesadas intermitentes.</span><span class="sxs-lookup"><span data-stu-id="b84b1-119">Use a queue that acts as a buffer between a task and a service that it invokes in order to smooth intermittent heavy loads.</span></span> |
| [<span data-ttu-id="b84b1-120">Scheduler Agent Supervisor</span><span class="sxs-lookup"><span data-stu-id="b84b1-120">Scheduler Agent Supervisor</span></span>](../scheduler-agent-supervisor.md) | <span data-ttu-id="b84b1-121">Coordina un conjunto de acciones en un conjunto distribuido de servicios y otros recursos remotos.</span><span class="sxs-lookup"><span data-stu-id="b84b1-121">Coordinate a set of actions across a distributed set of services and other remote resources.</span></span> |
