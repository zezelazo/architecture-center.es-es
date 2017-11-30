---
title: "Guía de diseño de una API"
description: "Guía sobre cómo crear una API bien diseñada."
author: dragon119
ms.date: 07/13/2016
pnp.series.title: Best Practices
ms.openlocfilehash: 3ffadce1b0c4a4da808e52d61cff0b7f0b27de11
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2017
---
# <a name="api-design"></a>Diseño de API
[!INCLUDE [header](../_includes/header.md)]

Muchas soluciones modernas basadas en web hacen uso de servicios web, hospedados en servidores web, para proporcionar funcionalidad a las aplicaciones cliente remotas. Las operaciones que expone un servicio web constituyen una API web. Una API web bien diseñada debe ser capaz de admitir:

* **Independencia de la plataforma**. Las aplicaciones cliente deben ser capaces de usar la API que proporciona el servicio web sin necesidad de saber cómo los datos o las operaciones que expone la API se implementan físicamente. Para ello, se requiere que la API se rija por las normas comunes que permiten que una aplicación cliente y un servicio web acuerden qué formatos de datos se usan y la estructura de los datos que se intercambian entre las aplicaciones cliente y el servicio web.
* **Evolución del servicio**. El servicio web debe ser capaz de evolucionar y agregar (o quitar) funcionalidad con independencia de las aplicaciones cliente. Las aplicaciones cliente existentes deben ser capaces de continuar funcionando sin cambios a pesar de que las características que proporcione el servicio web cambien. Toda la funcionalidad debe poderse detectar para que las aplicaciones cliente la puedan usar completamente.

El propósito de esta guía es describir los problemas que debe tener en cuenta al diseñar una API web.

## <a name="introduction-to-representational-state-transfer-rest"></a>Introducción a la transferencia de estado representacional (REST)
En su disertación en 2000, Roy Fielding propuso un enfoque alternativo sobre la arquitectura con el fin de estructurar las operaciones expuestas por los servicios web; REST. REST es un estilo de arquitectura para la creación de sistemas distribuidos basados en hipermedia. Una de las principales ventajas del modelo REST es que se basa en estándares abiertos y no vincula la implementación del modelo o las aplicaciones cliente que tienen acceso a él con ninguna implementación específica. Por ejemplo, un servicio web REST podría implementarse mediante la API web de Microsoft ASP.NET y las aplicaciones cliente podrían desarrollarse usando cualquier lenguaje y conjunto de herramientas que pueda generar solicitudes HTTP y analizar las respuestas HTTP.

> [!NOTE]
> REST es realmente independiente de cualquier protocolo subyacente y no está necesariamente unido a HTTP. Sin embargo, las implementaciones más comunes de los sistemas basados en REST usan HTTP como protocolo de aplicación para enviar y recibir solicitudes. Este documento se centra en la asignación de principios REST a sistemas diseñados para funcionar mediante HTTP.
>
>

El modelo REST usa un esquema de navegación para representar objetos y servicios en una red (denominados *recursos*). Muchos sistemas que implementan REST normalmente usan el protocolo HTTP para transmitir solicitudes de acceso a estos recursos. En estos sistemas, una aplicación cliente envía una solicitud en forma de un URI que identifica un recurso y un método HTTP (el más común GET, POST, PUT o DELETE) que indica la operación que se realizará en ese recurso.  El cuerpo de la solicitud HTTP contiene los datos necesarios para realizar la operación. Es importante que comprenda que REST define un modelo de solicitud sin estado. Las solicitudes HTTP deben ser independientes y pueden producirse en cualquier orden, por lo que no es factible intentar conservar la información de estado transitorio entre solicitudes.  El único lugar donde se almacena la información es en los propios recursos y cada solicitud debe ser una operación atómica. De hecho, un modelo REST implementa una máquina de estado finito en la que una solicitud pasa a un recurso desde un estado no transitorios bien definido a otro.

> [!NOTE]
> La naturaleza sin estado de las solicitudes individuales en el modelo REST permite que un sistema construido siguiendo estos principios sea muy escalable. No hay ninguna necesidad de conservar ninguna afinidad entre una aplicación cliente que realiza una serie de solicitudes y los servidores web específicos que controlan esas solicitudes.
>
>

Otro aspecto muy importante a tener en cuenta a la hora de implementar un modelo REST eficaz es comprender las relaciones entre los distintos recursos a los que el modelo proporciona acceso. Normalmente, estos recursos se organizan en relaciones y colecciones. Por ejemplo, suponga que un análisis rápido de un sistema de comercio electrónico muestra que hay dos colecciones en las que las aplicaciones cliente podrían estar interesadas: pedidos y clientes. Cada pedido y cada cliente deben tener su propia clave única para fines de identificación. El identificador URI de acceso a la colección de pedidos podría ser algo tan sencillo como */orders* e, igualmente, el identificador URI para recuperar todos los clientes podría ser */customers*. Al emitir una solicitud HTTP GET al URI */orders* se debería devolver una lista que representa todos los pedidos de la colección codificados como una respuesta HTTP:

```HTTP
GET http://adventure-works.com/orders HTTP/1.1
...
```

La respuesta que se muestra a continuación codifica los pedidos en una estructura de lista JSON:

```HTTP
HTTP/1.1 200 OK
...
Date: Fri, 22 Aug 2014 08:49:02 GMT
Content-Length: ...
[{"orderId":1,"orderValue":99.90,"productId":1,"quantity":1},{"orderId":2,"orderValue":10.00,"productId":4,"quantity":2},{"orderId":3,"orderValue":16.60,"productId":2,"quantity":4},{"orderId":4,"orderValue":25.90,"productId":3,"quantity":1},{"orderId":5,"orderValue":99.90,"productId":1,"quantity":1}]
```
Para capturar un pedido individual, es necesario especificar el identificador del pedido desde el recurso *orders*, como */orders/2*:

```HTTP
GET http://adventure-works.com/orders/2 HTTP/1.1
...
```

```HTTP
HTTP/1.1 200 OK
...
Date: Fri, 22 Aug 2014 08:49:02 GMT
Content-Length: ...
{"orderId":2,"orderValue":10.00,"productId":4,"quantity":2}
```

> [!NOTE]
> Por motivos de simplicidad, estos ejemplos muestran la información en respuestas que se devuelven como datos de texto JSON. Sin embargo, no hay ningún motivo para que los recursos no deban contener ningún otro tipo de dato compatible con HTTP, como información binaria o cifrada; el tipo de contenido en la respuesta HTTP debe especificar el tipo. Además, un modelo REST puede devolver los mismos datos en diferentes formatos, como XML o JSON. En este caso, el servicio web debe ser capaz de realizar la negociación de contenido con el cliente que realiza la solicitud. La solicitud puede incluir un encabezado *Accept* que especifica el formato preferido que desearía recibir el cliente y el servicio web debería intentar respetar este formato si es posible.
>
>

Observe que la respuesta de una solicitud REST hace uso de códigos de estado HTTP estándar. Por ejemplo, una solicitud que devuelve datos válidos debe incluir el código de respuesta HTTP 200 (correcto), mientras que una solicitud que no encuentra o elimina un recurso especificado debe devolver una respuesta que incluya el código de estado HTTP 404 (no encontrado).

## <a name="design-and-structure-of-a-restful-web-api"></a>Diseño y la estructura de una API web RESTful
Las claves para diseñar una API web correcta son sencillez y coherencia. Una API web que muestra estos dos factores facilita la creación de aplicaciones cliente que necesitan usar la API.

Una API web RESTful se centra en la exposición de un conjunto de recursos conectados y en proporcionar las operaciones básicas que permiten a una aplicación manipular estos recursos y navegar fácilmente entre ellos. Por este motivo, los URI que constituyen una API web RESTful típica deben estar orientados a los datos que dicha API expone y usar las funciones proporcionadas por HTTP para operar con estos datos. Este enfoque requiere una mentalidad diferente de la que normalmente se emplea al diseñar un conjunto de clases en una API orientada a objetos, que tiende a estar más motivada por el comportamiento de objetos y clases. Además, una API web RESTful no debe tener estado y no debe depender de operaciones que se invocan en una secuencia determinada. En las secciones siguientes se resumen los puntos que debe tener en cuenta al diseñar una API web RESTful.

### <a name="organizing-the-web-api-around-resources"></a>Organización de la API web en torno a los recursos
> [!TIP]
> Los URI expuesto por un servicio web REST deben basarse en nombres (los datos a la que la API web proporciona acceso) y no en verbos (lo que una aplicación puede hacer con los datos).
>
>

Se centran en las entidades empresariales que expone la API web. Por ejemplo, en una API diseñada para admitir el sistema de comercio electrónico descrito anteriormente, las entidades principales son los clientes y los pedidos. Los procesos, como el acto de realizar un pedido, se pueden lograr proporcionando una operación HTTP POST que toma la información del pedido y la agrega a la lista de pedidos del cliente. Internamente, esta operación POST puede realizar tareas como comprobar las existencias y facturar al cliente. La respuesta HTTP puede indicar si el pedido se realizó correctamente o no. Observe también que un recurso no tiene que estar basado en un solo elemento de datos físicos. Por ejemplo, un recurso de pedido podría implementarse internamente con información agregada de muchas filas repartidas entre varias tablas en una base de datos relacional, pero presentada al cliente como una entidad única.

> [!TIP]
> Evite diseñar una interfaz REST que refleje o dependa de la estructura interna de los datos que expone. REST está orientada más a la implementación de operaciones sencillas de creación, recuperación, actualización y eliminación en tablas independientes de una base de datos relacional. El propósito de REST es asignar entidades de negocio y las operaciones que puede realizar una aplicación en estas entidades a la implementación física de estas entidades, pero el cliente no debería estar expuesto a estos detalles físicos.
>
>

Apenas existen entidades empresariales individuales de forma aislada (aunque pueden existir algunos objetos Singleton); por el contrario, tienden a agruparse en colecciones. En términos de REST, cada entidad y cada colección son recursos. En una API web RESTful, cada colección tiene su propio URI en el servicio web y al realizar una solicitud HTTP GET con el URI de una colección se recupera una lista de elementos de la colección. Cada elemento individual tiene también su propio URI,y una aplicación puede enviar otra solicitud HTTP GET con ese URI para recuperar los detalles de ese elemento. Debe organizar los URI de colecciones y elementos de forma jerárquica. En el sistema de comercio electrónico, el URI */customers* representa la colección del cliente y */customers/5* recupera los detalles del cliente con el identificador 5 de esta colección. Este enfoque ayuda a mantener el carácter intuitivo de la API web.

> [!TIP]
> Adopte una convención de nomenclatura coherente en los URI; en general, resulta útil usar nombres plurales para los URI que hacen referencia a colecciones.
>
>

También necesitará tener en cuenta las relaciones entre los diferentes tipos de recursos y cómo podría exponer estas asociaciones. Por ejemplo, los clientes pueden realizar cero pedidos o más. Una manera natural para representar esta relación sería mediante un URI como */customers/5/orders* para encontrar todos los pedidos del cliente 5. También puede plantearse la posibilidad de representar la asociación de un pedido con un cliente específico mediante un URI como */orders/99/customer* para encontrar al cliente del pedido 99; sin embargo, llevar este modelo demasiado lejos puede hacer que sea difícil de implementar. Una solución mejor es proporcionar vínculos navegables a recursos asociados, como el cliente, en el cuerpo del mensaje de respuesta HTTP que se devuelven cuando se consulta el pedido. Este mecanismo se describe con más detalle en la sección Uso del enfoque HATEOAS para permitir la navegación a los recursos relacionados, más adelante en esta guía.

En sistemas más complejos puede haber muchos más tipos de entidades y puede resultar tentador proporcionar identificadores URI que permitan a una aplicación cliente navegar a través de varios niveles de relaciones, como */customers/1/orders/99/products* para obtener la lista de productos del pedido 99 realizado por el cliente 1. Sin embargo, este nivel de complejidad puede ser difícil de mantener y es inflexible si las relaciones entre los recursos cambian en el futuro. En su lugar, debe intentar mantener los URI relativamente sencillos. Tenga en cuenta que una vez que una aplicación tiene una referencia a un recurso, debe ser posible usar esta referencia para encontrar los elementos relacionados con ese recurso. La consulta anterior se puede reemplazar por el URI */customers/1/orders* para encontrar todos los pedidos del cliente 1 y luego consultar el URI */orders/99/products* para encontrar los productos en este orden (suponiendo que el pedido 99 lo realizó el cliente 1).

> [!TIP]
> Evite el uso de URI de recursos más complejos que *collection/item/collection*.
>
>

Otro aspecto a tener en cuenta es que todas las solicitudes web imponen una carga en el servidor web, y cuanto mayor sea el número de solicitudes mayor será la carga. Debe intentar definir los recursos para evitar API web "conversadoras" que expongan un gran número de recursos pequeños. Una API de este tipo puede requerir que una aplicación cliente envíe varias solicitudes para encontrar todos los datos que necesita. Puede resultar útil desnormalizar los datos y combinar la información relacionada en recursos más grandes que se puedan recuperar mediante la emisión de una única solicitud. Sin embargo, debe equilibrar este enfoque con la sobrecarga de la captura de datos, lo cual el cliente podría no necesitar con mucha frecuencia. Recuperar objetos de gran tamaño puede aumentar la latencia de una solicitud e incurrir en costes adicionales de ancho de banda a cambio de pocas ventajas si los datos adicionales no se usan a menudo.

Evite la introducción de dependencias entre la API web y la estructura, el tipo o la ubicación de los orígenes de datos subyacentes. Por ejemplo, si los datos se encuentran en una base de datos relacional, la API web no necesita exponer cada tabla como una colección de recursos. Considere la API web como una abstracción de la base de datos y si es necesario introducir una capa de asignación entre la base de datos y la API web. De esta forma, si cambia el diseño o la implementación de la base de datos (por ejemplo, pasa de una base de datos relacional que contiene una colección de tablas normalizadas a un sistema de almacenamiento NoSQL desnormalizado, como una base de datos de documento), las aplicaciones cliente quedan aisladas de estos cambios.

> [!TIP]
> El origen de datos que sustenta una API web no tiene que ser un almacén de datos; puede ser otro servicio o aplicación de línea de negocio, o incluso una aplicación heredada que se ejecuta de forma local dentro de una organización.
>
>

Por último, no sería posible asignar cada operación que implementa una API web a un recurso concreto. Por ejemplo, puede controlar tales escenarios *no relacionados con recursos* a través de solicitudes HTTP GET que invocan una parte de la funcionalidad y devuelven los resultados como un mensaje de respuesta HTTP. Una API web que implementa operaciones sencillas de tipo calculadora como sumar y restar podría proporcionar URI que expongan estas operaciones como seudorecursos y utilicen la cadena de consulta para especificar los parámetros necesarios. Por ejemplo, una solicitud GET al URI */add?operand1=99&operand2=1* podría devolver un mensaje de respuesta donde el cuerpo contiene el valor 100 y una solicitud GET al URI */subtract?operand1=50&operand2=20* podría devolver un mensaje de respuesta donde el cuerpo contiene el valor 30. Sin embargo, únicamente use estas formas de URI con moderación.

### <a name="defining-operations-in-terms-of-http-methods"></a>Definición de operaciones en términos de métodos HTTP
El protocolo HTTP define una serie de métodos que asignan significado semántico a una solicitud. Los métodos HTTP comunes que usan la mayoría de las API web RESTful son:

* **GET**, para recuperar una copia del recurso en el URI especificado. El cuerpo del mensaje de respuesta contiene los detalles del recurso solicitado.
* **POST**, para crear un nuevo recurso en el URI especificado. El cuerpo del mensaje de solicitud proporciona los detalles del nuevo recurso. Tenga en cuenta que POST también puede usarse para desencadenar operaciones que en realidad no crean recursos.
* **PUT**, para reemplazar o actualizar el recurso en el URI especificado. El cuerpo del mensaje de solicitud especifica el recurso que se va a modificar y los valores que se aplican.
* **DELETE**, para quitar el recurso en el URI especificado.

> [!NOTE]
> El protocolo HTTP también define otros métodos menos frecuentes, como PATCH, que se usa para solicitar actualizaciones selectivas a un recurso, HEAD, que se usa para solicitar una descripción de un recurso, OPTIONS, que permite que un cliente obtenga información sobre las opciones de comunicación admitidas por el servidor y TRACE, que permite que un cliente solicite información que puede usar para fines de prueba y diagnóstico.
>
>

El efecto de una solicitud específica dependerá de si el recurso al que se aplica es una colección o un elemento individual. En la tabla siguiente se resumen las convenciones comunes adoptadas por la mayoría de las implementaciones de RESTful usando el ejemplo de comercio electrónico. Tenga en cuenta que no todas estas solicitudes se pueden implementar; depende de cada situación específica.

| **Recurso** | **POST** | **GET** | **PUT** | **DELETE** |
| --- | --- | --- | --- | --- |
| /customers |Crear un nuevo cliente |Recuperar todos los clientes |Actualización masiva de clientes (*si está implementado*) |Eliminar todos los clientes |
| /customers/1 |Error |Recuperar los detalles del cliente 1 |Actualizar los detalles del cliente 1, si existe; en caso contrario, se devuelve un error |Quitar al cliente 1 |
| /customers/1/orders |Crear un nuevo pedido para el cliente 1 |Recuperar todos los pedidos del cliente 1 |Actualización masiva de pedidos del cliente 1 (*si está implementado*) |Eliminación de todos los pedidos del cliente 1 (*si está implementado*) |

El propósito de las solicitudes GET y DELETE es relativamente sencillo, pero existe un margen de confusión respecto a la finalidad y los efectos de las solicitudes POST y PUT.

Una solicitud POST debe crear un nuevo recurso con los datos proporcionados en el cuerpo de la solicitud. En el modelo REST, con frecuencia se aplican solicitudes POST a recursos que son colecciones; el nuevo recurso se agrega a la colección.

> [!NOTE]
> También puede definir solicitudes POST que desencadenen alguna funcionalidad (y que no devuelvan datos necesariamente), y estos tipos de solicitud se pueden aplicar a colecciones. Por ejemplo, podría usar una solicitud POST para pasar un parte de horas a un servicio de procesamiento de nóminas y obtener los impuestos calculados como respuesta.
>
>

Una solicitud PUT se diseñó para modificar un recurso existente. Si el recurso especificado no existe, la solicitud PUT podría devolver un error (en algunos casos, puede crear realmente el recurso). Las solicitudes PUT se aplican con mayor frecuencia a recursos que son elementos individuales (por ejemplo, un cliente o pedido concreto), aunque se pueden aplicar a colecciones, si bien es una implementación menos común. Tenga en cuenta que las solicitudes PUT son idempotentes, mientras que las solicitudes POST no lo son; si una aplicación envía la misma solicitud PUT varias veces, los resultados siempre debe ser los mismos (el mismo recurso se modificará con los mismos valores), pero si una aplicación repite la misma solicitud POST, el resultado será la creación de varios recursos.

> [!NOTE]
> En realidad, una solicitud HTTP PUT reemplaza un recurso existente por el recurso especificado en el cuerpo de la solicitud. Si su intención es modificar una selección de propiedades de un recurso y dejar otras propiedades sin cambiar, esta implementación requiere una solicitud HTTP PATCH. Sin embargo, muchas implementaciones de RESTful son flexibles con esta regla y usan PUT en ambos casos.
>
>

### <a name="processing-http-requests"></a>Procesamiento de solicitudes HTTP
Los datos incluidos por una aplicación de cliente en muchas solicitudes HTTP, y los correspondientes mensajes de respuesta del servidor web, podrían presentarse en una variedad de formatos (o tipos de medios). Por ejemplo, los datos que especifican los detalles de un cliente o un pedido se podrían proporcionar como XML, JSON o algún otro formato codificado y comprimido. Una API web RESTful debe admitir distintos tipos de medios, según solicite la aplicación cliente que envía la solicitud.

Cuando una aplicación cliente envía una solicitud que devuelve datos en el cuerpo de un mensaje, puede especificar los tipos de medios que puede controlar en el encabezado Accept de la solicitud. El código siguiente ilustra una solicitud HTTP GET que recupera los detalles del pedido 2 y solicita que el resultado se devuelva como JSON (el cliente todavía debe examinar el tipo de medio de los datos en la respuesta para comprobar el formato de los datos devueltos):

```HTTP
GET http://adventure-works.com/orders/2 HTTP/1.1
...
Accept: application/json
...
```

Si el servidor web admite este tipo de medio, puede responder con una respuesta que incluye el encabezado Content-Type que especifica el formato de los datos en el cuerpo del mensaje:

> [!NOTE]
> Para obtener la máxima interoperatividad, los tipos de medios a los que se hace referencia en los encabezados Accept y Content-Type deben ser tipos MIME reconocidos en lugar de algún tipo de medio personalizado.
>
>

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
Content-Length: ...
{"orderID":2,"productID":4,"quantity":2,"orderValue":10.00}
```

Si el servidor web no admite el tipo de medio solicitado, puede enviar los datos en un formato diferente. EN todos los casos, debe especificar el tipo de medio (como *application/json*) en el encabezado Content-Type. Es responsabilidad de la aplicación analizar el mensaje de respuesta e interpretar correctamente los resultados en el cuerpo del mensaje.

Tenga en cuenta que en este ejemplo, el servidor web recupera los datos solicitados correctamente, lo que indica mediante la devolución de un código de estado 200 en el encabezado de respuesta. Si no se encuentran datos coincidentes, debe devolver en su lugar un código de estado 404 (no encontrado) y el cuerpo del mensaje de respuesta puede contener información adicional. El formato de esta información se especifica en el encabezado Content-Type, tal como se muestra en el ejemplo siguiente:

```HTTP
GET http://adventure-works.com/orders/222 HTTP/1.1
...
Accept: application/json
...
```

El pedido 222 no existe, por lo que el mensaje de respuesta tiene el siguiente aspecto:

```HTTP
HTTP/1.1 404 Not Found
...
Content-Type: application/json; charset=utf-8
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
Content-Length: ...
{"message":"No such order"}
```

Cuando una aplicación envía una solicitud HTTP PUT para actualizar un recurso, especifica el URI del recurso y proporciona los datos que se van a modificar en el cuerpo del mensaje de solicitud. También debe especificar el formato de estos datos usando el encabezado Content-Type. Un formato común usado en la información basada en texto es *application/x-www-form-urlencoded*, que consta de un conjunto de pares de nombre y valor separados por el carácter &. En el ejemplo siguiente se muestra una solicitud HTTP PUT que modifica la información del pedido 1:

```HTTP
PUT http://adventure-works.com/orders/1 HTTP/1.1
...
Content-Type: application/x-www-form-urlencoded
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
Content-Length: ...
ProductID=3&Quantity=5&OrderValue=250
```

Si la modificación es correcta, la respuesta debería ser normalmente un código de estado HTTP 204, que indica que el proceso se ha controlado correctamente, pero que el cuerpo de respuesta no contiene ninguna información adicional. El encabezado Location en la respuesta contiene el URI del recurso recién actualizado:

```HTTP
HTTP/1.1 204 No Content
...
Location: http://adventure-works.com/orders/1
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
```

> [!TIP]
> Si los datos de un mensaje de solicitud HTTP PUT incluyen información de fecha y hora, asegúrese de que el servicio web acepte el formato de las fechas y horas que siguen la norma ISO 8601.
>
>

Si el recurso que se va a actualizar no existe, el servidor web puede responder con una respuesta de no encontrado como se describió anteriormente. O bien, si el servidor crea en realidad el propio objeto, podría devolver los códigos de estado HTTP 200 (correcto) o HTTP 201 (creado) y el cuerpo de respuesta podría contener los datos del nuevo recurso. Si el encabezado Content-Type de la solicitud especifica un formato de datos que el servidor web no puede controlar, debería responder con el código de estado HTTP 415 (tipo de medio no compatible).

> [!TIP]
> Considere la posibilidad de implementar operaciones HTTP PUT masivas que pueden procesar por lotes las actualizaciones de varios recursos de una colección. La solicitud PUT debe especificar el URI de la colección y el cuerpo de solicitud debe especificar los detalles de los recursos que se van a modificar. Este enfoque puede ayudar a reducir el intercambio de mensajes y mejorar el rendimiento.
>
>

El formato de una solicitud HTTP POST que crear nuevos recursos es parecido al de las solicitudes PUT; el cuerpo del mensaje contiene los detalles del nuevo recurso que se va a agregar. Sin embargo, el URI normalmente especifica la colección a la que se debe agregar el recurso. En el ejemplo siguiente se crea un nuevo pedido y se agrega a la colección de pedidos:

```HTTP
POST http://adventure-works.com/orders HTTP/1.1
...
Content-Type: application/x-www-form-urlencoded
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
Content-Length: ...
productID=5&quantity=15&orderValue=400
```

Si la solicitud es correcta, el servidor web debe responder con un código de mensaje con el código de estado HTTP 201 (creado). El encabezado Location debe contener el URI del recurso recién creado y el cuerpo de la respuesta debe contener una copia del nuevo recurso; el encabezado Content-Type especifica el formato de estos datos:

```HTTP
HTTP/1.1 201 Created
...
Content-Type: application/json; charset=utf-8
Location: http://adventure-works.com/orders/99
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
Content-Length: ...
{"orderID":99,"productID":5,"quantity":15,"orderValue":400}
```

> [!TIP]
> Si los datos que ha proporcionado una solicitud PUT o POST no son válidos, el servidor web debe responder con un mensaje con el código de estado HTTP 400 (solicitud incorrecta). El cuerpo del mensaje puede contener información adicional sobre el problema con la solicitud y los formatos esperados, o puede contener un vínculo a una dirección URL que proporciona más detalles.
>
>

Para quitar un recurso, una solicitud HTTP DELETE simplemente proporciona el URI del recurso que se va a eliminar. En el ejemplo siguiente se intenta quitar el pedido 99:

```HTTP
DELETE http://adventure-works.com/orders/99 HTTP/1.1
...
```

Si la operación de eliminación se realiza correctamente, el servidor web debe responder con el código de estado HTTP 204, que indica que el proceso se ha controlado correctamente, pero que el cuerpo de respuesta no contiene información adicional (es la misma respuesta que se devuelve en una operación PUT correcta, pero sin un encabezado Location dado que el recurso ya no existe). También es posible que una solicitud DELETE devuelva el código de estado HTTP 200 (correcto) o 202 (aceptado) si la eliminación se realiza de forma asincrónica.

```HTTP
HTTP/1.1 204 No Content
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
```

Si no se encuentra el recurso, el servidor web debe devolver en su lugar un mensaje 404 (no encontrado).

> [!TIP]
> Si todos los recursos de una colección deben eliminarse, permita que se especifique una solicitud HTTP DELETE para el URI de la colección, en lugar de obligar a una aplicación a quitar uno a uno cada recurso de la colección.
>
>

### <a name="filtering-and-paginating-data"></a>Filtrado y paginación de datos
Debe procurar que los URI sean sencillos e intuitivos. Exponer una colección de recursos con un único URI ayuda a este respecto, pero puede dar lugar a que las aplicaciones capturen grandes cantidades de datos cuando solo se requiere un subconjunto de la información. La generación de un gran volumen de tráfico no solo afecta al rendimiento y la escalabilidad del servidor web, sino también a la capacidad de respuesta de las aplicaciones cliente que solicitan los datos.

Por ejemplo, si los pedidos contienen el precio pagado por el pedido, una aplicación cliente que necesite recuperar todos los pedidos que tienen un coste sobre un valor concreto podría tener que recuperar todos los pedidos del URI */orders* y luego filtrar esos pedidos localmente. Obviamente, este proceso es muy ineficaz; desperdicia el ancho de banda de red y la potencia de procesamiento en el servidor que hospeda la API web.

Una solución podría ser proporcionar un esquema de URI como */orders/ordervalue_greater_than_n* donde *n* es el precio del pedido pero, para un número limitado de precios, este enfoque no resulta práctico. Además, si necesita consultar órdenes basándose en otros criterios, al final puede tener que proporcionar una larga lista de URI con nombres que posiblemente no sean intuitivos.

Una estrategia mejor para filtrar los datos es proporcionar los criterios de filtro en la cadena de consulta que se pasa a la API web, como */orders?ordervaluethreshold=n*. En este ejemplo, la operación correspondiente en la API web es responsable de analizar y controlar el parámetro `ordervaluethreshold` en la cadena de consulta y de devolver los resultados filtrados en la respuesta HTTP.

Algunas solicitudes HTTP GET simples sobre los recursos de colección podrían devolver un gran número de elementos. Para luchar contra la posibilidad de que esto suceda, debe diseñar la API web para limitar la cantidad de datos devueltos en una sola solicitud. Para ello, admita cadenas de consulta que permitan que el usuario especifique el número máximo de elementos que se van a recuperar (lo cual podría estar sujeto a un límite superior para ayudar a evitar ataques por denegación de servicio) y un desplazamiento inicial en la colección. Por ejemplo, la cadena de consulta en el URI */orders?limit=25&offset=50* debe recuperar 25 pedidos a partir del pedido 50 encontrado en la colección de pedidos. Al igual que en el filtrado de datos, la operación que implementa la solicitud GET en la API web es responsable de analizar y controlar los parámetros `limit` y `offset` en la cadena de consulta. Para ayudar a las aplicaciones cliente, las solicitudes GET que devuelven datos paginados deben incluir también alguna forma de metadatos que indiquen el número total de recursos disponibles en la colección. También puede considerar otras estrategias de paginación inteligente; para obtener más información, consulte [Notas para el diseño de API: paginación inteligente](http://bizcoder.com/api-design-notes-smart-paging)

Puede seguir una estrategia similar para ordenar los datos que se capturan; puede proporcionar un parámetro de ordenación que tome un nombre de campo como valor, por ejemplo */orders?sort=ProductID*. Sin embargo, tenga en cuenta que este enfoque puede tener un efecto negativo sobre el almacenamiento en caché (los parámetros de consulta forman parte del identificador de recursos que se usa en muchas implementaciones de caché como llave a los datos en caché).

Puede ampliar este enfoque para limitar (proyectar) los campos devueltos si un solo elemento de recurso contiene una gran cantidad de datos. Por ejemplo, podría usar un parámetro de cadena de consulta que acepte una lista de campos delimitados por coma, como */orders?fields=ProductID,Quantity*.

> [!TIP]
> Asigne a todos los parámetros opcionales de las cadenas de consulta valores predeterminados. Por ejemplo, establezca el parámetro `limit` en 10 y el parámetro `offset` en 0 si implementa paginación, establezca el parámetro de ordenación en la clave del recurso si implementa ordenación y establezca el parámetro `fields` en todos los campos del recurso si admite proyecciones.
>
>

### <a name="handling-large-binary-resources"></a>Control de recursos binarios de gran tamaño
Un único recurso puede contener campos binarios grandes, como imágenes o archivos. Para superar los problemas de transmisión ocasionados por conexiones intermitentes y poco confiables y para mejorar los tiempos de respuesta, considere la posibilidad de proporcionar operaciones que permitan que la aplicación  cliente recupere tales recursos en fragmentos. Para ello, la API web debe admitir el encabezado Accept-Ranges para solicitudes GET de recursos de gran tamaño, y lo ideal es implementar HTTP HEAD para estos recursos. El encabezado Accept-Ranges indica que la operación GET admite resultados parciales y que una aplicación cliente puede enviar solicitudes GET que devuelven un subconjunto de un recurso especificado como un intervalo de bytes. Una solicitud HEAD es similar a una solicitud GET, excepto que solo devuelve un encabezado que describe el recurso y un cuerpo de mensaje vacío. Una aplicación cliente puede emitir una solicitud HEAD para determinar si se debe capturar un recurso mediante solicitudes GET parciales. En el ejemplo siguiente se muestra una solicitud HEAD que obtiene información sobre una imagen de producto:

```HTTP
HEAD http://adventure-works.com/products/10?fields=productImage HTTP/1.1
...
```

El mensaje de respuesta contiene un encabezado que incluye el tamaño del recurso (4580 bytes) y el encabezado Accept-Ranges de que la operación GET correspondiente admite resultados parciales:

```HTTP
HTTP/1.1 200 OK
...
Accept-Ranges: bytes
Content-Type: image/jpeg
Content-Length: 4580
...
```

La aplicación cliente puede usar esta información para construir una serie de operaciones GET para recuperar la imagen en fragmentos más pequeños. La primera solicitud captura los primeros 2500 bytes mediante el encabezado Range:

```HTTP
GET http://adventure-works.com/products/10?fields=productImage HTTP/1.1
Range: bytes=0-2499
...
```

El mensaje de respuesta indica que es una respuesta parcial al devolver el código de estado HTTP 206. El encabezado Content-Length especifica el número real de bytes devuelto en el cuerpo del mensaje (no el tamaño del recurso) y el encabezado Content-Range indica qué parte del recurso es (bytes 0-2499 de 4580):

```HTTP
HTTP/1.1 206 Partial Content
...
Accept-Ranges: bytes
Content-Type: image/jpeg
Content-Length: 2500
Content-Range: bytes 0-2499/4580
...
_{binary data not shown}_
```

Una solicitud posterior de la aplicación cliente puede recuperar el resto de los recursos mediante el uso de un encabezado Range apropiado:

```HTTP
GET http://adventure-works.com/products/10?fields=productImage HTTP/1.1
Range: bytes=2500-
...
```

El mensaje de resultado correspondiente debe tener este aspecto:

```HTTP
HTTP/1.1 206 Partial Content
...
Accept-Ranges: bytes
Content-Type: image/jpeg
Content-Length: 2080
Content-Range: bytes 2500-4580/4580
...
```

## <a name="using-the-hateoas-approach-to-enable-navigation-to-related-resources"></a>Uso del enfoque HATEOAS para permitir la exploración a recursos relacionados
Uno de los principales propósitos que se esconden detrás de REST es que debe ser posible navegar por todo el conjunto de recursos sin necesidad de conocer el esquema de URI. Cada solicitud HTTP GET debe devolver la información necesaria para encontrar los recursos relacionados directamente con el objeto solicitado mediante los hipervínculos que se incluyen en la respuesta, y también se le debe proporcionar información que describa las operaciones disponibles en cada uno de estos recursos. Este principio se conoce como HATEOAS, del inglés Hypertext as the Engine of Application State (Hipertexto como motor del estado de la aplicación). El sistema es realmente una máquina de estado finito, y la respuesta a cada solicitud contiene la información necesaria para pasar de un estado a otro; ninguna otra información debería ser necesaria.

> [!NOTE]
> Actualmente no hay ninguna norma o especificación que define cómo modelar el principio HATEOAS. Los ejemplos mostrados en esta sección muestran una posible solución.
>
>

Por ejemplo, para controlar la relación entre clientes y pedidos, los datos devueltos en la respuesta para un pedido específico deben contener URI en forma de un hipervínculo que identifica al cliente que realizó el pedido y las operaciones que se pueden realizar en ese cliente.

```HTTP
GET http://adventure-works.com/orders/3 HTTP/1.1
Accept: application/json
...
```

El cuerpo del mensaje de respuesta contiene una matriz `links` (resaltada en el código de ejemplo) que especifica la naturaleza de la relación (*Customer*), el URI del cliente (*http://adventure-works.com/customers/3*), cómo recuperar los detalles de este cliente (*GET*) y los tipos MIME que admite el servidor web para recuperar esta información (*text/xml* y *application/json*). Esta es toda la información que una aplicación cliente necesita para capturar los detalles del cliente. Además, la matriz de vínculos también incluye vínculos para que las demás operaciones puedan realizarse, como PUT (para modificar al cliente, junto con el formato que el servidor web espera que proporcione el cliente) y DELETE.

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"orderID":3,"productID":2,"quantity":4,"orderValue":16.60,"links":[(some links omitted){"rel":"customer","href":" http://adventure-works.com/customers/3", "action":"GET","types":["text/xml","application/json"]},{"rel":"
customer","href":" http://adventure-works.com /customers/3", "action":"PUT","types":["application/x-www-form-urlencoded"]},{"rel":"customer","href":" http://adventure-works.com /customers/3","action":"DELETE","types":[]}]}
```

Por integridad, la matriz de vínculos también debe incluir información que hace referencia a sí misma relativa al recurso que se ha recuperado. Estos vínculos se han omitido en el ejemplo anterior, pero se resaltan en el código siguiente. Observe que en estos vínculos, la relación *self* se ha usado para indicar que se trata de una referencia al recurso devuelto por la operación:

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"orderID":3,"productID":2,"quantity":4,"orderValue":16.60,"links":[{"rel":"self","href":" http://adventure-works.com/orders/3", "action":"GET","types":["text/xml","application/json"]},{"rel":" self","href":" http://adventure-works.com /orders/3", "action":"PUT","types":["application/x-www-form-urlencoded"]},{"rel":"self","href":" http://adventure-works.com /orders/3", "action":"DELETE","types":[]},{"rel":"customer",
"href":" http://adventure-works.com /customers/3", "action":"GET","types":["text/xml","application/json"]},{"rel":" customer" (customer links omitted)}]}
```

Para que este método sea eficaz, las aplicaciones cliente deben estar preparadas para recuperar y analizar esta información adicional.

## <a name="versioning-a-restful-web-api"></a>Control de versiones de una API web RESTful
Es muy improbable que en todas las situaciones, excepto las más simples, una API web permanezca estática. Conforme los requisitos empresariales cambian, se pueden agregar nuevas colecciones de recursos, las relaciones entre los recursos pueden cambiar y la estructura de los datos de los recursos puede rectificarse. Si bien la actualización de una API para controlar los requisitos nuevos o diferentes es un proceso relativamente sencillo, debe tener en cuenta los efectos que tendrán dichos cambios en las aplicaciones cliente que utilizan la API web. El problema es que, aunque el desarrollador que diseña e implementa una API web tiene control total sobre dicha API, el desarrollador carece del mismo grado de control sobre las aplicaciones cliente que podrían crear organizaciones de terceros que operan de forma remota. La premisa principal es permitir que las aplicaciones cliente existentes sigan funcionando sin cambios y al mismo tiempo dejar que las nuevas aplicaciones cliente aprovechen las ventajas de nuevas características y recursos.

El control de versiones permite que una API web indique las características y recursos que expone y que una aplicación cliente pueda enviar solicitudes que se dirijan a una versión específica de una característica o un recurso. En las secciones siguientes se describen varios enfoques diferentes, cada uno de los cuales tiene sus propias ventajas y desventajas.

### <a name="no-versioning"></a>Sin control de versiones
Este es el enfoque más sencillo y puede ser aceptable para algunas API internas. Los grandes cambios podrían representarse como nuevos recursos o nuevos vínculos.  Agregar contenido a recursos existentes puede que no represente un cambio importante dado que las aplicaciones cliente que no esperan ver este contenido simplemente lo ignorarán.

Por ejemplo, una solicitud al URI *http://adventure-works.com/customers/3* debe devolver los detalles de un solo cliente que contiene los campos `id`, `name` y `address` esperados por la aplicación cliente:

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"id":3,"name":"Contoso LLC","address":"1 Microsoft Way Redmond WA 98053"}
```

> [!NOTE]
> Con fines de simplicidad y claridad, las respuestas de ejemplo que se muestran en esta sección no incluyen vínculos HATEOAS.
>
>

Si el campo `DateCreated` se agrega al esquema del recurso de cliente, la respuesta tendría el siguiente aspecto:

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":"1 Microsoft Way Redmond WA 98053"}
```

Las aplicaciones cliente existentes pueden seguir funcionando correctamente si son capaces de omitir los campos no reconocidos, pero las nuevas aplicaciones cliente se pueden diseñar para controlar este nuevo campo. Sin embargo, si se producen cambios más radicales en el esquema de recursos (por ejemplo, se quitan campos y se cambian de nombre) o cambian las relaciones entre los recursos, dichos cambios podrían constituir cambios importantes que impiden que las aplicaciones cliente existentes funcionen correctamente. En estas situaciones es aconsejable uno de los enfoques siguientes.

### <a name="uri-versioning"></a>Control de versiones de URI
Cada vez que modifica la API web o cambia el esquema de recursos, agrega un número de versión al URI para cada recurso. Los URI ya existentes deben seguir funcionando como antes y devolver los recursos conforme a su esquema original.

Ampliando el ejemplo anterior, si el campo `address` se reestructura en campos secundarios que contienen cada uno una parte constituyente de la dirección (como `streetAddress`, `city`, `state` y `zipCode`), esta versión del recurso podría exponerse a través de un URI que contenga un número de versión, como http://adventure-works.com/v2/customers/3:

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":{"streetAddress":"1 Microsoft Way","city":"Redmond","state":"WA","zipCode":98053}}
```

Este mecanismo de control de versiones es muy sencillo, pero depende del servidor que enruta la solicitud al extremo adecuado. Sin embargo, puede volverse difícil de manejar dado que la API web madura a través de varias iteraciones y el servidor tiene que admitir un número de versiones diferentes. Además, desde el punto de vista de los más puristas, en todos los casos las aplicaciones cliente capturan los mismos datos (cliente 3), así que el URI no debería ser realmente diferente según la versión. Este esquema también complica la implementación de HATEOAS ya que todos los vínculos deberán incluir el número de versión en sus URI.

### <a name="query-string-versioning"></a>Control de versiones de cadena de consulta
En lugar de proporcionar varios URI, puede especificar la versión del recurso mediante un parámetro dentro de la cadena de consulta anexada a la solicitud HTTP, como *http://adventure-works.com/customers/3?version=2*. El parámetro de versión debe adoptar de forma predeterminada un valor significativo como 1 si se omite en las aplicaciones cliente más antiguas.

Este enfoque tiene la ventaja semántica de que el mismo recurso siempre se recupera del mismo URI, pero depende del código que controla la solicitud para analizar la cadena de consulta y enviar la respuesta HTTP adecuada. Este método también presenta tiene las mismas complicaciones para implementar HATEOAS que el mecanismo de control de versiones de URI.

> [!NOTE]
> Algunos exploradores web y servidores proxy antiguos no almacenan en caché las respuestas de solicitudes que incluyen una cadena de consulta en la dirección URL. Esto puede tener un impacto negativo en el rendimiento de las aplicaciones web que usan una API web y que se ejecutan en este tipo de explorador web.
>
>

### <a name="header-versioning"></a>Control de versiones de encabezado
En lugar de anexar el número de versión como un parámetro de cadena de consulta, podría implementar un encabezado personalizado que indica la versión del recurso. Este enfoque requiere que la aplicación cliente agregue el encabezado adecuado a las solicitudes, aunque el código que controla la solicitud de cliente puede usar un valor predeterminado (versión 1) si se omite el encabezado de versión. En los ejemplos siguientes se usa un encabezado personalizado denominado *Custom-Header*. El valor de este encabezado indica la versión de la API web.

Versión 1:

```HTTP
GET http://adventure-works.com/customers/3 HTTP/1.1
...
Custom-Header: api-version=1
...
```

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"id":3,"name":"Contoso LLC","address":"1 Microsoft Way Redmond WA 98053"}
```

Versión 2:

```HTTP
GET http://adventure-works.com/customers/3 HTTP/1.1
...
Custom-Header: api-version=2
...
```

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":{"streetAddress":"1 Microsoft Way","city":"Redmond","state":"WA","zipCode":98053}}
```

Tenga en cuenta que al igual que en los dos enfoques anteriores, la implementación de HATEOAS requiere que se incluya el encabezado personalizado apropiado en los vínculos.

### <a name="media-type-versioning"></a>Control de versiones del tipo de medio
Cuando una aplicación cliente envía una solicitud HTTP GET a un servidor web, debe prever el formato del contenido que puede controlar mediante el uso de un encabezado Accept, como se ha descrito anteriormente en esta guía. Con frecuencia, el propósito del encabezado *Accept* es permitir que la aplicación cliente especifique si el cuerpo de la respuesta debe ser XML, JSON o algún otro formato común que pueda analizar el cliente. Sin embargo, es posible definir tipos de medios personalizados que incluyan información que permita que la aplicación cliente indique qué versión de un recurso que se espera. En el ejemplo siguiente se muestra una solicitud que especifica un encabezado *Accept* con el valor *application/vnd.adventure-works.v1+json*. El elemento *vnd.adventure-works.v1* indica al servidor web que debe devolver la versión 1 del recurso, mientras que el elemento *json* especifica que el formato del cuerpo de respuesta debe ser JSON:

```HTTP
GET http://adventure-works.com/customers/3 HTTP/1.1
...
Accept: application/vnd.adventure-works.v1+json
...
```

El código que controla la solicitud es responsable de procesar el encabezado *Accept* y de respetarlo siempre que sea posible (la aplicación cliente puede especificar varios formatos en el encabezado *Accept*, en cuyo caso el servidor web puede elegir el formato más adecuado para el cuerpo de respuesta). El servidor web confirma el formato de los datos en el cuerpo de respuesta mediante el encabezado Content-Type:

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/vnd.adventure-works.v1+json; charset=utf-8
...
Content-Length: ...
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
* Las [directrices de API de REST de Microsoft](https://github.com/Microsoft/api-guidelines/blob/master/Guidelines.md) contienen recomendaciones detalladas para diseñar API de REST públicas.
* El [Libro de cocina de RESTful](http://restcookbook.com/) contiene una introducción a la creación de API RESTful.
* La [lista de comprobación de Web API](https://mathieu.fenniak.net/the-api-checklist/) contiene una lista útil de elementos que se deben tener en cuenta al diseñar e implementar una API web.
* El sitio [Open API Initiative](https://www.openapis.org/) contiene toda la documentación e información detallada acerca de Open API.
