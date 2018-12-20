---
title: Enfoques de planeamiento del patrimonio digital
titleSuffix: Enterprise Cloud Adoption
description: Se describen algunos de los enfoques del planeamiento del patrimonio digital.
author: BrianBlanchard
ms.date: 12/10/2018
ms.openlocfilehash: 5803447ff87e733aa5a9c24ac626bba665b8394a
ms.sourcegitcommit: e7f8676bbffe500fc4d6deb603b7c0b7ba1884a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/10/2018
ms.locfileid: "53179724"
---
# <a name="enterprise-cloud-adoption-approaches-to-digital-estate-planning"></a>Adopción de la nube empresarial: Enfoques de planeamiento del patrimonio digital

El planeamiento del patrimonio digital puede adoptar varias formas, según los resultados deseados y el tamaño del patrimonio existente. También hay una serie de opciones en relación con el enfoque adoptado. Es importante establecer expectativas respecto al enfoque al principio de los ciclos de planeamiento. La falta de claridad de las expectativas suele dar lugar a retrasos asociados con ejercicios adicionales de recopilación de inventario. En este artículo se describen tres enfoques de análisis.

## <a name="workload-driven-approach"></a>Enfoque basado en la carga de trabajo

El enfoque de valoración descendente evalúa primero los aspectos de seguridad, como los requisitos de clasificación de los datos (impacto empresarial alto, medio o bajo), cumplimiento, soberanía y riesgo de seguridad. Después, valora la complejidad general de la arquitectura, evalúa aspectos como autenticación, estructura de datos, requisitos de latencia, dependencias y esperanza de vida de las aplicaciones. Seguidamente, el enfoque descendente mide los requisitos operativos de la aplicación, como los niveles de servicio, la integración, las ventanas de mantenimiento, la supervisión y las conclusiones. Cuando todos estos aspectos se han analizado y considerado, el resultado es una puntuación que refleja la dificultad relativa para migrar esta aplicación a cada una de las plataformas de nube: IaaS, PaaS y SaaS.

En segundo lugar, la valoración descendente evalúa las ventajas financieras de la aplicación, como las eficiencias operativas, el TCO, la rentabilidad de la inversión u otras métricas financieras adecuadas. Además, la valoración también examina la estacionalidad de la aplicación (¿existen picos de demanda en algunos momentos del año?) y la carga de proceso general. También, examina los tipos de usuarios que admite (ocasional/experto, conectado siempre/ocasionalmente), y por lo tanto, la escalabilidad y elasticidad necesarias. Para finalizar, la valoración examina los requisitos de continuidad empresarial y resiliencia que podría tener la aplicación, así como las dependencias para ejecutarla en caso de que se produzca una interrupción del servicio.

> [!TIP]
> Para llevar a cabo este enfoque se requieren entrevistas y comentarios anecdóticos de las partes interesadas empresariales y técnicas. El principal riesgo para el tiempo es la disponibilidad de los individuos clave. La naturaleza anecdótica de los orígenes de datos hace que sea más difícil realizar estimaciones precisas de costo y tiempo. Planee de antemano el calendario y valide todos los datos recopilados.

## <a name="asset-driven-approach"></a>Enfoque basado en los recursos

El enfoque basado en los recursos proporciona un plan basado en los recursos que admite una aplicación que se va a migrar. En este enfoque, se extraen datos de uso estadísticos de una base de datos de administración de configuración (CMDB) u otras herramientas de valoración de la infraestructura. Normalmente, en este enfoque se da por hecho un modelo IaaS de implementación como base de referencia. En este proceso, el análisis evalúa los atributos de cada recurso: memoria, número de procesadores (núcleos de CPU), espacio de almacenamiento del sistema operativo, unidades de datos, tarjetas de interfaz de red (NIC), IPv6, equilibrio de carga de red, agrupación en clústeres, versión del sistema operativo, versión de la base de datos (si es necesario), dominios admitidos y componentes o paquetes de software de terceros, entre otros. Los recursos inventariados en este enfoque se alinean luego con las cargas de trabajo o las aplicaciones con fines de agrupación y asignación de dependencias.

> [!TIP]
> Para llevar a cabo este enfoque se requiere un origen abundante de datos de uso estadísticos. El tiempo para examinar el inventario y recopilar datos es el principal riesgo para el cumplimiento de los plazos. Los orígenes de datos de bajo nivel pueden perder las dependencias entre recursos o aplicaciones. Planee el examen del inventario durante un mes por lo menos. Valide las dependencias antes de la implementación.

## <a name="incremental-approach"></a>Enfoque incremental

De forma muy parecida al marco de adopción de la nube empresarial, es muy recomendable un enfoque incremental. En el caso del planeamiento del patrimonio digital, eso equivale a un proceso de varias fases, como el siguiente:

- Análisis del costo inicial: si se requiere validación financiera, comience con un enfoque basado en los recursos, descrita anteriormente, a fin de obtener un cálculo de los costos iniciales del patrimonio digital entero, sin racionalización. Esto establece una referencia de escenario del peor caso.

- Planeamiento de la migración: una vez que se ha asignado un equipo de estrategia de nube, cree un trabajo pendiente de migración inicial mediante un enfoque basado en la carga de trabajo, en función solamente de su conocimiento colectivo y algunas entrevistas con las partes interesadas. Este enfoque crea rápidamente una ligera valoración de la carga de trabajo para fomentar la colaboración.

- Planeamiento de la liberación: en cada liberación, el trabajo pendiente de migración se elimina y se vuelve a clasificar para centrarse en el impacto empresarial más significativo. Durante este proceso, las siguientes 5&ndash;10 cargas de trabajo se seleccionarían como liberaciones con prioridad. En este punto, el equipo de estrategia de nube dedicará tiempo a realizar un enfoque exhaustivo basado en la carga de trabajo. El hecho de retrasar esta valoración hasta que esté lista una liberación respeta mejor el tiempo de las partes interesadas. También retrasa la inversión en un análisis completo hasta que la empresa comience a ver resultados de los esfuerzos anteriores.

- Análisis de ejecución: antes de la migración, modernización o replicación de cualquier recurso, este debe evaluarse de forma individual y como parte de una liberación colectiva. En este momento, se pueden escrutar los datos del enfoque anterior basado en los recursos para garantizar el tamaño preciso y las restricciones operacionales.

> [!TIP]
> Este enfoque incremental simplifica el planeamiento y acelera los resultados. Es muy importante que todas las partes implicadas entiendan el enfoque hacia el retraso en la toma de decisiones. Es igualmente importante que se documenten los supuestos realizados en cada fase para evitar la pérdida de detalles.

## <a name="next-steps"></a>Pasos siguientes

Una vez seleccionado un enfoque, se puede recopilar el inventario.

> [!div class="nextstepaction"]
> [Recopilación de datos de inventario](inventory.md)