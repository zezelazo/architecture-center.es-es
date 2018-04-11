---
title: Actualización de un recurso en una plantilla de Azure Resource Manager
description: Se describe cómo ampliar la funcionalidad de las plantillas de Azure Resource Manager para actualizar un recurso.
author: petertay
ms.date: 06/09/2017
ms.openlocfilehash: fc2565819c66ee7695224ef5793e7276e6e552e0
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2017
---
# <a name="update-a-resource-in-an-azure-resource-manager-template"></a>Actualización de un recurso en una plantilla de Azure Resource Manager

Hay algunos escenarios en los que es necesario actualizar un recurso durante una implementación. Este caso se puede dar cuando no se pueden especificar todas las propiedades de un recurso hasta que se creen otros recursos dependientes. Por ejemplo, si crea un grupo de back-end para un equilibrador de carga, puede actualizar los adaptadores de red en sus máquinas virtuales (VM) para incluirlos en el grupo de back-end. Resource Manager permite actualizar los recursos durante la implementación, pero debe diseñar su plantilla correctamente para evitar errores y garantizar que la implementación se trate como una actualización.

En primer lugar, debe hacer referencia al recurso una vez en la plantilla para crearlo y, después, debe hacer referencia al recurso con el mismo nombre para actualizarlo más adelante. Sin embargo, si dos recursos tienen el mismo nombre en una plantilla, Resource Manager genera una excepción. Para evitar este error, especifique el recurso actualizado en una segunda plantilla que haya vinculado o incluido como una subplantilla mediante el tipo de recurso `Microsoft.Resources/deployments`.

En segundo lugar, en la plantilla anidada, debe especificar el nombre de la propiedad existente que va a cambiar o proporcionar un nombre nuevo para una propiedad que se va a agregar. También debe especificar las propiedades originales y sus valores originales. Si no se proporcionan las propiedades y los valores originales, Resource Manager da por supuesto que desea crear un nuevo recurso y elimina el original.

## <a name="example-template"></a>Plantilla de ejemplo

Echemos un vistazo a una plantilla de ejemplo que muestra cómo hacerlo. Nuestra plantilla implementa una red virtual llamada `firstVNet`, que tiene una subred llamada `firstSubnet`. Después, implementa una interfaz de red virtual llamada `nic1` y la asocia a la subred. A continuación, un recurso de implementación llamado `updateVNet` incluye una plantilla anidada que actualiza nuestro recurso `firstVNet` y agrega una segunda subred llamada `secondSubnet`. 

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {},
  "resources": [
      {
      "apiVersion": "2016-03-30",
      "name": "firstVNet",
      "location":"[resourceGroup().location]",
      "type": "Microsoft.Network/virtualNetworks",
      "properties": {
          "addressSpace":{"addressPrefixes": [
              "10.0.0.0/22"
          ]},
          "subnets":[              
              {
                  "name":"firstSubnet",
                  "properties":{
                    "addressPrefix":"10.0.0.0/24"
                  }
              }
            ]
      }
    },
    {
        "apiVersion": "2015-06-15",
        "type":"Microsoft.Network/networkInterfaces",
        "name":"nic1",
        "location":"[resourceGroup().location]",
        "dependsOn": [
            "firstVNet"
        ],
        "properties": {
            "ipConfigurations":[
                {
                    "name":"ipconfig1",
                    "properties": {
                        "privateIPAllocationMethod":"Dynamic",
                        "subnet": {
                            "id": "[concat(resourceId('Microsoft.Network/virtualNetworks','firstVNet'),'/subnets/firstSubnet')]"
                        }
                    }
                }
            ]
        }
    },
    {
      "apiVersion": "2015-01-01",
      "type": "Microsoft.Resources/deployments",
      "name": "updateVNet",
      "dependsOn": [
          "nic1"
      ],
      "properties": {
        "mode": "Incremental",
        "parameters": {},
        "template": {
          "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {},
          "variables": {},
          "resources": [
              {
                  "apiVersion": "2016-03-30",
                  "name": "firstVNet",
                  "location":"[resourceGroup().location]",
                  "type": "Microsoft.Network/virtualNetworks",
                  "properties": {
                      "addressSpace": "[reference('firstVNet').addressSpace]",
                      "subnets":[
                          {
                              "name":"[reference('firstVNet').subnets[0].name]",
                              "properties":{
                                  "addressPrefix":"[reference('firstVNet').subnets[0].properties.addressPrefix]"
                                  }
                          },
                          {
                              "name":"secondSubnet",
                              "properties":{
                                  "addressPrefix":"10.0.1.0/24"
                                  }
                          }
                     ]
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

Veamos primero el objeto de recurso de nuestro recurso `firstVNet`. Observe que volvemos a especificar la configuración de nuestro recurso `firstVNet` en una plantilla anidada. Esto es porque Resource Manager no permite el mismo nombre de implementación en la misma plantilla, y las plantillas anidadas se consideran plantillas diferentes. Al volver a especificar los valores de nuestro recurso `firstSubnet`, estamos indicando a Resource Manager que actualice el recurso existente en lugar de eliminarlo y volver a implementarlo. Por último, se recoge la nueva configuración de `secondSubnet` durante esta actualización.

## <a name="try-the-template"></a>Prueba de la plantilla

Si desea experimentar con esta plantilla, siga estos pasos:

1.  Vaya a Azure Portal, seleccione el icono **+** y busque el tipo de recurso **implementación de plantilla**.
2.  Vaya a la página **Implementación de plantilla** y seleccione el botón **crear**. Este botón abre la hoja de **Implementación personalizada**.
3.  Seleccione el icono **Editar**.
4.  Elimine la plantilla vacía.
5.  Copie y pegue la plantilla de ejemplo en el panel derecho.
6.  Seleccione el botón **Guardar**.
7.  Volverá al panel **Implementación personalizada**, pero esta vez hay algunos cuadros desplegables. Seleccione la suscripción, cree o use un grupo de recursos existente y seleccione una ubicación. Revise los términos y condiciones y seleccione el botón **Acepto**.
8.  Seleccione el botón **Comprar**.

Cuando haya finalizado la implementación, abra el grupo de recursos especificado en el portal. Verá una red virtual llamada `firstVNet` y una NIC llamada `nic1`. Haga clic en `firstVNet` y luego en `subnets`. Verá el `firstSubnet` que se creó originalmente, y verá el `secondSubnet` que se agregó en el recurso `updateVNet`. 

![Subred original y subred actualizada](../_images/firstVNet-subnets.png)

A continuación, vuelva al grupo de recursos y haga clic en `nic1`; a continuación, haga clic en `IP configurations`. En la sección `IP configurations`, `subnet` se configura en `firstSubnet (10.0.0.0/24)`. 

![Parámetros de configuración de IP del adaptador de red 1](../_images/nic1-ipconfigurations.png)

El `firstVNet` original se ha actualizado, en lugar de haberse creado de nuevo. Si `firstVNet` se hubiera creado de nuevo, `nic1` no se asociaría con `firstVNet`.

## <a name="next-steps"></a>Pasos siguientes

* Esta técnica se implementa en el [proyecto de bloques de creación de plantillas](https://github.com/mspnp/template-building-blocks) y las [arquitecturas de referencia de Azure](/azure/architecture/reference-architectures/). Puede usarlas para crear su propia arquitectura o implementar una de nuestras arquitecturas de referencia.
