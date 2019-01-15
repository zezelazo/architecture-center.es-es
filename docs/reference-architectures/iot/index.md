---
title: Arquitectura de referencia de Azure IoT
description: Arquitectura recomendada para aplicaciones de IoT en Azure que usa componentes de PaaS (plataforma como servicio)
titleSuffix: Azure Reference Architectures
author: MikeWasson
ms.date: 01/09/2019
ms.openlocfilehash: c0aa1771abc99b1f17f1e553c9626e50a386f34c
ms.sourcegitcommit: d5ea427c25f9f7799cc859b99f328739ca2d8c1c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/15/2019
ms.locfileid: "54307695"
---
# <a name="azure-iot-reference-architecture"></a>Arquitectura de referencia de Azure IoT

Esta arquitectura de referencia muestra una arquitectura recomendada para aplicaciones de IoT en Azure que usa componentes de PaaS (plataforma como servicio).

![Diagrama de la arquitectura](./_images/iot.png)

Las aplicaciones de IoT se pueden describir como **objetos** (dispositivos) que envían datos que generan **conclusiones**. Estas conclusiones generan **acciones** que sirven para mejorar un negocio o proceso. Un ejemplo es un motor (el objeto) que envía datos de temperatura. Estos datos se utilizan para evaluar si el motor está funcionando según lo esperado (las conclusiones). Las conclusiones se emplean para clasificar por orden de prioridad y de manera proactiva la programación de mantenimiento del motor (las acciones).

Esta arquitectura de referencia usa componentes de PaaS (plataforma como servicio) de Azure. Otras opciones para crear soluciones de IoT en Azure incluyen:

- [Azure IoT Central](/azure/iot-central/). IoT Central es una solución SaaS (software como servicio) totalmente administrada. Resume las opciones técnicas y le permite centrarse exclusivamente en la solución. El inconveniente es que se trata de una solución menos personalizable que una solución basada en PaaS.
- Uso de componentes de OSS como la pila de SMACK (Spark, Mesos, Akka, Cassandra, Kafka) que se implementan en máquinas virtuales de Azure. Este enfoque ofrece un buen grado de control, pero es más complejo.

A nivel general, existen dos maneras de procesar datos de telemetría, las rutas de acceso activas y las inactivas. La diferencia tiene que ver con los requisitos de latencia y de acceso a datos.

- Las **rutas de acceso activas** analizan los datos según llegan, casi en tiempo real. En las rutas de acceso activas, los datos de telemetría se deben procesar con una latencia muy baja. Una ruta de acceso activa se implementa normalmente mediante un motor de procesamiento de flujos. La salida puede desencadenar una alerta o escribirse en un formato estructurado que se puede consultar mediante herramientas de análisis.
- Las **rutas de acceso inactivas** realizan un procesamiento por lotes en intervalos más prolongados (cada hora o diariamente). La ruta de acceso inactiva se emplea normalmente con grandes volúmenes de datos, pero los resultados no se necesitan de manera tan urgente como en el caso de las rutas de acceso activas. En las rutas de acceso inactivas, se capturan los datos de telemetría sin procesar y, posteriormente, se introducen en un proceso por lotes.

## <a name="architecture"></a>Arquitectura

Esta arquitectura consta de los siguientes componentes. Puede que algunas aplicaciones no requieran todos los componentes que se mencionan aquí.

**Dispositivos IoT**. Dispositivos se pueden registrar de forma segura con la nube y pueden conectarse a esta para enviar y recibir datos. Algunos dispositivos pueden ser **dispositivos perimetrales** que realizan algún tipo de procesamiento de datos en el propio dispositivo o en una puerta de enlace de campo. Se recomienda usar [Azure IoT Edge](/azure/iot-edge/) para el procesamiento perimetral.

**Puerta de enlace en la nube**. Una puerta de enlace en la nube proporciona un centro en la nube para que los dispositivos se conecten de forma segura a la nube y envíen datos. También proporciona administración de dispositivos y funcionalidades como comandos o el control de dispositivos. Para la puerta de enlace en la nube, se recomienda [IoT Hub](/azure/iot-hub/). IoT Hub es un servicio hospedado en la nube que ingiere eventos de dispositivos y que actúa como un agente de mensajes entre los dispositivos y los servicios back-end. IoT Hub proporciona conectividad segura, ingesta de eventos, comunicación bidireccional y administración de dispositivos.

**Aprovisionamiento de dispositivos**. Para registrar y conectar grandes conjuntos de dispositivos, se recomienda usar [IoT Hub Device Provisioning Service](/azure/iot-dps/) (DPS). DPS le permite asignar y registrar dispositivos en determinados puntos de conexión de Azure IoT Hub a escala.

**Procesamiento de flujos**. El procesamiento de flujos analiza grandes flujos de registros de datos y evalúa las reglas de esos flujos. Para el procesamiento de flujos de datos se recomienda usar [Azure Stream Analytics](/azure/stream-analytics/). Stream Analytics puede ejecutar un análisis complejo a escala mediante funciones basadas en ventanas de tiempo, agregaciones de flujos y combinaciones de orígenes de datos externos. Otra opción es Apache Spark en [Azure Databricks](/azure/azure-databricks/).

El aprendizaje automático permite que se ejecuten algoritmos de predicción en datos de telemetría históricos, lo cual permite escenarios como el de mantenimiento predictivo. Para el aprendizaje automático se recomienda usar el [servicio Azure Machine Learning](/azure/machine-learning/service/).

El **almacenamiento en rutas de acceso semiactivas** contiene los datos que deben estar disponibles inmediatamente desde el dispositivo para su uso en informes o su visualización. Para el almacenamiento en rutas de acceso semiactivas, se recomienda [Cosmos DB](/azure/cosmos-db/introduction). Cosmos DB es una base de datos de varios modelos distribuida de forma global.

El **almacenamiento en rutas de acceso inactivas** contiene datos que hay que mantener a largo plazo y se usa para el procesamiento por lotes. Para este tipo de almacenamiento, se recomienda [Azure Blob Storage](/azure/storage/blobs/storage-blobs-introduction). Los datos se pueden archivar en Blob Storage de forma indefinida a bajo costo y son fácilmente accesibles para el procesamiento por lotes.

La **transformación de datos** permite manipular o agregar el flujo de datos de telemetría. Algunos ejemplos incluyen la transformación de protocolos como, por ejemplo, la conversión de datos binarios a JSON o la combinación de puntos de datos. Si los datos deben transformarse antes de llegar a IoT Hub, se recomienda usar una [puerta de enlace de protocolo](/azure/iot-hub/iot-hub-protocol-gateway) (no se muestra). En caso contrario, se pueden transformar los datos después de que lleguen a IoT Hub. En ese caso, se recomienda usar [Azure Functions](/azure/azure-functions/). Functions dispone de integración incorporada con IoT Hub, Cosmos DB y Blob Storage.

La **integración de procesos empresariales** lleva a cabo acciones basadas en las conclusiones que se generan a partir de los datos del dispositivo. Esto puede incluir el almacenamiento de mensajes informativos, la generación de alarmas, el envío de mensajes de correo electrónico o de SMS, o la integración con CRM. Se recomienda usar [Azure Logic Apps](/azure/logic-apps/logic-apps-overview) para la integración de procesos empresariales.

La **administración de usuarios** restringe qué usuarios o grupos pueden realizar acciones en los dispositivos como, por ejemplo, actualizar el firmware. También permite definir funcionalidades para los usuarios en las aplicaciones. Se recomienda usar [Azure Active Directory](/azure/active-directory/) para autenticar y autorizar a los usuarios.

## <a name="scalability-considerations"></a>Consideraciones sobre escalabilidad

Una aplicación de IoT se debe crear como servicios discretos que se pueden escalar de forma independiente. Tenga en cuenta los siguientes puntos de escalabilidad:

**IoTHub**. Para IoT Hub, tenga en cuenta los siguientes factores de escala:

- La [cuota diaria](/azure/iot-hub/iot-hub-devguide-quotas-throttling) máxima de mensajes en IoT Hub.
- La cuota de los dispositivos conectados en una instancia de IoT Hub.
- Rendimiento de ingesta: la rapidez con la que IoT Hub puede ingerir mensajes.
- Rendimiento de procesamiento: la rapidez con la que se procesan los mensajes entrantes.

Cada instancia de IoT Hub se aprovisiona con un determinado número de unidades en un nivel determinado. El nivel y el número de unidades determinan la cuota diaria máxima de mensajes que los dispositivos pueden enviar al centro. Para más información, consulte las cuotas y limitaciones de IoT. Se puede escalar verticalmente un centro sin interrumpir las operaciones existentes.

**Stream Analytics**. Los trabajos de Stream Analytics se escalan mejor si son paralelos en todos los puntos de la canalización de Stream Analytics, desde el de entrada al de consulta y al de salida. Un trabajo totalmente paralelo permite a Stream Analytics dividir el trabajo entre varios nodos de proceso. En caso contrario, Stream Analytics tiene que combinar los datos del flujo en un solo lugar. Para más información, consulte [Aprovechamiento de la paralelización de consultas en Azure Stream Analytics](/azure/stream-analytics/stream-analytics-parallelization).

IoT Hub realiza automáticamente particiones de los mensajes de dispositivo según el id. de dispositivo. Todos los mensajes de un dispositivo concreto llegarán siempre a la misma partición, aunque una única partición puede tener mensajes de varios dispositivos. Por tanto, la unidad de paralelización es el identificador de partición.

**Funciones**. Cuando se lee desde el punto de conexión de Event Hubs, hay un máximo de instancias de funciones por cada partición del centro de eventos. La velocidad máxima de procesamiento viene determinada por la velocidad con la que una instancia de función puede procesar los eventos procedentes de una única partición. La función debe procesar los mensajes en lotes.

**Cosmos DB**. Para escalar horizontalmente una colección de Cosmos DB, cree la colección con una clave de partición e inclúyala en cada documento que escriba. Para más información, consulte [Procedimientos recomendados al elegir una clave de partición](/azure/cosmos-db/partition-data#best-practices-when-choosing-a-partition-key).

- Si almacena y actualiza un documento único por dispositivo, el identificador de dispositivo es una buena clave de partición. Las escrituras se distribuyen uniformemente entre las claves. El tamaño de cada partición está estrictamente limitado porque hay un solo documento para cada valor de clave.
- Si almacena un documento independiente para cada mensaje de dispositivo, el hecho de usar el id. de dispositivo como clave de partición haría que se superara rápidamente el límite de 10 GB por partición. En este caso, el identificador de mensaje sería una mejor clave de partición. Normalmente, seguiría incluyendo el id. de dispositivo en el documento con fines de indexación y consulta.

## <a name="security-considerations"></a>Consideraciones sobre la seguridad

### <a name="trustworthy-and-secure-communication"></a>Comunicación segura y confiable

Toda la información que se recibe desde un dispositivo o que se envía a este debe ser de confianza. A menos que un dispositivo pueda admitir las siguientes funcionalidades criptográficas, se debería limitar a la comunicación con las redes locales y toda comunicación con Internet debería pasar a través de una puerta de enlace de campo:

- Cifrado de datos con un algoritmo de cifrado de claves simétricas posiblemente seguro, analizado públicamente y ampliamente implementado.
- Firma digital con un algoritmo de firma de claves simétricas posiblemente seguro, analizado públicamente y ampliamente implementado.
- Compatibilidad con TLS 1.2 para TCP o con cualquier otra ruta de acceso de comunicación basada en flujos, o con DTLS 1.2 para rutas de acceso de comunicación basadas en datagramas. La compatibilidad con la administración de certificados X.509 es opcional y se puede sustituir por el modo de clave previamente compartida para TLS, que es más eficiente en el nivel de procesos y en el nivel inalámbrico, y que se puede implementar con compatibilidad con los algoritmos AES y SHA-2.
- Almacén de claves actualizable y claves por dispositivo. Cada dispositivo debe contar con material o tokens de clave exclusivos que lo identifiquen en el sistema. Los dispositivos deben almacenar la clave de forma segura en el dispositivo (por ejemplo, con un almacén de claves seguro). El dispositivo debe ser capaz de actualizar las claves o tokens periódicamente, o como reacción en situaciones de emergencia como, por ejemplo, una infracción del sistema.
- El firmware y el software de la aplicación del dispositivo deben permitir las actualizaciones para habilitar la reparación de las vulnerabilidades de seguridad descubiertas.

No obstante, muchos dispositivos están demasiado restringidos para admitir estos requisitos. En ese caso, debe usarse una puerta de enlace de campo. Los dispositivos se conectan de forma segura a la puerta de enlace de campo mediante una red de área local y la puerta de enlace permite una comunicación segura con la nube.

### <a name="physical-tamper-proofing"></a>Corrección de alteraciones físicas

Se recomienda encarecidamente que el diseño del dispositivo incorpore características que lo protejan contra cualquier intento de manipulación física, para ayudar a garantizar la integridad de la seguridad y la confiabilidad de todo el sistema.

Por ejemplo: 

- Elija microcontroladores o microprocesadores, o hardware auxiliar que proporcione almacenamiento seguro y uso de material de clave criptográfica como, por ejemplo, la integración de un módulo de plataforma segura (TPM).
- Cargador de arranque y carga de software seguras, ancladas en el TPM.
- Use sensores para detectar intentos de intrusión e intentos de manipular el entorno del dispositivo con alertas y una posible "autodestrucción digital" del dispositivo.

Para otras consideraciones de seguridad, consulte [Arquitectura de seguridad de Internet de las cosas (IoT)](/azure/iot-fundamentals/iot-security-architecture).

### <a name="monitoring-and-logging"></a>Supervisión y registro

Los sistemas de registro y supervisión se usan para determinar si la solución funciona y para ayudar a solucionar problemas. Los sistemas de supervisión y registro ayudan a responder a las siguientes preguntas operativas:

- ¿Se encuentran los dispositivos o sistemas en una condición de error?
- ¿Están configurados correctamente?
- ¿Generan los datos adecuados?
- ¿Cumplen los sistemas las expectativas de la empresa y de los clientes finales?

Las herramientas de registro y supervisión se componen normalmente de los siguientes cuatro componentes:

- Herramientas de rendimiento del sistema y de visualización de la escala de tiempo para supervisar el sistema y poder solucionar problemas básicos.
- Ingesta de datos almacenados en búfer para almacenar en búfer los datos de registro.
- Almacenamiento persistente para almacenar datos de registro.
- Funcionalidades de búsqueda y consulta para visualizar los datos de registro para su uso en un proceso de solución de problemas en profundidad.

Los sistemas de supervisión proporcionan información sobre el estado, la seguridad, la estabilidad y el rendimiento de una solución de IoT. Estos sistemas también pueden proporcionar una vista más detallada, registrar cambios de configuración de los componentes y proporcionar datos de registro extraídos que pueden revelar posibles vulnerabilidades de seguridad, mejorar el proceso de administración de incidentes y ayudar al propietario del sistema a solucionar problemas. Las soluciones integrales de supervisión incluyen la posibilidad de consultar la información de subsistemas concretos o de agregarla entre varios subsistemas.

El desarrollo del sistema de supervisión debe empezar por definir un funcionamiento correcto, el cumplimiento normativo y los requisitos de auditoría. Las métricas recopiladas pueden incluir:

- Dispositivos físicos, dispositivos perimetrales y componentes de infraestructura que notifican cambios de configuración.
- Aplicaciones que notifican cambios de configuración, registros de auditoría de seguridad, velocidad de solicitudes, tiempos de respuesta, tasas de error y estadísticas sobre recolección de elementos no utilizados para lenguajes administrados.
- Bases de datos, almacenes persistentes y memorias caché que informan sobre el rendimiento de las operaciones de consulta y escritura, los cambios de esquemas, los registros de auditoría de seguridad, los bloqueos e interbloqueos, el rendimiento de la indexación y el uso de la CPU, la memoria y el disco.
- Servicios administrados (IaaS, PaaS, SaaS, and FaaS) que informan sobre métricas de mantenimiento y cambios de configuración que afectan al estado y el rendimiento del sistema dependiente.

La visualización de las métricas de supervisión sirve para alertar a los operadores sobre inestabilidades del sistema y facilitan la respuesta ante incidentes.

### <a name="tracing-telemetry"></a>Seguimiento de los datos de telemetría

El seguimiento de los datos de telemetría permite a un operador seguir el recorrido de un fragmento de datos de telemetría desde la creación a través del sistema. Este seguimiento es importante para los procesos de depuración y solución de problemas. En el caso de las soluciones de IoT que usan Azure IoT Hub y los [SDK de dispositivos de IoT Hub](/azure/iot-hub/iot-hub-devguide-sdks), los datagramas de seguimiento se pueden originar como mensajes de la nube al dispositivo e incluir en el flujo de telemetría.

### <a name="logging"></a>Registro

Los sistemas de registro son una parte integral a la hora de comprender qué acciones o actividades ha realizado una solución, qué errores se han producido y puede proporcionar ayuda para solucionar esos errores. Los registros se pueden analizar para ayudar a entender y solucionar condiciones de error, mejorar las características de rendimiento y garantizar el cumplimiento con las normas y directrices vigentes.

Aunque el registro de texto sin formato tiene un menor impacto en los costos de desarrollo iniciales, es más difícil de analizar y leer por parte de una máquina. Se recomienda usar el registro estructurado, ya que la información recopilada resulta analizable para una máquina y legible para los usuarios. El registro estructurado agrega contexto situacional y metadatos a la información del registro. En el registro estructurado, las propiedades son el elemento principal, tienen un formato de pares clave-valor y se emplean con un esquema fijo para mejorar las funcionalidades de búsqueda y consulta.

## <a name="next-steps"></a>Pasos siguientes

- Para un análisis más detallado de la arquitectura y las opciones de implementación más recomendables, consulte el archivo PDF [Arquitectura de referencia de IoT de Microsoft Azure](http://aka.ms/iotrefarchitecture).

- Para obtener documentación detallada sobre los distintos servicios de IoT de Azure, consulte [Aspectos básicos de IoT de Azure](/azure/iot-fundamentals/).
