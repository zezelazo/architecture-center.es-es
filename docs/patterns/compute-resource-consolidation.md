---
title: Compute Resource Consolidation
description: "Consolida varias tareas u operaciones en una sola unidad de cálculo."
keywords: "Patrón de diseño"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories: design-implementation
ms.openlocfilehash: 85191fc630549559f8a1395e5a8622a7a6140a2d
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2017
---
# <a name="compute-resource-consolidation-pattern"></a><span data-ttu-id="ee815-104">Patrón Compute Resource Consolidation</span><span class="sxs-lookup"><span data-stu-id="ee815-104">Compute Resource Consolidation pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="ee815-105">Consolida varias tareas u operaciones en una sola unidad de cálculo.</span><span class="sxs-lookup"><span data-stu-id="ee815-105">Consolidate multiple tasks or operations into a single computational unit.</span></span> <span data-ttu-id="ee815-106">Esta consolidación puede aumentar el uso de recursos de proceso y reducir los costes y la sobrecarga de administración asociados a la realización del procesamiento de procesos en aplicaciones hospedadas en la nube.</span><span class="sxs-lookup"><span data-stu-id="ee815-106">This can increase compute resource utilization, and reduce the costs and management overhead associated with performing compute processing in cloud-hosted applications.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="ee815-107">Contexto y problema</span><span class="sxs-lookup"><span data-stu-id="ee815-107">Context and problem</span></span>

<span data-ttu-id="ee815-108">Con frecuencia una aplicación en la nube implementa diversas operaciones.</span><span class="sxs-lookup"><span data-stu-id="ee815-108">A cloud application often implements a variety of operations.</span></span> <span data-ttu-id="ee815-109">En algunas soluciones, resulta conveniente seguir el principio de diseño de separación inicial de problemas y dividir estas operaciones en unidades de cálculo independientes que se hospedan e implementan de manera individual (por ejemplo, como aplicaciones web de App Service independientes, instancias independientes de Virtual Machines o roles de servicios en la nube independientes).</span><span class="sxs-lookup"><span data-stu-id="ee815-109">In some solutions it makes sense to follow the design principle of separation of concerns initially, and divide these operations into separate computational units that are hosted and deployed individually (for example, as separate App Service web apps, separate Virtual Machines, or separate Cloud Service roles).</span></span> <span data-ttu-id="ee815-110">Sin embargo, aunque esta estrategia puede ayudar a simplificar el diseño lógico de la solución, la implementación de un gran número de unidades de cálculo como parte de la misma aplicación puede aumentar los costes de hospedaje en tiempo de ejecución y dificultar la administración del sistema.</span><span class="sxs-lookup"><span data-stu-id="ee815-110">However, although this strategy can help simplify the logical design of the solution, deploying a large number of computational units as part of the same application can increase runtime hosting costs and make management of the system more complex.</span></span>

<span data-ttu-id="ee815-111">Por ejemplo, en la ilustración se muestra la estructura simplificada de una solución hospedada en la nube que se implementa mediante más de una unidad de cálculo.</span><span class="sxs-lookup"><span data-stu-id="ee815-111">As an example, the figure shows the simplified structure of a cloud-hosted solution that is implemented using more than one computational unit.</span></span> <span data-ttu-id="ee815-112">Cada unidad de cálculo se ejecuta en su propio entorno virtual.</span><span class="sxs-lookup"><span data-stu-id="ee815-112">Each computational unit runs in its own virtual environment.</span></span> <span data-ttu-id="ee815-113">Cada función se ha implementado como una tarea independiente (etiquetadas de la tarea A-a la tarea E) que se ejecuta en su propia unidad de cálculo.</span><span class="sxs-lookup"><span data-stu-id="ee815-113">Each function has been implemented as a separate task (labeled Task A through Task E) running in its own computational unit.</span></span>

![Ejecución de tareas en un entorno de nube mediante un conjunto de unidades de cálculo dedicadas](./_images/compute-resource-consolidation-diagram.png)


<span data-ttu-id="ee815-115">Cada unidad de cálculo consume recursos facturables, incluso cuando está inactiva o se usa poco.</span><span class="sxs-lookup"><span data-stu-id="ee815-115">Each computational unit consumes chargeable resources, even when it's idle or lightly used.</span></span> <span data-ttu-id="ee815-116">Por lo tanto, esta solución no es siempre la más rentable.</span><span class="sxs-lookup"><span data-stu-id="ee815-116">Therefore, this isn't always the most cost-effective solution.</span></span>

<span data-ttu-id="ee815-117">En Azure, este problema se aplica a los roles de una instancia de Cloud Services, App Services y Virtual Machines.</span><span class="sxs-lookup"><span data-stu-id="ee815-117">In Azure, this concern applies to roles in a Cloud Service, App Services, and Virtual Machines.</span></span> <span data-ttu-id="ee815-118">Estos elementos se ejecutan en su propio entorno virtual.</span><span class="sxs-lookup"><span data-stu-id="ee815-118">These items run in their own virtual environment.</span></span> <span data-ttu-id="ee815-119">La ejecución de una colección de roles, sitios web o máquinas virtuales independientes que están diseñados para realizar un conjunto de operaciones bien definidas, pero que necesitan comunicarse y colaborar como parte de una única solución, puede suponer un uso ineficiente de los recursos.</span><span class="sxs-lookup"><span data-stu-id="ee815-119">Running a collection of separate roles, websites, or virtual machines that are designed to perform a set of well-defined operations, but that need to communicate and cooperate as part of a single solution, can be an inefficient use of resources.</span></span>

## <a name="solution"></a><span data-ttu-id="ee815-120">Solución</span><span class="sxs-lookup"><span data-stu-id="ee815-120">Solution</span></span>

<span data-ttu-id="ee815-121">Para ayudar a reducir los costes, aumentar la utilización, mejorar la velocidad de comunicación y reducir la administración, es posible consolidar varias tareas u operaciones en una sola unidad de cálculo.</span><span class="sxs-lookup"><span data-stu-id="ee815-121">To help reduce costs, increase utilization, improve communication speed, and reduce management it's possible to consolidate multiple tasks or operations into a single computational unit.</span></span>

<span data-ttu-id="ee815-122">Las tareas se pueden agrupar según criterios basados en las características que proporciona el entorno y los costes asociados con estas características.</span><span class="sxs-lookup"><span data-stu-id="ee815-122">Tasks can be grouped according to criteria based on the features provided by the environment and the costs associated with these features.</span></span> <span data-ttu-id="ee815-123">Un enfoque común consiste en buscar las tareas que tienen un perfil similar en lo relativo a sus requisitos de escalabilidad, duración y procesamiento.</span><span class="sxs-lookup"><span data-stu-id="ee815-123">A common approach is to look for tasks that have a similar profile concerning their scalability, lifetime, and processing requirements.</span></span> <span data-ttu-id="ee815-124">Cuando se agrupan juntas se pueden escalar como una unidad.</span><span class="sxs-lookup"><span data-stu-id="ee815-124">Grouping these together allows them to scale as a unit.</span></span> <span data-ttu-id="ee815-125">La elasticidad que proporcionan muchos entornos de nube permite que se inicien y detengan instancias adicionales de una unidad de cálculo según la carga de trabajo.</span><span class="sxs-lookup"><span data-stu-id="ee815-125">The elasticity provided by many cloud environments enables additional instances of a computational unit to be started and stopped according to the workload.</span></span> <span data-ttu-id="ee815-126">Por ejemplo, Azure ofrece escalado automático que se puede aplicar a los roles de una instancia de Cloud Services, App Services o Virtual Machines.</span><span class="sxs-lookup"><span data-stu-id="ee815-126">For example, Azure provides autoscaling that you can apply to roles in a Cloud Service, App Services, and Virtual Machines.</span></span> <span data-ttu-id="ee815-127">Para más información, consulte [Guía de escalado automático](https://msdn.microsoft.com/library/dn589774.aspx).</span><span class="sxs-lookup"><span data-stu-id="ee815-127">For more information, see [Autoscaling Guidance](https://msdn.microsoft.com/library/dn589774.aspx).</span></span>

<span data-ttu-id="ee815-128">Como ejemplo de contador para mostrar cómo se puede usar la escalabilidad para determinar qué operaciones no deben agruparse, tenga en cuenta las dos tareas siguientes:</span><span class="sxs-lookup"><span data-stu-id="ee815-128">As a counter example to show how scalability can be used to determine which operations shouldn't be grouped together, consider the following two tasks:</span></span>

- <span data-ttu-id="ee815-129">La tarea 1 sondea los mensajes poco frecuentes e independientes del tiempo enviados a una cola.</span><span class="sxs-lookup"><span data-stu-id="ee815-129">Task 1 polls for infrequent, time-insensitive messages sent to a queue.</span></span>
- <span data-ttu-id="ee815-130">La tarea 2 controla las ráfagas de gran volumen del tráfico de red.</span><span class="sxs-lookup"><span data-stu-id="ee815-130">Task 2 handles high-volume bursts of network traffic.</span></span>

<span data-ttu-id="ee815-131">La segunda tarea requiere elasticidad que puede implicar iniciar y detener un gran número de instancias de la unidad de cálculo.</span><span class="sxs-lookup"><span data-stu-id="ee815-131">The second task requires elasticity that can involve starting and stopping a large number of instances of the computational unit.</span></span> <span data-ttu-id="ee815-132">Aplicar la misma escala a la primera tarea simplemente supondría que más tareas escuchan mensajes poco frecuentes en la misma cola, lo que es una pérdida de recursos.</span><span class="sxs-lookup"><span data-stu-id="ee815-132">Applying the same scaling to the first task would simply result in more tasks listening for infrequent messages on the same queue, and is a waste of resources.</span></span>

<span data-ttu-id="ee815-133">En muchos entornos de nube, es posible especificar los recursos disponibles para una unidad de cálculo en términos del número de núcleos de CPU, la memoria, el espacio en disco, etc.</span><span class="sxs-lookup"><span data-stu-id="ee815-133">In many cloud environments it's possible to specify the resources available to a computational unit in terms of the number of CPU cores, memory, disk space, and so on.</span></span> <span data-ttu-id="ee815-134">Por lo general, cuantos más recursos se especifican, mayor es el coste.</span><span class="sxs-lookup"><span data-stu-id="ee815-134">Generally, the more resources specified, the greater the cost.</span></span> <span data-ttu-id="ee815-135">Para ahorrar dinero, es importante maximizar el trabajo que realiza una unidad de cálculo y no permitir que se vuelva inactiva durante un período de tiempo prolongado.</span><span class="sxs-lookup"><span data-stu-id="ee815-135">To save money, it's important to maximize the work an expensive computational unit performs, and not let it become inactive for an extended period.</span></span>

<span data-ttu-id="ee815-136">Si hay tareas que requieren gran cantidad de potencia de CPU en ráfagas cortas, considere la posibilidad de consolidar estos elementos en una sola unidad de cálculo que proporcione la potencia necesaria.</span><span class="sxs-lookup"><span data-stu-id="ee815-136">If there are tasks that require a great deal of CPU power in short bursts, consider consolidating these into a single computational unit that provides the necessary power.</span></span> <span data-ttu-id="ee815-137">Sin embargo, es importante equilibrar esta necesidad de mantener ocupados los recursos costosos con la contención que podría producirse si se someten a una sobrecarga.</span><span class="sxs-lookup"><span data-stu-id="ee815-137">However, it's important to balance this need to keep expensive resources busy against the contention that could occur if they are over stressed.</span></span> <span data-ttu-id="ee815-138">Las tareas de proceso intensivo de larga duración no deben compartir la misma unidad de cálculo, por ejemplo.</span><span class="sxs-lookup"><span data-stu-id="ee815-138">Long-running, compute-intensive tasks shouldn't share the same computational unit, for example.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="ee815-139">Problemas y consideraciones</span><span class="sxs-lookup"><span data-stu-id="ee815-139">Issues and considerations</span></span>

<span data-ttu-id="ee815-140">Tenga en cuenta los puntos siguientes al implementar este patrón:</span><span class="sxs-lookup"><span data-stu-id="ee815-140">Consider the following points when implementing this pattern:</span></span>

<span data-ttu-id="ee815-141">**Escalabilidad y elasticidad**.</span><span class="sxs-lookup"><span data-stu-id="ee815-141">**Scalability and elasticity**.</span></span> <span data-ttu-id="ee815-142">Muchas soluciones de nube implementan escalabilidad y elasticidad en el nivel de la unidad de cálculo mediante el inicio y la detención de instancias de unidades.</span><span class="sxs-lookup"><span data-stu-id="ee815-142">Many cloud solutions implement scalability and elasticity at the level of the computational unit by starting and stopping instances of units.</span></span> <span data-ttu-id="ee815-143">Evite agrupar tareas que tengan requisitos de escalabilidad en conflicto en la misma unidad de cálculo.</span><span class="sxs-lookup"><span data-stu-id="ee815-143">Avoid grouping tasks that have conflicting scalability requirements in the same computational unit.</span></span>

<span data-ttu-id="ee815-144">**Vigencia**.</span><span class="sxs-lookup"><span data-stu-id="ee815-144">**Lifetime**.</span></span> <span data-ttu-id="ee815-145">La infraestructura de nube recicla periódicamente el entorno virtual que hospeda una unidad de cálculo.</span><span class="sxs-lookup"><span data-stu-id="ee815-145">The cloud infrastructure periodically recycles the virtual environment that hosts a computational unit.</span></span> <span data-ttu-id="ee815-146">Cuando hay muchas tareas de larga ejecución dentro de una unidad de cálculo, podría ser necesario configurar la unidad para evitar que se recicle hasta que estas tareas hayan terminado.</span><span class="sxs-lookup"><span data-stu-id="ee815-146">When there are many long-running tasks inside a computational unit, it might be necessary to configure the unit to prevent it from being recycled until these tasks have finished.</span></span> <span data-ttu-id="ee815-147">Otra alternativa es diseñar las tareas mediante un enfoque de punto de comprobación que les permite detenerse correctamente y continuar en el punto en que se interrumpieron cuando la unidad de cálculo se reinicie.</span><span class="sxs-lookup"><span data-stu-id="ee815-147">Alternatively, design the tasks by using a check-pointing approach that enables them to stop cleanly, and continue at the point they were interrupted when the computational unit is restarted.</span></span>

<span data-ttu-id="ee815-148">**Ritmo de lanzamientos**.</span><span class="sxs-lookup"><span data-stu-id="ee815-148">**Release cadence**.</span></span> <span data-ttu-id="ee815-149">Si la implementación o la configuración de una tarea cambian con frecuencia, podría ser necesario detener la unidad de cálculo que hospeda el código actualizado, volver a configurarla e implementarla y, luego, reiniciarla.</span><span class="sxs-lookup"><span data-stu-id="ee815-149">If the implementation or configuration of a task changes frequently, it might be necessary to stop the computational unit hosting the updated code, reconfigure and redeploy the unit, and then restart it.</span></span> <span data-ttu-id="ee815-150">Este proceso también exigirá que todas las demás tareas dentro de la misma unidad de cálculo se detengan, se vuelvan a implementar y se reinicien.</span><span class="sxs-lookup"><span data-stu-id="ee815-150">This process will also require that all other tasks within the same computational unit are stopped, redeployed, and restarted.</span></span>

<span data-ttu-id="ee815-151">**Seguridad**.</span><span class="sxs-lookup"><span data-stu-id="ee815-151">**Security**.</span></span> <span data-ttu-id="ee815-152">Las tareas de la misma unidad de cálculo podrían compartir el mismo contexto de seguridad y acceder a los mismos recursos.</span><span class="sxs-lookup"><span data-stu-id="ee815-152">Tasks in the same computational unit might share the same security context and be able to access the same resources.</span></span> <span data-ttu-id="ee815-153">Debe haber un alto grado de confianza entre las tareas, y la seguridad de que una tarea no va a dañar o afectar a otra de manera negativa.</span><span class="sxs-lookup"><span data-stu-id="ee815-153">There must be a high degree of trust between the tasks, and confidence that one task isn't going to corrupt or adversely affect another.</span></span> <span data-ttu-id="ee815-154">Además, al aumentar el número de tareas que se ejecutan en una unidad de cálculo, aumenta la superficie expuesta a ataques de la unidad.</span><span class="sxs-lookup"><span data-stu-id="ee815-154">Additionally, increasing the number of tasks running in a computational unit increases the attack surface of the unit.</span></span> <span data-ttu-id="ee815-155">Cada tarea solo es tan segura como lo sea la que presenta la mayoría de las vulnerabilidades.</span><span class="sxs-lookup"><span data-stu-id="ee815-155">Each task is only as secure as the one with the most vulnerabilities.</span></span>

<span data-ttu-id="ee815-156">**Tolerancia a errores**.</span><span class="sxs-lookup"><span data-stu-id="ee815-156">**Fault tolerance**.</span></span> <span data-ttu-id="ee815-157">Si se produce un error en una tarea de una unidad de cálculo o la tarea se comporta de forma anómala, puede afectar a las demás tareas que se ejecutan en la misma unidad.</span><span class="sxs-lookup"><span data-stu-id="ee815-157">If one task in a computational unit fails or behaves abnormally, it can affect the other tasks running within the same unit.</span></span> <span data-ttu-id="ee815-158">Por ejemplo, una tarea que no se inicia correctamente puede provocar que la lógica completa de inicio de la unidad de cálculo produzca un error e impida que se ejecuten otras tareas de la misma unidad.</span><span class="sxs-lookup"><span data-stu-id="ee815-158">For example, if one task fails to start correctly it can cause the entire startup logic for the computational unit to fail, and prevent other tasks in the same unit from running.</span></span>

<span data-ttu-id="ee815-159">**Contención**.</span><span class="sxs-lookup"><span data-stu-id="ee815-159">**Contention**.</span></span> <span data-ttu-id="ee815-160">Evite la introducción de contención entre tareas que compiten por los recursos de la misma unidad de cálculo.</span><span class="sxs-lookup"><span data-stu-id="ee815-160">Avoid introducing contention between tasks that compete for resources in the same computational unit.</span></span> <span data-ttu-id="ee815-161">Lo ideal es que las tareas que comparten la misma unidad de cálculo presenten características diferentes en el uso de los recursos.</span><span class="sxs-lookup"><span data-stu-id="ee815-161">Ideally, tasks that share the same computational unit should exhibit different resource utilization characteristics.</span></span> <span data-ttu-id="ee815-162">Por ejemplo, dos tareas de proceso intensivo no deberían residir probablemente en la misma unidad de cálculo, y tampoco dos tareas que consuman grandes cantidades de memoria.</span><span class="sxs-lookup"><span data-stu-id="ee815-162">For example, two compute-intensive tasks should probably not reside in the same computational unit, and neither should two tasks that consume large amounts of memory.</span></span> <span data-ttu-id="ee815-163">Sin embargo, mezclar una tarea de proceso intensivo y una tarea que requiere una gran cantidad de memoria es una combinación factible.</span><span class="sxs-lookup"><span data-stu-id="ee815-163">However, mixing a compute intensive task with a task that requires a large amount of memory is a workable combination.</span></span>

> [!NOTE]
>  <span data-ttu-id="ee815-164">Considere la posibilidad de consolidar los recursos de proceso solo en sistemas que han estado en producción durante un período de tiempo, de forma que los operadores y los desarrolladores puedan supervisar los sistemas y crear un _mapa térmico_ que identifique el modo en que cada tarea usa recursos diferentes.</span><span class="sxs-lookup"><span data-stu-id="ee815-164">Consider consolidating compute resources only for a system that's been in production for a period of time so that operators and developers can monitor the system and create a _heat map_ that identifies how each task utilizes differing resources.</span></span> <span data-ttu-id="ee815-165">Este mapa se puede utilizar para determinar qué tareas son buenas candidatas a compartir recursos de proceso.</span><span class="sxs-lookup"><span data-stu-id="ee815-165">This map can be used to determine which tasks are good candidates for sharing compute resources.</span></span>

<span data-ttu-id="ee815-166">**Complejidad**.</span><span class="sxs-lookup"><span data-stu-id="ee815-166">**Complexity**.</span></span> <span data-ttu-id="ee815-167">Combinar varias tareas en una sola unidad de cálculo agrega complejidad al código de la unidad, de modo que posiblemente sea más difícil de probar, depurar y mantener.</span><span class="sxs-lookup"><span data-stu-id="ee815-167">Combining multiple tasks into a single computational unit adds complexity to the code in the unit, possibly making it more difficult to test, debug, and maintain.</span></span>

<span data-ttu-id="ee815-168">**Arquitectura lógica estable**.</span><span class="sxs-lookup"><span data-stu-id="ee815-168">**Stable logical architecture**.</span></span> <span data-ttu-id="ee815-169">Diseñe e implemente el código de cada tarea de forma que no sea necesario cambiarlo, incluso si lo hace el entorno físico en el que se ejecuta la tarea.</span><span class="sxs-lookup"><span data-stu-id="ee815-169">Design and implement the code in each task so that it shouldn't need to change, even if the physical environment the task runs in does change.</span></span>

<span data-ttu-id="ee815-170">**Otras estrategias**.</span><span class="sxs-lookup"><span data-stu-id="ee815-170">**Other strategies**.</span></span> <span data-ttu-id="ee815-171">La consolidación de recursos de proceso es solo una manera de ayudar a reducir los costes asociados a la ejecución simultánea de varias tareas.</span><span class="sxs-lookup"><span data-stu-id="ee815-171">Consolidating compute resources is only one way to help reduce costs associated with running multiple tasks concurrently.</span></span> <span data-ttu-id="ee815-172">Requiere un planeamiento y una supervisión rigurosos a fin de asegurarse de que siga siendo un enfoque efectivo.</span><span class="sxs-lookup"><span data-stu-id="ee815-172">It requires careful planning and monitoring to ensure that it remains an effective approach.</span></span> <span data-ttu-id="ee815-173">Otras estrategias podrían ser más adecuadas, según la naturaleza del trabajo y dónde se encuentren los usuarios que ejecutan estas tareas.</span><span class="sxs-lookup"><span data-stu-id="ee815-173">Other strategies might be more appropriate, depending on the nature of the work and where the users these tasks are running are located.</span></span> <span data-ttu-id="ee815-174">Por ejemplo, la descomposición funcional de la carga de trabajo (como se describe en el documento [Compute Partitioning Guidance](https://msdn.microsoft.com/library/dn589773.aspx)) (Guía de creación de particiones de proceso) podría ser una mejor opción.</span><span class="sxs-lookup"><span data-stu-id="ee815-174">For example, functional decomposition of the workload (as described by the [Compute Partitioning Guidance](https://msdn.microsoft.com/library/dn589773.aspx)) might be a better option.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="ee815-175">Cuándo usar este patrón</span><span class="sxs-lookup"><span data-stu-id="ee815-175">When to use this pattern</span></span>

<span data-ttu-id="ee815-176">Use este patrón con tareas que no sean rentables si se ejecutan en sus propias unidades de cálculo.</span><span class="sxs-lookup"><span data-stu-id="ee815-176">Use this pattern for tasks that are not cost effective if they run in their own computational units.</span></span> <span data-ttu-id="ee815-177">Si una tarea gasta mucho de su tiempo de inactividad, ejecutar esta tarea en una unidad dedicada puede resultar costoso.</span><span class="sxs-lookup"><span data-stu-id="ee815-177">If a task spends much of its time idle, running this task in a dedicated unit can be expensive.</span></span>

<span data-ttu-id="ee815-178">Este patrón podría no ser adecuado con tareas que realizan operaciones críticas de tolerancia a errores, o tareas que procesan datos de alto secreto o privados y requieren su propio contexto de seguridad.</span><span class="sxs-lookup"><span data-stu-id="ee815-178">This pattern might not be suitable for tasks that perform critical fault-tolerant operations, or tasks that process highly sensitive or private data and require their own security context.</span></span> <span data-ttu-id="ee815-179">Estas tareas se deben ejecutar en su propio entorno aislado, en una unidad de cálculo independiente.</span><span class="sxs-lookup"><span data-stu-id="ee815-179">These tasks should run in their own isolated environment, in a separate computational unit.</span></span>

## <a name="example"></a><span data-ttu-id="ee815-180">Ejemplo</span><span class="sxs-lookup"><span data-stu-id="ee815-180">Example</span></span>

<span data-ttu-id="ee815-181">Al crear un servicio en la nube en Azure, es posible consolidar el procesamiento que realizan varias tareas en un único rol.</span><span class="sxs-lookup"><span data-stu-id="ee815-181">When building a cloud service on Azure, it’s possible to consolidate the processing performed by multiple tasks into a single role.</span></span> <span data-ttu-id="ee815-182">Normalmente, es el rol de trabajo el encargado de realizar las tareas de procesamiento en segundo plano o asincrónicas.</span><span class="sxs-lookup"><span data-stu-id="ee815-182">Typically this is a worker role that performs background or asynchronous processing tasks.</span></span>

> <span data-ttu-id="ee815-183">En algunos casos, es posible incluir dichas tareas en el rol web.</span><span class="sxs-lookup"><span data-stu-id="ee815-183">In some cases it's possible to include background or asynchronous processing tasks in the web role.</span></span> <span data-ttu-id="ee815-184">Esta técnica ayuda reducir los costes y a simplificar la implementación, pero puede afectar a la escalabilidad y la capacidad de respuesta de la interfaz de acceso público proporcionada por el rol web.</span><span class="sxs-lookup"><span data-stu-id="ee815-184">This technique helps to reduce costs and simplify deployment, although it can impact the scalability and responsiveness of the public-facing interface provided by the web role.</span></span> <span data-ttu-id="ee815-185">El artículo [Combining Multiple Azure Worker Roles into an Azure Web Role](http://www.31a2ba2a-b718-11dc-8314-0800200c9a66.com/2012/02/combining-multiple-azure-worker-roles.html) (Combinación de varios roles de trabajo de Azure en un rol web de Azure) contiene una descripción detallada de la implementación de tareas de procesamiento en segundo plano o asincrónicas en un rol web.</span><span class="sxs-lookup"><span data-stu-id="ee815-185">The article [Combining Multiple Azure Worker Roles into an Azure Web Role](http://www.31a2ba2a-b718-11dc-8314-0800200c9a66.com/2012/02/combining-multiple-azure-worker-roles.html) contains a detailed description of implementing background or asynchronous processing tasks in a web role.</span></span>

<span data-ttu-id="ee815-186">El rol es responsable de iniciar y detener las tareas.</span><span class="sxs-lookup"><span data-stu-id="ee815-186">The role is responsible for starting and stopping the tasks.</span></span> <span data-ttu-id="ee815-187">Cuando el controlador de tejido de Azure carga un rol, se genera el evento `Start` para el rol.</span><span class="sxs-lookup"><span data-stu-id="ee815-187">When the Azure fabric controller loads a role, it raises the `Start` event for the role.</span></span> <span data-ttu-id="ee815-188">Puede invalidar el método `OnStart` de `WebRole` o la clase `WorkerRole` para tratar este evento, quizás para inicializar los datos y otros recursos de los que dependen las tareas de este método.</span><span class="sxs-lookup"><span data-stu-id="ee815-188">You can override the `OnStart` method of the `WebRole` or `WorkerRole` class to handle this event, perhaps to initialize the data and other resources the tasks in this method depend on.</span></span>

<span data-ttu-id="ee815-189">Cuando se completa el método `OnStart `, el rol puede comenzar a responder a las solicitudes.</span><span class="sxs-lookup"><span data-stu-id="ee815-189">When the `OnStart `method completes, the role can start responding to requests.</span></span> <span data-ttu-id="ee815-190">Puede encontrar más información e instrucciones sobre el uso de los métodos `OnStart` y `Run` en un rol en la sección [Application Startup Processes](https://msdn.microsoft.com/library/ff803371.aspx#sec16) (Procesos de inicio de aplicaciones) de la guía de patrones y prácticas [Moving Applications to the Cloud](https://msdn.microsoft.com/library/ff728592.aspx) (Movimiento de aplicaciones a la nube).</span><span class="sxs-lookup"><span data-stu-id="ee815-190">You can find more information and guidance about using the `OnStart` and `Run` methods in a role in the [Application Startup Processes](https://msdn.microsoft.com/library/ff803371.aspx#sec16) section in the patterns & practices guide [Moving Applications to the Cloud](https://msdn.microsoft.com/library/ff728592.aspx).</span></span>

> <span data-ttu-id="ee815-191">Mantenga el código del método `OnStart` lo más conciso posible.</span><span class="sxs-lookup"><span data-stu-id="ee815-191">Keep the code in the `OnStart` method as concise as possible.</span></span> <span data-ttu-id="ee815-192">Azure no impone ningún límite sobre el tiempo que tarda este método en completarse, pero el rol no podrá comenzar a responder a las solicitudes que recibe hasta que lo haga.</span><span class="sxs-lookup"><span data-stu-id="ee815-192">Azure doesn't impose any limit on the time taken for this method to complete, but the role won't be able to start responding to network requests sent to it until this method completes.</span></span>

<span data-ttu-id="ee815-193">Cuando el método `OnStart` ha finalizado, el rol ejecuta el método `Run`.</span><span class="sxs-lookup"><span data-stu-id="ee815-193">When the `OnStart` method has finished, the role executes the `Run` method.</span></span> <span data-ttu-id="ee815-194">En este momento, el controlador de tejido puede empezar a enviar solicitudes al rol.</span><span class="sxs-lookup"><span data-stu-id="ee815-194">At this point, the fabric controller can start sending requests to the role.</span></span>

<span data-ttu-id="ee815-195">Coloque el código que realmente crea las tareas en el método `Run`.</span><span class="sxs-lookup"><span data-stu-id="ee815-195">Place the code that actually creates the tasks in the `Run` method.</span></span> <span data-ttu-id="ee815-196">Tenga en cuenta que el método `Run` define la duración de la instancia de rol.</span><span class="sxs-lookup"><span data-stu-id="ee815-196">Note that the `Run` method defines the lifetime of the role instance.</span></span> <span data-ttu-id="ee815-197">Cuando este método finalice, el controlador de tejido se encargará de que el rol se apague.</span><span class="sxs-lookup"><span data-stu-id="ee815-197">When this method completes, the fabric controller will arrange for the role to be shut down.</span></span>

<span data-ttu-id="ee815-198">Cuando un rol se apaga o se recicla, el controlador de tejido impide que se reciban más solicitudes entrantes desde el equilibrador de carga y genera el evento `Stop`.</span><span class="sxs-lookup"><span data-stu-id="ee815-198">When a role shuts down or is recycled, the fabric controller prevents any more incoming requests being received from the load balancer and raises the `Stop` event.</span></span> <span data-ttu-id="ee815-199">Este evento se puede capturar si se invalida el método `OnStop` del rol y se realizan los arreglos necesarios antes de que termine el rol.</span><span class="sxs-lookup"><span data-stu-id="ee815-199">You can capture this event by overriding the `OnStop` method of the role and perform any tidying up required before the role terminates.</span></span>

> <span data-ttu-id="ee815-200">Las acciones llevadas a cabo en el método `OnStop` se deben completar al cabo de cinco minutos (o 30 segundos si va a usar el emulador de Azure en un equipo local).</span><span class="sxs-lookup"><span data-stu-id="ee815-200">Any actions performed in the `OnStop` method must be completed within five minutes (or 30 seconds if you are using the Azure emulator on a local computer).</span></span> <span data-ttu-id="ee815-201">En caso contrario, el controlador de tejido de Azure da por supuesto que el rol está parado y lo obliga a detenerse.</span><span class="sxs-lookup"><span data-stu-id="ee815-201">Otherwise the Azure fabric controller assumes that the role has stalled and will force it to stop.</span></span>

<span data-ttu-id="ee815-202">Las tareas se inician mediante el método `Run` que espera a que finalicen las tareas.</span><span class="sxs-lookup"><span data-stu-id="ee815-202">The tasks are started by the `Run` method that waits for the tasks to complete.</span></span> <span data-ttu-id="ee815-203">Las tareas implementan la lógica de negocios del servicio en la nube, y pueden responder a los mensajes publicados en el rol a través del equilibrador de carga de Azure.</span><span class="sxs-lookup"><span data-stu-id="ee815-203">The tasks implement the business logic of the cloud service, and can respond to messages posted to the role through the Azure load balancer.</span></span> <span data-ttu-id="ee815-204">En la ilustración se muestra el ciclo de vida de las tareas y los recursos de un rol en un servicio en la nube de Azure.</span><span class="sxs-lookup"><span data-stu-id="ee815-204">The figure shows the lifecycle of tasks and resources in a role in an Azure cloud service.</span></span>

![El ciclo de vida de las tareas y los recursos de un rol en un servicio en la nube de Azure](./_images/compute-resource-consolidation-lifecycle.png)


<span data-ttu-id="ee815-206">El archivo _WorkerRole.cs_ del proyecto _ComputeResourceConsolidation.Worker_ muestra un ejemplo de cómo podría implementar este patrón en un servicio en la nube de Azure.</span><span class="sxs-lookup"><span data-stu-id="ee815-206">The _WorkerRole.cs_ file in the _ComputeResourceConsolidation.Worker_ project shows an example of how you might implement this pattern in an Azure cloud service.</span></span>

> <span data-ttu-id="ee815-207">El proyecto _ComputeResourceConsolidation.Worker_ forma parte de la solución _ComputeResourceConsolidation_ disponible en [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/compute-resource-consolidation).</span><span class="sxs-lookup"><span data-stu-id="ee815-207">The _ComputeResourceConsolidation.Worker_ project is part of the _ComputeResourceConsolidation_ solution available for download from [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/compute-resource-consolidation).</span></span>

<span data-ttu-id="ee815-208">Los métodos `MyWorkerTask1` y `MyWorkerTask2` muestran cómo realizar diferentes tareas en el mismo rol de trabajo.</span><span class="sxs-lookup"><span data-stu-id="ee815-208">The `MyWorkerTask1` and the `MyWorkerTask2` methods illustrate how to perform different tasks within the same worker role.</span></span> <span data-ttu-id="ee815-209">El código siguiente muestra `MyWorkerTask1`.</span><span class="sxs-lookup"><span data-stu-id="ee815-209">The following code shows `MyWorkerTask1`.</span></span> <span data-ttu-id="ee815-210">Se trata de una tarea sencilla que se suspende durante 30 segundos y, a continuación, genera un mensaje de seguimiento.</span><span class="sxs-lookup"><span data-stu-id="ee815-210">This is a simple task that sleeps for 30 seconds and then outputs a trace message.</span></span> <span data-ttu-id="ee815-211">Este proceso se repite hasta que se cancela la tarea.</span><span class="sxs-lookup"><span data-stu-id="ee815-211">It repeats this process until the task is canceled.</span></span> <span data-ttu-id="ee815-212">El código de `MyWorkerTask2` es similar.</span><span class="sxs-lookup"><span data-stu-id="ee815-212">The code in `MyWorkerTask2` is similar.</span></span>

```csharp
// A sample worker role task.
private static async Task MyWorkerTask1(CancellationToken ct)
{
  // Fixed interval to wake up and check for work and/or do work.
  var interval = TimeSpan.FromSeconds(30);

  try
  {
    while (!ct.IsCancellationRequested)
    {
      // Wake up and do some background processing if not canceled.
      // TASK PROCESSING CODE HERE
      Trace.TraceInformation("Doing Worker Task 1 Work");

      // Go back to sleep for a period of time unless asked to cancel.
      // Task.Delay will throw an OperationCanceledException when canceled.
      await Task.Delay(interval, ct);
    }
  }
  catch (OperationCanceledException)
  {
    // Expect this exception to be thrown in normal circumstances or check
    // the cancellation token. If the role instances are shutting down, a
    // cancellation request will be signaled.
    Trace.TraceInformation("Stopping service, cancellation requested");

    // Rethrow the exception.
    throw;
  }
}
```

> <span data-ttu-id="ee815-213">El código de ejemplo muestra una implementación común de un proceso en segundo plano.</span><span class="sxs-lookup"><span data-stu-id="ee815-213">The sample code shows a common implementation of a background process.</span></span> <span data-ttu-id="ee815-214">En una aplicación real, puede seguir esta misma estructura, salvo que debe colocar su propia lógica de procesamiento en el cuerpo del bucle que espera la solicitud de cancelación.</span><span class="sxs-lookup"><span data-stu-id="ee815-214">In a real world application you can follow this same structure, except that you should place your own processing logic in the body of the loop that waits for the cancellation request.</span></span>

<span data-ttu-id="ee815-215">Después de que el rol de trabajo ha inicializado los recursos que usa, el método `Run` inicia las dos tareas de manera simultánea, como se muestra aquí.</span><span class="sxs-lookup"><span data-stu-id="ee815-215">After the worker role has initialized the resources it uses, the `Run` method starts the two tasks concurrently, as shown here.</span></span>

```csharp
/// <summary>
/// The cancellation token source use to cooperatively cancel running tasks
/// </summary>
private readonly CancellationTokenSource cts = new CancellationTokenSource();

/// <summary>
/// List of running tasks on the role instance
/// </summary>
private readonly List<Task> tasks = new List<Task>();

// RoleEntry Run() is called after OnStart().
// Returning from Run() will cause a role instance to recycle.
public override void Run()
{
  // Start worker tasks and add to the task list
  tasks.Add(MyWorkerTask1(cts.Token));
  tasks.Add(MyWorkerTask2(cts.Token));

  foreach (var worker in this.workerTasks)
  {
      this.tasks.Add(worker);
  }

  Trace.TraceInformation("Worker host tasks started");
  // The assumption is that all tasks should remain running and not return,
  // similar to role entry Run() behavior.
  try
  {
    Task.WaitAll(tasks.ToArray());
  }
  catch (AggregateException ex)
  {
    Trace.TraceError(ex.Message);

    // If any of the inner exceptions in the aggregate exception
    // are not cancellation exceptions then re-throw the exception.
    ex.Handle(innerEx => (innerEx is OperationCanceledException));
  }

  // If there wasn't a cancellation request, stop all tasks and return from Run()
  // An alternative to canceling and returning when a task exits would be to
  // restart the task.
  if (!cts.IsCancellationRequested)
  {
    Trace.TraceInformation("Task returned without cancellation request");
    Stop(TimeSpan.FromMinutes(5));
  }
}
...
```

<span data-ttu-id="ee815-216">En este ejemplo, el método `Run` espera a que se completen las tareas.</span><span class="sxs-lookup"><span data-stu-id="ee815-216">In this example, the `Run` method waits for tasks to be completed.</span></span> <span data-ttu-id="ee815-217">Si se cancela una tarea, el método `Run` da por supuesto que el rol se está cerrando y espera a que se cancele el resto de tareas antes de finalizar (espera un máximo de cinco minutos antes de terminar).</span><span class="sxs-lookup"><span data-stu-id="ee815-217">If a task is canceled, the `Run` method assumes that the role is being shut down and waits for the remaining tasks to be canceled before finishing (it waits for a maximum of five minutes before terminating).</span></span> <span data-ttu-id="ee815-218">Si se produce un error en una tarea debido a una excepción esperada, el método `Run` cancela la tarea.</span><span class="sxs-lookup"><span data-stu-id="ee815-218">If a task fails due to an expected exception, the `Run` method cancels the task.</span></span>

> <span data-ttu-id="ee815-219">Podría implementar estrategias de supervisión y tratamiento de excepciones más completas en el método `Run`, como reiniciar tareas que han producido error o incluir código que permite que el rol detenga e inicie tareas individuales.</span><span class="sxs-lookup"><span data-stu-id="ee815-219">You could implement more comprehensive monitoring and exception handling strategies in the `Run` method such as restarting tasks that have failed, or including code that enables the role to stop and start individual tasks.</span></span>

<span data-ttu-id="ee815-220">El método `Stop` que se muestra en el código siguiente se llama cuando el controlador de tejido cierra la instancia de rol (se invoca desde el método `OnStop`).</span><span class="sxs-lookup"><span data-stu-id="ee815-220">The `Stop` method shown in the following code is called when the fabric controller shuts down the role instance (it's invoked from the `OnStop` method).</span></span> <span data-ttu-id="ee815-221">El código detiene cada tarea de forma correcta mediante su cancelación.</span><span class="sxs-lookup"><span data-stu-id="ee815-221">The code stops each task gracefully by canceling it.</span></span> <span data-ttu-id="ee815-222">Si cualquier tarea tarda más de cinco minutos en completarse, el procesamiento de la cancelación en el método `Stop` deja de esperar y el rol se termina.</span><span class="sxs-lookup"><span data-stu-id="ee815-222">If any task takes more than five minutes to complete, the cancellation processing in the `Stop` method ceases waiting and the role is terminated.</span></span>

```csharp
// Stop running tasks and wait for tasks to complete before returning
// unless the timeout expires.
private void Stop(TimeSpan timeout)
{
  Trace.TraceInformation("Stop called. Canceling tasks.");
  // Cancel running tasks.
  cts.Cancel();

  Trace.TraceInformation("Waiting for canceled tasks to finish and return");

  // Wait for all the tasks to complete before returning. Note that the
  // emulator currently allows 30 seconds and Azure allows five
  // minutes for processing to complete.
  try
  {
    Task.WaitAll(tasks.ToArray(), timeout);
  }
  catch (AggregateException ex)
  {
    Trace.TraceError(ex.Message);

    // If any of the inner exceptions in the aggregate exception
    // are not cancellation exceptions then rethrow the exception.
    ex.Handle(innerEx => (innerEx is OperationCanceledException));
  }
}
```

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="ee815-223">Orientación y patrones relacionados</span><span class="sxs-lookup"><span data-stu-id="ee815-223">Related patterns and guidance</span></span>

<span data-ttu-id="ee815-224">Los patrones y las directrices siguientes también pueden ser importantes a la hora de implementar este modelo:</span><span class="sxs-lookup"><span data-stu-id="ee815-224">The following patterns and guidance might also be relevant when implementing this pattern:</span></span>

- <span data-ttu-id="ee815-225">[Guía de escalado automático](https://msdn.microsoft.com/library/dn589774.aspx).</span><span class="sxs-lookup"><span data-stu-id="ee815-225">[Autoscaling Guidance](https://msdn.microsoft.com/library/dn589774.aspx).</span></span> <span data-ttu-id="ee815-226">El escalado automático puede utilizarse para iniciar y detener instancias de servicio que hospedan recursos de cálculo, según la demanda prevista de procesamiento.</span><span class="sxs-lookup"><span data-stu-id="ee815-226">Autoscaling can be used to start and stop instances of service hosting computational resources, depending on the anticipated demand for processing.</span></span>

- <span data-ttu-id="ee815-227">[Compute Partitioning Guidance](https://msdn.microsoft.com/library/dn589773.aspx) (Orientación sobre la creación de particiones de proceso).</span><span class="sxs-lookup"><span data-stu-id="ee815-227">[Compute Partitioning Guidance](https://msdn.microsoft.com/library/dn589773.aspx).</span></span> <span data-ttu-id="ee815-228">Se describe cómo asignar los servicios y componentes de un servicio en la nube de tal forma que ayude a reducir los costes de ejecución mientras se mantiene la escalabilidad, el rendimiento, la disponibilidad y la seguridad del servicio.</span><span class="sxs-lookup"><span data-stu-id="ee815-228">Describes how to allocate the services and components in a cloud service in a way that helps to minimize running costs while maintaining the scalability, performance, availability, and security of the service.</span></span>

- <span data-ttu-id="ee815-229">Este patrón incluye una [aplicación de ejemplo](https://github.com/mspnp/cloud-design-patterns/tree/master/compute-resource-consolidation) descargable.</span><span class="sxs-lookup"><span data-stu-id="ee815-229">This pattern includes a downloadable [sample application](https://github.com/mspnp/cloud-design-patterns/tree/master/compute-resource-consolidation).</span></span>
