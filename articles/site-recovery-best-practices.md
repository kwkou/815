<properties
	pageTitle="准备站点恢复部署 | Azure"
	description="本文介绍如何准备使用 Azure Site Recovery 部署复制。"
	services="site-recovery"
	documentationCenter=""
	authors="rayne-wiselman"
	manager="jwhit"
	editor="tysonn"/>

<tags
	ms.service="site-recovery"
	ms.date="03/08/2016"
	wacn.date="04/05/2016"/>

# 准备 Azure Site Recovery 部署

Azure Site Recovery 服务有助于业务连续性和灾难恢复 (BCDR) 策略，因为它可以协调虚拟机和物理服务器的复制、故障转移和恢复。虚拟机可复制到 Azure 中，也可复制到本地数据中心中。如需快速概览，请阅读[什么是 Azure Site Recovery？](/documentation/articles/site-recovery-overview)。

####<a id="overview"></a> 概述

Azure Site Recovery 支持将 VMware 和 Hyper-V VM 以及物理服务器复制到 Azure 或辅助数据中心。本文介绍如何针对其中每个复制方案准备 Azure Site Recovery 部署。

请将任何评论或问题发布到本文底部，或者发布到 [Azure 恢复服务论坛](https://social.msdn.microsoft.com/Forums/zh-cn/home?forum=hypervrecovmgr)。

## Hyper-V 复制的部署要求

此表汇总了到 Azure 和辅助站点的 Hyper-V 复制（使用 VMM 和不使用 VMM）的常规部署要求。该表可帮助你了解和比较每个复制方案的常规要求。此外，还提供指向详细部署先决条件的链接。

**复制到 Azure（使用 VMM）** | **复制到 Azure（不使用 VMM）** | **复制到辅助站点（使用 VMM）**
---|---|---
**VMM**：至少有一个在 System Center 2012 R2 上运行的 VMM 服务器。VMM 服务器应至少设置一个云，其中包含一个或多个 VMM 主机组。<br/><br/> **Hyper-V**：本地数据中心内有一个或多个 Hyper-V 主机服务器至少运行 Windows Server 2012 R2。Hyper-V 服务器必须位于 VM 云中的主机组中。<br/><br/> **虚拟机**：在源 Hyper-V 服务器上至少需要一个 VM。复制到 Azure 的 VM 必须符合 [Azure 虚拟机先决条件](#azure-virtual-machine-requirements)。<br/><br/> **Azure 帐户**：你需要有 [Azure](https://azure.cn/) 帐户和订阅。<br/><br/> **Azure 存储空间**：需要使用 [Azure 存储帐户](/documentation/articles/storage-redundancy#geo-redundant-storage)来存储复制的数据。复制的数据存储在 Azure 存储空间中，Azure VM 在发生故障转移时启动。<br/><br/> **网络映射**：设置网络映射，以使同一 Azure 网络上执行故障转移的所有虚拟机都能彼此互连，而与这些虚拟机所属的恢复计划无关。如果目标 Azure 网络上有网关，虚拟机还可以连接到本地虚拟机。如果不设置网络映射，则只有同一个恢复计划中故障转移的计算机才能进行连接。<br/><br/> **提供程序/代理**：在部署期间，将在 VMM 服务器上安装 Azure Site Recovery 提供程序，并在 Hyper-V 主机服务器上安装 Azure 恢复服务代理。该提供程序将与 Azure Site Recovery 进行通信。该代理将处理源和目标 Hyper-V 服务器之间的复制。不在 VM 上安装任何内容。<br/><br/> **Internet 连接**：从 VMM 服务器和 Hyper-V 主机连接。<br/><br/> **提供程序连接**：如果提供程序将通过代理连接到站点恢复，则需要确保该代理可以访问站点恢复 URL。<br/><br/> [详细的部署先决条件](/documentation/articles/site-recovery-vmm-to-azure#before-you-start) | **Hyper-V**：在源和目标站点中，至少有一个 Hyper-V 服务器至少运行 Windows Server 2012 R2。<br/><br/> **虚拟机**：在源 Hyper-V 服务器上至少有一个 VM。复制到 Azure 的虚拟机必须符合 [Azure 虚拟机先决条件](#azure-virtual-machine-requirements)<br/><br/> **Azure 帐户**：你将需要有 [Azure](https://azure.cn/) 帐户和订阅。<br/><br/> **Azure 存储空间**：需要使用 [Azure 存储帐户](/documentation/articles/storage-redundancy#geo-redundant-storage)来存储复制的数据。<br/><br/> **提供程序/代理**：在部署期间，将在 Hyper-V 主机服务器或群集上同时安装 Azure Site Recovery 提供程序和 Azure 恢复服务代理。不在 VM 上安装任何内容。<br/><br/> **Internet 连接**：从 Hyper-V 主机连接。<br/><br/> **提供程序连接**：如果提供程序将通过代理连接，则需要确保该代理可以访问站点恢复 URL。<br/><br/> [详细的部署先决条件](/documentation/articles/site-recovery-hyper-v-site-to-azure#before-you-start#before-you-start) | **VMM**：源 VMM 服务器应至少设置一个云，其中包含一个或多个 VMM 主机组。云应具有 Hyper-V 功能配置文件集。 <br/><br/>**Hyper-V**：在源和目标站点中，有一个或多个 Hyper-V 服务器至少运行包含最新更新的 Windows Server 2012。Hyper-V 服务器必须位于 VMM 云中的主机组中。<br/><br/> **虚拟机**：在源 VMM 云中至少有一个 VM。<br/><br/> **网络映射**：设置网络映射，以使虚拟机在故障转移之后连接到适当的网络，并以最佳方式将副本虚拟机放置在目标 Hyper-V 主机服务器上。如果未配置网络映射，则故障转移之后，复制的计算机将不会连接到任何 VM 网络。<br/><br/> **存储映射**：可以选择设置存储映射，以确保虚拟机在故障转移后以最佳方式连接到存储（默认情况下，副本 VM 将存储在目标 Hyper-V 服务器上所指示的位置中）。<br/><br/> **SAN 复制** 如果要使用 SAN 复制在两个本地 VMM 站点之间复制，可以使用现有的 SAN 环境。请查看 [支持的 SAN 阵列](http://social.technet.microsoft.com/wiki/contents/articles/28317.deploying-azure-site-recovery-with-vmm-and-san-supported-storage-arrays.aspx)。<br/><br/> **提供程序/代理**：在部署期间，将在 VMM 服务器上安装 Azure Site Recovery 提供程序，以便与 Azure Site Recovery 通信。通过 LAN/VPN 在 Hyper-V 源和目标服务器之间进行复制。<br/><br/> **Internet 连接**：仅限在 VMM 服务器上。<br/><br/> **提供程序连接**：如果提供程序将通过代理连接，则需要确保可以访问站点恢复 URL。<br/><br/> [详细的部署先决条件](/documentation/articles/site-recovery-vmm-to-vmm#before-you-start)


## 复制 VMware VM 和物理服务器的部署要求

此表汇总了将 VMware VM 和 Windows/Linux 物理服务器复制到 Azure 和辅助站点的要求。

>[AZURE.NOTE] 可以使用[增强](/documentation/articles/site-recovery-vmware-to-azure-classic)部署模型或用于较旧部署的[旧版](/documentation/articles/site-recovery-vmware-to-azure-classic-legacy)模型将 VMware VM 和物理服务器复制到 Azure。下表包含每个模型的部署要求。

**复制到 Azure（增强型）** | **复制到 Azure（旧版）** | **复制到辅助站点**
---|---|---
**本地管理服务器**：在本地站点中，需要有将用作管理服务器的专用服务器。所有站点恢复组件将安装在此服务器上。<br/><br/> **附加进程服务器**：默认情况下，进程服务器安装在管理服务器上，但你可以选择安装其他附加进程服务器以缩放你的部署。<br/><br/> **VMware vCenter/ESXi**：如果你要复制 VMware VM（或要故障回复物理服务器），需要有 VM 位于其上的 vSphere ESX/ESXi。建议使用 vCenter 服务器来管理 ESXi 主机。</br><br/> **故障回复**：你需要 VMware 环境以便从 Azure 故障回复，即使你要复制物理服务器，也是如此。此外，你还需要将进程服务器设置为 Azure VM，如果你要故障回复大量流量，可能需要设置附加本地主目标服务器。[了解更多](/documentation/articles/site-recovery-failback-azure-to-vmware-classic)<br/><br/> **Azure 帐户**：你需要有 [Azure](https://azure.cn/) 帐户和订阅。<br/><br/> **Azure 存储空间**：需要使用 [Azure 存储帐户](/documentation/articles/storage-redundancy#geo-redundant-storage)来存储复制的数据。复制的数据将存储在 Azure 存储空间，Azure VM 在发生故障转移时启动。<br/><br/> **Azure 虚拟网络**：需要在发生故障转移时 Azure VM 能够连接到的 Azure 虚拟网络。若要在故障转移后进行故障回复，需要设置从 Azure 网络到本地站点的 VPN 连接（或 Azure ExpressRoute）。<br/><br/> **受保护的计算机**：至少一个 VMware 虚拟机或物理 Windows/Linux 服务器。在部署过程中，将在要复制的每个计算机上安装移动服务。<br/><br/> **连接**：如果管理服务器将通过代理连接到站点恢复，则需要确保代理服务器可以连接到特定 URL。<br/><br/> [详细的部署先决条件](/documentation/articles/site-recovery-vmware-to-azure-classic#before-you-start-deployment)。 | **主站点**：将需要设置进程服务器。<br/><br/> **故障回复**：你需要 VMware 环境以便从 Azure 故障回复，即使你要复制物理服务器，也是如此。在本地站点中，将需要设置 vContinuum 服务器和主目标服务器。在 Azure 中，将需要设置进程服务器。[了解更多](/documentation/articles/site-recovery-failback-azure-to-vmware-classic-legacy)<br/><br/> **Azure 帐户**：你需要有 [Azure](https://azure.cn/) 帐户和订阅。<br/><br/> **Azure 存储空间**：需要使用 [Azure 存储帐户](/documentation/articles/storage-redundancy#geo-redundant-storage)来存储复制的数据。复制的数据将存储在 Azure 存储空间，Azure VM 在发生故障转移时启动。<br/><br/> **Azure 基础结构 VM**：需要将配置服务器和主目标服务器设置为 Azure VM。<br/><br/> **Azure 虚拟网络**：需要一个 Azure 虚拟网络，配置服务器和主目标服务器将部署在该网络上。Azure VM 在故障转移后将连接到此网络。<br/><br/> **受保护的计算机**：至少一个 VMware 虚拟机或物理 Windows/Linux 服务器。在部署过程中，将在要复制的每个计算机上安装移动服务。<br/><br/> **连接**：如果管理服务器将通过代理连接到站点恢复，则需要确保代理服务器可以连接到特定 URL。<br/><br/> [详细的部署先决条件](/documentation/articles/site-recovery-vmware-to-azure-classic-legacy#before-you-start)。 | **主站点**：专用的 Windows 服务器（物理或 VMware 虚拟机）。<br/><br/> **辅助站点**：专用的配置服务器和主目标服务器。<br/><br/> **受保护的计算机**：至少一个 VMware 虚拟机或物理 Windows/Linux 服务器。部署期间，将在每个计算机上安装统一代理。





##<a id="virtual-machines"></a> Azure 虚拟机要求

可以部署站点恢复以复制运行受 Azure 支持的任何操作系统的虚拟机和物理服务器。这包括大多数的 Windows 和 Linux 版本。需要确保你要保护的本地虚拟机符合 Azure 要求。


**功能** | **支持** | **详细信息**
---|---|---
Hyper-V 主机操作系统 | Windows Server 2012 R2 | 如果不支持，先决条件检查将会失败
VMware 虚拟机监控程序操作系统 | 运行支持的操作系统 | [详细信息](/documentation/articles/site-recovery-vmware-to-azure-classic#before-you-start-deployment) 
来宾操作系统 | 对于从 Hyper-V 到 Azure 的复制，站点恢复支持 [Azure 支持](https://technet.microsoft.com/zh-cn/library/cc794868%28v=ws.10%29.aspx)的所有操作系统。<br/><br/> 对于 VMware 和物理服务器复制，请检查 Windows 和 Linux [先决条件](/documentation/articles/site-recovery-vmware-to-azure-classic.md#before-you-start-deployment) | 如果不支持，先决条件检查将会失败。 
来宾操作系统体系结构 | 64 位 | 如果不支持，先决条件检查将会失败
操作系统磁盘大小 | 最大 1023 GB | 如果不支持，先决条件检查将会失败
操作系统磁盘计数 | 1 | 如果不支持，先决条件检查将会失败。
数据磁盘计数 | 16 或更少（最大值取决于所创建的虚拟机大小。16 = XL） | 如果不支持，先决条件检查将会失败
数据磁盘 VHD 大小 | 最大 1023 GB | 如果不支持，先决条件检查将会失败
网络适配器 | 支持多个适配器 |
静态 IP 地址 | 支持 | 如果主虚拟机使用的是静态 IP 地址，则你可为要在 Azure 中创建的虚拟机指定静态 IP 地址。请注意，不支持为 Hyper-v 上运行的 linux 虚拟机指定静态 IP 地址。 
iSCSI 磁盘 | 不支持 | 如果不支持，先决条件检查将会失败
共享 VHD | 不支持 | 如果不支持，先决条件检查将会失败
FC 磁盘 | 不支持 | 如果不支持，先决条件检查将会失败
硬盘格式| VHD <br/><br/> VHDX | 尽管 Azure 当前不支持 VHDX，但当你故障转移到 Azure 时，站点恢复会自动将 VHDX 转换为 VHD。当你故障回复到本地时，虚拟机将继续使用 VHDX 格式。
虚拟机名称| 介于 1 和 63 个字符之间。限制为字母、数字和连字符。应以字母或数字开头和结尾 | 在 Site Recovery 中更新虚拟机属性中的值
虚拟机类型 | <p>第 1 代</p> <p>第 2 代 - Windows</p> | 支持第 2 代虚拟机：具备包含 1 或 2 个数据卷、磁盘格式为 VHDX、磁盘大小小于 300GB 的基本磁盘的操作系统磁盘类型。不支持 Linux 第 2 代虚拟机。[了解详细信息](https://azure.microsoft.com/blog/2015/04/28/disaster-recovery-to-azure-enhanced-and-were-listening/)



## 优化部署

使用以下提示来帮助优化和缩放部署。

- **操作系统卷大小**：将虚拟机复制到 Azure 时，操作系统卷必须小于 1TB。如果你的卷容量超过此值，可以在开始部署之前，手动将卷容量转移到另一个磁盘。
- **数据磁盘大小**：如果要复制到 Azure，一个虚拟机中最多可以包含 32 个数据磁盘，每个磁盘的最大大小为 1 TB。可以有效地复制和故障转移约 32 TB 的虚拟机。
- **恢复计划限制**：Site Recovery 可以扩展到数千个虚拟机。恢复计划旨在用作应一起故障转移的应用程序的模型，因此，我们可以将一个恢复计划中的计算机数目限制为 50。
- **Azure 服务限制**：每个 Azure 订阅在核心、云服务等方面附带了一组默认限制。我们建议你运行测试故障转移，以验证你的订阅中资源的可用性。可以通过 Azure 支持人员修改这些限制。
- **容量规划**：阅读站点恢复的[容量规划](/documentation/articles/site-recovery-capacity-planner)的相关信息。
- **复制带宽**：如果你的复制带宽不足，请注意：
	- **ExpressRoute**：可以配合 Azure ExpressRoute 和 WAN 优化器（如 Riverbed）来使用 Site Recovery。[详细了解](http://blogs.technet.com/b/virtualization/archive/2014/07/20/expressroute-and-azure-site-recovery.aspx)有关 ExpressRoute 的信息。
	- **复制流量**：Site Recovery 用户只能使用数据块（而不是整个 VHD）执行智能初始复制。在复制过程中，只会复制更改。
	- **网络流量**：你可以通过使用基于目标 IP 地址和端口的策略设置 [Windows QoS](https://technet.microsoft.com/zh-cn/library/hh967468.aspx)，来控制用于复制的网络流量。此外，如果要使用 Azure 备份代理复制到 Azure Site Recovery，你可以配置该代理的限制。[了解详细信息](https://support.microsoft.com/zh-cn/kb/3056159)。
- **RTO**：如果你想要度量使用 Site Recovery 时预期的恢复时间目标 (RTO)，我们建议你运行测试故障转移并查看 Site Recovery 作业，以分析完成操作所花费的时间。如果你要故障转移到 Azure，为实现最佳 RTO，我们建议你通过与 Azure 自动化和恢复计划集成来自动化所有手动操作。
- **RPO**：当你复制到 Azure 时，Site Recovery 支持近乎同步的恢复点目标 (RPO)。这种效果假设数据中心和 Azure 之间有足够的带宽。





## 后续步骤

了解并比较常规部署要求后，你可以阅读详细的先决条件并开始部署每个方案。

- [将 VMware 虚拟机复制到 Azure](/documentation/articles/site-recovery-vmware-to-azure-classic)
- [将物理服务器复制到 Azure](/documentation/articles/site-recovery-vmware-to-azure-classic)
- [将 VMM 云中的 Hyper-V 服务器复制到 Azure](/documentation/articles/site-recovery-vmm-to-azure)
- [将 Hyper-V 虚拟机（不使用 VMM）复制到 Azure](/documentation/articles/site-recovery-hyper-v-site-to-azure)
- [将 Hyper-V VM 复制到辅助站点](/documentation/articles/site-recovery-vmm-to-vmm)
- [使用 SAN 将 Hyper-V VM 复制到辅助站点](/documentation/articles/site-recovery-vmm-san)
- [复制 Hyper-V VM（带单个 VMM 服务器）](/documentation/articles/site-recovery-single-vmm)

<!---HONumber=Mooncake_0328_2016-->