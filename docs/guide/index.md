---
layout: LandingPage
ms.topic: landing-page
ms.date: 08/30/2018
ms.openlocfilehash: a1cac753d384c0fee0af204cddaeea1e63213b9f
ms.sourcegitcommit: ae8a1de6f4af7a89a66a8339879843d945201f85
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/31/2018
ms.locfileid: "43325866"
---
# <a name="azure-application-architecture-guide"></a>Guía de la arquitectura de aplicaciones en Azure

Esta guía presenta un enfoque estructurado para diseñar aplicaciones de Azure que sean escalables, resistentes y con una alta disponibilidad. Se basa en prácticas probadas que hemos aprendido a través de las interacciones con los clientes.

## <a name="introduction"></a>Introducción

La nube está cambiando la forma en que se diseñan las aplicaciones. En lugar de ser monolitos, las aplicaciones se descomponen en servicios menores y descentralizados. Estos servicios se comunican a través de API o mediante el uso de eventos o de mensajería asincrónica. Las aplicaciones se escalan horizontalmente, agregando nuevas instancias, tal y como exigen las necesidades. 

Estas tendencias agregan nuevos desafíos. El estado de las aplicaciones se distribuye. Las operaciones se realizan en paralelo y de forma asincrónica. El sistema como un todo debe ser resistente cuando se producen errores. Las implementaciones deben estar automatizadas y ser predecibles. La supervisión y la telemetría son fundamentales para obtener una visión general del sistema. La Guía de la arquitectura de aplicaciones de Azure está diseñada para ayudarle a sortear estos cambios. 

<table>
<thead>
    <tr><th>Local tradicional</th><th>Nube moderna</th></tr>
</thead>
<tbody>
<tr><td>Monolítica y centralizada<br/>
Diseño para una escalabilidad predecible<br/>
Base de datos relacional<br/>
Coherencia máxima<br/>
Procesamiento serie y sincronizado<br/>
Diseño para evitar errores (MTBF)<br/>
Actualizaciones grandes ocasionales<br/>
Administración manual<br/>
Servidores de copo de nieve</td>
<td>
Descompuesta y descentralizada<br/>
Diseño para escalado flexible<br/>
Persistencia de Polyglot (combinación de tecnologías de almacenamiento)<br/>
Coherencia final<br/>
Procesamiento asincrónico y paralelo<br/>
Diseño para errores (MTTR)<br/>
Pequeñas actualizaciones frecuentes<br/>
Administración automatizada<br/>
Infraestructura inmutable<br/>
</td>
</tbody>
</table>

Esta guía está destinada a los arquitectos de aplicaciones, los desarrolladores y los equipos de operaciones. No es una guía de procedimientos para usar servicios individuales de Azure. Después de leerla, comprenderá los patrones arquitectónicos y las prácticas recomendadas que se aplican al crear en la plataforma de nube de Azure. También puede descargar una [versión de la guía para libro electrónico][ebook].

## <a name="how-this-guide-is-structured"></a>Cómo se estructura esta guía

La Guía de la arquitectura de aplicaciones de Azure se organiza como una serie de pasos, desde la arquitectura y el diseño a la implementación. Para cada paso, hay instrucciones auxiliares que le ayudarán con el diseño de la arquitectura de la aplicación.

### <a name="architecture-styles"></a>Estilos de arquitectura

La primera decisión es la más importante. ¿Qué tipo de arquitectura va a crear? Puede ser una arquitectura de microservicios, una aplicación con n niveles más tradicional o una solución de macrodatos. Identificamos varios estilos de arquitectura distintos. Cada uno presenta ventajas y desafíos.

Más información:

- [Estilos de arquitectura](./architecture-styles/index.md)

### <a name="technology-choices"></a>Opciones de tecnología

Dos opciones de tecnología deben decidirse en una fase temprana, ya que afectan a toda la arquitectura. Se trata de la elección del servicio de proceso y de los almacenes de datos. *Proceso* hace referencia al modelo de hospedaje para los recursos informáticos en los que las aplicaciones se ejecutan. Los *almacenes de datos* incluyen las bases de datos y también el almacenamiento destinado a las colas de mensajes, las memorias caché, los datos de registros y cualquier otro que una aplicación pueda conservar en algún tipo de almacenamiento. 

Más información:

- [Selección de un servicio de proceso](./technology-choices/compute-overview.md)
- [Selección de un almacén de datos](./technology-choices/data-store-overview.md)

### <a name="design-principles"></a>Principios de diseño

Hemos identificado diez principios de diseño de alto nivel para que la aplicación sea más escalable, resistente y administrable. Estos principios de diseño se aplican a todos los estilos de arquitectura. Durante el proceso de diseño, se deben tener en cuenta estos diez principios de diseño de alto nivel. Siga el mejor conjunto de procedimientos recomendados para los aspectos específicos de la arquitectura como, por ejemplo, el escalado automático, el almacenamiento en caché, la partición de datos, el diseño de API, etc.

Más información:

- [Principios de diseño](./design-principles/index.md)


### <a name="quality-pillars"></a>Fundamentos de calidad

Una aplicación correcta en la nube se centrará en cinco fundamentos de calidad del software: la escalabilidad, la disponibilidad, la resistencia, la administración y la seguridad. Utilice nuestras listas de comprobación de la revisión del diseño para revisar la arquitectura de acuerdo con estos fundamentos de calidad.

- [Fundamentos de calidad](./pillars.md)


[ebook]: https://azure.microsoft.com/campaigns/cloud-application-architecture-guide/
