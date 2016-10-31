<properties
	pageTitle="针对 Azure 度量值警报配置 webhook | Azure"
	description="将 Azure 警报重新路由到其他非 Azure 系统。"
	authors="kamathashwin"
	manager=""
	editor=""
	services="monitoring-and-diagnostics"
	documentationCenter="monitoring-and-diagnostics"/>  


<tags
	ms.service="monitoring-and-diagnostics"
	ms.workload="na"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/15/2016"
	ms.author="ashwink"
	wacn.date="10/17/2016"/>  


# 针对 Azure 度量值警报配置 webhook

通过 webhook 可以将 Azure 警报通知路由到其他系统，以便进行后续处理或自定义操作。可以针对警报使用 webhook，以将警报路由到可以发送短信、记录 Bug、通过聊天/消息通知团队，或执行任意数量的其他操作的服务。本文介绍如何针对 Azure 度量值警报设置 webhook，以及 HTTP POST 对 Webhook 的有效负载情况。有关 Azure 活动日志警报（事件警报）的设置和架构的信息，[请参阅本页](/documentation/articles/insights-auditlog-to-webhook-email/)。

Azure 警报会将警报内容以 JSON 格式（架构定义如下）HTTP POST 到创建警报时提供的 webhook URI。此 URI 必须是有效的 HTTP 或 HTTPS 终结点。激活警报时，Azure 会针对每个请求发布一个条目。

## 通过门户配置 webhook

可在[门户预览](https://portal.azure.cn/)的“创建/更新警报”屏幕上添加或更新 webhook URI。

![添加警报规则](./media/insights-webhooks-alerts/Alertwebhook.png)  


还可以使用 [Azure PowerShell Cmdlet](/documentation/articles/insights-powershell-samples/#create-alert-rules)、[跨平台 CLI](/documentation/articles/insights-cli-samples/#work-with-alerts) 或 [Insights REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn933805.aspx) 将警报配置为发布到 webhook URI。

## 对 webhook 进行身份验证

Webhook 可以使用以下任一方法进行身份验证：

1. **基于令牌的授权** - 保存的 webhook URI 具有令牌 ID，例如 `https://mysamplealert/webcallback?tokenid=sometokenid&someparameter=somevalue`
2.	**基本授权** - 保存的 webhook URI 具有用户名和密码，例如 `https://userid:password@mysamplealert/webcallback?someparamater=somevalue&foo=bar`

## 负载架构

POST 操作对于所有基于度量值的警报包含以下 JSON 有效负载和架构。

```
{
"status": "Activated",
"context": {
            "timestamp": "2015-08-14T22:26:41.9975398Z",
            "id": "/subscriptions/s1/resourceGroups/useast/providers/microsoft.insights/alertrules/ruleName1",
            "name": "ruleName1",
            "description": "some description",
            "conditionType": "Metric",
            "condition": {
                        "metricName": "Requests",
                        "metricUnit": "Count",
                        "metricValue": "10",
                        "threshold": "10",
                        "windowSize": "15",
                        "timeAggregation": "Average",
                        "operator": "GreaterThanOrEqual"
                },
            "subscriptionId": "s1",
            "resourceGroupName": "useast",                                
            "resourceName": "mysite1",
            "resourceType": "microsoft.foo/sites",
            "resourceId": "/subscriptions/s1/resourceGroups/useast/providers/microsoft.foo/sites/mysite1",
            "resourceRegion": "chinanorth",
            "portalLink": "https://portal.azure.cn/#resource/subscriptions/s1/resourceGroups/useast/providers/microsoft.foo/sites/mysite1"
},
"properties": {
              "key1": "value1",
              "key2": "value2"
              }
}
```


| 字段 | 必需 | 一组固定的值 | 说明 |
| :-------------| :-------------   | :-------------   | :-------------   |
|status|Y|“Activated”, “Resolved”|以设置的条件为基础的警报的状态。|
|上下文| Y | | 警报上下文。|
|timestamp| Y | | 触发警报的时间。|
|id | Y | | 每个警报规则都具有一个唯一的 ID。|
|name |Y | | 警报名称。|
|description |Y | |警报的说明。|
|conditionType |Y |“Metric”, “Event” |支持两种类型的警报。一种基于度量值条件，另一种基于活动日志中的事件。使用此值可检查警报是基于度量值还是基于事件。|
|条件 |Y | | 根据 conditionType 要检查的特定字段。|
|metricName |用于指标警报 | |定义规则监视对象的指标的名称。|
|metricUnit |用于指标警报 |“Bytes”、“BytesPerSecond”、“Count”、“CountPerSecond”、“Percent”、“Seconds”|	 指标中允许使用的单位。[允许的值列于此处](https://msdn.microsoft.com/zh-cn/library/microsoft.azure.insights.models.unit.aspx)。|
|metricValue |用于指标警报 | |导致警报的实际度量值。|
|阈值 |用于指标警报 | |会激活警报的阈值。|
|windowSize |用于指标警报 | |用于根据阈值监视警报活动的时间段。必须介于 5 分钟到 1 天之间。ISO 8601 持续时间格式。|
|timeAggregation |用于指标警报 |“Average”、“Last”、“Maximum”、“Minimum”、“None”、“Total” |	随着时间推移，收集的数据应如何组合。默认值为 Average。[允许的值列于此处](https://msdn.microsoft.com/zh-cn/library/microsoft.azure.insights.models.aggregationtype.aspx)。|
|operator |用于指标警报 | |用于比较当前度量值数据和所设阈值的运算符。|
|subscriptionId |Y | |Azure 订阅 ID。|
|resourceGroupName |Y | |受影响资源的资源组的名称。|
|resourceName |Y | |受影响资源的资源名|
|resourceType |Y | |受影响资源的资源类型|
|resourceId |Y | |受影响资源的资源 ID。|
|resourceRegion |Y | |受影响资源的区域或位置。|
|portalLink |Y | |指向门户资源摘要页的直接链接。|
|属性 |N |可选 |一组包含事件详细信息的 `<Key, Value>` 对（即 `Dictionary<String, String>`）。properties 字段是可选的。在自定义 UI 或基于逻辑应用的工作流中，用户可以输入键/值，该键/值可通过有效负载传递。将自定义属性传递回 webhook 的替代方法是通过 webhook URI 本身（作为查询参数）|


>[AZURE.NOTE] 仅可以使用 [Insights REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn933805.aspx) 设置属性字段。

## 后续步骤

- [对 Azure 警报执行 Azure 自动化脚本 (Runbook)](http://go.microsoft.com/fwlink/?LinkId=627081)
- [使用逻辑应用通过 Twilio 从 Azure 警报发送短信](https://github.com/Azure/azure-quickstart-templates/tree/master/201-alert-to-text-message-with-logic-app)
- [使用逻辑应用从 Azure 警报发送 Slack 消息](https://github.com/Azure/azure-quickstart-templates/tree/master/201-alert-to-slack-with-logic-app)
- [使用逻辑应用从 Azure 警报将消息发送到 Azure 队列](https://github.com/Azure/azure-quickstart-templates/tree/master/201-alert-to-queue-with-logic-app)

<!---HONumber=Mooncake_1010_2016-->