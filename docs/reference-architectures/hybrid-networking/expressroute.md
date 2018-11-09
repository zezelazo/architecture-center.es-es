---
title: Conexión de una red local a Azure mediante ExpressRoute
description: Procedimiento para implementar una arquitectura de red de sitio a sitio segura que abarque una instancia de Azure Virtual Network y una red local conectada mediante Azure ExpressRoute.
author: telmosampaio
ms.date: 10/22/2017
pnp.series.title: Connect an on-premises network to Azure
pnp.series.next: expressroute-vpn-failover
pnp.series.prev: vpn
cardTitle: ExpressRoute
ms.openlocfilehash: 16711acb179c05152fc5ef8c7bf3eeb8d067a382
ms.sourcegitcommit: dbbf914757b03cdee7a274204f9579fa63d7eed2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/02/2018
ms.locfileid: "50916624"
---
# <a name="connect-an-on-premises-network-to-azure-using-expressroute"></a>Conexión de una red local a Azure mediante ExpressRoute

Esta arquitectura de referencia muestra cómo conectar una red local a las redes virtuales en Azure, mediante [Azure ExpressRoute][expressroute-introduction]. Las conexiones de ExpressRoute utilizan una conexión privada y dedicada a través de un proveedor de conectividad de terceros. La conexión privada extiende la red local en Azure. [**Implemente esta solución**.](#deploy-the-solution)

![[0]][0]

*Descargue un [archivo Visio][visio-download] de esta arquitectura.*

## <a name="architecture"></a>Arquitectura

La arquitectura consta de los siguientes componentes:

* **Red corporativa local**. Una red de área local privada que se ejecuta dentro de una organización.

* **Circuito ExpressRoute**. Un circuito de capa 2 o capa 3 suministrado por el proveedor de conectividad que se une a la red local con Azure a través de los enrutadores perimetrales. El circuito usa la infraestructura de hardware administrada por el proveedor de conectividad.

* **Enrutadores perimetrales locales**. Enrutadores que conectan la red local al circuito administrado por el proveedor. En función de cómo se aprovisione la conexión, debe proporcionar las direcciones IP públicas utilizadas por los enrutadores.
* **Enrutadores perimetrales de Microsoft**. Dos enrutadores en una configuración de alta disponibilidad activa/activa. Estos enrutadores permiten a un proveedor de conectividad conectar sus circuitos directamente a su centro de datos. En función de cómo se aprovisione la conexión, debe proporcionar las direcciones IP públicas utilizadas por los enrutadores.

* **Redes virtuales Azure (VNets)**. Cada red virtual se encuentra en una sola región de Azure y puede hospedar varios niveles de aplicación. Los niveles de aplicación se pueden segmentar con subredes en cada red virtual.

* **Servicios públicos de Azure**. Servicios de Azure que pueden usarse en una aplicación híbrida. Estos servicios también están disponibles a través de Internet, pero acceder a ellos mediante un circuito ExpressRoute proporciona baja latencia y un rendimiento más predecible, ya que el tráfico no pasa a través de Internet. Las conexiones se realizan mediante [emparejamiento público][expressroute-peering], con direcciones que son propiedad de su organización o proporcionadas por el proveedor de conectividad.

* **Servicios de Office 365**. Aplicaciones y servicios de Office 365 disponibles públicamente y proporcionados por Microsoft. Las conexiones se realizan mediante [emparejamiento de Microsoft][expressroute-peering], con direcciones que son propiedad de su organización o proporcionadas por el proveedor de conectividad. También puede conectarse directamente a Microsoft CRM Online a través del emparejamiento de Microsoft.

* **Proveedores de conectividad** (no se muestran). Empresas que proporcionan una conexión a través de conectividad de nivel 2 o de nivel 3 entre el centro de datos y un centro de datos de Azure.

## <a name="recommendations"></a>Recomendaciones

Las siguientes recomendaciones sirven para la mayoría de los escenarios. Sígalas a menos que tenga un requisito concreto que las invalide.

### <a name="connectivity-providers"></a>Proveedores de conectividad

Seleccione un proveedor de conectividad de ExpressRoute adecuado para su ubicación. Para obtener una lista de los proveedores de conectividad disponibles en su ubicación, utilice el siguiente comando de Azure PowerShell:

```powershell
Get-AzureRmExpressRouteServiceProvider
```

Los proveedores de conectividad de ExpressRoute conectan el centro de datos a Microsoft de las maneras siguientes:

* **Ubicación compartida en un intercambio en la nube**. Si comparte la ubicación en la misma instalación con un intercambio en la nube, puede solicitar conexiones cruzadas virtuales a Azure a través del intercambio de Ethernet del proveedor de la ubicación compartida. Los proveedores de ubicación compartida pueden ofrecer conexiones cruzadas de nivel 2, o conexiones cruzadas de nivel 3 administradas entre la infraestructura en la instalación de ubicación compartida y Azure.
* **Conexiones Ethernet de punto a punto**. Puede conectar los centros de datos u oficinas locales a Azure a través de vínculos de Ethernet de punto a punto. Los proveedores de Ethernet de punto a punto pueden ofrecer conexiones de nivel 2 o de nivel 3 administradas entre el sitio y Azure.
* **Redes de conexión universal (IPVPN)**. Puede integrar la red de área extensa (WAN) con Azure. Los proveedores de red privada virtual del protocolo de Internet (IPVPN), normalmente una VPN de conmutación de etiquetas multiprotocolo, ofrecen una conectividad universal entre los centros de datos y las sucursales. Azure puede estar conectada a la WAN de forma que parezca otra sucursal más. Los proveedores de WAN normalmente ofrecen una conectividad de nivel 3 administrada.

Para obtener más información acerca de los proveedores de conectividad, consulte la [Introducción a ExpressRoute][expressroute-introduction].

### <a name="expressroute-circuit"></a>Circuito ExpressRoute

Compruebe que su organización cumple los [requisitos previos de ExpressRoute][expressroute-prereqs] para conectarse a Azure.

Si aún no lo ha hecho, agregue una subred con el nombre `GatewaySubnet` a la red virtual de Azure y cree una puerta de enlace de red virtual de ExpressRoute con el servicio de puerta de enlace de VPN de Azure. Para más información sobre este proceso, vea [Flujos de trabajo de ExpressRoute para aprovisionamiento de circuitos y estados de circuitos][ExpressRoute-provisioning].

Cree un circuito ExpressRoute como se indica a continuación:

1. Ejecute el siguiente comando de PowerShell:
   
    ```powershell
    New-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>> -Location <<location>> -SkuTier <<sku-tier>> -SkuFamily <<sku-family>> -ServiceProviderName <<service-provider-name>> -PeeringLocation <<peering-location>> -BandwidthInMbps <<bandwidth-in-mbps>>
    ```
2. Envíe el valor de `ServiceKey` para el circuito nuevo al proveedor de servicios.

3. Espere a que el proveedor aprovisione el circuito. Para comprobar el estado de aprovisionamiento de un circuito, ejecute el siguiente comando de PowerShell:
   
    ```powershell
    Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
    ```

    El campo `Provisioning state` de la sección `Service Provider` de la salida cambiará de `NotProvisioned` a `Provisioned` cuando el circuito esté listo.

    > [!NOTE]
    > Si usa una conexión de nivel 3, el proveedor debe configurar y administrar el enrutamiento en su lugar. Proporcione la información necesaria para permitir que el proveedor implemente las rutas adecuadas.
    > 
    > 

4. Si usa una conexión de nivel 2:

    1. Reserve dos subredes /30 compuestas de direcciones IP públicas válidas para cada tipo de emparejamiento que desee implementar. Estas subredes /30 se usarán para proporcionar direcciones IP para los enrutadores utilizados para el circuito. Si va a implementar el emparejamiento de Microsoft público o privado, necesitará seis subredes /30 con direcciones IP públicas válidas.     

    2. Configure el enrutamiento para el circuito ExpressRoute. Ejecute los siguientes comandos de PowerShell para cada tipo de emparejamiento que desee configurar (privado, público y Microsoft). Para más información, vea [Creación y modificación del enrutamiento de un circuito ExpressRoute][configure-expressroute-routing].
   
        ```powershell
        Set-AzureRmExpressRouteCircuitPeeringConfig -Name <<peering-name>> -Circuit <<circuit-name>> -PeeringType <<peering-type>> -PeerASN <<peer-asn>> -PrimaryPeerAddressPrefix <<primary-peer-address-prefix>> -SecondaryPeerAddressPrefix <<secondary-peer-address-prefix>> -VlanId <<vlan-id>>

        Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit <<circuit-name>>
        ```

    3. Reserve otro grupo de direcciones IP públicas válidas para usarlas en la traducción de direcciones de red (NAT) para el emparejamiento público y de Microsoft. Se recomienda tener un grupo diferente para cada emparejamiento. Especifique el grupo para el proveedor de conectividad, de modo que puedan configurar los anuncios del Protocolo de puerta de enlace fronteriza (BGP) para esos intervalos.

5. Ejecute los siguientes comandos de PowerShell para vincular sus redes virtuales privadas al circuito ExpressRoute. Para más información, consulte el artículo [Vinculación de la red virtual a circuitos ExpressRoute][link-vnet-to-expressroute].

    ```powershell
    $circuit = Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
    $gw = Get-AzureRmVirtualNetworkGateway -Name <<gateway-name>> -ResourceGroupName <<resource-group>>
    New-AzureRmVirtualNetworkGatewayConnection -Name <<connection-name>> -ResourceGroupName <<resource-group>> -Location <<location> -VirtualNetworkGateway1 $gw -PeerId $circuit.Id -ConnectionType ExpressRoute
    ```

Puede conectar varias redes virtuales que se encuentren en regiones diferentes al mismo circuito ExpressRoute, siempre y cuando todos ellos se encuentren en la misma región geopolítica.

### <a name="troubleshooting"></a>solución de problemas 

Si un circuito de ExpressRoute que funcionaba ahora no se puede conectar, en ausencia de cambios de configuración local o dentro de la red virtual privada, debe ponerse en contacto con el proveedor de conectividad y colaborar para corregir el problema. Use los siguientes comandos de Powershell para verificar que se haya aprovisionado el circuito de ExpressRoute:

```powershell
Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
```

El resultado de este comando muestra varias propiedades para el circuito, incluidas `ProvisioningState`, `CircuitProvisioningState` y `ServiceProviderProvisioningState`, tal y como se muestra a continuación.

```
ProvisioningState                : Succeeded
Sku                              : {
                                     "Name": "Standard_MeteredData",
                                     "Tier": "Standard",
                                     "Family": "MeteredData"
                                   }
CircuitProvisioningState         : Enabled
ServiceProviderProvisioningState : NotProvisioned
```

Si la propiedad `ProvisioningState` no está establecida en `Succeeded` después de intentar crear un nuevo circuito, quite el circuito mediante el comando siguiente e intente crearlo de nuevo.

```powershell
Remove-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
```

Si el proveedor ya tenía aprovisionado el circuito y la propiedad `ProvisioningState` ya está establecida en `Failed`, o la propiedad `CircuitProvisioningState` no es `Enabled`, póngase en contacto con su proveedor para obtener más ayuda.

## <a name="scalability-considerations"></a>Consideraciones sobre escalabilidad

Los circuitos de ExpressRoute proporcionan una ruta de acceso de gran ancho de banda entre redes. Por lo general, cuanto mayor sea el ancho de banda mayor será el costo. 

ExpressRoute ofrece dos [planes de precios][expressroute-pricing] a los clientes, un plan de uso medido y un plan de servicio de datos ilimitado. Los cargos pueden variar según el ancho de banda del circuito. Es probable que el ancho de banda disponible varíe de un proveedor a otro. Use el cmdlet `Get-AzureRmExpressRouteServiceProvider` para ver los proveedores disponibles en su región y los anchos de banda que ofrecen.
 
Un solo circuito ExpressRoute puede admitir un número determinado de emparejamientos y vínculos de red virtual. Para más información, consulte el artículo [Límites de ExpressRoute](/azure/azure-subscription-service-limits).

Por un costo adicional, el complemento Premium de ExpressRoute proporciona más capacidad:

* Mayor límite de rutas para el emparejamiento público y privado. 
* Aumento del número de vínculos de red virtual por cada circuito ExpressRoute. 
* Conectividad global para los servicios.

Vea [Precios de ExpressRoute][expressroute-pricing] para conocer más detalles. 

Los circuitos ExpressRoute están diseñados para permitir hasta dos veces el límite de ancho de banda adquirido sin costo adicional. Esto se logra mediante el uso de vínculos redundantes. Sin embargo, no todos los proveedores de conectividad admiten esta característica. Verifique que el proveedor de conectividad habilita esta característica antes de depender de ella.

Aunque algunos proveedores le permiten cambiar el ancho de banda, asegúrese de elegir uno inicial que sobrepase sus necesidades y proporcione espacio para el crecimiento. Si necesita aumentar el ancho de banda en el futuro, le quedan dos opciones:

- Aumentar el ancho de banda. Debe evitar esta opción en la medida de lo posible; no todos los proveedores le permiten aumentar dinámicamente el ancho de banda. Pero, si se necesita aumentarlo, póngase en contacto con su proveedor para verificar que permite variar las propiedades de ancho de banda de ExpressRoute a través de comandos de Powershell. Si es así, ejecute los comandos siguientes.

    ```powershell
    $ckt = Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
    $ckt.ServiceProviderProperties.BandwidthInMbps = <<bandwidth-in-mbps>>
    Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
    ```

    Puede aumentar el ancho de banda sin perder conectividad. Degradar el ancho de banda provocará una interrupción en la conectividad, ya que debe eliminar el circuito y volver a crearlo con la nueva configuración.

- Cambiar el plan de precios o actualizar a Premium. Para ello, ejecute los siguientes comandos: La propiedad `Sku.Tier` puede ser `Standard` o `Premium`; la propiedad `Sku.Name` puede ser `MeteredData` o `UnlimitedData`.

    ```powershell
    $ckt = Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>

    $ckt.Sku.Tier = "Premium"
    $ckt.Sku.Family = "MeteredData"
    $ckt.Sku.Name = "Premium_MeteredData"

    Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
    ```

    > [!IMPORTANT]
    > Asegúrese de que la propiedad `Sku.Name` concuerda con las propiedades `Sku.Tier` y `Sku.Family`. Si cambia la familia y el nivel, pero no el nombre, la conexión se deshabilitará.
    > 
    > 

    Puede actualizar la SKU sin interrupciones, pero no se puede cambiar del plan de precios ilimitado al limitado. Al degradar la SKU, el consumo de ancho de banda debe permanecer en el límite predeterminado de la SKU estándar.

## <a name="availability-considerations"></a>Consideraciones sobre disponibilidad

ExpressRoute no admite protocolos de redundancia de enrutador como el Protocolo de enrutamiento en espera activa (HSRP) y el Protocolo de redundancia de enrutador virtual (VRRP) para implementar la alta disponibilidad. En su lugar, utiliza un par redundante de sesiones BGP por emparejamiento. Para facilitar las conexiones de alta disponibilidad a la red, Azure proporciona dos puertos de redundancia en los dos enrutadores (parte de Microsoft Edge) en una configuración activa/activa.

De forma predeterminada, las sesiones BGP usan un valor de tiempo de espera de inactividad de 60 segundos. Si una sesión agota el tiempo de espera tres veces (180 segundos en total), el enrutador se marca como no disponible y todo el tráfico se redirige al enrutador restante. Este tiempo de espera de 180 segundos puede ser demasiado largo para las aplicaciones críticas. Si es así, puede cambiar la configuración del tiempo de espera BGP en el enrutador local con un valor menor.

Puede configurar la alta disponibilidad para la conexión de Azure de maneras diferentes, dependiendo del tipo de proveedor que utilice y del número de circuitos ExpressRoute y de conexiones de puerta de enlace de red virtual que esté dispuesto a configurar. A continuación se resumen las opciones de disponibilidad:

* Si usa una conexión de nivel 2, implemente enrutadores redundantes en su red local en una configuración activa/activa. Conecte el circuito principal a un enrutador y el circuito secundario al otro. Esto le proporcionará una conexión de alta disponibilidad en ambos extremos de la conexión. Esto es necesario si requiere el Acuerdo de Nivel de Servicio (SLA) de ExpressRoute. Vea [SLA de Azure ExpressRoute][sla-for-expressroute] para obtener más detalles.

    El siguiente diagrama muestra una configuración con enrutadores redundantes locales conectados a los circuitos principales y secundarios. Cada circuito controla el tráfico de un emparejamiento público y de un emparejamiento privado (cada uno se designa con un par de espacios de direcciones /30, tal como se describe en la sección anterior).

    ![[1]][1]

* Si usa una conexión de nivel 3, verifique que proporciona sesiones BGP redundantes que controlan la disponibilidad en su lugar.

* Conecte la red virtual a varios circuitos ExpressRoute, proporcionados por otros proveedores de servicios. Esta estrategia proporciona funcionalidades de recuperación ante desastres y alta disponibilidad adicional.

* Configure una VPN de sitio a sitio como una ruta de acceso de conmutación por error para ExpressRoute. Para más información sobre esta opción, vea [Conexión de una red local a Azure mediante ExpressRoute con conmutación por error de VPN][highly-available-network-architecture].
 Esta opción solo se aplica al emparejamiento privado. Para los servicios de Azure y Office 365, Internet es la única ruta de acceso para la conmutación por error. 

## <a name="manageability-considerations"></a>Consideraciones sobre la manejabilidad

Puede usar [Azure Connectivity Toolkit (AzureCT)][azurect] para supervisar la conectividad entre el centro de datos local y Azure. 

## <a name="security-considerations"></a>Consideraciones sobre la seguridad

Puede configurar las opciones de seguridad para la conexión de Azure de maneras diferentes, en función de las necesidades de cumplimiento de normas y de los problemas de seguridad. 

ExpressRoute funciona en el nivel 3. Las amenazas del nivel de aplicación pueden evitarse mediante el uso de un dispositivo de seguridad de red que restrinja el tráfico a los recursos legítimos. Además, las conexiones de ExpressRoute con el emparejamiento público solo pueden iniciarse desde el entorno local. Así se evita que un servicio malicioso obtenga acceso y ponga en peligro los datos locales desde Internet.

Para maximizar la seguridad, agregue dispositivos de seguridad de red entre la red local y los enrutadores perimetrales del proveedor. Esto le ayudará a impedir el flujo de entrada de tráfico no autorizado de la red virtual:

![[2]][2]

Para fines relacionados con la auditoría y el cumplimiento de normas, puede ser necesario prohibir el acceso directo a Internet de los componentes que se ejecutan en la red virtual e implementar la [tunelización forzada][forced-tuneling]. En esta situación, se debe redirigir el tráfico de Internet a través de un proxy que se ejecute de forma local donde se pueda auditar. El proxy puede configurarse para bloquear el tráfico no autorizado que fluya hacia fuera y filtrar el tráfico de entrada potencialmente malintencionado.

![[3]][3]

Para maximizar la seguridad, no habilite una dirección IP pública para las máquinas virtuales y utilice los grupos de seguridad de red para asegurarse de que estas máquinas virtuales no están accesibles públicamente. Las máquinas virtuales solo deben estar disponibles mediante la dirección IP interna. Estas direcciones pueden hacerse accesibles a través de la red de ExpressRoute, permitiendo al personal de DevOps local para que realice la configuración o el mantenimiento.

Si debe exponer los extremos de administración para las máquinas virtuales a una red externa, utilice grupos de seguridad de red o listas de control de acceso a fin de restringir la visibilidad de estos puertos a una lista de direcciones IP o redes permitidas.

> [!NOTE]
> De forma predeterminada, las máquinas virtuales de Azure implementadas a través de Azure Portal incluyen una dirección IP pública que proporciona acceso de inicio de sesión.  
> 
> 


## <a name="deploy-the-solution"></a>Implementación de la solución

**Requisitos previos**. Debe tener una infraestructura local existente ya configurada con un dispositivo de red adecuado.

Para implementar la solución, siga estos pasos:

1. Haga clic en el botón a continuación:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute%2Fazuredeploy.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. Espere a que el vínculo abra Azure Portal y, a continuación, siga estos pasos:
   * El nombre del **Grupo de recursos** ya está definido en el archivo de parámetros, así que seleccione **Crear nuevo** y escriba `ra-hybrid-er-rg` en el cuadro de texto.
   * Seleccione la región en el cuadro de lista desplegable **Ubicación**.
   * No modifique los cuadros de texto **URI raíz de plantilla** o **URI raíz de parámetro**.
   * Revise los términos y condiciones, y haga clic en la casilla **Acepto los términos y condiciones indicados anteriormente**.
   * Haga clic en el botón **Comprar**.
3. Espere a que la implementación se complete.
4. Haga clic en el botón a continuación:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute%2Fazuredeploy-expressRouteCircuit.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
5. Espere a que el vínculo abra Azure Portal y, a continuación, siga estos pasos:
   * Seleccione **Usar existente** en la sección **Grupo de recursos** y escriba `ra-hybrid-er-rg` en el cuadro de texto.
   * Seleccione la región en el cuadro de lista desplegable **Ubicación**.
   * No modifique los cuadros de texto **URI raíz de plantilla** o **URI raíz de parámetro**.
   * Revise los términos y condiciones, y haga clic en la casilla **Acepto los términos y condiciones indicados anteriormente**.
   * Haga clic en el botón **Comprar**.
6. Espere a que la implementación se complete.


<!-- links -->
[forced-tuneling]: ../dmz/secure-vnet-hybrid.md
[highly-available-network-architecture]: ./expressroute-vpn-failover.md

[expressroute-technical-overview]: /azure/expressroute/expressroute-introduction
[expressroute-prereqs]: /azure/expressroute/expressroute-prerequisites
[configure-expressroute-routing]: /azure/expressroute/expressroute-howto-routing-arm
[sla-for-expressroute]: https://azure.microsoft.com/support/legal/sla/expressroute
[link-vnet-to-expressroute]: /azure/expressroute/expressroute-howto-linkvnet-arm
[ExpressRoute-provisioning]: /azure/expressroute/expressroute-workflows
[expressroute-introduction]: /azure/expressroute/expressroute-introduction
[expressroute-peering]: /azure/expressroute/expressroute-circuit-peerings
[expressroute-pricing]: https://azure.microsoft.com/pricing/details/expressroute/
[expressroute-limits]: /azure/azure-subscription-service-limits#networking-limits
[azurect]: https://github.com/Azure/NetworkMonitoring/tree/master/AzureCT
[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-architectures.vsdx
[er-circuit-parameters]: https://github.com/mspnp/reference-architectures/tree/master/hybrid-networking/expressroute/parameters/expressRouteCircuit.parameters.json
[azure-powershell-download]: https://azure.microsoft.com/documentation/articles/powershell-install-configure/
[azure-cli]: https://azure.microsoft.com/documentation/articles/xplat-cli-install/
[0]: ./images/expressroute.png "Arquitectura de red híbrida con Azure ExpressRoute"
[1]: ../_images/guidance-hybrid-network-expressroute/figure2.png "Uso de enrutadores redundantes con circuitos ExpressRoute principales y secundarios"
[2]: ../_images/guidance-hybrid-network-expressroute/figure3.png "Incorporación de dispositivos de seguridad a la red local"
[3]: ../_images/guidance-hybrid-network-expressroute/figure4.png "Tunelización forzada para auditar el tráfico de Internet"
[4]: ../_images/guidance-hybrid-network-expressroute/figure5.png "Ubicación del valor de ServiceKey de un circuito ExpressRoute"  