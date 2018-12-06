---
title: Elección de una tecnología de orquestación de canalizaciones de datos
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.openlocfilehash: 7d1fddf54216b756a5dc2c183a43449a2f45a122
ms.sourcegitcommit: e7e0e0282fa93f0063da3b57128ade395a9c1ef9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/05/2018
ms.locfileid: "52902389"
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
| Administrado | SÍ | Sin  | SÍ |
| Basado en la nube | SÍ | No (local) | SÍ |
| Requisito previo | Suscripción de Azure | SQL Server  | Suscripción de Azure, clúster de HDInsight |
| Herramientas de administración | Azure Portal, PowerShell, CLI, .NET SDK | SSMS, PowerShell | Shell de Bash, API REST de Oozie, interfaz de usuario web de Oozie |
| Precios | Pago por uso | Licencias/pago por características | Sin cargo adicional aparte de la ejecución del clúster de HDInsight |

### <a name="pipeline-capabilities"></a>Funcionalidades de canalización

| | Azure Data Factory | SQL Server Integration Services (SSIS) | Oozie en HDInsight
| --- | --- | --- | --- |
| Copia de datos | SÍ | Sí | SÍ |
| Transformaciones personalizadas | SÍ | SÍ | Sí (trabajos de MapReduce, Pig y Hive) |
| Puntuación de Azure Machine Learning | SÍ | Sí (con scripts) | Sin  |
| HDInsight a petición | SÍ | No | Sin  |
| Azure Batch | SÍ | No | Sin  |
| Pig, Hive, MapReduce | SÍ | Sin  | SÍ |
| Spark | SÍ | No | Sin  |
| Ejecución de paquetes SSIS | SÍ | SÍ | Sin  |
| Flujo de control | SÍ | Sí | SÍ |
| Acceso a datos locales | SÍ | SÍ | Sin  |

### <a name="scalability-capabilities"></a>Funcionalidades de escalabilidad

| | Azure Data Factory | SQL Server Integration Services (SSIS) | Oozie en HDInsight
| --- | --- | --- | --- |
| Escalado vertical | SÍ | No | Sin  |
| Escalado horizontal | SÍ | Sin  | Sí (mediante la adición de nodos de trabajo al clúster) |
| Optimizado para macrodatos | SÍ | Sin  | SÍ |

