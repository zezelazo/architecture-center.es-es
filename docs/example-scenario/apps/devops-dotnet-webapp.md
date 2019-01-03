---
title: Diseño de una canalización de CI/CD con Azure DevOps
titleSuffix: Azure Example Scenarios
description: Compilación y publicación de una aplicación .NET en Azure Web Apps con Azure DevOps.
author: christianreddington
ms.date: 12/06/2018
ms.custom:
- fasttrack
- seodec18
ms.openlocfilehash: ae2dddd7567c6b69f936b3b9c9339313389e3bf6
ms.sourcegitcommit: bb7fcffbb41e2c26a26f8781df32825eb60df70c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/20/2018
ms.locfileid: "53643805"
---
# <a name="design-a-cicd-pipeline-using-azure-devops"></a><span data-ttu-id="377d7-103">Diseño de una canalización de CI/CD con Azure DevOps</span><span class="sxs-lookup"><span data-stu-id="377d7-103">Design a CI/CD pipeline using Azure DevOps</span></span>

<span data-ttu-id="377d7-104">Este escenario proporciona una guía de arquitectura y diseño para la creación de una canalización de integración continua (CI) y de implementación continua (CD).</span><span class="sxs-lookup"><span data-stu-id="377d7-104">This scenario provides architecture and design guidance for building a continuous integration (CI) and continuous deployment (CD) pipeline.</span></span> <span data-ttu-id="377d7-105">En este ejemplo, la canalización de CI/CD implementa una aplicación web de .NET de dos niveles en Azure App Service.</span><span class="sxs-lookup"><span data-stu-id="377d7-105">In this example, the CI/CD pipeline deploys a two-tier .NET web application to the Azure App Service.</span></span>

<span data-ttu-id="377d7-106">La migración a procesos modernos de CI/CD ofrece muchos beneficios para la creación, implementación, prueba y supervisión de aplicaciones.</span><span class="sxs-lookup"><span data-stu-id="377d7-106">Migrating to modern CI/CD processes provides many benefits for application builds, deployments, testing, and monitoring.</span></span> <span data-ttu-id="377d7-107">Al utilizar Azure DevOps junto con otros servicios como App Service, las organizaciones pueden centrarse en el desarrollo de sus aplicaciones en lugar de en la administración de la infraestructura de soporte.</span><span class="sxs-lookup"><span data-stu-id="377d7-107">By utilizing Azure DevOps along with other services such as App Service, organizations can focus on the development of their apps rather than the management of the supporting infrastructure.</span></span>

## <a name="relevant-use-cases"></a><span data-ttu-id="377d7-108">Casos de uso pertinentes</span><span class="sxs-lookup"><span data-stu-id="377d7-108">Relevant use cases</span></span>

<span data-ttu-id="377d7-109">Considere DevOps Azure y los procesos de CI/CD para:</span><span class="sxs-lookup"><span data-stu-id="377d7-109">Consider Azure DevOps and CI/CD processes for:</span></span>

- <span data-ttu-id="377d7-110">Aceleración del desarrollo de aplicaciones y ciclos de vida de desarrollo</span><span class="sxs-lookup"><span data-stu-id="377d7-110">Accelerating application development and development life cycles</span></span>
- <span data-ttu-id="377d7-111">Generación de calidad y coherencia en un proceso automatizado de compilación y lanzamiento</span><span class="sxs-lookup"><span data-stu-id="377d7-111">Building quality and consistency into an automated build and release process</span></span>
- <span data-ttu-id="377d7-112">Aumento de la estabilidad y el tiempo de actividad de la aplicación</span><span class="sxs-lookup"><span data-stu-id="377d7-112">Increasing application stability and uptime</span></span>

## <a name="architecture"></a><span data-ttu-id="377d7-113">Arquitectura</span><span class="sxs-lookup"><span data-stu-id="377d7-113">Architecture</span></span>

![Diagrama de la arquitectura de los componentes de Azure implicados en un escenario de DevOps con Azure DevOps y Azure App Service][architecture]

<span data-ttu-id="377d7-115">Los datos fluyen por el escenario de la siguiente manera:</span><span class="sxs-lookup"><span data-stu-id="377d7-115">The data flows through the scenario as follows:</span></span>

1. <span data-ttu-id="377d7-116">Un desarrollador cambia el código fuente de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="377d7-116">A developer changes application source code.</span></span>
2. <span data-ttu-id="377d7-117">El código de la aplicación, incluido el archivo web.config, se envía al repositorio de código fuente de Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="377d7-117">Application code including the web.config file is committed to the source code repository in Azure Repos.</span></span>
3. <span data-ttu-id="377d7-118">La integración continua desencadena las pruebas unitarias y la compilación de la aplicación mediante Azure Test Plans.</span><span class="sxs-lookup"><span data-stu-id="377d7-118">Continuous integration triggers application build and unit tests using Azure Test Plans.</span></span>
4. <span data-ttu-id="377d7-119">La implementación continua en Azure Pipelines desencadena la implementación automatizada de los artefactos de la aplicación *con valores de configuración específicos del entorno*.</span><span class="sxs-lookup"><span data-stu-id="377d7-119">Continuous deployment within Azure Pipelines triggers an automated deployment of application artifacts *with environment-specific configuration values*.</span></span>
5. <span data-ttu-id="377d7-120">Los artefactos se implementan en Azure App Service.</span><span class="sxs-lookup"><span data-stu-id="377d7-120">The artifacts are deployed to Azure App Service.</span></span>
6. <span data-ttu-id="377d7-121">Azure Application Insights recopila y analiza datos de mantenimiento, rendimiento y uso.</span><span class="sxs-lookup"><span data-stu-id="377d7-121">Azure Application Insights collects and analyzes health, performance, and usage data.</span></span>
7. <span data-ttu-id="377d7-122">Los desarrolladores supervisan y administran la información sobre el estado, el rendimiento y el uso.</span><span class="sxs-lookup"><span data-stu-id="377d7-122">Developers monitor and mange health, performance, and usage information.</span></span>
8. <span data-ttu-id="377d7-123">La información de trabajos pendientes se utiliza para priorizar nuevas características y correcciones de errores mediante Azure Boards.</span><span class="sxs-lookup"><span data-stu-id="377d7-123">Backlog information is used to prioritize new features and bug fixes using Azure Boards.</span></span>

### <a name="components"></a><span data-ttu-id="377d7-124">Componentes</span><span class="sxs-lookup"><span data-stu-id="377d7-124">Components</span></span>

- <span data-ttu-id="377d7-125">[Azure DevOps][vsts] es un servicio para administrar el ciclo de vida del desarrollo de un extremo a otro, desde el planeamiento y administración del proyecto a la administración del código, continuando con la compilación y el lanzamiento.</span><span class="sxs-lookup"><span data-stu-id="377d7-125">[Azure DevOps][vsts] is a service for managing your development life cycle end-to-end &mdash; from planning and project management, to code management, and continuing to build and release.</span></span>

- <span data-ttu-id="377d7-126">[Azure Web Apps][web-apps] es un servicio PaaS para hospedar aplicaciones web, API REST y back-ends para dispositivos móviles.</span><span class="sxs-lookup"><span data-stu-id="377d7-126">[Azure Web Apps][web-apps] is a PaaS service for hosting web applications, REST APIs, and mobile back ends.</span></span> <span data-ttu-id="377d7-127">Aunque este artículo se centra en. NET, hay varias opciones de plataforma de desarrollo adicionales compatibles.</span><span class="sxs-lookup"><span data-stu-id="377d7-127">While this article focuses on .NET, there are several additional development platform options supported.</span></span>

- <span data-ttu-id="377d7-128">[Application Insights][application-insights] es un servicio propio de Application Performance Management (APM) extensible para desarrolladores web en varias plataformas.</span><span class="sxs-lookup"><span data-stu-id="377d7-128">[Application Insights][application-insights] is a first-party, extensible Application Performance Management (APM) service for web developers on multiple platforms.</span></span>

### <a name="alternatives"></a><span data-ttu-id="377d7-129">Alternativas</span><span class="sxs-lookup"><span data-stu-id="377d7-129">Alternatives</span></span>

<span data-ttu-id="377d7-130">Aunque este artículo se centra en Azure DevOps, se puede usar [Azure DevOps Server][azure-devops-server] (anteriormente conocido como Team Foundation Server) como sustituto en un entorno local.</span><span class="sxs-lookup"><span data-stu-id="377d7-130">While this article focuses on Azure DevOps, [Azure DevOps Server][azure-devops-server] (previously known as Team Foundation Server) could be used as an on-premises substitute.</span></span> <span data-ttu-id="377d7-131">Como alternativa, también puede usar un conjunto de tecnologías para una canalización de desarrollo de código abierto con [Jenkins][jenkins-on-azure].</span><span class="sxs-lookup"><span data-stu-id="377d7-131">Alternatively, you could also use a set of technologies for an open-source development pipeline using [Jenkins][jenkins-on-azure].</span></span>

<span data-ttu-id="377d7-132">Desde una perspectiva de infraestructura como código, las [plantillas de Resource Manager][arm-templates] se usaron como parte del proyecto de Azure DevOps, pero también se podrían considerar otras tecnologías de administración como [Terraform][terraform] o [Chef][chef].</span><span class="sxs-lookup"><span data-stu-id="377d7-132">From an infrastructure-as-code perspective, [Resource Manager templates][arm-templates] were used as part of the Azure DevOps project, but you could consider other management technologies such as [Terraform][terraform] or [Chef][chef].</span></span> <span data-ttu-id="377d7-133">Si prefiere una implementación basada en una infraestructura como servicio (IaaS) y necesita administración de la configuración, podría usar [Azure Automation State Configuration][desired-state-configuration], [Ansible][ansible] o [Chef][chef].</span><span class="sxs-lookup"><span data-stu-id="377d7-133">If you prefer an infrastructure-as-a-service (IaaS)-based deployment and require configuration management, you could consider either [Azure Automation State Configuration][desired-state-configuration], [Ansible][ansible], or [Chef][chef].</span></span>

<span data-ttu-id="377d7-134">Puede considerar estas alternativas al hospedaje en Azure Web Apps:</span><span class="sxs-lookup"><span data-stu-id="377d7-134">You could consider these alternatives to hosting in Azure Web Apps:</span></span>

- <span data-ttu-id="377d7-135">[Azure Virtual Machines][compare-vm-hosting] trata las cargas de trabajo que requieren un alto grado de control, o dependen de componentes o servicios del sistema operativo que no son posibles con Web Apps (por ejemplo, la caché global de ensamblados de Windows o COM).</span><span class="sxs-lookup"><span data-stu-id="377d7-135">[Azure Virtual Machines][compare-vm-hosting] handles workloads that require a high degree of control, or depend on OS components and services that are not possible with Web Apps (for example, the Windows GAC, or COM).</span></span>

- <span data-ttu-id="377d7-136">[Service Fabric][service-fabric] es una buena opción si la arquitectura de cargas de trabajo está centrada en torno a componentes distribuidos que se benefician de su implementación y ejecución en un clúster con un alto grado de control.</span><span class="sxs-lookup"><span data-stu-id="377d7-136">[Service Fabric][service-fabric] is a good option if the workload architecture is focused around distributed components that benefit from being deployed and run across a cluster with a high degree of control.</span></span> <span data-ttu-id="377d7-137">Service Fabric también se puede utilizar para hospedar contenedores.</span><span class="sxs-lookup"><span data-stu-id="377d7-137">Service Fabric can also be used to host containers.</span></span>

- <span data-ttu-id="377d7-138">[Azure Functions][azure-functions] proporciona un enfoque efectivo sin servidor si la arquitectura de las cargas de trabajo está centrada en torno a componentes distribuidos muy específicos, que requieren dependencias mínimas, donde solo se necesita que los componentes individuales se ejecuten a petición (no de forma continua) y en los que la orquestación de componentes no es necesaria.</span><span class="sxs-lookup"><span data-stu-id="377d7-138">[Azure Functions][azure-functions] provides an effective serverless approach if the workload architecture is centered around fine grained distributed components, requiring minimal dependencies, where individual components are only required to run on demand (not continuously) and orchestration of components is not required.</span></span>

<span data-ttu-id="377d7-139">Este [árbol de decisión para servicios de proceso de Azure](/azure/architecture/guide/technology-choices/compute-decision-tree) puede ayudar a la hora de elegir el camino correcto para realizar una migración.</span><span class="sxs-lookup"><span data-stu-id="377d7-139">This [decision tree for Azure compute services](/azure/architecture/guide/technology-choices/compute-decision-tree) may help when choosing the right path to take for a migration.</span></span>

## <a name="management-and-security-considerations"></a><span data-ttu-id="377d7-140">Consideraciones sobre administración y seguridad</span><span class="sxs-lookup"><span data-stu-id="377d7-140">Management and Security Considerations</span></span>

- <span data-ttu-id="377d7-141">Considere la posibilidad de usar una de las [tareas de tokenización][vsts-tokenization] que están disponibles en Marketplace de VSTS.</span><span class="sxs-lookup"><span data-stu-id="377d7-141">Consider leveraging one of the [tokenization tasks][vsts-tokenization] available in the VSTS marketplace.</span></span>

- <span data-ttu-id="377d7-142">Con las tareas de [Azure Key Vault][download-keyvault-secrets] se pueden descargar secretos desde una instancia de Azure Key Vault en su versión.</span><span class="sxs-lookup"><span data-stu-id="377d7-142">[Azure Key Vault][download-keyvault-secrets] tasks can download secrets from an Azure Key Vault into your release.</span></span> <span data-ttu-id="377d7-143">Luego puede usar esos secretos como variables que formen parte de la definición de versión, lo que evita almacenarlos en el control de código fuente.</span><span class="sxs-lookup"><span data-stu-id="377d7-143">You can then use those secrets as variables in your release definition, which avoids storing them in source control.</span></span>

- <span data-ttu-id="377d7-144">Utilice [variables de versión][vsts-release-variables] en las definiciones de versión para impulsar los cambios de configuración en los entornos.</span><span class="sxs-lookup"><span data-stu-id="377d7-144">Use [release variables][vsts-release-variables] in your release definitions to drive configuration changes of your environments.</span></span> <span data-ttu-id="377d7-145">Las variables de versión se pueden limitar a una versión completa o a un entorno determinado.</span><span class="sxs-lookup"><span data-stu-id="377d7-145">Release variables can be scoped to an entire release or a given environment.</span></span> <span data-ttu-id="377d7-146">Cuando use variables de información secreta, asegúrese de seleccionar el icono de candado.</span><span class="sxs-lookup"><span data-stu-id="377d7-146">When using variables for secret information, ensure that you select the padlock icon.</span></span>

- <span data-ttu-id="377d7-147">Las [puertas de implementación][vsts-deployment-gates] se deben utilizar en la canalización de versión.</span><span class="sxs-lookup"><span data-stu-id="377d7-147">[Deployment gates][vsts-deployment-gates] should be used in your release pipeline.</span></span> <span data-ttu-id="377d7-148">Esto le permite aprovechar los datos de supervisión en asociación con sistemas externos (por ejemplo, de administración de incidentes o con sistemas personalizados adicionales) para determinar si se debe promover una versión.</span><span class="sxs-lookup"><span data-stu-id="377d7-148">This lets you leverage monitoring data in association with external systems (for example, incident management or additional bespoke systems) to determine whether a release should be promoted.</span></span>

- <span data-ttu-id="377d7-149">Cuando se requiere intervención manual en una canalización de versión, utilice la funcionalidad de [aprobaciones][vsts-approvals].</span><span class="sxs-lookup"><span data-stu-id="377d7-149">Where manual intervention in a release pipeline is required, use the [approvals][vsts-approvals] functionality.</span></span>

- <span data-ttu-id="377d7-150">Considere el uso de [Application Insights][application-insights] y de herramientas de supervisión adicionales lo antes posible en la versión de canalización.</span><span class="sxs-lookup"><span data-stu-id="377d7-150">Consider using [Application Insights][application-insights] and additional monitoring tools as early as possible in your release pipeline.</span></span> <span data-ttu-id="377d7-151">Muchas organizaciones solo empiezan la supervisión en su entorno de producción.</span><span class="sxs-lookup"><span data-stu-id="377d7-151">Many organizations only begin monitoring in their production environment.</span></span> <span data-ttu-id="377d7-152">Al supervisar los demás entornos, puede identificar errores en un momento más temprano del proceso de desarrollo y evitar que los problemas lleguen al entorno de producción.</span><span class="sxs-lookup"><span data-stu-id="377d7-152">By monitoring your other environments, you can identify bugs earlier in the development process and avoid issues in your production environment.</span></span>

## <a name="deploy-the-scenario"></a><span data-ttu-id="377d7-153">Implementación del escenario</span><span class="sxs-lookup"><span data-stu-id="377d7-153">Deploy the scenario</span></span>

### <a name="prerequisites"></a><span data-ttu-id="377d7-154">Requisitos previos</span><span class="sxs-lookup"><span data-stu-id="377d7-154">Prerequisites</span></span>

- <span data-ttu-id="377d7-155">Debe tener una cuenta de Azure.</span><span class="sxs-lookup"><span data-stu-id="377d7-155">You must have an existing Azure account.</span></span> <span data-ttu-id="377d7-156">Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.</span><span class="sxs-lookup"><span data-stu-id="377d7-156">If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.</span></span>

- <span data-ttu-id="377d7-157">Tiene que registrarse como organización de Azure DevOps.</span><span class="sxs-lookup"><span data-stu-id="377d7-157">You must sign up for an Azure DevOps organization.</span></span> <span data-ttu-id="377d7-158">Para más información, consulte [Guía de inicio rápido: Creación de la organización][vsts-account-create].</span><span class="sxs-lookup"><span data-stu-id="377d7-158">For more information, see [Quickstart: Create your organization][vsts-account-create].</span></span>

### <a name="walk-through"></a><span data-ttu-id="377d7-159">Ejemplo paso a paso</span><span class="sxs-lookup"><span data-stu-id="377d7-159">Walk-through</span></span>

<span data-ttu-id="377d7-160">El [proyecto de Azure DevOps](/azure/devops-project/azure-devops-project-github) implementará un plan de App Service, una instancia de App Service y un recurso de App Insights, y configurará el proyecto de Azure DevOps.</span><span class="sxs-lookup"><span data-stu-id="377d7-160">The [Azure DevOps project](/azure/devops-project/azure-devops-project-github) will deploy an App Service Plan, App Service, and an App Insights resource for you, as well as configure the Azure DevOps project for you.</span></span>

<span data-ttu-id="377d7-161">Una vez que tenga implementado el proyecto de Azure DevOps y se haya completado la compilación, revise los cambios de código asociados, los elementos de trabajo y los resultados de las pruebas.</span><span class="sxs-lookup"><span data-stu-id="377d7-161">Once you've deployed the Azure DevOps project and the build is completed, review the associated code changes, work items, and test results.</span></span> <span data-ttu-id="377d7-162">Como puede observar, no se muestra ningún resultado de la prueba, ya que el código no contiene pruebas para ejecutar.</span><span class="sxs-lookup"><span data-stu-id="377d7-162">You will notice that no test results are displayed, because the code does not contain any tests to run.</span></span>

<span data-ttu-id="377d7-163">El proyecto crea una canalización de versión y un desencadenador de implementación continua, con lo que se implementa nuestra aplicación en el entorno de desarrollo.</span><span class="sxs-lookup"><span data-stu-id="377d7-163">The project creates a release pipeline and continuous deployment trigger, deploying our application into the Dev environment.</span></span> <span data-ttu-id="377d7-164">Como parte de un proceso de implementación continua, puede que vea versiones que se extienden en varios entornos.</span><span class="sxs-lookup"><span data-stu-id="377d7-164">As part of a continuous deployment process, you may see releases that span multiple environments.</span></span> <span data-ttu-id="377d7-165">Una versión puede abarcar la infraestructura (mediante técnicas como la de infraestructura como código) y también implementar los paquetes de aplicación necesarios así como todas las tareas posteriores a la configuración.</span><span class="sxs-lookup"><span data-stu-id="377d7-165">A release can span both infrastructure (using techniques such as infrastructure-as-code), and can also deploy the application packages required along with any post-configuration tasks.</span></span>

## <a name="pricing"></a><span data-ttu-id="377d7-166">Precios</span><span class="sxs-lookup"><span data-stu-id="377d7-166">Pricing</span></span>

<span data-ttu-id="377d7-167">El costo de Azure DevOps dependerá del número de usuarios de su organización que requieren acceso, además de otros factores como el número de versiones o compilaciones simultáneas necesarias y el número de usuarios de prueba.</span><span class="sxs-lookup"><span data-stu-id="377d7-167">Azure DevOps costs depend on the number of users in your organization that require access, along with other factors like the number of concurrent build/releases required and number of test users.</span></span> <span data-ttu-id="377d7-168">Para más información, consulte [Precios de Azure DevOps][vsts-pricing-page].</span><span class="sxs-lookup"><span data-stu-id="377d7-168">For more information, see [Azure DevOps pricing][vsts-pricing-page].</span></span>

<span data-ttu-id="377d7-169">Esta [calculadora de precios][vsts-pricing-calculator] proporciona una estimación para ejecutar Azure DevOps con 20 usuarios.</span><span class="sxs-lookup"><span data-stu-id="377d7-169">This [pricing calculator][vsts-pricing-calculator] provides an estimate for running Azure DevOps with 20 users.</span></span>

<span data-ttu-id="377d7-170">Azure DevOps se factura por usuario y mes.</span><span class="sxs-lookup"><span data-stu-id="377d7-170">Azure DevOps is billed on a per-user per-month basis.</span></span> <span data-ttu-id="377d7-171">Puede haber cargos adicionales en función de las canalizaciones simultáneas que se necesiten, además de usuarios de prueba o licencias de usuario básico adicionales.</span><span class="sxs-lookup"><span data-stu-id="377d7-171">There may be additional charges dependent upon concurrent pipelines needed, in addition to any additional test users or user basic licenses.</span></span>

## <a name="related-resources"></a><span data-ttu-id="377d7-172">Recursos relacionados</span><span class="sxs-lookup"><span data-stu-id="377d7-172">Related resources</span></span>

<span data-ttu-id="377d7-173">Consulte los siguientes recursos para aprender más sobre CI/CD y Azure DevOps:</span><span class="sxs-lookup"><span data-stu-id="377d7-173">Review the following resources to learn more about CI/CD and Azure DevOps:</span></span>

- <span data-ttu-id="377d7-174">[¿Qué es DevOps?][devops-whatis]</span><span class="sxs-lookup"><span data-stu-id="377d7-174">[What is DevOps?][devops-whatis]</span></span>
- <span data-ttu-id="377d7-175">[DevOps at Microsoft - How we work with Azure DevOps][devops-microsoft] (DevOps en Microsoft: Cómo se trabaja con Azure DevOps)</span><span class="sxs-lookup"><span data-stu-id="377d7-175">[DevOps at Microsoft - How we work with Azure DevOps][devops-microsoft]</span></span>
- <span data-ttu-id="377d7-176">[Tutoriales detallados: DevOps con Azure DevOps][devops-with-vsts]</span><span class="sxs-lookup"><span data-stu-id="377d7-176">[Step-by-step Tutorials: DevOps with Azure DevOps][devops-with-vsts]</span></span>
- <span data-ttu-id="377d7-177">[Lista de comprobación de DevOps][devops-checklist]</span><span class="sxs-lookup"><span data-stu-id="377d7-177">[Devops Checklist][devops-checklist]</span></span>
- <span data-ttu-id="377d7-178">[Creación de una canalización de CI/CD para .NET con un proyecto de Azure DevOps][devops-project-create]</span><span class="sxs-lookup"><span data-stu-id="377d7-178">[Create a CI/CD pipeline for .NET with the Azure DevOps project][devops-project-create]</span></span>

<!-- links -->

[ansible]: /azure/ansible/
[application-insights]: /azure/application-insights/app-insights-overview
[app-service-reference-architecture]: ../../reference-architectures/app-service-web-app/basic-web-app.md
[arm-templates]: /azure/azure-resource-manager/resource-group-overview#template-deployment
[architecture]: ./media/architecture-devops-dotnet-webapp.svg
[chef]: /azure/chef/
[design-patterns-availability]: /azure/architecture/patterns/category/availability
[design-patterns-resiliency]: /azure/architecture/patterns/category/resiliency
[design-patterns-scalability]: /azure/architecture/patterns/category/performance-scalability
[design-patterns-security]: /azure/architecture/patterns/category/security
[desired-state-configuration]: /azure/automation/automation-dsc-overview
[devops-microsoft]: /azure/devops/devops-at-microsoft/
[devops-with-vsts]: https://almvm.azurewebsites.net/labs/vsts/
[devops-checklist]: /azure/architecture/checklist/dev-ops
[application-insights]: https://azure.microsoft.com/services/application-insights/
[cloud-based-load-testing]: https://visualstudio.microsoft.com/team-services/cloud-load-testing/
[cloud-based-load-testing-on-premises]: /vsts/test/load-test/clt-with-private-machines?view=vsts
[jenkins-on-azure]: /azure/jenkins/
[devops-whatis]: /azure/devops/what-is-devops
[download-keyvault-secrets]: /vsts/pipelines/tasks/deploy/azure-key-vault?view=vsts
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[resiliency-app-service]: /azure/architecture/checklist/resiliency-per-service#app-service
[vsts]: /vsts/?view=vsts#pivot=services
[continuous-integration]: /azure/devops/what-is-continuous-integration
[continuous-delivery]: /azure/devops/what-is-continuous-delivery
[web-apps]: /azure/app-service/app-service-web-overview
[vsts-account-create]: /azure/devops/organizations/accounts/create-organization-msa-or-work-student?view=vsts
[vsts-approvals]: /vsts/pipelines/release/approvals/approvals?view=vsts
[devops-project]: https://portal.azure.com/?feature.customportal=false#create/Microsoft.AzureProject
[vsts-deployment-gates]: /vsts/pipelines/release/approvals/gates?view=vsts
[vsts-pricing-calculator]: https://azure.com/e/498aa024454445a8a352e75724f900b1
[vsts-pricing-page]: https://azure.microsoft.com/pricing/details/visual-studio-team-services/
[vsts-release-variables]: /vsts/pipelines/release/variables?view=vsts&tabs=batch
[vsts-tokenization]: https://marketplace.visualstudio.com/search?term=token&target=VSTS&category=All%20categories&sortBy=Relevance
[azure-key-vault]: /azure/key-vault/key-vault-overview
[infra-as-code]: https://blogs.msdn.microsoft.com/mvpawardprogram/2018/02/13/infrastructure-as-code/
[azure-devops-server]: https://visualstudio.microsoft.com/tfs/
[infra-as-code]: https://blogs.msdn.microsoft.com/mvpawardprogram/2018/02/13/infrastructure-as-code/
[service-fabric]: /azure/service-fabric/
[azure-functions]: /azure/azure-functions/
[azure-containers]: https://azure.microsoft.com/overview/containers/
[compare-vm-hosting]: /azure/app-service/choose-web-site-cloud-service-vm
[app-insights-cd-monitoring]: /azure/application-insights/app-insights-vsts-continuous-monitoring
[azure-region-pair-bcdr]: /azure/best-practices-availability-paired-regions
[devops-project-create]: /azure/devops-project/azure-devops-project-aspnet-core
[terraform]: /azure/terraform/
