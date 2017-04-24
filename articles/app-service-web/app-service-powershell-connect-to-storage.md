<properties
    pageTitle="Azure PowerShell 脚本示例 - 将 Web 应用连接到存储帐户 | Azure"
    description="Azure PowerShell 脚本示例 - 将 Web 应用连接到存储帐户"
    services="app-service\web"
    documentationcenter=""
    author="syntaxc4"
    manager="erikre"
    editor=""
    tags="azure-service-management"
    translationtype="Human Translation" />
<tags
    ms.assetid="e4831bdc-2068-4883-9474-0b34c2e3e255"
    ms.service="app-service"
    ms.devlang="multiple"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="web"
    ms.date="03/20/2017"
    wacn.date="04/24/2017"
    ms.author="cfowler"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="329993ea61fbb1993b3382c2620c6b470ff6e838"
    ms.lasthandoff="04/14/2017" />

# <a name="connect-a-web-app-to-a-storage-account"></a>将 Web 应用连接到存储帐户

在此方案中，你将了解如何创建 Azure 存储帐户和 Azure Web 应用。 然后，将使用应用设置将存储帐户链接到 Web 应用。

必要时，请使用 [Azure PowerShell 指南](https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs/)中的说明安装 Azure PowerShell，然后运行 `Login-AzureRmAccount -EnvironmentName AzureChinaCloud` 创建与 Azure 的连接。

## <a name="sample-script"></a>示例脚本

    # Generates a Random Value
    $Random=(New-Guid).ToString().Substring(0,8)

    # Variables
    $ResourceGroup="MyResourceGroup$Random"
    $AppName="webappwithStorage$Random"
    $StorageName="webappstorage$Random"
    $Location="China North"

    # Create a Resource Group
    New-AzureRMResourceGroup -Name $ResourceGroup -Location $Location

    # Create an App Service Plan
    New-AzureRMAppservicePlan -Name WebAppwithStoragePlan -ResourceGroupName $ResourceGroup -Location $Location -Tier Basic

    # Create a Web App in the App Service Plan
    New-AzureRMWebApp -Name $AppName -ResourceGroupName $ResourceGroup -Location $Location -AppServicePlan WebAppwithStoragePlan

    # Create Storage Account
    New-AzureRMStorageAccount -Name $StorageName -ResourceGroupName $ResourceGroup -Location $Location -SkuName Standard_LRS

    # Get Connection String for Storage Account
    $StorageKey=(Get-AzureRMStorageAccountKey -ResourceGroupName $ResourceGroup -Name $StorageName).Value[0]

    # Assign Connection String to App Setting 
    Set-AzureRMWebApp -ConnectionStrings @{ MyStorageConnStr = @{ Type="Custom"; Value="DefaultEndpointsProtocol=https;AccountName=$StorageName;AccountKey=$StorageKey;" } } -Name $AppName -ResourceGroupName $ResourceGroup

## <a name="clean-up-deployment"></a>清理部署 

运行脚本示例后，可以使用以下命令删除资源组、Web 应用以及所有相关资源。

    Remove-AzureRmResourceGroup -Name myResourceGroup -Force

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
| [New-AzureRmResourceGroup](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/AzureRM.Resources/v3.5.0/new-azurermresourcegroup) | 创建用于存储所有资源的资源组。 |
| [New-AzureRmAppServicePlan](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.websites/v2.5.0/new-azurermappserviceplan) | 创建应用服务计划。 |
| [New-AzureRmWebApp](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.websites/v2.5.0/new-azurermwebapp) | 创建 Web 应用。 |
| [New-AzureRMStorageAccount](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.storage/v2.5.0/New-AzureRmStorageAccount) | 创建存储帐户。 |
| [Get-AzureRMStorageAccountKey](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.storage/v2.5.0/get-azurermstorageaccountkey) | 获取 Azure 存储帐户的访问密钥。 |
| [Set-AzureRmWebApp](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.websites/v2.5.0/set-azurermwebapp) | 修改 Web 应用的配置。 |

## <a name="next-steps"></a>后续步骤

有关 Azure PowerShell 模块的详细信息，请参阅 [Azure PowerShell 文档](https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs/)。

可以在 [Azure PowerShell 示例](/documentation/articles/app-service-powershell-samples/)中找到 Azure 应用服务 Web 应用的其他 Azure Powershell 示例。