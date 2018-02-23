---
title: "Guía de arquitectura de datos de Azure"
description: 
author: zoinerTejada
ms:date: 02/12/2018
layout: LandingPage
ms.openlocfilehash: 848601f27faf56ea069852d8983e4d10fbad9d77
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/14/2018
---
# <a name="azure-data-architecture-guide"></a>Guía de arquitectura de datos de Azure

Esta guía presenta un enfoque estructurado para el diseño de soluciones basadas en datos en Microsoft Azure. Se basa en prácticas probadas derivadas de las interacciones con los clientes.

## <a name="introduction"></a>Introducción

La nube está cambiando la forma en que las se diseñan las aplicaciones, lo que incluye la forma en que se procesan y almacenan los datos. En lugar de una sola base de datos de uso general que controle todos los datos de la solución, las soluciones de _persistencia políglota_ usan varios almacenes de datos especializados, cada uno de los cuales están optimizados para proporcionar funcionalidades concretas. Como consecuencia de ello cambia la perspectiva de los datos de la solución. Ya no hay varias capas de lógica de negocios que leen y escriben en una única capa de datos. En su lugar, las soluciones se diseñan en torno a una *canalización de datos* que describe cómo fluyen los datos a través de una solución, donde se procesan, donde se almacenan y cómo los consume el siguiente componente de la canalización. 

## <a name="how-this-guide-is-structured"></a>Cómo se estructura esta guía

Esta guía se estructura alrededor de un eje básico: la distinción entre datos *relacionales* y datos *no relacionales*. 

![](./images/guide-steps.svg)

Por lo general, los datos relacionales se almacenan en un RDBMS tradicional o en un almacenamiento de datos. Tienen un esquema predefinido ("esquema en escritura") con un conjunto de restricciones para mantener la integridad referencial. La mayoría de las bases de datos relacionales usan SQL (lenguaje de consulta estructurado) para realizar consultas. Las soluciones que usan bases de datos relacionales incluyen el procesamiento de transacciones en línea (OLTP) y el procesamiento analítico en línea (OLAP).

Los datos no relacionales son aquellos que no usan el [modelo relacional](https://en.wikipedia.org/wiki/Relational_model) que se encuentra en los RDBMS tradicionales. Aquí se pueden incluir datos de clave-valor, datos JSON, datos de grafos, datos de series temporales y otros tipos de datos. El término *NoSQL* hace referencia a aquellas bases de datos que están diseñadas para contener varios tipos de datos no relacionales. Sin embargo, dicho término no es totalmente exacto, porque muchos almacenes de datos no relacionales admiten consultas compatibles con SQL. Tanto los datos no relacionales como las bases de datos NoSQL suelen aparecer en los debates acerca de las soluciones de *macrodatos*. Una arquitectura de macrodatos está diseñada para controlar la ingesta, el procesamiento y el análisis de datos que son demasiado grandes o complejos para los sistemas de bases de datos tradicionales. 

En cada una de estas dos categorías principales, la guía de arquitectura de datos contiene las siguientes secciones:

- **Conceptos.** Artículos de información general que presentan los conceptos principales que debe conocer al trabajar con este tipo de datos.
- **Escenarios.** Un conjunto representativo de escenarios de datos, lo que incluye una explicación de los servicios de Azure relevantes y de la arquitectura adecuada para el escenario.
- **Opciones de tecnología.** Comparaciones detalladas de varias tecnologías de datos disponibles en Azure, lo que incluye las opciones de código abierto. Dentro de cada categoría, se describen los principales criterios de selección y una matriz de funcionalidades que le ayudarán a la hora de elegir la tecnología adecuada para su escenario.

Esta guía no está pensada para enseñar teoría de base de datos ni ciencia de datos (hay libros enteros al respecto). En su lugar, el objetivo es ayudarle a seleccionar la arquitectura de datos o canalización de datos apropiadas para su escenario y, después, a seleccionar los servicios de Azure y las tecnologías que más se ajustes a sus requisitos. Si ya tiene una arquitectura en mente, puede pasar directamente a las opciones de tecnología.

## <a name="traditional-rdbms"></a>RDBMS tradicional

### <a name="concepts"></a>Conceptos

- [Datos relacionales](./concepts/relational-data.md) 
- [Datos transaccionales](./concepts/transactional-data.md) 
- [Modelos semánticos](./concepts/semantic-modeling.md) 

### <a name="scenarios"></a>Escenarios

- [Procesamiento analítico en línea (OLAP)](./scenarios/online-analytical-processing.md)
- [Procesamiento de transacciones en línea (OLTP)](./scenarios/online-transaction-processing.md) 
- [Almacenamiento de datos y data marts](./scenarios/data-warehousing.md)
- [ETL](./scenarios/etl.md) 

## <a name="big-data-and-nosql"></a>Macrodatos y NoSQL

### <a name="concepts"></a>Conceptos

- [Almacenes de datos no relacionales](./concepts/non-relational-data.md)
- [Uso de archivos CSV y JSON](./concepts/csv-and-json.md)
- [Arquitecturas de macrodatos](./concepts/big-data.md)
- [Análisis avanzado](./concepts/advanced-analytics.md) 
- [Aprendizaje automático a escala](./concepts/machine-learning-at-scale.md)

### <a name="scenarios"></a>Escenarios

- [Procesamiento por lotes](./scenarios/batch-processing.md)
- [Procesamiento en tiempo real](./scenarios/real-time-processing.md)
- [Búsqueda de texto de forma libre](./scenarios/search.md)
- [Exploración interactiva de datos](./scenarios/interactive-data-exploration.md)
- [Procesamiento de lenguaje natural](./scenarios/natural-language-processing.md)
- [Soluciones de serie temporal](./scenarios/time-series.md)

## <a name="cross-cutting-concerns"></a>Problemas transversales

- [Transferencia de datos](./scenarios/data-transfer.md) 
- [Ampliación de las soluciones de datos locales a la nube](./scenarios/hybrid-on-premises-and-cloud.md) 
- [Soluciones de protección de datos](./scenarios/securing-data-solutions.md) 
