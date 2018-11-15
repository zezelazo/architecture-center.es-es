---
title: Orientación sobre los trabajos de segundo plano
description: Orientación sobre las tareas en segundo plano que se ejecutan independientemente de la interfaz de usuario.
author: dragon119
ms.date: 11/05/2018
ms.openlocfilehash: 0c48121a0d5cff33893a8f242c70f4a275c46f73
ms.sourcegitcommit: d59e2631fb08665bc30f6b65bfc7e1b75935cbd5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/05/2018
ms.locfileid: "51021940"
---
# <a name="background-jobs"></a>Trabajos en segundo plano

Muchos tipos de aplicaciones requieren tareas en segundo plano que se ejecutan independientemente de la interfaz de usuario (UI). Algunos ejemplos incluyen trabajos por lotes, tareas de procesamiento intensivas y procesos de ejecución prolongada, como flujos de trabajo. Los trabajos en segundo plano pueden ejecutarse sin intervención del usuario. La aplicación puede iniciar el trabajo y seguir procesando las solicitudes interactivas de los usuarios. Esto puede ayudar a reducir la carga en la interfaz de usuario de la aplicación, lo que puede mejorar la disponibilidad y reducir los tiempos de respuesta interactiva.

Por ejemplo, si una aplicación necesita generar miniaturas de imágenes cargadas por los usuarios, puede hacerlo como un trabajo en segundo plano y guardar la miniatura en el almacenamiento una vez completado el proceso, sin que el usuario tenga que esperar hasta que el proceso se complete. Del mismo modo, un usuario que haga un pedido puede iniciar un flujo de trabajo en segundo plano que procese el pedido, mientras que la interfaz de usuario permite al usuario seguir explorando la aplicación web. Cuando se complete el trabajo en segundo plano, puede actualizar los datos de pedidos almacenados y enviar un correo electrónico al usuario que confirma el pedido.

Cuando considere si una tarea se debe implementar como un trabajo en segundo plano, el criterio principal es si la tarea se puede ejecutar sin la interacción del usuario y sin que la interfaz de usuario tenga que esperar a que el trabajo se complete. Las tareas que requieran que la interacción del usuario o de la interfaz de usuario espere mientras se completan, podrían no ser adecuadas como trabajos en segundo plano.

## <a name="types-of-background-jobs"></a>Tipos de trabajos en segundo plano
Los trabajos en segundo plano suelen incluir uno o más de los siguientes tipos de trabajos:

* Trabajos de uso intensivo de la CPU, tales como cálculos matemáticos o análisis de modelo estructural.
* Trabajos E/S intensivos, tales como la ejecución de una serie de transacciones de almacenamiento o la indexación de archivos.
* Trabajos por lotes, tales como actualizaciones de datos por la noche o procesamiento programado.
* Flujos de trabajo de ejecución prolongada, tales como la realización de pedidos o el aprovisionamiento de servicios y sistemas.
* Procesamiento de datos confidenciales en el que la tarea se entrega a una ubicación más segura para su procesamiento. Por ejemplo, quizás no quiera procesar datos confidenciales en una aplicación web. En su lugar, puede usar un patrón como [Gatekeeper](https://msdn.microsoft.com/library/dn589793.aspx) para transferir los datos a un proceso en segundo plano aislado que tenga acceso al almacenamiento protegido.

## <a name="triggers"></a>Desencadenadores
Los trabajos en segundo plano se pueden iniciar de varias maneras diferentes. Se dividen en una de las siguientes categorías:

* [**Desencadenadores impulsados por eventos**](#event-driven-triggers). La tarea se inicia en respuesta a un evento, normalmente una acción realizada por un usuario o un paso de un flujo de trabajo.
* [**Desencadenadores impulsados por programación**](#schedule-driven-triggers). La tarea se invoca según una programación basada en un temporizador. Esto podría ser una programación periódica o una invocación única que se especifica para una fecha posterior.

### <a name="event-driven-triggers"></a>Desencadenadores impulsados por eventos
La invocación impulsada por eventos usa un desencadenador para iniciar la tarea en segundo plano. Entre algunos ejemplos del uso de desencadenadores impulsados por eventos se incluyen:

* La interfaz de usuario u otro trabajo coloca un mensaje en una cola. El mensaje contiene datos sobre una acción que se ha producido, por ejemplo, el usuario realiza un pedido. La tarea en segundo plano escucha en esta cola y detecta la llegada de un nuevo mensaje. Lee el mensaje y usa los datos que este contiene como la entrada para el trabajo en segundo plano.
* La interfaz de usuario u otro trabajo guarda o actualiza un valor en el almacenamiento. La tarea en segundo plano supervisa el almacenamiento y detecta los cambios. Lee los datos y los usa como la entrada para el trabajo en segundo plano.
* La interfaz de usuario u otro trabajo realiza una solicitud a un punto de conexión, como un identificador URI HTTPS o una API que está expuesta como servicio web. Pasa los datos necesarios para completar la tarea en segundo plano como parte de la solicitud. El punto de conexión o servicio web invoca la tarea en segundo plano, que usa los datos como entrada.

Entre algunos ejemplos típicos de las tareas adecuadas para la invocación impulsada por eventos se incluyen procesamiento de imágenes, flujos de trabajo, envío de información a servicios remotos, envío de mensajes de correo electrónico y aprovisionamiento de nuevos usuarios en aplicaciones de varios inquilinos.

### <a name="schedule-driven-triggers"></a>Desencadenadores impulsados por programación
La invocación impulsada por programación usa un temporizador para iniciar la tarea en segundo plano. Entre algunos ejemplos del uso de desencadenadores impulsados por programación se incluyen:

* Un temporizador que se ejecuta localmente en la aplicación o como parte del sistema operativo de la aplicación invoca una tarea en segundo plano de forma periódica.
* Un temporizador que se ejecuta en otra aplicación o un servicio de temporizador, tal como Azure Scheduler, envía una solicitud a una API o a un servicio web de forma periódica. La API o servicio web invoca la tarea en segundo plano.
* Un proceso o aplicación independiente inicia un temporizador que provoca la invocación de tarea en segundo plano después de un retraso de tiempo especificado o en un momento determinado.

Entre algunos ejemplos típicos de las tareas adecuadas para la invocación impulsada por programación se incluyen rutinas de procesamiento por lotes (como la actualización de listas de productos relacionados con los usuarios basadas en su comportamiento reciente), tareas rutinarias de procesamiento de datos (como la actualización de índices o la generación de resultados acumulados), análisis de datos para informes diarios, limpieza de retención de datos y comprobaciones de coherencia de datos.

Si usa una tarea impulsada por programación que se debe ejecutar como una sola instancia, tenga en cuenta lo siguiente:

* Si se escala la instancia de proceso que ejecuta el programador (por ejemplo, una máquina virtual que usa tareas programadas de Windows), tendrá varias instancias del programador en ejecución. Estas podrían iniciar varias instancias de la tarea.
* Si las tareas se ejecutan durante más tiempo que el período entre eventos del programador, el programador puede iniciar otra instancia de la tarea mientras se siga ejecutando la anterior.

## <a name="returning-results"></a>Devolución de resultados
Los trabajos en segundo plano se ejecutan de forma asincrónica en un proceso independiente o incluso en una ubicación independiente, desde la interfaz de usuario o desde el proceso que invocó la tarea en segundo plano. Idealmente, las tareas en segundo plano son operaciones tipo "desencadenar y olvidar" y su progreso de ejecución no tiene ningún impacto en la interfaz de usuario o el proceso de llamada. Esto significa que el proceso de llamada no espera a la finalización de las tareas. Por lo tanto, no puede detectar automáticamente cuándo finaliza la tarea.

Si necesita que una tarea de segundo plano comunique con la tarea de llamada para indicar el progreso o la finalización, debe implementar un mecanismo para ello. A continuación, se indican algunos ejemplos:

* Escriba un valor del indicador de estado en el almacenamiento que sea accesible para la tarea de interfaz de usuario o de llamada, que puede supervisar o comprobar este valor cuando sea necesario. Otros datos que la tarea en segundo plano debe devolver a la llamada se pueden colocar en el mismo almacenamiento.
* Establecer una cola de respuesta en la que escucha la interfaz de usuario o la llamada. La tarea en segundo plano puede enviar mensajes a la cola que indica el estado y la finalización. Los datos que la tarea en segundo plano debe devolver a la llamada pueden colocarse en los mensajes. Si usa Azure Service Bus, puede usar las propiedades **ReplyTo** y **CorrelationId** para implementar esta funcionalidad.
* Exponer una API o punto de conexión desde la tarea en segundo plano a la que pueda acceder la interfaz de usuario o la llamada para obtener información sobre el estado. Los datos que la tarea en segundo plano debe devolver a la llamada pueden incluirse en la respuesta.
* Haga que la tarea en segundo plano realice una devolución de llamada a la interfaz de usuario o llamada a través de una API para indicar el estado en puntos predefinidos o en el momento de finalización. Esto podría hacerse a través de eventos generados localmente o a través de un mecanismo de publicación y suscripción. Los datos que debe devolver la tarea en segundo plano a la llamada se pueden incluir en la carga de la solicitud o evento.

## <a name="hosting-environment"></a>Entorno de hospedaje
Puede hospedar tareas en segundo plano usando una variedad de diferentes servicios de la plataforma de Azure:

* [**Azure Web Apps y WebJobs**](#azure-web-apps-and-webjobs). Puede usar WebJobs para ejecutar trabajos personalizados basados en una variedad de distintos tipos de scripts o programas ejecutables en el contexto de una aplicación web.
* [**Azure Virtual Machines**](#azure-virtual-machines). Si tiene un servicio de Windows o quiere usar el Programador de tareas de Windows, es común hospedar las tareas en segundo plano dentro de una máquina virtual dedicada.
* [**Azure Batch**](#azure-batch). Batch es un servicio de plataforma que programa el trabajo que hace un uso intensivo de los recursos de proceso para que se ejecute en una colección administrada de máquinas virtuales. Puede escalar automáticamente los recursos de proceso.
* [**Azure Kubernetes Service**](#azure-kubernetes-service) (AKS). Azure Kubernetes Service proporciona un entorno de hospedaje administrado para Kubernetes en Azure. 

En las siguientes secciones se describe cada una de estas opciones con más detalle y se incluyen consideraciones para ayudarle a elegir la opción adecuada.

### <a name="azure-web-apps-and-webjobs"></a>Azure Web Apps y WebJobs

Puede usar Azure WebJobs para ejecutar trabajos personalizados como tareas en segundo plano dentro de una aplicación web de Azure. WebJobs se ejecuta en el contexto de su aplicación web como un proceso continuo. WebJobs también se ejecuta en respuesta a un evento desencadenador desde Azure Scheduler o factores externos, como cambios en los blobs de almacenamiento y en las colas de mensajes. Los trabajos se pueden iniciar y detener a petición, y cerrarse correctamente. Si se produce un error continuamente al ejecutar un trabajo web, este se reinicia automáticamente. Las acciones de error y reintento son configurables.

Cuando configura un WebJob:

* Si quiere que el trabajo responda a un desencadenador impulsado por eventos, debería configurarlo como **Ejecutar continuamente**. El script o programa se almacena en la carpeta denominada site/wwwroot/app_data/jobs/continuous.
* Si quiere que el trabajo responda a un desencadenador controlado por programación, debería configurarlo como **Ejecutar de acuerdo con una programación**. El script o programa se almacena en la carpeta denominada site/wwwroot/app_data/jobs/triggered.
* Si elige la opción **Ejecutar a petición** al configurar un trabajo, se ejecutará el mismo código que la opción **Ejecutar de acuerdo con una programación** cuando se inicia.

Azure WebJobs se ejecuta en el espacio aislado de la aplicación web. Esto significa que pueden tener acceso a las variables de entorno y compartir información, como cadenas de conexión, con la aplicación web. El trabajo tiene acceso al identificador único de la máquina que está ejecutando el trabajo. La cadena de conexión denominada **AzureWebJobsStorage** proporciona acceso a las colas, los blobs y las tablas de Azure Storage para los datos de la aplicación, y acceso a Service Bus para la comunicación y la mensajería. La cadena de conexión denominada **AzureWebJobsDashboard** proporciona acceso a los archivos de registro de acciones del trabajo.

Azure WebJobs tiene las siguientes características:

* **Seguridad**: los trabajos web se protegen mediante las credenciales de implementación de la aplicación web.
* **Tipos de archivo admitidos**: puede definir WebJobs con scripts de comandos (.cmd), archivos por lotes (.bat), scripts de PowerShell (. ps1), scripts de shell de bash (.sh), scripts de PHP (.php), scripts de Python (.py), código JavaScript (.js) y programas ejecutables (.exe y .jar entre otros).
* **Implementación**: puede implementar los scripts y ejecutables mediante [Azure Portal](/azure/app-service-web/web-sites-create-web-jobs), [Visual Studio](/azure/app-service-web/websites-dotnet-deploy-webjobs) o [Azure WebJobs SDK](/azure/app-service/webjobs-sdk-get-started), o copiándolos directamente en las siguientes ubicaciones:
  * Para la ejecución desencadenada: site/wwwroot/app_data/jobs/triggered/{nombre del trabajo}
  * Para la ejecución continua: site/wwwroot/app_data/jobs/continuous/{nombre del trabajo}
* **Registro**: Console.Out se considera (marca) como INFO. Console.Error se trata como ERROR. Puede tener acceso a la información de diagnóstico y supervisión mediante el Portal de Azure. Puede descargar los archivos de registro directamente del sitio. Se guardan en las siguientes ubicaciones:
  * Para la ejecución desencadenada: Vfs/data/jobs/triggered/jobName
  * Para la ejecución continua: Vfs/data/jobs/continuous/jobName
* **Configuración**: puede configurar WebJobs mediante el portal, la API de REST y PowerShell. Puede usar un archivo de configuración denominado settings.job en el mismo directorio raíz que el script de trabajo para proporcionar la información de configuración de un trabajo. Por ejemplo: 
  * { "stopping_wait_time": 60 }
  * { "is_singleton": true }

#### <a name="considerations"></a>Consideraciones

* De forma predeterminada, los trabajos web se escalan con la aplicación web. Sin embargo, puede configurar los trabajos para que se ejecuten en una instancia única; para ello, establezca la propiedad de configuración **is_singleton** en **true**. Los WebJobs de instancia única son útiles para las tareas que no quiera escalar o ejecutar como varias instancias simultáneas, por ejemplo, la reindexación, el análisis de datos y tareas similares.
* Para minimizar el impacto de los trabajos en el rendimiento de la aplicación web, considere la posibilidad de crear una instancia vacía de aplicación web de Azure en un nuevo plan de App Service para hospedar WebJobs que pueden tardar tiempo en ejecutarse o que consumen muchos recursos.

### <a name="azure-virtual-machines"></a>Azure Virtual Machines
Las tareas en segundo plano se pueden implementar de forma tal que se evite su implementación en Azure Web Apps o puede que estas opciones no sean prácticas. Entre algunos ejemplos típicos se incluyen los servicios de Windows y las utilidades y programas ejecutables de terceros. Otro ejemplo podrían ser programas escritos para un entorno de ejecución diferente del que hospeda la aplicación. Por ejemplo, podría ser un programa de Unix o Linux que quiere ejecutar desde una aplicación de Windows o .NET. Puede elegir entre una variedad de sistemas operativos para una máquina virtual de Azure y ejecutar el servicio o ejecutable en esa máquina virtual.

Para ayudarle a elegir cuándo usar Virtual Machines, consulte [Comparación de Azure App Services, Cloud Services y Virtual Machines](/azure/app-service-web/choose-web-site-cloud-service-vm/). Para más información sobre las opciones de las máquinas virtuales, vea [Tamaños de las máquinas virtuales Windows en Azure](/azure/virtual-machines/windows/sizes). Para más información sobre los sistemas operativos y las imágenes preconfiguradas que están disponibles para las máquinas virtuales, consulte [Marketplace de Azure Virtual Machines](https://azure.microsoft.com/gallery/virtual-machines/).

Para iniciar la tarea en segundo plano en una máquina virtual independiente, tiene varias opciones:

* Puede ejecutar la tarea a petición directamente desde la aplicación mediante el envío de una solicitud a un punto de conexión que exponga la tarea. Esto pasa en cualquier dato que requiera la tarea. Este punto de conexión invoca la tarea.
* Puede configurar la tarea para que se ejecute según una programación con un programador o temporizador que esté disponible en el sistema operativo elegido. Por ejemplo, en Windows puede usar el Programador de tareas de Windows para ejecutar scripts y tareas. O bien, si tiene SQL Server instalado en la máquina virtual, puede usar el Agente SQL Server para ejecutar scripts y tareas.
* Puede usar Azure Scheduler para iniciar la tarea al agregar un mensaje a una cola en la que escucha la tarea o al enviar una solicitud a una API que la tarea expone.

Consulte la sección anterior [Desencadenadores](#triggers) para más información sobre cómo iniciar tareas en segundo plano.  

#### <a name="considerations"></a>Consideraciones
Tenga en cuenta los siguientes puntos cuando decida si va a implementar tareas en segundo plano en una máquina virtual de Azure:

* El hospedaje de tareas en segundo plano en una máquina virtual de Azure diferente proporciona flexibilidad y permite un control preciso sobre la iniciación, ejecución, programación y asignación de recursos. Sin embargo, aumentará el costo de tiempo de ejecución si una máquina virtual se debe implementar solo para ejecutar tareas en segundo plano.
* No existe ninguna utilidad que permita supervisar las tareas en Azure Portal ni ninguna funcionalidad de reinicio automatizado para las tareas con error; sin embargo, puede supervisar el estado básico de la máquina virtual y administrarla con los [cmdlets de Azure Service Management](https://msdn.microsoft.com/library/mt125356.aspx). No obstante, no hay funcionalidad para controlar los procesos y subprocesos en los nodos de proceso. Normalmente, el uso de una máquina virtual requiere un esfuerzo adicional para implementar un mecanismo que recopile datos de instrumentación en la tarea y del sistema operativo en la máquina virtual. Una solución que podría ser adecuada es usar el [paquete de administración de System Center para Azure](https://www.microsoft.com/download/details.aspx?id=50013).
* Considere la posibilidad de crear sondeos de supervisión que se exponen a través de puntos de conexión HTTP. El código para estos sondeos podría realizar comprobaciones de estado, recopilar información operativa y estadísticas o intercalar información de error y devolverla a una aplicación de administración. Para más información, consulte el artículo sobre el [patrón de supervisión del extremo de estado](../patterns/health-endpoint-monitoring.md).

Para más información, consulte:

* [Máquinas virtuales](https://azure.microsoft.com/services/virtual-machines/)
* [Preguntas más frecuentes sobre Azure Virtual Machines](/azure/virtual-machines/virtual-machines-linux-classic-faq?toc=%2fazure%2fvirtual-machines%2flinux%2fclassic%2ftoc.json)

### <a name="azure-batch"></a>Azure Batch 

Considere la posibilidad de usar [Azure Batch](/azure/batch/) si debe ejecutar cargas de trabajo de informática de alto rendimiento (HPC) de gran tamaño y en paralelo, en decenas, cientos o miles de máquinas virtuales.  

El servicio Batch aprovisiona las máquinas virtuales, asigna las tareas a las máquinas virtuales, ejecuta las tareas y supervisa el progreso. Batch puede escalar horizontalmente las máquinas virtuales de forma automática en respuesta a la carga de trabajo. Batch también proporciona la programación de trabajos. Azure Batch admite máquinas virtuales Windows y Linux.

#### <a name="considerations"></a>Consideraciones 

Batch funciona bien con cargas de trabajo intrínsecamente paralelas. También puede realizar cálculos en paralelo con un paso reducido al final, o ejecutar [aplicaciones de Message Passing Interface (MPI)](/azure/batch/batch-mpi) para las tareas en paralelo que requieren el paso de mensajes entre los nodos. 

Un trabajo de Azure Batch se ejecuta en un grupo de nodos (máquinas virtuales). Un enfoque es asignar un grupo solo cuando sea necesario y eliminarlo una vez completado el trabajo. Esto permite maximizar el uso porque los nodos no están inactivos, pero el trabajo debe esperar a que se asignen los nodos. También puede crear un grupo de antemano. Esto permite minimizar el tiempo que un trabajo tarda en iniciarse, pero el resultado puede ser tener nodos inactivos. Para más información, consulte [Vigencia de grupo y nodo de proceso](/azure/batch/batch-api-basics#pool-and-compute-node-lifetime).

Para más información, consulte:

* [¿Qué es Azure Batch?](/azure/batch/batch-technical-overview) 
* [Desarrollo de soluciones de procesos paralelos a gran escala con Batch](/azure/batch/batch-api-basics) 
* [Soluciones de Batch y HPC para cargas de trabajo de procesos a gran escala](/azure/batch/batch-hpc-solutions)

### <a name="azure-kubernetes-service"></a>Azure Kubernetes Service

Azure Kubernetes Service (AKS) administra su entorno hospedado de Kubernetes, lo que facilita la implementación y administración de aplicaciones en contenedores. 

Los contenedores pueden ser útiles para ejecutar trabajos en segundo plano. Estas son algunas de las ventajas: 

- Los contenedores admiten el hospedaje de alta densidad. Puede aislar una tarea en segundo plano en un contenedor y colocar varios contenedores en cada máquina virtual.
- El orquestador de contenedores controla el equilibrio de carga interno, configura la red interna y realiza otras tareas de configuración.
- Los contenedores se pueden iniciar y detener cuando sea necesario. 
- Azure Container Registry permite registrar los contenedores dentro de los límites de Azure. Esto aporta ventajas en cuanto a seguridad, privacidad y proximidad. 

#### <a name="considerations"></a>Consideraciones

- Requiere conocer cómo se usa un orquestador de contenedores. Según las habilidades de su equipo de DevOps, esto puede ser un problema o no.

Para más información, consulte:

* [Introducción a los contenedores en Azure](https://azure.microsoft.com/overview/containers/) 
* [Introducción a los registros de contenedores privados de Docker](/azure/container-registry/container-registry-intro) 

## <a name="partitioning"></a>Creación de particiones
Si decide incluir tareas en segundo plano en una instancia de proceso existente, debe tener en cuenta cómo afectará a los atributos de calidad de la instancia de proceso y la tarea de segundo plano propiamente dicha. Estos factores le ayudarán a decidir si las tareas se deben colocalizar con la instancia de proceso existente o si se deben separar en una instancia de proceso independiente:

* **Disponibilidad**: es posible que las tareas en segundo plano no necesiten tener el mismo nivel de disponibilidad que otras partes de la aplicación, en concreto, la interfaz de usuario y otras partes que están implicadas directamente en la interacción con el usuario. Las tareas en segundo plano podrían ser más tolerantes a la latencia, a los errores de reintento de conexión y a otros factores que afectan a la disponibilidad, ya que las operaciones se pueden poner en cola. Sin embargo, debe haber capacidad suficiente como para evitar la copia de seguridad de las solicitudes que podrían bloquear las colas y afectar a la aplicación en su totalidad.
* **Escalabilidad**: es probable que las tareas en segundo plano tengan un requisito de escalabilidad diferente al de la interfaz de usuario y las partes interactivas de la aplicación. El escalado de la interfaz de usuario podría ser necesario para satisfacer los picos de demanda, mientras que las tareas en segundo plano pendientes podrían completarse en horarios de menor actividad usando un menor número de instancias de proceso.
* **Resistencia**: el error de una instancia de proceso que hospeda solo tareas en segundo plano podría no afectar gravemente a la aplicación en su totalidad si las solicitudes para estas tareas se pueden poner en cola o posponer hasta que la tarea esté disponible de nuevo. Si la instancia de proceso o las tareas se pueden reiniciar dentro de un intervalo adecuado, es posible que los usuarios de la aplicación no se vean afectados.
* **Seguridad**: las tareas en segundo plano podrían tener requisitos o restricciones de seguridad distintos de los de la interfaz de usuario u otras partes de la aplicación. Al usar una instancia de proceso independiente, puede especificar un entorno de seguridad diferente para las tareas. También puede usar patrones, como Gatekeeper, para aislar las instancias de proceso de segundo plano de la interfaz de usuario con el fin de maximizar la seguridad y la separación.
* **Rendimiento**: puede elegir el tipo de instancia de proceso para las tareas de segundo plano de modo que satisfagan específicamente los requisitos de rendimiento de las tareas. Esto podría significar el uso de una opción de proceso menos costosa si las tareas no requieren las mismas funcionalidades de procesamiento que las de la interfaz de usuario, o una instancia mayor si requieren recursos y funcionalidades adicionales.
* **Manejabilidad**: las tareas en segundo plano podrían tener un ritmo de desarrollo e implementación diferente del código de aplicación principal o la interfaz de usuario. Su implementación en una instancia de proceso independiente puede simplificar las actualizaciones y el control de versiones.
* **Costo**: la adición de instancias de proceso para ejecutar tareas en segundo plano aumenta los costos de hospedaje. Debe considerar detenidamente el equilibrio entre la capacidad adicional y estos costos adicionales.

Para más información, consulte [Leader Election Pattern](../patterns/leader-election.md) (Patrón de elección de líder) y [Competing Consumers Pattern](../patterns/competing-consumers.md) (Patrón de consumidores en competencia).

## <a name="conflicts"></a>Conflictos
Si tiene varias instancias de un trabajo en segundo plano, es posible que compitan entre sí para tener acceso a los recursos y servicios, como las bases de datos y el almacenamiento. Este acceso simultáneo puede producir contención de recursos, que podría causar conflictos en la disponibilidad de los servicios y en la integridad de los datos en el almacenamiento. Puede resolver la contención de recursos usando un modo de bloqueo pesimista. Esto evita que las instancias de una tarea compitan por el acceso a un servicio o a la corrupción de datos de manera simultánea.

Otro método para resolver conflictos es definir tareas en segundo plano como un singleton, de modo que solo habrá una instancia en ejecución a la vez. Sin embargo, esto elimina las ventajas de confiabilidad y rendimiento que puede proporcionar una configuración de varias instancias. Esto sucede especialmente si la interfaz de usuario puede proporcionar suficiente trabajo para mantener ocupada más de una tarea en segundo plano.

Es fundamental asegurarse de que la tarea en segundo plano pueda reiniciarse automáticamente y que tenga suficiente capacidad como para hacer frente a los picos de demanda. Puede lograrlo mediante la asignación de una instancia de proceso con suficientes recursos, la implementación de un mecanismo de puesta en cola que pueda almacenar las solicitudes de ejecución posterior cuando disminuya la demanda, o usando una combinación de estas técnicas.

## <a name="coordination"></a>Coordinación
Las tareas en segundo plano podrían ser complejas y podrían requerir la ejecución de varias tareas individuales para generar un resultado o satisfacer todos los requisitos. Es habitual en estos escenarios dividir la tarea en pasos o subtareas discretos más pequeños que varios consumidores pueden ejecutar. Los trabajos de varios pasos pueden ser más eficaces y más flexibles porque los pasos individuales podrían volver a usarse en varios trabajos. También facilitan la adición, eliminación o modificación del orden de los pasos.

La coordinación de varias tareas y pasos puede suponer un reto, pero hay tres patrones comunes que puede usar y que le guiarán por el proceso de implementación de una solución:

* **Descomponer una tarea en varios pasos reutilizables**. Es posible que una aplicación tenga que realizar diversas tareas de complejidad variable en la información que procesa. Un método sencillo pero inflexible para implementar esta aplicación podría ser realizar este procesamiento como un módulo monolítico. Sin embargo, este método probablemente reducirá las posibilidades de refactorización del código, su optimización o su reutilización si partes del mismo procesamiento se necesitan en otro lugar dentro de la aplicación. Para más información, consulte el artículo sobre el [patrón de canalizaciones y filtros](../patterns/pipes-and-filters.md).
* **Administrar la ejecución de los pasos de una tarea**. Una aplicación podría realizar tareas que forman una serie de pasos (algunos de los cuales podrían invocar servicios remotos o acceder a recursos remotos). Los pasos individuales podrían ser independientes entre sí, pero están organizados por la lógica de la aplicación que implementa la tarea. Para más información, consulte el artículo sobre el [patrón de supervisor del agente de Scheduler](../patterns/scheduler-agent-supervisor.md).
* **Administrar la recuperación de los pasos de una tarea con error**. Una aplicación podría necesitar deshacer el trabajo que se realiza por una serie de pasos (que conjuntamente definen una operación coherente) si uno o varios de los pasos termina en error. Para más información, consulte el artículo sobre el [patrón de transacción de compensación](../patterns/compensating-transaction.md).


## <a name="resiliency-considerations"></a>Consideraciones de resistencia
Las tareas en segundo plano deben ser resistentes para proporcionar servicios confiables a la aplicación. Cuando esté planeando y diseñando tareas en segundo plano, tenga en cuenta los siguientes puntos:

* Las tareas en segundo plano deben poder controlar correctamente los reinicios sin dañar los datos ni introducir incoherencias en la aplicación. Para las tareas de ejecución prolongada o con varios pasos, considere el uso de *puntos de comprobación* guardando el estado de los trabajos en el almacenamiento persistente o como mensajes en una cola, si fuera apropiado. Por ejemplo, puede conservar la información de estado de un mensaje en una cola y actualizar esta información de estado en incrementos con el progreso de la tarea para que esta se pueda procesar desde el último punto de comprobación correcto conocido, en lugar de reiniciar desde el principio. Al usar las colas de Azure Service Bus, puede usar sesiones de mensajes para habilitar el mismo escenario. Las sesiones le permiten guardar y recuperar el estado de procesamiento de la aplicación mediante los métodos [SetState](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.messagesession.setstate?view=azureservicebus-4.0.0) y [GetState](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.messagesession.getstate?view=azureservicebus-4.0.0). Para más información sobre el diseño de procesos y flujos de trabajo confiables de varios pasos, consulte [Scheduler Agent Supervisor Pattern](../patterns/scheduler-agent-supervisor.md)(Patrón de supervisor de agente de programador).
* Cuando use las colas para comunicarse con las tareas en segundo plano, las colas pueden actuar como un búfer para almacenar las solicitudes que se envían a las tareas cuando la aplicación se encuentra con una mayor carga que la habitual. Esto permite que las tareas se pongan al día con la interfaz de usuario durante los períodos de menor actividad. También significa que los reinicios no bloquearán la interfaz de usuario. Para más información, consulte [Patrón de equilibrio de carga basado en colas](../patterns/queue-based-load-leveling.md). Si algunas tareas son más importantes que otras, considere la posibilidad de implementar el [patrón de cola de prioridad](../patterns/priority-queue.md) para asegurarse de que estas tareas se ejecuten antes que las tareas menos importantes.
* Las tareas en segundo plano que procesan los mensajes, o que estos inician, se deben diseñar para controlar las incoherencias, como los mensajes que llegan sin orden, los mensajes que generan un error de forma repetida (con frecuencia denominados *mensajes dudosos*) y los mensajes que se entregan más de una vez. Tenga en cuenta lo siguiente.
  * Los mensajes que se deben procesar en un orden específico, tales como los que cambian los datos según su valor de datos existente (por ejemplo, al agregar un valor a un valor existente), podrían no llegar en el orden original en el que se enviaron. Como alternativa, los podrían controlar instancias diferentes de una tarea en segundo plano en un orden diferente debido a cargas variables en cada instancia. Los mensajes que se deben procesar en un orden específico deben incluir un número de secuencia, clave o algún otro indicador que las tareas en segundo plano pueden usar para garantizar que se procesan en el orden correcto. Si usa Azure Service Bus, puede usar sesiones de mensajes para garantizar el orden de entrega. Sin embargo, suele ser más eficaz cuando sea posible diseñar el proceso de modo que el orden de los mensajes no sea importante.
  * Normalmente, una tarea en segundo plano inspeccionará los mensajes de la cola, que los oculta temporalmente de los demás consumidores de mensajes. A continuación, elimina los mensajes una vez que se hayan procesado correctamente. Si se produce un error en una tarea en segundo plano al procesar un mensaje, ese mensaje volverá a aparecer en la cola después de que expire el tiempo de espera de la inspección. Se procesará por otra instancia de la tarea o durante el próximo ciclo de procesamiento de esta instancia. Si el mensaje produce un error sistemáticamente en el consumidor, bloqueará la tarea, la cola y, en última instancia, la aplicación en sí cuando se llene la cola. Por lo tanto, es fundamental detectar y quitar los mensajes dudosos de la cola. Usa Azure Service Bus, los mensajes que produzcan un error se pueden mover automática o manualmente a una cola de mensajes fallidos asociada.
  * Las colas son mecanismos de *al menos una* entrega garantizada, pero podrían entregar el mismo mensaje más de una vez. Además, si se produce un error en una tarea en segundo plano después de procesar un mensaje pero antes de su eliminación de la cola, el mensaje estará disponible para nuevo procesamiento. Las tareas en segundo plano deben ser idempotentes, lo que significa que el procesamiento del mismo mensaje varias veces no produce un error o incoherencia en los datos de la aplicación. Algunas operaciones son naturalmente idempotentes, tal como establecer un valor almacenado en un nuevo valor específico. Sin embargo, las operaciones como, por ejemplo, agregar un valor a un valor almacenado existente sin comprobar que el valor almacenado sigue siendo el mismo que cuando se envió originalmente el mensaje, provocarán incoherencias. Las colas de Azure Service Bus pueden configurarse para quitar automáticamente los mensajes duplicados.
  * Algunos sistemas de mensajería, como las colas de Azure Storage y las colas de Azure Service Bus, admiten una propiedad de recuento de eliminación de la cola que indica el número de veces que se ha leído un mensaje de la cola. Esto puede ser de utilidad en el tratamiento de mensajes dudosos y repetidos. Para más información, consulte los artículos sobre el [manual de mensajería asincrónica](https://msdn.microsoft.com/library/dn589781.aspx) y [patrones de idempotencia](https://blog.jonathanoliver.com/idempotency-patterns/).

## <a name="scaling-and-performance-considerations"></a>Consideraciones de escalado y rendimiento
Las tareas en segundo plano deben ofrecer un rendimiento suficiente como para asegurarse de que no bloqueen la aplicación ni provoquen incoherencias debido al funcionamiento diferido cuando el sistema está bajo carga. Normalmente, el rendimiento se mejora al escalar las instancias de proceso que hospedan las tareas en segundo plano. Cuando esté planeando y diseñando tareas en segundo plano, tenga en cuenta los siguientes puntos en relación con la escalabilidad y el rendimiento:

* Azure admite el escalado automático (escalar horizontalmente y reducir horizontalmente) en función de la demanda y carga actuales, o según una programación predefinida, para implementaciones hospedadas de Web Apps y Virtual Machines. Use esta característica para garantizar que la aplicación en su conjunto tiene funcionalidades de rendimiento suficientes a la vez que se minimizan los costos de tiempo de ejecución.
* Si las tareas en segundo plano tienen una funcionalidad de rendimiento diferente de las demás partes de una aplicación (por ejemplo, la interfaz de usuario o componentes como la capa de acceso a datos), el hospedaje de todas las tareas en segundo plano en un servicio de proceso independiente permite que la interfaz de usuario y las tareas en segundo plano se escalen independientemente para administrar la carga. Si varias tareas en segundo plano tienen funcionalidades de rendimiento muy diferentes entre sí, considere la posibilidad de dividirlas y escalar cada tipo de forma independiente. Sin embargo, tenga en cuenta que esto puede aumentar los costos del runtime.
* Es posible que el simple escalado de los recursos de proceso no sea suficiente para impedir la pérdida de rendimiento bajo carga. También es posible que necesite escalar las colas de almacenamiento y otros recursos para impedir que un único punto de la cadena de procesamiento general se convierta en un cuello de botella. Asimismo, considere otras limitaciones, como por ejemplo, el rendimiento máximo de almacenamiento y otros servicios de los que dependen la aplicación y las tareas en segundo plano.
* Las tareas en segundo plano deben diseñarse para el escalado. Por ejemplo, debe poder detectar dinámicamente el número de colas de almacenamiento en uso para escuchar en la cola adecuada o enviar mensajes a ella.
* De manera predeterminada, los trabajos web se escalan con su instancia asociada de Azure Web Apps. Sin embargo, si quiere que un trabajo web se ejecute solo como una única instancia, puede crear un archivo Settings.job que contenga los datos JSON **{ "is_singleton": true }**. Esto obliga a Azure a ejecutar solo una instancia del WebJob, incluso si hay varias instancias de la aplicación web asociada. Esto puede ser una técnica útil para los trabajos programados que deben ejecutarse como una única instancia.

## <a name="related-patterns"></a>Patrones relacionados
* [Patrón de transacción de compensación](../patterns/compensating-transaction.md)
* [Patrón de consumidores de la competencia](../patterns/competing-consumers.md)
* [Orientación sobre la creación de particiones de proceso](https://msdn.microsoft.com/library/dn589773.aspx)
* [Patrón de Gatekeeper](../patterns/gatekeeper.md)
* [Patrón de elección de líder](../patterns/leader-election.md)
* [Patrón de canalizaciones y filtros](../patterns/pipes-and-filters.md)
* [Patrón de cola de prioridad](../patterns/priority-queue.md)
* [Patrón de equilibrio de carga basado en colas](../patterns/queue-based-load-leveling.md)
* [Patrón de supervisor de agente de programador](../patterns/scheduler-agent-supervisor.md)

