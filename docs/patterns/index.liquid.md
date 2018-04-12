---
title: Patrones de diseño en la nube
description: Patrones de diseño en la nube para Microsoft Azure
keywords: Azure
ms.openlocfilehash: 4747c896fc6fc5866be782d76c5290d6b49ad451
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2018
---
# <a name="cloud-design-patterns"></a>Patrones de diseño en la nube

[!INCLUDE [header](../../_includes/header.md)]

Estos patrones de diseño son útiles para crear aplicaciones confiables, escalables y seguras en la nube.

Cada patrón describe el problema al que hace frente, las consideraciones sobre su aplicación y un ejemplo basado en Microsoft Azure. La mayoría incluye ejemplos de código o fragmentos de código que muestran cómo implementar el patrón en Azure. Sin embargo, la mayoría de los patrones es importante para todos los sistemas distribuidos, tanto si están hospedados en Azure como en otras plataformas en la nube.

## <a name="problem-areas-in-the-cloud"></a>Áreas con problemas en la nube

<ul id="categories" class="panel">
{%- para las categorías de categorías %}
    <li>
    {% incluye 'patrón-categoría´-tarjeta' %}
    </li>
{%- endfor %}
</ul>

## <a name="catalog-of-patterns"></a>Catálogo de patrones

| Patrón | Resumen |
|---------|---------|
|         |         |

{%- para patrón en patrones %} | [{{patrón.título}}](./{{ pattern.file }}) | {{patrón.descripción}} | {endfor %}
