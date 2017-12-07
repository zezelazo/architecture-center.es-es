---
title: "Elección del almacén de datos apropiado"
description: "Introducción sobre la elección de almacenes de datos de Azure"
author: MikeWasson
ms.openlocfilehash: 3a5780c4a2dbd8a41e9c7bfa7f68d8a7916a7374
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2017
---
# <a name="choose-the-right-data-store"></a>Elección del almacén de datos apropiado

Los sistemas empresariales actuales administran volúmenes de datos cada vez mayores. Los datos se pueden ingerir a partir de servicios externos, puede que el propio sistema los genere o los pueden crear los usuarios. Estos conjuntos de datos pueden tener características y requisitos de procesamiento completamente diferentes. Las empresas usan los datos para evaluar tendencias, desencadenar procesos empresariales, auditar las operaciones, analizar el comportamiento de los clientes y muchas otras cosas. 

Esta heterogeneidad implica que un solo almacén de datos no suele ser la mejor opción. En lugar de eso, a menudo es mejor almacenar distintos tipos de datos en diferentes almacenes de datos, cada uno centrado en un patrón concreto de carga de trabajo o uso. El término *persistencia de Polyglot* se usa para describir soluciones que emplean una combinación de tecnologías de almacenamiento de datos.

Seleccionar el almacén de datos adecuado para sus requisitos es una decisión de diseño clave. Existen literalmente cientos de implementaciones a elegir entre bases de datos SQL y NoSQL. Los almacenes de datos a menudo se clasifican según la forma de estructurar los datos y según los tipos de operaciones que admiten. Este artículo describe algunos de los modelos de almacenamiento más comunes. Tenga en cuenta que una tecnología de almacén de datos determinada puede admitir varios modelos de almacenamiento. Por ejemplo, un sistema de administración de bases de datos relacionales (RDBMS) también puede admitir el almacenamiento de clave-valor o de gráficos. De hecho, hay una tendencia general hacia el denominado soporte *multimodelo* por el que un único sistema de base de datos es compatible con varios modelos. Pero aún así, es útil comprender los diferentes modelos en un nivel alto. 

No todos los almacenes de datos de una determinada categoría proporcionan el mismo conjunto de características. La mayoría de los almacenes de datos proporcionan funciones del lado servidor para consultar y procesar datos. A veces, estas funciones están integradas en el motor de almacenamiento de datos. En otros casos, se separan las funcionalidades de almacenamiento y procesamiento de datos y puede haber varias opciones para el procesamiento y el análisis. Los almacenes de datos también admiten distintas interfaces de programación y administración. 

Por lo general, lo primero que debe tener en cuenta es qué modelo de almacenamiento se ajusta mejor a sus requisitos. Posteriormente, debe considerar un almacén de datos determinado de esa categoría en función de factores como el conjunto de características, el costo y la facilidad de administración.

## <a name="relational-database-management-systems"></a>Sistemas de administración de bases de datos relacionales

Las bases de datos relacionales organizan los datos como una serie de tablas bidimensionales con filas y columnas. Cada tabla tiene sus propias columnas y cada fila de una tabla tiene el mismo conjunto de columnas. Este modelo se basa en cálculos matemáticos y la mayoría de los proveedores proporciona un dialecto del lenguaje de consulta estructurado (SQL) para recuperar y administrar datos. Normalmente, un sistema de administración de bases de datos relacionales implementa un mecanismo transaccionalmente coherente que se ajusta al modelo ACID (atomicidad, coherencia, aislamiento, durabilidad) para actualizar la información. 

Normalmente, un RDBMS admite un modelo de esquema basado en escritura, en la que la estructura de los datos se define por adelantado y todas las operaciones de lectura o escritura deben usar ese esquema. Esto lo hace diferente a la mayoría de almacenes de datos NoSQL, especialmente a los de tipo clave-valor, en los que el modelo de esquema basado en lectura da por supuesto que el cliente impondrá su propio esquema interpretativo sobre los datos que salen de la base de datos y no tiene en cuenta el formato en el que se van a escribir los datos.

Un RDBMS es muy útil cuando es importante contar con unas sólidas garantías de coherencia, en las que todos los cambios son atómicos y las transacciones siempre dejan los datos en un estado consistente. Sin embargo, las estructuras subyacentes no se prestan al escalado horizontal mediante la distribución del almacenamiento y el procesamiento en las máquinas. Además, la información almacenada en un RDBMS se debe situar en una estructura relacional siguiendo el proceso de normalización. Aunque este proceso se entiende fácilmente, puede conducir a ineficacias debido a la necesidad de desensamblar entidades lógicas en filas en tablas independientes y, a continuación, volver a ensamblar los datos al ejecutar consultas. 

Servicio de Azure correspondiente: 

- [Azure SQL Database][sql-db]
- [Azure Database for MySQL][mysql]
- [Azure Database para PostgreSQL][postgres]

## <a name="keyvalue-stores"></a>Almacenes clave-valor

Un almacén clave/valor es básicamente una tabla hash grande. Puede asociar cada valor de datos con una clave única y el almacén clave/valor usará esta clave para almacenar los datos mediante el uso de una función hash adecuada. La función hash se selecciona para proporcionar una distribución uniforme de claves hash en el almacenamiento de datos. 

La mayoría de los almacenes clave valor solo admiten operaciones simples de consulta, inserción y eliminación. Para modificar un valor (parcial o completamente), una aplicación debe sobrescribir los datos existentes para todo el valor. En la mayoría de las implementaciones, leer o escribir un valor único es una operación atómica. Si el valor es grande, la escritura puede tardar algún tiempo. 

Una aplicación puede almacenar datos arbitrarios como un conjunto de valores, aunque algunos almacenes clave/valor imponen límites sobre el tamaño máximo de los valores. Los valores almacenados son opacos para el software del sistema de almacenamiento. La aplicación debe ser la que proporcione e interprete toda información de esquema. Básicamente, los valores son blobs y el almacén clave valor solo recupera o almacena el valor por clave. 

![](./images/key-value.png)

Los almacenes clave/valor están altamente optimizados para las aplicaciones que realizan búsquedas simples, pero son menos adecuados para los sistemas que necesitan consultar datos entre diferentes almacenes clave/valor. Los almacenes clave/valor tampoco están optimizados para escenarios en los que la realización de consultas por valor es importante, en lugar de realizar búsquedas basándose solo en claves. Por ejemplo, con una base de datos relacional, puede encontrar un registro mediante el uso de una cláusula WHERE, pero los almacenes clave/valor no tienen normalmente este tipo de funcionalidad de búsqueda de valores.

Un único almacén clave/valor puede ser sumamente escalable, ya que el almacén de datos puede distribuir fácilmente los datos entre varios nodos de máquinas independientes. 

Servicios de Azure correspondientes: 

- [Cosmos DB][cosmosdb]
- [Azure Redis Cache][redis-cache]

## <a name="document-databases"></a>Bases de datos de documentos

Desde un punto de vista conceptual, una base de datos de documentos es similar a un almacén clave/valor, excepto en que almacena una colección de campos y datos con nombre (conocidos como documentos), en la que cada uno de ellos puede ser un elemento escalar sencillo o elementos compuestos como listas y colecciones secundarias. Los datos de los campos de un documento se pueden codificar de varias formas, incluidas XML, YAML, JSON, BSON, o incluso almacenarse como texto sin formato. A diferencia de los almacenes clave/valor, los campos de los documentos se exponen en el sistema de administración de almacenamiento, lo que permite que una aplicación pueda consultar y filtrar los datos mediante el uso de los valores de estos campos. 

Normalmente, un documento contiene todos los datos de una entidad. Los elementos que constituyen una entidad son específicos de la aplicación. Por ejemplo, una entidad puede contener los detalles de un cliente, un pedido o una combinación de ambos. Un solo documento puede contener información que se puede distribuir a través de varias tablas relacionales de un RDBMS. 

Un almacén de documentos no requiere que todos los documentos tengan la misma estructura. Este enfoque de forma libre proporciona gran flexibilidad. Las aplicaciones pueden almacenar datos diferentes en documentos a medida que cambian los requisitos empresariales.

![](./images/document.png)

La aplicación puede recuperar documentos mediante el uso de la clave del documento. Se trata de un identificador exclusivo del documento, al que a menudo se aplica un algoritmo hash para ayudar a distribuir los datos uniformemente. Algunas bases de datos de documentos crean automáticamente la clave del documento. Otras le permiten especificar un atributo del documento para usarlo como clave. La aplicación también puede consultar documentos según el valor de uno o más campos. Algunas bases de datos de documentos admiten la indexación para facilitar la búsqueda rápida de documentos basándose en uno o más campos indexados. 

Muchas bases de datos de documentos admiten actualizaciones en contexto, lo que permite que una aplicación pueda modificar los valores de campos específicos de un documento sin volver a escribir todo el documento. Las operaciones de lectura y escritura en varios campos de un solo documento son normalmente atómicas.

Servicio de Azure correspondiente: [Cosmos DB][cosmosdb]

## <a name="graph-databases"></a>Bases de datos de gráficos

Una base de datos de grafos almacena dos tipos de información: nodos y bordes. Puede pensar en los nodos como entidades. Los bordes especifican las relaciones entre los nodos. Los nodos y los bordes pueden tener propiedades que proporcionan información acerca de ese nodo o borde de forma parecida a las columnas de una tabla. Los bordes también pueden tener una dirección que indica la naturaleza de la relación.

El propósito de una base de datos de grafos es permitir a una aplicación realizar consultas que recorran la red de nodos y bordes de manera eficaz y analizar las relaciones entre entidades. El siguiente diagrama muestra una base de datos del personal de una organización estructurada como un grafo. Las entidades son los empleados y departamentos, y los bordes indican las relaciones jerárquicas y el departamento en el que trabajan los empleados. En este grafo, las flechas de los bordes muestran la dirección de las relaciones.
 
![](./images/graph.png)

Esta estructura hace que sea fácil realizar consultas como "buscar todos los empleados que dependen directa o indirectamente de Sarah" o "¿quién trabaja en el mismo departamento que John?". Para grafos de gran tamaño con una gran cantidad de entidades y relaciones, puede realizar análisis muy complejos de forma muy rápida. Muchas bases de datos de grafos proporcionan un lenguaje de consulta que puede usar para recorrer una red de relaciones de forma eficaz. 

Servicio de Azure correspondiente: [Cosmos DB][cosmosdb]

## <a name="column-family-databases"></a>Bases de datos de familia de columnas

Una base de datos de familia de columnas organiza los datos en filas y columnas. En su forma más simple, una base de datos de familia de columnas puede parecer muy similar a una base de datos relacional, al menos desde el punto de vista conceptual. La eficacia real de una base de datos de familia de columnas radica en su enfoque desnormalizado para estructurar datos dispersos. 

Puede pensar en una base de datos de familia de columnas como un elemento que contiene datos tabulares con filas y columnas, pero las columnas se dividen en grupos conocidos como *familias de columnas*. Cada familia de columnas contiene un conjunto de columnas que están relacionadas de forma lógica entre ellas y que se pueden recuperar o manipular normalmente como una unidad. Otros datos a los que se accede de forma independiente se pueden almacenar en familias de columnas independientes. En una familia de columnas, se pueden agregar nuevas columnas dinámicamente y las filas se pueden dispersar (es decir, no es necesario que una fila tenga un valor para cada columna).

El siguiente diagrama muestra un ejemplo con dos familias de columnas, `Identity` y `Contact Info`. Los datos de una sola entidad tienen la misma clave de fila en cada familia de columnas. Esta estructura, en la que las filas de cualquier objeto determinado de una familia de columnas puede variar dinámicamente, constituye una ventaja importante del enfoque de familia de columnas, que hace que este tipo de almacén de datos resulte muy adecuado para almacenar datos estructurados y volátiles.

![](./images/column-family.png) 

A diferencia de un almacén clave/valor o una base de datos de documentos, la mayoría de las bases de datos de familia de columnas almacenan los datos en el orden de la clave, en lugar de mediante el cálculo de un algoritmo hash. Muchas implementaciones le permiten crear índices en columnas específicas de una familia de columnas. Los índices le permiten recuperar datos por el valor de las columnas, en lugar de por la clave de fila.

Las operaciones de lectura y escritura de una fila son normalmente atómicas con una sola familia de columnas, aunque algunas implementaciones proporcionan atomicidad en toda la fila abarcando varias familias de columnas.

Servicio de Azure correspondiente: [HBase en HDInsight][hbase]

## <a name="data-analytics"></a>Análisis de datos

Los almacenes de análisis de datos proporcionan masivamente soluciones paralelas para ingerir, almacenar y analizar datos. Estos datos se distribuyen entre varios servidores mediante una arquitectura de uso no compartido para maximizar la escalabilidad y minimizar las dependencias. Es improbable que los datos sean estáticos, por lo que estos almacenes deben ser capaces de administrar grandes cantidades de información que llega en varios formatos desde varios flujos, mientras continúa procesando nuevas consultas. 

Servicios de Azure correspondientes:

- [SQL Data Warehouse][sql-dw]
- [Azure Data Lake][data-lake]

## <a name="search-engine-databases"></a>Bases de datos de motor de búsqueda  

Una base de datos de motor de búsqueda admite la capacidad de buscar información contenida en servicios y almacenes de datos externos. Una base de datos de motor de búsqueda se puede utilizar para indexar volúmenes de datos masivos y proporcionar casi en tiempo real acceso a estos índices. Aunque se considera a las bases de datos de motor de búsqueda similares a la web, muchos sistemas a gran escala las usan para proporcionar unas funcionalidades de búsqueda estructuradas y ad-hoc sobre sus propias bases de datos.

Las características clave de una base de datos de motor de búsqueda son la capacidad de almacenar e indexar la información muy rápidamente, y la de proporcionar unos tiempos de respuesta rápidos para las solicitudes de búsqueda. Los índices pueden ser multidimensionales y pueden admitir búsquedas de texto sin formato en grandes volúmenes de datos de texto. La indexación se puede realizar mediante un modelo de extracción, desencadenado por la base de datos de motor de búsqueda, o mediante un modelo de inserción, iniciado por un código externo de la aplicación. 

La búsqueda puede ser exacta o aproximada. Una búsqueda aproximada busca documentos que coinciden con un conjunto de términos y calcula el grado de coincidencia. Algunos motores de búsqueda también admiten el análisis lingüístico que puede devolver coincidencias basadas en sinónimos, expansiones de género (por ejemplo, una coincidencia entre `dogs` a `pets`) y lematización (coincidencia de palabras con la misma raíz). 

Servicio de Azure correspondiente: [Azure Search][search]

## <a name="time-series-databases"></a>Bases de datos de series temporales

Los datos de series temporales son un conjunto de valores organizados por tiempo, y una base de datos de series temporales es una base de datos optimizada para este tipo de datos. Las bases de datos de series temporales deben admitir un número muy elevado de operaciones de escritura, ya que suelen recopilar grandes cantidades de datos en tiempo real a partir de un gran número de orígenes. Las actualizaciones son poco frecuentes y las eliminaciones se realizan a menudo como operaciones masivas. Aunque los registros que se escriben en una base de datos de series temporales suelen ser pequeños, a menudo hay un gran número de registros y el tamaño total de los datos puede crecer rápidamente.

Las bases de datos de series temporales son buenas para almacenar los datos de telemetría. Los escenarios incluyen sensores de IoT o contadores de sistemas y aplicaciones.

Servicio de Azure correspondiente: [Time Series Insights][time-series]

## <a name="object-storage"></a>Almacenamiento de objetos  

El almacenamiento de objetos está optimizado para almacenar y recuperar objetos binarios grandes (imágenes, archivos, transmisiones de vídeo y audio, objetos de datos de aplicación de gran tamaño, documentos e imágenes de disco de máquina virtual). Los objetos de estos tipos de almacenes están compuestos de los datos almacenados, algunos metadatos y un identificador único para acceder al objeto. Los almacenes de objetos permiten la administración de cantidades enormes de datos no estructurados.  

Servicio de Azure correspondiente: [Blob Storage][blob]

## <a name="shared-files"></a>Archivos compartidos   

En ocasiones, el uso de archivos planos simples puede ser el medio más eficaz para almacenar y recuperar información. El uso de recursos compartidos de archivos permite acceder a los archivos a través de una red. Si se proporciona la seguridad adecuada y los mecanismos de control de acceso simultáneo, el uso compartido de datos de esta forma puede permitir a los servicios distribuidos proporcionar un acceso altamente escalable a los datos para realizar operaciones básicas, de nivel bajo, tales como solicitudes sencillas de lectura y escritura.

Servicio de Azure correspondiente: [File Storage][file-storage]

<!-- links -->

[blob]: https://azure.microsoft.com/services/storage/blobs/
[cosmosdb]: https://azure.microsoft.com/services/cosmos-db/
[data-lake]: https://azure.microsoft.com/solutions/data-lake/
[file-storage]: https://azure.microsoft.com/services/storage/files/
[hbase]: /azure/hdinsight/hdinsight-hbase-overview
[mysql]: https://azure.microsoft.com/services/mysql/
[postgres]: https://azure.microsoft.com/services/postgresql/
[redis-cache]: https://azure.microsoft.com/services/cache/
[search]: https://azure.microsoft.com/services/search/
[sql-db]: https://azure.microsoft.com/services/sql-database
[sql-dw]: https://azure.microsoft.com/services/sql-data-warehouse/
[time-series]: https://azure.microsoft.com/services/time-series-insights/