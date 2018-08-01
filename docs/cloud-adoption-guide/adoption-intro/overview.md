---
title: 'Adopción de Azure: fundamentales'
description: Describe el nivel de conocimiento básico que se necesita en una empresa para adoptar Azure
author: petertay
ms.openlocfilehash: b5a0a4a2c4ed1d06c97774b0eca643a89a5a2110
ms.sourcegitcommit: c704d5d51c8f9bbab26465941ddcf267040a8459
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2018
ms.locfileid: "39229174"
---
# <a name="adopting-microsoft-azure-foundational"></a>Adopción de Microsoft Azure: fundación

Para una organización que no está familiarizada con las tecnologías de nube, puede resultar difícil decidir cuál es el lugar idóneo para comenzar su proceso de adopción. El objetivo de la fase de adopción de fundación es proporcionar un punto de partida. Cuando las personas de la organización hayan trabajado en esta fase, tendrán todos los conocimientos y habilidades necesarios para implementar los recursos de proceso para una carga de trabajo simple en Azure. 

> [!NOTE]
> Esta guía no incluye el desarrollo de aplicaciones. Para más información sobre cómo desarrollar aplicaciones en Azure, consulte la [Guía de arquitectura de aplicaciones de Azure](/azure/architecture/guide/).

Los destinatarios de esta fase de la guía son los siguientes roles dentro de su organización:

- *Finanzas:* propietario del compromiso financiero con Azure, responsable del desarrollo de directivas y procedimientos para realizar el seguimiento de los costos de consumo de recursos, incluidos los contracargos y la facturación.
- *TI central:* responsable del gobierno de los recursos en la nube de su organización, incluida la administración de recursos y el acceso a los mismo, así como el estado y la supervisión de las cargas de trabajo.
- *Propietarios de cargas de trabajo:* todos los roles de desarrollo que están implicados en la implementación de cargas de trabajo en Azure, incluidos desarrolladores, evaluadores e ingenieros de compilación.

## <a name="section-1-azure-basics"></a>Sección 1: Conceptos básicos de Azure

En esta sección de introducción está destinada a los roles *Finanzas* y *TI central*. El objetivo de esta sección es adquirir los conocimientos básicos de [cómo funciona Azure](azure-explainer.md) como preparación para aprender el [concepto de gobierno en la nube](governance-explainer.md). También podría resultar útil que los *propietarios de cargas de trabajo* de su organización revisen este contenido para ayudarles a comprender cómo se administra el acceso a los recursos.

## <a name="section-2-governance-design-guide"></a>Sección 2: Guía de diseño de gobierno

Ahora que ya comprende cómo funciona Azure y conoce los conceptos básicos de gobierno en la nube, el primer paso para adoptar Azure es aprender sobre la [administración del acceso a los recursos](azure-resource-access.md) en Azure. En este artículo se describen los servicios de Azure que permiten realizar solicitudes de acceso a los recursos y los controles utilizados para validar las solicitudes.

El siguiente paso consiste en aprender cómo [diseñar un modelo de gobierno](governance-how-to.md) para un único equipo. En este artículo se describe cómo configurar los servicios de administración de acceso a los recursos y los controles que ha aprendido anteriormente.

## <a name="section-3-implementing-a-basic-resource-access-management-model"></a>Sección 3: Implementación de un modelo básico de administración de acceso a los recursos

El último paso en el proceso de adopción es aprender a implementar el modelo de gobierno diseñado anteriormente. 

Para empezar, la organización necesita una cuenta de Azure. Si su organización ya tiene un [Contrato Enterprise de Microsoft](https://www.microsoft.com/licensing/licensing-programs/enterprise.aspx) que no incluye Azure, se puede agregar Azure mediante un compromiso monetario por adelantado. Consulte las [licencias de Azure para la empresa](https://azure.microsoft.com/pricing/enterprise-agreement/) para más información. 

Cuando cree la cuenta de Azure, especifique el usuario de su organización que será el **propietario de la cuenta** de Azure. Después, se crea un inquilino de Azure Active Directory (Azure AD) de forma predeterminada. El **propietario de la cuenta** de Azure debe [crear la cuenta de usuario](/azure/active-directory/add-users-azure-active-directory) para el usuario de la organización que sea el **propietario de cargas de trabajo**. 

A continuación, el **propietario de la cuenta** de Azure debe [crear una suscripción](https://docs.microsoft.com/partner-center/create-a-new-subscription) y [asociar el inquilino de Azure AD](/azure/active-directory/fundamentals/active-directory-how-subscriptions-associated-directory) a ella.

Por último, con la suscripción creada y el inquilino de Azure AD asociado a ella, puede [agregar el **propietario de cargas de trabajo** a la suscripción con el rol integrado **propietario**](/azure/billing/billing-add-change-azure-subscription-administrator#add-an-rbac-owner-for-a-subscription-in-azure-portal).

## <a name="section-4-deploy-a-basic-workload-architecture-to-azure"></a>Sección 4: Implementación de una arquitectura de carga de trabajo básica en Azure

El destinatario de esta sección es el rol *propietario de cargas de trabajo*. Los *propietarios de cargas de trabajo* definen los requisitos de red y de proceso para sus cargas de trabajo, seleccionan los recursos correctos para cumplir con estos requisitos y los implementan en Azure. 

Para la fase de adopción de fundación, el propietario de cargas de trabajo puede seleccionar una aplicación web básica o una red virtual y una máquina virtual. Para más información acerca de estos tipos diferentes de opciones de proceso de Azure, consulte la [introducción a las opciones de proceso de Azure](/azure/architecture/guide/technology-choices/compute-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json).

Independientemente de qué opción de proceso se seleccione, cada una de estas implementaciones requiere un **grupo de recursos**. Su **propietario de cargas de trabajo** debe [crear el grupo de recursos](/azure/azure-resource-manager/vs-azure-tools-resource-groups-deployment-projects-create-deploy). Como parte de la implementación, el **propietario de cargas de trabajo** especifica un nombre para el grupo de recursos. Este nombre se usa en las secciones siguientes.

### <a name="basic-web-application-paas"></a>Aplicación web básica (Paas)

Para una aplicación web básica, seleccione una de las guías de inicio rápido en 5 minutos en la [documentación de Web Apps](/azure/app-service?toc=/azure/architecture/cloud-adoption-guide/toc.json) y siga los pasos. 

> [!NOTE]
> Algunas de las guías de inicio rápido implementarán un grupo de recursos de forma predeterminada. En este caso, el **propietario de cargas de trabajo** no necesita crear un grupo de recursos de forma explícita. En caso contrario, implemente la aplicación web en el grupo de recursos creado anteriormente.

Una vez implementada la carga de trabajo simple, puede aprender más sobre los procedimientos recomendados para implementar una [aplicación web básica](/azure/architecture/reference-architectures/app-service-web-app/basic-web-app?toc=/azure/architecture/cloud-adoption-guide/toc.json) en Azure.

### <a name="single-windows-or-linux-vm-iaas"></a>Una sola máquina virtual Windows o Linux (IaaS)

Para una carga de trabajo simple que se ejecuta en una máquina virtual, el primer paso es implementar una red virtual. Todos los recursos de IaaS en Azure tales como máquinas virtuales, equilibradores de carga y puertas de enlace requieren una red virtual. Vea más información sobre las [redes virtuales de Azure](/azure/virtual-network/virtual-networks-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json) y, a continuación, siga los pasos para [implementar una red virtual en Azure mediante el portal](/azure/virtual-network/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json). Al especificar la configuración de la red virtual en Azure Portal, especifique el nombre del grupo de recursos creado anteriormente.

El siguiente paso es decidir si va a implementar una sola máquina virtual Windows o Linux. En el caso de una máquina virtual Windows, siga los pasos para [implementar una máquina virtual Windows en Azure con el portal](/azure/virtual-machines/windows/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json). De nuevo, al especificar la configuración de la máquina virtual en Azure Portal, especifique el nombre del grupo de recursos creado anteriormente.

Después de seguir los pasos e implementar la máquina virtual, puede ver los [procedimientos recomendados para ejecutar una máquina virtual Windows en Azure](/azure/architecture/reference-architectures/virtual-machines-windows/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json). En el caso de una máquina virtual Linux, siga los pasos para [implementar una máquina virtual Linux en Azure con el portal](/azure/virtual-machines/linux/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json). También puede ver los [procedimientos recomendados para ejecutar una máquina virtual Linux en Azure](/azure/architecture/reference-architectures/virtual-machines-linux/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json).

## <a name="next-steps"></a>Pasos siguientes

La siguiente fase de la preparación para la nube es la [**fase intermedia**](../intermediate-stage/overview.md). En la fase intermedia, aprenderá a ampliar su red local ejecutando varias cargas de trabajo y modelos de gobierno para varios equipos.