---
title: "Recuperación de pérdidas de una región de Azure"
description: "Artículo para entender y diseñar aplicaciones resistentes, con alta disponibilidad y con tolerancia a errores y para planear la recuperación ante desastres."
author: adamglick
ms.date: 08/18/2016
ms.openlocfilehash: 42a7d865e101b43279f3198f3dd75df1b15a8565
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2017
---
[!INCLUDE [header](../_includes/header.md)]
# <a name="azure-resiliency-technical-guidance-recovery-from-a-region-wide-service-disruption"></a>Guía técnica sobre resistencia en Azure: recuperación ante una interrupción del servicio en toda la región
Azure se divide física y lógicamente en unidades denominadas regiones. Una región consta de uno o varios centros de datos muy cercanos. 

En raras ocasiones es posible que no se pueda acceder a las instalaciones de toda una región debido, por ejemplo, a errores en la red. O las instalaciones pueden perderse por completo debido, por ejemplo, a un desastre natural. Esta sección explica las funcionalidades de Azure para crear aplicaciones que se distribuyen entre diferentes regiones. Esta distribución está diseñada para minimizar la posibilidad de que un error en una región pueda afectar a otras.

## <a name="cloud-services"></a>Servicios en la nube
### <a name="resource-management"></a>Administración de recursos
Las instancias de proceso se pueden distribuir entre las regiones creando un servicio en la nube independiente en cada región de destino y publicando luego el paquete de implementación en cada servicio en la nube. Sin embargo, tenga en cuenta que la distribución del tráfico entre los servicios en la nube de distintas regiones debe implementarlo el desarrollador de la aplicación o puede implementarse mediante un servicio de administración de tráfico.

Determinar el número de instancias de rol de reserva que debe implementar de antemano para la recuperación ante desastres es un aspecto importante del planeamiento de la capacidad. La existencia de una implementación secundaria a escala completa garantiza que habrá capacidad disponible cuando sea necesario. Sin embargo, esto duplica realmente el costo. Un patrón común consiste en tener una pequeña implementación secundaria lo suficientemente grande como para ejecutar los servicios esenciales. Esta pequeña implementación secundaria es recomendable tanto para tener capacidad reservada como para probar la configuración del entorno secundario.

> [!NOTE]
> La cuota de suscripción no es una garantía de capacidad. La cuota es simplemente un límite de crédito. Para garantizar la capacidad se tiene que definir el número necesario de roles en el modelo de servicio y dichos roles se tienen que implementar.
> 
> 

### <a name="load-balancing"></a>Equilibrio de carga
Para equilibrar la carga de tráfico entre regiones es necesaria una solución de administración de tráfico. Azure proporciona el [Administrador de tráfico de Azure](https://azure.microsoft.com/services/traffic-manager/). También puede utilizar los servicios de terceros que proporcionan funcionalidades de administración de tráfico similares.

### <a name="strategies"></a>Estrategias
Existen muchas estrategias alternativas para implementar el proceso distribuido entre regiones. Estas se deben adaptar a los requisitos empresariales específicos y a las circunstancias de la aplicación. En un nivel alto, los enfoques pueden dividirse en las siguientes categorías:

* **Volver a implementar en caso de desastre**: en este enfoque, la aplicación se vuelve a implementar desde cero en el momento del desastre. Esto es adecuado para aplicaciones que no son esenciales y que no requieren un tiempo de recuperación garantizado.
* **Reserva semiactiva (activa/pasiva)**: se crea un servicio hospedado secundario en una región alternativa y se implementan los roles para garantizar una capacidad mínima; sin embargo, los roles no reciben el tráfico de producción. Este enfoque es útil para las aplicaciones que no se han diseñado para distribuir el tráfico entre las regiones.
* **Reserva activa (activa/activa)**: la aplicación está diseñada para recibir la carga de producción en varias regiones. Los servicios en la nube de cada región pueden configurarse para una capacidad mayor de la necesaria para fines de recuperación ante desastres. Como alternativa, se podrían escalar horizontalmente los servicios en la nube según sea necesario en el momento de un desastre y una conmutación por error. Este enfoque requiere una inversión sustancial en el diseño de la aplicación, pero tiene ventajas significativas. Estas incluyen un tiempo de recuperación bajo y garantizado, la comprobación continua de todas las ubicaciones de recuperación y un uso eficiente de la capacidad.

Un análisis completo del diseño distribuido está fuera del ámbito de este documento. Para más información, consulte [Recuperación ante desastres y alta disponibilidad para aplicaciones creadas en Microsoft Azure](https://aka.ms/drtechguide).

## <a name="virtual-machines"></a>Máquinas virtuales
La recuperación de máquinas virtuales (VMs) de infraestructura como servicio (IaaS) es similar en muchos aspectos a la recuperación de procesos de plataforma como servicio (PaaS). Sin embargo, hay diferencias importantes debido al hecho de que una máquina virtual de IaaS consta de la máquina virtual y el disco de esta.

* **Use Azure Backup para crear copias de seguridad entre regiones que sean coherentes con la aplicación**.
  [Azure Backup](https://azure.microsoft.com/services/backup/) permite a los clientes crear copias de seguridad coherentes con la aplicación en varios discos de máquinas virtuales y admite la replicación de tales copias entre regiones. Para ello elija replicar geográficamente el almacén de copia de seguridad en el momento de la creación. Tenga en cuenta que la replicación del almacén de copia de seguridad se tiene que configurar en el momento de la creación. Más adelante no será posible establecerla. Si se pierde una región, Microsoft pondrá las copias de seguridad a disponibilidad de los clientes. Los clientes podrán restaurar a cualquiera de sus puntos de restauración configurados.
* **Separe el disco de datos del disco del sistema operativo**. Una consideración importante para las máquinas virtuales de IaaS es que no se puede cambiar el disco del sistema operativo sin volver a crear la máquina virtual. Esto no supone ningún problema si su estrategia de recuperación es volver a implementar después de los desastres. Sin embargo, podría ser un problema si utiliza el método de espera semiactiva para capacidad reservada. Para implementar esto correctamente tiene que tener el disco del sistema operativo correcto implementado en las ubicaciones principal y secundaria, y los datos de la aplicación tienen que estar almacenados en una unidad independiente. Si es posible, utilice una configuración estándar de sistema operativo que se pueda proporcionar en ambas ubicaciones. Después de una conmutación por error, tiene que adjuntar la unidad de datos a las máquinas virtuales de IaaS existentes en el controlador de dominio secundario. Use AzCopy para copiar instantáneas de los discos de datos en un sitio remoto.
* **Tenga en cuenta los posibles problemas de coherencia después de una conmutación por error geográfica de varios discos de máquina virtual**. Los discos de máquinas virtuales se implementan como blobs de Azure Storage y tienen la misma característica de replicación geográfica. A menos que utilice [Azure Backup](https://azure.microsoft.com/services/backup/) no hay garantía de coherencia entre los discos, porque la replicación geográfica es asincrónica y se replica de forma independiente. Se garantiza que los discos de máquinas virtuales individuales se encontrarán en un estado de coherencia frente a bloqueos después de una conmutación por error geográfica, pero no se garantiza la coherencia entre los diferentes discos. Esto podría producir problemas en algunos casos (por ejemplo, en los casos de seccionamiento de discos).

## <a name="storage"></a>Storage
### <a name="recovery-by-using-geo-redundant-storage-of-blob-table-queue-and-vm-disk-storage"></a>Recuperación mediante almacenamiento con redundancia geográfica de blobs, tablas, colas y almacenamiento de disco de máquinas virtuales
En Azure, todos los blobs, tablas, colas y discos de máquinas virtuales se replican geográficamente de forma predeterminada. Esto se conoce como almacenamiento con redundancia geográfica (GRS). El almacenamiento con redundancia geográfica replica los datos de almacenamiento en un centro de datos emparejado a cientos de kilómetros de distancia dentro de una región geográfica concreta. El almacenamiento con redundancia geográfica (GRS) está diseñado para proporcionar durabilidad adicional si se produce un desastre en el centro de datos principal. Microsoft controla cuándo se produce la conmutación por error y esta se limita a casos de desastres importantes en los que la ubicación principal original se considera irrecuperable en un período de tiempo razonable. En algunos escenarios este periodo puede ser de varios días. Normalmente, los datos se replican en unos minutos, aunque el intervalo de sincronización aún no lo cubre el contrato de nivel de servicio.

En el caso de una conmutación geográfica, no habrá ningún cambio en la forma en la que se tiene acceso a la cuenta (la dirección URL y la clave no cambiarán). La cuenta de almacenamiento, sin embargo, estará en una región distinta después de la conmutación por error. Esto puede afectar a las aplicaciones que requieren afinidad regional con su cuenta de almacenamiento. Incluso para aquellos servicios y aplicaciones que no requieren una cuenta de almacenamiento en el mismo centro de datos, los gastos por ancho de banda y latencia entre centros de datos pueden ser razones de peso para mover el tráfico temporalmente a la región de conmutación por error. Esto podría tenerse en cuenta a la hora de crear una estrategia general de recuperación ante desastres.

Además de la conmutación por error automática proporcionada por GRS, Azure ha introducido un servicio que le proporciona acceso de lectura a la copia de los datos en la ubicación de almacenamiento secundaria. Este se denomina Almacenamiento con redundancia geográfica con acceso de lectura (RA-GRS).

Para más información sobre GRS y RA-GRS, consulte [Replicación de Azure Storage](/azure/storage/storage-redundancy/).

### <a name="geo-replication-region-mappings"></a>Asignaciones de regiones de replicación geográfica:
Es importante saber dónde se replican geográficamente los datos para saber dónde implementar las otras instancias de los datos que requieren afinidad regional con el almacenamiento. Para más información, vea [Azure Paired Regions](/azure/best-practices-availability-paired-regions) (Regiones emparejadas de Azure).

### <a name="geo-replication-pricing"></a>Precios de la replicación geográfica:
La replicación geográfica está incluida en el precio actual de Azure Storage. Se denomina almacenamiento con redundancia geográfica (GRS). Si no desea que los datos se repliquen geográficamente puede deshabilitar la replicación geográfica para su cuenta. Esto se denomina almacenamiento con redundancia local y se cobra a un precio reducido con respecto al almacenamiento con redundancia geográfica.

### <a name="determining-if-a-geo-failover-has-occurred"></a>Determinación de si se ha producido una conmutación por error geográfica
Si se produce una conmutación por error geográfica, se publicará en el [panel de estado del servicio de Azure](https://azure.microsoft.com/status/). Las aplicaciones pueden implementar medios automatizados para detectarlo mediante la supervisión de la región geográfica para su cuenta de almacenamiento. Esto se puede utilizar para desencadenar otras operaciones de recuperación como la activación de recursos de proceso en la región geográfica a la que se movió el almacenamiento. Puede realizar una consulta desde Service Management API mediante la [obtención de propiedades de la cuenta de almacenamiento](https://msdn.microsoft.com/library/ee460802.aspx). Las propiedades pertinentes son:

    <GeoPrimaryRegion>primary-region</GeoPrimaryRegion>
    <StatusOfPrimary>[Available|Unavailable]</StatusOfPrimary>
    <LastGeoFailoverTime>DateTime</LastGeoFailoverTime>
    <GeoSecondaryRegion>secondary-region</GeoSecondaryRegion>
    <StatusOfSecondary>[Available|Unavailable]</StatusOfSecondary>

### <a name="vm-disks-and-geo-failover"></a>Discos de máquinas virtuales y conmutación por error geográfica
Como se describe en la sección sobre discos de máquinas virtuales, no hay garantías de coherencia de los datos entre discos de máquinas virtuales después de una conmutación por error. Para garantizar la exactitud de las copias de seguridad, se debe utilizar un producto como Data Protection Manager para realizar una copia de seguridad y restaurar los datos de la aplicación.

## <a name="database"></a>Base de datos
### <a name="sql-database"></a>SQL Database
Azure SQL Database proporciona dos tipos de recuperación, la restauración geográfica y la replicación geográfica activa.

#### <a name="geo-restore"></a>Restauración geográfica
La funcionalidad de [restauración geográfica](/azure/sql-database/sql-database-recovery-using-backups/#geo-restore) también está disponible en las bases de datos básicas, estándares y premium. Esta restauración proporciona la opción de recuperación predeterminada cuando la base de datos no está disponible debido a una incidencia en la región en la que se hospeda la base de datos. Al igual que la restauración a un momento dado, la restauración geográfica se basa en copias de seguridad de la base de datos en el Almacenamiento de Azure con redundancia geográfica. Realiza la restauración a partir de la copia de seguridad de replicación geográfica. Por lo tanto es resistente a las interrupciones de almacenamiento en la región principal. Consulte [Restauración de Azure SQL Database o una conmutación por error en una secundaria](/azure/sql-database/sql-database-disaster-recovery/).

#### <a name="active-geo-replication"></a>Replicación geográfica activa
[Replicación geográfica activa](/azure/sql-database/sql-database-geo-replication-overview/) está disponible para todos los niveles de base de datos. Está diseñada para aplicaciones que tienen unos requisitos de recuperación más exigentes que los que puede ofrecer la restauración geográfica. Con la replicación geográfica activa, puede crear hasta cuatro bases de datos secundarias legibles en servidores situados en regiones diferentes. Puede iniciar la conmutación por error a cualquiera de las bases de datos secundarias. Además, la replicación geográfica activa puede utilizarse para los escenarios de actualización o reubicación de la aplicación, así como para el equilibrio de cargas de trabajo de solo lectura. Para obtener más información, consulte el artículo sobre cómo [configurar la replicación geográfica](/azure/sql-database/sql-database-geo-replication-portal/) y [conmutar por error a la base de datos secundaria](/azure/sql-database/sql-database-geo-replication-failover-portal/). Consulte [Diseño de una aplicación para la recuperación ante desastres en la nube mediante replicación geográfica activa en SQL Database](/azure/sql-database/sql-database-designing-cloud-solutions-for-disaster-recovery/) y [Administración de actualizaciones graduales de aplicaciones en la nube mediante la replicación geográfica activa de SQL Database](/azure/sql-database/sql-database-manage-application-rolling-upgrade/) para obtener información sobre cómo diseñar e implementar aplicaciones y actualizaciones de aplicaciones sin que se produzcan tiempos de inactividad.

### <a name="sql-server-on-virtual-machines"></a>SQL Server en máquinas virtuales
Hay varias opciones disponibles de recuperación y alta disponibilidad para SQL Server 2012 (y versiones posteriores) que se ejecutan en Azure Virtual Machines. Para más información, consulte [Alta disponibilidad y recuperación ante desastres para SQL Server en Azure Virtual Machines](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-high-availability-dr/).

## <a name="other-azure-platform-services"></a>Otros servicios de la plataforma de Azure
Al intentar ejecutar el servicio en la nube en varias regiones de Azure, debe tener en cuenta las implicaciones de cada una de las dependencias. En las secciones siguientes, la guía específica del servicio supone que debe usar el mismo servicio de Azure en un centro de datos de Azure alternativo. Esto implica tareas de configuración y de replicación de datos.

> [!NOTE]
> En algunos casos, estos pasos pueden ayudar a mitigar una interrupción específica del servicio en lugar de un evento de centro de datos completo. Desde la perspectiva de la aplicación, una interrupción específica del servicio puede ser igual de restrictiva y requerir la migración temporal del servicio en una región de Azure alternativa.
> 
> 

### <a name="service-bus"></a>Service Bus
Azure Service Bus usa un espacio de nombres único que no abarca varias regiones de Azure. El primer requisito es configurar los espacios de nombres necesarios del bus de servicio en la región alternativa. Sin embargo, también hay consideraciones sobre la durabilidad de los mensajes en cola. Existen varias estrategias para replicar mensajes entre regiones de Azure. Si desea información detallada sobre estas estrategias de replicación y otras estrategias de recuperación ante desastres, consulte [Procedimientos recomendados para aislar aplicaciones ante desastres e interrupciones de Service Bus](/azure/service-bus-messaging/service-bus-outages-disasters/). Para otras consideraciones sobre disponibilidad, consulte [Service Bus (disponibilidad)](recovery-local-failures.md#other-azure-platform-services).

### <a name="app-service"></a>App Service
Para migrar una aplicación de Azure App Service, como Web Apps o Mobile Apps, a una región de Azure secundaria, debe tener una copia de seguridad del sitio web disponible para su publicación. Si la interrupción no afecta al centro de datos de Azure completo, es posible utilizar una FTP para descargar una copia de seguridad reciente del contenido del sitio. A continuación, cree una nueva aplicación web en la región alternativa, a menos que ya lo haya hecho previamente para reservar capacidad. Publique el sitio en la nueva región y realice los cambios de configuración necesarios. Estos cambios podrían incluir cadenas de conexión de base de datos u otra configuración específica de la región. Si es necesario, agregue el certificado SSL del sitio y cambie el registro DNS CNAME para que el nombre de dominio personalizado señale a la dirección URL de la aplicación web de Azure reimplementada.

### <a name="hdinsight"></a>HDInsight
Los datos asociados a HDInsight se almacenan de forma predeterminada en el Azure Blob Storage. HDInsight requiere que un clúster de Hadoop que esté procesando trabajos de MapReduce esté ubicado en la misma región que la cuenta de almacenamiento que contiene los datos analizados. Si utiliza la característica de replicación geográfica disponible en Azure Storage, podrá acceder a datos en la región secundaria en la que estos se replicaron si por algún motivo la región principal ya no está disponible. Puede crear un nuevo clúster de Hadoop en la región en la que se han replicado los datos y seguir procesándolo. Para otras consideraciones sobre disponibilidad, consulte [HDInsight (disponibilidad)](recovery-local-failures.md#other-azure-platform-services).

### <a name="sql-reporting"></a>SQL Reporting
En la actualidad, la recuperación de la pérdida de una región de Azure requiere varias instancias de SQL Reporting en diferentes regiones de Azure. Estas instancias de SQL Reporting deben tener acceso a los mismos datos, y estos deben tener su propio plan de recuperación en caso de desastre. También puede mantener copias de seguridad externas del archivo RDL para cada informe.

### <a name="media-services"></a>Media Services
Azure Media Services tiene un enfoque de recuperación diferente para la codificación y el streaming. Normalmente, el streaming es más importante durante un apagón regional. Para prepararse para esto, debe tener una cuenta de Media Services en dos regiones distintas de Azure. El contenido codificado debe estar ubicado en ambas regiones. Durante un error, puede redirigir el tráfico de streaming a la región alternativa. La codificación puede realizarse en cualquier región de Azure. Si la codificación está sujeta a limitación temporal como, por ejemplo, durante el procesamiento de eventos en directo, debe estar preparado para enviar trabajos a un centro de datos alternativo durante los errores.

### <a name="virtual-network"></a>Red virtual
Los archivos de configuración proporcionan la manera más rápida de configurar una red virtual en otra región de Azure. Después de configurar la red virtual en la región principal de Azure, [exporte la configuración de red virtual](/azure/virtual-network/virtual-networks-create-vnet-classic-portal/) para la red actual a un archivo de configuración de red. En el caso de una interrupción en la región principal, [restaure la red virtual](/azure/virtual-network/virtual-networks-create-vnet-classic-portal/) a partir del archivo de configuración almacenado. Después, configure otros servicios en la nube, máquinas virtuales o configuración entre locales para que funcionen con la nueva red virtual.

## <a name="checklists-for-disaster-recovery"></a>Listas de comprobación para la recuperación ante desastres.
## <a name="cloud-services-checklist"></a>Lista de comprobación de Cloud Services
1. Revise la sección Cloud Services de este documento.
2. Cree una estrategia de recuperación ante desastres entre regiones.
3. Comprenda las ventajas e inconvenientes de reservar capacidad en regiones alternativas.
4. Use herramientas de enrutamiento de tráfico, como el Administrador de tráfico de Azure.

## <a name="virtual-machines-checklist"></a>Lista de comprobación de Máquinas virtuales
1. Revise la sección Máquinas virtuales de este documento.
2. Utilice [Azure Backup](https://azure.microsoft.com/services/backup/) para crear copias de seguridad de la aplicación coherentes entre regiones.

## <a name="storage-checklist"></a>Lista de comprobación de almacenamiento
1. Revise la sección Almacenamiento de este documento.
2. No deshabilite la replicación geográfica de los recursos de almacenamiento.
3. Conozca la región alternativa para la replicación geográfica en caso de conmutación por error.
4. Cree estrategias de copia de seguridad personalizadas para las estrategias de conmutación por error controladas por el usuario.

## <a name="sql-database-checklist"></a>Lista de comprobación de SQL Database
1. Revise la sección SQL Database de este documento.
2. Utilice la [restauración geográfica](/azure/sql-database/sql-database-recovery-using-backups/#geo-restore) o la [replicación geográfica](/azure/sql-database/sql-database-geo-replication-overview/) según corresponda.

## <a name="sql-server-on-virtual-machines-checklist"></a>Lista de comprobación de SQL Server en Máquinas virtuales
1. Revise la sección SQL Server en máquinas virtuales de este documento.
2. Use grupos de disponibilidad AlwaysOn entre regiones o creación de reflejo de base de datos.
3. También puede utilizar copia de seguridad y restaurar en el almacenamiento de blobs.

## <a name="service-bus-checklist"></a>Lista de comprobación de Service Bus
1. Revise la sección Service Bus de este documento.
2. Configure un espacio de nombres de Service Bus en una región alternativa.
3. Considere las estrategias de replicación personalizadas para los mensajes entre regiones.

## <a name="app-service-checklist"></a>Lista de comprobación de App Service
1. Revise la sección App Service de este documento.
2. Mantenga copias de seguridad del sitio fuera de la región primaria.
3. Si la interrupción es parcial, intente recuperar el sitio actual con FTP.
4. Planee la implementación del sitio en un sitio web nuevo o ya existente en una región alternativa.
5. Planee cambios de configuración para los registros de la aplicación y los de DNS CNAME.

## <a name="hdinsight-checklist"></a>Lista de comprobación de HDInsight
1. Revise la sección HDInsight de este documento.
2. Cree un nuevo clúster de Hadoop en la región con los datos replicados.

## <a name="sql-reporting-checklist"></a>Lista de comprobación de SQL Reporting
1. Revise la sección SQL Reporting de este documento.
2. Disponga de una instancia alternativa de SQL Reporting en una región diferente.
3. Mantenga un plan independiente para replicar el destino en esa región.

## <a name="media-services-checklist"></a>Lista de comprobación de Media Services
1. Revise la sección Media Services de este documento.
2. Cree una cuenta de Media Services en una región alternativa.
3. Codifique el mismo contenido en ambas regiones para que admitan la conmutación por error del streaming.
4. Envíe los trabajos de codificación a una región alternativa en caso de una interrupción del servicio.

## <a name="virtual-network-checklist"></a>Lista de comprobación de Virtual Network
1. Revise la sección Virtual Network de este documento.
2. Use la configuración exportada de la red virtual para volver a crearla en otra región.

