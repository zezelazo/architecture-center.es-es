---
title: Patrón Gateway Aggregation
description: Usa una puerta de enlace para agregar varias solicitudes individuales en una sola solicitud.
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: f59c8b8b02c6db28024d13621b782997e63a4e9e
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2017
---
# <a name="gateway-aggregation-pattern"></a>Patrón Gateway Aggregation

Usa una puerta de enlace para agregar varias solicitudes individuales en una sola solicitud. Este patrón es útil cuando un cliente debe hacer múltiples llamadas a diferentes sistemas back-end para realizar una operación.

## <a name="context-and-problem"></a>Contexto y problema

Para llevar a cabo una sola tarea, el cliente puede tener que realizar varias llamadas a diversos servicios back-end. Una aplicación que depende de muchos servicios para realizar una tarea debe gastar recursos en cada solicitud. Cuando se agrega cualquier característica o servicio nuevo a la aplicación, se necesitan solicitudes adicionales, lo que aumenta aún más los requisitos de recursos y las llamadas de red. Este intercambio de mensajes entre un cliente y un back-end puede afectar negativamente al rendimiento y la escalabilidad de la aplicación.  Las arquitecturas de microservicios han hecho que este problema sea más común, ya que las aplicaciones creadas en torno a muchos servicios más pequeños tienen naturalmente una mayor cantidad de llamadas de servicios cruzados. 

En el siguiente diagrama, el cliente envía solicitudes a cada servicio (1,2,3). Cada servicio procesa la solicitud y envía una respuesta de nuevo a la aplicación (4,5,6). A través de una red móvil con una latencia típicamente elevada, el uso de solicitudes individuales de esta manera es ineficiente y podría resultar en una conectividad rota o en solicitudes incompletas. Si bien cada solicitud puede realizarse en paralelo, la aplicación debe enviar, esperar y procesar los datos de cada solicitud, todo ello en conexiones independientes, lo que aumenta las posibilidades de error.

![](./_images/gateway-aggregation-problem.png) 

## <a name="solution"></a>Solución

Utilice una puerta de enlace para reducir el intercambio de mensajes entre el cliente y los servicios. La puerta de enlace recibe las solicitudes de los clientes, envía estas solicitudes a los distintos sistemas back-end y, después, agrega los resultados y los devuelve al cliente solicitante.

Este patrón puede reducir el número de solicitudes que la aplicación realiza a los servicios back-end y mejorar el rendimiento de la aplicación en redes de alta latencia.

En el diagrama siguiente, la aplicación envía una solicitud a la puerta de enlace (1). La solicitud contiene un paquete de solicitudes adicionales. La puerta de enlace descompone y procesa cada solicitud al enviarla al servicio correspondiente (2). Cada servicio devuelve una respuesta a la puerta de enlace (3). La puerta de enlace combina las respuestas de cada servicio y envía la respuesta a la aplicación (4). La aplicación realiza una única solicitud y recibe una sola respuesta de la puerta de enlace.

![](./_images/gateway-aggregation.png)

## <a name="issues-and-considerations"></a>Problemas y consideraciones

- La puerta de enlace no debe introducir el acoplamiento de servicio a través de los servicios back-end.
- La puerta de enlace debe estar cerca de los servicios back-end para reducir la latencia lo más posible.
- El servicio de puerta de enlace puede introducir un único punto de error. Asegúrese de que la puerta de enlace está diseñada correctamente para satisfacer los requisitos de disponibilidad de la aplicación.
- La puerta de enlace puede introducir un cuello de botella. Asegúrese de la puerta de enlace tiene un rendimiento adecuado para manejar la carga y de que se puede escalar para cumplir con su crecimiento previsto.
- Realice pruebas de carga en la puerta de enlace para asegurarse de que no introduce errores en cascada en los servicios.
- Implemente un diseño resistente, mediante técnicas como los patrones [Bulkhead][bulkhead], [Circuit breaking][circuit-breaker], [Retry][retry] y tiempos de espera.
- Si una o más llamadas de servicio tardan demasiado, puede ser aceptable agotar el tiempo de espera y devolver un conjunto parcial de datos. Considere cómo la aplicación va a manejar este escenario.
- Use la entrada/salida asincrónica para asegurarse de que un retraso en el back-end no causa problemas de rendimiento en la aplicación.
- Implemente el seguimiento distribuido mediante los identificadores de correlación para realizar el seguimiento de cada llamada individual.
- Supervise las métricas de solicitud y los tamaños de respuesta.
- Considere la posibilidad de devolver los datos almacenados en caché como una estrategia de conmutación por error para tratar los errores.
- En lugar de generar agregación en la puerta de enlace, considere colocar un servicio de agregación detrás de la puerta de enlace. Es probable que la agregación de solicitudes tenga necesidades de recursos diferentes a las de otros servicios en la puerta de enlace y puede afectar a la funcionalidad de enrutamiento y a la descarga de la puerta de enlace.

## <a name="when-to-use-this-pattern"></a>Cuándo usar este patrón

Use este patrón cuando:

- Un cliente necesite comunicarse con varios servicios de back-end para realizar una operación.
- El cliente puede usar redes con latencia elevada, por ejemplo, redes móviles.

Este patrón puede no ser adecuado cuando:

- Desea reducir el número de llamadas entre un cliente y un único servicio en varias operaciones. En ese caso, puede que sea mejor agregar una operación por lotes al servicio.
- El cliente o aplicación se encuentra cerca de los servicios back-end y la latencia no es un factor importante.

## <a name="example"></a>Ejemplo

En el ejemplo siguiente se muestra cómo crear un simple servicio NGINX de agregación de puerta de enlace mediante el uso de Lua.

```lua
worker_processes  4;

events {
  worker_connections 1024;
}

http {
  server {
    listen 80;

    location = /batch {
      content_by_lua '
        ngx.req.read_body()

        -- read json body content
        local cjson = require "cjson"
        local batch = cjson.decode(ngx.req.get_body_data())["batch"]

        -- create capture_multi table
        local requests = {}
        for i, item in ipairs(batch) do
          table.insert(requests, {item.relative_url, { method = ngx.HTTP_GET}})
        end

        -- execute batch requests in parallel
        local results = {}
        local resps = { ngx.location.capture_multi(requests) }
        for i, res in ipairs(resps) do
          table.insert(results, {status = res.status, body = cjson.decode(res.body), header = res.header})
        end

        ngx.say(cjson.encode({results = results}))
      ';
    }

    location = /service1 {
      default_type application/json;
      echo '{"attr1":"val1"}';
    }

    location = /service2 {
      default_type application/json;
      echo '{"attr2":"val2"}';
    }
  }
}
```

## <a name="related-guidance"></a>Instrucciones relacionadas

- [Patrón Backends for Frontends](./backends-for-frontends.md)
- [Patrón Gateway Offloading](./gateway-offloading.md)
- [Patrón Gateway Routing](./gateway-routing.md)

[bulkhead]: ./bulkhead.md
[circuit-breaker]: ./circuit-breaker.md
[retry]: ./retry.md