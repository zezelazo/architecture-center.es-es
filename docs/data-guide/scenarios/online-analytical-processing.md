---
title: Procesamiento analítico en línea (OLAP)
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: f5ceea9c9dd03812e92fff811e54316edc22b59c
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/31/2018
---
# <a name="online-analytical-processing-olap"></a>Procesamiento analítico en línea (OLAP)

El procesamiento analítico en línea (OLAP) es una tecnología que organiza grandes bases de datos empresariales y proporciona análisis complejo. Se puede utilizar para realizar consultas analíticas complejas sin afectar negativamente los sistemas transaccionales.

Las bases de datos que utiliza una empresa para almacenar todas sus transacciones y registros se llaman bases de datos de [procesamiento de transacciones en línea (OLTP)](online-transaction-processing.md). Normalmente, estas bases de datos tienen registros que se introducen uno cada vez. A menudo contienen una gran cantidad de información de valor para la organización. Sin embargo, las bases de datos que se usan para OLTP no se diseñaron para el análisis. Por lo tanto, obtener respuestas de estas bases de datos es costoso en términos de tiempo y esfuerzo. Los sistemas OLAP se han diseñado para ayudar a extraer de los datos esta información de inteligencia empresarial con un alto rendimiento. Esto se debe a que las bases de datos OLAP se optimizan para cargas de trabajo grandes en lecturas y pequeñas en escrituras.

![OLAP en Azure](./images/olap-data-pipeline.png) 

## <a name="when-to-use-this-solution"></a>Cuándo se debe utilizar esta solución

Considere el uso de OLAP en los siguientes escenarios:

- Necesita ejecutar consultas ad hoc y análisis complejos rápidamente, sin afectar negativamente a los sistemas OLTP. 
- Desea proporcionar a los usuarios empresariales una manera sencilla de generar informes a partir de los datos
- Desea proporcionar diversas agregaciones que permitirán a los usuarios obtener resultados rápidos y coherentes. 

OLAP es especialmente útil para aplicar cálculos agregados en grandes cantidades de datos. Los sistemas OLAP están optimizados para escenarios de uso intensivo de lectura, como el análisis y la inteligencia empresarial. OLAP permite a los usuarios dividir datos multidimensionales en segmentos que pueden visualizarse en dos dimensiones (por ejemplo, una tabla dinámica) o filtrar los datos por valores específicos. Este proceso se llama a veces "segmentar y desglosar" los datos y se puede levar a cabo independientemente de si los datos se dividen en varios orígenes de datos. Esto ayuda a los usuarios a encontrar tendencias, detectar patrones y explorar los datos sin tener que conocer los detalles del análisis de datos tradicional.

Los [modelos semánticos](../concepts/semantic-modeling.md) pueden ayudar a los usuarios empresariales en la abstracción de las complejidades de las relaciones y hacen más fácil analizar rápidamente los datos.

## <a name="challenges"></a>Desafíos

A pesar de todas las ventajas que proporcionan los sistemas OLAP, producen algunos desafíos:

- En tanto que los datos en los sistemas OLTP se actualizan constantemente a través de transacciones que fluyen procedentes de diversos orígenes, los almacenes de datos OLAP normalmente se actualizan a intervalos mucho más lentos, en función de las necesidades del negocio. Esto significa que los sistemas OLAP son más adecuados para tomar decisiones empresariales estratégicas, en lugar de dar respuestas inmediatas ante los cambios. Además, se debe planear cierto nivel de limpieza de datos y orquestación para mantener actualizados los almacenes de datos OLAP.
- A diferencia de las tablas tradicionales, normalizadas y relacionales encontradas en los sistemas OLTP, los modelos de datos OLAP suelen ser multidimensionales. Esto hace difícil o imposible la asignación directa a modelos entidad-relación y modelos orientados a objetos, en los que cada atributo se asigna a una columna. Los sistemas OLAP normalmente usan un esquema de estrella o copo de nieve en lugar de la normalización tradicional.

## <a name="olap-in-azure"></a>OLAP en Azure

En Azure, los datos que se mantienen en sistemas OLTP, como Azure SQL Database, se copian en el sistema OLAP, como [Azure Analysis Services](/azure/analysis-services/analysis-services-overview). Las herramientas de exploración y visualización de datos como [Power BI](https://powerbi.microsoft.com), Excel y otras herramientas de terceros se conectan a los servidores de Analysis Services y proporcionan a los usuarios una información interactiva y enriquecida visualmente sobre los datos modelados. El flujo de los datos desde OLTP a OLAP normalmente se orquesta con SQL Server Integration Services, que se puede ejecutar con [Azure Data Factory](/azure/data-factory/concepts-integration-runtime).

## <a name="technology-choices"></a>Opciones de tecnología

- [Almacenes de datos de procesamiento analítico en línea (OLAP)](../technology-choices/olap-data-stores.md)

