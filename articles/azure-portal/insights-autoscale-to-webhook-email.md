<properties
	pageTitle="Azure Insights：使用自动缩放操作发送电子邮件和 webhook 警报通知。| Azure"
	description="了解如何在 Azure Insights 中使用自动缩放操作来调用 Web URL 或发送电子邮件通知。"
	authors="kamathashwin"
	manager=""
	editor=""
	services="azure-portal"
	documentationCenter="na"/>

<tags
	ms.service="monitoring-and-diagnostics"
	ms.workload="na"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="07/19/2016"
	wacn.date="07/18/2016"
	ms.author="ashwink"/>

# 使用自动缩放操作在 Azure Insights 中发送电子邮件和 webhook 警报通知

本文演示如何设置触发器，以便你可以在 Azure 中基于自动缩放操作调用特定 Web URL 或发送电子邮件。

## Webhook
通过 webhook 可以将 Azure 警报通知路由到其他系统以便用于后处理或自定义通知。例如，将警报路由到可以处理传入 web 请求的服务，以便使用聊天或消息服务等来发送 SMS、记录 bug、通知团队。Webhook URI 必须是有效 HTTP 或 HTTPS 终结点。

## 电子邮件
电子邮件可以发送到任何有效电子邮件地址。还会通知在其中运行规则的订阅的管理员和协同管理员。


## 云服务和 Web Apps
可以从 Azure 门户预览选择加入云服务和服务器场 (Web Apps)。

- 选择**缩放依据**指标。

    ![缩放依据](./media/insights-autoscale-to-webhook-email/insights-autoscale-scale-by.png)

## 虚拟机规模集
对于较新的基于 ARM 的虚拟机（虚拟机规模集），可以使用 REST API、PowerShell 和 CLI 对此进行配置。门户预览界面尚不可用。


## webhook 中的身份验证
有两种身份验证 URI 形式：

1. 基于令牌的身份验证：将具有令牌 ID 的 webhook URI 保存为查询参数。例如 https://mysamplealert/webcallback?tokenid=sometokenid&someparameter=somevalue
2. 基本身份验证：使用用户 ID 和密码。例如 https://userid:password@mysamplealert/webcallback?someparamater=somevalue&parameter=value

## 自动缩放通知 webhook 负载架构
生成自动缩放通知时，以下元数据会包含在 webhook 负载中：

```
{
        "version": "1.0",
        "status": "Activated",
        "operation": "Scale In",
        "context": {
                "timestamp": "2016-03-11T07:31:04.5834118Z",
                "id": "/subscriptions/s1/resourceGroups/rg1/providers/microsoft.insights/autoscalesettings/myautoscaleSetting",
                "name": "myautoscaleSetting",
                "details": "Autoscale successfully started scale operation for resource 'MyCSRole' from capacity '3' to capacity '2'",
                "subscriptionId": "s1",
                "resourceGroupName": "rg1",
                "resourceName": "MyCSRole",
                "resourceType": "microsoft.classiccompute/domainnames/slots/roles",
                "resourceId": "/subscriptions/s1/resourceGroups/rg1/providers/microsoft.classicCompute/domainNames/myCloudService/slots/Production/roles/MyCSRole",
                "portalLink": "https://management.chinacloudapi.cn/#resource/subscriptions/s1/resourceGroups/rg1/providers/microsoft.classicCompute/domainNames/myCloudService",
                "oldCapacity": "3",
                "newCapacity": "2"
        },
        "properties": {
                "key1": "value1",
                "key2": "value2"
        }
}
```


|字段 |必需？|	说明|
|---|---|---|
|status |是 |指示生成自动缩放操作的状态|
|operation|	是 |对于实例的增加，它会是“Scale Out”，对于实例的减少，它会是“Scale In”|
|context|	是 |自动缩放操作上下文|
|timestamp|	是 |触发自动缩放操作时的时间戳|
|id |是|	自动缩放设置的 ARM (Azure Resource Manager) ID|
|name |是|	自动缩放设置的名称|
|details|	是 |自动缩放服务执行的操作和实例计数的更改的说明|
|subscriptionId|	是 |所缩放的目标资源的订阅 ID|
|resourceGroupName|	是|	所缩放的目标资源的资源组名|
|resourceName |是|	所缩放的目标资源的名称|
|resourceType |是|	三个支持的值：“microsoft.classiccompute/domainnames/slots/roles”— 云服务角色、“microsoft.compute/virtualmachinescalesets”— 虚拟机规模集和“Microsoft.Web/serverfarms”— Web 应用|
|resourceId |是|所缩放的目标资源的 ARM ID|
|portalLink |是 |指向目标资源摘要页的 Azure 门户预览链接|
|oldCapacity|	是 |自动缩放执行缩放操作时的当前（旧）实例计数|
|newCapacity|	是 |自动缩放将资源缩放到的新实例计数|
|Properties|	否|	可选。<Key  Value> 对集合（例如，字典 <String  String>）。properties 字段是可选的。在自定义用户界面或基于逻辑应用的工作流中，可以输入可使用负载传递的键和值。将自定义属性传递回传出 webhook 调用的替代方法是使用 webhook URI 本身（作为查询参数）|

<!---HONumber=Mooncake_0503_2016-->