---
title: Implementación de una zona DMZ entre Azure e Internet
titleSuffix: Azure Reference Architectures
description: Cómo implementar una arquitectura de red híbrida segura con acceso a Internet en Azure.
author: telmosampaio
ms.date: 10/22/2018
ms.custom: seodec18
ms.openlocfilehash: 10c8a23ab09da0555de6a51bc082deceb8c462ff
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/04/2019
ms.locfileid: "54011538"
---
# <a name="implement-a-dmz-between-azure-and-the-internet"></a>Implementación de una zona DMZ entre Azure e Internet

Esta arquitectura de referencia muestra una red híbrida segura que extiende una red local a Azure y también acepta el tráfico de Internet. [**Implemente esta solución**](#deploy-the-solution).

![Arquitectura de red híbrida segura](./images/dmz-public.png)

*Descargue un [archivo Visio][visio-download] de esta arquitectura.*

Esta arquitectura de referencia extiende la arquitectura descrita en [Implementación de una red perimetral entre Azure y el centro de datos local][implementing-a-secure-hybrid-network-architecture]. Agrega una zona DMZ pública que gestiona el tráfico de Internet, además de la zona DMZ privada que gestiona el tráfico de la red local.

Los usos habituales de esta arquitectura incluyen:

- Aplicaciones híbridas donde parte de las cargas de trabajo se ejecutan de forma local y parte en Azure.
- Infraestructura de Azure que enruta el tráfico entrante procedente del entorno local e Internet.

## <a name="architecture"></a>Arquitectura

La arquitectura consta de los siguientes componentes:

- **Dirección IP pública (PIP)** La dirección IP del punto de conexión público. Los usuarios externos conectados a Internet pueden acceder al sistema mediante esta dirección.
- **Aplicación virtual de red (NVA)**. Esta arquitectura incluye un grupo independiente de NVA para el tráfico que se origina en Internet.
- **Azure Load Balancer**. Todas las solicitudes entrantes de Internet pasan por el equilibrador de carga y se distribuyen a las aplicaciones virtuales de red en la red perimetral pública.
- **Subred de entrada de red perimetral pública**. Esta subred acepta las solicitudes del equilibrador de carga de Azure. Las solicitudes entrantes se pasan a una de las aplicaciones virtuales de red de la red perimetral pública.
- **Subred de salida de red perimetral pública**. Las solicitudes que apueba la aplicación virtual de red pasan por esta subred y se dirigen al equilibrador de carga interno en el nivel web.

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

- El equilibrador de carga accesible desde Internet dirige las solicitudes a las aplicaciones virtuales de red de la subred de la red experimental pública de entrada, y solo en los puertos necesarios de la aplicación.
- Las reglas NSG de las subredes de la red perimetral pública de entrada y salida impiden que las aplicaciones virtuales de red se pongan en peligro, ya que bloquean las solicitudes que se encuentran fuera de estas reglas.
- La configuración de enrutamiento NAT de las aplicaciones virtuales de red dirige las solicitudes entrantes en los puertos 80 y 443 al equilibrador de carga de nivel web, pero omite las solicitudes en todos los demás puertos.

Todas las solicitudes entrantes se deben registrar en todos los puertos. Realice auditorías de los registros con regularidad y preste atención a las solicitudes que se encuentran fuera de los parámetros previstos, ya que podrían ser indicio de intentos de intrusión.

## <a name="deploy-the-solution"></a>Implementación de la solución

Se puede encontrar una implementación de una arquitectura de referencia que implementa estas recomendaciones en [GitHub][github-folder].

### <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-resources"></a>Implementación de recursos

1. Vaya a la carpeta `/dmz/secure-vnet-dmz` del repositorio de GitHub de las arquitecturas de referencia.

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

    ![Captura de pantalla del campo de dirección IP](./images/local-net-gw.png)

5. Haga clic en **Guardar** y espere a que finalice la operación. Puede tardar unos 5 minutos.

6. Busque el recurso llamado `onprem-vpn-gateway1-pip`. Copie la dirección IP que se muestra en la hoja **Información general**.

7. Busque el recurso llamado `ra-vpn-lgw`.

8. Haga clic en la hoja **Configuración**. En **Dirección IP**, pegue la dirección IP del paso 6.

9. Haga clic en **Guardar** y espere a que finalice la operación.

10. Para comprobar la conexión, vaya a la hoja **Conexiones** de cada puerta de enlace. El estado debe ser **Conectado**.

### <a name="verify-that-network-traffic-reaches-the-web-tier"></a>Compruebe que el tráfico de red llega al nivel web

1. En Azure Portal, vaya al grupo de recursos que ha creado.

2. Busque el recurso llamado `pub-dmz-lb`, que es el equilibrador de carga que hay delante de la red perimetral pública.

3. Copie la dirección IP pública de la hoja **Información general** y ábrala en un explorador web. Debería ver la página principal predeterminada del servidor Apache2.

4. Busque el recurso llamado `int-dmz-lb`, que es el equilibrador de carga que hay delante de la red perimetral privada. Copie la dirección IP privada de la hoja **Información general**.

5. Busque la máquina virtual denominada `jb-vm1`. Haga clic en **Conectar** y use el escritorio remoto para conectarse a la máquina virtual. El nombre de usuario y la contraseña se especifican en el archivo onprem.json.

6. En la sesión del escritorio remoto, abra un explorador web y vaya a la dirección IP del paso 4. Debería ver la página principal predeterminada del servidor Apache2.

[availability-set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/dmz/secure-vnet-dmz

[implementing-a-secure-hybrid-network-architecture]: ./secure-vnet-hybrid.md
[iptables]: https://help.ubuntu.com/community/IptablesHowTo
[lb-probe]: /azure/load-balancer/load-balancer-custom-probe-overview
[load-balancer]: /azure/load-balancer/load-balancer-Internet-overview
[network-security-group]: /azure/virtual-network/virtual-networks-nsg

[visio-download]: https://archcenter.blob.core.windows.net/cdn/dmz-reference-architectures.vsdx
