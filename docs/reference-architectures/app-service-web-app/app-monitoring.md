---
title: Supervisión de aplicaciones web en Azure
description: Supervise una aplicación web hospedada en Azure App Service.
author: adamboeglin
ms.date: 12/12/2018
ms.custom: azcat
ms.openlocfilehash: 2333ab0884e37354dc00113c8c40b6184fdf6ff1
ms.sourcegitcommit: 8d951fd7e9534054b160be48a1881ae0857561ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/13/2018
ms.locfileid: "53329490"
---
# <a name="web-application-monitoring-on-azure"></a>Supervisión de aplicaciones web en Azure

Las ofertas de Plataforma como servicio (PaaS) de Azure administran automáticamente los recursos de proceso y afectan a la forma de supervisar las implementaciones. Azure incluye varios servicios de supervisión, cada uno de los cuales realiza una función específica. Juntos, estos servicios ofrecen una solución completa para recopilar, analizar y actuar sobre los datos de telemetría de las aplicaciones y los recursos de Azure que consumen.

Este escenario aborda los servicios de supervisión que puede usar y describe un modelo de flujo de datos para su uso con varios orígenes de datos. Cuando se trata de supervisión, muchas herramientas y servicios funcionan con implementaciones de Azure. En este escenario, hemos elegido servicios disponibles precisamente porque son fáciles de consumir. Más adelante en este artículo se tratan otras opciones de supervisión.

## <a name="relevant-use-cases"></a>Casos de uso pertinentes

Otros casos de uso pertinentes incluyen:

- Instrumentación de una aplicación web para la supervisión de la telemetría.
- Recopilación de datos de telemetría de front-end y de back-end de una aplicación implementada en Azure.
- Supervisión de las métricas y las cuotas asociadas con los servicios de Azure.

## <a name="architecture"></a>Arquitectura

!["Diagrama de arquitectura"](./images/architecture-diagram-app-monitoring.svg)

Este escenario usa un entorno administrado de Azure para hospedar una aplicación y la capa de datos. Los datos fluyen por el escenario de la siguiente manera:

1. Un usuario interactúa con la aplicación.
2. El explorador y el servicio de aplicaciones emiten datos de telemetría.
3. Application Insights recopila y analiza datos de mantenimiento, rendimiento y uso de la aplicación.
4. Los desarrolladores y los administradores pueden revisar la información de mantenimiento, rendimiento y uso.
5. Azure SQL Database emite datos de telemetría.
6. Azure Monitor recopila y analiza las cuotas y métricas de la infraestructura.
7. Log Analytics recopila y analiza registros y métricas.
8. Los desarrolladores y los administradores pueden revisar la información de mantenimiento, rendimiento y uso.

### <a name="components"></a>Componentes

- [Azure App Service](/azure/app-service/) es un servicio de PaaS para crear y hospedar aplicaciones en máquinas virtuales administradas. La infraestructura de proceso subyacente en la que se ejecutan las aplicaciones se administra automáticamente. App Service proporciona supervisión de las cuotas de uso de los recursos y las métricas de la aplicación, registro de la información de diagnóstico y alertas basadas en métricas. Mejor aún, puede usar Application Insights para crear [pruebas de disponibilidad][availability-tests] para probar la aplicación desde distintas regiones.
- [Application Insights][application-insights] es un servicio de administración del rendimiento de las aplicaciones (APM) extensible para desarrolladores y admite varias plataformas. Supervisa la aplicación, detecta anomalías de la aplicación como un rendimiento deficiente o errores y envía datos de telemetría a Azure Portal. Application Insights se puede utilizar también para el registro, el seguimiento distribuido y las métricas de aplicación personalizadas.
- [Azure Monitor][azure-monitor] ofrece [métricas y registros][metrics] de las infraestructuras de base para la mayoría de los servicios de Azure. Puede interactuar con las métricas de varias maneras, como la representación en gráficos en Azure Portal, el acceso a ellas mediante la API REST o consultarlas con PowerShell o la CLI. Azure Monitor también ofrece sus datos directamente en [Log Analytics y otros servicios], donde los puede consultar y combinar con datos de otros orígenes del entorno local o la nube.
- [Log Analytics][log-analytics] ayuda a poner en correlación los datos de uso y rendimiento recopilados por Application Insights con los datos de configuración y rendimiento de los recursos de Azure que respaldan la aplicación. Este escenario utiliza el [agente de Azure Log Analytics][Azure Log Analytics agent] para insertar los registros de auditoría de SQL Server en Log Analytics. Puede escribir consultas y visualizar los datos en la hoja Log Analytics de Azure Portal.

## <a name="considerations"></a>Consideraciones

Un procedimiento recomendado consiste en agregar Application Insights al código durante la etapa de desarrollo mediante los [SDK de Application Insights][Application Insights SDKs] y realizar la personalización pertinente por aplicación. Estos SDK de código abierto están disponibles para la mayoría de los marcos de la aplicación. Para enriquecer y controlar los datos recopilados, incorpore el uso de los SDK para las implementaciones de pruebas y de producción en el proceso de desarrollo. El requisito principal es que la aplicación tenga una línea de visión directa o indirecta al punto de conexión de ingesta de Application Insights hospedado con una dirección con conexión a Internet. A continuación, puede agregar datos de telemetría o enriquecer una colección de telemetría existente.

La supervisión en tiempo de ejecución es otra manera fácil de empezar a trabajar. Los datos de telemetría recopilados se deben controlar mediante archivos de configuración. Por ejemplo, puede incluir métodos en tiempo de ejecución que habiliten herramientas como el [Monitor de estado de Application Insights][Application Insights Status Monitor] para implementar los SDK en la carpeta correcta y agregar las configuraciones adecuadas para comenzar la supervisión.

Al igual que Application Insights, Log Analytics proporciona herramientas para [analizar datos en orígenes][analyzing data across sources], crear consultas complejas y [enviar alertas proactivas][sending proactive alerts] sobre condiciones especificadas. También puede visualizar los datos de telemetría en [Azure Portal][the Azure portal]. Log Analytics agrega valor a servicios de supervisión existentes como [Azure Monitor][azure-monitor] y también puede supervisar entornos locales.

Tanto Application Insights como Log Analytics utilizan el [lenguaje de consulta de Azure Log Analytics][Azure Log Analytics Query Language]. También puede usar [consultas entre recursos](https://azure.microsoft.com/blog/query-across-resources) para analizar los datos de telemetría recopilados por Application Insights y Log Analytics en una sola consulta.

Azure Monitor, Application Insights y Log Analytics envían [alertas](/azure/monitoring-and-diagnostics/monitoring-overview-alerts). Por ejemplo, Azure Monitor alerta sobre métricas de nivel de plataforma como el uso de CPU, mientras que Application Insights alerta sobre métricas de nivel de aplicación como el tiempo de respuesta del servidor. Azure Monitor alerta sobre nuevos eventos en el registro de actividad de Azure, mientras que Log Analytics puede emitir alertas sobre datos de métricas o eventos de los servicios configurados para usarlo. Las [alertas unificadas de Azure Monitor](/azure/monitoring-and-diagnostics/monitoring-overview-unified-alerts) son una nueva experiencia unificada de alertas de Azure que usa una taxonomía diferente.

### <a name="alternatives"></a>Alternativas

En este artículo se describen las opciones de supervisión disponibles con características populares, pero dispone de muchas opciones, incluida la opción de crear sus propios mecanismos de registro. Un procedimiento recomendado consiste en agregar servicios de supervisión a medida que crean las capas de una solución. Estas son algunas posibles extensiones y alternativas:

- Consolidar las métricas de Azure Monitor y Application Insights en Grafana mediante el uso del [origen de datos de Azure Monitor para Grafana][Azure Monitor Data Source For Grafana].
- [Data Dog][data-dog] incluye un conector para Azure Monitor
- Automatizar las funciones de supervisión mediante [Azure Automation][Azure Automation].
- Agregar comunicación con [soluciones de ITSM][ITSM solutions].
- Ampliar Log Analytics con una [solución de administración][management solution].

### <a name="scalability-and-availability"></a>Escalabilidad y disponibilidad

Este escenario se centra en soluciones de PaaS para la supervisión en gran parte porque administran adecuadamente la disponibilidad y la escalabilidad y están respaldadas por contratos de nivel de servicio (SLA). Por ejemplo, App Services proporciona un [SLA][SLA] garantizado para su disponibilidad.

Application Insights tiene [límites][app-insights-limits] en cuanto al número de solicitudes que se pueden procesar por segundo. Si se supera el límite de solicitudes, puede experimentar limitación de mensajes. Para evitar dicha limitación, implemente el [filtrado][message-filtering] o el [muestreo][message-sampling] para reducir la velocidad de los datos

Sin embargo, las consideraciones sobre alta disponibilidad para la aplicación que se va a ejecutar son responsabilidad del desarrollador. Para obtener información sobre la escala, por ejemplo, consulte la sección [Consideraciones sobre escalabilidad](#scalability-considerations) de la arquitectura de referencia de una aplicación web básica. Después de implementar una aplicación, puede configurar pruebas para [supervisar su disponibilidad][monitor its availability] con Application Insights.

### <a name="security"></a>Seguridad

La información confidencial y los requisitos de cumplimiento afectan a la recopilación, retención y almacenamiento de los datos. Más información acerca de cómo controlan [Application Insights][application-insights] y [Log Analytics][log-analytics] los datos de telemetría.

También se pueden aplicar las siguientes consideraciones de seguridad:

- Desarrolle un plan para controlar la información personal si se permite a los desarrolladores recopilar sus propios datos o enriquecer los datos de telemetría existentes.
- Tenga en cuenta la retención de datos. Por ejemplo, Application Insights conserva los datos de telemetría durante 90 días. Archive los datos a los que desea tener acceso durante períodos más largos con Microsoft Power BI, la exportación continua o la API REST. Se aplican tarifas de almacenamiento.
- Limite el acceso a los recursos de Azure para controlar el acceso a los datos y quién puede ver los datos de telemetría de una aplicación específica. Para ayudar a bloquear el acceso a los datos de telemetría de supervisión, consulte [Recursos, roles y control de acceso en Application Insights][Resources, roles, and access control in Application Insights].
- Considere la posibilidad de controlar el acceso de lectura y escritura al código de la aplicación para impedir que los usuarios puedan agregar marcadores de versión o de etiqueta que limiten la ingesta de datos de la aplicación. Con Application Insights, no hay control sobre los elementos individuales de datos una vez que se envían a un recurso, por lo que si un usuario tiene acceso a algún dato, tendrá acceso a todos los datos de un recurso individual.
- Si es necesario, agregue mecanismos de [gobierno](/azure/security/governance-in-azure) para aplicar controles de directivas y costos a los recursos de Azure. Por ejemplo, puede utilizar Log Analytics para la supervisión relacionada con la seguridad como las directivas y el control de acceso basado en rol o usar [Azure Policy](/azure/azure-policy/azure-policy-introduction) para crear, asignar y administrar definiciones de directivas.
- Para supervisar posibles problemas de seguridad y obtener una visión centralizada del estado de seguridad de los recursos de Azure, considere el uso de [Azure Security Center](/azure/security-center/security-center-intro).

## <a name="pricing"></a>Precios

Los cargos de supervisión pueden incrementarse rápidamente, por lo que puede considerar la posibilidad de los pagos por adelantado, comprobar lo que se va a supervisar y comprobar las cuotas asociadas a cada servicio. Azure Monitor proporciona [métricas básicas][basic metrics] sin costo alguno, en tanto que la supervisión de los costos de [Application Insights][application-insights-pricing] y [Log Analytics][log-analytics] se basa en la cantidad de datos ingeridos y el número de pruebas ejecutadas.

Para comenzar, utilice la [calculadora de precios][pricing] para estimar los costos. Para ver cómo cambiarían los precios en su caso particular, cambie las distintas opciones para hacerlas coincidir con la implementación esperada.

Los datos de telemetría de Application Insights se envían a Azure Portal durante la depuración y después de que se haya publicado la aplicación. Con fines de prueba y para evitar cargos, se instrumenta un volumen limitado de datos de telemetría. Para agregar más indicadores, puede aumentar el límite de datos de telemetría. Para un control más granular, consulte [Muestreo en Application Insights][Sampling in Application Insights].

Después de la implementación, puede ver un [Live Metrics Stream][Live Metrics Stream] de indicadores de rendimiento. Estos datos no se almacenan datos (está viendo métricas en tiempo real), pero los datos de telemetría se pueden recopilar y analizar posteriormente. No se efectúa ningún cargo por los datos de Live Stream.

Log Analytics se factura por gigabyte (GB) de datos ingeridos en el servicio. Los primeros 5 GB de datos ingeridos en el servicio Azure Log Analytics cada mes se ofrecen de forma gratuita y los datos se conservan sin ningún costo los primeros 31 días en el área de trabajo de Log Analytics.

## <a name="next-steps"></a>Pasos siguientes

Consulte estos recursos diseñados para ayudarle a empezar a trabajar con su propia solución de supervisión:

[Arquitectura de referencia de una aplicación web básica][Basic web application reference architecture]

[Inicio de la supervisión de la aplicación web ASP.NET][Start monitoring your ASP.NET Web Application]

[Recopilación de datos acerca de las máquinas virtuales de Azure][Collect data about Azure Virtual Machines]

## <a name="related-resources"></a>Recursos relacionados

[Supervisión de aplicaciones y recursos de Azure][Monitoring Azure applications and resources]

[Búsqueda y diagnóstico de excepciones en tiempo de ejecución con Azure Application Insights][Find and diagnose run-time exceptions with Azure Application Insights]

<!-- links -->
[architecture]: ./images/architecture-diagram-app-monitoring.svg
[availability-tests]: /azure/application-insights/app-insights-monitor-web-app-availability
[application-insights]: /azure/application-insights/app-insights-overview
[azure-monitor]: /azure/monitoring-and-diagnostics/monitoring-overview-azure-monitor
[metrics]: /azure/monitoring-and-diagnostics/monitoring-supported-metrics
[Log Analytics y otros servicios]: /azure/log-analytics/log-analytics-azure-storage
[log-analytics]: /azure/log-analytics/log-analytics-overview
[Azure Log Analytics agent]: https://blogs.msdn.microsoft.com/sqlsecurity/2017/12/28/azure-log-analytics-oms-agent-now-collects-sql-server-audit-logs/
[application-insights-pricing]: https://azure.microsoft.com/pricing/details/application-insights/
[Application Insights SDKs]: /azure/application-insights/app-insights-asp-net
[Application Insights Status Monitor]: https://azure.microsoft.com/updates/application-insights-status-monitor-and-sdk-updated/
[analyzing data across sources]: /azure/log-analytics/log-analytics-dashboards
[sending proactive alerts]: /azure/log-analytics/log-analytics-alerts
[the Azure portal]: /azure/log-analytics/log-analytics-tutorial-dashboards
[Azure Log Analytics Query Language]: https://docs.loganalytics.io/docs/Learn
[cross-resource queries]: https://azure.microsoft.com/blog/query-across-resources/
[alerts]: /azure/monitoring-and-diagnostics/monitoring-overview-alerts
[Alerts (Preview)]: /azure/monitoring-and-diagnostics/monitoring-overview-unified-alerts
[Azure Monitor Data Source For Grafana]: https://grafana.com/plugins/grafana-azure-monitor-datasource
[Azure Automation]: /azure/automation/automation-intro
[ITSM solutions]: https://azure.microsoft.com/blog/itsm-connector-for-azure-is-now-generally-available/
[management solution]: /azure/monitoring/monitoring-solutions
[SLA]: https://azure.microsoft.com/support/legal/sla/app-service/v1_4/
[monitor its availability]: /azure/application-insights/app-insights-monitor-web-app-availability
[Resources, roles, and access control in Application Insights]: /azure/application-insights/app-insights-resources-roles-access-control
[basic metrics]: /azure/monitoring-and-diagnostics/monitoring-supported-metrics
[pricing]: https://azure.microsoft.com/pricing/calculator/#log-analyticsc126d8c1-ec9c-4e5b-9b51-4db95d06a9b1
[Sampling in Application Insights]: /azure/application-insights/app-insights-sampling
[Live Metrics Stream]: /azure/application-insights/app-insights-live-stream
[Basic web application reference architecture]: /azure/architecture/reference-architectures/app-service-web-app/basic-web-app#scalability-considerations
[Start monitoring your ASP.NET Web Application]: /azure/application-insights/quick-monitor-portal
[Collect data about Azure Virtual Machines]: /azure/log-analytics/log-analytics-quick-collect-azurevm
[Monitoring Azure applications and resources]: /azure/monitoring-and-diagnostics/monitoring-overview
[Find and diagnose run-time exceptions with Azure Application Insights]: /azure/application-insights/app-insights-tutorial-runtime-exceptions
[data-dog]: https://www.datadoghq.com/blog/azure-monitoring-enhancements/
[app-insights-limits]: /azure/azure-subscription-service-limits#application-insights-limits
[message-filtering]: /azure/application-insights/app-insights-api-filtering-sampling
[message-sampling]: /azure/application-insights/app-insights-sampling
