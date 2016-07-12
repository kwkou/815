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
	ms.date="03/11/2016"
	wacn.date="06/06/2016"/>



# Azure Batch 服务的配额和限制

本文列出可在 Azure Batch 服务中使用的特定资源的默认值和最大限制。其中的大多数限制都是 Azure 应用于订阅或 Batch 帐户的配额。

如果你打算运行生产 Batch 工作负荷，则可能需要增加一个或多个高于默认值的配额。如果需要提高配额，可以免费提出在线客户支持请求。

>[AZURE.NOTE] 配额是一种信用限制，不附带容量保证。如果你有大规模的容量需求，请联系 Azure 支持。

## 订阅配额
资源|默认限制|最大限制
---|---|---
每个区域每个订阅的 Batch 帐户数|1|50

## Batch 帐户配额
[AZURE.INCLUDE [azure-batch-limits](../includes/azure-batch-limits.md)]

## 其他限制
资源|最大限制
---|---
每个计算节点的任务数 | 4 x 节点核心数
每个 Batch 帐户的应用程序数 | 20
每个应用程序的应用程序包数 | 40
应用程序包大小（每个） | 约 195 GB<sup>1</sup>

<sup>1</sup> 最大的块 Blob 大小的 Azure 存储空间限制

## 增加配额

在 Azure 门户中使用以下步骤请求增加配额（也可以在 [Azure 经典管理门户](https://manage.windowsazure.cn/)中请求增加）。

1. 在门户的仪表板中，单击“帮助 + 支持”。

2. 单击“新建支持请求”>“基本”。

3. 在“基本”边栏选项卡中执行以下操作：

	a.在“问题类型”中，选择“配额”。

	b.选择你的订阅。

	c.在“配额类型”中，选择“Batch”。

	d.在“支持计划”中，选择“Azure 支持计划 - 开发人员”。

	单击“下一步”。

4. 在“问题”边栏选项卡中执行以下操作：

	a.根据业务影响情况选择“严重性”。

	b.在“详细信息”中，列出要在特定帐户中更改的配额和所需的新限制。

	单击“下一步”。

5. 在“联系信息”边栏选项卡中输入你的详细联系信息，然后单击“下一步”。

6. 单击“创建”提交新的支持请求。

Azure 支持人员将与你联系。完成请求最长需要 2 个工作日。

## 相关主题

* [Azure Batch 功能概述](/documentation/articles/batch-api-basics/)

* [Azure 订阅和服务限制、配额和约束](/documentation/articles/azure-subscription-service-limits/)

[account_quotas]: ./media/batch-quota-limit/accountquota_portal.PNG

<!---HONumber=Mooncake_0530_2016-->