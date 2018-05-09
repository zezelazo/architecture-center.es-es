---
title: Estilos de arquitectura
description: Estilos comunes de arquitectura para aplicaciones en la nube
layout: LandingPage
ms.openlocfilehash: e647d1a0f3305e7754859e5ab8a9a3b46c3d4fb6
ms.sourcegitcommit: d08f6ee27e1e8a623aeee32d298e616bc9bb87ff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2018
---
# <a name="architecture-styles"></a>Estilos de arquitectura

Un *estilo de arquitectura* es una familia de arquitecturas que comparten determinadas características. Por ejemplo, [de n niveles][n-tier] es un estilo de arquitectura común. Últimamente, las [arquitecturas de microservicios][microservices] se han empezado a hacer más populares. Los estilos de arquitectura no requieren el uso de tecnologías concretas, pero algunas tecnologías son adecuadas para ciertas arquitecturas. Por ejemplo, los contenedores son una opción natural para los microservicios.  

Hemos identificado un conjunto de estilos de arquitectura que se suelen encontrar en aplicaciones en la nube. El artículo para cada estilo incluye:

- Una descripción y un diagrama lógico del estilo.
- Recomendaciones sobre cuándo elegir este estilo.
- Ventajas, desafíos y procedimientos recomendados.
- Una implementación recomendada mediante los servicios de Azure pertinentes.


## <a name="a-quick-tour-of-the-styles"></a>Un recorrido rápido por los estilos   

Esta sección ofrece un rápido recorrido por los estilos de arquitectura que hemos identificado, junto con algunas consideraciones de alto nivel para su uso. Conozca más información en los temas vinculados.

### <a name="n-tier"></a>N niveles

<img src="./images/n-tier-sketch.svg" style="float:left; margin-top:6px;"/>

**[N niveles][n-tier]** es una arquitectura tradicional para aplicaciones empresariales. Las dependencias se administran mediante la división de la aplicación en *capas* que realizan funciones lógicas como presentaciones, lógica de negocios y acceso a datos. Una capa solo puede llamar a las capas que se encuentran por debajo de ella. Sin embargo, este sistema de capas horizontales puede tener otros efectos. Puede ser difícil introducir cambios en una parte de la aplicación sin tocar el resto de esta. Eso hace que las actualizaciones frecuentes sean un problema que limita la frecuencia con la que se agregan nuevas características.

La arquitectura de n niveles es una opción habitual para migrar aplicaciones existentes que ya usan una arquitectura en capas. Por ese motivo, la arquitectura de n niveles es la que se encuentra más frecuentemente en soluciones de infraestructura como servicio (IaaS) o en aplicaciones que usan una combinación de IaaS y servicios administrados. 

### <a name="web-queue-worker"></a>Web-Cola-Trabajo

<img src="./images/web-queue-worker-sketch.svg" style="float:left; margin-top:6px;"/>

Para una solución puramente PaaS, considere la posibilidad de usar una arquitectura **[Web-Cola-Trabajo](./web-queue-worker.md)**. En este estilo, la aplicación tiene un front-end web que controla las solicitudes HTTP y un trabajo de back-end que realiza tareas de uso intensivo de la CPU u operaciones de larga duración. El front-end se comunica con el trabajo a través de una cola de mensajes asincrónicos. 

La arquitectura Web-Cola-Trabajo es adecuada para dominios relativamente sencillos con algunas tareas que consumen muchos recursos. Al igual que en la de n niveles, la arquitectura es fácil de entender. El uso de servicios administrados simplifica la implementación y las operaciones. Pero con un dominio complejo, puede que sea difícil administrar las dependencias. El front-end y el trabajo se pueden convertir fácilmente en componentes grandes y monolíticos que son difíciles de mantener y actualizar. Al igual que sucede con la de n niveles, esto puede reducir la frecuencia de las actualizaciones y limitar la innovación.

### <a name="microservices"></a>Microservicios

<img src="./images/microservices-sketch.svg" style="float:left; margin-top:6px;"/>

Si la aplicación tiene un dominio más complejo, considere la posibilidad de pasar a una arquitectura de **[Microservicios][microservices]**. Una aplicación de microservicios se compone de muchos servicios pequeños e independientes. Cada servicio implementa una sola función empresarial. Los servicios están acoplados de forma flexible y se comunican a través de contratos de API.

Un pequeño equipo de desarrollo con dedicación puede compilar cada servicio. Los servicios individuales se pueden implementar sin necesidad de mucha coordinación entre los equipos, lo cual fomenta unas actualizaciones frecuentes. Una arquitectura de microservicios es más compleja a la hora de compilar y administrar que la de n niveles o la de web-cola-trabajo. Requiere un desarrollo perfeccionado y una cultura de DevOps. Pero si se hace correctamente, este estilo puede dar lugar una velocidad de lanzamiento mayor, una innovación más rápida y una arquitectura más resistente. 

### <a name="cqrs"></a>CQRS

<img src="./images/cqrs-sketch.svg" style="float:left; margin-top:6px;"/>

El estilo **[CQRS](./cqrs.md)** (Segregación de responsabilidades de consultas y comandos) separa las operaciones de lectura y escritura en modelos independientes. Esto permite aislar las partes del sistema que actualizan los datos de las partes que leen los datos. Además, las operaciones de lectura se pueden ejecutar en una vista materializada que esté físicamente separada de la base de datos de escritura. Eso le permite escalar las cargas de trabajo de lectura y escritura independientemente, y optimizar la vista materializada para las consultas.

El estilo CQRS resulta idóneo cuando se aplica a un subsistema de una arquitectura mayor. De forma general, no debería imponerla en toda la aplicación, ya que eso simplemente creará una complejidad innecesaria. Debería tenerlo en cuenta para dominios colaborativos en los que muchos usuarios acceden a los mismos datos.

### <a name="event-driven-architecture"></a>Arquitectura basada en eventos 

<img src="./images/event-driven-sketch.svg" style="float:left; margin-top:6px;"/>

Las **[arquitecturas basadas en eventos](./event-driven.md)**  usan un modelo de publicación-suscripción (pub-sub), en el que los productores publican eventos y los consumidores de suscriben a ellos. Los productores son independientes de los consumidores y estos, a su vez, son independientes entre sí. 

Considere la posibilidad de implementar una arquitectura basada en eventos para las aplicaciones que ingieren y procesan un gran volumen de datos con una latencia muy baja como en el caso de las soluciones de IoT. Este estilo también es útil cuando diferentes subsistemas deben realizar distintos tipos de procesamiento en los mismos datos de evento.

<br />

### <a name="big-data-big-compute"></a>Big Data, Big Compute

**[Big Data](./big-data.md)** y **[Big Compute](./big-compute.md)** son los estilos de arquitectura especializados en cargas de trabajo que resultan más adecuados para determinados perfiles específicos. Big Data permite dividir un conjunto de datos muy grande en fragmentos, realizando un procesamiento paralelo en todo el conjunto, con fines de análisis y creación de informes. Big compute, también denominada informática de alto rendimiento (HPC), realiza cálculos en paralelo en un gran número (miles) de núcleos. Los dominios incluyen simulaciones, modelado y representaciones 3-D.

## <a name="architecture-styles-as-constraints"></a>Estilos de arquitectura como restricciones

Un estilo de arquitectura puede crear restricciones en el diseño, incluido en el conjunto de elementos que pueden aparecer, y en las relaciones permitidas entre los elementos. Las restricciones contribuyen a dar "forma" a una arquitectura restringiendo el universo de opciones. Cuando una arquitectura cumple con las restricciones de un estilo determinado, surgen determinadas propiedades deseables. 

Por ejemplo, entre las restricciones de los microservicios se incluyen: 

- Un servicio representa una única responsabilidad. 
- Cada servicio es independiente de los demás. 
- Los datos son privados para el servicio al que pertenecen. Los servicios no comparten datos.

Mediante la adhesión a estas restricciones, lo que surge es un sistema en el que los servicios se pueden implementar independientemente, los errores se aíslan, las actualizaciones frecuentes son posibles y resulta fácil introducir nuevas tecnologías en la aplicación.

Antes de elegir un estilo de arquitectura, asegúrese de que comprende los principios subyacentes y las restricciones de ese estilo. En caso contrario, puede acabar con un diseño que se adapta al estilo en un nivel superficial pero que no utiliza todas las posibilidades de ese estilo. También es importante ser práctico. A veces es mejor ser menos exigentes con una restricción en lugar de incidir en la pureza de una arquitectura.


En la tabla siguiente se resume cómo administra las dependencias cada estilo y qué tipos de dominios son los más adecuados para cada uno.

| Estilo de arquitectura |  Administración de dependencias | Tipo de dominio |
|--------------------|------------------------|-------------|
| N niveles | Niveles horizontales divididos por subred | Dominio empresarial tradicional. La frecuencia de las actualizaciones es baja. |
| Web-Cola-Trabajo | Trabajos de front-end y back-end, desacoplados mediante mensajería asincrónica. | Dominio relativamente sencillo con algunas tareas de uso intensivo de recursos. |
| Microservicios | Servicios descompuestos verticalmente (funcionalmente) que se llaman mutuamente mediante API. | Dominio complicado. Actualizaciones frecuentes. |
| CQRS | Segregación de lectura y escritura. El esquema y la escala se optimizan por separado. | Dominios colaborativos donde una gran cantidad de usuarios acceden a los mismos datos. |
| Arquitectura basada en eventos. | Productores y consumidores. Vista independiente por cada subsistema. | IoT y sistemas en tiempo real |
| Macrodatos | Divide un conjunto de datos grande en fragmentos pequeños. Procesamiento en paralelo en los conjuntos de datos locales. | Análisis de datos por lotes y en tiempo real. Análisis predictivo mediante Machine Learning. |
| Big Compute| Asignación de datos a miles de núcleos. | Dominios con procesos intensivos como simulaciones. |


## <a name="consider-challenges-and-benefits"></a>Análisis de desafíos y ventajas

Las restricciones también pueden conllevar desafíos, por lo que es importante comprender las ventajas y desventajas de adoptar cualquiera de estos estilos. ¿Son superiores las ventajas del estilo de arquitectura a los posibles desafíos *para este subdominio y el contexto enlazado?*. 

Estos son algunos de los tipos de desafíos que debe tener en cuenta al seleccionar un estilo de arquitectura:

- **Complejidad**. ¿Está justificada la complejidad de la arquitectura para el dominio? O, por el contrario, ¿es demasiado simple? En ese caso, se arriesga a terminar con un sistema "[sin arquitectura alguna observable][ball-of-mud]", ya que esta no le ayuda a administrar las dependencias correctamente.

- **Mensajería asincrónica y coherencia final**. La mensajería asincrónica puede usarse para desacoplar servicios y aumentar la confiabilidad (porque se pueden recuperar los mensajes) y la escalabilidad. No obstante, esto también genera desafíos como la semántica "siempre una vez" y la coherencia final.

- **Comunicación entre servicios**. Cuando descompone una aplicación en servicios independientes, existe el riesgo de que la comunicación entre estos provoque una latencia inaceptable o cree una congestión en la red (por ejemplo, en una arquitectura de microservicios). 

- **Manejabilidad**. ¿Es difícil administrar la aplicación y supervisar e implementar actualizaciones, etc?


[ball-of-mud]: https://en.wikipedia.org/wiki/Big_ball_of_mud
[microservices]: ./microservices.md
[n-tier]: ./n-tier.md
