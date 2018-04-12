---
title: Elección de un almacén de datos de búsqueda
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: ead07e307e96696faa5ddf48505eee378027523c
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2018
---
# <a name="choosing-a-search-data-store-in-azure"></a>Elección de un almacén de datos de búsqueda en Azure

Este artículo compara las opciones de tecnologías para almacenes de datos de búsqueda de Azure. Un almacén de datos de búsqueda se utiliza para crear y almacenar índices especializados para realizar búsquedas en texto sin formato. El texto que se indexa puede residir en un almacén de datos independiente, como el almacenamiento de blobs. Una aplicación envía una consulta al almacén de datos de búsqueda y el resultado es una lista de documentos coincidentes. Para más información acerca de este escenario, consulte [Procesamiento de texto de formato libre para búsquedas](../scenarios/search.md). 

## <a name="what-are-your-options-when-choosing-a-search-data-store"></a>¿Cuáles son las opciones al elegir un almacén de datos de búsqueda?
En Azure, todos los almacenes de datos siguientes cumplirán los requisitos principales para la búsqueda de datos de texto de formato libre proporcionando un índice de búsqueda:
- [Azure Search](/azure/search/search-what-is-azure-search)
- [Elasticsearch](https://azuremarketplace.microsoft.com/marketplace/apps/elastic.elasticsearch?tab=Overview)
- [HDInsight con Solr](/azure/hdinsight/hdinsight-hadoop-solr-install-linux)
- [Azure SQL Database con búsqueda de texto completo](/sql/relational-databases/search/full-text-search)


## <a name="key-selection-criteria"></a>Principales criterios de selección

En escenarios de búsqueda, puede comenzar por la selección del almacén de datos de búsqueda adecuado para sus necesidades respondiendo a estas preguntas:

- ¿Quiere un servicio administrado en lugar de administrar sus propios servidores?

- ¿Puede especificar el esquema del índice en tiempo de diseño? De lo contrario, elija una opción que admita esquemas actualizables.

- ¿Necesita un índice solo para la búsqueda de texto completo o también necesita agregación de datos numéricos rápida y otros análisis? Si necesita funcionalidad más allá de la búsqueda de texto completo, considere las opciones que admitan análisis adicionales.

- ¿Necesita un índice de búsqueda de análisis de registros, con compatibilidad con la recolección de registros, agregación y visualizaciones de datos indexados? Si es así, considere la posibilidad de Elasticsearch, que forma parte de una pila de análisis de registros.

- ¿Necesita indexar datos en formatos de documento comunes como PDF, Word, PowerPoint y Excel? En caso afirmativo, elija una opción que proporcione indexadores de documentos.

- ¿La base de datos tiene necesidades específicas de seguridad? En caso afirmativo, tenga en cuenta las siguientes características de seguridad.

## <a name="capability-matrix"></a>Matriz de funcionalidades

En las tablas siguientes se resumen las diferencias clave en cuanto a funcionalidades.

### <a name="general-capabilities"></a>Funcionalidades generales

| | Azure Search | Elasticsearch | HDInsight con Solr | SQL Database | 
| --- | --- | --- | --- | --- | 
| Es un servicio administrado | Sí | Sin  | Sí | Sí |  
| API DE REST | Sí | Sí | Sí | Sin  |
| Capacidad de programación | .NET | Java | Java | T-SQL | 
| Indexadores de documentos para los tipos de archivo más comunes (PDF, DOCX, TXT y otros) | Sí | Sin  | Sí | Sin  |

### <a name="manageability-capabilities"></a>Funcionalidades de administración

| | Azure Search | Elasticsearch | HDInsight con Solr | SQL Database | 
| --- | --- | --- | --- | --- |
| Esquema actualizable | Sin  | Sí | Sí | Sí |
| Admite el escalado horizontal  | Sí | Sí | Sí | Sin  |

### <a name="analytic-workload-capabilities"></a>Funcionalidades de cargas de trabajo de análisis

| | Azure Search | Elasticsearch | HDInsight con Solr | SQL Database | 
| --- | --- | --- | --- | --- | 
| Admite el análisis más allá de la búsqueda de texto completo | Sin  | Sí | Sí | Sí |
| Forma parte de una pila de análisis de registros | Sin  | Sí (ELK) |  Sin  | Sin  |
| Admite búsqueda semántica | Sí (solo búsqueda de documentos similares) | Sí | Sí | Sí | 

### <a name="security-capabilities"></a>Funcionalidades de seguridad

| | Azure Search | Elasticsearch | HDInsight con Solr | SQL Database | 
| --- | --- | --- | --- | --- | 
| Seguridad de nivel de fila | Parcial (requiere una consulta de la aplicación para filtrar por el identificador de grupo) | Parcial (requiere una consulta de la aplicación para filtrar por el identificador de grupo) | Sí | Sí | 
| Cifrado de datos transparente | Sin  | Sin  | Sin  | Sí |  
| Restricción del acceso a determinadas direcciones IP | Sin  | Sí | Sí | Sí |   
| Restricción del acceso para permitir solo el acceso de la red virtual | Sin  | Sí | Sí | Sí |  
| Autenticación de Active Directory (autenticación integrada) | Sin  | Sin  | Sin  | Sí | 

## <a name="see-also"></a>Otras referencias

[Procesamiento de texto de formato libre para búsquedas](../scenarios/search.md)