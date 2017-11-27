---
title: "Patrones de diseño en la nube"
description: "Patrones de diseño en la nube para Microsoft Azure"
keywords: Azure
ms.openlocfilehash: bf9fb2555f5c80cab9e4616ba52155bf1284d26f
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2017
---
# <a name="cloud-design-patterns"></a>Patrones de diseño en la nube

Estos patrones de diseño son útiles para crear aplicaciones confiables, escalables y seguras en la nube.

Cada patrón describe el problema al que hace frente, las consideraciones sobre su aplicación y un ejemplo basado en Microsoft Azure. La mayoría incluye ejemplos de código o fragmentos de código que muestran cómo implementar el patrón en Azure. Sin embargo, la mayoría de los patrones es importante para todos los sistemas distribuidos, tanto si están hospedados en Azure como en otras plataformas en la nube.

## <a name="challenges-in-cloud-development"></a>Desafíos del desarrollo en la nube

<table>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/availability.md"><img src="_images/category/availability.svg" alt="Availability" /></a></td>
    <td>
        <h3><a href="./category/availability.md">Disponibilidad</a></h3>
        <p>La disponibilidad es la proporción de tiempo que el sistema está operativo y en funcionamiento, normalmente se mide como un porcentaje del tiempo de actividad. Los errores del sistema, los problemas de infraestructura, los ataques malintencionados y la carga del sistema pueden afectar a la disponibilidad.  Las aplicaciones en la nube suelen proporcionar a los usuarios un Acuerdo de Nivel de Servicio, por lo que las aplicaciones se deben diseñar para maximizar la disponibilidad.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/data-management.md"><img src="_images/category/data-management.svg" alt="Data Management" /></a></td>
    <td>
        <h3><a href="./category/data-management.md">Administración de datos</a></h3>
        <p>La administración de datos es el elemento clave de las aplicaciones en la nube e influye en la mayoría de los atributos de calidad. Los datos se hospedan normalmente en distintas ubicaciones y entre varios servidores por motivos tales como el rendimiento, la escalabilidad o la disponibilidad, lo cual puede conllevar varios desafíos. Por ejemplo, se debe mantener la coherencia de los datos y estos deben estar sincronizados entre las diferentes ubicaciones.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/design-implementation.md"><img src="_images/category/design-implementation.svg" alt="Design and Implementation" /></a></td>
    <td>
        <h3><a href="./category/design-implementation.md">Diseño e implementación</a></h3>
        <p>Un buen diseño incluye factores como la coherencia en el diseño e implementación de los componentes, el mantenimiento para simplificar la administración y el desarrollo, y la reutilización para permitir que los componentes y subsistemas se puedan utilizar en otras aplicaciones y escenarios. Las decisiones tomadas durante la fase de diseño e implementación tienen un gran impacto en la calidad y el costo total de propiedad de las aplicaciones y servicios hospedados en la nube.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/messaging.md"><img src="_images/category/messaging.svg" alt="Messaging" /></a></td>
    <td>
        <h3><a href="./category/messaging.md">Mensajería</a></h3>
        <p>La naturaleza distribuida de las aplicaciones en la nube requiere una infraestructura de mensajería que permite conectar los componentes y servicios, idealmente mediante un acoplamiento flexible, para maximizar la escalabilidad. La mensajería asincrónica se usa ampliamente y ofrece numerosas ventajas, pero también supone desafíos como la ordenación de los mensajes, la administración de mensajes dudosos, la idempotencia, etc.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/management-monitoring.md"><img src="_images/category/management-monitoring.svg" alt="Management and Monitoring" /></a></td>
    <td>
        <h3><a href="./category/management-monitoring.md">Administración y supervisión</a></h3>
        <p>Las aplicaciones en la nube se ejecutan en un centro de datos remoto en el que no tiene un control completo de la infraestructura ni, en algunos casos, del sistema operativo. Esto puede hacer que la administración y la supervisión sean más complejas que en una implementación local. Las aplicaciones deben exponer la información del entorno de ejecución que los administradores y operadores pueden usar para administrar y supervisar el sistema, así como dar soporte a los cambios en los requisitos empresariales y de personalización sin necesidad de detener o volver a implementar la aplicación.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/performance-scalability.md"><img src="_images/category/performance-scalability.svg" alt="Performance and Scalability" /></a></td>
    <td>
        <h3><a href="./category/performance-scalability.md">Rendimiento y escalabilidad</a></h3>
        <p>El rendimiento es un indicativo de la capacidad de respuesta de un sistema a la hora de ejecutar cualquier acción dentro de un intervalo de tiempo determinado, mientras que la escalabilidad es la capacidad que tiene un sistema para controlar los aumentos de carga sin que afecte al rendimiento o para aumentar los recursos disponibles en el momento adecuado. Las aplicaciones en la nube normalmente se encuentran con cargas de trabajo variables y picos en la actividad. Predecir estos, especialmente en un escenario multiinquilino, es casi imposible. En lugar de eso, las aplicaciones se deben poder escalar horizontalmente dentro de ciertos límites para satisfacer los picos de demanda, y reducir horizontalmente cuando la demanda disminuya. La escalabilidad no se refiere solo a instancias de proceso, sino también a otros elementos como el almacenamiento de datos, la infraestructura de mensajería, etc.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/resiliency.md"><img src="_images/category/resiliency.svg" alt="Resiliency" /></a></td>
    <td>
        <h3><a href="./category/resiliency.md">Resistencia</a></h3>
        <p>La resistencia es la capacidad de un sistema para manejar los errores y recuperarse de ellos satisfactoriamente. La naturaleza del hospedaje en la nube, donde las aplicaciones a menudo son multiinquilino, usan servicios de plataforma compartidos, compiten por los recursos y el ancho de banda, y se ejecutan en hardware estándar, implica que hay una mayor probabilidad de que se produzcan errores transitorios o permanentes. La detección de errores y una recuperación de ellos que sea rápida y eficaz, son necesarias para mantener la resistencia.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/security.md"><img src="_images/category/security.svg" alt="Security" /></a></td>
    <td>
        <h3><a href="./category/security.md">Seguridad</a></h3>
        <p>La seguridad es la capacidad de un sistema para impedir acciones malintencionadas o involuntarias que se salgan del uso para el que fue diseñado, y para impedir la revelación o pérdida de información. Las aplicaciones en la nube están expuestas en Internet, más allá de los límites locales de confianza y, a menudo, están abiertas al público y pueden dar servicio a usuarios que no son de confianza. Se deben diseñar e implementar las aplicaciones de forma que se las proteja frente a ataques malintencionados, se restrinja el acceso solo a los usuarios autorizados y se proteja la información confidencial.</p>
    </td>
</tr>
</table>

## <a name="catalog-of-patterns"></a>Catálogo de patrones

| Patrón | Resumen |
| ------- | ------- |
| [Ambassador](./ambassador.md) | Crea servicios de aplicaciones auxiliares que envíen solicitudes de red en nombre de una aplicación o servicio de consumidor. |
| [Anti-Corruption Layer](./anti-corruption-layer.md) | Implementa una capa de fachada o de adaptador entre una aplicación moderna y un sistema heredado. |
| [Backends for Frontends](./backends-for-frontends.md) | Crea servicios independientes de back-end que determinadas aplicaciones de front-end o interfaces puedan usar. |
| [Bulkhead](./bulkhead.md) | Aísla los elementos de una aplicación en grupos para que si se produce un error en uno, los demás sigan funcionando. |
| [Cache-Aside](./cache-aside.md) | Carga datos a petición en una memoria caché desde un almacén de datos |
| [Circuit Breaker](./circuit-breaker.md) | Maneja errores que pueden tardar una cantidad variable de tiempo en solucionarse durante la conexión a un recurso o servicio remoto. |
| [CQRS](./cqrs.md) | Segrega las operaciones de lectura de datos de las de actualización de datos mediante interfaces independientes. |
| [Compensating Transaction](./compensating-transaction.md) | Deshace el trabajo realizado por una serie de pasos, que conjuntamente definen una operación de coherencia final. |
| [Competing Consumers](./competing-consumers.md) | Permite que varios consumidores simultáneos procesen los mensajes recibidos en el mismo canal de mensajería. |
| [Compute Resource Consolidation](./compute-resource-consolidation.md) | Consolida varias tareas u operaciones en una sola unidad de cálculo. |
| [Event Sourcing](./event-sourcing.md) | Usa un almacén de solo anexar para registrar la serie completa de eventos que describen las acciones realizadas en los datos de un dominio. |
| [External Configuration Store](./external-configuration-store.md) | Mueve la información de configuración fuera del paquete de implementación de la aplicación a una ubicación centralizada. |
| [Federated Identity](./federated-identity.md) | La autenticación se delega a un proveedor de identidad externo. |
| [Gatekeeper](./gatekeeper.md) | Protege aplicaciones y servicios mediante una instancia de host dedicada que actúa como agente entre los clientes y la aplicación o servicio, valida y sanea las solicitudes, y pasa las solicitudes y datos entre ellos. |
| [Gateway Aggregation](./gateway-aggregation.md) | Usa una puerta de enlace para agregar varias solicitudes individuales en una sola solicitud. |
| [Gateway Offloading](./gateway-offloading.md) | Descarga una funcionalidad compartida o especializada en un proxy de puerta de enlace. |
| [Gateway Routing](./gateway-routing.md) | Enruta las solicitudes a varios servicios mediante un solo punto de conexión. |
| [Health Endpoint Monitoring](./health-endpoint-monitoring.md) | Implementa comprobaciones funcionales en una aplicación a la que pueden acceder herramientas externas a través de los puntos de conexión expuestos en intervalos regulares. |
| [Index Table](./index-table.md) | Crea índices en los campos de los almacenes de datos a los que suelen hacer referencia las consultas. |
| [Leader Election](./leader-election.md) | Coordina las acciones realizadas por una colección de instancias de tareas de colaboración de una aplicación distribuida mediante la elección de una instancia como líder que asume la responsabilidad de administrar las demás instancias. |
| [Materialized View](./materialized-view.md) | Genera vistas rellenadas previamente de los datos en uno o más almacenes de datos cuando los datos no tienen el formato idóneo para las operaciones de consulta requeridas. |
| [Pipes and Filters](./pipes-and-filters.md) | Desglosa una tarea que realiza un procesamiento complejo en una serie de elementos independientes que se pueden volver a utilizar. |
| [Priority Queue](./priority-queue.md) | Clasifica por orden de prioridad las solicitudes enviadas a los servicios para que aquellas con una prioridad más alta se reciban y procesen más rápidamente que las que tienen una prioridad más baja. |
| [Queue-Based Load Leveling](./queue-based-load-leveling.md) | Use una cola que actúe como búfer entre una tarea y un servicio que invoca para equilibrar cargas pesadas intermitentes. |
| [Retry](./retry.md) | Permite que una aplicación trate los errores temporales anticipados cuando intenta conectarse a un servicio o un recurso de red, mediante el reintento de forma transparente de una operación que anteriormente produjo error. |
| [Scheduler Agent Supervisor](./scheduler-agent-supervisor.md) | Coordina un conjunto de acciones en un conjunto distribuido de servicios y otros recursos remotos. |
| [Sharding](./sharding.md) | Divide un almacén de datos en un conjunto de particiones horizontales o particiones de base de datos. |
| [Sidecar](./sidecar.md) | Implementa componentes de una aplicación en un proceso o contenedor independiente para proporcionar aislamiento y encapsulación. |
| [Static Content Hosting](./static-content-hosting.md) | Implementa contenido estático en un servicio de almacenamiento basado en la nube que puede enviarlo directamente al cliente. |
| [Strangler](./strangler.md) | Migra de forma incremental un sistema heredado reemplazando gradualmente funciones específicas por los servicios y aplicaciones nuevas. |
| [Limitaciones](./throttling.md) | Controlan el consumo de recursos que usa una instancia de una aplicación, un inquilino individual o un servicio completo. |
| [Valet Key](./valet-key.md) | Usa un token o clave que proporciona a los clientes acceso directo restringido a un recurso o servicio específico. |