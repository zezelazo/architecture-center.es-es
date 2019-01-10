---
title: Lista de comprobación de resistencia para servicios de Azure
titleSuffix: Azure Design Review Framework
description: Lista de comprobación que proporciona una orientación sobre la resistencia de varios servicios de Azure.
author: petertaylor9999
ms.date: 11/26/2018
ms.custom: resiliency, checklist
ms.openlocfilehash: e1fb780cf9f54a5078cc5d3c6b597b351f93e05e
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/08/2019
ms.locfileid: "54112675"
---
# <a name="resiliency-checklist-for-specific-azure-services"></a>Lista de comprobación de resistencia para servicios de Azure específicos

La resistencia es la capacidad de un sistema para recuperarse de errores y seguir en funcionamiento y es uno de los [pilares de la calidad del software](../guide/pillars.md). Todas las tecnologías tienen sus propios modos de error particulares que debe tener en cuenta al diseñar e implementar la aplicación. Use esta lista de comprobación para revisar las consideraciones de resistencia para servicios específicos de Azure. Revise también la [lista de comprobación de resistencia general](./resiliency.md).

## <a name="app-service"></a>App Service

**Utilice el nivel Estándar o Premium.** Estos niveles admiten espacios de ensayo y copias de seguridad automáticas. Para más información, consulte [Introducción detallada sobre los planes de Azure App Service](/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview/).

**Evite el escalado o la reducción verticales.** En su lugar, seleccione un nivel y un tamaño de instancia que satisfagan sus requisitos de rendimiento con la carga típica y, a continuación, [escale horizontalmente](/azure/app-service-web/web-sites-scale/) las instancias para controlar los cambios en el volumen de tráfico. El escalado y la reducción verticales pueden desencadenar un reinicio de la aplicación.

**Almacene la configuración como valores de configuración de la aplicación.** Utilice la configuración de la aplicación para almacenar los valores de configuración como valores de configuración de la aplicación. Defina la configuración en las plantillas de Resource Manager o utilice PowerShell para que pueda aplicarlas como parte de un proceso automatizado de implementación o actualización, que es más confiable. Para más información, consulte [Configuración de aplicaciones web en Azure App Service](/azure/app-service-web/web-sites-configure/).

**Cree planes de App Service independientes para producción y prueba.** No utilice ranuras en la implementación de producción para pruebas.  Todas las aplicaciones del mismo plan de App Service comparten las mismas instancias de máquina virtual. Si coloca implementaciones de producción y prueba en el mismo plan, puede afectar negativamente a la implementación de producción. Por ejemplo, las pruebas de carga pueden degradar el sitio de producción en vivo. Al colocar las implementaciones de prueba en un plan independiente, se aíslan de la versión de producción.

**Aplicaciones web independiente de las API web.** Si la solución tiene un front-end web y una API web, considere descomponerlas en aplicaciones de App Service independientes. Este diseño facilita la descomposición de la solución por carga de trabajo. Puede ejecutar la aplicación web y la API en planes de App Service diferentes, por lo que se pueden escalar de forma independiente. Si inicialmente no necesita ese nivel de escalabilidad, puede implementar las aplicaciones en el mismo plan y moverlas más tarde a planes diferentes si es necesario.

**Evite utilizar la característica de copia de seguridad de App Service para hacer copias de seguridad de las bases de datos Azure SQL Database.** En cambio, utilice las [copias de seguridad automatizadas de SQL Database][sql-backup]. La copia de seguridad de App Service exporta la base de datos a un archivo .bacpac de SQL, que cuesta unidades de transmisión de datos.

**Realice la implementación en un espacio de ensayo.** Cree una ranura de implementación para el almacenamiento provisional. Implemente las actualizaciones de la aplicación en el espacio de ensayo y compruebe la implementación antes de cambiarla a producción. Esto reduce la posibilidad de una mala actualización en la producción. También garantiza que todas las instancias estén preparadas antes de pasarlas a producción. Muchas aplicaciones presentan tiempos de preparación y arranque en frío considerables. Para más información, consulte [Configuración de entornos de ensayo para aplicaciones web en Azure App Service](/azure/app-service-web/web-sites-staged-publishing/).

**Cree una ranura de implementación para conservar la última implementación válida conocida.** Cuando implemente una actualización en producción, traslade la implementación de producción anterior a la ranura de la última configuración válida conocida. Esto facilita la reversión de una implementación incorrecta. Si detecta un problema más tarde, puede revertir rápidamente a la última versión válida conocida. Para más información, consulte [Basic web application](../reference-architectures/app-service-web-app/basic-web-app.md) (Aplicación web básica).

**Habilite el registro de diagnóstico**, incluido el registro de aplicaciones y el registro del servidor web. El registro es importante para la supervisión y diagnóstico. Consulte [Habilitación del registro de diagnóstico para aplicaciones web en Azure App Service](/azure/app-service-web/web-sites-enable-diagnostic-log/).

**Regístrese en Blob Storage.** Esto facilita la recopilación y el análisis de los datos.

**Cree una cuenta de almacenamiento para los registros.** No use la misma cuenta de almacenamiento para los registros y los datos de la aplicación. Esto ayuda a evitar que el registro reduzca el rendimiento de la aplicación.

**Supervise el rendimiento.** Use un servicio de supervisión de rendimiento, como [New Relic](https://newrelic.com/) o [Application Insights](/azure/application-insights/app-insights-overview/), para supervisar el rendimiento de la aplicación y el comportamiento sometido a carga.  La supervisión del rendimiento le ofrece una visión en tiempo real de la aplicación. Le permite diagnosticar problemas y realizar análisis de causa raíz de errores.

## <a name="application-gateway"></a>Application Gateway

**Aprovisione al menos dos instancias.** Implemente Application Gateway con al menos dos instancias. Una única instancia es un único punto de error. Utilice dos o más instancias para redundancia y escalabilidad. Para calificar para el [Acuerdo de Nivel de Servicio](https://azure.microsoft.com/support/legal/sla/application-gateway), debe proporcionar dos o más instancias medianas o más grandes.

## <a name="cosmos-db"></a>Cosmos DB

**Replique la base de datos en todas las regiones.** Cosmos DB permite asociar cualquier número de regiones de Azure con una cuenta de base de datos de Cosmos DB. Una base de datos Cosmos DB puede tener una región de escritura y varias regiones de lectura. Si se produce un error en la región de escritura, puede leer desde otra réplica. El SDK de cliente lo trata automáticamente. También puede conmutar por error la región de escritura a otra región. Para más información, consulte [Cómo se distribuyen datos globalmente con Azure Cosmos DB](/azure/cosmos-db/distribute-data-globally).

## <a name="event-hubs"></a>Event Hubs

**Use puntos de control**.  Un consumidor de eventos debe escribir su posición actual en el almacenamiento persistente según un intervalo predefinido. De este modo, si el consumidor experimenta un error (por ejemplo, el consumidor se bloquea o se produce un error en el host), una nueva instancia puede reanudar la lectura de la secuencia desde la última posición registrada. Para más información, consulte [Consumidores de eventos](/azure/event-hubs/event-hubs-features#event-consumers).

**Controle los mensajes duplicados.** Si se produce un error en un consumidor de eventos, el procesamiento de mensajes se reanudará desde el último punto de comprobación registrado. Todos los mensajes que ya se procesaron después del último punto de comprobación se procesarán de nuevo. Por tanto, su lógica de procesamiento de mensajes debe ser idempotente, o la aplicación debe poder desduplicar los mensajes.

**Controle las excepciones**. Normalmente, un consumidor de eventos procesa un lote de mensajes en un bucle. Debe controlar las excepciones dentro de este bucle de procesamiento para evitar la pérdida de un lote completo de mensajes si un solo mensaje provoca una excepción.

**Use una cola de mensajes con problemas de entrega**. Si al procesar un mensaje se produce un error no transitorio, coloque el mensaje en una cola de mensajes con problemas de entrega para que pueda hacer un seguimiento del estado. Según el escenario, puede volver a intentar enviar el mensaje más tarde, aplicar una transacción de compensación o realizar alguna otra acción. Tenga en cuenta que Event Hubs no tiene ninguna funcionalidad integrada de cola de mensajes con problemas de entrega. Puede usar Azure Queue Storage o Service Bus para implementar una cola de mensajes con problemas de entrega o emplear Azure Functions, o algún otro mecanismo de eventos.

**Implemente la recuperación ante desastres mediante la conmutación por error en un espacio de nombres secundario de Event Hubs.** Para más información, consulte [Recuperación ante desastres con localización geográfica de Azure Event Hubs](/azure/event-hubs/event-hubs-geo-dr).

## <a name="redis-cache"></a>Redis Cache

**Configurar la replicación geográfica**. La replicación geográfica proporciona un mecanismo para vincular dos instancias de Azure Redis Cache de nivel Premium. Los datos escritos en la caché principal se replican en una caché secundaria de solo lectura. Para más información, consulte [Configuración de replicación geográfica para Azure Redis Cache](/azure/redis-cache/cache-how-to-geo-replication).

**Configurar la persistencia de los datos.** La persistencia de Redis le permite conservar los datos almacenados en Redis. También puede tomar instantáneas y realizar copias de seguridad de los datos que puede cargar en el caso de un error de hardware. Para más información, consulte [Cómo configurar la persistencia de datos para una instancia premium de Azure Redis Cache](/azure/redis-cache/cache-how-to-premium-persistence).

Si usa Redis Cache como caché de datos temporal y no como almacén persistente, puede que estas recomendaciones no se apliquen.

## <a name="search"></a>Search

**Aprovisione más de una réplica.** Utilice al menos dos réplicas para alta disponibilidad de lectura, o tres para alta disponibilidad de lectura-escritura.

**Configure los indexadores para implementaciones en varias regiones**. Si tiene una implementación de varias regiones, tenga en cuenta las opciones para la continuidad en la indexación.

- Si el origen de datos está replicado geográficamente, por lo general debe dirigir cada indexador de cada servicio regional de Azure Search a su réplica local del origen de datos. Sin embargo, este enfoque no se recomienda para grandes conjuntos de datos almacenados en Azure SQL Database. La razón es que Azure Search no puede realizar la indexación incremental desde réplicas secundarias de SQL Database, solo desde réplicas principales. En su lugar, seleccione todos los indexadores a la réplica principal. Después de una conmutación por error, elija los indexadores de Azure Search a la nueva réplica principal.

- Si el origen de datos no está replicado geográficamente, seleccione varios indexadores en el mismo origen de datos, de modo que los servicios de Azure Search en múltiples regiones se indexen continua e independientemente en el origen de datos. Para más información, consulte [Consideraciones sobre el rendimiento y la optimización de Azure Search][search-optimization].

## <a name="service-bus"></a>Azure Service Bus

**Utilice el nivel Premium para cargas de trabajo de producción**. La [mensajería Premium de Service Bus](/azure/service-bus-messaging/service-bus-premium-messaging) proporciona recursos de procesamiento dedicados y reservados y capacidad de memoria para permitir un rendimiento predecible. El nivel de mensajería Premium también ofrece acceso a nuevas características que solo están disponibles para los clientes Premium en primer lugar. Puede decidir el número de unidades de mensajería en función de las cargas de trabajo esperadas.

**Controle los mensajes duplicados**. Si se produce un error en un publicador inmediatamente después de enviar un mensaje o experimenta problemas de red o del sistema, es posible que por error no se registre que se entregó el mensaje y se podría enviar el mismo mensaje al sistema dos veces. Service Bus puede controlar este problema habilitando la detección de duplicados. Para más información, consulte [Detección de duplicados](/azure/service-bus-messaging/duplicate-detection).

**Controle las excepciones**. Las API de mensajería generan excepciones cuando se produce un error de usuario, un error de configuración u otro error. El cliente (remitentes y receptores) debe controlar estas excepciones en su código. Esto es especialmente importante en el procesamiento por lotes, donde se puede usar el control de excepciones para evitar la pérdida de un lote completo de mensajes. Para más información, consulte [Excepciones de mensajería de Service Bus](/azure/service-bus-messaging/service-bus-messaging-exceptions).

**Directiva de reintentos**. Service Bus permite elegir la mejor directiva de reintentos para las aplicaciones. La directiva predeterminada es permitir un máximo de 9 intentos de reintento y esperar 30 segundos, pero esto se puede ajustar aún más. Para más información, consulte [Directiva de reintentos: Service Bus](/azure/architecture/best-practices/retry-service-specific#service-bus).

**Use una cola de mensajes fallidos**. Si un mensaje no se puede procesar o entregar a algún receptor después de varios reintentos, se mueve a una cola de mensajes fallidos. Implemente un proceso para leer los mensajes de la cola de mensajes fallidos, inspeccionarlos y corregir el problema. Según el escenario, puede reintentar el envío del mensaje tal cual es, realizar cambios y volver a intentarlo o descartar el mensaje. Para más información, consulte [Introducción a las colas de mensajes fallidos de Service Bus](/azure/service-bus-messaging/service-bus-dead-letter-queues).

**Utilice la recuperación ante desastres geográfica**. La recuperación ante desastres geográfica garantiza que el procesamiento de datos siga funcionando en otra región o centro de datos si una región de Azure completa o el centro de datos dejan de estar disponibles debido a un desastre. Para obtener más información, consulte [Recuperación ante desastres con localización geográfica de Azure Service Bus](/azure/service-bus-messaging/service-bus-geo-dr).

## <a name="storage"></a>Storage

**Para los datos de aplicación, use almacenamiento con redundancia geográfica con acceso de lectura (RA-GRS).** El almacenamiento con redundancia geográfica con acceso de lectura replica los datos en una región secundaria y proporciona acceso de solo lectura desde la región secundaria. Si hay una interrupción de almacenamiento en la región primaria, la aplicación puede leer los datos de la región secundaria. Para más información, consulte [Replicación de Azure Storage](/azure/storage/storage-redundancy/).

**Para los discos de máquinas virtuales, use Managed Disks.** [Managed Disks][managed-disks] proporciona una mayor confiabilidad para las máquinas virtuales en los conjuntos de disponibilidad, porque los discos están suficientemente aislados entre sí para evitar únicos puntos de error. Además, Managed Disks no está sujeto a los límites de IOPS de los discos duros virtuales creados en una cuenta de almacenamiento. Para más información, consulte [Administración de la disponibilidad de las máquinas virtuales Windows en Azure][vm-manage-availability].

**Para Queue Storage, cree una cola de copia de seguridad en otra región.** Para Queue Storage, las réplicas de solo lectura tienen un uso limitado, ya que no se puede poner elementos en cola o quitarlos. En su lugar, cree una cola de copia de seguridad en una cuenta de almacenamiento en otra región. Si se produce una interrupción de almacenamiento, la aplicación puede usar la cola de copia de seguridad hasta que la región primaria vuelva a estar disponible. De este modo, la aplicación puede seguir procesando nuevas solicitudes.

## <a name="sql-database"></a>SQL Database

**Utilice el nivel Estándar o Premium.** Estos niveles ofrecen un período de restauración a un momento dado más prolongado (35 días). Para más información, consulte [Opciones y rendimiento de SQL Database](/azure/sql-database/sql-database-service-tiers/).

**Habilite la auditoría de SQL Database.** La auditoría se puede utilizar para diagnosticar ataques maliciosos o errores humanos. Para obtener más información, consulte [Introducción a la auditoría de la Base de datos SQL](/azure/sql-database/sql-database-auditing-get-started/).

**Use la replicación geográfica activa** Utilice esta replicación para crear una réplica secundaria legible en una región distinta.  Si se produce un error en la base de datos principal o, simplemente, debe estar sin conexión, realice una conmutación por error manual a la base de datos secundaria.  Hasta que se realiza la conmutación por error, la base de datos secundaria sigue siendo de solo lectura.  Para más información, consulte [Replicación geográfica activa para Azure SQL Database](/azure/sql-database/sql-database-geo-replication-overview/).

**Utilice el particionamiento.** Considere el uso del particionamiento para realizar una partición horizontal de la base de datos. El particionamiento puede proporcionar aislamiento de errores. Para más información, consulte [Escalado horizontal con Azure SQL Database](/azure/sql-database/sql-database-elastic-scale-introduction/).

**Utilice la restauración a un momento dado para realizar la recuperación en caso de errores humanos.**  La restauración a un momento dado devuelve la base de datos a un momento anterior en el tiempo. Para más información, consulte [Recuperación de una instancia de Azure SQL Database mediante copias de seguridad de datos automatizadas][sql-restore].

**Use la restauración geográfica para recuperarse de una interrupción del servicio.** La restauración geográfica restaura una base de datos a partir de una copia de seguridad con redundancia geográfica.  Para más información, consulte [Recuperación de una instancia de Azure SQL Database mediante copias de seguridad de datos automatizadas][sql-restore].

## <a name="sql-data-warehouse"></a>SQL Data Warehouse

**No deshabilite la copia de seguridad con redundancia geográfica.** De forma predeterminada, SQL Data Warehouse realiza una copia de seguridad completa de los datos cada 24 horas por si es preciso realizar una recuperación ante un desastre. No se recomienda desactivar esta característica. Para más información, consulte [Copias de seguridad geográficas](/azure/sql-data-warehouse/backup-and-restore#geo-backups).

## <a name="sql-server-running-in-a-vm"></a>SQL Server que se ejecuta en una máquina virtual

**Replique la base de datos.** Use los Grupos de disponibilidad Always On de SQL Server para replicar la base de datos. Proporciona alta disponibilidad si se produce un error en una instancia de SQL Server. Para más información, consulte [Ejecución de máquinas virtuales Windows para una arquitectura de n niveles](../reference-architectures/virtual-machines-windows/n-tier.md).

**Realice una copia de seguridad de la base de datos**. Si ya utiliza [Azure Backup](/azure/backup/) para realizar copias de seguridad de sus máquinas virtuales, considere la posibilidad de usar [Azure Backup para cargas de trabajo de SQL Server con DPM](/azure/backup/backup-azure-backup-sql/). Con este enfoque, hay un rol de administrador de copias de seguridad para la organización y un procedimiento unificado de recuperación para máquinas virtuales y SQL Server. En caso contrario, consulte [Copia de seguridad administrada de SQL Server en Microsoft Azure](https://msdn.microsoft.com/library/dn449496.aspx).

## <a name="traffic-manager"></a>Traffic Manager

**Realice la conmutación por recuperación manual.** Después de una conmutación por error de Traffic Manager, realice la conmutación por recuperación manual, en lugar de la automática. Antes de la conmutación por recuperación, compruebe que todos los subsistemas de aplicación tengan un estado correcto.  En caso contrario, puede crear una situación donde la aplicación va y viene incesantemente entre centros de datos. Para más información, consulte [Ejecución de máquinas virtuales Windows en varias regiones para lograr alta disponibilidad](../reference-architectures/virtual-machines-windows/multi-region-application.md).

**Cree un punto de conexión del sondeo de estado.** Crear un punto de conexión personalizado que informe sobre el estado general de la aplicación. Esto permite que Traffic Manager conmute por error si se produce un error en cualquier ruta de acceso crítica, no solo el front-end. El punto de conexión debe devolver un código de error HTTP si cualquier dependencia crítica es incorrecta o inaccesible. Sin embargo, no informe de errores para servicios no críticos. De lo contrario, el sondeo de estado podría desencadenar una conmutación por error cuando no sea necesario, con lo que se crean falsos positivos. Para más información, consulte [Supervisión de puntos de conexión de Traffic Manager](/azure/traffic-manager/traffic-manager-monitoring/).

## <a name="virtual-machines"></a>Virtual Machines

**Evite ejecutar una carga de trabajo de producción en una única máquina virtual.** Una única implementación de máquina virtual no es resistente al mantenimiento planeado o no planeado. En su lugar, coloque varias máquinas virtuales en un conjunto de disponibilidad o [conjunto de escalado de máquina virtual](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview/), con un equilibrador de carga delante.

**Especifique un conjunto de disponibilidad cuando aprovisione la máquina virtual.** Actualmente, no hay ninguna manera de agregar una máquina virtual a un conjunto de disponibilidad después de aprovisionar la máquina virtual. Cuando se agrega una nueva máquina virtual a un grupo de disponibilidad existente, asegúrese de crear una NIC para la máquina virtual y agregue la NIC al grupo de direcciones back-end en el equilibrador de carga. De lo contrario, el equilibrador de carga no enrutará el tráfico de la red a esa máquina virtual.

**Ponga cada capa de aplicación en un conjunto de disponibilidad independiente.** En una aplicación de n niveles, no coloque máquinas virtuales de diferentes niveles en el mismo conjunto de disponibilidad. Las máquinas virtuales de un conjunto de disponibilidad se colocan en dominios de error y en dominios de actualización. Sin embargo, para obtener el beneficio de redundancia de los dominios de error y de los dominios de disponibilidad, cada máquina virtual en el conjunto de disponibilidad debe ser capaz de tratar las mismas solicitudes del cliente.

**Replicación de máquinas virtuales con Azure Site Recovery** Al replicar máquinas virtuales de Azure con [Site Recovery][site-recovery], todos los discos de máquina virtual se replican continuamente de forma asincrónica en la región de destino. Los puntos de recuperación se crean cada pocos minutos. Esto le ofrece un objetivo de punto de recuperación (RPO) en el orden de minutos. Puede realizar tantos simulacros de recuperación ante desastres como desee, sin que afecte a la aplicación de producción ni la replicación en curso. Para más información, consulte [Ejecución de un simulacro de recuperación ante desastres en Azure][site-recovery-test].

**Elija el tamaño de la máquina virtual adecuado según los requisitos de rendimiento.** Al mover una carga de trabajo existente a Azure, comience con el tamaño de máquina virtual que más se acerque a los servidores locales. Luego, mida el rendimiento de la carga de trabajo real con respecto a la CPU, la memoria y la IOPS de disco, y ajuste el tamaño, si es necesario. Esto ayuda a asegurar que la aplicación se comporte según lo previsto en un entorno de nube. Además, si tiene varias tarjetas NIC, tenga en cuenta el límite de NIC para cada tamaño.

**Use Managed Disks en discos duros virtuales.** [Managed Disks][managed-disks] proporciona una mayor confiabilidad para las máquinas virtuales en los conjuntos de disponibilidad, porque los discos están suficientemente aislados entre sí para evitar únicos puntos de error. Además, Managed Disks no está sujeto a los límites de IOPS de los discos duros virtuales creados en una cuenta de almacenamiento. Para más información, consulte [Administración de la disponibilidad de las máquinas virtuales Windows en Azure][vm-manage-availability].

**Instale las aplicaciones en un disco de datos, no en el disco del sistema operativo.** De lo contrario, puede alcanzar el límite de tamaño de disco.

**Use Azure Backup para hacer copia de seguridad de las máquinas virtuales.** Las copias de seguridad protegen contra la pérdida accidental de datos. Para más información, consulte [Protección de máquinas virtuales de Azure con un almacén de Recovery Services](/azure/backup/backup-azure-vms-first-look-arm/)

**Habilite registros de diagnóstico**, incluidas las métricas de mantenimiento básicas, los registros de infraestructura y los [diagnósticos de arranque][boot-diagnostics]. Los diagnósticos de arranque pueden ayudarle a diagnosticar un error de arranque si la VM entra en un estado de imposibilidad de arranque. Para más información, consulte [Información general sobre los registros de diagnóstico de Azure][diagnostics-logs].

**Use la extensión AzureLogCollector.** (Solo máquinas virtuales Windows). Esta extensión agrega los registros de la plataforma Azure y los carga en el almacenamiento de Azure, sin que el operador acceda de forma remota a la máquina virtual. Para más información, consulte la [extensión AzureLogCollector](/azure/virtual-machines/virtual-machines-windows-log-collector-extension/?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

## <a name="virtual-network"></a>Virtual Network

**Para permitir o bloquear direcciones IP públicas, agregue un grupo de seguridad de red a la subred.** Bloquee el acceso de usuarios malintencionados o permita el acceso solo a los usuarios que tienen privilegios para acceder a la aplicación.

**Cree un sondeo de estado personalizado.** Los sondeos de estado de Load Balancer pueden probar los protocolos HTTP o TCP. Si una máquina virtual ejecuta un servidor HTTP, el sondeo de HTTP es un indicador del estado de mantenimiento mejor que un sondeo de TCP. Para un sondeo de HTTP, utilice un punto de conexión personalizado que informa del estado general de la aplicación, incluidas todas las dependencias críticas. Para más información, consulte [Información general sobre Azure Load Balancer](/azure/load-balancer/load-balancer-overview/).

**No bloquee el sondeo de estado.** El sondeo de estado de Load Balancer se envía desde una dirección IP conocida, 168.63.129.16. No bloquee el tráfico hacia o desde esta dirección IP en las directivas de firewall o en las reglas de grupo de seguridad de red (NSG). El bloqueo del sondeo de estado podría causar que el equilibrador de carga quite la máquina virtual de la rotación.

**Habilite el registro de Load Balancer.** Los registros muestran cuántas máquinas virtuales del back-end no reciben tráfico de red debido a las respuestas de sondeo con error. Para más información, consulte [Log Analytics para Azure Load Balancer](/azure/load-balancer/load-balancer-monitor-log/).

<!-- links -->
[boot-diagnostics]: https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/
[diagnostics-logs]: /azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs/
[managed-disks]: /azure/storage/storage-managed-disks-overview
[search-optimization]: /azure/search/search-performance-optimization/
[site-recovery]: /azure/site-recovery/
[site-recovery-test]: /azure/site-recovery/site-recovery-test-failover-to-azure
[sql-backup]: /azure/sql-database/sql-database-automated-backups/
[sql-restore]: /azure/sql-database/sql-database-recovery-using-backups/
[vm-manage-availability]: /azure/virtual-machines/windows/manage-availability#use-managed-disks-for-vms-in-an-availability-set
