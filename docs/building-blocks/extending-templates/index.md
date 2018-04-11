---
title: Extensión de las funciones de las plantillas de Azure Resource Manager
description: Describe sugerencias y trucos sobre cómo ampliar la funcionalidad de las plantillas de Azure Resource Manager
author: petertay
ms.date: 06/09/2017
ms.openlocfilehash: 33ae6850ffa5b28108f30475804be5347859f0c3
ms.sourcegitcommit: ea7108f71dab09175ff69322874d1bcba800a37a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/17/2018
---
# <a name="extend-azure-resource-manager-template-functionality"></a>Extensión de las funciones de las plantillas de Azure Resource Manager

En 2016, el equipo de patrones y procedimientos de Microsoft creó un conjunto de [bloques de creación de plantillas](https://github.com/mspnp/template-building-blocks/wiki) de Azure Resource Manager con el fin de simplificar la implementación de recursos. Cada bloque de creación contiene un conjunto de plantillas integradas que implementan conjuntos de recursos especificados por los archivos de parámetros independientes.

Las plantillas de bloques de creación están diseñadas para combinarse y crear implementaciones mayores y más complejas. Por ejemplo, la implementación de una máquina virtual en Azure requiere una red virtual, cuentas de almacenamiento y otros recursos. La [plantilla de bloque de creación de red virtual](https://github.com/mspnp/template-building-blocks/wiki/VNet-(v1)) implementa una red virtual y las subredes. La [plantilla de bloque de creación de máquina virtual](https://github.com/mspnp/template-building-blocks/wiki/Windows-and-Linux-VMs-(v1)) implementa las cuentas de almacenamiento, las interfaces de red y las máquinas virtuales reales. A continuación, puede crear un script o una plantilla para llamar a ambas plantillas de bloques de creación con sus archivos de parámetros correspondientes, a fin de implementar una arquitectura completa con una sola operación.

Al desarrollar las plantillas de bloques de creación, el equipo de patrones y procedimientos diseñó varios conceptos para ampliar la funcionalidad de las plantillas de Azure Resource Manager. En esta serie, vamos a describir algunos de estos conceptos para que pueda usarlos en sus propias plantillas.

> [!NOTE]
> En estos artículos se supone que tiene un conocimiento avanzado de las plantillas Azure Resource Manager.