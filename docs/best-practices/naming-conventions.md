---
title: Convenciones de nomenclatura para los recursos de Azure
description: Convenciones de nomenclatura para los recursos de Azure. Qué nombre asignar a las máquinas virtuales, cuentas de almacenamiento, redes, redes virtuales, subredes y otras entidades de Azure
author: telmosampaio
ms.date: 05/18/2017
pnp.series.title: Best Practices
ms.openlocfilehash: 42d91da3eacdcda66b82dff82ba444170c11d7d1
ms.sourcegitcommit: 85334ab0ccb072dac80de78aa82bcfa0f0044d3f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2018
ms.locfileid: "35253067"
---
# <a name="naming-conventions"></a>Convenciones de nomenclatura

[!INCLUDE [header](../_includes/header.md)]

Este artículo es un resumen de las reglas y restricciones de la nomenclatura de los recursos de Azure, y un conjunto de recomendaciones que son la base de referencia de las convenciones de nomenclatura.  Dichas recomendaciones puede usarlas como punto de partida para la creación de convenciones propias específicas para sus necesidades.

La elección del nombre de cualquier recurso de Microsoft Azure es importante porque:

* Es difícil cambiar el nombre posteriormente.
* Los nombres deben cumplir los requisitos de su tipo de recurso concreto.

Una convención de nomenclatura coherente facilita la búsqueda de recursos. También puede indicar el rol de un recurso en una solución.

La clave para el éxito con las convenciones de nomenclatura es establecerlas y seguirlas en todas las aplicaciones y organizaciones.

## <a name="naming-subscriptions"></a>Nomenclatura de las suscripciones
Al asignar nombres a las suscripciones de Azure, los nombres detallados facilitan la comprensión del contexto y propósito de cada suscripción.  Si trabaja en un entorno con muchas suscripciones, seguir una convención de nomenclatura compartida puede mejorar la claridad.

Este es un patrón recomendado para la asignación de nombres a suscripciones:

`<Company> <Department (optional)> <Product Line (optional)> <Environment>`

* En la mayoría de los casos, la compañía será la misma en todas las suscripciones. Sin embargo, algunas compañías pueden tener compañías secundarios en su estructura organizativa. Estas compañías pueden ser administradas por un grupo de TI central. En estos casos, pueden diferenciarse porque tienen tanto el nombre de la empresa matriz (*Contoso*) como el nombre de la compañía secundaria (*Northwind*).
* El departamento es un nombre de la organización donde trabaja un grupo de individuos. Este elemento del espacio de nombres es opcional.
* La línea de productos es un nombre específico de un producto o una función que se realiza dentro del departamento. Esto es generalmente opcional para los servicios y aplicaciones internos. Sin embargo, se recomienda encarecidamente usarlo para los servicios de acceso público que requieran una sencilla separación e identificación (como una separación clara de los registros de facturación).
* El entorno es el nombre que describe el ciclo de vida de la implementación de las aplicaciones o servicios, como Desarrollo, Control de calidad o Producción.

| Compañía | Departamento | Línea de producto o servicio | Environment | Nombre completo |
| --- | --- | --- | --- | --- |
| Contoso |SocialGaming |AwesomeService |Producción |Producción de AwesomeService de SocialGaming de Contoso |
| Contoso |SocialGaming |AwesomeService |Desarrollo |Desarrollo de AwesomeService de SocialGaming de Contoso |
| Contoso |IT |InternalApps |Producción |Producción de InternalApps de TI de Contoso |
| Contoso |IT |InternalApps |Desarrollo |Producción de InternalApps de TI de Contoso |

Para más información acerca de cómo organizar las suscripciones en las empresas más grandes, lea las [instrucciones prescriptivas para la regulación de las suscripciones][scaffold].

## <a name="use-affixes-to-avoid-ambiguity"></a>Uso de afijos para evitar ambigüedades

Al asignar nombres a recursos de Azure, se recomienda usar prefijos o sufijos comunes para identificar el tipo de recurso y su contexto.  Aunque toda la información acerca del tipo, los metadatos y el contexto está disponible mediante programación, el uso de afijos comunes simplifica la identificación visual.  Al incorporar afijos a una convención de nomenclatura, es importante especificar con claridad si el afijo estará al principio del nombre (prefijo) o al final (sufijo).

Por ejemplo, a continuación se muestran dos nombres posibles para un servicio que hospeda un motor de cálculo:

* SvcCalculationEngine (prefijo)
* CalculationEngineSvc (sufijo)

Los afijos pueden hacer referencia a distintos aspectos que describen los recursos concretos. En la tabla siguiente se muestran algunos ejemplos que suelen usarse.

| Aspecto | Ejemplo | Notas |
| --- | --- | --- |
| Environment |desarrollo, producción, control de calidad |Identifica el entorno del recurso |
| Ubicación |uw (Oeste de EE. UU.), ue (este de EE. UU.) |Identifica la región en que se implementa el recurso |
| Instance |01, 02 |Para los recursos que tienen más de una instancia con nombre (servidores web, etc.). |
| Producto o servicio |Azure |Identifica el producto, la aplicación o el servicio que admite el recurso |
| Rol |sql, web, mensajería |Identifica el rol del recurso asociado |

Al desarrollar una convención de nomenclatura específica para una compañía o proyecto, es importante elegir un conjunto común de afijos, así como su posición (prefijo o sufijo).

## <a name="naming-rules-and-restrictions"></a>Reglas y restricciones de nomenclatura

Cada tipo de recurso o servicio de Azure exige un conjunto de restricciones y un ámbito de nomenclatura; todas las convenciones de nomenclatura o patrones deben adherirse a los requisitos de las reglas de nomenclaturas, así como a su ámbito.  Por ejemplo, mientras que el nombre de una máquina virtual se asigna a un nombre de DNS (y, por consiguiente, se requiere que sea único en todo Azure), el ámbito del nombre de una red virtual se sitúa en el grupo de recursos que se crea dentro.

En general, evite tener caracteres especiales (`-` o `_`) como primer o último carácter en ningún nombre. Estos caracteres harán que la mayoría de las reglas de validación produzcan un error.

### <a name="general"></a>General

| Entidad | Scope | Length | Uso de mayúsculas y minúsculas | Caracteres válidos | Patrón sugerido | Ejemplo |
| --- | --- | --- | --- | --- | --- | --- |
|Grupo de recursos |Subscription |1-90 |No distingue mayúsculas de minúsculas |Alfanuméricos, carácter de subrayado, paréntesis, guión, punto (excepto al final) |`<service short name>-<environment>-rg` |`profx-prod-rg` |
|Conjunto de disponibilidad |Grupo de recursos |1-80 |No distingue mayúsculas de minúsculas |Alfanuméricos, carácter de subrayado y guión |`<service-short-name>-<context>-as` |`profx-sql-as` |
|Etiqueta |Entidad asociada |512 (nombre), 256 (valor) |No distingue mayúsculas de minúsculas |Alfanuméricas |`"key" : "value"` |`"department" : "Central IT"` |

### <a name="compute"></a>Compute

| Entidad | Scope | Length | Uso de mayúsculas y minúsculas | Caracteres válidos | Patrón sugerido | Ejemplo |
| --- | --- | --- | --- | --- | --- | --- |
|Máquina virtual |Grupo de recursos |1-15 (Windows), 1-64 (Linux) |No distingue mayúsculas de minúsculas |Alfanuméricos y guión |`<name>-<role>-vm<number>` |`profx-sql-vm1` |
|Function App | Global |1-60 |No distingue mayúsculas de minúsculas |Alfanuméricos y guión |`<name>-func` |`calcprofit-func` |

> [!NOTE]
> Las máquinas virtuales en Azure tiene dos nombres distintos: el nombre de la máquina virtual y el nombre de host. Cuando se crea una máquina virtual en el portal, se utiliza el mismo nombre para el nombre de host y el nombre de recurso de máquina virtual. Las restricciones anteriores son para el nombre de host. El nombre de recurso real puede tener hasta 64 caracteres.

### <a name="storage"></a>Storage

| Entidad | Scope | Length | Uso de mayúsculas y minúsculas | Caracteres válidos | Patrón sugerido | Ejemplo |
| --- | --- | --- | --- | --- | --- | --- |
|Nombre de cuenta de almacenamiento (datos) |Global |3-24 |Minúsculas |Alfanuméricas |`<globally unique name><number>` (utilice una función para calcular un GUID único para asignar nombres a las cuentas de almacenamiento) |`profxdata001` |
|Nombre de cuenta de almacenamiento (discos) |Global |3-24 |Minúsculas |Alfanuméricas |`<vm name without hyphens>st<number>` |`profxsql001st0` |
| Nombre del contenedor |Cuenta de almacenamiento |3-63 |Minúsculas |Alfanuméricos y guión |`<context>` |`logs` |
|Nombre de blob | Contenedor |1-1024 |Distingue mayúsculas de minúsculas |Cualquier carácter de dirección URL |`<variable based on blob usage>` |`<variable based on blob usage>` |
|Nombre de cola |Cuenta de almacenamiento |3-63 |Minúsculas |Alfanuméricos y guión |`<service short name>-<context>-<num>` |`awesomeservice-messages-001` |
|Nombre de tabla | Cuenta de almacenamiento |3-63 |No distingue mayúsculas de minúsculas |Alfanuméricas |`<service short name><context>` |`awesomeservicelogs` |
|Nombre de archivo | Cuenta de almacenamiento |3-63 |Minúsculas | Alfanuméricas |`<variable based on blob usage>` |`<variable based on blob usage>` |
|Data Lake Store | Global |3-24 |Minúsculas | Alfanuméricas |`<name>dls` |`telemetrydls` |

### <a name="networking"></a>Redes

| Entidad | Scope | Length | Uso de mayúsculas y minúsculas | Caracteres válidos | Patrón sugerido | Ejemplo |
| --- | --- | --- | --- | --- | --- | --- |
|Virtual Network |Grupo de recursos |2-64 |No distingue mayúsculas de minúsculas |Alfanuméricos, guión, subrayado y punto |`<service short name>-vnet` |`profx-vnet` |
|Subred |Red virtual principal |2-80 |No distingue mayúsculas de minúsculas |Alfanuméricos, guión, subrayado y punto |`<descriptive context>` |`web` |
|Interfaz de red |Grupo de recursos |1-80 |No distingue mayúsculas de minúsculas |Alfanuméricos, guión, subrayado y punto |`<vmname>-nic<num>` |`profx-sql1-nic1` |
|Grupo de seguridad de red (NSG) |Grupo de recursos |1-80 |No distingue mayúsculas de minúsculas |Alfanuméricos, guión, subrayado y punto |`<service short name>-<context>-nsg` |`profx-app-nsg` |
|Regla de grupo de seguridad de red |Grupo de recursos |1-80 |No distingue mayúsculas de minúsculas |Alfanuméricos, guión, subrayado y punto |`<descriptive context>` |`sql-allow` |
|Dirección IP pública |Grupo de recursos |1-80 |No distingue mayúsculas de minúsculas |Alfanuméricos, guión, subrayado y punto |`<vm or service name>-pip` |`profx-sql1-pip` |
|Load Balancer |Grupo de recursos |1-80 |No distingue mayúsculas de minúsculas |Alfanuméricos, guión, subrayado y punto |`<service or role>-lb` |`profx-lb` |
|Configuración de reglas de carga equilibrada |Load Balancer |1-80 |No distingue mayúsculas de minúsculas |Alfanuméricos, guión, subrayado y punto |`<descriptive context>` |`http` |
|Azure Application Gateway |Grupo de recursos |1-80 |No distingue mayúsculas de minúsculas |Alfanuméricos, guión, subrayado y punto |`<service or role>-agw` |`profx-agw` |
|Perfil del Administrador de tráfico |Grupo de recursos |1-63 |No distingue mayúsculas de minúsculas |Alfanuméricos, guión y punto |`<descriptive context>` |`app1` |

## <a name="organize-resources-with-tags"></a>Organización de recursos con etiquetas

Azure Resource Manager admite entidades de etiquetado con cadenas de texto arbitrarias para identificar el contexto y simplificar la automatización.  Por ejemplo, la etiqueta `"sqlVersion: "sql2014ee"` puede identificar todas las máquinas virtuales de una implementación que ejecutan SQL Server 2014 Enterprise Edition con el fin de ejecutar un script automático en ellas.  Las etiquetas se deben utilizar para aumentar y mejorar el contexto en las convenciones de nomenclatura elegidas.

> [!TIP]
> Otra ventaja de las etiquetas es que abarcan grupos de recursos, lo que permite vincular y poner en correlación entidades de implementaciones dispares.

Cada recurso o grupo de recursos pueden tener un máximo de **15** etiquetas. El nombre de etiqueta está limitado a 512 caracteres y el valor de la etiqueta, a 256.

Para más información acerca del etiquetado de recursos, consulte [Uso de etiquetas para organizar los recursos de Azure](/azure/azure-resource-manager/resource-group-using-tags/).

Algunos de los casos de uso de etiquetado comunes son:

* **Facturación**; agrupación de recursos y su asociación con códigos de facturación o de contracargo.
* **Identificación de contexto de servicio**; identificar los grupos de recursos en Grupos de recursos para las operaciones comunes y la agrupación
* **Control de acceso y contexto de seguridad**; identificación de rol administrativo basado en cartera, sistema, servicio, aplicación, instancia, etc.

> [!TIP]
> Marcar pronto - marcar a menudo.  Es mejor tener un esquema de etiquetado de línea de base en vigor y ajustarlo con el paso del tiempo, en lugar de tener que actualizarlo a posteriori.

Un ejemplo de varios enfoques de etiquetado comunes:

| Nombre de etiqueta | Clave | Ejemplo | Comentario |
| --- | --- | --- | --- |
| Facturar a/ID de contracargo interno |billTo |`IT-Chargeback-1234` |Un código de facturación o de E/S internos |
| Operador o individuo directamente responsable (DRI) |managedBy |`joe@contoso.com` |Alias o dirección de correo electrónico |
| Nombre de proyecto |projectName |`myproject` |Nombre del proyecto o de la línea de producto |
| Versión de proyecto |projectVersion |`3.4` |Versión del proyecto o de la línea de producto |
| Environment |Environment |`<Production, Staging, QA >` |Identificador de entorno |
| Nivel: |Nivel: |`Front End, Back End, Data` |Identificación de nivel o rol/contexto |
| Perfil de datos |dataProfile |`Public, Confidential, Restricted, Internal` |Confidencialidad de los datos almacenados en el recurso |

## <a name="tips-and-tricks"></a>Trucos y sugerencias

Algunos tipos de recursos pueden requerir más atención en la asignación de nombres y en las convenciones.

### <a name="virtual-machines"></a>Máquinas virtuales

Especialmente en topologías grandes, la asignación meticulosa de nombres a las máquinas virtuales simplificará la identificación del rol y del propósito de cada máquina, y permitirá un scripting más predecible.

### <a name="storage-accounts-and-storage-entities"></a>Cuentas de almacenamiento y entidades de almacenamiento

Hay dos casos de uso principales para las cuentas de almacenamiento: respaldo de discos para máquinas virtuales y almacenamiento de datos en blobs, colas y tablas.  Las cuentas de almacenamiento que se usan para los discos de las máquinas virtuales deben seguir la convención de nomenclatura que asocia el nombre de la máquina virtual principal y, en caso de necesitar varias cuentas de almacenamiento para las SKU de máquina virtual de gama alta, también se puede aplicar un sufijo numérico.

> [!TIP]
> Las cuentas de almacenamiento, tanto para datos como para discos, deben seguir una convención de nomenclatura que permita sacar provecho de varias cuentas de almacenamiento (es decir, siempre con un sufijo numérico).

En la cuenta de Azure Storage se puede configurar un nombre de dominio personalizado para acceder a los datos del blob. El punto de conexión predeterminado de Blob service es https://\<name\>.blob.core.windows.net.

Pero si asigna un dominio personalizado (como www.contoso.com) al punto de conexión del blob para su cuenta de almacenamiento, también puede acceder a los datos del blob en su cuenta de almacenamiento mediante dicho dominio. Por ejemplo, con un nombre de dominio personalizado, `http://mystorage.blob.core.windows.net/mycontainer/myblob` podría tener acceso como `http://www.contoso.com/mycontainer/myblob`.

Para más información acerca de cómo configurar esta característica, consulte [Configurar un nombre de dominio personalizado para el punto de conexión de Almacenamiento de blobs](/azure/storage/storage-custom-domain-name/).

Para más información acerca de la asignación de nombres a blobs, contenedores y tablas, consulte la lista siguiente:

* [Asignar nombres y hacer referencia a contenedores, blobs y metadatos](https://msdn.microsoft.com/library/dd135715.aspx)
* [Asignar nombres a colas y metadatos](https://msdn.microsoft.com/library/dd179349.aspx)
* [Introducción al modelo de datos del servicio Tabla](https://msdn.microsoft.com/library/azure/dd179338.aspx)

Los nombres de blob pueden contener cualquier combinación de caracteres, pero a los caracteres de URL reservados se les debe aplicar la secuencia de escape correcta. Evite los nombres de blob que terminen en un punto (.), una barra diagonal (/), o una secuencia o combinación de ambos. Por convención, la barra diagonal es el separador de directorios **virtuales** . No use la barra invertida (\\) en los nombres de los blobs. Las API de cliente pueden permitirlo, pero el algoritmo hash no funcionará correctamente y las firmas no coincidirán.

Los nombres de las cuentas de almacenamiento o de los contenedores no se pueden modificar después de que se hayan creado. Si desea usar un nombre nuevo, debe eliminar el nombre anterior y crear uno nuevo.

> [!TIP]
> Se recomienda establecer una única convención de nomenclatura para todas las cuentas y tipos de almacenamiento antes de comenzar el desarrollo de un servicio o aplicación nuevos.

<!-- links -->

[scaffold]: /azure/azure-resource-manager/resource-manager-subscription-governance
