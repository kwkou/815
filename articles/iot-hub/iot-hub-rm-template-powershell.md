<properties
	pageTitle="使用 Resource Manager 模板和 PowerShell 创建 IoT 中心 | Azure"
	description="遵照本教程开始使用 Resource Manager 模板和 PowerShell 创建 IoT 中心。"
	services="iot-hub"
	documentationCenter=".net"
	authors="dominicbetts"
	manager="timlt"
	editor=""/>  


<tags
     ms.service="iot-hub"
     ms.devlang="multiple"
     ms.topic="article"
     ms.tgt_pltfrm="na"
     ms.workload="na"
     ms.date="09/07/2016"
     wacn.date="12/12/2016"
     ms.author="dobett"/>


# 使用 PowerShell 创建 IoT 中心

[AZURE.INCLUDE [iot-hub-resource-manager-selector](../../includes/iot-hub-resource-manager-selector.md)]

## 介绍

可以使用 Azure Resource Manager 以编程方式创建和管理 Azure IoT 中心。本教程说明如何结合使用 Resource Manager 模板和 PowerShell 来创建 IoT 中心。

> [AZURE.NOTE] Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍如何使用资源管理器部署模型。

完成本教程需具备以下条件：

- 有效的 Azure 帐户。<br/>如果你没有帐户，可以创建一个试用帐户，只需几分钟即可完成。有关详细信息，请参阅 [Azure 免费试用][lnk-free-trial]。
- [Azure PowerShell 1.0][lnk-powershell-install] 或更高版本。

> [AZURE.TIP] [将 Azure PowerShell 与 Azure Resource Manager 配合使用][lnk-powershell-arm]一文提供有关如何使用 PowerShell 脚本和 Resource Manager 模板创建 Azure 资源的详细信息。

## 连接到 Azure 订阅

在 PowerShell 命令提示符中，输入以下命令，登录 Azure 订阅：

```
Login-AzureRmAccount -Environment $(Get-AzureRmEnvironment -Name AzureChinaCloud)
```

可以使用以下命令来发现可部署 IoT 中心的位置和当前支持的 API 版本：

```
((Get-AzureRmResourceProvider -ProviderNamespace Microsoft.Devices).ResourceTypes | Where-Object ResourceTypeName -eq IoTHubs).Locations
((Get-AzureRmResourceProvider -ProviderNamespace Microsoft.Devices).ResourceTypes | Where-Object ResourceTypeName -eq IoTHubs).ApiVersions
```

在 IoT 中心支持的位置之一，使用以下命令创建资源组来包含你的 IoT 中心。本示例将创建一个名为 **MyIoTRG1** 的资源组：

```
New-AzureRmResourceGroup -Name MyIoTRG1 -Location "China East"
```

## 提交用于创建 IoT 中心的 Azure Resource Manager 模板
使用 JSON 模板在资源组中创建 IoT 中心。还可以使用 Azure Resource Manager 模板更改现有 IoT 中心。

1. 使用文本编辑器创建名为 **template.json** 的 Azure Resource Manager 模板，并指定以下资源定义来创建新的标准 IoT 中心。此示例将“中国东部”区域添加 IoT 中心，在事件中心兼容的终结点上创建两个使用者组（**cg1** 和 **cg2**），并使用 **2016-02-03** API 版本。此模板还要求你在名为 **hubName** 的参数中传入 IoT 中心名称。

    ```
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
    ```
2. 将 Azure Resource Manager 模板文件保存到本地计算机。此示例假设将它保存在名为 **c:\\templates** 的文件夹中。
3. 运行以下命令部署新的 IoT 中心，并传递 IoT 中心的名称作为参数。在此示例中，IoT 中心的名称是 **abcmyiothub**（请注意，此名称必须全局唯一，因此，应包含姓名或姓名首字母缩写）：
   
    ```
    New-AzureRmResourceGroupDeployment -ResourceGroupName MyIoTRG1 -TemplateFile C:\templates\template.json -hubName abcmyiothub
    ```

4. 输出将显示你创建的 IoT 中心的密钥。
5. 若要验证应用程序中是否添加了新 IoT 中心，可以访问[门户预览][lnk-azure-portal]并查看资源列表，或使用 **Get-AzureRmResource** PowerShell cmdlet。

> [AZURE.NOTE] 此示例应用程序添加用于计费的 S1 标准 IoT 中心。可以通过[门户预览][lnk-azure-portal]删除该 IoT 中心，或者在完成后使用 **Remove-AzureRmResource** PowerShell cmdlet。

## 后续步骤
现在，已使用 Azure Resource Manager 模板和 PowerShell 部署了一个 IoT 中心，接下来可以进一步进行探索：

- 阅读了解 [IoT 中心资源提供程序 REST API][lnk-rest-api] 的相关功能。
- 有关 Azure Resource Manager 功能的详细信息，请参阅 [Azure Resource Manager 概述][lnk-azure-rm-overview]。

若要深入了解如何开发 IoT 中心，请参阅以下内容：

- [C SDK 简介][lnk-c-sdk]
- [IoT 中心 SDK][lnk-sdks]

若要进一步探索 IoT 中心的功能，请参阅：

- [使用网关 SDK 模拟设备][lnk-gateway]

<!-- Links -->

[lnk-free-trial]: /pricing/1rmb-trial/
[lnk-azure-portal]: https://portal.azure.cn/
[lnk-powershell-install]: /documentation/articles/powershell-install-configure/
[lnk-rest-api]: https://msdn.microsoft.com/zh-cn/library/mt589014.aspx
[lnk-azure-rm-overview]: /documentation/articles/resource-group-overview/
[lnk-powershell-arm]: /documentation/articles/powershell-azure-resource-manager/

[lnk-c-sdk]: /documentation/articles/iot-hub-device-sdk-c-intro/
[lnk-sdks]: /documentation/articles/iot-hub-devguide-sdks/

[lnk-gateway]: /documentation/articles/iot-hub-linux-gateway-sdk-simulated-device/

<!---HONumber=Mooncake_1205_2016-->