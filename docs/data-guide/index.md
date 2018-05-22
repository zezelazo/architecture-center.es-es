---
title: Guía de arquitectura de datos de Azure
description: ''
author: zoinerTejada
ms:date: 02/12/2018
layout: LandingPage
ms.topic: landing-page
ms.openlocfilehash: 63c1cca45fe9d99b5d0679360ef487c3a42da956
ms.sourcegitcommit: bb348bd3a8a4e27ef61e8eee74b54b07b65dbf98
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/21/2018
---
# <a name="azure-data-architecture-guide"></a>Guía de arquitectura de datos de Azure

Esta guía presenta un enfoque estructurado para el diseño de soluciones basadas en datos en Microsoft Azure. Se basa en prácticas probadas derivadas de las interacciones con los clientes.

## <a name="introduction"></a>Introducción

La nube está cambiando la forma en que las se diseñan las aplicaciones, lo que incluye la forma en que se procesan y almacenan los datos. En lugar de una sola base de datos de uso general que controle todos los datos de la solución, las soluciones de _persistencia políglota_ usan varios almacenes de datos especializados, cada uno de los cuales están optimizados para proporcionar funcionalidades concretas. Como consecuencia de ello cambia la perspectiva de los datos de la solución. Ya no hay varias capas de lógica de negocios que leen y escriben en una única capa de datos. En su lugar, las soluciones se diseñan en torno a una *canalización de datos* que describe cómo fluyen los datos a través de una solución, donde se procesan, donde se almacenan y cómo los consume el siguiente componente de la canalización. 

## <a name="how-this-guide-is-structured"></a>Cómo se estructura esta guía

Esta guía está estructurada en torno a dos categorías generales de soluciones de datos: *cargas de trabajo RDBMS tradicionales* y *soluciones de macrodatos*. 

**[Cargas de trabajo RDBMS tradicionales](./relational-data/index.md)**. Estas cargas de trabajo utilizan procesamiento de transacciones en línea (OLTP) y procesamiento analítico en línea (OLAP). En los sistemas OLTP, los datos suelen ser relacionales con un esquema predefinido y un conjunto de restricciones para mantener la integridad referencial. A menudo, se pueden consolidar datos de varios orígenes de la organización en un almacenamiento de datos, utilizando un proceso ETL para mover y transformar los datos de origen.

![](./images/guide-rdbms.svg)

**[Soluciones de macrodatos](./big-data/index.md)**. Una arquitectura de macrodatos está diseñada para controlar la ingesta, el procesamiento y el análisis de datos que son demasiado grandes o complejos para los sistemas de bases de datos tradicionales. Los datos se pueden procesar por lotes o en tiempo real. Las soluciones de macrodatos suelen implicar una gran cantidad de datos no relacionales, tales como datos de clave y valor, documentos JSON o datos de series temporales. Los sistemas RDBMS tradicionales no suelen ser adecuados para almacenar este tipo de datos. El término *NoSQL* hace referencia a la familia de bases de datos que están diseñadas para contener datos no relacionales. (El término no es totalmente exacto, porque muchos almacenes de datos no relacionales admiten consultas compatibles con SQL).

![](./images/guide-big-data.svg)

Estas dos categorías no son mutuamente excluyentes y se superponen entre ellas, pero creemos que es una forma útil de enmarcar el análisis. En la guía se describen los **escenarios habituales** dentro de cada categoría, incluida una explicación de los servicios de Azure pertinentes y la arquitectura adecuada para el escenario. Además, la guía compara las **opciones de tecnología** para las soluciones de datos de Azure, incluidas las opciones de código abierto. Dentro de cada categoría, se describen los principales criterios de selección y una matriz de funcionalidades que le ayudarán a la hora de elegir la tecnología adecuada para su escenario. 

Esta guía no está pensada para enseñar teoría de base de datos ni ciencia de datos (hay libros enteros al respecto). En su lugar, el objetivo es ayudarle a seleccionar la arquitectura de datos o canalización de datos apropiadas para su escenario y, después, a seleccionar los servicios de Azure y las tecnologías que más se ajustes a sus requisitos. Si ya tiene una arquitectura en mente, puede pasar directamente a las opciones de tecnología.
