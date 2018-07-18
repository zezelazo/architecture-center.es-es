---
title: Bot de chat conversacional de Azure para reservas de hotel
description: Solución probada para la creación de un bot de chat de conversación para aplicaciones de comercio con Azure Bot Service, Cognitive Services y LUIS, Azure SQL Database y Application Insights.
author: iainfoulds
ms.date: 07/05/2018
ms.openlocfilehash: 85bdc3194961bbbd8d89db34e5c56e4baa8d8599
ms.sourcegitcommit: 5d99b195388b7cabba383c49a81390ac48f86e8a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/06/2018
ms.locfileid: "37891385"
---
# <a name="conversational-azure-chatbot-for-hotel-reservations"></a><span data-ttu-id="255f5-103">Bot de chat conversacional de Azure para reservas de hotel</span><span class="sxs-lookup"><span data-stu-id="255f5-103">Conversational Azure chatbot for hotel reservations</span></span>

<span data-ttu-id="255f5-104">Este escenario de ejemplo es aplicable a empresas que necesitan integrar un bot de chat de conversación en las aplicaciones.</span><span class="sxs-lookup"><span data-stu-id="255f5-104">This example scenario is applicable to businesses that need integrate a conversational chatbot into applications.</span></span> <span data-ttu-id="255f5-105">En esta solución, se usa un bot de chat de C# para una cadena de hoteles que permite a los clientes comprobar la disponibilidad y reservar alojamiento mediante una aplicación web o móvil.</span><span class="sxs-lookup"><span data-stu-id="255f5-105">In this solution, a C# chatbot is used for a hotel chain that allows customers to check availability and book accommodation through a web or mobile application.</span></span>

<span data-ttu-id="255f5-106">Algunos escenarios de ejemplo son proporcionar a los clientes una manera de ver la disponibilidad del hotel y reservar habitaciones, consultar el menú del restaurante para llevar y realizar un pedido de comida, o buscar y pedir copias impresas de fotografías.</span><span class="sxs-lookup"><span data-stu-id="255f5-106">Example scenarios include providing a way for customers to view hotel availability and book rooms, review a restaurant take-out menu and place a food order, or search for and order prints of photographs.</span></span> <span data-ttu-id="255f5-107">Tradicionalmente, las empresas tendrían que contratar y enseñar a los agentes de servicio al cliente para responder a estas solicitudes de los clientes, y los clientes tendrían que esperar a que hubiera un representante disponible para obtener asistencia.</span><span class="sxs-lookup"><span data-stu-id="255f5-107">Traditionally, businesses would need to hire and train customer service agents to respond to these customer requests, and customers would have to wait until a representative is available to provide assistance.</span></span>

<span data-ttu-id="255f5-108">Con los servicios de Azure como Bot Service, LUIS o Speech API, las compañías pueden ayudar a los clientes y procesar pedidos o reservas con bots automatizados y escalables.</span><span class="sxs-lookup"><span data-stu-id="255f5-108">By using Azure services such as the Bot Service and Language Understanding or Speech API services, companies can assist customers and process orders or reservations with automated, scalable bots.</span></span>

## <a name="potential-use-cases"></a><span data-ttu-id="255f5-109">Posibles casos de uso</span><span class="sxs-lookup"><span data-stu-id="255f5-109">Potential use cases</span></span>

<span data-ttu-id="255f5-110">Tenga en cuenta esta solución para los casos de uso siguientes:</span><span class="sxs-lookup"><span data-stu-id="255f5-110">Consider this solution for the following use cases:</span></span>

* <span data-ttu-id="255f5-111">Ver el menú del restaurante para llevar y pedir comida</span><span class="sxs-lookup"><span data-stu-id="255f5-111">View restaurant take-out menu and order food</span></span>
* <span data-ttu-id="255f5-112">Comprobar la disponibilidad del hotel y reservar una habitación</span><span class="sxs-lookup"><span data-stu-id="255f5-112">Check hotel availability and reserve a room</span></span>
* <span data-ttu-id="255f5-113">Buscar las fotografías disponibles y solicitar copias impresas</span><span class="sxs-lookup"><span data-stu-id="255f5-113">Search available photos and order prints</span></span>

## <a name="architecture"></a><span data-ttu-id="255f5-114">Arquitectura</span><span class="sxs-lookup"><span data-stu-id="255f5-114">Architecture</span></span>

![Introducción a la arquitectura de los componentes implicados en un bot de chat conversacional de Azure][architecture]

<span data-ttu-id="255f5-116">Esta solución incluye un bot de conversación que funciona como un conserje de hotel.</span><span class="sxs-lookup"><span data-stu-id="255f5-116">This solution covers a conversational bot that functions as a concierge for a hotel.</span></span> <span data-ttu-id="255f5-117">Los datos fluyen por la solución de la siguiente manera:</span><span class="sxs-lookup"><span data-stu-id="255f5-117">The data flows through the solution as follows:</span></span>

1. <span data-ttu-id="255f5-118">El cliente accede al bot de chat mediante una aplicación web o móvil.</span><span class="sxs-lookup"><span data-stu-id="255f5-118">The customer accesses the chatbot with a mobile or web app.</span></span>
2. <span data-ttu-id="255f5-119">El usuario se autentica con Azure Active Directory B2C.</span><span class="sxs-lookup"><span data-stu-id="255f5-119">Using Azure Active Directory B2C (Business 2 Customer), the user is authenticated.</span></span>
3. <span data-ttu-id="255f5-120">El usuario interactúa con Azure Bot Service y solicita información sobre la disponibilidad del hotel.</span><span class="sxs-lookup"><span data-stu-id="255f5-120">Interacting with the Bot Service, the user requests information about hotel availability.</span></span>
4. <span data-ttu-id="255f5-121">Cognitive Services procesa la solicitud en lenguaje natural para comprender la comunicación con el cliente.</span><span class="sxs-lookup"><span data-stu-id="255f5-121">Cognitive Services processes the natural language request to understand the customer communication.</span></span>
5. <span data-ttu-id="255f5-122">Cuando el usuario está satisfecho con el resultado, el bot agrega o actualiza la reserva del cliente en una base de datos de SQL Database.</span><span class="sxs-lookup"><span data-stu-id="255f5-122">After the user is happy with the results, the bot adds or updates the customer’s reservation in a SQL Database.</span></span>
6. <span data-ttu-id="255f5-123">Application Insights recopila los datos de telemetría en tiempo de ejecución durante todo el proceso para ayudar al equipo de DevOps con el uso y el rendimiento del bot.</span><span class="sxs-lookup"><span data-stu-id="255f5-123">Application Insights gathers runtime telemetry throughout the process to help the DevOps team with bot performance and usage.</span></span>

### <a name="components"></a><span data-ttu-id="255f5-124">Componentes</span><span class="sxs-lookup"><span data-stu-id="255f5-124">Components</span></span>

* <span data-ttu-id="255f5-125">[Azure Active Directory][aad-docs] es el directorio multiinquilino basado en la nube y el servicio de administración de identidades de Microsoft.</span><span class="sxs-lookup"><span data-stu-id="255f5-125">[Azure Active Directory][aad-docs] is Microsoft’s multi-tenant cloud-based directory and identity management service.</span></span> <span data-ttu-id="255f5-126">Azure AD admite un conector B2C, lo que permite identificar los individuos mediante identificadores externos, como Google, Facebook o cuenta Microsoft.</span><span class="sxs-lookup"><span data-stu-id="255f5-126">Azure AD supports a B2C connector allowing you to identify individuals using external IDs such as Google, Facebook, or a Microsoft Account.</span></span>
* <span data-ttu-id="255f5-127">[App Service][appservice-docs] le permite crear y hospedar aplicaciones web en el lenguaje de programación que prefiera sin tener que administrar la infraestructura.</span><span class="sxs-lookup"><span data-stu-id="255f5-127">[App Service][appservice-docs] enables you to build and host web applications in the programming language of your choice without managing infrastructure.</span></span>
* <span data-ttu-id="255f5-128">[Bot Service][botservice-docs] proporciona herramientas para crear, probar, implementar y administrar bots inteligentes.</span><span class="sxs-lookup"><span data-stu-id="255f5-128">[Bot Service][botservice-docs] provides tools to build, test, deploy, and manage intelligent bots.</span></span>
* <span data-ttu-id="255f5-129">[Cognitive Services][cognitive-docs] permite usar algoritmos inteligentes para ver, oír, hablar, comprender e interpretar las necesidades de los usuarios con formas de comunicación naturales.</span><span class="sxs-lookup"><span data-stu-id="255f5-129">[Cognitive Services][cognitive-docs] lets you use intelligent algorithms to see, hear, speak, understand and interpret your user needs through natural methods of communication.</span></span>
* <span data-ttu-id="255f5-130">[SQL Database][sqldatabase-docs] es un servicio de base de datos relacional en la nube totalmente administrado que proporciona compatibilidad con el motor de SQL Server.</span><span class="sxs-lookup"><span data-stu-id="255f5-130">[SQL Database][sqldatabase-docs] is a fully managed relational cloud database service that provides SQL Server engine compatibility.</span></span>
* <span data-ttu-id="255f5-131">[Application Insights][appinsights-docs] es un servicio de Application Performance Management (APM) extensible que permite supervisar el rendimiento de las aplicaciones, como su bot de chat.</span><span class="sxs-lookup"><span data-stu-id="255f5-131">[Application Insights][appinsights-docs] is an extensible Application Performance Management (APM) service that lets you monitor the performance of applications, such as your chatbot.</span></span>

### <a name="alternatives"></a><span data-ttu-id="255f5-132">Alternativas</span><span class="sxs-lookup"><span data-stu-id="255f5-132">Alternatives</span></span>

* <span data-ttu-id="255f5-133">[Microsoft Speech API][speech-api] puede usarse para cambiar la forma en que los clientes se comunican con el bot.</span><span class="sxs-lookup"><span data-stu-id="255f5-133">[Microsoft Speech API][speech-api] can be used to change how customers interface with your bot.</span></span>
* <span data-ttu-id="255f5-134">[QnA Maker][qna-maker] puede utilizarse para agregar rápidamente conocimiento al bot a partir de contenido semiestructurado, por ejemplo, preguntas frecuentes.</span><span class="sxs-lookup"><span data-stu-id="255f5-134">[QnA Maker][qna-maker] can be used as to quickly add knowledge to your bot from semi-structured content like an FAQ.</span></span>
* <span data-ttu-id="255f5-135">[Translator Text][translator] es un servicio que permite agregar al bot compatibilidad con varios idiomas fácilmente.</span><span class="sxs-lookup"><span data-stu-id="255f5-135">[Translator Text][translator] is a service that you might consider to easily add multi-lingual support to your bot.</span></span>

## <a name="considerations"></a><span data-ttu-id="255f5-136">Consideraciones</span><span class="sxs-lookup"><span data-stu-id="255f5-136">Considerations</span></span>

### <a name="availability"></a><span data-ttu-id="255f5-137">Disponibilidad</span><span class="sxs-lookup"><span data-stu-id="255f5-137">Availability</span></span>

<span data-ttu-id="255f5-138">Esta solución utiliza Azure SQL Database para almacenar las reservas de los clientes.</span><span class="sxs-lookup"><span data-stu-id="255f5-138">This solution uses Azure SQL Database for storing customer reservations.</span></span> <span data-ttu-id="255f5-139">SQL Database incluye bases de datos con redundancia de zona, grupos de conmutación por error y replicación geográfica.</span><span class="sxs-lookup"><span data-stu-id="255f5-139">SQL Database includes zone redundant databases, failover groups, and geo-replication.</span></span> <span data-ttu-id="255f5-140">Para más información, consulte [Funcionalidades de disponibilidad de Azure SQL Database][sqlavailability-docs].</span><span class="sxs-lookup"><span data-stu-id="255f5-140">For more information, see [Azure SQL Database availability capabilities][sqlavailability-docs].</span></span>

<span data-ttu-id="255f5-141">Para ver otros temas de escalabilidad, consulte la [lista de comprobación de disponibilidad][availability] que encontrará en el centro de arquitectura de Azure.</span><span class="sxs-lookup"><span data-stu-id="255f5-141">For other scalability topics, see the [availability checklist][availability] in the Azure Architecture Center.</span></span>

### <a name="scalability"></a><span data-ttu-id="255f5-142">Escalabilidad</span><span class="sxs-lookup"><span data-stu-id="255f5-142">Scalability</span></span>

<span data-ttu-id="255f5-143">Esta solución utiliza Azure App Service.</span><span class="sxs-lookup"><span data-stu-id="255f5-143">This solution uses Azure App Service.</span></span> <span data-ttu-id="255f5-144">Con App Service, puede escalar automáticamente el número de instancias que ejecutan su bot.</span><span class="sxs-lookup"><span data-stu-id="255f5-144">With App Service, you can automatically scale the number of instances that run your bot.</span></span> <span data-ttu-id="255f5-145">Esta funcionalidad le permite satisfacer la demanda de los clientes para su aplicación web y bot de chat.</span><span class="sxs-lookup"><span data-stu-id="255f5-145">This functionality lets you keep up with customer demand for your web application and chatbot.</span></span> <span data-ttu-id="255f5-146">Para más información sobre el escalado automático, consulte [Procedimientos recomendados de escalado automático][autoscaling] en el centro de arquitectura.</span><span class="sxs-lookup"><span data-stu-id="255f5-146">For more information on autoscale, see [Autoscaling best practices][autoscaling] in the architecture center.</span></span>

<span data-ttu-id="255f5-147">Para ver otros temas de escalabilidad, consulte la [lista de comprobación de escalabilidad][scalability] que encontrará en el centro de arquitectura de Azure.</span><span class="sxs-lookup"><span data-stu-id="255f5-147">For other scalability topics, see the [scalability checklist][scalability] in the Azure Architecture Center.</span></span>

### <a name="security"></a><span data-ttu-id="255f5-148">Seguridad</span><span class="sxs-lookup"><span data-stu-id="255f5-148">Security</span></span>

<span data-ttu-id="255f5-149">Esta solución usa Azure Active Directory B2C para autenticar los usuarios.</span><span class="sxs-lookup"><span data-stu-id="255f5-149">This solution uses Azure Active Directory B2C (Business 2 Consumer) to authenticate users.</span></span> <span data-ttu-id="255f5-150">Con AAD B2C, su bot de chat de no almacena ninguna información de cuenta confidencial de sus clientes ni las credenciales.</span><span class="sxs-lookup"><span data-stu-id="255f5-150">With AAD B2C, your chatbot doesn't store any sensitive customer account information or credentials.</span></span> <span data-ttu-id="255f5-151">Para más información, consulte [Introducción a Azure Active Directory B2C][aadb2c-docs].</span><span class="sxs-lookup"><span data-stu-id="255f5-151">For more information, see [Azure Active Directory B2C overview][aadb2c-docs].</span></span>

<span data-ttu-id="255f5-152">La información almacenada en Azure SQL Database se cifra en reposo con cifrado de datos transparente (TDE).</span><span class="sxs-lookup"><span data-stu-id="255f5-152">Information stored in Azure SQL Database is encrypted at rest with transparent data encryption (TDE).</span></span> <span data-ttu-id="255f5-153">SQL Database también ofrece Always Encrypted, que cifra los datos durante las operaciones de consulta y procesamiento.</span><span class="sxs-lookup"><span data-stu-id="255f5-153">SQL Database also offers Always Encrypted which encrypts data during querying and processing.</span></span> <span data-ttu-id="255f5-154">Para más información sobre la seguridad de SQL Database, consulte [Cumplimiento y seguridad de Azure SQL Database][sqlsecurity-docs].</span><span class="sxs-lookup"><span data-stu-id="255f5-154">For more information on SQL Database security, see [Azure SQL Database security and compliance][sqlsecurity-docs].</span></span>

<span data-ttu-id="255f5-155">Para obtener instrucciones generales sobre el diseño de soluciones seguras, consulte la [documentación de seguridad de Azure][security].</span><span class="sxs-lookup"><span data-stu-id="255f5-155">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="255f5-156">Resistencia</span><span class="sxs-lookup"><span data-stu-id="255f5-156">Resiliency</span></span>

<span data-ttu-id="255f5-157">Esta solución utiliza Azure SQL Database para almacenar las reservas de los clientes.</span><span class="sxs-lookup"><span data-stu-id="255f5-157">This solution uses Azure SQL Database for storing customer reservations.</span></span> <span data-ttu-id="255f5-158">SQL Database incluye bases de datos con redundancia de zona, grupos de conmutación por error, replicación geográfica y copias de seguridad automáticas.</span><span class="sxs-lookup"><span data-stu-id="255f5-158">SQL Database includes zone redundant databases, failover groups, geo-replication, and automatic backups.</span></span> <span data-ttu-id="255f5-159">Estas características permiten que la aplicación continúe ejecutándose en caso de un evento de mantenimiento o una interrupción.</span><span class="sxs-lookup"><span data-stu-id="255f5-159">These features allow your application to continue running in the event of a maintenance event or outage.</span></span> <span data-ttu-id="255f5-160">Para más información, consulte [Funcionalidades de disponibilidad de Azure SQL Database][sqlavailability-docs].</span><span class="sxs-lookup"><span data-stu-id="255f5-160">For more information, see [Azure SQL Database availability capabilities][sqlavailability-docs].</span></span>

<span data-ttu-id="255f5-161">Para supervisar el estado de mantenimiento de la aplicación, esta solución usa Application Insights.</span><span class="sxs-lookup"><span data-stu-id="255f5-161">To monitor the health of your application, this solution uses Application Insights.</span></span> <span data-ttu-id="255f5-162">Con Application Insights, puede generar alertas y responder a problemas de rendimiento que pueden afectar a la experiencia del cliente y la disponibilidad del bot de chat.</span><span class="sxs-lookup"><span data-stu-id="255f5-162">With Application Insights, you can generate alerts and respond to performance issues that would impact the customer experience and availability of the chatbot.</span></span> <span data-ttu-id="255f5-163">Para más información, consulte [¿Qué es Application Insights?][appinsights-docs]</span><span class="sxs-lookup"><span data-stu-id="255f5-163">For more information, see [What is Application Insights?][appinsights-docs]</span></span>

<span data-ttu-id="255f5-164">Para obtener instrucciones generales sobre el diseño de soluciones resistentes, consulte [Diseño de aplicaciones resistentes de Azure][resiliency].</span><span class="sxs-lookup"><span data-stu-id="255f5-164">For general guidance on designing resilient solutions, see [Designing resilient applications for Azure][resiliency].</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="255f5-165">Implementación de la solución</span><span class="sxs-lookup"><span data-stu-id="255f5-165">Deploy the solution</span></span>

<span data-ttu-id="255f5-166">Esta solución se divide en tres componentes para explorar las áreas que más le interesen:</span><span class="sxs-lookup"><span data-stu-id="255f5-166">This solution is divided into three components for you to explore areas that you are most focused on:</span></span>

* <span data-ttu-id="255f5-167">[Componentes de infraestructura](#deploy-infrastructure-components).</span><span class="sxs-lookup"><span data-stu-id="255f5-167">[Infrastructure components](#deploy-infrastructure-components).</span></span> <span data-ttu-id="255f5-168">Use una plantilla de Azure Resource Manager para implementar los componentes fundamentales de la infraestructura de un servicio de aplicaciones, una aplicación web, del servicio Application Insights, de una cuenta de almacenamiento y de SQL Server y una base de datos.</span><span class="sxs-lookup"><span data-stu-id="255f5-168">Use an Azure Resource Manger template to deploy the core infrastructure components of an App Service, Web App, Application Insights, Storage account, and SQL Server and database.</span></span>
* <span data-ttu-id="255f5-169">[Bot de chat de una aplicación web](#deploy-web-app-chatbot).</span><span class="sxs-lookup"><span data-stu-id="255f5-169">[Web App Chatbot](#deploy-web-app-chatbot).</span></span> <span data-ttu-id="255f5-170">Use la CLI de Azure para implementar un bot con Bot Service y una aplicación de Language Understanding Intelligent Services (LUIS).</span><span class="sxs-lookup"><span data-stu-id="255f5-170">Use the Azure CLI to deploy a bot with the Bot Service and Language Understanding and Intelligent Services (LUIS) app.</span></span>
* <span data-ttu-id="255f5-171">[Ejemplo de aplicación de bot de chat de C#](#deploy-chatbot-c-application-code).</span><span class="sxs-lookup"><span data-stu-id="255f5-171">[Sample C# chatbot application](#deploy-chatbot-c-application-code).</span></span> <span data-ttu-id="255f5-172">Use Visual Studio para revisar el código de ejemplo de una aplicación de reservas de hotel en C#, e implementar un bot en Azure.</span><span class="sxs-lookup"><span data-stu-id="255f5-172">Use Visual Studio to review the sample hotel reservation C# application code and deploy to a bot in Azure.</span></span>

<span data-ttu-id="255f5-173">**Requisitos previos**</span><span class="sxs-lookup"><span data-stu-id="255f5-173">**Prerequisites.**</span></span> <span data-ttu-id="255f5-174">Debe tener una cuenta de Azure.</span><span class="sxs-lookup"><span data-stu-id="255f5-174">You must have an existing Azure account.</span></span> <span data-ttu-id="255f5-175">Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.</span><span class="sxs-lookup"><span data-stu-id="255f5-175">If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.</span></span>

### <a name="deploy-infrastructure-components"></a><span data-ttu-id="255f5-176">Implementación de los componentes de infraestructura</span><span class="sxs-lookup"><span data-stu-id="255f5-176">Deploy infrastructure components</span></span>

<span data-ttu-id="255f5-177">Para implementar los componentes de infraestructura con una plantilla de Azure Resource Manager, realice los pasos siguientes.</span><span class="sxs-lookup"><span data-stu-id="255f5-177">To deploy the infrastructure components with an Azure Resource Manager template, perform the following steps.</span></span>

1. <span data-ttu-id="255f5-178">Haga clic en botón **Implementar en Azure**.</span><span class="sxs-lookup"><span data-stu-id="255f5-178">Click the **Deploy to Azure** button:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fcommerce-chatbot.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. <span data-ttu-id="255f5-179">Espere a que la implementación de plantilla se abra en Azure Portal y complete los pasos siguientes:</span><span class="sxs-lookup"><span data-stu-id="255f5-179">Wait for the template deployment to open in the Azure portal, then complete the following steps:</span></span>
   * <span data-ttu-id="255f5-180">Haga clic en **Crear nuevo** para el grupo de recursos y proporcione un nombre, por ejemplo, *myCommerceChatBotInfrastructure*.</span><span class="sxs-lookup"><span data-stu-id="255f5-180">Choose to **Create new** resource group, then provide a name such as *myCommerceChatBotInfrastructure* in the text box.</span></span>
   * <span data-ttu-id="255f5-181">Seleccione una región en el cuadro de lista desplegable **Ubicación**.</span><span class="sxs-lookup"><span data-stu-id="255f5-181">Select a region from the **Location** drop-down box.</span></span>
   * <span data-ttu-id="255f5-182">Proporcione un nombre de usuario y una contraseña segura para la cuenta de administrador de SQL Server.</span><span class="sxs-lookup"><span data-stu-id="255f5-182">Provide a username and secure password for the SQL Server administrator account.</span></span>
   * <span data-ttu-id="255f5-183">Revise los términos y condiciones, y seleccione **Acepto los términos y condiciones indicados anteriormente**.</span><span class="sxs-lookup"><span data-stu-id="255f5-183">Review the terms and conditions, then check **I agree to the terms and conditions stated above**.</span></span>
   * <span data-ttu-id="255f5-184">Seleccione el botón **Comprar**.</span><span class="sxs-lookup"><span data-stu-id="255f5-184">Select the **Purchase** button.</span></span>

<span data-ttu-id="255f5-185">La implementación tarda unos minutos en finalizar.</span><span class="sxs-lookup"><span data-stu-id="255f5-185">It takes a few minutes for the deployment to complete.</span></span>

### <a name="deploy-web-app-chatbot"></a><span data-ttu-id="255f5-186">Implementación de una aplicación web de bot de chat</span><span class="sxs-lookup"><span data-stu-id="255f5-186">Deploy Web App chatbot</span></span>

<span data-ttu-id="255f5-187">Para crear el bot de chat, use la CLI de Azure.</span><span class="sxs-lookup"><span data-stu-id="255f5-187">To create the chatbot, use the Azure CLI.</span></span> <span data-ttu-id="255f5-188">En el ejemplo siguiente se instala la extensión de la CLI para Bot Service, se crea un grupo de recursos y se implementa un bot que usa Application Insights.</span><span class="sxs-lookup"><span data-stu-id="255f5-188">The following example installs the CLI extension for Bot Service, creates a resource group, then deploys a bot that uses Application Insights.</span></span> <span data-ttu-id="255f5-189">Cuando se le solicite, autentique la cuenta Microsoft y permita al bot que se registre en Bot Service y la aplicación de Language Understanding Intelligent Services (LUIS).</span><span class="sxs-lookup"><span data-stu-id="255f5-189">When prompted, authenticate your Microsoft account and allow the bot to register itself with the Bot Service and Language Understanding and Intelligent Services (LUIS) app.</span></span>

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

### <a name="deploy-chatbot-c-application-code"></a><span data-ttu-id="255f5-190">Implementación del código de la aplicación de bot de chat de C#</span><span class="sxs-lookup"><span data-stu-id="255f5-190">Deploy chatbot C# application code</span></span>

<span data-ttu-id="255f5-191">En GitHub hay disponible una aplicación de ejemplo en C#:</span><span class="sxs-lookup"><span data-stu-id="255f5-191">A sample C# application is available on GitHub:</span></span> 

* [<span data-ttu-id="255f5-192">Ejemplo de bot comercial en C#</span><span class="sxs-lookup"><span data-stu-id="255f5-192">Commerce Bot C# sample</span></span>](https://github.com/Microsoft/AzureBotServices-scenarios/tree/master/CSharp/Commerce/src)

<span data-ttu-id="255f5-193">La aplicación de ejemplo incluye los componentes de autenticación de Azure Active Directory y la integración con el componente Language Understanding Intelligent Services (LUIS) de Cognitive Services.</span><span class="sxs-lookup"><span data-stu-id="255f5-193">The sample application includes the Azure Active Directory authentication components and integration with the Language Understanding and Intelligent Services (LUIS) component of Cognitive Services.</span></span> <span data-ttu-id="255f5-194">La aplicación requiere Visual Studio para compilar e implementar la solución.</span><span class="sxs-lookup"><span data-stu-id="255f5-194">The application requires Visual Studio to build and deploy the solution.</span></span> <span data-ttu-id="255f5-195">En la documentación del repositorio de GitHub encontrará información adicional sobre cómo configurar AAD B2C y la aplicación de LUIS.</span><span class="sxs-lookup"><span data-stu-id="255f5-195">Additional information on configuring AAD B2C and the LUIS app can be found in the GitHub repo documentation.</span></span>

## <a name="pricing"></a><span data-ttu-id="255f5-196">Precios</span><span class="sxs-lookup"><span data-stu-id="255f5-196">Pricing</span></span>

<span data-ttu-id="255f5-197">Para explorar el costo de ejecutar esta solución, todos los servicios están preconfigurados en la calculadora de costos.</span><span class="sxs-lookup"><span data-stu-id="255f5-197">To explore the cost of running this solution, all of the services are pre-configured in the cost calculator.</span></span> <span data-ttu-id="255f5-198">Para ver cómo cambiarían los precios en su caso concreto, cambie las variables pertinentes para que coincidan con el tráfico esperado.</span><span class="sxs-lookup"><span data-stu-id="255f5-198">To see how the pricing would change for your particular use case, change the appropriate variables to match your expected traffic.</span></span>

<span data-ttu-id="255f5-199">Hemos proporcionado tres ejemplos de perfiles de costo según la cantidad de mensajes que se espera que su bot de chat procese:</span><span class="sxs-lookup"><span data-stu-id="255f5-199">We have provided three sample cost profiles based on the amount of messages you expect your chatbot to process:</span></span>

* <span data-ttu-id="255f5-200">[Pequeño][small-pricing]: se corresponde con un procesamiento de menos de 10 000 mensajes al mes.</span><span class="sxs-lookup"><span data-stu-id="255f5-200">[Small][small-pricing]: this correlates to processing < 10,000 messages per month.</span></span>
* <span data-ttu-id="255f5-201">[Mediano][medium-pricing]: se corresponde con un procesamiento de menos de 500 000 mensajes al mes.</span><span class="sxs-lookup"><span data-stu-id="255f5-201">[Medium][medium-pricing]: this correlates to processing < 500,000 messages per month.</span></span>
* <span data-ttu-id="255f5-202">[Grande][large-pricing]: se corresponde con un procesamiento de menos de 10 millones de mensajes al mes.</span><span class="sxs-lookup"><span data-stu-id="255f5-202">[Large][large-pricing]: this correlates to processing < 10 million messages per month.</span></span>

## <a name="related-resources"></a><span data-ttu-id="255f5-203">Recursos relacionados</span><span class="sxs-lookup"><span data-stu-id="255f5-203">Related Resources</span></span>

<span data-ttu-id="255f5-204">Para obtener un conjunto de tutoriales guiados sobre cómo sacar partido de Azure Bot Service, consulte el [nodo de tutoriales][botservice-docs] de la documentación.</span><span class="sxs-lookup"><span data-stu-id="255f5-204">For a set of guided tutorials on leveraging the Azure Bot Service, see the [tutorial node][botservice-docs] of the documentation.</span></span>

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