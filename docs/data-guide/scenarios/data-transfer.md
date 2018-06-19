---
title: Elección de una tecnología de transferencia de datos
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 53dcf8a69ad8ae100dbdbb230a9280efd419342a
ms.sourcegitcommit: 85334ab0ccb072dac80de78aa82bcfa0f0044d3f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2018
ms.locfileid: "35252760"
---
# <a name="transferring-data-to-and-from-azure"></a>Transferencia de datos a Azure y desde este

Existen varias opciones para transferir datos a Azure, y desde este, en función de las necesidades de cada uno.

## <a name="physical-transfer"></a>Transferencia física

El uso de hardware físico para transferir datos a Azure es una opción recomendable cuando:

- La red funciona con lentitud o es poco confiable.
- El costo de obtener más ancho de banda de red es prohibitivo.
- Las directivas de seguridad o de la organización no permiten las conexiones salientes cuando se trabaja con información confidencial. 

Si su principal preocupación es el tiempo que se va a tardar en transferir los datos, es posible que desee ejecutar una prueba para comprobar si la transferencia de red es realmente más lenta que el transporte físico.

Hay dos opciones principales para transportar físicamente los datos a Azure:
- **Azure Import/Export**. El [servicio Azure Import/Export](/azure/storage/common/storage-import-export-service) permite transferir de forma segura grandes cantidades de datos a Azure Blob Storage o Azure Files mediante el envío de unidades de disco duro o SSD a un centro de datos de Azure. También puede usar este servicio para transferir datos Azure Storage desde tardar hasta las unidades de disco duro y enviarlas al sitio local.

- **Azure Data Box**. [Azure Data Box](https://azure.microsoft.com/services/storage/databox/) es un dispositivo proporcionado por Microsoft que funciona de forma muy parecida al servicio Azure Import/Export. Microsoft envía un dispositivo de transferencia propietario, seguro y resistente a manipulaciones, y controla la logística de un extremo a otro, pero el usuario puede hacer un seguimiento de ella desde el portal. Una ventaja del servicio Azure Data Box es lo fácil que es usarlo. No es preciso adquirir varios discos duros, prepararlos y transferir archivos a todos y cada uno de ellos. Azure Data Box es compatible con varios asociados de Azure líderes de su sector, lo que facilita la tarea de transportar archivos sin conexión a la nube desde sus productos. 

## <a name="command-line-tools-and-apis"></a>Herramientas de línea de comandos y API

Tenga en cuenta estas opciones cuando desee que la transferencia de datos se realice mediante programación y mediante scripts.

- **CLI de Azure** La [CLI de Azure](/azure/hdinsight/hdinsight-upload-data#commandline) es una herramienta multiplataforma que permite administrar los servicios de Azure y cargar datos en Azure Storage. 

- **AzCopy**. Use AzCopy desde una línea de comandos de [Windows](/azure/storage/common/storage-use-azcopy?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) o [Linux](/azure/storage/common/storage-use-azcopy-linux?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) para copiar datos fácilmente tanto a Azure Blob, File y Table Storage como desde estos servicios con un rendimiento óptimo. AzCopy admite la simultaneidad y el paralelismo, y permite reanudar operaciones de copia cuando si se interrumpen. También es una de las opciones más rápidas. Para acceder mediante programación, la [Biblioteca de movimiento de datos de Microsoft Azure Storage](/azure/storage/common/storage-use-data-movement-library) es el marco principal que alimenta AzCopy. Se proporciona en forma de biblioteca de .NET Core. 

- **PowerShell**. El [`Start-AzureStorageBlobCopy`cmdlet de PowerShell](/powershell/module/azure.storage/start-azurestorageblobcopy?view=azurermps-5.0.0) es una opción para los administradores de Windows acostumbrados a PowerShell.  

- **AdlCopy**. [AdlCopy](/azure/data-lake-store/data-lake-store-copy-data-azure-storage-blob) permite copiar datos de Azure Storage Blobs a Data Lake Store. También se puede usar para copiar datos entre dos cuentas de Azure Data Lake Store. Sin embargo, no se puede utilizar para copiar datos de Data Lake Store a Storage Blob.

- **Distcp**. Si tiene un clúster de HDInsight con acceso a Data Lake Store, puede usar herramientas del ecosistema de Hadoop, como [Distcp](/azure/data-lake-store/data-lake-store-copy-data-wasb-distcp), tanto para copiar datos a un almacenamiento de clúster de HDInsight (WASB) como para copiarlos desde este a una cuenta de Data Lake Store.

- **Sqoop**. [Sqoop](/azure/hdinsight/hadoop/hdinsight-use-sqoop) es un proyecto de Apache que forma parte del ecosistema de Hadoop. Viene preinstalado en todos los clústeres de HDInsight. Permite la transferencia de datos entre un clúster de HDInsight y bases de datos relacionales, como SQL, Oracle, MySQL, etc. Sqoop es una colección de herramientas relacionadas, incluidas la importación y exportación. Sqoop funciona con clústeres de HDInsight mediante blobs de Azure Storage o almacenamiento adjunto de Data Lake Store.

- **PolyBase**. [PolyBase](/sql/relational-databases/polybase/get-started-with-polybase) es una tecnología que accede a datos que están fuera de la base de datos a través del lenguaje de T-SQL. En SQL Server 2016, permite ejecutar consultas de datos externos en Hadoop o importar o exportar datos desde Azure Blob Storage. En Azure SQL Data Warehouse, puede importar y exportar datos tanto desde Azure Blob Storage como desde Azure Data Lake Store. Actualmente, PolyBase es el método más rápido de importación de datos en SQL Data Warehouse.

- **Línea de comandos de Hadoop**. Si tiene datos que residan en un nodo principal del clúster de HDInsight, puede usar el comando `hadoop -copyFromLocal` para copiarlos en el almacenamiento adjunto de su clúster, como Azure Storage Blob o Azure Data Lake Store. Para usar el comando de Hadoop, primero es preciso conectarse al nodo principal. Una vez conectado, puede cargar un archivo en el almacenamiento.

## <a name="graphical-interface"></a>Interfaz gráfica

Si va a transferir solo unos pocos archivos u objetos de datos y no necesita automatizar el proceso, tenga en cuenta las siguientes opciones.

- **Explorador de Azure Storage**. [Explorador de Azure Storage](https://azure.microsoft.com/features/storage-explorer/) es una herramienta multiplataforma que permite administrar el contenido de las cuentas de Azure Storage. Permite cargar, descargar y administrar blogs, archivos, colas, tablas y entidades de Azure Cosmos DB. Utilícelo con Blob Storage para administrar blobs y carpetas, así como para cargar y descargar blobs entre el sistema de archivos local y Blob Storage, o entre cuentas de almacenamiento.

- **Azure Portal**. Tanto Blob Storage como Data Lake Store proporcionan una interfaz basada en web para explorar archivos y cargar nuevos archivos de uno en uno. Es una buena opción si no desea instalar herramientas ni generar comandos para explorar rápidamente los archivos, o simplemente cargar archivos nuevos.

## <a name="data-pipeline"></a>Canalización de datos

**Azure Data Factory**. [Azure Data Factory](/azure/data-factory/) es un servicio administrado muy apropiado para transferir archivos con regularidad entre varios servicios de Azure, de forma local o una combinación de ambas posibilidades. Con Azure Data Factory puede crear y programar flujos de trabajo basados en datos (llamados canalizaciones) que ingieren datos de distintos almacenes de datos. Los datos se pueden procesar y transformar mediante servicios de proceso, como Azure HDInsight Hadoop, Spark, Azure Data Lake Analytics y Azure Machine Learning. Cree flujos de trabajo controlados por datos para [orquestar](../technology-choices/pipeline-orchestration-data-movement.md) y automatizar tanto el movimiento de datos como la transformación de datos.

## <a name="key-selection-criteria"></a>Principales criterios de selección

En los escenarios de transferencia de datos, elija el sistema que más se ajuste a sus necesidades, para lo que debe responder estas preguntas:

- ¿Necesita transferir grandes cantidades de datos y hacerlo a través de una conexión a Internet tardaría demasiado tiempo, sería poco confiables o demasiado caro? Si es así, considere la posibilidad de realizar transferencias físicas.

- ¿Prefiere realizar las tareas de transferencia de datos mediante scripts para que se puedan volver a utilizar? Si es así, seleccione una de las opciones de línea de comandos o Azure Data Factory.

- ¿Necesita transferir una gran cantidad de datos a través de una conexión de red? En ese caso, seleccione una opción que esté optimizada para macrodatos.

- ¿Necesita transferir datos a una base de datos relacional o desde ella? En caso afirmativo, elija una opción que admita una o varias bases de datos relacionales. Tenga en cuenta que algunas de estas opciones también requieren un clúster de Hadoop.

- ¿Necesita una canalización de datos o una orquestación de flujos de trabajo automatizadas? Si es así, considere la posibilidad de usar Azure Data Factory.

## <a name="capability-matrix"></a>Matriz de funcionalidades

En las tablas siguientes se resumen las diferencias clave en cuanto a funcionalidades.

### <a name="physical-transfer"></a>Transferencia física

| | Servicio Azure Import/Export | Azure Data Box |
| --- | --- | --- |
| Factor de forma | Unidades de disco duro o SSD SATA internas | Dispositivo de hardware individual seguro y a prueba de alteraciones |
| Microsoft administra la logística de envío | Sin  | Sí |
| Se integra con productos de asociados | Sin  | Sí |
| Dispositivo personalizado | Sin  | Sí |

### <a name="command-line-tools"></a>Herramientas de línea de comandos

**Hadoop/HDInsight**

| | Distcp | Sqoop | CLI de Hadoop |
| --- | --- | --- | --- |
| Optimizado para macrodatos | Sí | Sí |  Sí |
| Copiar a base de datos relacional |  Sin  | Sí | Sin  |
| Copiar de base de datos relacional |  Sin  | Sí | Sin  |
| Copiar a Blob Storage |  Sí | Sí | Sí |
| Copiar de Blob Storage | Sí |  Sí | Sin  |
| Copiar a Data Lake Store | Sí | Sí | Sí |
| Copiar de Data Lake Store | Sí | Sí | Sin  |

**Otros**

| | Azure CLI | AzCopy | PowerShell | AdlCopy | PolyBase |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Plataformas compatibles | Linux, OS X y Windows | Linux y Windows | Windows | Linux, OS X y Windows | SQL Server y Azure SQL Data Warehouse | 
| Optimizado para macrodatos | Sin  | Sin  | Sin  | Sí <sup>1</sup> | Sí <sup>2</sup> |
| Copiar a base de datos relacional | Sin  | Sin  | Sin  | Sin  | Sí | 
| Copiar de base de datos relacional | Sin  | Sin  | Sin  | Sin  | Sí | 
| Copiar a Blob Storage | Sí | Sí | Sí | Sin  | Sí | 
| Copiar de Blob Storage | Sí | Sí | Sí | Sí | Sí |
| Copiar a Data Lake Store | Sin  | Sin  | Sí | Sí |  Sí | 
| Copiar de Data Lake Store | Sin  | Sin  | Sí | Sí | Sí | 


[1] AdlCopy está optimizado para la transferencia de macrodatos cuando se utiliza con una cuenta de Data Lake Analytics.

[2] El [rendimiento de PolyBase se puede aumentar](/sql/relational-databases/polybase/polybase-guide#performance) mediante la inserción de cálculo en Hadoop y el uso de [grupos de escalado horizontal de PolyBase](/sql/relational-databases/polybase/polybase-scale-out-groups) para habilitar la transferencia de datos paralela entre instancias de SQL Server y nodos de Hadoop.

### <a name="graphical-interface-and-azure-data-factory"></a>Interfaz gráfica y Azure Data Factory

| | Explorador de Azure Storage | Azure Portal* | Azure Data Factory |
| --- | --- | --- | --- |
| Optimizado para macrodatos | Sin  | Sin  | Sí | 
| Copiar a base de datos relacional | Sin  | Sin  | Sí |
| Copiar a base de datos relacional | Sin  | Sin  | Sí |
| Copiar a Blob Storage | Sí | Sin  | Sí |
| Copiar de Blob Storage | Sí | Sin  | Sí |
| Copiar a Data Lake Store | Sin  | Sin  | Sí |
| Copiar de Data Lake Store | Sin  | Sin  | Sí |
| Cargar en Blob Storage | Sí | Sí | Sí |
| Cargar en Data Lake Store | Sí | Sí | Sí |
| Orquestar las transferencias de datos | Sin  | Sin  | Sí |
| Personalizar las transformaciones de datos | Sin  | Sin  | Sí |
| Modelo de precios | Gratuito | Gratuito | Pago por uso |

\* En este caso, Azure Portal significa usar las herramientas de exploración basada en web para Blob Storage y Data Lake Store.

