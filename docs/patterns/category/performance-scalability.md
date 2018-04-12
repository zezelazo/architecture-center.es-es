---
title: Patrones de rendimiento y escalabilidad
description: El rendimiento es un indicativo de la capacidad de respuesta de un sistema a la hora de ejecutar cualquier acción dentro de un intervalo de tiempo determinado, mientras que la escalabilidad es la capacidad que tiene un sistema para controlar los aumentos de carga sin que afecte al rendimiento o para aumentar los recursos disponibles en el momento adecuado. Las aplicaciones en la nube normalmente se encuentran con cargas de trabajo variables y picos en la actividad. Predecirlos, especialmente en un escenario multiinquilino, es casi imposible. En lugar de eso, las aplicaciones se deben poder escalar horizontalmente dentro de ciertos límites para satisfacer los picos de demanda, y reducir horizontalmente cuando la demanda disminuya. La escalabilidad no se refiere solo a instancias de proceso, sino también a otros elementos como el almacenamiento de datos, la infraestructura de mensajería, etc.
keywords: Patrón de diseño
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: e60c7c5779e73925d7eed51b41eb37e5c2ad49ff
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2018
---
# <a name="performance-and-scalability-patterns"></a>Patrones de rendimiento y escalabilidad

[!INCLUDE [header](../../_includes/header.md)]

El rendimiento es un indicativo de la capacidad de respuesta de un sistema a la hora de ejecutar cualquier acción dentro de un intervalo de tiempo determinado, mientras que la escalabilidad es la capacidad que tiene un sistema para controlar los aumentos de carga sin que afecte al rendimiento o para aumentar los recursos disponibles en el momento adecuado. Las aplicaciones en la nube normalmente se encuentran con cargas de trabajo variables y picos en la actividad. Predecirlos, especialmente en un escenario multiinquilino, es casi imposible. En lugar de eso, las aplicaciones se deben poder escalar horizontalmente dentro de ciertos límites para satisfacer los picos de demanda, y reducir horizontalmente cuando la demanda disminuya. La escalabilidad no se refiere solo a instancias de proceso, sino también a otros elementos como el almacenamiento de datos, la infraestructura de mensajería, etc.


|                           Patrón                            |                                                                        Resumen                                                                         |
|--------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------|
|               [Cache-Aside](../cache-aside.md)               |                                                   Carga datos a petición en una memoria caché desde un almacén de datos                                                   |
|                      [CQRS](../cqrs.md)                      |                           Segrega las operaciones de lectura de datos de las de actualización de datos mediante interfaces independientes.                           |
|            [Event Sourcing](../event-sourcing.md)            |                     Usa un almacén de solo anexar para registrar la serie completa de eventos que describen las acciones realizadas en los datos de un dominio.                      |
|               [Index Table](../index-table.md)               |                                Crea índices en los campos de los almacenes de datos a los que suelen hacer referencia las consultas.                                |
|         [Materialized View](../materialized-view.md)         |       Genera vistas rellenadas previamente de los datos en uno o más almacenes de datos cuando los datos no tienen el formato idóneo para las operaciones de consulta requeridas.        |
|            [Priority Queue](../priority-queue.md)            | Clasifica por orden de prioridad las solicitudes enviadas a los servicios para que aquellas con una prioridad más alta se reciban y procesen más rápidamente que las que tienen una prioridad más baja. |
| [Queue-Based Load Leveling](../queue-based-load-leveling.md) |              Use una cola que actúe como búfer entre una tarea y un servicio que invoca para equilibrar cargas pesadas intermitentes.               |
|                  [Sharding](../sharding.md)                  |                                           Divida un almacén de datos en un conjunto de particiones horizontales o particiones de base de datos.                                           |
|    [Static Content Hosting](../static-content-hosting.md)    |                          Implemente contenido estático en un servicio de almacenamiento basado en la nube que pueda entregarlo directamente al cliente.                          |
|                [Limitaciones](../throttling.md)                |                Controlan el consumo de recursos que usa una instancia de una aplicación, un inquilino individual o un servicio completo.                 |

