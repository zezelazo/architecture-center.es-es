---
title: Azure para profesionales de AWS
description: Comprenda los conceptos básicos de las cuentas, la plataforma y los servicios de Microsoft Azure. Conozca también las similitudes y diferencias principales entre las plataformas Azure y AWS. Aproveche su experiencia con AWS en Azure.
keywords: Expertos de AWS, comparación de Azure, comparación de AWS, diferencia entre azure y aws, azure y aws
author: lbrader
ms.date: 03/24/2017
pnp.series.title: Azure for AWS Professionals
ms.openlocfilehash: 04157b9a647779ae47ad0aff8132289a30544acf
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/28/2018
ms.locfileid: "47429645"
---
# <a name="azure-for-aws-professionals"></a>Azure para profesionales de AWS

Este artículo ayuda a los expertos de Amazon Web Services (AWS) a comprender los conceptos básicos de las cuentas, la plataforma y los servicios de Microsoft Azure. También se tratan las similitudes y diferencias principales entre las plataformas Azure y AWS.

Aprenderá a realizar los siguientes procedimientos:

* Cómo se organizan las cuentas y los recursos en Azure
* Cómo se estructuran las soluciones disponibles en Azure
* En qué se diferencian los servicios principales de Azure de los servicios de AWS

Azure y AWS crean sus funcionalidades de manera independiente en el tiempo, así que cada uno presenta importantes diferencias tanto en implementación como en diseño.

## <a name="overview"></a>Información general

Al igual que AWS, Microsoft Azure se crea en torno a un conjunto básico de servicios de proceso, almacenamiento, base de datos y red. En muchos casos, ambas plataformas ofrecen una equivalencia básica entre los productos y servicios que ofrecen. Tanto AWS como Azure permiten crear soluciones de alta disponibilidad basadas en hosts de Windows o Linux. Por lo tanto, si está acostumbrado a desarrollar con tecnología de Linux y OSS, ambas plataformas pueden servir.

Aunque las funcionalidades de ambas plataformas son similares, los recursos que proporcionan dichas funcionalidades con frecuencia se organizan de forma diferente. No siempre están claras las relaciones exactas uno a uno entre los servicios necesarios para crear una solución. También existen casos donde un determinado servicio puede ofrecerse en una plataforma, pero no en la otra. Consulte los [gráficos de servicios comparables de Azure y AWS](services.md).

## <a name="accounts-and-subscriptions"></a>Cuentas y suscripciones

Los servicios de Azure se pueden adquirir mediante varias opciones de precios, según el tamaño y las necesidades de su organización. Consulte la página de [información general de precios](https://azure.microsoft.com/pricing/) para más información.

[Las suscripciones de Azure](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-infrastructure-subscription-accounts-guidelines/) son una agrupación de recursos con un propietario asignado que es responsable de la facturación y de la administración de los permisos. A diferencia de AWS, donde los recursos creados con la cuenta de AWS están asociados a esa cuenta, las suscripciones existen de forma independiente de las cuentas de sus propietarios y pueden reasignarse a nuevos propietarios si es necesario.

![Comparación de la estructura y la propiedad de las cuentas de AWS y las suscripciones de Azure](./images/azure-aws-account-compare.png "Comparación de la estructura y la propiedad de las cuentas de AWS y las suscripciones de Azure")
<br/>*Comparación de la estructura y la propiedad de las cuentas de AWS y las suscripciones de Azure*
<br/><br/>

Las suscripciones se asignan a tres tipos de cuentas de administrador:

-   **Administrador de cuenta**: el propietario de la suscripción y la cuenta a la que se facturan los recursos usados en dicha suscripción. El administrador de cuenta solo puede cambiarse mediante la transferencia de propiedad de la suscripción.

-   **Administrador de servicios**: esta cuenta tiene derechos para crear y administrar recursos de la suscripción, pero no es responsable de la facturación. De forma predeterminada, el administrador de cuenta y el administrador de servicios se asignan a la misma cuenta. El administrador de cuenta puede asignar un usuario distinto a la cuenta del administrador de servicios para administrar los aspectos técnicos y operativos de una suscripción. Solo hay un administrador de servicios por suscripción.

-   **Coadministrador**: puede haber varias cuentas de coadministrador asignadas a una suscripción. Los coadministradores no pueden cambiar el administrador de servicios, pero, por lo demás, tienen control total sobre los recursos de la suscripción y los usuarios.

Por debajo del usuario de nivel de suscripción también se pueden asignar roles y permisos individuales a recursos específicos, de manera parecida a como se conceden los permisos a usuarios y grupos de IAM en AWS. En Azure todas las cuentas de usuario están asociadas con una cuenta de Microsoft o una cuenta profesional (una cuenta administrada mediante Azure Active Directory).

Al igual que las cuentas de AWS, las suscripciones tienen límites y cuotas de servicio predeterminados. Para ver una lista detallada de estos límites, consulte [Límites, cuotas y restricciones de suscripción y servicios de Microsoft Azure](https://azure.microsoft.com/documentation/articles/azure-subscription-service-limits/).
Estos límites se pueden aumentar hasta el máximo mediante la [presentación de una solicitud de soporte técnico en el portal de administración](https://blogs.msdn.microsoft.com/girishp/2015/09/20/increasing-core-quota-limits-in-azure/).

### <a name="see-also"></a>Otras referencias

-   [Adición o cambio de roles de administrador de Azure](https://azure.microsoft.com/documentation/articles/billing-add-change-azure-subscription-administrator/)

-   [Cómo descargar las datos de uso diario y de factura de facturación de Azure](https://azure.microsoft.com/documentation/articles/billing-download-azure-invoice-daily-usage-date/)

## <a name="resource-management"></a>Administración de recursos

El término "recurso" en Azure se usa de la misma manera que en AWS, con el significado de cualquier instancia de proceso, objeto de almacenamiento, dispositivo de red u otra entidad que se puede crear o configurar dentro de la plataforma.

Los recursos de Azure se implementan y administran mediante dos modelos: [Azure Resource Manager](/azure/azure-resource-manager/resource-group-overview) o el [modelo de implementación clásica](/azure/azure-resource-manager/resource-manager-deployment-model) de Azure más antiguo.
Los nuevos recursos se crean mediante el modelo de Resource Manager.

### <a name="resource-groups"></a>Grupos de recursos

Azure y AWS tienen entidades llamadas "grupos de recursos" que organizan los recursos, como máquinas virtuales, almacenamiento y dispositivos de red virtuales. Sin embargo, los [grupos de recursos de Azure](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-infrastructure-resource-groups-guidelines/) no son equiparables directamente a los grupos de recursos de AWS.

Mientras que AWS permite que un recurso se etiquete en varios grupos de recursos, un recurso de Azure siempre está asociado a un grupo de recursos. Un recurso creado en un grupo de recursos se puede mover a otro grupo, pero solo puede estar en un grupo de recursos a la vez. Los grupos de recursos son la agrupación fundamental que se usa en Azure Resource Manager.

Los recursos también se pueden organizar mediante [etiquetas](https://azure.microsoft.com/documentation/articles/resource-group-using-tags/).
Las etiquetas son pares de clave-valor que le permiten agrupar los recursos de su suscripción con independencia de la pertenencia al grupo de recursos.

### <a name="management-interfaces"></a>Interfaces de administración

Azure ofrece varias maneras de administrar los recursos:

-   [Interfaz web](https://azure.microsoft.com/documentation/articles/resource-group-portal/).
    Al igual que el panel de AWS, Azure Portal proporciona una interfaz de administración completa basada en web para los recursos de Azure.

-   [API DE REST](https://azure.microsoft.com/documentation/articles/resource-manager-rest-api/).
    La API de REST de Azure Resource Manager proporciona acceso mediante programación a la mayoría de las características disponibles en Azure Portal.

-   [Línea de comandos](https://azure.microsoft.com/documentation/articles/xplat-cli-azure-resource-manager/).
    La herramienta CLI de Azure 2.0 proporciona una interfaz de la línea de comandos capaz de crear y administrar recursos de Azure. La CLI de Azure está disponible para [Windows, Linux y Mac OS](https://aka.ms/azurecli2).

-   [PowerShell](https://azure.microsoft.com/documentation/articles/powershell-azure-resource-manager/).
    Los módulos de Azure para PowerShell permiten ejecutar tareas de administración automatizadas mediante un script. PowerShell está disponible para [Windows, Linux y Mac OS](https://github.com/PowerShell/PowerShell).

-   [Plantillas](https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/).
    Las plantillas de Azure Resource Manager proporcionan funcionalidades de administración de recursos basadas en plantillas JSON que son similares al servicio AWS CloudFormation.

En cada una de estas interfaces, el grupo de recursos ocupa un lugar central en la creación, la implementación o la modificación de los recursos de Azure. Su función es similar a la que desempeña una "pila" en la agrupación de recursos de AWS durante las implementaciones de CloudFormation.

La sintaxis y la estructura de estas interfaces son diferentes a las de sus equivalentes de AWS, pero proporcionan funcionalidades comparables. Además, muchas herramientas de administración de terceros usadas en AWS, como [Terraform del Hashicorp](https://www.terraform.io/docs/providers/azurerm/) y [Netflix Spinnaker](https://www.spinnaker.io/), están también disponibles en Azure.

### <a name="see-also"></a>Otras referencias

-   [Directrices para el grupo de recursos de Azure](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-infrastructure-resource-groups-guidelines/)

## <a name="regions-and-zones-high-availability"></a>Regiones y zonas (alta disponibilidad)

El alcance de los errores puede variar. Algunos errores de hardware, como un problema en un disco, pueden afectar a un único equipo host. Un error en un conmutador de red podría afectar a todo un bastidor del servidor. Menos frecuentes son los errores que afectan a todo un centro de datos, como los problemas de alimentación. Aún más improbables son los problemas por los que toda una región dejaría de estar disponible.

Uno de los mecanismos para conseguir que una aplicación sea resistente es la redundancia. Sin embargo, esta redundancia debe planearse al diseñar la aplicación. Además, el nivel de redundancia necesario dependerá de los requisitos de la compañía: no todas las aplicaciones necesitan tener redundancia entre regiones para protegerse contra una interrupción regional. En general, hay que buscar el equilibrio, ya que una mayor redundancia y confiabilidad implica una mayor complejidad y unos costos más elevados.  

En AWS, una región se divide en dos o más zonas de disponibilidad. Una zona de disponibilidad se corresponde con un centro de datos físicamente aislado en la región geográfica. Azure dispone de varias características para hacer que una aplicación sea redundante en todos los niveles de error. Entre estas características cabe destacar los **conjuntos de disponibilidad**, las **zonas de disponibilidad** y las **regiones emparejadas**. 

![](../resiliency/images/redundancy.svg)

En la tabla siguiente se resume cada opción.

| &nbsp; | Conjunto de disponibilidad | Zona de disponibilidad | Región emparejada |
|--------|------------------|-------------------|---------------|
| Alcance del error | Bastidor | Centro de datos | Region |
| Enrutamiento de solicitudes | Load Balancer | Equilibrador de carga entre zonas | Traffic Manager |
| Latencia de red | Muy baja | Bajo | Media-alta |
| Redes virtuales  | VNet | VNet | Emparejamiento de VNet entre regiones |

### <a name="availability-sets"></a>Conjuntos de disponibilidad 

Para protegerse frente a errores de hardware localizados, como un error en un conmutador de red o un disco, implemente dos o más máquinas virtuales en un conjunto de disponibilidad. Un conjunto de disponibilidad se compone de dos o más *dominios de error* que comparten una fuente de alimentación y un conmutador de red. Las máquinas virtuales incluidas en un conjunto de disponibilidad se distribuyen entre los dominios de error, por lo que, si un error de hardware afecta a un dominio de error, el tráfico de la red puede enrutarse a las máquinas virtuales de otros dominios de error. Para más información acerca de los conjuntos disponibilidad, consulte [Administración de la disponibilidad de las máquinas virtuales Windows en Azure](/azure/virtual-machines/windows/manage-availability).

Cuando se agregan instancias de máquina virtual a conjuntos de disponibilidad, también se asignan a un [dominio de actualización](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-manage-availability/). Un dominio de actualización es un grupo de máquinas virtuales que están configuradas para eventos de mantenimiento planeado al mismo tiempo. Distribuir máquinas virtuales entre varios dominios de actualización garantiza que, en cualquier momento dado, los eventos de actualización planeada y aplicación de revisiones solo afectan a un subconjunto de estas máquinas virtuales.

Los conjuntos de disponibilidad se deben organizar mediante el rol de la instancia de la aplicación para garantizar que una instancia de cada rol esté operativa. Por ejemplo, en una aplicación web de tres niveles, puede crear conjuntos de disponibilidad diferentes para los niveles de front-end, aplicación y datos.

![Conjuntos de disponibilidad de Azure para cada rol de aplicación](./images/three-tier-example.png "Conjuntos de disponibilidad para cada rol de aplicación")

### <a name="availability-zones"></a>Zonas de disponibilidad

Una [zona de disponibilidad](/azure/availability-zones/az-overview) es una zona separada físicamente dentro de una región de Azure. Cada zona de disponibilidad tiene una fuente de alimentación, una red y un sistema de refrigeración distintos. Cuando las máquinas virtuales están implementadas en diferentes zonas de disponibilidad, es más fácil proteger una aplicación frente a errores que afectan a todo el centro de datos. 

### <a name="paired-regions"></a>Regiones emparejadas

Para proteger una aplicación frente a una interrupción regional, puede implementar la aplicación en varias regiones y utilizar [Azure Traffic Manager][traffic-manager] para distribuir el tráfico de Internet en las distintas regiones. Cada región de Azure está emparejada con otra región. Juntas, forman un [par regional][paired-regions]. A excepción del Sur de Brasil, los pares regionales se encuentran en la misma ubicación geográfica para, de este modo, cumplir los requisitos de residencia de datos a efectos de jurisdicción fiscal y aplicación de las leyes.

A diferencia de las zonas de disponibilidad, que están físicamente alejadas de los centros de datos pero que pueden estar relativamente cerca de áreas geográficas, las regiones emparejadas suelen estar alejadas unas de otras 500 km como mínimo. La finalidad de ello es asegurarse de que los desastres a escala más grande solo afecten a una de las regiones del par. Los pares vecinos pueden establecerse para sincronizar los datos del servicio de base de datos y almacenamiento, y están configurados de tal modo que las actualizaciones de la plataforma solo se implementan en una región del par a la vez.

El [almacenamiento con redundancia geográfica](https://azure.microsoft.com/documentation/articles/storage-redundancy/#geo-redundant-storage) de Azure se copia automáticamente en la región emparejada apropiada. En el caso de todos los demás recursos, la creación de una solución completamente redundante mediante regiones emparejadas implica la creación de una copia completa de la solución en ambas regiones.


### <a name="see-also"></a>Otras referencias

-   [Regiones y disponibilidad de máquinas virtuales en Azure](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-regions-and-availability/)

-   [Alta disponibilidad para aplicaciones de Azure](../resiliency/high-availability-azure-applications.md)

-   [Disaster recovery for Azure applications](../resiliency/disaster-recovery-azure-applications.md) (Recuperación ante desastres para aplicaciones de Azure)

-   [Mantenimiento planeado de máquinas virtuales Linux en Azure](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-planned-maintenance/)

## <a name="services"></a>Services

Para obtener una lista de cómo se asignan servicios entre plataformas, consulte [Comparación de AWS con los servicios de Azure](./services.md).

No todos los servicios y productos de Azure están disponibles en todas las regiones. Consulte la página [Productos por región](https://azure.microsoft.com/regions/services/) para más información. Las garantías de tiempo de actividad y las directivas de crédito de tiempo de inactividad de cada producto o servicio de Azure pueden encontrarse en la página [Acuerdos de Nivel de Servicio](https://azure.microsoft.com/support/legal/sla/).

En las siguientes secciones se proporciona una breve explicación de las diferencias de las características y los servicios más usados entre las plataformas de Azure y AWS.

### <a name="compute-services"></a>Compute Services

#### <a name="ec2-instances-and-azure-virtual-machines"></a>Instancias de EC2 y máquinas virtuales de Azure

Aunque los tipos de instancia de AWS y los tamaños de máquina virtual de Azure tienen una composición similar, existen diferencias en las funcionalidades de memoria RAM, CPU y almacenamiento.

-   [Tipos de instancia de Amazon EC2](https://aws.amazon.com/ec2/instance-types/)

-   [Tamaños de las máquinas virtuales en Azure (Windows)](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-sizes/)

-   [Tamaños de las máquinas virtuales en Azure (Linux)](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-sizes/)

A diferencia de la facturación por segundo de AWS, las máquinas virtuales de Azure a petición se facturan por minuto.

#### <a name="ebs-and-azure-storage-for-vm-disks"></a>EBS y Azure Storage para discos de máquina virtual

Los [discos de datos](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-about-disks-vhds/) que residen en Blob Storage proporcionan almacenamiento de datos duradero para las máquinas virtuales de Azure. Este sistema es parecido al que usan las instancias de EC2 para almacenar volúmenes de disco en Elastic Block Store (EBS). El [almacenamiento temporal de Azure](https://blogs.msdn.microsoft.com/mast/2013/12/06/understanding-the-temporary-drive-on-windows-azure-virtual-machines/) también proporciona máquinas virtuales con el mismo almacenamiento de lectura y escritura temporal de baja latencia que el almacenamiento de instancias de EC2 (también llamado almacenamiento efímero).

Se admite E/S de disco de más alto rendimiento mediante [Azure Premium Storage](https://docs.microsoft.com/azure/storage/storage-premium-storage).
Esta funcionalidad es similar a las opciones de almacenamiento IOPS aprovisionado que se proporcionan con AWS.

#### <a name="lambda-azure-functions-azure-web-jobs-and-azure-logic-apps"></a>Lambda, Azure Functions, Azure Web Jobs y Azure Logic Apps

[Azure Functions](https://azure.microsoft.com/services/functions/) es el principal equivalente de AWS Lambda al proporcionar código a petición sin servidor.
Sin embargo, la funcionalidad de Lambda también se solapa con otros servicios de Azure:

-   [WebJobs](https://azure.microsoft.com/documentation/articles/web-sites-create-web-jobs/): le permite crear tareas en segundo plano con ejecución programada o continua.

-   [Logic Apps](https://azure.microsoft.com/services/logic-apps/): proporciona servicios de comunicación, integración y administración de reglas empresariales.

#### <a name="autoscaling-azure-vm-scaling-and-azure-app-service-autoscale"></a>Escalado automático, escalado de máquina virtual de Azure y escalado automático de Azure App Service

El escalado automático en Azure se controla mediante dos servicios:

-   [VM Scale Sets](https://azure.microsoft.com/documentation/articles/virtual-machine-scale-sets-overview/): permite implementar y administrar un conjunto idéntico de máquinas virtuales. El número de instancias se puede escalar automáticamente en función de las necesidades de rendimiento.

-   [Escalado automático de App Service](https://azure.microsoft.com/documentation/articles/web-sites-scale/): ofrece la funcionalidad para escalar automáticamente soluciones de Azure App Service.


#### <a name="container-service"></a>Container Service
[Azure Container Service](https://docs.microsoft.com/azure/container-service/container-service-intro): admite contenedores de Docker que se administran mediante Docker Swarm, Kubernetes o DC/OS.

#### <a name="other-compute-services"></a>Otros servicios de proceso 


Azure ofrece varios servicios de proceso que no tienen equivalentes directos en AWS:

-   [Azure Batch](https://azure.microsoft.com/documentation/articles/batch-technical-overview/): permite administrar trabajo de procesos intensivos en una colección escalable de máquinas virtuales.

-   [Service Fabric](https://azure.microsoft.com/documentation/articles/service-fabric-overview/): plataforma para el desarrollo y el hospedaje de soluciones de [microservicios](https://azure.microsoft.com/documentation/articles/service-fabric-overview-microservices/) escalables.

#### <a name="see-also"></a>Otras referencias

-   [Creación de una máquina virtual con Linux en Azure mediante el Portal](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-quick-create-portal/)

-   [Arquitectura de referencia de Azure: ejecución de una máquina virtual Linux en Azure](https://azure.microsoft.com/documentation/articles/guidance-compute-single-vm-linux/)

-   [Introducción a las aplicaciones web Node.js en Azure App Service](https://azure.microsoft.com/documentation/articles/app-service-web-nodejs-get-started/)

-   [Arquitectura de referencia de Azure: aplicación web básica](https://azure.microsoft.com/documentation/articles/guidance-web-apps-basic/)

-   [Creación de su primera función de Azure](https://azure.microsoft.com/documentation/articles/functions-create-first-azure-function/)

### <a name="storage"></a>Storage

#### <a name="s3ebsefs-and-azure-storage"></a>S3/EBS/EFS y Azure Storage

En la plataforma AWS, el almacenamiento en la nube se compone de tres servicios:

-   **Simple Storage Service (S3)**: almacenamiento básico de objetos. Los datos se ponen a disposición de los usuarios mediante una API accesible desde Internet.

-   **Elastic Block Storage (EBS)**: almacenamiento de nivel de bloque, destinado al acceso por una única máquina virtual.

-   **Elastic File System (EFS)**: almacenamiento de archivos para su uso como almacenamiento con particiones para hasta miles de instancias de EC2.

En Azure Storage, las [cuentas de almacenamiento](https://azure.microsoft.com/documentation/articles/storage-create-storage-account/) enlazadas a la suscripción le permiten crear y administrar los siguientes servicios de almacenamiento:

-   [Blob Storage](https://azure.microsoft.com/documentation/articles/storage-create-storage-account/): almacena cualquier tipo de dato de texto o binario, como un documento, un archivo multimedia o un instalador de aplicaciones. Se puede configurar para el acceso privado o compartir el contenido públicamente en Internet. Blob Storage tiene la misma finalidad que AWS S3 y EBS.
-   [Table Storage](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-table-storage/): almacena conjuntos de datos estructurados. Se trata de un almacén de datos de clave-atributo NoSQL, que permite el desarrollo rápido de grandes cantidades de datos y el acceso inmediato a ellos. Es parecido a los servicios SimpleDB y DynamoDB de AWS.

-   [Queue Storage](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-queues/): ofrece una solución de mensajería para el procesamiento de flujos de trabajo y para la comunicación entre los componentes de los servicios en la nube.

-   [File Storage](https://azure.microsoft.com/documentation/articles/storage-java-how-to-use-file-storage/): ofrece almacenamiento compartido para aplicaciones heredadas que usan el protocolo de bloque de mensajes del servidor (SMB). File Storage se usa de manera parecida a EFS en la plataforma AWS.
 
#### <a name="glacier-and-azure-storage"></a>Glacier y Azure Storage 

[Azure Archive Blob Storage](/azure/storage/blobs/storage-blob-storage-tiers#archive-access-tier) es comparable al servicio de almacenamiento AWS Glaciar. Está diseñado para los datos con acceso infrecuente que se almacenan durante 180 días, como mínimo, y pueden tolerar varias horas de latencia de recuperación. 

Para los datos a los que se accede con poca frecuencia, pero deben estar disponibles inmediatamente entonces, la [capa de Azure Blog Storage de acceso esporádico](/azure/storage/blobs/storage-blob-storage-tiers#cool-access-tier) proporciona un almacenamiento más económico que el de blob estándar. Este nivel de almacenamiento es comparable al servicio de almacenamiento de acceso infrecuente de AWS S3.

#### <a name="see-also"></a>Otras referencias

-   [Lista de comprobación de rendimiento y escalabilidad de Microsoft Azure Storage](https://azure.microsoft.com/documentation/articles/storage-performance-checklist/)

-   [Guía de seguridad de Azure Storage](https://azure.microsoft.com/documentation/articles/storage-security-guide/)

-   [Patrones y procedimientos: guía de Content Delivery Network (CDN)](https://azure.microsoft.com/documentation/articles/best-practices-cdn/)

### <a name="networking"></a>Redes

#### <a name="elastic-load-balancing-azure-load-balancer-and-azure-application-gateway"></a>Elastic Load Balancing, Azure Load Balancer y Azure Application Gateway

Los equivalentes de Azure de los dos servicios Elastic Load Balancing son:

-   [Load Balancer](https://azure.microsoft.com/documentation/articles/load-balancer-overview/): proporciona las mismas funcionalidades que AWS Classic Load Balancer, lo que permite distribuir el tráfico de varias máquinas virtuales en el nivel de red. También proporciona funcionalidad de conmutación por error.

-   [Application Gateway](https://azure.microsoft.com/documentation/articles/application-gateway-introduction/): ofrece enrutamiento basado en reglas de nivel de aplicación comparable a AWS Application Load Balancer.

#### <a name="route-53-azure-dns-and-azure-traffic-manager"></a>Route 53, Azure DNS y Azure Traffic Manager

En AWS, Route 53 proporciona servicios de administración de nombres DNS, enrutamiento del tráfico y conmutación por error. En Azure, estas funcionalidades se realizan mediante dos servicios:

-   [Azure DNS](https://azure.microsoft.com/documentation/services/dns/): proporciona administración de dominios y DNS.

-   [Traffic Manager][traffic-manager] proporciona funcionalidades de enrutamiento de tráfico de nivel DNS, equilibrio de carga y conmutación por error.

#### <a name="direct-connect-and-azure-expressroute"></a>Direct Connect y Azure ExpressRoute

Azure proporciona conexiones dedicadas de sitio a sitio parecidas mediante su servicio [ExpressRoute](https://azure.microsoft.com/documentation/services/expressroute/). ExpressRoute le permite conectar su red local directamente a recursos de Azure mediante una conexión de red privada dedicada. Azure también ofrece [conexiones VPN de sitio a sitio](https://azure.microsoft.com/documentation/articles/vpn-gateway-howto-site-to-site-resource-manager-portal/) más convencionales a un menor coste.

#### <a name="see-also"></a>Otras referencias

-   [Creación de una red virtual mediante Azure Portal](https://azure.microsoft.com/documentation/articles/virtual-networks-create-vnet-arm-pportal/)

-   [Planeamiento y diseño de Azure Virtual Networks](https://azure.microsoft.com/documentation/articles/virtual-network-vnet-plan-design-arm/)

-   [Procedimientos recomendados de Azure Network Security](https://azure.microsoft.com/documentation/articles/azure-security-network-security-best-practices/)

### <a name="database-services"></a>Servicios de base de datos

#### <a name="rds-and-azure-relational-database-services"></a>RDS y servicios de base de datos relacionales de Azure

Azure proporciona varios servicios de bases de datos relacionales diferentes equivalentes al servicio de base de datos relacional (RDS) de AWS.

-   [SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-technical-overview)
-   [Azure Database for MySQL](https://docs.microsoft.com/azure/mysql/overview)
-   [Azure Database para PostgreSQL](https://docs.microsoft.com/azure/postgresql/overview)

Se pueden implementar otros motores de base de datos, como [SQL Server](https://azure.microsoft.com/services/virtual-machines/sql-server/), [Oracle](https://azure.microsoft.com/campaigns/oracle/) y [MySQL](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-classic-mysql-2008r2/) mediante instancias de máquina virtual de Azure.

Los costes de AWS RDS vienen determinados por la cantidad de recursos de hardware que utiliza la instancia, como CPU, RAM, almacenamiento y ancho de banda de red. En los servicios de base de datos de Azure, el costo depende del tamaño de la base de datos, del número de conexiones simultáneas y de los niveles de rendimiento.

#### <a name="see-also"></a>Otras referencias

-   [Tutoriales de Azure SQL Database](https://azure.microsoft.com/documentation/articles/sql-database-explore-tutorials/)

-   [Configuración de replicación geográfica activa para Azure SQL Database en Azure Portal](https://azure.microsoft.com/documentation/articles/sql-database-geo-replication-portal/)

-   [Introducción a Azure Cosmos DB: una base de datos JSON NoSQL](/azure/cosmos-db/sql-api-introduction)

-   [Uso de Azure Table Storage en Node.js](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-table-storage/)

### <a name="security-and-identity"></a>Seguridad e identidad

#### <a name="directory-service-and-azure-active-directory"></a>Servicio de directorio y Azure Active Directory

Azure divide los servicios de directorio en las ofertas siguientes:

-   [Azure Active Directory](https://azure.microsoft.com/documentation/articles/active-directory-whatis/): servicio de administración de identidades y directorios basado en la nube.

-   [Azure Active Directory B2B](https://azure.microsoft.com/documentation/articles/active-directory-b2b-collaboration-overview/): permite el acceso a las aplicaciones corporativas desde identidades administradas de asociados.

-   [Azure Active Directory B2C](https://azure.microsoft.com/documentation/articles/active-directory-b2c-overview/): compatibilidad de la oferta de servicio con el inicio de sesión único y la administración de usuarios para aplicaciones orientadas a los consumidores.

-   [Azure Active Directory Domain Services](https://azure.microsoft.com/documentation/articles/active-directory-ds-overview/): servicio de controlador de dominio hospedado, que permite la funcionalidad de administración de usuarios y la unión a dominios compatibles de Active Directory.

#### <a name="web-application-firewall"></a>Firewall de aplicaciones web

Además del [firewall de aplicaciones web de Application Gateway](https://azure.microsoft.com/documentation/articles/application-gateway-webapplicationfirewall-overview/), también puede [usar firewalls de aplicación web](https://azure.microsoft.com/documentation/articles/application-gateway-webapplicationfirewall-overview/) de proveedores de terceros como [Barracuda Networks](https://azure.microsoft.com/marketplace/partners/barracudanetworks/waf/).

#### <a name="see-also"></a>Otras referencias

-   [Introducción a la seguridad de Microsoft Azure](https://azure.microsoft.com/documentation/articles/azure-security-getting-started/)

-   [Procedimientos recomendados para la administración de identidades y la seguridad del control de acceso en Azure](https://azure.microsoft.com/documentation/articles/azure-security-identity-management-best-practices/)

### <a name="application-and-messaging-services"></a>Servicios de aplicación y mensajería

#### <a name="simple-email-service"></a>Simple Email Service

AWS proporciona Simple Email Service (SES) para el envío de correos electrónicos de notificaciones, transacciones o marketing. En Azure, soluciones de terceros como [Sendgrid](https://sendgrid.com/partners/azure/) proporcionan servicios de correo electrónico.

#### <a name="simple-queueing-service"></a>Simple Queueing Service

AWS Simple Queueing Service (SQS) proporciona un sistema de mensajería para la conexión de aplicaciones, servicios y dispositivos dentro de la plataforma de AWS. Azure cuenta con dos servicios que proporcionan una funcionalidad parecida:

-   [Queue Storage](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-queues/): un servicio de mensajería en la nube que permite la comunicación entre los componentes de las aplicaciones dentro de la plataforma de Azure.

-   [Service Bus](https://azure.microsoft.com/services/service-bus/): un sistema de mensajería más sólido para la conexión de aplicaciones, servicios y dispositivos. Mediante el componente relacionado [Service Bus Relay](https://docs.microsoft.com/azure/service-bus-relay/relay-what-is-it), Service Bus también puede conectarse a aplicaciones y servicios hospedados de manera remota.

#### <a name="device-farm"></a>Device Farm

AWS Device Farm proporciona servicios de prueba de dispositivos. En Azure, [Xamarin Test Cloud](https://www.xamarin.com/test-cloud) ofrece pruebas front-end similares entre dispositivos para dispositivos móviles.

Además de pruebas front-end, [Azure DevTest Labs](https://azure.microsoft.com/services/devtest-lab/) proporciona recursos de prueba back-end para los entornos Linux y Windows.

#### <a name="see-also"></a>Otras referencias

-   [Uso del almacenamiento de colas de Node.js](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-queues/)

-   [Utilización de las colas de Service Bus](https://azure.microsoft.com/documentation/articles/service-bus-nodejs-how-to-use-queues/)

### <a name="analytics-and-big-data"></a>Análisis y macrodatos

[Cortana Intelligence Suite](https://azure.microsoft.com/suites/cortana-intelligence-suite/) es un paquete de productos y servicios de Azure diseñados para capturar, organizar, analizar y visualizar grandes cantidades de datos. El conjunto de aplicaciones de Cortana consta de los siguientes servicios:

-   [HDInsight](https://azure.microsoft.com/documentation/services/hdinsight/): distribución de Apache administrada que incluye Hadoop, Spark, Storm o HBase.

-   [Data Factory](https://azure.microsoft.com/documentation/services/data-factory/): proporciona funcionalidad de orquestación y canalización de datos.

-   [SQL Data Warehouse](https://azure.microsoft.com/documentation/services/sql-data-warehouse/): almacenamiento de datos relacionales a gran escala.

-   [Data Lake Store](https://azure.microsoft.com/documentation/services/data-lake-store/): almacenamiento a gran escala optimizado para cargas de trabajo de análisis de macrodatos.

-   [Machine Learning](https://azure.microsoft.com/documentation/services/machine-learning/): se usa para crear y aplicar análisis predictivos sobre los datos.

-   [Stream Analytics](https://azure.microsoft.com/documentation/services/stream-analytics/): análisis de los datos en tiempo real.

-   [Data Lake Analytics](https://azure.microsoft.com/documentation/articles/data-lake-analytics-overview/): servicio de análisis a gran escala para trabajar con Data Lake Store.

-   [PowerBI](https://powerbi.microsoft.com/): se usa para la visualización de datos de energía.

#### <a name="see-also"></a>Otras referencias

-   [Galería de Cortana Intelligence](https://gallery.cortanaintelligence.com/)

-   [Descripción de las soluciones de macrodatos de Microsoft](https://msdn.microsoft.com/library/dn749804.aspx)

-   [Blog de Azure Data Lake y Azure HDInsight](https://blogs.msdn.microsoft.com/azuredatalake/)

### <a name="internet-of-things"></a>Internet de las cosas

#### <a name="see-also"></a>Otras referencias

-   [Introducción a Azure IoT Hub](https://azure.microsoft.com/documentation/articles/iot-hub-csharp-csharp-getstarted/)

-   [Comparación de IoT Hub y Event Hubs](https://azure.microsoft.com/documentation/articles/iot-hub-compare-event-hubs/)

### <a name="mobile-services"></a>Servicios móviles

#### <a name="notifications"></a>Notificaciones

Notification Hubs no admite el envío de mensajes de correo electrónico o SMS, por lo que se requieren servicios de terceros para esos tipos de entrega.

#### <a name="see-also"></a>Otras referencias

-   [Creación de una aplicación de Android](https://azure.microsoft.com/documentation/articles/app-service-mobile-android-get-started/)

-   [Autenticación y autorización en Azure Mobile Apps](https://azure.microsoft.com/documentation/articles/app-service-mobile-auth/)

-   [Envío de notificaciones push a Android con Microsoft Azure Notification Hubs](https://azure.microsoft.com/documentation/articles/notification-hubs-android-push-notification-google-fcm-get-started/)

### <a name="management-and-monitoring"></a>Administración y supervisión

#### <a name="see-also"></a>Otras referencias
-   [Guía de supervisión y diagnósticos](https://azure.microsoft.com/documentation/articles/best-practices-monitoring/)

-   [Procedimientos recomendados para crear plantillas de Azure Resource Manager](https://azure.microsoft.com/documentation/articles/resource-manager-template-best-practices/)

-   [Plantillas de inicio rápido de Azure Resource Manager](https://azure.microsoft.com/documentation/templates/)


## <a name="next-steps"></a>Pasos siguientes

-   [Comience a usar Azure](https://azure.microsoft.com/get-started/)

-   [Arquitecturas de soluciones de Azure](https://azure.microsoft.com/solutions/architecture/)

-   [Arquitecturas de referencia de Azure](https://azure.microsoft.com/documentation/articles/guidance-architecture/)


<!-- links -->

[paired-regions]: https://azure.microsoft.com/documentation/articles/best-practices-availability-paired-regions/
[traffic-manager]: /azure/traffic-manager/
