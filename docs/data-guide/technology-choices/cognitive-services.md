---
title: "Elección de una tecnología de servicios cognitivos"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: d97e166abed4670e4bdc797cc8075be3314e677a
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/14/2018
---
# <a name="choosing-a-microsoft-cognitive-services-technology"></a>Elección de una tecnología de servicios cognitivos de Microsoft

Los servicios cognitivos de Microsoft son API basadas en la nube que puede usar en aplicaciones de inteligencia artificial (AI) y flujos de datos. Proporcionan modelos previamente entrenados que están listos para usarlos en cualquier aplicación, no requieren datos ni entrenamiento del modelo por su parte. Los servicios cognitivos los desarrolla el equipo de inteligencia artificial e investigación de Microsoft y aprovechan los algoritmos de aprendizaje profundo más recientes. Se consumen a través de las interfaces de REST de HTTP. Además, hay disponibles SDK para muchos marcos de desarrollo de aplicaciones comunes.

Los servicios cognitivos incluyen:

* Análisis de texto
* Visión del equipo
* Análisis de vídeo
* Reconocimiento y generación de voz
* Comprensión del lenguaje natural
* Búsqueda inteligente

Ventajas principales:

* Con un esfuerzo mínimo de desarrollo se logran servicios de AI de última generación.
* Fácil integración en aplicaciones a través de interfaces de REST de HTTP.
* Compatibilidad integrada para consumir servicios cognitivos en Azure Data Lake Analytics.

Consideraciones:

* Solo está disponible a través de Internet. Por lo general se requiere conectividad a Internet. Una excepción es Custom Vision Service, cuyo modelo entrenado se puede exportar para la predicción en dispositivos y en el borde de IoT.
* Aunque se admite una personalización considerable, es posible que los servicios disponibles no se ajusten a todos los requisitos de análisis predictivo.

## <a name="what-are-your-options-when-choosing-amongst-the-cognitive-services"></a>¿Qué opciones tiene al elegir entre los servicios cognitivos?
En Azure, existen docenas de servicios cognitivos disponibles. La lista actual de ellos está disponible en un directorio y están clasificados por el área funcional que admiten:
- [Visión](https://azure.microsoft.com/services/cognitive-services/directory/vision/)
- [Voz](https://azure.microsoft.com/services/cognitive-services/directory/speech/)
- [Conocimiento](https://azure.microsoft.com/services/cognitive-services/directory/know/)
- [Búsqueda](https://azure.microsoft.com/services/cognitive-services/directory/search/)
- [Lenguaje](https://azure.microsoft.com/services/cognitive-services/directory/lang/)

## <a name="key-selection-criteria"></a>Principales criterios de selección

Para restringir las opciones, empiece por responder a estas preguntas:

- ¿Con qué tipo de datos trata? Puede limitar las opciones en función del tipo de datos de entrada con el que trabaje. Por ejemplo, si la entrada es texto, seleccione uno de los servicios que tenga un tipo de entrada de texto. 

- ¿Tiene los datos necesarios para entrenar un modelo? En caso afirmativo, considere los servicios personalizados que le permiten entrenar sus modelos subyacentes con los datos que proporcione, con el fin de mejorar la precisión y el rendimiento. 

## <a name="capability-matrix"></a>Matriz de funcionalidades

En las tablas siguientes se resumen las diferencias clave en cuanto a funcionalidades. 

### <a name="uses-prebuilt-models"></a>Usa modelos creados previamente
| | Tipo de entrada | Principal ventaja |
| --- | --- | --- |
| Text Analytics API | Texto | Evalúe las opiniones y temas para comprender lo que los usuarios quieren. |
| Entity Linking API| Texto | Alimente los vínculos de datos de su aplicación con el reconocimiento y la anulación de ambigüedades de las entidades con nombre. |
| Language Understanding Intelligent Service (LUIS)| Texto | Enseñe a las aplicaciones a entender los comandos de los usuarios. |
| Servicio QnA Maker| Texto | Convierta la información con formato de preguntas frecuentes en respuestas conversacionales por las que sea fácil navegar. |
| Linguistic Analysis API | Texto | Simplifique conceptos complejos del lenguaje y analice el texto. |
| Servicio Knowledge Exploration | Texto | Permita experiencias de búsqueda interactiva a través de datos estructurados mediante entradas en lenguaje natural. | 
| Web Language Model API | Texto | Use modelos de lenguaje predictivos entrenados con datos de escala web. | 
| Academic Knowledge API | Texto | Descubra la riqueza del contenido académico de Microsoft Academic Graph rellenado por Bing. |
| Bing Autosuggest API | Texto | Proporcione a su aplicación opciones de sugerencias automáticas inteligentes para las búsquedas. |
| Bing Spell Check API | Texto | Detecte y corrija errores ortográficos en las aplicaciones. |
| Translator Text API | Texto | Servicio de traducción automática. |
| Recommendations API | Texto | Prediga y recomiende los artículos que sus clientes quieren. |
| Bing Entity Search API | Texto (consulta de búsqueda web) | Identifique y amplíe la información de las entidades desde la web. |
| Bing Image Search API | Texto (consulta de búsqueda web) | Busque imágenes. |
| Bing News Search API | Texto (consulta de búsqueda web) | Busque noticias. |
| Bing Video Search API | Texto (consulta de búsqueda web) | Busque vídeos. |
| Bing Web Search API | Texto (consulta de búsqueda web) | Obtenga detalles mejorados de la búsqueda de miles de millones de documentos web. |.
| Bing Speech API | Texto o voz | Convierta voz en texto, y viceversa. |
| Speaker Recognition API | Voz | Use la voz para identificar y autenticar a hablantes individuales. |
| Translator Speech API | Voz | Realice traducciones de voz en tiempo real. |
| Computer Vision API | Imágenes (o fotogramas de vídeo) | Convierta la información accionable de las imágenes, cree automáticamente la descripción de las fotos, derive etiquetas, reconozca a celebridades, extraiga texto y cree miniaturas precisas. |
| Content Moderator | Texto, imágenes o vídeo | Moderación automatizada de imágenes, texto y vídeo. |
| Emotion API | Imágenes (fotos con seres humanos) | Identifique la gama de emociones de los seres humanos. |
| Face API | Imágenes (fotos con seres humanos) | Detecte, identifique, analice, organice y etiquete las caras en las fotos. |
| Indexador de vídeo | Vídeo | Información acerca del vídeo, como la opinión, transcriba la voz, traduzca la voz, reconozca caras y emociones, y extraiga las palabras clave. | 

### <a name="trained-with-custom-data-you-provide"></a>Entrenado con los datos personalizados que proporcione
| | Tipo de entrada | Principal ventaja |
| --- | --- | --- |
| Servicio de visión personalizada | Imágenes (o fotogramas de vídeo) | Personalice sus propios modelos de visión del equipo. |
| Servicio de voz personalizado | Voz | Elimine las barreras del reconocimiento de voz, como el estilo en que se habla, el ruido de fondo y el vocabulario. | 
| Servicio de decisión personalizado | Contenido web (por ejemplo, fuente RSS) | Use el aprendizaje automático para seleccionar automáticamente el contenido adecuado para la página principal |
| Bing Custom Search API | Texto (consulta de búsqueda web) | Herramienta de búsqueda comercial. |

