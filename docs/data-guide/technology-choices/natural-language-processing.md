---
title: "Elección de una tecnología de procesamiento de lenguaje natural"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: dacf7bf9cf3e9efed212f34da93c1470954965cf
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/14/2018
---
# <a name="choosing-a-natural-language-processing-technology-in-azure"></a>Elección de una tecnología de procesamiento de lenguaje natural en Azure

El procesamiento de texto de forma libre se realiza en documentos que contienen párrafos de texto, normalmente con el propósito de ayudar a las búsquedas, pero también se usa para realizar otras tareas de procesamiento de lenguaje natural (NLP) como, por ejemplo, análisis de opiniones, detección de temas, detección de idioma, extracción de frases clave y clasificación de documentos. Este artículo se centra en las opciones de tecnología que actúan en apoyo de las tareas de procesamiento del lenguaje natural (NLP).

## <a name="what-are-your-options-when-choosing-an-nlp-service"></a>¿Cuáles son las opciones al elegir un servicio NLP?

En Azure, los servicios siguientes proporcionan funcionalidades de procesamiento de lenguaje natural (NLP):

- [Azure HDInsight con Spark y Spark MLlib](/azure/hdinsight/spark/apache-spark-overview)
- [Microsoft Cognitive Services](/azure/#pivot=products&panel=cognitive)

## <a name="key-selection-criteria"></a>Principales criterios de selección

Para restringir las opciones, empiece por responder a estas preguntas:

- ¿Desea usar modelos creados previamente? Si es así, considere la posibilidad de usar las API que ofrece Microsoft Cognitive Services.

- ¿Necesita entrenar modelos personalizados en un corpus grande de datos de texto? Si es así, considere la posibilidad de usar Azure HDInsight con Spark MLlib y Spark NLP.

- ¿Necesita funcionalidades de procesamiento de lenguaje natural de bajo nivel como tokenización, lematización y frecuencia de términos o frecuencia inversa de documento (TF/IDF)? Si es así, considere la posibilidad de usar Azure HDInsight con Spark MLlib y Spark NLP.

- ¿Necesita capacidades de procesamiento de lenguaje natural simples y de alto nivel como identificación de entidades e intenciones, detección de temas, corrector ortográfico o análisis de opiniones? Si es así, considere la posibilidad de usar las API que ofrece Microsoft Cognitive Services.

## <a name="capability-matrix"></a>Matriz de funcionalidades

En las tablas siguientes se resumen las diferencias clave en cuanto a funcionalidades.  

### <a name="general-capabilities"></a>Funcionalidades generales

| | HDInsight de Azure | Microsoft Cognitive Services |
| --- | --- | --- |
| Proporciona modelos previamente entrenados como un servicio | Sin  | Sí |
| API DE REST | Sí | Sí |
| Capacidad de programación | Python, Scala, Java | C#, Java, Node.js, Python, PHP, Ruby |
| Admite el procesamiento de macrodatos y documentos de gran tamaño | Sí | Sin  |

### <a name="low-level-natural-language-processing-capabilities"></a>Funcionalidades de procesamiento de lenguaje natural de bajo nivel

| | HDInsight de Azure | Microsoft Cognitive Services |  
| --- | --- | --- | 
| Tokenizador | Sí (Spark NLP) | Sí (Linguistic Analysis API) |
| Lematizador | Sí (Spark NLP) | Sin  |
| Lematizador | Sí (Spark NLP) | Sin  |
| Etiquetado de categorías gramaticales | Sí (Spark NLP) | Sí (Linguistic Analysis API) |
| Frecuencia de términos o frecuencia inversa de documento (TF/IDF) | Sí (Spark MLlib) | Sin  |
| Similitud de cadenas: edición del cálculo de distancias | Sí (Spark MLlib) | Sin  |
| Cálculo de n-gramas | Sí (Spark MLlib) | Sin  |
| Detención de la eliminación de palabras | Sí (Spark MLlib) | Sin  |

### <a name="high-level-natural-language-processing-capabilities"></a>Funcionalidades de procesamiento de lenguaje natural de alto nivel

| | HDInsight de Azure | Microsoft Cognitive Services |
| --- | --- | --- | 
| Identificación y extracción de entidades o intenciones | Sin  | Sí (Language Understanding Intelligent Service (LUIS) API) |    
| Detección de temas | Sí (Spark NLP) | Sí (Text Analytics API) |
| Corrector ortográfico | Sí (Spark NLP) | Sí (Bing Spell Check API) |
| análisis de opiniones | Sí (Spark NLP) | Sí (Text Analytics API) |
| Detección de idiomas | Sin  | Sí (Text Analytics API) |
| Admite varios idiomas además del inglés | Sin  | Sí (varía según la API) |

## <a name="see-also"></a>Otras referencias

[Procesamiento de lenguaje natural](../scenarios/natural-language-processing.md)