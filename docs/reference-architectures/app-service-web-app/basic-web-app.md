---
title: "Aplicación web básica"
description: "Arquitectura recomendada para una aplicación web básica que se ejecuta en Microsoft Azure."
author: MikeWasson
ms.date: 11/23/2016
cardTitle: Basic web application
ms.openlocfilehash: b7475c4087a184bb7608d0c45ffecee912c920d7
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2017
---
# <a name="basic-web-application"></a>Aplicación web básica
[!INCLUDE [header](../../_includes/header.md)]

Esta arquitectura de referencia muestra un conjunto de prácticas demostradas para una aplicación web que usa [Azure App Service][app-service] y [Azure SQL Database][sql-db]. [**Implemente esta solución.**](#deploy-the-solution)

![[0]][0]

*Descargue un [archivo Visio][visio-download] de esta arquitectura.*

## <a name="architecture"></a>Arquitectura 

> [!NOTE]
> Esta arquitectura no se centra en el desarrollo de aplicaciones y no supone ningún marco de trabajo de la aplicación concreto. El objetivo es comprender cómo encajan entre sí los distintos servicios de Azure.
>
>

La arquitectura consta de los siguientes componentes:

* **Grupo de recursos**. Un [grupo de recursos](/azure/azure-resource-manager/resource-group-overview) es un contenedor lógico de recursos de Azure.
* **Aplicación de App Service**. [Azure App Service][app-service] es una plataforma completamente administrada para crear e implementar aplicaciones en la nube.     
* **Plan de App Service**. Un [plan de App Service][app-service-plans] proporciona las máquinas virtuales (VM) administradas que hospedan su aplicación. Todas las aplicaciones asociadas con un plan se ejecutan en las mismas instancias de máquina virtual.

* **Ranuras de implementación**.  Una [ranura de implementación][deployment-slots] permite preconfigurar una implementación y, posteriormente, intercambiarla con la implementación de producción. De este modo, evita realizar la implementación directamente en producción. Consulte la sección [Manejabilidad](#manageability-considerations) para obtener recomendaciones específicas.

* **Dirección IP**. La aplicación de App Service tiene una dirección IP pública y un nombre de dominio. El nombre de dominio es un subdominio de `azurewebsites.net`, como `contoso.azurewebsites.net`. Para usar un nombre de dominio personalizado, como `contoso.com`, cree registros de servicio de nombre de dominio (DNS) que asignen el nombre de dominio personalizado a la dirección IP. Para más información, consulte [Configurar un nombre de dominio personalizado en Azure App Service][custom-domain-name].
* **Azure SQL Database**. [SQL Database][sql-db] es una base de datos como servicio relacional en la nube.
* **Servidor lógico**. En Azure SQL Database, un servidor lógico hospeda las bases de datos. Se pueden crear varias bases de datos por servidor lógico.
* **Azure Storage**. Cree una cuenta de almacenamiento de Azure con un contenedor de blobs para almacenar los registros de diagnóstico.
* **Azure Active Directory** (Azure AD). Use Azure AD u otro proveedor de identidades para la autenticación.

## <a name="recommendations"></a>Recomendaciones

Los requisitos pueden diferir de los de la arquitectura que se describe aquí. Use las recomendaciones de esta sección como punto de partida.

### <a name="app-service-plan"></a>Plan de App Service
Use los niveles Standard o Premium, ya que admiten las opciones de Escalabilidad horizontal, Escala automática y Capa de sockets seguros (SSL). Cada nivel admite varios *tamaños de instancia*, que se diferencian por el número de núcleos y la memoria. Puede cambiar el nivel o el tamaño de la instancia después de crear un plan. Para más información sobre los planes de App Service, consulte [Precios de App Service ][app-service-plans-tiers].

Se le cobra por las instancias del plan de App Service, aunque la aplicación esté detenida. Asegúrese de eliminar los planes que no use (por ejemplo, las implementaciones de prueba).

### <a name="sql-database"></a>SQL Database
Use la [versión V12][sql-db-v12] de SQL Database. SQL Database admite los [niveles de servicio][sql-db-service-tiers] Basic, Standard y Premium, con varios niveles de rendimiento dentro de cada nivel medidos en [unidades de transmisión de datos (DTU)][sql-dtu]. Planee la capacidad, y elija un nivel de servicio y un nivel de rendimiento que se ajusten a sus necesidades.

### <a name="region"></a>Region
Aprovisione el plan de App Service y la instancia de SQL Database en la misma región para minimizar la latencia de red. Por lo general, elija la región más cercana a los usuarios.

El grupo de recursos también tiene una región, que especifica dónde se almacenan los metadatos de la implementación. Coloque el grupo de recursos y sus recursos en la misma región. Esto puede mejorar la disponibilidad durante la implementación. 

## <a name="scalability-considerations"></a>Consideraciones sobre escalabilidad

Una de las ventajas principales de Azure App Service es la posibilidad de escalar la aplicación en función de la carga. Estas son algunas consideraciones que se deben tener en cuenta al planear el escalado de la aplicación.

### <a name="scaling-the-app-service-app"></a>Ajuste de escala de la aplicación de App Service

Hay dos formas de escalar una aplicación de App Service:

* *Escalar verticalmente*, que significa cambiar el tamaño de la instancia. El tamaño de la instancia determina la memoria, el número de núcleos y el almacenamiento en cada instancia de máquina virtual. La escalabilidad vertical se puede ajustar manualmente cambiando el tamaño de la instancia o el nivel del plan.  

* *Escalar horizontalmente*, que significa agregar instancias para controlar el aumento de la carga. Cada nivel de precios tiene un número máximo de instancias. 

  La escalabilidad horizontal se puede ajustar manualmente cambiando el recuento de instancias, o bien se puede usar el [escalado automático][web-app-autoscale] para que Azure agregue o quite instancias automáticamente según la programación y/o las métricas de rendimiento. Cada operación de escala se realiza rápidamente y, normalmente, en unos segundos. 

  Para habilitar el escalado automático, cree un *perfil* de escalado automático que defina el número mínimo y máximo de instancias. Los perfiles se pueden programar. Por ejemplo, podría crear perfiles distintos para los días laborables y los fines de semana. De manera opcional, un perfil contiene reglas sobre cuándo agregar o quitar instancias. (Ejemplo: Agregar dos instancias si el uso de la CPU es superior al 70 % durante 5 minutos).
  
Recomendaciones para el ajuste de escala de una aplicación web:

* En la medida de lo posible, evite la escalabilidad vertical y horizontal, ya que podría desencadenar un reinicio de la aplicación. En su lugar, seleccione un nivel y un tamaño que satisfagan sus requisitos de rendimiento con la carga típica y, a continuación, escale horizontalmente las instancias para controlar los cambios en el volumen de tráfico.    
* Habilite el escalado automático. Si la aplicación tiene una carga de trabajo normal predecible, cree perfiles para programar los recuentos de instancias con antelación. Si la carga de trabajo no es predecible, use el escalado automático basado en reglas para reaccionar a los cambios de carga cuando se produzcan. Puede combinar ambos enfoques.
* El uso de la CPU suele ser una buena métrica para las reglas de escalado automático. Sin embargo, debería realizar pruebas de carga de la aplicación, identificar posibles cuellos de botella y basar las reglas de escalado automático en esos datos.  
* Las reglas de escalado automático incluyen un período de *enfriamiento*, que es el intervalo que se debe esperar una vez completada una acción de ajuste de escala antes de iniciar la siguiente. El período de enfriamiento permite que el sistema se estabilice antes de realizar otro ajuste de escala. Establezca un período de enfriamiento más corto para agregar instancias y uno más prolongado para quitarlas. Por ejemplo, establezca 5 minutos para agregar una instancia y 60 minutos para quitarla. Es mejor agregar nuevas instancias rápidamente bajo una carga intensa para controlar el tráfico adicional y, a continuación, reducir el escalado gradualmente.

### <a name="scaling-sql-database"></a>Ajuste de escala de SQL Database
Si necesita un nivel de servicio o de rendimiento superior para SQL Database, puede escalar verticalmente bases de datos individuales sin tiempo de inactividad de la aplicación. Si quiere obtener más información, consulte el artículo [Opciones y rendimiento de Base de datos SQL: comprender lo que está disponible en cada nivel de servicio][sql-db-scale].

## <a name="availability-considerations"></a>Consideraciones sobre disponibilidad
En el momento de escribir este artículo, el acuerdo de nivel de servicio (SLA) de App Service es del 99,95 % y el SLA de SQL Database es del 99,99 % para los niveles Basic, Standard y Premium. 

> [!NOTE]
> El SLA de App Service se aplica tanto a instancias únicas como múltiples.  
>
>

### <a name="backups"></a>Copias de seguridad
En caso de pérdida de datos, SQL Database proporciona restauración a un momento dado y restauración geográfica. Estas características están disponibles en todos los niveles y se habilitan automáticamente. No es necesario programar ni administrar las copias de seguridad. 

- Use la restauración a un momento dado para la [recuperación de un error humano][sql-human-error] mediante el restablecimiento de la base de datos a un momento anterior en el tiempo. 
- Use la restauración geográfica para la [recuperación de una interrupción del servicio][sql-outage-recovery] mediante la restauración de una base de datos a partir de una copia de seguridad con redundancia geográfica. 

Para más información, consulte el tema sobre la [continuidad del negocio en la nube y la recuperación ante desastres de la base datos con SQL Database][sql-backup].

App Service proporciona una característica de [copia de seguridad y restauración][web-app-backup] para los archivos de la aplicación. Sin embargo, tenga en cuenta que los archivos de los que se realiza la copia de seguridad incluyen la configuración de la aplicación en texto sin formato y pueden incluir información confidencial, como cadenas de conexión. Evite usar la característica de copia de seguridad de App Service para realizar una copia de seguridad de las bases de datos SQL, ya que exporta la base de datos a un archivo .bacpac de SQL, que consume unidades [DTU][sql-dtu]. En su lugar, use la restauración a un momento dado de SQL Database descrita anteriormente.

## <a name="manageability-considerations"></a>Consideraciones sobre la manejabilidad
Cree grupos de recursos independientes para los entornos de producción, desarrollo y pruebas. Esto facilita la administración de implementaciones, la eliminación de implementaciones de prueba y la asignación de derechos de acceso.

Al asignar recursos a grupos de recursos, tenga en cuenta lo siguiente:

* Ciclo de vida. En general, coloque los recursos con el mismo ciclo de vida en el mismo grupo de recursos.
* Acceso. Puede usar el [control de acceso basado en rol][rbac] (RBAC) para aplicar directivas de acceso a los recursos de un grupo.
* Facturación. Puede ver los costes acumulados del grupo de recursos.  

Para más información, consulte [Información general de Azure Resource Manager](/azure/azure-resource-manager/resource-group-overview).

### <a name="deployment"></a>Implementación
La implementación implica dos pasos:

1. Aprovisionamiento de los recursos de Azure. Se recomienda usar las [plantillas de Azure Resource Manager][arm-template] para este paso. Las plantillas facilitan la automatización de las implementaciones a través de PowerShell o de la interfaz de línea de comandos (CLI) de Azure.
2. Implementación de la aplicación (código, archivos binarios y archivos de contenido). Tiene varias opciones, incluida la implementación desde un repositorio de Git local, mediante Visual Studio, o bien la implementación continua desde el control de código fuente basado en la nube. Consulte [Implementación de la aplicación en Azure App Service][deploy].  

Una aplicación de App Service siempre tiene una ranura de implementación denominada `production`, que representa el sitio de producción en vivo. Se recomienda crear un espacio de ensayo para la implementación de actualizaciones. Las ventajas de usar un espacio de ensayo incluyen:

* Puede comprobar si la implementación finalizó correctamente antes de pasarla a producción.
* La implementación en un espacio de ensayo garantiza que todas las instancias estén preparadas antes de pasarlas a producción. Muchas aplicaciones presentan tiempos de preparación y arranque en frío considerables.

También se recomienda crear un tercer espacio que contendrá la última implementación correcta conocida. Después de intercambiar ensayo y producción, mueva la implementación de producción anterior (que ahora está en la fase de ensayo) en el último espacio correcto conocido. De este modo, si detecta un problema más tarde, puede revertir rápidamente a la última versión correcta conocida.

![[1]][1]

Si realiza la reversión a una versión anterior, asegúrese de que los cambios del esquema de base de datos sean compatibles con versiones anteriores.

No use espacios en la implementación de producción para pruebas, ya que todas las aplicaciones del mismo plan de App Service comparten las mismas instancias de máquina virtual. Por ejemplo, las pruebas de carga pueden degradar el sitio de producción en vivo. En su lugar, use planes de App Service independientes para producción y prueba. Al colocar las implementaciones de prueba en un plan independiente, se aíslan de la versión de producción.

### <a name="configuration"></a>Configuración
Almacene los valores de configuración como [valores de configuración de la aplicación][app-settings]. Defina la configuración de la aplicación en las plantillas de Resource Manager, o bien mediante PowerShell. En tiempo de ejecución, la configuración de la aplicación está disponible para la aplicación en forma de variables de entorno.

Nunca compruebe contraseñas, claves de acceso ni cadenas de conexión en el control de código fuente. En su lugar, páselas como parámetros a un script de implementación que almacene estos valores como valores de configuración de la aplicación.

Cuando intercambia una ranura de implementación, los valores de configuración de la aplicación se intercambian de forma predeterminada. Si necesita una configuración diferente para producción y ensayo, puede crear configuraciones de la aplicación que se ciñan a un espacio y no se intercambien.

### <a name="diagnostics-and-monitoring"></a>Diagnóstico y supervisión
Habilite el [registro de diagnóstico][diagnostic-logs], incluido el registro de aplicaciones y el registro del servidor web. Configure el registro para usar Blob Storage. Con fines de rendimiento, cree una cuenta de almacenamiento independiente para contener los registros de diagnóstico. No use la misma cuenta de almacenamiento para los registros y los datos de la aplicación. Para obtener más instrucciones sobre el registro, consulte la [Guía de supervisión y diagnóstico][monitoring-guidance].

Use un servicio como [New Relic][new-relic] o [Application Insights][app-insights] para supervisar el rendimiento de la aplicación y el comportamiento sometido a carga. Tenga en cuenta los [límites de velocidad de datos][app-insights-data-rate] de Application Insights.

Realice pruebas de carga mediante una herramienta como [Visual Studio Team Services][vsts]. Para obtener una introducción general del análisis del rendimiento de las aplicaciones en la nube, consulte [Performance Analysis Primer][perf-analysis] (Manual básico de análisis del rendimiento).

Sugerencias para solucionar problemas de la aplicación:

* Use la [hoja de solución de problemas][troubleshoot-blade] de Azure Portal para buscar soluciones a problemas comunes.
* Habilite [Secuencias de registro][web-app-log-stream] para ver la información de registro prácticamente en tiempo real.
* El [panel Kudu][kudu] dispone de varias herramientas para supervisar y depurar la aplicación. Para obtener más información, consulte [Azure Websites online tools you should know about][kudu] [Herramientas en línea de Azure Websites que debe conocer (entrada de blog)]. Puede acceder al panel Kudu desde Azure Portal. Abra la hoja de su aplicación y haga clic en **Herramientas**; a continuación, haga clic en **Kudu**.
* Si usa Visual Studio, consulte el artículo [Solución de problemas de una aplicación web en Azure App Service con Visual Studio][troubleshoot-web-app] para obtener sugerencias de depuración y solución de problemas.

## <a name="security-considerations"></a>Consideraciones sobre la seguridad
En esta sección se enumeran las consideraciones de seguridad que son específicas de los servicios de Azure descritos en este artículo. No es una lista completa de procedimientos de seguridad recomendados. Si desea conocer algunas otras consideraciones de seguridad, consulte [Protección de una aplicación en Azure App Service][app-service-security].

### <a name="sql-database-auditing"></a>Auditoría de SQL Database
La auditoría puede ayudarle a mantener el cumplimiento de normativas, y a conocer las discrepancias y anomalías que pueden indicar problemas en el negocio o infracciones de seguridad sospechosas. Consulte [Introducción a la auditoría de SQL Database][sql-audit].

### <a name="deployment-slots"></a>Ranuras de implementación
Cada ranura de implementación tiene una dirección IP pública. Proteja los espacios que no sean de producción mediante el [inicio de sesión de Azure Active Directory][aad-auth] para que solo los miembros de los equipos de desarrollo y DevOps puedan acceder a esos puntos de conexión.

### <a name="logging"></a>Registro
Los registros nunca deben registrar las contraseñas de los usuarios ni otra información que pudiera utilizarse para cometer fraudes de identidad. Borre esos detalles de los datos antes de almacenarlos.   

### <a name="ssl"></a>SSL
Una aplicación de App Service incluye un punto de conexión SSL en un subdominio de `azurewebsites.net` sin costo adicional alguno. El punto de conexión SSL incluye un certificado comodín para el dominio `*.azurewebsites.net`. Si usa un nombre de dominio personalizado, debe proporcionar un certificado que coincida con el dominio personalizado. El enfoque más sencillo consiste en adquirir un certificado directamente a través de Azure Portal. También puede importar certificados de otras entidades emisoras de certificados. Para obtener más información, consulte [Compra y configuración de un certificado SSL para el Servicio de aplicaciones de Azure][ssl-cert].

Como procedimiento recomendado de seguridad, la aplicación debe exigir HTTPS mediante el redireccionamiento de las solicitudes HTTP. Puede hacerlo dentro de la aplicación o usar una regla de reescritura de direcciones URL como se describe en [Habilitación de HTTPS para una aplicación en Azure App Service][ssl-redirect].

### <a name="authentication"></a>Autenticación
Se recomienda realizar la autenticación a través de un proveedor de identidades (IDP), como Azure AD, Facebook, Google o Twitter. Use OAuth 2 u OpenID Connect (OIDC) para el flujo de autenticación. Azure AD proporciona funcionalidad para administrar usuarios y grupos, crear roles de aplicación, integrar las identidades locales y consumir servicios de back-end, como Office 365 y Skype Empresarial.

Evite que la aplicación administre credenciales e inicios de sesión de usuario directamente, ya que podría crear una superficie expuesta a ataques.  Como mínimo, debería tener confirmación por correo electrónico, recuperación de contraseñas y autenticación multifactor; validar la seguridad de la contraseña; y almacenar hashes de contraseña de forma segura. Los proveedores de identidades importantes controlan todas esas cosas por usted, y supervisan y mejoran constantemente sus prácticas de seguridad.

Considere usar la [autenticación de App Service][app-service-auth] para implementar el flujo de autenticación de OAuth/OIDC. Las ventajas de la autenticación del App Service incluyen:

* Es fácil de configurar.
* No se requiere ningún código para los escenarios de autenticación simples.
* Admite la autorización delegada mediante tokens de acceso de OAuth para consumir recursos en nombre del usuario.
* Proporciona una memoria caché de tokens integrada.

Algunas limitaciones de la autenticación de App Service:  

* Opciones de personalización limitadas.
* La autorización delegada está restringida a un recurso de back-end por cada sesión de inicio de sesión.
* Si usa más de un IDP, no hay ningún mecanismo integrado para la detección del dominio de inicio.
* Para los escenarios multiinquilino, la aplicación debe implementar la lógica para validar el emisor del token.

## <a name="deploy-the-solution"></a>Implementación de la solución
Un ejemplo de plantilla de Resource Manager para esta arquitectura está [disponible en GitHub][paas-basic-arm-template].

Para implementar la plantilla mediante PowerShell, ejecute los siguientes comandos:

```
New-AzureRmResourceGroup -Name <resource-group-name> -Location "West US"

$parameters = @{"appName"="<app-name>";"environment"="dev";"locationShort"="uw";"databaseName"="app-db";"administratorLogin"="<admin>";"administratorLoginPassword"="<password>"}

New-AzureRmResourceGroupDeployment -Name <deployment-name> -ResourceGroupName <resource-group-name> -TemplateFile .\PaaS-Basic.json -TemplateParameterObject  $parameters
```

Para más información, consulte [Implementación de recursos con las plantillas de Azure Resource Manager][deploy-arm-template].

<!-- links -->

[aad-auth]: /azure/app-service-mobile/app-service-mobile-how-to-configure-active-directory-authentication
[app-insights]: /azure/application-insights/app-insights-overview
[app-insights-data-rate]: /azure/application-insights/app-insights-pricing
[app-service]: https://azure.microsoft.com/documentation/services/app-service/
[app-service-auth]: /azure/app-service-api/app-service-api-authentication
[app-service-plans]: /azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview
[app-service-plans-tiers]: https://azure.microsoft.com/pricing/details/app-service/
[app-service-security]: /azure/app-service-web/web-sites-security
[app-settings]: /azure/app-service-web/web-sites-configure
[arm-template]: /azure/azure-resource-manager/resource-group-overview#resource-groups
[custom-domain-name]: /azure/app-service-web/web-sites-custom-domain-name
[deploy]: /azure/app-service-web/web-sites-deploy
[deploy-arm-template]: /azure/resource-group-template-deploy
[deployment-slots]: /azure/app-service-web/web-sites-staged-publishing
[diagnostic-logs]: /azure/app-service-web/web-sites-enable-diagnostic-log
[kudu]: https://azure.microsoft.com/blog/windows-azure-websites-online-tools-you-should-know-about/
[monitoring-guidance]: ../../best-practices/monitoring.md
[new-relic]: http://newrelic.com/
[paas-basic-arm-template]: https://github.com/mspnp/reference-architectures/tree/master/app-service-web-app/basic-web-app/Paas-Basic/Templates
[perf-analysis]: https://github.com/mspnp/performance-optimization/blob/master/Performance-Analysis-Primer.md
[rbac]: /azure/active-directory/role-based-access-control-what-is
[resource-group]: /azure/azure-resource-manager/resource-group-overview
[sla]: https://azure.microsoft.com/support/legal/sla/
[sql-audit]: /azure/sql-database/sql-database-auditing-get-started
[sql-backup]: /azure/sql-database/sql-database-business-continuity
[sql-db]: https://azure.microsoft.com/documentation/services/sql-database/
[sql-db-overview]: /azure/sql-database/sql-database-technical-overview
[sql-db-scale]: /azure/sql-database/sql-database-service-tiers#scaling-up-or-scaling-down-a-single-database
[sql-db-service-tiers]: /azure/sql-database/sql-database-service-tiers
[sql-db-v12]: /azure/sql-database/sql-database-features
[sql-dtu]: /azure/sql-database/sql-database-service-tiers
[sql-human-error]: /azure/sql-database/sql-database-business-continuity#recover-a-database-after-a-user-or-application-error
[sql-outage-recovery]: /azure/sql-database/sql-database-business-continuity#recover-a-database-to-another-region-from-an-azure-regional-data-center-outage
[ssl-redirect]: /azure/app-service-web/web-sites-configure-ssl-certificate#bkmk_enforce
[sql-resource-limits]: /azure/sql-database/sql-database-resource-limits
[ssl-cert]: /azure/app-service-web/web-sites-purchase-ssl-web-site
[troubleshoot-blade]: https://azure.microsoft.com/updates/self-service-troubleshooting-for-app-service-web-apps-customers/
[troubleshoot-web-app]: /azure/app-service-web/web-sites-dotnet-troubleshoot-visual-studio
[visio-download]: https://archcenter.azureedge.net/cdn/app-service-reference-architectures.vsdx
[vsts]: https://www.visualstudio.com/features/vso-cloud-load-testing-vs.aspx
[web-app-autoscale]: /azure/app-service-web/web-sites-scale
[web-app-backup]: /azure/app-service-web/web-sites-backup
[web-app-log-stream]: /azure/app-service-web/web-sites-enable-diagnostic-log#streamlogs
[0]: ./images/basic-web-app.png "Arquitectura de una aplicación web de Azure básica"
[1]: ./images/paas-basic-web-app-staging-slots.png "Intercambio de ranuras para implementaciones de producción y de almacenamiento profesional"
