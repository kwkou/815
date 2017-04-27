<properties
    pageTitle="基于 HD 的高性价比标准存储和 Azure VM 磁盘 | Azure"
    description="介绍高性价比标准存储以及非托管和托管 VM 磁盘。"
    services="storage"
    documentationcenter=""
    author="yuemlu"
    manager="aungoo-msft"
    editor="tysonn"
    translationtype="Human Translation" />
<tags
    ms.assetid="e2a20625-6224-4187-8401-abadc8f1de91"
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="02/18/2017"
    wacn.date="04/24/2017"
    ms.author="yuemlu"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="4e37cdb57605f166f018703f2a207e4b2e7e3cec"
    ms.lasthandoff="04/14/2017" />

# <a name="cost-effective-standard-storage-and-unmanaged-azure-vm-disks"></a>高性价比标准存储以及非托管 Azure VM 磁盘

Azure 标准存储为运行不区分延迟的工作负荷提供可靠、低成本的磁盘支持。 它还支持 Blob、表、队列和文件。 使用标准存储，将数据存储在硬盘驱动器 (HDD)。 使用 VM 时，可将标准存储磁盘用于开发/测试方案和不太重要的工作负荷，将高级存储磁盘用于任务关键型生产应用程序。 所有 Azure 区域均提供标准存储。 

本文重点介绍将标准存储用于 VM 磁盘。 有关将存储用于 blob、表、队列和文件的详细信息，请参阅[存储简介](/documentation/articles/storage-introduction/)。

## <a name="disk-types"></a>磁盘类型



**非托管磁盘**：这是管理用于存储与 VM 磁盘对应的 VHD 文件的存储帐户的原始方法。 VHD 文件作为页 Blob 存储在存储帐户中。 可将非托管磁盘附加到任意大小的 Azure VM，包括主要使用高级存储的 VM，如 DSv2 系列。 Azure VM 支持附加多个标准磁盘，每个 VM 最多支持 64 TB 的存储容量。



若要开始使用 Azure 标准存储，请访问[开始 1 元试用](/pricing/1rmb-trial/)。 


## <a name="standard-storage-features"></a>标准存储功能 

让我们看一下标准存储的一些功能。 有关详细信息，请参阅 [Azure 存储简介](/documentation/articles/storage-introduction/)。

**标准存储**：Azure 标准存储支持 Azure 磁盘、Azure Blob、Azure 文件、Azure 表和 Azure 队列。 要使用标准存储服务，请从[创建 Azure 存储帐户](/documentation/articles/storage-create-storage-account/#create-a-storage-account)开始。

**标准存储磁盘**：可将标准存储磁盘附加到所有 Azure VM，包括与高级存储配合使用的 VM 系列，如 DSv2 系列。标准存储磁盘只能附加到一个 VM。但可以将一个或多个此类磁盘附加到 VM，最多为该 VM 大小定义的最大磁盘计数。在下一部分“标准存储的可伸缩性和性能目标”中将详细介绍规范。

**标准页 Blob**：标准页 Blob 用于保留 VM 的永久磁盘，也可通过 REST 直接访问它（这与其他类型的 Azure Blob 类似）。 [页 Blob](https://docs.microsoft.com/zh-cn/rest/api/storageservices/fileservices/Understanding-Block-Blobs--Append-Blobs--and-Page-Blobs) 是 512 字节页的集合，已针对随机读写操作优化。 

**存储复制** ：在大多数区域中，标准存储帐户中的数据可以在本地复制，或跨多个数据中心实现异地复制。 可用的四种复制类型包括本地冗余存储 (LRS)、异地冗余存储 (GRS) 和读取访问异地冗余存储 (RA-GRS)。

## <a name="scalability-and-performance-targets"></a>可缩放性和性能目标

本部分介绍使用标准存储时需要考虑的可伸缩性和性能目标。

### <a name="account-limits--does-not-apply-to-managed-disks"></a>帐户限制 - 不适用于托管磁盘

| **资源** | **默认限制** |
|--------------|-------------------|
| 每个存储帐户的 TB  | 500 TB |
| 每个存储帐户的最大入口<sup>1</sup>（中国区域） | 如果已启用 GRS，则为 5 Gbps；对于 LRS，为 10 Gbps |
| 每个存储帐户的最大出口<sup>1</sup>（中国区域） | 如果已启用 RA-GRS/GRS，则为 10 Gbps；对于 LRS，为 15 Gbps |
| 每个存储帐户的总请求速率（假设对象大小为 1 KB） | 每秒实体或每秒消息数目最高 20,000 IOPS |

<sup>1</sup>“入口”是指发送到存储帐户的所有数据（请求）。“出口”是指从存储帐户接收的所有数据（响应）。

有关详细信息，请参阅 [Azure 存储可伸缩性和性能目标](/documentation/articles/storage-scalability-targets/)。

如果应用程序的需求超过了单个存储帐户的可伸缩性目标，则在生成应用程序时请让它使用多个存储帐户，并将数据分布在这些存储帐户中。 

### <a name="standard-disks-limits"></a>标准磁盘限制

与高级磁盘不同，标准磁盘不预配每秒输入/输出操作数 (IOPS) 和吞吐量（带宽）。 标准磁盘的性能因其附加到的 VM 大小（而非磁盘大小）而异。 其性能预期可达到下表所列的性能限制。

**标准磁盘限制（非托管）**

| **VM 层**            | **基本层 VM** | **标准层 VM** |
|------------------------|-------------------|----------------------|
| 最大磁盘大小          | 1023 GB           | 1023 GB              |
| 每个磁盘最大 8 KB IOPS | 最大 300         | 最大 500            |
| 每个磁盘的最大带宽 | 最高 60 MB/秒     | 最高 60 MB/秒        |

如果工作负荷要求高性能、低延迟磁盘支持，则应考虑使用高级存储。 若要深入了解高级存储的优点，请访问[高性能高级存储和 Azure VM 磁盘](/documentation/articles/storage-premium-storage/)。 

## <a name="snapshots-and-copy-blob"></a>快照和复制 Blob

对于存储服务而言，VHD 文件是页 Blob。 可以创建页 Blob 的快照，然后将其复制到其他位置，如不同的存储帐户。

### <a name="unmanaged-disks"></a>非托管磁盘

可以为非托管标准磁盘创建[增量快照](/documentation/articles/storage-incremental-snapshots/)，这与对标准存储使用快照的方式相同。 如果源磁盘是本地冗余存储帐户，则建议创建快照，并将那些快照复制到地域冗余的标准存储帐户。 有关详细信息，请参阅 [Azure 存储冗余选项](/documentation/articles/storage-redundancy/)。

如果磁盘已附加到 VM，则磁盘上将不允许某些 API 操作。例如，磁盘附加到 VM 后，无法在该 Blob 上执行[复制 Blob](https://docs.microsoft.com/rest/api/storageservices/fileservices/Copy-Blob) 操作。此时，必须先使用[快照 Blob](https://docs.microsoft.com/rest/api/storageservices/fileservices/Snapshot-Blob) REST API 方法创建该 Blob 的快照，然后对该快照执行[复制 Blob](https://docs.microsoft.com/rest/api/storageservices/fileservices/Copy-Blob) 以复制附加的磁盘。或者，可以中断附加磁盘，然后执行任何必要的操作。

若要维护快照的异地冗余副本，可以使用 AzCopy 或“复制 Blob”将本地冗余存储帐户中的快照复制到异地冗余的标准存储帐户。有关详细信息，请参阅[使用 AzCopy 命令行实用程序传输数据](/documentation/articles/storage-use-azcopy/)和 [Copy Blob](https://docs.microsoft.com/zh-cn/rest/api/storageservices/fileservices/Copy-Blob)（复制 Blob）。

有关在标准存储帐户中针对页 Blob 执行 REST 操作的详细信息，请参阅 [Azure 存储服务 REST API](https://docs.microsoft.com/zh-cn/rest/api/storageservices/fileservices/Azure-Storage-Services-REST-API-Reference)。


## <a name="pricing-and-billing"></a>定价和计费

使用标准存储时，请注意以下计费方式：

* 标准存储非托管磁盘/数据大小 
* 标准存储快照
* 出站数据传输
* 事务

**非托管存储数据和磁盘大小：**对于非托管磁盘和其他数据（blob、表、队列和文件），只需为所使用的存储空间付费。 例如，如果 VM 的页 Blob 预配为 127 GB，但 VM 实际只使用了 10 GB 的空间，则只需为 10 GB 空间付费。 


**快照**：对标准磁盘的快照使用的额外容量计费。有关快照的详细信息，请参阅 [创建 Blob 的快照](https://docs.microsoft.com/zh-cn/rest/api/storageservices/fileservices/Creating-a-Snapshot-of-a-Blob)。

**出站数据传输**：[出站数据传输](/pricing/details/data-transfer/)（Azure 数据中心送出的数据）会产生带宽使用费。

**事务**：Azure 事务按访问层和存储类型计费。 有关详细信息，请访问[存储定价](/pricing/details/storage/)。

有关标准存储、虚拟机定价的详细信息，请参阅：

* [Azure 存储定价](/pricing/details/storage/)
* [虚拟机定价](/pricing/details/virtual-machines/)


## <a name="next-steps"></a>后续步骤

* [Azure 存储简介](/documentation/articles/storage-introduction/)

* [创建存储帐户](/documentation/articles/storage-create-storage-account/)


* [使用 Resource Manager 和 PowerShell 创建 VM](/documentation/articles/virtual-machines-windows-quick-create-powershell/)

* [使用 Azure CLI 2.0 创建 Linux VM](/documentation/articles/virtual-machines-linux-quick-create-cli/)
<!--Update_Description: remove managed disk related content;add anchors to H2 titles-->