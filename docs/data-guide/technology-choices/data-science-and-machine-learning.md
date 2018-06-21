---
title: Elección de una tecnología de aprendizaje automático
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 995349c795066ec3067b20ad2615e40b0fb152db
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/14/2018
ms.locfileid: "29288937"
---
# <a name="choosing-a-machine-learning-technology-in-azure"></a>Elección de una tecnología de aprendizaje automático en Azure

La ciencia de datos y el aprendizaje automático constituyen una carga de trabajo que normalmente llevan a cabo los científicos de datos. Requiere herramientas especializadas, muchas de las cuales están diseñadas específicamente para el tipo de exploración interactiva de datos y las tareas de modelado que debe realizar un científico de datos.

Las soluciones de aprendizaje automático se compilan de forma iterativa y tienen dos fases distintas:
* Preparación y modelado de datos.
* Implementación y consumo de servicios de predicción.

## <a name="tools-and-services-for-data-preparation-and-modeling"></a>Herramientas y servicios para la preparación y el modelado de los datos
Los científicos de datos prefieren trabajar normalmente con datos mediante código personalizado escrito en Python o R. Generalmente, este código se ejecuta de forma interactiva y los científicos de datos lo usan para consultar y explorar los datos, y generar visualizaciones y estadísticas que ayuden a determinar las relaciones entre ellos. Hay muchos entornos interactivos para R y Python que pueden usar los científicos de datos. Uno de los favoritos es **Jupyter Notebooks** que proporciona un shell basado en el explorador que permite a los científicos de datos crear archivos de *cuaderno* que contienen código R o Python y texto Markdown. Esta es una manera eficaz de colaborar mediante el uso compartido y la documentación del código y los resultados en un solo documento.

Otras herramientas utilizadas habitualmente son:
* **Spyder**: el entorno de desarrollo interactivo (IDE) para Python que se proporciona con Anaconda Python Distribution.
* **R Studio**: un IDE para el lenguaje de programación R.
* **Visual Studio Code**: un entorno de codificación ligero, multiplataforma, que admite Python así como otras plataformas usadas habitualmente para el desarrollo de aprendizaje automático e inteligencia artificial.

Además de estas herramientas, los científicos de datos pueden aprovechar los servicios de Azure para simplificar la administración del código y de los modelos.

### <a name="azure-notebooks"></a>Azure Notebooks
Azure Notebooks es un servicio en línea de Jupyter Notebooks que permite a los científicos de datos crear, ejecutar y compartir instancias de Jupyter Notebooks en bibliotecas basadas en la nube.

Ventajas principales:

* Servicio gratis: no se necesita una suscripción de Azure.
* No es necesario instalar Jupyter ni las distribuciones compatibles de R o Python de forma local. Simplemente use un explorador.
* Administre sus propias bibliotecas en línea y acceda a ellas desde cualquier dispositivo.
* Comparta los cuadernos con colaboradores.

Consideraciones:

* No podrá acceder a los cuadernos cuando esté sin conexión.
* Las funcionalidades de procesamiento limitadas del servicio gratuito de cuadernos puede que no sean suficientes para entrenar modelos grandes o complejos.

### <a name="data-science-virtual-machine"></a>Data Science Virtual Machine
La máquina virtual de ciencia de datos es una imagen de máquina virtual de Azure que incluye las herramientas y plataformas usadas habitualmente por los científicos de datos como R, Python, Jupyter Notebooks, Visual Studio Code, y bibliotecas para el modelado de aprendizaje automático como Microsoft Cognitive Toolkit. Estas herramientas pueden ser complejas y lentas a la hora de instalarse, y contener muchas interdependencias, lo cual suele dar lugar a problemas de administración de versiones. Tener una imagen preinstalada puede reducir el tiempo que los científicos de datos emplean en solucionar los problemas del entorno, lo que les permite centrarse en las tareas de exploración y modelado de los datos que deben llevar a cabo.

Ventajas principales:
* Menor tiempo de instalación, administración y solución de problemas de las herramientas y plataformas de ciencia de datos.
* Se incluyen las versiones más recientes de todas las herramientas y plataformas usadas habitualmente.
* Las opciones de máquina virtual incluyen imágenes altamente escalables con funcionalidades GPU para el modelado de datos intensivo.

Consideraciones:
* No se puede acceder a la máquina virtual sin conexión.
* Si ejecuta una máquina virtual incurrirá en gastos de Azure, por lo que solo deberá ejecutarla en caso de necesidad.

### <a name="azure-machine-learning"></a>Azure Machine Learning

Azure Machine Learning es un servicio basado en la nube que se usa para administrar experimentos y modelos de aprendizaje automático. Incluye un servicio de experimentación que permite realizar un seguimiento de los scripts de preparación y modelado de datos, y conservar un historial de todas las ejecuciones para que pueda comparar el rendimiento del modelo entre iteraciones. Una herramienta de cliente multiplataforma denominada Azure Machine Learning Workbench proporciona una interfaz central para la administración de scripts y para el historial, al tiempo que permite a los científicos de datos crear scripts en la herramienta de su elección como, por ejemplo, en Jupyter Notebooks o Visual Studio Code.

En Azure Machine Learning Workbench, puede usar las herramientas de preparación de datos interactivos para simplificar tareas habituales de transformación de datos y puede configurar el entorno de ejecución del script para ejecutar scripts de entrenamiento de modelos de forma local, en un contenedor escalable de Docker, o en Spark.

Cuando esté listo para implementar el modelo, use el entorno de Workbench para empaquetar el modelo e implementarlo como un servicio web en un contenedor de Docker, Spark en Azure HDinsight, Microsoft Machine Learning Server o SQL Server. El servicio Administración de modelos de Azure Machine Learning, le permite realizar el seguimiento y administración de las implementaciones de modelos en la nube, en dispositivos perimetrales o en toda la empresa.

Ventajas principales:

* La administración centralizada de scripts y el historial de ejecución que facilitan la comparación entre las versiones de un modelo.
* Transformación de datos interactivos mediante un editor visual.
* Fácil implementación y administración de modelos en la nube o en dispositivos perimetrales.

Consideraciones:
* Se necesita cierta familiaridad con el modelo de administración de modelos y el entorno de la herramienta Workbench.

### <a name="azure-batch-ai"></a>Inteligencia artificial de Azure Batch

Azure Batch AI le permite ejecutar sus experimentos de aprendizaje automático en paralelo y realizar el entrenamiento del modelo a escala en un clúster de máquinas virtuales con GPU. El entrenamiento de Batch AI le permite escalar horizontalmente trabajos de entrenamiento profundo a través de GPU en clúster, con plataformas como Cognitive Toolkit, Caffe, Chainer y TensorFlow. 

Administración de modelos de Azure Machine Learning puede usarse para tomar los modelos de entrenamiento de Batch AI para implementarlos, administrarlos y supervisarlos. 

### <a name="azure-machine-learning-studio"></a>Azure Machine Learning Studio

Azure Machine Learning Studio es un entorno de desarrollo visual, basado en la nube que se usa para crear experimentos de datos, entrenar modelos de aprendizaje automático y publicarlos como servicios web en Azure. Su interfaz visual, en la que se puede arrastrar y colocar, permite a los científicos de datos y a los usuarios avanzados crear rápidamente soluciones de aprendizaje automático, al tiempo que admite lógicas de R y Python, una amplia gama de técnicas y algoritmos estadísticos establecidos para las tareas de modelado de aprendizaje automático y soporte integrado para Jupyter Notebooks.

Ventajas principales:

* La interfaz visual interactiva permite el modelado del aprendizaje automático con un código mínimo.
* Instancias integradas de Jupyter Notebooks para la exploración de datos.
* Implementación directa de los modelos entrenados como servicios web de Azure.

Consideraciones:

* Escalabilidad limitada. El tamaño máximo de un conjunto de datos de entrenamiento es de 10 GB.
* Solo en línea. No hay entorno de desarrollo sin conexión.

## <a name="tools-and-services-for-deploying-machine-learning-models"></a>Herramientas y servicios para implementar modelos de aprendizaje automático

Una vez que un científico de datos ha creado un modelo de aprendizaje automático, normalmente necesita implementarlo y consumirlo desde aplicaciones o en otros flujos de datos. Hay una serie de posibles destinos de implementación de modelos de aprendizaje automático.

### <a name="spark-on-azure-hdinsight"></a>Spark en Azure HDInsight

Apache Spark incluye Spark MLlib, una plataforma y una biblioteca de modelos de aprendizaje automático. La biblioteca de Microsoft Machine Learning de Spark (MMLSpark) también proporciona compatibilidad de los algoritmos de aprendizaje profundo con los modelos predictivos de Spark.

Ventajas principales:

* Spark es una plataforma distribuida que ofrece una alta escalabilidad para un gran volumen de procesos de aprendizaje automático.
* Puede implementar modelos directamente en Spark en HDinsight desde Azure Machine Learning Workbench y administrarlos mediante el servicio Administración de modelos de Azure Machine Learning.

Consideraciones:

* Spark se ejecuta en un clúster de HDinsght que incurre en gastos todo el tiempo que permanece en ejecución. Si el servicio de aprendizaje automático solo se usa en ocasiones, esto puede provocar costos innecesarios.

### <a name="web-service-in-a-container"></a>Servicio web en un contenedor

Puede implementar un modelo de aprendizaje automático como un servicio web de Python en un contenedor de Docker. Puede implementar el modelo en Azure o en un dispositivo perimetral, donde se puede utilizar de forma local con los datos en los que trabaja.

Ventajas principales:

* Los contenedores son una forma sencilla, y generalmente rentable, de empaquetar e implementar servicios.
* La posibilidad de implementar en un dispositivo perimetral le permite mover la lógica predictiva más cerca de los datos.
* Puede implementar en un contenedor directamente desde Azure Machine Learning Workbench.

Consideraciones:

* Este modelo de implementación se basa en contenedores de Docker, por lo que debería familiarizarse con esta tecnología antes de implementar un servicio web de esta manera.

### <a name="microsoft-machine-learning-server"></a>Servidor de Microsoft Machine Learning

Machine Learning Server (anteriormente conocido como Microsoft R Server) es una plataforma escalable para código R y Python, diseñada específicamente para escenarios de aprendizaje automático.

Ventajas principales:

* Alta escalabilidad.
* Implementación directa desde Azure Machine Learning Workbench.

Consideraciones:

* Debe implementar y administrar Machine Learning Server en la empresa.

### <a name="microsoft-sql-server"></a>Microsoft SQL Server

Microsoft SQL Server admite R y Python de forma nativa, lo que le permite encapsular modelos de aprendizaje automático compilados en estos lenguajes como funciones de Transact-SQL en una base de datos.

Ventajas principales:

* Permite encapsular la lógica predictiva en una función de base de datos, lo cual hace que sea más fácil de incluir en una lógica de capa de datos.

Consideraciones:

* Presupone una base de datos de SQL Server como la capa de datos de la aplicación.

### <a name="azure-machine-learning-web-service"></a>Servicio web de Azure Machine Learning

Cuando crea un modelo de aprendizaje automático mediante Azure Machine Learning Studio, puede implementarlo como un servicio web. Posteriormente, este se puede usar a través de una interfaz de REST de cualquier aplicación cliente capaz de comunicarse por HTTP.

Ventajas principales:

* Facilidad de desarrollo y de implementación.
* Portal de administración de servicios web con métricas de supervisión básicas.
* Compatibilidad integrada para llamar a los servicios web de Azure Machine Learning desde Azure Data Lake Analytics, Azure Data Factory y Azure Stream Analytics.

Consideraciones:

* Solo está disponible en los modelos compilados mediante Azure Machine Learning Studio.
* Únicamente acceso basado en web, los modelos entrenados no se pueden ejecutar de forma local o sin conexión.

