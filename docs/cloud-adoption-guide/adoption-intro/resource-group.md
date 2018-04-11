---
title: 'Guía: Diseño de grupos de recursos de Azure'
description: Guía para el diseño de grupos de recursos de Azure como parte de una estrategia de adopción básica en la nube
author: petertay
ms.openlocfilehash: ac6cbb03be8cdba020641d3b9034ad9d20101acf
ms.sourcegitcommit: 2e8b06e9c07875d65b91d5431bfd4bc465a7a242
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/09/2018
---
# <a name="guidance-azure-resource-group-design"></a>Guía: Diseño de grupos de recursos de Azure

En Azure, un [grupo de recursos](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview#resource-groups) es un contenedor lógico en el que se agrupan los recursos de Azure. Cada recurso implementado en Azure debe implementarse en un único grupo de recursos.

## <a name="design-considerations"></a>Consideraciones de diseño

- Todos los recursos de un grupo de recursos deben compartir el mismo ciclo de vida. Es decir, la práctica debería ser implementar, actualizar y eliminar los recursos con el mismo ciclo de vida como un grupo. Por ejemplo, los recursos de proceso de una aplicación web única se implementan normalmente como una sola unidad. Sin embargo, una base de datos compartida con otras aplicaciones web se administraría muy probablemente en un ciclo de vida diferente y debería estar en su propio grupo de recursos.
- Un grupo de recursos puede contener recursos que residen en diferentes regiones.
- Todos los recursos en un único grupo de recursos deben asociarse a una sola suscripción. 
- Un recurso se puede mover entre grupos de recursos pero no a un grupo de recursos que incluya un recurso implementado en una suscripción diferente.
- La asignación de un grupo de recursos no afecta a la conectividad o la interacción con los recursos de otros grupos de recursos. Por ejemplo, una máquina virtual asignada a un grupo de recursos puede conectarse a una base de datos asignada a otro grupo de recursos si no hay conectividad de red entre ellas.
- Un grupo de recursos puede utilizarse para definir el ámbito de control de acceso para las acciones administrativas. Puede aplicar permisos de control de acceso basado en rol (RBAC) en el nivel de suscripción o en el nivel de grupo de recursos. Los permisos asignados en el nivel de suscripción se heredan en el nivel de grupo de recursos. Aprenderá más acerca de los permisos de recursos y RBAC en las fases de adopción intermedia y avanzada.

## <a name="proven-practices"></a>Prácticas demostradas

- En la fase básica, es probable que administre solo unos pocos proyectos de prueba de concepto (POC), cada uno con un pequeño número de recursos. Dado que los recursos de prueba de concepto normalmente comparten el mismo ciclo de vida, puede crear un grupo de recursos único para cada uno de estos proyectos.
- Durante la fase de adopción intermedia, va a administrar varios proyectos. Diferentes tipos de proyectos pueden beneficiarse de los diseños de otro grupo de recursos. Si quiere promover cualquiera de los proyectos de prueba de concepto iniciales a producción, puede mover recursos a otro grupo de recursos si pertenece a la misma suscripción. Por lo tanto, en esta fase debe implementar estos recursos en la misma suscripción, para que pueda reorganizar los recursos en el futuro.

## <a name="next-steps"></a>pasos siguientes

* Ahora que ha aprendido las prácticas demostradas para la fase de adopción básica, es posible crear grupos de recursos y agregar recursos a los mismos. Mientras está administrando un pequeño número de recursos en esta fase, su administración se vuelve más compleja al ir agregando más recursos. Obtenga información acerca de las [convenciones de nomenclatura y etiquetado de Azure ](/azure/architecture/best-practices/naming-conventions?toc=/azure/architecture/cloud-adoption-guide/toc.json) para nombrar y etiquetar los recursos como preparación para la fase de adopción intermedia.
