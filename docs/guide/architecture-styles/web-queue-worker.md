---
title: Estilo de arquitectura web-cola-trabajo
description: Describe las ventajas, los desafíos y los procedimientos recomendados para las arquitecturas web-cola-trabajo en Azure.
author: MikeWasson
ms.openlocfilehash: 545472e71ffcd43717ad24af0dc9218a221ca910
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2017
---
# <a name="web-queue-worker-architecture-style"></a><span data-ttu-id="25df1-103">Estilo de arquitectura web-cola-trabajo</span><span class="sxs-lookup"><span data-stu-id="25df1-103">Web-Queue-Worker architecture style</span></span>

<span data-ttu-id="25df1-104">Los componentes principales de esta arquitectura son un **front-end web** que atiende solicitudes de cliente y un **trabajo** que realiza tareas que consumen muchos recursos, flujos de trabajo de ejecución prolongada o trabajos por lotes.</span><span class="sxs-lookup"><span data-stu-id="25df1-104">The core components of this architecture are a **web front end** that serves client requests, and a **worker** that performs resource-intensive tasks, long-running workflows, or batch jobs.</span></span>  <span data-ttu-id="25df1-105">El front-end web se comunica con el trabajo a través de una **cola de mensajes**.</span><span class="sxs-lookup"><span data-stu-id="25df1-105">The web front end communicates with the worker through a **message queue**.</span></span>  

![](./images/web-queue-worker-logical.svg)

<span data-ttu-id="25df1-106">Otros componentes que normalmente se incorporan a esta arquitectura son:</span><span class="sxs-lookup"><span data-stu-id="25df1-106">Other components that are commonly incorporated into this architecture include:</span></span>

- <span data-ttu-id="25df1-107">Una o más bases de datos.</span><span class="sxs-lookup"><span data-stu-id="25df1-107">One or more databases.</span></span> 
- <span data-ttu-id="25df1-108">Una caché para almacenar los valores de la base de datos para las lecturas rápidas.</span><span class="sxs-lookup"><span data-stu-id="25df1-108">A cache to store values from the database for quick reads.</span></span>
- <span data-ttu-id="25df1-109">Una red CDN para atender contenido estático.</span><span class="sxs-lookup"><span data-stu-id="25df1-109">CDN to serve static content</span></span>
- <span data-ttu-id="25df1-110">Servicios remotos, como correo electrónico o un servicio de SMS.</span><span class="sxs-lookup"><span data-stu-id="25df1-110">Remote services, such as email or SMS service.</span></span> <span data-ttu-id="25df1-111">A menudo, son terceros los que los proporcionan.</span><span class="sxs-lookup"><span data-stu-id="25df1-111">Often these are provided by third parties.</span></span>
- <span data-ttu-id="25df1-112">Un proveedor de identidades para la autenticación.</span><span class="sxs-lookup"><span data-stu-id="25df1-112">Identity provider for authentication.</span></span>

<span data-ttu-id="25df1-113">La web y el trabajo son ambos sin estado.</span><span class="sxs-lookup"><span data-stu-id="25df1-113">The web and worker are both stateless.</span></span> <span data-ttu-id="25df1-114">El estado de sesión se puede almacenar en una memoria caché distribuida.</span><span class="sxs-lookup"><span data-stu-id="25df1-114">Session state can be stored in a distributed cache.</span></span> <span data-ttu-id="25df1-115">El trabajo realiza cualquier trabajo de ejecución prolongada de forma asincrónica.</span><span class="sxs-lookup"><span data-stu-id="25df1-115">Any long-running work is done asynchronously by the worker.</span></span> <span data-ttu-id="25df1-116">El trabajo pueden desencadenarlo mensajes en la cola o se puede ejecutar según una programación para el procesamiento por lotes.</span><span class="sxs-lookup"><span data-stu-id="25df1-116">The worker can be triggered by messages on the queue, or run on a schedule for batch processing.</span></span> <span data-ttu-id="25df1-117">El trabajo es un componente opcional.</span><span class="sxs-lookup"><span data-stu-id="25df1-117">The worker is an optional component.</span></span> <span data-ttu-id="25df1-118">Si no hay ninguna operación de ejecución prolongada, se puede omitir el trabajo.</span><span class="sxs-lookup"><span data-stu-id="25df1-118">If there are no long-running operations, the worker can be omitted.</span></span>  

<span data-ttu-id="25df1-119">El front-end puede consistir en una API web.</span><span class="sxs-lookup"><span data-stu-id="25df1-119">The front end might consist of a web API.</span></span> <span data-ttu-id="25df1-120">En el lado del cliente, una aplicación de página única que realiza llamadas AJAX o una aplicación cliente nativa puede consumir la API web.</span><span class="sxs-lookup"><span data-stu-id="25df1-120">On the client side, the web API can be consumed by a single-page application that makes AJAX calls, or by a native client application.</span></span>

## <a name="when-to-use-this-architecture"></a><span data-ttu-id="25df1-121">Cuándo utilizar esta arquitectura</span><span class="sxs-lookup"><span data-stu-id="25df1-121">When to use this architecture</span></span>

<span data-ttu-id="25df1-122">La arquitectura web-cola-trabajo se suele implementar mediante servicios de proceso administrados, ya sea Azure App Service o Azure Cloud Services.</span><span class="sxs-lookup"><span data-stu-id="25df1-122">The Web-Queue-Worker architecture is typically implemented using managed compute services, either Azure App Service or Azure Cloud Services.</span></span> 

<span data-ttu-id="25df1-123">Considere este estilo de arquitectura para:</span><span class="sxs-lookup"><span data-stu-id="25df1-123">Consider this architecture style for:</span></span>

- <span data-ttu-id="25df1-124">Aplicaciones con un dominio relativamente sencillo.</span><span class="sxs-lookup"><span data-stu-id="25df1-124">Applications with a relatively simple domain.</span></span>
- <span data-ttu-id="25df1-125">Aplicaciones con algunos flujos de trabajo de ejecución prolongada u operaciones por lotes.</span><span class="sxs-lookup"><span data-stu-id="25df1-125">Applications with some long-running workflows or batch operations.</span></span>
- <span data-ttu-id="25df1-126">Si desea utilizar servicios administrados, en lugar de infraestructura como servicio (IaaS).</span><span class="sxs-lookup"><span data-stu-id="25df1-126">When you want to use managed services, rather than infrastructure as a service (IaaS).</span></span>

## <a name="benefits"></a><span data-ttu-id="25df1-127">Ventajas</span><span class="sxs-lookup"><span data-stu-id="25df1-127">Benefits</span></span>

- <span data-ttu-id="25df1-128">Arquitectura relativamente sencilla que sea fácil de entender.</span><span class="sxs-lookup"><span data-stu-id="25df1-128">Relatively simple architecture that is easy to understand.</span></span>
- <span data-ttu-id="25df1-129">Fácil de implementar y administrar.</span><span class="sxs-lookup"><span data-stu-id="25df1-129">Easy to deploy and manage.</span></span>
- <span data-ttu-id="25df1-130">Separación clara de cuestiones.</span><span class="sxs-lookup"><span data-stu-id="25df1-130">Clear separation of concerns.</span></span>
- <span data-ttu-id="25df1-131">El front-end se desacopla del trabajo mediante mensajería asincrónica.</span><span class="sxs-lookup"><span data-stu-id="25df1-131">The front end is decoupled from the worker using asynchronous messaging.</span></span>
- <span data-ttu-id="25df1-132">El front-end y el trabajo se pueden escalar de forma independiente.</span><span class="sxs-lookup"><span data-stu-id="25df1-132">The front end and the worker can be scaled independently.</span></span>

## <a name="challenges"></a><span data-ttu-id="25df1-133">Desafíos</span><span class="sxs-lookup"><span data-stu-id="25df1-133">Challenges</span></span>

- <span data-ttu-id="25df1-134">Sin un diseño cuidado, el front-end y el trabajo se pueden convertir en componentes grandes y monolíticos que son difíciles de mantener y actualizar.</span><span class="sxs-lookup"><span data-stu-id="25df1-134">Without careful design, the front end and the worker can become large, monolithic components that are difficult to maintain and update.</span></span>
- <span data-ttu-id="25df1-135">Puede haber dependencias ocultas si el servidor front-end y el trabajo comparten esquemas de datos o módulos de código.</span><span class="sxs-lookup"><span data-stu-id="25df1-135">There may be hidden dependencies, if the front end and worker share data schemas or code modules.</span></span> 

## <a name="best-practices"></a><span data-ttu-id="25df1-136">Prácticas recomendadas</span><span class="sxs-lookup"><span data-stu-id="25df1-136">Best practices</span></span>

- <span data-ttu-id="25df1-137">Exponer una API bien diseñada al cliente.</span><span class="sxs-lookup"><span data-stu-id="25df1-137">Expose a well-designed API to the client.</span></span> <span data-ttu-id="25df1-138">Consulte [Procedimientos recomendados para el diseño de API][api-design].</span><span class="sxs-lookup"><span data-stu-id="25df1-138">See [API design best practices][api-design].</span></span>
- <span data-ttu-id="25df1-139">Utilizar el escalado automático para tratar los cambios en la carga.</span><span class="sxs-lookup"><span data-stu-id="25df1-139">Autoscale to handle changes in load.</span></span> <span data-ttu-id="25df1-140">Consulte [Procedimientos recomendados de escalado automático][autoscaling]</span><span class="sxs-lookup"><span data-stu-id="25df1-140">See [Autoscaling best practices][autoscaling].</span></span>
- <span data-ttu-id="25df1-141">Almacene en caché datos semiestáticos.</span><span class="sxs-lookup"><span data-stu-id="25df1-141">Cache semi-static data.</span></span> <span data-ttu-id="25df1-142">Consulte [Procedimientos recomendados para el almacenamiento en caché][caching].</span><span class="sxs-lookup"><span data-stu-id="25df1-142">See [Caching best practices][caching].</span></span>
- <span data-ttu-id="25df1-143">Utilizar una red CDN para hospedar contenido estático.</span><span class="sxs-lookup"><span data-stu-id="25df1-143">Use a CDN to host static content.</span></span> <span data-ttu-id="25df1-144">Consulte [Procedimientos recomendados para la red CDN][cdn].</span><span class="sxs-lookup"><span data-stu-id="25df1-144">See [CDN best practices][cdn].</span></span>
- <span data-ttu-id="25df1-145">Utilizar la persistencia de Polyglot cuando corresponda.</span><span class="sxs-lookup"><span data-stu-id="25df1-145">Use polyglot persistence when appropriate.</span></span> <span data-ttu-id="25df1-146">Consulte [Uso del mejor almacén de datos para el trabajo][polyglot].</span><span class="sxs-lookup"><span data-stu-id="25df1-146">See [Use the best data store for the job][polyglot].</span></span>
- <span data-ttu-id="25df1-147">Crear particiones de datos para mejorar la escalabilidad, reducir la contención y optimizar el rendimiento.</span><span class="sxs-lookup"><span data-stu-id="25df1-147">Partition data to improve scalability, reduce contention, and optimize performance.</span></span> <span data-ttu-id="25df1-148">Consulte [Procedimientos recomendados para las particiones de datos][data-partition].</span><span class="sxs-lookup"><span data-stu-id="25df1-148">See [Data partitioning best practices][data-partition].</span></span>


## <a name="web-queue-worker-on-azure-app-service"></a><span data-ttu-id="25df1-149">Web-cola-trabajo en Azure App Service</span><span class="sxs-lookup"><span data-stu-id="25df1-149">Web-Queue-Worker on Azure App Service</span></span>

<span data-ttu-id="25df1-150">En esta sección se describe una arquitectura recomendada web-cola-trabajo que utiliza Azure App Service.</span><span class="sxs-lookup"><span data-stu-id="25df1-150">This section describes a recommended Web-Queue-Worker architecture that uses Azure App Service.</span></span> 

![](./images/web-queue-worker-physical.png)

<span data-ttu-id="25df1-151">El front-end se implementa como una aplicación web de Azure App Service y el trabajo se implementa como WebJob.</span><span class="sxs-lookup"><span data-stu-id="25df1-151">The front end is implemented as an Azure App Service web app, and the worker is implemented as a WebJob.</span></span> <span data-ttu-id="25df1-152">La aplicación web y WebJob están asociados a un plan de App Service que proporciona las instancias de máquina virtual.</span><span class="sxs-lookup"><span data-stu-id="25df1-152">The web app and the WebJob are both associated with an App Service plan that provides the VM instances.</span></span> 

<span data-ttu-id="25df1-153">Puede usar las colas de Azure Service Bus o de Azure Storage para la cola de mensajes.</span><span class="sxs-lookup"><span data-stu-id="25df1-153">You can use either Azure Service Bus or Azure Storage queues for the message queue.</span></span> <span data-ttu-id="25df1-154">(El diagrama muestra una cola de Azure Storage).</span><span class="sxs-lookup"><span data-stu-id="25df1-154">(The diagram shows an Azure Storage queue.)</span></span>

<span data-ttu-id="25df1-155">Azure Redis Cache almacena el estado de sesión y otros datos que necesitan acceso de baja latencia.</span><span class="sxs-lookup"><span data-stu-id="25df1-155">Azure Redis Cache stores session state and other data that needs low latency access.</span></span>

<span data-ttu-id="25df1-156">La red CDN de Azure se usa para almacenar en caché contenido estático como imágenes, CSS o HTML.</span><span class="sxs-lookup"><span data-stu-id="25df1-156">Azure CDN is used to cache static content such as images, CSS, or HTML.</span></span>

<span data-ttu-id="25df1-157">Para el almacenamiento, elija las tecnologías de almacenamiento que mejor se adapten a las necesidades de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="25df1-157">For storage, choose the storage technologies that best fit the needs of the application.</span></span> <span data-ttu-id="25df1-158">Puede usar varias tecnologías de almacenamiento (persistencia de Polyglot).</span><span class="sxs-lookup"><span data-stu-id="25df1-158">You might use multiple storage technologies (polyglot persistence).</span></span> <span data-ttu-id="25df1-159">Para ilustrar esta idea, el diagrama muestra de Azure SQL Database y Azure Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="25df1-159">To illustrate this idea, the diagram shows Azure SQL Database and Azure Cosmos DB.</span></span>  

<span data-ttu-id="25df1-160">Para más información, consulte [Arquitectura de referencia de aplicación web de App Service][scalable-web-app].</span><span class="sxs-lookup"><span data-stu-id="25df1-160">For more details, see [App Service web application reference architecture][scalable-web-app].</span></span>

### <a name="additional-considerations"></a><span data-ttu-id="25df1-161">Consideraciones adicionales</span><span class="sxs-lookup"><span data-stu-id="25df1-161">Additional considerations</span></span>

- <span data-ttu-id="25df1-162">No todas las transacciones tienen que pasar a través de la cola y el trabajo para ir al almacenamiento.</span><span class="sxs-lookup"><span data-stu-id="25df1-162">Not every transaction has to go through the queue and worker to storage.</span></span> <span data-ttu-id="25df1-163">El front-end web puede realizar directamente operaciones de lectura/escritura sencillas.</span><span class="sxs-lookup"><span data-stu-id="25df1-163">The web front end can perform simple read/write operations directly.</span></span> <span data-ttu-id="25df1-164">Los trabajos están diseñados para tareas que consumen muchos recursos o flujos de trabajo de ejecución prolongada.</span><span class="sxs-lookup"><span data-stu-id="25df1-164">Workers are designed for resource-intensive tasks or long-running workflows.</span></span> <span data-ttu-id="25df1-165">En algunos casos, es posible que no necesite ningún trabajo en absoluto.</span><span class="sxs-lookup"><span data-stu-id="25df1-165">In some cases, you might not need a worker at all.</span></span>

- <span data-ttu-id="25df1-166">Use la característica de escalado automático integrado de App Service para escalar horizontalmente el número de instancias de máquina virtual.</span><span class="sxs-lookup"><span data-stu-id="25df1-166">Use the built-in autoscale feature of App Service to scale out the number of VM instances.</span></span> <span data-ttu-id="25df1-167">Si la carga de la aplicación sigue patrones predecibles, use el escalado automático basado en programación.</span><span class="sxs-lookup"><span data-stu-id="25df1-167">If the load on the application follows predictable patterns, use schedule-based autoscale.</span></span> <span data-ttu-id="25df1-168">Si la carga es imprevisible, use reglas de escalado automático basadas en métricas.</span><span class="sxs-lookup"><span data-stu-id="25df1-168">If the load is unpredictable, use metrics-based autoscaling rules.</span></span>      

- <span data-ttu-id="25df1-169">Considere la posibilidad de poner la aplicación web y WebJob en planes de App Service separados.</span><span class="sxs-lookup"><span data-stu-id="25df1-169">Consider putting the web app and the WebJob into separate App Service plans.</span></span> <span data-ttu-id="25df1-170">De este modo, se hospedan en instancias de máquina virtual distintas y se pueden escalar de forma independiente.</span><span class="sxs-lookup"><span data-stu-id="25df1-170">That way, they are hosted on separate VM instances and can be scaled independently.</span></span> 

- <span data-ttu-id="25df1-171">Use planes de App Service independientes para producción y prueba.</span><span class="sxs-lookup"><span data-stu-id="25df1-171">Use separate App Service plans for production and testing.</span></span> <span data-ttu-id="25df1-172">En caso contrario, si usa el mismo plan para producción y prueba, significa que las pruebas se ejecutan en las máquinas virtuales de producción.</span><span class="sxs-lookup"><span data-stu-id="25df1-172">Otherwise, if you use the same plan for production and testing, it means your tests are running on your production VMs.</span></span>

- <span data-ttu-id="25df1-173">Utilice las ranuras de implementación para administrar implementaciones.</span><span class="sxs-lookup"><span data-stu-id="25df1-173">Use deployment slots to manage deployments.</span></span> <span data-ttu-id="25df1-174">Esto le permite implementar una versión actualizada en un espacio de ensayo e intercambiar luego a la nueva versión.</span><span class="sxs-lookup"><span data-stu-id="25df1-174">This lets you to deploy an updated version to a staging slot, then swap over to the new version.</span></span> <span data-ttu-id="25df1-175">También le permite volver a la versión anterior si ha habido un problema con la actualización.</span><span class="sxs-lookup"><span data-stu-id="25df1-175">It also lets you swap back to the previous version, if there was a problem with the update.</span></span>

<!-- links -->

[api-design]: ../../best-practices/api-design.md
[autoscaling]: ../../best-practices/auto-scaling.md
[caching]: ../../best-practices/caching.md
[cdn]: ../../best-practices/cdn.md
[data-partition]: ../../best-practices/data-partitioning.md
[polyglot]: ../design-principles/use-the-best-data-store.md
[scalable-web-app]: ../../reference-architectures/app-service-web-app/scalable-web-app.md