---
title: Antipatrón Monolithic Persistence
description: Colocar todos los datos de una aplicación en un único almacén de datos puede perjudicar el rendimiento.
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: 7f04b9f0805c281068b6b2edaf040683773e6f6e
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2017
ms.locfileid: "24538687"
---
# <a name="monolithic-persistence-antipattern"></a>Antipatrón Monolithic Persistence

Colocar todos los datos de una aplicación en un único almacén de datos puede perjudicar al rendimiento, ya sea porque da lugar a la contención de recursos o porque el almacén de datos no sea una buena opción para algunos de los datos.

## <a name="problem-description"></a>Descripción del problema

Históricamente, las aplicaciones a menudo usaban un único almacén de datos, independientemente de los distintos tipos de datos que la aplicación pudiera tener que almacenar. Normalmente, esto se hace para simplificar el diseño de la aplicación, o bien para que se corresponda con las capacidades actuales del equipo de desarrollo. 

Los sistemas modernos basados en la nube a menudo tienen requisitos no funcionales y funcionales adicionales, y necesitan almacenar muchos tipos de datos heterogéneos, como documentos, imágenes, datos almacenados en caché, mensajes en cola, registros de aplicación y telemetría. Si se sigue el enfoque tradicional y se coloca toda esta información en el mismo almacén de datos, puede perjudicar el rendimiento, por dos motivos principales:

- Almacenar y recuperar grandes cantidades de datos no relacionados en el mismo almacén de datos puede ocasionar una contención, lo que a su vez conduce a tiempos de respuesta mayores y a errores de conexión.
- Con independencia del almacén de datos elegido, podría no ser la mejor opción para todos los distintos tipos de datos o quizá no se puedan optimizar las operaciones que la aplicación realiza. 

En el ejemplo siguiente se muestra un controlador de ASP.NET Web API que agrega un nuevo registro a una base de datos y también registra el resultado en un registro. El registro se mantiene en la misma base de datos que los datos empresariales. Puede encontrar el ejemplo completo [aquí][sample-app].

```csharp
public class MonoController : ApiController
{
    private static readonly string ProductionDb = ...;

    public async Task<IHttpActionResult> PostAsync([FromBody]string value)
    {
        await DataAccess.InsertPurchaseOrderHeaderAsync(ProductionDb);
        await DataAccess.LogAsync(ProductionDb, LogTableName);
        return Ok();
    }
}
```

La velocidad a la que se generan registros probablemente afectará al rendimiento de las operaciones empresariales. Y, si otro componente, como un monitor de proceso de aplicación, lee y procesa los datos del registro con regularidad, también puede afectar a las operaciones empresariales.

## <a name="how-to-fix-the-problem"></a>Procedimiento para corregir el problema

Separe los datos según su uso. Para cada conjunto de datos, seleccione el almacén de datos que mejor coincida con el que se va a usar. En el ejemplo anterior, la aplicación debe registrarse en un almacén independiente de la base de datos que contenga los datos empresariales: 

```csharp
public class PolyController : ApiController
{
    private static readonly string ProductionDb = ...;
    private static readonly string LogDb = ...;

    public async Task<IHttpActionResult> PostAsync([FromBody]string value)
    {
        await DataAccess.InsertPurchaseOrderHeaderAsync(ProductionDb);
        // Log to a different data store.
        await DataAccess.LogAsync(LogDb, LogTableName);
        return Ok();
    }
}
```

## <a name="considerations"></a>Consideraciones

- Separe los datos por la forma en que se usan y cómo se accede a ellos. Por ejemplo, no almacene información de registro y datos empresariales en el mismo almacén de datos. Estos tipos de datos tienen requisitos y patrones de acceso significativamente diferentes. Las entradas del registro son intrínsecamente secuenciales, mientras que es más probable que los datos empresariales requieran acceso aleatorio y, a menudo, relacional.

- Considere el patrón de acceso a los datos para cada tipo de datos. Por ejemplo, almacene los documentos e informes con formato en una base de datos de documentos como [Cosmos DB][CosmosDB], pero use [Azure Redis Cache][ Azure-cache] para almacenar los datos en caché temporalmente.

- Si sigue esta guía pero todavía alcanza los límites de la base de datos, puede que deba ampliar la base de datos. También considere la posibilidad de escalar horizontalmente y dividir la carga entre servidores de base de datos. Sin embargo, esto puede requerir volver a diseñar la aplicación. Para más información, consulte [Partitioning guidance][DataPartitioningGuidance] (Creación de particiones de datos).

## <a name="how-to-detect-the-problem"></a>Procedimiento para detectar el problema

El sistema probablemente se ralentizará de forma considerable y terminará produciendo un error, ya que se queda sin recursos como las conexiones de base de datos.

Puede realizar los pasos siguientes para ayudar a identificar la causa:

1. Instrumente el sistema para registrar las estadísticas esenciales de rendimiento. Capture información de temporización para cada operación, así como los lugares en los que la aplicación lee y escribe datos.
1. Si es posible, supervise el sistema en ejecución durante unos días, en un entorno de producción, para obtener una visión real de cómo se utiliza. Si esto no es posible, ejecute pruebas de carga generadas por script con un volumen realista de usuarios virtuales que realizan una serie de operaciones habituales.
2. Utilice los datos de telemetría para identificar los períodos con un rendimiento bajo.
3. Identifique a qué almacenes de datos se accedió durante esos períodos.
4. Identifique los recursos de almacenamiento de datos que podrían estar experimentando alguna contención.

## <a name="example-diagnosis"></a>Diagnóstico de ejemplo

En las secciones siguientes se aplican estos pasos para la aplicación de ejemplo descrita anteriormente.

### <a name="instrument-and-monitor-the-system"></a>Instrumentación y supervisión del sistema

El siguiente gráfico muestra los resultados de la aplicación de ejemplo de la prueba de carga descrita antes. La prueba usaba una carga por pasos de hasta 1 000 usuarios simultáneos.

![Resultados de rendimiento de la prueba de carga para el controlador basado en SQL][MonolithicScenarioLoadTest]

A medida que aumenta la carga a 700 usuarios, aumenta el rendimiento. En ese momento, el rendimiento se estabiliza y parece que el sistema se esté ejecutando a su capacidad máxima. El promedio de respuesta aumenta gradualmente con la carga de usuarios, lo que evidencia que el sistema no puede hacer frente a la demanda.

### <a name="identify-periods-of-poor-performance"></a>Identificación de períodos con un rendimiento bajo

Si va a supervisar el sistema de producción, podría observar patrones. Por ejemplo, los tiempos de respuesta podrían caer significativamente a la misma hora cada día. Podría deberse a una carga de trabajo normal o a un trabajo por lotes programado, o simplemente a que el sistema tenga más usuarios en determinados momentos. Debe centrarse en los datos de telemetría en estos casos.

Busque correlaciones entre mayores tiempos de respuesta y la mayor actividad de las bases de datos, o en la E/S en los recursos compartidos. Si hay correlaciones, significa que la base de datos podría ser un cuello de botella.

### <a name="identify-which-data-stores-are-accessed-during-those-periods"></a>Identifique a qué almacenes de datos se accedió durante esos períodos.

El gráfico siguiente muestra la utilización de las unidades de rendimiento de la base de datos (DTU) durante la prueba de carga. (Una DTU es una medida de la capacidad disponible y combina el uso de CPU, la asignación de memoria y la velocidad de E/S). El uso de DTU alcanzó rápidamente el 100 %. Es, aproximadamente, el punto en el que el rendimiento alcanzó su máximo en el gráfico anterior. El uso de las bases de datos permaneció muy alto hasta que la prueba finalizó. Hay una ligera reducción hacia el final, que podría estar provocada por la limitación, la competencia por las conexiones de base de datos u otros factores.

![El monitor de base de datos en el Portal de Azure clásico que muestra el uso de recursos de la base de datos][MonolithicDatabaseUtilization]

### <a name="examine-the-telemetry-for-the-data-stores"></a>Examine la telemetría para los almacenes de datos

Instrumente los almacenes de datos para capturar los detalles de bajo nivel de la actividad. En la aplicación de ejemplo, la estadísticas de acceso a los datos mostraban un alto volumen de operaciones de inserción realizadas en ambas tablas: `PurchaseOrderHeader` y `MonoLog`. 

![Estadísticas de acceso a los datos para la aplicación de ejemplo][MonolithicDataAccessStats]

### <a name="identify-resource-contention"></a>Identificación de la contención de recursos

En este momento, puede revisar el código fuente, centrándose en los puntos donde la aplicación accede a los recursos que sufren la contención. Busque situaciones como:

- Datos que están separados lógicamente y se escriban en el mismo almacén. Datos como los registros, los informes y los mensajes en cola no deben mantenerse en la misma base de datos que la información empresarial.
- Una discrepancia entre la elección del almacén de datos y el tipo de datos, como blobs grandes o documentos XML en una base de datos relacional.
- Datos con patrones de uso significativamente diferentes que comparten el mismo almacén, como los datos con muchas escrituras y pocas lecturas que se almacenan con datos con pocas escrituras y muchas lecturas.

### <a name="implement-the-solution-and-verify-the-result"></a>Implementación de la solución y comprobación del resultado

La aplicación se cambió para escribir los registros en un almacén de datos independiente. Estos son los resultados de la prueba de carga:

![Resultados de rendimiento de la prueba de carga con el controlador Polyglot][PolyglotScenarioLoadTest]

El patrón de rendimiento es similar al del gráfico anterior, pero el punto en el que alcanza el rendimiento máximo es aproximadamente unas 500 solicitudes por segundo mayor. El tiempo de respuesta promedio es ligeramente inferior. Sin embargo, estas estadísticas no muestran todo el panorama. La telemetría para la base de datos empresarial muestra que el uso de DTU alcanza el máximo aproximadamente en un 75 %, en lugar del 100 %.

![Monitor de base de datos en el Portal de Azure clásico que muestra el uso de recursos de la base de datos en el escenario con Polyglot][PolyglotDatabaseUtilization]

De forma similar, la utilización de DTU máxima de la base de datos de registro solo alcanza aproximadamente el 70 %. Las bases de datos ya no son el factor que limita el rendimiento del sistema.

![Monitor de base de datos en el Portal de Azure clásico que muestra el uso de recursos de la base de datos de registro en el escenario con Polyglot][LogDatabaseUtilization]


## <a name="related-resources"></a>Recursos relacionados

- [Elección del almacén de datos apropiado][data-store-overview]
- [Criterios para elegir un almacén de datos][data-store-comparison]
- [Data Access for Highly-Scalable Solutions: Using SQL, NoSQL, and Polyglot Persistence][Data-Access-Guide] (Acceso a datos para soluciones muy escalables: uso de la persistencia de Polyglot, SQL y NoSQL)
- [Creación de particiones de datos][DataPartitioningGuidance]

[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/MonolithicPersistence
[CosmosDB]: http://azure.microsoft.com/services/cosmos-db/
[Azure-cache]: /azure/redis-cache/
[Data-Access-Guide]: https://msdn.microsoft.com/library/dn271399.aspx
[DataPartitioningGuidance]: ../../best-practices/data-partitioning.md
[data-store-overview]: ../../guide/technology-choices/data-store-overview.md
[data-store-comparison]: ../../guide/technology-choices/data-store-comparison.md

[MonolithicScenarioLoadTest]: _images/MonolithicScenarioLoadTest.jpg
[MonolithicDatabaseUtilization]: _images/MonolithicDatabaseUtilization.jpg
[MonolithicDataAccessStats]: _images/MonolithicDataAccessStats.jpg
[PolyglotScenarioLoadTest]: _images/PolyglotScenarioLoadTest.jpg
[PolyglotDatabaseUtilization]: _images/PolyglotDatabaseUtilization.jpg
[LogDatabaseUtilization]: _images/LogDatabaseUtilization.jpg
