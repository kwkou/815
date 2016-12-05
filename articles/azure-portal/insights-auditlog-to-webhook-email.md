<properties
	pageTitle="针对 Azure 活动日志警报配置 webhook | Azure"
	description="了解如何使用活动日志警报调用 webhook。"
	authors="kamathashwin"
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
	ms.date="10/20/2016"
	wacn.date="12/05/2016"
	ms.author="ashwink"/>  


# 针对 Azure 活动日志警报配置 webhook

通过 webhook 可以将 Azure 警报通知路由到其他系统，以便进行后续处理或自定义操作。可以针对警报使用 webhook，以将警报路由到可以发送短信、记录 Bug、通过聊天/消息通知团队，或执行任意数量的其他操作的服务。本文介绍如何针对 Azure 活动日志警报设置 webhook，以及 HTTP POST 对 webhook 的有效负载情况。有关 Azure 度量值警报的设置和架构的信息，[请参阅本页](/documentation/articles/insights-webhooks-alerts/)。还可以将活动日志警报设置为激活时发送电子邮件。

>[AZURE.NOTE] 此功能目前提供预览版，因此品质和性能可能会有变化。

可以使用 [Azure PowerShell Cmdlet](/documentation/articles/insights-powershell-samples/#create-alert-rules)、[跨平台 CLI](/documentation/articles/insights-cli-samples/#work-with-alerts) 或 [Insights REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn933805.aspx) 设置活动日志警报。

## 对 webhook 进行身份验证
Webhook 可以使用以下任一方法进行身份验证：

1. **基于令牌的授权** - 保存的 webhook URI 具有令牌 ID，例如 `https://mysamplealert/webcallback?tokenid=sometokenid&someparameter=somevalue`
2.	**基本授权** - 保存的 webhook URI 具有用户名和密码，例如 `https://userid:password@mysamplealert/webcallback?someparamater=somevalue&foo=bar`

## 负载架构
POST 操作对于所有基于活动日志的警报包含以下 JSON 有效负载和架构。此架构类似于基于度量值的警报使用的架构。

```
{
        "status": "Activated",
        "context": {
                "resourceProviderName": "Microsoft.Web",
                "event": {
                        "$type": "Microsoft.WindowsAzure.Management.Monitoring.Automation.Notifications.GenericNotifications.Datacontracts.InstanceEventContext, Microsoft.WindowsAzure.Management.Mon.Automation",
                        "authorization": {
                                "action": "Microsoft.Web/sites/start/action",
                                "scope": "/subscriptions/s1/resourcegroups/rg1/providers/Microsoft.Web/sites/leoalerttest"
                        },
                        "eventDataId": "327caaca-08d7-41b1-86d8-27d0a7adb92d",
                        "category": "Administrative",
                        "caller": "myname@mycompany.com",
                        "httpRequest": {
                                "clientRequestId": "f58cead8-c9ed-43af-8710-55e64def208d",
                                "clientIpAddress": "104.43.166.155",
                                "method": "POST"
                        },
                        "status": "Succeeded",
                        "subStatus": "OK",
                        "level": "Informational",
                        "correlationId": "4a40beaa-6a63-4d92-85c4-923a25abb590",
                        "eventDescription": "",
                        "operationName": "Microsoft.Web/sites/start/action",
                        "operationId": "4a40beaa-6a63-4d92-85c4-923a25abb590",
                        "properties": {
                                "$type": "Microsoft.WindowsAzure.Management.Common.Storage.CasePreservedDictionary, Microsoft.WindowsAzure.Management.Common.Storage",
                                "statusCode": "OK",
                                "serviceRequestId": "f7716681-496a-4f5c-8d14-d564bcf54714"
                        }
                },
                "timestamp": "Friday, March 11, 2016 9:13:23 PM",
                "id": "/subscriptions/s1/resourceGroups/rg1/providers/microsoft.insights/alertrules/alertonevent2",
                "name": "alertonevent2",
                "description": "test alert on event start",
                "conditionType": "Event",
                "subscriptionId": "s1",
                "resourceId": "/subscriptions/s1/resourcegroups/rg1/providers/Microsoft.Web/sites/leoalerttest",
                "resourceGroupName": "rg1"
        },
        "properties": {
                "key1": "value1",
                "key2": "value2"
        }
}
```

|元素名称|	说明|
|---|---|
|status |用于度量值警报。对于活动日志警报始终设置为“已激活”。|
|context|事件的上下文。|
|resourceProviderName|受影响资源的资源提供程序。|
|conditionType |始终为“事件”。|
|name |警报规则的名称。|
|id |警报的资源 ID。|
|description|	创建警报期间设置的警报说明。|
|subscriptionId |Azure 订阅 ID。|
|timestamp|	处理请求的 Azure 服务生成事件的时间。|
|resourceId |受影响资源的资源 ID。|
|resourceGroupName|受影响资源的资源组的名称|
|properties |一组包含事件详细信息的 `<Key, Value>` 对（即 `Dictionary<String, String>`）。|
|event|包含有关事件的元数据的元素。|
|authorization|事件的 RBAC 属性。这些通常包括“action”、“role”和“scope”。|
|category | 事件的类别。支持的值包括：Administrative、Alert、Security、ServiceHealth、Recommendation。|
|caller|执行操作的用户的电子邮件地址（基于可用性的 UPN 声明或 SPN 声明）。对于某些系统调用可以为 null。|
|correlationId|	通常是字符串格式的 GUID。具有属于同一较大操作的 correlationId 的事件，通常共享 correlationId。|
|eventDescription |事件的静态文本说明。|
|eventDataId|事件的唯一标识符。|
|eventSource |生成事件的 Azure 服务或基础结构的名称。|
|httpRequest|	通常包括“clientRequestId”、“clientIpAddress”和“method”（HTTP 方法，例如 PUT）。|
|level|以下值之一：“Critical”、“Error”、“Warning”、“Informational”和“Verbose”。|
|operationId|通常是在与单个操作对应的事件之间共享的 GUID。|
|operationName|操作的名称。|
|properties |事件的属性。|
|status|字符串。操作的状态。常见值包括：“Started”、“In Progress”、“Succeeded”、“Failed”、“Active”、“Resolved”。|
|subStatus|	通常包含对应 REST 调用的 HTTP 状态代码。它还可能包含描述子状态的其他字符串。常见子状态值包括：OK（HTTP 状态代码：200）、Created（HTTP 状态代码：201）、Accepted（HTTP 状态代码：202）、No Content（HTTP 状态代码：204）、Bad Request（HTTP 状态代码：400）、Not Found（HTTP 状态代码：404）、Conflict（HTTP 状态代码：409）、Internal Server Error（HTTP 状态代码：500）、Service Unavailable（HTTP 状态代码：503）、Gateway Timeout（HTTP 状态代码：504）|

## 后续步骤
- [了解有关活动日志的更多信息](/documentation/articles/monitoring-overview-activity-logs/)
- [对 Azure 警报执行 Azure 自动化脚本 (Runbook)](http://go.microsoft.com/fwlink/?LinkId=627081)
- [使用逻辑应用通过 Twilio 从 Azure 警报发送短信](https://github.com/Azure/azure-quickstart-templates/tree/master/201-alert-to-text-message-with-logic-app)。本示例适用于度量值警报，但经过修改后可用于活动日志警报。
- [使用逻辑应用从 Azure 警报发送 Slack 消息](https://github.com/Azure/azure-quickstart-templates/tree/master/201-alert-to-slack-with-logic-app)。本示例适用于度量值警报，但经过修改后可用于活动日志警报。
- [使用逻辑应用从 Azure 警报将消息发送到 Azure 队列](https://github.com/Azure/azure-quickstart-templates/tree/master/201-alert-to-queue-with-logic-app)。本示例适用于度量值警报，但经过修改后可用于活动日志警报。

<!---HONumber=Mooncake_1010_2016-->