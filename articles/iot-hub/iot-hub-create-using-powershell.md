<properties
    pageTitle="使用 PowerShell cmdlet 创建 Azure IoT 中心 | Azure"
    description="如何使用 PowerShell cmdlet 创建 IoT 中心。"
    services="iot-hub"
    documentationcenter=""
    author="dominicbetts"
    manager="timlt"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.service="iot-hub"
    ms.devlang="multiple"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date: 03/23/2017
    wacn.date="05/08/2017"
    ms.author="dobett"
    ms.sourcegitcommit="7cc8d7b9c616d399509cd9dbdd155b0e9a7987a8"
    ms.openlocfilehash="a949f1bff396bc5d0d05b2297ca8f4103f49e638"
    ms.lasthandoff="04/07/2017" />

# <a name="create-an-iot-hub-using-the-new-azurermiothub-cmdlet"></a>使用 New-AzureRmIotHub cmdlet 创建 IoT 中心

[AZURE.INCLUDE [iot-hub-resource-manager-selector](../../includes/iot-hub-resource-manager-selector.md)]

## <a name="introduction"></a>介绍

可以使用 Azure PowerShell cmdlet 创建和管理 Azure IoT 中心。 本教程介绍如何使用 PowerShell 创建 IoT 中心。

> [AZURE.NOTE]
> Azure 提供了用于创建和使用资源的两个不同部署模型：[Azure Resource Manager 模型和经典模型](/documentation/articles/resource-manager-deployment-model/)。  本文介绍了如何使用 Azure Resource Manager 部署模型。

要完成本教程，需要具备以下先决条件：

* 有效的 Azure 帐户。 <br/>如果没有帐户，可以创建一个[试用帐户][lnk-free-trial]，只需几分钟即可完成。
* [Azure PowerShell 1.0][lnk-powershell-install] 或更高版本。
* [Azure Resource Manager cmdlet][lnk-rm-install]。

## <a name="connect-to-your-azure-subscription"></a>连接到 Azure 订阅
在 PowerShell 命令提示符中，输入以下命令以登录你的 Azure 订阅：

    Login-AzureRmAccount -Environment $(Get-AzureRmEnvironment -Name AzureChinaCloud)

## <a name="create-resource-group"></a>创建资源组

需要一个资源组来部署 IoT 中心。 可以使用现有资源组，也可以创建新组。

可以使用以下命令来搜索可以部署 IoT 中心的位置：

    ((Get-AzureRmResourceProvider `
      -ProviderNamespace Microsoft.Devices).ResourceTypes `
      | Where-Object ResourceTypeName -eq IoTHubs).Locations

若要在 IoT 中心的其中一个支持位置中为 IoT 中心创建资源组，请使用以下命令。 此示例在“中国东部”区域中创建名为 **MyIoTRG1** 的资源组：

    New-AzureRmResourceGroup -Name MyIoTRG1 -Location "China East"

## <a name="create-an-iot-hub"></a>创建 IoT 中心

若要在上一步创建的资源组中创建 IoT 中心，请使用以下命令。 此示例在“中国东部”区域中创建名为 **MyTestIoTHub** 的 **S1** 中心：

    New-AzureRmIotHub `
        -ResourceGroupName MyIoTRG1 `
        -Name MyTestIoTHub `
        -SkuName S1 -Units 1 `
        -Location "China East"

> [AZURE.NOTE]
> IoT 中心的名称必须是唯一的。

可以使用以下命令列出订阅中的所有 IoT 中心：

    Get-AzureRmIotHub

上一个示例添加向你计费的 S1 标准 IoT 中心。 可以使用以下命令删除 IoT 中心：

    Remove-AzureRmIotHub `
        -ResourceGroupName MyIoTRG1 `
        -Name MyTestIoTHub

或者，可以使用以下命令删除资源组及其包含的所有资源：

    Remove-AzureRmResourceGroup -Name MyIoTRG1

## <a name="next-steps"></a>后续步骤

现在，已使用 PowerShell cmdlet 部署了 IoT 中心，接下来可以进一步进行探索：

* 发现其他[可用于 IoT 中心的 PowerShell cmdlet][lnk-iothub-cmdlets]。


若要详细了解如何开发 IoT 中心，请参阅以下文章：

* [C SDK 简介][lnk-c-sdk]
* [Azure IoT SDK][lnk-sdks]

若要进一步探索 IoT 中心的功能，请参阅：

* [使用 IoT 网关 SDK 模拟设备][lnk-gateway]

<!-- Links -->
[lnk-free-trial]: /pricing/1rmb-trial/
[lnk-powershell-install]: https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs
[lnk-iothub-cmdlets]: https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.iothub/v1.3.0/azurerm.iothub
[lnk-rm-install]: https://docs.microsoft.com/zh-cn/powershell/resourcemanager/


[lnk-c-sdk]: /documentation/articles/iot-hub-device-sdk-c-intro/
[lnk-sdks]: /documentation/articles/iot-hub-devguide-sdks/

[lnk-gateway]: /documentation/articles/iot-hub-linux-gateway-sdk-simulated-device/