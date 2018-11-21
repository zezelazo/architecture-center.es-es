---
title: Canalización de CI/CD para cargas de trabajo basadas en contenedores
description: Cree una canalización de DevOps para una aplicación web de Node.js con Jenkins, Azure Container Registry, Azure Kubernetes Service, Cosmos DB y Grafana.
author: iainfoulds
ms.date: 07/05/2018
ms.openlocfilehash: db8de674ee2789c5b41cebebee5745ecc8544122
ms.sourcegitcommit: 0a31fad9b68d54e2858314ca5fe6cba6c6b95ae4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/13/2018
ms.locfileid: "51610845"
---
# <a name="cicd-pipeline-for-container-based-workloads"></a>Canalización de CI/CD para cargas de trabajo basadas en contenedores

Este escenario de ejemplo es aplicable a empresas que desean modernizar el desarrollo de aplicaciones mediante el uso de contenedores y flujos de trabajo de DevOps. En este escenario, Jenkins genera e implementa una aplicación web de Node.js en Azure Container Registry y Azure Kubernetes Service. Para agregar una capa de base de datos distribuida globalmente, se usa Azure Cosmos DB. Para supervisar y solucionar problemas de rendimiento de la aplicación, Azure Monitor se integra con una instancia y un panel de Grafana.

Algunos ejemplos de escenarios de aplicación son la provisión de un entorno de desarrollo automatizado, la validación de nuevas confirmaciones de código y la inserción de nuevas implementaciones en entornos de ensayo o producción. Tradicionalmente, las empresas tenían que crear y compilar las aplicaciones y actualizaciones manualmente, y mantener una base de código grande y rígida. Un enfoque moderno del desarrollo de aplicaciones que usa integración continua (CI) e implementación continua (CD) permite crear, probar e implementar servicios más rápidamente. Este enfoque moderno permite lanzar aplicaciones y actualizaciones a los clientes más rápidamente y responder a las cambiantes demandas del negocio de manera más ágil.

Con servicios de Azure como Azure Kubernetes Service, Container Registry y Cosmos DB, las empresas pueden usar las últimas novedades en técnicas y herramientas de desarrollo de aplicaciones para simplificar el proceso de una implementación de alta disponibilidad.

## <a name="relevant-use-cases"></a>Casos de uso pertinentes

Otros casos de uso pertinentes incluyen:

* Modernización de procedimientos de desarrollo de aplicaciones a un modelo de microservicios basado en contenedores.
* Aceleración del desarrollo de aplicaciones y los ciclos de vida de implementación.
* Automatización de las implementaciones para la validación de entornos de prueba o aceptación.

## <a name="architecture"></a>Arquitectura

![Introducción a la arquitectura de los componentes de Azure implicados en un escenario de DevOps con Jenkins, Azure Container Registry y Azure Kubernetes Service][architecture]

Este escenario incluye una canalización de DevOps para una aplicación web de Node.js y un servidor back-end de base de datos. Los datos fluyen por el escenario de la siguiente manera:

1. Un programador hace cambios en el código fuente de la aplicación web de Node.js.
2. El cambio de código se confirma en un repositorio de control de código fuente, por ejemplo, GitHub.
3. Para iniciar el proceso de integración continua (CI), un webhook de GitHub desencadena una compilación del proyecto de Jenkins.
4. El trabajo de compilación de Jenkins usa un agente de compilación dinámica en Azure Kubernetes Service para realizar un proceso de compilación del contenedor.
5. Se crea una imagen de contenedor en el código, en el control de código fuente y, a continuación, se inserta en Azure Container Registry.
6. Mediante la implementación continua (CD), Jenkins implementa esta imagen de contenedor actualizada en el clúster de Kubernetes.
7. La aplicación web de Node.js usa Cosmos DB como su back-end. Cosmos DB y Azure Kubernetes Service notifican las métricas a Azure Monitor.
8. Una instancia de Grafana proporciona paneles visuales del rendimiento de la aplicación según los datos de Azure Monitor.

### <a name="components"></a>Componentes

* [Jenkins][jenkins] es un servidor de automatización de código abierto que se integra perfectamente con los servicios de Azure para permitir la integración continua (CI) y la entrega continua (CD). En este escenario, Jenkins orquesta la creación de nuevas imágenes de contenedor en función de las confirmaciones en el control de código fuente, inserta esas imágenes en Azure Container Registry, a continuación, actualiza las instancias de aplicación en Azure Kubernetes Service.
* [Azure Linux Virtual Machines][docs-virtual-machines] es la plataforma IaaS usada para ejecutar las instancias de Jenkins y Grafana.
* [Azure Container Registry][docs-acr] almacena y administra las imágenes de contenedor que usa el clúster de Azure Kubernetes Service. Las imágenes se almacenan de forma segura y se pueden replicar en otras regiones mediante la plataforma Azure, para acelerar los tiempos de implementación.
* [Azure Kubernetes Service][docs-aks] es una plataforma administrada de Kubernetes que permite implementar y administrar aplicaciones en contenedores sin necesidad de tener conocimientos de orquestación de contenedores. Como servicio hospedado de Kubernetes, Azure maneja tareas críticas como la supervisión del estado y el mantenimiento para usted.
* [Azure Cosmos DB][docs-cosmos-db] es una base de datos multimodelo distribuida globalmente que permite elegir entre distintos modelos de base de datos y coherencia en función de sus necesidades. Con Cosmos DB, sus datos se pueden replicar globalmente y no es necesario implementar ni configurar ningún componente de replicación o administración de clústeres.
* [Azure Monitor][docs-azure-monitor] le ayuda a realizar un seguimiento del rendimiento, a mantener la seguridad y a identificar las tendencias. Las métricas obtenidas por el Monitor se pueden usar en otros recursos y herramientas, como Grafana.
* [Grafana][grafana] es una solución de código abierto para consultar, visualizar, alertar y comprender las métricas. Un complemento de origen de datos de Azure Monitor permite a Grafana crear paneles visuales para supervisar el rendimiento de las aplicaciones que se ejecutan en Azure Kubernetes Service y que usan Cosmos DB.

### <a name="alternatives"></a>Alternativas

* [Azure Pipelines][azure-pipelines] ayudan a implementar una canalización de integración continua (CI), prueba e implementación continua (CD) para cualquier aplicación.
* [Kubernetes][kubernetes] se puede ejecutar directamente en máquinas virtuales de Azure en lugar de en un servicio administrado, si desea más control sobre el clúster.
* [Service Fabric][service-fabric] es otro orquestador de contenedores que puede reemplazar a AKS.

## <a name="considerations"></a>Consideraciones

### <a name="availability"></a>Disponibilidad

Para supervisar el rendimiento de la aplicación e informar sobre problemas, este escenario combina Azure Monitor con Grafana para disponer de paneles visuales. Estas herramientas permiten supervisar y solucionar problemas de rendimiento que pueden requerir actualizaciones de código, que después se pueden implementar con la canalización de CI/CD.

Como parte del clúster de Azure Kubernetes Service, un equilibrador de carga distribuye el tráfico de aplicación a uno o varios contenedores (pods) que ejecutan la aplicación. Este método para ejecutar aplicaciones en contenedor en Kubernetes proporciona una infraestructura de alta disponibilidad para sus clientes.

Para ver otros temas sobre disponibilidad, consulte la [lista de comprobación de disponibilidad][availability] que encontrará en el Centro de arquitectura de Azure.

### <a name="scalability"></a>Escalabilidad

Azure Kubernetes Service permite escalar el número de nodos del clúster para satisfacer la demanda de sus aplicaciones. A medida que aumenta la aplicación, puede escalar horizontalmente el número de nodos de Kubernetes que ejecutan el servicio.

Los datos de la aplicación se almacenan en Azure Cosmos DB, una base de datos multimodelo distribuida globalmente que puede ajustar su tamaño globalmente. Cosmos DB elimina la necesidad de escalar la infraestructura, como había que hacer con los componentes de base de datos tradicionales, y permite replicar su base de datos de Cosmos DB globalmente para satisfacer las demandas de los clientes.

Para ver otros temas sobre escalabilidad, consulte la [lista de comprobación de escalabilidad][scalability] que encontrará en el Centro de arquitectura de Azure.

### <a name="security"></a>Seguridad

Para minimizar la superficie de ataque, este escenario no expone la instancia de máquina virtual de Jenkins mediante HTTP. Para cualquier tarea de administración que necesite interactuar con Jenkins, cree una conexión remota segura usando un túnel SSH desde el equipo local. La autenticación de clave pública SSH solo se permite para las instancias de máquina virtual de Jenkins y Grafana. Los inicios de sesión basados en contraseña están deshabilitados. Para más información, consulte [Ejecución de un servidor de Jenkins en Azure](../../reference-architectures/jenkins/index.md).

Para separar las credenciales y los permisos, este escenario usa a una entidad de servicio dedicada de Azure Active Directory (AD). Las credenciales de esta entidad de servicio se almacenan como un objeto de credencial segura en Jenkins, por lo que no están directamente expuestas ni son visibles dentro de los scripts o la canalización de compilación.

Para obtener instrucciones generales sobre el diseño de soluciones seguras, consulte la [documentación de seguridad de Azure][security].

### <a name="resiliency"></a>Resistencia

Este escenario utiliza Azure Kubernetes Service para la aplicación. En Kubernetes se integran los componentes de resistencia que supervisan y reinician los contenedores (pods) en caso de problemas. Combinados con la ejecución de varios nodos de Kubernetes, la aplicación puede tolerar que un pod o un nodo no estén disponibles.

Para obtener instrucciones generales sobre el diseño de soluciones resistentes, consulte [Diseño de aplicaciones resistentes de Azure][resiliency].

## <a name="deploy-the-scenario"></a>Implementación del escenario

**Requisitos previos**

* Debe tener una cuenta de Azure. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.
* Necesita un par de claves públicas SSH. Para ver cómo crear un par de claves públicas, consulte [Creación y uso de un par de claves SSH para máquinas virtuales Linux][sshkeydocs].
* Necesita a una entidad de servicio de Azure Active Directory (AD) para la autenticación del servicio y los recursos. Si es necesario, cree una entidad de servicio con [az ad sp create-for-rbac][createsp].

    ```azurecli-interactive
    az ad sp create-for-rbac --name myDevOpsScenario
    ```

    Anote los valores de *appId* y *password* que se muestran en la salida de este comando. Estos valores se proporcionan en la plantilla al implementar el escenario.

Para implementar este escenario con una plantilla de Azure Resource Manager, realice los pasos siguientes.

1. Haga clic en botón **Implementar en Azure**.<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fdevops-with-aks%2Fazuredeploy.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. Espere a que la implementación de plantilla se abra en Azure Portal y complete los pasos siguientes:
   * Elija **Crear nuevo** para el grupo de recursos y proporcione un nombre, por ejemplo, *myAKSDevOpsScenario*.
   * Seleccione una región en el cuadro de lista desplegable **Ubicación**.
   * Escriba el identificador de aplicación de la entidad de servicio y la contraseña del comando `az ad sp create-for-rbac`.
   * Indique un nombre de usuario y una contraseña segura para la instancia de Jenkins y la consola de Grafana.
   * Proporcione una clave SSH para proteger los inicios de sesión en las máquinas virtuales Linux.
   * Revise los términos y condiciones, y seleccione **Acepto los términos y condiciones indicados anteriormente**.
   * Seleccione el botón **Comprar**.

La implementación puede tardar unos 15-20 minutos en completarse.

## <a name="pricing"></a>Precios

Para explorar el costo de ejecutar este escenario, todos los servicios están preconfigurados en la calculadora de costos. Para ver cómo cambiarían los precios en su caso concreto, cambie las variables pertinentes para que coincidan con el tráfico esperado.

Hemos incluido tres perfiles de costos de ejemplo basados en número de imágenes de contenedor que se almacenarán y los nodos de Kubernetes para ejecutar las aplicaciones.

* [Pequeño][small-pricing]: se corresponde con 1000 compilaciones de contenedor al mes.
* [Mediano][medium-pricing]: se corresponde con 100 000 compilaciones de contenedor al mes.
* [Grande][large-pricing]: se corresponde con 1 000 000 de compilaciones de contenedor al mes.

## <a name="related-resources"></a>Recursos relacionados

Este escenario utiliza Azure Container Registry y Azure Kubernetes Service para almacenar y ejecutar las aplicaciones basadas en contenedores. También se puede usar Azure Container Instances para ejecutar aplicaciones basadas en contenedores, sin tener que aprovisionar componentes de orquestación. Para más información, consulte [Introducción a Azure Container Instances][docs-aci].

<!-- links -->
[architecture]: ./media/architecture-devops-with-aks.png
[autoscaling]: ../../best-practices/auto-scaling.md
[availability]: ../../checklist/availability.md
[docs-aci]: /azure/container-instances/container-instances-overview
[docs-acr]: /azure/container-registry/container-registry-intro
[docs-aks]: /azure/aks/intro-kubernetes
[docs-azure-monitor]: /azure/monitoring-and-diagnostics/monitoring-overview
[docs-cosmos-db]: /azure/cosmos-db/introduction
[docs-virtual-machines]: /azure/virtual-machines/linux/overview
[createsp]: /cli/azure/ad/sp#az-ad-sp-create
[grafana]: https://grafana.com/
[jenkins]: https://jenkins.io/
[resiliency]: ../../resiliency/index.md
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[security]: /azure/security/
[scalability]: ../../checklist/scalability.md
[sshkeydocs]: /azure/virtual-machines/linux/mac-create-ssh-keys
[azure-pipelines]: /azure/devops/pipelines
[kubernetes]: https://kubernetes.io/
[service-fabric]: /azure/service-fabric/

[small-pricing]: https://azure.com/e/841f0a75b1ea4802ba1ac8f7918a71e7
[medium-pricing]: https://azure.com/e/eea0e6d79b4e45618a96d33383ec77ba
[large-pricing]: https://azure.com/e/3faab662c54c473da55a1e93a27e0e64