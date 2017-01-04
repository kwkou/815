<properties
	pageTitle="什么是 Site Recovery？| Azure" 
	description="概述 Azure Site Recovery 服务并汇总部署方案。"
	services="site-recovery"
	documentationCenter=""
	authors="rayne-wiselman"
	manager="cfreeman"
	editor=""/>  


<tags
    ms.assetid="e9b97b00-0c92-4970-ae92-5166a4d43b68"
    ms.service="site-recovery"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="na"
    ms.workload="storage-backup-recovery"
    ms.date="10/30/2016"
    wacn.date="01/03/2017"
    ms.author="raynew" />  


#  什么是 Site Recovery？

欢迎使用 Azure Site Recovery！ 本文提供有关 Site Recovery 服务的简要概述，并说明它如何为业务带来帮助。

组织需要制定业务连续性和灾难恢复 (BCDR) 策略来确保应用、工作负荷和数据在计划内和计划外停机期间保持安全可用，并尽快恢复正常运行。Site Recovery 就是能够帮助实现这种策略的一个 Azure 服务。

Site Recovery 可通过协调虚拟机与本地物理服务器的复制，为 BCDR 策略提供辅助。可以将服务器和 VM 从本地主数据中心复制到云 (Azure) 或辅助数据中心。

当主站点发生服务中断时，可以故障转移到辅助站点，使工作负荷保持可访问性和可用性。当主要位置恢复正常时，你可以故障转移回到主要位置。





## 为何要部署 Site Recovery？
以下是 Site Recovery 可以为企业提供的帮助：

* **简化 BCDR** — 可以在 Azure 经典管理门户中通过单个位置复制、故障转移和恢复多个工作负荷。Site Recovery 将协调复制和故障转移，且不会截获应用程序数据。
* **提供灵活的复制** — 可以复制受支持的 Hyper-V VM 和 Windows/Linux 物理服务器上运行的工作负荷。
* **轻松执行复制测试** — Site Recovery 提供测试故障转移来支持灾难恢复练习，且不会影响生产环境。
* **故障转移和恢复** — 可以运行计划的故障转移，因为是预期的中断，所以不会丢失任何数据；或是运行非计划的故障转移，以在发生非预期的灾难时将数据损失减到最少（取决于复制频率）。主站点恢复正常时，可以故障回复到主站点。
* **消除辅助数据中心** — 可将工作负荷复制到 Azure 而不用复制到辅助站点。这就消除了维护辅助数据中心时面临的成本和复杂性。复制的数据可以存储在 Azure 存储空间，可充分利用其所提供的弹性。发生故障转移时，将使用复制的数据创建 VM。
* **与现有 BCDR 技术集成** — Site Recovery 与其他 BCDR 技术集成。例如，可以使用 Site Recovery 保护企业工作负荷的 SQL Server 后端，包括本机支持 SQL Server AlwaysOn 以便管理可用性组的故障转移。

## 我可以复制哪些内容？
下面是可以使用 Site Recovery 复制的内容摘要。

![本地到本地](./media/site-recovery-overview/asr-overview-graphic.png)  


**REPLICATE** | **REPLICATE TO** 
---|---
在 VMM 云中管理的本地 Hyper-V VM 上运行的工作负荷 | [Azure](/documentation/articles/site-recovery-vmm-to-azure/)<br/><br/> [辅助站点](/documentation/articles/site-recovery-vmm-to-vmm/) 
在 VMM 云中管理的、具有 SAN 存储的本地 Hyper-V VM 上运行的工作负荷| [辅助站点](/documentation/articles/site-recovery-vmm-san/)
在不包含 VMM 的本地 Hyper-V VM 上运行的工作负荷 | [Azure](/documentation/articles/site-recovery-hyper-v-site-to-azure/)


## 我可以保护哪些工作负荷？

Site Recovery 可实现应用程序感知的 BCDR，让工作负荷和应用在发生中断时继续以一致的方式运行。Site Recovery 提供：

- **应用程序一致的快照** — 使用单个或多层应用的应用程序一致快照进行计算机复制。除了捕获磁盘数据以外，应用程序一致的快照还捕获内存中的所有数据以及流程中的所有事务。
- **近乎同步的复制** — Site Recovery 的 Hyper-V 复制频率最低可为 30 秒。
- **灵活的恢复计划** — 可以使用外部脚本与手动操作来创建和自定义恢复计划。借助与 Azure 自动化 Runbook 的集成，只需按一下鼠标就能恢复整个应用程序堆栈。
- **与 SQL Server AlwaysOn 集成** — 可以在 Site Recovery 恢复计划中管理可用性组的故障转移。
- **自动化库** — 丰富的 Azure 自动化库，提供特定于应用程序的生产就绪型脚本，可以下载并与 Site Recovery 集成。
- **简单的网络管理** — Site Recovery 和 Azure 中的高级网络管理简化了应用程序网络要求，包括保留 IP 地址、配置负载均衡器，以及集成 Azure 流量管理器来提高网络切换的效率。

## 后续步骤
- 有关详细信息请参阅 [Site Recovery 可以保护哪些工作负荷？](/documentation/articles/site-recovery-workload/)
- 有关 Site Recovery 体系结构的详细信息，请参阅 [Site Recovery 的工作原理](/documentation/articles/site-recovery-components/)

<!---HONumber=Mooncake_Quality_Review_1230_2016-->