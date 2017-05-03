<properties
    pageTitle="为 Azure 服务创建警报 - PowerShell | Azure"
    description="满足指定的条件时，触发电子邮件、通知、调用网站 URL (webhook) 或自动执行。"
    author="rboucher"
    manager="carmonm"
    editor=""
    services="monitoring-and-diagnostics"
    documentationcenter="monitoring-and-diagnostics"
    translationtype="Human Translation" />
<tags
    ms.assetid="d26ab15b-7b7e-42a9-81c8-3ce9ead5d252"
    ms.service="monitoring-and-diagnostics"
    ms.workload="na"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="10/20/2016"
    wacn.date="05/02/2017"
    ms.author="robb"
    ms.sourcegitcommit="78da854d58905bc82228bcbff1de0fcfbc12d5ac"
    ms.openlocfilehash="7fff31a6147055f39abef9406865208e82b0d71a"
    ms.lasthandoff="04/22/2017" />

# <a name="create-metric-alerts-in-azure-monitor-for-azure-services---powershell"></a>在 Azure Monitor 中为 Azure 服务创建指标警报 - PowerShell
> [AZURE.SELECTOR]
- [门户预览](/documentation/articles/insights-alerts-portal/)
- [PowerShell](/documentation/articles/insights-alerts-powershell/)
- [CLI](/documentation/articles/insights-alerts-command-line-interface/)

## <a name="overview"></a>概述
本文将展示如何使用 PowerShell 设置 Azure 指标警报。  

可以根据监视指标或事件接收 Azure 服务的警报。

- **指标值** - 当指定指标的值在任一方向越过了指定的阈值时警报将触发。也就是说，当条件先是满足以及之后不再满足该条件时，警报都会触发。
- **活动日志事件** - 警报可以在发生*每个*事件时都触发，也可以仅在发生特定数量的事件时触发。

可以配置指标警报，在其触发时执行以下操作：

- 向服务管理员和共同管理员发送电子邮件通知
- 向指定的其他电子邮件地址发送电子邮件。
- 调用 Webhook
- 开始执行 Azure Runbook（仅在 Azure 门户预览中可行）

可使用以下项配置并获取有关警报规则的信息 

- [Azure 门户预览](/documentation/articles/insights-alerts-portal/)
- [PowerShell](/documentation/articles/insights-alerts-powershell/)
- [命令行接口 (CLI)](/documentation/articles/insights-alerts-command-line-interface/) 
- [Azure 监视器 REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn931945.aspx)

若要获得更多信息，始终可以通过键入 ```get-help``` 然后键入希望获取其帮助信息的 PowerShell 命令来实现此目的。 

## <a name="create-alert-rules-in-powershell"></a>在 PowerShell 中创建警报规则

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
	
    获取资源 ID 的一种方法是使用 Azure 门户预览。 假设已创建该资源，在门户中选中它。 然后在下一个边栏选项卡中，选择“设置”分区下的“属性”。 **资源 ID** 是下一个边栏选项卡中的字段。 

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

7. 通过查看各个规则来验证是否已正确创建了警报。


        Get-AzureRmAlertRule -Name myMetricRuleWithWebhookAndEmail -ResourceGroup myresourcegroup -DetailedOutput
    
    	Get-AzureRmAlertRule -Name myLogAlertRule -ResourceGroup myresourcegroup -DetailedOutput


9. 删除警报。这些命令将删除本文中前面创建的规则。


        Remove-AzureRmAlertRule -ResourceGroup myresourcegroup -Name myrule
    	Remove-AzureRmAlertRule -ResourceGroup myresourcegroup -Name myMetricRuleWithWebhookAndEmail
    	Remove-AzureRmAlertRule -ResourceGroup myresourcegroup -Name myLogAlertRule

## <a name="next-steps"></a>后续步骤

* [获取 Azure 监视概述](/documentation/articles/monitoring-overview/)，包括可收集和监视的信息的类型。
* 了解[在警报中配置 Webhook](/documentation/articles/insights-webhooks-alerts/)的详细信息。

* 了解关于 [Azure 自动化 Runbook](/documentation/articles/automation-starting-a-runbook/) 的详细信息。
* 获取[指标集合概述](/documentation/articles/insights-how-to-customize-monitoring/)以确保你的服务可用且响应迅速。

<!--Update_Description:update wording and link references-->