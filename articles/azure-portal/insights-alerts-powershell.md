<properties
	pageTitle="使用 PowerShell 为 Azure 服务创建警报 | Azure"
    description="使用 PowerShell 创建 Azure 警报，以便在满足指定的条件时触发通知或自动化操作。"
	authors="rboucher"
	manager=""
	editor=""
	services="monitoring-and-diagnostics"
	documentationCenter="monitoring-and-diagnostics"/>

<tags
	ms.service="azure-portal"
	ms.workload="na"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="10/10/2016"
	wacn.date="12/05/2016"
	ms.author="robb"/>


# 使用 PowerShell 为 Azure 服务创建警报 

> [AZURE.SELECTOR]
- [门户预览](/documentation/articles/insights-alerts-portal/)
- [PowerShell](/documentation/articles/insights-alerts-powershell/)
- [CLI](/documentation/articles/insights-alerts-command-line-interface/)

## 概述

本文将展示如何使用 PowerShell 设置 Azure 警报。

可以根据监视指标或事件接收 Azure 服务的警报。

- **指标值** - 当指定指标的值在任一方向越过了指定的阈值时警报将触发。也就是说，当条件先是满足以及之后不再满足该条件时，警报都会触发。
- **活动日志事件** - 警报可以在发生*每个*事件时都触发，也可以仅在发生特定数量的事件时触发。

可以配置警报以在其触发时执行以下操作：
 
- 向服务管理员和共同管理员发送电子邮件通知
- 向指定的其他电子邮件地址发送电子邮件。
- 调用 Webhook
- 开始执行 Azure Runbook（仅在 Azure 门户预览中可行）

可以使用以下工具配置和获取关于警报的信息：

- [Azure 门户预览](/documentation/articles/insights-alerts-portal/)
- [PowerShell](/documentation/articles/insights-alerts-powershell/)
- [命令行界面 (CLI)](/documentation/articles/insights-alerts-command-line-interface/)
- [Azure Insights REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn931945.aspx)


若要获得更多信息，始终可以通过键入 ```get-help``` 然后键入希望获取其帮助信息的 PowerShell 命令来实现此目的。

## 在 PowerShell 中创建警报规则

1. 登录到 Azure。
 

    	Login-AzureRmAccount -Environment $(Get-AzureRmEnvironment -Name AzureChinaCloud)


2. 获取可用订阅的列表。验证你使用的是否为正确的订阅。如果不是，请根据 `Get-AzureRmSubscription` 的输出将其设置为正确的一个。


    	Get-AzureRmSubscription
    	Get-AzureRmContext
    	Set-AzureRmContext -SubscriptionId <subscriptionid>


3.  若要列出资源组上的现有规则，请使用以下命令：

        Get-AzureRmAlertRule -ResourceGroup <myresourcegroup> -DetailedOutput


4. 若要创建规则，需要首先获得以下几条重要信息。
	- 要为其设置警报的资源的**资源 ID**
	- 可用于该资源的**指标定义**
	
	获取资源 ID 的方法是使用 Azure 门户预览。假定该资源已创建，则在门户预览中将其选中。然后，在下一个边栏选项卡中的“设置”部分下选择“属性”。“资源 ID”是下一个边栏选项卡中的一个字段。

    下面是 Web 应用的一个示例资源 ID：

    	/subscriptions/dededede-7aa0-407d-a6fb-eb20c8bd1192/resourceGroups/myresourcegroupname/providers/Microsoft.Web/sites/mywebsitename


    可以使用 `Get-AzureRmMetricDefinition` 查看特定资源的所有指标定义的列表。

    	Get-AzureRmMetricDefinition -ResourceId <resource_id>


    以下示例将生成一个表，其中包含指标名称和该指标的单位。

    
    	Get-AzureRmMetricDefinition -ResourceId <resource_id> | Format-Table -Property Name,Unit

    可以通过运行 Get-MetricDefinitions 获取 Get-AzureRmMetricDefinition 的可用选项的完整列表。

 
5. 以下示例在一个网站资源上设置警报。当在 5 分钟内持续收到任何流量以及再次在 5 分钟内未收到任何流量时，警报将触发。


    	Add-AzureRmMetricAlertRule -Name myMetricRuleWithWebhookAndEmail -Location "china east" -ResourceGroup myresourcegroup -TargetResourceId /subscriptions/dededede-7aa0-407d-a6fb-eb20c8bd1192/resourceGroups/myresourcegroupname/providers/Microsoft.Web/sites/mywebsitename -MetricName "BytesReceived" -Operator GreaterThan -Threshold 2 -WindowSize 00:05:00 -TimeAggregationOperator Total -Description "alert on any website activity"
   


6. 若要在警报触发时创建 Webhook 或发送电子邮件，请首先创建电子邮件和/或 Webhook。然后，紧随其后使用 -Actions 标记创建规则，如以下示例中所示。无法通过 PowerShell 将 Webhook 或电子邮件与已创建的规则相关联。

 

        $actionEmail = New-AzureRmAlertRuleEmail -CustomEmail myname@company.com
    	$actionWebhook = New-AzureRmAlertRuleWebhook -ServiceUri https://www.contoso.com?token=mytoken
    	
    	Add-AzureRmMetricAlertRule -Name myMetricRuleWithWebhookAndEmail -Location "china east" -ResourceGroup myresourcegroup -TargetResourceId /subscriptions/dededede-7aa0-407d-a6fb-eb20c8bd1192/resourceGroups/myresourcegroupname/providers/Microsoft.Web/sites/mywebsitename -MetricName "BytesReceived" -Operator GreaterThan -Threshold 2 -WindowSize 00:05:00 -TimeAggregationOperator Total -Actions $actionEmail, $actionWebhook -Description "alert on any website activity"



7. 若要创建在活动日志中出现特定条件时触发的警报，请使用以下形式的命令：
 	
    
    	$actionEmail = New-AzureRmAlertRuleEmail -CustomEmail myname@company.com
    	$actionWebhook = New-AzureRmAlertRuleWebhook -ServiceUri https://www.contoso.com?token=mytoken
    
    	Add-AzureRmLogAlertRule -Name myLogAlertRule -Location "china east" -ResourceGroup myresourcegroup -OperationName microsoft.web/sites/start/action -Status Succeeded -TargetResourceGroup resourcegroupbeingmonitored -Actions $actionEmail, $actionWebhook


    -OperationName 对应于活动日志中某个事件类型。示例有 *Microsoft.Compute/virtualMachines/delete* 和 *microsoft.insights/diagnosticSettings/write*。

    可以使用 PowerShell 命令 [Get-AzureRmProviderOperation](https://msdn.microsoft.com/zh-cn/library/mt603720.aspx) 获取可能的 operationName 的列表。另外，还可以使用 Azure 门户预览来查询活动日志并查找要为其创建警报的过去的特定操作。各项操作以友好名称显示在图形化日志视图中。请查看条目的 JSON 并找出 OperationName 名称。

8. 通过查看各个规则来验证是否已正确创建了警报。


        Get-AzureRmAlertRule -Name myMetricRuleWithWebhookAndEmail -ResourceGroup myresourcegroup -DetailedOutput
    
    	Get-AzureRmAlertRule -Name myLogAlertRule -ResourceGroup myresourcegroup -DetailedOutput


9. 删除警报。这些命令将删除本文中前面创建的规则。


        Remove-AzureRmAlertRule -ResourceGroup myresourcegroup -Name myrule
    	Remove-AzureRmAlertRule -ResourceGroup myresourcegroup -Name myMetricRuleWithWebhookAndEmail
    	Remove-AzureRmAlertRule -ResourceGroup myresourcegroup -Name myLogAlertRule


## 后续步骤

* 详细了解[在警报中配置 Webhook](/documentation/articles/insights-webhooks-alerts/)。
* 详细了解 [Azure 自动化 Runbook](/documentation/articles/automation-starting-a-runbook/)。
* [大致了解指标收集](/documentation/articles/insights-how-to-customize-monitoring/)以确保你的服务可用且响应迅速。

<!---HONumber=Mooncake_1107_2016-->