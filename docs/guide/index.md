---
layout: LandingPage
ms.topic: landing-page
ms.openlocfilehash: 80cb7fde0694257a5c413b702505e27f18aed8d3
ms.sourcegitcommit: d702b4d27e96e7a5a248dc4f2f0e25cf6e82c134
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/23/2018
ms.locfileid: "31771693"
---
# <a name="azure-application-architecture-guide"></a>Guía de la arquitectura de aplicaciones en Azure

Esta guía presenta un enfoque estructurado para diseñar aplicaciones de Azure que sean escalables, resistentes y con una alta disponibilidad. Se basa en prácticas probadas que hemos aprendido a través de las interacciones con los clientes.

<br/>

<img src="./images/guide-steps.svg" style="max-width:800px;"/>

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

- [Estilos de arquitectura][arch-styles]
- [Arquitecturas de referencia de Azure][ref-archs]

### <a name="technology-choices"></a>Opciones de tecnología

Dos opciones de tecnología deben decidirse en una fase temprana, ya que afectan a toda la arquitectura. Se trata de la elección del servicio de proceso y de los almacenes de datos. *Proceso* hace referencia al modelo de hospedaje para los recursos informáticos en los que las aplicaciones se ejecutan. Los *almacenes de datos* incluyen las bases de datos y también el almacenamiento destinado a las colas de mensajes, las memorias caché, los datos de registros y cualquier otro que una aplicación pueda conservar en algún tipo de almacenamiento. 

Más información:

- [Selección de un servicio de proceso](./technology-choices/compute-overview.md)
- [Selección de un almacén de datos](./technology-choices/data-store-overview.md)

### <a name="design-principles"></a>Principios de diseño

Hemos identificado diez principios de diseño de alto nivel para que la aplicación sea más escalable, resistente y administrable. Estos principios de diseño se aplican a todos los estilos de arquitectura. Durante el proceso de diseño, se deben tener en cuenta estos diez principios de diseño de alto nivel. Siga el mejor conjunto de procedimientos recomendados para los aspectos específicos de la arquitectura como, por ejemplo, el escalado automático, el almacenamiento en caché, la partición de datos, el diseño de API, etc.

Más información:

- [Principios de diseño para las aplicaciones de Azure][design-principles]
- [Procedimientos recomendados al compilar para la nube][best-practices]

### <a name="quality-pillars"></a>Fundamentos de calidad

Una aplicación correcta en la nube se centrará en cinco fundamentos de calidad del software: la escalabilidad, la disponibilidad, la resistencia, la administración y la seguridad. Utilice nuestras listas de comprobación de la revisión del diseño para revisar la arquitectura de acuerdo con estos fundamentos de calidad.

Más información:

- [Fundamentos de calidad del software][pillars]
- [Listas de comprobación de la revisión de diseño][checklists] 

### <a name="cloud-design-patterns"></a>Modelos de diseño en la nube

Los modelos de diseño son soluciones generales a problemas habituales en el diseño del software. Hemos identificado un conjunto de modelos de diseño que son especialmente útiles al diseñar aplicaciones distribuidas para la nube.

Más información:

- [Catálogo de modelos de diseño en la nube](../patterns/index.md)


[arch-styles]: ./architecture-styles/index.md
[best-practices]: ../best-practices/index.md
[checklists]: ../checklist/index.md
[compute-options]: ./technology-choices/compute-comparison.md
[design-principles]: ./design-principles/index.md
[ebook]: https://azure.microsoft.com/campaigns/cloud-application-architecture-guide/
[patterns]: ../patterns/index.md?toc=/azure/architecture/guide/toc.json
[pillars]: ./pillars.md
[ref-archs]: ../reference-architectures/index.md
[storage-options]: ./technology-choices/data-store-comparison.md
[technology-choices]: ./technology-choices/index.md

