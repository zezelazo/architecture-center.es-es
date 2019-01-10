---
title: Patrones de mensajería
titleSuffix: Cloud Design Patterns
description: La naturaleza distribuida de las aplicaciones en la nube requiere una infraestructura de mensajería que permite conectar los componentes y servicios, idealmente mediante un acoplamiento flexible, para maximizar la escalabilidad. La mensajería asincrónica se usa ampliamente y ofrece numerosas ventajas, pero también supone desafíos como la ordenación de los mensajes, la administración de mensajes dudosos, la idempotencia, etc.
keywords: Patrón de diseño
author: dragon119
ms.date: 12/07/2018
ms.custom: seodec18
ms.openlocfilehash: 4619d30c152f050f3f95aee3f9983b8fe85911ed
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/04/2019
ms.locfileid: "54011334"
---
# <a name="messaging-patterns"></a>Patrones de mensajería

[!INCLUDE [header](../../_includes/header.md)]

La naturaleza distribuida de las aplicaciones en la nube requiere una infraestructura de mensajería que permite conectar los componentes y servicios, idealmente mediante un acoplamiento flexible, para maximizar la escalabilidad. La mensajería asincrónica se usa ampliamente y ofrece numerosas ventajas, pero también supone desafíos como la ordenación de los mensajes, la administración de mensajes dudosos, la idempotencia, etc.

| Patrón | Resumen |
| ------- | ------- |
| [Competing Consumers](../competing-consumers.md) | Permite que varios consumidores simultáneos procesen los mensajes recibidos en el mismo canal de mensajería. |
| [Pipes and Filters](../pipes-and-filters.md) | Desglosa una tarea que realiza un procesamiento complejo en una serie de elementos independientes que se pueden volver a utilizar. |
| [Priority Queue](../priority-queue.md) | Clasifica por orden de prioridad las solicitudes enviadas a los servicios para que aquellas con una prioridad más alta se reciban y procesen más rápidamente que las que tienen una prioridad más baja. |
| [Publisher-Subscriber](../publisher-subscriber.md) | Permita que una aplicación anuncie eventos de forma asincrónica a varios consumidores interesados, sin necesidad de emparejar los remitentes con los receptores. |
| [Queue-Based Load Leveling](../queue-based-load-leveling.md) | Usa una cola que actúa como búfer entre una tarea y un servicio que invoca para equilibrar cargas pesadas intermitentes. |
| [Scheduler Agent Supervisor](../scheduler-agent-supervisor.md) | Coordina un conjunto de acciones en un conjunto distribuido de servicios y otros recursos remotos. |
