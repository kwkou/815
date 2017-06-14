<properties
	pageTitle="使用 REST 备份和还原应用服务应用"
	description="了解如何使用 RESTful API 调用在 Azure App Service 中备份和还原应用"
	services="app-service"
	documentationCenter=""
	authors="NKing92"
	manager="wpickett"
    editor="" />

<tags
	ms.service="app-service"
	ms.workload="na"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="08/10/2016"
	wacn.date="12/26/2016"
	ms.author="nicking"/>
# 使用 REST 备份和还原应用服务应用

> [AZURE.SELECTOR]
- [PowerShell](/documentation/articles/app-service-powershell-backup/)
- [REST API](/documentation/articles/websites-csm-backup/)

[AZURE.INCLUDE [azure-sdk-developer-differences](../../includes/azure-sdk-developer-differences.md)]

[应用服务应用](/home/features/app-service/web-apps/)可以备份为 Azure 存储中的 Blob。备份还可以包含该应用的数据库。如果意外地删除了该应用，或者需要将该应用还原到以前的版本，则可以从任何以前的备份还原。可随时按需备份，也可以计划以合适的时间间隔备份。

本文介绍如何使用 RESTful API 请求备份和还原应用。如果要通过 Azure 门户以图形方式创建和管理应用备份，请参阅[在 Azure App Service 中备份 Web 应用](/documentation/articles/web-sites-backup/)

## <a name="gettingstarted"></a>入门
若要发送 REST 请求，需要知道应用的“名称”、“资源组”和“订阅 ID”。可通过在 [Azure 门户](https://portal.azure.cn)的“应用服务”边栏选项卡中单击应用找到此信息。对于本文中的示例，我们要配置网站 **backuprestoreapiexamples.chinacloudsites.cn**。它将存储在 Default-Web-ChinaEast 资源组中，并在 ID 为 00001111-2222-3333-4444-555566667777 的订阅上运行。

![示例网站信息][SampleWebsiteInformation]

## <a name="backup-restore-rest-api"></a>备份和还原 REST API
现在，我们将介绍如何使用 REST API 来备份和还原应用的几个示例。每个示例均包括 URL 和 HTTP 请求正文。示例 URL 包含包装在大括号内的占位符，如 {subscription-id}。应将这些占位符替换为应用的相应信息。为了参考方便，下面是示例 URL 中显示的每个占位符的说明。

* subscription-id - 包含应用的 Azure 订阅的 ID
* resource-group-name - 包含应用的资源组的名称
* name - 应用的名称
* backup-id - 应用备份的 ID

## <a name="backup-on-demand"></a>按需备份应用
若要立即备份应用，请向 **https://management.chinacloudapi.cn/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Web/sites/{name}/backup/** 发送 **POST** 请求。

使用我们的示例网站时，URL 如下所示。**https://management.chinacloudapi.cn/subscriptions/00001111-2222-3333-4444-555566667777/resourceGroups/Default-Web-ChinaNorth/providers/Microsoft.Web/sites/backuprestoreapiexamples/backup/**

在请求的正文中提供 JSON 对象，以指定要使用哪个存储帐户存储备份。JSON 对象必须具有一个名为 **storageAccountUrl** 的属性，其中包含将写入访问权限授予包含备份 blob 的 Azure 存储容器的 [SAS URL](/documentation/articles/storage-dotnet-shared-access-signature-part-1/)。如果要备份数据库，还必须提供一个包含要备份的数据库的名称、类型和连接字符串的列表。

	{
    	"properties":
    	{
        	"storageAccountUrl": "https://account.blob.core.chinacloudapi.cn/backups?sv=2015-02-21&sr=c&sig=DzlkBl7h32C8qCv%2BifdBRxE63r4iv0kZ9L7E0qP16sY%3D&se=2016-09-15T22%3A46%3A54Z&sp=rwdl",
        	"databases": [
    	        {
                	"databaseType": "SqlAzure",
                	"name": "MyDatabase1",
    	            	"connectionString": "<your database connection string>"
    	        }
    	    ]
    	}
	}

收到该请求后，将立即开始备份应用。备份过程可能需要很长时间才能完成。HTTP 响应包含一个 ID，可以在另一个请求中使用此 ID 来查看备份的状态。下面是针对我们的备份请求的 HTTP 响应的正文示例。

	{
    	"id": "/subscriptions/00001111-2222-3333-4444-555566667777/resourceGroups/Default-Web-ChinaNorth/providers/Microsoft.Web/sites/backuprestoreapiexamples",
    	"name": " backuprestoreapiexamples ",
    	"type": "Microsoft.Web/sites",
    	"location": "ChinaNorth",
    	"properties":    {
    	    "id": 1,
            "storageAccountUrl": "https://account.blob.core.chinacloudapi.cn/backups?sv=2015-02-21&sr=c&sig=DzlkBl7h32C8qCv%2BifdBRxE63r4iv0kZ9L7E0qP16sY%3D&se=2016-09-15T22%3A46%3A54Z&sp=rwdl",
            "blobName": "backup_201509291825.zip",
    	    "name": "backup_201509291825",
    	    "status": 4,
    	    "sizeInBytes": 0,
    	    "created": "2015-09-29T18:25:26.3992707Z",
    	    "log": null,
    	    "databases": [
    	        {
    	            "databaseType": "SqlAzure",
    	            "name": "MyDatabase1",
    	            "connectionString": "<your database connection string>"
    	        }
    	    ],
    	    "scheduled": false,
    	    "correlationId": "ea730f4d-dd06-4f7f-a090-9aa09k662h36",
    	}
	}

>[AZURE.NOTE] 可以在 HTTP 响应的 log 属性中找到错误消息。

## <a name="schedule-automatic-backups"></a>计划自动备份
除了按需备份应用外，还可以计划自动进行的备份。

### 设置新自动备份计划
若要设置备份计划，请向 **https://management.chinacloudapi.cn/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Web/sites/{name}/config/backup** 发送 **PUT** 请求。

对于我们的示例网站，URL 如下所示。**https://management.chinacloudapi.cn/subscriptions/00001111-2222-3333-4444-555566667777/resourceGroups/Default-Web-ChinaNorth/providers/Microsoft.Web/sites/backuprestoreapiexamples/config/backup**

请求正文必须包含指定备份配置的 JSON 对象。下面是包含所有必需参数的一个示例。

	{
    	"location": "ChinaNorth",
    	"properties": // Represents an app restore request
    	{
    	    "backupSchedule": { // Required for automatically scheduled backups
    	        "frequencyInterval": "7",
    	        "frequencyUnit": "Day",
    	        "keepAtLeastOneBackup": "True",
    	        "retentionPeriodInDays": "10",
    	    },
    	    "enabled": "True", // Must be set to true to enable automatic backups
            "name": "mysitebackup",
    	    "storageAccountUrl": "https://account.blob.core.chinacloudapi.cn/backups?sv=2015-02-21&sr=c&sig=DzlkBl7h32C8qCv%2BifdBRxE63r4iv0kZ9L7E0qP16sY%3D&se=2016-09-15T22%3A46%3A54Z&sp=rwdl"
    	}
	}

此示例将应用配置为每 7 天自动备份一次。参数 **frequencyInterval** 和 **frequencyUnit** 共同确定备份发生的频率。**frequencyUnit** 的有效值为 **hour** 和 **day**。例如，若要每 12 个小时备份一次应用，请将 frequencyInterval 设为 12 并将 frequencyUnit 设为 hour。

旧备份将自动从存储帐户中删除。可以通过设置 **retentionPeriodInDays** 参数来控制备份的保留期限。如果始终要至少保存一个备份，且不管它保留多长时间，请将 **keepAtLeastOneBackup** 设为 True。

### 获取自动备份计划
若要获取应用的备份配置，请向 URL **https://management.chinacloudapi.cn/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Web/sites/{name}/config/backup/list** 发送 **POST** 请求。

我们的示例站点的 URL 是 **https://management.chinacloudapi.cn/subscriptions/00001111-2222-3333-4444-555566667777/resourceGroups/Default-Web-ChinaNorth/providers/Microsoft.Web/sites/backuprestoreapiexamples/config/backup/list**。

## <a name="get-backup-status"></a>获取备份的状态
根据应用的大小，备份可能需要一段时间才能完成。备份也可能会失败、超时或部分成功。若要查看应用的所有备份的状态，请向 URL **https://management.chinacloudapi.cn/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Web/sites/{name}/backups** 发送 **GET** 请求。

若要查看特定备份的状态，请向 URL **https://management.chinacloudapi.cn/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Web/sites/{name}/backups/{backup-id}** 发送 GET 请求。

对于我们的示例网站，URL 如下所示。**https://management.chinacloudapi.cn/subscriptions/00001111-2222-3333-4444-555566667777/resourceGroups/Default-Web-ChinaNorth/providers/Microsoft.Web/sites/backuprestoreapiexamples/backups/1**

响应正文包含类似于此示例的 JSON 对象。

	{
    	"properties":  {
    	    "id": 1,
    	    "storageAccountUrl": "https://account.blob.core.chinacloudapi.cn/backups?sv=2015-02-21&sr=c&sig=DzlkBl7h32C8qCv%2BifdBRxE63r4iv0kZ9L7E0qP16sY%3D&se=2016-09-15T22%3A46%3A54Z&sp=rwdl",
    	    "blobName": "backup_201509291734.zip",
    	    "name": "backup_201509291734",
    	    "status": 2,
    	    "sizeInBytes": 151973,
    	    "created": "2015-09-29T17:34:57.2091605",
    	    "scheduled": false,
    	    "lastRestoreTimeStamp": null,
    	    "finishedTimeStamp": "2015-09-29T17:35:11.3293602",
    	    "correlationId": "2379fc13-418a-4b9c-920d-d06e73c5028d",
    	    "websiteSizeInBytes": 209920
    	}
	}

备份状态是一个枚举类型。下面是每种可能的状态。

* 0 - InProgress：备份已启动但尚未完成。
* 1 - Failed：备份未成功。
* 2 - Succeeded：备份成功完成。
* 3 – TimedOut：备份未按时完成，已被取消。
* 4 - Created：备份请求已排队，但尚未启动。
* 5 - Skipped：由于计划触发了太多备份，备份尚未进行。
* 6 - PartiallySucceeded：备份成功，但有些文件因无法读取而未备份。之所以会发生这种情况，通常是因为在这些文件上放置了排他锁。
* 7 - DeleteInProgress：已请求删除备份，但尚未删除。
* 8 - DeleteFailed：无法删除备份。之所以会发生这种情况，是因为用于创建备份的 SAS URL 已过期。
* 9 - Deleted：备份已成功删除。

## <a name="restore-app"></a>从备份中还原应用
如果你的应用已删除，或者想要将应用程还原到以前的版本，则可以从备份还原应用。若要调用还原，请向 URL **https://management.chinacloudapi.cn/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Web/sites/{name}/backups/{backup-id}/restore** 发送 **POST** 请求。

对于我们的示例网站，URL 如下所示。**https://management.chinacloudapi.cn/subscriptions/00001111-2222-3333-4444-555566667777/resourceGroups/Default-Web-ChinaNorth/providers/Microsoft.Web/sites/backuprestoreapiexamples/backups/1/restore**

在请求正文中，发送包含还原操作的属性的 JSON 对象。下面是一个包含所有必需属性的示例：

	{
    	"location": "ChinaNorth",
    	"properties":
    	{
    	    "blobName": "backup_201509280431.zip",
    	    "databases": [ // Not required unless you're restoring databases
    	        {
                "databaseType": "SqlAzure",
                "name": "MyDatabase1"
    	    }],
    	    "overwrite": "true",
    	    "storageAccountUrl": "https://account.blob.core.chinacloudapi.cn/backups?sv=2015-02-21&sr=c&sig=DzlkBl7h32C8qCv%2BifdBRxE63r4iv0kZ9L7E0qP16sY%3D&se=2016-09-15T22%3A46%3A54Z&sp=rwdl" // SAS URL to storage container containing your website backup
    	}
	}

### 还原到新应用
有时你可能想要在还原备份时创建新应用，而不是覆盖现有的应用。为此，请更改请求 URL 以指向要创建的新应用，并将 JSON 中的 **overwrite** 属性更改为 **false**。

## <a name="delete-app-backup"></a>删除应用备份
如果要删除备份，请向 URL **https://management.chinacloudapi.cn/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Web/sites/{name}/backups/{backup-id}** 发送 **DELETE** 请求。

对于我们的示例网站，URL 如下所示。**https://management.chinacloudapi.cn/subscriptions/00001111-2222-3333-4444-555566667777/resourceGroups/Default-Web-ChinaNorth/providers/Microsoft.Web/sites/backuprestoreapiexamples/backups/1**

## <a name="manage-sas-url"></a>管理备份的 SAS URL
Azure App Service 会尝试使用创建备份时提供的 SAS URL 从 Azure 存储中删除备份。如果此 SAS URL 失效，则无法通过 REST API 删除备份。但是，可以通过向 URL **https://management.chinacloudapi.cn/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Web/sites/{name}/backups/{backup-id}/list** 发送 **POST** 请求更新与备份关联的 SAS URL。

对于我们的示例网站，URL 如下所示。**https://management.chinacloudapi.cn/subscriptions/00001111-2222-3333-4444-555566667777/resourceGroups/Default-Web-ChinaNorth/providers/Microsoft.Web/sites/backuprestoreapiexamples/backups/1/list**

在请求正文中，发送包含新 SAS URL 的 JSON 对象。下面是一个示例。

	{
    	"properties":
    	{
    	    "storageAccountUrl": "https://account.blob.core.chinacloudapi.cn/backups?sv=2015-02-21&sr=c&sig=DzlkBl7h32C8qCv%2BifdBRxE63r4iv0kZ9L7E0qP16sY%3D&se=2016-09-15T22%3A46%3A54Z&sp=rwdl"
    	}
	}

>[AZURE.NOTE] 出于安全原因，在为特定备份发送 GET 请求时，将不返回与该备份关联的 SAS URL。如果要查看与备份关联的 SAS URL，请向上述同一 URL 发送 POST 请求。在请求正文中包含空 JSON 对象。服务器响应包含该备份的所有信息，包括其 SAS URL。

<!-- IMAGES -->
[SampleWebsiteInformation]: ./media/websites-csm-backup/01siteconfig.png

<!---HONumber=Mooncake_Quality_Review_1215_2016-->