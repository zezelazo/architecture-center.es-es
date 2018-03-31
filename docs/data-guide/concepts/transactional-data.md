---
title: Datos transaccionales
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: b7fdbb403d2a438ebc59e40ef58ed8067489dddc
ms.sourcegitcommit: 943e671a8d522cef5ddc8c6e04848134b03c2de4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/05/2018
---
# <a name="transactional-data"></a>Datos transaccionales

Los datos transaccionales son información que realiza un seguimiento de las interacciones relacionadas con las actividades de una organización. Estas interacciones normalmente son transacciones comerciales, tales como pagos recibidos de los clientes, pagos realizados a los proveedores, movimiento de productos en el inventario, pedidos obtenidos o servicios entregados. Los eventos transaccionales, que representan a las transacciones, normalmente contienen una dimensión de tiempo, algunos valores numéricos y referencias a otros datos. 

Normalmente, las transacciones deben ser *atómicas* y *coherentes*. Atomicidad significa que una transacción completa siempre se realiza, correctamente o con error, como una unidad de trabajo y nunca se deja en un estado medio completado. Si no se puede completar una transacción, el sistema de base de datos debe revertir todos los pasos que se han hecho como parte de esa transacción. En los RDBMS tradicionales, esta reversión sucede automáticamente si no se puede finalizar una transacción. Coherencia significa que las transacciones dejan siempre los datos en un estado válido. (Estas son descripciones muy informales de atomicidad y coherencia. Hay definiciones más formales de estas propiedades, como [ACID](https://en.wikipedia.org/wiki/ACID)).

Las bases de datos transaccionales posibilitan una coherencia alta de las transacciones mediante el uso de diversas estrategias de bloqueo, como el bloqueo pesimista, para asegurarse de que todos los datos son altamente coherentes dentro del contexto de la empresa, para todos los usuarios y procesos. 

La arquitectura de implementación más común que utiliza datos transaccionales es el nivel de almacén de datos en una arquitectura de 3 niveles. Una arquitectura de 3 niveles normalmente consta de un nivel de presentación, un nivel de lógica de negocios y un nivel de almacén de datos. Una arquitectura de implementación relacionada es la arquitectura de [n niveles](/azure/architecture/guide/architecture-styles/n-tier), que puede tener varios niveles intermedios para el control de la lógica de negocios.

![Ejemplo de una aplicación de 3 niveles](./images/three-tier-application.png)

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

## <a name="see-also"></a>Otras referencias

[Procesamiento de transacciones en línea](../scenarios/online-transaction-processing.md)
