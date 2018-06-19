---
title: Patrón Gateway Routing
description: Enruta las solicitudes a varios servicios mediante un solo punto de conexión.
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: ea0bc4d31b745043a7ac3afb277dfc46d87ff109
ms.sourcegitcommit: 85334ab0ccb072dac80de78aa82bcfa0f0044d3f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2018
ms.locfileid: "35252607"
---
# <a name="gateway-routing-pattern"></a><span data-ttu-id="a7975-103">Patrón Gateway Routing</span><span class="sxs-lookup"><span data-stu-id="a7975-103">Gateway Routing pattern</span></span>

<span data-ttu-id="a7975-104">Enruta las solicitudes a varios servicios mediante un solo punto de conexión.</span><span class="sxs-lookup"><span data-stu-id="a7975-104">Route requests to multiple services using a single endpoint.</span></span> <span data-ttu-id="a7975-105">Este patrón es útil cuando desea exponer varios servicios en un único punto de conexión y enrutarlos al servicio adecuado en función de la solicitud.</span><span class="sxs-lookup"><span data-stu-id="a7975-105">This pattern is useful when you wish to expose multiple services on a single endpoint and route to the appropriate service based on the request.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="a7975-106">Contexto y problema</span><span class="sxs-lookup"><span data-stu-id="a7975-106">Context and problem</span></span>

<span data-ttu-id="a7975-107">Cuando un cliente debe consumir varios servicios, puede resultar difícil configurar un punto de conexión diferente para cada servicio y hacer que el cliente administre cada punto de conexión.</span><span class="sxs-lookup"><span data-stu-id="a7975-107">When a client needs to consume multiple services, setting up a separate endpoint for each service and having the client manage each endpoint can be challenging.</span></span> <span data-ttu-id="a7975-108">Por ejemplo, una aplicación de comercio electrónico podría proporcionar servicios como búsqueda, revisiones, carro, finalización de la compra e historial de pedidos.</span><span class="sxs-lookup"><span data-stu-id="a7975-108">For example, an e-commerce application might provide services such as search, reviews, cart, checkout, and order history.</span></span> <span data-ttu-id="a7975-109">Cada servicio tiene una API diferente con la que el cliente debe interactuar, y el cliente debe conocer cada punto de conexión para conectarse a los servicios.</span><span class="sxs-lookup"><span data-stu-id="a7975-109">Each service has a different API that the client must interact with, and the client must know about each endpoint in order to connect to the services.</span></span> <span data-ttu-id="a7975-110">Si cambia una API, también se debe actualizar el cliente.</span><span class="sxs-lookup"><span data-stu-id="a7975-110">If an API changes, the client must be updated as well.</span></span> <span data-ttu-id="a7975-111">Si refactoriza un servicio en dos o más servicios independientes, el código debe cambiar en el servicio y en el cliente.</span><span class="sxs-lookup"><span data-stu-id="a7975-111">If you refactor a service into two or more separate services, the code must change in both the service and the client.</span></span>

## <a name="solution"></a><span data-ttu-id="a7975-112">Solución</span><span class="sxs-lookup"><span data-stu-id="a7975-112">Solution</span></span>

<span data-ttu-id="a7975-113">Coloque una puerta de enlace delante de un conjunto de aplicaciones, servicios o implementaciones.</span><span class="sxs-lookup"><span data-stu-id="a7975-113">Place a gateway in front of a set of applications, services, or deployments.</span></span> <span data-ttu-id="a7975-114">Use el enrutamiento del nivel de aplicación 7 para enrutar la solicitud a las instancias adecuadas.</span><span class="sxs-lookup"><span data-stu-id="a7975-114">Use application Layer 7 routing to route the request to the appropriate instances.</span></span>

<span data-ttu-id="a7975-115">Con este patrón, la aplicación cliente solo necesita conocer y comunicarse con un único punto de conexión.</span><span class="sxs-lookup"><span data-stu-id="a7975-115">With this pattern, the client application only needs to know about and communicate with a single endpoint.</span></span> <span data-ttu-id="a7975-116">Si un servicio se consolida o descompone, no se tiene que actualizar el cliente necesariamente.</span><span class="sxs-lookup"><span data-stu-id="a7975-116">If a service is consolidated or decomposed, the client does not necessarily require updating.</span></span> <span data-ttu-id="a7975-117">Puede seguir realizando solicitudes a la puerta de enlace y solo cambia el enrutamiento.</span><span class="sxs-lookup"><span data-stu-id="a7975-117">It can continue making requests to the gateway, and only the routing changes.</span></span>

<span data-ttu-id="a7975-118">Una puerta de enlace también ofrece la posibilidad de abstraer los servicios back-end de los clientes. De esta forma, las llamadas de clientes pueden simplificarse y se permiten cambios en estos servicios detrás de la puerta de enlace.</span><span class="sxs-lookup"><span data-stu-id="a7975-118">A gateway also lets you abstract backend services from the clients, allowing you to keep client calls simple while enabling changes in the backend services behind the gateway.</span></span> <span data-ttu-id="a7975-119">Las llamadas de clientes se pueden enrutar a cualquier servicio necesario para administrar el comportamiento esperado de los clientes, lo que le permite agregar, dividir y reorganizar los servicios detrás de la puerta de enlace sin cambiar el cliente.</span><span class="sxs-lookup"><span data-stu-id="a7975-119">Client calls can be routed to whatever service or services need to handle the expected client behavior, allowing you to add, split, and reorganize services behind the gateway without changing the client.</span></span>

![](./_images/gateway-routing.png)
 
<span data-ttu-id="a7975-120">Este patrón también puede ayudar con la implementación, ya que le permite administrar cómo se implementan las actualizaciones en los usuarios.</span><span class="sxs-lookup"><span data-stu-id="a7975-120">This pattern can also help with deployment, by allowing you to manage how updates are rolled out to users.</span></span> <span data-ttu-id="a7975-121">Cuando se implemente una nueva versión de su servicio, se puede hacer en paralelo con la versión existente.</span><span class="sxs-lookup"><span data-stu-id="a7975-121">When a new version of your service is deployed, it can be deployed in parallel with the existing version.</span></span> <span data-ttu-id="a7975-122">El enrutamiento le permite controlar qué versión del servicio se presenta a los clientes, lo que permite usar diversas estrategias de lanzamiento: lanzamientos de actualizaciones incrementales, paralelas o completas.</span><span class="sxs-lookup"><span data-stu-id="a7975-122">Routing lets you control what version of the service is presented to the clients, giving you the flexibility to use various release strategies, whether incremental, parallel, or complete rollouts of updates.</span></span> <span data-ttu-id="a7975-123">Cualquier problema detectado una vez que se implementa el nuevo servicio se puede revertir rápidamente mediante un cambio en la configuración en la puerta de enlace, sin afectar a los clientes.</span><span class="sxs-lookup"><span data-stu-id="a7975-123">Any issues discovered after the new service is deployed can be quickly reverted by making a configuration change at the gateway, without affecting clients.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="a7975-124">Problemas y consideraciones</span><span class="sxs-lookup"><span data-stu-id="a7975-124">Issues and considerations</span></span>

- <span data-ttu-id="a7975-125">El servicio de puerta de enlace puede introducir un único punto de error.</span><span class="sxs-lookup"><span data-stu-id="a7975-125">The gateway service may introduce a single point of failure.</span></span> <span data-ttu-id="a7975-126">Asegúrese de que esté diseñado correctamente para satisfacer sus necesidades de disponibilidad.</span><span class="sxs-lookup"><span data-stu-id="a7975-126">Ensure it is properly designed to meet your availability requirements.</span></span> <span data-ttu-id="a7975-127">A la hora de la implementación, tenga en cuenta las funcionalidades de resistencia y tolerancia a errores.</span><span class="sxs-lookup"><span data-stu-id="a7975-127">Consider resiliency and fault tolerance capabilities when implementing.</span></span>
- <span data-ttu-id="a7975-128">El servicio de puerta de enlace puede introducir un cuello de botella.</span><span class="sxs-lookup"><span data-stu-id="a7975-128">The gateway service may introduce a bottleneck.</span></span> <span data-ttu-id="a7975-129">Asegúrese de que la puerta de enlace tenga un rendimiento adecuado para administrar la carga y que pueda escalarse fácilmente en línea con sus expectativas de crecimiento.</span><span class="sxs-lookup"><span data-stu-id="a7975-129">Ensure the gateway has adequate performance to handle load and can easily scale in line with your growth expectations.</span></span>
- <span data-ttu-id="a7975-130">Realice pruebas de carga en la puerta de enlace para asegurarse de que no introduce errores en cascada en los servicios.</span><span class="sxs-lookup"><span data-stu-id="a7975-130">Perform load testing against the gateway to ensure you don't introduce cascading failures for services.</span></span>
- <span data-ttu-id="a7975-131">El enrutamiento de puerta de enlace es de nivel 7.</span><span class="sxs-lookup"><span data-stu-id="a7975-131">Gateway routing is level 7.</span></span> <span data-ttu-id="a7975-132">Se puede basar en la dirección IP, el puerto, el encabezado o la dirección URL.</span><span class="sxs-lookup"><span data-stu-id="a7975-132">It can be based on IP, port, header, or URL.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="a7975-133">Cuándo usar este patrón</span><span class="sxs-lookup"><span data-stu-id="a7975-133">When to use this pattern</span></span>

<span data-ttu-id="a7975-134">Use este patrón en los siguientes supuestos:</span><span class="sxs-lookup"><span data-stu-id="a7975-134">Use this pattern when:</span></span>

- <span data-ttu-id="a7975-135">Un cliente deba usar varios servicios a los que se pueda acceder detrás de una puerta de enlace.</span><span class="sxs-lookup"><span data-stu-id="a7975-135">A client needs to consume multiple services that can be accessed behind a gateway.</span></span>
- <span data-ttu-id="a7975-136">Quiera simplificar las aplicaciones cliente usando un único punto de conexión.</span><span class="sxs-lookup"><span data-stu-id="a7975-136">You wish to simplify client applications by using a single endpoint.</span></span>
- <span data-ttu-id="a7975-137">Necesite enrutar las solicitudes desde puntos de conexión externamente direccionables hasta puntos de conexión virtuales internos, por ejemplo, exponer los puertos de una máquina virtual a las direcciones IP virtuales del clúster.</span><span class="sxs-lookup"><span data-stu-id="a7975-137">You need to route requests from externally addressable endpoints to internal virtual endpoints, such as exposing ports on a VM to cluster virtual IP addresses.</span></span>

<span data-ttu-id="a7975-138">Este patrón puede no ser adecuado si tiene una aplicación sencilla que solo usa uno o dos servicios.</span><span class="sxs-lookup"><span data-stu-id="a7975-138">This pattern may not be suitable when you have a simple application that uses only one or two services.</span></span>

## <a name="example"></a><span data-ttu-id="a7975-139">Ejemplo</span><span class="sxs-lookup"><span data-stu-id="a7975-139">Example</span></span>

<span data-ttu-id="a7975-140">A continuación se muestra un ejemplo sencillo de archivo de configuración de un servidor que enruta solicitudes de aplicaciones que residen en distintos directorios virtuales a diferentes máquinas en el back end mediante el enrutador Ngnix.</span><span class="sxs-lookup"><span data-stu-id="a7975-140">Using Nginx as the router, the following is a simple example configuration file for a server that routes requests for applications residing on different virtual directories to different machines at the back end.</span></span>

```
server {
    listen 80;
    server_name domain.com;

    location /app1 {
        proxy_pass http://10.0.3.10:80;
    }

    location /app2 {
        proxy_pass http://10.0.3.20:80;
    }

    location /app3 {
        proxy_pass http://10.0.3.30:80;
    }
}
```

## <a name="related-guidance"></a><span data-ttu-id="a7975-141">Instrucciones relacionadas</span><span class="sxs-lookup"><span data-stu-id="a7975-141">Related guidance</span></span>

- [<span data-ttu-id="a7975-142">Patrón Backends for Frontends</span><span class="sxs-lookup"><span data-stu-id="a7975-142">Backends for Frontends pattern</span></span>](./backends-for-frontends.md)
- [<span data-ttu-id="a7975-143">Patrón Gateway Aggregation</span><span class="sxs-lookup"><span data-stu-id="a7975-143">Gateway Aggregation pattern</span></span>](./gateway-aggregation.md)
- [<span data-ttu-id="a7975-144">Patrón Gateway Offloading</span><span class="sxs-lookup"><span data-stu-id="a7975-144">Gateway Offloading pattern</span></span>](./gateway-offloading.md)



