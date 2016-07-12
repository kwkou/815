<properties
	pageTitle="使用 ARM 模板和 PowerShell 创建 IoT 中心 | Azure"
	description="遵照本教程开始使用 Resource Manager 模板和 PowerShell 创建 IoT 中心。"
	services="iot-hub"
	documentationCenter=".net"
	authors="dominicbetts"
	manager="timlt"
	editor=""/>

<tags
     ms.service="iot-hub"
     ms.date="04/07/2016"
     wacn.date="05/05/2016"/>

# 使用 PowerShell 创建 IoT 中心

[AZURE.INCLUDE [iot-hub-resource-manager-selector](../includes/iot-hub-resource-manager-selector.md)]

## 介绍

你可以使用 Azure Resource Manager (ARM) 以编程方式创建和管理 Azure IoT 中心。本教程说明如何结合使用 Resource Manager 模板和 PowerShell 来创建 IoT 中心。

> [AZURE.NOTE] Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍如何使用资源管理器部署模型。

为了完成本教程，你需要有：

- 有效的 Azure 帐户。<br/>如果你没有帐户，只需花费几分钟就能创建一个试用帐户。有关详细信息，请参阅 [Azure 试用][lnk-free-trial]。
- [Microsoft Azure PowerShell 1.0][lnk-powershell-install] 或更高版本。

> [AZURE.TIP] [将 Azure PowerShell 与 Azure Resource Manager 配合使用][lnk-powershell-arm]一文提供了有关如何使用 PowerShell 脚本和 ARM 模板创建 Azure 资源的详细信息。

## 连接到你的 Azure 订阅

在 PowerShell 命令提示符中，输入以下命令以登录到你的 Azure 订阅：

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

## 提交模板以创建 IoT 中心

使用 JSON 模板在资源组中创建新的 IoT 中心。还可以使用模板更改现有的 IoT 中心。

1. 使用文本编辑器创建名为 **template.json** 的 ARM 模板，并指定以下资源定义来创建新的标准 IoT 中心。本示例在**中国东部**区域添加 IoT 中心，并使用 **2016-02-03** API 版本。此模板还要求你在名为 **hubName** 的参数中传入 IoT 中心名称。

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

2. 将模板文件保存在你的本地计算机上。本示例假设将它保存在名为 **c:\\templates** 的文件夹中。

3. 运行以下命令部署新的 IoT 中心，并传递 IoT 中心的名称作为参数。在此示例中，IoT 中心的名称为 **myiothub**（请注意此名称必须全局唯一）：

    ```
    New-AzureRmResourceGroupDeployment -ResourceGroupName MyIoTRG1 -TemplateFile C:\templates\template.json -hubName myiothub
    ```

4. 输出将显示你创建的 IoT 中心的密钥。

5. 若要验证应用程序中是否添加了新 IoT 中心，你可以访问[门户][lnk-azure-portal]并查看资源列表，或使用 **Get-AzureRmResource** PowerShell cmdlet。

> [AZURE.NOTE] 本示例应用程序将添加用于对你计费的 S1 标准 IoT 中心。可以通过[门户][lnk-azure-portal]删除该 IoT 中心，或者在完成后使用 **Remove-AzureRmResource** PowerShell cmdlet。

## 后续步骤

既然你已使用 REST API 和 PowerShell 部署了一个 IoT 中心，接下来可以进一步进行探索：

- 阅读有关 [IoT 中心资源提供程序 REST API][lnk-rest-api] 的功能。
- 有关 Azure Resource Manager 功能的详细信息，请参阅 [Azure Resource Manager 概述][lnk-azure-rm-overview]。

<!-- Links -->
[lnk-free-trial]: /pricing/1rmb-trial/
[lnk-azure-portal]: https://manage.windowsazure.cn
[lnk-powershell-install]: /documentation/articles/powershell-install-configure/
[lnk-rest-api]: https://msdn.microsoft.com/zh-cn/library/mt589014.aspx
[lnk-azure-rm-overview]: /documentation/articles/resource-group-overview/
[lnk-powershell-arm]: /documentation/articles/powershell-azure-resource-manager/

<!---HONumber=Mooncake_0307_2016-->