---
title: Bot conversacional para reservas de hotel en Azure
description: Escenario probado para la creación de un bot de chat de conversación para aplicaciones de comercio con Azure Bot Service, Cognitive Services y LUIS, Azure SQL Database y Application Insights.
author: iainfoulds
ms.date: 07/05/2018
ms.openlocfilehash: b664faf20d806824c2581346aaa592b0d74207da
ms.sourcegitcommit: 71cbef121c40ef36e2d6e3a088cb85c4260599b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/14/2018
ms.locfileid: "39060870"
---
# <a name="conversational-chatbot-for-hotel-reservations-on-azure"></a>Bot conversacional para reservas de hotel en Azure

Este escenario de ejemplo es aplicable a empresas que necesitan integrar un bot de chat de conversación en las aplicaciones. En este escenario, se usa un bot de chat de C# para una cadena de hoteles que permite a los clientes comprobar la disponibilidad y reservar alojamiento mediante una aplicación web o móvil.

Algunos escenarios de ejemplo son proporcionar a los clientes una manera de ver la disponibilidad del hotel y reservar habitaciones, consultar el menú del restaurante para llevar y realizar un pedido de comida, o buscar y pedir copias impresas de fotografías. Tradicionalmente, las empresas tendrían que contratar y enseñar a los agentes de servicio al cliente para responder a estas solicitudes de los clientes, y los clientes tendrían que esperar a que hubiera un representante disponible para obtener asistencia.

Con los servicios de Azure como Bot Service, LUIS o Speech API, las compañías pueden ayudar a los clientes y procesar pedidos o reservas con bots automatizados y escalables.

## <a name="related-use-cases"></a>Casos de uso relacionados

Tenga en cuenta este escenario para los casos de uso siguientes:

* Ver el menú del restaurante para llevar y pedir comida
* Comprobar la disponibilidad del hotel y reservar una habitación
* Buscar las fotografías disponibles y solicitar copias impresas

## <a name="architecture"></a>Arquitectura

![Introducción a la arquitectura de los componentes implicados en un bot de chat conversacional de Azure][architecture]

Este escenario incluye un bot de conversación que funciona como un conserje de hotel. Los datos fluyen por el escenario de la siguiente manera:

1. El cliente accede al bot de chat mediante una aplicación web o móvil.
2. El usuario se autentica con Azure Active Directory B2C.
3. El usuario interactúa con Azure Bot Service y solicita información sobre la disponibilidad del hotel.
4. Cognitive Services procesa la solicitud en lenguaje natural para comprender la comunicación con el cliente.
5. Cuando el usuario está satisfecho con el resultado, el bot agrega o actualiza la reserva del cliente en una base de datos de SQL Database.
6. Application Insights recopila los datos de telemetría en tiempo de ejecución durante todo el proceso para ayudar al equipo de DevOps con el uso y el rendimiento del bot.

### <a name="components"></a>Componentes

* [Azure Active Directory][aad-docs] es el directorio multiinquilino basado en la nube y el servicio de administración de identidades de Microsoft. Azure AD admite un conector B2C, lo que permite identificar los individuos mediante identificadores externos, como Google, Facebook o cuenta Microsoft.
* [App Service][appservice-docs] le permite crear y hospedar aplicaciones web en el lenguaje de programación que prefiera sin tener que administrar la infraestructura.
* [Bot Service][botservice-docs] proporciona herramientas para crear, probar, implementar y administrar bots inteligentes.
* [Cognitive Services][cognitive-docs] permite usar algoritmos inteligentes para ver, oír, hablar, comprender e interpretar las necesidades de los usuarios con formas de comunicación naturales.
* [SQL Database][sqldatabase-docs] es un servicio de base de datos relacional en la nube totalmente administrado que proporciona compatibilidad con el motor de SQL Server.
* [Application Insights][appinsights-docs] es un servicio de Application Performance Management (APM) extensible que permite supervisar el rendimiento de las aplicaciones, como su bot de chat.

### <a name="alternatives"></a>Alternativas

* [Microsoft Speech API][speech-api] puede usarse para cambiar la forma en que los clientes se comunican con el bot.
* [QnA Maker][qna-maker] puede utilizarse para agregar rápidamente conocimiento al bot a partir de contenido semiestructurado, por ejemplo, preguntas frecuentes.
* [Translator Text][translator] es un servicio que permite agregar al bot compatibilidad con varios idiomas fácilmente.

## <a name="considerations"></a>Consideraciones

### <a name="availability"></a>Disponibilidad

Este escenario utiliza Azure SQL Database para almacenar las reservas de los clientes. SQL Database incluye bases de datos con redundancia de zona, grupos de conmutación por error y replicación geográfica. Para más información, consulte [Funcionalidades de disponibilidad de Azure SQL Database][sqlavailability-docs].

Para ver otros temas de disponibilidad, consulte la [lista de comprobación de disponibilidad][availability] que encontrará en Azure Architecture Center.

### <a name="scalability"></a>Escalabilidad

Este escenario utiliza Azure App Service. Con App Service, puede escalar automáticamente el número de instancias que ejecutan su bot. Esta funcionalidad le permite satisfacer la demanda de los clientes para su aplicación web y bot de chat. Para más información sobre el escalado automático, consulte [Procedimientos recomendados de escalado automático][autoscaling] en el centro de arquitectura.

Para ver otros temas de escalabilidad, consulte la [lista de comprobación de escalabilidad][scalability] que encontrará en el centro de arquitectura de Azure.

### <a name="security"></a>Seguridad

Este escenario usa Azure Active Directory B2C para autenticar los usuarios. Con AAD B2C, su bot de chat de no almacena ninguna información de cuenta confidencial de sus clientes ni las credenciales. Para más información, consulte [Introducción a Azure Active Directory B2C][aadb2c-docs].

La información almacenada en Azure SQL Database se cifra en reposo con cifrado de datos transparente (TDE). SQL Database también ofrece Always Encrypted, que cifra los datos durante las operaciones de consulta y procesamiento. Para más información sobre la seguridad de SQL Database, consulte [Cumplimiento y seguridad de Azure SQL Database][sqlsecurity-docs].

Para obtener instrucciones generales sobre el diseño de soluciones seguras, consulte la [documentación de seguridad de Azure][security].

### <a name="resiliency"></a>Resistencia

Este escenario utiliza Azure SQL Database para almacenar las reservas de los clientes. SQL Database incluye bases de datos con redundancia de zona, grupos de conmutación por error, replicación geográfica y copias de seguridad automáticas. Estas características permiten que la aplicación continúe ejecutándose en caso de un evento de mantenimiento o una interrupción. Para más información, consulte [Funcionalidades de disponibilidad de Azure SQL Database][sqlavailability-docs].

Para supervisar el estado de mantenimiento de la aplicación, este escenario usa Application Insights. Con Application Insights, puede generar alertas y responder a problemas de rendimiento que pueden afectar a la experiencia del cliente y la disponibilidad del bot de chat. Para más información, consulte [¿Qué es Application Insights?][appinsights-docs]

Para obtener instrucciones generales sobre el diseño de soluciones resistentes, consulte [Diseño de aplicaciones resistentes de Azure][resiliency].

## <a name="deploy-the-scenario"></a>Implementación del escenario

Este escenario se divide en tres componentes para explorar las áreas que más le interesen:

* [Componentes de infraestructura](#deploy-infrastructure-components). Use una plantilla de Azure Resource Manager para implementar los componentes fundamentales de la infraestructura de un servicio de aplicaciones, una aplicación web, del servicio Application Insights, de una cuenta de almacenamiento y de SQL Server y una base de datos.
* [Bot de chat de una aplicación web](#deploy-web-app-chatbot). Use la CLI de Azure para implementar un bot con Bot Service y una aplicación de Language Understanding Intelligent Services (LUIS).
* [Ejemplo de aplicación de bot de chat de C#](#deploy-chatbot-c-application-code). Use Visual Studio para revisar el código de ejemplo de una aplicación de reservas de hotel en C#, e implementar un bot en Azure.

**Requisitos previos** Debe tener una cuenta de Azure. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

### <a name="deploy-infrastructure-components"></a>Implementación de los componentes de infraestructura

Para implementar los componentes de infraestructura con una plantilla de Azure Resource Manager, realice los pasos siguientes.

1. Haga clic en botón **Implementar en Azure**.<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fcommerce-chatbot.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. Espere a que la implementación de plantilla se abra en Azure Portal y complete los pasos siguientes:
   * Haga clic en **Crear nuevo** para el grupo de recursos y proporcione un nombre, por ejemplo, *myCommerceChatBotInfrastructure*.
   * Seleccione una región en el cuadro de lista desplegable **Ubicación**.
   * Proporcione un nombre de usuario y una contraseña segura para la cuenta de administrador de SQL Server.
   * Revise los términos y condiciones, y seleccione **Acepto los términos y condiciones indicados anteriormente**.
   * Seleccione el botón **Comprar**.

La implementación tarda unos minutos en finalizar.

### <a name="deploy-web-app-chatbot"></a>Implementación de una aplicación web de bot de chat

Para crear el bot de chat, use la CLI de Azure. En el ejemplo siguiente se instala la extensión de la CLI para Bot Service, se crea un grupo de recursos y se implementa un bot que usa Application Insights. Cuando se le solicite, autentique la cuenta Microsoft y permita al bot que se registre en Bot Service y la aplicación de Language Understanding Intelligent Services (LUIS).

```azurecli-interactive
# Install the Azure CLI extension for the Bot Service
az extension add --name botservice --yes

# Create a resource group
az group create --name myCommerceChatbot --location eastus

# Create a Web App Chatbot that uses Application Insights
az bot create \
    --resource-group myCommerceChatbot \
    --name commerceChatbot \
    --location eastus \
    --kind webapp \
    --sku S1 \
    --insights eastus
```

### <a name="deploy-chatbot-c-application-code"></a>Implementación del código de la aplicación de bot de chat de C#

En GitHub hay disponible una aplicación de ejemplo en C#: 

* [Ejemplo de bot comercial en C#](https://github.com/Microsoft/AzureBotServices-scenarios/tree/master/CSharp/Commerce/src)

La aplicación de ejemplo incluye los componentes de autenticación de Azure Active Directory y la integración con el componente Language Understanding Intelligent Services (LUIS) de Cognitive Services. La aplicación requiere Visual Studio para compilar e implementar el escenario. En la documentación del repositorio de GitHub encontrará información adicional sobre cómo configurar AAD B2C y la aplicación de LUIS.

## <a name="pricing"></a>Precios

Para explorar el costo de ejecutar este escenario, todos los servicios están preconfigurados en la calculadora de costos. Para ver cómo cambiarían los precios en su caso concreto, cambie las variables pertinentes para que coincidan con el tráfico esperado.

Hemos proporcionado tres ejemplos de perfiles de costo según la cantidad de mensajes que se espera que su bot de chat procese:

* [Pequeño][small-pricing]: se corresponde con un procesamiento de menos de 10 000 mensajes al mes.
* [Mediano][medium-pricing]: se corresponde con un procesamiento de menos de 500 000 mensajes al mes.
* [Grande][large-pricing]: se corresponde con un procesamiento de menos de 10 millones de mensajes al mes.

## <a name="related-resources"></a>Recursos relacionados

Para obtener un conjunto de tutoriales guiados sobre cómo sacar partido de Azure Bot Service, consulte el [nodo de tutoriales][botservice-docs] de la documentación.

<!-- links -->
[aadb2c-docs]: /azure/active-directory-b2c/active-directory-b2c-overview
[aad-docs]: /azure/active-directory/
[appinsights-docs]: /azure/application-insights/app-insights-overview
[appservice-docs]: /azure/app-service/
[architecture]: ./media/commerce-chatbot/architecture-commerce-chatbot.png
[autoscaling]: ../../best-practices/auto-scaling.md
[availability]: ../../checklist/availability.md
[botservice-docs]: /azure/bot-service/
[cognitive-docs]: /azure/cognitive-services/
[resiliency]: ../../resiliency/index.md
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[security]: /azure/security/
[scalability]: ../../checklist/scalability.md
[sqlavailability-docs]: /azure/sql-database/sql-database-technical-overview#availability-capabilities
[sqldatabase-docs]: /azure/sql-database/
[sqlsecurity-docs]: /azure/sql-database/sql-database-technical-overview#advanced-security-and-compliance
[qna-maker]: /azure/cognitive-services/QnAMaker/Overview/overview
[speech-api]: /azure/cognitive-services/speech/home
[translator]: /azure/cognitive-services/translator/translator-info-overview

[small-pricing]: https://azure.com/e/dce05b6184904c50b38e1a8654f726b6
[medium-pricing]: https://azure.com/e/304d17106afc480dbc414f9726078a03
[large-pricing]: https://azure.com/e/8319dd5e5e3d4f118f9029e32a80e887