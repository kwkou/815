<properties
   pageTitle="Azure Batch 诊断日志记录 | Azure"
   description="记录并分析 Azure Batch 帐户资源（诸如池和任务）的诊断日志事件。"
   services="batch"
   documentationCenter=""
   authors="mmacy"
   manager="timlt"
   editor=""/>  


<tags
   ms.service="batch"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="multiple"
   ms.workload="big-compute"
   ms.date="10/12/2016"
   wacn.date="11/21/2016"
   ms.author="marsma"/>  


# Azure Batch 诊断日志记录

与许多 Azure 服务一样，Batch 服务也会在某些资源的生命周期内针对这些资源生成日志事件。可以启用 Azure Batch 诊断日志来记录资源（诸如池和任务）的事件，然后使用日志进行进行诊断评估和监视。Batch 诊断日志中包括诸如池创建、池删除、任务启动、任务完成之类的事件和其他事件。

>[AZURE.NOTE] 本文讨论了 Batch 帐户资源本身的日志记录事件，没有讨论作业和任务输出数据。有关如何存储作业和任务的输出数据的详细信息，请参阅[保存 Azure Batch 作业和任务输出](/documentation/articles/batch-task-output/)。

## 先决条件

* [Azure Batch 帐户](/documentation/articles/batch-account-create-portal/)

* [Azure 存储帐户](/documentation/articles/storage-create-storage-account/#create-a-storage-account/)

  若要暂留 Batch 诊断日志，必须创建一个将用来存储日志的 Azure 存储帐户。可以在为 Batch 帐户[启用诊断日志记录](#enable-diagnostic-logging)时指定此存储帐户。启用日志收集时指定的存储帐户与[应用程序包](/documentation/articles/batch-application-packages/)和[任务输出暂留](/documentation/articles/batch-task-output/)文章中所提到的链接存储帐户不是同一个。

  >[AZURE.WARNING] 对于存储在 Azure 存储帐户中的数据，将会向你**收费**。这包括本文中讨论的诊断日志。在设计日志保留策略时请记住这一点。

## 启用诊断日志记录

默认情况下没有为 Batch 帐户启用诊断日志记录。必须显式为要监视的每个 Batch 帐户启用诊断日志：

## 服务日志

Azure Batch 服务日志包含 Azure Batch 服务在 Batch 资源（诸如池或任务）的生命周期内生成的事件。Batch 生成的每个事件都采用 JSON 格式存储在指定的存储帐户中。例如，下面是一个**池创建事件**样本的正文：

json

	{
		"poolId": "myPool1",
		"displayName": "Production Pool",
		"vmSize": "Small",
		"cloudServiceConfiguration": {
			"osFamily": "4",
			"targetOsVersion": "*"
		},
		"networkConfiguration": {
			"subnetId": " "
		},
		"resizeTimeout": "300000",
		"targetDedicated": 2,
		"maxTasksPerNode": 1,
		"vmFillType": "Spread",
		"enableAutoscale": false,
		"enableInterNodeCommunication": false,
		"isAutoPool": false
	}


每个事件正文都位于指定 Azure 存储帐户中的一个 .json 文件中。

## 服务日志事件

Batch 服务当前会生成以下服务日志事件。此列表可能不完整，因为自本文最后更新以来可能又添加了其他事件。

| **服务日志事件** |
| ------------------ |
| [池创建][pool_create] |
| [池删除启动][pool_delete_start] |
| [池删除完成][pool_delete_complete] |
| [池调整大小启动][pool_resize_start] |
| [池调整大小完成][pool_resize_complete] |
| [任务启动][task_start] |
| [任务完成][task_complete] |
| [任务失败][task_fail] |

## 后续步骤

除了将诊断日志事件存储在 Azure 存储帐户中之外，还可以将 Batch 服务日志事件流式传输到 [Azure 事件中心](/documentation/articles/event-hubs-what-is-event-hubs/)，以及将它们发送到 Azure Log Analytics。

[pool_create]: https://msdn.microsoft.com/zh-cn/library/azure/mt743615.aspx
[pool_delete_start]: https://msdn.microsoft.com/zh-cn/library/azure/mt743610.aspx
[pool_delete_complete]: https://msdn.microsoft.com/zh-cn/library/azure/mt743618.aspx
[pool_resize_start]: https://msdn.microsoft.com/zh-cn/library/azure/mt743609.aspx
[pool_resize_complete]: https://msdn.microsoft.com/zh-cn/library/azure/mt743608.aspx
[task_start]: https://msdn.microsoft.com/zh-cn/library/azure/mt743616.aspx
[task_complete]: https://msdn.microsoft.com/zh-cn/library/azure/mt743612.aspx
[task_fail]: https://msdn.microsoft.com/zh-cn/library/azure/mt743607.aspx

<!---HONumber=Mooncake_1107_2016-->
