---
title: Patrones de diseño e implementación
description: Un buen diseño incluye factores como la coherencia en el diseño e implementación de los componentes, el mantenimiento para simplificar la administración y el desarrollo, y la reutilización para permitir que los componentes y subsistemas se puedan utilizar en otras aplicaciones y escenarios. Las decisiones tomadas durante la fase de diseño e implementación tienen un gran impacto en la calidad y el costo total de propiedad de las aplicaciones y servicios hospedados en la nube.
keywords: Patrón de diseño
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: 861445ceeca62e5b1e62fd4cb33924c35e10c0b0
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2018
ms.locfileid: "30847805"
---
# <a name="design-and-implementation-patterns"></a>Patrones de diseño e implementación

Un buen diseño incluye factores como la coherencia en el diseño e implementación de los componentes, el mantenimiento para simplificar la administración y el desarrollo, y la reutilización para permitir que los componentes y subsistemas se puedan utilizar en otras aplicaciones y escenarios. Las decisiones tomadas durante la fase de diseño e implementación tienen un gran impacto en la calidad y el costo total de propiedad de las aplicaciones y servicios hospedados en la nube.


|                                Patrón                                 |                                                                                                      Resumen                                                                                                       |
|------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|                     [Ambassador](../ambassador.md)                     |                                                         Crea servicios de aplicaciones auxiliares que envíen solicitudes de red en nombre de una aplicación o servicio de consumidor.                                                          |
|          [Anti-Corruption Layer](../anti-corruption-layer.md)          |                                                               Implementa una capa de fachada o de adaptador entre una aplicación moderna y un sistema heredado.                                                                |
|         [Backends for Frontends](../backends-for-frontends.md)         |                                                          Crea servicios independientes de back-end que determinadas aplicaciones de front-end o interfaces puedan usar.                                                          |
|                           [CQRS](../cqrs.md)                           |                                                         Segrega las operaciones de lectura de datos de las de actualización de datos mediante interfaces independientes.                                                         |
| [Compute Resource Consolidation](../compute-resource-consolidation.md) |                                                                     Consolida varias tareas u operaciones en una sola unidad de cálculo.                                                                      |
|   [External Configuration Store](../external-configuration-store.md)   |                                                        Extrae la información de configuración del paquete de implementación de la aplicación y la lleva a una ubicación centralizada.                                                         |
|            [Gateway Aggregation](../gateway-aggregation.md)            |                                                                   Usa una puerta de enlace para agregar varias solicitudes individuales en una sola solicitud.                                                                   |
|             [Gateway Offloading](../gateway-offloading.md)             |                                                                      Descarga una funcionalidad compartida o especializada en un proxy de puerta de enlace.                                                                       |
|                [Gateway Routing](../gateway-routing.md)                |                                                                            Enruta las solicitudes a varios servicios mediante un solo punto de conexión.                                                                            |
|                [Leader Election](../leader-election.md)                | Coordina las acciones realizadas por una colección de instancias de tareas de colaboración de una aplicación distribuida mediante la elección de una instancia como líder que asume la responsabilidad de administrar las demás instancias. |
|              [Pipes and Filters](../pipes-and-filters.md)              |                                                     Desglosa una tarea que realiza un procesamiento complejo en una serie de elementos independientes que se pueden volver a utilizar.                                                      |
|                        [Sidecar](../sidecar.md)                        |                                                  Implemente componentes de una aplicación en un proceso o contenedor independientes para proporcionar aislamiento y encapsulación.                                                  |
|         [Static Content Hosting](../static-content-hosting.md)         |                                                        Implementa contenido estático en un servicio de almacenamiento basado en la nube que puede enviarlo directamente al cliente.                                                        |
|                      [Strangler](../strangler.md)                      |                                         Migra de forma incremental un sistema heredado reemplazando gradualmente funciones específicas por los servicios y aplicaciones nuevas.                                          |

