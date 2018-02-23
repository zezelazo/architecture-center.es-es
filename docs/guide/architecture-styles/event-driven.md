---
title: Estilo de arquitectura basada en eventos
description: Describe las ventajas, las dificultades y los procedimientos recomendados para las arquitecturas IoT y basadas en eventos en Azure.
author: MikeWasson
ms.openlocfilehash: 3289bf784b02d62e3d0c1a29b4839c9be3501134
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/23/2018
---
# <a name="event-driven-architecture-style"></a>Estilo de arquitectura basada en eventos

Una arquitectura basada en eventos consta de **productores de eventos** que generan un flujo de eventos, y **consumidores de eventos** que escuchan los eventos. 

![](./images/event-driven.svg)

Los eventos se entregan casi en tiempo real, de modo que los consumidores pueden responder inmediatamente a los eventos cuando se producen. Los productores se desconectan de los consumidores &mdash; un productor no sabe que los consumidores están escuchando. Los consumidores también se desconectan entre sí, y cada consumidor ve todos los eventos. Esto difiere de un patrón [Competing Consumers][competing-consumers], donde los consumidores extraen los mensajes de una cola y los mensajes solo se procesan una vez (suponiendo que no haya errores). En algunos sistemas, como IoT, los eventos se deben ingerir en volúmenes muy altos.

Una arquitectura basada en eventos puede usar un modelo pub/sub o un modelo de flujo de eventos. 

- **Pub/sub**: la infraestructura de mensajería mantiene un seguimiento de las suscripciones. Cuando se publica un evento, se envía el evento a cada suscriptor. Después de que se recibe un evento, no se puede reproducir, y los nuevos suscriptores no ven el evento. 

- **Flujo de eventos**: los eventos se escriben en un registro. Los eventos siguen un orden estricto (dentro de una partición) y son duraderos. Los clientes no se suscriben al flujo, sino que un cliente puede leer desde cualquiera de sus partes. El cliente es responsable de avanzar su posición en el flujo. Esto significa que un cliente puede unirse en cualquier momento y puede reproducir los eventos.

En el lado del consumidor, hay algunas variaciones comunes:

- **Procesamiento sencillo de eventos**. Un evento desencadena inmediatamente una acción en el consumidor. Por ejemplo, podría usar Azure Functions con un desencadenador de Service Bus para que una función se ejecute cada vez que se publica un mensaje en un tema de Service Bus.

- **Procesamiento de eventos complejos**. Un consumidor procesa una serie de eventos, en busca de patrones en los datos de eventos, mediante una tecnología como Azure Stream Analytics o Apache Storm. Por ejemplo, podría agregar las lecturas de un dispositivo insertado durante una ventana de tiempo y generar una notificación si la media móvil supera un umbral determinado. 

- **Procesamiento de flujo de eventos**. Use una plataforma de flujo de datos, como Azure IoT Hub o Apache Kafka, como canalización para ingerir eventos y suministrarlos a procesadores de flujo. Los procesadores de flujos sirven para procesar o transformar el flujo. Puede haber varios procesadores de flujo para diferentes subsistemas de la aplicación. Este enfoque es una buena opción para las cargas de trabajo de IoT.

El origen de los eventos puede ser externo con respecto al sistema, como dispositivos físicos en una solución de IoT. En ese caso, el sistema debe ser capaz de ingerir los datos según el volumen y el rendimiento que requiere el origen de datos.

En el diagrama lógico anterior, se muestra cada tipo de consumidor como un solo cuadro. En la práctica, es habitual tener varias instancias de un consumidor para evitar hacer que el consumidor se convierta en un único punto de error en el sistema. También podrían ser necesarias varias instancias para administrar el volumen y la frecuencia de los eventos. Además, un único consumidor podría procesar eventos en varios subprocesos. Esto puede crear dificultades si los eventos se deben procesar en orden o requieren la semántica exactly-once. Consulte [Minimizar la coordinación][minimize-coordination]. 

## <a name="when-to-use-this-architecture"></a>Cuándo utilizar esta arquitectura

- Varios subsistemas deben procesar los mismos eventos. 
- Procesamiento en tiempo real con retardo mínimo.
- Procesamiento de eventos complejos, como coincidencia de patrones o agregación durante ventanas de tiempo.
- Gran volumen y alta velocidad de datos, como IoT.

## <a name="benefits"></a>Ventajas

- Se desvinculan productores y consumidores.
- Ninguna integración punto a punto. Es fácil agregar nuevos consumidores al sistema.
- Los consumidores pueden responder a eventos inmediatamente a medida que llegan. 
- Muy escalable y distribuida. 
- Los subsistemas tienen vistas independientes del flujo de eventos.

## <a name="challenges"></a>Desafíos

- Entrega garantizada. En algunos sistemas, especialmente en escenarios de IoT, es fundamental garantizar la entrega de los eventos.
- Procesamiento de eventos en orden o exactamente una vez. Cada tipo de consumidor normalmente se ejecuta en varias instancias, a fin de conseguir resistencia y escalabilidad. Esto puede suponer un desafío si se deben procesar los eventos en orden (dentro de un tipo de consumidor), o si la lógica de procesamiento no es idempotente.

## <a name="iot-architecture"></a>Arquitectura de IoT

Las arquitecturas basadas en eventos son básicas para las soluciones de IoT. En el siguiente diagrama se muestra una posible arquitectura lógica de IoT. En él se resaltan los componentes del flujo de eventos de la arquitectura.

![](./images/iot.png)

La **puerta de enlace de nube** ingiere eventos de dispositivo en el límite de nube, mediante un sistema de mensajería confiable y de baja latencia.

Los dispositivos pueden enviar eventos directamente a la puerta de enlace de nube, o a través de una **puerta de enlace de campo**. Una puerta de enlace de campo es un dispositivo o software especializado, que normalmente se coloca con los dispositivos, que recibe eventos y los reenvía a la puerta de enlace de nube. La puerta de enlace de campo también puede procesar previamente los eventos de dispositivo sin formato y realizar funciones como filtrado, agregación o transformación de protocolo.

Después de la ingesta, los eventos pasan por uno o varios **procesadores de flujo** que pueden enrutar los datos (por ejemplo, al almacenamiento) o realizar análisis u otras tareas de procesamiento.

Estos son algunos tipos comunes de procesamiento. (Ciertamente, esta lista no es exhaustiva).

- Escribir datos de eventos en almacenamiento en frío, para archivado o análisis por lotes.

- Análisis de ruta de acceso activa, que analiza el flujo de eventos (casi) en tiempo real para detectar anomalías, reconocer patrones durante ventanas de tiempo consecutivas o desencadenar alertas cuando se produce una condición especifica en el flujo. 

- Administrar tipos especiales de mensajes que no son de telemetría de dispositivos, tales como notificaciones y alarmas. 

- Machine Learning

Los cuadros que aparecen en gris sombreado muestran los componentes de un sistema IoT que no están directamente relacionados con el flujo de eventos, pero se incluyen aquí para ofrecer una visión completa.

- El **registro de dispositivos** es una base de datos de los dispositivos aprovisionados, incluidos los identificadores de dispositivo y habitualmente los metadatos de dispositivos, como la ubicación.

- La **API de aprovisionamiento** es una interfaz externa común para el aprovisionamiento y el registro de nuevos dispositivos.

- Algunas soluciones IoT permiten el envío de **mensajes de comando y control** a los dispositivos.

> En esta sección se ha presentado una vista de muy alto nivel de IoT y hay muchos matices y desafíos que se deben tener en cuenta. Para más información y obtener un análisis de una arquitectura de referencia, consulte [Microsoft Azure IoT Reference Architecture][iot-ref-arch] (Arquitectura de referencia de Microsoft Azure IoT) (descarga PDF).

 <!-- links -->

[competing-consumers]: ../../patterns/competing-consumers.md
[iot-ref-arch]: https://azure.microsoft.com/updates/microsoft-azure-iot-reference-architecture-available/
[minimize-coordination]: ../design-principles/minimize-coordination.md


