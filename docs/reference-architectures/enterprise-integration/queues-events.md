---
title: Integración empresarial con colas de mensajes y eventos
titleSuffix: Azure Reference Architectures
description: Arquitectura recomendada para la implementación de un patrón de integración empresarial con Azure Logic Apps, Azure API Management, Azure Service Bus y Azure Event Grid.
author: mattfarm
ms.reviewer: jonfan, estfan, LADocs
ms.topic: article
ms.date: 12/03/2018
ms.custom: integration-services
ms.openlocfilehash: 6357cb5015c8f10c0f4a8aa1b310ddbb38367004
ms.sourcegitcommit: a0a9981e7586bed8d876a54e055dea1e392118f8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/11/2018
ms.locfileid: "53233870"
---
# <a name="enterprise-integration-on-azure-using-message-queues-and-events"></a>Integración empresarial en Azure mediante colas de mensajes y eventos

Esta arquitectura de referencia integra sistemas de back-end empresariales y usa las colas de mensajes y los eventos para desacoplar los servicios, con el fin de aumentar la escalabilidad y la confiabilidad. Los sistemas de back-end pueden incluir sistemas de software como servicio (SaaS), servicios de Azure y servicios web existentes de su empresa.

![Arquitectura de referencia para la integración empresarial con colas y eventos](./_images/enterprise-integration-queues-events.png)

## <a name="architecture"></a>Arquitectura

La arquitectura que se muestra aquí se basa en una arquitectura más sencilla que se muestra en el artículo [Integración empresarial básica en Azure][basic-enterprise-integration]. Dicha arquitectura usa [Logic Apps][logic-apps] para organizar los flujos de trabajo y [API Management][apim] para crear catálogos de API.

Esta versión de la arquitectura agrega dos componentes que ayudan a que el sistema sea más confiable y escalable:

- **[Azure Service Bus][service-bus]**. Service Bus es un agente de mensajes seguro y confiable.

- **[Azure Event Grid][event-grid]**. Event Grid es un servicio de enrutamiento de eventos. Usa un modelo de generación de eventos de [publicación/suscripción](../../patterns/publisher-subscriber.md) (pub/sub).

La comunicación asincrónica mediante un agente de mensajes proporciona una serie de ventajas con respecto a la realización de llamadas directas, sincrónicas a servicios de back-end:

- Proporciona la nivelación de carga para controlar los picos de cargas de trabajo, mediante el [patrón de nivelación de carga basado en cola](../../patterns/queue-based-load-leveling.md).
- Realiza un seguimiento confiable del progreso de los flujos de trabajo de ejecución prolongada que implican varios pasos o varias aplicaciones.
- Ayuda a desacoplar aplicaciones.
- Se integra con los sistemas existentes basados en mensajes.
- Permite poner en cola el trabajo cuando algún sistema de back-end no está disponible.

Event Grid permite a los distintos componentes en el sistema reaccionar ante eventos cuando estos se produzcan, en lugar de confiar en sondeos o tareas programadas. Igual que hace una cola de mensajes, le ayuda a desacoplar aplicaciones y servicios. Una aplicación o un servicio puede publicar eventos y todos los suscriptores interesados recibirán notificaciones. Se pueden agregar suscriptores sin actualizar al remitente.

Muchos servicios de Azure admiten el envío de eventos a Event Grid. Por ejemplo, una aplicación lógica puede escuchar un evento cuando se agregan nuevos archivos a un almacén de blobs. Este patrón permite los flujos de trabajo reactivos, donde cargar un archivo o poner un mensaje en una cola inicia una serie de procesos. Los procesos se pueden ejecutar en paralelo o en una secuencia concreta.

## <a name="recommendations"></a>Recomendaciones

Las recomendaciones que se describen en el artículo [Integración empresarial básica en Azure][basic-enterprise-integration] se aplican a esta arquitectura. También se aplican las siguientes recomendaciones:

### <a name="service-bus"></a>Azure Service Bus

Service Bus tiene dos modos de entrega, *extracción* o *inserción*. En el modelo de extracción, el receptor realiza sondeos de continuamente para detectar mensajes nuevos. El sondeo puede ser ineficaz, sobre todo si hay varias colas y cada una de ellas recibe unos pocos mensajes, o si pasa mucho tiempo entre los mensajes. En el modelo de inserción, Service Bus envía un evento a través de Event Grid cuando hay mensajes nuevos. El receptor se suscribe al evento. Cuando se desencadena el evento, el receptor extrae el siguiente lote de mensajes de Service Bus.

Cuando se crea una aplicación lógica para consumir mensajes de Service Bus, se recomienda usar el modelo de inserción con la integración de Event Grid. A menudo resulta más económico, ya que la aplicación lógica no necesita sondear Service Bus. Para más información, consulte [Introducción a la integración de Azure Service Bus en Event Grid](/azure/service-bus-messaging/service-bus-to-event-grid-integration-concept). En la actualidad, se requiere el [nivel Premium](https://azure.microsoft.com/pricing/details/service-bus/) de Service Bus para las notificaciones de Event Grid.

Use [PeekLock](/azure/service-bus-messaging/service-bus-messaging-overview#queues) para acceder a un grupo de mensajes. Cuando usa PeekLock, la aplicación lógica puede realizar los pasos necesarios para validar cada mensaje antes de completar o abandonar el mensaje. Este enfoque protege contra la pérdida accidental de datos.

### <a name="event-grid"></a>Event Grid

Cuando se activa un desencadenador de Event Grid, significa que se ha producido *al menos un* evento. Por ejemplo, cuando una aplicación lógica recibe un desencadenador de Event Grid en un mensaje de Service Bus, debe asumir que varios mensajes puedan estar disponibles para procesarlos.

Event Grid usa un modelo sin servidor. La facturación se calcula en función del número de operaciones (ejecuciones de eventos). Para más información, consulte [Precios de Event Grid](https://azure.microsoft.com/pricing/details/event-grid/). Actualmente no existen consideraciones sobre los niveles en Event Grid.

## <a name="scalability-considerations"></a>Consideraciones sobre escalabilidad

El nivel Premium de Service Bus puede escalar horizontalmente el número de unidades de mensajería para alcanzar una mayor escalabilidad. Las configuraciones de nivel Premium pueden tener una, dos o cuatro unidades de mensajería. Para más información sobre el escalado de Service Bus, consulte [Procedimientos recomendados para mejorar el rendimiento mediante la mensajería de Service Bus](/azure/service-bus-messaging/service-bus-performance-improvements).

## <a name="availability-considerations"></a>Consideraciones sobre disponibilidad

Revise el SLA de cada servicio:

- [SLA de API Management][apim-sla]
- [Acuerdo de Nivel de Servicio de Event Grid][event-grid-sla]
- [SLA de Logic Apps][logic-apps-sla]
- [Acuerdo de Nivel de Servicio de Service Bus][sb-sla]

Considere la posibilidad de implementar la recuperación ante desastres geográfica en el nivel Premium de Service Bus para habilitar la conmutación por error si se produce una interrupción grave. Para más información, consulte [Recuperación ante desastres geográfica de Azure Service Bus](/azure/service-bus-messaging/service-bus-geo-dr).

## <a name="security-considerations"></a>Consideraciones sobre la seguridad

Proteja Service Bus con una firma de acceso compartido (SAS). Puede usar la [autenticación de SAS](/azure/service-bus-messaging/service-bus-sas) para conceder acceso a un usuario a los recursos de Service Bus con derechos específicos. Para más información, consulte [Autenticación y autorización de Service Bus](/azure/service-bus-messaging/service-bus-authentication-and-authorization).

Si necesita exponer una cola de Service Bus como un punto de conexión de HTTP para, por ejemplo, publicar nuevos mensajes, use API Management para proteger la cola por delante del punto de conexión. El punto de conexión se puede proteger con certificados o con autenticación de OAuth, según corresponda. La manera más sencilla de proteger un punto de conexión es mediante una aplicación lógica con un desencadenador HTTP de solicitud o respuesta como intermediario.

El servicio Event Grid protege la entrega de eventos con un código de validación. Si usa Logic Apps para consumir el evento, la validación se realiza automáticamente. Para más información, vea [Event Grid security and authentication](/azure/event-grid/security-authentication) (Seguridad y autenticación de Event Grid).

[apim]: /azure/api-management
[apim-sla]: https://azure.microsoft.com/support/legal/sla/api-management/
[event-grid]: /azure/event-grid/
[event-grid-sla]: https://azure.microsoft.com/support/legal/sla/event-grid
[logic-apps]: /azure/logic-apps/logic-apps-overview
[logic-apps-sla]: https://azure.microsoft.com/support/legal/sla/logic-apps
[sb-sla]: https://azure.microsoft.com/support/legal/sla/service-bus/
[service-bus]: /azure/service-bus-messaging/
[basic-enterprise-integration]: ./basic-enterprise-integration.md
