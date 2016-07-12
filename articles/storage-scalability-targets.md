<properties
	pageTitle="Azure 存储可伸缩性和性能目标 |Azure"
	description="了解有关 Azure 存储帐户的可伸缩性和性能目标的信息，包括标准和高级存储账户的容量、请求速率以及入站和出站带宽。了解每个 Azure 存储服务中各分区的性能目标。"
	services="storage"
	documentationCenter="na"
	authors="tamram"
	manager="na"
	editor="na" />
<tags 
	ms.service="storage"
   ms.date="04/19/2016"
	wacn.date="06/13/2016" />

# Azure 存储空间可伸缩性和性能目标

## 概述

本主题介绍 Azure 存储空间的可伸缩性和性能主题。有关其他 Azure 限制的摘要，请参阅 [Azure 订阅和服务限制、配额与约束](/documentation/articles/azure-subscription-service-limits/)。

>[AZURE.NOTE] 所有存储帐户都在新的扁平网络拓扑上运行，无论它们在何时创建，都支持下文概述的可伸缩性和性能目标。有关 Azure 存储的扁平网络体系结构和可伸缩性的详细信息，请参阅 [Azure 存储空间：具有高度一致性的高可用云存储服务](http://blogs.msdn.com/b/windowsazurestorage/archive/2011/11/20/windows-azure-storage-a-highly-available-cloud-storage-service-with-strong-consistency.aspx)。

>[AZURE.IMPORTANT] 以下所列的可伸缩性和性能目标为高端目标，但却是能够实现的。在任何情况下，你的存储帐户实现的请求速率和带宽取决于存储对象大小、使用的访问模式、应用程序执行的工作负荷类型。请务必测试你的服务，以确定其性能是否达到你的要求。如果可能，应避免流量速率突发峰值，并确保流量在各个分区上均匀分布。

>当你的应用程序达到分区能够处理的工作负荷极限时，Azure 存储将开始返回错误代码 503（服务器忙）或错误代码 500（操作超时）响应。发生这种情况时，应用程序应使用指数退让策略进行重试。使用指数退让策略，可以减少分区上的负载，缓解该分区的流量高峰。

如果您的应用程序的需求超过了单个存储帐户的可伸缩性目标值，您可以创建应用程序以使用多个存储帐户，并将数据对象分布到这些存储帐户中。有关批量定价的信息，请参阅 [Azure 存储空间定价](/home/features/storage/pricing/)。


##<a id="scalability-targets-for-standard-storage-accounts"></a> Blob、队列、表和文件的可伸缩性目标

[AZURE.INCLUDE [azure-storage-limits](../includes/azure-storage-limits.md)]

## 虚拟机磁盘的可伸缩性目标 

[AZURE.INCLUDE [azure-storage-limits-vm-disks](../includes/azure-storage-limits-vm-disks.md)]

请参阅 [Windows VM 大小](/documentation/articles/virtual-machines-windows-sizes/)或 [Linux VM 大小](/documentation/articles/virtual-machines-linux-sizes/)了解其他详细信息。

### 标准存储帐户

[AZURE.INCLUDE [azure-storage-limits-vm-disks-standard](../includes/azure-storage-limits-vm-disks-standard.md)]

###<a id="scalability-targets-for-premium-storage-accounts"></a> 高级存储帐户

[AZURE.INCLUDE [azure-storage-limits-vm-disks-premium](../includes/azure-storage-limits-vm-disks-premium.md)]

## Azure 资源管理器的可伸缩性目标

[AZURE.INCLUDE [azure-storage-limits-azure-resource-manager](../includes/azure-storage-limits-azure-resource-manager.md)]

## Azure 存储中的分区

可容纳存储在 Azure 存储中的数据的每个对象（Blob、消息、实体和文件）都属于某个分区，可用分区键进行标识。分区决定了 Azure 存储如何在多个服务器之间实现 Blob、消息、实体和文件的负载平衡，以满足这些对象的流量需求。分区键是唯一的，用于查找 Blob、消息或实体。

上面[标准存储帐户的可伸缩性目标](#standard-storage-accounts)中所示的表列出了每项服务在单个分区的性能目标。

分区会对每个存储服务的负载平衡和可伸缩性产生以下影响：

- **Blob**：Blob 的分区键是帐户名称 + 容器名称 + Blob 名称。这意味着如果 Blob 上的负载需要，每个 Blob 都可以具有其自己的分区。尽管可以在众多服务器间分布 Blob 以便扩大对其的访问权限，但只能由单个服务器为单个 Blob 提供服务。虽然 Blob 可在 Blob 容器中进行逻辑分组，但这种分组不会对分区产生影响。

- **文件**：文件的分区键是帐户名称 + 文件共享名称。这意味着一个文件共享中的所有文件也都位于单个分区。

- **消息**：消息的分区键是帐户名称 + 队列名称，因此一个队列中的所有消息都分组到单个分区中，由单个服务器提供服务。不同队列可以由不同服务器处理，无论存储帐户有多少队列，都可以平衡负载。

- **实体**：实体的分区键是帐户名称 + 表名称 + 分区键，其中，分区键是实体所需的用户定义的 **PartitionKey** 属性。具有相同分区键值的所有实体都分组到同一分区中，并由同一个分区服务器提供服务。在设计应用程序的过程中，了解这一点非常重要。将实体分布在多个分区中能够实现可伸缩性优势，而将实体分组到单个分区中则能够提供数据访问优势，你的应用程序应该平衡这两大优势。

将表中的一组实体分组到单个分区中的一大关键优势是能够对位于同一分区中的实体执行原子操作，因为一个分区存在于单个服务器上。因此，如果你想在一组实体上执行批处理操作，请考虑对具有相同分区键的实体进行分组。

另一方面，位于同一表中但具有不同分区键的实体可在不同服务器之间实现负载平衡，因而可能具有更好的伸缩性。

关于表的设计分区策略的详细建议可在[此处](https://msdn.microsoft.com/zh-cn/library/azure/hh508997.aspx)找到。

## 另请参阅

- [存储定价详细信息](/home/features/storage/pricing/)
- [Azure 订阅和服务限制、配额和约束](/documentation/articles/azure-subscription-service-limits/)
- [高级存储：适用于 Azure 虚拟机工作负荷的高性能存储](/documentation/articles/storage-premium-storage/)
- [Azure 存储复制](/documentation/articles/storage-redundancy/)
- [Azure 存储性能和可伸缩性清单](/documentation/articles/storage-performance-checklist/)
- [Azure 存储：具有高度一致性的高可用云存储服务](http://blogs.msdn.com/b/windowsazurestorage/archive/2011/11/20/windows-azure-storage-a-highly-available-cloud-storage-service-with-strong-consistency.aspx)

<!---HONumber=Mooncake_0606_2016-->