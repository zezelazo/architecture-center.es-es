---
title: Criterios para elegir un almacén de datos
description: Información general sobre las opciones de Azure Compute
author: MikeWasson
ms.openlocfilehash: 70f746f80c29623004620d83eb38747777df7f84
ms.sourcegitcommit: 85334ab0ccb072dac80de78aa82bcfa0f0044d3f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2018
ms.locfileid: "35252880"
---
# <a name="criteria-for-choosing-a-data-store"></a>Criterios para elegir un almacén de datos

Azure admite muchos tipos de soluciones de almacenamiento de datos, cada una con funcionalidades y características diferentes. En este artículo se describen los criterios de comparación que se deben utilizar al evaluar un almacén de datos. El objetivo es ayudarle a determinar qué tipos de almacenamientos de datos pueden cumplir los requisitos de la solución.

## <a name="general-considerations"></a>Consideraciones generales

Para iniciar la comparación, recopile la máxima información posible de lo que se indica a continuación sobre las necesidades de datos. Esta información le ayudará a determinar qué tipos de almacenamientos de datos se adaptará mejor a sus necesidades.

### <a name="functional-requirements"></a>Requisitos funcionales

- **Formato de datos**. ¿Qué tipo de datos pretende almacenar? Los tipos comunes son los datos transaccionales, objetos JSON, telemetría, índices de búsqueda o archivos sin formato.
- **Tamaño de los datos**. ¿De qué tamaño son las entidades que necesita almacenar? ¿Estas entidades deben conservarse como un único documento o se pueden dividir entre varios documentos, tablas, colecciones, etc.?
- **Escala y estructura**. ¿Qué cantidad total de capacidad de almacenamiento necesita? ¿Prevé realizar particiones de los datos? 
- **Relaciones de los datos**. ¿Los datos deben admitir relaciones de uno a varios o de muchos a muchos? ¿Son las relaciones en sí una parte importante de los datos? ¿Necesita unir o combinar datos de otra forma desde dentro del mismo conjunto de datos o desde conjuntos de datos externos? 
- **Modelo de coherencia**. ¿Qué importancia tiene para que las actualizaciones realizada en un nodo aparezcan en otros nodos, antes de que se puedan realizar más cambios? ¿Puede aceptar una coherencia definitiva? ¿Necesita garantías ACID para las transacciones?
- **Flexibilidad de los esquemas**. ¿Qué tipo de esquemas se aplicará a los datos? ¿Usará un esquema fijo, un enfoque de esquema basado en escritura o un enfoque de esquema basado en lectura?
- **Simultaneidad**. ¿Qué tipo de mecanismo de simultaneidad desea usar al actualizar y sincronizar los datos? ¿La aplicación realizará varias actualizaciones que puedan entrar en conflicto? Si es así, puede requerir el bloqueo de registros y el control de simultaneidad pesimista. Como alternativa, ¿puede admitir controles de simultaneidad pesimista? En su caso, ¿basta con un control de simultaneidad sencillo basado en marcas de tiempo o necesita la funcionalidad agregada del control de simultaneidad de varias versiones?
- **Movimiento de datos**. ¿La solución deberá realizar las tareas ETL para mover datos a otros almacenes o almacenamientos de datos?
- **Ciclo de vida de los datos**. ¿Los datos son de solo una escritura o de varias lecturas? ¿Pueden moverse a un almacenamiento de acceso esporádico o en frío?
- **Otras características compatibles**. ¿Necesita otras características específicas, como la validación del esquema, agregaciones, indexación, búsqueda de texto completo, MapReduce u otras funcionalidades de consulta?

### <a name="non-functional-requirements"></a>Requisitos no funcionales

- **Rendimiento y escalabilidad**. ¿Cuáles son los requisitos de rendimiento de datos? ¿Tiene requisitos específicos para la velocidad de ingesta de datos y la velocidad de procesamiento de datos? ¿Cuáles son los tiempos de respuesta aceptables para realizar consultas y agregaciones de datos una vez ingeridos? ¿Cuánto necesitará escalar verticalmente el tamaño del almacén de datos? ¿La carta de trabajo es más de lectura intensiva o de escritura intensiva?
- **Confiabilidad**. ¿Qué SLA general necesita admitir? ¿Qué nivel de tolerancia a errores es necesario proporcionar para los consumidores de datos? ¿Qué tipo de funcionalidades de copia de seguridad y restauración necesita? 
- **Replicación**. ¿Necesitará que los datos se distribuyan entre varias réplicas o regiones? ¿Qué tipo de funcionalidad de replicación de datos necesita? 
- **Límites**. ¿Los límites de un almacén de datos particular admiten los requisitos de escalado, el número de conexiones y el rendimiento? 

### <a name="management-and-cost"></a>Administración y costo

- **Servicio administrado**. Cuando sea posible, utilice un servicio de datos administrado, a menos que necesite funcionalidades específicas que solo se puedan encontrar en un almacén de datos hospedado en IaaS.
- **Disponibilidad en regiones**. Si se trata de servicios administrados, ¿el servicio se encuentra disponible en todas las regiones de Azure? ¿La solución debe hospedarse en determinadas regiones de Azure?
- **Portabilidad**. ¿Los datos deben migrarse a centros de datos externos locales o a otros entornos de hospedaje en la nube?
- **Licencias**. ¿Tiene preferencia por un propietario frente al tipo de licencia de OSS? ¿Existen otras restricciones externas del tipo de licencia que puede usar?
- **Costo total**. ¿Cuál es el costo total de usar el servicio en la solución? ¿Cuántas instancias necesitará ejecutar para satisfacer los requisitos de tiempo de actividad y rendimiento? Tenga en cuenta los costos de las operaciones en este cálculo. Una razón para preferir servicios administrados es el menor costo operativo.
- **Rentabilidad**. ¿Puede crear una partición de los datos para almacenarlos con más rentabilidad? Por ejemplo, ¿puede mover objetos grandes de una base de datos relacional cara a un almacén de objetos?

### <a name="security"></a>Seguridad

- **Seguridad**. ¿Qué tipo de cifrado necesita? ¿Necesita cifrado en reposo? ¿Qué mecanismo de autenticación desea usar para conectarse a los datos?
- **Auditoría**. ¿Qué tipo de registro de auditoría necesita generar?
- **Requisitos de red**. ¿Necesita restringir o administrar el acceso a los datos desde otros recursos de red? ¿Necesita que los datos solo sean accesibles desde dentro del entorno de Azure? ¿Los datos deben ser accesibles desde subredes o direcciones IP específicas? ¿Deben ser accesibles desde aplicaciones o servicios hospedados en centros de datos locales o en otros centros de datos externos?

### <a name="devops"></a>DevOps

- **Serie de aptitudes**. ¿Existen lenguajes de programación concretos, sistemas operativos u otra tecnología que el equipo esté especialmente acostumbrado a usar? ¿Hay otros con los que a su equipo le resultaría difícil trabajar?
- **Clientes** ¿Existe una buena compatibilidad del cliente con los lenguajes de desarrollo?

En las siguientes secciones se comparan varios modelos de almacén de datos según el perfil de carga de trabajo, los tipos de datos y los casos de uso de ejemplo.

## <a name="relational-database-management-systems-rdbms"></a>Sistemas de administración de bases de datos relacionales (RDBMS)

<table>
<tr><td><strong>Carga de trabajo</strong></td>
    <td>
        <ul>
            <li>La creación de registros nuevos y las actualizaciones de los datos existentes se producen con regularidad.</li>
            <li>Se deben realizar varias operaciones en una sola transacción.</li>
            <li>Requiere que las funciones de agregación realicen la tabulación cruzada.</li>
            <li>Se necesita una fuerte integración con herramientas de informes.</li>
            <li>Las relaciones se aplican mediante restricciones de base de datos.</li>
            <li>Se usan índices para optimizar el rendimiento de las consultas.</li>
            <li>Permite el acceso a subconjuntos específicos de datos.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Tipo de datos</strong></td>
    <td>
        <ul>
            <li>Los datos están muy normalizados.</li>
            <li>Los esquemas de base de datos son necesarios y se aplican.</li>
            <li>Relaciones de muchos a muchos entre entidades de datos de la base de datos.</li>
            <li>Las restricciones se definen en el esquema y se imponen en todos los datos de la base de datos.</li>
            <li>Los datos necesitan una integridad elevada. Los índices y las relaciones deben mantenerse con precisión.</li>
            <li>Los datos requieren una sólida coherencia. Las transacciones funcionan de forma que garantizan que todos los datos sean totalmente coherentes para todos los usuarios y procesos.</li>
            <li>El tamaño de las entradas de datos individuales suele ser de pequeño a mediano.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Ejemplos</strong></td>
    <td>
        <ul>
            <li>Línea de negocio (administración del capital humano, administración de relaciones con clientes y planeamiento de recursos empresariales)</li>
            <li>Administración de inventario</li>
            <li>Informes de base de datos</li>
            <li>Control</li>
            <li>Administración de recursos</li>
            <li>Administración de fondos</li>
            <li>Administración de pedidos</li>
        </ul>
    </td>
</tr>
</table>

## <a name="document-databases"></a>Bases de datos de documentos

<table>
<tr><td><strong>Carga de trabajo</strong></td>
    <td>
        <ul>
            <li>Uso general.</li>
            <li>Las operaciones de inserción y actualización son comunes. La creación de registros nuevos y las actualizaciones de los datos existentes se producen con regularidad.</li>
            <li>No hay ningún error de coincidencia de impedancia relacional de objetos. Los documentos pueden coincidir mejor con las estructuras de objetos usadas en el código de la aplicación.</li>
            <li>La simultaneidad optimista se usa con más frecuencia.</li>
            <li>La aplicación consumidora debe modificar y procesar los datos.</li>
            <li>Los datos necesitan un índice en varios campos.</li>
            <li>Los documentos individuales se recuperan y escriben como un solo bloque.</li>
    </td>
</tr>
<tr><td><strong>Tipo de datos</strong></td>
    <td>
        <ul>
            <li>Los datos pueden administrarse de manera no normalizada.</li>
            <li>El tamaño de los datos de documentos individuales es relativamente pequeño.</li>
            <li>Cada tipo de documento puede usar su propio esquema.</li>
            <li>Los documentos pueden incluir campos opcionales.</li>
            <li>Los datos del documento son semiestructurados, lo que significa que los tipo de datos de cada campo no se definen estrictamente.</li>
            <li>Se admite la agregación de datos.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Ejemplos</strong></td>
    <td>
        <ul>
            <li>Catálogo de productos</li>
            <li>Cuentas de usuario</li>
            <li>Lista de materiales</li>
            <li>Personalización</li>
            <li>Administración de contenido</li>
            <li>Datos de operaciones</li>
            <li>Administración de inventario</li>
            <li>Datos del historial de transacciones</li>
            <li>Vista materializada de otros almacenes NoSQL. Reemplaza la indexación de archivos/BLOB.</li>
        </ul>
    </td>
</tr>
</table>

## <a name="keyvalue-stores"></a>Almacenes clave-valor

<table>
<tr><td><strong>Carga de trabajo</strong></td>
    <td>
        <ul>
            <li>Los datos se identifican y se accede a ellos con una clave de identificador única, como un diccionario.</li>
            <li>Escalable a gran escala.</li>
            <li>No se necesitan combinaciones, bloqueos ni uniones.</li>
            <li>No se usa ningún mecanismo de agregación.</li>
            <li>Los índices secundarios no se suelen usar.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Tipo de datos</strong></td>
    <td>
        <ul>
            <li>El tamaño de los datos tiende a ser grande.</li>
            <li>Cada clave está asociada con un valor único, que es un BLOB de datos no administrado.</li>
            <li>No hay ninguna aplicación del esquema.</li>
            <li>No existen relaciones entre entidades.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Ejemplos</strong></td>
    <td>
        <ul>
            <li>Almacenamiento en caché de datos</li>
            <li>Administración de sesiones</li>
            <li>Administración de perfiles y preferencias de usuario</li>
            <li>Recomendación de producto y servicio</li>
            <li>Diccionarios</li>
        </ul>
    </td>
</tr>
</table>

## <a name="graph-databases"></a>Bases de datos de gráficos

<table>
<tr><td><strong>Carga de trabajo</strong></td>
    <td>
        <ul>
            <li>Las relaciones entre elementos de datos son muy complejas, y conllevan muchos saltos entre los elementos de datos relacionados.</li>
            <li>La relación entre elementos de datos es dinámica y cambia con el tiempo.</li>
            <li>Las relaciones entre objetos son ciudadanos de primera clase, sin requerir claves externas ni combinaciones que recorrer.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Tipo de datos</strong></td>
    <td>
        <ul>
            <li>Los datos constan de nodos y relaciones.</li>
            <li>Los nodos son similares a las filas de tabla o documentos JSON.</li>
            <li>Las relaciones son tan importantes como los nodos y se exponen directamente en el lenguaje de consulta.</li>
            <li>Los objetos compuestos, como una persona con varios números de teléfono, tienden a dividirse en nodos independientes más pequeños que se combinan con relaciones que se pueden recorrer </li>
        </ul>
    </td>
</tr>
<tr><td><strong>Ejemplos</strong></td>
    <td>
        <ul>
            <li>Organigramas</li>
            <li>Gráficos sociales</li>
            <li>Detección de fraudes</li>
            <li>Análisis</li>
            <li>Motores de recomendaciones</li>
        </ul>
    </td>
</tr>
</table>

## <a name="column-family-databases"></a>Bases de datos de familia de columnas

<table>
<tr><td><strong>Carga de trabajo</strong></td>
    <td>
        <ul>
            <li>La mayoría de las bases de datos de familia de columnas realizan operaciones de escritura muy rápidamente.</li>
            <li>Las operaciones de actualización y eliminación son poco habituales.</li>
            <li>Diseñado para proporcionar acceso de baja latencia y alto rendimiento.</li>
            <li>Admite el acceso de consulta sencillo a un conjunto determinado de campos dentro de un registro mucho más grande.</li>
            <li>Escalable a gran escala.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Tipo de datos</strong></td>
    <td>
        <ul>
            <li>Los datos se almacenan en tablas formadas por una columna de clave y una o varias familias de columnas.</li>
            <li>Las columnas específicas pueden variar en filas individuales.</li>
            <li>Se accede a las celdas individuales a través de los comandos GET y PUT.</li>
            <li>Se devuelven varias filas utilizando un comando SCAN.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Ejemplos</strong></td>
    <td>
        <ul>
            <li>Recomendaciones</li>
            <li>Personalización</li>
            <li>datos del sensor</li>
            <li>Telemetría</li>
            <li>Mensajería</li>
            <li>Análisis de redes sociales</li>
            <li>Análisis web</li>
            <li>Supervisión de la actividad</li>
            <li>El tiempo y otros datos de serie temporal</li>
        </ul>
    </td>
</tr>
</table>

## <a name="search-engine-databases"></a>Bases de datos del motor de búsqueda

<table>
<tr><td><strong>Carga de trabajo</strong></td>
    <td>
        <ul>
            <li>Indexación de datos de varios orígenes y servicios.</li>
            <li>Las consultas son ad-hoc y pueden ser complejas.</li>
            <li>Se requiere agregación.</li>
            <li>Se requiere la búsqueda de texto completo.</li>
            <li>Se necesita una consulta de autoservicio ad-hoc.</li>
            <li>Se requiere un análisis de datos con el índice en todos los campos.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Tipo de datos</strong></td>
    <td>
        <ul>
            <li>Datos semiestructurados o sin estructurar</li>
            <li>Texto</li>
            <li>Texto con referencia a los datos estructurados</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Ejemplos</strong></td>
    <td>
        <ul>
            <li>Catálogos de productos</li>
            <li>Búsqueda de sitio</li>
            <li>Registro</li>
            <li>Análisis</li>
            <li>Sitios de la compra</li>
        </ul>
    </td>
</tr>
</table>

## <a name="data-warehouse"></a>Almacenamiento de datos

<table>
<tr><td><strong>Carga de trabajo</strong></td>
    <td>
        <ul>
            <li>Análisis de datos</li>
            <li>BI empresarial   </li>
        </ul>
    </td>
</tr>
<tr><td><strong>Tipo de datos</strong></td>
    <td>
        <ul>
            <li>Datos históricos de varios orígenes.</li>
            <li>Normalmente se desnormaliza en un esquema de &quot;estrella&quot; o &quot;copo de nieve&quot;, que consta de tablas de hechos y dimensiones.</li>
            <li>Suele cargarse con datos nuevos de forma programada.</li>
            <li>Las tablas de dimensiones suelen incluir varias versiones históricas de una entidad, conocida como <em>dimensión de variación lenta</em>.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Ejemplos</strong></td>
    <td>Un almacenamiento de datos empresarial que proporciona datos para modelos analíticos, informes y paneles.
    </td>
</tr>
</table>


## <a name="time-series-databases"></a>Bases de datos de series temporales

<table>
<tr><td><strong>Carga de trabajo</strong></td>
    <td>
        <ul>
            <li>Una proporción de operaciones masiva (95-99 %) son las escrituras.</li>
            <li>Los registros suelen anexarse secuencialmente en orden cronológico.</li>
            <li>Las actualizaciones son poco frecuentes.</li>
            <li>Las eliminaciones se producen de forma masiva y se realizan en bloques contiguos o registros.</li>
            <li>Las solicitudes de lectura pueden ser mayores que la memoria disponible.</li>
            <li>Es habitual que se produzcan varias lecturas al mismo tiempo.</li>
            <li>Los datos se leen secuencialmente en orden cronológico ascendente o descendente.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Tipo de datos</strong></td>
    <td>
        <ul>
            <li>Una marca de tiempo que se utiliza como la clave principal y el mecanismo de ordenación.</li>
            <li>Medidas de la entrada o descripciones de lo que la entrada representa.</li>
            <li>Etiquetas que definen información adicional sobre el tipo, el origen y otra información sobre la entrada.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Ejemplos</strong></td>
    <td>
        <ul>
            <li>Supervisión y telemetría de eventos.</li>
            <li>Sensor u otros datos de IoT.</li>
        </ul>
    </td>
</tr>
</table>

## <a name="object-storage"></a>Almacenamiento de objetos

<table>
<tr><td><strong>Carga de trabajo</strong></td>
    <td>
        <ul>
            <li>Identificado por clave.</li>
            <li>Los objetos pueden ser accesibles por vía pública o privada.</li>
            <li>El contenido suele ser un recurso como una hoja de cálculo, una imagen o un archivo de vídeo.</li>
            <li>El contenido debe ser duradero (permanente) y externo a cualquier máquina virtual o capa de aplicación.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Tipo de datos</strong></td>
    <td>
        <ul>
            <li>El tamaño de los datos es grande.</li>
            <li>Datos de blob.</li>
            <li>El valor es opaco.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Ejemplos</strong></td>
    <td>
        <ul>
            <li>Imágenes, vídeos, documentos de Office y archivos PDF</li>
            <li>CSS, scripts y CSV</li>
            <li>HTML estático, JSON</li>
            <li>Archivos de registro y auditoría</li>
            <li>Copias de seguridad de bases de datos</li>
        </ul>
    </td>
</tr>
</table>

## <a name="shared-files"></a>Archivos compartidos

<table>
<tr><td><strong>Carga de trabajo</strong></td>
    <td>
        <ul>
            <li>Migración de las aplicaciones existentes que interactúan con el sistema de archivos.</li>
            <li>Requiere la interfaz SMB.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Tipo de datos</strong></td>
    <td>
        <ul>
            <li>Archivos en un conjunto jerárquico de carpetas.</li>
            <li>Se puede acceder con bibliotecas estándar de E/S.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Ejemplos</strong></td>
    <td>
        <ul>
            <li>Archivos heredados</li>
            <li>Contenido compartido accesible entre una serie de máquinas virtuales o instancias de aplicaciones</li>
        </ul>
    </td>
</tr>
</table>
