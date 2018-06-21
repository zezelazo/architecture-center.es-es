---
title: Procesamiento de transacciones en línea (OLTP)
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 8650b919fc1a59240343015493a1fe41c8729a72
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2018
ms.locfileid: "30848706"
---
# <a name="online-transaction-processing-oltp"></a>Procesamiento de transacciones en línea (OLTP)

La administración de datos transaccionales mediante sistemas de equipos se conoce como procesamiento de transacciones en línea (OLTP). Los sistemas de OLTP registran interacciones empresariales a medida que se producen en el funcionamiento diario de la organización y admiten consultas de estos datos para realizar inferencias.

## <a name="transactional-data"></a>Datos transaccionales

Los datos transaccionales son información que realiza un seguimiento de las interacciones relacionadas con las actividades de una organización. Estas interacciones normalmente son transacciones comerciales, tales como pagos recibidos de los clientes, pagos realizados a los proveedores, movimiento de productos en el inventario, pedidos obtenidos o servicios entregados. Los eventos transaccionales, que representan a las transacciones, normalmente contienen una dimensión de tiempo, algunos valores numéricos y referencias a otros datos. 

Normalmente, las transacciones deben ser *atómicas* y *coherentes*. Atomicidad significa que una transacción completa siempre se realiza, correctamente o con error, como una unidad de trabajo y nunca se deja en un estado medio completado. Si no se puede completar una transacción, el sistema de base de datos debe revertir todos los pasos que se han hecho como parte de esa transacción. En los RDBMS tradicionales, esta reversión sucede automáticamente si no se puede finalizar una transacción. Coherencia significa que las transacciones dejan siempre los datos en un estado válido. (Estas son descripciones muy informales de atomicidad y coherencia. Hay definiciones más formales de estas propiedades, como [ACID](https://en.wikipedia.org/wiki/ACID)).

Las bases de datos transaccionales posibilitan una coherencia alta de las transacciones mediante el uso de diversas estrategias de bloqueo, como el bloqueo pesimista, para asegurarse de que todos los datos son altamente coherentes dentro del contexto de la empresa, para todos los usuarios y procesos. 

La arquitectura de implementación más común que utiliza datos transaccionales es el nivel de almacén de datos en una arquitectura de 3 niveles. Una arquitectura de 3 niveles normalmente consta de un nivel de presentación, un nivel de lógica de negocios y un nivel de almacén de datos. Una arquitectura de implementación relacionada es la arquitectura de [n niveles](/azure/architecture/guide/architecture-styles/n-tier), que puede tener varios niveles intermedios para el control de la lógica de negocios.

## <a name="typical-traits-of-transactional-data"></a>Rasgos típicos de los datos transaccionales

Los datos transaccionales suelen tener los siguientes rasgos:

| Requisito | DESCRIPCIÓN |
| --- | --- |
| Normalización | Muy normalizados |
| Esquema | Esquema durante la escritura, altamente aplicado|
| Coherencia | Coherencia alta, garantías ACID |
| Integridad | Integridad alta |
| Usa transacciones | Sí |
| Estrategia de bloqueo | Optimista o pesimista|
| Actualizable | Sí |
| Anexable | Sí |
| Carga de trabajo | Grandes escrituras, lecturas moderadas |
| Indización | Índices principales y secundarios |
| Tamaño de los datos | Pequeño a mediano tamaño |
| Modelo | Relacional |
| Forma de los datos | Tabular |
| Flexibilidad de consulta | Muy flexible |
| Escala | Pequeño (MB) a grande (algunos TB) | 

## <a name="when-to-use-this-solution"></a>Cuándo se debe utilizar esta solución

Elija OLTP cuando necesite procesar y almacenar eficazmente transacciones comerciales, y que estén inmediatamente disponibles para las aplicaciones cliente de una manera coherente. Use esta arquitectura cuando cualquier retraso tangible en el procesamiento pueda tener un impacto negativo en el funcionamiento diario de la empresa.

Los sistemas de OLTP están diseñados para procesar y almacenar de forma eficaz las transacciones, así como para consultar los datos transaccionales. El objetivo de procesar y almacenar eficazmente las transacciones individuales por parte de un sistema de OLTP se logra parcialmente mediante la normalización de datos (es decir, dividir los datos en fragmentos más pequeños que sean menos redundantes). La eficacia se debe a que permite que el sistema de OLTP procese grandes cantidades de transacciones de forma independiente y evita el procesamiento adicional necesario para mantener la integridad de los datos en presencia de datos redundantes.

## <a name="challenges"></a>Desafíos
La implementación y el uso de un sistema de OLTP pueden crear algunos problemas:

- Los sistemas de OLTP no siempre son buenos para controlar agregados en grandes cantidades de datos, aunque hay excepciones, como una solución basada en SQL Server bien planeada. Los análisis de los datos, que se basan en cálculos agregados de millones de transacciones individuales, hacen un uso muy intensivo de los recursos en un sistema de OLTP. Pueden tardar en ejecutarse y puede provocar una ralentización porque bloqueen otras transacciones de la base de datos.
- Si se realizan informes y análisis de los datos que estén muy normalizados, las consultas tienden a ser complejas, ya que la mayor parte de ellas tienen que anular la normalización de los datos mediante réplicas. Además, las convenciones de nomenclatura de los objetos de base de datos en los sistemas de OLTP tienden a ser breves y concisas. El aumento de la normalización, junto con unas convenciones de nomenclatura breves, hacen que sea difícil para los usuarios empresariales realizar consultas en los sistemas de OLTP sin la ayuda de un DBA o desarrollador de datos.
- El almacenamiento del historial de transacciones de forma indefinida y el almacenamiento de demasiados datos en cualquier tabla puede provocar una ralentización del rendimiento de las consultas, en función del número de transacciones almacenadas. La solución habitual consiste en mantener una ventana de tiempo relevante (por ejemplo, el año fiscal actual) en el sistema de OLTP y descargar los datos históricos a otros sistemas, como un data mart o un [almacenamiento de datos](./data-warehousing.md).

## <a name="oltp-in-azure"></a>OLTP en Azure

Aplicaciones como los sitios web hospedados en [App Service Web Apps](/azure/app-service/app-service-web-overview), REST API que se ejecutan en App Service o las aplicaciones de escritorio o móviles se comunican con el sistema de OLTP normalmente a través de una REST API intermediaria.

En la práctica, la mayoría de las cargas de trabajo no son OLTP puras. Tiende a haber también un componente analítico. Además, hay una creciente demanda de informes en tiempo real, como los informes activos en el sistema operativo. Esto también se denomina HTAP (procesamiento transaccional y analítico híbrido). Para más información, consulte [Online Analytical Processing (OLAP)](./online-analytical-processing.md) (Procesamiento analítico en línea [OLAP]).

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

|                              | Azure SQL Database | SQL Server en una máquina virtual de Azure | Azure Database for MySQL | Azure Database for PostgreSQL |
|------------------------------|--------------------|----------------------------------------|--------------------------|-------------------------------|
|      Es un servicio administrado      |        Sí         |                   Sin                    |           Sí            |              Sí              |
|       Se ejecuta en una plataforma       |        N/D         |         Windows, Linux, Docker         |           N/D            |              N/D              |
| Capacidad de programación <sup>1</sup> |   T-SQL, .NET, R   |         T-SQL, .NET, R, Python         |  T-SQL, .NET, R, Python  |              SQL              |

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

|                                                                                                             | Azure SQL Database | SQL Server en una máquina virtual de Azure | Azure Database for MySQL | Azure Database for PostgreSQL |
|-------------------------------------------------------------------------------------------------------------|--------------------|----------------------------------------|--------------------------|-------------------------------|
|                                             Seguridad de nivel de fila                                              |        Sí         |                  Sí                   |           Sí            |              Sí              |
|                                                Enmascaramiento de datos                                                 |        Sí         |                  Sí                   |            Sin             |              Sin                |
|                                         Cifrado de datos transparente                                         |        Sí         |                  Sí                   |           Sí            |              Sí              |
|                                  Restricción del acceso a determinadas direcciones IP                                   |        Sí         |                  Sí                   |           Sí            |              Sí              |
|                                  Restricción del acceso para permitir solo el acceso de la red virtual                                  |        Sí         |                  Sí                   |            Sin             |              Sin                |
|                                    Autenticación con Azure Active Directory                                    |        Sí         |                  Sí                   |            Sin             |              Sin                |
|                                       Autenticación de Active Directory                                       |         Sin          |                  Sí                   |            Sin             |              Sin                |
|                                         Multi-Factor Authentication                                         |        Sí         |                  Sí                   |            Sin             |              Sin                |
| Compatible con [Always Encrypted](/sql/relational-databases/security/encryption/always-encrypted-database-engine) |        Sí         |                  Sí                   |           Sí            |              Sin                |
|                                                 Dirección IP privada                                                  |         Sin          |                  Sí                   |           Sí            |              Sin                |

