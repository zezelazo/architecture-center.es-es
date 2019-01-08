---
title: Sitios web en WordPress seguros y muy escalables
titleSuffix: Azure Example Scenarios
description: Cree un sitio web de WordPress altamente escalable y seguro para eventos multimedia.
author: david-stanford
ms.date: 09/18/2018
ms.openlocfilehash: c0dad12e1da1f17b75d0661195123da4a8267152
ms.sourcegitcommit: bb7fcffbb41e2c26a26f8781df32825eb60df70c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/20/2018
ms.locfileid: "53644057"
---
# <a name="highly-scalable-and-secure-wordpress-website"></a>Sitio web altamente escalable y seguro de WordPress

Este escenario de ejemplo es aplicable a las empresas que necesitan una instalación segura y altamente escalable de WordPress. Este escenario se basa en una implementación que utilizó para una gran convención y se pudo escalar correctamente para satisfacer el incremento del tráfico que las sesiones generaban en el sitio.

## <a name="relevant-use-cases"></a>Casos de uso pertinentes

Otros casos de uso pertinentes incluyen:

- Eventos de medios que provocan picos de tráfico.
- Blogs que usan WordPress como su sistema de administración de contenido.
- Sitios web de empresa o de comercio electrónico que usan WordPress.
- Sitios web creados con otros sistemas de administración de contenido.

## <a name="architecture"></a>Arquitectura

[![Introducción a la arquitectura de los componentes de Azure implicados en una implementación de WordPress escalable y segura](media/secure-scalable-wordpress.png)](media/secure-scalable-wordpress.png#lightbox)

Este escenario incluye una instalación segura y escalable de WordPress que utiliza servidores web de Ubuntu y MariaDB. Hay dos flujos de datos distintos en este escenario, el primero son los usuarios que acceden al sitio web:

1. Los usuarios acceden al sitio web front-end a través de una red CDN.
2. La red CDN usa un equilibrador de carga de Azure como origen y extrae los datos que no se almacenan en caché desde allí.
3. El equilibrador de carga de Azure distribuye las solicitudes a los conjuntos de escalado de máquinas virtuales de servidores web.
4. La aplicación WordPress extrae cualquier información dinámica de los clústeres de MariaDB; todo el contenido estático se hospeda en Azure Files.
5. Las claves SSL se almacenan en Azure Key Vault.

El segundo flujo de trabajo es cómo contribuyen los autores al nuevo contenido:

1. Los autores de conectan de forma segura a la puerta de enlace de VPN pública.
2. La información de autenticación de VPN se almacena en Azure Active Directory.
3. A continuación, se establece una conexión con el JumpBox de administración.
4. Desde el JumpBox de administración, el autor puede conectarse al equilibrador de carga de Azure para la creación de clústeres.
5. El equilibrador de carga de Azure distribuye el tráfico a los conjuntos de escalado de máquinas virtuales de servidores web que tienen acceso de escritura al clúster de MariaDB.
6. El nuevo contenido estático se carga en los archivos de Azure y el contenido dinámico se escribe en el clúster de MariaDB.
7. Estos cambios se replican, a continuación, en la región alternativa mediante rsync o la replicación maestro/subordinado.

### <a name="components"></a>Componentes

- [Azure Content Delivery Network (CDN)](/azure/cdn/cdn-overview) es una red distribuida de servidores que proporciona contenido web a los usuarios de manera eficaz. Las redes CDN minimizan la latencia al guardar contenido almacenado en caché en servidores perimetrales en ubicaciones de punto de presencia cercanas a los usuarios finales.
- [Las redes virtuales](/azure/virtual-network/virtual-networks-overview) permiten que varios recursos como, por ejemplo, máquinas virtuales, se puedan comunicar de forma segura entre ellos, con Internet y con redes locales. Las redes virtuales proporcionan aislamiento y segmentación, filtrado y enrutamiento de tráfico, y permiten la conexión entre ubicaciones. Las dos redes están conectadas mediante emparejamiento de VNET.
- Los [grupos de seguridad de red](/azure/virtual-network/security-overview) contienen una lista de reglas de seguridad que permiten o deniegan el tráfico entrante o saliente en función de las direcciones IP de origen o destino, el puerto y el protocolo. Las redes virtuales de este escenario se protegen con reglas de grupo de seguridad de red que restringen el flujo de tráfico entre los componentes de la aplicación.
- Los [equilibradores de carga](/azure/load-balancer/load-balancer-overview) distribuyen el tráfico entrante según reglas y sondeos de mantenimiento. Una instancia de Load Balancer proporciona baja latencia y alto rendimiento, y puede escalar hasta millones de flujos para todas las aplicaciones TCP y UDP. En este escenario, se utiliza un equilibrador de carga para distribuir el tráfico de la red de entrega de contenido a los servidores web front-end.
- Los [conjuntos de escalado de máquinas virtuales][docs-vmss] permiten crear y administrar un grupo de máquinas virtuales idénticas con equilibrio de carga. El número de instancias de máquina virtual puede aumentar o disminuir automáticamente según la demanda, o de acuerdo a una programación definida. En este escenario se utilizan dos conjuntos de escalado de máquinas virtuales independientes: uno para servidores web front-end que sirven contenido y otro para servidores web front-end utilizados para crear nuevo contenido.
- [Azure Files](/azure/storage/files/storage-files-introduction) proporciona un recurso compartido de archivos totalmente administrados en la nube que hospeda todo el contenido de WordPress en este escenario, para que todas las máquinas virtuales tengan acceso a los datos.
- [Azure Key Vault](/azure/key-vault/key-vault-overview) se usa para almacenar y controlar estrechamente el acceso a las contraseñas, certificados y claves.
- [Azure Active Directory (Azure AD)](/azure/active-directory/fundamentals/active-directory-whatis) es un directorio multiinquilino, basado en la nube y un servicio de administración de identidades. En este escenario, Azure AD proporciona servicios de autenticación para el sitio web y los túneles VPN.

### <a name="alternatives"></a>Alternativas

- [SQL Server para Linux](/azure/virtual-machines/linux/sql/sql-server-linux-virtual-machines-overview) puede reemplazar el almacén de datos de MariaDB.
- [Azure Database for MySQL](/azure/mysql/overview) puede reemplazar el almacén de datos de MariaDB si prefiere una solución totalmente administrada.

## <a name="considerations"></a>Consideraciones

### <a name="availability"></a>Disponibilidad

Las instancias de máquinas virtuales de este escenario están implementadas en varias regiones, con los datos replicados entre las dos a través de RSYNC para el contenido de WordPress y replicación maestro/subordinado para los clústeres de MariaDB.

Para ver otros temas de disponibilidad, consulte la [lista de comprobación de disponibilidad][availability] que encontrará en Azure Architecture Center.

### <a name="scalability"></a>Escalabilidad

Este escenario utiliza conjuntos de escalado de máquinas virtuales para los dos clústeres de servidores web front-end en cada región. Con los conjuntos de escalado, el número de instancias de máquina virtual que se ejecutan en la capa de aplicación de front-end se puede escalar de forma automática en respuesta a la demanda del cliente, o según una programación definida. Para más información, consulte [Introducción al escalado automático con conjuntos de escalado de máquinas virtuales][docs-vmss-autoscale].

El back-end es un clúster de MariaDB en un conjunto de disponibilidad. Para más información, consulte el [tutorial sobre clústeres de MariaDB][mariadb-tutorial].

Para ver otros temas de escalabilidad, consulte la [lista de comprobación de escalabilidad][scalability] que encontrará en el centro de arquitectura de Azure.

### <a name="security"></a>Seguridad

Los grupos de seguridad de red protegen todo el tráfico de la red virtual hacia la capa de aplicación de front-end. Las reglas limitan el flujo de tráfico de forma que solo las instancias de máquinas virtuales de la capa de aplicación de front-end pueden acceder al nivel de la base de datos de back-end. No se permite ningún tráfico de Internet saliente procedente del nivel de la base de datos. Para reducir la superficie de ataque, no hay ningún puerto de administración remota directa abierto. Para más información, consulte [Grupos de seguridad de red de Azure][docs-nsg].

Para obtener instrucciones generales sobre el diseño de escenarios seguros, consulte la [documentación de seguridad de Azure][security].

### <a name="resiliency"></a>Resistencia

Junto con el uso de múltiples regiones, la replicación de datos y los conjuntos de escalado de máquinas virtuales, este escenario utiliza equilibradores de carga de Azure. Estos componentes de redes distribuyen el tráfico a las instancias de máquinas virtuales conectadas e incluyen sondeos de estado que garantizan que el tráfico solo se distribuye a las máquinas virtuales correctas. Todos estos componentes de red están dirigidos a través de una red CDN. Esto hace que los recursos y aplicaciones de red sean resistentes frente a problemas que, de lo contrario, interrumpirían el tráfico y afectarían al acceso de los usuarios finales.

Para obtener instrucciones generales sobre el diseño de escenarios resistentes, consulte [Diseño de aplicaciones resistentes de Azure][resiliency].

## <a name="pricing"></a>Precios

Para explorar el costo de ejecutar este escenario, todos los servicios están preconfigurados en la calculadora de costos. Para ver cómo cambiarían los precios en su caso concreto, cambie las variables pertinentes para que coincidan con el tráfico esperado.

Hemos proporcionado un [perfil de costo][pricing] previamente configurado basado en el diagrama de arquitectura proporcionado anteriormente. Para configurar la calculadora de precios para el caso de uso, hay un par de puntos importantes que se deben tener en cuenta:

- ¿Cuánto tráfico espera en términos de GB/mes? La cantidad de tráfico tendrá el impacto más importante en el costo, ya que afectará al número de máquinas virtuales necesarias para mostrar los datos en el conjunto de escalado de máquinas virtuales. Además, estará directamente correlacionado con la cantidad de datos que se muestran a través de la red CDN.
- ¿Qué cantidad de datos nuevos va a escribir en su sitio web? Los datos nuevos escritos en el sitio web están correlacionados con la cantidad de datos que se reflejan entre las regiones.
- ¿Qué parte de su contenido es dinámico? ¿Qué parte es estática? La varianza entre el contenido dinámico y estático influye en la cantidad de datos que se van a recuperar de la capa de base de datos frente a la cantidad de datos que se almacenará en caché en la red CDN.

<!-- links -->
[architecture]: ./media/architecture-secure-scalable-wordpress.png
[mariadb-tutorial]: /azure/virtual-machines/linux/classic/mariadb-mysql-cluster
[docs-vmss]: /azure/virtual-machine-scale-sets/overview
[docs-vmss-autoscale]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview
[docs-nsg]: /azure/virtual-network/security-overview
[security]: /azure/security/
[availability]: ../../checklist/availability.md
[resiliency]: /azure/architecture/resiliency/
[scalability]: /azure/architecture/checklist/scalability
[pricing]: https://azure.com/e/a8c4809dab444c1ca4870c489fbb196b
