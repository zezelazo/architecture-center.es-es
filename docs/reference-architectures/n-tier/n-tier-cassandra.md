---
title: Aplicación de n niveles con Apache Cassandra
titleSuffix: Azure Reference Architectures
description: Ejecute máquinas virtuales Linux para una arquitectura de n niveles con Apache Cassandra en Microsoft Azure.
author: MikeWasson
ms.date: 11/12/2018
ms.custom: seodec18
ms.openlocfilehash: bbd1029fe17b5d88d54246127c5d8983a573b012
ms.sourcegitcommit: 88a68c7e9b6b772172b7faa4b9fd9c061a9f7e9d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/08/2018
ms.locfileid: "53120176"
---
# <a name="linux-n-tier-application-in-azure-with-apache-cassandra"></a>Aplicación Linux de n niveles en Azure con Apache Cassandra

Esta arquitectura de referencia muestra cómo implementar máquinas virtuales (VM) y una red virtual configurada para una aplicación de n niveles, con Apache Cassandra en Linux para la capa de datos. [**Implemente esta solución**](#deploy-the-solution).

![Arquitectura de n niveles con Microsoft Azure](./images/n-tier-cassandra.png)

*Descargue un [archivo Visio][visio-download] de esta arquitectura.*

## <a name="architecture"></a>Arquitectura

La arquitectura consta de los siguientes componentes:

- **Grupo de recursos**. Los [grupos de recursos][resource-manager-overview] se utilizan para agrupar los recursos, para que puedan administrarse según su duración, su propietario u otros criterios.

- **Red virtual (VNet) y subredes**. Cada máquina virtual de Azure se implementa en una red virtual que se puede dividir en subredes. Cree una subred independiente para cada nivel.

- **NSG**. Use [grupos de seguridad de red][nsg] (NSG) para restringir el tráfico de red dentro de la red virtual. Por ejemplo, en la arquitectura de tres niveles que se muestra aquí, el nivel de base de datos acepta tráfico del nivel de empresa y la subred de administración, pero no desde el front-end web.

- **Protección contra DDoS**. Aunque la plataforma Azure proporciona protección básica contra ataques de denegación de servicio distribuido (DDoS), se recomienda usar el [estándar de protección contra DDoS][ddos], que tiene características de mitigación de DDoS. Consulte [Consideraciones sobre la seguridad](#security-considerations).

- **Máquinas virtuales**. Para obtener recomendaciones sobre la configuración de máquinas virtuales, consulte [Ejecución de una VM con Windows en Azure](./windows-vm.md) y [Ejecución de una VM con Linux en Azure](./linux-vm.md).

- **Conjuntos de disponibilidad**. Cree un [conjunto de disponibilidad][azure-availability-sets] y aprovisione al menos dos máquinas virtuales en cada nivel, que hace que las máquinas virtuales que sean aptas para un [Acuerdo de Nivel de Servicio (SLA)][vm-sla] superior.

- **Equilibradores de carga de Azure**. Los [equilibradores de carga][load-balancer] distribuyen las solicitudes entrantes de Internet a las instancias de máquina virtual. Use un [equilibrador de carga público][load-balancer-external] para distribuir el tráfico entrante de Internet al nivel Web y un [equilibrador de carga interno][load-balancer-internal] para distribuir el tráfico de red del nivel Web al nivel Business.

- **Dirección IP pública**. Se necesita una dirección IP pública para que el equilibrador de carga reciba tráfico de Internet.

- **JumpBox**. También se denomina [host bastión]. Se trata de una máquina virtual segura en la red que usan los administradores para conectarse al resto de máquinas virtuales. El JumpBox tiene un NSG que solo permite el tráfico remoto que procede de direcciones IP públicas de una lista segura. El grupo de seguridad de red debe permitir el tráfico SSH.

- **Base de datos Apache Cassandra**. Proporciona alta disponibilidad en el nivel de datos, al habilitar la replicación y la conmutación por error.

- **Azure DNS**. [Azure DNS][azure-dns] es un servicio de hospedaje para dominios DNS. Proporciona resolución de nombres mediante la infraestructura de Microsoft Azure. Al hospedar dominios en Azure, puede administrar los registros DNS con las mismas credenciales, API, herramientas y facturación que con los demás servicios de Azure.

## <a name="recommendations"></a>Recomendaciones

Los requisitos pueden diferir de los de la arquitectura que se describe aquí. Use estas recomendaciones como punto inicial.

### <a name="vnet--subnets"></a>Red virtual/subredes

Cuando cree la red virtual, determine cuántas direcciones IP requieren los recursos de cada subred. Especifique una máscara de subred y un intervalo de direcciones de la red virtual lo suficientemente grande para la dirección IP requerida con el uso de la notación [CIDR]. Use un espacio de direcciones que se encuentre dentro de los [bloques de direcciones IP privados][private-ip-space] estándar, que son 10.0.0.0/8, 172.16.0.0/12 y 192.168.0.0/16.

Si va a necesitar configurar una puerta de enlace entre la red virtual y la red local más adelante, elija un intervalo de direcciones que no se superponga con la red local. Una vez creada la red virtual, no se puede cambiar el intervalo de direcciones.

Diseñe subredes teniendo en cuenta los requisitos de funcionalidad y seguridad. Todas las máquinas virtuales dentro del mismo nivel o rol deben incluirse en la misma subred, lo que puede servir como un límite de seguridad. Para obtener más información sobre el diseño de redes virtuales y subredes, vea [Planeación y diseño de redes virtuales de Azure][plan-network].

### <a name="load-balancers"></a>Equilibradores de carga

No exponga las máquinas virtuales directamente a Internet. En su lugar, asigne a cada máquina virtual una dirección IP privada. Los clientes se conectan con la dirección IP del equilibrador de carga público.

Defina reglas del equilibrador de carga para dirigir el tráfico de red a las máquinas virtuales. Por ejemplo, para habilitar el tráfico HTTP, cree una regla que asigne el puerto 80 de la configuración de front-end al puerto 80 del grupo de direcciones de back-end. Cuando un cliente envía una solicitud HTTP al puerto 80, el equilibrador de carga selecciona una dirección IP de back-end mediante un [algoritmo hash][load-balancer-hashing] que incluye la dirección IP de origen. Las solicitudes del cliente se distribuyen entre todas las máquinas virtuales.

### <a name="network-security-groups"></a>Grupos de seguridad de red

Use reglas NSG para restringir el tráfico entre los niveles. Por ejemplo, en la arquitectura de tres niveles mostrada anteriormente, el nivel web no se comunica directamente con el nivel de base de datos. Para exigir esto, el nivel de base de datos debe bloquear el tráfico entrante desde la subred del nivel Web.

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

Para lograr alta disponibilidad, implemente Cassandra en más de una región de Azure. En cada región, los nodos están configurados en modo compatible con bastidor con dominios de error y actualización para proporcionar resistencia dentro de la región.

### <a name="jumpbox"></a>JumpBox

No permita el acceso SSH desde la red pública de Internet a las máquinas virtuales que ejecutan la carga de trabajo de la aplicación. En su lugar, todo el acceso SSH a estas máquinas virtuales debe realizarse a través de JumpBox. Un administrador inicia sesión en JumpBox y, después, en la otra máquina virtual desde JumpBox. JumpBox permite el tráfico SSH desde Internet, pero solo desde direcciones IP conocidas y seguras.

JumpBox tiene unos requisitos de rendimiento mínimos, por lo que puede seleccionar un pequeño tamaño de máquina virtual. Cree una [dirección IP pública] para JumpBox. Coloque JumpBox en la misma red virtual que las demás máquinas virtuales, pero en una subred de administración independiente.

Para proteger JumpBox, agregue una regla de grupo de seguridad de red que permita las conexiones SSH solo desde un conjunto seguro de direcciones IP públicas. Configure el NSG para las demás subredes, a fin de permitir el tráfico SSH de la subred de administración.

## <a name="scalability-considerations"></a>Consideraciones sobre escalabilidad

Para los niveles web y de empresa, considere la posibilidad de usar [conjuntos de escalado de máquinas virtuales][vmss], en lugar de implementar máquinas virtuales independientes en un conjunto de disponibilidad. Los conjuntos de escalado facilitan la implementación y administración de un conjunto de máquinas virtuales idénticas y el escalado automático de dichas máquinas en función de sus métricas de rendimiento. A medida que aumenta la carga en las máquinas virtuales, se agregan más máquinas virtuales automáticamente al equilibrador de carga. Considere la posibilidad de usar conjuntos de escalado si necesita escalar horizontalmente las máquinas virtuales de inmediato o si necesita realizar el escalado automático.

Hay dos maneras básicas de configurar máquinas virtuales implementadas en un conjunto de escalado:

- Use extensiones para configurar la máquina virtual después de implementarla. Con este método, las nuevas instancias de máquina virtual pueden tardar más en iniciarse que una máquina virtual sin extensiones.

- Implemente un [disco administrado](/azure/storage/storage-managed-disks-overview) con una imagen de disco personalizada. Esta opción puede ser más rápida de implementar. Sin embargo, requiere que la imagen esté actualizada.

Para más información, consulte [Consideraciones de diseño para conjuntos de escalado][vmss-design].

> [!TIP]
> Cuando utilice cualquier solución de escalado automático, pruébela con antelación con cargas de trabajo de nivel de producción.

Cada suscripción de Azure tiene límites predeterminados establecidos, incluido un número máximo de máquinas virtuales por región. Puede aumentar el límite si rellena una solicitud de soporte técnico. Para más información, consulte [Límites, cuotas y restricciones de suscripción y servicios de Microsoft Azure][subscription-limits].

## <a name="availability-considerations"></a>Consideraciones sobre disponibilidad

Si no usa conjuntos de escalado de máquinas virtuales, coloque las máquinas virtuales del mismo nivel en un conjunto de disponibilidad. Cree al menos dos máquinas virtuales en el conjunto de disponibilidad, para admitir el [SLA de disponibilidad para máquinas virtuales de Azure][vm-sla]. Para más información, consulte [Administración de la disponibilidad de las máquinas virtuales][availability-set]. Los conjuntos de escalado usan automáticamente *grupos de selección de ubicación*, que actúan como un conjunto de disponibilidad implícito.

El equilibrador de carga usa [sondeos de mantenimiento][health-probes] para supervisar la disponibilidad de las instancias de máquina virtual. Si un sondeo no puede acceder a una instancia antes de que expire, el equilibrador de carga deja de enviar tráfico a esa máquina virtual. pero continua con el sondeo y si la máquina virtual vuelve a estar disponible, vuelve a enviarle tráfico.

A continuación, se presentan algunas recomendaciones sobre los sondeos de mantenimiento del equilibrador de carga:

- Los sondeos pueden probar los protocolos HTTP o TCP. Si las máquinas virtuales ejecutan un servidor HTTP, cree un sondeo HTTP. De lo contrario, cree un sondeo TCP.
- Para un sondeo HTTP, especifique la ruta de acceso a un punto de conexión HTTP. El sondeo comprueba si hay una respuesta HTTP 200 desde esta ruta de acceso. Puede ser la ruta de acceso raíz ("/") o un punto de conexión de supervisión de mantenimiento que implementa alguna lógica personalizada para comprobar el mantenimiento de la aplicación. El punto de conexión debe permitir solicitudes HTTP anónimas.
- El sondeo se envía desde una [dirección IP conocida][health-probe-ip], 168.63.129.16. Asegúrese de que no bloquea el tráfico que llega a esta dirección IP, ni el que parte de ella, en las directivas de firewall o en las reglas del grupo de seguridad de red.
- Use [registros de sondeo de mantenimiento][health-probe-log] para ver el estado de los sondeos de mantenimiento. Habilite el registro en Azure Portal para cada equilibrador de carga. Los registros se escriben en Azure Blob Storage. Los registros muestran el número de máquinas virtuales que no reciben tráfico de red debido la falta de respuesta en los sondeos.

En el caso del clúster de Cassandra. los escenarios de conmutación por error dependen tanto de los niveles de coherencia que use la aplicación como del número de réplicas. Para más información sobre los niveles de coherencia y el uso en Cassandra, consulte [Configuración de la coherencia de datos][cassandra-consistency] y [Cassandra: ¿Con cuántos nodos se comunica con cuórum?][cassandra-consistency-usage] La disponibilidad de datos en Cassandra viene determinada por el nivel de coherencia usado por la aplicación y el mecanismo de replicación. Para más información sobre la replicación en Cassandra, consulte [Data Replication in NoSQL Databases Explained][cassandra-replication] (Explicación de la replicación de datos en bases de datos NoSQL).

## <a name="security-considerations"></a>Consideraciones sobre la seguridad

Las redes virtuales son un límite de aislamiento del tráfico de Azure. Las máquinas virtuales de una red virtual no se pueden comunicar directamente con las de otra. Las máquinas virtuales que se encuentran en la misma red virtual se pueden comunicar entre sí, a menos que se creen [grupos de seguridad de red][nsg] (NSG) para restringir el tráfico. Para más información, consulte [Servicios en la nube de Microsoft y seguridad de red][network-security].

Para el tráfico entrante de Internet, las reglas del equilibrador de carga definen qué tráfico puede alcanzar el back-end. Sin embargo, las reglas del equilibrador de carga no son compatibles con las listas seguras de IP, por lo que si desea agregar determinadas direcciones IP públicas a una lista segura, agregue un NSG a la subred.

**DMZ**. Considere la posibilidad de agregar una aplicación virtual de red (NVA) para crear una red perimetral entre la red de Internet y la red virtual de Azure. NVA es un término genérico para una aplicación virtual que puede realizar tareas relacionadas con la red, como firewall, inspección de paquetes, auditoría y enrutamiento personalizado. Para más información, vea [Implementación de una red perimetral entre Internet y Azure][dmz].

**Cifrado**. Cifre información confidencial en reposo y use [Azure Key Vault][azure-key-vault] para administrar las claves de cifrado de la base de datos. Key Vault puede almacenar las claves de cifrado en módulos de seguridad de hardware (HSM). También se recomienda almacenar los secretos de aplicación como, por ejemplo, las cadenas de conexión de base de datos, en Key Vault.

**DDoS Protection**. De forma predeterminada, la plataforma Azure proporciona protección básica contra DDoS. El objetivo de dicha es proteger la infraestructura de Azure en su conjunto. Aunque la protección básica contra DDoS se habilita automáticamente, se recomienda usar [DDoS Protection Estándar][ddos]. Para detectar las amenazas, la protección estándar usa un ajuste que se puede adaptar en función de los patrones de tráfico de red de la aplicación, lo que le permite aplicar mitigaciones frente a ataques de denegación de servicio distribuido que podrían pasar desapercibidas para las directivas de DDoS para toda la infraestructura. La protección estándar también proporciona alertas, telemetría y análisis a través de Azure Monitor. Para más información, consulte [Azure DDoS Protection: procedimientos recomendados y arquitecturas de referencia][ddos-best-practices].

## <a name="deploy-the-solution"></a>Implementación de la solución

Hay disponible una implementación de esta arquitectura de referencia en [GitHub][github-folder].

### <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-the-solution-using-azbb"></a>Implementación de la solución con AZBB

Para implementar las máquinas virtuales de Linux en una arquitectura de referencia de la aplicación de N niveles, siga estos pasos:

1. Vaya a la carpeta `virtual-machines\n-tier-linux` del repositorio que clonó en el paso 1 donde se detallaron los requisitos previos.

2. El archivo de parámetros especifica un nombre de usuario administrador y una contraseña predeterminados para cada máquina virtual de la implementación. Cámbielos antes de implementar la arquitectura de referencia. Abra el archivo `n-tier-linux.json` y reemplace los campos **adminUsername** y **adminPassword** con la nueva configuración.   Guarde el archivo.

3. Implemente la arquitectura de referencia mediante la herramienta **azbb** como se muestra a continuación.

   ```azurecli
   azbb -s <your subscription_id> -g <your resource_group_name> -l <azure region> -p n-tier-linux.json --deploy
   ```

Para obtener más información sobre la implementación de esta arquitectura de referencia de ejemplo mediante Azure Bulding Blocks, visite el [repositorio de GitHub][git].

<!-- links -->

[dmz]: ../dmz/secure-vnet-dmz.md
[multi-vm]: ./multi-vm.md
[naming conventions]: /azure/guidance/guidance-naming-conventions
[azure-availability-sets]: /azure/virtual-machines/virtual-machines-linux-manage-availability
[azure-dns]: /azure/dns/dns-overview
[azure-key-vault]: https://azure.microsoft.com/services/key-vault

[host bastión]: https://en.wikipedia.org/wiki/Bastion_host
[cassandra-in-azure]: https://academy.datastax.com/resources/deployment-guide-azure
[cassandra-consistency]: https://docs.datastax.com/en/cassandra/2.0/cassandra/dml/dml_config_consistency_c.html
[cassandra-replication]: https://academy.datastax.com/planet-cassandra/data-replication-in-nosql-databases-explained
[cassandra-consistency-usage]: https://medium.com/@foundev/cassandra-how-many-nodes-are-talked-to-with-quorum-also-should-i-use-it-98074e75d7d5#.b4pb4alb2

[CIDR]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[datastax]: https://www.datastax.com/products/datastax-enterprise
[ddos]: /azure/virtual-network/ddos-protection-overview
[ddos-best-practices]: /azure/security/azure-ddos-best-practices
[git]: https://github.com/mspnp/template-building-blocks
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/n-tier-linux
[load-balancer-external]: /azure/load-balancer/load-balancer-internet-overview
[load-balancer-internal]: /azure/load-balancer/load-balancer-internal-overview
[nsg]: /azure/virtual-network/virtual-networks-nsg
[nsg-rules]: /azure/azure-resource-manager/best-practices-resource-manager-security#network-security-groups
[plan-network]: /azure/virtual-network/virtual-network-vnet-plan-design-arm
[private-ip-space]: https://en.wikipedia.org/wiki/Private_network#Private_IPv4_address_spaces
[Dirección IP pública]: /azure/virtual-network/virtual-network-ip-addresses-overview-arm
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[visio-download]: https://archcenter.blob.core.windows.net/cdn/vm-reference-architectures.vsdx

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
