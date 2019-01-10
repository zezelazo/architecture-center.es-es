---
title: Puntuación por lotes de modelos de Python en Azure
description: Cree una solución escalable para los modelos de puntuación por lotes en una programación en paralelo mediante Azure Batch AI.
author: njray
ms.date: 12/13/2018
ms.custom: azcat-ai
ms.openlocfilehash: 4c43a3dadab11cb8dcf163cf63618795299283ad
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/08/2019
ms.locfileid: "54111058"
---
# <a name="batch-scoring-of-python-models-on-azure"></a>Puntuación por lotes de modelos de Python en Azure

Esta arquitectura de referencia muestra cómo crear una solución escalable para la puntuación por lotes de muchos modelos de una programación en paralelo mediante Azure Batch AI. La solución se puede usar como plantilla y se puede generalizar para diferentes problemas.

En  [GitHub][github] hay disponible una implementación de referencia de esta arquitectura.

![Puntuación por lotes de modelos de Python en Azure](./_images/batch-scoring-python.png)

**Escenario**: Esta solución supervisa el funcionamiento de un gran número de dispositivos en una configuración de IoT en la que cada dispositivo envía lecturas de sensor continuamente. Se supone que cada dispositivo tiene modelos de detección de anomalías entrenados previamente que se deben usar para predecir si una serie de medidas, que se agregan a través de un intervalo predefinido, corresponde a una anomalía, o no. En escenarios del mundo real, podría tratarse de una secuencia de lecturas de los sensores que se deben filtrar y agregar antes de utilizarse en el entrenamiento o la puntuación en tiempo real. Por motivos de simplicidad, la solución utiliza el mismo archivo de datos al ejecutar trabajos de puntuación.

## <a name="architecture"></a>Arquitectura

Esta arquitectura consta de los siguientes componentes:

[Azure Event Hubs][event-hubs]. Este servicio de ingesta de mensajes puede ingerir millones de mensajes de eventos por segundo. En esta arquitectura, los sensores envían un flujo de datos al centro de eventos.

[Azure Stream Analytics][stream-analytics]. Un motor de procesamiento de eventos. Un trabajo de Stream Analytics lee los flujos de datos del centro de eventos y realiza su procesamiento.

[Azure Batch AI][batch-ai]. Este motor de computación distribuida se usa para entrenar y probar los modelos de aprendizaje automático e inteligencia artificial a escala en Azure. Batch AI crea máquinas virtuales a petición con una opción de escalado automático, donde cada nodo del clúster de Batch AI ejecuta un trabajo de puntuación para un sensor concreto. El [script][python-script] de puntuación en Python se ejecuta en contenedores de Docker que se crean en cada nodo del clúster, donde lee los datos de los sensores pertinentes, genera predicciones y las almacena en el almacenamiento de blobs.

[Azure Blob Storage][storage]. Los contenedores de blobs se usan para almacenar los modelos previamente entrenados, los datos y las predicciones de salida. Los modelos se cargan en el almacenamiento de blobs en el cuaderno [create\_resources.ipynb][create-resources]. Estos modelos de [SVM de una sola clase][one-class-svm] están entrenados en datos que representan los valores de los diferentes sensores de distintos dispositivos. En esta solución se da por supuesto que los valores de los datos se agregan en un intervalo fijo.

[Azure Logic Apps][logic-apps]. Esta solución crea una aplicación lógica que se ejecuta trabajos de Batch AI cada hora. Logic Apps proporciona una forma fácil de crear el flujo de trabajo del runtime y la programación de la solución. Los trabajos de Batch AI se envían mediante un [script][script] de Python que también se ejecuta en un contenedor de Docker.

[Azure Container Registry][acr]. Las imágenes de Docker se usan tanto en Batch AI como en Logic Apps, se crean en el cuaderno [create\_resources.ipynb][create-resources] y luego se insertan en Container Registry. Esto proporciona una forma cómoda de hospedar imágenes y crear instancias de contenedores a través de otros servicios de Azure (Logic Apps y Batch AI en esta solución).

## <a name="performance-considerations"></a>Consideraciones sobre rendimiento

En los modelos de Python estándar, se suele aceptar que las CPU son suficientes para controlar la carga de trabajo. Esta arquitectura utiliza varias CPU. Sin embargo, para [cargas de trabajo de aprendizaje profundo][deep], el rendimiento de las GPU normalmente es muy superior al de las CPU (en la medida en que un clúster del tamaño de las CPU suele ser necesario para obtener un rendimiento comparable).

### <a name="parallelizing-across-vms-vs-cores"></a>Poner en paralelo las máquinas virtuales con los núcleos

Al ejecutar los procesos de puntuación de muchos modelos de puntuación en modo por lotes, es preciso que los trabajos se ejecuten en paralelo en las máquinas virtuales. Son posibles dos enfoques:

* Crear un clúster mayor mediante máquinas virtuales de bajo costo.

* Crear un clúster menor mediante máquinas virtuales de alto rendimiento con más núcleos disponibles en cada una de ellas.

En general, la puntuación de los modelos de Python estándar no es tan exigente como la de los de aprendizaje profundo y un clúster pequeño debería poder controlar de forma eficaz un gran número de modelos en cola. Puede aumentar el número de nodos de clúster a medida que aumenten los tamaños de los conjunto de datos.

Por comodidad en este escenario, se envía una tarea de puntuación dentro de un trabajo de Batch AI individual. Sin embargo, es posible que sea más eficaz puntuar varios fragmentos de datos dentro del mismo trabajo de Batch AI. En esos casos, escriba código personalizado para leer en varios conjuntos de datos y ejecutar el script de puntuación para aquellos durante la ejecución de un único trabajo de Batch AI.

### <a name="file-servers"></a>Servidores de archivos

Al usar Batch AI, puede elegir varias opciones de almacenamiento según el rendimiento necesario para el escenario. Para cargas de trabajo con requisitos de rendimiento bajo, debe ser suficiente con Blob Storage. De forma alternativa, Batch AI también admite un [servidor de archivos de Batch AI][bai-file-server], un NFS administrado de un solo nodo que se puede montar automáticamente en los nodos del clúster para proporcionar una ubicación de almacenamiento accesible centralmente para los trabajos. Para la mayoría de los casos, se necesita un solo servidor de archivos en un área de trabajo y se pueden separar los datos para los trabajos de aprendizaje en directorios distintos.

Si un NFS de un solo nodo no es adecuado para las cargas de trabajo, Batch AI admite otras opciones de almacenamiento, como [Azure Files][azure-files], y soluciones personalizadas, como un sistema de archivos Gluster o Lustre.

## <a name="management-considerations"></a>Consideraciones de administración

### <a name="monitoring-batch-ai-jobs"></a>Supervisión de trabajos de Batch AI

Es importante supervisar el progreso de los trabajos en ejecución, pero puede ser un desafío hacerlo en un clúster de nodos activos. Para obtener una idea del estado general del clúster, vaya a la hoja **Batch AI** de [Azure Portal][portal] para inspeccionar el estado de los nodos del clúster. Si algún nodo está inactivo o se produce un error en algún trabajo, los registros de errores se guardan en Blob Storage, pero también se puede acceder a ellos desde la hoja **Trabajos** del portal.

Para que la supervisión sea más exhaustiva, conecte los registros a [Application Insights][ai] o ejecute procesos independientes para sondear el estado del clúster de Batch AI y sus trabajos.

### <a name="logging-in-batch-ai"></a>Registros en Batch AI

Batch AI registra todos los stdout/stderr en la cuenta de Azure Storage asociada. Para facilitar la navegación en los archivos de registro, use una herramienta de exploración de almacenamiento como [Explorador de Azure Storage][explorer].

Al implementar esta arquitectura de referencia, tiene la opción de configurar un sistema de registro más sencillo. Con esta opción, todos los registros de los distintos trabajos que se guardan en el mismo directorio del contenedor de blobs, como se muestra a continuación. Use dichos registros para supervisar el tiempo que tardan en procesarse casa trabajo y cada imagen, con el fin de que sepa mejor cómo optimizar el proceso.

![Explorador de Azure Storage](./_images/batch-scoring-python-monitor.png)

## <a name="cost-considerations"></a>Consideraciones sobre el costo

Los componentes más caros que se usan en esta arquitectura de referencia son los recursos de proceso.

El tamaño del clúster de Batch AI se escala y se reduce verticalmente en función de los trabajos que haya en la cola. El [escalado automático][automatic-scaling] se puede habilitar con Batch AI de una de las dos formas siguientes. Se puede hacer mediante programación, lo que se puede configurar en el archivo .env que forma parte de los [pasos de implementación][github], o bien se puede cambiar la fórmula de escalado directamente en el portal después de crear el clúster.

Para el trabajo que no requiera un procesamiento inmediato, configure la fórmula de escalado automático, con el fin de que el estado predeterminado (mínimo) sea un clúster de cero nodos. Con esta configuración, el clúster empieza con cero nodos y solo se escala verticalmente si detecta trabajos en la cola. Si el proceso de puntuación de Batch solo se produce algunas veces al día o menos, esta configuración permite obtener importantes ahorros.

Es posible que el escalado no sea apropiado para los trabajos por lotes que realizan con muy poco tiempo de diferencia entre sí. El tiempo que tarda un clúster en agilizarse y ralentizarse también incurre en costos, por tanto, si una carga de trabajo por lotes empieza solo unos minutos después de que el trabajo anterior termine, puede resultar más rentable mantener el clúster en ejecución entre los trabajos. Eso depende de si los procesos de puntuación están programados para ejecutarse con mucha frecuencia (por ejemplo, cada hora), o con poca (por ejemplo, una vez al mes).

## <a name="deploy-the-solution"></a>Implementación de la solución

Hay disponible una implementación de referencia de esta arquitectura en [GitHub][github]. Para crear una solución escalable que permita puntuar muchos modelos en paralelo mediante Batch AI, siga los pasos de la configuración.

[acr]: /azure/container-registry/container-registry-intro
[ai]: /azure/application-insights/app-insights-overview
[automatic-scaling]: /azure/batch/batch-automatic-scaling
[azure-files]: /azure/storage/files/storage-files-introduction
[batch-ai]: /azure/batch-ai/
[bai-file-server]: /azure/batch-ai/resource-concepts#file-server
[create-resources]: https://github.com/Azure/BatchAIAnomalyDetection/blob/master/create_resources.ipynb
[deep]: /azure/architecture/reference-architectures/ai/batch-scoring-deep-learning
[event-hubs]: /azure/event-hubs/event-hubs-geo-dr
[explorer]: https://azure.microsoft.com/en-us/features/storage-explorer/
[github]: https://github.com/Azure/BatchAIAnomalyDetection
[logic-apps]: /azure/logic-apps/logic-apps-overview
[one-class-svm]: http://scikit-learn.org/stable/modules/generated/sklearn.svm.OneClassSVM.html
[portal]: https://portal.azure.com
[python-script]: https://github.com/Azure/BatchAIAnomalyDetection/blob/master/batchai/predict.py
[script]: https://github.com/Azure/BatchAIAnomalyDetection/blob/master/sched/submit_jobs.py
[storage]: /azure/storage/blobs/storage-blobs-overview
[stream-analytics]: /azure/stream-analytics/
