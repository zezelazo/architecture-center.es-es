---
title: Árbol de decisión para los servicios de proceso de Azure
description: Un diagrama de flujo para la selección de un servicio de proceso
author: MikeWasson
ms.date: 06/13/2018
ms.openlocfilehash: 689ec3f265e563273a75ad98268d03624a7b4536
ms.sourcegitcommit: ce2fa8ac2d310f7078317cade12f1b89db1ffe06
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/23/2018
ms.locfileid: "36338190"
---
# <a name="decision-tree-for-azure-compute-services"></a><span data-ttu-id="1e0d7-103">Árbol de decisión para los servicios de proceso de Azure</span><span class="sxs-lookup"><span data-stu-id="1e0d7-103">Decision tree for Azure compute services</span></span>

<span data-ttu-id="1e0d7-104">Azure ofrece una serie de formas de hospedar el código de aplicación.</span><span class="sxs-lookup"><span data-stu-id="1e0d7-104">Azure offers a number of ways to host your application code.</span></span> <span data-ttu-id="1e0d7-105">El término *proceso* hace referencia al modelo de hospedaje para los recursos informáticos donde se ejecutan las aplicaciones.</span><span class="sxs-lookup"><span data-stu-id="1e0d7-105">The term *compute* refers to the hosting model for the computing resources that your application runs on.</span></span> <span data-ttu-id="1e0d7-106">El diagrama de flujo siguiente le ayudará a elegir un servicio de proceso para la aplicación.</span><span class="sxs-lookup"><span data-stu-id="1e0d7-106">The following flowchart will help you to choose a compute service for your application.</span></span> <span data-ttu-id="1e0d7-107">El diagrama de flujo le guía a través de un conjunto de criterios de decisión clave para alcanzar una recomendación.</span><span class="sxs-lookup"><span data-stu-id="1e0d7-107">The flowchart guides you through a set of key decision criteria to reach a recommendation.</span></span> 

<span data-ttu-id="1e0d7-108">**Tratar este diagrama de flujo como un punto de partida.**</span><span class="sxs-lookup"><span data-stu-id="1e0d7-108">**Treat this flowchart as a starting point.**</span></span> <span data-ttu-id="1e0d7-109">Cada aplicación tiene requisitos únicos, por lo que debe usar la recomendación como un punto de partida.</span><span class="sxs-lookup"><span data-stu-id="1e0d7-109">Every application has unique requirements, so use the recommendation as a starting point.</span></span> <span data-ttu-id="1e0d7-110">A partir de ahí, realice una evaluación más detallada y examine aspectos como:</span><span class="sxs-lookup"><span data-stu-id="1e0d7-110">Then perform a more detailed evaluation, looking at aspects such as:</span></span>
 
- <span data-ttu-id="1e0d7-111">Conjunto de características</span><span class="sxs-lookup"><span data-stu-id="1e0d7-111">Feature set</span></span>
- [<span data-ttu-id="1e0d7-112">Límites de servicio</span><span class="sxs-lookup"><span data-stu-id="1e0d7-112">Service limits</span></span>](/azure/azure-subscription-service-limits)
- [<span data-ttu-id="1e0d7-113">Costee</span><span class="sxs-lookup"><span data-stu-id="1e0d7-113">Cost</span></span>](https://azure.microsoft.com/pricing/)
- [<span data-ttu-id="1e0d7-114">Acuerdo de Nivel de Servicio</span><span class="sxs-lookup"><span data-stu-id="1e0d7-114">SLA</span></span>](https://azure.microsoft.com/support/legal/sla/)
- [<span data-ttu-id="1e0d7-115">Disponibilidad regional</span><span class="sxs-lookup"><span data-stu-id="1e0d7-115">Regional availability</span></span>](https://azure.microsoft.com/global-infrastructure/services/)
- <span data-ttu-id="1e0d7-116">Ecosistema del desarrollador y habilidades del equipo</span><span class="sxs-lookup"><span data-stu-id="1e0d7-116">Developer ecosystem and team skills</span></span>
- [<span data-ttu-id="1e0d7-117">Tablas de comparación de procesos</span><span class="sxs-lookup"><span data-stu-id="1e0d7-117">Compute comparison tables</span></span>](./compute-comparison.md)

<span data-ttu-id="1e0d7-118">Si la aplicación consta de varias cargas de trabajo, evalúe cada carga de trabajo por separado.</span><span class="sxs-lookup"><span data-stu-id="1e0d7-118">If your application consists of multiple workloads, evaluate each workload separately.</span></span> <span data-ttu-id="1e0d7-119">Una solución completa puede incluir dos o más servicios de proceso.</span><span class="sxs-lookup"><span data-stu-id="1e0d7-119">A complete solution may incorporate two or more compute services.</span></span>

<span data-ttu-id="1e0d7-120">Para más información acerca de las opciones de hospedaje de contenedores en Azure, consulte https://azure.microsoft.com/overview/containers/.</span><span class="sxs-lookup"><span data-stu-id="1e0d7-120">For more information about your options for hosting containers in Azure, see https://azure.microsoft.com/overview/containers/.</span></span>

## <a name="flowchart"></a><span data-ttu-id="1e0d7-121">Diagrama de flujo</span><span class="sxs-lookup"><span data-stu-id="1e0d7-121">Flowchart</span></span>

![](../images/compute-decision-tree.svg)

## <a name="definitions"></a><span data-ttu-id="1e0d7-122">Definiciones</span><span class="sxs-lookup"><span data-stu-id="1e0d7-122">Definitions</span></span>

- <span data-ttu-id="1e0d7-123">**Greenfield** describe un proyecto de software que es completamente nuevo y se ha creado desde cero.</span><span class="sxs-lookup"><span data-stu-id="1e0d7-123">**Greenfield** describes a software project that is completely new and built from scratch.</span></span> <span data-ttu-id="1e0d7-124">No incluye código heredado.</span><span class="sxs-lookup"><span data-stu-id="1e0d7-124">It does not include legacy code.</span></span> 

- <span data-ttu-id="1e0d7-125">**Brownfield** describe un proyecto de software que se basa en una aplicación existente.</span><span class="sxs-lookup"><span data-stu-id="1e0d7-125">**Brownfield** describes a software project that builds on an existing application.</span></span> <span data-ttu-id="1e0d7-126">Puede heredar código o marcos de trabajo.</span><span class="sxs-lookup"><span data-stu-id="1e0d7-126">It may inherit legacy code or frameworks.</span></span>

- <span data-ttu-id="1e0d7-127">**Lift-and-shift** es una estrategia de migración de una carga de trabajo a la nube sin volver a diseñar la aplicación ni realizar cambios en el código.</span><span class="sxs-lookup"><span data-stu-id="1e0d7-127">**Lift and shift** is a strategy for migrating a workload to the cloud without redesigning the application or making code changes.</span></span> <span data-ttu-id="1e0d7-128">También se denomina *rehospedaje*.</span><span class="sxs-lookup"><span data-stu-id="1e0d7-128">Also called *rehosting*.</span></span> <span data-ttu-id="1e0d7-129">Para más información, consulte [Azure Migration Center](https://azure.microsoft.com/migration/).</span><span class="sxs-lookup"><span data-stu-id="1e0d7-129">For more information, see [Azure migration center](https://azure.microsoft.com/migration/).</span></span>

- <span data-ttu-id="1e0d7-130">**Optimizado para la nube** es una estrategia de migración a la nube mediante la refactorización de una aplicación para aprovechar las funcionalidades y características nativas de la nube.</span><span class="sxs-lookup"><span data-stu-id="1e0d7-130">**Cloud optimized** is a strategy for migrating to the cloud by refactoring an application to take advantage of cloud-native features and capabilities.</span></span>

## <a name="next-steps"></a><span data-ttu-id="1e0d7-131">Pasos siguientes</span><span class="sxs-lookup"><span data-stu-id="1e0d7-131">Next steps</span></span>

<span data-ttu-id="1e0d7-132">Para que ver más criterios hay que tener en cuenta, consulte [Criterios para elegir un servicio de proceso de Azure](./compute-comparison.md).</span><span class="sxs-lookup"><span data-stu-id="1e0d7-132">For additional criteria to consider, see [Criteria for choosing an Azure compute service](./compute-comparison.md).</span></span>
