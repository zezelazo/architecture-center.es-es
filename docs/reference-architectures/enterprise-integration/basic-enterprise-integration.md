---
title: Integración empresarial mediante Azure Integration Services
description: Esta arquitectura de referencia muestra cómo implementar un patrón de integración empresarial sencilla con Azure Logic Apps y Azure API Management.
services: logic-apps
author: mattfarm
ms.author: mattfarm
ms.reviewer: jonfan, estfan, LADocs
ms.topic: article
ms.date: 12/03/2018
ms.openlocfilehash: c8aa3f8b736fabd1a6701778f22a7eec9bf46ee7
ms.sourcegitcommit: e7e0e0282fa93f0063da3b57128ade395a9c1ef9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/05/2018
ms.locfileid: "52919114"
---
# <a name="basic-enterprise-integration-on-azure"></a>Integración empresarial básica en Azure

Esta arquitectura de referencia usa [Azure Integration Services][ integration-services] para orquestar las llamadas a los sistemas de back-end empresariales. Los sistemas de back-end pueden incluir sistemas de software como servicio (SaaS), servicios de Azure y servicios web existentes de su empresa.

Azure Integration Services es una colección de servicios para la integración de aplicaciones y datos. Esta arquitectura emplea dos de esos servicios: [Logic Apps][logic-apps] para orquestar los flujos de trabajo, y [API Management][apim] para crear catálogos de API. Esta arquitectura es suficiente para los escenarios de integración básica en los que las llamadas sincrónicas a servicios back-end desencadenan el flujo de trabajo. Una arquitectura más sofisticada que usa [colas y eventos](./queues-events.md) se basa en esta arquitectura básica. 

![Diagrama de la arquitectura: integración empresarial sencilla](./_images/simple-enterprise-integration.png)

## <a name="architecture"></a>Arquitectura

La arquitectura consta de los siguientes componentes:

- **Sistemas de back-end**. El lado derecho del diagrama muestra los distintos sistemas de back-end que la empresa ha implementado o usa. Aquí se incluyen los sistemas SaaS, otros servicios de Azure o servicios web que exponen puntos de conexión REST o SOAP.

- **Azure Logic Apps**. [Logic Apps][ logic-apps] es una plataforma sin servidor para la creación de flujos de trabajo empresariales que integran aplicaciones, datos y servicios. En esta arquitectura, las aplicaciones lógicas se desencadenan mediante solicitudes HTTP. También se pueden anidar flujos de trabajo para una orquestación más compleja. Logic Apps usa [conectores][logic-apps-connectors] para integrarse con los servicios usados más comúnmente. Logic Apps ofrece cientos de conectores, y además puede crear conectores personalizados.

- **Azure API Management**. [API Management][apim] es un servicio administrado para la publicación de catálogos de API de HTTP que promueve la reutilización y la capacidad de detección. API Management consta de dos componentes relacionados:

    - **Puerta de enlace de API**. La puerta de enlace de API acepta llamadas HTTP y las enruta al back-end. 

    - **Portal para desarrolladores**. Cada instancia de Azure API Management proporciona acceso al [portal para desarrolladores][apim-dev-portal]. Este portal proporciona a los desarrolladores acceso a documentación y ejemplos de código para llamar a las API. También se pueden probar las API en el portal para desarrolladores.

    En esta arquitectura, se crean API compuestas mediante la [importando de aplicaciones lógicas][apim-logic-app] como API. También se pueden importar servicios web existentes mediante la [importación de especificaciones de OpenAPI][apim-openapi] (Swagger) o la [importación de API SOAP][apim-soap] a partir de especificaciones WSDL. 

    La puerta de enlace de API ayuda a desacoplar los clientes de front-end del back-end. Por ejemplo, puede reescribir las direcciones URL o transformar las solicitudes antes de que lleguen al back-end. También controla muchos aspectos transversales como la autenticación, la compatibilidad con uso compartido de recursos entre orígenes (CORS) y el almacenamiento en caché de respuestas.

- **Azure DNS**. [Azure DNS][dns] es un servicio de hospedaje para dominios DNS. Azure DNS proporciona resolución de nombres mediante el uso de la infraestructura de Microsoft Azure. Al hospedar dominios en Azure, puede administrar los registros DNS con las mismas credenciales, API, herramientas y facturación que usa con los demás servicios de Azure. Para usar un nombre de dominio personalizado, como contoso.com, cree registros DNS que asignen el nombre de dominio personalizado a la dirección IP. Para más información, consulte el artículo sobre cómo [configurar un nombre de dominio personalizado en API Management][apim-domain].

- **Azure Active Directory (Azure AD)**. Use [Azure AD][aad] para autenticar a los clientes que llaman a la puerta de enlace de API. Azure AD admite el protocolo OpenID Connect (OIDC). Los clientes obtienen un token de acceso de Azure AD y la puerta de enlace de API [valida el token][apim-jwt] para autorizar la solicitud. Cuando se usa el nivel Estándar o Premium de API Management, Azure AD también puede proteger el acceso al portal para desarrolladores.

## <a name="recommendations"></a>Recomendaciones

Los requisitos específicos pueden diferir de la arquitectura genérica que se describe aquí. Use las recomendaciones de esta sección como punto de partida.

### <a name="api-management"></a>API Management

Use los niveles Básico, Estándar o Premium de API Management. Estos niveles ofrecen un Acuerdo de Nivel de Servicio (SLA) de producción y admiten escalabilidad horizontal dentro de la región de Azure. La capacidad de rendimiento de API Management se mide en *unidades*. Cada plan de tarifa tiene una escalabilidad horizontal máxima. El nivel Premium también admite la escalabilidad horizontal entre varias regiones de Azure. Elija su nivel en función de su conjunto de características y el nivel de rendimiento necesario. Para más información, consulte [Precios de API Management][apim-pricing] y [Capacidad de una instancia de Azure API Management][apim-capacity].

Cada instancia de Azure API Management tiene un nombre de dominio predeterminado, que es un subdominio de `azure-api.net`, por ejemplo, `contoso.azure-api.net`. Considere la posibilidad de configurar un [dominio personalizado][apim-domain] para su organización.

### <a name="logic-apps"></a>Logic Apps 

Logic Apps funciona mejor en escenarios que no requieren baja latencia para una respuesta, como por ejemplo las llamadas a API asincrónicas o de ejecución semiprolongada. Si se requiere una latencia baja, por ejemplo en una llamada que bloquea una interfaz de usuario, use otra tecnología. Por ejemplo, use Azure Functions o una API web implementada en Azure App Service. Use API Management para enfrentar la API a los consumidores de API.

### <a name="region"></a>Region

Para minimizar la latencia de red, coloque API Management y Logic Apps en la misma región. En general, elija la región más cercana a los usuarios (o más cercana a los servicios de back-end).

El grupo de recursos también tiene una región. La región especifica dónde se almacenan los metadatos de implementación y dónde ejecutar la plantilla de implementación. Coloque el grupo de recursos y sus recursos en la misma región para mejorar la disponibilidad durante la implementación.

## <a name="scalability-considerations"></a>Consideraciones sobre escalabilidad

Para aumentar la escalabilidad de API Management, agregue [directivas de almacenamiento en caché][apim-caching] donde corresponda. El almacenamiento en caché también ayuda a reducir la carga de sus servicios back-end.

Para ofrecer mayor capacidad, puede escalar horizontalmente los niveles Básico, Estándar y Premium de Azure API Management en una región de Azure. Para analizar el uso de su servicio, en el menú **Métricas**, seleccione la opción **Métrica de capacidad** y luego escale verticalmente u horizontalmente según corresponda. El proceso de actualización o escalado puede tardar entre 15 y 45 minutos en aplicarse.

Recomendaciones de escalado de un servicio API Management:

- Considere los patrones de tráfico al escalar. Los clientes con patrones de tráfico más volátiles necesitan más capacidad.

- Una capacidad coherente superior al 66 % puede indicar la necesidad de escalar verticalmente.

- Una capacidad coherente inferior al 20 % puede indicar una oportunidad para reducir verticalmente.

- Antes de habilitar la carga en producción, realice siempre una prueba de carga del servicio API Management con una carga representativa.

Con el nivel Premium, puede escalar una instancia de API Management entre varias regiones de Azure. Esto hace que API Management sea apto para un SLA superior y le permite aprovisionar servicios cercanos a los usuarios en varias regiones.


El modelo sin servidor de Logic Apps significa que no es necesario que los administradores planifiquen la escalabilidad del servicio. El servicio se escala automáticamente para satisfacer la demanda.

## <a name="availability-considerations"></a>Consideraciones sobre disponibilidad

Revise el SLA de cada servicio:

- [SLA de API Management][apim-sla]
- [SLA de Logic Apps][logic-apps-sla]

Si API Management se implementa en dos o más regiones con el nivel Premium, es apto para un Acuerdo de Nivel de Servicio superior. Consulte [Precios de API Management][apim-pricing].

### <a name="backups"></a>Copias de seguridad

Realice una [copia de seguridad][apim-backup] periódica de la configuración de API Management. Almacene los archivos de copia de seguridad en una ubicación o región de Azure diferente de la región donde se implementa el servicio. En función de su [RTO][rto], elija una estrategia de recuperación ante desastres:

* En un evento de recuperación ante desastres, aprovisione una nueva instancia de API Management, restaure la copia de seguridad en la instancia nueva y redirija los registros DNS.

* Mantenga una instancia pasiva del servicio API Management en otra región de Azure. Restaure con regularidad las copias de seguridad a esa instancia para mantenerlas sincronizadas con el servicio activo. Para restaurar el servicio durante un evento de recuperación ante desastres, solo es necesario redirigir los registros DNS. Este enfoque conlleva costos adicionales, ya que se paga por la instancia pasiva, pero reduce el tiempo de recuperación. 

Para las aplicaciones lógicas, se recomienda un enfoque de configuración como código para la copia de seguridad y la restauración. Dado que las aplicaciones lógicas son sin servidor, puede volver a crearlas rápidamente a partir de plantillas de Azure Resource Manager. Guarde las plantillas en el control de código fuente e integre las plantillas con el proceso de implementación continua e integración continua (CI/CD). En un evento de recuperación ante desastres, implemente la plantilla en una nueva región.

Si implementa una aplicación lógica en una región diferente, actualice la configuración en API Management. Puede actualizar la propiedad **Backend** de la API mediante un script de PowerShell básico.

## <a name="manageability-considerations"></a>Consideraciones sobre la manejabilidad

Cree grupos de recursos independientes para entornos de producción, desarrollo y pruebas. Los grupos de recursos independientes facilitan la administración de implementaciones, la eliminación de implementaciones de prueba y la asignación de derechos de acceso.

Cuando asigna recursos a los grupos de recursos, debe considerar estos factores:

* **Ciclo de vida**. En general, coloque los recursos que tienen el mismo ciclo de vida en el mismo grupo de recursos.

* **Acceso**. Para aplicar directivas de acceso a los recursos de un grupo, puede usar el [control de acceso basado en rol][rbac] (RBAC).

* **Facturación**. Puede ver los costos acumulados del grupo de recursos.

* **Plan de tarifa de API Management**. Use el nivel Desarrollador para los entornos de desarrollo y pruebas. Para minimizar los costos durante la preproducción, implemente una réplica del entorno de producción, ejecute las pruebas y, a continuación, apáguela.

### <a name="deployment"></a>Implementación

Use [plantillas Azure Resource Manager][ arm] para implementar los recursos de Azure. Las plantillas facilitan la automatización de las implementaciones mediante PowerShell o la CLI de Azure.

Coloque las instancias de Azure API Management y cualquier aplicación lógica individual en sus propias plantillas independientes de Resource Manager. Mediante el uso de plantillas independientes, puede almacenar los recursos en sistemas de control de código fuente. Puede implementar las plantillas en conjunto o por separado como parte de un proceso de CI/CD.

### <a name="versions"></a>Versiones

Cada vez que realiza un cambio en la configuración de una aplicación lógica o implementa una actualización con una plantilla de Resource Manager, Azure guarda una copia de dicha versión y mantiene todas las versiones que tengan un historial de ejecución. Puede usar estas versiones para hacer el seguimiento de los cambios históricos o promover una versión como configuración actual de la aplicación lógica. Por ejemplo, puede revertir una aplicación lógica a una versión anterior.

Azure API Management admite dos conceptos de control de versiones distintos, pero complementarios:

* Las *versiones* permiten a los consumidores de API elegir una versión de API en función de sus necesidades, por ejemplo, v1, v2, beta o producción.

* Las *revisiones* permiten a los administradores de API realizar cambios no importantes en una API e implementar esos cambios, junto con un registro de cambios para informar a los consumidores de API sobre dichos cambios.

Puede crear una revisión en el entorno de desarrollo e implementar ese cambio entre otros entornos con el uso de plantillas de Resource Manager. Para más información, consulte [Publicación de varias versiones de la API][apim-versions].

También puede usar las revisiones para probar una API antes de realizar los cambios actuales y permitir que los usuarios puedan acceder a ella. Sin embargo, este método no se recomienda para pruebas de carga o pruebas de integración. En su lugar, use entornos de prueba o preproducción independientes.

## <a name="diagnostics-and-monitoring"></a>Diagnóstico y supervisión

Puede usar [Azure Monitor][monitor] para la supervisión operativa tanto en API Management como en Logic Apps. Azure Monitor proporciona información según las métricas que se configuran para cada servicio y está habilitado de manera predeterminada. Para más información, consulte:

- [Supervisión de API publicadas][apim-monitor]
- [Supervisar el estado, configurar el registro de diagnósticos y activar alertas para Azure Logic Apps][logic-apps-monitor]

Cada servicio también tiene estas opciones:

* Para realizar análisis más exhaustivos y agregarlos a los paneles, envíe registros de Logic Apps a [Azure Log Analytics][logic-apps-log-analytics].

* Para la supervisión de DevOps, puede configurar Azure Application Insights para API Management.

* API Management admite la [plantilla de solución de Power BI para análisis de API personalizados][apim-pbi]. Puede usar esta plantilla de solución para crear su propia solución de análisis. Los informes están disponibles en Power BI para los usuarios empresariales.

## <a name="security-considerations"></a>Consideraciones sobre la seguridad

Aunque esta lista no describe completamente todos los procedimientos recomendados de seguridad, estas son algunas consideraciones de seguridad que se aplican específicamente a esta arquitectura:

* El servicio Azure API Management tiene una dirección IP pública fija. Restrinja el acceso para llamar a los puntos de conexión de Logic Apps a solo la dirección IP de API Management. Para más información, consulte [Restricción de las direcciones IP entrantes][logic-apps-restrict-ip].

* Para asegurarse de que los usuarios tienen niveles de acceso adecuados, utilice el control de acceso basado en rol (RBAC).

* Proteja los puntos de conexión de API públicos en API Management con OAuth o bien OpenID Connect. Para proteger los puntos de conexión de API públicos, configure un proveedor de identidades y agregue una directiva de validación de JSON Web Token (JWT). Para más información, consulte [Protección de una API mediante OAuth 2.0 con Azure Active Directory y API Management][apim-oauth].

* Conéctese a servicios back-end desde API Management mediante certificados mutuos.

* Exija HTTPS en las API de API Management.

### <a name="storing-secrets"></a>Almacenamiento de secretos

Nunca compruebe contraseñas, claves de acceso ni cadenas de conexión en el control de código fuente. Si estos valores son necesarios, protéjalos e impleméntelos mediante las técnicas oportunas. 

Si una aplicación lógica requiere valores confidenciales que no se pueden crear dentro de una conexión, almacene esos valores en Azure Key Vault y haga referencia a ellos desde una plantilla de Resource Manager. Utilice parámetros de plantilla de implementación y archivos de parámetros para cada entorno. Para más información, consulte [Parámetros seguros y entradas dentro de un flujo de trabajo][logic-apps-secure].

API Management administra los secretos con objetos denominados *valores con nombre* o *propiedades*. Estos objetos almacenan de forma segura los valores a los que se puede acceder a través de directivas de API Management. Para más información, consulte [Cómo usar valores con nombre en las directivas de Azure API Management][apim-properties].

## <a name="cost-considerations"></a>Consideraciones sobre el costo

Se le cobra por todas las instancias de API Management cuando están en ejecución. Si ha escalado verticalmente y no necesita ese nivel de rendimiento todo el tiempo, reduzca verticalmente de forma manual o configure la [escalabilidad automática][apim-autoscale].

Logic Apps usa un modelo [sin servidor](/azure/logic-apps/logic-apps-serverless-overview). La facturación se calcula en función de la acción y la ejecución del conector. Para obtener más información, consulte [Precios de Logic Apps](https://azure.microsoft.com/pricing/details/logic-apps/). Actualmente no existen consideraciones sobre los niveles en Logic Apps.

## <a name="next-steps"></a>Pasos siguientes

Para obtener mayor confiabilidad y escalabilidad, utilice colas de mensajes y eventos para desacoplar los sistemas de back-end. Este patrón se muestra en la siguiente arquitectura de referencia de esta serie: [Integración empresarial mediante colas de mensajes y eventos](./queues-events.md).

<!-- links -->

[aad]: /azure/active-directory
[apim]: /azure/api-management
[apim-autoscale]: /azure/api-management/api-management-howto-autoscale
[apim-backup]: /azure/api-management/api-management-howto-disaster-recovery-backup-restore
[apim-caching]: /azure/api-management/api-management-howto-cache
[apim-capacity]: /azure/api-management/api-management-capacity
[apim-dev-portal]: /azure/api-management/api-management-key-concepts#a-namedeveloper-portal-a-developer-portal
[apim-domain]: /azure/api-management/configure-custom-domain
[apim-jwt]: /azure/api-management/policies/authorize-request-based-on-jwt-claims
[apim-logic-app]: /azure/api-management/import-logic-app-as-api
[apim-monitor]: /azure/api-management/api-management-howto-use-azure-monitor
[apim-oauth]: /azure/api-management/api-management-howto-protect-backend-with-aad
[apim-openapi]: /azure/api-management/import-api-from-oas
[apim-pbi]: http://aka.ms/apimpbi
[apim-pricing]: https://azure.microsoft.com/pricing/details/api-management/
[apim-properties]: /azure/api-management/api-management-howto-properties
[apim-sla]: https://azure.microsoft.com/support/legal/sla/api-management/
[apim-soap]: /azure/api-management/import-soap-api
[apim-versions]: /azure/api-management/api-management-get-started-publish-versions
[arm]: /azure/azure-resource-manager/resource-group-authoring-templates
[dns]: /azure/dns/
[integration-services]: https://azure.microsoft.com/product-categories/integration/
[logic-apps]: /azure/logic-apps/logic-apps-overview
[logic-apps-connectors]: /azure/connectors/apis-list
[logic-apps-log-analytics]: /azure/logic-apps/logic-apps-monitor-your-logic-apps-oms
[logic-apps-monitor]: /azure/logic-apps/logic-apps-monitor-your-logic-apps
[logic-apps-restrict-ip]: /azure/logic-apps/logic-apps-securing-a-logic-app#restrict-incoming-ip-addresses
[logic-apps-secure]: /azure/logic-apps/logic-apps-securing-a-logic-app#secure-parameters-and-inputs-within-a-workflow
[logic-apps-sla]: https://azure.microsoft.com/support/legal/sla/logic-apps
[monitor]: /azure/azure-monitor/overview
[rbac]: /azure/role-based-access-control/overview
[rto]: ../../resiliency/index.md#rto-and-rpo