---
title: Selección de una tecnología de procesamiento por lotes
description: ''
author: zoinerTejada
ms:date: 11/03/2018
ms.openlocfilehash: 2314a1413fa674f43bd7a4bcb868a5322ad99497
ms.sourcegitcommit: 225251ee2dd669432a9c9abe3aa8cd84d9e020b7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/04/2018
ms.locfileid: "50982004"
---
# <a name="choosing-a-batch-processing-technology-in-azure"></a>Selección de una tecnología de procesamiento por lotes en Azure

Las soluciones de macrodatos a menudo utilizan trabajos por lotes de ejecución prolongada para filtrar, agregar y preparan de otra forma los datos para su análisis. Normalmente estos trabajos implican leer archivos de código fuente desde un almacenamiento escalable (por ejemplo, HDFS, Azure Data Lake Store, and Azure Storage), procesarlos y escribir la salida a los nuevos archivos de almacenamiento escalable. 

El requisito clave de estos motores de procesamiento de lotes es la capacidad para escalar horizontalmente los cálculos, con el fin de administrar un gran volumen de datos. Sin embargo, a diferencia del procesamiento en tiempo real, el procesamiento por lotes se espera que tenga las latencias (el tiempo transcurrido entre la ingesta de datos y calcular un resultado) que se miden en minutos y horas.

## <a name="technology-choices-for-batch-processing"></a>Opciones de tecnología para el procesamiento por lotes

### <a name="azure-sql-data-warehouse"></a>Azure SQL Data Warehouse

[SQL Data Warehouse](/azure/sql-data-warehouse/) es un sistema distribuido diseñado para realizar análisis con datos de gran tamaño. Admite el procesamiento paralelo masivo (MPP), lo que lo hace idóneo para ejecutar análisis de alto rendimiento. Considere la posibilidad de usar SQL Data Warehouse cuando tenga grandes cantidades de datos (más de 1 TB) y esté ejecutando una carga de trabajo de análisis que aprovecharía el paralelismo.

### <a name="azure-data-lake-analytics"></a>Análisis con Azure Data Lake

[Data Lake Analytics](/azure/data-lake-analytics/data-lake-analytics-overview) es un servicio de trabajos de análisis a petición. Está optimizado para el procesamiento distribuido de conjuntos de datos muy grandes almacenados en Azure Data Lake Store. 

- Lenguajes: [U-SQL](/azure/data-lake-analytics/data-lake-analytics-u-sql-get-started) (incluidas las extensiones de Python, R y C#).
-  Se integra con Azure Data Lake Store, los blobs de Azure Storage, Azure SQL Database y SQL Data Warehouse.
- El modelo de precios es por trabajo.

### <a name="hdinsight"></a>HDInsight

HDInsight es un servicio de Hadoop administrado. Se usa para implementar y administrar clústeres de Hadoop en Azure. Para el procesamiento por lotes, puede usar [Spark](/azure/hdinsight/spark/apache-spark-overview), [Hive](/azure/hdinsight/hadoop/hdinsight-use-hive), [Hive LLAP](/azure/hdinsight/interactive-query/apache-interactive-query-get-started) y [MapReduce](/azure/hdinsight/hadoop/hdinsight-use-mapreduce).

- Lenguajes: R, Python, Java, Scala y SQL
- Autenticación Kerberos con Active Directory y control de acceso basado en Apache Ranger
- Proporciona control total del clúster de Hadoop

### <a name="azure-databricks"></a>Azure Databricks 

[Azure Databricks](/azure/azure-databricks/) es una plataforma de análisis basada en Apache Spark. Se puede considerar como "Spark como servicio". Es la forma más fácil de usar Spark en la plataforma Azure.  

- Lenguajes: R, Python, Java, Scala y Spark SQL
- Horas de inicio rápido del clúster, terminación automática, escalado automático.
- Administra automáticamente el clúster de Spark.
- Integración incorporada con Azure Blob Storage, Azure Data Lake Storage (ADLS), Azure SQL Data Warehouse (SQL DW) y otros servicios. Consulte [Orígenes de datos](https://docs.azuredatabricks.net/spark/latest/data-sources/index.html).
- Autenticación de usuario con Azure Active Directory.
- [Cuadernos](https://docs.azuredatabricks.net/user-guide/notebooks/index.html) web para la exploración de datos y la colaboración. 
- Admite [clústeres con GPU habilitado](https://docs.azuredatabricks.net/user-guide/clusters/gpu.html)

### <a name="azure-distributed-data-engineering-toolkit"></a>Azure Distributed Data Engineering Toolkit 

[Distributed Data Engineering Toolkit](https://github.com/azure/aztk) (AZTK) es una herramienta para el aprovisionamiento de Spark a petición en clústeres de Docker en Azure. 

AZTK no es un servicio de Azure. Más bien es una herramienta de cliente con una interfaz de CLI y SDK de Python que se basa en Azure Batch. Esta opción ofrece el máximo control sobre la infraestructura al implementar un clúster de Spark.

- Aporte su propia imagen de Docker.
- Use máquinas virtuales de prioridad baja para lograr un 80 % de descuento.
- Clústeres de modo mixto que usan no solo máquinas virtuales de prioridad baja sino también máquinas virtuales dedicadas.
- Compatibilidad integrada con una conexión de Azure Blob Storage y Azure Data Lake.

## <a name="key-selection-criteria"></a>Principales criterios de selección

Para restringir las opciones, empiece por responder a estas preguntas:

- ¿Quiere un servicio administrado en lugar de administrar sus propios servidores?

- ¿Desea crear la lógica del procesamiento por lotes de forma declarativa o imperativa?

- ¿Realizará el procesamiento por lotes en ráfagas? Si es así, considere el uso de opciones que le permitan terminar automáticamente el clúster o cuyo modelo de precios se por trabajo por lotes.

- ¿Necesita consultar almacenes de datos relacionales junto con procesamiento por lotes, por ejemplo, para buscar datos de referencia? En caso afirmativo, considere la posibilidad de usar opciones que permita la realización de consultas en almacenes relacionales externos.

## <a name="capability-matrix"></a>Matriz de funcionalidades

En las tablas siguientes se resumen las diferencias clave en cuanto a funcionalidades. 

### <a name="general-capabilities"></a>Funcionalidades generales

| | Análisis con Azure Data Lake | Azure SQL Data Warehouse | HDInsight | Azure Databricks |
| --- | --- | --- | --- | --- | --- |
| Es un servicio administrado | SÍ | SÍ | Sí <sup>1</sup> | SÍ | 
| Almacenes de datos relacionales | SÍ | SÍ | No | Sin  |
| Modelo de precios | Por trabajo por lotes | Por hora de clúster | Por hora de clúster | Unidad de Databricks<sup>2</sup> + hora de clúster |

[1] Con configuración y escalado manuales.

[2] Una unidad de Databricks (DBU) es una unidad de funcionalidad de procesamiento por hora.

### <a name="capabilities"></a>Capacidades

| | Análisis con Azure Data Lake | SQL Data Warehouse | HDInsight con Spark | HDInsight con Hive | HDInsight con Hive LLAP | Azure Databricks |
| --- | --- | --- | --- | --- | --- | --- |
| Escalado automático | Sin  | No | No | No | No | SÍ |
| Granularidad de escalabilidad horizontal  | Por trabajo | Por clúster | Por clúster | Por clúster | Por clúster | Por clúster |
| Admite el almacenamiento en caché en memoria de datos | Sin  | SÍ | SÍ | Sin  | SÍ | SÍ |
| Consulta de almacenes relacionales externos | SÍ | Sin  | Sí | No | No | SÍ |
| Autenticación  | Azure AD | SQL/Azure AD | Sin  | Azure AD<sup>1</sup> | Azure AD<sup>1</sup> | Azure AD |
| Auditoría  | SÍ | SÍ | Sin  | Sí <sup>1</sup> | Sí <sup>1</sup> | SÍ |
| Seguridad de nivel de fila | Sin  | No | Sin  | Sí <sup>1</sup> | Sí <sup>1</sup> | Sin  |
| Admite firewalls | SÍ | Sí | SÍ | Sí <sup>2</sup> | Sí <sup>2</sup> | Sin  |
| Enmascaramiento de datos dinámicos | Sin  | No | Sin  | Sí <sup>1</sup> | Sí <sup>1</sup> | Sin  |

[1] Requiere el uso de un [clúster de HDInsight unido a un dominio](/azure/hdinsight/domain-joined/apache-domain-joined-introduction).

[2] Se admite cuando [se usa en una instancia de Azure Virtual Network](/azure/hdinsight/hdinsight-extend-hadoop-virtual-network).
