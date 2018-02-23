---
title: "Selección de una tecnología de procesamiento por lotes"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: bfb850ee8e9d8fd41927b4ca3b612e15b5ae6b11
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/14/2018
---
# <a name="choosing-a-batch-processing-technology-in-azure"></a>Selección de una tecnología de procesamiento por lotes en Azure

Las soluciones de macrodatos a menudo utilizan trabajos por lotes de ejecución prolongada para filtrar, agregar y preparan de otra forma los datos para su análisis. Normalmente estos trabajos implican leer archivos de código fuente desde un almacenamiento escalable (por ejemplo, HDFS, Azure Data Lake Store, and Azure Storage), procesarlos y escribir la salida a los nuevos archivos de almacenamiento escalable. 

El requisito clave de estos motores de procesamiento de lotes es la capacidad para escalar horizontalmente los cálculos, con el fin de administrar un gran volumen de datos. Sin embargo, a diferencia del procesamiento en tiempo real, el procesamiento por lotes se espera que tenga las latencias (el tiempo transcurrido entre la ingesta de datos y calcular un resultado) que se miden en minutos y horas.

## <a name="what-are-your-options-when-choosing-a-batch-processing-technology"></a>¿Cuáles son las opciones a la hora de elegir una tecnología de procesamiento por lotes?

En Azure, los almacenes de datos siguientes cumplirán los requisitos principales para el procesamiento por lotes:

- [Análisis con Azure Data Lake](/azure/data-lake-analytics/)
- [Azure SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is)
- [HDInsight con Spark](/azure/hdinsight/spark/apache-spark-overview)
- [HDInsight con Hive](/azure/hdinsight/hadoop/hdinsight-use-hive)
- [HDInsight con Hive LLAP](/azure/hdinsight/interactive-query/apache-interactive-query-get-started)

## <a name="key-selection-criteria"></a>Principales criterios de selección

Para restringir las opciones, empiece por responder a estas preguntas:

- ¿Quiere un servicio administrado en lugar de administrar sus propios servidores?

- ¿Desea crear la lógica del procesamiento por lotes de forma declarativa o imperativa?

- ¿Realizará el procesamiento por lotes en ráfagas? Si es así, considere el uso de opciones que le permitan hacer una pausa del clúster o cuyo modelo de precios es por trabajo por lotes.

- ¿Necesita consultar almacenes de datos relacionales junto con procesamiento por lotes, por ejemplo, para buscar datos de referencia? En caso afirmativo, considere la posibilidad de usar opciones que permita la realización de consultas en almacenes relacionales externos.

## <a name="capability-matrix"></a>Matriz de funcionalidades

En las tablas siguientes se resumen las diferencias clave en cuanto a funcionalidades. 

### <a name="general-capabilities"></a>Funcionalidades generales

| | Análisis con Azure Data Lake | Azure SQL Data Warehouse | HDInsight con Spark | HDInsight con Hive | HDInsight con Hive LLAP |
| --- | --- | --- | --- | --- | --- |
| Es un servicio administrado | Sí | Sí | Sí <sup>1</sup> | Sí <sup>1</sup> | Sí <sup>1</sup> |
| Admite la pausa del proceso | Sin  | Sí | Sin  | Sin  | Sin  |
| Almacenes de datos relacionales | Sí | Sí | Sin  | Sin  | Sin  |
| Capacidad de programación | U-SQL | T-SQL | Python, Scala, Java, R | HiveQL | HiveQL |
| Paradigma de programación | Mezcla de declarativa e imperativa  | Declarativa | Mezcla de declarativa e imperativa | Declarativa | Declarativa | 
| Modelo de precios | Por trabajo por lotes | Por hora de clúster | Por hora de clúster | Por hora de clúster | Por hora de clúster |  

[1] Con configuración y escalado manuales.
 
### <a name="integration-capabilities"></a>Funcionalidades de integración
| | Análisis con Azure Data Lake | SQL Data Warehouse | HDInsight con Spark | HDInsight con Hive | HDInsight con Hive LLAP |
| --- | --- | --- | --- | --- | --- |
| Acceso desde Azure Data Lake Store | Sí | Sí | Sí | Sí | Sí |
| Consultas desde Azure Storage | Sí | Sí | Sí | Sí | Sí |
| Consulta de almacenes relacionales externos | Sí | Sin  | Sí | Sin  | Sin  |

### <a name="scalability-capabilities"></a>Funcionalidades de escalabilidad
| | Análisis con Azure Data Lake | SQL Data Warehouse | HDInsight con Spark | HDInsight con Hive | HDInsight con Hive LLAP |
| --- | --- | --- | --- | --- | --- |
| Granularidad de escalabilidad horizontal  | Por trabajo | Por clúster | Por clúster | Por clúster | Por clúster |
| Escalado horizontal rápido (en memos de 1 minuto) | Sí | Sí | Sin  | Sin  | Sin  |
| Admite el almacenamiento en caché en memoria de datos | Sin  | Sí | Sí | Sin  | Sí | 

### <a name="security-capabilities"></a>Funcionalidades de seguridad
| | Análisis con Azure Data Lake | SQL Data Warehouse | HDInsight con Spark | Apache Hive en HDInsight | Hive LLAP en HDInsight |
| --- | --- | --- | --- | --- | --- |
| Autenticación  | Azure Active Directory (Azure AD) | SQL/Azure AD | Sin  | local/Azure AD <sup>1</sup> | local/Azure AD <sup>1</sup> |
| Autorización  | Sí | Sí| Sin  | Sí <sup>1</sup> | Sí <sup>1</sup> |
| Auditoría  | Sí | Sí | Sin  | Sí <sup>1</sup> | Sí <sup>1</sup> |
| Cifrado de datos en reposo | Sí| Sí <sup>2</sup> | Sí | Sí | Sí |
| Seguridad de nivel de fila | Sin  | Sí | Sin  | Sí <sup>1</sup> | Sí <sup>1</sup> |
| Admite firewalls | Sí | Sí | Sí | Sí <sup>3</sup> | Sí <sup>3</sup> |
| Enmascaramiento de datos dinámicos | Sin  | Sin  | Sin  | Sí <sup>1</sup> | Sí <sup>1</sup> |

[1] Requiere el uso de un [clúster de HDInsight unido a un dominio](/azure/hdinsight/domain-joined/apache-domain-joined-introduction).

[2] Requiere el uso de cifrado de datos transparente (TDE) para cifrar y descifrar los datos en reposo.

[3] Se admite cuando se usa [en una instancia de Azure Virtual Network](/azure/hdinsight/hdinsight-extend-hadoop-virtual-network).
