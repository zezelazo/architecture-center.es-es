---
title: 'Guía técnica: recuperación ante errores locales en Azure'
description: Artículo para entender y diseñar aplicaciones resistentes, con alta disponibilidad y con tolerancia a errores, así como para planear la recuperación ante desastres centrada en errores locales dentro de Azure.
author: adamglick
ms.date: 08/18/2016
ms.openlocfilehash: 5fc929bd1affe3dd6616f908bae0e7d2fefb89d5
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2018
ms.locfileid: "30846841"
---
[!INCLUDE [header](../_includes/header.md)]

# <a name="azure-resiliency-technical-guidance-recovery-from-local-failures-in-azure"></a>Guía técnica sobre resistencia en Azure: recuperación ante errores locales en Azure

Hay dos amenazas principales para la disponibilidad de las aplicaciones:

* El error de los dispositivos, como las unidades de disco y los servidores
* El agotamiento de los recursos críticos, como el proceso en condiciones de carga máxima

Azure ofrece una combinación de administración de recursos, elasticidad, equilibrio de carga y creación de particiones que habilita la alta disponibilidad en estas circunstancias. Algunas de estas características se realizan automáticamente para todos los servicios de Azure. Sin embargo, en algunos casos, el desarrollador de aplicaciones debe realizar un trabajo adicional para beneficiarse de ellas.

## <a name="cloud-services"></a>Cloud Services
Azure Cloud Services consta de varias colecciones de uno o varios roles de trabajo o web. Se pueden ejecutar una o varias instancias de un rol simultáneamente. La configuración determina el número de instancias. Las instancias de rol se supervisan y administran mediante un componente denominado controlador de tejido. El controlador de tejido detecta y responde automáticamente ante los errores de software y de hardware.

Todas las instancias de rol se ejecutan en su propia máquina virtual (VM) y se comunica con su controlador de tejido a través de un agente invitado. El agente invitado recopila métricas de nodos y de recursos, entre las que se incluye el uso, el estado, los registros, el uso de recursos, las excepciones y las condiciones de error de la máquina virtual. El controlador de tejido consulta al agente invitado a intervalos configurables y reinicia la máquina virtual si el agente invitado no responde. En caso de error de hardware, el controlador de tejido asociado mueve todas las instancias de rol afectadas a un nuevo nodo de hardware y vuelve a configurar la red para enrutar el tráfico que hay allí.

Para beneficiarse de estas características, los desarrolladores deben garantizar que todos los roles de servicio impiden almacenar el estado en las instancias de rol. En su lugar, se debe acceder a todos los datos persistentes desde un almacenamiento duradero, como Azure Storage o Azure SQL Database. Esto permite que cualquier rol controle las solicitudes. También significa que las instancias de rol pueden dejar de funcionar en un momento dado sin que ello genere incoherencias en el estado transitorio o persistente del servicio.

El requisito para almacenar el estado externamente en los roles tiene varias implicaciones. Por ejemplo, implica que todos los cambios relacionados con una tabla de Azure Storage deben realizarse, si es posible, en una única transacción de grupos de entidades. Por supuesto, no siempre es posible realizar todos los cambios en una sola transacción. Debe tener especial cuidado de asegurarse de que los errores de instancias de rol no causan problemas cuando interrumpan operaciones de ejecución prolongada que abarcan dos o más actualizaciones en el estado persistente del servicio. Si otro rol vuelve a intentar esta operación, debe anticiparse y controlar aquella situación en la que el trabajo se completó parcialmente.

Por ejemplo, considere un servicio que crea particiones de datos en varios almacenes. Si un rol de trabajo deja de funcionar mientras se reubica una partición, es posible que dicha reubicación no finalice. O bien, la reubicación se pueden repetir desde su concepción por parte de otro un rol de trabajo, lo que puede causar datos huérfanos o daños en los datos. Para evitar problemas, las operaciones de ejecución prolongada deben ser:

* *Idempotente*: repetibles sin efectos secundarios. Para ser idempotente, una operación de ejecución prolongada debe tener el mismo efecto, independientemente de cuántas veces se ejecuta, incluso si la ejecución se interrumpe.
* *Reiniciable incrementalmente*: capaces de continuar desde el último punto de error. Para ser reiniciable incrementalmente, una operación de ejecución prolongada debe constar de una secuencia de operaciones atómicas menores. También debe registrar su progreso en un almacenamiento duradero, con el fin de que cada invocación posterior comience donde se detuvo su predecesora.

Por último, todas las operaciones de ejecución prolongada se deben invocar varias veces hasta que se realicen correctamente. Por ejemplo, un rol de trabajo puede colocar una operación de aprovisionamiento en una cola de Azure y, luego, quitarla de la cola solo cuando la operación se haya ejecutado correctamente. Es posible que la recolección de elementos no utilizados sea necesaria para limpiar los datos que interrumpieron la creación de operaciones.

### <a name="elasticity"></a>Elasticidad
El número inicial de instancias que se ejecutan para cada rol se determina en la configuración del mismo. Los administradores deben configurar inicialmente cada uno de los roles para que se ejecute con dos o más instancias según la carga esperada. Pero las instancias de rol pueden escalarse fácilmente de forma vertical u horizontal a medida que cambian los patrones de uso. Esto se puede hacer manualmente en el Portal de Azure, o bien se puede automatizar el proceso con Windows PowerShell, Service Management API o herramientas de terceros. Para más información, consulte [Cómo escalar automáticamente un servicio en la nube](/azure/cloud-services/cloud-services-how-to-scale/).

### <a name="partitioning"></a>Creación de particiones
El controlador de tejido de Azure usa dos tipos de particiones:

* Se usa un *dominio de actualización* para actualizar las instancias de rol de un servicio en grupos. Azure implementa las instancias de servicio en varios dominios de actualización. Para realizar una actualización en la ubicación, el controlador de tejido desactiva todas las instancias de un dominio de actualización, las actualiza y, después, las reinicia antes de pasar al siguiente dominio de actualización. Este método evita que todo el servicio se quede sin disponibilidad durante el proceso de actualización.
* Un *dominio de error* define los potenciales puntos de error de hardware o de la red. En el caso de los roles con más de una instancia, el controlador de tejido garantiza que las instancias se distribuyen entre varios dominios de error, lo que evitar que errores aislados de hardware interrumpan el servicio. Los dominios de error rigen toda la exposición a errores en el servidor y en el clúster.

El [Acuerdo de Nivel de Servicio (SLA) de Azure](https://azure.microsoft.com/support/legal/sla/) garantiza que cuando se implementan dos o más instancias de rol web en diferentes dominios de error y de actualización, tendrán conectividad externa al menos el 99,95 % del tiempo. A diferencia de los dominios de actualización, no hay forma de controlar el número de dominios de error. Azure asigna automáticamente los dominios de error y distribuye las instancias de rol entre ellos. Al menos las dos primeras instancias de cada rol se colocan en diferentes dominios de error y de actualización para garantizar que cualquier rol con un mínimo de dos instancias cumple el Acuerdo de Nivel de Servicio. Esto se representa en el diagrama siguiente.

![Vista simplificada del aislamiento de los dominios de error](./images/technical-guidance-recovery-local-failures/partitioning-1.png)

### <a name="load-balancing"></a>Equilibrio de carga
Todo el tráfico entrante a un rol web pasa a través de un equilibrador de carga sin estado, que distribuye las solicitudes de clientes entre las instancias de rol. Las instancias de rol individuales no tienen direcciones IP públicas y no se pueden direccionar directamente desde Internet. Los roles web no tienen estado, con el fin de que todas las solicitudes de clientes se puedan enrutar a cualquier instancia de rol. Se desencadena un evento [StatusCheck](https://msdn.microsoft.com/library/microsoft.windowsazure.serviceruntime.roleenvironment.statuscheck.aspx) cada 15 segundos. Esto se puede utilizar para indicar si el rol está listo para recibir tráfico o si está ocupado y debe quitarse de la rotación del equilibrador de carga.

## <a name="virtual-machines"></a>Virtual Machines
Azure Virtual Machines difiere de los roles de proceso de plataforma como servicio (PaaS) en varios aspectos relacionados con la alta disponibilidad. En algunos casos, deberá realizar tareas adicionales para garantizar una alta disponibilidad.

### <a name="disk-durability"></a>Durabilidad del disco
A diferencia de las instancias de rol de PaaS, los datos almacenados en las unidades de máquinas virtuales son persistentes, aunque la máquina virtual se reasigne. Azure Virtual Machines usa discos de máquinas virtuales que existen como blobs en Azure Storage. Debido a las características de disponibilidad de Azure Storage, los datos almacenados en las unidades de una máquina virtual también presentan una alta disponibilidad.

Tenga en cuenta que, en las máquinas virtuales de Windows, la unidad D es la excepción a esta regla. La unidad D es realmente un almacenamiento físico en el servidor en bastidor que hospeda la máquina virtual y sus datos se perderán si esta se recicla. La unidad D solo está prevista como almacenamiento temporal. En Linux, Azure "normalmente" (pero no siempre) expone el disco local temporal como un dispositivo de bloques /dev/sdb. A menudo, el agente de Linux de Azure lo monta como los puntos de montaje /mnt/resource o /mnt (configurables mediante /etc/waagent.conf).

### <a name="partitioning"></a>Creación de particiones
Azure entiende de manera nativa los niveles de una aplicación PaaS (rol web y rol de trabajo) y, por consiguiente, puede distribuirlos correctamente entre dominios de error y de actualización. En cambio, los niveles de una aplicación de infraestructura como servicio (IaaS) se deben definir manualmente a través de conjuntos de disponibilidad. Los conjuntos de disponibilidad son necesarios para un Acuerdo de Nivel de Servicio en IaaS.

![Conjuntos de disponibilidad para Máquinas virtuales de Azure](./images/technical-guidance-recovery-local-failures/partitioning-2.png)

En el diagrama anterior, el nivel de Internet Information Services (IIS) (que funciona como nivel de aplicación web) y el nivel de SQL (que funciona como nivel de datos), se asignan a distintos conjuntos de disponibilidad. Esto garantiza que todas las instancias de cada nivel tienen redundancia de hardware mediante la distribución de máquinas virtuales entre los dominios de error y que no se quitan capas completas durante una actualización.

### <a name="load-balancing"></a>Equilibrio de carga
Si hay que distribuir el tráfico entre las máquinas virtuales, tendrá que agruparlas en una aplicación y equilibrar la carga mediante un punto de conexión TCP o UDP específico. Para más información, consulte [Equilibrio de carga para servicios de infraestructura de Azure](/azure/virtual-machines/virtual-machines-linux-load-balance/?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json). Si las máquinas virtuales reciben entradas de otro origen (por ejemplo, un mecanismo de puesta en cola), no será necesario un equilibrador de carga. El equilibrador de carga utiliza una comprobación básica de mantenimiento para determinar si se debe enviar tráfico al nodo. También es posible crear sondeos propios que implementen métricas de mantenimiento específicas de la aplicación que determinen si la máquina virtual debe recibir tráfico.

## <a name="storage"></a>Storage
Azure Storage es el servicio de datos duradero de línea de base de Azure. Proporciona almacenamiento en blobs, tablas, colas y discos de máquinas virtuales. Utiliza una combinación de replicación y administración de recursos para proporcionar alta disponibilidad en un centro de datos individual. El Acuerdo de Nivel de Servicio de disponibilidad de Azure Storage garantiza que al menos un 99,9% del tiempo:

* Se procesarán correctamente las solicitudes con el formato correcto para agregar, actualizar, leer y eliminar datos.
* Las cuentas de almacenamiento tendrán conectividad con la puerta de enlace de Internet.

### <a name="replication"></a>Replicación
Azure Storage facilita la durabilidad de los datos, para lo que mantiene varias copias de todos los datos en diferentes unidades ubicadas en subsistemas de almacenamiento físico totalmente independientes dentro de la región. Los datos se replican de forma sincrónica y todas las copias se confirman antes de que se reconoce la operación de escritura. Azure Storage es muy coherente, lo cual significa que se garantiza que las operaciones de lectura reflejan las operaciones de escritura más recientes. Además, las copias de los datos se examinan continuamente para detectar y reparar la degradación de bits, una amenaza para la integridad de los datos almacenados que a menudo se pasa por alto.

Los servicios se pueden beneficiar de la replicación solo con usar Azure Storage. El desarrollador del servicio no necesita hacer ningún trabajo adicional para que se realice la recuperación de un error local.

### <a name="resource-management"></a>Administración de recursos
Las cuentas de almacenamiento creadas después de mayo de 2014 pueden tener hasta 500 TB (el máximo anterior era de 200 TB). Si se requiere espacio adicional, las aplicaciones se deberán diseñar para que usen varias cuentas de almacenamiento.

### <a name="virtual-machine-disks"></a>Discos de máquinas virtuales
Un disco de máquina virtual se almacena como un blob en páginas en Azure Storage, lo que le otorga las mismas propiedades de durabilidad y escalabilidad que a Blob Storage. Este diseño hace que los datos de un disco de máquina virtual sean persistentes aunque se produzca un error en el servidor que ejecuta la máquina virtual y esta deba reiniciarse en otro servidor.

## <a name="database"></a>Base de datos
### <a name="sql-database"></a>SQL Database
Azure SQL Database proporciona la base de datos como un servicio. Permite que las aplicaciones se aprovisionen rápidamente bases de datos relacionales, inserten datos en ellas y las consulten. Proporciona muchas de las conocidas características y funcionalidades de SQL Server, al tiempo que reduce la carga de hardware, configuración, aplicación de revisiones y resistencia.

> [!NOTE]
> Azure SQL Database no proporciona una paridad de características uno a uno con SQL Server. Está pensada para satisfacer un conjunto diferente de requisitos, que es adecuado exclusivamente para aplicaciones en la nube (escala elástica, base de datos como servicio para reducir costos de mantenimiento, etc.). Para obtener más información, consulte [Selección de una opción de SQL Server en la nube: Base de datos (PaaS) SQL de Azure o SQL Server en máquinas virtuales de Azure (IaaS)](/azure/sql-database/sql-database-paas-vs-sql-server-iaas/).
> 
> 

#### <a name="replication"></a>Replicación
Azure SQL Database proporciona resistencia integrada a errores en el nivel de nodo. Todas las operaciones de escritura que se realizan en una base de datos se replican automáticamente en dos o más nodos en segundo plano mediante una técnica de confirmación de quórum. (tanto el nodo principal como al menos un nodo secundario deben confirmar que la actividad se escribe en el registro de transacciones antes de que la transacción se considere correcta y se devuelve). En caso de error del nodo, la base de datos automáticamente se conmuta por error a una de las réplicas secundarias. Esto produce una interrupción de la conexión transitoria para las aplicaciones cliente. Por este motivo, todos los clientes de Azure SQL Database deben implementar alguna forma de control de conexiones transitorias. Para más información, consulte [Vuelva a intentar la orientación específica del servicio](/azure/best-practices-retry-service-specific/).

#### <a name="resource-management"></a>Administración de recursos
Cada base de datos, cuando se crea, se configura con un límite de tamaño superior. El tamaño máximo actualmente disponible es de 1 TB (los límites de tamaño varían según su nivel de servicio, consulte [Niveles de servicio y niveles de rendimiento de Azure SQL Database](/azure/sql-database/sql-database-resource-limits/#service-tiers-and-performance-levels). Cuando una base de datos alcanza su límite superior de tamaño, rechaza comandos INSERT o UPDATE adicionales (la consulta y eliminación de datos aún es posible).

En una base de datos, Azure SQL Database usa un tejido para administrar los recursos. No obstante, en lugar de un controlador de tejido, usa una topología en anillo para detectar errores. Cada réplica de un clúster tienen dos vecinos y es responsable de detectar cuándo estos dejan de funcionar. Cuando una réplica deja de funcionar, sus vecinos desencadenan un agente de reconfiguración que la vuelve a crear en otra máquina. Se proporciona limitación del motor para garantizar que un servidor lógico no usa demasiados recursos de una máquina ni supera los límites físicos de la misma.

### <a name="elasticity"></a>Elasticidad
Si la aplicación requiere más espacio que el límite de 1 TB de la base de datos, deberá implementar un enfoque de escalado horizontal. El escalado horizontal con Azure SQL Database se realiza mediante la creación manual de particiones, operación que también se denomina particionamiento, de datos en varias instancias de SQL Database. Este enfoque de escalado horizontal brinda la oportunidad de lograr un crecimiento del costo casi lineal con la escala. El crecimiento elástico o la capacidad a petición pueden aumentar según sea necesario, lo cual implicará un aumento de los costos, ya que las bases de datos se facturan según el tamaño real promedio usado cada día, y no según el tamaño máximo posible.

## <a name="sql-server-on-virtual-machines"></a>SQL Server en máquinas virtuales
Si instala SQL Server en (versión 2014 o posterior) Azure Virtual Machines, podrá sacar provecho de las características de disponibilidad tradicionales de SQL Server. Dichas características incluyen grupos de disponibilidad AlwaysOn y creación de reflejo de la base de datos. Tenga en cuenta que las máquinas virtuales de Azure, el almacenamiento y las redes tienen características operativas diferentes de las de una infraestructura de TI local y no virtualizada. Para implementar correctamente una solución de alta disponibilidad y recuperación ante desastres (HA/DR) de SQL Server en Azure es preciso conocer estas diferencias y diseñar la solución de tal forma que se adapte a ellas.

### <a name="high-availability-nodes-in-an-availability-set"></a>Nodos de alta disponibilidad en un conjunto de disponibilidad
Cuando implementa una solución de alta disponibilidad en Azure, se puede usar el conjunto de disponibilidad de Azure para colocar los nodos de alta disponibilidad en dominios de error y de actualización independientes. Para ser más precisos, "conjunto de disponibilidad" es un concepto de Azure. Se trata de un procedimiento recomendado que debe seguir para asegurarse de que las bases de datos tienen realmente una alta disponibilidad, independientemente de que se utilicen grupos de disponibilidad AlwaysOn, creación de reflejo de la base de datos o cualquier otra característica. Si no sigue este procedimiento, es posible que tenga la certeza incorrecta de que su sistema tiene alta disponibilidad. Pero en realidad, todos los nodos pueden dejar de funcionar simultáneamente porque están colocados en el mismo dominio de error de la región de Azure.

Esta recomendación no tiene tanta validez con el trasvase de registros. Al tratarse de una característica de recuperación ante desastres, debe asegurarse de que los servidores se ejecutan en regiones independientes de Azure. Por definición, estas regiones son dominios de error independientes.

En el caso de las máquinas virtuales en Azure Cloud Services implementadas mediante el portal clásico en el mismo conjunto de disponibilidad, debe implementarlas en el mismo servicio en la nube. Las máquinas virtuales implementadas mediante Azure Resource Manager (el portal actual) no tienen esta limitación. En el caso de las máquinas virtuales implementadas mediante el portal clásico en el servicio en la nube de Azure, solo los nodos en el mismo servicio en la nube pueden participar en el mismo conjunto de disponibilidad. Además, las máquinas virtuales en Cloud Services deben estar en la misma red virtual para asegurarse de que conservan sus direcciones IP incluso después de la recuperación del servicio, ya que esto evita las interrupciones de actualización de DNS.

### <a name="azure-only-high-availability-solutions"></a>Solo Azure: soluciones de alta disponibilidad
Puede tener una solución de alta disponibilidad para sus bases de datos SQL Server en Azure mediante el uso de grupos de disponibilidad AlwaysOn o de creación de reflejo de la base de datos.

El siguiente diagrama muestra la arquitectura de los grupos de disponibilidad AlwaysOn que se ejecutan en Azure Virtual Machines. Este diagrama se ha tomado del artículo [Alta disponibilidad y recuperación ante desastres para SQL Server en Azure Virtual Machines](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-high-availability-dr/), en el que se analiza en profundidad este tema.

![Grupos de disponibilidad de AlwaysOn en Microsoft Azure](./images/technical-guidance-recovery-local-failures/high_availability_solutions-1.png)

También puede aprovisionar automáticamente una implementación de grupos de disponibilidad AlwaysOn de un extremo a otro en máquinas virtuales de Azure mediante el uso de la plantilla de AlwaysOn en el Portal de Azure. Para obtener más información, consulte [Oferta de AlwaysOn de SQL Server en la Galería de Microsoft Azure Portal](https://blogs.technet.microsoft.com/dataplatforminsider/2014/08/25/sql-server-alwayson-offering-in-microsoft-azure-portal-gallery/).

El diagrama siguiente muestra el uso de creación de reflejo de la base de datos en Azure Virtual Machines. También se ha tomado del artículo [Alta disponibilidad y recuperación ante desastres para SQL Server en Azure Virtual Machines](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-high-availability-dr/).

![Creación de reflejo de la base de datos en Microsoft Azure](./images/technical-guidance-recovery-local-failures/high_availability_solutions-2.png)

> [!NOTE]
> En ambas arquitecturas se requiere un controlador de dominio. Sin embargo, con creación de reflejo de la base de datos se pueden utilizar certificados de servidor para eliminar la necesidad de un controlador de dominio.
> 
> 

## <a name="other-azure-platform-services"></a>Otros servicios de la plataforma de Azure
Las aplicaciones que se basan en Azure se benefician de las capacidades de la plataforma para recuperarse de errores locales. En algunos casos, se pueden realizar acciones concretas para aumentar la disponibilidad de un escenario concreto.

### <a name="service-bus"></a>Azure Service Bus
Para mitigar una interrupción temporal de Azure Service Bus, considere la posibilidad de crear una cola duradera en el lado de cliente. Con ello, utilizará temporalmente un mecanismo de almacenamiento local alternativo para almacenar los mensajes que no se puedan agregar a la cola de Service Bus. La aplicación puede decidir cómo controlar los mensajes almacenados temporalmente una vez restaurado el servicio. Para obtener más información, consulte [Prácticas recomendadas para la realización de mejoras con la mensajería asincrónica de Service Bus](/azure/service-bus-messaging/service-bus-performance-improvements/) y [Service Bus (recuperación ante desastres)](recovery-loss-azure-region.md#other-azure-platform-services).

### <a name="hdinsight"></a>HDInsight
Los datos asociados a HDInsight de Azure se almacenan de forma predeterminada en el Almacenamiento de blobs de Azure. Azure Storage especifica las propiedades de durabilidad y alta disponibilidad de Blob Storage. El procesamiento de varios nodos asociado a los trabajos MapReduce de Hadoop se realiza en un sistema de archivos distribuido Hadoop (HDFS) transitorio que se aprovisiona cuando HDInsight lo necesita. Los resultados de un trabajo MapReduce también se almacenan de forma predeterminada en Almacenamiento de blobs de Azure, con el fin de que los datos procesados sean duraderos y tengan una alta disponibilidad después de que el clúster de Hadoop se desaprovisione. Para obtener más información, consulte [HDInsight (recuperación ante desastres)](recovery-loss-azure-region.md#other-azure-platform-services).

## <a name="checklists-for-local-failures"></a>Listas de comprobación para errores locales
### <a name="cloud-services"></a>Cloud Services
1. Revise la sección Cloud Services de este documento.
2. Configure al menos dos instancias para cada rol.
3. Conserve el estado en un almacenamiento durable, no en instancias de rol.
4. Controle correctamente el evento StatusCheck.
5. Encapsule los cambios relacionados en transacciones siempre que sea posible.
6. Compruebe que las tareas del rol de trabajo son idempotentes y reiniciables.
7. Continúe invocando las operaciones hasta que se completen satisfactoriamente.
8. Considere estrategias de escalado automático.

### <a name="virtual-machines"></a>Virtual Machines
1. Revise la sección Máquinas virtuales de este documento.
2. No utilice la unidad D para el almacenamiento persistente.
3. Agrupe las máquinas de un nivel de servicio en un conjunto de disponibilidad.
4. Configure el equilibrio de carga y los sondeos opcionales.

### <a name="storage"></a>Storage
1. Revise la sección Almacenamiento de este documento.
2. Utilice varias cuentas de almacenamiento cuando los datos o el ancho de banda superen las cuotas.

### <a name="sql-database"></a>SQL Database
1. Revise la sección SQL Database de este documento.
2. Implemente una directiva de reintentos para controlar los errores transitorios.
3. Utilice el particionamiento como estrategia de escalado horizontal.

### <a name="sql-server-on-virtual-machines"></a>SQL Server en máquinas virtuales
1. Revise la sección SQL Server en máquinas virtuales de este documento.
2. Siga las recomendaciones anteriores para Máquinas virtuales.
3. Use las características de alta disponibilidad de SQL Server, como AlwaysOn.

### <a name="service-bus"></a>Azure Service Bus
1. Revise la sección Service Bus de este documento.
2. Considere la creación de una cola duradera en el lado del cliente como copia de seguridad.

### <a name="hdinsight"></a>HDInsight
1. Revise la sección HDInsight de este documento.
2. Para los errores locales no se requieren más pasos de disponibilidad.

