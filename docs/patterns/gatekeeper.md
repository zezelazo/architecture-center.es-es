---
title: Gatekeeper
description: Protege aplicaciones y servicios mediante una instancia de host dedicada que actúa como agente entre los clientes y la aplicación o servicio, valida y sanea las solicitudes, y pasa las solicitudes y datos entre ellos.
keywords: Patrón de diseño
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- security
ms.openlocfilehash: 39f8548bbccb5e19d433f65b2e7e09147d676996
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2017
ms.locfileid: "24541327"
---
# <a name="gatekeeper-pattern"></a>Patrón Gatekeeper

[!INCLUDE [header](../_includes/header.md)]

Protege aplicaciones y servicios mediante una instancia de host dedicada que actúa como agente entre los clientes y la aplicación o servicio, valida y sanea las solicitudes, y pasa las solicitudes y datos entre ellos. Esto puede proporcionar una capa adicional de seguridad y limitar la superficie de ataque del sistema.

## <a name="context-and-problem"></a>Contexto y problema

Las aplicaciones exponen su funcionalidad a los clientes mediante la aceptación y el procesamiento de solicitudes. En escenarios hospedados en la nube, las aplicaciones exponen los puntos de conexión a los que los clientes se conectan y suelen incluyen el código para controlar las solicitudes de clientes. Este código realiza la autenticación y la validación, el procesamiento de algunas de las solicitudes, o de todas, y es probable que acceda al almacenamiento y a otros servicios en nombre del cliente.

Si un usuario malintencionado puede poner en peligro el sistema y obtener acceso al entorno de hospedaje de la aplicación, los mecanismos de seguridad que usa, como las credenciales y las claves de almacenamiento, y se exponen los servicios y datos a los que accede. Como consecuencia, el usuario malintencionado puede obtener acceso libre a información confidencial y a otros servicios.

## <a name="solution"></a>Solución

Para minimizar el riesgo de que los clientes obtengan acceso a información confidencial y a servicios, desacople los hosts o las tareas que exponen puntos de conexión públicos desde el código que procesa las solicitudes y accede al almacenamiento. Puede conseguirlo mediante una fachada o una tarea dedicada que interactúa con los clientes y, a continuación, entrega la solicitud&mdash;quizás a través de una interfaz desacoplada&mdash;a los hosts o las tareas que controlarán la solicitud. La ilustración proporciona información general de alto nivel de este patrón.

![Información general de alto nivel de este patrón](./_images/gatekeeper-diagram.png)


El patrón del equipo selector se puede usar simplemente para proteger el almacenamiento, o bien se puede utilizar como una fachada más completa para proteger todas las funciones de la aplicación. Los factores importantes son:

- **Validación controlada.** El equipo selector valida todas las solicitudes y rechaza aquellas que no cumplen los requisitos de validación.
- **Riesgo y exposición limitados.** El equipo selector no tiene acceso a las credenciales o las claves que usa el host de confianza para acceder al almacenamiento y a los servicios. Si el equipo selector corre peligro, el atacante no obtiene acceso a estas credenciales o claves.
- **Seguridad adecuada.** El equipo selector se ejecuta en un modo con privilegios limitados, mientras que el resto de la aplicación se ejecuta en el modo de plena confianza necesario para acceder al almacenamiento y a los servicios. Si el equipo selector está en peligro, no puede acceder directamente a los servicios de la aplicación ni a los datos.

Este patrón actúa como un firewall en una topografía de red típica. Permite que el equipo selector examine las solicitudes y tome una decisión sobre si se debe pasar una de ellas al host de confianza (a veces denominado el maestro de claves) que realiza las tareas necesarias. Esta decisión normalmente requiere que el equipo selector valide y corrija el contenido de la solicitud antes de pasarlo al host de confianza.

## <a name="issues-and-considerations"></a>Problemas y consideraciones

Tenga en cuenta los puntos siguientes al decidir cómo implementar este patrón:

- Asegúrese de que los hosts de confianza a los que el equipo selector pasa las solicitudes exponen solo puntos de conexión internos o protegidos, y que se conectan solo al equipo selector. Los hosts de confianza no deben exponer los puntos de conexión externos ni las interfaces.
- El equipo selector debe ejecutarse en un modo con privilegios limitados. Normalmente esto significa que el equipo selector y el host de confianza se ejecutan en servicios hospedados o máquinas virtuales independientes.
- El equipo selector no debe realizar ningún tipo de procesamiento relacionado con la aplicación o los servicios, ni acceder a los datos. Su función es exclusivamente validar y corregir solicitudes. Es posible que los hosts de confianza tengan que realizar una validación adicional de las solicitudes, pero la validación principal debe hacerla el equipo selector.
- Cuando sea posible, utilice un canal de comunicación seguro (HTTPS, SSL o TLS) entre el equipo selector y los hosts de confianza o las tareas. Sin embargo, algunos entornos de hospedaje no admiten HTTPS en los puntos de conexión internos.
- La incorporación de una capa adicional a la aplicación para implementar el modelo del equipo selector es probable que afecte al rendimiento debido al procesamiento adicional y a la comunicación de red que requiere.
- La instancia del equipo selector podría ser un único punto de error. Para minimizar el impacto de un error, considere la posibilidad de implementar instancias adicionales y usar un mecanismo de escalado automático para garantizar la capacidad para mantener la disponibilidad.

## <a name="when-to-use-this-pattern"></a>Cuándo usar este patrón

Este patrón es útil para:

- Aplicaciones que controlan información confidencial, exponen servicios que deben tener un alto grado de protección frente a ataques malintencionados o realizar operaciones de misión crítica que no se deben interrumpir.
- Aplicaciones distribuidas en las que es necesario realizar la validación de las solicitudes de forma independiente con respecto a las tareas principales o centralizar esta validación para simplificar la administración y el mantenimiento.

## <a name="example"></a>Ejemplo

En un escenario hospedado en la nube, este patrón se puede implementar desacoplando el rol del equipo selector o la máquina virtual de los roles y servicios de confianza en una aplicación. Para ello, utilice un punto de conexión interno, una cola o un almacenamiento como mecanismo de comunicación intermedio. En la ilustración se muestra cómo utilizar un punto de conexión interno.

![Un ejemplo del patrón que usan los roles web y de trabajo de Cloud Services](./_images/gatekeeper-endpoint.png)


## <a name="related-patterns"></a>Patrones relacionados

El [patrón Valet Key](valet-key.md) también puede ser apropiado al implementar el patrón Gatekeeper. Cuando se establece comunicación entre Gatekeeper y los roles de confianza se recomienda mejorar la seguridad mediante el uso de claves o tokens que limiten los permisos para acceder a los recursos. Describe cómo usar un token o una clave que proporcione a los clientes acceso directo restringido a un recurso o servicio específicos.
