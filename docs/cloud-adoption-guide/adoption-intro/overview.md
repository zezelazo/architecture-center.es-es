---
title: "Adopción de Azure: fundamentales"
description: "Describe el nivel de línea de base de conocimiento que se requiere en una empresa para adoptar Azure"
author: petertay
ms.openlocfilehash: e9421b610e4eb07a3ed37bca56e513b0689484ef
ms.sourcegitcommit: 9ba82cf84cee06ccba398ec04c51dab0e1ca8974
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/13/2018
---
# <a name="adopting-azure-foundational"></a>Adopción de Azure: fundamentales

La adopción de Azure es la primera fase de madurez de la organización para una empresa. Al final de esta fase, los empleados de su organización podrán implementar cargas de trabajo simples en Azure.

La siguiente lista incluye las tareas para llevar a cabo la fase de primera adopción. La lista es progresiva, por lo que se deben completar las tareas en orden. Si previamente ha completado la tarea, pase a la siguiente tarea en la lista. 

1. Descripción de los aspectos internos de Azure:
    - **Explicación:** [¿Cómo funciona Azure?](azure-explainer.md)
2. Descripción de la identidad digital de la empresa en Azure:
    - **Explicación:** [¿Qué es un inquilino de Azure Active Directory?](tenant-explainer.md)
    - **Procedimiento:** [Obtención de un inquilino de Azure Active Directory](/azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json)
    - **Guía:** [Diseño de inquilinos de Azure AD](tenant.md)
    - **Procedimiento:** [Adición de nuevos usuarios a Azure Active Directory](/azure/active-directory/add-users-azure-active-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json)    
3. Descripción de las suscripciones de Azure:
    - **Explicación:** [¿Qué es una suscripción de Azure?](subscription-explainer.md)
    - **Guía:** [Diseño de una suscripción de Azure](subscription.md)
4. Descripción de la administración de recursos de Azure: 
    - **Explicación:** [¿Qué es Azure Resource Manager?](resource-manager-explainer.md)
    - **Explicación:** [¿Qué es un grupo de recursos de Azure?](resource-group-explainer.md)
    - **Explicación:** [Descripción de acceso a los recursos de Azure](/azure/active-directory/active-directory-understanding-resource-access?toc=/azure/architecture/cloud-adoption-guide/toc.json)
    - **Procedimiento:** [Creación de un grupo de recursos de Azure con Azure Portal](/azure/azure-resource-manager/resource-group-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)
    - **Guía:** [Diseño de grupos de recursos de Azure](resource-group.md)
    - **Guía:** [Convenciones de nomenclatura para los recursos de Azure](/azure/architecture/best-practices/naming-conventions?toc=/azure/architecture/cloud-adoption-guide/toc.json)
5. Implementación de una arquitectura básica de Azure:
    - Obtenga información sobre los diferentes tipos de opciones de proceso de Azure como infraestructura como servicio (IaaS) y como plataforma como servicio (PaaS) en la [Introducción a las opciones de proceso de Azure](/azure/architecture/guide/technology-choices/compute-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json).
    - Ahora que conoce los diferentes tipos de opciones de proceso de Azure, elija una aplicación web de PaaS o una máquina virtual de IaaS como su primer recurso de Azure:
    - PaaS: Introducción a la plataforma como servicio:
        - **Procedimiento:** [Implementación de una aplicación web básica en Azure](/azure/app-service/app-service-web-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **Guía:** Procedimientos recomendados para implementar una [aplicación web básica](/azure/architecture/reference-architectures/app-service-web-app/basic-web-app?toc=/azure/architecture/cloud-adoption-guide/toc.json) en Azure
    - IaaS: Introducción a las redes virtuales:
        - **Explicación:** [Red virtual de Azure](/azure/virtual-network/virtual-networks-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **Procedimiento:** [Implementación de una red virtual en Azure mediante el portal](/azure/virtual-network/virtual-networks-create-vnet-arm-pportal?toc=/azure/architecture/cloud-adoption-guide/toc.json)
    - IaaS: Implementación de una carga de trabajo de una máquina virtual (VM) individual (Windows y Linux):
        - **Procedimiento:** [Implementación de una máquina virtual Windows en Azure con el portal](/azure/virtual-machines/windows/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **Guía:** [Procedimientos recomendados para ejecutar una máquina virtual Windows en Azure](/azure/architecture/reference-architectures/virtual-machines-windows/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **Procedimiento:** [Implementación de una máquina virtual Linux en Azure con el portal](/azure/virtual-machines/linux/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **Guía:** [Procedimientos recomendados para ejecutar una máquina virtual Linux en Azure](/azure/architecture/reference-architectures/virtual-machines-linux/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json)
