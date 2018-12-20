---
title: Puntuación en tiempo real de los modelos de Machine Learning para R
description: Implemente un servicio de predicción en tiempo real en R mediante una instancia de Machine Learning Server que se ejecuta en Azure Kubernetes Service (AKS).
author: njray
ms.date: 12/12/18
ms.custom: azcat-ai
ms.openlocfilehash: a6069704c48fbc1f1a1e4b5df428011d6b5b883d
ms.sourcegitcommit: 62d2211badd1d6950e8cb819d70c9a4ab1ee01d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/12/2018
ms.locfileid: "53318999"
---
# <a name="real-time-scoring-of-r-machine-learning-models"></a>Puntuación en tiempo real de los modelos de Machine Learning para R

Esta arquitectura de referencia muestra cómo implementar un servicio de predicción en tiempo real (sincrónico) en R mediante Microsoft Machine Learning Server que se ejecuta en Azure Kubernetes Service (AKS). Esta arquitectura está pensada para ser genérica y adecuada para cualquier modelo predictivo creado en R que desee implementar como un servicio en tiempo real. **[Implemente la solución][github]**.

## <a name="architecture"></a>Arquitectura

![Puntuación en tiempo real de los modelos de Machine Learning para R en Azure][0]

Esta arquitectura de referencia adopta un enfoque basado en contenedores. Se construye una imagen de Docker que contiene R, así como los diversos artefactos necesarios para obtener nuevos datos. Estos incluyen el objeto modelo en sí y un script de puntuación. Esta imagen se envía a un registro de Docker hospedado en Azure, y después se implementa en un clúster de Kubernetes, también en Azure.

La arquitectura de este flujo de trabajo incluye los siguientes componentes.

- **[Azure Container Registry][acr]** se utiliza para almacenar las imágenes para este flujo de trabajo. Los registros creados con Container Registry se pueden administrar a través de la API estándar [Docker Registry V2 API][docker] y el cliente.

- **[Azure Kubernetes Service][aks]** se utiliza para hospedar el servicio y la implementación. Los clústeres creados con AKS se pueden administrar mediante la API estándar [Kubernetes API][k-api] y el cliente (kubectl).

- **[Microsoft Machine Learning Server][mmls]** se utiliza para definir la API REST para el servicio e incluye la [operacionalización del modelo][operationalization]. Este proceso de servidor web orientado al servicio escucha las solicitudes, que después se transfieren a otros procesos en segundo plano que ejecutan el código R real para generar los resultados. Todos estos procesos se ejecutan en un único nodo de esta configuración, que se encapsula en un contenedor. Para más información sobre el uso de este servicio fuera de un entorno de desarrollo o de prueba, póngase en contacto con su representante de Microsoft.

## <a name="performance-considerations"></a>Consideraciones sobre rendimiento

Las cargas de trabajo de aprendizaje automático tienden a ser muy intensivas, tanto cuando se trata del entrenamiento como cuando se obtienen nuevos datos. Como regla general, trate de no ejecutar más de un proceso de puntuación por núcleo. Machine Learning Server le permite definir el número de procesos de R que se ejecutan en cada contenedor. El valor predeterminado es de cinco procesos. Cuando se crea un modelo relativamente simple, como una regresión lineal con un pequeño número de variables, o un pequeño árbol de decisión, se puede aumentar el número de procesos. Supervise la carga de la CPU en los nodos del clúster para determinar el límite apropiado en el número de contenedores.

Un clúster compatible con la GPU puede acelerar algunos tipos de cargas de trabajo y, en particular, los modelos de aprendizaje profundo. No todas las cargas de trabajo pueden aprovechar las GPU &mdash;, solo aquellas que hacen un uso intensivo del álgebra de matrices. Por ejemplo, los modelos basados en árboles, incluidos los bosques aleatorios y los modelos de boosting, no suelen obtener ninguna ventaja de las GPU.

Algunos tipos de modelos como los bosques aleatorios se pueden paralelizar de forma masiva en las CPU. En estos casos, acelere la puntuación de una única solicitud al distribuir la carga de trabajo entre varios núcleos. Sin embargo, hacerlo reduce su capacidad para tratar varias solicitudes de puntuación dado un tamaño de clúster fijo.

En general, los modelos de R de código abierto almacenan todos los datos en la memoria, de modo que asegúrese de que los nodos tengan suficiente memoria para acomodar los procesos que planea ejecutar simultáneamente. Si utiliza Machine Learning Server para adaptarlo a sus modelos, utilice las bibliotecas que pueden procesar datos en disco, en lugar de leerlos todos en memoria. Esto puede ayudar a reducir significativamente los requisitos de memoria. Con independencia de si utiliza Machine Learning Server o R de código abierto, supervise los nodos para asegurarse de que los procesos de puntuación no se queden sin memoria.

## <a name="security-considerations"></a>Consideraciones sobre la seguridad

### <a name="network-encryption"></a>Cifrado de red

En esta arquitectura de referencia, HTTPS se habilita para la comunicación con el clúster, y se utiliza un certificado de almacenamiento provisional de [Let's Encrypt][encrypt]. Para fines de producción, sustituya su propio certificado por el de una autoridad de firma apropiada.

### <a name="authentication-and-authorization"></a>Autenticación y autorización

La [operacionalización de modelos][operationalization] de Machine Learning Server requiere la autenticación de las solicitudes de puntuación. En esta implementación, se utilizan un nombre de usuario y una contraseña. En un entorno empresarial, puede habilitar la autenticación mediante [Azure Active Directory][AAD] o crear un front-end independiente mediante [Azure API Management][API].

Para que la operacionalización del modelo funcione correctamente con Machine Learning Server en contenedores, debe instalar un certificado JSON Web Token (JWT). Esta implementación utiliza un certificado proporcionado por Microsoft. En un entorno de producción, proporcione los suyos propios.

Para el tráfico entre Container Registry y AKS, considere la posibilidad de habilitar [un control de acceso basado en rol][rbac] (RBAC) para limitar los privilegios de acceso solo a los necesarios. 

### <a name="separate-storage"></a>Almacenamiento independiente

Esta arquitectura de referencia agrupa la aplicación (R) y los datos (objeto de modelo y script de puntuación) en una sola imagen. En algunos casos, puede resultar útil separarlos. Puede colocar los datos y el código del modelo en el blob de Azure o en [almacenamiento][storage] de archivos, y recuperarlos al inicializar el contenedor. En este caso, asegúrese de que la cuenta de almacenamiento esté configurada para permitir únicamente el acceso autenticado y que requiera HTTPS.

## <a name="monitoring-and-logging-considerations"></a>Consideraciones acerca de la supervisión y el registro

Utilice el [panel de Kubernetes][dashboard] para supervisar el estado general del clúster de AKS. Consulte la hoja de información general del clúster en Azure Portal para obtener más detalles. Los recursos de [GitHub][github] también muestran cómo abrir el panel desde R.

Aunque el panel le ofrece una visión del estado general del clúster, también es importante hacer un seguimiento del estado de los contenedores individuales. Para ello, habilite [Azure Monitor Insights][monitor] desde la hoja de información general del clúster en Azure Portal, o consulte [Azure Monitor para contenedores][monitor-containers] (en versión preliminar).

## <a name="cost-considerations"></a>Consideraciones sobre el costo

Machine Learning Server tiene licencia por núcleo, y todos los núcleos del clúster que van a ejecutar Machine Learning Server cuentan para ello. Si es un cliente empresarial de Machine Learning Server o Microsoft SQL Server, póngase en contacto con su representante de Microsoft para obtener información sobre precios.

Una alternativa de código abierto a Machine Learning Server es [Plumber][plumber], un paquete de R que convierte el código en una API REST. Plumber tiene menos funciones que Machine Learning Server. Por ejemplo, de forma predeterminada no incluye ninguna característica que proporcione autenticación de solicitudes. Si se utiliza Plumber, se recomienda que habilite [Azure API Management][API] para tratar los detalles de autenticación.

Además de la concesión de licencias, la principal consideración de costos son los recursos de proceso del clúster de Kubernetes. El clúster debe ser lo suficientemente grande como para controlar el volumen de solicitudes previsto en las horas punta, pero este enfoque deja los recursos inactivos en otros momentos. Para limitar el impacto de los recursos inactivos, habilite la [escalabilidad automática horizontal][autoscaler] para el clúster mediante la herramienta kubectl. O bien, utilice la [escalabilidad automática de clústeres][cluster-autoscaler] de AKS.

## <a name="deploy-the-solution"></a>Implementación de la solución

Hay disponible una implementación de referencia de esta arquitectura en [GitHub][github]. Siga los pasos descritos para implementar un modelo predictivo simple como servicio.

<!-- links -->
[AAD]: /azure/active-directory/fundamentals/active-directory-whatis
[API]: /azure/api-management/api-management-key-concepts
[ACR]: /azure/container-registry/container-registry-intro
[AKS]: /azure/aks/intro-kubernetes
[autoscaler]: https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/
[cluster-autoscaler]: /azure/aks/autoscaler
[monitor]: /azure/monitoring/monitoring-container-insights-overview
[dashboard]: /azure/aks/kubernetes-dashboard
[docker]: https://docs.docker.com/registry/spec/api/
[encrypt]: https://letsencrypt.org/
[gitHub]: https://github.com/Azure/RealtimeRDeployment
[K-API]: https://kubernetes.io/docs/reference/
[MMLS]: /machine-learning-server/what-is-machine-learning-server
[monitor-containers]: /azure/azure-monitor/insights/container-insights-overview
[operationalization]: /machine-learning-server/what-is-operationalization
[plumber]: https://www.rplumber.io
[RBAC]: /azure/role-based-access-control/overview
[storage]: /azure/storage/common/storage-introduction
[0]: ./_images/realtime-scoring-r.png
