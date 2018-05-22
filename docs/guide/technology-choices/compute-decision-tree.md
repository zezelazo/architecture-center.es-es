---
title: Árbol de decisión para los servicios de proceso de Azure
description: Un diagrama de flujo para la selección de un servicio de proceso
author: MikeWasson
ms.date: 04/21/2018
ms.openlocfilehash: 3dcfbd156d4fced863a56bcc8bb74483aa665f9f
ms.sourcegitcommit: 7ced70ebc11aa0df0dc0104092d3cc6ad5c28bd6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2018
---
# <a name="decision-tree-for-azure-compute-services"></a><span data-ttu-id="5d167-103">Árbol de decisión para los servicios de proceso de Azure</span><span class="sxs-lookup"><span data-stu-id="5d167-103">Decision tree for Azure compute services</span></span>

<span data-ttu-id="5d167-104">Azure ofrece una serie de formas de hospedar el código de aplicación.</span><span class="sxs-lookup"><span data-stu-id="5d167-104">Azure offers a number of ways to host your application code.</span></span> <span data-ttu-id="5d167-105">El término *proceso* hace referencia al modelo de hospedaje para los recursos informáticos donde se ejecutan las aplicaciones.</span><span class="sxs-lookup"><span data-stu-id="5d167-105">The term *compute* refers to the hosting model for the computing resources that your application runs on.</span></span> <span data-ttu-id="5d167-106">El diagrama de flujo siguiente le ayudará a elegir un servicio de proceso para la aplicación.</span><span class="sxs-lookup"><span data-stu-id="5d167-106">The following flowchart will help you to choose a compute service for your application.</span></span> <span data-ttu-id="5d167-107">El diagrama de flujo le guía a través de un conjunto de criterios de decisión clave para alcanzar una recomendación.</span><span class="sxs-lookup"><span data-stu-id="5d167-107">The flowchart guides you through a set of key decision criteria to reach a recommendation.</span></span> 

<span data-ttu-id="5d167-108">**Tratar este diagrama de flujo como un punto de partida.**</span><span class="sxs-lookup"><span data-stu-id="5d167-108">**Treat this flowchart as a starting point.**</span></span> <span data-ttu-id="5d167-109">Cada aplicación tiene requisitos únicos, por lo que debe usar la recomendación como un punto de partida.</span><span class="sxs-lookup"><span data-stu-id="5d167-109">Every application has unique requirements, so use the recommendation as a starting point.</span></span> <span data-ttu-id="5d167-110">A partir de ahí, realice una evaluación más detallada y examine aspectos como:</span><span class="sxs-lookup"><span data-stu-id="5d167-110">Then perform a more detailed evaluation, looking at aspects such as:</span></span>
 
- <span data-ttu-id="5d167-111">Conjunto de características</span><span class="sxs-lookup"><span data-stu-id="5d167-111">Feature set</span></span>
- [<span data-ttu-id="5d167-112">Límites de servicio</span><span class="sxs-lookup"><span data-stu-id="5d167-112">Service limits</span></span>](/azure/azure-subscription-service-limits)
- [<span data-ttu-id="5d167-113">Costee</span><span class="sxs-lookup"><span data-stu-id="5d167-113">Cost</span></span>](https://azure.microsoft.com/pricing/)
- [<span data-ttu-id="5d167-114">Acuerdo de Nivel de Servicio</span><span class="sxs-lookup"><span data-stu-id="5d167-114">SLA</span></span>](https://azure.microsoft.com/support/legal/sla/)
- [<span data-ttu-id="5d167-115">Disponibilidad regional</span><span class="sxs-lookup"><span data-stu-id="5d167-115">Regional availability</span></span>](https://azure.microsoft.com/global-infrastructure/services/)
- <span data-ttu-id="5d167-116">Ecosistema del desarrollador y habilidades del equipo</span><span class="sxs-lookup"><span data-stu-id="5d167-116">Developer ecosystem and team skills</span></span>
- [<span data-ttu-id="5d167-117">Tablas de comparación de procesos</span><span class="sxs-lookup"><span data-stu-id="5d167-117">Compute comparison tables</span></span>](./compute-comparison.md)

<span data-ttu-id="5d167-118">Si la aplicación consta de varias cargas de trabajo, evalúe cada carga de trabajo por separado.</span><span class="sxs-lookup"><span data-stu-id="5d167-118">If your application consists of multiple workloads, evaluate each workload separately.</span></span> <span data-ttu-id="5d167-119">Una solución completa puede incluir dos o más servicios de proceso.</span><span class="sxs-lookup"><span data-stu-id="5d167-119">A complete solution may incorporate two or more compute services.</span></span>

![](../images/compute-decision-tree.svg)

