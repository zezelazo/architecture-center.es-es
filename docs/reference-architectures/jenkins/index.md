---
title: "Ejecución de un servidor Jenkins en Azure"
description: "Esta arquitectura de referencia muestra cómo implementar y hacer funcionar un servidor Jenkins escalable y de nivel empresarial en Azure protegido con un inicio de sesión único (SSO)."
author: njray
ms.date: 01/21/18
ms.openlocfilehash: 724185e43ed743013f52ded04b779552dd8e48c1
ms.sourcegitcommit: 29fbcb1eec44802d2c01b6d3bcf7d7bd0bae65fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/27/2018
---
# <a name="run-a-jenkins-server-on-azure"></a>Ejecución de un servidor Jenkins en Azure

Esta arquitectura de referencia muestra cómo implementar y hacer funcionar un servidor Jenkins escalable y de nivel empresarial en Azure protegido con un inicio de sesión único (SSO). La arquitectura también utiliza Azure Monitor para supervisar el estado del servidor Jenkins. [**Implemente esta solución**.](#deploy-the-solution)

![Servidor Jenkins que se ejecuta en Azure][0]

*Descargue un [archivo de Visio](https://arch-center.azureedge.net/cdn/Jenkins-architecture.vsdx) que contiene este diagrama de arquitectura.*

Esta arquitectura admite la recuperación ante desastres con los servicios de Azure, pero no cubre escenarios de escalabilidad horizontal más avanzados que impliquen múltiples maestros o alta disponibilidad (HA) sin tiempo de inactividad. Para información general sobre los distintos componentes de Azure, incluido un tutorial paso a paso sobre cómo compilar una canalización de CI/CD en Azure, consulte [Jenkins en Azure][jenkins-on-azure].

El enfoque de este documento se centra en las operaciones centrales de Azure necesarias para admitir Jenkins, como el uso de Azure Storage para mantener artefactos de compilación, los elementos de seguridad necesarios para el inicio de sesión único, otros servicios que se pueden integrar y la escalabilidad para la canalización. La arquitectura está diseñada para que funcione con un repositorio de control de código fuente existente. Por ejemplo, un escenario común consiste en iniciar trabajos de Jenkins basados en confirmaciones de GitHub.

## <a name="architecture"></a>Architecture

La arquitectura consta de los siguientes componentes:

-   **Grupo de recursos.** Un [grupo de recursos][rg] se utiliza para agrupar recursos de Azure, para que puedan administrarse según su duración, su propietario y otros criterios. Los grupos de recursos permiten implementar y supervisar los recursos como un grupo, y realizar un seguimiento de los costos de facturación por grupo de recursos. También se pueden eliminar recursos en conjunto, lo que resulta muy útil para implementaciones de prueba.

-   **Servidor Jenkins**. Se implementa una máquina virtual para ejecutar [Jenkins][azure-market] como servidor de automatización y servir como maestro de Jenkins. Esta arquitectura de referencia usa la [plantilla de solución para Jenkins en Azure][solution], instalada en una máquina virtual Linux (Ubuntu 16.04 LTS) en Azure. Otras ofertas de Jenkins están disponibles en Azure Marketplace.

    > [!NOTE]
    > Nginx se instala en la máquina virtual para actuar como un proxy inverso en Jenkins. Puede configurar Nginx para habilitar SSL para el servidor Jenkins.
    > 
    > 

-   **Red virtual**. Una [red virtual][vnet] conecta recursos de Azure entre sí y proporciona aislamiento lógico. En esta arquitectura, el servidor Jenkins se ejecuta en una red virtual.

-   **Subredes**. El servidor Jenkins está aislado en [subred][subnet] para facilitar la administración y segregación del tráfico de la red sin afectar el rendimiento.

-   **NSG**. Use los [grupos de seguridad de red][nsg] (NSG) para restringir el tráfico de red desde Internet a la subred de una red virtual.

-   **Managed Disks**. Un [disco administrado][managed-disk] es un disco duro virtual (VHD) persistente utilizado para el almacenamiento de aplicaciones y también para mantener el estado del servidor Jenkins y proporcionar recuperación ante desastres. Los discos de datos se almacenan en Azure Storage. Para un mayor rendimiento, se recomienda [Premium Storage][premium].

-   **Azure Blob Storage**. El [complemento de Microsoft Azure Storage][configure-storage] utiliza Azure Blob Storage para almacenar los artefactos de compilación que se crean y se comparten con otras compilaciones de Jenkins.

-   **Azure Active Directory (Azure AD)**. [Azure AD][azure-ad] admite la autenticación del usuario, lo que le permite configurar el inicio de sesión único. Las [entidades de servicio ][service-principal] de Azure AD definen la directiva y los permisos para cada autorización de rol en el flujo de trabajo mediante el [control de acceso basado en rol][rbac] (RBAC). Cada entidad de servicio está asociada a un trabajo Jenkins.

-   **Azure Key Vault**. Para administrar los secretos y las claves criptográficas utilizadas para aprovisionar recursos de Azure cuando se requieren secretos, esta arquitectura utiliza [Key Vault][key-vault]. Para obtener ayuda adicional sobre cómo almacenar los secretos asociados a la aplicación de la canalización, consulte también el complemento [Azure Credentials][configure-credential] para Jenkins.

-   **Servicios de supervisión de Azure**. Este servicio [supervisa][monitor] la máquina virtual de Azure que hospeda Jenkins. Esta implementación supervisa el estado de la máquina virtual y el uso de CPU, y envía las alertas.

## <a name="recommendations"></a>Recomendaciones

Las siguientes recomendaciones sirven para la mayoría de los escenarios. Sígalas a menos que tenga un requisito concreto que las invalide.

### <a name="azure-ad"></a>Azure AD

El inquilino de [Azure AD][azure-ad] para la suscripción a Azure se utiliza para habilitar el inicio de sesión único para los usuarios de Jenkins y establecer [entidades de servicio][service-principal] que permitan a los trabajos de Jenkins acceder a los recursos de Azure.

El complemento de Azure AD implementa la autenticación y autorización de inicio de sesión único instalado en el servidor Jenkins. El inicio de sesión único le permite autenticarse con sus credenciales de la organización de Azure AD al iniciar sesión en el servidor Jenkins. Al configurar el complemento de Azure AD, puede especificar el nivel de acceso autorizado del usuario al servidor Jenkin.

Para proporcionar trabajos de Jenkins con acceso a los recursos de Azure, un administrador de Azure AD crea entidades de servicio. Estas entidades conceden a las aplicaciones (en este caso, los trabajos de Jenkins) [un acceso autenticado y autorizado][ad-sp] a los recursos de Azure.

[RBAC][rbac] define y controla aún más el acceso a los recursos de Azure para los usuarios o entidades de servicio mediante su rol asignado. Se admiten roles integrados y personalizados. Los roles también ayudan a proteger la canalización y aseguran que las responsabilidades de un usuario o agente se asignen y autoricen correctamente. Además, RBAC se puede configurar para limitar el acceso a los recursos de Azure. Por ejemplo, un usuario puede limitarse a trabajar únicamente con los recursos de un grupo de recursos determinado.

### <a name="storage"></a>Storage

Utilice el complemento [Microsoft Azure Storage ][storage-plugin] de Jenkins, que se instala desde Azure Marketplace, para almacenar artefactos de compilación que se pueden compartir con otras compilaciones y pruebas. Debe configurarse una cuenta de Azure Storage antes de que los trabajos de Jenkins puedan usar este complemento.

### <a name="jenkins-azure-plugins"></a>Complementos de Jenkins de Azure

La plantilla de solución para Jenkins en Azure instala varios complementos de Azure. El equipo de DevOps de Azure crea y mantiene la plantilla de solución y los siguientes complementos, que funcionan con otras ofertas de Jenkins en Azure Marketplace, así como con cualquier maestro de Jenkins configurado en local:

-   [El complemento Azure AD][configure-azure-ad] permite que el servidor Jenkins admita el inicio de sesión único para los usuarios basados en Azure AD.

-   El complemento [Azure VM Agents][configure-agent] utiliza una plantilla de Azure Resource Manager (ARM) para crear agentes de Jenkins en máquinas virtuales de Azure.

-   El complemento [Azure Credentials][configure-credential] permite almacenar las credenciales de la entidad de servicio de Azure en Jenkins.

-   [El complemento Microsoft Azure Storage][configure-storage] carga artefactos de compilación en [Azure Blob Storage][blob] o descarga de allí dependencias de compilación.

También se recomienda revisar la creciente lista de todos los complementos de Azure disponibles que trabajan con recursos de Azure. Para la lista más reciente, visite el [índice de complementos de Jenkins][index] y busque Azure. Por ejemplo, los complementos siguientes están disponibles para implementación:

-   El complemento [Azure Container Agents][container-agents] le ayuda a ejecutar un contenedor como un agente en Jenkins.

-   [Kubernetes Continuous Deploy](https://aka.ms/azjenkinsk8s) implementa las configuraciones de recursos en un clúster de Kubernetes.

-   [Azure Container Service][acs] implementa configuraciones en Azure Container Service con Kubernetes, DC/OS con Marathon o Docker Swarm.

-   [Azure Functions][functions] implementa su proyecto en una función de Azure.

-   [Azure App Service][app-service] implementa en una instancia de Azure App Service.

## <a name="scalability-considerations"></a>Consideraciones sobre escalabilidad

Jenkins puede escalar para admitir cargas de trabajo muy grandes. Para las compilaciones elásticas, no ejecute las compilaciones en el servidor maestro de Jenkins. En su lugar, descargue las tareas de compilación en los agentes de Jenkins, que se pueden reducir y escalar horizontalmente según sea necesario. Tenga en cuenta dos opciones para escalar agentes:

- Utilice el complemento [Azure VM Agents][vm-agent] para crear agentes de Jenkins que se ejecutan en máquinas virtuales de Azure. Este complemento permite la escalabilidad horizontal elástica para agentes y puede utilizar distintos tipos de máquinas virtuales. Puede seleccionar otra imagen base de Azure Marketplace o usar una imagen personalizada. Para más información sobre cómo escalar los agentes de Jenkins, consulte [Architecting for Scale][scale] (Diseño para la escala) en la documentación de Jenkins.

- Utilice el complemento [Azure Container Agents][container-agents] para ejecutar un contenedor como un agente en [Azure Container Service con Kubernetes](/azure/container-service/kubernetes/) o en [Azure Container Instances](/azure/container-instances/).

Las máquinas virtuales generalmente son más costosas de escalar que los contenedores. Sin embargo, para utilizar contenedores para el escalado, el proceso de compilación debe ejecutarse con contenedores.

Además, utilice Azure Storage para compartir artefactos de compilación que puedan utilizarse en la siguiente etapa de la canalización por otros agentes de compilación.

### <a name="scaling-the-jenkins-server"></a>Escalado del servidor Jenkins 

Puede escalar o reducir verticalmente una máquina virtual del servidor Jenkins si cambia su tamaño. De forma predeterminada, la [plantilla de solución para Jenkins en Azure][azure-market] especifica el tamaño de DS2 v2 (con dos CPU, 7 GB). Este tamaño controla una carga de trabajo pequeña a mediana del equipo. Para cambiar el tamaño de la máquina virtual, elija una opción diferente al crear el servidor. 

Elegir el tamaño correcto del servidor depende el tamaño de la carga de trabajo esperada. La comunidad de Jenkins mantiene una [guía de selección][selection-guide] para ayudar a identificar la configuración que mejor se adapte a sus requisitos. Azure ofrece muchos [tamaños de máquinas virtuales Linux][sizes-linux] para cumplir cualquier requisito. Para más información sobre cómo ampliar el maestro de Jenkins, consulte la comunidad de Jenkins sobre los [procedimientos recomendados][best-practices], que también incluye detalles acerca de cómo escalar un maestro de Jenkins.


## <a name="availability-considerations"></a>Consideraciones sobre disponibilidad

Evalúe los requisitos de disponibilidad para su flujo de trabajo y cómo recuperar el estado de Jenkin en caso de que el servidor Jenkins deje de funcionar. Para evaluar los requisitos de disponibilidad, tenga en cuenta dos métricas comunes:

-   Objetivo de tiempo de recuperación (RTO) especifica cuánto tiempo se puede estar sin Jenkins.

-   Objetivo de punto de recuperación (RPO) indica la cantidad de datos que puede permitirse perder si una interrupción del servicio afecta a Jenkins.

En la práctica, las métricas RTO y el RPO implican redundancia y copia de seguridad. La disponibilidad no es una cuestión de recuperación de hardware, que forma parte de Azure, sino más bien de garantizar que mantenga el estado de su servidor Jenkins. Esta arquitectura de referencia usa el [Acuerdo de Nivel de Servicio de Azure][sla] (SLA), que garantiza un tiempo de actividad del 99,9 % para una única máquina virtual. Si este SLA no cumple sus requisitos de tiempo de actividad, asegúrese de que dispone de un plan de recuperación ante desastres o considere el uso de una implementación de [servidor Jenkins con varios maestros][multi-master] (no se tratan en este documento).

Considere el uso de los [scripts][disaster] de recuperación ante desastres del paso 7 de la implementación para crear una cuenta de Azure Storage con discos administrados para almacenar el estado del servidor Jenkins. Si Jenkins deja de funcionar, se puede restaurar al estado almacenado en esta cuenta de almacenamiento independiente.

## <a name="security-considerations"></a>Consideraciones sobre la seguridad

Utilice los métodos siguientes para poder bloquear la seguridad en un servidor Jenkins básico, ya que en su estado básico no es seguro.

-   Configure una forma de proteger el inicio de sesión en el servidor Jenkins. HTTP no es seguro; de forma predeterminada, esta arquitectura usa HTTP y tiene una dirección IP pública. Considere la posibilidad de configurar [HTTPS en el servidor Nginx][nginx] que se utiliza para un inicio de sesión seguro.

    > [!NOTE]
    > Al agregar SSL al servidor, cree una regla de grupo de seguridad de red para que la subred de Jenkins abra el puerto 443. Para más información, consulte [Apertura de puertos en una máquina virtual con Azure Portal][port443].
    > 

-   Asegúrese de que la configuración de Jenkins evita cross la falsificación de solicitud de entre sitios (Manage Jenkins \> Configure Global Security) [Administrar Jenkins > Configurar seguridad global]. Este es el valor predeterminado del servidor de Microsoft Jenkins.

-   Configure el acceso de solo lectura al panel de Jenkins mediante el uso del [complemento Matrix Authorization Strategy][matrix].

-   Instale el complemento [Azure Credentials][configure-credential] para utilizar Key Vault para controlar los secretos de los recursos de Azure, los agentes en la canalización y los componentes de terceros.

-   Utilice RBAC para restringir el acceso de la entidad de servicio al mínimo requerido para ejecutar los trabajos. Esto ayuda a limitar el alcance de los daños por un trabajo no autorizado.

Los trabajos de Jenkins a menudo requieren secretos para acceder a los servicios de Azure que requieren autorización, como Azure Container Service. Use [Key Vault][key-vault] junto con el complemento [Azure Credentials][configure-credential] para administrar estos secretos de forma segura. Use Key Vault para almacenar las credenciales de la entidad de servicio, las contraseñas, los tokens y otros secretos.

Para obtener una visión central del estado de la seguridad de sus recursos de Azure, utilice [Azure Security Center][security-center]. Security Center supervisa los posibles problemas de seguridad y proporciona una imagen completa del estado de seguridad de su implementación. El Centro de seguridad se configura por cada suscripción de Azure. Habilite la recopilación de datos de seguridad como se describe en la [Guía de inicio rápido de Azure Security Center][quick-start]. Una vez que habilite la recolección, Security Center busca automáticamente las máquinas virtuales creadas en esa suscripción.

El servidor Jenkins tiene su propio sistema de administración de usuarios y la comunidad Jenkins proporciona procedimientos recomendados para [proteger una instancia de Jenkins en Azure][secure-jenkins]. La plantilla de solución para Jenkins en Azure implementa estos procedimientos recomendados.

## <a name="manageability-considerations"></a>Consideraciones sobre la manejabilidad

Utilice los grupos de recursos para organizar los recursos de Azure que se implementan. Implemente entornos de producción y entornos de desarrollo o prueba en grupos de recursos independientes, de modo que pueda supervisar los recursos de cada entorno y acumular los costos de facturación por grupo de recursos. También se pueden eliminar recursos en conjunto, lo que resulta muy útil para implementaciones de prueba.

Azure proporciona varias características para la [supervisión y el diagnóstico][monitoring-diag] de la infraestructura global. Para supervisar el uso de CPU, esta arquitectura implementa Azure Monitor. Por ejemplo, puede utilizar Azure Monitor para supervisar el uso de CPU y enviar una notificación si dicho uso supera el 80 por ciento. (Un uso elevado de CPU indica que es posible que quiera escalar verticalmente la máquina virtual del servidor Jenkins). También puede notificar a un usuario designado si se produce un error en la máquina virtual o deja de estar disponible.

## <a name="communities"></a>Comunidades

Las comunidades pueden responder preguntas y ayudarle a configurar una implementación correcta. Tenga en cuenta lo siguiente.

-   [Blog de la comunidad Jenkins](https://jenkins.io/node/)
-   [Foro de Azure](https://azure.microsoft.com/support/forums/)
-   [Stack Overflow sobre Jenkins](https://stackoverflow.com/tags/jenkins/info)

Para conocer más procedimientos recomendados de la comunidad Jenkins, visite [Jenkins best practices][jenkins-best] (Procedimientos recomendados de Jenkins).

## <a name="deploy-the-solution"></a>Implementación de la solución

Para implementar esta arquitectura, siga los pasos que se indican a continuación para instalar la [plantilla de solución para Jenkins en Azure][azure-market] y, después, instale los scripts que configuran la supervisión y la recuperación ante desastres en los pasos siguientes.

### <a name="prerequisites"></a>requisitos previos

- Esta arquitectura de referencia requiere una suscripción de Azure. 
- Para crear una entidad de servicio de Azure, debe tener derechos de administrador para el inquilino de Azure AD que está asociado al servidor Jenkins implementado.
- Estas instrucciones asumen que el administrador de Jenkins también es un usuario de Azure con al menos privilegios de colaborador.

### <a name="step-1-deploy-the-jenkins-server"></a>Paso 1: Escalado del servidor Jenkins

1.  Abra la [imagen de Azure Marketplace para Jenkins][azure-market] en su explorador web y seleccione **OBTENERLA AHORA** en el lado izquierdo de la página.

2.  Revise los detalles de precios y seleccione **Continuar** y, después, seleccione **Crear** para configurar el servidor de Jenkins en Azure Portal.

Para una instrucción detallada, consulte [Creación de un servidor Jenkins en una máquina virtual Linux de Azure desde Azure Portal][create-jenkins]. Para esta arquitectura de referencia, es suficiente poner en marcha el servidor con el inicio de sesión de administrador. A continuación, puede aprovisionarlo para utilizar otros servicios.

### <a name="step-2-set-up-sso"></a>Paso 2: Configuración del inicio de sesión único

El administrador de Jenkins ejecuta este paso, que debe tener una cuenta de usuario en el directorio de Azure AD de la suscripción y se le debe asignar el rol de colaborador.

Utilice el [complemento Azure AD][configure-azure-ad] desde el centro de actualización de Jenkins en el servidor Jenkins y siga las instrucciones para configurar el inicio de sesión único.

### <a name="step-3-provision-jenkins-server-with-azure-vm-agent-plugin"></a>Paso 3: Aprovisionamiento del servidor Jenkins con el complemento Azure VM Agents

Este paso lo ejecuta el administrador de Jenkins para configurar el complemento Azure VM Agents, que ya está instalado.

[Siga estos pasos para configurar el complemento][configure-agent]. Para leer un tutorial sobre cómo configurar las entidades de servicio para el complemento, consulte [Escalado de las implementaciones de Jenkins para satisfacer la demanda con agentes de máquina virtual de Azure][scale-agent].

### <a name="step-4-provision-jenkins-server-with-azure-storage"></a>Paso 4: Aprovisionamiento del servidor Jenkins con el complemento Azure Storage

Este paso lo ejecuta el administrador de Jenkins, que configura el complemento Microsoft Azure Storage, que ya está instalado.

[Siga estos pasos para configurar el complemento][configure-storage].

### <a name="step-5-provision-jenkins-server-with-azure-credential-plugin"></a>Paso 5: Aprovisionamiento del servidor Jenkins con el complemento Azure Credentials

El paso lo ejecuta el administrador de Jenkins para configurar el complemento Azure Credentials, que ya está instalado.

[Siga estos pasos para configurar el complemento][configure-credential].

### <a name="step-6-provision-jenkins-server-for-monitoring-by-the-azure-monitor-service"></a>Paso 6: Aprovisionamiento del servidor Jenkins para su supervisión por el servicio Azure Monitor

Para configurar la supervisión del servidor Jenkins, siga las instrucciones del artículo [Creación de alertas de métricas en Azure Monitor para servicios de Azure: Azure Portal][create-metric].

### <a name="step-7-provision-jenkins-server-with-managed-disks-for-disaster-recovery"></a>Paso 7: Aprovisionamiento de un servidor Jenkins con discos administrados para la recuperación ante desastres

El grupo de productos de Microsoft Jenkins ha creado scripts de recuperación ante desastres que crean un disco administrado que se utiliza para guardar el estado de Jenkins. Si el servidor deja de funcionar, se puede restaurar a su estado más reciente.

Descargue y ejecute los scripts de recuperación ante desastres en [GitHub][disaster].

[acs]: https://aka.ms/azjenkinsacs
[ad-sp]: /azure/active-directory/develop/active-directory-integrating-applications
[app-service]: https://plugins.jenkins.io/azure-app-service
[azure-ad]: /azure/active-directory/
[azure-market]: https://azuremarketplace.microsoft.com/marketplace/apps/azure-oss.jenkins?tab=Overview
[best-practices]: https://jenkins.io/doc/book/architecting-for-scale/
[blob]: /azure/storage/common/storage-java-jenkins-continuous-integration-solution
[configure-azure-ad]: https://plugins.jenkins.io/azure-ad
[configure-agent]: https://plugins.jenkins.io/azure-vm-agents
[configure-credential]: https://plugins.jenkins.io/azure-credentials
[configure-storage]: https://plugins.jenkins.io/windows-azure-storage
[container-agents]: https://aka.ms/azcontaineragent
[create-jenkins]: /azure/jenkins/install-jenkins-solution-template
[create-metric]: /azure/monitoring-and-diagnostics/insights-alerts-portal
[disaster]: https://github.com/Azure/jenkins/tree/master/disaster_recovery
[functions]: https://aka.ms/azjenkinsfunctions
[index]: https://plugins.jenkins.io
[jenkins-best]: https://wiki.jenkins.io/display/JENKINS/Jenkins+Best+Practices
[jenkins-on-azure]: /azure/jenkins/
[key-vault]: /azure/key-vault/
[managed-disk]: /azure/virtual-machines/linux/managed-disks-overview
[matrix]: https://plugins.jenkins.io/matrix-auth
[monitor]: /azure/monitoring-and-diagnostics/
[monitoring-diag]: /azure/architecture/best-practices/monitoring
[multi-master]: https://jenkins.io/doc/book/architecting-for-scale/
[nginx]: https://www.digitalocean.com/community/tutorials/how-to-create-an-ssl-certificate-on-nginx-for-ubuntu-14-04
[nsg]: /azure/virtual-network/virtual-networks-nsg
[quick-start]: /azure/security-center/security-center-get-started
[port443]: /azure/virtual-machines/windows/nsg-quickstart-portal
[premium]: /azure/virtual-machines/linux/premium-storage
[rbac]: /azure/active-directory/role-based-access-control-what-is
[rg]: /azure/azure-resource-manager/resource-group-overview
[scale]: https://jenkins.io/doc/book/architecting-for-scale/
[scale-agent]: /azure/jenkins/jenkins-azure-vm-agents
[selection-guide]: https://jenkins.io/doc/book/hardware-recommendations/
[service-principal]: /azure/active-directory/develop/active-directory-application-objects
[secure-jenkins]: https://jenkins.io/blog/2017/04/20/secure-jenkins-on-azure/
[security-center]: /azure/security-center/security-center-intro
[sizes-linux]: /azure/virtual-machines/linux/sizes?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json
[solution]: https://azure.microsoft.com/blog/announcing-the-solution-template-for-jenkins-on-azure/
[sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines/
[storage-plugin]: https://wiki.jenkins.io/display/JENKINS/Windows+Azure+Storage+Plugin
[subnet]: /azure/virtual-network/virtual-network-manage-subnet
[vm-agent]: https://wiki.jenkins.io/display/JENKINS/Azure+VM+Agents+plugin
[vnet]: /azure/virtual-network/virtual-networks-overview
[0]: ./images/jenkins-server.png 
