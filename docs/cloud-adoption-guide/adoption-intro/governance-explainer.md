---
title: 'Explicación: ¿Qué es el gobierno en la nube?'
description: Explica el concepto de gobierno de los recursos de Azure y en la nube
author: petertay
ms.openlocfilehash: 63b04089aad5fc736641f8aaa6ff5247ea8ba13e
ms.sourcegitcommit: b3d74d8a89b2224fc796ce0e89cea447af43a0d4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2018
ms.locfileid: "35291252"
---
# <a name="explainer-what-is-cloud-resource-governance"></a>Explicación: ¿Qué es el gobierno de los recursos en la nube?

En la explicación [¿Cómo funciona Azure?](azure-explainer.md), ha aprendido que Azure es una colección de servidores y de hardware de red que ejecutan software y hardware virtualizado en nombre de los usuarios. Azure aporta agilidad a los departamentos de TI y desarrollo de su organización por que permite crear, leer, actualizar y eliminar fácilmente los recursos según sea necesario.

Sin embargo, aunque dar a los desarrolladores un acceso ilimitado a los recursos puede hacerlos muy ágiles, también puede producir consecuencias no deseadas en los costos. Por ejemplo, se podría aprobar que un equipo de desarrollo implemente un conjunto de recursos para pruebas pero olvida eliminarlos una vez completada la prueba. Estos recursos continuarán acumulando costos, aunque su uso no esté aprobado o no sea necesario. 

La solución a este problema es el **gobierno** de acceso a los recursos. Por gobierno se entiende el proceso continuo de administrar, supervisar y auditar el uso de los recursos de Azure para cumplir los objetivos y requisitos de su organización. 

Estos objetivos y requisitos son únicos para cada organización por lo que no es posible diseñar una solución de gobierno universal. En su lugar, Azure implementa dos herramientas de gobierno principales, el **control de acceso basado en rol (RBAC)** y la **directiva de recursos**, y cada organización debe decidir cuál usará para diseñar su modelo de gobierno.

RBAC define roles y los roles de definen las funcionalidades de un usuario al que se asigne el rol. Por ejemplo, el rol de **propietario** habilita todas las funcionalidades (crear, leer, actualizar y eliminar) para un recurso, mientras que el rol de **lector** solo habilitan la funcionalidad de lectura. Los roles pueden definirse para un ámbito amplio que se aplique a muchos tipos de recursos, para un ámbito reducido que se aplique solo a unos pocos. 

Las directivas de recursos definen las reglas de creación de recursos. Por ejemplo, una directiva de recursos puede limitar la SKU de una máquina virtual a un tamaño determinado aprobado previamente. O bien, una directiva de recursos puede exigir que se agregue una etiqueta con un centro de costo en la solicitud para crear el recurso. 

Al configurar estas herramientas, un aspecto importante a tener en cuenta es el equilibrio de gobierno frente a la agilidad organizativa. Es decir, cuanto más restrictivos sea la directiva de gobierno, menos ágiles serán los desarrolladores y los trabajadores de TI. Esto se debe a que una directiva de gobierno restrictiva puede requerir más pasos manuales, por ejemplo, que el desarrollador rellene un formulario o que envíe un correo electrónico a una persona del equipo de gobierno para crear un recurso manualmente. El equipo de gobierno tiene una capacidad finita y podría acumular tareas, lo que provocaría que los equipos de desarrollo no puedan producir a la espera de que se creen sus recursos, y que recursos no necesarios acumulen costos mientras esperan a ser eliminados.

## <a name="next-steps"></a>Pasos siguientes

El siguiente paso en la adopción de Azure es [comprender la identidad digital en Azure](tenant-explainer.md) y cómo [crear su primer usuario en Azure AD][docs-add-users-to-aad].

<!-- Links -->

[docs-add-users-to-aad]: /azure/active-directory/add-users-azure-active-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json