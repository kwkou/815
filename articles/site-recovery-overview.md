<properties
	pageTitle="什么是 Site Recovery？| Windows Azure" 
	description="Azure Site Recovery 可以协调位于本地的虚拟机和物理服务器到 Azure 或辅助本地站点的复制、故障转移和恢复。" 
	services="site-recovery" 
	documentationCenter="" 
	authors="rayne-wiselman" 
	manager="jwhit" 
	editor=""/>

<tags
	ms.service="site-recovery"
	ms.date="12/14/2015" 
	wacn.date="01/14/2016"/>

#  什么是 Site Recovery？

Site Recovery 是有助于实现业务连续性和灾难恢复 (BCDR) 策略的 Azure 服务，因为它可以协调从本地服务器和虚拟机到辅助本地数据中心或 Azure 的复制。Site Recovery 负责进行复制，只需单击一下，即可开始故障转移和恢复操作。阅读在[常见问题](/documentation/articles/site-recovery-faq)中列出的常见问题


## 为何使用 Site Recovery？ 

- **简化 BCDR 操作** - Site Recovery 可以简化本地工作负荷和应用程序的复制、故障转移和恢复操作。
- **增强复制的灵活性** - 你可以复制本地服务器、Hyper-V 虚拟机和 VMware 虚拟机。执行初始复制时，Site Recovery 将使用智能复制 - 只复制数据块而不是整个 VHD。对于正在进行的复制，只会复制增量更改。Site Recovery 支持脱机数据传输，并可与 WAN 优化器配合工作。 
- **不再需要辅助数据中心** - Site Recovery 可以自动执行数据中心之间的复制，但它也允许你复制到 Azure，因此不需要辅助性的现场位置。复制的数据可以存储在 Azure 存储空间，可充分利用其所提供的弹性。


## 部署方案

下表汇总了 Site Recovery 支持的复制方案。

**复制目标** | **复制源（本地）** | **详细信息** | **文章**
---|---|---|---
VMware 虚拟机 | 本地 VMware 服务器 | Azure 存储空间 | [部署](/documentation/articles/site-recovery-vmware-to-azure)
Windows/Linux 物理服务器 | 本地物理服务器 | Azure 存储空间 | [部署](/documentation/articles/site-recovery-vmware-to-azure)
Hyper-V 虚拟机 | VMM 云中的本地 Hyper-V 主机服务器 | Azure 存储空间 | [部署](/documentation/articles/site-recovery-vmm-to-azure)
Hyper-V 虚拟机 | 本地 Hyper-V 站点（一个或多个 Hyper-V 主机服务器） | Azure 存储空间 | [部署](/documentation/articles/site-recovery-hyper-v-site-to-azure)
本地 Hyper-V 虚拟机| VMM 云中的本地 Hyper-V 主机服务器 | VMM 云中的辅助数据中心的本地 Hyper-V 主机服务器 | [部署](/documentation/articles/site-recovery-vmm-to-vmm)
Hyper-V 虚拟机 | 使用 SAN 存储的 VMM 云中的本地 Hyper-V 主机服务器| 使用 SAN 存储的 VMM 云中的辅助数据中心的本地 Hyper-V 主机服务器 | [部署](/documentation/articles/site-recovery-vmm-san)
VMware 虚拟机 | 本地 VMware 服务器 | 运行 VMware 的辅助数据中心 | [部署](/documentation/articles/site-recovery-vmware-to-vmware) 
Windows/Linux 物理服务器 | 本地物理服务器 | 辅助数据中心 | [部署](/documentation/articles/site-recovery-vmware-to-vmware) 

以下关系图概述了这些内容。

![本地到本地](./media/site-recovery-overview/asr-overview-graphic.png)

## 我可以保护哪些工作负荷？

Site Recovery 有助于实现设备感知型业务连续性。可以使用 Site Recovery 来协调 Windows 和第三方应用的灾难恢复。这种应用程序感知型保护提供以下功能：


- 几乎同步的复制，Hyper-V 的 RPO 低至 30 秒，可以针对 VMware 持续进行复制，满足大多数关键应用程序的需要。
- 针对单层或 N 层应用程序的应用程序一致快照
- 集成 SQL Server AlwaysOn，纳入了其他应用程序级别的复制技术，包括 Active Directory 复制、Exchange DAGS 和 Oracle 数据防护。
- 灵活的恢复计划，一次单击即可恢复整个应用程序堆栈，包括外部脚本或手动操作。 
- Site Recovery 和 Azure 中的高级网络管理可以简化应用的网络要求，包括保留 IP 地址、配置负载平衡器或集成 Azure 流量管理器以降低 RTO 网络切换数。
- 丰富的自动化库，提供特定于应用程序的生产就绪型脚本，可以下载并与 Site Recovery 集成。  


详细信息请参阅 [Site Recovery 可以保护哪些工作负荷？](/documentation/articles/site-recovery-workload)。


## 后续步骤

完成本概述后，可[详细了解](/documentation/articles/site-recovery-components) Site Recovery 体系结构。

<!---HONumber=Mooncake_0104_2016-->