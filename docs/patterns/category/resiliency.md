---
title: Patrones de resistencia
description: La resistencia es la capacidad de un sistema para manejar los errores y recuperarse de ellos satisfactoriamente. La naturaleza del hospedaje en la nube, donde las aplicaciones a menudo son multiinquilino, usan servicios de plataforma compartidos, compiten por los recursos y el ancho de banda, y se ejecutan en hardware estándar, implica que hay una mayor probabilidad de que se produzcan errores transitorios o permanentes. La detección de errores y una recuperación de ellos que sea rápida y eficaz, son necesarias para mantener la resistencia.
keywords: Patrón de diseño
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: 5f5f9c6a23005b1b7ecfc75183f7730823de922e
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2018
ms.locfileid: "30847199"
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
|  [Queue-Based Load Leveling](../queue-based-load-leveling.md)  |                                            Use una cola que actúe como búfer entre una tarea y un servicio que invoca para equilibrar cargas pesadas intermitentes.                                             |
|                      [Retry](../retry.md)                      |             Permite que una aplicación trate los errores temporales anticipados cuando intenta conectarse a un servicio o un recurso de red, mediante el reintento de forma transparente de una operación que anteriormente produjo error.             |
| [Scheduler Agent Supervisor](../scheduler-agent-supervisor.md) |                                                            Coordina un conjunto de acciones en un conjunto distribuido de servicios y otros recursos remotos.                                                            |

