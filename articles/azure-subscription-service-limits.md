<properties
    pageTitle="Azure 订阅和服务限制、配额和约束"
    description="提供常见的 Azure 订阅和服务限制、配额和约束的列表。这包括有关如何增加限制以及最大值的信息。"
    services=""
    documentationcenter=""
    author="rothja"
    manager="jeffreyg"
    editor=""
    tags="billing" />

<tags
    ms.assetid="60d848f9-ff26-496e-a5ec-ccf92ad7d125"
    ms.service="billing"
    ms.workload="na"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="12/19/2016"
    wacn.date="02/10/2017"
    ms.author="byvinyal" />

# Azure 订阅和服务限制、配额和约束
本文列出了一些最常见的 Azure 限制，有时也称为配额。本文当前并不涵盖所有 Azure 服务。随着时间的推移，此列表将得以展开并更新，以便涵盖更多平台。

请访问 [Azure 定价概述](/pricing/)以了解有关 Azure 定价的详细信息。你可以在该网站上使用[定价计算器](/pricing/calculator/)或访问服务的定价详细信息页（例如，[Windows VM](/pricing/details/virtual-machines/#Windows/)）来估算成本。

> [AZURE.NOTE]如果想要提高**默认限制**之上的限制，可以[打开免费的联机客户支持请求](/support/support-ticket-form/)。无法将这些限制提高到超过下表中的**最大限制**值。如果没有任何**最大限制**列，则指定的资源不具有可调整的限制。

## <a name="limits-and-the-azure-resource-manager"></a> 限制和 Azure 资源管理器

现在可以将多个 Azure 资源合并到单个 Azure 资源组中。在使用资源组时，以前针对全局的限制会通过 Azure 资源管理器在区域级别进行管理。有关 Azure 资源组的详细信息，请参阅 [Azure资源管理器概述](/documentation/articles/resource-group-overview/)。

在下面的限制中，添加了一个新表以反映在使用 Azure 资源管理器时限制中的任何差异。例如，会存在一个**订阅限制**表和一个**订阅数限制 - Azure资源管理器** 表。如果某个限制同时适用于这两种方案，它将仅显示在第一个表中。除非另有说明，否则限制是跨所有区域的全局限制。

> [AZURE.NOTE]
请务必强调 Azure 资源组中的资源配额是用户的订阅可以访问的每个区域，而不像服务管理配额那样是可以访问的每个订阅。我们来使用核心配额作为示例。如果您需要根据对核心的支持请求增加配额，则需要决定您想要在哪个区域中使用多少核心，然后针对您希望的 Azure 资源组核心配额的数量和区域进行特定请求。因此，如果需要在中国北部站点 1 使用 30 个核心来运行应用程序，则应专门为中国东部站点 1 请求 30 个核心。但是，任何其他区域的核心配额不会增加 -- 仅中国北部站点 1 会有 30 个核心配额。
> <!-- -->
> 
> 因此，你可能会发现考虑决定你在任何一个区域中的工作负荷所需的 Azure 资源组配额数量，以及请求你考虑在其中进行部署的每个区域的数量很有用。请参阅[部署问题疑难解答](/documentation/articles/resource-manager-common-deployment-errors/)，了解有关发现你特定区域的当前配额的更多帮助。
> 
> 

## 服务特定的限制
- [Active Directory](#active-directory-limits)

- [App Service](#app-service-limits)
- [应用程序网关](#application-gateway-limits)

- [自动化](#automation-limits)

- [Azure Redis Cache](#azure-redis-cache-limits)

- [备份](#backup-limits)

- [批处理](#batch-limits)

- [CDN](#cdn-limits)

- [云服务](#cloud-services-limits)

- [DocumentDB](#documentdb-limits)
- [事件中心](#event-hubs-limits)

- [IoT 中心](#iot-hub-limits)

- [密钥保管库](#key-vault-limits)

- [媒体服务](#media-services-limits)

- [移动服务](#mobile-services-limits)

- [多重身份验证](#multi-factor-authentication)

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

- [流量管理器](#traffic-manager-limits)

- [虚拟机](#virtual-machines-limits)

- [虚拟机规模集](#virtual-machine-scale-sets-limits)

### 订阅限制 <a id="subscription-limits" name="subscription"></a>
#### 订阅限制
[AZURE.INCLUDE [azure-subscription-limits](../includes/azure-subscription-limits.md)]

#### 订阅限制 - Azure资源管理器
使用 Azure 资源管理器和 Azure 资源组时，以下限制适用。未使用 Azure 资源管理器更改的限制不会在下面列出。请参阅上表了解这些限制。

[AZURE.INCLUDE [azure-subscription-limits-azure-resource-manager](../includes/azure-subscription-limits-azure-resource-manager.md)]

### 资源组限制 <a id="resource-group-limits"></a>
[AZURE.INCLUDE [azure-resource-groups-limits](../includes/azure-resource-groups-limits.md)]

### 虚拟机限制 <a id="virtual-machines-limits"></a>
#### 虚拟机限制
[AZURE.INCLUDE [azure-virtual-machines-limits](../includes/azure-virtual-machines-limits.md)]

#### 虚拟机限制 - Azure资源管理器
使用 Azure 资源管理器和 Azure 资源组时，以下限制适用。未使用 Azure 资源管理器更改的限制不会在下面列出。请参阅上表了解这些限制。

[AZURE.INCLUDE [azure-virtual-machines-limits-azure-resource-manager](../includes/azure-virtual-machines-limits-azure-resource-manager.md)]

### <a name="virtual-machine-scale-sets-limits"></a> 虚拟机规模集限制

[AZURE.INCLUDE [virtual-machine-scale-sets-limits](../includes/azure-virtual-machine-scale-sets-limits.md)]

### <a name="networking-limits"></a> 网络限制

[AZURE.INCLUDE [expressroute-limits](../includes/expressroute-limits.md)]

#### 网络限制
[AZURE.INCLUDE [azure-virtual-network-limits](../includes/azure-virtual-network-limits.md)]

#### <a name="application-gateway-limits"></a>应用程序网关限制
[AZURE.INCLUDE [application-gateway-limits](../includes/application-gateway-limits.md)]

#### 流量管理器限制 <a id="traffic-manager-limits"></a>
[AZURE.INCLUDE [traffic-manager-limits](../includes/traffic-manager-limits.md)]


### 存储限制 <a id="storage-limits"></a>
有关存储帐户限制的详细信息，请参阅 [Azure 存储空间可伸缩性和性能目标](/documentation/articles/storage-scalability-targets/)。

#### 存储服务限制

[AZURE.INCLUDE [azure-storage-limits](../includes/azure-storage-limits.md)]

#### <a name="virtual-machine-disk-limits"></a> 虚拟机磁盘限制

[AZURE.INCLUDE [azure-storage-limits-vm-disks](../includes/azure-storage-limits-vm-disks.md)]

有关其他详细信息，请参阅[虚拟机大小](/documentation/articles/virtual-machines-linux-sizes?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json/)。

**标准存储帐户**

[AZURE.INCLUDE [azure-storage-limits-vm-disks-standard](../includes/azure-storage-limits-vm-disks-standard.md)]

**高级存储帐户**

[AZURE.INCLUDE [azure-storage-limits-vm-disks-premium](../includes/azure-storage-limits-vm-disks-premium.md)]

#### 存储资源提供程序限制
[AZURE.INCLUDE [azure-storage-limits-azure-resource-manager](../includes/azure-storage-limits-azure-resource-manager.md)]

### 云服务限制 <a id="cloud-services-limits"></a>
[AZURE.INCLUDE [azure-cloud-services-limits](../includes/azure-cloud-services-limits.md)]

### App Service 限制 <a id="app-service-limits"></a>
以下 App Service 限制包括 Web Apps 的限制。

[AZURE.INCLUDE [azure-websites-limits](../includes/azure-websites-limits.md)]

### 计划程序限制 <a id="scheduler-limits"></a>
[AZURE.INCLUDE [scheduler-limits-table](../includes/scheduler-limits-table.md)]

### Batch 限制 <a id="batch-limits"></a>
[AZURE.INCLUDE [azure-batch-limits](../includes/azure-batch-limits.md)]


### DocumentDB 限制 <a name="documentdb-limits"></a>
DocumentDB 是一个全规模数据库，该数据库对吞吐量和存储进行规模调整以满足应用程序的任何需求。如果你有任何关于 DocumentDB 提供的规模的问题，请发送电子邮件到 askdocdb@microsoft.com。


### 媒体服务限制 <a id="media-services-limits"></a>
[AZURE.INCLUDE [azure-mediaservices-limits](../includes/azure-mediaservices-limits.md)]

### CDN 限制 <a id="cdn-limits"></a>
[AZURE.INCLUDE [cdn-limits](../includes/cdn-limits.md)]

### 移动服务限制 <a id="mobile-services-limits"></a>
[AZURE.INCLUDE [mobile-services-limits](../includes/mobile-services-limits.md)]


### 通知中心服务限制 <a id="notification-hub-service-limits"></a>

[AZURE.INCLUDE [notification-hub-limits](../includes/notification-hub-limits.md)]

### 事件中心限制 <a id="event-hubs-limits"></a>
[AZURE.INCLUDE [azure-servicebus-limits](../includes/event-hubs-limits.md)]

### 服务总线限制 <a id="service-bus-limits"></a>
[AZURE.INCLUDE [azure-servicebus-limits](../includes/service-bus-quotas-table.md)]

### IoT 中心限制 <a id="iot-hub-limits"></a>
[AZURE.INCLUDE [azure-iothub-limits](../includes/iot-hub-limits.md)]


### 流分析限制 <a id="stream-analytics-limits"></a>
[AZURE.INCLUDE [stream-analytics-limits-table](../includes/stream-analytics-limits-table.md)]

### Active Directory 限制 <a id="active-directory-limits"></a>
[AZURE.INCLUDE [AAD-service-limits](../includes/active-directory-service-limits-include.md)]


### StorSimple 系统限制 <a id="storsimple-system-limits"></a>
[AZURE.INCLUDE [storsimple-limits-table](../includes/storsimple-limits-table.md)]


### 备份限制 <a id="backup-limits"></a>
[AZURE.INCLUDE [azure-backup-limits](../includes/azure-backup-limits.md)]

### 站点恢复限制 <a id="site-recovery-limits"></a>
[AZURE.INCLUDE [site-recovery-limits](../includes/site-recovery-limits.md)]


### Azure Redis 缓存限制 <a id="azure-redis-cache-limits"></a>
[AZURE.INCLUDE [redis-cache-service-limits](../includes/redis-cache-service-limits.md)]

### 密钥保管库限制 <a id="key-vault-limits"></a>
[AZURE.INCLUDE [key-vault-limits](../includes/key-vault-limits.md)]

### 多重身份验证 <a id="multi-factor-authentication"></a>
[AZURE.INCLUDE [azure-mfa-service-limits](../includes/azure-mfa-service-limits.md)]

### 自动化限制 <a id="automation-limits"></a>
[AZURE.INCLUDE [automation-limits](../includes/azure-automation-service-limits.md)]

### SQL 数据库限制 <a id="sql-database-limits"></a>
有关 Azure SQL 数据库限制，请参阅 [SQL 数据库资源限制](/documentation/articles/sql-database-resource-limits/)。

## 另请参阅

[Azure 的虚拟机和云服务大小](/documentation/articles/virtual-machines-linux-sizes?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json/)

[云服务的大小](/documentation/articles/cloud-services-sizes-specs/)

<!---HONumber=Mooncake_0103_2017-->
