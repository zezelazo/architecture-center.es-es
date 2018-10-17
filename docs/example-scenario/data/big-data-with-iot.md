---
title: IoT y el análisis de datos en el sector de la construcción
description: Use dispositivos de IoT y análisis de datos para proporcionar una administración y un funcionamiento integral de proyectos de construcción.
author: alexbuckgit
ms.date: 08/29/2018
ms.openlocfilehash: 7ab0de50b0eba1ab420e450f3408fe5dc45f04ac
ms.sourcegitcommit: b2a4eb132857afa70201e28d662f18458865a48e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/05/2018
ms.locfileid: "48818503"
---
# <a name="iot-and-data-analytics-in-the-construction-industry"></a>IoT y el análisis de datos en el sector de la construcción

Este escenario de ejemplo es pertinente para las organizaciones que crean soluciones que integran datos desde muchos dispositivos de IoT en una arquitectura de análisis integral de datos para mejorar y automatizar la toma de decisiones. Algunas aplicaciones posibles incluyen las soluciones de construcción, minería de datos, fabricación u otras del sector que implican grandes volúmenes de datos de muchas entradas de datos basadas en IoT.

En este escenario, un fabricante de equipos de construcción crea vehículos, medidores y drones que usan tecnologías de IoT y GPS para emitir los datos de telemetría. La empresa quiere modernizar su arquitectura de datos para supervisar mejor las condiciones de funcionamiento y de mantenimiento de los equipos. Reemplazar la solución antigua de la empresa con la infraestructura local costaría mucho tiempo y esfuerzo, y no permitiría escalar lo suficiente para controlar el volumen de datos previsto.

La empresa desea crear una solución de "creación inteligente" basada en la nube. Debe recopilar un conjunto completo de datos de un sitio de construcción y automatizar la operación y el mantenimiento de los diversos elementos del sitio. Los objetivos de la empresa incluyen:

* Integrar y analizar la construcción de todos los equipos de los sitios para minimizar el tiempo de inactividad y reducir el robo de datos.
* Controlar los equipos de construcción de forma remota y automática a fin de mitigar los efectos de la escasez de personal, dado que últimamente se requieren menos trabajadores y se permite que trabajadores con menos cualificación lleven a cabo su labor correctamente.
* Reducir los requisitos de mano de obra y los costos de explotación para la infraestructura de apoyo, al tiempo que aumenta la productividad y la seguridad.
* Escalar fácilmente la infraestructura para admitir incrementos en los datos de telemetría.
* Cumplir con todos los requisitos legales pertinentes mediante el aprovisionamiento de recursos en el país sin poner en peligro la disponibilidad del sistema.
* Usar software de código abierto a fin de maximizar la inversión para mejorar las aptitudes de los trabajadores.

Usar los servicios administrados de Azure, como IoT Hub y HDInsight, permitirá al cliente compilar e implementar con rapidez una solución completa con un menor costo de explotación. Si necesita un análisis de datos adicional, debe revisar la lista de los [servicios de análisis de datos totalmente administrados en Azure][product-category] disponibles.

## <a name="relevant-use-cases"></a>Casos de uso pertinentes

Tenga en cuenta esta solución para los casos de uso siguientes:

* Escenarios de construcción, minería de datos o fabricación de equipos
* Colección a gran escala de datos del dispositivo de almacenamiento y análisis
* Ingesta y análisis de grandes conjuntos de datos

## <a name="architecture"></a>Arquitectura

![Arquitectura para el análisis de datos e IoT en el sector de la construcción][architecture]

Los datos fluyen por la solución de la siguiente manera:

1. El equipo de construcción recopila los datos del sensor y envía los resultados de la construcción a intervalos regulares para cargar los servicios web con equilibrio de carga hospedados en un clúster de máquinas virtuales de Azure.
2. Los servicios web personalizados ingieren los datos de resultados de la construcción y los almacenan en un clúster de Apache Cassandra que también se ejecuta en máquinas virtuales de Azure.
3. Los sensores de IoT recopilan otro conjunto de datos en varios equipos de construcción y los envían a IoT Hub.
4. Los datos sin procesar recopilados se envían directamente desde IoT Hub a Azure Blob Storage y están disponibles inmediatamente para verlos y analizarlos.
5. Los datos recopilados a través de IoT Hub se obtienen casi en tiempo real en un trabajo de Azure Stream Analytics que los almacena en una base de datos SQL de Azure.
6. La aplicación web en la nube de construcción inteligente está disponible para que los analistas y usuarios finales vean y analicen imágenes y datos del sensor. 
7. Los trabajos por lotes se inician a petición de los usuarios de la aplicación web. El trabajo por lotes se ejecuta en Apache Spark en HDInsight y analiza los nuevos datos almacenados en el clúster de Cassandra. 

### <a name="components"></a>Componentes

* [IoT Hub](/azure/iot-hub/about-iot-hub) actúa como un centro de mensajes para proteger la comunicación bidireccional con la identidad por dispositivo entre la plataforma de nube y el material de construcción y otros elementos del sitio. IoT Hub puede recopilar rápidamente los datos de cada dispositivo para la ingesta en la canalización de análisis de datos. 
* [Azure Stream Analytics](/azure/stream-analytics/stream-analytics-introduction) es un motor de procesamiento de eventos que permite analizar grandes volúmenes de streaming de datos procedentes de dispositivos y otros orígenes de datos. También permite extraer información de los flujos de datos e identificar patrones y relaciones. En este escenario, Stream Analytics ingiere y analiza los datos desde los dispositivos IoT y almacena los resultados en la base de datos de SQL Azure. 
* [SQL Azure Database](/azure/sql-database/sql-database-technical-overview) contiene los resultados de los datos analizados desde los dispositivos y medidores de IoT, que los analistas y usuarios pueden ver a través de una aplicación web basada en Azure. 
* [Almacenamiento de blobs](/azure/storage/blobs/storage-blobs-introduction) almacena los datos de imágenes recopilados de los dispositivos de los centros de IoT. Los datos de imágenes pueden verse a través de la aplicación web.
* [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview) controla la distribución del tráfico de usuario en los puntos de conexión de servicio de las diferentes regiones de Azure.
* [Load Balancer](/azure/load-balancer/load-balancer-overview) distribuye los envíos de datos desde dispositivos de equipos de construcción a través de los servicios web basados en la máquina virtual para proporcionar alta disponibilidad.
* [Azure Virtual Machines](/azure/virtual-machines) hospeda los servicios web que reciben e ingieren los datos de los resultados de la construcción en la base de datos de Apache Cassandra.
* [Apache Cassandra](https://cassandra.apache.org) es una base de datos NoSQL distribuida que se usa para almacenar los datos de construcción para ser procesados posteriormente en Apache Spark.
* [Web Apps](/azure/app-service/app-service-web-overview) hospeda la aplicación web para el usuario final, que se puede usar para consultar y ver los datos de origen y las imágenes. Los usuarios también pueden iniciar trabajos por lotes en Apache Spark a través de la aplicación.
* [Apache Spark en HDInsight](/azure/hdinsight/spark/apache-spark-overview) admite el procesamiento en memoria para aumentar el rendimiento de las aplicaciones de análisis de macrodatos. En este escenario, Spark se usa para ejecutar algoritmos complejos en los datos almacenados en Apache Cassandra.


### <a name="alternatives"></a>Alternativas

* [Cosmos DB](/azure/cosmos-db/introduction) es una tecnología de base de datos NoSQL alternativa. Cosmos DB proporciona [compatibilidad con arquitectura multimaestro a escala global](/azure/cosmos-db/multi-region-writers) con [varios niveles de coherencia bien definidos](/azure/cosmos-db/consistency-levels) para cumplir los requisitos de distintos clientes. También admite [Cassandra API](/azure/cosmos-db/cassandra-introduction). 
* [Azure Databricks](/azure/azure-databricks/what-is-azure-databricks) es una plataforma de análisis basada en Apache Spark y optimizada para Azure. Se integra con Azure para proporcionar la instalación con un solo clic, flujos de trabajo optimizados y un área de trabajo colaborativa interactiva.
* [Data Lake Storage](/azure/storage/data-lake-storage) es una alternativa a Blob Storage. En este escenario, Data Lake Storage no estaba disponible en la región de destino.
* [Web Apps](/azure/app-service) también se puede usar a fin de hospedar los servicios web para ingerir los datos de los resultados de la construcción.
* Existen muchas opciones tecnológicas de ingesta de mensajes, almacenamiento de datos, procesamiento de secuencias, almacenamiento de datos analíticos en tiempo real, y de análisis e informes. Para obtener información general sobre estas opciones, sus funcionalidades y los principales criterios de selección, consulte [Arquitecturas de macrodatos: Procesamiento en tiempo real](/azure/architecture/data-guide/technology-choices/real-time-ingestion) en la [Guía de arquitectura de datos de Azure](/azure/architecture/data-guide).

## <a name="considerations"></a>Consideraciones

La amplia disponibilidad de regiones de Azure es un factor importante para este escenario. Tener más de una región en un único país puede permitir la recuperación ante desastres y facilitar también el cumplimiento de obligaciones contractuales y requisitos legales. Asimismo, la comunicación a alta velocidad de Azure entre regiones es un factor importante en este escenario.

El soporte técnico de Azure para las tecnologías de código abierto permite al cliente sacar provecho de las habilidades de su personal actual. El cliente también puede acelerar la adopción de nuevas tecnologías con menores costos y cargas de trabajo operativas en comparación con una solución local. 

## <a name="pricing"></a>Precios

Las siguientes consideraciones supondrán una parte considerable de los costos de esta solución.

* Los costos de la máquina virtual de Azure aumentarán linealmente a medida que se aprovisionen instancias adicionales. Las máquinas virtuales que se desasignen solo supondrán costos de almacenamiento y no de proceso. Estas máquinas desasignadas se pueden reasignar cuando la demanda sea alta.
* Los costos de [IoT Hub](https://azure.microsoft.com/pricing/details/iot-hub) se deben al número de unidades de IoT aprovisionadas, así como al nivel de servicio elegido, que determina el número de mensajes permitidos al día por unidad. 
* Los precios de [Azure Stream Analytics](https://azure.microsoft.com/pricing/details/stream-analytics) se basan en el número de unidades de streaming necesarias para procesar los datos en el servicio.

## <a name="related-resources"></a>Recursos relacionados

Para ver una implementación de una arquitectura similar, lea la [historia del cliente Komatsu][customer-story].

Puede encontrar instrucciones para las arquitecturas de macrodatos en la [Guía de arquitectura de datos de Azure](/azure/architecture/data-guide).

<!-- links -->
[product-category]: https://azure.microsoft.com/product-categories/analytics/
[customer-site]: https://home.komatsu/en/
[customer-story]: https://customers.microsoft.com/story/komatsu-manufacturing-azure-iot-hub-japan
[architecture]: ./media/architecture-big-data-with-iot.png
