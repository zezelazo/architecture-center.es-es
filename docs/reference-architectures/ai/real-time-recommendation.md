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
# <a name="build-a-real-time-recommendation-api-on-azure"></a><span data-ttu-id="513ce-103">Compilación de una API de recomendaciones en tiempo real en Azure</span><span class="sxs-lookup"><span data-stu-id="513ce-103">Build a real-time recommendation API on Azure</span></span>

<span data-ttu-id="513ce-104">Esta arquitectura de referencia muestra cómo entrenar un modelo de recomendación mediante Azure Databricks y cómo implementarlo como una API mediante el uso de Azure Cosmos DB, Azure Machine Learning y Azure Kubernetes Service (AKS).</span><span class="sxs-lookup"><span data-stu-id="513ce-104">This reference architecture shows how to train a recommendation model using Azure Databricks and deploy it as an API by using Azure Cosmos DB, Azure Machine Learning, and Azure Kubernetes Service (AKS).</span></span> <span data-ttu-id="513ce-105">Esta arquitectura se puede generalizar para la mayoría de los escenarios del motor de recomendaciones, lo que incluye recomendaciones de productos, películas y noticias.</span><span class="sxs-lookup"><span data-stu-id="513ce-105">This architecture can be generalized for most recommendation engine scenarios, including recommendations for products, movies, and news.</span></span>

<span data-ttu-id="513ce-106">Hay disponible una implementación de referencia de esta arquitectura en [GitHub](https://github.com/Microsoft/Recommenders/blob/master/notebooks/04_operationalize/als_movie_o16n.ipynb).</span><span class="sxs-lookup"><span data-stu-id="513ce-106">A reference implementation for this architecture is available on [GitHub](https://github.com/Microsoft/Recommenders/blob/master/notebooks/04_operationalize/als_movie_o16n.ipynb).</span></span>

![Arquitectura de un modelo de Machine Learning para el entrenamiento de recomendaciones de películas](./_images/recommenders-architecture.png)

<span data-ttu-id="513ce-108">**Escenario**: Una organización de elementos multimedia desea proporcionar recomendaciones de películas o vídeos a sus usuarios.</span><span class="sxs-lookup"><span data-stu-id="513ce-108">**Scenario**: A media organization wants to provide movie or video recommendations to its users.</span></span> <span data-ttu-id="513ce-109">Al proporcionar recomendaciones personalizadas, la organización cumple varios objetivos empresariales, entre los que se incluyen una mayor proporción de clics, un aumento en la involucración en el sitio y una mayor satisfacción del usuario.</span><span class="sxs-lookup"><span data-stu-id="513ce-109">By providing personalized recommendations, the organization meets several business goals, including increased click-through rates, increased engagement on site, and higher user satisfaction.</span></span>

<span data-ttu-id="513ce-110">Esta arquitectura de referencia es para entrenar e implementar una API del servicio de recomendaciones en tiempo real que pueda proporcionar las recomendaciones de las diez mejores películas para un usuario dado.</span><span class="sxs-lookup"><span data-stu-id="513ce-110">This reference architecture is for training and deploying a real-time recommender service API that can provide the top 10 movie recommendations for a given user.</span></span>

<span data-ttu-id="513ce-111">Este es el flujo de datos de este modelo de recomendaciones:</span><span class="sxs-lookup"><span data-stu-id="513ce-111">The data flow for this recommendation model is as follows:</span></span>

1. <span data-ttu-id="513ce-112">Seguimiento de los comportamientos de los usuarios.</span><span class="sxs-lookup"><span data-stu-id="513ce-112">Track user behaviors.</span></span> <span data-ttu-id="513ce-113">Por ejemplo, un servicio back-end podría crear un registro cada vez que un usuario evalúa una película o hace clic en un artículo acerca de un producto o una noticia.</span><span class="sxs-lookup"><span data-stu-id="513ce-113">For example, a backend service might log when a user rates a movie or clicks a product or news article.</span></span>

2. <span data-ttu-id="513ce-114">Cargue los datos en Azure Databricks desde un [origen de datos][data-source] disponible.</span><span class="sxs-lookup"><span data-stu-id="513ce-114">Load the data into Azure Databricks from an available [data source][data-source].</span></span>

3. <span data-ttu-id="513ce-115">Prepare los datos y divídalos en conjuntos de entrenamiento y de pruebas para entrenar el modelo</span><span class="sxs-lookup"><span data-stu-id="513ce-115">Prepare the data and split it into training and testing sets to train the model.</span></span> <span data-ttu-id="513ce-116">([esta guía][guide] describe las opciones para dividir los datos).</span><span class="sxs-lookup"><span data-stu-id="513ce-116">([This guide][guide] describes options for splitting data.)</span></span>

4. <span data-ttu-id="513ce-117">Ajuste el modelo de [filtrado en colaboración de Spark][als] modelos a los datos.</span><span class="sxs-lookup"><span data-stu-id="513ce-117">Fit the [Spark Collaborative Filtering][als] model to the data.</span></span>

5. <span data-ttu-id="513ce-118">Evalúe la calidad del modelo mediante métricas de clasificación.</span><span class="sxs-lookup"><span data-stu-id="513ce-118">Evaluate the quality of the model using rating and ranking metrics.</span></span> <span data-ttu-id="513ce-119">(en [esta guía][eval-guide] se proporcionan detalles acerca de las métricas por las que se pueden realizar las recomendaciones).</span><span class="sxs-lookup"><span data-stu-id="513ce-119">([This guide][eval-guide] provides details about the metrics you can evaluate your recommender on.)</span></span>

6. <span data-ttu-id="513ce-120">Calcule previamente las diez principales recomendaciones por usuario y almacénelas como una memoria caché en Azure Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="513ce-120">Precompute the top 10 recommendations per user and store as a cache in Azure Cosmos DB.</span></span>

7. <span data-ttu-id="513ce-121">Implemente un servicio de API en AKS mediante las API de Azure Machine Learning para incluir la API en un contenedor e implementarla.</span><span class="sxs-lookup"><span data-stu-id="513ce-121">Deploy an API service to AKS using the Azure Machine Learning APIs to containerize and deploy the API.</span></span>

8. <span data-ttu-id="513ce-122">Cuando el servicio de back-end recibe una solicitud de un usuario, llame a la API de recomendaciones hospedada en AKS para obtener las diez principales recomendaciones y mostrárselas al usuario.</span><span class="sxs-lookup"><span data-stu-id="513ce-122">When the backend service gets a request from a user, call the recommendations API hosted in AKS to get the top 10 recommendations and display them to the user.</span></span>

## <a name="architecture"></a><span data-ttu-id="513ce-123">Arquitectura</span><span class="sxs-lookup"><span data-stu-id="513ce-123">Architecture</span></span>

<span data-ttu-id="513ce-124">Esta arquitectura consta de los siguientes componentes:</span><span class="sxs-lookup"><span data-stu-id="513ce-124">This architecture consists of the following components:</span></span>

<span data-ttu-id="513ce-125">[Azure Databricks][databricks].</span><span class="sxs-lookup"><span data-stu-id="513ce-125">[Azure Databricks][databricks].</span></span> <span data-ttu-id="513ce-126">Databricks es un entorno de desarrollo que se usa para preparar los datos de entrada y entrenar el modelo de recomendación en un clúster de Spark.</span><span class="sxs-lookup"><span data-stu-id="513ce-126">Databricks is a development environment used to prepare input data and train the recommender model on a Spark cluster.</span></span> <span data-ttu-id="513ce-127">Azure Databricks también proporciona un área de trabajo interactiva para ejecutar cuadernos y colaborar en ellos en todas las tareas de procesamiento de datos o aprendizaje automático.</span><span class="sxs-lookup"><span data-stu-id="513ce-127">Azure Databricks also provides an interactive workspace to run and collaborate on notebooks for any data processing or machine learning tasks.</span></span>

<span data-ttu-id="513ce-128">[Azure Kubernetes Service][aks] (AKS).</span><span class="sxs-lookup"><span data-stu-id="513ce-128">[Azure Kubernetes Service][aks] (AKS).</span></span> <span data-ttu-id="513ce-129">AKS se usa para implementar y hacer operativa una API de servicio de un modelo de Machine Learning en un clúster de Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="513ce-129">AKS is used to deploy and operationalize a machine learning model service API on a Kubernetes cluster.</span></span> <span data-ttu-id="513ce-130">AKS hospeda el modelo en contenedores, lo que proporciona una escalabilidad que cumple los requisitos de rendimiento, la administración de identidades y accesos, y la supervisión del registro y mantenimiento.</span><span class="sxs-lookup"><span data-stu-id="513ce-130">AKS hosts the containerized model, providing scalability that meets your throughput requirements, identity and access management, and logging and health monitoring.</span></span>

<span data-ttu-id="513ce-131">[Azure Cosmos DB][cosmosdb].</span><span class="sxs-lookup"><span data-stu-id="513ce-131">[Azure Cosmos DB][cosmosdb].</span></span> <span data-ttu-id="513ce-132">Cosmos DB es un servicio de base de datos distribuida globalmente que se utiliza para almacenar las diez películas recomendadas a cada usuario.</span><span class="sxs-lookup"><span data-stu-id="513ce-132">Cosmos DB is a globally distributed database service used to store the top 10 recommended movies for each user.</span></span> <span data-ttu-id="513ce-133">Azure Cosmos DB es ideal para este escenario, ya que proporciona baja latencia (10 ms en el percentil 99) para leer los principales elementos recomendados para un usuario dado.</span><span class="sxs-lookup"><span data-stu-id="513ce-133">Azure Cosmos DB is well-suited for this scenario, because it provides low latency (10 ms at 99th percentile) to read the top recommended items for a given user.</span></span>

<span data-ttu-id="513ce-134">[Azure Machine Learning Service][mls].</span><span class="sxs-lookup"><span data-stu-id="513ce-134">[Azure Machine Learning Service][mls].</span></span> <span data-ttu-id="513ce-135">Este servicio se utiliza para realizar un seguimiento y administrar modelos de Machine Learning y, después, empaquetar e implementar dichos modelos en un entorno de AKS escalable.</span><span class="sxs-lookup"><span data-stu-id="513ce-135">This service is used to track and manage machine learning models, and then package and deploy these models to a scalable AKS environment.</span></span>

<span data-ttu-id="513ce-136">[Microsoft Recommenders][github].</span><span class="sxs-lookup"><span data-stu-id="513ce-136">[Microsoft Recommenders][github].</span></span> <span data-ttu-id="513ce-137">Este repositorio de código abierto contiene código y ejemplos de la utilidad que ayudan a los usuarios empezar a compilar, evaluar y poner en marcha un sistema de recomendaciones.</span><span class="sxs-lookup"><span data-stu-id="513ce-137">This open-source repository contains utility code and samples to help users get started in building, evaluating, and operationalizing a recommender system.</span></span>

## <a name="performance-considerations"></a><span data-ttu-id="513ce-138">Consideraciones sobre rendimiento</span><span class="sxs-lookup"><span data-stu-id="513ce-138">Performance considerations</span></span>

<span data-ttu-id="513ce-139">El rendimiento es una consideración primordial para las recomendaciones en tiempo real, ya que las recomendaciones normalmente se encuentran en la ruta crítica de la solicitud que un usuario realiza en el sitio.</span><span class="sxs-lookup"><span data-stu-id="513ce-139">Performance is a primary consideration for real-time recommendations, because recommendations usually fall in the critical path of the request a user makes on your site.</span></span>

<span data-ttu-id="513ce-140">La combinación de AKS y Azure Cosmos DB permite a esta arquitectura proporcionar un buen punto de partida para ofrecer recomendaciones para una carga de trabajo de tamaño medio con una sobrecarga mínima.</span><span class="sxs-lookup"><span data-stu-id="513ce-140">The combination of AKS and Azure Cosmos DB enables this architecture to provide a good starting point to provide recommendations for a medium-sized workload with minimal overhead.</span></span> <span data-ttu-id="513ce-141">En una prueba de carga con 200 usuarios simultáneos, esta arquitectura proporciona recomendaciones a una latencia mediana de aproximadamente 60 ms y funciona con un rendimiento de 180 solicitudes por segundo.</span><span class="sxs-lookup"><span data-stu-id="513ce-141">Under a load test with 200 concurrent users, this architecture provides recommendations at a median latency of about 60 ms and performs at a throughput of 180 requests per second.</span></span> <span data-ttu-id="513ce-142">La prueba de carga se ejecutó en la configuración de implementación predeterminado [un clúster de AKS 3x D3 v2 con 12 vCPU, 42 GB de memoria y 11 000 [unidades de solicitud (RU) por segundo] [ru] aprovisionadas para Azure Cosmos DB].</span><span class="sxs-lookup"><span data-stu-id="513ce-142">The load test was run against the default deployment configuration (a 3x D3 v2 AKS cluster with 12 vCPUs, 42 GB of memory, and 11,000 [Request Units (RUs) per second][ru] provisioned for Azure Cosmos DB).</span></span>

![Gráfico de rendimiento](./_images/recommenders-performance.png)

![Gráfico de capacidad de proceso](./_images/recommenders-throughput.png)

<span data-ttu-id="513ce-145">Azure Cosmos DB se recomienda por su distribución global llave en mano y su utilidad para satisfacer todos los requisitos de base de datos que tiene la aplicación.</span><span class="sxs-lookup"><span data-stu-id="513ce-145">Azure Cosmos DB is recommended for its turnkey global distribution and usefulness in meeting any database requirements your app has.</span></span> <span data-ttu-id="513ce-146">Para una [latencia un poco menor][latency], considere la posibilidad de uso [Azure Redis Cache][redis], en lugar de Azure Cosmos DB para atender las búsquedas.</span><span class="sxs-lookup"><span data-stu-id="513ce-146">For slightly [faster latency][latency], consider using [Azure Redis Cache][redis] instead of Azure Cosmos DB to serve lookups.</span></span> <span data-ttu-id="513ce-147">Redis Cache puede mejorar el rendimiento de los sistemas que usan intensamente los datos de almacenes de back-end.</span><span class="sxs-lookup"><span data-stu-id="513ce-147">Redis Cache can improve performance of systems that rely highly on data in back-end stores.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="513ce-148">Consideraciones sobre escalabilidad</span><span class="sxs-lookup"><span data-stu-id="513ce-148">Scalability considerations</span></span>

<span data-ttu-id="513ce-149">Si no planea usar Spark o tiene una carga de trabajo más pequeño, en la que no necesita la distribución, considere la posibilidad de usar [Data Science Virtual Machine][dsvm] (DSVM), en lugar de Azure Databricks.</span><span class="sxs-lookup"><span data-stu-id="513ce-149">If you don't plan to use Spark, or you have a smaller workload where you don't need distribution, consider using [Data Science Virtual Machine][dsvm] (DSVM) instead of Azure Databricks.</span></span> <span data-ttu-id="513ce-150">DSVM es una máquina virtual de Azure con marcos y herramientas de aprendizaje profundo para el aprendizaje automático y la ciencia de datos.</span><span class="sxs-lookup"><span data-stu-id="513ce-150">DSVM is an Azure virtual machine with deep learning frameworks and tools for machine learning and data science.</span></span> <span data-ttu-id="513ce-151">Igual que sucede con Azure Databricks, cualquier modelo que se crea en una instancia de DSVM puedan usarse como un servicio en AKS mediante Azure Machine Learning.</span><span class="sxs-lookup"><span data-stu-id="513ce-151">As with Azure Databricks, any model you create in a DSVM can be operationalized as a service on AKS via Azure Machine Learning.</span></span>

<span data-ttu-id="513ce-152">Durante el entrenamiento, aprovisione un clúster de Spark de mayor tamaño fijo en Azure Databricks o configure el [escalado automático][autoscaling].</span><span class="sxs-lookup"><span data-stu-id="513ce-152">During training, provision a larger fixed-size Spark cluster in Azure Databricks or configure [autoscaling][autoscaling].</span></span> <span data-ttu-id="513ce-153">Cuando el escalado automático está habilitado, Databricks supervisa la carga en el clúster y escala y reduce verticalmente cuando sea necesario.</span><span class="sxs-lookup"><span data-stu-id="513ce-153">When autoscaling is enabled, Databricks monitors the load on your cluster and scales up and downs when required.</span></span> <span data-ttu-id="513ce-154">Aprovisione o escale horizontalmente un clúster mayor si tiene un tamaño de datos grande y desea reducir el tiempo que tarda en preparar datos o modelar tareas.</span><span class="sxs-lookup"><span data-stu-id="513ce-154">Provision or scale out a larger cluster if you have a large data size and you want to reduce the amount of time it takes for data preparation or modeling tasks.</span></span>

<span data-ttu-id="513ce-155">Escale el clúster de AKS para satisfacer sus necesidades de rendimiento.</span><span class="sxs-lookup"><span data-stu-id="513ce-155">Scale the AKS cluster to meet your performance and throughput requirements.</span></span> <span data-ttu-id="513ce-156">Procure escalar verticalmente el número de [pods][scale] para utilizar plenamente el clúster y escalar la [nodos][nodes] del clúster para satisfacer la demanda del servicio.</span><span class="sxs-lookup"><span data-stu-id="513ce-156">Take care to scale up the number of [pods][scale] to fully utilize the cluster, and to scale the [nodes][nodes] of the cluster to meet the demand of your service.</span></span> <span data-ttu-id="513ce-157">Para más información acerca de cómo escalar un clúster para satisfacer los requisitos de rendimiento del servicio de recomendaciones, consulte [Escalado de clústeres de Azure Container Service][blog].</span><span class="sxs-lookup"><span data-stu-id="513ce-157">For more information on how to scale your cluster to meet the performance and throughput requirements of your recommender service, see [Scaling Azure Container Service Clusters][blog].</span></span>

<span data-ttu-id="513ce-158">Para administrar el rendimiento de Azure Cosmos DB, calcule el número de operaciones de lectura necesarias por segundo y aprovisione el número de [RU por segundo][ru] (rendimiento) necesarias.</span><span class="sxs-lookup"><span data-stu-id="513ce-158">To manage Azure Cosmos DB performance, estimate the number of reads required per second, and provision the number of [RUs per second][ru] (throughput) needed.</span></span> <span data-ttu-id="513ce-159">Utilice los procedimientos recomendados para [crear particiones y realizar el escalado horizontal][partition-data].</span><span class="sxs-lookup"><span data-stu-id="513ce-159">Use best practices for [partitioning and horizontal scaling][partition-data].</span></span>

## <a name="cost-considerations"></a><span data-ttu-id="513ce-160">Consideraciones sobre el costo</span><span class="sxs-lookup"><span data-stu-id="513ce-160">Cost considerations</span></span>

<span data-ttu-id="513ce-161">Los controladores principales del costo en este escenario son:</span><span class="sxs-lookup"><span data-stu-id="513ce-161">The main drivers of cost in this scenario are:</span></span>

- <span data-ttu-id="513ce-162">El tamaño de clúster de Azure Databricks necesario para el entrenamiento.</span><span class="sxs-lookup"><span data-stu-id="513ce-162">The Azure Databricks cluster size required for training.</span></span>
- <span data-ttu-id="513ce-163">El tamaño de clúster de AKS necesario para cumplir los requisitos de rendimiento.</span><span class="sxs-lookup"><span data-stu-id="513ce-163">The AKS cluster size required to meet your performance requirements.</span></span>
- <span data-ttu-id="513ce-164">Las unidades de solicitud de Azure Cosmos DB aprovisionadas para cumplir los requisitos de rendimiento.</span><span class="sxs-lookup"><span data-stu-id="513ce-164">Azure Cosmos DB RUs provisioned to meet your performance requirements.</span></span>

<span data-ttu-id="513ce-165">Administre los costos de Azure Databricks mediante la reducción de la frecuencia del entrenamiento y la desactivación del clúster de Spark cuando no esté en uso.</span><span class="sxs-lookup"><span data-stu-id="513ce-165">Manage the Azure Databricks costs by retraining less frequently and turning off the Spark cluster when not in use.</span></span> <span data-ttu-id="513ce-166">Los costos de AKS y de Azure Cosmos DB están enlazados al rendimiento que requiere su sitio y se escalarán y reducirán verticalmente en función del volumen de tráfico del sitio.</span><span class="sxs-lookup"><span data-stu-id="513ce-166">The AKS and Azure Cosmos DB costs are tied to the throughput and performance required by your site and will scale up and down depending on the volume of traffic to your site.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="513ce-167">Implementación de la solución</span><span class="sxs-lookup"><span data-stu-id="513ce-167">Deploy the solution</span></span>

<span data-ttu-id="513ce-168">Para implementar esta arquitectura, en primer lugar cree un entorno de Azure Databricks para preparar los datos y entrenar un modelo de recomendaciones:</span><span class="sxs-lookup"><span data-stu-id="513ce-168">To deploy this architecture, first create an Azure Databricks environment to prepare data and train a recommender model:</span></span>

1. <span data-ttu-id="513ce-169">Cree un [área de trabajo de Azure Databricks][workspace].</span><span class="sxs-lookup"><span data-stu-id="513ce-169">Create an [Azure Databricks workspace][workspace].</span></span>

2. <span data-ttu-id="513ce-170">Cree un clúster en Azure Databricks.</span><span class="sxs-lookup"><span data-stu-id="513ce-170">Create a new cluster in Azure Databricks.</span></span> <span data-ttu-id="513ce-171">Se requiere la siguiente configuración:</span><span class="sxs-lookup"><span data-stu-id="513ce-171">The following configuration is required:</span></span>

    - <span data-ttu-id="513ce-172">Modo de clúster: Estándar</span><span class="sxs-lookup"><span data-stu-id="513ce-172">Cluster mode: Standard</span></span>
    - <span data-ttu-id="513ce-173">Versión de Databricks Runtime: 4.1 (incluye Apache Spark 2.3.0 y Scala 2.11)</span><span class="sxs-lookup"><span data-stu-id="513ce-173">Databricks Runtime Version: 4.1 (includes Apache Spark 2.3.0, Scala 2.11)</span></span>
    - <span data-ttu-id="513ce-174">Versión de Python: 3</span><span class="sxs-lookup"><span data-stu-id="513ce-174">Python Version: 3</span></span>
    - <span data-ttu-id="513ce-175">Tipo de controlador: Standard\_DS3\_v2</span><span class="sxs-lookup"><span data-stu-id="513ce-175">Driver Type: Standard\_DS3\_v2</span></span>
    - <span data-ttu-id="513ce-176">Tipo de trabajo: Standard\_DS3\_v2 (mín. y máx., según sea necesario)</span><span class="sxs-lookup"><span data-stu-id="513ce-176">Worker Type: Standard\_DS3\_v2 (min and max as required)</span></span>
    - <span data-ttu-id="513ce-177">Terminación automática: (según sea necesario)</span><span class="sxs-lookup"><span data-stu-id="513ce-177">Auto Termination: (as required)</span></span>
    - <span data-ttu-id="513ce-178">Configuración de Spark: (según sea necesario)</span><span class="sxs-lookup"><span data-stu-id="513ce-178">Spark Config: (as required)</span></span>
    - <span data-ttu-id="513ce-179">Variables de entorno: (según sea necesario)</span><span class="sxs-lookup"><span data-stu-id="513ce-179">Environment Variables: (as required)</span></span>

3. <span data-ttu-id="513ce-180">Clone el repositorio [Microsoft Recommenders][github] del equipo local.</span><span class="sxs-lookup"><span data-stu-id="513ce-180">Clone the [Microsoft Recommenders][github] repository on your local computer.</span></span>

4. <span data-ttu-id="513ce-181">Comprima el contenido de la carpeta Recommenders:</span><span class="sxs-lookup"><span data-stu-id="513ce-181">Zip the content inside the Recommenders folder:</span></span>

    ```console
    cd Recommenders
    zip -r Recommenders.zip
    ```

5. <span data-ttu-id="513ce-182">Adjunte la biblioteca Recommenders al clúster como sigue:</span><span class="sxs-lookup"><span data-stu-id="513ce-182">Attach the Recommenders library to your cluster as follows:</span></span>

    1. <span data-ttu-id="513ce-183">En el siguiente menú, use la opción para importar una biblioteca ("para importar una biblioteca, como un archivo jar o huevo, haga clic aquí") y presione **haga clic aquí**.</span><span class="sxs-lookup"><span data-stu-id="513ce-183">In the next menu, use the option to import a library ("To import a library, such as a jar or egg, click here") and press **click here**.</span></span>

    2. <span data-ttu-id="513ce-184">En el primer menú desplegable, seleccione la opción **Upload Python egg or PyPI** (Cargar egg o PyPI de Python).</span><span class="sxs-lookup"><span data-stu-id="513ce-184">At the first drop-down menu, select the **Upload Python egg or PyPI** option.</span></span>

    3. <span data-ttu-id="513ce-185">Seleccione **Drop library egg here to upload** (Coloque aquí el archivo egg de la biblioteca para cargarlo) y seleccione el archivo Recommenders.zip que acaba de crear.</span><span class="sxs-lookup"><span data-stu-id="513ce-185">Select **Drop library egg here to upload** and select the Recommenders.zip file you just created.</span></span>

    4. <span data-ttu-id="513ce-186">Seleccione **Create library** (Crear biblioteca) para cargar el archivo .zip y que esté disponible en el área de trabajo.</span><span class="sxs-lookup"><span data-stu-id="513ce-186">Select **Create library** to upload the .zip file and make it available in your workspace.</span></span>

    5. <span data-ttu-id="513ce-187">En el menú siguiente, adjunte la biblioteca al clúster.</span><span class="sxs-lookup"><span data-stu-id="513ce-187">In the next menu, attach the library to your cluster.</span></span>

6. <span data-ttu-id="513ce-188">En el área de trabajo, importe el [ejemplo de ALS Movie Operationalization][als-example].</span><span class="sxs-lookup"><span data-stu-id="513ce-188">In your workspace, import the [ALS Movie Operationalization example][als-example].</span></span>

7. <span data-ttu-id="513ce-189">Ejecute el cuaderno ALS Movie Operationalization para crear los recursos necesarios para crear una API que proporciona las diez recomendaciones de películas principales para un usuario determinado.</span><span class="sxs-lookup"><span data-stu-id="513ce-189">Run the ALS Movie Operationalization notebook to create the resources required to create a recommendation API that provides the top-10 movie recommendations for a given user.</span></span>

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
