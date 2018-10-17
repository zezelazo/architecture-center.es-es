---
title: Canalización de CI/CD con Azure DevOps
description: Compilación y publicación de una aplicación .NET en Azure Web Apps con Azure DevOps.
author: christianreddington
ms.date: 07/11/18
ms.openlocfilehash: 80890784d4c97aac418cef4e49f9075dbef10b8a
ms.sourcegitcommit: b2a4eb132857afa70201e28d662f18458865a48e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/05/2018
ms.locfileid: "48818945"
---
# <a name="cicd-pipeline-with-azure-devops"></a>Canalización de CI/CD con Azure DevOps

DevOps es la integración de las operaciones de desarrollo, control de calidad y TI. DevOps requiere una referencia cultural unificada y un sólido conjunto de procesos para la entrega de software.

Este escenario de ejemplo muestra cómo los equipos de desarrollo pueden usar Azure DevOps para implementar una aplicación web .NET de dos niveles en Azure App Service. La aplicación web depende de servicios descendentes de plataforma como servicio (PaaS) de Azure. Este documento también señala algunas consideraciones que debe tener en cuenta al diseñar un escenario como este mediante PaaS de Azure.

Adoptar un enfoque moderno en el desarrollo de aplicaciones mediante el uso de la integración continua y la implementación continua (CI/CD) ayuda a proporcionar valor más rápidamente a los usuarios mediante un servicio sólido de compilación, prueba, implementación y supervisión. Con el uso de una plataforma como Azure DevOps junto con servicios de Azure como App Service, las organizaciones pueden centrarse en el desarrollo de su escenario en lugar de en la administración de la infraestructura de soporte.

## <a name="relevant-use-cases"></a>Casos de uso pertinentes

Considere la posibilidad de usar DevOps para los casos de uso siguientes:

* Aceleración del desarrollo de aplicaciones y ciclos de vida de desarrollo
* Generación de calidad y coherencia en un proceso automatizado de compilación y lanzamiento

## <a name="architecture"></a>Arquitectura

![Introducción a la arquitectura de los componentes de Azure implicados en un escenario de DevOps con Azure DevOps y Azure App Service][architecture]

Este escenario incluye una canalización de CI/CD para una aplicación web de .NET con Azure DevOps. Los datos fluyen por el escenario de la siguiente manera:

1. Cambie el código fuente de la aplicación.
2. Confirme el código de la aplicación y el archivo web.config de Web Apps.
3. La integración continua desencadena las pruebas unitarias y la compilación de la aplicación.
4. El desencadenador de implementación continua organiza la implementación de los artefactos de la aplicación *con valores parametrizados de configuración específicos del entorno*.
5. Implementación en Azure App Service.
6. Azure Application Insights recopila y analiza datos de mantenimiento, rendimiento y uso.
7. Revise la información de mantenimiento, rendimiento y uso.

### <a name="components"></a>Componentes

* [Azure DevOps][vsts] es un servicio para administrar el ciclo de vida del desarrollo de un extremo a otro, desde el planeamiento y administración del proyecto a la administración del código, continuando con la compilación y el lanzamiento.
* [Azure Web Apps][web-apps] es un servicio PaaS para hospedar aplicaciones web, API REST y back-ends para dispositivos móviles. Aunque este artículo se centra en. NET, hay varias opciones de plataforma de desarrollo adicionales compatibles.
* [Application Insights][application-insights] es un servicio propio de Application Performance Management (APM) extensible para desarrolladores web en varias plataformas.

### <a name="alternative-devops-tooling-options"></a>Opciones alternativas de herramientas de DevOps

Aunque este artículo se centra en Azure DevOps, se puede usar [Team Foundation Server][team-foundation-server] como sustituto en un entorno local. Como alternativa, también puede usar un conjunto de tecnologías para una canalización de desarrollo de código abierto con [Jenkins][jenkins-on-azure].

Desde una perspectiva de infraestructura como código, las [plantillas de Azure Resource Manager][arm-templates] se incluyen como parte del proyecto de Azure DevOps, pero también podría usar [Terraform][terraform] o [Chef][chef]. Si prefiere una implementación basada en una infraestructura como servicio (IaaS) y necesita administración de la configuración, podría usar [Azure Automation State Configuration][desired-state-configuration], [Ansible][ansible] o [Chef][chef].

### <a name="alternatives-to-azure-web-apps"></a>Alternativas a Azure Web Apps

Puede considerar estas alternativas al hospedaje en Azure Web Apps:

* [Azure Virtual Machines][compare-vm-hosting] &mdash; para cargas de trabajo que requieren un alto grado de control, o dependen de componentes o servicios del sistema operativo que no son posibles con Web Apps (por ejemplo, la caché global de ensamblados de Windows o COM).
* [Service Fabric][service-fabric] &mdash; una buena opción si la arquitectura de cargas de trabajo está centrada en torno a componentes distribuidos que se benefician de su implementación y ejecución en un clúster con un alto grado de control. Service Fabric también se puede utilizar para hospedar contenedores.
* [Azure Functions][azure-functions]: un enfoque efectivo sin servidor si la arquitectura de las cargas de trabajo está centrada en torno a componentes distribuidos muy específicos, que requieren dependencias mínimas, donde solo se necesita que los componentes individuales se ejecuten a petición (no de forma continua) y en los que la orquestación de componentes no es necesaria.

Este [árbol de decisión](/azure/architecture/guide/technology-choices/compute-decision-tree) puede ayudar a la hora de elegir el camino correcto para realizar una migración.

### <a name="devops"></a>DevOps

**[Integración continua (CI)][continuous-integration]** mantiene una versión estable, con varios desarrolladores que realizan con regularidad cambios pequeños y frecuentes en el código base compartido. Como parte de la canalización de integración continua debe:
* Confirmar con frecuencia pequeños cambios en el código. Evitar el procesamiento por lotes de cambios más grandes o complejos que pueden ser más difíciles de fusionar correctamente mediante combinación.
* Realizar pruebas unitarias de los componentes de la aplicación con suficiente cobertura de código (incluyendo la prueba de las rutas incorrectas).
* Garantizar que la compilación se ejecuta en la rama (o tronco) compartida maestra. Esta rama debe ser estable y permanecer "lista para implementación". Se deben aislar los cambios incompletos o en curso en una rama independiente con combinaciones frecuentes "de integración hacia delante" para evitar conflictos posteriores.

La **[entrega continua (CD)][continuous-delivery]** debe tener como objetivo mostrar no solo una compilación estable, sino también una implementación estable. Esto dificulta la comprensión de la implementación continua y requiere una configuración específica para el entorno y un mecanismo para configurar esos valores correctamente. Otras consideraciones de la implementación continua incluyen las siguientes:
* Se necesita una cobertura suficiente de las pruebas de integración para asegurarse de que los distintos componentes están configurados y funcionando correctamente de un extremo a otro.
* Puede que también necesite configurar y restablecer datos específicos del entorno y administrar versiones del esquema de la base de datos.
* La entrega continua también se puede ampliar a los entornos de pruebas de carga y pruebas de aceptación del usuario.
* La entrega continua se beneficia de la supervisión continua, idealmente en todos los entornos.
* La coherencia y confiabilidad de las implementaciones y pruebas de integración en los entornos se simplifica mediante la creación de scripts de creación y configuración de la infraestructura de hospedaje. Esto es considerablemente más fácil para cargas de trabajo basadas en la nube. Para más información, consulte [Infrastructure as Code][infra-as-code] (Infraestructura como código).
* Inicie la entrega continua tan pronto como sea posible en el ciclo de vida del proyecto. Cuanto más tarde se realice, más difícil será de incorporar.
* Se le debe dar la misma prioridad a las pruebas unitarias y de integración que a las características de la aplicación.
* Use paquetes de implementación independientes del entorno y administración de configuración específica del entorno a lo largo del proceso de lanzamiento.
* Proteja la configuración confidencial utilizando las herramientas de administración de versiones o mediante una llamada a un módulo de seguridad de hardware (HSM) o instancia de [Azure Key Vault][azure-key-vault] durante el proceso de lanzamiento. No almacene configuración confidencial dentro del control de código fuente.

**Aprendizaje continuo**. La supervisión más eficaz de un entorno de implementación continua la proporcionan las herramientas de supervisión de rendimiento de aplicaciones (APM) como, por ejemplo, [Application Insights][application-insights]. Un nivel suficiente de supervisión de la carga de trabajo de una aplicación resulta vital para entender los errores y el rendimiento con carga. Application Insights se puede integrar en VSTS para habilitar la [supervisión continua de la canalización de implementación continua][app-insights-cd-monitoring]. Esta se puede usar para habilitar una progresión continua a la siguiente fase, sin intervención del usuario, o una reversión si se detecta una alerta.

## <a name="considerations"></a>Consideraciones

### <a name="availability"></a>Disponibilidad

Considere la posibilidad de aprovechar los [patrones de diseño típicos de disponibilidad][design-patterns-availability] al compilar la aplicación en la nube.

Revise las consideraciones sobre disponibilidad en la correspondiente [arquitectura de referencia de aplicación web de App Service][app-service-reference-architecture]

Para ver otros temas de disponibilidad, consulte la [lista de comprobación de disponibilidad][availability] que encontrará en Azure Architecture Center.

### <a name="scalability"></a>Escalabilidad

Al compilar una aplicación en la nube debe tener en cuenta los [patrones de diseño típicos de escalabilidad][design-patterns-scalability].

Revise las consideraciones sobre escalabilidad en la correspondiente [arquitectura de referencia de aplicación web de App Service][app-service-reference-architecture]

Para ver otros temas de escalabilidad, consulte la [lista de comprobación de escalabilidad][scalability] que encontrará en el centro de arquitectura de Azure.

### <a name="security"></a>Seguridad

Considere la posibilidad de aprovechar los [patrones de diseño típicos de seguridad][design-patterns-security] donde corresponda.

Revise las consideraciones sobre seguridad en la correspondiente [arquitectura de referencia de aplicación web de App Service][app-service-reference-architecture].

Para obtener instrucciones generales sobre el diseño de soluciones seguras, consulte la [documentación de seguridad de Azure][security].

### <a name="resiliency"></a>Resistencia

Revise los [patrones de diseño típicos de resistencia][design-patterns-resiliency] y considere la posibilidad de implementar estos cuando corresponda.

Encontrará varios [procedimientos recomendados para App Service][resiliency-app-service] en el Centro de arquitectura de Azure.

Para obtener instrucciones generales sobre el diseño de soluciones resistentes, consulte [Diseño de aplicaciones resistentes de Azure][resiliency].

## <a name="deploy-the-scenario"></a>Implementación del escenario

### <a name="prerequisites"></a>Requisitos previos

* Debe tener una cuenta de Azure. Si no tiene ninguna suscripción a Azure, cree una [cuenta gratuita][azure-free-account] antes de empezar.
* Tiene que registrarse como organización de Azure DevOps. Para más información, consulte [Guía de inicio rápido: Creación de una organización][vsts-account-create].

### <a name="walk-through"></a>Ejemplo paso a paso

En este escenario va a usar el proyecto de Azure DevOps para crear la canalización de CI/CD.

El proyecto de DevOps implementará App Service, un plan de App Service y un recurso de App Insights, y configurará el proyecto de Azure DevOps.

Una vez que tenga implementado el proyecto de Azure DevOps y se haya completado la compilación, revise los cambios de código asociados, los elementos de trabajo y los resultados de las pruebas. Como puede observar, no se muestra ningún resultado de la prueba, ya que el código no contiene pruebas para ejecutar.

Revise las definiciones de versión. Tenga en cuenta que se ha configurado una canalización de versión que ha permitido publicar nuestra aplicación en el entorno de desarrollo. Observe que hay un **desencadenador de implementación continua** establecido en el artefacto de compilación **Drop**, con versiones automáticas en el entorno de desarrollo. Como parte de un proceso de implementación continua, puede que vea versiones que se extienden en varios entornos. Una versión puede abarcar la infraestructura (mediante técnicas como la de infraestructura como código) y también implementar los paquetes de aplicación necesarios así como todas las tareas posteriores a la configuración.

## <a name="additional-considerations"></a>Consideraciones adicionales

* Considere la posibilidad de usar una de las [tareas de tokenización][vsts-tokenization] que están disponibles en Marketplace de VSTS.
* Considere la posibilidad de usar la tarea [Deploy: Azure Key Vault][download-keyvault-secrets] de VSTS para descargar los secretos desde Azure Key Vault en su versión. Luego puede usar esos secretos como variables que formen parte de la definición de versión y así evitar almacenarlos en el control de código fuente.
* Considere la posibilidad de usar [variables de versión][vsts-release-variables] en las definiciones de versión para impulsar los cambios de configuración en los entornos. Las variables de versión se pueden limitar a una versión completa o a un entorno determinado. Cuando use variables de información secreta, asegúrese de seleccionar el icono de candado.
* Piense en la posibilidad de usar [puertas de implementación][vsts-deployment-gates] en la canalización de versión. Esto le permite aprovechar los datos de supervisión en asociación con sistemas externos (por ejemplo, de administración de incidentes o con sistemas personalizados adicionales) para determinar si se debe promover una versión.
* Cuando se requiere intervención manual en una canalización de versión, piense sobre la conveniencia de usar la funcionalidad de [aprobaciones][vsts-approvals].
* Considere el uso de [Application Insights][application-insights] y de herramientas de supervisión adicionales lo antes posible en la versión de canalización. Muchas organizaciones solo empiezan la supervisión en su entorno de producción; mediante la supervisión de los demás entornos, puede identificar errores en un momento más temprano del proceso de desarrollo y evitar que los problemas lleguen al entorno de producción.

## <a name="pricing"></a>Precios

El costo de Azure DevOps dependerá del número de usuarios de su organización que requieren acceso, además de otros factores como el número de versiones o compilaciones simultáneas necesarias y el número de usuarios de prueba. Para más información, consulte [Precios de Azure DevOps][vsts-pricing-page].

* [Azure DevOps][vsts-pricing-calculator] es un servicio que le permite administrar el ciclo de vida del desarrollo. La facturación es por usuario por mes. Puede haber cargos adicionales en función de las canalizaciones simultáneas que se necesiten, además de usuarios de prueba o licencias de usuario básico adicionales.

## <a name="related-resources"></a>Recursos relacionados

* [¿Qué es DevOps?][devops-whatis]
* [DevOps at Microsoft - How we work with Azure DevOps][devops-microsoft] (DevOps en Microsoft: Cómo se trabaja con Azure DevOps)
* [Tutoriales paso a paso: DevOps con Azure DevOps][devops-with-vsts]
* [Lista de comprobación de DevOps][devops-checklist]
* [Creación de una canalización de CI/CD para .NET con un proyecto de Azure DevOps][devops-project-create]

<!-- links -->
[ansible]: /azure/ansible/
[application-insights]: /azure/application-insights/app-insights-overview
[app-service-reference-architecture]: ../../reference-architectures/app-service-web-app/basic-web-app.md
[azure-free-account]: https://azure.microsoft.com/free/?WT.mc_id=A261C142F
[arm-templates]: /azure/azure-resource-manager/resource-group-overview#template-deployment
[architecture]: ./media/architecture-devops-dotnet-webapp.png
[availability]: /azure/architecture/checklist/availability
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
[resiliency]: /azure/architecture/checklist/resiliency
[scalability]: /azure/architecture/checklist/scalability
[vsts]: /vsts/?view=vsts#pivot=services
[continuous-integration]: /azure/devops/what-is-continuous-integration
[continuous-delivery]: /azure/devops/what-is-continuous-delivery
[web-apps]: /azure/app-service/app-service-web-overview
[terraform]: /azure/terraform/
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
[team-foundation-server]: https://visualstudio.microsoft.com/tfs/
[infra-as-code]: https://blogs.msdn.microsoft.com/mvpawardprogram/2018/02/13/infrastructure-as-code/
[service-fabric]: /azure/service-fabric/
[azure-functions]: /azure/azure-functions/
[azure-containers]: https://azure.microsoft.com/overview/containers/
[compare-vm-hosting]: /azure/app-service/choose-web-site-cloud-service-vm
[app-insights-cd-monitoring]: /azure/application-insights/app-insights-vsts-continuous-monitoring
[azure-region-pair-bcdr]: /azure/best-practices-availability-paired-regions
[devops-project-create]: /azure/devops-project/azure-devops-project-aspnet-core
[security]: /azure/security/