---
title: Inteligencia empresarial con SQL Data Warehouse
description: Use Azure para obtener perspectivas empresariales de datos relacionales almacenados localmente
author: MikeWasson
ms.date: 07/01/2018
ms.openlocfilehash: e3542e40b4b6d1f604f93bb21528f34ba7f22fc6
ms.sourcegitcommit: 58d93e7ac9a6d44d5668a187a6827d7cd4f5a34d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/02/2018
ms.locfileid: "37142342"
---
# <a name="enterprise-bi-with-sql-data-warehouse"></a>Inteligencia empresarial con SQL Data Warehouse

Esta arquitectura de referencia implementa una canalización [ELT](../../data-guide/relational-data/etl.md#extract-load-and-transform-elt) (extracción, carga y transformación) que mueve los datos desde una base de datos de SQL Server local a SQL Data Warehouse y transforma los datos para el análisis. [**Implemente esta solución**.](#deploy-the-solution)

![](./images/enterprise-bi-sqldw.png)

**Escenario**: una organización tiene un gran conjunto de datos OLTP almacenado en una base de datos de SQL Server de forma local. La organización desea utilizar SQL Data Warehouse para realizar análisis con Power BI. 

Esta arquitectura de referencia está diseñada para trabajos únicos o a petición. Si tiene que mover los datos de forma continua (cada hora o diariamente), se recomienda utilizar Azure Data Factory de Azure para definir un flujo de trabajo automatizado. Para ver una arquitectura de referencia que use Data Factory, consulte [Inteligencia empresarial automatizada con SQL Data Warehouse y Azure Data Factory](./enterprise-bi-adf.md).

## <a name="architecture"></a>Architecture

La arquitectura consta de los siguientes componentes:

### <a name="data-source"></a>Origen de datos

**SQL Server**. Los datos de origen se encuentran en una base de datos de SQL Server de forma local. Para simular el entorno local, los scripts de implementación de esta arquitectura aprovisionan una máquina virtual en Azure con SQL Server instalado. La [base de datos OLTP de ejemplo de OLTP Wide World Importers][wwi] se usa como datos de origen.

### <a name="ingestion-and-data-storage"></a>Ingesta y almacenamiento de datos

**Blob Storage**. Blob Storage se utiliza como un área de ensayo para copiar los datos antes de cargarlos en SQL Data Warehouse.

**Azure SQL Data Warehouse**. [SQL Data Warehouse](/azure/sql-data-warehouse/) es un sistema distribuido diseñado para realizar análisis con datos de gran tamaño. Admite el procesamiento paralelo masivo (MPP), lo que lo hace idóneo para ejecutar análisis de alto rendimiento. 

### <a name="analysis-and-reporting"></a>Análisis e informes

**Azure Analysis Services**. [Analysis Services](/azure/analysis-services/) es un servicio completamente administrado que proporciona funcionalidades de modelado de datos. Use Analysis Services para crear un modelo semántico que los usuarios puedan consultar. Analysis Services es especialmente útil en un escenario de panel de BI. En esta arquitectura, Analysis Services lee los datos del almacenamiento de datos para procesar el modelo semántico, y atiende a las consultas del panel. También admite simultaneidad elástica mediante el escalado horizontal de réplicas para agilizar el procesamiento de consultas.

Actualmente, Azure Analysis Services admite los modelos tabulares pero no los multidimensionales. Los modelos tabulares utilizan construcciones de modelado relacional (tablas y columnas), mientras que los modelos multidimensionales usan construcciones de modelado OLAP (cubos, dimensiones y medidas). Si necesita modelos multidimensionales, utilice SQL Server Analysis Services (SSAS). Para más información, consulte [Comparar soluciones tabulares y multidimensionales](/sql/analysis-services/comparing-tabular-and-multidimensional-solutions-ssas).

**Power BI**. Power BI es un conjunto de herramientas de análisis de negocios que sirve para analizar datos con el fin de obtener perspectivas empresariales. En el caso de esta arquitectura consulta el modelo semántico almacenado en Analysis Services.

### <a name="authentication"></a>Autenticación

**Azure Active Directory** (Azure AD) autentica a los usuarios que se conectan al servidor de Analysis Services mediante Power BI.

## <a name="data-pipeline"></a>Canalización de datos
 
Esta arquitectura de referencia usa la base de datos de ejemplo [WorldWideImporters](/sql/sample/world-wide-importers/wide-world-importers-oltp-database) como origen de datos. La canalización de datos tiene las siguientes fases:

1. Exportación de los datos de SQL Server a archivos planos (utilidad bcp).
2. Copia de los archivos planos en Azure Blob Storage (AzCopy).
3. Carga de los datos en SQL Data Warehouse (PolyBase).
4. Transformación de los datos en un esquema de estrella (T-SQL).
5. Carga de un modelo semántico en Analysis Services (SQL Server Data Tools).

![](./images/enterprise-bi-sqldw-pipeline.png)
 
> [!NOTE]
> Para los pasos del 1 al 3, considere la posibilidad de usar Data Platform Studio de Redgate. Data Platform Studio aplica las correcciones y optimizaciones de compatibilidad más adecuadas, por lo que es la forma más rápida de empezar a trabajar con SQL Data Warehouse. Para más información consulte [Carga de datos con Data Platform Studio de Redgate](/azure/sql-data-warehouse/sql-data-warehouse-load-with-redgate). 

En las secciones siguientes se describe en detalle cada una de estas etapas.

### <a name="export-data-from-sql-server"></a>Exportación de los datos de SQL Server

La utilidad [bcp](/sql/tools/bcp-utility) (programa de copia masiva) es una forma rápida de crear archivos planos de texto a partir de tablas SQL. En este paso, seleccione las columnas que desea exportar, pero no transforme los datos. Las transformaciones de datos deben realizarse en SQL Data Warehouse.

**Recomendaciones**

Si es posible, programe la extracción de datos durante las horas de poca actividad con el fin de minimizar la contención de recursos en el entorno de producción. 

Evite ejecutar bcp en el servidor de bases de datos. En su lugar, ejecútelo desde otra máquina. Escriba los archivos en una unidad local. Asegúrese de que tiene suficientes recursos de E/S para controlar las escrituras simultáneas. Para obtener un rendimiento óptimo, exporte los archivos a unidades de almacenamiento rápido dedicadas.

Puede acelerar la transferencia de red si guarda los datos exportados en un formato comprimido Gzip. De todas formas, la carga de archivos comprimidos en el almacenamiento es más lenta que la carga de archivos sin comprimir, por lo que la rapidez en la transferencia de red tiene como contrapartida una menor velocidad en la carga. Si decide utilizar compresión Gzip, no cree un único archivo Gzip. Divida los datos en varios archivos comprimidos.

### <a name="copy-flat-files-into-blob-storage"></a>Copia de los archivos planos en Blob Storage

La utilidad [AzCopy](/azure/storage/common/storage-use-azcopy) está diseñada para la copia de alto rendimiento de datos en Azure Blob Storage.

**Recomendaciones**

Cree la cuenta de almacenamiento en una región cerca de la ubicación del origen de datos. Implemente la cuenta de almacenamiento y la instancia de SQL Data Warehouse en la misma región. 

No ejecute AzCopy en la misma máquina que ejecuta las cargas de trabajo de producción, ya que el consumo de CPU y E/S puede interferir con la carga de trabajo de producción. 

Pruebe la carga primero para ver cómo es la velocidad de carga. Puede utilizar la opción/NC en AzCopy para especificar el número de operaciones de copia simultáneas. Comience con el valor predeterminado, a continuación, experimente con este valor para optimizar el rendimiento. En un entorno con poco ancho de banda, demasiadas operaciones simultáneas pueden saturar la conexión de red e impedir que las operaciones se completen con éxito.  

AzCopy lleva los datos al almacenamiento a través de la red pública de Internet. Si esto no es lo suficientemente rápido, considere la posibilidad de configurar un circuito [ExpressRoute](/azure/expressroute/). ExpressRoute es un servicio que enruta los datos a Azure a través de una conexión privada dedicada. Otra opción, si la conexión de red es demasiado lenta, es enviar físicamente los datos en un disco a un centro de datos de Azure. Para más información consulte [Transferencia de datos a Azure y desde este](/azure/architecture/data-guide/scenarios/data-transfer).

Durante una operación de copia, AzCopy crea un archivo de diario temporal, lo que permite a AzCopy reiniciar la operación si esta se interrumpiese (por ejemplo, debido a un error de red). Asegúrese de que hay suficiente espacio en disco para almacenar los archivos de diario. Puede utilizar la opción /Z para especificar dónde se escriben los archivos de diario.

### <a name="load-data-into-sql-data-warehouse"></a>Carga de los datos en SQL Data Warehouse

Use [PolyBase](/sql/relational-databases/polybase/polybase-guide) para cargar los archivos de Blob Storage al almacenamiento de datos. PolyBase está diseñado para aprovechar la arquitectura MPP (procesamiento masivo en paralelo) de SQL Data Warehouse, lo que lo convierte en la forma más rápida de cargar datos en SQL Data Warehouse. 

La carga de datos es un proceso de dos pasos:

1. Creación de un conjunto de tablas externas para los datos. Una tabla externa es una definición de tabla que apunta a los datos almacenados fuera del almacén &mdash; en este caso, a los archivos planos en Blob Storage. Este paso no mueve ningún dato al almacenamiento.
2. Creación de tablas de almacenamiento provisional y carga de los datos en las tablas de almacenamiento provisional. Este paso copia los datos en el almacenamiento.

**Recomendaciones**

Considere la posibilidad de usar SQL Data Warehouse cuando tenga grandes cantidades de datos (más de 1 TB) y esté ejecutando una carga de trabajo de análisis que aprovecharía el paralelismo. SQL Data Warehouse no es una buena opción para cargas de trabajo OLTP o conjuntos de datos más pequeños (< 250 GB). Para conjuntos de datos de un tamaño inferior a 250 GB, considere la posibilidad de usar Azure SQL Database o SQL Server. Para más información, consulte el artículo sobre [Almacenamiento de datos](../../data-guide/relational-data/data-warehousing.md).

Cree las tablas de almacenamiento provisional como tablas de montón, que no están indexadas. Las consultas que crean las tablas de producción darán como resultado un examen completo de la tabla, así que no hay ninguna razón para indexar las tablas de almacenamiento provisional.

PolyBase aprovecha automáticamente las ventajas del paralelismo en el almacenamiento. El rendimiento de carga se escala a medida que aumentan las unidades de almacenamiento de datos. Para obtener el mejor rendimiento, utilice una operación de carga única. Dividir los datos de entrada en fragmentos y ejecutar varias cargas simultáneas no beneficia al rendimiento.

PolyBase puede leer archivos comprimidos con Gzip. De todas formas, se usa un lector único por archivo comprimido, porque descomprimir el archivo es una operación de un solo subproceso. Por ello, evite cargar un único archivo comprimido grande. En su lugar, divida los datos en varios archivos comprimidos, para aprovechar las ventajas del paralelismo. 

Tenga en cuenta las siguientes limitaciones:

- PolyBase admite un tamaño máximo de columna de `varchar(8000)`, `nvarchar(4000)` o `varbinary(8000)`. Si tiene datos que superan estos límites, tiene la opción de dividir los datos en fragmentos al exportarlos y luego volver a ensamblar los fragmentos tras la importación. 

- PolyBase usa un terminador de fila fijo de \n o una línea nueva. Esto puede causar problemas si los caracteres de nueva línea aparecen en los datos de origen.

- El esquema de origen de datos podría contener tipos de datos que SQL Data Warehouse no admite.

Para evitar estas limitaciones, puede crear un procedimiento almacenado que realiza las conversiones necesarias. Haga referencia a este procedimiento almacenado cuando ejecute bcp. De forma alternativa, [Data Platform Studio de Redgate](/azure/sql-data-warehouse/sql-data-warehouse-load-with-redgate) convierte automáticamente los tipos de datos que SQL Data Warehouse no admite.

Para más información, consulte los siguientes artículos.

- [Procedimientos recomendados para la carga de datos en Azure SQL Data Warehouse](/azure/sql-data-warehouse/guidance-for-loading-data).
- [Migración de esquemas a SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-migrate-schema)
- [Guía para definir los tipos de datos para las tablas en SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-tables-data-types)

### <a name="transform-the-data"></a>Transformar los datos

Transforme los datos y muévalos a tablas de producción. En este paso, los datos se transforman en un esquema en estrella con tablas de dimensiones y tablas de hechos, adecuadas para el modelado semántico.

Cree las tablas de producción con índices de almacén de columnas en clúster que ofrecen el mejor rendimiento general para las consultas. Los índices de almacén de columnas están optimizados para consultas que examinan muchos registros. Sin embargo no funcionan tan bien para las búsquedas singleton (es decir aquellas en las que la búsqueda se reduce a una sola fila). Si tiene que realizar búsquedas singleton frecuentes, puede agregar un índice no agrupado en una tabla. Este tipo de búsquedas pueden ejecutarse de forma considerablemente más rápida usando un índice no agrupado. De todas formas, las búsquedas singleton son normalmente menos comunes en escenarios de almacenamiento de datos que las cargas de trabajo OLTP. Para más información, consulte [Indexación de tablas en SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-tables-index).

> [!NOTE]
> Las tablas de almacén de columnas agrupado no admiten tipos de datos `varchar(max)`, `nvarchar(max)` o `varbinary(max)`. En ese caso, considere la posibilidad de un índice agrupado o de montón. Puede colocar esas columnas en una tabla independiente.

Dado que la base de datos de ejemplo no es muy grande, hemos creado las tablas replicadas sin particiones. Para las cargas de trabajo de producción, el uso de tablas distribuidas probablemente mejore el rendimiento de las consultas. Consulte [Instrucciones para diseñar tablas distribuidas en Azure SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-tables-distribute). Los scripts de ejemplo ejecutan las consultas utilizando una [clase de recursos](/azure/sql-data-warehouse/resource-classes-for-workload-management) estática.

### <a name="load-the-semantic-model"></a>Carga del modelo semántico

Cargue los datos en un modelo tabular en Azure Analysis Services. En este paso, creará un modelo de datos semánticos usando SQL Server Data Tools (SSDT). También puede crear un modelo mediante la importación desde un archivo de Power BI Desktop. Teniendo en cuenta que SQL Data Warehouse no admite las claves externas, tiene que agregar las relaciones al modelo semántico para que se puedan realizar uniones entre las tablas.

### <a name="use-power-bi-to-visualize-the-data"></a>Uso de Power BI para visualizar los datos

Power BI admite dos opciones para la conexión a Azure Analysis Services:

- Importación. Se importan los datos en el modelo BI.
- Conexión dinámica. Se extraen los datos directamente desde Analysis Services.

Se recomienda la conexión dinámica, ya que no requiere copiar los datos en el modelo de Power BI. Además, al usar DirectQuery se garantiza que los resultados siempre sean coherentes con los últimos datos de origen. Para más información, consulte [Conexión con Power BI](/azure/analysis-services/analysis-services-connect-pbi).

**Recomendaciones**

Evite ejecutar consultas de panel de BI directamente en el almacenamiento de datos. Los paneles BI requieren tiempos de respuesta muy bajos, es posible que las consultas directas realizadas en el almacenamiento no pueden satisfacer esta necesidad. Además, la actualización del panel se reflejará en el número de consultas simultáneas, lo que podría afectar al rendimiento. 

Azure Analysis Services está diseñado para controlar los requisitos de consulta de un panel de BI, por lo que es el método recomendado es consultar Analysis Services desde Power BI.

## <a name="scalability-considerations"></a>Consideraciones sobre escalabilidad

### <a name="sql-data-warehouse"></a>SQL Data Warehouse

Con SQL Data Warehouse, puede escalar horizontalmente los recursos de proceso a petición. El motor de consultas optimiza las consultas para el procesamiento paralelo en función del número de nodos de ejecución y mueve los datos entre los nodos según sea necesario. Para más información consulte [Administración de proceso en Azure SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-manage-compute-overview).

### <a name="analysis-services"></a>Analysis Services

Para las cargas de trabajo de producción, se recomienda el nivel estándar para Azure Analysis Services, ya que admite la creación de particiones y DirectQuery. Dentro de un nivel, el tamaño de la instancia determina la memoria y potencia de procesamiento. La capacidad de procesamiento se mide en unidades de procesamiento de consultas (QPU). Supervise el uso de las QPU para seleccionar el tamaño adecuado. Para más información, consulte [Supervisión de las métricas del servidor](/azure/analysis-services/analysis-services-monitor).

Bajo una carga alta, el rendimiento de las consultas se puede ver degradado debido a la simultaneidad de las mismas. Puede escalar horizontalmente Analysis Services mediante la creación de un grupo de réplicas para procesar las consultas, para que se pueden realizar más consultas simultáneamente. El trabajo de procesar el modelo de datos siempre se produce en el servidor principal. De forma predeterminada, el servidor principal también controla las consultas. Si lo desea, puede designar al servidor principal para que ejecute el procesamiento de modo exclusivo, de modo que el grupo de consulta controle todas las consultas. Si los requisitos de procesamiento son altos, debe separar el procesamiento del grupo de consulta. Si tiene cargas de consulta elevadas, y un procesamiento relativamente ligero, puede incluir el servidor principal en el grupo de consulta. Para más información consulte [Escalabilidad horizontal de Azure Analysis Services](/azure/analysis-services/analysis-services-scale-out). 

Para reducir la cantidad de procesamiento innecesario, considere la posibilidad de utilizar particiones para dividir el modelo tabular en partes lógicas. Cada partición se puede procesar por separado. Para más información consulte [Particiones](/sql/analysis-services/tabular-models/partitions-ssas-tabular).

## <a name="security-considerations"></a>Consideraciones sobre la seguridad

### <a name="ip-whitelisting-of-analysis-services-clients"></a>Creación de listas de direcciones IP permitidas de clientes de Analysis Services

Considere el uso de la característica de firewall de Analysis Services para crear una lista de direcciones IP permitidas de clientes. Cuando está habilitada, la característica de firewall bloquea todas las conexiones de cliente a excepción de las especificadas en las reglas de firewall. El servicio Power BI se encuentra en la lista de permitidos de forma predeterminada, pero se puede deshabilitar esta regla si lo desea. Para más información, consulte [Hardening Azure Analysis Services with the new firewall capability](https://azure.microsoft.com/blog/hardening-azure-analysis-services-with-the-new-firewall-capability/) (Protección de Azure Analysis Services con la nueva funcionalidad de firewall).

### <a name="authorization"></a>Autorización

Azure Analysis Services usa Azure Active Directory (Azure AD) para autenticar a los usuarios que se conectan a un servidor de Analysis Services. Puede restringir los datos que un usuario determinado puede ver mediante la creación de roles, y después asignar grupos o usuarios de Azure AD a esos roles. Para cada rol puede hacer lo siguiente: 

- Proteger las tablas o columnas individuales. 
- Proteger filas individuales en base a expresiones de filtro. 

Para más información, consulte [Administración de usuarios y roles de base de datos](/azure/analysis-services/analysis-services-database-users).

## <a name="deploy-the-solution"></a>Implementación de la solución

Hay disponible una implementación de esta arquitectura de referencia en [GitHub][ref-arch-repo-folder]. Implementa lo siguiente:

  * Una máquina virtual Windows para simular un servidor de bases de datos local. Incluye SQL Server 2017 y herramientas relacionadas, junto con Power BI Desktop.
  * Una cuenta de almacenamiento de Azure que proporciona almacenamiento de blobs para almacenar los datos exportados de la base de datos de SQL Server.
  * Una instancia de Azure SQL Data Warehouse.
  * Una instancia de Azure Analysis Services.

### <a name="prerequisites"></a>requisitos previos

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-the-simulated-on-premises-server"></a>Implementación del servidor local simulado

En primer lugar va a implementar una máquina virtual como un servidor local simulado, que incluye SQL Server 2017 y las herramientas relacionadas. Este paso también carga la [base de datos OLTP Wide World Importers][wwi] en SQL Server.

1. Vaya a la carpeta `data\enterprise_bi_sqldw\onprem\templates` del repositorio.

2. En el archivo `onprem.parameters.json` reemplace los valores de `adminUsername` y `adminPassword`. Cambie también los valores en la sección `SqlUserCredentials` para que coincidan con el nombre de usuario y la contraseña. Tenga en cuenta el prefijo `.\\` en la propiedad de nombre de usuario.
    
    ```bash
    "SqlUserCredentials": {
      "userName": ".\\username",
      "password": "password"
    }
    ```

3. Ejecute `azbb` tal y como se muestra a continuación para implementar el servidor local.

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <region> -p onprem.parameters.json --deploy
    ```

    Especifique una región que admita SQL Data Warehouse y Azure Analysis Services. Consulte [Productos de Azure por región](https://azure.microsoft.com/global-infrastructure/services/)

4. La implementación puede tardar de 20 a 30 minutos en completarse, lo que incluye ejecutar el script [DSC](/powershell/dsc/overview) para instalar las herramientas y restaurar la base de datos. Compruebe la implementación en Azure Portal revisando los recursos del grupo de recursos. Debería ver la máquina virtual `sql-vm1` y sus recursos asociados.

### <a name="deploy-the-azure-resources"></a>Implementación de los recursos de Azure

Este paso aprovisiona SQL Data Warehouse y Azure Analysis Services, junto con una cuenta de Storage. Si lo desea, puede ejecutar este paso en paralelo con el paso anterior.

1. Vaya a la carpeta `data\enterprise_bi_sqldw\azure\templates` del repositorio.

2. Ejecute el siguiente comando de la CLI de Azure para crear un grupo de recursos. Puede realizar la implementación en un grupo de recursos diferente que el paso anterior, pero elegir la misma región. 

    ```bash
    az group create --name <resource_group_name> --location <region>  
    ```

3. Ejecute el siguiente comando de la CLI de Azure para implementar los recursos de Azure. Reemplace los valores del parámetro que se muestran entre paréntesis angulares. 

    ```bash
    az group deployment create --resource-group <resource_group_name> \
     --template-file azure-resources-deploy.json \
     --parameters "dwServerName"="<server_name>" \
     "dwAdminLogin"="<admin_username>" "dwAdminPassword"="<password>" \ 
     "storageAccountName"="<storage_account_name>" \
     "analysisServerName"="<analysis_server_name>" \
     "analysisServerAdmin"="user@contoso.com"
    ```

    - El parámetro `storageAccountName` tiene que seguir las [reglas de nomenclatura](../../best-practices/naming-conventions.md#naming-rules-and-restrictions) para las cuentas de almacenamiento.
    - Para el parámetro `analysisServerAdmin`, use el nombre principal de usuario (UPN) de Azure Active Directory.

4. Compruebe la implementación en Azure Portal revisando los recursos del grupo de recursos. Debería ver una cuenta de almacenamiento, la instancia de Azure SQL Data Warehouse y la instancia de Analysis Services.

5. Use Azure Portal para obtener la clave de acceso para la cuenta de almacenamiento. Seleccione la cuenta de almacenamiento para abrirla. En **Configuración**, seleccione **Claves de acceso**. Anote el valor de la clave principal. Lo usará en el paso siguiente.

### <a name="export-the-source-data-to-azure-blob-storage"></a>Exportación de los datos de origen a Azure Blob Storage 

En este paso, ejecutará un script de PowerShell que usa bcp para exportar la base de datos SQL a archivos planos en la máquina virtual, y luego usa AzCopy para copiar estos archivos en Azure Blob Storage.

1. Use el Escritorio remoto para conectarse a la máquina virtual simulada local.

2. Mientas tiene iniciada sesión en la máquina virtual, ejecute los comandos siguientes en la ventana de PowerShell.  

    ```powershell
    cd 'C:\SampleDataFiles\reference-architectures\data\enterprise_bi_sqldw\onprem'

    .\Load_SourceData_To_Blob.ps1 -File .\sql_scripts\db_objects.txt -Destination 'https://<storage_account_name>.blob.core.windows.net/wwi' -StorageAccountKey '<storage_account_key>'
    ```

    Para el parámetro `Destination`, reemplace `<storage_account_name>` con el nombre de la cuenta de almacenamiento que creó anteriormente. Para el parámetro `StorageAccountKey`, use la tecla de acceso para esa cuenta de almacenamiento.

3. En Azure Portal, compruebe que los datos de origen se copiaron al almacenamiento de blobs yendo a la cuenta de almacenamiento, seleccionando el servicio Blob y abriendo el contenedor `wwi`. Debería ver una lista de tablas que se prologa con `WorldWideImporters_Application_*`.

### <a name="run-the-data-warehouse-scripts"></a>Ejecución de los scripts de almacenamiento de datos

1. En su sesión de escritorio remoto, inicie SQL Server Management Studio (SSMS). 

2. Conexión a SQL Data Warehouse

    - Tipo de servidor: motor de base de datos
    
    - Nombre del servidor: `<dwServerName>.database.windows.net`, donde `<dwServerName>` es el nombre que especificó al implementar los recursos de Azure. Encontrará este nombre en Azure Portal.
    
    - Autenticación: autenticación de SQL Server. Utilice las credenciales que especificó al implementar los recursos de Azure, en los parámetros `dwAdminLogin` y `dwAdminPassword`.

2. Vaya a la carpeta `C:\SampleDataFiles\reference-architectures\data\enterprise_bi_sqldw\azure\sqldw_scripts` en la máquina virtual. Ejecutará los scripts de esta carpeta en orden numérico, de `STEP_1` hasta `STEP_7`.

3. Seleccione la base de datos `master` en SSMS y abra el script `STEP_1`. Cambie el valor de la contraseña en la siguiente línea y ejecute el script.

    ```sql
    CREATE LOGIN LoaderRC20 WITH PASSWORD = '<change this value>';
    ```

4. Seleccione la base de datos `wwi` en SSMS. Abra el script `STEP_2` y ejecútelo. Si recibe un error, asegúrese de que está ejecutando el script con la base de datos `wwi` y no `master`.

5. Abra una nueva conexión a SQL Data Warehouse, usando el usuario y la contraseña de `LoaderRC20` que se indican en el script `STEP_1`.

6. Usando esta conexión abra el script `STEP_3`. Establezca los valores siguientes en el script:

    - SECRET: use la clave de acceso para la cuenta de almacenamiento.
    - LOCATION: use el nombre de la cuenta de almacenamiento como sigue: `wasbs://wwi@<storage_account_name>.blob.core.windows.net`.

7. Con la misma conexión, ejecute los script del `STEP_4` hasta el `STEP_7` secuencialmente. Compruebe que cada script se completa correctamente antes de ejecutar el siguiente.

En SMSS, debería ver un conjunto de tablas `prd.*` en la base de datos `wwi`. Para comprobar que se generaron los datos, ejecute la siguiente consulta: 

```sql
SELECT TOP 10 * FROM prd.CityDimensions
```

## <a name="build-the-analysis-services-model"></a>Creación del modelo de Analysis Services

En este paso, creará un modelo tabular que importa los datos del almacenamiento de datos. A continuación, implementará el modelo en Azure Analysis Services.

1. En su sesión de escritorio remoto, inicie SQL Server Data Tools 2015.

2. Seleccione **Archivo** > **Nuevo** > **Proyecto**.

3. En el cuadro de diálogo **Nuevo proyecto** dentro de **Plantillas** seleccione **Business Intelligence** > **Analysis Services** > **Proyecto tabular de Analysis Services**. 

4. Asigne un nombre al proyecto y haga clic en **Aceptar**.

5. En el cuadro de diálogo **Diseñador de modelos tabulares**, seleccione **Área de trabajo integrada** y establezca el **Nivel de compatibilidad** en `SQL Server 2017 / Azure Analysis Services (1400)`. Haga clic en **OK**.

6. En la ventana del **Explorador de modelos tabulares**, haga clic con el botón derecho en el proyecto y seleccione **Importar desde el origen de datos**.

7. Seleccione **Azure SQL Data Warehouse** y haga clic en **Conectar**.

8. Para **Servidor**, escriba el nombre completo del servidor de Azure SQL Data Warehouse. Para **Base de datos**, escriba `wwi`. Haga clic en **OK**.

9. En el siguiente cuadro de diálogo, elija autenticación de **Base de datos** y escriba el nombre de usuario y la contraseña de Azure SQL Data Warehouse y haga clic en **Aceptar**.

10. En el cuadro de diálogo **Navegador**, seleccione las casillas de verificación para **prd.CityDimensions**, **prd.DateDimensions**, y **prd.SalesFact**. 

    ![](./images/analysis-services-import.png)

11. Haga clic en **Cargar**. Una vez completado el procesamiento, haga clic en **Cerrar**. Debería ver una vista tabular de los datos.

12. En la ventana del **Explorador de modelos tabulares**, haga clic con el botón derecho en el proyecto y seleccione **Vista de modelo** > **Vista de diagrama**.

13. Arrastre el campo **[prd.SalesFact].[WWI City ID]** hasta el campo **[prd.CityDimensions].[WWI City ID]** para crear una relación.  

14. Arrastre el campo **[prd.SalesFact].[Invoice Date Key]** hasta el campo **[prd.DateDimensions].[Date]**.  
    ![](./images/analysis-services-relations.png)

15. En el menú **Archivo**, haga clic en **Guardar todo**.  

16. En el **Explorador de soluciones**, haga clic con el botón derecho en el proyecto y seleccione **Propiedades**. 

17. En **Servidor**, escriba la dirección URL de la instancia de Azure Analysis Services. Puede encontrar este valor en Azure Portal. En el portal, seleccione el recurso Analysis Services, haga clic en el panel de información general y busque la propiedad **Nombre del servidor**. Será algo similar a `asazure://westus.asazure.windows.net/contoso`. Haga clic en **OK**.

    ![](./images/analysis-services-properties.png)

18. En el **Explorador de soluciones**, haga clic con el botón derecho en el proyecto y seleccione **Implementar**. Inicie sesión en Azure si se le solicita. Una vez completado el procesamiento, haga clic en **Cerrar**.

19. En Azure Portal, vea los detalles de la instancia de Azure Analysis Services. Compruebe que el modelo aparece en la lista de modelos.

    ![](./images/analysis-services-models.png)

## <a name="analyze-the-data-in-power-bi-desktop"></a>Análisis de los datos de Power BI Desktop

En este paso, usará Power BI para crear un informe de los datos en Analysis Services.

1. En su sesión de escritorio remoto, inicie Power BI Desktop.

2. En la pantalla de bienvenida, haga clic en **Get Data** (Obtener datos).

3. Seleccione **Azure** > **Base de datos de Azure Analysis Services**. Haga clic en **Conectar**

    ![](./images/power-bi-get-data.png)

4. Escriba la dirección URL de la instancia de Analysis Services y haga clic en **Aceptar**. Inicie sesión en Azure si se le solicita.

5. En el cuadro de diálogo **Navegador**, expanda el proyecto tabular que ha implementado, seleccione el modelo que ha creado y haga clic en **Aceptar**.

2. En el panel **Visualizaciones**, seleccione el icono **Gráfico de barras apiladas**. En la vista de informe, cambie el tamaño de la visualización para que sea mayor.

6. En el panel **Campos**, expanda **prd.CityDimensions**.

7. Arrastre **prd.CityDimensions** > **WWI City ID** al espacio **Eje**.

8. Arrastre **prd.CityDimensions** > **City** al espacio **Leyenda**.

9. En el panel **Campos** expanda **prd.SalesFact**.

10. Arrastre **prd.SalesFact** > **Total Excluding Tax** al espacio **Valor**.

    ![](./images/power-bi-visualization.png)

11. En **Filtros de nivel de objeto visual**, seleccione **WWI City ID**.

12. Establezca el **Tipo de filtro** en `Top N` y **Mostrar artículos** en `Top 10`.

13. Arrastre **prd.SalesFact** > **Total Excluding Tax** al espacio **Por valor**

    ![](./images/power-bi-visualization2.png)

14. Haga clic en **Aplicar filtro**. La visualización muestra las 10 ventas totales principales por ciudad.

    ![](./images/power-bi-report.png)

Para aprender más acerca de Power BI Desktop consulte [Introducción a Power BI Desktop](/power-bi/desktop-getting-started).

## <a name="next-steps"></a>Pasos siguientes

- Para más información acerca de esta arquitectura de referencia visite el [repositorio de GitHub][ref-arch-repo-folder].
- Más información acerca de [Azure Building Blocks][azbb-repo].

<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azbb-repo]: https://github.com/mspnp/template-building-blocks
[azbb-wiki]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/data/enterprise_bi_sqldw
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[ref-arch-repo-folder]: https://github.com/mspnp/reference-architectures/tree/master/data/enterprise_bi_sqldw
[wwi]: /sql/sample/world-wide-importers/wide-world-importers-oltp-database
