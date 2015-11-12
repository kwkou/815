<properties
	pageTitle="Windows Azure 订阅和服务限制、配额和约束"
	description="提供常见的 Azure 订阅和服务限制、配额和约束的列表。这包括有关如何增加限制以及最大值的信息。"
	services=""
	documentationCenter=""
	authors="rothja"
	manager="jeffreyg"
	editor="monicar"/>

<tags
	ms.service="multiple"
	ms.date="08/09/2015"
	wacn.date="10/03/2015"/>

# Azure 订阅和服务限制、配额和约束

## 概述

本文档指定一些最常见的 Windows Azure 限制。请注意，本文档目前未涵盖所有 Azure 服务。一段时间后，将展开并更新这些限制以包含多个平台。

> [AZURE.NOTE]如果想要提高**默认限制**之上的限制，可以[打开免费的联机客户支持请求](http://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/)。无法将这些限制提高到超过下表中的**最大限制**值。如果没有任何**最大限制**列，则指定的资源不具有可调整的限制。

### 限制和 Azure 资源管理器

现在可以将多个 Azure 资源合并到单个 Azure 资源组中。在使用资源组时，以前针对全局的限制会通过 Azure 资源管理器在区域级别进行管理。<!--有关 Azure 资源组的详细信息，请参阅[使用资源组管理 Azure 资源](/documentation/articles/resource-group-portal)。-->

在下面的限制中，添加了一个新表以反映在使用 Azure 资源管理器时限制中的任何差异。例如，会存在一个**订阅限制**表和一个**订阅数限制 - Azure 资源管理器**表。如果某个限制同时适用于这两种方案，它将仅显示在第一个表中。除非另有说明，否则限制是跨所有区域的全局限制。

> [AZURE.NOTE]请务必强调 Azure 资源组中的资源配额是您的订阅可以访问的每个区域，而不像服务管理配额那样是可以访问的每个订阅。我们来使用核心配额作为示例。如果您需要根据对核心的支持请求增加配额，则需要决定您想要在哪个区域中使用多少核心，然后针对您希望的 Azure 资源组核心配额的数量和区域进行特定请求。因此，如果您需要在西欧使用 30 个核心以在那里运行您的应用程序，则应专门在西欧请求 30 个核心。但这不会增加您在任何其他区域的核心配额 -- 仅西欧会有 30 个核心配额。因此，您可能会发现考虑决定您在任何一个区域中的工作负荷所需的 Azure 资源组配额数量，以及请求您考虑在其中进行部署的每个区域的数量很有用。<!--请参阅[部署问题疑难解答](/documentation/articles/resource-group-deploy-debug##authentication-subscription-role-and-quota-issues)，了解有关发现您特定区域的当前配额的更多帮助。-->

## 订阅限制

[AZURE.INCLUDE [azure-subscription-limits](../includes/azure-subscription-limits.md)]

### 订阅限制 - Azure 资源管理器

使用 Azure 资源管理器和 Azure 资源组时，以下限制适用。未使用 Azure 资源管理器更改的限制不会在下面列出。请参阅上表了解这些限制。

[AZURE.INCLUDE [azure-subscription-limits-azure-resource-manager](../includes/azure-subscription-limits-azure-resource-manager.md)]


## 资源组限制

[AZURE.INCLUDE [azure-resource-groups-limits](../includes/azure-resource-groups-limits.md)]


## 虚拟机限制

[AZURE.INCLUDE [azure-virtual-machines-limits](../includes/azure-virtual-machines-limits.md)]


### 虚拟机限制 - Azure 资源管理器

使用 Azure 资源管理器和 Azure 资源组时，以下限制适用。未使用 Azure 资源管理器更改的限制不会在下面列出。请参阅上表了解这些限制。

[AZURE.INCLUDE [azure-virtual-machines-limits-azure-resource-manager](../includes/azure-virtual-machines-limits-azure-resource-manager.md)]


## 网络限制

[AZURE.INCLUDE [azure-virtual-network-limits](../includes/azure-virtual-network-limits.md)]

### 流量管理器限制

[AZURE.INCLUDE [traffic-manager-limits](../includes/traffic-manager-limits.md)]

## 存储限制

### 标准存储限制

[AZURE.INCLUDE [azure-storage-limits](../includes/azure-storage-limits.md)]

有关存储帐户限制的详细信息，请参阅 [Azure 存储空间可伸缩性和性能目标](/documentation/articles/storage-scalability-targets)。


### 高级存储限制

[AZURE.INCLUDE [azure-storage-limits-premium-storage](../includes/azure-storage-limits-premium-storage.md)]


### 存储限制- Azure 资源管理器

[AZURE.INCLUDE [azure-storage-limits-azure-resource-manager](../includes/azure-storage-limits-azure-resource-manager.md)]


## 云服务限制

[AZURE.INCLUDE [azure-cloud-services-limits](../includes/azure-cloud-services-limits.md)]

## 计划程序限制

[AZURE.INCLUDE [scheduler-limits-table](../includes/scheduler-limits-table.md)]

## SQL 数据库限制

有关 SQL 数据库限制的其他详细信息，请参阅以下主题：

 - [Azure SQL 数据库服务层（版本）](http://msdn.microsoft.com/library/azure/dn741340.aspx)
 - <!--[-->Azure SQL 数据库服务层和性能级别<!--](http://msdn.microsoft.com/library/azure/dn741336.aspx)-->
 - <!--[-->数据库吞吐量单位 (DTU) 配额<!--](http://msdn.microsoft.com/library/azure/ee336245.aspx#DTUs)-->
 - [SQL 数据库资源限制](/documentation/articles/sql-database-resource-limits)

## 媒体服务限制

[AZURE.INCLUDE [azure-mediaservices-limits](../includes/azure-mediaservices-limits.md)]

## CDN 限制

[AZURE.INCLUDE [cdn-limits](../includes/cdn-limits.md)]

## 移动服务限制

[AZURE.INCLUDE [mobile-services-limits](../includes/mobile-services-limits.md)]

## 服务总线限制

[AZURE.INCLUDE [azure-servicebus-limits](../includes/service-bus-quotas-table.md)]

## 流分析限制

[AZURE.INCLUDE [stream-analytics-limits-table](../includes/stream-analytics-limits-table.md)]

## Active Directory 限制

[AZURE.INCLUDE [AAD-service-limits](../includes/active-directory-service-limits-include.md)]

## StorSimple 系统限制

[AZURE.INCLUDE [storsimple-limits-table](../includes/storsimple-limits-table.md)]

## 备份限制

[AZURE.INCLUDE [azure-backup-limits](../includes/azure-backup-limits.md)]

## 站点恢复限制

[AZURE.INCLUDE [site-recovery-limits](../includes/site-recovery-limits.md)]

## Azure Redis 缓存限制

[AZURE.INCLUDE [redis-cache-service-limits](../includes/redis-cache-service-limits.md)]

## 另请参阅

[了解 Azure 限制和增加](http://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/)

[Azure 的虚拟机和云服务大小](http://msdn.microsoft.com/zh-cn/library/azure/dn197896.aspx)

<!---HONumber=71-->