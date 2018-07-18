---
title: 'Aplicaciones inteligentes: procesamiento de imágenes en Azure'
description: Solución probada para incorporar procesamiento de imágenes en sus aplicaciones de Azure.
author: david-stanford
ms.date: 07/05/2018
ms.openlocfilehash: c5bfb9a929ddddda4336e1cbc8665a0b4d3bbe2c
ms.sourcegitcommit: 5d99b195388b7cabba383c49a81390ac48f86e8a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/06/2018
ms.locfileid: "37891386"
---
# <a name="insurance-claim-image-classification-on-azure"></a><span data-ttu-id="83e70-103">Clasificación de imágenes de reclamaciones de seguros en Azure</span><span class="sxs-lookup"><span data-stu-id="83e70-103">Insurance claim image classification on Azure</span></span>

<span data-ttu-id="83e70-104">Este escenario de ejemplo es aplicable a empresas que necesitan procesar imágenes.</span><span class="sxs-lookup"><span data-stu-id="83e70-104">This example scenario is applicable for businesses that need to process images.</span></span>

<span data-ttu-id="83e70-105">Algunas aplicaciones posibles son clasificar las imágenes de un sitio web de moda, analizar texto e imágenes para reclamaciones de seguros o reconocer los datos de telemetría de capturas de pantalla de juegos.</span><span class="sxs-lookup"><span data-stu-id="83e70-105">Potential applications include classifying images for a fashion website, analyzing text and images for insurance claims, or understanding telemetry data from game screenshots.</span></span> <span data-ttu-id="83e70-106">Tradicionalmente, las empresas necesitaban convertirse en expertos en modelos de aprendizaje automático, entrenar los modelos y, por último, ejecutar las imágenes en su proceso personalizado para extraer los datos de las imágenes.</span><span class="sxs-lookup"><span data-stu-id="83e70-106">Traditionally, companies would need to develop expertise in machine learning models, train the models, and finally run the images through their custom process to get the data out of the images.</span></span>

<span data-ttu-id="83e70-107">Con los servicios de Azure tales como Computer Vision API y Azure Functions, las empresas pueden eliminar la necesidad de administrar servidores individuales, al tiempo que reducen los costos y aprovechan los conocimientos que Microsoft ya ha desarrollado alrededor del procesamiento de imágenes con Cognitive Services.</span><span class="sxs-lookup"><span data-stu-id="83e70-107">By using Azure services such as the Computer Vision API and Azure Functions, companies can eliminate the need to manage individual servers, while reducing costs and leveraging the expertise that Microsoft has already developed around processing images with Cognitive services.</span></span> <span data-ttu-id="83e70-108">En concreto, este es un escenario de procesamiento de imágenes.</span><span class="sxs-lookup"><span data-stu-id="83e70-108">This scenario specifically addresses an image processing scenario.</span></span> <span data-ttu-id="83e70-109">Si tiene distintas necesidades de inteligencia artificial, tenga en cuenta el conjunto completo de [Cognitive Services][cognitive-docs].</span><span class="sxs-lookup"><span data-stu-id="83e70-109">If you have different AI needs, consider the full suite of [Cognitive Services][cognitive-docs].</span></span>

## <a name="potential-use-cases"></a><span data-ttu-id="83e70-110">Posibles casos de uso</span><span class="sxs-lookup"><span data-stu-id="83e70-110">Potential use cases</span></span>

<span data-ttu-id="83e70-111">Tenga en cuenta esta solución para los casos de uso siguientes:</span><span class="sxs-lookup"><span data-stu-id="83e70-111">Consider this solution for the following use cases:</span></span>

* <span data-ttu-id="83e70-112">Clasificar imágenes en un sitio web de moda.</span><span class="sxs-lookup"><span data-stu-id="83e70-112">Classify images on a fashion website.</span></span>
* <span data-ttu-id="83e70-113">Clasificar imágenes para las reclamaciones de seguros</span><span class="sxs-lookup"><span data-stu-id="83e70-113">Classify images for insurance claims</span></span>
* <span data-ttu-id="83e70-114">Clasificar los datos de telemetría de las capturas de pantalla de juegos.</span><span class="sxs-lookup"><span data-stu-id="83e70-114">Classify telemetry data from screenshots of games.</span></span>

## <a name="architecture"></a><span data-ttu-id="83e70-115">Arquitectura</span><span class="sxs-lookup"><span data-stu-id="83e70-115">Architecture</span></span>

![Arquitectura de aplicaciones inteligente: Computer Vision][architecture-computer-vision]

<span data-ttu-id="83e70-117">Esta solución trata los componentes de back-end de una aplicación web o móvil.</span><span class="sxs-lookup"><span data-stu-id="83e70-117">This solution covers the back-end components of a web or mobile application.</span></span> <span data-ttu-id="83e70-118">Los datos fluyen por la solución de la siguiente manera:</span><span class="sxs-lookup"><span data-stu-id="83e70-118">Data flows through the solution as follows:</span></span>

1. <span data-ttu-id="83e70-119">Azure Functions actúa como capa de API.</span><span class="sxs-lookup"><span data-stu-id="83e70-119">Azure Functions acts as the API layer.</span></span> <span data-ttu-id="83e70-120">Estas API permiten que la aplicación carguen imágenes y recuperen datos de Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="83e70-120">These APIs enable the application to upload images and retrieve data from Cosmos DB.</span></span>

2. <span data-ttu-id="83e70-121">Cuando se carga una imagen mediante una llamada API, se almacena en Blob Storage.</span><span class="sxs-lookup"><span data-stu-id="83e70-121">When an image is uploaded via an API call, it's stored in Blob storage.</span></span>

3. <span data-ttu-id="83e70-122">Al agregar nuevos archivos a Blob Storage, se desencadena el envío de una notificación EventGrid a Azure Functions.</span><span class="sxs-lookup"><span data-stu-id="83e70-122">Adding new files to Blob storage triggers an EventGrid notification to be sent to an Azure Function.</span></span>

4. <span data-ttu-id="83e70-123">Azure Functions envía un vínculo al nuevo archivo cargado a Computer Vision API para su análisis.</span><span class="sxs-lookup"><span data-stu-id="83e70-123">Azure Functions sends a link to the newly uploaded file to the Computer Vision API to analyze.</span></span>

5. <span data-ttu-id="83e70-124">Una vez que hayan obtenido los datos de Computer Vision API, Azure Functions crea una entrada en Cosmos DB para conservar los resultados del análisis junto con los metadatos de la imagen.</span><span class="sxs-lookup"><span data-stu-id="83e70-124">Once the data has been returned from the Computer Vision API, Azure Functions makes an entry in Cosmos DB to persist the results of the analysis alongside the image metadata.</span></span>

### <a name="components"></a><span data-ttu-id="83e70-125">Componentes</span><span class="sxs-lookup"><span data-stu-id="83e70-125">Components</span></span>

* <span data-ttu-id="83e70-126">[Computer Vision API][computer-vision-docs] es parte del conjunto de productos Cognitive Services y se usa para recuperar información acerca de cada imagen.</span><span class="sxs-lookup"><span data-stu-id="83e70-126">[Computer Vision API][computer-vision-docs] is part of the Cognitive Services suite and is used to retrieve information about each image.</span></span>

* <span data-ttu-id="83e70-127">[Azure Functions][functions-docs] proporciona la API de back-end para la aplicación web, así como el procesamiento de eventos de las imágenes cargadas.</span><span class="sxs-lookup"><span data-stu-id="83e70-127">[Azure Functions][functions-docs] provides the backend API for the web application, as well as the event processing for uploaded images.</span></span>

* <span data-ttu-id="83e70-128">[Event Grid][eventgrid-docs] desencadena un evento cuando se carga una nueva imagen en Blob Storage.</span><span class="sxs-lookup"><span data-stu-id="83e70-128">[Event Grid][eventgrid-docs] triggers an event when a new image is uploaded to blob storage.</span></span> <span data-ttu-id="83e70-129">A continuación, la imagen se procesa con Azure Functions.</span><span class="sxs-lookup"><span data-stu-id="83e70-129">The image is then processed with Azure functions.</span></span>

* <span data-ttu-id="83e70-130">[Blob Storage][storage-docs] almacena todos los archivos de imagen que se cargan en la aplicación web, como también los archivos estáticos que consume la aplicación web.</span><span class="sxs-lookup"><span data-stu-id="83e70-130">[Blob Storage][storage-docs] stores all of the image files that are uploaded into the web application, as well any static files that the web application consumes.</span></span>

* <span data-ttu-id="83e70-131">[Cosmos DB][cosmos-docs] almacena los metadatos de cada imagen que se haya cargado, incluidos los resultados del procesamiento de Computer Vision API.</span><span class="sxs-lookup"><span data-stu-id="83e70-131">[Cosmos DB][cosmos-docs] stores metadata about each image that is uploaded, including the results of the processing from Computer Vision API.</span></span>

## <a name="alternatives"></a><span data-ttu-id="83e70-132">Alternativas</span><span class="sxs-lookup"><span data-stu-id="83e70-132">Alternatives</span></span>

* <span data-ttu-id="83e70-133">[Custom Vision Service][custom-vision-docs].</span><span class="sxs-lookup"><span data-stu-id="83e70-133">[Custom Vision Service][custom-vision-docs].</span></span> <span data-ttu-id="83e70-134">Computer Vision API devuelve un conjunto de [categorías basadas en la taxonomía][cv-categories].</span><span class="sxs-lookup"><span data-stu-id="83e70-134">The Computer Vision API returns a set of [taxonomy-based categories][cv-categories].</span></span> <span data-ttu-id="83e70-135">Si necesita procesar la información que Computer Vision API no devuelve, considere la posibilidad de usar Custom Vision Service, que permite crear clasificadores de imágenes personalizados.</span><span class="sxs-lookup"><span data-stu-id="83e70-135">If you need to process information that isn't returned by the Computer Vision API, consider the Custom Vision Service, which lets you build custom image classifiers.</span></span>

* <span data-ttu-id="83e70-136">[Azure Search][azure-search-docs].</span><span class="sxs-lookup"><span data-stu-id="83e70-136">[Azure Search][azure-search-docs].</span></span> <span data-ttu-id="83e70-137">Si su caso de usuario implica consultar los metadatos para buscar las imágenes que cumplen determinados criterios, considere la posibilidad de usar Azure Search.</span><span class="sxs-lookup"><span data-stu-id="83e70-137">If your use case involves querying the metadata to find images that meet specific criteria, consider using Azure Search.</span></span> <span data-ttu-id="83e70-138">Actualmente en versión preliminar, [Cognitive Search][cognitive-search] se integra sin problemas en este flujo de trabajo.</span><span class="sxs-lookup"><span data-stu-id="83e70-138">Currently in preview, [Cognitive search][cognitive-search] seamlessly integrates this workflow.</span></span>

## <a name="considerations"></a><span data-ttu-id="83e70-139">Consideraciones</span><span class="sxs-lookup"><span data-stu-id="83e70-139">Considerations</span></span>

### <a name="scalability"></a><span data-ttu-id="83e70-140">Escalabilidad</span><span class="sxs-lookup"><span data-stu-id="83e70-140">Scalability</span></span>

<span data-ttu-id="83e70-141">La mayor parte todos los componentes de esta solución son servicios administrados que escalan automáticamente.</span><span class="sxs-lookup"><span data-stu-id="83e70-141">For the most part all of the components of this solution are managed services that will automatically scale.</span></span> <span data-ttu-id="83e70-142">Hay dos excepciones destacables: Azure Functions tiene un límite máximo de 200 instancias.</span><span class="sxs-lookup"><span data-stu-id="83e70-142">A couple notable exceptions: Azure Functions has a limit of a maximum of 200 instances.</span></span> <span data-ttu-id="83e70-143">Si necesita escalar más allá, considere la posibilidad de usar varias regiones o planes de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="83e70-143">If you need to scale beyond, consider multiple regions or app plans.</span></span>

<span data-ttu-id="83e70-144">Cosmos DB no reduce horizontalmente la escala de forma automática en términos de unidades de solicitud (RU) aprovisionadas.</span><span class="sxs-lookup"><span data-stu-id="83e70-144">Cosmos DB doesn’t auto-scale in terms of provisioned request units (RUs).</span></span>  <span data-ttu-id="83e70-145">Para obtener instrucciones sobre cómo calcular los requisitos, consulte [unidades de solicitud][request-units] en nuestra documentación.</span><span class="sxs-lookup"><span data-stu-id="83e70-145">For guidance on estimating your requirements see [request units][request-units] in our documentation.</span></span> <span data-ttu-id="83e70-146">Para aprovechar al máximo el escalado en Cosmos DB, eche un vistazo también a las [claves de partición][partition-key].</span><span class="sxs-lookup"><span data-stu-id="83e70-146">To fully take advantage of the scaling in Cosmos DB you should also take a look at [partition keys][partition-key].</span></span>

<span data-ttu-id="83e70-147">Las bases de datos NoSQL suelen renunciar a la coherencia (en el sentido del teorema CAP) frente a la disponibilidad, escalabilidad y partición.</span><span class="sxs-lookup"><span data-stu-id="83e70-147">NoSQL databases frequently trade consistency (in the sense of the CAP theorem) for availability, scalability and partition.</span></span>  <span data-ttu-id="83e70-148">Sin embargo, en el caso de los modelos de datos de pares clave-valor que se usa en este escenario, la coherencia de la transacción rara vez es necesaria porque, por definición, la mayoría de las operaciones son atómicas.</span><span class="sxs-lookup"><span data-stu-id="83e70-148">However, in the case of key-value data models which is used in this scenario, transaction consistency is rarely needed as most operations are by definition atomic.</span></span> <span data-ttu-id="83e70-149">Para más información sobre cómo [elegir el almacén de datos correcto](../../guide/technology-choices/data-store-overview.md), consulte el centro de arquitectura.</span><span class="sxs-lookup"><span data-stu-id="83e70-149">Additional guidance to [Choose the right data store](../../guide/technology-choices/data-store-overview.md) is available in the architecture center.</span></span>

<span data-ttu-id="83e70-150">Para obtener instrucciones generales sobre cómo diseñar soluciones escalables, consulte la [lista de comprobación de escalabilidad][scalability] en el centro de arquitectura de Azure.</span><span class="sxs-lookup"><span data-stu-id="83e70-150">For general guidance on designing scalable solutions, see the [scalability checklist][scalability] in the Azure Architecture Center.</span></span>

### <a name="security"></a><span data-ttu-id="83e70-151">Seguridad</span><span class="sxs-lookup"><span data-stu-id="83e70-151">Security</span></span>

<span data-ttu-id="83e70-152">[Managed Service Identity][msi] (MSI) se utiliza para proporcionar a otros recursos internos acceso a su cuenta y, a continuación, se asigna a las funciones de Azure.</span><span class="sxs-lookup"><span data-stu-id="83e70-152">[Managed service identities][msi] (MSI) are used to provide access to other resources internal to your account and then assigned to your Azure Functions.</span></span> <span data-ttu-id="83e70-153">Permita el acceso solo a los recursos necesarios en esas identidades para evitar exponer nada que no sea necesario a las funciones (y, potencialmente, a sus clientes).</span><span class="sxs-lookup"><span data-stu-id="83e70-153">Only allow access to the requisite resources in those identities to ensure that nothing extra is exposed to your functions (and potentially to your customers).</span></span>  

<span data-ttu-id="83e70-154">Para obtener instrucciones generales sobre el diseño de soluciones seguras, consulte la [documentación de seguridad de Azure][security].</span><span class="sxs-lookup"><span data-stu-id="83e70-154">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="83e70-155">Resistencia</span><span class="sxs-lookup"><span data-stu-id="83e70-155">Resiliency</span></span>

<span data-ttu-id="83e70-156">En esta solución, todos los componentes son administrados, por lo que, en un nivel regional, son resistentes de forma automática.</span><span class="sxs-lookup"><span data-stu-id="83e70-156">All of the components in this solution are managed, so at a regional level they are all resilient automatically.</span></span> 

<span data-ttu-id="83e70-157">Para obtener instrucciones generales sobre el diseño de soluciones resistentes, consulte [Diseño de aplicaciones resistentes de Azure][resiliency].</span><span class="sxs-lookup"><span data-stu-id="83e70-157">For general guidance on designing resilient solutions, see [Designing resilient applications for Azure][resiliency].</span></span>

## <a name="pricing"></a><span data-ttu-id="83e70-158">Precios</span><span class="sxs-lookup"><span data-stu-id="83e70-158">Pricing</span></span>

<span data-ttu-id="83e70-159">Para explorar el costo de ejecutar esta solución, todos los servicios están preconfigurados en la calculadora de costos.</span><span class="sxs-lookup"><span data-stu-id="83e70-159">To explore the cost of running this solution, all of the services are pre-configured in the cost calculator.</span></span> <span data-ttu-id="83e70-160">Para ver cómo cambiarían los precios en su caso concreto, cambie las variables pertinentes para que coincidan con el tráfico esperado.</span><span class="sxs-lookup"><span data-stu-id="83e70-160">To see how the pricing would change for your particular use case, change the appropriate variables to match your expected traffic.</span></span>

<span data-ttu-id="83e70-161">Hemos proporcionado tres ejemplos de perfiles de costo según la cantidad de tráfico (hemos dado por hecho que todas las imágenes tienen un tamaño de 100 Kb):</span><span class="sxs-lookup"><span data-stu-id="83e70-161">We have provided three sample cost profiles based on amount of traffic (we assume all images are 100kb in size):</span></span>

* <span data-ttu-id="83e70-162">[Pequeño][pricing]: se corresponde con un procesamiento de menos de 5 000 mensajes al mes.</span><span class="sxs-lookup"><span data-stu-id="83e70-162">[Small][pricing]: this correlates to processing &lt; 5000 images a month.</span></span>
* <span data-ttu-id="83e70-163">[Mediano][medium-pricing]: se corresponde con un procesamiento de 500 000 mensajes al mes.</span><span class="sxs-lookup"><span data-stu-id="83e70-163">[Medium][medium-pricing]: this correlates to processing 500,000 images a month.</span></span>
* <span data-ttu-id="83e70-164">[Grande][large-pricing]: se corresponde con un procesamiento de 50 millones de mensajes al mes.</span><span class="sxs-lookup"><span data-stu-id="83e70-164">[Large][large-pricing]: this correlates to processing 50 million images a month.</span></span>

## <a name="related-resources"></a><span data-ttu-id="83e70-165">Recursos relacionados</span><span class="sxs-lookup"><span data-stu-id="83e70-165">Related Resources</span></span>

<span data-ttu-id="83e70-166">Para ver una ruta de aprendizaje guiado de esta solución, consulte [Compilación de una aplicación web sin servidor en Azure][serverless].</span><span class="sxs-lookup"><span data-stu-id="83e70-166">For a guided learning path of this solution, see [Build a serverless web app in Azure][serverless].</span></span>  

<span data-ttu-id="83e70-167">Antes de poner esto en un entorno de producción, revise los [procedimientos recomendados][functions-best-practices] de Azure Functions.</span><span class="sxs-lookup"><span data-stu-id="83e70-167">Before putting this in a production environment, review the Azure Functions [best practices][functions-best-practices].</span></span>

<!-- links -->
[pricing]: https://azure.com/e/f9b59d238b43423683db73f4a31dc380
[medium-pricing]: https://azure.com/e/7c7fc474db344b87aae93bc29ae27108
[large-pricing]: https://azure.com/e/cbadbca30f8640d6a061f8457a74ba7d
[functions-docs]: /azure/azure-functions/
[computer-vision-docs]: /azure/cognitive-services/computer-vision/home
[storage-docs]: /azure/storage/
[azure-search-docs]: /azure/search/
[cognitive-search]: /azure/search/cognitive-search-concept-intro
[architecture-computer-vision]: ./media/architecture-computer-vision.png
[serverless]: /azure/functions/tutorial-static-website-serverless-api-with-database
[cosmos-docs]: /azure/cosmos-db/
[eventgrid-docs]: /azure/event-grid/
[cognitive-docs]: /azure/#pivot=products&panel=ai
[custom-vision-docs]: /azure/cognitive-services/Custom-Vision-Service/home
[cv-categories]: /azure/cognitive-services/computer-vision/home#the-86-category-concept
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[functions-best-practices]: /azure/azure-functions/functions-best-practices
[msi]: /azure/app-service/app-service-managed-service-identity
[request-units]: /azure/cosmos-db/request-units
[partition-key]: /azure/cosmos-db/partition-data