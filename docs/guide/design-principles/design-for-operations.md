---
title: Diseño para operaciones
description: Diseñe una aplicación para que el equipo de operaciones tenga las herramientas que necesita
author: MikeWasson
ms.date: 08/30/2018
ms.openlocfilehash: a73479a7661c042d05db61907d1f993fc04ac11d
ms.sourcegitcommit: ae8a1de6f4af7a89a66a8339879843d945201f85
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/31/2018
ms.locfileid: "43326245"
---
# <a name="design-for-operations"></a>Diseño para operaciones

## <a name="design-an-application-so-that-the-operations-team-has-the-tools-they-need"></a>Diseñe una aplicación para que el equipo de operaciones tenga las herramientas que necesita

La nube ha cambiado considerablemente el rol del equipo de operaciones. Ya no es responsable de administrar el hardware y la infraestructura que hospeda la aplicación.  Ahora bien, las operaciones siguen siendo una parte fundamental de la ejecución de una aplicación en la nube de forma correcta. Algunas de las funciones importantes del equipo de operaciones son:

- Implementación
- Supervisión
- Escalado
- Respuesta a los incidentes
- Auditoría de seguridad

Un seguimiento y un registro sólidos son especialmente importantes en las aplicaciones en la nube. Implique al equipo de operaciones en el diseño y la planificación para asegurarse de que la aplicación les proporcione los datos y la información que necesitan para tener éxito.  <!-- to do: Link to DevOps checklist -->

## <a name="recommendations"></a>Recomendaciones

**Hacer que todo sea observable**. Una vez implementada y en ejecución una solución, los registros y seguimientos son la información principal sobre el sistema. El *seguimiento* registra una ruta de acceso a través del sistema y es útil para identificar los cuellos de botella, los problemas de rendimiento y los puntos de error. El *registro* captura los eventos individuales, como cambios de estado de la aplicación, errores y excepciones. Lleve un registro en producción o, si no, perderá información en los momentos en los que más la necesite.

**Instrumento para la supervisión**. La supervisión permite comprender mejor cómo funciona, bien o mal, una aplicación, en términos de disponibilidad, rendimiento y estado del sistema. Por ejemplo, la supervisión le indica si está cumpliendo el Acuerdo de Nivel de Servicio. La supervisión se produce durante el funcionamiento normal del sistema. Debe ser lo más cercana posible al tiempo real, para que el personal de operaciones pueda reaccionar ante los problemas rápidamente. Idealmente, la supervisión puede ayudar a evitar problemas antes de que se produzca un error crítico. Para más información, consulte [Supervisión y diagnóstico][monitoring].

**Instrumento para el análisis de la causa principal**. El análisis de la causa principal es el proceso de búsqueda de la causa subyacente de los errores. Se produce después de que haya surgido un error. 

**Utilizar un seguimiento distribuido**. Utilice un sistema de seguimiento distribuido que se ha diseñado para la simultaneidad, asincronía y escalado en la nube. Los seguimientos deben incluir un identificador de correlación que traspase los límites del servicio. Una única operación puede implicar llamadas a varios servicios de aplicación. Si se produce un error en una operación, el identificador de correlación ayuda a identificar la causa del error. 

**Estandarizar registros y métricas**. El equipo de operaciones necesitará agregar registros de los diversos servicios de la solución. Si cada servicio utiliza su propio formato de registro, resulta difícil o imposible obtener información útil de él. Defina un esquema común que incluya campos como identificador de correlación, nombre del evento, dirección IP del remitente, etc. Los servicios individuales pueden derivar esquemas personalizados que heredan el esquema de base y contienen campos adicionales.

**Automatizar las tareas de administración**, incluidos el aprovisionamiento, la implementación y la supervisión. La automatización de una tarea hace que sea repetible y menos propensa a errores humanos. 

**Tratar la configuración como código**. Compruebe los archivos de configuración en un sistema de control de versiones para que pueda realizar el seguimiento y controlar la versión de sus cambios, y revertirlos si es necesario. 


<!-- links -->

[monitoring]: ../../best-practices/monitoring.md


