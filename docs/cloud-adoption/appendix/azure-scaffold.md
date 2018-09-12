---
title: Prácticas recomendadas para empresas que migran su infraestructura a Azure
description: En este artículo se describe una plantilla scaffold que las empresas pueden utilizar para garantizar que el entorno sea seguro y fácil de administrar.
author: rdendtler
ms.date: 03/31/2017
ms.author: rodend;karlku;tomfitz
ms.openlocfilehash: 91adce796ae7785d3831e9628fce0193076eec9b
ms.sourcegitcommit: f4069cf68456b5c74acb1b890dc4e45e11f12b59
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2018
ms.locfileid: "43675790"
---
# <a name="azure-enterprise-scaffold---prescriptive-subscription-governance"></a>Scaffolding empresarial de Azure: gobierno de suscripción preceptivo
Cada vez son más las empresas que adoptan la tecnología de nube pública para ganar agilidad y flexibilidad. Aprovechan los puntos fuertes de la nube para generar ingresos u optimizar los recursos de la empresa. Microsoft Azure proporciona un gran número de servicios que las empresas pueden ensamblar como bloques de creación con el objetivo de abordar diversas aplicaciones y cargas de trabajo. 

Sin embargo, a menudo es difícil saber por dónde comenzar. Después de decantarse por Azure, suelen surgirles unas cuantas preguntas:

* "¿Cómo puedo cumplir los requisitos legales de soberanía de datos en determinados países?"
* "¿Cómo puedo asegurarme de que alguien no modifique por error un sistema importante?"
* "¿Cómo puedo saber lo que cada recurso admite para poder tenerlo en cuenta y realizar la facturación de forma adecuada?"

La posibilidad de que existan suscripciones vacías sin ninguna protección es desalentadora. Este espacio en blanco puede dificultar la migración a Azure.

En este artículo se ofrece un punto de partida para que los profesionales técnicos satisfagan la necesidad de contar con un sistema de gobierno y la equilibren con el imperativo de ganar agilidad. Además, se presenta el concepto de scaffolding empresarial, que guía a las organizaciones en la implementación y administración de suscripciones de Azure. 

## <a name="need-for-governance"></a>Necesidad de contar con un sistema de gobierno
Al migrar la infraestructura a Azure, hay que abordar el asunto del sistema de gobierno en la primera fase para garantizar que la nube se utilice correctamente en la empresa. Desafortunadamente, debido al tiempo que se tarda en crear un sistema de gobierno completo y toda la administración que conlleva, algunos grupos de negocios recurren directamente a los proveedores sin contar con su equipo de TI. Este enfoque puede dejar a la empresa expuesta a vulnerabilidades si los recursos no se administran correctamente. Las características de la nube pública —agilidad, flexibilidad y precios basados en el consumo— son importantes para los grupos de negocios que necesitan satisfacer rápidamente las necesidades de los clientes (internos y externos). Sin embargo, el equipo de TI de la empresa debe garantizar que los datos y sistemas estén protegidos de forma eficaz.

En condiciones reales, se utiliza la técnica scaffolding para crear la base de la estructura. Las plantillas scaffold sirven de guía para el esquema general y proporcionan puntos de anclaje para poder montar sistemas más permanentes. Las scaffold empresariales son lo mismo: una serie de controles flexibles y funcionalidades de Azure que proporcionan la estructura al entorno, así como delimitadores para los servicios basados en la nube pública. Además, brindan a los fabricantes (grupos de negocio y departamentos de TI) los cimientos para crear y asociar nuevos servicios.

Estas plantillas se basan en las prácticas que hemos recopilado de muchas interacciones con clientes de varios tamaños. Desde pequeñas organizaciones que desarrollan soluciones en la nube para empresas de la lista Fortune 500 hasta fabricantes independientes de software que van a migrar y desarrollar soluciones en la nube. Las plantillas scaffold empresariales se han creado con la suficiente flexibilidad como para admitir las cargas de trabajo de TI tradicionales y las del método ágil; por ejemplo, los desarrolladores que crean aplicaciones de software como servicio (SaaS) basadas en las funcionalidades de Azure.

Asimismo, están diseñadas para ser el pilar de todas las suscripciones nuevas de Azure. Estas plantillas permiten a los administradores asegurarse de que las cargas de trabajo cumplan los requisitos mínimos de gobierno de una organización sin impedir que los grupos de negocios y los desarrolladores cumplan rápidamente sus objetivos.

> [!IMPORTANT]
> Contar con un sistema de gobierno es fundamental para que Azure tenga éxito. En este artículo se aborda la implementación técnica de una scaffold empresarial, pero solo se mencionan el proceso más amplio y las relaciones entre los componentes. El gobierno de directivas es jerárquico y viene determinado por lo que la empresa desee conseguir. Naturalmente, en la creación de un modelo de gobierno de Azure participan los representantes del departamento de TI. Sin embargo, es más importante que los líderes del grupo de negocios y los responsables de seguridad y riesgos tengan una importante representación en dicho proceso. Al final, una scaffold empresarial tiene que ver con mitigar los riesgos empresariales para facilitar la misión y los objetivos de la organización.
> 
> 

En la imagen siguiente se describen los componentes de la scaffold. La base se sustenta en un plan sólido para departamentos, cuentas y suscripciones. Los pilares constan de directivas de Resource Manager y de estándares de nomenclatura eficaces. El resto de los componentes provienen de las principales funcionalidades y características de Azure que posibilitan un entorno seguro y fácil de administrar.

![Componentes de una plantilla scaffold](./_images/components.png)

> [!NOTE]
> Azure ha crecido rápidamente desde que se presentara en el 2008. Debido a este desarrollo, los equipos de ingeniería de Microsoft tuvieron que replantear su enfoque de administración e implementación de servicios. El modelo de Azure Resource Manager se introdujo en 2014 y reemplaza al de implementación clásica. Resource Manager permite a las organizaciones implementar, organizar y controlar los recursos de Azure de una forma más sencilla. Además, con esta herramienta se pueden crear recursos de forma paralela, de modo que es posible implementar más rápido soluciones complejas e interdependientes. También se puede controlar el acceso de manera pormenorizada y etiquetar los recursos con metadatos. Microsoft recomienda crear todos los recursos mediante el modelo de implementación de Resource Manager. Las plantillas scaffold empresariales se han diseñado expresamente para el modelo de Resource Manager.
> 
> 

## <a name="define-your-hierarchy"></a>Definición de la jerarquía
La base de una plantilla scaffold es la inscripción de Azure Enterprise (y Enterprise Portal). La inscripción Enterprise define la forma y el uso de los servicios de Azure en una empresa y su estructura de gobierno básica. En el contrato Enterprise, los clientes pueden subdividir el entorno en departamentos, cuentas y, finalmente, suscripciones. Una suscripción de Azure es la unidad básica donde se encuentran todos los recursos. También define varios límites en Azure, como el número de núcleos, recursos, etc.

![Jerarquía](./_images/agreement.png)

Cada empresa es diferente, y la jerarquía de la imagen anterior permite una gran flexibilidad en lo que respecta a cómo se organiza Azure en la empresa. Antes de implementar las instrucciones de este documento, debe modelar la jerarquía y comprender el impacto en la facturación, el acceso a los recursos y la complejidad.

Los tres patrones comunes de las inscripciones de Azure son los siguientes:

* El patrón **funcional**
  
    ![functional](./_images/functional.png)
* El patrón de **unidad de negocio** 
  
    ![Patrón de unidad de negocio](./_images/business.png)
* El patrón **geográfico**
  
    ![Patrón geográfico](./_images/geographic.png)

Las plantillas scaffold se aplican en el nivel de suscripción para ampliar los requisitos de gobierno de la empresa en la suscripción.

## <a name="naming-standards"></a>Estándares de nomenclatura
El primer pilar de una plantilla scaffold son los estándares de nomenclatura. Cuando se diseñan bien, se pueden identificar los recursos en el portal, en una factura y en los scripts. Es probable que ya tenga estándares de nomenclatura para la infraestructura local. Al agregar Azure a su entorno, debe extender estos estándares de nomenclatura a los recursos de Azure. Un estándar de nomenclatura permite administrar de forma más eficaz el entorno en todos los niveles.

> [!TIP]
> Convenciones de nomenclatura:
> * Revise y adopte siempre que sea posible esta [guía de patrones y prácticas](../../best-practices/naming-conventions.md), que lo ayudará a decidirse por un estándar de nomenclatura significativo.
> * Alterne el uso de mayúsculas y minúsculas en los nombres de recursos (por ejemplo, miGrupoDeRecursos y NombreDeLaRedVirtual). Nota: Hay determinados recursos, como las cuentas de almacenamiento, en los que solo se pueden emplear minúsculas (y ningún otro carácter especial).
> * Plantéese usar las directivas de Azure Resource Manager (que se describen en la sección siguiente) para aplicar los estándares de nomenclatura.
> 
> Las sugerencias anteriores lo ayudan a implementar una convención de nomenclatura coherente.

## <a name="policies-and-auditing"></a>Directivas y auditoría
El segundo pilar de scaffolding implica la creación de [directivas de Azure](/azure/azure-policy/azure-policy-introduction) y la [auditoría del registro de actividades](/azure/azure-resource-manager/resource-group-audit). Las directivas de Resource Manager ofrecen la posibilidad de administrar riesgos en Azure. Puede definir directivas que garanticen la soberanía de datos restringiendo, aplicando o auditando determinadas acciones. 

* Las directivas son un sistema de **permisos** predeterminado. Las acciones se controlan mediante la definición y asignación de directivas a los recursos que deniegan o auditan acciones en los recursos.
* Las directivas se describen a través de las definiciones de directiva en un lenguaje de definición de directivas (condiciones If-Then).
* Las directivas se crean con archivos en formato JSON (notación de objetos JavaScript). Después de definir una directiva, se asigna a un ámbito determinado: suscripción, grupo de recursos o recurso.

Las directivas tienen varias acciones que permiten adoptar un enfoque específico en sus escenarios. Las acciones son las siguientes:

* **Denegar**: bloquea la solicitud del recurso.
* **Auditoría**: permite la solicitud, pero se agrega una línea en el registro de actividad (que se puede usar para proporcionar alertas o desencadenar runbooks).
* **Anexar**: agrega información especificada al recurso. Por ejemplo, si no hay una etiqueta CostCenter en un recurso, agregue dicha etiqueta con un valor predeterminado.

### <a name="common-uses-of-resource-manager-policies"></a>Usos comunes de las directivas de Resource Manager
Las directivas de Azure Resource Manager constituyen una solución eficaz del kit de herramientas de Azure. Gracias a ellas, podrá evitar costos inesperados, identificar un centro de costos de recursos a través del etiquetado y asegurarse de que se cumplan los requisitos de cumplimiento. Cuando las directivas se combinan con las características de auditoría integradas, puede crear soluciones complejas y flexibles. Las directivas permiten a las compañías proporcionar controles para las cargas de trabajo de TI tradicional y las del método ágil; por ejemplo, desarrollando aplicaciones para los clientes. Los patrones más comunes que se pueden encontrar para las directivas son los siguientes:

* **Soberanía de datos y cumplimiento geográfico**: Azure proporciona regiones en todo el mundo. Las empresas suelen querer controlar el lugar dónde se crean los recursos (ya sea para garantizar la soberanía de datos o simplemente para asegurarse de que se crean cerca de los consumidores finales de los recursos).
* **Administración de costos**: una suscripción de Azure puede contener recursos de muchos tipos y escala. A menudo, las organizaciones quieren asegurarse de que las suscripciones estándares no empleen recursos innecesariamente grandes, lo que puede acarrear un costo de cientos de dólares, o más, al mes.
* **Gobierno predeterminado a través de las etiquetas obligatorias**: requerir etiquetas es una de las características más habituales y solicitadas. Al utilizar las directivas de Azure Resource Manager, las empresas pueden asegurarse de que un recurso esté correctamente etiquetado. Las etiquetas más comunes son las de departamento, propietario del recurso y tipo de entorno (por ejemplo, producción, pruebas o desarrollo).

**Ejemplos**

Suscripción de TI tradicional para aplicaciones de línea de negocio

* Aplique las etiquetas Department y Owner en todos los recursos.
* Restrinja la creación de recursos en la región de Norteamérica.
* Restrinja la capacidad de crear máquinas virtuales de serie G y clústeres de HDInsight.

Entorno del método ágil para una unidad de negocio que va a crear aplicaciones en la nube

* Para cumplir los requisitos de soberanía de datos, permita la creación de recursos SOLO en una región específica.
* Aplique la etiqueta Environment en todos los recursos. Si un recurso se crea sin una etiqueta, anexe la etiqueta **Environment: Unknown** al recurso.
* Realice la auditoría cuando los recursos se creen fuera de Norteamérica, pero no la impida.
* Realice la auditoría cuando se creen recursos de alto costo.

> [!TIP]
> El uso más común de las directivas de Resource Manager en las organizaciones es controlar *qué* tipos de recursos pueden crearse y *dónde*. Además de proporcionar controles de *ubicación* y *tipo*, muchas empresas utilizan directivas para garantizar que los recursos tengan los metadatos apropiados para facturar por consumo. Se recomienda aplicar directivas en el nivel de suscripción para los siguientes patrones:
> 
> * Soberanía de datos y cumplimiento geográfico
> * Administración de costos
> * Etiquetas obligatorias (se determinan según la necesidad de la empresa, como BillTo y Application Owner)
> 
> Puede aplicar más directivas en niveles inferiores del ámbito.
> 
> 

### <a name="audit---what-happened"></a>Auditoría: ¿qué ha ocurrido?
Para ver cómo funciona el entorno, debe auditar la actividad de usuario. La mayoría de los tipos de recursos de Azure crean registros de diagnóstico que se pueden analizar a través de una herramienta de registro o en Azure Log Analytics. Puede recopilar registros de actividades a través de varias suscripciones para proporcionar información de la empresa o de departamentos concretos. Los registros de auditoría constituyen una herramienta de diagnóstico importante y un mecanismo fundamental para desencadenar eventos en el entorno de Azure.

Los registros de actividades de las implementaciones de Resource Manager permiten determinar las **operaciones** que se realizaron y las personas responsables. Además, pueden recopilarse y agregarse mediante herramientas como Log Analytics.


## <a name="resource-group"></a>Grupos de recursos
Resource Manager permite colocar recursos en grupos significativos de administración, facturación y afinidad natural. Como se mencionó anteriormente, Azure tiene dos modelos de implementación. En el modelo clásico anterior, la unidad básica de administración era la suscripción. Era difícil desglosar los recursos dentro de una suscripción, así que había que crear un gran número de suscripciones. Con el modelo de Resource Manager, se han introducido los grupos de recursos. Los grupos de recursos son contenedores de recursos que tienen un ciclo de vida común o comparten un atributo como "Todos los servidores SQL Server" o "Aplicación A".

Además, no pueden estar dentro de otros grupos y los recursos solo pueden pertenecer a un solo grupo de recursos. Puede aplicar determinadas acciones en todos los recursos de un grupo de recursos. Por ejemplo, al eliminar un grupo de recursos, se quitan todos los recursos del grupo de recursos. Normalmente, las aplicaciones completas o los sistemas relacionados se colocan en el mismo grupo de recursos. Por ejemplo, una aplicación de tres niveles denominada "aplicación web de Contoso" contendría el servidor web, el servidor de aplicaciones y el servidor SQL Server en el mismo grupo de recursos.

> [!TIP]
> El modo de organizar los grupos de recursos puede variar según se trate de cargas de trabajo de TI tradicional o del método ágil:
> 
> * Las cargas de trabajo de TI tradicional se suelen agrupar por elementos dentro del mismo ciclo de vida, como una aplicación. Si se agrupan las aplicaciones, se podrán administrar las aplicaciones de forma individual.
> * Las cargas de trabajo de TI o del método ágil tienden a centrarse en las aplicaciones en la nube orientadas a los clientes externos. Los grupos de recursos deben reflejar las capas de implementación (por ejemplo, el nivel web y el de aplicaciones) y administración.
> 
> La comprensión de la carga de trabajo lo ayudará a desarrollar una estrategia de grupo de recursos.


## <a name="resource-tags"></a>Etiquetas del recurso
A medida que los usuarios de su organización agregan recursos a la suscripción, cada vez adquiere más importancia asociar los recursos con el departamento, el cliente y el entorno adecuados. Puede asociar metadatos a los recursos a través de las [etiquetas](/azure/azure-resource-manager/resource-group-using-tags). Estas sirven para proporcionar información sobre el recurso o el propietario. Además, no solo permiten agregar y agrupar los recursos de varias maneras, sino también emplear esos datos para fines de contracargo. Puede etiquetar recursos con hasta 15 pares clave-valor. 

Las etiquetas de recurso son flexibles y deben asociarse a la mayoría de los recursos. Ejemplos de etiquetas de recursos comunes:

* BillTo
* Department (o Business Unit)
* Environment (Production, Stage y Development)
* Tier (Web Tier y Application Tier)
* Application Owner
* ProjectName

![etiquetas](./_images/resource-group-tagging.png)

Consulte [Recommended naming conventions for Azure resources](../../best-practices/naming-conventions.md) (Convenciones de nomenclatura recomendadas para los recursos de Azure).

> [!TIP]
> Considere la posibilidad de utilizar una directiva que exija etiquetar los siguientes recursos:
> 
> * Grupos de recursos
> * Storage
> * Virtual Machines
> * Servidores web y entornos de servicios de aplicaciones
> 
> Esta estrategia de etiquetado identifica en las distintas suscripciones qué metadatos se necesitan para los aspectos comerciales, económicos, de seguridad, de administración de riesgos y de administración general del entorno. 



## <a name="role-based-access-control"></a>Control de acceso basado en rol
Probablemente se esté preguntando quién debe tener acceso a los recursos y cómo puede controlar dicho acceso. Permitir o denegar el acceso a Azure Portal, así como controlar el acceso a los recursos del portal, resulta fundamental. 

Cuando se publicó inicialmente Azure, los controles de acceso a una suscripción eran básicos: Administrador o Coadministrador. Acceder una suscripción del modelo clásico implicaba hacerlo a todos los recursos del portal. Esta falta de un control específico provocó una proliferación de suscripciones que proporcionaban un nivel de control de acceso razonable a una inscripción de Azure.

Esta proliferación de suscripciones ya no es necesaria. Gracias al [control de acceso basado en rol](/azure/role-based-access-control/overview), puede asignar usuarios a roles estándares (por ejemplo, los tipos de roles comunes de "lector" y "escritor"). También se pueden definir reglas personalizadas.

> [!TIP]
> Para implementar el control de acceso basado en roles:
> * Conecte su almacén de identidades corporativo (normalmente, Active Directory) a Azure Active Directory con la herramienta AD Connect.
> * Controle el administrador o coadministrador de una suscripción mediante una identidad administrada. **No** asigne administradores o coadministradores a un nuevo propietario de la suscripción. En su lugar, use los roles RBAC para proporcionar derechos de **propietario** a un grupo o usuario.
> * Agregue los usuarios de Azure a un grupo (por ejemplo, los propietarios de la aplicación X) en Active Directory. Utilice el grupo sincronizado para proporcionar a los miembros del grupo los derechos adecuados para administrar el grupo de recursos que contiene la aplicación.
> * Siga el principio de conceder los **privilegios mínimos** necesarios para realizar el trabajo previsto. Por ejemplo: 
>   * Grupo de implementación: un grupo que solo puede implementar recursos.
>   * Administración de máquinas virtuales: un grupo que puede reiniciar las máquinas virtuales (para realizar operaciones)
> 
> Estas sugerencias lo ayudarán a administrar el acceso de los usuarios en la totalidad de su suscripción.

## <a name="azure-resource-locks"></a>Bloqueos de recursos de Azure
A medida que su organización agrega servicios básicos a la suscripción, cada vez reviste más importancia asegurarse de que estos servicios estén disponibles para evitar la interrupción de la actividad empresarial. Gracias a los [bloqueos de recursos](/azure/azure-resource-manager/resource-group-lock-resources), se pueden restringir las operaciones en recursos de gran valor donde modificarlas o eliminarlas tendría un gran impacto en las aplicaciones o la infraestructura en la nube. Puede aplicar bloqueos a una suscripción, un recurso o un grupo de recursos. Normalmente, los bloqueos se aplican a recursos fundamentales como redes virtuales, puertas de enlace y cuentas de almacenamiento. 

Además, en estos momentos, admiten dos valores: CanNotDelete y ReadOnly. CanNotDelete significa que los usuarios (con los derechos adecuados) todavía pueden leer o modificar un recurso, pero no eliminarlo. ReadOnly significa que los usuarios autorizados no pueden eliminar o modificar un recurso.

Para crear o eliminar bloqueos de administración, debe tener acceso a las acciones `Microsoft.Authorization/*` o `Microsoft.Authorization/locks/*`.
Entre los roles integrados, solamente se conceden esas acciones al propietario y al administrador de acceso de usuarios.

> [!TIP]
> Las opciones de red principales deben protegerse con bloqueos. La eliminación accidental de una puerta de enlace o una VPN de sitio a sitio podría resultar desastroso para una suscripción de Azure. Azure no permite eliminar una red virtual que se esté utilizando, pero, por preocupación, recomendamos aplicar más restricciones. 
> 
> * Virtual Network: CanNotDelete
> * Grupo de seguridad de red: CanNotDelete
> * Directivas: CanNotDelete
> 
> Las directivas también resultan cruciales para el mantenimiento de los controles adecuados. Recomendamos aplicar un bloqueo **CanNotDelete** en las directivas que se están utilizando.

## <a name="core-networking-resources"></a>Recursos de red principales
El acceso a los recursos puede ser interno (dentro de la red corporativa) o externo (a través de Internet). Es muy fácil que los usuarios de una organización puedan colocar accidentalmente los recursos en la zona incorrecta y, posiblemente, abrirlos para acceder con intenciones malintencionado. Al igual que con los dispositivos locales, las empresas deben agregar los controles adecuados para asegurarse de que los usuarios de Azure toman las decisiones correctas. Para el gobierno de suscripciones, identificamos los recursos principales que proporcionan un control de acceso básico. Se trata de los siguientes:

* Las **redes virtuales** son objetos de contenedor de las subredes. Aunque no es estrictamente necesario, se suelen utilizar al conectar las aplicaciones a los recursos corporativos internos.
* Los **grupos de seguridad de red** son similares a los firewalls y proporcionan reglas de cómo un recurso puede "hablar" a través de la red. Proporcionan un control granular sobre cómo o bajo qué condición una subred (o máquina virtual) puede conectarse a Internet o a otras subredes de la misma red virtual.

![Redes principales](./_images/core-network.png)

> [!TIP]
> En el caso de las redes:
> * Cree redes virtuales específicas para las cargas de trabajo orientadas tanto a clientes externos como a internos. Con este enfoque se reduce la posibilidad de colocar accidentalmente máquinas virtuales que están diseñadas para cargas de trabajo internas en un espacio orientado a clientes externos.
> * Configure grupos de seguridad de red para limitar el acceso. Como mínimo, bloquean el acceso a Internet desde las redes virtuales internas y a la red corporativa desde las externas.
> 
> Estas sugerencias lo ayudarán a implementar recursos de red seguros.

## <a name="automation"></a>Automation
Administrar los recursos de manera individual es un proceso lento y potencialmente propenso a errores en determinadas operaciones. Azure proporciona varias funciones de automatización, como Azure Automation, Logic Apps y Azure Functions. [Azure Automation](/azure/automation/automation-intro) permite a los administradores crear y definir runbooks para administrar las tareas comunes de administración de recursos. Puede crear runbooks mediante un editor de código de PowerShell o uno de tipo gráfico. Puede generar flujos de trabajo complejos de varias fases. Azure Automation suele utilizarse para controlar tareas comunes, como detener recursos sin usar o crear recursos como respuesta a un desencadenador específico sin necesidad de intervención humana.

> [!TIP]
> En el caso de la automatización:
> * Cree una cuenta de Azure Automation y revise los runbooks disponibles (de línea de comandos y gráficos) en la [Galería de runbooks](/azure/automation/automation-runbook-gallery).
> * Importe y personalice los runbooks claves para utilizarlos.
> 
> Un escenario común es la capacidad de iniciar o apagar máquinas virtuales según una programación. Hay runbooks de ejemplo disponibles en la galería que controlan este escenario y explican cómo expandirlo.
> 
> 

## <a name="azure-security-center"></a>Azure Security Center
Quizás, uno de los principales impedimentos para adoptar la tecnología de nube han sido las preocupaciones por la seguridad. Los departamentos de riesgos de TI y seguridad tienen que asegurarse de que los recursos de Azure estén protegidos. 

[Azure Security Center](/azure/security-center/security-center-intro) proporciona una perspectiva central del estado de seguridad de los recursos de las suscripciones y ofrece recomendaciones que ayudan a evitar que se realicen ataques a los recursos. Asimismo, puede habilitar directivas más granulares (por ejemplo, aplicar directivas a grupos de recursos concretos que permiten a la empresa adaptar su postura con respecto al riesgo que están abordando). Finalmente, Azure Security Center es una plataforma abierta que permite a los asociados de Microsoft y fabricantes independientes de software crear software que se integre en Azure Security Center para mejorar sus funcionalidades. 

> [!TIP]
> Azure Security Center está habilitado de forma predeterminada en todas las suscripciones. Sin embargo, debe habilitar la recopilación de datos de máquinas virtuales para permitir que Azure Security Center instale su agente y empiece a recopilar datos.
> 
> ![recopilación de datos](./_images/data-collection.png)
> 
> 

## <a name="next-steps"></a>Pasos siguientes
* Ahora que ha obtenido información sobre el gobierno de suscripciones, es hora de ver estas recomendaciones en la práctica. Vea [Examples of implementing Azure subscription governance](azure-scaffold-examples.md) (Ejemplos de implementación de un sistema de gobierno de suscripciones).

> [!div class="nextstepaction"]
> [Implementación de un ejemplo](azure-scaffold-examples.md)
