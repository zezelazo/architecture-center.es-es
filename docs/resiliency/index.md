---
title: Diseño de aplicaciones resistentes de Azure
description: Cómo crear aplicaciones resistentes de Azure, para alta disponibilidad y recuperación ante desastres.
author: MikeWasson
ms.date: 05/26/2017
ms.custom: resiliency
ms.openlocfilehash: f5dfca30b3587e198c1c0f99ad4903ed7b0e33d4
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/28/2018
ms.locfileid: "47429302"
---
# <a name="designing-resilient-applications-for-azure"></a>Diseño de aplicaciones resistentes de Azure

En un sistema distribuido, se pueden producir errores. Pueden producirse errores de hardware. La red puede tener errores transitorios. En raras ocasiones, todo un servicio o toda una región pueden experimentar interrupciones, pero incluso estas deben planificarse. 

Crear una aplicación confiable en la nube es diferente a crear una aplicación confiable en un entorno empresarial. Si bien históricamente puede haber adquirido hardware de gama superior para escalar verticalmente, en un entorno de nube debe escalar horizontalmente en lugar de verticalmente. Los costos de los entornos de nube se mantienen bajos gracias al uso de hardware básico. En lugar de centrarse en la prevención de errores y en la optimización del "tiempo medio entre errores", en este nuevo entorno, el enfoque cambia a "tiempo medio para restaurar". El objetivo es minimizar el efecto de los errores.

En este artículo se introduce a la creación de aplicaciones resistentes en Microsoft Azure. Se inicia con una definición del término *resistencia* y conceptos relacionados. Después se describe un proceso para lograr resistencia, mediante un enfoque estructurado durante la vigencia de una aplicación, desde el diseño y la puesta en marcha hasta la implementación y operaciones.

## <a name="what-is-resiliency"></a>¿Qué es la resistencia?
**Resistencia** es la capacidad de un sistema de recuperarse de los errores y seguir funcionando. No se trata de *evitar* los errores, sino de *responder* a ellos de manera que se evite el tiempo de inactividad o la pérdida de datos. El objetivo de la resistencia es devolver la aplicación a un estado plenamente operativo después de un error.

Dos aspectos importantes de la resistencia son la alta disponibilidad y la recuperación ante desastres.

* **Alta disponibilidad** (HA) es la capacidad de la aplicación de seguir ejecutándose en un estado correcto, sin tiempo de inactividad relevante. Por "estado correcto," queremos decir que la aplicación responde y los usuarios pueden conectarse e interactuar con ella.  
* La **recuperación ante desastres** es la capacidad de recuperación de incidentes importantes pero poco frecuentes: errores a gran escala no transitorios, tales como una interrupción del servicio que afecta a toda la región. La recuperación ante desastres incluye la copia de seguridad y el archivado de datos, y puede incluir la intervención manual, como la restauración manual de una base de datos a partir de la copia de seguridad.

Una manera de pensar en alta disponibilidad frente a la recuperación ante desastres es que esta se inicia cuando el impacto de un error supera la capacidad del diseño de alta disponibilidad de controlarlo.  

Si va a diseñar la resistencia, debe conocer los requisitos de disponibilidad. ¿Cuánto tiempo de inactividad es aceptable? Depende en parte del costo. ¿Cuánto le costará el tiempo de inactividad potencial a su negocio? ¿Cuánto debe invertir para que la aplicación tenga alta disponibilidad? También debe definir lo que significa que la aplicación esté disponible. Por ejemplo, ¿se considera que la aplicación está "fuera de servicio" si un cliente puede enviar un pedido pero el sistema no puede procesarlo en el período de tiempo normal? También considere la probabilidad de que ocurra un determinado tipo de interrupción y si una estrategia de mitigación es rentable.

Otro término común es la **continuidad del negocio**, que consiste en la capacidad de realizar funciones empresariales esenciales durante y después de condiciones adversas, como un desastre natural o un servicio que ha dejado de funcionar. La continuidad de negocio cubre toda la operación de la empresa, incluidas las instalaciones físicas, las personas, las comunicaciones, el transporte e IT. Este artículo se centra en las aplicaciones en la nube, pero el planeamiento de la resistencia debe realizarse en el contexto de los requisitos generales de la continuidad del negocio. 

Las **copias de seguridad de datos** constituyen una parte fundamental de la recuperación ante desastres. Si se produce un error en los componentes sin estado de una aplicación, siempre puede implementarlos de nuevo. Sin embargo, si se pierden los datos, el sistema no podrá volver a un estado estable. Por tanto, es necesario hacer copias de seguridad, y lo ideal es hacerlo en una región diferente por si se produjera un desastre que afectara a toda una región. 

Las copias de seguridad no son lo mismo que la **replicación de datos**. La replicación de datos implica copiar datos casi en tiempo real, por lo que el sistema puede conmutar por error a una réplica rápidamente. Muchos sistemas de bases de datos admiten la replicación; por ejemplo, SQL Server es compatible con Grupos de disponibilidad Always On de SQL Server. La replicación de datos puede reducir el tiempo necesario para recuperarse de una interrupción, ya que garantiza que siempre hay una réplica de datos disponible. Sin embargo, la replicación de datos no sirve de protección frente a los errores humanos. Si los datos resultan dañados por un error humano, se copiarán en las réplicas. Por tanto, necesitará incorporar también las copias de seguridad a largo plazo en la estrategia de recuperación ante desastres. 

## <a name="process-to-achieve-resiliency"></a>Proceso para lograr resistencia
La resistencia no es un complemento. Deberá diseñarse en el sistema y aplicarse en la práctica operativa. Este es un modelo general para seguir:

1. **Defina** sus requisitos de disponibilidad, en función de las necesidades del negocio.
2. **Diseñe** la aplicación para proporcionar resistencia. Comience con una arquitectura que siga las prácticas comprobadas e identifique los puntos de error posibles en esa arquitectura.
3. **Implemente** estrategias para detectar y recuperarse de errores. 
4. **Pruebe** la implementación simulando errores y desencadene conmutaciones por error forzadas. 
5. **Implemente** la aplicación en producción mediante un proceso confiable y repetible. 
6. **Supervise** la aplicación para detectar errores. Al supervisar el sistema, puede medir el estado de la aplicación y responder a las incidencias en caso necesario. 
7. **Responda** si existen errores que requieran intervenciones manuales.

En el resto de este artículo, se describen estos pasos con más detalle.

## <a name="define-your-availability-requirements"></a>Definición de sus requisitos de disponibilidad
El planeamiento de la resistencia comienza con los requisitos empresariales. Estos son algunos enfoques para pensar en resistencia en esos términos.

### <a name="decompose-by-workload"></a>Descomposición por carga de trabajo
Muchas soluciones en la nube constan de varias cargas de trabajo de aplicación. El término "carga de trabajo" en este contexto significa una funcionalidad discreta o tarea de computación, que se puede separar lógicamente de otras tareas, en términos de requisitos de lógica de negocio y de almacenamiento de datos. Por ejemplo, una aplicación de comercio electrónico podría incluir las cargas de trabajo siguientes:

* Examinar y buscar un catálogo de productos.
* Crear y realizar el seguimiento de pedidos.
* Ver recomendaciones.

Estas cargas de trabajo pueden tener diferentes requisitos de disponibilidad, escalabilidad, coherencia de los datos o recuperación ante desastres, entre otros. De nuevo, son decisiones empresariales.

Considere también la posibilidad de patrones de uso. ¿Hay ciertos períodos críticos en los que el sistema debe estar disponible? Por ejemplo, un servicio de declaración de impuestos no puede estar inactivo justo antes de la fecha límite de presentación, un servicio de transmisión de vídeo debe estar activo durante un gran evento deportivo y así sucesivamente. Durante los períodos críticos, es posible que tenga implementaciones redundantes en varias regiones, por lo que la aplicación puede conmutar por error si se produce un error en una región. Sin embargo, una implementación en varias regiones es más costosa, por lo que en los momentos menos críticos, se puede ejecutar la aplicación en una única región.

### <a name="rto-and-rpo"></a>RTO y RPO
Dos métricas importantes a tener en cuenta son el objetivo de tiempo de recuperación (RTO) y el objetivo de punto de recuperación (RPO).

* **Objetivo de tiempo de recuperación** (RTO) es el tiempo máximo aceptable que una aplicación puede no estar disponible después de un incidente. Si el RTO es de 90 minutos, debe ser capaz de restaurar la aplicación a un estado en ejecución en un plazo de 90 minutos desde el inicio de un desastre. Si tiene un RTO muy bajo, puede mantener una segunda implementación ejecutándose continuamente en modo de espera, para protegerse contra una interrupción regional.

* **Objetivo de punto de recuperación** (RPO) es la duración máxima de la pérdida de datos que es aceptable durante un desastre. Por ejemplo, si se almacenan datos en una única base de datos, sin replicación en otras bases de datos y se realizan copias de seguridad por hora, se podría perder hasta una hora de datos. 

RTO y el RPO son requisitos empresariales. Llevar a cabo una evaluación de riesgos puede ayudarle a definir el RTO y el RPO de la aplicación. Otra medida común es el **tiempo medio para recuperación** (MTTR), que es el tiempo medio que se tarda en restaurar la aplicación después de un error. MTTR es un hecho empírico acerca de un sistema. Si MTTR excede el RTO, un error en el sistema causará una interrupción inaceptable del negocio, porque no será posible restaurar el sistema dentro del RTO definido. 

### <a name="slas"></a>SLA
En Azure, el [Acuerdo de Nivel de Servicio][sla] explica los compromisos de Microsoft en cuanto a tiempo de actividad y conectividad. Si el Acuerdo de Nivel de Servicio para un servicio determinado es del 99,9 %, significa que debe esperar a que el servicio esté disponible un 99,9 % del tiempo.

> [!NOTE]
> El Acuerdo de Nivel de Servicio de Azure también incluye disposiciones para obtener un crédito de servicio si no se cumpla el Acuerdo, junto con definiciones específicas de "disponibilidad" para cada servicio. Ese aspecto del Acuerdo de Nivel de Servicio actúa como una directiva de cumplimiento. 
> 
> 

Debe definir sus propios Acuerdos de Nivel de Servicio de destino para cada carga de trabajo de la solución. Un Acuerdo de Nivel de Servicio permite evaluar si la arquitectura cumple con los requisitos empresariales. Por ejemplo, si una carga de trabajo requiere un tiempo de actividad del 99,99 %, pero depende de un servicio con un Acuerdo de Nivel de Servicio del 99,9 %, ese servicio no puede ser un único punto de error en el sistema. Una solución es tener una ruta de reserva en caso de que se produzca un error en el servicio, o bien tomar otras medidas para recuperarse de un error en ese servicio. 

En la tabla siguiente se muestra el tiempo de inactividad acumulativo potencial para varios niveles de Acuerdo de Nivel de Servicio. 

| Contrato de nivel de servicio | Tiempo de inactividad por semana | Tiempo de inactividad por mes | Tiempo de inactividad por año |
| --- | --- | --- | --- |
| 99% |1,68 horas |7,2 horas |3,65 días |
| 99,9 % |10,1 minutos |43,2 minutos |8,76 horas |
| 99,95 % |5 minutos |21,6 minutos |4,38 horas |
| 99,99% |1,01 minutos |4,32 minutos |52,56 minutos |
| 99,999 % |6 segundos |25,9 segundos |5,26 minutos |

Por supuesto, una mayor disponibilidad es mejor, todo lo demás es igual. Pero a medida que aspira a tener más números 9, el costo y la complejidad para lograr ese nivel de disponibilidad aumenta. Un tiempo de actividad del 99,99 % se traduce en unos 5 minutos de tiempo de inactividad total por mes. ¿Vale la pena la complejidad y el costo adicionales para llegar a cinco 9? La respuesta depende de los requisitos del negocio. 

Estas son algunas otras consideraciones al definir un Acuerdo de Nivel de Servicio:

* Para lograr cuatro 9 (99,99 %), probablemente no pueda confiar en una intervención manual para recuperarse de errores. La aplicación debe autodiagnosticarse y recuperarse automáticamente. 
* Más allá de cuatro 9, es difícil detectar interrupciones del sistema lo suficientemente rápido como para cumplir con el Acuerdo de Nivel de Servicio.
* Piense en la ventana de tiempo con la que se mide que el Acuerdo de Nivel de Servicio. Cuanto menor sea la ventana, más estrictas serán las tolerancias. Probablemente no tenga sentido definir el Acuerdo de Nivel de Servicio en términos de tiempo de actividad a cada hora o a diario. 

### <a name="composite-slas"></a>Acuerdos de Nivel de Servicio compuestos
Considere una aplicación web de App Service que escribe en Azure SQL Database. En el momento de redactar este artículo, estos servicios de Azure tienen los siguientes Acuerdos de Nivel de Servicio:

* App Service Web Apps = 99,95 %
* SQL Database = 99,99 %

![Acuerdo de Nivel de Servicio compuesto](./images/sla1.png)

¿Cuál es el tiempo de inactividad máximo que se esperaría para esta aplicación? Si se produce un error en cualquiera de los servicios, se produce un error en toda la aplicación. En general, la probabilidad de que se produzca un error en cada servicio es independiente, por lo que el Acuerdo de Nivel de Servicio compuesto para esta aplicación es del 99,95 % &times; 99,99 % = 99,94 %. Esto es menor que los Acuerdos de Nivel de Servicio individuales, lo cual no es sorprendente, porque una aplicación que depende de varios servicios tiene más puntos de error posibles. 

Por otra parte, puede mejorar el Acuerdo de Nivel de Servicio compuesto mediante la creación de rutas de reserva independientes. Por ejemplo, si la instancia de SQL Database no está disponible, coloque las transacciones en una cola para procesarla más adelante.

![Acuerdo de Nivel de Servicio compuesto](./images/sla2.png)

Con este diseño, la aplicación sigue estando disponible, aunque no pueda conectarse a la base de datos. Sin embargo, se produce un error si la base de datos y la cola fallan al mismo tiempo. El porcentaje de tiempo previsto para un error simultáneo es 0,0001 &times; 0,001, por lo que el Acuerdo de Nivel de Servicio compuesto para esta ruta combinada es:  

* Base de datos O cola = 1,0 &minus; (0,0001 &times; 0,001) = 99,99999 %

El Acuerdo de Nivel de Servicio compuesto total es:

* Aplicación web Y (base de datos O cola) = 99,95 % &times; 99,99999 % = ~99,95 %

Pero este enfoque tiene sus ventajas e inconvenientes. La lógica de aplicación es más compleja, está pagando por la cola y puede haber problemas de coherencia de datos a tener en cuenta.

**Acuerdo de Nivel de Servicio para implementaciones en varias regiones**. Otra técnica de alta disponibilidad consiste en implementar la aplicación en más de una región y usar Azure Traffic Manager para conmutar por error si la aplicación produce un error en una región. Para una implementación de dos regiones, el Acuerdo de Nivel de Servicio compuesto se calcula del siguiente modo. 

Supongamos que *N* es el Acuerdo de Nivel de Servicio compuesto para la aplicación implementada en una región. La probabilidad de que la aplicación produzca un error en ambas regiones al mismo tiempo es (1 &minus; N) &times; (1 &minus; N). Por lo tanto,

* Acuerdo de Nivel de Servicio combinado para ambas regiones = 1 &minus; (1 &minus; N)(1 &minus; N) = N + (1 &minus; N)N

Por último, debe tener en cuenta el [Acuerdo de Nivel de Servicio para Traffic Manager][tm-sla]. En el momento de redactar este artículo, el Acuerdo de Nivel de Servicio del de Traffic Manager es 99,99 %.

* Acuerdo de Nivel de Servicio compuesto = 99,99 % &times; (Acuerdo de Nivel de Servicio combinado para ambas regiones)

Además, la conmutación por error no es instantánea y puede dar lugar un cierto tiempo de inactividad durante ese momento. Consulte [Supervisión de puntos de conexión y de Traffic Manager][tm-failover].

El número calculado del Acuerdo de Nivel de Servicio es una base de referencia útil, pero no indica la historia completa sobre la disponibilidad. A menudo, una aplicación puede degradarse correctamente cuando se produce un error en una ruta no crítica. Considere una aplicación que muestre un catálogo de libros. Si la aplicación no puede recuperar la imagen en miniatura de la portada, puede que muestre una imagen de marcador de posición. En ese caso, no obtener la imagen no reduce el tiempo de actividad de la aplicación, aunque afecta a la experiencia del usuario.  

## <a name="design-for-resiliency"></a>Diseño para lograr resistencia

Durante la fase de diseño, se debe realizar un análisis del modo de error (FMA). El objetivo del análisis del modo de error es identificar los posibles puntos de error y definir cómo responderá la aplicación a esos errores.

* ¿Cómo detectará la aplicación este tipo de error?
* ¿Cómo responderá la aplicación a este tipo de error?
* ¿Cómo se va a registrar y supervisar este tipo de error? 

Para más información sobre el proceso de análisis del modo de error, con recomendaciones específicas para Azure, consulte [Azure resiliency guidance: Failure mode analysis][fma] (Guía de resistencia de Azure: análisis del modo de error).

### <a name="example-of-identifying-failure-modes-and-detection-strategy"></a>Ejemplo de identificación de modos de error y estrategia de detección
**Punto de error:** llamada a una API o servicio web externo.

| Modo de error | Estrategia de detección |
| --- | --- |
| El servicio no está disponible |HTTP 5xx |
| Limitaciones |HTTP 429 (Demasiadas solicitudes) |
| Autenticación |HTTP 401 (No autorizado) |
| Respuesta lenta |Tiempo de espera de la solicitud agotado |


### <a name="redundancy-and-designing-for-failure"></a>Redundancia y diseño en caso de error

El alcance de los errores puede variar. Algunos errores de hardware, como un problema en un disco, pueden afectar a un único equipo host. Un error en un conmutador de red podría afectar a todo un bastidor del servidor. Menos frecuentes son los errores que afectan a todo un centro de datos, como los problemas de alimentación. Aún más improbables son los problemas por los que toda una región dejaría de estar disponible.

Uno de los mecanismos para conseguir que una aplicación sea resistente es la redundancia. Sin embargo, esta redundancia debe planearse al diseñar la aplicación. Además, el nivel de redundancia necesario dependerá de los requisitos de la compañía: no todas las aplicaciones necesitan tener redundancia entre regiones para protegerse contra una interrupción regional. En general, hay que buscar el equilibrio, ya que una mayor redundancia y confiabilidad implica una mayor complejidad y unos costos más elevados.  

Azure dispone de una serie de características que permiten hacer que la aplicación sea redundante sea cual sea el nivel del error, desde los que se producen en una única máquina virtual hasta los que tienen lugar en toda una región. 

![](./images/redundancy.svg)

**Una única máquina virtual**. Azure proporciona un Acuerdo de Nivel de Servicio de tiempo de actividad para máquinas virtuales individuales. Aunque se puede conseguir un Acuerdo de Nivel de Servicio mayor ejecutando dos o más máquinas virtuales, una única máquina virtual puede resultar suficientemente confiable con algunas cargas de trabajo. En las cargas de trabajo de producción, se recomienda usar dos o más máquinas virtuales para tener redundancia. 

**Conjuntos de disponibilidad**. Para protegerse frente a errores de hardware localizados, como un error en un conmutador de red o un disco, implemente dos o más máquinas virtuales en un conjunto de disponibilidad. Un conjunto de disponibilidad se compone de dos o más *dominios de error* que comparten una fuente de alimentación y un conmutador de red. Las máquinas virtuales incluidas en un conjunto de disponibilidad se distribuyen entre los dominios de error, por lo que, si un error de hardware afecta a un dominio de error, el tráfico de la red puede enrutarse a las máquinas virtuales de otros dominios de error. Para más información acerca de los conjuntos disponibilidad, consulte [Administración de la disponibilidad de las máquinas virtuales Windows en Azure](/azure/virtual-machines/windows/manage-availability).

**Zonas de disponibilidad**.  Una zona de disponibilidad es una zona separada físicamente dentro de una región de Azure. Cada zona de disponibilidad tiene una fuente de alimentación, una red y un sistema de refrigeración distintos. Cuando las máquinas virtuales están implementadas en diferentes zonas de disponibilidad, es más fácil proteger una aplicación frente a errores que afectan a todo el centro de datos. 

**Regiones emparejadas**. Para proteger una aplicación frente a una interrupción regional, puede implementar la aplicación en varias regiones y utilizar Azure Traffic Manager para distribuir el tráfico de Internet en las distintas regiones. Cada región de Azure está emparejada con otra región. Juntas, forman un [par regional](/azure/best-practices-availability-paired-regions). A excepción del Sur de Brasil, los pares regionales se encuentran en la misma ubicación geográfica para, de este modo, cumplir los requisitos de residencia de datos a efectos de jurisdicción fiscal y aplicación de las leyes.

Si diseña una aplicación para varias regiones, tenga en cuenta que la latencia de red entre regiones es mayor que dentro de una región. Por ejemplo, si va a replicar una base de datos para habilitar la conmutación por error, utilice la replicación de datos sincrónica dentro de una misma región y la replicación asincrónica de datos entre diferentes regiones.

| &nbsp; | Conjunto de disponibilidad | Zona de disponibilidad | Región emparejada |
|--------|------------------|-------------------|---------------|
| Alcance del error | Bastidor | Centro de datos | Region |
| Enrutamiento de solicitudes | Load Balancer | Equilibrador de carga entre zonas | Traffic Manager |
| Latencia de red | Muy baja | Bajo | Media-alta |
| Virtual network  | VNet | VNet | Emparejamiento de VNet entre regiones |

## <a name="implement-resiliency-strategies"></a>Implementación de estrategias de resistencia
En esta sección se proporciona un estudio de algunas estrategias comunes de resistencia. La mayoría de ellas no se limitan a una tecnología en particular. Las descripciones de esta sección resumen la idea general que subyace en cada una de ellas, con vínculos a más información.

**Reintente los errores transitorios**. Los errores transitorios pueden deberse a una pérdida momentánea de conectividad de red, una conexión de base de datos caída o un tiempo de espera agotado cuando un servicio está ocupado. A menudo, un error transitorio se puede resolver simplemente con volver a intentar la solicitud. Para muchos de los servicios de Azure, el SDK de cliente implementa reintentos automáticos, de manera que es transparente para el autor de la llamada; consulte [Retry service specific guidance][retry-service-specific guidance] (Orientación específica sobre el servicio de reintentos).

Cada reintento se suma a la latencia total. Además, demasiadas solicitudes con error pueden causar un cuello de botella, ya que las solicitudes pendientes se acumulan en la cola. Estas solicitudes bloqueadas pueden contener recursos críticos del sistema, tales como la memoria, subprocesos o conexiones de base de datos, entre otros, que pueden causar errores en cascada. Para evitar este problema, aumente el retraso entre cada intento de reintento y limite el número total de solicitudes con error. 

![](./images/retry.png)

**Equilibre la carga entre instancias**. Para ofrecer escalabilidad, una aplicación en la nube debe ser capaz de escalar horizontalmente agregando más instancias. Este enfoque también mejora la resistencia, ya que los casos con estado incorrecto se pueden quitar de la rotación. Por ejemplo: 

* Coloque dos o más máquinas virtuales detrás de un equilibrador de carga. El equilibrador de carga distribuye el tráfico a todas las máquinas virtuales. Consulte [Run load-balanced VMs for scalability and availability][ra-multi-vm] (Ejecución de máquinas virtuales con equilibrio de carga por escalabilidad y disponibilidad).
* Escale horizontalmente una aplicación de Azure App Service en varias instancias. App Service equilibra automáticamente la carga entre instancias. Consulte [Basic web application][ra-basic-web] (Aplicación web básica).
* Use [Azure Traffic Manager][tm] para distribuir el tráfico a través de un conjunto de puntos de conexión.

**Replique los datos**. La replicación de datos es una estrategia general para tratar errores no transitorios en un almacén de datos. Muchas tecnologías de almacenamiento proporcionan replicación integrada, como Azure SQL Datase y Cosmos DB y Apache Cassandra. Es importante tener en cuenta las rutas de lectura y escritura. Según la tecnología de almacenamiento, puede tener varias réplicas de escritura, o una única réplica de escritura y múltiples réplicas de solo lectura. 

Para maximizar la disponibilidad, las réplicas pueden colocarse en varias regiones. Sin embargo, esto aumenta la latencia al replicar los datos. Por lo general, la replicación entre las regiones se realiza de forma asincrónica, lo que implica un modelo de coherencia final y la posible pérdida de datos si se produce un error en una réplica. 

**Degrade de manera correcta**. Si se produce un error en un servicio y no hay ninguna ruta de conmutación por error, es posible que la aplicación pueda degradarse correctamente y al mismo tiempo proporcionar una experiencia de usuario aceptable. Por ejemplo: 

* Coloque un elemento de trabajo en una cola, para su tratamiento posterior. 
* Devuelva un valor estimado.
* Use datos almacenados en caché local. 
* Muestre al usuario un mensaje de error. (Esta opción es mejor que dejar que la aplicación deje de responder a las solicitudes).

**Limitación a usuarios de gran volumen**. A veces un número reducido de usuarios crea una carga excesiva. Que puede tener un impacto en otros usuarios, lo que reduce la disponibilidad general de la aplicación.

Cuando un solo cliente realiza un número excesivo de solicitudes, la aplicación puede limitar al cliente durante un cierto período de tiempo. Durante el período de limitación, la aplicación rechaza algunas o todas las solicitudes de ese cliente (dependiendo de la estrategia exacta de limitación). El umbral para la limitación puede depender del nivel de servicio del cliente. 

La limitación no implica que el cliente haya actuado necesariamente de forma malintencionada, sino que ha superado su cuota de servicio. En algunos casos, un consumidor puede superar sistemáticamente su cuota o por lo demás comportarse incorrectamente. En ese caso, podría ir más lejos y bloquear al usuario. Normalmente, esto se hace bloqueando una clave de API o un rango de direcciones IP. Para más información, consulte [Throttling Pattern][throttling-pattern] (Patrón Throttling).

**Use un interruptor**. El patrón [Interruptor][circuit-breaker-pattern] puede evitar que una aplicación intente repetidamente realizar una operación que probablemente produzca errores. El interruptor encapsula las llamadas a un servicio y realiza el seguimiento del número de errores recientes. Si el número de errores supera un umbral, el interruptor comienza devolviendo un código de error sin llamar al servicio. Esto proporciona al servicio el tiempo necesario para recuperarse. 

**Use la nivelación de la carga para suavizar picos de tráfico**. Las aplicaciones pueden experimentar picos repentinos en el tráfico, lo que puede sobrecargar los servicios en el back-end. Si un servicio back-end no puede responder a las solicitudes con la suficiente rapidez, puede causar que las solicitudes se coloquen en cola (copia de seguridad) o que el servicio limite la aplicación. Para evitar esto, puede utilizar una cola como búfer. Cuando hay un nuevo elemento de trabajo, en lugar de llamar al servicio de back-end inmediatamente, la aplicación pone en cola un elemento de trabajo para ejecutarlo de forma asincrónica. La cola actúa como un búfer que suaviza los picos de carga. Para más información, consulte [Queue-Based Load Leveling Pattern][load-leveling-pattern] (Patrón de equilibrio de carga basado en colas).

**Aislamiento los recursos críticos**. Los errores en un subsistema a veces pueden producirse en cascada, lo que provoca errores en otras partes de la aplicación. Esto puede ocurrir si un error provoca que algunos recursos, como subprocesos o sockets, no se liberen a tiempo, lo que conduce a un agotamiento de los recursos. 

Para evitar esto, puede realizar una partición de un sistema en grupos aislados, de modo que un error en una partición no destruya todo el sistema. Esta técnica se denomina a veces, el patrón Bulkhead.

Ejemplos:

* Realizar una partición de una base de datos (por ejemplo, por inquilino) y asignar un grupo independiente de instancias del servidor web para cada partición.  
* Usar grupos de subprocesos independientes para aislar las llamadas a servicios diferentes. Esto ayuda a evitar errores en cascada, si se produce un error en uno de los servicios. Para ver un ejemplo, consulte la [biblioteca Hystrix][hystrix] de Netflix.
* Usar [contenedores][containers] para limitar los recursos disponibles para un subsistema determinado. 

![](./images/bulkhead.png)

**Aplique transacciones de compensación**. Una [transacción de compensación][compensating-transaction-pattern] es una transacción que deshace los efectos de otra transacción completada. En un sistema distribuido, puede ser muy difícil lograr una fuerte coherencia transaccional. Las transacciones de compensación son una forma de lograr coherencia mediante el uso de una serie de transacciones individuales más pequeñas que pueden deshacerse en cada paso.

Por ejemplo, para reservar un viaje, el cliente puede reservar un coche, una habitación de hotel y un vuelo. Si se produce un error en alguno de estos pasos, fallará toda la operación. En lugar de intentar utilizar una única transacción distribuida para toda la operación, puede definir una transacción de compensación para cada paso. Por ejemplo, para anular una reserva de coche, se cancela la reserva. Para completar toda la operación, un coordinador ejecuta cada paso. Si se produce un error en algún paso, el coordinador aplica transacciones de compensación para deshacer todos los pasos que se hayan completado. 

## <a name="test-for-resiliency"></a>Pruebas de resistencia
Por lo general, no se puede probar la resistencia de la misma manera que se prueba la funcionalidad de la aplicación (al ejecutar pruebas unitarias, por ejemplo). En su lugar, debe comprobar cómo se realiza la carga de trabajo de un extremo a otro en condiciones de error que solo se producen de forma intermitente.

Las pruebas son un proceso iterativo. Pruebe la aplicación, mida el resultado, analice y resuelva los errores que se producen, y repita el proceso.

**Pruebas de inserción de errores**. Pruebe la resistencia del sistema durante los errores, ya sea mediante la activación de errores reales o mediante su simulación. A continuación se presentan algunos escenarios de error comunes para probar:

* Cierre de instancias de máquina virtual.
* Bloqueo de procesos.
* Expiración de certificados.
* Cambio de claves de acceso.
* Cierre del servicio DNS en controladores de dominio.
* Límite de los recursos del sistema disponibles, como RAM o número de subprocesos.
* Desmontaje de discos.
* Reimplementación de una máquina virtual.

Medir los tiempos de recuperación y compruebe que se cumplen los requisitos de negocio. Pruebe también combinaciones de modos de error. Asegúrese de que los errores no están en cascada y se traten de forma aislada.

Este es otro motivo por el cual es importante analizar los posibles puntos de error durante la fase de diseño. Los resultados de ese análisis deben incluirse en el plan de pruebas.

**Prueba de carga**. La prueba de carga es fundamental para identificar errores que se producen solo bajo carga, como que la base de datos de back-end esté desbordada o que haya una limitación del servicio. Realice pruebas de carga máxima, mediante el uso de datos de producción o datos sintéticos que estén lo más cerca posible de los datos de producción. El objetivo es ver cómo se comporta la aplicación en condiciones reales.   

## <a name="deploy-using-reliable-processes"></a>Implementación mediante procesos confiables
Una vez que una aplicación se implementa en producción, las actualizaciones son un posible origen de errores. En el peor de los casos, una actualización incorrecta puede dar lugar a tiempos de inactividad. Para evitarlo, el proceso de implementación debe ser predecible y repetible. La implementación incluye el aprovisionamiento de recursos de Azure, la implementación del código de aplicación y la aplicación de las opciones de configuración. Una actualización puede implicar a los tres o a un subconjunto. 

El punto fundamental es que las implementaciones manuales son propensas a errores. Por lo tanto, se recomienda tener un proceso automatizado e idempotente que pueda ejecutar a petición y volver a ejecutarlo si se produce un error. 

* Use plantillas de Azure Resource Manager para automatizar el aprovisionamiento de recursos de Azure.
* Utilice [Desired State Configuration (DSC) de Azure Automation][dsc] para configurar las máquinas virtuales.
* Utilice un proceso de implementación automatizada para el código de aplicación.

Dos conceptos relacionados con la implementación resistente son la *infraestructura como código* y la *infraestructura inmutable*.

* La **infraestructura como código** es la práctica de utilizar código para aprovisionar y configurar la infraestructura. La infraestructura como código puede utilizar un enfoque declarativo o un enfoque imperativo (o una combinación de ambos). Las plantillas de Resource Manager son un ejemplo de un enfoque declarativo. Los scripts de PowerShell son un ejemplo de un enfoque imperativo.
* **Infraestructura inmutable** implica que no se debe modificar la infraestructura una vez implementada en producción. En caso contrario, puede entrar en un estado donde se han aplicado cambios ad hoc, por lo que resulta difícil saber exactamente qué es lo que ha cambiado y difícil pensar sobre el sistema. 

Otra cuestión es cómo poner en marcha una actualización de la aplicación. Se recomiendan técnicas como la implementación Blue-Green o lanzamientos controlados, que insertan actualizaciones de forma altamente controlada para minimizar los posibles impactos de una implementación incorrecta.

* La [implementación Blue-Green][blue-green] es una técnica en la que se implementa una actualización en un entorno de producción independiente de la aplicación activa. Después de validar la implementación, cambie el enrutamiento de tráfico a la versión actualizada. Por ejemplo, Azure App Service Web Apps lo habilita con espacios de ensayo.
* Los [lanzamientos controlados][canary-release] son similares a las implementaciones Blue-Green. En lugar de cambiar todo el tráfico a la versión actualizada, se implementa la actualización a un pequeño porcentaje de usuarios, mediante el enrutamiento de una parte del tráfico a la nueva implementación. Si hay un problema, retroceda y vuelva a la implementación anterior. De lo contrario, enrute más del tráfico a la nueva versión, hasta que se consiga el 100 % del tráfico.

Sea cual sea el enfoque que se adopte, asegúrese de que puede volver a la última versión conocida, en caso de que la nueva versión no funcione. Además, si se producen errores, los registros de aplicaciones deben indicar qué versión ha causado el error. 

## <a name="monitor-to-detect-failures"></a>Supervisión para detectar errores
La supervisión y el diagnóstico son fundamentales para proporcionar resistencia. Si algo va mal, debe saber que se produjo un error y necesita comprender la causa del error. 

La supervisión de un sistema distribuido a gran escala plantea un desafío importante. Piense en una aplicación que se ejecuta en unas pocas docenas de máquinas virtuales; no es práctico volver a iniciar sesión en cada máquina virtual, una a la vez, y examine los archivos de registro tratando de solucionar un problema. Además, el número de instancias de máquinas virtuales probablemente no es estático. Las máquinas virtuales se agregan o se quitan a medida que la aplicación se reduce o se escala horizontalmente, y en ocasiones pueden producirse errores en una instancia y necesita ser reaprovisionada. Además, una aplicación en la nube típica puede utilizar varios almacenes de datos (Azure Storage, SQL Database, Cosmos DB, Redis Cache) y una acción de un único usuario puede abarcar varios subsistemas. 

El proceso de supervisión y diagnóstico se puede considerar como una canalización con varias fases distintas:

![Acuerdo de Nivel de Servicio compuesto](./images/monitoring.png)

* **Instrumentación**. Los datos sin procesar para la supervisión y el diagnóstico provienen de una variedad de orígenes, incluidos registros de aplicación, registros de servidor web, contadores de rendimiento del sistema operativo, registros de base de datos y diagnóstico integrado en la plataforma Azure. La mayoría de los servicios de Azure tienen una característica de diagnóstico que puede usar para determinar la causa de los problemas.
* **Recopilación y almacenamiento**. Se pueden guardar los datos de instrumentación sin formato en varias ubicaciones y con diversos formatos (por ejemplo, registros de seguimiento de aplicaciones, registros de IIS, contadores de rendimiento). Estos orígenes dispares se recopilan, se consolidan y se colocan en un almacenamiento confiable.
* **Análisis y diagnóstico**. Después de consolidar los datos, se pueden analizar para solucionar problemas y proporcionar una visión general del estado de las aplicaciones.
* **Visualización y alertas**. En esta fase, los datos de telemetría se presentan de tal manera que un operador puede detectar rápidamente problemas o tendencias. En el ejemplo se incluyen paneles o alertas por correo electrónico.  

La supervisión no es lo mismo que la detección de errores. Por ejemplo, la aplicación podría detectar un error transitorio y volver a intentarlo, lo que resultaría en ningún tiempo de inactividad. Pero también debe registrar la operación de reintento, para que pueda supervisar la tasa de errores, a fin de obtener una imagen general del estado de la aplicación. 

Los registros de aplicación constituyen un origen importante de datos de diagnóstico. Los procedimientos recomendados para el registro de aplicaciones incluyen:

* Iniciar sesión en producción. De lo contrario, perderá la perspectiva donde más la necesita.
* Registrar los eventos en los límites del servicio. Incluya un identificador de correlación que traspase los límites del servicio. Si una transacción fluye a través de varios servicios y se produce un error en uno de ellos, el identificador de correlación le ayudará identificar el motivo del error de la transacción.
* Utilizar el registro semántico, también conocido como registro estructurado. Los registros no estructurados dificultan la automatización del consumo y el análisis de los datos de registro que se necesitan a escala de nube.
* Usar llamadas asincrónicas. De lo contrario, el mismo sistema de registro puede hacer que la aplicación produzca un error causando la copia de seguridad de las solicitudes, ya que se bloquean mientras esperan para escribir un evento de registro.
* El registro de aplicaciones no es lo mismo que auditoría. La auditoría puede realizarse por motivos de cumplimiento o normativos. Por tanto, los registros de auditoría deben estar completos y no se aconseja quitar ninguno mientras se procesan las transacciones. Si una aplicación requiere auditoría, esta debe mantenerse separada del registro de diagnóstico. 

Para más información sobre la supervisión y diagnóstico, consulte la guía [Monitoring and diagnostics][monitoring-guidance] (Supervisión y diagnóstico).

## <a name="respond-to-failures"></a>Respuesta a los errores
Las secciones anteriores se han centrado en las estrategias de recuperación automatizada, que son fundamentales para la alta disponibilidad. Sin embargo, a veces es necesaria la intervención manual.

* **Alertas**. Supervise la aplicación para detectar señales de advertencia que puedan requerir una intervención proactiva. Por ejemplo, si observa que SQL Database o Cosmos DB limitan constantemente la aplicación, es posible que tenga que aumentar la capacidad de la base de datos u optimizar las consultas. En este ejemplo, aunque la aplicación puede tratar los errores de limitación de forma transparente, la telemetría debería generar una alerta para que se pueda realizar el seguimiento.  
* **Conmutación por error manual**. Algunos sistemas no pueden conmutar por error automáticamente y requieren una conmutación por error manual. 
* **Pruebas de disponibilidad operativa**. Si la aplicación conmuta por error en una región secundaria, debe realizar una prueba de disponibilidad operativa antes de conmutar por recuperación en la región primaria. La prueba debe comprobar que la región primaria es correcta y que está lista para recibir tráfico de nuevo.
* **Comprobación de la coherencia de datos**. Si se produce un error en un almacén de datos, puede haber incoherencias de datos cuando el almacén vuelve a estar disponible, especialmente si los datos se han replicado. 
* **Restauración desde la copia de seguridad**. Por ejemplo, si SQL Database experimenta una interrupción regional, puede realizar una restauración geográfica de la base de datos desde la última copia de seguridad.

Documente y pruebe el plan de recuperación ante desastres. Evalúe el impacto de negocio de los errores de las aplicaciones. Automatice el proceso en la medida de lo posible y documente todos los pasos manuales, como la conmutación por error manual o la restauración de datos de las copias de seguridad. Pruebe regularmente el proceso de recuperación ante desastres para validar y mejorar el plan. 

## <a name="summary"></a>Resumen
En este artículo se ha descrito la resistencia desde una perspectiva holística, haciendo hincapié en algunos de los desafíos únicos de la nube. Estos incluyen la naturaleza distribuida de la informática en la nube, el uso de hardware básico y la presencia de errores transitorios en la red.

Estos son los puntos principales que se pueden extraer de este artículo:

* La resistencia conduce a una mayor disponibilidad y a un menor tiempo medio para recuperarse de los errores. 
* Alcanzar la resistencia en la nube requiere un conjunto de técnicas diferentes a las soluciones locales tradicionales. 
* La resistencia no ocurre por accidente. Debe diseñarse e integrarse desde el principio.
* La resistencia afecta a las partes de la vigencia de la aplicación, desde el planeamiento y codificación a las operaciones.
* ¡Pruebe y supervise!


<!-- links -->

[blue-green]: https://martinfowler.com/bliki/BlueGreenDeployment.html
[canary-release]: https://martinfowler.com/bliki/CanaryRelease.html
[circuit-breaker-pattern]: https://msdn.microsoft.com/library/dn589784.aspx
[compensating-transaction-pattern]: https://msdn.microsoft.com/library/dn589804.aspx
[containers]: https://en.wikipedia.org/wiki/Operating-system-level_virtualization
[dsc]: /azure/automation/automation-dsc-overview
[contingency-planning-guide]: https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-34r1.pdf
[fma]: failure-mode-analysis.md
[hystrix]: https://medium.com/netflix-techblog/introducing-hystrix-for-resilience-engineering-13531c1ab362
[jmeter]: https://jmeter.apache.org/
[load-leveling-pattern]: ../patterns/queue-based-load-leveling.md
[monitoring-guidance]: ../best-practices/monitoring.md
[ra-basic-web]: ../reference-architectures/app-service-web-app/basic-web-app.md
[ra-multi-vm]: ../reference-architectures/virtual-machines-windows/multi-vm.md
[checklist]: ../checklist/resiliency.md
[retry-pattern]: ../patterns/retry.md
[retry-service-specific guidance]: ../best-practices/retry-service-specific.md
[sla]: https://azure.microsoft.com/support/legal/sla/
[throttling-pattern]: ../patterns/throttling.md
[tm]: https://azure.microsoft.com/services/traffic-manager/
[tm-failover]: /azure/traffic-manager/traffic-manager-monitoring
[tm-sla]: https://azure.microsoft.com/support/legal/sla/traffic-manager/v1_0/
