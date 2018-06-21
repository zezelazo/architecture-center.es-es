---
title: Selección de una solución para la integración de Active Directory local con Azure
description: Se comparan las arquitecturas de referencia para la integración de Active Directory local con Azure.
ms.date: 04/06/2017
ms.openlocfilehash: 413a5463d90547197c4b6834d353b4ecf61483ee
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2017
ms.locfileid: "24541663"
---
# <a name="choose-a-solution-for-integrating-on-premises-active-directory-with-azure"></a><span data-ttu-id="c5fa5-103">Selección de una solución para la integración de Active Directory local con Azure</span><span class="sxs-lookup"><span data-stu-id="c5fa5-103">Choose a solution for integrating on-premises Active Directory with Azure</span></span>

<span data-ttu-id="c5fa5-104">En este artículo se comparan opciones para integrar el entorno local de Active Directory (AD) con una red de Azure.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-104">This article compares options for integrating your on-premises Active Directory (AD) environment with an Azure network.</span></span> <span data-ttu-id="c5fa5-105">Se proporciona una arquitectura de referencia y una solución implementable para cada opción.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-105">We provide a reference architecture and a deployable solution for each option.</span></span>

<span data-ttu-id="c5fa5-106">Muchas organizaciones usan [Active Directory Domain Services (AD DS)][active-directory-domain-services] para autenticar las identidades asociadas con usuarios, equipos, aplicaciones y otros recursos que se incluyen en un límite de seguridad.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-106">Many organizations use [Active Directory Domain Services (AD DS)][active-directory-domain-services] to authenticate identities associated with users, computers, applications, or other resources that are included in a security boundary.</span></span> <span data-ttu-id="c5fa5-107">Los servicios de identidad y directorio se hospedan normalmente de forma local, pero si parte de su aplicación está hospedada en el entorno local y parte en Azure, puede que se produzca una latencia al enviar las solicitudes de autenticación desde Azure de vuelta al entorno local.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-107">Directory and identity services are typically hosted on-premises, but if your application is hosted partly on-premises and partly in Azure, there may be latency sending authentication requests from Azure back to on-premises.</span></span> <span data-ttu-id="c5fa5-108">La implementación de servicios de directorio e identidad en Azure puede reducir esta latencia.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-108">Implementing directory and identity services in Azure can reduce this latency.</span></span>

<span data-ttu-id="c5fa5-109">Azure proporciona dos soluciones para implementar servicios de directorio e identidad en Azure:</span><span class="sxs-lookup"><span data-stu-id="c5fa5-109">Azure provides two solutions for implementing directory and identity services in Azure:</span></span> 

* <span data-ttu-id="c5fa5-110">Use [Azure AD][azure-active-directory] para crear un dominio de Active Directory en la nube y conectarlo a su dominio local de Active Directory.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-110">Use [Azure AD][azure-active-directory] to create an Active Directory domain in the cloud and connect it to your on-premises Active Directory domain.</span></span> <span data-ttu-id="c5fa5-111">[Azure AD Connect][azure-ad-connect] integra sus directorios locales con Azure AD.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-111">[Azure AD Connect][azure-ad-connect] integrates your on-premises directories with Azure AD.</span></span>

* <span data-ttu-id="c5fa5-112">Amplíe la infraestructura existente de Active Directory local a Azure mediante la implementación de una máquina virtual en Azure que ejecute AD DS como un controlador de dominio.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-112">Extend your existing on-premises Active Directory infrastructure to Azure, by deploying a VM in Azure that runs AD DS as a domain controller.</span></span> <span data-ttu-id="c5fa5-113">Esta arquitectura es más frecuente cuando la red local y la red virtual de Azure (VNet) están conectadas mediante una conexión VPN o ExpressRoute.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-113">This architecture is more common when the on-premises network and the Azure virtual network (VNet) are connected by a VPN or ExpressRoute connection.</span></span> <span data-ttu-id="c5fa5-114">Son posibles diversas variantes de esta arquitectura:</span><span class="sxs-lookup"><span data-stu-id="c5fa5-114">Several variations of this architecture are possible:</span></span> 

    - <span data-ttu-id="c5fa5-115">Cree un dominio en Azure y únalo a su bosque de AD local.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-115">Create a domain in Azure and join it to your on-premises AD forest.</span></span>
    - <span data-ttu-id="c5fa5-116">Cree un bosque independiente en Azure en el que confíen los dominios del bosque local.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-116">Create a separate forest in Azure that is trusted by domains in your on-premises forest.</span></span>
    - <span data-ttu-id="c5fa5-117">Replique una implementación de Servicios de federación de Active Directory (AD FS) en Azure.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-117">Replicate an Active Directory Federation Services (AD FS) deployment to Azure.</span></span> 

<span data-ttu-id="c5fa5-118">En las secciones siguientes se describe cada una de estas opciones con más detalle.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-118">The next sections describe each of these options in more detail.</span></span>

## <a name="integrate-your-on-premises-domains-with-azure-ad"></a><span data-ttu-id="c5fa5-119">Integración de los dominios locales con Azure AD</span><span class="sxs-lookup"><span data-stu-id="c5fa5-119">Integrate your on-premises domains with Azure AD</span></span>

<span data-ttu-id="c5fa5-120">Use Azure Active Directory (Azure AD) para crear un dominio en Azure y vincularlo a un dominio de AD local.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-120">Use Azure Active Directory (Azure AD) to create a domain in Azure and link it to an on-premises AD domain.</span></span> 

<span data-ttu-id="c5fa5-121">El directorio de Azure AD no es una extensión de un directorio local.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-121">The Azure AD directory is not an extension of an on-premises directory.</span></span> <span data-ttu-id="c5fa5-122">Por el contrario, es una copia que contiene los mismos objetos e identidades.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-122">Rather, it's a copy that contains the same objects and identities.</span></span> <span data-ttu-id="c5fa5-123">Los cambios realizados en estos elementos locales se copian en Azure AD, pero los realizados en Azure AD no se replican al dominio local.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-123">Changes made to these items on-premises are copied to Azure AD, but changes made in Azure AD are not replicated back to the on-premises domain.</span></span>

<span data-ttu-id="c5fa5-124">También puede usar Azure AD sin utilizar un directorio local.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-124">You can also use Azure AD without using an on-premises directory.</span></span> <span data-ttu-id="c5fa5-125">En este caso, Azure AD actúa como el origen principal de toda la información de identidad, en lugar de contener los datos replicados desde un directorio local.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-125">In this case, Azure AD acts as the primary source of all identity information, rather than containing data replicated from an on-premises directory.</span></span>


<span data-ttu-id="c5fa5-126">**Ventajas**</span><span class="sxs-lookup"><span data-stu-id="c5fa5-126">**Benefits**</span></span>

* <span data-ttu-id="c5fa5-127">No es necesario mantener una infraestructura de AD en la nube.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-127">You don't need to maintain an AD infrastructure in the cloud.</span></span> <span data-ttu-id="c5fa5-128">Azure AD es un servicio que administra y mantiene completamente Microsoft.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-128">Azure AD is entirely managed and maintained by Microsoft.</span></span>
* <span data-ttu-id="c5fa5-129">Azure AD proporciona la misma información de identidad que está disponible en el entorno local.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-129">Azure AD provides the same identity information that is available on-premises.</span></span>
* <span data-ttu-id="c5fa5-130">La autenticación puede tener lugar en Azure, lo que reduce la necesidad de que las aplicaciones y los usuarios externos se pongan en contacto con el dominio local.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-130">Authentication can happen in Azure, reducing the need for external applications and users to contact the on-premises domain.</span></span>

<span data-ttu-id="c5fa5-131">**Desafíos**</span><span class="sxs-lookup"><span data-stu-id="c5fa5-131">**Challenges**</span></span>

* <span data-ttu-id="c5fa5-132">Los servicios de identidad se limitan a usuarios y grupos.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-132">Identity services are limited to users and groups.</span></span> <span data-ttu-id="c5fa5-133">No existe la posibilidad de autenticar cuentas de servicio y de equipo.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-133">There is no ability to authenticate service and computer accounts.</span></span>
* <span data-ttu-id="c5fa5-134">Debe configurar la conectividad con el dominio local para mantener sincronizado el directorio de Azure AD.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-134">You must configure connectivity with your on-premises domain to keep the Azure AD directory synchronized.</span></span> 
* <span data-ttu-id="c5fa5-135">Puede que sea necesario volver a escribir las aplicaciones para permitir la autenticación mediante Azure AD.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-135">Applications may need to be rewritten to enable authentication through Azure AD.</span></span>

<span data-ttu-id="c5fa5-136">**[Más información...][aad]**</span><span class="sxs-lookup"><span data-stu-id="c5fa5-136">**[Read more...][aad]**</span></span>

## <a name="ad-ds-in-azure-joined-to-an-on-premises-forest"></a><span data-ttu-id="c5fa5-137">AD DS en Azure unido a un bosque local</span><span class="sxs-lookup"><span data-stu-id="c5fa5-137">AD DS in Azure joined to an on-premises forest</span></span>

<span data-ttu-id="c5fa5-138">Implemente servidores de Servicios de dominio de Active Directory (AD DS) en Azure.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-138">Deploy AD Domain Services (AD DS) servers to Azure.</span></span> <span data-ttu-id="c5fa5-139">Cree un dominio en Azure y únalo a su bosque de AD local.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-139">Create a domain in Azure and join it to your on-premises AD forest.</span></span> 

<span data-ttu-id="c5fa5-140">Tenga en cuenta esta opción si tiene que usar características de AD DS que no están implementadas actualmente por Azure AD.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-140">Consider this option if you need to use AD DS features that are not currently implemented by Azure AD.</span></span> 

<span data-ttu-id="c5fa5-141">**Ventajas**</span><span class="sxs-lookup"><span data-stu-id="c5fa5-141">**Benefits**</span></span>

* <span data-ttu-id="c5fa5-142">Proporciona acceso a la misma información de identidad que está disponible en el entorno local.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-142">Provides access to the same identity information that is available on-premises.</span></span>
* <span data-ttu-id="c5fa5-143">Puede autenticar las cuentas de usuario, servicio y equipo de forma local y en Azure.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-143">You can authenticate user, service, and computer accounts on-premises and in Azure.</span></span>
* <span data-ttu-id="c5fa5-144">No es necesario administrar un bosque de AD independiente.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-144">You don't need to manage a separate AD forest.</span></span> <span data-ttu-id="c5fa5-145">El dominio de Azure puede pertenecer al bosque local.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-145">The domain in Azure can belong to the on-premises forest.</span></span>
* <span data-ttu-id="c5fa5-146">Puede aplicar la directiva de grupo definida por objetos de directiva de grupo local al dominio de Azure.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-146">You can apply group policy defined by on-premises Group Policy Objects to the domain in Azure.</span></span>

<span data-ttu-id="c5fa5-147">**Desafíos**</span><span class="sxs-lookup"><span data-stu-id="c5fa5-147">**Challenges**</span></span>

* <span data-ttu-id="c5fa5-148">Debe implementar y administrar sus propios servidores y dominios de AD DS en la nube.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-148">You must deploy and manage your own AD DS servers and domain in the cloud.</span></span>
* <span data-ttu-id="c5fa5-149">Puede haber cierta latencia de sincronización entre los servidores de dominio en la nube y los servidores que se ejecutan en el entorno local.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-149">There may be some synchronization latency between the domain servers in the cloud and the servers running on-premises.</span></span>

<span data-ttu-id="c5fa5-150">**[Más información...][ad-ds]**</span><span class="sxs-lookup"><span data-stu-id="c5fa5-150">**[Read more...][ad-ds]**</span></span>

## <a name="ad-ds-in-azure-with-a-separate-forest"></a><span data-ttu-id="c5fa5-151">AD DS en Azure con un bosque independiente</span><span class="sxs-lookup"><span data-stu-id="c5fa5-151">AD DS in Azure with a separate forest</span></span>

<span data-ttu-id="c5fa5-152">Implemente servidores de AD Domain Services (AD DS) en Azure, pero cree un bosque [independiente][ad-forest-defn] de Azure AD que esté separado del bosque local.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-152">Deploy AD Domain Services (AD DS) servers to Azure, but create a separate Active Directory [forest][ad-forest-defn] that is separate from the on-premises forest.</span></span> <span data-ttu-id="c5fa5-153">Todos los dominios del bosque local confían en este bosque.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-153">This forest is trusted by domains in your on-premises forest.</span></span>

<span data-ttu-id="c5fa5-154">Los usos habituales de esta arquitectura incluyen el mantenimiento de la separación de seguridad de objetos e identidades mantenida en la nube y la migración de dominios individuales del entorno local a la nube.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-154">Typical uses for this architecture include maintaining security separation for objects and identities held in the cloud, and migrating individual domains from on-premises to the cloud.</span></span>

<span data-ttu-id="c5fa5-155">**Ventajas**</span><span class="sxs-lookup"><span data-stu-id="c5fa5-155">**Benefits**</span></span>

* <span data-ttu-id="c5fa5-156">Puede implementar identidades locales y separar las identidades que son solo de Azure.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-156">You can implement on-premises identities and separate Azure-only identities.</span></span>
* <span data-ttu-id="c5fa5-157">No es necesario realizar la replicación del bosque local de AD a Azure.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-157">You don't need to replicate from the on-premises AD forest to Azure.</span></span>

<span data-ttu-id="c5fa5-158">**Desafíos**</span><span class="sxs-lookup"><span data-stu-id="c5fa5-158">**Challenges**</span></span>

* <span data-ttu-id="c5fa5-159">La autenticación en Azure de las identidades locales requiere saltos de red adicionales a los servidores de AD local.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-159">Authentication within Azure for on-premises identities requires extra network hops to the on-premises AD servers.</span></span>
* <span data-ttu-id="c5fa5-160">Debe implementar sus propios servidores y bosques de AD DS en la nube y establecer las relaciones de confianza adecuadas entre los bosques.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-160">You must deploy your own AD DS servers and forest in the cloud, and establish the appropriate trust relationships between forests.</span></span>

<span data-ttu-id="c5fa5-161">**[Más información...][ad-ds-forest]**</span><span class="sxs-lookup"><span data-stu-id="c5fa5-161">**[Read more...][ad-ds-forest]**</span></span>

## <a name="extend-ad-fs-to-azure"></a><span data-ttu-id="c5fa5-162">Extensión de AD FS a Azure</span><span class="sxs-lookup"><span data-stu-id="c5fa5-162">Extend AD FS to Azure</span></span>

<span data-ttu-id="c5fa5-163">Replique una implementación de Servicios de federación de Active Directory (AD FS) a Azure, para realizar la autenticación y la autorización federadas de los componentes que se ejecutan en Azure.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-163">Replicate an Active Directory Federation Services (AD FS) deployment to Azure, to perform federated authentication and authorization for components running in Azure.</span></span> 

<span data-ttu-id="c5fa5-164">Usos habituales de esta arquitectura:</span><span class="sxs-lookup"><span data-stu-id="c5fa5-164">Typical uses for this architecture:</span></span>

* <span data-ttu-id="c5fa5-165">Autenticar y autorizar a los usuarios de organizaciones asociadas.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-165">Authenticate and authorize users from partner organizations.</span></span>
* <span data-ttu-id="c5fa5-166">Permitir a los usuarios autenticarse en exploradores web que se ejecutan fuera del firewall de la organización.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-166">Allow users to authenticate from web browsers running outside of the organizational firewall.</span></span>
* <span data-ttu-id="c5fa5-167">Permitir a los usuarios conectarse desde dispositivos externos autorizados, como dispositivos móviles.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-167">Allow users to connect from authorized external devices such as mobile devices.</span></span> 

<span data-ttu-id="c5fa5-168">**Ventajas**</span><span class="sxs-lookup"><span data-stu-id="c5fa5-168">**Benefits**</span></span>

* <span data-ttu-id="c5fa5-169">Puede aprovechar las aplicaciones basadas en notificaciones.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-169">You can leverage claims-aware applications.</span></span>
* <span data-ttu-id="c5fa5-170">Ofrece la posibilidad de confiar la autenticación a asociados externos.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-170">Provides the ability to trust external partners for authentication.</span></span>
* <span data-ttu-id="c5fa5-171">Compatibilidad con un gran conjunto de protocolos de autenticación.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-171">Compatibility with large set of authentication protocols.</span></span>

<span data-ttu-id="c5fa5-172">**Desafíos**</span><span class="sxs-lookup"><span data-stu-id="c5fa5-172">**Challenges**</span></span>

* <span data-ttu-id="c5fa5-173">Debe implementar sus propios servidores de AD DS, AD FS y de proxy de aplicación web de AD FS en Azure.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-173">You must deploy your own AD DS, AD FS, and AD FS Web Application Proxy servers in Azure.</span></span>
* <span data-ttu-id="c5fa5-174">Esta arquitectura puede ser difícil de configurar.</span><span class="sxs-lookup"><span data-stu-id="c5fa5-174">This architecture can be complex to configure.</span></span>

<span data-ttu-id="c5fa5-175">**[Más información...][adfs]**</span><span class="sxs-lookup"><span data-stu-id="c5fa5-175">**[Read more...][adfs]**</span></span>

<!-- links -->

[aad]: ./azure-ad.md
[ad-ds]: ./adds-extend-domain.md
[ad-ds-forest]: ./adds-forest.md
[ad-forest-defn]: https://msdn.microsoft.com/library/ms676906.aspx
[adfs]: ./adfs.md

[active-directory-domain-services]: https://technet.microsoft.com/library/dd448614.aspx
[azure-active-directory]: /azure/active-directory-domain-services/active-directory-ds-overview
[azure-ad-connect]: /azure/active-directory/active-directory-aadconnect
