<properties
    pageTitle="了解虚拟机规模集模板 | Azure"
    description="了解如何创建虚拟机规模集的最小可行规模集模板"
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
    wacn.date="05/02/2017"
    ms.author="negat"
    ms.sourcegitcommit="78da854d58905bc82228bcbff1de0fcfbc12d5ac"
    ms.openlocfilehash="479e96eee43284195d57845813095c63171ae38e"
    ms.lasthandoff="04/22/2017" />

# <a name="learn-about-virtual-machine-scale-set-templates"></a>了解虚拟机规模集模板
[Azure Resource Manager 模板](/documentation/articles/resource-group-overview/#template-deployment)是部署成组的相关资源的好办法。 本系列教程演示如何创建最小的可行规模集模板，以及如何修改此模板以满足各种场景。 所有示例都来自此 [GitHub 存储库](https://github.com/gatneil/mvss)。 

此模板简单易用。 有关更完整的规模集模板的示例，请参阅 [Azure 快速入门模板 GitHub 存储库](https://github.com/Azure/azure-quickstart-templates)，并搜索包含字符串 `vmss` 的文件夹。

如果你已能够熟练创建模板，可以跳到“后续步骤”部分，查看如何修改此模板。

## <a name="review-the-template"></a>查看模板

使用 GitHub 查看最小的可行规模集模板 [azuredeploy.json](https://raw.githubusercontent.com/gatneil/mvss/minimum-viable-scale-set/azuredeploy.json)。

本教程探讨如何使用 diff (`git diff master minimum-viable-scale-set`) 创建最小的可行规模集模板，内容细化到了每个代码片段。

## <a name="define-schema-and-contentversion"></a>定义 $schema 和 contentVersion
首先，在模板中定义 `$schema` 和 `contentVersion`。 `$schema` 元素定义模板语言的版本，用于 Visual Studio 语法突出显示和类似的验证功能。 Azure 不使用 `contentVersion` 元素。 但是，该元素可帮助你跟踪模板版本。

    {
      "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json",
      "contentVersion": "1.0.0.0",

## <a name="define-parameters"></a>定义参数
接下来，定义两个参数，`adminUsername` 和 `adminPassword`。 参数是部署时指定的值。 `adminUsername` 参数只是一个 `string` 类型，但由于 `adminPassword` 是机密，因此我们将其类型指定为 `securestring`。 稍后，要将这些参数传递到规模集配置。

      "parameters": {
        "adminUsername": {
          "type": "string"
        },
        "adminPassword": {
          "type": "securestring"
        }
      },

## <a name="define-variables"></a>定义变量
Resource Manager 模板还可用于定义稍后要在模板中使用的变量。 本示例未使用任何变量，因此我们将 JSON 对象保留为空。

      "variables": {},

## <a name="define-resources"></a>定义资源
接下来是模板的资源部分。 在此处定义实际要部署的内容。 与 `parameters` 和 `variables`（JSON 对象）不同，`resources` 是 JSON 对象的 JSON 列表。

       "resources": [

所有资源都需要 `type`、`name`、`apiVersion` 和 `location` 属性。 本示例的第一个资源的类型为 `Microsft.Network/virtualNetwork`，名称为 `myVnet`，apiVersion 为 `2016-03-30`。 （若要查找资源类型的最新 API 版本，请参阅 [Azure REST API 文档](https://docs.microsoft.com/zh-cn/rest/api/)。）

         {
           "type": "Microsoft.Network/virtualNetworks",
           "name": "myVnet",
           "apiVersion": "2016-12-01",

## <a name="specify-location"></a>指定位置
为了指定虚拟网络的位置，我们使用了 [Resource Manager 模板函数](/documentation/articles/resource-group-template-functions/)。 必须将此函数括在引号和方括号中，如下所示：`"[<template-function>]"`。 在本例中，我们使用了 `resourceGroup` 函数。 该函数不使用任何参数，返回 JSON 对象以及有关要将部署部署到的资源组的元数据。 资源组在部署时由用户进行设置。 然后使用 `.location` 为此 JSON 对象建立索引，以便从此 JSON 对象中获取位置。

           "location": "[resourceGroup().location]",

## <a name="specify-virtual-network-properties"></a>指定虚拟网络属性
每个 Resource Manager 资源都有自己的针对特定于的资源的配置的 `properties` 部分。 在本例中，我们指定虚拟网络应有一个使用专用 IP 地址范围 `10.0.0.0/16` 的子网。 规模集始终包含在一个子网中。 它不能跨子网。

           "properties": {
             "addressSpace": {
               "addressPrefixes": [
                 "10.0.0.0/16"
               ]
             },
             "subnets": [
               {
                 "name": "mySubnet",
                 "properties": {
                   "addressPrefix": "10.0.0.0/16"
                 }
               }
             ]
           }
         },

## <a name="add-dependson-list"></a>添加 dependsOn 列表
除了所需的 `type`、`name`、`apiVersion` 和 `location` 属性以外，每个资源还可以包含字符串的可选 `dependsOn` 列表。 此列表指定在部署此资源之前必须先完成此部署中的其他哪些资源。

在本例中，列表只包含一个元素，即上一示例中所述的虚拟网络。 指定此依赖项的原因是在创建任何虚拟机之前规模集需要有网络。 这样，规模集可为这些虚拟机提供前面在网络属性中指定的 IP 地址范围中的专用 IP 地址。 dependsOn 列表中每个字符串的格式为 `<type>/<name>`。 使用前面在虚拟网络资源定义中所用的 `type` 和 `name`。

         {
           "type": "Microsoft.Compute/virtualMachineScaleSets",
           "name": "myScaleSet",
           "apiVersion": "2016-04-30-preview",
           "location": "[resourceGroup().location]",
           "dependsOn": [
             "Microsoft.Network/virtualNetworks/myVnet"
           ],

## <a name="specify-scale-set-properties"></a>指定规模集属性
规模集提供许多用于在规模集中自定义 VM 的属性。 有关这些属性的完整列表，请参阅[规模集 REST API 文档](https://docs.microsoft.com/zh-cn/rest/api/virtualmachinescalesets/create-or-update-a-set)。 本教程只设置几个常用的属性。
### <a name="supply-vm-size-and-capacity"></a>提供 VM 大小和容量
规模集需要知道要创建的 VM 的大小（“sku name”）和要创建多少个这样的 VM（“sku capacity”）。 若要查看可用的 VM 大小，请参阅 [VM 大小文档](/documentation/articles/virtual-machines-windows-sizes/)。

           "sku": {
             "name": "Standard_A1",
             "capacity": 2
           },

### <a name="choose-type-of-updates"></a>选择更新类型
规模集还需要知道如何处理规模集的更新。 目前有两个选项：`Manual` 和 `Automatic`。 有关这两者之间的区别的详细信息，请参阅文档[如何升级规模集](/documentation/articles/virtual-machine-scale-sets-upgrade-scale-set/)。

           "properties": {
             "upgradePolicy": {
               "mode": "Manual"
             },

### <a name="choose-vm-operating-system"></a>选择 VM 操作系统
规模集需要知道要在虚拟机上安装什么操作系统。 此处我们使用完全修补的 Ubuntu 16.04-LTS 映像创建虚拟机。

             "virtualMachineProfile": {
               "storageProfile": {
                 "imageReference": {
                   "publisher": "Canonical",
                   "offer": "UbuntuServer",
                   "sku": "16.04-LTS",
                   "version": "latest"
                 }
               },

### <a name="specify-computernameprefix"></a>指定 computerNamePrefix
规模集部署多个 VM。 我们不要指定每个 VM 名称，而是指定 `computerNamePrefix`。 规模集在每个 VM 的前缀中附加一个索引，因此 VM 名称格式为 `<computerNamePrefix>_<auto-generated-index>`。

在以下代码片段中，我们使用前面提到的参数为规模集中所有 VM 设置管理员用户名和密码。 可以使用 `parameters` 模板函数实现此目的。 此函数将采用一个字符串来指定要引用的参数并输出该参数的值。

               "osProfile": {
                 "computerNamePrefix": "vm",
                 "adminUsername": "[parameters('adminUsername')]",
                 "adminPassword": "[parameters('adminPassword')]"
               },

### <a name="specify-vm-network-configuration"></a>指定 VM 网络配置
最后，需要指定规模集中 VM 的网络配置。 在本例中，我们只需指定前面创建的子网的 ID。 这样，规模集便知道要在此子网中放置网络接口。

可以使用 `resourceId` 模板函数获取包含该子网的虚拟网络的 ID。 此函数采用资源的类型和名称，并返回该资源的完全限定标识符。 此 ID 的格式为：`/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/<resourceProviderNamespace>/<resourceType>/<resourceName>`

但是，仅有虚拟网络的标识符是不够的。 必须指定规模集 VM 应该所在的特定子网。 为此，请将 `/subnets/mySubnet` 串联到虚拟网络的 ID。 结果是该子网的完全限定的 ID。 可以使用 `concat` 函数执行此串联，该函数采用一系列字符串并返回它们的串联字符串。

               "networkProfile": {
                 "networkInterfaceConfigurations": [
                   {
                     "name": "myNic",
                     "properties": {
                       "primary": "true",
                       "ipConfigurations": [
                         {
                           "name": "myIpConfig",
                           "properties": {
                             "subnet": {
                               "id": "[concat(resourceId('Microsoft.Network/virtualNetworks', 'myVnet'), '/subnets/mySubnet')]"
                             }
                           }
                         }
                       ]
                     }
                   }
                 ]
               }
             }
           }
         }
       ]
    }


## <a name="next-steps"></a>后续步骤

[AZURE.INCLUDE [mvss-next-steps-include](../../includes/mvss-next-steps.md)]

<!--Update_Description: wording update-->