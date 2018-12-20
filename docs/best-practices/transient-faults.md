---
title: Orientación general sobre reintentos
titleSuffix: Best practices for cloud applications
description: Guía de reintentos para el control de errores transitorios.
author: dragon119
ms.date: 07/13/2016
ms.custom: seodec18
ms.openlocfilehash: fe07364e1a6846f9b7b47b2b79ce8031122edbbd
ms.sourcegitcommit: 4ba3304eebaa8c493c3e5307bdd9d723cd90b655
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/12/2018
ms.locfileid: "53307119"
---
# <a name="transient-fault-handling"></a>Control de errores transitorios

Todas las aplicaciones que se comunican con los recursos y servicios remotos deben ser sensibles a errores transitorios. Esto se aplica especialmente en el caso de aplicaciones que se ejecutan en la nube, donde la naturaleza del entorno y la conectividad a través de Internet significan que estos tipos de errores suelen encontrarse con más frecuencia. Los errores transitorios incluyen la pérdida momentánea de conectividad de red en componentes y servicios, la indisponibilidad temporal de un servicio o tiempos de espera que surgen cuando un servicio está ocupado. Estos errores suelen ser de corrección automática y si se repite la acción después de un intervalo adecuado, es probable que se realice correctamente.

En este documento se cubre la guía general de control de errores transitorios. Para obtener información sobre cómo controlar los errores transitorios al usar los servicios de Microsoft Azure, consulte [Directrices de reintento específicas del servicio Azure](./retry-service-specific.md).

<!-- markdownlint-disable MD026 -->

## <a name="why-do-transient-faults-occur-in-the-cloud"></a>¿Por qué se producen errores transitorios en la nube?

<!-- markdownlint-enable MD026 -->

En cualquier entorno, plataforma o sistema operativo y en cualquier tipo de aplicación pueden producirse errores transitorios. En soluciones que se ejecutan en infraestructuras locales, el rendimiento y la disponibilidad de la aplicación y sus componentes se mantiene normalmente a través de una costosa e infrautilizada redundancia de hardware y los componentes y los recursos están ubicados cerca entre sí. Aunque esto provoca que un error sea menos probable, todavía puede provocar errores transitorios (e incluso una interrupción del servicio a través de eventos imprevistos como problemas con fuentes de alimentación externas o de red u otros escenarios de desastre.

El hospedaje en nube, incluidos los sistemas de nube privada, pueden ofrecer una mayor disponibilidad general mediante el uso de recursos compartidos, redundancia, conmutación por error automática y asignación dinámica de recursos a través de un gran número de nodos de proceso de mercancías. Sin embargo, la naturaleza de estos entornos puede significar que están más probable que se produzcan errores transitorios. Hay varias razones para esto:

- Se comparten muchos recursos en un entorno de nube y el acceso a estos recursos está sujeto a limitación para proteger el recurso. Algunos servicios rechazarán las conexiones cuando la carga alcance un nivel específico o un índice de rendimiento máximo para permitir el procesamiento de las solicitudes existentes y para mantener el rendimiento del servicio para todos los usuarios. La limitación ayuda a mantener la calidad del servicio para vecinos y otros inquilinos mediante el recurso compartido.

- Los entornos de nube se crean con una gran cantidad de unidades de hardware de mercancías. Ofrecen rendimiento al distribuir dinámicamente la carga entre varias unidades informáticas y componentes de infraestructura y proporcionan confiabilidad reciclando o reemplazando unidades con errores automáticamente. Esta naturaleza dinámica significa que en ocasiones pueden producirse errores transitorios y errores de conexión temporales.

- A menudo hay más componentes de hardware, incluida la infraestructura de red, como enrutadores y equilibradores de carga, entre la aplicación y los recursos y servicios que usa. Esta infraestructura adicional ocasionalmente puede introducir latencia de conexión adicional y errores transitorios de conexión.

- Las condiciones de red entre el cliente y el servidor pueden ser variables, especialmente cuando la comunicación cruza Internet. Incluso en ubicaciones locales, cargas de tráfico muy intensas pueden ralentizar la comunicación y causar errores de conexión intermitentes.

## <a name="challenges"></a>Desafíos

Los errores transitorios pueden tener un gran impacto en la disponibilidad aparente de una aplicación, incluso si se ha probado exhaustivamente en todas las circunstancias previsibles. Para asegurarse de que las aplicaciones hospedadas en nube funcionan de manera fiable, debe poder responder a los siguientes desafíos:

- La aplicación debe ser capaz de detectar errores cuando se producen y determinar si es probable que estos errores sean transitorios, más duraderos o de terminal. Recursos diferentes tienen grandes posibilidades de devolver respuestas diferentes cuando se produce un error, y estas respuestas también pueden variar dependiendo del contexto de la operación; por ejemplo, la respuesta para un error al leer desde el almacenamiento puede variar de la respuesta para un error al escribir en el almacenamiento. Muchos recursos y servicios tienen contratos de error transitorio bien documentados. Sin embargo, cuando dicha información no está disponible, puede ser difícil de detectar la naturaleza del error y si es probable que sea transitorio.

- La aplicación debe poder volver a intentar la operación si se determina que es posible que el error sea transitorio y realizar un seguimiento de la cantidad de veces que se vuelve a intentar la operación.

- La aplicación debe usar una estrategia adecuada para los reintentos. Esta estrategia especifica el número de veces que debe reintentar, el intervalo entre cada intento y las acciones a realizar después de un intento fallido. El número de intentos y el intervalo adecuado entre cada uno suelen ser difíciles de determinar y varían según el tipo de recurso, así como las condiciones de funcionamiento actuales del recurso y la propia aplicación.

## <a name="general-guidelines"></a>Directrices generales

Las directrices siguientes le ayudarán a diseñar un mecanismo de control de errores transitorio adecuado para sus aplicaciones:

- **Determine si hay un mecanismo de reintento integrado:**

  - Muchos servicios proporcionan una biblioteca SDK o de cliente que contiene un mecanismo de control de errores transitorios. La directiva de reintentos que usa normalmente se adapta a la naturaleza y los requisitos del servicio de destino. Como alternativa, las interfaces REST de servicios pueden devolver información útil para determinar si un reintento es adecuado y cuánto tiempo se debe esperar antes del siguiente reintento.

  - Use el mecanismo de reintento integrado donde está disponible a menos que tenga requisitos específicos y conocidos que indiquen que es más apropiado un comportamiento de reintento diferente.

- **Determine si la operación es adecuada para el reintento**:

  - Solo debe volver a intentar operaciones en las que los errores sean transitorios (normalmente indicado por la naturaleza del error) y, si hay al menos una probabilidad de que la operación se realice correctamente cuando se vuelve a intentar. No se deben reintentar las operaciones que indican una operación no válida, como una actualización de una base de datos en un elemento que no existe, o las solicitudes a un servicio o recurso en el que se ha producido un error grave

  - En general, debería implementar reintentos solamente donde se puedan determinar las consecuencias totales de este, las condiciones se comprendan bien y se puedan validar. De lo contrario, déjelo al código de llamada para implementar los reintentos. Recuerde que los errores devueltos de los recursos y servicios situados fuera de su control pueden evolucionar con el tiempo, y es posible que necesite volver a visitar la lógica de detección de errores transitorios.

  - Al crear servicios o componentes, considere la posibilidad de implementar los códigos y mensajes de error que le ayudarán a los clientes a determinar si deben reintentar las operaciones con errores. En concreto, indique si el cliente debería reintentar la operación (quizás devolviendo un valor **isTransient** ) y sugiera un intervalo adecuado antes del siguiente reintento. Si crea un servicio web, considere la posibilidad de devolver los errores personalizados definidos dentro de los contratos de servicio. Aunque es posible que los clientes genéricos no puedan leer estos, serán útiles a la hora de crear clientes personalizados.

- **Determinar un número adecuado de reintentos y el intervalo:**

  - Es fundamental optimizar el número de reintentos y el intervalo para el tipo de caso de uso. Si no efectúa reintentos un número suficiente de veces, la aplicación no podrá completar la operación y es probable que experimente un error. Si efectúa reintentos demasiadas veces o en un intervalo demasiado corto entre intentos, la aplicación puede retener potencialmente los recursos como subprocesos, conexiones y memoria durante largos períodos, lo que afectará negativamente al estado de la aplicación.

  - Los valores adecuados para el intervalo de tiempo y el número de reintentos dependen del tipo de operación que se esté intentando. Por ejemplo, si la operación forma parte de una interacción del usuario, el intervalo deberá ser corto y solo se deben efectuar unos pocos reintentos para evitar que los usuarios tengan que esperar una respuesta (que contiene conexiones abiertas y puede reducir la disponibilidad de otros usuarios). Si la operación forma parte de un flujo de trabajo de ejecución prolongada o crítico, donde cancelar y reiniciar el proceso resulta lento o costoso, conviene esperar más tiempo entre intentos y efectuar más reintentos.

  - Determinar los intervalos entre reintentos adecuados es la parte más difícil de diseñar una estrategia satisfactoria. Las estrategias típicas usan los siguientes tipos de intervalo de reintento:

    - **Interrupción exponencial**. La aplicación espera poco tiempo antes del primer reintento y, a continuación, aumenta exponencialmente los tiempos entre cada intento posterior. Por ejemplo, puede reintentar la operación después de 3 segundos, 12 segundos, 30 segundos y así sucesivamente.

    - **Intervalos incrementales**. La aplicación espera poco tiempo antes del primer reintento y, a continuación, aumenta incrementalmente los tiempos entre cada intento posterior. Por ejemplo, puede reintentar la operación después de 3 segundos, 7 segundos, 13 segundos y así sucesivamente.

    - **Intervalos regulares**. La aplicación espera el mismo período de tiempo entre cada intento. Por ejemplo, puede reintentar la operación cada 3 segundos.

    - **Reintento inmediato**. A veces un error transitorio es muy corto, quizás debido a un evento como una colisión de paquetes de red o un pico en un componente de hardware. En este caso, es conveniente volver a intentar la operación inmediatamente porque puede tener éxito si se resolvió el problema en el tiempo que tarda la aplicación en ensamblar y enviar la solicitud siguiente. Sin embargo, nunca debería haber más de un reintento inmediato y debe cambiar a estrategias alternativas, como acciones de interrupción o de conmutación por error exponenciales si se produce un error en el reintento inmediato.

    - **Selección aleatoria**. Cualquiera de las estrategias de reintento enumeradas anteriormente pueden incluir una selección aleatoria para evitar que varias instancias del cliente envíen reintentos posteriores al mismo tiempo. Por ejemplo, una instancia puede reintentar la operación después de 3 segundos, 11 segundos, 28 segundos y así sucesivamente, mientras que otra instancia puede reintentar la operación después de 4 segundos, 12 segundos, 26 segundos y así sucesivamente. La selección aleatoria es una técnica útil que se puede combinarse con otras estrategias.

  - Como norma general, use una estrategia de interrupción exponencial para operaciones en segundo plano y estrategias de reintento de intervalo inmediato o normal para operaciones interactivas. En ambos casos, debe elegir el retraso y el número de reintentos para que la latencia máxima de todos los reintentos se encuentre dentro del requisito de latencia integral necesario.

  - Tenga en cuenta la combinación de todos los factores que contribuyen al tiempo de espera máximo global de una operación reintentada. Estos factores incluyen el tiempo necesario para que un error en la conexión genere una respuesta (normalmente establecido por un valor de tiempo de espera en el cliente), así como el intervalo entre reintentos y el número máximo de reintentos. El total de todos estos tiempos puede provocar tiempos de duración generales de las operaciones muy elevados, especialmente cuando se usa una estrategia de retraso exponencial en la que intervalo entre reintentos crece rápidamente después de cada error. Si un proceso debe cumplir un acuerdo de nivel de servicio específico (SLA), el tiempo total de la operación, incluidos todos los tiempos de espera y retrasos, debe encontrarse dentro del definido en el SLA.

  - Las estrategias de reintento demasiado agresivas, que tienen intervalos demasiado cortos o demasiados reintentos, pueden tener un efecto adverso en el recurso de destino o el servicio. Esto puede impedir que el recurso o servicio se recupere de su estado de sobrecarga y continuará bloqueando o rechazando las solicitudes. Esto provoca un círculo vicioso en el que cada vez se envían más solicitudes al recurso o servicio y, por consiguiente, su capacidad para recuperar el resultado se reduce todavía más.

  - Tenga en cuenta el tiempo de espera de las operaciones al elegir los intervalos de reintento para evitar el inicio de intentos posteriores inmediatamente (por ejemplo, si el tiempo de espera es similar del intervalo de reintento). También considere si necesita mantener el período total posible (el tiempo de espera más los intervalos de reintento) por debajo de un tiempo total específico. Las operaciones que tienen tiempos de espera muy cortos o muy largos pueden influir en cuánto se espera y la frecuencia de los reintentos de la operación.

  - Utilice el tipo de la excepción y los datos que contiene o los códigos de error y los mensajes devueltos desde el servicio para optimizar el intervalo y el número de reintentos. Por ejemplo, algunas excepciones o códigos de error (como el código HTTP 503 Servicio no disponible con un encabezado Retry-After en la respuesta) pueden indicar cuánto puede durar el error o que el servicio ha producido un error y no responderá a todos los intentos posteriores.

- **Evitar antipatrones**:

  - En la mayoría de los casos, debe evitar las implementaciones que incluyen capas duplicadas de código de reintento. Evite los diseños que incluyan mecanismos de reintento en cascada, o que implementan reintentos en cada etapa de una operación que implica a una jerarquía de solicitudes, a menos que tenga requisitos específicos que requieran esto. En estas circunstancias excepcionales, use directivas que impidan números excesivos de reintentos y de períodos de retraso y asegúrese de que comprende las consecuencias. Por ejemplo, si un componente realiza una solicitud a otro, que a continuación accede al servicio de destino e implementa el reintento con un recuento de tres en ambas llamadas habrá nueve reintentos en total en el servicio. Muchos servicios y recursos implementan un mecanismo de reintento integrado y debe investigar cómo deshabilitar o modificar este comportamiento si necesita implementar reintentos en un nivel superior.

  - Nunca implemente un mecanismo de reintento infinito. Esto es probable que evite la recuperación del recurso o servicio de situaciones de sobrecarga y que provoque una limitación y conexiones rechazas para continuar durante un período más largo. Use un número finito de reintentos o implemente un patrón como [Disyuntor](../patterns/circuit-breaker.md) para permitir que el servicio se recupere.

  - Nunca realice un reintento inmediato más de una vez.

  - Evite el uso de un intervalo de reintento regular, especialmente si hay un gran número de reintentos, al obtener acceso a servicios y recursos de Azure. El enfoque óptimo es que este escenario es una estrategia de interrupción exponencial con una capacidad de disyunción.

  - Evite que varias instancias del mismo cliente o varias instancias de clientes diferentes, envíen reintentos al mismo tiempo. Si es probable que esto se produzca, introduzca la selección aleatoria en los intervalos de reintento.

- **Pruebe su estrategia de reintento e implementación:**

  - Asegúrese de probar completamente la implementación de la estrategia de reintento en el conjunto de circunstancias más amplio posible, especialmente cuando tanto la aplicación y los recursos de destino o servicios que use estén bajo una carga extrema. Para comprobar el comportamiento durante las pruebas, puede:

    - Insertar errores transitorios y no transitorios en el servicio. Por ejemplo, envíe solicitudes no válidas o agregue código que detecte solicitudes de prueba y responda con distintos tipos de errores. Para ver un ejemplo con TestApi, consulte [Pruebas de inyección de errores con TestApi](https://msdn.microsoft.com/magazine/ff898404.aspx) e [Introducción a TestApi – Parte 5: API de inyección de errores de código administrado](https://blogs.msdn.microsoft.com/ivo_manolov/2009/11/25/introduction-to-testapi-part-5-managed-code-fault-injection-apis/).

    - Cree un simulacro del recurso o servicio que devuelve un rango de errores que puede devolver el servicio real. Asegúrese de abarcar todos los tipos de error que su estrategia de reintento está diseñada para detectar.

    - Fuerce el que se produzcan errores transitorios deshabilitando o sobrecargando el servicio temporalmente si es un servicio personalizado que ha creado e implementado (no debe, por supuesto, intentar sobrecargar los recursos o servicios compartidos dentro de Azure).

    - Para las API basadas en HTTP, considere la posibilidad de usar la biblioteca FiddlerCore en las pruebas automatizadas para cambiar el resultado de las solicitudes HTTP, mediante la adición de tiempos de ida y vuelta adicionales o cambiando la respuesta (por ejemplo, el código de estado HTTP, los encabezados, el cuerpo u otros factores). Esto permite realizar una prueba determinista de un subconjunto de condiciones de error, tanto si son errores transitorios como si son otros tipos de error. Para obtener más información, consulte [FiddlerCore](https://www.telerik.com/fiddler/fiddlercore). Para obtener ejemplos de cómo usar la biblioteca, especialmente la clase **HttpMangler** , examine el [código fuente del SDK de Azure Storage](https://github.com/Azure/azure-storage-net/tree/master/Test).

    - Realice pruebas simultáneas y de factor de carga elevado para asegurarse de que el mecanismo de reintento y la estrategia funcionan correctamente en estas condiciones y no tienen un efecto adverso en el funcionamiento del cliente ni provocan contaminación cruzada entre las solicitudes.

- **Administrar las configuraciones de directivas de reintento:**

  - Una *directiva de reintentos* es una combinación de todos los elementos de la estrategia de reintentos. Define el mecanismo de detección que determina si es posible que un error sea transitorio, el tipo de intervalo que se debe usar (por ejemplo, normal, interrupción exponencial y selección aleatoria), los valores de intervalo reales y el número de reintentos.

  - Los reintentos deben implementarse en varios lugares dentro de incluso la aplicación más sencilla y en todas las capas de aplicaciones más complejas. En lugar de codificar los elementos de cada directiva en varias ubicaciones, considere el uso de un punto central para almacenar todas las directivas. Por ejemplo, almacene los valores como el intervalo y el recuento de reintentos en los archivos de configuración de la aplicación, léalos en tiempo de ejecución y cree las directivas de reintento mediante programación. Esto facilita la administración de la configuración y la modificación y ajuste preciso de los valores para responder a los cambiantes requisitos y escenarios. Sin embargo, diseñe el sistema para almacenar los valores en lugar de volver a leer un archivo de configuración cada vez y asegúrese de que se usan valores predeterminados adecuados si no se puede obtener los valores de la configuración.

  - En una aplicación de Azure Cloud Services, considere la posibilidad de almacenar los valores que se usan para crear las directivas de reintento en tiempo de ejecución en el archivo de configuración de servicio para que se puedan cambiar sin necesidad de reiniciar la aplicación.

  - Aproveche las estrategias de reintento integradas o predeterminadas disponibles en las API de cliente que use, pero sólo cuando resulten adecuadas para su escenario. Estas estrategias son normalmente para fines generales. En algunos escenarios pueden ser lo único necesario, pero en otros escenarios puede que no ofrezcan la gama completa de opciones necesarias para satisfacer sus requisitos específicos. Debe entender cómo afectará la configuración a la aplicación a través de pruebas para determinar los valores más adecuados.

- **Registre y realice un seguimiento de errores transitorios y no transitorios:**

  - Como parte de su estrategia de reintentos, incluya el control de excepciones y otras herramientas que se registren cuando se realicen reintentos. Aunque se espere un error transitorio ocasional y un reintento y no indiquen un problema, números de reintentos normales y en aumento a menudo son indicadores de un problema que puede producir un error o que actualmente está afectando al rendimiento de la aplicación y a su disponibilidad.

  - Registre errores transitorios como entradas de advertencia en lugar de entradas de error de manera que los sistemas de supervisión no los detecten como errores de aplicación que podrían desencadenar alertas falsas.

  - Considere la posibilidad de almacenar un valor en las entradas de registro que indique si los reintentos fueron originados por una limitación en el servicio o por otros tipos de errores como los errores de conexión, para que pueda diferenciarlos durante el análisis de los datos. A menudo, un aumento del número de errores de limitación es un indicador de un defecto de diseño de la aplicación o la necesidad de cambiar a un servicio premium que ofrece hardware específico.

  - Considere la posibilidad de medir y registrar el tiempo total necesario para las operaciones que incluyen un mecanismo de reintento. Esto es un buen indicador del efecto global de los errores transitorios en tiempos de respuesta de usuario, latencia de procesos y la eficacia de los casos de uso de la aplicación. Registre también el número de reintentos que se produjo para comprender los factores que han contribuido a obtener el tiempo de respuesta.

  - Considere la posibilidad de implementar un sistema de telemetría y supervisión que pueda generar alertas cuando el número y la tasa de errores, el promedio de reintentos o los tiempos generales adoptados para que las operaciones se realicen correctamente, está aumentando.

- **Administrar operaciones que producen un error continuamente:**
  
  - Habrá casos en los que la operación sigue produciendo errores en cada intento y es esencial tener en cuenta cómo tratar esta situación:

    - Aunque una estrategia de reintento definirá el número máximo de veces que se debe reintentar una operación, ello no impide que la aplicación repeta la operación de nuevo, con el mismo número de reintentos. Por ejemplo, si se produce un error grave en un servicio de procesamiento de pedidos que lo pone fuera de funcionamiento de manera permanente, la estrategia de reintento puede detectar un tiempo de espera de conexión y considerarlo un error transitorio. El código reintentará la operación un número especificado de veces y, a continuación, renunciará. Sin embargo, cuando otro cliente realiza un pedido, la operación se intentará de nuevo (aunque sea seguro que vaya a fallar siempre).

    - Para evitar reintentos continuos de operaciones que producen un error continuamente, considere la posibilidad de implementar el [patrón de disyuntor](../patterns/circuit-breaker.md). En este patrón, si el número de errores dentro de un período de tiempo especificado supera el umbral, las solicitudes se devuelven al llamador inmediatamente como errores, sin intentar tener acceso al recurso o servicio con error.

    - La aplicación puede comprobar periódicamente el servicio, de forma intermitente y con intervalos muy largos entre solicitudes para detectar cuando está disponible. Un intervalo adecuado dependerá de la situación, por ejemplo, la importancia de la operación y la naturaleza del servicio y puede estar comprendido entre unos pocos minutos y varias horas. En el punto en el que la prueba se realiza correctamente, la aplicación puede reanudar las operaciones normales y pasar solicitudes al servicio recién recuperado.

    - Mientras tanto, es posible revertir a otra instancia del servicio (quizás en un centro de datos o una aplicación diferente), use un servicio similar que ofrezca una funcionalidad (quizás más sencilla) compatible o realice algún operación alternativa con la esperanza de que el servicio pase a estar disponible pronto. Por ejemplo, puede ser adecuado almacenar solicitudes para el servicio en una cola o almacén de datos y volver a reproducirlas posteriormente. En caso contrario, es posible que pueda redirigir al usuario a una instancia alternativa de la aplicación, reducir el rendimiento de la aplicación pero seguir ofreciendo funcionalidad aceptable o simplemente presentar un mensaje al usuario que indique que la aplicación no está disponible en este momento.

- **Otras consideraciones**
  
  - Al decidir los valores para el número de reintentos y los intervalos de reintento para una directiva, tenga en cuenta si la operación del servicio o recurso forma parte de una operación de ejecución prolongada o de varios pasos. Puede ser difícil o costoso compensar los demás pasos operativos que ya se han realizado correctamente cuando uno falla. En este caso, un intervalo muy largo y un gran número de reintentos pueden ser aceptables siempre y cuando no bloquee otras operaciones manteniendo o bloqueando recursos insuficientes.

  - Tenga en cuenta si al volver a intentar la misma operación pueden producirse incoherencias en los datos. Si se repiten algunas partes de un proceso de varios pasos y las operaciones no son idempotentes, puede darse como resultado una incoherencia. Por ejemplo, una operación que incrementa un valor, si se repite, generará un resultado no válido. Repetir una operación que envía un mensaje a una cola puede provocar una incoherencia en el consumidor de mensajes si no puede detectar mensajes duplicados. Para evitarlo, asegúrese de diseñar cada paso como una operación idempotente. Para más información acerca de la idempotencia, consulte los [patrones de idempotencia][idempotency-patterns].

  - Tenga en cuenta el ámbito de las operaciones que se volverán a intentar. Por ejemplo, puede ser más fácil de implementar el código de reintento en un nivel que abarca varias operaciones y reintentarlos de nuevo todos si se produce un error en uno. Sin embargo, esto podría provocar problemas de idempotencia u operaciones de reversión innecesarias.

  - Si elige un ámbito de reintento que abarca varias operaciones, tenga en cuenta la latencia total de todos ellos al determinar los intervalos de reintento, al supervisar el tiempo que tardan y antes de generar alertas para los errores.

  - Tenga en cuenta cómo puede afectar su estrategia de reintento a vecinos y otros inquilinos de una aplicación compartida o al usar servicios y recursos compartidos. Las directivas de reintento agresivas puede ocasionar un número creciente de errores transitorios a estos otros usuarios y para las aplicaciones que comparten los recursos y servicios. Del mismo modo, es posible que la aplicación se vea afectada por las directivas de reintento implementadas por otros usuarios de los servicios y recursos. Para las aplicaciones críticas, puede decidir usar servicios Premium que no se comparten. Esto le brinda un mayor control sobre la carga y la consiguiente limitación de estos recursos y servicios, que pueden ayudar a justificar el coste adicional.

## <a name="more-information"></a>Más información

- [Directrices de reintento específicas del servicio de Azure](./retry-service-specific.md)
- [Circuit Breaker pattern](../patterns/circuit-breaker.md) (Patrón Circuit Breaker)
- [Patrón de transacción de compensación](../patterns/compensating-transaction.md)
- [Patrones de idempotencia][idempotency-patterns]

<!-- links -->

[idempotency-patterns]: https://blog.jonathanoliver.com/idempotency-patterns/
