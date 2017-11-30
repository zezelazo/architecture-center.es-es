---
title: "Ejecución de máquinas virtuales Linux en varias regiones de Azure para conseguir alta disponibilidad"
description: "Cómo implementar máquinas virtuales en varias regiones de Azure para conseguir alta disponibilidad y resistencia."
author: MikeWasson
ms.date: 11/22/2016
pnp.series.title: Linux VM workloads
pnp.series.prev: n-tier
ms.openlocfilehash: 3b68f6fc79ba4b29e41ba2b04537b834bb8859b0
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2017
---
# <a name="run-linux-vms-in-multiple-regions-for-high-availability"></a>Ejecución de máquinas virtuales Linux en varias regiones para conseguir alta disponibilidad

Esta arquitectura de referencia muestra un conjunto de procedimientos de demostrada eficacia para la ejecución de una aplicación de N niveles en varias regiones de Azure, con la finalidad de conseguir disponibilidad y una sólida estructura de recuperación ante desastres. 

![[0]][0]

*Descargue un [archivo Visio][visio-download] de esta arquitectura.*

## <a name="architecture"></a>Arquitectura 

Esta arquitectura se basa en la que se muestra en [Run Linux VMs for an N-tier application](n-tier.md) (Ejecución de máquinas virtuales Linux para una aplicación de N niveles). 

* **Regiones primarias y secundarias** Use dos regiones para lograr una mayor disponibilidad. Una es la región primaria y la otra para la conmutación por error.
* **Azure Traffic Manager**. [Traffic Manager][traffic-manager] enruta las solicitudes entrantes a una de las regiones. Durante las operaciones normales enruta las solicitudes a la región primaria. Si dicha región no está disponible, Traffic Manager conmuta por error a la región secundaria. Para más información, consulte la sección [Configuración de Traffic Manager](#traffic-manager-configuration).
* **Grupos de recursos**. Cree [grupos de recursos][resource groups] independientes para la región primaria, la región secundaria y para Traffic Manager. De esta manera, obtiene la flexibilidad para administrar cada región como una única colección de recursos. Por ejemplo, podría volver a implementar una región, sin quitar la otra. [Vincule los grupos de recursos][resource-group-links], de modo que pueda ejecutar una consulta para obtener una lista de todos los recursos de la aplicación.
* **Redes virtuales**. Cree una red virtual independiente para cada región. Asegúrese de que los espacios de direcciones no se superpongan.
* **Apache Cassandra**. Implemente Casandra en centros de datos en regiones de Azure para lograr alta disponibilidad. Dentro de cada región, los nodos están configurados en modo compatible con bastidor con dominios de error y actualización, para proporcionar resistencia dentro de la región.

## <a name="recommendations"></a>Recomendaciones

Una arquitectura de varias regiones puede proporcionar una mayor disponibilidad que la implementación en una sola región. Si una interrupción regional afecta a la región primaria, puede usar [Traffic Manager][traffic-manager] para conmutar por error en la región secundaria. Esta arquitectura también puede ayudar si un determinado subsistema de la aplicación produce un error.

Existen varios enfoques generales para lograr una alta disponibilidad en regiones:   

* Activo/pasivo con espera activa. Un tráfico se dirige a una región, mientras el otro se encuentra en espera activa. Espera activa significa que las máquinas virtuales de la región secundaria están siempre asignadas y en funcionamiento.
* Activo/pasivo con espera pasiva. El tráfico se dirige a una región, mientras el otro se encuentra en espera pasiva. Espera pasiva significa que las máquinas virtuales de la región secundaria no se asignan hasta que son necesarias para una conmutación por error. Este enfoque tiene un coste menor de ejecución, pero generalmente tarda más en ponerse en línea durante un error.
* Activo/activo. Ambas regiones están activas y se equilibra la carga de las solicitudes entre ellas. Si una región no está disponible, se saca de la rotación. 

El enfoque de esta arquitectura es activo/pasivo con espera activa, con Traffic Manager para la conmutación por error. Tenga en cuenta que podría implementar un número pequeño de máquinas virtuales para la espera activa y luego escalarlas horizontalmente si es necesario.


### <a name="regional-pairing"></a>Emparejamiento regional

Cada región de Azure se empareja con otra región de la misma zona geográfica. En general, elija regiones del mismo par de regional (por ejemplo, Este de EE. UU. 2 y Centro de EE. UU.). Las ventajas de hacerlo son:

* Si se produce una interrupción prolongada, se establece como prioridad la recuperación de al menos una región de cada par.
* Las actualizaciones planeadas del sistema de Azure se implementan en las regiones emparejadas de manera secuencial para reducir el posible tiempo de inactividad.
* Los pares residen dentro de la misma zona geográfica, de forma que se cumplen los requisitos de residencia de los datos.

Sin embargo, asegúrese de que ambas regiones admitan todos los servicios de Azure que necesita su aplicación (consulte [Servicios por región][services-by-region]). Para más información sobre los pares regionales, consulte [Continuidad empresarial y recuperación ante desastres (BCDR): regiones emparejadas de Azure][regional-pairs].

### <a name="traffic-manager-configuration"></a>Configuración de Traffic Manager

Al configurar Traffic Manager, tenga en cuenta lo siguiente:

* **Enrutamiento**. Traffic Manager admite varios [algoritmos de enrutamiento][tm-routing]. Para el escenario descrito en este artículo, use el enrutamiento de *prioridad* (anteriormente conocido como enrutamiento de *conmutación por error*). Con esta configuración, Traffic Manager envía todas las solicitudes a la región primaria, a no ser que no sea posible comunicarse con ella. En ese momento, conmuta por error automáticamente a la región secundaria. Consulte [Configuración del método de conmutación por error][tm-configure-failover].
* **Sondeo de mantenimiento**. Traffic Manager usa un [sondeo][tm-monitoring] HTTP (o HTTPS) para supervisar la disponibilidad de cada región. El sondeo comprueba si hay una respuesta HTTP 200 para una ruta de acceso de dirección URL especificada. Como procedimiento recomendado, cree un punto de conexión que indique el estado general de la aplicación y úselo para el sondeo de estado. En caso contrario, el sondeo podría informar de un punto de conexión correcto cuando realmente se producen errores en partes críticas de la aplicación. Para más información, consulte [Health Endpoint Monitoring Pattern][health-endpoint-monitoring-pattern] (Patrón Health Endpoint Monitoring).

Cuando Traffic Manager conmuta por error, hay un período de tiempo en que los clientes no pueden comunicarse con la aplicación. La duración viene determinada por los siguientes factores:

* El sondeo de estado debe detectar que la región primaria se ha vuelto inaccesible.
* Los servidores DNS deben actualizar los registros DNS almacenados en caché con la dirección IP, lo cual depende del período de vida (TTL) de DNS. El TTL predeterminado es de 300 segundos (5 minutos), pero puede configurar este valor al crear el perfil de Traffic Manager.

Para más información, consulte [Acerca de la supervisión de Traffic Manager][tm-monitoring].

Si Traffic Manager conmuta por error, se recomienda realizar una conmutación por recuperación manual en lugar de implementar una automática. En caso contrario, puede crear una situación donde la aplicación va y viene incesantemente entre regiones. Compruebe que todos los subsistemas de aplicación tengan un estado correcto antes de la conmutación por recuperación.

Tenga en cuenta que Traffic Manager conmuta por recuperación automáticamente de forma predeterminada. Para evitar esto, reduzca manualmente la prioridad de la región primaria después de un evento de conmutación por error. Por ejemplo, suponga que la región primaria tiene la prioridad 1 y la secundaria la prioridad 2. Después de una conmutación por error, establezca la región primaria en la prioridad 3 para evitar la conmutación por recuperación automática. Cuando esté listo para cambiar de nuevo, actualice la prioridad a 1.

El siguiente comando de la [CLI de Azure][install-azure-cli] actualiza la prioridad:

```bat
azure network traffic-manager  endpoint set --resource-group <resource-group> --profile-name <profile>
    --name <traffic-manager-name> --type AzureEndpoints --priority 3
```    

Otro enfoque consiste en deshabilitar temporalmente el punto de conexión hasta que esté listo para conmutar por recuperación:

```bat
azure network traffic-manager  endpoint set --resource-group <resource-group> --profile-name <profile>
    --name <traffic-manager-name> --type AzureEndpoints --status Disabled
```    

Dependiendo de la causa de una conmutación por error, tendrá que volver a implementar los recursos dentro de una región. Antes de proceder a la conmutación por recuperación, realice una prueba de preparación operativa. La prueba debe comprobar aspectos como:

* Máquinas virtuales configuradas correctamente. (Todo el software necesario está instalado, IIS está en funcionamiento, etc.).
* El estado de los subsistemas de aplicación es correcto.
* Pruebas funcionales. (Por ejemplo, la capa de base de datos es accesible desde la capa web).

### <a name="cassandra-deployment-across-multiple-regions"></a>Implementación de Cassandra en varias regiones

Los centros de datos de Cassandra son un grupo de nodos de datos relacionados que se configuran juntos dentro de un clúster para la replicación y la segregación de la carga de trabajo.

Se recomienda [DataStax Enterprise][datastax] para su uso en producción. Para más información sobre cómo ejecutar DataStax en Azure, consulte [DataStax Enterprise Deployment Guide for Azure][cassandra-in-azure] (Guía de implementación de DataStax Enterprise Deployment para Azure). Las siguientes recomendaciones generales se aplican a cualquier edición de Cassandra: 

* Asigne una dirección IP pública a cada nodo. De esta manera, los clústeres se pueden comunicar entre regiones mediante la infraestructura base de Azure, lo que proporciona un alto rendimiento a un coste bajo.
* Proteja los nodos con las configuraciones adecuadas de firewall y grupo de seguridad de red (NSG) para permitir solo el tráfico que entra y sale de hosts conocidos, incluidos los clientes y otros nodos de clúster. Tenga en cuenta que Cassandra usa distintos puertos para la comunicación, como OpsCenter, Spark, etc. Para saber cómo usar los puertos en Cassandra, consulte [Configuring firewall port access][cassandra-ports] (Configuración del acceso a los puertos del firewall).
* Use el cifrado SSL en todas las comunicaciones de [cliente a nodo][ssl-client-node] y de [nodo a nodo][ssl-node-node].
* Dentro de una región, siga las [recomendaciones de Cassandra](n-tier.md#cassandra).

## <a name="availability-considerations"></a>Consideraciones sobre disponibilidad

Con una aplicación de N niveles compleja, no es necesario replicar toda la aplicación en la región secundaria. En su lugar, puede replicar simplemente un subsistema crítico que sea necesario para permitir la continuidad empresarial.

Traffic Manager es un posible punto de error en el sistema. Si se produce un error en Traffic Manager, los clientes no podrán acceder a la aplicación durante el tiempo de inactividad. Revise el [SLA de Traffic Manager][tm-sla] y determine si el uso de Traffic Manager por sí solo cumple sus requisitos empresariales de alta disponibilidad. Si no es así, considere la posibilidad de agregar otra solución de administración de tráfico como una conmutación por recuperación. Si el servicio Azure Traffic Manager no funciona, cambie los registros CNAME en DNS para que apunten a otro servicio de administración del tráfico. (Este paso debe realizarse manualmente, y la aplicación dejará de estar disponible hasta que se propaguen los cambios de DNS).

En un clúster de Cassandra. los escenarios de conmutación por error que se deben tener en cuenta dependen de los niveles de coherencia usados por la aplicación, así como del número de réplicas utilizadas. Para más información sobre los niveles de coherencia y el uso en Cassandra, consulte [Configuring data consistency][cassandra-consistency] (Configuración de la coherencia de datos) y [Cassandra: How many nodes are talked to with Quorum?][cassandra-consistency-usage] (Cassandra: ¿Con cuántos nodos se habla con cuórum?) La disponibilidad de datos en Cassandra viene determinada por el nivel de coherencia usado por la aplicación y el mecanismo de replicación. Para más información sobre la replicación en Cassandra, consulte [Data Replication in NoSQL Databases Explained][cassandra-replication] (Explicación de la replicación de datos en bases de datos NoSQL).

## <a name="manageability-considerations"></a>Consideraciones sobre la manejabilidad

Al actualizar la implementación, actualice una región cada vez para reducir la probabilidad de un error global derivado de una configuración incorrecta o un error en la aplicación.

Pruebe la resistencia del sistema a los errores. Estos son algunos escenarios comunes de error que se pueden probar:

* Apagado de las instancias de máquina virtual.
* Recursos de presión, como CPU y memoria.
* Desconexión o retraso de la red.
* Bloqueo de procesos.
* Caducidad de certificados.
* Simulación de errores de hardware.
* Apagado del servicio DNS en los controladores de dominio.

Medición de los tiempos de recuperación y comprobación de que cumplen los requisitos empresariales. Pruebe también combinaciones de modos de error.


<!-- Links -->
[hybrid-vpn]: ../hybrid-networking/vpn.md

[cassandra-in-azure]: https://academy.datastax.com/resources/deployment-guide-azure
[cassandra-consistency]: http://docs.datastax.com/en/cassandra/2.0/cassandra/dml/dml_config_consistency_c.html
[cassandra-replication]: http://www.planetcassandra.org/data-replication-in-nosql-databases-explained/
[cassandra-consistency-usage]: https://medium.com/@foundev/cassandra-how-many-nodes-are-talked-to-with-quorum-also-should-i-use-it-98074e75d7d5#.b4pb4alb2
[cassandra-ports]: https://docs.datastax.com/en/datastax_enterprise/5.0/datastax_enterprise/sec/configFirewallPorts.html
[datastax]: https://www.datastax.com/products/datastax-enterprise
[health-endpoint-monitoring-pattern]: https://msdn.microsoft.com/library/dn589789.aspx
[install-azure-cli]: /azure/xplat-cli-install
[regional-pairs]: /azure/best-practices-availability-paired-regions
[resource groups]: /azure/azure-resource-manager/resource-group-overview
[resource-group-links]: /azure/resource-group-link-resources
[services-by-region]: https://azure.microsoft.com/regions/#services
[ssl-client-node]: http://docs.datastax.com/en/cassandra/2.0/cassandra/security/secureSSLClientToNode_t.html
[ssl-node-node]: http://docs.datastax.com/en/cassandra/2.0/cassandra/security/secureSSLNodeToNode_t.html
[tablediff]: https://msdn.microsoft.com/library/ms162843.aspx
[tm-configure-failover]: /azure/traffic-manager/traffic-manager-configure-failover-routing-method
[tm-monitoring]: /azure/traffic-manager/traffic-manager-monitoring
[tm-routing]: /azure/traffic-manager/traffic-manager-routing-methods
[tm-sla]: https://azure.microsoft.com/support/legal/sla/traffic-manager/v1_0/
[traffic-manager]: https://azure.microsoft.com/services/traffic-manager/
[visio-download]: https://archcenter.azureedge.net/cdn/vm-reference-architectures.vsdx
[wsfc]: https://msdn.microsoft.com/library/hh270278.aspx
[0]: ./images/multi-region-application-diagram.png "Arquitectura de red de alta disponibilidad para aplicaciones de N niveles en Azure"
