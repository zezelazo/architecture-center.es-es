---
title: Uso del mejor almacén de datos para el trabajo
description: Elija la tecnología de almacenamiento que encaje mejor con sus datos y el modo en que se utilizarán.
author: MikeWasson
ms.date: 08/30/2018
ms.openlocfilehash: 25839f5a749881f415c923db5497984d32b8ac91
ms.sourcegitcommit: ae8a1de6f4af7a89a66a8339879843d945201f85
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/31/2018
ms.locfileid: "43326098"
---
# <a name="use-the-best-data-store-for-the-job"></a>Uso del mejor almacén de datos para el trabajo

## <a name="pick-the-storage-technology-that-is-the-best-fit-for-your-data-and-how-it-will-be-used"></a>Elija la tecnología de almacenamiento que encaje mejor con sus datos y el modo en que se utilizarán.

Lejos quedan los días en los que bastaba con colocar todos los datos en una base de datos SQL relacional de gran tamaño. Las bases de datos relacionales son muy buenas en aquello que hacen: ofrecer garantías de ACID para las transacciones sobre los datos relacionales. Pero incluyen algunos inconvenientes:

- Las consultas pueden requerir combinaciones costosas.
- Los datos se deben normalizar y se ajustan a un esquema predefinido (esquema en escritura).
- La contención de bloqueo puede afectar al rendimiento.

En cualquier solución de gran tamaño, es probable que una tecnología sencilla de almacén de datos no satisfaga todas sus necesidades. Las alternativas a las bases de datos relacionales incluyen almacenes de clave/valor, bases de datos de documentos, bases de datos de motores de búsqueda, bases de datos de series de tiempo, bases de datos de familias de columnas y bases de datos de gráficos. Cada una tiene ventajas y desventajas, y diferentes tipos de datos encajan de manera más natural en una u otra. 

Por ejemplo, podría almacenar un catálogo de productos en una base de datos de documentos, como Cosmos DB, lo que permite un esquema flexible. En ese caso, cada descripción de producto es un documento independiente. Para las consultas en todo el catálogo, puede indexar el catálogo y almacenar el índice en Azure Search. El inventario de productos puede ir en una base de datos SQL, porque esos datos requieren garantías ACID.

Recuerde que los datos incluyen más que simplemente los datos de aplicación conservados. También contienen registros de aplicación, eventos, mensajes y memorias caché.

## <a name="recommendations"></a>Recomendaciones

**No utilice una base de datos relacional para todo**. Considere el uso de otros almacenes de datos cuando sea apropiado. Consulte [Elección del almacén de datos apropiado][data-store-overview].

**Adopte la persistencia de Polyglot**. En cualquier solución de gran tamaño, es probable que una tecnología sencilla de almacén de datos no satisfaga todas sus necesidades. 

**Tenga en cuenta el tipo de datos**. Por ejemplo, coloque los datos transaccionales en SQL, coloque los documentos JSON en una base de datos de documentos, coloque los datos de telemetría en una base de datos de series de tiempo, coloque los registros de aplicaciones en Elasticsearch y coloque los blobs en Azure Blob Storage.

**Elija disponibilidad sobre (sólida) coherencia**. El teorema de CAP implica que un sistema distribuido debe compensar entre disponibilidad y coherencia. (Las particiones de la red, el otro segmento del teorema de CAP, no se pueden evitar nunca por completo). A menudo, puede lograr una mayor disponibilidad mediante el uso de un modelo de *coherencia final*. 

**Tenga en cuenta el conjunto de conocimientos del equipo de desarrollo**. El uso de Polyglot entraña ventajas, pero es posible acabar abusando de ello. La adopción de una nueva tecnología de almacenamiento de datos requiere un nuevo conjunto de conocimientos. El equipo de desarrollo debe aprender a sacar el máximo partido de la tecnología. Es necesario comprender los patrones de uso correspondientes, cómo optimizar las consultas, ajustar el rendimiento, etc. Tenga esto en cuenta al considerar las tecnologías de almacenamiento. 

**Use transacciones de compensación**. Un efecto secundario de la persistencia de Polyglot es que una única transacción puede escribir datos en varios almacenes. Si algo va mal, utilice las transacciones de compensación para deshacer todos los pasos que ya haya completado.

**Fíjese en los contextos enlazados**. *Contexto enlazado* es un término procedente del diseño orientado al dominio. Un contexto enlazado es un límite explícito alrededor de un modelo de dominio y define a qué partes del dominio se aplica el modelo. Idealmente, un contexto enlazado se asigna a un subdominio del dominio empresarial. Los contextos enlazados en el sistema son un lugar idóneo para considerar la persistencia de Polyglot. Por ejemplo, los "productos" pueden aparecer en el subdominio del catálogo de productos y el subdominio del inventario de productos, pero es muy probable que estos dos subdominios tengan requisitos diferentes para almacenar, actualizar y consultar los productos.

[data-store-overview]: ../technology-choices/data-store-overview.md