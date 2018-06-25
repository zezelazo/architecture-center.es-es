---
title: Introducción a las opciones de proceso de Azure
description: Introducción a las opciones de proceso de Azure
author: MikeWasson
ms.date: 06/13/2018
ms.openlocfilehash: ceb70f8eeff42e6cadb8a63c2f36986f26322201
ms.sourcegitcommit: 26b04f138a860979aea5d253ba7fecffc654841e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/19/2018
ms.locfileid: "36206746"
---
# <a name="overview-of-azure-compute-options"></a>Introducción a las opciones de proceso de Azure

El término *proceso* hace referencia al modelo de hospedaje para los recursos informáticos donde se ejecutan las aplicaciones. 

## <a name="overview"></a>Información general

En un extremo del espectro se encuentra la **infraestructura como servicio** (IaaS). Con IaaS puede aprovisionar las máquinas virtuales que necesite, junto con la red y los componentes de almacenamiento asociados. A continuación, puede implementar el software y las aplicaciones que desee en esas máquinas virtuales. Este modelo es lo más parecido a un entorno local tradicional, salvo que Microsoft administra la infraestructura. Aunque el usuario administra las máquinas virtuales.  

La **plataforma como servicio** (PaaS) proporciona un entorno de hospedaje administrado, donde puede implementar la aplicación sin necesidad de administrar las máquinas virtuales o los recursos de red. Por ejemplo, en lugar de crear máquinas virtuales individuales, se crea un recuento de instancias y el servicio aprovisiona, configura y administra los recursos necesarios. Azure App Service es un ejemplo de un servicio de PaaS.

Existen varios servicios desde IaaS hasta el PaaS más puro. Por ejemplo, las máquinas virtuales de Azure se pueden escalar automáticamente con VM Scale Sets. Esta funcionalidad de escalado automático no es estrictamente PaaS, pero es el tipo de característica de administración que se encuentra en un servicio de PaaS.

Las **funciones como servicio** (FaaS) van más allá al eliminar la necesidad de preocuparse por el entorno de hospedaje. En lugar de crear instancias de proceso y de implementar código en las instancias, solo tiene que implementar el código y el servicio lo ejecuta automáticamente. No es necesario administrar los recursos de proceso. Estos servicios usan arquitectura sin servidor y se escalan o reducen verticalmente hasta cualquier nivel necesario para controlar el tráfico. Azure Functions es un servicio de FaaS.

IaaS ofrece más control, flexibilidad y portabilidad. FaaS ofrece simplicidad, escalado elástico y quizá hasta ahorro, ya que solo se paga por el tiempo de ejecución del código. PaaS es algo intermedio. En general, cuanta más flexibilidad proporciona un servicio, mayor es la responsabilidad del usuario en cuanto a la configuración y la administración de los recursos. Los servicios de FaaS administran automáticamente casi todos los aspectos de la ejecución de una aplicación, mientras que las soluciones IaaS requieren el aprovisionamiento, la configuración y la administración de las máquinas virtuales y los componentes de red que se creen.

## <a name="azure-compute-options"></a>Opciones de proceso de Azure

Estas son las opciones de proceso principales actualmente disponibles en Azure:

- [Virtual Machines](/azure/virtual-machines/) es un servicio de IaaS que permite implementar y administrar las máquinas virtuales de una red virtual.
- [App Service](/azure/app-service/app-service-value-prop-what-is) es una oferta de PaaS administrada para hospedar aplicaciones web, back-end de aplicaciones móviles, API de RESTful o procesos empresariales automatizados.
- [Service Fabric](/azure/service-fabric/service-fabric-overview) es una plataforma de sistemas distribuidos que se puede ejecutar en muchos entornos, tanto localmente como en Azure, por ejemplo. Service Fabric es un orquestador de microservicios en un clúster de máquinas. 
- [Azure Container Service](/azure/container-service/container-service-intro) permite crear, configurar y administrar un clúster de máquinas virtuales preconfiguradas para ejecutar aplicaciones en contenedor.
- [Azure Container Instances](/azure/container-instances/container-instances-overview) ofrece la forma más rápida y sencilla de ejecutar un contenedor en Azure, sin tener que aprovisionar ninguna máquina virtual y sin necesidad de adoptar un servicio de nivel superior.
- [Azure Functions](/azure/azure-functions/functions-overview) es un servicio administrado de FaaS.
- [Azure Batch](/azure/batch/batch-technical-overview) es un servicio administrado para ejecutar aplicaciones a gran escala en paralelo y de informática de alto rendimiento (HPC).
- [Cloud Services](/azure/cloud-services/cloud-services-choose-me) es un servicio administrado para ejecutar aplicaciones en la nube. Utiliza un modelo de hospedaje PaaS. 

Al seleccionar una opción de proceso, estos son algunos de los factores que tener en cuenta:

- El modelo de hospedaje. ¿Cómo se hospeda el servicio? ¿Qué requisitos y limitaciones impone este entorno de hospedaje? 
- DevOps. ¿Hay compatibilidad integrada para las actualizaciones de las aplicaciones? ¿Cuál es el modelo de implementación?
- La escalabilidad. ¿Cómo administra el servicio la incorporación y la eliminación de instancias? ¿Puede realizar el escalado automático en función de la carga y otras métricas? 
- La disponibilidad. ¿Qué es el Acuerdo de Nivel de Servicio? 
- El costo. Además del costo del propio servicio, tenga en cuenta el de las operaciones de administración de una solución basada en ese servicio. Por ejemplo, las operaciones de las soluciones IaaS podrían tener mayor costo.
- ¿Cuáles son las limitaciones generales de cada servicio? 
- ¿Qué variantes de arquitecturas de aplicación son adecuados para este servicio? 

## <a name="next-steps"></a>Pasos siguientes

Para seleccionar un servicio de proceso para la aplicación, utilice el [Árbol de decisión para los servicios de proceso de Azure](./compute-decision-tree.md)

Para una comparación más detallada de las opciones de proceso de Azure, consulte [Criterios para elegir un servicio de proceso de Azure](./compute-comparison.md).
