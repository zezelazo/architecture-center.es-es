---
title: Implementación de aplicaciones virtuales de red de alta disponibilidad
titleSuffix: Azure Reference Architectures
description: Implementación de aplicaciones virtuales de red con alta disponibilidad.
author: telmosampaio
ms.date: 12/08/2018
ms.custom: seodec18
ms.openlocfilehash: 646721f80d19f493b7674884f8108762d743201b
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/04/2019
ms.locfileid: "54011096"
---
# <a name="deploy-highly-available-network-virtual-appliances"></a>Implementación de aplicaciones virtuales de red de alta disponibilidad

En este artículo se muestra cómo implementar un conjunto de aplicaciones de red virtual (NVA) de alta disponibilidad en Azure. Una NVA se usa normalmente para controlar el flujo del tráfico de red desde una red perimetral, conocida también como DMZ, hasta otras redes o subredes. Para aprender sobre la implementación de una red perimetral en Azure, consulte [Servicios en la nube de Microsoft y seguridad de red][cloud-security]. El artículo incluye arquitecturas de ejemplo de solo entrada, solo salida y de entrada y salida.

**Requisitos previos:** En este artículo se presupone un conocimiento básico de redes de Azure, [equilibradores de carga de Azure][lb-overview] y [rutas definidas por el usuario][udr-overview] (UDR).

## <a name="architecture-diagrams"></a>Diagramas de arquitectura

Una aplicación de red virtual se puede implementar en una red perimetral en muchas arquitecturas diferentes. Por ejemplo, en la siguiente ilustración se muestra el uso de una [única NVA][ nva-scenario] para la entrada.

![[0]][0]

En esta arquitectura, la aplicación de red virtual proporciona un límite de red segura, ya que comprueba todo el tráfico de red de entrada y salida y solo deja pasar aquel que cumple las reglas de seguridad de la red. Sin embargo, el hecho de que todo el tráfico de la red deba pasar por la aplicación de red virtual significa que dicha aplicación es un único punto de error en la red. Así que, si se produce un error en la aplicación de red virtual, no habrá ninguna otra ruta para el tráfico de red ni ninguna subred back-end disponible.

Para conseguir que una aplicación virtual de red tenga una alta disponibilidad, implemente varias de estas aplicaciones en un conjunto de disponibilidad.

Las arquitecturas siguientes describen los recursos y la configuración que son necesarios para dotar a las aplicaciones virtuales de red de una alta disponibilidad:

<!-- markdownlint-disable MD033 -->

| Solución | Ventajas | Consideraciones |
| --- | --- | --- |
| [Entrada con NVA de capa 7][ingress-with-layer-7] |Todos los nodos NVA están activos. |Requiere una aplicación virtual de red que pueda terminar las conexiones y usar SNAT.<br/> Requiere un conjunto independiente de NVA para el tráfico procedente de Internet y de Azure. <br/> Solo puede usarse con el tráfico que se origina fuera de Azure. |
| [Salida con NVA de capa 7][egress-with-layer-7] |Todos los nodos NVA están activos. | Requiere una aplicación virtual de red que pueda terminar las conexiones e implemente la traducción de direcciones de red de origen (SNAT).
| [Entrada y salida con NVA de capa 7][ingress-egress-with-layer-7] |Todos los nodos están activos.<br/>Puede controlar el tráfico que se origina en Azure. |Requiere una aplicación virtual de red que pueda terminar las conexiones y usar SNAT.<br/>Requiere un conjunto independiente de NVA para el tráfico procedente de Internet y de Azure. |
| [Conmutador PIP-UDR][pip-udr-switch] |Un solo conjunto de NVA para todo el tráfico.<br/>Puede controlar todo el tráfico (sin límite en las reglas de puerto). |Activo-pasivo<br/>Requiere un proceso de conmutación por error. |
| [PIP-UDR sin SNAT](#pip-udr-nvas-without-snat) | Un solo conjunto de NVA para todo el tráfico.<br/>Puede controlar todo el tráfico (sin límite en las reglas de puerto).<br/>No requiere la configuración de SNAT para las solicitudes entrantes |Activo-pasivo<br/>Requiere un proceso de conmutación por error.<br/>La lógica de sondeo y de conmutación por error se ejecutan fuera de la red virtual |

<!-- markdown-enable MD033 -->

## <a name="ingress-with-layer-7-nvas"></a>Entrada con NVA de capa 7

En la siguiente ilustración se muestra una arquitectura de alta disponibilidad que implementa una red perimetral de entrada detrás de un equilibrador de carga accesible desde Internet. Esta arquitectura está diseñada para proporcionar conectividad a las cargas de trabajo de Azure con tráfico de capa de 7, como HTTP o HTTPS:

![[1]][1]

La ventaja de esta arquitectura es que todas las aplicaciones virtuales de red están activas y, si una no funciona correctamente, el equilibrador de carga dirige el tráfico de red a la otra. Ambas aplicaciones virtuales de red enrutan el tráfico al equilibrador de carga interno, de modo que, mientras una aplicación virtual de red esté activa, el tráfico seguirá fluyendo. Las aplicaciones virtuales de red son necesarias para terminar el tráfico de SSL destinado a las máquinas virtuales de nivel web. Estas aplicaciones virtuales de red no se pueden ampliar para administrar el tráfico local puesto que este necesita otro conjunto dedicado de aplicaciones virtuales de red con sus propias rutas de red.

> [!NOTE]
> Esta arquitectura se usa en las arquitecturas de referencia de [red perimetral entre Azure y el centro de datos local][dmz-on-prem] y [red perimetral entre Internet y Azure][ dmz-internet]. Cada una de estas arquitecturas de referencia incluye una solución de implementación que puede usar. Para más información, siga los vínculos.

## <a name="egress-with-layer-7-nvas"></a>Salida con NVA de capa 7

La arquitectura anterior puede ampliarse para proporcionar una red perimetral de salida para las solicitudes que se originan en la carga de trabajo de Azure. La siguiente arquitectura está diseñada para proporcionar alta disponibilidad de las aplicaciones virtuales de red en la red perimetral para el tráfico de capa 7, como HTTP o HTTPS:

![[2]][2]

En esta arquitectura, todo el tráfico que se origina en Azure se enruta a un equilibrador de carga interno. El equilibrador de carga distribuye las solicitudes salientes entre un conjunto de aplicaciones virtuales de red. Estas aplicaciones virtuales de red dirigen el tráfico a Internet mediante sus direcciones IP públicas individuales.

> [!NOTE]
> Esta arquitectura se usa en las arquitecturas de referencia de [red perimetral entre Azure y el centro de datos local][dmz-on-prem] y [red perimetral entre Internet y Azure][ dmz-internet]. Cada una de estas arquitecturas de referencia incluye una solución de implementación que puede usar. Para más información, siga los vínculos.

## <a name="ingress-egress-with-layer-7-nvas"></a>Entrada y salida con NVA de capa 7

En las dos arquitecturas anteriores, había una red perimetral distinta para la entrada y la salida. La siguiente arquitectura muestra cómo crear una red perimetral que puede usarse para la entrada y la salida con el tráfico de capa 7, como HTTP o HTTPS:

![[4]][4]

En esta arquitectura, las aplicaciones virtuales de red procesan las solicitudes que entran desde la puerta de enlace de aplicaciones. Las aplicaciones virtuales de red también procesan las solicitudes que salen de las máquinas virtuales de carga de trabajo en el grupo back-end del equilibrador de carga. Como el tráfico entrante se enruta con una puerta de enlace de aplicaciones y el tráfico saliente lo hace con un equilibrador de carga, las aplicaciones virtuales de red son responsables de mantener la afinidad de la sesión. Es decir, la puerta de enlace de aplicaciones mantiene una asignación de solicitudes entrantes y salientes, de forma que pueda reenviar la respuesta correcta al solicitante original. Sin embargo, el equilibrador de carga interno no tiene acceso a las asignaciones de la puerta de enlace de aplicaciones, así que emplea su propia lógica para enviar las respuestas a las aplicaciones virtuales de red. Es posible que el equilibrador de carga envíe una respuesta a una aplicación virtual de red que inicialmente no recibió la solicitud de la puerta de enlace de aplicaciones. En este caso, las aplicaciones virtuales de red deben comunicarse y transferirse la respuesta para que la aplicación virtual de red correcta pueda reenviar la respuesta a la puerta de enlace de aplicaciones.

> [!NOTE]
> Otra manera de resolver el problema del enrutamiento asimétrico es asegurarse de que las aplicaciones virtuales de red realicen la traducción de las direcciones de red de origen (SNAT). De esta forma, se reemplazaría la dirección IP de origen original del solicitante por una de las direcciones IP de la aplicación virtual de red usada en el flujo de entrada. Con ello se garantiza que pueda usar varias aplicaciones virtuales de red al mismo tiempo sin perder la simetría de la ruta.

## <a name="pip-udr-switch-with-layer-4-nvas"></a>Conmutador PIP-UDR con NVA de capa 4

La siguiente arquitectura ilustra una arquitectura con una aplicación virtual de red activa y otra pasiva. Esta arquitectura administra tanto la entrada como la salida del tráfico de capa 4:

![[3]][3]

> [!TIP]
> Hay disponible una solución completa de esta arquitectura en [GitHub][pnp-ha-nva].

Esta arquitectura es parecida a la primera arquitectura descrita en este artículo, que incluía una única aplicación virtual de red que acepta y filtra las solicitudes entrantes de capa 4. Pero a diferencia de ella, agrega una segunda aplicación virtual de red pasiva para proporcionar alta disponibilidad. Si se produce un error en la aplicación virtual de red activa, la aplicación virtual de red pasiva se vuelve activa y el UDR y la PIP cambian para apuntar a las tarjetas NIC de la aplicación virtual de red ahora activa. Estos cambios en el UDR y la PIP pueden realizarse manualmente o mediante un proceso automatizado. El proceso automatizado suele ser un demonio u otro servicio de supervisión que se ejecuta en Azure. Lo que hace es consultar un sondeo de estado en la aplicación virtual de red activa y realizar la conmutación del UDR y la PIP cuando detecta un error de la aplicación virtual de red.

En la ilustración anterior se muestra un clúster [ZooKeeper][zookeeper] de ejemplo que proporciona un demonio de alta disponibilidad. En el clúster ZooKeeper, un cuórum de nodos elige un nodo principal. Si se produce un error en el nodo principal, el resto de los nodos celebran elecciones para elegir uno nuevo principal. En esta arquitectura, el nodo principal ejecuta el demonio que consulta el punto de conexión de mantenimiento en la aplicación virtual de red. Si la aplicación virtual de red no logra responder al sondeo de estado, el demonio activa la aplicación virtual de red pasiva. A continuación, el demonio llama a la API de REST de Azure para quitar la PIP de la aplicación virtual de red con errores y asociarla a la aplicación virtual de red recién creada. Seguidamente, el demonio modifica el UDR para que apunte a la dirección IP interna de la aplicación virtual de red recién activada.

No incluya los nodos ZooKeeper en una subred a la que solo se pueda acceder mediante una ruta que incluya la aplicación virtual de red. Si lo hace, los nodos ZooKeeper serán inaccesibles en caso de que se produzca un error en la aplicación virtual de red. Si por alguna razón se produce un error en el demonio, no podrá acceder a ninguno de los nodos ZooKeeper para diagnosticar el problema.

Para ver la solución completa, incluido el código de ejemplo, consulte los archivos en el [repositorio de GitHub][pnp-ha-nva].

## <a name="pip-udr-nvas-without-snat"></a>Aplicaciones virtuales de red PIP-UDR sin SNAT

Esta arquitectura emplea dos máquinas virtuales de Azure para hospedar el firewall de aplicación virtual de red en una configuración activo/pasivo que admite la conmutación por error automática, pero no requiere la traducción de direcciones de red de origen (SNAT).

![Arquitectura de aplicaciones virtuales de red PIP-UDR sin SNAT](./images/nva-ha/pip-udr-without-snat.png)

> [!TIP]
> Hay disponible una solución completa de esta arquitectura en [GitHub][ha-nva-fo].

Esta solución está diseñada para clientes de Azure que no pueden configurar SNAT para las solicitudes entrantes en los firewalls de aplicación virtual de red. SNAT oculta la dirección IP de cliente de origen original. Si necesita registrar las direcciones IP originales o usarlas en otros componentes de seguridad por capas detrás de las aplicaciones virtuales de red, esta solución ofrece un enfoque básico.

La conmutación por error de las entradas de la tabla UDR se automatiza mediante una dirección de próximo salto establecida en la dirección IP de una interfaz en la máquina de virtual del firewall de la aplicación virtual de red activa. La lógica de conmutación por error automatizada se hospeda en una aplicación de función que se crea con [Azure Functions](/azure/azure-functions/). El código de la conmutación por error se ejecuta como una función sin servidor en Azure Functions. La implementación es cómoda, rentable y fácil de mantener y personalizar. Además, la aplicación de función se hospeda en Azure Functions, por lo que no tiene ninguna dependencia en la red virtual. Si los cambios realizados en la red virtual afectan a los firewalls de aplicación virtual de red, la aplicación de función continúa su ejecución de forma independiente. Así, las pruebas son más precisas porque tienen lugar fuera de la red virtual con la misma ruta que las solicitudes de cliente entrantes.

Para comprobar la disponibilidad del firewall de la aplicación virtual de red, el código de la aplicación de función realiza sondeos de dos maneras:

- Mediante la supervisión del estado de las máquinas virtuales de Azure que hospedan el firewall de la aplicación virtual de red.

- Mediante pruebas de la existencia de un puerto abierto en el firewall hacia el servidor web de back-end. Para esta opción, el dispositivo virtual de red debe exponer un socket mediante PIP para que el código de la aplicación de función pueda hacer pruebas.

Elija el tipo de sondeo que desea usar al configurar la aplicación de función. Para ver la solución completa, incluido el código de ejemplo, consulte los archivos en el [repositorio de GitHub][ha-nva-fo].

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [implementar una red perimetral entre Azure y el centro de datos local][dmz-on-prem] mediante aplicaciones virtuales de red de capa 7.
- Aprenda a [implementar una red perimetral entre Azure e Internet] [dmz-internet] mediante aplicaciones virtuales de red de capa 7.
- [Solución de problemas de aplicaciones virtuales de red en Azure](/azure/virtual-network/virtual-network-troubleshoot-nva)

<!-- links -->

[cloud-security]: /azure/best-practices-network-security
[dmz-on-prem]: ./secure-vnet-hybrid.md
[dmz-internet]: ./secure-vnet-dmz.md
[egress-with-layer-7]: #egress-with-layer-7-nvas
[ingress-with-layer-7]: #ingress-with-layer-7-nvas
[ingress-egress-with-layer-7]: #ingress-egress-with-layer-7-nvas
[lb-overview]: /azure/load-balancer/load-balancer-overview/
[nva-scenario]: /azure/virtual-network/virtual-network-scenario-udr-gw-nva/
[pip-udr-switch]: #pip-udr-switch-with-layer-4-nvas
[udr-overview]: /azure/virtual-network/virtual-networks-udr-overview/
[zookeeper]: https://zookeeper.apache.org/
[pnp-ha-nva]: https://github.com/mspnp/ha-nva
[ha-nva-fo]: https://aka.ms/ha-nva-fo

<!-- images -->

[0]: ./images/nva-ha/single-nva.png "Arquitectura de una única aplicación virtual de red"
[1]: ./images/nva-ha/l7-ingress.png "Entrada de capa 7"
[2]: ./images/nva-ha/l7-ingress-egress.png "Salida de capa 7"
[3]: ./images/nva-ha/active-passive.png "Clústeres activo/pasivo"
[4]: ./images/nva-ha/l7-ingress-egress-ag.png
