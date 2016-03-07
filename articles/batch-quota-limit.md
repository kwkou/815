<properties
	pageTitle="Batch 服务配额和限制 | Azure"
	description="了解使用 Azure Batch 服务时的配额、限制和约束"
	services="batch"
	documentationCenter=""
	authors="dlepow"
	manager="timlt"
	editor=""/>

<tags
	ms.service="batch"
	ms.date="10/26/2015"
	wacn.date="01/14/2016"/>



# Azure Batch 服务的配额和限制

本文列出可在 Azure Batch 服务中使用的特定资源的默认值和最大限制。其中的大多数限制都是 Azure 应用于订阅或 Batch 帐户的配额。

如果你打算运行生产 Batch 工作负荷，则可能需要增加一个或多个高于默认值的配额。如果需要提高配额，可以免费提出在线客户支持请求。

>[AZURE.NOTE]配额是一种信用限制，不附带容量保证。如果你有大规模的容量需求，请联系 Azure 支持。

## 订阅配额
资源|默认限制|最大限制
---|---|---
每个区域每个订阅的 Batch 帐户数|1|50

## Batch 帐户配额
[AZURE.INCLUDE [azure-batch-limits](../includes/azure-batch-limits.md)]

## 其他限制
资源|最大限制
---|---
每个计算节点的任务数|4 x 节点核心数


## 相关主题

* [Azure Batch 功能概述](batch-api-basics.md)
[account_quotas]: ./media/batch-quota-limit/accountquota_portal.PNG

<!---HONumber=Mooncake_0118_2016-->