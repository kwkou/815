<properties
    pageTitle="引用 Azure 规模集模板中的虚拟网络 | Azure"
    description="如何将虚拟网络添加到现有 Azure 虚拟机规模集模板"
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
    ms.date="3/06/2017"
    wacn.date="05/02/2017"
    ms.author="negat"
    ms.sourcegitcommit="78da854d58905bc82228bcbff1de0fcfbc12d5ac"
    ms.openlocfilehash="3bff289e88eae1889c7a324092bf7d66493d9cd4"
    ms.lasthandoff="04/22/2017" />

# <a name="add-reference-to-a-virtual-network-to-an-azure-scale-set-template"></a>将虚拟网络引用添加到 Azure 规模集模板

本文介绍了如何修改[最小可行规模集模板](/documentation/articles/virtual-machine-scale-sets-mvss-start/)，以便部署到现有虚拟网络而非创建新的虚拟网络。

## <a name="change-the-template-definition"></a>更改模板定义

可以[在此处](https://raw.githubusercontent.com/gatneil/mvss/minimum-viable-scale-set/azuredeploy.json)查看最小可行规模集模板，可以[在此处](https://raw.githubusercontent.com/gatneil/mvss/existing-vnet/azuredeploy.json)查看用于将规模集部署到现有虚拟网络的模板。 让我们逐一查看创建此模板 (`git diff master minimum-viable-scale-set`) 时使用的差异内容：

首先，我们添加一个 `subnetId` 参数。 此字符串将传递到规模集配置，使得规模集能够识别要将虚拟机部署到的预先创建的子网。 此字符串必须采用以下格式：`/subscriptions/<subscription-id>resourceGroups/<resource-group-name>/providers/Microsoft.Network/virtualNetworks/<virtual-network-name>/subnets/<subnet-name>`。 例如，若要将规模集部署到具有名称 `myvnet`、子网 `mysubnet`、资源组 `myrg` 和订阅 `00000000-0000-0000-0000-000000000000` 的现有虚拟网络，则 subnetId 将是：`/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myrg/providers/Microsoft.Network/virtualNetworks/myvnet/subnets/mysubnet`。

         },
         "adminPassword": {
           "type": "securestring"
    +    },
    +    "subnetId": {
    +      "type": "string"
         }
       },

接下来，我们可以从 `resources` 阵列中删除虚拟网络资源，因为我们使用现有虚拟网络，不需要部署新的虚拟网络。

       "variables": {},
       "resources": [
    -    {
    -      "type": "Microsoft.Network/virtualNetworks",
    -      "name": "myVnet",
    -      "location": "[resourceGroup().location]",
    -      "apiVersion": "2016-12-01",
    -      "properties": {
    -        "addressSpace": {
    -          "addressPrefixes": [
    -            "10.0.0.0/16"
    -          ]
    -        },
    -        "subnets": [
    -          {
    -            "name": "mySubnet",
    -            "properties": {
    -              "addressPrefix": "10.0.0.0/16"
    -            }
    -          }
    -        ]
    -      }
    -    },

虚拟网络在部署模板前已存在，因此不需要指定从规模集到虚拟网络的 dependsOn 子句。 因此，我们删除以下行：

         {
           "type": "Microsoft.Compute/virtualMachineScaleSets",
           "name": "myScaleSet",
           "location": "[resourceGroup().location]",
           "apiVersion": "2016-04-30-preview",
    -      "dependsOn": [
    -        "Microsoft.Network/virtualNetworks/myVnet"
    -      ],
           "sku": {
             "name": "Standard_A1",
             "capacity": 2

最后，我们传入用户设置的 `subnetId` 参数（而非使用 `resourceId` 来获取同一部署中某个 vnet 的 id，这是最小可行规模集模板执行的操作）。

                           "name": "myIpConfig",
                           "properties": {
                             "subnet": {
    -                          "id": "[concat(resourceId('Microsoft.Network/virtualNetworks', 'myVnet'), '/subnets/mySubnet')]"
    +                          "id": "[parameters('subnetId')]"
                             }
                           }
                         }

## <a name="next-steps"></a>后续步骤

[AZURE.INCLUDE [mvss-next-steps-include](../../includes/mvss-next-steps.md)]

<!--Update_Description: wording update-->