---
title: Aplicación de n niveles con Apache Cassandra
description: Cómo ejecutar máquinas virtuales Linux para una arquitectura de n niveles en Microsoft Azure.
author: MikeWasson
ms.date: 09/13/2018
ms.openlocfilehash: 2eceb0b5d939c0aa2cc9fc3209d0f86449fdd72b
ms.sourcegitcommit: dbbf914757b03cdee7a274204f9579fa63d7eed2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/02/2018
ms.locfileid: "50916573"
---
# <a name="n-tier-application-with-apache-cassandra"></a>Aplicación de n niveles con Apache Cassandra

Esta arquitectura de referencia muestra cómo implementar máquinas virtuales y una red virtual configurada para una aplicación de n niveles, con Apache Cassandra en Linux para la capa de datos. [**Implemente esta solución**.](#deploy-the-solution) 

![[0]][0]

*Descargue un [archivo Visio][visio-download] de esta arquitectura.*

## <a name="architecture"></a>Arquitectura 

La arquitectura consta de los siguientes componentes:

* **Grupo de recursos.** Los [grupos de recursos][resource-manager-overview] se utilizan para agrupar los recursos, para que puedan administrarse según su duración, su propietario u otros criterios.

* **Red virtual y subredes.** Cada máquina virtual de Azure se implementa en una red virtual que se puede dividir en varias subredes. Cree una subred independiente para cada nivel. 

* **Grupos de seguridad de red.** Use [grupos de seguridad de red][nsg] (NSG) para restringir el tráfico de red dentro de la red virtual. Por ejemplo, en la arquitectura de tres niveles que se muestra aquí, el nivel de base de datos no acepta el tráfico desde el front-end web, solo desde el nivel Business y la subred de administración.

* **Máquinas virtuales**. Para obtener recomendaciones sobre la configuración de máquinas virtuales, consulte [Ejecución de una VM con Windows en Azure](./windows-vm.md) y [Ejecución de una VM con Linux en Azure](./linux-vm.md).

* **Conjuntos de disponibilidad**. Cree un [conjunto de disponibilidad][azure-availability-sets] para cada nivel y aprovisione al menos dos máquinas virtuales en cada nivel. Esto hace que las máquinas virtuales sean aptas para un [Acuerdo de Nivel de Servicio (SLA)][vm-sla] mayor. 

* **Conjunto de escalado de máquinas virtuales** (no se muestra). Un [conjunto de escalado de máquinas virtuales][vmss] es una alternativa al uso de un conjunto de disponibilidad. Los conjuntos de escalado facilitan el escalado horizontal de las máquinas virtuales de un nivel, de forma manual o automática, y según reglas predefinidas.

* **Equilibradores de carga de Azure.** Los [equilibradores de carga][load-balancer] distribuyen las solicitudes entrantes de Internet a las instancias de máquina virtual. Use un [equilibrador de carga público][load-balancer-external] para distribuir el tráfico entrante de Internet al nivel Web y un [equilibrador de carga interno][load-balancer-internal] para distribuir el tráfico de red del nivel Web al nivel Business.

* **Dirección IP pública**. Se necesita una dirección IP pública para que el equilibrador de carga reciba tráfico de Internet.

* **JumpBox.** También se denomina [host bastión]. Se trata de una máquina virtual segura en la red que usan los administradores para conectarse al resto de máquinas virtuales. El Jumpbox tiene un NSG que solo permite el tráfico remoto que procede de direcciones IP públicas de una lista segura. El grupo de seguridad de red debe permitir el tráfico SSH.

* **Base de datos Apache Cassandra**. Proporciona alta disponibilidad en el nivel de datos, al habilitar la replicación y la conmutación por error.

* **Azure DNS**. [Azure DNS][azure-dns] es un servicio de hospedaje para dominios DNS que permite resolver nombres mediante la infraestructura de Microsoft Azure. Al hospedar dominios en Azure, puede administrar los registros DNS con las mismas credenciales, API, herramientas y facturación que con los demás servicios de Azure.

## <a name="recommendations"></a>Recomendaciones

Los requisitos pueden diferir de los de la arquitectura que se describe aquí. Use estas recomendaciones como punto inicial. 

### <a name="vnet--subnets"></a>Red virtual/subredes

Cuando cree la red virtual, determine cuántas direcciones IP requieren los recursos de cada subred. Especifique una máscara de subred y un intervalo de direcciones de la red virtual lo suficientemente grande para la dirección IP requerida con el uso de la notación [CIDR]. Use un espacio de direcciones que se encuentre dentro de los [bloques de direcciones IP privados][private-ip-space] estándar, que son 10.0.0.0/8, 172.16.0.0/12 y 192.168.0.0/16.

Elija un intervalo de direcciones que no se superponga con la red local, en caso de que necesite configurar una puerta de enlace entre la red virtual y la red local más adelante. Una vez creada la red virtual, no se puede cambiar el intervalo de direcciones.

Diseñe subredes teniendo en cuenta los requisitos de funcionalidad y seguridad. Todas las máquinas virtuales dentro del mismo nivel o rol deben incluirse en la misma subred, lo que puede servir como un límite de seguridad. Para obtener más información sobre el diseño de redes virtuales y subredes, vea [Planeación y diseño de redes virtuales de Azure][plan-network].

### <a name="load-balancers"></a>Equilibradores de carga

No exponga las máquinas virtuales directamente a Internet; en su lugar, asigne una dirección IP privada a cada máquina virtual. Los clientes se conectan con la dirección IP del equilibrador de carga público.

Defina reglas del equilibrador de carga para dirigir el tráfico de red a las máquinas virtuales. Por ejemplo, para habilitar el tráfico HTTP, cree una regla que asigne el puerto 80 de la configuración de front-end al puerto 80 del grupo de direcciones de back-end. Cuando un cliente envía una solicitud HTTP al puerto 80, el equilibrador de carga selecciona una dirección IP de back-end mediante un [algoritmo hash][load-balancer-hashing] que incluye la dirección IP de origen. De ese modo, las solicitudes de cliente se distribuyen entre todas las máquinas virtuales.

### <a name="network-security-groups"></a>Grupos de seguridad de red

Use reglas NSG para restringir el tráfico entre los niveles. Por ejemplo, en la arquitectura de tres niveles mostrada anteriormente, el nivel Web no se comunica directamente con el nivel de base de datos. Para exigir esto, el nivel de base de datos debe bloquear el tráfico entrante desde la subred del nivel Web.  

1. Deniegue todo el tráfico entrante de la red virtual. (Use la etiqueta `VIRTUAL_NETWORK` de la regla). 
2. Permita el tráfico entrante de la subred del nivel Business.  
3. Permita el tráfico entrante de la propia subred del nivel de la base de datos. Esta regla permite la comunicación entre las máquinas virtuales de la base de datos, lo cual es necesario para la replicación y la conmutación por error de esta.
4. Permita el tráfico SSH (puerto 22) desde la subred de JumpBox. Esta regla permite a los administradores conectarse al nivel de base de datos desde JumpBox.

Cree las reglas 2 &ndash; 4 con una prioridad más alta que la primera regla para que puedan invalidarla.

### <a name="cassandra"></a>Cassandra

Se recomienda [DataStax Enterprise][datastax] para usos de producción, pero estas recomendaciones se aplican a cualquier edición de Cassandra. Para más información sobre cómo ejecutar DataStax en Azure, consulte [DataStax Enterprise Deployment Guide for Azure][cassandra-in-azure] (Guía de implementación de DataStax Enterprise Deployment para Azure). 

Coloque las máquinas virtuales de un clúster de Cassandra en un conjunto de disponibilidad para asegurarse de que las réplicas de Cassandra se distribuyen entre varios dominios de error y dominios de actualización. Para obtener más información sobre los dominios de error y los dominios de actualización, vea [Administración de la disponibilidad de las máquinas virtuales][azure-availability-sets]. 

Configure tres dominios de error como máximo por cada conjunto de disponibilidad y dieciocho dominios de actualización por cada conjunto de disponibilidad. Esto proporciona el número máximo de dominios de actualización que todavía se pueden distribuir uniformemente entre los dominios de error.   

Configure los nodos en el modo de reconocimiento del bastidor. Asigne los dominios de error a los bastidores en el archivo `cassandra-rackdc.properties`.

No se necesita un equilibrador de carga delante del clúster. El cliente se conecta directamente a un nodo del clúster.

Para lograr alta disponibilidad, implemente Cassandra en más de una región de Azure. Dentro de cada región, los nodos están configurados en modo compatible con bastidor con dominios de error y actualización, para proporcionar resistencia dentro de la región.


### <a name="jumpbox"></a>JumpBox

No permita el acceso SSH desde la red pública de Internet a las máquinas virtuales que ejecutan la carga de trabajo de la aplicación. En su lugar, todo el acceso SSH a estas máquinas virtuales debe realizarse a través de JumpBox. Un administrador inicia sesión en JumpBox y, después, en la otra máquina virtual desde JumpBox. JumpBox permite el tráfico SSH desde Internet, pero solo desde direcciones IP conocidas y seguras.

JumpBox tiene unos requisitos de rendimiento mínimos, por lo que puede seleccionar un pequeño tamaño de máquina virtual. Cree una [dirección IP pública] para JumpBox. Coloque JumpBox en la misma red virtual que las demás máquinas virtuales, pero en una subred de administración independiente.

Para proteger JumpBox, agregue una regla de grupo de seguridad de red que permita las conexiones SSH solo desde un conjunto seguro de direcciones IP públicas. Configure el NSG para las demás subredes, a fin de permitir el tráfico SSH de la subred de administración.

## <a name="scalability-considerations"></a>Consideraciones sobre escalabilidad

Los [conjuntos de escalado de máquinas virtuales][vmss] permiten implementar y administrar un conjunto de máquinas virtuales idénticas. Los conjuntos de escalado admiten el escalado automático en función de las métricas de rendimiento. A medida que aumenta la carga en las máquinas virtuales, se agregan más máquinas virtuales automáticamente al equilibrador de carga. Considere la posibilidad de usar conjuntos de escalado si necesita escalar horizontalmente las máquinas virtuales de inmediato o si necesita realizar el escalado automático.

Hay dos maneras básicas de configurar máquinas virtuales implementadas en un conjunto de escalado:

- Use extensiones para configurar la máquina virtual después de aprovisionarla. Con este método, las nuevas instancias de máquina virtual pueden tardar más en iniciarse que una máquina virtual sin extensiones.

- Implemente un [disco administrado](/azure/storage/storage-managed-disks-overview) con una imagen de disco personalizada. Esta opción puede ser más rápida de implementar. Sin embargo, requiere mantener la imagen actualizada.

Para obtener consideraciones adicionales, consulte [Consideraciones de diseño para conjuntos de escalado][vmss-design].

> [!TIP]
> Cuando utilice cualquier solución de escalado automático, pruébela con antelación con cargas de trabajo de nivel de producción.

Cada suscripción de Azure tiene límites predeterminados establecidos, incluido un número máximo de máquinas virtuales por región. Puede aumentar el límite si rellena una solicitud de soporte técnico. Para más información, consulte [Límites, cuotas y restricciones de suscripción y servicios de Microsoft Azure][subscription-limits].

## <a name="availability-considerations"></a>Consideraciones sobre disponibilidad

Si no va a usar conjuntos de escalado de máquinas virtuales, ponga estas en el mismo nivel en un conjunto de disponibilidad. Cree al menos dos máquinas virtuales en el conjunto de disponibilidad, para admitir el [SLA de disponibilidad para máquinas virtuales de Azure][vm-sla]. Para más información, consulte [Administración de la disponibilidad de las máquinas virtuales][availability-set]. 

El equilibrador de carga usa [sondeos de mantenimiento][health-probes] para supervisar la disponibilidad de las instancias de máquina virtual. Si un sondeo no puede conectar con una instancia durante un período de tiempo de espera, el equilibrador de carga deja de enviar tráfico a esa máquina virtual. Sin embargo, el equilibrador de carga continuará con el sondeo y, si la máquina virtual vuelve a estar disponible, el equilibrador de carga reanudará el envío del tráfico a esa máquina virtual.

A continuación, se presentan algunas recomendaciones sobre los sondeos de mantenimiento del equilibrador de carga:

* Los sondeos pueden probar los protocolos HTTP o TCP. Si las máquinas virtuales ejecutan un servidor HTTP, cree un sondeo HTTP. De lo contrario, cree un sondeo TCP.
* Para un sondeo HTTP, especifique la ruta de acceso a un punto de conexión HTTP. El sondeo comprueba si hay una respuesta HTTP 200 desde esta ruta de acceso. Puede ser la ruta de acceso raíz ("/") o un punto de conexión de supervisión de mantenimiento que implementa alguna lógica personalizada para comprobar el mantenimiento de la aplicación. El punto de conexión debe permitir solicitudes HTTP anónimas.
* El sondeo se envía desde una [dirección IP conocida][health-probe-ip], 168.63.129.16. Asegúrese de que no bloquear el tráfico hacia o desde esta dirección IP en las directivas de firewall o en las reglas de grupo de seguridad de red (NSG).
* Use [registros de sondeo de mantenimiento][health-probe-log] para ver el estado de los sondeos de mantenimiento. Habilite el registro en Azure Portal para cada equilibrador de carga. Los registros se escriben en Azure Blob Storage. Los registros muestran cuántas máquinas virtuales del back-end no reciben tráfico de red debido a las respuestas de sondeo con error.

En un clúster de Cassandra. los escenarios de conmutación por error que se deben tener en cuenta dependen de los niveles de coherencia usados por la aplicación, así como del número de réplicas utilizadas. Para más información sobre los niveles de coherencia y el uso en Cassandra, consulte [Configuring data consistency][cassandra-consistency] (Configuración de la coherencia de datos) y [Cassandra: How many nodes are talked to with Quorum?][cassandra-consistency-usage] (Cassandra: ¿Con cuántos nodos se habla con cuórum?) La disponibilidad de datos en Cassandra viene determinada por el nivel de coherencia usado por la aplicación y el mecanismo de replicación. Para más información sobre la replicación en Cassandra, consulte [Data Replication in NoSQL Databases Explained][cassandra-replication] (Explicación de la replicación de datos en bases de datos NoSQL).

## <a name="security-considerations"></a>Consideraciones sobre la seguridad

Las redes virtuales son un límite de aislamiento del tráfico de Azure. Las máquinas virtuales de una red virtual no se pueden comunicar directamente con las máquinas virtuales de una red virtual diferente. Las máquinas virtuales que se encuentran en la misma red virtual se pueden comunicar entre sí, a menos que se creen [grupos de seguridad de red][nsg] (NSG) para restringir el tráfico. Para más información, consulte [Servicios en la nube de Microsoft y seguridad de red][network-security].

Para el tráfico entrante de Internet, las reglas del equilibrador de carga definen qué tráfico puede alcanzar el back-end. Sin embargo, las reglas del equilibrador de carga no son compatibles con las listas seguras de IP, por lo que si desea agregar determinadas direcciones IP públicas a una lista segura, agregue un NSG a la subred.

Considere la posibilidad de agregar una aplicación virtual de red (NVA) para crear una red perimetral entre la red de Internet y la red virtual de Azure. NVA es un término genérico para una aplicación virtual que puede realizar tareas relacionadas con la red, como firewall, inspección de paquetes, auditoría y enrutamiento personalizado. Para más información, vea [Implementación de una red perimetral entre Internet y Azure][dmz].

Cifre información confidencial en reposo y use [Azure Key Vault][azure-key-vault] para administrar las claves de cifrado de la base de datos. Key Vault puede almacenar las claves de cifrado en módulos de seguridad de hardware (HSM). También se recomienda almacenar los secretos de aplicación como, por ejemplo, las cadenas de conexión de base de datos, en Key Vault.

Se recomienda habilitar [DDoS Protection Standard](/azure/virtual-network/ddos-protection-overview), que mitiga los riesgos de DDoS para los recursos de una red virtual. Aunque la protección contra DDoS básica se habilita automáticamente como parte de la plataforma Azure, DDoS Protection Standard proporciona funcionalidades de mitigación de riesgos ajustadas específicamente a los recursos de Azure Virtual Network.  

## <a name="deploy-the-solution"></a>Implementación de la solución

Hay disponible una implementación de esta arquitectura de referencia en [GitHub][github-folder]. 

### <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-the-solution-using-azbb"></a>Implementación de la solución con AZBB

Para implementar las máquinas virtuales de Linux en una arquitectura de referencia de la aplicación de N niveles, siga estos pasos:

1. Vaya a la carpeta `virtual-machines\n-tier-linux` del repositorio que clonó en el paso 1 donde se detallaron los requisitos previos.

2. El archivo de parámetros especifica un nombre de usuario administrador y una contraseña predeterminados para cada máquina virtual de la implementación. Debe cambiar estos datos antes de implementar la arquitectura de referencia. Abra el archivo `n-tier-linux.json` y reemplace los campos **adminUsername** y **adminPassword** con la nueva configuración.   Guarde el archivo.

3. Implemente la arquitectura de referencia mediante la herramienta de línea de comandos **azbb** tal y como se muestra a continuación.

   ```bash
   azbb -s <your subscription_id> -g <your resource_group_name> -l <azure region> -p n-tier-linux.json --deploy
   ```

Para obtener más información sobre la implementación de esta arquitectura de referencia de ejemplo mediante Azure Bulding Blocks, visite el [repositorio de GitHub][git].

<!-- links -->
[dmz]: ../dmz/secure-vnet-dmz.md
[multi-vm]: ./multi-vm.md
[naming conventions]: /azure/guidance/guidance-naming-conventions
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azure-administration]: /azure/automation/automation-intro
[azure-availability-sets]: /azure/virtual-machines/virtual-machines-linux-manage-availability
[azure-cli-2]: https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest
[azure-dns]: /azure/dns/dns-overview
[azure-key-vault]: https://azure.microsoft.com/services/key-vault

[host bastión]: https://en.wikipedia.org/wiki/Bastion_host
[cassandra-in-azure]: https://academy.datastax.com/resources/deployment-guide-azure
[cassandra-consistency]: https://docs.datastax.com/en/cassandra/2.0/cassandra/dml/dml_config_consistency_c.html
[cassandra-replication]: https://academy.datastax.com/planet-cassandra/data-replication-in-nosql-databases-explained
[cassandra-consistency-usage]: https://medium.com/@foundev/cassandra-how-many-nodes-are-talked-to-with-quorum-also-should-i-use-it-98074e75d7d5#.b4pb4alb2

[CIDR]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[chef]: https://www.chef.io/solutions/azure/
[datastax]: https://www.datastax.com/products/datastax-enterprise
[git]: https://github.com/mspnp/template-building-blocks
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/n-tier-linux
[lb-external-create]: /azure/load-balancer/load-balancer-get-started-internet-portal
[lb-internal-create]: /azure/load-balancer/load-balancer-get-started-ilb-arm-portal
[load-balancer-external]: /azure/load-balancer/load-balancer-internet-overview
[load-balancer-internal]: /azure/load-balancer/load-balancer-internal-overview
[nsg]: /azure/virtual-network/virtual-networks-nsg
[nsg-rules]: /azure/azure-resource-manager/best-practices-resource-manager-security#network-security-groups
[operations-management-suite]: https://www.microsoft.com/server-cloud/operations-management-suite/overview.aspx
[plan-network]: /azure/virtual-network/virtual-network-vnet-plan-design-arm
[private-ip-space]: https://en.wikipedia.org/wiki/Private_network#Private_IPv4_address_spaces
[Dirección IP pública]: /azure/virtual-network/virtual-network-ip-addresses-overview-arm
[puppet]: https://puppetlabs.com/blog/managing-azure-virtual-machines-puppet
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[vnet faq]: /azure/virtual-network/virtual-networks-faq
[visio-download]: https://archcenter.blob.core.windows.net/cdn/vm-reference-architectures.vsdx
[Nagios]: https://www.nagios.org/
[Zabbix]: https://www.zabbix.com/
[Icinga]: https://www.icinga.org/
[0]: ./images/n-tier-cassandra.png "Arquitectura de n niveles con Microsoft Azure"

[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview 
[vmss]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
[load-balancer]: /azure/load-balancer/load-balancer-get-started-internet-arm-cli
[load-balancer-hashing]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[vmss-design]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-design-overview
[subscription-limits]: /azure/azure-subscription-service-limits
[availability-set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[health-probes]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[health-probe-log]: /azure/load-balancer/load-balancer-monitor-log
[health-probe-ip]: /azure/virtual-network/virtual-networks-nsg#special-rules
[network-security]: /azure/best-practices-network-security
