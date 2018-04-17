---
layout: LandingPage
ms.topic: landing-page
ms.openlocfilehash: 530844a0d3b1256cec807e7bad509a40dca304f6
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2018
---
# <a name="azure-application-architecture-guide"></a>Guía de la arquitectura de aplicaciones en Azure

Esta guía presenta un enfoque estructurado para diseñar aplicaciones de Azure que sean escalables, resistentes y con una alta disponibilidad. Se basa en prácticas probadas que hemos aprendido a través de las interacciones con los clientes.

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

**[Estilos de arquitectura][arch-styles]**. La primera decisión es la más importante. ¿Qué tipo de arquitectura va a crear? Puede ser una arquitectura de microservicios, una aplicación con n niveles más tradicional o una solución de macrodatos. Identificamos siete estilos de arquitectura distintos. Cada uno presenta ventajas y desafíos.

> &#10148; [Las arquitecturas de referencia de Azure][ref-archs] muestran implementaciones recomendadas en Azure, junto con consideraciones sobre escalabilidad, disponibilidad, capacidad de administración y seguridad. La mayoría también incluye plantillas de Resource Manager que se pueden implementar.

**[Opciones de tecnología][technology-choices]**. Dos opciones de tecnología deben decidirse en una fase temprana, ya que afectan a toda la arquitectura. Se trata de la elección de las tecnologías de almacenamiento y de proceso. El término *proceso* hace referencia al modelo de hospedaje para los recursos informáticos en los que las aplicaciones se ejecutan. El almacenamiento incluye las bases de datos y también el destinado a las colas de mensajes, las memorias caché, los datos IoT, los datos de registro no estructurados y cualquier otro que una aplicación pueda conservar en algún tipo de almacenamiento. 

> &#10148; [Las opciones de proceso] [ compute-options] y las [opciones de almacenamiento][storage-options] especifican criterios de comparación detallados para seleccionar los servicios de proceso y de almacenamiento.

**[Principios de diseño][design-principles]**. Durante el proceso de diseño, se deben tener en cuenta estos diez principios de diseño de alto nivel. 

> &#10148; Los artículos sobre [procedimientos recomendados][best-practices] proporcionan una guía específica en áreas como el escalado automático, el almacenamiento en caché, la creación de particiones de datos, el diseño de API y otros.   

**[Pilares][pillars]**. Una aplicación correcta en la nube se centrará en estos cinco fundamentos de calidad del software: la escalabilidad, la disponibilidad, la resistencia, la administración y la seguridad. 

> &#10148; Utilice nuestras [listas de comprobación de la revisión del diseño] [checklists] para revisar el diseño de acuerdo con estos fundamentos de calidad. 

**[Patrones de diseño en la nube][patterns]**. Estos patrones de diseño son útiles para crear aplicaciones confiables, escalables y seguras en Azure. Cada patrón describe un problema, un patrón que aborda el problema y un ejemplo basado en Azure.

> &#10148; Vea la sección completa [Catálogo de patrones de diseño en la nube](../patterns/index.md).


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

