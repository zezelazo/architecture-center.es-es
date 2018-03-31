---
title: "Procesamiento de transacciones en línea (OLTP)"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 07e7f680c8ee5e8589ff7cd2236ff95f6ee84f4c
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/14/2018
---
# <a name="online-transaction-processing-oltp"></a>Procesamiento de transacciones en línea (OLTP)

La administración de [datos transaccionales](../concepts/transactional-data.md) mediante sistemas de equipos se conoce como procesamiento de transacciones en línea (OLTP). Los sistemas de OLTP registran interacciones empresariales a medida que se producen en el funcionamiento diario de la organización y admiten consultas de estos datos para realizar inferencias.

![OLTP en Azure](./images/oltp-data-pipeline.png)

## <a name="when-to-use-this-solution"></a>Cuándo se debe utilizar esta solución

Elija OLTP cuando necesite procesar y almacenar eficazmente transacciones comerciales, y que estén inmediatamente disponibles para las aplicaciones cliente de una manera coherente. Use esta arquitectura cuando cualquier retraso tangible en el procesamiento pueda tener un impacto negativo en el funcionamiento diario de la empresa.

Los sistemas de OLTP están diseñados para procesar y almacenar de forma eficaz las transacciones, así como para consultar los datos transaccionales. El objetivo de procesar y almacenar eficazmente las transacciones individuales por parte de un sistema de OLTP se logra parcialmente mediante la normalización de datos (es decir, dividir los datos en fragmentos más pequeños que sean menos redundantes). La eficacia se debe a que permite que el sistema de OLTP procese grandes cantidades de transacciones de forma independiente y evita el procesamiento adicional necesario para mantener la integridad de los datos en presencia de datos redundantes.

## <a name="challenges"></a>Desafíos
La implementación y el uso de un sistema de OLTP pueden crear algunos problemas:

- Los sistemas de OLTP no siempre son buenos para controlar agregados en grandes cantidades de datos, aunque hay excepciones, como una solución basada en SQL Server bien planeada. Los análisis de los datos, que se basan en cálculos agregados de millones de transacciones individuales, hacen un uso muy intensivo de los recursos en un sistema de OLTP. Pueden tardar en ejecutarse y puede provocar una ralentización porque bloqueen otras transacciones de la base de datos.
- Si se realizan informes y análisis de los datos que estén muy normalizados, las consultas tienden a ser complejas, ya que la mayor parte de ellas tienen que anular la normalización de los datos mediante réplicas. Además, las convenciones de nomenclatura de los objetos de base de datos en los sistemas de OLTP tienden a ser breves y concisas. El aumento de la normalización, junto con unas convenciones de nomenclatura breves, hacen que sea difícil para los usuarios empresariales realizar consultas en los sistemas de OLTP sin la ayuda de un DBA o desarrollador de datos.
- El almacenamiento del historial de transacciones de forma indefinida y el almacenamiento de demasiados datos en cualquier tabla puede provocar una ralentización del rendimiento de las consultas, en función del número de transacciones almacenadas. La solución habitual consiste en mantener una ventana de tiempo relevante (por ejemplo, el año fiscal actual) en el sistema de OLTP y descargar los datos históricos a otros sistemas, como un data mart o un [almacenamiento de datos](../technology-choices/data-warehouses.md).

## <a name="oltp-in-azure"></a>OLTP en Azure

Aplicaciones como los sitios web hospedados en [App Service Web Apps](/azure/app-service/app-service-web-overview), REST API que se ejecutan en App Service o las aplicaciones de escritorio o móviles se comunican con el sistema de OLTP normalmente a través de una REST API intermediaria.

En la práctica, la mayoría de las cargas de trabajo no son OLTP puras. Tiende a haber también un [componente analítico](../scenarios/online-analytical-processing.md). Además, hay una creciente demanda de informes en tiempo real, como los informes activos en el sistema operativo. Esto también se denomina HTAP (procesamiento transaccional y analítico híbrido). Para más información, consulte [Choosing an OLAP data store in Azure](../technology-choices/olap-data-stores.md) (Elección de un almacén de datos OLAP en Azure).

## <a name="technology-choices"></a>Opciones de tecnología

Almacenamiento de datos:

- [Azure SQL Database](/azure/sql-database/)
- [SQL Server en una máquina virtual de Azure](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-server-iaas-overview?toc=%2Fazure%2Fvirtual-machines%2Fwindows%2Ftoc.json)
- [Azure Database for MySQL](/azure/mysql/)
- [Azure Database para PostgreSQL](/azure/postgresql/)

Para más información, consulte [Elección de un almacén de datos de OLTP](../technology-choices/oltp-data-stores.md).

Orígenes de datos:

- [App Service](/azure/app-service/)
- [Mobile Apps](/azure/app-service-mobile/)

