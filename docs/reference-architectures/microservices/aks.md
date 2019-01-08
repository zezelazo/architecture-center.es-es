---
title: Arquitectura de microservicios en Azure Kubernetes Service (AKS)
description: Implementación de una arquitectura de microservicios en Azure Kubernetes Service (AKS)
author: MikeWasson
ms.date: 12/10/2018
ms.openlocfilehash: 9e4b607cd7f5b33bbf08ce3af67dd5d4071ae8ef
ms.sourcegitcommit: bb7fcffbb41e2c26a26f8781df32825eb60df70c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/20/2018
ms.locfileid: "53644247"
---
# <a name="microservices-architecture-on-azure-kubernetes-service-aks"></a>Arquitectura de microservicios en Azure Kubernetes Service (AKS)

Esta arquitectura de referencia muestra una aplicación de microservicios implementada en Azure Kubernetes Service (AKS). Muestra una configuración básica de AKS que puede ser el punto de partida para la mayoría de las implementaciones. Las opciones más avanzadas, incluidas las opciones de redes avanzadas, se tratarán en una arquitectura de referencia independiente.

En este artículo se da por supuesto un conocimiento básico de Kubernetes. El artículo se centra principalmente en la infraestructura y las consideraciones sobre DevOps de la ejecución de una arquitectura de microservicios en AKS. Para obtener instrucciones sobre cómo diseñar microservicios desde una perspectiva de diseño controlado por dominio (DDD), consulte [Diseño, creación y operación de microservicios en Azure](/azure/architecture/microservices).

> [!NOTE]
> Estamos trabajando en una implementación de referencia (RI) para acompañar a este artículo que esperamos publicar a principios de 2019. Este artículo se actualizará para incorporar los procedimientos recomendados adicionales de dicha RI.

![Arquitectura de referencia de AKS](./_images/aks.png)

## <a name="architecture"></a>Arquitectura

La arquitectura consta de los siguientes componentes:

**Azure Kubernetes Service** (AKS). AKS es un servicio de Azure que implementa un clúster de Kubernetes administrado. 

**Clúster de Kubernetes**. AKS es responsable de implementar el clúster de Kubernetes y de administrar los maestros de Kubernetes. Solo debe administrar los nodos de agente.

**Red virtual**. De forma predeterminada, AKS crea una red virtual para implementar en ella los nodos de agente. Para escenarios más avanzados, puede crear la red virtual en primer lugar, lo que le permite controlar elementos como la configuración de las subredes, la conectividad local y el direccionamiento IP. Para más información, consulte [Configuración de redes avanzadas en Azure Kubernetes Service (AKS)](/azure/aks/configure-advanced-networking).

**Entrada**. Una entrada expone rutas HTTP(S) a servicios dentro del clúster. Para más información, consulte la sección [Puerta de enlace de API](#api-gateway) a continuación.

**Almacenes de datos externos**. Los microservicios son normalmente sin estado y escriben el estado en almacenes de datos externos, como Azure SQL Database o Cosmos DB.

**Azure Active Directory**. AKS usa una identidad de Azure Active Directory (Azure AD) para crear y administrar otros recursos de Azure, como los equilibradores de carga de Azure. Azure AD también se recomienda para la autenticación de usuario en aplicaciones cliente.

**Azure Container Registry**. Utilice Container Registry para almacenar imágenes de Docker privadas, que se implementan en el clúster. AKS se puede autenticar con Container Registry con su identidad de Azure AD. Tenga en cuenta que AKS no requiere Azure Container Registry. Puede usar otros registros de contenedor, como Docker Hub.

**Azure Pipelines**. Pipelines es parte de Azure DevOps Services y ejecuta compilaciones, pruebas e implementaciones automatizadas. También puede usar soluciones de CI/CD de terceros como Jenkins. 

**Helm**. Helm es un administrador de paquetes para Kubernetes: una manera de agrupar objetos de Kubernetes en una sola unidad que se puede publicar, implementar, versionar y actualizar.

**Azure Monitor**. Azure Monitor recopila y almacena métricas y registros, incluidas las métricas de plataforma para los servicios de Azure de la solución y los datos de telemetría de la aplicación. Use estos datos para supervisar la aplicación, configurar alertas y paneles y realizar el análisis de causa principal de los errores. Azure Monitor se integra con AKS para recopilar métricas de controladores, nodos y contenedores, así como los registros de contenedor y los registros del nodo maestro.

## <a name="design-considerations"></a>Consideraciones de diseño

Esta arquitectura de referencia se centra en las arquitecturas de microservicios, aunque muchos de los procedimientos recomendados se aplicarán a otras cargas de trabajo que se ejecutan en AKS.

### <a name="microservices"></a>Microservicios

El objeto de Kubernetes Service es una manera natural para modelar microservicios en Kubernetes. Un microservicio es una unidad de código de acoplamiento flexible que se puede implementar de forma independiente. Los microservicios normalmente se comunican mediante API bien definidas y son detectables por algún tipo de detección de servicios. El objeto de Kubernetes Service proporciona un conjunto de funcionalidades que coinciden con estos requisitos:

- Dirección IP. El objeto del servicio proporciona una dirección IP interna estática para un grupo de pods (conjunto de réplicas). A medida que los pods se crean o se mueven, el servicio siempre está accesible en esta dirección IP interna.

- Equilibrio de carga. La carga del tráfico enviado a la dirección IP del servicio se equilibra en los pods. 

- Detección de servicios. El servicio DNS de Kubernetes asigna entradas DNS internas a los servicios. Esto significa que la puerta de enlace de API puede llamar a un servicio de back-end con el nombre DNS. Se puede usar el mismo mecanismo para la comunicación de servicio a servicio. Las entradas DNS están organizadas por espacio de nombres, por lo que si los espacios de nombres se corresponden con contextos limitados, el nombre DNS de un servicio se asignará de forma natural al dominio de aplicación.

El diagrama siguiente muestra la relación conceptual entre servicios y pods. La asignación real a los puertos y las direcciones IP del punto de conexión se realiza mediante kube-proxy, el proxy de red de Kubernetes.

![Servicios y pods](./_images/aks-services.png)

### <a name="api-gateway"></a>API Gateway

Una *puerta de enlace de API* es una puerta de enlace que se encuentra entre los clientes externos y los microservicios. Actúa como un proxy inverso, enrutando las solicitudes de los clientes a los microservicios. También puede realizar diversas tareas transversales como la autenticación, la terminación SSL y la limitación de velocidad. 

La funcionalidad proporcionada por una puerta de enlace se puede agrupar como sigue:

- [Enrutamiento de puerta de enlace](../../patterns/gateway-routing.md): Enrutamiento de las solicitudes de cliente a los servicios de back-end correctos. Esto proporciona un único punto de conexión para los clientes y ayuda a desacoplar los clientes de los servicios.

- [Agregación de puerta de enlace](../../patterns/gateway-aggregation.md): Agregación de varias solicitudes en una sola solicitud, para reducir el intercambio de mensajes entre el cliente y el back-end.

- [Descarga con puertas de enlace](../../patterns/gateway-offloading.md). Una puerta de enlace puede descargar de funcionalidades a los servicios de back-end, como la terminación SSL, la autenticación, las listas de direcciones IP permitidas o la limitación de velocidad de cliente.

Las puertas de enlace de API son un [patrón de diseño de microservicios](https://microservices.io/patterns/apigateway.html) general. Se pueden implementar mediante una serie de tecnologías diferentes. Probablemente la implementación más común es implementar un enrutador perimetral o un proxy inverso, como Nginx, HAProxy o Traefik, dentro del clúster. 

Otras opciones incluyen:

- Azure Application Gateway o Azure API-Management, que son servicios administrados que residen fuera del clúster. Actualmente está en versión beta un controlador de entrada de la puerta de enlace de aplicaciones.

- Azure Functions Proxies. Los servidores proxy pueden modificar las solicitudes y las respuestas y enrutar las solicitudes en función de direcciones URL.

El tipo de recurso **Entrada** de Kubernetes abstrae las opciones de configuración de un servidor proxy. Funciona junto con un controlador de entrada, que proporciona la implementación subyacente de la entrada. Hay controladores de entrada para Nginx, HAProxy, Traefik y Application Gateway (versión preliminar), entre otros.

El controlador de entrada controla la configuración del servidor proxy. A menudo requieren archivos de configuración complejos, que pueden ser difíciles de ajustar si no es un experto, por lo que el controlador de entrada es una buena abstracción. Además, el controlador de entrada tiene acceso a la API de Kubernetes, por lo que puede tomar decisiones inteligentes sobre enrutamiento y equilibrio de carga. Por ejemplo, el controlador de entrada Nginx omite al proxy de red kube-proxy.

Por otro lado, si necesita control absoluto sobre la configuración, puede omitir esta abstracción y configurar manualmente el servidor proxy. 

Un servidor proxy inverso es un posible cuello de botella o un único punto de error, por lo que siempre debe implementar al menos dos réplicas para la alta disponibilidad.

### <a name="data-storage"></a>Almacenamiento de datos

En una arquitectura de microservicios, los servicios no deben compartir el almacenamiento de datos. Cada servicio debe ser propietario de sus propios datos privados en un almacenamiento lógico independiente, para evitar dependencias ocultas entre servicios. La razón es evitar el acoplamiento involuntario entre servicios, lo que puede suceder cuando los servicios comparten los mismos esquemas de datos subyacentes. Además, cuando los servicios administran sus propios almacenes de datos, pueden usar el almacén de datos adecuado para sus requisitos particulares. Para más información, consulte [Diseño de microservicios: consideraciones sobre los datos](/azure/architecture/microservices/data-considerations).

Evite almacenar datos permanentes en el almacenamiento del clúster local, ya que esto ata los datos al nodo. En su lugar: 

- Use un servicio externo como Azure SQL Database o Cosmos DB. *O bien,*

- Monte un volumen persistente con discos de Azure o Azure Files. Use Azure Files si el mismo volumen debe ser compartido por varios pods.

### <a name="namespaces"></a>Espacios de nombres

Use espacios de nombres para organizar los servicios dentro del clúster. Todos los objetos de un clúster de Kubernetes pertenecen a un espacio de nombres. De forma predeterminada, cuando se crea un nuevo objeto, este se coloca en el espacio de nombres `default`. Pero es una buena práctica crear espacios de nombres más descriptivos para ayudar a organizar los recursos del clúster.

En primer lugar, los espacios de nombres ayudar a evitar colisiones de nomenclatura. Cuando varios equipos implementan microservicios en el mismo clúster, posiblemente con cientos de microservicios, se hace difícil de administrar si todos se incluyen en el mismo espacio de nombres. Además, los espacios de nombres permiten:

- Aplicar restricciones de recursos a un espacio de nombres para que el conjunto total de pods asignados a ese espacio de nombres no pueda superar la cuota de recursos del espacio de nombres.

- Aplicar directivas en el nivel de espacio de nombres, incluidas las directivas de RBAC y seguridad.

Para una arquitectura de microservicios, considere la organización de los microservicios en contextos limitados y la creación de espacios de nombres para cada contexto limitado. Por ejemplo, todos los microservicios relacionados con el contexto limitado "Pedidos" podrían ir en el mismo espacio de nombres. Como alternativa, cree un espacio de nombres para cada equipo de desarrollo.

Coloque los servicios auxiliares en su propio espacio de nombres independiente. Por ejemplo, podría implementar Elasticsearch o Prometheus para la supervisión de clústeres o Tiller para Helm.

## <a name="scalability-considerations"></a>Consideraciones sobre escalabilidad

Kubernetes admite el escalado horizontal en dos niveles:

- Escalado del número de pods que se asigna a una implementación.
- Escalado de los nodos del clúster, para aumentar los recursos de proceso totales disponibles para el clúster.

Aunque puede escalar horizontalmente los pods y los nodos manualmente, se recomienda usar el escalado automático para minimizar la posibilidad de que los servicios se queden con recursos escasos en cargas elevadas. Una estrategia de escalado automático debe tienen en cuenta los pods y los nodos. Si solo escala horizontalmente los pods, podría alcanzar los límites de recursos de los nodos. 

### <a name="pod-autoscaling"></a>Escalado automático de pods

Horizontal Pod Autoscaler (HPA) escala los pods en función de la CPU y la memoria observadas o de métricas personalizadas. Para configurar el escalado horizontal de pods, especifique una métrica de destino (por ejemplo, el 70% de CPU) y el número mínimo y máximo de réplicas. Debe realizar una prueba de carga de los servicios para averiguar estas cifras.

Un efecto secundario del escalado automático es que los pods podrían crearse o expulsarse con más frecuencia, a medida que se producen eventos de escalado horizontal y reducción horizontal. Para mitigar los efectos de esto:

- Use sondeos de preparación para que Kubernetes sepa cuándo está listo un pod nuevo para aceptar tráfico.
- Use presupuestos de interrupciones de pods para limitar el número de pods que se puede expulsar de un servicio a la vez.

### <a name="cluster-autoscaling"></a>Escalado automático del clúster

El escalador automático del clúster escala el número de nodos. Si no se pueden programar pods debido a restricciones de recursos, el escalador automático del clúster aprovisionará más nodos.  (Nota: La integración entre AKS y el escalador automático del clúster está actualmente en versión preliminar).

Mientras que HPA examina los recursos reales consumidos u otras métricas de la ejecución de pods, el escalador automático del clúster aprovisiona nodos para pods que no están programados todavía. Por lo tanto, examina los recursos solicitados, como se especifica en la especificación de pods de Kubernetes para una implementación. Use pruebas de carga para ajustar estos valores.

No se puede cambiar el tamaño de máquina virtual después de crear el clúster, por lo que debe realizar algún planeamiento inicial de capacidad para elegir un tamaño de máquina virtual adecuado para los nodos de agente cuando se crea el clúster. 

## <a name="availability-considerations"></a>Consideraciones sobre disponibilidad

### <a name="health-probes"></a>Sondeos de estado

Kubernetes define dos tipos de sondeo de mantenimiento que puede exponer un pod:

- Sondeo de preparación: indica a Kubernetes si el pod está listo para aceptar solicitudes.

- Sondeo de ejecución: indica a Kubernetes si se debería quitar un pod e iniciar una nueva instancia.

Al pensar en los sondeos, es útil recordar cómo funciona un servicio en Kubernetes. Un servicio tiene un selector de etiqueta que coincide con un conjunto de pods (cero o más). Kubernetes equilibra el tráfico a los pods que coinciden con el selector. Solo los pods que se iniciaron correctamente y que están en buen estado recibirán tráfico. Si se bloquea un contenedor, Kubernetes elimina el pod y programa un reemplazo.

En ocasiones, un pod puede no estar listo para recibir tráfico aunque el pod se inició correctamente. Por ejemplo, puede haber tareas de inicialización, en las que la aplicación que se ejecuta en el contenedor carga elementos en la memoria o lee datos de configuración. Para indicar que un pod está correcto pero no está listo para recibir tráfico, defina un sondeo de preparación. 

Los sondeos de ejecución controlan el caso de un pod que sigue en ejecución, pero está en mal estado y se debe reciclar. Por ejemplo, suponga que un contenedor da servicio a solicitudes HTTP, pero se bloquea por algún motivo. El contenedor no se bloquea, pero ha dejado de servir solicitudes. Si define un sondeo de ejecución HTTP, el sondeo dejará de responder y esto informa a Kubernetes para reiniciar el pod.

Estas son algunas consideraciones al diseñar los sondeos:

- Si el código tiene un tiempo de inicio elevado, hay un riesgo de que un sondeo de ejecución notifique un error antes de que finalice el inicio. Para evitar esto, use la configuración initialDelaySeconds, que retrasa el inicio del sondeo.

- Un sondeo de ejecución no ayuda a menos que reiniciar el pod lo restaure a un estado correcto. Puede usar un sondeo de ejecución para mitigar bloqueos inesperados o fugas de memoria, pero no hay ningún interés en reiniciar un pod que va a producir inmediatamente un nuevo error.

- A veces se usan sondeos de preparación para comprobar los servicios dependientes. Por ejemplo, si un pod tiene una dependencia en una base de datos, el sondeo de ejecución podría comprobar la conexión de base de datos. Sin embargo, este enfoque puede crear problemas inesperados. Un servicio externo podría no estar disponible temporalmente por algún motivo. Esto hará que el sondeo de preparación genere un error para todos los pods del servicio, causando la eliminación de todos ellos del equilibrio de carga y así crear errores en cascada ascendentes. Un enfoque mejor consiste en implementar un control de reintentos en el servicio, para que el servicio se pueda recuperar correctamente de errores transitorios.

### <a name="resource-constraints"></a>Restricciones de recursos

La contención de recursos puede afectar a la disponibilidad de un servicio. Defina restricciones de recursos para los contenedores, para que un único contenedor no pueda sobrecargar los recursos de clúster (memoria y CPU). Para recursos que no están en contenedores, como los subprocesos o las conexiones de red, considere el uso del [patrón Bulkhead](/azure/architecture/patterns/bulkhead) para aislar los recursos.

Utilice cuotas de recursos para limitar los recursos totales permitidos para un espacio de nombres. De este modo, el front-end no puede dejar sin recursos a los servicios de back-end o viceversa.

## <a name="security-considerations"></a>Consideraciones sobre la seguridad

### <a name="role-based-access-control-rbac"></a>Control de acceso basado en roles (RBAC)

Kubernetes y Azure tienen mecanismos de control de acceso basado en rol (RBAC):

- El control de acceso basado en rol de Azure controla el acceso a los recursos de Azure, incluida la capacidad de crear nuevos recursos de Azure. Los permisos se pueden asignar a usuarios, grupos o entidades de servicio. (Una entidad de servicio es una identidad de seguridad utilizada por las aplicaciones).

- El control de acceso basado en rol de Kubernetes controla los permisos para la API de Kubernetes. Por ejemplo, la creación de pods y la enumeración de pods son acciones que se pueden autorizar (o denegar) a un usuario mediante RBAC. Para asignar permisos de Kubernetes a los usuarios, puede crear *roles* y *enlaces de rol*:

  - Un rol es un conjunto de permisos que se aplican dentro de un espacio de nombres. Los permisos se definen como verbos (obtener, actualizar, crear, eliminar) en los recursos (pods, implementaciones, etc).

  - Un enlace de rol asigna usuarios o grupos a un rol.

  - También hay un objeto ClusterRole, que es similar a un rol, pero se aplica a todo el clúster en todos los espacios de nombres. Para asignar usuarios o grupos a un objeto ClusterRole, cree un objeto ClusterRoleBinding.

AKS integra estos dos mecanismos de RBAC. Cuando se crea un clúster de AKS, puede configurarlo para usar Azure AD para la autenticación de usuario. Para más información sobre cómo configurar esta opción, consulte [Integración de Azure Active Directory con Azure Kubernetes Service](/azure/aks/aad-integration).

Una vez configurado, un usuario que desea acceder a la API de Kubernetes (por ejemplo, mediante kubectl) debe iniciar sesión con sus credenciales de Azure AD.

De forma predeterminada, un usuario de Azure AD no tiene acceso al clúster. Para conceder acceso, el administrador del clúster crea enlaces de rol que hacen referencia a usuarios o grupos de Azure AD. Si un usuario no tiene permisos para una operación determinada, se producirá un error.

Si los usuarios no tienen acceso de forma predeterminada, ¿cómo tiene permisos el administrador del clúster para crear los enlaces de rol en primer lugar? Un clúster de AKS realmente tiene dos tipos de credenciales para llamar al servidor de API de Kubernetes: usuario del clúster y administrador del clúster. Las credenciales de administrador del clúster otorgan acceso completo al clúster. El comando `az aks get-credentials --admin` de la CLI de Azure descarga las credenciales de administrador del clúster y las guarda en el archivo kubeconfig. El administrador del clúster puede utilizar este archivo kubeconfig para crear roles y enlaces de rol.

Dado que las credenciales de administrador del clúster son muy potentes, utilice RBAC de Azure para restringir su acceso:

- El "rol de administrador del clúster de Azure Kubernetes Service" tiene permisos para descargar las credenciales de administrador del clúster. Solo se debe asignar este rol a los administradores del clúster.

- El "rol de usuario del clúster de Azure Kubernetes Service" tiene permisos para descargar las credenciales de usuario del clúster. Se puede asignar este rol a los usuarios que no son administradores. Este rol no otorga permisos específicos sobre los recursos de Kubernetes del clúster; simplemente permite que un usuario se conecte al servidor de API. 

Al definir las directivas de RBAC (en Kubernetes y Azure), piense en los roles de la organización:

- ¿Quién puede crear o eliminar un clúster de AKS y descargar las credenciales de administrador?
- ¿Quién puede administrar un clúster?
- ¿Quién puede crear o actualizar los recursos de un espacio de nombres?

Es una buena práctica delimitar el ámbito de los permisos de RBAC de Kubernetes por espacio de nombres, mediante roles y enlaces de roles, en lugar de utilizar objetos ClusterRoles y ClusterRoleBindings.

Por último, queda la pregunta de qué permisos tiene el clúster de AKS para crear y administrar recursos de Azure, como equilibradores de carga, redes o almacenamiento. Para autenticarse a sí mismo con las API de Azure, el clúster usa a una entidad de servicio de Azure AD. Si no se especifica una entidad de servicio al crear el clúster, se crea una automáticamente. Sin embargo, es una buena práctica de seguridad crear la entidad de servicio en primer lugar y asignarle permisos de RBAC mínimos. Para más información, consulte [Entidades de servicio con Azure Kubernetes Service](/azure/aks/kubernetes-service-principal).

### <a name="secrets-management-and-application-credentials"></a>Administración de secretos y credenciales de aplicaciones

Las aplicaciones y los servicios a menudo necesitan credenciales para conectarse a servicios externos, como Azure Storage o SQL Database. El desafío consiste en proteger estas credenciales y que no se filtren. 

Para los recursos de Azure, una opción es usar identidades administradas. La idea de una identidad administrada es que una aplicación o un servicio tienen una identidad que se almacena en Azure AD y usan esta identidad para autenticarse con un servicio de Azure. La aplicación o el servicio tienen una entidad de servicio creada en Azure AD y se autentican mediante tokens de OAuth 2.0. El proceso en ejecución llama a una dirección de host local para obtener el token. De este modo, no es necesario almacenar contraseñas ni cadenas de conexión. Puede utilizar identidades administradas en AKS mediante la asignación de identidades a pods individuales, con el proyecto [aad-pod-identity](https://github.com/Azure/aad-pod-identity).

Actualmente, no todos los servicios de Azure admiten la autenticación con identidades administradas. Para obtener una lista, consulte [Servicios de Azure que admiten la autenticación de Azure AD](/azure/active-directory/managed-identities-azure-resources/services-support-msi).

Incluso con identidades administradas, probablemente necesitará almacenar algunas credenciales u otros secretos de aplicación, ya sea para servicios de Azure que no admiten identidades administradas, servicios de terceros o claves de API. Estas son algunas opciones para almacenar secretos de forma segura:

- Azure Key Vault. En AKS, puede montar uno o más secretos de Key Vault como un volumen. El volumen lee los secretos de Key Vault. El pod, a continuación, puede leer los secretos al igual que en un volumen normal. Para más información, consulte el proyecto [Kubernetes-KeyVault-FlexVolume](https://github.com/Azure/kubernetes-keyvault-flexvol) en GitHub.

    El pod se autentica mediante una identidad de pod (descrita anteriormente) o mediante el uso de una entidad de servicio de Azure AD junto con un secreto de cliente. Se recomienda el uso de identidades de pod porque el secreto de cliente no es necesario en ese caso. 

- HashiCorp Vault. Las aplicaciones de Kubernetes se pueden autenticar con HashiCorp Vault mediante identidades administradas de Azure AD. Consulte [HashiCorp Vault speaks Azure Active Directory](https://open.microsoft.com/2018/04/10/scaling-tips-hashicorp-vault-azure-active-directory/) (HashiCorp Vault se comunica con Azure Active Directory). Puede implementar el propio almacén en Kubernetes, pero es aconsejable ejecutarlo en un clúster dedicado independiente del clúster de la aplicación. 

- Secretos de Kubernetes. Otra opción es usar secretos de Kubernetes. Esta opción es la más fácil de configurar pero tiene algunos desafíos. Los secretos se almacenan en etcd, que es un almacén distribuido de pares clave-valor. AKS [cifra etcd en reposo](https://github.com/Azure/kubernetes-kms#azure-kubernetes-service-aks). Microsoft administra las claves de cifrado.

El uso de un sistema como HashiCorp Vault o Azure Key Vault proporciona varias ventajas, como:

- Control centralizado de los secretos.
- Garantías de que todos los secretos se cifran en reposo.
- Administración de claves centralizada.
- Control de acceso a los secretos.
- Auditoría

### <a name="pod-and-container-security"></a>Seguridad del pod y el contenedor

Esta lista ciertamente no es exhaustiva, pero estos son algunos procedimientos recomendados para proteger los pods y los contenedores: 

No ejecute contenedores en modo con privilegios. El modo con privilegios otorga a un contenedor acceso a todos los dispositivos del host. Puede establecer una directiva de seguridad del pod para no permitir que los contenedores se ejecuten en modo con privilegios. 

Cuando sea posible, evite ejecutar procesos como root dentro de los contenedores. Los contenedores no proporcionan un aislamiento completo desde la perspectiva de la seguridad, por lo que es mejor ejecutar un proceso de contenedor como un usuario sin privilegios. 

Almacene las imágenes en un registro privado de confianza, como Azure Container Registry o Docker Trusted Registry. Use un webhook de validación de admisión en Kubernetes para asegurarse de que los pods solo pueden extraer imágenes desde el registro de confianza.

Examine las imágenes en busca de vulnerabilidades conocidas, mediante una solución de análisis como Twistlock y Aqua, que están disponibles en Azure Marketplace.

Automatice la aplicación de revisiones de imágenes con ACR Tasks, una característica de Azure Container Registry. Una imagen de contenedor se compone de capas. Las capas de base incluyen la imagen del sistema operativo y las imágenes del marco de trabajo de la aplicación, como ASP.NET Core o Node.js. Los desarrolladores de aplicaciones normalmente crean de modo ascendente las imágenes de base y otros desarrolladores del proyecto las mantienen. Cuando se aplican revisiones de modo ascendente a estas imágenes, es importante actualizar, probar y volver a implementar las propias imágenes para no dejar vulnerabilidades de seguridad conocidas. ACR Tasks puede ayudar a automatizar este proceso.

## <a name="deployment-cicd-considerations"></a>Consideraciones de implementación (CI/CD)

Estos son algunos objetivos de un proceso de CI/CD sólido para una arquitectura de microservicios:

- Cada equipo puede compilar e implementar los servicios que posee de forma independiente, sin afectar ni interrumpir a otros equipos.

- Antes de implementar en producción una nueva versión de un servicio, se implementa en entornos de desarrollo, pruebas y control de calidad para su validación. Se aplican pruebas de calidad en cada fase.

- Se puede implementar una nueva versión de un servicio en paralelo con la versión anterior.

- Hay en vigor directivas de control de acceso suficientes.

- Puede confiar en las imágenes de contenedor que se implementan en producción.

### <a name="isolation-of-environments"></a>Aislamiento de los entornos

Tendrá varios entornos en los que implementa los servicios, incluidos los entornos de desarrollo, prueba de aceptación de la compilación, pruebas de integración, pruebas de carga y, por último, producción. Estos entornos necesitan cierto nivel de aislamiento. En Kubernetes, puede optar entre el aislamiento físico y el aislamiento lógico. El aislamiento físico significa implementar en clústeres independientes. El aislamiento lógico hace uso de espacios de nombres y directivas, como se describió anteriormente.

Nuestra recomendación es crear un clúster de producción dedicado junto con un clúster independiente para los entornos de desarrollo y pruebas. Use el aislamiento lógico para separar los entornos dentro del clúster de desarrollo y pruebas. Los servicios implementados en el clúster de desarrollo y pruebas nunca deben tener acceso a almacenes de datos que contengan datos empresariales. 

### <a name="helm"></a>Helm

Considere el uso de Helm para administrar la creación e implementación de servicios. Algunas de las características de Helm que ayudan en CI/CD son:

- Organización de todos los objetos de Kubernetes de un microservicio determinado en un único gráfico de Helm.
- Implementación del gráfico como un comando de Helm único, en lugar de una serie de comandos de kubectl.
- Seguimiento de actualizaciones y revisiones, con control de versiones semántico, junto con la capacidad de revertir a una versión anterior.
- El uso de plantillas para evitar la duplicación de información, como etiquetas y selectores, en muchos archivos.
- Administración de dependencias entre gráficos.
- Publicación de gráficos en un repositorio de Helm, como Azure Container Registry y su integración con la canalización de compilación.

Para más información sobre el uso de Container Registry como un repositorio de Helm, consulte [Uso de Azure Container Registry como un repositorio de Helm para los gráficos de aplicación](/azure/container-registry/container-registry-helm-repos).

### <a name="cicd-workflow"></a>Flujo de trabajo de CI/CD

Antes de crear un flujo de trabajo de CI/CD, es preciso saber cómo se va a estructurar y administrar el código base.

- ¿Los equipos trabajan en repositorios independientes o en un único repositorio?
- ¿Cuál es su estrategia de ramificación?
- ¿Quién puede insertar código en producción? ¿Hay un rol de administrador de versión?

El enfoque del repositorio único cada vez tiene más adeptos, pero ambos métodos tienen sus ventajas y desventajas.

| &nbsp; | Un repositorio | Varios repositorios |
|--------|----------|----------------|
| **Ventajas** | Uso compartido de código<br/>Mayor facilidad para estandarizar el código y las herramientas<br/>Mayor facilidad para refactorizar el código<br/>Detectabilidad (vista única del código)<br/> | Propiedad clara por equipo<br/>Posiblemente menos conflictos en la fusión mediante combinación<br/>Ayuda a aplicar el desacoplamiento de los microservicios |
| **Desafíos** | Los cambios en el código compartido pueden afectar a varios microservicios<br/>Mayor posibilidad de conflictos en la fusión mediante combinación<br/>Las herramientas se deben escalar a un código base grande<br/>Control de acceso<br/>Proceso de implementación más complejo | Mayor dificultad para compartir código<br/>Mayor dificultad para aplicar los estándares de codificación<br/>Administración de dependencias<br/>Código base difuso, mala detectabilidad<br/>Falta de infraestructura compartida

En esta sección, presentamos un posible flujo de trabajo de CI/CD, en función de los siguientes supuestos:

- El repositorio de código es un único y sus carpetas están organizadas por microservicio.
- La estrategia de ramificación del equipo se basa en el [desarrollo basado en el tronco](https://trunkbaseddevelopment.com/).
- El equipo usa [Azure Pipelines](/azure/devops/pipelines) para ejecutar el proceso de CI/CD.
- El equipo usa [espacios de nombres](/azure/container-registry/container-registry-best-practices#repository-namespaces) en Azure Container Registry para aislar las imágenes que están aprobadas para la producción de las imágenes que se siguen probando.

En este ejemplo, un desarrollador está trabajando en un microservicio denominado Delivery Service (el nombre procede de la implementación de referencia que se describe [aquí](../../microservices/index.md#the-drone-delivery-application)). Al desarrollar una característica nueva, el desarrollador comprueba el código de una rama de características.

![Flujo de trabajo de CI/CD](./_images/aks-cicd-1.png)

Las confirmaciones de inserción en esta rama desencadena una compilación de CI para el microservicio. Por convención, las ramas de características se denominan `feature/*`. El [archivo de definición de compilación](/azure/devops/pipelines/yaml-schema) incluye un desencadenador que filtra por el nombre de rama y la ruta de acceso de origen. Con este enfoque, cada equipo puede tener su propia canalización de compilación.

```yaml
trigger:
  batch: true
  branches:
    include:
    - master
    - feature/*

    exclude:
    - feature/experimental/*

  paths:
     include:
     - /src/shipping/delivery/
```

En este punto del flujo de trabajo, la compilación de CI ejecuta una comprobación mínima del código:

1. Compile el código
1. Ejecute pruebas unitarias

La idea aquí es que las compilaciones tarden poco en completarse, con el fin de que el desarrollador pueda obtener comentarios rápidamente. Cuando la característica esté lista para combinarse con la maestra, el desarrollador abre una solicitud de inserción. Esto desencadena otra compilación de integración continua que realiza más comprobaciones:

1. Compile el código
1. Ejecute pruebas unitarias
1. Compile la imagen de contenedor del runtime
1. Ejecute exámenes de vulnerabilidades en la imagen

![Flujo de trabajo de CI/CD](./_images/aks-cicd-2.png)

> [!NOTE]
> En Azure Repos se pueden definir [directivas](/azure/devops/repos/git/branch-policies) para proteger las ramas. Por ejemplo, la directiva puede requerir una compilación de CI correcta, además de que un aprobador cierre sesión para poder realizan la combinación con el maestro.

En algún momento, el equipo estará preparado para implementar una nueva versión del microservicio Delivery Service. Para ello, el administrador de versiones crea una rama a partir de la maestra con este patrón de nomenclatura: `release/<microservice name>/<semver>`. Por ejemplo, `release/delivery/v1.0.2`.
De esta forma se desencadena una compilación de integración continua completa que ejecuta todos los pasos anteriores y, además, los siguientes:

1. Inserte la imagen de Docker en Azure Container Registry. La imagen se etiqueta con el número de versión tomado del nombre de la rama.
2. Ejecute `helm package` para empaquetar el gráfico de Helm
3. Inserte el paquete de Helm en Container Registry mediante la ejecución de `az acr helm push`.

Si esta compilación funciona correctamente, desencadena un proceso de implementación mediante una [canalización de versiones](/azure/devops/pipelines/release/what-is-release-management) de Azure Pipelines. Esta canalización

1. Ejecute `helm upgrade` para implementar el gráfico de Helm en un entorno de control de calidad.
1. Un aprobador cierra sesión antes de que el paquete pase a producción. Consulte [Release deployment control using approvals](/azure/devops/pipelines/release/approvals/approvals) (Liberación del control de implementación mediante aprobaciones).
1. Vuelva etiquetar la imagen de Docker para el espacio de nombres de producción en Azure Container Registry. Por ejemplo, si la etiqueta actual es `myrepo.azurecr.io/delivery:v1.0.2`, la etiqueta de producción es `reponame.azurecr.io/prod/delivery:v1.0.2`.
1. Ejecute `helm upgrade` para implementar el gráfico de Helm en un entorno de producción.

![Flujo de trabajo de CI/CD](./_images/aks-cicd-3.png)

Es importante recordar que incluso en los repositorios únicos, el ámbito de estas tareas pueden ser a los microservicios individuales, con el fin de que los equipos puedan realizar las implementaciones a gran velocidad. Hay varios pasos manuales en el proceso: la aprobación de solicitudes de inserción, la creación de ramas de la versión y la aprobación de implementaciones en el clúster de producción. Estos pasos son manuales debido a la directiva, pero pueden ser completamente automáticos si la organización lo prefiere.
