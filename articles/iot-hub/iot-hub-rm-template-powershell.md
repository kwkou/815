<properties
    pageTitle="使用模板创建 Azure IoT 中心 (PowerShell) | Azure"
    description="如何使用 Azure Resource Manager 模板和 PowerShell 创建 IoT 中心。"
    services="iot-hub"
    documentationcenter=""
    author="dominicbetts"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="7eade855-c289-4ffb-b5ef-02be8c5f670f"
    ms.service="iot-hub"
    ms.devlang="multiple"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="02/24/2017"
    wacn.date="05/15/2017"
    ms.author="dobett"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="46f44171701ceea3c526e6cb5ab48a7eee67dbc8"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="create-an-iot-hub-using-azure-resource-manager-template-powershell"></a>使用 Azure Resource Manager 模板创建 IoT 中心 (PowerShell)
[AZURE.INCLUDE [iot-hub-resource-manager-selector](../../includes/iot-hub-resource-manager-selector.md)]

## <a name="introduction"></a>介绍
可以使用 Azure Resource Manager 以编程方式创建和管理 Azure IoT 中心。 本教程介绍如何将 Azure Resource Manager 模板与 PowerShell 配合使用来创建 IoT 中心。

> [AZURE.NOTE]
> Azure 提供了用于创建和使用资源的两个不同部署模型：[Azure Resource Manager 模型和经典模型](/documentation/articles/resource-manager-deployment-model/)。  本文介绍了如何使用 Azure Resource Manager 部署模型。
> 
> 

要完成本教程，需要具备以下先决条件：

* 有效的 Azure 帐户。 <br/>如果没有帐户，可以创建一个[试用帐户][lnk-free-trial]，只需几分钟即可完成。
- [Azure PowerShell 1.0][lnk-powershell-install] 或更高版本。

> [AZURE.TIP]
> [将 Azure PowerShell 与 Azure Resource Manager 配合使用][lnk-powershell-arm]一文提供了有关如何使用 PowerShell 脚本和 Azure Resource Manager 模板创建 Azure 资源的详细信息。 

## <a name="connect-to-your-azure-subscription"></a>连接到 Azure 订阅

在 PowerShell 命令提示符中，输入以下命令以登录 Azure 订阅：


	Login-AzureRmAccount -Environment $(Get-AzureRmEnvironment -Name AzureChinaCloud)


可以使用以下命令来发现可部署 IoT 中心的位置和当前支持的 API 版本：


	((Get-AzureRmResourceProvider -ProviderNamespace Microsoft.Devices).ResourceTypes | Where-Object ResourceTypeName -eq IoTHubs).Locations
	((Get-AzureRmResourceProvider -ProviderNamespace Microsoft.Devices).ResourceTypes | Where-Object ResourceTypeName -eq IoTHubs).ApiVersions


在 IoT 中心支持的位置之一，使用以下命令创建资源组来包含 IoT 中心。本示例将创建一个名为 **MyIoTRG1** 的资源组：


	New-AzureRmResourceGroup -Name MyIoTRG1 -Location "China East"

## <a name="submit-an-azure-resource-manager-template-to-create-an-iot-hub"></a>提交用于创建 IoT 中心的 Azure Resource Manager 模板
使用 JSON 模板在资源组中创建 IoT 中心。 还可以使用 Azure Resource Manager 模板更改现有 IoT 中心。

1. 使用文本编辑器创建名为 **template.json** 的 Azure Resource Manager 模板，并指定以下资源定义来创建新的标准 IoT 中心。 此示例在**中国东部**区域中添加 IoT 中心，在事件中心兼容的终结点上创建两个使用者组（**cg1** 和 **cg2**），并使用 **2016-02-03** API 版本。 此模板还要求你在名为 **hubName**的参数中传入 IoT 中心名称。 有关支持 IoT 中心的位置的最新列表，请参阅 [Azure 状态][lnk-status]。

    
		    {
		      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
		      "contentVersion": "1.0.0.0",
		      "parameters": {
		        "hubName": {
		          "type": "string"
		        }
		      },
		      "resources": [
		      {
		        "apiVersion": "2016-02-03",
		        "type": "Microsoft.Devices/IotHubs",
		        "name": "[parameters('hubName')]",
		        "location": "China East",
		        "sku": {
		          "name": "S1",
		          "tier": "Standard",
		          "capacity": 1
		        },
		        "properties": {
		          "location": "China East"
		        }
		      },
		      {
		        "apiVersion": "2016-02-03",
		        "type": "Microsoft.Devices/IotHubs/eventhubEndpoints/ConsumerGroups",
		        "name": "[concat(parameters('hubName'), '/events/cg1')]",
		        "dependsOn": [
		          "[concat('Microsoft.Devices/Iothubs/', parameters('hubName'))]"
		        ]
		      },
		      {
		        "apiVersion": "2016-02-03",
		        "type": "Microsoft.Devices/IotHubs/eventhubEndpoints/ConsumerGroups",
		        "name": "[concat(parameters('hubName'), '/events/cg2')]",
		        "dependsOn": [
		          "[concat('Microsoft.Devices/Iothubs/', parameters('hubName'))]"
		        ]
		      }
		      ],
		      "outputs": {
		        "hubKeys": {
		          "value": "[listKeys(resourceId('Microsoft.Devices/IotHubs', parameters('hubName')), '2016-02-03')]",
		          "type": "object"
		        }
		      }
		    }
    

2. 将 Azure Resource Manager 模板文件保存到本地计算机。 本示例假设将它保存在名为 **c:\templates** 的文件夹中。
3. 运行以下命令部署新的 IoT 中心，并传递 IoT 中心的名称作为参数。 在本示例中，IoT 中心名称是 **abcmyiothub**（请注意，此名称必须全局唯一，因此应包含你的姓名或姓名的首字母缩写）：

    
		New-AzureRmResourceGroupDeployment -ResourceGroupName MyIoTRG1 -TemplateFile C:\templates\template.json -hubName abcmyiothub
    

4. 输出将显示你创建的 IoT 中心的密钥。
5. 若要验证应用程序中是否添加了新 IoT 中心，可以访问[ Azure 门户][lnk-azure-portal]并查看资源列表，或使用 **Get-AzureRmResource** PowerShell cmdlet。

> [AZURE.NOTE]
> 此示例应用程序添加用于计费的 S1 标准 IoT 中心。 可以通过 [Azure 门户][lnk-azure-portal] 删除该 IoT 中心，或者在完成后使用 **Remove-AzureRmResource** PowerShell cmdlet。

## <a name="next-steps"></a>后续步骤
现在，已使用 Azure Resource Manager 模板和 PowerShell 部署了一个 IoT 中心，接下来可以进一步进行探索：

- 有关 Azure Resource Manager 功能的详细信息，请参阅 [Azure Resource Manager 概述][lnk-azure-rm-overview] 。

若要详细了解如何开发 IoT 中心，请参阅以下文章：

- [C SDK 简介][lnk-c-sdk]
- [Azure IoT SDK][lnk-sdks]

若要进一步探索 IoT 中心的功能，请参阅：

* [使用 IoT 网关 SDK 模拟设备][lnk-gateway]

<!-- Links -->
[lnk-free-trial]: /pricing/1rmb-trial/
[lnk-azure-portal]: https://portal.azure.cn/
[lnk-powershell-install]: /documentation/articles/powershell-install-configure/
[lnk-azure-rm-overview]: /documentation/articles/resource-group-overview/
[lnk-powershell-arm]: /documentation/articles/powershell-azure-resource-manager/

[lnk-c-sdk]: /documentation/articles/iot-hub-device-sdk-c-intro/
[lnk-sdks]: /documentation/articles/iot-hub-devguide-sdks/

[lnk-gateway]: /documentation/articles/iot-hub-linux-gateway-sdk-simulated-device/


<!--Update_Description:update meta properties-->