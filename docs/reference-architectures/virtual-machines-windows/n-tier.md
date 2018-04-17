---
title: Ejecución de VM con Windows para una arquitectura de n niveles
description: Cómo implementar una arquitectura de varios niveles en Azure, prestando especial atención a la disponibilidad, seguridad, escalabilidad y seguridad de facilidad de uso.
author: MikeWasson
ms.date: 11/22/2016
pnp.series.title: Windows VM workloads
pnp.series.next: multi-region-application
pnp.series.prev: multi-vm
ms.openlocfilehash: 5ed94eb9ab8203d35d9597336e367d54e03944d7
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2018
---
# <a name="run-windows-vms-for-an-n-tier-application"></a>Ejecución de máquinas virtuales Windows para una arquitectura de n niveles

En esta arquitectura de referencia se muestra un conjunto de prácticas demostradas para ejecutar máquinas virtuales Windows para una aplicación de n niveles. [**Implemente esta solución**.](#deploy-the-solution) 

![[0]][0]

*Descargue un [archivo Visio][visio-download] de esta arquitectura.*

## <a name="architecture"></a>Architecture 

Hay muchas maneras de implementar una arquitectura de n niveles. En el diagrama se muestra una aplicación web típica de tres niveles. Esta arquitectura se basa en [Ejecución de máquinas virtuales de carga equilibrada para escalabilidad y disponibilidad][multi-vm]. Los niveles Web y Business usan máquinas virtuales de carga equilibrada.

* **Conjuntos de disponibilidad**. Cree un [conjunto de disponibilidad][azure-availability-sets] para cada nivel y aprovisione al menos dos máquinas virtuales en cada nivel. Esto hace que las máquinas virtuales sean aptas para un [Acuerdo de Nivel de Servicio (SLA)][vm-sla] mayor. Puede implementar una sola máquina virtual en un conjunto de disponibilidad, pero esta no reunirá los requisitos de la garantía de SLA a menos que use Azure Premium Storage en todos los sistemas operativos y discos de datos.  
* **Subredes**. Cree una subred independiente para cada nivel. Especifique el intervalo de direcciones y la máscara de subred con la notación [CIDR]. 
* **Equilibradores de carga.** Use un [equilibrador de carga con conexión a Internet][load-balancer-external] para distribuir el tráfico entrante de Internet al nivel Web y un [equilibrador de carga interno][load-balancer-internal] para distribuir el tráfico de red del nivel Web al nivel Business.
* **JumpBox.** También se denomina [host bastión]. Se trata de una máquina virtual segura en la red que usan los administradores para conectarse al resto de máquinas virtuales. El Jumpbox tiene un NSG que solo permite el tráfico remoto que procede de direcciones IP públicas de una lista segura. El NSG debe permitir el tráfico de escritorio remoto (RDP).
* **Supervisión.** El software de supervisión, como [Nagios], [Zabbix] o [Icinga], puede ofrecerle información sobre el tiempo de respuesta, el tiempo de actividad de la máquina virtual y el estado general del sistema. Instale el software de supervisión en una máquina virtual que se encuentre en una subred de administración independiente.
* <strong>Grupos de seguridad de red.</strong> Use [grupos de seguridad de red][nsg] (NSG) para restringir el tráfico de red dentro de la red virtual. Por ejemplo, en la arquitectura de tres niveles que se muestra aquí, el nivel de base de datos no acepta el tráfico desde el front-end web, solo desde el nivel Business y la subred de administración.
* **Grupo de disponibilidad AlwaysOn de SQL Server**. Proporciona alta disponibilidad en el nivel de datos, al habilitar la replicación y la conmutación por error.
* **Servidores de Active Directory Domain Services (AD DS)**. Antes de Windows Server 2016, los grupos de disponibilidad AlwaysOn de SQL Server deben estar unidos a un dominio. Esto se debe a que los grupos de disponibilidad dependen de la tecnología del clúster de conmutación por error de Windows Server (WSFC). Windows Server 2016 incorpora la capacidad de crear un clúster de conmutación por error sin Active Directory, en cuyo caso los servidores de AD DS no son necesarios para esta arquitectura. Para más información, vea [Novedades en el clúster de conmutación por error en Windows Server 2016][wsfc-whats-new].
* **Azure DNS**. [Azure DNS][azure-dns] es un servicio de hospedaje para dominios DNS que permite resolver nombres mediante la infraestructura de Microsoft Azure. Al hospedar dominios en Azure, puede administrar los registros DNS con las mismas credenciales, API, herramientas y facturación que con los demás servicios de Azure.

## <a name="recommendations"></a>Recomendaciones

Los requisitos pueden diferir de los de la arquitectura que se describe aquí. Use estas recomendaciones como punto inicial. 

### <a name="vnet--subnets"></a>Red virtual/subredes

Cuando cree la red virtual, determine cuántas direcciones IP requieren los recursos de cada subred. Especifique una máscara de subred y un intervalo de direcciones de la red virtual lo suficientemente grande para la dirección IP requerida con el uso de la notación [CIDR]. Use un espacio de direcciones que se encuentre dentro de los [bloques de direcciones IP privados][private-ip-space] estándar, que son 10.0.0.0/8, 172.16.0.0/12 y 192.168.0.0/16.

Elija un intervalo de direcciones que no se superponga con la red local, en caso de que necesite configurar una puerta de enlace entre la red virtual y la red local más adelante. Una vez creada la red virtual, no se puede cambiar el intervalo de direcciones.

Diseñe subredes teniendo en cuenta los requisitos de funcionalidad y seguridad. Todas las máquinas virtuales dentro del mismo nivel o rol deben incluirse en la misma subred, lo que puede servir como un límite de seguridad. Para obtener más información sobre el diseño de redes virtuales y subredes, vea [Planeación y diseño de redes virtuales de Azure][plan-network].

Para cada subred, especifique el espacio de direcciones de la subred en la notación CIDR. Por ejemplo, "10.0.0.0/24" crea un intervalo de doscientas cincuenta y seis direcciones IP. Las máquinas virtuales pueden usar doscientas cincuenta y una; cinco están reservadas. Asegúrese de que los intervalos de direcciones no se superponen entre las subredes. Vea [Preguntas más frecuentes (P+F) acerca de Azure Virtual Network][vnet faq].

### <a name="network-security-groups"></a>Grupos de seguridad de red

Use reglas NSG para restringir el tráfico entre los niveles. Por ejemplo, en la arquitectura de tres niveles mostrada anteriormente, el nivel Web no se comunica directamente con el nivel de base de datos. Para exigir esto, el nivel de base de datos debe bloquear el tráfico entrante desde la subred del nivel Web.  

1. Cree un NSG y asócielo a la subred del nivel de base de datos.
2. Agregue una regla que deniegue todo el tráfico entrante de la red virtual. (Use la etiqueta `VIRTUAL_NETWORK` de la regla). 
3. Agregue una regla con mayor prioridad que permita el tráfico entrante de la subred del nivel Business. Esta regla invalida la regla anterior y permite al nivel Business comunicarse con el nivel de base de datos.
4. Agregue una regla que permita el tráfico entrante desde dentro de la subred del nivel de base de datos. Esta regla permite la comunicación entre las máquinas virtuales en el nivel de base de datos, que es necesario para la replicación y la conmutación por error de la base de datos.
5. Agregue una regla que permita el tráfico RDP de la subred de JumpBox. Esta regla permite a los administradores conectarse al nivel de base de datos desde JumpBox.
   
   > [!NOTE]
   > Un NSG tiene reglas predeterminadas que permiten todo el tráfico entrante desde dentro de la red virtual. Estas reglas no se pueden eliminar, pero sí se pueden invalidar con la creación de reglas de mayor prioridad.
   > 
   > 

### <a name="load-balancers"></a>Equilibradores de carga

El equilibrador de carga externo distribuye el tráfico de Internet al nivel Web. Cree una dirección IP pública para este equilibrador de carga. Vea [Creación de un equilibrador de carga orientado a Internet mediante Azure Portal][lb-external-create].

El equilibrador de carga interno distribuye el tráfico de red del nivel Web al nivel Business. Para asignar una dirección IP privada a este equilibrador de carga, cree una configuración de IP de front-end y asóciela a la subred para el nivel Business. Vea [Creación de un equilibrador de carga interno en Azure Portal][lb-internal-create].

### <a name="sql-server-always-on-availability-groups"></a>Grupos de disponibilidad AlwaysOn de SQL Server

Se recomiendan los [grupos de disponibilidad AlwaysOn][sql-alwayson] para alta disponibilidad de SQL Server. Antes de Windows Server 2016, los grupos de disponibilidad AlwaysOn requerían un controlador de dominio y todos los nodos del grupo de disponibilidad debían estar en el mismo dominio de AD.

Otros niveles se conectan a la base de datos a través de una [escucha de grupo de disponibilidad][sql-alwayson-listeners]. La escucha permite a un cliente SQL conectarse sin conocer el nombre de la instancia física de SQL Server. Las máquinas virtuales que acceden a la base de datos deben estar unidas al dominio. El cliente (en este caso, otro nivel) utiliza DNS para resolver el nombre de la red virtual de la escucha en direcciones IP.

Configure el grupo de disponibilidad AlwaysOn de SQL Server como sigue:

1. Cree un clúster de clústeres de conmutación por error de Windows Server (WSFC), un grupo de disponibilidad AlwaysOn de SQL Server y una réplica principal. Para más información, vea [Introducción a Grupos de disponibilidad AlwaysOn][sql-alwayson-getting-started]. 
2. Cree un equilibrador de carga interno con una dirección IP privada estática.
3. Cree una escucha de grupo de disponibilidad y asigne el nombre DNS de la escucha a la dirección IP del equilibrador de carga interno. 
4. Cree una regla del equilibrador de carga para el puerto de escucha de SQL Server (puerto TCP 1433 de forma predeterminada). La regla del equilibrador de carga debe habilitar la *IP flotante*, también denominada Direct Server Return. Esto causa que la máquina virtual responda directamente al cliente, lo que permite establecer una conexión directa con la réplica principal.
  
   > [!NOTE]
   > Cuando la IP flotante está habilitada, el número de puerto de front-end debe ser el mismo que el número de puerto de back-end en la regla del equilibrador de carga.
   > 
   > 

Cuando un cliente SQL intenta conectarse, el equilibrador de carga enruta la solicitud de conexión a la réplica principal. Si se produce una conmutación por error a otra réplica, el equilibrador de carga enruta automáticamente las solicitudes posteriores a una nueva réplica principal. Para más información, vea [Configuración de un equilibrador de carga para Grupos de disponibilidad AlwaysOn de SQL Server][sql-alwayson-ilb].

Durante una conmutación por error, se cierran las conexiones de cliente existentes. Una vez completada la conmutación por error, las conexiones nuevas se enrutarán a la nueva réplica principal.

Si la aplicación realiza significativamente más lecturas que escrituras, puede descargar algunas de las consultas de solo lectura en una réplica secundaria. Vea [Usar un agente de escucha para conectarse a una réplica secundaria de solo lectura (enrutamiento de solo lectura)][sql-alwayson-read-only-routing].

Pruebe la implementación mediante el [forzado de una conmutación por error manual][sql-alwayson-force-failover] del grupo de disponibilidad.

### <a name="jumpbox"></a>JumpBox

JumpBox tendrá requisitos mínimos de rendimiento, por lo que se debe seleccionar un tamaño pequeño de máquina virtual para JumpBox, como Estándar A1. 

Cree una [dirección IP pública] para JumpBox. Coloque JumpBox en la misma red virtual que las demás máquinas virtuales, pero en una subred de administración independiente.

No permita el acceso RDP desde la red pública de Internet a las máquinas virtuales que ejecutan la carga de trabajo de la aplicación. En su lugar, todo el acceso RDP a estas máquinas virtuales debe realizarse a través de JumpBox. Un administrador inicia sesión en JumpBox y, después, en la otra máquina virtual desde JumpBox. JumpBox permite el tráfico RDP desde Internet, pero solo desde direcciones IP conocidas y seguras.

Para proteger JumpBox, cree un NSG y aplíquelo a la subred de JumpBox. Agregue una regla NSG que permita las conexiones RDP solo desde un conjunto seguro de direcciones IP públicas. El NSG puede adjuntarse a la subred o al NIC de JumpBox. En este caso, se recomienda asociarlo a la NIC, para que solo se permita el tráfico RDP a JumpBox, incluso si agrega otras máquinas virtuales a la misma subred.

Configure el NSG para las demás subredes, a fin de permitir el tráfico RDP de la subred de administración.

## <a name="availability-considerations"></a>Consideraciones sobre disponibilidad

En el nivel de base de datos, tener varias máquinas virtuales no supone automáticamente disponer de una base de datos de alta disponibilidad. Para una base de datos relacional, normalmente deberá usar la replicación y la conmutación por error para lograr una alta disponibilidad. Para SQL Server, se recomienda usar los [grupos de disponibilidad AlwaysOn][sql-alwayson]. 

Si necesita más disponibilidad de la que proporciona el [SLA de Azure para máquinas virtuales][vm-sla], replique la aplicación entre dos regiones y use Azure Traffic Manager para la conmutación por error. Para más información, vea [Run Windows VMs in multiple regions for high availability][multi-dc] (Ejecución de máquinas virtuales Windows en varias regiones para alta disponibilidad).   

## <a name="security-considerations"></a>Consideraciones sobre la seguridad

Cifre información confidencial en reposo y use [Azure Key Vault][azure-key-vault] para administrar las claves de cifrado de la base de datos. Key Vault puede almacenar las claves de cifrado en módulos de seguridad de hardware (HSM). Para más información, vea [Configurar la integración de Azure Key Vault para SQL Server en máquinas virtuales de Azure][sql-keyvault]. También se recomienda almacenar secretos de aplicación, como cadenas de conexión de base de datos, en Key Vault.

Considere la posibilidad de agregar una aplicación virtual de red (NVA) para crear una red perimetral entre la red de Internet y la red virtual de Azure. NVA es un término genérico para una aplicación virtual que puede realizar tareas relacionadas con la red, como firewall, inspección de paquetes, auditoría y enrutamiento personalizado. Para más información, vea [Implementación de una red perimetral entre Internet y Azure][dmz].

## <a name="scalability-considerations"></a>Consideraciones sobre escalabilidad

Los equilibradores de carga distribuyen el tráfico de red a los niveles Web y Business. Escale horizontalmente mediante la adición de instancias de máquina virtual nuevas. Tenga en cuenta que se pueden escalar los niveles Web y Business por separado, según la carga. Para reducir las posibles complicaciones causadas por la necesidad de mantener la afinidad del cliente, las máquinas virtuales del nivel Web no deben tener ningún estado. Las máquinas virtuales que hospedan la lógica de negocios tampoco deben tener ningún estado.

## <a name="manageability-considerations"></a>Consideraciones sobre la manejabilidad

Simplifique la administración de todo el sistema mediante las herramientas de administración centralizadas, como [Azure Automation][azure-administration], [Microsoft Operations Management Suite] [ operations-management-suite], [Chef][chef] o [Puppet][puppet]. Estas herramientas pueden consolidar la información de diagnóstico y mantenimiento capturada de varias máquinas virtuales para proporcionar una visión general del sistema.

## <a name="deploy-the-solution"></a>Implementación de la solución

Hay disponible una implementación de esta arquitectura de referencia en [GitHub][github-folder]. 

### <a name="prerequisites"></a>requisitos previos

Antes de poder implementar la arquitectura de referencia en su propia suscripción, debe realizar los pasos siguientes.

1. Clone, bifurque o descargue el archivo ZIP del repositorio de GitHub de [arquitecturas de referencia][ref-arch-repo].

2. Asegúrese de que tiene la CLI de Azure 2.0 instalada en el equipo. Para instalar la CLI, siga las instrucciones de [Instalación de la CLI de Azure 2.0][azure-cli-2].

3. Instale el paquete de NPM de [Azure Building Blocks][azbb].

   ```bash
   npm install -g @mspnp/azure-building-blocks
   ```

4. Desde un símbolo del sistema, un símbolo del sistema de Bash o un símbolo del sistema de PowerShell, inicie sesión en la cuenta de Azure con alguno de los comandos siguientes y siga las indicaciones.

   ```bash
   az login
   ```

### <a name="deploy-the-solution-using-azbb"></a>Implementación de la solución con AZBB

Para implementar las máquinas virtuales de Windows en una arquitectura de referencia de la aplicación de N niveles, siga estos pasos:

1. Vaya a la carpeta `virtual-machines\n-tier-windows` del repositorio que clonó en el paso 1 donde se detallaron los requisitos previos.

2. El archivo de parámetros especifica un nombre de usuario administrador y una contraseña predeterminados para cada máquina virtual de la implementación. Debe cambiar estos datos antes de implementar la arquitectura de referencia. Abra el archivo `n-tier-windows.json` y reemplace los campos **adminUsername** y **adminPassword** con la nueva configuración.
  
   > [!NOTE]
   > En esta implementación existen varios scripts que se ejecutan tanto en los objetos **VirtualMachineExtension** como en la configuración de las **extensiones** de algunos objetos **VirtualMachine**. Algunos de estos scripts requieren el nombre de usuario administrador y la contraseña que acaba de cambiar. Le recomendamos que los revise para asegurarse de que especificó las credenciales correctas. Recuerde que es posible que se produzca un error en la implementación si no especifica las credenciales adecuadas.
   > 
   > 

Guarde el archivo.

3. Implemente la arquitectura de referencia mediante la herramienta de línea de comandos **azbb** tal y como se muestra a continuación.

   ```bash
   azbb -s <your subscription_id> -g <your resource_group_name> -l <azure region> -p n-tier-windows.json --deploy
   ```

Para obtener más información sobre la implementación de esta arquitectura de referencia de ejemplo mediante Azure Bulding Blocks, visite el [repositorio de GitHub][git].


<!-- links -->
[dmz]: ../dmz/secure-vnet-dmz.md
[multi-dc]: multi-region-application.md
[multi-vm]: multi-vm.md
[n-tier]: n-tier.md
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azure-administration]: /azure/automation/automation-intro
[azure-availability-sets]: /azure/virtual-machines/virtual-machines-windows-manage-availability#configure-each-application-tier-into-separate-availability-sets
[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-cli-2]: https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest
[azure-dns]: /azure/dns/dns-overview
[azure-key-vault]: https://azure.microsoft.com/services/key-vault
[host bastión]: https://en.wikipedia.org/wiki/Bastion_host
[CIDR]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[chef]: https://www.chef.io/solutions/azure/
[git]: https://github.com/mspnp/template-building-blocks
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/n-tier-windows
[lb-external-create]: /azure/load-balancer/load-balancer-get-started-internet-portal
[lb-internal-create]: /azure/load-balancer/load-balancer-get-started-ilb-arm-portal
[load-balancer-external]: /azure/load-balancer/load-balancer-internet-overview
[load-balancer-internal]: /azure/load-balancer/load-balancer-internal-overview
[nsg]: /azure/virtual-network/virtual-networks-nsg
[operations-management-suite]: https://www.microsoft.com/server-cloud/operations-management-suite/overview.aspx
[plan-network]: /azure/virtual-network/virtual-network-vnet-plan-design-arm
[private-ip-space]: https://en.wikipedia.org/wiki/Private_network#Private_IPv4_address_spaces
[dirección IP pública]: /azure/virtual-network/virtual-network-ip-addresses-overview-arm
[puppet]: https://puppetlabs.com/blog/managing-azure-virtual-machines-puppet
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[sql-alwayson]: https://msdn.microsoft.com/library/hh510230.aspx
[sql-alwayson-force-failover]: https://msdn.microsoft.com/library/ff877957.aspx
[sql-alwayson-getting-started]: https://msdn.microsoft.com/library/gg509118.aspx
[sql-alwayson-ilb]: /azure/virtual-machines/windows/sql/virtual-machines-windows-portal-sql-alwayson-int-listener
[sql-alwayson-listeners]: https://msdn.microsoft.com/library/hh213417.aspx
[sql-alwayson-read-only-routing]: https://technet.microsoft.com/library/hh213417.aspx#ConnectToSecondary
[sql-keyvault]: /azure/virtual-machines/virtual-machines-windows-ps-sql-keyvault
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[vnet faq]: /azure/virtual-network/virtual-networks-faq
[wsfc-whats-new]: https://technet.microsoft.com/windows-server-docs/failover-clustering/whats-new-in-failover-clustering
[Nagios]: https://www.nagios.org/
[Zabbix]: http://www.zabbix.com/
[Icinga]: http://www.icinga.org/
[visio-download]: https://archcenter.blob.core.windows.net/cdn/vm-reference-architectures.vsdx
[0]: ./images/n-tier-diagram.png "Arquitectura de n niveles con Microsoft Azure"