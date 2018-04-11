---
title: Implementación condicional de un recurso en una plantilla de Azure Resource Manager
description: Describe cómo ampliar la funcionalidad de las plantillas de Azure Resource Manager para implementar condicionalmente un recurso en función del valor de un parámetro
author: petertay
ms.date: 06/09/2017
ms.openlocfilehash: e911e7dc41b4f71ebfaf13a00f8cdbb5b4e2578b
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2017
---
# <a name="conditionally-deploy-a-resource-in-an-azure-resource-manager-template"></a><span data-ttu-id="e5323-103">Implementación condicional de un recurso en una plantilla de Azure Resource Manager</span><span class="sxs-lookup"><span data-stu-id="e5323-103">Conditionally deploy a resource in an Azure Resource Manager template</span></span>

<span data-ttu-id="e5323-104">En algunos escenarios, es necesario diseñar la plantilla para implementar un recurso en función de una condición, por ejemplo, si un valor de parámetro está presente o no.</span><span class="sxs-lookup"><span data-stu-id="e5323-104">There are some scenarios in which you need to design your template to deploy a resource based on a condition, such as whether or not a parameter value is present.</span></span> <span data-ttu-id="e5323-105">Por ejemplo, la plantilla puede implementar una red virtual e incluir parámetros para especificar otras redes virtuales para el emparejamiento.</span><span class="sxs-lookup"><span data-stu-id="e5323-105">For example, your template may deploy a virtual network and include parameters to specify other virtual networks for peering.</span></span> <span data-ttu-id="e5323-106">Si no se han especificado los valores de parámetro para el emparejamiento, no desea que Resource Manager implemente el recurso de emparejamiento.</span><span class="sxs-lookup"><span data-stu-id="e5323-106">If you've not specified any parameter values for peering, you don't want Resource Manager to deploy the peering resource.</span></span>

<span data-ttu-id="e5323-107">Para ello, use el [elemento `condition`][azure-resource-manager-condition] en el recurso para probar la longitud de la matriz de parámetros.</span><span class="sxs-lookup"><span data-stu-id="e5323-107">To accomplish this, use the [`condition` element][azure-resource-manager-condition] in the resource to test the length of your parameter array.</span></span> <span data-ttu-id="e5323-108">Si la longitud es cero, devuelve `false` para evitar la implementación pero, para todos los valores mayores que cero, devuelve `true` para permitir la implementación.</span><span class="sxs-lookup"><span data-stu-id="e5323-108">If the length is zero, return `false` to prevent deployment, but for all values greater than zero return `true` to allow deployment.</span></span>

## <a name="example-template"></a><span data-ttu-id="e5323-109">Plantilla de ejemplo</span><span class="sxs-lookup"><span data-stu-id="e5323-109">Example template</span></span>

<span data-ttu-id="e5323-110">Echemos un vistazo a una plantilla de ejemplo que muestra cómo hacerlo.</span><span class="sxs-lookup"><span data-stu-id="e5323-110">Let's look at an example template that demonstrates this.</span></span> <span data-ttu-id="e5323-111">La plantilla usa el [elemento `condition`][azure-resource-manager-condition] para controlar la implementación del recurso `Microsoft.Network/virtualNetworks/virtualNetworkPeerings`.</span><span class="sxs-lookup"><span data-stu-id="e5323-111">Our template uses the [`condition` element][azure-resource-manager-condition] to control deployment of the `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` resource.</span></span> <span data-ttu-id="e5323-112">Este recurso crea un emparejamiento entre dos redes virtuales de Azure en la misma región.</span><span class="sxs-lookup"><span data-stu-id="e5323-112">This resource creates a peering between two Azure Virtual Networks in the same region.</span></span>

<span data-ttu-id="e5323-113">Veamos cada sección de la plantilla.</span><span class="sxs-lookup"><span data-stu-id="e5323-113">Let's take a look at each section of the template.</span></span>

<span data-ttu-id="e5323-114">El elemento `parameters` define un único parámetro llamado `virtualNetworkPeerings`:</span><span class="sxs-lookup"><span data-stu-id="e5323-114">The `parameters` element defines a single parameter named `virtualNetworkPeerings`:</span></span> 

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
<span data-ttu-id="e5323-115">Nuestro parámetro `virtualNetworkPeerings` es un valor de `array` y tiene el siguiente esquema:</span><span class="sxs-lookup"><span data-stu-id="e5323-115">Our `virtualNetworkPeerings` parameter is an `array` and has the following schema:</span></span>

```json
"virtualNetworkPeerings": [
    {
        "remoteVirtualNetwork": {
            "name": "my-other-virtual-network"
        },
        "allowForwardedTraffic": true,
        "allowGatewayTransit": true,
        "useRemoteGateways": false
    }
]
```

<span data-ttu-id="e5323-116">Las propiedades del parámetro especifican la [configuración relativa al emparejamiento de las redes virtuales][vnet-peering-resource-schema].</span><span class="sxs-lookup"><span data-stu-id="e5323-116">The properties in our parameter specify the [settings related to peering virtual networks][vnet-peering-resource-schema].</span></span> <span data-ttu-id="e5323-117">Proporcionaremos los valores de estas propiedades cuando especifiquemos el recurso `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` en la sección `resources`:</span><span class="sxs-lookup"><span data-stu-id="e5323-117">We'll provide the values for these properties when we specify the `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` resource in the `resources` section:</span></span>

```json
"resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2017-05-10",
      "name": "[concat('vnp-', copyIndex())]",
      "condition": "[greater(length(parameters('virtualNetworkPeerings')), 0)]",
      "dependsOn": [
        "virtualNetworks"
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
<span data-ttu-id="e5323-118">Hay un par de cosas en esta parte de la plantilla.</span><span class="sxs-lookup"><span data-stu-id="e5323-118">There are a couple of things going on in this part of our template.</span></span> <span data-ttu-id="e5323-119">En primer lugar, el recurso real que se va a implementar es una plantilla en línea del tipo `Microsoft.Resources/deployments`, que incluye su propia plantilla que implementa realmente `Microsoft.Network/virtualNetworks/virtualNetworkPeerings`.</span><span class="sxs-lookup"><span data-stu-id="e5323-119">First, the actual resource being deployed is an inline template of type `Microsoft.Resources/deployments` that includes its own template that actually deploys the `Microsoft.Network/virtualNetworks/virtualNetworkPeerings`.</span></span>

<span data-ttu-id="e5323-120">Para que el valor de `name` de la plantilla en línea sea único, se concatena la iteración actual de `copyIndex()` con el prefijo `vnp-`.</span><span class="sxs-lookup"><span data-stu-id="e5323-120">Our `name` for the inline template is made unique by concatenating the current iteration of the `copyIndex()` with the prefix `vnp-`.</span></span> 

<span data-ttu-id="e5323-121">El elemento `condition` especifica que el recurso debe procesarse cuando la función `greater()` se evalúe en `true`.</span><span class="sxs-lookup"><span data-stu-id="e5323-121">The `condition` element specifies that our resource should be processed when the `greater()` function evaluates to `true`.</span></span> <span data-ttu-id="e5323-122">En este caso, vamos a probar si la matriz de parámetros `virtualNetworkPeerings` es `greater()` que cero.</span><span class="sxs-lookup"><span data-stu-id="e5323-122">Here, we're testing if the `virtualNetworkPeerings` parameter array is `greater()` than zero.</span></span> <span data-ttu-id="e5323-123">Si lo es, se evalúa en `true` y se cumple `condition`.</span><span class="sxs-lookup"><span data-stu-id="e5323-123">If it is, it evaluates to `true` and the `condition` is satisfied.</span></span> <span data-ttu-id="e5323-124">De lo contrario, es `false`.</span><span class="sxs-lookup"><span data-stu-id="e5323-124">Otherwise, it's `false`.</span></span>

<span data-ttu-id="e5323-125">Después, especificamos el bucle `copy`.</span><span class="sxs-lookup"><span data-stu-id="e5323-125">Next, we specify our `copy` loop.</span></span> <span data-ttu-id="e5323-126">Es un bucle `serial`, lo que significa que el bucle se realiza secuencialmente; cada recurso debe esperar a que el recurso anterior se haya implementado.</span><span class="sxs-lookup"><span data-stu-id="e5323-126">It's a `serial` loop that means the loop is done in sequence, with each resource waiting until the last resource has been deployed.</span></span> <span data-ttu-id="e5323-127">La propiedad `count` especifica el número de veces que se recorre el bucle en iteración.</span><span class="sxs-lookup"><span data-stu-id="e5323-127">The `count` property specifies the number of times the loop iterates.</span></span> <span data-ttu-id="e5323-128">Normalmente, aquí se establecería en la longitud de la matriz `virtualNetworkPeerings` porque contiene los objetos de parámetro que especifican el recurso que queremos implementar.</span><span class="sxs-lookup"><span data-stu-id="e5323-128">Here, normally we'd set it to the length of the `virtualNetworkPeerings` array because it contains the parameter objects specifying the resource we want to deploy.</span></span> <span data-ttu-id="e5323-129">Sin embargo, si hacemos esto, se producirá un error de validación si la matriz está vacía porque Resource Manager se da cuenta de que estamos intentando acceder a propiedades que no existen.</span><span class="sxs-lookup"><span data-stu-id="e5323-129">However, if we do that, validation will fail if the array is empty because Resource Manager notices that we are attempting to access properties that do not exist.</span></span> <span data-ttu-id="e5323-130">Sin embargo, esto tiene solución.</span><span class="sxs-lookup"><span data-stu-id="e5323-130">We can work around this, however.</span></span> <span data-ttu-id="e5323-131">Veamos las variables que necesitamos:</span><span class="sxs-lookup"><span data-stu-id="e5323-131">Let's take a look at the variables we'll need:</span></span>

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

<span data-ttu-id="e5323-132">Nuestra variable `workaround` incluye dos propiedades, una llamada `true` y otra llamada `false`.</span><span class="sxs-lookup"><span data-stu-id="e5323-132">Our `workaround` variable includes two properties, one named `true` and one named `false`.</span></span> <span data-ttu-id="e5323-133">La propiedad `true` se evalúa en el valor de la matriz de parámetros `virtualNetworkPeerings`.</span><span class="sxs-lookup"><span data-stu-id="e5323-133">The `true` property evaluates to the value of the `virtualNetworkPeerings` parameter array.</span></span> <span data-ttu-id="e5323-134">La propiedad `false` se evalúa en un objeto vacío, incluidas las propiedades con nombre que Resource Manager espera ver. Tenga en cuenta que, en realidad, `false` es una matriz, al igual que el `virtualNetworkPeerings` parámetro, que cumplirá la validación.</span><span class="sxs-lookup"><span data-stu-id="e5323-134">The `false` property evaluates to an empty object including the named properties that Resource Manager expects to see&mdash;note that `false` is actually an array, just as our `virtualNetworkPeerings` parameter is, which will satisfy validation.</span></span> 

<span data-ttu-id="e5323-135">La variable `peerings` utiliza la variable `workaround` y prueba una vez más si la longitud de la matriz de parámetros `virtualNetworkPeerings` es mayor que cero.</span><span class="sxs-lookup"><span data-stu-id="e5323-135">Our `peerings` variable uses our `workaround` variable by once again testing if the length of the `virtualNetworkPeerings` parameter array is greater than zero.</span></span> <span data-ttu-id="e5323-136">Si es así, `string` se evalúa en `true` y la variable `workaround` se evalúa en la matriz de parámetros `virtualNetworkPeerings`.</span><span class="sxs-lookup"><span data-stu-id="e5323-136">If it is, the `string` evaluates to `true` and the `workaround` variable evalutes to the `virtualNetworkPeerings` parameter array.</span></span> <span data-ttu-id="e5323-137">En caso contrario, se evalúa en `false` y la variable `workaround` se evalúa en nuestro objeto vacío en el primer elemento de la matriz.</span><span class="sxs-lookup"><span data-stu-id="e5323-137">Otherwise, it evaluates to `false` and the `workaround` variable evaluates to our empty object in the first element of the array.</span></span>

<span data-ttu-id="e5323-138">Ahora que hemos solucionado el problema de validación, solo tenemos que especificar la implementación del recurso `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` en la plantilla anidada, pasando los valores de `name` y `properties` de nuestra matriz de parámetros `virtualNetworkPeerings`.</span><span class="sxs-lookup"><span data-stu-id="e5323-138">Now that we've worked around the validation issue, we can simply specify the deployment of the `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` resource in the nested template, passing the `name` and `properties` from our `virtualNetworkPeerings` parameter array.</span></span> <span data-ttu-id="e5323-139">Puede verlo en el elemento `template` anidado en el elemento `properties` de nuestro recurso.</span><span class="sxs-lookup"><span data-stu-id="e5323-139">You can see this in the `template` element nested in the `properties` element of our resource.</span></span>

## <a name="next-steps"></a><span data-ttu-id="e5323-140">Pasos siguientes</span><span class="sxs-lookup"><span data-stu-id="e5323-140">Next steps</span></span>

* <span data-ttu-id="e5323-141">Esta técnica se implementa en el [proyecto de bloques de creación de plantillas](https://github.com/mspnp/template-building-blocks) y las [arquitecturas de referencia de Azure](/azure/architecture/reference-architectures/).</span><span class="sxs-lookup"><span data-stu-id="e5323-141">This technique is implemented in the [template building blocks project](https://github.com/mspnp/template-building-blocks) and the [Azure reference architectures](/azure/architecture/reference-architectures/).</span></span> <span data-ttu-id="e5323-142">Puede usarlas para crear su propia arquitectura o implementar una de nuestras arquitecturas de referencia.</span><span class="sxs-lookup"><span data-stu-id="e5323-142">You can use these to create your own architecture or deploy one of our reference architectures.</span></span>

<!-- links -->
[azure-resource-manager-condition]: /azure/azure-resource-manager/resource-group-authoring-templates#resources
[azure-resource-manager-variable]: /azure/azure-resource-manager/resource-group-authoring-templates#variables
[vnet-peering-resource-schema]: /azure/templates/microsoft.network/virtualnetworks/virtualnetworkpeerings