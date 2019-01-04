---
title: Antipatrón Busy Database
titleSuffix: Performance antipatterns for cloud apps
description: Descargar el procesamiento en un servidor de base de datos puede provocar problemas de rendimiento y escalabilidad.
author: dragon119
ms.date: 06/05/2017
ms.custom: seodec18
ms.openlocfilehash: 11bce03aed2e988d0a814b3298818715ba42c1c5
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/04/2019
ms.locfileid: "54011470"
---
# <a name="busy-database-antipattern"></a><span data-ttu-id="5a96a-103">Antipatrón Busy Database</span><span class="sxs-lookup"><span data-stu-id="5a96a-103">Busy Database antipattern</span></span>

<span data-ttu-id="5a96a-104">Descargar el procesamiento en un servidor de base de datos puede provocar que se dedique una proporción considerable de tiempo en ejecutar el código, en lugar de responder a las solicitudes para almacenar y recuperar datos.</span><span class="sxs-lookup"><span data-stu-id="5a96a-104">Offloading processing to a database server can cause it to spend a significant proportion of time running code, rather than responding to requests to store and retrieve data.</span></span>

## <a name="problem-description"></a><span data-ttu-id="5a96a-105">Descripción del problema</span><span class="sxs-lookup"><span data-stu-id="5a96a-105">Problem description</span></span>

<span data-ttu-id="5a96a-106">Muchos sistemas de base de datos pueden ejecutar código.</span><span class="sxs-lookup"><span data-stu-id="5a96a-106">Many database systems can run code.</span></span> <span data-ttu-id="5a96a-107">Algunos ejemplos son los procedimientos almacenados y los desencadenadores.</span><span class="sxs-lookup"><span data-stu-id="5a96a-107">Examples include stored procedures and triggers.</span></span> <span data-ttu-id="5a96a-108">A menudo, es más eficaz realizar este procesamiento cerca de los datos, en lugar de transmitirlos a una aplicación cliente para ser procesados.</span><span class="sxs-lookup"><span data-stu-id="5a96a-108">Often, it's more efficient to perform this processing close to the data, rather than transmitting the data to a client application for processing.</span></span> <span data-ttu-id="5a96a-109">Sin embargo, el uso excesivo de estas características puede afectar al rendimiento, por varias razones:</span><span class="sxs-lookup"><span data-stu-id="5a96a-109">However, overusing these features can hurt performance, for several reasons:</span></span>

- <span data-ttu-id="5a96a-110">El servidor de base de datos puede dedicar demasiado tiempo en el procesamiento, en lugar de aceptar nuevas solicitudes de cliente y capturar los datos.</span><span class="sxs-lookup"><span data-stu-id="5a96a-110">The database server may spend too much time processing, rather than accepting new client requests and fetching data.</span></span>
- <span data-ttu-id="5a96a-111">Una base de datos suele ser un recurso compartido, por lo que puede convertirse en un cuello de botella durante los períodos de uso elevado.</span><span class="sxs-lookup"><span data-stu-id="5a96a-111">A database is usually a shared resource, so it can become a bottleneck during periods of high use.</span></span>
- <span data-ttu-id="5a96a-112">Los costos del entorno de tiempo de ejecución pueden ser excesivos si se mide el almacén de datos.</span><span class="sxs-lookup"><span data-stu-id="5a96a-112">Runtime costs may be excessive if the data store is metered.</span></span> <span data-ttu-id="5a96a-113">Eso es especialmente cierto en los servicios administrados de la base de datos.</span><span class="sxs-lookup"><span data-stu-id="5a96a-113">That's particularly true of managed database services.</span></span> <span data-ttu-id="5a96a-114">Por ejemplo, Azure SQL Dabatase cobra por las [unidades de transacción de base de datos] [ dtu] (DTU).</span><span class="sxs-lookup"><span data-stu-id="5a96a-114">For example, Azure SQL Database charges for [Database Transaction Units][dtu] (DTUs).</span></span>
- <span data-ttu-id="5a96a-115">Las bases de datos tienen una capacidad limitada para escalarse verticalmente y escalarlas horizontalmente no resulta trivial.</span><span class="sxs-lookup"><span data-stu-id="5a96a-115">Databases have finite capacity to scale up, and it's not trivial to scale a database horizontally.</span></span> <span data-ttu-id="5a96a-116">Por lo tanto, es posible que convenga mover el procesamiento a un recurso de proceso, como una aplicación de App Service o una máquina virtual, que se puede escalar fácilmente.</span><span class="sxs-lookup"><span data-stu-id="5a96a-116">Therefore, it may be better to move processing into a compute resource, such as a VM or App Service app, that can easily scale out.</span></span>

<span data-ttu-id="5a96a-117">Este antipatrón suele ocurrir porque:</span><span class="sxs-lookup"><span data-stu-id="5a96a-117">This antipattern typically occurs because:</span></span>

- <span data-ttu-id="5a96a-118">La base de datos se ve como un servicio en lugar de un repositorio.</span><span class="sxs-lookup"><span data-stu-id="5a96a-118">The database is viewed as a service rather than a repository.</span></span> <span data-ttu-id="5a96a-119">Una aplicación puede utilizar el servidor de base de datos para dar formato a loa datos (por ejemplo, convertirlos en XML), manipular datos de cadena o realizar cálculos complejos.</span><span class="sxs-lookup"><span data-stu-id="5a96a-119">An application might use the database server to format data (for example, converting to XML), manipulate string data, or perform complex calculations.</span></span>
- <span data-ttu-id="5a96a-120">Los desarrolladores intentan escribir las consultas cuyos resultados se pueden mostrar directamente a los usuarios.</span><span class="sxs-lookup"><span data-stu-id="5a96a-120">Developers try to write queries whose results can be displayed directly to users.</span></span> <span data-ttu-id="5a96a-121">Por ejemplo, una consulta puede combinar campos o dar formato a fechas, horas y monedas según la configuración regional.</span><span class="sxs-lookup"><span data-stu-id="5a96a-121">For example a query might combine fields, or format dates, times, and currency according to locale.</span></span>
- <span data-ttu-id="5a96a-122">Los desarrolladores intentan corregir el antipatrón [Extraneous Fetching] [ExtraneousFetching] insertando cálculos en la base de datos.</span><span class="sxs-lookup"><span data-stu-id="5a96a-122">Developers are trying to correct the [Extraneous Fetching][ExtraneousFetching] antipattern by pushing computations to the database.</span></span>
- <span data-ttu-id="5a96a-123">Los procedimientos almacenados se usan para encapsular la lógica de negocios, quizás porque se consideran más fáciles de mantener y actualizar.</span><span class="sxs-lookup"><span data-stu-id="5a96a-123">Stored procedures are used to encapsulate business logic, perhaps because they are considered easier to maintain and update.</span></span>

<span data-ttu-id="5a96a-124">En el ejemplo siguiente se recuperan los 20 pedidos más valiosos de una zona de ventas especificada y se da formato a los resultados como XML.</span><span class="sxs-lookup"><span data-stu-id="5a96a-124">The following example retrieves the 20 most valuable orders for a specified sales territory and formats the results as XML.</span></span>
<span data-ttu-id="5a96a-125">Usa las funciones de Transact-SQL para analizar los datos y convertir los resultados en XML.</span><span class="sxs-lookup"><span data-stu-id="5a96a-125">It uses Transact-SQL functions to parse the data and convert the results to XML.</span></span> <span data-ttu-id="5a96a-126">Puede encontrar el ejemplo completo [aquí][sample-app].</span><span class="sxs-lookup"><span data-stu-id="5a96a-126">You can find the complete sample [here][sample-app].</span></span>

```SQL
SELECT TOP 20
  soh.[SalesOrderNumber]  AS '@OrderNumber',
  soh.[Status]            AS '@Status',
  soh.[ShipDate]          AS '@ShipDate',
  YEAR(soh.[OrderDate])   AS '@OrderDateYear',
  MONTH(soh.[OrderDate])  AS '@OrderDateMonth',
  soh.[DueDate]           AS '@DueDate',
  FORMAT(ROUND(soh.[SubTotal],2),'C')
                          AS '@SubTotal',
  FORMAT(ROUND(soh.[TaxAmt],2),'C')
                          AS '@TaxAmt',
  FORMAT(ROUND(soh.[TotalDue],2),'C')
                          AS '@TotalDue',
  CASE WHEN soh.[TotalDue] > 5000 THEN 'Y' ELSE 'N' END
                          AS '@ReviewRequired',
  (
  SELECT
    c.[AccountNumber]     AS '@AccountNumber',
    UPPER(LTRIM(RTRIM(REPLACE(
    CONCAT( p.[Title], ' ', p.[FirstName], ' ', p.[MiddleName], ' ', p.[LastName], ' ', p.[Suffix]),
    '  ', ' '))))         AS '@FullName'
  FROM [Sales].[Customer] c
    INNER JOIN [Person].[Person] p
  ON c.[PersonID] = p.[BusinessEntityID]
  WHERE c.[CustomerID] = soh.[CustomerID]
  FOR XML PATH ('Customer'), TYPE
  ),

  (
  SELECT
    sod.[OrderQty]      AS '@Quantity',
    FORMAT(sod.[UnitPrice],'C')
                        AS '@UnitPrice',
    FORMAT(ROUND(sod.[LineTotal],2),'C')
                        AS '@LineTotal',
    sod.[ProductID]     AS '@ProductId',
    CASE WHEN (sod.[ProductID] >= 710) AND (sod.[ProductID] <= 720) AND (sod.[OrderQty] >= 5) THEN 'Y' ELSE 'N' END
                        AS '@InventoryCheckRequired'

  FROM [Sales].[SalesOrderDetail] sod
  WHERE sod.[SalesOrderID] = soh.[SalesOrderID]
  ORDER BY sod.[SalesOrderDetailID]
  FOR XML PATH ('LineItem'), TYPE, ROOT('OrderLineItems')
  )

FROM [Sales].[SalesOrderHeader] soh
WHERE soh.[TerritoryId] = @TerritoryId
ORDER BY soh.[TotalDue] DESC
FOR XML PATH ('Order'), ROOT('Orders')
```

<span data-ttu-id="5a96a-127">Claramente, se trata de una consulta compleja.</span><span class="sxs-lookup"><span data-stu-id="5a96a-127">Clearly, this is complex query.</span></span> <span data-ttu-id="5a96a-128">Como veremos más adelante, termina por usar recursos de procesamiento significativos en el servidor de base de datos.</span><span class="sxs-lookup"><span data-stu-id="5a96a-128">As we'll see later, it turns out to use significant processing resources on the database server.</span></span>

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="5a96a-129">Procedimiento para corregir el problema</span><span class="sxs-lookup"><span data-stu-id="5a96a-129">How to fix the problem</span></span>

<span data-ttu-id="5a96a-130">Pase el procesamiento del servidor de base de datos a otros niveles de aplicación.</span><span class="sxs-lookup"><span data-stu-id="5a96a-130">Move processing from the database server into other application tiers.</span></span> <span data-ttu-id="5a96a-131">Lo mejor sería limitar la base de datos para realizar las operaciones de acceso a datos, utilizando solo las funcionalidades para las que la base de datos esté optimizada, por ejemplo, la agregación en un RDBMS.</span><span class="sxs-lookup"><span data-stu-id="5a96a-131">Ideally, you should limit the database to performing data access operations, using only the capabilities that the database is optimized for, such as aggregation in an RDBMS.</span></span>

<span data-ttu-id="5a96a-132">Por ejemplo, el código de Transact-SQL anterior puede sustituirse por una instrucción que simplemente recupere los datos que se van a procesar.</span><span class="sxs-lookup"><span data-stu-id="5a96a-132">For example, the previous Transact-SQL code can be replaced with a statement that simply retrieves the data to be processed.</span></span>

```SQL
SELECT
soh.[SalesOrderNumber]  AS [OrderNumber],
soh.[Status]            AS [Status],
soh.[OrderDate]         AS [OrderDate],
soh.[DueDate]           AS [DueDate],
soh.[ShipDate]          AS [ShipDate],
soh.[SubTotal]          AS [SubTotal],
soh.[TaxAmt]            AS [TaxAmt],
soh.[TotalDue]          AS [TotalDue],
c.[AccountNumber]       AS [AccountNumber],
p.[Title]               AS [CustomerTitle],
p.[FirstName]           AS [CustomerFirstName],
p.[MiddleName]          AS [CustomerMiddleName],
p.[LastName]            AS [CustomerLastName],
p.[Suffix]              AS [CustomerSuffix],
sod.[OrderQty]          AS [Quantity],
sod.[UnitPrice]         AS [UnitPrice],
sod.[LineTotal]         AS [LineTotal],
sod.[ProductID]         AS [ProductId]
FROM [Sales].[SalesOrderHeader] soh
INNER JOIN [Sales].[Customer] c ON soh.[CustomerID] = c.[CustomerID]
INNER JOIN [Person].[Person] p ON c.[PersonID] = p.[BusinessEntityID]
INNER JOIN [Sales].[SalesOrderDetail] sod ON soh.[SalesOrderID] = sod.[SalesOrderID]
WHERE soh.[TerritoryId] = @TerritoryId
AND soh.[SalesOrderId] IN (
    SELECT TOP 20 SalesOrderId
    FROM [Sales].[SalesOrderHeader] soh
    WHERE soh.[TerritoryId] = @TerritoryId
    ORDER BY soh.[TotalDue] DESC)
ORDER BY soh.[TotalDue] DESC, sod.[SalesOrderDetailID]
```

<span data-ttu-id="5a96a-133">A continuación, la aplicación utiliza las API de .NET Framework `System.Xml.Linq` para dar formato a los resultados como XML.</span><span class="sxs-lookup"><span data-stu-id="5a96a-133">The application then uses the .NET Framework `System.Xml.Linq` APIs to format the results as XML.</span></span>

```csharp
// Create a new SqlCommand to run the Transact-SQL query
using (var command = new SqlCommand(...))
{
    command.Parameters.AddWithValue("@TerritoryId", id);

    // Run the query and create the initial XML document
    using (var reader = await command.ExecuteReaderAsync())
    {
        var lastOrderNumber = string.Empty;
        var doc = new XDocument();
        var orders = new XElement("Orders");
        doc.Add(orders);

        XElement lineItems = null;
        // Fetch each row in turn, format the results as XML, and add them to the XML document
        while (await reader.ReadAsync())
        {
            var orderNumber = reader["OrderNumber"].ToString();
            if (orderNumber != lastOrderNumber)
            {
                lastOrderNumber = orderNumber;

                var order = new XElement("Order");
                orders.Add(order);
                var customer = new XElement("Customer");
                lineItems = new XElement("OrderLineItems");
                order.Add(customer, lineItems);

                var orderDate = (DateTime)reader["OrderDate"];
                var totalDue = (Decimal)reader["TotalDue"];
                var reviewRequired = totalDue > 5000 ? 'Y' : 'N';

                order.Add(
                    new XAttribute("OrderNumber", orderNumber),
                    new XAttribute("Status", reader["Status"]),
                    new XAttribute("ShipDate", reader["ShipDate"]),
                    ... // More attributes, not shown.

                    var fullName = string.Join(" ",
                        reader["CustomerTitle"],
                        reader["CustomerFirstName"],
                        reader["CustomerMiddleName"],
                        reader["CustomerLastName"],
                        reader["CustomerSuffix"]
                    )
                   .Replace("  ", " ") //remove double spaces
                   .Trim()
                   .ToUpper();

               customer.Add(
                    new XAttribute("AccountNumber", reader["AccountNumber"]),
                    new XAttribute("FullName", fullName));
            }

            var productId = (int)reader["ProductID"];
            var quantity = (short)reader["Quantity"];
            var inventoryCheckRequired = (productId >= 710 && productId <= 720 && quantity >= 5) ? 'Y' : 'N';

            lineItems.Add(
                new XElement("LineItem",
                    new XAttribute("Quantity", quantity),
                    new XAttribute("UnitPrice", ((Decimal)reader["UnitPrice"]).ToString("C")),
                    new XAttribute("LineTotal", RoundAndFormat(reader["LineTotal"])),
                    new XAttribute("ProductId", productId),
                    new XAttribute("InventoryCheckRequired", inventoryCheckRequired)
                ));
        }
        // Match the exact formatting of the XML returned from SQL
        var xml = doc
            .ToString(SaveOptions.DisableFormatting)
            .Replace(" />", "/>");
    }
}
```

> [!NOTE]
> <span data-ttu-id="5a96a-134">Este código es bastante complejo.</span><span class="sxs-lookup"><span data-stu-id="5a96a-134">This code is somewhat complex.</span></span> <span data-ttu-id="5a96a-135">Para una nueva aplicación, es preferible utilizar una biblioteca de serialización.</span><span class="sxs-lookup"><span data-stu-id="5a96a-135">For a new application, you might prefer to use a serialization library.</span></span> <span data-ttu-id="5a96a-136">Sin embargo, aquí se supone que el equipo de desarrollo está refactorizando una aplicación existente, por lo que el método debe devolver exactamente el mismo formato que el código original.</span><span class="sxs-lookup"><span data-stu-id="5a96a-136">However, the assumption here is that the development team is refactoring an existing application, so the method needs to return the exact same format as the original code.</span></span>

## <a name="considerations"></a><span data-ttu-id="5a96a-137">Consideraciones</span><span class="sxs-lookup"><span data-stu-id="5a96a-137">Considerations</span></span>

- <span data-ttu-id="5a96a-138">Muchos sistemas de base de datos están muy optimizados para llevar a cabo ciertos tipos de procesamiento de datos, por ejemplo, calcular valores agregados en grandes conjuntos de datos.</span><span class="sxs-lookup"><span data-stu-id="5a96a-138">Many database systems are highly optimized to perform certain types of data processing, such as calculating aggregate values over large datasets.</span></span> <span data-ttu-id="5a96a-139">No saque de la base de datos esos tipos de procesamiento.</span><span class="sxs-lookup"><span data-stu-id="5a96a-139">Don't move those types of processing out of the database.</span></span>

- <span data-ttu-id="5a96a-140">No reubique el procesamiento si al hacerlo provoca que la base de datos transfiera más datos a través de la red.</span><span class="sxs-lookup"><span data-stu-id="5a96a-140">Do not relocate processing if doing so causes the database to transfer far more data over the network.</span></span> <span data-ttu-id="5a96a-141">Vea el [antipatrón Extraneous Fetching ][ExtraneousFetching].</span><span class="sxs-lookup"><span data-stu-id="5a96a-141">See the [Extraneous Fetching antipattern][ExtraneousFetching].</span></span>

- <span data-ttu-id="5a96a-142">Si pasa el procesamiento a un nivel de aplicación, ese nivel tiene que escalarse horizontalmente para controlar el trabajo adicional.</span><span class="sxs-lookup"><span data-stu-id="5a96a-142">If you move processing to an application tier, that tier may need to scale out to handle the additional work.</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="5a96a-143">Procedimiento para detectar el problema</span><span class="sxs-lookup"><span data-stu-id="5a96a-143">How to detect the problem</span></span>

<span data-ttu-id="5a96a-144">Los síntomas de una base de datos ocupada incluyen un descenso desproporcionado en los tiempos de respuesta y el rendimiento de las operaciones que tienen acceso a la base de datos.</span><span class="sxs-lookup"><span data-stu-id="5a96a-144">Symptoms of a busy database include a disproportionate decline in throughput and response times in operations that access the database.</span></span>

<span data-ttu-id="5a96a-145">Puede realizar los pasos siguientes para ayudar a identificar este problema:</span><span class="sxs-lookup"><span data-stu-id="5a96a-145">You can perform the following steps to help identify this problem:</span></span>

1. <span data-ttu-id="5a96a-146">Utilice la supervisión del rendimiento para identificar cuánto tiempo invierte el sistema de producción en la actividad de base de datos.</span><span class="sxs-lookup"><span data-stu-id="5a96a-146">Use performance monitoring to identify how much time the production system spends performing database activity.</span></span>

2. <span data-ttu-id="5a96a-147">Examine el trabajo realizado por la base de datos durante estos períodos.</span><span class="sxs-lookup"><span data-stu-id="5a96a-147">Examine the work performed by the database during these periods.</span></span>

3. <span data-ttu-id="5a96a-148">Si sospecha que operaciones determinadas podrían ocasionar demasiada actividad de base de datos, realice pruebas de carga en un entorno controlado.</span><span class="sxs-lookup"><span data-stu-id="5a96a-148">If you suspect that particular operations might cause too much database activity, perform load testing in a controlled environment.</span></span> <span data-ttu-id="5a96a-149">Cada prueba debe ejecutar una combinación de las operaciones sospechosas con una carga de usuarios variable.</span><span class="sxs-lookup"><span data-stu-id="5a96a-149">Each test should run a mixture of the suspect operations with a variable user load.</span></span> <span data-ttu-id="5a96a-150">Examine la telemetría de las pruebas de carga para observar cómo se utiliza la base de datos.</span><span class="sxs-lookup"><span data-stu-id="5a96a-150">Examine the telemetry from the load tests to observe how the database is used.</span></span>

4. <span data-ttu-id="5a96a-151">Si la actividad de la base de datos revela un procesamiento significativo pero poco tráfico de datos, revise el código fuente para determinar si el procesamiento puede realizarse mejor en otro lugar.</span><span class="sxs-lookup"><span data-stu-id="5a96a-151">If the database activity reveals significant processing but little data traffic, review the source code to determine whether the processing can better be performed elsewhere.</span></span>

<span data-ttu-id="5a96a-152">Si el volumen de actividad de la base de datos es bajo o los tiempos de respuesta son relativamente rápidos, no es probable que una base de datos ocupada constituya un problema de rendimiento.</span><span class="sxs-lookup"><span data-stu-id="5a96a-152">If the volume of database activity is low or response times are relatively fast, then a busy database is unlikely to be a performance problem.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="5a96a-153">Diagnóstico de ejemplo</span><span class="sxs-lookup"><span data-stu-id="5a96a-153">Example diagnosis</span></span>

<span data-ttu-id="5a96a-154">En las secciones siguientes se aplican estos pasos para la aplicación de ejemplo descrita anteriormente.</span><span class="sxs-lookup"><span data-stu-id="5a96a-154">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="monitor-the-volume-of-database-activity"></a><span data-ttu-id="5a96a-155">Supervisión del volumen de actividad de la base de datos</span><span class="sxs-lookup"><span data-stu-id="5a96a-155">Monitor the volume of database activity</span></span>

<span data-ttu-id="5a96a-156">El siguiente gráfico muestra los resultados de la ejecución de una prueba de carga en la aplicación de ejemplo, con una carga por pasos de hasta 50 usuarios simultáneos.</span><span class="sxs-lookup"><span data-stu-id="5a96a-156">The following graph shows the results of running a load test against the sample application, using a step load of up to 50 concurrent users.</span></span> <span data-ttu-id="5a96a-157">El volumen de solicitudes alcanza rápidamente el límite y permanece en ese nivel, mientras que el tiempo promedio de respuesta aumenta regularmente.</span><span class="sxs-lookup"><span data-stu-id="5a96a-157">The volume of requests quickly reaches a limit and stays at that level, while the average response time steadily increases.</span></span> <span data-ttu-id="5a96a-158">Tenga en cuenta que se utiliza una escala logarítmica para esas dos métricas.</span><span class="sxs-lookup"><span data-stu-id="5a96a-158">Note that a logarithmic scale is used for those two metrics.</span></span>

![Resultados de las pruebas de carga para realizar el procesamiento en la base de datos][ProcessingInDatabaseLoadTest]

<span data-ttu-id="5a96a-160">El gráfico siguiente muestra el uso de CPU y las DTU como un porcentaje de la cuota de servicio.</span><span class="sxs-lookup"><span data-stu-id="5a96a-160">The next graph shows CPU utilization and DTUs as a percentage of service quota.</span></span> <span data-ttu-id="5a96a-161">Las DTU proporciona una medida de cuánto procesamiento se realiza de la base de datos.</span><span class="sxs-lookup"><span data-stu-id="5a96a-161">DTUs provides a measure of how much processing the database performs.</span></span> <span data-ttu-id="5a96a-162">El gráfico muestra que tanto el uso de la CPU como el de DTU llegaron ambos rápidamente al 100 %.</span><span class="sxs-lookup"><span data-stu-id="5a96a-162">The graph shows that CPU and DTU utilization both quickly reached 100%.</span></span>

![Monitor de Azure SQL Database que muestra el rendimiento de la base de datos al realizar el procesamiento][ProcessingInDatabaseMonitor]

### <a name="examine-the-work-performed-by-the-database"></a><span data-ttu-id="5a96a-164">Examen del trabajo realizado por la base de datos</span><span class="sxs-lookup"><span data-stu-id="5a96a-164">Examine the work performed by the database</span></span>

<span data-ttu-id="5a96a-165">Es posible que las tareas realizadas por la base de datos sean operaciones de acceso a datos originales, en lugar de operaciones de procesamiento, por lo que es importante comprender las instrucciones SQL que se van a ejecutar mientras la base de datos está ocupada.</span><span class="sxs-lookup"><span data-stu-id="5a96a-165">It could be that the tasks performed by the database are genuine data access operations, rather than processing, so it is important to understand the SQL statements being run while the database is busy.</span></span> <span data-ttu-id="5a96a-166">Supervise el sistema para capturar el tráfico SQL y poner en correlación las operaciones SQL con las solicitudes de aplicación.</span><span class="sxs-lookup"><span data-stu-id="5a96a-166">Monitor the system to capture the SQL traffic and correlate the SQL operations with application requests.</span></span>

<span data-ttu-id="5a96a-167">Si las operaciones de base de datos son estrictamente de acceso a datos, sin una gran cantidad de procesamiento, entonces el problema podría ser el antipatrón [Extraneous Fetching ][ExtraneousFetching].</span><span class="sxs-lookup"><span data-stu-id="5a96a-167">If the database operations are purely data access operations, without a lot of processing, then the problem might be [Extraneous Fetching][ExtraneousFetching].</span></span>

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="5a96a-168">Implementación de la solución y comprobación del resultado</span><span class="sxs-lookup"><span data-stu-id="5a96a-168">Implement the solution and verify the result</span></span>

<span data-ttu-id="5a96a-169">El gráfico siguiente muestra una prueba de carga con el código actualizado.</span><span class="sxs-lookup"><span data-stu-id="5a96a-169">The following graph shows a load test using the updated code.</span></span> <span data-ttu-id="5a96a-170">El rendimiento es considerablemente mayor, de más de 400 solicitudes por segundo frente a las 12 anteriores.</span><span class="sxs-lookup"><span data-stu-id="5a96a-170">Throughput is significantly higher, over 400 requests per second versus 12 earlier.</span></span> <span data-ttu-id="5a96a-171">El tiempo promedio de respuesta también es mucho menor, justo por encima de 0,1 segundos en comparación con más de 4 segundos.</span><span class="sxs-lookup"><span data-stu-id="5a96a-171">The average response time is also much lower, just above 0.1 seconds compared to over 4 seconds.</span></span>

![Resultados de las pruebas de carga para realizar el procesamiento en la base de datos][ProcessingInClientApplicationLoadTest]

<span data-ttu-id="5a96a-173">El uso de CPU y de DTU muestra que el sistema tardó más tiempo en alcanzar la saturación, a pesar del aumento de rendimiento.</span><span class="sxs-lookup"><span data-stu-id="5a96a-173">CPU and DTU utilization shows that the system took longer to reach saturation, despite the increased throughput.</span></span>

![Monitor de Azure SQL Database que muestra el rendimiento de la base de datos al realizar el procesamiento en la aplicación cliente][ProcessingInClientApplicationMonitor]

## <a name="related-resources"></a><span data-ttu-id="5a96a-175">Recursos relacionados</span><span class="sxs-lookup"><span data-stu-id="5a96a-175">Related resources</span></span>

- <span data-ttu-id="5a96a-176">[Antipatrón Extraneous Fetching][ExtraneousFetching]</span><span class="sxs-lookup"><span data-stu-id="5a96a-176">[Extraneous Fetching antipattern][ExtraneousFetching]</span></span>

[dtu]: /azure/sql-database/sql-database-service-tiers-dtu
[ExtraneousFetching]: ../extraneous-fetching/index.md
[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/BusyDatabase

[ProcessingInDatabaseLoadTest]: ./_images/ProcessingInDatabaseLoadTest.jpg
[ProcessingInClientApplicationLoadTest]: ./_images/ProcessingInClientApplicationLoadTest.jpg
[ProcessingInDatabaseMonitor]: ./_images/ProcessingInDatabaseMonitor.jpg
[ProcessingInClientApplicationMonitor]: ./_images/ProcessingInClientApplicationMonitor.jpg
