---
title: "Procesamiento analítico en línea (OLAP)"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: f5ceea9c9dd03812e92fff811e54316edc22b59c
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/14/2018
---
# <a name="online-analytical-processing-olap"></a><span data-ttu-id="62434-102">Procesamiento analítico en línea (OLAP)</span><span class="sxs-lookup"><span data-stu-id="62434-102">Online analytical processing (OLAP)</span></span>

<span data-ttu-id="62434-103">El procesamiento analítico en línea (OLAP) es una tecnología que organiza grandes bases de datos empresariales y proporciona análisis complejo.</span><span class="sxs-lookup"><span data-stu-id="62434-103">Online analytical processing (OLAP) is a technology that organizes large business databases and supports complex analysis.</span></span> <span data-ttu-id="62434-104">Se puede utilizar para realizar consultas analíticas complejas sin afectar negativamente los sistemas transaccionales.</span><span class="sxs-lookup"><span data-stu-id="62434-104">It can be used to perform complex analytical queries without negatively affecting transactional systems.</span></span>

<span data-ttu-id="62434-105">Las bases de datos que utiliza una empresa para almacenar todas sus transacciones y registros se llaman bases de datos de [procesamiento de transacciones en línea (OLTP)](online-transaction-processing.md).</span><span class="sxs-lookup"><span data-stu-id="62434-105">The databases that a business uses to store all its transactions and records are called [online transaction processing (OLTP)](online-transaction-processing.md) databases.</span></span> <span data-ttu-id="62434-106">Normalmente, estas bases de datos tienen registros que se introducen uno cada vez.</span><span class="sxs-lookup"><span data-stu-id="62434-106">These databases usually have records that are entered one at a time.</span></span> <span data-ttu-id="62434-107">A menudo contienen una gran cantidad de información de valor para la organización.</span><span class="sxs-lookup"><span data-stu-id="62434-107">Often they contain a great deal of information that is valuable to the organization.</span></span> <span data-ttu-id="62434-108">Sin embargo, las bases de datos que se usan para OLTP no se diseñaron para el análisis.</span><span class="sxs-lookup"><span data-stu-id="62434-108">The databases that are used for OLTP, however, were not designed for analysis.</span></span> <span data-ttu-id="62434-109">Por lo tanto, obtener respuestas de estas bases de datos es costoso en términos de tiempo y esfuerzo.</span><span class="sxs-lookup"><span data-stu-id="62434-109">Therefore, retrieving answers from these databases is costly in terms of time and effort.</span></span> <span data-ttu-id="62434-110">Los sistemas OLAP se han diseñado para ayudar a extraer de los datos esta información de inteligencia empresarial con un alto rendimiento.</span><span class="sxs-lookup"><span data-stu-id="62434-110">OLAP systems were designed to help extract this business intelligence information from the data in a highly performant way.</span></span> <span data-ttu-id="62434-111">Esto se debe a que las bases de datos OLAP se optimizan para cargas de trabajo grandes en lecturas y pequeñas en escrituras.</span><span class="sxs-lookup"><span data-stu-id="62434-111">This is because OLAP databases are optimized for heavy read, low write workloads.</span></span>

![OLAP en Azure](./images/olap-data-pipeline.png) 

## <a name="when-to-use-this-solution"></a><span data-ttu-id="62434-113">Cuándo se debe utilizar esta solución</span><span class="sxs-lookup"><span data-stu-id="62434-113">When to use this solution</span></span>

<span data-ttu-id="62434-114">Considere el uso de OLAP en los siguientes escenarios:</span><span class="sxs-lookup"><span data-stu-id="62434-114">Consider OLAP in the following scenarios:</span></span>

- <span data-ttu-id="62434-115">Necesita ejecutar consultas ad hoc y análisis complejos rápidamente, sin afectar negativamente a los sistemas OLTP.</span><span class="sxs-lookup"><span data-stu-id="62434-115">You need to execute complex analytical and ad hoc queries rapidly, without negatively affecting your OLTP systems.</span></span> 
- <span data-ttu-id="62434-116">Desea proporcionar a los usuarios empresariales una manera sencilla de generar informes a partir de los datos</span><span class="sxs-lookup"><span data-stu-id="62434-116">You want to provide business users with a simple way to generate reports from your data</span></span>
- <span data-ttu-id="62434-117">Desea proporcionar diversas agregaciones que permitirán a los usuarios obtener resultados rápidos y coherentes.</span><span class="sxs-lookup"><span data-stu-id="62434-117">You want to provide a number of aggregations that will allow users to get fast, consistent results.</span></span> 

<span data-ttu-id="62434-118">OLAP es especialmente útil para aplicar cálculos agregados en grandes cantidades de datos.</span><span class="sxs-lookup"><span data-stu-id="62434-118">OLAP is especially useful for applying aggregate calculations over large amounts of data.</span></span> <span data-ttu-id="62434-119">Los sistemas OLAP están optimizados para escenarios de uso intensivo de lectura, como el análisis y la inteligencia empresarial.</span><span class="sxs-lookup"><span data-stu-id="62434-119">OLAP systems are optimized for read-heavy scenarios, such as analytics and business intelligence.</span></span> <span data-ttu-id="62434-120">OLAP permite a los usuarios dividir datos multidimensionales en segmentos que pueden visualizarse en dos dimensiones (por ejemplo, una tabla dinámica) o filtrar los datos por valores específicos.</span><span class="sxs-lookup"><span data-stu-id="62434-120">OLAP allows users to segment multi-dimensional data into slices that can be viewed in two dimensions (such as a pivot table) or filter the data by specific values.</span></span> <span data-ttu-id="62434-121">Este proceso se llama a veces "segmentar y desglosar" los datos y se puede levar a cabo independientemente de si los datos se dividen en varios orígenes de datos.</span><span class="sxs-lookup"><span data-stu-id="62434-121">This process is sometimes called "slicing and dicing" the data, and can be done regardless of whether the data is partitioned across several data sources.</span></span> <span data-ttu-id="62434-122">Esto ayuda a los usuarios a encontrar tendencias, detectar patrones y explorar los datos sin tener que conocer los detalles del análisis de datos tradicional.</span><span class="sxs-lookup"><span data-stu-id="62434-122">This helps users to find trends, spot patterns, and explore the data without having to know the details of traditional data analysis.</span></span>

<span data-ttu-id="62434-123">Los [modelos semánticos](../concepts/semantic-modeling.md) pueden ayudar a los usuarios empresariales en la abstracción de las complejidades de las relaciones y hacen más fácil analizar rápidamente los datos.</span><span class="sxs-lookup"><span data-stu-id="62434-123">[Semantic models](../concepts/semantic-modeling.md) can help business users abstract relationship complexities and make it easier to analyze data quickly.</span></span>

## <a name="challenges"></a><span data-ttu-id="62434-124">Desafíos</span><span class="sxs-lookup"><span data-stu-id="62434-124">Challenges</span></span>

<span data-ttu-id="62434-125">A pesar de todas las ventajas que proporcionan los sistemas OLAP, producen algunos desafíos:</span><span class="sxs-lookup"><span data-stu-id="62434-125">For all the benefits OLAP systems provide, they do produce a few challenges:</span></span>

- <span data-ttu-id="62434-126">En tanto que los datos en los sistemas OLTP se actualizan constantemente a través de transacciones que fluyen procedentes de diversos orígenes, los almacenes de datos OLAP normalmente se actualizan a intervalos mucho más lentos, en función de las necesidades del negocio.</span><span class="sxs-lookup"><span data-stu-id="62434-126">Whereas data in OLTP systems is constantly updated through transactions flowing in from various sources, OLAP data stores are typically refreshed at a much slower intervals, depending on business needs.</span></span> <span data-ttu-id="62434-127">Esto significa que los sistemas OLAP son más adecuados para tomar decisiones empresariales estratégicas, en lugar de dar respuestas inmediatas ante los cambios.</span><span class="sxs-lookup"><span data-stu-id="62434-127">This means OLAP systems are better suited for strategic business decisions, rather than immediate responses to changes.</span></span> <span data-ttu-id="62434-128">Además, se debe planear cierto nivel de limpieza de datos y orquestación para mantener actualizados los almacenes de datos OLAP.</span><span class="sxs-lookup"><span data-stu-id="62434-128">Also, some level of data cleansing and orchestration needs to be planned to keep the OLAP data stores up-to-date.</span></span>
- <span data-ttu-id="62434-129">A diferencia de las tablas tradicionales, normalizadas y relacionales encontradas en los sistemas OLTP, los modelos de datos OLAP suelen ser multidimensionales.</span><span class="sxs-lookup"><span data-stu-id="62434-129">Unlike traditional, normalized, relational tables found in OLTP systems, OLAP data models tend to be multidimensional.</span></span> <span data-ttu-id="62434-130">Esto hace difícil o imposible la asignación directa a modelos entidad-relación y modelos orientados a objetos, en los que cada atributo se asigna a una columna.</span><span class="sxs-lookup"><span data-stu-id="62434-130">This makes it difficult or impossible to directly map to entity-relationship or object-oriented models, where each attribute is mapped to one column.</span></span> <span data-ttu-id="62434-131">Los sistemas OLAP normalmente usan un esquema de estrella o copo de nieve en lugar de la normalización tradicional.</span><span class="sxs-lookup"><span data-stu-id="62434-131">Instead, OLAP systems typically use a star or snowflake schema in place of traditional normalization.</span></span>

## <a name="olap-in-azure"></a><span data-ttu-id="62434-132">OLAP en Azure</span><span class="sxs-lookup"><span data-stu-id="62434-132">OLAP in Azure</span></span>

<span data-ttu-id="62434-133">En Azure, los datos que se mantienen en sistemas OLTP, como Azure SQL Database, se copian en el sistema OLAP, como [Azure Analysis Services](/azure/analysis-services/analysis-services-overview).</span><span class="sxs-lookup"><span data-stu-id="62434-133">In Azure, data held in OLTP systems such as Azure SQL Database is copied into the OLAP system, such as [Azure Analysis Services](/azure/analysis-services/analysis-services-overview).</span></span> <span data-ttu-id="62434-134">Las herramientas de exploración y visualización de datos como [Power BI](https://powerbi.microsoft.com), Excel y otras herramientas de terceros se conectan a los servidores de Analysis Services y proporcionan a los usuarios una información interactiva y enriquecida visualmente sobre los datos modelados.</span><span class="sxs-lookup"><span data-stu-id="62434-134">Data exploration and visualization tools like [Power BI](https://powerbi.microsoft.com), Excel, and third-party options connect to Analysis Services servers and provide users with highly interactive and visually rich insights into the modeled data.</span></span> <span data-ttu-id="62434-135">El flujo de los datos desde OLTP a OLAP normalmente se orquesta con SQL Server Integration Services, que se puede ejecutar con [Azure Data Factory](/azure/data-factory/concepts-integration-runtime).</span><span class="sxs-lookup"><span data-stu-id="62434-135">The flow of data from OLTP data to OLAP is typically orchestrated using SQL Server Integration Services, which can be executed using [Azure Data Factory](/azure/data-factory/concepts-integration-runtime).</span></span>

## <a name="technology-choices"></a><span data-ttu-id="62434-136">Opciones de tecnología</span><span class="sxs-lookup"><span data-stu-id="62434-136">Technology choices</span></span>

- [<span data-ttu-id="62434-137">Almacenes de datos de procesamiento analítico en línea (OLAP)</span><span class="sxs-lookup"><span data-stu-id="62434-137">Online Analytical Processing (OLAP) data stores</span></span>](../technology-choices/olap-data-stores.md)
