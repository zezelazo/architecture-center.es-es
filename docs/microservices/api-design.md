---
title: Diseño de API
description: Diseño de API para microservicios
author: MikeWasson
ms.date: 12/08/2017
ms.openlocfilehash: e3524fca177d8c15b280d0f8a706539369c1773a
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/28/2018
ms.locfileid: "47429135"
---
# <a name="designing-microservices-api-design"></a>Diseño de microservicios: diseño de API

Las arquitecturas de los microservicios necesitan un buen diseño de API, ya que todos los intercambios de datos entre servicios se producen mediante mensajes o llamadas API. Las API deben ser eficaces para evitar los patrones [Chatty I/O](../antipatterns/chatty-io/index.md). Dado que los servicios están diseñados por equipos que trabajan de forma independiente, las API deben tener semántica bien definida y esquemas de control de versiones, de manera que las actualizaciones no interrumpan otros servicios.

![](./images/api-design.png)

Es importante distinguir entre dos tipos de API:

- Las API públicas que llaman las aplicaciones cliente. 
- Las API de back-end que se usan para la comunicación entre servicios.

Para estos dos casos, los requisitos son ligeramente distintos. Una API pública debe ser compatible con las aplicaciones cliente, normalmente las aplicaciones de explorador o las aplicaciones nativas para dispositivos móviles. En la mayoría de los casos, esto significa que la API pública usa REST a través de HTTP. Sin embargo, para la API de back-end, es necesario considerar el rendimiento de la red. Según la granularidad de los servicios, la comunicación entre ellos puede producir una gran cantidad de tráfico. Los servicios pueden volverse rápidamente dependientes de las operaciones de E/S. Por esa razón, gana importancia la consideración, por ejemplo, de la velocidad de serialización y el tamaño de la carga. Algunas alternativas populares al uso de REST a través de HTTP incluyen gRPC, Apache Avro y Apache Thrift. Estos protocolos admiten la serialización binaria y normalmente son más eficaces que HTTP.

## <a name="considerations"></a>Consideraciones

Estas son algunas consideraciones que pensar al elegir cómo implementar una API.

**REST frente a RPC**. Tenga en cuenta los inconvenientes de usar una interfaz tipo REST en comparación con una tipo RPC.

- REST modela recursos, lo cual puede ser una manera natural de expresar el modelo de dominio. Define una interfaz uniforme en función de verbos HTTP, que fomenta la evolución. Tiene una semántica bien definida en términos de idempotencia, efectos secundarios y códigos de respuesta. Además, aplica la comunicación sin estado, lo que mejora la escalabilidad. 

- RPC está más orientado a las operaciones o los comandos. Como las interfaces RPC parecen llamadas a métodos locales, pueden provocar el diseño de API demasiado fragmentadas. Sin embargo, eso no significa que la interfaz RPC deba estar demasiado fragmentada. Simplemente, hay que tener cuidado al diseñarla.

En una interfaz RESTful, la opción más común es REST a través de HTTP con JSON. Para una interfaz tipo RPC, hay varias plataformas populares, como gRPC, Apache Avro y Apache Thrift.

**Eficacia**. Considere la eficacia en cuanto a tamaño de carga, memoria y velocidad. Una interfaz basada en gRPC suele ser más rápida que REST a través de HTTP.

**Lenguaje de definición de interfaz (IDL)**. El lenguaje de definición de interfaz se utiliza para definir los métodos, los parámetros y los valores devueltos de una API. Puede utilizarse para generar código de cliente, código de serialización y documentación de API. Pero también lo pueden consumir herramientas de prueba de API como Postman. Las plataformas como gRPC, Avro y Thrift definen sus propias especificaciones de lenguaje de definición de interfaz. REST a través de HTTP no tiene un formato de lenguaje de definición de interfaz estándar, pero una opción común es OpenAPI (anteriormente denominado Swagger). También puede crear una API de REST de HTTP sin un lenguaje de definición formal, pero se perderán las ventajas de la generación de código y las pruebas.

**Serialización**. ¿Cómo se serializan los objetos a través de la conexión? Las opciones incluyen formatos de texto (principalmente, JSON) y formatos binarios como el búfer de protocolo. Los formatos binarios son generalmente más rápidos que los de texto. Sin embargo, JSON tiene ventajas en cuanto a interoperabilidad, porque la mayoría de lenguajes y plataformas admiten la serialización de JSON. Algunos formatos de serialización requieren un esquema fijo y, algunos, la compilación de un archivo de definición de esquemas. En ese caso, debe incorporar este paso al proceso de compilación. 

**Compatibilidad del lenguaje y la plataforma**. HTTP se admite en casi todas las plataformas y lenguajes. gRPC, Avro y Thrift tienen bibliotecas de C++, C#, Java y Python. Thrift y gRPC también admiten Go. 

**Compatibilidad e interoperabilidad**. Si elige un protocolo como gRPC, puede que necesite una capa de traducción de protocolo entre la API pública y el back-end. Una [puerta de enlace](./gateway.md) puede realizar esa función. Si utiliza una malla de servicio, considere los protocolos compatibles. Por ejemplo, linkerd tiene compatibilidad integrada para HTTP, Thrift y gRPC. 

Nuestra recomendación de base de referencia es elegir REST a través de HTTP, a menos que necesite los beneficios de rendimiento de un protocolo binario. REST a través de HTTP no necesita bibliotecas especiales. Crea un acoplamiento mínimo, dado que los autores de las llamadas no necesitan código auxiliar de cliente para comunicarse con el servicio. No hay ecosistemas enriquecidos de herramientas para admitir las definiciones de esquema, las pruebas y la supervisión de puntos de conexión HTTP de RESTful. Por último, HTTP es compatible con los clientes de explorador, por lo que no se necesita una capa de traducción de protocolo entre el cliente y el back-end. 

Sin embargo, si elige REST a través de HTTP, debe probar el rendimiento y la carga al principio del desarrollo para asegurarse de si son suficientes para el escenario.

## <a name="restful-api-design"></a>Diseño de la API de RESTful

Hay muchos recursos para diseñar las API de RESTful. Estos son algunos que pueden resultarle útiles:

- [Diseño de API](../best-practices/api-design.md) 

- [Implementación de API](../best-practices/api-implementation.md) 

- [Directrices para API de REST de Microsoft](https://github.com/Microsoft/api-guidelines)

Estas son algunas consideraciones específicas que tener en cuenta.

- Esté atento a las API con fugas de detalles de implementación internos o que simplemente reflejan un esquema de base de datos interna. La API debe modelar el dominio. Es un contrato entre los servicios y lo ideal es que solo cambie cuando se agreguen nuevas funcionalidades, no solo al refactorizar código o normalizar una tabla de base datos. 

- Distintos tipos de cliente, por ejemplo una aplicación para dispositivos móviles o un explorador web de escritorio, pueden requerir tamaños de carga o patrones de interacción diferentes. Considere el uso del [patrón Backends for Frontends](../patterns/backends-for-frontends.md) para crear servidores back-end independientes para cada cliente que expongan una interfaz óptima para ellos.

- Para las operaciones con efectos secundarios, considere la posibilidad de hacerlos idempotentes e implementarlos como métodos PUT. Esto protegerá los reintentos y mejorará la resiliencia. En los capítulos de [ingesta y flujo de trabajo](./ingestion-workflow.md#idempotent-vs-non-idempotent-operations) y [comunicación entre servicios](./interservice-communication.md) se habla de este tema con más detalle.

- Los métodos HTTP pueden tener semántica asincrónica, donde el método devuelve una respuesta inmediatamente, pero el servicio lleva a cabo la operación de forma asincrónica. En ese caso, el método debe devolver un código de respuesta [HTTP 202](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html), que indica que la solicitud se ha aceptado para el procesamiento, pero este aún no se ha completado.

## <a name="mapping-rest-to-ddd-patterns"></a>Asignación de REST a patrones de diseño basado en el dominio (DDD)

Los patrones como la entidad, el agregado y el objeto de valor están diseñados para difundir determinadas restricciones en los objetos del modelo de dominio. En muchos debates sobre el diseño basado en el dominio, los patrones se modelan con conceptos de lenguaje basado en los objetos como los constructores o los captadores de propiedades y los establecedores. Por ejemplo, se asume que los *objetos de valor* son inmutables. En un lenguaje de programación basado en los objetos, podría aplicar esto al asignar los valores en el constructor y establecer las propiedades de solo lectura:

```ts
export class Location {
    readonly latitude: number;
    readonly longitude: number;

    constructor(latitude: number, longitude: number) {
        if (latitude < -90 || latitude > 90) {
            throw new RangeError('latitude must be between -90 and 90');
        }
        if (longitude < -180 || longitude > 180) {
            throw new RangeError('longitude must be between -180 and 180');
        }
        this.latitude = latitude;
        this.longitude = longitude;
    }
}
```

Este tipo de prácticas de codificación es especialmente importante al compilar una aplicación monolítica tradicional. Con una base de código de gran tamaño, puede que muchos subsistemas usen el objeto `Location`, por lo que es importante que este fomente el comportamiento correcto. 

Otro ejemplo es el patrón Repository, que garantiza que otras partes de la aplicación no realizan lecturas ni escrituras directas en el almacén de datos:

![](./images/repository.png)

Sin embargo, en una arquitectura de microservicios, los servicios no comparten el mismo código base ni almacenes de datos. En su lugar, se comunican mediante las API. Considere el caso en que el servicio Scheduler solicita información acerca de un dron al servicio Drone. El servicio Drone tiene su modelo interno de dron, que se expresa mediante código. Pero Scheduler no lo ve. En su lugar, recibe una *representación* de la entidad del dron (quizás un objeto JSON en una respuesta HTTP).

![](./images/ddd-rest.png)

El servicio Scheduler no puede modificar los modelos internos del servicio Drone ni escribir en su almacén de datos. Esto significa que el código que implementa el servicio Drone tiene una menor superficie expuesta, en comparación con el código de un monolito tradicional. Si el servicio Drone define una clase Location, se limita el ámbito de esa clase, ningún otro servicio consumirá directamente la clase. 

Por estas razones, esta guía no se centra mucho en las prácticas de codificación, ya que afectan a los patrones tácticos de diseño basado en el dominio. Pero resulta que también puede modelar muchos de los patrones de dominio basado en el dominio mediante las API de REST. 

Por ejemplo: 

- Los agregados se asignan de forma natural a los *recursos* de REST. Por ejemplo, el agregado Delivery se expondría como recurso de Delivery API.

- Los agregados son los límites de coherencia. Las operaciones en los agregados nunca deben dejar uno en estado incoherente. Por lo tanto, es necesario evitar crear API que permitan a un cliente manipular el estado interno de un agregado. En su lugar, dé prioridad a las API genéricas que exponen los agregados como recursos.

- Las entidades tienen identidad única. En REST, los recursos tienen identificadores únicos en forma de direcciones URL. Cree direcciones URL de recurso que se correspondan con la identidad de dominio de una entidad. La asignación de direcciones URL a la identidad de dominio puede ser opaca para el cliente.

- A las entidades secundarias de un agregado se accede desde la entidad raíz. Si sigue los principios [HATEOAS](https://en.wikipedia.org/wiki/HATEOAS), podrá acceder a las entidades secundarias a través de vínculos en la representación de la entidad primaria. 

- Dado que los objetos de valor son inmutables, las actualizaciones se realizan mediante el reemplazo del objeto de valor completo. En REST, implemente las actualizaciones mediante solicitudes PUT o PATCH. 

- Un repositorio permite a los clientes consultar, añadir o eliminar objetos de una colección y extrae los detalles del almacén de datos subyacente. En REST, una colección puede ser un recurso distinto, con métodos para consultar la colección o agregar nuevas entidades a esta.

Al diseñar las API, tenga en cuenta cómo expresan el modelo de dominio, no solo los datos dentro de este, sino también las operaciones empresariales y las restricciones en los datos.

| Concepto de DDD | Equivalente en REST | Ejemplo | 
|-------------|-----------------|---------|
| Agregado | Recurso | `{ "1":1234, "status":"pending"... }` | 
| Identidad | URL | `https://delivery-service/deliveries/1` |
| Entidades secundarias | Vínculos | `{ "href": "/deliveries/1/confirmation" }` |
| Actualización de objetos de valor | PUT o PATCH | `PUT https://delivery-service/deliveries/1/dropoff` |
| Repositorio | Colección | `https://delivery-service/deliveries?status=pending` |


## <a name="api-versioning"></a>Control de versiones de la API

Una API es un contrato entre un servicio y los clientes o los consumidores de ese servicio. Si una API cambia, existe el riesgo de afectar a los clientes que dependen de la API, ya sean clientes externos u otros microservicios. Por lo tanto, es buena idea minimizar el número de cambios en la API. A menudo, los cambios en la implementación subyacente no requieren ningún cambio en la API. En la práctica, sin embargo, en algún momento querrá agregar características o nuevas funcionalidades que necesiten cambiar una API existente.

Siempre que sea posible, asegúrese de que los cambios en la API sean compatibles con las versiones anteriores. Por ejemplo, evite eliminar un campo de un modelo, ya que puede afectar a los clientes que esperan que el campo esté ahí. Agregar un campo no elimina la compatibilidad, dado que los clientes deben prescindir de los campos que no comprendan en las respuestas. Sin embargo, el servicio debe considerar si un cliente anterior omite el nuevo campo en una solicitud. 

Permita el control de versiones en el contrato de API. Si el cambio de API puede afectar, introduzca una nueva versión de API. Siga permitiendo la versión anterior y permita que los clientes seleccionen la versión a la cual llamar. Hay un par de formas de hacerlo. Una es simplemente para exponer ambas versiones en el mismo servicio. Otra opción consiste en ejecutar dos versiones del servicio en paralelo y enrutar las solicitudes a una u otra, en función de las reglas de enrutamiento de HTTP. 

![](./images/versioning1.svg)

La compatibilidad con varias versiones tiene costo en términos de tiempo del desarrollador, las pruebas y la sobrecarga operativa. Por lo tanto, es conveniente dejar de utilizar las versiones anteriores tan pronto como sea posible. Para las API internas, el equipo que posee la API puede trabajar con otros que le ayuden a migrar a la nueva versión. Aquí resultan útiles los procesos de gobierno entre equipos. Para las API (públicas) externas, puede resultar más difícil dejar de utilizar una versión de API, especialmente si la consumen terceros o aplicaciones cliente nativas. 

Cuando cambia de una implementación de servicio, es útil etiquetar el cambio con una versión. La versión proporciona información importante para solucionar errores. Saber exactamente qué versión del servicio se llamaba puede resultar muy útil para el análisis de la causa principal. Considere el uso de [Semantic Versioning](https://semver.org/) para las versiones del servicio. Semantic Versioning usa el formato *MAJOR.MINOR.PATCH*. Sin embargo, los clientes solo tienen que seleccionar una API por el número de versión principal, o quizá la secundaria, (en caso de cambios considerables, pero sin interrupción, entre las versiones secundarias). En otras palabras, es razonable que los clientes seleccionen entre las versiones 1 y 2 de una API, pero no que seleccionen la versión 2.1.3. Si permite este nivel de granularidad, corre el riesgo de tener que admitir la proliferación de versiones. 

Para más información acerca del control de versiones de API, consulte [Control de versiones de una API web RESTful](../best-practices/api-design.md#versioning-a-restful-web-api).

> [!div class="nextstepaction"]
> [Ingesta de datos y flujo de trabajo](./ingestion-workflow.md)