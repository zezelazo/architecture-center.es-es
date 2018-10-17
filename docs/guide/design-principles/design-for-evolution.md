---
title: Diseño para cambiar
description: Un diseño evolutivo es clave para una innovación continua.
author: MikeWasson
ms.date: 08/30/2018
ms.openlocfilehash: df5a2d0756295a9632b3ea336527b2fbfb35318c
ms.sourcegitcommit: f6be2825bf2d37dfe25cfab92b9e3973a6b51e16
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/08/2018
ms.locfileid: "48858153"
---
# <a name="design-for-evolution"></a>Diseño para evolucionar

## <a name="an-evolutionary-design-is-key-for-continuous-innovation"></a>Un diseño evolutivo es clave para una innovación continua.

Todas las aplicaciones que tienen éxito cambian a lo largo del tiempo para corregir errores, agregar nuevas características, incorporar nuevas tecnologías o hacer que los sistemas existentes sean más resistentes y escalables. Si todas las partes de una aplicación están estrechamente acopladas, resulta muy difícil introducir cambios en el sistema. Un cambio en una parte de la aplicación puede interrumpir otra parte o provocar cambios que se propaguen por todo el código base.

Este problema no se limita a las aplicaciones monolíticas. Una aplicación puede estar dividida en servicios pero seguir mostrando el tipo de estrecho acoplamiento que deja el sistema rígido y frágil. Pero cuando los servicios están diseñados para evolucionar, los equipos pueden innovar y proporcionar continuamente nuevas características. 

Los microservicios se están convirtiendo en una manera popular de lograr un diseño evolutivo, ya que abordan muchos de los aspectos que se enumeran aquí.

## <a name="recommendations"></a>Recomendaciones

**Exigir una cohesión alta y un acoplamiento flexible**. Un servicio está *cohesionado* si proporciona una funcionalidad que está unida de forma lógica. Unos servicios tienen un *acoplamiento flexible* si puede cambiar un servicio sin cambiar el otro. Una cohesión alta significa, en general, que los cambios en una función requerirán realizar cambios en otras funciones relacionadas. Si cree que la actualización de un servicio requiere actualizaciones coordinadas con otros servicios, puede ser una señal de que los servicios no están cohesionados. Uno de los objetivos del diseño basado en el dominio (DDD) es identificar estos límites.

**Encapsular el conocimiento del dominio**. Cuando un cliente utiliza un servicio, la responsabilidad de aplicar las reglas de negocio del dominio no debe recaer en el cliente. En su lugar, el servicio debe encapsular todo el conocimiento del dominio que recae bajo su responsabilidad. De lo contrario, cada cliente debería aplicar las reglas de negocio y se tendría un conocimiento del dominio propagado por distintas partes de la aplicación. 

**Utilizar la mensajería asincrónica**. La mensajería asincrónica es una manera de desacoplar el productor de mensajes del consumidor. El productor no depende de que el consumidor responda al mensaje o realice una acción determinada. Con una arquitectura de publicación/suscripción, es posible que el productor ni siquiera sepa quién consume el mensaje. Unos servicios nuevos puedan consumir fácilmente los mensajes sin ninguna modificación en el productor.

**No crear el conocimiento del dominio en una puerta de enlace**. Las puertas de enlace pueden ser útiles en una arquitectura de microservicios para cosas como enrutamiento de solicitud, traducción de protocolos, equilibrio de carga o autenticación. Sin embargo, la puerta de enlace debe estar restringida a este tipo de funcionalidad de infraestructura. No debe implementar ningún conocimiento del dominio para evitar convertirse en una dependencia intensa.

**Exponer interfaces abiertas**. Evite crear capas de traducción personalizadas que residan entre los servicios. En su lugar, un servicio debe exponer una API con un contrato de API bien definido. La API debería ser tener control de versiones, para que la API pueda evolucionar mientras se mantiene la compatibilidad con versiones anteriores. De este modo, puede actualizar un servicio sin coordinar las actualizaciones de todos los servicios ascendentes que dependen de él. Los servicios orientados al público deben exponer una API de RESTful a través de HTTP. Los servicios back-end pueden usar un protocolo de mensajería de estilo RPC por motivos de rendimiento. 

**Diseñar y probar mediante contratos de servicio**. Cuando los servicios exponen unas API bien definidas, puede desarrollar y probar mediante estas API. De este modo, puede desarrollar y probar un servicio individual sin poner en marcha todos sus servicios dependientes. (Por supuesto, seguiría realizando la integración y la prueba de carga en los servicios reales.)

**Infraestructura abstracta fuera de la lógica del dominio**. No permita que la lógica del dominio se mezcle con una funcionalidad relacionada con la infraestructura, como la mensajería o la persistencia. En caso contrario, los cambios en la lógica del dominio requerirán actualizaciones en las capas de infraestructura y viceversa. 

**Descargar cuestiones transversales en un servicio independiente**. Por ejemplo, si varios servicios necesitan autenticar solicitudes, puede mover esta funcionalidad a su propio servicio. Puede desarrollar luego el servicio de autenticación &mdash;por ejemplo, agregando un nuevo flujo de autenticación&mdash; sin tocar ninguno de los servicios que lo usan.

**Implementar servicios de forma independiente**. Cuando el equipo de DevOps puede implementar un único servicio independientemente de otros servicios de la aplicación, las actualizaciones pueden realizarse de forma más rápida y segura. Las correcciones de errores y las nuevas características pueden implementarse a un ritmo más regular. Diseñe la aplicación y el proceso de lanzamiento para que admitan actualizaciones independientes.
