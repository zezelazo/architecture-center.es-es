---
title: Antipatrón Busy Database
description: Descargar el procesamiento en un servidor de base de datos puede provocar problemas de rendimiento y escalabilidad.
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: a14a350aefc1801ae08cb4a8d0eb3d5b248c92bf
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/28/2018
ms.locfileid: "47428914"
---
# <a name="busy-database-antipattern"></a>Antipatrón Busy Database

Descargar el procesamiento en un servidor de base de datos puede provocar que se dedique una proporción considerable de tiempo en ejecutar el código, en lugar de responder a las solicitudes para almacenar y recuperar datos. 

## <a name="problem-description"></a>Descripción del problema

Muchos sistemas de base de datos pueden ejecutar código. Algunos ejemplos son los procedimientos almacenados y los desencadenadores. A menudo, es más eficaz realizar este procesamiento cerca de los datos, en lugar de transmitirlos a una aplicación cliente para ser procesados. Sin embargo, el uso excesivo de estas características puede afectar al rendimiento, por varias razones:

- El servidor de base de datos puede dedicar demasiado tiempo en el procesamiento, en lugar de aceptar nuevas solicitudes de cliente y capturar los datos.
- Una base de datos suele ser un recurso compartido, por lo que puede convertirse en un cuello de botella durante los períodos de uso elevado.
- Los costos del entorno de tiempo de ejecución pueden ser excesivos si se mide el almacén de datos. Eso es especialmente cierto en los servicios administrados de la base de datos. Por ejemplo, Azure SQL Dabatase cobra por las [unidades de transacción de base de datos] [ dtu] (DTU).
- Las bases de datos tienen una capacidad limitada para escalarse verticalmente y escalarlas horizontalmente no resulta trivial. Por lo tanto, es posible que convenga mover el procesamiento a un recurso de proceso, como una aplicación de App Service o una máquina virtual, que se puede escalar fácilmente.

Este antipatrón suele ocurrir porque:

- La base de datos se ve como un servicio en lugar de un repositorio. Una aplicación puede utilizar el servidor de base de datos para dar formato a loa datos (por ejemplo, convertirlos en XML), manipular datos de cadena o realizar cálculos complejos.
- Los desarrolladores intentan escribir las consultas cuyos resultados se pueden mostrar directamente a los usuarios. Por ejemplo, una consulta puede combinar campos o dar formato a fechas, horas y monedas según la configuración regional.
- Los desarrolladores intentan corregir el antipatrón [Extraneous Fetching] [ExtraneousFetching] insertando cálculos en la base de datos.
- Los procedimientos almacenados se usan para encapsular la lógica de negocios, quizás porque se consideran más fáciles de mantener y actualizar.

En el ejemplo siguiente se recuperan los 20 pedidos más valiosos de una zona de ventas especificada y se da formato a los resultados como XML.
Usa las funciones de Transact-SQL para analizar los datos y convertir los resultados en XML. Puede encontrar el ejemplo completo [aquí][sample-app].

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

Claramente, se trata de una consulta compleja. Como veremos más adelante, termina por usar recursos de procesamiento significativos en el servidor de base de datos.

## <a name="how-to-fix-the-problem"></a>Procedimiento para corregir el problema

Pase el procesamiento del servidor de base de datos a otros niveles de aplicación. Lo mejor sería limitar la base de datos para realizar las operaciones de acceso a datos, utilizando solo las funcionalidades para las que la base de datos esté optimizada, por ejemplo, la agregación en un RDBMS.

Por ejemplo, el código de Transact-SQL anterior puede sustituirse por una instrucción que simplemente recupere los datos que se van a procesar.

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

A continuación, la aplicación utiliza las API de .NET Framework `System.Xml.Linq` para dar formato a los resultados como XML.

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
> Este código es bastante complejo. Para una nueva aplicación, es preferible utilizar una biblioteca de serialización. Sin embargo, aquí se supone que el equipo de desarrollo está refactorizando una aplicación existente, por lo que el método debe devolver exactamente el mismo formato que el código original.

## <a name="considerations"></a>Consideraciones

- Muchos sistemas de base de datos están muy optimizados para llevar a cabo ciertos tipos de procesamiento de datos, por ejemplo, calcular valores agregados en grandes conjuntos de datos. No saque de la base de datos esos tipos de procesamiento.

- No reubique el procesamiento si al hacerlo provoca que la base de datos transfiera más datos a través de la red. Vea el [antipatrón Extraneous Fetching ][ExtraneousFetching].

- Si pasa el procesamiento a un nivel de aplicación, ese nivel tiene que escalarse horizontalmente para controlar el trabajo adicional.

## <a name="how-to-detect-the-problem"></a>Procedimiento para detectar el problema

Los síntomas de una base de datos ocupada incluyen un descenso desproporcionado en los tiempos de respuesta y el rendimiento de las operaciones que tienen acceso a la base de datos. 

Puede realizar los pasos siguientes para ayudar a identificar este problema: 

1. Utilice la supervisión del rendimiento para identificar cuánto tiempo invierte el sistema de producción en la actividad de base de datos.

2. Examine el trabajo realizado por la base de datos durante estos períodos.

3. Si sospecha que operaciones determinadas podrían ocasionar demasiada actividad de base de datos, realice pruebas de carga en un entorno controlado. Cada prueba debe ejecutar una combinación de las operaciones sospechosas con una carga de usuarios variable. Examine la telemetría de las pruebas de carga para observar cómo se utiliza la base de datos.

4. Si la actividad de la base de datos revela un procesamiento significativo pero poco tráfico de datos, revise el código fuente para determinar si el procesamiento puede realizarse mejor en otro lugar.

Si el volumen de actividad de la base de datos es bajo o los tiempos de respuesta son relativamente rápidos, no es probable que una base de datos ocupada constituya un problema de rendimiento.

## <a name="example-diagnosis"></a>Diagnóstico de ejemplo

En las secciones siguientes se aplican estos pasos para la aplicación de ejemplo descrita anteriormente.

### <a name="monitor-the-volume-of-database-activity"></a>Supervisión del volumen de actividad de la base de datos

El siguiente gráfico muestra los resultados de la ejecución de una prueba de carga en la aplicación de ejemplo, con una carga por pasos de hasta 50 usuarios simultáneos. El volumen de solicitudes alcanza rápidamente el límite y permanece en ese nivel, mientras que el tiempo promedio de respuesta aumenta regularmente. Tenga en cuenta que se utiliza una escala logarítmica para esas dos métricas.

![Resultados de las pruebas de carga para realizar el procesamiento en la base de datos][ProcessingInDatabaseLoadTest]

El gráfico siguiente muestra el uso de CPU y las DTU como un porcentaje de la cuota de servicio. Las DTU proporciona una medida de cuánto procesamiento se realiza de la base de datos. El gráfico muestra que tanto el uso de la CPU como el de DTU llegaron ambos rápidamente al 100 %.

![Monitor de Azure SQL Database que muestra el rendimiento de la base de datos al realizar el procesamiento][ProcessingInDatabaseMonitor]

### <a name="examine-the-work-performed-by-the-database"></a>Examen del trabajo realizado por la base de datos

Es posible que las tareas realizadas por la base de datos sean operaciones de acceso a datos originales, en lugar de operaciones de procesamiento, por lo que es importante comprender las instrucciones SQL que se van a ejecutar mientras la base de datos está ocupada. Supervise el sistema para capturar el tráfico SQL y poner en correlación las operaciones SQL con las solicitudes de aplicación.

Si las operaciones de base de datos son estrictamente de acceso a datos, sin una gran cantidad de procesamiento, entonces el problema podría ser el antipatrón [Extraneous Fetching ][ExtraneousFetching].

### <a name="implement-the-solution-and-verify-the-result"></a>Implementación de la solución y comprobación del resultado

El gráfico siguiente muestra una prueba de carga con el código actualizado. El rendimiento es considerablemente mayor, de más de 400 solicitudes por segundo frente a las 12 anteriores. El tiempo promedio de respuesta también es mucho menor, justo por encima de 0,1 segundos en comparación con más de 4 segundos.

![Resultados de las pruebas de carga para realizar el procesamiento en la base de datos][ProcessingInClientApplicationLoadTest]

El uso de CPU y de DTU muestra que el sistema tardó más tiempo en alcanzar la saturación, a pesar del aumento de rendimiento.

![Monitor de Azure SQL Database que muestra el rendimiento de la base de datos al realizar el procesamiento en la aplicación cliente][ProcessingInClientApplicationMonitor]

## <a name="related-resources"></a>Recursos relacionados 

- [Antipatrón Extraneous Fetching][ExtraneousFetching]


[dtu]: /azure/sql-database/sql-database-service-tiers-dtu
[ExtraneousFetching]: ../extraneous-fetching/index.md
[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/BusyDatabase

[ProcessingInDatabaseLoadTest]: ./_images/ProcessingInDatabaseLoadTest.jpg
[ProcessingInClientApplicationLoadTest]: ./_images/ProcessingInClientApplicationLoadTest.jpg
[ProcessingInDatabaseMonitor]: ./_images/ProcessingInDatabaseMonitor.jpg
[ProcessingInClientApplicationMonitor]: ./_images/ProcessingInClientApplicationMonitor.jpg
