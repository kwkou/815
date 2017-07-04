<properties
    pageTitle="有关 Azure IaaS VM 磁盘的常见问题 (FAQ) | Azure"
    description="Azure IaaS VM 磁盘和高级磁盘（非托管）常见问题解答"
    services="storage"
    documentationcenter=""
    author="robinsh"
    manager="timlt"
    editor="tysonn" />
<tags
    ms.assetid="e2a20625-6224-4187-8401-abadc8f1de91"
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/19/2017"
    wacn.date="05/31/2017"
    ms.author="robinsh"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="4a18b6116e37e365e2d4c4e2d144d7588310292e"
    ms.openlocfilehash="669f7dc2d87b449ece117127dc000f85cbbe7c93"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/19/2017" />

# <a name="frequently-asked-questions-about-azure-iaas-vm-disks-and-unmanaged-premium-disks"></a>Azure IaaS VM 磁盘和非托管高级磁盘常见问题解答

本文提供有关高级存储的一些常见问题的解答。



## <a name="premium-disks--unmanaged"></a>高级磁盘 - 非托管

**如果 VM 使用支持高级存储的大小系列（比如 DSv2），是否可以同时附加高级和标准数据磁盘？** 

是的。

**是否可以同时将高级和标准数据磁盘附加到不支持高级存储的大小系列，例如 D、Dv2 或 F 系列？**

否。 只能将标准数据磁盘附加到不使用支持高级存储的大小系列的 VM。


**使用高级存储是否产生事务成本？**

每个磁盘大小都有固定成本，其根据 IOPS 和吞吐量的特定限制进行预配。 其他成本包括出站带宽和快照容量（如果适用）。 有关详细信息，请查看[定价页](/pricing/details/storage/)。

**可从磁盘缓存获取的 IOPS 和吞吐量限制是什么？**

DS 系列的缓存和本地 SSD 合并限制是每个核心 4000 IOPS，以及每个核心每秒 33 MB。 

## <a name="what-if-my-question-isnt-answered-here"></a>如果未在此处找到相关问题怎么办？

如果未在此处找到相关问题，请联系我们，我们会帮助你找到答案。 可将问题发布在本文末尾的评论中或 MSDN [Azure 存储论坛](https://social.msdn.microsoft.com/forums/azure/home?forum=windowsazuredata) 中，围绕本文内容，与 Azure 存储团队和其他社区成员展开探讨。