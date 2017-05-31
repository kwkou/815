<properties
    pageTitle="引用 Azure 规模集模板中的自定义映像 | Azure"
    description="如何将自定义映像添加到现有 Azure 虚拟机规模集模板"
    services="virtual-machine-scale-sets"
    documentationcenter=""
    author="gatneil"
    manager="timlt"
    editor=""
    tags="azure-resource-manager" />
<tags
    ms.assetid="76ac7fd7-2e05-4762-88ca-3b499e87906e"
    ms.service="virtual-machine-scale-sets"
    ms.workload="na"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="5/10/2017"
    wacn.date="05/31/2017"
    ms.author="negat"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="4a18b6116e37e365e2d4c4e2d144d7588310292e"
    ms.openlocfilehash="66b176fb51fe4680de8e05184f7728fe8923f033"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/19/2017" />

# <a name="add-a-custom-image-to-an-azure-scale-set-template"></a>将自定义映像添加到 Azure 规模集模板

本文介绍了如何修改[最小可行规模集模板](/documentation/articles/virtual-machine-scale-sets-mvss-start/)，以便通过自定义映像进行部署。

## <a name="change-the-template-definition"></a>更改模板定义

可以在[此处](https://raw.githubusercontent.com/gatneil/mvss/minimum-viable-scale-set/azuredeploy.json)查看最小可行规模集模板，在[此处](https://raw.githubusercontent.com/gatneil/mvss/custom-image/azuredeploy.json)查看用于通过自定义映像部署规模集的模板。 让我们逐一查看创建此模板 (`git diff minimum-viable-scale-set custom-image`) 时使用的差异内容：

### <a name="creating-a-managed-disk-image"></a>创建托管磁盘映像

如果已有自定义的托管磁盘映像（`Microsoft.Compute/images` 类型的资源），则可以跳过此部分。

首先，添加 `sourceImageVhdUri` 参数，即 Azure 存储中通用 blob 的 URI，Azure 存储包含用于部署的自定义映像。

         },
         "adminPassword": {
           "type": "securestring"
    +    },
    +    "sourceImageVhdUri": {
    +      "type": "string",
    +      "metadata": {
    +        "description": "The source of the generalized blob containing the custom image"
    +      }
         }
       },
       "variables": {},

接下来，添加 `Microsoft.Compute/images` 类型的资源，即托管磁盘映像，该映像基于位于 URI `sourceImageVhdUri` 中的通用 blob。 该映像必须与使用它的规模集位于同一区域。 在映像的属性中，指定 OS 类型，blob 的位置（通过 `sourceImageVhdUri` 参数）以及存储帐户类型：

       "resources": [
         {
    +      "type": "Microsoft.Compute/images",
    +      "apiVersion": "2016-04-30-preview",
    +      "name": "myCustomImage",
    +      "location": "[resourceGroup().location]",
    +      "properties": {
    +        "storageProfile": {
    +          "osDisk": {
    +            "osType": "Linux",
    +            "osState": "Generalized",
    +            "blobUri": "[parameters('sourceImageVhdUri')]",
    +            "storageAccountType": "Standard_LRS"
    +          }
    +        }
    +      }
    +    },
    +    {
           "type": "Microsoft.Network/virtualNetworks",
           "name": "myVnet",
           "location": "[resourceGroup().location]",


在规模集资源中，添加涉及自定义映像的 `dependsOn` 子句，确保先创建该映像，以便规模集尝试通过该映像进行部署：

           "location": "[resourceGroup().location]",
           "apiVersion": "2016-04-30-preview",
           "dependsOn": [
    -        "Microsoft.Network/virtualNetworks/myVnet"
    +        "Microsoft.Network/virtualNetworks/myVnet",
    +        "Microsoft.Compute/images/myCustomImage"
           ],
           "sku": {
             "name": "Standard_A1",


### <a name="changing-scale-set-properties-to-use-the-managed-disk-image"></a>更改规模集属性，使用托管磁盘映像

在规模集 `storageProfile` 的 `imageReference` 中，指定 `Microsoft.Compute/images` 资源的 `id`，而不是指定平台映像的发布者、优惠、SKU 以及版本：

             "virtualMachineProfile": {
               "storageProfile": {
                 "imageReference": {
    -              "publisher": "Canonical",
    -              "offer": "UbuntuServer",
    -              "sku": "16.04-LTS",
    -              "version": "latest"
    +              "id": "[resourceId('Microsoft.Compute/images', 'myCustomImage')]"
                 }
               },
               "osProfile": {

此示例使用 `resourceId` 函数获取用同一模板创建的映像的资源 ID。 如果已事先创建托管磁盘映像，应改为提供该映像的 ID。 此 ID 必须采用以下格式：`/subscriptions/<subscription-id>resourceGroups/<resource-group-name>/providers/Microsoft.Compute/images/<image-name>`。

## <a name="next-steps"></a>后续步骤

[AZURE.INCLUDE [mvss-next-steps-include](../../includes/mvss-next-steps.md)]