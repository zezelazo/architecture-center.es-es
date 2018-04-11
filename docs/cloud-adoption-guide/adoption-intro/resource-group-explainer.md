---
title: 'Explicación: ¿Qué es un grupo de recursos de Azure?'
description: Explica la función de Azure interna de un grupo de recursos
author: petertay
ms.openlocfilehash: e7c7334bd88c28f57498486bd2bed3c349565222
ms.sourcegitcommit: 2e8b06e9c07875d65b91d5431bfd4bc465a7a242
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/09/2018
---
# <a name="what-is-an-azure-resource-group"></a>¿Qué es un grupo de recursos de Azure?

En el artículo de la explicación [¿Qué es Azure Resource Manager?](resource-manager-explainer.md) ha aprendido que Azure Resource Manager requiere un **identificador de grupo de recursos** al realizar una llamada para crear, leer, actualizar o eliminar un recurso. Este identificador de grupo de recursos hace referencia a un **grupo de recursos**. Un grupo de recursos es simplemente un identificador que Azure Resource Manager aplica a los recursos para agruparlos. Este identificador de grupo de recursos permite a Azure Resource Manager realizar operaciones en un grupo de recursos que comparten este identificador.

Por ejemplo, un usuario puede realizar una llamada de **eliminación** a una API RESTful de Azure Resource Manager especificando el identificador de grupo de recursos sin incluir ningún identificador de recurso específico. Azure Resource Manager consulta una base de datos interna de Azure para todos los recursos que tengan el identificador de grupo de recursos especificado y llama a la API RESTful para eliminar cada uno de los recursos.

Un grupo de recursos no puede incluir recursos de distintas suscripciones. Esto se debe a que no hay una relación de uno a varios entre el identificador de inquilino y el identificador de suscripción: varias suscripciones pueden confiar en el mismo inquilino para que proporcione autenticación y autorización pero cada suscripción puede confiar solo en un inquilino. También hay una relación de uno a varios entre el identificador de suscripción y el identificador de grupo de recursos: varios grupos de recursos pueden pertenecer a la misma suscripción pero cada grupo de recursos puede pertenecer solo a una suscripción. Por último, hay una relación de uno a varios entre el identificador de grupo de recursos y el identificador de recurso: un único grupo de recursos puede tener varios recursos pero cada recurso puede pertenecer solo a un único grupo de recursos.

## <a name="next-steps"></a>pasos siguientes

* Ahora que ha aprendido acerca de los grupos de recursos de Azure, amplíe sus conocimientos básicos [sobre cómo restringir el acceso a los recursos](/azure/active-directory/active-directory-understanding-resource-access?toc=/azure/architecture/cloud-adoption-guide/toc.json). Aunque no forme parte de la fase de adopción básica, es importante durante la fase de adopción intermedia. Después, puede [crear el primer grupo de recursos](/azure/azure-resource-manager/resource-group-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json) y revisar la [guía de diseño para grupos de recursos de Azure](resource-group.md).
