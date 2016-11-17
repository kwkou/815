<properties
	pageTitle="存档 Azure 活动日志 | Azure"
	description="了解如何存档 Azure 活动日志，将其长期保留在存储帐户中。"
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
	ms.date="08/23/2016"
	wacn.date="10/17/2016"
	ms.author="johnkem"/>  


# 存档 Azure 活动日志
本文介绍如何使用 Azure 门户、PowerShell Cmdlet 或跨平台 CLI 将 [**Azure 活动日志**](/documentation/articles/monitoring-overview-activity-logs/)存档到存储帐户中。此选项适用于对保留时长超过 90 天的活动日志进行审核、静态分析或备份（对保留策略具备完全控制权限）。如果只需将事件保留 90 天或更短的时间，则不需设置到存储帐户的存档，因为在不启用存档的情况下，活动日志事件保留在 Azure 平台中的时间是 90 天。

## 先决条件
在开始之前，需要[创建存储帐户](/documentation/articles/storage-create-storage-account/#create-a-storage-account)，将活动日志存档到其中。强烈建议用户不要使用其中存储了其他非监视数据的现有存储帐户，以便更好地控制监视数据所需的访问权限。但是，如果还要将诊断日志和指标存档到存储帐户，则也可将该存储帐户用于活动日志，使得所有监视数据都位于一个中心位置。所使用的存储帐户必须是一个通用存储帐户，而不是一个 blob 存储帐户。

## 日志配置文件
若要使用下述任意方法存档活动日志，可为订阅设置**日志配置文件**。日志配置文件定义已存储或流式传输的事件的类型，以及输出 - 存储帐户和/或事件中心。它还定义存储在存储帐户中的事件的保留策略（需保留的天数）。如果将保留策略设置为零，事件将无限期存储。如果不想让事件无限期存储，可将保留策略设置为 1 到 2147483647 之间的任何值。[单击此处了解日志配置文件的详细信息](/documentation/articles/monitoring-overview-activity-logs/#export-the-activity-log-with-log-profiles)。

## 使用门户存档活动日志
1. 在门户中，单击左侧导航中的“活动日志”链接。如果看不到活动日志的链接，请先单击“更多服务”链接。

    ![导航到“活动日志”边栏选项卡](./media/monitoring-archive-activity-log/act-log-portal-navigate.png)  

2. 在边栏选项卡顶部单击“导出”。

    ![单击“导出”按钮](./media/monitoring-archive-activity-log/act-log-portal-export-button.png)  

3. 在显示的边栏选项卡中，选中与“导出到存储帐户”相对应的框 ，然后选择存储帐户。

    ![设置存储帐户](./media/monitoring-archive-activity-log/act-log-portal-export-blade.png)  

4. 使用滑块或文本框，定义活动日志事件在存储帐户中的保留天数。若要让数据无限期保留在存储帐户中，可将此数值设置为零。
5. 单击“保存”。

## 通过 PowerShell 存档活动日志
```
Add-AzureRmLogProfile -Name my_log_profile -StorageAccountId /subscriptions/s1/resourceGroups/myrg1/providers/Microsoft.Storage/storageAccounts/my_storage -Locations chinaeast,chinanorth -RetentionInDays 180 -Categories Write,Delete,Action
```

| 属性 | 必选 | 说明 |
|------------------|----------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| StorageAccountId | 否 | 应该将活动日志保存到其中的存储帐户的资源 ID。 |
| Locations | 是 | 要为其收集活动日志事件的逗号分隔区域的列表。[访问此页](https://azure.microsoft.com/zh-CN/regions)或使用 [Azure 管理 REST API](https://msdn.microsoft.com/zh-cn/library/azure/gg441293.aspx) 即可查看所有区域的列表。 |
| RetentionInDays | 是 | 事件的保留天数，介于 1 到 2147483647 之间。值为零时，将无限期（永久）存储日志。 |
| Categories | 是 | 应收集的事件类别的逗号分隔列表。可能值包括：Write、Delete 和 Action。 |
## 通过 CLI 存档活动日志
```
azure insights logprofile add --name my_log_profile --storageId /subscriptions/s1/resourceGroups/insights-integration/providers/Microsoft.Storage/storageAccounts/my_storage --locations chinaeast,chinanorth --retentionInDays 180 –categories Write,Delete,Action
```

| 属性 | 必选 | 说明 |
|-----------------|----------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 名称 | 是 | 日志配置文件的名称。 |
| storageId | 否 | 应该将活动日志保存到其中的存储帐户的资源 ID。 |
| locations | 是 | 要为其收集活动日志事件的逗号分隔区域的列表。使用 [Azure 管理 REST API](https://msdn.microsoft.com/zh-cn/library/azure/gg441293.aspx) 即可查看所有区域的列表。 |
| retentionInDays | 是 | 事件的保留天数，介于 1 到 2147483647 之间。值为零时，将无限期（永久）存储日志。 |
| categories | 是 | 应收集的事件类别的逗号分隔列表。可能值包括：Write、Delete 和 Action。 |

## 活动日志的存储架构
设置存档以后，一旦出现活动日志事件，就会在存储帐户中创建存储容器。在活动日志和诊断日志中，容器中的 blob 遵循相同的格式。这些 blob 的结构为：

> insights-operational-logs/name=default/resourceId=/SUBSCRIPTIONS/{订阅 ID}/y={四位数年份}/m={两位数月份}/d={两位数日期}/h={两位数 24 小时制小时}/m=00/PT1H.json

例如，blob 名称可以为：

> insights-operational-logs/name=default/resourceId=/SUBSCRIPTIONS/s1id1234-5679-0123-4567-890123456789/y=2016/m=08/d=22/h=18/m=00/PT1H.json

每个 PT1H.json blob 都包含一个 JSON blob，其中的事件为在 blob URL 中指定的小时（例如 h=12）内发生的。在当前的小时内发生的事件将附加到 PT1H.json 文件。分钟值始终为 00 (m=00)，因为活动日志事件按小时细分成单个 blob。

在 PT1H.json 文件中，每个事件都按以下格式存储在“records”数组中：

```
{
	"records": [
		{
			"time": "2015-01-21T22:14:26.9792776Z",
			"resourceId": "/subscriptions/s1/resourceGroups/MSSupportGroup/providers/microsoft.support/supporttickets/115012112305841",
			"operationName": "microsoft.support/supporttickets/write",
			"category": "Write",
			"resultType": "Success",
			"resultSignature": "Succeeded.Created",
			"durationMs": 2826,
			"callerIpAddress": "111.111.111.11",
			"correlationId": "c776f9f4-36e5-4e0e-809b-c9b3c3fb62a8",
			"identity": {
				"authorization": {
					"scope": "/subscriptions/s1/resourceGroups/MSSupportGroup/providers/microsoft.support/supporttickets/115012112305841",
					"action": "microsoft.support/supporttickets/write",
					"evidence": {
						"role": "Subscription Admin"
					}
				},
				"claims": {
					"aud": "https://management.core.chinacloudapi.cn/",
					"iss": "https://sts.windows.net/72f988bf-86f1-41af-91ab-2d7cd011db47/",
					"iat": "1421876371",
					"nbf": "1421876371",
					"exp": "1421880271",
					"ver": "1.0",
					"http://schemas.microsoft.com/identity/claims/tenantid": "1e8d8218-c5e7-4578-9acc-9abbd5d23315 ",
					"http://schemas.microsoft.com/claims/authnmethodsreferences": "pwd",
					"http://schemas.microsoft.com/identity/claims/objectidentifier": "2468adf0-8211-44e3-95xq-85137af64708",
					"http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn": "admin@contoso.com",
					"puid": "20030000801A118C",
					"http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier": "9vckmEGF7zDKk1YzIY8k0t1_EAPaXoeHyPRn6f413zM",
					"http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname": "John",
					"http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname": "Smith",
					"name": "John Smith",
					"groups": "cacfe77c-e058-4712-83qw-f9b08849fd60,7f71d11d-4c41-4b23-99d2-d32ce7aa621c,31522864-0578-4ea0-9gdc-e66cc564d18c",
					"http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name": " admin@contoso.com",
					"appid": "c44b4083-3bq0-49c1-b47d-974e53cbdf3c",
					"appidacr": "2",
					"http://schemas.microsoft.com/identity/claims/scope": "user_impersonation",
					"http://schemas.microsoft.com/claims/authnclassreference": "1"
				}
			},
			"level": "Information",
			"location": "global",
			"properties": {
				"statusCode": "Created",
				"serviceRequestId": "50d5cddb-8ca0-47ad-9b80-6cde2207f97c"
			}
		}
	]
}
```


| 元素名称 | 说明 |
|-----------------|----------------------------------------------------------------------------------------------------------------|
| time | 处理与事件对应的请求的 Azure 服务生成事件时的时间戳。 |
| resourceId | 受影响资源的资源 ID。 |
| operationName | 操作的名称。 |
| category | 操作的类别，例如 Write、Read、Action。 |
| resultType | 结果的类型，例如 Success、Failure、Start |
| resultSignature | 取决于资源类型。 |
| durationMs | 操作持续时间，以毫秒为单位 |
| callerIpAddress | 执行操作（UPN 声明或 SPN 声明，具体取决于可用性）的用户的 IP 地址。 |
| correlationId | 通常为字符串格式的 GUID。共享 correlationId 的事件属于同一 uber 操作。 |
| identity | 描述授权和声明的 JSON blob。 |
| authorization | 包含事件的 RBAC 属性的 Blob。通常包括“action”、“role”和“scope”属性。 |
| level | 事件的级别。以下值之一：“Critical”、“Error”、“Warning”、“Informational”和“Verbose” |
| location | 位置所在的区域（或全局）。 |
| properties | `<Key, Value>` 对集合（即字典），描述事件的详细信息。 |

> [AZURE.NOTE] 这些属性的属性和使用情况各不相同，具体取决于资源。

## 后续步骤
- [下载 blob 进行分析](/documentation/articles/storage-dotnet-how-to-use-blobs/#download-blobs)
- [将活动日志流式传输到事件中心](/documentation/articles/monitoring-stream-activity-logs-event-hubs/)
- [详细了解活动日志](/documentation/articles/monitoring-overview-activity-logs/)

<!---HONumber=Mooncake_1010_2016-->