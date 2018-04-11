---
title: Elección de una tecnología de ingesta de mensajes en tiempo real
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 2e6578b779950b5ef11bda7b8ba1fb2e45e09f4e
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/23/2018
---
# <a name="choosing-a-real-time-message-ingestion-technology-in-azure"></a>Elección de una tecnología de ingesta de mensajes en tiempo real en Azure

El procesamiento en tiempo real trata sobre los flujos de datos que se capturan en tiempo real y se procesan con una latencia mínima. Muchas soluciones de procesamiento en tiempo real necesitan un almacén de ingesta de mensajes para que actúe como búfer de mensajes y para admitir el procesamiento de escalabilidad horizontal, la entrega confiable y otra semántica de puesta en cola de mensajes. 

## <a name="what-are-your-options-for-real-time-message-ingestion"></a>¿Cuáles son las opciones de ingesta de mensajes en tiempo real?

- [Azure Event Hubs](/azure/event-hubs/)
- [Azure IoT Hub](/azure/iot-hub/)
- [Kafka en HDInsight](/azure/hdinsight/kafka/apache-kafka-get-started)

## <a name="azure-event-hubs"></a>Azure Event Hubs

[Azure Event Hubs](/azure/event-hubs/) es una plataforma de streaming de datos y servicio de ingesta de eventos de gran escalabilidad capaz de recibir y procesar millones de eventos por segundo. Event Hubs puede procesar y almacenar eventos, datos o telemetría generados por dispositivos y software distribuido. Los datos enviados a un centro de eventos se pueden transformar y almacenar con cualquier proveedor de análisis en tiempo real o adaptadores de procesamiento por lotes y almacenamiento. Event Hubs proporciona funcionalidades de publicación-suscripción con baja latencia a gran escala, lo cual hace que resulte apropiado para escenarios de macrodatos.

## <a name="azure-iot-hub"></a>Azure IoT Hub

[Azure IoT Hub](/azure/iot-hub/) es un servicio totalmente administrado que permite la comunicación bidireccional confiable y segura entre millones de dispositivos IoT y un back-end basado en la nube.

Entre las características de IoT Hub se incluyen:

* Varias opciones de comunicación de dispositivo a nube y de nube a dispositivo. Dichas opciones incluyen la mensajería unidireccional, la transferencia de archivos y los métodos de solicitud y respuesta.
* Enrutamiento de mensajes a otros servicios de Azure.
* Almacenamiento consultable para metadatos del dispositivo e información de estado sincronizada.
* Comunicaciones seguras y control de acceso con claves de seguridad por dispositivo o certificados X.509.
* Supervisión exhaustiva para la conectividad de dispositivos y los eventos de administración de identidad de dispositivos.

En términos de ingesta de mensajes, IoT Hub es similar a Event Hubs. Sin embargo, IoT Hub se ha diseñado específicamente para administrar la conectividad de dispositivos de IoT, no solo la ingesta de mensajes. Para más información, consulte [Comparación entre Azure IoT Hub y Azure Event Hubs](/azure/iot-hub/iot-hub-compare-event-hubs). 

## <a name="kafka-on-hdinsight"></a>Kafka en HDInsight

[Apache Kafka](https://kafka.apache.org/) es una plataforma de streaming distribuido de código abierto que se puede utilizar para compilar canalizaciones de datos en tiempo real y aplicaciones de streaming. Kafka también proporciona funcionalidad de agente de mensajería similar a una cola de mensajes, donde puede publicar y suscribirse a las transmisiones de datos con nombre. Es escalable horizontalmente, con tolerancia a errores y extremadamente rápido. [Kafka en HDInsight](/azure/hdinsight/kafka/apache-kafka-get-started) proporciona una instancia de Kafka como un servicio administrado, altamente escalable y de alta disponibilidad en Azure. 

Algunos casos de uso habituales de Kafka son:

* **Mensajería**. Como admite el patrón de publicación de mensajes y suscripción, Kafka a menudo se utiliza como agente de mensajería.
* **Seguimiento de actividad**. Como Kafka proporciona un registro en orden, este se puede utilizar para realizar un seguimiento y volver a crear actividades como acciones de usuario en un sitio web.
* **Agregación**. Mediante el procesamiento de los flujos, puede agregar información de distintos flujos para combinar y centralizar la información en datos operativos.
* **Transformación**. Mediante el procesamiento de los flujos, puede combinar y enriquecer los datos de varios temas de entrada en uno o más temas de salida.

## <a name="key-selection-criteria"></a>Principales criterios de selección

Para restringir las opciones, empiece por responder a estas preguntas:

- ¿Necesita comunicación bidireccional entre los dispositivos de IoT y Azure? Si es así, elija IoT Hub.

- ¿Necesita administrar el acceso de dispositivos individuales y poder revocar el acceso a un dispositivo específico? Si es así, elija IoT Hub.

## <a name="capability-matrix"></a>Matriz de funcionalidades

En las tablas siguientes se resumen las diferencias clave en cuanto a funcionalidades. 

| | IoT Hub | Event Hubs | Kafka en HDInsight |
| --- | --- | --- | --- |
| Comunicaciones de nube a dispositivo | Sí | Sin  | Sin  |
| Carga de archivos iniciada por dispositivo | Sí | Sin  | Sin  |
| Información de estado de dispositivo | [Dispositivos gemelos](/azure/iot-hub/iot-hub-devguide-device-twins) | Sin  | Sin  |
| Compatibilidad con protocolos | MQTT, AMQP, HTTPS <sup>1</sup> | AMQP, HTTPS | [Kafka Protocol](https://cwiki.apache.org/confluence/display/KAFKA/A+Guide+To+The+Kafka+Protocol) |
| Seguridad | Identidad por dispositivo y control de acceso revocable. | Directivas de acceso compartido, revocación limitada a través de las directivas de publicador. | Compatible con la autenticación mediante SASL, autorización conectable, integración con servicios de autenticación externos. |

[1] Puede usar la [Puerta de enlace de protocolos de Azure IoT](/azure/iot-hub/iot-hub-protocol-gateway) como puerta de enlace personalizada para permitir la adaptación de protocolos para IoT Hub.

Para más información, consulte [Comparación entre Azure IoT Hub y Azure Event Hubs](/azure/iot-hub/iot-hub-compare-event-hubs).
