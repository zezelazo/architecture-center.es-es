---
title: Exploración interactiva de datos
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 20740a8fe912a63526c847416b832941f4ac33ec
ms.sourcegitcommit: 51f49026ec46af0860de55f6c082490e46792794
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/03/2018
ms.locfileid: "30297967"
---
# <a name="interactive-data-exploration"></a>Exploración interactiva de datos

En muchas soluciones corporativas de inteligencia empresarial (BI), los especialistas en inteligencia empresarial crean informes y modelos semánticos y los administran de forma centralizada. Cada vez más, sin embargo, las organizaciones desean permitir a los usuarios tomar decisiones basadas en los datos. Además, un número creciente de organizaciones están contratando *científicos de datos* o *analistas de datos* cuyo trabajo es explorar datos de forma interactiva y aplicar los modelos estadísticos y las técnicas analíticas para buscar tendencias y patrones en los datos. La exploración interactiva de datos requiere herramientas y plataformas que proporcionen un procesamiento de baja latencia para consultas ad-hoc y visualizaciones de datos.

![](./images/data-exploration.png)

## <a name="self-service-bi"></a>Inteligencia artificial con características de autoservicio

Inteligencia artificial con características de autoservicio es un nombre asignado a un enfoque moderno para la toma de decisiones empresariales en el que los usuarios pueden buscar, explorar y compartir información a partir de datos con toda la empresa. Para lograr esto, la solución de datos debe admitir varios requisitos:

* Detección de orígenes de datos empresariales a través de un catálogo de datos.
* Administración de datos maestros para asegurar la coherencia entre las definiciones y los valores de las entidades de datos.
* Modelado de datos interactivo y herramientas de visualización para usuarios empresariales.

En una solución de inteligencia empresarial con características de autoservicio, los usuarios empresariales suelen encontrar y utilizar orígenes de datos que son relevantes para su área concreta del negocio, y utilizan herramientas intuitivas y aplicaciones de productividad para definir los modelos de datos personales y los informes que pueden compartir con los compañeros.

Servicios de Azure correspondientes:

- [Azure Data Catalog](/azure/data-catalog/data-catalog-what-is-data-catalog)
- [Microsoft Power BI](https://powerbi.microsoft.com/)

## <a name="data-science-experimentation"></a>Experimentación en ciencia de datos
Si una organización requiere un análisis avanzado y modelado predictivo, el trabajo de preparación inicial normalmente lo llevan a cabo científicos de datos especializados. Un científico de datos explora los datos y aplica técnicas de análisis estadístico para encontrar relaciones entre las *características* de los datos y las *etiquetas* predichas deseadas. La exploración de los datos se suele realizar mediante lenguajes de programación como Python o R que admiten el modelo estadístico y la visualización de forma nativa. Los scripts que se usan para explorar los datos normalmente se hospedan en entornos especializados como Jupyter Notebooks. Estas herramientas permiten a los científicos de datos explorar los datos mediante programación mientras documentan y comparten la información que encuentran.

Servicios de Azure correspondientes:

- [Azure Notebooks](https://notebooks.azure.com/)
- [Azure Machine Learning Studio](/azure/machine-learning/studio/what-is-ml-studio)
- [Servicio Experimentación de Azure Machine Learning](/azure/machine-learning/preview/experimentation-service-configuration)
- [Data Science Virtual Machine](/azure/machine-learning/data-science-virtual-machine/overview)

## <a name="challenges"></a>Desafíos

- **Cumplimiento de las normas de privacidad de datos.** Debe ser cuidadoso a la hora de hacer que los datos personales estén disponibles para los usuarios para su posterior análisis y generación de informes en modo de autoservicio. Es probable que haya algunas consideraciones de cumplimiento normativo, debido a las directivas de la organización, e incluso problemas legales. 

- **Volumen de datos**. Aunque puede ser útil proporcionar a los usuarios acceso al origen de datos completo, esto puede dar lugar a operaciones de Excel o Power BI de ejecución prolongada, o a consultas de Spark SQL que utilizan muchos recursos de un clúster.

- **Conocimientos del usuario**. Los usuarios crean sus propias consultas y agregaciones para tomar decisiones empresariales fundamentadas. ¿Está seguro de que los usuarios tienen los conocimientos analíticos y de consultas necesarios para obtener resultados precisos?

- **Uso compartido de los resultados**. Puede haber algunas consideraciones de seguridad a tener en cuenta si los usuarios pueden crear y compartir informes o visualizaciones de datos.

## <a name="architecture"></a>Architecture

Aunque el objetivo de este escenario es permitir el análisis de datos interactivo, las tareas de limpieza de datos, muestreo y estructuración que implica la ciencia de datos, a menudo incluyen procesos de ejecución prolongada. Esto hace que una arquitectura de [procesamiento por lotes](../big-data/batch-processing.md) sea adecuada.

## <a name="technology-choices"></a>Opciones de tecnología

Se recomiendan las siguientes tecnologías para la exploración de datos interactiva en Azure.

### <a name="data-storage"></a>Almacenamiento de datos

- **Contenedores de blobs de Azure Storage** o **Azure Data Lake Store**. Los científicos de datos suelen trabajar con datos de origen sin procesar para asegurarse de que tienen acceso a todas las características, valores atípicos y errores posibles de los datos. En un escenario de macrodatos, estos datos normalmente tienen la forma de archivos en un almacén de datos.

Para más información, consulte [Almacenamiento de datos](../technology-choices/data-storage.md).

### <a name="batch-processing"></a>Procesamiento por lotes

- **R Server** o **Spark**. La mayoría de los científicos de datos usan lenguajes de programación con gran compatibilidad para paquetes matemáticos y estadísticos, como R o Python. A la hora de trabajar con grandes volúmenes de datos, puede reducir la latencia mediante el uso de plataformas que permiten a estos lenguajes usar un procesamiento distribuido. R Server se puede utilizar por sí solo o en combinación con Spark para escalar horizontalmente las funciones de procesamiento de R. Spark admite de forma nativa Python para funcionalidades similares de escalabilidad horizontal en ese lenguaje.
- **Hive**. Hive es una buena opción para transformar los datos mediante una semántica similar a SQL. Los usuarios pueden crear y cargar las tablas mediante instrucciones de HiveQL que son semánticamente parecidas a SQL.

Para más información, consulte [Procesamiento por lotes](../technology-choices/batch-processing.md).

### <a name="analytical-data-store"></a>Almacén de datos analíticos

- **Spark SQL**. Spark SQL es una API basada en Spark que admite la creación de tramas de datos y tablas que se pueden consultar mediante la sintaxis de SQL. Independientemente de si los archivos de datos que se analizan son archivos de código fuente sin procesar o nuevos archivos que se han limpiado y preparado mediante un proceso por lotes, los usuarios pueden definir tablas de Spark SQL en ellos para realizar consultas adicionales en un análisis. 
- **Hive**. Además de realizar un procesamiento por lotes de los datos sin procesar mediante Hive, puede crear una base de datos de Hive que contiene tablas y vistas basadas en las carpetas en las que se almacenan los datos, permitiendo las consultas interactivas para el análisis y la generación de informes. HDInsight incluye un tipo de clúster de Hive interactivo que usa el almacenamiento en caché en memoria para reducir los tiempos de respuesta de las consultas de Hive. Los usuarios que se sienten cómodos con una sintaxis parecida a SQL pueden usar Hive interactivo para explorar los datos.

Para más información, consulte [Almacenes de datos analíticos](../technology-choices/analytical-data-stores.md).

### <a name="analytics-and-reporting"></a>Análisis e informes

- **Jupyter**. Jupyter Notebooks proporciona una interfaz basada en el explorador para ejecutar código en lenguajes como R, Python o Scala. Cuando se usa R Server o Spark para procesar los datos por lotes, o al usar Spark SQL para definir un esquema de tablas para las consultas, Jupyter puede ser una buena opción para consultar los datos. Si usa Spark, puede usar la API de trama de datos de Spark o la API de Spark SQL, así como instrucciones de SQL insertadas, para consultar los datos y producir visualizaciones.
- **Clientes de Hive interactivo**. Si utiliza un clúster de Hive interactivo para consultar los datos, puede usar la vista de Hive en el panel de clústeres de Ambari, la herramienta de línea de comandos Beeline o cualquier herramienta basada en ODBC (mediante el controlador ODBC de Hive), como Microsoft Excel o Power BI.

Para más información, consulte [Tecnología de análisis de datos e informes](../technology-choices/analysis-visualizations-reporting.md).