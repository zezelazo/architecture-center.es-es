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
# <a name="adopting-azure-foundational"></a><span data-ttu-id="e84ad-103">Adopción de Azure: fundamentales</span><span class="sxs-lookup"><span data-stu-id="e84ad-103">Adopting Azure: Foundational</span></span>

<span data-ttu-id="e84ad-104">La adopción de Azure es la primera fase de madurez de la organización para una empresa.</span><span class="sxs-lookup"><span data-stu-id="e84ad-104">Adopting Azure is the first stage of organizational maturity for an enterprise.</span></span> <span data-ttu-id="e84ad-105">Al final de esta fase, los empleados de su organización podrán implementar cargas de trabajo simples en Azure.</span><span class="sxs-lookup"><span data-stu-id="e84ad-105">By the end of this stage, people in your organization can deploy simple workloads to Azure.</span></span>

<span data-ttu-id="e84ad-106">La siguiente lista incluye las tareas para llevar a cabo la fase de primera adopción.</span><span class="sxs-lookup"><span data-stu-id="e84ad-106">The list below includes the tasks for completing the foundational adoption stage.</span></span> <span data-ttu-id="e84ad-107">La lista es progresiva, por lo que se deben completar las tareas en orden.</span><span class="sxs-lookup"><span data-stu-id="e84ad-107">The list is progressive so complete each task in order.</span></span> <span data-ttu-id="e84ad-108">Si previamente ha completado la tarea, pase a la siguiente tarea en la lista.</span><span class="sxs-lookup"><span data-stu-id="e84ad-108">If you have previously completed the task, move on the next task in the list.</span></span> 

1. <span data-ttu-id="e84ad-109">Descripción de los aspectos internos de Azure:</span><span class="sxs-lookup"><span data-stu-id="e84ad-109">Understand Azure internals:</span></span>
    - <span data-ttu-id="e84ad-110">**Explicación:** [¿Cómo funciona Azure?](azure-explainer.md)</span><span class="sxs-lookup"><span data-stu-id="e84ad-110">**Explainer:** [how does Azure work?](azure-explainer.md)</span></span>
2. <span data-ttu-id="e84ad-111">Descripción de la identidad digital de la empresa en Azure:</span><span class="sxs-lookup"><span data-stu-id="e84ad-111">Understand enterprise digital identity in Azure:</span></span>
    - <span data-ttu-id="e84ad-112">**Explicación:** [¿Qué es un inquilino de Azure Active Directory?](tenant-explainer.md)</span><span class="sxs-lookup"><span data-stu-id="e84ad-112">**Explainer:** [what is an Azure Active Directory Tenant?](tenant-explainer.md)</span></span>
    - <span data-ttu-id="e84ad-113">**Procedimiento:** [Obtención de un inquilino de Azure Active Directory](/azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json)</span><span class="sxs-lookup"><span data-stu-id="e84ad-113">**How to:** [get an Azure Active Directory Tenant](/azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json)</span></span>
    - <span data-ttu-id="e84ad-114">**Guía:** [Diseño de inquilinos de Azure AD](tenant.md)</span><span class="sxs-lookup"><span data-stu-id="e84ad-114">**Guidance:** [Azure AD tenant design guidance](tenant.md)</span></span>
    - <span data-ttu-id="e84ad-115">**Procedimiento:** [Adición de nuevos usuarios a Azure Active Directory](/azure/active-directory/add-users-azure-active-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json)</span><span class="sxs-lookup"><span data-stu-id="e84ad-115">**How to:** [add new users to Azure Active Directory](/azure/active-directory/add-users-azure-active-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json)</span></span>    
3. <span data-ttu-id="e84ad-116">Descripción de las suscripciones de Azure:</span><span class="sxs-lookup"><span data-stu-id="e84ad-116">Understand subscriptions in Azure:</span></span>
    - <span data-ttu-id="e84ad-117">**Explicación:** [¿Qué es una suscripción de Azure?](subscription-explainer.md)</span><span class="sxs-lookup"><span data-stu-id="e84ad-117">**Explainer:** [what is an Azure subscription?](subscription-explainer.md)</span></span>
    - <span data-ttu-id="e84ad-118">**Guía:** [Diseño de una suscripción de Azure](subscription.md)</span><span class="sxs-lookup"><span data-stu-id="e84ad-118">**Guidance:** [Azure subscription design](subscription.md)</span></span>
4. <span data-ttu-id="e84ad-119">Descripción de la administración de recursos de Azure:</span><span class="sxs-lookup"><span data-stu-id="e84ad-119">Understand resource management in Azure:</span></span> 
    - <span data-ttu-id="e84ad-120">**Explicación:** [¿Qué es Azure Resource Manager?](resource-manager-explainer.md)</span><span class="sxs-lookup"><span data-stu-id="e84ad-120">**Explainer:** [what is Azure Resource Manager?](resource-manager-explainer.md)</span></span>
    - <span data-ttu-id="e84ad-121">**Explicación:** [¿Qué es un grupo de recursos de Azure?](resource-group-explainer.md)</span><span class="sxs-lookup"><span data-stu-id="e84ad-121">**Explainer:** [what is an Azure resource group?](resource-group-explainer.md)</span></span>
    - <span data-ttu-id="e84ad-122">**Explicación:** [Descripción de acceso a los recursos de Azure](/azure/active-directory/active-directory-understanding-resource-access?toc=/azure/architecture/cloud-adoption-guide/toc.json)</span><span class="sxs-lookup"><span data-stu-id="e84ad-122">**Explainer:** [understanding resource access in Azure](/azure/active-directory/active-directory-understanding-resource-access?toc=/azure/architecture/cloud-adoption-guide/toc.json)</span></span>
    - <span data-ttu-id="e84ad-123">**Procedimiento:** [Creación de un grupo de recursos de Azure con Azure Portal](/azure/azure-resource-manager/resource-group-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)</span><span class="sxs-lookup"><span data-stu-id="e84ad-123">**How to:** [create an Azure resource group using the Azure portal](/azure/azure-resource-manager/resource-group-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)</span></span>
    - <span data-ttu-id="e84ad-124">**Guía:** [Diseño de grupos de recursos de Azure](resource-group.md)</span><span class="sxs-lookup"><span data-stu-id="e84ad-124">**Guidance:** [Azure resource group design guidance](resource-group.md)</span></span>
    - <span data-ttu-id="e84ad-125">**Guía:** [Convenciones de nomenclatura para los recursos de Azure](/azure/architecture/best-practices/naming-conventions?toc=/azure/architecture/cloud-adoption-guide/toc.json)</span><span class="sxs-lookup"><span data-stu-id="e84ad-125">**Guidance:** [naming conventions for Azure resources](/azure/architecture/best-practices/naming-conventions?toc=/azure/architecture/cloud-adoption-guide/toc.json)</span></span>
5. <span data-ttu-id="e84ad-126">Implementación de una arquitectura básica de Azure:</span><span class="sxs-lookup"><span data-stu-id="e84ad-126">Deploy a basic Azure architecture:</span></span>
    - <span data-ttu-id="e84ad-127">Obtenga información sobre los diferentes tipos de opciones de proceso de Azure como infraestructura como servicio (IaaS) y como plataforma como servicio (PaaS) en la [Introducción a las opciones de proceso de Azure](/azure/architecture/guide/technology-choices/compute-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json).</span><span class="sxs-lookup"><span data-stu-id="e84ad-127">Learn about the different types of Azure compute options such as Infrastructure-as-a-Service (IaaS) and Platform-as-a-Service (PaaS) in the [overview of Azure compute options](/azure/architecture/guide/technology-choices/compute-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json).</span></span>
    - <span data-ttu-id="e84ad-128">Ahora que conoce los diferentes tipos de opciones de proceso de Azure, elija una aplicación web de PaaS o una máquina virtual de IaaS como su primer recurso de Azure:</span><span class="sxs-lookup"><span data-stu-id="e84ad-128">Now that you understand the different types of Azure compute options, pick either a PaaS web application or IaaS virtual machine as your first resource in Azure:</span></span>
    - <span data-ttu-id="e84ad-129">PaaS: Introducción a la plataforma como servicio:</span><span class="sxs-lookup"><span data-stu-id="e84ad-129">PaaS: Introduction to Platform as a Service:</span></span>
        - <span data-ttu-id="e84ad-130">**Procedimiento:** [Implementación de una aplicación web básica en Azure](/azure/app-service/app-service-web-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json)</span><span class="sxs-lookup"><span data-stu-id="e84ad-130">**How to:** [deploy a basic web application to Azure](/azure/app-service/app-service-web-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json)</span></span>
        - <span data-ttu-id="e84ad-131">**Guía:** Procedimientos recomendados para implementar una [aplicación web básica](/azure/architecture/reference-architectures/app-service-web-app/basic-web-app?toc=/azure/architecture/cloud-adoption-guide/toc.json) en Azure</span><span class="sxs-lookup"><span data-stu-id="e84ad-131">**Guidance:** proven practices for deploying a [basic web application](/azure/architecture/reference-architectures/app-service-web-app/basic-web-app?toc=/azure/architecture/cloud-adoption-guide/toc.json) to Azure</span></span>
    - <span data-ttu-id="e84ad-132">IaaS: Introducción a las redes virtuales:</span><span class="sxs-lookup"><span data-stu-id="e84ad-132">IaaS: Introduction to Virtual Networking:</span></span>
        - <span data-ttu-id="e84ad-133">**Explicación:** [Red virtual de Azure](/azure/virtual-network/virtual-networks-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json)</span><span class="sxs-lookup"><span data-stu-id="e84ad-133">**Explainer:** [Azure virtual network](/azure/virtual-network/virtual-networks-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json)</span></span>
        - <span data-ttu-id="e84ad-134">**Procedimiento:** [Implementación de una red virtual en Azure mediante el portal](/azure/virtual-network/virtual-networks-create-vnet-arm-pportal?toc=/azure/architecture/cloud-adoption-guide/toc.json)</span><span class="sxs-lookup"><span data-stu-id="e84ad-134">**How to:** [deploy a Virtual Network to Azure using the portal](/azure/virtual-network/virtual-networks-create-vnet-arm-pportal?toc=/azure/architecture/cloud-adoption-guide/toc.json)</span></span>
    - <span data-ttu-id="e84ad-135">IaaS: Implementación de una carga de trabajo de una máquina virtual (VM) individual (Windows y Linux):</span><span class="sxs-lookup"><span data-stu-id="e84ad-135">IasS: Deploy a single virtual machine(VM) workload (Windows and Linux):</span></span>
        - <span data-ttu-id="e84ad-136">**Procedimiento:** [Implementación de una máquina virtual Windows en Azure con el portal](/azure/virtual-machines/windows/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)</span><span class="sxs-lookup"><span data-stu-id="e84ad-136">**How to:** [deploy a Windows VM to Azure with the portal](/azure/virtual-machines/windows/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)</span></span>
        - <span data-ttu-id="e84ad-137">**Guía:** [Procedimientos recomendados para ejecutar una máquina virtual Windows en Azure](/azure/architecture/reference-architectures/virtual-machines-windows/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json)</span><span class="sxs-lookup"><span data-stu-id="e84ad-137">**Guidance:** [proven practices for running a Windows VM on Azure](/azure/architecture/reference-architectures/virtual-machines-windows/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json)</span></span>
        - <span data-ttu-id="e84ad-138">**Procedimiento:** [Implementación de una máquina virtual Linux en Azure con el portal](/azure/virtual-machines/linux/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)</span><span class="sxs-lookup"><span data-stu-id="e84ad-138">**How to:** [deploy a Linux VM to Azure with the portal](/azure/virtual-machines/linux/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)</span></span>
        - <span data-ttu-id="e84ad-139">**Guía:** [Procedimientos recomendados para ejecutar una máquina virtual Linux en Azure](/azure/architecture/reference-architectures/virtual-machines-linux/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json)</span><span class="sxs-lookup"><span data-stu-id="e84ad-139">**Guidance:** [proven practices for running a Linux VM on Azure](/azure/architecture/reference-architectures/virtual-machines-linux/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json)</span></span>