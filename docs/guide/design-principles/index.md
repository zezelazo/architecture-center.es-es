---
title: Principios de diseño para las aplicaciones de Azure
description: Principios de diseño para las aplicaciones de Azure
author: MikeWasson
ms.openlocfilehash: 462896098c668c0775464ca498925266cd73c6e1
ms.sourcegitcommit: 26b04f138a860979aea5d253ba7fecffc654841e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/19/2018
ms.locfileid: "36206811"
---
# <a name="design-principles-for-azure-applications"></a><span data-ttu-id="73ed7-103">Principios de diseño para las aplicaciones de Azure</span><span class="sxs-lookup"><span data-stu-id="73ed7-103">Design principles for Azure applications</span></span>

<span data-ttu-id="73ed7-104">Siga estos principios de diseño para que la aplicación sea más escalable, resistente y administrable.</span><span class="sxs-lookup"><span data-stu-id="73ed7-104">Follow these design principles to make your application more scalable, resilient, and manageable.</span></span> 

<span data-ttu-id="73ed7-105">**[Diseñe para la recuperación automática](self-healing.md)**.</span><span class="sxs-lookup"><span data-stu-id="73ed7-105">**[Design for self healing](self-healing.md)**.</span></span> <span data-ttu-id="73ed7-106">En un sistema distribuido, se producen errores.</span><span class="sxs-lookup"><span data-stu-id="73ed7-106">In a distributed system, failures happen.</span></span> <span data-ttu-id="73ed7-107">Diseñe la aplicación para que se recupere automáticamente cuando esto suceda.</span><span class="sxs-lookup"><span data-stu-id="73ed7-107">Design your application to be self healing when failures occur.</span></span>

<span data-ttu-id="73ed7-108">**[Haga que todo sea redundante](redundancy.md)**.</span><span class="sxs-lookup"><span data-stu-id="73ed7-108">**[Make all things redundant](redundancy.md)**.</span></span> <span data-ttu-id="73ed7-109">Cree redundancia en la aplicación, para evitar tener puntos únicos de error.</span><span class="sxs-lookup"><span data-stu-id="73ed7-109">Build redundancy into your application, to avoid having single points of failure.</span></span>
 
<span data-ttu-id="73ed7-110">**[Minimice la coordinación](minimize-coordination.md)**.</span><span class="sxs-lookup"><span data-stu-id="73ed7-110">**[Minimize coordination](minimize-coordination.md)**.</span></span> <span data-ttu-id="73ed7-111">Minimice la coordinación entre los servicios de aplicación para lograr escalabilidad.</span><span class="sxs-lookup"><span data-stu-id="73ed7-111">Minimize coordination between application services to achieve scalability.</span></span>
 
<span data-ttu-id="73ed7-112">**[Diseñe el escalado horizontal](scale-out.md)**. Diseñe la aplicación para que pueda escalarse horizontalmente, agregando o quitando nuevas instancias a medida que se requiera.</span><span class="sxs-lookup"><span data-stu-id="73ed7-112">**[Design to scale out](scale-out.md)**. Design your application so that it can scale horizontally, adding or removing new instances as demand requires.</span></span>

<span data-ttu-id="73ed7-113">**[Cree particiones alrededor de límites](partition.md)**.</span><span class="sxs-lookup"><span data-stu-id="73ed7-113">**[Partition around limits](partition.md)**.</span></span> <span data-ttu-id="73ed7-114">Use particiones para evitar los limites en la base de datos, la red y el proceso.</span><span class="sxs-lookup"><span data-stu-id="73ed7-114">Use partitioning to work around database, network, and compute limits.</span></span>

<span data-ttu-id="73ed7-115">**[Diseñe para las operaciones](design-for-operations.md)**.</span><span class="sxs-lookup"><span data-stu-id="73ed7-115">**[Design for operations](design-for-operations.md)**.</span></span> <span data-ttu-id="73ed7-116">Diseñe la aplicación para que el equipo de operaciones tenga las herramientas que necesita.</span><span class="sxs-lookup"><span data-stu-id="73ed7-116">Design your application so that the operations team has the tools they need.</span></span>

<span data-ttu-id="73ed7-117">**[Use servicios administrados](managed-services.md)**.</span><span class="sxs-lookup"><span data-stu-id="73ed7-117">**[Use managed services](managed-services.md)**.</span></span> <span data-ttu-id="73ed7-118">Cuando sea posible, use la plataforma como servicio (PaaS) en lugar de la infraestructura como servicio (IaaS).</span><span class="sxs-lookup"><span data-stu-id="73ed7-118">When possible, use platform as a service (PaaS) rather than infrastructure as a service (IaaS).</span></span>

<span data-ttu-id="73ed7-119">**[Use el mejor almacén de datos para el trabajo](use-the-best-data-store.md)**.</span><span class="sxs-lookup"><span data-stu-id="73ed7-119">**[Use the best data store for the job](use-the-best-data-store.md)**.</span></span> <span data-ttu-id="73ed7-120">Elija la tecnología de almacenamiento que encaje mejor con sus datos y el modo en que se utilizarán.</span><span class="sxs-lookup"><span data-stu-id="73ed7-120">Pick the storage technology that is the best fit for your data and how it will be used.</span></span> 
 
<span data-ttu-id="73ed7-121">**[Diseñe para evolucionar](design-for-evolution.md)**.</span><span class="sxs-lookup"><span data-stu-id="73ed7-121">**[Design for evolution](design-for-evolution.md)**.</span></span> <span data-ttu-id="73ed7-122">Todas las aplicaciones correctas cambian con el tiempo.</span><span class="sxs-lookup"><span data-stu-id="73ed7-122">All successful applications change over time.</span></span> <span data-ttu-id="73ed7-123">Un diseño evolutivo es clave para una innovación continua.</span><span class="sxs-lookup"><span data-stu-id="73ed7-123">An evolutionary design is key for continuous innovation.</span></span>

<span data-ttu-id="73ed7-124">**[Cree teniendo en cuenta las necesidades de la empresa](build-for-business.md)**.</span><span class="sxs-lookup"><span data-stu-id="73ed7-124">**[Build for the needs of business](build-for-business.md)**.</span></span> <span data-ttu-id="73ed7-125">Cada decisión de diseño debe estar justificada por un requisito empresarial.</span><span class="sxs-lookup"><span data-stu-id="73ed7-125">Every design decision must be justified by a business requirement.</span></span>

