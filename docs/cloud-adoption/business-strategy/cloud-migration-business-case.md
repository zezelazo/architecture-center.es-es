---
title: Creación de un caso empresarial de migración a nube
titleSuffix: Enterprise Cloud Adoption
description: Aspectos que se deben tener en cuenta al crear una justificación empresarial para la migración a la nube
author: BrianBlanchard
ms.date: 12/10/2018
ms.openlocfilehash: dd712c83fd82e7d9efe40f4e382910e2403088b3
ms.sourcegitcommit: e7f8676bbffe500fc4d6deb603b7c0b7ba1884a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/10/2018
ms.locfileid: "53179738"
---
# <a name="enterprise-cloud-adoption-building-a-cloud-migration-business-case"></a>Adopción de la nube empresarial: Creación de un caso empresarial de migración a nube

Las migraciones a la nube pueden generar una rápida rentabilidad de la inversión (ROI) por los esfuerzos de transformación en la nube. Sin embargo, desarrollar una justificación empresarial clara con costos y rendimientos pertinentes y tangibles puede ser un proceso complejo. Este artículo le ayudará a pensar en qué datos son necesarios para crear un modelo financiero que se alinee con los resultados de la migración a la nube. En primer lugar, vamos a destruir unos cuantos mitos sobre la migración a la nube, de forma que su organización pueda evitar cometer algunos errores comunes.

## <a name="dispelling-cloud-migration-myths"></a>Acabar con los mitos de la migración a la nube

**Mito: la nube es siempre más barata.** Es una creencia común que tener un centro de datos en la nube es más barato que en el entorno local. Esto no es del todo cierto. En algunos casos, los costos de funcionamiento en la nube son en realidad más elevados. Esto se debe con frecuencia a la gestión deficiente de los costos, la falta de alineación de las arquitecturas del sistema, la duplicación de procesos, configuraciones inusuales del sistema o el aumento en los costos de personal. Por suerte, muchos de estos problemas se pueden mitigar para crear una rápida ROI. Siga las instrucciones que se indican en [Creación de la justificación empresarial](#building-the-business-justification) para que le ayuden a detectar y evitar estos desajustes. También puede ayudarle la información que aquí se proporciona sobre cómo acabar con los otros mitos.

**Mito: todo debe introducirse en la nube.** De hecho, hay una serie de factores empresariales que pueden llevarle a elegir una solución híbrida. Antes de finalizar el modelo de negocio, es aconsejable realizar una primera ronda de análisis cuantitativo, como se describe en los [artículos sobre el patrimonio digital](../digital-estate/5-rs-of-rationalization.md). Para más información sobre los factores cuantitativos individuales implicados en la racionalización, consulte [Las 5 "R" de la racionalización](../digital-estate/5-rs-of-rationalization.md). En cualquiera de los enfoques se usarán datos de inventario obtenidos fácilmente y un breve análisis cuantitativo para identificar cargas de trabajo o aplicaciones que podrían dar lugar a costos más elevados en la nube. Estos enfoques podrían identificar también dependencias o patrones de tráfico que necesitarían una solución híbrida.

**Mito: crear un reflejo de mi entorno local me ayuda a ahorrar dinero en la nube.** Durante el planeamiento del patrimonio digital, no es algo insólito que los clientes detecten capacidad sin usar por encima del 50 % del entorno aprovisionado. Si se aprovisionan recursos en la nube para que coincidan con el aprovisionamiento actual, será difícil que haya ahorros en los costos. Considere la posibilidad de reducir el tamaño de los recursos implementados para que se alineen con los patrones de uso, no con los patrones de aprovisionamiento.

**Mito: en la migración a la nube, los costos del servidor controlan los casos empresariales.** En ocasiones, es cierto. Para algunas compañías, es importante reducir los gastos de capital constantes relacionados con los servidores. Sin embargo, esto depende de varios factores. No es probable que las compañías con un ciclo de actualización de hardware de entre 5&ndash; y 8&ndash;años encuentren rendimientos rápidos en su migración a la nube. Las compañías con ciclos de actualización estandarizados o impuestos pueden puedan llegar a un umbral de rentabilidad rápidamente. En cualquier caso, otros gastos pueden ser los desencadenadores financieros que justifiquen la migración. Los siguientes son algunos ejemplos de los costos que normalmente se pasan por alto al adoptar una visión de los costos basada únicamente en el servidor o en la máquina virtual:

- Los costos de software de virtualización, servidores y middleware pueden ser amplios. Los proveedores de nube eliminan algunos de estos costos. Dos ejemplos de un proveedor de nube que reduce los costos de virtualización son los programas [Ventaja híbrida de Azure](https://azure.microsoft.com/pricing/hybrid-benefit/#services) y [Reservas](https://azure.microsoft.com/reservations/).
- Las pérdidas empresariales debido a interrupciones pueden superar rápidamente los costos de hardware o software. Si el centro de datos actual es inestable, trabaje con la empresa para cuantificar el impacto de las interrupciones en términos de costos de oportunidad o costos empresariales reales.
- Los costos del entorno también pueden influir en gran medida. Para una familia americana media, su casa es la inversión más grande y el costo más alto en su presupuesto. Con frecuencia, lo mismo puede decirse de los centros de datos. Los costos de bienes inmuebles, instalaciones y servicios públicos representan una buena parte de los costos locales. Cuando se retiran los centros de datos, la empresa puede reutilizar esas instalaciones, o podría quedar liberada de los costos completamente.

**Mito: el gasto operativo (OpEx) es mejor que el gasto de capital (CapEx).** Como se explica en el artículo sobre los [resultados fiscales](business-outcomes/fiscal-outcomes.md), el OpEx puede ser una buena opción. Sin embargo, hay un número de sectores que pueden considerar el OpEx de forma negativa. Los siguientes son algunos ejemplos que desencadenarían una integración más fuerte con las unidades de contabilidad y negocio respecto al debate OpEx:

- Cuando la empresa considera los activos fijos como un factor para la valoración del negocio, las reducciones de CapEx podrían ser un resultado negativo. Aunque no es un estándar universal, esta opinión se observa con más frecuencia en los sectores del comercio minorista, fabricación y construcción.
- Los aumentos de OpEx se pueden considerar también un resultado negativo en empresas propiedad de capital privado o que buscan una afluencia de capitales.
- Si la empresa está centrada principalmente en mejorar los márgenes de ventas o reducir el costo de bienes vendidos (COGS), el OpEx podría ser un resultado negativo.

OpEx no es siempre una mala opción. Es más probable que las empresas consideren OpEx más favorable que CapEx. Por ejemplo, este enfoque pueden adoptarlo también las empresas que intentan mejorar el flujo de efectivo, reducir las inversiones de capital o disminuir las tenencias de activos.

Antes de proporcionar una justificación empresarial centrada en una conversión de CapEx a OpEx, sepa qué es lo mejor para la empresa. Contabilidad y compras pueden con frecuencia alinear mejor el mensaje con los objetivos financieros.

**Mito: pasar a la nube es como darle a un botón.** Las migraciones son una transformación técnica manualmente intensa. Al desarrollar una justificación empresarial, en especial justificaciones que dependen del tiempo, tenga en cuenta los siguientes aspectos que podrían aumentar el tiempo necesario para migrar los recursos:

- Limitaciones de ancho de banda: la cantidad de ancho de banda entre el centro de datos actual y el proveedor de nube determinará las escalas de tiempo durante la migración.
- Escalas de tiempo de pruebas empresariales: probar las aplicaciones con la empresa para certificar la preparación y el rendimiento pueden llevar mucho tiempo. Es fundamental alinear los usuarios avanzados y los procesos de prueba.
- Plazos de ejecución de la migración: la cantidad de tiempo y esfuerzo humano necesarios para ejecutar la migración pueden aumentar los costos y retrasar los plazos. La asignación de empleados o partes contratantes puede retrasar también el proceso y se debe justificar en el plan.

Impedimentos técnicos y culturales pueden ralentizar la adopción de la nube. Cuando el tiempo es un aspecto importante de la justificación empresarial, la mejor solución de mitigación es el planeamiento adecuado. Existen dos enfoques sugeridos durante el planeamiento que pueden ayudar a mitigar los riesgos para los plazos.

- En primer lugar, invierta tiempo y energía en comprender las restricciones de la adopción técnica. Aunque la presión para realizar la transición rápidamente puede ser alta, es importante considerar plazos de ejecución realistas.
- En segundo lugar, si surgen impedimentos culturales o relacionados con las personas, su efecto será más grave que las restricciones técnicas. La adopción de la nube crea cambios, que producen la transformación deseada. Por desgracia, a veces las personas temen el cambio y pueden necesitar apoyo adicional para alinearse con el plan. Identifique las personas claves del equipo que se oponen al cambio y consiga su adhesión desde el principio.

Para aumentar la buena disposición y reducir los riesgos para los plazos, prepare a las partes interesadas para conseguir una fuerte alineación del valor y los resultados empresariales. Ayude a esas partes interesadas a comprender los cambios que acompañan a esta transformación. Sea claro y establezca expectativas realistas desde el principio. Cuando las personas o la tecnología ralenticen el proceso, será más fácil conseguir el apoyo ejecutivo.

## <a name="building-the-business-justification"></a>Creación de la justificación empresarial

El proceso siguiente define un enfoque para desarrollar la justificación empresarial para las migraciones a la nube. Mientras lee este contenido, si es necesario explicar mejor los cálculos o términos financieros, consulte el artículo sobre los [modelos financieros](financial-models.md).

En términos generales, la fórmula de justificación empresarial es sencilla. Sin embargo, los puntos de datos sutiles necesarios para rellenar la fórmula pueden ser difíciles de alinear. En general, la justificación empresarial se centra en la rentabilidad de la inversión (ROI) asociada con el cambio técnico propuesto. La fórmula genérica de la ROI es:

ROI = (ganancia de la inversión &minus; inversión inicial)/inversión inicial

Al desempaquetar esta fórmula se crea una vista de las fórmulas específica de la migración que lleva cada una de las variables de entrada por el lado adecuado de esta ecuación. En el resto de las secciones de este artículo se ofrecen algunos aspectos que se deben tener en cuenta.

![ROI es igual a (ganancia de la inversión - costo de la inversión)/costo de la inversión](../_images/formula-roi.png)

## <a name="migration-specific-initial-investment"></a>Inversión inicial específica de la migración

- Los proveedores de nube como Azure ofrecen calculadoras para estimar las inversiones en la nube. Una calculadora así es la [Calculadora de precios de Azure](https://azure.microsoft.com/en-in/pricing/).
- Algunos proveedores de nube también admiten calculadoras de costo diferencial. Un ejemplo de una calculadora de costo diferencial es la [Calculadora de costo total de propiedad (TCO) de Azure](https://azure.com/tco).
- Para estructuras de costo más refinadas, considere un ejercicio de [planeamiento del patrimonio digital](../digital-estate/overview.md).
- Estime el costo de la migración.
- Estime el costo de las oportunidades de aprendizaje esperadas. [Microsoft Learn](https://docs.microsoft.com/learn/) puede ayudarle a reducir esos costos.
- En algunas compañías, puede ser necesario incluir en los costos iniciales el tiempo invertido por los miembros del personal existentes. Para más información, consulte con la administración de hacienda.
- Valide con ellos los costos adicionales o costos de cargas.

## <a name="migration-specific-revenue-deltas"></a>Ingresos diferenciales específicos de la migración

Con frecuencia, este aspecto se pasa por alto al crear una justificación empresarial para la migración. En algunas áreas, la nube puede reducir los costos. Sin embargo, el objetivo final de cualquier transformación es producir mejores resultados con el tiempo. Para comprender las mejoras de los ingresos a largo plazo, considere los efectos de bajada. ¿Qué tecnologías nuevas estarán disponibles para la empresa tras la migración, que no se puedan aprovechar actualmente? ¿Qué proyectos u objetivos de negocio están bloqueados por las dependencias de tecnologías heredadas? ¿Qué programas están en espera, pendientes de los costos de tecnología con alto CapEx?

Después de considerar las oportunidades que crea la nube, trabaje con la empresa para calcular el aumento de los ingresos que podría provenir de esas oportunidades.

## <a name="migration-specific-cost-deltas"></a>Costos diferenciales específicos de la migración

Calcule los cambios en los costos como resultado de la migración propuesta. Para más información sobre los diferentes tipos de costos diferenciales, consulte [Modelos financieros](financial-models.md). Los proveedores de nube con frecuencia proporcionan herramientas para el cálculo de los costos diferenciales. Un ejemplo de una calculadora de costo diferencial es la [Calculadora de costo total de propiedad (TCO) de Azure](https://azure.com/tco).

Otros ejemplos de costos que pueden reducirse con una migración a la nube:

- Terminación o reducción del centro de datos (costos de entorno)
- Reducción en la energía consumida (costos de entorno)
- Terminación del bastidor (recuperación de recursos físicos)
- Impedir actualizaciones de hardware (prevención de costos)
- Evitar una renovación de software (reducción de costos operativos o prevención de costos)
- Consolidación del proveedor (reducción de costos operativos y posible reducción de costos indirectos)

## <a name="when-roi-results-are-surprising"></a>Cuando los resultados de la ROI son sorprendentes

Si la ROI de una migración a la nube no está en consonancia con las expectativas, puede ser útil revisar los mitos comunes enumerados al principio de este artículo.

Sin embargo, es importante entender que un resultado de ahorro en los costos no siempre es posible. Hay aplicaciones que cuesta más tener en la nube que en el entorno local. Estas aplicaciones pueden sesgar considerablemente los resultados en un análisis.

Cuando el ROI sea inferior al 20 %, podría realizar un ejercicio de [planeamiento del patrimonio digital](../digital-estate/overview.md) con una atención específica sobre la [racionalización](../digital-estate/rationalize.md). Durante el análisis cuantitativo, realice una revisión de cada aplicación para encontrar las cargas de trabajo que sesgan los resultados. Podría ser aconsejable quitar esas cargas de trabajo del plan. Si existen datos de uso, considere la posibilidad de reducir el tamaño de las máquinas virtuales para que coincidan con el uso.

Si aún no está alineado el ROI, solicite la ayuda de un representante de ventas de Microsoft o [hable con un asociado experimentado](https://azure.microsoft.com/en-us/migration/partners/).

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación de un modelo financiero para la transformación en la nube](financial-models.md)