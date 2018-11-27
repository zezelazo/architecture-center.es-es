---
title: Elección de una solución para conectar una red local a Azure
description: Compara las arquitecturas de referencia para conectar una red local a Azure.
author: telmosampaio
ms.date: 07/02/2018
ms.openlocfilehash: a9e2a212d65530e714635bbfae3a57766e77c3a6
ms.sourcegitcommit: 19a517a2fb70768b3edb9a7c3c37197baa61d9b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/26/2018
ms.locfileid: "52295499"
---
# <a name="connect-an-on-premises-network-to-azure"></a>Conexión de una red local a Azure

En este artículo se comparan las opciones para conectar una red local a una instancia de Azure Virtual Network (VNet). Para cada opción, hay disponible una arquitectura de referencia más detallada.

## <a name="vpn-connection"></a>Conexión VPN

Una [puerta de enlace de VPN](/azure/vpn-gateway/vpn-gateway-about-vpngateways) es un tipo de puerta de enlace de red virtual que envía tráfico cifrado entre una instancia de Azure Virtual Network y una ubicación local. El tráfico cifrado pasa a través de Internet público.

Esta arquitectura es adecuada para las aplicaciones híbridas en las que es probable que el tráfico entre el hardware local y la nube sea ligero, o en la que esté dispuesto a compensar una latencia ligeramente mayor con la flexibilidad y el poder de procesamiento de la nube.

**Ventajas**

- Fácil de configurar.

**Desafíos**

- Requiere un dispositivo VPN local.
- Aunque Microsoft garantiza la disponibilidad del 99,9 % para cada VPN Gateway, este [Acuerdo de Nivel de Servicio](https://azure.microsoft.com/support/legal/sla/vpn-gateway/) solo cubre VPN Gateway y no la conexión de red a la puerta de enlace.
- Una conexión VPN a través de Azure VPN Gateway admite actualmente un ancho de banda máximo de 1,25 Gbps. Quizá tenga que dividir su red Azure Virtual Network entre varias conexiones VPN si espera que se supere este rendimiento.

**Arquitectura de referencia**

- [Red híbrida con VPN Gateway](./vpn.md)

## <a name="azure-expressroute-connection"></a>Conexión de Azure ExpressRoute

Las conexiones de [ExpressRoute](/azure/expressroute/) utilizan una conexión privada y dedicada a través de un proveedor de conectividad de terceros. La conexión privada extiende la red local en Azure. 

Esta arquitectura es adecuada para aplicaciones híbridas que ejecutan cargas de trabajo críticas a gran escala que requieren un alto grado de escalabilidad. 

**Ventajas**

- Ancho de banda mucho más alto disponible; hasta 10 Gbps en función del proveedor de conectividad.
- Admite el escalado dinámico de ancho de banda para ayudar a reducir los costos durante los períodos de menor actividad. Sin embargo, no todos los proveedores de conectividad tienen esta opción.
- Puede permitir que su organización tenga acceso directo a las nubes nacionales, en función del proveedor de conectividad.
- Acuerdo de Nivel de Servicio del 99,9 % de disponibilidad en toda la conexión.

**Desafíos**

- Puede ser difícil de configurar. La creación de una conexión ExpressRoute requiere trabajar con un proveedor de conectividad de terceros. El proveedor es responsable del aprovisionamiento de la conexión de red.
- Requiere que enrutadores de un alto ancho de banda a nivel local.

**Arquitectura de referencia**

- [Red híbrida con ExpressRoute](./expressroute.md)

## <a name="expressroute-with-vpn-failover"></a>ExpressRoute con conmutación por error de VPN

Esta opción combina los dos anteriores: uso de ExpressRoute en condiciones normales, pero conmutación por error a una conexión VPN si se produce una pérdida de conectividad en el circuito de ExpressRoute.

Esta arquitectura es adecuada para aplicaciones híbridas que necesitan un mayor ancho de banda que ExpressRoute y también requieren conectividad de red de alta disponibilidad. 

**Ventajas**

- Alta disponibilidad si se produce un error en el circuito de ExpressRoute, aunque la conexión de reserva está en una red de ancho de banda inferior.

**Desafíos**

- Configuración compleja. Debe configurar tanto una conexión VPN como un circuito de ExpressRoute.
- Requiere hardware redundante (dispositivos VPN) y una conexión redundante de Azure VPN Gateway que tiene un coste.

**Arquitectura de referencia**

- [Red híbrida con ExpressRoute y conmutación por error de VPN](./expressroute-vpn-failover.md)


## <a name="hub-spoke-network-topology"></a>Topología de red en estrella tipo hub-and-spoke

Una topología de red en estrella de tipo hub-and-spoke constituye una forma de aislar cargas de trabajo mientras se comparten servicios como los de identidad y seguridad. El centro (hub) es una red virtual (VNet) en Azure que actúa como un punto central de conectividad para la red local. Los radios son redes virtuales que se emparejan con el concentrador. Los servicios compartidos se implementan en el centro, mientras que las cargas de trabajo individuales se implementan como radios.


**Arquitecturas de referencia**

- [Topología en estrella tipo hub-and-spoke](./hub-spoke.md)
- [Topología en estrella tipo hub-and-spoke con servicios compartidos](./shared-services.md)
