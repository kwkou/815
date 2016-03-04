<properties
	pageTitle="准备站点恢复部署 | Microsoft Azure"
	description="Azure Site Recovery 可以协调位于本地服务器的虚拟机和物理服务器到 Azure 或辅助数据中心的复制、故障转移和恢复。"
	services="site-recovery"
	documentationCenter=""
	authors="rayne-wiselman"
	manager="jwhit"
	editor="tysonn"/>

<tags
	ms.service="site-recovery"
	ms.date="12/07/2015"
	wacn.date="01/29/2016"/>

# 准备 Azure Site Recovery 部署

##<a id="overview"></a> 关于本文

本文介绍如何为 Azure Site Recovery 部署做好准备。如果在阅读本文后有任何问题，请在 [Azure 恢复服务论坛](https://social.msdn.microsoft.com/Forums/zh-cn/home?forum=hypervrecovmgr)上发布你的问题。

## 保护 Hyper-V 虚拟机

可以使用多种部署选项来保护 Hyper-V 虚拟机。你可以将本地 Hyper-V VM 复制到 Azure 或辅助数据中心。每种部署有不同的要求。

**要求** | **复制到 Azure（使用 VMM）** | **将 Hyper-V VM 复制到 Azure（不使用 VMM）** | **将 Hyper-V VM 复制到辅助站点（使用 VMM）** | **详细信息**
---|---|---|---|---
**VMM** | System Center 2012 R2 上运行的 VMM<br/><br/>至少一个 VMM 云，其中包含一个或多个 VMM 主机组。 | 不可用 | 至少在包含最新更新的 System Center 2012 SP1 上运行的主要和辅助站点中的 VMM 服务器。<br/><br/>每个 VMM 服务器上至少有一个云。云应该设置了 Hyper-V 容量配置文件。<br/><br/> 源云应至少有一个 VMM 主机组。 | 可选。无需部署 System Center VMM 即可将 Hyper-V 虚拟机复制到 Azure，但如果需要部署 VMM，则需要确保正确设置 VMM 服务器。这包括确保运行正确的 VMM 版本，且已设置云。
**Hyper-V** | 本地数据中心至少有一个 Hyper-V 主机服务器运行 Windows Server 2012 R2 | 在源和目标站点中，至少有一个 Hyper-V 服务器至少运行 Windows Server 2012 R2 并已连接到 Internet。<br/><br/> Hyper-V 服务器必须位于 VMM 云中的主机组中。 | 在源和目标网站中，至少有一个 Hyper-V 服务器至少运行已安装最新更新的 Windows Server 2012，并已连接到 Internet。<br/><br/> Hyper-V 服务器必须位于 VMM 云中的主机组中。 | 
**虚拟机** | 源 Hyper-V 主机服务器上至少有一个 VM | 源 VMM 云中的 Hyper-V 主机服务器上至少有一个 VM | 源 VMM 云中的 Hyper-V 主机服务器上至少有一个 VM。 | 复制到 Azure 的 VM 必须符合 [Azure 虚拟机先决条件](/documentation/articles/site-recovery-best-practices#virtual-machines)
**Azure 帐户** | 需要一个 [Microsoft Azure](/) 帐户和站点恢复服务的订阅。 | 需要一个 [Microsoft Azure](/) 帐户和站点恢复服务的订阅。 | 不可用 | 如果没有帐户，请先使用 [1 元试用版](/pricing/1rmb-trial/)。阅读有关服务的[定价](/home/features/site-recovery#price)。
**Azure 存储空间** | 必须有已启用异地复制的 Azure 存储帐户的订阅。 | 必须有已启用异地复制的 Azure 存储帐户的订阅。 | 不可用 | 该帐户应该位于 Azure Site Recovery 保管库所在的区域中，并与相同订阅关联。[阅读有关存储空间的详细信息](/documentation/articles/storage-introduction)。
**存储映射** | 不可用 | 不可用 | 你可以选择设置存储映射，以确保虚拟机在故障转移后以最佳方式连接到存储。在两个本地 VMM 站点之间复制时，默认情况下，副本虚拟机将存储在目标 Hyper-V 主机服务器上的指定位置。可以配置源和目标 VMM 服务器上的 VMM 存储分类之间的映射。 | 若要使用此功能，需要在开始部署之前设置存储分类。[了解详细信息](/documentation/articles/site-recovery-storage-mapping)。
**SAN 复制** | 不可用 | 不可用 | 如果你要使用 SAN 复制在两个在本地 VMM 站点之间复制，可以使用现有的 SAN 环境。 | 需要使用[支持的 SAN 阵列](http://social.technet.microsoft.com/wiki/contents/articles/28317.deploying-azure-site-recovery-with-vmm-and-san-supported-storage-arrays.aspx)，并且必须在 VMM 中发现和分类 SAN 存储。<br/><br/>如果当前没有复制，则需要创建 LUN 并在 VMM 控制台中配置存储。如果你已开始复制，则可以跳过此步骤。
**联网** | 设置网络映射，以确保同一 Azure 网络上执行故障转移的所有虚拟机都能彼此互连，与这些虚拟机所属的恢复计划无关。此外，如果在目标 Azure 网络上配置了网络网关，则虚拟机可以连接到其他本地虚拟机。如果不设置网络映射，则只有同一个恢复计划中故障转移的计算机才能进行连接。 | 不可用 | <br/><br/>设置网络映射，以确保在故障转移之后虚拟机连接到适当的网络，并以最佳方式将副本虚拟机放置在 Hyper-V 主机服务器上。如果不配置网络映射，则故障转移之后，复制的计算机将不会连接到任何 VM 网络。 | [详细了解](/documentation/articles/site-recovery-network-mapping)有关网络映射的信息。<br/><br/>若要使用 VMM 设置网络映射，需要确保正确配置 VMM 逻辑和 VM 网络。[详细了解](http://blogs.technet.com/b/scvmm/archive/2013/02/14/networking-in-vmm-2012-sp1-logical-networks-part-i.aspx) [VM 网络](https://technet.microsoft.com/zh-cn/library/jj721575.aspx)。此外，请阅读 [VMM 的网络注意事项](/documentation/articles/site-recovery-network-design#vmm-design)。  
**提供程序和代理** | 在部署期间，将在 VMM 服务器上安装 Azure Site Recovery 提供程序。将在 VMM 云中的 Hyper-V 服务器上安装 Azure 恢复服务代理。 | 在部署期间，将在 Hyper-V 主机服务器或群集上安装 Azure Site Recovery 提供程序和 Azure 恢复服务代理。| 在部署期间，将在 VMM 服务器上安装 Azure Site Recovery 提供程序。将在 VMM 云中的 Hyper-V 服务器上安装 Azure 恢复服务代理。 | 提供程序和代理使用加密的 HTTPS 连接通过 Internet 连接到站点恢复。你不需要为提供程序连接添加防火墙例外或创建特定的代理，但是，如果你要使用自定义代理，则应在开始部署之前允许这些 URL 通过防火墙：<br/><br/> *.hypervrecoverymanager.windowsazure.cn <br/><br/> *.accesscontrol.chinacloudapi.cn<br/><br/> *.backup.windowsazure.cn <br/><br/> *.blob.core.chinacloudapi.cn <br/><br/>*.store.core.chinacloudapi.cn 
**Internet 连接** | 只有 VMM 服务器需要 Internet 连接 | 只有 Hyper-V 主机服务器需要 Internet 连接 | 只有 VMM 服务器需要 Internet 连接 | 虚拟机上不需要安装任何软件，它们不会直接连接到 Internet。



## 保护 VMware 虚拟机或物理服务器

你可以使用一些部署选项来保护 VMware 虚拟机或 Windows/Linux 物理服务器。你可以将它们复制到 Azure 或辅助数据中心。每种部署有不同的要求。

**要求** | **将 VMware VM/物理服务器复制到 Azure** | **将 VMware VM/物理服务器复制到辅助站点**  
---|---|--- 
**主站点** | **进程服务器**：专用的 Windows 服务器（物理或虚拟） | **进程服务器**：专用的 Windows 服务器（物理或 VMware 虚拟机）<br/><br/>  
**辅助本地站点** | 不可用 | **配置服务器**：专用的 Windows 服务器（物理或虚拟）<br/><br/>**主目标服务器**：专用的服务器（物理或虚拟）。在 Windows 中配置 Windows 计算机保护，或者在 Linux 中配置 Linux 计算机保护。
**Azure** | **订阅**：需要站点恢复服务的订阅。阅读有关[定价](/home/features/site-recovery#price)的信息<br/><br/>**存储帐户**：需要一个已启用异地复制的存储帐户。该帐户应该位于站点恢复保管库所在的区域中，并与相同订阅关联。[了解详细信息](/documentation/articles/storage-introduction)。<br/><br/> **配置服务器**：需将配置服务器设置为 Azure VM<br/><br/>**主目标服务器**：需将主目标服务器设置为 Azure VM<br/><br/>在 Windows 中配置 Windows 计算机保护，或者在 Linux 中配置 Linux 计算机保护。<br/><br/> **Azure 虚拟网络**：需要一个 Azure 虚拟网络，配置服务器和主目标服务器将部署在该网络上。它应该位于 Azure Site Recovery 保管库所在的订阅和区域中。 | 不可用  
**虚拟机/物理服务器** | 至少一个 VMware 虚拟机或物理 Windows/Linux 服务器。<br/><br/>部署期间将在每个计算机上安装移动服务| 至少一个 VMware VM 或物理 Windows/Linux 服务器。<br/><br/> 部署期间将在每个计算机上安装统一代理。




##<a id="virtual-machines"></a> Azure 虚拟机要求

可以部署站点恢复以复制运行受 Azure 支持的任何操作系统的虚拟机和物理服务器。这包括大多数的 Windows 和 Linux 版本。需确保你想要保护的本地虚拟机符合 Azure 请求。


**功能** | **支持** | **详细信息**
---|---|---
Hyper-V 主机操作系统 | Windows Server 2012 R2 | 如果不支持，先决条件检查将会失败
VMware 虚拟机监控程序操作系统 | 运行支持的操作系统 | [详细信息](/documentation/articles/site-recovery-vmware-to-azure#before-you-start) 
来宾操作系统 | 对于从 Hyper-V 到 Azure 的复制，站点恢复支持 [Azure 支持](https://technet.microsoft.com/zh-cn/library/cc794868%28v=ws.10%29.aspx)的所有操作系统。<br/><br/>对于 VMware 和物理服务器复制，请检查 Windows 和 Linux [先决条件](/documentation/articles/site-recovery-vmware-to-azure#before-you-start) | 如果不支持，先决条件检查将会失败。 
来宾操作系统体系结构 | 64 位 | 如果不支持，先决条件检查将会失败
操作系统磁盘大小 | 最大 1023 GB | 如果不支持，先决条件检查将会失败
操作系统磁盘计数 | 1 | 如果不支持，先决条件检查将会失败。 
数据磁盘计数 | 16 或更少（最大值取决于所创建的虚拟机大小。16 = XL） | 如果不支持，先决条件检查将会失败
数据磁盘 VHD 大小 | 最大 1023 GB | 如果不支持，先决条件检查将会失败
网络适配器 | 支持多个适配器 |
静态 IP 地址 | 支持 | 如果主虚拟机正在使用静态 IP 地址，则你可为 Azure 中创建的虚拟机指定静态 IP 地址
iSCSI 磁盘 | 不支持 | 如果不支持，先决条件检查将会失败
共享 VHD | 不支持 | 如果不支持，先决条件检查将会失败
FC 磁盘 | 不支持 | 如果不支持，先决条件检查将会失败
硬盘格式| VHD <br/><br/> VHDX | 尽管 Azure 当前不支持 VHDX，但当你故障转移到 Azure 时，站点恢复会自动将 VHDX 转换为 VHD。当你故障回复到本地时，虚拟机将继续使用 VHDX 格式。
虚拟机名称| 介于 1 和 63 个字符之间。限制为字母、数字和连字符。应以字母或数字开头和结尾 | 在 Site Recovery 中更新虚拟机属性中的值
虚拟机类型 | <p>第 1 代</p> <p>第 2 代 - Windows</p> | 支持第 2 代虚拟机：具备包含 1 或 2 个数据卷、磁盘格式为 VHDX、磁盘大小小于 300GB 的基本磁盘的操作系统磁盘类型。不支持 Linux 第 2 代虚拟机。[了解详细信息](http://azure.microsoft.com/blog/2015/04/28/disaster-recovery-to-azure-enhanced-and-were-listening/)



## 优化部署

使用以下提示来帮助优化和缩放部署。

- **操作系统卷大小**：将虚拟机复制到 Azure 时，操作系统卷必须小于 1TB。如果你的卷容量超过此值，可以在开始部署之前，手动将卷容量转移到另一个磁盘。
- **数据磁盘大小**：如果要复制到 Azure，一个虚拟机中最多可以包含 32 个数据磁盘，每个磁盘的最大大小为 1 TB。可以有效地复制和故障转移约 32 TB 的虚拟机。
- **恢复计划限制**：Site Recovery 可以扩展到数千个虚拟机。恢复计划旨在用作应一起故障转移的应用程序的模型，因此，我们可以将一个恢复计划中的计算机数目限制为 50。
- **Azure 服务限制**：每个 Azure 订阅在核心、云服务等方面附带了一组默认限制。我们建议你运行测试故障转移，以验证你的订阅中资源的可用性。可以通过 Azure 支持人员修改这些限制。
- **容量规划**：如果要复制 Hyper-V VM，请阅读 [Capacity Planner](http://www.microsoft.com/en-us/download/details.aspx?id=39057)。
- **复制带宽**：如果你的复制带宽不足，请注意：
	- **ExpressRoute**：可以配合 Azure ExpressRoute 和 WAN 优化器（如 Riverbed）来使用 Site Recovery。[详细了解](http://blogs.technet.com/b/virtualization/archive/2014/07/20/expressroute-and-azure-site-recovery.aspx)有关 ExpressRoute 的信息。
	- **复制流量**：Site Recovery 用户只能使用数据块（而不是整个 VHD）执行智能初始复制。在复制过程中，只会复制更改。
	- **网络流量**：你可以通过使用基于目标 IP 地址和端口的策略设置 [Windows QoS](https://technet.microsoft.com/zh-cn/library/hh967468.aspx)，来控制用于复制的网络流量。此外，如果要使用 Azure 备份代理复制到 Azure Site Recovery，你可以配置该代理的限制。[了解详细信息](https://support.microsoft.com/zh-cn/kb/3056159)。
- **RTO**：如果你想要度量使用 Site Recovery 时预期的恢复时间目标 (RTO)，我们建议你运行测试故障转移并查看 Site Recovery 作业，以分析完成操作所花费的时间。如果你要故障转移到 Azure，为实现最佳 RTO，我们建议你通过与 Azure 自动化和恢复计划集成来自动化所有手动操作。
- **RPO**：当你复制到 Azure 时，Site Recovery 支持近乎同步的恢复点目标 (RPO)。这种效果假设数据中心和 Azure 之间有足够的带宽。





## 后续步骤

在了解这些最佳实践后，你可以开始部署 Site Recovery：

- [设置本地 VMM 站点与 Azure 之间的保护](/documentation/articles/site-recovery-vmm-to-azure)
- [在本地 Hyper-V 站点与 Azure 之间设置保护](/documentation/articles/site-recovery-hyper-v-site-to-azure)
- [设置两个本地 VMM 站点之间的保护](/documentation/articles/site-recovery-vmm-to-vmm)
- [使用 SAN 在两个本地 VMM 站点之间设置保护](/documentation/articles/site-recovery-vmm-san)
- [使用单个 VMM 服务器设置保护](/documentation/articles/site-recovery-single-vmm)

<!---HONumber=Mooncake_0118_2016-->