---
title: Procesamiento de archivos CSV y JSON
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 02e684d562cfe555f9e3596ad0a2f1a00d05c7a7
ms.sourcegitcommit: 51f49026ec46af0860de55f6c082490e46792794
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/03/2018
ms.locfileid: "30298613"
---
# <a name="working-with-csv-and-json-files-for-data-solutions"></a>Trabajo con archivos CSV y JSON en soluciones de datos

CSV y JSON son probablemente los formatos más utilizados para la ingesta, intercambio y almacenamiento de datos no estructurados o semiestructurados. 

## <a name="about-csv-format"></a>Acerca del formato CSV

Los archivos CSV (valores separados por comas) se usan normalmente para intercambiar datos tabulares entre sistemas en texto sin formato. Normalmente contienen una fila de encabezado que proporciona los nombres de columna para los datos, pero estos se consideran semiestructurados. Esto es debido a que los archivos CSV no pueden representar de forma natural datos jerárquicos o relacionales. Las relaciones de datos normalmente se controlan con varios archivos CSV, donde las claves externas se almacenan en columnas de uno o más archivos, pero las relaciones entre esos archivos no se expresan con el propio formato. Los archivos en formato CSV pueden usar otros delimitadores además de las comas, por ejemplo, tabulaciones o espacios.

A pesar de sus limitaciones, los archivos CSV son una opción popular para el intercambio de datos, ya que son compatibles con una amplia gama de aplicaciones de negocio, de consumo y científicas. Por ejemplo, los programas de base de datos y hojas de cálculo pueden importar y exportar archivos CSV. Del mismo modo, la mayoría de los motores de procesamiento por lotes y de procesamiento de secuencias de datos, como Spark y Hadoop, admiten de forma nativa serializar y deserializar archivos con formato CSV y ofrecen formas de aplicar un esquema en la lectura. Esto hace más fácil trabajar con los datos, ya que ofrece opciones para realizar consultas en ellos y almacenar la información en un formato de datos más eficaz para un procesamiento más rápido.

## <a name="about-json-format"></a>Acerca del formato JSON

Los datos JSON (notación de objetos JavaScript) se representan como pares de clave y valor en un formato semiestructurado. JSON se compara a menudo con XML, ya que ambos son capaces de almacenar los datos en un formato jerárquico con los datos secundarios representados alineados con su elemento primario. Ambos son autodescriptivos y legibles, pero los documentos JSON tienden a ser mucho más pequeños, lo que ha llevado a su uso popular en el intercambio de datos en línea, sobre todo con la llegada de los servicios web basados en REST. 

Los archivos con formato JSON tienen varias ventajas sobre CSV:

* JSON mantiene las estructuras jerárquicas, facilitando la tarea de almacenar datos relacionados en un único documento y representar relaciones complejas.
* La mayoría de los lenguajes de programación proporcionan compatibilidad nativa para deserializar JSON en objetos o proporcionan bibliotecas de serialización de JSON ligeras.
* JSON admite listas de objetos, lo que ayuda a evitar traducciones desordenadas de listas en un modelo de datos relacional.
* JSON es un formato de archivo utilizado habitualmente en bases de datos no SQL, como MongoDB, Couchbase y Azure Cosmos DB.

Dado que una gran cantidad de datos que llegan a través de la línea ya está en formato JSON, la mayoría de los lenguajes de programación basados en la web admiten trabajar con JSON de forma nativa o mediante el uso de bibliotecas externas para serializar y deserializar los datos JSON. Esta compatibilidad universal con JSON ha llevado a su uso en formatos lógicos a través de la representación de la estructura de datos, formatos de intercambio de datos activos y el almacenamiento de datos para los datos inactivos.

Muchos motores de procesamiento por lotes y de procesamiento de datos de secuencias admiten de forma nativa la serialización y deserialización de JSON. Aunque los datos contenidos en documentos JSON en última instancia pueden almacenarse en formatos más optimizado para el rendimiento, como Parquet o Avro, actúa como los datos sin procesar para el origen único, lo cual resulta esencial para volver a procesar los datos según sea necesario.

## <a name="when-to-use-csv-or-json-formats"></a>Cuándo utilizar los formatos CSV o JSON

Los archivos CSV se utilizan normalmente para la exportación y la importación de datos o para su procesamiento para el análisis o el aprendizaje automático. Los archivos con formato JSON tienen las mismas ventajas, pero son más comunes en soluciones de intercambio de datos activos. Los documentos JSON son enviados a menudo por web y por dispositivos móviles para realizar transacciones en línea, por los dispositivos de IoT (internet de las cosas) para comunicaciones unidireccionales o bidireccionales o por aplicaciones de cliente comunicándose con servicios de SaaS y PaaS o arquitecturas sin servidor. 

Los formatos de archivo CSV y JSON facilitan el intercambio de datos entre diferentes sistemas o dispositivos. Sus formatos semiestructurados aportan flexibilidad para transferir casi cualquier tipo de datos y la compatibilidad universal con estos formatos hace sencillo trabajar con ellos. Ambos se pueden utilizar como origen único sin tratamiento en aquellos casos en los que los datos procesados se almacenan en formatos binarios para una consulta más eficaz. 

## <a name="working-with-csv-and-json-data-in-azure"></a>Trabajo con datos CSV y JSON en Azure

Azure proporciona varias soluciones para trabajar con archivos CSV y JSON, dependiendo de sus necesidades. El sitio de aterrizaje principal de estos archivos es Azure Storage o Azure Data Lake Store. La mayoría de los servicios de Azure que trabajan con estos y otros archivos basados en texto se integran con estos servicios de almacenamiento de objetos. En algunas situaciones, sin embargo, puede elegir importar directamente los datos en Azure SQL u otro almacén de datos. SQL Server ofrece compatibilidad nativa para almacenar y trabajar con documentos JSON, lo que facilita [importar y procesar esos tipos de archivos](/sql/relational-databases/json/import-json-documents-into-sql-server). Puede usar una herramienta como la importación en bloque de SQL para [importar archivos CSV](/sql/relational-databases/json/import-json-documents-into-sql-server) fácilmente.

Según el escenario, puede realizar [procesamiento por lotes](../big-data/batch-processing.md) o [procesamiento en tiempo real](../big-data/real-time-processing.md) de los datos.

## <a name="challenges"></a>Desafíos

Hay algunos desafíos a tener en cuenta al trabajar con estos formatos:

* Sin restricciones en el modelo de datos, los archivos CSV y JSON son proclives a daños en los datos ("entrada y salida de elementos no utilizados"). Por ejemplo, no hay noción de un objeto de tipo fecha y hora en el archivo, por lo que el formato de archivo no impide la inserción de "ABC123" en un campo de fecha, por ejemplo.

* El uso de archivos CSV y JSON como solución de almacenamiento esporádico no escala adecuadamente cuando se trabaja con macrodatos. En la mayoría de los casos no se pueden dividir en particiones para su procesamiento en paralelo y no se pueden comprimir como los formatos binarios. Esto conduce a menudo a procesar y almacenar estos datos en formatos optimizados para la lectura como Parquet y ORC (columnas de filas optimizadas), que también proporcionan índices y estadísticas en línea acerca de los datos contenidos.

* Necesitará aplicar un esquema en los datos semiestructurados para que resulten más fáciles de consultar y analizar. Normalmente, esto requiere almacenar los datos en otro formato que cumpla con las necesidades de almacenamiento de datos de su entorno, como una base de datos.

