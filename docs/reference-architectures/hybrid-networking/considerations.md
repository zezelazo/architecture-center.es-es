---
title: Elección de una solución para conectar una red local a Azure
description: Compara las arquitecturas de referencia para conectar una red local a Azure.
author: telmosampaio
ms.date: 04/06/2017
ms.openlocfilehash: 274b9df1817632a7f3eaafa8bf02e965fdc3feea
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2017
---
# <a name="choose-a-solution-for-connecting-an-on-premises-network-to-azure"></a>Elección de una solución para conectar una red local a Azure

En este artículo se comparan las opciones para conectar una red local a una instancia de Azure Virtual Network (VNet). Se proporciona una arquitectura de referencia y una solución implementable para cada opción.

## <a name="vpn-connection"></a>Conexión VPN

Use una red privada virtual (VPN) para conectar su red local con una Azure VNet a través de un túnel de VPN con IPSec.

Esta arquitectura es adecuada para las aplicaciones híbridas en las que es probable que el tráfico entre el hardware local y la nube sea ligero, o en la que esté dispuesto a compensar una latencia ligeramente mayor con la flexibilidad y el poder de procesamiento de la nube.

**Ventajas**

- Fácil de configurar.

**Desafíos**

- Requiere un dispositivo VPN local.
- Aunque Microsoft garantiza la disponibilidad del 99,9 % para cada VPN Gateway, este Acuerdo de Nivel de Servicio solo cubre VPN Gateway y no la conexión de red para la puerta de enlace.
- Una conexión VPN a través de Azure VPN Gateway admite actualmente un ancho de banda máximo de 200 Mbps. Quizá tenga que dividir su red Azure Virtual Network entre varias conexiones VPN si espera que se supere este rendimiento.

**[Más información...][vpn]**

## <a name="azure-expressroute-connection"></a>Conexión de Azure ExpressRoute

Las conexiones de ExpressRoute utilizan una conexión privada y dedicada a través de un proveedor de conectividad de terceros. La conexión privada extiende la red local en Azure. 

Esta arquitectura es adecuada para aplicaciones híbridas que ejecutan cargas de trabajo críticas a gran escala que requieren un alto grado de escalabilidad. 

**Ventajas**

- Ancho de banda mucho más alto disponible; hasta 10 Gbps en función del proveedor de conectividad.
- Admite el escalado dinámico de ancho de banda para ayudar a reducir los costos durante los períodos de menor actividad. Sin embargo, no todos los proveedores de conectividad tienen esta opción.
- Puede permitir que su organización tenga acceso directo a las nubes nacionales, en función del proveedor de conectividad.
- Acuerdo de Nivel de Servicio del 99,9 % de disponibilidad en toda la conexión.

**Desafíos**

- Puede ser difícil de configurar. La creación de una conexión ExpressRoute requiere trabajar con un proveedor de conectividad de terceros. El proveedor es responsable del aprovisionamiento de la conexión de red.
- Requiere que enrutadores de un alto ancho de banda a nivel local.

**[Más información...][expressroute]**

## <a name="expressroute-with-vpn-failover"></a>ExpressRoute con conmutación por error de VPN

Esta opción combina los dos anteriores: uso de ExpressRoute en condiciones normales, pero conmutación por error a una conexión VPN si se produce una pérdida de conectividad en el circuito de ExpressRoute.

Esta arquitectura es adecuada para aplicaciones híbridas que necesitan un mayor ancho de banda que ExpressRoute y también requieren conectividad de red de alta disponibilidad. 

**Ventajas**

- Alta disponibilidad si se produce un error en el circuito de ExpressRoute, aunque la conexión de reserva está en una red de ancho de banda inferior.

**Desafíos**

- Configuración compleja. Debe configurar tanto una conexión VPN como un circuito de ExpressRoute.
- Requiere hardware redundante (dispositivos VPN) y una conexión redundante de Azure VPN Gateway que tiene un coste.

**[Más información...][expressroute-vpn-failover]**

<!-- links -->
[expressroute]: ./expressroute.md
[expressroute-vpn-failover]: ./expressroute-vpn-failover.md
[vpn]: ./vpn.md