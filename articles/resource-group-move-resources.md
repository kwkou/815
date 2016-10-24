<!-- Remove Api-management, Azure Portal -->
<properties 
	pageTitle="将资源移到新的资源组 | Azure" 
	description="使用 Azure Resource Manager 将资源移到新的资源组或订阅。" 
	services="azure-resource-manager" 
	documentationCenter="" 
	authors="tfitzmac" 
	manager="timlt" 
	editor="tysonn"/>  


<tags 
	ms.service="azure-resource-manager" 
	ms.workload="multiple" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="09/12/2016" 
	wacn.date="10/24/2016"/>

# 将资源移到新资源组或订阅中

本主题演示如何将资源从一个资源组移到另一个资源组。你还可以将资源移到新的订阅（但该订阅必须位于同一个[租户](/documentation/articles/active-directory-howto-tenant/)中）。当您决定以下事项时，您可能需要移动资源：

1. 为便于计费，资源需在不同的订阅中。
2. 某个资源不再与之前分组在一起的资源共享相同的生命周期。您想要将它移动到新的资源组以便您可以单独管理该资源，而不涉及其他资源。

移动资源时，源组和目标组将被锁定，直到移动操作完成。在完成移动之前，将阻止对组执行写入和删除操作。

您不能更改该资源的位置。移动资源仅能够将其移动到新的资源组。新的资源组可能有不同的位置，但这不会更改该资源的位置。


## 移动资源前需查看的清单

移动资源之前需执行的一些重要步骤。验证这些条件可以避免错误。

1. 该服务必须支持移动资源的功能。请参阅以下列表，以了解[哪些服务支持移动资源](#services-that-support-move)。
2. 必须针对要移动的资源的资源提供程序注册目标订阅。否则会出现错误“未针对资源类型注册订阅”。将资源移到新的订阅时，可能会遇到此问题，但该订阅从未配合该资源类型使用。若要了解如何检查注册状态和注册资源提供程序，请参阅 [Resource providers and types](/documentation/articles/resource-manager-supported-services/#resource-providers-and-types)（资源提供程序和类型）。
3. 如果你要使用 Azure PowerShell 或 Azure CLI，请使用最新版本。若要更新您的版本，请运行 Microsoft Web 平台安装程序并检查新版本是否可用。有关详细信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/) 以及[安装 Azure CLI](/documentation/articles/xplat-cli-install/)。
4. 如果你要移动 App Service 应用，则你已查看 [App Service 限制](#app-service-limitations)。
5. 如果你要移动通过经典模型部署的资源，则你已查看[经典部署限制](#classic-deployment-limitations)。

## 支持移动的服务

就目前而言，支持移动到新资源组和订阅的服务是：

<!-- - API 管理 -->
- 自动化
- 批处理
- CDN
- 云服务 - 请参阅[经典部署限制](#classic-deployment-limitations)
- DocumentDB
- HDInsight 群集
- 密钥保管库
- 通知中心
- Redis Cache
- 计划程序
- 存储
- 存储（经典）- 请参阅[经典部署限制](#classic-deployment-limitations)
- SQL 数据库服务器 - 数据库和服务器必须位于同一个资源组中。当你移动 SQL 服务器时，其所有数据库也会一起移动。
- 虚拟机
- 虚拟机（经典）- 请参阅[经典部署限制](#classic-deployment-limitations)
- 虚拟网络

## 不支持移动的服务

目前不支持移动资源的服务包括：

- 应用程序网关
- Application Insights
- Express Route
- 虚拟机规模集
- 虚拟网络（经典）- 请参阅[经典部署限制](#classic-deployment-limitations)
- VPN 网关

## App Service 限制

使用 Web 应用时，你不能只移动 App Service 计划。若要移动 Web 应用，可以使用以下选项：

- 将该资源组中的 App Service 计划以及所有其他 App Service 资源移到尚无 App Service 资源的新资源组。这一要求意味着，与 App Service 计划不关联的 App Service 资源也必须移动。
- 将应用移到另一个资源组中，但保留原始资源组中的所有 App Service 计划。

如果你的原始资源组还包括 Application Insights 资源，则不能移动该资源，因为 Application Insights 目前不支持移动操作。如果在移动 App Service 应用时包括了 Application Insights 资源，则整个移动操作会失败。不过，即使 Application Insights 和 App Service 计划不与应用位于同一资源组中，应用也可以正常运行。

例如，如果你的资源组包含：

- **web-a**，与 **plan-a** 和 **app-insights-a** 相关联
- **web-b**，与 **plan-b** 和 **app-insights-b** 相关联

你的选项包括：

- 移动 **web-a**、**plan-a**、**web-b** 和 **plan-b**
- 移动 **web-a** 和 **web-b**
- 移动 **web-a**
- 移动 **web-b**

所有其他的组合都涉及移动不能移动的资源类型 (Application Insights)，或保留在移动 App Service 计划时不能保留的资源类型（任何 App Service 资源类型）。

如果你的 Web 应用与其 App Service 计划位于不同的资源组中，而你想要将二者都移到新的资源组，则必须分两步执行移动操作。例如：

- **web-a** 位于 **web-group** 中
- **plan-a** 位于 **plan-group** 中
- 想要让 **web-a** 和 **plan-a** 位于 **combined-group** 中

若要完成此移动操作，可按以下顺序执行两个独立的移动操作：

1. 将 **web-a** 移到 **plan-group** 中
2. 将 **web-a** 和 **plan-a** 移到 **combined-group** 中。

## 恢复服务限制

移动不支持用于使用 Azure Site Recovery 设置灾难恢复的“存储”、“网络”或“计算”资源。

例如，假设已设置将本地计算机复制到存储帐户 (Storage1)，并且想要受保护的计算机在故障转移到 Azure 之后显示为连接到虚拟网络 (Network1) 的虚拟机 (VM1)。不能在同一订阅中的资源组之间或在订阅之间移动这些 Azure 资源 - Storage1、VM1 和 Network1。

## 经典部署限制

移动通过经典模型部署的资源时，其选项各不相同，具体取决于是在订阅内移动资源，还是将资源移到新的订阅。

**在同一订阅内**将资源从一个资源组移到另一个资源组时存在以下限制：

- 不能移动虚拟网络（经典）。
- 虚拟机（经典）必须与云服务一起移动。
- 移动云服务时，必须移动其所有虚拟机。
- 一次只能移动一项云服务。
- 一次只能移动一个存储帐户（经典）。
- 存储帐户（经典）与虚拟机或云服务不能在同一操作中移动。

将资源移到**新订阅**时存在以下限制：

- 必须在同一操作中移动订阅中的所有经典资源。
- 目标订阅不得包含任何其他经典资源。
- 只能通过独立的适用于经典移动的 REST API 来请求移动。将经典资源移到新订阅时，不能使用标准的 Resource Manager 移动命令。

若要将经典资源移到**同一订阅内**的新资源组，请使用[门户](#use-portal)、[Azure PowerShell](#use-powershell)、[Azure CLI](#use-azure-cli) 或 [REST API](#use-rest-api)。

若要**将经典资源移动到新订阅**，则必须使用特定于经典资源的 REST 操作。若要查看在跨订阅移动经典资源时某项订阅能否以源订阅或目标订阅的形式参与，请使用以下操作：

    POST https://management.chinacloudapi.cn/subscriptions/{subscriptionId}/providers/Microsoft.ClassicCompute/validateSubscriptionMoveAvailability?api-version=2016-04-01
    
对于源订阅，请使用以下请求正文：

    {
        "role": "source"
    }

对于目标订阅，请使用以下请求正文：

    {
        "role": "target"
    }

任一验证操作的响应为：

    {
        "status": "{status}",
        "reasons": [
            "reason1",
            "reason2"
        ]
    }

若要将所有经典资源从一个订阅移到另一个订阅，请使用以下操作：

    POST https://management.chinacloudapi.cn/subscriptions/{subscription-id}/providers/Microsoft.ClassicCompute/moveSubscriptionResources?api-version=2016-04-01

使用以下请求正文：

    {
        "target": "/subscriptions/{target-subscription-id}"
    }


## 使用门户

若要将资源移到同一订阅中的新资源组，请选择包含这些资源的资源组，然后选择“移动”按钮。

![移动资源](./media/resource-group-move-resources/edit-rg-icon.png)  


若要将资源移到新订阅，请选择包含这些资源的资源组，然后选择“编辑订阅”图标。

![移动资源](./media/resource-group-move-resources/change-subscription.png)  


选择要移动的资源和目标资源组。确认需要更新这些资源的脚本，选择“确定”。如果在上一步中已选择“编辑订阅”图标，则还必须选择目标订阅。

![选择目标](./media/resource-group-move-resources/select-destination.png)  


在“通知”中，可以看到移动操作正在运行。

![显示移动状态](./media/resource-group-move-resources/show-status.png)  


操作完成后，你会获得结果通知。

![显示移动结果](./media/resource-group-move-resources/show-result.png)  


## 使用 PowerShell

若要将现有资源移到另一个资源组或订阅，请使用 **Move-AzureRmResource** 命令。

第一个示例演示如何将一个资源移到新的资源组。

    $resource = Get-AzureRmResource -ResourceName ExampleApp -ResourceGroupName OldRG
    Move-AzureRmResource -DestinationResourceGroupName NewRG -ResourceId $resource.ResourceId

第二个示例演示如何将多个资源移到新的资源组。

    $webapp = Get-AzureRmResource -ResourceGroupName OldRG -ResourceName ExampleSite
    $plan = Get-AzureRmResource -ResourceGroupName OldRG -ResourceName ExamplePlan
    Move-AzureRmResource -DestinationResourceGroupName NewRG -ResourceId $webapp.ResourceId, $plan.ResourceId

若要移动到新的订阅，请包含 **DestinationSubscriptionId** 参数的值。

系统将要求确认是否想要移动指定的资源。

    Confirm
    Are you sure you want to move these resources to the resource group
    '/subscriptions/{guid}/resourceGroups/newRG' the resources:

    /subscriptions/{guid}/resourceGroups/destinationgroup/providers/Microsoft.Web/serverFarms/exampleplan
    /subscriptions/{guid}/resourceGroups/destinationgroup/providers/Microsoft.Web/sites/examplesite
    [Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): y

## 使用 Azure CLI

若要将现有资源移到另一个资源组或订阅，请使用 **azure resource move** 命令。以下示例演示如何将一个 Redis 缓存移到新的资源组。在 **-i** 参数中，提供要移动的资源 ID 的逗号分隔列表。

    azure resource move -i "/subscriptions/{guid}/resourceGroups/OldRG/providers/Microsoft.Cache/Redis/examplecache" -d "NewRG"
	
系统将要求确认是否想要移动指定的资源。
	
    info:    Executing command resource move
    Move selected resources in OldRG to NewRG? [y/n] y
    + Moving selected resources to NewRG
    info:    resource move command OK

## 使用 REST API

若要将现有资源移到另一个资源组或订阅中，请运行：

    POST https://management.chinacloudapi.cn/subscriptions/{source-subscription-id}/resourcegroups/{source-resource-group-name}/moveResources?api-version={api-version} 

在请求正文中，指定目标资源组和要移动的资源。有关移动 REST 操作的详细信息，请参阅[移动资源](https://msdn.microsoft.com/library/azure/mt218710.aspx)。




## 后续步骤
- 若要了解管理订阅所需的 PowerShell cmdlet，请参阅 [Using Azure PowerShell with Resource Manager](/documentation/articles/powershell-azure-resource-manager/)（将 Azure PowerShell 与 Resource Manager 配合使用）。
- 若要了解管理订阅所需的 Azure CLI 命令，请参阅 [Using the Azure CLI with Resource Manager](/documentation/articles/xplat-cli-azure-resource-manager/)（将 Azure CLI 与 Resource Manager 配合使用）。
- 若要了解管理订阅所需的门户功能，请参阅 [Using the Azure Portal to manage resources](/documentation/articles/resource-group-portal/)（使用 Azure 门户管理资源）。
- 若要了解如何对资源应用逻辑组织，请参阅 [Using tags to organize your resources](/documentation/articles/resource-group-using-tags/)（使用标记来组织资源）。

<!---HONumber=Mooncake_1017_2016-->