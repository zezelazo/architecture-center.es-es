---
title: Datos relacionales
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: fa2fef27f47acadf00cbfc821c7432c07a3947be
ms.sourcegitcommit: 51f49026ec46af0860de55f6c082490e46792794
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/03/2018
---
# <a name="traditional-relational-database-solutions"></a>Soluciones de bases de datos relacionales tradicionales

Los datos relacionales son datos que se modelan mediante el modelo relacional. En este modelo, los datos se expresan como tuplas. Una *tupla* es un conjunto de pares atributo/valor. Por ejemplo, una tupla podría ser (artículoid = 5, pedidoid = 1, artículo= "Silla" cantidad = 200,00). Un conjunto de tuplas que comparten los mismos atributos se llama una *relación*. 

Las relaciones se representan de forma natural como tablas, donde cada tupla se expone como una fila en la tabla. Sin embargo, las filas tienen un orden explícito, a diferencia de las tuplas. El esquema de la base de datos define las columnas (encabezados) de cada tabla. Cada columna se define con un nombre y un tipo de datos para todos los valores almacenados en dicha columna en todas las filas de la tabla.

![Ejemplo que muestra datos en una base de datos relacional](../images/example-relational.png)

Un almacén de datos que organiza los datos mediante el modelo relacional se conoce como una base de datos relacional. Las claves principales identifican de forma única las filas dentro de una tabla. Los campos de las claves externas se utilizan en una tabla para hacer referencia a una fila de otra tabla haciendo referencia a la clave principal de la otra tabla. Las claves externas se utilizan para mantener la integridad referencial, garantizando que las filas a las que se hace referencia no se modifican ni eliminan, ya que la fila que hace referencia depende de ellas. 

![Ejemplo que muestra datos en una base de datos relacional](../images/example-relational2.png)

Las bases de datos relacionales admiten varios tipos de restricciones que ayudan a garantizar la integridad de los datos:

- Las restricciones únicas garantizan que todos los valores de una columna sean únicos. 

- Las restricciones de claves externas imponen un vínculo entre los datos de dos tablas. Una clave externa hace referencia a la clave principal o a otra clave única de otra tabla. Una restricción de claves externas exige integridad referencial y no se permiten los cambios que producen valores de claves externas no válidos.

- Las restricciones de comprobación, también conocidas como restricciones de integridad de entidad, limitan los valores que se pueden almacenar en una sola columna o en relación con los valores de otras columnas de la misma fila. 

La mayoría de las bases de datos relacionales utilizan el lenguaje de consulta estructurado (SQL), que permite un enfoque declarativo en las consultas. La consulta describe el resultado deseado, pero no los pasos para ejecutar la consulta. El motor, a continuación, decide la mejor manera de ejecutar la consulta. Esto difiere de la aproximación basada en procedimientos, donde el programa de consulta especifica los pasos de procesamiento de manera explícita. Sin embargo, las bases de datos relacionales pueden almacenar rutinas de código ejecutable en forma de procedimientos almacenados y funciones, lo que permite una combinación de los enfoques declarativos y por procedimientos.

Para mejorar el rendimiento de las consultas, las bases de datos relacionales utilizan *índices*. Los índices principales, que son usados por la clave principal, definen el orden de los datos en el disco. Los índices secundarios proporcionan una combinación alternativa de campos, para que se puedan consultar las filas deseadas de forma eficaz sin tener que volver a ordenar todos los datos del disco.

Dado que las bases de datos relacionales imponen la integridad referencial, el escalado de una base de datos relacional puede ser un desafío. Esto se debe a que cualquier operación de consulta o inserción puede tocar cualquier número de tablas. También es posible escalar horizontalmente una base de datos relacional mediante el *particionamiento* de los datos, pero esto requiere un cuidadoso diseño del esquema. Para más información, consulte el [patrón de particionamiento](../../patterns/sharding.md).

Si los datos son no relacionales o tienen requisitos que no son adecuados para una base de datos relacional, considere la posibilidad de un almacén de datos [no relacional o no SQL](../big-data/non-relational-data.md).
