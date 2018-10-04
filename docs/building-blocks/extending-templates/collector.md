---
title: Implementación de un recopilador y transformador de propiedades en una plantilla de Azure Resource Manager
description: Describe cómo implementar un recopilador y transformador de propiedades en una plantilla de Azure Resource Manager
author: petertay
ms.date: 06/09/2017
ms.openlocfilehash: 2c2fd93c977b82bed05ebe0ae68233a700df0f4f
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/28/2018
ms.locfileid: "47428591"
---
# <a name="implement-a-property-transformer-and-collector-in-an-azure-resource-manager-template"></a>Implementación de un recopilador y transformador de propiedades en una plantilla de Azure Resource Manager

En [Uso de un objeto como parámetro en una plantilla de Azure Resource Manager][objects-as-parameters], ha aprendido cómo almacenar valores de propiedad de recurso en un objeto y aplicarlos a un recurso durante la implementación. Aunque esta es una forma muy útil de administrar los parámetros, sigue siendo necesario que asigne las propiedades del objeto a las propiedades de los recursos cada vez que lo use en la plantilla.

Para evitar esto, puede implementar una plantilla de recopilador y transformador de propiedades que recorre la matriz de objetos en iteración y los transforma en el esquema JSON que el recurso espera.

> [!IMPORTANT]
> Esta técnica requiere que conozca a fondo las plantillas y las funciones de Resource Manager.

Veamos cómo se puede implementar un recopilador y transformador de propiedades con un ejemplo que implementa un [grupo de seguridad de red (NSG)][nsg]. El diagrama siguiente muestra la relación entre nuestras plantillas y nuestros recursos dentro de esas plantillas:

![arquitectura del recopilador y transformador de propiedades](../_images/collector-transformer.png)

Nuestra **plantilla de llamada** incluye dos recursos:
* Un vínculo de plantilla que invoca a nuestra **plantilla de recopilador**.
* El grupo de seguridad de red que se va a implementar.

Nuestra **plantilla de recopilador** incluye dos recursos:
* Un recurso de **delimitador**.
* Un vínculo de plantilla que invoca la plantilla de transformador en un bucle de copia.

Nuestra **plantilla de transformador** incluye un único recurso: una plantilla vacía con una variable que transforma nuestro código JSON `source` en el esquema JSON que esperaba el grupo de seguridad de red de la **plantilla principal**.

## <a name="parameter-object"></a>Objeto de parámetro

Usaremos nuestro objeto de parámetro `securityRules` de [objetos como parámetros][objects-as-parameters]. Nuestra **plantilla de transformación** transforma cada objeto de la matriz `securityRules` en el esquema JSON que esperaba el grupo de seguridad de red de nuestra **plantilla de llamada**.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters":{ 
      "networkSecurityGroupsSettings": {
      "value": {
          "securityRules": [
            {
              "name": "RDPAllow",
              "description": "allow RDP connections",
              "direction": "Inbound",
              "priority": 100,
              "sourceAddressPrefix": "*",
              "destinationAddressPrefix": "10.0.0.0/24",
              "sourcePortRange": "*",
              "destinationPortRange": "3389",
              "access": "Allow",
              "protocol": "Tcp"
            },
            {
              "name": "HTTPAllow",
              "description": "allow HTTP connections",
              "direction": "Inbound",
              "priority": 200,
              "sourceAddressPrefix": "*",
              "destinationAddressPrefix": "10.0.1.0/24",
              "sourcePortRange": "*",
              "destinationPortRange": "80",
              "access": "Allow",
              "protocol": "Tcp"
            }
          ]
        }
      }
    }
  }
```

Veamos primero la **plantilla de transformación**.

## <a name="transform-template"></a>Plantilla de transformación

La **Plantilla de transformación** incluye dos parámetros que se pasan desde la **plantilla de recopilador**: 
* `source` es un objeto que recibe uno de los objetos de valor de propiedad de la matriz de propiedades. En nuestro ejemplo, cada objeto de la matriz `"securityRules"` se pasará uno a uno.
* `state` es una matriz que recibe los resultados concatenados de todas las transformaciones anteriores. Esta es la colección de código JSON transformado.

Nuestros parámetros tienen este aspecto:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "source": { "type": "object" },
    "state": {
      "type": "array",
      "defaultValue": [ ]
    }
  },
```

La plantilla también define una variable llamada `instance`. Realiza la transformación real del objeto `source` en el esquema JSON requerido:

```json
  "variables": {
    "instance": [
      {
        "name": "[parameters('source').name]",
        "properties":{
            "description": "[parameters('source').description]",
            "protocol": "[parameters('source').protocol]",
            "sourcePortRange": "[parameters('source').sourcePortRange]",
            "destinationPortRange": "[parameters('source').destinationPortRange]",
            "sourceAddressPrefix": "[parameters('source').sourceAddressPrefix]",
            "destinationAddressPrefix": "[parameters('source').destinationAddressPrefix]",
            "access": "[parameters('source').access]",
            "priority": "[parameters('source').priority]",
            "direction": "[parameters('source').direction]"            
        }
      }
    ]

  },
```

Por último, el valor de `output` de nuestra plantilla concatena las transformaciones recopiladas del parámetro `state` con la transformación actual realizada por la variable `instance`:

```json
  "outputs": {
    "collection": {
      "type": "array",
      "value": "[concat(parameters('state'), variables('instance'))]"
    }
```

A continuación, veamos la **plantilla de recopilador** para ver cómo pasa los valores de parámetro.

## <a name="collector-template"></a>Plantilla de recopilador

La **plantilla de recopilador** incluye tres parámetros:
* `source` es la matriz completa de objetos de parámetro. La pasa la **plantilla de llamada**. Tiene el mismo nombre que el parámetro `source` de la **plantilla de transformación**, pero hay una diferencia importante que quizás haya observado ya: se trata de la matriz completa, pero solo pasamos un elemento de esta matriz a la **plantilla de transformación** a la vez.
* `transformTemplateUri` es el URI de nuestra **plantilla de transformación**. Aquí lo definimos como parámetro para poder volver a usar la plantilla.
* `state` es una matriz inicialmente vacía que pasamos a nuestra **plantilla de transformación**. Almacena la colección de objetos de parámetro transformados cuando el bucle de copia se completa.

Nuestros parámetros tienen este aspecto:

```json
  "parameters": {
    "source": { "type": "array" },
    "transformTemplateUri": { "type": "string" },
    "state": {
      "type": "array",
      "defaultValue": [ ]
    }
``` 

A continuación, definimos una variable llamada `count`. Su valor es la longitud de la matriz de objetos de parámetro `source`:

```json
  "variables": {
    "count": "[length(parameters('source'))]"
  },
```

Como puede imaginar, se usa para el número de iteraciones en nuestro bucle de copia.

Ahora, echemos un vistazo a nuestros recursos. Se definen dos recursos:
* `loop-0` es el recurso de base cero de nuestro bucle de copia.
* `loop-` se concatena con el resultado de la función `copyIndex(1)` para generar un nombre único basado en iteración para el recurso, comenzando por `1`.

Nuestros recursos tienen este aspecto:

```json
  "resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2015-01-01",
      "name": "loop-0",
      "properties": {
        "mode": "Incremental",
        "parameters": { },
        "template": {
          "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": { },
          "variables": { },
          "resources": [ ],
          "outputs": {
            "collection": {
              "type": "array",
              "value": "[parameters('state')]"
            }
          }
        }
      }
    },
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2015-01-01",
      "name": "[concat('loop-', copyindex(1))]",
      "copy": {
        "name": "iterator",
        "count": "[variables('count')]",
        "mode": "serial"
      },
      "dependsOn": [
        "loop-0"
      ],
      "properties": {
        "mode": "Incremental",
        "templateLink": { "uri": "[parameters('transformTemplateUri')]" },
        "parameters": {
          "source": { "value": "[parameters('source')[copyindex()]]" },
          "state": { "value": "[reference(concat('loop-', copyindex())).outputs.collection.value]" }
        }
      }
    }
  ],
```

Veamos más de cerca los parámetros que pasamos a nuestra **plantilla de transformación**, en la plantilla anidada. Recuerde que nuestro parámetro `source` pasa el objeto actual a la matriz de objetos de parámetro `source`. En el parámetro `state` se produce la recopilación, porque toma la salida de la iteración anterior de nuestro bucle de copia y lo pasa a la iteración actual. Observe que la función `reference()` usa la función `copyIndex()` sin ningún parámetro para hacer referencia al valor de `name` de nuestro objeto de plantilla vinculada anterior.

Por último, el valor de `output` de la plantilla devuelve el valor de `output` de la última iteración de la **plantilla de transformación**:

```json
  "outputs": {
    "result": {
      "type": "array",
      "value": "[reference(concat('loop-', variables('count'))).outputs.collection.value]"
    }
  }
```
A simple vista, puede parecer extraño devolver el valor de `output` de la última iteración de la **plantilla de transformación** a la **plantilla de llamada**, porque parecería que lo estamos almacenando en el parámetro `source`. Sin embargo, recuerde que la última iteración de la **plantilla de transformación** es la que contiene la matriz completa de los objetos de propiedad transformados, y eso es lo que queremos devolver.

Por último, echemos un vistazo a cómo llamar a la **plantilla de recopilador** desde la **plantilla de llamada**.

## <a name="calling-template"></a>Plantilla de llamada

La **plantilla de llamada** define un único parámetro llamado `networkSecurityGroupsSettings`:

```json
...
"parameters": {
    "networkSecurityGroupsSettings": {
        "type": "object"
    }
```

Después, la plantilla define una sola variable llamada `collectorTemplateUri`:

```json
"variables": {
    "collectorTemplateUri": "[uri(deployment().properties.templateLink.uri, 'collector.template.json')]"
  }
```

Como cabría esperar, este es el URI de la **plantilla de recopilador** que usará el recurso de plantilla vinculada:

```json
{
    "apiVersion": "2015-01-01",
    "name": "collector",
    "type": "Microsoft.Resources/deployments",
    "properties": {
        "mode": "Incremental",
        "templateLink": {
            "uri": "[variables('linkedTemplateUri')]",
            "contentVersion": "1.0.0.0"
        },
        "parameters": {
            "source" : {"value": "[parameters('networkSecurityGroupsSettings').securityRules]"},
            "transformTemplateUri": { "value": "[uri(deployment().properties.templateLink.uri, 'transform.json')]"}
        }
    }
}
```

Pasamos dos parámetros a la **plantilla de recopilador**:
* `source` es la matriz de objetos de propiedad. En nuestro ejemplo, es el parámetro `networkSecurityGroupsSettings`.
* `transformTemplateUri` es la variable que acabamos de definir con el URI de la **plantilla de recopilador**.

Por último, el recurso `Microsoft.Network/networkSecurityGroups` asigna directamente el valor de `output` del recurso de plantilla vinculada `collector` a su propiedad `securityRules`:

```json
    {
      "apiVersion": "2015-06-15",
      "type": "Microsoft.Network/networkSecurityGroups",
      "name": "networkSecurityGroup1",
      "location": "[resourceGroup().location]",
      "properties": {
        "securityRules": "[reference('firstResource').outputs.result.value]"
      }
    }
  ],
  "outputs": {
      "instance":{
          "type": "array",
          "value": "[reference('firstResource').outputs.result.value]"
      }

  }
```

## <a name="next-steps"></a>Pasos siguientes

* Esta técnica se implementa en el [proyecto de bloques de creación de plantillas](https://github.com/mspnp/template-building-blocks) y las [arquitecturas de referencia de Azure](/azure/architecture/reference-architectures/). Puede usarlas para crear su propia arquitectura o implementar una de nuestras arquitecturas de referencia.

<!-- links -->
[objects-as-parameters]: ./objects-as-parameters.md
[resource-manager-linked-template]: /azure/azure-resource-manager/resource-group-linked-templates
[resource-manager-variables]: /azure/azure-resource-manager/resource-group-template-functions-deployment
[nsg]: /azure/virtual-network/virtual-networks-nsg
