---
title: Guía de diseño de una API
description: Guía sobre cómo crear una API web bien diseñada.
author: dragon119
ms.date: 01/12/2018
pnp.series.title: Best Practices
ms.openlocfilehash: 1bd53a7ccc54d086978891f1df5fdc2e25a5d638
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/28/2018
ms.locfileid: "47429390"
---
# <a name="api-design"></a>Diseño de API

Las mayoría de las aplicaciones web modernas exponen interfaces de programación de aplicaciones (API) que los clientes pueden usar para interactuar con la aplicación. Una API web bien diseñada debe ser capaz de admitir:

* **Independencia de la plataforma**. Cualquier cliente debe poder llamar a la API, con independencia de cómo esté implementada internamente. Para ello, es necesario usar protocolos estándar y contar con un mecanismo por medio del cual el cliente y el servicio web puedan acordar el formato de los datos que se intercambian.

* **Evolución del servicio**. La API web debe poder evolucionar y agregar funcionalidad con independencia de las aplicaciones cliente. A medida que evoluciona la API, las aplicaciones cliente existentes deben seguir funcionando sin necesidad de modificarlas. Toda la funcionalidad debe poderse detectar, así las aplicaciones cliente puedan usarla completamente.

En esta guía se describen los problemas que se deben tener en cuenta al diseñar una API web.

## <a name="introduction-to-rest"></a>Introducción a REST

En 2000, Roy Fielding propuso la transferencia de estado representacional (REST) como enfoque de arquitectura para el diseño de servicios web. REST es un estilo de arquitectura para la creación de sistemas distribuidos basados en hipermedia. REST es independiente de cualquier protocolo subyacente y no está necesariamente unido a HTTP. Sin embargo, en las implementaciones más comunes de REST se usa HTTP como protocolo de aplicación, y esta guía se centra en el diseño de API de REST para HTTP.

Una de las principales ventajas de REST sobre HTTP es que usa estándares abiertos y no vincula la implementación de la API o de las aplicaciones cliente con ninguna implementación específica. Por ejemplo, se podría escribir un servicio web de REST en ASP.NET, y las aplicaciones cliente pueden usar cualquier lenguaje o conjunto de herramientas que puedan generar solicitudes HTTP y analizar respuestas HTTP.

Estos son algunos de los principios de diseño más importantes de las API de RESTful mediante HTTP:

- Las API de REST se diseñan en torno a *recursos*, que son cualquier tipo de objeto, dato o servicio al que puede acceder el cliente. 

- Un recurso tiene un *identificador*, que es un URI que identifica de forma única ese recurso. Por ejemplo, el URI de un pedido de cliente en particular podría ser: 
 
    ```http
    https://adventure-works.com/orders/1
    ```
 
- Los clientes interactúan con un servicio mediante el intercambio de *representaciones* de recursos. Muchas API web usan JSON como formato de intercambio. Por ejemplo, una solicitud GET al URI mencionado anteriormente podría devolver este cuerpo de respuesta:

    ```json
    {"orderId":1,"orderValue":99.90,"productId":1,"quantity":1}
    ```

- Las API de REST usan una interfaz uniforme, que ayuda a desacoplar las implementaciones de clientes y servicios. En las API REST basadas en HTTP, la interfaz uniforme incluye el uso de verbos HTTP estándar para realizar operaciones en los recursos. Las operaciones más comunes son GET, POST, PUT, PATCH y DELETE. 

- Las API de REST usan un modelo de solicitud sin estado. Las solicitudes HTTP deben ser independientes y pueden producirse en cualquier orden, por lo que no es factible conservar la información de estado transitoria entre solicitudes. El único lugar donde se almacena la información es en los propios recursos y cada solicitud debe ser una operación atómica. Esta restricción permite que los servicios web sean muy escalables, porque no es necesario conservar ninguna afinidad entre clientes y servidores específicos. Cualquier servidor puede administrar cualquier solicitud de cualquier cliente. Ahora bien, otros factores pueden limitar la escalabilidad. Por ejemplo, muchos servicios web escriben en un almacén de datos back-end, que puede ser difícil de escalar horizontalmente. (En el artículo [Creación de particiones de datos](./data-partitioning.md) se describen las estrategias para escalar horizontalmente un almacén de datos).

- Las API de REST se controlan mediante vínculos de hipermedia contenidos en la representación. Por ejemplo, a continuación se muestra una representación JSON de un pedido. Contiene vínculos para obtener o actualizar el cliente asociado con el pedido. 
 
    ```json
    {
        "orderID":3,
        "productID":2,
        "quantity":4,
        "orderValue":16.60,
        "links": [
            {"rel":"product","href":"https://adventure-works.com/customers/3", "action":"GET" },
            {"rel":"product","href":"https://adventure-works.com/customers/3", "action":"PUT" } 
        ]
    } 
    ```


En 2008, Leonard Richardson propuso el siguiente [modelo de madurez](https://martinfowler.com/articles/richardsonMaturityModel.html) para las API web:

- Nivel 0: definir un URI, y todas las operaciones son solicitudes POST a este URI.
- Nivel 1: crear distintos URI para recursos individuales.
- Nivel 2: usar métodos HTTP para definir operaciones en los recursos.
- Nivel 3: usar hipermedia (HATEOAS, se describe a continuación).

El nivel 3 corresponde a una API verdaderamente RESTful. de acuerdo con la definición del Fielding. En la práctica, muchas API web publicadas se sitúan más bien alrededor del nivel 2.  

## <a name="organize-the-api-around-resources"></a>Organización de la API en torno a los recursos

Se centran en las entidades empresariales que expone la API web. Por ejemplo, en un sistema de comercio electrónico, las entidades principales podrían ser clientes y pedidos. La creación de un pedido se puede lograr mediante el envío de una solicitud HTTP POST que contiene la información del pedido. La respuesta HTTP indica si el pedido se realizó correctamente o no. Siempre que sea posible, los URI de recursos deben basarse en nombres (el recurso) y no en verbos (las operaciones en el recurso). 

```HTTP
https://adventure-works.com/orders // Good

https://adventure-works.com/create-order // Avoid
```

Además, un recurso no tiene que estar basado en un solo elemento de datos físicos. Por ejemplo, un recurso de pedido podría implementarse internamente en forma de varias tablas de una base de datos relacional, pero presentarse al cliente como una única entidad. Evite la creación de API que simplemente reflejen la estructura interna de una base de datos. La finalidad de REST es modelar entidades y las operaciones que una aplicación puede realizar sobre esas entidades. Un cliente no debe exponerse a la implementación interna.

Las entidades a menudo se agrupan en colecciones (pedidos, clientes). Una colección es un recurso independiente del elemento de la colección y debe tener su propio URI. Por ejemplo, el siguiente URI podría representar la colección de pedidos: 

```HTTP
https://adventure-works.com/orders
```

Al enviar una solicitud HTTP GET al URI de la colección, se recupera una lista de elementos de la colección. Cada elemento de la colección también tiene su propio URI único. Una solicitud HTTP GET al URI del elemento devuelve los detalles de ese elemento. 

Adopte una convención de nomenclatura coherente para los URI. En general, resulta útil usar nombres plurales que hagan referencia a colecciones. Es recomendable organizar los URI de colecciones y elementos en una jerarquía. Por ejemplo, `/customers` es la ruta de acceso a la colección de clientes y `/customers/5` es la ruta de acceso al cliente con el identificador igual a 5. Este enfoque ayuda a mantener el carácter intuitivo de la API web. Además, muchas plataformas de API web pueden enrutar solicitudes basadas en rutas de acceso a URI con parámetros, así que podría definir una ruta para la ruta de acceso `/customers/{id}`.

Considere también las relaciones entre los diferentes tipos de recursos y cómo podría exponer estas asociaciones. Por ejemplo, `/customers/5/orders` podría representar todos los pedidos del cliente 5. También puede ir en la otra dirección y representar la asociación de un pedido con un cliente con un URI como `/orders/99/customer`. Sin embargo, este modelo llevado demasiado lejos puede ser difícil de implementar. Una solución mejor es proporcionar vínculos navegables a los recursos asociados en el cuerpo del mensaje de respuesta HTTP. Este mecanismo se describe con más detalle en la sección [Uso del enfoque HATEOAS para permitir la navegación a los recursos relacionados](#using-the-hateoas-approach-to-enable-navigation-to-related-resources) más adelante.

En sistemas más complejos, puede resultar tentador proporcionar identificadores URI que permitan que un cliente navegue por varios niveles de relaciones, como `/customers/1/orders/99/products`. Sin embargo, este nivel de complejidad puede ser difícil de mantener y es inflexible si las relaciones entre los recursos cambian en el futuro. En su lugar, intente que los identificadores URI sean relativamente sencillos. Una vez que una aplicación tiene una referencia a un recurso, debería ser posible usar esta referencia para buscar los elementos relacionados con ese recurso. La consulta anterior se puede reemplazar por el URI `/customers/1/orders` para buscar todos los pedidos del cliente 1 y, a continuación, por `/orders/99/products` para buscar los productos de este pedido.

> [!TIP]
> Evite el uso de URI de recursos más complejos que *collection/item/collection*.

Otro factor es que todas las solicitudes web imponen una carga en el servidor web. Cuantas más solicitudes, más grande la carga. Por lo tanto, intente evitar las API web "locuaces" que exponen un gran número de recursos pequeños. Una API de este tipo puede requerir que una aplicación cliente envíe varias solicitudes para encontrar todos los datos que necesita. En su lugar, puede que desee desnormalizar los datos y combinar la información relacionada en recursos más grandes que se puedan recuperar con una única solicitud. Sin embargo, debe equilibrar este enfoque con la sobrecarga de la captura de datos que el cliente no necesita. Recuperar objetos grandes puede aumentar la latencia de una solicitud y generar costos de ancho de banda adicionales. Para más información sobre estos antipatrones de rendimiento, consulte [Antipatrón Chatty I/O](../antipatterns/chatty-io/index.md) y [Extraneous Fetching](../antipatterns/extraneous-fetching/index.md).

Evite la introducción de dependencias entre la API web y los orígenes de datos subyacentes. Por ejemplo, si los datos están almacenados en una base de datos relacional, la API web no necesita exponer cada una de las tablas como una colección de recursos. De hecho, ese sea probablemente un mal diseño. En su lugar, considere la API web como una abstracción de la base de datos. Si es necesario, introduzca una capa de asignación entre la base de datos y la API web. De este modo, las aplicaciones cliente se aíslan de los cambios en el esquema de base de datos subyacente.

Por último, no sería posible asignar cada operación que implementa una API web a un recurso concreto. Tales escenarios *sin recursos* se pueden administrar mediante solicitudes HTTP que invocan una función y devuelven los resultados en forma de un mensaje de respuesta HTTP. Por ejemplo, una API web que implementa operaciones sencillas de tipo calculadora como sumar y restar podría proporcionar identificadores URI que expongan estas operaciones como seudorecursos y usar la cadena de consulta para especificar los parámetros necesarios. Por ejemplo, una solicitud GET al URI */add?operand1=99&operand2=1* devolvería un mensaje de respuesta donde el cuerpo contiene el valor 100. Sin embargo, únicamente use estas formas de URI con moderación.

## <a name="define-operations-in-terms-of-http-methods"></a>Definición de operaciones en términos de métodos HTTP

El protocolo HTTP define una serie de métodos que asignan significado semántico a una solicitud. Los métodos HTTP comunes que usan la mayoría de las API web RESTful son:

* **GET** recupera una representación del recurso en el URI especificado. El cuerpo del mensaje de respuesta contiene los detalles del recurso solicitado.
* **POST** crea un nuevo recurso en el URI especificado. El cuerpo del mensaje de solicitud proporciona los detalles del nuevo recurso. Tenga en cuenta que POST también puede usarse para desencadenar operaciones que en realidad no crean recursos.
* **PUT** crea o sustituye el recurso en el URI especificado. El cuerpo del mensaje de solicitud especifica el recurso que se va a crear o actualizar.
* **PATCH** realiza una actualización parcial de un recurso. El cuerpo de la solicitud especifica el conjunto de cambios que se aplican al recurso.
* **DELETE** quita el recurso en el URI especificado.

El efecto de una solicitud específica debería depender de si el recurso es una colección o un elemento individual. En la tabla siguiente se resumen las convenciones comunes adoptadas por la mayoría de las implementaciones de RESTful usando el ejemplo de comercio electrónico. Tenga en cuenta que no todas estas solicitudes se pueden implementar; depende de cada situación específica.

| **Recurso** | **POST** | **GET** | **PUT** | **DELETE** |
| --- | --- | --- | --- | --- |
| /customers |Crear un nuevo cliente |Recuperar todos los clientes |Actualización masiva de clientes |Eliminar todos los clientes |
| /customers/1 |Error |Recuperar los detalles del cliente 1 |Actualizar los detalles del cliente 1 si existe |Quitar al cliente 1 |
| /customers/1/orders |Crear un nuevo pedido para el cliente 1 |Recuperar todos los pedidos del cliente 1 |Actualización masiva de pedidos del cliente 1 |Recuperar todos los pedidos del cliente 1 |

Las diferencias entre POST, PUT y PATCH pueden resultar confusas.

- Una solicitud POST crea un recurso. El servidor le asigna un URI y devuelve ese URI al cliente. En el modelo REST, con frecuencia se aplican solicitudes POST a colecciones. El nuevo recurso se agrega a la colección. Una solicitud POST se puede usar también para enviar datos a un recurso existente para su procesamiento, sin que se cree ningún nuevo recurso.

- Una solicitud PUT crea un recurso *o* actualiza un recurso existente. El cliente especifica el URI del recurso. El cuerpo de la solicitud contiene una representación completa del recurso. Si ya existe un recurso con este URI, se reemplaza. De lo contrario se crea un nuevo recurso, si el servidor así lo admite. Las solicitudes PUT se aplican con más frecuencia a recursos que son elementos individuales, por ejemplo, un cliente especifico, en lugar de colecciones. Un servidor puede admitir actualizaciones, pero no la creación mediante PUT, si ello depende de que el cliente pueda asignar un URI de manera significativa a un recurso antes de que exista. Si este no es el caso, use POST para crear recursos y PUT o PATCH para actualizarlos.

- Una solicitud PATCH realiza una *actualización parcial* de un recurso existente. El cliente especifica el URI del recurso. El cuerpo de la solicitud especifica un conjunto de *cambios* para aplicar al recurso. Este enfoque puede ser más eficaz que el uso de PUT, dado que el cliente solo envía los cambios, no la representación completa del recurso. Técnicamente PATCH puede crear también un nuevo recurso (mediante la especificación de un conjunto de actualizaciones en un recurso "nulo"), si el servidor así lo admite. 

Las solicitudes PUT deben ser idempotentes. Si un cliente envía la misma solicitud PUT varias veces, los resultados siempre deben ser los mismos (el mismo recurso se modificará con los mismos valores). No se garantiza que las solicitudes POST y PATCH sean idempotentes.

## <a name="conform-to-http-semantics"></a>Conformidad con la semántica HTTP

En esta sección se describen algunas consideraciones habituales para diseñar una API que guarde conformidad con la especificación HTTP. Sin embargo, no se tratan todos los posibles detalles o escenarios. En caso de duda, consulte las especificaciones HTTP.

### <a name="media-types"></a>Tipos de medios

Como se mencionó anteriormente, los clientes y servidores intercambian representaciones de recursos. Por ejemplo, en una solicitud POST, el cuerpo de la solicitud contiene una representación del recurso que se va a crear. En una solicitud GET, el cuerpo de respuesta contiene una representación del recurso capturado.

En el protocolo HTTP, los formatos se especifican mediante el uso de *tipos de medios*, también denominados tipos MIME. En el caso de datos no binarios, la mayoría de las API web admiten JSON (tipo de medio = application/json) y posiblemente XML (tipo de medio = application/xml). 

El encabezado Content-Type en una solicitud o respuesta especifica el formato de la representación. Este es un ejemplo de una solicitud POST que incluye datos JSON:

```HTTP
POST https://adventure-works.com/orders HTTP/1.1
Content-Type: application/json; charset=utf-8
Content-Length: 57

{"Id":1,"Name":"Gizmo","Category":"Widgets","Price":1.99}
```

Si el servidor no admite el tipo de medio, debe devolver el código de estado HTTP 415 (tipo de medio no compatible).

Una solicitud de cliente puede incluir un encabezado Accept que contenga una lista de tipos de medios que el cliente aceptará del servidor en el mensaje de respuesta. Por ejemplo: 

```HTTP
GET https://adventure-works.com/orders/2 HTTP/1.1
Accept: application/json
```

Si el servidor no puede encontrar la correspondencia con ninguno de los tipos de medios enumerados, debe devolver el código de estado HTTP 406 (No aceptable). 

### <a name="get-methods"></a>Métodos GET

Normalmente, un método GET correcto devuelve el código de estado HTTP 200 (Correcto). Si no se encuentra el recurso, el método debe devolver 404 (No encontrado).

### <a name="post-methods"></a>Métodos POST

Si un método POST crea un nuevo recurso, devuelve el código de estado HTTP 201 (Creado). El URI del nuevo recurso se incluye en el encabezado Location de la respuesta. El cuerpo de respuesta contiene una representación del recurso.

Si el método realiza algún procesamiento pero no crea un nuevo recurso, puede devolver el código de estado HTTP 200 e incluir el resultado de la operación en el cuerpo de respuesta. O bien, si no hay ningún resultado para devolver, el método puede devolver el código de estado HTTP 204 (Sin contenido) sin cuerpo de respuesta.

Si el cliente coloca datos no válidos en la solicitud, el servidor debe devolver el código de estado HTTP 400 (Solicitud incorrecta). El cuerpo de respuesta puede contener información adicional sobre el error o un vínculo a un URI que proporciona más detalles.

### <a name="put-methods"></a>Métodos PUT

Si un método PUT crea un nuevo recurso, devuelve el código de estado HTTP 201 (Creado), al igual que con un método POST. Si el método actualiza un recurso existente, devuelve 200 (Correcto) o 204 (Sin contenido). En algunos casos, no será posible actualizar un recurso existente. En ese caso, considere la posibilidad de devolver el código de estado HTTP 409 (Conflicto). 

Considere la posibilidad de implementar operaciones HTTP PUT masivas que pueden procesar por lotes las actualizaciones de varios recursos de una colección. La solicitud PUT debe especificar el URI de la colección y el cuerpo de solicitud debe especificar los detalles de los recursos que se van a modificar. Este enfoque puede ayudar a reducir el intercambio de mensajes y mejorar el rendimiento.

### <a name="patch-methods"></a>Métodos PATCH

Con una solicitud PATCH, el cliente envía un conjunto de actualizaciones a un recurso existente, en forma de un *documento de revisión*. El servidor procesa el documento de revisión para realizar la actualización. El documento de revisión no describe el recurso completo, solo un conjunto de cambios para aplicar. La especificación del método PATCH ([RFC 5789](https://tools.ietf.org/html/rfc5789)) no define un formato determinado de documentos de revisión. El formato debe deducirse del tipo de medio de la solicitud.

JSON es probablemente el formato de datos más común para las API web. Hay dos formatos principales de revisión basados en JSON, que se conocen como *revisión JSON* y *revisión de combinación JSON*.

La revisión de combinación JSON es algo más simple. El documento de revisión tiene la misma estructura que el recurso original JSON, pero incluye solo el subconjunto de campos que se deben cambiar o agregar. Además, se puede eliminar un campo mediante la especificación de `null` como valor del campo en el documento de revisión. (Eso significa que la revisión de combinación no es adecuada si el recurso original puede tener valores null explícitos).

Por ejemplo, suponga que el recurso original tiene la representación JSON siguiente:

```json
{ 
    "name":"gizmo",
    "category":"widgets",
    "color":"blue",
    "price":10
}
```

Esta es una posible revisión de combinación JSON para este recurso:

```json
{ 
    "price":12,
    "color":null,
    "size":"small"
}
```

Esta revisión indica al servidor que actualice el valor de "price", elimine el valor de "color" y agregue un valor para "size". "Name" y "category" no se modifican. Para conocer los detalles exactos de la revisión de combinación JSON, consulte [RFC 7396](https://tools.ietf.org/html/rfc7396). El tipo de medio para la revisión de combinación JSON es "application/merge-patch + json".

La revisión de combinación no es adecuada si el recurso original puede contener valores null explícitos, debido al significado especial de `null` en el documento de revisión. Además, el documento de revisión no especifica el orden en el que el servidor debe aplicar las actualizaciones. Eso puede ser o no importante, según los datos y el dominio. La revisión JSON, definida en [RFC 6902](https://tools.ietf.org/html/rfc6902), es más flexible. Los cambios se especifican como una secuencia de operaciones que se aplicarán. Las operaciones incluyen agregar, quitar, reemplazar, copiar y probar (para validar los valores). El tipo de medio para la revisión JSON es "application/json-patch + json".

Estas son algunas condiciones de error habituales que podrían producirse al procesar una solicitud PATCH, junto con el código de estado HTTP adecuado.

| Condición de error | Código de estado HTTP |
|-----------|------------|
| No se admite el formato de documento de revisión. | 415 (Tipo de medio no compatible) |
| El documento de revisión tiene un formato incorrecto. | 400 (Solicitud incorrecta) |
| El documento de revisión es válido, pero los cambios no se pueden aplicar al recurso en su estado actual. | 409 (Conflicto)

### <a name="delete-methods"></a>Métodos DELETE

Si la operación de eliminación es correcta, el servidor web debe responder con el código de estado HTTP 204, que indica que el proceso se ha administrado correctamente, pero que el cuerpo de respuesta no contiene información adicional. Si el recurso no existe, el servidor web puede devolver HTTP 404 (No encontrado).

### <a name="asynchronous-operations"></a>Operaciones asincrónicas

En ocasiones, una operación POST, PUT, PATCH o DELETE podría requerir procesamiento que tarda algún tiempo en completarse. Si espera a que finalice antes de enviar una respuesta al cliente, podría producirse una latencia inaceptable. En este caso, considere la posibilidad de realizar la operación asincrónica. Se devuelve el código de estado HTTP 202 (Aceptado) para indicar que se aceptó la solicitud para el procesamiento, pero no se ha completado. 

Debe exponer un punto de conexión que devuelva el estado de una solicitud asincrónica, así el cliente puede supervisar el estado mediante el sondeo del punto de conexión de estado. Incluya el URI del punto de conexión de estado en el encabezado Location de la respuesta 202. Por ejemplo: 

```http
HTTP/1.1 202 Accepted
Location: /api/status/12345
```

Si el cliente envía una solicitud GET a este punto de conexión, la respuesta debe contener el estado actual de la solicitud. Opcionalmente, también puede incluir un tiempo de finalización estimado o un vínculo para cancelar la operación. 

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
    "status":"In progress",
    "link": { "rel":"cancel", "method":"delete", "href":"/api/status/12345" }
}
```

Si la operación asincrónica crea un nuevo recurso, el punto de conexión de estado debe devolver el código de estado 303 (ver Otros) una vez completada la operación. En la respuesta 303, se incluye un encabezado Location que proporciona el URI del nuevo recurso:

```http
HTTP/1.1 303 See Other
Location: /api/orders/12345
```

Para más información, consulte [Asynchronous operations in REST](https://www.adayinthelifeof.nl/2011/06/02/asynchronous-operations-in-rest/) (Operaciones asincrónicas en REST).

## <a name="filter-and-paginate-data"></a>Filtrado y paginación de los datos

Exponer una colección de recursos con un único URI puede dar lugar a que las aplicaciones capturen grandes cantidades de datos cuando solo se requiere un subconjunto de la información. Por ejemplo, suponga que una aplicación cliente necesita encontrar todos los pedidos con un costo cercano a un valor específico. Se podrían recuperar todos los pedidos del URI */orders* y, a continuación, filtrar estos pedidos en el lado cliente. Es obvio que este proceso es muy ineficaz. Se malgasta ancho de banda de red y potencia de procesamiento en el servidor que hospeda la API web.

En su lugar, la API puede permitir que pase un filtro en la cadena de consulta del URI, como */orders?minCost=n*. La API web es responsable entonces de analizar y administrar el parámetro `minCost` en la cadena de consulta y de devolver los resultados filtrados en el lado servidor. 

Las solicitudes GET sobre recursos de la colección pueden devolver posiblemente un gran número de elementos. Debe diseñar una API que limite la cantidad de datos devueltos en una única solicitud. Considere la posibilidad de admitir cadenas de consulta que especifiquen el número máximo de elementos que se van a recuperar y un desplazamiento inicial en la colección. Por ejemplo: 

```
/orders?limit=25&offset=50
```

Considere también la posibilidad de imponer un límite superior sobre el número de elementos devueltos, para ayudar a evitar ataques por denegación de servicio. Para ayudar a las aplicaciones cliente, las solicitudes GET que devuelven datos paginados deben incluir también alguna forma de metadatos que indiquen el número total de recursos disponibles en la colección. 

Puede seguir una estrategia parecida para ordenar los datos que se capturan y proporcionar un parámetro de clasificación que tome un nombre de campo como valor, por ejemplo */orders?sort=ProductID*. Sin embargo, este enfoque puede tener un efecto negativo sobre el almacenamiento en caché, dado que los parámetros de la cadena de consulta forman parte del identificador de recursos que se usa en muchas implementaciones de caché como llave a los datos en caché.

Puede ampliar este enfoque para limitar los campos devueltos para cada elemento, si cada elemento contiene una gran cantidad de datos. Por ejemplo, podría usar un parámetro de cadena de consulta que acepte una lista de campos delimitados por coma, como */orders?fields=ProductID,Quantity*. 

Asigne a todos los parámetros opcionales de las cadenas de consulta valores predeterminados. Por ejemplo, establezca el parámetro `limit` en 10 y el parámetro `offset` en 0 si implementa paginación, establezca el parámetro de ordenación en la clave del recurso si implementa ordenación y establezca el parámetro `fields` en todos los campos del recurso si admite proyecciones.

## <a name="support-partial-responses-for-large-binary-resources"></a>Compatibilidad con respuestas parciales en recursos binarios de gran tamaño

Un recurso puede contener campos binarios de gran tamaño, como imágenes o archivos. Para solucionar los problemas ocasionados por conexiones intermitentes y poco confiables y para mejorar los tiempos de respuesta, considere la posibilidad de permitir que estos recursos se recuperen en fragmentos. Para ello, la API web debe admitir el encabezado Accept-Ranges en solicitudes GET de recursos de gran tamaño. Este encabezado indica que la operación GET admite solicitudes parciales. La aplicación cliente puede enviar solicitudes GET que devuelven un subconjunto de un recurso, especificado como un intervalo de bytes. 

Además, considere la posibilidad de implementar solicitudes HTTP HEAD para estos recursos. Una solicitud HEAD es similar a una solicitud GET, excepto que solo devuelve los encabezados HTTP que describen el recurso, con un cuerpo de mensaje vacío. Una aplicación cliente puede emitir una solicitud HEAD para determinar si se debe capturar un recurso mediante solicitudes GET parciales. Por ejemplo: 

```HTTP
HEAD https://adventure-works.com/products/10?fields=productImage HTTP/1.1
```

Este es un ejemplo de un mensaje de respuesta: 

```HTTP
HTTP/1.1 200 OK

Accept-Ranges: bytes
Content-Type: image/jpeg
Content-Length: 4580
```

El encabezado Content-Length proporciona el tamaño total del recurso, y el encabezado Accept-Ranges indica que la operación GET correspondiente admite resultados parciales. La aplicación cliente puede usar esta información para recuperar la imagen en fragmentos más pequeños. La primera solicitud captura los primeros 2500 bytes mediante el encabezado Range:

```HTTP
GET https://adventure-works.com/products/10?fields=productImage HTTP/1.1
Range: bytes=0-2499
```

El mensaje de respuesta indica que es una respuesta parcial al devolver el código de estado HTTP 206. El encabezado Content-Length especifica el número real de bytes devuelto en el cuerpo del mensaje (no el tamaño del recurso) y el encabezado Content-Range indica qué parte del recurso es (bytes 0-2499 de 4580):

```HTTP
HTTP/1.1 206 Partial Content

Accept-Ranges: bytes
Content-Type: image/jpeg
Content-Length: 2500
Content-Range: bytes 0-2499/4580

[...]
```

Una solicitud posterior desde la aplicación cliente puede recuperar el resto del recurso.

## <a name="use-hateoas-to-enable-navigation-to-related-resources"></a>Uso de HATEOAS para permitir la navegación a los recursos relacionados

Uno de los principales propósitos que se esconden detrás de REST es que debe ser posible navegar por todo el conjunto de recursos sin necesidad de conocer el esquema de URI. Cada solicitud HTTP GET debe devolver la información necesaria para encontrar los recursos relacionados directamente con el objeto solicitado mediante los hipervínculos que se incluyen en la respuesta, y también se le debe proporcionar información que describa las operaciones disponibles en cada uno de estos recursos. Este principio se conoce como HATEOAS, del inglés Hypertext as the Engine of Application State (Hipertexto como motor del estado de la aplicación). El sistema es realmente una máquina de estado finito, y la respuesta a cada solicitud contiene la información necesaria para pasar de un estado a otro; ninguna otra información debería ser necesaria.

> [!NOTE]
> Actualmente no hay ninguna norma o especificación que define cómo modelar el principio HATEOAS. Los ejemplos mostrados en esta sección muestran una posible solución.
>
>

Por ejemplo, para administrar la relación entre un pedido y un cliente, la representación de un pedido podría incluir vínculos que identifican las operaciones disponibles para el cliente del pedido. Esta sería una posible representación: 

```json
{
  "orderID":3,
  "productID":2,
  "quantity":4,
  "orderValue":16.60,
  "links":[
    {
      "rel":"customer",
      "href":"https://adventure-works.com/customers/3", 
      "action":"GET",
      "types":["text/xml","application/json"] 
    },
    {
      "rel":"customer",
      "href":"https://adventure-works.com/customers/3", 
      "action":"PUT",
      "types":["application/x-www-form-urlencoded"]
    },
    {
      "rel":"customer",
      "href":"https://adventure-works.com/customers/3",
      "action":"DELETE",
      "types":[]
    },
    {
      "rel":"self",
      "href":"https://adventure-works.com/orders/3", 
      "action":"GET",
      "types":["text/xml","application/json"]
    },
    {
      "rel":"self",
      "href":"https://adventure-works.com/orders/3", 
      "action":"PUT",
      "types":["application/x-www-form-urlencoded"]
    },
    {
      "rel":"self",
      "href":"https://adventure-works.com/orders/3", 
      "action":"DELETE",
      "types":[]
    }]
}
```

En este ejemplo, la matriz `links` tiene un conjunto de vínculos. Cada vínculo representa una operación en una entidad relacionada. Los datos de cada vínculo incluyen la relación ("customer"), el URI (`https://adventure-works.com/customers/3`), el método HTTP y el tipo MIME admitido. Esta es toda la información que necesita una aplicación cliente para invocar la operación. 

La matriz `links` también incluye información con una referencia a sí misma sobre el recurso propiamente dicho que se ha recuperado. Este tiene la relación *self*.

El conjunto de vínculos que se devuelve puede cambiar, según el estado del recurso. A esto nos referimos cuando decimos que el hipertexto es el "motor de estado de aplicación".

## <a name="versioning-a-restful-web-api"></a>Control de versiones de una API web RESTful

Es muy poco probable que una API web permanezca estática. Conforme los requisitos empresariales cambian, se pueden agregar nuevas colecciones de recursos, las relaciones entre los recursos pueden cambiar y la estructura de los datos de los recursos puede rectificarse. Si bien la actualización de una API para controlar los requisitos nuevos o diferentes es un proceso relativamente sencillo, debe tener en cuenta los efectos que tendrán dichos cambios en las aplicaciones cliente que utilizan la API web. El problema es que, aunque el desarrollador que diseña e implementa una API web tiene control total sobre dicha API, el desarrollador carece del mismo grado de control sobre las aplicaciones cliente que podrían crear organizaciones de terceros que operan de forma remota. La premisa principal es permitir que las aplicaciones cliente existentes sigan funcionando sin cambios y al mismo tiempo dejar que las nuevas aplicaciones cliente aprovechen las ventajas de nuevas características y recursos.

El control de versiones permite que una API web indique las características y recursos que expone y que una aplicación cliente pueda enviar solicitudes que se dirijan a una versión específica de una característica o un recurso. En las secciones siguientes se describen varios enfoques diferentes, cada uno de los cuales tiene sus propias ventajas y desventajas.

### <a name="no-versioning"></a>Sin control de versiones
Este es el enfoque más sencillo y puede ser aceptable para algunas API internas. Los grandes cambios podrían representarse como nuevos recursos o nuevos vínculos.  Agregar contenido a recursos existentes puede que no represente un cambio importante dado que las aplicaciones cliente que no esperan ver este contenido simplemente lo ignorarán.

Por ejemplo, una solicitud al URI *https://adventure-works.com/customers/3* debe devolver los detalles de un solo cliente que contiene los campos `id`, `name` y `address` esperados por la aplicación cliente:

```HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"id":3,"name":"Contoso LLC","address":"1 Microsoft Way Redmond WA 98053"}
```

> [!NOTE]
> Por motivos de simplicidad, las respuestas de ejemplo que se muestran en esta sección no incluyen vínculos HATEOAS.
>
>

Si el campo `DateCreated` se agrega al esquema del recurso de cliente, la respuesta tendría el siguiente aspecto:

```HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":"1 Microsoft Way Redmond WA 98053"}
```

Las aplicaciones cliente existentes pueden seguir funcionando correctamente si son capaces de omitir los campos no reconocidos, pero las nuevas aplicaciones cliente se pueden diseñar para controlar este nuevo campo. Sin embargo, si se producen cambios más radicales en el esquema de recursos (por ejemplo, se quitan campos y se cambian de nombre) o cambian las relaciones entre los recursos, dichos cambios podrían constituir cambios importantes que impiden que las aplicaciones cliente existentes funcionen correctamente. En estas situaciones es aconsejable uno de los enfoques siguientes.

### <a name="uri-versioning"></a>Control de versiones de URI
Cada vez que modifica la API web o cambia el esquema de recursos, agrega un número de versión al URI para cada recurso. Los URI ya existentes deben seguir funcionando como antes y devolver los recursos conforme a su esquema original.

Ampliando el ejemplo anterior, si el campo `address` se reestructura en campos secundarios que contienen cada uno una parte constituyente de la dirección (como `streetAddress`, `city`, `state` y `zipCode`), esta versión del recurso podría exponerse a través de un URI que contenga un número de versión, como https://adventure-works.com/v2/customers/3:

```HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":{"streetAddress":"1 Microsoft Way","city":"Redmond","state":"WA","zipCode":98053}}
```

Este mecanismo de control de versiones es muy sencillo, pero depende del servidor que enruta la solicitud al extremo adecuado. Sin embargo, puede volverse difícil de manejar dado que la API web madura a través de varias iteraciones y el servidor tiene que admitir un número de versiones diferentes. Además, desde el punto de vista de los más puristas, en todos los casos las aplicaciones cliente capturan los mismos datos (cliente 3), así que el URI no debería ser realmente diferente según la versión. Este esquema también complica la implementación de HATEOAS ya que todos los vínculos deberán incluir el número de versión en sus URI.

### <a name="query-string-versioning"></a>Control de versiones de cadena de consulta
En lugar de proporcionar varios URI, puede especificar la versión del recurso mediante un parámetro dentro de la cadena de consulta anexada a la solicitud HTTP, como *https://adventure-works.com/customers/3?version=2*. El parámetro de versión debe adoptar de forma predeterminada un valor significativo como 1 si se omite en las aplicaciones cliente más antiguas.

Este enfoque tiene la ventaja semántica de que el mismo recurso siempre se recupera del mismo URI, pero depende del código que controla la solicitud para analizar la cadena de consulta y enviar la respuesta HTTP adecuada. Este método también presenta tiene las mismas complicaciones para implementar HATEOAS que el mecanismo de control de versiones de URI.

> [!NOTE]
> Algunos exploradores web y servidores proxy antiguos no almacenan en caché las respuestas de solicitudes que incluyen una cadena de consulta en el URI. Esto puede tener un impacto negativo en el rendimiento de las aplicaciones web que usan una API web y que se ejecutan en este tipo de explorador web.
>
>

### <a name="header-versioning"></a>Control de versiones de encabezado
En lugar de anexar el número de versión como un parámetro de cadena de consulta, podría implementar un encabezado personalizado que indica la versión del recurso. Este enfoque requiere que la aplicación cliente agregue el encabezado adecuado a las solicitudes, aunque el código que controla la solicitud de cliente puede usar un valor predeterminado (versión 1) si se omite el encabezado de versión. En los ejemplos siguientes se usa un encabezado personalizado denominado *Custom-Header*. El valor de este encabezado indica la versión de la API web.

Versión 1:

```HTTP
GET https://adventure-works.com/customers/3 HTTP/1.1
Custom-Header: api-version=1
```

```HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"id":3,"name":"Contoso LLC","address":"1 Microsoft Way Redmond WA 98053"}
```

Versión 2:

```HTTP
GET https://adventure-works.com/customers/3 HTTP/1.1
Custom-Header: api-version=2
```

```HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":{"streetAddress":"1 Microsoft Way","city":"Redmond","state":"WA","zipCode":98053}}
```

Tenga en cuenta que al igual que en los dos enfoques anteriores, la implementación de HATEOAS requiere que se incluya el encabezado personalizado apropiado en los vínculos.

### <a name="media-type-versioning"></a>Control de versiones del tipo de medio
Cuando una aplicación cliente envía una solicitud HTTP GET a un servidor web, debe prever el formato del contenido que puede controlar mediante el uso de un encabezado Accept, como se ha descrito anteriormente en esta guía. Con frecuencia, el propósito del encabezado *Accept* es permitir que la aplicación cliente especifique si el cuerpo de la respuesta debe ser XML, JSON o algún otro formato común que pueda analizar el cliente. Sin embargo, es posible definir tipos de medios personalizados que incluyan información que permita que la aplicación cliente indique qué versión de un recurso que se espera. En el ejemplo siguiente se muestra una solicitud que especifica un encabezado *Accept* con el valor *application/vnd.adventure-works.v1+json*. El elemento *vnd.adventure-works.v1* indica al servidor web que debe devolver la versión 1 del recurso, mientras que el elemento *json* especifica que el formato del cuerpo de respuesta debe ser JSON:

```HTTP
GET https://adventure-works.com/customers/3 HTTP/1.1
Accept: application/vnd.adventure-works.v1+json
```

El código que controla la solicitud es responsable de procesar el encabezado *Accept* y de respetarlo siempre que sea posible (la aplicación cliente puede especificar varios formatos en el encabezado *Accept*, en cuyo caso el servidor web puede elegir el formato más adecuado para el cuerpo de respuesta). El servidor web confirma el formato de los datos en el cuerpo de respuesta mediante el encabezado Content-Type:

```HTTP
HTTP/1.1 200 OK
Content-Type: application/vnd.adventure-works.v1+json; charset=utf-8

{"id":3,"name":"Contoso LLC","address":"1 Microsoft Way Redmond WA 98053"}
```

Si el encabezado Accept no especifica ningún tipo de medio conocido, el servidor web podría generar un mensaje de respuesta HTTP 406 (no aceptable) o devolver un mensaje con un tipo de medio predeterminado.

Este enfoque es posiblemente el más puro de los mecanismos de control de versiones y se presta de forma natural a HATEOAS, que puede incluir el tipo MIME de los datos relacionados en los vínculos de recursos.

> [!NOTE]
> Al seleccionar una estrategia de control de versiones, también debe considerar las implicaciones en el rendimiento, especialmente en el almacenamiento en caché en el servidor web. Los esquemas de control de versiones de URI y de control de versiones de cadena de consulta son compatibles con la caché puesto que la misma combinación de URI y cadena de consulta hace referencia siempre a los mismos datos.
>
> Los mecanismos de control de versiones de encabezado y de control de versiones de tipo de medio normalmente requieren lógica adicional para examinar los valores del encabezado personalizado o del encabezado Accept. En un entorno a gran escala, muchos clientes que usan versiones diferentes de una API web pueden producir una cantidad significativa de datos duplicados en una caché del servidor. Este problema puede ser agudo si una aplicación cliente se comunica con un servidor web a través de un proxy que implementa almacenamiento en caché y que solo reenvía una solicitud a dicho servidor si no contiene actualmente una copia de los datos solicitados en su caché.
>
>

## <a name="open-api-initiative"></a>Iniciativa Open API
La [iniciativa Open API](https://www.openapis.org/) fue creada por un consorcio de la industria para normalizar las descripciones de las API de REST de los distintos proveedores. Como parte de esta iniciativa, la especificación Swagger 2.0 se cambió a OpenAPI Specification (OAS) y se incluyó en la iniciativa Open API.

Puede adoptar OpenAPI para sus API web. Algunos puntos que se deben tener en cuenta:

- OpenAPI Specification incluye un conjunto de directrices bien fundamentadas acerca de cómo se debe diseñar una API de REST. Esto presenta ventajas en cuanto a interoperabilidad, pero requiere más cuidado al diseñar las API para que cumplan la especificación.
- OpenAPI promueve un enfoque "el contrato es lo primero", en lugar de un enfoque "la implementación es lo primero". Que el contrato sea lo primero, significa que primero se diseña el contrato de la API (la interfaz) y después se escribe el código que implementa el contrato. 
- Herramientas como Swagger pueden generar las bibliotecas de cliente o la documentación de los contratos de API. Por ejemplo, consulte [Páginas de ayuda de ASP.NET Core Web API mediante Swagger](/aspnet/core/tutorials/web-api-help-pages-using-swagger).

## <a name="more-information"></a>Más información
* [Directrices para API de REST de Microsoft](https://github.com/Microsoft/api-guidelines/blob/master/Guidelines.md). Recomendaciones detalladas para diseñar API de REST públicas.
* [Web API Checklist](https://mathieu.fenniak.net/the-api-checklist/) (Lista de comprobación de API web). Una lista útil de elementos que se deben tener en cuenta al diseñar e implementar una API web.
* [Open API Initiative](https://www.openapis.org/). Detalles de la documentación y la implementación en Open API.
