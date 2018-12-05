---
title: 'Centro de datos virtual de Azure: una perspectiva de red'
description: Aprenda a crear su centro de datos virtual en Azure
author: tracsman
manager: rossort
tags: azure-resource-manager
ms.service: virtual-network
ms.date: 09/24/2018
ms.author: jonor
ms.openlocfilehash: b10930ac6d6458872d8b626825d21bd0bed2b807
ms.sourcegitcommit: 16bc6a91b6b9565ca3bcc72d6eb27c2c4ae935e4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/28/2018
ms.locfileid: "52550468"
---
# <a name="azure-virtual-datacenter-a-network-perspective"></a>Centro de datos virtual de Azure: una perspectiva de red

## <a name="overview"></a>Información general
La migración de aplicaciones locales a Azure, incluso sin realizar cambios importantes, proporciona a las organizaciones las ventajas de una infraestructura segura y rentable. Este enfoque se conoce como **lift- and -shift**. Para aprovechar al máximo posible la agilidad de la informática en la nube, las empresas deben evolucionar sus arquitecturas para aprovechar las ventajas de las funcionalidades de Azure. Microsoft Azure ofrece servicios e infraestructura a gran escala, funcionalidades de calidad empresarial, confiabilidad y numerosas opciones de conectividad híbrida. 

Los clientes pueden elegir acceder a estos servicios en la nube a través de Internet o con Azure ExpressRoute, que proporciona conectividad de red privada. Con la plataforma Microsoft Azure, los clientes pueden ampliar su infraestructura en la nube sin problemas y crear arquitecturas de varios niveles. Los asociados de Microsoft también proporcionan funcionalidades mejoradas al ofrecer servicios de seguridad y aplicaciones virtuales optimizados para su ejecución en Azure.

Este artículo proporciona información general sobre patrones y diseños que permiten resolver los conceptos de arquitectura de escala, rendimiento y seguridad a los que muchos clientes se enfrentan al pensar en moverse masivamente a la nube. También se trata información general sobre el ajuste de distintos roles de TI de la organización en la administración y gobierno del sistema. Se hace especial hincapié en los requisitos de seguridad y la optimización de costos.

## <a name="what-is-a-virtual-datacenter"></a>¿Qué es un centro de datos virtual?
Las soluciones en la nube se diseñaron para hospedar aplicaciones únicas y relativamente aisladas en el espectro público. Este método funcionó bien unos años. Posteriormente, las ventajas de las soluciones en la nube se hicieron obvias y se hospedaban muchas cargas de trabajo a gran escala en la nube. Dar respuesta a los problemas de seguridad, confiabilidad, rendimiento y costo de las implementaciones en una o varias regiones resultó ser vital a lo largo del ciclo de vida del servicio en la nube.

El siguiente diagrama de implementación en la nube muestra un ejemplo de una brecha de seguridad en el **recuadro rojo**. El **recuadro amarillo** muestra áreas de optimización de aplicaciones virtuales de red en las cargas de trabajo.

[![0]][0]

El centro de datos virtual (VDC) surgió de la necesidad de escalar para poder admitir cargas de trabajo empresariales. También pretende dar respuesta a los problemas que surgen al admitir aplicaciones a gran escala en la nube pública.

El centro de datos virtual no solo tiene que ver con las cargas de trabajo de las aplicaciones en la nube. También tiene que ver con la red, la seguridad, la administración y la infraestructura. Algunos ejemplos son el DNS y los servicios de directorio. Habitualmente proporciona una conexión privada con una red o centro de datos local. Dado que cada vez se mueven más cargas de trabajo a Azure, es importante pensar en la infraestructura de soporte y los objetos en los que se colocan estas cargas de trabajo. Piense detenidamente sobre cómo se estructuran los recursos para evitar la proliferación de cientos de **islas de cargas de trabajo** que deben administrarse por separado con flujo de datos y modelos de seguridad independientes y desafíos de cumplimiento de normas.

Un centro de datos virtual es una colección de entidades independientes, aunque relacionadas, con funciones, características e infraestructura de soporte en común. Considerando las cargas de trabajo como un centro de datos virtual integrado, puede reducir los costos de un sistema de escalado y seguridad optimizados mediante la centralización del flujo de datos y de componentes. También conseguirá unas operaciones, administración y auditorías de cumplimiento normativo mucho más sencillas.

> [!NOTE]  
> Es importante comprender que el centro de datos virtual **no** es un producto independiente de Azure. Es la combinación de varias características y funcionalidades creada para satisfacer sus necesidades exactas. Un centro de datos virtual es una manera de pensar en sus cargas de trabajo y el uso de Azure para maximizar los recursos y funcionalidades en la nube. El centro de datos virtual, por tanto, es un enfoque modular sobre cómo crear los servicios de TI en Azure, respetando las responsabilidades y roles de la organización.

El centro de datos virtual puede ayudar a que las empresas lleven cargas de trabajo y aplicaciones a Azure en los siguientes escenarios:

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

## <a name="who-should-implement-a-virtual-datacenter"></a>¿Quién debe implementar un centro de datos virtual?
Cualquier cliente de Azure que necesite mover varias cargas de trabajo a Azure puede beneficiarse del uso de recursos comunes. Dependiendo del tamaño, incluso las aplicaciones únicas pueden beneficiarse del uso de los patrones y componentes que se utilizan para crear el centro de datos virtual.

Si su organización cuenta con un equipo o departamento de TI, redes, seguridad y cumplimiento normativo centralizado, el VDC puede ayudarle a aplicar los puntos de las directivas y la segregación del servicio. También se garantiza la uniformidad de los componentes comunes subyacentes al tiempo que da a los equipos de aplicación la libertad y control apropiados para sus requisitos.

Las organizaciones que se plantean el uso de Azure DevOps pueden utilizar los conceptos del centro de datos virtual para proporcionar paquetes autorizados de recursos de Azure y garantizar el control total dentro de ese grupo. Los grupos son suscripciones o grupos recursos de una suscripción común. No obstante, se debe mantener el cumplimiento de los límites de red y seguridad definidos por una directiva centralizada en una red virtual y grupo de recursos de un centro.

## <a name="considerations-when-you-implement-a-virtual-datacenter"></a>Consideraciones a la hora de implementar un centro de datos virtual
Hay varios asuntos esenciales a tener en cuenta al diseñar un centro de datos virtual:

-   Servicios de identidad y directorio.
-   Infraestructura de seguridad.
-   Conectividad a la nube.
-   Conectividad dentro de la nube.

### <a name="identity-and-directory-services"></a>Servicios de identidad y directorio
Los servicios de identidad y directorio son un aspecto clave de todos los centros de datos, tanto en local como en la nube. La identidad está relacionada con todos los aspectos de acceso y autorización a los servicios dentro del centro de datos virtual. Para garantizar que solo los usuarios y procesos autorizados tienen acceso a la cuenta de Azure y los recursos, Azure usa varios tipos de credenciales para la autenticación. Estos incluyen contraseñas (para acceder a la cuenta de Azure), claves de cifrado, firmas digitales y certificados. 

[Azure Multi-Factor Authentication (MFA)][MFA] es una capa adicional de seguridad para el acceso a los servicios de Azure. Proporciona autenticación segura con una gama de opciones de comprobación sencillas. Los usuarios pueden elegir el método que prefieran. Entre las opciones se incluyen una llamada telefónica, un mensaje de texto o una notificación para una aplicación móvil. 

Cualquier empresa de gran tamaño precisa definir un proceso de administración de identidad que describa la administración de identidades individuales, su autenticación, autorización, roles y privilegios dentro del centro de datos virtual. Los objetivos de este proceso son, por una parte, mejorar la seguridad y productividad y, por otra, reducir el costo, el tiempo de inactividad y las tareas manuales repetitivas.

Puede que las empresas y organizaciones requieran una combinación exigente de servicios para las diferentes líneas de negocio. Y los empleados a menudo tienen distintos roles cuando se involucran en proyectos diferentes. Un centro de datos virtual requiere buena cooperación entre distintos equipos, cada uno con definiciones de rol específicas, para tener una buena gestión de los sistemas en ejecución. La matriz de responsabilidades, acceso y derechos puede ser compleja. La administración de identidad en el centro de datos virtual se implementa a través de [Azure Active Directory (Azure AD)][AAD] y el control de acceso basado en rol (RBAC).

Un servicio de directorio es una infraestructura de información compartida para buscar, gestionar, administrar y organizar los elementos cotidianos y los recursos de red. Estos recursos pueden incluir volúmenes, carpetas, archivos, impresoras, usuarios, grupos, dispositivos y otros objetos. El servidor de directorio considera cada recurso de la red como un objeto. La información sobre un recurso se almacena como una colección de atributos asociada a ese recurso u objeto.

Todos los servicios de negocio en línea de Microsoft dependen de Azure Active Directory (Azure AD) para el inicio de sesión y otras necesidades de identidad. Azure Active Directory es solución en la nube de administración de acceso e identidades completa y de alta disponibilidad que combina una administración de acceso a aplicaciones, gobernancia de identidades avanzada y servicios de directorio fundamentales. Azure AD se puede integrar con Active Directory local a fin de habilitar el inicio de sesión único para todas las aplicaciones basadas en la nube y hospedadas localmente. Los atributos de usuario de Active Directory local se pueden sincronizar automáticamente con Azure AD.

No es necesario que un administrador global único asigne todos los permisos en una implementación de VDC. En su lugar, cada departamento específico (o grupo de usuarios o servicios en el servicio de directorio) puede tener los permisos necesarios para administrar sus propios recursos dentro del centro de datos virtual. La estructura de permisos requiere un equilibrio. Demasiados permisos impiden un rendimiento eficaz. Pocos permisos o unos permisos demasiado flexibles aumentan los riesgos de seguridad. Gracias al control de acceso basado en rol (RBAC) de Azure podrá abordar este problema, ya que es posible realizar una administración avanzada del acceso a los recursos del centro de datos virtual.

#### <a name="security-infrastructure"></a>Infraestructura de seguridad
La infraestructura de seguridad hace referencia a la segregación del tráfico en el segmento de red virtual específico del centro de datos virtual y a cómo controlar los flujos de entrada y salida en el centro de datos virtual. Azure se basa en una arquitectura multiinquilino que impide el tráfico no autorizado y no intencionado entre implementaciones. Usa aislamiento de red virtual, listas de control de acceso (ACL), equilibradores de carga, filtros IP y directivas de flujo de tráfico. La traducción de direcciones de red (NAT) se usa para separar el tráfico de red interno del tráfico externo.

El tejido de Azure asigna recursos de infraestructura para las cargas de trabajo del inquilino y administra las comunicaciones a y desde las máquinas virtuales (VM). El hipervisor de Azure fuerza la separación de la memoria y el proceso entre las máquinas virtuales y enruta de forma segura el tráfico de red a los SO invitado de cada inquilino.

#### <a name="connectivity-to-the-cloud"></a>Conectividad a la nube
La implementación de un centro de datos virtual necesita conectividad con redes externas para ofrecer servicios a los clientes, asociados y usuarios internos. Esto suele significar conectividad no solo con Internet, sino también con redes y centros de datos locales.

Los clientes controlan qué servicios pueden acceder al Internet público y a cuáles se puede acceder desde este mediante [Azure Firewall][AzFW] o cualquier otro tipo de aplicaciones de red virtual (NVA), directivas de enrutamiento personalizadas mediante [rutas definidas por el usuario][UDR] y filtrado de redes mediante [grupos de seguridad de red][NSG]. Se recomienda que todos los recursos accesibles desde Internet estén también protegidos por [Azure DDOS Protection estándar][DDOS].

Las empresas a menudo necesitan conectar las implementaciones de VDC con centros de datos locales u otros recursos. La conectividad entre Azure y las redes locales es un aspecto fundamental al diseñar una arquitectura eficaz. Las empresas tienen dos maneras diferentes de crear interconexiones entre el centro de datos virtual y el local en Azure: tránsito a través de Internet o de conexiones directas privadas.

Una [VPN de sitio a sitio de Azure][VPN] conecta la red local de una empresa a su implementación del centro de datos virtual a través de la red pública de Internet. El tráfico a través de la VPN se cifra mediante el uso de IPSec y la tunelización de IKE. Una conexión de sitio a sitio de Azure es flexible y se crea rápidamente. Esta VPN no requiere ninguna adquisición adicional ya que todas las conexiones se realizan a través de Internet.

Para un gran número de conexiones VPN, [Azure Virtual WAN][vWAN] es un servicio de redes que ofrece conectividad de rama a rama automatizada y optimizada mediante Azure. Con Virtual WAN puede conectar y configurar los dispositivos de rama para comunicarse con Azure. Esta conexión se puede realizar manualmente o con dispositivos de proveedores preferidos mediante un asociado de Virtual WAN. El uso de dispositivos de proveedores preferidos permite la administración de la configuración, facilita el uso y simplifica la conectividad. El panel integrado de Azure WAN proporciona información instantánea para solucionar problemas que puede ayudarle a ahorrar tiempo. Y le ofrece una manera fácil de ver la conectividad de sitio a sitio a gran escala.

[ExpressRoute][ExR], un servicio de conectividad de Azure, permite las conexiones privadas entre la red local de la empresa y la implementación de su centro de datos virtual. Ofrece una mayor seguridad, confiabilidad y velocidades más altas, hasta 10 Gbps, con una latencia constante. ExpressRoute es muy útil para las implementaciones de centros de datos virtuales, ya que los clientes de ExpressRoute se pueden beneficiar de las reglas de cumplimiento de normas asociadas con las conexiones privadas. Con [ExpressRoute Direct][ExRD] puede conectarse directamente a los enrutadores de Microsoft a 100 Gbps, para los clientes con mayores necesidades de ancho de banda.

La implementación de conexiones ExpressRoute implica normalmente tomar contacto con un proveedor de servicios de ExpressRoute. Para clientes que necesitan comenzar rápidamente, es habitual usar inicialmente una red privada virtual de sitio a sitio para establecer la conectividad entre el centro de datos virtual y los recursos locales. Posteriormente, migran a una conexión ExpressRoute cuando finaliza la interconexión física con su proveedor de servicios.

#### <a name="connectivity-within-the-cloud"></a>Conectividad dentro de la nube
Las [redes virtuales][VNet] y el [emparejamiento de redes virtuales][VNetPeering] son los servicios de conectividad de red básica dentro del centro de datos virtual. Una red virtual garantiza un límite de aislamiento natural para los recursos del centro de datos virtual. Y el emparejamiento de redes virtuales permite la intercomunicación entre distintas redes virtuales dentro de la misma región de Azure o incluso entre regiones. El control del tráfico dentro de una red virtual y entre redes virtuales debe cumplir un conjunto de reglas de seguridad que se especifican mediante las listas de control de acceso. Consulte más información en [grupos de seguridad de red][NSG], [aplicaciones virtuales de red (NVA)][NVA] y [tablas de enrutamiento personalizadas para rutas definidas por el usuario][UDR].

## <a name="virtual-datacenter-overview"></a>Introducción al centro de datos virtual

### <a name="topology"></a>Topología
La topología en estrella tipo **hub-and-spoke** es un modelo que extiende el centro de datos virtual por una única región de Azure.

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
En Azure, todos los componentes, de cualquier tipo, se implementan en una suscripción de Azure. El aislamiento de componentes de Azure en distintas suscripciones de Azure puede satisfacer los requisitos de diferentes líneas de negocio. Un ejemplo es la configuración de niveles diferenciados de acceso y autorización.

Una única implementación de centro de datos virtual puede escalarse verticalmente a un gran número de radios. Pero, al igual que pasa con todos los sistemas de TI, hay límites en las plataformas. La implementación del centro se enlaza a una suscripción de Azure específica que tiene restricciones y límites. Un ejemplo es un número máximo de emparejamientos de redes virtuales. Para más información, consulte [Límites, cuotas y restricciones de suscripción y servicios de Azure][Limits]. En aquellos casos donde los límites podrían ser un problema, la arquitectura se puede escalar verticalmente aún más. Amplíe el modelo de una topología simple en estrella tipo hub-and-spoke a un clúster de centros (hubs) y radios (spokes). Conecte entre sí varios centros en una o más regiones de Azure mediante el emparejamiento de redes virtuales, ExpressRoute, Virtual WAN o una red privada virtual de sitio a sitio.

[![2]][2]

La introducción de varios centros aumenta el costo y el esfuerzo de administración del sistema. Solo se podría justificar por motivos de escalabilidad, como en el caso de los límites del sistema, y de replicación regional, como en el caso de rendimiento del usuario final o recuperación ante desastres. En escenarios que requieran múltiples concentradores, todos los concentradores deben procurar ofrecer el mismo conjunto de servicios para facilitar la operativa.

#### <a name="interconnection-between-spokes"></a>Interconexión entre los radios
En un único radio es posible implementar cargas de trabajo complejas de varios niveles. Puede implementar configuraciones de varios niveles mediante una subred para cada nivel de la misma red virtual. Filtre los flujos mediante grupos de seguridad de red.

Un arquitecto podría querer implementar una carga de trabajo de varios niveles entre varias redes virtuales. Con el emparejamiento de redes virtuales, los radios pueden conectarse a otros radios en el mismo centro o en centros diferentes. Un ejemplo típico de este escenario es el caso en el que los servidores de procesamiento de la aplicación están en un radio o red virtual. La base de datos se implementa en un radio o red virtual diferente. En este caso, es fácil interconectar los radios con emparejamiento de redes virtuales y así evitar el tránsito a través del centro. Debe realizarse una revisión cuidadosa de la arquitectura y la seguridad para asegurarse de que evitar el paso por el concentrador no omite puntos de seguridad o auditoría de importancia que podrían existir solo en el centro.

[![3]][3]

Los radios también pueden estar conectados entre sí mediante un radio que actúa como concentrador. Este método crea una jerarquía de dos niveles: los radios del nivel superior (nivel 0) se convierten en el centro de radios inferiores (nivel 1) en la jerarquía. Los radios de una implementación de centro de datos virtual están obligados a reenviar el tráfico al centro principal para llegar a la red local o a Internet. Una arquitectura con dos niveles de concentradores presenta un enrutamiento complejo que anula las ventajas de una relación simple concentrador-radio.

Aunque Azure permite topologías complejas, uno de los principios básicos del concepto de centro de datos virtual es la repetibilidad y simplicidad. Para minimizar el esfuerzo de administración, el diseño simple en estrella tipo hub-and-spoke es la arquitectura de referencia recomendada para un centro de datos virtual.

### <a name="components"></a>Componentes
La implementación de un centro de datos virtual se compone de cuatro tipos de componentes básicos: **infraestructura**, **redes perimetrales**, **cargas de trabajo** y **supervisión**.

Cada tipo de componente consta de varias características de Azure y recursos. La implementación de un centro de datos virtual de una empresa se compone de instancias de varios tipos de componentes y múltiples variaciones del mismo tipo de componente. Por ejemplo, puede tener muchas instancias de cargas de trabajo diferentes, separadas lógicamente, que representan las distintas aplicaciones. Estos distintos tipos de componentes e instancias se utilizan para crear el centro de datos virtual.

[![4]][4]

La arquitectura de alto nivel anterior de la implementación de un centro de datos virtual muestra distintos tipos de componentes utilizados en distintas zonas de la topología concentrador-radios. El diagrama muestra los componentes de la infraestructura en distintas partes de la arquitectura.

Como procedimiento recomendado para un centro de datos local o una implementación de un centro de datos virtual, los derechos de acceso y los privilegios deben estar basados en grupos. Trabajar con grupos, en lugar de usuarios individuales le ayuda a mantener las directivas de acceso de forma coherente entre los distintos equipos y ayuda a minimizar los errores de configuración. Asigne y elimine usuarios de los grupos adecuados para mantener actualizados los privilegios de un usuario específico.

Cada grupo de roles debe tener un prefijo único en sus nombres. Este prefijo facilita una rápida identificación de qué grupo está asociado con qué carga de trabajo. Por ejemplo, una carga de trabajo que hospeda un servicio de autenticación podría tener grupos denominados **AuthServiceNetOps**, **AuthServiceSecOps**, **AuthServiceDevOps** y **AuthServiceInfraOps**. Los roles centralizados o los roles no relacionados con un servicio concreto, podrían estar precedidos de **Corp**. Por ejemplo: **CorpNetOps**.

Muchas organizaciones usan una variación de los grupos siguientes para proporcionar un desglose mayor de los roles:

-   El grupo de TI central, **Corp,** tiene los derechos de propiedad para controlar los componentes de la infraestructura. Ejemplos de redes y seguridad. El grupo debe tener el rol de colaborador en la suscripción, el control del centro y derechos de colaborador de la red en los radios. Las grandes organizaciones suelen dividir estas responsabilidades de administración entre varios equipos. Ejemplos: un grupo **CorpNetOps** de operaciones de red con atención exclusiva a las redes, y un grupo **CorpSecOps** de operaciones de seguridad responsable del firewall y la directiva de seguridad. En este caso específico, deben crearse dos grupos diferentes para la asignación de estos roles personalizados.
-   El grupo de desarrollo y pruebas **AppDevOps** tiene la responsabilidad de implementar las cargas de trabajo de aplicaciones o servicios. Este grupo tiene el rol de colaborador de la máquina Virtual para las implementaciones de IaaS y uno o más roles de colaborador de PaaS. Consulte [Roles integrados en los recursos de Azure][Roles]. Opcionalmente, el equipo de desarrollo y pruebas podría tener visibilidad en las directivas de seguridad (NSG) y en las directivas de enrutamiento (UDR) en el centro o en un radio específico. Además de los roles de colaborador para las cargas de trabajo, este grupo también necesitaría el rol de lector de red.
-   El grupo de operaciones y mantenimiento **CorpInfraOps** o **AppInfraOps** tiene la responsabilidad de administrar las cargas de trabajo en producción. Este grupo debe ser un colaborador de la suscripción en las cargas de trabajo en las suscripciones de producción. Algunas organizaciones también pueden evaluar si necesitan un grupo de equipo de soporte técnico con el rol de colaborador en la suscripción de producción y en la suscripción del centro principal. El grupo adicional permite solucionar posibles problemas de configuración en el entorno de producción.

Un centro de datos virtual se estructura para que los grupos de TI centrales creados que administran el centro tengan grupos correspondientes en el nivel de carga de trabajo. Además de la administración de los recursos del centro, los grupos de TI central deberían ser los únicos que controlaran el acceso externo y los permisos de nivel superior en la suscripción. Sin embargo, los grupos de cargas de trabajo controlarían los recursos y los permisos de su red virtual con independencia de TI central.

Cree particiones en el centro de datos virtual para hospedar varios proyectos de forma segura en diferentes líneas de negocio. Todos los proyectos requieren diferentes entornos aislados como, por ejemplo, desarrollo, UAT y producción. El uso de suscripciones de Azure independientes para cada uno de estos entornos proporciona un aislamiento natural.

[![5]][5]

El diagrama anterior muestra la relación entre los proyectos de una organización, los usuarios, grupos y los entornos donde se implementan los componentes de Azure.

Normalmente, en TI, un entorno (o nivel) es un sistema en el que se implementan y ejecutan varias aplicaciones. Las empresas de gran tamaño usan un entorno de desarrollo para realizar y probar cambios, y un entorno de producción que emplean los usuarios finales. Esos entornos están separados, a menudo con varios entornos de ensayo entre ellos para permitir la implementación por fases (lanzamiento), las pruebas y la reversión en caso de problemas. Las arquitecturas de implementación varían considerablemente, aunque normalmente siguen el proceso básico que comienza en un desarrollo **DEV** y termina en producción **PROD**.

Una arquitectura común para estos tipos de entornos de varios niveles se compone de Azure DevOps para desarrollo y pruebas, UAT para ensayo y entornos de producción. Las organizaciones pueden aprovechar uno o varios inquilinos de Azure AD para definir el acceso y los derechos para estos entornos. El diagrama anterior muestra un caso en el que se usan dos inquilinos diferentes de Azure AD: uno para Azure DevOps y UAT y otro exclusivamente para producción.

La presencia de inquilinos de Azure AD diferentes refuerza la separación entre entornos. El mismo grupo de usuarios, por ejemplo, TI central, debe autenticarse con un identificador URI diferente para acceder a un inquilino de Azure AD diferente y modificar los roles o permisos de los entornos de Azure DevOps o de producción de un proyecto. La presencia de una autenticación de usuario diferente para acceder a diferentes entornos reduce posibles interrupciones y otros problemas causados por errores humanos.

#### <a name="component-type-infrastructure"></a>Tipo de componente: infraestructura
Este tipo de componente es en el que reside la mayor parte de la infraestructura de soporte. También es donde los equipos de TI central, seguridad y cumplimiento dedican la mayor parte de su tiempo.

[![6]][6]

Los componentes de infraestructura proporcionan una interconexión entre los distintos componentes de un centro de datos virtual. Ellos están presentes tanto en el centro como en los radios. Normalmente se asigna la responsabilidad de administrar y mantener los componentes de infraestructura al equipo de TI central o al de seguridad.

Una de las tareas principales del equipo de infraestructura de TI es garantizar la coherencia de los esquemas de direcciones IP en toda la empresa. El espacio de direcciones IP privado asignado a las necesidades del centro de datos virtual debe ser coherente. No puede solaparse con las direcciones IP privadas asignadas en sus redes locales.

NAT en los enrutadores perimetrales locales o en entornos de Azure puede evitar conflictos de dirección IP. Pero agrega complicaciones a los componentes de la infraestructura. La simplicidad de administración es uno de los principales objetivos del centro de datos virtual. Por tanto, usar NAT para controlar los problemas de IP no es una solución recomendable.

Los componentes de infraestructura tienen la siguiente funcionalidad:

-   [**Servicios de identidad y directorio**][AAD]. El acceso a cada tipo de recurso en Azure se controla mediante una identidad que se almacena en un servicio de directorio. El servicio de directorio almacena la lista de usuarios y los derechos de acceso a los recursos de una suscripción de Azure específica. Estos servicios pueden existir solo en la nube. O bien, se pueden sincronizar con una identidad local almacenada en Azure Active Directory.
-   [**Red virtual**][VPN]. Las redes virtuales son uno de los componentes principales del centro de datos virtual. Mediante el uso de redes virtuales, puede crear un límite para el aislamiento de tráfico en la plataforma Azure. Una red virtual se compone de uno o varios segmentos de red virtual. Cada uno tiene un prefijo de red IP específico, una subred. La red virtual define un área de perímetro interno en el que las máquinas virtuales de IaaS y los servicios de PaaS pueden establecer comunicaciones privadas. Los servicios de máquinas virtuales y de PaaS de una red virtual no se pueden comunicar directamente con los de una red virtual diferente. Este aislamiento se produce incluso si el mismo cliente crea ambas redes virtuales en la misma suscripción. El aislamiento consiste en una propiedad fundamental que garantiza que las máquinas virtuales del cliente y la comunicación siguen siendo privadas en una red virtual.
-   [**UDR**][UDR]. El tráfico en una red virtual se enruta de forma predeterminada en función de la tabla de enrutamiento del sistema. Una ruta definida por el usuario es una tabla de enrutamiento personalizada que los administradores de red pueden asociar a una o varias subredes. Esta asociación sobrescribe el comportamiento de la tabla de enrutamiento del sistema y define una ruta de comunicación en una red virtual. La presencia de una UDR garantiza que el tráfico de salida del radio transita a través de máquinas virtuales personalizadas específicas o aplicaciones virtuales de red y equilibradores de carga presentes en el centro y los radios.
-   [**NSG**][NSG]. Un grupo de seguridad de red (NSG) es una lista de reglas de seguridad que actúa como filtro de tráfico en direcciones IP de origen y destino, protocolos y puertos IP de origen y destino. El NSG se puede aplicar a una subred, a una tarjeta NIC virtual asociada a una máquina virtual de Azure, o a ambos. Los grupos de seguridad de red son fundamentales para implementar un control de flujo correcto en el centro y en los radios. El nivel de seguridad permitido por el grupo de seguridad de red está en función de los puertos que abra y su finalidad. Los clientes deben aplicar filtros adicionales en cada máquina virtual con firewalls basados en host, como IPtables o el Firewall de Windows.
-   [**DNS**][DNS]. La resolución de nombres de recursos en las redes virtuales de un centro de datos virtual se proporciona mediante DNS. Azure proporciona servicios DNS tanto para la resolución de nombres [públicos][DNS] como [privados][PrivateDNS]. Las zonas privadas proporcionan la resolución de nombres dentro de una red virtual o en redes virtuales distintas. Puede hacer que las zonas privadas abarquen redes virtuales de la misma región, e incluso de distintas regiones y suscripciones. Para la resolución pública, Azure DNS proporciona un servicio de hospedaje para dominios DNS. Proporciona resolución de nombres mediante el uso de la infraestructura de Microsoft Azure. Al hospedar dominios en Azure, puede administrar los registros de DNS con las mismas credenciales, API, herramientas y facturación que con los demás servicios de Azure.
-   [**Administración de la suscripción**][SubMgmt] y del [**Grupo de recursos**][RGMgmt]. Una suscripción define un límite natural para crear varios grupos de recursos en Azure. Los recursos de una suscripción se ensamblan juntos en contenedores lógicos llamados **grupos de recursos**. El grupo de recursos representa un grupo lógico para organizar los recursos de un centro de datos virtual.
-   [**RBAC**][RBAC]. Mediante RBAC, es posible asignar roles organizativos junto con los derechos de acceso a recursos específicos de Azure, lo que le permite restringir a los usuarios a solo un subconjunto determinado de acciones. Con RBAC puede conceder acceso asignando el rol adecuado a usuarios, grupos y aplicaciones dentro del ámbito correspondiente. El ámbito de una asignación de roles puede ser una suscripción de Azure, un grupo de recursos o un único recurso. RBAC permite la herencia de permisos. Un rol asignado en un ámbito principal también concede acceso a los elementos secundarios dentro del mismo. Con RBAC podrá repartir las tareas y conceder a los usuarios únicamente el nivel de acceso que necesitan para realizar su trabajo. Por ejemplo, utilice RBAC para que un empleado puede administrar las máquinas virtuales de una suscripción. Otro puede administrar las bases de datos SQL de la misma suscripción.
-   [**Emparejamiento de redes virtuales**][VNetPeering]. La característica fundamental que se utiliza para crear la infraestructura del centro de datos virtual es el emparejamiento de redes virtuales. Es un mecanismo que conecta dos redes virtuales en la misma región a través de la red del centro de datos de Azure o mediante el uso de la red troncal internacional de Azure entre regiones.

#### <a name="component-type-perimeter-networks"></a>Tipo de componente: redes perimetrales
Con los componentes de la [red perimetral][DMZ] (también conocida como una red DMZ), puede proporcionar conectividad de red con su red local o con la del centro de datos físico, junto con cualquier tipo de conectividad hacia Internet y desde este. También es donde probablemente los equipos de red y seguridad empleen la mayor parte de su tiempo.

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

El diagrama anterior muestra la aplicación de dos perímetros con acceso a Internet y a una red local. Ambos residen en los centros DMZ y WAN virtual. En el centro DMZ, la red perimetral a Internet puede escalarse verticalmente para admitir grandes cantidades de líneas de negocio mediante varias granjas de firewalls de aplicaciones web (WAF) o instancias de Azure Firewall. En el centro WAN virtual, la conectividad altamente escalable rama a rama y rama a Azure se logra a través de VPN o ExpressRoute según sea necesario.

[**Virtual Networks**][VNet]. El centro normalmente se basa en una red virtual con varias subredes para hospedar los distintos tipos de servicios con filtrado e inspección del tráfico hacia o desde Internet mediante NVA, WAF e instancias de Azure Application Gateway.

[**Rutas definidas por el usuario**][UDR]. Con las rutas definidas por el usuario, los clientes pueden implementar firewalls, IDS o IPS, y otras aplicaciones virtuales. También pueden enrutar el tráfico de red a través de estas aplicaciones de seguridad para la aplicación de directivas de límites de seguridad, auditoría e inspección. Puede crear UDR en el centro y en los radios. Estas garantizan que el tráfico transita a través de máquinas virtuales personalizadas específicas, aplicaciones virtuales de red y equilibradores de carga utilizados en el centro de datos virtual. Para asegurarse de que el tráfico generado desde las máquinas virtuales que residen en el radio transite hacia las aplicaciones virtuales correctas, establezca una UDR en las subredes del radio. Establezca la dirección IP de front-end del equilibrador de carga interno como el tipo de próximo salto. El equilibrador de carga interno distribuye el tráfico interno a las aplicaciones virtuales o al grupo de back-end de equilibradores de carga.

[**Azure Firewall**][AzFW] es un servicio de seguridad de red administrado y basado en la nube que protege los recursos de Azure Virtual Network. Se trata de un firewall como servicio con estado completo que incorpora alta disponibilidad y escalabilidad a la nube sin restricciones. Puede crear, aplicar y registrar directivas de aplicaciones y de conectividad de red a nivel central en suscripciones y redes virtuales. Azure Firewall usa una dirección IP pública estática para los recursos de red virtual. Esto permite que los firewalls externos identifiquen el tráfico que procede de su red virtual. El servicio está totalmente integrado con Azure Monitor para los registros y análisis.

[**Aplicaciones virtuales de red**][NVA]. En el centro, la red perimetral con acceso a Internet se administra normalmente a través de una instancia de Azure Firewall o una granja de firewalls o mediante un firewall de aplicaciones web (WAF).

Las diferentes líneas de negocio normalmente utilizan muchas aplicaciones web. Estas aplicaciones tienden a sufrir varias vulnerabilidades y posibles puntos débiles. Los firewall de aplicaciones web son un tipo especial de producto que se usa para detectar ataques contra las aplicaciones web (HTTP/HTTPS) con mayor profundidad que un firewall genérico. En comparación con la tecnología de firewall tradicional, los WAF tienen un conjunto de características específicas para proteger los servidores web internos frente a amenazas.

No obstante, se requiere que una única implementación de un centro de datos virtual se hospede en una única región ya que los requisitos de la suscripción no permiten la ampliación de regiones. Este requisito hace que una única implementación de un centro de datos virtual sea vulnerable a las interrupciones de servicio regionales.

Una instancia de Azure Firewall o una granja de firewalls de aplicación virtual de red utilizan un plano de administración común. Un conjunto de reglas de seguridad protege las cargas de trabajo hospedadas en los radios y controla el acceso a las redes locales. Azure Firewall tiene escalabilidad integrada, mientras que los firewalls de aplicación virtual de red se pueden escalar manualmente detrás de un equilibrador de carga. Por lo general, una granja de firewalls tiene menos software especializado en comparación con WAF. Pero tiene un ámbito de aplicación más amplio para filtrar e inspeccionar cualquier tipo de tráfico de salida o de entrada. Si se usa un enfoque de aplicación virtual de red, puede encontrarlos e implementarlos desde Azure Marketplace.

Es recomendable que use un conjunto de instancias de Azure Firewall o de aplicaciones virtuales de red para el tráfico que se origina en Internet. Use otro para el tráfico que se origina en el entorno local. Utilizar únicamente un conjunto de firewalls para ambos es un riesgo de seguridad, ya que no proporciona ningún perímetro de seguridad entre los dos conjuntos de tráfico de red. El uso de capas de firewalls independientes reduce la complejidad de la comprobación de las reglas de seguridad y deja claro qué reglas se corresponden con cada solicitud de red entrante.

La mayoría de las empresas de tamaño grande administran varios dominios. [Azure DNS][DNS] se puede usar para hospedar los registros DNS de un dominio concreto. Como ejemplo, se puede registrar la dirección IP virtual (VIP) del equilibrador de carga externo de Azure (o de los WAF) en el registro **A** de un registro de Azure DNS. [**DNS privado** ][PrivateDNS] también está disponible para administrar los espacios de direcciones privadas dentro de las redes virtuales.

[**Azure Load Balancer**][ALB] ofrece un servicio de capa 4 de alta disponibilidad, TCP o UDP. Puede distribuir el tráfico entrante entre instancias de servicio definidas en un conjunto de carga equilibrada. El tráfico enviado al equilibrador de carga desde los puntos de conexión front-end, con direcciones IP públicas o privadas, se puede redistribuir con o sin traducción de direcciones a un conjunto de grupo de direcciones IP de back-end. Algunos ejemplos son las aplicaciones virtuales de red o las máquinas virtuales.

Azure Load Balancer puede sondear también el estado de las distintas instancias de servidor. Cuando un sondeo no responde, el equilibrador de carga deja de enviar tráfico a las instancias incorrectas. En el centro de datos virtual, un equilibrador de carga externo del centro equilibra el tráfico hacia las aplicaciones virtuales de red, por ejemplo. En los radios, realiza tareas como equilibrar el tráfico entre diferentes máquinas virtuales de una aplicación de varios niveles.

[**Azure Front Door**][AFD] es la plataforma de aceleración de aplicaciones web altamente disponible y escalable de Microsoft, el equilibrador de carga HTTP global, la protección de aplicaciones y la red de entrega de contenido. Front Door se ejecuta en más de 100 ubicaciones de la red global de Microsoft. Con Front Door puede compilar, operar y escalar horizontalmente sus aplicaciones web dinámicas y el contenido estático. Front Door proporciona a la aplicación las siguientes ventajas: 
* Rendimiento de primer orden para el usuario final.
* Automatización unificada regional o de mantenimiento de marcas.
* Automatización de la continuidad empresarial y recuperación ante desastres.
* Información de cliente o usuario unificada. 
* Almacenamiento en caché. 
* Información de servicio. La plataforma ofrece rendimiento, confiabilidad y compatibilidad con los Acuerdos de Nivel de Servicio, certificaciones de cumplimiento y procedimientos de seguridad auditables desarrollados y operados por Azure, y compatibles de forma nativa con Azure.

[**Application Gateway**][AppGW] es una aplicación virtual dedicada que ofrece un controlador de entrega de aplicaciones (ADC) que se ofrece como servicio y que proporciona numerosas funcionalidades de equilibrio de carga de nivel 7 para una aplicación. Puede optimizar la productividad de las granjas de servidores web traspasando la carga de la terminación SSL con uso intensivo de la CPU a la instancia de Application Gateway. También proporciona otras funcionalidades de enrutamiento de capa 7 entre las que se incluyen las siguientes: 
* Distribución round robin del tráfico entrante. 
* Afinidad de sesión basada en cookies. 
* Enrutamiento basado en URL. 
* La capacidad de hospedar varios sitios web en una sola instancia de Application Gateway. También se proporciona un firewall de aplicaciones web (WAF) como parte de la SKU de WAF de Application Gateway. Esta SKU proporciona protección a las aplicaciones web frente a vulnerabilidades web y vulnerabilidades de seguridad comunes. Application Gateway puede configurarse como una puerta de enlace accesible desde Internet, una puerta de enlace solo para uso interno o una combinación de las dos. 

[**Direcciones IP públicas**][PIP]. Algunas características de Azure le permiten asociar los puntos de conexión de servicio a una dirección IP pública que permite al recurso estar accesible desde Internet. Este punto de conexión usa la traducción de direcciones de red (NAT) para enrutar el tráfico a la dirección interna y el puerto de la red virtual de Azure. Esta ruta es la vía principal para que el tráfico externo pase a la red virtual. Las direcciones IP públicas se pueden configurar para determinar qué tráfico se pasa y cómo y a dónde se traslada en la red virtual.

[**Azure DDoS Protection estándar**][DDOS] ofrece funcionalidades adicionales de mitigación en comparación con el nivel de [Servicio básico][DDOS], adaptadas específicamente a los recursos de Azure Virtual Network. El servicio Protección contra DDoS estándar es fácil de habilitar y no requiere ningún cambio en la aplicación. Las directivas de protección se ajustan a través de la supervisión del tráfico dedicado y los algoritmos de Machine Learning. Las directivas se aplican a direcciones IP públicas asociadas a recursos implementados en redes virtuales. Algunos ejemplos son instancias de Azure Load Balancer, Azure Application Gateway y Azure Service Fabric. La telemetría en tiempo real está disponible a través de las vistas de Azure Monitor durante un ataque y para el historial. Se puede agregar protección en la capa de aplicación mediante el firewall de aplicaciones web de Azure Application Gateway. Se proporciona protección para direcciones IP públicas de Azure IPv4.

#### <a name="component-type-monitoring"></a>Tipo de componente: supervisión
Los componentes de supervisión proporcionan visibilidad y alertas sobre todos los demás tipos de componentes. Todos los equipos deben tener acceso a la supervisión de los componentes y servicios a los que tienen acceso. Si tiene equipos de soporte técnico o de operaciones centralizados, necesitarán un acceso integrado a los datos proporcionados por estos componentes.

Azure ofrece diferentes tipos de servicios de registro y supervisión para realizar un seguimiento del comportamiento de los recursos hospedados en Azure. La regulación y el control de las cargas de trabajo en Azure no se basa solo en recopilar datos de registro, sino también en la posibilidad de desencadenar acciones basándose en eventos notificados específicos.

[**Azure Monitor**][Monitor]. Azure incluye varios servicios que realizan individualmente una tarea o un rol específico en el espacio de supervisión. Juntos, estos servicios ofrecen una solución completa para recopilar, analizar y actuar en la telemetría de la aplicación y los recursos de Azure que las admiten. También pueden servir para supervisar recursos locales críticos, a fin de proporcionar un entorno de supervisión híbrido. Conocer las herramientas y los datos que están disponibles es el primer paso para desarrollar una estrategia de supervisión completa para la aplicación.

Hay dos tipos principales de registros en Azure:

-   Los [registros de actividad de Azure][ActLog] (conocidos anteriormente como **Registros operativos**) proporcionan una visión general de las operaciones realizadas en recursos de la suscripción de Azure. Estos registros informan sobre los eventos en el plano de control de las suscripciones. Todos los recursos de Azure generan registros de auditoría.

-   [Los registros de diagnóstico de Azure Monitor][DiagLog] son registros generados por un recurso que proporcionan datos exhaustivos y frecuentes acerca del funcionamiento de ese recurso. El contenido de estos registros varía según el tipo de recurso.

[![9]][9]

En un centro de datos virtual es muy importante realizar el seguimiento de los registros de los grupos de seguridad de red, especialmente esta información:

-   [Registros de eventos][NSGLog]: proporcionan información sobre qué reglas de grupos de seguridad de red se aplican a las máquinas virtuales y a los roles de instancia en función de la dirección MAC.
-   [Registros de contador][NSGLog]: realizan el seguimiento sobre cuántas veces se aplica cada regla de los grupos de seguridad de red para denegar o permitir el tráfico.

Todos los registros se pueden almacenar en cuentas de Azure Storage con fines de auditoría, análisis estático o copia de seguridad. Cuando los registros se almacenan en una cuenta de Azure Storage, los clientes pueden usar diferentes tipos de plataformas de trabajo para recuperar, preparar, analizar y visualizar estos datos para notificar el estado de los recursos en la nube. 

Las grandes empresas ya deberían haber adquirido una plataforma estándar para la supervisión de sistemas locales. Pueden ampliar esa plataforma para que integre los registros generados por las implementaciones de nube. Mediante el uso de [Azure Log Analytics] [https://docs.microsoft.com/en-us/azure/log-analytics/log-analytics-queries], las organizaciones pueden mantener todos los registros en la nube. Log Analytics se implementa como un servicio basado en la nube. Por tanto, lo puede encender y ejecutar rápidamente con una mínima inversión en servicios de infraestructura. Log Analytics puede también integrarse con componentes de System Center, como System Center Operations Manager, para ampliar sus inversiones existentes de administración en la nube. 

Log Analytics es un servicio de Azure que ayuda a recopilar, correlacionar, buscar y actuar en los datos de registro y rendimiento generados por los sistemas operativos, aplicaciones y componentes de infraestructura en la nube. Ofrece a los clientes una visión operativa en tiempo real mediante la búsqueda integrada y los paneles personalizados para analizar todos los registros en todas las cargas de trabajo del centro de datos virtual.

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

#### <a name="component-type-workloads"></a>Tipo de componente: cargas de trabajo
Los componentes de carga de trabajo son donde residen los servicios y aplicaciones reales. También es donde los equipos de desarrollo de aplicaciones dedican la mayor parte de su tiempo.

Las posibilidades de cargas de trabajo son infinitas. Estos son solo algunos de los tipos posibles de cargas de trabajo:

Las **aplicaciones internas de línea de negocio** son aplicaciones informáticas necesarias para el trabajo diario de una empresa. Las aplicaciones de línea de negocio tienen algunas características comunes:

-   Son **interactivas** por naturaleza. Se especifican los datos y se devuelven resultados o informes.
-   Se **basan en el uso intensivo de datos**&mdash; con acceso frecuente a bases de datos o a otros almacenamientos de datos.
-   Integración de ofertas **integrada**&mdash; con otros sistemas dentro y fuera de la organización.

**Sitios web accesibles para clientes (accesibles desde Internet o internos)**. La mayoría de las aplicaciones que interactúan con Internet son sitios web. Azure ofrece la posibilidad de ejecutar un sitio web en una máquina virtual de IaaS o desde una [característica Web Apps de Microsoft Azure App Service][WebApps] (PaaS). Web Apps admite la integración con redes virtuales que permite implementar Web Apps en los radios de un centro de datos virtual. Al examinar sitios web internos, con integración de red virtual, no es necesario exponer un punto de conexión de Internet para sus aplicaciones. Puede usar los recursos a través de direcciones privadas enrutables que no son de Internet desde la red virtual privada en su lugar.

**Macrodatos y análisis**. Cuando necesita escalar verticalmente los datos hasta un volumen muy grande, las bases de datos pueden no escalar correctamente. La tecnología de Apache Hadoop ofrece un sistema para ejecutar consultas distribuidas en paralelo en un gran número de nodos. Los clientes pueden ejecutar las cargas de trabajo de datos en máquinas virtuales IaaS o bien en PaaS ([Azure HDInsight][HDI]). HDInsight admite la implementación en una red virtual basada en la ubicación. Se puede implementar en un clúster de un radio del centro de datos virtual.

**Eventos y mensajería**. [Azure Event Hubs][EventHubs] es un servicio de ingesta de datos de telemetría a hiperescala que recopila, transforma y almacena millones de eventos. Como plataforma de streaming distribuida, ofrece retención de tiempo configurable y baja latencia. Por tanto, puede ingerir grandes cantidades de datos de telemetría en Azure y leerlos desde múltiples aplicaciones. Con Event Hubs, una única transmisión puede admitir canalizaciones en tiempo real y basadas en lotes.

Puede implementar un servicio de mensajería altamente confiable en la nube entre aplicaciones y servicios mediante [Azure Service Bus][ServiceBus]. Este ofrece mensajería asincrónica entre cliente y servidor, además de mensajería estructurada de tipo FIFO (primero en entrar, primero en salir) y funcionalidades de publicación y suscripción.

[![10]][10]

### <a name="multiple-vdc-implementations"></a>Varias implementaciones de centros de datos virtuales


Sin embargo, un único centro de datos virtual se hospeda en una única región. Es vulnerable a una interrupción importante del servicio que podría afectar a toda esa región. Los clientes que desean conseguir Acuerdos de Nivel de Servicio de alto nivel necesitan proteger los servicios a través de implementaciones del mismo proyecto en dos o más implementaciones de centros de datos virtuales, ubicados en regiones diferentes.

Además de aspectos relacionados con los Acuerdos de Nivel de Servicio, hay varios escenarios comunes en los que tiene sentido implementar varios centros de datos virtuales:

-   Presencia regional o global.
-   Recuperación ante desastres
-   Un mecanismo para desviar el tráfico entre centros de datos.

#### <a name="regional-and-global-presence"></a>Presencia regional y global
Los centros de datos de Azure están presentes en muchas regiones de todo el mundo. Al seleccionar varios centros de datos de Azure, los clientes deben tener en cuenta dos factores relacionados: distancias geográficas y latencia. Los clientes necesitan evaluar la distancia geográfica entre las implementaciones de centros de datos virtuales y la distancia entre estas y los usuarios finales para ofrecer la mejor experiencia de usuario.

La región de Azure donde se hospedan las implementaciones de los centros de datos virtuales también debe cumplir con los requisitos de regulación establecidos por cualquier jurisdicción en la que opere la organización.

#### <a name="disaster-recovery"></a>Recuperación ante desastres
La implementación de un plan de recuperación ante desastres está relacionada con el tipo de carga de trabajo del que se trate y la capacidad de sincronizar el estado de la carga de trabajo entre diferentes implementaciones de centros de datos virtuales. De un modo ideal, la mayoría de los clientes desea sincronizar los datos de aplicación entre implementaciones que se ejecutan en dos centros de datos virtuales diferentes para implementar un mecanismo de conmutación por error rápido. La mayoría de las aplicaciones es sensible a la latencia, lo que puede provocar un posible tiempo de espera y retraso en la sincronización de los datos.

La sincronización o la supervisión de latido de aplicaciones en diferentes implementaciones de centros de datos virtuales exige comunicación entre ellos. Dos implementaciones de centros de datos virtuales en diferentes regiones se pueden conectar de la siguiente forma:

-   El emparejamiento de redes virtuales puede conectar centros de diferentes regiones.
-   Emparejamiento privado de ExpressRoute cuando los concentradores de los centros de datos virtuales están conectados al mismo circuito de ExpressRoute.
-   Varios circuitos de ExpressRoute conectados a través de la red troncal corporativa y la malla de centros de datos virtuales conectada a circuitos de ExpressRoute.
-   Conexiones VPN de sitio a sitio entre los concentradores de los centros de datos virtuales en cada región de Azure.

Normalmente se prefiere el mecanismo de conexión ExpressRoute o de emparejamiento de redes virtuales por su mayor ancho de banda y latencia constante al transitar por la red troncal de Microsoft.

No hay ninguna receta mágica para validar una aplicación distribuida entre dos o más implementaciones de centros de datos virtuales diferentes ubicadas en regiones diferentes. Los clientes deben ejecutar pruebas de calificación de red para comprobar la latencia y el ancho de banda de las conexiones. Después, deben decidir si la replicación de datos sincrónica o asincrónica es adecuada y cuál puede ser el objetivo de tiempo de recuperación óptimo (RTO) para las cargas de trabajo.

#### <a name="mechanism-to-divert-traffic-between-datacenters"></a>Mecanismo para desviar el tráfico entre centros de datos
Una técnica eficaz para desviar el tráfico entrante en un centro de datos a otro se basa en el Sistema de nombres de dominio (DNS). [Azure Traffic Manager][TM] utiliza el mecanismo del sistema de nombres de dominio para dirigir el tráfico del usuario final hacia el punto de conexión público más adecuado en un centro de datos virtual específico. Mediante sondeos, Traffic Manager comprueba periódicamente el estado del servicio de los puntos de conexión públicos en diferentes implementaciones de centros de datos virtuales. Si se produce un error en esos puntos de conexión, el tráfico se enrutará automáticamente hacia el centro de datos virtual secundario.

Traffic Manager funciona en puntos de conexión públicos de Azure. Por ejemplo, puede controlar o desviar el tráfico a las máquinas virtuales de Azure y a Web Apps en el centro de datos virtual adecuado. Traffic Manager es resistente incluso en caso de errores que afecten a toda una región de Azure. Puede controlar la distribución del tráfico de usuario de los puntos de conexión de servicio de las diferentes implementaciones de centros de datos virtuales según varios criterios. Por ejemplo, cuando se producen errores en un centro de datos virtual específico o en la selección del centro de datos virtual con la menor latencia de red para el cliente.

### <a name="conclusion"></a>Conclusión
El centro de datos virtual es un enfoque para la migración del centro de datos a la nube que utiliza una combinación de características y funcionalidades para crear una arquitectura escalable de Azure. Esta maximiza el uso de recursos en la nube, reduce los costos y simplifica la regulación del sistema. El concepto de centro de datos virtual se basa en una topología en estrella tipo hub-and-spoke que proporciona servicios compartidos comunes en el centro. Permite determinadas aplicaciones o cargas de trabajo en los radios. El centro de datos virtual coincide con la estructura de los roles de la empresa, donde diferentes departamentos funcionan de forma conjunta. Por ejemplo, TI central, Azure DevOps, operaciones y mantenimiento. Cada departamento tiene una lista específica de roles y tareas. El concepto de centro de datos virtual cumple los requisitos para una migración **lift- and -shift**. Pero también ofrece muchas ventajas para las implementaciones nativas en la nube.

## <a name="references"></a>Referencias
Más información acerca de las siguientes características tratadas en este artículo:

| | | |
|-|-|-|
|Características de red|Equilibrio de carga|Conectividad|
|[Azure Virtual Network][VNet]</br>[Grupos de seguridad de red][NSG]</br>[Registros NSG][NSGLog]</br>[Rutas definidas por el usuario][UDR]</br>[Aplicaciones virtuales de red][NVA]</br>[Direcciones IP públicas][PIP]</br>[Azure DDoS Protection][DDOS]</br>[Azure Firewall][AzFW]</br>[Azure DNS][DNS]|[Azure Front Door][AFD]</br>[Azure Load Balancer (L3)][ALB]</br>[Application Gateway (L7)][AppGW]</br>[Firewall de aplicaciones web][WAF]</br>[Azure Traffic Manager][TM]</br></br></br></br></br> |[Emparejamiento de redes virtuales][VNetPeering]</br>[Red privada virtual][VPN]</br>[Virtual WAN][vWAN]</br>[ExpressRoute][ExR]</br>[ExpressRoute Direct][ExRD]</br></br></br></br></br>
|Identidad</br>|Supervisión</br>|Procedimientos recomendados</br>|
|[Azure Active Directory][AAD]</br>[Multi-Factor Authentication][MFA]</br>[Control de acceso basado en rol][RBAC]</br>[Roles de Azure AD predeterminados][Roles]</br></br></br> |[Network Watcher][NetWatch]</br>[Azure Monitor][Monitor]</br>[Registro de actividad][ActLog]</br>[Supervisión de los registros de diagnóstico][DiagLog]</br>[Microsoft Operations Management Suite][OMS]</br>[Network Performance Monitor][NPM]|[Procedimientos recomendados en redes perimetrales][DMZ]</br>[Administración de suscripciones][SubMgmt]</br>[Administración de grupos de recursos][RGMgmt]</br>[Límites de la suscripción de Azure][Limits] </br></br></br>|
|Otros servicios de Azure|
|[Web Apps][WebApps]</br>[HDInsight (Hadoop)][HDI]</br>[Event Hubs][EventHubs]</br>[Service Bus][ServiceBus]|

## <a name="next-steps"></a>Pasos siguientes
 - Explore [Emparejamiento de redes virtuales][VNetPeering], la tecnología subyacente a los diseños de centros y radios del centro de datos virtual.
 - Implemente [Azure AD][AAD] para empezar a trabajar con la exploración de [RBAC][RBAC].
 - Desarrolle un modelo para la administración de los recursos y suscripciones, y un modelo de RBAC para satisfacer la estructura, requisitos y directivas de su organización. La actividad más importante es el planeamiento. Siempre que resulte práctico, planee reorganizaciones, fusiones, nuevas líneas de producto y otras posibilidades.

<!--Image References-->
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

