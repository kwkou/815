<!-- Remove rm-troubleshoot-portal -->
<properties
	pageTitle="使用 Resource Manager 执行审核操作 | Azure"
	description="使用资源管理器中的审核日志查看用户操作和错误。显示 PowerShell、Azure CLI 和 REST。"
	services="azure-resource-manager"
	documentationCenter=""
	authors="tfitzmac"
	manager="timlt"
	editor="tysonn"/>

<tags
	ms.service="azure-resource-manager"
	ms.date="03/21/2016"
	wacn.date="06/27/2016"/>

# 使用资源管理器执行审核操作

通过审核日志，你可以确定：

- 对你订阅中的资源执行了什么操作
- 谁启动的操作（虽然由后端服务启动的操作不返回用户作为调用方）
- 操作何时发生
- 操作的状态
- 其他可能有助于你研究操作的属性的值

[AZURE.INCLUDE [resource-manager-audit-limitations](../includes/resource-manager-audit-limitations.md)]

本主题着重于审核操作。 <!-- 若要了解如何使用审核日志对部署进行故障排除，请参阅 [Azure 中的资源组部署故障排除](/documentation/articles/resource-manager-troubleshoot-deployments-portal/)。 -->

可以通过 Azure PowerShell、Azure CLI、Insights REST API 或 [Insights .NET 库](https://www.nuget.org/packages/Microsoft.Azure.Insights/)检索审核日志中的信息。

## 查看审核日志的 PowerShell

1. 若要检索日志条目，请运行 **Get-AzureRmLog** 命令。你可以提供附加参数来筛选条目列表。如果未指定开始和结束时间，将返回最后一个小时的条目。
例如，若要检索过去一小时针对某个资源组的操作，请运行：

        Get-AzureRmLog -ResourceGroup ExampleGroup

    以下示例演示了如何使用审核日志来调查在指定时间内执行的操作。开始日期和结束日期以日期格式指定。

        Get-AzureRmLog -ResourceGroup ExampleGroup -StartTime 2015-08-28T06:00 -EndTime 2015-09-10T06:00

    或者，可以使用 date 函数来指定日期范围，例如过去 14 天。

        Get-AzureRmLog -ResourceGroup ExampleGroup -StartTime (Get-Date).AddDays(-14)

2. 根据指定的开始时间，前面的命令可能会返回对该资源组执行的一长串操作。你可以提供搜索条件，以筛选所要查找的结果。例如，如果你想要调查 Web 应用的停止方式，可以运行以下命令。

        Get-AzureRmLog -ResourceGroup ExampleGroup -StartTime (Get-Date).AddDays(-14) | Where-Object OperationName -eq Microsoft.Web/sites/stop/action
        
    就此示例来说，该命令显示 someone@contoso.com 执行了一项停止操作。
        
        Authorization     :
        Scope     : /subscriptions/xxxxx/resourcegroups/ExampleGroup/providers/Microsoft.Web/sites/ExampleSite
        Action    : Microsoft.Web/sites/stop/action
        Role      : Subscription Admin
        Condition :
        Caller            : someone@contoso.com
        CorrelationId     : 84beae59-92aa-4662-a6fc-b6fecc0ff8da
        EventSource       : Administrative
        EventTimestamp    : 8/28/2015 4:08:18 PM
        OperationName     : Microsoft.Web/sites/stop/action
        ResourceGroupName : ExampleGroup
        ResourceId        : /subscriptions/xxxxx/resourcegroups/ExampleGroup/providers/Microsoft.Web/sites/ExampleSite
        Status            : Succeeded
        SubscriptionId    : xxxxx
        SubStatus         : OK

3. 你可以查看特定用户针对某个资源组执行的操作，即使该资源组不再存在。

        Get-AzureRmLog -ResourceGroup deletedgroup -StartTime (Get-Date).AddDays(-14) -Caller someone@contoso.com

## 查看审核日志的 Azure CLI

1. 若要检索日志条目，请运行 **azure group log show** 命令。

        azure group log show ExampleGroup

2. 可以使用 JSON 实用程序（如 [jq](http://stedolan.github.io/jq/download/)）来筛选结果。以下示例演示如何查找更新了 Web 配置文件的操作。

        azure group log show ExampleGroup --json | jq ".[] | select(.operationName.localizedValue == "Update web sites config")"

3. 你可以查看特定用户的操作。

        azure group log show ExampleGroup --json | jq ".[] | select(.caller=="someone@contoso.com")"

## 查看审核日志的 REST API

可用于处理审核日志的 REST 操作属于 [Insights REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn931943.aspx)。若要检索审核日志事件，请参阅[列出订阅中的管理事件](https://msdn.microsoft.com/zh-cn/library/azure/dn931934.aspx)。

## 后续步骤

- 若要了解对部署进行故障排除的命令，请参阅 [Azure 中的资源组部署故障排除](/documentation/articles/resource-manager-troubleshoot-deployments-portal/)。
- 若要了解如何防止对所有用户的资源执行删除操作，请参阅[使用 Azure Resource Manager 锁定资源](/documentation/articles/resource-group-lock-resources/)。


<!---HONumber=Mooncake_0620_2016-->