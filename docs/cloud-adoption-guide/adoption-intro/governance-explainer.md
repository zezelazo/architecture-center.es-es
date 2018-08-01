---
title: ¿Qué es el gobierno en la nube?
description: Explicación del concepto de gobierno del acceso a los recursos en Azure
author: petertay
ms.openlocfilehash: 3338ed5652296c5cd01277a51be1cb5893320b65
ms.sourcegitcommit: c704d5d51c8f9bbab26465941ddcf267040a8459
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2018
ms.locfileid: "39229208"
---
# <a name="what-is-cloud-resource-governance"></a>¿Qué es el gobierno de los recursos en la nube?

En [¿Cómo funciona Azure?](azure-explainer.md), ha aprendido que Azure es una colección de servidores y de hardware de red que ejecutan software y hardware virtualizado en nombre de los usuarios. Azure aporta agilidad a los departamentos de TI y desarrollo de su organización por que permite crear, leer, actualizar y eliminar fácilmente los recursos según sea necesario.

Sin embargo, aunque dar a los desarrolladores un acceso ilimitado a los recursos puede hacerlos muy ágiles, también puede producir consecuencias no deseadas en los costos. Por ejemplo, se podría aprobar que un equipo de desarrollo implemente un conjunto de recursos para pruebas pero olvida eliminarlos una vez completada la prueba. Estos recursos continuarán acumulando costos, aunque su uso no esté aprobado o no sea necesario. 

La solución a este problema es el **gobierno** de acceso a los recursos. Por gobierno se entiende el proceso continuo de administrar, supervisar y auditar el uso de los recursos de Azure para cumplir los objetivos y requisitos de su organización. 

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE2ii94] 

Estos objetivos y requisitos son únicos para cada organización por lo que no es posible diseñar una solución de gobierno universal. En su lugar, Azure implementa dos herramientas de gobierno principales, el **control de acceso basado en rol (RBAC)** y la **directiva de recursos**, y cada organización debe decidir cuál usará para diseñar su modelo de gobierno.

RBAC define roles y los roles de definen las funcionalidades de un usuario al que se asigne el rol. Por ejemplo, el rol de **propietario** habilita todas las funcionalidades (crear, leer, actualizar y eliminar) para un recurso, mientras que el rol de **lector** solo habilitan la funcionalidad de lectura. Los roles pueden definirse para un ámbito amplio que se aplique a muchos tipos de recursos, para un ámbito reducido que se aplique solo a unos pocos. 

Las directivas de recursos definen las reglas de creación de recursos. Por ejemplo, una directiva de recursos puede limitar la SKU de una máquina virtual a un tamaño determinado aprobado previamente. O bien, una directiva de recursos puede exigir que se agregue una etiqueta con un centro de costo en la solicitud para crear el recurso. 

Al configurar estas herramientas, un aspecto importante a tener en cuenta es el equilibrio de gobierno frente a la agilidad organizativa. Es decir, cuanto más restrictivos sea la directiva de gobierno, menos ágiles serán los desarrolladores y los trabajadores de TI. Esto se debe a que una directiva de gobierno restrictiva puede requerir más pasos manuales, por ejemplo, que el desarrollador rellene un formulario o que envíe un correo electrónico a una persona del equipo de gobierno para crear un recurso manualmente. El equipo de gobierno tiene una capacidad finita y podría acumular tareas, lo que provocaría que los equipos de desarrollo no puedan producir a la espera de que se creen sus recursos, y que recursos no necesarios acumulen costos mientras esperan a ser eliminados.

## <a name="next-steps"></a>Pasos siguientes

Ahora que comprende el concepto de gobierno de recursos en la nube, vea [cómo se administra el acceso a los recursos](azure-resource-access.md) en Azure como preparación para aprender [cómo diseñar un modelo de gobierno](governance-how-to.md).