---
title: 'Centro de datos virtual de Azure: una perspectiva de red'
description: Aprenda a crear su centro de datos virtual en Azure
author: tracsman
manager: rossort
tags: azure-resource-manager
ms.service: virtual-network
ms.date: 04/3/2018
ms.author: jonor
ms.openlocfilehash: 2dbbad3dd8d1a45b94bb4e265d306815d1f5242c
ms.sourcegitcommit: f1dcc388c8b4fc983549c36d7e6b009fa1f072ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/19/2018
ms.locfileid: "46329919"
---
# <a name="azure-virtual-datacenter-a-network-perspective"></a>Centro de datos virtual de Azure: una perspectiva de red

## <a name="overview"></a>Información general
La migración de aplicaciones locales a Azure, incluso sin realizar cambios importantes (un enfoque conocido como "levantar y mover"), proporciona a las organizaciones las ventajas de una infraestructura segura y rentable. Sin embargo, para aprovechar al máximo posible la agilidad de la informática en la nube, las empresas deben evolucionar sus arquitecturas para aprovechar las ventajas de las funcionalidades de Azure. Microsoft Azure ofrece servicios e infraestructura a gran escala, funcionalidades de calidad empresarial, confiabilidad y numerosas opciones de conectividad híbrida. Los clientes pueden elegir tener acceso a estos servicios en la nube a través de Internet o con Azure ExpressRoute, que proporciona conectividad de red privada. La plataforma Microsoft Azure permite a los clientes ampliar su infraestructura en la nube sin problemas y crear arquitecturas de varios niveles. Además, los asociados de Microsoft proporcionan funcionalidades mejoradas al ofrecer servicios de seguridad y aplicaciones virtuales optimizados para su ejecución en Azure.

Este artículo proporciona información general sobre patrones y diseños que pueden utilizarse para resolver los conceptos de arquitectura de escala, rendimiento y seguridad que muchos clientes abordan al pensar en moverse masivamente a la nube. También se trata información general sobre el ajuste de distintos roles de TI de la organización en la administración y regulación del sistema, con especial hincapié en los requisitos de seguridad y optimización de costos.

## <a name="what-is-a-virtual-data-center"></a>¿Qué es un centro de datos virtual?
En sus primeros días, las soluciones en la nube se diseñaron para hospedar aplicaciones únicas y relativamente aisladas en el espectro público. Este método funcionó bien unos años. Sin embargo, como las ventajas de las soluciones en la nube se hicieron evidentes y se han hospedado en la nube múltiples cargas de trabajo a gran escala, la gestión de la seguridad, confiabilidad, rendimiento y costo pasan a ser esenciales durante todo el ciclo de vida del servicio en la nube.

El siguiente diagrama de implementación en la nube muestra algunos ejemplos de brechas de seguridad (cuadro rojo) y el espacio para la optimización de los aplicaciones virtuales de red en las cargas de trabajo (cuadro amarillo).

[![0]][0]

El centro de datos virtual (VDC) surgió de esta necesidad de escalado para admitir cargas de trabajo empresariales y de la necesidad de tratar los problemas surgidos al implementar aplicaciones a gran escala en la nube pública.

Un centro de datos virtual no es solo las cargas de trabajo de aplicaciones en la nube, sino también la red, la seguridad, la administración y la infraestructura (por ejemplo, los servicios de directorio y DNS). Habitualmente también proporciona una conexión privada con una red o centro de datos local. Dado que cada vez se mueven más cargas de trabajo a Azure, es importante pensar en la infraestructura de soporte y los objetos en los que se colocan estas cargas de trabajo. Pensar detenidamente sobre cómo se estructuran los recursos puede evitar la proliferación de cientos de "islas de cargas de trabajo" que deben administrarse por separado con flujo de datos y modelos de seguridad independientes y desafíos de cumplimiento de normas.

Un centro de datos virtual es esencialmente una colección de entidades independientes, aunque relacionadas, con funciones, características e infraestructura de soporte en común. Si mira las cargas de trabajo como un centro de datos virtual integrado, se dará cuenta de la reducción de costos que aportan las economías de escala y la optimización de la seguridad gracias a la centralización de los componentes y flujos de datos, además de más facilidad en las operaciones, la administración y el cumplimiento de auditorías.

> [!NOTE]
> Es importante comprender que el centro de datos virtual **no** es un producto independiente de Azure, sino la combinación de varias características y funcionalidades para satisfacer sus necesidades exactas. Un centro de datos virtual es una manera de pensar en sus cargas de trabajo y el uso de Azure para maximizar los recursos y funcionalidades en la nube. El centro de datos virtual, por tanto, es un enfoque modular sobre cómo crear los servicios de TI en Azure, respetando las responsabilidades y roles de la organización.

El centro de datos virtual puede ayudar a que las empresas lleven cargas de trabajo y aplicaciones a Azure en los siguientes escenarios:

-   Hospedaje de varias cargas de trabajo relacionadas
-   Migración de cargas de trabajo desde un entorno local a Azure
-   Implementación de requisitos de seguridad y acceso compartido o centralizado para las cargas de trabajo
-   Combinación de DevOps e IT centralizada adecuada para una gran empresa

La clave para poder disfrutar de todas las ventajas del centro de datos virtual es una topología centralizada (concentrador y radios) con una combinación de características de Azure: [red virtual de Azure][VNet], [grupos de seguridad de red][NSG], [emparejamiento de redes virtuales][VNetPeering], [rutas definidas por el usuario (UDR)][UDR] e identidad de Azure con [control de acceso basado en roles (RBAC)][RBAC].

## <a name="who-needs-a-virtual-data-center"></a>¿Quién necesita un centro de datos virtual?
Cualquier cliente de Azure que necesite mover más de un par de cargas de trabajo a Azure puede obtener ventajas al contemplar el uso de recursos comunes. Dependiendo de la magnitud, incluso las aplicaciones únicas pueden beneficiarse del uso de los patrones y componentes que se utilizan para crear un centro de datos virtual.

Si su organización tiene TI, red y seguridad centralizadas o si tiene un equipo o departamento de cumplimiento, un centro de datos virtual puede ayudar a aplicar los puntos de las directivas, la segregación del servicio y asegurar la uniformidad de los componentes comunes subyacentes al tiempo que da a los equipos de aplicación la libertad y control apropiados para sus requisitos.

Las organizaciones que se plantean DevOps pueden usar los conceptos del centro de datos virtual para proporcionar paquetes autorizados de recursos de Azure y garantizar el control total dentro de ese grupo (sea suscripción o grupo de recursos en una suscripción común), manteniendo el cumplimiento de los límites de red y seguridad definidos por una directiva centralizada en un concentrador de red virtual y grupo de recursos.

## <a name="considerations-on-implementing-a-virtual-data-center"></a>Consideraciones sobre la implementación de un centro de datos virtual
Hay varios asuntos esenciales a tener en cuenta al diseñar un centro de datos virtual:

-   Servicios de identidad y directorio
-   Infraestructura de seguridad
-   Conectividad a la nube
-   Conectividad dentro de la nube

### <a name="identity-and-directory-service"></a>Servicios de identidad y directorio
Los servicios de identidad y directorio son un aspecto clave de todos los centros de datos, tanto en local como en la nube. La identidad está relacionada con todos los aspectos de acceso y autorización a los servicios dentro del centro de datos virtual. Para ayudar a garantizar que solo los usuarios y procesos autorizados tienen acceso a la cuenta de Azure y los recursos, Azure usa varios tipos de credenciales para la autenticación. Estos incluyen contraseñas (para obtener acceso a la cuenta de Azure), claves de cifrado, firmas digitales y certificados. [*Azure Multi-Factor Authentication* (MFA)][MFA] es una capa adicional de seguridad para el acceso a los servicios de Azure. Azure MFA proporciona una autenticación segura gracias a una gran variedad de opciones sencillas de verificación, como llamadas telefónicas, mensajes de texto o notificaciones de aplicaciones móviles y permite a los clientes elegir el mecanismo que prefieran.

Cualquier empresa de gran tamaño precisa definir un proceso de administración de identidad que describa la administración de identidades individuales, su autenticación, autorización, roles y privilegios dentro del centro de datos virtual. Los objetivos de este proceso deben ser mejorar la seguridad y productividad al reducir el costo, el tiempo de inactividad y las tareas manuales repetitivas.

La empresas y organizaciones pueden requerir una combinación exigente de servicios para diferentes líneas de negocio (LOB) y los empleados a menudo tienen distintos roles cuando están implicados en proyectos diferentes. Un centro de datos virtual requiere buena cooperación entre distintos equipos, cada uno con definiciones de rol específicas, para tener una buena gestión de los sistemas en ejecución. La matriz de responsabilidades, acceso y derechos puede ser muy compleja. La administración de identidad en el centro de datos virtual se implementa a través de [*Azure Active Directory* (AAD)][AAD] y el control de acceso basado en roles (RBAC).

Un servicio de directorio es una infraestructura de información compartida para buscar, gestionar, administrar y organizar los elementos cotidianos y los recursos de red. Estos recursos pueden incluir volúmenes, carpetas, archivos, impresoras, usuarios, grupos, dispositivos y otros objetos. El servidor de directorio considera cada recurso de la red como un objeto. La información sobre un recurso se almacena como una colección de atributos asociada a ese recurso u objeto.

Todos los servicios de negocio en línea de Microsoft dependen de Azure Active Directory (AAD) para el inicio de sesión y otras necesidades de identidad. Azure Active Directory es solución en la nube de administración de acceso e identidades completa y de alta disponibilidad que combina una administración de acceso a aplicaciones, gobernancia de identidades avanzada y servicios de directorio fundamentales. AAD se puede integrar con Active Directory local para habilitar el inicio de sesión único para todas las aplicaciones basadas en la nube y hospedadas localmente. Los atributos de usuario de Active Directory local se pueden sincronizar automáticamente con AAD.

No es necesario un administrador global único para asignar todos los permisos en un centro de datos virtual. En su lugar, cada departamento específico (o grupo de usuarios o servicios en el servicio de directorio) puede tener los permisos necesarios para administrar sus propios recursos dentro del centro de datos virtual. La estructura de permisos requiere un equilibrio. Demasiados permisos pueden disminuir el rendimiento y muy pocos permisos o permisos flexibles pueden aumentar los riesgos de seguridad. Gracias al control de acceso basado en roles (RBAC) de Azure, podrá abordar este problema, ya que es posible realizar una administración avanzada del acceso a los recursos del centro de datos virtual.

#### <a name="security-infrastructure"></a>Infraestructura de seguridad
La infraestructura de seguridad, en el contexto de un centro de datos virtual, está relacionada fundamentalmente con la segregación del tráfico en el segmento de red virtual específico del centro de datos virtual y con el control de los flujos de entrada y salida en el centro de datos virtual. Azure se basa en una arquitectura multiinquilino que impide que el tráfico no autorizado y no intencionado entre implementaciones, con aislamiento de red virtual (VNet), listas de control de acceso (ACL), equilibradores de carga y filtros de direcciones IP, junto con directivas de flujo de tráfico. La traducción de direcciones de red (NAT) se usa para separar el tráfico de red interno del tráfico externo.

El tejido de Azure asigna recursos de infraestructura para las cargas de trabajo del inquilino y administra las comunicaciones a y desde las máquinas virtuales (VM). El hipervisor de Azure fuerza la separación de la memoria y el proceso entre las máquinas virtuales y enruta de forma segura el tráfico de red a los SO invitado de cada inquilino.

#### <a name="connectivity-to-the-cloud"></a>Conectividad a la nube
El centro de datos virtual necesita conectividad con redes externas para ofrecer servicios a los clientes, asociados o usuarios internos. Esto suele significar conectividad no solo con Internet, sino también con redes y centros de datos locales.

Los clientes pueden crear directivas de seguridad para controlar qué servicios hospedados en el centro de datos virtual son accesibles desde Internet y de qué modo lo son mediante el uso de aplicaciones virtuales de red (con filtrado e inspección de tráfico) y de directivas de enrutamiento personalizadas y filtrado de red (enrutamiento definido por el usuario y grupos de seguridad de red).

Las empresas a menudo necesitan conectar los centros de datos virtuales con centros de datos locales u otros recursos. La conectividad entre Azure y las redes locales, por tanto, es un aspecto fundamental al diseñar una arquitectura efectiva. Las empresas tienen dos maneras diferentes de crear una interconexión entre el centro de datos virtual y el local en Azure: tránsito a través de Internet o a través de conexiones directas privadas.

Una [**red privada virtual sitio a sitio de Azure**][VPN] es un servicio de interconexión a través de Internet entre redes locales y el centro de datos virtual establecido a través de conexiones cifradas seguras (túneles IPsec y IKE). La conexión sitio a sitio de Azure es flexible, rápida de crear y no requiere ninguna adquisición adicional, ya que todas las conexiones se realizan a través de Internet.

[**ExpressRoute**][ExR] es un servicio de conectividad de Azure que le permite crear conexiones privadas entre el centro de datos virtual y las redes locales. Las conexiones ExpressRoute no se realizan sobre una conexión a Internet pública, ofrecen una mayor confiabilidad, seguridad y velocidades mayores (hasta 10 Gbps) con una latencia menor. ExpressRoute es muy útil para los centros de datos virtuales, ya que los clientes de ExpressRoute pueden tener las ventajas de las reglas de cumplimiento de normas asociadas con las conexiones privadas.

La implementación de conexiones ExpressRoute implica tomar contacto con un proveedor de servicios de ExpressRoute. Para clientes que necesitan comenzar rápidamente, es habitual usar inicialmente una red privada virtual de sitio a sitio para establecer la conectividad entre el centro de datos virtual y los recursos locales y, posteriormente, migrar a una conexión ExpressRoute.

#### <a name="connectivity-within-the-cloud"></a>*Conectividad dentro de la nube*
Las [redes virtuales][VNet] y el [emparejamiento de redes virtuales][VNetPeering] son los servicios de conectividad de red básica dentro de un centro de datos virtual. Una red virtual garantiza un límite de aislamiento natural para los recursos del centro de datos virtual y el emparejamiento de redes virtuales permite la intercomunicación entre distintas redes virtuales dentro de la misma región de Azure o incluso de distintas regiones. El control de tráfico dentro de una red virtual y entre redes virtuales debe cumplir con un conjunto de reglas de seguridad especificado a través de listas de control de acceso ([Grupo de seguridad de red][NSG]), [aplicaciones virtuales de red][NVA] y tablas de enrutamiento personalizadas ([UDR][UDR]).

## <a name="virtual-data-center-overview"></a>Información general del centro de datos virtual

### <a name="topology"></a>Topología
_Concentrador y radios_ es un modelo que extiende el centro de datos virtual por una única región de Azure.

[![1]][1]

El concentrador es la zona central que controla e inspecciona el tráfico de entrada y salida entre las diferentes zonas: Internet, local y los radios. La topología de concentrador y radio ofrece al departamento de TI una manera eficaz de aplicar directivas de seguridad en una ubicación central, al tiempo que reduce la posibilidad de una configuración incorrecta y la exposición.

El concentrador contiene los componentes de los servicios comunes utilizados por los radios. Estos son algunos ejemplos típicos de servicios centrales comunes:

-   La infraestructura de Active Directory de Windows (con el servicio ADFS relacionado) necesaria para la autenticación de usuarios de terceros que acceden desde redes que no son de confianza antes de obtener acceso a las cargas de trabajo del radio
-   Un servicio DNS para resolver nombres para la carga de trabajo en los radios para el acceso a recursos locales y en Internet
-   Una infraestructura PKI, para implementar el inicio de sesión único en las cargas de trabajo
-   Control de flujo (TCP/UDP) entre los radios e Internet
-   Control de flujo entre el radio y local
-   Si lo desea, control de flujo entre un radio y otro

El centro de datos virtual reduce el costo total al utilizar la infraestructura de concentrador compartida entre varios radios.

El rol de cada radio puede ser el hospedaje de diferentes tipos de cargas de trabajo. Los radios también pueden proporcionar un enfoque modular para implementaciones repetibles (por ejemplo, desarrollo y pruebas, pruebas de aceptación de usuario, el entorno de preproducción y producción) de las mismas cargas de trabajo. También se pueden utilizar los radios segregar y habilitar varios grupos dentro de su organización (por ejemplo, los grupos de DevOps). Dentro de un radio, es posible implementar una carga de trabajo básica o cargas de trabajo de varios niveles complejas con control de tráfico entre los niveles.

#### <a name="subscription-limits-and-multiple-hubs"></a>Límites de la suscripción y múltiples concentradores
En Azure, todos los componentes, de cualquier tipo, se implementan en una suscripción de Azure. El aislamiento de componentes de Azure en distintas suscripciones de Azure puede satisfacer los requisitos de diferentes líneas de negocio, como la configuración de niveles diferenciados de acceso y autorización.

Un único centro de datos virtual puede escalar verticalmente a gran número de radios, aunque, al igual que en todos los sistemas de TI, existen límites en las plataformas. La implementación de un concentrador se enlaza a una suscripción de Azure específica, que tiene restricciones y límites (por ejemplo, un número máximo de emparejamientos de redes virtuales: consulte [Suscripción de Azure y límites de servicio, cuotas y restricciones][Limits] para más información). En los casos en los que los límites puedan ser un problema, la arquitectura se puede escalar verticalmente más allá extiendo el modelo desde un único concentrador-radios a un clúster de concentradores y radios. Varios centros en una o más regiones de Azure pueden estar conectados entre sí mediante el emparejamiento de redes virtuales, ExpressRoute o una red privada virtual de sitio a sitio.

[![2]][2]

La introducción de varios concentradores aumenta el costo y el esfuerzo de administración del sistema y solo se justifica por la escalabilidad (por ejemplo, límites del sistema o redundancia) y por la replicación regional (por ejemplo, recuperación ante desastres o rendimiento del usuario final). En escenarios que requieran múltiples concentradores, todos los concentradores deben procurar ofrecer el mismo conjunto de servicios para facilitar la operativa.

#### <a name="interconnection-between-spokes"></a>Interconexión entre los radios
Dentro de un radio único, es posible implementar cargas de trabajo complejas en múltiples niveles. Las configuraciones de múltiples niveles se pueden implementar utilizando subredes (una para cada nivel) en la misma red virtual y filtrando los flujos con grupos de seguridad de red.

Por otro lado, un arquitecto podría querer implementar una carga de trabajo de varios niveles entre varias redes virtuales. Con el emparejamiento de redes virtuales, los radios pueden conectarse a otros radios en el mismo concentrador o en concentradores diferentes. Un ejemplo típico de este escenario es el caso en el que los servidores de procesamiento de la aplicación están en un radio (red virtual), mientras la base de datos se implementa en otro radio diferente (red virtual). En este caso, es fácil interconectar los radios con emparejamiento de redes virtuales y así evitar el tránsito a través del concentrador. Debe realizarse una revisión cuidadosa de la arquitectura y la seguridad para asegurarse de que evitar el paso por el concentrador no omite puntos de seguridad o auditoría de importancia que podrían existir solo en el concentrador.

[![3]][3]

Los radios también pueden estar conectados entre sí mediante un radio que actúa como concentrador. Este método crea una jerarquía de dos niveles: el radio del nivel superior (nivel 0) se convierten en el concentrador de radios inferiores (nivel 1) en la jerarquía. Los radios del centro de datos virtual deben reenviar el tráfico al concentrador central para llegar a la red local o a Internet. Una arquitectura con dos niveles de concentradores presenta un enrutamiento complejo que anula las ventajas de una relación simple concentrador-radio.

Aunque Azure permite topologías complejas, uno de los principios básicos del concepto de centro de datos virtual es la repetibilidad y simplicidad. Para minimizar el esfuerzo de administración, el diseño concentrador-radio simple es la arquitectura de referencia recomendada para un centro de datos virtual.

### <a name="components"></a>Componentes
Un centro de datos virtual se compone de cuatro tipos de componentes básicos: **infraestructura**, **redes perimetrales**, **cargas de trabajo** y **supervisión**.

Cada tipo de componente consta de varias características de Azure y recursos. El centro de datos virtual se compone de instancias de varios tipos de componentes y múltiples variaciones del mismo tipo de componente. Por ejemplo, puede tener muchas instancias de cargas de trabajo diferentes, separadas lógicamente, que representan las distintas aplicaciones. Estos distintos tipos de componentes e instancias se utilizan para crear el centro de datos virtual.

[![4]][4]

La arquitectura de alto nivel anterior de un centro de datos virtual muestra distintos tipos de componentes utilizados en distintas zonas de la topología concentrador-radios. El diagrama muestra los componentes de la infraestructura en distintas partes de la arquitectura.

Como procedimiento recomendado (para centros de datos locales y virtuales), los derechos de acceso y privilegios deben estar basados en grupos. Trabajar con grupos, en lugar de usuarios individuales le ayuda a mantener las directivas de acceso de forma coherente a través de los distintos equipos y ayuda a minimizar los errores de configuración. Asignar y eliminar usuarios de los grupos adecuados le ayuda a mantener actualizados los privilegios de un usuario específico.

Cada grupo de roles debe tener un prefijo único en sus nombres facilitando el proceso de identificar a qué grupo se ha asociado cada carga de trabajo. Por ejemplo, una carga de trabajo que hospeda un servicio de autenticación podría tener grupos denominados *AuthServiceNetOps, AuthServiceSecOps, AuthServiceDevOps y AuthServiceInfraOps.* Del mismo modo, para roles centralizados o roles no relacionados con un servicio concreto, podrían estar precedidos de "Corp", *CorpNetOps* por ejemplo.

Muchas organizaciones usan una variación de los grupos siguientes para proporcionar un desglose mayor de los roles:

-   El *grupo de TI central (Corp)* tiene los derechos de propietario para controlar los componentes de la infraestructura (por ejemplo, redes y seguridad) y, por lo tanto, debe disponer del rol de colaborador en la suscripción (y tiene el control del concentrador) y los derechos de colaborador de red en los radios. Las organizaciones de gran tamaño con frecuencia dividen estas responsabilidades de administración entre varios equipos como: un grupo de operaciones de red (CorpNetOps, con el foco exclusivo en la red) y un grupo de operaciones de seguridad (CorpSecOps, responsable de las directivas de seguridad y firewall). En este caso específico, deben crearse dos grupos diferentes para la asignación de estos roles personalizados.
-   El *grupo de desarrollo y pruebas (AppDevOps)* tiene la responsabilidad de implementar las cargas de trabajo (aplicaciones o servicios). Este grupo tiene el rol de colaborador de la máquina Virtual para las implementaciones de IaaS y uno o más roles del colaborador de PaaS (consulte [Roles integrados del control de acceso basado en roles de Azure][Roles]). Opcionalmente, el equipo de desarrollo y pruebas podría tener visibilidad en las directivas de seguridad (NSG) y en las directivas de enrutamiento (UDR) en el concentrador o un radio específico. Por lo tanto, además de los roles de colaborador para las cargas de trabajo, este grupo también necesitaría el rol de lector de red.
-   El *grupo de operaciones y mantenimiento (CorpInfraOps o AppInfraOps)* tiene la responsabilidad de administrar las cargas de trabajo en producción. Este grupo debe ser un colaborador de la suscripción en las cargas de trabajo en las suscripciones de producción. Algunas organizaciones también pueden evaluar si necesitan un grupo de equipo de soporte técnico con el rol de colaborador en la suscripción de producción y en la suscripción del concentrador central, con el fin de solucionar posibles problemas de configuración en el entorno de producción.

Un centro de datos virtual se estructura para que los grupos de TI centrales creados para la administración del concentrador tengan grupos correspondientes en el nivel de carga de trabajo. Además de la administración de los recursos del concentrador, los grupos de TI central deberían ser los únicos capaces de controlar el acceso externo y los permisos de nivel superior en la suscripción. Sin embargo, los grupos de cargas de trabajo serían capaces de controlar los recursos y los permisos de su red virtual con independencia de TI central.

Se deben crear particiones en el centro de datos virtual para alojar varios proyectos de forma segura en diferentes líneas de negocio (LOB). Todos los proyectos requieren diferentes entornos aislados (desarrollo, UAT, producción). El uso de suscripciones de Azure independientes para cada uno de estos entornos proporciona un aislamiento natural.

[![5]][5]

El diagrama anterior muestra la relación entre los proyectos de una organización, los usuarios, grupos y los entornos donde se implementan los componentes de Azure.

Normalmente, en TI, un entorno (o nivel) es un sistema en el que se implementan y ejecutan varias aplicaciones. Las empresas de tamaño grande usan un entorno de desarrollo (en el que se realizan originalmente los cambios y pruebas) y un entorno de producción (el que los usuarios finales utilizan). Esos entornos están separados, a menudo con varios entornos de ensayo entre ellos para permitir la implementación por fases (lanzamiento), las pruebas y la reversión en caso de problemas. Las arquitecturas de implementación varían considerablemente, aunque normalmente siguen el proceso básico de comenzar en un desarrollo (DEV) y terminar en producción (PROD).

Una arquitectura común para estos tipos de entornos de varios niveles se compone de DevOps (desarrollo y pruebas), UAT (ensayo) y entornos de producción. Las organizaciones pueden aprovechar uno o varios inquilinos de Azure AD para definir el acceso y los derechos para estos entornos. El diagrama anterior muestra un caso en el que se usan dos inquilinos diferentes de Azure AD: uno para DevOps y UAT y otro exclusivamente para producción.

La presencia de inquilinos de Azure AD diferentes refuerza la separación entre entornos. El mismo grupo de usuarios (por ejemplo, TI central) debe autenticarse con un identificador URI diferente para tener acceso a un inquilino de AD diferente y modificar los roles o permisos de los entornos de DevOps o de producción de un proyecto. La presencia de una autenticación de usuario diferente para tener acceso a diferentes entornos reduce posibles interrupciones y otros problemas causados por errores humanos.

#### <a name="component-type-infrastructure"></a>Tipo de componente: infraestructura
Este tipo de componente es en el que reside la mayor parte de la infraestructura de soporte. También es donde los equipos de TI central, seguridad y cumplimiento dedican la mayor parte de su tiempo.

[![6]][6]

Los componentes de infraestructura proporcionan una interconexión entre los distintos componentes de un centro de datos virtual y están presentes tanto en el concentrador como en los radios. Normalmente se asigna la responsabilidad de administrar y mantener los componentes de infraestructura al equipo de TI central o al de seguridad.

Una de las tareas principales del equipo de infraestructura de TI es garantizar la coherencia de los esquemas de direcciones IP en toda la empresa. El espacio de direcciones IP privadas asignado al centro de datos virtual debe ser coherente y no superponerse con direcciones IP privadas asignadas a redes locales.

Aunque el uso de NAT en los enrutadores locales perimetrales o en entornos de Azure puede evitar conflictos de direcciones IP, agrega complicaciones a los componentes de infraestructura. La simplicidad de administración es uno de los principales objetivos del centro de datos virtual, por lo que el uso de NAT para controlar cuestiones de direcciones IP no es una solución recomendada.

Los componentes de infraestructura contienen la siguiente funcionalidad:

-   [**Servicios de identidad y directorio**][AAD]. El acceso a cada tipo de recurso en Azure se controla mediante una identidad que se almacena en un servicio de directorio. El servicio de directorio almacena no solo la lista de usuarios, sino también los derechos de acceso a los recursos en una suscripción de Azure específica. Estos servicios pueden existir solo en la nube, o se pueden sincronizar con la identidad local almacenada en Active Directory.
-   [**Virtual Network**][VPN]. Las redes virtuales son uno de los componentes principales de un centro de datos virtual y le permiten crear un límite de aislamiento de tráfico en la plataforma de Azure. Una red virtual se compone de uno o varios segmentos de red virtual, cada uno con un prefijo de red IP específico (una subred). La red virtual define un área de perímetro interno en el que las máquinas virtuales de IaaS y los servicios de PaaS pueden establecer comunicaciones privadas. Las máquinas virtuales (y los servicios de PaaS) de una red virtual no pueden comunicar directamente con máquinas virtuales (y servicios de PaaS) de una red virtual diferente, incluso si ambas redes virtuales se han creado en la misma suscripción y por el mismo cliente. El aislamiento consiste en una propiedad fundamental que garantiza que las máquinas virtuales del cliente y la comunicación sigan siendo privadas en una red virtual.
-   [**UDR**][UDR]. El tráfico en una red virtual se enruta de forma predeterminada en función de la tabla de enrutamiento del sistema. Una ruta definida por el usuario (UDR) es una tabla de enrutamiento personalizada que los administradores de red pueden asociar a una o varias subredes para sobrescribir el comportamiento de la tabla de enrutamiento del sistema y definir una ruta de comunicación en una red virtual. La presencia de una UDR garantiza que el tráfico de salida del radio transita a través de máquinas virtuales personalizadas específicas o aplicaciones virtuales de red y equilibradores de carga presentes en el concentrador y los radios.
-   [**NSG**][NSG]. Un grupo de seguridad de red (NSG) es una lista de reglas de seguridad que actúa como filtro de tráfico en direcciones IP de origen y destino, protocolos y puertos IP de origen y destino. El NSG se puede aplicar a una subred, una tarjeta NIC virtual asociada a una máquina virtual de Azure, o ambos. Los NSG son fundamentales para implementar un control de flujo correcto en el concentrador y los radios. El nivel de seguridad permitido por el NSG está una función de los puertos que abra y su finalidad. Los clientes deben aplicar filtros adicionales en cada máquina virtual con firewalls basados en host, como IPtables o el Firewall de Windows.
-   [**DNS**][DNS]. La resolución de nombres de recursos en las redes virtuales de un centro de datos virtual se proporciona mediante DNS. Azure proporciona servicios DNS para la resolución de nombres tanto [DNS][pública] como [privada][PrivateDNS]. Las zonas privadas proporcionan la resolución de nombres dentro de una red virtual o en redes virtuales distintas. Puede hacer que las zonas privadas abarquen redes virtuales de la misma región, e incluso de distintas regiones y suscripciones. Azure DNS proporciona un servicio de hospedaje de dominios DNS que permite resolver nombres mediante la infraestructura de Microsoft Azure (resolución pública). Al hospedar dominios en Azure, puede administrar los registros DNS con las mismas credenciales, API, herramientas y facturación que con los demás servicios de Azure.
-   [**Administración de la suscripción**][SubMgmt] y del [**Grupo de recursos**][RGMgmt]. Una suscripción define un límite natural para crear varios grupos de recursos en Azure. Los recursos de una suscripción se ensamblan juntos en contenedores lógicos llamados grupos de recursos. El grupo de recursos representa un grupo lógico para organizar los recursos de un centro de datos virtual.
-   [**RBAC**][RBAC]. Mediante RBAC, es posible asignar roles organizativos junto con los derechos de acceso a recursos específicos de Azure, lo que le permite restringir a los usuarios a solo un subconjunto determinado de acciones. Con RBAC puede conceder acceso asignando el rol adecuado a usuarios, grupos y aplicaciones dentro del ámbito correspondiente. El ámbito de una asignación de roles puede ser una suscripción de Azure, un grupo de recursos o un único recurso. RBAC permite la herencia de permisos. Un rol asignado en un ámbito principal también concede acceso a los elementos secundarios dentro del mismo. Con RBAC podrá repartir las tareas y conceder a los usuarios únicamente el nivel de acceso que necesitan para realizar su trabajo. Por ejemplo, utilice RBAC para dejar que un empleado administre máquinas virtuales en una suscripción y que otro pueda administrar bases de datos SQL en la misma suscripción.
-   [**Emparejamiento de redes virtuales**][VNetPeering]. La característica fundamental utilizada para crear la infraestructura de un centro de datos virtual es el emparejamiento de redes virtuales, un mecanismo que conecta dos redes virtuales (VNet) de la misma región mediante la red del centro de datos de Azure o de la red troncal mundial de Azure para regiones distintas.

#### <a name="component-type-perimeter-networks"></a>Tipo de componente: redes perimetrales
Los componentes de la [Red perimetral] [ DMZ] (también conocida como una red DMZ) le permiten proporcionar conectividad de red con su redes locales y centro de datos físico, junto con cualquier tipo de conectividad hacia y desde Internet. También es donde probablemente los equipos de red y seguridad empleen la mayor parte de su tiempo.

Los paquetes entrantes deben fluir a través de los dispositivos de seguridad del concentrador, como el firewall, IDS e IPS, antes de llegar a los servidores back-end en los radios. Los paquetes enlazados a Internet desde las cargas de trabajo también deberían fluir a través de los dispositivos de seguridad de la red perimetral para fines de auditoría, inspección y aplicación de directivas antes de salir de la red.

Los componentes de la red perimetral proporcionan las siguientes características:

-   [Redes virtuales][VNet], [UDR][UDR], [NSG][NSG]
-   [Aplicación virtual de red][NVA]
-   [Equilibrador de carga][ALB]
-   [Application Gateway][AppGW] / [WAF][WAF]
-   [Direcciones IP públicas][PIP]

Normalmente, los equipos de TI central y de seguridad tienen la responsabilidad de la definición de requisitos y las operaciones de las redes perimetrales.

[![7]][7]

El diagrama anterior muestra el cumplimiento de dos perímetros con acceso a Internet y a una red local, ambos residentes en el concentrador. En un único concentrador, la red perimetral a Internet puede escalar verticalmente para admitir grandes cantidades de líneas de negocio, utilizando varias granjas de firewalls de aplicaciones web (WAF) y/o firewalls.

[**Redes virtuales**][VNet] El concentrador normalmente se basa en una red virtual con varias subredes para hospedar los distintos tipos de servicios con filtrado e inspección del tráfico hacia o desde Internet utilizando NVA, WAF y Azure Application Gateway.

[**UDR**][UDR] Con UDR, los clientes pueden implementar un firewall, IDS/IPS y otras aplicaciones virtuales y enrutar el tráfico de red a través de estas aplicaciones de seguridad para la aplicación de directivas de límites de seguridad, auditoría e inspección. Las UDR pueden crearse en el concentrador y los radios para garantizar que el tráfico transita a través de máquinas virtuales personalizadas específicas, aplicaciones virtuales de red y equilibradores de carga utilizados en el centro de datos virtual. Para garantizar que el tráfico generado desde máquinas virtuales residentes en el radio transita a las aplicaciones virtuales correctas, se debe establecer una UDR en las subredes del radio configurando la dirección IP del front-end del equilibrador de carga interno como el próximo salto. El equilibrador de carga interno distribuye el tráfico interno a las aplicaciones virtuales (grupo de back-end de equilibradores de carga).

[![8]][8]

[**Aplicaciones virtuales de red**][NVA] En el concentrador, la red perimetral con acceso a Internet se administra normalmente a través de una granja de firewalls o firewalls de aplicaciones web (WAF).

Las diferentes líneas de negocio normalmente utilizan diversas aplicaciones web y estas aplicaciones tienden a sufrir varias vulnerabilidades y puntos débiles potenciales. Los firewall de aplicaciones web son un tipo especial de producto que se usa para detectar ataques contra las aplicaciones web (HTTP/HTTPS) con mayor profundidad que un firewall genérico. En comparación con la tecnología de firewall tradicional, los WAF tienen un conjunto de características específicas para proteger los servidores web internos frente a amenazas.

Una granja de firewalls es un grupo de firewalls que trabajan conjuntamente bajo una administración común, con un conjunto de reglas de seguridad para proteger las cargas de trabajo hospedadas en los radios y controlar el acceso a las redes locales. Una granja de firewalls tiene menos software especializado en comparación con un WAF, pero tiene un ámbito más amplio de aplicación para filtrar e inspeccionar cualquier tipo de tráfico de entrada y salida. Las granjas de firewalls se implementan normalmente en Azure a través de aplicaciones virtuales de red (NVA), que están disponibles en Azure Marketplace.

Se recomienda usar un conjunto de NVA para el tráfico que se origina en Internet y otro para el tráfico que se origina en local. Utilizar únicamente un conjunto de NVA para ambos es un riesgo de seguridad, ya que no proporciona ningún perímetro de seguridad entre los dos conjuntos de tráfico de red. El uso de NVA independientes reduce la complejidad de la comprobación de las reglas de seguridad y deja claro qué reglas se corresponden con cada solicitud de red entrante.

La mayoría de las empresas de tamaño grande administran varios dominios. El servicio DNS de Azure se puede usar para hospedar los registros DNS de un dominio concreto. Como ejemplo, se puede registrar la dirección IP virtual (VIP) del equilibrador de carga externo de Azure (o de los WAF) en el registro A de un registro DNS de Azure.

[**Azure Load Balancer**][ALB] El equilibrador de carga de Azure ofrece un servicio de nivel 4 (TCP, UDP) de alta disponibilidad que puede distribuir el tráfico entrante entre instancias del servicio definidas en un conjunto de equilibrio de carga. El tráfico enviado al equilibrador de carga desde los puntos de conexión front-end (puntos de conexión de direcciones IP públicas o privadas) se puede redistribuir con o sin traducción de direcciones a un conjunto de grupo de direcciones IP de back-end (ejemplo: aplicaciones virtuales de red o máquinas virtuales).

Azure Load Balancer puede sondear el estado de las diversas instancias de servidor y, cuando un sondeo no responde, el equilibrador de carga deja de enviar tráfico a la instancia incorrecta. En un centro de datos virtual, tenemos la presencia de un equilibrador de carga externo en el concentrador (por ejemplo, para equilibrar el tráfico a los NVA) y en los radios (para realizar tareas como equilibrar el tráfico entre diferentes máquinas virtuales de una aplicación de varios niveles).

[**Application Gateway**][AppGW] Microsoft Azure Application Gateway es una aplicación virtual dedicada que cuenta con un controlador de entrega de aplicaciones (ADC) que se ofrece como servicio y que proporciona numerosas funcionalidades de equilibrio de carga de nivel 7 para la aplicación. Le permite optimizar la productividad de las granjas de servidores web traspasando la carga de la terminación SSL con mayor actividad de la CPU a Application Gateway. Además, dispone de otras funcionalidades de enrutamiento de nivel 7, como la distribución round robin del tráfico entrante, la afinidad de sesiones basada en cookies, el enrutamiento basado en rutas de acceso URL y la capacidad de hospedar varios sitios web detrás de una única puerta de enlace de aplicaciones. También se proporciona un firewall de aplicaciones web (WAF) como parte de la SKU de WAF de Application Gateway. Esta SKU proporciona protección a las aplicaciones web frente a vulnerabilidades web y vulnerabilidades de seguridad comunes. Application Gateway puede configurarse como una puerta de enlace orientada a Internet, una puerta de enlace solo para uso interno o una combinación de las dos. 

[**Direcciones IP públicas**][PIP] Algunas características de Azure le permiten asociar los puntos de conexión de servicio a una dirección IP pública que permite al recurso estar accesible desde Internet. Este punto de conexión usa la traducción de direcciones de red (NAT) para enrutar el tráfico a la dirección interna y el puerto de la red virtual de Azure. Esta ruta es la vía principal para que el tráfico externo pase a la red virtual. Las direcciones IP públicas se pueden configurar para determinar qué tráfico se pasa y cómo y dónde se traduce en la red virtual.

#### <a name="component-type-monitoring"></a>Tipo de componente: supervisión
Los componentes de supervisión proporcionan visibilidad y alertas sobre todos los demás tipos de componentes. Todos los equipos deben tener acceso a la supervisión de los componentes y servicios a los que tienen acceso. Si tiene equipos de soporte técnico o de operaciones centralizados, necesitarán un acceso integrado a los datos proporcionados por estos componentes.

Azure ofrece diferentes tipos de servicios de registro y supervisión para realizar un seguimiento del comportamiento de los recursos hospedados en Azure. La regulación y el control de las cargas de trabajo en Azure no se basa solo en recopilar datos de registro, sino también en la posibilidad de desencadenar acciones basándose en eventos notificados específicos.

[**Azure Monitor**][Monitor]: Azure incluye varios servicios que realizan individualmente una tarea o un rol específico en el espacio de supervisión. Juntos, estos servicios ofrecen una solución completa para recopilar, analizar y actuar en la telemetría de la aplicación y los recursos de Azure que las admiten. También pueden servir para supervisar recursos locales críticos, a fin de proporcionar un entorno de supervisión híbrido. Conocer las herramientas y los datos que están disponibles es el primer paso para desarrollar una estrategia de supervisión completa para la aplicación.

Hay dos tipos principales de registros en Azure:

-   Los [**Registros de actividad** ] [ ActLog] (conocido también como "Registro operativo") proporcionan una visión general de las operaciones realizadas en recursos de la suscripción de Azure. Estos registros informan sobre los eventos en el plano de control de las suscripciones. Todos los recursos de Azure generan registros de auditoría.

-   Los [**registros de diagnóstico de Azure**][DiagLog] son registros generados por un recurso que proporcionan datos exhaustivos y frecuentes acerca del funcionamiento de ese recurso. El contenido de estos registros varía según el tipo de recurso.

[![9]][9]

En un centro de datos virtual es muy importante realizar el seguimiento de los registros de los grupos de seguridad de red, especialmente esta información:

-   [**Registros de eventos**][NSGLog]: proporcionan información sobre qué reglas de grupos de seguridad de red se aplican a las máquinas virtuales y a los roles de instancia en función de la dirección MAC.
-   [**Registros de contador**][NSGLog]: realizan el seguimiento sobre cuántas veces se aplica cada regla de los grupos de seguridad de red para denegar o permitir el tráfico.

Todos los registros se pueden almacenar en cuentas de Azure Storage con fines de auditoría, análisis estático o copia de seguridad. Cuando los registros se almacenan en una cuenta de Azure Storage, los clientes pueden usar diferentes tipos de marcos de trabajo para recuperar, preparar, analizar y visualizar estos datos para notificar el estado de los recursos en la nube.

Las grandes empresas pueden haber adquirido previamente un marco estándar para la supervisión de los sistemas locales y pueden ampliar ese marco de trabajo para integrar los registros generados por las implementaciones en la nube. Para las organizaciones que deseen mantener todo el registro en la nube, [Log Analytics][../log-analytics/log-analytics-overview .md] es una excelente opción. Puesto que Log Analytics se implementa como un servicio basado en la nube, puede hacer que funcione rápidamente con una inversión mínima en servicios de infraestructura. Log Analytics puede también integrarse con componentes de System Center, como System Center Operations Manager para ampliar sus inversiones existentes de administración en la nube.

Log Analytics es un servicio de Azure que ayuda a recopilar, correlacionar, buscar y actuar en los datos de registro y rendimiento generados por los sistemas operativos, aplicaciones y componentes de infraestructura en la nube. Ofrece a los clientes una visión operativa en tiempo real mediante la búsqueda integrada y los paneles personalizados para analizar todos los registros en todas las cargas de trabajo del centro de datos virtual.

La solución [Network Performance Monitor (NPM)][NPM] de OMS proporciona información de un extremo a otro de la red, incluida una vista única de las redes de Azure y las locales. Con monitores específicos para los servicios públicos y ExpressRoute.

#### <a name="component-type-workloads"></a>Tipo de componente: cargas de trabajo
Los componentes de carga de trabajo son donde residen los servicios y aplicaciones reales. También es donde los equipos de desarrollo de aplicaciones dedican la mayor parte de su tiempo.

Las posibilidades de cargas de trabajo son realmente infinitas. Estos son solo algunos de los tipos posibles de cargas de trabajo:

**Aplicaciones de líneas de negocio internas**. Las aplicaciones de línea de negocio son aplicaciones informáticas necesarias para la operación en curso de una empresa. Las aplicaciones de línea de negocio tienen algunas características comunes:

-   **Interactividad**. Las aplicaciones de línea de negocio son interactivas por naturaleza: los datos se captan y se devuelven resultados e informes.
-   **Controladas por datos**. Las aplicaciones de línea de negocio son intensivas en el uso de datos, con acceso frecuente a bases de datos o a otro almacenamiento de datos.
-   **Integradas**. Las aplicaciones de línea de negocio ofrecen integración con otros sistemas dentro y fuera de la organización.

**Sitios web accesibles para clientes (accesibles desde Internet o internos)**. La mayoría de las aplicaciones que interactúan con Internet son sitios web. Azure ofrece la funcionalidad de ejecutar un sitio web en una máquina virtual de IaaS o desde un sitio de [Azure Web Apps] [ WebApps] (PaaS). Azure Web Apps admite la integración con redes virtuales que permite implementar las aplicaciones web en los radios de un centro de datos virtual. Al observar los sitios web de orientación interna, con la integración con redes virtuales no es necesario exponer un punto de conexión en Internet para las aplicaciones, sino que puede usar en su lugar los recursos a través de direcciones no enrutables a Internet privadas desde la red virtual privada.

**Big Data y análisis** cuando necesita escalar verticalmente los datos hasta un volumen muy grande, las bases de datos pueden no escalar correctamente. La tecnología de Hadoop ofrece un sistema para ejecutar consultas distribuidas en paralelo en un gran número de nodos. Los clientes tienen la opción de ejecutar las cargas de trabajo de datos en máquinas virtuales IaaS o bien en PaaS ([HDInsight][HDI]). HDInsight admite la implementación en una red virtual basada en la ubicación y se puede implementar en un clúster en un radio del centro de datos virtual.

**Eventos y mensajería**. [Azure Event Hubs][EventHubs] es un servicio de ingestión de datos de telemetría a hiperescala que recopila, transforma y almacena millones de eventos. Como plataforma de streaming distribuida, ofrece retención de tiempo configurable y baja latencia, lo que permite recopilar grandes cantidades de datos de telemetría en Azure y leer los datos desde varias aplicaciones. Con Event Hubs, una única transmisión puede admitir canalizaciones en tiempo real y basadas en lotes.

Se puede implementar un servicio de mensajería en la nube altamente confiable entre aplicaciones y servicios con [Azure Service Bus][ServiceBus] que ofrece mensajería asincrónica entre cliente y servidor, junto con mensajería de estructura primero en entrar primero en salir (FIFO) y funcionalidades de publicación y suscripción.

[![10]][10]

### <a name="multiple-vdc"></a>Múltiples centros de datos virtuales
Hasta ahora, este artículo se ha centrado en un único centro de datos virtual y la descripción de los componentes básicos y arquitectura que contribuyen a un centro de datos virtual resistente. Características de Azure como el equilibrador de carga de Azure, las aplicaciones virtuales de red, los conjuntos de disponibilidad, los conjuntos de escala y otros mecanismos contribuyen a un sistema que le permite crear niveles de SLA sólidos en los servicios de producción.

Sin embargo, un único centro de datos virtual se hospeda en una única región y es vulnerable a una interrupción importante que pudiera afectar a toda esa región. Los clientes que desean lograr altos niveles de SLA necesitan proteger los servicios a través de implementaciones del mismo proyecto en dos (o más) centros de datos virtuales, ubicados en regiones diferentes.

Además de aspectos de SLA, hay varios escenarios comunes en los que tiene sentido implementar varios centros de datos virtuales:

-   Presencia regional y global
-   Recuperación ante desastres
-   Mecanismo para desviar el tráfico entre centros de datos

#### <a name="regionalglobal-presence"></a>Presencia regional y global
Los centros de datos de Azure están presentes en varias regiones de todo el mundo. Al seleccionar varios centros de datos de Azure, los clientes deben tener en cuenta dos factores relacionados: distancias geográficas y latencia. Los clientes necesitan evaluar la distancia geográfica entre los centros de datos virtuales y la distancia entre los centros de datos virtuales y los usuarios finales para ofrecer la mejor experiencia de usuario.

La región de Azure donde se hospedan los centros de datos virtuales también debe cumplir con los requisitos de regulación establecidos por cualquier jurisdicción en la que opere la organización.

#### <a name="disaster-recovery"></a>Recuperación ante desastres
La implementación de un plan de recuperación ante desastres está muy relacionada con el tipo de carga de trabajo del que se trate y la capacidad de sincronizar el estado de la carga de trabajo entre diferentes centros de datos virtuales. De un modo ideal, la mayoría de los clientes desea sincronizar los datos de aplicación entre dos implementaciones que se ejecutan en dos centros de datos virtuales diferentes para implementar un mecanismo de conmutación por error rápido. La mayoría de las aplicaciones es sensible a la latencia, lo que puede provocar un posible tiempo de espera y retraso en la sincronización de los datos.

La sincronización o la supervisión de latido de aplicaciones en diferentes centros de datos virtuales exige comunicación entre ellos. Dos centros de datos virtuales en regiones diferentes pueden estar conectados a través de:

-   Emparejamiento de redes virtuales: permite la conexión de centros de diferentes regiones
-   Emparejamiento privado de ExpressRoute cuando los concentradores de los centros de datos virtuales están conectados al mismo circuito de ExpressRoute
-   Varios circuitos de ExpressRoute conectados a través de la red troncal corporativa y la malla de centros de datos virtuales conectada a circuitos de ExpressRoute
-   Conexiones VPN de sitio a sitio entre los concentradores de los centros de datos virtuales en cada región de Azure

Normalmente se prefiere el mecanismo de conexión ExpressRoute o de enrutamiento de redes virtuales por su mayor ancho de banda y latencia coherente al transitar por la red troncal de Microsoft.

No hay ninguna receta mágica para validar una aplicación distribuida entre dos (o más) centros de datos virtuales diferentes ubicados en regiones diferentes. Los clientes deben ejecutar pruebas de calificación de red para comprobar la latencia y el ancho de banda de las conexiones y decidir si la replicación de datos sincrónica o asincrónica es adecuada y cuál puede ser el objetivo de tiempo de recuperación óptimo (RTO) para las cargas de trabajo.

#### <a name="mechanism-to-divert-traffic-between-dc"></a>Mecanismo para desviar el tráfico entre centros de datos
Una técnica eficaz para desviar el tráfico entrante en un centro de datos a otro se basa en DNS. [Azure Traffic Manager][TM] utiliza el mecanismo de sistema de nombres de dominio (DNS) para dirigir el tráfico del usuario final hacia el punto de conexión público más adecuado en un centro de datos específico. Mediante sondeos, Traffic Manager comprueba periódicamente el estado del servicio de los puntos de conexión públicos en centros de datos virtuales diferentes y, en caso de error en esos puntos de conexión, enruta automáticamente al centro de datos virtual secundario.

Traffic Manager funciona en puntos de conexión públicos de Azure y puede utilizarse, por ejemplo, para controlar y desviar el tráfico a Web Apps y Azure Virtual Machines en el centro de datos virtual adecuado. Traffic Manager es resistente incluso al enfrentarse un error de una región de Azure completa y puede controlar la distribución del tráfico de usuario hacia puntos de conexión de servicio en centros de datos virtuales diferentes en función de varios criterios (por ejemplo, error de un servicio en un centro de datos virtual específico o seleccionar el centro de datos virtual con la menor latencia de red para el cliente).

### <a name="conclusion"></a>Conclusión
El centro de datos virtual es un enfoque para la migración del centro de datos a la nube que utiliza una combinación de características y funcionalidades para crear una arquitectura escalable de Azure que maximiza el uso de recursos en la nube, se reducen los costos y se simplifica la regulación de sistema. El concepto de centro de datos virtual se basa en una topología de concentrador y radios que proporciona servicios compartidos comunes en el concentrador y permite aplicaciones y cargas de trabajo específicas en los radios. El centro de datos virtual coincide con la estructura de roles de la empresa, donde diferentes departamentos (TI Central, DevOps, operaciones y mantenimiento) funcionan conjuntamente, cada uno con una lista específica de roles y tareas. El centro de datos virtual cumple los requisitos de una migración "levantar y mover", pero también proporciona muchas ventajas para las implementaciones nativas en la nube.

## <a name="references"></a>Referencias
Las siguientes características se describen en este documento. Haga clic en los vínculos para más información.

| | | |
|-|-|-|
|Características de red|Equilibrio de carga|Conectividad|
|[Redes virtuales de Azure][VNet]</br>[Grupos de seguridad de red][NSG]</br>[Registros NSG][NSGLog]</br>[Enrutamiento definido por el usuario][UDR]</br>[Aplicaciones virtuales de red][NVA]</br>[Direcciones IP públicas][PIP]</br>[DNS]|[Azure Load Balancer (L3) ][ALB]</br>[Application Gateway (L7) ][AppGW]</br>[Firewall de aplicaciones web][WAF]</br>[Azure Traffic Manager][TM] |[Emparejamiento de redes virtuales][VNetPeering]</br>[Red privada virtual][VPN]</br>[ExpressRoute][ExR]
|Identidad</br>|Supervisión</br>|Prácticas recomendadas</br>|
|[Azure Active Directory][AAD]</br>[Multi-Factor Authentication][MFA]</br>[Control de acceso basado en roles][RBAC]</br>[Roles predeterminados de AAD][Roles] |[Azure Monitor][Monitor]</br>[Registros de actividad][ActLog]</br>[Registros de diagnóstico][DiagLog]</br>[Microsoft Operations Management Suite][OMS]</br>[Network Performance Monitor][NPM]|[Procedimientos recomendados en redes perimetrales][DMZ]</br>[Administración de suscripciones][SubMgmt]</br>[Administración de grupos de recursos][RGMgmt]</br>[Límites de la suscripción de Azure][Limits] |
|Otros servicios de Azure|
|[Azure Web Apps][WebApps]</br>[HDInsights (Hadoop) ][HDI]</br>[Event Hubs][EventHubs]</br>[Service Bus][ServiceBus]|



## <a name="next-steps"></a>Pasos siguientes
 - Explore [Emparejamiento de redes virtuales][VNetPeering], la tecnología subyacente a los diseños de concentrador y radio del centro de datos virtual
 - Implemente [AAD][AAD] para empezar a trabajar con la exploración de [RBAC][RBAC]
 - Desarrolle un modelo para la administración de los recursos y suscripciones, y un modelo de RBAC para satisfacer la estructura, requisitos y directivas de su organización. La actividad más importante es el planeamiento. Siempre que resulte práctico, planee reorganizaciones, fusiones, nuevas líneas de producto, etc.

<!--Image References-->
[0]: ./images/networking-redundant-equipment.png "Ejemplos de superposición de componentes" 
[1]: ./images/networking-vdc-high-level.png "Ejemplo genérico de centro de datos virtual con concentrador y radios"
[2]: ./images/networking-hub-spokes-cluster.png "Clúster de concentradores y radios"
[3]: ./images/networking-spoke-to-spoke.png "Radio a radio"
[4]: ./images/networking-vdc-block-level-diagram.png "Diagrama de nivel de bloques del centro de datos virtual"
[5]: ./images/networking-users-groups-subsciptions.png "Usuarios, grupos, suscripciones y proyectos"
[6]: ./images/networking-infrastructure-high-level.png "Diagrama genérico de infraestructura"
[7]: ./images/networking-highlevel-perimeter-networks.png "Diagrama genérico de infraestructura"
[8]: ./images/networking-vnet-peering-perimeter-neworks.png "Emparejamiento de VNET y redes perimetrales"
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
[NVA]: /azure/architecture/reference-architectures/dmz/nva-ha
[SubMgmt]: /azure/architecture/cloud-adoption/appendix/azure-scaffold 
[RGMgmt]: /azure/azure-resource-manager/resource-group-overview
[DMZ]: /azure/best-practices-network-security
[ALB]: /azure/load-balancer/load-balancer-overview
[PIP]: /azure/virtual-network/resource-groups-networking#public-ip-address
[AppGW]: /azure/application-gateway/application-gateway-introduction
[WAF]: /azure/application-gateway/application-gateway-web-application-firewall-overview
[Monitor]: /azure/monitoring-and-diagnostics/
[ActLog]: /azure/monitoring-and-diagnostics/monitoring-overview-activity-logs 
[DiagLog]: /azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs
[NSGLog]: /azure/virtual-network/virtual-network-nsg-manage-log
[OMS]: /azure/operations-management-suite/operations-management-suite-overview
[NPM]: /azure/log-analytics/log-analytics-network-performance-monitor
[WebApps]: /azure/app-service/
[HDI]: /azure/hdinsight/hdinsight-hadoop-introduction
[EventHubs]: /azure/event-hubs/event-hubs-what-is-event-hubs 
[ServiceBus]: /azure/service-bus-messaging/service-bus-messaging-overview
[TM]: /azure/traffic-manager/traffic-manager-overview
