---
title: Canalización de CI/CD con VSTS
description: Un ejemplo de compilación y publicación de una aplicación .NET en Azure Web Apps
author: christianreddington
ms.date: 07/11/18
ms.openlocfilehash: aea757087f4a505a8c52658abe1841c5455977cc
ms.sourcegitcommit: c49aeef818d7dfe271bc4128b230cfc676f05230
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/11/2018
ms.locfileid: "44389293"
---
# <a name="cicd-pipeline-with-vsts"></a>Canalización de CI/CD con VSTS

DevOps es la integración de las operaciones de desarrollo, control de calidad y TI. DevOps requiere una referencia cultural unificada y un sólido conjunto de procesos para la entrega de software.

Este escenario de ejemplo muestra cómo los equipos de desarrollo pueden usar Visual Studio Team Services para implementar una aplicación web .NET de dos niveles en Azure App Service. La aplicación web depende de servicios de plataforma como servicio (PaaS) de Azure de nivel inferior. Este documento también señala algunas consideraciones que debe tener en cuenta al diseñar un escenario como este mediante una plataforma como servicio (PaaS) de Azure.

Adoptar un enfoque moderno en el desarrollo de aplicaciones mediante la integración continua (CI) y la implementación continua (CD) ayuda a proporcionar valor más rápidamente a los usuarios mediante un servicio sólido de compilación, prueba, implementación y supervisión. Con el uso de una plataforma como Visual Studio Team Services además de servicios de Azure como App Service, las organizaciones se aseguran de que se centran en el desarrollo de su escenario en lugar de en la administración de la infraestructura que necesitan para habilitarlo.

## <a name="related-use-cases"></a>Casos de uso relacionados

Considere la posibilidad de usar DevOps para los casos de uso siguientes:

* Aceleración del desarrollo de aplicaciones y los ciclos de vida de desarrollo
* Generación de calidad y coherencia en un proceso automatizado de compilación y lanzamiento

## <a name="architecture"></a>Arquitectura

![Introducción a la arquitectura de los componentes de Azure implicados en un escenario de DevOps con Visual Studio Team Services y Azure App Service][architecture]

Este escenario incluye una canalización de DevOps para una aplicación web de .NET con Visual Studio Team Services (VSTS). Los datos fluyen por el escenario de la siguiente manera:

1. Cambie el código fuente de la aplicación.
2. Confirme el código de la aplicación y el archivo web.config de Web Apps.
3. La integración continua desencadena las pruebas unitarias y la compilación de la aplicación.
4. El desencadenador de implementación continua organiza la implementación de los artefactos de la aplicación *con valores parametrizados de configuración específicos del entorno*.
5. Implementación en Azure App Service.
6. Azure Application Insights recopila y analiza datos de mantenimiento, rendimiento y uso.
7. Revise la información de mantenimiento, rendimiento y uso.

### <a name="components"></a>Componentes

* Los [grupos de recursos][resource-groups] son contenedores lógicos de recursos de Azure y también proporcionan un límite de control de acceso para el plano de administración. Piense en un grupo de recursos como en la representación de una "unidad de implementación".
* [Visual Studio Team Services (VSTS)][vsts] es un servicio que le permite administrar el ciclo de vida del desarrollo de un extremo a otro, desde la planeación y administración del proyecto a la administración del código pasando por la compilación y el lanzamiento.
* [Azure Web Apps][web-apps] es un servicio de plataforma como servicio (PaaS) para hospedar aplicaciones web, API REST y back-ends para dispositivos móviles. Aunque este artículo se centra en. NET, hay varias opciones de plataforma de desarrollo adicionales compatibles.
* [Application Insights][application-insights] es un servicio propio de Application Performance Management (APM) extensible para desarrolladores web en varias plataformas.

### <a name="alternative-devops-tooling-options"></a>Opciones alternativas de herramientas de DevOps

Aunque este artículo se centra en Visual Studio Team Services, se puede usar [Team Foundation Server][team-foundation-server] como sustituto en un entorno local. Como alternativa, también puede encontrar un conjunto de tecnologías que se usan conjuntamente para lograr un aprovechamiento de la canalización de desarrollo de código abierto [Jenkins][jenkins-on-azure].

Desde una perspectiva de infraestructura como código, las [plantillas de Azure Resource Manager][arm-templates] se incluyen como parte del proyecto de Azure DevOps, pero también podría usar [Terraform][terraform] o [Chef][chef] si ha invertido en ellos. Si prefiere una implementación basada en una infraestructura como servicio (IaaS) y necesita administración de la configuración, podría usar [Azure Automation State Configuration][desired-state-configuration], [Ansible][ansible] o [Chef][chef].

### <a name="alternatives-to-web-app-hosting"></a>Alternativas al hospedaje en Web Apps

Alternativas al hospedaje en Azure Web Apps:

* [Máquina virtual][compare-vm-hosting]: para cargas de trabajo que requieren un alto grado de control, o dependen de componentes o servicios del sistema operativo que no son posibles con Web Apps (por ejemplo, la caché global de ensamblados de Windows o COM).
* [Hospedaje de contenedores][azure-containers]: donde hay dependencias del sistema operativo y portabilidad de hospedaje o densidad de hospedaje, también hay requisitos.
* [Service Fabric][service-fabric]: una buena opción si la arquitectura de cargas de trabajo está centrada en torno a componentes distribuidos que se benefician de su implementación y ejecución en un clúster con un alto grado de control. Service Fabric también se puede utilizar para hospedar contenedores.
* [Funciones de Azure sin servidor][azure-functions]: una buena opción si la arquitectura de cargas de trabajo está centrada en torno a componentes distribuidos más precisos, que requieren dependencias mínimas, donde solo se necesita que los componentes individuales se ejecuten a petición (no de forma continua) y en los que la orquestación de componentes no es necesaria.

### <a name="devops"></a>DevOps

La **[integración continua (CI)][continuous-integration]** debe tener como objetivo mostrar una compilación sólida, con más de un desarrollador individual o equipo que constantemente realizan pequeños cambios frecuentes en el código base compartido.
Como parte de la canalización de integración continua debe

* Insertar en el repositorio pequeñas cantidades de código frecuentemente (para evitar el procesamiento por lotes de cambios más grandes o complejos ya que estos pueden ser más difíciles de combinar correctamente)
* Realizar una prueba unitaria de los componentes de la aplicación con suficiente cobertura de código (incluidas las rutas incorrectas)
* Garantizar que la compilación se ejecuta en la rama (o tronco) compartida maestra. Esta rama debe ser estable y permanecer "lista para implementación". Se deben aislar cambios incompletos o en curso en una rama independiente con combinaciones frecuentes "de integración hacia delante" para evitar conflictos posteriores.

La **[entrega continua (CD)][continuous-delivery]** debe tener como objetivo mostrar no solo una compilación estable, sino también una implementación estable. Esto dificulta la comprensión de la entrega continua y requiere una configuración específica para el entorno y un mecanismo para configurar esos valores correctamente.

Además se necesita una cobertura suficiente de las pruebas de integración para asegurarse de que los distintos componentes están configurados y funcionando correctamente de un extremo a otro.

Puede que también necesite configurar y restablecer datos específicos del entorno y administrar versiones del esquema de la base de datos.

La entrega continua también se puede ampliar a los entornos de pruebas de carga y pruebas de aceptación de usuario.

La entrega continua se beneficia de la supervisión continua, idealmente en todos los entornos.
Para faciliar la coherencia y confiabilidad de las implementaciones y las pruebas de integración de los entornos, se pueden usar scripts de creación y configuración o la infraestructura de hospedaje (algo que es considerablemente más fácil para cargas de trabajo basadas en la nube; consulte la infraestructura de Azure como código). Esto se conoce también como ["infraestructura como código"][infra-as-code].

* Inicie la entrega continua tan pronto como sea posible en el ciclo de vida del proyecto. Cuanto más tarde se realice, más difícil será.
* Se le debe dar la misma prioridad a las pruebas unitarias y de integración que a las características del proyecto
* Uso de paquetes de implementación independientes del entorno y administración de configuración específica del entorno a lo largo del proceso de lanzamiento.
* Proteja la configuración confidencial dentro de las herramientas de administración de versiones o mediante una llamada a un módulo de seguridad de hardware (HSM) o instancia de [Key Vault][azure-key-vault] durante el proceso de lanzamiento. No almacene configuración confidencial dentro del control de código fuente.

**Aprendizaje continuo**: la supervisión más eficaz de un entorno de implementación continua la proporcionan las herramientas de supervisión de rendimiento de aplicaciones como, por ejemplo, [Application Insights][application-insights] de Microsoft. Un nivel suficiente de supervisión de la carga de trabajo de una aplicación resulta vital para entender los errores y el rendimiento con carga. [App Insights se puede integrar en VSTS para habilitar la supervisión continua de la canalización de implementación continua][app-insights-cd-monitoring]. Esta se puede usar para habilitar una progresión continua a la siguiente fase, sin intervención del usuario, o una reversión si se detecta una alerta.

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
* Debe tener una cuenta existente de Visual Studio Team Services (VSTS). Obtenga más información acerca de la [creación de una cuenta de Visual Studio Team Services (VSTS)][vsts-account-create].

### <a name="walk-through"></a>Tutorial

En este escenario va a usar el proyecto de Azure DevOps para crear la canalización de CI/CD.

El proyecto de DevOps implementará App Service, un plan de App Service y un recurso de App Insights, y configurará el proyecto de Visual Studio Team Services en su lugar.

Una vez que tenga implementado el proyecto de DevOps y se haya completado la compilación, revise los cambios de código asociados, los elementos de trabajo y los resultados de las pruebas. Como puede observar, no se muestra ningún resultado de la prueba, ya que el código no contiene pruebas para ejecutarlas.

Revise las definiciones de versión. Tenga en cuenta que se ha configurado una canalización de versión que ha permitido publicar nuestra aplicación en Dev. Observe que hay un **desencadenador de implementación continua** establecido en el artefacto de compilación **Drop**, con versiones automáticas en los entornos de Dev. Como parte de un proceso de implementación continua, puede que vea extenderse versiones en varios entornos. Una versión puede abarcar la infraestructura (mediante técnicas como la de infraestructura como código) y también implementar los paquetes de aplicación necesarios así como todas las tareas posteriores a la configuración.

**Consideraciones adicionales.**

* Considere la posibilidad de usar una de las [tareas de tokenización][vsts-tokenization] que están disponibles en Marketplace de VSTS.
* Considere la posibilidad de usar la tarea [Deploy: Azure Key Vault][download-keyvault-secrets] de VSTS para descargar los secretos desde Azure Key Vault en su versión. Posteriormente, puede usar esos secretos como variables que formen parte de la definición de versión y no debería almacenarlos en el control de código fuente.
* Considere la posibilidad de usar [variables de versión][vsts-release-variables] en las definiciones de versión para impulsar los cambios de configuración en los entornos. Las variables de versión se pueden limitar a una versión completa o a un entorno determinado. Si usa variables de información secreta, asegúrese de seleccionar el icono de candado.
* Piense en la posibilidad de usar [puertas de implementación][vsts-deployment-gates] en la canalización de versión. Esto le permite aprovechar los datos de supervisión en asociación con sistemas externos (por ejemplo, de administración de incidentes o con sistemas personalizados adicionales) para determinar si se debe promover una versión.
* Cuando se requiere intervención manual en una canalización de versión, piense sobre la conveniencia de usar la funcionalidad de [aprobaciones][vsts-approvals].
* Considere el uso de [Application Insights][application-insights] y de herramientas de supervisión adicionales lo antes posible en la versión de canalización. La mayoría de las organizaciones solo empiezan a supervisar en el entorno de producción, aunque se podrían identificar posibles errores en un punto anterior del proceso y evitar que los usuarios de producción se vean afectados.

## <a name="pricing"></a>Precios

El costo de Visual Studio Team Services dependerá del número de usuarios de su organización que requieren acceso, además de factores como el número de versiones o compilaciones simultáneas necesarias y el número de usuarios de prueba. La información se detalla más adelante en la [página de precios de VSTS][vsts-pricing-page].

* [Visual Studio Team Services (VSTS)][vsts-pricing-calculator] es un servicio que le permite administrar el ciclo de vida del desarrollo y se paga por usuario y por mes. Puede que se produzcan otros gastos adicionales en función de las canalizaciones simultáneas que se necesiten, además de por otros usuarios de prueba o licencias de usuario básico adicionales.

## <a name="related-resources"></a>Recursos relacionados

* [¿Qué es DevOps?][devops-whatis]
* [DevOps en Microsoft: ¿cómo se trabaja con Visual Studio Team Services?][devops-microsoft]
* [Tutoriales detallados: DevOps con Visual Studio Team Services][devops-with-vsts]
* [Creación de una canalización de CI/CD para .NET con un proyecto de Azure DevOps][devops-project-create]

<!-- links -->
[ansible]: /azure/ansible/
[application-insights]: /azure/application-insights/app-insights-overview
[app-service-reference-architecture]: /azure/architecture/reference-architectures/app-service-web-app/
[azure-free-account]: https://azure.microsoft.com/free/?WT.mc_id=A261C142F
[arm-templates]: /azure/azure-resource-manager/resource-group-overview#template-deployment
[architecture]: ./media/devops-dotnet-webapp/architecture-devops-dotnet-webapp.png
[availability]: /azure/architecture/checklist/availability
[chef]: /azure/chef/
[design-patterns-availability]: /azure/architecture/patterns/category/availability
[design-patterns-resiliency]: /azure/architecture/patterns/category/resiliency
[design-patterns-scalability]: /azure/architecture/patterns/category/performance-scalability
[design-patterns-security]: /azure/architecture/patterns/category/security
[desired-state-configuration]: /azure/automation/automation-dsc-overview
[devops-microsoft]: /azure/devops/devops-at-microsoft/
[devops-with-vsts]: https://almvm.azurewebsites.net/labs/vsts/
[application-insights]: https://azure.microsoft.com/en-gb/services/application-insights/
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
[vsts-account-create]: /vsts/organizations/accounts/create-account-msa-or-work-student?view=vsts
[vsts-approvals]: /vsts/pipelines/release/approvals/approvals?view=vsts
[devops-project]: https://portal.azure.com/?feature.customportal=false#create/Microsoft.AzureProject
[vsts-deployment-gates]: /vsts/pipelines/release/approvals/gates?view=vsts
[vsts-pricing-calculator]: https://azure.com/e/498aa024454445a8a352e75724f900b1
[vsts-pricing-page]: https://azure.microsoft.com/en-us/pricing/details/visual-studio-team-services/
[vsts-release-variables]: /vsts/pipelines/release/variables?view=vsts&tabs=batch
[vsts-tokenization]: https://marketplace.visualstudio.com/search?term=token&target=VSTS&category=All%20categories&sortBy=Relevance
[azure-key-vault]: /azure/key-vault/key-vault-overview
[infra-as-code]: https://blogs.msdn.microsoft.com/mvpawardprogram/2018/02/13/infrastructure-as-code/
[team-foundation-server]: https://visualstudio.microsoft.com/tfs/
[infra-as-code]: https://blogs.msdn.microsoft.com/mvpawardprogram/2018/02/13/infrastructure-as-code/
[service-fabric]:/azure/service-fabric/
[azure-functions]:/azure/azure-functions/
[azure-containers]:https://azure.microsoft.com/en-us/overview/containers/
[compare-vm-hosting]:/azure/app-service/choose-web-site-cloud-service-vm
[app-insights-cd-monitoring]:/azure/application-insights/app-insights-vsts-continuous-monitoring
[azure-region-pair-bcdr]:/azure/best-practices-availability-paired-regions
[devops-project-create]: /vsts/pipelines/apps/cd/azure/azure-devops-project-aspnetcore?view=vsts
[security]: /azure/security/