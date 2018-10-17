---
title: Inteligencia empresarial automatizada con SQL Data Warehouse y Azure Data Factory
description: Automatizar un flujo de trabajo de ECT en Azure mediante Azure Data Factory
author: MikeWasson
ms.date: 07/01/2018
ms.openlocfilehash: f004c02da93335e74b07b9720236832ad7f744db
ms.sourcegitcommit: 62945777e519d650159f0f963a2489b6bb6ce094
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/09/2018
ms.locfileid: "48876909"
---
# <a name="automated-enterprise-bi-with-sql-data-warehouse-and-azure-data-factory"></a>Inteligencia empresarial automatizada con SQL Data Warehouse y Azure Data Factory

Esta arquitectura de referencia muestra cómo realizar una carga incremental en una canalización de [ECT](../../data-guide/relational-data/etl.md#extract-load-and-transform-elt) (extracción, carga y transformación). Usa Azure Data Factory para automatizar la canalización de ECT. La canalización mueve de forma incremental los datos más recientes de OLTP de una base de datos de SQL Server local a SQL Data Warehouse. Los datos transaccionales se transforman en un modelo tabular para su análisis. [**Implemente esta solución**.](#deploy-the-solution)

![](./images/enterprise-bi-sqldw-adf.png)

Esta arquitectura se basa en la que se muestra en [Inteligencia empresarial con SQL Data Warehouse](./enterprise-bi-sqldw.md), pero agrega algunas características que son importantes para los escenarios de almacenamiento de datos empresariales.

-   Automatización de la canalización mediante Data Factory.
-   Carga incremental.
-   Integración de varios orígenes de datos.
-   Carga de datos binarios como datos geoespaciales e imágenes.

## <a name="architecture"></a>Arquitectura

La arquitectura consta de los siguientes componentes:

### <a name="data-sources"></a>Orígenes de datos

**SQL Server local**. Los datos de origen se encuentran en una base de datos de SQL Server de forma local. Para simular el entorno local, los scripts de implementación para esta arquitectura aprovisionan una máquina virtual en Azure con SQL Server instalado. La [base de datos OLTP de ejemplo de OLTP Wide World Importers] [wwi] se usa como base de datos de origen.

**Datos externos**. Un escenario común para el almacenamiento de datos es integrar varios orígenes de datos. Esta arquitectura de referencia carga un conjunto de datos externo que contiene las poblaciones de las ciudades por año y lo integra con los datos de la base de datos de OLTP. Estos datos se pueden usar para conclusiones como: "¿Supera o iguala el crecimiento de las ventas en cada región el crecimiento poblacional?"

### <a name="ingestion-and-data-storage"></a>Ingesta y almacenamiento de datos

**Blob Storage**. Blob Storage se utiliza como área de ensayo del origen de datos antes de cargarlos en SQL Data Warehouse.

**Azure SQL Data Warehouse**. [SQL Data Warehouse](/azure/sql-data-warehouse/) es un sistema distribuido diseñado para realizar análisis con datos de gran tamaño. Admite el procesamiento paralelo masivo (MPP), lo que lo hace idóneo para ejecutar análisis de alto rendimiento. 

**Azure Data Factory**. [Data Factory] [adf] es un servicio administrado que organiza y automatiza el movimiento y la transformación de datos. En esta arquitectura, coordina las distintas fases del proceso de ELT.

### <a name="analysis-and-reporting"></a>Análisis e informes

**Azure Analysis Services**. [Analysis Services](/azure/analysis-services/) es un servicio completamente administrado que proporciona funcionalidades de modelado de datos. El modelo semántico se carga en Analysis Services.

**Power BI**. Power BI es un conjunto de herramientas de análisis de negocios que sirve para analizar datos con el fin de obtener perspectivas empresariales. En el caso de esta arquitectura consulta el modelo semántico almacenado en Analysis Services.

### <a name="authentication"></a>Autenticación

**Azure Active Directory** (Azure AD) autentica a los usuarios que se conectan al servidor de Analysis Services mediante Power BI.

Data Factory puede usar también Azure AD para autenticarse en SQL Data Warehouse mediante el uso de una entidad de servicio o de Managed Service Identity (MSI). Por simplicidad, la implementación de ejemplo utiliza la autenticación de SQL Server.

## <a name="data-pipeline"></a>Canalización de datos

En [Azure Data Factory] [adf], una canalización es una agrupación lógica de actividades que se usa para coordinar una tarea (en este caso, cargar y transformar los datos en SQL Data Warehouse). 

Esta arquitectura de referencia define una canalización maestra que ejecuta una secuencia de canalizaciones de secundarias. Cada canalización secundaria carga datos en una o varias tablas de un almacén de datos.

![](./images/adf-pipeline.png)

## <a name="incremental-loading"></a>Carga incremental

Cuando se ejecuta un proceso automatizado de ETL o ELT, resulta más eficaz cargar solo los datos que han cambiado desde la última vez que se ejecutó. Esto se denomina una *carga incremental*, frente a una carga completa, en la que se cargan todos los datos. Para realizar una carga incremental, se necesita alguna forma de identificar qué datos han cambiado. El método más común es usar un valor *de marca de límite superior*, lo que significa que se hace un seguimiento del valor más reciente de alguna de las columnas de la tabla de origen, una columna de fecha y hora o una columna de entero único. 

A partir de SQL Server 2016, se pueden usar las [tablas temporales](/sql/relational-databases/tables/temporal-tables), que son tablas con versiones de sistema que conservan el historial completo de los cambios de datos. El motor de base de datos registra automáticamente el historial de cada cambio en una tabla de historial independiente. Para consultar los datos históricos hay que agregar una cláusula FOR SYSTEM_TIME a una consulta. Internamente, el motor de base de datos consulta la tabla del historial, pero la aplicación no se percata de ello. 

> [!NOTE]
> Para las versiones anteriores de SQL Server, puede usar [captura de datos modificados](/sql/relational-databases/track-changes/about-change-data-capture-sql-server) (CDC). Este método es menos práctico que las tablas temporales, ya que hay que consultar una tabla de cambios independiente y el seguimiento de los cambios se realiza por un número de secuencia de registro, en lugar de una marca de tiempo. 

Las tablas temporales son útiles para los datos de dimensión, que pueden cambiar con el tiempo. Las tablas de hechos suele representar una transacción inmutable, como por ejemplo una venta; en ese caso no tiene sentido mantener el historial de versiones del sistema. En su lugar, las transacciones suelen tener una columna que representa la fecha de la transacción, que se puede usar como valor de marca de agua. Por ejemplo, en la base de datos OLTP Wide World Importers, las tablas Sales.Invoices y Sales.InvoiceLines tienen un campo `LastEditedWhen` cuyo valor predeterminado es `sysdatetime()`. 

Este es el flujo general de la canalización de ELT:

1. En todas las tablas de la base de datos de origen, realice un seguimiento de la hora límite en que se ejecutó el último trabajo de ELT. Almacene esta información en la base de datos de almacenamiento de datos (en la instalación inicial, todas las horas se establecen en el "1-1-1900").

2. Durante el paso de exportación de datos, la hora límite se pasa como parámetro a un conjunto de procedimientos almacenados de la base de datos de origen. Dichos procedimientos consultan todos los registros que se cambiaron o crearon después de la hora límite. Para la tabla de hechos Sales, se usa la columna `LastEditedWhen`. Para los datos de dimensiones, se usan tablas temporales con la versión del sistema.

3. Una vez completada la migración de los datos, actualice la tabla que almacena las horas límite.

También es útil registrar un *linaje* para cada ejecución de ELT. En el caso de un registro concreto, el linaje asocia dicho registro con la ejecución de ELT que generó los datos. En cada ejecución de ETL, se crea un nuevo registro de linaje para todas las tablas, en el que se muestran la hora inicial y final de la carga. Las claves del linaje de los registros se almacenan en tablas de hechos y de dimensiones.

![](./images/city-dimension-table.png)

Después cargar un nuevo lote de datos en el almacén, actualice el modelo tabular de Analysis Services. Consulte [Actualización asincrónica con la API REST](/azure/analysis-services/analysis-services-async-refresh).

## <a name="data-cleansing"></a>Limpieza de datos

La limpieza de los datos debe formar parte del proceso de ELT. En esta arquitectura de referencia, un origen de datos incorrectos es la tabla de población de ciudades, donde algunas ciudades tienen una población cero, quizás porque no había datos disponibles. Durante el procesamiento, la canalización de ELT quita esas ciudades de la tabla de la población de ciudades. La limpieza de datos se debe realizar en las tablas de almacenamiento provisional, no en las tablas externas.

Este es el procedimiento almacenado que elimina las ciudades con población cero rellenado de la tabla City Population (el archivo de origen se puede encontrar [aquí](https://github.com/mspnp/reference-architectures/blob/master/data/enterprise_bi_sqldw_advanced/azure/sqldw_scripts/citypopulation/%5BIntegration%5D.%5BMigrateExternalCityPopulationData%5D.sql)). 

```sql
DELETE FROM [Integration].[CityPopulation_Staging]
WHERE RowNumber in (SELECT DISTINCT RowNumber
FROM [Integration].[CityPopulation_Staging]
WHERE POPULATION = 0
GROUP BY RowNumber
HAVING COUNT(RowNumber) = 4)
```

## <a name="external-data-sources"></a>Orígenes de datos externos

Las bases de datos de almacenamiento de datos a menudo consolidan datos de varios orígenes. Esta arquitectura de referencia carga un origen de datos externo que contiene datos demográficos. Este conjunto de datos está disponible en Azure Blob Storage como parte del ejemplo [WorldWideImportersDW](https://github.com/Microsoft/sql-server-samples/tree/master/samples/databases/wide-world-importers/sample-scripts/polybase).

Azure Data Factory puede realizar la copia directamente desde Blob Storage, mediante el [conector de Blob Storage](/azure/data-factory/connector-azure-blob-storage). Sin embargo, el conector requiere una cadena de conexión o una firma de acceso compartido, por lo que no se puede usar para copiar un blob con acceso de lectura público. Como alternativa, puede usar PolyBase para crear una tabla externa a través de Blob Storage y, después, copiar las tablas externas en SQL Data Warehouse. 

## <a name="handling-large-binary-data"></a>Control de datos binarios de gran tamaño 

En la base de datos de origen, la tabla Cities tiene una columna Location que contiene un tipo de datos espaciales [geography](/sql/t-sql/spatial-geography/spatial-types-geography). De forma nativa SQL Data Warehouse no admite el tipo **geography**, por lo que este campo pasa a ser de tipo **varbinary** durante la carga (consulte [Soluciones alternativas para los tipos de datos no admitidos](/azure/sql-data-warehouse/sql-data-warehouse-tables-data-types#unsupported-data-types)).

Sin embargo, PolyBase admite un tamaño de columna máximo de `varbinary(8000)`, lo que significa que algunos datos podrían aparecer truncados. Una solución alternativa a este problema es dividir los datos en fragmentos durante la exportación y, después, ensamblar dichos fragmentos como se indica a continuación:

1. Cree una tabla de almacenamiento provisional para la columna Location.

2. En cada ciudad, divida los datos de ubicación en fragmentos de 8000 bytes, lo que da como resultado 1 &ndash; N filas por cada ciudad.

3. Para ensamblar los fragmentos, use el operador [PIVOT](/sql/t-sql/queries/from-using-pivot-and-unpivot) de T-SQL para convertir las filas en columnas y, después, concatene los valores de columna de cada ciudad.

El desafío es que cada ciudad se divida en un número diferente de filas, en función del tamaño de los datos geográficos. Para que el operador PIVOT funcione, todas las ciudades debe tener el mismo número de filas. Para que esto funcione, la consulta de T-SQL (que se puede ver [aquí][MergeLocation]) realiza algunos trucos para rellenar las filas con valores en blanco, con el fin de que todas las ciudades tengan el mismo número de columnas después de la dinamización. La consulta resultante resulta ser mucho más rápida que crear bucles en las filas de una en una.

Pata los datos de imagen se usa el mismo método.

## <a name="slowly-changing-dimensions"></a>Cambio lento de dimensiones

Los datos de dimensiones son relativamente estáticos, pero se pueden cambiar. Por ejemplo, un producto se puede reasignar a otra categoría. Hay varios métodos para el control del cambio lento de dimensiones. Una técnica común, llamada de [tipo 2](https://wikipedia.org/wiki/Slowly_changing_dimension#Type_2:_add_new_row), consiste en agregar un nuevo registro cada vez que cambia de una dimensión. 

Para implementar este método, las tablas de dimensiones necesitan columnas adicionales que especifiquen el intervalo de fechas de vigencia de un registro determinado. Además, las claves principales de la base de datos de origen se duplicarán, por lo que la tabla de dimensiones debe tener una clave principal artificial.

La siguiente imagen muestra la tabla Dimension.City. La columna `WWI City ID` es la clave principal de la base de datos de origen. La columna `City Key` es una clave artificial generada durante la canalización de ETL. Observe también que la tabla tiene las columnas `Valid From` y `Valid To`, que definen el intervalo de validez de cada fila. El valor de `Valid To` de los valores actuales es "9999-12-31".

![](./images/city-dimension-table.png)

La ventaja de este método es que conserva los datos históricos, lo que puede resultar muy útil de cara al análisis. Sin embargo, también significa que habrá varias filas para la misma entidad. Por ejemplo, estos son los registros que coinciden con `WWI City ID` = 28561:

![](./images/city-dimension-table-2.png)

Para cada dato de Sales, desea asociar dicho hecho a una sola fila de la tabla de dimensiones City, correspondiente a la fecha de factura. Como parte del proceso de ETL, cree una columna. 

La siguiente consulta de T-SQL crea una tabla temporal que asocia cada factura a la clave de ciudad correcta de la tabla de dimensiones City.

```sql
CREATE TABLE CityHolder
WITH (HEAP , DISTRIBUTION = HASH([WWI Invoice ID]))
AS
SELECT DISTINCT s1.[WWI Invoice ID] AS [WWI Invoice ID],
                c.[City Key] AS [City Key]
    FROM [Integration].[Sale_Staging] s1
    CROSS APPLY (
                SELECT TOP 1 [City Key]
                    FROM [Dimension].[City]
                WHERE [WWI City ID] = s1.[WWI City ID]
                    AND s1.[Last Modified When] > [Valid From]
                    AND s1.[Last Modified When] <= [Valid To]
                ORDER BY [Valid From], [City Key] DESC
                ) c

```

Dicha tabla se utiliza para rellenar una columna de la tabla de hechos Sales:

```sql
UPDATE [Integration].[Sale_Staging]
SET [Integration].[Sale_Staging].[WWI Customer ID] =  CustomerHolder.[WWI Customer ID]
```

Esta columna permite que una consulta de Power BI encuentre el registro de City correcto de una factura de venta determinada.

## <a name="security-considerations"></a>Consideraciones sobre la seguridad

Para mayor seguridad, puede usar los [puntos de conexión de Virtual Network](/azure/virtual-network/virtual-network-service-endpoints-overview) para proteger los recursos del servicio de Azure a solo la red virtual. Esto elimina por completo el acceso público a Internet de esos recursos, solo permite el tráfico solo desde la red virtual.

Con este método se crea una red virtual en Azure y, después, se crean puntos de conexión de servicio privados para los servicios de Azure. Luego se aplica una restricción a dichos servicios, por lo que solo le llega el tráfico de la red virtual. También se puede acceder ellos desde la red local a través de una puerta de enlace.

Tenga en cuenta las siguientes limitaciones:

- En el momento de creación de esta arquitectura de referencia, los puntos de conexión de servicio de red virtual se admiten en Azure Storage y Azure SQL Data Warehouse, pero no en Azure Analysis Service. [Aquí](https://azure.microsoft.com/updates/?product=virtual-network) puede comprobar el estado más reciente. 

- Si se habilitan los puntos de conexión de servicio para Azure Storage, PolyBase no puede copiar datos de Storage a SQL Data Warehouse. Pero este problema se puede mitigar. Para más información, consulte [Efectos del uso de puntos de conexión de servicio de la red virtual con Azure Storage](/azure/sql-database/sql-database-vnet-service-endpoint-rule-overview?toc=%2fazure%2fvirtual-network%2ftoc.json#impact-of-using-vnet-service-endpoints-with-azure-storage). 

- Para mover datos desde el entorno local a Azure Storage, será preciso incorporar a la lista blanca las direcciones IP públicas desde su entorno local o desde ExpressRoute. Para más información, consulte [Protección de servicios de Azure para las redes virtuales](/azure/virtual-network/virtual-network-service-endpoints-overview#securing-azure-services-to-virtual-networks).

- Para permitir que Analysis Services lea datos de SQL Data Warehouse, implemente una máquina virtual Windows en la red virtual que contiene el punto de conexión de servicio de SQL Data Warehouse. Instale la [puerta de enlace de datos local de Azure](/azure/analysis-services/analysis-services-gateway) en esta máquina virtual. Luego, conecte el servicio Azure Analysis a dicha puerta de enlace.

## <a name="deploy-the-solution"></a>Implementación de la solución

Hay disponible una implementación para esta arquitectura de referencia en [GitHub][ref-arch-repo-folder]. Implementa lo siguiente:

  * Una máquina virtual Windows para simular un servidor de bases de datos local. Incluye SQL Server 2017 y herramientas relacionadas, junto con Power BI Desktop.
  * Una cuenta de almacenamiento de Azure que proporciona almacenamiento de blobs para almacenar los datos exportados de la base de datos de SQL Server.
  * Una instancia de Azure SQL Data Warehouse.
  * Una instancia de Azure Analysis Services.
  * Azure Data Factory y la canalización de Data Factory para el trabajo de ELT.

### <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="variables"></a>variables

Los pasos siguientes incluyen algunas variables definidas por el usuario. Deberá reemplazarlas por los valores que defina.

- `<data_factory_name>`. Nombre de Data Factory.
- `<analysis_server_name>`. Nombre del servidor de Analysis Services.
- `<active_directory_upn>`. Nombre principal de usuario (UPN) de Azure Active Directory. Por ejemplo, `user@contoso.com`.
- `<data_warehouse_server_name>`. Nombre de servidor de SQL Data Warehouse.
- `<data_warehouse_password>`. Contraseña de administrador de SQL Data Warehouse.
- `<resource_group_name>`. Nombre del grupo de recursos.
- `<region>`. La región de Azure en la que se implementarán los recursos.
- `<storage_account_name>`. Nombre de la cuenta de almacenamiento. Debe seguir las [reglas de nomenclatura](../../best-practices/naming-conventions.md#naming-rules-and-restrictions) de las cuentas de almacenamiento.
- `<sql-db-password>`. Contraseña de inicio de sesión de SQL Server.

### <a name="deploy-azure-data-factory"></a>Implementación de Azure Data Factory

1. Navegue a la carpeta `data\enterprise_bi_sqldw_advanced\azure\templates` del [repositorio de GitHub][ref-arch-repo].

2. Ejecute el siguiente comando de la CLI de Azure para crear un grupo de recursos.  

    ```bash
    az group create --name <resource_group_name> --location <region>  
    ```

    Especifique una región que admita SQL Data Warehouse, Azure Analysis Services y Data Factory v2. Consulte [Productos de Azure por región](https://azure.microsoft.com/global-infrastructure/services/)

3. Ejecute el siguiente comando.

    ```
    az group deployment create --resource-group <resource_group_name> \
        --template-file adf-create-deploy.json \
        --parameters factoryName=<data_factory_name> location=<location>
    ```

A continuación, use Azure Portal para obtener la clave de autenticación del [entorno de ejecución de integración](/azure/data-factory/concepts-integration-runtime) de Azure Data Factory, como se indica a continuación:

1. En [Azure Portal](https://portal.azure.com/), navegue hasta la instancia de Data Factory.

2. En la hoja Data Factory, haga clic en **Crear y supervisar**. Se abre el portal de Azure Data Factory en otra ventana del explorador.

    ![](./images/adf-blade.png)

3. En dicho portal, seleccione el icono del lápiz ("Crear"). 

4. Haga clic en **Connections** (Conexiones) y seleccione **Integration Runtimes** (Entornos de ejecución de integración).

5. En **sourceIntegrationRuntime**, haga clic en el icono del lápiz ("Editar").

    > [!NOTE]
    > El portal mostrará el estado como "no disponible". Aparecerá hasta que se implemente el servidor local.

6. Busque **Key1** y copie el valor de la clave de autenticación,

ya que lo necesitará para el paso siguiente.

### <a name="deploy-the-simulated-on-premises-server"></a>Implementación del servidor local simulado

En este paso se implementa una máquina virtual como un servidor local simulado, que incluye SQL Server 2017 y las herramientas relacionadas. También se carga la [base de datos OLTP Wide World Importers][wwi] en SQL Server.

1. Vaya a la carpeta `data\enterprise_bi_sqldw_advanced\onprem\templates` del repositorio.

2. En el archivo `onprem.parameters.json`, busque `adminPassword`. Esta es la contraseña para iniciar sesión en la máquina virtual de SQL Server. Reemplace el valor por otra contraseña.

3. En el mismo archivo, busque `SqlUserCredentials`. Esta propiedad especifica las credenciales de la cuenta de SQL Server. Reemplace la contraseña por otra.

4. En el mismo archivo, pegue la clave de autenticación de Integration Runtime en el parámetro `IntegrationRuntimeGatewayKey`, como se muestra a continuación:

    ```json
    "protectedSettings": {
        "configurationArguments": {
            "SqlUserCredentials": {
                "userName": ".\\adminUser",
                "password": "<sql-db-password>"
            },
            "IntegrationRuntimeGatewayKey": "<authentication key>"
        }
    ```

5. Ejecute el siguiente comando.

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <region> -p onprem.parameters.json --deploy
    ```

Este paso puede tardar entre 20 y 30 minutos en completarse. Incluye la ejecución de un script de [DSC](/powershell/dsc/overview) para instalar las herramientas y restaurar la base de datos. 

### <a name="deploy-azure-resources"></a>Implementación de recursos de Azure

En este paso se aprovisionan SQL Data Warehouse, Azure Analysis Services y Data Factory v2.

1. Navegue a la carpeta `data\enterprise_bi_sqldw_advanced\azure\templates` del [repositorio de GitHub][ref-arch-repo].

2. Ejecute el siguiente comando de la CLI de Azure. Reemplace los valores del parámetro que se muestran entre paréntesis angulares.

    ```bash
    az group deployment create --resource-group <resource_group_name> \
     --template-file azure-resources-deploy.json \
     --parameters "dwServerName"="<data_warehouse_server_name>" \
     "dwAdminLogin"="adminuser" "dwAdminPassword"="<data_warehouse_password>" \ 
     "storageAccountName"="<storage_account_name>" \
     "analysisServerName"="<analysis_server_name>" \
     "analysisServerAdmin"="<user@contoso.com>"
    ```

    - El parámetro `storageAccountName` tiene que seguir las [reglas de nomenclatura](../../best-practices/naming-conventions.md#naming-rules-and-restrictions) para las cuentas de almacenamiento. 
    - Para el parámetro `analysisServerAdmin`, use el nombre principal de usuario (UPN) de Azure Active Directory.

3. Ejecute el siguiente comando de la CLI de Azure para obtener la clave de acceso a la cuenta de almacenamiento. Dicha clave se usará en el paso siguiente.

    ```bash
    az storage account keys list -n <storage_account_name> -g <resource_group_name> --query [0].value
    ```

4. Ejecute el siguiente comando de la CLI de Azure. Reemplace los valores del parámetro que se muestran entre paréntesis angulares. 

    ```bash
    az group deployment create --resource-group <resource_group_name> \
    --template-file adf-pipeline-deploy.json \
    --parameters "factoryName"="<data_factory_name>" \
    "sinkDWConnectionString"="Server=tcp:<data_warehouse_server_name>.database.windows.net,1433;Initial Catalog=wwi;Persist Security Info=False;User ID=adminuser;Password=<data_warehouse_password>;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;" \
    "blobConnectionString"="DefaultEndpointsProtocol=https;AccountName=<storage_account_name>;AccountKey=<storage_account_key>;EndpointSuffix=core.windows.net" \
    "sourceDBConnectionString"="Server=sql1;Database=WideWorldImporters;User Id=adminuser;Password=<sql-db-password>;Trusted_Connection=True;"
    ```

    Las cadenas de conexión tienen subcadenas, que se muestran entre paréntesis angulares, que debe reemplazarse. Para `<storage_account_key>`, use la clave que obtuvo en el paso anterior. Para `<sql-db-password>`, use la contraseña de la cuenta de SQL Server que especificó en el archivo `onprem.parameters.json` anteriormente.

### <a name="run-the-data-warehouse-scripts"></a>Ejecución de los scripts de almacenamiento de datos

1. En [Azure Portal](https://portal.azure.com/), busque la máquina virtual local, que se llama `sql-vm1`. El nombre de usuario y la contraseña de la máquina virtual se especifican en el archivo `onprem.parameters.json`.

2. Haga clic en **Conectar** y use el escritorio remoto para conectarse a la máquina virtual.

3. En la sesión del escritorio remoto, abra un símbolo del sistema y vaya a la siguiente carpeta de la máquina virtual:

    ```
    cd C:\SampleDataFiles\reference-architectures\data\enterprise_bi_sqldw_advanced\azure\sqldw_scripts
    ```

4. Ejecute el siguiente comando:

    ```
    deploy_database.cmd -S <data_warehouse_server_name>.database.windows.net -d wwi -U adminuser -P <data_warehouse_password> -N -I
    ```

    Para `<data_warehouse_server_name>` y `<data_warehouse_password>`, utilice el nombre y la contraseña anteriores del servidor de almacenamiento de datos.

Para comprobar este paso, puede usar SQL Server Management Studio (SSMS) para conectarse a la base de datos de Azure SQL Data Warehouse. Debería ver los esquemas de la tabla de base de datos.

### <a name="run-the-data-factory-pipeline"></a>Ejecución de la canalización de Data Factory

1. En la misma sesión de escritorio remoto, abra una ventana de PowerShell.

2. Ejecute el siguiente comando de PowerShell. Elija **Sí** cuando se le solicite.

    ```powershell
    Install-Module -Name AzureRM -AllowClobber
    ```

3. Ejecute el siguiente comando de PowerShell. Escriba sus credenciales de Azure cuando se le solicite.

    ```powershell
    Connect-AzureRmAccount 
    ```

4. Ejecute los comandos de PowerShell siguientes. Reemplace los valores de los paréntesis angulares.

    ```powershell
    Set-AzureRmContext -SubscriptionId <subscription id>

    Invoke-AzureRmDataFactoryV2Pipeline -DataFactory <data-factory-name> -PipelineName "MasterPipeline" -ResourceGroupName <resource_group_name>

5. In the Azure Portal, navigate to the Data Factory instance that was created earlier.

6. In the Data Factory blade, click **Author & Monitor**. This opens the Azure Data Factory portal in another browser window.

    ![](./images/adf-blade.png)

7. In the Azure Data Factory portal, click the **Monitor** icon. 

8. Verify that the pipeline completes successfully. It can take a few minutes.

    ![](./images/adf-pipeline-progress.png)


## Build the Analysis Services model

In this step, you will create a tabular model that imports data from the data warehouse. Then you will deploy the model to Azure Analysis Services.

**Create a new tabular project**

1. From your Remote Desktop session, launch SQL Server Data Tools 2015.

2. Select **File** > **New** > **Project**.

3. In the **New Project** dialog, under **Templates**, select  **Business Intelligence** > **Analysis Services** > **Analysis Services Tabular Project**. 

4. Name the project and click **OK**.

5. In the **Tabular model designer** dialog, select **Integrated workspace**  and set **Compatibility level** to `SQL Server 2017 / Azure Analysis Services (1400)`. 

6. Click **OK**.


**Import data**

1. In the **Tabular Model Explorer** window, right-click the project and select **Import from Data Source**.

2. Select **Azure SQL Data Warehouse** and click **Connect**.

3. For **Server**, enter the fully qualified name of your Azure SQL Data Warehouse server. You can get this value from the Azure Portal. For **Database**, enter `wwi`. Click **OK**.

4. In the next dialog, choose **Database** authentication and enter your Azure SQL Data Warehouse user name and password, and click **OK**.

5. In the **Navigator** dialog, select the checkboxes for the **Fact.\*** and **Dimension.\*** tables.

    ![](./images/analysis-services-import-2.png)

6. Click **Load**. When processing is complete, click **Close**. You should now see a tabular view of the data.

**Create measures**

1. In the model designer, select the **Fact Sale** table.

2. Click a cell in the the measure grid. By default, the measure grid is displayed below the table. 

    ![](./images/tabular-model-measures.png)

3. In the formula bar, enter the following and press ENTER:

    ```
    Total Sales:=SUM('Fact Sale'[Total Including Tax])
    ```

4. Repeat these steps to create the following measures:

    ```
    Number of Years:=(MAX('Fact CityPopulation'[YearNumber])-MIN('Fact CityPopulation'[YearNumber]))+1
    
    Beginning Population:=CALCULATE(SUM('Fact CityPopulation'[Population]),FILTER('Fact CityPopulation','Fact CityPopulation'[YearNumber]=MIN('Fact CityPopulation'[YearNumber])))
    
    Ending Population:=CALCULATE(SUM('Fact CityPopulation'[Population]),FILTER('Fact CityPopulation','Fact CityPopulation'[YearNumber]=MAX('Fact CityPopulation'[YearNumber])))
    
    CAGR:=IFERROR((([Ending Population]/[Beginning Population])^(1/[Number of Years]))-1,0)
    ```

    ![](./images/analysis-services-measures.png)

For more information about creating measures in SQL Server Data Tools, see [Measures](/sql/analysis-services/tabular-models/measures-ssas-tabular).

**Create relationships**

1. In the **Tabular Model Explorer** window, right-click the project and select **Model View** > **Diagram View**.

2. Drag the **[Fact Sale].[City Key]** field to the **[Dimension City].[City Key]** field to create a relationship.  

3. Drag the **[Face CityPopulation].[City Key]** field to the **[Dimension City].[City Key]** field.  

    ![](./images/analysis-services-relations-2.png)

**Deploy the model**

1. From the **File** menu, choose **Save All**.

2. In **Solution Explorer**, right-click the project and select **Properties**. 

3. Under **Server**, enter the URL of your Azure Analysis Services instance. You can get this value from the Azure Portal. In the portal, select the Analysis Services resource, click the Overview pane, and look for the **Server Name** property. It will be similar to `asazure://westus.asazure.windows.net/contoso`. Click **OK**.

    ![](./images/analysis-services-properties.png)

4. In **Solution Explorer**, right-click the project and select **Deploy**. Sign into Azure if prompted. When processing is complete, click **Close**.

5. In the Azure portal, view the details for your Azure Analysis Services instance. Verify that your model appears in the list of models.

    ![](./images/analysis-services-models.png)

## Analyze the data in Power BI Desktop

In this step, you will use Power BI to create a report from the data in Analysis Services.

1. From your Remote Desktop session, launch Power BI Desktop.

2. In the Welcome Scren, click **Get Data**.

3. Select **Azure** > **Azure Analysis Services database**. Click **Connect**

    ![](./images/power-bi-get-data.png)

4. Enter the URL of your Analysis Services instance, then click **OK**. Sign into Azure if prompted.

5. In the **Navigator** dialog, expand the tabular project, select the model, and click **OK**.

2. In the **Visualizations** pane, select the **Table** icon. In the Report view, resize the visualization to make it larger.

6. In the **Fields** pane, expand **Dimension City**.

7. From **Dimension City**, drag **City** and **State Province** to the **Values** well.

9. In the **Fields** pane, expand **Fact Sale**.

10. From **Fact Sale**, drag **CAGR**, **Ending Population**,  and **Total Sales** to the **Value** well.

11. Under **Visual Level Filters**, select **Ending Population**. Set the filter to "is greater than 100000" and click **Apply filter**.

12. Under **Visual Level Filters**, select **Total Sales**. Set the filter to "is 0" and click **Apply filter**.

![](./images/power-bi-report-2.png)

The table now shows cities with population greater than 100,000 and zero sales. CAGR  stands for Compounded Annual Growth Rate and measures the rate of population growth per city. You could use this value to find cities with high growth rates, for example. However, note that the values for CAGR in the model aren't accurate, because they are derived from sample data.

To learn more about Power BI Desktop, see [Getting started with Power BI Desktop](/power-bi/desktop-getting-started).


[adf]: //azure/data-factory
[azure-cli-2]: //azure/install-azure-cli
[azbb-repo]: https://github.com/mspnp/template-building-blocks
[azbb-wiki]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[MergeLocation]: https://github.com/mspnp/reference-architectures/blob/master/data/enterprise_bi_sqldw_advanced/azure/sqldw_scripts/city/%5BIntegration%5D.%5BMergeLocation%5D.sql
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[ref-arch-repo-folder]: https://github.com/mspnp/reference-architectures/tree/master/data/enterprise_bi_sqldw_advanced
[wwi]: //sql/sample/world-wide-importers/wide-world-importers-oltp-database
