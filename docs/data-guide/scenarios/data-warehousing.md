---
title: Almacenamiento de datos y data marts
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: eec883c68cf94637c3061814d0841c73b58d7e52
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/14/2018
---
# <a name="data-warehousing-and-data-marts"></a>Almacenamiento de datos y data marts

Un almacenamiento de datos es un repositorio central, empresarial y relacional de datos integrados procedentes de uno o más orígenes dispares que incluye muchas o todas las áreas temáticas. Los almacenamientos de datos almacenan datos históricos y actuales, y se utilizan para realizar informes y análisis de los datos de diferentes maneras.

![Almacenamiento de datos en Azure](./images/data-warehousing.png)

Para pasar los datos a un almacenamiento de datos, estos se extraen de forma periódica de diversos orígenes que contienen información empresarial de importancia. Cuando se mueven los datos se puede dar formato, limpiar, validar, resumir y reorganizar. Como alternativa, los datos pueden almacenarse en el nivel más bajo de detalle, con vistas agregadas proporcionadas por el almacenamiento de datos para realizar informes. En cualquier caso, el almacenamiento de datos se convierte en un espacio de almacenamiento permanente de los datos utilizados en informes, en el análisis y en la toma de decisiones empresariales importantes mediante herramientas de inteligencia empresarial (BI).

## <a name="data-marts-and-operational-data-stores"></a>Data marts y almacenes de datos operativos

La administración de datos a escala es compleja y cada vez es menos habitual tener un almacenamiento de datos único que representa todos los datos de toda la empresa. En su lugar, las organizaciones crean almacenamientos de datos más pequeños y concretos llamados *data marts*, que exponen los datos deseados con fines de análisis. Un proceso de orquestación rellena los data marts con datos que se mantienen en un almacén de datos de tipo operativo. El almacén de datos de tipo operativo actúa como intermediario entre el sistema transaccional de origen y el data mart. Los datos administrados por el almacén de datos operativo son una versión limpia de los datos presentes en el sistema transaccional de origen y normalmente son un subconjunto de los datos históricos mantenidos por el almacenamiento de datos o el data mart. 

## <a name="when-to-use-this-solution"></a>Cuándo se debe utilizar esta solución

Elija un almacenamiento de datos cuando necesite convertir grandes cantidades de datos procedentes de los sistemas operacionales a un formato que sea fácil de entender, actual y preciso. No es necesario que los almacenamientos de datos sigan la misma estructura de datos simplificada utilizada en las bases de datos operativas y de OLTP. Puede usar nombres de columna que tengan sentido para los analistas y los usuarios empresariales, reestructurar el esquema para simplificar las relaciones de datos y consolidar varias tablas en una. Estos pasos ayudan a guiar a los usuarios que necesitan crear informes ad hoc o crear informes y analizar los datos en sistemas de BI, sin la ayuda de un administrador de bases de datos (DBA) o desarrollador de datos.

Considere el uso de un almacenamiento de datos cuando necesite separar los datos históricos de los sistemas transaccionales de origen por motivos de rendimiento. Los almacenamientos de datos facilitan el acceso a los datos históricos desde varias ubicaciones, proporcionando una ubicación centralizada con formatos comunes, claves comunes, modelos de datos comunes y métodos de acceso comunes.

Los almacenamientos de datos están optimizados para el acceso de lectura, lo que produce una generación de informes más rápida en comparación con la ejecución de informes en el sistema transaccional de origen. Además, los almacenamientos de datos proporcionan las siguientes ventajas:

* Es posible almacenar y acceder a todos los datos históricos procedentes de varios orígenes desde un almacenamiento de datos como origen único.
* Puede mejorar la calidad de datos al limpiar estos conforme se importan en el almacenamiento de datos, obteniendo datos más precisos, además de proporcionar descripciones y códigos coherentes.
* Las herramientas de informes no entran en conflicto con los sistemas transaccionales de origen debido a los ciclos de procesamiento de las consultas. Un almacenamiento de datos permite que el sistema transaccional se centre principalmente en el control de las escrituras, mientras que el almacenamiento de datos atiende la mayoría de las solicitudes de lectura.
* Un almacenamiento de datos puede ayudar a consolidar los datos procedentes de software diferentes.
* Las herramientas de minería de datos pueden ayudarle a identificar patrones ocultos mediante metodologías automáticas sobre los datos del almacenamiento.
* Un almacenamiento de datos hace que sea más fácil proporcionar acceso seguro a los usuarios autorizados y restringir el acceso a otros usuarios. No es necesario conceder a los usuarios empresariales acceso a los datos de origen, lo que elimina un vector de ataque potencial contra uno o varios sistemas transaccionales de producción.
* Los almacenamientos de datos hacen que sea más fácil crear soluciones de inteligencia empresarial sobre los datos, como los [cubos OLAP](online-analytical-processing.md).

## <a name="challenges"></a>Desafíos

Para configurar correctamente un almacenamiento de datos para satisfacer las necesidades de su empresa puede encontrarse algunos de los siguientes desafíos:

* Confirmar el tiempo necesario para modelar correctamente los conceptos del negocio. Se trata de un paso importante, ya que los almacenamientos de datos son controlados por la información y la asignación de conceptos dirige el resto del proyecto. Esto implica la normalización de los términos relacionados con el negocio y el uso de formatos comunes (por ejemplo, la moneda y las fechas) y la reestructuración del esquema de forma que tenga sentido para los usuarios empresariales y al tiempo garantice la precisión de los datos agregados y las relaciones.
* Planificación y configuración de la orquestación de datos. Incluye cómo copiar los datos desde el sistema transaccional de origen al almacenamiento de datos y cuándo se deben mover los datos históricos desde los almacenes de datos operativos al almacenamiento.
* Mantener o mejorar la calidad de los datos mediante la limpieza de los mismos en el momento de su importación en el almacenamiento.

## <a name="data-warehousing-in-azure"></a>Almacenamiento de datos en Azure

En Azure, puede tener uno o varios orígenes de datos, ya sea desde las transacciones de los clientes o desde diversas aplicaciones de negocio usadas por varios departamentos. Estos datos se almacenan normalmente en una o varias bases de datos [OLTP](online-transaction-processing.md). Los datos podrían conservarse en otros medios de almacenamiento, como recursos compartidos de red, blobs de Azure Storage o una instancia de Data Lake. Los datos también se pueden almacenar en el propio almacenamiento de datos o en una base de datos relacional como Azure SQL Database. El propósito de la capa de almacenamiento de datos analíticos es satisfacer las consultas emitidas por las herramientas de generación de informes y análisis sobre el almacenamiento de datos o el data mart. En Azure, esta funcionalidad de almacenamiento analítico se puede lograr con Azure SQL Data Warehouse o con Azure HDInsight con Hive o Interactive Query. Además, necesitará cierto nivel de orquestación para mover o copiar los datos del almacén de datos al almacenamiento de datos periódicamente, algo que se puede llevar a cabo mediante Azure Data Factory o Oozie en Azure HDInsight.

Servicios relacionados:

* [Azure SQL Database](/azure/sql-database/)
* [SQL Server en una máquina virtual](/sql/sql-server/sql-server-technical-documentation)
* [Almacenamiento de datos de Azure](/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is)
* [Apache Hive en HDInsight](/azure/hdinsight/hadoop/hdinsight-use-hive)
* [Interactive Query (Hive LLAP) en HDInsight](/azure/hdinsight/interactive-query/apache-interactive-query-get-started)


## <a name="technology-choices"></a>Opciones de tecnología

- [Almacenamientos de datos](../technology-choices/data-warehouses.md)
- [Orquestación de canalizaciones](../technology-choices/pipeline-orchestration-data-movement.md)

