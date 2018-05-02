---
title: Árbol de decisión para los servicios de proceso de Azure
description: Un diagrama de flujo para la selección de un servicio de proceso
author: MikeWasson
ms.date: 04/21/2018
ms.openlocfilehash: f5703f4906ca2ea6f825b383710eb4bd335f5043
ms.sourcegitcommit: d702b4d27e96e7a5a248dc4f2f0e25cf6e82c134
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/23/2018
---
# <a name="decision-tree-for-azure-compute-services"></a><span data-ttu-id="365a4-103">Árbol de decisión para los servicios de proceso de Azure</span><span class="sxs-lookup"><span data-stu-id="365a4-103">Decision tree for Azure compute services</span></span>

<span data-ttu-id="365a4-104">Azure ofrece una serie de formas de hospedar el código de aplicación.</span><span class="sxs-lookup"><span data-stu-id="365a4-104">Azure offers a number of ways to host your application code.</span></span> <span data-ttu-id="365a4-105">El término *proceso* hace referencia al modelo de hospedaje para los recursos informáticos donde se ejecutan las aplicaciones.</span><span class="sxs-lookup"><span data-stu-id="365a4-105">The term *compute* refers to the hosting model for the computing resources that your application runs on.</span></span> <span data-ttu-id="365a4-106">El diagrama de flujo siguiente le ayudará a elegir un servicio de proceso para la aplicación.</span><span class="sxs-lookup"><span data-stu-id="365a4-106">The following flowchart will help you to choose a compute service for your application.</span></span>
 
![](../images/compute-decision-tree.svg)

<span data-ttu-id="365a4-107">El diagrama de flujo le guía a través de un conjunto de criterios de decisión clave para alcanzar una recomendación.</span><span class="sxs-lookup"><span data-stu-id="365a4-107">The flowchart guides you through a set of key decision criteria to reach a recommendation.</span></span> <span data-ttu-id="365a4-108">Cada aplicación tiene requisitos únicos, por lo que debe tratar la recomendación como un punto inicial.</span><span class="sxs-lookup"><span data-stu-id="365a4-108">Every application has unique requirements, so you should treat the recommendation as a starting point.</span></span> <span data-ttu-id="365a4-109">A partir de ahí realice un análisis más detallado, examinando aspectos como:</span><span class="sxs-lookup"><span data-stu-id="365a4-109">Then perform a more detailed analysis, looking at aspects such as:</span></span>
 
- <span data-ttu-id="365a4-110">Conjunto de características</span><span class="sxs-lookup"><span data-stu-id="365a4-110">Feature set</span></span>
- [<span data-ttu-id="365a4-111">Límites de servicio</span><span class="sxs-lookup"><span data-stu-id="365a4-111">Service limits</span></span>](/azure/azure-subscription-service-limits)
- [<span data-ttu-id="365a4-112">Costee</span><span class="sxs-lookup"><span data-stu-id="365a4-112">Cost</span></span>](https://azure.microsoft.com/pricing/)
- [<span data-ttu-id="365a4-113">Acuerdo de Nivel de Servicio</span><span class="sxs-lookup"><span data-stu-id="365a4-113">SLA</span></span>](https://azure.microsoft.com/support/legal/sla/)
- [<span data-ttu-id="365a4-114">Disponibilidad regional</span><span class="sxs-lookup"><span data-stu-id="365a4-114">Regional availability</span></span>](https://azure.microsoft.com/global-infrastructure/services/)
- <span data-ttu-id="365a4-115">Ecosistema del desarrollador y habilidades del equipo</span><span class="sxs-lookup"><span data-stu-id="365a4-115">Developer ecosystem and team skills</span></span>
- [<span data-ttu-id="365a4-116">Tablas de comparación de procesos</span><span class="sxs-lookup"><span data-stu-id="365a4-116">Compute comparison tables</span></span>](./compute-comparison.md)

<span data-ttu-id="365a4-117">Si la aplicación consta de varias cargas de trabajo, evalúe cada carga de trabajo por separado.</span><span class="sxs-lookup"><span data-stu-id="365a4-117">If your application consists of multiple workloads, evaluate each workload separately.</span></span> <span data-ttu-id="365a4-118">Una solución completa puede incluir dos o más servicios de proceso.</span><span class="sxs-lookup"><span data-stu-id="365a4-118">A complete solution may incorporate two or more compute services.</span></span>

