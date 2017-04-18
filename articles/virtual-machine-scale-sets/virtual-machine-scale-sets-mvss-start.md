<properties
    pageTitle="Azure 虚拟机规模集：最小的可行规模集 | Azure"
    description="了解如何创建最小的可行规模集模板"
    services="virtual-machine-scale-sets"
    documentationcenter=""
    author="gatneil"
    manager="timlt"
    editor=""
    tags="azure-resource-manager"
    translationtype="Human Translation" />
<tags
    ms.assetid="76ac7fd7-2e05-4762-88ca-3b499e87906e"
    ms.service="virtual-machine-scale-sets"
    ms.workload="na"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="2/14/2017"
    wacn.date="04/17/2017"
    ms.author="negat"
    ms.sourcegitcommit="e0e6e13098e42358a7eaf3a810930af750e724dd"
    ms.openlocfilehash="551c7174ba120616922a9513c3e7645fdc73c2d9"
    ms.lasthandoff="04/06/2017" />

# <a name="about-this-tutorial"></a>关于本教程

[Azure Resource Manager 模板](/documentation/articles/resource-group-overview/#template-deployment)是部署成组的相关资源的好办法。 本系列教程演示如何创建最小的可行规模集模板，以及如何修改此模板以满足各种场景。 所有示例都来自此 [github 存储库](https://github.com/gatneil/mvss)。 本演练中所示的每个差异都是在此存储库中的分支之间执行 `git diff` 的结果。 这些模板和差异可作为简单但不完整的示例。 有关更完整的规模集模板的示例，请参阅 [Azure 快速入门模板 github 存储库](https://github.com/Azure/azure-quickstart-templates)，并搜索包含字符串 `vmss` 的文件夹。

## <a name="a-minimum-viable-scale-set"></a>最小的可行规模集

可在[此处](https://raw.githubusercontent.com/gatneil/mvss/minimum-viable-scale-set/azuredeploy.json)查看最小的可行规模集模板。 如果你已熟悉模板，则可以放心跳到“后续步骤”部分，以查看如何针对其他场景修改此模板。 但是，如果你不太熟悉模板，则会发现此逐条说明很有用。 开始之前，让我们检查差异，以便逐步创建模板 (`git diff master minimum-viable-scale-set`)：

首先，定义模板的 `$schema` 和 `contentVersion`。 `$schema` 定义模板语言的版本，并用于 Visual Studio 语法突出显示和类似的验证功能。 Azure 实际上不使用 `contentVersion`。 此项旨在帮助跟踪模板的版本。

    +{
    +  "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json",
    +  "contentVersion": "1.0.0.0",

接下来，定义两个参数，`adminUsername` 和 `adminPassword`。 参数为部署时由用户指定的值。 `adminUsername` 只是一个 `string`，但是 `adminPassword` 是一个密码，因此将其类型指定为 `securestring`。 稍后我们将看到这些参数传递到规模集配置。

    +  "parameters": {
    +    "adminUsername": {
    +      "type": "string"
    +    },
    +    "adminPassword": {
    +      "type": "securestring"
    +    }
    +  },

Resource Manager 模板还可用于定义以后要在模板中使用的变量。 在本示例中我们不使用任何变量，因此将 JSON 对象保留为空。

    +  "variables": {},

接着创建模板的资源，在模板中定义实际要部署的项目。 与 `parameters` 和 `variables`（它们是 JSON 对象）不同，`resources` 是 JSON 对象的 JSON 列表。

    +  "resources": [

所有资源都需要 `type`、`name`、`apiVersion` 和 `location`。 第一个资源的类型为 `Microsft.Network/virtualNetwork`，名称为 `myVnet`，apiVersion 为 `2016-03-30`。 若要找出资源类型的最新 api 版本，请参阅 [Azure REST API 文档](https://docs.microsoft.com/zh-cn/rest/api/)。

    +    {
    +      "type": "Microsoft.Network/virtualNetworks",
    +      "name": "myVnet",
    +      "apiVersion": "2016-12-01",

若要指定虚拟网络的位置，则使用 [Resource Manager 模板函数](/documentation/articles/resource-group-template-functions/)，该函数必须使用引号和方括号括起来，如下所示：`"[<template-function>]"`。 在本示例中，我们使用 resourceGroup 函数，该函数不使用任何参数，并返回 JSON 对象和有关要将部署部署到的资源组的元数据。 资源组在部署时由用户进行设置。 然后使用 `.location` 为此 JSON 对象建立索引，以便从此 JSON 对象中获取位置。

    +      "location": "[resourceGroup().location]",

每个 Resource Manager 资源都有自己的针对特定于的资源的配置的 `properties` 部分。 在本示例中，我们指定虚拟网络应具有一个子网，并使用专用的 IP 地址范围 `10.0.0.0/16`。 规模集始终包含在一个子网中。 它不能跨子网。

    +      "properties": {
    +        "addressSpace": {
    +          "addressPrefixes": [
    +            "10.0.0.0/16"
    +          ]
    +        },
    +        "subnets": [
    +          {
    +            "name": "mySubnet",
    +            "properties": {
    +              "addressPrefix": "10.0.0.0/16"
    +            }
    +          }
    +        ]
    +      }
    +    },

除了所需的 `type`、`name`、`apiVersion` 和 `location` 属性，每个资源还可以具有字符串的 `dependsOn` 列表，该列表指定在部署此资源之前必须完成部署哪些其他资源。 在本示例中，列表中只有一个元素，即前面提到的虚拟网络。 指定此依赖项的原因是在创建任何虚拟机之前规模集需要有网络。 这样，规模集可为这些虚拟机提供之前在网络属性中指定的 IP 地址范围中的专用 IP 地址。 dependsOn 列表中的每个字符串的格式为 `<type>/<name>`（`type` 和 `name` 与之前在虚拟网络资源定义中的相同）。

    +    {
    +      "type": "Microsoft.Compute/virtualMachineScaleSets",
    +      "name": "myScaleSet",
    +      "apiVersion": "2016-04-30-preview",
    +      "location": "[resourceGroup().location]",
    +      "dependsOn": [
    +        "Microsoft.Network/virtualNetworks/myVnet"
    +      ],

规模集需要知道要创建的 VM 的大小（“sku name”）和要创建多少个这样的 VM（“sku capacity”）。 若要查看可用的 VM 大小，请参阅 [VM 大小文档](/documentation/articles/virtual-machines-windows-sizes/)。

    +      "sku": {
    +        "name": "Standard_A1",
    +        "capacity": 2
    +      },

规模集还需要知道如何处理规模集的更新。 当前有两个选项，`Manual` 和 `Automatic`。 有关这两者之间的区别的详细信息，请参阅文档[如何升级规模集](/documentation/articles/virtual-machine-scale-sets-upgrade-scale-set/)。

    +      "properties": {
    +        "upgradePolicy": {
    +          "mode": "Manual"
    +        },

规模集还需要知道要在虚拟机上安装什么操作系统。 此处我们使用完全修补的 Ubuntu 16.04-LTS 映像创建虚拟机。

    +        "virtualMachineProfile": {
    +          "storageProfile": {
    +            "imageReference": {
    +              "publisher": "Canonical",
    +              "offer": "UbuntuServer",
    +              "sku": "16.04-LTS",
    +              "version": "latest"
    +            }
    +          },

既然规模集要部署多个 VM，我们指定 `computerNamePrefix`，而不是指定每个 VM 的名称。 规模集在每个 VM 的前缀中附加一个索引，因此 VM 名称格式为 `<computerNamePrefix>_<auto-generated-index>`。 在此代码段中，我们还可以看到使用前面提到的参数为规模集中所有 VM 设置管理员用户名和密码。 我们使用 `parameters` 模板函数来实现此操作，该函数使用一个字符串指定想要引用的参数，并输出该参数的值。

    +          "osProfile": {
    +            "computerNamePrefix": "vm",
    +            "adminUsername": "[parameters('adminUsername')]",
    +            "adminPassword": "[parameters('adminPassword')]"
    +          },

最后，需要指定规模集中 VM 的网络配置。 在本示例中，我们只需指定之前创建的子网的 id，以便规模集知道将网络接口放在此子网中。 可以使用 `resourceId` 模板函数获取包含子网的虚拟网络的 id。 此函数采用资源的类型和名称，并返回该资源的完全限定的标识符（此 id 的格式为：`/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/<resourceProviderNamespace>/<resourceType>/<resourceName>`）。 但是，仅有虚拟网络的标识符是不够的。 必须指定规模集的 VM 所在的特定子网，因此将 `/subnets/mySubnet` 串联到虚拟网络的 id。 其结果是子网的完全限定的 id。 我们使用 `concat` 函数执行此串联，该函数使用一系列字符串并返回它们的串联字符串。

    +          "networkProfile": {
    +            "networkInterfaceConfigurations": [
    +              {
    +                "name": "myNic",
    +                "properties": {
    +                  "primary": "true",
    +                  "ipConfigurations": [
    +                    {
    +                      "name": "myIpConfig",
    +                      "properties": {
    +                        "subnet": {
    +                          "id": "[concat(resourceId('Microsoft.Network/virtualNetworks', 'myVnet'), '/subnets/mySubnet')]"
    +                        }
    +                      }
    +                    }
    +                  ]
    +                }
    +              }
    +            ]
    +          }
    +        }
    +      }
    +    }
    +  ]
    +}


## <a name="next-steps"></a>后续步骤

[AZURE.INCLUDE [mvss-next-steps-include](../../includes/mvss-next-steps.md)]