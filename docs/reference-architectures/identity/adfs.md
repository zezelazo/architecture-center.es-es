---
title: Extensión de AD FS local a Azure
titleSuffix: Azure Reference Architectures
description: Implemente una arquitectura de red híbrida segura con la autorización del Servicio de federación de Active Directory en Azure.
author: telmosampaio
ms.date: 12/18.2018
ms.custom: seodec18
ms.openlocfilehash: bd07ce1502c29c1543dca42f74b2f19f3a6d3878
ms.sourcegitcommit: bb7fcffbb41e2c26a26f8781df32825eb60df70c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/20/2018
ms.locfileid: "53644111"
---
# <a name="extend-active-directory-federation-services-ad-fs-to-azure"></a><span data-ttu-id="36a9d-103">Extensión de Servicios de federación de Active Directory (AD FS) a Azure</span><span class="sxs-lookup"><span data-stu-id="36a9d-103">Extend Active Directory Federation Services (AD FS) to Azure</span></span>

<span data-ttu-id="36a9d-104">Esta arquitectura de referencia implementa una red segura híbrida que extiende la red local a Azure y usa los [Servicios de federación de Active Directory (AD FS)][active-directory-federation-services] para realizar la autenticación y la autorización federada para los componentes que se ejecutan en Azure.</span><span class="sxs-lookup"><span data-stu-id="36a9d-104">This reference architecture implements a secure hybrid network that extends your on-premises network to Azure and uses [Active Directory Federation Services (AD FS)][active-directory-federation-services] to perform federated authentication and authorization for components running in Azure.</span></span> <span data-ttu-id="36a9d-105">[**Implemente esta solución**](#deploy-the-solution).</span><span class="sxs-lookup"><span data-stu-id="36a9d-105">[**Deploy this solution**](#deploy-the-solution).</span></span>

![Arquitectura de red híbrida segura con Active Directory](./images/adfs.png)

<span data-ttu-id="36a9d-107">*Descargue un [archivo Visio][visio-download] de esta arquitectura.*</span><span class="sxs-lookup"><span data-stu-id="36a9d-107">*Download a [Visio file][visio-download] of this architecture.*</span></span>

<span data-ttu-id="36a9d-108">AD FS puede hospedarse de forma local, pero, si la aplicación es un híbrido en el que algunas partes se implementan en Azure, puede ser más eficaz replicar AD FS en la nube.</span><span class="sxs-lookup"><span data-stu-id="36a9d-108">AD FS can be hosted on-premises, but if your application is a hybrid in which some parts are implemented in Azure, it may be more efficient to replicate AD FS in the cloud.</span></span>

<span data-ttu-id="36a9d-109">El diagrama muestra los siguientes escenarios:</span><span class="sxs-lookup"><span data-stu-id="36a9d-109">The diagram shows the following scenarios:</span></span>

- <span data-ttu-id="36a9d-110">El código de la aplicación de una organización asociada tiene acceso a una aplicación web que se hospeda dentro de la red virtual de Azure.</span><span class="sxs-lookup"><span data-stu-id="36a9d-110">Application code from a partner organization accesses a web application hosted inside your Azure VNet.</span></span>
- <span data-ttu-id="36a9d-111">Un usuario externo y registrado con las credenciales almacenadas en Active Directory Domain Services (DS) tiene acceso a una aplicación web que se hospeda dentro de la red virtual de Azure.</span><span class="sxs-lookup"><span data-stu-id="36a9d-111">An external, registered user with credentials stored inside Active Directory Domain Services (DS) accesses a web application hosted inside your Azure VNet.</span></span>
- <span data-ttu-id="36a9d-112">Un usuario conectado a la red virtual con un dispositivo autorizado ejecuta una aplicación web que se hospeda dentro de la red virtual de Azure.</span><span class="sxs-lookup"><span data-stu-id="36a9d-112">A user connected to your VNet using an authorized device executes a web application hosted inside your Azure VNet.</span></span>

<span data-ttu-id="36a9d-113">Los usos habituales de esta arquitectura incluyen:</span><span class="sxs-lookup"><span data-stu-id="36a9d-113">Typical uses for this architecture include:</span></span>

- <span data-ttu-id="36a9d-114">Aplicaciones híbridas donde una parte de las cargas de trabajo se ejecutan de forma local y otra parte en Azure.</span><span class="sxs-lookup"><span data-stu-id="36a9d-114">Hybrid applications where workloads run partly on-premises and partly in Azure.</span></span>
- <span data-ttu-id="36a9d-115">Soluciones que usan autorización federada para exponer las aplicaciones web a las organizaciones asociadas.</span><span class="sxs-lookup"><span data-stu-id="36a9d-115">Solutions that use federated authorization to expose web applications to partner organizations.</span></span>
- <span data-ttu-id="36a9d-116">Sistemas que permiten el acceso desde exploradores web que se ejecutan fuera del firewall de la organización.</span><span class="sxs-lookup"><span data-stu-id="36a9d-116">Systems that support access from web browsers running outside of the organizational firewall.</span></span>
- <span data-ttu-id="36a9d-117">Sistemas que permiten a los usuarios el acceso a las aplicaciones web mediante la conexión de dispositivos externos autorizados como equipos remotos, portátiles y otros dispositivos móviles.</span><span class="sxs-lookup"><span data-stu-id="36a9d-117">Systems that enable users to access to web applications by connecting from authorized external devices such as remote computers, notebooks, and other mobile devices.</span></span>

<span data-ttu-id="36a9d-118">Esta arquitectura de referencia se centra en la *federación pasiva*, en la que los servidores de federación deciden cómo y cuándo autenticar a un usuario.</span><span class="sxs-lookup"><span data-stu-id="36a9d-118">This reference architecture focuses on *passive federation*, in which the federation servers decide how and when to authenticate a user.</span></span> <span data-ttu-id="36a9d-119">El usuario proporciona información de inicio de sesión cuando se inicia la aplicación.</span><span class="sxs-lookup"><span data-stu-id="36a9d-119">The user provides sign in information when the application is started.</span></span> <span data-ttu-id="36a9d-120">Este mecanismo se usa normalmente en los exploradores web e implica un protocolo que redirige el explorador a un sitio donde el usuario se autentica.</span><span class="sxs-lookup"><span data-stu-id="36a9d-120">This mechanism is most commonly used by web browsers and involves a protocol that redirects the browser to a site where the user authenticates.</span></span> <span data-ttu-id="36a9d-121">AD FS también admite la *federación activa*, en la que una aplicación asume la responsabilidad de proporcionar las credenciales sin la intervención del usuario, pero ese escenario está fuera del ámbito de esta arquitectura.</span><span class="sxs-lookup"><span data-stu-id="36a9d-121">AD FS also supports *active federation*, where an application takes on responsibility for supplying credentials without further user interaction, but that scenario is outside the scope of this architecture.</span></span>

<span data-ttu-id="36a9d-122">Para consideraciones adicionales, consulte [Selección de una solución para la integración de Active Directory local con Azure][considerations].</span><span class="sxs-lookup"><span data-stu-id="36a9d-122">For additional considerations, see [Choose a solution for integrating on-premises Active Directory with Azure][considerations].</span></span>

## <a name="architecture"></a><span data-ttu-id="36a9d-123">Arquitectura</span><span class="sxs-lookup"><span data-stu-id="36a9d-123">Architecture</span></span>

<span data-ttu-id="36a9d-124">Esta arquitectura amplía la implementación que se describe en [Extensión de AD DS a Azure][extending-ad-to-azure].</span><span class="sxs-lookup"><span data-stu-id="36a9d-124">This architecture extends the implementation described in [Extending AD DS to Azure][extending-ad-to-azure].</span></span> <span data-ttu-id="36a9d-125">Contiene los componentes siguientes:</span><span class="sxs-lookup"><span data-stu-id="36a9d-125">It contains the followign components.</span></span>

- <span data-ttu-id="36a9d-126">**Subred de AD DS**.</span><span class="sxs-lookup"><span data-stu-id="36a9d-126">**AD DS subnet**.</span></span> <span data-ttu-id="36a9d-127">Los servidores de AD DS se encuentran en su propia subred con reglas de grupo de seguridad de red (NSG) que actúan como un firewall.</span><span class="sxs-lookup"><span data-stu-id="36a9d-127">The AD DS servers are contained in their own subnet with network security group (NSG) rules acting as a firewall.</span></span>

- <span data-ttu-id="36a9d-128">**Servidores de AD FS**.</span><span class="sxs-lookup"><span data-stu-id="36a9d-128">**AD DS servers**.</span></span> <span data-ttu-id="36a9d-129">Controladores de dominio que se ejecutan como máquinas virtuales en Azure.</span><span class="sxs-lookup"><span data-stu-id="36a9d-129">Domain controllers running as VMs in Azure.</span></span> <span data-ttu-id="36a9d-130">Estos servidores proporcionan autenticación de las identidades locales dentro del dominio.</span><span class="sxs-lookup"><span data-stu-id="36a9d-130">These servers provide authentication of local identities within the domain.</span></span>

- <span data-ttu-id="36a9d-131">**Subred de AD FS**.</span><span class="sxs-lookup"><span data-stu-id="36a9d-131">**AD FS subnet**.</span></span> <span data-ttu-id="36a9d-132">Los servidores de AD FS se encuentran dentro de su propia subred con reglas NSG que actúan como un firewall.</span><span class="sxs-lookup"><span data-stu-id="36a9d-132">The AD FS servers are located within their own subnet with NSG rules acting as a firewall.</span></span>

- <span data-ttu-id="36a9d-133">**Servidores de AD FS**.</span><span class="sxs-lookup"><span data-stu-id="36a9d-133">**AD FS servers**.</span></span> <span data-ttu-id="36a9d-134">Los servidores de AD FS proporcionan autenticación y autorización federadas.</span><span class="sxs-lookup"><span data-stu-id="36a9d-134">The AD FS servers provide federated authorization and authentication.</span></span> <span data-ttu-id="36a9d-135">En esta arquitectura, se realizan las siguientes tareas:</span><span class="sxs-lookup"><span data-stu-id="36a9d-135">In this architecture, they perform the following tasks:</span></span>

  - <span data-ttu-id="36a9d-136">Recibir tokens de seguridad que contienen notificaciones realizadas por un servidor de federación asociado en nombre del usuario asociado.</span><span class="sxs-lookup"><span data-stu-id="36a9d-136">Receiving security tokens containing claims made by a partner federation server on behalf of a partner user.</span></span> <span data-ttu-id="36a9d-137">AD FS comprueba que los tokens son válidos antes de pasar las notificaciones a la aplicación web que se ejecuta en Azure para autorizar las solicitudes.</span><span class="sxs-lookup"><span data-stu-id="36a9d-137">AD FS verifies that the tokens are valid before passing the claims to the web application running in Azure to authorize requests.</span></span>

    <span data-ttu-id="36a9d-138">La aplicación que se ejecuta en Azure es el *usuario de confianza*.</span><span class="sxs-lookup"><span data-stu-id="36a9d-138">The application running in Azure is the *relying party*.</span></span> <span data-ttu-id="36a9d-139">El servidor de federación asociado debe emitir notificaciones que la aplicación web entienda.</span><span class="sxs-lookup"><span data-stu-id="36a9d-139">The partner federation server must issue claims that are understood by the web application.</span></span> <span data-ttu-id="36a9d-140">Los servidores de federación asociados se conocen como *asociados de cuenta*, ya que envían las solicitudes de acceso en nombre de las cuentas autenticadas en la organización del asociado.</span><span class="sxs-lookup"><span data-stu-id="36a9d-140">The partner federation servers are referred to as *account partners*, because they submit access requests on behalf of authenticated accounts in the partner organization.</span></span> <span data-ttu-id="36a9d-141">Los servidores de AD FS se denominan *asociados de recursos* ya que proporcionan acceso a los recursos (la aplicación web).</span><span class="sxs-lookup"><span data-stu-id="36a9d-141">The AD FS servers are called *resource partners* because they provide access to resources (the web application).</span></span>

  - <span data-ttu-id="36a9d-142">Autenticar y autorizar las solicitudes entrantes de los usuarios externos que ejecutan un explorador web o un dispositivo que necesita tener acceso a las aplicaciones web, mediante AD DS y el [Servicio de registro de dispositivos de Active Directory][ADDRS].</span><span class="sxs-lookup"><span data-stu-id="36a9d-142">Authenticating and authorizing incoming requests from external users running a web browser or device that needs access to web applications, by using AD DS and the [Active Directory Device Registration Service][ADDRS].</span></span>

  <span data-ttu-id="36a9d-143">Los servidores de AD FS se configuran como una granja de servidores a los que se accede a través de un equilibrador de carga de Azure.</span><span class="sxs-lookup"><span data-stu-id="36a9d-143">The AD FS servers are configured as a farm accessed through an Azure load balancer.</span></span> <span data-ttu-id="36a9d-144">Esta implementación mejora la disponibilidad y la escalabilidad.</span><span class="sxs-lookup"><span data-stu-id="36a9d-144">This implementation improves availability and scalability.</span></span> <span data-ttu-id="36a9d-145">Los servidores de AD FS no se exponen directamente a Internet.</span><span class="sxs-lookup"><span data-stu-id="36a9d-145">The AD FS servers are not exposed directly to the Internet.</span></span> <span data-ttu-id="36a9d-146">Todo el tráfico de Internet se filtra a través de servidores proxy de aplicación web de AD FS y una red perimetral (también conocida como DMZ).</span><span class="sxs-lookup"><span data-stu-id="36a9d-146">All Internet traffic is filtered through AD FS web application proxy servers and a DMZ (also referred to as a perimeter network).</span></span>

  <span data-ttu-id="36a9d-147">Para obtener más información acerca del funcionamiento de AD FS, consulte [Introducción a los Servicios de federación de Active Directory][active-directory-federation-services-overview].</span><span class="sxs-lookup"><span data-stu-id="36a9d-147">For more information about how AD FS works, see [Active Directory Federation Services Overview][active-directory-federation-services-overview].</span></span> <span data-ttu-id="36a9d-148">Además, el artículo [Implementación de AD FS en Azure][adfs-intro] contiene una introducción detallada paso a paso para la implementación.</span><span class="sxs-lookup"><span data-stu-id="36a9d-148">Also, the article [AD FS deployment in Azure][adfs-intro] contains a detailed step-by-step introduction to implementation.</span></span>

- <span data-ttu-id="36a9d-149">**Subred de proxy de AD FS**.</span><span class="sxs-lookup"><span data-stu-id="36a9d-149">**AD FS proxy subnet**.</span></span> <span data-ttu-id="36a9d-150">Los servidores proxy de AD FS pueden estar dentro de su propia subred, con reglas NSG que proporcionan la protección.</span><span class="sxs-lookup"><span data-stu-id="36a9d-150">The AD FS proxy servers can be contained within their own subnet, with NSG rules providing protection.</span></span> <span data-ttu-id="36a9d-151">Los servidores de esta subred se exponen a Internet a través de un conjunto de aplicaciones virtuales de red que proporcionan un firewall entre la red virtual de Azure e Internet.</span><span class="sxs-lookup"><span data-stu-id="36a9d-151">The servers in this subnet are exposed to the Internet through a set of network virtual appliances that provide a firewall between your Azure virtual network and the Internet.</span></span>

- <span data-ttu-id="36a9d-152">**Servidores proxy de aplicación web (WAP) de AD FS**.</span><span class="sxs-lookup"><span data-stu-id="36a9d-152">**AD FS web application proxy (WAP) servers**.</span></span> <span data-ttu-id="36a9d-153">Estas máquinas virtuales actúan como servidores de AD FS para las solicitudes entrantes de las organizaciones asociadas y los dispositivos externos.</span><span class="sxs-lookup"><span data-stu-id="36a9d-153">These VMs act as AD FS servers for incoming requests from partner organizations and external devices.</span></span> <span data-ttu-id="36a9d-154">Los servidores WAP actúan como un filtro, blindando a los servidores de AD FS frente al acceso directo desde Internet.</span><span class="sxs-lookup"><span data-stu-id="36a9d-154">The WAP servers act as a filter, shielding the AD FS servers from direct access from the Internet.</span></span> <span data-ttu-id="36a9d-155">Al igual que con los servidores de AD FS, implementar los servidores WAP en una granja de servidores con equilibrio de carga ofrece mayor disponibilidad y escalabilidad que la implementación de una colección de servidores independientes.</span><span class="sxs-lookup"><span data-stu-id="36a9d-155">As with the AD FS servers, deploying the WAP servers in a farm with load balancing gives you greater availability and scalability than deploying a collection of stand-alone servers.</span></span>

  > [!NOTE]
  > <span data-ttu-id="36a9d-156">Para información detallada acerca de cómo instalar los servidores WAP, consulte [Instalar y configurar el servidor de Proxy de aplicación web][install_and_configure_the_web_application_proxy_server]</span><span class="sxs-lookup"><span data-stu-id="36a9d-156">For detailed information about installing WAP servers, see [Install and Configure the Web Application Proxy Server][install_and_configure_the_web_application_proxy_server]</span></span>
  >

- <span data-ttu-id="36a9d-157">**Organización del asociado**.</span><span class="sxs-lookup"><span data-stu-id="36a9d-157">**Partner organization**.</span></span> <span data-ttu-id="36a9d-158">La organización de un asociado ejecuta una aplicación web que solicita acceso a una aplicación web que se ejecuta en Azure.</span><span class="sxs-lookup"><span data-stu-id="36a9d-158">A partner organization running a web application that requests access to a web application running in Azure.</span></span> <span data-ttu-id="36a9d-159">El servidor de federación de la organización del asociado autentica localmente las solicitudes y envía tokens de seguridad que contienen notificaciones a AD FS que se ejecuta en Azure.</span><span class="sxs-lookup"><span data-stu-id="36a9d-159">The federation server at the partner organization authenticates requests locally, and submits security tokens containing claims to AD FS running in Azure.</span></span> <span data-ttu-id="36a9d-160">AD FS en Azure valida los tokens de seguridad y, si son válidos, puede pasar las notificaciones a la aplicación web que se ejecuta en Azure para que las autorice.</span><span class="sxs-lookup"><span data-stu-id="36a9d-160">AD FS in Azure validates the security tokens, and if valid can pass the claims to the web application running in Azure to authorize them.</span></span>

  > [!NOTE]
  > <span data-ttu-id="36a9d-161">También puede configurar un túnel VPN con la puerta de enlace de Azure para proporcionar acceso directo a AD FS para los asociados de confianza.</span><span class="sxs-lookup"><span data-stu-id="36a9d-161">You can also configure a VPN tunnel using Azure gateway to provide direct access to AD FS for trusted partners.</span></span> <span data-ttu-id="36a9d-162">Las solicitudes recibidas desde estos asociados no pasan a través de los servidores WAP.</span><span class="sxs-lookup"><span data-stu-id="36a9d-162">Requests received from these partners do not pass through the WAP servers.</span></span>
  >

## <a name="recommendations"></a><span data-ttu-id="36a9d-163">Recomendaciones</span><span class="sxs-lookup"><span data-stu-id="36a9d-163">Recommendations</span></span>

<span data-ttu-id="36a9d-164">Las siguientes recomendaciones sirven para la mayoría de los escenarios.</span><span class="sxs-lookup"><span data-stu-id="36a9d-164">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="36a9d-165">Sígalas a menos que tenga un requisito concreto que las invalide.</span><span class="sxs-lookup"><span data-stu-id="36a9d-165">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="networking-recommendations"></a><span data-ttu-id="36a9d-166">Recomendaciones de redes</span><span class="sxs-lookup"><span data-stu-id="36a9d-166">Networking recommendations</span></span>

<span data-ttu-id="36a9d-167">Configure la interfaz de red para cada una de las máquinas virtuales que hospedan servidores de AD FS y WAP con direcciones IP privadas estáticas.</span><span class="sxs-lookup"><span data-stu-id="36a9d-167">Configure the network interface for each of the VMs hosting AD FS and WAP servers with static private IP addresses.</span></span>

<span data-ttu-id="36a9d-168">No conceda a las máquinas virtuales de AD FS direcciones IP públicas.</span><span class="sxs-lookup"><span data-stu-id="36a9d-168">Do not give the AD FS VMs public IP addresses.</span></span> <span data-ttu-id="36a9d-169">Para más información, consulte la sección [Consideraciones de seguridad](#security-considerations).</span><span class="sxs-lookup"><span data-stu-id="36a9d-169">For more information, see the [Security considerations](#security-considerations) section.</span></span>

<span data-ttu-id="36a9d-170">Establezca la dirección IP de los servidores del servicio de nombres de dominio preferido y secundario (DNS) para las interfaces de red de cada máquina virtual AD FS y WAP para hacer referencia a las máquinas virtuales de Active Directory DS.</span><span class="sxs-lookup"><span data-stu-id="36a9d-170">Set the IP address of the preferred and secondary domain name service (DNS) servers for the network interfaces for each AD FS and WAP VM to reference the Active Directory DS VMs.</span></span> <span data-ttu-id="36a9d-171">Las máquinas virtuales de Active Directory DS deben ejecutar DNS.</span><span class="sxs-lookup"><span data-stu-id="36a9d-171">The Active Directory DS VMs should be running DNS.</span></span> <span data-ttu-id="36a9d-172">Este paso es necesario para permitir que cada máquina virtual se una al dominio.</span><span class="sxs-lookup"><span data-stu-id="36a9d-172">This step is necessary to enable each VM to join the domain.</span></span>

### <a name="ad-fs-installation"></a><span data-ttu-id="36a9d-173">Instalación de AD FS</span><span class="sxs-lookup"><span data-stu-id="36a9d-173">AD FS installation</span></span>

<span data-ttu-id="36a9d-174">El artículo [Implementación de una granja de servidores de federación] [ Deploying_a_federation_server_farm] proporciona instrucciones detalladas para instalar y configurar AD FS.</span><span class="sxs-lookup"><span data-stu-id="36a9d-174">The article [Deploying a Federation Server Farm][Deploying_a_federation_server_farm] provides detailed instructions for installing and configuring AD FS.</span></span> <span data-ttu-id="36a9d-175">Antes de configurar el primer servidor de AD FS en la granja de servidores, haga lo siguiente:</span><span class="sxs-lookup"><span data-stu-id="36a9d-175">Perform the following tasks before configuring the first AD FS server in the farm:</span></span>

1. <span data-ttu-id="36a9d-176">Obtenga un certificado público de confianza para realizar la autenticación de los servidores.</span><span class="sxs-lookup"><span data-stu-id="36a9d-176">Obtain a publicly trusted certificate for performing server authentication.</span></span> <span data-ttu-id="36a9d-177">El *nombre de sujeto* debe contener el nombre que los clientes usan para tener acceso al servicio de federación.</span><span class="sxs-lookup"><span data-stu-id="36a9d-177">The *subject name* must contain the name clients use to access the federation service.</span></span> <span data-ttu-id="36a9d-178">Puede tratarse del nombre DNS registrado para el equilibrador de carga, por ejemplo *adfs.contoso.com* (por motivos de seguridad, evite utilizar nombres con caracteres comodín como \**.contoso.com*).</span><span class="sxs-lookup"><span data-stu-id="36a9d-178">This can be the DNS name registered for the load balancer, for example, *adfs.contoso.com* (avoid using wildcard names such as \**.contoso.com*, for security reasons).</span></span> <span data-ttu-id="36a9d-179">Use el mismo certificado en todas las máquinas virtuales de servidores de AD FS.</span><span class="sxs-lookup"><span data-stu-id="36a9d-179">Use the same certificate on all AD FS server VMs.</span></span> <span data-ttu-id="36a9d-180">Puede adquirir un certificado de una entidad de certificación de confianza, pero, si su organización usa Servicios de certificados de Active Directory, puede crear los suyos propios.</span><span class="sxs-lookup"><span data-stu-id="36a9d-180">You can purchase a certificate from a trusted certification authority, but if your organization uses Active Directory Certificate Services you can create your own.</span></span>

    <span data-ttu-id="36a9d-181">El *nombre alternativo del firmante* se utiliza en el servicio de registro de dispositivos (DRS) para habilitar el acceso desde dispositivos externos.</span><span class="sxs-lookup"><span data-stu-id="36a9d-181">The *subject alternative name* is used by the device registration service (DRS) to enable access from external devices.</span></span> <span data-ttu-id="36a9d-182">Debe tener el formato *enterpriseregistration.contoso.com*.</span><span class="sxs-lookup"><span data-stu-id="36a9d-182">This should be of the form *enterpriseregistration.contoso.com*.</span></span>

    <span data-ttu-id="36a9d-183">Para más información, consulte [Obtain and Configure a Secure Sockets Layer (SSL) Certificate for AD FS][adfs_certificates] [Obtención y configuración de un certificado Capa de sockets seguros (SSL) para AD FS].</span><span class="sxs-lookup"><span data-stu-id="36a9d-183">For more information, see [Obtain and Configure a Secure Sockets Layer (SSL) Certificate for AD FS][adfs_certificates].</span></span>

2. <span data-ttu-id="36a9d-184">En el controlador de dominio, genere una nueva clave raíz para el Servicio de distribución de claves.</span><span class="sxs-lookup"><span data-stu-id="36a9d-184">On the domain controller, generate a new root key for the Key Distribution Service.</span></span> <span data-ttu-id="36a9d-185">Establezca el tiempo efectivo en la hora actual menos 10 horas (esta configuración reduce el retardo que se puede producir en la distribución y la sincronización de las claves a través del dominio).</span><span class="sxs-lookup"><span data-stu-id="36a9d-185">Set the effective time to the current time minus 10 hours (this configuration reduces the delay that can occur in distributing and synchronizing keys across the domain).</span></span> <span data-ttu-id="36a9d-186">Este paso es necesario para permitir la creación de la cuenta del servicio de grupo que se usa para ejecutar el servicio AD FS.</span><span class="sxs-lookup"><span data-stu-id="36a9d-186">This step is necessary to support creating the group service account that is used to run the AD FS service.</span></span> <span data-ttu-id="36a9d-187">El siguiente comando de PowerShell muestra un ejemplo de cómo hacerlo:</span><span class="sxs-lookup"><span data-stu-id="36a9d-187">The following PowerShell command shows an example of how to do this:</span></span>

    ```powershell
    Add-KdsRootKey -EffectiveTime (Get-Date).AddHours(-10)
    ```

3. <span data-ttu-id="36a9d-188">Agregue cada máquina virtual del servidor de AD FS al dominio.</span><span class="sxs-lookup"><span data-stu-id="36a9d-188">Add each AD FS server VM to the domain.</span></span>

> [!NOTE]
> <span data-ttu-id="36a9d-189">Para instalar AD FS, el controlador de dominio que ejecuta el rol de Operaciones de maestro único flexible (FSMO) del emulador del controlador de dominio principal (PDC) para el dominio debe estar ejecutándose y ser accesible desde las máquinas virtuales de AD FS.</span><span class="sxs-lookup"><span data-stu-id="36a9d-189">To install AD FS, the domain controller running the primary domain controller (PDC) emulator flexible single master operation (FSMO) role for the domain must be running and accessible from the AD FS VMs.</span></span> <span data-ttu-id="36a9d-190"><<RBC: ¿hay alguna manera de hacer esto menos repetitivo?>></span><span class="sxs-lookup"><span data-stu-id="36a9d-190"><<RBC: Is there a way to make this less repetitive?>></span></span>
>

### <a name="ad-fs-trust"></a><span data-ttu-id="36a9d-191">Confianza de AD FS</span><span class="sxs-lookup"><span data-stu-id="36a9d-191">AD FS trust</span></span>

<span data-ttu-id="36a9d-192">Establezca la confianza de federación entre la instalación de AD FS y los servidores de federación de las organizaciones asociadas.</span><span class="sxs-lookup"><span data-stu-id="36a9d-192">Establish federation trust between your AD FS installation, and the federation servers of any partner organizations.</span></span> <span data-ttu-id="36a9d-193">Configure las notificaciones de filtrado y asignación necesarias.</span><span class="sxs-lookup"><span data-stu-id="36a9d-193">Configure any claims filtering and mapping required.</span></span>

- <span data-ttu-id="36a9d-194">El personal de DevOps de cada organización asociada debe agregar una relación de confianza para las aplicaciones web accesibles a través de los servidores de AD FS.</span><span class="sxs-lookup"><span data-stu-id="36a9d-194">DevOps staff at each partner organization must add a relying party trust for the web applications accessible through your AD FS servers.</span></span>
- <span data-ttu-id="36a9d-195">El personal de DevOps de la organización debe configurar la confianza del proveedor de notificaciones a fin de habilitar los servidores de AD FS para confiar en las notificaciones que las organizaciones asociadas proporcionan.</span><span class="sxs-lookup"><span data-stu-id="36a9d-195">DevOps staff in your organization must configure claims-provider trust to enable your AD FS servers to trust the claims that partner organizations provide.</span></span>
- <span data-ttu-id="36a9d-196">También debe configurar AD FS para pasar las notificaciones a las aplicaciones web de la organización.</span><span class="sxs-lookup"><span data-stu-id="36a9d-196">DevOps staff in your organization must also configure AD FS to pass claims on to your organization's web applications.</span></span>

<span data-ttu-id="36a9d-197">Para más información, consulte [Establishing Federation Trust][establishing-federation-trust] (Establecimiento de la confianza de la federación).</span><span class="sxs-lookup"><span data-stu-id="36a9d-197">For more information, see [Establishing Federation Trust][establishing-federation-trust].</span></span>

<span data-ttu-id="36a9d-198">Publique las aplicaciones web de su organización y haga que estén disponibles para los asociados externos con la autenticación previa a través de servidores WAP.</span><span class="sxs-lookup"><span data-stu-id="36a9d-198">Publish your organization's web applications and make them available to external partners by using preauthentication through the WAP servers.</span></span> <span data-ttu-id="36a9d-199">Para más información, consulte [Publish Applications using AD FS Preauthentication][publish_applications_using_AD_FS_preauthentication] (Publicación de aplicaciones mediante la autenticación previa de Azure AD)</span><span class="sxs-lookup"><span data-stu-id="36a9d-199">For more information, see [Publish Applications using AD FS Preauthentication][publish_applications_using_AD_FS_preauthentication]</span></span>

<span data-ttu-id="36a9d-200">AD FS admite el aumento y la transformación de tokens.</span><span class="sxs-lookup"><span data-stu-id="36a9d-200">AD FS supports token transformation and augmentation.</span></span> <span data-ttu-id="36a9d-201">Azure Active Directory no proporciona esta característica.</span><span class="sxs-lookup"><span data-stu-id="36a9d-201">Azure Active Directory does not provide this feature.</span></span> <span data-ttu-id="36a9d-202">Con AD FS, al configurar las relaciones de confianza, puede:</span><span class="sxs-lookup"><span data-stu-id="36a9d-202">With AD FS, when you set up the trust relationships, you can:</span></span>

- <span data-ttu-id="36a9d-203">Configurar transformaciones de notificación para las reglas de autorización.</span><span class="sxs-lookup"><span data-stu-id="36a9d-203">Configure claim transformations for authorization rules.</span></span> <span data-ttu-id="36a9d-204">Por ejemplo, puede asignar la seguridad de grupo desde una representación utilizada por una organización asociada diferente de Microsoft a algo que Active Directory DS pueda autorizar en la organización.</span><span class="sxs-lookup"><span data-stu-id="36a9d-204">For example, you can map group security from a representation used by a non-Microsoft partner organization to something that that Active Directory DS can authorize in your organization.</span></span>
- <span data-ttu-id="36a9d-205">Transformar las notificaciones de un formato a otro.</span><span class="sxs-lookup"><span data-stu-id="36a9d-205">Transform claims from one format to another.</span></span> <span data-ttu-id="36a9d-206">Por ejemplo, puede pasar de SAML 2.0 a SAML 1.1, si la aplicación solo admite las notificaciones de SAML 1.1.</span><span class="sxs-lookup"><span data-stu-id="36a9d-206">For example, you can map from SAML 2.0 to SAML 1.1 if your application only supports SAML 1.1 claims.</span></span>

### <a name="ad-fs-monitoring"></a><span data-ttu-id="36a9d-207">Supervisión de AD FS</span><span class="sxs-lookup"><span data-stu-id="36a9d-207">AD FS monitoring</span></span>

<span data-ttu-id="36a9d-208">El [Módulo de administración de Microsoft System Center para Active Directory Federation Services 2012 R2][oms-adfs-pack] proporciona la supervisión tanto reactiva como proactiva de la implementación de AD FS para el servidor de federación.</span><span class="sxs-lookup"><span data-stu-id="36a9d-208">The [Microsoft System Center Management Pack for Active Directory Federation Services 2012 R2][oms-adfs-pack] provides both proactive and reactive monitoring of your AD FS deployment for the federation server.</span></span> <span data-ttu-id="36a9d-209">Este módulo de administración supervisa:</span><span class="sxs-lookup"><span data-stu-id="36a9d-209">This management pack monitors:</span></span>

- <span data-ttu-id="36a9d-210">Los eventos que el servicio AD FS incluye en sus registros de eventos.</span><span class="sxs-lookup"><span data-stu-id="36a9d-210">Events that the AD FS service records in its event logs.</span></span>
- <span data-ttu-id="36a9d-211">Los datos de rendimiento que los contadores de rendimiento de AD FS recopilan.</span><span class="sxs-lookup"><span data-stu-id="36a9d-211">The performance data that the AD FS performance counters collect.</span></span>
- <span data-ttu-id="36a9d-212">El estado general del sistema AD FS y las aplicaciones web (usuarios de confianza), además de proporcionar alertas para los problemas críticos y las advertencias.</span><span class="sxs-lookup"><span data-stu-id="36a9d-212">The overall health of the AD FS system and web applications (relying parties), and provides alerts for critical issues and warnings.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="36a9d-213">Consideraciones sobre escalabilidad</span><span class="sxs-lookup"><span data-stu-id="36a9d-213">Scalability considerations</span></span>

<span data-ttu-id="36a9d-214">Las consideraciones siguientes, que se resumen a partir del artículo [Planear la implementación de AD FS][plan-your-adfs-deployment], proporcionan un punto de partida para ajustar el tamaño de las granjas de servidores de AD FS:</span><span class="sxs-lookup"><span data-stu-id="36a9d-214">The following considerations, summarized from the article [Plan your AD FS deployment][plan-your-adfs-deployment], give a starting point for sizing AD FS farms:</span></span>

- <span data-ttu-id="36a9d-215">Si tiene menos de 1 000 usuarios, no cree servidores dedicados, sino que, en su lugar, instale AD FS en cada uno de los servidores de Active Directory DS de la nube.</span><span class="sxs-lookup"><span data-stu-id="36a9d-215">If you have fewer than 1000 users, do not create dedicated servers, but instead install AD FS on each of the Active Directory DS servers in the cloud.</span></span> <span data-ttu-id="36a9d-216">Asegúrese de que tiene al menos dos servidores de Active Directory DS para mantener la disponibilidad.</span><span class="sxs-lookup"><span data-stu-id="36a9d-216">Make sure that you have at least two Active Directory DS servers to maintain availability.</span></span> <span data-ttu-id="36a9d-217">Cree un único servidor WAP.</span><span class="sxs-lookup"><span data-stu-id="36a9d-217">Create a single WAP server.</span></span>
- <span data-ttu-id="36a9d-218">Si tiene entre 1 000 y 15 000 usuarios, cree dos servidores de AD FS dedicados y dos servidores WAP dedicados.</span><span class="sxs-lookup"><span data-stu-id="36a9d-218">If you have between 1000 and 15000 users, create two dedicated AD FS servers and two dedicated WAP servers.</span></span>
- <span data-ttu-id="36a9d-219">Si tiene entre 15 000 y 60 000 usuarios, cree entre tres y cinco servidores AD FS dedicados y al menos dos servidores WAP dedicados.</span><span class="sxs-lookup"><span data-stu-id="36a9d-219">If you have between 15000 and 60000 users, create between three and five dedicated AD FS servers and at least two dedicated WAP servers.</span></span>

<span data-ttu-id="36a9d-220">En estas consideraciones se supone que se utilizan los tamaños de máquinas virtuales duales de cuatro núcleos (D4_v2 estándar o posterior) en Azure.</span><span class="sxs-lookup"><span data-stu-id="36a9d-220">These considerations assume that you are using dual quad-core VM (Standard D4_v2, or better) sizes in Azure.</span></span>

<span data-ttu-id="36a9d-221">Si usa Windows Internal Database para almacenar los datos de configuración de AD FS, estará limitado a ocho servidores de AD FS en la granja de servidores.</span><span class="sxs-lookup"><span data-stu-id="36a9d-221">If you are using the Windows Internal Database to store AD FS configuration data, you are limited to eight AD FS servers in the farm.</span></span> <span data-ttu-id="36a9d-222">Si prevé que necesitará más en el futuro, utilice SQL Server.</span><span class="sxs-lookup"><span data-stu-id="36a9d-222">If you anticipate that you will need more in the future, use SQL Server.</span></span> <span data-ttu-id="36a9d-223">Para más información, consulte [La función de la base de datos de configuración de AD FS][adfs-configuration-database].</span><span class="sxs-lookup"><span data-stu-id="36a9d-223">For more information, see [The Role of the AD FS Configuration Database][adfs-configuration-database].</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="36a9d-224">Consideraciones sobre disponibilidad</span><span class="sxs-lookup"><span data-stu-id="36a9d-224">Availability considerations</span></span>

<span data-ttu-id="36a9d-225">Cree una granja de servidores de AD FS con al menos dos servidores para aumentar la disponibilidad del servicio.</span><span class="sxs-lookup"><span data-stu-id="36a9d-225">Create an AD FS farm with at least two servers to increase availability of the service.</span></span> <span data-ttu-id="36a9d-226">Use cuentas de almacenamiento diferentes para cada máquina virtual de AD FS en la granja de servidores.</span><span class="sxs-lookup"><span data-stu-id="36a9d-226">Use different storage accounts for each AD FS VM in the farm.</span></span> <span data-ttu-id="36a9d-227">Este enfoque ayuda a garantizar que un error en una única cuenta de almacenamiento no haga que toda la granja quede inaccesible.</span><span class="sxs-lookup"><span data-stu-id="36a9d-227">This approach helps to ensure that a failure in a single storage account does not make the entire farm inaccessible.</span></span>

<span data-ttu-id="36a9d-228">Cree conjuntos de disponibilidad de Azure diferentes para las máquinas virtuales de AD FS y WAP.</span><span class="sxs-lookup"><span data-stu-id="36a9d-228">Create separate Azure availability sets for the AD FS and WAP VMs.</span></span> <span data-ttu-id="36a9d-229">Asegúrese de que hay al menos dos máquinas virtuales en cada conjunto.</span><span class="sxs-lookup"><span data-stu-id="36a9d-229">Ensure that there are at least two VMs in each set.</span></span> <span data-ttu-id="36a9d-230">Cada conjunto de disponibilidad debe tener dos dominios de actualización y dos dominios de error, como mínimo.</span><span class="sxs-lookup"><span data-stu-id="36a9d-230">Each availability set must have at least two update domains and two fault domains.</span></span>

<span data-ttu-id="36a9d-231">Configure los equilibradores de carga para las máquinas virtuales de AD FS y WAP de la manera siguiente:</span><span class="sxs-lookup"><span data-stu-id="36a9d-231">Configure the load balancers for the AD FS VMs and WAP VMs as follows:</span></span>

- <span data-ttu-id="36a9d-232">Use un equilibrador de carga de Azure para proporcionar acceso externo a las máquinas virtuales de WAP y uno interno para distribuir la carga entre los servidores de AD FS en la granja de servidores.</span><span class="sxs-lookup"><span data-stu-id="36a9d-232">Use an Azure load balancer to provide external access to the WAP VMs, and an internal load balancer to distribute the load across the AD FS servers in the farm.</span></span>
- <span data-ttu-id="36a9d-233">Pase únicamente el tráfico que aparezca en el puerto 443 (HTTPS) a los servidores de AD FS y WAP.</span><span class="sxs-lookup"><span data-stu-id="36a9d-233">Only pass traffic appearing on port 443 (HTTPS) to the AD FS/WAP servers.</span></span>
- <span data-ttu-id="36a9d-234">Asigne al equilibrador de carga una dirección IP estática.</span><span class="sxs-lookup"><span data-stu-id="36a9d-234">Give the load balancer a static IP address.</span></span>
- <span data-ttu-id="36a9d-235">Crear un sondeo de mantenimiento mediante HTTP en `/adfs/probe`.</span><span class="sxs-lookup"><span data-stu-id="36a9d-235">Create a health probe using HTTP against `/adfs/probe`.</span></span> <span data-ttu-id="36a9d-236">Para más información, consulte [Hardware Load Balancer Health Checks and Web Application Proxy / AD FS 2012 R2](https://blogs.technet.microsoft.com/applicationproxyblog/2014/10/17/hardware-load-balancer-health-checks-and-web-application-proxy-ad-fs-2012-r2/) (Comprobaciones de mantenimiento de equilibrador de carga de hardware y proxy de aplicación web / AD FS 2012 R2).</span><span class="sxs-lookup"><span data-stu-id="36a9d-236">For more information, see [Hardware Load Balancer Health Checks and Web Application Proxy / AD FS 2012 R2](https://blogs.technet.microsoft.com/applicationproxyblog/2014/10/17/hardware-load-balancer-health-checks-and-web-application-proxy-ad-fs-2012-r2/).</span></span>

  > [!NOTE]
  > <span data-ttu-id="36a9d-237">Los servidores de AD FS usan el protocolo Indicación de nombre de servidor (SNI), por lo que el intento de sondear con un punto de conexión HTTPS desde el equilibrador de carga fracasa.</span><span class="sxs-lookup"><span data-stu-id="36a9d-237">AD FS servers use the Server Name Indication (SNI) protocol, so attempting to probe using an HTTPS endpoint from the load balancer fails.</span></span>
  >

- <span data-ttu-id="36a9d-238">Agregue un registro *A* DNS al dominio para el equilibrador de carga de AD FS.</span><span class="sxs-lookup"><span data-stu-id="36a9d-238">Add a DNS *A* record to the domain for the AD FS load balancer.</span></span> <span data-ttu-id="36a9d-239">Especifique la dirección IP del equilibrador de carga y asígnele un nombre en el dominio (por ejemplo, adfs.contoso.com).</span><span class="sxs-lookup"><span data-stu-id="36a9d-239">Specify the IP address of the load balancer, and give it a name in the domain (such as adfs.contoso.com).</span></span> <span data-ttu-id="36a9d-240">Se trata del nombre que los clientes y los servidores WAP usan para tener acceso a la granja de servidores de AD FS.</span><span class="sxs-lookup"><span data-stu-id="36a9d-240">This is the name clients and the WAP servers use to access the AD FS server farm.</span></span>

<span data-ttu-id="36a9d-241">Puede usar SQL Server o Windows Internal Database para contener la información de configuración de AD FS.</span><span class="sxs-lookup"><span data-stu-id="36a9d-241">You can use either SQL Server or the Windows Internal Database to hold AD FS configuration information.</span></span> <span data-ttu-id="36a9d-242">Windows Internal Database proporciona redundancia básica.</span><span class="sxs-lookup"><span data-stu-id="36a9d-242">The Windows Internal Database provides basic redundancy.</span></span> <span data-ttu-id="36a9d-243">Los cambios se escriben directamente en una de las bases de datos de AD FS en el clúster de AD FS, mientras que los demás servidores usan replicación de extracción para mantener sus bases de datos actualizadas.</span><span class="sxs-lookup"><span data-stu-id="36a9d-243">Changes are written directly to only one of the AD FS databases in the AD FS cluster, while the other servers use pull replication to keep their databases up to date.</span></span> <span data-ttu-id="36a9d-244">Con SQL Server, puede proporcionar una redundancia completa de las bases de datos y una elevada disponibilidad mediante clústeres de conmutación por error o la creación de reflejo.</span><span class="sxs-lookup"><span data-stu-id="36a9d-244">Using SQL Server can provide full database redundancy and high availability using failover clustering or mirroring.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="36a9d-245">Consideraciones sobre la manejabilidad</span><span class="sxs-lookup"><span data-stu-id="36a9d-245">Manageability considerations</span></span>

<span data-ttu-id="36a9d-246">El personal de DevOps debe estar preparado para realizar las siguientes tareas:</span><span class="sxs-lookup"><span data-stu-id="36a9d-246">DevOps staff should be prepared to perform the following tasks:</span></span>

- <span data-ttu-id="36a9d-247">Administración de los servidores de federación, incluida la administración de la granja de servidores de AD FS, de la directiva de confianza en los servidores de federación y de los certificados usados por los servicios de federación.</span><span class="sxs-lookup"><span data-stu-id="36a9d-247">Managing the federation servers, including managing the AD FS farm, managing trust policy on the federation servers, and managing the certificates used by the federation services.</span></span>
- <span data-ttu-id="36a9d-248">Administración de los servidores WAP, incluida la administración de la granja de servidores WAP y certificados.</span><span class="sxs-lookup"><span data-stu-id="36a9d-248">Managing the WAP servers including managing the WAP farm and certificates.</span></span>
- <span data-ttu-id="36a9d-249">Administración de las aplicaciones web, incluida la configuración de los usuarios de confianza, los métodos de autenticación y las asignaciones de notificaciones.</span><span class="sxs-lookup"><span data-stu-id="36a9d-249">Managing web applications including configuring relying parties, authentication methods, and claims mappings.</span></span>
- <span data-ttu-id="36a9d-250">Copia de seguridad de los componentes de AD FS.</span><span class="sxs-lookup"><span data-stu-id="36a9d-250">Backing up AD FS components.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="36a9d-251">Consideraciones sobre la seguridad</span><span class="sxs-lookup"><span data-stu-id="36a9d-251">Security considerations</span></span>

<span data-ttu-id="36a9d-252">AD FS usa HTTPS, así que asegúrese de que las reglas de NSG para la subred que contiene las máquinas virtuales del nivel web permiten solicitudes de HTTPS.</span><span class="sxs-lookup"><span data-stu-id="36a9d-252">AD FS uses HTTPS, so make sure that the NSG rules for the subnet containing the web tier VMs permit HTTPS requests.</span></span> <span data-ttu-id="36a9d-253">Estas solicitudes pueden originarse en la red local, las subredes que contienen el nivel web, el nivel empresarial, el nivel de datos, la red perimetral privada, la red perimetral pública y la subred que contiene los servidores de AD FS.</span><span class="sxs-lookup"><span data-stu-id="36a9d-253">These requests can originate from the on-premises network, the subnets containing the web tier, business tier, data tier, private DMZ, public DMZ, and the subnet containing the AD FS servers.</span></span>

<span data-ttu-id="36a9d-254">Evite la exposición directa de los servidores de AD FS a Internet.</span><span class="sxs-lookup"><span data-stu-id="36a9d-254">Prevent direct exposure of the AD FS servers to the Internet.</span></span> <span data-ttu-id="36a9d-255">Los servidores de AD FS son equipos unidos a un dominio que tienen autorización completa para conceder tokens de seguridad.</span><span class="sxs-lookup"><span data-stu-id="36a9d-255">AD FS servers are domain-joined computers that have full authorization to grant security tokens.</span></span> <span data-ttu-id="36a9d-256">Si un servidor se ve comprometido, un usuario malintencionado puede emitir tokens de acceso completo a todas las aplicaciones web y a todos los servidores de federación que estén protegidos por AD FS.</span><span class="sxs-lookup"><span data-stu-id="36a9d-256">If a server is compromised, a malicious user can issue full access tokens to all web applications and to all federation servers that are protected by AD FS.</span></span> <span data-ttu-id="36a9d-257">Si el sistema debe controlar las solicitudes de los usuarios externos que no se conectan desde sitios de confianza asociados, use los servidores WAP para controlar estas solicitudes.</span><span class="sxs-lookup"><span data-stu-id="36a9d-257">If your system must handle requests from external users not connecting from trusted partner sites, use WAP servers to handle these requests.</span></span> <span data-ttu-id="36a9d-258">Para más información, consulte [Ubicación de un servidor proxy de federación][where-to-place-an-fs-proxy].</span><span class="sxs-lookup"><span data-stu-id="36a9d-258">For more information, see [Where to Place a Federation Server Proxy][where-to-place-an-fs-proxy].</span></span>

<span data-ttu-id="36a9d-259">Coloque los servidores de AD FS y los servidores WAP en subredes independientes con sus propios firewalls.</span><span class="sxs-lookup"><span data-stu-id="36a9d-259">Place AD FS servers and WAP servers in separate subnets with their own firewalls.</span></span> <span data-ttu-id="36a9d-260">Puede usar las reglas NSG para definir las reglas de firewall.</span><span class="sxs-lookup"><span data-stu-id="36a9d-260">You can use NSG rules to define firewall rules.</span></span> <span data-ttu-id="36a9d-261">Todos los firewalls deben permitir el tráfico en el puerto 443 (HTTPS).</span><span class="sxs-lookup"><span data-stu-id="36a9d-261">All firewalls should allow traffic on port 443 (HTTPS).</span></span>

<span data-ttu-id="36a9d-262">Restrinja el acceso de inicio de sesión directo a los servidores de AD FS y WAP.</span><span class="sxs-lookup"><span data-stu-id="36a9d-262">Restrict direct sign in access to the AD FS and WAP servers.</span></span> <span data-ttu-id="36a9d-263">Solo el personal de DevOps debe ser capaz de conectarse.</span><span class="sxs-lookup"><span data-stu-id="36a9d-263">Only DevOps staff should be able to connect.</span></span> <span data-ttu-id="36a9d-264">No una los servidores WAP al dominio.</span><span class="sxs-lookup"><span data-stu-id="36a9d-264">Do not join the WAP servers to the domain.</span></span>

<span data-ttu-id="36a9d-265">Considere usar un conjunto de aplicaciones de red virtual que registre información detallada sobre el tráfico que atraviesa la frontera de la red virtual con fines de auditoría.</span><span class="sxs-lookup"><span data-stu-id="36a9d-265">Consider using a set of network virtual appliances that logs detailed information on traffic traversing the edge of your virtual network for auditing purposes.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="36a9d-266">Implementación de la solución</span><span class="sxs-lookup"><span data-stu-id="36a9d-266">Deploy the solution</span></span>

<span data-ttu-id="36a9d-267">Hay disponible una implementación de esta arquitectura en [GitHub][github].</span><span class="sxs-lookup"><span data-stu-id="36a9d-267">A deployment for this architecture is available on [GitHub][github].</span></span> <span data-ttu-id="36a9d-268">Tenga en cuenta que la implementación completa puede durar un máximo de dos horas, lo que incluye la creación de una instancia de VPN Gateway y la ejecución de los scripts que configuran Active Directory y AD FS.</span><span class="sxs-lookup"><span data-stu-id="36a9d-268">Note that the entire deployment can take up to two hours, which includes creating the VPN gateway and running the scripts that configure Active Directory and AD FS.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="36a9d-269">Requisitos previos</span><span class="sxs-lookup"><span data-stu-id="36a9d-269">Prerequisites</span></span>

1. <span data-ttu-id="36a9d-270">Clone, bifurque o descargue el archivo zip del [repositorio de GitHub](https://github.com/mspnp/identity-reference-architectures).</span><span class="sxs-lookup"><span data-stu-id="36a9d-270">Clone, fork, or download the zip file for the [GitHub repository](https://github.com/mspnp/identity-reference-architectures).</span></span>

1. <span data-ttu-id="36a9d-271">Instale la [CLI de Azure 2.0](/cli/azure/install-azure-cli?view=azure-cli-latest).</span><span class="sxs-lookup"><span data-stu-id="36a9d-271">Install [Azure CLI 2.0](/cli/azure/install-azure-cli?view=azure-cli-latest).</span></span>

1. <span data-ttu-id="36a9d-272">Instale el paquete de npm de [bloques de creación de Azure](https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks).</span><span class="sxs-lookup"><span data-stu-id="36a9d-272">Install the [Azure building blocks](https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks) npm package.</span></span>

   ```bash
   npm install -g @mspnp/azure-building-blocks
   ```

1. <span data-ttu-id="36a9d-273">Desde un símbolo del sistema, un símbolo del sistema de Bash o un símbolo del sistema de PowerShell, inicie sesión en su cuenta de Azure como se indica a continuación:</span><span class="sxs-lookup"><span data-stu-id="36a9d-273">From a command prompt, bash prompt, or PowerShell prompt, sign into your Azure account as follows:</span></span>

   ```bash
   az login
   ```

### <a name="deploy-the-simulated-on-premises-datacenter"></a><span data-ttu-id="36a9d-274">Implementación del centro de datos local simulado</span><span class="sxs-lookup"><span data-stu-id="36a9d-274">Deploy the simulated on-premises datacenter</span></span>

1. <span data-ttu-id="36a9d-275">Vaya a la carpeta `adfs` del repositorio de GitHub.</span><span class="sxs-lookup"><span data-stu-id="36a9d-275">Navigate to the `adfs` folder of the GitHub repository.</span></span>

1. <span data-ttu-id="36a9d-276">Abra el archivo `onprem.json` .</span><span class="sxs-lookup"><span data-stu-id="36a9d-276">Open the `onprem.json` file.</span></span> <span data-ttu-id="36a9d-277">Busque instancias de `adminPassword`, `Password` y `SafeModeAdminPassword`, y actualice las contraseñas.</span><span class="sxs-lookup"><span data-stu-id="36a9d-277">Search for instances of `adminPassword`, `Password`, and `SafeModeAdminPassword` and update the passwords.</span></span>

1. <span data-ttu-id="36a9d-278">Ejecute el siguiente comando y espere a que finalice la implementación:</span><span class="sxs-lookup"><span data-stu-id="36a9d-278">Run the following command and wait for the deployment to finish:</span></span>

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p onprem.json --deploy
    ```

### <a name="deploy-the-azure-infrastructure"></a><span data-ttu-id="36a9d-279">Implementación de la infraestructura de Azure</span><span class="sxs-lookup"><span data-stu-id="36a9d-279">Deploy the Azure infrastructure</span></span>

1. <span data-ttu-id="36a9d-280">Abra el archivo `azure.json` .</span><span class="sxs-lookup"><span data-stu-id="36a9d-280">Open the `azure.json` file.</span></span>  <span data-ttu-id="36a9d-281">Busque instancias de `adminPassword` y `Password` y agregue valores para las contraseñas.</span><span class="sxs-lookup"><span data-stu-id="36a9d-281">Search for instances of `adminPassword` and `Password` and add values for the passwords.</span></span>

1. <span data-ttu-id="36a9d-282">Ejecute el siguiente comando y espere a que finalice la implementación:</span><span class="sxs-lookup"><span data-stu-id="36a9d-282">Run the following command and wait for the deployment to finish:</span></span>

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p azure.json --deploy
    ```

### <a name="set-up-the-ad-fs-farm"></a><span data-ttu-id="36a9d-283">Configuración de la granja de AD FS</span><span class="sxs-lookup"><span data-stu-id="36a9d-283">Set up the AD FS farm</span></span>

1. <span data-ttu-id="36a9d-284">Abra el archivo `adfs-farm-first.json` .</span><span class="sxs-lookup"><span data-stu-id="36a9d-284">Open the `adfs-farm-first.json` file.</span></span>  <span data-ttu-id="36a9d-285">Busque `AdminPassword` y reemplace la contraseña predeterminada.</span><span class="sxs-lookup"><span data-stu-id="36a9d-285">Search for `AdminPassword` and replace the default password.</span></span>

1. <span data-ttu-id="36a9d-286">Ejecute el siguiente comando:</span><span class="sxs-lookup"><span data-stu-id="36a9d-286">Run the following command:</span></span>

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p adfs-farm-first.json --deploy
    ```

1. <span data-ttu-id="36a9d-287">Abra el archivo `adfs-farm-rest.json` .</span><span class="sxs-lookup"><span data-stu-id="36a9d-287">Open the `adfs-farm-rest.json` file.</span></span>  <span data-ttu-id="36a9d-288">Busque `AdminPassword` y reemplace la contraseña predeterminada.</span><span class="sxs-lookup"><span data-stu-id="36a9d-288">Search for `AdminPassword` and replace the default password.</span></span>

1. <span data-ttu-id="36a9d-289">Ejecute el siguiente comando y espere a que finalice la implementación:</span><span class="sxs-lookup"><span data-stu-id="36a9d-289">Run the following command and wait for the deployment to finish:</span></span>

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p adfs-farm-rest.json --deploy
    ```

### <a name="configure-ad-fs-part-1"></a><span data-ttu-id="36a9d-290">Configuración de AD FS (parte 1)</span><span class="sxs-lookup"><span data-stu-id="36a9d-290">Configure AD FS (part 1)</span></span>

1. <span data-ttu-id="36a9d-291">Abra una sesión de escritorio remoto en la máquina virtual denominada `ra-adfs-jb-vm1`, que es la máquina virtual del jumpbox.</span><span class="sxs-lookup"><span data-stu-id="36a9d-291">Open a remote desktop session to the VM named `ra-adfs-jb-vm1`, which is the jumpbox VM.</span></span> <span data-ttu-id="36a9d-292">El nombre de usuario es `testuser`.</span><span class="sxs-lookup"><span data-stu-id="36a9d-292">The user name is `testuser`.</span></span>

1. <span data-ttu-id="36a9d-293">En el jumpbox, abra una sesión de escritorio remoto en la máquina virtual denominada `ra-adfs-proxy-vm1`.</span><span class="sxs-lookup"><span data-stu-id="36a9d-293">From the jumpbox, open a remote desktop session to the VM named `ra-adfs-proxy-vm1`.</span></span> <span data-ttu-id="36a9d-294">La dirección IP privada es 10.0.6.4.</span><span class="sxs-lookup"><span data-stu-id="36a9d-294">The private IP address is 10.0.6.4.</span></span>

1. <span data-ttu-id="36a9d-295">En esta sesión de escritorio remoto, ejecute [PowerShell ISE](/powershell/scripting/components/ise/windows-powershell-integrated-scripting-environment--ise-).</span><span class="sxs-lookup"><span data-stu-id="36a9d-295">From this remote desktop session, run the [PowerShell ISE](/powershell/scripting/components/ise/windows-powershell-integrated-scripting-environment--ise-).</span></span>

1. <span data-ttu-id="36a9d-296">En PowerShell, vaya al siguiente directorio:</span><span class="sxs-lookup"><span data-stu-id="36a9d-296">In PowerShell, navigate to the following directory:</span></span>

    ```powershell
    C:\Packages\Plugins\Microsoft.Powershell.DSC\2.77.0.0\DSCWork\adfs-v2.0
    ```

1. <span data-ttu-id="36a9d-297">Pegue el código siguiente en un panel de scripts y ejecútelo:</span><span class="sxs-lookup"><span data-stu-id="36a9d-297">Paste the following code into a script pane and run it:</span></span>

    ```powershell
    . .\adfs-webproxy.ps1
    $cd = @{
        AllNodes = @(
            @{
                NodeName = 'localhost'
                PSDscAllowPlainTextPassword = $true
                PSDscAllowDomainUser = $true
            }
        )
    }

    $c1 = Get-Credential -UserName testuser -Message "Enter password"
    InstallWebProxyApp -DomainName contoso.com -FederationName adfs.contoso.com -WebApplicationProxyName "Contoso App" -AdminCreds $c1 -ConfigurationData $cd
    Start-DscConfiguration .\InstallWebProxyApp
    ```

    <span data-ttu-id="36a9d-298">En el indicador `Get-Credential`, escriba la contraseña que especificó en el archivo de parámetros de la implementación.</span><span class="sxs-lookup"><span data-stu-id="36a9d-298">At the `Get-Credential` prompt, enter the password that you specified in the deployment parameter file.</span></span>

1. <span data-ttu-id="36a9d-299">Ejecute el comando siguiente para supervisar el progreso de la configuración de [DSC](/powershell/dsc/overview/overview):</span><span class="sxs-lookup"><span data-stu-id="36a9d-299">Run the following command to monitor the progress of the [DSC](/powershell/dsc/overview/overview) configuration:</span></span>

    ```powershell
    Get-DscConfigurationStatus
    ```

    <span data-ttu-id="36a9d-300">La consistencia puede tardar varios minutos en lograrse.</span><span class="sxs-lookup"><span data-stu-id="36a9d-300">It can take several minutes to reach consistency.</span></span> <span data-ttu-id="36a9d-301">Durante este tiempo, es posible que vea errores del comando.</span><span class="sxs-lookup"><span data-stu-id="36a9d-301">During this time, you may see errors from the command.</span></span> <span data-ttu-id="36a9d-302">Si la configuración finaliza de forma satisfactoria, el resultado debe ser similar al siguiente:</span><span class="sxs-lookup"><span data-stu-id="36a9d-302">When the configuration succeeds, the output should look similar to the following:</span></span>

    ```powershell
    PS C:\Packages\Plugins\Microsoft.Powershell.DSC\2.77.0.0\DSCWork\adfs-v2.0> Get-DscConfigurationStatus

    Status     StartDate                 Type            Mode  RebootRequested      NumberOfResources
    ------     ---------                 ----            ----  ---------------      -----------------
    Success    12/17/2018 8:21:09 PM     Consistency     PUSH  True                 4
    ```

### <a name="configure-ad-fs-part-2"></a><span data-ttu-id="36a9d-303">Configuración de AD FS (parte 2)</span><span class="sxs-lookup"><span data-stu-id="36a9d-303">Configure AD FS (part 2)</span></span>

1. <span data-ttu-id="36a9d-304">En el jumpbox, abra una sesión de escritorio remoto en la máquina virtual denominada `ra-adfs-proxy-vm2`.</span><span class="sxs-lookup"><span data-stu-id="36a9d-304">From the jumpbox, open a remote desktop session to the VM named `ra-adfs-proxy-vm2`.</span></span> <span data-ttu-id="36a9d-305">La dirección IP privada es 10.0.6.5.</span><span class="sxs-lookup"><span data-stu-id="36a9d-305">The private IP address is 10.0.6.5.</span></span>

1. <span data-ttu-id="36a9d-306">En esta sesión de escritorio remoto, ejecute [PowerShell ISE](/powershell/scripting/components/ise/windows-powershell-integrated-scripting-environment--ise-).</span><span class="sxs-lookup"><span data-stu-id="36a9d-306">From this remote desktop session, run the [PowerShell ISE](/powershell/scripting/components/ise/windows-powershell-integrated-scripting-environment--ise-).</span></span>

1. <span data-ttu-id="36a9d-307">Vaya al siguiente directorio:</span><span class="sxs-lookup"><span data-stu-id="36a9d-307">Navigate to the following directory:</span></span>

    ```powershell
    C:\Packages\Plugins\Microsoft.Powershell.DSC\2.77.0.0\DSCWork\adfs-v2.0
    ```

1. <span data-ttu-id="36a9d-308">Pegue lo siguiente en un panel de scripts y ejecute el script:</span><span class="sxs-lookup"><span data-stu-id="36a9d-308">Past the following in a script pane and run the script:</span></span>

    ```powershell
    . .\adfs-webproxy-rest.ps1
    $cd = @{
        AllNodes = @(
            @{
                NodeName = 'localhost'
                PSDscAllowPlainTextPassword = $true
                PSDscAllowDomainUser = $true
            }
        )
    }

    $c1 = Get-Credential -UserName testuser -Message "Enter password"
    InstallWebProxy -DomainName contoso.com -FederationName adfs.contoso.com -WebApplicationProxyName "Contoso App" -AdminCreds $c1 -ConfigurationData $cd
    Start-DscConfiguration .\InstallWebProxy
    ```

    <span data-ttu-id="36a9d-309">En el indicador `Get-Credential`, escriba la contraseña que especificó en el archivo de parámetros de la implementación.</span><span class="sxs-lookup"><span data-stu-id="36a9d-309">At the `Get-Credential` prompt, enter the password that you specified in the deployment parameter file.</span></span>

1. <span data-ttu-id="36a9d-310">Ejecute el comando siguiente para supervisar el progreso de la configuración de DSC:</span><span class="sxs-lookup"><span data-stu-id="36a9d-310">Run the following command to monitor the progress of the DSC configuration:</span></span>

    ```powershell
    Get-DscConfigurationStatus
    ```

    <span data-ttu-id="36a9d-311">La consistencia puede tardar varios minutos en lograrse.</span><span class="sxs-lookup"><span data-stu-id="36a9d-311">It can take several minutes to reach consistency.</span></span> <span data-ttu-id="36a9d-312">Durante este tiempo, es posible que vea errores del comando.</span><span class="sxs-lookup"><span data-stu-id="36a9d-312">During this time, you may see errors from the command.</span></span> <span data-ttu-id="36a9d-313">Si la configuración finaliza de forma satisfactoria, el resultado debe ser similar al siguiente:</span><span class="sxs-lookup"><span data-stu-id="36a9d-313">When the configuration succeeds, the output should look similar to the following:</span></span>

    ```powershell
    PS C:\Packages\Plugins\Microsoft.Powershell.DSC\2.77.0.0\DSCWork\adfs-v2.0> Get-DscConfigurationStatus

    Status     StartDate                 Type            Mode  RebootRequested      NumberOfResources
    ------     ---------                 ----            ----  ---------------      -----------------
    Success    12/17/2018 8:21:09 PM     Consistency     PUSH  True                 4
    ```

    <span data-ttu-id="36a9d-314">En ocasiones, se producen errores en esta DSC.</span><span class="sxs-lookup"><span data-stu-id="36a9d-314">Sometimes this DSC fails.</span></span> <span data-ttu-id="36a9d-315">Si la comprobación del estado muestra `Status=Failure` y `Type=Consistency`, pruebe a volver a ejecutar el paso 4.</span><span class="sxs-lookup"><span data-stu-id="36a9d-315">If the status check shows `Status=Failure` and `Type=Consistency`, try re-running step 4.</span></span>

### <a name="sign-into-ad-fs"></a><span data-ttu-id="36a9d-316">Inicio de sesión en AD FS</span><span class="sxs-lookup"><span data-stu-id="36a9d-316">Sign into AD FS</span></span>

1. <span data-ttu-id="36a9d-317">En el jumpbox, abra una sesión de escritorio remoto en la máquina virtual denominada `ra-adfs-adfs-vm1`.</span><span class="sxs-lookup"><span data-stu-id="36a9d-317">From the jumpbox, open a remote desktop session to the VM named `ra-adfs-adfs-vm1`.</span></span> <span data-ttu-id="36a9d-318">La dirección IP privada es 10.0.5.4.</span><span class="sxs-lookup"><span data-stu-id="36a9d-318">The private IP address is 10.0.5.4.</span></span>

1. <span data-ttu-id="36a9d-319">Para habilitar la página de inicio de sesión, siga los pasos de [Enable the Idp-Intiated Sign on page](/windows-server/identity/ad-fs/troubleshooting/ad-fs-tshoot-initiatedsignon#enable-the-idp-intiated-sign-on-page) (Habilitación de la página de inicio de sesión iniciado por ldp).</span><span class="sxs-lookup"><span data-stu-id="36a9d-319">Follow the steps in [Enable the Idp-Intiated Sign on page](/windows-server/identity/ad-fs/troubleshooting/ad-fs-tshoot-initiatedsignon#enable-the-idp-intiated-sign-on-page) to enable the sign-on page.</span></span>

1. <span data-ttu-id="36a9d-320">En el jumpbox, vaya a `https://adfs.contoso.com/adfs/ls/idpinitiatedsignon.htm`.</span><span class="sxs-lookup"><span data-stu-id="36a9d-320">From the jump box, browse to `https://adfs.contoso.com/adfs/ls/idpinitiatedsignon.htm`.</span></span> <span data-ttu-id="36a9d-321">Puede recibir una advertencia que puede ignorar para esta prueba.</span><span class="sxs-lookup"><span data-stu-id="36a9d-321">You may receive a certificate warning that you can ignore for this test.</span></span>

1. <span data-ttu-id="36a9d-322">Compruebe que aparece la página de inicio de sesión de Contoso Corporation.</span><span class="sxs-lookup"><span data-stu-id="36a9d-322">Verify that the Contoso Corporation sign-in page appears.</span></span> <span data-ttu-id="36a9d-323">Inicie sesión como **contoso\testuser**.</span><span class="sxs-lookup"><span data-stu-id="36a9d-323">Sign in as **contoso\testuser**.</span></span>

<!-- links -->
[extending-ad-to-azure]: adds-extend-domain.md

[vm-recommendations]: ../virtual-machines-windows/single-vm.md
[implementing-a-secure-hybrid-network-architecture]: ../dmz/secure-vnet-hybrid.md
[implementing-a-secure-hybrid-network-architecture-with-internet-access]: ../dmz/secure-vnet-dmz.md
[hybrid-azure-on-prem-vpn]: ../hybrid-networking/vpn.md

[azure-cli]: /azure/azure-resource-manager/xplat-cli-azure-resource-manager
[DRS]: https://technet.microsoft.com/library/dn280945.aspx
[where-to-place-an-fs-proxy]: https://technet.microsoft.com/library/dd807048.aspx
[ADDRS]: https://technet.microsoft.com/library/dn486831.aspx
[plan-your-adfs-deployment]: https://msdn.microsoft.com/library/azure/dn151324.aspx
[ad_network_recommendations]: #network_configuration_recommendations_for_AD_DS_VMs
[adfs_certificates]: https://technet.microsoft.com/library/dn781428(v=ws.11).aspx
[create_service_account_for_adfs_farm]: https://technet.microsoft.com/library/dd807078.aspx
[adfs-configuration-database]: https://technet.microsoft.com/library/ee913581(v=ws.11).aspx
[active-directory-federation-services]: https://technet.microsoft.com/windowsserver/dd448613.aspx
[security-considerations]: #security-considerations
[recommendations]: #recommendations
[active-directory-federation-services-overview]: https://technet.microsoft.com/library/hh831502(v=ws.11).aspx
[establishing-federation-trust]: https://blogs.msdn.microsoft.com/alextch/2011/06/27/establishing-federation-trust/
[Deploying_a_federation_server_farm]:  /windows-server/identity/ad-fs/deployment/deploying-a-federation-server-farm
[install_and_configure_the_web_application_proxy_server]: https://technet.microsoft.com/library/dn383662.aspx
[publish_applications_using_AD_FS_preauthentication]: https://technet.microsoft.com/library/dn383640.aspx
[managing-adfs-components]: https://technet.microsoft.com/library/cc759026.aspx
[oms-adfs-pack]: https://www.microsoft.com/download/details.aspx?id=41184
[azure-powershell-download]: https://azure.microsoft.com/documentation/articles/powershell-install-configure/
[aad]: https://azure.microsoft.com/documentation/services/active-directory/
[aadb2c]: https://azure.microsoft.com/documentation/services/active-directory-b2c/
[adfs-intro]: /azure/active-directory/hybrid/whatis-hybrid-identity
[github]: https://github.com/mspnp/identity-reference-architectures/tree/master/adfs
[adfs_certificates]: https://technet.microsoft.com/library/dn781428(v=ws.11).aspx
[considerations]: ./considerations.md
[visio-download]: https://archcenter.blob.core.windows.net/cdn/identity-architectures.vsdx
