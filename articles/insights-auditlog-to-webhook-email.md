<properties
	pageTitle="Azure Insights：使用审核日志在 Azure Insights 中发送电子邮件和 webhook 警报通知。| Azure"
	description="了解如何在 Azure Insights 中使用服务审核日志条目来调用 Web URL 或发送电子邮件通知。"
	authors="kamathashwin"
	manager=""
	editor=""
	services="azure-portal"
	documentationCenter="na"/>

<tags
	ms.service="azure-portal"
	ms.date="03/30/2016"
	wacn.date="05/09/2016"/>

# 使用审核日志在 Azure Insights 中发送电子邮件和 webhook 警报通知

本文演示在审核日志事件触发 webhook 时的负载架构，并介绍如何使用相同事件发送电子邮件。

>[AZURE.NOTE] 此功能目前处于预览状态。在未来几个月内，“事件警报”基础结构和性能会得到改进。在此预览版中，此功能只能使用 Azure PowerShell 和 CLI 进行访问。将来可以使用 Azure 门户预览访问相同功能。

## Webhook
通过 webhook 可以将 Azure 警报通知路由到其他系统以便用于后处理或自定义通知。例如，将警报路由到可以处理传入 web 请求的服务，以便使用聊天或消息服务来发送 SMS、记录 bug 或通知某人。Webhook URI 必须是有效 HTTP 或 HTTPS 终结点。

## 电子邮件
电子邮件可以发送到任何有效电子邮件地址。还会通知在其中运行规则的订阅的管理员和协同管理员。

### 示例电子邮件规则
必须设置电子邮件规则、webhook 规则，然后告诉这些规则在审核日志事件发生时启动。可以在 [Azure Insights PowerShell 快速入门示例](/documentation/articles/insights-powershell-samples/#alert-on-audit-log-event)处查看使用 PowerShell 的示例。


## 身份验证
有两种身份验证 URI 形式：

1. 基于令牌的身份验证（通过将具有令牌 ID 的 webhook URI 保存为查询参数）。例如 https://mysamplealert/webcallback?tokenid=sometokenid&someparameter=somevalue
2. 基本身份验证（通过使用用户 ID 和密码）。例如 https://userid:password@mysamplealert/webcallback?someparamater=somevalue&parameter=value

## 审核日志事件通知 webhook 负载架构
当新事件可用时，有关审核日志事件的警报会对 webhook 负载中的事件元数据执行配置的 webhook。下面的示例演示 webhook 负载架构：

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
|status |始终设置为“activated”|
|context|事件的上下文|
|resourceProviderName|受影响资源的资源提供程序|
|conditionType |“Event”|
|name |警报规则的名称|
|id |警报的资源 ID|
|description|	警报创建者对警报设置的说明|
|subscriptionId |Azure 订阅 GUID|
|timestamp|	处理与事件对应的请求的 Azure 服务生成事件时的时间戳|
|resourceId |唯一标识资源的资源 ID URI|
|resourceGroupName|受影响资源的资源组名|
|properties |<Key  Value> 对集合（即字典<String  String>），包括有关事件的详细信息|
|event|包含有关事件的元数据的元素|
|authorization|捕获事件的 RBAC 属性。这些通常包括“action”、“role”和“scope”。|
|category | 事件的类别。支持的值包括：Administrative、Alert、Security、ServiceHealth、Recommendation|
|caller|执行操作的用户的电子邮件地址（基于可用性是 UPN 声明或 SPN 声明）。对于某些系统调用可以为 null。|
|correlationId|	通常是字符串格式的 GUID。具有 correlationId 的事件属于更大的相同操作，通常共享相同的 correlationId。|
|eventDescription |描述事件的静态文本|
|eventDataId|事件的唯一标识符|
|eventSource |生成事件的 Azure 服务或基础结构的名称|
|httpRequest|	通常包括“clientRequestId”、“clientIpAddress”和“method”（HTTP 方法，例如 PUT）。|
|level|以下值之一：“Critical”、“Error”、“Warning”、“Informational”和“Verbose”|
|operationId|通常是在与单个操作对应的事件之间共享的 GUID|
|operationName|操作的名称|
|properties |event 元素中的元素包含事件的属性。|
|status|描述操作状态的字符串。常用值包括：Started、In Progress、Succeeded、Failed、Active、Resolved|
|subStatus|	通常包含对应 REST 调用的 HTTP 状态代码。它还可能包含描述子状态的其他字符串。常见子状态值包括：OK（HTTP 状态代码：200）、Created（HTTP 状态代码：201）、Accepted（HTTP 状态代码：202）、No Content（HTTP 状态代码：204）、Bad Request（HTTP 状态代码：400）、Not Found（HTTP 状态代码：404）、Conflict（HTTP 状态代码：409）、Internal Server Error（HTTP 状态代码：500）、Service Unavailable（HTTP 状态代码：503）、Gateway Timeout（HTTP 状态代码：504）|

<!---HONumber=Mooncake_0503_2016-->