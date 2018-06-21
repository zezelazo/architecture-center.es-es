---
title: Guía de implementación de API
description: Guía sobre cómo implementar una API.
author: dragon119
ms.date: 07/13/2016
pnp.series.title: Best Practices
ms.openlocfilehash: cc28864de36afdeed2f8a7155a307e312c3a398e
ms.sourcegitcommit: c93f1b210b3deff17cc969fb66133bc6399cfd10
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/05/2018
ms.locfileid: "27596026"
---
# <a name="api-implementation"></a>Implementación de API

Una API web de RESTful diseñada cuidadosamente define los recursos, las relaciones y los esquemas de navegación a los que tienen acceso las aplicaciones cliente. Al implementar una API web, debe considerar los requisitos físicos del entorno que hospeda la API web y la forma en que se construye la misma, en lugar de la estructura lógica de los datos. Esta guía se centra en los procedimientos recomendados para implementar una API web y publicarla para que esté disponible para las aplicaciones cliente. Para obtener información detallada sobre el diseño de API web, consulte [Guía de diseño de API](/azure/architecture/best-practices/api-design).

## <a name="processing-requests"></a>Procesamiento de solicitudes

Tenga en cuenta los puntos siguientes cuando implemente el código para administrar las solicitudes.

### <a name="get-put-delete-head-and-patch-actions-should-be-idempotent"></a>Las acciones GET, PUT, DELETE, HEAD y PATCH deben ser idempotentes.

El código que implementa estas solicitudes no debe imponer efectos secundarios. La misma solicitud repetida en el mismo recurso debe dar como resultado el mismo estado. Por ejemplo, enviar varias solicitudes DELETE para el mismo URI debe tener el mismo efecto, aunque el código de estado HTTP en los mensajes de respuesta sea diferente. La primera solicitud DELETE podría devolver el código de estado 204 (sin contenido), mientras que una solicitud DELETE posterior podría devolver el código de estado 404 (no encontrado).

> [!NOTE]
> En el artículo [Idempotency Patterns](http://blog.jonathanoliver.com/idempotency-patterns/) del blog de Jonathan Oliver se ofrece una visión general de la idempotencia y cómo se relaciona con las operaciones de administración de datos.
>

### <a name="post-actions-that-create-new-resources-should-not-have-unrelated-side-effects"></a>Las acciones POST que crean nuevos recursos no deben tener efectos secundarios no relacionados.

Si el objetivo de una solicitud POST es crear un recurso nuevo, los efectos de la solicitud deben limitarse al nuevo recurso (y posiblemente cualquier recurso relacionado directamente si hay algún tipo de vinculación implicada). Por ejemplo, en un sistema de comercio electrónico, una solicitud POST que crea un nuevo pedido para un cliente podría también modificar los niveles del inventario y generar información de facturación, pero no debería modificar la información no relacionada directamente con el pedido o tener otros efectos secundarios en el estado general del sistema.

### <a name="avoid-implementing-chatty-post-put-and-delete-operations"></a>Evite implementar operaciones POST, PUT y DELETE que generen mucha conversación.

Admita las solicitudes POST, PUT y DELETE en las colecciones de recursos. Una solicitud POST puede contener los detalles de varios recursos nuevos y agregarlos todos a la misma colección, una solicitud PUT puede reemplazar todo el conjunto de recursos de una colección y una solicitud DELETE puede eliminar una colección completa.

La compatibilidad con OData que se incluye en API Web 2 de ASP.NET permite las solicitudes por lotes. Una aplicación cliente puede empaquetar varias solicitudes de API web, enviarlas al servidor en una sola solicitud HTTP y recibir una respuesta HTTP única que contenga las respuestas a cada solicitud. Para más información, consulte [Introducing Batch Support in Web API and Web API OData](http://blogs.msdn.com/b/webdev/archive/2013/11/01/introducing-batch-support-in-web-api-and-web-api-odata.aspx) (Incorporación de compatibilidad para procesos por lotes en Web API Y Web API OData).

### <a name="follow-the-http-specification-when-sending-a-response"></a>Siga la especificación HTTP al enviar una respuesta. 

Una API web debe devolver mensajes que contengan el código de estado HTTP correcto para que el cliente determine cómo quiere controlar el resultado, los encabezados HTTP adecuados para que el cliente entienda la naturaleza de los resultados y un cuerpo con el formato apropiado para que el cliente analice los resultados. 

Por ejemplo, una operación POST debe devolver el código de estado 201 (creado) y el mensaje de respuesta debe incluir el URI del recurso recién creado en el encabezado Location del mensaje de respuesta.

### <a name="support-content-negotiation"></a>Admita la negociación de contenido.

El cuerpo de un mensaje de respuesta puede contener datos en una variedad de formatos. Por ejemplo, una solicitud HTTP GET podría devolver datos en formato JSON o XML. Cuando el cliente envía una solicitud, puede incluir un encabezado Accept que especifica los formatos de datos que puede controlar. Estos formatos se especifican como tipos de medios. Por ejemplo, un cliente que emite una solicitud GET para recuperar una imagen puede especificar un encabezado Accept que enumere los tipos de medios que el cliente puede administrar, como por ejemplo "image/jpeg, image/gif, image/png".  Cuando la API web devuelve el resultado, debe dar formato a los datos con uno de estos tipos de medios y especificar el formato en el encabezado Content-Type de la respuesta.

Si el cliente no especifica un encabezado Accept, use un formato predeterminado razonable para el cuerpo de respuesta. Por ejemplo, el tipo predeterminado del marco API Web de ASP.NET es JSON para datos basados en texto.

### <a name="provide-links-to-support-hateoas-style-navigation-and-discovery-of-resources"></a>Proporcione vínculos para admitir la navegación y la detección de recursos de estilo HATEOAS.

El enfoque de HATEOAS permite que un cliente navegue y detecte recursos desde un punto de partida inicial. Esto se logra mediante el uso de vínculos que contienen URI; Cuando un cliente emite una solicitud HTTP GET para obtener un recurso, la respuesta debe contener URI que permitan que una aplicación cliente localice rápidamente los recursos directamente relacionados. Por ejemplo, en una API web que admita una solución de comercio electrónico, un cliente puede haber realizado muchos pedidos. Cuando una aplicación cliente recupera los detalles de un cliente, la respuesta debe incluir vínculos que permitan que la aplicación cliente envíe solicitudes HTTP GET que pueden recuperar esos pedidos. Además, los vínculos de estilo HATEOAS deben describir las demás operaciones (POST, PUT, DELETE, etc.) que admite cada recurso vinculado junto con el URI correspondiente para realizar cada solicitud. Este enfoque se describe con más detalle en [Diseño de API][api-design].

Actualmente no hay estándares que rijan la implementación de HATEOAS, pero en el ejemplo siguiente se muestra un posible enfoque. En este ejemplo, una solicitud HTTP GET que encuentre los detalles de un cliente devuelve una respuesta que incluye vínculos de HATEOAS que hacen referencia a los pedidos de ese cliente:

```HTTP
GET http://adventure-works.com/customers/2 HTTP/1.1
Accept: text/json
...
```

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"CustomerID":2,"CustomerName":"Bert","Links":[
    {"rel":"self",
    "href":"http://adventure-works.com/customers/2",
    "action":"GET",
    "types":["text/xml","application/json"]},
    {"rel":"self",
    "href":"http://adventure-works.com/customers/2",
    "action":"PUT",
    "types":["application/x-www-form-urlencoded"]},
    {"rel":"self",
    "href":"http://adventure-works.com/customers/2",
    "action":"DELETE",
    "types":[]},
    {"rel":"orders",
    "href":"http://adventure-works.com/customers/2/orders",
    "action":"GET",
    "types":["text/xml","application/json"]},
    {"rel":"orders",
    "href":"http://adventure-works.com/customers/2/orders",
    "action":"POST",
    "types":["application/x-www-form-urlencoded"]}
]}
```

En este ejemplo, los datos del cliente se representan mediante la clase `Customer` que se muestra en el siguiente fragmento de código. Los vínculos de HATEOAS se mantienen en la propiedad de la colección `Links` :

```csharp
public class Customer
{
    public int CustomerID { get; set; }
    public string CustomerName { get; set; }
    public List<Link> Links { get; set; }
    ...
}

public class Link
{
    public string Rel { get; set; }
    public string Href { get; set; }
    public string Action { get; set; }
    public string [] Types { get; set; }
}
```

La operación HTTP GET recupera los datos del cliente del almacenamiento, construye un objeto `Customer` y, después, rellena la colección `Links`. El resultado tiene el formato de un mensaje de respuesta JSON. Cada vínculo consta de los siguientes campos:

* La relación entre el objeto que se devuelve y el objeto que describe el vínculo. En este caso "self" indica que el vínculo es una referencia al propio objeto (similar a un puntero `this` en muchos lenguajes orientados a objetos) y "orders" es el nombre de una colección que contiene la información de pedido relacionada.
* El hipervínculo (`Href`) del objeto que describe el vínculo tiene la forma de un URI.
* El tipo de solicitud HTTP (`Action`) que se puede enviar a este URI.
* El formato de los datos (`Types`) que deben facilitarse en la solicitud HTTP o que se pueden devolver en la respuesta, según el tipo de la solicitud.

Los vínculos de HATEOAS que se muestran en la respuesta HTTP de ejemplo indican que una aplicación cliente puede realizar las siguientes operaciones:

* Una solicitud HTTP GET al URI `http://adventure-works.com/customers/2` para capturar los detalles del cliente (nuevamente). Los datos se pueden devolver como XML o JSON.
* Una solicitud HTTP PUT al URI `http://adventure-works.com/customers/2` para modificar los detalles del cliente. Los nuevos datos se deben facilitar en el mensaje de solicitud en formato x-www-form-urlencoded.
* Una solicitud HTTP DELETE al URI `http://adventure-works.com/customers/2` para eliminar el cliente. La solicitud no espera ninguna información adicional ni datos devueltos en el cuerpo del mensaje de respuesta.
* Una solicitud HTTP GET al URI `http://adventure-works.com/customers/2/orders` para buscar todos los pedidos del cliente. Los datos se pueden devolver como XML o JSON.
* Una solicitud HTTP PUT al URI `http://adventure-works.com/customers/2/orders` para crear un nuevo pedido para este cliente. Los datos se deben facilitar en el mensaje de solicitud en formato x-www-form-urlencoded.

## <a name="handling-exceptions"></a>Control de excepciones

Tenga en cuenta lo siguiente si una operación produce una excepción no detectada.

### <a name="capture-exceptions-and-return-a-meaningful-response-to-clients"></a>Capture las excepciones y devuelva una respuesta significativa a los clientes.

El código que implementa una operación HTTP debe proporcionar un control total de las excepciones en lugar de permitir que las excepciones no detectadas se propaguen al marco. Si una excepción impide que se complete correctamente la operación, puede devolver la excepción en el mensaje de respuesta, pero debe incluir una descripción significativa del error que provocó la excepción. La excepción también debe incluir el código de estado HTTP adecuado, en lugar de simplemente devolver el código de estado 500 para todas las situaciones. Por ejemplo, si una solicitud de usuario provoca una actualización de la base de datos que infringe una restricción (por ejemplo, intenta eliminar un cliente que tiene pedidos pendientes), se debe devolver el código de estado 409 (conflicto) y un cuerpo de mensaje que indique el motivo del conflicto. Si alguna otra condición hace que la solicitud no se pueda procesar, puede devolver un código de estado 400 (solicitud incorrecta). Puede encontrar una lista completa de códigos de estado HTTP en la página [Status Code Definitions](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html) del sitio web de W3C.

El código de ejemplo intercepta diferentes condiciones y devuelve una respuesta adecuada.

```csharp
[HttpDelete]
[Route("customers/{id:int}")]
public IHttpActionResult DeleteCustomer(int id)
{
    try
    {
        // Find the customer to be deleted in the repository
        var customerToDelete = repository.GetCustomer(id);

        // If there is no such customer, return an error response
        // with status code 404 (Not Found)
        if (customerToDelete == null)
        {
                return NotFound();
        }

        // Remove the customer from the repository
        // The DeleteCustomer method returns true if the customer
        // was successfully deleted
        if (repository.DeleteCustomer(id))
        {
            // Return a response message with status code 204 (No Content)
            // To indicate that the operation was successful
            return StatusCode(HttpStatusCode.NoContent);
        }
        else
        {
            // Otherwise return a 400 (Bad Request) error response
            return BadRequest(Strings.CustomerNotDeleted);
        }
    }
    catch
    {
        // If an uncaught exception occurs, return an error response
        // with status code 500 (Internal Server Error)
        return InternalServerError();
    }
}
```

> [!TIP]
> No incluya información que pueda ser útil para un atacante que intenta entrar en su API.
  
Muchos servidores web interceptan condiciones de error por sí mismos antes de que lleguen a la API web. Por ejemplo, si configura la autenticación de un sitio web y el usuario no proporciona la información de autenticación correcta, el servidor web debe responder con código de estado 401 (no autorizado). Cuando un cliente se ha autenticado, el código puede realizar sus propias comprobaciones para asegurarse de que el cliente puede obtener acceso al recurso solicitado. Si se produce un error en esta autorización, debe devolver un código de estado 403 (prohibido).
 
### <a name="handle-exceptions-consistently-and-log-information-about-errors"></a>Controle las excepciones de una forma coherente y registre la información sobre los errores.

Para controlar las excepciones de una manera coherente, considere la posibilidad de implementar una estrategia global de control de errores para toda la API web. También debería incluir el registro de errores que captura los detalles completos de cada excepción; este registro de errores puede contener información detallada siempre y cuando los clientes a través de la web no tengan acceso a ella. 

### <a name="distinguish-between-client-side-errors-and-server-side-errors"></a>Distinga entre los errores del lado cliente y del lado servidor.

El protocolo HTTP distingue entre los errores que se producen debido a la aplicación de cliente (los códigos de estado HTTP 4xx) y los errores causados por un problema en el servidor (los códigos de estado HTTP 5xx). Asegúrese de que respeta esta convención en los mensajes de error de respuesta.

## <a name="optimizing-client-side-data-access"></a>Optimización del acceso a los datos en el lado cliente
En un entorno distribuido, como los que implican un servidor web y aplicaciones cliente, una de las principales fuentes de preocupación es la red. Puede actuar como un considerable cuello de botella, especialmente si una aplicación cliente envía solicitudes o recibe datos con frecuencia. Por lo tanto, se debe tratar de minimizar la cantidad de tráfico que fluye a través de la red. Tenga en cuenta los puntos siguientes cuando implemente el código para recuperar y mantener datos:

### <a name="support-client-side-caching"></a>Admita el almacenamiento en caché del lado cliente.

El protocolo HTTP 1.1 admite el almacenamiento en caché de los clientes y servidores intermedios a través de los cuales se enruta una solicitud mediante el uso del encabezado Cache-Control. Cuando una aplicación cliente envía una solicitud HTTP GET a la API web, la respuesta puede incluir un encabezado Cache-Control que indique si los datos en el cuerpo de la respuesta pueden almacenarse en caché de forma segura por parte del cliente o un servidor intermedio a través del cual se ha enrutado la solicitud, y cuánto tiempo debe pasar antes de que expiren y se consideren caducados. En el ejemplo siguiente se muestra una solicitud HTTP GET y la respuesta correspondiente que incluye un encabezado Cache-Control:

```HTTP
GET http://adventure-works.com/orders/2 HTTP/1.1
```

```HTTP
HTTP/1.1 200 OK
...
Cache-Control: max-age=600, private
Content-Type: text/json; charset=utf-8
Content-Length: ...
{"orderID":2,"productID":4,"quantity":2,"orderValue":10.00}
```

En este ejemplo, el encabezado Cache-Control especifica que los datos devueltos deben expirar a los 600 segundos, solo son adecuados para un solo cliente y no deben almacenarse en una memoria caché compartida que usen otros clientes (es *private*). El encabezado Cache-Control podría especificar *public* en lugar de *private*, en cuyo caso los datos pueden almacenarse en una memoria caché compartida, o puede especificar *no-store*, en cuyo caso el cliente **no** debe almacenar los datos en memoria caché. En el ejemplo de código siguiente se muestra cómo construir un encabezado Cache-Control en un mensaje de respuesta:

```csharp
public class OrdersController : ApiController
{
    ...
    [Route("api/orders/{id:int:min(0)}")]
    [HttpGet]
    public IHttpActionResult FindOrderByID(int id)
    {
        // Find the matching order
        Order order = ...;
        ...
        // Create a Cache-Control header for the response
        var cacheControlHeader = new CacheControlHeaderValue();
        cacheControlHeader.Private = true;
        cacheControlHeader.MaxAge = new TimeSpan(0, 10, 0);
        ...

        // Return a response message containing the order and the cache control header
        OkResultWithCaching<Order> response = new OkResultWithCaching<Order>(order, this)
        {
            CacheControlHeader = cacheControlHeader
        };
        return response;
    }
    ...
}
```

Este código usa una clase `IHttpActionResult` personalizada denominada `OkResultWithCaching`. Esta clase permite que el controlador establezca el contenido de encabezado de la memoria caché:

```csharp
public class OkResultWithCaching<T> : OkNegotiatedContentResult<T>
{
    public OkResultWithCaching(T content, ApiController controller)
        : base(content, controller) { }

    public OkResultWithCaching(T content, IContentNegotiator contentNegotiator, HttpRequestMessage request, IEnumerable<MediaTypeFormatter> formatters)
        : base(content, contentNegotiator, request, formatters) { }

    public CacheControlHeaderValue CacheControlHeader { get; set; }
    public EntityTagHeaderValue ETag { get; set; }

    public override async Task<HttpResponseMessage> ExecuteAsync(CancellationToken cancellationToken)
    {
        HttpResponseMessage response;
        try
        {
            response = await base.ExecuteAsync(cancellationToken);
            response.Headers.CacheControl = this.CacheControlHeader;
            response.Headers.ETag = ETag;
        }
        catch (OperationCanceledException)
        {
            response = new HttpResponseMessage(HttpStatusCode.Conflict) {ReasonPhrase = "Operation was cancelled"};
        }
        return response;
    }
}
```

> [!NOTE]
> El protocolo HTTP también define la directiva *no-cache* para el encabezado Cache-Control. De forma bastante confusa, esta directiva no significa "no almacenar en memoria caché", sino "volver a validar la información almacenada en caché con el servidor antes de devolverla"; los datos siguen pudiendo almacenarse en memoria caché, pero se comprueban cada vez que se usan para asegurar que sigan siendo actuales.
>
>

La administración en memoria caché es responsabilidad de la aplicación cliente o el servidor intermedio, pero si se implementa correctamente, puede ahorrar ancho de banda y mejorar el rendimiento, eliminando la necesidad de capturar los datos que ya se ha recuperado recientemente.

El valor de *max-age* en el encabezado Cache-Control es solo una guía y no una garantía de que los datos correspondientes no cambiarán durante el tiempo especificado. La API web debe establecer max-age en un valor adecuado según la volatilidad esperada de los datos. Cuando expire este período, el cliente debería descartar el objeto de la memoria caché.

> [!NOTE]
> La mayoría de los exploradores web modernos admiten el almacenamiento en memoria caché del lado cliente, agregando los encabezados cache-control adecuados a las solicitudes y examinando los encabezados de los resultados, como se describe. Sin embargo, algunos exploradores más antiguos no almacenarán en memoria caché los valores devueltos desde una dirección URL que incluya una cadena de consulta. Normalmente, esto no es un problema para las aplicaciones de cliente personalizadas que implementan su propia estrategia de administración de la memoria caché basada en el protocolo que se trata aquí.
>
> Algunos servidores proxy antiguos tienen el mismo comportamiento y podrían no almacenar en memoria caché las solicitudes basadas en direcciones URL con cadenas de consulta. Esto podría ser un problema para las aplicaciones cliente personalizadas que se conectan a un servidor web a través de un proxy de este tipo.
>

### <a name="provide-etags-to-optimize-query-processing"></a>Proporcione ETags para optimizar el procesamiento de las consultas.

Cuando una aplicación cliente recupera un objeto, el mensaje de respuesta también puede incluir una *ETag* (etiqueta de entidad). Una ETag es una cadena opaca que indica la versión de un recurso; cada vez que un recurso cambia, también se modifica la Etag. Esta ETag debe almacenarse en memoria caché como parte de los datos de la aplicación cliente. En el ejemplo de código siguiente se muestra cómo agregar una ETag como parte de la respuesta a una solicitud HTTP GET. Este código usa el método `GetHashCode` de un objeto para generar un valor numérico que identifica el objeto (puede reemplazar este método si es necesario y generar su propio hash con un algoritmo como MD5):

```csharp
public class OrdersController : ApiController
{
    ...
    public IHttpActionResult FindOrderByID(int id)
    {
        // Find the matching order
        Order order = ...;
        ...

        var hashedOrder = order.GetHashCode();
        string hashedOrderEtag = $"\"{hashedOrder}\"";
        var eTag = new EntityTagHeaderValue(hashedOrderEtag);

        // Return a response message containing the order and the cache control header
        OkResultWithCaching<Order> response = new OkResultWithCaching<Order>(order, this)
        {
            ...,
            ETag = eTag
        };
        return response;
    }
    ...
}
```

El mensaje de respuesta que publica la API web tiene este aspecto:

```HTTP
HTTP/1.1 200 OK
...
Cache-Control: max-age=600, private
Content-Type: text/json; charset=utf-8
ETag: "2147483648"
Content-Length: ...
{"orderID":2,"productID":4,"quantity":2,"orderValue":10.00}
```

> [!TIP]
> Por motivos de seguridad, no permita que los datos confidenciales o los que se devuelvan a través de una conexión autenticada (HTTPS) se almacenen en memoria caché.
>
>

Una aplicación cliente puede emitir una solicitud GET posterior para recuperar el mismo recurso en cualquier momento y, si ha cambiado el recurso (tiene una ETag diferente), debe descartarse la versión en caché y agregarse la nueva versión a la memoria caché. Si un recurso es grande y requiere una gran cantidad de ancho de banda para transmitirse de vuelta al cliente, las solicitudes repetidas para capturar los mismos datos pueden resultar ineficaces. Para combatir esta situación, el protocolo HTTP define el proceso siguiente para optimizar las solicitudes GET que se deben admitir en una API web:

* El cliente crea una solicitud GET que contiene la ETag de la versión actualmente en caché del recurso al que hace referencia en un encabezado If-None-Match HTTP:

    ```HTTP
    GET http://adventure-works.com/orders/2 HTTP/1.1
    If-None-Match: "2147483648"
    ```
* La operación GET de la API web obtiene la ETag actual de los datos solicitados (el pedido 2 en el ejemplo anterior) y la compara con el valor del encabezado If-None-Match.
* Si la ETag actual de los datos solicitados coincide con la que proporciona la solicitud, el recurso no ha cambiado y la API web debe devolver una respuesta HTTP con un cuerpo de mensaje vacío y un código de estado 304 (no modificado).
* Si la ETag actual de los datos solicitados no coincide con la que proporciona la solicitud, los datos han cambiado y la API web debe devolver una respuesta HTTP con los datos nuevos en el cuerpo del mensaje y un código de estado 200 (correcto).
* Si los datos solicitados ya no existen, la API web debe devolver una respuesta HTTP con el código de estado 404 (no encontrado).
* El cliente usa el código de estado para mantener la memoria caché. Si los datos no ha cambiado (código de estado 304), el objeto puede permanecer en la memoria caché y la aplicación cliente debe seguir usando esta versión del objeto. Si los datos han cambiado (código de estado 200), debe descartarse el objeto almacenado en caché e insertarse uno nuevo. Si los datos ya no están disponibles (código de estado 404), el objeto debe quitarse de la memoria caché.

> [!NOTE]
> Si el encabezado de la respuesta contiene el encabezado Cache-Control no-store, el objeto debe quitarse siempre de la memoria caché sin tener en cuenta el código de estado HTTP.
>

El código siguiente muestra el método `FindOrderByID` extendido para admitir el encabezado If-None-Match. Tenga en cuenta que si se omite el encabezado If-None-Match, siempre se recupera el pedido especificado:

```csharp
public class OrdersController : ApiController
{
    [Route("api/orders/{id:int:min(0)}")]
    [HttpGet]
    public IHttpActionResult FindOrderByID(int id)
    {
        try
        {
            // Find the matching order
            Order order = ...;

            // If there is no such order then return NotFound
            if (order == null)
            {
                return NotFound();
            }

            // Generate the ETag for the order
            var hashedOrder = order.GetHashCode();
            string hashedOrderEtag = $"\"{hashedOrder}\"";

            // Create the Cache-Control and ETag headers for the response
            IHttpActionResult response;
            var cacheControlHeader = new CacheControlHeaderValue();
            cacheControlHeader.Public = true;
            cacheControlHeader.MaxAge = new TimeSpan(0, 10, 0);
            var eTag = new EntityTagHeaderValue(hashedOrderEtag);

            // Retrieve the If-None-Match header from the request (if it exists)
            var nonMatchEtags = Request.Headers.IfNoneMatch;

            // If there is an ETag in the If-None-Match header and
            // this ETag matches that of the order just retrieved,
            // then create a Not Modified response message
            if (nonMatchEtags.Count > 0 &&
                String.CompareOrdinal(nonMatchEtags.First().Tag, hashedOrderEtag) == 0)
            {
                response = new EmptyResultWithCaching()
                {
                    StatusCode = HttpStatusCode.NotModified,
                    CacheControlHeader = cacheControlHeader,
                    ETag = eTag
                };
            }
            // Otherwise create a response message that contains the order details
            else
            {
                response = new OkResultWithCaching<Order>(order, this)
                {
                    CacheControlHeader = cacheControlHeader,
                    ETag = eTag
                };
            }

            return response;
        }
        catch
        {
            return InternalServerError();
        }
    }
...
}
```

Este ejemplo incorpora una clase `IHttpActionResult` personalizada adicional denominada `EmptyResultWithCaching`. Esta clase solo actúa como un contenedor alrededor de un objeto `HttpResponseMessage` que no contiene un cuerpo de respuesta:

```csharp
public class EmptyResultWithCaching : IHttpActionResult
{
    public CacheControlHeaderValue CacheControlHeader { get; set; }
    public EntityTagHeaderValue ETag { get; set; }
    public HttpStatusCode StatusCode { get; set; }
    public Uri Location { get; set; }

    public async Task<HttpResponseMessage> ExecuteAsync(CancellationToken cancellationToken)
    {
        HttpResponseMessage response = new HttpResponseMessage(StatusCode);
        response.Headers.CacheControl = this.CacheControlHeader;
        response.Headers.ETag = this.ETag;
        response.Headers.Location = this.Location;
        return response;
    }
}
```

> [!TIP]
> En este ejemplo, la ETag para los datos se genera mediante la aplicación de un algoritmo hash a los datos recuperados del origen de datos subyacente. Si la ETag se puede calcular de alguna otra manera, es posible optimizar aún más el proceso y los datos solo deben capturarse desde el origen de datos si han cambiado.  Este enfoque es especialmente útil si los datos son grandes o el acceso al origen de datos puede provocar una latencia significativa (por ejemplo, si el origen de datos es una base de datos remota).
>

### <a name="use-etags-to-support-optimistic-concurrency"></a>Use ETags para admitir la simultaneidad optimista.

Para habilitar las actualizaciones en datos anteriormente almacenados en caché, el protocolo HTTP admite una estrategia de simultaneidad optimista. Si, después de capturar y almacenar en caché un recurso, la aplicación cliente envía posteriormente una solicitud PUT o DELETE para cambiar o quitar el recurso, debe incluir un encabezado If-Match que haga referencia a la ETag. La API web puede entonces usar esta información para determinar si otro usuario ha cambiado ya el recurso desde que se recuperó y enviar una respuesta adecuada de vuelta a la aplicación cliente, como se indica a continuación:

* El cliente crea una solicitud PUT que contiene los nuevos detalles para el recurso y la ETag de la versión actualmente en caché del recurso al que hace referencia en un encabezado If-Match HTTP: En el ejemplo siguiente se muestra una solicitud PUT que actualiza un pedido:

    ```HTTP
    PUT http://adventure-works.com/orders/1 HTTP/1.1
    If-Match: "2282343857"
    Content-Type: application/x-www-form-urlencoded
    Content-Length: ...
    productID=3&quantity=5&orderValue=250
    ```
* La operación PUT de la API web obtiene la ETag actual de los datos solicitados (el pedido 1 en el ejemplo anterior) y la compara con el valor del encabezado If-Match.
* Si la ETag actual de los datos solicitados coincide con la que proporciona la solicitud, el recurso no ha cambiado y la API web debe realizar la actualización y devolver un mensaje con el código de estado HTTP 204 (sin contenido). La respuesta puede incluir los encabezados Cache-Control y ETag de la versión actualizada del recurso. La respuesta siempre debe incluir el encabezado Location que hace referencia al URI del recurso recién actualizado.
* Si la ETag actual de los datos solicitados no coincide con la que proporciona la solicitud, otro usuario ha cambiado los datos desde que se capturaron y la API web debe devolver una respuesta HTTP con un cuerpo de mensaje vacío y un código de estado 412 (error de condición previa).
* Si el recurso que se va a actualizar ya no existe, la API web debe devolver una respuesta HTTP con el código de estado 404 (no encontrado).
* El cliente usa el código de estado y los encabezados de las respuestas para mantener la memoria caché. Si los datos se han actualizado (código de estado 204), el objeto puede permanecer en la memoria caché (siempre y cuando el encabezado Cache-Control no especifique no-store), pero debe actualizarse la ETag. Si otro usuario ha cambiado los datos (código de estado 412) o no se encuentran (código de estado 404), el objeto en caché debe descartarse.

En el ejemplo de código siguiente se muestra una implementación de la operación PUT para el controlador Orders:

```csharp
public class OrdersController : ApiController
{
    [HttpPut]
    [Route("api/orders/{id:int}")]
    public IHttpActionResult UpdateExistingOrder(int id, DTOOrder order)
    {
        try
        {
            var baseUri = Constants.GetUriFromConfig();
            var orderToUpdate = this.ordersRepository.GetOrder(id);
            if (orderToUpdate == null)
            {
                return NotFound();
            }

            var hashedOrder = orderToUpdate.GetHashCode();
            string hashedOrderEtag = $"\"{hashedOrder}\"";

            // Retrieve the If-Match header from the request (if it exists)
            var matchEtags = Request.Headers.IfMatch;

            // If there is an Etag in the If-Match header and
            // this etag matches that of the order just retrieved,
            // or if there is no etag, then update the Order
            if (((matchEtags.Count > 0 &&
                String.CompareOrdinal(matchEtags.First().Tag, hashedOrderEtag) == 0)) ||
                matchEtags.Count == 0)
            {
                // Modify the order
                orderToUpdate.OrderValue = order.OrderValue;
                orderToUpdate.ProductID = order.ProductID;
                orderToUpdate.Quantity = order.Quantity;

                // Save the order back to the data store
                // ...

                // Create the No Content response with Cache-Control, ETag, and Location headers
                var cacheControlHeader = new CacheControlHeaderValue();
                cacheControlHeader.Private = true;
                cacheControlHeader.MaxAge = new TimeSpan(0, 10, 0);

                hashedOrder = order.GetHashCode();
                hashedOrderEtag = $"\"{hashedOrder}\"";
                var eTag = new EntityTagHeaderValue(hashedOrderEtag);

                var location = new Uri($"{baseUri}/{Constants.ORDERS}/{id}");
                var response = new EmptyResultWithCaching()
                {
                    StatusCode = HttpStatusCode.NoContent,
                    CacheControlHeader = cacheControlHeader,
                    ETag = eTag,
                    Location = location
                };

                return response;
            }

            // Otherwise return a Precondition Failed response
            return StatusCode(HttpStatusCode.PreconditionFailed);
        }
        catch
        {
            return InternalServerError();
        }
    }
    ...
}
```

> [!TIP]
> El uso del encabezado If-Match es totalmente opcional y, si se omite, la API web siempre intentará actualizar el pedido especificado, posiblemente sobrescribiendo ciegamente una actualización que otro usuario haya realizado. Para evitar problemas debidos a actualizaciones perdidas, facilite siempre un encabezado If-Match.
>
>

## <a name="handling-large-requests-and-responses"></a>Administración de respuestas y solicitudes de gran tamaño
Puede haber ocasiones en que una aplicación cliente necesite emitir solicitudes que envíen o reciban datos que pueden tener un tamaño de varios megabytes (o mayor). Si se espera mientras se transmite esa cantidad de datos, podría provocar que la aplicación cliente deje de responder. Tenga en cuenta los siguientes puntos cuando necesite controlar las solicitudes que incluyen grandes cantidades de datos:

### <a name="optimize-requests-and-responses-that-involve-large-objects"></a>Optimice las solicitudes y respuestas que impliquen objetos grandes.

Algunos recursos pueden ser objetos grandes o incluir campos de gran tamaño, como imágenes gráficas u otros tipos de datos binarios. Una API web debe admitir streaming para habilitar la carga y descarga optimizadas de estos recursos.

El protocolo HTTP ofrece el mecanismo de codificación de transferencia fragmentada para hacer streaming de objetos de datos de gran tamaño a un cliente. Cuando el cliente envía una solicitud HTTP GET para un objeto grande, la API web puede enviar de vuelta la respuesta por *fragmentos* de forma gradual a través de una conexión HTTP. Es posible que la longitud de los datos de la respuesta no se conozca inicialmente (se puede generar), por lo que el servidor que hospeda la API web debe enviar un mensaje de respuesta con cada fragmento que especifique el encabezado Transfer-Encoding: Chunked en lugar de Content-Length. La aplicación cliente puede a su vez recibir cada fragmento para construir la respuesta completa. La transferencia de datos se completa cuando el servidor devuelve un fragmento final con tamaño cero. 

Una única solicitud podrían producir un objeto masivo que consume gran cantidad de recursos. Si durante el proceso de streaming la API web determina que la cantidad de datos de una solicitud ha superado algunos límites aceptables, puede anular la operación y devolver un mensaje de respuesta con el código de estado 413 (entidad solicitada demasiado grande).

Puede minimizar el tamaño de los objetos grandes transmitidos por la red con la compresión HTTP. Este enfoque ayuda a reducir la cantidad de tráfico de red y la latencia de red asociada, pero a costa de requerir un procesamiento adicional en el cliente y en el servidor que hospeda la API web. Por ejemplo, una aplicación cliente que espera recibir datos comprimidos puede incluir un encabezado de solicitud Accept-Encoding: gzip (también se pueden especificar otros algoritmos de compresión de datos). Si el servidor admite la compresión, debe responder con el contenido almacenado en formato gzip en el cuerpo del mensaje y el encabezado de respuesta Content-Encoding: gzip.

Puede combinar la compresión codificada con streaming; comprima los datos antes de hacer streaming y especifique la codificación de contenido gzip y la codificación de transferencia fragmentada en los encabezados de los mensajes. Tenga en cuenta también que algunos servidores web (por ejemplo, Internet Information Server) pueden configurarse para comprimir automáticamente las respuestas HTTP sin tener en cuenta si la API web comprime los datos o no.

### <a name="implement-partial-responses-for-clients-that-do-not-support-asynchronous-operations"></a>Implemente respuestas parciales para los clientes que no admitan operaciones asincrónicas.

Como alternativa al streaming asincrónico, una aplicación cliente puede solicitar explícitamente los datos de objetos grandes en fragmentos, que se conocen como respuestas parciales. La aplicación cliente envía una solicitud HTTP HEAD para obtener información sobre el objeto. Si la API web admite respuestas parciales, debe responder a la solicitud HEAD con un mensaje de respuesta que contenga un encabezado Accept-Ranges y un encabezado Content-Length que indique el tamaño total del objeto, pero el cuerpo del mensaje debe estar vacío. La aplicación cliente puede usar esta información para construir una serie de solicitudes GET que especifiquen un intervalo de bytes que se recibirán. La API web debería devolver un mensaje de respuesta con el código de estado HTTP 206 (contenido parcial), un encabezado Content-Length que especifique la cantidad de datos incluidos en el cuerpo del mensaje de respuesta y un encabezado Content-Range que indique la parte (por ejemplo, los bytes del 4000 al 8000) del objeto que representan esos datos.

Las respuestas parciales y solicitudes HTTP HEAD se describen con más detalle en [Diseño de API][api-design].

### <a name="avoid-sending-unnecessary-100-continue-status-messages-in-client-applications"></a>Evite enviar mensajes de estado 100-Continuar innecesarios en las aplicaciones cliente.

Una aplicación de cliente que vaya a enviar una gran cantidad de datos a un servidor puede determinar primero si el servidor está realmente dispuesto a aceptar la solicitud. Antes de enviar los datos, la aplicación cliente puede enviar una solicitud HTTP con un encabezado Expect: 100-Continue, un encabezado Content-Length que indique el tamaño de los datos y un cuerpo de mensaje vacío. Si el servidor está dispuesto a atender la solicitud, debe responder con un mensaje que especifique el código de estado HTTP 100 (continuar). La aplicación cliente puede entonces continuar y enviar la solicitud completa, incluidos los datos en el cuerpo del mensaje.

Si hospeda un servicio mediante IIS, el controlador HTTP.sys detecta y controla automáticamente los encabezados Expect: 100-Continue antes de pasar las solicitudes a la aplicación web. Esto significa que no es probable que vea estos encabezados en el código de la aplicación y se puede suponer que IIS ya ha filtrado los mensajes que considera como no aptos o demasiado grandes.

Si está creando aplicaciones cliente mediante .NET Framework, todos los mensajes POST y PUT enviarán primero mensajes con encabezados Expect: 100-Continue de forma predeterminada. Al igual que con el lado servidor, el proceso lo controla de forma transparente .NET Framework. Sin embargo, este proceso hace que cada solicitud POST y PUT necesite dos envíos de ida y vuelta al servidor, incluso para las solicitudes pequeñas. Si la aplicación no envía solicitudes con grandes cantidades de datos, puede deshabilitar esta característica mediante la clase `ServicePointManager` para crear objetos `ServicePoint` en la aplicación cliente. Un objeto `ServicePoint` controla las conexiones que el cliente realiza en un servidor basado en el esquema y los fragmentos de host de los URI que identifican recursos en el servidor. Después, puede establecer la propiedad `Expect100Continue` del objeto `ServicePoint` en false. Todas las solicitudes POST y PUT posteriores que realice el cliente a través de un URI que coincida con el esquema y los fragmentos de host del objeto `ServicePoint` se enviarán sin encabezados Expect: 100-Continue. El código siguiente muestra cómo configurar un objeto `ServicePoint` que configura todas las solicitudes enviadas a los URI con un esquema de `http` y un host de `www.contoso.com`.

```csharp
Uri uri = new Uri("http://www.contoso.com/");
ServicePoint sp = ServicePointManager.FindServicePoint(uri);
sp.Expect100Continue = false;
```

También puede establecer la propiedad estática `Expect100Continue` de la clase `ServicePointManager` para especificar el valor predeterminado de esta propiedad para todos los objetos `ServicePoint` creados posteriormente. Para más información, consulte [ServicePoint (clase)](https://msdn.microsoft.com/library/system.net.servicepoint.aspx).

### <a name="support-pagination-for-requests-that-may-return-large-numbers-of-objects"></a>Admita la paginación de las solicitudes que pueden devolver grandes cantidades de objetos.

Si una colección contiene un gran número de recursos, al emitir una solicitud GET en el URI correspondiente podría producirse un procesamiento importante en el servidor que hospeda la API web, que afectaría al rendimiento y generaría una cantidad significativa de tráfico de red, lo que provocaría una mayor latencia.

Para controlar estos casos, la API web debe admitir las cadenas de consulta que permiten que la aplicación cliente refina las solicitudes o capture los datos en bloques (o páginas) discretos más fáciles de administrar. El código siguiente muestra el método `GetAllOrders` en el controlador `Orders`. Este método recupera los detalles de los pedidos. Si este método no tenía restricciones, posiblemente podría devolver una gran cantidad de datos. Los parámetros `limit` y `offset` están diseñados para reducir el volumen de datos a un subconjunto más pequeño, en este caso solo los 10 primeros pedidos de forma predeterminada:

```csharp
public class OrdersController : ApiController
{
    ...
    [Route("api/orders")]
    [HttpGet]
    public IEnumerable<Order> GetAllOrders(int limit=10, int offset=0)
    {
        // Find the number of orders specified by the limit parameter
        // starting with the order specified by the offset parameter
        var orders = ...
        return orders;
    }
    ...
}
```

Una aplicación cliente puede emitir una solicitud para recuperar 30 pedidos empezando con un desplazamiento de 50 mediante el URI `http://www.adventure-works.com/api/orders?limit=30&offset=50`.

> [!TIP]
> Evite permitir que las aplicaciones de cliente especifiquen las cadenas de consulta que resultan en un URI que tenga más de 2000 caracteres. Muchos servidores y clientes web no pueden controlar los URI así de largos.
>
>

## <a name="maintaining-responsiveness-scalability-and-availability"></a>Mantenimiento de la capacidad de respuesta, la escalabilidad y la disponibilidad
La misma API web pueden usarla muchas aplicaciones cliente que se ejecuten en cualquier lugar del mundo. Es importante asegurarse de que la API web se implementa para mantener la capacidad de respuesta bajo cargas intensas, para que se pueda escalar a fin de admitir una carga de trabajo que varíe mucho y para garantizar la disponibilidad para los clientes que realizan operaciones esenciales para el negocio. Tenga en cuenta los puntos siguientes al determinar cómo cumplir estos requisitos:

### <a name="provide-asynchronous-support-for-long-running-requests"></a>Ofrezca compatibilidad asincrónica para las solicitudes de ejecución prolongada.

Una solicitud que podría tardar mucho tiempo en procesarse debe realizarse sin bloquear al cliente que la envió. La API web puede realizar ciertas comprobaciones iniciales para validar la solicitud, iniciar una tarea independiente para realizar el trabajo y, después, devolver un mensaje de respuesta con el código HTTP 202 (aceptado). La tarea puede ejecutarse de forma asincrónica como parte del procesamiento de la API, o puede descargarse en una tarea en segundo plano.

La API web también debe proporcionar un mecanismo para devolver los resultados del procesamiento a la aplicación cliente. Puede lograr esto si proporciona un mecanismo de sondeo de aplicaciones cliente para consultar periódicamente si ha finalizado el procesamiento y se ha obtenido el resultado o si habilita la API para que envíe una notificación cuando la operación se haya completado.

Puede implementar un mecanismo de sondeo sencillo si ofrece una URI *polling* que actúe como un recurso virtual con el siguiente enfoque:

1. La aplicación cliente envía la solicitud inicial a la API web.
2. La API web almacena información sobre la solicitud en una tabla que se conserva en Table Storage o en Microsoft Azure Cache y genera una clave única para esta entrada, posiblemente en forma de un GUID.
3. La API web inicia el procesamiento como una tarea independiente. La API web registra el estado de la tarea en la tabla como *Running*.
4. La API web devuelve un mensaje de respuesta con el código de estado HTTP 202 (aceptado) y el GUID de la entrada de la tabla en el cuerpo del mensaje.
5. Cuando se completa la tarea, la API web almacena los resultados en la tabla y establece el estado de la tarea en *Complete*. Tenga en cuenta que si se produce un error en la tarea, la API web también podría almacenar información sobre el error y establecer el estado en *Failed*.
6. Mientras se ejecuta la tarea, el cliente puede continuar realizando su propio procesamiento. Puede enviar periódicamente una solicitud al URI */polling/{guid}* donde *{guid}* es el GUID devuelto en el mensaje de respuesta 202 de la API web.
7. La API web en el URI */polling/{guid}* consulta el estado de la tarea correspondiente en la tabla y devuelve un mensaje de respuesta con el código de estado HTTP 200 (correcto) que contiene este estado (*Running*, *Complete* o *Failed*). Si la tarea se ha completado o ha producido un error, el mensaje de respuesta también puede incluir los resultados del procesamiento o la información disponible sobre el motivo del error.

Algunas opciones para implementar notificaciones son:

- El uso de un Centro de notificaciones de Azure para insertar respuestas asíncronas en las aplicaciones cliente. Para más información, consulte [Azure Notification Hubs notifica a los usuarios](/azure/notification-hubs/notification-hubs-aspnet-backend-windows-dotnet-wns-notification/).
- El uso del modelo Comet para conservar una conexión de red persistente entre el cliente y el servidor que hospeda la API web y el uso de esta conexión para insertar mensajes desde el servidor de nuevo en el cliente. El artículo de MSDN magazine [Creación de una aplicación Comet sencilla en Microsoft .NET Framework](https://msdn.microsoft.com/magazine/jj891053.aspx) describe una solución de ejemplo.
- El uso de SignalR para insertar datos en tiempo real desde el servidor web en el cliente a través de una conexión de red persistente. SignalR está disponible para las aplicaciones web de ASP.NET como un paquete de NuGet. Puede encontrar más información en el sitio web de [ASP.NET SignalR](http://signalr.net/) .

### <a name="ensure-that-each-request-is-stateless"></a>Asegúrese de que ninguna de las solicitudes tenga estado.

Cada solicitud debe considerarse atómica. No debería haber dependencias entre una solicitud que realiza una aplicación de cliente y las siguientes solicitudes que envía el mismo cliente. Este enfoque ayuda a la escalabilidad; las instancias del servicio web se pueden implementar en varios servidores. Las solicitudes de cliente se pueden dirigir a cualquiera de estas instancias y los resultados siempre deben ser los mismos. También mejora la disponibilidad por un motivo similar; si se produce un error en un servidor web, las solicitudes se pueden enrutar a otra instancia (mediante el Administrador de tráfico de Azure) mientras se reinicia el servidor, sin que se produzcan efectos negativos en las aplicaciones cliente.

### <a name="track-clients-and-implement-throttling-to-reduce-the-chances-of-dos-attacks"></a>Realice un seguimiento de los clientes e implemente la limitación para reducir las posibilidades de ataques de denegación de servicio.

Si un cliente específico realiza un gran número de solicitudes en un período determinado de tiempo, podría monopolizar el servicio y afectar al rendimiento de otros clientes. Para mitigar este problema, una API web puede supervisar las llamadas de las aplicaciones de cliente mediante el seguimiento de la dirección IP de todas las solicitudes entrantes o mediante el registro de cada acceso autenticado. Puede usar esta información para limitar el acceso a los recursos. Si un cliente supera un límite definido, la API web puede devolver un mensaje de respuesta con el estado 503 (servicio no disponible) e incluir un encabezado Retry-After que especifique cuándo puede enviar el cliente la siguiente solicitud sin que se rechace. Esta estrategia puede ayudarle a reducir las posibilidades de un ataque de denegación de servicio (DOS) de un conjunto de clientes que quieran detener el sistema.

### <a name="manage-persistent-http-connections-carefully"></a>Administre con cuidado las conexiones HTTP persistentes.

El protocolo HTTP admite conexiones HTTP persistentes si están disponibles. La especificación HTTP 1.0 agrega el encabezado Connection:Keep-Alive, que permite que una aplicación cliente indique al servidor que puede usar la misma conexión para enviar solicitudes posteriores, en lugar de abrir nuevas. La conexión se cierra automáticamente si el cliente no vuelve a utilizarla en un período definido por el host. Este comportamiento es el predeterminado en HTTP 1.1 como lo usan los servicios de Azure, por lo que no es necesario incluir encabezados Keep-Alive en los mensajes.

Si se mantiene abierta una conexión, puede ayudar a mejorar la capacidad de respuesta al reducir la latencia y la congestión de la red, pero puede ser perjudicial para la escalabilidad porque se mantienen conexiones innecesarias abiertas durante más tiempo del necesario, lo que limita la capacidad de conexión de otros clientes simultáneos. También puede afectar a la duración de la batería si la aplicación cliente se ejecuta en un dispositivo móvil; si la aplicación solo realiza solicitudes ocasionales al servidor, mantener abierta una conexión puede hacer que la batería se gaste con mayor rapidez. Para asegurarse de que una conexión no se hace persistente con HTTP 1.1, el cliente puede incluir un encabezado Connection:Close con los mensajes para reemplazar el comportamiento predeterminado. De forma similar, si un servidor está controlando un gran número de clientes, puede incluir un encabezado Connection:Close en los mensajes de respuesta, lo que debería cerrar la conexión y ahorrar recursos del servidor.

> [!NOTE]
> Las conexiones HTTP persistentes son una característica puramente opcional para reducir la sobrecarga de la red asociada al restablecimiento repetido de un canal de comunicaciones. Ni la API web ni la aplicación cliente deben depender de que haya una conexión HTTP persistente disponible. No use conexiones HTTP persistentes para implementar sistemas de notificación de estilo Comet; en su lugar, debería usar sockets (o websockets si están disponible) en la capa TCP. Por último, tenga en cuenta que los encabezados Keep-Alive son de uso limitado si una aplicación cliente se comunica con un servidor mediante un proxy; solo la conexión con el cliente y el proxy será persistente.
>
>

## <a name="publishing-and-managing-a-web-api"></a>Publicación y administración de una API web
Para que una API web esté disponible para las aplicaciones cliente, la API web debe implementarse en un entorno de host. Normalmente este entorno es un servidor web, aunque puede ser otro tipo de proceso de host. Debe considerar los siguientes puntos cuando publique una API web:

* Todas las solicitudes deben autenticarse y autorizarse, y debe aplicarse el nivel de control de acceso adecuado.
* Una API web comercial puede estar sujeta a diversas garantías de calidad relativas a los tiempos de respuesta. Es importante asegurarse de que ese entorno de host es escalable si la carga puede variar considerablemente con el tiempo.
* Puede ser necesario realizar mediciones de las solicitudes para fines de monetización.
* Es posible que sea necesario regular el flujo de tráfico a la API web e implementar la limitación para clientes concretos que hayan agotado sus cuotas.
* Los requisitos normativos podrían requerir un registro y una auditoría de todas las solicitudes y respuestas.
* Para garantizar la disponibilidad, puede ser necesario supervisar el estado del servidor que hospeda la API web y reiniciarlo si hiciera falta.

Es útil poder separar estos problemas de los problemas técnicos relativos a la implementación de la API web. Por este motivo, considere la creación de una [fachada](http://en.wikipedia.org/wiki/Facade_pattern),que se ejecute como un proceso independiente y que enrute las solicitudes a la API web. La fachada puede proporcionar las operaciones de administración y redirigir las solicitudes validadas a la API web. El uso de una fachada también puede aportar muchas ventajas funcionales, entre las que se incluyen:

* Actuar como un punto de integración para varias API web.
* Transformar los mensajes y traducir protocolos de comunicaciones de los clientes creados mediante tecnologías distintas.
* Almacenar en caché solicitudes y respuestas para reducir la carga en el servidor que hospeda la API web.

## <a name="testing-a-web-api"></a>Prueba de una API web
Una API web debe probarse tan exhaustivamente como cualquier otro producto de software. Considere la posibilidad de crear pruebas unitarias para validar la funcionalidad de la naturaleza de la API web. Por su naturaleza, una API web tiene sus propios requisitos adicionales para comprobar que funciona correctamente. Debe prestar especial atención a los siguientes aspectos:

* Pruebe todas las rutas para comprobar que invocan las operaciones correctas. Preste especial atención al código de estado HTTP 405 (método no permitido) que se devuelven de forma inesperada, ya que esto puede indicar una desigualdad entre una ruta y los métodos HTTP (GET, POST, PUT, DELETE) que se pueden enviar a esa ruta.

    Envíe solicitudes HTTP a las rutas que no las admitan, como el envío de una solicitud POST a un recurso concreto (las solicitudes POST solo se pueden enviar a las colecciones de recursos). En estos casos, la única respuesta válida *debería* ser el código de estado 405 (no permitido).
* Compruebe que todas las rutas estén protegidas correctamente y que estén sujetas a las comprobaciones de autenticación y autorización apropiadas.

  > [!NOTE]
  > Algunos aspectos de seguridad, como la autenticación de los usuarios, tienen más probabilidades de ser responsabilidad del entorno de host y no de la API web, pero sigue siendo necesario incluir las pruebas de seguridad como parte del proceso de implementación.
  >
  >
* Pruebe el control de excepciones que realiza cada operación y compruebe que se pasa una respuesta HTTP adecuada y significativa de vuelta a la aplicación cliente.
* Compruebe que los mensajes de solicitud y respuesta están formados correctamente. Por ejemplo, si una solicitud HTTP POST contiene los datos de un recurso nuevo en formato x-www-form-urlencoded, confirme que la operación correspondiente analiza correctamente los datos, crea los recursos y devuelve una respuesta que contiene los detalles del nuevo recurso, incluido el encabezado Location correcto.
* Compruebe todos los vínculos y los URI de los mensajes de respuesta. Por ejemplo, un mensaje HTTP POST debe devolver el URI del recurso recién creado. Todos los vínculos HATEOAS deben ser válidos.

* Asegúrese de que cada operación devuelve los códigos de estado correctos para diferentes combinaciones de entradas. Por ejemplo: 

  * Si una consulta es correcta, debe devolver el código de estado 200 (correcto).
  * Si no se encuentra un recurso, la operación debe devolver el código de estado HTTP 404 (no encontrado).
  * Si el cliente envía una solicitud que elimina correctamente un recurso, el código de estado debe ser 204 (sin contenido).
  * Si el cliente envía una solicitud que crea un nuevo recurso, el código de estado debe ser 201 (creado).

Tenga cuidado con los códigos de estado de respuestas inesperados en el intervalo 5xx. Normalmente, el servidor host informa de estos mensajes para indicar que no pudo completar una solicitud válida.

* Pruebe las distintas combinaciones de encabezados de solicitud que una aplicación cliente puede especificar y asegúrese de que la API web devuelve la información esperada en los mensajes de respuesta.
* Pruebe las cadenas de consultas. Si una operación puede tomar parámetros opcionales (por ejemplo, las solicitudes de paginación), pruebe las diferentes combinaciones y ordenaciones de los parámetros.
* Compruebe que las operaciones asincrónicas se completan correctamente. Si la API web admite streaming para las solicitudes que devuelven objetos binarios de gran tamaño (como vídeo o audio), asegúrese de que las solicitudes de cliente no se bloquean mientras se transmiten los datos. Si la API web implementa sondeo para las operaciones de modificación de datos de ejecución prolongada, compruebe que las operaciones informan de su estado a medida que se procesan.

También debería crear y ejecutar pruebas de rendimiento para comprobar que la API web funciona satisfactoriamente bajo presión. Puede crear un proyecto de pruebas de carga y rendimiento web con Visual Studio Ultimate. Para más información, consulte [Ejecutar pruebas de rendimiento en la aplicación](https://msdn.microsoft.com/library/dn250793.aspx).

## <a name="using-azure-api-management"></a>Uso de Azure API Management 

En Azure, considere la posibilidad de usar [Azue API Management](https://azure.microsoft.com/documentation/services/api-management/) para publicar y administrar una API web. Con esta utilidad, puede generar un servicio que actúe como una fachada para una o varias API web. El servicio es en sí mismo un servicio web escalable que puede crear y configurar mediante el Portal de administración de Azure. Puede utilizar este servicio para publicar y administrar una API web como se indica a continuación:

1. Implemente la API web en un sitio web, un servicio en la nube de Azure o una máquina virtual de Azure.
2. Conecte el servicio de administración de API a la API web. Las solicitudes enviadas a la dirección URL de la API de administración se asignan a URI de la API web. El mismo servicio de administración de API puede enrutar las solicitudes a más de una API web. Esto le permite agregar varias API web en un servicio de administración único. De forma similar, es posible hacer referencia a la misma API web desde más de un servicio de administración de API si necesita restringir o hacer particiones de la funcionalidad disponible para distintas aplicaciones.

   > [!NOTE]
   > Los URI de vínculos HATEOAS generados como parte de la respuesta de solicitudes HTTP GET deben hacer referencia a la dirección URL del servicio de administración de API y no al servidor web que hospeda la API web.
   >
   >
3. Para cada API web, especifique las operaciones HTTP que expone la API web junto con los parámetros opcionales que una operación puede tomar como entrada. También puede configurar si el servicio de administración de API debe almacenar en caché la respuesta recibida de la API web para optimizar las solicitudes repetidas para los mismos datos. Registre los detalles de las respuestas HTTP que puede generar cada operación. Esta información se usa para generar la documentación para desarrolladores, por lo que es importante que sea precisa y completa.

   Puede definir operaciones manualmente mediante los asistentes que se ofrecen en el Portal de administración de Azure o puede importarlas desde un archivo que contenga las definiciones en formato WADL o Swagger.
4. Establezca la configuración de seguridad para las comunicaciones entre el servicio de administración de API y el servidor web que hospeda la API web. El servicio de administración de API actualmente admite la autenticación básica y mutua mediante certificados y la autorización de usuario de OAuth 2.0.
5. Cree un producto. Un producto es la unidad de publicación; agregue las API web que había conectado previamente al servicio de administración al producto. Cuando se publica el producto, las API web están disponibles para los desarrolladores.

   > [!NOTE]
   > Antes de publicar un producto, también puede definir grupos de usuarios que pueden tener acceso al producto y agregar usuarios a esos grupos. Esto le da control sobre los desarrolladores y las aplicaciones que puede usar la API web. Si una API web está sujeta a aprobación, antes de poder tener acceso a ella, un desarrollador debe enviar una solicitud al administrador del producto. El administrador puede conceder o denegar el acceso al desarrollador. Los desarrolladores existentes también se pueden bloquear si cambian las circunstancias.
   >
   >
6. Configure directivas para cada API web. Las directivas rigen aspectos como si se deben permitir las llamadas entre dominios, cómo autenticar los clientes, si se debe hacer una conversión entre los formatos de datos XML y JSON de forma transparente, si se quieren restringir las llamadas de un determinado intervalo de IP, las cuotas de uso y si se quiere limitar la tasa de llamadas. Las directivas pueden aplicarse globalmente a todo el producto, a una API web única de un producto o a operaciones individuales de una API web.

Para más información, consulte la [documentación de API Management](/azure/api-management/). 

> [!TIP]
> Azure ofrece el Administrador de tráfico de Azure que le permite implementar la conmutación por error y equilibrio de carga y reducir la latencia en varias instancias de un sitio web hospedado en distintas ubicaciones geográficas. Puede usar el Administrador de tráfico de Azure junto con el servicio de administración de API, que puede enrutar solicitudes a instancias de un sitio web a través del Administrador de tráfico de Azure.  Para más información, consulte [Métodos de enrutamiento de Traffic Manager](/azure/traffic-manager/traffic-manager-routing-methods/).
>
> En esta estructura, si está utilizando nombres DNS personalizados para sus sitios web, debe configurar el registro CNAME adecuado para que cada sitio web apunte al nombre DNS del sitio web del Administrador de tráfico de Azure.
>

## <a name="supporting-client-side-developers"></a>Compatibilidad con desarrolladores en el lado cliente
Los desarrolladores que crean aplicaciones cliente normalmente necesitan información sobre cómo obtener acceso a la API web y a la documentación relativa a los parámetros, los tipos de datos, los tipos de valores devueltos y los códigos de retorno que describen las distintas solicitudes y respuestas entre el servicio web y la aplicación cliente.

### <a name="document-the-rest-operations-for-a-web-api"></a>Documentación de las operaciones REST para una API web
El servicio de administración de API de Azure incluye un portal para desarrolladores que describe las operaciones REST que expone una API web. Cuando un producto se ha publicado, aparece en este portal. Los desarrolladores pueden usar este portal para registrarse y obtener acceso; el administrador puede después aprobar o denegar la solicitud. Si se aprueba el desarrollador, se le asigna una clave de suscripción que se usa para autenticar las llamadas de las aplicaciones cliente que desarrolla. Esta clave debe facilitarse con cada llamada de API web; en caso contrario, se rechazará.

Este portal también ofrece:

* Documentación del producto, donde se enumeran las operaciones que expone, los parámetros necesarios y las distintas respuestas que se pueden devolver. Tenga en cuenta que esta información se genera a partir de los detalles proporcionados en el paso 3 de la lista en la sección Publicación de una API web mediante el servicio de Microsoft Azure API Management.
* Fragmentos de código que muestran cómo invocar operaciones de varios lenguajes, incluidos JavaScript, C#, Java, Ruby, Python y PHP.
* Una consola para desarrolladores que permite que un desarrollador envíe una solicitud HTTP para probar cada una de las operaciones del producto y ver los resultados.
* Una página donde el desarrollador puede informar de los problemas encontrados.

El Portal de administración de Azure le permite personalizar el portal para desarrolladores para cambiar el estilo y el diseño, de forma que coincida con la marca de su organización.

### <a name="implement-a-client-sdk"></a>Implementación de un SDK de cliente
La creación de una aplicación cliente que invoque solicitudes REST para tener acceso a una API web requiere que se escriba una cantidad significativa de código para construir cada solicitud y se le dé el formato adecuado, que se envié la solicitud al servidor que hospeda el servicio web y se analice la respuesta para determinar si la solicitud se realizó correctamente o no y que se extraigan los datos devueltos. Para aislar la aplicación cliente de estas cuestiones, puede proporcionar un SDK que encapsule la interfaz REST y abstraiga estos detalles de bajo nivel dentro de un conjunto de métodos más funcional. Una aplicación cliente usa estos métodos, que convierten las llamadas en solicitudes REST de forma transparente y, luego, vuelve a convertir las respuestas a los valores que el método ha devuelto. Se trata de una técnica común que se implementa en muchos servicios, incluidos el SDK de Azure.

La creación de un SDK del lado cliente es una tarea de una envergadura considerable, ya que debe implementarse de forma coherente y probarse cuidadosamente. Sin embargo, gran parte de este proceso se puede realizar de forma mecánica y muchos proveedores ofrecen herramientas que pueden automatizar muchas de estas tareas.

## <a name="monitoring-a-web-api"></a>Supervisión de una API web
Según cómo haya publicado e implementado la API web, podrá supervisar la API web directamente o podrá recopilar información del uso y del estado, mediante un análisis del tráfico que pasa a través del servicio de administración de API.

### <a name="monitoring-a-web-api-directly"></a>Supervisión de una API web directamente
Si ha implementado la API web mediante la plantilla de API Web de ASP.NET (ya sea como un proyecto de API Web o como un rol web en un servicio en la nube de Azure) y Visual Studio 2013, puede recopilar datos de uso, rendimiento y disponibilidad con Application Insights de ASP.NET. Application Insights es un paquete que, de forma transparente, realiza un seguimiento y registra información sobre las solicitudes y respuestas cuando se implementa la API web en la nube; cuando el paquete está instalado y configurado, no es necesario modificar nada del código en la API web para usarlo. Cuando implementa la API web en un sitio web de Azure, se examina todo el tráfico y se recopilan las estadísticas siguientes:

* El tiempo de respuesta del servidor.
* El número de solicitudes del servidor y los detalles de cada una.
* Las solicitudes más lentas en cuanto al tiempo de respuesta medio.
* Los detalles de las solicitudes con errores.
* El número de sesiones que han iniciado otros exploradores y agentes de usuario.
* Las páginas visitadas con más frecuencia (lo que es útil principalmente para las aplicaciones web en lugar de las API web).
* Los distintos roles de usuario que obtienen acceso a la API web.

Puede ver estos datos en tiempo real desde el Portal de administración de Azure. También puede crear pruebas web que supervisen el estado de la API web. Una prueba web envía una solicitud periódica a un URI especificado en la API web y captura la respuesta. Puede especificar la definición de una respuesta correcta (por ejemplo, el código de estado HTTP 200) y, si la solicitud no devuelve esta respuesta, puede establecer que se envíe una alerta a un administrador. Si es necesario, el administrador puede reiniciar el servidor que hospeda la API web en caso de que se haya producido un error.

Para más información, consulte [Configurar Application Insights para ASP.NET](/azure/application-insights/app-insights-asp-net/).

### <a name="monitoring-a-web-api-through-the-api-management-service"></a>Supervisión de una API web a través del servicio de administración de API
Si ha publicado la API web mediante el servicio de administración de API, la página de administración de API del Portal de administración de Azure contiene un panel que le permite ver el rendimiento general del servicio. La página de análisis le permite profundizar en los detalles de cómo se usa el producto. Esta página contiene los siguientes campos:

* **Uso**. Esta pestaña proporciona información sobre el número de llamadas API realizadas y el ancho de banda que se ha usado para controlar esas llamadas en el tiempo. Puede filtrar los detalles de uso por producto, API y operación.
* **Estado**. Esta pestaña permite ver el resultado de las solicitudes de API (los códigos de estado HTTP devueltos), la eficacia de la directiva de almacenamiento en caché, el tiempo de respuesta de las API y el tiempo de respuesta del servicio. De nuevo, puede filtrar los datos del estado por producto, API y operación.
* **Actividad**. Esta pestaña ofrece un resumen de texto de los números de llamadas correctas, erróneas y bloqueadas, el tiempo medio de respuesta y los tiempos de respuesta de cada producto, API web y operación. Esta página también muestra el número de llamadas que ha realizado cada desarrollador.
* **En un vistazo**. Esta pestaña muestra un resumen de los datos de rendimiento, incluidos los desarrolladores responsables de realizar la mayoría de las llamadas a API y los productos, API web y operaciones que recibieron estas llamadas.

Puede usar esta información para determinar si una operación o API web determinadas están causando un cuello de botella y si es necesario escalar el entorno de host y agregar más servidores. También puede determinar si una o varias aplicaciones usan un volumen de recursos desproporcionado y aplican las directivas adecuadas para establecer las cuotas y limitar las tasas de llamadas.

> [!NOTE]
> Puede cambiar los detalles de un producto publicado y los cambios se aplicarán inmediatamente. Por ejemplo, puede agregar o quitar una operación de una API web sin necesidad de volver a publicar el producto que contenga la API web.
>
>

## <a name="more-information"></a>Más información
* [ASP.NET Web API OData](http://www.asp.net/web-api/overview/odata-support-in-aspnet-web-api) contiene ejemplos e información adicional sobre la implementación de una API web de OData mediante ASP.NET.
* [Introducing batch support in Web API and Web API OData](http://blogs.msdn.com/b/webdev/archive/2013/11/01/introducing-batch-support-in-web-api-and-web-api-odata.aspx) (Incorporación de compatibilidad con procesos por lotes en Web API y Web API OData).
* En el artículo [Idempotency Patterns](http://blog.jonathanoliver.com/idempotency-patterns/) del blog de Jonathan Oliver se ofrece una visión general de la idempotencia y cómo se relaciona con las operaciones de administración de datos.
* La página [Status Code Definitions](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html) del sitio web de W3C contiene una lista completa de códigos de estado HTTP y sus descripciones.
* En la página [Ejecutar tareas en segundo plano con WebJobs](/azure/app-service-web/web-sites-create-web-jobs/) se ofrece información y ejemplos sobre el uso de WebJobs para realizar operaciones en segundo plano.
* En la página [Notificación a los usuarios con Azure Notification Hubs](/azure/notification-hubs/notification-hubs-aspnet-backend-windows-dotnet-wns-notification/) se muestra cómo usar un centro de notificaciones de Azure para insertar respuestas asincrónicas en las aplicaciones cliente.
* En la página [API Management](https://azure.microsoft.com/services/api-management/) se describe cómo publicar un producto que ofrezca un acceso seguro y controlado a una API web.
* En la página [Referencia de la API de REST de Azure API Management](https://msdn.microsoft.com/library/azure/dn776326.aspx) se describe cómo usar la API de REST de administración de API para crear aplicaciones de administración personalizadas.
* En la página [Métodos de enrutamiento de Traffic Manager](/azure/traffic-manager/traffic-manager-routing-methods/) se resume cómo se puede usar Azure Traffic Manager para realizar un equilibrio de carga de las solicitudes entre varias instancias de un sitio web que hospeda una API web.
* En la página [Configurar Application Insights para ASP.NET](/azure/application-insights/app-insights-asp-net/) se ofrece información detallada sobre la instalación y configuración de Application Insights en un proyecto ASP.NET Web API.


<!-- links -->

[api-design]: ./api-design.md
