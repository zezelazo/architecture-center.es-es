---
title: Implementación condicional de un recurso en una plantilla de Azure Resource Manager
description: Describe cómo ampliar la funcionalidad de las plantillas de Azure Resource Manager para implementar condicionalmente un recurso en función del valor de un parámetro
author: petertay
ms.date: 10/30/2018
ms.openlocfilehash: 2c74e17a5f38f9225b696640a23b55b1285276bb
ms.sourcegitcommit: e9eb2b895037da0633ef3ccebdea2fcce047620f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/30/2018
ms.locfileid: "50251845"
---
# <a name="conditionally-deploy-a-resource-in-an-azure-resource-manager-template"></a>Implementación condicional de un recurso en una plantilla de Azure Resource Manager

En algunos escenarios, es necesario diseñar la plantilla para implementar un recurso en función de una condición, por ejemplo, si un valor de parámetro está presente o no. Por ejemplo, la plantilla puede implementar una red virtual e incluir parámetros para especificar otras redes virtuales para el emparejamiento. Si no se han especificado los valores de parámetro para el emparejamiento, no desea que Resource Manager implemente el recurso de emparejamiento.

Para ello, use el [elemento condition][azure-resource-manager-condition] en el recurso para probar la longitud de la matriz de parámetros. Si la longitud es cero, devuelve `false` para evitar la implementación pero, para todos los valores mayores que cero, devuelve `true` para permitir la implementación.

## <a name="example-template"></a>Plantilla de ejemplo

Echemos un vistazo a una plantilla de ejemplo que muestra cómo hacerlo. La plantilla usa el [elemento condition ][azure-resource-manager-condition] para controlar la implementación del recurso `Microsoft.Network/virtualNetworks/virtualNetworkPeerings`. Este recurso crea un emparejamiento entre dos redes virtuales de Azure en la misma región.

Veamos cada sección de la plantilla.

El elemento `parameters` define un único parámetro llamado `virtualNetworkPeerings`: 

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "virtualNetworkPeerings": {
      "type": "array",
      "defaultValue": []
    }
  },
```
Nuestro parámetro `virtualNetworkPeerings` es un valor de `array` y tiene el siguiente esquema:

```json
"virtualNetworkPeerings": [
    {
      "name": "firstVNet/peering1",
      "properties": {
          "remoteVirtualNetwork": {
              "id": "[resourceId('Microsoft.Network/virtualNetworks','secondVNet')]"
          },
          "allowForwardedTraffic": true,
          "allowGatewayTransit": true,
          "useRemoteGateways": false
      }
    }
]
```

Las propiedades del parámetro especifican la [configuración relativa al emparejamiento de las redes virtuales][vnet-peering-resource-schema]. Proporcionaremos los valores de estas propiedades cuando especifiquemos el recurso `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` en la sección `resources`:

```json
"resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2017-05-10",
      "name": "[concat('vnp-', copyIndex())]",
      "condition": "[greater(length(parameters('virtualNetworkPeerings')), 0)]",
      "dependsOn": [
        "firstVNet", "secondVNet"
      ],
      "copy": {
          "name": "iterator",
          "count": "[length(variables('peerings'))]",
          "mode": "serial"
      },
      "properties": {
        "mode": "Incremental",
        "template": {
          "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {
          },
          "variables": {
          },
          "resources": [
            {
              "type": "Microsoft.Network/virtualNetworks/virtualNetworkPeerings",
              "apiVersion": "2016-06-01",
              "location": "[resourceGroup().location]",
              "name": "[variables('peerings')[copyIndex()].name]",
              "properties": "[variables('peerings')[copyIndex()].properties]"
            }
          ],
          "outputs": {
          }
        }
      }
    }
]
```
Hay un par de cosas en esta parte de la plantilla. En primer lugar, el recurso real que se va a implementar es una plantilla en línea del tipo `Microsoft.Resources/deployments`, que incluye su propia plantilla que implementa realmente `Microsoft.Network/virtualNetworks/virtualNetworkPeerings`.

Para que el valor de `name` de la plantilla en línea sea único, se concatena la iteración actual de `copyIndex()` con el prefijo `vnp-`. 

El elemento `condition` especifica que el recurso debe procesarse cuando la función `greater()` se evalúe en `true`. En este caso, vamos a probar si la matriz de parámetros `virtualNetworkPeerings` es `greater()` que cero. Si lo es, se evalúa en `true` y se cumple `condition`. De lo contrario, es `false`.

Después, especificamos el bucle `copy`. Es un bucle `serial`, lo que significa que el bucle se realiza secuencialmente; cada recurso debe esperar a que el recurso anterior se haya implementado. La propiedad `count` especifica el número de veces que se recorre el bucle en iteración. Normalmente, aquí se establecería en la longitud de la matriz `virtualNetworkPeerings` porque contiene los objetos de parámetro que especifican el recurso que queremos implementar. Sin embargo, si hacemos esto, se producirá un error de validación si la matriz está vacía porque Resource Manager se da cuenta de que estamos intentando acceder a propiedades que no existen. Sin embargo, esto tiene solución. Veamos las variables que necesitamos:

```json
  "variables": {
    "workaround": {
       "true": "[parameters('virtualNetworkPeerings')]",
       "false": [{
           "name": "workaround",
           "properties": {}
       }]
     },
     "peerings": "[variables('workaround')[string(greater(length(parameters('virtualNetworkPeerings')), 0))]]"
  },
```

Nuestra variable `workaround` incluye dos propiedades, una llamada `true` y otra llamada `false`. La propiedad `true` se evalúa en el valor de la matriz de parámetros `virtualNetworkPeerings`. La propiedad `false` se evalúa en un objeto vacío, incluidas las propiedades con nombre que Resource Manager espera ver. Tenga en cuenta que, en realidad, `false` es una matriz, al igual que el parámetro `virtualNetworkPeerings`, que cumplirá la validación. 

La variable `peerings` utiliza la variable `workaround` y prueba una vez más si la longitud de la matriz de parámetros `virtualNetworkPeerings` es mayor que cero. Si es así, `string` se evalúa en `true` y la variable `workaround` se evalúa en la matriz de parámetros `virtualNetworkPeerings`. En caso contrario, se evalúa en `false` y la variable `workaround` se evalúa en nuestro objeto vacío en el primer elemento de la matriz.

Ahora que hemos solucionado el problema de validación, solo tenemos que especificar la implementación del recurso `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` en la plantilla anidada, pasando los valores de `name` y `properties` de nuestra matriz de parámetros `virtualNetworkPeerings`. Puede verlo en el elemento `template` anidado en el elemento `properties` de nuestro recurso.

## <a name="try-the-template"></a>Prueba de la plantilla

[GitHub][github] tiene una plantilla de ejemplo a su disposición. Para implementar la plantilla, ejecute los siguientes comandos de la [CLI de Azure][cli]:

```bash
az group create --location <location> --name <resource-group-name>
az group deployment create -g <resource-group-name> \
    --template-uri https://raw.githubusercontent.com/mspnp/template-examples/master/example2-conditional/deploy.json
```

## <a name="next-steps"></a>Pasos siguientes

* Use objetos en lugar de valores escalares como parámetros de plantilla. Consulte [Uso de un objeto como parámetro en una plantilla de Azure Resource Manager](./objects-as-parameters.md)

<!-- links -->
[azure-resource-manager-condition]: /azure/azure-resource-manager/resource-group-authoring-templates#resources
[azure-resource-manager-variable]: /azure/azure-resource-manager/resource-group-authoring-templates#variables
[vnet-peering-resource-schema]: /azure/templates/microsoft.network/virtualnetworks/virtualnetworkpeerings
[cli]: /cli/azure/?view=azure-cli-latest
[github]: https://github.com/mspnp/template-examples