<properties
	pageTitle="Azure 订阅和服务限制、配额和约束"
	description="提供常见的 Azure 订阅和服务限制、配额和约束的列表。这包括有关如何增加限制以及最大值的信息。"
	services=""
	documentationCenter=""
	authors="rothja"
	manager="jeffreyg"
	editor="monicar"/>

<tags
	ms.service="multiple"
	ms.date="04/11/2016"
	wacn.date="05/23/2016"/>

# Azure 订阅和服务限制、配额和约束

## 概述

本文档指定一些最常见的 Azure 限制。请注意，本文档目前未涵盖所有 Azure 服务。一段时间后，将展开并更新这些限制以包含多个平台。

> [AZURE.NOTE]如果想要提高**默认限制**之上的限制，可以[打开免费的联机客户支持请求](/support/support-ticket-form/?l=zh-cn)。无法将这些限制提高到超过下表中的**最大限制**值。如果没有任何**最大限制**列，则指定的资源不具有可调整的限制。

## 服务特定的限制

- [Active Directory](#active-directory-limits)
- [Azure Redis Cache](#azure-redis-cache-limits)
- [备份](#backup-limits)
- [批处理](#batch-limits)
- [CDN](#cdn-limits)
- [云服务](#cloud-services-limits)
- [IoT 中心](#iot-hub-limits)
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

<a id="resource-group-limits"></a>
### 资源组限制

[AZURE.INCLUDE [azure-resource-groups-limits](../includes/azure-resource-groups-limits.md)]

<a id="virtual-machines-limits"></a>
### 虚拟机限制
#### 虚拟机限制
[AZURE.INCLUDE [azure-virtual-machines-limits](../includes/azure-virtual-machines-limits.md)]

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

有关存储帐户限制的详细信息，请参阅 [Azure 存储空间可伸缩性和性能目标](/documentation/articles/storage-scalability-targets/)。


#### 高级存储限制

[AZURE.INCLUDE [azure-storage-limits-premium-storage](../includes/azure-storage-limits-premium-storage.md)]

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

<a id="iot-hub-limits"></a>
### IoT 中心 限制

[AZURE.INCLUDE [azure-iothub-limits](../includes/iot-hub-limits.md)]

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

有关 Azure SQL 数据库限制，请参阅 [SQL 数据库资源限制](/documentation/articles/sql-database-resource-limits/)。

## 另请参阅

[Azure虚拟机大小](/documentation/articles/virtual-machines-windows-sizes/)

[云服务的大小](/documentation/articles/cloud-services-sizes-specs/)

<!---HONumber=Mooncake_0328_2016-->