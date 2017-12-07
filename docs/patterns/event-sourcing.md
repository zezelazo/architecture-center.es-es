---
title: Event Sourcing
description: "Usa un almacén de solo anexar para registrar la serie completa de eventos que describen las acciones realizadas en los datos de un dominio."
keywords: "Patrón de diseño"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- data-management
- performance-scalability
ms.openlocfilehash: d5d4e99a6ff49cb823f592c83590471c0d68bfd1
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2017
---
# <a name="event-sourcing-pattern"></a>Patrón Event Sourcing

[!INCLUDE [header](../_includes/header.md)]

En lugar de almacenar simplemente el estado actual de los datos de un dominio, use un almacén de solo anexar para registrar toda la serie de acciones realizadas en los datos.
El almacén actúa como el sistema de registro y puede usarse para materializar los objetos de dominio. Esto puede simplificar las tareas en los dominios complejos, al evitar la necesidad de sincronizar el modelo de datos y el dominio empresarial, al tiempo que mejora el rendimiento, la escalabilidad y la capacidad de respuesta. También proporciona coherencia a los datos transaccionales y conserva trazas de auditoría e historiales completos que permiten acciones de compensación.

## <a name="context-and-problem"></a>Contexto y problema

La mayoría de las aplicaciones trabajan con datos; el enfoque típico es que la aplicación mantenga el estado actual de los datos mediante su actualización conforme los usuarios trabajan con ellos. Por ejemplo, en el modelo tradicional de creación, lectura, actualización y eliminación (CRUD), un proceso de datos típico consiste en leer datos del almacén, realizar algunas modificaciones en ellos y actualizar su estado actual con los nuevos valores (a menudo mediante transacciones que bloquean los datos).

El enfoque CRUD tiene algunas limitaciones:

- Los sistemas CRUD realizan operaciones de actualización directamente en un almacén de datos, lo que ralentiza el rendimiento y la capacidad de respuesta y limita la escalabilidad, a causa de la sobrecarga de procesamiento que requiere.

- En un dominio de colaboración con muchos usuarios simultáneos, los conflictos de actualización de datos son más probables, dado que las operaciones de actualización tienen lugar en un solo elemento de datos.

- A menos que exista un mecanismo de auditoría adicional que registre los detalles de cada operación en un registro independiente, el historial se pierde.

> Para una mejor comprensión de los límites del enfoque CRUD, consulte [CRUD, Only When You Can Afford It](https://blogs.msdn.microsoft.com/maarten_mullender/2004/07/23/crud-only-when-you-can-afford-it-revisited/) (CRUD, solo cuando se lo pueda permitir).

## <a name="solution"></a>Solución

El patrón Event Sourcing define un enfoque para controlar las operaciones basado en una secuencia de eventos, cada uno de los cuales se registra en un almacén de solo anexar. El código de la aplicación envía una serie de eventos que imperativamente describen cada acción que se ha producido en los datos del almacén de eventos, donde se conservan. Cada evento representa un conjunto de cambios en los datos (como `AddedItemToOrder`).

Los eventos se conservan en un almacén de eventos que actúa como sistema de registro (origen de datos acreditado) del estado actual de los datos. Normalmente, el almacén de eventos publica estos eventos para que los consumidores pueden recibir notificaciones y controlarlos si lo necesitan. Los consumidores podrían, por ejemplo, iniciar tareas que se aplican las operaciones en los eventos a otros sistemas o realizar cualquier acción asociada que se necesita para completar la operación. Tenga en cuenta que el código de aplicación que genera los eventos es independiente de los sistemas que se suscriben a los eventos.

Los usos típicos de los eventos publicados por el almacén de eventos son el mantenimiento de vistas materializadas de entidades según las cambian las acciones de la aplicación y la integración con sistemas externos. Por ejemplo, un sistema puede mantener una vista materializada de todos los pedidos de cliente que se usa para rellenar elementos de la interfaz de usuario. Cuando la aplicación agrega pedidos nuevos, agrega o quita elementos del pedido y agrega información de envío, los eventos que describen estos cambios pueden se controlan y se utilizan para actualizar la [vista materializada](materialized-view.md).

Además, las aplicaciones pueden leer el historial de eventos en cualquier momento y usarlo para materializar el estado actual de una entidad al reproducir y consumir todos los eventos relacionados con esa entidad. Esto puede ocurrir a petición para materializar un objeto de dominio al controlar una solicitud, o a través de una tarea programada para que se puede almacenar el estado de la entidad como vista materializada para admitir la capa de presentación.

La ilustración muestra una introducción al patrón, incluidas algunas de las opciones de uso del flujo de eventos, como la creación de una vista materializada, la integración de eventos con aplicaciones y sistemas externos y la reproducción de eventos para crear proyecciones del estado actual de entidades específicas.

![Introducción y ejemplo del patrón Event Sourcing](./_images/event-sourcing-overview.png)


El patrón Event Sourcing proporciona las siguientes ventajas:

Los eventos son inmutables y pueden almacenarse mediante una operación de solo anexar. La interfaz de usuario, el flujo de trabajo o el proceso que iniciaran un evento pueden continuar y las tareas que controlan los eventos se pueden ejecutar en segundo plano. Esto, junto con el hecho de que no hay ningún contención durante el procesamiento de transacciones, puede mejorar considerablemente el rendimiento y la escalabilidad de las aplicaciones, especialmente el nivel de presentación o la interfaz de usuario.

Los eventos son objetos simples que describen alguna acción que se ha producido, junto con los datos asociados necesarios para describir la acción que representa el evento. Los eventos no actualizan directamente un almacén de datos. Simplemente se registran para el control en el momento adecuado. Esto puede simplificar la implementación y la administración.

Los eventos suelen tener significado para un experto de dominio, mientras que los [errores de coincidencia de impedancia relacional de objetos](https://en.wikipedia.org/wiki/Object-relational_impedance_mismatch) pueden dificultar la comprensión de las tablas de base de datos complejas. Las tablas son construcciones artificiales que representan el estado actual del sistema, no los eventos que se han producido.

Event Sourcing puede ayudar a impedir que las actualizaciones simultáneas causen conflictos, ya que evita la necesidad de actualizar directamente los objetos en el almacén de datos. Sin embargo, el modelo de dominio debe diseñarse aún para protegerlo de las solicitudes que puedan generar incoherencias.

El almacenamiento de solo anexar eventos proporciona una traza de auditoría para supervisar las acciones realizadas en un almacén de datos, volver a crear el estado actual como vista materializada o proyecciones (al reproducir los eventos en cualquier momento) y ayudar a probar y depurar el sistema. Además, el requisito de usar eventos de compensación para cancelar cambios proporciona un historial de los cambios revertidos, que no sería el caso si el modelo simplemente almacenara el estado actual. La lista de eventos también puede utilizarse para analizar el rendimiento de la aplicación y detectar las tendencias de comportamiento de los usuarios o para obtener otra información empresarial de utilidad.

El almacén de eventos genera eventos y las tareas realizan operaciones en respuesta a esos eventos. Esta separación de las tareas de los eventos proporciona flexibilidad y extensibilidad. Las tareas conocen el tipo de evento y los datos del evento, pero no la operación que lo desencadenó. Además, varias tareas pueden controlar el mismo evento. Esto facilita la integración con otros servicios y sistemas que solo escuchen los nuevos eventos que genere el almacén de eventos. Sin embargo, los eventos de Event Sourcing tienden a ser de muy bajo nivel, por lo que podría ser necesaria la creación de eventos de integración específicos en su lugar.

> Event Sourcing suele combinarse con el patrón CQRS, ya que realiza las tareas de administración de datos en respuesta a los eventos y materializa las vistas de los eventos almacenados.

## <a name="issues-and-considerations"></a>Problemas y consideraciones

Tenga en cuenta los puntos siguientes al decidir cómo implementar este patrón:

El sistema solo será coherente al crear vistas materializadas o proyecciones de los datos mediante la reproducción de eventos. Existe cierto retardo desde que la aplicación agrega eventos al almacén tras administrar una solicitud, hasta que se publican los eventos y los consumidores de estos los controlan. Durante este período pueden haber llegado al almacén eventos nuevos que describan más cambios en las entidades.

> [!NOTE]
> Consulte [Data Consistency Primer](https://msdn.microsoft.com/library/dn589800.aspx)(Manual básico de coherencia de datos) para información sobre la coherencia final.

El almacén de eventos es la fuente permanente de información, por lo que los datos de los eventos no deben actualizarse. Es la única manera de actualizar una entidad para deshacer un cambio es agregar un evento de compensación al almacén de eventos. Si necesita cambiar el formato (en lugar de los propios datos) de los eventos persistentes, quizás durante una migración, puede ser difícil combinar los eventos existentes en el almacén con la nueva versión. Podría ser necesario recorrer en iteración todos los eventos con cambios para que sean compatibles con el nuevo formato o agregar nuevos eventos que utilicen el nuevo formato. Considere el uso de una marca de versión para cada versión del esquema de evento para mantener el formato antiguo y el nuevo.

En el almacén de eventos pueden guardar eventos las aplicaciones multiproceso y varias instancias de aplicaciones. La coherencia de los eventos del almacén es muy importante, ya que supone el orden de los eventos que afectan a una entidad específica (el orden en que se producen los cambios en una entidad afecta a su estado actual). Agregar una marca de tiempo para cada evento puede ayudar a evitar problemas. Otra práctica común consiste en anotar cada evento derivado de una solicitud con un identificador incremental. Si dos acciones intentan agregar eventos a la misma entidad al mismo tiempo, el almacén de eventos puede rechazar uno cuyo identificador de entidad existente coincida con el identificador de evento.

No hay ningún enfoque estándar ni mecanismos existentes como consultas SQL para leer los eventos y obtener información. Los únicos datos que se pueden extraer son un flujo de eventos con un identificador de evento como criterio. El identificador de evento normalmente se asigna a entidades individuales. Se puede determinar el estado actual de una entidad solo mediante la reproducción de todos los eventos relacionados con ella en comparación con el estado original de la entidad.

La longitud de los flujos de eventos afecta a la administración y la actualización del sistema. Si son grandes, considere la posibilidad de crear instantáneas a intervalos específicos, como de un número determinado de eventos. El estado actual de la entidad puede obtenerse de la instantánea y al reproducir los eventos que se produjeran después de ese momento. Para más información sobre la creación de instantáneas de datos, consulte [Snapshot](http://martinfowler.com/eaaDev/Snapshot.html) (Instantáneas) en el sitio web de la arquitectura de las aplicaciones empresariales de Martin Fowler y [Master-Subordinate Snapshot Replication](https://msdn.microsoft.com/library/ff650012.aspx) (Replicación maestro-subordinado de instantáneas).

Aunque Event Sourcing reduce la posibilidad de conflicto entre actualizaciones de los datos, la aplicación sigue teniendo que procesar las incoherencias derivadas de la coherencia final y la falta de transacciones. Por ejemplo, un evento que indica una reducción en el inventario estándar puede llegar al almacén de datos mientras se realiza un pedido de ese elemento, lo que requeriría una conciliación de las dos operaciones mediante un aviso al cliente o la creación de un pedido en espera.

La publicación de eventos podría ser "al menos una vez", de manera que los consumidores de los eventos deben ser idempotentes. No debe volver a aplicar la actualización descrita en un evento si este se controla más de una vez. Por ejemplo, si varias instancias de un consumidor mantienen un agregado de propiedades de una entidad, como el total de pedidos realizados, solo una debe incrementar correctamente el agregado cuando se produzca un evento de pedido realizado. Aunque esto no es una característica clave de Event Sourcing, es la decisión de implementación habitual.

## <a name="when-to-use-this-pattern"></a>Cuándo usar este patrón

Use este patrón en los escenarios siguientes:

- Si desea capturar la intención, el propósito o el motivo en los datos. Por ejemplo, los cambios en una entidad de cliente se pueden capturar como una serie de tipos de evento específicos como _Cambio de domicilio_, _Cierre de cuenta_ o _Fallecimiento_.

- Cuando es vital reducir o evitar por completo la aparición de actualizaciones de datos en conflicto.

- Si desea registrar los eventos que se produzcan y poder reproducirlos para restaurar el estado de un sistema, revertir cambios o mantener un historial y un registro de auditoría. Por ejemplo, cuando una tarea implica varios pasos, quizá deba ejecutar acciones para revertir las actualizaciones y, a continuación, reproducir algunos pasos para devolver los datos a un estado coherente.

- Cuando el uso de eventos es una característica natural de la operación de la aplicación y requiere poco esfuerzo de implementación o desarrollo adicional.

- Cuando necesite desacoplar el proceso de entrada o actualización de datos de las tareas necesarias para aplicar estas acciones. Quizá para mejorar el rendimiento de la interfaz de usuario o para distribuir eventos a otros agentes de escucha que realicen acciones cuando se producen los eventos. Por ejemplo, la integración de un sistema de nóminas con un sitio web de envío de gastos para que los eventos generados por el almacén de eventos en respuesta a las actualizaciones de datos en el sitio web los consuma tanto el sistema de nóminas como el sistema de nóminas.

- Si desea flexibilidad para cambiar el formato de los modelos materializados y los datos de la entidad si cambian los requisitos o cuando tenga que adaptar un modelo de lectura o las vistas que exponen los datos (cuando se utiliza junto con CQRS).

- Cuando se usa junto con CQRS y la coherencia final es aceptable mientras se actualiza un modelo de lectura o si el impacto sobre el rendimiento de rehidratar las entidades y los datos de un flujo de eventos es aceptable.

Este patrón podría no ser útil en las siguientes situaciones:

- Dominios pequeños o simples, sistemas con poca o ninguna lógica de negocios o sistemas sin dominio que funcionan bien naturalmente con mecanismos de administración de datos CRUD tradicionales.

- Sistemas en los que se necesite coherencia y actualizaciones en las vistas de los datos en tiempo real.

- Sistemas donde no se necesiten trazas de auditoría, historial y capacidades para revertir y reproducir acciones.

- Sistemas donde la cantidad de actualizaciones en conflicto en los datos subyacentes es muy baja. Por ejemplo, los sistemas que fundamentalmente agregan datos en lugar de actualizarlos.

## <a name="example"></a>Ejemplo

Un sistema de administración de conferencias necesita realizar un seguimiento del número de reservas completadas para una conferencia con el fin de comprobar si hay plazas disponibles cuando un posible asistente intenta realizar una reserva. El sistema puede almacenar el número total de reservas para una conferencia de al menos dos formas:

- Almacenar la información sobre el número total de reservas como una entidad independiente en una base de datos con la información de reserva. Como las reservas se realizan o se cancelan, el sistema puede incrementar o disminuir este número según corresponda. En teoría, este enfoque es sencillo, pero puede causar problemas de escalabilidad si un gran número de asistentes intenta reservar plaza en un tiempo breve. Por ejemplo, en el último día o dos antes del cierre del período de reserva.

- El sistema podría almacenar información acerca de las reservas y cancelaciones como eventos que se encuentran en un almacén de eventos. A continuación, calcularía el número de plazas disponibles mediante la reproducción de estos eventos. Este enfoque puede ser más escalable debido a la inmutabilidad de los eventos. El sistema solo necesita leer los datos del almacén de eventos o anexarlos al almacén de eventos. La información de reservas y cancelaciones de eventos nunca se modifica.

El siguiente diagrama ilustra cómo se podría implementar con Event Sourcing el subsistema de reserva plazas del sistema de administración de conferencias.

![Captura de información sobre las reservas de plaza en un sistema de administración de conferencias con Event Sourcing](./_images/event-sourcing-bounded-context.png)


La secuencia de acciones para reservar dos plazas es la siguiente:

1. La interfaz de usuario emite un comando para reservar plaza a dos asistentes. El comando lo controla un controlador de comandos independiente. Se separa una parte de la lógica de la interfaz de usuario y es responsable de controlar las solicitudes que se publican como comandos.

2. Al consultar los eventos que se describen las reservas y las cancelaciones se crea un agregado que contiene información sobre todas las reservas de la conferencia. Este agregado se denomina `SeatAvailability` y se encuentra dentro de un modelo de dominio que expone métodos para consultar y modificar sus datos.

    > Algunas optimizaciones que tener en cuenta son el uso de instantáneas (para no tener que consultar y reproducir la lista completa de eventos para obtener el estado actual del agregado) y la conservación de una copia en caché del agregado en la memoria.

3. El controlador de comandos invoca un método expuesto por el modelo de dominio para realizar las reservas.

4. El agregado `SeatAvailability` registra un evento que contiene el número de plazas que se han reservado. La próxima vez que el agregado aplique eventos, todas las reservas se utilizarán para calcular el número de plazas restantes.

5. El sistema anexa el nuevo evento a la lista del almacén de eventos.

Si un usuario cancela una plaza, el sistema sigue un proceso similar, salvo que el controlador de comandos emite un comando que genera un evento de cancelación de plaza y lo anexa al almacén de eventos.

Además de proporcionar un ámbito más escalabilidad, un almacén de eventos también proporciona un historial completo (o traza de auditoría) de las reservas y las cancelaciones de una conferencia. Los eventos del almacén son el registro exacto. No es necesario conservar los agregados de otra manera, ya que el sistema puede reproducir fácilmente los eventos y restaurar el estado a cualquier punto en el tiempo.

> Más información acerca de este ejemplo en el artículo de [introducción Event Sourcing](https://msdn.microsoft.com/library/jj591559.aspx).

## <a name="related-patterns-and-guidance"></a>Orientación y patrones relacionados

Los patrones y las directrices siguientes también pueden ser importantes a la hora de implementar este modelo:

- [Patrón Command and Query Responsibility Segregation (CQRS)](cqrs.md). El almacén de escritura que proporciona el origen de información permanente para una implementación de CQRS a menudo se basa en una implementación del patrón Event Sourcing. Describe cómo se segregan las operaciones de lectura de las de actualización de datos mediante interfaces independientes en una aplicación.

- [Patrón Materialized View](materialized-view.md). Normalmente, el almacén de datos que se utiliza en un sistema basado en Event Sourcing no se adapta perfectamente para una consulta eficaz. En su lugar, un enfoque común consiste en generar vistas rellenadas previamente con los datos a intervalos regulares o cuando los datos cambian. Muestra la manera de hacerlo.

- [Patrón Compensating Transaction](compensating-transaction.md). Los datos existentes en un almacén de Event Sourcing no se actualizan; en su lugar, se agregan entradas nuevas de transición del estado de las entidades a los nuevos valores. Para invertir un cambio, se usan las entradas de compensación, puesto que no es posible revertir simplemente el cambio. Describe cómo deshacer el trabajo que realizó una operación anterior.

- [Data Consistency Primer](https://msdn.microsoft.com/library/dn589800.aspx) (Manual básico de coherencia de datos). Al usar Event Sourcing con otro almacén de lectura o vistas materializadas independientes, los datos de lectura no serán coherentes de manera inmediata, sino que tendrán coherencia final. Resume los problemas que pueden surgir sobre el mantenimiento de la coherencia en los datos distribuidos.

- [Guía de creación de particiones de datos](https://msdn.microsoft.com/library/dn589795.aspx). Se suelen generar particiones en los datos al usar Event Sourcing para mejorar la escalabilidad, reducir la contención y optimizar el rendimiento. Describe cómo dividir los datos en particiones discretas y los problemas que pueden surgir.

- Publicación [Why use Event Sourcing?](http://codebetter.com/gregyoung/2010/02/20/why-use-event-sourcing/) (¿Por qué usar Event Sourcing?) de Greg Young.
