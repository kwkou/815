<properties
    pageTitle="为 Azure 服务创建警报 - Azure 门户 | Azure"
    description="满足指定的条件时，触发电子邮件、通知、调用网站 URL (webhook) 或自动执行。"
    author="rboucher"
    manager="carmonm"
    editor=""
    services="monitoring-and-diagnostics"
    documentationcenter="monitoring-and-diagnostics"
    translationtype="Human Translation" />
<tags
    ms.assetid="f7457655-ced6-4102-a9dd-7ddf2265c0e2"
    ms.service="monitoring-and-diagnostics"
    ms.workload="na"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="09/23/2016"
    wacn.date="05/02/2017"
    ms.author="robb"
    ms.sourcegitcommit="78da854d58905bc82228bcbff1de0fcfbc12d5ac"
    ms.openlocfilehash="fceb0beb8d35648c8787189c7c321a6cfaa757f3"
    ms.lasthandoff="04/22/2017" />

# <a name="create-metric-alerts-in-azure-monitor-for-azure-services---azure-portal"></a>在 Azure Monitor 中为 Azure 服务创建指标警报 - Azure 门户
> [AZURE.SELECTOR]
- [门户预览](/documentation/articles/insights-alerts-portal/)
- [PowerShell](/documentation/articles/insights-alerts-powershell/)
- [CLI](/documentation/articles/insights-alerts-command-line-interface/)

## <a name="overview"></a>概述
本文将展示如何使用 Azure 门户预览设置 Azure 指标警报。

可以根据监视指标或事件接收 Azure 服务的警报。

- **指标值** - 当指定指标的值在任一方向越过了指定的阈值时警报将触发。也就是说，当条件先是满足以及之后不再满足该条件时，警报都会触发。
- **活动日志事件** - 警报可以在发生 *每个* 事件时都触发，也可以仅在发生特定数量的事件时触发。

可以配置指标警报，在其触发时执行以下操作：

- 向服务管理员和共同管理员发送电子邮件通知
- 向指定的其他电子邮件地址发送电子邮件。
- 调用 Webhook
- 开始执行 Azure Runbook（仅在 Azure 门户预览中可行）

可以使用以下工具配置和获取关于指标警报的信息：

- [Azure 门户预览](/documentation/articles/insights-alerts-portal/)
- [PowerShell](/documentation/articles/insights-alerts-powershell/)
- [命令行接口 (CLI)](/documentation/articles/insights-alerts-command-line-interface/) 
- [Azure 监视器 REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn931945.aspx)

## <a name="create-an-alert-rule-on-a-metric-with-the-azure-portal"></a>使用 Azure 门户创建指标的警报规则

1. 在[门户预览](https://portal.azure.cn/)中，找到想要监视的资源并选中它。

2. 在“监视”部分下，选择“警报”或“警报规则”。 对于不同的资源，文本和图标可能会略有不同。  

	![监视](./media/insights-alerts-portal/AlertRulesButton.png)  

3. 选择“添加通知”命令，并填写字段。

	![添加警报](./media/insights-alerts-portal/AddAlertOnlyParamsPage.png)  

4. **命名**警报规则，并选择也在通知电子邮件中显示的“说明”。
5. 选择想要监视的“指标”为该指标选择一个“条件”和“阈值”。 还选择了触发警报前指标规则必须满足的时间**段**。 例如，如果使用了时间“PT5M”，并且警报监视使用率高于  80% 的 CPU，则当 CPU 的使用率持续高于 80% 达 5 分钟时，该警报将触发。 在发生第一次触发后，当 CPU 使用率保持低于 80% 达到 5 分钟时，该触发器将再次触发。 每 1 分钟对 CPU 进行一次测量。   

6. 如果触发警报时希望向管理员和共同管理员发送电子邮件，则选择“向所有者发送电子邮件...”。

7. 触发警报时，如果希望其他电子邮件收到通知，请将其添加到“其他管理员电子邮件”字段下。 用分号隔开多个电子邮件 - *email@contoso.com;email2@contoso.com* 

8. 触发警报时，如果希望调用有效的 URI，请将其放入“Webhook”字段。

9. 如果使用 Azure 自动化时，则触发警报时可选择要运行的 Runbook。 

10. 警报创建完成后，选择“确定”。   

几分钟后，警报将处于活动状态，并按前面所述进行触发。

## <a name="managing-your-alerts"></a>管理警报

在创建警报后，可以选择警报并且：

- 查看其中显示了指标阈值和前一天的实际值的图形。
- 编辑或删除警报。
- 如果想要暂时停止或恢复接收该警报的通知，可**禁用**或**启用**它。 

## <a name="next-steps"></a>后续步骤

* [获取 Azure 监视概述](/documentation/articles/monitoring-overview/)，包括可收集和监视的信息的类型。
* 了解[在警报中配置 Webhook](/documentation/articles/insights-webhooks-alerts/)的详细信息。
* 了解关于 [Azure 自动化 Runbook](/documentation/articles/automation-starting-a-runbook/) 的详细信息。
* 获取[指标集合概述](/documentation/articles/insights-how-to-customize-monitoring/)以确保你的服务可用且响应迅速。

<!--Update_Description:update wording -->