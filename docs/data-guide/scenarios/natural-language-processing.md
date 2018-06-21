---
title: Procesamiento de lenguaje natural
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 0afd8ac9a8a2e56f79ade0b2e10328630866c03c
ms.sourcegitcommit: 51f49026ec46af0860de55f6c082490e46792794
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/03/2018
ms.locfileid: "30297978"
---
# <a name="natural-language-processing"></a>Procesamiento de lenguaje natural

El procesamiento de lenguaje natural (NLP) se utiliza para tareas como el análisis de opiniones, la detección de temas, la detección de idioma, la extracción de frases clave y la clasificación de documentos.

![](./images/nlp-pipeline.png)

## <a name="when-to-use-this-solution"></a>Cuándo se debe utilizar esta solución

El procesamiento de lenguaje natural se puede usar para clasificar documentos, por ejemplo, etiquetar documentos como confidenciales o como correo no deseado. La salida del procesamiento de lenguaje natural se puede usar para su procesamiento subsiguiente o para búsquedas. Otro uso del procesamiento de lenguaje natural es resumir texto mediante la identificación de las entidades presentes en el documento. Estas entidades también pueden utilizarse para etiquetar documentos con palabras clave, lo que permite la búsqueda y recuperación basadas en el contenido. Las entidades pueden combinarse en temas, con resúmenes que describen los temas importantes presentes en cada documento. Los temas detectados pueden utilizarse para clasificar los documentos para la navegación o para enumerar los documentos relacionados con un tema seleccionado. Otro uso del procesamiento de lenguaje natural es la puntuación del texto según la opinión, para evaluar el tono positivo o negativo de un documento. Estos enfoques utilizan muchas técnicas del procesamiento de lenguaje natural, como: 

- **Tokenizador**. Dividir el texto en palabras o frases.
- **Lematización**. Normalizar las palabras para asignar las distintas formas a la palabra canónica con el mismo significado. Por ejemplo, "corriendo" y "corrió" se asignan a "correr". 
- **Extracción de entidades**. Identificación de sujetos en el texto.
- **Detección de partes de la oración**. Identifica el texto como un verbo, nombre, participio, frase verbal, etc.
- **Detección del límite de las frases**. Detectar frases completas en párrafos de texto.

Cuando se utiliza el procesamiento de lenguaje natural para extraer información y datos a partir de texto sin formato, el punto inicial son normalmente los documentos sin formato almacenados en el almacenamiento de objetos, como Azure Storage o Azure Data Lake Store. 

## <a name="challenges"></a>Desafíos

- El procesamiento de una colección de documentos de texto sin formato suele consumir muchos recursos de cálculo y también en términos de tiempo.
- Sin un formato de documento estándar, puede ser muy difícil lograr resultados coherentes y precisos del procesamiento de texto sin formato para extraer datos concretos de un documento. Por ejemplo, imagine una representación en texto de una factura; puede ser difícil crear un proceso que extraiga correctamente la fecha de la factura y el número de la factura de las facturas de múltiples proveedores.

## <a name="architecture"></a>Architecture

En una solución de procesamiento de lenguaje natural, el procesamiento de texto sin formato se realiza en documentos que contienen párrafos de texto. La arquitectura general puede ser una arquitectura de [procesamiento por lotes](../big-data/batch-processing.md) o de [procesamiento de flujos en tiempo real](../big-data/real-time-processing.md).

El procesamiento real varía según el resultado deseado, pero, en cuanto a la canalización, se puede aplicar el procesamiento de lenguaje natural sobre un lote o en tiempo real. Por ejemplo, el análisis de opiniones se puede usar en bloques de texto para generar una puntuación de opinión. Esto se puede realizar mediante la ejecución de un proceso por lotes con los datos del almacenamiento o en tiempo real mediante fragmentos más pequeños de datos que fluyen a través de un servicio de mensajería.

## <a name="technology-choices"></a>Opciones de tecnología

- [Procesamiento de lenguaje natural](../technology-choices/natural-language-processing.md)