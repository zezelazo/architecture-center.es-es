# <a name="data-lakes"></a>Lagos de datos

Un lago de datos es un repositorio de almacenamiento que contiene una gran cantidad de datos en su formato nativo y sin procesar. Los lagos de datos están optimizados para escalar a terabytes y petabytes de datos. Los datos provienen típicamente de múltiples orígenes heterogéneos y pueden ser estructurados, semiestructurados o no estructurados. La idea con los lagos de datos es almacenar todo en su estado original, no transformado. Este enfoque difiere del [almacenamiento de datos](../relational-data/data-warehousing.md) tradicional, que transforma y procesa los datos en el momento de la ingesta.

Ventajas de un lago de datos:

- Los datos nunca se desechan, porque se almacenan en su formato sin procesar. Esto es especialmente útil en un entorno de macrodatos, cuando es posible que no se conozca de antemano qué información está disponible a partir de los datos.
- Los usuarios pueden explorar los datos y crear sus propias consultas.
- Puede ser más rápido que las herramientas de extracción, transformación y carga de datos tradicionales.
- Es más flexible que un almacenamiento de datos, porque puede almacenar datos no estructurados y semiestructurados. 

Una solución completa de lagos de datos consta de almacenamiento y procesamiento. El almacenamiento del lago de datos está diseñado para la tolerancia a errores, una escalabilidad infinita e ingesta de datos de alto rendimiento con diferentes formas y tamaños. El procesamiento del lago de datos implica uno o varios motores de procesamiento creados teniendo en cuenta estos objetivos y pueden operar en los datos almacenados en un lago de datos a escala.

## <a name="when-to-use-a-data-lake"></a>Cuándo usar un lago de datos

Entre los usos típicos de un lago de datos se incluye la [exploración de datos](./interactive-data-exploration.md), el análisis de datos y el aprendizaje automático. 

Un lago de datos también puede actuar como origen de datos para un almacenamiento de datos. Con este enfoque, los datos sin procesar se ingieren en el lago de datos y, después, se transforman en un formato estructurado consultable. Normalmente, esta transformación usa una canalización [ETL](../relational-data/etl.md#extract-load-and-transform-elt) (extracción, carga y transformación), donde los datos se ingieren y se transforman en su lugar. Los datos de origen que ya son relacionales pueden ir directamente al almacenamiento de datos, mediante un proceso ETL, omitiendo el lago de datos.

Los almacenes del lago de datos a menudo se usan en streaming de eventos o en escenarios de IoT, porque pueden guardar grandes cantidades de datos relacionales y no relacionales sin transformación o definición de esquema. Se crean para controlar grandes volúmenes de pequeñas operaciones de escritura en una latencia baja y se optimizan para que el rendimiento sea masivo.

## <a name="challenges"></a>Desafíos

- La falta de un esquema o de metadatos descriptivos puede hacer que los datos sean difíciles de consumir o de consultar.
- La falta de coherencia semántica en los datos puede dificultar la realización de análisis de los mismos, a menos que los usuarios estén altamente capacitados en analítica de datos.
- Puede ser difícil garantizar la calidad de los datos que entran en el lago de datos. 
- Sin un gobierno adecuado, el control del acceso y las cuestiones de privacidad pueden constituir problemas. ¿Qué información está entrando en el lago de datos, quién puede acceder a esos datos y para qué usos?
- Es posible que un lago de datos no sea la mejor forma de integrar datos que ya son relacionales.
- Por sí solo, un lago de datos no proporciona vistas integradas u holísticas a través de la organización. 
- Un lago de datos puede convertirse en un volcado de datos que nunca se analiza o se extrae para obtener información.

## <a name="relevant-azure-services"></a>Servicios correspondientes de Azure

- [Data Lake Store](/azure/data-lake-store/) es un repositorio de gran escala compatible con Hadoop.
- [Data Lake Analytics](/azure/data-lake-analytics/) es un servicio de trabajos de análisis a petición que simplifica el análisis de macrodatos.

