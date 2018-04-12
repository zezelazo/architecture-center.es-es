---
title: Elección de una tecnología de almacenamiento de datos
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: b14611a2dc34bcb145cf420441795d4124e7baeb
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2018
---
# <a name="choosing-a-big-data-storage-technology-in-azure"></a>Elección de una tecnología de almacenamiento de macrodatos en Azure

En este tema se comparan opciones de almacenamiento de datos para soluciones de macrodatos &mdash; en concreto, el almacenamiento de datos para la ingesta de datos de forma masiva y el procesamiento por lotes, en lugar de [almacenes de datos analíticos](./analytical-data-stores.md) o la [ingesta de streaming en tiempo real](./real-time-ingestion.md).

## <a name="what-are-your-options-when-choosing-data-storage-in-azure"></a>¿Cuáles son las opciones al elegir el almacenamiento de datos en Azure?

Existen varias opciones para la ingesta de datos en Azure en función de sus necesidades:

**Almacenamiento de archivos**

- [Blobs de Azure Storage](/azure/storage/blobs/storage-blobs-introduction)
- [Azure Data Lake Store](/azure/data-lake-store/)

**Bases de datos no SQL**

- [Azure Cosmos DB](/azure/cosmos-db/)
- [HBase en HDInsight](http://hbase.apache.org/)

## <a name="azure-storage-blobs"></a>Blobs de Azure Storage

Azure Storage es un servicio de almacenamiento administrado altamente disponible, seguro, duradero, escalable y redundante. Microsoft se encarga del mantenimiento y soluciona automáticamente los problemas críticos. Azure Storage es la solución de almacenamiento más ubicua que proporciona Azure, debido al número de servicios y herramientas que se pueden usar.

Hay diversos servicios de Azure Storage que puede usar para almacenar datos. La opción más flexible para almacenar blobs de varios orígenes de datos es [Blob Storage](/azure/storage/blobs/storage-blobs-introduction). Los blobs son básicamente archivos. Almacenan imágenes, documentos, archivos HTML, discos duros virtuales (VHD), macrodatos, como registros, copias de seguridad de bases de datos &mdash; cualquier cosa. Los blobs se almacenan en contenedores, que son similares a las carpetas. Un contenedor proporciona una agrupación de un conjunto de blobs. Una cuenta de almacenamiento puede contener un número ilimitado de contenedores y un contenedor puede almacenar un número ilimitado de blobs.

Azure Storage es una buena elección para soluciones de macrodatos y análisis debido a su flexibilidad, alta disponibilidad y bajo costo. Proporciona niveles de almacenamiento de acceso frecuente, esporádico y de archivo para distintos casos de uso. Para más información, consulte [Azure Blob Storage: niveles de almacenamiento de acceso frecuente, esporádico y de archivo](/azure/storage/blobs/storage-blob-storage-tiers).

Se puede acceder a Azure Blob Storage desde Hadoop (disponible a través de HDInsight). HDInsight puede usar un contenedor de blobs en Azure Storage como el sistema de archivos predeterminado para el clúster. Mediante una interfaz del sistema de archivos distribuido de Hadoop (HDFS) proporcionada por un controlador WASB, el conjunto completo de componentes de HDInsight puede operar directamente en datos estructurados o no estructurados almacenados como blobs. También es posible acceder a Azure Blob Storage a través de Azure SQL Data Warehouse con su característica PolyBase.

Otras características que hacen de Azure Storage una buena opción de almacenamiento son:

- [Varias estrategias de simultaneidad](/azure/storage/common/storage-concurrency?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).
- [Opciones de recuperación ante desastres y alta disponibilidad ](/azure/storage/common/storage-disaster-recovery-guidance?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).
- [Cifrado en reposo](/azure/storage/common/storage-service-encryption?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).
- [Control de acceso basado en roles (RBAC)](/azure/storage/common/storage-security-guide?toc=%2fazure%2fstorage%2fblobs%2ftoc.json#management-plane-security) para controlar el acceso mediante grupos y usuarios de Azure Active Directory.

## <a name="azure-data-lake-store"></a>Almacén de Azure Data Lake

[Azure Data Lake Store](/azure/data-lake-store/) es un repositorio de gran escala en toda la empresa para cargas de trabajo de análisis de macrodatos. Data Lake permite capturar datos de cualquier tamaño, tipo y velocidad de ingesta en una única ubicación [segura](/azure/data-lake-store/data-lake-store-overview#DataLakeStoreSecurity) para realizar análisis exploratorios y operativos.

Data Lake Store no impone ningún límite al tamaño de cuenta, el tamaño de archivo o la cantidad de datos que se pueden almacenar en una instancia de Data Lake. Los datos se almacenan de forma duradera mediante la realización de varias copias y no hay ningún límite al período de tiempo durante el que se pueden almacenar los datos en la instancia de Data Lake. Además de realizar varias copias de los archivos para protegerse contra errores inesperados, Data Lake propaga partes de un archivo entre varios servidores de almacenamiento individuales. Esto mejora el rendimiento de lectura cuando se lee el archivo en paralelo para realizar análisis de datos.

Se puede acceder a Data Lake Store desde Hadoop (disponible a través de HDInsight) mediante las API de REST compatibles con WebHDFS. Puede utilizar esto como una alternativa a Azure Storage cuando los tamaños de archivo individuales o combinados superan lo que admite Azure Storage. Sin embargo, hay [directrices de ajuste del rendimiento](/azure/data-lake-store/data-lake-store-performance-tuning-guidance#optimizing-io-intensive-jobs-on-hadoop-and-spark-workloads-on-hdinsight) que se deben seguir cuando se usa Data Lake Store como el almacenamiento principal para un clúster de HDInsight, con instrucciones específicas para [Spark](/azure/data-lake-store/data-lake-store-performance-tuning-spark), [Hive](/azure/data-lake-store/data-lake-store-performance-tuning-hive), [MapReduce](/azure/data-lake-store/data-lake-store-performance-tuning-mapreduce) y [Storm](/azure/data-lake-store/data-lake-store-performance-tuning-storm). Además, no olvide comprobar la [disponibilidad regional](https://azure.microsoft.com/regions/#services) de Data Lake Store, ya que no está disponible en tantas regiones como Azure Storage y debe estar ubicado en la misma región que el clúster de HDInsight.

Junto con Azure Data Lake Analytics, Data Lake Store está diseñado específicamente para habilitar el análisis de los datos almacenados y está optimizado para el rendimiento en escenarios de análisis de datos. También es posible acceder a Data Lake Store a través de Azure SQL Data Warehouse con su característica PolyBase.

## <a name="azure-cosmos-db"></a>Azure Cosmos DB

[Azure Cosmos DB](/azure/cosmos-db/) es la base de datos multimodelo de distribución global de Microsoft. Cosmos DB garantiza una latencia inferior a 10 milisegundos el 99 % del tiempo en cualquier parte del mundo, ofrece varios modelos de coherencia bien definidos para ajustar el rendimiento y garantiza alta disponibilidad con funcionalidades multi-homing.

Azure Cosmos DB es independiente del esquema. Indexa todos los datos automáticamente sin que haya que ocuparse de la administración de esquemas ni de índices. Es también multimodelo, admite de forma nativa modelos de datos de documentos, pares clave-valor, grafos y en columnas. 

Características de Azure Cosmos DB:

- [Replicación geográfica](/azure/cosmos-db/distribute-data-globally)
- [Escalado elástico del rendimiento y almacenamiento](/azure/cosmos-db/partition-data) en todo el mundo
- [Cinco niveles de coherencia bien definidos](/azure/cosmos-db/consistency-levels)

## <a name="hbase-on-hdinsight"></a>HBase en HDInsight

[Apache HBase](http://hbase.apache.org/) es una base de datos no SQL de código abierto construida sobre Hadoop y modelada según Google BigTable. HBase proporciona acceso aleatorio y alta coherencia para grandes cantidades de datos no estructurados y semiestructurados en una base de datos sin esquemas organizada por familias de columnas.

Los datos se almacenan en las filas de una tabla, mientras que los datos de una fila se agrupan por familia de columnas. HBase no tiene esquema en el sentido de que no es preciso que ni las columnas ni el tipo de datos almacenados en ellas se definan antes de usarlos. El código abierto se escala linealmente para controlar petabytes de datos en miles de nodos. Puede basarse en la redundancia de datos, el procesamiento por lotes y otras características proporcionadas por aplicaciones distribuidas en el ecosistema Hadoop.

La [implementación de HDInsight](/azure/hdinsight/hbase/apache-hbase-overview) aprovecha la arquitectura de escalabilidad horizontal de HBase para proporcionar particionamiento automático de tablas, alta coherencia para lecturas y escrituras y conmutación por error automática. Se mejora el rendimiento mediante el almacenamiento en caché de memoria para lecturas y streaming de alto rendimiento para escrituras. En la mayoría de los casos, deseará [crear el clúster de HBase dentro de una red virtual](/azure/hdinsight/hbase/apache-hbase-provision-vnet) para que otros clústeres y aplicaciones de HDInsight puedan tener acceso directo a las tablas.

## <a name="key-selection-criteria"></a>Principales criterios de selección

Para restringir las opciones, empiece por responder a estas preguntas:

- ¿Necesita almacenamiento administrado de alta velocidad basado en la nube para cualquier tipo de datos de texto o binarios? En caso afirmativo, seleccione una de las opciones de almacenamiento de archivos.

- ¿Necesita almacenamiento de archivos optimizado para cargas de trabajo de análisis en paralelo y alto IOPS y rendimiento? En caso afirmativo, elija una opción que se ajuste al rendimiento de cargas de trabajo de análisis.

- ¿Necesita almacenar datos no estructurados o semiestructurados en una base de datos sin esquema? Si es así, seleccione una de las opciones no relacionales. Compare las opciones de indexación y modelos de base de datos. Según el tipo de datos que necesite almacenar, los modelos de la base de datos principal pueden ser el factor más relevante.

- ¿Puede utilizar el servicio en su región? Compruebe la disponibilidad regional para cada servicio de Azure. Consulte los [Productos disponibles por región](https://azure.microsoft.com/regions/services/).

## <a name="capability-matrix"></a>Matriz de funcionalidades

En las tablas siguientes se resumen las diferencias clave en cuanto a funcionalidades.

### <a name="file-storage-capabilities"></a>Funcionalidades de almacenamiento de archivos

|  | Almacén de Azure Data Lake | Contenedores de Azure Blob Storage |
| --- | --- | --- |
| Propósito | Almacenamiento optimizado para cargas de trabajo de análisis de macrodatos |Almacén general de objetos para una amplia variedad de escenarios de almacenamiento |
| Casos de uso | Datos en lotes, de análisis de streaming y de aprendizaje automático, como archivos de registro, datos de IoT, transmisiones de clic y conjuntos de datos grandes | Cualquier tipo de texto o datos binarios, por ejemplo, back-end de aplicación, datos de copia de seguridad, almacenamiento multimedia para streaming y datos de propósito general |
| sección Estructura | Sistema de archivos jerárquico | Almacén de objetos con el espacio de nombres plano |
| Autenticación | Basado en las [identidades de Azure Active Directory](/azure/active-directory/active-directory-authentication-scenarios) | Basado en secretos compartidos: [Claves de acceso de cuenta](/azure/storage/common/storage-create-storage-account#manage-your-storage-account), [Claves de firma de acceso compartido](/azure/storage/common/storage-dotnet-shared-access-signature-part-1) y [Control de acceso basado en roles (RBAC)](/azure/security/security-storage-overview) |
| Protocolo de autenticación | OAuth 2.0. Las llamadas deben contener un JWT válido (token web JSON) emitido por Azure Active Directory | Código de autenticación de mensajes basado en hash (HMAC). Las llamadas deben contener un hash SHA-256 codificado en Base64 en una parte de la solicitud HTTP. |
| Autorización | Listas de control de acceso (ACL) de POSIX. Las ACL basadas en identidades de Azure Active Directory se pueden establecer en el nivel de archivo y el de carpeta. | Para la autorización en el nivel de cuenta utilice [claves de acceso de cuenta](/azure/storage/common/storage-create-storage-account#manage-your-storage-account). Para la autorización de cuenta, contenedor o blob utilice las [claves de firma de acceso compartido](/azure/storage/common/storage-dotnet-shared-access-signature-part-1). |
| Auditoría | Disponible.  |Disponible |
| Cifrado en reposo | Transparente, en el servidor | Transparente, en el servidor; cifrado en el cliente |
| SDK para desarrolladores | .NET, Java, Python, Node.js | .Net, Java, Python, Node.js, C++, Ruby |
| Rendimiento de cargas de trabajo de análisis | Rendimiento optimizado para cargas de trabajo de análisis en paralelo, alto rendimiento e IOPS | No está optimizado para cargas de trabajo de análisis |
| Límites de tamaño | Sin límites para el tamaño de cuenta, de archivo o el número de archivos | Los límites específicos se documentan [aquí](/azure/azure-subscription-service-limits#storage-limits) |
| Redundancia geográfica | Con redundancia local (varias copias de datos en una región de Azure) | Con redundancia local (LRS) o global (GRS) y acceso de lectura redundante global (RA-GRS). Más información [aquí](/azure/storage/common/storage-redundancy) |

### <a name="nosql-database-capabilities"></a>Funcionalidades de bases de datos no SQL

|                                    |                                           Azure Cosmos DB                                           |                                                             HBase en HDInsight                                                             |
|------------------------------------|-----------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------|
|       Modelo de la base de datos principal       |                      Almacenamiento de documentos, grafos, almacenamiento de clave-valor, almacenamiento de columnas anchas                      |                                                             Almacenamiento de columnas anchas                                                              |
|         Índices secundarios          |                                                 Sí                                                 |                                                                     Sin                                                                      |
|        Compatibilidad con lenguaje SQL        |                                                 Sí                                                 |                                     Sí (mediante el controlador JDBC [Phoenix](http://phoenix.apache.org/))                                      |
|            Coherencia             |                   Alta, de obsolescencia limitada, sesión, prefijo coherente, eventual                   |                                                                   Alta                                                                   |
| Integración nativa con Azure Functions |                        [Sí](/azure/cosmos-db/serverless-computing-database)                        |                                                                     Sin                                                                      |
|   Distribución global automática    |                          [Sí](/azure/cosmos-db/distribute-data-globally)                           | No es posible [configurar la replicación de clúster de HBase](/azure/hdinsight/hbase/apache-hbase-replication) entre regiones con coherencia eventual |
|           Modelo de precios            | Las unidades de solicitud (RU) escalables de manera flexible se cobran por segundo según sea necesario, almacenamiento escalable de forma elástica |                              Precios del clúster de HDInsight (escalado horizontal de nodos) por minuto, almacenamiento                               |

