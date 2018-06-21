---
title: Ampliación de las soluciones de datos locales a la nube
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 66fc225dc123202ba587d82f15ea0883e1bbf3b5
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/14/2018
ms.locfileid: "29289691"
---
# <a name="extending-on-premises-data-solutions-to-the-cloud"></a>Ampliación de las soluciones de datos locales a la nube

Cuando las organizaciones mueven cargas de trabajo y datos a la nube, sus centros de datos locales a menudo siguen desempeñando un rol importante. El término *nube híbrida* hace referencia a una combinación de la nube pública y centros de datos locales para crear un entorno de TI integrado que abarca ambas cosas. Algunas organizaciones utilizan nube híbrida como una ruta de acceso para migrar todo su centro de datos a la nube con el paso del tiempo. Otras organizaciones usan los servicios en la nube para ampliar su infraestructura local existente. 

En este artículo se describen algunas consideraciones y procedimientos recomendados para administrar los datos en una solución de nube híbrida.

## <a name="when-to-use-a-hybrid-solution"></a>Cuándo se usa una solución híbrida

Una solución híbrida se puede usar en los escenarios siguientes:

* Como estrategia de transición durante una migración a largo plazo a una solución nativa totalmente en la nube.
* Cuando las regulaciones o las directivas no permiten mover datos específicos o cargas de trabajo a la nube.
* Para la recuperación ante desastres y la tolerancia a errores, mediante la replicación de datos y servicios entre los entornos local y de la nube.
* Para reducir la latencia entre el centro de datos local y las ubicaciones remotas, mediante el hospedaje de parte de la arquitectura de Azure.

## <a name="challenges"></a>Desafíos

* Crear un entorno coherente en cuanto a seguridad, administración y desarrollo, y evitar la duplicación de trabajo.

* Crear conexión de datos confiable, de baja latencia y segura entre sus entornos local y en la nube.

* Replicar los datos y modificar las aplicaciones y herramientas para usar los almacenes de datos en cada entorno.

* Proteger y cifrar los datos que se hospedan en la nube, pero a los que se accede desde un entorno local, o viceversa.

## <a name="on-premises-data-stores"></a>Almacenes de datos locales

Los almacenes de datos locales incluyen bases de datos y archivos. Puede haber varias razones para mantenerlos en un entorno local. Puede haber regulaciones o directivas que no permitan mover datos o cargas de trabajo específicos a la nube. Cuestiones de soberanía, privacidad o seguridad de datos pueden favorecer la colocación en un entorno local. Durante una migración, puede que desee mantener algunos datos en un entorno local para alguna aplicación que aún no se haya migrado.

Las consideraciones que se tienen en cuenta al colocar los datos de una aplicación en una nube pública incluyen:

* **Costo**. El costo del almacenamiento de Azure puede ser significativamente menor que el costo de mantenimiento de un almacenamiento con características similares en un centro de datos local. Por supuesto, muchas compañías tienen han realizado inversiones en redes de área de almacenamiento de alta gama, por lo que es posible que estas ventajas de costo no se cristalicen totalmente hasta que el software quede anticuado.
* **Escalado elástico** El planeamiento y la administración del crecimiento de la capacidad de los datos en un entorno local puede resultar todo un desafío, sobre todo cuando es difícil de predecir. Estas aplicaciones pueden aprovechar tanto la capacidad a petición como el almacenamiento prácticamente ilimitado disponibles en la nube. Esta consideración es menos relevante para las aplicaciones que se componen de conjuntos de datos con tamaños relativamente estáticos.
* **Recuperación ante desastres**. Los datos almacenados en Azure se pueden replicar automáticamente tanto dentro de una región de Azure como en diferentes regiones geográficas. En entornos híbridos, estas mismas tecnologías se pueden utilizar para realizar replicaciones entre almacenes de datos locales y en la nube.

## <a name="extending-data-stores-to-the-cloud"></a>Ampliación de los almacenes de datos a la nube

Hay varias opciones para ampliar los almacenes de datos locales a la nube. Una opción consiste en tener réplicas locales y en la nube. Dicha opción puede ayudar a lograr un alto nivel de tolerancia a errores, pero puede que sea preciso realizar cambios en las aplicaciones para que se conecten al almacén de datos adecuado en caso de una conmutación por error.

Otra opción consiste en mover una parte de los datos al almacenamiento de nube, mientras que los más actuales o a los que más se accede se mantienen en un entorno local. Este método puede proporcionar una opción más rentable para el almacenamiento a largo plazo, así como mejorar los tiempos de respuesta del acceso a datos, ya que se reduce el conjunto de datos operativo.

Una tercera opción consiste en mantener todos los datos en un entorno local, pero usar la informática en la nube para hospedar las aplicaciones. Para ello, debería hospedar las aplicaciones en la nube y conectarlas al almacén de datos local a través de una conexión segura. 

## <a name="azure-stack"></a>Azure Stack

Si lo que desea es una solución en la nube híbrida completa, considere la posibilidad de usar [Microsoft Azure Stack](/azure/azure-stack/). Azure Stack es una plataforma en la nube híbrida que permite proporcionar servicios de Azure desde un centro su datos. Esto ayuda a mantener la coherencia entre el entorno local y Azure, ya que se usan herramientas idénticas y no se requiere que se realice ningún cambio en el código. 

Éstos son que algunos casos de uso de Azure y Azure Stack:

* **Soluciones perimetrales y desconectadas**. Para satisfacer los requisitos de latencia y conectividad, procese los datos en el entorno local en Azure Stack y agréguelos después a Azure para analizarlos, con lógica de aplicaciones común entre ambos. 
* **Aplicaciones en la nube que cumplen diversas regulaciones**. Desarrolle e implemente aplicaciones en Azure con la flexibilidad de implementar las mismas en un entorno local con Azure Stack para satisfacer los requisitos de cumplimiento normativo o las directivas.
* **Modelo de aplicación en la nube en el entorno local**. Use Azure para actualizar y ampliar las aplicaciones existentes o crear otras nuevas. Utilice un proceso de DevOps coherente a través de Azure en la nube y Azure Stack en un entorno local.

## <a name="sql-server-data-stores"></a>Almacenes de datos de SQL Server

Si ejecuta SQL Server de forma local, puede utilizar el servicio Microsoft Azure Blob Storage para realizar copias de seguridad y restauraciones. Para más información, consulte [Copia de seguridad y restauración de SQL Server con el servicio Microsoft Azure Blob Storage](/sql/relational-databases/backup-restore/sql-server-backup-and-restore-with-microsoft-azure-blob-storage-service). Esta funcionalidad proporciona un almacenamiento externo ilimitado y la capacidad de compartir las mismas copias de seguridad entre una versión de SQL Server que se ejecuta en local y otra versión que se ejecuta en una máquina virtual en Azure. 

[Azure SQL Database](/azure/sql-database/) es una base de datos relacional como servicio administrada. Como Azure SQL Database usa el motor de Microsoft SQL Server, las aplicaciones pueden acceder a los datos de la misma manera con ambas tecnologías. Azure SQL Database también se puede combinar con SQL Server de varias formas. Por ejemplo, la característica [SQL Server Stretch Database](/sql/sql-server/stretch-database/stretch-database) permite a una aplicación acceder a lo que parece una tabla individual de una base de datos de SQL Server mientras que algunas filas, o todas ellas, de dicha tabla se pueden almacenar en Azure SQL Database. Esta tecnología mueve automáticamente a la nube los datos a los que no se accede durante un período de tiempo. Las aplicaciones que leen estos datos no son conscientes de que se han movido datos a la nube.

El mantenimiento de almacenes de datos en un entorno local y en la nube puede resultar complicado cuando se desea mantener los datos sincronizados. Este problema se puede resolver con [SQL Data Sync](/azure/sql-database/sql-database-sync-data), un servicio basado en Azure SQL Database que permite sincronizar los datos seleccionados de manera bidireccional entre varias instancias de Azure SQL e instancias de SQL Server. Aunque que Data Sync facilita la labor de mantener actualizados los datos a través de estos almacenes de datos, no debe usarse para la recuperación ante desastres ni para realizar la migración SQL Server local a Microsoft Azure SQL Database.

Tanto para la recuperación ante desastres como para la continuidad empresarial, puede usar [Grupos de disponibilidad AlwaysOn](/sql/database-engine/availability-groups/windows/overview-of-always-on-availability-groups-sql-server) para replicar los datos en dos o más instancias de SQL Server, algunas de las cuales pueden ejecutarse en máquinas virtuales de Azure en otra región geográfica.

## <a name="network-shares-and-file-based-data-stores"></a>Recursos compartidos de red y almacenes de datos basados en archivos

En una arquitectura de nube híbrida, es habitual que una organización conserve los archivos más recientes en un entorno local, mientras que los archivos antiguos se guardan en la nube. Esto a veces recibe el nombre de creación de niveles de archivo, donde hay un acceso ininterrumpido a ambos conjuntos de archivos, los hospedados en un entorno local y los hospedados en la nube. Este enfoque ayuda a minimizar el uso del ancho de banda de red y los tiempos de acceso a los archivos más recientes, que son a los que probablemente se acceda con más frecuencia. Al mismo tiempo se obtienen las ventajas del almacenamiento en la nube de los datos archivados. 

Las organizaciones también pueden desear mover sus recursos compartidos de red completamente a la nube. Esto sería deseable, por ejemplo, si las aplicaciones que acceden a ellos también se encuentran en la nube. Este procedimiento se puede realizar mediante las herramientas de [orquestación de datos](../technology-choices/pipeline-orchestration-data-movement.md).


[Azure StorSimple](/azure/storsimple/) ofrece la más completa solución de almacenamiento integrado para administrar las tareas de almacenamiento entre los dispositivos locales y el almacenamiento en la nube de Azure. StorSimple es una solución de red de área de almacenamiento (SAN) eficaz, rentable y fácilmente administrable que elimina muchos de los problemas y gastos asociados con el almacenamiento empresarial y la protección de datos. Usa el dispositivo de la serie StorSimple 8000, se integra con los servicios en la nube y proporciona un conjunto de herramientas de administración integradas.

Otra forma de usar los recursos de red locales junto con el almacenamiento de archivos en la nube es con [Azure Files](/azure/storage/files/storage-files-introduction). Azure Files ofrece recursos compartidos de archivos completamente administrados a los que se puede acceder con el protocolo [Bloque de mensajes del servidor](https://msdn.microsoft.com/library/windows/desktop/aa365233.aspx?f=255&MSPPError=-2147217396) (SMB), que a veces se denomina CIFS. Azure Files se puede montar como un recurso compartido de archivos en un equipo local, o bien se puede usar con las aplicaciones existentes que acceden a archivos locales o del recurso compartido de red.

Para sincronizar los recursos compartidos de archivos en Azure Files con los servidores de Windows locales, use [Azure File Sync](/azure/storage/files/storage-sync-files-planning). Una de las principales ventajas de Azure File Sync es la capacidad para colocar los archivos en niveles entre el servidor de archivos local y Azure Files. Esto permite mantener solo los más recientes y los últimos a los que se ha accedido localmente. 

Para más información, consulte [Decisión sobre cuándo usar Azure Blobs, Azure Files o Azure Disks](/azure/storage/common/storage-decide-blobs-files-disks).

## <a name="hybrid-networking"></a>Redes híbridas

Este artículo se ha centrado en soluciones de datos híbridas, pero también se ha indicado cómo ampliar la red local a Azure. Para más información acerca de este aspecto de las soluciones híbridas, consulte los siguientes temas:

- [Elección de una solución para conectar una red local a Azure](../../reference-architectures/hybrid-networking/considerations.md)
- [Conexión de una red local a Azure](../../reference-architectures/hybrid-networking/index.md)

