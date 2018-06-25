---
title: 'Adopción de Azure: intermedio'
description: Describe el nivel intermedio de conocimiento que una empresa necesita para adoptar Azure.
author: petertay
ms.openlocfilehash: 227d9558647ed8076b2832d95e192f2f0c43b9db
ms.sourcegitcommit: 26b04f138a860979aea5d253ba7fecffc654841e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/19/2018
ms.locfileid: "36206368"
---
# <a name="azure-cloud-adoption-guide-intermediate-overview"></a>Guía de adopción de la nube de Azure: Introducción al nivel intermedio

En la fase de adopción fundamental, se introdujeron los conceptos básicos del gobierno de los recursos de Azure. La fase de fundación se diseñó para ayudarle a comenzar su viaje de adopción de Azure y le guió por la implementación de una carga de trabajo simple con un único equipo pequeño. En realidad, la mayoría de las grandes organizaciones tienen muchos equipos que trabajan en muchas cargas de trabajo diferentes al mismo tiempo. Como cabría esperar, un modelo de gobierno simple no es suficiente para administrar escenarios de organización y de desarrollo más complejos.

La fase intermedia de la adopción de Azure se centra en es diseñar el modelo de gobierno para varios equipos que trabajan en varias cargas de trabajo de desarrollo de Azure nuevas.  

Los destinatarios de esta fase de la guía son los siguientes roles dentro de su organización:
- *Finanzas:* propietario del compromiso financiero con Azure, responsable del desarrollo de directivas y procedimientos para realizar el seguimiento de los costos de consumo de recursos, incluidos los contracargos y la facturación.
- *TI central:* responsable del gobierno de los recursos en la nube de su organización, incluida la administración de recursos y el acceso a los mismo, así como el estado y la supervisión de las cargas de trabajo.
- *Propietario de la infraestructura compartida*: roles técnicos responsables de la conectividad de red del entorno local a la nube.
- *Operaciones de seguridad*: responsable de implementar la directiva de seguridad necesaria para ampliar el límite de seguridad local para incluir a Azure. También puede poseer infraestructura de seguridad en Azure para almacenar secretos.
- *Propietario de carga de trabajo:* responsable de la publicación de una carga de trabajo en Azure. Dependiendo de la estructura de los equipos de desarrollo de su organización, podría tratarse de un responsable de desarrollo, un responsable de administración de programas o un responsable de ingeniería. Parte del proceso de publicación puede incluir la implementación de recursos en Azure.
  - *Colaborador de carga de trabajo:* responsable que contribuye a la publicación de una carga de trabajo en Azure. Puede requerir acceso de lectura a los recursos de Azure con fines de supervisión u optimización del rendimiento. No requieren permisos para crear, actualizar o eliminar los recursos.

## <a name="section-1-azure-concepts-for-multiple-workloads-and-multiple-teams"></a>Sección 1: Conceptos de Azure para varias cargas de trabajo y varios equipos

En la fase de adopción fundamental, aprendió algunos conceptos básicos sobre las características internas de Azure y cómo se crean, se leen, se actualizan y se eliminan los recursos. También aprendió acerca de la identidad y que Azure solo confía en Azure Active Directory (AD) para autenticar y autorizar a los usuarios que necesitan acceder a esos recursos.

También comenzó a ver cómo configurar las herramientas de gobierno de Azure para administrar el uso que su organización hace de los recursos de Azure. En la fase de fundación, explicamos cómo gobernar el acceso de un único equipo a los recursos necesarios para implementar una carga de trabajo simple. En realidad, su organización va a estar formada por varios equipos que trabajarán simultáneamente en varias cargas de trabajo. 

Antes de comenzar, veamos lo que realmente significa el término **carga de trabajo**. Es un término que normalmente se utiliza para definir una unidad arbitraria de funcionalidad, como una aplicación o un servicio. Pensamos en una carga de trabajo como los artefactos de código que se implementan en un servidor, así como otros servicios, como una base de datos, que sean necesarios. Se trata de una definición útil para una aplicación o servicio local pero, en la nube, necesitamos ampliarla. 

En la nube, una carga de trabajo no solo abarca todos los artefactos, también incluye los recursos en la nube. Incluimos los recursos en la nube como parte de nuestra definición debido a un concepto conocido como **infraestructura como código**. Como ya ha visto en "cómo funciona Azure", los recursos de Azure se implementan mediante un servicio de orquestador. El servicio de orquestador expone esta funcionalidad a través de una API web y esta API web se puede llamar con varias herramientas, como Powershell, la interfaz de línea de comandos (CLI) de Azure o Azure Portal. Esto significa que podemos especificar nuestros recursos en un archivo legible por máquina que se puede almacenar junto con los artefactos de código asociados a la aplicación.

Esto nos permite definir una carga de trabajo en términos de artefactos de código y los recursos en la nube necesarios, lo que nos permite **aislar** más nuestras cargas de trabajo. Las cargas de trabajo pueden aislarse según la forma en que se organizan los recursos, por su topología de red o por otros atributos. El objetivo del aislamiento de las cargas de trabajo es asociar recursos específicos de la carga de trabajo a un equipo para que este pueda administrar todos los aspectos de esos recursos de forma independiente. Esto permite que varios equipos compartan los servicios de administración de recursos de Azure y evitar la eliminación o modificación accidental de los recursos de los demás.

Este aislamiento también incluye otro concepto conocido como [DevOps](https://azure.microsoft.com/solutions/devops/). DevOps incluye las prácticas de desarrollo de software que incluyen tanto el desarrollo de software y como las operaciones de TI anteriores, pero agrega el uso de la automatización tanto como sea posible. Uno de los principios de DevOps se conoce como integración continua y entrega continua (CI/CD). La integración continua hace referencia a los procesos de compilación automatizados que se ejecutan cada vez que un desarrollador confirma un cambio en el código, y la entrega continua hace referencia a los procesos automatizados que implementa este código en los diversos **entornos**, como un **entorno de desarrollo** para realizar pruebas o un **entorno de producción** para la implementación final.

## <a name="section-2-governance-design-for-multiple-teams-and-multiple-workloads"></a>Sección 2: Diseño del gobierno para varios equipos y varias cargas de trabajo

En la [fase de fundación](/azure/architecture/cloud-adoption-guide/adoption-intro/overview) de la guía de adopción de la nube de Azure, se presentó el concepto de gobierno en la nube. Aprendió a diseñar un modelo de gobierno simple para un solo equipo que trabaja en una sola carga de trabajo. 

En la fase intermedia, la [guía de diseño de gobierno](governance-design-guide.md) amplía los conceptos básicos para agregar varios equipos, varias cargas de trabajo y varios entornos. Una vez que haya visto los ejemplos del documento, puede aplicar los principios de diseño para diseñar e implementar el modelo de gobierno de su organización.

## <a name="section-3-implementing-a-resource-management-model"></a>Sección 3: Implementación de un modelo de administración de recursos

El modelo de gobierno en la nube de su organización representa la intersección entre las herramientas de administración del acceso a los recursos de Azure, los usuarios y las reglas de administración de acceso que haya definido. En la guía de diseño del gobierno, aprendió a varios modelos diferentes para gobernar el acceso a los recursos de Azure. Ahora veremos los pasos necesarios para implementar el modelo de administración de recursos con una suscripción para cada uno de los entornos de **infraestructura compartida**, **producción** y **desarrollo** con la guía de diseño. Tendremos un **propietario de la suscripción** para los tres entornos. Cada carga de trabajo se aislará en un **grupo de recursos** y se agregará **propietario de carga de trabajo** con el rol de **colaborador**.

> [!NOTE]
> Lectura la [introducción al acceso a los recursos de Azure][understand-resource-access-in-azure] para obtener más información sobre la relación entre las cuentas y las suscripciones de Azure. 

Siga estos pasos:

1. Cree una [cuenta de Azure](/azure/active-directory/sign-up-organization) si su organización aún no tiene una. La persona que se suscribe a la cuenta de Azure se convierte en el administrador de la cuenta de Azure y los responsables de su organización deben seleccionar la persona que asumirá este rol. Esta persona será responsable de:
    * Crear suscripciones.
    * Crear y administrar los inquilinos de [Azure Active Directory (AD)](/azure/active-directory/active-directory-whatis) que almacenan las identidades de usuario de esas suscripciones.    
2. El equipo directivo de su organización decide qué personas son responsables de:
    * Administrar las identidades de usuario; de forma predeterminada, se crea un [inquilino de Azure AD](/azure/active-directory/develop/active-directory-howto-tenant) cuando se crea la cuenta de Azure de su organización y el administrador de la cuenta se agrega como [administrador global de Azure AD](/azure/active-directory/active-directory-assign-admin-roles-azure-portal#details-about-the-global-administrator-role). Su organización puede elegir que otro usuario administre las identidades de usuario; para ello, puede [asignar el rol de administrador global de Azure AD para ese usuario](/azure/active-directory/active-directory-users-assign-role-azure-portal). 
    * Suscripciones, lo que significa que estos usuarios son responsables de:
        * Administrar los costos asociados con el uso de recursos en esa suscripción.
        * Implementar y mantener el modelo de permisos mínimos para el acceso a los recursos.
        * Realizar un seguimiento de los límites de servicio.
    * Servicios de infraestructura compartida (si su organización decide usar este modelo), lo que significa que este usuario es responsable de:
        * La conectividad del entorno local a la red de Azure. 
        * La propiedad de la conectividad de red dentro de Azure mediante el emparejamiento de redes virtuales.
    * Propietarios de cargas de trabajo. 
3. El administrador global de Azure AD [crea las cuentas de usuario nuevas](/azure/active-directory/add-users-azure-active-directory) para:
    * La persona que será el **propietario de la suscripción** para cada suscripción asociada con cada entorno. Tenga en cuenta que esto es necesario solo si el **administrador de servicios** de la suscripción no será responsable de administrar el acceso a los recursos para cada suscripción o entorno.
    * La persona que será el **usuario de operaciones de red**.
    * Las personas que serán **propietarios de cargas de trabajo**.
4. El administrador de la cuenta de Azure crea las siguientes tres suscripciones con el [portal de cuentas de Azure](https://account.azure.com):
    * Una suscripción para el entorno de **infraestructura compartida**.
    * Una suscripción para el entorno de **producción**. 
    * Una suscripción para el entorno de **desarrollo**. 
5. El administrador de la cuenta de Azure [agrega el propietario del servicio de suscripción a cada suscripción](/azure/billing/billing-add-change-azure-subscription-administrator#add-an-rbac-owner-admin-for-a-subscription-in-azure-portal).
6. Cree un proceso de aprobación para que **propietarios de cargas de trabajo** soliciten la creación de grupos de recursos. El proceso de aprobación se puede implementar de muchas maneras, por ejemplo, por correo electrónico, o bien puede usar una herramienta de administración de procesos como [flujos de trabajo de Sharepoint](https://support.office.com/article/introduction-to-sharepoint-workflow-07982276-54e8-4e17-8699-5056eff4d9e3). El proceso de aprobación puede seguir estos pasos:  
    * El **propietario de la carga de trabajo** prepara una lista de materiales de los recursos de Azure que necesita en el entorno de **desarrollo**, de **producción** o ambos, y la envía al **propietario de la suscripción**.
    * El **propietario de la suscripción** revisa la lista de materiales y valida los recursos solicitados para asegurarse de que son los adecuados para el uso previsto; por ejemplo, comprueba que los [ tamaños de máquina virtual](/azure/virtual-machines/windows/sizes) solicitados son los correctos.
    * Si no se aprueba la solicitud, el **propietario de la carga de trabajo** recibe una notificación. Si se aprueba la solicitud, el **propietario de la suscripción** [crea el grupo de recursos solicitado](/azure/azure-resource-manager/resource-group-portal#manage-resource-groups) según las [convenciones de nomenclatura](/azure/architecture/best-practices/naming-conventions) de la organización, [agrega el **propietario de carga de trabajo**](/azure/role-based-access-control/role-assignments-portal#add-access) con el rol de [**colaborador**](/azure/role-based-access-control/built-in-roles#contributor) y notifica al **propietario de carga de trabajo** que se ha creado el grupo de recursos.
7. Crear un proceso de aprobación para que los propietarios de cargas de trabajo soliciten una conexión de emparejamiento de red virtual desde el propietario de la infraestructura compartida. Al igual que con el paso anterior, este proceso de aprobación se puede implementar mediante correo electrónico o con una herramienta de administración de procesos.

Ahora que ha implementado el modelo de gobierno, puede implementar los servicios de infraestructura compartida.

## <a name="section-4-deploy-shared-infrastructure-services"></a>Sección 4: implementación de los servicios de infraestructura compartida

Hay varias [arquitecturas de referencia de red híbrida](/azure/architecture/reference-architectures/hybrid-networking/) que su organización puede utilizar para conectar su red local con Azure. Cada una de estas arquitecturas de referencia incluye una implementación que requiere un identificador de suscripción. Durante la implementación, especifique el identificador de suscripción para la suscripción asociada con su entorno de **infraestructura compartida**. También debe editar los archivos de plantilla para especificar el grupo de recursos administrado por el usuario de **operaciones de red**, o bien puede usar los grupos de recursos predeterminados de la implementación y agregar el usuario de **operaciones de red**  con el rol de **colaborador** .

<!-- links -->
[understand-resource-access-in-azure]: /azure/role-based-access-control/rbac-and-directory-admin-roles