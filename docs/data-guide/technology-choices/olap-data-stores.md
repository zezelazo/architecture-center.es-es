---
title: "Elección de un almacén de datos OLAP"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: f3041b95696c9408a2c9ab747fe1ec3041db0743
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/14/2018
---
# <a name="choosing-an-olap-data-store-in-azure"></a>Elección de un almacén de datos OLAP en Azure

El procesamiento analítico en línea (OLAP) es una tecnología que organiza grandes bases de datos empresariales y proporciona análisis complejo. Este tema compara las diversas opciones de soluciones de OLAP en Azure.

> [!NOTE]
> Para más información acerca de cuándo se debe usar un almacén de datos OLAP, consulte [Procesamiento analítico en línea](../scenarios/online-analytical-processing.md).

## <a name="what-are-your-options-when-choosing-an-olap-data-store"></a>¿Cuáles son las opciones al elegir un almacén de datos OLAP?

En Azure, los almacenes de datos siguientes cumplirán los requisitos principales para OLAP:

- [SQL Server con índices de almacén de columnas](/sql/relational-databases/indexes/get-started-with-columnstore-for-real-time-operational-analytics)
- [Azure Analysis Services](/azure/analysis-services/analysis-services-overview)
- [SQL Server Analysis Services (SSAS)](/sql/analysis-services/analysis-services)

SQL Server Analysis Services (SSAS) ofrece funciones de OLAP y de minería para aplicaciones de inteligencia empresarial. SSAS se puede instalar en servidores locales, o bien se puede hospedar en una máquina virtual en Azure. Azure Analysis Services es un servicio completamente administrado que proporciona las mismas características principales que SSAS. Azure Analysis Services admite la conexión a [varios orígenes de datos](/azure/analysis-services/analysis-services-datasource), que se encuentren tanto en la nube como en el entorno local, de su organización.

Los índices de almacén de columnas en clúster están disponibles en SQL Server 2014, y en las versiones posteriores, así como en Azure SQL Database, y son ideales para cargas de trabajo de OLAP. Sin embargo, a partir de SQL Server 2016 (incluido Azure SQL Database), puede sacar partido del procesamiento transaccional/analítico híbrido (HTAP) mediante el uso de índices de almacén de columnas no agrupados actualizables. HTAP permite realizar el procesamiento de OLTP y OLAP en la misma plataforma, lo que elimina la necesidad de almacenar varias copias de los datos y de tener distintos sistemas de OLTP y OLAP. Para más información, consulte [Introducción al almacén de columnas para análisis operativos en tiempo real](/sql/relational-databases/indexes/get-started-with-columnstore-for-real-time-operational-analytics).

## <a name="key-selection-criteria"></a>Principales criterios de selección

Para restringir las opciones, empiece por responder a estas preguntas:

- ¿Quiere un servicio administrado en lugar de administrar sus propios servidores?

- ¿Requiere autenticación segura mediante Azure Active Directory (Azure AD)?

- ¿Desea realizar análisis en tiempo real? En ese caso, limite las opciones a las que admitan análisis en tiempo real. 

    En este contexto, *análisis en tiempo real* se aplica a un origen de datos único, como una aplicación de planificación de recursos empresariales (ERP), que ejecutará una carga de trabajo tanto operativa como de análisis. Si tiene que integrar datos de varios orígenes, o requieren un rendimiento extremo de los análisis mediante el uso de datos previamente agregados, como cubos, puede que necesite un almacén de datos independiente.

- ¿Necesita usar datos previamente agregados, por ejemplo, para proporcionar los modelos semánticos que hacen que los usuarios empresariales entiendan mejor los análisis? En ese caso, elija una opción que admita cubos multidimensionales o modelos semánticos tabulares. 

    El hecho de proporcionar agregados puede ayudar a los usuarios a calcular agregados de datos de forma coherente. Los datos agregados previamente también pueden mejoras significativamente el rendimiento cuando se trabaja con varias columnas en muchas filas. Los datos se pueden agregar previamente en cubos multidimensionales o modelos semánticos tabulares.

- ¿Necesita integrar datos de varios orígenes, más allá de su almacén de datos de OLTP? Si es así, considere la posibilidad de usar opciones que integren fácilmente varios orígenes de datos.

## <a name="capability-matrix"></a>Matriz de funcionalidades

En las tablas siguientes se resumen las diferencias clave en cuanto a funcionalidades.

### <a name="general-capabilities"></a>Funcionalidades generales

| | Azure Analysis Services | SQL Server Analysis Services | SQL Server con índices de almacén de columnas | Azure SQL Database con índices de almacén de columnas |
| --- | --- | --- | --- | --- |
| Es un servicio administrado | Sí | Sin  | Sin  | Sí |
| Admite cubos multidimensionales | Sin  | Sí | Sin  | Sin  |
| Admite modelos semánticos tabulares | Sí | Sí | Sin  | Sin  |
| Integra fácilmente varios orígenes de datos | Sí | Sí | No <sup>1</sup> | No <sup>1</sup> |
| Admite análisis en tiempo real | Sin  | Sin  | Sí | Sí |
| Requiere un proceso para copiar los datos de los orígenes | Sí | Sí | Sin  | Sin  |
| Integración de Azure AD | Sí | Sin  | No <sup>2</sup> | Sí |

[1] Aunque SQL Server y Azure SQL Database no se pueden usar para consultar varios orígenes de datos externos, e integrarlos, puede crear una canalización que lo haga automáticamente mediante [SSIS](/sql/integration-services/sql-server-integration-services) o [Azure Data Factory](/azure/data-factory/). SQL Server hospedado en una máquina virtual de Azure tiene opciones adicionales, como los servidores vinculados y [PolyBase](/sql/relational-databases/polybase/polybase-guide). Para más información, consulte [Choosing a data pipeline orchestration technology in Azure](../technology-choices/pipeline-orchestration-data-movement.md) (Elección de una tecnología de orquestación de canalizaciones de datos en Azure).

[2] La conexión a una instancia de SQL Server que se ejecute en Azure Virtual Machine no se admite si se usa una cuenta de Azure AD. Utilice en su lugar una cuenta de Active Directory del dominio.

### <a name="scalability-capabilities"></a>Funcionalidades de escalabilidad

| | Azure Analysis Services | SQL Server Analysis Services | SQL Server con índices de almacén de columnas | Azure SQL Database con índices de almacén de columnas |
| --- | --- | --- | --- | --- |
| Servidores regionales redundantes para lograr alta disponibilidad  | Sí | Sin  | Sí | Sí |
| Admite el escalado horizontal de consultas  | Sí | Sin  | Sí | Sin  |
| Escalabilidad dinámica (escalado vertical)  | Sí | Sin  | Sí | Sin  |

