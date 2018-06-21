---
title: 'Explicación: ¿Qué es Azure Resource Manager?'
description: Explica el funcionamiento interno de Azure Resource Manager
author: petertay
ms.openlocfilehash: 60f09901bdc4b292abd73335b78c7d56a76f27a6
ms.sourcegitcommit: 2e8b06e9c07875d65b91d5431bfd4bc465a7a242
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/09/2018
ms.locfileid: "29062033"
---
# <a name="explainer-what-is-azure-resource-manager"></a>Explicación: ¿Qué es Azure Resource Manager?

En la explicación [¿Cómo funciona Azure?](azure-explainer.md), ha aprendido acerca de la arquitectura interna de Azure. Esta arquitectura incluye un front-end que hospeda las aplicaciones distribuidas que administran los servicios internos de Azure.

El front-end de Azure incluye un servicio denominado Azure Resource Manager. Azure Resource Manager es responsable del ciclo de vida de los recursos hospedados en Azure desde la creación hasta la eliminación. Hay muchas formas de interactuar con Azure Resource Manager &mdash;mediante Powershell, la interfaz de la línea de comandos de Azure, los SDK&mdash; pero cada una de estas herramientas es sencillamente un contenedor sobre la API RESTful hospedada por Azure Resource Manager.

La API RESTful proporcionada por Azure Resource Manager es una interfaz coherente sobre un conjunto de **proveedores de recursos**. Los proveedores de recursos son simplemente servicios de Azure que crean, leen, actualizan y eliminan recursos de Azure. De hecho, la API RESTful incluye métodos para cada una de estas funciones. 

La API RESTful requiere un token de acceso para el usuario, un **identificador de suscripción** y algo nuevo, un **identificador de grupo de recursos**. Trataremos sobre grupos de recursos en el artículo de la [explicación sobre el grupo de recursos](resource-group-explainer.md). Azure Resource Manager también requiere el **identificador de inquilino**, que está codificado como parte del token de acceso. 

Cuando se recibe una llamada de API válida para crear un recurso, Azure Resource Manager busca capacidad en la región especificada y copia los archivos necesarios en una ubicación de almacenamiento provisional. Después, se envía la solicitud al controlador de tejido en el bastidor y este controlador asigna los recursos. El controlador de tejido responde a la solicitud con una notificación de operación correcta o errónea, junto con un **identificador de recurso** para el recurso recién creado. Estos cuatro identificadores se almacenan internamente en Azure y actúan juntos como identificador único para un recurso implementado.

## <a name="next-steps"></a>pasos siguientes

* Ahora que comprende el funcionamiento interno de Azure Resource Manager, obtenga información [acerca de los grupos de recursos](resource-group-explainer.md) que le ayude a crear el primer grupo de recursos.
