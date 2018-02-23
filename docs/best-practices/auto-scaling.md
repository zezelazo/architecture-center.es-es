---
title: "Instrucciones de escalado automático"
description: "Orientación sobre cómo usar el escalado automático para asignar dinámicamente los recursos que requiere una aplicación."
author: dragon119
ms.date: 05/17/2017
pnp.series.title: Best Practices
ms.openlocfilehash: a8489aaabab2b8523fbc9f026f4f435bb6d1ad29
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/23/2018
---
# <a name="autoscaling"></a>Escalado automático
[!INCLUDE [header](../_includes/header.md)]

El escalado automático es el proceso por el cual se asignan recursos dinámicamente para satisfacer los requisitos de rendimiento. A medida que aumenta el volumen de trabajo, una aplicación puede necesitar más recursos para mantener los niveles de rendimiento deseados y cumplir los Acuerdos de Nivel de Servicio (SLA). Cuando la demanda se reduce y los recursos adicionales ya no son necesarios, se pueden desasignar para minimizar los costos.

El escalado automático aprovecha la elasticidad de entornos hospedados en la nube mientras alivia la sobrecarga de administración. Reduce la necesidad de que un operador tenga que supervisar continuamente el rendimiento de un sistema y tomar decisiones sobre agregar o quitar recursos.

Una aplicación se puede escalar fundamentalmente de dos maneras: 

* **Escalado vertical**, que significa cambiar la capacidad de un recurso. Por ejemplo, puede mover una aplicación a un tamaño de máquina virtual mayor. El escalado vertical suele requerir que el sistema deje de estar disponible temporalmente mientras se vuelve a implementar. Por lo tanto, es menos habitual automatizar el escalado vertical.
* **Escalado horizontal**, que significa agregar o quitar instancias de un recurso. La aplicación puede seguir ejecutándose sin interrupciones mientras se aprovisionan los nuevos recursos. Una vez completado el proceso de aprovisionamiento, la solución se implementa en estos recursos adicionales. Si cae la demanda, los recursos adicionales pueden cerrarse limpiamente y desasignarse. 

Muchos sistemas basados en la nube, incluido Microsoft Azure, admiten la automatización del escalado horizontal. El resto de este artículo se centra en el escalado horizontal.

> [!NOTE]
> El escalado automático se aplica principalmente a los recursos de proceso. Aunque es posible escalar horizontalmente una cola de mensajes o una base de datos, normalmente, esto implica [particionar los datos][data-partitioning], que generalmente no se automatiza.
>

## <a name="overview"></a>Información general

La implementación de una estrategia de escalado automático suele implicar las siguientes partes:

* Sistemas de instrumentación y supervisión en los niveles de aplicación, servicio e infraestructura. Estos sistemas capturan métricas clave, como tiempos de respuesta, longitudes de cola, uso de CPU y uso de memoria.
* La lógica de toma de decisiones que evalúa estas métricas frente a las programaciones o los umbrales predefinidos, y que decide si se debe escalar.
* Los componentes que escalan el sistema.
* Pruebas, supervisión y ajuste de la estrategia de escalado automático para asegurarse de que funciona según lo esperado.

Azure ofrece mecanismos de escalado automático integrados que abordan escenarios comunes. Si un servicio o una tecnología determinados no tiene funcionalidad de escala automática integrada, o si tiene requisitos de escalado automático específicos más allá de sus capacidades, puede considerar una implementación personalizada. Una implementación personalizada podría recopilar métricas operativas y del sistema, analizarlas y, luego, escalar los recursos en consecuencia.

## <a name="configure-autoscaling-for-an-azure-solution"></a>Configuración de escalado automático para una solución de Azure

Azure proporciona escalado automático integrado para la mayoría de las opciones de proceso.

* **Virtual Machines** admite el escalado automático mediante [VM Scale Sets][vm-scale-sets], que es una forma de administrar un conjunto de máquinas virtuales de Azure como un grupo. Consulte [Procedimiento para usar el escalado automático y los conjuntos de escalado de máquinas virtuales][vm-scale-sets-autoscale].

* **Service Fabric** también admite el escalado automático mediante VM Scale Sets. Cada tipo de nodo en un clúster de Service Fabric está configurado como un conjunto de escalado de máquinas virtuales independiente. De este modo, cada tipo de nodo se puede escalar horizontalmente por separado. Consulte [Escalado o reducción horizontal de un clúster de Service Fabric usando reglas de escalado automático][service-fabric-autoscale].

* **Azure App Service** tiene escalado automático integrado. La configuración de escalado automático se aplica a todas las aplicaciones dentro de una instancia de App Service. Consulte [Escalado manual o automático del número de instancias][app-service-autoscale].

* **Azure Cloud Services** tiene escalado automático integrado en el nivel de rol. Consulte [Procedimiento para configurar el escalado automático para un servicio en la nube en el Portal][cloud-services-autoscale].

Todas estas opciones de proceso usan [escalado automático de Azure Monitor][monitoring] para proporcionar un conjunto común de funcionalidad de escalado automático.

* **Azure Functions** difiere de las opciones de proceso anteriores, porque no es necesario configurar reglas de escalado automático. En su lugar, Azure Functions asigna automáticamente capacidad de proceso cuando se ejecuta el código, y escala horizontalmente según sea necesario para administrar la carga. Para más información, vea [Comparación de los planes de hospedaje de Azure Functions][functions-scale].

Por último, a veces resulta útil una solución de escalado automático personalizada. Por ejemplo, podría usar Azure Diagnostics y métricas de aplicación, además de código personalizado, para supervisar y exportar las métricas de la aplicación. Después, podría definir reglas personalizadas basadas en estas métricas y usar las API de REST de Resource Manager para desencadenar el escalado automático. Pero una solución personalizada no es fácil de implementar y solo debería contemplarse si ninguno de los métodos anteriores puede satisfacer los requisitos.

Si satisfacen sus requisitos, use las características de escalado automático integradas de la plataforma. Si no es así, considere detenidamente si realmente necesita características de escalado más complejas. Algunos ejemplos de requisitos adicionales podrían ser un control más detallado, diferentes maneras de detectar eventos desencadenadores del escalado, escalado en varias suscripciones y escalado de otros tipos de recursos.

## <a name="use-azure-monitor-autoscale"></a>Uso del escalado automático de Azure Monitor

El [escalado automático de Azure Monitor][monitoring] proporciona un conjunto común de funciones de escalado automático para VM Scale Sets, Azure App Service y Azure Cloud Services. El escalado se puede realizar según una programación o en función de una métrica en tiempo de ejecución, como el uso de CPU o memoria. Ejemplos:

- Aumentar a 10 instancias los días laborables y reducir a 4 instancias el sábado y el domingo. 
- Aumentar una instancia si el uso medio de la CPU es superior al 70 % y reducir una instancia si el uso de la CPU cae por debajo del 50 %.
- Aumentar una instancia si el número de mensajes en una cola supera un cierto umbral.

Para obtener una lista de las métricas integradas, consulte [Métricas comunes de escalado automático de Azure Monitor][autoscale-metrics]. También puede implementar métricas personalizadas mediante Application Insights. 

Puede configurar el escalado automático con PowerShell, la CLI de Azure, una plantilla de Azure Resource Manager o Azure Portal. Para obtener más control, use la [API de REST de Azure Resource Manager](https://msdn.microsoft.com//library/azure/dn790568.aspx). La [biblioteca de administración de servicios de supervisión de Azure](http://www.nuget.org/packages/Microsoft.WindowsAzure.Management.Monitoring) y la [biblioteca de Microsoft Insights](https://www.nuget.org/packages/Microsoft.Azure.Insights/) (en versión preliminar) son SDK que permiten recopilar métricas de diferentes recursos y realizar escalado automático haciendo uso de las API de REST. En recursos en los que no existe compatibilidad con el Azure Resource Manager, o si se utiliza Azure Cloud Services, puede usar la API de REST de Administración de servicios para escalado automático. En los demás casos, utilice Azure Resource Manager.

Al usar el escalado automático de Azure, tenga en cuenta los siguientes puntos:

* Tenga en cuenta si puede predecir la carga en la aplicación con precisión suficiente como para usar el escalado automático programado (adición y eliminación de instancias para satisfacer los picos de demanda anticipados). Si no es posible, use el escalado automático reactivo en función de las métricas recopiladas en tiempo de ejecución para administrar los cambios imprevisibles en la demanda. Normalmente, puede combinar estos métodos. Por ejemplo, cree una estrategia que agregue recursos según una programación de las horas en las que sabe que la aplicación está más ocupada. Esto ayuda a garantizar que la capacidad estará disponible cuando se necesite, sin provocar el retraso que se produce al iniciar nuevas instancias. Para cada regla programada, defina métricas que permitan el escalado automático reactivo durante ese período para asegurarse de que la aplicación pueda atender picos sostenidos pero imprevisibles en la demanda.
* A menudo, resulta difícil comprender la relación entre las métricas y los requisitos de capacidad, especialmente durante la implementación inicial de una aplicación. Aprovisione un poco de capacidad adicional al principio y, luego, supervise y optimice las reglas de escalado automático para aproximar la capacidad a la carga real.
* Configure reglas de escalado automático y, luego, supervise el rendimiento de la aplicación con el tiempo. Use los resultados de esta supervisión para ajustar la manera en que se escala el sistema, si fuera necesario. Pero tenga en cuenta que el escalado automático no es un proceso instantáneo. Se tarda en reaccionar ante una métrica, como, por ejemplo, un uso medio de CPU que supere un umbral especificado o sea inferior a este.
* Las reglas de escalado automático que usan un mecanismo de detección basado en un atributo desencadenador medido (por ejemplo, el uso de CPU o la longitud de cola) usan un valor agregado a lo largo del tiempo, en lugar de valores instantáneos, para desencadenar una acción de escalado automático. De forma predeterminada, el agregado es un promedio de los valores. Esto impide que el sistema reaccione demasiado rápido o que provoque una oscilación rápida. También permite tiempo para que las nuevas instancias de inicio automático entren en modo de ejecución, lo que impide que se produzcan acciones adicionales de escalado automático mientras se inician las nuevas instancias. En el caso de Azure Cloud Services y Azure Virtual Machines, el período predeterminado para la agregación es de 45 minutos, por lo que la métrica puede tardar hasta este período de tiempo para desencadenar el escalado automático en respuesta a picos de demanda. Puede cambiar el período de agregación mediante el SDK. Pero tenga en cuenta que los períodos de menos de 25 minutos pueden producir resultados imprevisibles (para más información, consulte [Escalado automático de Cloud Services según el porcentaje de CPU con Azure Monitoring Services Management Library](http://rickrainey.com/2013/12/15/auto-scaling-cloud-services-on-cpu-percentage-with-the-windows-azure-monitoring-services-management-library/)). En el caso de Web Apps, el período medio es mucho más corto, lo que permite que las nuevas instancias estén disponibles en unos cinco minutos después de que se produzca un cambio en la medida de desencadenador media.
* Si configura el escalado automático mediante el SDK en lugar del portal, puede especificar una programación más detallada durante la cuál las reglas estarán activas. También puede crear sus propias métricas y usarlas con o sin las métricas existentes en sus reglas de escalado automático. Por ejemplo, quizás desee usar contadores alternativos, como, por ejemplo, el número de solicitudes por segundo o la disponibilidad media de memoria, o bien usar contadores personalizados que midan procesos empresariales específicos.
* Con el escalado de Service Fabric, los tipos de nodo del clúster están formados por conjuntos de escalado de máquinas virtuales en el back-end, por lo que tendrá que configurar reglas de escalado automático para cada tipo de nodo. Antes de configurar el escalado automático, tenga en cuenta el número de nodos que debe tener. El número mínimo de nodos que debe tener para el tipo de nodo principal está controlado por el nivel de confiabilidad que haya elegido. Para más información, consulte [Escalado de un clúster de Service Fabric usando reglas de escalado automático](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-scale-up-down).
* Puede usar el portal para vincular recursos, tales como colas e instancias de SQL Database, a una instancia de un servicio en la nube. Esto permite acceder más fácilmente las opciones separadas de configuración de escalado manual y automático para cada uno de los recursos vinculados. Para más información, consulte [Vinculación de un recurso a un servicio en la nube](/azure/cloud-services/cloud-services-how-to-manage).
* Si configura varias directivas y reglas, podrían producirse conflictos entre ellas. El escalado automático usa las siguientes reglas de resolución de conflictos para garantizar que siempre haya un número suficiente de instancias en ejecución:
  * Las operaciones de escalar horizontalmente siempre tienen prioridad sobre las operaciones de reducir horizontalmente.
  * Cuando se produce un conflicto en las operaciones de escalar horizontalmente, tiene prioridad la regla que inicia el mayor aumento en el número de instancias.
  * Cuando se produce un conflicto en las operaciones de reducir horizontalmente, tiene prioridad la regla que inicia la menor reducción en el número de instancias.
* En un entorno de App Service Environment, se puede usar cualquier grupo de trabajo o métrica de front-end para definir las reglas de escalado automático. Para más información, consulte [Escalado automático y App Service Environment](/azure/app-service/app-service-environment-auto-scale).

## <a name="application-design-considerations"></a>Consideraciones sobre el diseño de aplicaciones
El escalado automático no es una solución instantánea. La simple adición de recursos a un sistema o la ejecución de instancias adicionales de un proceso no garantiza que mejorará el rendimiento del sistema. Tenga en cuenta lo siguiente a la hora de diseñar una estrategia de escalado automático:

* El sistema debe diseñarse para el escalado horizontal. Evite hacer suposiciones acerca de la afinidad de la instancia; no diseñe soluciones que requieran la ejecución constante del código en una instancia específica de un proceso. Al escalar horizontalmente un servicio en la nube o sitio web, no presuponga que una serie de solicitudes del mismo origen siempre se enrutará a la misma instancia. Por la misma razón, diseñe los servicios sin estado para evitar la necesidad de disponer de una serie de solicitudes de que una aplicación siempre se dirija a la misma instancia de un servicio. Al diseñar un servicio que lee y procesa mensajes de una cola, no haga ninguna suposición sobre qué instancia de servicio controla un mensaje específico. El escalado automático podría iniciar instancias adicionales de un servicio a medida que crezca la longitud de la cola. En el tema [Patrón Competing Consumers][competing-consumers] se describe cómo administrar este escenario.
* Si la solución implementa una tarea de ejecución prolongada, diseñe esta tarea para que admita el escalado horizontal (tanto reducción como ampliación). Sin la debida atención, este tipo de tarea podría impedir que una instancia de un proceso se cierre correctamente al reducir horizontalmente el sistema, o bien podría perder datos si el proceso finaliza de manera forzada. De manera ideal, refactorice una tarea de ejecución prolongada y divida el procesamiento que realiza en fragmentos más pequeños y discretos. En el tema [Patrón Pipes and Filters][pipes-and-filters] se ofrece un ejemplo de cómo hacerlo.
* Como alternativa, puede implementar un mecanismo de punto de comprobación que registre información sobre el estado de la tarea en intervalos regulares y que guarde este estado en almacenamiento duradero accesible para cualquier instancia del proceso que ejecute la tarea. De este modo, si el proceso se cierra, el trabajo que estaba realizando puede reanudarse desde el último punto de comprobación con otra instancia.
* Cuando las aplicaciones en segundo plano se ejecutan en instancias de proceso independientes, como es el caso en los roles de trabajo de una aplicación hospedada en los Servicios en la nube, es posible que necesite escalar diferentes partes de la aplicación con distintas directivas de escalado. Por ejemplo, es posible que tenga que implementar instancias adicionales de proceso de interfaz de usuario sin aumentar el número de instancias de proceso en segundo plano o viceversa. Si ofrece diferentes niveles de servicio (por ejemplo, paquetes de servicio básico y premium), es posible que, para cumplir el SLA, tenga que escalar horizontalmente los recursos de proceso para los paquetes de servicio premium de forma más agresiva que aquellos para los paquetes de servicio básico.
* Considere la posibilidad de usar la longitud de la cola sobre la que se comunican las instancias de proceso de interfaz de usuario y en segundo plano como criterio para la estrategia de escalado automático. Este es el mejor indicador de la existencia de un desequilibrio o diferencia entre la carga actual y la capacidad de procesamiento de la tarea en segundo plano.
* Si basa su estrategia de escalado automático en contadores que miden procesos empresariales, tal como el número de pedidos realizados por hora o el tiempo de ejecución promedio de una transacción compleja, asegúrese de comprender a fondo la relación entre los resultados de estos tipos de contadores y los requisitos reales de capacidad de proceso. Es posible que sea necesario escalar más de un componente o unidad de proceso en respuesta a los cambios en los contadores de proceso de negocio.  
* Para impedir que un sistema intente escalar horizontalmente de manera excesiva y evitar los costos asociados con la ejecución de varios miles de instancias, considere la posibilidad de limitar el número máximo de instancias que se pueden agregar automáticamente. La mayoría de los mecanismos de escalado automático le permiten especificar el número mínimo y máximo de instancias de una regla. Considere también la posibilidad de degradar correctamente la funcionalidad que brinda el sistema si se implementó el número máximo de instancias y el sistema aún está sobrecargado.
* Tenga en cuenta que el escalado automático podría no ser el mecanismo más adecuado para controlar una ráfaga súbita de carga de trabajo. Se necesita tiempo para aprovisionar e iniciar nuevas instancias de un servicio o agregar recursos a un sistema. Además, es posible que la máxima demanda se haya superado cuando estos recursos adicionales estén disponibles. En este escenario, es posible que sea mejor limitar el servicio. Para más información, consulte [Patrón Throttling][throttling].
* Por el contrario, si necesita la capacidad para procesar todas las solicitudes cuando el volumen fluctúa rápidamente, y el costo no es un factor de impacto principal, considere la posibilidad de usar una estrategia agresiva de escalado automático en la que las instancias adicionales se inicien con mayor rapidez. También puede usar una directiva programada que inicie un número suficiente de instancias como para satisfacer la carga máxima antes de que se espere esa carga.
* El mecanismo de escalado automático debe supervisar el proceso de escalado automático y registrar los detalles de cada evento de escalado automático (qué lo desencadenó, qué recursos se agregaron o quitaron y cuándo). Si crea un mecanismo de escalado automático personalizado, asegúrese de que incorpore esta capacidad. Analice la información para ayudar a medir la eficacia de la estrategia de escalado automático y ajustarla según sea necesario. Puede ajustarla tanto a corto plazo, a medida que los patrones de uso se hagan más evidentes, y a largo plazo, a medida que el negocio se expanda o evolucionen los requisitos de la aplicación. Si una aplicación alcanza el límite superior definido para el escalado automático, el mecanismo también podría alertar a un operador que pueda iniciar manualmente recursos adicionales en caso necesario. Tenga en cuenta que, en estas circunstancias, el operador también puede ser responsable de quitar manualmente estos recursos una vez que se reduzca la carga de trabajo.

## <a name="related-patterns-and-guidance"></a>Orientación y patrones relacionados
Los siguientes patrones y orientación también pueden ser pertinentes para su escenario al implementar el escalado automático:

* [Patrón Throttling][throttling]. En este patrón se describe cómo una aplicación puede seguir funcionando y cumplir los contratos de nivel de servicio cuando un aumento en la demanda aplica una carga muy elevada en los recursos. La limitación se puede usar con la escalado automático para impedir que un sistema se vea superado durante la operación de escalar horizontalmente.
* [Patrón Competing Consumers][competing-consumers]. En este patrón se describe cómo implementar un grupo de instancias de servicio que puede controlar mensajes de cualquier instancia de la aplicación. El escalado automático se puede usar para iniciar y detener las instancias de servicio para que coincidan con la carga de trabajo anticipada. Este método permite que un sistema procese varios mensajes simultáneamente a fin de optimizar el rendimiento, mejorar la escalabilidad y disponibilidad, y equilibrar la carga de trabajo.
* [Supervisión y diagnóstico](./monitoring.md). La instrumentación y telemetría son vitales para la recopilación de información que puede impulsar el proceso de escalado automático.


<!-- links -->

[monitoring]: /azure/monitoring-and-diagnostics/monitoring-overview-autoscale
[app-service-autoscale]: /azure/monitoring-and-diagnostics/insights-how-to-scale?toc=%2fazure%2fapp-service-web%2ftoc.json#scaling-based-on-a-pre-set-metric
[app-service-plan]: /azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview
[autoscale-metrics]: /azure/monitoring-and-diagnostics/insights-autoscale-common-metrics
[cloud-services-autoscale]: /azure/cloud-services/cloud-services-how-to-scale-portal
[competing-consumers]: ../patterns/competing-consumers.md
[data-partitioning]: ./data-partitioning.md
[functions-scale]: /azure/azure-functions/functions-scale
[link-resource-to-cloud-service]: /azure/cloud-services/cloud-services-how-to-manage#how-to-link-a-resource-to-a-cloud-service
[pipes-and-filters]: ../patterns/pipes-and-filters.md
[service-fabric-autoscale]: /azure/service-fabric/service-fabric-cluster-scale-up-down
[throttling]: ../patterns/throttling.md
[vm-scale-sets]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
[vm-scale-sets-autoscale]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview
