---
title: Clasificación de imágenes de reclamaciones de seguros en Azure
description: Compile el procesamiento de imágenes en sus aplicaciones de Azure.
author: david-stanford
ms.date: 07/05/2018
ms.openlocfilehash: 9640f8b5454891ed00f669bada9f7c9c69b89734
ms.sourcegitcommit: 0a31fad9b68d54e2858314ca5fe6cba6c6b95ae4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/13/2018
ms.locfileid: "51610539"
---
# <a name="image-classification-for-insurance-claims-on-azure"></a>Clasificación de imágenes de reclamaciones de seguros en Azure

Este escenario es pertinente para las empresas que necesitan procesar imágenes.

Algunas aplicaciones posibles son clasificar las imágenes de un sitio web de moda, analizar texto e imágenes para reclamaciones de seguros o reconocer los datos de telemetría de capturas de pantalla de juegos. Tradicionalmente, las empresas necesitaban convertirse en expertos en modelos de aprendizaje automático, entrenar los modelos y, por último, ejecutar las imágenes en su proceso personalizado para extraer los datos de las imágenes.

Con los servicios de Azure tales como Computer Vision API y Azure Functions, las empresas pueden eliminar la necesidad de administrar servidores individuales, al tiempo que reducen los costos y aprovechan los conocimientos que Microsoft ya ha desarrollado alrededor del procesamiento de imágenes con Cognitive Services. En concreto, este es un escenario de ejemplo de procesamiento de imágenes. Si tiene distintas necesidades de inteligencia artificial, tenga en cuenta el conjunto completo de [Cognitive Services](/azure/#pivot=products&panel=ai).

## <a name="relevant-use-cases"></a>Casos de uso pertinentes

Otros casos de uso pertinentes incluyen:

* Clasificar las imágenes en un sitio web de moda.
* Clasificar los datos de telemetría de las capturas de pantalla de juegos.

## <a name="architecture"></a>Arquitectura

![Arquitectura para la clasificación de imágenes][architecture]

Este escenario trata los componentes de back-end de una aplicación web o móvil. Los datos fluyen por el escenario de la siguiente manera:

1. El nivel de API se ha creado con Azure Functions. Estas API permiten que la aplicación carguen imágenes y recuperen datos de Cosmos DB.
2. Cuando se carga una imagen mediante una llamada API, se almacena en Blob Storage.
3. Al agregar nuevos archivos a Blob Storage, se desencadena el envío de una notificación de Event Grid a Azure Functions.
4. Azure Functions envía un vínculo al nuevo archivo cargado a Computer Vision API para su análisis.
5. Una vez que hayan obtenido los datos de Computer Vision API, Azure Functions crea una entrada en Cosmos DB para conservar los resultados del análisis junto con los metadatos de la imagen.

### <a name="components"></a>Componentes

* [Computer Vision API](/azure/cognitive-services/computer-vision/home) es parte del conjunto de productos Cognitive Services y se usa para recuperar información acerca de cada imagen.
* [Azure Functions](/azure/azure-functions/functions-overview) proporciona la API de back-end para la aplicación web, así como el procesamiento de eventos de las imágenes cargadas.
* [Event Grid](/azure/event-grid/overview) desencadena un evento cuando se carga una nueva imagen en Blob Storage. A continuación, la imagen se procesa con Azure Functions.
* [Blob Storage](/azure/storage/blobs/storage-blobs-introduction) almacena todos los archivos de imagen que se cargan en la aplicación web, así como también los archivos estáticos que consume la aplicación web.
* [Cosmos DB](/azure/cosmos-db/introduction) almacena los metadatos de cada imagen que se haya cargado, incluidos los resultados del procesamiento de Computer Vision API.

## <a name="alternatives"></a>Alternativas

* [Custom Vision Service](/azure/cognitive-services/custom-vision-service/home). Computer Vision API devuelve un conjunto de [categorías basadas en la taxonomía][cv-categories]. Si necesita procesar la información que Computer Vision API no devuelve, considere la posibilidad de usar Custom Vision Service, que permite crear clasificadores de imágenes personalizados.
* [Azure Search](/azure/search/search-what-is-azure-search). Si su caso de usuario implica consultar los metadatos para buscar las imágenes que cumplen determinados criterios, considere la posibilidad de usar Azure Search. Actualmente en versión preliminar, [Cognitive Search](/azure/search/cognitive-search-concept-intro) se integra sin problemas en este flujo de trabajo.

## <a name="considerations"></a>Consideraciones

### <a name="scalability"></a>Escalabilidad

La mayoría de los componentes usados en este escenario de ejemplo son servicios administrados que escalan automáticamente. Hay dos excepciones destacables: Azure Functions tiene un límite máximo de 200 instancias. Si necesita escalar más allá de este límite, considere la posibilidad de usar varias regiones o planes de aplicación.

Cosmos DB no realiza escalabilidad automática en términos de unidades de solicitud (RU) aprovisionadas. Para obtener instrucciones sobre cómo calcular los requisitos, consulte [unidades de solicitud](/azure/cosmos-db/request-units) en nuestra documentación. Para aprovechar al máximo el escalado en Cosmos DB, sepa cómo funcionan las [claves de partición](/azure/cosmos-db/partition-data) en CosmosDB.

Las bases de datos NoSQL suelen renunciar a la coherencia (en el sentido del teorema CAP) frente a la disponibilidad, escalabilidad y creación de particiones. En este escenario de ejemplo, se usa un modelo de datos de pares clave-valor y la coherencia de la transacción rara vez es necesaria porque, por definición, la mayoría de las operaciones son atómicas. Para más información sobre cómo [elegir el almacén de datos correcto](../../guide/technology-choices/data-store-overview.md), consulte el Centro de arquitectura de Azure. Si su implementación requiere una coherencia alta, puede [elegir su nivel de coherencia](/azure/cosmos-db/consistency-levels) en CosmosDB.

Para obtener instrucciones generales sobre cómo diseñar soluciones escalables, consulte la [lista de comprobación de escalabilidad][scalability] en el centro de arquitectura de Azure.

### <a name="security"></a>Seguridad

[Managed Service Identities para recursos de Azure][msi] (MSI) se utiliza para proporcionar a otros recursos internos acceso a su cuenta y, a continuación, se asigna a Azure Functions. Permita el acceso solo a los recursos necesarios en esas identidades para evitar exponer nada que no sea necesario a las funciones (y, potencialmente, a sus clientes).

Para obtener instrucciones generales sobre el diseño de soluciones seguras, consulte la [documentación de seguridad de Azure][security].

### <a name="resiliency"></a>Resistencia

En este escenario, todos los componentes son administrados, por lo que, en un nivel regional, son resistentes de forma automática.

Para obtener instrucciones generales sobre el diseño de soluciones resistentes, consulte [Diseño de aplicaciones resistentes de Azure][resiliency].

## <a name="pricing"></a>Precios

Para explorar el costo de ejecutar este escenario, todos los servicios están preconfigurados en la calculadora de costos. Para ver cómo cambiarían los precios en su caso concreto, cambie las variables pertinentes para que coincidan con el tráfico esperado.

Hemos proporcionado tres ejemplos de perfiles de costo según la cantidad de tráfico (hemos dado por hecho que todas las imágenes tienen un tamaño de 100 KB):

* [Pequeño][small-pricing]: se corresponde con un procesamiento de menos de 5000 imágenes al mes.
* [Mediano][medium-pricing]: se corresponde con un procesamiento de menos de 500 000 imágenes al mes.
* [Grande][large-pricing]: se corresponde con un procesamiento de menos de 50 millones de imágenes al mes.

## <a name="related-resources"></a>Recursos relacionados

Para ver una ruta de aprendizaje guiado, consulte [Compilación de una aplicación web sin servidor en Azure][serverless].

Antes de implementar este escenario de ejemplo en un entorno de producción, consulte las prácticas recomendadas para [optimizar el rendimiento y confiabilidad de Azure Functions][functions-best-practices].

<!-- links -->
[architecture]: ./media/architecture-intelligent-apps-image-processing.png
[small-pricing]: https://azure.com/e/f9b59d238b43423683db73f4a31dc380
[medium-pricing]: https://azure.com/e/7c7fc474db344b87aae93bc29ae27108
[large-pricing]: https://azure.com/e/cbadbca30f8640d6a061f8457a74ba7d
[cognitive-search]: /azure/search/cognitive-search-concept-intro
[serverless]: /azure/functions/tutorial-static-website-serverless-api-with-database
[cv-categories]: /azure/cognitive-services/computer-vision/home#the-86-category-concept
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[functions-best-practices]: /azure/azure-functions/functions-best-practices
[msi]: /azure/app-service/app-service-managed-service-identity
