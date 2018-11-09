---
title: Implementación de una arquitectura de red híbrida segura en Azure
description: Proceso para implementar una arquitectura de red híbrida segura en Azure.
author: telmosampaio
ms.date: 10/22/2018
pnp.series.title: Network DMZ
pnp.series.prev: ./index
pnp.series.next: secure-vnet-dmz
cardTitle: DMZ between Azure and on-premises
ms.openlocfilehash: e13503c65430e46ef50898e471594a2ade3824b6
ms.sourcegitcommit: dbbf914757b03cdee7a274204f9579fa63d7eed2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/02/2018
ms.locfileid: "50916488"
---
# <a name="dmz-between-azure-and-your-on-premises-datacenter"></a>Red perimetral entre Azure y el centro de datos local

Esta arquitectura de referencia muestra una red híbrida segura que extiende una red local a Azure. La arquitectura implementa una zona DMZ, también conocida como *red perimetral*, entre la red local y una red virtual (VNet). La red perimetral incluye aplicaciones virtuales de red (NVA) que implementan la funcionalidad de seguridad, como firewalls e inspección de paquetes. Todo el tráfico saliente de la red virtual se realiza mediante tunelización forzada a Internet a través de la red local, por lo que se puede auditar. [**Implemente esta solución**.](#deploy-the-solution)

[![0]][0] 

*Descargue un [archivo Visio][visio-download] de esta arquitectura.*

Esta arquitectura requiere una conexión a su centro de datos local, mediante una [puerta de enlace VPN][ra-vpn] o una conexión [ExpressRoute][ra-expressroute]. Los usos habituales de esta arquitectura incluyen:

* Aplicaciones híbridas donde una parte de las cargas de trabajo se ejecutan de forma local y otra parte en Azure.
* Infraestructura que requiere un control específico sobre el tráfico que entra en una red virtual de Azure desde un centro de datos local.
* Aplicaciones que deben auditar el tráfico saliente. Esto suele ser un requisito de regulación de muchos sistemas comerciales y puede ayudar a evitar la revelación de información privada.

## <a name="architecture"></a>Arquitectura

La arquitectura consta de los siguientes componentes:

* **Red local**. Una red de área local privada implementada dentro de una organización.
* **Azure Virtual Network (VNet)**. La red virtual hospeda la aplicación y otros recursos que se ejecutan en Azure.
* **Puerta de enlace**. Proporciona conectividad entre los enrutadores de la red local y la red virtual.
* **Aplicación virtual de red (NVA)**. NVA es un término genérico que describe una máquina virtual para realizar tareas tales como permitir o denegar el acceso como un firewall; optimizar las operaciones de red de área extensa (WAN), incluida la compresión de red; la distribución personalizada; u otra funcionalidad de red.
* **Subredes del nivel de datos, nivel empresarial y nivel web**. Las subredes hospedan las máquinas virtuales y los servicios que implementan una aplicación de ejemplo de tres niveles que se ejecuta en la nube. Para más información, consulte [Ejecución de máquinas virtuales Windows para una arquitectura de n niveles][ra-n-tier].
* **Rutas definidas por el usuario (UDR)**. [Las rutas definidas por el usuario][udr-overview] definen el flujo de tráfico IP en las redes virtuales Azure.

    > [!NOTE]
    > Según los requisitos de la conexión VPN, puede configurar las rutas del Protocolo de puerta de enlace fronteriza (BGP) en lugar de usar UDR para implementar las reglas de reenvío que dirigen el tráfico a través de la red local.
    > 
    > 

* **Subred de administración.** Esta subred contiene máquinas virtuales que implementan las funcionalidades de administración y supervisión de los componentes que se ejecutan en la red virtual.

## <a name="recommendations"></a>Recomendaciones

Las siguientes recomendaciones sirven para la mayoría de los escenarios. Sígalas a menos que tenga un requisito concreto que las invalide. 

### <a name="access-control-recommendations"></a>Recomendaciones de control de acceso

Use [control de acceso basado en rol][ rbac] (RBAC) para administrar los recursos de la aplicación. Considere la posibilidad de crear los siguientes [roles personalizados][rbac-custom-roles]:

- Un rol de DevOps con permisos para administrar la infraestructura de la aplicación, implementar los componentes de las aplicaciones y supervisar y reiniciar las máquinas virtuales.  

- Un rol de administrador de TI centralizado para administrar y supervisar los recursos de red.

- Un rol de administrador de TI de seguridad para administrar los recursos de red seguros, como las aplicaciones virtuales de red (NVA) de seguridad. 

Los roles de DevOps y administrador de TI no deberían tener acceso a los recursos de las aplicaciones virtuales de red. Debería limitarse al rol de administrador de TI de seguridad.

### <a name="resource-group-recommendations"></a>Recomendaciones para grupos de recursos

Los recursos de Azure, como las máquinas virtuales, las redes virtuales y los equilibradores de carga, se pueden administrar fácilmente agrupándolos en grupos de recursos. Asigne roles RBAC a cada grupo de recursos para restringir el acceso.

Se recomienda crear los grupos de recursos siguientes:

* Un grupo de recursos que contenga la red virtual (excepto las máquinas virtuales), los grupos de seguridad de red y los recursos de puerta de enlace para conectarse a la red local. Asigne el rol de administrador de TI centralizado a este grupo de recursos.
* Un grupo de recursos que contenga las máquinas virtuales para las aplicaciones virtuales de red (NVA) (incluido el equilibrador de carga), el JumpBox y otras máquinas virtuales de administración y el UDR para la subred de puerta de enlace que obliga a todo el tráfico a pasar a través de las NVA. Asigne el rol de administrador de TI de seguridad a este grupo de recursos.
* Separe los grupos de recursos para cada nivel de aplicación que contenga el equilibrador de carga y las máquinas virtuales. Tenga en cuenta que este grupo de recursos no debe incluir las subredes de cada nivel. Asigne el rol de DevOps a este grupo de recursos.

### <a name="virtual-network-gateway-recommendations"></a>Recomendaciones para las puertas de enlace de red virtual

El tráfico local pasa a la red virtual a través de una puerta de enlace de red virtual. Se recomienda una [puerta de enlace Azure VPN][ guidance-vpn-gateway] o una [puerta de enlace Azure ExpressRoute][guidance-expressroute].

### <a name="nva-recommendations"></a>Recomendaciones para aplicaciones virtuales de red

Las aplicaciones virtuales de red proporcionan servicios diferentes para administrar y supervisar el tráfico de red. [Azure Marketplace][ azure-marketplace-nva] ofrece varias aplicaciones virtuales de red de otros proveedores que puede usar. Si ninguna de ellas cumple sus requisitos, puede crear una aplicación virtual de red personalizada usando máquinas virtuales. 

Por ejemplo, la implementación de la solución para esta arquitectura de referencia implementa una aplicación virtual de red personalizada con la funcionalidad siguiente en una máquina virtual:

* El tráfico se enruta mediante [reenvío IP][ip-forwarding] en las interfaces de red NVA personalizadas (NIC).
* Solo se permite pasar el tráfico a través de la aplicación virtual de red personalizada si es apropiado. Cada máquina virtual NVA de la arquitectura de referencia es un simple enrutador Linux. El tráfico entrante llega a la interfaz de red *eth0* y el tráfico de salida hace coincidir las reglas definidas por scripts personalizados que se envían a través de la interfaz de red *eth1*.
* Las aplicaciones virtuales de red personalizadas solo se pueden configurar desde la subred de administración. 
* El tráfico enrutado a la subred de administración no pasa por ellas. En caso contrario, si se produjera un error en ellas, no habría ninguna ruta a la subred de administración para corregirlo.  
* Las máquinas virtuales para la aplicación virtual de red personalizada se colocan en un [conjunto de disponibilidad][availability-set] detrás de un equilibrador de carga. El UDR de la subred de puerta de enlace dirige las solicitudes de aplicación virtual de red al equilibrador de carga.

Incluya una aplicación virtual de red de nivel 7 para terminar las conexiones de aplicación en el nivel de NVA y mantener la afinidad con los niveles de back-end. Así se garantiza una conectividad simétrica en la que el tráfico de respuesta de los niveles de back-end se devuelve a través de la aplicación virtual de red.

Otra opción que tener en cuenta conecta varias aplicaciones virtuales de red en serie, y cada una realiza una tarea de seguridad especializada. Así se permite que cada función de seguridad se administre en función de cada aplicación virtual de red. Por ejemplo, una aplicación virtual de red que implemente un servidor de seguridad podría colocarse en serie con otra que ejecute los servicios de identidad. El inconveniente con respecto a la facilidad de administración es la adición de saltos de red adicionales que pueden aumentar la latencia, por tanto, asegúrese de que esto no afecta al rendimiento de su aplicación.


### <a name="nsg-recommendations"></a>Recomendaciones para las aplicaciones virtuales de red

La puerta de enlace VPN expone una dirección IP pública para la conexión a la red local. Se recomienda crear un grupo de seguridad de red (NSG) para cada subred de aplicación virtual de red entrante, con reglas para bloquear todo el tráfico que no se origine en la red local.

También se recomienda usar grupos de seguridad de red para que cada subred proporcione un segundo nivel de protección contra el tráfico entrante a omitir una puerta de enlace configurada incorrectamente o deshabilitada. Por ejemplo, la subred de nivel web en la arquitectura de referencia implementa un grupo de seguridad de red con una regla para pasar por alto todas las solicitudes que no sean de las recibidas desde la red local (192.168.0.0/16) o la red virtual, y otra regla que pasa por alto todas las solicitudes que no se realizan en el puerto 80.

### <a name="internet-access-recommendations"></a>Recomendaciones de acceso a Internet

[Aplique tunelización forzada][azure-forced-tunneling] a todo el tráfico saliente de Internet a través de la red local mediante el túnel VPN de sitio a sitio para enrutar a Internet con la traducción de direcciones de red (NAT). Así se evita la pérdida accidental de la información confidencial almacenada en el nivel de datos y se permite la inspección y auditoría de todo el tráfico saliente.

> [!NOTE]
> No bloquee por completo el tráfico de Internet de los niveles de aplicación, ya que esto evitará que estos niveles usen los servicios PaaS de Azure que se basan en direcciones IP públicas, como el registro de diagnóstico de máquina virtual, la descarga de extensiones de máquina virtual y otras funciones. Los diagnósticos de Azure también requieren que los componentes puedan leer y escribir en una cuenta de Azure Storage.
> 
> 

Compruebe que el tráfico saliente de Internet se realiza correctamente a través de tunelización forzada. Si utiliza una conexión VPN con el [servicio de acceso remoto y enrutamiento] [ routing-and-remote-access-service] en un servidor local, use una herramienta como [WireShark][wireshark] o [Microsoft Message Analyzer](https://www.microsoft.com/download/details.aspx?id=44226).

### <a name="management-subnet-recommendations"></a>Recomendaciones para la subred de administración

La subred de administración contiene un JumpBox que realiza la funcionalidad de supervisión y administración. Restrinja la ejecución de todas las tareas de administración segura al JumpBox.
 
No cree una dirección IP pública para el JumpBox. En su lugar, cree una ruta para tener acceso al JumpBox a través de la puerta de enlace entrante. Cree reglas NSG para que la subred de administración solo responda a las solicitudes desde la ruta de acceso permitida.

## <a name="scalability-considerations"></a>Consideraciones sobre escalabilidad

La arquitectura de referencia utiliza un equilibrador de carga para dirigir el tráfico de red local a un grupo de dispositivos de aplicaciones virtuales de red, lo que enruta el tráfico. Las aplicaciones virtuales de red se colocan en un [conjunto de disponibilidad][availability-set]. Este diseño permite supervisar el rendimiento de las aplicaciones virtuales de red a lo largo del tiempo y agregar dispositivos NVA en respuesta a los aumentos de carga.

La puerta de enlace VPN de SKU estándar admite un rendimiento sostenido de hasta 100 Mbps. La SKU de alto rendimiento proporciona hasta 200 Mbps. Para anchos de banda mayores, considere la posibilidad de actualizar a una puerta de enlace de ExpressRoute. ExpressRoute proporciona hasta 10 GB/s de ancho de banda con una latencia inferior que una conexión VPN.

Para más información acerca de la escalabilidad de las puertas de enlace de Azure, vea la sección sobre la consideración de la escalabilidad en [Implementing a hybrid network architecture with Azure and on-premises VPN][guidance-vpn-gateway-scalability] (Implementación de una arquitectura de red híbrida con Azure y VPN local) e [Implementación de una arquitectura de red híbrida con Azure y VPN local][guidance-expressroute-scalability] (Implementación de una arquitectura de red híbrida con Azure ExpressRoute).

## <a name="availability-considerations"></a>Consideraciones sobre disponibilidad

Como se mencionó, la arquitectura de referencia utiliza un grupo de dispositivos NVA detrás de un equilibrador de carga. El equilibrador de carga utiliza un sondeo de estado para supervisar cada aplicación virtual de red y quitará la que no responda del grupo.

Si usa Azure ExpressRoute para proporcionar conectividad entre la red local y la red virtual, [configure una puerta de enlace VPN para proporcionar conmutación por error] [ra-vpn-failover] si la conexión ExpressRoute no está disponible.

Para obtener información específica sobre el mantenimiento de la disponibilidad para conexiones VPN y ExpressRoute, consulte las consideraciones de disponibilidad en [Implementing a hybrid network architecture with Azure and on-premises VPN][guidance-vpn-gateway-availability] (Implementación de una arquitectura de red híbrida con Azure y VPN local) e [Implementing a hybrid network architecture with Azure ExpressRoute][guidance-expressroute-availability] (Implementación de una arquitectura de red híbrida con Azure ExpressRoute). 

## <a name="manageability-considerations"></a>Consideraciones sobre la manejabilidad

La supervisión de todos los recursos y aplicaciones debería realizarla el JumpBox en la subred de administración. Dependiendo de los requisitos de la aplicación, puede que necesite recursos adicionales de supervisión en dicha subred. Si es así, se debe tener acceso a estos recursos a través del JumpBox.

Aunque la conectividad de puerta de enlace de la red local a Azure esté fuera de servicio, todavía puede comunicarse con el JumpBox; para ello, implemente una dirección IP pública, agréguela al JumpBox y conéctese de forma remota desde Internet.

La subred de cada nivel de la arquitectura de referencia está protegido por las reglas de NSG. Debe crear una regla para abrir el puerto 3389 para el acceso del Protocolo de escritorio remoto (RDP) en las máquinas virtuales Windows o el puerto 22 para el acceso de shell seguro (SSH) en las máquinas virtuales Linux. Otras herramientas de supervisión y administración pueden requerir reglas para abrir puertos adicionales.

Si está usando ExpressRoute para proporcionar conectividad entre el centro de datos local y Azure, use [Azure Connectivity Toolkit (AzureCT)][azurect] para supervisar y solucionar problemas de conexión.

Puede obtener información adicional destinada en concreto a la supervisión y administración de las conexiones VPN y ExpressRoute en los artículos [Implementing a hybrid network architecture with Azure and on-premises VPN][guidance-vpn-gateway-manageability] (Implementación de una arquitectura de red híbrida con Azure y VPN local) e [Implementing a hybrid network architecture with Azure ExpressRoute][guidance-expressroute-manageability] (Implementación de una arquitectura de red híbrida con Azure ExpressRoute).

## <a name="security-considerations"></a>Consideraciones sobre la seguridad

Esta arquitectura de referencia implementa varios niveles de seguridad.

### <a name="routing-all-on-premises-user-requests-through-the-nva"></a>Enrutamiento de todas las solicitudes de usuario local a través de la aplicación virtual de red
El UDR en la subred de puerta de enlace bloquea todas las solicitudes de usuario distintas de las recibidas desde el entorno local. El UDR pasa las solicitudes permitidas a las aplicaciones virtuales de red en la subred privada de la red perimetral, y estas solicitudes se pasan a la aplicación, si se permiten las reglas de NVA. Puede agregar otras rutas al UDR, pero debe asegurarse de que no omiten accidentalmente las aplicaciones virtuales de red ni bloquean el tráfico administrativo destinado a la subred de administración.

El equilibrador de carga delante de las aplicaciones virtuales de red también actúa como un dispositivo de seguridad al pasar por alto el tráfico en los puertos que no estén abiertos en las reglas de equilibrio de carga. Los equilibradores de carga en la arquitectura de referencia solo escuchan las solicitudes HTTP en el puerto 80 y las solicitudes HTTPS en el puerto 443. Documente las reglas adicionales que agregue a los equilibradores de carga y supervise el tráfico para asegurarse de que no hay ningún problema de seguridad.

### <a name="using-nsgs-to-blockpass-traffic-between-application-tiers"></a>Grupos de seguridad de red para bloquear o permitir el tráfico entre los niveles de aplicación
El tráfico entre niveles se restringe mediante grupos de seguridad de red (NSG). El nivel empresarial bloquea todo el tráfico que no se origina en el nivel web y el de datos bloquea todo el tráfico que no se origina en el nivel empresarial. Si necesita expandir las reglas de NSG a fin de permitir un mayor acceso a estos niveles, sopese estos requisitos con respecto a los riesgos de seguridad. Cada nueva ruta de entrada representa una oportunidad para que se produzca la pérdida accidental o intencionada de datos o la aplicación resulte dañada.

### <a name="devops-access"></a>Acceso de DevOps
Use [RBAC][rbac] para restringir las operaciones que DevOps puede realizar en cada nivel. Al conceder permisos, use el [principio de los privilegios mínimos][security-principle-of-least-privilege]. Registre todas las operaciones administrativas y realice auditorías periódicas para asegurarse de que los cambios de configuración se habían planeado.

## <a name="deploy-the-solution"></a>Implementación de la solución

Se puede encontrar una implementación de una arquitectura de referencia que implementa estas recomendaciones en [GitHub][github-folder]. 

### <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-resources"></a>Implementación de recursos

1. Vaya a la carpeta `/dmz/secure-vnet-hybrid` del repositorio de GitHub de las arquitecturas de referencia.

2. Ejecute el siguiente comando:

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <region> -p onprem.json --deploy
    ```

3. Ejecute el siguiente comando:

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <region> -p secure-vnet-hybrid.json --deploy
    ```

### <a name="connect-the-on-premises-and-azure-gateways"></a>Conexión de las puertas de enlace local y de Azure

En este paso, conectará las dos puertas de enlace de red local.

1. En Azure Portal, vaya al grupo de recursos que ha creado. 

2. Busque el recurso llamado `ra-vpn-vgw-pip` y copie la dirección IP que se muestra en la hoja **Información general**.

3. Busque el recurso llamado `onprem-vpn-lgw`.

4. Haga clic en la hoja **Configuración**. En **Dirección IP**, pegue la dirección IP del paso 2.

    ![](./images/local-net-gw.png)

5. Haga clic en **Guardar** y espere a que finalice la operación. Puede tardar unos 5 minutos.

6. Busque el recurso llamado `onprem-vpn-gateway1-pip`. Copie la dirección IP que se muestra en la hoja **Información general**.

7. Busque el recurso llamado `ra-vpn-lgw`. 

8. Haga clic en la hoja **Configuración**. En **Dirección IP**, pegue la dirección IP del paso 6.

9. Haga clic en **Guardar** y espere a que finalice la operación.

10. Para comprobar la conexión, vaya a la hoja **Conexiones** de cada puerta de enlace. El estado debe ser **Conectado**.

### <a name="verify-that-network-traffic-reaches-the-web-tier"></a>Compruebe que el tráfico de red llega al nivel web

1. En Azure Portal, vaya al grupo de recursos que ha creado. 

2. Busque el recurso llamado `int-dmz-lb`, que es el equilibrador de carga que hay delante de la red perimetral privada. Copie la dirección IP privada de la hoja **Información general**.

3. Busque la máquina virtual denominada `jb-vm1`. Haga clic en **Conectar** y use el escritorio remoto para conectarse a la máquina virtual. El nombre de usuario y la contraseña se especifican en el archivo onprem.json.

4. En la sesión del escritorio remoto, abra un explorador web y vaya a la dirección IP del paso 2. Debería ver la página principal predeterminada del servidor Apache2.

## <a name="next-steps"></a>Pasos siguientes

* Aprenda a implementar una [red perimetral entre Azure e Internet](secure-vnet-dmz.md).
* Aprenda a implementar una [arquitectura de red híbrida de alta disponibilidad][ra-vpn-failover].
* Para más información acerca de cómo administrar la seguridad de red con Azure, consulte [Servicios en la nube de Microsoft y seguridad de red][cloud-services-network-security].
* Para información detallada sobre cómo proteger los recursos de Azure, consulte [Introducción a la seguridad de Microsoft Azure][getting-started-with-azure-security]. 
* Para detalles adicionales acerca de cómo tratar problemas de seguridad en la conexión de puertas de enlace de Azure, vea [Implementación de una arquitectura de red híbrida con Azure y VPN local][guidance-vpn-gateway-security] e [Implementación de una arquitectura de red híbrida con Azure ExpressRoute][guidance-expressroute-security].
* [Solución de problemas de aplicaciones virtuales de red en Azure](/azure/virtual-network/virtual-network-troubleshoot-nva)
  > 

<!-- links -->

[availability-set]: /azure/virtual-machines/virtual-machines-windows-create-availability-set
[azurect]: https://github.com/Azure/NetworkMonitoring/tree/master/AzureCT
[azure-forced-tunneling]: https://azure.microsoft.com/en-gb/documentation/articles/vpn-gateway-forced-tunneling-rm/
[azure-marketplace-nva]: https://azuremarketplace.microsoft.com/marketplace/apps/category/networking
[cloud-services-network-security]: https://azure.microsoft.com/documentation/articles/best-practices-network-security/
[getting-started-with-azure-security]: /azure/security/azure-security-getting-started
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/dmz/secure-vnet-hybrid
[guidance-expressroute]: ../hybrid-networking/expressroute.md
[guidance-expressroute-availability]: ../hybrid-networking/expressroute.md#availability-considerations
[guidance-expressroute-manageability]: ../hybrid-networking/expressroute.md#manageability-considerations
[guidance-expressroute-security]: ../hybrid-networking/expressroute.md#security-considerations
[guidance-expressroute-scalability]: ../hybrid-networking/expressroute.md#scalability-considerations
[guidance-vpn-gateway]: ../hybrid-networking/vpn.md
[guidance-vpn-gateway-availability]: ../hybrid-networking/vpn.md#availability-considerations
[guidance-vpn-gateway-manageability]: ../hybrid-networking/vpn.md#manageability-considerations
[guidance-vpn-gateway-scalability]: ../hybrid-networking/vpn.md#scalability-considerations
[guidance-vpn-gateway-security]: ../hybrid-networking/vpn.md#security-considerations
[ip-forwarding]: /azure/virtual-network/virtual-networks-udr-overview#ip-forwarding
[ra-expressroute]: ../hybrid-networking/expressroute.md
[ra-n-tier]: ../virtual-machines-windows/n-tier.md
[ra-vpn]: ../hybrid-networking/vpn.md
[ra-vpn-failover]: ../hybrid-networking/expressroute-vpn-failover.md
[rbac]: /azure/active-directory/role-based-access-control-configure
[rbac-custom-roles]: /azure/active-directory/role-based-access-control-custom-roles
[routing-and-remote-access-service]: https://technet.microsoft.com/library/dd469790(v=ws.11).aspx
[security-principle-of-least-privilege]: https://msdn.microsoft.com/library/hdb58b2f(v=vs.110).aspx#Anchor_1
[udr-overview]: /azure/virtual-network/virtual-networks-udr-overview
[visio-download]: https://archcenter.blob.core.windows.net/cdn/dmz-reference-architectures.vsdx
[wireshark]: https://www.wireshark.org/
[0]: ./images/dmz-private.png "Arquitectura de red híbrida segura"
