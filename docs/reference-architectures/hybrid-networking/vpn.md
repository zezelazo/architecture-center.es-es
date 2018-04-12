---
title: Conexión de una red local a Azure mediante VPN
description: Procedimiento para implementar una arquitectura de red de sitio a sitio segura que abarque una instancia de Azure Virtual Network y una red local conectada mediante VPN.
author: RohitSharma-pnp
ms.date: 11/28/2016
pnp.series.title: Connect an on-premises network to Azure
pnp.series.next: expressroute
pnp.series.prev: ./index
cardTitle: VPN
ms.openlocfilehash: dafcee6607d9cc7c56c332f9ed5d9568ff70f0e7
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2018
---
# <a name="connect-an-on-premises-network-to-azure-using-a-vpn-gateway"></a>Conexión de una red local a Azure mediante VPN Gateway

Esta arquitectura de referencia muestra cómo extender una red local a Azure mediante una red privada virtual (VPN) de sitio a sitio. El tráfico fluye entre la red local y Azure Virtual Network (VNet) a través de un túnel VPN de IPSec. [**Implemente esta solución**.](#deploy-the-solution)

![[0]][0]

*Descargue un [archivo Visio][visio-download] de esta arquitectura.*

## <a name="architecture"></a>Architecture 

La arquitectura consta de los siguientes componentes:

* **Red local**. Una red de área local privada que se ejecuta dentro de una organización.

* **Dispositivo VPN**. Un dispositivo o servicio que proporciona conectividad externa a la red local. El dispositivo VPN puede ser un dispositivo de hardware, o puede ser una solución de software como el servicio de Enrutamiento y acceso remoto (RRAS) en Windows Server 2012. Para obtener una lista de dispositivos VPN admitidos e información sobre la configuración para conectarlos a Azure VPN Gateway, consulte las instrucciones para el dispositivo seleccionado en el artículo [Acerca de los dispositivos VPN y los parámetros de IPsec/IKE para conexiones de VPN Gateway de sitio a sitio][vpn-appliance].

* **Red virtual**. La aplicación en la nube y los componentes de Azure VPN Gateway se encuentran en la misma red virtual [(VNet)][azure-virtual-network].

* **Azure VPN Gateway**. El servicio [VPN Gateway][azure-vpn-gateway] le permite conectar la red virtual a la red local mediante un dispositivo VPN. Para más información, consulte [Conectar una red local con una red virtual de Microsoft Azure][connect-to-an-Azure-vnet]. VPN Gateway incluye los siguientes elementos:
  
  * **Puerta de enlace de red virtual**. Recurso que proporciona un dispositivo VPN virtual para la red virtual. Es responsable de enrutar el tráfico de la red local a la red virtual.
  * **Puerta de enlace de red local**. Abstracción del dispositivo VPN local. El tráfico de red de la aplicación en la nube a la red local se enruta a través de esta puerta de enlace.
  * **Conexión**. La conexión tiene propiedades que especifican el tipo de conexión (IPSec) y la clave compartida con el dispositivo VPN local para cifrar el tráfico.
  * **Subred de puerta de enlace**. La puerta de enlace de red virtual se mantiene en su propia subred, que está sujeta a los distintos requisitos que se describen a continuación, en la sección Recomendaciones.

* **Aplicación en la nube**. La aplicación hospedada en Azure. Puede incluir varios niveles, con varias subredes que se conectan a través de equilibradores de carga de Azure. Para obtener más información acerca de la infraestructura de aplicaciones, consulte [Ejecutar cargas de trabajo de máquinas virtuales Windows][windows-vm-ra] y [Ejecutar cargas de trabajo de máquinas virtuales Linux][linux-vm-ra].

* **Equilibrador de carga interno**. El tráfico de red de VPN Gateway se enruta a la aplicación en la nube a través de un equilibrador de carga interno. El equilibrador de carga se encuentra en la subred front-end de la aplicación.

## <a name="recommendations"></a>Recomendaciones

Las siguientes recomendaciones sirven para la mayoría de los escenarios. Sígalas a menos que tenga un requisito concreto que las invalide.

### <a name="vnet-and-gateway-subnet"></a>Red virtual y subred de puerta de enlace

Cree una red virtual de Azure con un espacio de direcciones lo suficientemente grande para todos los recursos necesarios. Asegúrese de que el espacio de direcciones de la red virtual tiene suficiente espacio para crecer si es probable que se necesiten máquinas virtuales adicionales en el futuro. El espacio de direcciones de la red virtual no debe superponerse a la red local. Por ejemplo, en el diagrama anterior se usa el espacio de direcciones 10.20.0.0/16 para la red virtual.

Cree una subred denominada *GatewaySubnet*, con un intervalo de direcciones de /27. Esta subred es necesaria para la puerta de enlace de red virtual. La asignación de treinta y dos direcciones a esta subred ayudará a evitar que se alcancen las limitaciones de tamaño de la puerta de enlace en el futuro. Evite también colocar esta subred en el centro del espacio de direcciones. Una práctica recomendada consiste en establecer el espacio de direcciones para la subred de la puerta de enlace en el extremo superior del espacio de direcciones de la red virtual. El ejemplo que aparece en el diagrama usa 10.20.255.224/27.  Este es un procedimiento rápido para calcular el [CIDR]:

1. Establezca los bits variables del espacio de direcciones de la red virtual en 1 hasta alcanzar los bits que usa la subred de puerta de enlace. A continuación, establezca los bits restantes en 0.
2. Convierta los bits resultantes a decimales y expréselos como un espacio de direcciones con la longitud del prefijo establecida en el tamaño de la subred de puerta de enlace.

Por ejemplo, al aplicar el paso 1 anterior a una red virtual con un intervalo de direcciones IP de 10.20.0.0/16, se convierte en 10.20.0b11111111.0b11100000.  Si se convierte en decimales y se expresa como un espacio de direcciones, da como resultado 10.20.255.224/27. 

> [!WARNING]
> No implemente ninguna máquina virtual en la subred de puerta de enlace. Tampoco asigne ningún NSG a esta subred, ya que causaría que la puerta de enlace dejase de funcionar.
> 
> 

### <a name="virtual-network-gateway"></a>Puerta de enlace de red virtual

Asigne una dirección IP pública para la puerta de enlace de la red virtual.

Cree la puerta de enlace de red virtual en la subred de puerta de enlace y asígnele la dirección IP pública recién asignada. Use el tipo de puerta de enlace que mejor se ajuste a sus requisitos y que su dispositivo VPN haya habilitado:

- Cree una [puerta de enlace basada en directivas][policy-based-routing] si necesita controlar estrechamente cómo se enrutan las solicitudes en función de los criterios de las directivas, tales como los prefijos de las direcciones. Las puertas de enlace basadas en directivas utilizan el enrutamiento estático y solo funcionan con conexiones de sitio a sitio.

- Cree una [puerta de enlace basada en rutas][route-based-routing] si se conecta a la red local mediante RRAS, admite conexiones multisitio o entre regiones, o bien si implementa conexiones de red virtual a red virtual (incluidas las rutas que atraviesan varias redes virtuales). Las puertas de enlace basadas en rutas utilizan el enrutamiento dinámico para dirigir el tráfico entre redes. Dado que pueden intentar rutas alternativas, pueden tolerar mejor los errores en la ruta de acceso de red que las rutas estáticas. Las puertas de enlace basadas en rutas también pueden reducir la sobrecarga de administración porque es posible que las rutas no tengan que actualizarse manualmente cuando las direcciones de red cambien.

Para obtener una lista de dispositivos VPN compatibles, consulte [Acerca de los dispositivos VPN y los parámetros de IPsec/IKE para conexiones de VPN Gateway de sitio a sitio][vpn-appliances].

> [!NOTE]
> Tras crear la puerta de enlace, no se puede cambiar entre los tipos de puerta de enlace sin antes eliminarla y volverla a crear.
> 
> 

Seleccione la SKU de Azure VPN Gateway que mejor se ajuste a sus requisitos de rendimiento. Azure VPN Gateway está disponible en tres SKU, que se muestran en la tabla siguiente. 

| SKU | Rendimiento de VPN | Número máximo de túneles IPSec |
| --- | --- | --- |
| Básica |100 Mbps |10 |
| Estándar |100 Mbps |10 |
| Alto rendimiento |200 Mbps |30 |

> [!NOTE]
> La SKU Básica no es compatible con Azure ExpressRoute. También puede [cambiar la SKU][changing-SKUs] tras crear la puerta de enlace.
> 
> 

Se le aplicará un cargo dependiendo del tiempo de aprovisionamiento y de la disponibilidad de la puerta de enlace. Vea [Precios de VPN Gateway][azure-gateway-charges].

En lugar de permitir que las solicitudes pasen directamente a las máquinas virtuales de la aplicación, cree reglas de enrutamiento para la subred de puerta de enlace que dirige el tráfico de aplicaciones entrante desde la puerta de enlace hacia el equilibrador de carga interno.

### <a name="on-premises-network-connection"></a>Conexión de red local

Cree una puerta de enlace de red local. Especifique la dirección IP pública del dispositivo VPN local y el espacio de direcciones de la red local. Tenga en cuenta que el dispositivo VPN local debe tener una dirección IP pública a la cual pueda tener acceso la puerta de enlace de red local de Azure VPN Gateway. El dispositivo VPN no puede ubicarse detrás de un dispositivo de traducción de direcciones de red (NAT).

Cree una conexión de sitio a sitio para la puerta de enlace de red virtual y la puerta de enlace de red local. Seleccione el tipo de conexión de sitio a sitio (IPSec) y especifique la clave compartida. El cifrado de sitio a sitio con Azure VPN Gateway se basa en el protocolo IPSec y utiliza claves compartidas previamente para la autenticación. La clave se especifica al crear la instancia de Azure VPN Gateway. Debe configurar el dispositivo VPN que se ejecuta localmente con la misma clave. Actualmente, no se admiten otros mecanismos de autenticación.

Asegúrese de que la infraestructura de enrutamiento local esté configurada para reenviar al dispositivo VPN las solicitudes destinadas a direcciones de la red virtual de Azure.

Abra los puertos de la red local que requiera la aplicación en la nube.

Pruebe la conexión para comprobar que:

* El dispositivo VPN local enruta correctamente el tráfico a la aplicación en la nube a través de Azure VPN Gateway.
* La red virtual enruta correctamente el tráfico a la red local.
* El tráfico prohibido en ambas direcciones está bloqueado correctamente.

## <a name="scalability-considerations"></a>Consideraciones sobre escalabilidad

Puede lograr una escalabilidad vertical limitada si cambia de las SKU de VPN Gateway Estándar o Básica a la SKU Alto rendimiento de VPN.

Para las redes virtuales que esperan un gran volumen de tráfico VPN, considere la posibilidad de distribuir las diferentes cargas de trabajo en redes virtuales más pequeñas independientes y de configurar una puerta de enlace VPN para cada una de ellas.

Puede dividir la red virtual de forma horizontal o vertical. Para dividirla horizontalmente, mueva algunas instancias de máquina virtual de cada capa a subredes de la red virtual nueva. El resultado es que cada red virtual tiene la misma estructura y funcionalidad. Para dividirla verticalmente, vuelva a diseñar cada capa para dividir la funcionalidad en diferentes áreas lógicas (por ejemplo, gestión de pedidos, facturación, administración de cuentas de clientes, etc.). Cada área funcional se puede colocar en su propia red virtual.

Con la replicación de un controlador de dominio de Active Directory local en la red virtual y la implementación de DNS en la red virtual, puede ayudar a reducir tráfico relacionado con la seguridad y la administración que fluye de la red local a la nube. Para obtener más información, consulte [Extensión de Active Directory Domain Services (AD DS) a Azure][adds-extend-domain].

## <a name="availability-considerations"></a>Consideraciones sobre disponibilidad

Si necesita asegurarse de que la red local sigue estando disponible en Azure VPN Gateway, implemente un clúster de conmutación por error para la instancia de VPN Gateway local.

Si su organización tiene varios sitios locales, cree [conexiones multisitio][vpn-gateway-multi-site] a una o varias redes virtuales de Azure. Este enfoque requiere enrutamiento dinámico (basado en rutas), así que debe asegurarse de que la instancia de VPN Gateway local admite esta característica.

Para obtener más información acerca de los acuerdos de nivel de servicio, consulte [Contrato de nivel de servicio para VPN Gateway][sla-for-vpn-gateway]. 

## <a name="manageability-considerations"></a>Consideraciones sobre la manejabilidad

Supervise la información de diagnóstico de los dispositivos VPN locales. Este proceso depende de las características que proporcione el dispositivo VPN. Por ejemplo, si usa el servicio de Enrutamiento y acceso remoto en Windows Server 2012, consulte [RRAS logging][rras-logging] (Registro de RRAS).

Use los [diagnósticos de Azure VPN Gateway][ gateway-diagnostic-logs] para capturar información sobre problemas de conectividad. Estos registros se pueden utilizar para realizar el seguimiento de información, como el origen y los destinos de las solicitudes de conexión, el protocolo que se utilizó y cómo se estableció la conexión (o por qué el intento fue erróneo).

Supervise los registros operativos de Azure VPN Gateway mediante los registros de auditoría disponibles en Azure Portal. Hay disponibles registros independientes para la puerta de enlace de red local, la puerta de enlace de red de Azure y la conexión. Esta información puede utilizarse para realizar un seguimiento de los cambios realizados en la puerta de enlace y puede resultar útil si una puerta de enlace que funcionaba previamente dejó de hacerlo por algún motivo.

![[2]][2]

Supervise la conectividad y realice un seguimiento de los eventos de error de conectividad. Puede usar un paquete de supervisión como [Nagios][nagios] para capturar y notificar esta información.

## <a name="security-considerations"></a>Consideraciones sobre la seguridad

Genere una clave compartida diferente para cada instancia de VPN Gateway. Use una clave compartida segura para resistir los ataques de fuerza bruta.

> [!NOTE]
> Actualmente, no se puede usar Azure Key Vault para la compartición de claves previa en Azure VPN Gateway.
> 
> 

Asegúrese de que el dispositivo VPN local use un método de cifrado que sea [compatible con Azure VPN Gateway][vpn-appliance-ipsec]. Para el enrutamiento basado en directivas, Azure VPN Gateway admite los algoritmos de cifrado AES256, AES128 y 3DES. Las puertas de enlace basadas en rutas admiten AES256 y 3DES.

Si su dispositivo VPN local se encuentra en una red perimetral (DMZ) que tiene un firewall entre la red perimetral e Internet, es posible que deba configurar [reglas de firewall adicionales][additional-firewall-rules] para permitir la conexión VPN de sitio a sitio.

Si la aplicación en la red virtual envía datos a Internet, considere la posibilidad de [implementar la tunelización forzada][forced-tunneling] para enrutar todo el tráfico vinculado a Internet a través de la red local. Este enfoque le permite auditar las solicitudes salientes que realiza la aplicación desde la infraestructura local.

> [!NOTE]
> La tunelización forzada puede afectar a la conectividad a los servicios de Azure (por ejemplo, el servicio de almacenamiento) y el administrador de licencias de Windows.
> 
> 


## <a name="troubleshooting"></a>solución de problemas 

Para obtener información general sobre cómo solucionar errores comunes relacionados con la VPN, consulte [Troubleshooting common VPN related errors][troubleshooting-vpn-errors] (Solución de errores comunes relacionados con la VPN).

Las recomendaciones siguientes son útiles para determinar si su dispositivo VPN local funciona correctamente.

- **Compruebe si hay errores en cualquier archivo de registro que generó el dispositivo VPN.**

    Esto le ayudará a determinar si el dispositivo VPN funciona correctamente. La ubicación de esta información variará según el dispositivo. Por ejemplo, si está usando RRAS en Windows Server 2012, puede usar el siguiente comando de PowerShell para mostrar información de los eventos de error para el servicio RRAS:

    ```PowerShell
    Get-EventLog -LogName System -EntryType Error -Source RemoteAccess | Format-List -Property *
    ```

    La propiedad *Message* de cada entrada proporciona una descripción del error. Algunos ejemplos comunes:

        - Inability to connect, possibly due to an incorrect IP address specified for the Azure VPN gateway in the RRAS VPN network interface configuration.

        ```
        EventID            : 20111
        MachineName        : on-prem-vm
        Data               : {41, 3, 0, 0}
        Index              : 14231
        Category           : (0)
        CategoryNumber     : 0
        EntryType          : Error
        Message            : RoutingDomainID- {00000000-0000-0000-0000-000000000000}: A demand dial connection to the remote
                             interface AzureGateway on port VPN2-4 was successfully initiated but failed to complete
                             successfully because of the  following error: The network connection between your computer and
                             the VPN server could not be established because the remote server is not responding. This could
                             be because one of the network devices (for example, firewalls, NAT, routers, and so on) between your computer
                             and the remote server is not configured to allow VPN connections. Please contact your
                             Administrator or your service provider to determine which device may be causing the problem.
        Source             : RemoteAccess
        ReplacementStrings : {{00000000-0000-0000-0000-000000000000}, AzureGateway, VPN2-4, The network connection between
                             your computer and the VPN server could not be established because the remote server is not
                             responding. This could be because one of the network devices (for example, firewalls, NAT, routers, and so on)
                             between your computer and the remote server is not configured to allow VPN connections. Please
                             contact your Administrator or your service provider to determine which device may be causing the
                             problem.}
        InstanceId         : 20111
        TimeGenerated      : 3/18/2016 1:26:02 PM
        TimeWritten        : 3/18/2016 1:26:02 PM
        UserName           :
        Site               :
        Container          :
        ```

        - The wrong shared key being specified in the RRAS VPN network interface configuration.

        ```
        EventID            : 20111
        MachineName        : on-prem-vm
        Data               : {233, 53, 0, 0}
        Index              : 14245
        Category           : (0)
        CategoryNumber     : 0
        EntryType          : Error
        Message            : RoutingDomainID- {00000000-0000-0000-0000-000000000000}: A demand dial connection to the remote
                             interface AzureGateway on port VPN2-4 was successfully initiated but failed to complete
                             successfully because of the  following error: Internet key exchange (IKE) authentication credentials are unacceptable.

        Source             : RemoteAccess
        ReplacementStrings : {{00000000-0000-0000-0000-000000000000}, AzureGateway, VPN2-4, IKE authentication credentials are
                             unacceptable.
                             }
        InstanceId         : 20111
        TimeGenerated      : 3/18/2016 1:34:22 PM
        TimeWritten        : 3/18/2016 1:34:22 PM
        UserName           :
        Site               :
        Container          :
        ```

    También puede obtener información de registro de eventos sobre los intentos de conexión a través del servicio RRAS mediante el siguiente comando de PowerShell: 

    ```
    Get-EventLog -LogName Application -Source RasClient | Format-List -Property *
    ```

    Si se produce un error en la conexión, este registro contendrá errores con un aspecto similar al siguiente:

    ```
    EventID            : 20227
    MachineName        : on-prem-vm
    Data               : {}
    Index              : 4203
    Category           : (0)
    CategoryNumber     : 0
    EntryType          : Error
    Message            : CoId={B4000371-A67F-452F-AA4C-3125AA9CFC78}: The user SYSTEM dialed a connection named
                         AzureGateway that has failed. The error code returned on failure is 809.
    Source             : RasClient
    ReplacementStrings : {{B4000371-A67F-452F-AA4C-3125AA9CFC78}, SYSTEM, AzureGateway, 809}
    InstanceId         : 20227
    TimeGenerated      : 3/18/2016 1:29:21 PM
    TimeWritten        : 3/18/2016 1:29:21 PM
    UserName           :
    Site               :
    Container          :
    ```

- **Compruebe la conectividad y el enrutamiento en VPN Gateway.**

    Es posible que el dispositivo VPN no enrute correctamente el tráfico a través de Azure VPN Gateway. Use una herramienta como [PsPing][psping] para comprobar la conectividad y el enrutamiento en VPN Gateway. Por ejemplo, para probar la conectividad de una máquina local a un servidor web que se encuentre en la red virtual, ejecute el siguiente comando (reemplazando `<<web-server-address>>` por la dirección del servidor web):

    ```
    PsPing -t <<web-server-address>>:80
    ```

    Si la máquina local puede enrutar el tráfico al servidor web, debería ver un resultado similar al siguiente:

    ```
    D:\PSTools>psping -t 10.20.0.5:80

    PsPing v2.01 - PsPing - ping, latency, bandwidth measurement utility
    Copyright (C) 2012-2014 Mark Russinovich
    Sysinternals - www.sysinternals.com

    TCP connect to 10.20.0.5:80:
    Infinite iterations (warmup 1) connecting test:
    Connecting to 10.20.0.5:80 (warmup): 6.21ms
    Connecting to 10.20.0.5:80: 3.79ms
    Connecting to 10.20.0.5:80: 3.44ms
    Connecting to 10.20.0.5:80: 4.81ms

      Sent = 3, Received = 3, Lost = 0 (0% loss),
      Minimum = 3.44ms, Maximum = 4.81ms, Average = 4.01ms
    ```

    Si la máquina local no puede comunicarse con el destino especificado, verá mensajes similares al siguiente:

    ```
    D:\PSTools>psping -t 10.20.1.6:80

    PsPing v2.01 - PsPing - ping, latency, bandwidth measurement utility
    Copyright (C) 2012-2014 Mark Russinovich
    Sysinternals - www.sysinternals.com

    TCP connect to 10.20.1.6:80:
    Infinite iterations (warmup 1) connecting test:
    Connecting to 10.20.1.6:80 (warmup): This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80: This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80: This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80: This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80:
      Sent = 3, Received = 0, Lost = 3 (100% loss),
      Minimum = 0.00ms, Maximum = 0.00ms, Average = 0.00ms
    ```

- **Compruebe que el firewall local permita que pase el tráfico VPN y que estén abiertos los puertos correctos.**

- **Compruebe que el dispositivo VPN local use un método de cifrado que sea [compatible con Azure VPN Gateway][vpn-appliance].** Para el enrutamiento basado en directivas, Azure VPN Gateway admite los algoritmos de cifrado AES256, AES128 y 3DES. Las puertas de enlace basadas en rutas admiten AES256 y 3DES.

Las recomendaciones siguientes son útiles para determinar si hay algún problema con Azure VPN Gateway:

- **Examine los [registros de diagnóstico de Azure VPN Gateway][gateway-diagnostic-logs] para identificar posibles problemas.**

- **Compruebe que Azure VPN Gateway y el dispositivo VPN local estén configurados con la misma clave de autenticación compartida.**

    Puede ver la clave compartida almacenada en Azure VPN Gateway mediante el siguiente comando de la CLI de Azure:

    ```
    azure network vpn-connection shared-key show <<resource-group>> <<vpn-connection-name>>
    ```

    Use el comando adecuado para que su dispositivo VPN local muestre la clave compartida que se configuró.

    Compruebe que la subred *GatewaySubnet* que contiene la Azure VPN Gateway no esté asociada con un NSG.

    Puede ver los detalles de la subred con el siguiente comando de la CLI de Azure:

    ```
    azure network vnet subnet show -g <<resource-group>> -e <<vnet-name>> -n GatewaySubnet
    ```

    Asegúrese de que no hay ningún campo de datos denominado *Network Security Group id*. En el ejemplo siguiente se muestran los resultados de una instancia de *GatewaySubnet* que tiene un NSG asignado (*VPN-Gateway-Group*). Esto puede impedir que la puerta de enlace funcione correctamente si no hay ninguna regla definida para este NSG.

    ```
    C:\>azure network vnet subnet show -g profx-prod-rg -e profx-vnet -n GatewaySubnet
        info:    Executing command network vnet subnet show
        + Looking up virtual network "profx-vnet"
        + Looking up the subnet "GatewaySubnet"
        data:    Id                              : /subscriptions/########-####-####-####-############/resourceGroups/profx-prod-rg/providers/Microsoft.Network/virtualNetworks/profx-vnet/subnets/GatewaySubnet
        data:    Name                            : GatewaySubnet
        data:    Provisioning state              : Succeeded
        data:    Address prefix                  : 10.20.3.0/27
        data:    Network Security Group id       : /subscriptions/########-####-####-####-############/resourceGroups/profx-prod-rg/providers/Microsoft.Network/networkSecurityGroups/VPN-Gateway-Group
        info:    network vnet subnet show command OK
    ```

- **Compruebe que las máquinas virtuales en la red virtual de Azure estén configuradas para permitir el tráfico procedente de fuera de la red virtual.**

    Compruebe si hay alguna regla de NSG asociada con las subredes que contienen estas máquinas virtuales. Puede ver todas las reglas de NSG con el siguiente comando de la CLI de Azure:

    ```
    azure network nsg show -g <<resource-group>> -n <<nsg-name>>
    ```

- **Compruebe que Azure VPN Gateway esté conectado.**

    Puede usar el siguiente comando de Azure PowerShell para comprobar el estado actual de la conexión VPN de Azure. El parámetro `<<connection-name>>` es el nombre de la conexión VPN de Azure que vincula la puerta de enlace de red virtual y la puerta de enlace local.

    ```
    Get-AzureRmVirtualNetworkGatewayConnection -Name <<connection-name>> - ResourceGroupName <<resource-group>>
    ```

    Los fragmentos de código siguientes resaltan la salida que se genera si la puerta de enlace está conectada (primer ejemplo) o desconectada (segundo ejemplo):

    ```
    PS C:\> Get-AzureRmVirtualNetworkGatewayConnection -Name profx-gateway-connection -ResourceGroupName profx-prod-rg

    AuthorizationKey           :
    VirtualNetworkGateway1     : Microsoft.Azure.Commands.Network.Models.PSVirtualNetworkGateway
    VirtualNetworkGateway2     :
    LocalNetworkGateway2       : Microsoft.Azure.Commands.Network.Models.PSLocalNetworkGateway
    Peer                       :
    ConnectionType             : IPsec
    RoutingWeight              : 0
    SharedKey                  : ####################################
    ConnectionStatus           : Connected
    EgressBytesTransferred     : 55254803
    IngressBytesTransferred    : 32227221
    ProvisioningState          : Succeeded
    ...
    ```

    ```
    PS C:\> Get-AzureRmVirtualNetworkGatewayConnection -Name profx-gateway-connection2 -ResourceGroupName profx-prod-rg

    AuthorizationKey           :
    VirtualNetworkGateway1     : Microsoft.Azure.Commands.Network.Models.PSVirtualNetworkGateway
    VirtualNetworkGateway2     :
    LocalNetworkGateway2       : Microsoft.Azure.Commands.Network.Models.PSLocalNetworkGateway
    Peer                       :
    ConnectionType             : IPsec
    RoutingWeight              : 0
    SharedKey                  : ####################################
    ConnectionStatus           : NotConnected
    EgressBytesTransferred     : 0
    IngressBytesTransferred    : 0
    ProvisioningState          : Succeeded
    ...
    ```

Las recomendaciones siguientes son útiles para determinar si hay algún problema con el uso del ancho de banda, la configuración de la máquina virtual host o el rendimiento de la aplicación:

- **Compruebe que el firewall del sistema operativo invitado que se ejecuta en las máquinas virtuales de Azure en la subred esté configurado correctamente para permitir el tráfico de los intervalos de IP locales.**

- **Compruebe que el volumen de tráfico no se acerque al límite del ancho de banda disponible en Azure VPN Gateway.**

    La manera de comprobarlo depende del dispositivo VPN que se ejecute localmente. Por ejemplo, si está usando RRAS en Windows Server 2012, puede usar el Monitor de rendimiento para el seguimiento del volumen de datos que se recibe y se transmite a través de la conexión VPN. Mediante el objeto *Total de RAS*, seleccione los contadores *Bytes recibidos/s* y *Bytes transmitidos/s*:

    ![[3]][3]

    Debería comparar los resultados con el ancho de banda disponible para VPN Gateway (100 Mbps para las SKU Estándar y Básica, y 200 Mbps para la SKU Alto rendimiento):

    ![[4]][4]

- **Compruebe que implementó la cantidad y el tamaño correctos de máquinas virtuales para la carga de la aplicación.**

    Determine si alguna de las máquinas virtuales de la red virtual de Azure se ejecuta con lentitud. Si es así, es posible que estén sobrecargadas, que sean insuficientes para controlar la carga o que los equilibradores de carga no estén configurados correctamente. Para averiguarlo, consulte [Microsoft Azure Virtual Machine Monitoring with Azure Diagnostics Extension][azure-vm-diagnostics] (Supervisión de máquinas virtuales de Microsoft Azure mediante la extensión de Azure Diagnostics). Puede examinar los resultados mediante Azure Portal, aunque también existen muchas herramientas de terceros que pueden proporcionar información detallada sobre los datos de rendimiento.

- **Compruebe que la aplicación haga un uso eficaz de los recursos de nube.**

    Instrumente el código de aplicación que se ejecuta en cada máquina virtual para determinar si las aplicaciones están sacando el máximo partido de los recursos. Puede usar herramientas como [Application Insights][application-insights].

## <a name="deploy-the-solution"></a>Implementación de la solución


**Requisitos previos**. Debe tener una infraestructura local existente ya configurada con un dispositivo de red adecuado.

Para implementar la solución, siga estos pasos:

1. Haga clic en el botón a continuación:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fvpn%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. Espere a que el vínculo abra Azure Portal y, a continuación, siga estos pasos: 
   * El nombre del **Grupo de recursos** ya está definido en el archivo de parámetros, así que seleccione **Crear nuevo** y escriba `ra-hybrid-vpn-rg` en el cuadro de texto.
   * Seleccione la región en el cuadro de lista desplegable **Ubicación**.
   * No modifique los cuadros de texto **URI raíz de plantilla** o **URI raíz de parámetro**.
   * Revise los términos y condiciones, y haga clic en la casilla **Acepto los términos y condiciones indicados anteriormente**.
   * Haga clic en el botón **Comprar**.
3. Espere a que la implementación se complete.



<!-- links -->

[adds-extend-domain]: ../identity/adds-extend-domain.md
[expressroute]: ../hybrid-networking/expressroute.md
[windows-vm-ra]: ../virtual-machines-windows/index.md
[linux-vm-ra]: ../virtual-machines-linux/index.md

[naming conventions]: /azure/guidance/guidance-naming-conventions

[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[arm-templates]: /azure/resource-group-authoring-templates
[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-portal]: /azure/azure-portal/resource-group-portal
[azure-powershell]: /azure/powershell-azure-resource-manager
[azure-virtual-network]: /azure/virtual-network/virtual-networks-overview
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[azure-vpn-gateway]: https://azure.microsoft.com/services/vpn-gateway/
[azure-gateway-charges]: https://azure.microsoft.com/pricing/details/vpn-gateway/
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[vpn-gateway-multi-site]: /azure/vpn-gateway/vpn-gateway-multi-site
[policy-based-routing]: https://en.wikipedia.org/wiki/Policy-based_routing
[route-based-routing]: https://en.wikipedia.org/wiki/Static_routing
[network-security-group]: /azure/virtual-network/virtual-networks-nsg
[sla-for-vpn-gateway]: https://azure.microsoft.com/support/legal/sla/vpn-gateway/v1_2/
[additional-firewall-rules]: https://technet.microsoft.com/library/dn786406.aspx#firewall
[nagios]: https://www.nagios.org/
[azure-vpn-gateway-diagnostics]: http://blogs.technet.com/b/keithmayer/archive/2014/12/18/diagnose-azure-virtual-network-vpn-connectivity-issues-with-powershell.aspx
[ping]: https://technet.microsoft.com/library/ff961503.aspx
[tracert]: https://technet.microsoft.com/library/ff961507.aspx
[psping]: http://technet.microsoft.com/sysinternals/jj729731.aspx
[nmap]: http://nmap.org
[changing-SKUs]: https://azure.microsoft.com/blog/azure-virtual-network-gateway-improvements/
[gateway-diagnostic-logs]: http://blogs.technet.com/b/keithmayer/archive/2015/12/07/step-by-step-capturing-azure-resource-manager-arm-vnet-gateway-diagnostic-logs.aspx
[troubleshooting-vpn-errors]: https://blogs.technet.microsoft.com/rrasblog/2009/08/12/troubleshooting-common-vpn-related-errors/
[rras-logging]: https://www.petri.com/enable-diagnostic-logging-in-windows-server-2012-r2-routing-and-remote-access
[create-on-prem-network]: https://technet.microsoft.com/library/dn786406.aspx#routing
[azure-vm-diagnostics]: https://azure.microsoft.com/blog/windows-azure-virtual-machine-monitoring-with-wad-extension/
[application-insights]: /azure/application-insights/app-insights-overview-usage
[forced-tunneling]: https://azure.microsoft.com/documentation/articles/vpn-gateway-about-forced-tunneling/
[vpn-appliances]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-architectures.vsdx
[vpn-appliance-ipsec]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices#ipsec-parameters
<!--[solution-script]: https://github.com/mspnp/reference-architectures/tree/master/guidance-hybrid-network-vpn/Deploy-ReferenceArchitecture.ps1-->
<!--[solution-script-bash]: https://github.com/mspnp/reference-architectures/tree/master/guidance-hybrid-network-vpn/deploy-reference-architecture.sh-->
<!--[virtualNetworkGateway-parameters]: https://github.com/mspnp/reference-architectures/tree/master/guidance-hybrid-network-vpn/parameters/virtualNetworkGateway.parameters.json-->
[azure-cli]: https://azure.microsoft.com/documentation/articles/xplat-cli-install/
[CIDR]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[0]: ./images/vpn.png "Red híbrida que abarca las infraestructuras de Azure y local"
[2]: ../_images/guidance-hybrid-network-vpn/audit-logs.png "Registros de auditoría en Azure Portal"
[3]: ../_images/guidance-hybrid-network-vpn/RRAS-perf-counters.png "Contadores de rendimiento para supervisar el tráfico de red VPN"
[4]: ../_images/guidance-hybrid-network-vpn/RRAS-perf-graph.png "Ejemplo de gráfico de rendimiento de red VPN"