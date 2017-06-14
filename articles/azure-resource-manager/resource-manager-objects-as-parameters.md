<properties
    pageTitle="Azure Resource Manager 模板 - 对象即参数 | Azure"
    description="介绍如何扩展 Azure Resource Manager 模板的功能，以便将对象用作参数"
    services="azure-resource-manager, guidance"
    documentationcenter="na"
    author="petertay"
    manager="christb"
    editor="" />
<tags
    ms.service="guidance"
    ms.topic="article"
    ms.date="05/01/2017"
    wacn.date="06/05/2017"
    ms.author="v-yeche"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="08618ee31568db24eba7a7d9a5fc3b079cf34577"
    ms.openlocfilehash="a956270dd25adf4c6dbb9827112e5c22fae96003"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="patterns-for-extending-the-functionality-of-azure-resource-manager-templates---objects-as-parameters"></a>用于扩展 Azure Resource Manager 模板功能的模式 - 对象即参数

Azure Resource Manager 模板支持使用参数来指定值，以便自定义资源部署。 此功能非常有用，可让你创建复杂部署。单个模板中的参数不能超过 255 个。 如果对资源中的每个属性使用参数并且部署规模较大，则可能会用尽参数。

## <a name="create-object-as-parameter"></a>以参数的形式创建对象

解决此问题的方法之一是将对象用作参数。 模式是在模板中将参数指定为对象，然后以值或值数组的形式提供该对象。 使用 `parameter()` 函数和点运算符引用该对象的子属性。 例如：

    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
          "VNetSettings":{"type":"object"}
      },
      "variables": {},
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
      ],          
      "outputs": {}
    }


相应的参数文件如下所示：

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
        "contentVersion": "1.0.0.0",
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
    }

在此示例中，为 VNet 指定的所有值来自于单个参数 `VNetSettings`。 此模式对属性值管理非常有用，因为可以在单个对象中保留特定资源的所有值。 尽管此示例将对象用作参数，你也可以将对象数组用作参数。 使用数组中的索引引用这些对象。

## <a name="use-object-instead-of-multiple-arrays"></a>使用对象取代多个数组

以前，你可能已使用类似的模式通过以下方式创建了资源的多个实例：创建属性值的多个数组，然后循环访问每个数组以选择值。 创建相同类型的多个资源时，此模式可以正常运行，但是，要使用它来创建子资源，可能会造成很多不便。 

此问题的原因有两个。 首先，Resource Manager 尝试并行部署子资源，但是，当两个子资源同时更新父资源时，部署将会失败。 

其次，每个属性值数组并行迭代，如果每个数组的形状不同，则会出错。 例如，考虑以下属性数组：

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
            "one","two"
        ]
    }

用于将这些值分配到复制循环中的属性的典型模式是使用 `variables()` 函数访问属性，并使用 `copyIndex()` 作为数组中的索引。 例如：

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

请注意，复制循环的 `count` 基于 `firstProperty` 数组中的属性数目。 但是，`secondProperty` 数组中的属性数目并不相同。 由于 `secondProperty` 数组的长度不同，此模板的验证失败。

但是，如果在单个对象中包含所有属性，则可以更轻松地了解何时缺少某个值。 例如：

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

## <a name="try-the-template"></a>尝试模板

如果想要体验这些模板，请执行以下步骤：

1.    转到 Azure 门户，选择“+”图标，然后搜索“模板部署”资源类型。 在搜索结果中找到后，请选择它。
2.    转到“模板部署”页后，选择“创建”按钮。 随后会打开“自定义部署”边栏选项卡。
3.    选择“编辑模板”按钮。
4.    删除右侧窗格中的空模板。
5.    将示例模板复制并粘贴到右侧窗格中。
6.    选择“保存”按钮。
7.    返回到“自定义部署”窗格后，请选择“编辑参数”按钮。
8.  在“编辑参数”边栏选项卡中，删除现有的模板。
9.  复制并粘贴上述示例参数模板。
10. 选择“保存”按钮，返回到“自定义部署”边栏选项卡。
11. 在“自定义部署”边栏选项卡中选择订阅，新建资源组或使用现有资源组，然后选择位置。 查看条款和条件，然后选中“我同意”复选框。
12.    选择“购买”按钮。

## <a name="next-steps"></a>后续步骤

如果所需的参数数目超过了每个部署允许的最大参数数目 (255)，请使用此模式在模板中指定较少的参数。 也可以使用此模式来更轻松地管理资源的属性，然后使用顺序循环模式部署资源，这样就不会发生冲突。

* 有关 `parameter()` 函数和数组用法的简介，请参阅 [Azure Resource Manager 模板函数](/documentation/articles/resource-group-template-functions/)。
* 此模式还可在[模板构建块项目](https://github.com/mspnp/template-building-blocks)和 [Azure 参考体系结构](https://docs.microsoft.com/azure/architecture/reference-architectures/)中实现。

<!--Update_Description:new article about illustrating how to use objects as parameters-->
