---
title: Criterios para elegir una opción de proceso de Azure
description: Compare los servicios de proceso de Azure a través de varios ejes.
author: MikeWasson
layout: LandingPage
ms.openlocfilehash: 36b57d1fb674b5a1452a0e8208de836963b2b01b
ms.sourcegitcommit: c53adf50d3a787956fc4ebc951b163a10eeb5d20
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/23/2017
---
# <a name="criteria-for-choosing-an-azure-compute-option"></a>Criterios para elegir una opción de proceso de Azure

El término *proceso* hace referencia al modelo de hospedaje para los recursos informáticos en los que las aplicaciones se ejecutan. En las tablas siguientes se comparan los servicios de proceso de Azure a través de varios ejes. Consulte estas tablas al seleccionar una opción de proceso para la aplicación.

## <a name="hosting-model"></a>Modelo de alojamiento

| Criterios | Máquinas virtuales | App Service | Service Fabric | Azure Functions | Azure Container Service | Cloud Services | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Composición de la aplicación | Independiente | Aplicaciones | Ejecutables de invitado, servicios, contenedores | Functions | Contenedores | Roles | Scheduled jobs  |
| Densidad | Independiente | Varias aplicaciones por instancia a través de los planes de aplicación | Varios servicios por VM | Sin instancias dedicadas <a href="#note1"><sup>1</sup></a> | Varios contenedores por VM | Una instancia de rol por VM | Varias aplicaciones por VM |
| Número mínimo de nodos | 1 <a href="#note2"><sup>2</sup></a>  | 1 | 5 <a href="#note3"><sup>3</sup></a> | Sin nodos dedicados <a href="#note1"><sup>1</sup></a> | 3 | 2 | 1 <a href="#note4"><sup>4</sup></a> |
| Administración de estados | Con o sin estado | Sin estado | Con o sin estado | Sin estado | Con o sin estado | Sin estado | Sin estado |
| Hospedaje web | Independiente | Integrado | Independiente | No aplicable | Independiente | Integrado (IIS) | No |
| SO | Windows, Linux | Windows, Linux  | Windows, Linux | No aplicable | Windows (versión preliminar), Linux | Windows | Windows, Linux |
| ¿Se puede implementar en una red virtual dedicada? | Compatible | Compatible <a href="#note5"><sup>5</sup></a> | Compatible | No compatible | Compatible | Compatible <a href="#note6"><sup>6</sup></a> | Compatible |
| Conectividad híbrida | Compatible | Compatible <a href="#note1"><sup>7</sup></a>  | Compatible | No compatible | Compatible | Compatible <a href="#note8"><sup>8</sup></a> | Compatible |

Notas

1. <span id="note1">Si usa un plan de consumo. Si usa un plan de App Service, las funciones se ejecutan en las VM asignadas al plan de App Service. Consulte [Elija el plan de servicio correcta para Azure Functions][function-plans].</a>
2. <span id="note2">Acuerdo de Nivel de Servicio superior con dos o más instancias</a>.
3. </a>Para entornos de producción.<span id="note3">
4. <span id="note4">Se puede reducir verticalmente hasta cero una vez completado el trabajo</a>.
5. <span id="note5">Requiere App Service Environment.</a>
6. <span id="note6">Solo red virtual clásica.</a>
7. <span id="note7">Requiere App Service Environment o conexiones híbridas de BizTalk</a>
8. <span id="note8">Red virtual clásica o red virtual de Resource Manager a través del emparejamiento de redes virtuales</a>

## <a name="devops"></a>DevOps

| Criterios | Máquinas virtuales | App Service | Service Fabric | Azure Functions | Azure Container Service | Cloud Services | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Depuración local | Independiente | IIS Express, otros <a href="#note1b"><sup>1</sup></a> | Clúster de nodo local | CLI de Azure Functions | Tiempo de ejecución de contenedor local | Emulador local | No compatible |
| Modelo de programación | Independiente | Aplicación web, WebJobs para tareas en segundo plano | Invitado ejecutable, modelo de servicio, modelo de actor, contenedores | Functions con desencadenadores | Independiente | Rol web, rol de trabajo | Aplicación de línea de comandos |
| Resource Manager | Compatible | Compatible | Compatible | Compatible | Compatible | Limitado <a href="#note2b"><sup>2</sup></a> | Compatible |  
| Actualización de aplicaciones | Sin compatibilidad integrada | Ranuras de implementación | Actualización (por servicio) gradual | Sin compatibilidad integrada | Depende del orquestador. La mayoría admiten las actualizaciones graduales | Intercambio de VIP o actualización gradual | No aplicable |

Notas

1. <span id="note1b">Las opciones incluyen IIS Express para ASP.NET o node.js (iisnode); servidor web PHP; kit de herramientas de Azure para IntelliJ, kit de herramientas de Azure para Eclipse. App Service también admite la depuración remota de una aplicación web implementada.</a>
2. <span id="note2b">Vea [Proveedores, regiones, versiones de API y esquemas de Resource Manager][resource-manager-supported-services]. 


## <a name="scalability"></a>Escalabilidad

| Criterios | Máquinas virtuales | App Service | Service Fabric | Azure Functions | Azure Container Service | Cloud Services | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Escalado automático | Conjuntos de escalado de máquinas virtuales | Servicio integrado | Conjuntos de escalas de máquina virtual | Servicio integrado | No compatible | Servicio integrado | N/D |
| Equilibrador de carga | Azure Load Balancer | Integrado | Azure Load Balancer | Integrado | Azure Load Balancer | Integrado | Azure Load Balancer |
| Límite de escala | Imagen de la plataforma: 1000 nodos por VMSS, Imagen personalizada: 100 nodos por VMSS | 20 instancias, 50 con App Service Environment | 100 nodos por VMSS | Infinito <a href="#note1c"><sup>1</sup></a> | 100 | Sin límite definido, 200 máximo recomendado | Límite de 20 núcleos predeterminado. Póngase en contacto con el servicio de atención al cliente para solicitar un aumento. |

Notas

1. <span id="note1c">Si usa un plan de consumo. Si usa un plan de App Service, se aplican los límites de escalado de App Service. Consulte [Elija el plan de servicio correcta para Azure Functions][function-plans].</a>

## <a name="availability"></a>Disponibilidad

| Criterios | Máquinas virtuales | App Service | Service Fabric | Azure Functions | Azure Container Service | Cloud Services | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Contrato de nivel de servicio | [Acuerdo de Nivel de Servicio para Virtual Machines][sla-vm] | [Acuerdo de Nivel de Servicio para App Service][sla-app-service] | [Acuerdo de Nivel de Servicio para Service Fabric][sla-sf] | [Acuerdo de Nivel de Servicio para Functions][sla-functions] | [Acuerdo de Nivel de Servicio para Azure Container Service][sla-acs] | [Acuerdo de Nivel de Servicio para Cloud Services][sla-cloud-service] | [Acuerdo de Nivel de Servicio para Azure Batch][sla-batch] |
| Conmutación por error de múltiples regiones | Administrador de tráfico | Administrador de tráfico | Traffic Manager, clúster de varias regiones | No compatible  | Administrador de tráfico | Administrador de tráfico | No compatible |

## <a name="security"></a>Seguridad

| Criterios | Máquinas virtuales | App Service | Service Fabric | Azure Functions | Azure Container Service | Cloud Services | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| SSL | Configurado en VM | Compatible | Compatible  | Compatible | Configurado en VM | Compatible | Compatible |
| RBAC | Compatible | Compatible | Compatible | Compatible | Compatible | No compatible | Compatible |

## <a name="other"></a>Otros

| Criterios | Máquinas virtuales | App Service | Service Fabric | Azure Functions | Azure Container Service | Cloud Services | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Coste | [Windows][cost-windows-vm], [Linux][cost-linux-vm] | [Precios de App Service][cost-app-service] | [Precios de Service Fabric][cost-service-fabric] | [Precios de Azure Functions][cost-functions] | [Precios de Azure Container Service][cost-acs] | [Precios de Cloud Services][cost-cloud-services] | [Precios de Azure Batch][cost-batch]
| Estilos de arquitectura idóneos | Big compute (HPC) de n niveles | Web-Cola-Trabajo | Microservicios, arquitectura orientada a eventos (EDA) | Microservicios, EDA | Microservicios, EDA | Web-Cola-Trabajo | Big Compute |

[cost-linux-vm]: https://azure.microsoft.com/pricing/details/virtual-machines/linux/
[cost-windows-vm]: https://azure.microsoft.com/pricing/details/virtual-machines/windows/
[cost-app-service]: https://azure.microsoft.com/pricing/details/app-service/
[cost-service-fabric]: https://azure.microsoft.com/pricing/details/service-fabric/
[cost-functions]: https://azure.microsoft.com/pricing/details/functions/
[cost-acs]: https://azure.microsoft.com/pricing/details/container-service/
[cost-cloud-services]: https://azure.microsoft.com/pricing/details/cloud-services/
[cost-batch]: https://azure.microsoft.com/pricing/details/batch/

[function-plans]: /azure/azure-functions/functions-scale
[sla-acs]: https://azure.microsoft.com/support/legal/sla/container-service/
[sla-app-service]: https://azure.microsoft.com/support/legal/sla/app-service/
[sla-batch]: https://azure.microsoft.com/support/legal/sla/batch/
[sla-cloud-service]: https://azure.microsoft.com/support/legal/sla/cloud-services/
[sla-functions]: https://azure.microsoft.com/support/legal/sla/functions/
[sla-sf]: https://azure.microsoft.com/support/legal/sla/service-fabric/
[sla-vm]: https://azure.microsoft.com/support/legal/sla/virtual-machines/

[resource-manager-supported-services]: /azure/azure-resource-manager/resource-manager-supported-services