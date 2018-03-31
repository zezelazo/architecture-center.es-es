---
title: "Elección de un almacén de datos de OLTP"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 1c27d7d5f3b78f40822de6b77664dbf49b1367f6
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/14/2018
---
# <a name="choosing-an-oltp-data-store-in-azure"></a>Elección de un almacén de datos de OLTP en Azure

El procesamiento de transacciones en línea (OLTP) es la administración de datos transaccionales y el procesamiento de transacciones. Este tema compara las diversas opciones de soluciones de OLTP en Azure.

> [!NOTE]
> Para más información acerca de cuándo se debe usar un almacén de datos de OLTP, consulte [Procesamiento de transacciones en línea](../scenarios/online-analytical-processing.md).

## <a name="what-are-your-options-when-choosing-an-oltp-data-store"></a>¿Cuáles son las opciones al elegir un almacén de datos de OLTP?

En Azure, todos los almacenes de datos siguientes cumplen los requisitos principales para OLTP y para la administración de datos de transacciones:

- [Azure SQL Database](/azure/sql-database/)
- [SQL Server en una máquina virtual de Azure](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-server-iaas-overview?toc=%2Fazure%2Fvirtual-machines%2Fwindows%2Ftoc.json)
- [Azure Database for MySQL](/azure/mysql/)
- [Azure Database para PostgreSQL](/azure/postgresql/)

## <a name="key-selection-criteria"></a>Principales criterios de selección

Para restringir las opciones, empiece por responder a estas preguntas:

- ¿Quiere un servicio administrado en lugar de administrar sus propios servidores?

- ¿Tiene la solución dependencias específicas para Microsoft SQL Server, MySQL, o compatibilidad con PostgreSQL? La aplicación puede limitar los almacenes de datos que puede elegir en función de los controladores que admite para la comunicación con el almacén de datos o las suposiciones que este hace sobre qué base de datos se utiliza.

- ¿Son especialmente importantes sus requisitos de rendimiento de escritura? En caso afirmativo, elija una opción que proporcione tablas en memoria. 

- ¿Su solución es multiinquilino? Si es así, considere la posibilidad de usar grupos de capacidad, donde varias instancias de bases de datos parten de un grupo elástico de recursos, en lugar de recursos fijos por base de datos. Esto puede ayudarle a distribuir mejor la capacidad entre todas las instancias de bases de datos y puede hacer que la solución sea más rentable.

- ¿Hace falta que los datos sean legibles con una latencia baja en varias regiones? En caso afirmativo, elija una opción que admita réplicas secundarias legibles.

- ¿Necesita que la base de datos tenga alta disponibilidad entre regiones geográficas? En caso afirmativo, elija una opción que admita la replicación geográfica. Considere también las opciones que admiten la conmutación automática por error desde la réplica principal a una réplica secundaria.

- ¿La base de datos tiene necesidades específicas de seguridad? Si es así, examine las opciones que proporcionan funcionalidades como la seguridad de nivel de fila, el enmascaramiento de datos y el cifrado de datos transparente.

## <a name="capability-matrix"></a>Matriz de funcionalidades

En las tablas siguientes se resumen las diferencias clave en cuanto a funcionalidades.

### <a name="general-capabilities"></a>Funcionalidades generales 
| | Azure SQL Database | SQL Server en una máquina virtual de Azure | Azure Database for MySQL | Azure Database for PostgreSQL |
| --- | --- | --- | --- | --- | --- |
| Es un servicio administrado | Sí | Sin  | Sí | Sí |
| Se ejecuta en una plataforma | N/D | Windows, Linux, Docker | N/D | N/D |
| Capacidad de programación <sup>1</sup> | T-SQL, .NET, R | T-SQL, .NET, R, Python | T-SQL, .NET, R, Python | SQL | SQL |

[1] No incluye compatibilidad con controladores de cliente, lo que permite a muchos lenguajes de programación conectarse y usar el almacén de datos de OLTP.

### <a name="scalability-capabilities"></a>Funcionalidades de escalabilidad
| | Azure SQL Database | SQL Server en una máquina virtual de Azure| Azure Database for MySQL | Azure Database for PostgreSQL|
| --- | --- | --- | --- | --- | --- |
| Tamaño máximo de la instancia de base de datos | [4 TB](/azure/sql-database/sql-database-resource-limits) | 256 TB | [1 TB](/azure/mysql/concepts-limits) | [1 TB](/azure/postgresql/concepts-limits) |
| Es compatible con grupos de capacidad  | Sí | Sí | Sin  | Sin  |
| Es compatible con el escalado horizontal de clústeres  | Sin  | Sí | Sin  | Sin  |
| Escalabilidad dinámica (escalado vertical)  | Sí | Sin  | Sí | Sí |

### <a name="analytic-workload-capabilities"></a>Funcionalidades de cargas de trabajo de análisis
| | Azure SQL Database | SQL Server en una máquina virtual de Azure| Azure Database for MySQL | Azure Database for PostgreSQL|
| --- | --- | --- | --- | --- | --- | 
| Tablas temporales | Sí | Sí | Sin  | Sin  |
| Tablas en memoria (optimizadas para memoria) | Sí | Sí | Sin  | Sin  |
| Compatible con almacén de columnas | Sí | Sí | Sin  | Sin  |
| Procesamiento adaptable de consultas | Sí | Sí | Sin  | Sin  |

### <a name="availability-capabilities"></a>Funcionalidades de disponibilidad
| | Azure SQL Database | SQL Server en una máquina virtual de Azure| Azure Database for MySQL | Azure Database for PostgreSQL|
| --- | --- | --- | --- | --- | --- | 
| Réplicas secundarias legibles | Sí | Sí | Sin  | Sin  | 
| Replicación geográfica | Sí | Sí | Sin  | Sin  | 
| Conmutación automática por error en replicación secundaria | Sí | Sin  | Sin  | Sin |
| Restauración a un momento dado | Sí | Sí | Sí | Sí |

### <a name="security-capabilities"></a>Funcionalidades de seguridad
| | Azure SQL Database | SQL Server en una máquina virtual de Azure| Azure Database for MySQL | Azure Database for PostgreSQL|
| --- | --- | --- | --- | --- | --- | 
| Seguridad de nivel de fila | Sí | Sí | Sí | Sí |
| Enmascaramiento de datos | Sí | Sí | Sin  | Sin  |
| Cifrado de datos transparente | Sí | Sí | Sí | Sí |
| Restricción del acceso a determinadas direcciones IP | Sí | Sí | Sí | Sí |
| Restricción del acceso para permitir solo el acceso de la red virtual | Sí | Sí | Sin  | Sin  |
| Autenticación con Azure Active Directory | Sí | Sí | Sin  | Sin  |
| Autenticación de Active Directory | Sin  | Sí | Sin  | Sin  |
| Multi-Factor Authentication | Sí | Sí | Sin  | Sin  |
| Compatible con [Always Encrypted](/sql/relational-databases/security/encryption/always-encrypted-database-engine) | Sí | Sí | Sí | Sin  | Sin  |
| Dirección IP privada | Sin  | Sí | Sí | Sin  | Sin  |

