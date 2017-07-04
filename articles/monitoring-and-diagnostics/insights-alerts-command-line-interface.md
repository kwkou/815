<properties
    pageTitle="为 Azure 服务创建警报 - 跨平台 CLI | Azure"
    description="满足指定的条件时，触发电子邮件、通知、调用网站 URL (webhook) 或自动执行。"
    author="rboucher"
    manager="carmonm"
    editor=""
    services="monitoring-and-diagnostics"
    documentationcenter="monitoring-and-diagnostics"
    translationtype="Human Translation" />
<tags
    ms.assetid="5c6a2d27-7dcc-4f89-8752-9bb31b05ff35"
    ms.service="monitoring-and-diagnostics"
    ms.workload="na"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="10/24/2016"
    wacn.date="05/02/2017"
    ms.author="robb"
    ms.sourcegitcommit="78da854d58905bc82228bcbff1de0fcfbc12d5ac"
    ms.openlocfilehash="94754c6f0b12312734e8f150fbecb05aec8cce87"
    ms.lasthandoff="04/22/2017" />

# <a name="create-metric-alerts-in-azure-monitor-for-azure-services---cross-platform-cli"></a>在 Azure Monitor 中为 Azure 服务创建指标警报 - 跨平台 CLI
> [AZURE.SELECTOR]
- [门户](/documentation/articles/insights-alerts-portal/)
- [PowerShell](/documentation/articles/insights-alerts-powershell/)
- [CLI](/documentation/articles/insights-alerts-command-line-interface/)

## <a name="overview"></a>概述
本文说明如何使用跨平台命令行界面 (CLI) 设置 Azure 指标警报。

> [AZURE.NOTE]
> “Azure Insights”在 2016 年 9 月 25 日后称为 Azure Monitor。 但是，命名空间和以下命令中仍然包含“insights”。
> 
> 

可以根据监视指标或事件接收 Azure 服务的警报。

- **指标值** - 当指定指标的值在任一方向越过了指定的阈值时警报将触发。也就是说，当条件先是满足以及之后不再满足该条件时，警报都会触发。
- **活动日志事件** - 警报可以在发生每个事件时都触发，也可以仅在发生特定数量的事件时触发。

可以配置指标警报，在其触发时执行以下操作：

- 向服务管理员和共同管理员发送电子邮件通知
- 向指定的其他电子邮件地址发送电子邮件。
- 调用 Webhook
- 开始执行 Azure Runbook（目前仅在 Azure 门户中可行）

可以使用以下工具配置和获取关于指标警报的信息：

- [Azure 门户](/documentation/articles/insights-alerts-portal/)
- [PowerShell](/documentation/articles/insights-alerts-powershell/)
- [命令行接口 (CLI)](/documentation/articles/insights-alerts-command-line-interface/) 
- [Azure 监视器 REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn931945.aspx)

可以随时通过键入命令并在结尾处放置 -help 来获取命令的帮助。 例如：

	azure insights alerts -help
	azure insights alerts actions email create -help

## <a name="create-alert-rules-using-the-cli"></a>使用 CLI 创建警报规则
1. 执行先决条件并登录到 Azure。 请参阅 [Azure 监视器 CLI 示例](/documentation/articles/insights-cli-samples/)。 简而言之，就是安装 CLI 并运行以下命令。 通过它们可进行登录，查看正在使用的订阅，并为运行 Azure 监视器命令做好准备。

    	azure login -e AzureChinaCloud
    	azure account show
    	azure config mode arm 


2.  若要列出资源组的现有规则，请使用以下形式，**azure insights alerts rule list** *[options] &lt;resourceGroup&gt;*

        azure insights alerts rule list myresourcegroupname

3. 若要创建一条规则，首先需要有以下几条重要信息。 
	- 要为其设置警报的资源的**资源 ID**
	- 可用于该资源的**指标定义**
	
    获取资源 ID 的一种方法是使用 Azure 门户。 假设已创建该资源，在门户中选中它。 然后在下一个边栏选项卡中，选择“设置”分区下的“属性”。 *资源 ID* 是下一个边栏选项卡中的字段。 
    下面是 Web 应用的一个示例资源 ID：
  

    	/subscriptions/dededede-7aa0-407d-a6fb-eb20c8bd1192/resourceGroups/myresourcegroupname/providers/Microsoft.Web/sites/mywebsitename

    若要获取上一个资源示例的这些指标的可用指标和单位列表，请使用以下 CLI命令：  


    	azure insights metrics list /subscriptions/dededede-7aa0-407d-a6fb-eb20c8bd1192/resourceGroups/myresourcegroupname/providers/Microsoft.Web/sites/mywebsitename PT1M 

     PT1M 是可用度量的粒度（1分钟间隔）。 使用不同的粒度可以提供不同的指标选项。
4. 若要创建基于指标的警报规则，请使用以下形式的命令：
 
	**azure insights alerts rule metric set** *[options] &lt;ruleName&gt; &lt;location&gt; &lt;resourceGroup&gt; &lt;windowSize&gt; &lt;operator&gt; &lt;threshold&gt; &lt;targetResourceId&gt; &lt;metricName&gt; &lt;timeAggregationOperator&gt;*
	
    以下示例设置了一个关于网站资源的警报。 当在 5 分钟内持续收到任何流量以及再次在 5 分钟内未收到任何流量时，警报将触发。 


    	azure insights alerts rule metric set myrule eastus myreasourcegroup PT5M GreaterThan 2 /subscriptions/dededede-7aa0-407d-a6fb-eb20c8bd1192/resourceGroups/myresourcegroupname/providers/Microsoft.Web/sites/mywebsitename BytesReceived Total


5. 若要在指标警报触发时创建 Webhook 或发送电子邮件，请首先创建电子邮件和/或 Webhook。 然后紧随其后创建规则。 无法使用 CLI 将 Webhook 或电子邮件与已创建的规则相关联。
 

    	azure insights alerts actions email create --customEmails myemail@contoso.com
    
    	azure insights alerts actions webhook create https://www.contoso.com
    
    	azure insights alerts rule metric set myrulewithwebhookandemail chinaeast myreasourcegroup PT5M GreaterThan 2 /subscriptions/dededede-7aa0-407d-a6fb-eb20c8bd1192/resourceGroups/myresourcegroupname/providers/Microsoft.Web/sites/mywebsitename BytesReceived Total

6. 可以通过查看各个规则来验证是否已正确创建了警报。


        azure insights alerts rule list myresourcegroup --ruleName myrule

8. 若要删除规则，请使用以下格式的命令： 

	**insights alerts rule delete** [options] &lt;resourceGroup&gt; &lt;ruleName&gt;

    这些命令将删除在本文前面创建的规则。

    
        azure insights alerts rule delete myresourcegroup myrule
        azure insights alerts rule delete myresourcegroup myrulewithwebhookandemail
        azure insights alerts rule delete myresourcegroup myActivityLogRule

## <a name="next-steps"></a>后续步骤

* [获取 Azure 监视概述](/documentation/articles/monitoring-overview/)，包括可收集和监视的信息的类型。
* 了解[在警报中配置 Webhook](/documentation/articles/insights-webhooks-alerts/)的详细信息。
* 了解关于 [Azure 自动化 Runbook](/documentation/articles/automation-starting-a-runbook/) 的详细信息。
* 获取[指标集合概述](/documentation/articles/insights-how-to-customize-monitoring/)以确保你的服务可用且响应迅速。
<!---HONumber=Mooncake_0227_2017-->
<!--Update_Description:update wording and link references -->