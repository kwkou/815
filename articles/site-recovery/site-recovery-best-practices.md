<properties
    pageTitle="Azure Site Recovery 最佳做法 | Azure"
    description="本文介绍 Azure Site Recovery 部署的最佳做法"
    services="site-recovery"
    documentationCenter=""
    author="rayne-wiselman"
    manager="cfreeman"
    editor=""
    ROBOTS="NOINDEX, NOFOLLOW"
    redirect_url="site-recovery-support-matrix-to-azure" />
<tags
    ms.assetid="c413efcd-d750-4b22-b34b-15bcaa03934a"
    ms.service="site-recovery"
    ms.workload="backup-recovery"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.date="03/14/2017"
    wacn.date="03/31/2017"
    ms.author="raynew" />

# 准备部署 Azure Site Recovery

本文介绍如何为 Azure Site Recovery 部署做好准备。

## 保护 Hyper-V 虚拟机

可以使用多种部署选项来保护 Hyper-V 虚拟机。你可以将本地 Hyper-V VM 复制到 Azure 或辅助数据中心。每种部署有不同的要求。

**要求** | **复制到 Azure（使用 VMM）** | **将 Hyper-V VM 复制到 Azure（不使用 VMM）** | **将 Hyper-V VM 复制到辅助站点（使用 VMM）** | **详细信息**
---|---|---|---|---
**VMM** | <p>System Center 2012 R2 上运行的 VMM</p><p>至少一个 VMM 云，其中包含一个或多个 VMM 主机组。</p> | 不可用 | <p>至少在包含最新更新的 System Center 2012 SP1 上运行的主要和辅助站点中的 VMM 服务器。</p><p> 每个 VMM 服务器上至少有一个云。云应该设置了 Hyper-V 容量配置文件。</p><p> 源云应至少有一个 VMM 主机组。</p> | 可选。无需部署 System Center VMM 即可将 Hyper-V 虚拟机复制到 Azure，但如果需要部署 VMM，则需要确保正确设置 VMM 服务器。这包括确保运行正确的 VMM 版本，且已设置云。
**Hyper-V** | 本地站点至少有一个 Hyper-V 主机服务器运行 Windows Server 2012 R2 或更高版本 | <p>在源和目标网站中，至少有一个 Hyper-V 服务器至少运行已安装最新更新的 Windows Server 2012，并已连接到 Internet。</p><p> Hyper-V 服务器必须位于 VMM 云中的主机组中。</p> | <p>在源和目标网站中，至少有一个 Hyper-V 服务器至少运行已安装最新更新的 Windows Server 2012，并已连接到 Internet。</p><p> Hyper-V 服务器必须位于 VMM 云中的主机组中。</p> |
**虚拟机** | 源 Hyper-V 主机服务器上至少有一个 VM | 源 VMM 云中的 Hyper-V 主机服务器上至少有一个 VM | 源 VMM 云中的 Hyper-V 主机服务器上至少有一个 VM。 | 复制到 Azure 的 VM 必须符合 Azure 虚拟机先决条件
**Azure 帐户** | 需要一个 Azure 帐户和 Site Recovery 服务的订阅。 | 需要一个 Azure 帐户和 Site Recovery 服务的订阅。 | 不可用 | 如果没有帐户，可以先使用免费试用版。
**Azure 存储** | 必须有已启用异地复制的 Azure 存储帐户的订阅。 | 必须有已启用异地复制的 Azure 存储帐户的订阅。 | 不可用 | 该帐户应该位于 Azure Site Recovery 保管库所在的区域中，并与相同订阅关联。
**联网** | 设置网络映射，以确保同一 Azure 网络上执行故障转移的所有虚拟机都能彼此互连，与这些虚拟机所属的恢复计划无关。此外，如果在目标 Azure 网络上配置了网络网关，则虚拟机可以连接到其他本地虚拟机。如果不设置网络映射，则只有同一个恢复计划中故障转移的计算机才能进行连接。 | 不可用 | <br/><br/>设置网络映射，以确保在故障转移之后虚拟机连接到适当的网络，并以最佳方式将副本虚拟机放置在 Hyper-V 主机服务器上。如果不配置网络映射，则故障转移之后，复制的计算机将不会连接到任何 VM 网络。 | 若要使用 VMM 设置网络映射，需要确保正确配置 VMM 逻辑和 VM 网络。
**提供程序和代理** | 在部署期间，将在 VMM 服务器上安装 Azure Site Recovery 提供程序。将在 VMM 云中的 Hyper-V 服务器上安装 Azure 恢复服务代理。 | 在部署期间，将在 Hyper-V 主机服务器或群集上安装 Azure Site Recovery 提供程序和 Azure 恢复服务代理。| 在部署期间，将在 VMM 服务器上安装 Azure Site Recovery 提供程序。将在 VMM 云中的 Hyper-V 服务器上安装 Azure 恢复服务代理。 | 提供程序和代理使用加密的 HTTPS 连接通过 Internet 连接到站点恢复。若要完成提供程序连接，无需添加防火墙例外或创建特定的代理。
**Internet 连接** | 仅 VMM 服务器需要 Internet 连接 | 仅 Hyper-V 主机服务器需要 Internet 连接 | 仅 VMM 服务器需要 Internet 连接 | 不需在虚拟机上安装任何软件，也不需将虚拟机直接连接到 Internet。



## 保护物理服务器

你可以使用一些部署选项来保护 Windows/Linux 物理服务器。你可以将它们复制到 Azure 或辅助数据中心。每种部署有不同的要求。

**要求** | **将物理服务器复制到 Azure** | *将物理服务器复制到辅助站点**  
---|---|---
**主站点** | **进程服务器**：专用的 Windows 服务器（物理或虚拟） | **进程服务器**：专用的 Windows 服务器（物理）<br/><br/>  
**辅助本地站点** | 不可用 | <p>**配置服务器**：专用的 Windows 服务器（物理或虚拟）</p><p>**主目标服务器**：专用的服务器（物理或虚拟）。在 Windows 中配置 Windows 计算机保护，或者在 Linux 中配置 Linux 计算机保护。</p>
**Azure** | <p>**订阅**：需要 Site Recovery 服务的订阅。</p><p> **存储帐户**：需要一个已启用异地复制的存储帐户。该帐户应该位于 Site Recovery 保管库所在的区域中，并与相同订阅关联。</p><p> **配置服务器**：需将配置服务器设置为 Azure VM </p><p> **主目标服务器**：需将主目标服务器设置为 Azure VM </p><p> 在 Windows 中配置 Windows 计算机保护，或者在 Linux 中配置 Linux 计算机保护。</p><p> **Azure 虚拟网络**：需要一个 Azure 虚拟网络，配置服务器和主目标服务器将部署在该网络上。它应该位于 Azure Site Recovery 保管库所在的订阅和区域中 </p>| 不可用  
**虚拟机/物理服务器** | <p>至少一个物理 Windows/Linux 服务器。</p><p>部署期间将在每个计算机上安装移动服务</p>| <p>至少一个物理 Windows/Linux 服务器。</p><p> 部署期间将在每个计算机上安装统一代理。</p>




##<a name="azure-virtual-machine-requirements"></a> Azure 虚拟机要求

可以部署站点恢复以复制运行受 Azure 支持的任何操作系统的虚拟机和物理服务器。这包括大多数的 Windows 和 Linux 版本。需确保你想要保护的本地虚拟机符合 Azure 请求。


## 优化部署

使用以下提示帮助优化和缩放部署。

- **操作系统卷大小**：将虚拟机复制到 Azure 时，操作系统卷必须小于 1TB。如果你的卷容量超过此值，可以在开始部署之前，手动将卷容量转移到另一个磁盘。
- **数据磁盘大小**：如果要复制到 Azure，一个虚拟机中最多可以包含 32 个数据磁盘，每个磁盘的最大大小为 1 TB。可以有效地复制和故障转移约 32 TB 的虚拟机。
- **恢复计划限制**：Site Recovery 可以扩展到数千个虚拟机。恢复计划旨在用作应一起故障转移的应用程序的模型，因此，我们可以将一个恢复计划中的计算机数目限制为 50。
- **Azure 服务限制**：每个 Azure 订阅在核心、云服务等方面附带了一组默认限制。建议运行测试故障转移，验证订阅中资源的可用性。可以通过 Azure 支持人员修改这些限制。
- **容量规划**：缩放和性能规划。
- **复制带宽**：如果你的复制带宽不足，请注意：
	- **ExpressRoute**：可以配合 Azure ExpressRoute 和 WAN 优化器（如 Riverbed）来使用 Site Recovery。
	- **复制流量**：Site Recovery 用户只能使用数据块（而不是整个 VHD）执行智能初始复制。在复制过程中，只会复制更改。
	- **网络流量**：可以通过使用基于目标 IP 地址和端口的策略设置 Windows QoS，来控制用于复制的网络流量。此外，如果要使用 Azure 备份代理复制到 Azure Site Recovery，你可以配置该代理的限制。
- **RTO**：如果你想要度量使用 Site Recovery 时预期的恢复时间目标 (RTO)，我们建议你运行测试故障转移并查看 Site Recovery 作业，以分析完成操作所花费的时间。如果你要故障转移到 Azure，为实现最佳 RTO，我们建议你通过与 Azure 自动化和恢复计划集成来自动化所有手动操作。
- **RPO**：当你复制到 Azure 时，Site Recovery 支持近乎同步的恢复点目标 (RPO)。这假设数据中心和 Azure 之间有足够的带宽。

<!---HONumber=Mooncake_0327_2017-->
<!--Update_Descritpion: content structure simplify, move details to site-recovery-support-matrix-to-azure-->