---
title: Conexión de una red local a Azure
titleSuffix: Azure Reference Architectures
description: Compare las arquitecturas de referencia para conectar una red local a Azure.
author: telmosampaio
ms.date: 07/02/2018
ms.openlocfilehash: f13249f225ad7ab5072de2b2175cdc2ffb6d0074
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/04/2019
ms.locfileid: "54011062"
---
# <a name="choose-a-solution-for-connecting-an-on-premises-network-to-azure"></a>Elección de una solución para conectar una red local a Azure

En este artículo se comparan las opciones para conectar una red local a una instancia de Azure Virtual Network (VNet). Para cada opción, hay disponible una arquitectura de referencia más detallada.

## <a name="vpn-connection"></a>Conexión VPN

Una [puerta de enlace de VPN](/azure/vpn-gateway/vpn-gateway-about-vpngateways) es un tipo de puerta de enlace de red virtual que envía tráfico cifrado entre una instancia de Azure Virtual Network y una ubicación local. El tráfico cifrado pasa a través de Internet público.

Esta arquitectura es adecuada para las aplicaciones híbridas en las que es probable que el tráfico entre el hardware local y la nube sea ligero, o en la que esté dispuesto a compensar una latencia ligeramente mayor con la flexibilidad y el poder de procesamiento de la nube.

### <a name="benefits"></a>Ventajas

- Fácil de configurar.

### <a name="challenges"></a>Desafíos

- Requiere un dispositivo VPN local.
- Aunque Microsoft garantiza la disponibilidad del 99,9 % para cada VPN Gateway, este [Acuerdo de Nivel de Servicio](https://azure.microsoft.com/support/legal/sla/vpn-gateway/) solo cubre VPN Gateway y no la conexión de red a la puerta de enlace.
- Una conexión VPN a través de Azure VPN Gateway admite actualmente un ancho de banda máximo de 1,25 Gbps. Quizá tenga que dividir su red Azure Virtual Network entre varias conexiones VPN si espera que se supere este rendimiento.

### <a name="reference-architecture"></a>Arquitectura de referencia

- [Red híbrida con VPN Gateway](./vpn.md)

<!-- markdownlint-disable MD024 -->

## <a name="azure-expressroute-connection"></a>Conexión de Azure ExpressRoute

Las conexiones de [ExpressRoute](/azure/expressroute/) utilizan una conexión privada y dedicada a través de un proveedor de conectividad de terceros. La conexión privada extiende la red local en Azure.

Esta arquitectura es adecuada para aplicaciones híbridas que ejecutan cargas de trabajo críticas a gran escala que requieren un alto grado de escalabilidad.

### <a name="benefits"></a>Ventajas

- Ancho de banda mucho más alto disponible; hasta 10 Gbps en función del proveedor de conectividad.
- Admite el escalado dinámico de ancho de banda para ayudar a reducir los costos durante los períodos de menor actividad. Sin embargo, no todos los proveedores de conectividad tienen esta opción.
- Puede permitir que su organización tenga acceso directo a las nubes nacionales, en función del proveedor de conectividad.
- Acuerdo de Nivel de Servicio del 99,9 % de disponibilidad en toda la conexión.

### <a name="challenges"></a>Desafíos

- Puede ser difícil de configurar. La creación de una conexión ExpressRoute requiere trabajar con un proveedor de conectividad de terceros. El proveedor es responsable del aprovisionamiento de la conexión de red.
- Requiere que enrutadores de un alto ancho de banda a nivel local.

### <a name="reference-architecture"></a>Arquitectura de referencia

- [Red híbrida con ExpressRoute](./expressroute.md)

## <a name="expressroute-with-vpn-failover"></a>ExpressRoute con conmutación por error de VPN

Esta opción combina los dos anteriores: uso de ExpressRoute en condiciones normales, pero conmutación por error a una conexión VPN si se produce una pérdida de conectividad en el circuito de ExpressRoute.

Esta arquitectura es adecuada para aplicaciones híbridas que necesitan un mayor ancho de banda que ExpressRoute y también requieren conectividad de red de alta disponibilidad.

### <a name="benefits"></a>Ventajas

- Alta disponibilidad si se produce un error en el circuito de ExpressRoute, aunque la conexión de reserva está en una red de ancho de banda inferior.

### <a name="challenges"></a>Desafíos

- Configuración compleja. Debe configurar tanto una conexión VPN como un circuito de ExpressRoute.
- Requiere hardware redundante (dispositivos VPN) y una conexión redundante de Azure VPN Gateway que tiene un coste.

### <a name="reference-architecture"></a>Arquitectura de referencia

- [Red híbrida con ExpressRoute y conmutación por error de VPN](./expressroute-vpn-failover.md)

<!-- markdownlint-disable MD024 -->

## <a name="hub-spoke-network-topology"></a>Topología de red en estrella tipo hub-and-spoke

Una topología de red en estrella de tipo hub-and-spoke constituye una forma de aislar cargas de trabajo mientras se comparten servicios como los de identidad y seguridad. El centro (hub) es una red virtual (VNet) en Azure que actúa como un punto central de conectividad para la red local. Los radios son redes virtuales que se emparejan con el concentrador. Los servicios compartidos se implementan en el centro, mientras que las cargas de trabajo individuales se implementan como radios.

### <a name="reference-architectures"></a>Arquitecturas de referencia

- [Topología en estrella tipo hub-and-spoke](./hub-spoke.md)
- [Topología en estrella tipo hub-and-spoke con servicios compartidos](./shared-services.md)
