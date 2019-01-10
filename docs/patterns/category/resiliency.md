---
title: Patrones de resistencia
titleSuffix: Cloud Design Patterns
description: La resistencia es la capacidad de un sistema para manejar los errores y recuperarse de ellos satisfactoriamente. La naturaleza del hospedaje en la nube, donde las aplicaciones a menudo son multiinquilino, usan servicios de plataforma compartidos, compiten por los recursos y el ancho de banda, y se ejecutan en hardware estándar, implica que hay una mayor probabilidad de que se produzcan errores transitorios o permanentes. La detección de errores y una recuperación de ellos que sea rápida y eficaz, son necesarias para mantener la resistencia.
keywords: Patrón de diseño
author: dragon119
ms.date: 06/23/2017
ms.custom: seodec18
ms.openlocfilehash: d478c0fb42e89c6bb5d84b4d077259ff4bf35f16
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/04/2019
ms.locfileid: "54009719"
---
# <a name="resiliency-patterns"></a>Patrones de resistencia

La resistencia es la capacidad de un sistema para manejar los errores y recuperarse de ellos satisfactoriamente. La naturaleza del hospedaje en la nube, donde las aplicaciones a menudo son multiinquilino, usan servicios de plataforma compartidos, compiten por los recursos y el ancho de banda, y se ejecutan en hardware estándar, implica que hay una mayor probabilidad de que se produzcan errores transitorios o permanentes. La detección de errores y una recuperación de ellos que sea rápida y eficaz, son necesarias para mantener la resistencia.

|                            Patrón                             |                                                                                                      Resumen                                                                                                       |
|----------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|                   [Bulkhead](../bulkhead.md)                   |                                                     Aísle los elementos de una aplicación en grupos para que, si se produce un error en uno, los demás sigan funcionando.                                                      |
|            [Circuit Breaker](../circuit-breaker.md)            |                                                  Controla los errores que pueden tardar una cantidad variable de tiempo en solucionarse durante la conexión a un recurso o servicio remoto.                                                   |
|   [Compensating Transaction](../compensating-transaction.md)   |                                                      Deshace el trabajo realizado mediante una serie de pasos, que conjuntamente definen una operación definitivamente coherente.                                                       |
| [Health Endpoint Monitoring](../health-endpoint-monitoring.md) |                                            Implementa comprobaciones funcionales en una aplicación a la que pueden acceder herramientas externas a través de los puntos de conexión expuestos en intervalos regulares.                                            |
|            [Leader Election](../leader-election.md)            | Coordina las acciones realizadas por una colección de instancias de tareas de colaboración de una aplicación distribuida mediante la elección de una instancia como líder que asume la responsabilidad de administrar las demás instancias. |
|  [Queue-Based Load Leveling](../queue-based-load-leveling.md)  |                                            Usa una cola que actúa como búfer entre una tarea y un servicio que invoca para equilibrar cargas pesadas intermitentes.                                             |
|                      [Retry](../retry.md)                      |             Permite que una aplicación trate los errores temporales anticipados cuando intenta conectarse a un servicio o un recurso de red, mediante el reintento de forma transparente de una operación que anteriormente produjo error.             |
| [Scheduler Agent Supervisor](../scheduler-agent-supervisor.md) |                                                            Coordina un conjunto de acciones en un conjunto distribuido de servicios y otros recursos remotos.                                                            |