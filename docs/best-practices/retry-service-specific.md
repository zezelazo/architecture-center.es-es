---
title: Vuelva a intentar la orientación específica del servicio
description: Instrucciones específicas de servicios para establecer el mecanismo de reintento.
author: dragon119
ms.date: 07/13/2016
pnp.series.title: Best Practices
ms.openlocfilehash: 77cf5d90373da2118d34301bd5c790080d3cf63f
ms.sourcegitcommit: 9a2d56ac7927f0a2bbfee07198d43d9c5cb85755
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/22/2018
ms.locfileid: "36327694"
---
# <a name="retry-guidance-for-specific-services"></a><span data-ttu-id="06513-103">Guía de reintentos para servicios específicos</span><span class="sxs-lookup"><span data-stu-id="06513-103">Retry guidance for specific services</span></span>

<span data-ttu-id="06513-104">La mayoría de SDK de cliente y de servicios de Azure incluye un mecanismo de reintento.</span><span class="sxs-lookup"><span data-stu-id="06513-104">Most Azure services and client SDKs include a retry mechanism.</span></span> <span data-ttu-id="06513-105">Sin embargo, estos son diferentes porque cada servicio tiene requisitos y características diferentes, por lo que cada mecanismo de reintento se ajusta a un servicio específico.</span><span class="sxs-lookup"><span data-stu-id="06513-105">However, these differ because each service has different characteristics and requirements, and so each retry mechanism is tuned to a specific service.</span></span> <span data-ttu-id="06513-106">Esta guía resume las características de mecanismo de reintento de la mayoría de los servicios de Azure e incluye información para ayudarle a usar, adaptar o ampliar el mecanismo de reintento para ese servicio.</span><span class="sxs-lookup"><span data-stu-id="06513-106">This guide summarizes the retry mechanism features for the majority of Azure services, and includes information to help you use, adapt, or extend the retry mechanism for that service.</span></span>

<span data-ttu-id="06513-107">Para obtener instrucciones generales sobre el control de errores transitorios y reintentar conexiones y operaciones en servicios y recursos, consulte [Guía de reintentos](./transient-faults.md).</span><span class="sxs-lookup"><span data-stu-id="06513-107">For general guidance on handling transient faults, and retrying connections and operations against services and resources, see [Retry guidance](./transient-faults.md).</span></span>

<span data-ttu-id="06513-108">En la tabla siguiente se resumen las características de reintento de los servicios de Azure descritos en esta guía.</span><span class="sxs-lookup"><span data-stu-id="06513-108">The following table summarizes the retry features for the Azure services described in this guidance.</span></span>

| <span data-ttu-id="06513-109">**Servicio**</span><span class="sxs-lookup"><span data-stu-id="06513-109">**Service**</span></span> | <span data-ttu-id="06513-110">**Capacidades de reintento**</span><span class="sxs-lookup"><span data-stu-id="06513-110">**Retry capabilities**</span></span> | <span data-ttu-id="06513-111">**Configuración de directivas**</span><span class="sxs-lookup"><span data-stu-id="06513-111">**Policy configuration**</span></span> | <span data-ttu-id="06513-112">**Ámbito**</span><span class="sxs-lookup"><span data-stu-id="06513-112">**Scope**</span></span> | <span data-ttu-id="06513-113">**Características de telemetría**</span><span class="sxs-lookup"><span data-stu-id="06513-113">**Telemetry features**</span></span> |
| --- | --- | --- | --- | --- |
| <span data-ttu-id="06513-114">**[Azure Active Directory](#azure-active-directory)**</span><span class="sxs-lookup"><span data-stu-id="06513-114">**[Azure Active Directory](#azure-active-directory)**</span></span> |<span data-ttu-id="06513-115">Nativo en la biblioteca ADAL</span><span class="sxs-lookup"><span data-stu-id="06513-115">Native in ADAL library</span></span> |<span data-ttu-id="06513-116">Insertado en la biblioteca ADAL</span><span class="sxs-lookup"><span data-stu-id="06513-116">Embeded into ADAL library</span></span> |<span data-ttu-id="06513-117">Interno</span><span class="sxs-lookup"><span data-stu-id="06513-117">Internal</span></span> |<span data-ttu-id="06513-118">None</span><span class="sxs-lookup"><span data-stu-id="06513-118">None</span></span> |
| <span data-ttu-id="06513-119">**[Cosmos DB](#cosmos-db)**</span><span class="sxs-lookup"><span data-stu-id="06513-119">**[Cosmos DB](#cosmos-db)**</span></span> |<span data-ttu-id="06513-120">Nativo en servicio</span><span class="sxs-lookup"><span data-stu-id="06513-120">Native in service</span></span> |<span data-ttu-id="06513-121">No configurable</span><span class="sxs-lookup"><span data-stu-id="06513-121">Non-configurable</span></span> |<span data-ttu-id="06513-122">Global</span><span class="sxs-lookup"><span data-stu-id="06513-122">Global</span></span> |<span data-ttu-id="06513-123">TraceSource</span><span class="sxs-lookup"><span data-stu-id="06513-123">TraceSource</span></span> |
| <span data-ttu-id="06513-124">**[Event Hubs](#azure-event-hubs)**</span><span class="sxs-lookup"><span data-stu-id="06513-124">**[Event Hubs](#azure-event-hubs)**</span></span> |<span data-ttu-id="06513-125">Nativo en el cliente</span><span class="sxs-lookup"><span data-stu-id="06513-125">Native in client</span></span> |<span data-ttu-id="06513-126">Programático</span><span class="sxs-lookup"><span data-stu-id="06513-126">Programmatic</span></span> |<span data-ttu-id="06513-127">Cliente</span><span class="sxs-lookup"><span data-stu-id="06513-127">Client</span></span> |<span data-ttu-id="06513-128">None</span><span class="sxs-lookup"><span data-stu-id="06513-128">None</span></span> |
| <span data-ttu-id="06513-129">**[Redis Cache](#azure-redis-cache)**</span><span class="sxs-lookup"><span data-stu-id="06513-129">**[Redis Cache](#azure-redis-cache)**</span></span> |<span data-ttu-id="06513-130">Nativo en el cliente</span><span class="sxs-lookup"><span data-stu-id="06513-130">Native in client</span></span> |<span data-ttu-id="06513-131">Programático</span><span class="sxs-lookup"><span data-stu-id="06513-131">Programmatic</span></span> |<span data-ttu-id="06513-132">Cliente</span><span class="sxs-lookup"><span data-stu-id="06513-132">Client</span></span> |<span data-ttu-id="06513-133">TextWriter</span><span class="sxs-lookup"><span data-stu-id="06513-133">TextWriter</span></span> |
| <span data-ttu-id="06513-134">**[Search](#azure-search)**</span><span class="sxs-lookup"><span data-stu-id="06513-134">**[Search](#azure-search)**</span></span> |<span data-ttu-id="06513-135">Nativo en el cliente</span><span class="sxs-lookup"><span data-stu-id="06513-135">Native in client</span></span> |<span data-ttu-id="06513-136">Programático</span><span class="sxs-lookup"><span data-stu-id="06513-136">Programmatic</span></span> |<span data-ttu-id="06513-137">Cliente</span><span class="sxs-lookup"><span data-stu-id="06513-137">Client</span></span> |<span data-ttu-id="06513-138">ETW o personalizado</span><span class="sxs-lookup"><span data-stu-id="06513-138">ETW or Custom</span></span> |
| <span data-ttu-id="06513-139">**[Service Bus](#service-bus)**</span><span class="sxs-lookup"><span data-stu-id="06513-139">**[Service Bus](#service-bus)**</span></span> |<span data-ttu-id="06513-140">Nativo en el cliente</span><span class="sxs-lookup"><span data-stu-id="06513-140">Native in client</span></span> |<span data-ttu-id="06513-141">Programático</span><span class="sxs-lookup"><span data-stu-id="06513-141">Programmatic</span></span> |<span data-ttu-id="06513-142">Administrador de espacio de nombres, fábrica de mensajería y cliente</span><span class="sxs-lookup"><span data-stu-id="06513-142">Namespace Manager, Messaging Factory, and Client</span></span> |<span data-ttu-id="06513-143">ETW</span><span class="sxs-lookup"><span data-stu-id="06513-143">ETW</span></span> |
| <span data-ttu-id="06513-144">**[Service Fabric](#service-fabric)**</span><span class="sxs-lookup"><span data-stu-id="06513-144">**[Service Fabric](#service-fabric)**</span></span> |<span data-ttu-id="06513-145">Nativo en el cliente</span><span class="sxs-lookup"><span data-stu-id="06513-145">Native in client</span></span> |<span data-ttu-id="06513-146">Programático</span><span class="sxs-lookup"><span data-stu-id="06513-146">Programmatic</span></span> |<span data-ttu-id="06513-147">Cliente</span><span class="sxs-lookup"><span data-stu-id="06513-147">Client</span></span> |<span data-ttu-id="06513-148">None</span><span class="sxs-lookup"><span data-stu-id="06513-148">None</span></span> | 
| <span data-ttu-id="06513-149">**[SQL Database con ADO.NET](#sql-database-using-adonet)**</span><span class="sxs-lookup"><span data-stu-id="06513-149">**[SQL Database with ADO.NET](#sql-database-using-adonet)**</span></span> |[<span data-ttu-id="06513-150">Polly</span><span class="sxs-lookup"><span data-stu-id="06513-150">Polly</span></span>](#transient-fault-handling-with-polly) |<span data-ttu-id="06513-151">Declarativo y programático</span><span class="sxs-lookup"><span data-stu-id="06513-151">Declarative and programmatic</span></span> |<span data-ttu-id="06513-152">Instrucciones únicas o bloques de código</span><span class="sxs-lookup"><span data-stu-id="06513-152">Single statements or blocks of code</span></span> |<span data-ttu-id="06513-153">Personalizado</span><span class="sxs-lookup"><span data-stu-id="06513-153">Custom</span></span> |
| <span data-ttu-id="06513-154">**[SQL Database con Entity Framework](#sql-database-using-entity-framework-6)**</span><span class="sxs-lookup"><span data-stu-id="06513-154">**[SQL Database with Entity Framework](#sql-database-using-entity-framework-6)**</span></span> |<span data-ttu-id="06513-155">Nativo en el cliente</span><span class="sxs-lookup"><span data-stu-id="06513-155">Native in client</span></span> |<span data-ttu-id="06513-156">Programático</span><span class="sxs-lookup"><span data-stu-id="06513-156">Programmatic</span></span> |<span data-ttu-id="06513-157">Global por AppDomain</span><span class="sxs-lookup"><span data-stu-id="06513-157">Global per AppDomain</span></span> |<span data-ttu-id="06513-158">None</span><span class="sxs-lookup"><span data-stu-id="06513-158">None</span></span> |
| <span data-ttu-id="06513-159">**[SQL Database con Entity Framework Core](#sql-database-using-entity-framework-core)**</span><span class="sxs-lookup"><span data-stu-id="06513-159">**[SQL Database with Entity Framework Core](#sql-database-using-entity-framework-core)**</span></span> |<span data-ttu-id="06513-160">Nativo en el cliente</span><span class="sxs-lookup"><span data-stu-id="06513-160">Native in client</span></span> |<span data-ttu-id="06513-161">Programático</span><span class="sxs-lookup"><span data-stu-id="06513-161">Programmatic</span></span> |<span data-ttu-id="06513-162">Global por AppDomain</span><span class="sxs-lookup"><span data-stu-id="06513-162">Global per AppDomain</span></span> |<span data-ttu-id="06513-163">None</span><span class="sxs-lookup"><span data-stu-id="06513-163">None</span></span> |
| <span data-ttu-id="06513-164">**[Storage](#azure-storage)**</span><span class="sxs-lookup"><span data-stu-id="06513-164">**[Storage](#azure-storage)**</span></span> |<span data-ttu-id="06513-165">Nativo en el cliente</span><span class="sxs-lookup"><span data-stu-id="06513-165">Native in client</span></span> |<span data-ttu-id="06513-166">Programático</span><span class="sxs-lookup"><span data-stu-id="06513-166">Programmatic</span></span> |<span data-ttu-id="06513-167">Operaciones de cliente e individuales</span><span class="sxs-lookup"><span data-stu-id="06513-167">Client and individual operations</span></span> |<span data-ttu-id="06513-168">TraceSource</span><span class="sxs-lookup"><span data-stu-id="06513-168">TraceSource</span></span> |

> [!NOTE]
> <span data-ttu-id="06513-169">Para la mayoría de los mecanismos de reintento integrados de Azure, actualmente no hay ninguna manera de aplicar una directiva de reintento diferente para distintos tipos de error o excepción.</span><span class="sxs-lookup"><span data-stu-id="06513-169">For most of the Azure built-in retry mechanisms, there is currently no way apply a different retry policy for different types of error or exception.</span></span> <span data-ttu-id="06513-170">Debe configurar una directiva que proporcione un promedio de rendimiento y disponibilidad óptimo.</span><span class="sxs-lookup"><span data-stu-id="06513-170">You should configure a policy that provides the optimum average performance and availability.</span></span> <span data-ttu-id="06513-171">Una forma de ajustar la directiva consiste en analizar archivos de registro para determinar el tipo de errores transitorios que se están produciendo.</span><span class="sxs-lookup"><span data-stu-id="06513-171">One way to fine-tune the policy is to analyze log files to determine the type of transient faults that are occurring.</span></span> 

## <a name="azure-active-directory"></a><span data-ttu-id="06513-172">Azure Active Directory</span><span class="sxs-lookup"><span data-stu-id="06513-172">Azure Active Directory</span></span>
<span data-ttu-id="06513-173">Azure Active Directory (Azure AD) es una solución de nube de administración de identidades y accesos integral que combina servicios de directorio de núcleo, gobierno de identidades avanzado, seguridad y administración de acceso a aplicaciones.</span><span class="sxs-lookup"><span data-stu-id="06513-173">Azure Active Directory (Azure AD) is a comprehensive identity and access management cloud solution that combines core directory services, advanced identity governance, security, and application access management.</span></span> <span data-ttu-id="06513-174">Azure AD también ofrece a los desarrolladores una plataforma de administración de identidades para proporcionar control de acceso a sus aplicaciones, según las reglas y directivas centralizadas.</span><span class="sxs-lookup"><span data-stu-id="06513-174">Azure AD also offers developers an identity management platform to deliver access control to their applications, based on centralized policy and rules.</span></span>

> [!NOTE]
> <span data-ttu-id="06513-175">Para obtener instrucciones de reintento en los puntos de conexión de Managed Service Identity, consulte [Uso de una identidad de servicio administrada de máquina virtual de Azure para obtener tokens](/azure/active-directory/managed-service-identity/how-to-use-vm-token#error-handling).</span><span class="sxs-lookup"><span data-stu-id="06513-175">For retry guidance on Managed Service Identity endpoints, see [How to use an Azure VM Managed Service Identity (MSI) for token acquisition](/azure/active-directory/managed-service-identity/how-to-use-vm-token#error-handling).</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="06513-176">Mecanismo de reintento</span><span class="sxs-lookup"><span data-stu-id="06513-176">Retry mechanism</span></span>
<span data-ttu-id="06513-177">Hay un mecanismo de reintento integrado para Azure Active Directory en la biblioteca de autenticación de Active Directory (ADAL).</span><span class="sxs-lookup"><span data-stu-id="06513-177">There is a built-in retry mechanism for Azure Active Directory in the Active Directory Authentication Library (ADAL).</span></span> <span data-ttu-id="06513-178">Para evitar bloqueos inesperados, se recomienda que las bibliotecas de terceros y el código de la aplicación **no** reintenten las conexiones con error y que dejen que ADAL controle los reintentos.</span><span class="sxs-lookup"><span data-stu-id="06513-178">To avoid unexpected lockouts, we recommend that third party libraries and application code do **not** retry failed connections, but allow ADAL to handle retries.</span></span> 

### <a name="retry-usage-guidance"></a><span data-ttu-id="06513-179">Instrucciones de uso del reintento</span><span class="sxs-lookup"><span data-stu-id="06513-179">Retry usage guidance</span></span>
<span data-ttu-id="06513-180">Tenga en cuenta las siguientes directrices cuando use Azure Active Directory:</span><span class="sxs-lookup"><span data-stu-id="06513-180">Consider the following guidelines when using Azure Active Directory:</span></span>

* <span data-ttu-id="06513-181">Cuando sea posible, utilice la biblioteca ADAL y la compatibilidad para reintentos integrada.</span><span class="sxs-lookup"><span data-stu-id="06513-181">When possible, use the ADAL library and the built-in support for retries.</span></span>
* <span data-ttu-id="06513-182">Si va a usar la API REST para Azure Active Directory, vuelva a intentar la operación si el código de resultado es 429 (Demasiadas solicitudes) o un error en el intervalo 5xx.</span><span class="sxs-lookup"><span data-stu-id="06513-182">If you are using the REST API for Azure Active Directory, retry the operation if the result code is 429 (Too Many Requests) or an error in the 5xx range.</span></span> <span data-ttu-id="06513-183">No lo intente de nuevo para cualquier otro error.</span><span class="sxs-lookup"><span data-stu-id="06513-183">Do not retry for any other errors.</span></span>
* <span data-ttu-id="06513-184">Se recomienda una directiva de retroceso exponencial para su uso en escenarios de lotes con Azure Active Directory.</span><span class="sxs-lookup"><span data-stu-id="06513-184">An exponential back-off policy is recommended for use in batch scenarios with Azure Active Directory.</span></span>

<span data-ttu-id="06513-185">Considere la posibilidad de comenzar con la configuración siguiente para las operaciones de reintento.</span><span class="sxs-lookup"><span data-stu-id="06513-185">Consider starting with the following settings for retrying operations.</span></span> <span data-ttu-id="06513-186">Esta es la configuración de propósito general, y debe supervisar las operaciones y ajustar los valores para adaptarlos a su propio escenario.</span><span class="sxs-lookup"><span data-stu-id="06513-186">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="06513-187">**Contexto**</span><span class="sxs-lookup"><span data-stu-id="06513-187">**Context**</span></span> | <span data-ttu-id="06513-188">**Destino de ejemplo E2E<br />latencia máxima**</span><span class="sxs-lookup"><span data-stu-id="06513-188">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="06513-189">**Estrategia de reintento**</span><span class="sxs-lookup"><span data-stu-id="06513-189">**Retry strategy**</span></span> | <span data-ttu-id="06513-190">**Configuración**</span><span class="sxs-lookup"><span data-stu-id="06513-190">**Settings**</span></span> | <span data-ttu-id="06513-191">**Valores**</span><span class="sxs-lookup"><span data-stu-id="06513-191">**Values**</span></span> | <span data-ttu-id="06513-192">**Cómo funciona**</span><span class="sxs-lookup"><span data-stu-id="06513-192">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="06513-193">Interactivo, interfaz de usuario</span><span class="sxs-lookup"><span data-stu-id="06513-193">Interactive, UI,</span></span><br /><span data-ttu-id="06513-194">o primer plano</span><span class="sxs-lookup"><span data-stu-id="06513-194">or foreground</span></span> |<span data-ttu-id="06513-195">2 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-195">2 sec</span></span> |<span data-ttu-id="06513-196">FixedInterval</span><span class="sxs-lookup"><span data-stu-id="06513-196">FixedInterval</span></span> |<span data-ttu-id="06513-197">Número de reintentos</span><span class="sxs-lookup"><span data-stu-id="06513-197">Retry count</span></span><br /><span data-ttu-id="06513-198">Intervalo de reintento</span><span class="sxs-lookup"><span data-stu-id="06513-198">Retry interval</span></span><br /><span data-ttu-id="06513-199">Primer reintento rápido</span><span class="sxs-lookup"><span data-stu-id="06513-199">First fast retry</span></span> |<span data-ttu-id="06513-200">3</span><span class="sxs-lookup"><span data-stu-id="06513-200">3</span></span><br /><span data-ttu-id="06513-201">500 ms</span><span class="sxs-lookup"><span data-stu-id="06513-201">500 ms</span></span><br /><span data-ttu-id="06513-202">true</span><span class="sxs-lookup"><span data-stu-id="06513-202">true</span></span> |<span data-ttu-id="06513-203">Intento 1 - retraso de 0 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-203">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="06513-204">Intento 2 - retraso de 500 ms</span><span class="sxs-lookup"><span data-stu-id="06513-204">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="06513-205">Intento 3 – retraso de 500 ms</span><span class="sxs-lookup"><span data-stu-id="06513-205">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="06513-206">Segundo plano o</span><span class="sxs-lookup"><span data-stu-id="06513-206">Background or</span></span><br /><span data-ttu-id="06513-207">proceso por lotes</span><span class="sxs-lookup"><span data-stu-id="06513-207">batch</span></span> |<span data-ttu-id="06513-208">60 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-208">60 sec</span></span> |<span data-ttu-id="06513-209">ExponentialBackoff</span><span class="sxs-lookup"><span data-stu-id="06513-209">ExponentialBackoff</span></span> |<span data-ttu-id="06513-210">Número de reintentos</span><span class="sxs-lookup"><span data-stu-id="06513-210">Retry count</span></span><br /><span data-ttu-id="06513-211">Interrupción mínima</span><span class="sxs-lookup"><span data-stu-id="06513-211">Min back-off</span></span><br /><span data-ttu-id="06513-212">Interrupción máxima</span><span class="sxs-lookup"><span data-stu-id="06513-212">Max back-off</span></span><br /><span data-ttu-id="06513-213">Interrupción delta</span><span class="sxs-lookup"><span data-stu-id="06513-213">Delta back-off</span></span><br /><span data-ttu-id="06513-214">Primer reintento rápido</span><span class="sxs-lookup"><span data-stu-id="06513-214">First fast retry</span></span> |<span data-ttu-id="06513-215">5</span><span class="sxs-lookup"><span data-stu-id="06513-215">5</span></span><br /><span data-ttu-id="06513-216">0 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-216">0 sec</span></span><br /><span data-ttu-id="06513-217">60 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-217">60 sec</span></span><br /><span data-ttu-id="06513-218">2 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-218">2 sec</span></span><br /><span data-ttu-id="06513-219">false</span><span class="sxs-lookup"><span data-stu-id="06513-219">false</span></span> |<span data-ttu-id="06513-220">Intento 1 - retraso de 0 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-220">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="06513-221">Intento 2 - retraso de ~2 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-221">Attempt 2 - delay ~2 sec</span></span><br /><span data-ttu-id="06513-222">Intento 3 - retraso de ~6 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-222">Attempt 3 - delay ~6 sec</span></span><br /><span data-ttu-id="06513-223">Intento 4 - retraso de ~14 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-223">Attempt 4 - delay ~14 sec</span></span><br /><span data-ttu-id="06513-224">Intento 5 - retraso de ~30 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-224">Attempt 5 - delay ~30 sec</span></span> |

### <a name="more-information"></a><span data-ttu-id="06513-225">Más información</span><span class="sxs-lookup"><span data-stu-id="06513-225">More information</span></span>
* <span data-ttu-id="06513-226">[Bibliotecas de autenticación de Azure Active Directory][adal]</span><span class="sxs-lookup"><span data-stu-id="06513-226">[Azure Active Directory Authentication Libraries][adal]</span></span>

## <a name="cosmos-db"></a><span data-ttu-id="06513-227">Cosmos DB</span><span class="sxs-lookup"><span data-stu-id="06513-227">Cosmos DB</span></span>

<span data-ttu-id="06513-228">Cosmos DB es una base de datos multimodelo completamente administrada que admite datos JSON sin esquema.</span><span class="sxs-lookup"><span data-stu-id="06513-228">Cosmos DB is a fully-managed multi-model database that supports schema-less JSON data.</span></span> <span data-ttu-id="06513-229">Ofrece un rendimiento confiable y configurable, procesamiento transaccional de JavaScript nativo y se ha creado para la nube con la escala elástica.</span><span class="sxs-lookup"><span data-stu-id="06513-229">It offers configurable and reliable performance, native JavaScript transactional processing, and is built for the cloud with elastic scale.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="06513-230">Mecanismo de reintento</span><span class="sxs-lookup"><span data-stu-id="06513-230">Retry mechanism</span></span>
<span data-ttu-id="06513-231">La clase `DocumentClient` reintenta automáticamente los intentos con error.</span><span class="sxs-lookup"><span data-stu-id="06513-231">The `DocumentClient` class automatically retries failed attempts.</span></span> <span data-ttu-id="06513-232">Para establecer el número de reintentos y el tiempo de espera máximo, configure [ConnectionPolicy.RetryOptions].</span><span class="sxs-lookup"><span data-stu-id="06513-232">To set the number of retries and the maximum wait time, configure [ConnectionPolicy.RetryOptions].</span></span> <span data-ttu-id="06513-233">Las excepciones que genera el cliente se encuentran más allá de la directiva de reintentos o no son errores transitorios.</span><span class="sxs-lookup"><span data-stu-id="06513-233">Exceptions that the client raises are either beyond the retry policy or are not transient errors.</span></span>

<span data-ttu-id="06513-234">Si Cosmos DB limita el cliente, devuelve un error HTTP 429.</span><span class="sxs-lookup"><span data-stu-id="06513-234">If Cosmos DB throttles the client, it returns an HTTP 429 error.</span></span> <span data-ttu-id="06513-235">Compruebe el código de estado en `DocumentClientException`.</span><span class="sxs-lookup"><span data-stu-id="06513-235">Check the status code in the `DocumentClientException`.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="06513-236">Configuración de directivas</span><span class="sxs-lookup"><span data-stu-id="06513-236">Policy configuration</span></span>
<span data-ttu-id="06513-237">La siguiente tabla muestra la configuración predeterminada de la clase `RetryOptions`.</span><span class="sxs-lookup"><span data-stu-id="06513-237">The following table shows the default settings for the `RetryOptions` class.</span></span>

| <span data-ttu-id="06513-238">Configuración</span><span class="sxs-lookup"><span data-stu-id="06513-238">Setting</span></span> | <span data-ttu-id="06513-239">Valor predeterminado</span><span class="sxs-lookup"><span data-stu-id="06513-239">Default value</span></span> | <span data-ttu-id="06513-240">DESCRIPCIÓN</span><span class="sxs-lookup"><span data-stu-id="06513-240">Description</span></span> |
| --- | --- | --- |
| <span data-ttu-id="06513-241">MaxRetryAttemptsOnThrottledRequests</span><span class="sxs-lookup"><span data-stu-id="06513-241">MaxRetryAttemptsOnThrottledRequests</span></span> |<span data-ttu-id="06513-242">9</span><span class="sxs-lookup"><span data-stu-id="06513-242">9</span></span> |<span data-ttu-id="06513-243">Número máximo de reintentos si se produce un error en la solicitud porque Cosmos DB aplicó la limitación de velocidad en el cliente.</span><span class="sxs-lookup"><span data-stu-id="06513-243">The maximum number of retries if the request fails because Cosmos DB applied rate limiting on the client.</span></span> |
| <span data-ttu-id="06513-244">MaxRetryWaitTimeInSeconds</span><span class="sxs-lookup"><span data-stu-id="06513-244">MaxRetryWaitTimeInSeconds</span></span> |<span data-ttu-id="06513-245">30</span><span class="sxs-lookup"><span data-stu-id="06513-245">30</span></span> |<span data-ttu-id="06513-246">El tiempo de reintentos máximo en segundos.</span><span class="sxs-lookup"><span data-stu-id="06513-246">The maximum retry time in seconds.</span></span> |

### <a name="example"></a><span data-ttu-id="06513-247">Ejemplo</span><span class="sxs-lookup"><span data-stu-id="06513-247">Example</span></span>
```csharp
DocumentClient client = new DocumentClient(new Uri(endpoint), authKey); ;
var options = client.ConnectionPolicy.RetryOptions;
options.MaxRetryAttemptsOnThrottledRequests = 5;
options.MaxRetryWaitTimeInSeconds = 15;
```

### <a name="telemetry"></a><span data-ttu-id="06513-248">Telemetría</span><span class="sxs-lookup"><span data-stu-id="06513-248">Telemetry</span></span>
<span data-ttu-id="06513-249">Los reintentos se registran como mensajes de seguimiento no estructurados a través de .NET **TraceSource**.</span><span class="sxs-lookup"><span data-stu-id="06513-249">Retry attempts are logged as unstructured trace messages through a .NET **TraceSource**.</span></span> <span data-ttu-id="06513-250">Debe configurar un **TraceListener** para capturar los eventos y escribirlos en un registro de destino adecuado.</span><span class="sxs-lookup"><span data-stu-id="06513-250">You must configure a **TraceListener** to capture the events and write them to a suitable destination log.</span></span>

<span data-ttu-id="06513-251">Por ejemplo, si agrega lo siguiente al archivo App.config, se generarán seguimientos en un archivo de texto en la misma ubicación que el archivo ejecutable:</span><span class="sxs-lookup"><span data-stu-id="06513-251">For example, if you add the following to your App.config file, traces will be generated in a text file in the same location as the executable:</span></span>

```xml
<configuration>
  <system.diagnostics>
    <switches>
      <add name="SourceSwitch" value="Verbose"/>
    </switches>
    <sources>
      <source name="DocDBTrace" switchName="SourceSwitch" switchType="System.Diagnostics.SourceSwitch" >
        <listeners>
          <add name="MyTextListener" type="System.Diagnostics.TextWriterTraceListener" traceOutputOptions="DateTime,ProcessId,ThreadId" initializeData="CosmosDBTrace.txt"></add>
        </listeners>
      </source>
    </sources>
  </system.diagnostics>
</configuration>
```

## <a name="event-hubs"></a><span data-ttu-id="06513-252">Event Hubs</span><span class="sxs-lookup"><span data-stu-id="06513-252">Event Hubs</span></span>

<span data-ttu-id="06513-253">Azure Event Hubs es un servicio de ingestión de datos de telemetría a hiperescala que recopila, transforma y almacena millones de eventos.</span><span class="sxs-lookup"><span data-stu-id="06513-253">Azure Event Hubs is a hyper-scale telemetry ingestion service that collects, transforms, and stores millions of events.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="06513-254">Mecanismo de reintento</span><span class="sxs-lookup"><span data-stu-id="06513-254">Retry mechanism</span></span>
<span data-ttu-id="06513-255">El comportamiento de reintentos de la biblioteca de cliente de Azure Event Hubs se controla mediante la propiedad `RetryPolicy` de la clase `EventHubClient`.</span><span class="sxs-lookup"><span data-stu-id="06513-255">Retry behavior in the Azure Event Hubs Client Library is controlled by the `RetryPolicy` property on the `EventHubClient` class.</span></span> <span data-ttu-id="06513-256">La directiva predeterminada realiza reintentos con retroceso exponencial cuando Azure Event Hubs devuelve una excepción transitoria `EventHubsException` o `OperationCanceledException`.</span><span class="sxs-lookup"><span data-stu-id="06513-256">The default policy retries with exponential backoff when Azure Event Hub returns a transient `EventHubsException` or an `OperationCanceledException`.</span></span>

### <a name="example"></a><span data-ttu-id="06513-257">Ejemplo</span><span class="sxs-lookup"><span data-stu-id="06513-257">Example</span></span>
```csharp
EventHubClient client = EventHubClient.CreateFromConnectionString("[event_hub_connection_string]");
client.RetryPolicy = RetryPolicy.Default;
```

### <a name="more-information"></a><span data-ttu-id="06513-258">Más información</span><span class="sxs-lookup"><span data-stu-id="06513-258">More information</span></span>
[<span data-ttu-id="06513-259">Biblioteca de cliente .NET estándar para Azure Event Hubs</span><span class="sxs-lookup"><span data-stu-id="06513-259"> .NET Standard client library for Azure Event Hubs</span></span>](https://github.com/Azure/azure-event-hubs-dotnet)

## <a name="azure-redis-cache"></a><span data-ttu-id="06513-260">Azure Redis Cache</span><span class="sxs-lookup"><span data-stu-id="06513-260">Azure Redis Cache</span></span>
<span data-ttu-id="06513-261">Azure Redis Cache es un servicio de caché de acceso rápido a datos y de baja latencia basado en Redis Cache de código abierto popular.</span><span class="sxs-lookup"><span data-stu-id="06513-261">Azure Redis Cache is a fast data access and low latency cache service based on the popular open source Redis Cache.</span></span> <span data-ttu-id="06513-262">Es segura, está administrada por Microsoft y es accesible desde cualquier aplicación en Azure.</span><span class="sxs-lookup"><span data-stu-id="06513-262">It is secure, managed by Microsoft, and is accessible from any application in Azure.</span></span>

<span data-ttu-id="06513-263">Las instrucciones de esta sección se basan en el uso del cliente StackExchange.Redis para tener acceso a la memoria caché.</span><span class="sxs-lookup"><span data-stu-id="06513-263">The guidance in this section is based on using the StackExchange.Redis client to access the cache.</span></span> <span data-ttu-id="06513-264">Puede encontrar una lista de otros clientes adecuados en el [sitio web de Redis](http://redis.io/clients), y estos pueden tener mecanismos de reintento diferentes.</span><span class="sxs-lookup"><span data-stu-id="06513-264">A list of other suitable clients can be found on the [Redis website](http://redis.io/clients), and these may have different retry mechanisms.</span></span>

<span data-ttu-id="06513-265">Tenga en cuenta que el cliente StackExchange.Redis usa multiplexación a través de una sola conexión.</span><span class="sxs-lookup"><span data-stu-id="06513-265">Note that the StackExchange.Redis client uses multiplexing through a single connection.</span></span> <span data-ttu-id="06513-266">El uso recomendado es crear una instancia del cliente al iniciar la aplicación y usar esta instancia para todas las operaciones en la memoria caché.</span><span class="sxs-lookup"><span data-stu-id="06513-266">The recommended usage is to create an instance of the client at application startup and use this instance for all operations against the cache.</span></span> <span data-ttu-id="06513-267">Por este motivo, la conexión a la memoria caché se realiza solo una vez y todas las instrucciones de esta sección están relacionadas con la directiva de reintentos de esta conexión inicial y no para cada operación que tiene acceso a la memoria caché.</span><span class="sxs-lookup"><span data-stu-id="06513-267">For this reason, the connection to the cache is made only once, and so all of the guidance in this section is related to the retry policy for this initial connection—and not for each operation that accesses the cache.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="06513-268">Mecanismo de reintento</span><span class="sxs-lookup"><span data-stu-id="06513-268">Retry mechanism</span></span>
<span data-ttu-id="06513-269">El cliente StackExchange.Redis usa una clase de administrador de conexiones que se configura mediante un conjunto de opciones, entre ellas:</span><span class="sxs-lookup"><span data-stu-id="06513-269">The StackExchange.Redis client uses a connection manager class that is configured through a set of options, incuding:</span></span>

- <span data-ttu-id="06513-270">**ConnectRetry**.</span><span class="sxs-lookup"><span data-stu-id="06513-270">**ConnectRetry**.</span></span> <span data-ttu-id="06513-271">Número de veces que se volverá a intentar una conexión con error a la memoria caché.</span><span class="sxs-lookup"><span data-stu-id="06513-271">The number of times a failed connection to the cache will be retried.</span></span>
- <span data-ttu-id="06513-272">**ReconnectRetryPolicy**.</span><span class="sxs-lookup"><span data-stu-id="06513-272">**ReconnectRetryPolicy**.</span></span> <span data-ttu-id="06513-273">Estrategia de reintentos que se usará.</span><span class="sxs-lookup"><span data-stu-id="06513-273">The retry strategy to use.</span></span>
- <span data-ttu-id="06513-274">**ConnectTimeout**.</span><span class="sxs-lookup"><span data-stu-id="06513-274">**ConnectTimeout**.</span></span> <span data-ttu-id="06513-275">Tiempo de espera máximo en milisegundos.</span><span class="sxs-lookup"><span data-stu-id="06513-275">The maximum waiting time in milliseconds.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="06513-276">Configuración de directivas</span><span class="sxs-lookup"><span data-stu-id="06513-276">Policy configuration</span></span>
<span data-ttu-id="06513-277">Las directivas de reintento se configuran mediante programación, estableciendo las opciones para el cliente antes de conectarse a la memoria caché.</span><span class="sxs-lookup"><span data-stu-id="06513-277">Retry policies are configured programmatically by setting the options for the client before connecting to the cache.</span></span> <span data-ttu-id="06513-278">Esto puede hacerse mediante la creación de una instancia de la clase **ConfigurationOptions**, rellenando sus propiedades y pasándola al método **Conectar**.</span><span class="sxs-lookup"><span data-stu-id="06513-278">This can be done by creating an instance of the **ConfigurationOptions** class, populating its properties, and passing it to the **Connect** method.</span></span>

<span data-ttu-id="06513-279">Las clases integradas admiten retrasos lineales (constantes) y retroceso exponencial con intervalos de reintento aleatorios.</span><span class="sxs-lookup"><span data-stu-id="06513-279">The built-in classes support linear (constant) delay and exponential backoff with randomized retry intervals.</span></span> <span data-ttu-id="06513-280">También puede implementar la interfaz **IReconnectRetryPolicy** para crear una directiva de reintentos personalizada.</span><span class="sxs-lookup"><span data-stu-id="06513-280">You can also create a custom retry policy by implementing the **IReconnectRetryPolicy** interface.</span></span>

<span data-ttu-id="06513-281">En el ejemplo siguiente se configura una estrategia de reintentos con retroceso exponencial.</span><span class="sxs-lookup"><span data-stu-id="06513-281">The following example configures a retry strategy using exponential backoff.</span></span>

```csharp
var deltaBackOffInMilliseconds = TimeSpan.FromSeconds(5).Milliseconds;
var maxDeltaBackOffInMilliseconds = TimeSpan.FromSeconds(20).Milliseconds;
var options = new ConfigurationOptions
{
    EndPoints = {"localhost"},
    ConnectRetry = 3,
    ReconnectRetryPolicy = new ExponentialRetry(deltaBackOffInMilliseconds, maxDeltaBackOffInMilliseconds),
    ConnectTimeout = 2000
};
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

<span data-ttu-id="06513-282">Como alternativa, puede especificar las opciones como una cadena y pasar esto al método **Conectar** .</span><span class="sxs-lookup"><span data-stu-id="06513-282">Alternatively, you can specify the options as a string, and pass this to the **Connect** method.</span></span> <span data-ttu-id="06513-283">Tenga en cuenta que la propiedad **ReconnectRetryPolicy** no se puede establecer de esta manera, sino solo mediante código.</span><span class="sxs-lookup"><span data-stu-id="06513-283">Note that the **ReconnectRetryPolicy** property cannot be set this way, only through code.</span></span>

```csharp
var options = "localhost,connectRetry=3,connectTimeout=2000";
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

<span data-ttu-id="06513-284">También es posible especificar las opciones directamente al conectarse a la memoria caché.</span><span class="sxs-lookup"><span data-stu-id="06513-284">You can also specify options directly when you connect to the cache.</span></span>

```csharp
var conn = ConnectionMultiplexer.Connect("redis0:6380,redis1:6380,connectRetry=3");
```

<span data-ttu-id="06513-285">Para más información, consulte [Configuración de StackExchange.Redis](https://stackexchange.github.io/StackExchange.Redis/Configuration) en la documentación de StackExchange.Redis.</span><span class="sxs-lookup"><span data-stu-id="06513-285">For more information, see [Stack Exchange Redis Configuration](https://stackexchange.github.io/StackExchange.Redis/Configuration) in the StackExchange.Redis documentation.</span></span>

<span data-ttu-id="06513-286">La siguiente tabla muestra la configuración predeterminada de la directiva de reintento integrada.</span><span class="sxs-lookup"><span data-stu-id="06513-286">The following table shows the default settings for the built-in retry policy.</span></span>

| <span data-ttu-id="06513-287">**Contexto**</span><span class="sxs-lookup"><span data-stu-id="06513-287">**Context**</span></span> | <span data-ttu-id="06513-288">**Configuración**</span><span class="sxs-lookup"><span data-stu-id="06513-288">**Setting**</span></span> | <span data-ttu-id="06513-289">**Valor predeterminado**</span><span class="sxs-lookup"><span data-stu-id="06513-289">**Default value**</span></span><br /><span data-ttu-id="06513-290">(v 1.2.2)</span><span class="sxs-lookup"><span data-stu-id="06513-290">(v 1.2.2)</span></span> | <span data-ttu-id="06513-291">**Significado**</span><span class="sxs-lookup"><span data-stu-id="06513-291">**Meaning**</span></span> |
| --- | --- | --- | --- |
| <span data-ttu-id="06513-292">ConfigurationOptions</span><span class="sxs-lookup"><span data-stu-id="06513-292">ConfigurationOptions</span></span> |<span data-ttu-id="06513-293">ConnectRetry</span><span class="sxs-lookup"><span data-stu-id="06513-293">ConnectRetry</span></span><br /><br /><span data-ttu-id="06513-294">ConnectTimeout</span><span class="sxs-lookup"><span data-stu-id="06513-294">ConnectTimeout</span></span><br /><br /><span data-ttu-id="06513-295">SyncTimeout</span><span class="sxs-lookup"><span data-stu-id="06513-295">SyncTimeout</span></span><br /><br /><span data-ttu-id="06513-296">ReconnectRetryPolicy</span><span class="sxs-lookup"><span data-stu-id="06513-296">ReconnectRetryPolicy</span></span> |<span data-ttu-id="06513-297">3</span><span class="sxs-lookup"><span data-stu-id="06513-297">3</span></span><br /><br /><span data-ttu-id="06513-298">Máximo de 5000 ms más SyncTimeout</span><span class="sxs-lookup"><span data-stu-id="06513-298">Maximum 5000 ms plus SyncTimeout</span></span><br /><span data-ttu-id="06513-299">1000</span><span class="sxs-lookup"><span data-stu-id="06513-299">1000</span></span><br /><br /><span data-ttu-id="06513-300">LinearRetry 5000 ms</span><span class="sxs-lookup"><span data-stu-id="06513-300">LinearRetry 5000 ms</span></span> |<span data-ttu-id="06513-301">El número de veces que se repiten los intentos de conexión durante la operación de conexión inicial.</span><span class="sxs-lookup"><span data-stu-id="06513-301">The number of times to repeat connect attempts during the initial connection operation.</span></span><br /><span data-ttu-id="06513-302">Tiempo de espera (ms) para conectar las operaciones.</span><span class="sxs-lookup"><span data-stu-id="06513-302">Timeout (ms) for connect operations.</span></span> <span data-ttu-id="06513-303">No un retraso entre reintentos.</span><span class="sxs-lookup"><span data-stu-id="06513-303">Not a delay between retry attempts.</span></span><br /><span data-ttu-id="06513-304">Tiempo (ms) para permitir las operaciones sincrónicas.</span><span class="sxs-lookup"><span data-stu-id="06513-304">Time (ms) to allow for synchronous operations.</span></span><br /><br /><span data-ttu-id="06513-305">Reintentar cada 5000 ms.</span><span class="sxs-lookup"><span data-stu-id="06513-305">Retry every 5000 ms.</span></span>|

> [!NOTE]
> <span data-ttu-id="06513-306">Para las operaciones sincrónicas, `SyncTimeout` puede agregar latencia de extremo a extremo, pero si se establece un valor demasiado bajo, puede provocar tiempos de espera excesivos.</span><span class="sxs-lookup"><span data-stu-id="06513-306">For synchronous operations, `SyncTimeout` can add to the end-to-end latency, but setting the value too low can cause excessive timeouts.</span></span> <span data-ttu-id="06513-307">Consulte [Solución de problemas de Azure Redis Cache][redis-cache-troubleshoot].</span><span class="sxs-lookup"><span data-stu-id="06513-307">See [How to troubleshoot Azure Redis Cache][redis-cache-troubleshoot].</span></span> <span data-ttu-id="06513-308">Por lo general, evite el uso de las operaciones sincrónicas y use operaciones asincrónicas en su lugar.</span><span class="sxs-lookup"><span data-stu-id="06513-308">In general, avoid using synchronous operations, and use asynchronous operations instead.</span></span> <span data-ttu-id="06513-309">Para obtener más información, consulte [Canalizaciones y multiplexores](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/PipelinesMultiplexers.md).</span><span class="sxs-lookup"><span data-stu-id="06513-309">For more information see [Pipelines and Multiplexers](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/PipelinesMultiplexers.md).</span></span>
>
>

### <a name="retry-usage-guidance"></a><span data-ttu-id="06513-310">Instrucciones de uso del reintento</span><span class="sxs-lookup"><span data-stu-id="06513-310">Retry usage guidance</span></span>
<span data-ttu-id="06513-311">Tenga en cuenta las siguientes directrices cuando use Azure Redis Cache:</span><span class="sxs-lookup"><span data-stu-id="06513-311">Consider the following guidelines when using Azure Redis Cache:</span></span>

* <span data-ttu-id="06513-312">El cliente Redis StackExchange administra sus propios reintentos, pero solo al establecer una conexión a la memoria caché cuando la aplicación se inicia por primera vez.</span><span class="sxs-lookup"><span data-stu-id="06513-312">The StackExchange Redis client manages its own retries, but only when establishing a connection to the cache when the application first starts.</span></span> <span data-ttu-id="06513-313">Puede configurar el tiempo de espera de conexión, el número de reintentos y el tiempo entre reintentos para establecer esta conexión, pero la directiva de reintentos no se aplica a las operaciones en la memoria caché.</span><span class="sxs-lookup"><span data-stu-id="06513-313">You can configure the connection timeout, the number of retry attempts, and the time between retries to establish this connection, but the retry policy does not apply to operations against the cache.</span></span>
* <span data-ttu-id="06513-314">En lugar de usar un gran número de reintentos, considere la posibilidad de recurrir a obtener acceso a un origen de datos original en su lugar.</span><span class="sxs-lookup"><span data-stu-id="06513-314">Instead of using a large number of retry attempts, consider falling back by accessing the original data source instead.</span></span>

### <a name="telemetry"></a><span data-ttu-id="06513-315">Telemetría</span><span class="sxs-lookup"><span data-stu-id="06513-315">Telemetry</span></span>
<span data-ttu-id="06513-316">Puede recopilar información acerca de las conexiones (pero no otras operaciones) con un **TextWriter**.</span><span class="sxs-lookup"><span data-stu-id="06513-316">You can collect information about connections (but not other operations) using a **TextWriter**.</span></span>

```csharp
var writer = new StringWriter();
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

<span data-ttu-id="06513-317">A continuación, se muestra un ejemplo del resultado que este genera.</span><span class="sxs-lookup"><span data-stu-id="06513-317">An example of the output this generates is shown below.</span></span>

```text
localhost:6379,connectTimeout=2000,connectRetry=3
1 unique nodes specified
Requesting tie-break from localhost:6379 > __Booksleeve_TieBreak...
Allowing endpoints 00:00:02 to respond...
localhost:6379 faulted: SocketFailure on PING
localhost:6379 failed to nominate (Faulted)
> UnableToResolvePhysicalConnection on GET
No masters detected
localhost:6379: Standalone v2.0.0, master; keep-alive: 00:01:00; int: Connecting; sub: Connecting; not in use: DidNotRespond
localhost:6379: int ops=0, qu=0, qs=0, qc=1, wr=0, sync=1, socks=2; sub ops=0, qu=0, qs=0, qc=0, wr=0, socks=2
Circular op-count snapshot; int: 0 (0.00 ops/s; spans 10s); sub: 0 (0.00 ops/s; spans 10s)
Sync timeouts: 0; fire and forget: 0; last heartbeat: -1s ago
resetting failing connections to retry...
retrying; attempts left: 2...
...
```

### <a name="examples"></a><span data-ttu-id="06513-318">Ejemplos</span><span class="sxs-lookup"><span data-stu-id="06513-318">Examples</span></span>
<span data-ttu-id="06513-319">En el ejemplo de código siguiente se configura un retraso constante (lineal) entre los reintentos al inicializar el cliente StackExchange.Redis.</span><span class="sxs-lookup"><span data-stu-id="06513-319">The following code example configures a constant (linear) delay between retries when initializing the StackExchange.Redis client.</span></span> <span data-ttu-id="06513-320">Este ejemplo muestra cómo establecer la configuración mediante una instancia de **ConfigurationOptions**.</span><span class="sxs-lookup"><span data-stu-id="06513-320">This example shows how to set the configuration using a **ConfigurationOptions** instance.</span></span>

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using StackExchange.Redis;

namespace RetryCodeSamples
{
    class CacheRedisCodeSamples
    {
        public async static Task Samples()
        {
            var writer = new StringWriter();
            {
                try
                {
                    var retryTimeInMilliseconds = TimeSpan.FromSeconds(4).Milliseconds; // delay between retries

                    // Using object-based configuration.
                    var options = new ConfigurationOptions
                                        {
                                            EndPoints = { "localhost" },
                                            ConnectRetry = 3,
                                            ReconnectRetryPolicy = new LinearRetry(retryTimeInMilliseconds)
                                        };
                    ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);

                    // Store a reference to the multiplexer for use in the application.
                }
                catch
                {
                    Console.WriteLine(writer.ToString());
                    throw;
                }
            }
        }
    }
}
```

<span data-ttu-id="06513-321">En este ejemplo se especifican las opciones como una cadena para establecer la configuración.</span><span class="sxs-lookup"><span data-stu-id="06513-321">The next example sets the configuration by specifying the options as a string.</span></span> <span data-ttu-id="06513-322">El tiempo de espera de conexión es el período de tiempo máximo que se esperará a que se realice la conexión con la memoria caché; no es el retraso entre los reintentos.</span><span class="sxs-lookup"><span data-stu-id="06513-322">The connection timeout is the maximum period of time to wait for a connection to the cache, not the delay between retry attempts.</span></span> <span data-ttu-id="06513-323">Tenga en cuenta que la propiedad **ReconnectRetryPolicy** solo puede establecerse mediante código.</span><span class="sxs-lookup"><span data-stu-id="06513-323">Note that the **ReconnectRetryPolicy** property can only be set by code.</span></span>

```csharp
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using StackExchange.Redis;

namespace RetryCodeSamples
{
    class CacheRedisCodeSamples
    {
        public async static Task Samples()
        {
            var writer = new StringWriter();
            {
                try
                {
                    // Using string-based configuration.
                    var options = "localhost,connectRetry=3,connectTimeout=2000";
                    ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);

                    // Store a reference to the multiplexer for use in the application.
                }
                catch
                {
                    Console.WriteLine(writer.ToString());
                    throw;
                }
            }
        }
    }
}
```

<span data-ttu-id="06513-324">Para obtener más ejemplos, consulte [Configuración](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/Configuration.md#configuration) en el sitio web del proyecto.</span><span class="sxs-lookup"><span data-stu-id="06513-324">For more examples, see [Configuration](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/Configuration.md#configuration) on the project website.</span></span>

### <a name="more-information"></a><span data-ttu-id="06513-325">Más información</span><span class="sxs-lookup"><span data-stu-id="06513-325">More information</span></span>
* [<span data-ttu-id="06513-326">Sitio web de Redis</span><span class="sxs-lookup"><span data-stu-id="06513-326">Redis website</span></span>](http://redis.io/)

## <a name="azure-search"></a><span data-ttu-id="06513-327">Azure Search</span><span class="sxs-lookup"><span data-stu-id="06513-327">Azure Search</span></span>
<span data-ttu-id="06513-328">Azure Search puede usarse para agregar capacidades de búsqueda eficaces y sofisticadas a un sitio web o aplicación, ajustar de manera rápida y fácil los resultados de la búsqueda y construir modelos de clasificación enriquecidos y optimizados.</span><span class="sxs-lookup"><span data-stu-id="06513-328">Azure Search can be used to add powerful and sophisticated search capabilities to a website or application, quickly and easily tune search results, and construct rich and fine-tuned ranking models.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="06513-329">Mecanismo de reintento</span><span class="sxs-lookup"><span data-stu-id="06513-329">Retry mechanism</span></span>
<span data-ttu-id="06513-330">El comportamiento de reintento en el SDK de Azure Search se controla mediante el método `SetRetryPolicy` en las clases [SearchServiceClient] y [SearchIndexClient].</span><span class="sxs-lookup"><span data-stu-id="06513-330">Retry behavior in the Azure Search SDK is controlled by the `SetRetryPolicy` method on the [SearchServiceClient] and [SearchIndexClient] classes.</span></span> <span data-ttu-id="06513-331">Los reintentos de directiva predeterminada con interrupción exponencial cuando Azure Search devuelve una respuesta 5xx o 408 (tiempo de espera de solicitud).</span><span class="sxs-lookup"><span data-stu-id="06513-331">The default policy retries with exponential backoff when Azure Search returns a 5xx or 408 (Request Timeout) response.</span></span>

### <a name="telemetry"></a><span data-ttu-id="06513-332">Telemetría</span><span class="sxs-lookup"><span data-stu-id="06513-332">Telemetry</span></span>
<span data-ttu-id="06513-333">Realice un seguimiento con ETW o mediante el registro de un proveedor de seguimiento personalizado.</span><span class="sxs-lookup"><span data-stu-id="06513-333">Trace with ETW or by registering a custom trace provider.</span></span> <span data-ttu-id="06513-334">Para más información, consulte la [documentación de AutoRest][autorest].</span><span class="sxs-lookup"><span data-stu-id="06513-334">For more information, see the [AutoRest documentation][autorest].</span></span>

## <a name="service-bus"></a><span data-ttu-id="06513-335">Azure Service Bus</span><span class="sxs-lookup"><span data-stu-id="06513-335">Service Bus</span></span>
<span data-ttu-id="06513-336">Service Bus es una plataforma de mensajería de nube que proporciona el intercambio de mensajes de acoplamiento flexible con mejor escala y resistencia para los componentes de una aplicación, ya esté hospedada en la nube o localmente.</span><span class="sxs-lookup"><span data-stu-id="06513-336">Service Bus is a cloud messaging platform that provides loosely coupled message exchange with improved scale and resiliency for components of an application, whether hosted in the cloud or on-premises.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="06513-337">Mecanismo de reintento</span><span class="sxs-lookup"><span data-stu-id="06513-337">Retry mechanism</span></span>
<span data-ttu-id="06513-338">Service Bus implementa reintentos mediante implementaciones de la clase base [RetryPolicy](http://msdn.microsoft.com/library/microsoft.servicebus.retrypolicy.aspx) .</span><span class="sxs-lookup"><span data-stu-id="06513-338">Service Bus implements retries using implementations of the [RetryPolicy](http://msdn.microsoft.com/library/microsoft.servicebus.retrypolicy.aspx) base class.</span></span> <span data-ttu-id="06513-339">Todos los clientes de Service Bus exponen una propiedad **RetryPolicy** que puede establecerse en una de las implementaciones de la clase base **RetryPolicy**.</span><span class="sxs-lookup"><span data-stu-id="06513-339">All of the Service Bus clients expose a **RetryPolicy** property that can be set to one of the implementations of the **RetryPolicy** base class.</span></span> <span data-ttu-id="06513-340">Las implementaciones integradas son:</span><span class="sxs-lookup"><span data-stu-id="06513-340">The built-in implementations are:</span></span>

* <span data-ttu-id="06513-341">La [clase RetryExponential](http://msdn.microsoft.com/library/microsoft.servicebus.retryexponential.aspx).</span><span class="sxs-lookup"><span data-stu-id="06513-341">The [RetryExponential Class](http://msdn.microsoft.com/library/microsoft.servicebus.retryexponential.aspx).</span></span> <span data-ttu-id="06513-342">Esto expone las propiedades que controlan el intervalo de interrupción, el número de reintentos y la propiedad **TerminationTimeBuffer** que se utiliza para limitar el tiempo total para que se complete la operación.</span><span class="sxs-lookup"><span data-stu-id="06513-342">This exposes properties that control the back-off interval, the retry count, and the **TerminationTimeBuffer** property that is used to limit the total time for the operation to complete.</span></span>
* <span data-ttu-id="06513-343">La [clase NoRetry](http://msdn.microsoft.com/library/microsoft.servicebus.noretry.aspx).</span><span class="sxs-lookup"><span data-stu-id="06513-343">The [NoRetry Class](http://msdn.microsoft.com/library/microsoft.servicebus.noretry.aspx).</span></span> <span data-ttu-id="06513-344">Se utiliza cuando los reintentos en el nivel de la API de Service Bus no son necesarios, como cuando otro proceso administra los reintentos como parte de una operación en lotes o de múltiples pasos.</span><span class="sxs-lookup"><span data-stu-id="06513-344">This is used when retries at the Service Bus API level are not required, such as when retries are managed by another process as part of a batch or multiple step operation.</span></span>

<span data-ttu-id="06513-345">Las acciones de Service Bus pueden devolver una amplia gama de excepciones, como se muestra en [Excepciones de mensajería de Service Bus](/azure/service-bus-messaging/service-bus-messaging-exceptions).</span><span class="sxs-lookup"><span data-stu-id="06513-345">Service Bus actions can return a range of exceptions, as listed in [Service Bus messaging exceptions](/azure/service-bus-messaging/service-bus-messaging-exceptions).</span></span> <span data-ttu-id="06513-346">La lista proporciona información sobre si estas indican que la operación de reintento es adecuada.</span><span class="sxs-lookup"><span data-stu-id="06513-346">The list provides information about which if these indicate that retrying the operation is appropriate.</span></span> <span data-ttu-id="06513-347">Por ejemplo, un **ServerBusyException** indica que el cliente debe esperar durante un período de tiempo y, a continuación, volver a intentar la operación.</span><span class="sxs-lookup"><span data-stu-id="06513-347">For example, a **ServerBusyException** indicates that the client should wait for a period of time, then retry the operation.</span></span> <span data-ttu-id="06513-348">La aparición de una **ServerBusyException** también hace que Service Bus cambie a un modo diferente, en el que se agrega un retraso adicional de 10 segundos a los retrasos de reintento calculados.</span><span class="sxs-lookup"><span data-stu-id="06513-348">The occurrence of a **ServerBusyException** also causes Service Bus to switch to a different mode, in which an extra 10-second delay is added to the computed retry delays.</span></span> <span data-ttu-id="06513-349">Este modo se restablece tras un breve período.</span><span class="sxs-lookup"><span data-stu-id="06513-349">This mode is reset after a short period.</span></span>

<span data-ttu-id="06513-350">Las excepciones devueltas de Service Bus exponen la propiedad **IsTransient** propiedad que indica si el cliente debería reintentar la operación.</span><span class="sxs-lookup"><span data-stu-id="06513-350">The exceptions returned from Service Bus expose the **IsTransient** property that indicates if the client should retry the operation.</span></span> <span data-ttu-id="06513-351">La directiva **RetryExponential** se basa en la propiedad **IsTransient** de la clase **MessagingException**, que es la clase base para todas las excepciones de Service Bus.</span><span class="sxs-lookup"><span data-stu-id="06513-351">The built-in **RetryExponential** policy relies on the **IsTransient** property in the **MessagingException** class, which is the base class for all Service Bus exceptions.</span></span> <span data-ttu-id="06513-352">Si crea implementaciones personalizadas de la clase de base **RetryPolicy**, podría usar una combinación del tipo de excepción y la propiedad **IsTransient** para proporcionar un control más fino sobre las acciones de reintento.</span><span class="sxs-lookup"><span data-stu-id="06513-352">If you create custom implementations of the **RetryPolicy** base class you could use a combination of the exception type and the **IsTransient** property to provide more fine-grained control over retry actions.</span></span> <span data-ttu-id="06513-353">Por ejemplo, se podría detectar una **QuotaExceededException** y tomar medidas para vaciar la cola antes de volver a intentar enviar un mensaje.</span><span class="sxs-lookup"><span data-stu-id="06513-353">For example, you could detect a **QuotaExceededException** and take action to drain the queue before retrying sending a message to it.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="06513-354">Configuración de directivas</span><span class="sxs-lookup"><span data-stu-id="06513-354">Policy configuration</span></span>
<span data-ttu-id="06513-355">Las directivas de reintento se establecen mediante programación y se pueden establecer como una directiva predeterminada para un **NamespaceManager** y una **MessagingFactory**, o por separado para cada cliente de mensajería.</span><span class="sxs-lookup"><span data-stu-id="06513-355">Retry policies are set programmatically, and can be set as a default policy for a **NamespaceManager** and for a **MessagingFactory**, or individually for each messaging client.</span></span> <span data-ttu-id="06513-356">Para establecer la directiva de reintentos predeterminada para una sesión de mensajería, establezca **RetryPolicy** de **NamespaceManager**.</span><span class="sxs-lookup"><span data-stu-id="06513-356">To set the default retry policy for a messaging session you set the **RetryPolicy** of the **NamespaceManager**.</span></span>

```csharp
namespaceManager.Settings.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                                maxBackoff: TimeSpan.FromSeconds(30),
                                                                maxRetryCount: 3);
```

<span data-ttu-id="06513-357">Para establecer la directiva de reintentos predeterminada para todos los clientes creada a partir de una fábrica de mensajería, establezca **RetryPolicy** de **MessagingFactory**.</span><span class="sxs-lookup"><span data-stu-id="06513-357">To set the default retry policy for all clients created from a messaging factory, you set the **RetryPolicy** of the **MessagingFactory**.</span></span>

```csharp
messagingFactory.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                    maxBackoff: TimeSpan.FromSeconds(30),
                                                    maxRetryCount: 3);
```

<span data-ttu-id="06513-358">Para establecer la directiva de reintentos para un cliente de mensajería o para invalidar su directiva predeterminada, establezca su propiedad **RetryPolicy** mediante una instancia de la clase de directiva requerida:</span><span class="sxs-lookup"><span data-stu-id="06513-358">To set the retry policy for a messaging client, or to override its default policy, you set its **RetryPolicy** property using an instance of the required policy class:</span></span>

```csharp
client.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                            maxBackoff: TimeSpan.FromSeconds(30),
                                            maxRetryCount: 3);
```

<span data-ttu-id="06513-359">No se puede establecer la directiva de reintentos en el nivel de operación individual.</span><span class="sxs-lookup"><span data-stu-id="06513-359">The retry policy cannot be set at the individual operation level.</span></span> <span data-ttu-id="06513-360">Se aplica a todas las operaciones para el cliente de mensajería.</span><span class="sxs-lookup"><span data-stu-id="06513-360">It applies to all operations for the messaging client.</span></span>
<span data-ttu-id="06513-361">La siguiente tabla muestra la configuración predeterminada de la directiva de reintento integrada.</span><span class="sxs-lookup"><span data-stu-id="06513-361">The following table shows the default settings for the built-in retry policy.</span></span>

| <span data-ttu-id="06513-362">Configuración</span><span class="sxs-lookup"><span data-stu-id="06513-362">Setting</span></span> | <span data-ttu-id="06513-363">Valor predeterminado</span><span class="sxs-lookup"><span data-stu-id="06513-363">Default value</span></span> | <span data-ttu-id="06513-364">Significado</span><span class="sxs-lookup"><span data-stu-id="06513-364">Meaning</span></span> |
|---------|---------------|---------|
| <span data-ttu-id="06513-365">Directiva</span><span class="sxs-lookup"><span data-stu-id="06513-365">Policy</span></span> | <span data-ttu-id="06513-366">Exponencial</span><span class="sxs-lookup"><span data-stu-id="06513-366">Exponential</span></span> | <span data-ttu-id="06513-367">Retroceso exponencial.</span><span class="sxs-lookup"><span data-stu-id="06513-367">Exponential back-off.</span></span> |
| <span data-ttu-id="06513-368">MinimalBackoff</span><span class="sxs-lookup"><span data-stu-id="06513-368">MinimalBackoff</span></span> | <span data-ttu-id="06513-369">0</span><span class="sxs-lookup"><span data-stu-id="06513-369">0</span></span> | <span data-ttu-id="06513-370">El intervalo de retroceso mínimo.</span><span class="sxs-lookup"><span data-stu-id="06513-370">Minimum back-off interval.</span></span> <span data-ttu-id="06513-371">Se agrega al intervalo de reintento calculado a partir de deltaBackoff.</span><span class="sxs-lookup"><span data-stu-id="06513-371">This is added to the retry interval computed from deltaBackoff.</span></span> |
| <span data-ttu-id="06513-372">MaximumBackoff</span><span class="sxs-lookup"><span data-stu-id="06513-372">MaximumBackoff</span></span> | <span data-ttu-id="06513-373">30 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-373">30 seconds</span></span> | <span data-ttu-id="06513-374">El intervalo de retroceso máximo.</span><span class="sxs-lookup"><span data-stu-id="06513-374">Maximum back-off interval.</span></span> <span data-ttu-id="06513-375">MaximumBackoff se usa si el intervalo de reintento calculado es mayor que MaxBackoff.</span><span class="sxs-lookup"><span data-stu-id="06513-375">MaximumBackoff is used if the computed retry interval is greater than MaxBackoff.</span></span> |
| <span data-ttu-id="06513-376">DeltaBackoff</span><span class="sxs-lookup"><span data-stu-id="06513-376">DeltaBackoff</span></span> | <span data-ttu-id="06513-377">3 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-377">3 seconds</span></span> | <span data-ttu-id="06513-378">Intervalo de espera entre reintentos.</span><span class="sxs-lookup"><span data-stu-id="06513-378">Back-off interval between retries.</span></span> <span data-ttu-id="06513-379">Se usará múltiplos de este período de tiempo para los intentos de reintentos posteriores.</span><span class="sxs-lookup"><span data-stu-id="06513-379">Multiples of this timespan will be used for subsequent retry attempts.</span></span> |
| <span data-ttu-id="06513-380">TimeBuffer</span><span class="sxs-lookup"><span data-stu-id="06513-380">TimeBuffer</span></span> | <span data-ttu-id="06513-381">5 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-381">5 seconds</span></span> | <span data-ttu-id="06513-382">El búfer del tiempo de finalización asociado con el reintento.</span><span class="sxs-lookup"><span data-stu-id="06513-382">The termination time buffer associated with the retry.</span></span> <span data-ttu-id="06513-383">Si el tiempo restante es menor que TimeBuffer, se abandonan los intentos de reintento.</span><span class="sxs-lookup"><span data-stu-id="06513-383">Retry attempts will be abandoned if the remaining time is less than TimeBuffer.</span></span> |
| <span data-ttu-id="06513-384">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="06513-384">MaxRetryCount</span></span> | <span data-ttu-id="06513-385">10</span><span class="sxs-lookup"><span data-stu-id="06513-385">10</span></span> | <span data-ttu-id="06513-386">El número máximo de reintentos.</span><span class="sxs-lookup"><span data-stu-id="06513-386">The maximum number of retries.</span></span> |
| <span data-ttu-id="06513-387">ServerBusyBaseSleepTime</span><span class="sxs-lookup"><span data-stu-id="06513-387">ServerBusyBaseSleepTime</span></span> | <span data-ttu-id="06513-388">10 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-388">10 seconds</span></span> | <span data-ttu-id="06513-389">Si la última excepción encontrada fue **ServerBusyException**, este valor se agregará al intervalo de reintento calculado.</span><span class="sxs-lookup"><span data-stu-id="06513-389">If the last exception encountered was **ServerBusyException**, this value will be added to the computed retry interval.</span></span> <span data-ttu-id="06513-390">No se puede cambiar este valor.</span><span class="sxs-lookup"><span data-stu-id="06513-390">This value cannot be changed.</span></span> |

### <a name="retry-usage-guidance"></a><span data-ttu-id="06513-391">Instrucciones de uso del reintento</span><span class="sxs-lookup"><span data-stu-id="06513-391">Retry usage guidance</span></span>
<span data-ttu-id="06513-392">Cuando se usa Service Bus, tenga en cuenta las siguientes directrices:</span><span class="sxs-lookup"><span data-stu-id="06513-392">Consider the following guidelines when using Service Bus:</span></span>

* <span data-ttu-id="06513-393">Al utilizar la implementación **RetryExponential** integrada, no implemente una operación de reserva, ya que la directiva reacciona a las excepciones de servidor ocupado y automáticamente cambia a un modo de reintentos adecuado.</span><span class="sxs-lookup"><span data-stu-id="06513-393">When using the built-in **RetryExponential** implementation, do not implement a fallback operation as the policy reacts to Server Busy exceptions and automatically switches to an appropriate retry mode.</span></span>
* <span data-ttu-id="06513-394">Service Bus admite una característica denominada espacios de nombres emparejados, que implementa la conmutación automática por error a una cola de copia de seguridad en un espacio de nombres independiente si se produce un error en la cola del espacio de nombres principal.</span><span class="sxs-lookup"><span data-stu-id="06513-394">Service Bus supports a feature called Paired Namespaces, which implements automatic failover to a backup queue in a separate namespace if the queue in the primary namespace fails.</span></span> <span data-ttu-id="06513-395">Los mensajes de la cola secundaria pueden enviarse a la cola principal cuando se recupera.</span><span class="sxs-lookup"><span data-stu-id="06513-395">Messages from the secondary queue can be sent back to the primary queue when it recovers.</span></span> <span data-ttu-id="06513-396">Esta característica ayuda a controlar los errores transitorios.</span><span class="sxs-lookup"><span data-stu-id="06513-396">This feature helps to address transient failures.</span></span> <span data-ttu-id="06513-397">Para obtener más información, consulte [Patrones de mensajería asincrónica y alta disponibilidad](http://msdn.microsoft.com/library/azure/dn292562.aspx).</span><span class="sxs-lookup"><span data-stu-id="06513-397">For more information, see [Asynchronous Messaging Patterns and High Availability](http://msdn.microsoft.com/library/azure/dn292562.aspx).</span></span>

<span data-ttu-id="06513-398">Considere la posibilidad de comenzar con la configuración siguiente para volver a intentar las operaciones.</span><span class="sxs-lookup"><span data-stu-id="06513-398">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="06513-399">Esta es la configuración de propósito general, y debe supervisar las operaciones y ajustar los valores para adaptarlos a su propio escenario.</span><span class="sxs-lookup"><span data-stu-id="06513-399">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="06513-400">Context</span><span class="sxs-lookup"><span data-stu-id="06513-400">Context</span></span> | <span data-ttu-id="06513-401">Latencia máxima de ejemplo</span><span class="sxs-lookup"><span data-stu-id="06513-401">Example maximum latency</span></span> | <span data-ttu-id="06513-402">Directiva de reintentos</span><span class="sxs-lookup"><span data-stu-id="06513-402">Retry policy</span></span> | <span data-ttu-id="06513-403">Settings</span><span class="sxs-lookup"><span data-stu-id="06513-403">Settings</span></span> | <span data-ttu-id="06513-404">Cómo funciona</span><span class="sxs-lookup"><span data-stu-id="06513-404">How it works</span></span> |
|---------|---------|---------|---------|---------|
| <span data-ttu-id="06513-405">Interactivo, interfaz de usuario o primer plano</span><span class="sxs-lookup"><span data-stu-id="06513-405">Interactive, UI, or foreground</span></span> | <span data-ttu-id="06513-406">2 segundos\*</span><span class="sxs-lookup"><span data-stu-id="06513-406">2 seconds\*</span></span>  | <span data-ttu-id="06513-407">Exponencial</span><span class="sxs-lookup"><span data-stu-id="06513-407">Exponential</span></span> | <span data-ttu-id="06513-408">MinimumBackoff = 0</span><span class="sxs-lookup"><span data-stu-id="06513-408">MinimumBackoff = 0</span></span> <br/> <span data-ttu-id="06513-409">MaximumBackoff = 30 s</span><span class="sxs-lookup"><span data-stu-id="06513-409">MaximumBackoff = 30 sec.</span></span> <br/> <span data-ttu-id="06513-410">DeltaBackoff = 300 ms</span><span class="sxs-lookup"><span data-stu-id="06513-410">DeltaBackoff = 300 msec.</span></span> <br/> <span data-ttu-id="06513-411">TimeBuffer = 300 ms</span><span class="sxs-lookup"><span data-stu-id="06513-411">TimeBuffer = 300 msec.</span></span> <br/> <span data-ttu-id="06513-412">MaxRetryCount = 2</span><span class="sxs-lookup"><span data-stu-id="06513-412">MaxRetryCount = 2</span></span> | <span data-ttu-id="06513-413">Intento 1: retraso de 0 s</span><span class="sxs-lookup"><span data-stu-id="06513-413">Attempt 1: Delay 0 sec.</span></span> <br/> <span data-ttu-id="06513-414">Intento 2: retraso de ~300 ms</span><span class="sxs-lookup"><span data-stu-id="06513-414">Attempt 2: Delay ~300 msec.</span></span> <br/> <span data-ttu-id="06513-415">Intento 3: retraso de ~900 ms</span><span class="sxs-lookup"><span data-stu-id="06513-415">Attempt 3: Delay ~900 msec.</span></span> |
| <span data-ttu-id="06513-416">Segundo plano o lote</span><span class="sxs-lookup"><span data-stu-id="06513-416">Background or batch</span></span> | <span data-ttu-id="06513-417">30 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-417">30 seconds</span></span> | <span data-ttu-id="06513-418">Exponencial</span><span class="sxs-lookup"><span data-stu-id="06513-418">Exponential</span></span> | <span data-ttu-id="06513-419">MinimumBackoff = 1</span><span class="sxs-lookup"><span data-stu-id="06513-419">MinimumBackoff = 1</span></span> <br/> <span data-ttu-id="06513-420">MaximumBackoff = 30 s</span><span class="sxs-lookup"><span data-stu-id="06513-420">MaximumBackoff = 30 sec.</span></span> <br/> <span data-ttu-id="06513-421">DeltaBackoff = 1,75 s</span><span class="sxs-lookup"><span data-stu-id="06513-421">DeltaBackoff = 1.75 sec.</span></span> <br/> <span data-ttu-id="06513-422">TimeBuffer = 5 s</span><span class="sxs-lookup"><span data-stu-id="06513-422">TimeBuffer = 5 sec.</span></span> <br/> <span data-ttu-id="06513-423">MaxRetryCount = 3</span><span class="sxs-lookup"><span data-stu-id="06513-423">MaxRetryCount = 3</span></span> | <span data-ttu-id="06513-424">Intento 1: retraso de ~1 s</span><span class="sxs-lookup"><span data-stu-id="06513-424">Attempt 1: Delay ~1 sec.</span></span> <br/> <span data-ttu-id="06513-425">Intento 2: retraso de ~3 s</span><span class="sxs-lookup"><span data-stu-id="06513-425">Attempt 2: Delay ~3 sec.</span></span> <br/> <span data-ttu-id="06513-426">Intento 3: retraso de ~6 ms</span><span class="sxs-lookup"><span data-stu-id="06513-426">Attempt 3: Delay ~6 msec.</span></span> <br/> <span data-ttu-id="06513-427">Intento 4: retraso de ~13 ms</span><span class="sxs-lookup"><span data-stu-id="06513-427">Attempt 4: Delay ~13 msec.</span></span> |

<span data-ttu-id="06513-428">\*Sin incluir el retraso adicional que se suma si se recibe una respuesta de servidor ocupado.</span><span class="sxs-lookup"><span data-stu-id="06513-428">\* Not including additional delay that is added if a Server Busy response is received.</span></span>

### <a name="telemetry"></a><span data-ttu-id="06513-429">Telemetría</span><span class="sxs-lookup"><span data-stu-id="06513-429">Telemetry</span></span>
<span data-ttu-id="06513-430">Service Bus registra reintentos como eventos ETW mediante un **EventSource**.</span><span class="sxs-lookup"><span data-stu-id="06513-430">Service Bus logs retries as ETW events using an **EventSource**.</span></span> <span data-ttu-id="06513-431">Debe asociar un **EventListener** al origen de eventos para capturar los eventos y verlos en el Visor de rendimiento o escribirlos en un registro de destino adecuado.</span><span class="sxs-lookup"><span data-stu-id="06513-431">You must attach an **EventListener** to the event source to capture the events and view them in Performance Viewer, or write them to a suitable destination log.</span></span> <span data-ttu-id="06513-432">Puede usar el [Bloque de aplicación de registro semántico](http://msdn.microsoft.com/library/dn775006.aspx) para ello.</span><span class="sxs-lookup"><span data-stu-id="06513-432">You could use the [Semantic Logging Application Block](http://msdn.microsoft.com/library/dn775006.aspx) to do this.</span></span> <span data-ttu-id="06513-433">Los eventos de reintento tienen la forma siguiente:</span><span class="sxs-lookup"><span data-stu-id="06513-433">The retry events are of the following form:</span></span>

```text
Microsoft-ServiceBus-Client/RetryPolicyIteration
ThreadID="14,500"
FormattedMessage="[TrackingId:] RetryExponential: Operation Get:https://retry-tests.servicebus.windows.net/TestQueue/?api-version=2014-05 at iteration 0 is retrying after 00:00:00.1000000 sleep because of Microsoft.ServiceBus.Messaging.MessagingCommunicationException: The remote name could not be resolved: 'retry-tests.servicebus.windows.net'.TrackingId:6a26f99c-dc6d-422e-8565-f89fdd0d4fe3, TimeStamp:9/5/2014 10:00:13 PM."
trackingId=""
policyType="RetryExponential"
operation="Get:https://retry-tests.servicebus.windows.net/TestQueue/?api-version=2014-05"
iteration="0"
iterationSleep="00:00:00.1000000"
lastExceptionType="Microsoft.ServiceBus.Messaging.MessagingCommunicationException"
exceptionMessage="The remote name could not be resolved: 'retry-tests.servicebus.windows.net'.TrackingId:6a26f99c-dc6d-422e-8565-f89fdd0d4fe3,TimeStamp:9/5/2014 10:00:13 PM"
```

### <a name="examples"></a><span data-ttu-id="06513-434">Ejemplos</span><span class="sxs-lookup"><span data-stu-id="06513-434">Examples</span></span>
<span data-ttu-id="06513-435">En el siguiente ejemplo de código se muestra cómo establecer la directiva de reintentos para:</span><span class="sxs-lookup"><span data-stu-id="06513-435">The following code example shows how to set the retry policy for:</span></span>

* <span data-ttu-id="06513-436">Un administrador de espacio de nombres.</span><span class="sxs-lookup"><span data-stu-id="06513-436">A namespace manager.</span></span> <span data-ttu-id="06513-437">La directiva se aplica a todas las operaciones de ese administrador y no se puede reemplazar para las operaciones individuales.</span><span class="sxs-lookup"><span data-stu-id="06513-437">The policy applies to all operations on that manager, and cannot be overridden for individual operations.</span></span>
* <span data-ttu-id="06513-438">Una fábrica de mensajería.</span><span class="sxs-lookup"><span data-stu-id="06513-438">A messaging factory.</span></span> <span data-ttu-id="06513-439">La directiva se aplica a todos los clientes que se crean a partir de ese generador y no se puede invalidar al crear clientes individuales.</span><span class="sxs-lookup"><span data-stu-id="06513-439">The policy applies to all clients created from that factory, and cannot be overridden when creating individual clients.</span></span>
* <span data-ttu-id="06513-440">Un cliente de mensajería individual.</span><span class="sxs-lookup"><span data-stu-id="06513-440">An individual messaging client.</span></span> <span data-ttu-id="06513-441">Después de crear un cliente, puede establecer la directiva de reintentos para el cliente.</span><span class="sxs-lookup"><span data-stu-id="06513-441">After a client has been created, you can set the retry policy for that client.</span></span> <span data-ttu-id="06513-442">La directiva se aplica a todas las operaciones de ese cliente.</span><span class="sxs-lookup"><span data-stu-id="06513-442">The policy applies to all operations on that client.</span></span>

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.ServiceBus;
using Microsoft.ServiceBus.Messaging;

namespace RetryCodeSamples
{
    class ServiceBusCodeSamples
    {
        private const string connectionString =
            @"Endpoint=sb://[my-namespace].servicebus.windows.net/;
                SharedAccessKeyName=RootManageSharedAccessKey;
                SharedAccessKey=C99..........Mk=";

        public async static Task Samples()
        {
            const string QueueName = "TestQueue";

            ServiceBusEnvironment.SystemConnectivity.Mode = ConnectivityMode.Http;

            var namespaceManager = NamespaceManager.CreateFromConnectionString(connectionString);

            // The namespace manager will have a default exponential policy with 10 retry attempts
            // and a 3 second delay delta.
            // Retry delays will be approximately 0 sec, 3 sec, 9 sec, 25 sec and the fixed 30 sec,
            // with an extra 10 sec added when receiving a ServiceBusyException.

            {
                // Set different values for the retry policy, used for all operations on the namespace manager.
                namespaceManager.Settings.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(0),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);

                // Policies cannot be specified on a per-operation basis.
                if (!await namespaceManager.QueueExistsAsync(QueueName))
                {
                    await namespaceManager.CreateQueueAsync(QueueName);
                }
            }

            var messagingFactory = MessagingFactory.Create(
                namespaceManager.Address, namespaceManager.Settings.TokenProvider);
            // The messaging factory will have a default exponential policy with 10 retry attempts
            // and a 3 second delay delta.
            // Retry delays will be approximately 0 sec, 3 sec, 9 sec, 25 sec and the fixed 30 sec,
            // with an extra 10 sec added when receiving a ServiceBusyException.

            {
                // Set different values for the retry policy, used for clients created from it.
                messagingFactory.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(1),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);


                // Policies cannot be specified on a per-operation basis.
                var session = await messagingFactory.AcceptMessageSessionAsync();
            }

            {
                var client = messagingFactory.CreateQueueClient(QueueName);
                // The client inherits the policy from the factory that created it.


                // Set different values for the retry policy on the client.
                client.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(0.1),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);

                // Policies cannot be specified on a per-operation basis.
                var session = await client.AcceptMessageSessionAsync();
            }
        }
    }
}
```

### <a name="more-information"></a><span data-ttu-id="06513-443">Más información</span><span class="sxs-lookup"><span data-stu-id="06513-443">More information</span></span>
* [<span data-ttu-id="06513-444">Patrones de mensajería asincrónica y alta disponibilidad.</span><span class="sxs-lookup"><span data-stu-id="06513-444">Asynchronous Messaging Patterns and High Availability</span></span>](http://msdn.microsoft.com/library/azure/dn292562.aspx)

## <a name="service-fabric"></a><span data-ttu-id="06513-445">Service Fabric</span><span class="sxs-lookup"><span data-stu-id="06513-445">Service Fabric</span></span>

<span data-ttu-id="06513-446">Distribuir servicios confiables en un clúster de Service Fabric protege frente a la mayoría de los errores transitorios posibles descritos en este artículo.</span><span class="sxs-lookup"><span data-stu-id="06513-446">Distributing reliable services in a Service Fabric cluster guards against most of the potential transient faults discussed in this article.</span></span> <span data-ttu-id="06513-447">Sin embargo, aún se pueden producir algunos errores transitorios.</span><span class="sxs-lookup"><span data-stu-id="06513-447">Some transient faults are still possible, however.</span></span> <span data-ttu-id="06513-448">Por ejemplo, el servicio de nombres podría estar en medio de un cambio de enrutamiento cuando recibe una solicitud, lo que hace que se produzca una excepción.</span><span class="sxs-lookup"><span data-stu-id="06513-448">For example, the naming service might be in the middle of a routing change when it gets a request, causing it to throw an exception.</span></span> <span data-ttu-id="06513-449">Si la misma solicitud llega 100 milisegundos después, probablemente se realizará correctamente.</span><span class="sxs-lookup"><span data-stu-id="06513-449">If the same request comes 100 milliseconds later, it will probably succeed.</span></span>

<span data-ttu-id="06513-450">Internamente, Service Fabric administra este tipo de errores transitorios.</span><span class="sxs-lookup"><span data-stu-id="06513-450">Internally, Service Fabric manages this kind of transient fault.</span></span> <span data-ttu-id="06513-451">Puede configurar algunas opciones mediante la clase `OperationRetrySettings` al configurar los servicios.</span><span class="sxs-lookup"><span data-stu-id="06513-451">You can configure some settings by using the `OperationRetrySettings` class while setting up your services.</span></span>  <span data-ttu-id="06513-452">El código siguiente muestra un ejemplo.</span><span class="sxs-lookup"><span data-stu-id="06513-452">The following code shows an example.</span></span> <span data-ttu-id="06513-453">En la mayoría de los casos, esto no será necesario y la configuración predeterminada funcionará bien.</span><span class="sxs-lookup"><span data-stu-id="06513-453">In most cases, this should not be necessary, and the default settings will be fine.</span></span>

```csharp
FabricTransportRemotingSettings transportSettings = new FabricTransportRemotingSettings
{
    OperationTimeout = TimeSpan.FromSeconds(30)
};

var retrySettings = new OperationRetrySettings(TimeSpan.FromSeconds(15), TimeSpan.FromSeconds(1), 5);

var clientFactory = new FabricTransportServiceRemotingClientFactory(transportSettings);

var serviceProxyFactory = new ServiceProxyFactory((c) => clientFactory, retrySettings);

var client = serviceProxyFactory.CreateServiceProxy<ISomeService>(
    new Uri("fabric:/SomeApp/SomeStatefulReliableService"),
    new ServicePartitionKey(0));
```

### <a name="more-information"></a><span data-ttu-id="06513-454">Más información</span><span class="sxs-lookup"><span data-stu-id="06513-454">More information</span></span>

* [<span data-ttu-id="06513-455">Control remoto de excepciones</span><span class="sxs-lookup"><span data-stu-id="06513-455">Remote Exception Handling</span></span>](https://github.com/Microsoft/azure-docs/blob/master/articles/service-fabric/service-fabric-reliable-services-communication-remoting.md#remoting-exception-handling)

## <a name="sql-database-using-adonet"></a><span data-ttu-id="06513-456">SQL Database mediante ADO.NET</span><span class="sxs-lookup"><span data-stu-id="06513-456">SQL Database using ADO.NET</span></span>
<span data-ttu-id="06513-457">SQL Database es una instancia de SQL Database hospedada que está disponible en una amplia variedad de tamaños y como un servicio estándar (compartido) y premium (no compartido).</span><span class="sxs-lookup"><span data-stu-id="06513-457">SQL Database is a hosted SQL database available in a range of sizes and as both a standard (shared) and premium (non-shared) service.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="06513-458">Mecanismo de reintento</span><span class="sxs-lookup"><span data-stu-id="06513-458">Retry mechanism</span></span>
<span data-ttu-id="06513-459">SQL Database no tiene soporte integrado para reintentos cuando se tiene acceso mediante ADO.NET.</span><span class="sxs-lookup"><span data-stu-id="06513-459">SQL Database has no built-in support for retries when accessed using ADO.NET.</span></span> <span data-ttu-id="06513-460">Sin embargo, los códigos de retorno de solicitudes pueden usarse para determinar el motivo del error de una solicitud.</span><span class="sxs-lookup"><span data-stu-id="06513-460">However, the return codes from requests can be used to determine why a request failed.</span></span> <span data-ttu-id="06513-461">Para más información acerca de la limitación de SQL Database, consulte [Límites de recursos de Azure SQL Database](/azure/sql-database/sql-database-resource-limits).</span><span class="sxs-lookup"><span data-stu-id="06513-461">For more information about SQL Database throttling, see [Azure SQL Database resource limits](/azure/sql-database/sql-database-resource-limits).</span></span> <span data-ttu-id="06513-462">Para obtener una lista de los códigos de error pertinentes, consulte [Códigos de error SQL para las aplicaciones de cliente de SQL Database](/azure/sql-database/sql-database-develop-error-messages).</span><span class="sxs-lookup"><span data-stu-id="06513-462">For a list of relevant error codes, see [SQL error codes for SQL Database client applications](/azure/sql-database/sql-database-develop-error-messages).</span></span>

<span data-ttu-id="06513-463">Puede usar la biblioteca de Polly implementar reintentos para SQL Database.</span><span class="sxs-lookup"><span data-stu-id="06513-463">You can use the Polly library to implement retries for SQL Database.</span></span> <span data-ttu-id="06513-464">Consulte [Control de errores transitorios con Polly](#transient-fault-handling-with-polly).</span><span class="sxs-lookup"><span data-stu-id="06513-464">See [Transient fault handling with Polly](#transient-fault-handling-with-polly).</span></span>

### <a name="retry-usage-guidance"></a><span data-ttu-id="06513-465">Instrucciones de uso del reintento</span><span class="sxs-lookup"><span data-stu-id="06513-465">Retry usage guidance</span></span>
<span data-ttu-id="06513-466">Al obtener acceso a la SQL Database mediante ADO.NET, tenga en cuenta las siguientes directrices:</span><span class="sxs-lookup"><span data-stu-id="06513-466">Consider the following guidelines when accessing SQL Database using ADO.NET:</span></span>

* <span data-ttu-id="06513-467">Elija la opción de servicio adecuada (compartida o premium).</span><span class="sxs-lookup"><span data-stu-id="06513-467">Choose the appropriate service option (shared or premium).</span></span> <span data-ttu-id="06513-468">Una instancia compartida puede sufrir retrasos en la conexión más largos de lo habitual y limitaciones debido al uso de otros inquilinos del servidor compartido.</span><span class="sxs-lookup"><span data-stu-id="06513-468">A shared instance may suffer longer than usual connection delays and throttling due to the usage by other tenants of the shared server.</span></span> <span data-ttu-id="06513-469">Si se requieren más operaciones de rendimiento predecible y de latencia baja confiable, considere la posibilidad de elegir la opción premium.</span><span class="sxs-lookup"><span data-stu-id="06513-469">If more predictable performance and reliable low latency operations are required, consider choosing the premium option.</span></span>
* <span data-ttu-id="06513-470">Asegúrese de realizar reintentos al nivel o ámbito adecuado para evitar las operaciones no idempotentes que provocan incoherencias en los datos.</span><span class="sxs-lookup"><span data-stu-id="06513-470">Ensure that you perform retries at the appropriate level or scope to avoid non-idempotent operations causing inconsistency in the data.</span></span> <span data-ttu-id="06513-471">Lo ideal es que todas las operaciones sean idempotentes para que puedan repetirse sin causar incoherencias.</span><span class="sxs-lookup"><span data-stu-id="06513-471">Ideally, all operations should be idempotent so that they can be repeated without causing inconsistency.</span></span> <span data-ttu-id="06513-472">Si no es el caso, el reintento deberá realizarse en un nivel o ámbito que permita que todos los cambios relacionados se deshagan si se produce un error en una operación; por ejemplo, desde dentro de un ámbito transaccional.</span><span class="sxs-lookup"><span data-stu-id="06513-472">Where this is not the case, the retry should be performed at a level or scope that allows all related changes to be undone if one operation fails; for example, from within a transactional scope.</span></span> <span data-ttu-id="06513-473">Para obtener más información, consulte [Capa de acceso a datos de fundamentos del servicio de nube: tratamiento de errores transitorios](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx#Idempotent_Guarantee).</span><span class="sxs-lookup"><span data-stu-id="06513-473">For more information, see [Cloud Service Fundamentals Data Access Layer – Transient Fault Handling](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx#Idempotent_Guarantee).</span></span>
* <span data-ttu-id="06513-474">No se recomienda una estrategia de intervalo fijo para su uso con Azure SQL Database excepto para los escenarios interactivos donde hay solo unos cuantos reintentos a intervalos muy cortos.</span><span class="sxs-lookup"><span data-stu-id="06513-474">A fixed interval strategy is not recommended for use with Azure SQL Database except for interactive scenarios where there are only a few retries at very short intervals.</span></span> <span data-ttu-id="06513-475">En su lugar, considere el uso de una estrategia de retroceso exponencial para la mayoría de escenarios.</span><span class="sxs-lookup"><span data-stu-id="06513-475">Instead, consider using an exponential back-off strategy for the majority of scenarios.</span></span>
* <span data-ttu-id="06513-476">Elija un valor adecuado para los tiempos de espera de conexión y comando a la hora de definir las conexiones.</span><span class="sxs-lookup"><span data-stu-id="06513-476">Choose a suitable value for the connection and command timeouts when defining connections.</span></span> <span data-ttu-id="06513-477">Un tiempo de espera demasiado corto puede provocar errores prematuros en las conexiones cuando la base de datos está ocupada.</span><span class="sxs-lookup"><span data-stu-id="06513-477">Too short a timeout may result in premature failures of connections when the database is busy.</span></span> <span data-ttu-id="06513-478">Un tiempo de espera demasiado largo puede impedir que la lógica de reintento funcione correctamente esperando demasiado tiempo antes de detectar un error en la conexión.</span><span class="sxs-lookup"><span data-stu-id="06513-478">Too long a timeout may prevent the retry logic working correctly by waiting too long before detecting a failed connection.</span></span> <span data-ttu-id="06513-479">El valor de tiempo de espera es un componente de la latencia de extremo a extremo; se agrega eficazmente al intervalo entre reintentos especificado en la directiva de reintento para cada reintento.</span><span class="sxs-lookup"><span data-stu-id="06513-479">The value of the timeout is a component of the end-to-end latency; it is effectively added to the retry delay specified in the retry policy for every retry attempt.</span></span>
* <span data-ttu-id="06513-480">Cierre la conexión después de un cierto número de reintentos, incluso cuando se usa una lógica de reintentos de interrupción exponencial y vuelva a intentar la operación en una conexión nueva.</span><span class="sxs-lookup"><span data-stu-id="06513-480">Close the connection after a certain number of retries, even when using an exponential back off retry logic, and retry the operation on a new connection.</span></span> <span data-ttu-id="06513-481">Reintentar la misma operación varias veces en la misma conexión puede ser un factor que contribuya a ocasionar problemas de conexión.</span><span class="sxs-lookup"><span data-stu-id="06513-481">Retrying the same operation multiple times on the same connection can be a factor that contributes to connection problems.</span></span> <span data-ttu-id="06513-482">Para obtener un ejemplo de esta técnica, consulte [Capa de acceso a datos de fundamentos del servicio de nube: tratamiento de errores transitorios](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx).</span><span class="sxs-lookup"><span data-stu-id="06513-482">For an example of this technique, see [Cloud Service Fundamentals Data Access Layer – Transient Fault Handling](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx).</span></span>
* <span data-ttu-id="06513-483">Cuando la agrupación de conexiones está en uso (valor predeterminado) es probable que se elija la misma conexión de la agrupación, incluso después de cerrar y volver a abrir una conexión.</span><span class="sxs-lookup"><span data-stu-id="06513-483">When connection pooling is in use (the default) there is a chance that the same connection will be chosen from the pool, even after closing and reopening a connection.</span></span> <span data-ttu-id="06513-484">Si este es el caso, una técnica para resolverlo es llamar al método **ClearPool** de la clase **SqlConnection** para marcar la conexión como no reutilizable.</span><span class="sxs-lookup"><span data-stu-id="06513-484">If this is the case, a technique to resolve it is to call the **ClearPool** method of the **SqlConnection** class to mark the connection as not reusable.</span></span> <span data-ttu-id="06513-485">Sin embargo, debería hacerlo solo después de que fallen varios intentos de conexión y solo al encontrar la clase específica de errores transitorios como tiempos de espera SQL (código de error -2) relacionados con conexiones erróneas.</span><span class="sxs-lookup"><span data-stu-id="06513-485">However, you should do this only after several connection attempts have failed, and only when encountering the specific class of transient failures such as SQL timeouts (error code -2) related to faulty connections.</span></span>
* <span data-ttu-id="06513-486">Si el código de acceso de datos usa las transacciones iniciadas como instancias **TransactionScope** , la lógica de reintento debe volver a abrir la conexión e iniciar un nuevo ámbito de transacción.</span><span class="sxs-lookup"><span data-stu-id="06513-486">If the data access code uses transactions initiated as **TransactionScope** instances, the retry logic should reopen the connection and initiate a new transaction scope.</span></span> <span data-ttu-id="06513-487">Por este motivo, el bloque de código que se puede reintentar debe abarcar todo el ámbito de la transacción.</span><span class="sxs-lookup"><span data-stu-id="06513-487">For this reason, the retryable code block should encompass the entire scope of the transaction.</span></span>

<span data-ttu-id="06513-488">Considere la posibilidad de comenzar con la configuración siguiente para volver a intentar las operaciones.</span><span class="sxs-lookup"><span data-stu-id="06513-488">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="06513-489">Esta es la configuración de propósito general, y debe supervisar las operaciones y ajustar los valores para adaptarlos a su propio escenario.</span><span class="sxs-lookup"><span data-stu-id="06513-489">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="06513-490">**Contexto**</span><span class="sxs-lookup"><span data-stu-id="06513-490">**Context**</span></span> | <span data-ttu-id="06513-491">**Destino de ejemplo E2E<br />latencia máxima**</span><span class="sxs-lookup"><span data-stu-id="06513-491">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="06513-492">**Estrategia de reintento**</span><span class="sxs-lookup"><span data-stu-id="06513-492">**Retry strategy**</span></span> | <span data-ttu-id="06513-493">**Configuración**</span><span class="sxs-lookup"><span data-stu-id="06513-493">**Settings**</span></span> | <span data-ttu-id="06513-494">**Valores**</span><span class="sxs-lookup"><span data-stu-id="06513-494">**Values**</span></span> | <span data-ttu-id="06513-495">**Cómo funciona**</span><span class="sxs-lookup"><span data-stu-id="06513-495">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="06513-496">Interactivo, interfaz de usuario</span><span class="sxs-lookup"><span data-stu-id="06513-496">Interactive, UI,</span></span><br /><span data-ttu-id="06513-497">o primer plano</span><span class="sxs-lookup"><span data-stu-id="06513-497">or foreground</span></span> |<span data-ttu-id="06513-498">2 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-498">2 sec</span></span> |<span data-ttu-id="06513-499">FixedInterval</span><span class="sxs-lookup"><span data-stu-id="06513-499">FixedInterval</span></span> |<span data-ttu-id="06513-500">Número de reintentos</span><span class="sxs-lookup"><span data-stu-id="06513-500">Retry count</span></span><br /><span data-ttu-id="06513-501">Intervalo de reintento</span><span class="sxs-lookup"><span data-stu-id="06513-501">Retry interval</span></span><br /><span data-ttu-id="06513-502">Primer reintento rápido</span><span class="sxs-lookup"><span data-stu-id="06513-502">First fast retry</span></span> |<span data-ttu-id="06513-503">3</span><span class="sxs-lookup"><span data-stu-id="06513-503">3</span></span><br /><span data-ttu-id="06513-504">500 ms</span><span class="sxs-lookup"><span data-stu-id="06513-504">500 ms</span></span><br /><span data-ttu-id="06513-505">true</span><span class="sxs-lookup"><span data-stu-id="06513-505">true</span></span> |<span data-ttu-id="06513-506">Intento 1 - retraso de 0 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-506">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="06513-507">Intento 2 - retraso de 500 ms</span><span class="sxs-lookup"><span data-stu-id="06513-507">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="06513-508">Intento 3 – retraso de 500 ms</span><span class="sxs-lookup"><span data-stu-id="06513-508">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="06513-509">Fondo</span><span class="sxs-lookup"><span data-stu-id="06513-509">Background</span></span><br /><span data-ttu-id="06513-510">o proceso por lotes</span><span class="sxs-lookup"><span data-stu-id="06513-510">or batch</span></span> |<span data-ttu-id="06513-511">30 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-511">30 sec</span></span> |<span data-ttu-id="06513-512">ExponentialBackoff</span><span class="sxs-lookup"><span data-stu-id="06513-512">ExponentialBackoff</span></span> |<span data-ttu-id="06513-513">Número de reintentos</span><span class="sxs-lookup"><span data-stu-id="06513-513">Retry count</span></span><br /><span data-ttu-id="06513-514">Interrupción mínima</span><span class="sxs-lookup"><span data-stu-id="06513-514">Min back-off</span></span><br /><span data-ttu-id="06513-515">Interrupción máxima</span><span class="sxs-lookup"><span data-stu-id="06513-515">Max back-off</span></span><br /><span data-ttu-id="06513-516">Interrupción delta</span><span class="sxs-lookup"><span data-stu-id="06513-516">Delta back-off</span></span><br /><span data-ttu-id="06513-517">Primer reintento rápido</span><span class="sxs-lookup"><span data-stu-id="06513-517">First fast retry</span></span> |<span data-ttu-id="06513-518">5</span><span class="sxs-lookup"><span data-stu-id="06513-518">5</span></span><br /><span data-ttu-id="06513-519">0 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-519">0 sec</span></span><br /><span data-ttu-id="06513-520">60 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-520">60 sec</span></span><br /><span data-ttu-id="06513-521">2 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-521">2 sec</span></span><br /><span data-ttu-id="06513-522">false</span><span class="sxs-lookup"><span data-stu-id="06513-522">false</span></span> |<span data-ttu-id="06513-523">Intento 1 - retraso de 0 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-523">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="06513-524">Intento 2 - retraso de ~2 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-524">Attempt 2 - delay ~2 sec</span></span><br /><span data-ttu-id="06513-525">Intento 3 - retraso de ~6 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-525">Attempt 3 - delay ~6 sec</span></span><br /><span data-ttu-id="06513-526">Intento 4 - retraso de ~14 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-526">Attempt 4 - delay ~14 sec</span></span><br /><span data-ttu-id="06513-527">Intento 5 - retraso de ~30 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-527">Attempt 5 - delay ~30 sec</span></span> |

> [!NOTE]
> <span data-ttu-id="06513-528">Los destinos de latencia de extremo a extremo suponen el tiempo de espera predeterminado para las conexiones con el servicio.</span><span class="sxs-lookup"><span data-stu-id="06513-528">The end-to-end latency targets assume the default timeout for connections to the service.</span></span> <span data-ttu-id="06513-529">Si especifica tiempos de espera de conexión más largos, la latencia de extremo a extremo se extenderá este tiempo adicional en cada reintento.</span><span class="sxs-lookup"><span data-stu-id="06513-529">If you specify longer connection timeouts, the end-to-end latency will be extended by this additional time for every retry attempt.</span></span>
>
>

### <a name="examples"></a><span data-ttu-id="06513-530">Ejemplos</span><span class="sxs-lookup"><span data-stu-id="06513-530">Examples</span></span>
<span data-ttu-id="06513-531">En esta sección se muestra cómo usar Polly para acceder a Azure SQL Database mediante un conjunto de directivas de reintento configurado en la clase `Policy`.</span><span class="sxs-lookup"><span data-stu-id="06513-531">This section shows how you can use Polly to access Azure SQL Database using a set of retry policies configured in the `Policy` class.</span></span>

<span data-ttu-id="06513-532">El código siguiente muestra un método de extensión en la clase `SqlCommand` que llama a `ExecuteAsync` con retroceso exponencial.</span><span class="sxs-lookup"><span data-stu-id="06513-532">The following code shows an extension method on the `SqlCommand` class that calls `ExecuteAsync` with exponential backoff.</span></span>

```csharp
public async static Task<SqlDataReader> ExecuteReaderWithRetryAsync(this SqlCommand command)
{
    GuardConnectionIsNotNull(command);

    var policy = Policy.Handle<Exception>().WaitAndRetryAsync(
        retryCount: 3, // Retry 3 times
        sleepDurationProvider: attempt => TimeSpan.FromMilliseconds(200 * Math.Pow(2, attempt - 1)), // Exponential backoff based on an initial 200ms delay.
        onRetry: (exception, attempt) => 
        {
            // Capture some info for logging/telemetry.  
            logger.LogWarn($"ExecuteReaderWithRetryAsync: Retry {attempt} due to {exception}.");
        });

    // Retry the following call according to the policy.
    await policy.ExecuteAsync<SqlDataReader>(async token =>
    {
        // This code is executed within the Policy 

        if (conn.State != System.Data.ConnectionState.Open) await conn.OpenAsync(token);
        return await command.ExecuteReaderAsync(System.Data.CommandBehavior.Default, token);

    }, cancellationToken);
}
```

<span data-ttu-id="06513-533">Este método de extensión asincrónico puede utilizarse como se indica a continuación.</span><span class="sxs-lookup"><span data-stu-id="06513-533">This asynchronous extension method can be used as follows.</span></span>

```csharp
var sqlCommand = sqlConnection.CreateCommand();
sqlCommand.CommandText = "[some query]";

using (var reader = await sqlCommand.ExecuteReaderWithRetryAsync())
{
    // Do something with the values
}
```

### <a name="more-information"></a><span data-ttu-id="06513-534">Más información</span><span class="sxs-lookup"><span data-stu-id="06513-534">More information</span></span>
* [<span data-ttu-id="06513-535">Capa de acceso a datos de fundamentos del servicio de nube: tratamiento de errores transitorios</span><span class="sxs-lookup"><span data-stu-id="06513-535">Cloud Service Fundamentals Data Access Layer – Transient Fault Handling</span></span>](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx)

<span data-ttu-id="06513-536">Para obtener instrucciones generales sobre cómo sacar el máximo partido de SQL Database, consulte [Azure SQL Database Performance and Elasticity Guide](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx) (Guía de elasticidad y rendimiento de base de datos de SQL Azure).</span><span class="sxs-lookup"><span data-stu-id="06513-536">For general guidance on getting the most from SQL Database, see [Azure SQL Database Performance and Elasticity Guide](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx).</span></span>

## <a name="sql-database-using-entity-framework-6"></a><span data-ttu-id="06513-537">SQL Database mediante Entity Framework 6</span><span class="sxs-lookup"><span data-stu-id="06513-537">SQL Database using Entity Framework 6</span></span>
<span data-ttu-id="06513-538">SQL Database es una instancia de SQL Database hospedada que está disponible en una amplia variedad de tamaños y como un servicio estándar (compartido) y premium (no compartido).</span><span class="sxs-lookup"><span data-stu-id="06513-538">SQL Database is a hosted SQL database available in a range of sizes and as both a standard (shared) and premium (non-shared) service.</span></span> <span data-ttu-id="06513-539">Entity Framework es un mapeador relacional de objetos que permite a los desarrolladores de .NET trabajar con datos relacionales usando objetos específicos del dominio.</span><span class="sxs-lookup"><span data-stu-id="06513-539">Entity Framework is an object-relational mapper that enables .NET developers to work with relational data using domain-specific objects.</span></span> <span data-ttu-id="06513-540">Elimina la necesidad de usar la mayoría del código de acceso a datos que los programadores suelen tener que escribir.</span><span class="sxs-lookup"><span data-stu-id="06513-540">It eliminates the need for most of the data-access code that developers usually need to write.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="06513-541">Mecanismo de reintento</span><span class="sxs-lookup"><span data-stu-id="06513-541">Retry mechanism</span></span>
<span data-ttu-id="06513-542">Se proporciona compatibilidad con los reintentos al obtener acceso a la SQL Database mediante Entity Framework 6.0 y versiones posteriores mediante un mecanismo denominado [Resistencia de la conexión y lógica de reintento](http://msdn.microsoft.com/data/dn456835.aspx).</span><span class="sxs-lookup"><span data-stu-id="06513-542">Retry support is provided when accessing SQL Database using Entity Framework 6.0 and higher through a mechanism called [Connection Resiliency / Retry Logic](http://msdn.microsoft.com/data/dn456835.aspx).</span></span> <span data-ttu-id="06513-543">Las características principales del mecanismo de reintento son:</span><span class="sxs-lookup"><span data-stu-id="06513-543">The main features of the retry mechanism are:</span></span>

* <span data-ttu-id="06513-544">La abstracción principal es la interfaz **IDbExecutionStrategy** .</span><span class="sxs-lookup"><span data-stu-id="06513-544">The primary abstraction is the **IDbExecutionStrategy** interface.</span></span> <span data-ttu-id="06513-545">Esta interfaz:</span><span class="sxs-lookup"><span data-stu-id="06513-545">This interface:</span></span>
  * <span data-ttu-id="06513-546">Define métodos **Execute**\* sincrónicos y asincrónicos.</span><span class="sxs-lookup"><span data-stu-id="06513-546">Defines synchronous and asynchronous **Execute**\* methods.</span></span>
  * <span data-ttu-id="06513-547">Define las clases que pueden usarse o configurarse directamente en un contexto de base de datos como una estrategia predeterminada, asignada al nombre de un proveedor o asignada a un nombre de proveedor y de servidor.</span><span class="sxs-lookup"><span data-stu-id="06513-547">Defines classes that can be used directly or can be configured on a database context as a default strategy, mapped to provider name, or mapped to a provider name and server name.</span></span> <span data-ttu-id="06513-548">Cuando se configura en un contexto, los reintentos se producen en el nivel de operaciones de base de datos individual, de las que podría haber varias para una operación de contexto especificada.</span><span class="sxs-lookup"><span data-stu-id="06513-548">When configured on a context, retries occur at the level of individual database operations, of which there might be several for a given context operation.</span></span>
  * <span data-ttu-id="06513-549">Define cuándo se debe reintentar una conexión fallida y cómo.</span><span class="sxs-lookup"><span data-stu-id="06513-549">Defines when to retry a failed connection, and how.</span></span>
* <span data-ttu-id="06513-550">Incluye varias implementaciones integradas de la interfaz **IDbExecutionStrategy** :</span><span class="sxs-lookup"><span data-stu-id="06513-550">It includes several built-in implementations of the **IDbExecutionStrategy** interface:</span></span>
  * <span data-ttu-id="06513-551">Predeterminado: ningún reintento.</span><span class="sxs-lookup"><span data-stu-id="06513-551">Default - no retrying.</span></span>
  * <span data-ttu-id="06513-552">Valor predeterminado para la SQL Database (automático): sin reintento, pero inspecciona las excepciones y las ajusta con la recomendación de usar la estrategia de SQL Database.</span><span class="sxs-lookup"><span data-stu-id="06513-552">Default for SQL Database (automatic) - no retrying, but inspects exceptions and wraps them with suggestion to use the SQL Database strategy.</span></span>
  * <span data-ttu-id="06513-553">Predeterminado para la SQL Database: exponencial (heredado de la clase base) más la lógica de detección de la SQL Database.</span><span class="sxs-lookup"><span data-stu-id="06513-553">Default for SQL Database - exponential (inherited from base class) plus SQL Database detection logic.</span></span>
* <span data-ttu-id="06513-554">Implementa una estrategia de retroceso exponencial que incluye la selección aleatoria.</span><span class="sxs-lookup"><span data-stu-id="06513-554">It implements an exponential back-off strategy that includes randomization.</span></span>
* <span data-ttu-id="06513-555">Las clases de reintento integradas tienen estado y no están protegidas para subprocesos.</span><span class="sxs-lookup"><span data-stu-id="06513-555">The built-in retry classes are stateful and are not thread safe.</span></span> <span data-ttu-id="06513-556">Sin embargo, se pueden volver a usar una vez completada la operación actual.</span><span class="sxs-lookup"><span data-stu-id="06513-556">However, they can be reused after the current operation is completed.</span></span>
* <span data-ttu-id="06513-557">Si se supera el número de reintentos especificado, los resultados se encapsulan en una nueva excepción.</span><span class="sxs-lookup"><span data-stu-id="06513-557">If the specified retry count is exceeded, the results are wrapped in a new exception.</span></span> <span data-ttu-id="06513-558">No se propaga la excepción actual.</span><span class="sxs-lookup"><span data-stu-id="06513-558">It does not bubble up the current exception.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="06513-559">Configuración de directivas</span><span class="sxs-lookup"><span data-stu-id="06513-559">Policy configuration</span></span>
<span data-ttu-id="06513-560">Al obtener acceso a SQL Database mediante Entity Framework 6.0 y versiones posteriores, se proporciona compatibilidad con los reintentos.</span><span class="sxs-lookup"><span data-stu-id="06513-560">Retry support is provided when accessing SQL Database using Entity Framework 6.0 and higher.</span></span> <span data-ttu-id="06513-561">Las directivas de reintento se configuran mediante programación.</span><span class="sxs-lookup"><span data-stu-id="06513-561">Retry policies are configured programmatically.</span></span> <span data-ttu-id="06513-562">No se puede cambiar la configuración en cada operación.</span><span class="sxs-lookup"><span data-stu-id="06513-562">The configuration cannot be changed on a per-operation basis.</span></span>

<span data-ttu-id="06513-563">Al configurar una estrategia en el contexto como valor predeterminado, especifique una función que cree una nueva estrategia a petición.</span><span class="sxs-lookup"><span data-stu-id="06513-563">When configuring a strategy on the context as the default, you specify a function that creates a new strategy on demand.</span></span> <span data-ttu-id="06513-564">El código siguiente muestra cómo puede crear una clase de configuración de reintento que amplíe la clase base **DbConfiguration** .</span><span class="sxs-lookup"><span data-stu-id="06513-564">The following code shows how you can create a retry configuration class that extends the **DbConfiguration** base class.</span></span>

```csharp
public class BloggingContextConfiguration : DbConfiguration
{
  public BlogConfiguration()
  {
    // Set up the execution strategy for SQL Database (exponential) with 5 retries and 4 sec delay
    this.SetExecutionStrategy(
         "System.Data.SqlClient", () => new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(4)));
  }
}
```

<span data-ttu-id="06513-565">A continuación, puede especificar esto como estrategia de reintento predeterminada para todas las operaciones mediante el método **SetConfiguration** de la instancia **DbConfiguration** cuando se inicia la aplicación.</span><span class="sxs-lookup"><span data-stu-id="06513-565">You can then specify this as the default retry strategy for all operations using the **SetConfiguration** method of the **DbConfiguration** instance when the application starts.</span></span> <span data-ttu-id="06513-566">De forma predeterminada, EF detectará y usará automáticamente la clase de configuración.</span><span class="sxs-lookup"><span data-stu-id="06513-566">By default, EF will automatically discover and use the configuration class.</span></span>

```csharp
DbConfiguration.SetConfiguration(new BloggingContextConfiguration());
```

<span data-ttu-id="06513-567">Puede especificar la clase de configuración de reintento para un contexto anotando la clase de contexto con un atributo **DbConfigurationType** .</span><span class="sxs-lookup"><span data-stu-id="06513-567">You can specify the retry configuration class for a context by annotating the context class with a **DbConfigurationType** attribute.</span></span> <span data-ttu-id="06513-568">Sin embargo, si solo tiene una clase de configuración, EF la usará sin necesidad de anotar el contexto.</span><span class="sxs-lookup"><span data-stu-id="06513-568">However, if you have only one configuration class, EF will use it without the need to annotate the context.</span></span>

```csharp
[DbConfigurationType(typeof(BloggingContextConfiguration))]
public class BloggingContext : DbContext
```

<span data-ttu-id="06513-569">Si necesita usar estrategias de reintento diferentes para operaciones específicas o deshabilitar reintentos para operaciones específicas, puede crear una clase de configuración que permita suspender o intercambiar estrategias estableciendo un indicador en **CallContext**.</span><span class="sxs-lookup"><span data-stu-id="06513-569">If you need to use different retry strategies for specific operations, or disable retries for specific operations, you can create a configuration class that allows you to suspend or swap strategies by setting a flag in the **CallContext**.</span></span> <span data-ttu-id="06513-570">La clase de configuración puede usar este indicador para cambiar las estrategias o deshabilitar la estrategia proporcionada por usted y usar una estrategia predeterminada.</span><span class="sxs-lookup"><span data-stu-id="06513-570">The configuration class can use this flag to switch strategies, or disable the strategy you provide and use a default strategy.</span></span> <span data-ttu-id="06513-571">Para obtener más información, consulte [Suspender la estrategia de ejecución](http://msdn.microsoft.com/dn307226#transactions_workarounds) en la página Limitaciones con estrategias de ejecución reintentos (EF6 y versiones posteriores).</span><span class="sxs-lookup"><span data-stu-id="06513-571">For more information, see [Suspend Execution Strategy](http://msdn.microsoft.com/dn307226#transactions_workarounds) in the page Limitations with Retrying Execution Strategies (EF6 onwards).</span></span>

<span data-ttu-id="06513-572">Otra técnica para el uso de estrategias de reintento específica para las operaciones individuales es crear una instancia de la clase de estrategia necesaria y proporcionar la configuración deseada a través de parámetros.</span><span class="sxs-lookup"><span data-stu-id="06513-572">Another technique for using specific retry strategies for individual operations is to create an instance of the required strategy class and supply the desired settings through parameters.</span></span> <span data-ttu-id="06513-573">A continuación, invoque su método **ExecuteAsync** .</span><span class="sxs-lookup"><span data-stu-id="06513-573">You then invoke its **ExecuteAsync** method.</span></span>

```csharp
var executionStrategy = new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(4));
var blogs = await executionStrategy.ExecuteAsync(
    async () =>
    {
        using (var db = new BloggingContext("Blogs"))
        {
            // Acquire some values asynchronously and return them
        }
    },
    new CancellationToken()
);
```

<span data-ttu-id="06513-574">La manera más sencilla de usar una clase **DbConfiguration** es buscarla en el mismo ensamblado que la clase **DbContext**.</span><span class="sxs-lookup"><span data-stu-id="06513-574">The simplest way to use a **DbConfiguration** class is to locate it in the same assembly as the **DbContext** class.</span></span> <span data-ttu-id="06513-575">Sin embargo, esto no es adecuado cuando se requiere el mismo contexto en diferentes escenarios, como diferentes estrategias de reintento interactivo y en segundo plano.</span><span class="sxs-lookup"><span data-stu-id="06513-575">However, this is not appropriate when the same context is required in different scenarios, such as different interactive and background retry strategies.</span></span> <span data-ttu-id="06513-576">Si los contextos diferentes se ejecutan en dominios de aplicación independientes, puede usar la compatibilidad integrada para especificar clases de configuración en el archivo de configuración o establecerla explícitamente mediante código.</span><span class="sxs-lookup"><span data-stu-id="06513-576">If the different contexts execute in separate AppDomains, you can use the built-in support for specifying configuration classes in the configuration file or set it explicitly using code.</span></span> <span data-ttu-id="06513-577">Si se deben ejecutar los contextos diferentes en el mismo dominio de aplicación, se requerirá una solución personalizada.</span><span class="sxs-lookup"><span data-stu-id="06513-577">If the different contexts must execute in the same AppDomain, a custom solution will be required.</span></span>

<span data-ttu-id="06513-578">Para obtener más información, consulte [Configuración basada en código (EF6 y versiones posteriores)](http://msdn.microsoft.com/data/jj680699.aspx).</span><span class="sxs-lookup"><span data-stu-id="06513-578">For more information, see [Code-Based Configuration (EF6 onwards)](http://msdn.microsoft.com/data/jj680699.aspx).</span></span>

<span data-ttu-id="06513-579">La siguiente tabla muestra la configuración predeterminada de las directivas de reintento integradas cuando se usa EF6.</span><span class="sxs-lookup"><span data-stu-id="06513-579">The following table shows the default settings for the built-in retry policy when using EF6.</span></span>

| <span data-ttu-id="06513-580">Configuración</span><span class="sxs-lookup"><span data-stu-id="06513-580">Setting</span></span> | <span data-ttu-id="06513-581">Valor predeterminado</span><span class="sxs-lookup"><span data-stu-id="06513-581">Default value</span></span> | <span data-ttu-id="06513-582">Significado</span><span class="sxs-lookup"><span data-stu-id="06513-582">Meaning</span></span> |
|---------|---------------|---------|
| <span data-ttu-id="06513-583">Directiva</span><span class="sxs-lookup"><span data-stu-id="06513-583">Policy</span></span> | <span data-ttu-id="06513-584">Exponencial</span><span class="sxs-lookup"><span data-stu-id="06513-584">Exponential</span></span> | <span data-ttu-id="06513-585">Retroceso exponencial.</span><span class="sxs-lookup"><span data-stu-id="06513-585">Exponential back-off.</span></span> |
| <span data-ttu-id="06513-586">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="06513-586">MaxRetryCount</span></span> | <span data-ttu-id="06513-587">5</span><span class="sxs-lookup"><span data-stu-id="06513-587">5</span></span> | <span data-ttu-id="06513-588">El número máximo de reintentos.</span><span class="sxs-lookup"><span data-stu-id="06513-588">The maximum number of retries.</span></span> |
| <span data-ttu-id="06513-589">MaxDelay</span><span class="sxs-lookup"><span data-stu-id="06513-589">MaxDelay</span></span> | <span data-ttu-id="06513-590">30 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-590">30 seconds</span></span> | <span data-ttu-id="06513-591">El retraso máximo entre los reintentos.</span><span class="sxs-lookup"><span data-stu-id="06513-591">The maximum delay between retries.</span></span> <span data-ttu-id="06513-592">Este valor no afecta al modo en que se calcula la serie de retrasos.</span><span class="sxs-lookup"><span data-stu-id="06513-592">This value does not affect how the series of delays are computed.</span></span> <span data-ttu-id="06513-593">Solo define un límite superior.</span><span class="sxs-lookup"><span data-stu-id="06513-593">It only defines an upper bound.</span></span> |
| <span data-ttu-id="06513-594">DefaultCoefficient</span><span class="sxs-lookup"><span data-stu-id="06513-594">DefaultCoefficient</span></span> | <span data-ttu-id="06513-595">1 segundo</span><span class="sxs-lookup"><span data-stu-id="06513-595">1 second</span></span> | <span data-ttu-id="06513-596">El coeficiente para el cálculo del retroceso exponencial.</span><span class="sxs-lookup"><span data-stu-id="06513-596">The coefficient for the exponential back-off computation.</span></span> <span data-ttu-id="06513-597">No se puede cambiar este valor.</span><span class="sxs-lookup"><span data-stu-id="06513-597">This value cannot be changed.</span></span> |
| <span data-ttu-id="06513-598">DefaultRandomFactor</span><span class="sxs-lookup"><span data-stu-id="06513-598">DefaultRandomFactor</span></span> | <span data-ttu-id="06513-599">1.1</span><span class="sxs-lookup"><span data-stu-id="06513-599">1.1</span></span> | <span data-ttu-id="06513-600">El multiplicador utilizado para agregar un retraso aleatorio para cada entrada.</span><span class="sxs-lookup"><span data-stu-id="06513-600">The multiplier used to add a random delay for each entry.</span></span> <span data-ttu-id="06513-601">No se puede cambiar este valor.</span><span class="sxs-lookup"><span data-stu-id="06513-601">This value cannot be changed.</span></span> |
| <span data-ttu-id="06513-602">DefaultExponentialBase</span><span class="sxs-lookup"><span data-stu-id="06513-602">DefaultExponentialBase</span></span> | <span data-ttu-id="06513-603">2</span><span class="sxs-lookup"><span data-stu-id="06513-603">2</span></span> | <span data-ttu-id="06513-604">El multiplicador utilizado para calcular el retraso siguiente.</span><span class="sxs-lookup"><span data-stu-id="06513-604">The multiplier used to calculate the next delay.</span></span> <span data-ttu-id="06513-605">No se puede cambiar este valor.</span><span class="sxs-lookup"><span data-stu-id="06513-605">This value cannot be changed.</span></span> |

### <a name="retry-usage-guidance"></a><span data-ttu-id="06513-606">Instrucciones de uso del reintento</span><span class="sxs-lookup"><span data-stu-id="06513-606">Retry usage guidance</span></span>
<span data-ttu-id="06513-607">Al obtener acceso a la SQL Database mediante EF6, tenga en cuenta las siguientes directrices:</span><span class="sxs-lookup"><span data-stu-id="06513-607">Consider the following guidelines when accessing SQL Database using EF6:</span></span>

* <span data-ttu-id="06513-608">Elija la opción de servicio adecuada (compartida o premium).</span><span class="sxs-lookup"><span data-stu-id="06513-608">Choose the appropriate service option (shared or premium).</span></span> <span data-ttu-id="06513-609">Una instancia compartida puede sufrir retrasos en la conexión más largos de lo habitual y limitaciones debido al uso de otros inquilinos del servidor compartido.</span><span class="sxs-lookup"><span data-stu-id="06513-609">A shared instance may suffer longer than usual connection delays and throttling due to the usage by other tenants of the shared server.</span></span> <span data-ttu-id="06513-610">Si se requieren un rendimiento predecible y operaciones de latencia baja confiable, considere la posibilidad de elegir la opción premium.</span><span class="sxs-lookup"><span data-stu-id="06513-610">If predictable performance and reliable low latency operations are required, consider choosing the premium option.</span></span>
* <span data-ttu-id="06513-611">No se recomienda una estrategia de intervalo fijo para su uso con Azure SQL Database.</span><span class="sxs-lookup"><span data-stu-id="06513-611">A fixed interval strategy is not recommended for use with Azure SQL Database.</span></span> <span data-ttu-id="06513-612">En su lugar, use una estrategia de retroceso exponencial debido a que el servicio esté sobrecargado, y retrasos más prolongados permiten un mayor tiempo para la recuperación.</span><span class="sxs-lookup"><span data-stu-id="06513-612">Instead, use an exponential back-off strategy because the service may be overloaded, and longer delays allow more time for it to recover.</span></span>
* <span data-ttu-id="06513-613">Elija un valor adecuado para los tiempos de espera de conexión y comando a la hora de definir las conexiones.</span><span class="sxs-lookup"><span data-stu-id="06513-613">Choose a suitable value for the connection and command timeouts when defining connections.</span></span> <span data-ttu-id="06513-614">Base el tiempo de espera en el diseño de la lógica de negocios y a través de pruebas.</span><span class="sxs-lookup"><span data-stu-id="06513-614">Base the timeout on both your business logic design and through testing.</span></span> <span data-ttu-id="06513-615">Puede que necesite modificar este valor con el tiempo a medida que los volúmenes de datos o los procesos empresariales cambien.</span><span class="sxs-lookup"><span data-stu-id="06513-615">You may need to modify this value over time as the volumes of data or the business processes change.</span></span> <span data-ttu-id="06513-616">Un tiempo de espera demasiado corto puede provocar errores prematuros en las conexiones cuando la base de datos está ocupada.</span><span class="sxs-lookup"><span data-stu-id="06513-616">Too short a timeout may result in premature failures of connections when the database is busy.</span></span> <span data-ttu-id="06513-617">Un tiempo de espera demasiado largo puede impedir que la lógica de reintento funcione correctamente esperando demasiado tiempo antes de detectar un error en la conexión.</span><span class="sxs-lookup"><span data-stu-id="06513-617">Too long a timeout may prevent the retry logic working correctly by waiting too long before detecting a failed connection.</span></span> <span data-ttu-id="06513-618">El valor de tiempo de espera es un componente de la latencia de extremo a extremo, aunque no podrá determinar fácilmente cuántos comandos se ejecutarán cuando se guarde el contexto.</span><span class="sxs-lookup"><span data-stu-id="06513-618">The value of the timeout is a component of the end-to-end latency, although you cannot easily determine how many commands will execute when saving the context.</span></span> <span data-ttu-id="06513-619">Puede cambiar el tiempo de espera predeterminado estableciendo la propiedad **CommandTimeout** de la instancia **DbContext**.</span><span class="sxs-lookup"><span data-stu-id="06513-619">You can change the default timeout by setting the **CommandTimeout** property of the **DbContext** instance.</span></span>
* <span data-ttu-id="06513-620">Entity Framework admite configuraciones de reintento definidas en archivos de configuración.</span><span class="sxs-lookup"><span data-stu-id="06513-620">Entity Framework supports retry configurations defined in configuration files.</span></span> <span data-ttu-id="06513-621">Sin embargo, para obtener la máxima flexibilidad en Azure puede crear la configuración mediante programación dentro de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="06513-621">However, for maximum flexibility on Azure you should consider creating the configuration programmatically within the application.</span></span> <span data-ttu-id="06513-622">Los parámetros específicos para las directivas de reintento, como el número de reintentos y los intervalos de reintento, se pueden almacenar en el archivo de configuración del servicio y usar en tiempo de ejecución para crear las directivas apropiadas.</span><span class="sxs-lookup"><span data-stu-id="06513-622">The specific parameters for the retry policies, such as the number of retries and the retry intervals, can be stored in the service configuration file and used at runtime to create the appropriate policies.</span></span> <span data-ttu-id="06513-623">Esto permite cambiar la configuración sin necesidad de que la aplicación se reinicie.</span><span class="sxs-lookup"><span data-stu-id="06513-623">This allows the settings to be changed without requiring the application to be restarted.</span></span>

<span data-ttu-id="06513-624">Considere la posibilidad de comenzar con la configuración siguiente para las operaciones de reintento.</span><span class="sxs-lookup"><span data-stu-id="06513-624">Consider starting with the following settings for retrying operations.</span></span> <span data-ttu-id="06513-625">No se puede especificar el retraso entre reintentos (es fijo y se genera como una secuencia exponencial).</span><span class="sxs-lookup"><span data-stu-id="06513-625">You cannot specify the delay between retry attempts (it is fixed and generated as an exponential sequence).</span></span> <span data-ttu-id="06513-626">Puede especificar solo los valores máximos, como se muestra aquí, a menos que cree una estrategia de reintento personalizada.</span><span class="sxs-lookup"><span data-stu-id="06513-626">You can specify only the maximum values, as shown here; unless you create a custom retry strategy.</span></span> <span data-ttu-id="06513-627">Esta es la configuración de propósito general, y debe supervisar las operaciones y ajustar los valores para adaptarlos a su propio escenario.</span><span class="sxs-lookup"><span data-stu-id="06513-627">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="06513-628">**Contexto**</span><span class="sxs-lookup"><span data-stu-id="06513-628">**Context**</span></span> | <span data-ttu-id="06513-629">**Destino de ejemplo E2E<br />latencia máxima**</span><span class="sxs-lookup"><span data-stu-id="06513-629">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="06513-630">**Directiva de reintentos**</span><span class="sxs-lookup"><span data-stu-id="06513-630">**Retry policy**</span></span> | <span data-ttu-id="06513-631">**Configuración**</span><span class="sxs-lookup"><span data-stu-id="06513-631">**Settings**</span></span> | <span data-ttu-id="06513-632">**Valores**</span><span class="sxs-lookup"><span data-stu-id="06513-632">**Values**</span></span> | <span data-ttu-id="06513-633">**Cómo funciona**</span><span class="sxs-lookup"><span data-stu-id="06513-633">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="06513-634">Interactivo, interfaz de usuario</span><span class="sxs-lookup"><span data-stu-id="06513-634">Interactive, UI,</span></span><br /><span data-ttu-id="06513-635">o primer plano</span><span class="sxs-lookup"><span data-stu-id="06513-635">or foreground</span></span> |<span data-ttu-id="06513-636">2 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-636">2 seconds</span></span> |<span data-ttu-id="06513-637">Exponencial</span><span class="sxs-lookup"><span data-stu-id="06513-637">Exponential</span></span> |<span data-ttu-id="06513-638">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="06513-638">MaxRetryCount</span></span><br /><span data-ttu-id="06513-639">MaxDelay</span><span class="sxs-lookup"><span data-stu-id="06513-639">MaxDelay</span></span> |<span data-ttu-id="06513-640">3</span><span class="sxs-lookup"><span data-stu-id="06513-640">3</span></span><br /><span data-ttu-id="06513-641">750 ms</span><span class="sxs-lookup"><span data-stu-id="06513-641">750 ms</span></span> |<span data-ttu-id="06513-642">Intento 1 - retraso de 0 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-642">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="06513-643">Intento 2 - retraso de 750 ms</span><span class="sxs-lookup"><span data-stu-id="06513-643">Attempt 2 - delay 750 ms</span></span><br /><span data-ttu-id="06513-644">Intento 3 – retraso de 750 ms</span><span class="sxs-lookup"><span data-stu-id="06513-644">Attempt 3 – delay 750 ms</span></span> |
| <span data-ttu-id="06513-645">Fondo</span><span class="sxs-lookup"><span data-stu-id="06513-645">Background</span></span><br /> <span data-ttu-id="06513-646">o proceso por lotes</span><span class="sxs-lookup"><span data-stu-id="06513-646">or batch</span></span> |<span data-ttu-id="06513-647">30 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-647">30 seconds</span></span> |<span data-ttu-id="06513-648">Exponencial</span><span class="sxs-lookup"><span data-stu-id="06513-648">Exponential</span></span> |<span data-ttu-id="06513-649">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="06513-649">MaxRetryCount</span></span><br /><span data-ttu-id="06513-650">MaxDelay</span><span class="sxs-lookup"><span data-stu-id="06513-650">MaxDelay</span></span> |<span data-ttu-id="06513-651">5</span><span class="sxs-lookup"><span data-stu-id="06513-651">5</span></span><br /><span data-ttu-id="06513-652">12 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-652">12 seconds</span></span> |<span data-ttu-id="06513-653">Intento 1 - retraso de 0 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-653">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="06513-654">Intento 2 - retraso de ~1 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-654">Attempt 2 - delay ~1 sec</span></span><br /><span data-ttu-id="06513-655">Intento 3 - retraso de ~3 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-655">Attempt 3 - delay ~3 sec</span></span><br /><span data-ttu-id="06513-656">Intento 4 - retraso de ~7 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-656">Attempt 4 - delay ~7 sec</span></span><br /><span data-ttu-id="06513-657">Intento 5 - retraso de 12 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-657">Attempt 5 - delay 12 sec</span></span> |

> [!NOTE]
> <span data-ttu-id="06513-658">Los destinos de latencia de extremo a extremo suponen el tiempo de espera predeterminado para las conexiones con el servicio.</span><span class="sxs-lookup"><span data-stu-id="06513-658">The end-to-end latency targets assume the default timeout for connections to the service.</span></span> <span data-ttu-id="06513-659">Si especifica tiempos de espera de conexión más largos, la latencia de extremo a extremo se extenderá este tiempo adicional en cada reintento.</span><span class="sxs-lookup"><span data-stu-id="06513-659">If you specify longer connection timeouts, the end-to-end latency will be extended by this additional time for every retry attempt.</span></span>
>
>

### <a name="examples"></a><span data-ttu-id="06513-660">Ejemplos</span><span class="sxs-lookup"><span data-stu-id="06513-660">Examples</span></span>
<span data-ttu-id="06513-661">En el ejemplo de código siguiente se define una solución de acceso de datos simple que utiliza Entity Framework.</span><span class="sxs-lookup"><span data-stu-id="06513-661">The following code example defines a simple data access solution that uses Entity Framework.</span></span> <span data-ttu-id="06513-662">Establece una estrategia de reintento específica mediante la definición de una instancia de una clase denominada **BlogConfiguration** que extiende **DbConfiguration**.</span><span class="sxs-lookup"><span data-stu-id="06513-662">It sets a specific retry strategy by defining an instance of a class named **BlogConfiguration** that extends **DbConfiguration**.</span></span>

```csharp
using System;
using System.Collections.Generic;
using System.Data.Entity;
using System.Data.Entity.SqlServer;
using System.Threading.Tasks;

namespace RetryCodeSamples
{
    public class BlogConfiguration : DbConfiguration
    {
        public BlogConfiguration()
        {
            // Set up the execution strategy for SQL Database (exponential) with 5 retries and 12 sec delay.
            // These values could be loaded from configuration rather than being hard-coded.
            this.SetExecutionStrategy(
                    "System.Data.SqlClient", () => new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(12)));
        }
    }

    // Specify the configuration type if more than one has been defined.
    // [DbConfigurationType(typeof(BlogConfiguration))]
    public class BloggingContext : DbContext
    {
        // Definition of content goes here.
    }

    class EF6CodeSamples
    {
        public async static Task Samples()
        {
            // Execution strategy configured by DbConfiguration subclass, discovered automatically or
            // or explicitly indicated through configuration or with an attribute. Default is no retries.
            using (var db = new BloggingContext("Blogs"))
            {
                // Add, edit, delete blog items here, then:
                await db.SaveChangesAsync();
            }
        }
    }
}
```

<span data-ttu-id="06513-663">Más ejemplos de uso del mecanismo de reintento de Entity Framework se pueden encontrar en [Resistencia de la conexión/lógica de reintento](http://msdn.microsoft.com/data/dn456835.aspx).</span><span class="sxs-lookup"><span data-stu-id="06513-663">More examples of using the Entity Framework retry mechanism can be found in [Connection Resiliency / Retry Logic](http://msdn.microsoft.com/data/dn456835.aspx).</span></span>

### <a name="more-information"></a><span data-ttu-id="06513-664">Más información</span><span class="sxs-lookup"><span data-stu-id="06513-664">More information</span></span>
* [<span data-ttu-id="06513-665">Guía sobre rendimiento y elasticidad de Azure SQL Database</span><span class="sxs-lookup"><span data-stu-id="06513-665">Azure SQL Database Performance and Elasticity Guide</span></span>](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx)

## <a name="sql-database-using-entity-framework-core"></a><span data-ttu-id="06513-666">SQL Database mediante Entity Framework Core</span><span class="sxs-lookup"><span data-stu-id="06513-666">SQL Database using Entity Framework Core</span></span>
<span data-ttu-id="06513-667">[Entity Framework Core](/ef/core/) es un mapeador relacional de objetos que permite a los desarrolladores de .NET trabajar con datos usando objetos específicos del dominio.</span><span class="sxs-lookup"><span data-stu-id="06513-667">[Entity Framework Core](/ef/core/) is an object-relational mapper that enables .NET Core developers to work with data using domain-specific objects.</span></span> <span data-ttu-id="06513-668">Elimina la necesidad de usar la mayoría del código de acceso a datos que los programadores suelen tener que escribir.</span><span class="sxs-lookup"><span data-stu-id="06513-668">It eliminates the need for most of the data-access code that developers usually need to write.</span></span> <span data-ttu-id="06513-669">Esta versión de Entity Framework se escribió desde el principio y no hereda automáticamente todas las características de EF6.x.</span><span class="sxs-lookup"><span data-stu-id="06513-669">This version of Entity Framework was written from the ground up, and doesn't automatically inherit all the features from EF6.x.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="06513-670">Mecanismo de reintento</span><span class="sxs-lookup"><span data-stu-id="06513-670">Retry mechanism</span></span>
<span data-ttu-id="06513-671">La compatibilidad con los reintentos se proporciona al obtener acceso a SQL Database mediante Entity Framework Core mediante un mecanismo denominado [Resistencia de la conexión](/ef/core/miscellaneous/connection-resiliency).</span><span class="sxs-lookup"><span data-stu-id="06513-671">Retry support is provided when accessing SQL Database using Entity Framework Core through a mechanism called [Connection Resiliency](/ef/core/miscellaneous/connection-resiliency).</span></span> <span data-ttu-id="06513-672">La resistencia de la conexión se incorporó por primera vez en EF Core 1.1.0.</span><span class="sxs-lookup"><span data-stu-id="06513-672">Connection resiliency was introduced in EF Core 1.1.0.</span></span>

<span data-ttu-id="06513-673">La abstracción principal es la interfaz `IExecutionStrategy`.</span><span class="sxs-lookup"><span data-stu-id="06513-673">The primary abstraction is the `IExecutionStrategy` interface.</span></span> <span data-ttu-id="06513-674">La estrategia de ejecución para SQL Server, incluido SQL Azure, conoce los tipos de excepción que se pueden recuperar y tiene valores predeterminados razonables para el número máximo de reintentos o el retraso entre los reintentos, entre otros.</span><span class="sxs-lookup"><span data-stu-id="06513-674">The execution strategy for SQL Server, including SQL Azure, is aware of the exception types that can be retried and has sensible defaults for maximum retries, delay between retries, and so on.</span></span>

### <a name="examples"></a><span data-ttu-id="06513-675">Ejemplos</span><span class="sxs-lookup"><span data-stu-id="06513-675">Examples</span></span>

<span data-ttu-id="06513-676">El código siguiente permite reintentos automáticos cuando se configura el objeto DbContext, que representa una sesión con la base de datos.</span><span class="sxs-lookup"><span data-stu-id="06513-676">The following code enables automatic retries when configuring the DbContext object, which represents a session with the database.</span></span> 

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
        .UseSqlServer(
            @"Server=(localdb)\mssqllocaldb;Database=EFMiscellanous.ConnectionResiliency;Trusted_Connection=True;",
            options => options.EnableRetryOnFailure());
}
```

<span data-ttu-id="06513-677">El código siguiente muestra cómo ejecutar una transacción con reintentos automáticos, mediante el uso de una estrategia de ejecución.</span><span class="sxs-lookup"><span data-stu-id="06513-677">The following code shows how to execute a transaction with automatic retries, by using an execution strategy.</span></span> <span data-ttu-id="06513-678">La transacción se define en un delegado.</span><span class="sxs-lookup"><span data-stu-id="06513-678">The transaction is defined in a delegate.</span></span> <span data-ttu-id="06513-679">Si se produce un error transitorio, la estrategia de ejecución invoca al delegado de nuevo.</span><span class="sxs-lookup"><span data-stu-id="06513-679">If a transient failure occurs, the execution strategy will invoke the delegate again.</span></span>

```csharp
using (var db = new BloggingContext())
{
    var strategy = db.Database.CreateExecutionStrategy();

    strategy.Execute(() =>
    {
        using (var transaction = db.Database.BeginTransaction())
        {
            db.Blogs.Add(new Blog { Url = "http://blogs.msdn.com/dotnet" });
            db.SaveChanges();

            db.Blogs.Add(new Blog { Url = "http://blogs.msdn.com/visualstudio" });
            db.SaveChanges();

            transaction.Commit();
        }
    });
}
```

## <a name="azure-storage"></a><span data-ttu-id="06513-680">Azure Storage</span><span class="sxs-lookup"><span data-stu-id="06513-680">Azure Storage</span></span>
<span data-ttu-id="06513-681">Los servicios de almacenamiento de Azure incluyen almacenamiento de tablas y blobs, archivos y colas de almacenamiento.</span><span class="sxs-lookup"><span data-stu-id="06513-681">Azure storage services include table and blob storage, files, and storage queues.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="06513-682">Mecanismo de reintento</span><span class="sxs-lookup"><span data-stu-id="06513-682">Retry mechanism</span></span>
<span data-ttu-id="06513-683">Los reintentos se producen en el nivel de operación REST individual y son parte integral de la implementación de la API de cliente.</span><span class="sxs-lookup"><span data-stu-id="06513-683">Retries occur at the individual REST operation level and are an integral part of the client API implementation.</span></span> <span data-ttu-id="06513-684">El SDK de almacenamiento de cliente usa clases que implementan la [Interfaz IExtendedRetryPolicy](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.aspx).</span><span class="sxs-lookup"><span data-stu-id="06513-684">The client storage SDK uses classes that implement the [IExtendedRetryPolicy Interface](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.aspx).</span></span>

<span data-ttu-id="06513-685">Hay diferentes implementaciones de la interfaz.</span><span class="sxs-lookup"><span data-stu-id="06513-685">There are different implementations of the interface.</span></span> <span data-ttu-id="06513-686">Los clientes de almacenamiento pueden elegir entre directivas diseñadas específicamente para el acceso a tablas, blobs y colas.</span><span class="sxs-lookup"><span data-stu-id="06513-686">Storage clients can choose from policies specifically designed for accessing tables, blobs, and queues.</span></span> <span data-ttu-id="06513-687">Cada implementación utiliza una estrategia de reintento diferente que define el intervalo de reintento y otros detalles.</span><span class="sxs-lookup"><span data-stu-id="06513-687">Each implementation uses a different retry strategy that essentially defines the retry interval and other details.</span></span>

<span data-ttu-id="06513-688">Las clases integradas proporcionan compatibilidad para intervalos de reintento lineales (retraso constante) y exponenciales de selección aleatoria.</span><span class="sxs-lookup"><span data-stu-id="06513-688">The built-in classes provide support for linear (constant delay) and exponential with randomization retry intervals.</span></span> <span data-ttu-id="06513-689">También hay una directiva de no realización de reintentos a usar cuando otro proceso está controlando los reintentos a un nivel superior.</span><span class="sxs-lookup"><span data-stu-id="06513-689">There is also a no retry policy for use when another process is handling retries at a higher level.</span></span> <span data-ttu-id="06513-690">Sin embargo, puede implementar sus propias clases de reintentos si tiene requisitos específicos no proporcionados por las clases integradas.</span><span class="sxs-lookup"><span data-stu-id="06513-690">However, you can implement your own retry classes if you have specific requirements not provided by the built-in classes.</span></span>

<span data-ttu-id="06513-691">Los reintentos alternativos cambian entre ubicación del servicio de almacenamiento principal y secundario si usa almacenamiento con redundancia geográfica con acceso de lectura (RA-GRS) y el resultado de la solicitud es un error que se puede reproducir.</span><span class="sxs-lookup"><span data-stu-id="06513-691">Alternate retries switch between primary and secondary storage service location if you are using read access geo-redundant storage (RA-GRS) and the result of the request is a retryable error.</span></span> <span data-ttu-id="06513-692">Para obtener más información, consulte [Opciones de redundancia de Azure Storage](http://msdn.microsoft.com/library/azure/dn727290.aspx) .</span><span class="sxs-lookup"><span data-stu-id="06513-692">See [Azure Storage Redundancy Options](http://msdn.microsoft.com/library/azure/dn727290.aspx) for more information.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="06513-693">Configuración de directivas</span><span class="sxs-lookup"><span data-stu-id="06513-693">Policy configuration</span></span>
<span data-ttu-id="06513-694">Las directivas de reintento se configuran mediante programación.</span><span class="sxs-lookup"><span data-stu-id="06513-694">Retry policies are configured programmatically.</span></span> <span data-ttu-id="06513-695">Un procedimiento típico consiste en crear y rellenar una instancia **TableRequestOptions**, **BlobRequestOptions**, **FileRequestOptions** o **QueueRequestOptions** instancia.</span><span class="sxs-lookup"><span data-stu-id="06513-695">A typical procedure is to create and populate a **TableRequestOptions**, **BlobRequestOptions**, **FileRequestOptions**, or **QueueRequestOptions** instance.</span></span>

```csharp
TableRequestOptions interactiveRequestOption = new TableRequestOptions()
{
  RetryPolicy = new LinearRetry(TimeSpan.FromMilliseconds(500), 3),
  // For Read-access geo-redundant storage, use PrimaryThenSecondary.
  // Otherwise set this to PrimaryOnly.
  LocationMode = LocationMode.PrimaryThenSecondary,
  // Maximum execution time based on the business use case. 
  MaximumExecutionTime = TimeSpan.FromSeconds(2)
};
```

<span data-ttu-id="06513-696">A continuación, la instancia de opciones de solicitud se puede establecer en el cliente y todas las operaciones con el cliente usarán las opciones de solicitud especificadas.</span><span class="sxs-lookup"><span data-stu-id="06513-696">The request options instance can then be set on the client, and all operations with the client will use the specified request options.</span></span>

```csharp
client.DefaultRequestOptions = interactiveRequestOption;
var stats = await client.GetServiceStatsAsync();
```

<span data-ttu-id="06513-697">Puede anular las opciones de solicitud de cliente pasando una instancia completada de la clase de opciones de solicitud como parámetro a los métodos de operación.</span><span class="sxs-lookup"><span data-stu-id="06513-697">You can override the client request options by passing a populated instance of the request options class as a parameter to operation methods.</span></span>

```csharp
var stats = await client.GetServiceStatsAsync(interactiveRequestOption, operationContext: null);
```

<span data-ttu-id="06513-698">Puede usar una instancia **OperationContext** para especificar el código para ejecutar cuando se produce un reintento y cuando se ha completado una operación.</span><span class="sxs-lookup"><span data-stu-id="06513-698">You use an **OperationContext** instance to specify the code to execute when a retry occurs and when an operation has completed.</span></span> <span data-ttu-id="06513-699">Este código puede recopilar información acerca de la operación para su uso en registros y telemetría.</span><span class="sxs-lookup"><span data-stu-id="06513-699">This code can collect information about the operation for use in logs and telemetry.</span></span>

```csharp
// Set up notifications for an operation
var context = new OperationContext();
context.ClientRequestID = "some request id";
context.Retrying += (sender, args) =>
{
    /* Collect retry information */
};
context.RequestCompleted += (sender, args) =>
{
    /* Collect operation completion information */
};
var stats = await client.GetServiceStatsAsync(null, context);
```

<span data-ttu-id="06513-700">Además de indicar si se puede volver a intentar un error, las directivas de reintento ampliadas devuelven un objeto **RetryContext** que indica el número de reintentos, los resultados de la última solicitud y si se realizará el siguiente reintento en la ubicación principal o secundaria (consulte la tabla siguiente para obtener más información).</span><span class="sxs-lookup"><span data-stu-id="06513-700">In addition to indicating whether a failure is suitable for retry, the extended retry policies return a **RetryContext** object that indicates the number of retries, the results of the last request, whether the next retry will happen in the primary or secondary location (see table below for details).</span></span> <span data-ttu-id="06513-701">Las propiedades del objeto **RetryContext** pueden usarse para decidir cuándo y si realizar un reintento.</span><span class="sxs-lookup"><span data-stu-id="06513-701">The properties of the **RetryContext** object can be used to decide if and when to attempt a retry.</span></span> <span data-ttu-id="06513-702">Para obtener más información, consulte [Método IExtendedRetryPolicy.Evaluate](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.evaluate.aspx).</span><span class="sxs-lookup"><span data-stu-id="06513-702">For more details, see [IExtendedRetryPolicy.Evaluate Method](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.evaluate.aspx).</span></span>

<span data-ttu-id="06513-703">La siguiente tabla muestra la configuración predeterminada de las directivas de reintento integradas.</span><span class="sxs-lookup"><span data-stu-id="06513-703">The following tables show the default settings for the built-in retry policies.</span></span>

<span data-ttu-id="06513-704">**Opciones de solicitud**</span><span class="sxs-lookup"><span data-stu-id="06513-704">**Request options**</span></span>

| <span data-ttu-id="06513-705">**Configuración**</span><span class="sxs-lookup"><span data-stu-id="06513-705">**Setting**</span></span> | <span data-ttu-id="06513-706">**Valor predeterminado**</span><span class="sxs-lookup"><span data-stu-id="06513-706">**Default value**</span></span> | <span data-ttu-id="06513-707">**Significado**</span><span class="sxs-lookup"><span data-stu-id="06513-707">**Meaning**</span></span> |
| --- | --- | --- |
| <span data-ttu-id="06513-708">MaximumExecutionTime</span><span class="sxs-lookup"><span data-stu-id="06513-708">MaximumExecutionTime</span></span> | <span data-ttu-id="06513-709">None</span><span class="sxs-lookup"><span data-stu-id="06513-709">None</span></span> | <span data-ttu-id="06513-710">Tiempo de ejecución máximo para la solicitud, incluidos todos los posibles reintentos.</span><span class="sxs-lookup"><span data-stu-id="06513-710">Maximum execution time for the request, including all potential retry attempts.</span></span> <span data-ttu-id="06513-711">Si no se especifica, el periodo que se permite a una solicitud es ilimitado.</span><span class="sxs-lookup"><span data-stu-id="06513-711">If it is not specified, then the amount of time that a request is permitted to take is unlimited.</span></span> <span data-ttu-id="06513-712">En otras palabras, la solicitud podría dejar de responder.</span><span class="sxs-lookup"><span data-stu-id="06513-712">In other words, the request might hang.</span></span> |
| <span data-ttu-id="06513-713">ServerTimeout</span><span class="sxs-lookup"><span data-stu-id="06513-713">ServerTimeout</span></span> | <span data-ttu-id="06513-714">None</span><span class="sxs-lookup"><span data-stu-id="06513-714">None</span></span> | <span data-ttu-id="06513-715">Intervalo de tiempo de espera del servidor para la solicitud (el valor se redondea en segundos).</span><span class="sxs-lookup"><span data-stu-id="06513-715">Server timeout interval for the request (value is rounded to seconds).</span></span> <span data-ttu-id="06513-716">Si no se especifica, usará el valor predeterminado para todas las solicitudes al servidor.</span><span class="sxs-lookup"><span data-stu-id="06513-716">If not specified, it will use the default value for all requests to the server.</span></span> <span data-ttu-id="06513-717">Normalmente, la mejor opción es omitir este valor para que se utilice la configuración predeterminada del servidor.</span><span class="sxs-lookup"><span data-stu-id="06513-717">Usually, the best option is to omit this setting so that the server default is used.</span></span> | 
| <span data-ttu-id="06513-718">LocationMode</span><span class="sxs-lookup"><span data-stu-id="06513-718">LocationMode</span></span> | <span data-ttu-id="06513-719">None</span><span class="sxs-lookup"><span data-stu-id="06513-719">None</span></span> | <span data-ttu-id="06513-720">Si la cuenta de almacenamiento se crea con la opción de replicación del almacenamiento con redundancia geográfica de acceso de lectura (RA-GRS), puede usar el modo de ubicación para indicar qué ubicación debe recibir la solicitud.</span><span class="sxs-lookup"><span data-stu-id="06513-720">If the storage account is created with the Read access geo-redundant storage (RA-GRS) replication option, you can use the location mode to indicate which location should receive the request.</span></span> <span data-ttu-id="06513-721">Por ejemplo, si se especifica **PrimaryThenSecondary** , las solicitudes siempre se envían a la ubicación principal en primer lugar.</span><span class="sxs-lookup"><span data-stu-id="06513-721">For example, if **PrimaryThenSecondary** is specified, requests are always sent to the primary location first.</span></span> <span data-ttu-id="06513-722">Si se produce un error en una solicitud, se envía a la ubicación secundaria.</span><span class="sxs-lookup"><span data-stu-id="06513-722">If a request fails, it is sent to the secondary location.</span></span> |
| <span data-ttu-id="06513-723">RetryPolicy</span><span class="sxs-lookup"><span data-stu-id="06513-723">RetryPolicy</span></span> | <span data-ttu-id="06513-724">ExponentialPolicy</span><span class="sxs-lookup"><span data-stu-id="06513-724">ExponentialPolicy</span></span> | <span data-ttu-id="06513-725">Consulte a continuación los detalles de cada opción.</span><span class="sxs-lookup"><span data-stu-id="06513-725">See below for details of each option.</span></span> |

<span data-ttu-id="06513-726">**Directiva exponencial**</span><span class="sxs-lookup"><span data-stu-id="06513-726">**Exponential policy**</span></span> 

| <span data-ttu-id="06513-727">**Configuración**</span><span class="sxs-lookup"><span data-stu-id="06513-727">**Setting**</span></span> | <span data-ttu-id="06513-728">**Valor predeterminado**</span><span class="sxs-lookup"><span data-stu-id="06513-728">**Default value**</span></span> | <span data-ttu-id="06513-729">**Significado**</span><span class="sxs-lookup"><span data-stu-id="06513-729">**Meaning**</span></span> |
| --- | --- | --- |
| <span data-ttu-id="06513-730">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="06513-730">maxAttempt</span></span> | <span data-ttu-id="06513-731">3</span><span class="sxs-lookup"><span data-stu-id="06513-731">3</span></span> | <span data-ttu-id="06513-732">Número de reintentos.</span><span class="sxs-lookup"><span data-stu-id="06513-732">Number of retry attempts.</span></span> |
| <span data-ttu-id="06513-733">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="06513-733">deltaBackoff</span></span> | <span data-ttu-id="06513-734">4 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-734">4 seconds</span></span> | <span data-ttu-id="06513-735">Intervalo de espera entre reintentos.</span><span class="sxs-lookup"><span data-stu-id="06513-735">Back-off interval between retries.</span></span> <span data-ttu-id="06513-736">Se usarán múltiplos de este período de tiempo, incluyendo un elemento aleatorio, para los reintentos posteriores.</span><span class="sxs-lookup"><span data-stu-id="06513-736">Multiples of this timespan, including a random element, will be used for subsequent retry attempts.</span></span> |
| <span data-ttu-id="06513-737">MinBackoff</span><span class="sxs-lookup"><span data-stu-id="06513-737">MinBackoff</span></span> | <span data-ttu-id="06513-738">3 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-738">3 seconds</span></span> | <span data-ttu-id="06513-739">Agregado a todos los intervalos de reintento calculados a partir de deltaBackoff.</span><span class="sxs-lookup"><span data-stu-id="06513-739">Added to all retry intervals computed from deltaBackoff.</span></span> <span data-ttu-id="06513-740">No se puede cambiar este valor.</span><span class="sxs-lookup"><span data-stu-id="06513-740">This value cannot be changed.</span></span>
| <span data-ttu-id="06513-741">MaxBackoff</span><span class="sxs-lookup"><span data-stu-id="06513-741">MaxBackoff</span></span> | <span data-ttu-id="06513-742">120 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-742">120 seconds</span></span> | <span data-ttu-id="06513-743">MaxBackoff se usa si el intervalo de reintento calculado es mayor que MaxBackoff.</span><span class="sxs-lookup"><span data-stu-id="06513-743">MaxBackoff is used if the computed retry interval is greater than MaxBackoff.</span></span> <span data-ttu-id="06513-744">No se puede cambiar este valor.</span><span class="sxs-lookup"><span data-stu-id="06513-744">This value cannot be changed.</span></span> |

<span data-ttu-id="06513-745">**Directiva lineal**</span><span class="sxs-lookup"><span data-stu-id="06513-745">**Linear policy**</span></span>

| <span data-ttu-id="06513-746">**Configuración**</span><span class="sxs-lookup"><span data-stu-id="06513-746">**Setting**</span></span> | <span data-ttu-id="06513-747">**Valor predeterminado**</span><span class="sxs-lookup"><span data-stu-id="06513-747">**Default value**</span></span> | <span data-ttu-id="06513-748">**Significado**</span><span class="sxs-lookup"><span data-stu-id="06513-748">**Meaning**</span></span> |
| --- | --- | --- |
| <span data-ttu-id="06513-749">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="06513-749">maxAttempt</span></span> | <span data-ttu-id="06513-750">3</span><span class="sxs-lookup"><span data-stu-id="06513-750">3</span></span> | <span data-ttu-id="06513-751">Número de reintentos.</span><span class="sxs-lookup"><span data-stu-id="06513-751">Number of retry attempts.</span></span> |
| <span data-ttu-id="06513-752">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="06513-752">deltaBackoff</span></span> | <span data-ttu-id="06513-753">30 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-753">30 seconds</span></span> | <span data-ttu-id="06513-754">Intervalo de espera entre reintentos.</span><span class="sxs-lookup"><span data-stu-id="06513-754">Back-off interval between retries.</span></span> |

### <a name="retry-usage-guidance"></a><span data-ttu-id="06513-755">Instrucciones de uso del reintento</span><span class="sxs-lookup"><span data-stu-id="06513-755">Retry usage guidance</span></span>
<span data-ttu-id="06513-756">Al obtener acceso a los servicios de almacenamiento de Azure mediante la API de cliente de almacenamiento, tenga en cuenta las siguientes directrices:</span><span class="sxs-lookup"><span data-stu-id="06513-756">Consider the following guidelines when accessing Azure storage services using the storage client API:</span></span>

* <span data-ttu-id="06513-757">Use las directivas de reintento integradas del espacio de nombres Microsoft.WindowsAzure.Storage.RetryPolicies donde son adecuadas para sus requisitos.</span><span class="sxs-lookup"><span data-stu-id="06513-757">Use the built-in retry policies from the Microsoft.WindowsAzure.Storage.RetryPolicies namespace where they are appropriate for your requirements.</span></span> <span data-ttu-id="06513-758">En la mayoría de los casos, estas directivas serán suficientes.</span><span class="sxs-lookup"><span data-stu-id="06513-758">In most cases, these policies will be sufficient.</span></span>
* <span data-ttu-id="06513-759">Use la directiva **ExponentialRetry** en operaciones por lotes, tareas en segundo plano o escenarios no interactivos.</span><span class="sxs-lookup"><span data-stu-id="06513-759">Use the **ExponentialRetry** policy in batch operations, background tasks, or non-interactive scenarios.</span></span> <span data-ttu-id="06513-760">En estos escenarios, normalmente puede permitir más tiempo para recuperar el servicio (por consiguiente, con unas mayores posibilidades de que la operación se efectúe correctamente).</span><span class="sxs-lookup"><span data-stu-id="06513-760">In these scenarios, you can typically allow more time for the service to recover—with a consequently increased chance of the operation eventually succeeding.</span></span>
* <span data-ttu-id="06513-761">Considere la posibilidad de especificar la propiedad **MaximumExecutionTime** del parámetro **RequestOptions** para limitar el tiempo de ejecución total, pero tenga en cuenta el tipo y tamaño de la operación al elegir un valor de tiempo de espera.</span><span class="sxs-lookup"><span data-stu-id="06513-761">Consider specifying the **MaximumExecutionTime** property of the **RequestOptions** parameter to limit the total execution time, but take into account the type and size of the operation when choosing a timeout value.</span></span>
* <span data-ttu-id="06513-762">Si necesita implementar un reintento personalizado, evite crear contenedores en torno a las clases de cliente de almacenamiento.</span><span class="sxs-lookup"><span data-stu-id="06513-762">If you need to implement a custom retry, avoid creating wrappers around the storage client classes.</span></span> <span data-ttu-id="06513-763">En su lugar, use las capacidades para ampliar las directivas existentes a través de la interfaz **IExtendedRetryPolicy** .</span><span class="sxs-lookup"><span data-stu-id="06513-763">Instead, use the capabilities to extend the existing policies through the **IExtendedRetryPolicy** interface.</span></span>
* <span data-ttu-id="06513-764">Si utiliza almacenamiento con redundancia geográfica con acceso de lectura (RA-GRS) puede usar el **LocationMode** para especificar que los reintentos tengan acceso a la copia de solo lectura secundaria de la tienda en caso de error en el acceso principal.</span><span class="sxs-lookup"><span data-stu-id="06513-764">If you are using read access geo-redundant storage (RA-GRS) you can use the **LocationMode** to specify that retry attempts will access the secondary read-only copy of the store should the primary access fail.</span></span> <span data-ttu-id="06513-765">Sin embargo, al utilizar esta opción debe asegurarse de que la aplicación pueda trabajar correctamente con datos que pueden ser obsoletos si todavía no ha completado la replicación desde el almacén principal.</span><span class="sxs-lookup"><span data-stu-id="06513-765">However, when using this option you must ensure that your application can work successfully with data that may be stale if the replication from the primary store has not yet completed.</span></span>

<span data-ttu-id="06513-766">Considere la posibilidad de comenzar con la configuración siguiente para volver a intentar las operaciones.</span><span class="sxs-lookup"><span data-stu-id="06513-766">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="06513-767">Esta es la configuración de propósito general, y debe supervisar las operaciones y ajustar los valores para adaptarlos a su propio escenario.</span><span class="sxs-lookup"><span data-stu-id="06513-767">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>  

| <span data-ttu-id="06513-768">**Contexto**</span><span class="sxs-lookup"><span data-stu-id="06513-768">**Context**</span></span> | <span data-ttu-id="06513-769">**Destino de ejemplo E2E<br />latencia máxima**</span><span class="sxs-lookup"><span data-stu-id="06513-769">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="06513-770">**Directiva de reintentos**</span><span class="sxs-lookup"><span data-stu-id="06513-770">**Retry policy**</span></span> | <span data-ttu-id="06513-771">**Configuración**</span><span class="sxs-lookup"><span data-stu-id="06513-771">**Settings**</span></span> | <span data-ttu-id="06513-772">**Valores**</span><span class="sxs-lookup"><span data-stu-id="06513-772">**Values**</span></span> | <span data-ttu-id="06513-773">**Cómo funciona**</span><span class="sxs-lookup"><span data-stu-id="06513-773">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="06513-774">Interactivo, interfaz de usuario</span><span class="sxs-lookup"><span data-stu-id="06513-774">Interactive, UI,</span></span><br /><span data-ttu-id="06513-775">o primer plano</span><span class="sxs-lookup"><span data-stu-id="06513-775">or foreground</span></span> |<span data-ttu-id="06513-776">2 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-776">2 seconds</span></span> |<span data-ttu-id="06513-777">Lineal</span><span class="sxs-lookup"><span data-stu-id="06513-777">Linear</span></span> |<span data-ttu-id="06513-778">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="06513-778">maxAttempt</span></span><br /><span data-ttu-id="06513-779">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="06513-779">deltaBackoff</span></span> |<span data-ttu-id="06513-780">3</span><span class="sxs-lookup"><span data-stu-id="06513-780">3</span></span><br /><span data-ttu-id="06513-781">500 ms</span><span class="sxs-lookup"><span data-stu-id="06513-781">500 ms</span></span> |<span data-ttu-id="06513-782">Intento 1 - retraso de 500 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-782">Attempt 1 - delay 500 ms</span></span><br /><span data-ttu-id="06513-783">Intento 2 - retraso de 500 ms</span><span class="sxs-lookup"><span data-stu-id="06513-783">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="06513-784">Intento 3 – retraso de 500 ms</span><span class="sxs-lookup"><span data-stu-id="06513-784">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="06513-785">Fondo</span><span class="sxs-lookup"><span data-stu-id="06513-785">Background</span></span><br /><span data-ttu-id="06513-786">o proceso por lotes</span><span class="sxs-lookup"><span data-stu-id="06513-786">or batch</span></span> |<span data-ttu-id="06513-787">30 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-787">30 seconds</span></span> |<span data-ttu-id="06513-788">Exponencial</span><span class="sxs-lookup"><span data-stu-id="06513-788">Exponential</span></span> |<span data-ttu-id="06513-789">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="06513-789">maxAttempt</span></span><br /><span data-ttu-id="06513-790">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="06513-790">deltaBackoff</span></span> |<span data-ttu-id="06513-791">5</span><span class="sxs-lookup"><span data-stu-id="06513-791">5</span></span><br /><span data-ttu-id="06513-792">4 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-792">4 seconds</span></span> |<span data-ttu-id="06513-793">Intento 1 - retraso de ~3 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-793">Attempt 1 - delay ~3 sec</span></span><br /><span data-ttu-id="06513-794">Intento 2 - retraso de ~7 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-794">Attempt 2 - delay ~7 sec</span></span><br /><span data-ttu-id="06513-795">Intento 3 - retraso de ~15 segundos</span><span class="sxs-lookup"><span data-stu-id="06513-795">Attempt 3 - delay ~15 sec</span></span> |

### <a name="telemetry"></a><span data-ttu-id="06513-796">Telemetría</span><span class="sxs-lookup"><span data-stu-id="06513-796">Telemetry</span></span>
<span data-ttu-id="06513-797">Los reintentos se registran en un **TraceSource**.</span><span class="sxs-lookup"><span data-stu-id="06513-797">Retry attempts are logged to a **TraceSource**.</span></span> <span data-ttu-id="06513-798">Debe configurar un **TraceListener** para capturar los eventos y escribirlos en un registro de destino adecuado.</span><span class="sxs-lookup"><span data-stu-id="06513-798">You must configure a **TraceListener** to capture the events and write them to a suitable destination log.</span></span> <span data-ttu-id="06513-799">Puede usar el **TextWriterTraceListener** o **XmlWriterTraceListener** para escribir los datos en un archivo de registro, el **EventLogTraceListener** para escribir en el registro de eventos de Windows o el **EventProviderTraceListener** para escribir los datos de seguimiento en el subsistema ETW.</span><span class="sxs-lookup"><span data-stu-id="06513-799">You can use the **TextWriterTraceListener** or **XmlWriterTraceListener** to write the data to a log file, the **EventLogTraceListener** to write to the Windows Event Log, or the **EventProviderTraceListener** to write trace data to the ETW subsystem.</span></span> <span data-ttu-id="06513-800">También puede configurar automáticamente el vaciado del búfer y el nivel de detalle de los eventos que se registrarán (por ejemplo, Error, Advertencia, Informativo y Detallado).</span><span class="sxs-lookup"><span data-stu-id="06513-800">You can also configure auto-flushing of the buffer, and the verbosity of events that will be logged (for example, Error, Warning, Informational, and Verbose).</span></span> <span data-ttu-id="06513-801">Para obtener más información, consulte [Registro de cliente con biblioteca de cliente de almacenamiento .NET](http://msdn.microsoft.com/library/azure/dn782839.aspx).</span><span class="sxs-lookup"><span data-stu-id="06513-801">For more information, see [Client-side Logging with the .NET Storage Client Library](http://msdn.microsoft.com/library/azure/dn782839.aspx).</span></span>

<span data-ttu-id="06513-802">Las operaciones pueden recibir una instancia **OperationContext**, que expone un evento de **Reintento** que puede usarse para adjuntar la lógica personalizada de telemetría.</span><span class="sxs-lookup"><span data-stu-id="06513-802">Operations can receive an **OperationContext** instance, which exposes a **Retrying** event that can be used to attach custom telemetry logic.</span></span> <span data-ttu-id="06513-803">Para obtener más información, consulte [Evento OperationContext.Retrying](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.operationcontext.retrying.aspx).</span><span class="sxs-lookup"><span data-stu-id="06513-803">For more information, see [OperationContext.Retrying Event](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.operationcontext.retrying.aspx).</span></span>

### <a name="examples"></a><span data-ttu-id="06513-804">Ejemplos</span><span class="sxs-lookup"><span data-stu-id="06513-804">Examples</span></span>
<span data-ttu-id="06513-805">En el ejemplo de código siguiente se muestra cómo crear dos instancias **TableRequestOptions** con diferentes configuraciones de reintento; una para solicitudes interactivas y otra para solicitudes en segundo plano.</span><span class="sxs-lookup"><span data-stu-id="06513-805">The following code example shows how to create two **TableRequestOptions** instances with different retry settings; one for interactive requests and one for background requests.</span></span> <span data-ttu-id="06513-806">A continuación, el ejemplo establece estas dos directivas de reintento en el cliente para que puedan aplicarse a todas las solicitudes y también establece la estrategia interactiva en una solicitud concreta para que reemplace la configuración predeterminada que se aplica al cliente.</span><span class="sxs-lookup"><span data-stu-id="06513-806">The example then sets these two retry policies on the client so that they apply for all requests, and also sets the interactive strategy on a specific request so that it overrides the default settings applied to the client.</span></span>

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.WindowsAzure.Storage;
using Microsoft.WindowsAzure.Storage.RetryPolicies;
using Microsoft.WindowsAzure.Storage.Table;

namespace RetryCodeSamples
{
    class AzureStorageCodeSamples
    {
        private const string connectionString = "UseDevelopmentStorage=true";

        public async static Task Samples()
        {
            var storageAccount = CloudStorageAccount.Parse(connectionString);

            TableRequestOptions interactiveRequestOption = new TableRequestOptions()
            {
                RetryPolicy = new LinearRetry(TimeSpan.FromMilliseconds(500), 3),
                // For Read-access geo-redundant storage, use PrimaryThenSecondary.
                // Otherwise set this to PrimaryOnly.
                LocationMode = LocationMode.PrimaryThenSecondary,
                // Maximum execution time based on the business use case. 
                MaximumExecutionTime = TimeSpan.FromSeconds(2)
            };

            TableRequestOptions backgroundRequestOption = new TableRequestOptions()
            {
                // Client has a default exponential retry policy with 4 sec delay and 3 retry attempts
                // Retry delays will be approximately 3 sec, 7 sec, and 15 sec
                MaximumExecutionTime = TimeSpan.FromSeconds(30),
                // PrimaryThenSecondary in case of Read-access geo-redundant storage, else set this to PrimaryOnly
                LocationMode = LocationMode.PrimaryThenSecondary
            };

            var client = storageAccount.CreateCloudTableClient();
            // Client has a default exponential retry policy with 4 sec delay and 3 retry attempts
            // Retry delays will be approximately 3 sec, 7 sec, and 15 sec
            // ServerTimeout and MaximumExecutionTime are not set

            {
                // Set properties for the client (used on all requests unless overridden)
                // Different exponential policy parameters for background scenarios
                client.DefaultRequestOptions = backgroundRequestOption;
                // Linear policy for interactive scenarios
                client.DefaultRequestOptions = interactiveRequestOption;
            }

            {
                // set properties for a specific request
                var stats = await client.GetServiceStatsAsync(interactiveRequestOption, operationContext: null);
            }

            {
                // Set up notifications for an operation
                var context = new OperationContext();
                context.ClientRequestID = "some request id";
                context.Retrying += (sender, args) =>
                {
                    /* Collect retry information */
                };
                context.RequestCompleted += (sender, args) =>
                {
                    /* Collect operation completion information */
                };
                var stats = await client.GetServiceStatsAsync(null, context);
            }
        }
    }
}
```

### <a name="more-information"></a><span data-ttu-id="06513-807">Más información</span><span class="sxs-lookup"><span data-stu-id="06513-807">More information</span></span>
* [<span data-ttu-id="06513-808">Recomendaciones de la directiva de reintento de biblioteca de cliente de Azure Storage</span><span class="sxs-lookup"><span data-stu-id="06513-808">Azure Storage Client Library Retry Policy Recommendations</span></span>](https://azure.microsoft.com/blog/2014/05/22/azure-storage-client-library-retry-policy-recommendations/)
* [<span data-ttu-id="06513-809">Biblioteca de cliente de almacenamiento 2.0: implementación de directivas de reintento</span><span class="sxs-lookup"><span data-stu-id="06513-809">Storage Client Library 2.0 – Implementing Retry Policies</span></span>](http://gauravmantri.com/2012/12/30/storage-client-library-2-0-implementing-retry-policies/)

## <a name="general-rest-and-retry-guidelines"></a><span data-ttu-id="06513-810">Directrices generales de REST y de reintento</span><span class="sxs-lookup"><span data-stu-id="06513-810">General REST and retry guidelines</span></span>
<span data-ttu-id="06513-811">Al obtener acceso a los servicios de Azure o de terceros, tenga en cuenta lo siguiente:</span><span class="sxs-lookup"><span data-stu-id="06513-811">Consider the following when accessing Azure or third party services:</span></span>

* <span data-ttu-id="06513-812">Use un enfoque sistemático para administrar reintentos, quizás como código reutilizable, para que pueda aplicar una metodología coherente en todos los clientes y todas las soluciones.</span><span class="sxs-lookup"><span data-stu-id="06513-812">Use a systematic approach to managing retries, perhaps as reusable code, so that you can apply a consistent methodology across all clients and all solutions.</span></span>
* <span data-ttu-id="06513-813">Considere el uso de un marco de reintento como el bloque de aplicaciones de control de errores transitorios para administrar reintentos si el servicio de destino o el cliente no tiene ningún mecanismo de reintento integrado.</span><span class="sxs-lookup"><span data-stu-id="06513-813">Consider using a retry framework such as the Transient Fault Handling Application Block to manage retries if the target service or client has no built-in retry mechanism.</span></span> <span data-ttu-id="06513-814">Esto le ayudará a implementar un comportamiento de reintento coherente y puede proporcionar una estrategia de reintento predeterminada adecuados para el servicio de destino.</span><span class="sxs-lookup"><span data-stu-id="06513-814">This will help you implement a consistent retry behavior, and it may provide a suitable default retry strategy for the target service.</span></span> <span data-ttu-id="06513-815">Sin embargo, puede que necesite crear código de reintento personalizado para los servicios que tienen un comportamiento no estándar, que no dependen de excepciones para indicar errores transitorios o si desea usar una respuesta de **respuesta de reintento** para administrar el comportamiento de reintento.</span><span class="sxs-lookup"><span data-stu-id="06513-815">However, you may need to create custom retry code for services that have non-standard behavior, that do not rely on exceptions to indicate transient failures, or if you want to use a **Retry-Response** reply to manage retry behavior.</span></span>
* <span data-ttu-id="06513-816">La lógica de detección transitoria dependerá de la API de cliente real que use para invocar las llamadas REST.</span><span class="sxs-lookup"><span data-stu-id="06513-816">The transient detection logic will depend on the actual client API you use to invoke the REST calls.</span></span> <span data-ttu-id="06513-817">Algunos clientes, como los de la clase **HttpClient** más reciente no producirán excepciones para las solicitudes completadas con un código de estado HTTP no correcto.</span><span class="sxs-lookup"><span data-stu-id="06513-817">Some clients, such as the newer **HttpClient** class, will not throw exceptions for completed requests with a non-success HTTP status code.</span></span> <span data-ttu-id="06513-818">Esto mejora el rendimiento, pero impide el uso del bloque de aplicaciones de control de errores transitorios.</span><span class="sxs-lookup"><span data-stu-id="06513-818">This improves performance but prevents the use of the Transient Fault Handling Application Block.</span></span> <span data-ttu-id="06513-819">En este caso podría encapsular la llamada a la API de REST con código que produce excepciones para códigos de estado HTTP no correctos que, a continuación, pueden ser procesados por el bloque.</span><span class="sxs-lookup"><span data-stu-id="06513-819">In this case you could wrap the call to the REST API with code that produces exceptions for non-success HTTP status codes, which can then be processed by the block.</span></span> <span data-ttu-id="06513-820">Como alternativa, puede usar un mecanismo diferente para controlar los reintentos.</span><span class="sxs-lookup"><span data-stu-id="06513-820">Alternatively, you can use a different mechanism to drive the retries.</span></span>
* <span data-ttu-id="06513-821">El código de estado HTTP devuelto desde el servicio puede ayudar a indicar si el error es transitorio.</span><span class="sxs-lookup"><span data-stu-id="06513-821">The HTTP status code returned from the service can help to indicate whether the failure is transient.</span></span> <span data-ttu-id="06513-822">Puede que necesite examinar las excepciones generadas por un cliente o el marco de trabajo de reintento para obtener acceso al código de estado o para determinar el tipo de excepción equivalente.</span><span class="sxs-lookup"><span data-stu-id="06513-822">You may need to examine the exceptions generated by a client or the retry framework to access the status code or to determine the equivalent exception type.</span></span> <span data-ttu-id="06513-823">Los siguientes códigos HTTP normalmente indican que un reintento es adecuado:</span><span class="sxs-lookup"><span data-stu-id="06513-823">The following HTTP codes typically indicate that a retry is appropriate:</span></span>
  * <span data-ttu-id="06513-824">Tiempo de espera de solicitud 408</span><span class="sxs-lookup"><span data-stu-id="06513-824">408 Request Timeout</span></span>
  * <span data-ttu-id="06513-825">429 Demasiadas solicitudes</span><span class="sxs-lookup"><span data-stu-id="06513-825">429 Too Many Requests</span></span>
  * <span data-ttu-id="06513-826">Error de servidor interno 500</span><span class="sxs-lookup"><span data-stu-id="06513-826">500 Internal Server Error</span></span>
  * <span data-ttu-id="06513-827">Puerta de enlace incorrecta 502</span><span class="sxs-lookup"><span data-stu-id="06513-827">502 Bad Gateway</span></span>
  * <span data-ttu-id="06513-828">Servicio no disponible 503</span><span class="sxs-lookup"><span data-stu-id="06513-828">503 Service Unavailable</span></span>
  * <span data-ttu-id="06513-829">Tiempo de espera de puerta de enlace 504</span><span class="sxs-lookup"><span data-stu-id="06513-829">504 Gateway Timeout</span></span>
* <span data-ttu-id="06513-830">Si la lógica de reintento se basa en excepciones, las siguientes suelen indican un error transitorio donde no se pudo establecer ninguna conexión:</span><span class="sxs-lookup"><span data-stu-id="06513-830">If you base your retry logic on exceptions, the following typically indicate a transient failure where no connection could be established:</span></span>
  * <span data-ttu-id="06513-831">WebExceptionStatus.ConnectionClosed</span><span class="sxs-lookup"><span data-stu-id="06513-831">WebExceptionStatus.ConnectionClosed</span></span>
  * <span data-ttu-id="06513-832">WebExceptionStatus.ConnectFailure</span><span class="sxs-lookup"><span data-stu-id="06513-832">WebExceptionStatus.ConnectFailure</span></span>
  * <span data-ttu-id="06513-833">WebExceptionStatus.Timeout</span><span class="sxs-lookup"><span data-stu-id="06513-833">WebExceptionStatus.Timeout</span></span>
  * <span data-ttu-id="06513-834">WebExceptionStatus.RequestCanceled</span><span class="sxs-lookup"><span data-stu-id="06513-834">WebExceptionStatus.RequestCanceled</span></span>
* <span data-ttu-id="06513-835">En el caso de un estado no disponible del servicio, el servicio podría indicar el retraso adecuado antes de reintentar en el encabezado de respuesta **Retry-After** o un encabezado personalizado diferente.</span><span class="sxs-lookup"><span data-stu-id="06513-835">In the case of a service unavailable status, the service might indicate the appropriate delay before retrying in the **Retry-After** response header or a different custom header.</span></span> <span data-ttu-id="06513-836">Los servicios también pueden enviar información adicional como encabezados personalizados o incrustados en el contenido de la respuesta.</span><span class="sxs-lookup"><span data-stu-id="06513-836">Services might also send additional information as custom headers, or embedded in the content of the response.</span></span> <span data-ttu-id="06513-837">El bloque de aplicaciones de control de errores transitorios no puede usar los encabezados ni ningún encabezado "reintentar después de" personalizado.</span><span class="sxs-lookup"><span data-stu-id="06513-837">The Transient Fault Handling Application Block cannot use the standard or any custom “retry-after” headers.</span></span>
* <span data-ttu-id="06513-838">No intente de nuevo los códigos de estado que representan errores de cliente (errores en el intervalo 4xx) excepto un Tiempo de espera de solicitud 408.</span><span class="sxs-lookup"><span data-stu-id="06513-838">Do not retry for status codes representing client errors (errors in the 4xx range) except for a 408 Request Timeout.</span></span>
* <span data-ttu-id="06513-839">Pruebe las estrategias y mecanismos de reintento a fondo en una amplia variedad de condiciones, como diferentes estados de red diferente y diferentes cargas de sistema.</span><span class="sxs-lookup"><span data-stu-id="06513-839">Thoroughly test your retry strategies and mechanisms under a range of conditions, such as different network states and varying system loadings.</span></span>

### <a name="retry-strategies"></a><span data-ttu-id="06513-840">Estrategia de reintento</span><span class="sxs-lookup"><span data-stu-id="06513-840">Retry strategies</span></span>
<span data-ttu-id="06513-841">Estos son los tipos típicos de intervalos de estrategia de reintento:</span><span class="sxs-lookup"><span data-stu-id="06513-841">The following are the typical types of retry strategy intervals:</span></span>

* <span data-ttu-id="06513-842">**Exponencial**.</span><span class="sxs-lookup"><span data-stu-id="06513-842">**Exponential**.</span></span> <span data-ttu-id="06513-843">Una directiva de reintentos que realiza un número especificado de reintentos, mediante un enfoque de retroceso exponencial aleatorio para determinar el intervalo entre reintentos.</span><span class="sxs-lookup"><span data-stu-id="06513-843">A retry policy that performs a specified number of retries, using a randomized exponential back off approach to determine the interval between retries.</span></span> <span data-ttu-id="06513-844">Por ejemplo: </span><span class="sxs-lookup"><span data-stu-id="06513-844">For example:</span></span>

    ```csharp
    var random = new Random();

    var delta = (int)((Math.Pow(2.0, currentRetryCount) - 1.0) *
                random.Next((int)(this.deltaBackoff.TotalMilliseconds * 0.8),
                (int)(this.deltaBackoff.TotalMilliseconds * 1.2)));
    var interval = (int)Math.Min(checked(this.minBackoff.TotalMilliseconds + delta),
                    this.maxBackoff.TotalMilliseconds);
    retryInterval = TimeSpan.FromMilliseconds(interval);
    ```

* <span data-ttu-id="06513-845">**Incremental**.</span><span class="sxs-lookup"><span data-stu-id="06513-845">**Incremental**.</span></span> <span data-ttu-id="06513-846">Una estrategia de reintentos con un número especificado de reintentos y un intervalo de tiempo incremental entre los reintentos.</span><span class="sxs-lookup"><span data-stu-id="06513-846">A retry strategy with a specified number of retry attempts and an incremental time interval between retries.</span></span> <span data-ttu-id="06513-847">Por ejemplo: </span><span class="sxs-lookup"><span data-stu-id="06513-847">For example:</span></span>

    ```csharp
    retryInterval = TimeSpan.FromMilliseconds(this.initialInterval.TotalMilliseconds +
                    (this.increment.TotalMilliseconds * currentRetryCount));
    ```

* <span data-ttu-id="06513-848">**LinearRetry**.</span><span class="sxs-lookup"><span data-stu-id="06513-848">**LinearRetry**.</span></span> <span data-ttu-id="06513-849">Una directiva de reintentos que realiza un número especificado de reintentos, usando un intervalo de tiempo fijo especificado entre los reintentos.</span><span class="sxs-lookup"><span data-stu-id="06513-849">A retry policy that performs a specified number of retries, using a specified fixed time interval between retries.</span></span> <span data-ttu-id="06513-850">Por ejemplo: </span><span class="sxs-lookup"><span data-stu-id="06513-850">For example:</span></span>

    ```csharp
    retryInterval = this.deltaBackoff;
    ```

### <a name="transient-fault-handling-with-polly"></a><span data-ttu-id="06513-851">Consulte Control de errores transitorios con Polly.</span><span class="sxs-lookup"><span data-stu-id="06513-851">Transient fault handling with Polly</span></span>
<span data-ttu-id="06513-852">[Polly][polly] es una biblioteca para controlar mediante programación los reintentos y las estrategias [Circuit Breaker][circuit-breaker].</span><span class="sxs-lookup"><span data-stu-id="06513-852">[Polly][polly] is a library to programatically handle retries and [circuit breaker][circuit-breaker] strategies.</span></span> <span data-ttu-id="06513-853">El proyecto Polly es miembro de [.NET Foundation][dotnet-foundation].</span><span class="sxs-lookup"><span data-stu-id="06513-853">The Polly project is a member of the [.NET Foundation][dotnet-foundation].</span></span> <span data-ttu-id="06513-854">Para los servicios en los que el cliente no admite los reintentos de forma nativa, Polly es una alternativa válida y evita tener que escribir código de reintentos personalizado, que puede ser difícil de implementar correctamente.</span><span class="sxs-lookup"><span data-stu-id="06513-854">For services where the client does not natively support retries, Polly is a valid alternative and avoids the need to write custom retry code, which can be hard to implement correctly.</span></span> <span data-ttu-id="06513-855">Polly también permite el seguimiento de los errores cuando se produzcan, para poder registrar los reintentos.</span><span class="sxs-lookup"><span data-stu-id="06513-855">Polly also provides a way to trace errors when they occur, so that you can log retries.</span></span>


<!-- links -->

[adal]: /azure/active-directory/develop/active-directory-authentication-libraries
[autorest]: https://github.com/Azure/autorest/tree/master/docs
[circuit-breaker]: ../patterns/circuit-breaker.md
[ConnectionPolicy.RetryOptions]: https://msdn.microsoft.com/library/azure/microsoft.azure.documents.client.connectionpolicy.retryoptions.aspx
[dotnet-foundation]: https://dotnetfoundation.org/
[polly]: http://www.thepollyproject.org
[redis-cache-troubleshoot]: /azure/redis-cache/cache-how-to-troubleshoot
[SearchIndexClient]: https://msdn.microsoft.com/library/azure/microsoft.azure.search.searchindexclient.aspx
[SearchServiceClient]: https://msdn.microsoft.com/library/microsoft.azure.search.searchserviceclient.aspx


### <a name="more-information"></a><span data-ttu-id="06513-859">Más información</span><span class="sxs-lookup"><span data-stu-id="06513-859">More information</span></span>
* [<span data-ttu-id="06513-860">Resistencia de la conexión</span><span class="sxs-lookup"><span data-stu-id="06513-860">Connection Resiliency</span></span>](/ef/core/miscellaneous/connection-resiliency)
* [<span data-ttu-id="06513-861">Puntos de datos - EF Core 1.1</span><span class="sxs-lookup"><span data-stu-id="06513-861">Data Points - EF Core 1.1</span></span>](https://msdn.microsoft.com/magazine/mt745093.aspx)


