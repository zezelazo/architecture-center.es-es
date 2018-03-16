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
# <a name="choosing-a-natural-language-processing-technology-in-azure"></a><span data-ttu-id="d1bb9-102">Elección de una tecnología de procesamiento de lenguaje natural en Azure</span><span class="sxs-lookup"><span data-stu-id="d1bb9-102">Choosing a natural language processing technology in Azure</span></span>

<span data-ttu-id="d1bb9-103">El procesamiento de texto de forma libre se realiza en documentos que contienen párrafos de texto, normalmente con el propósito de ayudar a las búsquedas, pero también se usa para realizar otras tareas de procesamiento de lenguaje natural (NLP) como, por ejemplo, análisis de opiniones, detección de temas, detección de idioma, extracción de frases clave y clasificación de documentos.</span><span class="sxs-lookup"><span data-stu-id="d1bb9-103">Free-form text processing is performed against documents containing paragraphs of text, typically for the purpose of supporting search, but is also used to perform other natural language processing (NLP) tasks such as sentiment analysis, topic detection, language detection, key phrase extraction, and document categorization.</span></span> <span data-ttu-id="d1bb9-104">Este artículo se centra en las opciones de tecnología que actúan en apoyo de las tareas de procesamiento del lenguaje natural (NLP).</span><span class="sxs-lookup"><span data-stu-id="d1bb9-104">This article focuses on the technology choices that act in support of the NLP tasks.</span></span>

## <a name="what-are-your-options-when-choosing-an-nlp-service"></a><span data-ttu-id="d1bb9-105">¿Cuáles son las opciones al elegir un servicio NLP?</span><span class="sxs-lookup"><span data-stu-id="d1bb9-105">What are your options when choosing an NLP service?</span></span>

<span data-ttu-id="d1bb9-106">En Azure, los servicios siguientes proporcionan funcionalidades de procesamiento de lenguaje natural (NLP):</span><span class="sxs-lookup"><span data-stu-id="d1bb9-106">In Azure, the following services provide natural language processing (NLP) capabilities:</span></span>

- [<span data-ttu-id="d1bb9-107">Azure HDInsight con Spark y Spark MLlib</span><span class="sxs-lookup"><span data-stu-id="d1bb9-107">Azure HDInsight with Spark and Spark MLlib</span></span>](/azure/hdinsight/spark/apache-spark-overview)
- [<span data-ttu-id="d1bb9-108">Microsoft Cognitive Services</span><span class="sxs-lookup"><span data-stu-id="d1bb9-108">Microsoft Cognitive Services</span></span>](/azure/#pivot=products&panel=cognitive)

## <a name="key-selection-criteria"></a><span data-ttu-id="d1bb9-109">Principales criterios de selección</span><span class="sxs-lookup"><span data-stu-id="d1bb9-109">Key selection criteria</span></span>

<span data-ttu-id="d1bb9-110">Para restringir las opciones, empiece por responder a estas preguntas:</span><span class="sxs-lookup"><span data-stu-id="d1bb9-110">To narrow the choices, start by answering these questions:</span></span>

- <span data-ttu-id="d1bb9-111">¿Desea usar modelos creados previamente?</span><span class="sxs-lookup"><span data-stu-id="d1bb9-111">Do you want to use prebuilt models?</span></span> <span data-ttu-id="d1bb9-112">Si es así, considere la posibilidad de usar las API que ofrece Microsoft Cognitive Services.</span><span class="sxs-lookup"><span data-stu-id="d1bb9-112">If yes, consider using the APIs offered by Microsoft Cognitive Services.</span></span>

- <span data-ttu-id="d1bb9-113">¿Necesita entrenar modelos personalizados en un corpus grande de datos de texto?</span><span class="sxs-lookup"><span data-stu-id="d1bb9-113">Do you need to train custom models against a large corpus of text data?</span></span> <span data-ttu-id="d1bb9-114">Si es así, considere la posibilidad de usar Azure HDInsight con Spark MLlib y Spark NLP.</span><span class="sxs-lookup"><span data-stu-id="d1bb9-114">If yes, consider using Azure HDInsight with Spark MLlib and Spark NLP.</span></span>

- <span data-ttu-id="d1bb9-115">¿Necesita funcionalidades de procesamiento de lenguaje natural de bajo nivel como tokenización, lematización y frecuencia de términos o frecuencia inversa de documento (TF/IDF)?</span><span class="sxs-lookup"><span data-stu-id="d1bb9-115">Do you need low-level NLP capabilities like tokenization, stemming, lemmatization, and term frequency/inverse document frequency (TF/IDF)?</span></span> <span data-ttu-id="d1bb9-116">Si es así, considere la posibilidad de usar Azure HDInsight con Spark MLlib y Spark NLP.</span><span class="sxs-lookup"><span data-stu-id="d1bb9-116">If yes, consider using Azure HDInsight with Spark MLlib and Spark NLP.</span></span>

- <span data-ttu-id="d1bb9-117">¿Necesita capacidades de procesamiento de lenguaje natural simples y de alto nivel como identificación de entidades e intenciones, detección de temas, corrector ortográfico o análisis de opiniones?</span><span class="sxs-lookup"><span data-stu-id="d1bb9-117">Do you need simple, high-level NLP capabilities like entity and intent identification, topic detection, spell check, or sentiment analysis?</span></span> <span data-ttu-id="d1bb9-118">Si es así, considere la posibilidad de usar las API que ofrece Microsoft Cognitive Services.</span><span class="sxs-lookup"><span data-stu-id="d1bb9-118">If yes, consider using the APIs offered by Microsoft Cognitive Services.</span></span>

## <a name="capability-matrix"></a><span data-ttu-id="d1bb9-119">Matriz de funcionalidades</span><span class="sxs-lookup"><span data-stu-id="d1bb9-119">Capability matrix</span></span>

<span data-ttu-id="d1bb9-120">En las tablas siguientes se resumen las diferencias clave en cuanto a funcionalidades.</span><span class="sxs-lookup"><span data-stu-id="d1bb9-120">The following tables summarize the key differences in capabilities.</span></span>  

### <a name="general-capabilities"></a><span data-ttu-id="d1bb9-121">Funcionalidades generales</span><span class="sxs-lookup"><span data-stu-id="d1bb9-121">General capabilities</span></span>

| | <span data-ttu-id="d1bb9-122">HDInsight de Azure</span><span class="sxs-lookup"><span data-stu-id="d1bb9-122">Azure HDInsight</span></span> | <span data-ttu-id="d1bb9-123">Microsoft Cognitive Services</span><span class="sxs-lookup"><span data-stu-id="d1bb9-123">Microsoft Cognitive Services</span></span> |
| --- | --- | --- |
| <span data-ttu-id="d1bb9-124">Proporciona modelos previamente entrenados como un servicio</span><span class="sxs-lookup"><span data-stu-id="d1bb9-124">Provides pretrained models as a service</span></span> | <span data-ttu-id="d1bb9-125">Sin </span><span class="sxs-lookup"><span data-stu-id="d1bb9-125">No</span></span> | <span data-ttu-id="d1bb9-126">Sí</span><span class="sxs-lookup"><span data-stu-id="d1bb9-126">Yes</span></span> |
| <span data-ttu-id="d1bb9-127">API DE REST</span><span class="sxs-lookup"><span data-stu-id="d1bb9-127">REST API</span></span> | <span data-ttu-id="d1bb9-128">Sí</span><span class="sxs-lookup"><span data-stu-id="d1bb9-128">Yes</span></span> | <span data-ttu-id="d1bb9-129">Sí</span><span class="sxs-lookup"><span data-stu-id="d1bb9-129">Yes</span></span> |
| <span data-ttu-id="d1bb9-130">Capacidad de programación</span><span class="sxs-lookup"><span data-stu-id="d1bb9-130">Programmability</span></span> | <span data-ttu-id="d1bb9-131">Python, Scala, Java</span><span class="sxs-lookup"><span data-stu-id="d1bb9-131">Python, Scala, Java</span></span> | <span data-ttu-id="d1bb9-132">C#, Java, Node.js, Python, PHP, Ruby</span><span class="sxs-lookup"><span data-stu-id="d1bb9-132">C#, Java, Node.js, Python, PHP, Ruby</span></span> |
| <span data-ttu-id="d1bb9-133">Admite el procesamiento de macrodatos y documentos de gran tamaño</span><span class="sxs-lookup"><span data-stu-id="d1bb9-133">Support processing of big data sets and large documents</span></span> | <span data-ttu-id="d1bb9-134">Sí</span><span class="sxs-lookup"><span data-stu-id="d1bb9-134">Yes</span></span> | <span data-ttu-id="d1bb9-135">Sin </span><span class="sxs-lookup"><span data-stu-id="d1bb9-135">No</span></span> |

### <a name="low-level-natural-language-processing-capabilities"></a><span data-ttu-id="d1bb9-136">Funcionalidades de procesamiento de lenguaje natural de bajo nivel</span><span class="sxs-lookup"><span data-stu-id="d1bb9-136">Low-level natural language processing capabilities</span></span>

| | <span data-ttu-id="d1bb9-137">HDInsight de Azure</span><span class="sxs-lookup"><span data-stu-id="d1bb9-137">Azure HDInsight</span></span> | <span data-ttu-id="d1bb9-138">Microsoft Cognitive Services</span><span class="sxs-lookup"><span data-stu-id="d1bb9-138">Microsoft Cognitive Services</span></span> |  
| --- | --- | --- | 
| <span data-ttu-id="d1bb9-139">Tokenizador</span><span class="sxs-lookup"><span data-stu-id="d1bb9-139">Tokenizer</span></span> | <span data-ttu-id="d1bb9-140">Sí (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="d1bb9-140">Yes (Spark NLP)</span></span> | <span data-ttu-id="d1bb9-141">Sí (Linguistic Analysis API)</span><span class="sxs-lookup"><span data-stu-id="d1bb9-141">Yes (Linguistic Analysis API)</span></span> |
| <span data-ttu-id="d1bb9-142">Lematizador</span><span class="sxs-lookup"><span data-stu-id="d1bb9-142">Stemmer</span></span> | <span data-ttu-id="d1bb9-143">Sí (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="d1bb9-143">Yes (Spark NLP)</span></span> | <span data-ttu-id="d1bb9-144">Sin </span><span class="sxs-lookup"><span data-stu-id="d1bb9-144">No</span></span> |
| <span data-ttu-id="d1bb9-145">Lematizador</span><span class="sxs-lookup"><span data-stu-id="d1bb9-145">Lemmatizer</span></span> | <span data-ttu-id="d1bb9-146">Sí (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="d1bb9-146">Yes (Spark NLP)</span></span> | <span data-ttu-id="d1bb9-147">Sin </span><span class="sxs-lookup"><span data-stu-id="d1bb9-147">No</span></span> |
| <span data-ttu-id="d1bb9-148">Etiquetado de categorías gramaticales</span><span class="sxs-lookup"><span data-stu-id="d1bb9-148">Part of speech tagging</span></span> | <span data-ttu-id="d1bb9-149">Sí (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="d1bb9-149">Yes (Spark NLP)</span></span> | <span data-ttu-id="d1bb9-150">Sí (Linguistic Analysis API)</span><span class="sxs-lookup"><span data-stu-id="d1bb9-150">Yes (Linguistic Analysis API)</span></span> |
| <span data-ttu-id="d1bb9-151">Frecuencia de términos o frecuencia inversa de documento (TF/IDF)</span><span class="sxs-lookup"><span data-stu-id="d1bb9-151">Term frequency/inverse-document frequency (TF/IDF)</span></span> | <span data-ttu-id="d1bb9-152">Sí (Spark MLlib)</span><span class="sxs-lookup"><span data-stu-id="d1bb9-152">Yes (Spark MLlib)</span></span> | <span data-ttu-id="d1bb9-153">Sin </span><span class="sxs-lookup"><span data-stu-id="d1bb9-153">No</span></span> |
| <span data-ttu-id="d1bb9-154">Similitud de cadenas: edición del cálculo de distancias</span><span class="sxs-lookup"><span data-stu-id="d1bb9-154">String similarity&mdash;edit distance calculation</span></span> | <span data-ttu-id="d1bb9-155">Sí (Spark MLlib)</span><span class="sxs-lookup"><span data-stu-id="d1bb9-155">Yes (Spark MLlib)</span></span> | <span data-ttu-id="d1bb9-156">Sin </span><span class="sxs-lookup"><span data-stu-id="d1bb9-156">No</span></span> |
| <span data-ttu-id="d1bb9-157">Cálculo de n-gramas</span><span class="sxs-lookup"><span data-stu-id="d1bb9-157">N-gram calculation</span></span> | <span data-ttu-id="d1bb9-158">Sí (Spark MLlib)</span><span class="sxs-lookup"><span data-stu-id="d1bb9-158">Yes (Spark MLlib)</span></span> | <span data-ttu-id="d1bb9-159">Sin </span><span class="sxs-lookup"><span data-stu-id="d1bb9-159">No</span></span> |
| <span data-ttu-id="d1bb9-160">Detención de la eliminación de palabras</span><span class="sxs-lookup"><span data-stu-id="d1bb9-160">Stop word removal</span></span> | <span data-ttu-id="d1bb9-161">Sí (Spark MLlib)</span><span class="sxs-lookup"><span data-stu-id="d1bb9-161">Yes (Spark MLlib)</span></span> | <span data-ttu-id="d1bb9-162">Sin </span><span class="sxs-lookup"><span data-stu-id="d1bb9-162">No</span></span> |

### <a name="high-level-natural-language-processing-capabilities"></a><span data-ttu-id="d1bb9-163">Funcionalidades de procesamiento de lenguaje natural de alto nivel</span><span class="sxs-lookup"><span data-stu-id="d1bb9-163">High-level natural language processing capabilities</span></span>

| | <span data-ttu-id="d1bb9-164">HDInsight de Azure</span><span class="sxs-lookup"><span data-stu-id="d1bb9-164">Azure HDInsight</span></span> | <span data-ttu-id="d1bb9-165">Microsoft Cognitive Services</span><span class="sxs-lookup"><span data-stu-id="d1bb9-165">Microsoft Cognitive Services</span></span> |
| --- | --- | --- | 
| <span data-ttu-id="d1bb9-166">Identificación y extracción de entidades o intenciones</span><span class="sxs-lookup"><span data-stu-id="d1bb9-166">Entity/intent identification & extraction</span></span> | <span data-ttu-id="d1bb9-167">Sin </span><span class="sxs-lookup"><span data-stu-id="d1bb9-167">No</span></span> | <span data-ttu-id="d1bb9-168">Sí (Language Understanding Intelligent Service (LUIS) API)</span><span class="sxs-lookup"><span data-stu-id="d1bb9-168">Yes (Language Understanding Intelligent Service (LUIS) API)</span></span> |    
| <span data-ttu-id="d1bb9-169">Detección de temas</span><span class="sxs-lookup"><span data-stu-id="d1bb9-169">Topic detection</span></span> | <span data-ttu-id="d1bb9-170">Sí (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="d1bb9-170">Yes (Spark NLP)</span></span> | <span data-ttu-id="d1bb9-171">Sí (Text Analytics API)</span><span class="sxs-lookup"><span data-stu-id="d1bb9-171">Yes (Text Analytics API)</span></span> |
| <span data-ttu-id="d1bb9-172">Corrector ortográfico</span><span class="sxs-lookup"><span data-stu-id="d1bb9-172">Spell checking</span></span> | <span data-ttu-id="d1bb9-173">Sí (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="d1bb9-173">Yes (Spark NLP)</span></span> | <span data-ttu-id="d1bb9-174">Sí (Bing Spell Check API)</span><span class="sxs-lookup"><span data-stu-id="d1bb9-174">Yes (Bing Spell Check API)</span></span> |
| <span data-ttu-id="d1bb9-175">análisis de opiniones</span><span class="sxs-lookup"><span data-stu-id="d1bb9-175">Sentiment analysis</span></span> | <span data-ttu-id="d1bb9-176">Sí (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="d1bb9-176">Yes (Spark NLP)</span></span> | <span data-ttu-id="d1bb9-177">Sí (Text Analytics API)</span><span class="sxs-lookup"><span data-stu-id="d1bb9-177">Yes (Text Analytics API)</span></span> |
| <span data-ttu-id="d1bb9-178">Detección de idiomas</span><span class="sxs-lookup"><span data-stu-id="d1bb9-178">Language detection</span></span> | <span data-ttu-id="d1bb9-179">Sin </span><span class="sxs-lookup"><span data-stu-id="d1bb9-179">No</span></span> | <span data-ttu-id="d1bb9-180">Sí (Text Analytics API)</span><span class="sxs-lookup"><span data-stu-id="d1bb9-180">Yes (Text Analytics API)</span></span> |
| <span data-ttu-id="d1bb9-181">Admite varios idiomas además del inglés</span><span class="sxs-lookup"><span data-stu-id="d1bb9-181">Supports multiple languages besides English</span></span> | <span data-ttu-id="d1bb9-182">Sin </span><span class="sxs-lookup"><span data-stu-id="d1bb9-182">No</span></span> | <span data-ttu-id="d1bb9-183">Sí (varía según la API)</span><span class="sxs-lookup"><span data-stu-id="d1bb9-183">Yes (varies by API)</span></span> |

## <a name="see-also"></a><span data-ttu-id="d1bb9-184">Otras referencias</span><span class="sxs-lookup"><span data-stu-id="d1bb9-184">See also</span></span>

[<span data-ttu-id="d1bb9-185">Procesamiento de lenguaje natural</span><span class="sxs-lookup"><span data-stu-id="d1bb9-185">Natural language processing</span></span>](../scenarios/natural-language-processing.md)