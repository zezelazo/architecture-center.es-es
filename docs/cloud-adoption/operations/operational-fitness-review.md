---
title: 'Adopción de la nube empresarial: fundamentos operacionales'
description: Orientación sobre los fundamentos operacionales
author: petertaylor9999
ms.date: 09/20/2018
ms.openlocfilehash: d5f4c6529e92be387465a6ab9dca55267c584c11
ms.sourcegitcommit: b7e521ba317f4fcd3253c80ac0c0a355eaaa56c5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/21/2018
ms.locfileid: "46534365"
---
# <a name="establishing-an-operational-fitness-review"></a>Establecimiento de una revisión de adecuación operativa

Una vez que su empresa empieza a operar las cargas de trabajo en Azure, el paso siguiente consiste en establecer un proceso de **revisión de adecuación operativa** para enumerar, implementar y revisar de forma iterativa los requisitos **no funcionales** de estas cargas de trabajo. Los requisitos _no funcionales_ están relacionados con el comportamiento operativo esperado del servicio. Existen cinco categorías básicas de requisitos no funcionales que se conocen como [fundamentos de calidad del software](../../guide/pillars.md): escalabilidad, disponibilidad, resistencia (lo que incluye la continuidad del negocio y la recuperación ante desastres), administración y seguridad. La finalidad de un proceso de revisión de adecuación operativa es garantizar que sus cargas de trabajo críticas cumplen las expectativas de la empresa con respecto a los fundamentos de calidad.

Por este motivo, la empresa debe llevar a cabo un proceso de revisión de adecuación operativa para entender totalmente los problemas derivados de ejecutar la carga de trabajo en un entorno de producción, determinar cómo solucionar los problemas y resolverlos. En este artículo se describe un proceso de revisión de adecuación operativa de alto nivel que puede usar su empresa para conseguir este objetivo.

## <a name="operational-fitness-at-microsoft"></a>Adecuación operativa en Microsoft

Desde el principio, el desarrollo de la plataforma Azure ha sido un proyecto de integración y desarrollo continuados acometido por muchos equipos en Microsoft. Sería muy difícil garantizar la calidad y la coherencia de un proyecto del tamaño y la complejidad de Azure sin un proceso sólido para enumerar e implementar los requisitos no funcionales esenciales periódicamente.

Estos procesos que sigue Microsoft constituyen la base de los descritos en este documento.

## <a name="understanding-the-problem"></a>Comprensión del problema

Como aprendió en la [Introducción](../../cloud-adoption/getting-started/overview.md), el primer paso en la transformación digital de una empresa es identificar los problemas empresariales que se van a resolver con la adopción de Azure. El siguiente paso es determinar una solución de alto nivel para el problema, como la migración de una carga de trabajo a la nube o la adaptación de un servicio local existente para incluir funcionalidad de nube. Finalmente, la solución está diseñada e implementada.

Durante este proceso, el foco está con frecuencia sobre las _características_ del servicio. Es decir, hay un conjunto de requisitos _funcionales_ deseados que el servicio debe realizar. Por ejemplo, un servicio de entrega de productos requiere características para determinar las ubicaciones de origen y destino del producto, realizar el seguimiento del producto durante la entrega, enviar notificaciones al cliente, etc.

En cambio, los requisitos _no funcionales_ se relacionan con propiedades como la [disponibilidad](../../checklist/availability.md), la [resistencia](../../resiliency/index.md) y la [escalabilidad](../../checklist/scalability.md) del servicio. Estas propiedades se diferencian de los requisitos funcionales en que no influyen directamente sobre la función final de ninguna característica en particular del servicio. Sin embargo, estos requisitos no funcionales están relacionados con el _rendimiento_ y la _continuidad_ del servicio.

Algunos requisitos no funcionales se pueden especificar en los términos de un Acuerdo de Nivel de Servicio (SLA). Por ejemplo, en el caso de la continuidad del servicio, un requisito de disponibilidad del servicio se puede expresar como un porcentaje, por ejemplo, **disponible el 99,99 % del tiempo**. Otros requisitos no funcionales pueden ser más difíciles de definir y pueden cambiar a medida que evolucionan las necesidades de producción. Por ejemplo, un servicio orientado al consumidor podría comenzar a toparse con requisitos de rendimiento no anticipados tras un aumento repentino de popularidad.

[NOTA] La definición de los requisitos de resistencia, incluidas las explicaciones de RPO, RTO, SLA y conceptos relacionados, se explora con más detalle en [Diseño de aplicaciones resistentes de Azure](../../resiliency/index.md#define-your-availability-requirements).

## <a name="operational-fitness-review-process"></a>Proceso de revisión de adecuación operativa

La clave para mantener el rendimiento y la continuidad de los servicios de una empresa es implementar un proceso de _revisión de adecuación operativa_.

![Una introducción al proceso de revisión de adecuación operativa](_images/ofr-flow.png)

En un nivel alto, el proceso tiene dos fases. En la fase de requisitos previos, los requisitos se establecen y se asignan a servicios complementarios. Esto se produce con menos frecuencia; quizás cada año o cuando se introducen nuevas operaciones. El resultado de la fase de requisitos previos se usa en la fase de flujo. La fase de flujo se produce con más frecuencia; se recomienda una vez al mes.

### <a name="prerequisites-phase"></a>Fase de requisitos previos

Los pasos descritos en esta fase tienen como fin capturar los requisitos necesarios para llevar a cabo una revisión periódica de los servicios importantes.

- **Identificar las operaciones empresariales críticas**. Identifique las operaciones empresariales **críticas** de la empresa. Las operaciones empresariales son independientes de cualquier funcionalidad de servicio complementaria. En otras palabras, las operaciones empresariales representan las actividades reales que el negocio debe realizar y están respaldadas por un conjunto de servicios de TI. El término _crítico_ o también _crítico para el negocio_, refleja un efecto grave sobre la empresa si se impide la operación. Por ejemplo, un minorista en línea puede tener una operación empresarial del tipo "permitir que un cliente agregue un artículo al carro de la compra" o "procesar un pago con tarjeta de crédito". Si alguna de estas operaciones genera un error, el cliente no podrá realizar la transacción y la empresa no podría realizar ventas.

- **Asignar operaciones a servicios**. Asigne estas operaciones empresariales a los servicios que las respaldan. En el ejemplo anterior del carro de la compra, pueden intervenir varios servicios: un servicio de administración del stock de inventario, un servicio de carro de la compra y otros. En el ejemplo anterior de pago con tarjeta de crédito, un servicio de pago local podría interactuar con un servicio de procesamiento de pagos de terceros.

- **Analizar las dependencias de los servicios**. La mayoría de las operaciones empresariales requiere la orquestación entre varios servicios complementarios. Es importante comprender las dependencias entre los servicios y el flujo de transacciones críticas a través de estos servicios. También es necesario tener en cuenta las dependencias entre los servicios locales y los servicios de Azure. En el ejemplo del carro de la compra, el servicio de administración del stock de inventario se podría hospedar en el entorno local e ingerir los datos que especifican los empleados desde un almacén físico, pero podría almacenar los datos en un servicio de Azure como [Azure Storage](/azure/storage/common/storage-introduction) o una base de datos como [Azure Cosmos DB](/azure/cosmos-db/introduction).

Una salida de estas actividades es un conjunto de **métricas del cuadro de mandos** de las operaciones de servicio. Las métricas se clasifican en términos de criterios no funcionales, como disponibilidad, escalabilidad y recuperación ante desastres. Las métricas del cuadro de mandos expresan los criterios que se espera que cumpla el servicio de forma operativa. Estas métricas se pueden expresar con cualquier nivel de pormenorización que sea adecuado para la operación de servicio.

El cuadro de mandos debe expresarse en términos sencillos para facilitar la comunicación significativa entre los propietarios de empresa y los ingenieros. Por ejemplo, una métrica del cuadro de mandos de escalabilidad se puede expresar como _verde_ para indicar que el rendimiento se ajusta a los criterios deseados, _amarillo_ para indicar que no se cumplen los criterios deseados, pero se está en proceso de implantar un plan de corrección, y _rojo_ para indicar que no se cumplen los criterios deseados con ningún plan o acción.

Es importante destacar que estas métricas deben reflejar directamente las necesidades empresariales.

### <a name="service-review-phase"></a>Fase de revisión de los servicios

La fase de revisión de los servicios es el núcleo del proceso de revisión de adecuación operativa.

- **Medir las métricas de los servicios**. Con las métricas del cuadro de mandos, los servicios se deben supervisar para garantizar que cumplen las expectativas empresariales. Esto significa que la supervisión de los servicios es fundamental. Si no es capaz de supervisar un conjunto de servicios con respecto a los requisitos no funcionales, las métricas del cuadro de mando correspondientes se deben considerar en rojo. En este caso, el primer paso para solucionarlo es implementar la supervisión de los servicios adecuada.
Por ejemplo, si la empresa espera que un servicio funcione con una disponibilidad del 99,99 %, pero no dispone de datos de telemetría de producción para medir la disponibilidad, debe asumir que no satisface el requisito.

- **Planear las acciones correctivas**. Con cada operación de servicio con métricas que se encuentren por debajo de un umbral aceptable, determine el costo de corregir el servicio para llevar la operación a una métrica aceptable. Si el costo de la corrección del servicio es mayor que la generación de ingresos esperados del servicio, pase a considerar los costos no tangibles como la experiencia del cliente. Por ejemplo, si los clientes tienen dificultad para realizar pedidos correctamente con el servicio, podrían elegir en su lugar a la competencia.

- **Implementar el plan correctivo**. Una vez que los propietarios de empresa y los ingenieros acuerdan un plan, se debe implementar. Cada vez que se revisen las métricas del cuadro de mandos, se debe notificar el estado de la implementación.

Este proceso es iterativo, y lo ideal es que la empresa tenga un equipo destinado a apropiarse de él. Este equipo debe reunirse periódicamente para revisar los proyectos de corrección existentes, poner en marcha la revisión de aspectos básicos de nuevas cargas de trabajo y realizar un seguimiento del cuadro de mandos general de la empresa. El equipo debe tener la autoridad para garantizar la responsabilidad de los equipos de corrección que están detrás de la programación o la falta de cumplimiento de las métricas.

## <a name="structure-of-the-operational-fitness-review-team"></a>Estructura del equipo de revisión de adecuación operativa

El equipo de revisión de adecuación operativa se compone de los siguientes roles:

1. **Propietario de la empresa**. Este rol proporciona conocimiento del negocio para identificar y clasificar en orden de prioridad cada operación empresarial "crítica". Este rol también compara el costo de la solución con la repercusión para el negocio y conduce la decisión final sobre la corrección.

2. **Defensor del negocio**. Este rol es responsable de dividir las operaciones empresariales en partes discretas y de asignar esas partes a la infraestructura y los servicios en la nube y locales. El rol requiere un conocimiento profundo de la tecnología asociada con cada operación empresarial.

3. **Propietario de ingeniería**. Este rol es responsable de implementar los servicios asociados con la operación empresarial. Estas personas pueden participar en el diseño y la implementación de posibles soluciones para resolver los problemas de requisitos no funcionales detectados por el equipo de revisión de adecuación operativa.

4. **Propietario del servicio**. Este rol es responsable del funcionamiento de las aplicaciones y los servicios de la empresa. Estos individuos recopilan datos de registro y uso de estas aplicaciones y servicios. Estos datos se usan para identificar problemas y comprobar las soluciones una vez implementadas.

## <a name="operational-fitness-review-meeting"></a>Reunión de revisión de adecuación operativa

Es recomendable que el equipo de revisión de adecuación operativa se reúna periódicamente. Por ejemplo, el equipo se podría reunir todos los meses, e informar del estado y las métricas al personal directivo de forma trimestral.

Los detalles del proceso y la reunión se deben adaptar a sus necesidades específicas. Como punto de partida, se recomiendan las siguientes tareas:

1. El propietario de la empresa y el defensor del negocio enumeran y determinan los requisitos no funcionales de cada operación empresarial, con la aportación de los propietarios de ingeniería y del servicio. Con las operaciones empresariales que se han identificado anteriormente, se revisa y comprueba la prioridad. Con las operaciones de negocio nuevas, se asigna una prioridad de la lista existente.

2. Los propietarios de ingeniería y del servicio asignan el **estado actual** de las operaciones empresariales a los servicios en la nube y locales correspondientes. La asignación se compone de una lista de los componentes de cada servicio, orientada como un árbol de dependencias. Una vez que se generan la lista y el árbol de dependencias, se determinan las **rutas críticas** a través del árbol.

3. Los propietarios de ingeniería y del servicio revisan el estado actual del registro y la supervisión operativos de los servicios enumerados en el paso anterior. Para identificar los componentes del servicio que contribuyen al incumplimiento de los requisitos no funcionales, son esenciales procedimientos de registro y supervisión de gran robustez. Si no se disponen de suficientes procedimientos de registro y supervisión, se debe crear e implementar un plan para implantarlos.

4. Se crean métricas del cuadro de mandos para la nueva operación empresarial. El cuadro de mandos se compone de la lista de componentes constituyentes de cada servicio identificado en el paso 2, en consonancia con los requisitos no funcionales y una métrica que representa el grado en que el componente cumple el requisito.

5. Para los componentes constituyentes que no cumplen los requisitos no funcionales, se diseña una solución de alto nivel y se asigna un propietario de ingeniería. En este momento, el propietario de la empresa y el defensor del negocio deben establecer un presupuesto para el trabajo de corrección, en función de los ingresos previstos de la operación empresarial.

6. Por último, se lleva a cabo una revisión del trabajo de corrección en curso. Cada una de las métricas del cuadro de mandos del trabajo en curso se revisa comparándola con las métricas esperadas. En el caso de los componentes constituyentes que satisfacen las métricas, el propietario del servicio presenta los datos de registro y supervisión para confirmar que se cumple la métrica. En cuanto a esos componentes constituyentes que no satisfacen las métricas, cada propietario de ingeniería explica los problemas que impiden que se alcancen las métricas y las posibles nuevas formas de corregir el problema.

## <a name="recommended-resources"></a>Recursos recomendados

- [Fundamentos de calidad del software](../../guide/pillars.md).
En esta sección de la guía de arquitectura de aplicaciones de Azure de describen los cinco fundamentos de la calidad del software: escalabilidad, disponibilidad, resistencia, administración y seguridad.
- [Diez principios de diseño para las aplicaciones de Azure](../../guide/design-principles/index.md).
En esta sección de la guía de arquitectura de aplicaciones de Azure se analiza un conjunto de principios de diseño para que la aplicación sea más escalable, resistente y fácil de administrar.
- [Diseño de aplicaciones resistentes de Azure](../../resiliency/index.md).
Esta guía comienza con una definición del término resistencia y los conceptos relacionados. Después se describe un proceso para lograr resistencia, mediante un enfoque estructurado durante la vigencia de una aplicación, desde el diseño y la puesta en marcha hasta la implementación y operaciones.
- [Patrones de diseño en la nube](../../patterns/index.md).
Estos patrones de diseño son útiles para los equipos de ingeniería al compilar aplicaciones sobre los fundamentos de calidad del software.