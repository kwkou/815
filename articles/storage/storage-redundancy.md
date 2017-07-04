<properties
    pageTitle="Azure 存储中的数据复制 | Azure"
    description="复制 Azure 存储帐户中的数据以实现持久性和高可用性。复制选项包括本地冗余存储 (LRS)、异地冗余存储 (GRS) 和读取访问异地冗余存储 (RA-GRS)。"
    services="storage"
    documentationcenter=""
    author="mmacy"
    manager="timlt"
    editor="tysonn" />
<tags
    ms.assetid="86bdb6d4-da59-4337-8375-2527b6bdf73f"
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="05/15/2017"
    wacn.date="05/31/2017"
    ms.author="marsma"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="4a18b6116e37e365e2d4c4e2d144d7588310292e"
    ms.openlocfilehash="eafc727a28290bf0d02307f5deaa025827630f05"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/19/2017" />

# <a name="azure-storage-replication"></a>Azure 存储复制

始终复制 Azure 存储帐户中的数据，确保持久性和高可用性。 根据所选的复制选项，复制操作将在同一数据中心内复制数据或将其复制到辅助数据中心。 发生临时硬件故障时，复制会保护数据，并保证应用程序继续正常运行。 如果数据复制到第二个数据中心，还可以保护数据，以免主要位置发生灾难性故障。

即使面临故障时，复制也可确保存储帐户满足[存储的服务级别协议 (SLA)](/support/sla/storage/)的要求。 请参阅 SLA，了解有关 Azure 存储确保持续性和可用性的信息。 

创建存储帐户时，可以选择以下复制选项之一：

- [本地冗余存储 (LRS)](#locally-redundant-storage)
- [异地冗余存储 (GRS)](#geo-redundant-storage)
- [读取访问异地冗余存储 (RA-GRS)](#read-access-geo-redundant-storage)

创建存储帐户时，默认选项为读取访问异地冗余存储 (RA-GRS)。

下表简要概述了 LRS、GRS 和 RA-GRS 之间的差异，而后续章节将详细介绍每种类型的复制。


| 复制策略 | LRS | GRS | RA-GRS |
|:-----------------------------------------------------------------------------------|:----|:----|:-------|
| 数据在多个数据中心之间进行复制。 | 否 | 是 | 是 |
| 可以从辅助位置和主位置读取数据。 | 否 | 否 | 是 |
| 在单独的节点上维护的数据副本数。 | 3 | 6 | 6 |

有关不同冗余选项的定价信息，请参阅 [Azure 存储定价](/pricing/details/storage/)。

> [AZURE.NOTE]
> 高级存储仅支持本地冗余存储 (LRS)。 有关高级存储的信息，请参阅[高级存储：适用于 Azure 虚拟机工作负荷的高性能存储](/documentation/articles/storage-premium-storage/)。
>

## <a name="locally-redundant-storage"></a>本地冗余存储
本地冗余存储 (LRS) 在存储缩放单位中复制数据三次，该缩放单位托管在创建存储帐户的区域中的数据中心内。 写入请求仅在写入到全部 3 个副本中后才会成功返回。 这三个副本各驻留在一个存储扩展单元的单独容错域和升级域中。

存储扩展单元是指存储节点的机架的集合。 容错域 (FD) 是一组表示出错的物理单元的节点，可将其视为属于同一物理机架的节点。 升级域 (UD) 是一组在服务升级（推出）过程中一起升级的节点。 3 个副本将分布在一个存储扩展单元的 UD 和 FD 上，确保即使硬件故障影响单个机架或在推出期间升级节点时，仍可使用数据。

LRS 的成本最低，与其他选项相比，存储的持久性最低。 如果发生数据中心级灾难（火灾、洪灾等），所有三个副本可能都会丢失或无法恢复。 为减小此风险，建议大多数应用程序使用异地冗余存储 (GRS)。

某些情况下，可能仍需要本地冗余存储：

* 在 Azure 存储复制选项中，提供最高带宽上限。
* 如果应用程序存储可轻松重构的数据，则可以选择 LRS。


## <a name="geo-redundant-storage"></a>异地冗余存储
异地冗余存储 (GRS) 将数据复制到距主要区域数百英里以外的次要区域。如果存储帐户启用了 GRS，即使在遇到区域完全停电或导致主要区域不可恢复的灾难时，数据也能持久保存。

对于启用了 GRS 的存储帐户，更新将首先提交到主要区域，并在其中复制三次。然后，更新将异步复制到次要区域（该操作同样复制 3 次）。

使用 GRS 时，如 LRS 所述，主要和次要区域在一个存储缩放单位内管理跨单独容错域和升级域的副本。

注意事项：

* 由于异步复制涉及延迟，因此在遇到区域性灾难时，如果无法从主要区域中恢复数据，则可能会丢失尚未复制到次要区域的更改。
* 在 Azure 启动故障转移到次要区域之前，该副本不可用。如果 Azure 启动了到次要区域的故障转移，则在故障转移完成以后，用户将具有该数据的读取和写入访问权限。有关详细信息，请参阅[灾难恢复指南](/documentation/articles/storage-disaster-recovery-guidance/)。
* 如果应用程序想要从次要区域读取，用户应启用 RA-GRS。

创建存储帐户时，可以为帐户选择主要区域。次要区域是根据主要区域确定的且无法更改。下表显示了配对的主要区域和次要区域。

|主要            |次要
| ---------------   |----------------
|中国北部   |中国东部
|中国东部   |中国北部 

## <a name="read-access-geo-redundant-storage"></a>读取访问异地冗余存储
除了在 GRS 所提供的两个区域之间进行复制外，读取访问异地冗余存储 (RA-GRS) 还提供对次要位置中的数据的只读访问权限，从而在最大程度上提高存储帐户的可用性。

当启用对辅助区域中的数据的只读访问权限时，除了存储帐户的主终结点外，还可以在辅助终结点上获取数据。辅助终结点与主终结点类似，但会在帐户名称后面追加后缀`-secondary`。例如，如果 Blob 服务的主终结点是  `myaccount.blob.core.chinacloudapi.cn`，辅助终结点则是  `myaccount-secondary.blob.core.chinacloudapi.cn`。存储帐户的访问密钥对于主终结点和辅助终结点是相同的。

注意事项：

* 应用程序必须管理在使用 RA-GRS 时要与哪些终结点进行交互。
* 由于异步复制涉及延迟，因此在遇到区域性灾难时，如果无法从主要区域中恢复数据，则可能会丢失尚未复制到次要区域的更改。
* 如果 Azure 启动了到次要区域的故障转移，则在故障转移完成以后，用户将具有该数据的读取和写入访问权限。 有关详细信息，请参阅[灾难恢复指南](/documentation/articles/storage-disaster-recovery-guidance/)。 
* 要实现高可用性时使用 RA-GRS。 有关可伸缩性的指南，请查看[性能清单](/documentation/articles/storage-performance-checklist/)。

## <a name="next-steps"></a>后续步骤
- [使用 RA-GRS 存储设计高度可用的应用程序](/documentation/articles/storage-designing-ha-apps-with-ragrs/)
- [Azure 存储定价](/pricing/details/storage/)
- [关于 Azure 存储帐户](/documentation/articles/storage-create-storage-account/)
- [Azure 存储可伸缩性和性能目标](/documentation/articles/storage-scalability-targets/)
- [Azure 存储冗余选项和读取访问异地冗余存储](http://blogs.msdn.com/b/windowsazurestorage/archive/2013/12/11/introducing-read-access-geo-replicated-storage-ra-grs-for-windows-azure-storage.aspx)  
- [SOSP 论文 - Azure 存储空间：具有高度一致性的高可用云存储服务](http://blogs.msdn.com/b/windowsazurestorage/archive/2011/11/20/windows-azure-storage-a-highly-available-cloud-storage-service-with-strong-consistency.aspx)
<!--Update_Description: wording update;add anchors to subtitles-->