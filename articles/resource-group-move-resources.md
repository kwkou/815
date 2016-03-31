<properties 
	pageTitle="将资源移到新的资源组" 
	description="使用 Azure PowerShell 或 REST API 将资源移到 Azure 资源管理器的新的资源组中。" 
	services="azure-resource-manager" 
	documentationCenter="" 
	authors="tfitzmac" 
	manager="wpickett" 
	editor=""/>

<tags 
	ms.service="azure-resource-manager" 
	ms.date="01/28/2016" 
	wacn.date="03/21/2016"/>

# 将资源移动到新的资源组或订阅中

本主题演示如何将资源从一个资源组移到另一个资源组。此外可以将资源移到新的订阅。当您决定以下事项时，您可能需要移动资源：

1. 出于计费目的，需要使资源位于不同的订阅中。
2. 某个资源不再与之前分组在一起的资源共享相同的生命周期。您想要将它移动到新的资源组以便您可以单独管理该资源，而不涉及其他资源。
3. 某个资源现在与不同在一个资源组的其他资源共享相同的生命周期。您想要将它移动到包含其他资源的资源组中以便可以同时管理它们。

移动资源时有一些重要注意事项。

1. 您不能更改该资源的位置。移动资源仅能够将其移动到新的资源组。新的资源组可能有不同的位置，但这不会更改该资源的位置。
2. 目标资源组应仅包含与您在移动的资源共享相同的应用程序生命周期的资源。
3. 如果你在使用 Azure PowerShell 或 Azure CLI，请确保使用的是最新版本。若要更新您的版本，请运行 Microsoft Web 平台安装程序并检查新版本是否可用。有关详细信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure) 以及[安装 Azure CLI](/documentation/articles/xplat-cli-install)。
4. 移动操作可能需要一段时间才能完成，在这个过程中，提示符将处于等待状态，直到操作完成。
5. 移动资源时，源组和目标组将被锁定，直到移动操作完成。在完成移动之前，将阻止对组执行写入和删除操作。

## 支持的服务

不是所有服务目前都支持移动资源的功能。

就目前而言，支持移动到新资源组和订阅的服务是：

- API 管理
- 自动化
- 批处理
- HDInsight 群集
- 密钥保管库
- 通知中心
- Redis Cache
- SQL 数据库

支持移到新的资源组而非新的订阅的服务包括：

- 虚拟机
- 存储

目前不支持移动资源的服务包括：

- 虚拟网络

当使用 Web 应用时，不能仅移动 App Service 计划。若要移动 Web 应用，您的选项包括：

- 如果目标资源组不具有 Microsoft.Web 资源，则将所有资源从一个资源组移到另一个资源组中。
- 将 web 应用移到另一个资源组中，但保留原始资源组中的 App Service 计划。

## 使用 PowerShell 来移动资源

[AZURE.INCLUDE [powershell-preview-inline-include](../includes/powershell-preview-inline-include.md)]

若要将现有资源移到另一个资源组或订阅，请使用 **Move-AzureRmResource** 命令。

第一个示例演示如何将一个资源移到新的资源组。

    PS C:\> $resource = Get-AzureRmResource -ResourceName ExampleApp -ResourceGroupName OldRG
    PS C:\> Move-AzureRmResource -DestinationResourceGroupName NewRG -ResourceId $resource.ResourceId

第二个示例演示如何将多个资源移到新的资源组。

    PS C:\> $webapp = Get-AzureRmResource -ResourceGroupName OldRG -ResourceName ExampleSite
    PS C:\> $plan = Get-AzureRmResource -ResourceGroupName OldRG -ResourceName ExamplePlan
    PS C:\> Move-AzureRmResource -DestinationResourceGroupName NewRG -ResourceId ($webapp.ResourceId, $plan.ResourceId)

若要移动到新的订阅，请包含 **DestinationSubscriptionId** 参数的值。

## 使用 Azure CLI 移动资源

若要将现有资源移到另一个资源组或订阅，请使用 **azure resource move** 命令。以下示例演示如何将一个 Redis 缓存移到新的资源组。在 **-i** 参数中，提供要移动的资源 ID 的逗号分隔列表。

    azure resource move -i "/subscriptions/{guid}/resourceGroups/OldRG/providers/Microsoft.Cache/Redis/examplecache" -d "NewRG"
    info:    Executing command resource move
    Move selected resources in OldRG to NewRG? [y/n] y
    + Moving selected resources to NewRG
    info:    resource move command OK

## 使用 REST API 移动资源

若要将现有资源移到另一个资源组或订阅中，请运行：

    POST https://management.azure.com/subscriptions/{source-subscription-id}/resourcegroups/{source-resource-group-name}/moveResources?api-version={api-version} 

在请求正文中，指定目标资源组和要移动的资源。有关移动 REST 操作的详细信息，请参阅[移动资源](https://msdn.microsoft.com/zh-cn/library/azure/mt218710.aspx)。

## 后续步骤
- [将 Azure PowerShell 用于资源管理器](/documentation/articles/powershell-azure-resource-manager)
- [将 Azure CLI 用于资源管理器](/documentation/articles/xplat-cli-azure-resource-manager)
- [使用标记来组织资源](/documentation/articles/resource-group-using-tags)

<!---HONumber=Mooncake_0314_2016-->