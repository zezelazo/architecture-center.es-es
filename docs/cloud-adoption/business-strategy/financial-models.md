---
title: Creación de un modelo financiero para la transformación en la nube
titleSuffix: Enterprise Cloud Adoption
description: Cómo crear un modelo financiero para la transformación en la nube
author: BrianBlanchard
ms.date: 12/10/2018
ms.openlocfilehash: 9a4d6b7c30d3dc0509a1b26fa8d247ae39ab87ca
ms.sourcegitcommit: e7f8676bbffe500fc4d6deb603b7c0b7ba1884a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/10/2018
ms.locfileid: "53179733"
---
# <a name="enterprise-cloud-adoption-how-to-create-a-financial-model-for-cloud-transformation"></a>Adopción de la nube empresarial: Cómo crear un modelo financiero para la transformación en la nube

Crear un modelo financiero que represente con precisión el valor empresarial completo de cualquier transformación en la nube puede ser complicado. Los modelos financieros y las justificaciones empresariales tienden a ser diferente de una organización a otra. En este artículo se establecen algunas fórmulas y se señalan algunas cosas que normalmente faltan cuando se crea un modelo financiero.

## <a name="return-on-investment-roi"></a>Rentabilidad de la inversión (ROI)

La rentabilidad de la inversión (ROI) suele ser un criterio importante con los ejecutivos de máximo rango o la junta. ROI se usa para comparar diferentes formas de invertir recursos limitados de capital. La fórmula de ROI es bastante sencilla. Pero los datos necesarios para crear cada entrada de la fórmula pueden no ser tan simples. Básicamente, ROI es la cantidad de rentabilidad generada por una inversión inicial. Normalmente se representa como un porcentaje:

![La rentabilidad de la inversión (ROI) es igual a (ganancia de la inversión - costo de la inversión)/costo de la inversión](../_images/formula-roi.png)

<!-- markdownlint-disable MD036 -->
*ROI = (ganancia de la inversión &minus; inversión inicial)/inversión inicial*
<!-- markdownlint-enable MD036 -->

En las secciones siguientes, se le guía por los datos necesarios para calcular la inversión inicial y la ganancia de la inversión (beneficios).

## <a name="calculating-initial-investment"></a>Cálculo de la inversión inicial

La inversión inicial es el gasto de capital (CapEx) y el gasto operativo (OpEx) necesarios para llevar a cabo una transformación. La clasificación de los costos puede variar según los modelos de contabilidad y las preferencias del CFO. Sin embargo, esta categoría incluye aspectos como: servicios profesionales para transformar, licencias de software que se usan únicamente durante la transformación, costo de los servicios en la nube durante la transformación y, posiblemente, el costo de los empleados asalariados durante la transformación.

Sume estos costos para crear una estimación de la inversión inicial.

## <a name="calculating-the-gain-from-investment"></a>Cálculo de la ganancia de la inversión

El cálculo de la ganancia de la inversión requiere con frecuencia una segunda fórmula, que es muy específica de los resultados empresariales y los cambios técnicos asociados. Calcular los beneficios no es tan sencillo como calcular la reducción en los costos.

Para calcular los beneficios, se requieren dos variables:

![Ganancia de la inversión es igual a ingresos diferenciales + costos diferenciales](../_images/formula-gain-from-investment.png)

<!-- markdownlint-disable MD036 -->
*Ganancia de la inversión = ingresos diferenciales + costos diferenciales*
<!-- markdownlint-enable MD036 -->

A continuación se describe cada uno.

## <a name="revenue-delta"></a>Ingresos diferenciales

Los ingresos diferenciales se deben prever en colaboración con la empresa. Una vez que las partes interesadas acuerdan un impacto sobre los ingresos, se puede usar para mejorar la posición de los beneficios.

## <a name="cost-deltas"></a>Costos diferenciales

Los costos diferenciales son la cantidad de aumento o disminución que llegará como resultado de la transformación. Hay una serie de variables independientes que pueden afectar a los costos diferenciales. Los beneficios se basan en gran medida en los costos directos como la reducción de gastos de capital, la prevención de costos, las reducciones de costos operativos y las reducciones de amortización. Las secciones siguientes son ejemplos de costos diferenciales que se deben tener en cuenta.

### <a name="depreciation-reductions-or-acceleration"></a>Reducción o aceleración de la depreciación

Para más información sobre la depreciación, hable con el CFO o el equipo financiero. La siguiente información sirve de referencia general sobre al tema de la depreciación.

Cuando se invierte capital en la adquisición de un recurso, esa inversión podría usarse con fines fiscales o financieros para producir beneficios continuados durante la vida útil del recurso. Algunas compañías consideran que la depreciación es una ventaja fiscal positiva. Otras la consideran un gasto que se realiza constantemente, de forma parecida a otros gastos recurrentes atribuidos al presupuesto anual de TI.

Hable con la administración de hacienda para ver si es posible eliminar la depreciación y si contribuiría de forma positiva a los costos diferenciales.

### <a name="physical-asset-recovery"></a>Recuperación de los recursos físicos

En algunos casos, se pueden vender recursos retirados como una fuente de ingresos. A menudo, estos ingresos se agrupan dentro de la reducción de costos por motivos de simplicidad. Sin embargo, realmente es un aumento de los ingresos y debe tributarse como tal. Hable con la administración de hacienda para conocer la viabilidad de esta opción y cómo justificar los ingresos resultantes.

### <a name="operational-cost-reductions"></a>Reducciones de costos operativos

Los gastos recurrentes necesarios para el funcionamiento de la empresa a menudo se conocen como gastos operativos (OpEx). OpEx es una categoría muy amplia. En la mayoría de los modelos de contabilidad, incluiría las licencias de software, los gastos de hospedaje, las facturas de electricidad, los alquileres de bienes inmuebles, los gastos de refrigeración, el personal temporal necesario para las operaciones, el alquiler de equipos, las piezas de recambio, los contratos de mantenimiento, los servicios de reparación, los servicios de continuidad del negocio y recuperación ante desastres (BC/DR) y otros diversos gastos que no requieren la aprobación de gastos de capital.

Esta categoría es una de las áreas de ingresos más grandes a la hora de considerar un viaje de transformación operativa. Rara vez se desperdicia el tiempo invertido en hacer esta lista exhaustiva. Formule preguntas al CIO y al equipo financiero para asegurarse de que se justifican todos los costos operativos.

### <a name="cost-avoidance"></a>Prevención de costos

Cuando se espera un gasto operativo (OpEx), pero no se encuentra aún en un presupuesto aprobado, no se puede incluir en una categoría de reducción de costos. Por ejemplo, si las licencias de Microsoft y VMWare se deben volver a negociar y pagarse el próximo año, no son aún costos completos. Las reducciones en esos costos esperados se tratarían como costos operativos solo para el cálculo de los costos diferenciales. Sin embargo, de manera informal, se deben considerar "prevención de costos" hasta que finalice la negociación y aprobación del presupuesto.

### <a name="soft-cost-reductions"></a>Reducciones de costos indirectos

En algunas compañías también se podrían incluir los costos indirectos, como las reducciones en la complejidad operativa o la reducción del personal a jornada completa para operar un centro de datos. Sin embargo, la inclusión de tales costos puede ser desaconsejable. Incluir los costos indirectos introduce un supuesto sin documentar de que la reducción en los costos será igual a ahorros tangibles en estos. Los proyectos de tecnología rara vez dan lugar a una recuperación real de los costos indirectos.

### <a name="headcount-reductions"></a>Reducciones de plantilla

Los ahorros de tiempo para el personal se incluyen a menudo en la reducción de costos indirectos. Cuando esos ahorros de tiempo se asignan a la reducción real de salario o personal de TI, se podrían calcular por separado como una reducción de plantilla.

Dicho esto, las aptitudes necesarias en el entorno local se asignan generalmente a un conjunto de aptitudes parecidas (o de nivel superior) necesarias en la nube. Esto significa que, por lo general, no se despide a la gente tras una migración a la nube.

Una excepción es cuando un tercero o un proveedor de servicios administrados (MSP) proporcionan capacidad operativa. Si los sistemas de TI están administrados por un tercero, los costos operativos podrían reemplazarse por una solución o un MSP nativos de la nube. Es probable que un MSP nativo de nube trabaje de manera más eficiente y posiblemente a un menor costo. Si es así, las reducciones de costos operativos pertenecen a los cálculos de costos directos.

### <a name="capital-expense-reductions-or-avoidance"></a>Reducción o prevención de gasto de capital

Los gastos de capital (CapEx) son ligeramente diferentes a los gastos operativos. Por lo general, esta categoría está controlada por los ciclos de actualización o la expansión del centro de datos. Un ejemplo de una expansión del centro de datos sería un nuevo clúster de alto rendimiento para hospedar una solución de macrodatos o almacenamiento de datos, y normalmente se incluiría en una categoría de CapEx. Más comunes son los ciclos de actualización básicos. Algunas compañías tienen ciclos de actualización de hardware rígidos, lo que significa que los recursos se retiran y se sustituyen en ciclos regulares (normalmente cada 3, 5 u 8 años). A menudo, estos ciclos coinciden con los ciclos de concesión de recursos o la duración prevista de los equipos. Cuando se alcanza un ciclo de actualización, TI extrae el CapEx para adquirir nuevos equipos.

Si se aprueba y se presupuesta un ciclo de actualización, la transformación en la nube podría ayudar a eliminar ese costo. Si un ciclo de actualización está planeado, pero aún no se ha aprobado, la transformación en la nube podría crear una prevención de costos de CapEx. Ambos escenarios se agregarían al costo diferencial.
