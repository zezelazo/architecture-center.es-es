---
title: Principios de diseño para las aplicaciones de Azure
titleSuffix: Azure Application Architecture Guide
description: Principios de diseño para las aplicaciones de Azure.
author: MikeWasson
ms.date: 08/30/2018
ms.custom: seojan19
ms.openlocfilehash: 0ea6d6dd8a030591ce00a42aad5c693ea7809f6a
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/08/2019
ms.locfileid: "54110465"
---
# <a name="ten-design-principles-for-azure-applications"></a><span data-ttu-id="056c8-103">Diez principios de diseño para las aplicaciones de Azure</span><span class="sxs-lookup"><span data-stu-id="056c8-103">Ten design principles for Azure applications</span></span>

<span data-ttu-id="056c8-104">Siga estos principios de diseño para que la aplicación sea más escalable, resistente y administrable.</span><span class="sxs-lookup"><span data-stu-id="056c8-104">Follow these design principles to make your application more scalable, resilient, and manageable.</span></span>

<span data-ttu-id="056c8-105">**[Diseñe para la recuperación automática](self-healing.md)**.</span><span class="sxs-lookup"><span data-stu-id="056c8-105">**[Design for self healing](self-healing.md)**.</span></span> <span data-ttu-id="056c8-106">En un sistema distribuido, se producen errores.</span><span class="sxs-lookup"><span data-stu-id="056c8-106">In a distributed system, failures happen.</span></span> <span data-ttu-id="056c8-107">Diseñe la aplicación para que se recupere automáticamente cuando esto suceda.</span><span class="sxs-lookup"><span data-stu-id="056c8-107">Design your application to be self healing when failures occur.</span></span>

<span data-ttu-id="056c8-108">**[Haga que todo sea redundante](redundancy.md)**.</span><span class="sxs-lookup"><span data-stu-id="056c8-108">**[Make all things redundant](redundancy.md)**.</span></span> <span data-ttu-id="056c8-109">Cree redundancia en la aplicación, para evitar tener puntos únicos de error.</span><span class="sxs-lookup"><span data-stu-id="056c8-109">Build redundancy into your application, to avoid having single points of failure.</span></span>

<span data-ttu-id="056c8-110">**[Minimice la coordinación](minimize-coordination.md)**.</span><span class="sxs-lookup"><span data-stu-id="056c8-110">**[Minimize coordination](minimize-coordination.md)**.</span></span> <span data-ttu-id="056c8-111">Minimice la coordinación entre los servicios de aplicación para lograr escalabilidad.</span><span class="sxs-lookup"><span data-stu-id="056c8-111">Minimize coordination between application services to achieve scalability.</span></span>

<span data-ttu-id="056c8-112">**[Diseñe el escalado horizontal](scale-out.md)**. Diseñe la aplicación para que pueda escalarse horizontalmente, agregando o quitando nuevas instancias a medida que se requiera.</span><span class="sxs-lookup"><span data-stu-id="056c8-112">**[Design to scale out](scale-out.md)**. Design your application so that it can scale horizontally, adding or removing new instances as demand requires.</span></span>

<span data-ttu-id="056c8-113">**[Cree particiones alrededor de límites](partition.md)**.</span><span class="sxs-lookup"><span data-stu-id="056c8-113">**[Partition around limits](partition.md)**.</span></span> <span data-ttu-id="056c8-114">Use particiones para evitar los limites en la base de datos, la red y el proceso.</span><span class="sxs-lookup"><span data-stu-id="056c8-114">Use partitioning to work around database, network, and compute limits.</span></span>

<span data-ttu-id="056c8-115">**[Diseñe para las operaciones](design-for-operations.md)**.</span><span class="sxs-lookup"><span data-stu-id="056c8-115">**[Design for operations](design-for-operations.md)**.</span></span> <span data-ttu-id="056c8-116">Diseñe la aplicación para que el equipo de operaciones tenga las herramientas que necesita.</span><span class="sxs-lookup"><span data-stu-id="056c8-116">Design your application so that the operations team has the tools they need.</span></span>

<span data-ttu-id="056c8-117">**[Use servicios administrados](managed-services.md)**.</span><span class="sxs-lookup"><span data-stu-id="056c8-117">**[Use managed services](managed-services.md)**.</span></span> <span data-ttu-id="056c8-118">Cuando sea posible, use la plataforma como servicio (PaaS) en lugar de la infraestructura como servicio (IaaS).</span><span class="sxs-lookup"><span data-stu-id="056c8-118">When possible, use platform as a service (PaaS) rather than infrastructure as a service (IaaS).</span></span>

<span data-ttu-id="056c8-119">**[Use el mejor almacén de datos para el trabajo](use-the-best-data-store.md)**.</span><span class="sxs-lookup"><span data-stu-id="056c8-119">**[Use the best data store for the job](use-the-best-data-store.md)**.</span></span> <span data-ttu-id="056c8-120">Elija la tecnología de almacenamiento que encaje mejor con sus datos y el modo en que se utilizarán.</span><span class="sxs-lookup"><span data-stu-id="056c8-120">Pick the storage technology that is the best fit for your data and how it will be used.</span></span>

<span data-ttu-id="056c8-121">**[Diseñe para evolucionar](design-for-evolution.md)**.</span><span class="sxs-lookup"><span data-stu-id="056c8-121">**[Design for evolution](design-for-evolution.md)**.</span></span> <span data-ttu-id="056c8-122">Todas las aplicaciones correctas cambian con el tiempo.</span><span class="sxs-lookup"><span data-stu-id="056c8-122">All successful applications change over time.</span></span> <span data-ttu-id="056c8-123">Un diseño evolutivo es clave para una innovación continua.</span><span class="sxs-lookup"><span data-stu-id="056c8-123">An evolutionary design is key for continuous innovation.</span></span>

<span data-ttu-id="056c8-124">**[Cree teniendo en cuenta las necesidades de la empresa](build-for-business.md)**.</span><span class="sxs-lookup"><span data-stu-id="056c8-124">**[Build for the needs of business](build-for-business.md)**.</span></span> <span data-ttu-id="056c8-125">Cada decisión de diseño debe estar justificada por un requisito empresarial.</span><span class="sxs-lookup"><span data-stu-id="056c8-125">Every design decision must be justified by a business requirement.</span></span>
