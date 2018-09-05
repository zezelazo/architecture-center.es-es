---
title: Criterios para elegir un servicio de proceso de Azure
description: Compare los servicios de proceso de Azure a través de varios ejes
author: MikeWasson
ms.date: 06/13/2018
ms.openlocfilehash: b7a5b08e1d9a9eba33003b3d478a61388a496272
ms.sourcegitcommit: c4106b58ad08f490e170e461009a4693578294ea
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2018
ms.locfileid: "43016082"
---
# <a name="criteria-for-choosing-an-azure-compute-service"></a>Criterios para elegir un servicio de proceso de Azure

El término *proceso* hace referencia al modelo de hospedaje para los recursos informáticos en los que las aplicaciones se ejecutan. En las tablas siguientes se comparan los servicios de proceso de Azure a través de varios ejes. Consulte estas tablas al seleccionar una opción de proceso para la aplicación.

## <a name="hosting-model"></a>Modelo de alojamiento

| Criterios | Virtual Machines | App Service | Service Fabric | Azure Functions | Azure Kubernetes Service | Azure Container Instances | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Composición de la aplicación | Independiente | Aplicaciones, contenedores | Ejecutables de invitado, servicios, contenedores | Functions | Contenedores | Contenedores | Scheduled jobs  |
| Densidad | Independiente | Varias aplicaciones por instancia a través de planes de App Service | Varios servicios por VM | Sin servidor <a href="#note1"><sup>1</sup></a> | Varios contenedores por nodo |Sin instancias dedicadas | Varias aplicaciones por VM |
| Número mínimo de nodos | 1 <a href="#note2"><sup>2</sup></a>  | 1 | 5 <a href="#note3"><sup>3</sup></a> | Sin servidor <a href="#note1"><sup>1</sup></a> | 3 <a href="#note3"><sup>3</sup></a> | Sin nodos dedicados | 1 <a href="#note4"><sup>4</sup></a> |
| Administración de estados | Con o sin estado | Sin estado | Con o sin estado | Sin estado | Con o sin estado | Sin estado | Sin estado |
| Hospedaje web | Independiente | Integrado | Independiente | No aplicable | Independiente | Independiente | Sin  |
| ¿Se puede implementar en una red virtual dedicada? | Compatible | Compatible<a href="#note5"><sup>5</sup></a> | Compatible | Compatible <a href="#note5"><sup>5</sup></a> | [Compatible](/azure/aks/networking-overview) | No compatible | Compatible |
| Conectividad híbrida | Compatible | Compatible <a href="#note6"><sup>6</sup></a>  | Compatible | Compatible <a href="#node7"><sup>7</sup></a> | Compatible | No compatible | Compatible |

Notas

1. <span id="note1">Si usa un plan de consumo. Si usa un plan de App Service, las funciones se ejecutan en las VM asignadas al plan de App Service. Consulte [Elija el plan de servicio correcta para Azure Functions][function-plans].</span>
2. <span id="note2">Acuerdo de Nivel de Servicio superior con dos o más instancias</span>.
3. <span id="note3">Se recomienda para entornos de producción.</span>
4. <span id="note4">Se puede reducir verticalmente hasta cero una vez completado el trabajo</span>.
5. <span id="note5">Requiere App Service Environment.</span>
6. <span id="note6">Use [Hybrid Connections de Azure App Service][app-service-hybrid].</span>
7. <span id="note7">Requiere un plan de App Service.</span>

## <a name="devops"></a>DevOps

| Criterios | Virtual Machines | App Service | Service Fabric | Azure Functions | Azure Kubernetes Service | Azure Container Instances | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Depuración local | Independiente | IIS Express, otros <a href="#note1b"><sup>1</sup></a> | Clúster de nodo local | Visual Studio o CLI de Azure Functions | Minikube, otros | Tiempo de ejecución de contenedor local | No compatible |
| Modelo de programación | Independiente | Aplicaciones web y API, WebJobs para tareas en segundo plano | Invitado ejecutable, modelo de servicio, modelo de actor, contenedores | Functions con desencadenadores | Independiente | Independiente | Aplicación de línea de comandos |
| Actualización de aplicaciones | Sin compatibilidad integrada | Ranuras de implementación | Actualización (por servicio) gradual | Ranuras de implementación | Actualización gradual | No aplicable |

Notas

1. <span id="note1b">Las opciones incluyen IIS Express para ASP.NET o node.js (iisnode); servidor web PHP; kit de herramientas de Azure para IntelliJ, kit de herramientas de Azure para Eclipse. App Service también admite la depuración remota de una aplicación web implementada.</span>
2. <span id="note2b">Vea [Proveedores, regiones, versiones de API y esquemas de Resource Manager][resource-manager-supported-services].</span> 


## <a name="scalability"></a>Escalabilidad

| Criterios | Virtual Machines | App Service | Service Fabric | Azure Functions | Azure Kubernetes Service | Azure Container Instances | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Escalado automático | VM Scale Sets | Servicio integrado | VM Scale Sets | Servicio integrado | No compatible | No compatible | N/D |
| Equilibrador de carga | Azure Load Balancer | Integrado | Azure Load Balancer | Integrado | Integrado |  Sin compatibilidad integrada | Azure Load Balancer |
| Límite de escalado<a href="#note1c"><sup>1</sup></a> | Imagen de la plataforma: 1000 nodos por VMSS, Imagen personalizada: 100 nodos por VMSS | 20 instancias, 100 con App Service Environment | 100 nodos por VMSS | 200 instancias por aplicación Function | 100 nodos por clúster (límite predeterminado) |20 grupos de contenedores por suscripción (límite predeterminado). | 20 núcleos (límite predeterminado). |

Notas

1. <span id="note1c">Consulte [Límites, cuotas y restricciones de suscripción y servicios de Azure](/azure/azure-subscription-service-limits)</span>.

## <a name="availability"></a>Disponibilidad

| Criterios | Virtual Machines | App Service | Service Fabric | Azure Functions | Azure Kubernetes Service | Azure Container Instances | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Contrato de nivel de servicio | [Acuerdo de Nivel de Servicio para Virtual Machines][sla-vm] | [Acuerdo de Nivel de Servicio para App Service][sla-app-service] | [Acuerdo de Nivel de Servicio para Service Fabric][sla-sf] | [Acuerdo de Nivel de Servicio para Functions][sla-functions] | [SLA para AKS][sla-acs] | [Acuerdo de nivel de servicio para Container Instances](https://azure.microsoft.com/support/legal/sla/container-instances/) | [Acuerdo de Nivel de Servicio para Azure Batch][sla-batch] |
| Conmutación por error de múltiples regiones | Traffic Manager | Traffic Manager | Traffic Manager, clúster de varias regiones | No compatible  | Traffic Manager | No compatible | No compatible |

## <a name="other"></a>Otros

| Criterios | Virtual Machines | App Service | Service Fabric | Azure Functions | Azure Kubernetes Service | Azure Container Instances | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| SSL | Configurado en VM | Compatible | Compatible  | Compatible | [Controlador de entrada](/azure/aks/ingress) | Uso de contenedores [sidecar](../../patterns/sidecar.md) | Compatible |
| Coste | [Windows][cost-windows-vm], [Linux][cost-linux-vm] | [Precios de App Service][cost-app-service] | [Precios de Service Fabric][cost-service-fabric] | [Precios de Azure Functions][cost-functions] | [Precios de AKS][cost-acs] | [Precios de Container Instances](https://azure.microsoft.com/pricing/details/container-instances/) | [Precios de Azure Batch][cost-batch]
| Estilos de arquitectura idóneos | [N-Tier][n-tier], [Big Compute][big-compute] (HPC) | [Web-Cola-Trabajo][w-q-w], [N niveles][n-tier] | [Microservicios][microservices], [arquitectura orientada a eventos][event-driven] | [Microservicios][microservices], [arquitectura orientada a eventos][event-driven] | [Microservicios][microservices], [arquitectura orientada a eventos][event-driven] | [Microservicios][microservices], automatización de tareas, trabajos por lotes  | [Big Compute][big-compute] (HPC) |

[cost-linux-vm]: https://azure.microsoft.com/pricing/details/virtual-machines/linux/
[cost-windows-vm]: https://azure.microsoft.com/pricing/details/virtual-machines/windows/
[cost-app-service]: https://azure.microsoft.com/pricing/details/app-service/
[cost-service-fabric]: https://azure.microsoft.com/pricing/details/service-fabric/
[cost-functions]: https://azure.microsoft.com/pricing/details/functions/
[cost-acs]: https://azure.microsoft.com/pricing/details/kubernetes-service/
[cost-batch]: https://azure.microsoft.com/pricing/details/batch/

[function-plans]: /azure/azure-functions/functions-scale
[sla-acs]: https://azure.microsoft.com/support/legal/sla/kubernetes-service
[sla-app-service]: https://azure.microsoft.com/support/legal/sla/app-service/
[sla-batch]: https://azure.microsoft.com/support/legal/sla/batch/
[sla-functions]: https://azure.microsoft.com/support/legal/sla/functions/
[sla-sf]: https://azure.microsoft.com/support/legal/sla/service-fabric/
[sla-vm]: https://azure.microsoft.com/support/legal/sla/virtual-machines/

[resource-manager-supported-services]: /azure/azure-resource-manager/resource-manager-supported-services
[scale-acs]: /azure/container-service/kubernetes/container-service-scale#scaling-considerations

[n-tier]: ../architecture-styles/n-tier.md
[w-q-w]: ../architecture-styles/web-queue-worker.md
[microservices]: ../architecture-styles/microservices.md
[event-driven]: ../architecture-styles/event-driven.md
[big-date]: ../architecture-styles/big-data.md
[big-compute]: ../architecture-styles/big-compute.md

[app-service-hybrid]: /azure/app-service/app-service-hybrid-connections
