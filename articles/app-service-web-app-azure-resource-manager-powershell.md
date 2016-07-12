<properties
	pageTitle="适用于 Azure Web 应用的基于 Azure Resource Manager 的 PowerShell 命令 | Azure"
	description="了解如何使用基于 Azure Resource Manager 的新 PowerShell 命令来管理 Azure Web Apps。"
	services="app-service\web"
	documentationCenter=""
	authors="ahmedelnably"
	manager="stefsch"
	editor=""/>

<tags
	ms.service="app-service-web"
	ms.date="06/14/2016"
	wacn.date="07/04/2016"/>

# 使用基于 Azure Resource Manager 的 PowerShell 来管理 Azure Web Apps#

发行的 Azure PowerShell 版本 1.0.0 中添加了新新命令，可让用户使用基于 Azure Resource Manager 的 PowerShell 命令来管理 Web Apps。

若要了解如何管理资源组，请参阅[将 Azure PowerShell 与 Azure Resource Manager 搭配使用](/documentation/articles/powershell-azure-resource-manager/)。

若要了解 Web 应用 Azure Resource Manager PowerShell cmdlet 的完整参数和选项列表，请参阅 [Web 应用基于 Azure Resource Manager 的 PowerShell Cmdlet 的完整 Cmdlet 参考](https://msdn.microsoft.com/zh-cn/library/mt619237.aspx)

## 管理 App Service 计划 ##

### 创建 App Service 计划 ###
若要创建新的 App Service 计划，请使用 **New-AzureRmAppServicePlan** cmdlet。

以下是各种参数的说明：

- 	**Name**：App Service 计划的名称。
- 	**Location**：服务计划位置。
- 	**ResourceGroupName**：包含新创建的 App Service 计划的资源组。
- 	**Tier**：所需的定价层（默认值是“免费”，其他选项包括“共享”、“基本”和“标准”）。
- 	**WorkerSize**：辅助角色大小（如果 Tier 参数指定为“基本”或“标准”，则默认值为“小”。其他选项包括“中”和“大”。）
- 	**NumberofWorkers**：App Service 计划中的辅助角色数目（默认值为 1）。 

使用此 cmdlet 的示例：

    New-AzureRmAppServicePlan -Name ContosoAppServicePlan -Location "China East" -ResourceGroupName ContosoAzureResourceGroup -Tier Standard -WorkerSize Large -NumberofWorkers 10

### 列出现有的 App Service 计划 ###

若要列出现有的 App Service 计划，请使用 **Get-AzureRmAppServicePlan** cmdlet。

若要列出订阅之下的所有 App Service 计划，请使用：

    Get-AzureRmAppServicePlan

若要列出特定资源组之下的所有 App Service 计划，请使用：

    Get-AzureRmAppServicePlan -ResourceGroupname ContosoAzureResourceGroup

若要获取特定的 App Service 计划，请使用：

    Get-AzureRmAppServicePlan -Name ContosoAppServicePlan


### 配置现有的 App Service 计划 ###

若要更改现有 App Service 计划的设置，请使用 **Set-AzureRmAppServicePlan** cmdlet。可以更改层、辅助角色大小和辅助角色数目

    Set-AzureRmAppServicePlan -Name ContosoAppServicePlan -ResourceGroupName ContosoAzureResourceGroup -Tier Standard -WorkerSize Medium -NumberofWorkers 9

#### 缩放 App Service 计划 ####

若要缩放现有的 App Service 计划，请使用：

    Set-AzureRmAppServicePlan -Name ContosoAppServicePlan -ResourceGroupName ContosoAzureResourceGroup -NumberofWorkers 9

#### 更改 App Service 计划的辅助角色大小 ####

若要更改现有 App Service 计划中的辅助角色大小，请使用：

    Set-AzureRmAppServicePlan -Name ContosoAppServicePlan -ResourceGroupName ContosoAzureResourceGroup -WorkerSize Medium

#### 更改 App Service 计划的层 ####

若要更改现有 App Service 计划的层，请使用：

    Set-AzureRmAppServicePlan -Name ContosoAppServicePlan -ResourceGroupName ContosoAzureResourceGroup -Tier Standard

### 删除现有的 App Service 计划 ###

若要删除现有的 App Service 计划，必须先移动或删除分配的所有 Web Apps，然后使用 **Remove-AzureRmAppServicePlan** cmdlet，即可删除 App Service 计划。

    Remove-AzureRmAppServicePlan -Name ContosoAppServicePlan -ResourceGroupName ContosoAzureResourceGroup

## 管理 Azure Web Apps ##

### 创建新的 Web 应用 ###

若要创建新的 Web 应用，请使用 **New-AzureRmWebApp** cmdlet。

以下是各种参数的说明：

- **Name**：Web 应用的名称。
- **AppServicePlan**：用于托管 Web 应用的服务计划的名称。
- **ResourceGroupName**：托管 App Service 计划的资源组。
- **Location**：Web 应用的位置。

使用此 cmdlet 的示例：

    New-AzureRmWebApp -Name ContosoWebApp -AppServicePlan ContosoAppServicePlan -ResourceGroupName ContosoAzureResourceGroup -Location "China East"

### 删除现有的 Web 应用 ###

若要删除现有的 Web 应用，可以使用 New-**Remove-AzureRmWebApp** cmdlet，并且需要指定 Web 应用名称和资源组名称。

    Remove-AzureRmWebApp -Name ContosoWebApp -ResourceGroupName ContosoAzureResourceGroup


### 列出现有的 Web 应用 ###

若要列出现有的 Web 应用，请使用 **Get-AzureRmWebApp** cmdlet。

若要列出订阅之下的所有 Web 应用，请使用：

    Get-AzureRmWebApp

若要列出特定资源组之下的所有 Web 应用，请使用：

    Get-AzureRmWebApp -ResourceGroupname ContosoAzureResourceGroup

若要获取特定的 Web 应用，请使用：

    Get-AzureRmWebApp -Name ContosoWebApp

### 配置现有的 Web 应用 ###

若要更改现有 Web 应用的设置和配置，请使用 **Set-AzureRmWebApp** cmdlet。有关完整的参数列表，请查看 [Cmdlet 参考链接](https://msdn.microsoft.com/zh-cn/library/mt652487.aspx)

示例 (1)：使用此 cmdlet 来更改连接字符串

	$connectionstrings = @{ ContosoConn1 = @{ Type = "MySql"; Value = "MySqlConn"}; ContosoConn2 = @{ Type = "SQLAzure"; Value = "SQLAzureConn"} }
	Set-AzureRmWebApp -Name ContosoWebApp -ResourceGroupName ContosoAzureResourceGroup -ConnectionStrings $connectionstrings

示例 (2)：添加应用设置的示例

	$appsettings = @{appsetting1 = "appsetting1value"; appsetting2 = "appsetting2value"}
	Set-AzureRmWebApp -Name ContosoWebApp -ResourceGroupName ContosoAzureResourceGroup -AppSettings $appsettings


示例 (3)：将 Web 应用设置为在 64 位模式下运行

	Set-AzureRmWebApp -Name ContosoWebApp -ResourceGroupName ContosoAzureResourceGroup -Use32BitWorkerProcess $False

### 更改现有 Web 应用的状态 ###

#### 重新启动 Web 应用 ####

若要重新启动 Web 应用，必须指定 Web 应用的名称和资源组。

    Restart-AzureRmWebapp -Name ContosoWebApp -ResourceGroupName ContosoAzureResourceGroup

#### 停止 Web 应用 ####

若要停止 Web 应用，必须指定 Web 应用的名称和资源组。

    Stop-AzureRmWebapp -Name ContosoWebApp -ResourceGroupName ContosoAzureResourceGroup

#### 启动 Web 应用 ####

若要启动 Web 应用，必须指定 Web 应用的名称和资源组。

    Start-AzureRmWebapp -Name ContosoWebApp -ResourceGroupName ContosoAzureResourceGroup

### 管理 Web 应用发布配置文件 ###

每个 Web 应用都有可用于发布应用的发布配置文件，并且可对发布配置文件执行许多操作。

#### 获取发布配置文件 ####

若要获取 Web 应用的发布配置文件，请使用：

    Get-AzureRmWebAppPublishingProfile -Name ContosoWebApp -ResourceGroupName ContosoAzureResourceGroup -OutputFile .\publishingprofile.txt

请注意，这会将发布配置文件回显至命令行，以及将发布配置文件输出至文本文件。

#### 重置发布配置文件 ####

若要同时对 Web 应用的 FTP 和 Web 部署重置发布密码，请使用：

    Reset-AzureRmWebAppPublishingProfile -Name ContosoWebApp -ResourceGroupName ContosoAzureResourceGroup

### 管理 Web 应用证书 ###

若要了解如何管理 Web 应用证书，请参阅[使用 PowerShell 创建 SSL 证书绑定](/documentation/articles/app-service-web-app-powershell-ssl-binding/)



### 后续步骤 ###
- 若要了解 Azure Resource Manager PowerShell 支持，请参阅[将 Azure PowerShell 与 Azure Resource Manager 搭配使用](/documentation/articles/powershell-azure-resource-manager/)。
- 若要了解如何使用 PowerShell 管理 Azure SSL 证书，请参阅[使用 PowerShell 创建 SSL 证书绑定](/documentation/articles/app-service-web-app-powershell-ssl-binding/)。
- 若要了解适用于 Azure Web Apps 的基于 Azure Resource Manager 的 PowerShell cmdlet 的完整列表，请参阅 [Web Apps Azure Resource Manager PowerShell Cmdlet 的 Azure Cmdlet 参考](https://msdn.microsoft.com/zh-cn/library/mt619237.aspx)。

<!---HONumber=Mooncake_0627_2016-->