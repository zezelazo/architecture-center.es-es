---
title: Extracción, transformación y carga de datos híbrida con instancias locales ya existentes de SQL Server Integration Services y Azure Data Factory
description: Extracción, transformación y carga de datos híbrida con implementaciones locales ya existentes de SQL Server Integration Services (SSIS) y Azure Data Factory
author: alhieng
ms.date: 9/20/2018
ms.openlocfilehash: c4c0cfd63ef1d6c620eb36e16622ad9ffb7b5d80
ms.sourcegitcommit: 16bc6a91b6b9565ca3bcc72d6eb27c2c4ae935e4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/28/2018
ms.locfileid: "52579481"
---
# <a name="hybrid-etl-with-existing-on-premises-ssis-and-azure-data-factory"></a>Extracción, transformación y carga de datos híbrida con instancias locales ya existentes de SQL Server Integration Services y Azure Data Factory

Las organizaciones que van a migrar sus bases de datos de SQL Server a la nube podrán ver el enorme ahorro de costos, las mejoras de rendimiento y las mayores flexibilidad y escalabilidad. Sin embargo, adaptar los procesos existentes de extracción, transformación y carga (ETL) creados con SQL Server Integration Services (SSIS) puede resultar un obstáculo para la migración. En otros casos, el proceso de carga de datos requiere una lógica compleja o componentes específicos de herramientas de datos que aún no se admiten en la versión 2 de Azure Data Factory (ADF). Entre las funcionalidades SSIS que se utilizan frecuentemente están las transformaciones de búsqueda y agrupación aproximada, la captura de datos modificados (CDC), las dimensiones de cambio lento (SCD) y los servicios de calidad de datos (DQS).

Para facilitar una migración mediante lift-and-shift de una base de datos SQL existente, lo más adecuado puede ser un enfoque ETL híbrido. Un enfoque híbrido usa ADF como el motor de orquestación principal, pero continúa aprovechando los paquetes SSIS existentes para limpiar los datos y trabajar con los recursos locales. Este enfoque utiliza el entorno de ejecución de integración de SQL Server en ADF para habilitar la migración mediante lift-and-shift de las bases de datos existentes a la nube, usando el código existente y los paquetes SSIS.

Este escenario de ejemplo es aplicable a las organizaciones que van a trasladar las bases de datos a la nube y están pensando en utilizar ADF como motor de ETL principal en la nube durante la incorporación de los paquetes SSIS existentes a su nuevo flujo de trabajo de datos en la nube. Muchas organizaciones han realizado grandes inversiones en el desarrollo de paquetes ETL SSIS para tareas específicas de datos. Volver a escribir estos paquetes puede resultar desalentador. Además, muchos de los paquetes de código existentes tienen dependencias de recursos locales, lo que impide la migración a la nube.

ADF permite a los clientes aprovechar las ventajas de sus paquetes ETL existentes, lo que reduce las futuras inversiones en el desarrollo de ETL en el entorno local. En este ejemplo se describen posibles casos sobre cómo usar Azure Data Factory v2 para aprovechar los paquetes SSIS existentes como parte de un nuevo flujo de datos en la nube.

## <a name="potential-use-cases"></a>Posibles casos de uso

Tradicionalmente, SSIS ha sido la herramienta ETL preferida por muchos profesionales de datos de SQL Server para la transformación y carga de datos. A veces, se han utilizado características específicas de SSIS o componentes de terceros para acelerar las labores de desarrollo. Reemplazar o volver a desarrollar estos paquetes podría no ser viable, lo que impide que los clientes puedan migrar sus bases de datos a la nube. Los clientes buscan enfoques de bajo impacto para migrar sus bases de datos existentes a la nube y sacar partido de sus paquetes SSIS existentes.

A continuación se enumeran varios posibles casos de uso en el entorno local:

* Cargar los registros de enrutador de red a una base de datos para su análisis.
* Preparar los datos de empleo de recursos humanos para informes analíticos.
* Cargar datos de productos y ventas en un almacén de datos para la previsión de ventas.
* Automatizar la carga, los almacenes de datos operativos o los almacenamientos de datos para finanzas y contabilidad.

## <a name="architecture"></a>Arquitectura

![Introducción a la arquitectura de un proceso ETL híbrido mediante Azure Data Factory][architecture-diagram]

1. Los datos se obtienen de Blob Storage a Data Factory.
2. La canalización de Data Factory invoca un procedimiento almacenado para ejecutar un trabajo de SSIS hospedado en el entorno local mediante el entorno de ejecución de integración.
3. Los trabajos de limpieza de datos se ejecutan para preparar los datos para el consumo en niveles inferiores.
4. Una vez que la tarea de limpieza de datos se completa correctamente, se ejecuta una tarea de copia para cargar los datos limpios en Azure.
5. A continuación, los datos limpios se cargan en tablas en SQL Data Warehouse.

### <a name="components"></a>Componentes

* Se utiliza [Blob Storage][docs-blob-storage] para almacenar archivos y como origen de Data Factory para recuperar los datos.
* [SQL Server Integration Services][docs-ssis] contiene los paquetes ETL locales que se usan para ejecutar cargas de trabajo específicas de la tarea.
* [Azure Data Factory][docs-data-factory] es el motor de orquestación en la nube que toma los datos de varios orígenes y los combina, los organiza y los carga en un almacenamiento de datos.
* [SQL Data Warehouse][docs-sql-data-warehouse] centraliza los datos en la nube para facilitar el acceso mediante consultas SQL ANSI estándar.

### <a name="alternatives"></a>Alternativas

Data Factory podría invocar procedimientos de limpieza de datos implementados mediante otras tecnologías, como un cuaderno de Databricks, un script de Python o una instancia de SSIS que se ejecuta en una máquina virtual. [La instalación componentes personalizados de pago o con licencia el entorno de ejecución de integración de SSIS en Azure](/azure/data-factory/how-to-develop-azure-ssis-ir-licensed-components) puede ser una alternativa viable para el enfoque híbrido.

## <a name="considerations"></a>Consideraciones

El entorno de ejecución de integración (IR) admite dos modelos: IR autohospedado o IR hospedado en Azure En primer lugar, debe decidir entre estas dos opciones. El autohospedaje es más rentable, pero tiene más sobrecarga de administración y mantenimiento. Para más información, consulte [IR autohospedado](/azure/data-factory/concepts-integration-runtime#self-hosted-integration-runtime). Si necesita ayuda para determinar el tipo de IR, consulte [Determinar qué tipo de IR se usará](/azure/data-factory/concepts-integration-runtime#determining-which-ir-to-use).

Para el sistema hospedado en Azure, debe decidir cuánta capacidad necesita para procesar los datos. La configuración hospedada en Azure le permite seleccionar el tamaño de máquina virtual como parte de los pasos de configuración. Para más información acerca de cómo seleccionar los tamaños de máquina virtual, consulte [Consideraciones sobre el rendimiento de las máquinas virtuales](/azure/cloud-services/cloud-services-sizes-specs#performance-considerations).

La decisión es mucho más fácil si ya tiene paquetes SSIS existentes que tienen dependencias en el entorno local, como orígenes de datos o archivos que no son accesibles desde Azure. En este escenario, la única opción es un IR autohospedado. Este enfoque ofrece la máxima flexibilidad para aprovechar la nube como el motor de orquestación, sin tener que reescribir los paquetes existentes.

En última instancia, la intención es trasladar los datos procesados a la nube para seguir refinándolos o combinándolos con otros datos almacenados en la nube. Como parte del proceso de diseño, realice un seguimiento de la cantidad de actividades que se usan en las canalizaciones ADF. Para más información, consulte [Canalizaciones y actividades en Azure Data Factory](/azure/data-factory/concepts-pipelines-activities).

## <a name="pricing"></a>Precios

Azure Data Factory es una opción rentable para orquestar el movimiento de datos en la nube. El costo se basa en diversos factores.

* Número de ejecuciones de canalización
* Número de entidades o actividades utilizadas dentro de la canalización
* Número de operaciones de supervisión
* Número de ejecuciones de integración (IR hospedado en Azure o IR autohospedado)

ADF utiliza facturación por consumo Por lo tanto, solo se incurre en costos durante las ejecuciones y la supervisión de las canalizaciones. La ejecución de una canalización básica costaría tan solo 50 céntimos y, la supervisión, solo 25 céntimos. Use la [calculadora de costos de Azure](https://azure.microsoft.com/pricing/calculator/) para crear una estimación más precisa en función de su carga específica.

Cuando se ejecuta una carga de trabajo de ETL híbrida, debe incluir el costo de la máquina virtual que se utiliza para hospedar los paquetes SSIS. Este costo se basa en el tamaño de la máquina virtual, que van desde D1v2 (1 núcleo, 3,5 GB de RAM, 50 GB en disco) a E64V3 (64 núcleos, 432 GB de RAM, disco de 1600 GB).  Si necesita más orientación sobre cómo seleccionar el tamaño adecuado de máquina virtual, consulte [Consideraciones sobre el rendimiento de las máquinas virtuales](/azure/cloud-services/cloud-services-sizes-specs#performance-considerations).

## <a name="next-steps"></a>Pasos siguientes

* Más información acerca de [Azure Data Factory](https://azure.microsoft.com/services/data-factory/).
* Empiece a trabajar con Azure Data Factory siguiendo el [tutorial paso a paso](/azure/data-factory/#step-by-step-tutorials).
* [Aprovisionamiento de Azure-SSIS Integration Runtime en Azure Data Factory](/azure/data-factory/tutorial-deploy-ssis-packages-azure)

<!-- links -->
[architecture-diagram]: ./media/architecture-diagram-hybrid-etl-with-adf.png
[small-pricing]: https://azure.com/e/
[medium-pricing]: https://azure.com/e/
[large-pricing]: https://azure.com/e/
[availability]: /azure/architecture/checklist/availability
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[docs-blob-storage]: /azure/storage/blobs/
[docs-data-factory]: /azure/data-factory/introduction
[docs-resource-groups]: /azure/azure-resource-manager/resource-group-overview
[docs-ssis]: /sql/integration-services/sql-server-integration-services
[docs-sql-data-warehouse]: /azure/sql-data-warehouse/sql-data-warehouse-overview-what-is