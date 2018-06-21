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
# <a name="security-patterns"></a>Patrones de seguridad

[!INCLUDE [header](../../_includes/header.md)]

La seguridad es la capacidad de un sistema para impedir acciones malintencionadas o involuntarias que se salgan del uso para el que fue diseñado, y para impedir la revelación o pérdida de información. Las aplicaciones en la nube están expuestas en Internet, más allá de los límites locales de confianza y, a menudo, están abiertas al público y pueden dar servicio a usuarios que no son de confianza. Se deben diseñar e implementar las aplicaciones de forma que se las proteja frente a ataques malintencionados, se restrinja el acceso solo a los usuarios autorizados y se proteja la información confidencial.


|                    Patrón                     |                                                                                                         Resumen                                                                                                         |
|------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [Federated Identity](../federated-identity.md) |                                                                                La autenticación se delega a un proveedor de identidad externo.                                                                                |
|         [Gatekeeper](../gatekeeper.md)         | Protege aplicaciones y servicios mediante una instancia de host dedicada que actúa como agente entre los clientes y la aplicación o servicio, valida y sanea las solicitudes, y pasa las solicitudes y datos entre ellos. |
|          [Valet Key](../valet-key.md)          |                                                        Usa un token o clave que proporciona a los clientes acceso directo restringido a un recurso o servicio específico.                                                        |

