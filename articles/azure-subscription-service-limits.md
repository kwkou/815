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
	ms.date="11/10/2015"
	wacn.date="12/31/2015"/>

# Azure 订阅和服务限制、配额和约束

## 概述

本文档指定一些最常见的 Windows Azure 限制。请注意，本文档目前未涵盖所有 Azure 服务。一段时间后，将展开并更新这些限制以包含多个平台。

> [AZURE.NOTE]如果想要提高**默认限制**之上的限制，可以[打开免费的联机客户支持请求](http://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/)。无法将这些限制提高到超过下表中的**最大限制**值。如果没有任何**最大限制**列，则指定的资源不具有可调整的限制。

## 限制和 Azure 资源管理器

现在可以将多个 Azure 资源合并到单个 Azure 资源组中。在使用资源组时，以前针对全局的限制会通过 Azure 资源管理器在区域级别进行管理。<!--有关 Azure 资源组的详细信息，请参阅[使用资源组管理 Azure 资源](/documentation/articles/resource-group-portal)。-->

在下面的限制中，添加了一个新表以反映在使用 Azure 资源管理器时限制中的任何差异。例如，会存在一个**订阅限制**表和一个**订阅数限制 - Azure 资源管理器**表。如果某个限制同时适用于这两种方案，它将仅显示在第一个表中。除非另有说明，否则限制是跨所有区域的全局限制。

> [AZURE.NOTE]请务必强调 Azure 资源组中的资源配额是您的订阅可以访问的每个区域，而不像服务管理配额那样是可以访问的每个订阅。我们来使用核心配额作为示例。如果您需要根据对核心的支持请求增加配额，则需要决定您想要在哪个区域中使用多少核心，然后针对您希望的 Azure 资源组核心配额的数量和区域进行特定请求。因此，如果您需要在西欧使用 30 个核心以在那里运行您的应用程序，则应专门在西欧请求 30 个核心。但这不会增加您在任何其他区域的核心配额 -- 仅西欧会有 30 个核心配额。
<!-- -->
因此，您可能会发现考虑决定您在任何一个区域中的工作负荷所需的 Azure 资源组配额数量，以及请求您考虑在其中进行部署的每个区域的数量很有用。


## 服务特定的限制

- [Active Directory](#active-directory-limits)
- [Azure Redis Cache](#azure-redis-cache-limits)
- [备份](#backup-limits)
- [批处理](#batch-limits)
- [CDN](#cdn-limits)
- [云服务](#cloud-services-limits)
- [密钥保管库](#key-vault-limits)
- [媒体服务](#media-services-limits)
- [移动服务](#mobile-services-limits)
- [多因素身份验证](#multi-factor-authentication)
- [联网](#networking-limits)
- [通知中心服务](#notification-hub-service-limits)
- [资源组](#resource-group-limits)
- [计划程序](#scheduler-limits)
- [服务总线](#service-bus-limits)
- [站点恢复](#site-recovery-limits)
- [SQL 数据库](#sql-database-limits)
- [存储](#storage-limits)
- [StorSimple 系统](#storsimple-system-limits)
- [流分析](#stream-analytics-limits)
- [订阅](#subscription-limits)
- [虚拟机](#virtual-machines-limits)

<a id="subscription-limits"></a>
### 订阅限制
#### 订阅限制
[AZURE.INCLUDE [azure-subscription-limits](../includes/azure-subscription-limits.md)]

#### 订阅限制 - Azure 资源管理器

使用 Azure 资源管理器和 Azure 资源组时，以下限制适用。未使用 Azure 资源管理器更改的限制不会在下面列出。请参阅上表了解这些限制。

[AZURE.INCLUDE [azure-subscription-limits-azure-resource-manager](../includes/azure-subscription-limits-azure-resource-manager.md)]

<a id="resource-group-limits"></a>
### 资源组限制

[AZURE.INCLUDE [azure-resource-groups-limits](../includes/azure-resource-groups-limits.md)]

<a id="virtual-machines-limits"></a>
### 虚拟机限制
#### 虚拟机限制
[AZURE.INCLUDE [azure-virtual-machines-limits](../includes/azure-virtual-machines-limits.md)]


#### 虚拟机限制 - Azure 资源管理器

使用 Azure 资源管理器和 Azure 资源组时，以下限制适用。未使用 Azure 资源管理器更改的限制不会在下面列出。请参阅上表了解这些限制。

[AZURE.INCLUDE [azure-virtual-machines-limits-azure-resource-manager](../includes/azure-virtual-machines-limits-azure-resource-manager.md)]

<a id="networking-limits"></a>
### 网络限制

[AZURE.INCLUDE [expressroute-limits](../includes/expressroute-limits.md)]

#### 网络限制
[AZURE.INCLUDE [azure-virtual-network-limits](../includes/azure-virtual-network-limits.md)]

#### 流量管理器限制

[AZURE.INCLUDE [traffic-manager-limits](../includes/traffic-manager-limits.md)]

<a id="storage-limits"></a>
### 存储限制

#### 标准存储限制

[AZURE.INCLUDE [azure-storage-limits](../includes/azure-storage-limits.md)]

有关存储帐户限制的详细信息，请参阅 [Azure 存储空间可伸缩性和性能目标](/documentation/articles/storage-scalability-targets)。


#### 高级存储限制

[AZURE.INCLUDE [azure-storage-limits-premium-storage](../includes/azure-storage-limits-premium-storage.md)]


#### 存储限制- Azure 资源管理器

[AZURE.INCLUDE [azure-storage-limits-azure-resource-manager](../includes/azure-storage-limits-azure-resource-manager.md)]

<a id="cloud-services-limits"></a>
### 云服务限制

[AZURE.INCLUDE [azure-cloud-services-limits](../includes/azure-cloud-services-limits.md)]

<a id="scheduler-limits"></a>
### 计划程序限制

[AZURE.INCLUDE [scheduler-limits-table](../includes/scheduler-limits-table.md)]

<a id="batch-limits"></a>
### Batch 限制

[AZURE.INCLUDE [azure-batch-limits](../includes/azure-batch-limits.md)]

<a id="media-services-limits"></a>
### 媒体服务限制

[AZURE.INCLUDE [azure-mediaservices-limits](../includes/azure-mediaservices-limits.md)]

<a id="cdn-limits"></a>
### CDN 限制

[AZURE.INCLUDE [cdn-limits](../includes/cdn-limits.md)]

<a id="mobile-services-limits"></a>
### 移动服务限制

[AZURE.INCLUDE [mobile-services-limits](../includes/mobile-services-limits.md)]

<a id="notification-hub-service-limits"></a>
### 通知中心服务限制

[AZURE.INCLUDE [notification-hub-limits](../includes/notification-hub-limits.md)]

<a id="service-bus-limits"></a>
### 服务总线限制

[AZURE.INCLUDE [azure-servicebus-limits](../includes/service-bus-quotas-table.md)]

<a id="stream-analytics-limits"></a>
### 流分析限制

[AZURE.INCLUDE [stream-analytics-limits-table](../includes/stream-analytics-limits-table.md)]

<a id="active-directory-limits"></a>
### Active Directory 限制

[AZURE.INCLUDE [AAD-service-limits](../includes/active-directory-service-limits-include.md)]

<a id="storsimple-system-limits"></a>
### StorSimple 系统限制

[AZURE.INCLUDE [storsimple-limits-table](../includes/storsimple-limits-table.md)]

<a id="backup-limits"></a>
### 备份限制

[AZURE.INCLUDE [azure-backup-limits](../includes/azure-backup-limits.md)]

<a id="site-recovery-limits"></a>
### 站点恢复限制

[AZURE.INCLUDE [site-recovery-limits](../includes/site-recovery-limits.md)]

<a id="azure-redis-cache-limits"></a>
### Azure Redis 缓存限制

[AZURE.INCLUDE [redis-cache-service-limits](../includes/redis-cache-service-limits.md)]

<a id="key-vault-limits"></a>
### 密钥保管库限制

[AZURE.INCLUDE [key-vault-limits](../includes/key-vault-limits.md)]

<a id="multi-factor-authentication"></a>
### 多因素身份验证
[AZURE.INCLUDE [azure-mfa-service-limits](../includes/azure-mfa-service-limits.md)]

<a id="sql-database-limits"></a>
### SQL 数据库限制

有关 Azure SQL 数据库限制，请参阅 [SQL 数据库资源限制](/documentation/articles/sql-database-resource-limits)。

## 另请参阅

[了解 Azure 限制和增加](http://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/)

[Azure 的虚拟机和云服务大小](http://msdn.microsoft.com/zh-cn/library/azure/dn197896.aspx)

<!---HONumber=Mooncake_1221_2015-->