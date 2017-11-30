---
title: "Patrón Sidecar"
description: "Implemente componentes de una aplicación en un proceso o contenedor independientes para proporcionar aislamiento y encapsulación."
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: ec168009aa99f412c3f1222a1c404ea4ea5cb669
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2017
---
# <a name="sidecar-pattern"></a>Patrón Sidecar

Implemente componentes de una aplicación en un proceso o contenedor independientes para proporcionar aislamiento y encapsulación. Este patrón también puede habilitar las aplicaciones compuestas de tecnologías y componentes heterogéneos.

Este patrón se denomina *Sidecar* porque parece un sidecar acoplado a una motocicleta. En el patrón, el sidecar está acoplado a una aplicación primaria y proporciona funciones auxiliares para la aplicación. El sidecar también comparte el mismo ciclo de vida que la aplicación primaria, se crea y se retira junto con el elemento primario. El patrón Sidecar a veces se denomina patrón Sidekick y es un patrón jerárquico.

## <a name="context-and-problem"></a>Contexto y problema

A menudo las aplicaciones y lo servicios requieren una funcionalidad relacionada, como la supervisión, el registro, la configuración y los servicios de red. Estas tareas periféricas se pueden implementar como componentes o servicios independientes. 

Si están estrechamente integradas en la aplicación, se pueden ejecutar en el mismo proceso que la aplicación, que realiza un uso eficaz de los recursos compartidos. Sin embargo, esto también significa que no están bien aisladas y una interrupción en uno de estos componentes puede afectar a otros componentes o a toda la aplicación. Además, normalmente deben implementarse con el mismo idioma que la aplicación primaria. Como consecuencia, el componente y la aplicación tienen una gran dependencia entre ellos.

Si la aplicación se descompone en servicios, cada uno de ellos se puede crear con distintos idiomas y tecnologías. Aunque esto proporciona más flexibilidad, significa que cada componente tiene sus propias dependencias y requiere bibliotecas específicas del lenguaje para acceder tanto a la plataforma subyacente como a los recursos compartidos con la aplicación primaria. Además, la implementación de estas características como servicios independientes puede agregar latencia a la aplicación. La administración del código y las dependencias de estas interfaces específicas del lenguaje también puede agregar una considerable complejidad, especialmente en lo referente al hospedaje, la implementación y la administración.

## <a name="solution"></a>Solución

Coloque un conjunto de tareas cohesionadas con la aplicación principal, pero colóquelas en su propio proceso o contenedor, lo que proporciona una interfaz homogénea para los servicios de plataformas de varios lenguajes. 

![](./_images/sidecar.png)

Un servicio sidecar no necesariamente forma parte de la aplicación, pero está conectado a ella. Va allá donde vaya la aplicación primaria. Estos son tipos de procesos o servicios auxiliares que se implementan con la aplicación principal. En el caso de las motocicletas, el sidecar está acoplado a una motocicleta y cada motocicleta puede tener su propio sidecar. De la misma forma, un servicio sidecar comparte el destino de su aplicación primaria. Para cada instancia de la aplicación, se implementa una instancia del patrón Sidecar, y se hospeda junto a ella. 

Entre las ventajas de usar un patrón Sidecar se incluyen:

- Un patrón Sidecar es independiente de su aplicación principal en cuanto al entorno del runtime y al lenguaje de programación, por lo que no es preciso desarrollar un sidecar por lenguaje. 

- El sidecar puede acceder a los mismos recursos que la aplicación principal. Por ejemplo, un sidecar puede supervisar los recursos del sistema que usan tanto el sidecar como la aplicación principal. 

- Dada su proximidad a la aplicación principal, la latencia de la comunicación entre ellos no es significativa.

- Hasta en las aplicaciones que no proporcionan un mecanismo de extensibilidad se puede usar un sidecar para ampliar la funcionalidad, para lo que se adjunta como proceso propio proceso en el mismo host o subcontenedor que la aplicación principal.

A menudo, el patrón Sidecar se usa con contenedores y se conoce como contenedor sidecar o contenedor sidekick. 

## <a name="issues-and-considerations"></a>Problemas y consideraciones

- Tenga en cuenta el formato de implementación y empaquetado que va a usar para implementar servicios, procesos o contenedores. Los contenedores se ajustan especialmente bien al patrón Sidecar.
- Al diseñar un servicio sidecar, decida con cuidado el mecanismo de comunicación entre procesos. Pruebe a utilizar tecnologías independiente del lenguaje o del marco, a menos que los requisitos de rendimiento hagan que no sea práctico.
- Antes de colocar la funcionalidad en un servicio sidecar, plantéese si funcionaría mejor como un servicio independiente o un demonio más tradicional.
- Tenga también en cuenta si la funcionalidad se puede implementar como una biblioteca o mediante un mecanismo de extensión tradicional. Las bibliotecas específicas del lenguaje pueden tener un mayor nivel de integración y menor sobrecarga de red.

## <a name="when-to-use-this-pattern"></a>Cuándo se usa este patrón

Use este patrón en los siguientes supuestos:

- La aplicación principal utiliza un conjunto heterogéneo de lenguajes y marcos. Un componente que se encuentra en un servicio sidecar lo pueden usar aplicaciones escritas en otros lenguajes y que usen marcos diferentes.
- Un componente pertenece a un equipo remoto o a otra organización.
- Un componente o una característica deben encontrarse en el mismo host que la aplicación
- Necesita un servicio que comparta el ciclo de vida general de la aplicación principal, pero que se pueda actualizar de forma independiente.
- Necesita un control específico sobre los límites de recursos de un componente o recurso concreto. Por ejemplo, puede restringir la cantidad de memoria que usa un componente específico. Puede implementar el componente como sidecar y administrar el uso de la memoria independientemente de la aplicación principal.

Este patrón puede no ser adecuado:

- Cuando debe optimizarse la comunicación entre procesos. La comunicación entre una aplicación primaria y los servicios sidecar tiene cierta sobrecarga, en particular latencia en las llamadas. Es posible que esta contrapartida no sea aceptable para interfaces que emiten información.
- En el caso de aplicaciones pequeñas, en las que el costo, en términos de recursos, de implementar un servicio de sidecar por cada instancia no merece la pena, en comparación la ventaja del aislamiento.
- Cuando el servicio necesita escalar de forma diferente a las aplicaciones principales, o independientemente de ellas. Si es así, es posible que sea mejor implementar la característica como un servicio independiente.

## <a name="example"></a>Ejemplo

Este patrón sidecar se puede aplicar a muchos escenarios. Algunos ejemplos comunes:

- API de infraestructura. El equipo de desarrollo de la infraestructura crea un servicio que se implementa junto con cada aplicación, en lugar de una biblioteca de cliente específica del lenguaje para acceder a la infraestructura. El servicio se carga como un sidecar y proporciona una capa común para los servicios de infraestructura, incluidos el registro, los datos del entorno, el almacén de configuración, la detección, las comprobaciones de mantenimiento y los servicios de vigilancia. El sidecar también supervisa el entorno de host y el proceso (o contenedor) de la aplicación principal, y registra la información en un servicio centralizado.
- Administración de NGINX/HAProxy. Implemente NGINX con un servicio sidecar que supervise el estado del entorno y, después, actualice el archivo de configuración de NGINX y recicle el proceso cuando se necesite un cambio de estado.
- Sidecar Ambassador. Implemente un servicio [Ambassador][ambassador] como un sidecar. La aplicación llama a través del servicio Ambassador, que controla el registro de solicitudes, enrutamiento, la interrupción de circuitos y otras características relacionadas con la conectividad.
- Descarga de proxy. Coloque un proxy NGINX delante de una instancia de servicio de node.js para controlar que se sirve el contenido del archivo estático para el servicio.


## <a name="related-guidance"></a>Instrucciones relacionadas

- [Patrón Ambassador][ambassador]


[ambassador]: ./ambassador.md

