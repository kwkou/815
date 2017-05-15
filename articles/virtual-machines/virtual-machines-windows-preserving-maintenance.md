<properties
    pageTitle="Azure 中 Windows VM 的 VM 保留维护 | Azure"
    description="用于内存保留更新的就地 VM 迁移。"
    services="virtual-machines-windows"
    documentationcenter=""
    author=""
    manager="timlt"
    editor=""
    tags="azure-service-management,azure-resource-manager" />
<tags
    ms.assetid=""
    ms.service="virtual-machines-windows"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="vm-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/27/2017"
    wacn.date="05/15/2017"
    ms.author=""
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="f2572e61b06ca00c069e6f5243dc18c2e2041c2f"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="vm-preserving-maintenance-with-in-place-vm-migration"></a>使用就地 VM 迁移的 VM 保留维护

尽管大多数更新对托管的 VM 无任何影响，但在某些情况下，组件或服务更新会对运行的 VM（未完全重启虚拟机）产生一些干扰。

这些更新通过启用就地实时迁移的技术（也称为“内存保留更新”）实现。 更新主机时，虚拟机进入“暂停”状态，以保留 RAM 中的内存，而托管环境（例如基础操作系统）则应用必要的更新和修补程序。
虚拟机在暂停后 30 秒内恢复正常。
恢复后，虚拟机的时钟将自动同步。

并非所有更新都可以通过使用此机制进行部署，但如果短暂时间较短，使用此方法部署更新可大大减少对虚拟机的影响。

多实例更新（针对可用性集中的虚拟机）一次应用一个更新域。

通过调用元数据服务计划事件，虚拟机中运行的应用程序可获知即将进行的更新。 有关计划事件的详细信息，请参阅 [Azure 元数据服务 - 计划事件](/documentation/articles/virtual-machines-scheduled-events/)。