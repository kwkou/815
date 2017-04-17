<properties
    pageTitle="Azure 事件中心诊断日志 | Azure"
    description="了解如何为 Azure 中的事件中心设置诊断日志。"
    keywords=""
    documentationcenter=""
    services="event-hubs"
    author="banisadr"
    manager=""
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid=""
    ms.service="event-hubs"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="data-services"
    ms.date="02/01/2017"
    wacn.date="04/17/2017"
    ms.author="babanisa"
    ms.sourcegitcommit="7cc8d7b9c616d399509cd9dbdd155b0e9a7987a8"
    ms.openlocfilehash="53ff2da916405074de80280d2c0b2e88585f27be"
    ms.lasthandoff="04/07/2017" />

# <a name="event-hubs-diagnostic-logs"></a>事件中心诊断日志

可以查看两种类型的 Azure 事件中心日志：
* **[活动日志](/documentation/articles/monitoring-overview-activity-logs/)**。 此类日志提供对作业执行的操作的相关信息。 此类日志始终开启。
* **[诊断日志](/documentation/articles/monitoring-overview-of-diagnostic-logs/)**。 可以配置诊断日志，以便更深入地了解作业发生的所有情况。 诊断日志涵盖从创建作业开始到删除作业为止的所有活动，其中包括作业运行时发生的更新和活动。

## <a name="turn-on-diagnostic-logs"></a>启用诊断日志
诊断日志默认已**禁用**。 若要启用诊断日志，请执行下列步骤：

1. 在 Azure 门户中，转到“流式处理作业”边栏选项卡。

2. 在“监视”下面，转到“诊断日志”边栏选项卡。

    ![在边栏选项卡中导航到诊断日志](./media/event-hubs-diagnostic-logs/image1.png)  

3. 选择“启用诊断”。

    ![启用诊断日志](./media/event-hubs-diagnostic-logs/image2.png)

4. 对于“状态”，选择“打开”。

    ![更改诊断日志的状态](./media/event-hubs-diagnostic-logs/image3.png)

5. 设置所需的存档目标，例如存储帐户、事件中心或 Azure Log Analytics。

6. 选择想要收集的日志类别，例如“执行”或“创作”。

7. 保存新的诊断设置。

新设置在大约 10 分钟后生效。 在此之后，日志将出现在“诊断日志”边栏选项卡上配置的存档目标中。

有关配置诊断的详细信息，请参阅 [Azure 诊断日志概述](/documentation/articles/monitoring-overview-of-diagnostic-logs/)。

## <a name="diagnostic-logs-categories"></a>诊断日志类别
事件中心会捕获两种类别的诊断日志：

* **ArchivalLogs** 捕获与事件中心存档相关（特别是与存档错误相关）的日志。
* **OperationalLogs** 捕获事件中心操作期间发生的日志，特别是包括事件中心创建在内的操作类型、所使用的资源和操作的状态。

## <a name="diagnostic-logs-schema"></a>诊断日志架构
所有日志均以 JavaScript 对象表示法 (JSON) 格式存储。 每个日志项目均包含字符串字段，这些字段采用以下示例中所述的格式。

### <a name="archive-logs-schema"></a>存档日志架构

存档日志 JSON 字符串包括下表列出的元素：

名称 | 说明
------- | -------
TaskName | 描述失败的任务
ActivityId | 用于跟踪的内部 ID
trackingId | 用于跟踪的内部 ID
resourceId | Azure Resource Manager 资源 ID
eventHub | 事件中心的完整名称（包括命名空间名称）
partitionId | 写入到的事件中心分区
archiveStep | ArchiveFlushWriter
startTime | 失败开始时间
failures | 发生失败的次数
durationInSeconds | 失败持续时间
message | 错误消息
category | ArchiveLogs

下面是存档日志 JSON 字符串的示例：

    {
         "TaskName": "EventHubArchiveUserError",
         "ActivityId": "21b89a0b-8095-471a-9db8-d151d74ecf26",
         "trackingId": "21b89a0b-8095-471a-9db8-d151d74ecf26_B7",
         "resourceId": "/SUBSCRIPTIONS/854D368F-1828-428F-8F3C-F2AFFA9B2F7D/RESOURCEGROUPS/DEFAULT-EVENTHUB-CENTRALUS/PROVIDERS/MICROSOFT.EVENTHUB/NAMESPACES/FBETTATI-OPERA-EVENTHUB",
         "eventHub": "fbettati-opera-eventhub:eventhub:eh123~32766",
         "partitionId": "1",
         "archiveStep": "ArchiveFlushWriter",
         "startTime": "9/22/2016 5:11:21 AM",
         "failures": 3,
         "durationInSeconds": 360,
         "message": "Microsoft.WindowsAzure.Storage.StorageException: The remote server returned an error: (404) Not Found. ---> System.Net.WebException: The remote server returned an error: (404) Not Found.\r\n   at Microsoft.WindowsAzure.Storage.Shared.Protocol.HttpResponseParsers.ProcessExpectedStatusCodeNoException[T](HttpStatusCode expectedStatusCode, HttpStatusCode actualStatusCode, T retVal, StorageCommandBase`1 cmd, Exception ex)\r\n   at Microsoft.WindowsAzure.Storage.Blob.CloudBlockBlob.<PutBlockImpl>b__3e(RESTCommand`1 cmd, HttpWebResponse resp, Exception ex, OperationContext ctx)\r\n   at Microsoft.WindowsAzure.Storage.Core.Executor.Executor.EndGetResponse[T](IAsyncResult getResponseResult)\r\n   --- End of inner exception stack trace ---\r\n   at Microsoft.WindowsAzure.Storage.Core.Util.StorageAsyncResult`1.End()\r\n   at Microsoft.WindowsAzure.Storage.Core.Util.AsyncExtensions.<>c__DisplayClass4.<CreateCallbackVoid>b__3(IAsyncResult ar)\r\n--- End of stack trace from previous location where exception was thrown ---\r\n   at System.",
         "category": "ArchiveLogs"
    }

### <a name="operation-logs-schema"></a>操作日志架构

操作日志 JSON 字符串包括下表列出的元素：

名称 | 说明
------- | -------
ActivityId | 用于跟踪目的的内部 ID
EventName | 操作名称
resourceId | Azure Resource Manager 资源 ID
SubscriptionId | 订阅 ID
EventTimeString | 操作时间
EventProperties | 操作属性
Status | 操作状态
Caller | 操作的调用方（Azure 门户或管理客户端）
category | OperationalLogs

下面是操作日志 JSON 字符串的示例：

    Example:
    {
         "ActivityId": "6aa994ac-b56e-4292-8448-0767a5657cc7",
         "EventName": "Create EventHub",
         "resourceId": "/SUBSCRIPTIONS/1A2109E3-9DA0-455B-B937-E35E36C1163C/RESOURCEGROUPS/DEFAULT-SERVICEBUS-CENTRALUS/PROVIDERS/MICROSOFT.EVENTHUB/NAMESPACES/SHOEBOXEHNS-CY4001",
         "SubscriptionId": "1a2109e3-9da0-455b-b937-e35e36c1163c",
         "EventTimeString": "9/28/2016 8:40:06 PM +00:00",
         "EventProperties": "{\"SubscriptionId\":\"1a2109e3-9da0-455b-b937-e35e36c1163c\",\"Namespace\":\"shoeboxehns-cy4001\",\"Via\":\"https://shoeboxehns-cy4001.servicebus.chinacloudapi.cn/f8096791adb448579ee83d30e006a13e/?api-version=2016-07\",\"TrackingId\":\"5ee74c9e-72b5-4e98-97c4-08a62e56e221_G1\"}",
         "Status": "Succeeded",
         "Caller": "ServiceBus Client",
         "category": "OperationalLogs"
    }

## <a name="next-steps"></a>后续步骤
* [事件中心简介](/documentation/articles/event-hubs-what-is-event-hubs/)
* [事件中心 API 概述](/documentation/articles/event-hubs-api-overview/)
* [事件中心入门](/documentation/articles/event-hubs-csharp-ephcs-getstarted/)

<!--Update_Description:wroding update;Update content about turn on diagnostic logs -->