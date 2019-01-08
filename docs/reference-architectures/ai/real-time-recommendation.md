---
title: Compilación de una API de recomendaciones en tiempo real en Azure
description: Use el aprendizaje automático para automatizar las recomendaciones mediante el uso de Azure Databricks y Azure Data Science Virtual Machines (DSVM) para entrenar un modelo en Azure.
author: njray
ms.date: 12/12/2018
ms.custom: azcat-ai
ms.openlocfilehash: ca9d854f0e29ae769f5a86648b94cce7a2fd146e
ms.sourcegitcommit: fb22348f917a76e30a6c090fcd4a18decba0b398
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/16/2018
ms.locfileid: "53450842"
---
# <a name="build-a-real-time-recommendation-api-on-azure"></a>Compilación de una API de recomendaciones en tiempo real en Azure

Esta arquitectura de referencia muestra cómo entrenar un modelo de recomendación mediante Azure Databricks y cómo implementarlo como una API mediante el uso de Azure Cosmos DB, Azure Machine Learning y Azure Kubernetes Service (AKS). Esta arquitectura se puede generalizar para la mayoría de los escenarios del motor de recomendaciones, lo que incluye recomendaciones de productos, películas y noticias.

Hay disponible una implementación de referencia de esta arquitectura en [GitHub](https://github.com/Microsoft/Recommenders/blob/master/notebooks/04_operationalize/als_movie_o16n.ipynb).

![Arquitectura de un modelo de Machine Learning para el entrenamiento de recomendaciones de películas](./_images/recommenders-architecture.png)

**Escenario**: Una organización de elementos multimedia desea proporcionar recomendaciones de películas o vídeos a sus usuarios. Al proporcionar recomendaciones personalizadas, la organización cumple varios objetivos empresariales, entre los que se incluyen una mayor proporción de clics, un aumento en la involucración en el sitio y una mayor satisfacción del usuario.

Esta arquitectura de referencia es para entrenar e implementar una API del servicio de recomendaciones en tiempo real que pueda proporcionar las recomendaciones de las diez mejores películas para un usuario dado.

Este es el flujo de datos de este modelo de recomendaciones:

1. Seguimiento de los comportamientos de los usuarios. Por ejemplo, un servicio back-end podría crear un registro cada vez que un usuario evalúa una película o hace clic en un artículo acerca de un producto o una noticia.

2. Cargue los datos en Azure Databricks desde un [origen de datos][data-source] disponible.

3. Prepare los datos y divídalos en conjuntos de entrenamiento y de pruebas para entrenar el modelo ([esta guía][guide] describe las opciones para dividir los datos).

4. Ajuste el modelo de [filtrado en colaboración de Spark][als] modelos a los datos.

5. Evalúe la calidad del modelo mediante métricas de clasificación. (en [esta guía][eval-guide] se proporcionan detalles acerca de las métricas por las que se pueden realizar las recomendaciones).

6. Calcule previamente las diez principales recomendaciones por usuario y almacénelas como una memoria caché en Azure Cosmos DB.

7. Implemente un servicio de API en AKS mediante las API de Azure Machine Learning para incluir la API en un contenedor e implementarla.

8. Cuando el servicio de back-end recibe una solicitud de un usuario, llame a la API de recomendaciones hospedada en AKS para obtener las diez principales recomendaciones y mostrárselas al usuario.

## <a name="architecture"></a>Arquitectura

Esta arquitectura consta de los siguientes componentes:

[Azure Databricks][databricks]. Databricks es un entorno de desarrollo que se usa para preparar los datos de entrada y entrenar el modelo de recomendación en un clúster de Spark. Azure Databricks también proporciona un área de trabajo interactiva para ejecutar cuadernos y colaborar en ellos en todas las tareas de procesamiento de datos o aprendizaje automático.

[Azure Kubernetes Service][aks] (AKS). AKS se usa para implementar y hacer operativa una API de servicio de un modelo de Machine Learning en un clúster de Kubernetes. AKS hospeda el modelo en contenedores, lo que proporciona una escalabilidad que cumple los requisitos de rendimiento, la administración de identidades y accesos, y la supervisión del registro y mantenimiento.

[Azure Cosmos DB][cosmosdb]. Cosmos DB es un servicio de base de datos distribuida globalmente que se utiliza para almacenar las diez películas recomendadas a cada usuario. Azure Cosmos DB es ideal para este escenario, ya que proporciona baja latencia (10 ms en el percentil 99) para leer los principales elementos recomendados para un usuario dado.

[Azure Machine Learning Service][mls]. Este servicio se utiliza para realizar un seguimiento y administrar modelos de Machine Learning y, después, empaquetar e implementar dichos modelos en un entorno de AKS escalable.

[Microsoft Recommenders][github]. Este repositorio de código abierto contiene código y ejemplos de la utilidad que ayudan a los usuarios empezar a compilar, evaluar y poner en marcha un sistema de recomendaciones.

## <a name="performance-considerations"></a>Consideraciones sobre rendimiento

El rendimiento es una consideración primordial para las recomendaciones en tiempo real, ya que las recomendaciones normalmente se encuentran en la ruta crítica de la solicitud que un usuario realiza en el sitio.

La combinación de AKS y Azure Cosmos DB permite a esta arquitectura proporcionar un buen punto de partida para ofrecer recomendaciones para una carga de trabajo de tamaño medio con una sobrecarga mínima. En una prueba de carga con 200 usuarios simultáneos, esta arquitectura proporciona recomendaciones a una latencia mediana de aproximadamente 60 ms y funciona con un rendimiento de 180 solicitudes por segundo. La prueba de carga se ejecutó en la configuración de implementación predeterminado [un clúster de AKS 3x D3 v2 con 12 vCPU, 42 GB de memoria y 11 000 [unidades de solicitud (RU) por segundo] [ru] aprovisionadas para Azure Cosmos DB].

![Gráfico de rendimiento](./_images/recommenders-performance.png)

![Gráfico de capacidad de proceso](./_images/recommenders-throughput.png)

Azure Cosmos DB se recomienda por su distribución global llave en mano y su utilidad para satisfacer todos los requisitos de base de datos que tiene la aplicación. Para una [latencia un poco menor][latency], considere la posibilidad de uso [Azure Redis Cache][redis], en lugar de Azure Cosmos DB para atender las búsquedas. Redis Cache puede mejorar el rendimiento de los sistemas que usan intensamente los datos de almacenes de back-end.

## <a name="scalability-considerations"></a>Consideraciones sobre escalabilidad

Si no planea usar Spark o tiene una carga de trabajo más pequeño, en la que no necesita la distribución, considere la posibilidad de usar [Data Science Virtual Machine][dsvm] (DSVM), en lugar de Azure Databricks. DSVM es una máquina virtual de Azure con marcos y herramientas de aprendizaje profundo para el aprendizaje automático y la ciencia de datos. Igual que sucede con Azure Databricks, cualquier modelo que se crea en una instancia de DSVM puedan usarse como un servicio en AKS mediante Azure Machine Learning.

Durante el entrenamiento, aprovisione un clúster de Spark de mayor tamaño fijo en Azure Databricks o configure el [escalado automático][autoscaling]. Cuando el escalado automático está habilitado, Databricks supervisa la carga en el clúster y escala y reduce verticalmente cuando sea necesario. Aprovisione o escale horizontalmente un clúster mayor si tiene un tamaño de datos grande y desea reducir el tiempo que tarda en preparar datos o modelar tareas.

Escale el clúster de AKS para satisfacer sus necesidades de rendimiento. Procure escalar verticalmente el número de [pods][scale] para utilizar plenamente el clúster y escalar la [nodos][nodes] del clúster para satisfacer la demanda del servicio. Para más información acerca de cómo escalar un clúster para satisfacer los requisitos de rendimiento del servicio de recomendaciones, consulte [Escalado de clústeres de Azure Container Service][blog].

Para administrar el rendimiento de Azure Cosmos DB, calcule el número de operaciones de lectura necesarias por segundo y aprovisione el número de [RU por segundo][ru] (rendimiento) necesarias. Utilice los procedimientos recomendados para [crear particiones y realizar el escalado horizontal][partition-data].

## <a name="cost-considerations"></a>Consideraciones sobre el costo

Los controladores principales del costo en este escenario son:

- El tamaño de clúster de Azure Databricks necesario para el entrenamiento.
- El tamaño de clúster de AKS necesario para cumplir los requisitos de rendimiento.
- Las unidades de solicitud de Azure Cosmos DB aprovisionadas para cumplir los requisitos de rendimiento.

Administre los costos de Azure Databricks mediante la reducción de la frecuencia del entrenamiento y la desactivación del clúster de Spark cuando no esté en uso. Los costos de AKS y de Azure Cosmos DB están enlazados al rendimiento que requiere su sitio y se escalarán y reducirán verticalmente en función del volumen de tráfico del sitio.

## <a name="deploy-the-solution"></a>Implementación de la solución

Para implementar esta arquitectura, en primer lugar cree un entorno de Azure Databricks para preparar los datos y entrenar un modelo de recomendaciones:

1. Cree un [área de trabajo de Azure Databricks][workspace].

2. Cree un clúster en Azure Databricks. Se requiere la siguiente configuración:

    - Modo de clúster: Estándar
    - Versión de Databricks Runtime: 4.1 (incluye Apache Spark 2.3.0 y Scala 2.11)
    - Versión de Python: 3
    - Tipo de controlador: Standard\_DS3\_v2
    - Tipo de trabajo: Standard\_DS3\_v2 (mín. y máx., según sea necesario)
    - Terminación automática: (según sea necesario)
    - Configuración de Spark: (según sea necesario)
    - Variables de entorno: (según sea necesario)

3. Clone el repositorio [Microsoft Recommenders][github] del equipo local.

4. Comprima el contenido de la carpeta Recommenders:

    ```console
    cd Recommenders
    zip -r Recommenders.zip
    ```

5. Adjunte la biblioteca Recommenders al clúster como sigue:

    1. En el siguiente menú, use la opción para importar una biblioteca ("para importar una biblioteca, como un archivo jar o huevo, haga clic aquí") y presione **haga clic aquí**.

    2. En el primer menú desplegable, seleccione la opción **Upload Python egg or PyPI** (Cargar egg o PyPI de Python).

    3. Seleccione **Drop library egg here to upload** (Coloque aquí el archivo egg de la biblioteca para cargarlo) y seleccione el archivo Recommenders.zip que acaba de crear.

    4. Seleccione **Create library** (Crear biblioteca) para cargar el archivo .zip y que esté disponible en el área de trabajo.

    5. En el menú siguiente, adjunte la biblioteca al clúster.

6. En el área de trabajo, importe el [ejemplo de ALS Movie Operationalization][als-example].

7. Ejecute el cuaderno ALS Movie Operationalization para crear los recursos necesarios para crear una API que proporciona las diez recomendaciones de películas principales para un usuario determinado.

<!-- links -->
[aci]: /azure/container-instances/container-instances-overview
[aad]: /azure/active-directory-b2c/active-directory-b2c-overview
[aks]: /azure/aks/intro-kubernetes
[als]: https://spark.apache.org/docs/latest/ml-collaborative-filtering.html
[als-example]: https://github.com/Microsoft/Recommenders/blob/master/notebooks/04_operationalize/als_movie_o16n.ipynb
[autoscaling]: https://docs.azuredatabricks.net/user-guide/clusters/sizing.html
[autoscale]: https://docs.azuredatabricks.net/user-guide/clusters/sizing.html#autoscaling
[availability]: /azure/architecture/checklist/availability
[blob]: /azure/storage/blobs/storage-blobs-introduction
[blog]: https://blogs.technet.microsoft.com/machinelearning/2018/03/20/scaling-azure-container-service-cluster/
[clusters]: https://docs.azuredatabricks.net/user-guide/clusters/configure.html
[cosmosdb]: /azure/cosmos-db/introduction
[data-source]: https://docs.azuredatabricks.net/spark/latest/data-sources/index.html
[databricks]: /azure/azure-databricks/what-is-azure-databricks
[dsvm]: /azure/machine-learning/data-science-virtual-machine/overview
[dsvm-ubuntu]: /azure/machine-learning/data-science-virtual-machine/dsvm-ubuntu-intro
[eval-guide]: https://github.com/Microsoft/Recommenders/blob/master/notebooks/03_evaluate/evaluation.ipynb
[free]: https://azure.microsoft.com/free/?WT.mc_id=A261C142F
[github]: https://github.com/Microsoft/Recommenders
[guide]: https://github.com/Microsoft/Recommenders/blob/master/notebooks/01_prepare_data/data_split.ipynb
[latency]: https://github.com/jessebenson/azure-performance
[mls]: /azure/machine-learning/service/
[n-tier]: /azure/architecture/reference-architectures/n-tier/n-tier-cassandra
[ndcg]: https://en.wikipedia.org/wiki/Discounted_cumulative_gain
[nodes]: /azure/aks/scale-cluster
[notebook]: https://github.com/Microsoft/Recommenders/notebooks/00_quick_start/als_pyspark_movielens.ipynb
[partition-data]: /azure/cosmos-db/partition-data
[redis]: /azure/redis-cache/cache-overview
[regions]: https://azure.microsoft.com/en-us/global-infrastructure/services/?products=virtual-machines&regions=all
[resiliency]: /azure/architecture/resiliency/
[ru]: /azure/cosmos-db/request-units
[sec-docs]: /azure/security/
[setup]: https://github.com/Microsoft/Recommenders/blob/master/SETUP.md%60
[scale]: /azure/aks/tutorial-kubernetes-scale
[sla]: https://azure.microsoft.com/en-us/support/legal/sla/virtual-machines/v1_8/
[vm-size]: /azure/virtual-machines/virtual-machines-linux-change-vm-size
[workspace]: https://docs.azuredatabricks.net/getting-started/index.html
