---
title: "Explicación: ¿Cómo funciona Azure?"
description: Explica el funcionamiento interno de Azure
author: petertay
ms.openlocfilehash: 847d24b7057d80f3d34aac7900cfb64fec60a640
ms.sourcegitcommit: 2e8b06e9c07875d65b91d5431bfd4bc465a7a242
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/09/2018
---
# <a name="explainer-how-does-azure-work"></a><span data-ttu-id="87d30-103">Explicación: ¿Cómo funciona Azure?</span><span class="sxs-lookup"><span data-stu-id="87d30-103">Explainer: How does Azure work?</span></span>

<span data-ttu-id="87d30-104">Azure es la plataforma pública en la nube de Microsoft.</span><span class="sxs-lookup"><span data-stu-id="87d30-104">Azure is Microsoft's public cloud platform.</span></span> <span data-ttu-id="87d30-105">Azure ofrece una amplia serie de servicios entre los que se incluyen la plataforma como servicio (PaaS), la infraestructura como servicio (IaaS), la base de datos como servicio (DBaaS) y muchos otros.</span><span class="sxs-lookup"><span data-stu-id="87d30-105">Azure offers a large collection of services including platform as a service (PaaS), infrastructure as a service (IaaS), database as a service (DBaaS), and many others.</span></span> <span data-ttu-id="87d30-106">Pero, ¿qué es exactamente Azure y cómo funciona?</span><span class="sxs-lookup"><span data-stu-id="87d30-106">But what exactly is Azure, and how does it work?</span></span>

<span data-ttu-id="87d30-107">Azure, al igual que otras plataformas en la nube, se basa en una tecnología conocida como **virtualización**.</span><span class="sxs-lookup"><span data-stu-id="87d30-107">Azure, like other cloud platforms, relies on a technology known as **virtualization**.</span></span> <span data-ttu-id="87d30-108">La mayor parte del hardware de equipo puede emularse en el software porque es simplemente un conjunto de instrucciones codificadas de forma permanente o semipermanente en silicio.</span><span class="sxs-lookup"><span data-stu-id="87d30-108">Most computer hardware can be emulated in software, because most computer hardware is simply a set of instructions permanently or semi-permanently encoded in silicon.</span></span> <span data-ttu-id="87d30-109">Mediante una capa de emulación que asigna instrucciones de software a instrucciones de hardware, el hardware virtualizado puede ejecutarse en el software como si fuera el propio hardware.</span><span class="sxs-lookup"><span data-stu-id="87d30-109">Using an emulation layer that maps software instructions to hardware instructions, virtualized hardware can execute in software as if it were the actual hardware itself.</span></span>

<span data-ttu-id="87d30-110">Básicamente, la nube es un conjunto de servidores físicos en uno o varios centros de datos que ejecutan hardware virtualizado en nombre de los clientes.</span><span class="sxs-lookup"><span data-stu-id="87d30-110">Essentially, the cloud is a set of physical servers in one or more datacenters that execute virtualized hardware on behalf of customers.</span></span> <span data-ttu-id="87d30-111">Entonces, ¿cómo crea, inicia, detiene y elimina la nube millones de instancias de hardware virtualizado para millones de clientes a la vez?</span><span class="sxs-lookup"><span data-stu-id="87d30-111">So how does the cloud create, start, stop, and delete millions of instances of virtualized hardware for millions of customers simultaneously?</span></span>

<span data-ttu-id="87d30-112">Para comprenderlo, vamos a echar un vistazo a la arquitectura del hardware del centro de datos.</span><span class="sxs-lookup"><span data-stu-id="87d30-112">To understand this, let's look at the architecture of the hardware in the datacenter.</span></span>  <span data-ttu-id="87d30-113">En cada centro de datos hay una colección de servidores que se encuentran en bastidores de servidores.</span><span class="sxs-lookup"><span data-stu-id="87d30-113">Within each datacenter is a collection of servers sitting in server racks.</span></span> <span data-ttu-id="87d30-114">Cada bastidor de servidor contiene muchas **hojas** de servidor, así como un conmutador de red que proporciona conectividad de red y una unidad de distribución de energía (PDU) que suministra la alimentación.</span><span class="sxs-lookup"><span data-stu-id="87d30-114">Each server rack contains many server **blades** as well as a network switch providing network connectivity and a power distribution unit (PDU) providing power.</span></span> <span data-ttu-id="87d30-115">A veces, los bastidores se agrupan conjuntamente en unidades más grandes que se conocen como **clústeres**.</span><span class="sxs-lookup"><span data-stu-id="87d30-115">Racks are sometimes grouped together in larger units known as **clusters**.</span></span> 

<span data-ttu-id="87d30-116">En cada bastidor o clúster, la mayoría de los servidores están designados para ejecutar estas instancias de hardware virtualizado en nombre del usuario.</span><span class="sxs-lookup"><span data-stu-id="87d30-116">Within each rack or cluster, most of the servers are designated to run these virtualized hardware instances on behalf of the user.</span></span> <span data-ttu-id="87d30-117">Sin embargo, algunos servidores ejecutan un software de administración en la nube que se conoce como controlador de tejido.</span><span class="sxs-lookup"><span data-stu-id="87d30-117">However, a number of the servers run cloud management software known as a fabric controller.</span></span> <span data-ttu-id="87d30-118">El **controlador de tejido** es una aplicación distribuida con muchas responsabilidades.</span><span class="sxs-lookup"><span data-stu-id="87d30-118">The **fabric controller** is a distributed application with many responsibilities.</span></span> <span data-ttu-id="87d30-119">Asigna servicios, supervisa el mantenimiento del servidor y los servicios que se ejecutan en él, y recupera los servidores cuando se produce un error.</span><span class="sxs-lookup"><span data-stu-id="87d30-119">It allocates services, monitors the health of the server and the services running on it, and heals servers when they fail.</span></span>

<span data-ttu-id="87d30-120">Cada instancia del controlador de tejido se conecta a otro conjunto de servidores que ejecutan software de orquestación en la nube, conocido normalmente como **front-end**.</span><span class="sxs-lookup"><span data-stu-id="87d30-120">Each instance of the fabric controller is connected to another set of servers running cloud orchestration software, typically known as a **front end**.</span></span> <span data-ttu-id="87d30-121">El front-end hospeda los servicios web, las API RESTful y las bases de datos internas de Azure utilizadas para todas las funciones que realiza la nube.</span><span class="sxs-lookup"><span data-stu-id="87d30-121">The front end hosts the web services, RESTful APIs, and internal Azure databases used for all functions the cloud performs.</span></span> 

<span data-ttu-id="87d30-122">Por ejemplo, el front-end hospeda los servicios que controlan las solicitudes de cliente para asignar recursos de Azure como [redes virtuales][vnet], [máquinas virtuales][vms] y servicios como [CosmosDB].</span><span class="sxs-lookup"><span data-stu-id="87d30-122">For example, the front end hosts the services that handle customer requests to allocate Azure resources such as [virtual networks][vnet], [virtual machines][vms], and services like [CosmosDB].</span></span> <span data-ttu-id="87d30-123">En un primer tiempo, el front-end valida el usuario y comprueba que esté autorizado para asignar los recursos solicitados.</span><span class="sxs-lookup"><span data-stu-id="87d30-123">First, the front end validates the user and verifies the user is authorized to allocate the requested resources.</span></span> <span data-ttu-id="87d30-124">Si es así, el front-end consulta una base de datos para buscar un bastidor de servidor con una capacidad suficiente y, luego, indica al controlador de tejido del bastidor que asigne el recurso.</span><span class="sxs-lookup"><span data-stu-id="87d30-124">If so, the front end consults a database to locate a server rack with sufficient capacity, and then instructs the fabric controller on the rack to allocate the resource.</span></span>

<span data-ttu-id="87d30-125">Por lo tanto, y para simplificar, Azure es una inmensa colección de servidores y de hardware de red, junto con un complejo conjunto de aplicaciones distribuidas que orquesta la configuración y el funcionamiento del hardware virtualizado y del software de estos servidores.</span><span class="sxs-lookup"><span data-stu-id="87d30-125">So, very simply, Azure is a huge collection of servers and networking hardware, along with a complex set of distributed applications that orchestrate the configuration and operation of the virtualized hardware and software on those servers.</span></span> <span data-ttu-id="87d30-126">Y es esta orquestación la que hace que Azure sea tan eficaz: los usuarios ya no son responsables de mantener y actualizar el hardware, ya que de todo esto se encarga Azure en segundo plano.</span><span class="sxs-lookup"><span data-stu-id="87d30-126">And it is this orchestration that makes Azure so powerful - users are no longer responsible for maintaining and upgrading hardware, Azure does all this behind the scenes.</span></span> 

## <a name="next-steps"></a><span data-ttu-id="87d30-127">pasos siguientes</span><span class="sxs-lookup"><span data-stu-id="87d30-127">Next steps</span></span>

* <span data-ttu-id="87d30-128">Ahora que comprende el funcionamiento interno de Azure, el primer paso para adoptar Azure es [entender la identidad digital en Azure](tenant-explainer.md).</span><span class="sxs-lookup"><span data-stu-id="87d30-128">Now that you understand the internal functioning of Azure, the first step to adopting Azure is to [understand digital identity in Azure](tenant-explainer.md).</span></span> <span data-ttu-id="87d30-129">Ya está preparado para [crear el primer usuario en Azure AD][docs-add-users-to-aad].</span><span class="sxs-lookup"><span data-stu-id="87d30-129">You are then ready to [create your first user in Azure AD][docs-add-users-to-aad].</span></span>

<!-- Links -->

[cosmosdb]: /azure/cosmos-db/introduction
[docs-add-users-to-aad]: /azure/active-directory/add-users-azure-active-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
[vms]: /azure/virtual-machines/
[vnet]: /azure/virtual-network/virtual-networks-overview