---
title: "Patrón Backends for Frontends"
description: Crea servicios independientes de back-end que determinadas aplicaciones de front-end o interfaces puedan usar.
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: 87acd39d021c5e44594a2e7c9574e4dd363ce83b
ms.sourcegitcommit: c93f1b210b3deff17cc969fb66133bc6399cfd10
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/05/2018
---
# <a name="backends-for-frontends-pattern"></a>Patrón Backends for Frontends

Crea servicios independientes de back-end que determinadas aplicaciones de front-end o interfaces puedan usar. Este patrón es útil cuando desea evitar personalizar un único back-end para varias interfaces. Sam Newman describió por primera vez este patrón.

## <a name="context-and-problem"></a>Contexto y problema

Inicialmente se puede destinar una aplicación a una interfaz de usuario web de escritorio. Normalmente, un servicio back-end se desarrolla en paralelo y proporciona las características necesarias para esa interfaz de usuario. A medida que crece la base de usuarios de la aplicación, se desarrolla una aplicación móvil que debe interactuar con el mismo back-end. El servicio back-end se convierte en un back-end de uso general que atiende los requisitos de la interfaz tanto de escritorio como móvil.

Pero las funcionalidades de un dispositivo móvil difieren significativamente de las de un explorador de escritorio, en términos de tamaño de pantalla, rendimiento y limitaciones de pantalla. En consecuencia, los requisitos para un back-end de aplicación móvil difieren de los de la interfaz de usuario web de escritorio. 

Estas diferencias dan lugar a requisitos contrapuestos para el back-end. El back-end requiere cambios significativos y regulares para atender tanto la interfaz de usuario web de escritorio como la aplicación móvil. A menudo, hay equipos independientes de la interfaz que funcionan en cada front-end y hacen que el back-end se convierta un cuello de botella en el proceso de desarrollo. Los requisitos de actualización conflictivos y la necesidad de mantener el servicio en funcionamiento para ambos front-ends pueden suponer dedicar un gran esfuerzo a un único recurso que se puede implementar.

![](./_images/backend-for-frontend.png) 

Como la actividad de desarrollo se centra en el servicio back-end, se puede crear un equipo independiente para administrar y mantener el back-end. En última instancia, esto produce una desconexión entre los equipos de desarrollo de la interfaz y del back-end, lo que supone una carga para el equipo del back-end a la hora de equilibrar los requisitos contrapuestos de los diferentes equipos de la interfaz de usuario. Cuando un equipo de la interfaz requiere cambios en el back-end, esos cambios se deben validar con otros equipos de la interfaz antes de que se pueden integrar en el back-end. 

## <a name="solution"></a>Solución

Cree un back-end por interfaz de usuario. Ajuste el comportamiento y el rendimiento de cada back-end para que se adapte mejor a las necesidades del entorno de front-end, sin preocuparse de que pueda afectar a otras experiencias de front-end.

![](./_images/backend-for-frontend-example.png) 

Como cada back-end es específico de una interfaz, se puede optimizar para esa interfaz. En consecuencia, será más pequeño, menos complejo y es probable que más rápido que un back-end genérico que intenta satisfacer los requisitos de todas las interfaces. Cada equipo de la interfaz tiene autonomía para controlar su propio back-end y no se apoya en un equipo de desarrollo de back-end centralizado. Esto proporciona flexibilidad al equipo de la interfaz en la selección del lenguaje, el ritmo de lanzamiento, la priorización de la carga de trabajo y la integración de características en el back-end.

Para más información, consulte [Pattern: Backends For Frontends](http://samnewman.io/patterns/architectural/bff/) (Patrón: servidores back-end para servidores front-end).

## <a name="issues-and-considerations"></a>Problemas y consideraciones

- Tenga en cuenta cuántos servidores back-end va a implementar.
- Si distintas interfaces (por ejemplo, clientes móviles) van a hacer las mismas solicitudes, calcule si es necesario implementar un back-end para cada interfaz o si será suficiente con un back-end único.
- Es muy probable que se produzca una duplicación de código en los servicios al implementar este patrón.
- Los servicios back-end centrados en front-end solo deben contener una lógica y un comportamientos específicos del cliente. La lógica de negocios general y otras características globales deben administrarse en otra parte de la aplicación.
- Piense en cómo puede reflejarse este patrón en las responsabilidades de un equipo de desarrollo.
- Tenga en cuenta el tiempo que se tardará en implementar este patrón. ¿Incurrirá el esfuerzo de creación de los nuevos backend en una deuda técnica, mientras continúa apoyando el back-end genérico existente?

## <a name="when-to-use-this-pattern"></a>Cuándo usar este patrón

Use este patrón en los siguientes supuestos:

- Un servicio back-end de uso general o compartido debe mantenerse con una sobrecarga de desarrollo importante.
- Desea optimizar el back-end para los requisitos de interfaces de cliente específicas.
- Las personalizaciones se realizan en un back-end de uso general para dar cabida a varias interfaces.
- Un lenguaje de programación alternativo es más adecuado para el back-end de una interfaz de usuario diferente.

Este patrón puede no ser adecuado:

- Cuando las interfaces realizan las mismas solicitudes o similares al back-end.
- Cuando se usa solo una interfaz para interactuar con el back-end.

## <a name="related-guidance"></a>Instrucciones relacionadas

- [Patrón Gateway Aggregation](./gateway-aggregation.md)
- [Patrón Gateway Offloading](./gateway-offloading.md)
- [Patrón Gateway Routing](./gateway-routing.md)


