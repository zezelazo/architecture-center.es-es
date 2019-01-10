---
title: Almacenamiento de datos y análisis de ventas y marketing
titleSuffix: Azure Example Scenarios
description: Consolide datos de varios orígenes y optimice el análisis de datos.
author: alexbuckgit
ms.date: 09/15/2018
ms.openlocfilehash: 2ac06fcd0805b66371fcc004794b123c46a6ce0e
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/08/2019
ms.locfileid: "54112386"
---
# <a name="data-warehousing-and-analytics-for-sales-and-marketing"></a>Almacenamiento de datos y análisis de ventas y marketing

En este escenario de ejemplo se muestra una canalización de datos que integra grandes cantidades de datos de varios orígenes en una plataforma unificada de análisis de Azure. Este escenario concreto se basa en una solución de venta y marketing, pero los modelos de diseño son importantes para muchas industrias que requieren análisis avanzado de grandes conjuntos de datos, como la asistencia sanitaria, el comercio electrónico y la venta al por menor.

En este ejemplo se muestra una empresa de marketing y venta que crea los programas de incentivos. Estos programas recompensan a los clientes, los proveedores, los vendedores y los empleados. Los datos son fundamentales para estos programas y la empresa quiere mejorar los conocimientos adquiridos mediante el análisis de datos con Azure.

La empresa necesita un enfoque moderno para analizar los datos, para que las decisiones se tomen con los datos adecuados en el momento oportuno. Los objetivos de la empresa incluyen:

- La combinación de distintos tipos de orígenes de datos en una plataforma en la nube.
- La transformación de los datos de origen a una estructura y taxonomía comunes, de manera que estos sean coherentes y se comparen con facilidad.
- La carga de datos mediante un enfoque altamente paralelizado que admita miles de programas de incentivos, sin el elevado costo de implementación y mantenimiento de infraestructura local.
- La reducción considerable del tiempo necesario para recopilar y transformar datos, para poder centrarse en el análisis de los datos.

## <a name="relevant-use-cases"></a>Casos de uso pertinentes

Este enfoque también se puede utilizar para:

- Establecer un almacén de datos como origen de datos único.
- Integrar orígenes de datos relacionales con otros conjuntos de datos desestructurados.
- Usar el modelado semántico y potentes herramientas de visualización para simplificar el análisis de los datos.

## <a name="architecture"></a>Arquitectura

![Arquitectura de un escenario de almacenamiento y análisis de datos en Azure][architecture]

Los datos fluyen por la solución de la siguiente manera:

1. Para cada origen de datos, las actualizaciones se exportan periódicamente a un área de almacenamiento provisional en Azure Blob Storage.
2. Data Factory carga los datos de manera incremental de Blob Storage a tablas de almacenamiento provisional en SQL Data Warehouse. Durante este proceso, los datos se limpian y se transforman. Polybase puede paralelizar el proceso para grandes conjuntos de datos.
3. Después de cargar un nuevo lote de datos en el almacén, se actualiza un modelo tabular de Analysis Services creado anteriormente. Este modelo semántico simplifica el análisis de datos y relaciones empresariales.
4. Los analistas de negocios usan Microsoft Power BI para analizar los datos del almacén mediante el modelo semántico de Analysis Services.

### <a name="components"></a>Componentes

La empresa tiene orígenes de datos en muchas plataformas diferentes:

- SQL Server local
- Oracle local
- Azure SQL Database
- Almacenamiento de tablas de Azure
- Cosmos DB

De estos orígenes de datos diferentes, los datos se cargan con varios componentes de Azure:

- [Blob Storage](/azure/storage/blobs/storage-blobs-introduction) se usa para almacenar los datos de origen antes de cargarlos en SQL Data Warehouse.
- [Data Factory](/azure/data-factory) organiza la transformación de los datos almacenados provisionalmente en una estructura común en SQL Data Warehouse. Data Factory [usa Polybase al cargar los datos en SQL Data Warehouse](/azure/data-factory/connector-azure-sql-data-warehouse#use-polybase-to-load-data-into-azure-sql-data-warehouse) para conseguir el máximo rendimiento.
- [SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is) es un sistema distribuido para almacenar y analizar grandes conjuntos de datos. Su uso del procesamiento paralelo masivo (MPP) lo hace idóneo para ejecutar análisis de alto rendimiento. SQL Data Warehouse puede usar [PolyBase](/sql/relational-databases/polybase/polybase-guide) para cargar datos rápidamente de Blob Storage.
- [Analysis Services](/azure/analysis-services) proporciona un modelo semántico para los datos. También puede aumentar el rendimiento del sistema al analizar los datos.
- [Power BI](/power-bi) es un conjunto de herramientas de análisis de negocios que sirve para analizar datos y compartir conocimientos. Power BI puede consultar un modelo semántico almacenado en Analysis Services o SQL Data Warehouse directamente.
- [Azure Active Directory (Azure AD)](/azure/active-directory) autentica a los usuarios que se conectan al servidor de Analysis Services mediante Power BI. Data Factory puede usar también Azure AD para autenticarse en SQL Data Warehouse mediante una entidad de servicio o una [Identidad administrada para los recursos de Azure](/azure/active-directory/managed-identities-azure-resources/overview).

### <a name="alternatives"></a>Alternativas

- La canalización de ejemplo incluye varios tipos diferentes de orígenes de datos. Esta arquitectura funciona con una amplia variedad de orígenes de datos relacionales y de otro tipo.
- Data Factory orquesta los flujos de trabajo para la canalización de datos. Si desea cargar datos solo una vez o a petición, también puede usar herramientas como la copia masiva de SQL Server (bcp) y AzCopy para copiar datos en Blob Storage. Después puede cargar los datos con PolyBase directamente en SQL Data Warehouse.
- Si tiene grandes conjuntos de datos, considere [Data Lake Storage](/azure/storage/data-lake-storage/introduction), que proporciona almacenamiento ilimitado para datos de análisis.
- Para el procesamiento de macrodatos también puede usarse un dispositivo de [Parallel Data Warehouse de SQL Server](/sql/analytics-platform-system). Sin embargo, a menudo los costos operativos son mucho menores con una solución administrada en la nube del tipo SQL Data Warehouse.
- SQL Data Warehouse no es una buena opción para cargas de trabajo OLTP o conjuntos de datos menores que 250 GB. En esos casos debe usar Azure SQL Database o SQL Server.
- Para comparar con otras alternativas, consulte:

  - [Elección de una tecnología de orquestación de canalizaciones de datos en Azure](/azure/architecture/data-guide/technology-choices/pipeline-orchestration-data-movement)
  - [Selección de una tecnología de procesamiento por lotes en Azure](/azure/architecture/data-guide/technology-choices/batch-processing)
  - [Elección de un almacén de datos analíticos en Azure](/azure/architecture/data-guide/technology-choices/analytical-data-stores)
  - [Elección de una tecnología de análisis de datos en Azure](/azure/architecture/data-guide/technology-choices/analysis-visualizations-reporting)

## <a name="considerations"></a>Consideraciones

Las tecnologías de esta arquitectura se eligieron porque cumplen requisitos de la empresa respecto a escalabilidad y disponibilidad, la tiempo que ayudan a controlar los costos.

- La [arquitectura de procesamiento paralelo masivo](/azure/sql-data-warehouse/massively-parallel-processing-mpp-architecture) de SQL Data Warehouse proporciona escalabilidad y alto rendimiento.
- SQL Data Warehouse tiene [Acuerdos de Nivel de Servicio garantizados](https://azure.microsoft.com/support/legal/sla/sql-data-warehouse) y [procedimientos recomendados para una alta disponibilidad](/azure/sql-data-warehouse/sql-data-warehouse-best-practices).
- Cuando haya poca actividad de análisis, la empresa puede [escalar SQL Data Warehouse a petición](/azure/sql-data-warehouse/sql-data-warehouse-manage-compute-overview), lo que reduce o incluso pausa el proceso para reducir los costos.
- Azure Analysis Services se puede [escalar horizontalmente](/azure/analysis-services/analysis-services-scale-out) para reducir los tiempos de respuesta durante las grandes cargas de trabajo de consulta. También puede separar el procesamiento del grupo de consultas, de manera que las consultas de los clientes no se ralenticen a causa del procesamiento.
- Azure Analysis Services también tiene [Acuerdos de Nivel de Servicio garantizados](https://azure.microsoft.com/support/legal/sla/analysis-services) y [procedimientos recomendados para una alta disponibilidad](/azure/analysis-services/analysis-services-bcdr).
- El [modelo de seguridad de SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-overview-manage-security) proporciona seguridad de conexión, [autenticación y autorización](/azure/sql-data-warehouse/sql-data-warehouse-authentication) a través de la autenticación de Azure AD o SQL Server y el cifrado. [Azure Analysis Services](/azure/analysis-services/analysis-services-manage-users) usa Azure AD para la administración de identidades y la autenticación de usuarios.

## <a name="pricing"></a>Precios

Revise un [precio de ejemplo para un escenario de almacenamiento de datos][calculator] con la calculadora de precios de Azure. Ajuste los valores para ver cómo afectan los requisitos a los costos.

- [SQL Data Warehouse](https://azure.microsoft.com/pricing/details/sql-data-warehouse/gen2) permite escalar los niveles de proceso y almacenamiento por separado. Los recursos de proceso se cobran por hora; además, estos recursos se pueden escalar o pausar a petición. Los recursos de almacenamiento se facturan por terabyte, por lo que los costos aumentan con la ingestión de datos.
- Los costos de [Data Factory](https://azure.microsoft.com/pricing/details/data-factory) se basan en el número de operaciones de lectura/escritura, las operaciones de supervisión y las actividades de orquestación realizadas en una carga de trabajo. Estos aumentan con cada flujo de datos adicional y la cantidad de datos que procese cada uno.
- [Analysis Services](https://azure.microsoft.com/pricing/details/analysis-services) está disponible en los planes de tarifa estándar, básica y de desarrollador. Las instancias se pagan en función de las unidades de procesamiento de consultas (QPU) y la memoria disponible. Para mantener los costos más bajos, minimice el número de consultas que ejecuta, la cantidad de datos que procesan y la frecuencia de ejecución.
- [Power BI](https://powerbi.microsoft.com/pricing) tiene opciones de producto diferentes para distintos requisitos. [Power BI Embedded](https://azure.microsoft.com/pricing/details/power-bi-embedded) proporciona una opción basada en Azure para insertar la funcionalidad de Power BI en las aplicaciones. En el precio de ejemplo anterior se incluye una instancia de Power BI Embedded.

## <a name="next-steps"></a>Pasos siguientes

- Revise la [arquitectura de referencia de Azure para la inteligencia empresarial automatizada](/azure/architecture/reference-architectures/data/enterprise-bi-adf), que incluye instrucciones para implementar una instancia de esta arquitectura en Azure.
- Lea el [historia del cliente sobre las soluciones de motivación Maritz][source-document]. Esa historia describe un enfoque similar para administrar los datos del cliente.
- En la [Guía de arquitectura de datos de Azure](/azure/architecture/data-guide) encontrará información estructural completa sobre las canalizaciones de datos, el almacenamiento de datos, el procesamiento analítico en línea (OLAP) y los macrodatos.

<!-- links -->

[source-document]: https://customers.microsoft.com/story/maritz
[calculator]: https://azure.com/e/b798fb70c53e4dd19fdeacea4db78276
[architecture]: ./media/architecture-data-warehouse.png
