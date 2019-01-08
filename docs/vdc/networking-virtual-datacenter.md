---
title: 'Centro de datos virtual de Azure: Una perspectiva de la red'
description: Aprenda a crear su centro de datos virtual en Azure
author: tracsman
manager: rossort
tags: azure-resource-manager
ms.service: virtual-network
ms.date: 11/28/2018
ms.author: jonor
ms.openlocfilehash: 1d8a9e860ab1a66104dc4133eb5f22ffb4706b84
ms.sourcegitcommit: 5a3fa0bf35376bbe4a6dd668f2d7d44f9cf9c806
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/14/2018
ms.locfileid: "53411691"
---
# <a name="azure-virtual-datacenter-a-network-perspective"></a>Centro de datos virtual de Azure: Una perspectiva de la red

## <a name="overview"></a>Introducción

La migración de aplicaciones locales a Azure proporciona a las organizaciones las ventajas de una infraestructura segura y rentable, aunque las aplicaciones se migren con unos cambios mínimos. Sin embargo, para sacar el máximo partido de la agilidad de la informática en la nube, las empresas deben evolucionar sus arquitecturas para aprovechar las funcionalidades de Azure. 

Microsoft Azure ofrece servicios y una infraestructura a gran escala, con funcionalidades de calidad empresarial y total confiabilidad. Estos servicios y esta infraestructura ofrecen muchas opciones de conectividad híbrida, así que los clientes pueden elegir acceder a ellos de forma pública a través de Internet, o bien a través de una conexión Azure ExpressRoute privada. Los asociados de Microsoft también proporcionan funcionalidades mejoradas al ofrecer servicios de seguridad y aplicaciones virtuales optimizados para su ejecución en Azure.

Los clientes pueden elegir acceder a estos servicios en la nube a través de Internet o con Azure ExpressRoute, que proporciona conectividad de red privada. Con la plataforma Microsoft Azure, los clientes pueden ampliar su infraestructura en la nube sin problemas y crear arquitecturas de varios niveles. Los asociados de Microsoft también proporcionan funcionalidades mejoradas al ofrecer servicios de seguridad y aplicaciones virtuales optimizados para su ejecución en Azure.

## <a name="what-is-the-virtual-datacenter"></a>¿Qué es el centro de datos virtual?

En sus orígenes, la nube era básicamente una plataforma para hospedar aplicaciones orientadas al público. Las empresas comenzaron a entender el valor de la nube y a mover las aplicaciones de línea de negocio internas a la nube. Dichos tipos de aplicaciones incluyeron factores adicionales de seguridad, confiabilidad, rendimiento y costo que requerían mayor flexibilidad en la forma en que se prestaban los servicios en la nube. Esto allanó el camino a una nueva infraestructura y unos nuevos servicios de red diseñados para proporcionar dicha flexibilidad, pero también a nuevas características de escalabilidad, recuperación ante desastres y otras consideraciones.

## <a name="what-is-a-virtual-datacenter"></a>¿Qué es un centro de datos virtual?
Las soluciones en la nube se diseñaron para hospedar aplicaciones únicas y relativamente aisladas en el espectro público. Este método funcionó bien unos años. Posteriormente, las ventajas de las soluciones en la nube se hicieron obvias y se hospedaban muchas cargas de trabajo a gran escala en la nube. Dar respuesta a los problemas de seguridad, confiabilidad, rendimiento y costo de las implementaciones en una o varias regiones resultó ser vital a lo largo del ciclo de vida del servicio en la nube.

El siguiente diagrama de implementación en la nube muestra un ejemplo de una brecha de seguridad en el **recuadro rojo**. El **recuadro amarillo** muestra áreas de optimización de aplicaciones virtuales de red en las cargas de trabajo.

[![0]][0]

El centro de datos virtual (VDC) es un concepto que nació de la necesidad de escalado para soportar las cargas de trabajo de la empresa, sin perder de vista la necesidad de tratar los problemas surgidos al dar soporte a aplicaciones a gran escala en la nube pública.

La implementación de un centro de datos virtual no solo representa las cargas de trabajo de las aplicaciones en la nube, sino también la red, seguridad, administración e infraestructura (por ejemplo, los servicios de directorio y el DNS). Dado que cada vez se mueven más cargas de trabajo de las empresas a Azure, es importante pensar en la infraestructura de soporte y en los objetos en los que se colocan dichas cargas. Pensar detenidamente sobre cómo se estructuran los recursos puede evitar la proliferación de cientos de "islas de cargas de trabajo" que deben administrarse por separado con flujo de datos y modelos de seguridad independientes y desafíos de cumplimiento de normas.

El concepto de centro de datos virtual es un conjunto de recomendaciones y procedimientos recomendados para implementar una colección de entidades independientes, pero relacionadas, en características, funciones auxiliare y una infraestructura comunes. Si mira las cargas de trabajo a través de la lente del centro de datos virtual, es posible que se dé cuenta de la reducción de costos que aportan las economías de escala y de la optimización de la seguridad que se logra con la centralización de los componentes y flujos de datos, además de la facilidad para realizar las distintas operaciones, la administración y las auditorías.

> [!NOTE]
> Es importante saber que el centro de datos virtual **NO** es un producto independiente de Azure, sino una combinación de varias características y funcionalidades para satisfacer sus necesidades exactas. Un centro de datos virtual es una manera de pensar en sus cargas de trabajo y el uso de Azure para maximizar los recursos y funcionalidades en la nube. Es un enfoque modular para la creación de los servicios de TI en Azure, al mismo tiempo que se respetan los roles organizativos y las responsabilidades de la empresa.

La implementación de un centro de datos virtual puede ayudar a que las empresas lleven cargas de trabajo y aplicaciones a Azure en los siguientes escenarios:

-   Hospedaje de varias cargas de trabajo relacionadas.
-   Migración de cargas de trabajo desde un entorno local a Azure.
-   Implementación de requisitos de seguridad y acceso compartido o centralizado para las cargas de trabajo.
-   Combinación de Azure DevOps y TI centralizada adecuada para una gran empresa.

La clave para desbloquear las ventajas del centro de datos virtual es una topología en estrella tipo hub-and-spoke centralizada, con una mezcla de características de Azure: 

- [Azure Virtual Network][VNet]. 
- [Grupos de seguridad de red (NSG)][NSG].
- [Emparejamiento de redes virtuales][VNetPeering]. 
- [Rutas definidas por el usuario (UDR)][UDR].
- Servicios de identidad de Azure con el [control de acceso basado en rol (RBAC)][RBAC]. 
- Opcionalmente, [Azure Firewall][AzFW], [Azure DNS][DNS], [Azure Front Door][AFD] y [Azure Virtual WAN][vWAN].

La clave para desbloquear las ventajas del centro de datos virtual es una topología en estrella del tipo hub-and-spoke centralizada, con una mezcla de servicios y características de Azure:

* [Azure Virtual Network][VNet],
* [Grupos de seguridad de red][NSG],
* [Emparejamiento de redes virtuales][VNetPeering],
* [Rutas definidas por el usuario (UDR)][UDR] y
* Identidad de Azure con [Control de acceso basado en roles (RBAC)] [ RBAC] y, opcionalmente, [Azure Firewall][AzFW], [Azure DNS][DNS], [Azure Front Door][AFD] y [Azure Virtual WAN][vWAN].

## <a name="who-should-implement-a-virtual-datacenter"></a>¿Quién debe implementar un centro de datos virtual?

Todos los clientes de Azure que hayan decidido usar la nube pueden beneficiarse de la eficacia de la configuración de un conjunto de recursos que pueden usar todas las aplicaciones. Dependiendo de la magnitud, hasta las aplicaciones individuales pueden beneficiarse del uso de los patrones y componentes que se utilizan para crear una implementación de un centro de datos virtual.

Si su organización tiene TI, una red y seguridad centralizadas o si tiene un equipo o departamento de cumplimiento, la implementación de un centro de datos virtual puede ayudar a aplicar los puntos de las directivas, la segregación del servicio y garantizar la uniformidad de los componentes comunes subyacentes, al tiempo que da a los equipos de aplicación la libertad y control apropiados para sus requisitos.

Las organizaciones que recurren a DevOps también pueden usar los conceptos del centro de datos virtual para proporcionar paquetes autorizados de recursos de Azure y garantizar el control total dentro de ese grupo (una suscripción o un grupo de recursos de una suscripción común), manteniendo el cumplimiento de los límites de red y seguridad definidos por una directiva centralizada en una red virtual de un centro de conectividad y un grupo de recursos.

## <a name="considerations-for-implementing-a-virtual-datacenter"></a>Consideraciones para la implementación de un centro de datos virtual

Al diseñar una implementación de un centro de datos virtual, hay varios problemas esenciales que se deben tener en cuenta:

### <a name="identity-and-directory-services"></a>Servicios de identidad y directorio
Los servicios de identidad y directorio son un aspecto clave de todos los centros de datos, tanto en local como en la nube. La identidad está relacionada con todos los aspectos de acceso y autorización a los servicios dentro del centro de datos virtual. Para garantizar que solo los usuarios y procesos autorizados tienen acceso a la cuenta de Azure y los recursos, Azure usa varios tipos de credenciales para la autenticación. Estos incluyen contraseñas (para acceder a la cuenta de Azure), claves de cifrado, firmas digitales y certificados. 

### <a name="identity-and-directory-service"></a>Servicios de identidad y directorio

Los servicios de identidad y directorio son un aspecto clave de todos los centros de datos, tanto locales como en la nube. La identidad está relacionada con todos los aspectos del acceso y autorización a los servicios de una implementación del centro de datos virtual. Para ayudar a garantizar que solo los usuarios y procesos autorizados tienen acceso a la cuenta de Azure y los recursos, Azure usa varios tipos de credenciales para la autenticación. Estos incluyen contraseñas (para obtener acceso a la cuenta de Azure), claves de cifrado, firmas digitales y certificados. [Azure Multi-Factor Authentication (MFA)][MFA] es una capa adicional de seguridad para acceder a los servicios de Azure. Azure MFA proporciona una autenticación sólida gracias a su variedad de opciones de verificación sencillas (notificación mediante llamada telefónica, mensaje de texto o aplicación para dispositivos móviles) y permite a los clientes elegir el método que prefieran.

Todas las empresas grandes necesitan definir un proceso de administración de identidad que describa la administración de las identidades individuales, su autenticación, autorización, roles y privilegios dentro de la implementación de su centro de datos virtual. Los objetivos de este proceso deben ser mejorar la seguridad y productividad al reducir el costo, el tiempo de inactividad y las tareas manuales repetitivas.

La empresas y organizaciones pueden requerir una combinación exigente de servicios para diferentes líneas de negocio (LOB) y los empleados a menudo tienen distintos roles cuando están implicados en proyectos diferentes. Un centro de datos virtual requiere buena cooperación entre distintos equipos, cada uno con definiciones de rol específicas, para tener una buena gestión de los sistemas en ejecución. La matriz de responsabilidades, acceso y derechos puede ser compleja. En el centro de datos virtual, la administración de identidad se implementa a través de [*Azure Active Directory* (Azure AD)][AAD] y el control de acceso basado en rol (RBAC).

Un servicio de directorio es una infraestructura de información compartida para buscar, gestionar, administrar y organizar los elementos cotidianos y los recursos de red. Estos recursos pueden incluir volúmenes, carpetas, archivos, impresoras, usuarios, grupos, dispositivos y otros objetos. El servidor de directorio considera cada recurso de la red como un objeto. La información sobre un recurso se almacena como una colección de atributos asociada a ese recurso u objeto.

Todos los servicios de negocio en línea de Microsoft dependen de Azure Active Directory (Azure AD) para el inicio de sesión y otras necesidades de identidad. Azure Active Directory es solución en la nube de administración de acceso e identidades completa y de alta disponibilidad que combina una administración de acceso a aplicaciones, gobernancia de identidades avanzada y servicios de directorio fundamentales. Azure AD se puede integrar con Active Directory local a fin de habilitar el inicio de sesión único para todas las aplicaciones basadas en la nube y hospedadas localmente. Los atributos de usuario de Active Directory local se pueden sincronizar automáticamente con Azure AD.

En una implementación de un centro de datos virtual no se requiere un administrador global individual para asignar todos los permisos. En su lugar, cada departamento o grupo de usuarios o servicios específico del servicio de directorio puede tener los permisos necesarios para administrar sus propios recursos dentro de una implementación del centro de datos virtual. La estructura de permisos requiere un equilibrio. Demasiados permisos pueden disminuir el rendimiento y muy pocos permisos o permisos flexibles pueden aumentar los riesgos de seguridad. El control de acceso basado en rol (RBAC) de Azure le ayuda a abordar este problema, ya que ofrece una administración avanzada del acceso a los recursos de una implementación de un centro de datos virtual.

#### <a name="security-infrastructure"></a>Infraestructura de seguridad

Por infraestructura de seguridad se entiende la segregación del tráfico del segmento de red virtual específico de una implementación de un centro de datos virtual. Esta infraestructura especifica cómo se controlan la entrada y la salida en una implementación de un centro de datos virtual. Azure se basa en una arquitectura multiinquilino que impide que el tráfico no autorizado e involuntario entre implementaciones mediante el uso del aislamiento de la red virtual (VNet), las listas de control de acceso (ACL), los equilibradores de carga, los filtros de direcciones IP y las directivas de flujo de tráfico. La traducción de direcciones de red (NAT) se usa para separar el tráfico de red interno del tráfico externo.

El tejido de Azure asigna recursos de infraestructura para las cargas de trabajo del inquilino y administra las comunicaciones a y desde las máquinas virtuales (VM). El hipervisor de Azure fuerza la separación de la memoria y el proceso entre las máquinas virtuales y enruta de forma segura el tráfico de red a los SO invitado de cada inquilino.

#### <a name="connectivity-to-the-cloud"></a>Conectividad a la nube

Una implementación de un centro de datos virtual necesita conectividad con redes externas para ofrecer servicios a los clientes, asociados o usuarios internos. Esta necesidad de conectividad no solo sucede en Internet, sino en las redes y centros de datos locales.

Los clientes controlan qué servicios pueden acceder al Internet público y a cuáles se puede acceder desde este mediante [Azure Firewall][AzFW] o cualquier otro tipo de aplicaciones de red virtual (NVA), directivas de enrutamiento personalizadas mediante [rutas definidas por el usuario][UDR] y filtrado de redes mediante [grupos de seguridad de red][NSG]. Se recomienda que todos los recursos accesibles desde Internet estén también protegidos por [Azure DDOS Protection estándar][DDOS].

Las empresas a menudo necesitan conectar su implementación de un centro de datos virtual centros de datos u otros recursos locales. Esta conectividad entre las redes de Azure y las locales es un aspecto fundamental del diseño de una arquitectura eficaz. Las empresas tienen dos formas distintas de crear esta interconexión: pasar por Internet o mediante conexiones directas privadas.

Una [**red privada virtual de sitio a sitio de Azure**][VPN] es un servicio de interconexión a través de Internet entre redes locales y una implementación de un centro de datos virtual, que se establece a través de conexiones cifradas seguras (túneles IPsec o IKE). La conexión sitio a sitio de Azure es flexible, rápida de crear y no requiere ninguna adquisición adicional, ya que todas las conexiones se realizan a través de Internet.

Para muchas conexiones VPN, [**Azure Virtual WAN**][vWAN] es un servicio de red que ofrece conectividad automatizada y optimizada entre ramas mediante Azure. Azure Virtual WAN permite conectar y configurar los dispositivos de rama para comunicarse con Azure. La conexión y configuración se pueden realizar manualmente o mediante los dispositivos de proveedores preferidos mediante un asociado de Virtual WAN. El uso de dispositivos de proveedores preferidos permite la administración de la configuración, facilita el uso y simplifica la conectividad. El panel integrado de Azure WAN proporciona información instantánea para solucionar problemas que puede ayudarle a ahorrar tiempo y le ofrece una manera fácil de ver la conectividad de sitio a sitio y a gran escala.

[**ExpressRoute**][ExR] es un servicio de conectividad de Azure que permite conexiones privadas entre una implementación de un centro de datos virtual y todas las redes locales. Las conexiones ExpressRoute no se realizan sobre una conexión a Internet pública, ofrecen una mayor confiabilidad, seguridad y velocidades mayores (hasta 10 Gbps) con una latencia menor. ExpressRoute es muy útil para las implementaciones de centros de datos virtuales, ya que los clientes de ExpressRoute se pueden beneficiar de las reglas de cumplimiento de normas asociadas con las conexiones privadas. Con ExpressRoute Direct,][ExRD] puede conectarse directamente a los enrutadores de Microsoft a 100 Gbps, para los clientes con mayores necesidades de ancho de banda.

La implementación de conexiones ExpressRoute implica normalmente tomar contacto con un proveedor de servicios de ExpressRoute. Para los clientes que necesitan comenzar rápidamente, lo habitual es usar inicialmente una VPN de sitio a sitio para establecer la conectividad entre una implementación de un centro de datos virtual y los recursos locales y, posteriormente, migrar a una conexión ExpressRoute cuando la interconexión física con el proveedor de servicios se haya completado.

#### <a name="connectivity-within-the-cloud"></a>*Conectividad dentro de la nube*

[Redes virtuales][VNet] y [Emparejamiento de redes virtuales][VNetPeering] son los servicios de conectividad de red básicos de una implementación de un centro de datos virtual. Una red virtual garantiza un límite de aislamiento natural para los recursos del centro de datos virtual y el emparejamiento de redes virtuales permite la intercomunicación entre distintas redes virtuales dentro de la misma región de Azure o incluso de distintas regiones. El control de tráfico dentro de una red virtual y entre redes virtuales debe cumplir con un conjunto de reglas de seguridad especificado a través de listas de control de acceso ([Grupo de seguridad de red][NSG]), [aplicaciones virtuales de red][NVA] y tablas de enrutamiento personalizadas ([UDR][UDR]).

## <a name="virtual-datacenter-overview"></a>Introducción al centro de datos virtual

### <a name="topology"></a>Topología

_Hub and spoke_ es un modelo para diseñar la topología de red de una implementación de un centro de datos virtual. 

[![1]][1]

Un centro (hub) es la zona central que controla e inspecciona el tráfico de entrada y salida entre las diferentes zonas: Internet, local y los radios (spokes). La topología en estrella tipo hub-and-spoke ofrece al departamento de TI una manera eficaz de aplicar directivas de seguridad en una ubicación central. También reduce la posibilidad de exposición y de configuración incorrecta.

El concentrador contiene los componentes de los servicios comunes utilizados por los radios. Los ejemplos siguientes son servicios centrales comunes:

-   La infraestructura de Active Directory de Windows necesaria para la autenticación de usuarios de terceros que acceden desde redes que no son de confianza antes de obtener acceso a las cargas de trabajo del radio (spoke). Incluye los Servicios de federación de Active Directory (AD FS) relacionados.
-   Un servicio DNS que resuelve los nombres de la carga de trabajo en los radios (spokes) para acceder a recursos locales y en Internet si no se utiliza [Azure DNS][DNS].
-   Una infraestructura de clave pública (PKI), para implementar el inicio de sesión único en las cargas de trabajo.
-   Control de flujo de tráfico TCP y UDP entre las zonas de red de los radios (spokes) e Internet.
-   Control de flujo entre los radios (spokes) y local.
-   Si es necesario, control de flujo entre un radio y otro.

El centro de datos virtual reduce el costo total al utilizar la infraestructura de concentrador compartida entre varios radios.

El rol de cada radio puede ser el hospedaje de diferentes tipos de cargas de trabajo. Los radios (spokes) también proporcionan un enfoque modular para implementaciones repetibles de las mismas cargas de trabajo. Algunos ejemplos son el desarrollo y las pruebas, las pruebas de aceptación de usuario, la preproducción y la producción. Los radios también pueden segregar y habilitar varios grupos dentro de su organización. Por ejemplo, los grupos de Azure DevOps. Dentro de un radio, es posible implementar una carga de trabajo básica o cargas de trabajo de varios niveles complejas con control de tráfico entre los niveles.

#### <a name="subscription-limits-and-multiple-hubs"></a>Límites de la suscripción y múltiples concentradores

En Azure, todos los componentes, de cualquier tipo, se implementan en una suscripción de Azure. El aislamiento de componentes de Azure en distintas suscripciones de Azure puede satisfacer los requisitos de diferentes líneas de negocio, como la configuración de niveles diferenciados de acceso y autorización.

Una implementación individual de un centro de datos virtual puede escalar verticalmente a gran número de radios, aunque, al igual que en todos los sistemas de TI, existen límites en las plataformas. La implementación de un concentrador se enlaza a una suscripción de Azure específica, que tiene restricciones y límites (por ejemplo, un número máximo de emparejamientos de redes virtuales: consulte [Suscripción de Azure y límites de servicio, cuotas y restricciones][Limits] para más información). En los casos en los que los límites puedan ser un problema, la arquitectura se puede escalar verticalmente más allá extiendo el modelo desde un único concentrador-radios a un clúster de concentradores y radios. Varios centros en una o más regiones de Azure pueden estar conectados entre sí mediante el emparejamiento de VNET, ExpressRoute, Virtual WAN o una red privada virtual de sitio a sitio.

[![2]][2]

La introducción de varios centros aumenta el costo y el esfuerzo de administración del sistema. Solo se podría justificar por motivos de escalabilidad, como en el caso de los límites del sistema, y de replicación regional, como en el caso de rendimiento del usuario final o recuperación ante desastres. En escenarios que requieran múltiples concentradores, todos los concentradores deben procurar ofrecer el mismo conjunto de servicios para facilitar la operativa.

#### <a name="interconnection-between-spokes"></a>Interconexión entre los radios

Dentro de un radio único, es posible implementar cargas de trabajo complejas en múltiples niveles. Las configuraciones de múltiples niveles se pueden implementar utilizando subredes (una para cada nivel) en la misma red virtual y filtrando los flujos con grupos de seguridad de red.

Un arquitecto podría querer implementar una carga de trabajo de varios niveles entre varias redes virtuales. Con el emparejamiento de redes virtuales, los radios pueden conectarse a otros radios en el mismo centro o en centros diferentes. Un ejemplo típico de este escenario es el caso en el que los servidores de procesamiento de la aplicación están en un radio o red virtual. La base de datos se implementa en un radio o red virtual diferente. En este caso, es fácil interconectar los radios con emparejamiento de redes virtuales y así evitar el tránsito a través del centro. Debe realizarse una revisión cuidadosa de la arquitectura y la seguridad para asegurarse de que evitar el paso por el concentrador no omite puntos de seguridad o auditoría de importancia que podrían existir solo en el centro.

[![3]][3]

Los radios también pueden estar conectados entre sí mediante un radio que actúa como concentrador. Este método crea una jerarquía de dos niveles: el radio del nivel superior (nivel 0) se convierten en el concentrador de radios inferiores (nivel 1) en la jerarquía. Los radios de una implementación de un centro de datos virtual están obligados a reenviar el tráfico al centro principal, con el fin de que pueda llegar a su destino pasando por la red local o Internet. Una arquitectura con dos niveles de concentradores presenta un enrutamiento complejo que anula las ventajas de una relación simple concentrador-radio.

Aunque Azure permite topologías complejas, uno de los principios básicos del concepto de centro de datos virtual es la repetibilidad y simplicidad. Para minimizar el esfuerzo de administración, el diseño simple en estrella tipo hub-and-spoke es la arquitectura de referencia recomendada para un centro de datos virtual.

### <a name="components"></a>Componentes

El centro de datos virtual se compone de cuatro tipos de componentes básicos: **Infraestructura**, **redes perimetrales**, **cargas de trabajo** y **supervisión**.

Cada tipo de componente consta de varias características de Azure y recursos. La implementación del centro de datos virtual se compone de instancias de varios tipos de componentes y múltiples variaciones del mismo tipo de componente. Por ejemplo, puede tener muchas instancias de cargas de trabajo diferentes, separadas lógicamente, que representan las distintas aplicaciones. Estos distintos tipos de componentes e instancias se utilizan para crear el centro de datos virtual.

[![4]][4]

La arquitectura conceptual de alto nivel anterior del centro de datos virtual muestra distintos tipos de componentes utilizados en distintas zonas de la topología en estrella tipo hub-and-spoke. El diagrama muestra los componentes de la infraestructura en distintas partes de la arquitectura.

Como procedimiento recomendado en general, los derechos de acceso y privilegios deben estar basados en grupos. Trabajar con grupos, en lugar de con usuarios individuales, facilita el mantenimiento de directivas de acceso, ya que proporciona una manera coherente de administrarlo a en varios equipos  y ayuda a minimizar los errores de configuración. Asignar y eliminar usuarios de los grupos adecuados le ayuda a mantener actualizados los privilegios de un usuario específico.

Cada grupo de roles debe tener un prefijo único en sus nombres. Este prefijo facilita una rápida identificación de qué grupo está asociado con qué carga de trabajo. Por ejemplo, una carga de trabajo que hospeda un servicio de autenticación podría tener grupos denominados **AuthServiceNetOps**, **AuthServiceSecOps**, **AuthServiceDevOps** y **AuthServiceInfraOps**. Los roles centralizados o los roles no relacionados con un servicio concreto, podrían estar precedidos de **Corp**. Por ejemplo: **CorpNetOps**.

Muchas organizaciones usan una variación de los grupos siguientes para proporcionar un desglose mayor de los roles:

-   El grupo de TI central, **Corp,** tiene los derechos de propiedad para controlar los componentes de la infraestructura. Ejemplos de redes y seguridad. El grupo debe tener el rol de colaborador en la suscripción, el control del centro y derechos de colaborador de la red en los radios. Las grandes organizaciones suelen dividir estas responsabilidades de administración entre varios equipos. Ejemplos: un grupo **CorpNetOps** de operaciones de red con atención exclusiva a las redes, y un grupo **CorpSecOps** de operaciones de seguridad responsable del firewall y la directiva de seguridad. En este caso específico, deben crearse dos grupos diferentes para la asignación de estos roles personalizados.
-   El grupo de desarrollo y pruebas **AppDevOps** tiene la responsabilidad de implementar las cargas de trabajo de aplicaciones o servicios. Este grupo tiene el rol de colaborador de la máquina Virtual para las implementaciones de IaaS y uno o más roles de colaborador de PaaS. Consulte [Roles integrados en los recursos de Azure][Roles]. Opcionalmente, el equipo de desarrollo y pruebas podría tener visibilidad en las directivas de seguridad (NSG) y en las directivas de enrutamiento (UDR) en el centro o en un radio específico. Además de los roles de colaborador para las cargas de trabajo, este grupo también necesitaría el rol de lector de red.
-   El grupo de operaciones y mantenimiento **CorpInfraOps** o **AppInfraOps** tiene la responsabilidad de administrar las cargas de trabajo en producción. Este grupo debe ser un colaborador de la suscripción en las cargas de trabajo en las suscripciones de producción. Algunas organizaciones también pueden evaluar si necesitan un grupo de equipo de soporte técnico con el rol de colaborador en la suscripción de producción y en la suscripción del centro principal. El grupo adicional permite solucionar posibles problemas de configuración en el entorno de producción.

El centro de datos virtual está diseñado de forma que los grupos de TI centrales creados para la administración del centro tengan grupos correspondientes en el nivel de carga de trabajo. Además de la administración de los recursos del centro de conectividad, el grupo de TI central puede controlar el acceso externo y los permisos de nivel superior en la suscripción. Los grupos de cargas de trabajo también pueden controlar los recursos y permisos de su red virtual con independencia de TI central.

Las particiones del centro de datos virtual sirven para alojar de forma segura varios proyectos en líneas de negocios (LOB) diferentes. Todos los proyectos requieren diferentes entornos aislados (desarrollo, UAT, producción). El uso de suscripciones de Azure independientes para cada uno de estos entornos proporciona un aislamiento natural.

[![5]][5]

El diagrama anterior muestra la relación entre los proyectos de una organización, los usuarios, grupos y los entornos donde se implementan los componentes de Azure.

Normalmente, en TI, un entorno (o nivel) es un sistema en el que se implementan y ejecutan varias aplicaciones. Las empresas de tamaño grande usan un entorno de desarrollo (en el que se realizan originalmente los cambios y pruebas) y un entorno de producción (el que los usuarios finales utilizan). Dichos entornos están separados y a menudo hay varios entornos de ensayo entre ellos para permitir la implementación por fases (lanzamiento), la realización de pruebas y la reversión si surgen problemas. Las arquitecturas de implementación varían considerablemente, aunque normalmente siguen el proceso básico de comenzar en un desarrollo (DEV) y terminar en producción (PROD).

Una arquitectura común para estos tipos de entornos de varios niveles se compone de Azure DevOps para desarrollo y pruebas, UAT para ensayo y entornos de producción. Las organizaciones pueden aprovechar uno o varios inquilinos de Azure AD para definir el acceso y los derechos para estos entornos. El diagrama anterior muestra un caso en el que se usan dos inquilinos diferentes de Azure AD: uno para Azure DevOps y UAT y otro exclusivamente para producción.

La presencia de inquilinos de Azure AD diferentes refuerza la separación entre entornos. El mismo grupo de usuarios, por ejemplo, TI central, debe autenticarse con un identificador URI diferente para acceder a un inquilino de Azure AD diferente y modificar los roles o permisos de los entornos de Azure DevOps o de producción de un proyecto. La presencia de una autenticación de usuario diferente para acceder a diferentes entornos reduce posibles interrupciones y otros problemas causados por errores humanos.

#### <a name="component-type-infrastructure"></a>Tipo de componente: Infraestructura

Este tipo de componente es en el que reside la mayor parte de la infraestructura de soporte. También es donde los equipos de TI central, seguridad y cumplimiento dedican la mayor parte de su tiempo.

[![6]][6]

Los componentes de la infraestructura proporcionan una interconexión para los distintos componentes de una implementación de un centro de datos virtual y están presentes tanto en el centro de conectividad como en los radios. Normalmente se asigna la responsabilidad de administrar y mantener los componentes de infraestructura al equipo de TI central o al de seguridad.

Una de las tareas principales del equipo de infraestructura de TI es garantizar la coherencia de los esquemas de direcciones IP en toda la empresa. El espacio de direcciones IP privadas asignado a una implementación de un centro de datos virtual debe ser coherente y NO superponerse con direcciones IP privadas asignadas a redes locales.

Aunque el uso de NAT en los enrutadores locales perimetrales o en entornos de Azure puede evitar conflictos de direcciones IP, agrega complicaciones a los componentes de infraestructura. La simplicidad de la administración es uno de los principales objetivos del centro de datos virtual, por lo que el uso de NAT para controlar cuestiones de direcciones IP no es una solución recomendada.

Los componentes de infraestructura tienen la siguiente funcionalidad:

-   [**Servicios de identidad y directorio**][AAD]. El acceso a cada tipo de recurso en Azure se controla mediante una identidad que se almacena en un servicio de directorio. El servicio de directorio almacena no solo la lista de usuarios, sino también los derechos de acceso a los recursos en una suscripción de Azure específica. Estos servicios pueden existir solo en la nube, o se pueden sincronizar con la identidad local almacenada en Active Directory.
-   [**Virtual Network**][VPN]. Las redes virtuales son uno de los componentes principales del centro de datos virtual y le permiten crear un límite de aislamiento de tráfico en la plataforma de Azure. Una red virtual se compone de uno o varios segmentos de red virtual, cada uno con un prefijo de red IP específico (una subred). La red virtual define un área de perímetro interno en el que las máquinas virtuales de IaaS y los servicios de PaaS pueden establecer comunicaciones privadas. Las máquinas virtuales (y los servicios de PaaS) de una red virtual no pueden comunicar directamente con máquinas virtuales (y servicios de PaaS) de una red virtual diferente, incluso si ambas redes virtuales se han creado en la misma suscripción y por el mismo cliente. El aislamiento consiste en una propiedad fundamental que garantiza que las máquinas virtuales del cliente y la comunicación sigan siendo privadas en una red virtual.
-   [**UDR**][UDR]. El tráfico en una red virtual se enruta de forma predeterminada en función de la tabla de enrutamiento del sistema. Una ruta definida por el usuario es una tabla de enrutamiento personalizada que los administradores de red pueden asociar a una o varias subredes para sobrescribir el comportamiento de la tabla de enrutamiento del sistema y definir una ruta de comunicación en una red virtual. La presencia de una UDR garantiza que el tráfico de salida del radio transita a través de máquinas virtuales personalizadas específicas o aplicaciones virtuales de red y equilibradores de carga presentes en el concentrador y los radios.
-   [**NSG**][NSG]. Un grupo de seguridad de red (NSG) es una lista de reglas de seguridad que actúa como filtro de tráfico en direcciones IP de origen y destino, protocolos y puertos IP de origen y destino. El NSG se puede aplicar a una subred, una tarjeta NIC virtual asociada a una máquina virtual de Azure, o ambos. Los NSG son fundamentales para implementar un control de flujo correcto en el concentrador y los radios. El nivel de seguridad permitido por el NSG está una función de los puertos que abra y su finalidad. Los clientes deben aplicar filtros adicionales en cada máquina virtual con firewalls basados en host, como IPtables o el Firewall de Windows.
-   [**DNS**][DNS]. La resolución de nombres de recursos en las redes virtuales de una implementación de un centro de datos virtual se proporciona mediante DNS. Azure proporciona servicios DNS tanto para la resolución de nombres [públicos][DNS] como [privados][PrivateDNS]. Las zonas privadas proporcionan la resolución de nombres dentro de una red virtual o en redes virtuales distintas. Puede hacer que las zonas privadas abarquen redes virtuales de la misma región, e incluso de distintas regiones y suscripciones. Azure DNS proporciona un servicio de hospedaje de dominios DNS que permite resolver nombres mediante la infraestructura de Microsoft Azure (resolución pública). Al hospedar dominios en Azure, puede administrar los registros DNS con las mismas credenciales, API, herramientas y facturación que con los demás servicios de Azure.
-   [**Administración de la suscripción**][SubMgmt] y del [**Grupo de recursos**][RGMgmt]. Una suscripción define un límite natural para crear varios grupos de recursos en Azure. Los recursos de una suscripción se ensamblan juntos en contenedores lógicos llamados grupos de recursos. El grupo de recursos representa un grupo lógico para organizar los recursos de una implementación de un centro de datos virtual.
-   [**RBAC**][RBAC]. Mediante RBAC, es posible asignar roles organizativos junto con los derechos de acceso a recursos específicos de Azure, lo que le permite restringir a los usuarios a solo un subconjunto determinado de acciones. Con RBAC puede conceder acceso asignando el rol adecuado a usuarios, grupos y aplicaciones dentro del ámbito correspondiente. El ámbito de una asignación de roles puede ser una suscripción de Azure, un grupo de recursos o un único recurso. RBAC permite la herencia de permisos. Un rol asignado en un ámbito principal también concede acceso a los elementos secundarios dentro del mismo. Con RBAC podrá repartir las tareas y conceder a los usuarios únicamente el nivel de acceso que necesitan para realizar su trabajo. Por ejemplo, utilice RBAC para dejar que un empleado administre máquinas virtuales en una suscripción y que otro pueda administrar bases de datos SQL en la misma suscripción.
-   [**Emparejamiento de redes virtuales**][VNetPeering]. La característica fundamental que se usa para crear la infraestructura del centro de datos virtual es Emparejamiento de redes virtuales, un mecanismo que conecta dos redes virtuales (VNet) de la misma región mediante la red del centro de datos de Azure o la red troncal mundial de Azure para regiones distintas.

#### <a name="component-type-perimeter-networks"></a>Tipo de componente: Redes perimetrales

Los componentes de la [red perimetral][DMZ] habilitan la conectividad de red entre su redes locales o con la del centro de datos físico, así como todo tipo de conectividad hacia Internet y desde este. También es donde probablemente los equipos de red y seguridad empleen la mayor parte de su tiempo.

Los paquetes entrantes deben fluir a través de las aplicaciones de seguridad del centro antes de llegar a los servidores back-end en los radios. Algunos ejemplos: el firewall, IDS e IPS. Antes de abandonar la red, los paquetes enlazados a Internet desde las cargas de trabajo también deberían fluir a través de las aplicaciones de seguridad de la red perimetral. Este flujo tiene fines de auditoría, inspección y aplicación de directivas.

Los componentes de la red perimetral proporcionan las siguientes características:

-   [Redes virtuales][VNet], [UDR][UDR] y [NSG][NSG]
-   [Aplicaciones virtuales de red][NVA]
-   [Azure Load Balancer][ALB]
-   [Azure Application Gateway][AppGW] y [Firewall de aplicaciones web (WAF)][WAF]
-   [Direcciones IP públicas][PIP]
-   [Azure Front Door][AFD]
-   [Azure Firewall][AzFW]

Normalmente, los equipos de TI central y de seguridad tienen la responsabilidad de la definición de requisitos y el funcionamiento de las redes perimetrales.

[![7]][7]

El diagrama anterior muestra el cumplimiento de dos perímetros con acceso a Internet y a una red local, ambos residentes en los centros DMZ y WAN virtual. En el centro DMZ, la red perimetral a Internet puede escalar verticalmente para admitir grandes cantidades de líneas de negocio mediante varias granjas de firewalls de aplicaciones web (WAF) y/o instancias de Azure Firewall. En el centro de WAN virtual, la conectividad altamente escalable entre ramas y de rama a Azure se logra a través de VPN o ExpressRoute cuando sea necesario.

[**Virtual Networks**][VNet]. El centro normalmente se basa en una red virtual con varias subredes para hospedar los distintos tipos de servicios con filtrado e inspección del tráfico hacia o desde Internet mediante NVA, WAF e instancias de Azure Application Gateway.

[**UDR**][UDR] Mediante las UDR, los clientes pueden implementar firewalls, IDS/IPS y otras aplicaciones virtuales, y enrutar el tráfico de red a través de estas aplicaciones de seguridad para la aplicación de directivas de límites de seguridad, auditoría e inspección. Las UDR pueden crearse en el concentrador y los radios para garantizar que el tráfico transita a través de máquinas virtuales personalizadas específicas, aplicaciones virtuales de red y equilibradores de carga utilizados en una implementación de un centro de datos virtual. Para garantizar que el tráfico generado desde máquinas virtuales residentes en el radio transita a las aplicaciones virtuales correctas, se debe establecer una UDR en las subredes del radio configurando la dirección IP del front-end del equilibrador de carga interno como el próximo salto. El equilibrador de carga interno distribuye el tráfico interno a las aplicaciones virtuales (grupo de back-end de equilibradores de carga).

[**Azure Firewall**][AzFW] es un servicio de seguridad de red administrado y basado en la nube que protege los recursos de Azure Virtual Network. Se trata de un firewall como servicio con estado completo que incorpora alta disponibilidad y escalabilidad a la nube sin restricciones. Puede crear, aplicar y registrar directivas de aplicaciones y de conectividad de red a nivel central en suscripciones y redes virtuales. Azure Firewall usa una dirección IP pública estática para los recursos de red virtual. Esto permite que los firewalls externos identifiquen el tráfico que procede de su red virtual. El servicio está totalmente integrado con Azure Monitor para los registros y análisis.

[**Aplicaciones virtuales de red**][NVA]. En el centro, la red perimetral con acceso a Internet se administra normalmente a través de una instancia de Azure Firewall o una granja de firewalls o mediante un firewall de aplicaciones web (WAF).

Las diferentes líneas de negocio normalmente utilizan muchas aplicaciones web. Estas aplicaciones tienden a sufrir varias vulnerabilidades y posibles puntos débiles. Los firewall de aplicaciones web son un tipo especial de producto que se usa para detectar ataques contra las aplicaciones web (HTTP/HTTPS) con mayor profundidad que un firewall genérico. En comparación con la tecnología de firewall tradicional, los WAF tienen un conjunto de características específicas para proteger los servidores web internos frente a amenazas.

Una instancia de Azure Firewall o un firewall de aplicación virtual de red utilizan un plano de administración común, con un conjunto de reglas de seguridad para proteger las cargas de trabajo hospedadas en los radios y controlar el acceso a las redes locales. Azure Firewall tiene escalabilidad integrada, mientras que los firewalls de aplicación virtual de red se pueden escalar manualmente detrás de un equilibrador de carga. En general, una granja de firewalls tiene menos software especializado en comparación con un WAF, pero tiene un ámbito más amplio de aplicación para filtrar e inspeccionar cualquier tipo de tráfico de entrada y salida. Si se usa un enfoque de aplicación virtual de red, puede encontrarlos e implementarlos desde Azure Marketplace.

Use un conjunto de instancias de Azure Firewall (o aplicaciones de red virtual) para el tráfico que se origina en Internet y otro para el que se origina en el entorno local. Utilizar únicamente un conjunto de firewalls para ambos es un riesgo de seguridad, ya que no proporciona ningún perímetro de seguridad entre los dos conjuntos de tráfico de red. El uso de capas de firewalls independientes reduce la complejidad de la comprobación de las reglas de seguridad y deja claro qué reglas se corresponden con cada solicitud de red entrante.

Es recomendable que use un conjunto de instancias de Azure Firewall o de aplicaciones virtuales de red para el tráfico que se origina en Internet. Use otro para el tráfico que se origina en el entorno local. Utilizar únicamente un conjunto de firewalls para ambos es un riesgo de seguridad, ya que no proporciona ningún perímetro de seguridad entre los dos conjuntos de tráfico de red. El uso de capas de firewalls independientes reduce la complejidad de la comprobación de las reglas de seguridad y deja claro qué reglas se corresponden con cada solicitud de red entrante.

[**Azure Load Balancer**][ALB] ofrece un servicio de nivel 4 (TCP y UDP) de alta disponibilidad que puede distribuir el tráfico entrante entre instancias del servicio definidas en un conjunto de equilibrio de carga. El tráfico enviado al equilibrador de carga desde los puntos de conexión front-end (puntos de conexión de direcciones IP públicas o privadas) se puede redistribuir con o sin traducción de direcciones a un conjunto de grupo de direcciones IP de back-end (ejemplo: aplicaciones virtuales de red o máquinas virtuales).

Azure Load Balancer puede sondear el estado de las diversas instancias de servidor y, cuando un sondeo no responde, el equilibrador de carga deja de enviar tráfico a la instancia incorrecta. En el centro de datos virtual se implementa un equilibrador de carga externo en el centro y en los radios. En el centro, el equilibrador de carga se usa para enrutar el tráfico de forma eficaz a los servicios en los radios, mientras que en los radios, los equilibradores de carga se utilizan para administrar el tráfico de la aplicación.

[**Azure Front Door**][AFD] (AFD) es la plataforma de aceleración de aplicaciones web altamente disponible y escalable de Microsoft, un equilibrador de carga HTTP global con capacidades de protección de aplicaciones y red de entrega de contenido. Al ejecutarse en más de 100 ubicaciones en el perímetro de la red global de Microsoft, AFD le permite crear, operar y escalar horizontalmente una aplicación web dinámica y contenido estático. AFD proporciona a la aplicación un rendimiento de usuario final de primer orden, una automatización de mantenimiento de marca/regional unificada, una automatización de BCDR, una información de cliente/usuario unificada, un almacenamiento en caché y una información detallada de servicios. La plataforma ofrece rendimiento, confiabilidad y compatibilidad con los acuerdos de nivel de servicio, certificaciones de cumplimiento y prácticas de seguridad auditables desarrolladas y operadas por Azure, y compatibles de forma nativa con Azure.

[**Application Gateway**][AppGW] Microsoft Azure Application Gateway es una aplicación virtual dedicada que proporciona un controlador de entrega de aplicaciones (ADC) como servicio y ofrece numerosas funcionalidades de equilibrio de carga de nivel 7 para la aplicación. Le permite optimizar la productividad de las granjas de servidores web traspasando la carga de la terminación SSL con mayor actividad de la CPU a Application Gateway. Además, dispone de otras funcionalidades de enrutamiento de nivel 7, como la distribución round robin del tráfico entrante, la afinidad de sesiones basada en cookies, el enrutamiento basado en rutas de acceso URL y la capacidad de hospedar varios sitios web detrás de una única puerta de enlace de aplicaciones. También se proporciona un firewall de aplicaciones web (WAF) como parte de la SKU de WAF de Application Gateway. Esta SKU proporciona protección a las aplicaciones web frente a vulnerabilidades web y vulnerabilidades de seguridad comunes. Application Gateway puede configurarse como una puerta de enlace orientada a Internet, una puerta de enlace solo para uso interno o una combinación de las dos. 

[**Application Gateway**][AppGW] es una aplicación virtual dedicada que ofrece un controlador de entrega de aplicaciones (ADC) que se ofrece como servicio y que proporciona numerosas funcionalidades de equilibrio de carga de nivel 7 para una aplicación. Puede optimizar la productividad de las granjas de servidores web traspasando la carga de la terminación SSL con uso intensivo de la CPU a la instancia de Application Gateway. También proporciona otras funcionalidades de enrutamiento de capa 7 entre las que se incluyen las siguientes: 
* Distribución round robin del tráfico entrante. 
* Afinidad de sesión basada en cookies. 
* Enrutamiento basado en URL. 
* La capacidad de hospedar varios sitios web en una sola instancia de Application Gateway. También se proporciona un firewall de aplicaciones web (WAF) como parte de la SKU de WAF de Application Gateway. Esta SKU proporciona protección a las aplicaciones web frente a vulnerabilidades web y vulnerabilidades de seguridad comunes. Application Gateway puede configurarse como una puerta de enlace accesible desde Internet, una puerta de enlace solo para uso interno o una combinación de las dos. 

[**Direcciones IP públicas**][PIP]. Algunas características de Azure le permiten asociar los puntos de conexión de servicio a una dirección IP pública que permite al recurso estar accesible desde Internet. Este punto de conexión usa la traducción de direcciones de red (NAT) para enrutar el tráfico a la dirección interna y el puerto de la red virtual de Azure. Esta ruta es la vía principal para que el tráfico externo pase a la red virtual. Las direcciones IP públicas se pueden configurar para determinar qué tráfico se pasa y cómo y a dónde se traslada en la red virtual.

[**Azure DDoS Protection estándar**][DDOS] ofrece funcionalidades adicionales de mitigación en comparación con el nivel de [Servicio básico][DDOS], adaptadas específicamente a los recursos de Azure Virtual Network. El servicio Protección contra DDoS estándar es fácil de habilitar y no requiere ningún cambio en la aplicación. Las directivas de protección se ajustan a través de la supervisión del tráfico dedicado y los algoritmos de Machine Learning. Las directivas se aplican a direcciones IP públicas asociadas a recursos implementados en redes virtuales. Algunos ejemplos son instancias de Azure Load Balancer, Azure Application Gateway y Azure Service Fabric. La telemetría en tiempo real está disponible a través de las vistas de Azure Monitor durante un ataque y para el historial. Se puede agregar protección en la capa de aplicación mediante el firewall de aplicaciones web de Azure Application Gateway. Se proporciona protección para direcciones IP públicas de Azure IPv4.

#### <a name="component-type-monitoring"></a>Tipo de componente: Supervisión

Los componentes de supervisión proporcionan visibilidad y alertas sobre todos los demás tipos de componentes. Todos los equipos deben tener acceso a la supervisión de los componentes y servicios a los que tienen acceso. Si tiene equipos de soporte técnico o de operaciones centralizados, necesitan un acceso integrado a los datos que proporcionan estos componentes.

Azure ofrece diferentes tipos de servicios de registro y supervisión para realizar un seguimiento del comportamiento de los recursos hospedados en Azure. La regulación y el control de las cargas de trabajo en Azure no se basa solo en recopilar datos de registro, sino también en la posibilidad de desencadenar acciones basándose en eventos notificados específicos.

[**Azure Monitor**][Monitor]. Azure incluye varios servicios que realizan individualmente una tarea o un rol específico en el espacio de supervisión. Juntos, estos servicios ofrecen una solución completa para recopilar, analizar y actuar en la telemetría de la aplicación y los recursos de Azure que las admiten. También pueden servir para supervisar recursos locales críticos, a fin de proporcionar un entorno de supervisión híbrido. Conocer las herramientas y los datos que están disponibles es el primer paso para desarrollar una estrategia de supervisión completa para la aplicación.

Hay dos tipos principales de registros en Azure:

-   Los [registros de actividad de Azure][ActLog] (conocidos anteriormente como **Registros operativos**) proporcionan una visión general de las operaciones realizadas en recursos de la suscripción de Azure. Estos registros informan sobre los eventos en el plano de control de las suscripciones. Todos los recursos de Azure generan registros de auditoría.

-   [Los registros de diagnóstico de Azure Monitor][DiagLog] son registros generados por un recurso que proporcionan datos exhaustivos y frecuentes acerca del funcionamiento de ese recurso. El contenido de estos registros varía según el tipo de recurso.

[![9]][9]

Es importante realizar el seguimiento de los registros de los grupos de seguridad de red, especialmente de esta información:

-   [Registros de eventos][NSGLog]: proporcionan información sobre qué reglas de grupos de seguridad de red se aplican a las máquinas virtuales y a los roles de instancia en función de la dirección MAC.
-   [Registros de contador][NSGLog]: realizan el seguimiento sobre cuántas veces se aplica cada regla de los grupos de seguridad de red para denegar o permitir el tráfico.

Todos los registros se pueden almacenar en cuentas de Azure Storage con fines de auditoría, análisis estático o copia de seguridad. Cuando los registros se almacenan en una cuenta de Azure Storage, los clientes pueden usar diferentes tipos de plataformas de trabajo para recuperar, preparar, analizar y visualizar estos datos para notificar el estado de los recursos en la nube. 

Las grandes empresas ya deberían haber adquirido una plataforma estándar para la supervisión de sistemas locales. Pueden ampliar esa plataforma para que integre los registros generados por las implementaciones de nube. Mediante el uso de [Azure Log Analytics] [https://docs.microsoft.com/en-us/azure/log-analytics/log-analytics-queries], las organizaciones pueden mantener todos los registros en la nube. Log Analytics se implementa como un servicio basado en la nube. Por tanto, lo puede encender y ejecutar rápidamente con una mínima inversión en servicios de infraestructura. Log Analytics puede también integrarse con componentes de System Center, como System Center Operations Manager, para ampliar sus inversiones existentes de administración en la nube. 

Log Analytics es un servicio de Azure que ayuda a recopilar, correlacionar, buscar y actuar en los datos de registro y rendimiento generados por los sistemas operativos, aplicaciones y componentes de infraestructura en la nube. Ofrece a los clientes información detallada acerca de las operaciones en tiempo real mediante la búsqueda integrada y paneles personalizados para analizar todos los registros de todas las cargas de trabajo del centro de datos virtual.

[Azure Network Watcher][NetWatch] proporciona herramientas para supervisar, diagnosticar, ver las métricas y habilitar o deshabilitar registros de recursos en una red virtual de Azure. Es un servicio polifacético, lo que permite las funcionalidades siguientes y muchas más:
-    Supervisión de la comunicación entre una máquina virtual y un punto de conexión.
-    Visualización de recursos de una red virtual y sus relaciones.
-    Diagnóstico de problemas de filtrado del tráfico de red hacia o desde una máquina virtual.
-    Diagnóstico de problemas de enrutamiento de red desde una máquina virtual.
-    Diagnóstico de conexiones de salida desde una máquina virtual.
-    Captura de paquetes hacia y desde una máquina virtual.
-    Diagnóstico de problemas con conexiones y la puerta de enlace de una red virtual de Azure.
-    Determinación de latencias relativas entre las regiones de Azure y los proveedores de acceso a Internet.
-    Visualización de las reglas de seguridad para una interfaz de red.
-    Visualización de métricas de red.
-    Análisis del tráfico hacia y desde un grupo de seguridad de red.
-    Visualización de registros de diagnóstico de recursos de red.

La solución [Network Performance Monitor][NPM] de Operations Management Suite puede proporcionar una información de red detallada de un extremo a otro. Esta información incluye una vista única de las redes de Azure y las redes locales. La solución dispone de monitores específicos para los servicios públicos y ExpressRoute.

#### <a name="component-type-workloads"></a>Tipo de componente: Cargas de trabajo

Los componentes de carga de trabajo son donde residen los servicios y aplicaciones reales. También es donde los equipos de desarrollo de aplicaciones dedican la mayor parte de su tiempo.

Las posibilidades de cargas de trabajo son infinitas. Estos son solo algunos de los tipos posibles de cargas de trabajo:

**Aplicaciones de línea de negocio internas**: Las aplicaciones de línea de negocio son aplicaciones informáticas necesarias para la operación en curso de una empresa. Las aplicaciones de línea de negocio tienen algunas características comunes:

-   Son **interactivas** por naturaleza. Se especifican los datos y se devuelven resultados o informes.
-   Se **basan en el uso intensivo de datos**&mdash; con acceso frecuente a bases de datos o a otros almacenamientos de datos.
-   Integración de ofertas **integrada**&mdash; con otros sistemas dentro y fuera de la organización.

**Sitios web accesibles para clientes (accesibles desde Internet o internamente)**: La mayoría de las aplicaciones que interactúan con Internet son sitios web. Azure ofrece la funcionalidad de ejecutar un sitio web en una máquina virtual de IaaS o desde un sitio de [Azure Web Apps] [ WebApps] (PaaS). Azure Web Apps admite la integración con redes virtuales que permiten la implementación de Web Apps en una zona de red de radios. Los sitios web a los que se accede internamente no necesitan exponer un punto de conexión público de Internet, ya que se puede acceder a los recursos a través de direcciones privadas enrutables sin conexión a internet desde la red privada virtual.

**Macrodatos y análisis**: Cuando los datos tengan que escalarse verticalmente a volumen grande, es posible que las bases de datos no se escalen correctamente. La tecnología de Hadoop ofrece un sistema para ejecutar consultas distribuidas en paralelo en un gran número de nodos. Los clientes tienen la opción de ejecutar las cargas de trabajo de datos en máquinas virtuales IaaS o bien en PaaS ([HDInsight][HDI]). HDInsight admite la implementación en una red virtual basada en la ubicación y se puede implementar en un clúster en un radio del centro de datos virtual.

**Eventos y mensajería**: [Azure Event Hubs][EventHubs] es un servicio de ingestión de datos de telemetría a hiperescala que recopila, transforma y almacena millones de eventos. Como plataforma de streaming distribuida, ofrece retención de tiempo configurable y baja latencia, lo que permite recopilar grandes cantidades de datos de telemetría en Azure y leer los datos desde varias aplicaciones. Con Event Hubs, una única transmisión puede admitir canalizaciones en tiempo real y basadas en lotes.

Puede implementar un servicio de mensajería altamente confiable en la nube entre aplicaciones y servicios mediante [Azure Service Bus][ServiceBus]. Este ofrece mensajería asincrónica entre cliente y servidor, además de mensajería estructurada de tipo FIFO (primero en entrar, primero en salir) y funcionalidades de publicación y suscripción.

[![10]][10]

### <a name="making-the-vdc-highly-available-multiple-vdcs"></a>Hacer que el centro de datos virtual esté altamente disponible: varios centros de datos virtuales

Hasta ahora, este artículo se ha centrado en el diseño de un único centro de datos virtual y en él se describen los componentes básicos y la arquitectura que contribuyen a su resistencia. Las características de Azure como el equilibrador de carga de Azure, las aplicaciones virtuales de red, los conjuntos de disponibilidad, los conjuntos de escalado y otros mecanismos contribuyen a un sistema que le permite crear niveles de Acuerdo de Nivel de Servicio sólidos en los servicios de producción.

Sin embargo, como los centros de datos virtuales individuales se implementa normalmente en una sola región, pueden ser vulnerables a cualquier interrupción importante que afecte a toda esa región. Los clientes que requieran acuerdos de nivel de servicio de alto nivel deben proteger los servicios a través de implementaciones del mismo proyecto en dos implementaciones de centros de datos virtuales (o más) ubicadas en regiones diferentes.

Además de aspectos relacionados con los Acuerdos de Nivel de Servicio, hay varios escenarios comunes en los que tiene sentido implementar varios centros de datos virtuales:

-   Presencia regional o global.
-   Recuperación ante desastres
-   Un mecanismo para desviar el tráfico entre centros de datos.

#### <a name="regionalglobal-presence"></a>Presencia regional y global

Los centros de datos de Azure están presentes en varias regiones de todo el mundo. Si seleccionan varios centros de datos de Azure, los clientes deben tener en cuenta dos factores relacionados: distancias geográficas y latencia. Para ofrecer la mejor experiencia posible al usuario, evalúe la distancia geográfica entre cada implementación de un centro de datos virtual, así como la distancia entre estas y los usuarios finales.

La región donde se hospedan las implementaciones de los centros de datos virtuales deben cumplir los requisitos de regulación establecidos por la jurisdicción en la que opere la organización.

#### <a name="disaster-recovery"></a>Recuperación ante desastres

El diseño de un plan de recuperación ante desastres depende de los tipos de cargas de trabajo y de la capacidad parar sincronizar el estado dichas cargas entre diferentes implementaciones de centros de datos virtuales. Idealmente, la mayoría de los clientes desea un mecanismo de conmutación por error rápido, lo que puede que requiera la sincronización de los datos de las aplicaciones entre las implementaciones que ejecutan varias implementaciones de centros de datos virtuales. Sin embargo, al diseñar los planes de recuperación ante desastres, es importante tener en cuenta que a la mayoría de las aplicaciones les afecta la latencia que puede provocar dicha sincronización de datos.

La sincronización y la supervisión del latido de las aplicaciones en diferentes implementaciones de centros de datos virtuales exige comunicación a través de la red. Dos implementaciones de centros de datos virtuales de diferentes regiones se pueden conectar:

-   Emparejamiento de redes virtuales: permite la conexión de centros de diferentes regiones
-   Emparejamiento privado de ExpressRoute cuando los centros cada todas las implementaciones de centros de datos virtuales están conectados al mismo circuito de ExpressRoute
-   Varios circuitos de ExpressRoute conectados a través de la red troncal corporativa y varias implementaciones de centros de datos virtuales conectadas a los circuitos de ExpressRoute
-   Conexiones VPN de sitio a sitio entre la zona del centro de conectividad de las implementaciones de los centros de datos virtuales de cada región de Azure

Normalmente se prefiere el emparejamiento de redes virtuales o las conexiones de ExpressRoute son el tipo de conectividad de red preferido, ya que tienen mayor ancho de banda y niveles de latencia coherentes al pasar por la red troncal de Microsoft.

Es aconsejable que los clientes ejecuten pruebas de calificación de red para comprobar la latencia y el ancho de banda de dichas conexiones y decidir si la replicación de datos sincrónica o asincrónica es adecuada en función del resultado. También es importante sopesar estos resultados teniendo en cuenta el objetivo de tiempo de recuperación óptimo (RTO).

#### <a name="disaster-recovery-diverting-traffic-from-one-region-to-another"></a>Recuperación ante desastres: desvío del tráfico de una región a otra

[Azure Traffic Manager][TM] comprueba periódicamente el mantenimiento del servicio de los puntos de conexión públicos en implementaciones de centros de datos virtuales diferentes y, si se produce algún error en ellos, enruta automáticamente al centro de datos virtual secundario mediante el Sistema de nombres de dominio (DNS). 

Al usar DNS, Traffic Manager solo se puede usar con puntos de conexión públicos de Azure.  El servicio se utiliza normalmente para controlar o desviar el tráfico a máquinas virtuales de Azure y aplicaciones web en la instancia en buen estado de una implementación de un centro de datos virtual. Traffic Manager es resistente incluso al enfrentarse un error de una región de Azure completa y puede controlar la distribución del tráfico de usuario hacia puntos de conexión de servicio en centros de datos virtuales diferentes en función de varios criterios. Por ejemplo, el error de un servicio de una implementación de un centro de datos virtual específico o la selección del centro de datos virtual con la menor latencia de red.

### <a name="summary"></a>Resumen

El centro de datos virtual es un enfoque hacia la migración del centro de datos para crear una arquitectura escalable de Azure que maximiza el uso de los recursos en la nube, reduce los costos y simplifica el gobierno del sistema. El centro de datos virtual se basa en una topología de red en estrella tipo hub-and-spoke que proporciona servicios compartidos comunes en centro de conectividad y permite aplicaciones y cargas de trabajo específicas en los radios. El centro de datos virtual coincide con la estructura de roles de la empresa, en la que diferentes departamentos, como TI Central, DevOps, operaciones y mantenimiento, funcionan conjuntamente, aunque realicen sus roles concretos. El centro de datos virtual cumple los requisitos de una migración mediante "Lift-and-Shift", pero también proporciona muchas ventajas para las implementaciones nativas en la nube.

## <a name="references"></a>Referencias

Las siguientes características se describen en este documento. Para más información, siga los vínculos.

| | | |
|-|-|-|
|Características de red|Equilibrio de carga|Conectividad|
|[Redes virtuales de Azure][VNet]</br>[Grupos de seguridad de red][NSG]</br>[Registros NSG][NSGLog]</br>[Enrutamiento definido por el usuario][UDR]</br>[Aplicaciones virtuales de red][NVA]</br>[Direcciones IP públicas][PIP]</br>[Azure DDOS][DDOS]</br>[Azure Firewall][AzFW]</br>[Azure DNS][DNS]|[Azure Front Door][AFD]</br>[Azure Load Balancer (L3) ][ALB]</br>[Application Gateway (L7) ][AppGW]</br>[Firewall de aplicaciones web][WAF]</br>[Azure Traffic Manager][TM]</br></br></br></br></br> |[Emparejamiento de redes virtuales][VNetPeering]</br>[Red privada virtual][VPN]</br>[Virtual WAN][vWAN]</br>[ExpressRoute][ExR]</br>[ExpressRoute Direct][ExRD]</br></br></br></br></br>
|Identidad</br>|Supervisión</br>|Prácticas recomendadas</br>|
|[Azure Active Directory][AAD]</br>[Multi-Factor Authentication][MFA]</br>[Control de acceso basado en roles][RBAC]</br>[Roles de Azure AD predeterminados][Roles]</br></br></br> |[Network Watcher][NetWatch]</br>[Azure Monitor][Monitor]</br>[Registros de actividad][ActLog]</br>[Registros de diagnóstico][DiagLog]</br>[Microsoft Operations Management Suite][OMS]</br>[Network Performance Monitor][NPM]|[Procedimientos recomendados en redes perimetrales][DMZ]</br>[Administración de suscripciones][SubMgmt]</br>[Administración de grupos de recursos][RGMgmt]</br>[Límites de la suscripción de Azure][Limits] </br></br></br>|
|Otros servicios de Azure|
|[Azure Web Apps][WebApps]</br>[HDInsights (Hadoop) ][HDI]</br>[Event Hubs][EventHubs]</br>[Service Bus][ServiceBus]|

## <a name="next-steps"></a>Pasos siguientes

 - Explore [Emparejamiento de redes virtuales][VNetPeering], la tecnología subyacente a los diseños de concentrador y radio del centro de datos virtual
 - Implemente [Azure AD][AAD] para empezar a trabajar con la exploración de [RBAC][RBAC]
 - Desarrolle un modelo de administración de recursos y suscripciones, y un modelo de RBAC que cumpla la estructura, los requisitos y las directivas de su organización. La actividad más importante es el planeamiento. Siempre que resulte práctico, planee las reorganizaciones, fusiones, nuevas líneas de producto, etc.<!--Image References-->
[0]: ./images/networking-redundant-equipment.png "Ejemplos de superposición de componentes" 
[1]: ./images/networking-vdc-high-level.png "Ejemplo genérico de centro de datos virtual con concentrador y radios"
[2]: ./images/networking-hub-spokes-cluster.png "Clúster de concentradores y radios"
[3]: ./images/networking-spoke-to-spoke.png "Radio a radio"
[4]: ./images/networking-vdc-block-level-diagram.png "Diagrama de nivel de bloques del centro de datos virtual"
[5]: ./images/networking-users-groups-subsciptions.png "Usuarios, grupos, suscripciones y proyectos"
[6]: ./images/networking-infrastructure-high-level.png "Diagrama genérico de infraestructura"
[7]: ./images/networking-highlevel-perimeter-networks.png "Diagrama genérico de infraestructura"
[8]: ./images/networking-vnet-peering-perimeter-neworks.png "Emparejamiento de redes virtuales y redes perimetrales"
[9]: ./images/networking-high-level-diagram-monitoring.png "Diagrama genérico para supervisión"
[10]: ./images/networking-high-level-workloads.png "Diagrama genérico para cargas de trabajo"

<!--Link References-->
[Limits]: /azure/azure-subscription-service-limits
[Roles]: /azure/role-based-access-control/built-in-roles
[VNet]: /azure/virtual-network/virtual-networks-overview
[NSG]: /azure/virtual-network/virtual-networks-nsg
[DNS]: /azure/dns/dns-overview
[PrivateDNS]: /azure/dns/private-dns-overview
[VNetPeering]: /azure/virtual-network/virtual-network-peering-overview 
[UDR]: /azure/virtual-network/virtual-networks-udr-overview 
[RBAC]: /azure/role-based-access-control/overview
[MFA]: /azure/multi-factor-authentication/multi-factor-authentication
[AAD]: /azure/active-directory/active-directory-whatis
[VPN]: /azure/vpn-gateway/vpn-gateway-about-vpngateways 
[ExR]: /azure/expressroute/expressroute-introduction
[ExRD]: https://docs.microsoft.com/en-us/azure/expressroute/expressroute-erdirect-about
[vWAN]: /azure/virtual-wan/virtual-wan-about
[NVA]: /azure/architecture/reference-architectures/dmz/nva-ha
[AzFW]: /azure/firewall/overview
[SubMgmt]: /azure/architecture/cloud-adoption/appendix/azure-scaffold 
[RGMgmt]: /azure/azure-resource-manager/resource-group-overview
[DMZ]: /azure/best-practices-network-security
[ALB]: /azure/load-balancer/load-balancer-overview
[DDOS]: /azure/virtual-network/ddos-protection-overview
[PIP]: /azure/virtual-network/resource-groups-networking#public-ip-address
[AFD]: https://docs.microsoft.com/en-us/azure/frontdoor/front-door-overview
[AppGW]: /azure/application-gateway/application-gateway-introduction
[WAF]: /azure/application-gateway/application-gateway-web-application-firewall-overview
[Monitor]: /azure/monitoring-and-diagnostics/
[ActLog]: /azure/monitoring-and-diagnostics/monitoring-overview-activity-logs 
[DiagLog]: /azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs
[NSGLog]: /azure/virtual-network/virtual-network-nsg-manage-log
[OMS]: /azure/operations-management-suite/operations-management-suite-overview
[NPM]: /azure/log-analytics/log-analytics-network-performance-monitor
[NetWatch]: /azure/network-watcher/network-watcher-monitoring-overview
[WebApps]: /azure/app-service/
[HDI]: /azure/hdinsight/hdinsight-hadoop-introduction
[EventHubs]: /azure/event-hubs/event-hubs-what-is-event-hubs 
[ServiceBus]: /azure/service-bus-messaging/service-bus-messaging-overview
[TM]: /azure/traffic-manager/traffic-manager-overview