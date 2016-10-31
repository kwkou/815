<properties
	pageTitle="适用于 Azure Web 应用的基于 Azure Resource Manager 的跨平台命令行工具 | Azure"
	description="了解如何使用新的基于 Azure Resource Manager 的跨平台命令行工具来管理 Azure Web 应用。"
	services="app-service\web"
	documentationCenter=""
	authors="ahmedelnably"
	manager="stefsch"
	editor=""/>  


<tags
	ms.service="app-service-web"
	ms.workload="web"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/29/2016"
	wacn.date="10/31/2016"
	ms.author="aelnably"/>  


# 使用适用于 Azure Web 应用的基于 Azure Resource Manager 的 XPlat CLI#

> [AZURE.SELECTOR]
- [Azure CLI](/documentation/articles/app-service-web-app-azure-resource-manager-xplat-cli/)
- [Azure PowerShell](/documentation/articles/app-service-web-app-azure-resource-manager-powershell/)

在发布的 Azure 跨平台命令行工具版本 0.10.5 中，添加了新的命令。这些命令使用户能够使用基于 Azure Resource Manager 的 PowerShell 命令来管理 Web 应用。

若要了解管理资源组的相关信息，请参阅[使用 Azure CLI 管理 Azure 资源和资源组](/documentation/articles/xplat-cli-azure-resource-manager/)。


## 管理 App Service 计划 ##

### 创建 App Service 计划 ###
若要创建应用服务计划，请使用 **azure appserviceplan create** 命令。

以下是各种参数的说明：

- 	**--resource-group**：包含新创建的应用服务计划的资源组。
- 	**--name**：应用服务计划的名称。
- 	**--location**：应用服务计划的位置。
- 	**--tier**：所需的定价 SKU（选项包括：F1（免费）。D1（共享）。B1（基本小规模）、B2（基本中等规模）和 B3（基本大规模）。S1（标准小规模）、S2（标准中等规模）和 S3（标准大规模）。P1（高级小规模）、P2（高级中等规模）和 P3（高级大规模）。）
- 	**--instances**：应用服务计划中的辅助角色数目（默认值为 1）。

使用此 cmdlet 的示例：

    azure appserviceplan create --name ContosoAppServicePlan --location "China East" --resource-group ContosoAzureResourceGroup --sku P1 --instances 10

### 列出现有的 App Service 计划 ###

若要列出现有的应用服务计划，请使用 **azure appserviceplan list** 命令。

若要列出特定资源组之下的所有 App Service 计划，请使用：

    azure appserviceplan list --resource-group ContosoAzureResourceGroup

若要获取特定的应用服务计划，请使用 **azure appserviceplan show** 命令：

    azure appserviceplan show --name ContosoAppServicePlan --resource-group southchinaeast

### 配置现有的 App Service 计划 ###

若要更改现有应用服务计划的设置，请使用 **azure appserviceplan config** 命令。可更改 SKU 和辅助角色数目

    azure appserviceplan config --nameContosoAppServicePlan --resource-group ContosoAzureResourceGroup --sku S1 --instances 9

#### 缩放 App Service 计划 ####

若要缩放现有的 App Service 计划，请使用：

    azure appserviceplan config --nameContosoAppServicePlan --resource-group ContosoAzureResourceGroup --instances 9

#### 更改应用服务计划的 SKU ####

若要更改现有应用服务计划的 SKU，请使用：

    azure appserviceplan config --nameContosoAppServicePlan --resource-group ContosoAzureResourceGroup --sku S1


### 删除现有的 App Service 计划 ###

若要删除现有的应用服务计划，请先移动或删除所有已分配的 Web 应用。然后使用 **azure webapp delete** 命令删除应用服务计划。

    azure appserviceplan delete --name ContosoAppServicePlan --resource-group southchinaeast

## 管理应用服务 Web 应用 ##

### 创建 Web 应用 ###

若要创建 Web 应用，请使用 **azure webapp create** 命令。

以下是各种参数的说明：

- **--name**：Web 应用的名称。
- **--plan**：用于托管 Web 应用的服务计划名称。
- **--resource-group**：托管应用服务计划的资源组。
- **--location**：Web 应用的位置。

使用此 cmdlet 的示例：

    azure webapp create --name ContosoWebApp --resource-group ContosoAzureResourceGroup --plan ContosoAppServicePlan --location "China East"

### 删除现有的 Web 应用 ###

若要删除现有的 Web 应用，可以使用 **azure webapp delete** 命令，并且需要指定 Web 应用名称和资源组名称。

    azure webapp delete --name ContosoWebApp --resource-group ContosoAzureResourceGroup

### 列出现有的 Web 应用 ###

若要列出现有的 Web 应用，请使用 **azure webapp list** 命令。

若要列出特定资源组之下的所有 Web 应用，请使用：

    azure webapp list --resource-group ContosoAzureResourceGroup

若要获取特定的 Web 应用，请使用 **azure webapp show** 命令。

    azure webapp show --name ContosoWebApp --resource-group ContosoAzureResourceGroup

### 配置现有的 Web 应用 ###

若要更改现有 Web 应用的设置和配置，请使用 **azure webapp config set** 命令。

示例 (1)：更改 Web 应用的 php 版本

	azure webapp config set --name ContosoWebApp --resource-group ContosoAzureResourceGroup --phpversion 5.6

示例 (2)：添加或更改应用设置

	webapp config appsettings set --name ContosoWebApp --resource-group ContosoAzureResourceGroup appsetting1=appsetting1value,appsetting2=appsetting2value

若要了解可更改的其他配置，请使用 **azure webapp config set -h** 命令。

### 更改现有 Web 应用的状态 ###

#### 重新启动 Web 应用 ####

若要重新启动 Web 应用，必须指定 Web 应用的名称和资源组。

    azure webapp restart --name ContosoWebApp --resource-group ContosoAzureResourceGroup

#### 停止 Web 应用 ####

若要停止 Web 应用，必须指定 Web 应用的名称和资源组。

    azure webapp stop --name ContosoWebApp --resource-group ContosoAzureResourceGroup

#### 启动 Web 应用 ####

若要启动 Web 应用，必须指定 Web 应用的名称和资源组。

    azure webapp start --name ContosoWebApp --resource-group ContosoAzureResourceGroup

### 管理 Web 应用发布配置文件 ###

每个 Web 应用都具有可用于发布应用的发布配置文件。

#### 获取发布配置文件 ####

若要获取 Web 应用的发布配置文件，请使用：

    azure webapp publishingprofile --name ContosoWebApp --resource-group ContosoAzureResourceGroup

此命令会向命令行回显发布配置文件用户名和密码。

### 管理 Web 应用主机名 ###

若要管理 Web 应用的主机名绑定，请使用 **azure webapp config hostnames** 命令

#### 列出主机名绑定 ####

若要获取 Web 应用的当前主机名绑定，请使用：

    azure webapp config hostnames list --name ContosoWebApp --resource-group ContosoAzureResourceGroup

#### 添加主机名绑定 ####

若要将主机名绑定添加到 Web 应用，请使用：

    azure webapp config hostnames add --name ContosoWebApp --resource-group ContosoAzureResourceGroup --hostname www.contoso.com

#### 删除主机名绑定 ####

若要删除主机名绑定，请使用：

    azure webapp config hostnames delete --name ContosoWebApp --resource-group ContosoAzureResourceGroup --hostname www.contoso.com

### 后续步骤 ###
- 若要了解 Azure Resource Manager CLI 支持的相关信息，请参阅[使用 Azure CLI 管理 Azure 资源和资源组](/documentation/articles/xplat-cli-azure-resource-manager/)。
- 若要了解使用 PowerShell 管理应用服务的相关信息，请参阅[使用基于 Azure Resource Manager 的 PowerShell 来管理 Azure Web 应用](/documentation/articles/app-service-web-app-azure-resource-manager-powershell/)。

<!---HONumber=Mooncake_1024_2016-->