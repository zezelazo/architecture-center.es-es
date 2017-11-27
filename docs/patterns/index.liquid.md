---
title: "Patrones de diseño en la nube"
description: "Patrones de diseño en la nube para Microsoft Azure"
keywords: Azure
ms.openlocfilehash: 264b8296a428f9c1b87314b782efcabc89cf010f
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2017
---
# <a name="cloud-design-patterns"></a><span data-ttu-id="6f317-104">Patrones de diseño en la nube</span><span class="sxs-lookup"><span data-stu-id="6f317-104">Cloud Design Patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="6f317-105">Estos patrones de diseño son útiles para crear aplicaciones confiables, escalables y seguras en la nube.</span><span class="sxs-lookup"><span data-stu-id="6f317-105">These design patterns are useful for building reliable, scalable, secure applications in the cloud.</span></span>

<span data-ttu-id="6f317-106">Cada patrón describe el problema al que hace frente, las consideraciones sobre su aplicación y un ejemplo basado en Microsoft Azure.</span><span class="sxs-lookup"><span data-stu-id="6f317-106">Each pattern describes the problem that the pattern addresses, considerations for applying the pattern, and an example based on Microsoft Azure.</span></span> <span data-ttu-id="6f317-107">La mayoría incluye ejemplos de código o fragmentos de código que muestran cómo implementar el patrón en Azure.</span><span class="sxs-lookup"><span data-stu-id="6f317-107">Most of the patterns include code samples or snippets that show how to implement the pattern on Azure.</span></span> <span data-ttu-id="6f317-108">Sin embargo, la mayoría de los patrones es importante para todos los sistemas distribuidos, tanto si están hospedados en Azure como en otras plataformas en la nube.</span><span class="sxs-lookup"><span data-stu-id="6f317-108">However, most of the patterns are relevant to any distributed system, whether hosted on Azure or on other cloud platforms.</span></span>

## <a name="problem-areas-in-the-cloud"></a><span data-ttu-id="6f317-109">Áreas con problemas en la nube</span><span class="sxs-lookup"><span data-stu-id="6f317-109">Problem areas in the cloud</span></span>

<ul id="categories" class="panel">
<span data-ttu-id="6f317-110">{%- para las categorías de categorías %}</span><span class="sxs-lookup"><span data-stu-id="6f317-110">{%- for category in categories %}</span></span>
    <li>
    <span data-ttu-id="6f317-111">{% incluye 'patrón-categoría´-tarjeta' %}</span><span class="sxs-lookup"><span data-stu-id="6f317-111">{% include 'pattern-category-card' %}</span></span>
    </li>
<span data-ttu-id="6f317-112">{%- endfor %}</span><span class="sxs-lookup"><span data-stu-id="6f317-112">{%- endfor %}</span></span>
</ul>

## <a name="catalog-of-patterns"></a><span data-ttu-id="6f317-113">Catálogo de patrones</span><span class="sxs-lookup"><span data-stu-id="6f317-113">Catalog of patterns</span></span>

| <span data-ttu-id="6f317-114">Patrón</span><span class="sxs-lookup"><span data-stu-id="6f317-114">Pattern</span></span> | <span data-ttu-id="6f317-115">Resumen</span><span class="sxs-lookup"><span data-stu-id="6f317-115">Summary</span></span> |
| ------- | ------- |
<span data-ttu-id="6f317-116">{%- para patrón en patrones %} | [{{patrón.título}}](./{{ pattern.file }}) | {{patrón.descripción}} | {endfor %}</span><span class="sxs-lookup"><span data-stu-id="6f317-116">{%- for pattern in patterns %} | [{{ pattern.title }}](./{{ pattern.file }}) | {{ pattern.description }} | {%- endfor %}</span></span>