---
title: Diseño de escalado horizontal
description: Las aplicaciones en la nube deben diseñarse para el escalado horizontal.
author: MikeWasson
ms.openlocfilehash: 8207f322d4312f6a30a8b0db7328b272c1d82de0
ms.sourcegitcommit: 26b04f138a860979aea5d253ba7fecffc654841e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/19/2018
ms.locfileid: "36206782"
---
# <a name="design-to-scale-out"></a>Diseño de escalado horizontal

## <a name="design-your-application-so-that-it-can-scale-horizontally"></a>Diseñe la aplicación para que se pueda escalar horizontalmente.

Una ventaja importante de la nube es el escalado elástico: la capacidad de usar tanta capacidad según sea necesario, escalando horizontalmente cuando aumente la carga y reduciendo horizontalmente cuando no se requiera la capacidad adicional. Diseñe la aplicación para que pueda escalarse horizontalmente, agregando o quitando nuevas instancias a medida que se requiera.

## <a name="recommendations"></a>Recomendaciones

**Evite la permanencia de las instancias**. La permanencia, o *la afinidad de la sesión*, se produce cuando las solicitudes del mismo cliente se enrutan siempre al mismo servidor. La permanencia limita la capacidad de la aplicación de escalar horizontalmente. Por ejemplo, el tráfico de un usuario de gran volumen no se distribuirá entre las instancias. Entre las causas de esta permanencia está el almacenamiento del estado de sesión en la memoria y el uso de claves específicas del equipo para el cifrado. Asegúrese de que cualquier instancia puede controlar cualquier solicitud. 

**Identifique los cuellos de botella**. El escalado horizontal no es una solución mágica para todos los problemas de rendimiento. Por ejemplo, si la base de datos de back-end es el cuello de botella, no ayudará el hecho de agregar más servidores web. Identifique y resuelva los cuellos de botella del equipo antes de lanzar más instancias al problema. Los elementos con estado del sistema son la causa más probable de los cuellos de botella. 

**Descomponga las cargas de trabajo de acuerdo con los requisitos de escalabilidad.**  A menudo, las aplicaciones constan de varias cargas de trabajo, con diferentes requisitos para el escalado. Por ejemplo, una aplicación podría tener un sitio orientado al público y un sitio de administración independiente. El sitio público puede experimentar un repentino aumento de tráfico, mientras que el sitio de administración tiene una carga más pequeña y predecible. 

**Descargue tareas que consumen muchos recursos.** Las tareas que requieren una gran cantidad de CPU o recursos de E/S se deben mover a [trabajos en segundo plano][background-jobs] cuando sea posible, a fin de reducir la carga en el front-end que está controlando las solicitudes de usuario.

**Use características de escalado automático integrado**. Muchos servicios de proceso de Azure tienen compatibilidad integrada con el escalado automático. Si la aplicación tiene una carga de trabajo predecible y regular, aplique el escalado horizontal según una programación. Por ejemplo, escale horizontalmente durante el horario laboral. En caso contrario, si la carga de trabajo no es predecible, use métricas de rendimiento como la longitud de cola de solicitudes o la CPU para activar el escalado automático. Para conocer los procedimientos recomendados del escalado automático, consulte [Autoscaling][autoscaling] (Escalado automático).

**Considere la posibilidad de utilizar un escalado automático agresivo para cargas de trabajo críticas**. En el caso de las cargas de trabajo críticas, probablemente deseará anticiparse a la demanda. Es mejor agregar nuevas instancias rápidamente bajo una carga intensa para controlar el tráfico adicional y, a continuación, reducir el escalado gradualmente.

**Aplique un diseño para la reducción horizontal**.  Recuerde que con el escalado elástico, la aplicación tendrá períodos de reducción horizontal durante los cuales se quitan instancias. La aplicación debe administrar correctamente las instancias que se quitan. Aquí se muestran algunas maneras de controlar la reducción horizontal:

- Escuche los eventos de apagado (cuando estén disponibles) y apague correctamente. 
- Los clientes o los consumidores de un servicio deben admitir el control de errores transitorios y el reintento. 
- Para las tareas de ejecución prolongada, considere la posibilidad de dividir el trabajo, utilizando los puntos de control o el patrón [Pipes and Filters][pipes-filters-pattern]. 
- Coloque los elementos de trabajo en una cola para que otra instancia puede recoger el trabajo, en caso de que se quite una instancia en medio de procesamiento. 


<!-- links -->

[autoscaling]: ../../best-practices/auto-scaling.md
[background-jobs]: ../../best-practices/background-jobs.md
[pipes-filters-pattern]: ../../patterns/pipes-and-filters.md
