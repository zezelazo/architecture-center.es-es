---
title: "Análisis avanzado"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 31ba357fe37b1de35a6eea324d2d1d6766e172e5
ms.sourcegitcommit: 29fbcb1eec44802d2c01b6d3bcf7d7bd0bae65fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/27/2018
---
# <a name="advanced-analytics"></a>Análisis avanzado

El análisis avanzado va más allá de los informes históricos y la agregación de los datos tradicionales de inteligencia empresarial (BI), y usa técnicas de modelado matemático, probabilístico y estadístico que permiten el procesamiento predictivo y la toma de decisiones automatizadas.

Las soluciones de análisis avanzado normalmente conllevan las siguientes cargas de trabajo:

* Exploración y visualización de datos interactivos
* Entrenamientos de modelos de Machine Learning
* Procesamiento en tiempo real o predictivo por lotes

La mayoría de las arquitecturas de análisis avanzado incluyen algunos de los componentes siguientes (o todos ellos):

* **Almacenamiento de datos**. Las soluciones de análisis avanzado requieren datos para entrenar modelos de aprendizaje automático. Los científicos de datos normalmente tienen que explorar los datos para identificar sus características predictivas y las relaciones estadísticas entre ellos y los valores que predicen (llamados etiquetas). La etiqueta predicha puede ser un valor cuantitativo, como el valor financiero de algo en el futuro o la duración en minutos del retraso de un vuelo. O puede representar una clase categórica, como "true" o "false", "retraso de vuelo" o "ningún retraso de vuelo", o categorías como "bajo riesgo", "riesgo medio" o "alto riesgo".

* **Procesamiento por lotes**. Para entrenar un modelo de aprendizaje automático, normalmente tiene que procesar un gran volumen de datos de entrenamiento. El entrenamiento del modelo puede tardar algún tiempo (desde minutos a horas). Este entrenamiento se puede realizar mediante scripts escritos en lenguajes como Python o R, y se puede escalar horizontalmente para reducir el tiempo de entrenamiento con plataformas de procesamiento distribuido como Apache Spark hospedado en HDInsight o en un contenedor de Docker.

* **Ingesta de mensajes en tiempo real**. En producción, muchos análisis avanzados alimentan flujos de datos en tiempo real para un modelo predictivo que se ha publicado como un servicio web. Normalmente, el flujo de datos de entrada se captura en algún tipo de cola y un motor de procesamiento de flujos extrae los datos de esta cola y aplica la predicción a los datos de entrada casi en tiempo real.  

* **Procesamiento de flujos**. Una vez que tiene un modelo entrenado, la predicción (o puntuación) suele ser una operación muy rápida (del orden de milisegundos) para un determinado conjunto de características. Después de capturar los mensajes en tiempo real, los valores de las características pertinentes pueden pasarse al servicio de predicción para generar una etiqueta predicha.

* **Almacén de datos analíticos**. En algunos casos, los valores de las etiquetas predichas se escriben en el almacén de datos analíticos para la generación de informes y su análisis posterior.

* **Análisis e informes**. Como sugiere su nombre, las soluciones de análisis avanzado suelen producir algún tipo de informe o fuente analítica que incluye los valores de los datos predichos. A menudo, los valores de las etiquetas predichas se utilizan para rellenar los paneles en tiempo real.

* **Orquestación**. Aunque los científicos de datos realizan la exploración y modelado iniciales de los datos de forma interactiva, muchas soluciones de análisis avanzado vuelven a entrenar periódicamente los modelos con nuevos datos, lo cual contribuye continuamente a refinar la precisión de los modelos. Este reentrenamiento se puede automatizar mediante un flujo de trabajo orquestado.

## <a name="machine-learning"></a>Machine Learning
El aprendizaje automático es una técnica de modelado matemático utilizada para entrenar un modelo predictivo. El principio general consiste en aplicar un algoritmo estadístico a un conjunto de datos grande de datos históricos para descubrir las relaciones entre los campos que contiene.

Normalmente, el modelado del entrenamiento automático lo realizan los científicos de datos que tienen que explorar y preparar cuidadosamente los datos antes de entrenar un modelo. Esta preparación y exploración normalmente implica una gran cantidad de análisis y visualización de datos interactivos, usualmente mediante lenguajes como Python y R en herramientas y entornos interactivos que están especialmente diseñados para esta tarea.

En algunos casos, puede usar los [modelos previamente aprendidos](/machine-learning-server/install/microsoftml-install-pretrained-models) que vienen con los datos de entrenamiento que Microsoft obtiene y desarrolla. La ventaja de los modelos previamente aprendidos es que puede puntuar y clasificar el nuevo contenido inmediatamente incluso si no tiene los datos de entrenamiento necesarios, los recursos para administrar grandes conjuntos de datos o para entrenar modelos complejos.

Hay dos grandes categorías de aprendizaje automático:

* **Aprendizaje supervisado**. El aprendizaje supervisado es el enfoque más común utilizado por el aprendizaje automático. En un modelo de aprendizaje supervisado, el origen de datos consta de un conjunto de campos de datos de *características* que tienen una relación matemática con uno o varios campos de datos de *etiquetas*. Durante la fase de entrenamiento del proceso de aprendizaje automático, el conjunto de datos incluye las características y las etiquetas conocidas, y se aplica un algoritmo para ajustarse a una función que actúa sobre las características para calcular las predicciones correspondientes de etiquetas. Normalmente, se retiene un subconjunto del conjunto de datos de entrenamiento y se usa para validar el rendimiento del modelo entrenado. Una vez entrenado el modelo, se puede implementar en producción y usar para predecir valores desconocidos. 

* **Aprendizaje sin supervisión**. En un modelo de entrenamiento sin supervisión, los datos de entrenamiento no incluyen valores de etiqueta conocidos. En su lugar, el algoritmo realiza sus predicciones basándose en su primera exposición a los datos. La forma más habitual de aprendizaje sin supervisión es la *agrupación en clústeres*, donde el algoritmo determina la mejor manera de dividir los datos entre un número especificado de clústeres basándose en similitudes estadísticas de las características. En la agrupación en clústeres, el resultado predicho es el número de clúster al que pertenecen las características de entrada. Aunque a veces se pueden utilizar directamente para generar predicciones útiles como, por ejemplo, usar la agrupación en clústeres para identificar grupos de usuarios en una base de datos de clientes, los enfoques de entrenamiento sin supervisión se usan más a menudo para identificar qué datos son más útiles para proporcionarlos a un algoritmo de entrenamiento supervisado de un modelo de entrenamiento.

Servicios de Azure correspondientes:

- [Azure Machine Learning](/azure/machine-learning/)
- [Machine Learning Server (R Server) en HDInsight](/azure/hdinsight/r-server/r-server-overview)

## <a name="deep-learning"></a>Aprendizaje profundo

Los modelos de aprendizaje automático basados en técnicas matemáticas como la regresión lineal o logística han estado disponibles durante algún tiempo. Más recientemente, el uso de técnicas de *aprendizaje profundo* basadas en redes neuronales ha aumentado. Esto se controla en parte mediante la disponibilidad de sistemas de procesamiento altamente escalables que reducen el tiempo necesario para entrenar modelos complejos. Además, la mayor prevalencia de los macrodatos hace que sea más fácil entrenar modelos de aprendizaje profundo en diversos dominios.

Al diseñar una arquitectura en la nube para procesos de análisis avanzado, debe tener en cuenta la necesidad de procesamiento a gran escala de los modelos de aprendizaje profundo. Estos se pueden proporcionar a través de plataformas de procesamiento distribuido como Apache Spark y la última generación de máquinas virtuales que incluyen el acceso a hardware de GPU.

Servicios de Azure correspondientes:

- [Deep Learning Virtual Machine](/azure/machine-learning/data-science-virtual-machine/deep-learning-dsvm-overview)
- [Apache Spark en HDInsight](/azure/hdinsight/spark/apache-spark-overview)

## <a name="artificial-intelligence"></a>Inteligencia artificial

La inteligencia artificial (AI) hace referencia a los escenarios en los que una máquina imita las funciones cognitivas asociadas a las mentes humanas como, por ejemplo, el aprendizaje y la solución de problemas. Como la inteligencia artificial se sirve de algoritmos de aprendizaje automático, se considera un término genérico. La mayoría de soluciones de inteligencia artificial se basan en una combinación de servicios predictivos, a menudo implementados como servicios web, y de interfaces de lenguaje natural, por ejemplo los bots de chat que permiten interactuar mediante texto o voz, que las aplicaciones de inteligencia artificial presentan en ejecución en dispositivos móviles u otros clientes. En algunos casos, el modelo de aprendizaje automático se inserta en la aplicación de inteligencia artificial. 

## <a name="model-deployment"></a>Implementación del modelo

Los servicios de predicción que admiten las aplicaciones de inteligencia artificial pueden aprovechar los modelos de aprendizaje automático personalizados o los servicios cognitivos existentes que proporcionan acceso a los modelos previamente entrenados. El proceso de implementación de modelos personalizados en producción se conoce como operacionalización. Este es un proceso en el que los mismos modelos de inteligencia artificial que se entrenaron y probaron en el entorno de procesamiento se serializan y se ponen a disposición de las aplicaciones y servicios externos para la realización de predicciones por lotes o de autoservicio. Para usar la funcionalidad predictiva del modelo, este se deserializa y se carga mediante la misma biblioteca de aprendizaje automático que contiene el algoritmo que se usó para entrenar el modelo en primer lugar. Esta biblioteca proporciona funciones predictivas (a menudo denominadas puntuación o predicción) que usan el modelo y las características como entrada y devuelven la predicción. Esta lógica se encapsula en una función a la que una aplicación puede llamar directamente o que se puede exponer como servicio web. 

Servicios de Azure correspondientes:

- [Azure Machine Learning](/azure/machine-learning/)
- [Machine Learning Server (R Server) en HDInsight](/azure/hdinsight/r-server/r-server-overview)


## <a name="see-also"></a>Otras referencias

- [Elección de una tecnología de servicios cognitivos](../technology-choices/cognitive-services.md)
- [Elección de una tecnología de aprendizaje automático](../technology-choices/data-science-and-machine-learning.md)
