---
title: Patrón Anti-Corruption Layer
description: Implementa una capa de fachada o de adaptador entre una aplicación moderna y un sistema heredado.
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: ac898519c9aa0a0aa2301da9f48756db0eb2af7c
ms.sourcegitcommit: f665226cec96ec818ca06ac6c2d83edb23c9f29c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2018
---
# <a name="anti-corruption-layer-pattern"></a>Patrón Anti-Corruption Layer

Implementar una capa de fachada o adaptador entre los diferentes subsistemas que no comparten la misma semántica. Esta capa traduce las solicitudes que un subsistema hace al otro subsistema. Use este patrón para asegurarse de que el diseño de la aplicación no se vea limitado por dependencias de subsistemas externos. Eric Evans describió por primera vez este patrón en *Domain-Driven Design* (Diseño basado en dominios).

## <a name="context-and-problem"></a>Contexto y problema

La mayoría de las aplicaciones dependen de otros sistemas para obtener determinados datos o funcionalidades. Por ejemplo, cuando se migra una aplicación heredada a un sistema moderno, es posible que siga necesitando recursos heredados existentes. Las nuevas características deben poder llamar al sistema heredado. Esto ocurre especialmente con las migraciones graduales, donde diversas características de una aplicación mayor se trasladan a un sistema moderno con el tiempo.

A menudo estos sistemas heredados experimentan problemas de calidad, como esquemas de datos convolucionados o API obsoletas. Las características y tecnologías que se usan en sistemas heredados pueden variar ampliamente respecto a las empleadas en sistemas más modernos. Para interoperar con el sistema heredado, puede que la nueva aplicación necesite admitir elementos no actualizados, como una infraestructura, protocolos, modelos de datos, API u otras características, que de lo contrario no incluiría en una aplicación moderna.

Mantener el acceso entre sistemas nuevos y heredados puede forzar al nuevo sistema a ajustarse a al menos algunas de las API o a una semántica distinta del sistema heredado. Cuando estas características heredadas tienen problemas de calidad, al admitirlas se "corrompe" lo que, en caso contrario, podría ser una aplicación moderna con un diseño inmaculado. 

Pueden surgir problemas similares con cualquier sistema externo que su equipo de desarrollo no controle, no solo con sistemas heredados. 

## <a name="solution"></a>Solución

Para aislar los diferentes subsistemas, coloque una capa anticorrupción entre ellos. Esta capa traduce las comunicaciones entre los dos sistemas, lo que permite a un sistema permanecer inalterado mientras que el otro evita poner el riesgo su diseño y su enfoque tecnológico.

![](./_images/anti-corruption-layer.png) 

El diagrama anterior muestra una aplicación con dos subsistemas. El subsistema A llama al subsistema B a través de una capa anticorrupción. La comunicación entre el subsistema A y la capa anticorrupción siempre usa el modelo de datos y la arquitectura del subsistema A. Las llamadas desde la capa anticorrupción al subsistema B se ajusta al modelo de datos o los métodos de ese subsistema. La capa para evitar daños contiene toda la lógica necesaria para traducir entre los dos sistemas. La capa puede implementarse como un componente dentro de la aplicación o como un servicio independiente.

## <a name="issues-and-considerations"></a>Problemas y consideraciones

- La capa para evitar daños puede añadir latencia a las llamadas entre los dos sistemas.
- La capa para evitar daños añade un servicio adicional que debe administrarse y mantenerse.
- Tenga en cuenta cómo se escalará la capa para evitar daños.
- Sopese si va a necesitar más de una capa para evitar daños. Es posible que desee descomponer la funcionalidad en varios servicios con diferentes tecnologías o lenguajes o que, por otros motivos, desee crear particiones de la capa para evitar daños.
- Tenga en cuenta cómo se administrará la capa para evitar daños en relación con sus otras aplicaciones o servicios. ¿Cómo se integrará en sus procesos de supervisión, lanzamiento y configuración?
- Asegúrese de mantener la coherencia de la transacción y los datos y de que se pueda supervisar.
- Tenga en cuenta si será necesario que la capa anticorrupción administre todas las comunicaciones entre los diferentes subsistemas o solo un subconjunto de características. 
- Si la capa anticorrupción es parte de una estrategia de migración de aplicaciones, considere si será permanente o se retirará después de migrar toda la funcionalidad heredada.

## <a name="when-to-use-this-pattern"></a>Cuándo usar este patrón

Use este patrón en los siguientes supuestos:

- Se haya planificado una migración en varias fases, pero deba mantenerse la integración entre el sistema nuevo y el heredado.
- Dos o más subsistemas usan una semántica diferente, pero necesitan comunicarse. 

Este patrón puede no ser adecuado si no hay ninguna diferencia semántica importante entre el sistema nuevo y el heredado. 

## <a name="related-guidance"></a>Instrucciones relacionadas

- [Patrón Sustitución](./strangler.md)
