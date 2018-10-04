---
title: Elección de una tecnología de análisis de datos e informes
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 05e33a3da0933036a604d2bc4cc5a20ae70fe772
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/28/2018
ms.locfileid: "47428325"
---
# <a name="choosing-a-data-analytics-technology-in-azure"></a>Elección de una tecnología de análisis de datos en Azure

El objetivo de la mayoría de soluciones de macrodatos consiste en proporcionar información sobre los datos a través de análisis e informes. Esto puede incluir los informes preconfigurados y las visualizaciones, o la exploración de datos interactivos. 

## <a name="what-are-your-options-when-choosing-a-data-analytics-technology"></a>¿Cuáles son las opciones a la hora de elegir una tecnología de análisis de datos?

Hay varias opciones de generación de análisis, visualizaciones e informes en Azure dependiendo de sus necesidades:

- [Power BI](/power-bi/)
- [Jupyter Notebooks](https://jupyter.readthedocs.io/en/latest/index.html)
- [Zeppelin Notebooks](https://zeppelin.apache.org/)
- [Microsoft Azure Notebooks](https://notebooks.azure.com/)

### <a name="power-bi"></a>Power BI

[Power BI](/power-bi/) es un conjunto de herramientas de análisis de negocios. Puede conectarse a cientos de orígenes de datos y se puede usar para el análisis ad-hoc. Consulte [esta lista](/power-bi/desktop-data-sources) de los orígenes de datos disponibles actualmente. Use [Power BI Embedded](https://azure.microsoft.com/services/power-bi-embedded/) para integrar Power BI dentro de sus propias aplicaciones sin necesidad de ninguna licencia adicional.

Las organizaciones pueden usar Power BI para generar informes y publicarlos en la organización. Todos los usuarios pueden crear paneles personalizados, con regulación y [seguridad integradas](/power-bi/service-admin-power-bi-security). Power BI usa [Azure Active Directory](/azure/active-directory/) (Azure AD) para autenticar a los usuarios que inician sesión en el servicio Power BI, y utiliza las credenciales de inicio de sesión de Power BI cada vez que un usuario intenta acceder a los recursos que requieren autenticación.

### <a name="jupyter-notebooks"></a>Jupyter Notebooks 

[Jupyter Notebooks](https://jupyter.readthedocs.io/en/latest/index.html) proporciona un shell basado en el explorador que permite a los científicos de datos crear archivos de *cuaderno* que contienen código Python, Scala o R, y texto Markdown, lo cual lo convierte en una herramienta eficaz de colaboración a través del uso compartido y la documentación del código y los resultados en un único documento.

La mayoría de las variedades de clústeres de HDInsight, como Spark o Hadoop, ya vienen [preconfiguradas con Jupyter Notebooks](/azure/hdinsight/spark/apache-spark-jupyter-notebook-kernels) para interactuar con datos y enviar trabajos para su procesamiento. Según el tipo de clúster de HDInsight que use, se proporcionarán uno o varios kernels para interpretar y ejecutar el código. Por ejemplo, los clústeres de Spark en HDInsight proporcionan kernels relacionados con Spark entre los que puede seleccionar para ejecutar código Python o Scala con el motor de Spark.

Jupyter Notebooks proporciona un entorno estupendo para analizar, visualizar y procesar los datos antes de generar visualizaciones más avanzadas con una herramienta de inteligencia empresarial o de informes como Power BI.

### <a name="zeppelin-notebooks"></a>Zeppelin Notebooks

[Zeppelin Notebooks](https://zeppelin.apache.org/) es otra opción de un shell basado en el explorador parecida a la funcionalidad de Jupyter. Algunos clústeres de HDInsight vienen [preconfigurados con Zeppelin Notebooks](/azure/hdinsight/spark/apache-spark-zeppelin-notebook). Sin embargo, si usa un clúster de [HDInsight Interactive Query](/azure/hdinsight/interactive-query/apache-interactive-query-get-started) (Hive LLAP), [Zeppelin](/azure/hdinsight/hdinsight-connect-hive-zeppelin) es la única opción de cuaderno que puede usar actualmente para ejecutar consultas interactivas de Hive. Además, si usa un [clúster de HDInsight unido a un dominio](/azure/hdinsight/domain-joined/apache-domain-joined-introduction), Zeppelin Notebooks es el único tipo que le permite asignar inicios de sesión de usuario diferentes para controlar el acceso a los cuadernos y a las tablas subyacentes de Hive.

### <a name="microsoft-azure-notebooks"></a>Microsoft Azure Notebooks

[Azure Notebooks](https://notebooks.azure.com/) es un servicio en línea basado en Jupyter Notebooks que permite a los científicos de datos crear, ejecutar y compartir instancias de Jupyter Notebooks en bibliotecas basadas en la nube. Azure Notebooks proporciona entornos de ejecución para Python 2, Python 3, F# y R y proporciona varias bibliotecas de gráficos para visualizar los datos como ggplot, matplotlib, bokeh y seaborn.

A diferencia de las instancias de Jupyter Notebooks que se ejecutan en un clúster de HDInsight y se conectan a la cuenta de almacenamiento predeterminada del clúster, Azure Notebooks no proporciona ningún dato. Puede [cargar datos](https://notebooks.azure.com/Microsoft/libraries/samples/html/Getting%20to%20your%20Data%20in%20Azure%20Notebooks.ipynb) de varias formas como, por ejemplo, descargar los datos de un origen en línea, interactuar con Azure Blobs o Table Storage, conectarse con SQL Database o cargar datos con el asistente para copia de Azure Data Factory.

Ventajas principales:

* Servicio gratis: no se necesita una suscripción de Azure.
* No es necesario instalar Jupyter ni las distribuciones compatibles de R o Python de forma local. Simplemente use un explorador.
* Administre sus propias bibliotecas en línea y acceda a ellas desde cualquier dispositivo.
* Comparta los cuadernos con colaboradores.

Consideraciones:

* No podrá acceder a los cuadernos cuando esté sin conexión.
* Las funcionalidades de procesamiento limitadas del servicio gratuito de cuadernos puede que no sean suficientes para entrenar modelos grandes o complejos.

## <a name="key-selection-criteria"></a>Principales criterios de selección

Para restringir las opciones, empiece por responder a estas preguntas:

- ¿Necesita conectarse a varios orígenes de datos y proporcionar un lugar centralizado para crear informes de datos propagados por todo el dominio? Si es así, elija una opción que le permita conectarse a 100s de orígenes de datos.

- ¿Desea insertar visualizaciones dinámicas en un sitio web o aplicación externos? Si es así, elija una opción que proporcione funcionalidades de inserción.

- ¿Quiere diseñar las visualizaciones y los informes cuando está sin conexión? En caso afirmativo, elija una opción que disponga de funcionalidades sin conexión.

- ¿Necesita una gran capacidad de procesamiento para entrenar modelos de inteligencia artificial grandes o complejos o trabajar con conjuntos de datos muy grandes? En caso afirmativo, elija una opción en la que pueda conectarse a un clúster de macrodatos.

## <a name="capability-matrix"></a>Matriz de funcionalidades

En las tablas siguientes se resumen las diferencias clave en cuanto a funcionalidades. 

### <a name="general-capabilities"></a>Funcionalidades generales

| | Power BI | Jupyter Notebooks | Zeppelin Notebooks | Microsoft Azure Notebooks |
| --- | --- | --- | --- | --- |
| Conexión a clúster de macrodatos para procesamiento avanzado | SÍ | Sí | SÍ | Sin  |
| Servicio administrado | SÍ | Sí <sup>1</sup> | Sí <sup>1</sup> | SÍ |
| Conexión a 100s de orígenes de datos | SÍ | No | No | Sin  |
| Funcionalidades sin conexión | Sí <sup>2</sup> | Sin  | No | Sin  |
| Funcionalidades de inserción | SÍ | No | No | Sin  |
| Actualización de datos automática | SÍ | No | No | Sin  |
| Acceso a numerosos paquetes de código abierto | Sin  | Sí <sup>3</sup> | Sí <sup>3</sup> | Sí <sup>4</sup> |
| Opciones de transformación y limpieza de datos | [Power Query](https://powerbi.microsoft.com/blog/getting-started-with-power-query-part-i/), R | 40 lenguajes, incluidos Python, R, Julia y Scala | Más de 20 intérpretes, incluidos Python, JDBC y R | Python, F#, R |
| Precios | Es gratis para Power BI Desktop (creación), consulte los [precios](https://powerbi.microsoft.com/pricing/) de las opciones de hospedaje | Gratuito | Gratuito | Gratuito |
| Colaboración multiusuario | [Sí](/power-bi/service-how-to-collaborate-distribute-dashboards-reports) | Sí (mediante el uso compartido o con un servidor multiusuario como [JupyterHub](https://github.com/jupyterhub/jupyterhub)) | SÍ | Sí (mediante el uso compartido) |

[1] Cuando se utiliza como parte de un clúster de HDInsight administrado.

[2] Con el uso de Power BI Desktop.

[2] Puede buscar el [repositorio Maven](https://search.maven.org/) para obtener paquetes en los que contribuyó la comunidad.

[3] Los paquetes de Python se pueden instalar a través de pip o conda. Los paquetes de R se pueden instalar desde CRAN o GitHub. Los paquetes en F # se pueden instalar a través de nuget.org mediante el [administrador de dependencias Paket](https://fsprojects.github.io/Paket/).

