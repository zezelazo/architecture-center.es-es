---
title: Extensión de Active Directory Domain Services (AD DS) a Azure
description: >-
  Cómo implementar una arquitectura de red híbrida segura con la autorización de Active Directory en Azure.

  guidance,vpn-gateway,expressroute,load-balancer,virtual-network,active-directory
author: telmosampaio
ms.date: 11/28/2016
pnp.series.title: Identity management
pnp.series.prev: azure-ad
pnp.series.next: adds-forest
ms.openlocfilehash: 007d244f29bf11c6e2bd703c7f4f245d22c02f0f
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2018
---
# <a name="extend-active-directory-domain-services-ad-ds-to-azure"></a>Extensión de Active Directory Domain Services (AD DS) a Azure

En esta arquitectura de referencia se muestra cómo extender el entorno de Active Directory a Azure para proporcionar servicios de autenticación distribuidos mediante [Active Directory Domain Services (AD DS)][active-directory-domain-services].  [**Implemente esta solución**.](#deploy-the-solution)

[![0]][0] 

*Descargue un [archivo Visio][visio-download] de esta arquitectura.*

AD DS se usa para autenticar usuarios, equipos, aplicaciones u otras entidades que se incluyen en un dominio de seguridad. Se puede hospedar de forma local, pero si parte de la aplicación se hospeda en el entorno local y parte en Azure, puede que resulte más eficaz replicar esta funcionalidad en Azure. Esto puede reducir la latencia causada por el envío de solicitudes de autorización locales y de autenticación desde la nube a los servicios AD DS que se ejecutan en un entorno local. 

Esta arquitectura suele usarse cuando la red local y la red virtual de Azure están conectadas mediante una conexión VPN o ExpressRoute. Esta arquitectura también admite la replicación bidireccional, lo que significa que los cambios se pueden realizar en el entorno local o en la nube, de tal forma que se mantiene la coherencia de ambos orígenes. Los usos típicos de esta arquitectura incluyen aplicaciones híbridas en las que la funcionalidad se distribuye entre el entorno local y Azure, y las aplicaciones y los servicios que realizan la autenticación con Active Directory.

Para consideraciones adicionales, consulte [Selección de una solución para la integración de Active Directory local con Azure][considerations]. 

## <a name="architecture"></a>Architecture 

Esta arquitectura extiende la arquitectura mostrada en [Red perimetral entre Internet y Azure][implementing-a-secure-hybrid-network-architecture-with-internet-access]. Tiene los siguientes componentes.

* **Red local**. La red local incluye servidores locales de Active Directory que pueden realizar la autenticación y autorización de componentes que se encuentran en entornos locales.
* **Servidores de Active Directory**. Se trata de controladores de dominio que implementan servicios de directorio (AD DS) que se ejecutan como máquinas virtuales en la nube. Estos servidores pueden proporcionar la autenticación de componentes que se ejecutan en la red virtual de Azure.
* **Subred de Active Directory**. Los servidores de AD DS se hospedan en una subred independiente. Las reglas de los grupos de seguridad de red (NSG) protegen los servidores de AD DS y proporcionan un firewall contra el tráfico procedente de orígenes inesperados.
* **Azure Gateway y sincronización de Active Directory**. Azure Gateway proporciona una conexión entre la red local y la red virtual de Azure. Puede ser una [conexión VPN][azure-vpn-gateway] o [Azure ExpressRoute][azure-expressroute]. Todas las solicitudes de sincronización entre los servidores de Active Directory en la nube y locales pasan a través de la puerta de enlace. Las rutas definidas por el usuario (UDR) controlan el enrutamiento del tráfico local que pasa a Azure. El tráfico hacia y desde los servidores de Active Directory no pasa a través de las aplicaciones virtuales de red (NVA) utilizadas en este escenario.

Para más información sobre cómo configurar UDR y NVA, vea [Implementación de una arquitectura de red híbrida segura en Azure][implementing-a-secure-hybrid-network-architecture]. 

## <a name="recommendations"></a>Recomendaciones

Las siguientes recomendaciones sirven para la mayoría de los escenarios. Sígalas a menos que tenga un requisito concreto que las invalide. 

### <a name="vm-recommendations"></a>Recomendaciones de VM

Determine los requisitos de [tamaño de la máquina virtual][vm-windows-sizes] en función del volumen esperado de solicitudes de autenticación. Use las especificaciones de los equipos que hospedan AD DS de forma local como punto de partida y hágalas coincidir con los tamaños de máquina virtual de Azure. Una vez realizada la implementación, supervise la utilización y escale o reduzca verticalmente en función de la carga real de las máquinas virtuales. Para más información sobre cómo ajustar el tamaño de los controladores de dominio de AD DS, vea [Capacity Planning for Active Directory Domain Services][capacity-planning-for-adds] (Planeamiento de capacidad de Active Directory Domain Services).

Crear un disco de datos virtual independiente para almacenar la base de datos, los registros y SYSVOL de Active Directory. No almacene estos elementos en el mismo disco que el sistema operativo. Tenga en cuenta que, de forma predeterminada, los disco de datos que están conectados a una máquina virtual usan una caché de escritura simultánea. Sin embargo, esta forma de almacenamiento en caché puede entrar en conflicto con los requisitos de AD DS. Por ello, establezca la *preferencia de caché de host* en el disco de datos en *Ninguna*. Para más información, vea [Selección de ubicación de la base de datos y SYSVOL de Windows Server AD DS][adds-data-disks].

Implemente al menos dos máquinas virtuales que ejecutan AD DS como controladores de dominio y agréguelas a un [conjunto de disponibilidad][availability-set].

### <a name="networking-recommendations"></a>Recomendaciones de redes

Configure la interfaz de red (NIC) de VM para cada servidor de AD DS con una dirección IP privada estática para la compatibilidad total del Servicio de nombres de dominio (DNS). Para más información, vea [Configuración de direcciones IP privadas para una máquina virtual mediante Azure Portal][set-a-static-ip-address].

> [!NOTE]
> No configure la NIC de VM para cualquier AD DS con una dirección IP pública. Vea las [consideraciones de seguridad][security-considerations] para más información.
> 
> 

El NSG de la subred de Active Directory requiere reglas para permitir el tráfico entrante desde el entorno local. Para obtener información detallada sobre los puertos que AD DS utiliza, vea [Active Directory and Active Directory Domain Services Port Requirements][ad-ds-ports] (Requisitos de puertos de Active Directory Domain Services y Active Directory). Asegúrese también de que las tablas UDR no enruten el tráfico de AD DS a través de las NVA utilizadas en esta arquitectura. 

### <a name="active-directory-site"></a>Sitio de Active Directory

En AD DS, un sitio representa una ubicación física, una red o un conjunto de dispositivos. Los sitios de AD DS se utilizan para administrar la replicación de base de datos de AD DS mediante la agrupación de objetos de AD DS que se encuentran cerca unos de otros y que están conectados mediante una red de alta velocidad. AD DS incluye la lógica para seleccionar la mejor estrategia para replicar la base de datos de AD DS entre sitios.

Se recomienda crear un sitio de AD DS, incluidas las subredes definidas para la aplicación en Azure. Después, configure un vínculo de sitio entre los sitios de AD DS locales, y AD DS realizará automáticamente la replicación de base de datos más eficaz posible. Tenga en cuenta que esta replicación de base de datos requiere poco más aparte de la configuración inicial.

### <a name="active-directory-operations-masters"></a>Maestros de operaciones de Active Directory

El rol de maestro de operaciones se puede asignar a los controladores de dominio de AD DS para admitir la comprobación de coherencia entre instancias de bases de datos de AD DS replicadas. Hay cinco roles de maestro de operaciones: maestro de esquema, maestro de nomenclatura de dominios, maestro de identificadores relativos, maestro emulador del controlador de dominio principal y maestro de infraestructuras. Para más información sobre estos roles, vea [What are Operations Masters?][ad-ds-operations-masters] (¿Qué son los maestros de operaciones?).

Se recomienda no asignar roles de maestros de operaciones a los controladores de dominio implementados en Azure.

### <a name="monitoring"></a>Supervisión

Supervise los recursos de las máquinas virtuales del controlador de dominio y los servicios de AD DS y cree un plan para corregir rápidamente los problemas. Para más información, vea [Monitoring Active Directory][monitoring_ad] (Supervisión de Active Directory). También puede instalar herramientas como [Microsoft Systems Center][microsoft_systems_center] en un servidor de supervisión (vea el diagrama de la arquitectura) para ayudar a realizar estas tareas.  

## <a name="scalability-considerations"></a>Consideraciones sobre escalabilidad

AD DS está diseñado para ofrecer escalabilidad. No es necesario configurar un equilibrador de carga o un controlador de tráfico para dirigir las solicitudes a controladores de dominio de AD DS. La única consideración de escalabilidad consiste en configurar las máquinas virtuales que ejecutan AD DS con el tamaño correcto para los requisitos de la carga de red, supervisar la carga en las máquinas virtuales y escalar y reducir verticalmente, según sea necesario.

## <a name="availability-considerations"></a>Consideraciones sobre disponibilidad

Implemente las máquinas virtuales que ejecutan AD DS en un [conjunto de disponibilidad][availability-set]. Además, considere la posibilidad de asignar el rol de [maestro de operaciones en modo de espera][standby-operations-masters] al menos a un servidor y, posiblemente más, en función de los requisitos. Un maestro de operaciones en modo de espera es una copia activa del maestro de operaciones que puede usarse en lugar del servidor maestro de operaciones principal durante una conmutación por error.

## <a name="manageability-considerations"></a>Consideraciones sobre la manejabilidad

Realice copias de seguridad periódicas de AD DS. No basta con copiar los archivos VHD de los controladores de dominio en lugar de realizar copias de seguridad periódicas, dado que el archivo de base de datos de AD DS del disco duro virtual puede no tener un estado coherente cuando se copia, lo que puede hacer que sea imposible reiniciar la base de datos.

No apague una máquina virtual del controlador de dominio mediante Azure Portal. En su lugar, apague y reinicie desde el sistema operativo invitado. Si la operación de apagado se realiza desde el portal, la máquina virtual se desasigna, lo que hace que `VM-GenerationID` y `invocationID` se restablezcan en el repositorio de Active Directory. Esto descarta el grupo de identificadores relativos (RID) de AD DS y marca SYSVOL como no autoritativo; además, puede requerir la reconfiguración del controlador de dominio.

## <a name="security-considerations"></a>Consideraciones sobre la seguridad

Los servidores de AD DS proporcionan servicios de autenticación y son un objetivo muy atractivo para los ataques. Para protegerlos, evite la conexión directa a Internet mediante la colocación de los servidores de AD DS en una subred independiente con un NSG que actúe como un firewall. Cierre todos los puertos en los servidores de AD DS, excepto los necesarios para la autenticación, autorización y sincronización del servidor. Para más información, vea [Active Directory and Active Directory Domain Services Port Requirements][ad-ds-ports] (Requisitos de puertos de Active Directory Domain Services y Active Directory).

Considere la posibilidad de implementar un perímetro de seguridad adicional alrededor de los servidores con un par de subredes y NVA, como se describe en [Implementación de una arquitectura de red híbrida segura con acceso a Internet en Azure][implementing-a-secure-hybrid-network-architecture-with-internet-access].

Utilice BitLocker o Azure Disk Encryption para cifrar el disco que hospeda la base de datos de AD DS.

## <a name="deploy-the-solution"></a>Implementación de la solución

Hay una solución disponible en [GitHub][github] para implementar esta arquitectura de referencia. Necesitará la versión más reciente de la [CLI de Azure][azure-powershell] para ejecutar el script de Powershell que implementa la solución. Para implementar la arquitectura de referencia, siga estos pasos:

1. Descargue o clone la carpeta de soluciones de [GitHub][github] en la máquina local.

2. Abra la CLI de Azure y navegue hasta la carpeta local de la solución.

3. Ejecute el siguiente comando:
    ```Powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> <mode>
    ```
    Reemplace `<subscription id>` con la identificación de su suscripción de Azure.
    Para `<location>`, especifique una región de Azure, como `eastus` o `westus`.
    El parámetro `<mode>` controla la granularidad de la implementación y puede ser uno de los siguientes valores:
    * `Onpremise`: implementa el entorno local simulado.
    * `Infrastructure`: implementa la infraestructura de VNet y el JumBox en Azure.
    * `CreateVpn`: implementa la puerta de enlace de red virtual de Azure y la conecta a la red local simulada.
    * `AzureADDS`: implementa las máquinas virtuales que actúan como servidores de AD DS, implementa Active Directory en estas máquinas virtuales e implementa el dominio en Azure.
    * `Workload`: implementa las redes perimetrales pública y privada y el nivel de carga de trabajo.
    * `All`: implementa todas las implementaciones anteriores. **Esta es la opción recomendada si no tiene una red local existente pero quiere implementar la arquitectura de referencia completa que se ha descrito anteriormente para pruebas o evaluación.**

4. Espere a que la implementación se complete. Si va a realizar la implementación `All`, tardará varias horas.

## <a name="next-steps"></a>Pasos siguientes

* Conozca los procedimientos recomendados para [crear un bosque de recursos de AD DS][adds-resource-forest] en Azure.
* Conozca los procedimientos recomendados para [crear una infraestructura de Active Directory Federation Services (AD FS)][adfs] en Azure.

<!-- links -->
[adds-resource-forest]: adds-forest.md
[adfs]: adfs.md

[implementing-a-secure-hybrid-network-architecture]: ../dmz/secure-vnet-hybrid.md
[implementing-a-secure-hybrid-network-architecture-with-internet-access]: ../dmz/secure-vnet-dmz.md

[active-directory-domain-services]: https://technet.microsoft.com/library/dd448614.aspx
[adds-data-disks]: https://msdn.microsoft.com/library/azure/jj156090.aspx#BKMK_PlaceDB
[ad-ds-operations-masters]: https://technet.microsoft.com/library/cc779716(v=ws.10).aspx
[ad-ds-ports]: https://technet.microsoft.com/library/dd772723(v=ws.11).aspx
[availability-set]: /azure/virtual-machines/virtual-machines-windows-create-availability-set
[azure-expressroute]: https://azure.microsoft.com/documentation/articles/expressroute-introduction/
[azure-powershell]: /powershell/azureps-cmdlets-docs
[azure-vpn-gateway]: https://azure.microsoft.com/documentation/articles/vpn-gateway-about-vpngateways/
[capacity-planning-for-adds]: http://social.technet.microsoft.com/wiki/contents/articles/14355.capacity-planning-for-active-directory-domain-services.aspx
[considerations]: ./considerations.md
[GitHub]: https://github.com/mspnp/reference-architectures/tree/master/identity/adds-extend-domain
[microsoft_systems_center]: https://www.microsoft.com/server-cloud/products/system-center-2016/
[monitoring_ad]: https://msdn.microsoft.com/library/bb727046.aspx
[security-considerations]: #security-considerations
[set-a-static-ip-address]: https://azure.microsoft.com/documentation/articles/virtual-networks-static-private-ip-arm-pportal/
[standby-operations-masters]: https://technet.microsoft.com/library/cc794737(v=ws.10).aspx
[visio-download]: https://archcenter.blob.core.windows.net/cdn/identity-architectures.vsdx
[vm-windows-sizes]: /azure/virtual-machines/virtual-machines-windows-sizes

[0]: ./images/adds-extend-domain.png "Protección de la arquitectura de red híbrida con Active Directory"
