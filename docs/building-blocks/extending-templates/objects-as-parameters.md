---
title: Uso de un objeto como parámetro en una plantilla de Azure Resource Manager
description: Se describe cómo ampliar la funcionalidad de las plantillas de Azure Resource Manager para utilizar objetos como parámetros.
author: petertay
ms.date: 06/09/2017
ms.openlocfilehash: dd53c55a26b2452c375d8d1e1a98886b15febaeb
ms.sourcegitcommit: 62945777e519d650159f0f963a2489b6bb6ce094
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/09/2018
ms.locfileid: "48876770"
---
# <a name="use-an-object-as-a-parameter-in-an-azure-resource-manager-template"></a>Uso de un objeto como parámetro en una plantilla de Azure Resource Manager

Cuando se [crean plantillas de Azure Resource Manager][azure-resource-manager-create-template], los valores de las propiedades de los recursos se pueden especificar directamente en la plantilla, o definir un parámetro y proporcionar los valores durante la implementación. Es correcto usar un parámetro para cada valor de propiedad en las implementaciones pequeñas, pero hay un límite de 255 parámetros por implementación. En las implementaciones más grandes y complejas, podría quedarse sin parámetros.

Una manera de solucionar este problema es usar un objeto como parámetro en lugar de un valor. Para ello, defina el parámetro en la plantilla y especifique un objeto JSON en lugar de un valor único durante la implementación. Después, haga referencia a las subpropiedades del parámetro con la [función `parameter()`][azure-resource-manager-functions] y un operador punto en la plantilla.

Veamos un ejemplo que implementa un recurso de red virtual. En primer lugar, vamos a especificar un parámetro `VNetSettings` en la plantilla y a establecer `type` en `object`:

```json
...
"parameters": {
    "VNetSettings":{"type":"object"}
},
```
Después, vamos a proporcionar los valores para el objeto `VNetSettings`:

> [!NOTE]
> Para ver cómo proporcionar los valores de los parámetros durante la implementación, consulte la sección **parámetros** del tema [Nociones sobre la estructura y la sintaxis de las plantillas de Azure Resource Manager][azure-resource-manager-authoring-templates]. 

```json
"parameters":{
    "VNetSettings":{
        "value":{
            "name":"VNet1",
            "addressPrefixes": [
                {
                    "name": "firstPrefix",
                    "addressPrefix": "10.0.0.0/22"
                }
            ],
            "subnets":[
                {
                    "name": "firstSubnet",
                    "addressPrefix": "10.0.0.0/24"
                },
                {
                    "name":"secondSubnet",
                    "addressPrefix":"10.0.1.0/24"
                }
            ]
        }
    }
}
```

Como puede ver, nuestro único parámetro especifica tres subpropiedades: `name`, `addressPrefixes` y `subnets`. Cada una de estas subpropiedades especifica un valor u otras subpropiedades. El resultado es que el único parámetro especifica todos los valores necesarios para implementar la red virtual.

Ahora, echemos un vistazo el resto de la plantilla para ver cómo se usa el objeto `VNetSettings`:

```json
...
"resources": [
    {
        "apiVersion": "2015-06-15",
        "type": "Microsoft.Network/virtualNetworks",
        "name": "[parameters('VNetSettings').name]",
        "location":"[resourceGroup().location]",
        "properties": {
          "addressSpace":{
              "addressPrefixes": [
                "[parameters('VNetSettings').addressPrefixes[0].addressPrefix]"
              ]
          },
          "subnets":[
              {
                  "name":"[parameters('VNetSettings').subnets[0].name]",
                  "properties": {
                      "addressPrefix": "[parameters('VNetSettings').subnets[0].addressPrefix]"
                  }
              },
              {
                  "name":"[parameters('VNetSettings').subnets[1].name]",
                  "properties": {
                      "addressPrefix": "[parameters('VNetSettings').subnets[1].addressPrefix]"
                  }
              }
          ]
        }
    }
  ]
```
Los valores de nuestro objeto `VNetSettings` se aplican a las propiedades requeridas por el recurso de red virtual, usando la función `parameters()` con el indexador de matriz `[]` y el operador punto. Esta técnica funciona si desea aplicar estáticamente los valores del objeto de parámetro al recurso. Sin embargo, si desea asignar una matriz de valores de propiedad dinámicamente durante la implementación, puede usar un [bucle de copia][azure-resource-manager-create-multiple-instances]. Para usar un bucle de copia, proporcione una matriz JSON de valores de propiedad de recurso, y el bucle de copia aplicará dinámicamente los valores a las propiedades del recurso. 

Hay un problema que hay que tener en cuenta si usa el método dinámico. Para ver este problema en acción, echemos un vistazo a una matriz de valores de propiedad típica. En este ejemplo, los valores de nuestras propiedades se almacenan en una variable. Observe que tenemos dos matrices aquí: una llamada `firstProperty` y otra llamada `secondProperty`. 

```json
"variables": {
    "firstProperty": [
        {
            "name": "A",
            "type": "typeA"
        },
        {
            "name": "B",
            "type": "typeB"
        },
        {
            "name": "C",
            "type": "typeC"
        }
    ],
    "secondProperty": [
        "one","two", "three"
    ]
}
```

Ahora, veamos cómo accedemos a las propiedades en la variable mediante un bucle de copia.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    ...
    "copy": {
        "name": "copyLoop1",
        "count": "[length(variables('firstProperty'))]"
    },
    ...
    "properties": {
        "name": { "value": "[variables('firstProperty')[copyIndex()].name]" },
        "type": { "value": "[variables('firstProperty')[copyIndex()].type]" },
        "number": { "value": "[variables('secondProperty')[copyIndex()]]" }
    }
}
```

La función `copyIndex()` devuelve la iteración actual del bucle de copia, que usamos como índice en cada una de las dos matrices simultáneamente.

Esto funciona bien cuando las dos matrices tienen la misma longitud. El problema surge si ha cometido un error y las dos matrices tienen distintas longitudes. En este caso, la plantilla producirá un error de validación durante la implementación. Para evitar este problema, incluya todas las propiedades en un único objeto, porque así es mucho más fácil ver cuándo falta un valor. Por ejemplo, veamos otro objeto de parámetro en el que cada elemento de la matriz `propertyObject` es la unión de las matrices `firstProperty` y `secondProperty` de antes.

```json
"variables": {
    "propertyObject": [
        {
            "name": "A",
            "type": "typeA",
            "number": "one"
        },
        {
            "name": "B",
            "type": "typeB",
            "number": "two"
        },
        {
            "name": "C",
            "type": "typeC"
        }
    ]
}
```

¿Ve el tercer elemento de la matriz? Falta la propiedad `number`, pero es mucho más fácil darse cuenta de que falta cuando los valores de parámetro se crean de esta manera.

## <a name="using-a-property-object-in-a-copy-loop"></a>Uso de un objeto de propiedad en un bucle de copia

Esta técnica resulta aún más útil cuando se combina con [serial copy loop][azure-resource-manager-create-multiple], especialmente para implementar recursos secundarios. 

Para demostrarlo, vamos una plantilla que implementa un [grupo de seguridad de red (NSG)][nsg] con dos reglas de seguridad. 

Primero, echemos un vistazo a nuestros parámetros. Si observamos nuestra plantilla, veremos que hemos definido un parámetro llamado `networkSecurityGroupsSettings`, que incluye una matriz llamada `securityRules`. Esta matriz contiene dos objetos JSON que especifican una serie de valores para una regla de seguridad.

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

Ahora, echemos un vistazo a la plantilla. El primer recurso, llamado `NSG1`, implementa el grupo de seguridad de red. El segundo recurso, llamado `loop-0`, realiza dos funciones: en primer lugar, `dependsOn` indica que depende del grupo de seguridad de red, de manera que su implementación no comenzará hasta que `NSG1` se haya completado, y es la primera iteración del bucle secuencial. El tercer recurso es una plantilla anidada que implementa las reglas de seguridad utilizando un objeto para sus valores de parámetro, como en el ejemplo anterior.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
      "networkSecurityGroupsSettings": {"type":"object"}
  },
  "variables": {},
  "resources": [
    {
      "apiVersion": "2015-06-15",
      "type": "Microsoft.Network/networkSecurityGroups",
      "name": "NSG1",
      "location":"[resourceGroup().location]",
      "properties": {
          "securityRules":[]
      }
    },
    {
        "apiVersion": "2015-01-01",
        "type": "Microsoft.Resources/deployments",
        "name": "loop-0",
        "dependsOn": [
            "NSG1"
        ],
        "properties": {
            "mode":"Incremental",
            "parameters":{},
            "template": {
                "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
                "contentVersion": "1.0.0.0",
                "parameters": {},
                "variables": {},
                "resources": [],
                "outputs": {}
            }
        }       
    },
    {
        "apiVersion": "2015-01-01",
        "type": "Microsoft.Resources/deployments",
        "name": "[concat('loop-', copyIndex(1))]",
        "dependsOn": [
          "[concat('loop-', copyIndex())]"
        ],
        "copy": {
          "name": "iterator",
          "count": "[length(parameters('networkSecurityGroupsSettings').securityRules)]"
        },
        "properties": {
          "mode": "Incremental",
          "template": {
            "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
            "contentVersion": "1.0.0.0",
           "parameters": {},
            "variables": {},
            "resources": [
                {
                    "name": "[concat('NSG1/' , parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].name)]",
                    "type": "Microsoft.Network/networkSecurityGroups/securityRules",
                    "apiVersion": "2016-09-01",
                    "location":"[resourceGroup().location]",
                    "properties":{
                        "description": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].description]",
                        "priority":"[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].priority]",
                        "protocol":"[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].protocol]",
                        "sourcePortRange": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].sourcePortRange]",
                        "destinationPortRange": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].destinationPortRange]",
                        "sourceAddressPrefix": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].sourceAddressPrefix]",
                        "destinationAddressPrefix": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].destinationAddressPrefix]",
                        "access":"[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].access]",
                        "direction":"[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].direction]"
                        }
                  }
            ],
            "outputs": {}
          }
        }
    }
  ],          
  "outputs": {}
}
```

Vamos a echar un vistazo más de cerca a cómo se especifican los valores de propiedad en el recurso secundario `securityRules`. Para hacer referencia a todas las propiedades, se usa la función `parameter()` y, a continuación, se usa el operador punto para hacer referencia a la matriz `securityRules`, indexada por el valor actual de la iteración. Por último, se usa otro operador punto para hacer referencia al nombre del objeto. 

## <a name="try-the-template"></a>Prueba de la plantilla

Si desea experimentar con esta plantilla, siga estos pasos: 

1.  Vaya a Azure Portal, seleccione el icono **+** y busque el tipo de recurso **implementación de plantilla**.
2.  Vaya a la página **Implementación de plantilla** y seleccione el botón **crear**. Este botón abre la hoja de **Implementación personalizada**.
3.  Seleccione el botón **Editar plantilla**.
4.  Elimine la plantilla vacía. 
5.  Copie y pegue la plantilla de ejemplo en el panel derecho.
6.  Seleccione el botón **Guardar**.
7.  Cuando vuelva al panel **Implementación personalizada**, seleccione el botón **Editar parámetros**.
8.  En la hoja **Editar parámetros**, elimine la plantilla existente.
9.  Copie y pegue la plantilla de parámetro de ejemplo anterior.
10. Seleccione el botón **Guardar**, que vuelve a abrir la hoja **Implementación personalizada**.
11. En la hoja **Implementación personalizada**, seleccione la suscripción, cree un grupo de recursos nuevo o use uno existente y seleccione una ubicación. Revise los términos y condiciones, y haga clic en la casilla **Acepto**.
12. Seleccione el botón **Comprar**.

## <a name="next-steps"></a>Pasos siguientes

* Puede ampliar estas técnicas para implementar un [recopilador y transformador de objetos de propiedad](./collector.md). Las técnicas de recopilador y transformador son más generales y se pueden vincular desde las plantillas.
* Esta técnica se implementa también en el [proyecto de bloques de creación de plantillas](https://github.com/mspnp/template-building-blocks) y las [arquitecturas de referencia de Azure](/azure/architecture/reference-architectures/). Puede revisar nuestras plantillas para ver cómo hemos implementado esta técnica.

<!-- links -->
[azure-resource-manager-authoring-templates]: /azure/azure-resource-manager/resource-group-authoring-templates
[azure-resource-manager-create-template]: /azure/azure-resource-manager/resource-manager-create-first-template
[azure-resource-manager-create-multiple-instances]: /azure/azure-resource-manager/resource-group-create-multiple
[azure-resource-manager-functions]: /azure/azure-resource-manager/resource-group-template-functions-resource
[nsg]: /azure/virtual-network/virtual-networks-nsg
