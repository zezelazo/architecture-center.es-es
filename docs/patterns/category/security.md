---
title: Patrones de seguridad
description: La seguridad es la capacidad de un sistema para impedir acciones malintencionadas o involuntarias que se salgan del uso para el que fue diseñado, y para impedir la revelación o pérdida de información. Las aplicaciones en la nube están expuestas en Internet, más allá de los límites locales de confianza y, a menudo, están abiertas al público y pueden dar servicio a usuarios que no son de confianza. Se deben diseñar e implementar las aplicaciones de forma que se las proteja frente a ataques malintencionados, se restrinja el acceso solo a los usuarios autorizados y se proteja la información confidencial.
keywords: Patrón de diseño
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: 8437b8dfef751226580437a1b5678ca0e0e71f18
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2018
ms.locfileid: "30847005"
---
# <a name="security-patterns"></a><span data-ttu-id="7e807-106">Patrones de seguridad</span><span class="sxs-lookup"><span data-stu-id="7e807-106">Security patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="7e807-107">La seguridad es la capacidad de un sistema para impedir acciones malintencionadas o involuntarias que se salgan del uso para el que fue diseñado, y para impedir la revelación o pérdida de información.</span><span class="sxs-lookup"><span data-stu-id="7e807-107">Security is the capability of a system to prevent malicious or accidental actions outside of the designed usage, and to prevent disclosure or loss of information.</span></span> <span data-ttu-id="7e807-108">Las aplicaciones en la nube están expuestas en Internet, más allá de los límites locales de confianza y, a menudo, están abiertas al público y pueden dar servicio a usuarios que no son de confianza.</span><span class="sxs-lookup"><span data-stu-id="7e807-108">Cloud applications are exposed on the Internet outside trusted on-premises boundaries, are often open to the public, and may serve untrusted users.</span></span> <span data-ttu-id="7e807-109">Se deben diseñar e implementar las aplicaciones de forma que se las proteja frente a ataques malintencionados, se restrinja el acceso solo a los usuarios autorizados y se proteja la información confidencial.</span><span class="sxs-lookup"><span data-stu-id="7e807-109">Applications must be designed and deployed in a way that protects them from malicious attacks, restricts access to only approved users, and protects sensitive data.</span></span>


|                    <span data-ttu-id="7e807-110">Patrón</span><span class="sxs-lookup"><span data-stu-id="7e807-110">Pattern</span></span>                     |                                                                                                         <span data-ttu-id="7e807-111">Resumen</span><span class="sxs-lookup"><span data-stu-id="7e807-111">Summary</span></span>                                                                                                         |
|------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [<span data-ttu-id="7e807-112">Federated Identity</span><span class="sxs-lookup"><span data-stu-id="7e807-112">Federated Identity</span></span>](../federated-identity.md) |                                                                                <span data-ttu-id="7e807-113">La autenticación se delega a un proveedor de identidad externo.</span><span class="sxs-lookup"><span data-stu-id="7e807-113">Delegate authentication to an external identity provider.</span></span>                                                                                |
|         [<span data-ttu-id="7e807-114">Gatekeeper</span><span class="sxs-lookup"><span data-stu-id="7e807-114">Gatekeeper</span></span>](../gatekeeper.md)         | <span data-ttu-id="7e807-115">Protege aplicaciones y servicios mediante una instancia de host dedicada que actúa como agente entre los clientes y la aplicación o servicio, valida y sanea las solicitudes, y pasa las solicitudes y datos entre ellos.</span><span class="sxs-lookup"><span data-stu-id="7e807-115">Protect applications and services by using a dedicated host instance that acts as a broker between clients and the application or service, validates and sanitizes requests, and passes requests and data between them.</span></span> |
|          [<span data-ttu-id="7e807-116">Valet Key</span><span class="sxs-lookup"><span data-stu-id="7e807-116">Valet Key</span></span>](../valet-key.md)          |                                                        <span data-ttu-id="7e807-117">Usa un token o clave que proporciona a los clientes acceso directo restringido a un recurso o servicio específico.</span><span class="sxs-lookup"><span data-stu-id="7e807-117">Use a token or key that provides clients with restricted direct access to a specific resource or service.</span></span>                                                        |

