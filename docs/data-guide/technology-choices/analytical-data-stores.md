---
title: Elección de un almacén de datos analíticos
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: cdc32c16e30aec5e1c0cb6959182215f99d56b56
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2018
ms.locfileid: "30846889"
---
# <a name="choosing-an-analytical-data-store-in-azure"></a>Elección de un almacén de datos analíticos en Azure

En una arquitectura de [macrodatos](../big-data/index.md), a menudo se necesita un almacén de datos analíticos que sirva los datos procesados en un formato estructurado que se pueda consultar mediante herramientas de análisis. Los almacenes de datos analíticos que admiten la consulta tanto de los datos de la ruta de acceso rápido como de la ruta de acceso preciso se denominan colectivamente la capa de servicios o el almacenamiento de servicios de datos.

La capa de servicio afecta a los datos procesados tanto de la ruta de acceso activa a la ruta de acceso inactiva. En la [arquitectura lambda](../big-data/index.md#lambda-architecture), la capa de servicio se subdivide en una capa de _servicios de velocidad_, que almacena los datos que se ha procesado de forma incremental, y una capa de _servicios por lotes_, que contiene la salida procesada por lotes. La capa de servicios requiere una compatibilidad total con las lecturas aleatorias con latencia baja. El almacenamiento de datos en la capa de velocidad también debe admitir operaciones de escritura aleatorias, ya que el lote que carga los datos en este almacén introduciría retrasos no deseados. Por otra parte, el almacenamiento de datos en la capa por lotes no es preciso que admita operaciones de escritura aleatorias, sino operaciones de escritura por lotes.

No hay ninguna opción individual de administración de datos que sea la mejor para todas las tareas del almacenamiento de datos. Las distintas soluciones de administración de datos están optimizadas para tareas diferentes. La mayoría de las aplicaciones en la nube y procesos de macrodatos reales tienen una gran variedad de requisitos de almacenamiento de datos y a menudo usan una combinación de soluciones de almacenamiento de datos.

## <a name="what-are-your-options-when-choosing-an-analytical-data-store"></a>¿Cuáles son las opciones disponibles cuando se elige un almacén de datos analíticos?

Hay varias opciones para el almacenamiento de servicios de datos en Azure, con el fin de que pueda elegir la que más se ajuste a sus necesidades:

- [SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is)
- [Azure SQL Database](/azure/sql-database/)
- [SQL Server en VM de Azure](/sql/sql-server/sql-server-technical-documentation)
- [HBase/Phoenix en HDInsight](/azure/hdinsight/hbase/apache-hbase-overview)
- [Hive LLAP en HDInsight](/azure/hdinsight/interactive-query/apache-interactive-query-get-started)
- [Azure Analysis Services](/azure/analysis-services/analysis-services-overview)
- [Azure Cosmos DB](/azure/cosmos-db/)

Estas opciones proporcionan varios modelos de base de datos que están optimizados para los distintos tipos de tareas:

- Las bases de datos de [clave/valor](https://msdn.microsoft.com/library/dn313285.aspx#sec7) contienen un objeto serializado individual para cada valor de clave. Se recomiendan para almacenar grandes volúmenes de datos donde se desee obtener un elemento para un valor de clave determinado valor de clave y no tiene que realizar consultas basándose en otras propiedades del elemento.
- Las bases de datos de [documentos](https://msdn.microsoft.com/library/dn313285.aspx#sec8) son las bases de datos de clave/valor en las que los valores son *documentos*. En este contexto, un "documento" es una colección de campos con nombre y valores. Normalmente, la base de datos almacena los datos en un formato como XML, YAML, JSON o BSON, pero puede usar texto sin formato. Las bases de datos de documentos pueden realizar consultas en los campos que no sean clave y definir índices secundarios, con el fin de aumentar la eficacia de las consultas. Esto hace que las bases de datos de documentos sean más apropiadas para aplicaciones que requieren recuperar datos según más complejos que el valor de la clave del documento. Por ejemplo, puede realizar consultas en campos como el de id. del producto, id. del cliente o nombre del cliente.
- Las bases de datos de [familia de columnas](https://msdn.microsoft.com/library/dn313285.aspx#sec9) datos son los almacenes de datos de clave/valor que estructuran el almacenamiento de datos en colecciones de columnas relacionadas denominadas familias de columnas. Por ejemplo, una base de datos del censo podría tener un grupo de columnas para el nombre de una persona (nombre, segundo nombre y apellidos), un grupo para la dirección de la persona y un grupo para la información del perfil de la persona (fecha de nacimiento y género). La base de datos puede almacenar cada familia de columnas en una partición independiente, al tiempo que mantiene todos los datos de una persona relacionados con la misma clave. Una aplicación puede leer la familia de una sola columna sin necesidad de leer todos los datos de una entidad.
- Las bases de datos de [grafos](https://msdn.microsoft.com/library/dn313285.aspx#sec10) almacenan la información como una colección de objetos y relaciones. Una base de datos de grafos puede realizar de manera eficaz consultas que atraviesan la red de objetos y las relaciones entre ellos. Por ejemplo, los objetos podrían ser los empleados en una base de datos de recursos humanos y puede desear facilitar consultas como "buscar todos los empleados que trabajan directa o indirectamente para Scott".

## <a name="key-selection-criteria"></a>Principales criterios de selección

Para restringir las opciones, empiece por responder a estas preguntas:

- ¿Necesita almacenamiento de servicios que pueda servir como ruta de acceso activa para los datos? En caso afirmativo, limite las opciones a las que están optimizados para una capa de servicios de velocidad.

- ¿Necesita compatibilidad con el procesamiento paralelo masivo (MPP), donde las consultas se distribuyen automáticamente a través de varios procesos o nodos? Si es así, seleccione una opción que admita el escalado horizontal de consultas.

- ¿Prefiere usar un almacén de datos relacional? Si es así, limite las opciones a las que tengan un modelo de base de datos relacional. Sin embargo, tenga en cuenta que algunos almacenes no relacionales admiten la sintaxis SQL para consultar y herramientas como PolyBase se pueden usar para consultar los almacenes de datos no relacionales.

## <a name="capability-matrix"></a>Matriz de funcionalidades

En las tablas siguientes se resumen las diferencias clave en cuanto a funcionalidades.

### <a name="general-capabilities"></a>Funcionalidades generales

| | SQL Database | SQL Data Warehouse | HBase/Phoenix en HDInsight | Hive LLAP en HDInsight | Azure Analysis Services | Cosmos DB |
| --- | --- | --- | --- | --- | --- | --- |
| Es un servicio administrado | Sí | Sí | Sí <sup>1</sup> | Sí <sup>1</sup> | Sí | Sí |
| Modelo de la base de datos principal | Relacional (formato de columnas cuando se usan índices de almacén de columnas) | Tablas relacionales con almacenamiento en columnas | Almacenamiento de columnas anchas | Hive/In-Memory | Modelos semánticos tabular/MOLAP | Almacenamiento de documentos, grafos, almacenamiento de clave-valor, almacenamiento de columnas anchas |
| Compatibilidad con lenguaje SQL | Sí | Sí | Sí (mediante el controlador JDBC [Phoenix](http://phoenix.apache.org/)) | Sí | Sin  | Sí |
| Optimizado para la capa de servicios de velocidad | Sí <sup>2</sup> | Sin  | Sí | Sí | Sin  | Sí |

[1] Con configuración y escalado manuales.

[2] Mediante tablas optimizadas para memoria y hash o índices no agrupados en clúster.
 
### <a name="scalability-capabilities"></a>Funcionalidades de escalabilidad

|                                                  | SQL Database | SQL Data Warehouse | HBase/Phoenix en HDInsight | Hive LLAP en HDInsight | Azure Analysis Services | Cosmos DB |
|--------------------------------------------------|--------------|--------------------|----------------------------|------------------------|-------------------------|-----------|
| Servidores regionales redundantes para lograr alta disponibilidad |     Sí      |        Sí         |            Sí             |           Sin            |           Sin             |    Sí    |
|             Admite el escalado horizontal de consultas             |      Sin       |        Sí         |            Sí             |          Sí           |           Sí           |    Sí    |
|          Escalabilidad dinámica (escalado vertical)          |     Sí      |        Sí         |             Sin              |           Sin            |           Sí           |    Sí    |
|        Admite el almacenamiento en caché en memoria de datos        |     Sí      |        Sí         |             Sin              |          Sí           |           Sí           |    Sin      |

### <a name="security-capabilities"></a>Funcionalidades de seguridad

| | SQL Database | SQL Data Warehouse | HBase/Phoenix en HDInsight | Hive LLAP en HDInsight | Azure Analysis Services | Cosmos DB |
| --- | --- | --- | --- | --- | --- | --- |
| Autenticación  | SQL/Azure Active Directory (Azure AD) | SQL/Azure AD | local/Azure AD <sup>1</sup> | local/Azure AD <sup>1</sup> | Azure AD | usuarios de la base de datos / Azure AD a través del control de acceso (IAM) |
| Cifrado de datos en reposo | Sí <sup>2</sup> | Sí <sup>2</sup> | Sí <sup>1</sup> | Sí <sup>1</sup> | Sí | Sí |
| Seguridad de nivel de fila | Sí | Sin  | Sí <sup>1</sup> | Sí <sup>1</sup> | Sí (mediante la seguridad a nivel de objeto en el modelo) | Sin  |
| Admite firewalls | Sí | Sí | Sí <sup>3</sup> | Sí <sup>3</sup> | Sí | Sí |
| Enmascaramiento de datos dinámicos | Sí | Sin  | Sí <sup>1</sup> | Sí * | Sin  | Sin  |

[1] Requiere el uso de un [clúster de HDInsight unido a un dominio](/azure/hdinsight/domain-joined/apache-domain-joined-introduction).

[2] Requiere el uso de cifrado de datos transparente (TDE) para cifrar y descifrar los datos en reposo.

[3] Cuando se usa en una instancia de Azure Virtual Network. Consulte [Extender Azure HDInsight mediante una instancia de Azure Virtual Network](/azure/hdinsight/hdinsight-extend-hadoop-virtual-network).
