---
title: Uso de servicios administrados
description: Cuando sea posible, use la plataforma como servicio (PaaS) en lugar de la infraestructura como servicio (IaaS).
author: MikeWasson
ms.date: 08/30/2018
ms.openlocfilehash: f6777a19e126a8a7f64be05dfad9bc503d27b1c3
ms.sourcegitcommit: ae8a1de6f4af7a89a66a8339879843d945201f85
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/31/2018
ms.locfileid: "43325781"
---
# <a name="use-managed-services"></a><span data-ttu-id="6b832-103">Uso de servicios administrados</span><span class="sxs-lookup"><span data-stu-id="6b832-103">Use managed services</span></span>

## <a name="when-possible-use-platform-as-a-service-paas-rather-than-infrastructure-as-a-service-iaas"></a><span data-ttu-id="6b832-104">Cuando sea posible, use la plataforma como servicio (PaaS) en lugar de la infraestructura como servicio (IaaS).</span><span class="sxs-lookup"><span data-stu-id="6b832-104">When possible, use platform as a service (PaaS) rather than infrastructure as a service (IaaS)</span></span>

<span data-ttu-id="6b832-105">IaaS es como tener una caja de piezas.</span><span class="sxs-lookup"><span data-stu-id="6b832-105">IaaS is like having a box of parts.</span></span> <span data-ttu-id="6b832-106">Puede crear cualquier cosa pero lo debe ensamblar usted mismo.</span><span class="sxs-lookup"><span data-stu-id="6b832-106">You can build anything, but you have to assemble it yourself.</span></span> <span data-ttu-id="6b832-107">Los servicios administrados son más fáciles de configurar y administrar.</span><span class="sxs-lookup"><span data-stu-id="6b832-107">Managed services are easier to configure and administer.</span></span> <span data-ttu-id="6b832-108">No es necesario aprovisionar máquinas virtuales, configurar redes virtuales, administrar revisiones y actualizaciones, y toda la sobrecarga asociada a la ejecución de software en una máquina virtual.</span><span class="sxs-lookup"><span data-stu-id="6b832-108">You don't need to provision VMs, set up VNets, manage patches and updates, and all of the other overhead associated with running software on a VM.</span></span>

<span data-ttu-id="6b832-109">Por ejemplo, suponga que la aplicación necesita una cola de mensajes.</span><span class="sxs-lookup"><span data-stu-id="6b832-109">For example, suppose your application needs a message queue.</span></span> <span data-ttu-id="6b832-110">Puede configurar su propio servicio de mensajería en una máquina virtual, con algo parecido a RabbitMQ.</span><span class="sxs-lookup"><span data-stu-id="6b832-110">You could set up your own messaging service on a VM, using something like RabbitMQ.</span></span> <span data-ttu-id="6b832-111">Pero Azure Service Bus ya proporciona una mensajería de confianza como servicio y es más fácil de configurar.</span><span class="sxs-lookup"><span data-stu-id="6b832-111">But Azure Service Bus already provides reliable messaging as service, and it's simpler to set up.</span></span> <span data-ttu-id="6b832-112">Basta con crear un espacio de nombres de Service Bus (que puede realizarse como parte de un script de implementación) y, después, llamar a Service Bus con el SDK del cliente.</span><span class="sxs-lookup"><span data-stu-id="6b832-112">Just create a Service Bus namespace (which can be done as part of a deployment script) and then call Service Bus using the client SDK.</span></span> 

<span data-ttu-id="6b832-113">Por supuesto, la aplicación puede tener requisitos específicos que hacen que un enfoque de IaaS sea más adecuado.</span><span class="sxs-lookup"><span data-stu-id="6b832-113">Of course, your application may have specific requirements that make an IaaS approach more suitable.</span></span> <span data-ttu-id="6b832-114">Sin embargo, incluso si la aplicación se basa en IaaS, busque lugares en los que sea natural incorporar servicios administrados.</span><span class="sxs-lookup"><span data-stu-id="6b832-114">However, even if your application is based on IaaS, look for places where it may be natural to incorporate managed services.</span></span> <span data-ttu-id="6b832-115">Esto incluye cachés, colas y almacenamiento de datos.</span><span class="sxs-lookup"><span data-stu-id="6b832-115">These include cache, queues, and data storage.</span></span>

| <span data-ttu-id="6b832-116">En lugar de ejecutar...</span><span class="sxs-lookup"><span data-stu-id="6b832-116">Instead of running...</span></span> | <span data-ttu-id="6b832-117">Considere la posibilidad de utilizar...</span><span class="sxs-lookup"><span data-stu-id="6b832-117">Consider using...</span></span> |
|-----------------------|-------------|
| <span data-ttu-id="6b832-118">Active Directory</span><span class="sxs-lookup"><span data-stu-id="6b832-118">Active Directory</span></span> | <span data-ttu-id="6b832-119">Azure Active Directory Domain Services</span><span class="sxs-lookup"><span data-stu-id="6b832-119">Azure Active Directory Domain Services</span></span> |
| <span data-ttu-id="6b832-120">Elasticsearch</span><span class="sxs-lookup"><span data-stu-id="6b832-120">Elasticsearch</span></span> | <span data-ttu-id="6b832-121">Azure Search</span><span class="sxs-lookup"><span data-stu-id="6b832-121">Azure Search</span></span> |
| <span data-ttu-id="6b832-122">Hadoop</span><span class="sxs-lookup"><span data-stu-id="6b832-122">Hadoop</span></span> | <span data-ttu-id="6b832-123">HDInsight</span><span class="sxs-lookup"><span data-stu-id="6b832-123">HDInsight</span></span> |
| <span data-ttu-id="6b832-124">IIS</span><span class="sxs-lookup"><span data-stu-id="6b832-124">IIS</span></span> | <span data-ttu-id="6b832-125">App Service</span><span class="sxs-lookup"><span data-stu-id="6b832-125">App Service</span></span> |
| <span data-ttu-id="6b832-126">MongoDB</span><span class="sxs-lookup"><span data-stu-id="6b832-126">MongoDB</span></span> | <span data-ttu-id="6b832-127">Cosmos DB</span><span class="sxs-lookup"><span data-stu-id="6b832-127">Cosmos DB</span></span> |
| <span data-ttu-id="6b832-128">Redis</span><span class="sxs-lookup"><span data-stu-id="6b832-128">Redis</span></span> | <span data-ttu-id="6b832-129">Azure Redis Cache</span><span class="sxs-lookup"><span data-stu-id="6b832-129">Azure Redis Cache</span></span> |
| <span data-ttu-id="6b832-130">SQL Server</span><span class="sxs-lookup"><span data-stu-id="6b832-130">SQL Server</span></span> | <span data-ttu-id="6b832-131">Azure SQL Database</span><span class="sxs-lookup"><span data-stu-id="6b832-131">Azure SQL Database</span></span> |


