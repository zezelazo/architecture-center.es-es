---
title: CQRS
description: Segrega las operaciones de lectura de datos de las de actualización de datos mediante interfaces independientes.
keywords: Patrón de diseño
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- data-management
- design-implementation
- performance-scalability
ms.openlocfilehash: de9530f7dd55c0ce5460cd3b58ab9f216c9b5c8c
ms.sourcegitcommit: fb22348f917a76e30a6c090fcd4a18decba0b398
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/16/2018
ms.locfileid: "53450877"
---
# <a name="command-and-query-responsibility-segregation-cqrs-pattern"></a><span data-ttu-id="61485-104">Patrón Command and Query Responsibility Segregation (CQRS).</span><span class="sxs-lookup"><span data-stu-id="61485-104">Command and Query Responsibility Segregation (CQRS) pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="61485-105">Segrega las operaciones de lectura de datos de las de actualización de datos mediante interfaces independientes.</span><span class="sxs-lookup"><span data-stu-id="61485-105">Segregate operations that read data from operations that update data by using separate interfaces.</span></span> <span data-ttu-id="61485-106">Esto puede maximizar el rendimiento, la escalabilidad y la seguridad.</span><span class="sxs-lookup"><span data-stu-id="61485-106">This can maximize performance, scalability, and security.</span></span> <span data-ttu-id="61485-107">Admite la evolución del sistema con el tiempo gracias a una mayor flexibilidad e impide que los comandos de actualización provoquen conflictos de combinación en el nivel de dominio.</span><span class="sxs-lookup"><span data-stu-id="61485-107">Supports the evolution of the system over time through higher flexibility, and prevents update commands from causing merge conflicts at the domain level.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="61485-108">Contexto y problema</span><span class="sxs-lookup"><span data-stu-id="61485-108">Context and problem</span></span>

<span data-ttu-id="61485-109">En los sistemas de administración de datos tradicionales, los comandos (actualizaciones de datos) y las consultas (solicitudes de datos) se ejecutan en el mismo conjunto de entidades en un único repositorio de datos.</span><span class="sxs-lookup"><span data-stu-id="61485-109">In traditional data management systems, both commands (updates to the data) and queries (requests for data) are executed against the same set of entities in a single data repository.</span></span> <span data-ttu-id="61485-110">Estas entidades pueden ser un subconjunto de filas de una o varias tablas de una base de datos relacional como SQL Server.</span><span class="sxs-lookup"><span data-stu-id="61485-110">These entities can be a subset of the rows in one or more tables in a relational database such as SQL Server.</span></span>

<span data-ttu-id="61485-111">Por lo general, en estos sistemas, todas las operaciones de creación, lectura, actualización y eliminación (CRUD) se aplican a la misma representación de la entidad.</span><span class="sxs-lookup"><span data-stu-id="61485-111">Typically in these systems, all create, read, update, and delete (CRUD) operations are applied to the same representation of the entity.</span></span> <span data-ttu-id="61485-112">Por ejemplo, un objeto de transferencia de datos (DTO) que representa a un cliente se recupera del almacén de datos mediante el nivel de acceso a los datos (DAL) y se muestra en la pantalla.</span><span class="sxs-lookup"><span data-stu-id="61485-112">For example, a data transfer object (DTO) representing a customer is retrieved from the data store by the data access layer (DAL) and displayed on the screen.</span></span> <span data-ttu-id="61485-113">Un usuario actualiza algunos campos del DTO (quizás mediante enlace de datos) y, a continuación, el DTO se guarda en el almacén de datos mediante el DAL.</span><span class="sxs-lookup"><span data-stu-id="61485-113">A user updates some fields of the DTO (perhaps through data binding) and the DTO is then saved back in the data store by the DAL.</span></span> <span data-ttu-id="61485-114">El mismo DTO se utiliza para las operaciones de lectura y de escritura.</span><span class="sxs-lookup"><span data-stu-id="61485-114">The same DTO is used for both the read and write operations.</span></span> <span data-ttu-id="61485-115">La ilustración muestra una arquitectura CRUD tradicional.</span><span class="sxs-lookup"><span data-stu-id="61485-115">The figure illustrates a traditional CRUD architecture.</span></span>

![Arquitectura CRUD tradicional](./_images/command-and-query-responsibility-segregation-cqrs-tradition-crud.png)

<span data-ttu-id="61485-117">Los diseños CRUD tradicionales funcionan bien si solo se aplica una lógica empresarial limitada a las operaciones de datos.</span><span class="sxs-lookup"><span data-stu-id="61485-117">Traditional CRUD designs work well when only limited business logic is applied to the data operations.</span></span> <span data-ttu-id="61485-118">Los mecanismos de scaffolding proporcionados por las herramientas de desarrollo pueden crear código de acceso a los datos de forma muy rápida y, posteriormente, este código se puede personalizar según sea necesario.</span><span class="sxs-lookup"><span data-stu-id="61485-118">Scaffold mechanisms provided by development tools can create data access code very quickly, which can then be customized as required.</span></span>

<span data-ttu-id="61485-119">No obstante, el enfoque CRUD tradicional tiene algunas desventajas:</span><span class="sxs-lookup"><span data-stu-id="61485-119">However, the traditional CRUD approach has some disadvantages:</span></span>

- <span data-ttu-id="61485-120">A menudo, eso significa que hay una incoherencia entre las representaciones de lectura y escritura de los datos, como columnas o propiedades adicionales que se deben actualizar correctamente incluso aunque no sean necesarias como parte de una operación.</span><span class="sxs-lookup"><span data-stu-id="61485-120">It often means that there's a mismatch between the read and write representations of the data, such as additional columns or properties that must be updated correctly even though they aren't required as part of an operation.</span></span>

- <span data-ttu-id="61485-121">Pone en riesgo la contención de datos cuando los registros están bloqueados en el almacén de datos de un dominio colaborativo, en los que varios actores trabajan en paralelo en el mismo conjunto de datos.</span><span class="sxs-lookup"><span data-stu-id="61485-121">It risks data contention when records are locked in the data store in a collaborative domain, where multiple actors operate in parallel on the same set of data.</span></span> <span data-ttu-id="61485-122">O actualiza los conflictos causados por las actualizaciones simultáneas cuando se utiliza el bloqueo optimista.</span><span class="sxs-lookup"><span data-stu-id="61485-122">Or update conflicts caused by concurrent updates when optimistic locking is used.</span></span> <span data-ttu-id="61485-123">Estos riesgos aumentan a medida que crece la complejidad y el rendimiento del sistema.</span><span class="sxs-lookup"><span data-stu-id="61485-123">These risks increase as the complexity and throughput of the system grows.</span></span> <span data-ttu-id="61485-124">Además, el enfoque tradicional puede tener un impacto negativo en el rendimiento debido a la carga en el almacén de datos y al nivel de acceso a los datos, y la complejidad de las consultas necesarias para recuperar la información.</span><span class="sxs-lookup"><span data-stu-id="61485-124">In addition, the traditional approach can have a negative effect on performance due to load on the data store and data access layer, and the complexity of queries required to retrieve information.</span></span>

- <span data-ttu-id="61485-125">Esto puede hacer que la administración de la seguridad y los permisos sea más compleja ya que cada entidad está sujeta a operaciones de lectura y de escritura, lo cual podría exponer los datos en el contexto equivocado.</span><span class="sxs-lookup"><span data-stu-id="61485-125">It can make managing security and permissions more complex because each entity is subject to both read and write operations, which might expose data in the wrong context.</span></span>

## <a name="solution"></a><span data-ttu-id="61485-126">Solución</span><span class="sxs-lookup"><span data-stu-id="61485-126">Solution</span></span>

<span data-ttu-id="61485-127">Command and Query Responsibility Segregation (CQRS) es un patrón que segrega las operaciones que leen datos (consultas) de las operaciones que actualizan los datos (comandos) mediante interfaces independientes.</span><span class="sxs-lookup"><span data-stu-id="61485-127">Command and Query Responsibility Segregation (CQRS) is a pattern that segregates the operations that read data (queries) from the operations that update data (commands) by using separate interfaces.</span></span> <span data-ttu-id="61485-128">Esto significa que los modelos de datos utilizados para realizar las consultas y las actualizaciones son diferentes.</span><span class="sxs-lookup"><span data-stu-id="61485-128">This means that the data models used for querying and updates are different.</span></span> <span data-ttu-id="61485-129">Posteriormente, se pueden aislar los modelos, como se indica en la siguiente ilustración, aunque eso no es un requisito imprescindible.</span><span class="sxs-lookup"><span data-stu-id="61485-129">The models can then be isolated, as shown in the following figure, although that's not an absolute requirement.</span></span>

![Una arquitectura CQRS básica](./_images/command-and-query-responsibility-segregation-cqrs-basic.png)

<span data-ttu-id="61485-131">En comparación con el modelo de datos único que se usa en los sistemas basados en CRUD, el uso de modelos de consulta y actualización independientes para los datos de los sistemas basados en CQRS simplifica el diseño y la implementación.</span><span class="sxs-lookup"><span data-stu-id="61485-131">Compared to the single data model used in CRUD-based systems, the use of separate query and update models for the data in CQRS-based systems simplifies design and implementation.</span></span> <span data-ttu-id="61485-132">Sin embargo, una desventaja es que, a diferencia de los diseños CRUD, el código CQRS no se genera automáticamente mediante los mecanismos de scaffolding.</span><span class="sxs-lookup"><span data-stu-id="61485-132">However, one disadvantage is that unlike CRUD designs, CQRS code can't automatically be generated using scaffold mechanisms.</span></span>

<span data-ttu-id="61485-133">El modelo de consulta para leer datos y el de actualización para escribirlos pueden acceder el mismo almacén físico, quizás mediante vistas SQL o mediante la generación de proyecciones sobre la marcha.</span><span class="sxs-lookup"><span data-stu-id="61485-133">The query model for reading data and the update model for writing data can access the same physical store, perhaps by using SQL views or by generating projections on the fly.</span></span> <span data-ttu-id="61485-134">Sin embargo, es habitual separar los datos en almacenes físicos diferentes para maximizar el rendimiento, la escalabilidad y la seguridad, tal como se muestra en la ilustración siguiente.</span><span class="sxs-lookup"><span data-stu-id="61485-134">However, it's common to separate the data into different physical stores to maximize performance, scalability, and security, as shown in the next figure.</span></span>

![Una arquitectura CQRS con almacenes independientes de lectura y escritura](./_images/command-and-query-responsibility-segregation-cqrs-separate-stores.png)

<span data-ttu-id="61485-136">El almacén de lectura puede ser una réplica de solo lectura del almacén de escritura o los almacenes de lectura y escritura pueden tener una estructura diferente por completo.</span><span class="sxs-lookup"><span data-stu-id="61485-136">The read store can be a read-only replica of the write store, or the read and write stores can have a different structure altogether.</span></span> <span data-ttu-id="61485-137">El uso de réplicas de solo lectura del almacén de lectura puede aumentar mucho el rendimiento de las consultas y la capacidad de respuesta de la interfaz de usuario de la aplicación, especialmente en escenarios distribuidos en los que las réplicas de solo lectura se sitúan cerca de las instancias de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="61485-137">Using multiple read-only replicas of the read store can greatly increase query performance and application UI responsiveness, especially in distributed scenarios where read-only replicas are located close to the application instances.</span></span> <span data-ttu-id="61485-138">Algunos sistemas de bases de datos (SQL Server) proporcionan características adicionales como la conmutación por error de réplicas para maximizar la disponibilidad.</span><span class="sxs-lookup"><span data-stu-id="61485-138">Some database systems (SQL Server) provide additional features such as failover replicas to maximize availability.</span></span>

<span data-ttu-id="61485-139">La separación de los almacenes de lectura y escritura permite también que cada uno de ellos pueda escalarse adecuadamente para adaptarse a la carga.</span><span class="sxs-lookup"><span data-stu-id="61485-139">Separation of the read and write stores also allows each to be scaled appropriately to match the load.</span></span> <span data-ttu-id="61485-140">Por ejemplo, los almacenes de lectura normalmente se encuentran con una carga mayor que los de escritura.</span><span class="sxs-lookup"><span data-stu-id="61485-140">For example, read stores typically encounter a much higher load than write stores.</span></span>

<span data-ttu-id="61485-141">Cuando el modelo de la consulta/lectura contiene datos desnormalizados (consulte [Patrón Materialized View](materialized-view.md)), se maximiza el rendimiento al leer los datos para cada una de las vistas de una aplicación o cuando se consultan los datos en el sistema.</span><span class="sxs-lookup"><span data-stu-id="61485-141">When the query/read model contains denormalized data (see [Materialized View pattern](materialized-view.md)), performance is maximized when reading data for each of the views in an application or when querying the data in the system.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="61485-142">Problemas y consideraciones</span><span class="sxs-lookup"><span data-stu-id="61485-142">Issues and considerations</span></span>

<span data-ttu-id="61485-143">Tenga en cuenta los puntos siguientes al decidir cómo implementar este patrón:</span><span class="sxs-lookup"><span data-stu-id="61485-143">Consider the following points when deciding how to implement this pattern:</span></span>

- <span data-ttu-id="61485-144">Si divide el almacén de datos en almacenes físicos independientes para las operaciones de lectura y escritura puede aumentar el rendimiento y seguridad de un sistema, pero también puede agregar complejidad en términos de resistencia y coherencia final.</span><span class="sxs-lookup"><span data-stu-id="61485-144">Dividing the data store into separate physical stores for read and write operations can increase the performance and security of a system, but it can add complexity in terms of resiliency and eventual consistency.</span></span> <span data-ttu-id="61485-145">El almacén de modelos de lectura debe actualizarse para reflejar los cambios del almacén de modelos de escritura, y puede ser difícil detectar cuándo un usuario ha emitido una solicitud basada en datos de lectura obsoletos, lo que significa que no se puede completar la operación.</span><span class="sxs-lookup"><span data-stu-id="61485-145">The read model store must be updated to reflect changes to the write model store, and it can be difficult to detect when a user has issued a request based on stale read data, which means that the operation can't be completed.</span></span>

    > <span data-ttu-id="61485-146">Para obtener una descripción de la coherencia final, consulte [Data Consistency Primer](https://msdn.microsoft.com/library/dn589800.aspx) (Manual básico de coherencia de datos).</span><span class="sxs-lookup"><span data-stu-id="61485-146">For a description of eventual consistency see the [Data Consistency Primer](https://msdn.microsoft.com/library/dn589800.aspx).</span></span>

- <span data-ttu-id="61485-147">Considere la posibilidad de aplicar CQRS a secciones limitadas del sistema donde será más valioso.</span><span class="sxs-lookup"><span data-stu-id="61485-147">Consider applying CQRS to limited sections of your system where it will be most valuable.</span></span>

- <span data-ttu-id="61485-148">Un enfoque habitual para la implementación de consistencia final consiste en usar Event Sourcing en combinación con CQRS para que el modelo de escritura sea un flujo de solo anexión de eventos controlados por la ejecución de comandos.</span><span class="sxs-lookup"><span data-stu-id="61485-148">A typical approach to deploying eventual consistency is to use event sourcing in conjunction with CQRS so that the write model is an append-only stream of events driven by execution of commands.</span></span> <span data-ttu-id="61485-149">Estos eventos se usan para actualizar las vistas materializadas que actúan como modelo de lectura.</span><span class="sxs-lookup"><span data-stu-id="61485-149">These events are used to update materialized views that act as the read model.</span></span> <span data-ttu-id="61485-150">Para más información, consulte [Event Sourcing y CQRS](/azure/architecture/patterns/cqrs#event-sourcing-and-cqrs).</span><span class="sxs-lookup"><span data-stu-id="61485-150">For more information see [Event Sourcing and CQRS](/azure/architecture/patterns/cqrs#event-sourcing-and-cqrs).</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="61485-151">Cuándo usar este patrón</span><span class="sxs-lookup"><span data-stu-id="61485-151">When to use this pattern</span></span>

<span data-ttu-id="61485-152">Use este patrón en las situaciones siguientes:</span><span class="sxs-lookup"><span data-stu-id="61485-152">Use this pattern in the following situations:</span></span>

- <span data-ttu-id="61485-153">Dominios colaborativos en los que se realizan varias operaciones en paralelo en los mismos datos.</span><span class="sxs-lookup"><span data-stu-id="61485-153">Collaborative domains where multiple operations are performed in parallel on the same data.</span></span> <span data-ttu-id="61485-154">CQRS le permite definir comandos con la suficiente granularidad para minimizar los conflictos de combinación en el nivel de dominio (cualquier conflicto que surja se podrá combinar mediante el comando), incluso cuando se actualiza lo que parece ser el mismo tipo de datos.</span><span class="sxs-lookup"><span data-stu-id="61485-154">CQRS allows you to define commands with enough granularity to minimize merge conflicts at the domain level (any conflicts that do arise can be merged by the command), even when updating what appears to be the same type of data.</span></span>

- <span data-ttu-id="61485-155">Interfaces de usuario basadas en tareas en las que se guía al usuario a través de un complejo proceso como una serie de pasos o con modelos de dominio complejos.</span><span class="sxs-lookup"><span data-stu-id="61485-155">Task-based user interfaces where users are guided through a complex process as a series of steps or with complex domain models.</span></span> <span data-ttu-id="61485-156">Esto también resulta útil para los equipos ya familiarizados con las técnicas de diseño controladas por dominios (DDD).</span><span class="sxs-lookup"><span data-stu-id="61485-156">Also, useful for teams already familiar with domain-driven design (DDD) techniques.</span></span> <span data-ttu-id="61485-157">El modelo de escritura tiene una pila de procesamiento de comandos completa con lógica de negocios, validación de entradas y validación empresarial para garantizar que todo es siempre coherente para cada uno de los agregados (cada clúster de objetos asociados que se trata como una unidad para los cambios de datos) del modelo de escritura.</span><span class="sxs-lookup"><span data-stu-id="61485-157">The write model has a full command-processing stack with business logic, input validation, and business validation to ensure that everything is always consistent for each of the aggregates (each cluster of associated objects treated as a unit for data changes) in the write model.</span></span> <span data-ttu-id="61485-158">El modelo de lectura no tiene ninguna lógica de negocios ni pila de validación y solo devuelve un DTO para su uso en un modelo de vista.</span><span class="sxs-lookup"><span data-stu-id="61485-158">The read model has no business logic or validation stack and just returns a DTO for use in a view model.</span></span> <span data-ttu-id="61485-159">El modelo de lectura tiene coherencia final con el modelo de escritura.</span><span class="sxs-lookup"><span data-stu-id="61485-159">The read model is eventually consistent with the write model.</span></span>

- <span data-ttu-id="61485-160">Los escenarios en los que el rendimiento de las lecturas de datos se debe ajustar de manera independiente a la del rendimiento de las escrituras de datos, especialmente cuando la relación de lectura/escritura es muy alta y se requiere un escalado horizontal.</span><span class="sxs-lookup"><span data-stu-id="61485-160">Scenarios where performance of data reads must be fine tuned separately from performance of data writes, especially when the read/write ratio is very high, and when horizontal scaling is required.</span></span> <span data-ttu-id="61485-161">Por ejemplo, en muchos sistemas el número de operaciones de lectura es muchas veces mayor que el número de operaciones de escritura.</span><span class="sxs-lookup"><span data-stu-id="61485-161">For example, in many systems the number of read operations is many times greater than the number of write operations.</span></span> <span data-ttu-id="61485-162">Para dar cabida a esto, considere la posibilidad de escalar horizontalmente el modelo de lectura, pero ejecutando el modelo de escritura en solo una o algunas instancias.</span><span class="sxs-lookup"><span data-stu-id="61485-162">To accommodate this, consider scaling out the read model, but running the write model on only one or a few instances.</span></span> <span data-ttu-id="61485-163">Un número reducido de instancias de modelo de escritura también ayuda a minimizar la aparición de conflictos de combinación.</span><span class="sxs-lookup"><span data-stu-id="61485-163">A small number of write model instances also helps to minimize the occurrence of merge conflicts.</span></span>

- <span data-ttu-id="61485-164">Escenarios en los que un equipo de desarrolladores pueda centrarse en el modelo de dominio complejo que forma parte del modelo de escritura, y otro equipo pueda centrarse en el modelo de lectura y en las interfaces de usuario.</span><span class="sxs-lookup"><span data-stu-id="61485-164">Scenarios where one team of developers can focus on the complex domain model that is part of the write model, and another team can focus on the read model and the user interfaces.</span></span>

- <span data-ttu-id="61485-165">Escenarios en los que se espera que el sistema evolucione con el tiempo y que podrían contener varias versiones del modelo, o en los que las reglas de negocio cambian con regularidad.</span><span class="sxs-lookup"><span data-stu-id="61485-165">Scenarios where the system is expected to evolve over time and might contain multiple versions of the model, or where business rules change regularly.</span></span>

- <span data-ttu-id="61485-166">Integración con otros sistemas, especialmente en combinación con Event Sourcing, en los que el error temporal de un subsistema no debería afectar a la disponibilidad de los demás.</span><span class="sxs-lookup"><span data-stu-id="61485-166">Integration with other systems, especially in combination with event sourcing, where the temporal failure of one subsystem shouldn't affect the availability of the others.</span></span>

<span data-ttu-id="61485-167">Este patrón no es recomendable en las situaciones siguientes:</span><span class="sxs-lookup"><span data-stu-id="61485-167">This pattern isn't recommended in the following situations:</span></span>

- <span data-ttu-id="61485-168">Allí donde el dominio o las reglas de negocio son simples.</span><span class="sxs-lookup"><span data-stu-id="61485-168">Where the domain or the business rules are simple.</span></span>

- <span data-ttu-id="61485-169">Allí donde una interfaz de usuario simple, de estilo CRUD, y las operaciones relacionadas de acceso a datos son suficientes.</span><span class="sxs-lookup"><span data-stu-id="61485-169">Where a simple CRUD-style user interface and the related data access operations are sufficient.</span></span>

- <span data-ttu-id="61485-170">Para la implementación en todo el sistema.</span><span class="sxs-lookup"><span data-stu-id="61485-170">For implementation across the whole system.</span></span> <span data-ttu-id="61485-171">Hay componentes específicos de un escenario de administración de datos general donde CQRS puede ser útil, pero puede agregar una complejidad considerable e innecesaria sin ser necesario.</span><span class="sxs-lookup"><span data-stu-id="61485-171">There are specific components of an overall data management scenario where CQRS can be useful, but it can add considerable and unnecessary complexity when it isn't required.</span></span>

## <a name="event-sourcing-and-cqrs"></a><span data-ttu-id="61485-172">Event Sourcing y CQRS</span><span class="sxs-lookup"><span data-stu-id="61485-172">Event Sourcing and CQRS</span></span>

<span data-ttu-id="61485-173">El patrón CQRS se utiliza a menudo junto con el patrón Event Sourcing.</span><span class="sxs-lookup"><span data-stu-id="61485-173">The CQRS pattern is often used along with the Event Sourcing pattern.</span></span> <span data-ttu-id="61485-174">Los sistemas basados en CQRS usan modelos de lectura y escritura de datos independientes, cada uno de ellos adaptado a las tareas correspondientes y, a menudo, ubicados en almacenes físicos independientes.</span><span class="sxs-lookup"><span data-stu-id="61485-174">CQRS-based systems use separate read and write data models, each tailored to relevant tasks and often located in physically separate stores.</span></span> <span data-ttu-id="61485-175">Cuando se usa con el patrón [Event Sourcing](event-sourcing.md), el almacén de eventos es el modelo de escritura y es el origen oficial de información.</span><span class="sxs-lookup"><span data-stu-id="61485-175">When used with the [Event Sourcing](event-sourcing.md) pattern, the store of events is the write model, and is the official source of information.</span></span> <span data-ttu-id="61485-176">El modelo de lectura de un sistema basado en CQRS proporciona vistas materializadas de los datos, normalmente como vistas altamente desnormalizadas.</span><span class="sxs-lookup"><span data-stu-id="61485-176">The read model of a CQRS-based system provides materialized views of the data, typically as highly denormalized views.</span></span> <span data-ttu-id="61485-177">Estas vistas se adaptan a las interfaces y muestran los requisitos de la aplicación, lo cual ayuda a maximizar la apariencia y el rendimiento de las consultas.</span><span class="sxs-lookup"><span data-stu-id="61485-177">These views are tailored to the interfaces and display requirements of the application, which helps to maximize both display and query performance.</span></span>

<span data-ttu-id="61485-178">Mediante el uso del flujo de eventos como almacén de escritura, en lugar de los datos reales en un momento dado, evita conflictos de actualización en un único agregado y maximiza el rendimiento y la escalabilidad.</span><span class="sxs-lookup"><span data-stu-id="61485-178">Using the stream of events as the write store, rather than the actual data at a point in time, avoids update conflicts on a single aggregate and maximizes performance and scalability.</span></span> <span data-ttu-id="61485-179">Los eventos se pueden utilizar para generar vistas materializadas de forma asincrónica de los datos que se usan para rellenar el almacén de lectura.</span><span class="sxs-lookup"><span data-stu-id="61485-179">The events can be used to asynchronously generate materialized views of the data that are used to populate the read store.</span></span>

<span data-ttu-id="61485-180">Dado que el almacén de eventos es el origen oficial de información, es posible eliminar las vistas materializadas y reproducir todos los eventos pasados para crear una nueva representación del estado actual cuando evolucione el sistema, o cuando deba cambiar el modelo de lectura.</span><span class="sxs-lookup"><span data-stu-id="61485-180">Because the event store is the official source of information, it is possible to delete the materialized views and replay all past events to create a new representation of the current state when the system evolves, or when the read model must change.</span></span> <span data-ttu-id="61485-181">Las vistas materializadas constituyen, en efecto, una memoria caché duradera de solo lectura de los datos.</span><span class="sxs-lookup"><span data-stu-id="61485-181">The materialized views are in effect a durable read-only cache of the data.</span></span>

<span data-ttu-id="61485-182">Cuando utilice CQRS en combinación con el patrón Event Sourcing, tenga en cuenta lo siguiente:</span><span class="sxs-lookup"><span data-stu-id="61485-182">When using CQRS combined with the Event Sourcing pattern, consider the following:</span></span>

- <span data-ttu-id="61485-183">Al igual que con cualquier sistema en el que los almacenes de escritura y lectura son independientes, los sistemas basados en este patrón solo tienen coherencia final.</span><span class="sxs-lookup"><span data-stu-id="61485-183">As with any system where the write and read stores are separate, systems based on this pattern are only eventually consistent.</span></span> <span data-ttu-id="61485-184">Habrá un cierto retraso entre la generación del evento y la actualización del almacén de datos.</span><span class="sxs-lookup"><span data-stu-id="61485-184">There will be some delay between the event being generated and the data store being updated.</span></span>

- <span data-ttu-id="61485-185">El patrón agrega complejidad porque se debe crear código para iniciar y controlar los eventos, y ensamblar o actualizar las vistas adecuadas o los objetos que necesitan las consultas o un modelo de lectura.</span><span class="sxs-lookup"><span data-stu-id="61485-185">The pattern adds complexity because code must be created to initiate and handle events, and assemble or update the appropriate views or objects required by queries or a read model.</span></span> <span data-ttu-id="61485-186">La complejidad del patrón CQRS cuando se usa con el patrón Event Sourcing puede dificultar una implementación correcta y requiere un enfoque diferente para el diseño de sistemas.</span><span class="sxs-lookup"><span data-stu-id="61485-186">The complexity of the CQRS pattern when used with the Event Sourcing pattern can make a successful implementation more difficult, and requires a different approach to designing systems.</span></span> <span data-ttu-id="61485-187">No obstante, Event Sourcing puede facilitar el modelado del dominio, y puede facilitar la recompilación de vistas o la creación de otras nuevas ya que se preserva la intención de los cambios de los datos.</span><span class="sxs-lookup"><span data-stu-id="61485-187">However, event sourcing can make it easier to model the domain, and makes it easier to rebuild views or create new ones because the intent of the changes in the data is preserved.</span></span>

- <span data-ttu-id="61485-188">La generación de vistas materializadas para su uso en el modelo de lectura o las proyecciones de los datos mediante la reproducción y control de eventos para entidades específicas o colecciones de estas, puede conllevar un tiempo de procesamiento y un uso de los recursos significativo.</span><span class="sxs-lookup"><span data-stu-id="61485-188">Generating materialized views for use in the read model or projections of the data by replaying and handling the events for specific entities or collections of entities can require significant processing time and resource usage.</span></span> <span data-ttu-id="61485-189">Esto es especialmente cierto si requiere la suma o el análisis de los valores durante períodos largos, ya que puede que se tengan que examinar todos los eventos asociados.</span><span class="sxs-lookup"><span data-stu-id="61485-189">This is especially true if it requires summation or analysis of values over long periods, because all the associated events might need to be examined.</span></span> <span data-ttu-id="61485-190">Puede resolver este problema mediante la implementación de instantáneas de los datos a intervalos programados, por ejemplo, un recuento total del número de veces que se ha producido una acción concreta o el estado actual de una entidad.</span><span class="sxs-lookup"><span data-stu-id="61485-190">Resolve this by implementing snapshots of the data at scheduled intervals, such as a total count of the number of a specific action that have occurred, or the current state of an entity.</span></span>

## <a name="example"></a><span data-ttu-id="61485-191">Ejemplo</span><span class="sxs-lookup"><span data-stu-id="61485-191">Example</span></span>

<span data-ttu-id="61485-192">El código siguiente muestra algunos extractos de un ejemplo de una implementación CQRS que usa definiciones diferentes para los modelos de lectura y de escritura.</span><span class="sxs-lookup"><span data-stu-id="61485-192">The following code shows some extracts from an example of a CQRS implementation that uses different definitions for the read and the write models.</span></span> <span data-ttu-id="61485-193">Las interfaces de los modelos no dictan ninguna característica de los almacenes de datos subyacentes, y pueden evolucionar y adaptarse de forma independiente ya que estas interfaces están separadas.</span><span class="sxs-lookup"><span data-stu-id="61485-193">The model interfaces don't dictate any features of the underlying data stores, and they can evolve and be fine-tuned independently because these interfaces are separated.</span></span>

<span data-ttu-id="61485-194">El código siguiente muestra la definición del modelo de lectura.</span><span class="sxs-lookup"><span data-stu-id="61485-194">The following code shows the read model definition.</span></span>

```csharp
// Query interface
namespace ReadModel
{
  public interface ProductsDao
  {
    ProductDisplay FindById(int productId);
    ICollection<ProductDisplay> FindByName(string name);
    ICollection<ProductInventory> FindOutOfStockProducts();
    ICollection<ProductDisplay> FindRelatedProducts(int productId);
  }

  public class ProductDisplay
  {
    public int Id { get; set; }
    public string Name { get; set; }
    public string Description { get; set; }
    public decimal UnitPrice { get; set; }
    public bool IsOutOfStock { get; set; }
    public double UserRating { get; set; }
  }

  public class ProductInventory
  {
    public int Id { get; set; }
    public string Name { get; set; }
    public int CurrentStock { get; set; }
  }
}
```

<span data-ttu-id="61485-195">El sistema permite a los usuarios valorar los productos.</span><span class="sxs-lookup"><span data-stu-id="61485-195">The system allows users to rate products.</span></span> <span data-ttu-id="61485-196">El código de la aplicación hace esto mediante el comando `RateProduct` que aparece en el código siguiente.</span><span class="sxs-lookup"><span data-stu-id="61485-196">The application code does this using the `RateProduct` command shown in the following code.</span></span>

```csharp
public interface ICommand
{
  Guid Id { get; }
}

public class RateProduct : ICommand
{
  public RateProduct()
  {
    this.Id = Guid.NewGuid();
  }
  public Guid Id { get; set; }
  public int ProductId { get; set; }
  public int Rating { get; set; }
  public int UserId {get; set; }
}
```

<span data-ttu-id="61485-197">El sistema usa la clase `ProductsCommandHandler` para controlar los comandos enviados por la aplicación.</span><span class="sxs-lookup"><span data-stu-id="61485-197">The system uses the `ProductsCommandHandler` class to handle commands sent by the application.</span></span> <span data-ttu-id="61485-198">Normalmente, los clientes envían comandos al dominio a través de un sistema de mensajería como, por ejemplo, una cola.</span><span class="sxs-lookup"><span data-stu-id="61485-198">Clients typically send commands to the domain through a messaging system such as a queue.</span></span> <span data-ttu-id="61485-199">El controlador de comandos acepta estos comandos e invoca los métodos de la interfaz de dominio.</span><span class="sxs-lookup"><span data-stu-id="61485-199">The command handler accepts these commands and invokes methods of the domain interface.</span></span> <span data-ttu-id="61485-200">La granularidad de cada comando está diseñada para reducir la posibilidad de que haya solicitudes en conflicto.</span><span class="sxs-lookup"><span data-stu-id="61485-200">The granularity of each command is designed to reduce the chance of conflicting requests.</span></span> <span data-ttu-id="61485-201">El código siguiente muestra un esquema de la clase `ProductsCommandHandler`.</span><span class="sxs-lookup"><span data-stu-id="61485-201">The following code shows an outline of the `ProductsCommandHandler` class.</span></span>

```csharp
public class ProductsCommandHandler :
    ICommandHandler<AddNewProduct>,
    ICommandHandler<RateProduct>,
    ICommandHandler<AddToInventory>,
    ICommandHandler<ConfirmItemShipped>,
    ICommandHandler<UpdateStockFromInventoryRecount>
{
  private readonly IRepository<Product> repository;

  public ProductsCommandHandler (IRepository<Product> repository)
  {
    this.repository = repository;
  }

  void Handle (AddNewProduct command)
  {
    ...
  }

  void Handle (RateProduct command)
  {
    var product = repository.Find(command.ProductId);
    if (product != null)
    {
      product.RateProduct(command.UserId, command.Rating);
      repository.Save(product);
    }
  }

  void Handle (AddToInventory command)
  {
    ...
  }

  void Handle (ConfirmItemsShipped command)
  {
    ...
  }

  void Handle (UpdateStockFromInventoryRecount command)
  {
    ...
  }
}
```

<span data-ttu-id="61485-202">El código siguiente muestra la interfaz `IProductsDomain` del modelo de escritura.</span><span class="sxs-lookup"><span data-stu-id="61485-202">The following code shows the `IProductsDomain` interface from the write model.</span></span>

```csharp
public interface IProductsDomain
{
  void AddNewProduct(int id, string name, string description, decimal price);
  void RateProduct(int userId, int rating);
  void AddToInventory(int productId, int quantity);
  void ConfirmItemsShipped(int productId, int quantity);
  void UpdateStockFromInventoryRecount(int productId, int updatedQuantity);
}
```

<span data-ttu-id="61485-203">Observe también cómo la interfaz `IProductsDomain` contiene métodos que tienen un significado en el dominio.</span><span class="sxs-lookup"><span data-stu-id="61485-203">Also notice how the `IProductsDomain` interface contains methods that have a meaning in the domain.</span></span> <span data-ttu-id="61485-204">Por lo general, en un entorno CRUD estos métodos tendrían nombres genéricos como `Save` o `Update`, y un DTO como único argumento.</span><span class="sxs-lookup"><span data-stu-id="61485-204">Typically, in a CRUD environment these methods would have generic names such as `Save` or `Update`, and have a DTO as the only argument.</span></span> <span data-ttu-id="61485-205">El enfoque CQRS se puede diseñar para satisfacer las necesidades de negocio y de sistemas de administración de inventario de esta organización.</span><span class="sxs-lookup"><span data-stu-id="61485-205">The CQRS approach can be designed to meet the needs of this organization's business and inventory management systems.</span></span>

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="61485-206">Orientación y patrones relacionados</span><span class="sxs-lookup"><span data-stu-id="61485-206">Related patterns and guidance</span></span>

<span data-ttu-id="61485-207">Los patrones y las directrices siguientes son útiles a la hora de implementar este patrón:</span><span class="sxs-lookup"><span data-stu-id="61485-207">The following patterns and guidance are useful when implementing this pattern:</span></span>

- <span data-ttu-id="61485-208">Para obtener una comparación de CQRS con otros estilos de arquitectura, consulte [Estilos de arquitectura](/azure/architecture/guide/architecture-styles/) y [Estilo de arquitectura CQRS](/azure/architecture/guide/architecture-styles/cqrs).</span><span class="sxs-lookup"><span data-stu-id="61485-208">For a comparison of CQRS with other architectural styles, see [Architecture styles](/azure/architecture/guide/architecture-styles/) and [CQRS architecture style](/azure/architecture/guide/architecture-styles/cqrs).</span></span>

- <span data-ttu-id="61485-209">[Data Consistency Primer](https://msdn.microsoft.com/library/dn589800.aspx) (Manual básico de coherencia de datos).</span><span class="sxs-lookup"><span data-stu-id="61485-209">[Data Consistency Primer](https://msdn.microsoft.com/library/dn589800.aspx).</span></span> <span data-ttu-id="61485-210">Explica los problemas que normalmente se producen debido a la coherencia final entre los almacenes de datos de lectura y de escritura cuando se usa el patrón CQRS y cómo se pueden resolver estos problemas.</span><span class="sxs-lookup"><span data-stu-id="61485-210">Explains the issues that are typically encountered due to eventual consistency between the read and write data stores when using the CQRS pattern, and how these issues can be resolved.</span></span>

- <span data-ttu-id="61485-211">[Guía de creación de particiones de datos](https://msdn.microsoft.com/library/dn589795.aspx).</span><span class="sxs-lookup"><span data-stu-id="61485-211">[Data Partitioning Guidance](https://msdn.microsoft.com/library/dn589795.aspx).</span></span> <span data-ttu-id="61485-212">Describe cómo se pueden dividir en particiones los almacenes de datos de lectura y escritura utilizados en el patrón CQRS, particiones que se pueden administrar y a las que se puede acceder por separado para mejorar la escalabilidad, reducir la contención y optimizar el rendimiento.</span><span class="sxs-lookup"><span data-stu-id="61485-212">Describes how the read and write data stores used in the CQRS pattern can be divided into partitions that can be managed and accessed separately to improve scalability, reduce contention, and optimize performance.</span></span>

- <span data-ttu-id="61485-213">[Patrón Event Sourcing](event-sourcing.md).</span><span class="sxs-lookup"><span data-stu-id="61485-213">[Event Sourcing Pattern](event-sourcing.md).</span></span> <span data-ttu-id="61485-214">Describe con más detalle como se puede usar Event Sourcing con el patrón CQRS para simplificar las tareas de dominios complejos al tiempo que se mejora el rendimiento, la escalabilidad y la capacidad de respuesta.</span><span class="sxs-lookup"><span data-stu-id="61485-214">Describes in more detail how Event Sourcing can be used with the CQRS pattern to simplify tasks in complex domains while improving performance, scalability, and responsiveness.</span></span> <span data-ttu-id="61485-215">También muestra cómo proporcionar coherencia a los datos transaccionales al tiempo que se mantienen registros de auditorías e historiales completos que pueden permitir acciones de compensación.</span><span class="sxs-lookup"><span data-stu-id="61485-215">As well as how to provide consistency for transactional data while maintaining full audit trails and history that can enable compensating actions.</span></span>

- <span data-ttu-id="61485-216">[Patrón Materialized View](materialized-view.md).</span><span class="sxs-lookup"><span data-stu-id="61485-216">[Materialized View Pattern](materialized-view.md).</span></span> <span data-ttu-id="61485-217">El modelo de lectura de una implementación de CQRS puede contener vistas materializadas de los datos del modelo de escritura o el modelo de lectura se puede utilizar para generar vistas materializadas.</span><span class="sxs-lookup"><span data-stu-id="61485-217">The read model of a CQRS implementation can contain materialized views of the write model data, or the read model can be used to generate materialized views.</span></span>

- <span data-ttu-id="61485-218">La guía de patrones y prácticas [CQRS Journey](https://aka.ms/cqrs) (Introducción a CQRS).</span><span class="sxs-lookup"><span data-stu-id="61485-218">The patterns & practices guide [CQRS Journey](https://aka.ms/cqrs).</span></span> <span data-ttu-id="61485-219">En concreto, [Introducing the Command Query Responsibility Segregation Pattern](https://msdn.microsoft.com/library/jj591573.aspx) (Introducción al patrón de segregación de responsabilidades de la consulta de comandos) explora el patrón y cuándo es útil, y [Epilogue: Lessons Learned](https://msdn.microsoft.com/library/jj591568.aspx) (Epílogo: lecciones aprendidas) le ayudarán a conocer algunos de los problemas que surgen al utilizar este patrón.</span><span class="sxs-lookup"><span data-stu-id="61485-219">In particular, [Introducing the Command Query Responsibility Segregation Pattern](https://msdn.microsoft.com/library/jj591573.aspx) explores the pattern and when it's useful, and [Epilogue: Lessons Learned](https://msdn.microsoft.com/library/jj591568.aspx) helps you understand some of the issues that come up when using this pattern.</span></span>

- <span data-ttu-id="61485-220">La entrada de blog sobre [CQRS de Martin Fowler](https://martinfowler.com/bliki/CQRS.html), que explica los aspectos básicos del patrón y contiene vínculos a otros recursos útiles.</span><span class="sxs-lookup"><span data-stu-id="61485-220">The post [CQRS by Martin Fowler](https://martinfowler.com/bliki/CQRS.html), which explains the basics of the pattern and links to other useful resources.</span></span>
