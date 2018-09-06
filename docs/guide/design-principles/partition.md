---
title: Partición alrededor de límites
description: Uso de particiones para evitar los limites en bases de datos, redes y procesos.
author: MikeWasson
ms.date: 08/30/2018
ms.openlocfilehash: 2f6bf797c2c7e5af7c487635c19eaf77eee77dec
ms.sourcegitcommit: ae8a1de6f4af7a89a66a8339879843d945201f85
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/31/2018
ms.locfileid: "43326303"
---
# <a name="partition-around-limits"></a>Partición alrededor de límites

## <a name="use-partitioning-to-work-around-database-network-and-compute-limits"></a>Uso de particiones para evitar los limites en bases de datos, redes y procesos.

En la nube, todos los servicios tienen límites en cuanto a la posibilidad de escalarse verticalmente. Los límites de servicio de Azure se documentan en [Límites, cuotas y restricciones de suscripción y servicios de Azure][azure-limits]. Los límites incluyen el número de núcleos, el tamaño de base de datos, el rendimiento de consultas y el rendimiento de la red. Si el sistema crece lo suficiente, puede alcanzar uno o varios de estos límites. Use particiones para evitar estos límites.

Existen muchas maneras de crear particiones de un sistema, como:

- Crear particiones de una base de datos para evitar límites en el tamaño de la base de datos, la E/S de datos o el número de sesiones simultáneas.

- Crear particiones de una cola o un bus de mensajes para evitar límites en el número de solicitudes o el número de conexiones simultáneas.

- Crear particiones de una aplicación web de App Service para evitar límites en el número de instancias por plan de App Service. 

Se pueden crear particiones en una base de datos *horizontalmente*, *verticalmente* o *funcionalmente*.

- En la creación de particiones horizontal, también denominada particionamiento, cada partición contiene datos de un subconjunto del conjunto de datos total. Las particiones comparten el mismo esquema de datos. Por ejemplo, los clientes cuyos nombres comiencen por A&ndash;M entran en una partición, los que comiencen por N&ndash;Z en otra, etc.

- En el particionamiento vertical, cada partición contiene un subconjunto de los campos de los elementos del almacén de datos. Por ejemplo, colocar los campos a los que se accede con frecuencia en una partición y a los que se accede raramente en otra.

- En el particionamiento funcional, los datos se particionan según cómo se usan en cada contexto limitado en el sistema. Por ejemplo, almacenar los datos de facturas en una partición y los de inventario de productos en otra. Los esquemas son independientes.

Para obtener instrucciones más detalladas, consulte [Data partitioning][data-partitioning-guidance] (Creación de particiones de los datos).

## <a name="recommendations"></a>Recomendaciones

**Crear particiones de distintas partes de la aplicación**. Las bases de datos son candidatas obvias a la creación de particiones, pero otras son el almacenamiento, la caché, las colas y las instancias de proceso.

**Diseñar la clave de partición para evitar zonas activas**. Si se crean particiones de una base de datos, pero una partición todavía obtiene la mayoría de las solicitudes, el problema sigue ahí. Lo mejor es que la carga se distribuya de manera uniforme entre todas las particiones. Por ejemplo, código hash por id. de cliente y no la primera letra del nombre del cliente, ya que algunas letras son más frecuentes. El mismo principio se aplica al crear particiones de una cola de mensajes. Elija una clave de partición que lleve a una distribución uniforme de los mensajes en el conjunto de colas. Para más información, consulte [Particionamiento][sharding].

**Partición alrededor de los límites de suscripción y servicio de Azure**. Los servicios y componentes individuales tienen límites, pero también los hay para las suscripciones y los grupos de recursos. En el caso de aplicaciones muy grandes, puede que deba crear particiones en torno a esos límites.  

**Partición en niveles diferentes**. Considere un servidor de base de datos implementado en una máquina virtual. La máquina virtual tiene un disco duro virtual que está respaldado por Azure Storage. La cuenta de almacenamiento pertenece a una suscripción de Azure. Tenga en cuenta que cada paso de la jerarquía tiene límites. El servidor de base de datos puede tener un límite de grupo de conexiones. Las máquinas virtuales tienen límites de CPU y red. El almacenamiento tiene límites de IOPS. La suscripción tiene límites en el número de núcleos de la máquina virtual. Por lo general, es más fácil crear particiones en los niveles inferiores de la jerarquía. Solo las aplicaciones de gran tamaño deben crear particiones en el nivel de suscripción. 

<!-- links -->

[azure-limits]: /azure/azure-subscription-service-limits
[data-partitioning-guidance]: ../../best-practices/data-partitioning.md
[sharding]: ../../patterns/sharding.md

 