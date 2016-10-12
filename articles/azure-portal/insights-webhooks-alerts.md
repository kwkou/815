<properties
	pageTitle="如何配置 Azure 警报以发送到其他系统"
	description="将 Azure 警报重新路由到其他非 Azure 系统。"
	authors="kamathashwin"
	manager=""
	editor=""
	services="azure-portal"
	documentationCenter="na"/>

<tags
	ms.service="azure-portal"
	ms.date="02/16/2016"
	wacn.date="05/09/2016"/>

# 如何为警报配置 webhook

利用 webhook，用户可以将 Azure 警报通知路由到其他系统以便用于后处理或自定义通知。例如，将警报路由到可以处理传入 Web 请求的服务，以便通过聊天/消息服务等来发送 SMS、记录 Bug 或通知某个团队。

webhook URI 必须是有效 HTTP 或 HTTPS 终结点。Azure 警报服务将在指定的 webhook 上执行 POST 操作，传递特定的 JSON 负载和架构。

## 通过门户配置 webhook

在 [Azure 门户预览](https://portal.azure.cn/)的“创建/更新警报”屏幕上，可以添加或更新 webhook URI。

![添加警报规则](./media/insights-webhooks-alerts/Alertwebhook.png)


## 身份验证

可以通过以下两种类型之一进行身份验证︰

1. **基于令牌的身份验证** — 在这种情况下，将使用令牌 ID 保存 webhook URI，例如 https://mysamplealert/webcallback?tokenid=sometokenid&someparameter=somevalue
2.	**基本身份验证** — 使用用户 ID 和密码︰
在这种情况下，会将 webhook URI 保存为 https://userid:password@mysamplealert/webcallback?someparamater=somevalue&foo=bar

## 负载架构

POST 操作将为所有基于指标的警报包含以下 JSON 负载和架构。

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
            "resourceRegion": "Chinaeast",
            "portalLink": “https://portal.azure.cn/#resource/subscriptions/s1/resourceGroups/useast/providers/microsoft.foo/sites/mysite1”                                
},
"properties": {
              "key1": "value1",
              "key2": "value2"
              }
}
```

>[AZURE.NOTE] 在下一次刷新时，我们将在事件上添加对警报的支持 (“conditionType” : “Event”)


| 字段 | 必需？ | 一组固定的值？ | 说明 |
| :-------------| :-------------   | :-------------   | :-------------   |
|status|Y|“Activated”, “Resolved”|这就是如何确定是哪种警报类型。Azure 针对设置的条件自动发送已激活并且已解决的警报。|
|上下文| Y | | 警报上下文|
|timestamp| Y | | 触发警报的时间。从诊断存储中读取该指标后，立即触发警报。|
|id | Y | | 每个警报规则都具有一个唯一的 ID。|
|name|Y | |
|description |Y | |有关警报的描述。|
|conditionType |Y |“Metric”, “Event” |支持两种类型的警报。一种基于指标，另一种基于事件。将来，我们将支持针对事件的警报，因此使用此值来检查警报是基于指标还是事件|
|条件 |Y | |这将使用特定字段来依据 conditionType 进行检查|
|metricName |用于指标警报 | |定义规则监视对象的指标的名称。|
|metricUnit |用于指标警报 |"Bytes", "BytesPerSecond" , "Count" , "CountPerSecond" , "Percent", "Seconds"|	 指标中允许使用的单位。允许的值：https://msdn.microsoft.com/zh-cn/library/microsoft.azure.insights.models.unit.aspx|
|metricValue |用于指标警报 | |导致警报的实际指标值|
|阈值 |用于指标警报 | |激活警报的阈值|
|windowSize |用于指标警报 | |用于根据阈值监视警报活动的时间段。必须介于 5 分钟到 1 天之间。ISO 8601 持续时间格式。|
|timeAggregation |用于指标警报 |"Average", "Last" , "Maximum" , "Minimum" , "None", "Total" |	随着时间推移，收集的数据应如何组合。默认值为 Average。允许的值：https://msdn.microsoft.com/zh-cn/library/microsoft.azure.insights.models.aggregationtype.aspx|
|operator |用于指标警报 | |用于比较数据和阈值的运算符。|
|subscriptionId |Y | |Azure 订阅 GUID|
|resourceGroupName |Y | |受影响资源的资源组名|
|resourceName |Y | |受影响资源的资源名|
|resourceType |Y | |受影响资源的资源类型|
|resourceId |Y | |URI 中唯一标识该资源的资源 ID|
|resourceRegion |Y | |受影响资源的区域/位置|
|portalLink |Y | |指向资源摘要页的直接 azure 门户预览链接|
|properties |N |可选 |<Key  Value>对集合（即字典<String  String>），包括有关事件的详细信息。properties 字段是可选的。在自定义用户界面或基于逻辑应用的工作流中，用户可以输入键/值，该键/值可通过负载传递。将自定义属性传递回 webhook 的替代方法是通过 webhook URI 本身（作为查询参数）|


>[AZURE.NOTE] 不能通过门户预览使用属性字段。在我们即将发布的 Insights SDK 版本中，可以通过警报 API 设置属性。

## 后续步骤


若要了解以编程方式创建 webhook 的方法，请参阅[使用 Azure Insights SDK (C#) 借助 Webhook 创建警报](https://code.msdn.microsoft.com/Create-Azure-Alerts-with-b938077a)。

设置完 webhook 和警报后，可探索其他选项来启动自动化脚本。

[执行 Azure 自动化脚本 (Runbook)](http://go.microsoft.com/fwlink/?LinkId=627081)

使用 Azure 警报，以将消息发送到其他服务。使用以下示例模板来开始。

[使用逻辑应用来通过 Twilio API 发送短信](https://github.com/Azure/azure-quickstart-templates/tree/master/201-alert-to-text-message-with-logic-app)

[使用逻辑应用来发送 Slack 消息](https://github.com/Azure/azure-quickstart-templates/tree/master/201-alert-to-slack-with-logic-app)

[使用逻辑应用来将消息发送到 Azure 队列](https://github.com/Azure/azure-quickstart-templates/tree/master/201-alert-to-queue-with-logic-app)

<!---HONumber=Mooncake_0503_2016-->