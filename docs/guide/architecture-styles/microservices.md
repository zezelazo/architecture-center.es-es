---
title: Estilo de arquitectura de microservicios
description: Describe las ventajas, las dificultades y los procedimientos recomendados para las arquitecturas de microservicios en Azure.
author: MikeWasson
ms.date: 11/13/2018
ms.openlocfilehash: 4e5d50f829323829c953977257e690354566ebf6
ms.sourcegitcommit: 19a517a2fb70768b3edb9a7c3c37197baa61d9b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/26/2018
ms.locfileid: "52295538"
---
# <a name="microservices-architecture-style"></a>Estilo de arquitectura de microservicios

Una arquitectura de microservicios consta de una colección de servicios autónomos y pequeños. Los servicios son independientes entre sí y cada uno debe implementar una funcionalidad de negocio individual. 

![](./images/microservices-logical.svg)
 
En cierto modo, los microservicios son la evolución natural de las arquitecturas orientadas a servicios, pero hay diferencias entre los microservicios y estas arquitecturas. Estas son algunas de las características que definen un microservicio:

- En una arquitectura de microservicios, los servicios son pequeños e independientes y están acoplados de forma flexible.

- Cada servicio es un código base independiente, que puede administrarse por un equipo de desarrollo pequeño.

- Los servicios pueden implementarse de manera independiente. Un equipo puede actualizar un servicio existente sin tener que volver a generar e implementar toda la aplicación.

- Los servicios son los responsables de conservar sus propios datos o estado externo. Esto difiere del modelo tradicional, donde una capa de datos independiente controla la persistencia de los datos.

- Los servicios se comunican entre sí mediante API bien definidas. Los detalles de la implementación interna de cada servicio se ocultan frente a otros servicios.

- No es necesario que los servicios compartan la misma pila de tecnología, las bibliotecas o los marcos de trabajo.

Además de los propios servicios, hay otros componentes que aparecen en una arquitectura típica de microservicios:

**Administración**. El componente de administración es responsable de la colocación de servicios en los nodos, la identificación de errores, el reequilibrio de servicios entre nodos, etc.  

**Detección de servicios**.  Mantiene una lista de servicios y los nodos en que se encuentran. Permite la búsqueda de servicios para localizar el punto de conexión de un servicio. 

**Puerta de enlace de API** La puerta de enlace de API es el punto de entrada para los clientes. Los clientes no llaman directamente a los servicios. En su lugar, llaman a la puerta de enlace de API, que reenvía la llamada a los servicios apropiados en el back-end. La puerta de enlace de API podría agregar las respuestas de varios servicios y devolver la respuesta agregada. 

Entre las ventajas del uso de una puerta de enlace de API se encuentran las siguientes:

- Desacoplan los clientes de los servicios. Los servicios pueden cambiar de versión o refactorizarse sin necesidad de actualizar todos los clientes.

-  Los servicios pueden utilizar los protocolos de mensajería que no son fáciles de usar para un servicio web, como AMQP.

- La puerta de enlace de API puede realizar otras funciones transversales como la autenticación, el registro, la terminación SSL y el equilibrio de carga.

## <a name="when-to-use-this-architecture"></a>Cuándo utilizar esta arquitectura

Considere este estilo de arquitectura para:

- Aplicaciones grandes que requieren una alta velocidad de publicación.

- Aplicaciones complejas que necesitan gran escalabilidad.

- Aplicaciones con dominios complejos o muchos subdominios.

- Una organización que disponga de pequeños equipos de desarrollo.


## <a name="benefits"></a>Ventajas 

- **Implementaciones independientes**. Puede actualizar un servicio sin volver a implementar toda la aplicación y revertir o poner al día una actualización si algo va mal. Las correcciones de errores y las publicaciones de características son más fáciles de administrar y entrañan menos riesgo.

- **Desarrollo independiente**. Un solo equipo de desarrollo puede compilar, probar e implementar un servicio. El resultado es la innovación continua y un ritmo más rápido de publicación. 

- **Equipos pequeños y centrados**. Los equipos pueden centrarse en un servicio. El menor ámbito de cada servicio hace que la base del código sea más fácil de entender, con lo que los nuevos miembros de los equipos pueden ponerse al día con mayor facilidad.

- **Aislamiento de errores**. Si un servicio deja de funcionar, no es necesario paralizar toda la aplicación. Sin embargo, eso no significa que obtenga resistencia sin nada a cambio. Tendrá que seguir los procedimientos recomendados de resistencia y los patrones de diseño. Vea [Diseño de aplicaciones resistentes de Azure][resiliency-overview].

- **Pilas de tecnología mixta**. Los equipos pueden elegir la tecnología que mejor se adapte a su servicio. 

- **Escalado pormenorizado**. Los servicios pueden escalarse de manera independiente. Al mismo tiempo, la mayor densidad de servicios por máquina virtual implica que los recursos de máquinas virtuales funcionan a máxima capacidad. Con las restricciones de posición, un servicio puede asociarse con un perfil de VM (alta CPU, alta memoria, etc.).

## <a name="challenges"></a>Desafíos

- **Complejidad**. Una aplicación de microservicios tiene más partes en movimiento que la aplicación monolítica equivalente. Cada servicio es más sencillo, pero el sistema como un todo es más complejo.

- **Desarrollo y pruebas**. El desarrollo con dependencias de servicios requiere un enfoque diferente. Las herramientas existentes no están necesariamente diseñadas para trabajar con dependencias de servicios. La refactorización en los límites del servicio puede resultar difícil. También supone un desafío probar las dependencias de los servicios, especialmente cuando la aplicación está evolucionando rápidamente.

- **Falta de gobernanza**. El enfoque descentralizado para la generación de microservicios tiene ventajas, pero también puede causar problemas. Puede acabar con tantos lenguajes y marcos de trabajo diferentes que la aplicación puede ser difícil de mantener. Puede resultar útil establecer algunos estándares para todo el proyecto sin restringir excesivamente la flexibilidad de los equipos. Esto se aplica especialmente a las funcionalidades transversales, como el registro.

- **Congestión y latencia de red**. El uso de muchos servicios pequeños y detallados puede dar lugar a más comunicación interservicios. Además, si la cadena de dependencias del servicio se hace demasiado larga (el servicio A llama a B, que llama a C...), la latencia adicional puede constituir un problema. Tendrá que diseñar las API con atención. Evite que las API se comuniquen demasiado, piense en formatos de serialización y busque lugares para utilizar patrones de comunicación asincrónica.

- **Integridad de datos**. Cada microservicio es responsable de la conservación de sus propios datos. Como consecuencia, la coherencia de los datos puede suponer un problema. Adopte una coherencia final cuando sea posible.

- **Administración**. Para tener éxito con los microservicios se necesita una cultura de DevOps consolidada. El registro correlacionado entre servicios puede resultar un desafío. Normalmente, el registro debe correlacionar varias llamadas de servicio para una sola operación de usuario.

- **Control de versiones**. Las actualizaciones de un servicio no deben interrumpir servicios que dependen de él. Es posible que varios servicios se actualicen en cualquier momento; por lo tanto, sin un cuidadoso diseño, podrían surgir problemas con la compatibilidad con versiones anteriores o posteriores.

- **Conjunto de habilidades**. Los microservicios son sistemas muy distribuidos. Evalúe cuidadosamente si el equipo tiene los conocimientos y la experiencia para desenvolverse correctamente.

## <a name="best-practices"></a>Procedimientos recomendados

- Adapte los servicios al dominio empresarial. 

- Descentralice todo. Los equipos individuales son responsables de diseñar y compilar servicios. Evite el uso compartido de código o esquemas de datos. 

- El almacenamiento de datos debería ser privado para el servicio que posee los datos. Use el almacenamiento recomendado para cada tipo de servicio y de datos. 

- Los servicios se comunican a través de API bien diseñadas. Evite la pérdida de detalles de la implementación. Las API deben modelar el dominio, no la implementación interna del servicio.

- Evite el acoplamiento entre servicios. Entre las causas de acoplamiento se encuentran los protocolos de comunicación rígidos y los esquemas de bases de datos compartidos.

- Descargue a la puerta de enlace de cuestiones transversales, como la autenticación y la terminación SSL.

- Conserve el conocimiento del dominio fuera de la puerta de enlace. La puerta de enlace debe controlar y enrutar las solicitudes de cliente sin ningún conocimiento de las reglas de negocios o la lógica de dominio. En caso contrario, la puerta de enlace se convierte en una dependencia y puede provocar el acoplamiento entre servicios.

- Los servicios deben tener un acoplamiento flexible y una alta cohesión funcional. Las funciones que es probable que cambien juntas se deben empaquetar e implementar en conjunto. Si residen en distintos servicios, estos terminan estrechamente acoplados, porque un cambio en un servicio requerirá la actualización del otro servicio. La comunicación demasiado intensa entre dos servicios puede ser un síntoma de un acoplamiento estrecho y una cohesión baja. 

- Aísle los errores. Utilice estrategias de resistencia para impedir que los errores dentro de un servicio se reproduzcan en cascada. Vea [Resiliency patterns][resiliency-patterns] (Patrones de resistencia) y [Diseño de aplicaciones resistentes de Azure][resiliency-overview].

## <a name="next-steps"></a>Pasos siguientes

Para obtener instrucciones detalladas sobre cómo crear una arquitectura de microservicios en Azure, consulte [Designing, building, and operating microservices on Azure](../../microservices/index.md) (Diseño, creación y funcionamiento de microservicios en Azure).


<!-- links -->

[resiliency-overview]: ../../resiliency/index.md
[resiliency-patterns]: ../../patterns/category/resiliency.md



