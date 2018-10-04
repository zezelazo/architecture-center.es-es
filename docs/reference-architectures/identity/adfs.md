---
title: Implementación de los Servicios de federación de Active Directory (AD FS) en Azure
description: >-
  Procedimiento para implementar una arquitectura de red híbrida segura con la autorización de los Servicios de Federación de Active Directory en Azure.

  guidance,vpn-gateway,expressroute,load-balancer,virtual-network,active-directory
author: telmosampaio
ms.date: 11/28/2016
pnp.series.title: Identity management
pnp.series.prev: adds-forest
cardTitle: Extend AD FS to Azure
ms.openlocfilehash: f48f8e41b6b90931e7393808cf41ae01d7187416
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/28/2018
ms.locfileid: "47429271"
---
# <a name="extend-active-directory-federation-services-ad-fs-to-azure"></a>Extensión de Servicios de federación de Active Directory (AD FS) a Azure

Esta arquitectura de referencia implementa una red segura híbrida que extiende la red local a Azure y usa los [Servicios de federación de Active Directory (AD FS)][active-directory-federation-services] para realizar la autenticación y la autorización federada para los componentes que se ejecutan en Azure. [**Implemente esta solución**.](#deploy-the-solution)

[![0]][0]

*Descargue un [archivo Visio][visio-download] de esta arquitectura.*

AD FS puede hospedarse de forma local, pero, si la aplicación es un híbrido en el que algunas partes se implementan en Azure, puede ser más eficaz replicar AD FS en la nube. 

El diagrama muestra los siguientes escenarios:

* El código de la aplicación de una organización asociada tiene acceso a una aplicación web que se hospeda dentro de la red virtual de Azure.
* Un usuario externo y registrado con las credenciales almacenadas en Active Directory Domain Services (DS) tiene acceso a una aplicación web que se hospeda dentro de la red virtual de Azure.
* Un usuario conectado a la red virtual con un dispositivo autorizado ejecuta una aplicación web que se hospeda dentro de la red virtual de Azure.

Los usos habituales de esta arquitectura incluyen:

* Aplicaciones híbridas donde una parte de las cargas de trabajo se ejecutan de forma local y otra parte en Azure.
* Soluciones que usan autorización federada para exponer las aplicaciones web a las organizaciones asociadas.
* Sistemas que permiten el acceso desde exploradores web que se ejecutan fuera del firewall de la organización.
* Sistemas que permiten a los usuarios el acceso a las aplicaciones web mediante la conexión de dispositivos externos autorizados como equipos remotos, portátiles y otros dispositivos móviles. 

Esta arquitectura de referencia se centra en la *federación pasiva*, en la que los servidores de federación deciden cómo y cuándo autenticar a un usuario. El usuario proporciona información de inicio de sesión cuando se inicia la aplicación. Este mecanismo se usa normalmente en los exploradores web e implica un protocolo que redirige el explorador a un sitio donde el usuario se autentica. AD FS también admite la *federación activa*, en la que una aplicación asume la responsabilidad de proporcionar las credenciales sin la intervención del usuario, pero ese escenario está fuera del ámbito de esta arquitectura.

Para consideraciones adicionales, consulte [Selección de una solución para la integración de Active Directory local con Azure][considerations]. 

## <a name="architecture"></a>Arquitectura

Esta arquitectura amplía la implementación que se describe en [Extensión de AD DS a Azure][extending-ad-to-azure]. Contiene los componentes siguientes:

* **Subred de AD DS**. Los servidores de AD DS se encuentran en su propia subred con reglas de grupo de seguridad de red (NSG) que actúan como un firewall.

* **Servidores de AD FS**. Controladores de dominio que se ejecutan como máquinas virtuales en Azure. Estos servidores proporcionan autenticación de las identidades locales dentro del dominio.

* **Subred de AD FS**. Los servidores de AD FS se encuentran dentro de su propia subred con reglas NSG que actúan como un firewall.

* **Servidores de AD FS**. Los servidores de AD FS proporcionan autenticación y autorización federadas. En esta arquitectura, se realizan las siguientes tareas:
  
  * Recibir tokens de seguridad que contienen notificaciones realizadas por un servidor de federación asociado en nombre del usuario asociado. AD FS comprueba que los tokens son válidos antes de pasar las notificaciones a la aplicación web que se ejecuta en Azure para autorizar las solicitudes. 
  
    La aplicación web que se ejecuta en Azure es el *usuario de confianza*. El servidor de federación asociado debe emitir notificaciones que la aplicación web entienda. Los servidores de federación asociados se conocen como *asociados de cuenta*, ya que envían las solicitudes de acceso en nombre de las cuentas autenticadas en la organización del asociado. Los servidores de AD FS se denominan *asociados de recursos* ya que proporcionan acceso a los recursos (la aplicación web).

  * Autenticar y autorizar las solicitudes entrantes de los usuarios externos que ejecutan un explorador web o un dispositivo que necesita tener acceso a las aplicaciones web, mediante AD DS y el [Servicio de registro de dispositivos de Active Directory][ADDRS].
    
  Los servidores de AD FS se configuran como una granja de servidores a los que se accede a través de un equilibrador de carga de Azure. Esta implementación mejora la disponibilidad y la escalabilidad. Los servidores de AD FS no se exponen directamente a Internet. Todo el tráfico de Internet se filtra a través de servidores proxy de aplicación web de AD FS y una red perimetral (también conocida como DMZ).

  Para obtener más información acerca del funcionamiento de AD FS, consulte [Introducción a los Servicios de federación de Active Directory][active-directory-federation-services-overview]. Además, el artículo [Implementación de AD FS en Azure][adfs-intro] contiene una introducción detallada paso a paso para la implementación.

* **Subred de proxy de AD FS**. Los servidores proxy de AD FS pueden estar dentro de su propia subred, con reglas NSG que proporcionan la protección. Los servidores de esta subred se exponen a Internet a través de un conjunto de aplicaciones virtuales de red que proporcionan un firewall entre la red virtual de Azure e Internet.

* **Servidores proxy de aplicación web (WAP) de AD FS**. Estas máquinas virtuales actúan como servidores de AD FS para las solicitudes entrantes de las organizaciones asociadas y los dispositivos externos. Los servidores WAP actúan como un filtro, blindando a los servidores de AD FS frente al acceso directo desde Internet. Al igual que con los servidores de AD FS, implementar los servidores WAP en una granja de servidores con equilibrio de carga ofrece mayor disponibilidad y escalabilidad que la implementación de una colección de servidores independientes.
  
  > [!NOTE]
  > Para información detallada acerca de cómo instalar los servidores WAP, consulte [Instalar y configurar el servidor de Proxy de aplicación web][install_and_configure_the_web_application_proxy_server]
  > 
  > 

* **Organización del asociado**. La organización de un asociado ejecuta una aplicación web que solicita acceso a una aplicación web que se ejecuta en Azure. El servidor de federación de la organización del asociado autentica localmente las solicitudes y envía tokens de seguridad que contienen notificaciones a AD FS que se ejecuta en Azure. AD FS en Azure valida los tokens de seguridad y, si son válidos, puede pasar las notificaciones a la aplicación web que se ejecuta en Azure para que las autorice.
  
  > [!NOTE]
  > También puede configurar un túnel VPN con la puerta de enlace de Azure para proporcionar acceso directo a AD FS para los asociados de confianza. Las solicitudes recibidas desde estos asociados no pasan a través de los servidores WAP.
  > 
  > 

Para más información sobre los elementos de la arquitectura que no están relacionados con AD FS, vea lo siguiente:
- [Implementación de una arquitectura de red híbrida segura en Azure][implementing-a-secure-hybrid-network-architecture]
- [Implementación de una arquitectura de red híbrida segura con acceso a Internet en Azure][implementing-a-secure-hybrid-network-architecture-with-internet-access]
- [Implementación de una arquitectura de red híbrida segura con identidades de Active Directory en Azure][extending-ad-to-azure].


## <a name="recommendations"></a>Recomendaciones

Las siguientes recomendaciones sirven para la mayoría de los escenarios. Sígalas a menos que tenga un requisito concreto que las invalide. 

### <a name="vm-recommendations"></a>Recomendaciones de VM

Cree máquinas virtuales con recursos suficientes para controlar el volumen de tráfico esperado. Utilice el tamaño de las máquinas existentes que hospedan AD FS en una instalación local como punto de partida. Supervise la utilización de recursos. Puede cambiar el tamaño de las máquinas virtuales y reducirlas verticalmente si son demasiado grandes.

Siga las recomendaciones enumeradas en [Ejecución de una máquina virtual Windows en Azure][vm-recommendations].

### <a name="networking-recommendations"></a>Recomendaciones de redes

Configure la interfaz de red para cada una de las máquinas virtuales que hospedan servidores de AD FS y WAP con direcciones IP privadas estáticas.

No conceda a las máquinas virtuales de AD FS direcciones IP públicas. Para más información, consulte la sección Consideraciones de seguridad.

Establezca la dirección IP de los servidores del servicio de nombres de dominio preferido y secundario (DNS) para las interfaces de red de cada máquina virtual AD FS y WAP para hacer referencia a las máquinas virtuales de Active Directory DS. Las máquinas virtuales de Active Directory DS deben estar ejecutando DNS. Este paso es necesario para permitir que cada máquina virtual se una al dominio.

### <a name="ad-fs-availability"></a>Disponibilidad de AD FS 

Cree una granja de servidores de AD FS con al menos dos servidores para aumentar la disponibilidad del servicio. Use cuentas de almacenamiento diferentes para cada máquina virtual de AD FS en la granja de servidores. Este enfoque ayuda a garantizar que un error en una única cuenta de almacenamiento no haga que toda la granja quede inaccesible.

> [!IMPORTANT]
> Se recomienda usar [discos administrados](/azure/storage/storage-managed-disks-overview). Los discos administrados no requieren una cuenta de almacenamiento. Solo debe especificar el tamaño y el tipo de disco, y se implementa en un modo de alta disponibilidad. Nuestras [arquitecturas de referencia](/azure/architecture/reference-architectures/) no implementan actualmente discos administrados, pero los [bloques de creación de plantillas](https://github.com/mspnp/template-building-blocks/wiki) se actualizarán para implementar los discos administrados en la versión 2.

Cree conjuntos de disponibilidad de Azure diferentes para las máquinas virtuales de AD FS y WAP. Asegúrese de que hay al menos dos máquinas virtuales en cada conjunto. Cada conjunto de disponibilidad debe tener dos dominios de actualización y dos dominios de error, como mínimo.

Configure los equilibradores de carga para las máquinas virtuales de AD FS y WAP de la manera siguiente:

* Use un equilibrador de carga de Azure para proporcionar acceso externo a las máquinas virtuales de WAP y uno interno para distribuir la carga entre los servidores de AD FS en la granja de servidores.
* Pase únicamente el tráfico que aparezca en el puerto 443 (HTTPS) a los servidores de AD FS y WAP.
* Asigne al equilibrador de carga una dirección IP estática.
* Crear un sondeo de mantenimiento mediante HTTP en `/adfs/probe`. Para más información, consulte [Hardware Load Balancer Health Checks and Web Application Proxy / AD FS 2012 R2](https://blogs.technet.microsoft.com/applicationproxyblog/2014/10/17/hardware-load-balancer-health-checks-and-web-application-proxy-ad-fs-2012-r2/) (Comprobaciones de mantenimiento de equilibrador de carga de hardware y proxy de aplicación web / AD FS 2012 R2).
  
  > [!NOTE]
  > Los servidores de AD FS usan el protocolo Indicación de nombre de servidor (SNI), por lo que el intento de sondear con un punto de conexión HTTPS desde el equilibrador de carga fracasa.
  > 
  > 
* Agregue un registro *A* DNS al dominio para el equilibrador de carga de AD FS. Especifique la dirección IP del equilibrador de carga y asígnele un nombre en el dominio (por ejemplo, adfs.contoso.com). Se trata del nombre que los clientes y los servidores WAP usan para tener acceso a la granja de servidores de AD FS.

### <a name="ad-fs-security"></a>Seguridad de AD FS 

Evite la exposición directa de los servidores de AD FS a Internet. Los servidores de AD FS son equipos unidos a un dominio que tienen autorización completa para conceder tokens de seguridad. Si un servidor se ve comprometido, un usuario malintencionado puede emitir tokens de acceso completo a todas las aplicaciones web y a todos los servidores de federación que estén protegidos por AD FS. Si el sistema debe controlar las solicitudes de los usuarios externos que no se conectan desde sitios de confianza asociados, use los servidores WAP para controlar estas solicitudes. Para más información, consulte [Ubicación de un servidor proxy de federación][where-to-place-an-fs-proxy].

Coloque los servidores de AD FS y los servidores WAP en subredes independientes con sus propios firewalls. Puede usar las reglas NSG para definir las reglas de firewall. Si necesita una protección más completa, puede implementar un perímetro de seguridad adicional alrededor de los servidores mediante el uso de un par de subredes y aplicaciones virtuales de red (NVA), como se describe en el documento [Implementación de una arquitectura de red híbrida segura con acceso a Internet en Azure][implementing-a-secure-hybrid-network-architecture-with-internet-access]. Todos los firewalls deben permitir el tráfico en el puerto 443 (HTTPS).

Restrinja el acceso de inicio de sesión directo a los servidores de AD FS y WAP. Solo el personal de DevOps debe ser capaz de conectarse.

No una los servidores WAP al dominio.

### <a name="ad-fs-installation"></a>Instalación de AD FS 

El artículo [Implementación de una granja de servidores de federación] [ Deploying_a_federation_server_farm] proporciona instrucciones detalladas para instalar y configurar AD FS. Antes de configurar el primer servidor de AD FS en la granja de servidores, haga lo siguiente:

1. Obtenga un certificado público de confianza para realizar la autenticación de los servidores. El *nombre de sujeto* debe contener el nombre que los clientes usan para tener acceso al servicio de federación. Puede tratarse del nombre DNS registrado para el equilibrador de carga, por ejemplo *adfs.contoso.com* (por motivos de seguridad, evite utilizar nombres con caracteres comodín como **.contoso.com*). Use el mismo certificado en todas las máquinas virtuales de servidores de AD FS. Puede adquirir un certificado de una entidad de certificación de confianza, pero, si su organización usa Servicios de certificados de Active Directory, puede crear los suyos propios. 
   
    El *nombre alternativo del firmante* se utiliza en el servicio de registro de dispositivos (DRS) para habilitar el acceso desde dispositivos externos. Debe tener el formato *enterpriseregistration.contoso.com*.
   
    Para más información, consulte [Obtain and Configure a Secure Sockets Layer (SSL) Certificate for AD FS][adfs_certificates] [Obtención y configuración de un certificado Capa de sockets seguros (SSL) para AD FS].

2. En el controlador de dominio, genere una nueva clave raíz para el Servicio de distribución de claves. Establezca el tiempo efectivo en la hora actual menos 10 horas (esta configuración reduce el retardo que se puede producir en la distribución y la sincronización de las claves a través del dominio). Este paso es necesario para permitir la creación de la cuenta del servicio de grupo que se usa para ejecutar el servicio AD FS. El siguiente comando de PowerShell muestra un ejemplo de cómo hacerlo:
   
    ```powershell
    Add-KdsRootKey -EffectiveTime (Get-Date).AddHours(-10)
    ```

3. Agregue cada máquina virtual del servidor de AD FS al dominio.

> [!NOTE]
> Para instalar AD FS, el controlador de dominio que ejecuta el rol de Operaciones de maestro único flexible (FSMO) del emulador del controlador de dominio principal (PDC) para el dominio debe estar ejecutándose y ser accesible desde las máquinas virtuales de AD FS. <<RBC: ¿hay alguna manera de hacerlo menos repetitivo?>>
> 
> 

### <a name="ad-fs-trust"></a>Confianza de AD FS 

Establezca la confianza de federación entre la instalación de AD FS y los servidores de federación de las organizaciones asociadas. Configure las notificaciones de filtrado y asignación necesarias. 

* El personal de DevOps de cada organización asociada debe agregar una relación de confianza para las aplicaciones web accesibles a través de los servidores de AD FS.
* El personal de DevOps de la organización debe configurar la confianza del proveedor de notificaciones a fin de habilitar los servidores de AD FS para confiar en las notificaciones que las organizaciones asociadas proporcionan.
* También debe configurar AD FS para pasar las notificaciones a las aplicaciones web de la organización.
  
Para más información, consulte [Establishing Federation Trust][establishing-federation-trust] (Establecimiento de la confianza de la federación).

Publique las aplicaciones web de su organización y haga que estén disponibles para los asociados externos con la autenticación previa a través de servidores WAP. Para más información, consulte [Publish Applications using AD FS Preauthentication][publish_applications_using_AD_FS_preauthentication] (Publicación de aplicaciones mediante la autenticación previa de Azure AD)

AD FS admite el aumento y la transformación de tokens. Azure Active Directory no proporciona esta característica. Con AD FS, al configurar las relaciones de confianza, puede:

* Configurar transformaciones de notificación para las reglas de autorización. Por ejemplo, puede asignar la seguridad de grupo desde una representación utilizada por una organización asociada diferente de Microsoft a algo que Active Directory DS pueda autorizar en la organización.
* Transformar las notificaciones de un formato a otro. Por ejemplo, puede pasar de SAML 2.0 a SAML 1.1, si la aplicación solo admite las notificaciones de SAML 1.1.

### <a name="ad-fs-monitoring"></a>Supervisión de AD FS 

El [Módulo de administración de Microsoft System Center para Active Directory Federation Services 2012 R2][oms-adfs-pack] proporciona la supervisión tanto reactiva como proactiva de la implementación de AD FS para el servidor de federación. Este módulo de administración supervisa:

* Los eventos que el servicio AD FS incluye en sus registros de eventos.
* Los datos de rendimiento que los contadores de rendimiento de AD FS recopilan. 
* El estado general del sistema AD FS y las aplicaciones web (usuarios de confianza), además de proporcionar alertas para los problemas críticos y las advertencias. 

## <a name="scalability-considerations"></a>Consideraciones sobre escalabilidad

Las consideraciones siguientes, que se resumen a partir del artículo [Planear la implementación de AD FS][plan-your-adfs-deployment], proporcionan un punto de partida para ajustar el tamaño de las granjas de servidores de AD FS:

* Si tiene menos de 1 000 usuarios, no cree servidores dedicados, sino que, en su lugar, instale AD FS en cada uno de los servidores de Active Directory DS de la nube. Asegúrese de que tiene al menos dos servidores de Active Directory DS para mantener la disponibilidad. Cree un único servidor WAP.
* Si tiene entre 1 000 y 15 000 usuarios, cree dos servidores de AD FS dedicados y dos servidores WAP dedicados.
* Si tiene entre 15 000 y 60 000 usuarios, cree entre tres y cinco servidores AD FS dedicados y al menos dos servidores WAP dedicados.

En estas consideraciones se supone que se utilizan los tamaños de máquinas virtuales duales de cuatro núcleos (D4_v2 estándar o posterior) en Azure.

Si usa Windows Internal Database para almacenar los datos de configuración de AD FS, estará limitado a ocho servidores de AD FS en la granja de servidores. Si prevé que necesitará más en el futuro, utilice SQL Server. Para más información, consulte [La función de la base de datos de configuración de AD FS][adfs-configuration-database].

## <a name="availability-considerations"></a>Consideraciones sobre disponibilidad

Puede usar SQL Server o Windows Internal Database para contener la información de configuración de AD FS. Windows Internal Database proporciona redundancia básica. Los cambios se escriben directamente en una de las bases de datos de AD FS en el clúster de AD FS, mientras que los demás servidores usan replicación de extracción para mantener sus bases de datos actualizadas. Con SQL Server, puede proporcionar una redundancia completa de las bases de datos y una elevada disponibilidad mediante clústeres de conmutación por error o la creación de reflejo.

## <a name="manageability-considerations"></a>Consideraciones sobre la manejabilidad

El personal de DevOps debe estar preparado para realizar las siguientes tareas:

* Administración de los servidores de federación, incluida la administración de la granja de servidores de AD FS, de la directiva de confianza en los servidores de federación y de los certificados usados por los servicios de federación.
* Administración de los servidores WAP, incluida la administración de la granja de servidores WAP y certificados.
* Administración de las aplicaciones web, incluida la configuración de los usuarios de confianza, los métodos de autenticación y las asignaciones de notificaciones.
* Copia de seguridad de los componentes de AD FS.

## <a name="security-considerations"></a>Consideraciones sobre la seguridad

AD FS usa el protocolo HTTPS, así que asegúrese de que las reglas NSG para la subred que contienen las máquinas virtuales del nivel web permiten las solicitudes HTTPS. Estas solicitudes pueden originarse en la red local, las subredes que contienen el nivel web, el nivel empresarial, el nivel de datos, la red perimetral privada, la red perimetral pública y la subred que contiene los servidores de AD FS.

Considere usar un conjunto de aplicaciones de red virtual que registre información detallada sobre el tráfico que atraviesa la frontera de la red virtual con fines de auditoría.

## <a name="deploy-the-solution"></a>Implementación de la solución

Hay una solución disponible en [GitHub][github] para implementar esta arquitectura de referencia. Necesitará la versión más reciente de la [CLI de Azure][azure-cli] para ejecutar el script de Powershell que implementa la solución. Para implementar la arquitectura de referencia, siga estos pasos:

1. Descargue o clone la carpeta de soluciones de [GitHub][github] en la máquina local.

2. Abra la CLI de Azure y navegue hasta la carpeta local de la solución.

3. Ejecute el siguiente comando:
   
    ```powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> <mode>
    ```
   
    Reemplace `<subscription id>` con la identificación de su suscripción de Azure.
   
    Para `<location>`, especifique una región de Azure, como `eastus` o `westus`.
   
    El parámetro `<mode>` controla la granularidad de la implementación y puede ser uno de los siguientes valores:
   
   * `Onpremise`: implementa un entorno local simulado. También puede usar esta implementación para pruebas y experimentar si no tiene una red local existente o si desea probar esta arquitectura de referencia sin cambiar la configuración de la red local existente.
   * `Infrastructure`: implementa la infraestructura de VNet y el JumBox.
   * `CreateVpn`: implementa la puerta de enlace de red virtual de Azure y la conecta a la red local simulada.
   * `AzureADDS`: implementa las máquinas virtuales que actúan como servidores de Active Directory DS, implementa Active Directory en estas máquinas virtuales y crea el dominio en Azure.
   * `AdfsVm`: implementa la máquina virtual de AD FS y se une al dominio en Azure.
   * `PublicDMZ`: implementa la red perimetral pública en Azure.
   * `ProxyVm`: implementa las máquinas virtuales de proxy de AD FS y la une al dominio en Azure.
   * `Prepare`: implementa todas las implementaciones anteriores. **Esta es la opción recomendada si va a crear una implementación totalmente nueva y no tiene una infraestructura local existente.** 
   * `Workload`: opcionalmente, implementa las máquinas virtuales del nivel web, el nivel empresarial y el nivel de datos, así como la red auxiliar. No se incluyen en el modo de implementación `Prepare`.
   * `PrivateDMZ`: opcionalmente, implementa la red perimetral privada en Azure delante de las máquinas virtuales `Workload` implementadas por encima. No se incluye en el modo de implementación `Prepare`.

4. Espere a que la implementación se complete. Si usó la opción `Prepare`, la implementación tarda varias horas en completarse y finaliza con el mensaje`Preparation is completed. Please install certificate to all AD FS and proxy VMs.`

5. Reinicie el JumBox (*ra-adfs-mgmt-vm1* en el grupo *ra-adfs-security-rg*) para permitir que su configuración DNS surta efecto.

6. [Obtenga un certificado SSL para AD FS][adfs_certificates] e instálelo en las máquinas virtuales de AD FS. Tenga en cuenta que puede conectarse a ellas a través del JumBox. Las direcciones IP son <em>10.0.5.4</em> y <em>10.0.5.5</em>. El nombre de usuario predeterminado es <em>contoso\testuser</em> y la contraseña <em>AweSome@PW</em>.
   
   > [!NOTE]
   > Los comentarios en el script de Deploy-ReferenceArchitecture.ps1 proporcionan instrucciones detalladas para crear un certificado de prueba autofirmado y una entidad utilizando el comando `makecert`. Sin embargo, siga estos pasos solo como una **prueba** y no use los certificados generados por makecert en un entorno de producción.
   > 
   > 

7. Ejecute el comando de PowerShell siguiente para implementar la granja de servidores AD FS:
   
    ```powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> Adfs
    ``` 

8. En el JumpBox, vaya a `https://adfs.contoso.com/adfs/ls/idpinitiatedsignon.htm` para probar la instalación de AD FS (puede recibir una advertencia sobre el certificado, que puede ignorar en esta prueba). Compruebe que aparece la página de inicio de sesión de Contoso Corporation. Inicie sesión como <em>contoso\testuser</em> con la contraseña <em>AweS0me@PW</em>.

9. Instale el certificado SSL en las máquinas virtuales de proxy de AD FS. Las direcciones IP son *10.0.6.4* y *10.0.6.5*.

10. Ejecute el comando de PowerShell siguiente para implementar el primer servidor proxy AD FS:
   
    ```powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> Proxy1
    ```

11. Siga las instrucciones que muestra el script para probar la instalación del primer servidor proxy.

12. Ejecute el comando de PowerShell siguiente para implementar el segundo servidor proxy:
    
    ```powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> Proxy2
    ```

13. Siga las instrucciones que muestra el script para probar la configuración del proxy completo.

## <a name="next-steps"></a>Pasos siguientes

* Información acerca de [Azure Active Directory][aad].
* Información acerca de [Azure Active Directory B2C][aadb2c].

<!-- links -->
[extending-ad-to-azure]: adds-extend-domain.md

[vm-recommendations]: ../virtual-machines-windows/single-vm.md
[implementing-a-secure-hybrid-network-architecture]: ../dmz/secure-vnet-hybrid.md
[implementing-a-secure-hybrid-network-architecture-with-internet-access]: ../dmz/secure-vnet-dmz.md
[hybrid-azure-on-prem-vpn]: ../hybrid-networking/vpn.md

[azure-cli]: /azure/azure-resource-manager/xplat-cli-azure-resource-manager
[DRS]: https://technet.microsoft.com/library/dn280945.aspx
[where-to-place-an-fs-proxy]: https://technet.microsoft.com/library/dd807048.aspx
[ADDRS]: https://technet.microsoft.com/library/dn486831.aspx
[plan-your-adfs-deployment]: https://msdn.microsoft.com/library/azure/dn151324.aspx
[ad_network_recommendations]: #network_configuration_recommendations_for_AD_DS_VMs
[adfs_certificates]: https://technet.microsoft.com/library/dn781428(v=ws.11).aspx
[create_service_account_for_adfs_farm]: https://technet.microsoft.com/library/dd807078.aspx
[adfs-configuration-database]: https://technet.microsoft.com/library/ee913581(v=ws.11).aspx
[active-directory-federation-services]: https://technet.microsoft.com/windowsserver/dd448613.aspx
[security-considerations]: #security-considerations
[recommendations]: #recommendations
[active-directory-federation-services-overview]: https://technet.microsoft.com/library/hh831502(v=ws.11).aspx
[establishing-federation-trust]: https://blogs.msdn.microsoft.com/alextch/2011/06/27/establishing-federation-trust/
[Deploying_a_federation_server_farm]:  /windows-server/identity/ad-fs/deployment/deploying-a-federation-server-farm
[install_and_configure_the_web_application_proxy_server]: https://technet.microsoft.com/library/dn383662.aspx
[publish_applications_using_AD_FS_preauthentication]: https://technet.microsoft.com/library/dn383640.aspx
[managing-adfs-components]: https://technet.microsoft.com/library/cc759026.aspx
[oms-adfs-pack]: https://www.microsoft.com/download/details.aspx?id=41184
[azure-powershell-download]: https://azure.microsoft.com/documentation/articles/powershell-install-configure/
[aad]: https://azure.microsoft.com/documentation/services/active-directory/
[aadb2c]: https://azure.microsoft.com/documentation/services/active-directory-b2c/
[adfs-intro]: /azure/active-directory/hybrid/whatis-hybrid-identity
[github]: https://github.com/mspnp/reference-architectures/tree/master/identity/adfs
[adfs_certificates]: https://technet.microsoft.com/library/dn781428(v=ws.11).aspx
[considerations]: ./considerations.md
[visio-download]: https://archcenter.blob.core.windows.net/cdn/identity-architectures.vsdx
[0]: ./images/adfs.png "Protección de la arquitectura de red híbrida con Active Directory"
