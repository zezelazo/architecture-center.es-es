---
title: Elección de una tecnología de orquestación de canalizaciones de datos
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 17aeb871bc815793295ed610795e5e83de72c637
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/14/2018
ms.locfileid: "29288807"
---
# <a name="choosing-a-data-pipeline-orchestration-technology-in-azure"></a>Elección de una tecnología de orquestación de canalizaciones de datos en Azure

La mayoría de las soluciones de macrodatos incluyen operaciones de procesamiento de datos repetidas, encapsuladas en flujos de trabajo. Un orquestador de canalizaciones es una herramienta que ayuda a automatizar estos flujos de trabajo. El orquestador puede programar trabajos, ejecutar flujos de trabajo y coordinar dependencias entre tareas.

## <a name="what-are-your-options-for-data-pipeline-orchestration"></a>¿De qué opciones de orquestación de canalizaciones de datos dispone?

En Azure, los siguientes servicios y herramientas cumplirán los requisitos principales de orquestación de canalizaciones, flujo de control y movimiento de datos:

- [Azure Data Factory](/azure/data-factory/)
- [Oozie en HDInsight](/azure/hdinsight/hdinsight-use-oozie-linux-mac)
- [SQL Server Integration Services (SSIS)](/sql/integration-services/sql-server-integration-services)

Estos servicios y herramientas se pueden usar de forma independiente entre sí o conjuntamente para crear una solución híbrida. Por ejemplo, Integration Runtime (IR) en Azure Data Factory V2 puede ejecutar paquetes SSIS de forma nativa en un entorno de proceso de Azure administrado. Aunque algunas de las funcionalidades de estos servicios son similares, existen varias diferencias importantes.

## <a name="key-selection-criteria"></a>Principales criterios de selección

Para restringir las opciones, empiece por responder a estas preguntas:

- ¿Necesita funcionalidades de macrodatos para mover y transformar los datos? Normalmente, esto exigiría de varios gigabytes a terabytes de datos. Si realmente lo necesitara, limite sus opciones a las más adecuadas para macrodatos.

- ¿Necesita un servicio administrado que pueda operar a escala? Si es así, seleccione uno de los servicios basados en la nube, que no están limitados por su capacidad de procesamiento local.

- ¿Algunos de sus orígenes de datos se encuentran en entornos locales? Si es así, busque opciones que puedan funcionar con orígenes de datos o destinos tanto en la nube como en entornos locales.

- ¿Los datos de origen están almacenados en Blob Storage en un sistema de archivos HDFS? En caso afirmativo, elija una opción que admita consultas de Hive.

## <a name="capability-matrix"></a>Matriz de funcionalidades

En las tablas siguientes se resumen las diferencias clave en cuanto a funcionalidades.

### <a name="general-capabilities"></a>Funcionalidades generales

| | Azure Data Factory | SQL Server Integration Services (SSIS) | Oozie en HDInsight
| --- | --- | --- | --- |
| Administrado | Sí | Sin  | Sí |
| Basado en la nube | Sí | No (local) | Sí |
| Requisito previo | Suscripción de Azure | SQL Server  | Suscripción de Azure, clúster de HDInsight |
| Herramientas de administración | Azure Portal, PowerShell, CLI, .NET SDK | SSMS, PowerShell | Shell de Bash, API REST de Oozie, interfaz de usuario web de Oozie |
| Precios | Pago por uso | Licencias/pago por características | Sin cargo adicional aparte de la ejecución del clúster de HDInsight |

### <a name="pipeline-capabilities"></a>Funcionalidades de canalización

| | Azure Data Factory | SQL Server Integration Services (SSIS) | Oozie en HDInsight
| --- | --- | --- | --- |
| Copia de datos | Sí | Sí | Sí |
| Transformaciones personalizadas | Sí | Sí | Sí (trabajos de MapReduce, Pig y Hive) |
| Puntuación de Azure Machine Learning | Sí | Sí (con scripts) | Sin  |
| HDInsight a petición | Sí | Sin  | Sin  |
| Azure Batch | Sí | Sin  | Sin  |
| Pig, Hive, MapReduce | Sí | Sin  | Sí |
| Spark | Sí | Sin  | Sin  |
| Ejecución de paquetes SSIS | Sí | Sí | Sin  |
| Flujo de control | Sí | Sí | Sí |
| Acceso a datos locales | Sí | Sí | Sin  |

### <a name="scalability-capabilities"></a>Funcionalidades de escalabilidad

| | Azure Data Factory | SQL Server Integration Services (SSIS) | Oozie en HDInsight
| --- | --- | --- | --- |
| Escalado vertical | Sí | Sin  | Sin  |
| Escalado horizontal | Sí | Sin  | Sí (mediante la adición de nodos de trabajo al clúster) |
| Optimizado para macrodatos | Sí | Sin  | Sí |

