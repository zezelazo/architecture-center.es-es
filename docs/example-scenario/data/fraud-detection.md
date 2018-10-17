---
title: Detección de fraudes en tiempo real en Azure
description: Detecte actividades fraudulentas en tiempo real con Azure Event Hubs y Stream Analytics.
author: alexbuckgit
ms.date: 07/05/2018
ms.openlocfilehash: 4de988731aa1c5b0e4c0ba06fa5aed59e2bb7d81
ms.sourcegitcommit: b2a4eb132857afa70201e28d662f18458865a48e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/05/2018
ms.locfileid: "48818673"
---
# <a name="real-time-fraud-detection-on-azure"></a>Detección de fraudes en tiempo real en Azure

Este escenario de ejemplo es pertinente para las organizaciones que necesitan analizar los datos en tiempo real para detectar transacciones fraudulentas u otras actividades anómalas.

Las posibles aplicaciones son la identificación de actividades con tarjetas de crédito o llamadas de teléfono móvil fraudulentas. Los sistemas de análisis en línea tradicionales podrían tardar horas en transformar y analizar los datos para identificar la actividad anómala.

Gracias a los servicios de Azure totalmente administrados, como Event Hubs y Stream Analytics, las empresas pueden eliminar la necesidad de administrar servidores individuales, reducir los costos y aprovechar la experiencia que Microsoft tiene en ingesta de datos de escala de nube y análisis en tiempo real. En concreto, en este escenario se aborda la detección de actividades fraudulentas. Si tiene otras necesidades de análisis de datos, consulte la lista de [servicios de análisis de Azure][product-category] disponibles.

Este ejemplo representa una parte de una estrategia y una arquitectura de procesamiento de datos más amplia. Otras opciones para este aspecto de una arquitectura global se tratan más adelante en este artículo.

## <a name="relevant-use-cases"></a>Casos de uso pertinentes

Tenga en cuenta este escenario para los casos de uso siguientes:

* Detección de llamadas de teléfono móvil fraudulentas en escenarios de telecomunicaciones.
* Identificación de transacciones fraudulentas de tarjetas de crédito para las instituciones de banca.
* Identificación de compras fraudulentas en escenarios de comercio físico o electrónico.

## <a name="architecture"></a>Arquitectura

![Introducción a la arquitectura de los componentes de Azure de un escenario de detección de fraudes en tiempo real][architecture]

Este escenario trata los componentes de back-end de una canalización de análisis en tiempo real. Los datos fluyen por el escenario de la siguiente manera:

1. Los metadatos de las llamadas de teléfono móvil se envían desde el sistema de origen a una instancia de Azure Event Hubs. 
2. Se inicia un trabajo de Stream Analytics, que recibe los datos mediante el origen del centro de eventos.
3. El trabajo de Stream Analytics ejecuta una consulta predefinida para transformar el flujo de entrada y analizarlo según un algoritmo de transacciones fraudulentas. Esta consulta utiliza una ventana de saltos de tamaño constante para segmentar el flujo en distintas unidades temporales.
4. El trabajo de Stream Analytics escribe el flujo transformado que representa las llamadas fraudulentas detectadas en un receptor de salida en Azure Blob Storage.

### <a name="components"></a>Componentes

* [Azure Event Hubs][docs-event-hubs] es una plataforma de streaming en tiempo real y un servicio de ingesta de eventos de gran escalabilidad capaz de recibir y procesar millones de eventos por segundo. Event Hubs puede procesar y almacenar eventos, datos o telemetría generados por dispositivos y software distribuido. En este escenario, Event Hubs recibe todos los metadatos de las llamadas de teléfono que se van a analizar en busca de actividades fraudulentas.
* [Azure Stream Analytics][docs-stream-analytics] es un motor de procesamiento de eventos que permite analizar grandes volúmenes de streaming de datos procedentes de dispositivos y otros orígenes de datos. También permite extraer información de los flujos de datos e identificar patrones y relaciones. Estos patrones pueden desencadenar otras acciones en niveles inferiores. En este escenario, Stream Analytics transforma la secuencia de entrada de Event Hubs para identificar las llamadas fraudulentas.
* [Blob Storage](/azure/storage/blobs/storage-blobs-introduction) se utiliza en este escenario para almacenar los resultados del trabajo de Stream Analytics.

## <a name="considerations"></a>Consideraciones

### <a name="alternatives"></a>Alternativas

Existen muchas opciones tecnológicas de ingesta de mensajes en tiempo real, almacenamiento de datos, flujo de procesamiento, almacenamiento de datos análisis e informes. Para obtener información general sobre estas opciones, sus funcionalidades y los principales criterios de selección, consulte [Arquitecturas de macrodatos: Procesamiento en tiempo real](/azure/architecture/data-guide/technology-choices/real-time-ingestion) en la Guía de arquitectura de datos de Azure.

Además, se pueden producir algoritmos más complejos para la detección de fraudes con diversos servicios de aprendizaje automático de Azure. Para obtener información general sobre estas opciones, consulte [Opciones de tecnología: Machine Learning](/azure/architecture/data-guide/technology-choices/data-science-and-machine-learning) en la [Guía de arquitectura de datos de Azure](../../data-guide/index.md).

### <a name="availability"></a>Disponibilidad

Azure Monitor proporciona interfaces de usuario unificadas para la supervisión de distintos servicios de Azure. Para más información, consulte [Supervisión en Microsoft Azure](/azure/monitoring-and-diagnostics/monitoring-overview). Event Hubs y Stream Analytics se integran ambos con Azure Monitor. 

Para ver otras consideraciones sobre escalabilidad, consulte la [lista de comprobación de disponibilidad][availability] en el centro de arquitectura de Azure.

### <a name="scalability"></a>Escalabilidad

Los componentes de este escenario están diseñados para una ingesta a hiperescala y análisis en tiempo real masivo en paralelo. Azure Event Hubs es muy escalable, capaz de recibir y procesar millones de eventos por segundo con una baja latencia. Event Hubs puede [escalar verticalmente](/azure/event-hubs/event-hubs-auto-inflate) el número de unidades de rendimiento para responder a las necesidades de uso. Azure Stream Analytics puede analizar grandes volúmenes de datos de varios orígenes de streaming. Para escalar Stream Analytics verticalmente, puede aumentar el número de [unidades de streaming](/azure/stream-analytics/stream-analytics-streaming-unit-consumption) asignado para ejecutar el trabajo de streaming.

Para obtener instrucciones generales sobre cómo diseñar escenarios escalables, consulte la [lista de comprobación de escalabilidad][scalability] en el centro de arquitectura de Azure.

### <a name="security"></a>Seguridad

Azure Event Hubs protege los datos mediante un [modelo de autenticación y seguridad][docs-event-hubs-security-model] basado en una combinación de tokens de firma de acceso compartido (SAS) y publicadores de eventos. Un publicador de eventos define un punto de conexión virtual para un centro de eventos. El publicador solo puede usarse para enviar mensajes a un centro de eventos. No es posible recibir mensajes desde un publicador.

Para obtener instrucciones generales sobre el diseño de soluciones seguras, consulte la [documentación de seguridad de Azure][security].

### <a name="resiliency"></a>Resistencia

Para obtener instrucciones generales sobre el diseño de soluciones resistentes, consulte [Diseño de aplicaciones resistentes de Azure][resiliency].

## <a name="deploy-the-scenario"></a>Implementación del escenario

Para implementar este escenario, puede seguir este [tutorial detallado][tutorial] que muestra cómo implementar manualmente cada componente. Este tutorial también proporciona una aplicación de cliente de .NET para generar los metadatos de ejemplo de llamadas de teléfono y enviar los datos a un centro de eventos.

## <a name="pricing"></a>Precios

Para explorar el costo de ejecutar este escenario, todos los servicios están preconfigurados en la calculadora de costos. Para ver cómo cambiarían los precios en su caso concreto, cambie las variables pertinentes para que coincidan con el volumen de datos esperado.

Hemos proporcionado tres ejemplos de perfiles de costo según la cantidad de tráfico que se espera obtener:

* [Pequeño][small-pricing]: se procesa un millón de eventos en una unidad de streaming estándar al mes.
* [Mediano][medium-pricing]: se procesan 100 millones de eventos en cinco unidades de streaming estándar al mes.
* [Grande][large-pricing]: se procesan 999 millones de eventos en 20 unidades de streaming estándar al mes.

## <a name="related-resources"></a>Recursos relacionados

Los escenarios de detección de fraudes más complejos pueden beneficiarse de un modelo de aprendizaje automático. Para ver escenarios creados con Machine Learning Server, consulte [Detección de fraudes con Machine Learning Server][r-server-fraud-detection]. Para ver otras plantillas de solución con Machine Learning Server, consulte [Escenarios de ciencia de datos y plantillas de solución][docs-r-server-sample-solutions]. Para ver una solución de ejemplo con Azure Data Lake Analytics, consulte [Uso de Azure Data Lake y R para la detección de fraudes][technet-fraud-detection].

<!-- links -->
[product-category]: https://azure.microsoft.com/product-categories/analytics/
[tutorial]: /azure/stream-analytics/stream-analytics-real-time-fraud-detection
[small-pricing]: https://azure.com/e/74149ec312c049ccba79bfb3cfa67606
[medium-pricing]: https://azure.com/e/4fc94f7376de484d8ae67a6958cae60a
[large-pricing]: https://azure.com/e/7da8804396f9428a984578700003ba42
[architecture]: ./media/architecture-fraud-detection.png
[docs-event-hubs]: /azure/event-hubs/event-hubs-what-is-event-hubs
[docs-event-hubs-security-model]: /azure/event-hubs/event-hubs-authentication-and-security-model-overview
[docs-stream-analytics]: /azure/stream-analytics/stream-analytics-introduction
[docs-r-server-sample-solutions]: /machine-learning-server/r/sample-solutions
[r-server-fraud-detection]: https://microsoft.github.io/r-server-fraud-detection/
[technet-fraud-detection]: https://blogs.technet.microsoft.com/machinelearning/2017/06/28/using-azure-data-lake-and-r-for-fraud-detection/
[availability]: /azure/architecture/checklist/availability
[scalability]: /azure/architecture/checklist/scalability
[resiliency]: ../../resiliency/index.md
[security]: /azure/security/

