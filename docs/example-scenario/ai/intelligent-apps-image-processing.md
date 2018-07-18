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
# <a name="insurance-claim-image-classification-on-azure"></a>Clasificación de imágenes de reclamaciones de seguros en Azure

Este escenario de ejemplo es aplicable a empresas que necesitan procesar imágenes.

Algunas aplicaciones posibles son clasificar las imágenes de un sitio web de moda, analizar texto e imágenes para reclamaciones de seguros o reconocer los datos de telemetría de capturas de pantalla de juegos. Tradicionalmente, las empresas necesitaban convertirse en expertos en modelos de aprendizaje automático, entrenar los modelos y, por último, ejecutar las imágenes en su proceso personalizado para extraer los datos de las imágenes.

Con los servicios de Azure tales como Computer Vision API y Azure Functions, las empresas pueden eliminar la necesidad de administrar servidores individuales, al tiempo que reducen los costos y aprovechan los conocimientos que Microsoft ya ha desarrollado alrededor del procesamiento de imágenes con Cognitive Services. En concreto, este es un escenario de procesamiento de imágenes. Si tiene distintas necesidades de inteligencia artificial, tenga en cuenta el conjunto completo de [Cognitive Services][cognitive-docs].

## <a name="potential-use-cases"></a>Posibles casos de uso

Tenga en cuenta esta solución para los casos de uso siguientes:

* Clasificar imágenes en un sitio web de moda.
* Clasificar imágenes para las reclamaciones de seguros
* Clasificar los datos de telemetría de las capturas de pantalla de juegos.

## <a name="architecture"></a>Arquitectura

![Arquitectura de aplicaciones inteligente: Computer Vision][architecture-computer-vision]

Esta solución trata los componentes de back-end de una aplicación web o móvil. Los datos fluyen por la solución de la siguiente manera:

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

La mayor parte todos los componentes de esta solución son servicios administrados que escalan automáticamente. Hay dos excepciones destacables: Azure Functions tiene un límite máximo de 200 instancias. Si necesita escalar más allá, considere la posibilidad de usar varias regiones o planes de la aplicación.

Cosmos DB no reduce horizontalmente la escala de forma automática en términos de unidades de solicitud (RU) aprovisionadas.  Para obtener instrucciones sobre cómo calcular los requisitos, consulte [unidades de solicitud][request-units] en nuestra documentación. Para aprovechar al máximo el escalado en Cosmos DB, eche un vistazo también a las [claves de partición][partition-key].

Las bases de datos NoSQL suelen renunciar a la coherencia (en el sentido del teorema CAP) frente a la disponibilidad, escalabilidad y partición.  Sin embargo, en el caso de los modelos de datos de pares clave-valor que se usa en este escenario, la coherencia de la transacción rara vez es necesaria porque, por definición, la mayoría de las operaciones son atómicas. Para más información sobre cómo [elegir el almacén de datos correcto](../../guide/technology-choices/data-store-overview.md), consulte el centro de arquitectura.

Para obtener instrucciones generales sobre cómo diseñar soluciones escalables, consulte la [lista de comprobación de escalabilidad][scalability] en el centro de arquitectura de Azure.

### <a name="security"></a>Seguridad

[Managed Service Identity][msi] (MSI) se utiliza para proporcionar a otros recursos internos acceso a su cuenta y, a continuación, se asigna a las funciones de Azure. Permita el acceso solo a los recursos necesarios en esas identidades para evitar exponer nada que no sea necesario a las funciones (y, potencialmente, a sus clientes).  

Para obtener instrucciones generales sobre el diseño de soluciones seguras, consulte la [documentación de seguridad de Azure][security].

### <a name="resiliency"></a>Resistencia

En esta solución, todos los componentes son administrados, por lo que, en un nivel regional, son resistentes de forma automática. 

Para obtener instrucciones generales sobre el diseño de soluciones resistentes, consulte [Diseño de aplicaciones resistentes de Azure][resiliency].

## <a name="pricing"></a>Precios

Para explorar el costo de ejecutar esta solución, todos los servicios están preconfigurados en la calculadora de costos. Para ver cómo cambiarían los precios en su caso concreto, cambie las variables pertinentes para que coincidan con el tráfico esperado.

Hemos proporcionado tres ejemplos de perfiles de costo según la cantidad de tráfico (hemos dado por hecho que todas las imágenes tienen un tamaño de 100 Kb):

* [Pequeño][pricing]: se corresponde con un procesamiento de menos de 5 000 mensajes al mes.
* [Mediano][medium-pricing]: se corresponde con un procesamiento de 500 000 mensajes al mes.
* [Grande][large-pricing]: se corresponde con un procesamiento de 50 millones de mensajes al mes.

## <a name="related-resources"></a>Recursos relacionados

Para ver una ruta de aprendizaje guiado de esta solución, consulte [Compilación de una aplicación web sin servidor en Azure][serverless].  

Antes de poner esto en un entorno de producción, revise los [procedimientos recomendados][functions-best-practices] de Azure Functions.

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