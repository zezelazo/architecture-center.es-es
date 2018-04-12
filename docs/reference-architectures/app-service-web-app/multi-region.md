---
title: Aplicación web de varias regiones
description: Arquitectura recomendada para aplicaciones web con alta disponibilidad que se ejecutan en Microsoft Azure.
author: MikeWasson
ms.date: 11/23/2016
cardTitle: Run in multiple regions
ms.openlocfilehash: 00309e58c163a64f6d9796bedc19d936afcd09ab
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2018
---
# <a name="run-a-web-application-in-multiple-regions"></a>Ejecución de una aplicación web en varias regiones
[!INCLUDE [header](../../_includes/header.md)]

Esta arquitectura de referencia muestra cómo ejecutar una aplicación de Azure App Service en varias regiones para lograr una alta disponibilidad. 

![Arquitectura de referencia de Azure: aplicación web con alta disponibilidad](./images/multi-region-web-app-diagram.png) 

*Descargue un [archivo Visio][visio-download] de esta arquitectura.*

## <a name="architecture"></a>Architecture 

Esta arquitectura se basa en la que se muestra en [Improve scalability in a web application][guidance-web-apps-scalability] (Mejora de la escalabilidad en una aplicación web). Las principales diferencias son:

* **Regiones primarias y secundarias** Esta arquitectura emplea dos regiones para lograr una mayor disponibilidad. La aplicación se implementa en cada región. Durante las operaciones normales, el tráfico de red se enruta a la región primaria. Si la región primaria deja de estar disponible, el tráfico se enruta a la región secundaria. 
* **Azure DNS**. [Azure DNS][azure-dns] es un servicio de hospedaje para dominios DNS que permite resolver nombres mediante la infraestructura de Microsoft Azure. Al hospedar dominios en Azure, puede administrar los registros DNS con las mismas credenciales, API, herramientas y facturación que con los demás servicios de Azure.
* **Azure Traffic Manager**. [Traffic Manager][traffic-manager] enruta las solicitudes entrantes a la región primaria. Si la aplicación que se ejecuta en dicha región no está disponible, Traffic Manager conmuta por error a la región secundaria.
* **Replicación geográfica** de SQL Database y Cosmos DB. 

Una arquitectura de varias regiones puede proporcionar una mayor disponibilidad que la implementación en una sola región. Si una interrupción regional afecta a la región primaria, puede usar [Traffic Manager][traffic-manager] para conmutar por error en la región secundaria. Esta arquitectura también puede ayudar si un determinado subsistema de la aplicación produce un error.

Existen varios enfoques generales para lograr una alta disponibilidad en regiones: 

* Activo/pasivo con espera activa. Un tráfico se dirige a una región, mientras el otro se encuentra en espera activa. Espera activa significa que las máquinas virtuales de la región secundaria están siempre asignadas y en funcionamiento.
* Activo/pasivo con espera pasiva. El tráfico se dirige a una región, mientras el otro se encuentra en espera pasiva. Espera pasiva significa que las máquinas virtuales de la región secundaria no se asignan hasta que son necesarias para una conmutación por error. Este enfoque tiene un coste menor de ejecución, pero generalmente tarda más en ponerse en línea durante un error.
* Activo/activo. Ambas regiones están activas y se equilibra la carga de las solicitudes entre ellas. Si una región no está disponible, se saca de la rotación. 

Esta arquitectura de referencia se centra en el enfoque activo/pasivo con espera activa y usa Traffic Manager para la conmutación por error. 


## <a name="recommendations"></a>Recomendaciones

Los requisitos pueden diferir de los de la arquitectura que se describe aquí. Use las recomendaciones de esta sección como punto de partida.

### <a name="regional-pairing"></a>Emparejamiento regional
Cada región de Azure se empareja con otra región de la misma zona geográfica. En general, elija regiones del mismo par de regional (por ejemplo, Este de EE. UU. 2 y Centro de EE. UU.). Las ventajas de hacerlo son:

* Si se produce una interrupción prolongada, se establece como prioridad la recuperación de al menos una región de cada par.
* Las actualizaciones planeadas del sistema de Azure se implementan en las regiones emparejadas de manera secuencial para reducir el posible tiempo de inactividad.
* En la mayoría de los casos, los pares regionales residen en la misma zona geográfica para cumplir los requisitos de residencia de datos.

Sin embargo, asegúrese de que ambas regiones admitan todos los servicios de Azure necesarios para la aplicación. Consulte [Servicios por región][services-by-region]. Para más información sobre los pares regionales, consulte [Continuidad empresarial y recuperación ante desastres (BCDR): regiones emparejadas de Azure][regional-pairs].

### <a name="resource-groups"></a>Grupos de recursos
Considere la posibilidad de colocar la región primaria, la región secundaria y Traffic Manager en diferentes [grupos de recursos][resource groups]. De esta forma, podrá administrar los recursos implementados en cada región como una sola colección.

### <a name="traffic-manager-configuration"></a>Configuración de Traffic Manager 

**Enrutamiento**. Traffic Manager admite varios [algoritmos de enrutamiento][tm-routing]. Para el escenario descrito en este artículo, use el enrutamiento de *prioridad* (anteriormente conocido como enrutamiento de *conmutación por error*). Con esta configuración, Traffic Manager envía todas las solicitudes a la región primaria, a menos que el punto de conexión de esa región se vuelva inaccesible. En ese momento, conmuta por error automáticamente a la región secundaria. Consulte [Configuración del método de conmutación por error][tm-configure-failover].

**Sondeo de mantenimiento**. Traffic Manager usa un sondeo HTTP (o HTTPS) para supervisar la disponibilidad de cada punto de conexión. El sondeo proporciona a Traffic Manager una prueba de acierto/error para conmutar por error a la región secundaria. Funciona mediante el envío de una solicitud a la ruta de acceso de una dirección URL especificada. Si recibe una respuesta distinta de 200 dentro de un período de tiempo de espera, el sondeo produce un error. Después de cuatro solicitudes con error, Traffic Manager marca el punto de conexión como degradado y conmuta por error al otro punto de conexión. Para más información, consulte [Supervisión de puntos de conexión y conmutación por error de Traffic Manager][tm-monitoring].

Como procedimiento recomendado, cree un punto de conexión de sondeo de estado que indique el estado general de la aplicación y úselo para el sondeo de estado. El punto de conexión debe comprobar dependencias críticas, como las aplicaciones de App Service, Storage Queue y SQL Database. En caso contrario, el sondeo podría informar de un punto de conexión correcto cuando realmente se producen errores en partes críticas de la aplicación.

Por otro lado, no use el sondeo de estado para comprobar los servicios de prioridad inferior. Por ejemplo, si un servicio de correo electrónico queda fuera de servicio, la aplicación puede cambiar a un segundo proveedor o simplemente enviar los mensajes de correo electrónico más tarde. Esta no es una prioridad lo suficientemente alta como para provocar que la aplicación conmute por error. Para más información, consulte [Health Endpoint Monitoring Pattern][health-endpoint-monitoring-pattern] (Patrón Health Endpoint Monitoring).
 
### <a name="sql-database"></a>SQL Database
Use la [replicación geográfica activa][sql-replication] para crear una réplica secundaria legible en una región distinta. Puede tener hasta cuatro réplicas secundarias legibles. Conmute por error a una base de datos secundaria si la base de datos principal da error o debe desconectarse. La replicación geográfica activa puede configurarse para cualquier base de datos de cualquier grupo de bases de datos elásticas.

### <a name="cosmos-db"></a>Cosmos DB
Cosmos DB admite la replicación geográfica entre regiones. Una región se designa como de escritura y los demás son réplicas de solo lectura.

Si se produce una interrupción regional del sistema, puede conmutar por error y seleccionar otra región como la región de escritura. El SDK de cliente envía automáticamente las solicitudes de escritura a la región de escritura actual, por lo que no es necesario actualizar la configuración del cliente después de una conmutación por error. Para más información, consulte [Cómo se distribuyen datos globalmente con Azure Cosmos DB][cosmosdb-geo].

> [!NOTE]
> Todas las réplicas pertenecen al mismo grupo de recursos.
>
>

### <a name="storage"></a>Storage
Para Azure Storage, use [almacenamiento con redundancia geográfica con acceso de lectura][ra-grs] (RA-GRS). Con el almacenamiento RA-GRS, los datos se replican en una región secundaria. Tiene acceso de solo lectura a los datos de la región secundaria mediante un punto de conexión independiente. Si se produce una interrupción regional o una situación de desastre, el equipo de Azure Storage puede decidir realizar una conmutación por error geográfica a la región secundaria. En este caso, no es necesario que el cliente intervenga de ninguna forma.

Para Queue Storage, cree una cola de copia de seguridad en la región secundaria. Durante la conmutación por error, la aplicación puede usar la cola de copia de seguridad hasta que la región primaria vuelva a estar disponible. De este modo, la aplicación puede seguir procesando nuevas solicitudes.

## <a name="availability-considerations"></a>Consideraciones sobre disponibilidad


### <a name="traffic-manager"></a>Traffic Manager

Traffic Manager conmuta por error automáticamente si la región primaria deja de estar disponible. Cuando Traffic Manager conmuta por error, hay un período de tiempo en que los clientes no pueden comunicarse con la aplicación. La duración viene determinada por los siguientes factores:

* El sondeo de estado debe detectar que el centro de datos principal se ha vuelto inaccesible.
* Los servidores del servicio de nombres de dominio (DNS) deben actualizar los registros DNS almacenados en caché con la dirección IP, lo cual depende del período de vida (TTL) de DNS. El TTL predeterminado es de 300 segundos (5 minutos), pero puede configurar este valor al crear el perfil de Traffic Manager.

Para más información, consulte [Acerca de la supervisión de Traffic Manager][tm-monitoring].

Traffic Manager es un posible punto de error en el sistema. Si el servicio no funciona, los clientes no pueden acceder a la aplicación durante el tiempo de inactividad. Revise el [Acuerdo de Nivel de Servicio (SLA) de Traffic Manager] [tm-sla] y determine si el uso de Traffic Manager por sí solo cumple sus requisitos empresariales de alta disponibilidad. Si no es así, considere la posibilidad de agregar otra solución de administración de tráfico como conmutación por recuperación. Si el servicio Azure Traffic Manager no funciona, cambie los registros de nombre canónico (CNAME) en DNS para que apunten a otro servicio de administración del tráfico. Este paso debe realizarse manualmente, y la aplicación dejará de estar disponible hasta que se propaguen los cambios de DNS.

### <a name="sql-database"></a>SQL Database
El objetivo de punto de recuperación (RPO) y el tiempo de recuperación estimado (ERT) de SQL Database se documentan en [Introducción a la continuidad empresarial con Azure SQL Database][sql-rpo]. 

### <a name="storage"></a>Storage
RA-GRS proporciona almacenamiento duradero, pero es importante comprender lo que sucede durante una interrupción:

* Si se produce una interrupción en el almacenamiento, habrá un período de tiempo en que no tenga acceso de escritura a los datos. Aún así, puede leer desde el punto de conexión secundario durante la interrupción.
* Si una interrupción regional o una situación de desastre afecta a la ubicación principal y no se pueden recuperar los datos, el equipo de Azure Storage puede decidir realizar una conmutación por error geográfica a la región secundaria.
* La replicación de datos en la región secundaria se realiza de forma asincrónica. Por lo tanto, si se realiza una conmutación por error geográfica, podría haber alguna pérdida de datos si los datos no se pueden recuperar de la región primaria.
* Los errores transitorios, como una interrupción de la red, no desencadenarán una conmutación por error de almacenamiento. Diseñe la aplicación para que sea resistente a los errores transitorios. Posibles remedios:
  
  * Realizar las operaciones de lectura desde la región secundaria.
  * Cambiar temporalmente a otra cuenta de almacenamiento con nuevas operaciones de escritura (por ejemplo, poner en cola los mensajes).
  * Copiar los datos de la región secundaria a otra cuenta de almacenamiento.
  * Proporcionan funcionalidad reducida hasta que el sistema con errores conmute por recuperación.

Para más información, consulte [Qué hacer si se produce una interrupción del servicio Azure Storage][storage-outage].

## <a name="manageability-considerations"></a>Consideraciones sobre manejabilidad

### <a name="traffic-manager"></a>Traffic Manager

Si Traffic Manager conmuta por error, se recomienda realizar una conmutación por recuperación manual en lugar de implementar una automática. En caso contrario, puede crear una situación donde la aplicación va y viene incesantemente entre regiones. Compruebe que todos los subsistemas de aplicación tengan un estado correcto antes de la conmutación por recuperación.

Tenga en cuenta que Traffic Manager conmuta por recuperación automáticamente de forma predeterminada. Para evitar esto, reduzca manualmente la prioridad de la región primaria después de un evento de conmutación por error. Por ejemplo, suponga que la región primaria tiene la prioridad 1 y la secundaria la prioridad 2. Después de una conmutación por error, establezca la región primaria en la prioridad 3 para evitar la conmutación por recuperación automática. Cuando esté listo para cambiar de nuevo, actualice la prioridad a 1.

Los siguientes comandos actualizan la prioridad.

**PowerShell**

```bat
$endpoint = Get-AzureRmTrafficManagerEndpoint -Name <endpoint> -ProfileName <profile> -ResourceGroupName <resource-group> -Type AzureEndpoints
$endpoint.Priority = 3
Set-AzureRmTrafficManagerEndpoint -TrafficManagerEndpoint $endpoint
```

Para más información, consulte [Cmdlets de Azure Traffic Manager][tm-ps].

**Interfaz de la línea de comandos (CLI) de Azure**

```bat
azure network traffic-manager endpoint set --name <endpoint> --profile-name <profile> --resource-group <resource-group> --type AzureEndpoints --priority 3
```    

### <a name="sql-database"></a>SQL Database
Si se produce un error en la base de datos principal, realice una conmutación por error manual a la base de datos secundaria. Consulte [Restauración de una base de datos SQL de Azure o una conmutación por error en una secundaria][sql-failover]. La base de datos secundaria sigue siendo de solo lectura hasta que se realiza la conmutación por error.


<!-- links -->

[azure-sql-db]: https://azure.microsoft.com/documentation/services/sql-database/
[azure-dns]: /azure/dns/dns-overview
[cosmosdb-geo]: /azure/cosmos-db/distribute-data-globally
[guidance-web-apps-scalability]: ./scalable-web-app.md
[health-endpoint-monitoring-pattern]: https://msdn.microsoft.com/library/dn589789.aspx
[ra-grs]: /azure/storage/storage-redundancy#read-access-geo-redundant-storage
[regional-pairs]: /azure/best-practices-availability-paired-regions
[resource groups]: /azure/azure-resource-manager/resource-group-overview#resource-groups
[services-by-region]: https://azure.microsoft.com/regions/#services
[sql-failover]: /azure/sql-database/sql-database-disaster-recovery
[sql-replication]: /azure/sql-database/sql-database-geo-replication-overview
[sql-rpo]: /azure/sql-database/sql-database-business-continuity#sql-database-features-that-you-can-use-to-provide-business-continuity
[storage-outage]: /azure/storage/storage-disaster-recovery-guidance
[tm-configure-failover]: /azure/traffic-manager/traffic-manager-configure-failover-routing-method
[tm-monitoring]: /azure/traffic-manager/traffic-manager-monitoring
[tm-ps]: https://msdn.microsoft.com/library/mt125941.aspx
[tm-routing]: /azure/traffic-manager/traffic-manager-routing-methods
[tm-sla]: https://azure.microsoft.com/support/legal/sla/traffic-manager/v1_0/
[traffic-manager]: https://azure.microsoft.com/services/traffic-manager/
[visio-download]: https://archcenter.blob.core.windows.net/cdn/app-service-reference-architectures.vsdx
