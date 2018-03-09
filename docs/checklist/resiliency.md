---
title: "Lista de comprobación de resistencia"
description: "Lista de comprobación que ofrece una guía para las preocupaciones de resistencia durante el diseño."
author: petertaylor9999
ms.date: 01/10/2018
ms.custom: resiliency, checklist
ms.openlocfilehash: ca4bf77c9348f6c656348d9cd61d3a1241d69ba8
ms.sourcegitcommit: 2123c25b1a0b5501ff1887f98030787191cf6994
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/08/2018
---
# <a name="resiliency-checklist"></a>Lista de comprobación de resistencia

La resistencia es la capacidad de un sistema para recuperarse de errores y seguir en funcionamiento y es uno de los [pilares de la calidad del software](../guide/pillars.md). Diseñar la aplicación para la resistencia requiere planear una variedad de modos de error que podrían ocurrir, y mitigarlos. Use esta lista de comprobación para revisar la arquitectura de la aplicación desde un punto de vista de la resistencia. Revise también la [Lista de comprobación de resistencia para servicios de Azure específicos](./resiliency-per-service.md).

## <a name="requirements"></a>Requisitos

**Defina los requisitos de disponibilidad del cliente.** El cliente tendrá requisitos de disponibilidad para los componentes de la aplicación y esto afectará su diseño. Acuerde con el cliente los objetivos de disponibilidad de cada parte de la aplicación; de lo contrario, es posible que el diseño no cumpla con las expectativas del cliente. Para más información, consulte [Definición de los requisitos de resistencia](../resiliency/index.md#defining-your-resiliency-requirements).

## <a name="application-design"></a>Diseño de aplicaciones

**Realice un análisis del modo de error para la aplicación.** El análisis del modo de error es un proceso para crear resistencia en una aplicación al principio de la etapa de diseño. Para más información, consulte [Failure mode analysis][fma] (Análisis del modo de error). Los objetivos de un análisis del modo de error incluyen:  

* Identificar los tipos de errores que podría experimentar una aplicación.
* Capturar los posibles efectos y el impacto de cada tipo de error en la aplicación.
* Identificar las estrategias de recuperación.
  

**Implemente varias instancias de servicios.** Si la aplicación depende de una única instancia de un servicio, crea un único punto de error. El aprovisionamiento de varias instancias mejora tanto la resistencia como la escalabilidad. Para [Azure App Service](/azure/app-service/app-service-value-prop-what-is/), seleccione un [Plan de App Service](/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview/) que ofrezca varias instancias. Para Azure Cloud Services, configure cada uno de los roles para utilizar [varias instancias](/azure/cloud-services/cloud-services-choose-me/#scaling-and-management). Para [Azure Virtual Machines](/azure/virtual-machines/virtual-machines-windows-about/?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json), asegúrese de que la arquitectura de máquina virtual incluya más de una máquina virtual y que cada una de ellas se incluya en un [conjunto de disponibilidad][availability-sets].   

**Use el escalado automático para responder a los aumentos de carga.** Si su aplicación no está configurada para escalar horizontalmente de manera automática a medida que aumenta la carga, es posible que se produzca un error en los servicios de la aplicación si se saturan con las solicitudes de los usuarios. Para más información, consulte lo siguiente:

* General: [Lista de comprobación de escalabilidad](./scalability.md)
* Azure App Service: [Escalación del recuento de instancias de forma manual o automática][app-service-autoscale]
* Cloud Services: [Procedimiento para configurar el escalado automático para un servicio en la nube en el Portal][cloud-service-autoscale]
* Virtual Machines: [Introducción a los registros de escalado automático con conjuntos de escalado de máquinas virtuales de Azure][vmss-autoscale]

**Use el equilibrio de carga para distribuir las solicitudes.** El equilibrio de carga distribuye las solicitudes de la aplicación a las instancias de servicio correctas mediante la eliminación de la rotación de las instancias no correctas. Si el servicio utiliza Azure App Service o Azure Cloud Services, ya tiene la carga equilibrada. Sin embargo, si la aplicación usa Azure Virtual Machines, debe aprovisionar un equilibrador de carga. Para más información, consulte la introducción a [Azure Load Balancer](/azure/load-balancer/load-balancer-overview/).

**Configure instancias de Azure Application Gateway para usar varias instancias.** Dependiendo de los requisitos de la aplicación, una instancia de [Azure Application Gateway](/azure/application-gateway/application-gateway-introduction/) puede ser más adecuada para distribuir solicitudes a los servicios de la aplicación. Sin embargo, las instancias individuales del servicio Application Gateway no están garantizadas por un Acuerdo de Nivel de Servicio, por lo que es posible que la aplicación pueda producir un error si la instancia Application Gateway también produce un error. Proporcione más de una instancia de Application Gateway mediana o más grande para garantizar la disponibilidad del servicio bajo los términos del [Acuerdo de Nivel de Servicio](https://azure.microsoft.com/support/legal/sla/application-gateway/v1_0/).

**Use los conjuntos de disponibilidad para cada capa de aplicación.** Al colocar las instancias en un [conjunto de disponibilidad][availability-sets], se proporciona un mayor [Acuerdo de Nivel de Servicio](https://azure.microsoft.com/support/legal/sla/virtual-machines/). 

**Considere la posibilidad de implementar la aplicación en varias regiones.** Si su aplicación se implementa en una sola región, en el caso excepcional de que toda la región no esté disponible, la aplicación tampoco estará disponible. Esto puede ser inaceptable bajo los términos del Acuerdo de Nivel de Servicio de la aplicación. Si ese fuera el caso, considere la posibilidad de implementar la aplicación en varias regiones. Una implementación en varias regiones puede utilizar un modelo activo-activo (que distribuye las solicitudes en varias instancias activas) o un modelo activo-pasivo (que mantiene una instancia "activa" en reserva, en caso de que la instancia principal produzca un error). Le recomendamos que implemente varias instancias de los servicios de la aplicación en pares de regiones. Para más información, consulte [Continuidad empresarial y recuperación ante desastres (BCDR): regiones emparejadas de Azure](/azure/best-practices-availability-paired-regions).

**Utilice Azure Traffic Manager para enrutar el tráfico de la aplicación a diferentes regiones.**  [Azure Traffic Manager][traffic-manager] realiza el equilibrio de carga a nivel de DNS y enrutará el tráfico a regiones diferentes en función del método de [enrutamiento del tráfico][traffic-manager-routing] que se especifique y del estado de los puntos de conexión de la aplicación. Sin Traffic Manager, el usuario está limitado a una única región para la implementación, lo que limita la escala, aumenta la latencia para algunos usuarios y causa tiempo de inactividad de la aplicación en caso de una interrupción del servicio a nivel regional.

**Configure y pruebe el sondeo de estado para los equilibradores de carga y administradores de tráfico.** Asegúrese de que la lógica de estado compruebe las partes críticas del sistema y responda adecuadamente a los sondeos de estado.

* Los sondeos de estado para [Azure Traffic Manager][traffic-manager] y [Azure Load Balancer][load-balancer] cumplen una función específica. Para Traffic Manager, el sondeo de estado determina si se debe conmutar por error a otra región. Para un equilibrador de carga, determina si se debe quitar una máquina virtual de la rotación.      
* Para un sondeo de Traffic Manager, el punto de conexión de estado debe comprobar las dependencias críticas que se implementan en la misma región, y cuyo error debería desencadenar una conmutación por error a otra región.  
* Para un equilibrador de carga, el punto de conexión de estado debe informar del estado de la máquina virtual. No se incluyen otros niveles o servicios externos. De lo contrario, un error que se produzca fuera de la máquina virtual hará que el equilibrador de carga quite la máquina virtual de la rotación.
* Para obtener una guía sobre la implementación de la supervisión del estado en la aplicación, consulte [Health Endpoint Monitoring Pattern](https://msdn.microsoft.com/library/dn589789.aspx) (Patrón de supervisión de puntos de conexión de estado).

**Supervise los servicios de terceros.** Si la aplicación tiene dependencias en servicios de terceros, identifique dónde y cómo se produce un error en estos servicios de terceros y qué efecto tendrán esos errores en la aplicación. Un servicio de terceros puede no incluir supervisión y diagnóstico, por lo que es importante registrar sus invocaciones y correlacionarlas con el estado de la aplicación y el registro de diagnóstico mediante un identificador único. Para más información sobre las prácticas demostradas para la supervisión y diagnóstico, consulte la [Guía de supervisión y diagnóstico][monitoring-and-diagnostics-guidance].

**Asegúrese de que cualquier servicio de terceros que consuma proporciona un Acuerdo de Nivel de Servicio.** Si la aplicación depende de un servicio de terceros, pero el tercero no ofrece ninguna garantía de disponibilidad en forma de Acuerdo de Nivel de Servicio, tampoco se puede garantizar la disponibilidad de su aplicación. El Acuerdo de Nivel de Servicio es tan bueno como el componente menos disponible de la aplicación.

**Implemente modelos de resistencia para operaciones remotas cuando sea apropiado.** Si la aplicación depende de la comunicación entre servicios remotos, siga los modelos de diseño para tratar errores transitorios, como [patrón Retry][retry-pattern] y [patrón Circuit Breaker][circuit-breaker]. Para más información, consulte [Estrategias de resistencia](../resiliency/index.md#resiliency-strategies).

**Implemente operaciones asincrónicas siempre que sea posible.** Las operaciones sincrónicas pueden monopolizar los recursos y bloquear otras operaciones mientras la persona que llama espera que se complete el proceso. Diseñe cada parte de la aplicación para permitir operaciones asincrónicas, siempre que sea posible. Para más información sobre cómo implementar la programación asincrónica en C#, consulte el artículo sobre [programación asincrónica con async y await][asynchronous-c-sharp].

## <a name="data-management"></a>Administración de datos

**Comprenda los métodos de replicación para los orígenes de datos de la aplicación.** Los datos de aplicación se almacenarán en orígenes de datos diferentes y tendrán distintos requisitos de disponibilidad. Evalúe los métodos de replicación para cada tipo de almacenamiento de datos en Azure, incluidos la [replicación de Azure Storage](/azure/storage/storage-redundancy/) y la [replicación geográfica activa de SQL Database](/azure/sql-database/sql-database-geo-replication-overview/) para asegurarse de que se cumplen los requisitos de datos de la aplicación.

**Asegúrese de que ninguna cuenta de usuario individual tenga acceso a los datos de producción y de copia de seguridad.** Las copias de seguridad de datos estarán expuestas a riesgos si una sola cuenta de usuario tiene permiso para escribir tanto en los orígenes de producción y como en los de copia de seguridad. Un usuario malintencionado puede eliminar intencionadamente todos los datos, mientras que un usuario normal puede eliminarlos accidentalmente. Diseñe la aplicación para limitar los permisos de cada cuenta de usuario de modo que solo los usuarios que requieren acceso de escritura tengan acceso de escritura y solo sea a producción o a copia de seguridad, pero no a ambas.

**Documente la conmutación por error del origen de datos y el proceso de conmutación por recuperación y pruébelo.** En el caso de que se produzca un error catastrófico en el origen de datos, el operador humano tendrá que seguir un conjunto de instrucciones documentadas para conmutar por error a un nuevo origen de datos. Si los pasos documentados tienen errores, el operador no podrá seguirlos correctamente y conmutará por error el recurso. Pruebe con regularidad los pasos de instrucciones para comprobar que el operador que los sigue puede realizar correctamente la conmutación por error y la conmutación por recuperación del origen de datos.

**Valide las copias de seguridad de datos.** Compruebe regularmente que sus datos de copia de seguridad son lo que espera mediante la ejecución de un script para validar la integridad de datos, el esquema y las consultas. No tiene sentido tener una copia de seguridad si no es útil para restaurar los orígenes de datos. Registre e informe de cualquier inconsistencia para que el servicio de copia de seguridad se pueda reparar.

**Considere la posibilidad de utilizar un tipo de cuenta de almacenamiento con redundancia geográfica.** Los datos almacenados en una cuenta de Azure Storage siempre se replican de forma local. Sin embargo, hay varias estrategias de replicación para elegir cuando se aprovisiona una cuenta de almacenamiento. Seleccione [Almacenamiento con redundancia geográfica con acceso de lectura (RA-GRS)](/azure/storage/storage-redundancy/#read-access-geo-redundant-storage) para proteger los datos de la aplicación en el raro caso en que una región entera no esté disponible.

> [!NOTE]
> En el caso de las máquinas virtuales, no confíe en la replicación RA-GRS para restaurar los discos de máquinas virtuales (archivos VHD). En su lugar, utilice [Azure Backup][azure-backup].   
>
>

## <a name="security"></a>Seguridad

**Implemente la protección de nivel de aplicación frente a la denegación de servicio distribuido (DDoS).** Los servicios de Azure están protegidos frente a ataques de DDos en la capa de red. Sin embargo, Azure no puede protegerse contra los ataques de la capa de aplicación, ya que es difícil distinguir entre las solicitudes de usuarios reales y las solicitudes de usuarios malintencionados. Para más información sobre cómo protegerse de ataques de DDoS de capa de aplicación, consulte la sección "Protecting against DDoS" (Proteger frente a DDoS) de [Microsoft Azure Network Security](http://download.microsoft.com/download/C/A/3/CA3FC5C0-ECE0-4F87-BF4B-D74064A00846/AzureNetworkSecurity_v3_Feb2015.pdf) (descarga de PDF).

**Implemente el principio del privilegio mínimo para acceder a los recursos de la aplicación.** El acceso predeterminado a los recursos de la aplicación debe ser lo más restrictivo posible. Otorgue permisos de mayor nivel sobre la base de la aprobación. Conceder un acceso excesivamente permisivo a los recursos de la aplicación de forma predeterminada puede dar lugar a que alguien elimine recursos de forma intencionada o accidental. Azure proporciona [control de acceso basado en rol](/azure/active-directory/role-based-access-built-in-roles/) para administrar privilegios de usuario, pero es importante comprobar los permisos de privilegio mínimo para otros recursos que tienen sus propios sistemas de permisos, como SQL Server.

## <a name="testing"></a>Prueba

**Realice pruebas de conmutación por error y de conmutación por recuperación para la aplicación.** Si no ha probado completamente la conmutación por error y la conmutación por recuperación, no puede estar seguro de que los servicios dependientes de la aplicación se recuperen de manera sincronizada durante la recuperación ante desastres. Asegúrese de que los servicios dependientes de la aplicación realizan la conmutación por error y la conmutación por recuperación en el orden correcto.

**Realice pruebas de inyección de errores en la aplicación.** La aplicación puede producir un error por muchas razones diferentes, como la expiración de certificados, el agotamiento de los recursos del sistema en una máquina virtual o errores de almacenamiento. Pruebe la aplicación en un entorno lo más cercano posible a producción, mediante la simulación o el desencadenamiento de errores reales. Por ejemplo, elimine certificados, consuma recursos del sistema de forma artificial o elimine un origen de almacenamiento. Compruebe la capacidad de la aplicación para recuperarse de todo tipo de errores, sola y en combinación. Compruebe que los errores no se estén propagando en cascada por todo el sistema.

**Ejecute pruebas en producción mediante datos de usuario tanto sintéticos como reales.** El entorno de prueba y de producción rara vez son idénticos, por lo que es importante utilizar una implementación azul-verde o canario y de valor controlado y pruebe la aplicación en producción. Esto le permite probar la aplicación en producción bajo carga real y asegurarse de que funcionará como se espera cuando esté completamente implementada.

## <a name="deployment"></a>Implementación

**Documente el proceso de lanzamiento de la aplicación.** Sin la documentación detallada del proceso de lanzamiento, el operador puede implementar una mala actualización o configurar de manera incorrecta la configuración de la aplicación. Defina y documente claramente el proceso de lanzamiento y asegúrese de que esté disponible para todo el equipo de operaciones. 

**Automatice el proceso de implementación de la aplicación.** Si se requiere que el personal de operaciones implemente manualmente la aplicación, un error humano puede causar un error en la implementación. 

**Diseñe el proceso de lanzamiento para maximizar la disponibilidad de la aplicación.** Si su proceso de lanzamiento requiere que los servicios se desconecten durante la implementación, la aplicación no estará disponible hasta que vuelvan a conectarse. Utilice la técnica de implementación [verde/azul](http://martinfowler.com/bliki/BlueGreenDeployment.html) o de [lanzamiento controlado](http://martinfowler.com/bliki/CanaryRelease.html) para implementar la aplicación en producción. Ambas técnicas implican la implementación del código de lanzamiento junto con el código de producción, para que los usuarios del código de lanzamiento puedan redireccionarse al código de producción en caso de producirse un error.

**Registre y audite las implementaciones de la aplicación.** Si utiliza técnicas de implementación por etapas, como las implementaciones azul/verde o el lanzamiento controlado, habrá más de una versión de su aplicación que se ejecuta en producción. Si se produce un problema, es fundamental determinar qué versión de la aplicación lo está causando. Implemente una estrategia de registro robusta para capturar tanta información específica de la versión como sea posible.

**Tenga un plan de reversión para la implementación.** Es posible que la implementación de la aplicación pueda producir un error y hacer que la aplicación no esté disponible. Diseñe un proceso de reversión para regresar a una última versión buena conocida y minimizar el tiempo de inactividad. 

## <a name="operations"></a>Operaciones

**Implemente los procedimientos recomendados de supervisión y alertas en la aplicación.** Sin una supervisión, un diagnóstico y unas alertas adecuados, no hay manera de detectar errores en la aplicación y alertar al operador para que los solucione. Para más información, consulte la [Guía de supervisión y diagnóstico][monitoring-and-diagnostics-guidance].

**Mida las estadísticas de llamadas remotas y ponga la información a disposición del equipo de la aplicación.**  Si no realiza el seguimiento e informa de las estadísticas de llamadas remotas en tiempo real ni proporciona una forma fácil de revisar esta información, el equipo de operaciones no tendrá una visión instantánea del estado de la aplicación. Y si solo mide el tiempo medio de las llamadas remotas, no tendrá suficiente información para revelar problemas en los servicios. Resuma las métricas de llamadas remotas tales como latencia, rendimiento y errores en los percentiles 99 y 95. Realice un análisis estadístico de las métricas para descubrir los errores que ocurren dentro de cada percentil.

**Realice el seguimiento del número de excepciones transitorias y de reintentos durante un período de tiempo apropiado.** Si no realiza el seguimiento y supervisa las excepciones transitorias y los intentos de reintentos a lo largo del tiempo, es posible que un problema o error pueda estar oculto por la lógica de reintento de la aplicación. Es decir, si la supervisión y registro solo muestra el éxito o fracaso de una operación, se ocultará el hecho de que la operación tuvo que ser repetida varias veces debido a las excepciones. Una tendencia al aumento de las excepciones con el tiempo indica que el servicio tiene un problema y puede producir errores. Para más información, consulte [Guía de reintentos para servicios específicos][retry-service-guidance].

**Implemente un sistema de advertencia temprana que alerte al operador.** Identifique los indicadores clave de rendimiento del estado de la aplicación, como las excepciones transitorias y la latencia de llamadas remotas, y establezca los valores de umbral apropiados para cada uno de ellos. Envíe una alerta a las operaciones cuando se alcanza el valor de umbral. Establezca estos umbrales a niveles que identifiquen los problemas antes de que se vuelvan críticos y requieran una respuesta de recuperación.

**Asegúrese de que más de una persona del equipo esté entrenada para supervisar la aplicación y realizar cualquier paso de recuperación manual.** Si solo hay un único operador en el equipo que pueda supervisar la aplicación y poner en marcha los pasos de recuperación, esa persona se convierte en un único punto de error. Entrene a varias personas en la detección y recuperación, y asegúrese de que siempre haya al menos una persona activa en todo momento.

**Asegúrese de que la aplicación no se encuentre con los [límites de suscripción de Azure](/azure/azure-subscription-service-limits/).** Las suscripciones a Azure tienen límites en ciertos tipos de recursos, como el número de grupos de recursos, el número de núcleos y el número de cuentas de almacenamiento.  Si los requisitos de aplicación superan los límites de suscripción de Azure, cree otra suscripción de Azure y aprovisione allí recursos suficientes.

**Asegúrese de que la aplicación no se encuentre con los [límites por servicio](/azure/azure-subscription-service-limits/).** Los servicios individuales de Azure tienen límites de consumo; por ejemplo, límites de almacenamiento, rendimiento, número de conexiones, solicitudes por segundo y otras métricas. La aplicación producirá un error si intenta utilizar recursos más allá de estos límites. Esto provocará una limitación del servicio y un posible tiempo de inactividad a los usuarios afectados. Dependiendo del servicio específico y de los requisitos de su aplicación, a menudo puede evitar estos límites escalando verticalmente (por ejemplo, al seleccionar otro plan de tarifas) u horizontalmente (al agregar nuevas instancias).  

**Diseñe los requisitos de almacenamiento de la aplicación para que se ajusten a los objetivos de escalabilidad y rendimiento del almacenamiento de Azure.** El almacenamiento de Azure está diseñado para funcionar dentro de objetivos de escalabilidad y rendimiento predefinidos, por lo que diseñe la aplicación para utilizar el almacenamiento dentro de dichos objetivos. Si supera estos objetivos, la aplicación experimentará una limitación del almacenamiento. Para solucionar este problema, proporcione cuentas de almacenamiento adicionales. Si se encuentra con el límite de la cuenta de almacenamiento, proporcione suscripciones de Azure adicionales y, después, proporcione cuentas de almacenamiento adicionales allí. Para obtener más información, consulte [Objetivos de escalabilidad y rendimiento de Azure Storage](/azure/storage/storage-scalability-targets/).

**Seleccione el tamaño adecuado de la máquina virtual para la aplicación.** Mida la CPU, la memoria, el disco y la E/S reales de las máquinas virtuales en producción y compruebe que el tamaño de la máquina virtual seleccionado sea suficiente. Si no es así, la aplicación puede experimentar problemas de capacidad a medida que las máquinas virtuales se acercan a sus límites. Los tamaños de las máquinas virtuales se describen con detalle en [Tamaños de máquinas virtuales en Azure](/azure/virtual-machines/virtual-machines-windows-sizes/?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

**Determine si la carga de trabajo de la aplicación es estable o fluctúa con el tiempo.** Si la carga de trabajo fluctúa con el tiempo, utilice Azure VM Scale Sets para escalar automáticamente el número de instancias de máquinas virtuales. De lo contrario, tendrá que aumentar o disminuir de manera manual el número de máquinas virtuales. Para más información, consulte [Introducción a los conjuntos de escalado de máquinas virtuales](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview/).

**Seleccione el nivel de servicio correcto para Azure SQL Database.** Si la aplicación utiliza Azure SQL Database, asegúrese de que ha seleccionado el nivel de servicio apropiado. Si selecciona un nivel que no es capaz de controlar los requisitos de la unidad de transmisión de datos (DTU) de la aplicación, el uso de datos estará limitado. Si quiere obtener más información sobre cómo seleccionar el plan de servicio correcto, consulte [Opciones y rendimiento de SQL Database: comprender lo que está disponible en cada nivel de servicio](/azure/sql-database/sql-database-service-tiers/).

**Cree un proceso para interactuar con el soporte técnico de Azure.** Si el proceso para contactar con el [soporte técnico de Azure](https://azure.microsoft.com/support/plans/) no está establecido antes de que surja la necesidad de ponerse en contacto con el soporte técnico, el tiempo de inactividad se prolongará a medida que el proceso de soporte técnico se recorra por primera vez. Incluya el proceso para ponerse en contacto con el soporte técnico y escalar los problemas como parte de la resistencia de la aplicación desde el principio.

**Asegúrese de que la aplicación no utilice más del número máximo de cuentas de almacenamiento por suscripción.** Azure permite un máximo de 200 cuentas de almacenamiento por suscripción. Si la aplicación requiere más cuentas de almacenamiento que las disponibles actualmente en la suscripción, deberá crear una nueva suscripción y crear cuentas de almacenamiento adicionales. Para más información, consulte [Límites, cuotas y restricciones de suscripción y servicios de Microsoft Azure](/azure/azure-subscription-service-limits/#storage-limits).

**Asegúrese de que la aplicación no supere los objetivos de escalabilidad para los discos de máquinas virtuales.** Las máquinas virtuales IaaS de Azure permiten asociar un número de discos de datos dependiendo de varios factores, como el tamaño de la máquina virtual y el tipo de cuenta de almacenamiento. Si la aplicación supera los objetivos de escalabilidad para los discos de máquinas virtuales, aprovisione cuentas de almacenamiento adicionales y cree allí los discos de máquinas virtuales. Para más información, consulte [Objetivos de escalabilidad y rendimiento de Azure Storage](/azure/storage/storage-scalability-targets/#scalability-targets-for-virtual-machine-disks).

## <a name="telemetry"></a>Telemetría

**Registre los datos de telemetría mientras la aplicación se ejecuta en el entorno de producción.** Capture la información de telemetría eficaz mientras la aplicación se está ejecutando en el entorno de producción o no tendrá suficiente información para diagnosticar la causa de los problemas mientras presta servicio activamente a los usuarios. Para más información, consulte [Supervisión y diagnóstico][monitoring-and-diagnostics-guidance].

**Implemente el registro con un patrón asincrónico.** Si las operaciones de registro son sincrónicas, podrían bloquear el código de aplicación. Asegúrese de que las operaciones de registro se implementan como operaciones asincrónicas.

**Correlacione los datos de registro en los límites de servicio.** En una aplicación típica de n niveles, una solicitud de usuario puede atravesar varios límites de servicio. Por ejemplo, una solicitud de usuario normalmente se origina en el nivel web y se pasa al nivel empresarial y, finalmente. persiste en la capa de datos. En escenarios más complejos, se puede distribuir una solicitud de usuario a muchos servicios y almacenes de datos diferentes. Asegúrese de que el sistema de registro correlacione las llamadas entre los límites del servicio para que pueda realizar un seguimiento de la solicitud en toda la aplicación.

## <a name="azure-resources"></a>Recursos de Azure.

**Use las plantillas de Azure Resource Manager para aprovisionar recursos.** Las plantillas de Resource Manager facilitan la automatización de implementaciones a través de PowerShell o la CLI de Azure, lo que permite un proceso de implementación más confiable. Para más información, consulte [Información general de Azure Resource Manager][resource-manager].

**Asigne a los recursos nombres descriptivos.** Al asignar a los recursos nombre descriptivos, será más fácil encontrarlos y comprender su rol. Para más información, consulte [Convenciones de nomenclatura](../best-practices/naming-conventions.md).

**Use el control de acceso basado en rol (RBAC).** Utilice RBAC para controlar el acceso a los recursos de Azure que implemente. RBAC le permite asignar roles de autorización a los miembros del equipo de DevOps para evitar eliminaciones o modificaciones accidentales en los recursos implementados. Para más información, consulte [Introducción al control de acceso basado en roles en Azure Portal](/azure/active-directory/role-based-access-control-what-is/).

**Utilice los bloqueos de recursos para recursos críticos, como máquinas virtuales.** Los bloqueos de recursos evitan que un operador elimine accidentalmente un recurso. Para más información, consulte [Bloqueo de recursos con Azure Resource Manager](/azure/azure-resource-manager/resource-group-lock-resources/).

**Elija pares regionales.** Cuando se realiza la implementación en dos regiones, elija regiones del mismo par regional. En el caso de una interrupción amplia, tiene prioridad la recuperación de una región de cada pareja. Algunos servicios, como el almacenamiento con redundancia geográfica ofrecen replicación automática a la región emparejada. Para más información, consulte [Continuidad empresarial y recuperación ante desastres (BCDR): regiones emparejadas de Azure](/azure/best-practices-availability-paired-regions).

**Organice los grupos de recursos por función y ciclo de vida.**  En general, un grupo de recursos debe contener recursos que comparten el mismo ciclo de vida. Esto facilita la administración de implementaciones, elimina implementaciones de prueba y asigna derechos de acceso, con lo que se reduce la posibilidad de que se elimine o modifique accidentalmente una implementación de producción. Cree grupos de recursos independientes para entornos de producción, desarrollo y pruebas. En una implementación de varias regiones, ponga los recursos para cada región en grupos de recursos independientes. Esto facilita volver a implementar una región sin que afecte a las demás regiones.

## <a name="next-steps"></a>Pasos siguientes

- [Lista de comprobación de resistencia para servicios de Azure específicos](./resiliency-per-service.md)
- [Análisis del modo de error](../resiliency/failure-mode-analysis.md)


<!-- links -->
[app-service-autoscale]: /azure/monitoring-and-diagnostics/insights-how-to-scale/
[asynchronous-c-sharp]: /dotnet/articles/csharp/async
[availability-sets]:/azure/virtual-machines/virtual-machines-windows-manage-availability/
[azure-backup]: https://azure.microsoft.com/documentation/services/backup/
[circuit-breaker]: ../patterns/circuit-breaker.md
[cloud-service-autoscale]: /azure/cloud-services/cloud-services-how-to-scale/
[fma]: ../resiliency/failure-mode-analysis.md
[load-balancer]: /azure/load-balancer/load-balancer-overview/
[monitoring-and-diagnostics-guidance]: ../best-practices/monitoring.md
[resource-manager]: /azure/azure-resource-manager/resource-group-overview/
[retry-pattern]: ../patterns/retry.md
[retry-service-guidance]: ../best-practices/retry-service-specific.md
[traffic-manager]: /azure/traffic-manager/traffic-manager-overview/
[traffic-manager-routing]: /azure/traffic-manager/traffic-manager-routing-methods/
[vmss-autoscale]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview/
