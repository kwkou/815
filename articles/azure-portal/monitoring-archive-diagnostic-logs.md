<properties
	pageTitle="存档 Azure 诊断日志 | Azure"
	description="了解如何存档 Azure 诊断日志，将其长期保留在存储帐户中。"
	authors="johnkemnetz"
	manager="rboucher"
	editor=""
	services="monitoring-and-diagnostics"
	documentationCenter="monitoring-and-diagnostics"/>

<tags
	ms.service="monitoring-and-diagnostics"
	ms.workload="na"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="08/26/2016"
	ms.author="johnkem"
	wacn.date="10/17/2016"/>  


# 存档 Azure 诊断日志
本文介绍如何使用 Azure 门户、PowerShell Cmdlet、CLI 或 REST API 将 [Azure 诊断日志](/documentation/articles/monitoring-overview-of-diagnostic-logs/)存档到存储帐户中。此选项适用于实施可选保留策略的诊断日志，将其保留下来进行审核、静态分析或备份。

## 先决条件
在开始之前，需要[创建存储帐户](/documentation/articles/storage-create-storage-account/#create-a-storage-account)，将诊断日志存档到其中。强烈建议用户不要使用其中存储了其他非监视数据的现有存储帐户，以便更好地控制监视数据所需的访问权限。但是，如果还要将活动日志和诊断指标存档到存储帐户，则也可将该存储帐户用于诊断日志，使得所有监视数据都位于一个中心位置。所使用的存储帐户必须是一个通用存储帐户，而不是一个 blob 存储帐户。

## 诊断设置
若要使用下述任意方法存档诊断日志，可针对特定资源设置“诊断设置”。资源的诊断设置定义已存储或流式传输的日志的类别，以及输出 - 存储帐户和/或事件中心。它还定义存储在存储帐户中的每个日志类别的事件的保留策略（需保留的天数）。如果将保留策略设置为零，则会无限期（即永久）存储该日志类别的事件。如果不需要无限期存储，可将保留策略设置为 1 到 2147483647 之间的任意天数。[单击此处详细了解诊断设置](/documentation/articles/monitoring-overview-of-diagnostic-logs/#diagnostic-settings)。

## 使用门户存档诊断日志

1. 在门户中，单击需要启用诊断日志存档功能的资源的资源边栏选项卡。
2. 在资源设置菜单的“监视”部分，选择“诊断”。

    ![资源菜单的“监视”部分](./media/monitoring-archive-diagnostic-logs/diag-log-monitoring-sec.png)  

3. 选中与“导出到存储帐户”对应的框，然后选择存储帐户。（可选）使用“保留期(天)”滑块设置这些日志的保留天数。如果保留期为 0 天，则会无限期存储日志。

	![“诊断日志”边栏选项卡](./media/monitoring-archive-diagnostic-logs/diag-log-monitoring-blade.png)  

4. 单击“保存”。

一旦生成新的事件数据，就会将诊断日志存档到该存储帐户。

## 通过 PowerShell Cmdlet 存档诊断日志

```
Set-AzureRmDiagnosticSetting -ResourceId /subscriptions/s1id1234-5679-0123-4567-890123456789/resourceGroups/testresourcegroup/providers/Microsoft.Network/networkSecurityGroups/testnsg -StorageAccountId /subscriptions/s1id1234-5679-0123-4567-890123456789/resourceGroups/myrg1/providers/Microsoft.Storage/storageAccounts/my_storage -Categories networksecuritygroupevent,networksecuritygrouprulecounter -Enabled $true -RetentionEnabled $true -RetentionInDays 90
```

| 属性 | 必选 | 说明 |
|------------------|----------|-------------------------------------------------------------------------------------------------------|
| ResourceId | 是 | 要设置诊断设置的资源的资源 ID。 |
| StorageAccountId | 否 | 应该将诊断日志保存到其中的存储帐户的资源 ID。 |
| Categories | 否 | 要启用的日志类别的逗号分隔列表。 |
| Enabled | 是 | 一个布尔值，表示此资源是启用还是禁用了诊断。 |
| RetentionEnabled | 否 | 一个布尔值，表示此资源是否启用了保留策略。 |
| RetentionInDays | 否 | 事件的保留天数，介于 1 到 2147483647 之间。值为零时，将无限期存储日志。 |

## 通过跨平台 CLI 存档活动日志

```
azure insights diagnostic set --resourceId /subscriptions/s1id1234-5679-0123-4567-890123456789/resourceGroups/testresourcegroup/providers/Microsoft.Network/networkSecurityGroups/testnsg --storageId /subscriptions/s1id1234-5679-0123-4567-890123456789/resourceGroups/myrg1/providers/Microsoft.Storage/storageAccounts/my_storage –categories networksecuritygroupevent,networksecuritygrouprulecounter --enabled true
```

| 属性 | 必选 | 说明 |
|------------------|----------|-------------------------------------------------------------------------------------------------------|
| resourceId | 是 | 要设置诊断设置的资源的资源 ID。 |
| storageId | 否 | 应该将诊断日志保存到其中的存储帐户的资源 ID。 |
| categories | 否 | 要启用的日志类别的逗号分隔列表。 |
| enabled | 是 | 一个布尔值，表示此资源是启用还是禁用了诊断。 |

## 通过 REST API 存档诊断日志
若要了解如何使用 Insights REST API 设置诊断设置，请[参阅此文档](https://msdn.microsoft.com/zh-cn/library/azure/dn931931.aspx)。

## 存储帐户中诊断日志的架构
设置存档以后，一旦在某个已启用的日志类别中出现事件，就会在存储帐户中创建存储容器。在诊断日志和活动日志中，容器中的 blob 遵循相同的格式。这些 blob 的结构为：

> insights-logs-{日志类别名称}/resourceId=/SUBSCRIPTIONS/{订阅 ID}/RESOURCEGROUPS/{资源组名称}/PROVIDERS/{资源提供程序名称}/{资源类型}/{资源名称}/y={四位数年份}/m={两位数月份}/d={两位数日期}/h={两位数 24 小时制小时}/m=00/PT1H.json

或者使用更简单的形式：

> insights-logs-{日志类别名称}/resourceId=/{资源 ID}/y={四位数年份}/m={两位数月份}/d={两位数日期}/h={两位数 24 小时制小时}/m=00/PT1H.json

例如，blob 名称可以为：

> insights-logs-networksecuritygrouprulecounter/resourceId=/SUBSCRIPTIONS/s1id1234-5679-0123-4567-890123456789/RESOURCEGROUPS/TESTRESOURCEGROUP/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUP/TESTNSG/y=2016/m=08/d=22/h=18/m=00/PT1H.json

每个 PT1H.json blob 都包含一个 JSON blob，其中的事件为在 blob URL 中指定的小时（例如 h=12）内发生的。在当前的小时内发生的事件将附加到 PT1H.json 文件。分钟值始终为 00 (m=00)，因为诊断日志事件按小时细分成单个 blob。

在 PT1H.json 文件中，每个事件都按以下格式存储在“records”数组中：

```
{
	"records": [
		{
			"time": "2016-07-01T00:00:37.2040000Z",
			"systemId": "46cdbb41-cb9c-4f3d-a5b4-1d458d827ff1",
			"category": "NetworkSecurityGroupRuleCounter",
			"resourceId": "/SUBSCRIPTIONS/s1id1234-5679-0123-4567-890123456789/RESOURCEGROUPS/TESTRESOURCEGROUP/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/TESTNSG",
			"operationName": "NetworkSecurityGroupCounters",
			"properties": {
				"vnetResourceGuid": "{12345678-9012-3456-7890-123456789012}",
				"subnetPrefix": "10.3.0.0/24",
				"macAddress": "000123456789",
				"ruleName": "/subscriptions/ s1id1234-5679-0123-4567-890123456789/resourceGroups/testresourcegroup/providers/Microsoft.Network/networkSecurityGroups/testnsg/securityRules/default-allow-rdp",
				"direction": "In",
				"type": "allow",
				"matchedConnections": 1988
			}
		}
	]
}
```

| 元素名称 | 说明 |
|---------------|-------------------------------------------------------------------------------------------------------------|
| time | 处理与事件对应的请求的 Azure 服务生成事件时的时间戳。 |
| resourceId | 受影响资源的资源 ID。 |
| operationName | 操作的名称。 |
| category | 事件的日志类别。 |
| properties | `<Key, Value>` 对集合（即字典），描述事件的详细信息。 |

> [AZURE.NOTE] 这些属性的属性和使用情况各不相同，具体取决于资源。

## 后续步骤
- [下载 blob 进行分析](/documentation/articles/storage-dotnet-how-to-use-blobs/#download-blobs)
- [将诊断日志流式传输到事件中心](/documentation/articles/monitoring-stream-diagnostic-logs-to-event-hubs/)
- [详细了解诊断日志](/documentation/articles/monitoring-overview-of-diagnostic-logs/)

<!---HONumber=Mooncake_1010_2016-->