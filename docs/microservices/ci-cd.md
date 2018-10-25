---
title: CI/CD para microservicios
description: Integración continua y entrega continua para microservicios
author: MikeWasson
ms.date: 10/23/2018
ms.openlocfilehash: b411e687a111e55a5821d4fdc66975e80f73584b
ms.sourcegitcommit: fdcacbfdc77370532a4dde776c5d9b82227dff2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49962864"
---
# <a name="designing-microservices-continuous-integration"></a>Diseño de microservicios: integración continua

La integración continua y la entrega continua (CI/CD) son un requisito clave para el buen funcionamiento de los microservicios. Sin un buen proceso de CI/CD, no disfrutará de la agilidad que los microservicios prometen. Algunos de los desafíos de CI/CD para microservicios surgen debido a la existencia de varias bases de código y de entornos de compilación heterogéneos para los diversos servicios. En este capítulo se describen los desafíos y se recomiendan algunas estrategias para resolver los problemas.

![](./images/ci-cd.png)

Una de las principales razones para adoptar una arquitectura de microservicios es la existencia de ciclos de lanzamiento más rápidos. 

En una aplicación completamente monolítica, hay una canalización de compilación única cuya salida es el ejecutable de la aplicación. Todo el trabajo de desarrollo se envía a esta canalización. Si se encuentra un error de alta prioridad, debe integrarse, probarse y publicarse una corrección, lo que puede retrasar el lanzamiento de nuevas características. Es cierto que puede mitigar estos problemas con módulos correctamente factorizados y el uso de ramas de características para minimizar el impacto de los cambios de código. No obstante, a medida que la aplicación se hace más compleja y se van añadiendo más características, el proceso de publicación de un monolito tiende a hacerse más frágil, con lo que tiene más posibilidades de presentar errores. 

Según la filosofía de los microservicios, nunca debería haber una larga serie de versiones en la que todos los equipos tengan que hacer cola. El equipo que compila el servicio "A" podrá publicar una actualización en cualquier momento, sin tener que esperar a que los cambios realizados en el servicio "B" se combinen, prueben e implementen. El proceso de CI/CD es fundamental para que esto sea posible. La canalización de versión debe estar automatizada y ser altamente fiable, de forma que el riesgo de tener que implementar actualizaciones se reduzca al mínimo. Si va a promover versiones a producción una o varias veces al día, las regresiones e interrupciones del servicio deben ser muy poco frecuentes. Al mismo tiempo, si se implementa una actualización con errores, debe disponer de un método fiable para ponerla al día o revertirla rápidamente a una versión anterior de un servicio.

![](./images/cicd-monolith.png)

Cuando hablamos de CI/CD, estamos hablando realmente de varios procesos relacionados: la integración continua, la entrega continua y la implementación continua.

- La integración continua implica que los cambios en el código se combinarán frecuentemente en la rama principal mediante procesos de compilación y prueba automatizados, a fin de garantizar que el código de la rama principal siempre tenga calidad de producción.

- La entrega continua hace que los cambios en el código que pasan el proceso de integración continua se publiquen automáticamente en un entorno similar a la producción. La implementación en el entorno de producción real puede requerir aprobación manual, pero, en caso contrario, se realiza automáticamente. El objetivo es que el código siempre esté *listo* para implementarse en producción.

- La implementación continua hace que los cambios en el código que pasan el proceso de CI/CD se implementen automáticamente en producción.

En el contexto de Kubernetes y de los microservicios, durante la fase de integración continua se compilan y prueban imágenes de contenedor, además de insertar esas imágenes en un registro de contenedor. En la fase de implementación, se actualizan las especificaciones de pod para recoger la imagen de producción más reciente.

## <a name="challenges"></a>Desafíos

- **Muchas bases de código pequeñas e independientes**. Cada equipo es responsable de compilar su propio servicio, con su propia canalización de compilación. En algunas organizaciones, los equipos pueden usar repositorios de código distintos. Esto podría provocar una situación en la que los diferentes equipos se especialicen en una parte distinta del proceso de compilación del sistema, pero nadie en la organización sepa realmente cómo implementar la aplicación completa. Por ejemplo, ¿qué ocurriría en un escenario de recuperación ante desastres si fuera necesario implementar rápidamente en un nuevo clúster?   

- **Varios lenguajes y plataformas**. Si cada equipo usa su propio conjunto de tecnologías, puede ser difícil crear un único proceso de compilación que sirva para toda la organización. El proceso de compilación debe ser lo suficientemente flexible como para que cada equipo pueda adaptarlo al lenguaje o la plataforma que utiliza. 

- **Integración y pruebas de carga**. Si cada equipo publica actualizaciones a su propio ritmo, puede resultar complicado diseñar una solución de pruebas integral sólida, especialmente si los servicios tienen dependencias en otros servicios. Además, poner en marcha un clúster de producción completo puede ser costoso, por lo que resulta poco probable que cada equipo pueda ejecutar su propio clúster completo a escala de producción solo para realizar pruebas. 

- **Administración de versiones**. Cada equipo debe tener la capacidad de implementar una actualización en producción. Eso no significa que cada miembro del equipo tenga permiso para hacerlo. Sin embargo, contar con un rol de administrador de versiones centralizado puede reducir la velocidad de las implementaciones. Cuanto más automatizado y fiable sea el proceso de CI/CD, menor será la necesidad de contar con una autoridad central. No obstante, puede tener una directiva para publicar actualizaciones de características importantes y otra distinta para publicar correcciones de errores menores. Un enfoque descentralizado no implica la ausencia completa de autoridad.

- **Control de versiones de imágenes de contenedor**. Durante el ciclo de desarrollo y pruebas, el proceso de CI/CD compilará muchas imágenes de contenedor. Solo algunas de ellas serán candidatas a publicarse y, de estas, solo algunas se enviarán a producción. Debe tener una estrategia de control de versiones clara, para saber qué imágenes se están implementando actualmente en producción y pueda revertir a una versión anterior si es necesario. 

- **Actualizaciones del servicio**. Cuando se actualiza un servicio a una nueva versión, no debería interrumpir otros servicios que dependen de él. Si hace una actualización gradual, durante un tiempo se ejecutarán distintas versiones a la vez. 
 
Estos desafíos reflejan una tensión fundamental. Por una parte, los equipos tienen que trabajar con la mayor independencia posible. Por otro lado, se necesita cierta coordinación para que una sola persona pueda realizar tareas como ejecutar una prueba de integración, volver a implementar toda la solución en un nuevo clúster o revertir una actualización con errores. 
 
## <a name="cicd-approaches-for-microservices"></a>Estrategias de CI/CD para microservicios

Es una buena práctica que cada equipo de servicio incluya su entorno de compilación en un contenedor. Este contenedor debe tener todas las herramientas de compilación necesarias para compilar los artefactos de código para su servicio. A menudo puede encontrar una imagen de Docker oficial para su lenguaje y plataforma. Puede usar `docker run` o Docker Compose para ejecutar la compilación. 

Con este enfoque, es muy fácil configurar un nuevo entorno de compilación. Un desarrollador que desee compilar el código no tendrá que instalar un conjunto de herramientas de compilación, sino que solo tendrá que ejecutar la imagen de contenedor. Y, lo que puede ser más importante: podrá configurarse el servidor de compilación para que haga lo mismo. De este modo, no necesita instalar esas herramientas en el servidor de compilación ni administrar versiones en conflicto de herramientas. 

Para el desarrollo y las pruebas locales, use Docker para ejecutar el servicio dentro de un contenedor. Como parte de este proceso, puede que tenga que ejecutar otros contenedores que tienen servicios ficticios o bases de datos de prueba necesarios para realizar pruebas locales. Podría utilizar Docker Compose para coordinar estos contenedores, o usar Minikube para ejecutar Kubernetes localmente. 

Cuando el código esté listo, abra una solicitud de incorporación de cambios y combínelo con la versión maestra. Esto iniciará un trabajo en el servidor de compilación:

1. Compile los recursos del código. 
2. Ejecute pruebas unitarias en el código.
3. Compile la imagen de contenedor.
4. Pruebe la imagen de contenedor mediante la ejecución de pruebas funcionales en un contenedor en ejecución. Este paso puede detectar errores en el archivo de Docker, como, por ejemplo, un punto de entrada incorrecto.
5. Inserte la imagen en el registro de contenedor.
6. Actualice el clúster de prueba con la nueva imagen para ejecutar pruebas de integración.

Cuando la imagen esté lista para pasar a producción, actualice los archivos de implementación según sea necesario para especificar la imagen más reciente, incluidos los archivos de configuración de Kubernetes. A continuación, aplique la actualización al clúster de producción.

A continuación, se incluyen diversas recomendaciones para que las implementaciones sean más fiables:
 
- Defina convenciones para toda la organización de etiquetas contenedoras, control de versiones y nomenclatura para los recursos implementados en el clúster (pods, servicios, etc.). Esto puede ayudar a diagnosticar problemas de implementación más fácilmente. 

- Cree dos registros de contenedor independientes, uno para desarrollo/pruebas y otro para producción. No inserte una imagen en el registro de producción hasta que esté listo para implementarla en producción. Si combina esta práctica con el versionamiento semántico de imágenes de contenedor, puede reducir la posibilidad de implementar accidentalmente una versión que no se ha aprobado para su publicación.

## <a name="updating-services"></a>Actualización de los servicios

Existen diversas estrategias de actualización de un servicio que ya está en producción. Aquí analizaremos tres de las más frecuentes: la actualización gradual, la implementación azul-verde y el lanzamiento controlado.

### <a name="rolling-update"></a>Actualización gradual 

En una actualización gradual, se implementan nuevas instancias de un servicio y las instancias nuevas empiezan a recibir solicitudes de forma inmediata. A medida que llegan nuevas instancias, se eliminan las anteriores.

Las actualizaciones graduales son el comportamiento predeterminado en Kubernetes cuando se actualiza la especificación de pod para una implementación. El controlador de implementación crea un ReplicaSet nuevo para los pods actualizados. A continuación, se escala verticalmente el nuevo ReplicaSet mientras se reduce verticalmente el anterior a fin de mantener el número de réplicas deseado. Los pods antiguos no se eliminarán hasta que los nuevos estén listos. Kubernetes mantiene un historial de la actualización, por lo que puede usar kubectl para revertir una actualización si es necesario. 

Si el servicio lleva a cabo una tarea de inicio prolongada, puede definir un sondeo de disponibilidad. El sondeo de disponibilidad indica si el contenedor está listo para empezar a recibir tráfico. Kubernetes no enviará tráfico al pod hasta que el sondeo devuelva un resultado satisfactorio. 

Las actualizaciones graduales plantean el desafío de que, durante el proceso de actualización, se ejecuta una combinación de versiones antiguas y nuevas que reciben tráfico. Durante este período, una solicitud podría dirigirse a cualquiera de las dos versiones. Esto podría ser problemático, en función del alcance de los cambios entre las dos versiones. 

### <a name="blue-green-deployment"></a>Implementación azul-verde

En una implementación azul-verde, se implemente la nueva versión junto con la versión anterior. Después de validar la nueva versión, se cambia todo el tráfico a la vez desde la versión anterior a la nueva versión. Tras el cambio, deberá supervisar la aplicación para detectar posibles problemas. Si algo va mal, puede volver a la versión anterior. Si no hubiera ningún problema, puede eliminar la versión anterior.

Con una aplicación de n niveles o monolítica más tradicional, la implementación azul-verde normalmente implicaba el aprovisionamiento de dos entornos idénticos. La nueva versión se implementaría en un entorno de ensayo y se redirigiría el tráfico del cliente al entorno de ensayo &mdash;, por ejemplo, intercambiando las direcciones IP virtuales.

En Kubernetes, no es necesario aprovisionar un clúster diferente para realizar implementaciones azul-verde. En su lugar, puede utilizar selectores. Cree un nuevo recurso de implementación con una nueva especificación de pod y un conjunto diferente de etiquetas. Cree esta implementación sin eliminar la implementación anterior ni modificar el servicio que hace referencia a ella. Una vez que se ejecutan los nuevos pods, puede actualizar el selector del servicio para que se corresponda con la nueva implementación. 

Una de las ventajas de las implementaciones azul-verde es que el servicio cambia todos los pods al mismo tiempo. Después de haber actualizado el servicio, todas las nuevas solicitudes se dirigirán a la nueva versión. Una desventaja es que durante la actualización, se ejecuta el doble de pods para el servicio (actual y siguiente). Si los pods requieren una gran cantidad de recursos de CPU o memoria, debe escalar horizontalmente el clúster de forma temporal para controlar el consumo de recursos. 

### <a name="canary-release"></a>Lanzamiento controlado

En un lanzamiento controlado, se implementa una versión actualizada en un pequeño número de clientes. A continuación, supervisará el comportamiento del servicio nuevo antes de distribuirlo a todos los clientes. Esto le permitirá realizar una implementación lenta de un modo controlado, observar datos reales y detectar problemas antes de que se ven afectados todos los clientes.

Un lanzamiento controlado es más complejo de administrar que una actualización gradual o azul-verde, ya que debe dirigir de forma dinámica las solicitudes a versiones distintas del servicio. En Kubernetes, puede configurar un servicio para abarcar dos conjuntos de réplicas (uno para cada versión) y ajustar manualmente el número de réplicas. Sin embargo, este enfoque es bastante genérico, debido a la forma en que Kubernetes equilibra la carga entre los pods. Por ejemplo, si tiene un total de diez réplicas, solo puede cambiar el tráfico en incrementos del 10 %. Si utiliza una malla de servicio, puede utilizar las reglas de enrutamiento de malla de servicio para implementar una estrategia de lanzamiento controlado más sofisticada. Aquí tiene algunos recursos que pueden serle de ayuda:

- Kubernetes sin malla de servicio: [Implementaciones de lanzamiento controlado](https://kubernetes.io/docs/concepts/cluster-administration/manage-deployment/#canary-deployments)
- Linkerd: [Enrutamiento de solicitud dinámica](https://linkerd.io/features/routing/)
- Istio: [Implementaciones de lanzamiento controlado con Istio](https://istio.io/blog/canary-deployments-using-istio.html)

## <a name="conclusion"></a>Conclusión

En los últimos años, ha habido un cambio drástico en el sector, al pasar de la creación de *sistemas de registro* a la creación de *sistemas de interacción*.

Los sistemas de registro son aplicaciones de administración de datos del área de operaciones tradicionales. Estos sistemas se basan en un sistema de administración de bases de datos relacionales, que es el único origen de datos. El término "sistema de interacción" (System of Engagement) se atribuye a Geoffrey Moore, que lo acuñó en su trabajo *Systems of Engagement and the Future of Enterprise IT* de 2011. Los sistemas de interacción son aplicaciones que se centran en la comunicación y la colaboración. Conectan personas en tiempo real. Deben estar disponibles en todo momento. Se introducen periódicamente nuevas características sin tener que desconectar la aplicación. Los usuarios tienen mayores expectativas y menos paciencia con los retrasos o tiempos de inactividad inesperados.

Para los consumidores, una mejor experiencia de usuario puede tener un valor cuantificable en el negocio. La cantidad de tiempo que un usuario interactúe con una aplicación puede traducirse directamente en ingresos. Además, en el mundo de los sistemas empresariales, las expectativas de los usuarios han cambiado. Si estos sistemas tienen como objetivo fomentar la comunicación y la colaboración, deben seguir el ejemplo de aplicaciones orientadas al consumidor.

Los microservicios ofrecen una respuesta a este panorama cambiante. Al descomponer una aplicación monolítica en un grupo de servicios con acoplamiento flexible, podemos controlar el ciclo de versiones de cada servicio y habilitar actualizaciones frecuentes sin tiempo de inactividad ni cambios drásticos. Los microservicios también ayudan con la escalabilidad, el aislamiento de errores y la resistencia. Mientras tanto, las plataformas en la nube permiten compilar y ejecutar microservicios cada vez más fácilmente, con aprovisionamiento automatizado de recursos de proceso, orquestadores de contenedor como servicio y entornos sin servidor basados en eventos.

No obstante, como hemos visto, las arquitecturas de microservicios también presentan una gran cantidad de desafíos. Para tener éxito, debe empezar con un diseño sólido. Debe pensar detenidamente cómo analizar el dominio, elegir las tecnologías, modelar datos, diseñar API y crear una referencia cultural de DevOps consolidada. Esperamos que esta guía y la [implementación de referencia](https://github.com/mspnp/microservices-reference-implementation) adjunta le hayan ayudado a aclarar el proceso. 

