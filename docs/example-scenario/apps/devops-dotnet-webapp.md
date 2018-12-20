---
title: Diseño de una canalización de CI/CD con Azure DevOps
description: Compilación y publicación de una aplicación .NET en Azure Web Apps con Azure DevOps.
author: christianreddington
ms.date: 12/06/2018
ms.custom:
- fasttrack
- seodec18
ms.openlocfilehash: 23945493115522d099b6b26922f567653da0367e
ms.sourcegitcommit: 4ba3304eebaa8c493c3e5307bdd9d723cd90b655
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/12/2018
ms.locfileid: "53307289"
---
# <a name="design-a-cicd-pipeline-using-azure-devops"></a>Diseño de una canalización de CI/CD con Azure DevOps

Este escenario proporciona una guía de arquitectura y diseño para la creación de una canalización de integración continua (CI) y de implementación continua (CD).  En este ejemplo, la canalización de CI/CD implementa una aplicación web de .NET de dos niveles en Azure App Service.

La migración a procesos modernos de CI/CD ofrece muchos beneficios para la creación, implementación, prueba y supervisión de aplicaciones. Al utilizar Azure DevOps junto con otros servicios como App Service, las organizaciones pueden centrarse en el desarrollo de sus aplicaciones en lugar de en la administración de la infraestructura de soporte.

## <a name="relevant-use-cases"></a>Casos de uso pertinentes

Considere DevOps Azure y los procesos de CI/CD para:

- Aceleración del desarrollo de aplicaciones y ciclos de vida de desarrollo
- Generación de calidad y coherencia en un proceso automatizado de compilación y lanzamiento
- Aumento de la estabilidad y el tiempo de actividad de la aplicación

## <a name="architecture"></a>Arquitectura

![Diagrama de la arquitectura de los componentes de Azure implicados en un escenario de DevOps con Azure DevOps y Azure App Service][architecture]

Los datos fluyen por el escenario de la siguiente manera:

1. Un desarrollador cambia el código fuente de la aplicación.
2. El código de la aplicación, incluido el archivo web.config, se envía al repositorio de código fuente de Azure Repos.
3. La integración continua desencadena las pruebas unitarias y la compilación de la aplicación mediante Azure Test Plans.
4. La implementación continua en Azure Pipelines desencadena la implementación automatizada de los artefactos de la aplicación *con valores de configuración específicos del entorno*.
5. Los artefactos se implementan en Azure App Service.
6. Azure Application Insights recopila y analiza datos de mantenimiento, rendimiento y uso.
7. Los desarrolladores supervisan y administran la información sobre el estado, el rendimiento y el uso.
8. La información de trabajos pendientes se utiliza para priorizar nuevas características y correcciones de errores mediante Azure Boards.

### <a name="components"></a>Componentes

- [Azure DevOps][vsts] es un servicio para administrar el ciclo de vida del desarrollo de un extremo a otro, desde el planeamiento y administración del proyecto a la administración del código, continuando con la compilación y el lanzamiento.

- [Azure Web Apps][web-apps] es un servicio PaaS para hospedar aplicaciones web, API REST y back-ends para dispositivos móviles. Aunque este artículo se centra en. NET, hay varias opciones de plataforma de desarrollo adicionales compatibles.

- [Application Insights][application-insights] es un servicio propio de Application Performance Management (APM) extensible para desarrolladores web en varias plataformas.

### <a name="alternatives"></a>Alternativas

Aunque este artículo se centra en Azure DevOps, se puede usar [Azure DevOps Server][azure-devops-server] (anteriormente conocido como Team Foundation Server) como sustituto en un entorno local. Como alternativa, también puede usar un conjunto de tecnologías para una canalización de desarrollo de código abierto con [Jenkins][jenkins-on-azure].

Desde una perspectiva de infraestructura como código, las [plantillas de Resource Manager][arm-templates] se usaron como parte del proyecto de Azure DevOps, pero también se podrían considerar otras tecnologías de administración como [Terraform][terraform] o [Chef][chef]. Si prefiere una implementación basada en una infraestructura como servicio (IaaS) y necesita administración de la configuración, podría usar [Azure Automation State Configuration][desired-state-configuration], [Ansible][ansible] o [Chef][chef].

Puede considerar estas alternativas al hospedaje en Azure Web Apps:

- [Azure Virtual Machines][compare-vm-hosting] trata las cargas de trabajo que requieren un alto grado de control, o dependen de componentes o servicios del sistema operativo que no son posibles con Web Apps (por ejemplo, la caché global de ensamblados de Windows o COM).

- [Service Fabric][service-fabric] es una buena opción si la arquitectura de cargas de trabajo está centrada en torno a componentes distribuidos que se benefician de su implementación y ejecución en un clúster con un alto grado de control. Service Fabric también se puede utilizar para hospedar contenedores.

- [Azure Functions][azure-functions] proporciona un enfoque efectivo sin servidor si la arquitectura de las cargas de trabajo está centrada en torno a componentes distribuidos muy específicos, que requieren dependencias mínimas, donde solo se necesita que los componentes individuales se ejecuten a petición (no de forma continua) y en los que la orquestación de componentes no es necesaria.

Este [árbol de decisión para servicios de proceso de Azure](/azure/architecture/guide/technology-choices/compute-decision-tree) puede ayudar a la hora de elegir el camino correcto para realizar una migración.

## <a name="management-and-security-considerations"></a>Consideraciones sobre administración y seguridad

- Considere la posibilidad de usar una de las [tareas de tokenización][vsts-tokenization] que están disponibles en Marketplace de VSTS.

- Con las tareas de [Azure Key Vault][download-keyvault-secrets] se pueden descargar secretos desde una instancia de Azure Key Vault en su versión. Luego puede usar esos secretos como variables que formen parte de la definición de versión, lo que evita almacenarlos en el control de código fuente.

- Utilice [variables de versión][vsts-release-variables] en las definiciones de versión para impulsar los cambios de configuración en los entornos. Las variables de versión se pueden limitar a una versión completa o a un entorno determinado. Cuando use variables de información secreta, asegúrese de seleccionar el icono de candado.

- Las [puertas de implementación][vsts-deployment-gates] se deben utilizar en la canalización de versión. Esto le permite aprovechar los datos de supervisión en asociación con sistemas externos (por ejemplo, de administración de incidentes o con sistemas personalizados adicionales) para determinar si se debe promover una versión.

- Cuando se requiere intervención manual en una canalización de versión, utilice la funcionalidad de [aprobaciones][vsts-approvals].

- Considere el uso de [Application Insights][application-insights] y de herramientas de supervisión adicionales lo antes posible en la versión de canalización. Muchas organizaciones solo empiezan la supervisión en su entorno de producción. Al supervisar los demás entornos, puede identificar errores en un momento más temprano del proceso de desarrollo y evitar que los problemas lleguen al entorno de producción.

## <a name="deploy-the-scenario"></a>Implementación del escenario

### <a name="prerequisites"></a>Requisitos previos

- Debe tener una cuenta de Azure. Si no tiene ninguna suscripción a Azure, cree una [cuenta gratuita][azure-free-account] antes de empezar.

- Tiene que registrarse como organización de Azure DevOps. Para más información, consulte [Guía de inicio rápido: Creación de la organización][vsts-account-create].

### <a name="walk-through"></a>Ejemplo paso a paso

El [proyecto de Azure DevOps](/azure/devops-project/azure-devops-project-github) implementará un plan de App Service, una instancia de App Service y un recurso de App Insights, y configurará el proyecto de Azure DevOps.

Una vez que tenga implementado el proyecto de Azure DevOps y se haya completado la compilación, revise los cambios de código asociados, los elementos de trabajo y los resultados de las pruebas. Como puede observar, no se muestra ningún resultado de la prueba, ya que el código no contiene pruebas para ejecutar.

El proyecto crea una canalización de versión y un desencadenador de implementación continua, con lo que se implementa nuestra aplicación en el entorno de desarrollo. Como parte de un proceso de implementación continua, puede que vea versiones que se extienden en varios entornos. Una versión puede abarcar la infraestructura (mediante técnicas como la de infraestructura como código) y también implementar los paquetes de aplicación necesarios así como todas las tareas posteriores a la configuración.

## <a name="pricing"></a>Precios

El costo de Azure DevOps dependerá del número de usuarios de su organización que requieren acceso, además de otros factores como el número de versiones o compilaciones simultáneas necesarias y el número de usuarios de prueba. Para más información, consulte [Precios de Azure DevOps][vsts-pricing-page].

Esta [calculadora de precios][vsts-pricing-calculator] proporciona una estimación para ejecutar Azure DevOps con 20 usuarios.

Azure DevOps se factura por usuario y mes. Puede haber cargos adicionales en función de las canalizaciones simultáneas que se necesiten, además de usuarios de prueba o licencias de usuario básico adicionales.

## <a name="related-resources"></a>Recursos relacionados

Consulte los siguientes recursos para aprender más sobre CI/CD y Azure DevOps:

- [¿Qué es DevOps?][devops-whatis]
- [DevOps at Microsoft - How we work with Azure DevOps][devops-microsoft] (DevOps en Microsoft: Cómo se trabaja con Azure DevOps)
- [Tutoriales detallados: DevOps con Azure DevOps][devops-with-vsts]
- [Lista de comprobación de DevOps][devops-checklist]
- [Creación de una canalización de CI/CD para .NET con un proyecto de Azure DevOps][devops-project-create]

<!-- links -->

[ansible]: /azure/ansible/
[application-insights]: /azure/application-insights/app-insights-overview
[app-service-reference-architecture]: ../../reference-architectures/app-service-web-app/basic-web-app.md
[azure-free-account]: https://azure.microsoft.com/free/?WT.mc_id=A261C142F
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
