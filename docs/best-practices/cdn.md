---
title: Orientación sobre Content Delivery Network
description: Orientación sobre Content Delivery Network (CDN) para entregar contenido de alto ancho de banda hospedado en Azure.
author: dragon119
ms.date: 02/02/2018
pnp.series.title: Best Practices
ms.openlocfilehash: 9805b1b6df8cedd7668eb9e85f741ee81c3dfa58
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/28/2018
ms.locfileid: "47428897"
---
# <a name="best-practices-for-using-content-delivery-networks-cdns"></a>Procedimientos recomendados para el uso de redes de entrega de contenido (CDN)

Una red de entrega de contenido (CDN) es una red distribuida de servidores que puede proporcionar contenido web a los usuarios de manera eficaz. Las redes CDN guardan contenido almacenado en caché en servidores perimetrales que están cerca de los usuarios finales, para minimizar la latencia. 

Las redes CDN se suelen usar para proporcionar contenido estático, como imágenes, hojas de estilo, documentos, scripts del lado cliente y páginas HTML. Las principales ventajas de usar una red CDN son una menor latencia y una entrega de contenido más rápida a los usuarios, independientemente de su ubicación geográfica en relación con el centro de datos en que se hospeda la aplicación. Las redes CDN también pueden ayudar a reducir la carga de una aplicación web, ya que la aplicación no tiene que atender las solicitudes para el contenido que se hospeda en la red CDN.
 
![Diagrama de la red CDN](./images/cdn/CDN.png)

En Azure, [Azure Content Delivery Network](/azure/cdn/cdn-overview) es una solución de CDN global que proporciona contenido con un ancho de banda alto hospedado en Azure o en cualquier otra ubicación. Mediante Azure CDN puede almacenar en caché objetos disponibles públicamente cargados desde Azure Blob Storage, una aplicación web, una máquina virtual o cualquier servidor web accesible de forma pública. 

En este tema se describen algunos procedimientos recomendados y consideraciones generales cuando se usa una red CDN. Para más información acerca del uso de Azure CDN, consulte [Documentación de CDN](/azure/cdn/).

## <a name="how-and-why-a-cdn-is-used"></a>Cómo y por qué se usa una red CDN

Entre los usos típicos de la red CDN se incluyen:  

* La entrega de recursos estáticos para aplicaciones cliente, a menudo desde un sitio web. Dichos recursos pueden ser imágenes, hojas de estilo, documentos, archivos, scripts de cliente, páginas HTML, fragmentos de HTML o cualquier otro contenido que el servidor no necesite modificar para cada solicitud. La aplicación puede crear elementos en tiempo de ejecución y ponerlos a disposición de la red CDN (por ejemplo, mediante la creación de una lista de los titulares más recientes), pero no lo hace para cada solicitud.
* Entrega de contenido compartido y estático público a dispositivos como teléfonos móviles y Tablet PC. La propia aplicación es un servicio web que ofrece una API a los clientes que se ejecutan en los distintos dispositivos. La red CDN también puede ofrecer conjuntos de datos estáticos (mediante el servicio web) para que los clientes los usen, quizás para generar la interfaz de usuario del cliente. Por ejemplo, la red CDN se puede usar para distribuir documentos JSON o XML.
* La prestación de servicio de sitios web completos que constan únicamente de contenido público estático que se entrega a los clientes, sin necesidad de disponer de recursos de proceso dedicados.
* El streaming de archivos de vídeo en el cliente a petición. El vídeo aprovecha la latencia baja y una conectividad confiable disponible en los centros de datos distribuidos en todo el mundo que ofrecen conexiones de red CDN. Microsoft Azure Media Services (AMS) se integra con Azure CDN para entregar contenido directamente a la red CDN para su distribución posterior. Para obtener más información, consulte [Streaming endpoints overview](/azure/media-services/media-services-streaming-endpoints-overview) (Información general de puntos de conexión de streaming).
* Por lo general, esto mejora la experiencia de los usuarios, sobre todo la de los que están lejos del centro de datos que hospeda la aplicación. De lo contrario, dichos usuarios podrían sufrir una mayor latencia. Una gran parte del tamaño total del contenido de una aplicación web suele ser estática y el uso de la red CDN puede ayudar a mantener el rendimiento y la experiencia general del usuario al tiempo que elimina la necesidad de implementar la aplicación en varios centros de datos. Para una lista de ubicaciones de nodos de Azure CDN, consulte [Ubicaciones POP de Azure CDN](/azure/cdn/cdn-pop-locations/).
* Compatibilidad de soluciones de IoT (Internet de las cosas). La enorme cantidad de dispositivos y aparatos que participan en una solución de IoT podría saturar fácilmente una aplicación si tuviera que distribuir actualizaciones de firmware directamente a cada dispositivo.
* La capacidad de hacer frente a los picos y aumentos repentinos en la demanda sin que se sea preciso realizar ningún tipo de escalado, lo que evitar el consiguiente aumento de los gastos de mantenimiento. Por ejemplo, cuando se publique una actualización del sistema operativo de un dispositivo de hardware (como un modelo específico de enrutador) o de un dispositivo de consumo (por ejemplo, un televisor inteligente), se producirá un importante pico en la demanda, ya que la descargarán millones de usuarios en millones de dispositivos en un breve período de tiempo.

## <a name="challenges"></a>Desafíos

Hay varios desafíos que se deben tener en cuenta a la hora de usar una red CDN.  

* **Implementación**. Decida el origen del que la red CDN obtendrá el contenido y si es preciso implementar el contenido en más de un sistema de almacenamiento. Tenga en cuenta el proceso de implementación de contenido y de recursos estáticos. Por ejemplo, puede que necesite implementar un paso independiente para cargar el contenido en Almacenamiento de blobs de Azure.
* **Control de versiones y control de caché**. Debe tener en cuenta cómo va a actualizar el contenido estático y a implementar las nuevas versiones. Entienda cómo ejecuta la red CDN el almacenamiento en caché y el período de vida (TTL). Para Azure CDN, consulte [Cómo funciona el almacenamiento en caché](/azure/cdn/cdn-how-caching-works).
* **Prueba**. Puede resultar difícil realizar pruebas locales de la configuración de la red CDN al desarrollar y probar una aplicación localmente o en un entorno de ensayo.
* **Optimización del motor de búsqueda (SEO)**. Cierto contenido, como imágenes y documentos, se sirve desde un dominio diferente cuando se usa la red CDN. Esto puede tener un efecto en la SEO de dicho contenido.
* **Seguridad de contenido**. No todas las redes CDN ofrecen formas de control de acceso para el contenido. Algunos servicios de redes CDN, incluida Azure CDN, admiten la autenticación basada en token para proteger el contenido de las redes CDN. Para más información, consulte [Protección de recursos de Azure Content Delivery Network con la autenticación por tokens](/azure/cdn/cdn-token-auth).
* **Seguridad de los clientes**. Los clientes se pueden conectar desde un entorno que no permita el acceso a los recursos de la red CDN. Podría tratarse de un entorno con restricciones de seguridad que limita el acceso a un conjunto de orígenes conocidos o uno que impide la carga de recursos desde cualquier otra cosa que no sea el origen de la página. Para tratar estos casos se requiere una implementación de reserva.
* **Resistencia**. La red CDN es un potencial punto único de error de una aplicación. 

Escenarios donde puede resultar útil incluir la red CDN:  

* Si el contenido tiene una tasa de aciertos baja, solo se puede acceder a él pocas veces mientras sea válido (esto lo determina su período de vida). 
* Si los datos son privados, como por ejemplo en grandes empresas o en ecosistemas de la cadena de suministros.

## <a name="general-guidelines-and-good-practices"></a>Directrices generales y prácticas recomendadas

El uso de una red CDN es una buena manera de minimizar la carga en la aplicación, y de maximizar la disponibilidad y el rendimiento. Considere la posibilidad de adoptar esta estrategia para todo el contenido adecuado y los recursos que usa la aplicación. Al diseñar la estrategia de uso de una red CDN, tenga en cuenta los puntos de las siguientes secciones.

### <a name="deployment"></a>Implementación
Es posible que sea preciso que el contenido estático se aprovisione e implemente de forma independiente de la aplicación si no se incluye en el paquete de implementación o en el proceso de la aplicación. Tenga en cuenta cómo esto afectará al método de control de versiones que se use para administrar tanto los componentes de aplicaciones como el contenido de recursos estáticos.

Tenga en cuenta la posibilidad de usar técnicas de unión y minificación para reducir los tiempos de carga para los clientes. La unión combina varios archivos en un único archivo. La minificación quita caracteres innecesarios de los scripts y archivos CSS sin modificar la funcionalidad.

Si necesita implementar el contenido en una ubicación adicional, será un paso adicional en el proceso de implementación. Si la aplicación actualiza el contenido para la red CDN, quizás a intervalos regulares o en respuesta a un evento, debe almacenar el contenido actualizado en las ubicaciones adicionales, así como el extremo de la red CDN.

Tenga en cuenta cómo va a controlar el desarrollo local y las pruebas cuando se espera que se va a proporcionar contenido estático desde una red CDN. Por ejemplo, puede implementar previamente el contenido en la red CDN como parte del script de compilación. También puede usar directivas de compilación o marcas para controlar cómo carga la aplicación los recursos. Por ejemplo, en modo de depuración, la aplicación puede cargar recursos estáticos desde una carpeta local. En modo de versión, la aplicación utilizaría la red CDN.

Tenga en cuenta las opciones para la compresión de archivos, como gzip (GNU zip). El hospedaje de aplicaciones web puede realizar la compresión en el servidor de origen o la red CDN la puede realizar directamente en los servidores perimetrales. Para más información, consulte [Mejora del rendimiento comprimiendo archivos en Azure CDN](/azure/cdn/cdn-improve-performance).


### <a name="routing-and-versioning"></a>Enrutamiento y control de versiones
Es posible que necesite distintas instancias de la red CDN en momentos diferentes. Por ejemplo, al implementar una versión nueva de la aplicación puede usar una red CDN nueva y conservar la red CDN anterior (que tiene el contenido en un formato anterior) para las versiones anteriores. Si usa Azure Blob Storage como origen del contenido, puede crear una cuenta de almacenamiento independiente o un contenedor independiente y elegir el punto de conexión de la red CDN. 

No use la cadena de consulta para indicar versiones diferentes de la aplicación en los vínculos a los recursos de la red CDN porque, al recuperar el contenido de Almacenamiento de blobs de Azure, la cadena de consulta forma parte del nombre del recurso (el nombre del blob). Este enfoque también puede afectar a la manera en que el cliente almacena los recursos en la caché.

La implementación de nuevas versiones de contenido estático cuando se actualiza una aplicación puede ser un desafío si los recursos anteriores se almacenan en caché en la red CDN. Para más información, consulte la sección sobre control de caché, a continuación.

Considere la posibilidad de restringir el acceso al contenido de la red CDN por país. Azure CDN le permite filtrar las solicitudes en función del país de origen y restringir el contenido que se entrega. Para más información, consulte [Restricción del acceso a contenidos por país](/azure/cdn/cdn-restrict-access-by-country/).

### <a name="cache-control"></a>Control de caché
Considere cómo administrar el almacenamiento en la caché en el sistema. Por ejemplo, en Azure CDN, puede establecer reglas de almacenamiento en caché globales y, luego, establecer un almacenamiento en caché personalizado para determinados puntos de conexión de origen. Puede controlar igualmente cómo se realiza el almacenamiento en caché en una red CDN mediante el envío de encabezados de directiva de caché en el origen. 

Para más información, consulte [Cómo funciona el almacenamiento en caché](/azure/cdn/cdn-how-caching-works).

Para impedir que los objetos estén disponibles en la red CDN, puede eliminarlos del origen, quitar o eliminar el punto de conexión de la red CDN, o, en el caso de Blob Storage, hacer que el contenedor o el blob sean privados. Sin embargo, los elementos no se quitan hasta que expire el período de vida. También puede purgar un punto de conexión de CDN de forma manual.

### <a name="security"></a>Seguridad

La red CDN puede entregar contenido a través de HTTPS (SSL) con el certificado que proporciona la red CDN, así como a través de HTTP estándar. Es posible que tenga que usar HTTPS para solicitar el contenido estático que se muestra en las páginas que se cargan a través de HTTPS para evitar las advertencias del explorador sobre contenido mixto.

Si entrega recursos estáticos, tales como archivos de fuentes, mediante la red CDN, pueden surgir problemas de directiva del mismo origen si usa una llamada *XMLHttpRequest* para solicitar estos recursos a un dominio distinto. Muchos exploradores web impiden el uso compartido de recursos entre orígenes (CORS), a menos que el servidor web se haya configurado para establecer los encabezados de respuesta adecuados. Para admitir CORS, puede configurar la red CDN con uno de los métodos siguientes:

* Configure la red CDN para agregar encabezados de CORS a las respuestas. Para más información, consulte [Uso de Azure CDN con CORS](/azure/cdn/cdn-cors). 
* Si el origen es Azure Blob Storage, agregue reglas de CORS al punto de conexión de almacenamiento. Para más información, consulte [Compatibilidad con Uso compartido de recursos entre orígenes (CORS) para los Servicios de Azure Storage](/rest/api/storageservices/Cross-Origin-Resource-Sharing--CORS--Support-for-the-Azure-Storage-Services).
* Configure la aplicación para establecer los encabezados de CORS. Por ejemplo,consulte [Habilitación de solicitudes entre orígenes (CORS)](/aspnet/core/security/cors) en la documentación de ASP.NET Core.

### <a name="cdn-fallback"></a>Reserva de red CDN
Debe considerar la forma en que la aplicación hará frente a un error o a la no disponibilidad temporal de la red CDN. Las aplicaciones cliente pueden usar copias de los recursos almacenados en la caché local (en el cliente) durante las solicitudes anteriores, o bien, se puede incluir código que detecte errores y, en su lugar, solicite recursos del origen (la carpeta de aplicaciones o el contenedor de blobs de Azure que contiene los recursos) si la red CDN no está disponible.
