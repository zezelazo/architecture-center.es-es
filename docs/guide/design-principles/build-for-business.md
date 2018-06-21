---
title: Tener en cuenta las necesidades del negocio
description: Cada decisión de diseño debe estar justificada por un requisito empresarial.
author: MikeWasson
ms.openlocfilehash: 768f2298860d91774d93c1917cf95000bb2b873d
ms.sourcegitcommit: 26b04f138a860979aea5d253ba7fecffc654841e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/19/2018
ms.locfileid: "36206749"
---
# <a name="build-for-the-needs-of-the-business"></a>Tener en cuenta las necesidades del negocio

## <a name="every-design-decision-must-be-justified-by-a-business-requirement"></a>Cada decisión de diseño debe estar justificada por un requisito empresarial.

Este principio de diseño puede parecer obvio pero es fundamental tenerlo en cuenta al diseñar una solución. ¿Prevé tener millones de usuarios o unos pocos miles? ¿Es aceptable una interrupción de la aplicación de una hora? ¿Espera grandes picos de tráfico o una carga de trabajo muy predecible? En última instancia, cada decisión de diseño debe estar justificada por un requisito empresarial. 

## <a name="recommendations"></a>Recomendaciones

**Definir objetivos de negocio**, incluidos el objetivo de tiempo de recuperación (RTO), el objetivo de punto de recuperación (RPO) y la interrupción tolerable máxima (MTO). Estos números deben dar información para ayudar a tomar decisiones acerca de la arquitectura. Por ejemplo, para lograr un objetivo de tiempo de recuperación bajo, podría implementar la conmutación por error automática a una región secundaria. Pero si la solución puede tolerar un objetivo de tiempo de recuperación superior, este grado de redundancia no es necesario.

**Documentar Acuerdos de Nivel de Servicio (SLA) y Objetivos de Nivel de Servicio (SLO)**, incluidas las métricas de rendimiento y la disponibilidad. Puede crear una solución que ofrezca una disponibilidad del 99,95 %. ¿Es suficiente? La respuesta es una decisión empresarial. 

**Modelar la aplicación en torno al dominio empresarial**. Comience por analizar los requisitos empresariales. Use estos requisitos para modelar la aplicación. Considere la posibilidad de usar un enfoque de diseño basado en el dominio (DDD) para crear [modelos de dominio][domain-model] que reflejen los procesos empresariales y los casos de uso. 

**Tener en cuenta los requisitos funcionales y los no funcionales**. Los requisitos funcionales le permiten evaluar si la aplicación realiza las operaciones adecuadas. Los requisitos no funcionales le permiten evaluar si la aplicación realiza estas operaciones *correctamente*. En concreto, asegúrese de que entiende sus requisitos de escalabilidad, disponibilidad y latencia. Estos requisitos influirán en las decisiones de diseño y la elección de la tecnología.

**Dividir por carga de trabajo**. El término "carga de trabajo" en este contexto significa una funcionalidad o tarea de computación discreta, que se puede separar lógicamente de otras tareas. Distintas cargas de trabajo pueden tener diferentes requisitos de disponibilidad, escalabilidad, coherencia de datos y recuperación ante desastres. 

**Planear el crecimiento**. Una solución podría satisfacer sus necesidades actuales, en términos de número de usuarios, volumen de transacciones, almacenamiento de datos, etc. Sin embargo, una aplicación sólida puede controlar el crecimiento sin cambios de arquitectura importantes. Consulte [Diseño de escalado horizontal](scale-out.md) y [Partición alrededor de límites](partition.md). Tenga también en cuenta que el modelo empresarial y los requisitos empresariales cambiarán probablemente con el tiempo. Si el modelo de servicio y los modelos de datos de una aplicación son demasiado rígidos, será difícil hacer evolucionar la aplicación para nuevos casos de uso y escenarios. Consulte [Diseño para evolucionar](design-for-evolution.md).

**Administrar los costos**. En una aplicación local tradicional, paga por adelantado por el hardware (CAPEX). En una aplicación en la nube, paga por los recursos que consume. Asegúrese de que comprende el modelo de precios para los servicios que consume. El costo total incluirá el uso de ancho de banda de red, almacenamiento, direcciones IP, consumo del servicio y otros factores. Para más información, consulte [Precios de Azure][pricing]. Tenga también en cuenta los costos de las operaciones. En la nube, no tiene que administrar el hardware u otra infraestructura pero sigue necesitando administrar las aplicaciones, incluidos DevOps, respuesta a incidentes, recuperación ante desastres, etc. 

[domain-model]: https://martinfowler.com/eaaCatalog/domainModel.html
[pricing]: https://azure.microsoft.com/pricing/
