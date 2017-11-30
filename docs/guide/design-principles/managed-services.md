---
title: Uso de servicios administrados
description: Cuando sea posible, use la plataforma como servicio (PaaS) en lugar de la infraestructura como servicio (IaaS).
author: MikeWasson
layout: LandingPage
ms.openlocfilehash: 7156c073db3e047fb38e031309ddb637a9e44c02
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2017
---
# <a name="use-managed-services"></a>Uso de servicios administrados

## <a name="when-possible-use-platform-as-a-service-paas-rather-than-infrastructure-as-a-service-iaas"></a>Cuando sea posible, use la plataforma como servicio (PaaS) en lugar de la infraestructura como servicio (IaaS).

IaaS es como tener una caja de piezas. Puede crear cualquier cosa pero lo debe ensamblar usted mismo. Los servicios administrados son más fáciles de configurar y administrar. No es necesario aprovisionar máquinas virtuales, configurar redes virtuales, administrar revisiones y actualizaciones, y toda la sobrecarga asociada a la ejecución de software en una máquina virtual.

Por ejemplo, suponga que la aplicación necesita una cola de mensajes. Puede configurar su propio servicio de mensajería en una máquina virtual, con algo parecido a RabbitMQ. Pero Azure Service Bus ya proporciona una mensajería de confianza como servicio y es más fácil de configurar. Basta con crear un espacio de nombres de Service Bus (que puede realizarse como parte de un script de implementación) y, después, llamar a Service Bus con el SDK del cliente. 

Por supuesto, la aplicación puede tener requisitos específicos que hacen que un enfoque de IaaS sea más adecuado. Sin embargo, incluso si la aplicación se basa en IaaS, busque lugares en los que sea natural incorporar servicios administrados. Esto incluye cachés, colas y almacenamiento de datos.

| En lugar de ejecutar... | Considere la posibilidad de utilizar... |
|-----------------------|-------------|
| Active Directory | Azure Active Directory Domain Services |
| Elasticsearch | Azure Search |
| Hadoop | HDInsight |
| IIS | App Service |
| MongoDB | Cosmos DB |
| Redis | Azure Redis Cache |
| SQL Server | Azure SQL Database |


