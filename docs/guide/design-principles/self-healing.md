---
title: "Diseño para la recuperación automática"
description: "Las aplicaciones resistentes pueden recuperarse de errores sin intervención manual."
author: MikeWasson
layout: LandingPage
ms.openlocfilehash: 0782b65b77615f7c006724264ab0ca2d2c7c04e2
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2017
---
# <a name="design-for-self-healing"></a>Diseño para la recuperación automática

## <a name="design-your-application-to-be-self-healing-when-failures-occur"></a>Diseñe la aplicación para que se recupere automáticamente cuando esto suceda.

En un sistema distribuido, se producen errores. Pueden producirse errores de hardware. La red puede tener errores transitorios. En raras ocasiones, todo un servicio o toda una región pueden experimentar interrupciones, pero incluso estas deben planificarse.

Por tanto, diseñe una aplicación para que se recupere automáticamente cuando esto suceda. Esto requiere un enfoque basado en tres puntos:

- Detectar errores.
- Responder a los errores correctamente.
- Registrar y supervisar los errores a fin de conseguir una perspectiva operativa.

La manera de responder a un determinado tipo de error puede depender de los requisitos de disponibilidad de su aplicación. Por ejemplo, si necesita una disponibilidad muy alta, puede conmutar por error automáticamente a una región secundaria durante una interrupción regional del sistema. Sin embargo, con esto incurrirá en un costo mayor que el de una implementación en una sola región. 

Además, no piense solo en grandes eventos como interrupciones regionales, que suelen ser poco frecuentes. Debe centrarse igualmente, si no más, en la gestión de los errores locales y de corta duración, como los errores de conectividad de red o los errores en la conexión a una base de datos.

## <a name="recommendations"></a>Recomendaciones

**Vuelva a intentar las operaciones que den error**. Los errores transitorios pueden deberse a una pérdida momentánea de conectividad de red, una conexión de base de datos caída o un tiempo de espera agotado cuando un servicio está ocupado. Cree la lógica de reintento en la aplicación para controlar los errores transitorios. En muchos de los servicios de Azure, el SDK de cliente implementa los reintentos automáticos. Para obtener más información, consulte [Transient fault handling][transient-fault-handling] (Control de errores transitorios) y [Retry Pattern][retry] (Patron de reintento).

**Proteja los servicios remotos defectuosos (Circuit Breaker)**. Es conveniente hacer un reintento después de un error transitorio, pero si el error persiste, puede acabar con demasiados llamadores agolpándose en un servicio con errores. Esto puede provocar errores en cascada, ya que se hace una copia de seguridad de las solicitudes. Use el [patrón Circuit Breaker][circuit-breaker] para pasar rápidamente por un error (sin realizar la llamada remota) cuando es probable que una operación genere un error.  

**Aísle recursos críticos (Bulkhead)**. Los errores de un subsistema a veces pueden reproducirse en cascada. Esto puede ocurrir si un error provoca que algunos recursos, como subprocesos o sockets, no se liberen a tiempo, lo que conduce a un agotamiento de estos. Para evitarlo, realice una partición de un sistema en grupos aislados, de modo que un error en una partición no destruya todo el sistema.  

**Realice una redistribución de la carga**. Las aplicaciones pueden experimentar picos repentinos en el tráfico, lo que puede sobrecargar los servicios en el back-end. Para evitarlo, utilice el [patrón Queue-Based Load Leveling][load-level] para poner en cola los elementos de trabajo que se ejecutan de forma asincrónica. La cola actúa como un búfer que suaviza los picos de carga. 

**Realice una conmutación por error**. Si no se puede tener acceso a una instancia, aplique una conmutación por error a otra instancia. En el caso de los elementos que no tienen estado, como un servidor web, coloque varias instancias detrás de un equilibrador de carga o el administrador de tráfico. En el caso de los elementos que almacenan el estado, como una base de datos, use réplicas y la conmutación por error. En función del almacén de datos y del modo en que se replique, esto puede requerir que la aplicación adopte la coherencia final. 

**Compense las transacciones con error**. En general, evite las transacciones distribuidas, ya que requieren coordinación a través de servicios y recursos. En su lugar, componga una operación a partir de transacciones individuales más pequeñas. Si la operación genera un error a mitad del proceso, use [las transacciones de compensación][compensating-transactions] para deshacer cualquier paso que ya haya completado. 

**Aplique puntos de control a las transacciones de ejecución prolongada**. Los puntos de control pueden proporcionar resistencia si se produce un error en una operación de ejecución prolongada. Cuando la operación se reinicia (por ejemplo, si la recoge otra máquina virtual), es posible reanudar desde el último punto de control.

**Degrade de manera correcta**. En ocasiones no es posible solucionar un problema, pero puede proporcionar una funcionalidad reducida que sigue siendo útil. Piense en una aplicación que muestre un catálogo de libros. Si la aplicación no puede recuperar la imagen en miniatura de la portada, puede que muestre una imagen de marcador de posición. Es posible que para la aplicación no sea esencial contar con subsistemas completos. Por ejemplo, en un sitio de comercio electrónico, mostrar las recomendaciones de productos es probablemente menos crítico que procesar los pedidos.

**Limite los clientes**. En ocasiones, un número reducido de usuarios crea una carga excesiva, lo que puede reducir la disponibilidad de la aplicación para otros usuarios. En esta situación, limite el cliente durante un período de tiempo determinado. Vea el [patrón de limitación][throttle].

**Bloquee a los actores no válidos**. El hecho de que limite a un cliente no significa que ese cliente estuviera actuando de manera malintencionada. Simplemente significa que el cliente ha superado su cuota de servicio. Pero si un cliente supera constantemente su cuota o se comporta de manera incorrecta, tiene la posibilidad de bloquearlo. Defina un proceso fuera de banda para que el usuario solicite ser desbloqueado.

**Use Leader Election**. Cuando necesite coordinar una tarea, use [Leader Election][leader-election] para seleccionar un coordinador. De este modo, el coordinador no es un único punto de error. Si se produce un error en el coordinador, se selecciona uno nuevo. En lugar de implementar un algoritmo de elección de líder desde el principio, considere la posibilidad de una solución inmediata, como Zookeeper.  

**Realice pruebas con inserción de errores**. Con demasiada frecuencia, la ruta de acceso correcta está bien probada, pero no así la ruta de acceso para errores. Un sistema podría ejecutarse en producción durante mucho tiempo antes de que se utilice una ruta de acceso para errores. Use la inserción de errores para probar la resistencia del sistema a los errores, ya sea mediante la activación de errores reales o mediante su simulación. 

**Adopte la ingeniería del caos**. La ingeniería del caos amplía la noción de inserción de errores mediante la inserción aleatoria de errores o condiciones irregulares en instancias de producción. 

Para consultar un enfoque estructurado que haga que sus aplicaciones se recuperen de manera automática, consulte [Diseño de aplicaciones resistentes de Azure][resiliency-overview].  

[circuit-breaker]: ../../patterns/circuit-breaker.md
[compensating-transactions]: ../../patterns/compensating-transaction.md
[leader-election]: ../../patterns/leader-election.md
[load-level]: ../../patterns/queue-based-load-leveling.md
[resiliency-overview]: ../../resiliency/index.md
[retry]: ../../patterns/retry.md
[throttle]: ../../patterns/throttling.md
[transient-fault-handling]: ../../best-practices/transient-faults.md

