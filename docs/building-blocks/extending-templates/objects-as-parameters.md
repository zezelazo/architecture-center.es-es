---
title: Uso de un objeto como parámetro en una plantilla de Azure Resource Manager
description: Se describe cómo ampliar la funcionalidad de las plantillas de Azure Resource Manager para utilizar objetos como parámetros.
author: petertay
ms.date: 10/30/2018
ms.openlocfilehash: c1955823b3474efa0abea1d9634add5f13d02eda
ms.sourcegitcommit: e9eb2b895037da0633ef3ccebdea2fcce047620f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/30/2018
ms.locfileid: "50251896"
---
# <a name="use-an-object-as-a-parameter-in-an-azure-resource-manager-template"></a><span data-ttu-id="ac037-103">Uso de un objeto como parámetro en una plantilla de Azure Resource Manager</span><span class="sxs-lookup"><span data-stu-id="ac037-103">Use an object as a parameter in an Azure Resource Manager template</span></span>

<span data-ttu-id="ac037-104">Cuando se [crean plantillas de Azure Resource Manager][azure-resource-manager-create-template], los valores de las propiedades de los recursos se pueden especificar directamente en la plantilla, o definir un parámetro y proporcionar los valores durante la implementación.</span><span class="sxs-lookup"><span data-stu-id="ac037-104">When you [author Azure Resource Manager templates][azure-resource-manager-create-template], you can either specify resource property values directly in the template or define a parameter and provide values during deployment.</span></span> <span data-ttu-id="ac037-105">Es correcto usar un parámetro para cada valor de propiedad en las implementaciones pequeñas, pero hay un límite de 255 parámetros por implementación.</span><span class="sxs-lookup"><span data-stu-id="ac037-105">It's fine to use a parameter for each property value for small deployments, but there is a limit of 255 parameters per deployment.</span></span> <span data-ttu-id="ac037-106">En las implementaciones más grandes y complejas, podría quedarse sin parámetros.</span><span class="sxs-lookup"><span data-stu-id="ac037-106">Once you get to larger and more complex deployments you may run out of parameters.</span></span>

<span data-ttu-id="ac037-107">Una manera de solucionar este problema es usar un objeto como parámetro en lugar de un valor.</span><span class="sxs-lookup"><span data-stu-id="ac037-107">One way to solve this problem is to use an object as a parameter instead of a value.</span></span> <span data-ttu-id="ac037-108">Para ello, defina el parámetro en la plantilla y especifique un objeto JSON en lugar de un valor único durante la implementación.</span><span class="sxs-lookup"><span data-stu-id="ac037-108">To do this, define the parameter in your template and specify a JSON object instead of a single value during deployment.</span></span> <span data-ttu-id="ac037-109">Después, haga referencia a las subpropiedades del parámetro con la [función `parameter()`][azure-resource-manager-functions] y un operador punto en la plantilla.</span><span class="sxs-lookup"><span data-stu-id="ac037-109">Then, reference the subproperties of the parameter using the [`parameter()` function][azure-resource-manager-functions] and dot operator in your template.</span></span>

<span data-ttu-id="ac037-110">Veamos un ejemplo que implementa un recurso de red virtual.</span><span class="sxs-lookup"><span data-stu-id="ac037-110">Let's take a look at an example that deploys a virtual network resource.</span></span> <span data-ttu-id="ac037-111">En primer lugar, vamos a especificar un parámetro `VNetSettings` en la plantilla y a establecer `type` en `object`:</span><span class="sxs-lookup"><span data-stu-id="ac037-111">First, let's specify a `VNetSettings` parameter in our template and set the `type` to `object`:</span></span>

```json
...
"parameters": {
    "VNetSettings":{"type":"object"}
},
```
<span data-ttu-id="ac037-112">Después, vamos a proporcionar los valores para el objeto `VNetSettings`:</span><span class="sxs-lookup"><span data-stu-id="ac037-112">Next, let's provide values for the `VNetSettings` object:</span></span>

> [!NOTE]
> <span data-ttu-id="ac037-113">Para ver cómo proporcionar los valores de los parámetros durante la implementación, consulte la sección **parámetros** del tema [Nociones sobre la estructura y la sintaxis de las plantillas de Azure Resource Manager][azure-resource-manager-authoring-templates].</span><span class="sxs-lookup"><span data-stu-id="ac037-113">To learn how to provide parameter values during deployment, see the **parameters** section of [understand the structure and syntax of Azure Resource Manager templates][azure-resource-manager-authoring-templates].</span></span> 

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

<span data-ttu-id="ac037-114">Como puede ver, nuestro único parámetro especifica tres subpropiedades: `name`, `addressPrefixes` y `subnets`.</span><span class="sxs-lookup"><span data-stu-id="ac037-114">As you can see, our single parameter actually specifies three subproperties: `name`, `addressPrefixes`, and `subnets`.</span></span> <span data-ttu-id="ac037-115">Cada una de estas subpropiedades especifica un valor u otras subpropiedades.</span><span class="sxs-lookup"><span data-stu-id="ac037-115">Each of these subproperties either specifies a value or other subproperties.</span></span> <span data-ttu-id="ac037-116">El resultado es que el único parámetro especifica todos los valores necesarios para implementar la red virtual.</span><span class="sxs-lookup"><span data-stu-id="ac037-116">The result is that our single parameter specifies all the values necessary to deploy our virtual network.</span></span>

<span data-ttu-id="ac037-117">Ahora, echemos un vistazo el resto de la plantilla para ver cómo se usa el objeto `VNetSettings`:</span><span class="sxs-lookup"><span data-stu-id="ac037-117">Now let's have a look at the rest of our template to see how the `VNetSettings` object is used:</span></span>

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
<span data-ttu-id="ac037-118">Los valores de nuestro objeto `VNetSettings` se aplican a las propiedades requeridas por el recurso de red virtual, usando la función `parameters()` con el indexador de matriz `[]` y el operador punto.</span><span class="sxs-lookup"><span data-stu-id="ac037-118">The values of our `VNetSettings` object are applied to the properties required by our virtual network resource using the `parameters()` function with both the `[]` array indexer and the dot operator.</span></span> <span data-ttu-id="ac037-119">Esta técnica funciona si desea aplicar estáticamente los valores del objeto de parámetro al recurso.</span><span class="sxs-lookup"><span data-stu-id="ac037-119">This approach works if you just want to statically apply the values of the parameter object to the resource.</span></span> <span data-ttu-id="ac037-120">Sin embargo, si desea asignar una matriz de valores de propiedad dinámicamente durante la implementación, puede usar un [bucle de copia][azure-resource-manager-create-multiple-instances].</span><span class="sxs-lookup"><span data-stu-id="ac037-120">However, if you want to dynamically assign an array of property values during deployment you can use a [copy loop][azure-resource-manager-create-multiple-instances].</span></span> <span data-ttu-id="ac037-121">Para usar un bucle de copia, proporcione una matriz JSON de valores de propiedad de recurso, y el bucle de copia aplicará dinámicamente los valores a las propiedades del recurso.</span><span class="sxs-lookup"><span data-stu-id="ac037-121">To use a copy loop, you provide a JSON array of resource property values and the copy loop dynamically applies the values to the resource's properties.</span></span> 

<span data-ttu-id="ac037-122">Hay un problema que hay que tener en cuenta si usa el método dinámico.</span><span class="sxs-lookup"><span data-stu-id="ac037-122">There is one issue to be aware of if you use the dynamic approach.</span></span> <span data-ttu-id="ac037-123">Para ver este problema en acción, echemos un vistazo a una matriz de valores de propiedad típica.</span><span class="sxs-lookup"><span data-stu-id="ac037-123">To demonstrate the issue, let's take a look at a typical array of property values.</span></span> <span data-ttu-id="ac037-124">En este ejemplo, los valores de nuestras propiedades se almacenan en una variable.</span><span class="sxs-lookup"><span data-stu-id="ac037-124">In this example the values for our properties are stored in a variable.</span></span> <span data-ttu-id="ac037-125">Observe que tenemos dos matrices aquí: una llamada `firstProperty` y otra llamada `secondProperty`.</span><span class="sxs-lookup"><span data-stu-id="ac037-125">Notice we have two arrays here&mdash;one named `firstProperty` and one named `secondProperty`.</span></span> 

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

<span data-ttu-id="ac037-126">Ahora, veamos cómo accedemos a las propiedades en la variable mediante un bucle de copia.</span><span class="sxs-lookup"><span data-stu-id="ac037-126">Now let's take a look at the way we access the properties in the variable using a copy loop.</span></span>

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

<span data-ttu-id="ac037-127">La función `copyIndex()` devuelve la iteración actual del bucle de copia, que usamos como índice en cada una de las dos matrices simultáneamente.</span><span class="sxs-lookup"><span data-stu-id="ac037-127">The `copyIndex()` function returns the current iteration of the copy loop, and we use that as an index into each of the two arrays simultaneously.</span></span>

<span data-ttu-id="ac037-128">Esto funciona bien cuando las dos matrices tienen la misma longitud.</span><span class="sxs-lookup"><span data-stu-id="ac037-128">This works fine when the two arrays are the same length.</span></span> <span data-ttu-id="ac037-129">El problema surge si ha cometido un error y las dos matrices tienen distintas longitudes. En este caso, la plantilla producirá un error de validación durante la implementación.</span><span class="sxs-lookup"><span data-stu-id="ac037-129">The issue arises if you've made a mistake and the two arrays are different lengths&mdash;in this case your template will fail validation during deployment.</span></span> <span data-ttu-id="ac037-130">Para evitar este problema, incluya todas las propiedades en un único objeto, porque así es mucho más fácil ver cuándo falta un valor.</span><span class="sxs-lookup"><span data-stu-id="ac037-130">You can avoid this issue by including all your properties in a single object, because it is much easier to see when a value is missing.</span></span> <span data-ttu-id="ac037-131">Por ejemplo, veamos otro objeto de parámetro en el que cada elemento de la matriz `propertyObject` es la unión de las matrices `firstProperty` y `secondProperty` de antes.</span><span class="sxs-lookup"><span data-stu-id="ac037-131">For example, let's take a look another parameter object in which each element of the `propertyObject` array is the union of the `firstProperty` and `secondProperty` arrays from earlier.</span></span>

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

<span data-ttu-id="ac037-132">¿Ve el tercer elemento de la matriz?</span><span class="sxs-lookup"><span data-stu-id="ac037-132">Notice the third element in the array?</span></span> <span data-ttu-id="ac037-133">Falta la propiedad `number`, pero es mucho más fácil darse cuenta de que falta cuando los valores de parámetro se crean de esta manera.</span><span class="sxs-lookup"><span data-stu-id="ac037-133">It's missing the `number` property, but it's much easier to notice that you've missed it when you're authoring the parameter values this way.</span></span>

## <a name="using-a-property-object-in-a-copy-loop"></a><span data-ttu-id="ac037-134">Uso de un objeto de propiedad en un bucle de copia</span><span class="sxs-lookup"><span data-stu-id="ac037-134">Using a property object in a copy loop</span></span>

<span data-ttu-id="ac037-135">Esta técnica resulta aún más útil cuando se combina con [serial copy loop][azure-resource-manager-create-multiple], especialmente para implementar recursos secundarios.</span><span class="sxs-lookup"><span data-stu-id="ac037-135">This approach becomes even more useful when combined with the [serial copy loop][azure-resource-manager-create-multiple], particularly for deploying child resources.</span></span> 

<span data-ttu-id="ac037-136">Para demostrarlo, vamos una plantilla que implementa un [grupo de seguridad de red (NSG)][nsg] con dos reglas de seguridad.</span><span class="sxs-lookup"><span data-stu-id="ac037-136">To demonstrate this, let's look at a template that deploys a [network security group (NSG)][nsg] with two security rules.</span></span> 

<span data-ttu-id="ac037-137">Primero, echemos un vistazo a nuestros parámetros.</span><span class="sxs-lookup"><span data-stu-id="ac037-137">First, let's take a look at our parameters.</span></span> <span data-ttu-id="ac037-138">Si observamos nuestra plantilla, veremos que hemos definido un parámetro llamado `networkSecurityGroupsSettings`, que incluye una matriz llamada `securityRules`.</span><span class="sxs-lookup"><span data-stu-id="ac037-138">When we look at our template we'll see that we've defined one parameter named `networkSecurityGroupsSettings` that includes an array named `securityRules`.</span></span> <span data-ttu-id="ac037-139">Esta matriz contiene dos objetos JSON que especifican una serie de valores para una regla de seguridad.</span><span class="sxs-lookup"><span data-stu-id="ac037-139">This array contains two JSON objects that specify a number of settings for a security rule.</span></span>

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

<span data-ttu-id="ac037-140">Ahora, echemos un vistazo a la plantilla.</span><span class="sxs-lookup"><span data-stu-id="ac037-140">Now let's take a look at our template.</span></span> <span data-ttu-id="ac037-141">El primer recurso, llamado `NSG1`, implementa el grupo de seguridad de red.</span><span class="sxs-lookup"><span data-stu-id="ac037-141">Our first resource named `NSG1` deploys the NSG.</span></span> <span data-ttu-id="ac037-142">El segundo recurso, llamado `loop-0`, realiza dos funciones: en primer lugar, `dependsOn` indica que depende del grupo de seguridad de red, de manera que su implementación no comenzará hasta que `NSG1` se haya completado, y es la primera iteración del bucle secuencial.</span><span class="sxs-lookup"><span data-stu-id="ac037-142">Our second resource named `loop-0` performs two functions: first, it `dependsOn` the NSG so its deployment doesn't begin until `NSG1` is completed, and it is the first iteration of the sequential loop.</span></span> <span data-ttu-id="ac037-143">El tercer recurso es una plantilla anidada que implementa las reglas de seguridad utilizando un objeto para sus valores de parámetro, como en el ejemplo anterior.</span><span class="sxs-lookup"><span data-stu-id="ac037-143">Our third resource is a nested template that deploys our security rules using an object for its parameter values as in the last example.</span></span>

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

<span data-ttu-id="ac037-144">Vamos a echar un vistazo más de cerca a cómo se especifican los valores de propiedad en el recurso secundario `securityRules`.</span><span class="sxs-lookup"><span data-stu-id="ac037-144">Let's take a closer look at how we specify our property values in the `securityRules` child resource.</span></span> <span data-ttu-id="ac037-145">Para hacer referencia a todas las propiedades, se usa la función `parameter()` y, a continuación, se usa el operador punto para hacer referencia a la matriz `securityRules`, indexada por el valor actual de la iteración.</span><span class="sxs-lookup"><span data-stu-id="ac037-145">All of our properties are referenced using the `parameter()` function, and then we use the dot operator to reference our `securityRules` array, indexed by the current value of the iteration.</span></span> <span data-ttu-id="ac037-146">Por último, se usa otro operador punto para hacer referencia al nombre del objeto.</span><span class="sxs-lookup"><span data-stu-id="ac037-146">Finally, we use another dot operator to reference the name of the object.</span></span> 

## <a name="try-the-template"></a><span data-ttu-id="ac037-147">Prueba de la plantilla</span><span class="sxs-lookup"><span data-stu-id="ac037-147">Try the template</span></span>

<span data-ttu-id="ac037-148">[GitHub][github] tiene una plantilla de ejemplo a su disposición.</span><span class="sxs-lookup"><span data-stu-id="ac037-148">An example template is available on [GitHub][github].</span></span> <span data-ttu-id="ac037-149">Para implementar la plantilla, clone el repositorio y ejecute los siguientes comandos de la [CLI de Azure][cli]:</span><span class="sxs-lookup"><span data-stu-id="ac037-149">To deploy the template, clone the repo and run the following [Azure CLI][cli] commands:</span></span>

```bash
git clone https://github.com/mspnp/template-examples.git
cd template-examples/example3-object-param
az group create --location <location> --name <resource-group-name>
az group deployment create -g <resource-group-name> \
    --template-uri https://raw.githubusercontent.com/mspnp/template-examples/master/example3-object-param/deploy.json \
    --parameters deploy.parameters.json
```

## <a name="next-steps"></a><span data-ttu-id="ac037-150">Pasos siguientes</span><span class="sxs-lookup"><span data-stu-id="ac037-150">Next steps</span></span>

- <span data-ttu-id="ac037-151">Aprenda a crear una plantilla que recorre en iteración una matriz de objetos y los transforma en un esquema JSON.</span><span class="sxs-lookup"><span data-stu-id="ac037-151">Learn how to create a template that iterates through an object array and transforms it into a JSON schema.</span></span> <span data-ttu-id="ac037-152">Consulte [Implementación de un recopilador y transformador de propiedades en una plantilla de Azure Resource Manager](./collector.md)</span><span class="sxs-lookup"><span data-stu-id="ac037-152">See [Implement a property transformer and collector in an Azure Resource Manager template](./collector.md)</span></span>


<!-- links -->
[azure-resource-manager-authoring-templates]: /azure/azure-resource-manager/resource-group-authoring-templates
[azure-resource-manager-create-template]: /azure/azure-resource-manager/resource-manager-create-first-template
[azure-resource-manager-create-multiple-instances]: /azure/azure-resource-manager/resource-group-create-multiple
[azure-resource-manager-functions]: /azure/azure-resource-manager/resource-group-template-functions-resource
[nsg]: /azure/virtual-network/virtual-networks-nsg
[cli]: /cli/azure/?view=azure-cli-latest
[github]: https://github.com/mspnp/template-examples
