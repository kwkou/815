<properties
    pageTitle="使用 Azure PowerShell 配置文件上传 | Azure"
    description="如何使用 Azure PowerShell cmdlet 配置 IoT 中心，以便从连接的设备上载文件。 包括有关配置目标 Azure 存储帐户的信息。"
    services="iot-hub"
    documentationcenter=""
    author="dominicbetts"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="915f1597-272d-4fd4-8c5b-a0ccb1df0d91"
    ms.service="iot-hub"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="03/23/2017"
    wacn.date="05/15/2017"
    ms.author="dobett"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="a49f4c420c2a53f71482f9ad37f5627e1e9dac6e"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="configure-iot-hub-file-uploads-using-powershell"></a>使用 PowerShell 配置 IoT 中心文件上载

[AZURE.INCLUDE [iot-hub-file-upload-selector](../../includes/iot-hub-file-upload-selector.md)]

若要使用 [IoT 中心的文件上载功能][lnk-upload]，必须先将 Azure 存储帐户与 IoT 中心关联。 可以使用现有存储帐户，也可以创建新的存储帐户。

若要完成本教程，需要以下各项：

* 有效的 Azure 帐户。 如果没有帐户，可以创建一个[试用帐户][lnk-free-trial]，只需几分钟即可完成。
* [Azure PowerShell cmdlet][lnk-powershell-install]。
* Azure IoT 中心。 如果你没有 IoT 中心，可以使用 [New-AzureRmIoTHub cmdlet][lnk-powershell-iothub] 创建一个，或使用门户[创建 IoT 中心][lnk-portal-hub]。
* 一个 Azure 存储帐户。 如果你没有 Azure 存储帐户，可以使用 [Azure 存储 PowerShell cmdlet][lnk-powershell-storage] 创建一个，或使用门户[创建存储帐户][lnk-portal-storage]。

## <a name="sign-in-and-set-your-azure-account"></a>登录并设置 Azure 帐户

登录到你的 Azure 帐户，然后选择你的订阅。

1. 在 PowerShell 提示符下，运行 **Login-AzureRmAccount** cmdlet：

        Login-AzureRmAccount -Environment $(Get-AzureRmEnvironment -Name AzureChinaCloud)

1. 如果你有多个 Azure 订阅，则访问 Azure 即有权访问与凭据关联的所有 Azure 订阅。 使用以下命令，列出可供使用的 Azure 订阅：

        Get-AzureRMSubscription

    使用以下命令，选择想要用于运行命令以管理 IoT 中心的订阅。 可使用上一命令输出中的订阅名称或 ID：

        Select-AzureRMSubscription `
            -SubscriptionName "{your subscription name}"

## <a name="retrieve-your-storage-account-details"></a>检索存储帐户详细信息

以下步骤假设已使用 **Resource Manager** 部署模型而不**经典**部署模型创建了存储帐户。

如 IoT 中心配置文件将上载从设备，需要 Azure 存储帐户相同的订阅中的连接字符串。 还需要存储帐户中 Blob 容器的名称。 使用以下命令检索存储帐户密钥：

    Get-AzureRmStorageAccountKey `
      -Name {your storage account name} `
      -ResourceGroupName {your storage account resource group}

记下 **key1** 存储帐户密钥值。 在后续步骤中需要用到它。

可将现有的 Blob 容器用于文件上载，或新建一个容器：

* 若要列出存储帐户中的现有 Blob 容器，请使用以下命令：

        $ctx = New-AzureStorageContext `
            -StorageAccountName {your storage account name} `
            -StorageAccountKey {your storage account key}
        Get-AzureStorageContainer -Context $ctx

* 若要在存储帐户中创建 Blob 容器，请使用以下命令：

        $ctx = New-AzureStorageContext `
            -StorageAccountName {your storage account name} `
            -StorageAccountKey {your storage account key}
        New-AzureStorageContainer `
            -Name {your new container name} `
            -Permission Off `
            -Context $ctx

## <a name="configure-your-iot-hub"></a>配置 IoT 中心

现在可以配置 IoT 中心启用[文件上载功能][lnk-upload]使用存储帐户详细信息。

配置需要以下值：

**存储容器**：当前 Azure 订阅中要与 IoT 中心关联的 Azure 存储帐户中的 Blob 容器。 检索在上一部分中必要的存储帐户信息。 IoT 中心会自动生成对此 Blob 容器具有写入权限的 SAS URI，以供设备上传文件时使用。

**接收已上载文件的通知**：启用或禁用文件上载通知。

**SAS TTL**：此设置是 IoT 中心返回给设备的 SAS URI 生存时间。 默认设置为一小时。

**文件通知设置默认 TTL**：文件上载通知到期前的生存时间。 默认设置为一天。

**文件通知最大传送数**：IoT 中心将尝试传送文件上载通知的次数。 默认设置为 10。

使用以下 PowerShell cmdlet 在 IoT 中心内配置上载文件设置：

    Set-AzureRmIotHub `
        -ResourceGroupName "{your iot hub resource group}" `
        -Name "{your iot hub name}" `
        -FileUploadNotificationTtl "01:00:00" `
        -FileUploadSasUriTtl "01:00:00" `
        -EnableFileUploadNotifications $true `
        -FileUploadStorageConnectionString "DefaultEndpointsProtocol=https;AccountName={your storage account name};AccountKey={your storage account key};EndpointSuffix=core.chinacloudapi.cn" `
        -FileUploadContainerName "{your blob container name}" `
        -FileUploadNotificationMaxDeliveryCount 10

## <a name="next-steps"></a>后续步骤
有关 IoT 中心文件上传功能的详细信息，请参阅 IoT 中心开发人员指南中的 [从设备上传文件][lnk-upload] 。

若要了解有关如何管理 Azure IoT 中心的详细信息，请参阅以下链接：

* [批量管理 IoT 设备][lnk-bulk]
* [IoT 中心指标][lnk-metrics]
* [操作监视][lnk-monitor]

若要进一步探索 IoT 中心的功能，请参阅：

* [IoT 中心开发人员指南][lnk-devguide]
* [使用 IoT 网关 SDK 模拟设备][lnk-gateway]
* [从根本上保护 IoT 解决方案][lnk-securing]

[lnk-upload]: /documentation/articles/iot-hub-devguide-file-upload/

[lnk-bulk]: /documentation/articles/iot-hub-bulk-identity-mgmt/
[lnk-metrics]: /documentation/articles/iot-hub-metrics/
[lnk-monitor]: /documentation/articles/iot-hub-operations-monitoring/

[lnk-devguide]: /documentation/articles/iot-hub-devguide/
[lnk-gateway]: /documentation/articles/iot-hub-linux-gateway-sdk-simulated-device/
[lnk-securing]: /documentation/articles/iot-hub-security-ground-up/
[lnk-powershell-install]: https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs/
[lnk-powershell-storage]: https://docs.microsoft.com/zh-cn/powershell/storage/
[lnk-powershell-iothub]: https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.iothub/v1.1.0/new-azurermiothub
[lnk-portal-hub]: /documentation/articles/iot-hub-create-through-portal/
[lnk-free-trial]: /pricing/1rmb-trial/
[lnk-portal-storage]: /documentation/articles/storage-create-storage-account/