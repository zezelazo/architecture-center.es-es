---
title: Diseño, compilación y operación de microservicios en Azure con Kubernetes
description: Diseño, compilación y operación de microservicios en Azure
author: MikeWasson
ms.date: 10/23/2018
ms.openlocfilehash: cac16c9212432c72aeaecac1a578828a00838431
ms.sourcegitcommit: fdcacbfdc77370532a4dde776c5d9b82227dff2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49962779"
---
# <a name="designing-building-and-operating-microservices-on-azure"></a>Diseño, compilación y operación de microservicios en Azure

![](./images/drone.svg)

Los microservicios se han convertido en un estilo arquitectónico popular para la compilación de aplicaciones en la nube resistentes, altamente escalables, que se pueden implementar de forma independiente y que evolucionan rápidamente. Sin embargo, para que sean algo más que la palabra de moda, los microservicios requieren un enfoque diferente para diseñar y compilar aplicaciones. 

En esta serie de artículos, exploramos cómo compilar y ejecutar una arquitectura de microservicios en Azure. Contenido de los temas:

- Diseño basado en el dominio (DDD) para una arquitectura de microservicios. 
- Tecnologías de Azure adecuadas para los procesos, el almacenamiento, la mensajería y otros elementos del diseño.
- Descripción de los patrones de diseño de los microservicios.
- Diseño para conseguir resistencia, escalabilidad y rendimiento.
- Compilación de una canalización de integración y entrega continuas.


En resumen, nos centraremos en un escenario completo: un servicio de entrega mediante drones que permite a los clientes programar la recogida y la entrega de paquetes mediante un dron. Puede encontrar el código de nuestra implementación de referencia en GitHub

[![GitHub](../_images/github.png) Implementación de referencia][drone-ri]

Pero primero, empecemos con los conceptos básicos. ¿Qué son los microservicios y cuáles son las ventajas de adoptar una arquitectura de microservicios?

## <a name="why-build-microservices"></a>¿Por qué compilar microservicios?

En una arquitectura de microservicios, la aplicación se compone de muchos servicios pequeños e independientes. Estas son algunas de las características que definen los microservicios:

- Cada servicio implementa una sola funcionalidad empresarial.
- Un microservicio es lo suficientemente pequeño como para que un único equipo reducido de programadores lo pueda escribir y mantener.
- Los microservicios se ejecutan en procesos independientes, que se comunican mediante API bien definidas o patrones de mensajería. 
- Los microservicios no comparten los almacenes de datos ni los esquemas de datos. Cada microservicio es responsable de administrar sus propios datos. 
- Los microservicios tienen bases de código independientes y no comparten el código fuente. Sin embargo, pueden usar bibliotecas de utilidades comunes.
- Cada microservicio se puede implementar y actualizar independientemente de otros servicios. 

Si se realiza correctamente, los microservicios proporcionan una serie de beneficios útiles:

- **Agilidad.** Dado que microservicios se implementan de forma independiente, resulta más fácil de administrar las correcciones de errores y las versiones de características. Puede actualizar un servicio sin volver a implementar toda la aplicación y revertir una actualización si algo va mal. En muchas aplicaciones tradicionales, un error en una parte de la aplicación puede bloquear el proceso de la versión completa; como resultado, las nuevas características se pueden mantener la espera de la integración, prueba y publicación de una corrección de errores.  

- **Código pequeño, equipos pequeños.** Un microservicio debe ser lo suficientemente pequeño como para que un solo equipo de características lo pueda compilar, probar e implementar. Las bases de código pequeño son fáciles de entender. En una gran aplicación monolítica, las dependencias de código tienden a enredarse con el tiempo, por consiguiente, para agregar una nueva característica se necesita tocar el código en muchos lugares. Al no compartir el código ni los almacenes de datos, la arquitectura de microservicios minimiza las dependencias y resulta más fácil agregar nuevas características. Los equipos pequeños favorecen la agilidad. La regla de las dos pizzas establece que el equipo debe ser lo suficientemente pequeño para que se alimente con dos pizzas. Obviamente, no es una medida exacta y depende del apetito del equipo. Pero lo que se quiere decir es que los grupos grandes suelen ser menos productivos, porque la comunicación es más lenta, la administración se sobrecarga y la agilidad disminuye.  

- **Mezcla de tecnologías**. Los equipos pueden elegir la tecnología que mejor se adapte al servicio de una combinación de pilas de tecnología, según corresponda. 

- **Resistencia**. Si un microservicio individual no está disponible, no interrumpe toda la aplicación, siempre que los microservicios de nivel superior estén diseñados para controlar los errores correctamente (por ejemplo, mediante la implementación de disyuntores).

- **Escalabilidad**. Una arquitectura de microservicios permite el escalado independiente de cada microservicio. Esto permite escalar horizontalmente los subsistemas que requieren más recursos, sin tener que escalar horizontalmente toda la aplicación. Si implementa servicios dentro de contenedores, también puede empaquetar una mayor densidad de microservicios en un solo host, lo que permite un uso más eficaz de los recursos.

- **Aislamiento de los datos**. Al verse afectado solo un microservicio, es mucho más fácil realizar actualizaciones del esquema. En una aplicación monolítica, las actualizaciones del esquema pueden ser muy complicadas, ya que las distintas partes de la aplicación pueden tocar los mismos datos, por lo que realizar modificaciones en el esquema resulta peligroso.
 
## <a name="no-free-lunch"></a>Esto no es gratis

Estas ventajas tienen un precio. Esta serie de artículos está diseñada para abordar algunos de los desafíos de la compilación de microservicios que son flexibles, escalables y fáciles de administrar.

- **Límites de servicio**. Al compilar microservicios debe pensar detenidamente dónde debe marcar los límites entre los servicios. Una vez compilados e implementados en producción los servicios, pueden resultar difíciles de refactorizar más allá de los límites. Elegir los límites de servicio adecuados es uno de los mayores desafíos al diseñar una arquitectura de microservicios. ¿Qué tamaño debe tener cada servicio? ¿Cuándo se debe factorizar una funcionalidad entre varios servicios y cuándo debe permanecer en un servicio? En esta guía se describe una estrategia que utiliza el diseño basado en el dominio para determinar los límites del servicio. Comienza con el [análisis del dominio](./domain-analysis.md) para buscar los contextos enlazados y aplica un conjunto de [patrones tácticos de diseño basado en dominios](./microservice-boundaries.md) en función de los requisitos funcionales y no funcionales. 

- **Coherencia e integridad de los datos**. Un principio básico de los microservicios es que cada servicio administra sus propios datos. Esto mantiene los servicios desacoplados, pero puede dar lugar a dificultades de integridad o redundancia de los datos. Exploramos algunos de estos problemas en las [consideraciones sobre los datos](./data-considerations.md).

- **Congestión y latencia de red**. El uso de muchos servicios pequeños y pormenorizados puede facilitar la comunicación entre servicios y alargar la latencia de un extremo a otro. En el capítulo sobre la [comunicación entre servicios](./interservice-communication.md) se describen las consideraciones para la mensajería entre servicios. La comunicación sincrónica y asincrónica tienen su utilidad en las arquitecturas de microservicios. Se necesita un buen [diseño de API](./api-design.md) para que los servicios permanezcan flexiblemente acoplados y para que se puedan implementar y actualizar de manera independiente.
 
- **Complejidad**. Una aplicación de microservicios tiene más partes móviles. Los servicios pueden ser sencillos, pero deben trabajar juntos como un todo. Una única operación de usuario puede abarcar varios servicios. En el capítulo sobre [ingesta y flujo de trabajo](./ingestion-workflow.md) examinaremos algunos de los problemas relacionados con la ingesta de las solicitudes con gran rendimiento, la coordinación de un flujo de trabajo y el control de errores. 

- **Comunicación entre los clientes y la aplicación.**  Al descomponer una aplicación en muchos servicios pequeños, ¿cómo deben los clientes comunicarse con esos servicios? ¿Debe un cliente llamar directamente a cada servicio individual o enrutar las solicitudes mediante una [puerta de enlace de API](./gateway.md)?

- **Supervisión**. La supervisión de una aplicación distribuida puede ser mucho más difícil que la de una aplicación monolítica, ya que es necesario correlacionar telemetría de varios servicios. En el capítulo sobre [supervisión y registro](./logging-monitoring.md) se abordan estos problemas.

- **Integración y entrega continuas (CI/CD)**. Uno de los objetivos principales de los microservicios es la agilidad. Para lograrla, debe contar con [integración y entrega continuas](./ci-cd.md) automatizadas y sólidas, de modo que pueda implementar rápidamente y de forma confiable los servicios individuales en entornos de prueba y producción.

## <a name="the-drone-delivery-application"></a>Aplicación Delivery de Drone

Para explorar estos problemas e ilustrar algunos de los procedimientos recomendados para una arquitectura de microservicios, hemos creado una implementación de referencia que denominamos aplicación Delivery de Drone. Puede encontrar la implementación de referencia en [GitHub][drone-ri].

Fabrikam, Inc. está iniciando un servicio de entrega con drones. La empresa administra una flota de drones. Las empresas se registran en el servicio y los usuarios pueden solicitar que un dron recoja los bienes para la entrega. Cuando un cliente programa una recogida, un sistema back-end asigna un dron y notifica al usuario con un tiempo de entrega estimado. Con la entrega en curso, el cliente puede realizar el seguimiento del dron, con una fecha estimada que se actualiza constantemente.

Este escenario implica un dominio bastante complicado. Algunas de las cuestiones empresariales incluyen la programación de los drones, el seguimiento de los paquetes, la administración de las cuentas de usuario, y el almacenamiento y análisis de los datos históricos. Además, Fabrikam desea salir al mercado y evolucionar rápidamente con la incorporación de funciones y funcionalidades nuevas. La aplicación necesita operar en la nube, con un objetivo de nivel de servicio alto. Fabrikam también prevé que distintas partes del sistema tendrán requisitos muy diferentes respecto al almacenamiento y la consulta de datos. Todas estas consideraciones hacen que Fabrikam se decante por una arquitectura de microservicios para la aplicación Delivery de Drone.

> [!NOTE]
> Para ayuda para elegir entre una arquitectura de microservicios y otros estilos de arquitectura, consulte la [Guía de la arquitectura de aplicaciones en Azure](../guide/index.md).

Nuestra implementación de referencia usa Kubernetes con [Azure Kubernetes Service](/azure/aks/) (AKS). Sin embargo, muchos de los desafíos y las decisiones de arquitectura de alto nivel se aplicarán a cualquier orquestador de contenedor, incluido [Azure Service Fabric](/azure/service-fabric/). 

> [!div class="nextstepaction"]
> [Análisis del dominio](./domain-analysis.md)


<!-- links -->

[drone-ri]: https://github.com/mspnp/microservices-reference-implementation
