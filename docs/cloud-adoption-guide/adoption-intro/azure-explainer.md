---
title: 'Explicación: ¿Cómo funciona Azure?'
description: Explica el funcionamiento interno de Azure
author: petertay
ms.openlocfilehash: 1cebcc001b8d2ae93d8b0271c48d54617281c7c2
ms.sourcegitcommit: b3d74d8a89b2224fc796ce0e89cea447af43a0d4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2018
ms.locfileid: "35290516"
---
# <a name="explainer-how-does-azure-work"></a>Explicación: ¿Cómo funciona Azure?

Azure es la plataforma pública en la nube de Microsoft. Azure ofrece una amplia serie de servicios entre los que se incluyen la plataforma como servicio (PaaS), la infraestructura como servicio (IaaS), la base de datos como servicio (DBaaS) y muchos otros. Pero, ¿qué es exactamente Azure y cómo funciona?

Azure, al igual que otras plataformas en la nube, se basa en una tecnología conocida como **virtualización**. La mayor parte del hardware de equipo puede emularse en el software porque es simplemente un conjunto de instrucciones codificadas de forma permanente o semipermanente en silicio. Mediante una capa de emulación que asigna instrucciones de software a instrucciones de hardware, el hardware virtualizado puede ejecutarse en el software como si fuera el propio hardware.

Básicamente, la nube es un conjunto de servidores físicos en uno o varios centros de datos que ejecutan hardware virtualizado en nombre de los clientes. Entonces, ¿cómo crea, inicia, detiene y elimina la nube millones de instancias de hardware virtualizado para millones de clientes a la vez?

Para comprenderlo, vamos a echar un vistazo a la arquitectura del hardware del centro de datos.  En cada centro de datos hay una colección de servidores que se encuentran en bastidores de servidores. Cada bastidor de servidor contiene muchas **hojas** de servidor, así como un conmutador de red que proporciona conectividad de red y una unidad de distribución de energía (PDU) que suministra la alimentación. A veces, los bastidores se agrupan conjuntamente en unidades más grandes que se conocen como **clústeres**. 

En cada bastidor o clúster, la mayoría de los servidores están designados para ejecutar estas instancias de hardware virtualizado en nombre del usuario. Sin embargo, algunos servidores ejecutan un software de administración en la nube que se conoce como controlador de tejido. El **controlador de tejido** es una aplicación distribuida con muchas responsabilidades. Asigna servicios, supervisa el mantenimiento del servidor y los servicios que se ejecutan en él, y recupera los servidores cuando se produce un error.

Cada instancia del controlador de tejido se conecta a otro conjunto de servidores que ejecutan software de orquestación en la nube, conocido normalmente como **front-end**. El front-end hospeda los servicios web, las API RESTful y las bases de datos internas de Azure utilizadas para todas las funciones que realiza la nube. 

Por ejemplo, el front-end hospeda los servicios que controlan las solicitudes de cliente para asignar recursos de Azure como [redes virtuales][vnet], [máquinas virtuales][vms] y servicios como [Cosmos DB][cosmosdb]. En un primer tiempo, el front-end valida el usuario y comprueba que esté autorizado para asignar los recursos solicitados. Si es así, el front-end consulta una base de datos para buscar un bastidor de servidor con una capacidad suficiente y, luego, indica al controlador de tejido del bastidor que asigne el recurso.

Por lo tanto, y para simplificar, Azure es una inmensa colección de servidores y de hardware de red, junto con un complejo conjunto de aplicaciones distribuidas que orquesta la configuración y el funcionamiento del hardware virtualizado y del software de estos servidores. Y es esta orquestación la que hace que Azure sea tan eficaz: los usuarios ya no son responsables de mantener y actualizar el hardware, ya que de todo esto se encarga Azure en segundo plano. 

## <a name="next-steps"></a>Pasos siguientes

* Ahora que comprende el funcionamiento interno de Azure, obtenga información acerca del [gobierno del acceso a los recursos](governance-explainer.md). Después, vaya al primer paso de la adopción de Azure, que consiste en [comprender el concepto de identidad digital en Azure](tenant-explainer.md). Después de completar paso, está listo para [crear su primer usuario en Azure AD][docs-add-users-to-aad].

<!-- Links -->

[cosmosdb]: /azure/cosmos-db/introduction
[docs-add-users-to-aad]: /azure/active-directory/add-users-azure-active-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
[vms]: /azure/virtual-machines/
[vnet]: /azure/virtual-network/virtual-networks-overview
