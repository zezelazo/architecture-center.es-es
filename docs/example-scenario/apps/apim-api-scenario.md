---
title: Migración de una aplicación web heredada a una arquitectura basada en API en Azure
description: Use Azure API Management para modernizar una aplicación web heredada.
author: begim
ms.date: 09/13/2018
ms.openlocfilehash: 1aa7ea6dc895146e13677dd9867fb2530f0a8f04
ms.sourcegitcommit: 62945777e519d650159f0f963a2489b6bb6ce094
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/09/2018
ms.locfileid: "48876805"
---
# <a name="migrating-a-legacy-web-application-to-an-api-based-architecture-on-azure"></a>Migración de una aplicación web heredada a una arquitectura basada en API en Azure

Una empresa de comercio electrónico del sector del turismo va a modernizar su pila de software heredado basado en explorador. Aunque la pila existente es principalmente monolítica, existen algunos [servicios HTTP basados en SOAP][soap] desde un proyecto reciente. Se están planteando la creación de flujos de ingresos adicionales para rentabilizar parte de la propiedad intelectual interna que han desarrollado.

Los objetivos del proyecto incluyen la solución de la deuda técnica, la mejora del mantenimiento continuo y la aceleración del desarrollo de características con menos errores de regresión. El proyecto usará un proceso iterativo para evitar el riesgo, con algunos pasos que se realizan en paralelo:

* El equipo de desarrollo modernizará el back-end de la aplicación, que se compone de bases de datos relacionales hospedadas en máquinas virtuales.
* El equipo de desarrollo de la empresa escribirá nuevas funcionalidades de negocio, que se expondrán mediante nuevas API HTTP.
* Un equipo de desarrollo subcontratado creará una nueva interfaz de usuario basada en explorador, que se hospedará en Azure.

Las nuevas características de la aplicación se entregará en fases. Se realizará una *sustitución gradual* de la funcionalidad de la interfaz de usuario cliente-servidor basada en explorador existente (hospedada en el entorno local) que impulsa su negocio de comercio electrónico actualmente.

El equipo de administración no está interesado en modernizar innecesariamente. También desean mantener el control del ámbito y los costos. Para ello, se ha decidido mantener los servicios HTTP SOAP existentes. También pretenden minimizar los cambios en la interfaz de usuario existente. [Azure API Management (APIM)][apim] se puede utilizar para solucionar muchos de los requisitos y restricciones del proyecto.

## <a name="architecture"></a>Arquitectura

![Diagrama de la arquitectura][architecture]

La nueva interfaz de usuario se hospedará como una aplicación de plataforma como servicio (PaaS) en Azure y dependerá de las API HTTP nuevas y existentes. Estas API se entregarán con un conjunto de interfaces mejor diseñado que permita un mejor rendimiento y facilite la integración y la extensibilidad futura.

### <a name="components-and-security"></a>Componentes y seguridad

1. La aplicación web local existente continuará con el consumo directo de los servicios web locales existentes.
2. Las llamadas desde la aplicación web existente a los servicios HTTP existentes permanecerán sin cambios. Estas llamadas son internas en la red corporativa.
3. Las llamadas entrantes se realizan desde Azure a los servicios internos existentes:
    * El equipo de seguridad permite que el tráfico desde la instancia de APIM pase a través del firewall corporativo hacia los servicios locales existentes [con transporte seguro (HTTPS/SSL)][apim-ssl].
    * El equipo de operaciones solo permitirá las llamadas entrantes a los servicios desde la instancia de APIM. Este requisito se cumple mediante la [inclusión en la lista de permitidos de la dirección IP de la instancia de APIM][apim-whitelist-ip] dentro del perímetro de la red corporativa.
    * Se configura un nuevo módulo en la canalización de solicitudes de servicios HTTP local (para actuar **solo** en las conexiones que se originan externamente), la cual validará [un certificado que proporcionará la instancia de APIM][apim-mutualcert-auth].
1. La nueva API:
    * Se expone solo mediante la instancia de APIM, que proporcionará la fachada de la API. No se podrá acceder a la nueva API directamente.
    * Se ha desarrollado y publicado como una [aplicación de API web de PaaS de Azure][azure-api-apps].
    * Se incluye en la lista de permitidos (mediante la [configuración de la aplicación web][azure-appservice-ip-restrict]) para aceptar solo la dirección [VIP de APIM][apim-faq-vip].
    * Se hospeda en Azure Web Apps con el transporte seguro SSL activado.
    * Tiene habilitada la autorización, [proporcionada por Azure App Service][azure-appservice-auth] con Azure Active Directory y OAuth2.
2. La nueva aplicación web basada en explorador dependerá de la instancia de Azure API Management para **ambas** API: la API HTTP existente y la nueva API.

La instancia de APIM se configurará para asignar los servicios HTTP heredados a un nuevo contrato de API. Al hacerlo, la nueva interfaz de usuario web no es consciente de la integración con un conjunto de servicios y API heredadas y nuevas API. En el futuro, el equipo del proyecto migrará gradualmente la funcionalidad a las nuevas API y retirará los servicios originales. Estos cambios se controlarán dentro de la configuración de APIM, lo que no afecta a la interfaz de usuario del front-end y evita el trabajo de renovación.

### <a name="alternatives"></a>Alternativas

* Si la organización planea mover la infraestructura completamente a Azure, incluidas las máquinas virtuales que hospedan las aplicaciones heredadas, APIM todavía sería una buena opción, ya que puede actuar como una fachada para cualquier punto de conexión HTTP direccionable.
* Si el cliente ha decidido mantener la privacidad de los puntos de conexión existentes y no los expone públicamente, su instancia de API Management podría vincularse a una instancia de [Azure Virtual Network (VNet)][azure-vnet]:
  * En un [escenario de migración mediante lift-and-shift de Azure][azure-vm-lift-shift] vinculado a la red virtual de Azure implementada, el cliente podría direccionar directamente el servicio de back-end con direcciones IP privadas.
  * En el escenario local, la instancia de API Management llegara al servicio interno de forma privada mediante [Azure VPN Gateway y conexiones VPN de IPSec de sitio a sitio][azure-vpn] o [ ExpressRoute][azure-er], por lo que es un [escenario híbrido de Azure y entorno local][azure-hybrid].
* La instancia de API Management se puede mantener privada mediante la implementación de la instancia de API Management en modo interno. A continuación, se podría utilizar la implementación con [Azure Application Gateway][azure-appgw] para permitir el acceso público para algunas API mientras que otras siguen siendo internas. Para más información, consulte [Conexión de APIM en modo interno a una red virtual][apim-vnet-internal].

> [!NOTE]
> Para obtener información general sobre la conexión de API Management a una red virtual, [consulte aquí][apim-vnet].

### <a name="availability-and-scalability"></a>Disponibilidad y escalabilidad

* Azure API Management se puede [escalar horizontalmente][apim-scaleout] mediante la elección de un plan de tarifa y la posterior adición de unidades.
* El escalado también puede ocurrir [automáticamente con el escalado automático][apim-autoscale].
* La [implementación en varias regiones][apim-multi-regions] permitirá opciones de conmutación por error y se puede realizar en el [plan Premium][apim-pricing].
* Considere la posibilidad de la [integración con Azure Application Insights][azure-apim-ai], que también expone las métricas mediante [Azure Monitor][azure-mon] para la supervisión.

## <a name="deployment"></a>Implementación

Para empezar a trabajar, [cree una instancia de Azure API Management en el portal.][apim-create]

Como alternativa, puede elegir una [plantilla de inicio rápido][azure-quickstart-templates-apim] de Azure Resource Manager existente que se adapte a sus necesidades específicas.

## <a name="pricing"></a>Precios

API Management se ofrece en cuatro niveles: desarrollador, básico, estándar y premium. Puede encontrar instrucciones detalladas sobre las diferencias entre estos niveles en la [guía de precios de Azure API Management.][apim-pricing]

Los clientes pueden escalar API Management agregando o quitando unidades. Cada unidad tiene una capacidad que depende de su nivel.

> [!NOTE]
> El nivel Desarrollador se puede usar para la evaluación de las características de API Management. El nivel Desarrollador no se debe usar en producción.

Para ver los costos proyectados y personalizar según las necesidades de la implementación, puede modificar el número de unidades de escalado y las instancias de App Service en la [Calculadora de precios de Azure][pricing-calculator].

## <a name="related-resources"></a>Recursos relacionados

Consulte la amplia [documentación y artículos de referencia][apim] de Azure API Management.

<!-- links -->
[architecture]: ./media/architecture-apim-api-scenario.png
[apim-create]: /azure/api-management/get-started-create-service-instance
[apim-git]: /azure/api-management/api-management-configuration-repository-git
[apim-multi-regions]: /azure/api-management/api-management-howto-deploy-multi-region
[apim-autoscale]: /azure/api-management/api-management-howto-autoscale
[apim-scaleout]: /azure/api-management/upgrade-and-scale
[azure-apim-ai]: /azure/api-management/api-management-howto-app-insights
[azure-ai]: /azure/application-insights/
[azure-mon]: /azure/monitoring-and-diagnostics/monitoring-overview
[azure-appgw]: /azure/application-gateway/application-gateway-introduction
[apim-vnet-internal]: /azure/api-management/api-management-howto-integrate-internal-vnet-appgateway
[apim-vnet]: /azure/api-management/api-management-using-with-vnet
[azure-hybrid]: /azure/architecture/reference-architectures/hybrid-networking/
[azure-er]: /azure/expressroute/expressroute-introduction
[azure-vpn]: /azure/vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal
[azure-vnet]: /azure/virtual-network/virtual-networks-overview
[azure-appservice-auth]: /azure/app-service/app-service-authentication-overview#identity-providers
[apim-faq-vip]: /azure/api-management/api-management-faq#is-the-api-management-gateway-ip-address-constant-can-i-use-it-in-firewall-rules
[azure-appservice-ip-restrict]: /azure/app-service/app-service-ip-restrictions
[azure-api-apps]: /azure/app-service/
[apim-ssl]: /azure/api-management/api-management-howto-manage-protocols-ciphers
[apim-mutualcert-auth]: /azure/api-management/api-management-howto-mutual-certificates
[apim-whitelist-ip]: /azure/api-management/api-management-faq#is-the-api-management-gateway-ip-address-constant-can-i-use-it-in-firewall-rules
[anti-corruption-layer-pattern]: /azure/architecture/patterns/anti-corruption-layer
[apim]: /azure/api-management/api-management-key-concepts
[apim-api-design-guidance]: /azure/architecture/best-practices/api-design
[visualstudio-youtube-solid-design]: https://youtu.be/agkWYPUcLpg
[azure-vm-lift-shift]: https://azure.microsoft.com/resources/azure-virtual-datacenter-lift-and-shift-guide/
[standard-pricing-calc]: https://azure.com/e/
[premium-pricing-calc]: https://azure.com/e/
[apim-pricing]: https://azure.microsoft.com/pricing/details/api-management/
[azure-quickstart-templates-apim]: https://azure.microsoft.com/resources/templates/?term=API+Management&pageNumber=1
[soap]: https://en.wikipedia.org/wiki/SOAP
[pricing-calculator]: https://azure.com/e/0e916a861fac464db61342d378cc0bd6
