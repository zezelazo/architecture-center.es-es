---
title: "Patrón Gateway Offloading"
description: Descarga una funcionalidad de servicio compartida o especializada en un proxy de puerta de enlace.
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: 6b3e4541aae77349ca91c18c788ddb508912361d
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2017
---
# <a name="gateway-offloading-pattern"></a>Patrón Gateway Offloading

Descarga una funcionalidad de servicio compartida o especializada en un proxy de puerta de enlace. Este patrón puede simplificar la implementación de la aplicación al mover la funcionalidad del servicio compartido, como el uso de certificados SSL, de otras partes de la aplicación a la puerta de enlace.

## <a name="context-and-problem"></a>Contexto y problema

Algunas de las características que se usan habitualmente en varios servicios requieren configuración, administración y mantenimiento. Un servicio compartido o especializado que se distribuye con cada implementación de aplicación aumenta la sobrecarga administrativa y la probabilidad de errores de implementación. Las actualizaciones de las características compartidas deben implementarse en todos los servicios que la comparten.

La correcta administración de los problemas de seguridad (validación de tokens, cifrado, administración de certificados SSL) y otras tareas complejas puede requerir conocimientos muy especializados de los miembros del equipo. Por ejemplo, un certificado que una aplicación necesita debe configurarse e implementarse en todas las instancias de la aplicación. Con cada nueva implementación, el certificado debe administrarse para garantizar que no va a expirar. Cualquier certificado común a punto de caducar debe actualizarse, probarse y comprobarse en cada implementación de la aplicación.

Otros servicios comunes, como la autenticación, la autorización, el registro, la supervisión, o l [limitación](./throttling.md) pueden resultar difíciles de implementar y administrar en un gran número de implementaciones. Es posible que convenga combinar este tipo de funcionalidad con el fin de reducir la sobrecarga y la posibilidad de errores.

## <a name="solution"></a>Solución

Descargue algunas características en una puerta de enlace de API, especialmente cuestiones transversales, como la administración de certificados, la autenticación, la terminación SSL, la supervisión, la traducción de protocolos o la limitación. 

En el siguiente diagrama se muestra una puerta de enlace de API que termina las conexiones entrantes de SSL. Solicita datos en nombre del solicitante original desde cualquier servidor HTTP en dirección ascendente de la puerta de enlace de API.

 ![](./_images/gateway-offload.png)
 
Estos son algunos ejemplos de las ventajas de este patrón:

- Simplifica el desarrollo de servicios mediante la eliminación de la necesidad de distribuir y mantener los recursos de compatibilidad, como los certificados de servidor web y la configuración de sitios web seguros. Una configuración más simple facilita la administración y la escalabilidad, y simplifica las actualizaciones del servicio.

- Permite a los equipos expertos implementar características que requieren conocimientos especializados, como la seguridad. Esto permite que el equipo principal se centre en la funcionalidad de la aplicación, dejando estos aspectos especializadas pero transversales en manos de los expertos correspondientes.

- Proporciona coherencia en el registro y la supervisión de las solicitudes y las respuestas. Aunque un servicio no se haya instrumentado correctamente, la puerta de enlace se puede configurar para garantizar un registro y una supervisión mínimos.

## <a name="issues-and-considerations"></a>Problemas y consideraciones

- Asegúrese de que la puerta de enlace de API es resistente a errores y de alta disponibilidad. Ejecute varias instancias de la puerta de enlace de API para evitar los puntos únicos de error. 
- Asegúrese de que la puerta de enlace está diseñada para los requisitos de capacidad y escalado de la aplicación y los puntos de conexión. Asegúrese de que la puerta de enlace no se convierte en cuello de botella para la aplicación y de que es lo suficientemente escalable.
- Descargue solo las características que use toda la aplicación, como la seguridad o la transferencia de datos.
- La lógica de negocios no debe descargarse a la puerta de enlace de API. 
- Si necesita realizar un seguimiento de las transacciones, considere la posibilidad de generar identificadores de correlación para fines de registro.

## <a name="when-to-use-this-pattern"></a>Cuándo usar este patrón

Use este patrón cuando:

- Una implementación de una aplicación tenga un problema compartido, como de los certificados SSL o el cifrado.
- Una característica común a las implementaciones de la aplicación pueda tener requisitos de recursos diferentes, como los recursos de memoria, la capacidad de almacenamiento o las conexiones de red.
- Quiera que la responsabilidad de problemas como la seguridad de red, la limitación u otros problemas de límite de red recaiga en un equipo más especializado.

Este patrón puede no ser adecuado si introduce acoplamiento entre servicios.

## <a name="example"></a>Ejemplo

Al usar Nginx como dispositivo para descargar SSL, la siguiente configuración finaliza una conexión entrante de SSL y distribuye la conexión a uno de los tres servidores HTTP ascendentes.

```
upstream iis {
        server  10.3.0.10    max_fails=3    fail_timeout=15s;
        server  10.3.0.20    max_fails=3    fail_timeout=15s;
        server  10.3.0.30    max_fails=3    fail_timeout=15s;
}

server {
        listen 443;
        ssl on;
        ssl_certificate /etc/nginx/ssl/domain.cer;
        ssl_certificate_key /etc/nginx/ssl/domain.key;

        location / {
                set $targ iis;
                proxy_pass http://$targ;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto https;
proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header Host $host;
        }
}
```

## <a name="related-guidance"></a>Instrucciones relacionadas

- [Patrón Backends for Frontends](./backends-for-frontends.md)
- [Patrón Gateway Aggregation](./gateway-aggregation.md)
- [Patrón Gateway Routing](./gateway-routing.md)

