---
title: Estrategias de creación de particiones de datos
description: Orientaciones sobre cómo separar las particiones para administrarlas y tener acceso a ellas por separado.
author: dragon119
ms.date: 11/04/2018
ms.openlocfilehash: 95dd25ec0081431d45caf89952300fd90299b4c9
ms.sourcegitcommit: 949b9d3e5a9cdee1051e6be700ed169113e914ae
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/05/2018
ms.locfileid: "50983473"
---
# <a name="data-partitioning-strategies"></a>Estrategias de creación de particiones de datos

En este artículo se describen algunas estrategias para crear particiones de datos en varios almacenes de datos de Azure. Para obtener instrucciones generales acerca de cuándo crear particiones de datos y los procedimientos recomendados para hacerlo, consulte [Creación de particiones de datos](./data-partitioning.md)

## <a name="partitioning-azure-sql-database"></a>Creación de particiones en Azure SQL Database

Una única base de datos SQL puede contener un volumen de datos limitado. El rendimiento está restringido por factores arquitectónicos y el número de conexiones simultáneas que admite. 

Los [grupos de bases de datos elásticas](/azure/sql-database/sql-database-elastic-pool) admiten el escalado horizontal de las bases de datos SQL. Mediante los grupos de bases de datos elásticas se pueden crear particiones de datos que se reparten en varias bases de datos SQL. También puede agregar o quitar particiones a medida que crezca y disminuya el volumen de datos que necesita administrar. Los grupos de bases de datos elásticas también pueden ayudar a reducir la contención, ya que distribuyen la carga entre las bases de datos.

Cada partición se implementa como una base de datos SQL. Una partición puede contener más de un conjunto de datos (denominado *shardlet*). Cada base de datos mantiene metadatos que describen los shardlets que contiene. Un shardlet puede ser un único elemento de datos o un grupo de elementos que comparten la misma clave de shardlet. Por ejemplo, en una aplicación de varios inquilinos, la clave de shardlet puede ser el identificador del inquilino y todos los datos de un inquilino se pueden mantener en el mismo shardlet.

Las aplicaciones cliente son responsables de la asociación de un conjunto de datos con una clave de shardlet. Una base de datos SQL independiente actúa como administrador global de mapas de particiones. Esta base de datos tiene una lista de todas las particiones y shardlets del sistema. La aplicación se conecta a la base de datos del administrador del mapa de particiones para obtener una copia de este. Almacena en caché localmente el mapa de particiones y utiliza dicho mapa para enrutar las solicitudes de datos a la partición apropiada. Esta funcionalidad se oculta detrás de una serie de API que se encuentran en la [biblioteca de cliente de Elastic Database](/azure/sql-database/sql-database-elastic-database-client-library), que está disponible para Java y .NET. 

Para más información acerca de los grupos de bases de datos elásticas, consulte [Escalado horizontal con Azure SQL Database](/azure/sql-database/sql-database-elastic-scale-introduction).

Para reducir la latencia y mejorar la disponibilidad, puede replicar la base de datos de administrador global del mapa de particiones. Con los planes de tarifa Premium, puede configurar la replicación geográfica activa para copiar continuamente datos a bases de datos de diferentes regiones. 

Como alternativa, use [Azure SQL Data Sync](/azure/sql-database/sql-database-sync-data) o [Azure Data Factory](/azure/data-factory/) para replicar la base de datos del administrador del mapa de particiones entre regiones. Esta forma de replicación se ejecuta periódicamente y es más adecuada si el mapa de particiones no cambia con frecuencia y no requiere el nivel Premium.

Elastic Database proporciona dos esquemas para asignar datos a shardlets y para almacenarlos en particiones:

* Un **mapa de particiones de lista** asocia una clave única a un shardlet. Por ejemplo, en un sistema de varios inquilinos, los datos de cada inquilino podrían asociarse a una clave única y almacenarse en su propio shardlet. Para garantizar el aislamiento, cada shardlet puede mantenerse dentro de su propia partición.

    ![Uso de un mapa de particiones de lista para almacenar datos de inquilino en particiones independientes](./images/data-partitioning/PointShardlet.png)

* Un **mapa de particiones de intervalos** asocia un conjunto de valores de clave contiguos a un shardlet. Por ejemplo, puede agrupar los datos de un conjunto de inquilinos (cada uno de ellos con su propia clave) en el mismo shardlet. Este esquema es más barato que el primero, ya que los inquilinos comparten el almacenamiento de datos, pero su aislamiento es menor.

    ![Uso de un mapa de particiones de intervalo para almacenar los datos de un intervalo de inquilinos de una partición](./images/data-partitioning/RangeShardlet.png)

Una partición individual puede contener los datos de varios shardlets. Por ejemplo, puede utilizar shardlets de lista para almacenar datos de varios inquilinos no contiguos en la misma partición. También puede mezclar shardlets de intervalo y de lista en la misma partición, aunque se tratarán a través de diferentes mapas. En el siguiente diagrama se ilustra este enfoque:

![Implementación de varios mapas de particiones](./images/data-partitioning/MultipleShardMaps.png)

Los grupo de bases de datos elásticas permiten agregar y quitar particiones cuando el volumen de datos disminuye o crece. Las aplicaciones cliente pueden crear y eliminar particiones dinámicamente y actualizar el administrador del mapa de particiones. sin embargo, eliminar una partición es una operación destructiva que también requiere la eliminación de todos los datos de esa partición.

Si una aplicación necesita dividir una partición en dos particiones independientes o combinar particiones, use la [herramienta de división y combinación](/azure/sql-database/sql-database-elastic-scale-overview-split-and-merge). Esta herramienta se ejecuta como un servicio web de Azure y migra los datos entre particiones de forma segura. 

El esquema de particiones puede afectar considerablemente al rendimiento del sistema. También puede afectar a la frecuencia con que deben agregarse o quitarse particiones, o con que se deben volver a crear particiones de datos. Considere los siguientes puntos:

* Agrupe los datos que se utilizan conjuntamente en la misma partición y evite operaciones que accedan a datos de varias particiones. Una partición es una base de datos SQL por derecho propio y las combinaciones entre bases de datos se deben realizar en el cliente. 

    Aunque SQL Database no admite combinaciones entre bases de datos, puede usar las herramientas de Elastic Database para realizar [consultas en varias particiones](/azure/sql-database/sql-database-elastic-scale-multishard-querying). Una consulta en varias particiones envía consultas individuales a cada base de datos y combina los resultados.

* No diseñe un sistema que tenga dependencias entre las particiones. Las restricciones de integridad referencial, los desencadenadores y los procedimientos almacenados de una base de datos no pueden hacer referencia a otra. 

* Si tiene datos de referencia que las consultas usen con frecuencia, considere la posibilidad de replicar dichos datos entre las particiones. Este enfoque puede eliminar la necesidad de combinar datos en las bases de datos. Lo ideal es que dichos datos sean estáticos o lentos para minimizar el esfuerzo de replicación y reducir las posibilidades de que se vuelvan obsoletos.

* Los shardlets que pertenecen al mismo mapa de particiones deben tener el mismo esquema. Esta regla no se aplica mediante SQL Database, pero la administración y consulta de datos se vuelven muy complejas si cada shardlet tiene un esquema diferente. En su lugar, cree mapas de particiones independientes para cada esquema. Recuerde que los datos que pertenecen a diferentes shardlets pueden almacenarse en la misma partición.

* Las operaciones transaccionales solo se admiten para los datos que se encuentren en una partición, no entre particiones. Las transacciones pueden abarcar shardlets siempre que formen parte de la misma partición. Por consiguiente, si la lógica de negocios debe realizar transacciones, almacene los datos en la misma partición o implemente la coherencia eventual. 

* Coloque las particiones cerca de los usuarios que acceden a los datos de dichas particiones. Esta estrategia ayuda a reducir la latencia.

* Evite tener una mezcla de particiones muy activas y relativamente inactivas. Pruebe a distribuir la carga uniformemente entre las particiones. Esto puede requerir la aplicación de algoritmos hash a las claves de particionamiento.  Si está geolocalizando las particiones, asegúrese de que las claves hash se asignan a shardlets de particiones almacenadas cerca de los usuarios que tienen acceso a esos datos.

### <a name="partitioning-azure-table-storage"></a>Creación de particiones en el almacenamiento de tablas de Azure

El Almacenamiento de tablas de Azure es un almacén del par clave-valor diseñado en torno a la creación de particiones. Todas las entidades se almacenan en una partición y las particiones son administradas internamente por el almacenamiento de tablas de Azure. Cada entidad almacenada en una tabla debe proporcionar una clave de dos partes que incluye:

* **La clave de la partición**. Se trata de un valor de cadena que determina la partición en que Almacenamiento de tablas de Azure colocará la entidad. Todas las entidades con la misma clave de partición se almacenan en la misma partición.
* **La clave de la fila**. Es un valor de cadena que identifica la entidad dentro de la partición. Todas las entidades dentro de una partición se ordenan léxicamente, en orden ascendente, por parte de esta clave. La combinación de clave de fila/partición debe ser única para cada entidad y no puede superar 1 KB de longitud.

Si se agrega una entidad a una tabla con una clave de partición no usada anteriormente, Almacenamiento de tablas de Azure crea una nueva partición para esta entidad. Otras entidades con la misma clave de partición se almacenarán en la misma partición.

Este mecanismo implementa de forma efectiva una estrategia de escalado automático. Cada partición se almacena en el mismo servidor de un centro de datos de Azure para ayudar a garantizar que las consultas que recuperan datos de una sola partición se ejecutan rápidamente. 

Microsoft ha publicado los [objetivos de escalabilidad] de Azure Storage. Si es probable que su sistema supere estos límites, considere la posibilidad de dividir las entidades en varias tablas. Utilice particiones verticales para dividir los campos en los grupos a los que se vaya a acceder en conjunto con mayor probabilidad.

El diagrama siguiente muestra la estructura lógica de una cuenta de almacenamiento de ejemplo. La cuenta de almacenamiento contiene tres tablas: información del cliente, información del producto e información del pedido. 

![Las tablas y las particiones de una cuenta de almacenamiento de ejemplo](./images/data-partitioning/TableStorage.png)

Cada tabla tiene varias particiones.

- En la tabla Información del cliente, los datos se particionan según la ciudad en la que está ubicado el cliente. La clave de fila contiene el Id. de cliente. 
- En la tabla Información del producto, los productos se particionan por categoría de producto y la clave de fila contiene el número de producto. 
- En la tabla Order Info (Información del pedido), los pedidos se particionan por fecha de pedido y la clave de fila especifica el momento en que recibió el pedido. Tenga en cuenta que todos los datos se ordenan por la clave de fila de cada partición.

Considere los siguientes puntos cuando diseñe las entidades para el almacenamiento de tablas de Azure:

* Seleccione una clave de partición y clave de fila por la forma en que se accede a los datos. Elija una combinación de claves de partición y fila que admita la mayoría de las consultas. Las consultas más eficaces recuperan los datos especificando la clave de partición y la clave de fila. Las consultas que especifican una clave de partición y un intervalo de claves de fila pueden completarse mediante el análisis de una sola partición. Esto es relativamente rápido porque los datos se mantienen en el orden de la clave de fila. Si las consultas no especifican la partición que se va a examinar, se deben examinar todas.

- Si una entidad tiene una clave natural, a continuación, úsela como clave de partición y especifique una cadena vacía como clave de fila. Si una entidad tiene una clave compuesta que conste de dos propiedades, seleccione la propiedad que cambie más despacio como clave de partición y la otra como clave de fila. Si una entidad tiene más de dos propiedades de clave, use una concatenación de propiedades para proporcionar las claves de partición y fila.

* Si realiza regularmente consultas que buscan datos con campos que no sean las claves de partición y fila, considere la posibilidad de implementar el [patrón de tabla de índice] o de usar otro almacén de datos que admita la indexación, como Cosmos DB.

* Si genera claves de partición mediante una secuencia monotónica (como, "0001", "0002" y "0003") y cada partición solo contiene una cantidad limitada de datos, es posible que el Almacenamiento de tablas de Azure agrupe físicamente estas particiones en el mismo servidor. Azure Storage supone que es muy probable que la aplicación realice consultas en un intervalo contiguo de particiones (consultas por rango) y está optimizada para este caso. Sin embargo, este enfoque puede provocar la aparición de puntos con mucho tráfico, ya que es probable que todas las inserciones de nuevas entidades se concentren en uno de los extremos del rango contiguo. También puede reducir la escalabilidad. Para distribuir la carga más uniformemente, considere la posibilidad de crear un algoritmo hash de la clave de partición.

* El almacenamiento de tablas de Azure admite operaciones transaccionales para entidades que pertenecen a la misma partición. Una aplicación puede realizar varias operaciones de inserción, actualización, eliminación, sustitución o combinación como una unidad atómica, siempre que la transacción no incluya más de cien entidades y que la carga de la solicitud no supere los 4 MB. Las operaciones que abarcan varias particiones no son transaccionales y pueden requerir la implementación de la coherencia eventual. Para más información acerca del almacenamiento de tablas y las transacciones, consulte [Performing entity group transactions] (Realización de transacciones en el grupo de entidades).

* Tenga en cuenta la granularidad de la clave de partición:

  * El uso de la misma clave de partición en todas las entidades genera una única partición que se mantiene en un servidor. Esto impide el escalado horizontal de la partición y centra la carga en un único servidor. Como resultado, este enfoque solo es adecuado para almacenar un pequeño número de entidades. Sin embargo, garantiza que todas las entidades puedan participar en transacciones del grupo de entidades.

  * El uso de una clave de partición única para cada entidad provoca que el servicio Table Storage cree una partición independiente para cada entidad, lo cual puede dar lugar a un número elevado de particiones pequeñas. Este enfoque es más escalable que el uso de una clave de partición única, pero no es posible realizar transacciones de grupo de entidad. Además, las consultas que capturan más de una entidad podrían implicar lecturas desde más de un servidor. Sin embargo, si la aplicación realiza consultas por rango, el uso de una secuencia monotónica para las claves de partición puede ayudar a optimizar dichas consultas.

  * Compartir la clave de partición en un subconjunto de entidades permite agrupar las entidades relacionadas de la misma partición. Se pueden realizar operaciones que implican entidades relacionadas con transacciones de grupo de entidades, y es posible dar respuesta a las consultas que capturan un conjunto de entidades relacionadas mediante el acceso a un único servidor.

Para más información, consulte [Guía de diseño de Table Storage].

## <a name="partitioning-azure-blob-storage"></a>Creación de particiones en el almacenamiento de blobs de Azure

Azure Blob Storage permite almacenar objetos binarios grandes. Use blobs en bloques cuando sea necesario cargar o descargar grandes volúmenes de datos rápidamente. Use blobs en páginas para aplicaciones que requieran acceso aleatorio en lugar de acceso de serie a partes de los datos.

Cada blob (en bloque o en página) se mantiene en un contenedor de una cuenta de almacenamiento de Azure. Puede usar contenedores para agrupar los blobs relacionados que tengan los mismos requisitos de seguridad. Esta agrupación es más lógica que física. Dentro de un contenedor, cada blob tiene un nombre único.

La clave de partición de un blob es el nombre de la cuenta más el nombre del contenedor y el nombre del blob. La clave de partición se usa para particionar datos en intervalos con una carga equilibrada en el sistema. Los blobs se pueden distribuir en varios servidores para escalar horizontalmente el acceso a ellos, pero los blobs únicos solo pueden proporcionarse mediante servidores únicos.  

Si su esquema de nombre usa marcas de tiempo o identificadores numéricos, puede provocar que vaya demasiado tráfico a una partición, lo que limita el eficaz equilibrio de carga del sistema. Por ejemplo, si tiene operaciones diarias que usan un objeto blob con una marca de tiempo como *aaaa-mm-dd*, todo el tráfico de dichas operaciones irían a un solo servidor de particiones. En su lugar, considere la posibilidad de colocar un hash de 3 dígitos delante del nombre. Para más información, consulte [Convención de nomenclatura de particiones](/azure/storage/common/storage-performance-checklist#subheading47)

Las acciones de escribir un solo bloque o página son atómicas, pero las operaciones que abarcan bloques, páginas o blobs no lo son. Si necesita asegurar la coherencia cuando se realizan operaciones de escritura en bloques, páginas y blobs, realice un bloqueo de escritura mediante el uso de una concesión de blob.

## <a name="partitioning-azure-storage-queues"></a>Creación de particiones en colas de almacenamiento de Azure

Las colas de almacenamiento de Azure le permiten implementar mensajería asincrónica entre procesos. Una cuenta de almacenamiento de Azure puede contener cualquier número de colas y cada una de ellas puede contener cualquier número de mensajes. La única limitación es el espacio disponible en la cuenta de almacenamiento. El tamaño máximo de un mensaje individual es de 64 KB. Si necesita mensajes de mayor tamaño, considere la posibilidad de usar colas de Azure Service Bus en su lugar.

Cada cola de almacenamiento tiene un nombre único dentro de la cuenta de almacenamiento en la que se encuentra. Azure crea colas de particiones basadas en el nombre. Todos los mensajes de la misma cola se almacenan en la misma partición, que se controla mediante un solo servidor. Distintas colas pueden ser administradas por distintos servidores para ayudar a equilibrar la carga. La asignación de colas a servidores es transparente para las aplicaciones y usuarios.

 En una aplicación a gran escala, no utilice la misma cola de almacenamiento para todas las instancias de la aplicación porque esto puede provocar que el servidor que hospeda la cola soporte una carga excesiva. En su lugar, use colas diferentes para distintas áreas funcionales de la aplicación. Las colas de almacenamiento de Azure no admiten transacciones, por lo que dirigir los mensajes a distintas colas debería tener poco impacto en la coherencia de los mensajes.

Una cola de almacenamiento de Azure puede controlar hasta 2000 mensajes por segundo.  Si necesita procesar los mensajes a una velocidad mayor, puede crear varias colas. Por ejemplo, en una aplicación global, cree colas de almacenamiento independientes en cuentas de almacenamiento separadas para controlar las instancias de aplicación que se ejecutan en cada región.

## <a name="partitioning-azure-service-bus"></a>Creación de particiones en Azure Service Bus
Azure Service Bus usa un agente de mensajes para controlar los mensajes enviados a una cola o un tema de Service Bus. De forma predeterminada, todos los mensajes enviados a una cola o tema se controlan mediante el mismo proceso de agente de mensaje. Esta arquitectura puede colocar una limitación en el rendimiento general de la cola de mensajes. Sin embargo, también puede dividir una cola o tema cuando se crea. Para ello, establezca la propiedad *EnablePartitioning* de la descripción de la cola o el tema en *true*.

Una cola o tema con particiones se divide en varios fragmentos, cada uno de ellos respaldado por un almacén de mensajes y un agente de mensajes independientes. Service Bus asume la responsabilidad de crear y administrar estos fragmentos. Cuando una aplicación envía un mensaje a una cola o tema particionado, Service Bus asigna el mensaje a un fragmento de esa cola o tema. Cuando una aplicación recibe un mensaje de una cola o suscripción, Service Bus comprueba cada fragmento para detectar el siguiente mensaje disponible y, a continuación, lo pasa a la aplicación para su procesamiento.

Esta estructura le ayuda a distribuir la carga entre los agentes de mensajes y los almacenes de mensajes, aumentando la escalabilidad y mejorando la disponibilidad. Si el almacén de mensajes o el agente de mensajes de un fragmento no está disponible temporalmente, Service Bus puede recuperar mensajes de uno de los demás fragmentos disponibles.

Service Bus asigna un mensaje a un fragmento de la siguiente manera:

* Si el mensaje pertenece a una sesión, se envían todos los mensajes con el mismo valor para la propiedad *SessionId* al mismo fragmento.
* Si el mensaje no pertenece a una sesión pero el remitente ha especificado un valor para la propiedad *PartitionKey*, todos los mensajes con el mismo valor *PartitionKey* se enviarán al mismo fragmento.

  > [!NOTE]
  > Si se especifican las propiedades *SessionId* y *PartitionKey*, se deben establecer en el mismo valor; de lo contrario, se rechazará el mensaje.
  >
  >
* Si las propiedades *SessionId* y *PartitionKey* de un mensaje no se han especificado pero se habilita la detección de duplicados, se usará la propiedad *MessageId*. Todos los mensajes con el mismo *MessageId* se dirigirán al mismo fragmento.
* Si los mensajes no incluyen una propiedad *SessionId, PartitionKey,* o *MessageId*, Service Bus asigna los mensajes a los fragmentos de manera secuencial. Si un fragmento no está disponible, Service Bus pasará al siguiente. Esto significa que un error temporal de la infraestructura de mensajería no provoca un error en la operación de envío de mensajes.

Debe tener en cuenta los siguientes puntos cuando decida si crear una partición en un tema o una cola de mensajes de Service Bus o un tema, y cómo realizarla:

* Los temas y colas de Service Bus se crean dentro del ámbito de un espacio de nombres de Service Bus. Actualmente Service Bus permite hasta 100 colas o temas particionados por espacio de nombres.
* Cada espacio de nombres de Service Bus impone cuotas en los recursos disponibles, como el número de suscripciones por tema, el número de solicitudes de envío y recepción simultáneas por segundo y el número máximo de conexiones simultáneas que pueden establecerse. Estas cuotas se documentan en [Cuotas de Service Bus]. Si espera que superar estos valores, cree espacios de nombres adicionales con sus propios temas y colas y distribuya el trabajo en estos espacios de nombres. Por ejemplo, en una aplicación global, cree espacios de nombres independientes en cada región y configure instancias de la aplicación para usar las colas y temas en el espacio de nombres más cercano.
* Los mensajes que se envían como parte de una transacción deben especificar una clave de partición. Esta puede ser una propiedad *SessionId*, *PartitionKey* o *MessageId*. Todos los mensajes que se envían como parte de la misma transacción deben especificar la misma clave de partición porque deben tratarse mediante el mismo proceso de agente de mensaje. No se pueden enviar mensajes a diferentes colas o temas de dentro de la misma transacción.
* Temas y colas con particiones no pueden configurarse para eliminarse automáticamente cuando se queden inactivas.
* Si está creando soluciones multiplataforma o híbridas, en este momento no podrá utilizar colas o temas con particiones con el protocolo Advanced Message Queuing Protocol (AMQP).

## <a name="partitioning-cosmos-db"></a>Creación de particiones en Cosmos DB

Azure Cosmos DB es una base de datos NoSQL que puede almacenar documentos JSON mediante la [API de SQL de Azure Cosmos DB][cosmosdb-sql-api]. En una base de datos de Cosmos DB, un documento es una representación serializada de un objeto o de otro elemento de datos. No se aplican esquemas fijos, excepto que cada documento debe contener un identificador único.

Los documentos se organizan en colecciones. Puede agrupar documentos relacionados entre sí en una colección. Por ejemplo, en un sistema que mantiene las entradas de un blog, puede almacenar el contenido de cada entrada como un documento de una colección. También puede crear colecciones para cada tipo de asunto. O bien, en una aplicación para varios inquilinos, como un sistema donde diferentes autores controlan y administran sus propias publicaciones de blog, podría crear particiones en los blogs por autor y crear una colección independiente para cada autor. El espacio de almacenamiento asignado a las colecciones es elástico y puede reducirse o ampliarse según sea necesario.

Cosmos DB admite la creación automática de particiones de datos basadas en una clave de partición definida por la aplicación. Una *partición lógica* es una partición que almacena todos los datos de un valor de clave de partición única. Todos los documentos que comparten el mismo valor de la clave de partición se colocan dentro de la misma partición lógica. Cosmos DB distribuye los valores según el código hash de la clave de partición. Una partición lógica tiene un tamaño máximo de 10 GB. Por tanto, la elección de la clave de partición es una decisión importante que debe realizarse en el momento del diseño. Elija una propiedad con una amplia gama de valores e incluso patrones de acceso. Para más información, consulte [Partición y escalado en Azure Cosmos DB](/azure/cosmos-db/partition-data).

> [!NOTE]
> Cada base de datos de Cosmos DB tiene un *nivel de rendimiento* que determina la cantidad de recursos que obtiene. Los niveles de rendimiento están asociados a un límite de velocidad de *unidad de solicitud*. El límite de velocidad de unidad de solicitud especifica el volumen de los recursos reservados y disponibles para el uso exclusivo de esa colección. El costo de una colección depende del nivel de rendimiento seleccionado para esa colección. Cuanto mayor sea el nivel re rendimiento (y el límite de velocidad de la unidad de solicitud), mayor será el costo. Puede ajustar el nivel de rendimiento de una colección mediante el Portal de Azure. Para más información, consulte [Unidades de solicitud en Azure Cosmos DB][cosmos-db-ru].
>
>

Si el mecanismo de creación de particiones que proporciona Cosmos DB no es suficiente, puede que necesite particionar los datos en el nivel de la aplicación. Las colecciones de documentos proporcionan un mecanismo natural para dividir los datos de una base de datos única. La manera más sencilla de implementar las particiones es crear una colección para cada partición. Los contenedores son recursos lógicos y pueden abarcar uno o varios servidores. Los contenedores de tamaño fijo tienen un límite máximo de 10 GB y un rendimiento de 10 000 RU/s. Un número ilimitado de contenedores no tienen un tamaño máximo de almacenamiento, pero deben especificar una clave de partición. Con el particionamiento de la aplicación, la aplicación cliente debe dirigir las solicitudes a la partición apropiada, normalmente mediante la implementación de su propio mecanismo de asignación en función de algunos atributos de los datos que definen la clave de partición. 

Todas las bases de datos se crean en el contexto de una cuenta de base de datos de Cosmos DB. Una sola cuenta puede contener varias bases de datos, y especifica en qué regiones se crean las bases de datos. Cada cuenta también impone su propio control de acceso. Puede utilizar cuentas de Cosmos DB para localizar geográficamente particiones (colecciones dentro de bases de datos) cerca de los usuarios que necesitan tener acceso a ellas y aplicar restricciones de modo que solo esos usuarios puedan conectarse.

A la hora de decidir cómo particionar los datos con la API de SQL de Cosmos DB, tenga en cuenta los siguientes puntos:

* **Los recursos disponibles para una base de datos de Cosmos DB están sujetos a las limitaciones de cuota de la cuenta**. Cada base de datos puede contener un número de colecciones y cada colección está asociada a un nivel de rendimiento que rige el límite de velocidad de RU (rendimiento reservado) para esa colección. Para más información, consulte [Límites, cuotas y restricciones de suscripción y servicios de Microsoft Azure][azure-limits].
* **Cada documento debe tener un atributo que pueda usarse para identificar de manera única dicho documento dentro de la colección en la que se encuentra**. Este atributo es diferente de la clave de partición, que define en qué colección se encuentra el documento. Una colección puede contener un gran número de documentos. En teoría, solo está limitada por la longitud máxima del identificador del documento. El identificador del documento puede contener hasta 255 caracteres.
* **Todas las operaciones de un documento se realizan en el contexto de una transacción. Las transacciones se limitan a la colección que contiene el documento.** Si se produce un error en una operación, se revierte el trabajo que ha realizado. Mientras se realiza una operación sobre un documento, los cambios realizados están sujetos a aislamiento a nivel de instantánea. Este mecanismo garantiza que si, por ejemplo, se produce un error en una solicitud para crear un nuevo documento, otro usuario que consulte la base de datos al mismo tiempo no verá un documento parcial que luego se elimine.
* **Las consultas de base de datos también se limitan al nivel de colección**. Una sola consulta solo puede recuperar datos de una colección. Si necesita recuperar datos de varias colecciones, debe consultar cada colección individualmente y combinar los resultados en el código de aplicación.
* **Las bases de datos de Cosmos DB admiten elementos programables que pueden almacenarse en una colección junto con los documentos**. Estos incluyen procedimientos almacenados, funciones definidas por el usuario y desencadenadores (escritos en JavaScript). Estos elementos pueden tener acceso a cualquier documento en la misma colección. Además, estos elementos se ejecutan dentro del ámbito de la transacción de ambiente (en el caso de un desencadenador que se activa como resultado de una operación de crear, eliminar o reemplazar realizada en un documento), o iniciando una nueva transacción (en el caso de un procedimiento almacenado que se ejecuta como resultado de una solicitud de cliente explícita). Si el código de un elemento programable produce una excepción, la transacción se revierte. Puede usar procedimientos almacenados y desencadenadores para mantener la integridad y la coherencia entre los documentos, pero estos documentos deben formar parte de la misma colección.
* **Debe procurar que las colecciones que desea almacenar en las bases de datos no superen los límites de rendimiento definidos por los niveles de rendimiento de las colecciones**. Para más información, consulte [Unidades de solicitud en Azure Cosmos DB][cosmos-db-ru]. Si prevé alcanzar estos límites, considere la posibilidad de dividir las colecciones entre bases de datos en diferentes cuentas para reducir la carga de cada colección.

## <a name="partitioning-azure-search"></a>Creación de particiones en Azure Search
La función de búsqueda de datos suele ser el método principal de navegación y exploración proporcionado por muchas aplicaciones web. Permite a los usuarios encontrar recursos rápidamente (por ejemplo, los productos de una aplicación de comercio electrónico) según determinadas combinaciones de criterios de búsqueda. El servicio Azure Search proporciona capacidades de búsqueda de texto completo a través de contenido web e incluye características como las consultas sugeridas de escritura anticipada basadas en coincidencias cercanas y en la navegación por facetas. Para más información, consulte [¿Qué es Azure Search?]

Azure Search almacena contenido en el que es posible realizar búsquedas como documentos JSON en una base de datos. El usuario es quien define los índices que especifican los campos de búsqueda de estos documentos y proporciona estas definiciones al servicio Azure Search. Cuando se emite una solicitud de búsqueda, Azure Search usa los índices adecuados para buscar los elementos coincidentes.

Para reducir la contención, es posible dividir el almacenamiento utilizado por Azure Search en 1, 2, 3, 4, 6 o 12 particiones, y cada partición se puede replicar hasta 6 veces. El producto del número de particiones multiplicado por el número de réplicas se denomina *unidad de búsqueda*. Una única instancia de Azure Search puede contener un máximo de 36 unidades de búsqueda (una base de datos con 12 particiones solo admite un máximo de 3 réplicas).

Se le facturará cada SU que se asigne a su servicio. A medida que crezca el volumen de contenido sobre el que es posible realizar búsquedas o la tasa de solicitudes de búsqueda, puede agregar unidades de búsqueda a una instancia existente de Azure Search para administrar la carga adicional. El propio servicio Azure Search distribuye los documentos uniformemente entre las particiones. Actualmente no se admite la aplicación de ninguna estrategia de partición manual.

Cada partición puede contener un máximo de 15 millones de documentos u ocupar 300 GB de espacio de almacenamiento (lo que sea menor). Puede crear hasta 50 índices. El rendimiento del servicio varía y depende de la complejidad de los documentos, los índices disponibles y los efectos de la latencia de la red. De media, una única réplica (1 unidad de búsqueda) debe ser capaz de administrar 15 consultas por segundo, aunque es recomendable realizar pruebas comparativas realizadas con sus propios datos para obtener una medida más precisa del rendimiento. Para más información, consulte [Límites de servicio en Azure Search].

> [!NOTE]
> Puede almacenar un conjunto limitado de tipos de datos en documentos que se pueden buscar, incluidas cadenas, valores booleanos, datos numéricos, datos de fecha y hora y algunos datos geográficos. Para obtener más detalles, consulte la página [Tipos de datos admitidos (Azure Search)] en el sitio web de Microsoft.
>
>

Tiene control limitado sobre cómo divide en particiones el servicio Azure Search los datos de cada instancia del servicio. Sin embargo, en un entorno global puede mejorar el rendimiento y reducir la latencia y la contención aún más mediante la partición del propio servicio; para ello, aplique alguna de las estrategias siguientes:

* Cree una instancia de Azure Search en cada región geográfica y asegúrese de que las aplicaciones cliente se dirigen hacia la instancia disponible más cercana. Esta estrategia requiere la replicación de las actualizaciones de contenido de búsqueda de manera oportuna en todas las instancias del servicio.
* Cree dos niveles en Azure Search:

  * Un servicio local en cada región que contenga los datos a los que se accede con mayor frecuencia por parte de los usuarios de esa región. Los usuarios podrá dirigir solicitudes aquí para obtener resultados rápidos pero limitados.
  * Un servicio global que abarque todos los datos. Los usuarios podrán dirigir solicitudes aquí para obtener resultados más lentos pero más completos.

Este enfoque es más adecuado cuando existe una variación regional considerable en los datos que se están buscando.

## <a name="partitioning-azure-redis-cache"></a>Creación de particiones en Azure Redis Cache
Azure Redis Cache proporciona un servicio de almacenamiento en caché compartido en la nube que se basa en el almacén de datos de clave-valor de Redis. Como su nombre implica, el servicio Azure Redis Cache está concebido como una solución de almacenamiento en caché. Úselo solo para almacenar datos provisionales y no como un almacén de datos permanentes. Las aplicaciones que usan Azure Redis Cache podrán seguir funcionando si la caché no está disponible. Azure Redis Cache admite la replicación principal y secundaria para proporcionar alta disponibilidad, pero actualmente limita el tamaño de caché máximo a 53 GB. Si necesita más espacio que este, deberá crear cachés adicionales. Para más información, consulte [Azure Redis Cache].

Crear particiones en un almacén de datos Redis implica dividir los datos entre instancias del servicio Redis. Cada instancia constituye una sola partición. Azure Redis Cache abstrae los servicios de Redis detrás de una fachada y no los expone directamente. La manera más sencilla de implementar la creación de particiones es crear varias instancias de Azure Redis Cache y repartir los datos entre ellas.

Puede asociar cada uno de los elementos de datos con un identificador (una clave de partición) que especifica en qué caché se almacenan. La lógica de la aplicación cliente puede usar luego este identificador para enrutar las solicitudes a la partición apropiada. Este esquema es muy sencillo, pero si cambia el esquema de creación de particiones (por ejemplo, si se crean instancias de Azure Redis Cache adicionales), es posible que las aplicaciones cliente deban volver a configurarse.

Redis nativo (no Azure Redis Cache) admite particiones de servidor basadas en la agrupación en clústeres de Redis. En este enfoque, los datos se dividen uniformemente entre servidores mediante un mecanismo hash. Cada servidor Redis almacena metadatos que describen el intervalo de claves hash que contiene la partición, y también contiene información acerca de qué claves hash se encuentran en las particiones de otros servidores.

Las aplicaciones cliente simplemente envían solicitudes a cualquiera de los servidores Redis participantes (probablemente el más cercano). El servidor Redis examina la solicitud del cliente. Si se puede resolver localmente, este realiza la operación solicitada. De lo contrario, reenvía la solicitud al servidor apropiado.

Este modelo se implementa mediante el uso de clústeres de Redis y se describe con más detalle en la página [Tutorial de clúster Redis] en el sitio web de Redis. La agrupación en clústeres de Redis es transparente para las aplicaciones cliente. Es posible agregar servidores Redis adicionales al clúster (y es posible volver a crear particiones en los datos) sin necesidad de volver a configurar los clientes.

> [!IMPORTANT]
> Actualmente Azure Redis Cache no admite la agrupación en clústeres Redis. Si desea implementar este enfoque con Azure, deberá implementar entonces sus propios servidores Redis instalando Redis en un conjunto de máquinas virtuales de Azure y configurándolas manualmente. La página [Running Redis on a CentOS Linux VM in Windows Azure] (Ejecución de Redis en una máquina virtual Linux de CentOS en Azure) le guía a través de un ejemplo que muestra cómo crear y configurar un nodo Redis que se ejecuta como una máquina virtual de Azure.
>
>

La página [Partitioning: how to split data among multiple Redis instances] (Creación de particiones: cómo dividir los datos entre varias instancias de Redis) del sitio web de Redis ofrece más información acerca de cómo implementar la creación de particiones con Redis. En el resto de esta sección se supone que se está implementando las particiones del lado cliente o asistidas por el proxy.

Deberá tener en cuenta los siguientes puntos a la hora de decidir cómo crear particiones en los datos con Azure Redis Cache:

* El servicio Azure Redis Cache no está pensado para actuar como un almacén de datos permanentes, por lo que, independientemente del esquema de particiones que implemente, el código de la aplicación debe estar preparado para recuperar datos desde una ubicación distinta de la caché.
* Los datos a los que se suele acceder en conjunto deben mantenerse en la misma partición. Redis es un eficaz almacén de clave-valor que proporciona varios mecanismos enormemente optimizados para estructurar los datos. Estos mecanismos pueden ser uno de los siguientes elementos:

  * Cadenas simples (datos binarios de hasta 512 MB)
  * Tipos de agregado como listas (que pueden actuar como colas y pilas)
  * Conjuntos (ordenados y desordenados)
  * Algoritmos hash (que pueden agrupar campos relacionados, como los elementos que representan los campos de un objeto)
* Los tipos de agregado permiten asociar muchos valores relacionados con la misma clave. Una clave de Redis identifica una lista, un conjunto o un valor hash en lugar de los elementos de datos que contiene. Estos tipos están disponibles con Azure Redis Cache y se describen en la página [Tipo de datos] (Tipos de datos) del sitio web de Redis. Por ejemplo, en la parte de un sistema de comercio electrónico que realiza el seguimiento de los pedidos realizados por los clientes, los detalles de cada cliente podrían almacenarse en un hash de Redis con clave utilizando el identificador de cliente. Cada valor hash puede contener una colección de identificadores de pedido para el cliente. Un conjunto Redis independiente podría contener los pedidos, estructurados como algoritmos hash y codificados por medio del identificador de pedido. La figura 8 muestra esta estructura. Tenga en cuenta que Redis no implementa ningún tipo de integridad referencial, por lo que es responsabilidad del desarrollador mantener las relaciones entre clientes y pedidos.

![Estructura sugerida en el almacenamiento de Redis para registrar los pedidos de clientes y sus detalles](./images/data-partitioning/RedisCustomersandOrders.png)

*Ilustración 8. Estructura sugerida en el almacenamiento de Redis para registrar los pedidos de clientes y sus detalles*

> [!NOTE]
> En Redis, todas las claves son valores de datos binarios (como cadenas Redis) y pueden contener hasta 512 MB de datos. En teoría, una clave puede contener casi cualquier información. No obstante, debe adoptar una convención de nomenclatura coherente para las claves que sea descriptiva del tipo de datos y que identifique la entidad, pero que no sea demasiado larga. Es habitual utilizar claves con el formato "tipo_entidad: ID". Por ejemplo, puede usar "cliente: 99" para indicar la clave de un cliente con el identificador 99.
>
>

* Puede implementar las particiones verticales almacenando información relacionada en agregaciones diferentes en la misma base de datos. Por ejemplo, en una aplicación de comercio electrónico podría almacenar información a la que accede frecuentemente acerca de los productos en un hash de Redis y la información detallada usada con menos frecuencia en otro.
  Ambos valores hash pueden utilizar el mismo identificador de producto como parte de la clave. Por ejemplo, puede utilizar "producto: *nn*" (donde *nn* es el identificador del producto) para la información del producto y "detalles_producto: *nn*" para los datos detallados. Esta estrategia puede ayudar a reducir el volumen de datos que es probable que recuperen la mayoría de las consultas.
* Puede volver a particionar un almacén de datos de Redis, pero tenga en cuenta que es una tarea compleja y lenta. La agrupación en clústeres de Redis puede volver a crear particiones de datos automáticamente, pero esta función no está disponible con Azure Redis Cache. Por lo tanto, al diseñar el esquema de partición, deje suficiente espacio libre en cada partición para permitir el crecimiento de datos que se espera con el tiempo. Sin embargo, recuerde que Azure Redis Cache está diseñado para almacenar datos en la caché temporalmente y que los datos que se mantienen en la memoria caché pueden tener una duración limitada especificada como valor de período de vida (TTL). En el caso de los datos relativamente volátiles, el período de vida debe ser corto; sin embargo, para datos estáticos, el período de vida puede ser mucho más largo. Evite almacenar grandes cantidades de datos de larga duración en la caché si el volumen de estos datos es probable que llene la caché. Puede especificar una directiva de expulsión que provoque que Azure Redis Cache elimine datos si el espacio es crítico.

  > [!NOTE]
  > Cuando utiliza Caché en Redis de Azure, puede especificar el tamaño máximo de caché (de 250 MB a 53 GB) seleccionando el plan de tarifa adecuado. Sin embargo, una vez que se ha creado Azure Redis Cache, no podrá aumentar (ni reducir) su tamaño.
  >
  >
* Los lotes y las transacciones de Redis no pueden abarcar varias conexiones, por lo que todos los datos afectados por un lote o transacción deben permanecer en la misma base de datos (partición).

  > [!NOTE]
  > Una secuencia de operaciones en una transacción Redis no es necesariamente atómica. Los comandos que componen una transacción se comprueban y se ponen en cola antes de ejecutarse. Si se produce un error durante esta fase, se descarta toda la cola. Sin embargo, una vez que la transacción se haya enviado correctamente, se ejecutarán los comandos en cola en secuencia. Si se produce un error en algún comando, únicamente ese comando deja de ejecutarse. Se ejecutarán todos los comandos situados delante y detrás en la cola. Para obtener más información, visite la página [Transactions] (Transacciones) en el sitio web de Redis.
  >
  >
* Redis admite un número limitado de operaciones atómicas. Las únicas operaciones de este tipo que admiten varias claves y valores son operaciones MGET y MSET. Las operaciones MGET devuelven una colección de valores para una lista especificada de claves, y las operaciones MSET almacenan una colección de valores para una lista especificada de claves. Si necesita usar estas operaciones, los pares clave-valor a los que hacen referencia los comandos MSET y MGET deben almacenarse en la misma base de datos.

## <a name="partitioning-azure-service-fabric"></a>Creación de particiones en Azure Service Fabric
Azure Service Fabric es una plataforma de microservicios que proporciona un sistema de tiempo de ejecución para las aplicaciones distribuidas en la nube. Service Fabric admite archivos ejecutables de invitado de .NET, servicios con y sin estado, y contenedores. Los servicios con estado proporcionan una [colección confiable] [ service-fabric-reliable-collections] para almacenar datos de forma persistente en una colección de pares clave-valor en el clúster de Service Fabric. Para más información acerca de las estrategias de las claves de creación de particiones en una colección de confianza, consulte [Directrices y recomendaciones de las colecciones confiables en Azure Service Fabric].

### <a name="more-information"></a>Más información
* [Información general de Azure Service Fabric] es una introducción a Azure Service Fabric.
* [Partición de los servicios confiables de Service Fabric] proporciona más información acerca de los servicios confiables en Azure Service Fabric.

## <a name="partitioning-azure-event-hubs"></a>Creación de particiones en Azure Event Hubs

[Azure Event Hubs][event-hubs] está diseñado para la trasmisión de datos a escala masiva, y la creación de particiones está integrada en el servicio para habilitar el escalado horizontal. Cada consumidor lee únicamente una partición específica del flujo de mensajes. 

El publicador de eventos solo conoce su clave de partición, no la partición en la que se publican los eventos. Este desacoplamiento de la clave y la partición evita al remitente la necesidad de conocer demasiado sobre el procesamiento de bajada. (También se puede enviar eventos directamente a una partición determinada, pero generalmente no se recomienda).  

Considere la posibilidad de escalar a largo plazo cuando seleccione el número de particiones. Después de crear un centro de eventos, no se puede cambiar el número de particiones. 

Para más información sobre el uso de particiones en Event Hubs, consulte [¿Qué es Event Hubs?].

Para ver otras consideraciones sobre el equilibrio entre la disponibilidad y la coherencia, consulte [Disponibilidad y coherencia en Event Hubs].

[Disponibilidad y coherencia en Event Hubs]: /azure/event-hubs/event-hubs-availability-and-consistency
[azure-limits]: /azure/azure-subscription-service-limits
[Azure Content Delivery Network]: /azure/cdn/cdn-overview
[Azure Redis Cache]: https://azure.microsoft.com/services/cache/
[Azure Storage Scalability and Performance Targets]: /azure/storage/storage-scalability-targets
[Guía de diseño de Table Storage]: /azure/storage/storage-table-design-guide
[Building a Polyglot Solution]: https://msdn.microsoft.com/library/dn313279.aspx
[cosmos-db-ru]: /azure/cosmos-db/request-units
[Data Access for Highly-Scalable Solutions: Using SQL, NoSQL, and Polyglot Persistence]: https://msdn.microsoft.com/library/dn271399.aspx
[Data consistency primer]: https://aka.ms/Data-Consistency-Primer
[Data Partitioning Guidance]: https://msdn.microsoft.com/library/dn589795.aspx
[Tipo de datos]: https://redis.io/topics/data-types
[cosmosdb-sql-api]: /azure/cosmos-db/sql-api-introduction
[Elastic Database features overview]: /azure/sql-database/sql-database-elastic-scale-introduction
[event-hubs]: /azure/event-hubs
[Federations Migration Utility]: https://code.msdn.microsoft.com/vstudio/Federations-Migration-ce61e9c1
[Directrices y recomendaciones de las colecciones confiables en Azure Service Fabric]: /azure/service-fabric/service-fabric-reliable-services-reliable-collections-guidelines
[patrón de tabla de índice]: ../patterns/index-table.md
[Materialized View Pattern]: ../patterns/materialized-view.md
[Multi-shard querying]: /azure/sql-database/sql-database-elastic-scale-multishard-querying
[Información general de Azure Service Fabric]: /azure/service-fabric/service-fabric-overview
[Partición de los servicios confiables de Service Fabric]: /azure/service-fabric/service-fabric-concepts-partitioning
[Partitioning: how to split data among multiple Redis instances]: https://redis.io/topics/partitioning
[Performing Entity Group Transactions]: /rest/api/storageservices/Performing-Entity-Group-Transactions
[Tutorial de clúster Redis]: https://redis.io/topics/cluster-tutorial
[Running Redis on a CentOS Linux VM in Windows Azure]: https://blogs.msdn.microsoft.com/tconte/2012/06/08/running-redis-on-a-centos-linux-vm-in-windows-azure/
[Scaling using the Elastic Database split-merge tool]: /azure/sql-database/sql-database-elastic-scale-overview-split-and-merge
[Using Azure Content Delivery Network]: /azure/cdn/cdn-create-new-endpoint
[Cuotas de Service Bus]: /azure/service-bus-messaging/service-bus-quotas
[service-fabric-reliable-collections]: /azure/service-fabric/service-fabric-reliable-services-reliable-collections
[Límites de servicio en Azure Search]:  /azure/search/search-limits-quotas-capacity
[Sharding pattern]: ../patterns/sharding.md
[Tipos de datos admitidos (Azure Search)]:  https://msdn.microsoft.com/library/azure/dn798938.aspx
[Transactions]: https://redis.io/topics/transactions
[¿Qué es Event Hubs?]: /azure/event-hubs/event-hubs-what-is-event-hubs
[¿Qué es Azure Search?]: /azure/search/search-what-is-azure-search
[What is Azure SQL Database?]: /azure/sql-database/sql-database-technical-overview

[Objetivos de escalabilidad]: /azure/storage/common/storage-scalability-targets