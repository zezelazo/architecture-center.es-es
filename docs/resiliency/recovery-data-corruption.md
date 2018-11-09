---
title: Recuperación de daños o eliminación accidental de los datos
description: Artículo para entender cómo recuperarse ante datos dañados o eliminación accidental de datos y para diseñar aplicaciones resistentes, con alta disponibilidad y con tolerancia a errores, así como para planear la recuperación ante desastres
author: MikeWasson
ms.date: 11/11/2018
ms.openlocfilehash: 1f3dd448ac6172727481c437fb8a113f25d83464
ms.sourcegitcommit: dbbf914757b03cdee7a274204f9579fa63d7eed2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/02/2018
ms.locfileid: "50916294"
---
# <a name="recover-from-data-corruption-or-accidental-deletion"></a>Recuperación de daños o eliminación accidental de los datos 

Una parte de un plan de continuidad empresarial sólido consiste en tener un plan por si los datos resultan dañados o se eliminan accidentalmente. A continuación se proporciona información acerca de la recuperación en caso de datos dañados o eliminados accidentalmente debido a errores de la aplicación o a un error del operador.

## <a name="virtual-machines"></a>Virtual Machines

Para proteger máquinas virtuales (VM) de Azure de errores de aplicación o de eliminaciones accidentales, use [Azure Backup](/azure/backup/). Azure Backup permite la creación de copias de seguridad consistentes en varios discos de máquinas virtuales. Además, el almacén de Backup se puede replicar entre varias regiones para proporcionar recuperación en caso de pérdida de la región.

## <a name="storage"></a>Storage

Azure Storage ofrece resistencia de datos gracias a las réplicas automatizadas. Sin embargo, esto no evita que el código de aplicación o los usuarios dañen los datos, de forma accidental o malintencionada. Mantener la fidelidad de los datos en caso de error de la aplicación o del usuario requiere técnicas más avanzadas, como copiar los datos en una ubicación de almacenamiento secundaria con un registro de auditoría. 

- **Blobs en bloques**. Cree una instantánea de un momento dado de cada blob en bloques. Para obtener más información, consulte [Crear una instantánea de un blob](/rest/api/storageservices/creating-a-snapshot-of-a-blob). Por cada instantánea, solo se le cobrará por el almacenamiento que necesite para almacenar las diferencias producidas en el blob desde el último estado de instantánea. Las instantáneas dependen de la existencia del blob original en el que se basan, por lo que se recomienda realizar una operación de copia a otro blob, o incluso a otra cuenta de almacenamiento. Esto garantiza que los datos de copia de seguridad que los datos de la copia de seguridad estén protegidos correctamente contra la eliminación accidental. Puede usar [AzCopy](/azure/storage/common/storage-use-azcopy) o [Azure PowerShell](/azure/storage/common/storage-powershell-guide-full) para copiar los blobs en otra cuenta de almacenamiento.

- **Files**. Use [instantáneas de recurso compartido](/azure/storage/files/storage-snapshots-files), o AzCopy o PowerShell para copiar los archivos en otra cuenta de almacenamiento.

- **Tables**. Use AzCopy para exportar los datos de tablas a otra cuenta de almacenamiento de otra región.

## <a name="database"></a>Base de datos

### <a name="azure-sql-database"></a>Azure SQL Database 

SQL Database realiza automáticamente una combinación de copias de seguridad completas semanales, copias de seguridad diferenciales cada hora y copias de seguridad del registro de transacciones cada 5 o 10 minutos con el fin de proteger su empresa contra la pérdida de datos. Use la restauración a un momento dado para restaurar una base de datos a un momento anterior. Para más información, consulte:

- [Recuperación de una Base de datos SQL de Azure mediante copias de seguridad automatizadas](/azure/sql-database/sql-database-recovery-using-backups)

- [Introducción a la continuidad empresarial con Azure SQL Database](/azure/sql-database/sql-database-business-continuity)

### <a name="sql-server-on-vms"></a>SQL Server en máquinas virtuales

Para ejecutar SQL Server en máquinas virtuales, hay dos opciones: copias de seguridad tradicionales y trasvase de registros. Las copias de seguridad tradicionales le permiten restaurar a un momento anterior específico, pero el proceso de recuperación es lento. La restauración de copias de seguridad tradicionales requiere empezar con una copia de seguridad completa inicial y, a continuación, aplicar las copias de seguridad realizadas después de ella. La segunda opción es configurar una sesión de trasvase de registros para retrasar la restauración de las copias de seguridad del registro (por ejemplo, en dos horas). Esto proporciona un tiempo para recuperarse de los errores que se produzcan en el servidor principal.

### <a name="azure-cosmos-db"></a>Azure Cosmos DB

Azure Cosmos DB crea automáticamente copias de seguridad a intervalos regulares. Las copias de seguridad se almacenan por separado en otro servicio de almacenamiento y se replican globalmente para lograr resistencia frente a desastres regionales. En caso de que elimine accidentalmente su base de datos o colección, puede presentar una incidencia de soporte técnico o llamar al servicio de soporte técnico de Azure para restaurar los datos a partir de la copia de seguridad automática más reciente. Para más información, consulte [Copias de seguridad y restauración automáticas en línea con Azure Cosmos DB](/azure/cosmos-db/online-backup-and-restore).

### <a name="azure-database-for-mysql-azure-database-for-postresql"></a>Azure Database for MySQL, Azure Database for PostreSQL

Cuando se usa Azure Database for MySQL o Azure Database for PostreSQL, el servicio de base de datos crea automáticamente una copia de seguridad del servicio cada cinco minutos. Con esta característica de copia de seguridad automática, puede restaurar el servidor y todas sus bases de datos en un servidor nuevo a un momento dado anterior. Para más información, consulte:

- [Copia de seguridad y restauración de un servidor en Azure Database for MySQL mediante Azure Portal](/azure/mysql/howto-restore-server-portal)

- [Copia de seguridad y restauración de un servidor en Azure Database for PostgreSQL mediante Azure Portal](/azure/postgresql/howto-restore-server-portal)

