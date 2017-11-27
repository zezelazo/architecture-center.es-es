---
title: Antipatrones de rendimiento para aplicaciones en la nube
description: "Prácticas habituales que pueden provocar problemas de escalabilidad."
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: 423fe6533e57268610f625f523714cd1bce89546
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2017
---
# <a name="performance-antipatterns-for-cloud-applications"></a>Antipatrones de rendimiento para aplicaciones en la nube

Un *antipatrón de rendimiento* es una práctica habitual que puede provocar problemas de escalabilidad cuando una aplicación está bajo presión. 

Este es un escenario común: una aplicación se comporta correctamente durante las pruebas de rendimiento. Se pasa a producción y empieza a tratar cargas de trabajo reales. En ese momento, empieza comportarse mal &mdash;rechazando las solicitudes del usuario, demorándose o generando excepciones. El equipo de desarrollo se enfrenta entonces a dos cuestiones:

- ¿Por qué no se dio este comportamiento durante las pruebas?
- ¿Cómo se corrige esto?

La respuesta a la primera pregunta es sencilla. En un entorno de prueba, es muy difícil simular a los usuarios reales, sus patrones de comportamiento y los volúmenes de trabajo que podrían generar. La única manera completamente segura de comprender cómo se comporta un sistema bajo carga es observarlo en producción. Es preciso aclarar que no estamos sugiriendo que deba omitir las pruebas de rendimiento. Resultan cruciales para obtener las métricas de rendimiento de línea de base. Pero debe estar preparado para observar y corregir los problemas de rendimiento que surjan en el sistema real.

La respuesta a la segunda cuestión, cómo corregir el problema, es menos sencilla. Diversos factores podrían influir y, a veces, el problema solo se presenta en determinadas circunstancias. La instrumentación y el registro son factores clave para buscar la causa raíz, pero también tiene que saber lo que se debe buscar. 

Según nuestras interacciones con los clientes de Microsoft Azure, hemos identificado algunos de los problemas de rendimiento más comunes que observan en producción. Para cada antipatrón, se describe por qué se suele producir, sus síntomas y las técnicas para solucionar el problema. También se proporciona código de ejemplo que ilustra tanto el antipatrón como la solución sugerida. 

Algunos de estos antipatrones pueden parecer una obviedad cuando lea las descripciones, pero se producen con más frecuencia de lo que se podría pensar. A veces, una aplicación hereda un diseño que funcionaba de forma local, pero no se escala en la nube. O una aplicación podría iniciarse con un diseño muy limpio, pero, cuando se agregan características nuevas, se cuelan uno o varios de estos antipatrones. En cualquier caso, esta guía le ayudará a identificarlos y corregirlos.

Esta es la lista de los antipatrones que hemos identificado: 

| Antipatrón | Descripción |
|-------------|-------------|
| [Busy Database][BusyDatabase] | Descarga demasiados procesos en un almacén de datos. |
| [Busy Front End][BusyFrontEnd] | Mueve las tareas que consumen muchos recursos a subprocesos en segundo plano. |
| [Chatty I/O][ChattyIO] | Envía continuamente muchas solicitudes pequeñas de red. |
| [Extraneous Fetching][ExtraneousFetching] | Recupera más datos de lo que es necesario, lo que da lugar a E/S innecesarias. |
| [Improper Instantiation][ImproperInstantiation] | Crea y destruye objetos varias veces que están diseñados para compartirse y reutilizarse. |
| [Monolithic Persistence][MonolithicPersistence] | Utiliza el mismo almacén de datos para datos con patrones de uso muy diferentes. |
| [No Caching][NoCaching] | No se puede almacenar datos en caché. |
| [Synchronous I/O][SynchronousIO] | Bloquea el subproceso de llamada mientras se completa la E/S. | 

[BusyDatabase]: ./busy-database/index.md
[BusyFrontEnd]: ./busy-front-end/index.md
[ChattyIO]: ./chatty-io/index.md
[ExtraneousFetching]: ./extraneous-fetching/index.md
[ImproperInstantiation]: ./improper-instantiation/index.md
[MonolithicPersistence]: ./monolithic-persistence/index.md
[NoCaching]: ./no-caching/index.md
[SynchronousIO]: ./synchronous-io/index.md