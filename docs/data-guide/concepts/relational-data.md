---
title: Datos relacionales
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 55c7354b8cec13318bbf3fda1c648cde17288854
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/14/2018
---
# <a name="relational-data"></a><span data-ttu-id="9c67d-102">Datos relacionales</span><span class="sxs-lookup"><span data-stu-id="9c67d-102">Relational data</span></span>

<span data-ttu-id="9c67d-103">Los datos relacionales son datos que se modelan mediante el modelo relacional.</span><span class="sxs-lookup"><span data-stu-id="9c67d-103">Relational data is data modeled using the relational model.</span></span> <span data-ttu-id="9c67d-104">En este modelo, los datos se expresan como tuplas.</span><span class="sxs-lookup"><span data-stu-id="9c67d-104">In this model, data is expressed as tuples.</span></span> <span data-ttu-id="9c67d-105">Una *tupla* es un conjunto de pares atributo/valor.</span><span class="sxs-lookup"><span data-stu-id="9c67d-105">A *tuple* is a set of attribute/value pairs.</span></span> <span data-ttu-id="9c67d-106">Por ejemplo, una tupla podría ser (artículoid = 5, pedidoid = 1, artículo= "Silla" cantidad = 200,00).</span><span class="sxs-lookup"><span data-stu-id="9c67d-106">For example, a tuple might be (itemid = 5, orderid = 1, item = "Chair", amount = 200.00).</span></span> <span data-ttu-id="9c67d-107">Un conjunto de tuplas que comparten los mismos atributos se llama una *relación*.</span><span class="sxs-lookup"><span data-stu-id="9c67d-107">A set of tuples that all share the same attributes is called a *relation*.</span></span> 

<span data-ttu-id="9c67d-108">Las relaciones se representan de forma natural como tablas, donde cada tupla se expone como una fila en la tabla.</span><span class="sxs-lookup"><span data-stu-id="9c67d-108">Relations are naturally represented as tables, where each tuple is exposed as a row in the table.</span></span> <span data-ttu-id="9c67d-109">Sin embargo, las filas tienen un orden explícito, a diferencia de las tuplas.</span><span class="sxs-lookup"><span data-stu-id="9c67d-109">However, rows have an explicit ordering, unlike tuples.</span></span> <span data-ttu-id="9c67d-110">El esquema de la base de datos define las columnas (encabezados) de cada tabla.</span><span class="sxs-lookup"><span data-stu-id="9c67d-110">The database schema defines the columns (headings) of each table.</span></span> <span data-ttu-id="9c67d-111">Cada columna se define con un nombre y un tipo de datos para todos los valores almacenados en dicha columna en todas las filas de la tabla.</span><span class="sxs-lookup"><span data-stu-id="9c67d-111">Each column is defined with a name and a data type for all values stored in that column across all rows in the table.</span></span>

![Ejemplo que muestra datos en una base de datos relacional](./images/example-relational.png)

<span data-ttu-id="9c67d-113">Un almacén de datos que organiza los datos mediante el modelo relacional se conoce como una base de datos relacional.</span><span class="sxs-lookup"><span data-stu-id="9c67d-113">A data store that organizes data using the relational model is referred to as a relational database.</span></span> <span data-ttu-id="9c67d-114">Las claves principales identifican de forma única las filas dentro de una tabla.</span><span class="sxs-lookup"><span data-stu-id="9c67d-114">Primary keys uniquely identify rows within a table.</span></span> <span data-ttu-id="9c67d-115">Los campos de las claves externas se utilizan en una tabla para hacer referencia a una fila de otra tabla haciendo referencia a la clave principal de la otra tabla.</span><span class="sxs-lookup"><span data-stu-id="9c67d-115">Foreign key fields are used in one table to refer to a row in another table by referencing the primary key of the other table.</span></span> <span data-ttu-id="9c67d-116">Las claves externas se utilizan para mantener la integridad referencial, garantizando que las filas a las que se hace referencia no se modifican ni eliminan, ya que la fila que hace referencia depende de ellas.</span><span class="sxs-lookup"><span data-stu-id="9c67d-116">Foreign keys are used to maintain referential integrity, ensuring that the referenced rows are not altered or deleted while the referencing row depends on them.</span></span> 

![Ejemplo que muestra datos en una base de datos relacional](./images/example-relational2.png)

<span data-ttu-id="9c67d-118">Las bases de datos relacionales admiten varios tipos de restricciones que ayudan a garantizar la integridad de los datos:</span><span class="sxs-lookup"><span data-stu-id="9c67d-118">Relational databases support various types of constraints that help to ensure data integrity:</span></span>

- <span data-ttu-id="9c67d-119">Las restricciones únicas garantizan que todos los valores de una columna sean únicos.</span><span class="sxs-lookup"><span data-stu-id="9c67d-119">Unique constraints ensure that all values in a column are unique.</span></span> 

- <span data-ttu-id="9c67d-120">Las restricciones de claves externas imponen un vínculo entre los datos de dos tablas.</span><span class="sxs-lookup"><span data-stu-id="9c67d-120">Foreign key constraints enforce a link between the data in two tables.</span></span> <span data-ttu-id="9c67d-121">Una clave externa hace referencia a la clave principal o a otra clave única de otra tabla.</span><span class="sxs-lookup"><span data-stu-id="9c67d-121">A foreign key references the primary key or another unique key from another table.</span></span> <span data-ttu-id="9c67d-122">Una restricción de claves externas exige integridad referencial y no se permiten los cambios que producen valores de claves externas no válidos.</span><span class="sxs-lookup"><span data-stu-id="9c67d-122">A foreign key constraint enforces referential integrity, disallowing changes that cause invalid foreign key values.</span></span>

- <span data-ttu-id="9c67d-123">Las restricciones de comprobación, también conocidas como restricciones de integridad de entidad, limitan los valores que se pueden almacenar en una sola columna o en relación con los valores de otras columnas de la misma fila.</span><span class="sxs-lookup"><span data-stu-id="9c67d-123">Check constraints, also known as entity integrity constraints, limit the values that can be stored within a single column, or in relationship to values in other columns of the same row.</span></span> 

<span data-ttu-id="9c67d-124">La mayoría de las bases de datos relacionales utilizan el lenguaje de consulta estructurado (SQL), que permite un enfoque declarativo en las consultas.</span><span class="sxs-lookup"><span data-stu-id="9c67d-124">Most relational databases use the Structured Query Language (SQL) language that enables a declarative approach to querying.</span></span> <span data-ttu-id="9c67d-125">La consulta describe el resultado deseado, pero no los pasos para ejecutar la consulta.</span><span class="sxs-lookup"><span data-stu-id="9c67d-125">The query describes the desired result, but not the steps to execute the query.</span></span> <span data-ttu-id="9c67d-126">El motor, a continuación, decide la mejor manera de ejecutar la consulta.</span><span class="sxs-lookup"><span data-stu-id="9c67d-126">The engine then decides the best way to execute the query.</span></span> <span data-ttu-id="9c67d-127">Esto difiere de la aproximación basada en procedimientos, donde el programa de consulta especifica los pasos de procesamiento de manera explícita.</span><span class="sxs-lookup"><span data-stu-id="9c67d-127">This differs from a procedural approach, where the query program specifies the processing steps explicitly.</span></span> <span data-ttu-id="9c67d-128">Sin embargo, las bases de datos relacionales pueden almacenar rutinas de código ejecutable en forma de procedimientos almacenados y funciones, lo que permite una combinación de los enfoques declarativos y por procedimientos.</span><span class="sxs-lookup"><span data-stu-id="9c67d-128">However, relational databases can store executable code routines in the form of stored procedures and functions, which enables a mixture of declarative and procedural approaches.</span></span>

<span data-ttu-id="9c67d-129">Para mejorar el rendimiento de las consultas, las bases de datos relacionales utilizan *índices*.</span><span class="sxs-lookup"><span data-stu-id="9c67d-129">To improve query performance, relational databases use *indexes*.</span></span> <span data-ttu-id="9c67d-130">Los índices principales, que son usados por la clave principal, definen el orden de los datos en el disco.</span><span class="sxs-lookup"><span data-stu-id="9c67d-130">Primary indexes, which are used by the primary key, define the order of the data as it sits on disk.</span></span> <span data-ttu-id="9c67d-131">Los índices secundarios proporcionan una combinación alternativa de campos, para que se puedan consultar las filas deseadas de forma eficaz sin tener que volver a ordenar todos los datos del disco.</span><span class="sxs-lookup"><span data-stu-id="9c67d-131">Secondary indexes provide an alternative combination of fields, so the desired rows can be queried efficiently, without having to re-sort the entire data on disk.</span></span>

<span data-ttu-id="9c67d-132">Dado que las bases de datos relacionales imponen la integridad referencial, el escalado de una base de datos relacional puede ser un desafío.</span><span class="sxs-lookup"><span data-stu-id="9c67d-132">Because relational databases enforce referential integrity, scaling a relational database can become challenging.</span></span> <span data-ttu-id="9c67d-133">Esto se debe a que cualquier operación de consulta o inserción puede tocar cualquier número de tablas.</span><span class="sxs-lookup"><span data-stu-id="9c67d-133">That's because any query or insert operation might touch any number of tables.</span></span> <span data-ttu-id="9c67d-134">También es posible escalar horizontalmente una base de datos relacional mediante el *particionamiento* de los datos, pero esto requiere un cuidadoso diseño del esquema.</span><span class="sxs-lookup"><span data-stu-id="9c67d-134">You can scale out a relational database by *sharding* the data, but this requires careful design of the schema.</span></span> <span data-ttu-id="9c67d-135">Para más información, consulte el [patrón de particionamiento](../../patterns/sharding.md).</span><span class="sxs-lookup"><span data-stu-id="9c67d-135">For more information, see [Sharding pattern](../../patterns/sharding.md).</span></span>

<span data-ttu-id="9c67d-136">Si los datos son no relacionales o tienen requisitos que no son adecuados para una base de datos relacional, considere la posibilidad de un almacén de datos [no relacional o no SQL](./non-relational-data.md).</span><span class="sxs-lookup"><span data-stu-id="9c67d-136">If data is non-relational or has requirements that are not suited to a relational database, consider a [Non-relational or NoSQL](./non-relational-data.md) data store.</span></span>