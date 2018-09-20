---
title: Clasificación de imágenes de reclamaciones de seguros en Azure
description: Escenario probado para incorporar procesamiento de imágenes en sus aplicaciones de Azure.
author: david-stanford
ms.date: 07/05/2018
ms.openlocfilehash: 0ca0b46e83219afc5e22c2ac6467bf4be945c97a
ms.sourcegitcommit: c49aeef818d7dfe271bc4128b230cfc676f05230
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/11/2018
ms.locfileid: "44389169"
---
# <a name="image-classification-for-insurance-claims-on-azure"></a>Clasificación de imágenes de reclamaciones de seguros en Azure

Este escenario de ejemplo es aplicable a empresas que necesitan procesar imágenes.

Algunas aplicaciones posibles son clasificar las imágenes de un sitio web de moda, analizar texto e imágenes para reclamaciones de seguros o reconocer los datos de telemetría de capturas de pantalla de juegos. Tradicionalmente, las empresas necesitaban convertirse en expertos en modelos de aprendizaje automático, entrenar los modelos y, por último, ejecutar las imágenes en su proceso personalizado para extraer los datos de las imágenes.

Con los servicios de Azure tales como Computer Vision API y Azure Functions, las empresas pueden eliminar la necesidad de administrar servidores individuales, al tiempo que reducen los costos y aprovechan los conocimientos que Microsoft ya ha desarrollado alrededor del procesamiento de imágenes con Cognitive Services. En concreto, este es un escenario de ejemplo de procesamiento de imágenes. Si tiene distintas necesidades de inteligencia artificial, tenga en cuenta el conjunto completo de [Cognitive Services][cognitive-docs].

## <a name="related-use-cases"></a>Casos de uso relacionados

Tenga en cuenta este escenario para los casos de uso siguientes:

* Clasificar imágenes en un sitio web de moda.
* Clasificar los datos de telemetría de las capturas de pantalla de juegos.

## <a name="architecture"></a>Arquitectura

![Arquitectura de aplicaciones inteligente: Computer Vision][architecture-computer-vision]

Este escenario trata los componentes de back-end de una aplicación web o móvil. Los datos fluyen por el escenario de la siguiente manera:

1. Azure Functions actúa como capa de API. Estas API permiten que la aplicación carguen imágenes y recuperen datos de Cosmos DB.

2. Cuando se carga una imagen mediante una llamada API, se almacena en Blob Storage.

3. Al agregar nuevos archivos a Blob Storage, se desencadena el envío de una notificación EventGrid a Azure Functions.

4. Azure Functions envía un vínculo al nuevo archivo cargado a Computer Vision API para su análisis.

5. Una vez que hayan obtenido los datos de Computer Vision API, Azure Functions crea una entrada en Cosmos DB para conservar los resultados del análisis junto con los metadatos de la imagen.

### <a name="components"></a>Componentes

* [Computer Vision API][computer-vision-docs] es parte del conjunto de productos Cognitive Services y se usa para recuperar información acerca de cada imagen.

* [Azure Functions][functions-docs] proporciona la API de back-end para la aplicación web, así como el procesamiento de eventos de las imágenes cargadas.

* [Event Grid][eventgrid-docs] desencadena un evento cuando se carga una nueva imagen en Blob Storage. A continuación, la imagen se procesa con Azure Functions.

* [Blob Storage][storage-docs] almacena todos los archivos de imagen que se cargan en la aplicación web, como también los archivos estáticos que consume la aplicación web.

* [Cosmos DB][cosmos-docs] almacena los metadatos de cada imagen que se haya cargado, incluidos los resultados del procesamiento de Computer Vision API.

## <a name="alternatives"></a>Alternativas

* [Custom Vision Service][custom-vision-docs]. Computer Vision API devuelve un conjunto de [categorías basadas en la taxonomía][cv-categories]. Si necesita procesar la información que Computer Vision API no devuelve, considere la posibilidad de usar Custom Vision Service, que permite crear clasificadores de imágenes personalizados.

* [Azure Search][azure-search-docs]. Si su caso de usuario implica consultar los metadatos para buscar las imágenes que cumplen determinados criterios, considere la posibilidad de usar Azure Search. Actualmente en versión preliminar, [Cognitive Search][cognitive-search] se integra sin problemas en este flujo de trabajo.

## <a name="considerations"></a>Consideraciones

### <a name="scalability"></a>Escalabilidad

La mayoría de los componentes usados en este escenario de ejemplo son servicios administrados que escalan automáticamente. Hay dos excepciones destacables: Azure Functions tiene un límite máximo de 200 instancias. Si necesita escalar más allá de este límite, considere la posibilidad de usar varias regiones o planes de aplicación.

Cosmos DB no reduce horizontalmente la escala de forma automática en términos de unidades de solicitud (RU) aprovisionadas.  Para obtener instrucciones sobre cómo calcular los requisitos, consulte [unidades de solicitud][request-units] en nuestra documentación. Para aprovechar al máximo el escalado en Cosmos DB, eche un vistazo a las [claves de partición][partition-key].

Las bases de datos NoSQL suelen renunciar a la coherencia (en el sentido del teorema CAP) frente a la disponibilidad, escalabilidad y creación de particiones.  En este escenario de ejemplo, se usa un modelo de datos de pares clave-valor y la coherencia de la transacción rara vez es necesaria porque, por definición, la mayoría de las operaciones son atómicas. Para más información sobre cómo [elegir el almacén de datos correcto](../../guide/technology-choices/data-store-overview.md), consulte el Centro de arquitectura de Azure.

Para obtener instrucciones generales sobre cómo diseñar soluciones escalables, consulte la [lista de comprobación de escalabilidad][scalability] en el centro de arquitectura de Azure.

### <a name="security"></a>Seguridad

[Managed Service Identity][msi] (MSI) se utiliza para proporcionar a otros recursos internos acceso a su cuenta y, a continuación, se asigna a las funciones de Azure. Permita el acceso solo a los recursos necesarios en esas identidades para evitar exponer nada que no sea necesario a las funciones (y, potencialmente, a sus clientes).  

Para obtener instrucciones generales sobre el diseño de soluciones seguras, consulte la [documentación de seguridad de Azure][security].

### <a name="resiliency"></a>Resistencia

En este escenario, todos los componentes son administrados, por lo que, en un nivel regional, son resistentes de forma automática.

Para obtener instrucciones generales sobre el diseño de soluciones resistentes, consulte [Diseño de aplicaciones resistentes de Azure][resiliency].

## <a name="pricing"></a>Precios

Para explorar el costo de ejecutar este escenario, todos los servicios están preconfigurados en la calculadora de costos. Para ver cómo cambiarían los precios en su caso concreto, cambie las variables pertinentes para que coincidan con el tráfico esperado.

Hemos proporcionado tres ejemplos de perfiles de costo según la cantidad de tráfico (hemos dado por hecho que todas las imágenes tienen un tamaño de 100 KB):

* [Pequeño][pricing]: se corresponde con un procesamiento de menos de 5000 imágenes al mes.
* [Mediano][medium-pricing]: se corresponde con un procesamiento de menos de 500 000 imágenes al mes.
* [Grande][large-pricing]: se corresponde con un procesamiento de menos de 50 millones de imágenes al mes.

## <a name="related-resources"></a>Recursos relacionados

Para ver una ruta de aprendizaje guiado de este escenario, consulte [Compilación de una aplicación web sin servidor en Azure][serverless].  

Antes de implementar este escenario de ejemplo en un entorno de producción, revise los [procedimientos recomendados][functions-best-practices] de Azure Functions.

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