---
title: 'Adopción de la nube empresarial: Implementación de una carga de trabajo básica'
description: Describe cómo implementar una carga de trabajo básica en Azure
author: petertaylor9999
ms.openlocfilehash: d6dd8e107b4b9e48697c4c7bc4d32018eb79de0b
ms.sourcegitcommit: ae8a1de6f4af7a89a66a8339879843d945201f85
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/31/2018
ms.locfileid: "43327418"
---
# <a name="enterprise-cloud-adoption-deploy-a-basic-workload"></a>Adopción de la nube empresarial: Implementación de una carga de trabajo básica

El término **carga de trabajo** normalmente se utiliza para definir una unidad arbitraria de funcionalidad, como una aplicación o un servicio. Pensamos en una carga de trabajo en términos de los artefactos de código que se implementan en un servidor, pero también en términos de otros servicios que sean necesarios. Se trata de una definición útil para una aplicación o servicio local pero, en la nube, necesitamos ampliarla.

En la nube, una carga de trabajo no solo abarca todos los artefactos, también incluye los recursos en la nube. Incluimos los recursos en la nube como parte de la definición debido a un concepto conocido como infraestructura como código. Como ya ha visto en [Cómo funciona Azure](../getting-started/what-is-azure.md), los recursos de Azure se implementan mediante un servicio de orquestador. El servicio de orquestador expone esta funcionalidad a través de una API web y esta API web se puede llamar con varias herramientas, como Powershell, la interfaz de línea de comandos (CLI) de Azure o Azure Portal. Esto significa que podemos especificar nuestros recursos en un archivo legible por máquina que se puede almacenar junto con los artefactos de código asociados a la aplicación.

Esto nos permite definir una carga de trabajo en términos de artefactos de código y los recursos en la nube necesarios, lo que nos permite aislar las cargas de trabajo. Las cargas de trabajo pueden aislarse según la forma en que se organizan los recursos, por su topología de red o por otros atributos. El objetivo del aislamiento de las cargas de trabajo es asociar recursos específicos de la carga de trabajo a un equipo para que este pueda administrar todos los aspectos de esos recursos de forma independiente. Esto permite que varios equipos compartan los servicios de administración de recursos de Azure y evitar la eliminación o modificación accidental de los recursos de los demás.

Este aislamiento también incluye otro concepto conocido como DevOps. DevOps incluye las prácticas de desarrollo de software que incluyen tanto el desarrollo de software y como las operaciones de TI anteriores, pero agrega el uso de la automatización tanto como sea posible. Uno de los principios de DevOps se conoce como integración continua y entrega continua (CI/CD). La integración continua hace referencia a los procesos de compilación automatizados que se ejecutan cada vez que un desarrollador confirma un cambio en el código y la entrega continua hace referencia a los procesos automatizados que implementan este código en los diversos entornos, como un entorno de desarrollo para realizar pruebas o un entorno de producción para la implementación final.

## <a name="basic-workload"></a>Carga de trabajo básica

Una **carga de trabajo básica** normalmente se define como una sola aplicación web o una red virtual (VNet) con máquina virtual (VM). 

> [!NOTE]
> Esta guía no incluye el desarrollo de aplicaciones. Para más información sobre cómo desarrollar aplicaciones en Azure, consulte la [Guía de arquitectura de aplicaciones de Azure](/azure/architecture/guide/).

Independientemente de si la carga de trabajo es una aplicación web o una máquina virtual, cada una de estas implementaciones requiere un **grupo de recursos**. Un usuario con permisos para crear un grupo de recursos debe hacerlo antes de seguir los pasos siguientes.

## <a name="basic-web-application-paas"></a>Aplicación web básica (Paas)

Para una aplicación web básica, seleccione una de las guías de inicio rápido en 5 minutos en la [documentación de Web Apps](/azure/app-service?toc=/azure/architecture/cloud-adoption-guide/toc.json) y siga los pasos. 

> [!NOTE]
> Algunas de las guías de inicio rápido implementarán un grupo de recursos de forma predeterminada. En este caso, no es necesario crear un grupo de recursos de forma explícita. En caso contrario, implemente la aplicación web en el grupo de recursos creado anteriormente.

Una vez implementada la carga de trabajo simple, puede aprender más sobre los procedimientos recomendados para implementar una [aplicación web básica](/azure/architecture/reference-architectures/app-service-web-app/basic-web-app?toc=/azure/architecture/cloud-adoption-guide/toc.json) en Azure.

## <a name="single-windows-or-linux-vm-iaas"></a>Una sola máquina virtual Windows o Linux (IaaS)

Para una carga de trabajo simple que se ejecuta en una máquina virtual, el primer paso es implementar una red virtual. Todos los recursos de IaaS en Azure tales como máquinas virtuales, equilibradores de carga y puertas de enlace requieren una red virtual. Vea más información sobre las [redes virtuales de Azure](/azure/virtual-network/virtual-networks-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json) y, a continuación, siga los pasos para [implementar una red virtual en Azure mediante el portal](/azure/virtual-network/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json). Al especificar la configuración de la red virtual en Azure Portal, especifique el nombre del grupo de recursos creado anteriormente.

El siguiente paso es decidir si va a implementar una sola máquina virtual Windows o Linux. En el caso de una máquina virtual Windows, siga los pasos para [implementar una máquina virtual Windows en Azure con el portal](/azure/virtual-machines/windows/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json). De nuevo, al especificar la configuración de la máquina virtual en Azure Portal, especifique el nombre del grupo de recursos creado anteriormente.

Después de seguir los pasos e implementar la máquina virtual, puede ver los [procedimientos recomendados para ejecutar una máquina virtual Windows en Azure](/azure/architecture/reference-architectures/virtual-machines-windows/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json). En el caso de una máquina virtual Linux, siga los pasos para [implementar una máquina virtual Linux en Azure con el portal](/azure/virtual-machines/linux/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json). También puede ver los [procedimientos recomendados para ejecutar una máquina virtual Linux en Azure](/azure/architecture/reference-architectures/virtual-machines-linux/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json).