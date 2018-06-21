---
title: Implementación de una red perimetral entre Azure e Internet
description: Cómo implementar una arquitectura de red híbrida segura con acceso a Internet en Azure.
author: telmosampaio
ms.date: 11/23/2016
pnp.series.title: Network DMZ
pnp.series.next: nva-ha
pnp.series.prev: secure-vnet-hybrid
cardTitle: DMZ between Azure and the Internet
ms.openlocfilehash: c88545b1fcae49b413e7e2b6ac5bd92d3fd3456d
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2018
ms.locfileid: "30270411"
---
# <a name="dmz-between-azure-and-the-internet"></a>Red perimetral entre Internet y Azure

Esta arquitectura de referencia muestra una red híbrida segura que extiende una red local a Azure y también acepta el tráfico de Internet. 

[![0]][0] 

*Descargue un [archivo Visio][visio-download] de esta arquitectura.*

Esta arquitectura de referencia extiende la arquitectura descrita en [Implementación de una red perimetral entre Azure y el centro de datos local][implementing-a-secure-hybrid-network-architecture]. Agrega una red perimetral pública que gestiona el tráfico de Internet, además de la red perimetral privada que gestiona el tráfico de la red local. 

Los usos habituales de esta arquitectura incluyen:

* Aplicaciones híbridas donde parte de las cargas de trabajo se ejecutan de forma local y parte en Azure.
* Infraestructura de Azure que enruta el tráfico entrante procedente del entorno local e Internet.

## <a name="architecture"></a>Architecture

La arquitectura consta de los siguientes componentes:

* **Dirección IP pública (PIP)** La dirección IP del punto de conexión público. Los usuarios externos conectados a Internet pueden acceder al sistema mediante esta dirección.
* **Aplicación virtual de red (NVA)**. Esta arquitectura incluye un grupo independiente de NVA para el tráfico que se origina en Internet.
* **Azure Load Balancer**. Todas las solicitudes entrantes de Internet pasan por el equilibrador de carga y se distribuyen a las aplicaciones virtuales de red en la red perimetral pública.
* **Subred de entrada de red perimetral pública**. Esta subred acepta las solicitudes del equilibrador de carga de Azure. Las solicitudes entrantes se pasan a una de las aplicaciones virtuales de red de la red perimetral pública.
* **Subred de salida de red perimetral pública**. Las solicitudes que apueba la aplicación virtual de red pasan por esta subred y se dirigen al equilibrador de carga interno en el nivel web.

## <a name="recommendations"></a>Recomendaciones

Las siguientes recomendaciones sirven para la mayoría de los escenarios. Sígalas a menos que tenga un requisito concreto que las invalide. 

### <a name="nva-recommendations"></a>Recomendaciones para aplicaciones virtuales de red

Se recomienda usar un conjunto de NVA para el tráfico que se origina en Internet y otro para el tráfico que se origina en el entorno local. Utilizar únicamente un conjunto de NVA para ambos es un riesgo de seguridad, ya que no proporciona ningún perímetro de seguridad entre los dos conjuntos de tráfico de red. El uso de NVA independientes reduce la complejidad de la comprobación de las reglas de seguridad y deja claro qué reglas se corresponden con cada solicitud de red entrante. Un conjunto de NVA implementa reglas solo para el tráfico de Internet, mientras que otro lo hace solo para el tráfico local.

Incluya una aplicación virtual de red de capa 7 para terminar las conexiones de aplicación en el nivel de NVA y mantener la compatibilidad con los niveles de back-end. Así se garantiza una conectividad simétrica donde el tráfico de respuesta de los niveles de back-end se devuelve a través de la aplicación virtual de red.  

### <a name="public-load-balancer-recommendations"></a>Recomendaciones para el equilibrador de carga público

Para conseguir escalabilidad y disponibilidad, implemente las aplicaciones virtuales de red de la red perimetral en un [conjunto de disponibilidad][availability-set] y use un [equilibrador de carga accesible desde Internet][load-balancer] para distribuir las solicitudes de Internet entre las aplicaciones virtuales de red de dicho conjunto.  

Configure el equilibrador de carga para que solo acepte solicitudes en los puertos necesarios con el tráfico de Internet. Por ejemplo, limite las solicitudes HTTP entrantes al puerto 80 y las solicitudes HTTPS entrantes al puerto 443.

## <a name="scalability-considerations"></a>Consideraciones sobre escalabilidad

Aunque la arquitectura requiera inicialmente una única aplicación virtual de red en la red perimetral pública, se recomienda colocar un equilibrador de carga delante de la red perimetral pública desde el principio. De esta manera, será más sencillo escalar a varias aplicaciones virtuales de red si es necesario en el futuro.

## <a name="availability-considerations"></a>Consideraciones sobre disponibilidad

El equilibrador de carga accesible desde Internet necesita cada una de las aplicaciones virtuales de red de la subred de entrada de la red perimetral pública para implementar un [sondeo de estado][lb-probe]. Un sondeo de estado que no responde en este punto de conexión se considera que no disponible y el equilibrador de carga dirigirá las solicitudes a otras aplicaciones virtuales de red del mismo conjunto de disponibilidad. Tenga en cuenta que si ninguna aplicación virtual de red responde, la aplicación dará error, así que es importante tener configurada la supervisión para que DevOps reviva el aviso cuando el número de instancias de NVA en buen estado descienda por debajo de un umbral definido.

## <a name="manageability-considerations"></a>Consideraciones sobre la manejabilidad

Toda la supervisión y administración de las aplicaciones virtuales de red de la red perimetral pública se debe realizar en el Jumpbox en la subred de administración. Como se describe en [Implementación de una red perimetral entre Azure y el centro de datos local][implementing-a-secure-hybrid-network-architecture], defina una ruta de red única entre la red local y el Jumpbox que pase por la puerta de enlace, a fin de restringir acceso.

Aunque la conectividad de puerta de enlace de la red local a Azure esté fuera de servicio, puede comunicarse con el Jumpbox; para ello, implemente una dirección IP pública, agréguela al JumpBox e inicie sesión desde Internet.

## <a name="security-considerations"></a>Consideraciones sobre la seguridad

Esta arquitectura de referencia implementa varios niveles de seguridad:

* El equilibrador de carga accesible desde Internet dirige las solicitudes a las aplicaciones virtuales de red de la subred de la red experimental pública de entrada, y solo en los puertos necesarios de la aplicación.
* Las reglas NSG de las subredes de la red perimetral pública de entrada y salida impiden que las aplicaciones virtuales de red se pongan en peligro, ya que bloquean las solicitudes que se encuentran fuera de estas reglas.
* La configuración de enrutamiento NAT de las aplicaciones virtuales de red dirige las solicitudes entrantes en los puertos 80 y 443 al equilibrador de carga de nivel web, pero omite las solicitudes en todos los demás puertos.

Todas las solicitudes entrantes se deben registrar en todos los puertos. Realice auditorías de los registros con regularidad y preste atención a las solicitudes que se encuentran fuera de los parámetros previstos, ya que podrían ser indicio de intentos de intrusión.

## <a name="solution-deployment"></a>Implementación de la solución

Se puede encontrar una implementación de una arquitectura de referencia que implementa estas recomendaciones en [GitHub][github-folder]. La arquitectura de referencia se puede implementar con máquinas virtuales Windows o Linux siguiendo estas instrucciones:

1. Haga clic en el botón a continuación:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fdmz%2Fsecure-vnet-dmz%2FvirtualNetwork.azuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. Una vez abierto el vínculo en Azure Portal, debe especificar los valores de algunas de las opciones:
   * El nombre del **Grupo de recursos** ya está definido en el archivo de parámetros, así que seleccione **Crear nuevo** y escriba `ra-public-dmz-network-rg` en el cuadro de texto.
   * Seleccione la región en el cuadro de lista desplegable **Ubicación**.
   * No modifique los cuadros de texto **URI raíz de plantilla** o **URI raíz de parámetro**.
   * Seleccione el **tipo de SO** en el cuadro de lista desplegable: **windows** o **linux**.
   * Revise los términos y condiciones, y haga clic en la casilla **Acepto los términos y condiciones indicados anteriormente**.
   * Haga clic en el botón **Comprar**.
3. Espere a que la implementación se complete.
4. Haga clic en el botón a continuación:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fdmz%2Fsecure-vnet-dmz%2Fworkload.azuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
5. Una vez abierto el vínculo en Azure Portal, debe especificar los valores de algunas de las opciones:
   * El nombre del **Grupo de recursos** ya está definido en el archivo de parámetros, así que seleccione **Crear nuevo** y escriba `ra-public-dmz-wl-rg` en el cuadro de texto.
   * Seleccione la región en el cuadro de lista desplegable **Ubicación**.
   * No modifique los cuadros de texto **URI raíz de plantilla** o **URI raíz de parámetro**.
   * Revise los términos y condiciones, y haga clic en la casilla **Acepto los términos y condiciones indicados anteriormente**.
   * Haga clic en el botón **Comprar**.
6. Espere a que la implementación se complete.
7. Haga clic en el botón a continuación:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fdmz%2Fsecure-vnet-dmz%2Fsecurity.azuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
8. Una vez abierto el vínculo en Azure Portal, debe especificar los valores de algunas de las opciones:
   * El nombre del **grupo de recursos** ya está definido en el archivo de parámetros, así que seleccione **Usar existente** y escriba `ra-public-dmz-network-rg` en el cuadro de texto.
   * Seleccione la región en el cuadro de lista desplegable **Ubicación**.
   * No modifique los cuadros de texto **URI raíz de plantilla** o **URI raíz de parámetro**.
   * Revise los términos y condiciones, y haga clic en la casilla **Acepto los términos y condiciones indicados anteriormente**.
   * Haga clic en el botón **Comprar**.
9. Espere a que la implementación se complete.
10. Los archivos de parámetros incluyen un nombre de usuario y una contraseña de administrador codificados de forma rígida para todas las máquinas virtuales, y es muy recomendable cambiarlos inmediatamente. Seleccione cada máquina virtual de la implementación en Azure Portal y haga clic en **Restablecer contraseña** en la hoja **Soporte técnico y solución de problemas**. Seleccione **Restablecer contraseña** en el cuadro de lista desplegable **Modo**, seleccione un valor de **Nombre de usuario** y un valor de **Contraseña**. Haga clic en el botón **Actualizar** para guardar.


[availability-set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/dmz/secure-vnet-dmz

[implementing-a-secure-hybrid-network-architecture]: ./secure-vnet-hybrid.md
[iptables]: https://help.ubuntu.com/community/IptablesHowTo
[lb-probe]: /azure/load-balancer/load-balancer-custom-probe-overview
[load-balancer]: /azure/load-balancer/load-balancer-Internet-overview
[network-security-group]: /azure/virtual-network/virtual-networks-nsg

[visio-download]: https://archcenter.blob.core.windows.net/cdn/dmz-reference-architectures.vsdx


[0]: ./images/dmz-public.png "Arquitectura de red híbrida segura"
