---
title: Prácticas recomendadas para empresas que migran su infraestructura a Azure
description: En este artículo se describe una plantilla scaffold que las empresas pueden utilizar para garantizar que el entorno sea seguro y fácil de administrar.
author: rdendtler
ms.author: rodend
ms.date: 9/22/2018
ms.openlocfilehash: a1962a3d5be79715de4781d955b6be1db5923bea
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/28/2018
ms.locfileid: "47429305"
---
# <a name="azure-enterprise-scaffold-prescriptive-subscription-governance"></a>Scaffolding empresarial de Azure: gobierno de suscripción prescriptivo

Cada vez son más las empresas que adoptan la tecnología de nube pública para ganar agilidad y flexibilidad. Aprovechan los puntos fuertes de la nube para generar ingresos y optimizar el uso de los recursos de la empresa. Microsoft Azure proporciona un gran número de servicios y funcionalidades que las empresas ensamblan como bloques de creación con el objetivo de abordar diversas aplicaciones y cargas de trabajo.

Decidirse a usar Microsoft Azure es solo el primer paso para poder beneficiarse de la ventaja de la nube. El segundo paso es comprender cómo la empresa puede usar Azure con eficacia e identificar las funcionalidades básicas que deben existir para responder preguntas como:

* "Me preocupa la soberanía de datos; ¿cómo puedo garantizar que los datos y sistemas cumplan nuestros requisitos normativos?"
* "¿Cómo puedo saber lo que cada recurso admite para poder tenerlo en cuenta y realizar la facturación de forma adecuada?"
* "Quiero asegurarme de que todo lo que se implementa o hace en la nube pública se ponga en marcha pensando en todo momento en la seguridad como máxima prioridad; ¿cómo puedo facilitar que así sea?"

La posibilidad de que existan suscripciones vacías sin ninguna protección es desalentadora. Este espacio en blanco puede dificultar la migración a Azure.

En este artículo sirve como punto de partida para que los profesionales técnicos satisfagan la necesidad de contar con un sistema de gobierno y la equilibren con el imperativo de ganar agilidad. Además, se presenta el concepto de scaffolding empresarial, que guía a las organizaciones en la implementación y administración de los entornos de Azure de forma segura. Proporciona el marco de trabajo para desarrollar controles eficaces y eficientes.

## <a name="need-for-governance"></a>Necesidad de contar con un sistema de gobierno

Al migrar la infraestructura a Azure, hay que abordar el asunto del sistema de gobierno en la primera fase para garantizar que la nube se utilice correctamente en la empresa. Desafortunadamente, debido al tiempo que se tarda en crear un sistema de gobierno completo y toda la administración que conlleva, algunos grupos de negocios recurren directamente a los proveedores sin contar con su equipo de TI. Este enfoque puede dejar a la empresa expuesta a vulnerabilidades si los recursos no se administran correctamente. Las características de la nube pública —agilidad, flexibilidad y precios basados en el consumo— son importantes para los grupos de negocios que necesitan satisfacer rápidamente las necesidades de los clientes (internos y externos). Sin embargo, el equipo de TI de la empresa debe garantizar que los datos y sistemas estén protegidos de forma eficaz.

Al realizar una compilación, se utiliza la técnica scaffolding para crear la base de una estructura. Las plantillas scaffold sirven de guía para el esquema general y proporcionan puntos de anclaje para poder montar sistemas más permanentes. Las scaffold empresariales son lo mismo: una serie de controles flexibles y funcionalidades de Azure que proporcionan la estructura al entorno, así como delimitadores para los servicios basados en la nube pública. Además, brindan a los fabricantes (grupos de negocio y departamentos de TI) los cimientos para crear y asociar nuevos servicios, teniendo en cuenta la velocidad de entrega.

Estas plantillas se basan en las prácticas que hemos recopilado de muchas interacciones con clientes de varios tamaños. Desde pequeñas organizaciones que desarrollan soluciones en la nube para grandes empresas internacionales hasta fabricantes de software independientes que van a migrar cargas de trabajo y desarrollar soluciones nativas de la nube. Las plantillas scaffold empresariales se han creado con la suficiente flexibilidad como para admitir las cargas de trabajo de TI tradicionales y las del método ágil; por ejemplo, los desarrolladores que crean aplicaciones de software como servicio (SaaS) basadas en las funcionalidades de la plataforma Azure.

Asimismo, están diseñadas para ser el pilar de todas las suscripciones nuevas de Azure. Estas plantillas permiten a los administradores asegurarse de que las cargas de trabajo cumplan los requisitos mínimos de gobierno de una organización sin impedir que los grupos de negocios y los desarrolladores cumplan rápidamente sus objetivos. Nuestra experiencia demuestra que esto agiliza significativamente el crecimiento de la nube pública, en lugar de obstaculizarlo.

> [!NOTE]
> Microsoft ha publicado en versión preliminar una nueva funcionalidad denominada [Azure Blueprint](/azure/governance/blueprints/overview) que le permitirá empaquetar, administrar e implementar imágenes, plantillas, directivas y scripts comunes en las suscripciones y grupos de administración. Esta funcionalidad es el puente entre el propósito de la plantilla scaffold como modelo de referencia y la implementación de ese modelo en la organización.
>
En la imagen siguiente se muestran los componentes de la plantilla scaffold. La base se sustenta en un plan sólido para la administración de la jerarquía y las suscripciones. Los pilares constan de directivas de Resource Manager y de estándares de nomenclatura eficaces. El resto de los componentes de scaffolding son las principales funcionalidades y características de Azure que habilitan y conectan un entorno seguro y fácil de administrar.

![Scaffold de Enterprise](./_images/scaffoldv2.png)

## <a name="define-your-hierarchy"></a>Definición de la jerarquía

La base de una plantilla scaffold es la jerarquía y la relación de la inscripción Enterprise de Azure a través de suscripciones y grupos de recursos. La inscripción Enterprise define la forma y el uso de los servicios de Azure en la empresa desde un punto de vista contractual. En el contrato Enterprise, puede subdividir el entorno en departamentos, cuentas y, finalmente, suscripciones y grupos de recursos para que coincidan con la estructura de la organización.

![Jerarquía](./_images/agreement.png)

Una suscripción de Azure es la unidad básica donde se encuentran todos los recursos. También define varios límites en Azure, como el número de núcleos, redes virtuales y otros recursos. Los grupos de recursos de Azure se utilizan para perfeccionar más el modelo de suscripción y habilitar una agrupación de recursos más natural.

Cada empresa es diferente, y la jerarquía de la imagen anterior permite una gran flexibilidad en lo que respecta a cómo se organiza Azure en la empresa. El modelado de la jerarquía para reflejar las necesidades de su empresa en cuanto a facturación, administración de recursos y acceso a los recursos es la primera decisión, y la más importante, que se debe tomar para empezar a trabajar en la nube pública.

### <a name="departments-and-accounts"></a>Departamentos y cuentas

Los tres patrones comunes de las inscripciones de Azure son los siguientes:

* El patrón **funcional**

    ![functional](./_images/functional.png)
* El patrón de **unidad de negocio**

    ![Patrón de unidad de negocio](./_images/business.png)
* El patrón **geográfico**

    ![Patrón geográfico](./_images/geographic.png)

Aunque cada uno de estos patrones tiene su sitio, cada vez se adopta más el patrón de la **unidad de negocio** por su flexibilidad para dar forma al modelo de costos de una organización y para reflejar el ámbito de control. El grupo de operaciones e ingeniería principal de Microsoft ha creado un subconjunto del patrón de la **unidad de negocio** que resulta muy eficaz, cuyo modelado se realiza a nivel **federal**, **estatal** y **local**. Para más información, vea [Organizing subscriptions and resource groups within the Enterprise](https://azure.microsoft.com/blog/organizing-subscriptions-and-resource-groups-within-the-enterprise/) (Organización de suscripciones y grupos de recursos en la empresa).

### <a name="management-groups"></a>Grupos de administración

Microsoft ha lanzado recientemente una nueva forma de modelar la jerarquía: [grupos de administración de Azure](/azure/azure-resource-manager/management-groups-overview). Los grupos de administración son mucho más flexibles que los departamentos y las cuentas y se pueden anidar hasta seis niveles. Los grupos de administración permiten crear una jerarquía que es independiente de la jerarquía de facturación, únicamente para una administración eficaz de los recursos. Los grupos de administración pueden reflejar la jerarquía de facturación, y las empresas suelen empezar de esta forma. Sin embargo, la eficacia de los grupos de administración se observa al usarlos para modelar la organización, cuando las suscripciones relacionadas, con independencia de que se encuentren en la jerarquía de facturación, se agrupan y necesitan tener asignados roles comunes, además de directivas e iniciativas. Ejemplos:

* **Producción/No producción**. Algunas empresas crean grupos de administración para identificar sus suscripciones de producción y las que no son de producción. Los grupos de administración permiten a estos clientes administrar con más facilidad los roles y las directivas; por ejemplo: la suscripción que no es de producción puede conceder a los desarrolladores acceso como "colaborador", mientras que, en producción, solo disponen de acceso de "lectura".
* **Servicios internos/servicios externos**. Al igual que sucede con los escenarios de producción/no producción, las empresas suelen tener requisitos, directivas y roles distintos para los servicios internos en comparación con los externos, estos últimos orientados al cliente.

Los grupos de administración bien diseñados son, junto con las iniciativas y las directivas de Azure, la columna vertebral de un gobierno eficaz de Azure.

### <a name="subscriptions"></a>Suscripciones

A la hora de tomar decisiones sobre departamentos, cuentas o grupos de administración, se interesa principalmente en cómo dividir el entorno de Azure para adaptarlo a su organización. Sin embargo, las decisiones relativas a las suscripciones son más importantes, ya que repercuten en la seguridad, la escalabilidad y la facturación.  Muchas organizaciones usan los siguientes patrones como referencia:

* **Aplicación o servicio**: las suscripciones representan una aplicación o un servicio (cartera de aplicaciones).
* **Ciclo de vida**: las suscripciones representan el ciclo de vida de un servicio, como producción o desarrollo.
* **Departamento**: las suscripciones representan los departamentos de la organización.

Los dos primeros patrones son los más usados y ambos son muy recomendables. El enfoque del ciclo de vida es adecuado para la mayoría de las organizaciones. En este caso, la recomendación general es usar dos suscripciones de base. "Producción" y "No producción" y luego usar grupos de recursos para desglosar aún más los entornos.

### <a name="resource-groups"></a>Grupos de recursos

Azure Resource Manager permite colocar recursos en grupos significativos de administración, facturación o afinidad natural. Los grupos de recursos son contenedores de recursos que tienen un ciclo de vida común o comparten un atributo como "Todos los servidores SQL Server" o "Aplicación A".

Los grupos de recursos no pueden anidarse, y los recursos solo pueden pertenecer a un solo grupo de recursos. Algunas acciones pueden aplicarse a todos los recursos de un grupo de recursos. Por ejemplo, al eliminar un grupo de recursos, se quitan todos los recursos del grupo de recursos. Al igual que sucede con las suscripciones, existen patrones comunes al crear grupos de recursos, que variarán de las cargas de trabajo de "TI tradicional" a las cargas de trabajo de "TI ágil":

* Las cargas de trabajo de "TI tradicional" se suelen agrupar por elementos dentro del mismo ciclo de vida, como una aplicación. Si se agrupan las aplicaciones, se podrán administrar las aplicaciones de forma individual.
* Las cargas de trabajo de TI o del método ágil tienden a centrarse en las aplicaciones en la nube orientadas a los clientes externos. Los grupos de recursos suelen reflejar las capas de implementación (por ejemplo, el nivel web y el de aplicaciones) y administración.

> [!NOTE]
> La comprensión de la carga de trabajo lo ayudará a desarrollar una estrategia de grupo de recursos. Estos patrones se pueden mezclar y asociar. Por ejemplo, un grupo de recursos de servicios compartidos en la misma suscripción como grupos de recursos "ágiles".

## <a name="naming-standards"></a>Estándares de nomenclatura

El primer pilar de una plantilla scaffold es un estándar de nomenclatura coherente. Cuando se diseñan bien, se pueden identificar los recursos en el portal, en una factura y en los scripts. Puede que ya disponga de estándares de nomenclatura para la infraestructura local. Al agregar Azure a su entorno, debe extender estos estándares de nomenclatura a los recursos de Azure.

> [!TIP]
> Convenciones de nomenclatura:
> * Revise y adopte siempre que sea posible esta [guía de patrones y prácticas](https://docs.microsoft.com/en-us/azure/architecture/best-practices/naming-conventions), que lo ayudará a decidirse por un estándar de nomenclatura significativo e ilustrará ejemplos amplios.
> * Uso de las directivas de Resource Manager para ayudar a reforzar los estándares de nomenclatura
>
>Recuerde que es difícil cambiar los nombres más adelante, así que, si dedica ahora unos minutos, se ahorrará problemas más tarde.

Centre los estándares de nomenclatura en aquellos recursos que suelen usarse y buscarse con más frecuencia.  Por ejemplo, todos los grupos de recursos deben respetar un estricto estándar de claridad.

### <a name="resource-tags"></a>Etiquetas del recurso

Las etiquetas de los recursos están estrechamente vinculadas con los estándares de nomenclatura. A medida que se agregan recursos a las suscripciones, cada vez resulta más importante clasificarlos de forma lógica para fines de facturación, administración y operativos. Para más información, vea [Uso de etiquetas para organizar los recursos de Azure](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-using-tags).

> [!IMPORTANT]
> Las etiquetas pueden contener información personal y pueden estar sujetas a las disposiciones del RGPD. Planee la administración de las etiquetas detenidamente. Si desea obtener información general sobre el RGPD, consulte la [sección sobre el RGPD del Portal de confianza de servicios](https://servicetrust.microsoft.com/ViewPage/GDPRGetStarted).

En muchos aspectos, las etiquetas se usan más allá de la facturación y la administración. A menudo se usan como parte de la automatización (consulte la sección siguiente). Esto puede ocasionar conflictos si no se considera por adelantado. La práctica recomendada consiste en identificar todas las etiquetas comunes a nivel de empresa (por ejemplo, ApplicationOwner o CostCenter) y aplicarlas de forma coherente al implementar recursos mediante automatización.

## <a name="azure-policy-and-initiatives"></a>Iniciativas y directivas de Azure

El segundo pilar implica el uso de [iniciativas y directivas de Azure](/azure/azure-policy/azure-policy-introduction) para administrar el riesgo con la aplicación de reglas (con efectos) en los recursos y servicios de las suscripciones. Las iniciativas de Azure son colecciones de directivas diseñadas para lograr un objetivo único. Las iniciativas y directivas de Azure se asignan posteriormente a un ámbito de recursos para la aplicación de directivas concretas.
<IMG de iniciativas/directivas/asignaciones>

Las iniciativas y directivas de Azure son aún más eficaces cuando se usan con los grupos de administración mencionados anteriormente. Los grupos de administración permiten la asignación de una iniciativa o directiva a un conjunto completo de suscripciones.

### <a name="common-uses-of-resource-manager-policies"></a>Usos comunes de las directivas de Resource Manager

Las iniciativas y directivas de Azure son una herramienta eficaz del kit de herramientas de Azure. Las directivas permiten a las empresas proporcionar controles para las cargas de trabajo de "TI tradicional" que aportan la estabilidad necesaria para las aplicaciones de línea de negocio, además de permitir las cargas de trabajo "ágiles", como desarrollar aplicaciones de cliente sin tener que exponer a la empresa a riesgos adicionales. Los patrones más comunes que se pueden encontrar para las directivas son los siguientes:

* **Soberanía de datos y cumplimiento geográfico**. Azure tiene una lista creciente de regiones en todo el mundo. Las empresas a menudo necesitan asegurarse de que los recursos de un ámbito concreto permanecen en una región geográfica para abordar los requisitos normativos.
* **Evite exponer los servidores públicamente**. Una directiva de Azure puede prohibir la implementación de determinados tipos de recursos. Un uso común es crear una directiva para denegar la creación de una dirección IP pública dentro de un ámbito determinado, para evitar la exposición no deseada de un servidor a Internet.
* **Metadatos y administración de costos**. Las etiquetas de recursos suelen usarse para agregar datos de facturación importantes a los recursos y grupos de recursos, como CostCenter, Owner y mucho más. Estas etiquetas ofrecen un valor incalculable para una facturación y administración precisas de los recursos. Las directivas pueden exigir la aplicación de etiquetas de recursos a todos los recursos implementados, lo que facilita la administración.

### <a name="common-uses-of-initiatives"></a>Usos comunes de iniciativas

La introducción de las iniciativas proporciona a las empresas una manera de agrupar directivas lógicas y realizar un seguimiento como un conjunto. Las iniciativas ayudan aún más a las empresas a satisfacer las necesidades de las cargas de trabajo "ágiles" y "tradicionales". Hemos visto usos muy creativos de iniciativas, pero con frecuencia observamos:

* **Habilitar la supervisión en Azure Security Center**. Se trata de una iniciativa predeterminada de la directiva de Azure y de un ejemplo excelente de lo que son las iniciativas. Permite adoptar directivas que identifican bases de datos SQL sin cifrar, vulnerabilidades de las máquinas virtuales y necesidades más comunes relacionadas con la seguridad.
* **Iniciativa normativa específica**. Las empresas suelen agrupar directivas comunes a una disposición normativa (como HIPAA), a fin de poder realizar un seguimiento eficaz de los controles y de la conformidad de dichos controles.
* **SKU y tipos de recursos**. La creación de una iniciativa que restringe los tipos de recursos que se pueden implementar y las SKU que se pueden implementar puede ayudar a controlar los costos y a asegurarse de que la organización solo implementa los recursos que su equipo puede respaldar con los procedimientos y el conjunto de aptitudes necesarios.

> [!TIP]
> Se recomienda usar siempre definiciones de iniciativas en lugar de definiciones de directivas. Después de asignar una iniciativa a un ámbito, por ejemplo, una suscripción o un grupo de administración, puede agregar fácilmente otra directiva a la iniciativa sin tener que cambiar las asignaciones. Esto facilita bastante el reconocimiento de lo que se ha aplicado y el seguimiento del cumplimiento.

### <a name="policy-and-initiative-assignments"></a>Asignaciones de iniciativas y directivas

Una vez creadas las directivas y tras haberlas agrupado en iniciativas lógicas, debe asignar la directiva a un ámbito, ya sea un grupo de administración, una suscripción o incluso un grupo de recursos. Las asignaciones permite excluir un subámbito de la asignación de una directiva. Por ejemplo, si deniega la creación de direcciones IP públicas dentro de una suscripción, podría crear una asignación con una exclusión para un grupo de recursos conectado a la red perimetral protegida.

Encontrará varios ejemplos de directivas que muestran cómo se pueden aplicar directivas e iniciativas a varios recursos en Azure en este repositorio de [GitHub](https://github.com/Azure/azure-policy).

## <a name="identity-and-access-management"></a>Administración de identidades y acceso

Una de las primeras preguntas, y la más importante, que se plantea al empezar a trabajar con la nube pública es "¿Quién debe tener acceso a los recursos?". y cómo puede controlar dicho acceso. Permitir o denegar el acceso a Azure Portal, así como controlar el acceso a los recursos del portal, resulta fundamental para el éxito y la seguridad a largo plazo de los recursos de la nube.

Para realizar la tarea de proteger el acceso a los recursos, primero deberá configurar el proveedor de identidades y luego configurar los roles y el acceso. Azure Active Directory (Azure AD), conectado a su entorno local de Active Directory, es la base de la identidad de Azure. Dicho esto, Azure AD *no* es Active Directory, y es importante comprender qué es un inquilino de Azure y qué relación guarda con la inscripción a Azure.  Revise la [información](../getting-started/azure-resource-access.md) disponible para obtener una base sólida sobre Azure AD y AD. Para conectar y sincronizar Active Directory con Azure AD, instale y configure la [herramienta AD Connect](/azure/active-directory/connect/active-directory-aadconnect) a nivel local.

![arch.png](./_images/arch.png)

Cuando se publicó inicialmente Azure, los controles de acceso a una suscripción eran básicos: Administrador o Coadministrador. Acceder una suscripción del modelo clásico implicaba hacerlo a todos los recursos del portal. Esta falta de un control específico provocó una proliferación de suscripciones que proporcionaban un nivel de control de acceso razonable a una inscripción de Azure. Esta proliferación de suscripciones ya no es necesaria. Con el control de acceso basado en rol (RBAC), puede asignar usuarios a roles estándar que proporcionan acceso común como "propietario", "colaborador" o "lector" o incluso crear roles propios.

Al implementar el acceso basado en roles, los siguientes aspectos son muy recomendables:

* Controlar al administrador y coadministrador de una suscripción, ya que estos roles tienen permisos amplios. Solo debe agregar al propietario de la suscripción como un coadministrador si es necesario administrar las implementaciones clásicas de Azure.

* Usar grupos de administración para asignar [roles](/azure/azure-resource-manager/management-groups-overview#management-group-access) en varias suscripciones y reducir la carga que supone administrarlos a nivel de suscripción.
* Agregue los usuarios de Azure a un grupo (por ejemplo, los propietarios de la aplicación X) en Active Directory. Utilice el grupo sincronizado para proporcionar a los miembros del grupo los derechos adecuados para administrar el grupo de recursos que contiene la aplicación.
* Siga el principio de conceder los **privilegios mínimos** necesarios para realizar el trabajo previsto.

> [!IMPORTANT]
>Considere la posibilidad de utilizar las funcionalidades [Azure AD Privileged Identity Management](/azure/active-directory/privileged-identity-management/pim-configure), Azure [Multi-Factor Authentication](/azure/active-directory/authentication/howto-mfa-getstarted) y [acceso condicional](/azure/active-directory/active-directory-conditional-access-azure-portal) para ofrecer más seguridad y visibilidad de las acciones administrativas en las suscripciones de Azure. Estas funcionalidades derivan de una licencia válida de Azure AD Premium (según la característica) para elevar el nivel de protección y administración de su identidad. Azure AD PIM permite el acceso administrativo "Just-in-Time" con el flujo de trabajo de aprobación, así como una auditoría completa de las actividades y activaciones del administrador. Azure MFA es otra funcionalidad crítica y permite la verificación en dos pasos para iniciar sesión en Azure Portal. Al combinarla con los controles de acceso condicional, puede administrar con eficacia el riesgo de vulnerabilidad.

La planeación y la preparación de los controles de acceso e identidad y el seguimiento del procedimiento recomendado para la administración de identidades en Azure ([vínculo](/azure/security/azure-security-identity-management-best-practices)) es una de las mejores estrategias de mitigación de riesgos que se pueden utilizar y, por tanto, debe considerarse como obligatoria en cada implementación.

## <a name="security"></a>Seguridad

Uno de los principales impedimentos para adoptar la tecnología de la nube siempre han sido las preocupaciones por la seguridad. Los departamentos de riesgos de TI y seguridad tienen que asegurarse de que los recursos de Azure estén protegidos y seguros de forma predeterminada. Azure proporciona una serie de funcionalidades que puede aprovechar para proteger los recursos y detectar o prevenir las amenazas a esos recursos.

### <a name="azure-security-center"></a>Azure Security Center

[Azure Security Center](/azure/security-center/security-center-intro) proporciona una vista unificada del estado de seguridad de los recursos de su entorno, además de protección avanzada contra amenazas. Azure Security Center es una plataforma abierta que permite a los asociados de Microsoft crear software que se integre para mejorar sus funcionalidades. Las funcionalidades básicas de Azure Security Center (nivel Gratis) proporcionan la evaluación y las recomendaciones que mejorarán la situación de seguridad. Sus niveles de pago ofrecerán funcionalidades adicionales importantes, como controles de aplicaciones adaptables (inclusión en listas blancas) y acceso de administrador Just-In-Time.

> [!TIP]
>Azure Security Center es una herramienta muy eficaz que se ha mejorado constantemente y que incorpora nuevas funcionalidades que puede usar para detectar amenazas y proteger su empresa. Se recomienda encarecidamente habilitar siempre ASC.

### <a name="azure-resource-locks"></a>Bloqueos de recursos de Azure

A medida que su organización agrega servicios básicos a la suscripción, cada vez reviste más importancia evitar la interrupción de la actividad empresarial. Un tipo de interrupción que se observa con frecuencia son las consecuencias no deseadas de scripts y herramientas que actúan en contra de una suscripción de Azure eliminando recursos por error. Gracias a los [bloqueos de recursos](/azure/azure-resource-manager/resource-group-lock-resources), se pueden restringir las operaciones en recursos de gran valor donde modificarlas o eliminarlas tendría un gran impacto. Los bloqueos se aplican a una suscripción, un grupo de recursos o incluso a recursos individuales. En el caso de uso común es aplicar bloqueos a recursos fundamentales como redes virtuales, puertas de enlace, grupos de seguridad de red y cuentas de almacenamiento claves.

### <a name="secure-devops-toolkit"></a>Secure DevOps Toolkit

"Secure DevOps Kit para Azure" (AzSK) es una colección de scripts, herramientas, extensiones, automatizaciones, etc. creada originalmente por el propio equipo de TI de Microsoft y lanzada en código abierto a través de GitHub ([vínculo](https://github.com/azsk/DevOpsKit-docs)). AzSK satisface las necesidades de los equipos en cuanto a seguridad de los recursos y las suscripciones de Azure con el uso de una amplia automatización y la integración sin problemas en flujos de trabajo de DevOps nativos, con el fin de ayudar a realizar tareas de desarrollo sencillas con estas seis áreas de enfoque:

* Proteger la suscripción
* Habilitar un desarrollo seguro
* Integrar la seguridad en CI/CD
* Control continuo
* Alertas y supervisión
* Gobierno de riesgos en la nube

![Kit de herramientas de Azure DevOps](_images/Secure_DevOps_Kit_Azure.png)

AzSK es un amplio conjunto de herramientas, scripts e información que constituyen una parte importante de un plan completo de gobierno de Azure, por lo que incorporarlo a la plantilla scaffold es fundamental para respaldar los objetos de administración de riesgos de las organizaciones.

### <a name="azure-update-management"></a>Azure Update Management

Una de las tareas claves que puede hacer para mantener un entorno seguro es asegurarse de que los servidores tienen instaladas las actualizaciones más recientes. Aunque hay muchas herramientas para lograr esto, Azure ofrece la solución [Azure Update Management](/azure/automation/automation-update-management) para abordar la identificación y el lanzamiento de revisiones críticas del sistema operativo.  Usa Azure Automation (que se aborda en la sección [Automatización](#automate) más adelante en esta guía).

## <a name="monitor-and-alerts"></a>Supervisión y alertas

Recopilar y analizar datos de telemetría que proporcionan una clara visión de las actividades, las métricas de rendimiento, el estado y la disponibilidad de los servicios que se usan en todas las suscripciones de Azure es fundamental para administrar las aplicaciones y la infraestructura de forma proactiva y, además, es una necesidad fundamental de cada suscripción de Azure. Cada servicio de Azure genera datos de telemetría en forma de registros de actividad, métricas y registro de diagnóstico.

* En los **registros de actividad** se describen todas las operaciones realizadas en los recursos de las suscripciones.
* Las **métricas** son información numérica generada en un recurso en las que se describen el rendimiento y el estado de dicho recurso.
* Los **registros de diagnóstico** los emite un servicio de Azure y en ellos se proporcionan datos exhaustivos y frecuentes sobre el funcionamiento de dicho servicio.

Esta información se puede ver y examinar a varios niveles y está sujeta a mejoras continuas. Azure ofrece funcionalidades de supervisión **compartida**, **principal** y **exhaustiva** de los recursos de Azure en los servicios descritos en el diagrama siguiente.
![monitoring](./_images/monitoring.png)

### <a name="shared-capabilities"></a>Funcionalidades compartidas

* **Alertas**: puede recopilar cada registro, evento y métrica de los recursos de Azure, pero sin la capacidad de recibir una notificación de condiciones y eventos críticos, estos datos solo son útiles para fines históricos y de análisis forense. Las Alertas de Azure informan de forma proactiva sobre las condiciones definidas en todas las aplicaciones y en la infraestructura. Cree reglas de alertas en los registros, eventos y métricas que usan grupos de acciones para notificar a una serie de destinatarios. Los grupos de acciones también ofrecen la capacidad de automatizar las correcciones con acciones externas, como webhooks, para ejecutar runbooks de Azure Automation y Azure Functions.

* **Paneles**: los paneles permiten agregar vistas de supervisión y combinar datos entre recursos y suscripciones para ofrecerle una visión empresarial de los datos de telemetría de los recursos de Azure. Puede crear y configurar sus propias vistas y compartirlas con otros usuarios. Por ejemplo, podría crear un panel que consta de varios iconos para los administradores de bases de datos, a fin de proporcionar información de todos los servicios de base de datos de Azure, incluidos Azure SQL DB, Azure Database for PostgreSQL y Azure Database for MySQL.

* **Explorador de métricas**: las métricas son valores numéricos generados por los recursos de Azure (por ejemplo, % de CPU o E/S de disco), que proporcionan información detallada sobre el funcionamiento y el rendimiento de los recursos. Mediante el Explorador de métricas puede definir y enviar las métricas que le interesan a Log Analytics para fines de agregación y análisis.

### <a name="core-monitoring"></a>Supervisión básica

* **Azure Monitor**: es el servicio de la plataforma principal que proporciona un único origen para la supervisión de recursos de Azure. La interfaz de Azure Monitor en Azure Portal ofrece un punto de partida centralizado para todas las cara de supervisión de Azure, incluidas las características exhaustivas de supervisión de Application Insights, Log Analytics, Supervisión de red, Soluciones de administración y Service Map. Con Azure Monitor, puede visualizar, consultar, redirigir y archivar las métricas y los registros procedentes de recursos de Azure de toda la nube, así como trabajar con ellos. Además de usar el portal, también puede recuperar datos con los cmdlets de PowerShell para Monitor, la CLI multiplataforma o las API de REST de Azure Monitor.

* **Azure Advisor**: Azure Advisor supervisa constantemente los datos de telemetría de las suscripciones y los entornos y ofrece recomendaciones sobre los procedimientos recomendados en relación a cómo optimizar los recursos de Azure para ahorrar dinero y mejorar el rendimiento, la seguridad y la disponibilidad de los recursos que conforman las aplicaciones.

* **Service Health**: Azure Service Health identifica cualquier problema con los servicios de Azure que pueden afectar a las aplicaciones y le ayuda a planear ventanas de mantenimiento programado.

* **Registro de actividad**: en él se describen todas las operaciones realizadas en los recursos de su suscripción. Proporciona una pista de auditoría para determinar "qué" y "quién" creó, actualizó o eliminó una operación en los recursos y "cuándo" se hizo. Los eventos del registro de actividad se almacenan en la plataforma y están disponibles para consulta durante 90 días. Puede ingerir los registros de actividad en Log Analytics para beneficiarse de períodos de retención más largos y consultas y análisis más exhaustivos entre varios recursos.

### <a name="deep-application-monitoring"></a>Supervisión de aplicaciones en profundidad

* **Application Insights**: permite recopilar telemetría específica de la aplicación y supervisar el rendimiento, la disponibilidad y el uso de las aplicaciones en la nube o en el entorno local. Esto se consigue al equipar a la aplicación de los SDK compatibles con varios lenguajes, como .NET, JavaScript, JAVA, Node.js, Ruby y Python. Los eventos de Application Insights se ingieren en el mismo almacén de datos de Log Analytics que admite la supervisión de seguridad e infraestructura, a fin de permitir la agregación de eventos y ponerlos en correlación con el tiempo mediante un lenguaje de consulta enriquecido.

### <a name="deep-infrastructure-monitoring"></a>Supervisión de infraestructura en profundidad

* **Log Analytics**: desempeña un papel fundamental en la supervisión de Azure al recopilar datos de telemetría y otros desde diversos orígenes y al proporcionar un motor de análisis y un lenguaje de consultas que ofrece información sobre el funcionamiento de las aplicaciones y los recursos. También puede interactuar directamente con los datos de Log Analytics mediante vistas y búsquedas de registros de alto rendimiento, o puede usar herramientas de análisis en otros servicios de Azure que almacenen sus datos en Log Analytics como Application Insights o Azure Security Center.

* **Supervisión de red**: los servicios de supervisión de red de Azure permiten obtener información detallada sobre el flujo del tráfico de red, el rendimiento, la seguridad, la conectividad y los cuellos de botella. Un diseño de red bien planeado debe incluir la configuración de servicios de supervisión de red de Azure, como Network Watcher y Supervisión de ExpressRoute.

* **Soluciones de administración**: estas soluciones son conjuntos empaquetados de lógica, información detallada y consultas predefinidas de Log Analytics de una aplicación o un servicio. Usan Log Analytics como base para almacenar y analizar datos de eventos. Las solución de administración de ejemplo incluyen la supervisión de contenedores y análisis de Azure SQL Database.

* **Service Map**: proporciona una vista gráfica de los componentes de la infraestructura y sus procesos e interdependencias en otros equipos y procesos externos. Integra eventos, datos de rendimiento y soluciones de administración en Log Analytics.

> [!TIP]
> Antes de crear alertas individuales, cree y mantenga un conjunto de grupos de acciones compartidos que se puedan usar en las alertas de Azure. Esto permitirá realizar un mantenimiento centralizado del ciclo de vida de las listas de destinatarios, los métodos de entrega de notificaciones (correo electrónico, SMS y números de teléfono) y webhooks para acciones externas (runbooks de Azure Automation, Azure Functions/Logic Apps e ITSM).

## <a name="cost-management"></a>Administración de costos

Uno de los cambios más importantes al que se enfrentará al realizar la migración de la nube local a la nube pública es el cambio del gasto de capital (comprar hardware) al gasto operativo (pagar por el servicio a medida que lo usa). Este cambio de los gastos de capital por los gastos operativos también genera la necesidad de administrar los costos con más atención. La ventaja de la nube es que puede reducir de forma significativa y positiva el costo de un servicio con tan solo desactivarlo o cambiarlo de tamaño cuando no lo necesite. La administración minuciosa de los costos en la nube es un práctica recomendada que los clientes consolidados aplican a diario.

Microsoft ofrece varias herramientas para poder visualizar, controlar y administrar los costos. También proporcionamos un conjunto completo de API para que pueda personalizar e integrar la administración de costos en sus propias herramientas y paneles. Estas herramientas pueden clasificarse a nivel general en: funcionalidades de Azure Portal y funcionalidades externas

### <a name="azure-portal-capabilities"></a>Funcionalidades de Azure Portal

Se trata de herramientas que ofrecen información instantánea de los costos y la posibilidad de realizar acciones.

* **Costo de recursos de suscripción**: en Azure Portal, la vista [Análisis de costos de Azure](/azure/cost-management/overview) proporciona una visión rápida de los costos e información sobre el gasto diario por recurso o grupo de recursos.
* **Azure Cost Management**: este producto es el resultado de la compra de Cloudyn por parte de Microsoft, que permite administrar y analizar el gasto en Azure y el gasto en otros proveedores de nube pública. Se encuentran disponibles niveles gratuitos y de pago, con un magnífico conjunto de funcionalidades como se observa en la [Introducción](/azure/cost-management/overview).
* **Grupos de acciones y Azure Budgets**: saber lo que cuesta algo y hacer algo al respecto hasta hace poco ha sido más que un ejercicio manual. Con la introducción de Azure Budgets y sus API, ahora se pueden crear acciones (como se observa en [este](https://channel9.msdn.com/Shows/Azure-Friday/Managing-costs-with-the-Azure-Budgets-API-and-Action-Groups) ejemplo) cuando los costos alcanzan un límite. Por ejemplo, cierra un grupo de recursos de "prueba" cuando alcanza el 100 % de su presupuesto u [otro ejemplo].
* **Azure Advisor**: saber lo que cuesta algo es solo la mitad de la batalla; la otra mitad consiste en saber qué hacer con esa información. [Azure Advisor](/azure/advisor/advisor-overview) proporciona recomendaciones sobre las acciones necesarias para ahorrar dinero, mejorar la confiabilidad o incluso aumentar la seguridad.

### <a name="external-cost-management-tools"></a>Herramientas de administración de costos externas

* **Azure Consumption Insights para Power BI**. ¿Desea crear visualizaciones propias de la organización? En su caso, el paquete de contenido de Azure Consumption Insights para Power BI es la herramienta más adecuada. Con este paquete de contenido y Power BI puede crear visualizaciones personalizadas para representar su organización, realizar análisis más exhaustivos de los costos y agregar otros orígenes de datos para obtener información más precisa.

* **API de consumo**. Las [API de consumo](/rest/api/consumption/) otorgan acceso programático a datos de costo y uso, así como a información sobre presupuestos, instancias reservadas y cambios en Marketplace. Estas API son accesibles solo para las inscripciones Enterprise y algunas suscripciones a Web Direct; no obstante, ofrecen la posibilidad de integrar los datos de costos en almacenamiento de datos y herramientas propios. También puede acceder a estas API con la CLI de Azure, como se detalla [aquí](/cli/azure/consumption?view=azure-cli-latest).

Si nos centramos en los clientes que han usado la nube durante mucho tiempo y han "consolidado" su uso, nos planteamos una serie de prácticas muy recomendadas.

* **Supervisión activa de costos**. Las organizaciones que son usuarios consolidados de Azure supervisan constantemente los costos y adoptan medidas cuando es necesario. Algunas organizaciones incluso dedican personal a realizar análisis y sugerir cambios de uso, y estas personas en lugar de financiarse de forma autónoma, encuentran un clúster de HDInsight sin utilizar que se ha estado ejecutando durante meses.
* **Uso de instancias reservadas**. Otro inquilino clave para administrar los costos en la nube consiste en utilizar la herramienta adecuada para el trabajo. Si dispone de una máquina virtual IaaS que debe estar disponible ininterrumpidamente, el uso de una instancia reservada le permitirá ahorrar bastante dinero. Hallar un equilibrio adecuado entre la automatización del apagado de las máquinas virtuales y el uso de instancias reservadas conlleva experiencia y análisis.
* **Uso eficaz de la automatización**: muchas cargas de trabajo no necesitan ejecutarse a diario. Incluso apagar una máquina virtual durante cuatro horas al día puede suponer un ahorro del 15 % del costo. Automation se amortizará rápidamente.
* **Uso de etiquetas en recursos para dar visibilidad**: como se mencionó en otra sección de este documento, el uso de etiquetas en recurso permitirá mejorar el análisis de los costos.

La administración de costos es una disciplina fundamental para una ejecución eficaz y efectiva de una nube pública. Las empresas que consigan el éxito podrán controlar sus costos y adaptarlos a la demanda real, en lugar de comprar en exceso con la esperanza de que haya demanda.

## <a name="automate"></a>Automatizar

Una de las muchas funcionalidades que diferencia la madurez de las organizaciones en el uso de los proveedores de la nube es el nivel de automatización que han incorporado.  La automatización es un proceso interminable y, a medida que la organización realiza la transición a la nube, se convierte en un área en el que es necesario invertir recursos y tiempo para realizar compilaciones.  La automatización sirve para muchos propósitos, entre otros, el lanzamiento coherente de recursos (donde está directamente vinculada a otro concepto clave de scaffolding, a saber, las plantillas y DevOps), para resolver cualquier problema.  La automatización es el "tejido conectivo" de la plantilla scaffold de Azure y vincula cada área.

Hay una serie de herramientas disponibles a medida que se crea esta funcionalidad, desde herramientas propias, como Azure Automation, EventGrid y la CLI de Azure, hasta un amplio conjunto de herramientas de terceros, como Terraform, Jenkins, Chef y Puppet, entre otras. Las herramientas fundamentales para ofrecer a su equipo de operaciones la capacidad de automatizar son Azure Automation, Event Grid y Azure Cloud Shell:

* **Azure Automation**: es una funcionalidad basada en la nube que permite crear runbooks (en PowerShell o Python) y permite automatizar procesos, configurar recursos e incluso aplicar revisiones.  [Azure Automation](/azure/automation/automation-intro) dispone de un conjunto amplio de funcionalidades multiplataforma que forman parte integral de la implementación, pero son demasiado extensas como para tratarlas aquí en profundidad.
* **Event Grid**: este [servicio](/azure/event-grid) es un sistema de enrutamiento de eventos totalmente administrado que permite reaccionar a eventos dentro del entorno de Azure. Como la automatización es el tejido conectivo de organizaciones maduras en la nube, Event Grid es el tejido conectivo de una buena automatización. Con Event Grid, puede crear una acción sencilla y sin servidor para enviar un correo electrónico a un administrador cada vez que se crea un recurso y registrar dicho recurso en la base de datos. Esa misma acción de Event Grid puede notificar cuándo se elimina un recurso y eliminar el elemento de la base de datos.
* **Azure Cloud Shell**: es un [shell](/azure/cloud-shell/overview) interactivo basado en el explorador para administrar recursos en Azure. Proporciona un entorno completo de PowerShell o Bash que se inicia según sea necesario y de cuyo mantenimiento se encarga el usuario, para poder disponer de un entorno coherente desde el que ejecutar los scripts. Azure Cloud Shell proporciona acceso a herramientas clave adicionales ya instaladas para automatizar el entorno, como la [CLI de Azure](/cli/azure/get-started-with-azure-cli?view=azure-cli-latest), [Terraform](/azure/virtual-machines/linux/terraform-install-configure) y una lista creciente de [herramientas](https://azure.microsoft.com/updates/cloud-shell-new-cli-tools-and-font-size-selection/) adicionales para administrar contenedores, bases de datos (sqlcmd) y mucho más.

La automatización es un trabajo de jornada completa y rápidamente se convertirá en una de las tareas operativas más importantes en su equipo en la nube. Las organizaciones que adoptan el enfoque de "automatizar primero" tienen más éxito al usar Azure:

* Administración de costos: buscar oportunidades de forma activa y crear automatización para cambiar el tamaño de los recursos, escalar y reducir verticalmente y apagar los recursos que no se utilizan.
* Flexibilidad operativa: con el uso de la automatización (junto con las plantillas y DevOps), conseguirá un nivel de repetición que aumenta la disponibilidad, mejora la seguridad y permite a su equipo centrarse en solucionar problemas empresariales.

## <a name="templates-and-devops"></a>Plantillas y DevOps

Como se explica en la sección de automatización, el objeto como organización debe ser aprovisionar recursos mediante scripts y plantillas controlados por código fuente y minimizar la configuración interactiva de los entornos. Este enfoque de "infraestructura como código" junto con un proceso disciplinado de DevOps para la implementación continua puede garantizar la coherencia y reducir el desfase en todos los entornos. Casi todos los recursos de Azure se pueden implementar con [plantillas JSON de Azure Resource Manager (ARM)](/azure/azure-resource-manager/resource-group-template-deploy) junto con PowerShell o las herramientas y la CLI multiplataforma de Azure, como Terraform de Hashicorp (que ofrece compatibilidad de primera clase y se integra en Azure Cloud Shell).

Un artículo como [este](https://blogs.msdn.microsoft.com/mvpawardprogram/2018/05/01/azure-resource-manager/) ofrece un excelente debate sobre los procedimientos recomendados y las lecciones aprendidas en relación con la aplicación de un enfoque de DevOps a plantillas de ARM con la cadena de herramientas de [Azure DevOps](/azure/devops/user-guide/?view=vsts). Dedique tiempo y esfuerzo para desarrollar un conjunto de plantillas específicas de los requisitos de su organización y para desarrollar canalizaciones de entrega continua con las cadenas de herramientas de DevOps (Azure DevOps, Jenkins, Bamboo, Teamcity y Concourse), especialmente para los entornos de producción y de control de calidad. Existe una gran biblioteca de [plantillas de inicio rápido de Azure](https://github.com/Azure/azure-quickstart-templates) en GitHub que puede usar como punto de partida para las plantillas, a fin de poder crear rápidamente canalizaciones de entrega basadas en la nube con Azure DevOps.

Como procedimiento recomendado para grupos de recursos o suscripciones de producción, el objetivo debe ser utilizar la seguridad RBAC para impedir usuarios interactivos de forma predeterminada y utilizar canalizaciones de entrega continua automatizadas basadas en entidades de servicio para aprovisionar todos los recursos y entregar todo el código de la aplicación. Ningún administrador o desarrollador debe usar Azure Portal para configurar los recursos de forma interactiva. Este nivel de DevOps realiza un esfuerzo conjunto, utiliza todos los conceptos de las plantillas scaffold de Azure y proporciona un entorno coherente y más seguro que se adapta a sus organizaciones para aumentar la escala.

> [!TIP]
> Al diseñar y desarrollar plantillas de ARM complejas, utilice [plantillas vinculadas](/azure/azure-resource-manager/resource-group-linked-templates) para organizar y refactorizar relaciones complejas de recursos desde archivos JSON monolíticos. Esto le permitirá administrar los recursos individualmente y permitirá elevar el nivel de legibilidad, prueba y reutilización de las plantillas.

Azure es un proveedor de la nube de hiperescala y, a medida que se migra la organización del mundo de los servidores locales a la nube, utilizar los mismos conceptos que usan los proveedores de la nube y las aplicaciones SaaS permitirá a la organización satisfacer las necesidades de la empresa de una forma bastante más eficaz.

## <a name="core-network"></a>Red principal

El componente definitivo del modelo de referencia de las plantillas scaffold de Azure es fundamental para que la organización acceda a Azure de forma segura. El acceso a los recursos puede ser interno (dentro de la red corporativa) o externo (a través de Internet). Es muy fácil que los usuarios de una organización puedan colocar accidentalmente los recursos en la zona incorrecta y, posiblemente, abrirlos para acceder con intenciones malintencionado. Al igual que con los dispositivos locales, las empresas deben agregar los controles adecuados para asegurarse de que los usuarios de Azure toman las decisiones correctas. Para el gobierno de suscripciones, identificamos los recursos principales que proporcionan un control de acceso básico. Se trata de los siguientes:

* Las **redes virtuales** son objetos de contenedor de las subredes. Aunque no es estrictamente necesario, se suelen utilizar al conectar las aplicaciones a los recursos corporativos internos.
* Las **rutas definidas por el usuario** permiten manipular la tabla de rutas dentro de una subred, lo que permite enviar tráfico a través de una aplicación virtual de red o a una puerta de enlace remota en una red virtual emparejada.
* El **emparejamiento de red virtual** permite conectar sin problemas dos o más redes virtuales de Azure y crear redes de servicios compartidas o diseños tipo hub-and-spoke más complejos.
* **Puntos de conexión de servicio**. En el pasado, los servicios PaaS se basaban en métodos diferentes para proteger el acceso a esos recursos desde las redes virtuales. Los puntos de conexión de servicio permiten proteger el acceso a los servicios PaaS habilitados SOLO desde puntos de conexión conectados, de tal forma que se aumenta la seguridad general.
* Los **grupos de seguridad** son un amplio conjunto de reglas que proporcionan la capacidad de permitir o denegar el tráfico entrante y saliente hacia y desde los recursos de Azure. Los [grupos de seguridad](/azure/virtual-network/security-overview) constan de reglas de seguridad, que pueden ampliarse con **etiquetas de servicio** (que definen los servicios de Azure comunes como Azure Key Vault, SQL etc.) y **grupos de aplicaciones** (que definen la estructura de una aplicación, como servidores web, servidores de aplicaciones, etc.).

> [!TIP]
> Use etiquetas de servicio y grupos de aplicaciones en los grupos de seguridad de red no solo para mejorar la confiabilidad de las reglas, que resulta fundamental para entender el impacto, sino también para permitir una microsegmentación eficaz en una subred más grande, para reducir así la expansión y aumentar la flexibilidad.

### <a name="virtual-data-center"></a>Centro de datos virtual

Azure proporciona funcionalidades internas y de terceros mediante nuestra amplia red de asociados que permite disponer de una postura de seguridad eficaz. Más importante aún, Microsoft proporciona procedimientos recomendados y orientación en el formulario del [centro de datos virtual de Azure](/azure/architecture/vdc/networking-virtual-datacenter). A medida que pasa de una única carga de trabajo a varias cargas de trabajo que se benefician de las funcionalidades híbridas, la orientación sobre el centro de datos virtual le dará la "receta" para habilitar una red flexible que crecerá al mismo ritmo que las cargas de trabajo de Azure.  

## <a name="next-steps"></a>Pasos siguientes

Contar con un sistema de gobierno es fundamental para que Azure tenga éxito. En este artículo se aborda la implementación técnica de una scaffold empresarial, pero solo se mencionan el proceso más amplio y las relaciones entre los componentes. El gobierno de directivas es jerárquico y viene determinado por lo que la empresa desee conseguir. Naturalmente, en la creación de un modelo de gobierno de Azure participan los representantes del departamento de TI. Sin embargo, es más importante que los líderes del grupo de negocios y los responsables de seguridad y riesgos tengan una importante representación en dicho proceso. Al final, una scaffold empresarial tiene que ver con mitigar los riesgos empresariales para facilitar la misión y los objetivos de la organización.

Ahora que ha obtenido información sobre el gobierno de suscripciones, es hora de ver estas recomendaciones en la práctica. Vea [Examples of implementing Azure subscription governance](azure-scaffold-examples.md) (Ejemplos de implementación de un sistema de gobierno de suscripciones).
