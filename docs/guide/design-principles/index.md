---
title: Principios de diseño para las aplicaciones de Azure
description: Principios de diseño para las aplicaciones de Azure
author: MikeWasson
layout: LandingPage
ms.openlocfilehash: 57b04839e14804ad97fc9c86e1f9c4fe6e0da472
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2017
---
# <a name="design-principles-for-azure-applications"></a>Principios de diseño para las aplicaciones de Azure

Siga estos principios de diseño para que la aplicación sea más escalable, resistente y administrable. 

**[Diseñe para la recuperación automática](self-healing.md)**. En un sistema distribuido, se producen errores. Diseñe la aplicación para que se recupere automáticamente cuando esto suceda.

**[Haga que todo sea redundante](redundancy.md)**. Cree redundancia en la aplicación, para evitar tener puntos únicos de error.
 
**[Minimice la coordinación](minimize-coordination.md)**. Minimice la coordinación entre los servicios de aplicación para lograr escalabilidad.
 
**[Diseñe el escalado horizontal](scale-out.md)**. Diseñe la aplicación para que pueda escalarse horizontalmente, agregando o quitando nuevas instancias a medida que se requiera.

**[Cree particiones alrededor de límites](partition.md)**. Use particiones para evitar los limites en la base de datos, la red y el proceso.

**[Diseñe para las operaciones](design-for-operations.md)**. Diseñe la aplicación para que el equipo de operaciones tenga las herramientas que necesita.

**[Use servicios administrados](managed-services.md)**. Cuando sea posible, use la plataforma como servicio (PaaS) en lugar de la infraestructura como servicio (IaaS).

**[Use el mejor almacén de datos para el trabajo](use-the-best-data-store.md)**. Elija la tecnología de almacenamiento que encaje mejor con sus datos y el modo en que se utilizarán. 
 
**[Diseñe para evolucionar](design-for-evolution.md)**. Todas las aplicaciones correctas cambian con el tiempo. Un diseño evolutivo es clave para una innovación continua.

**[Cree teniendo en cuenta las necesidades de la empresa](build-for-business.md)**. Cada decisión de diseño debe estar justificada por un requisito empresarial.

