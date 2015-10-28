<properties
	pageTitle="Site Recovery 部署最佳实践"
	description="Azure Site Recovery 可以协调位于本地服务器中的虚拟机到 Azure 或辅助数据中心的复制、故障转移和恢复。"
	services="site-recovery"
	documentationCenter=""
	authors="rayne-wiselman"
	manager="jwhit"
	editor="tysonn"/>

<tags
	ms.service="site-recovery"
	ms.date="05/08/2015"
	wacn.date="10/03/2015"/>

# Site Recovery 部署最佳实践


<a id="overview" name="overview" href="#overview">

## 关于本文

本文包括你部署 Azure Site Recovery 之前应阅读并实施的最佳实践。如果你正在寻找 Site Recovery 和相关部署方案的简介，请阅读 [Site Recovery 概述](/documentation/articles/hyper-v-recovery-manager-overview)。

如果在阅读本文后有任何问题，请在 [Azure 恢复服务论坛](https://social.msdn.microsoft.com/Forums/zh-cn/home?forum=hypervrecovmgr)上发布你的问题。


## 准备 Azure

- **Azure 帐户**：你需要一个 [Windows Azure](http://www.windowsazure.cn/) 帐户。如果没有，请先使用[试用帐户](/pricing/1rmb-trial/)。
- 阅读 Site Recovery 服务的[定价](/home/features/site-recovery/#price)。
- **Azure 存储空间**：如果通过复制到 Azure 的方式部署 Site Recovery，则你需要一个 Azure 存储帐户。你可以在部署期间设置该帐户，也可以在开始之前准备该帐户。该帐户应已启用地域复制。它应该位于 Azure Site Recovery 保管库所在的区域中，并与相同订阅关联。请阅读 [Windows Azure 存储空间简介](/documentation/articles/storage-introduction)。

## 虚拟机

如果你要复制到 Azure 存储空间，请注意以下问题：

- **支持的操作系统**：你可以部署 Site Recovery，以协调运行 Azure 所支持的任何操作系统的虚拟机的保护。这包括大多数的 Windows 和 Linux 版本。
- **VHDX 支持**：尽管 Azure 当前不支持 VHDX，但当你故障转移到 Azure 时，Site Recovery 会自动将 VHDX 转换为 VHD。当你故障回复到本地时，虚拟机将继续使用 VHDX 格式。
- **Azure 要求**：确保你要保护的虚拟机符合 Azure 要求。

**虚拟机功能** | **支持** | **详细信息**
---|---|---
主机操作系统 | Windows Server 2012 R2 | 如果不支持，先决条件检查将会失败
来宾操作系统 | <p>Windows Server 2008 R2 或更高版本</p><p>Linux：Centos、openSUSE、SUSE、Ubuntu</p> | 如果不支持，先决条件检查将会失败。在 VMM 控制台中更新值。
来宾操作系统体系结构 | 64 位 | 如果不支持，先决条件检查将会失败
操作系统磁盘大小 | 介于 20 MB 和 1023 GB 之间 | 如果不支持，先决条件检查将会失败
操作系统磁盘计数 | 1 | 如果不支持，先决条件检查将会失败。在 VMM 控制台中更新值
数据磁盘计数 | 16 或更少（最大值取决于所创建的虚拟机大小。16 = XL） | 如果不支持，先决条件检查将会失败
数据磁盘 VHD 大小 | 介于 20 MB 和 1023 GB 之间 | 如果不支持，先决条件检查将会失败
网络适配器 | 支持多个适配器 |
静态 IP 地址 | 支持 | 如果主虚拟机正在使用静态 IP 地址，则你可为 Azure 中创建的虚拟机指定静态 IP 地址
iSCSI 磁盘 | 不支持 | 如果不支持，先决条件检查将会失败
共享 VHD | 不支持 | 如果不支持，先决条件检查将会失败
FC 磁盘 | 不支持 | 如果不支持，先决条件检查将会失败
硬盘格式| <p>VHD</p><p>VHDX（仅限第 1 代）</p> | 如果不支持，先决条件检查将会失败
虚拟机名称| 介于 1 和 63 个字符之间。限制为字母、数字和连字符。应以字母或数字开头和结尾 | 在 Site Recovery 中更新虚拟机属性中的值
虚拟机类型 | <p>第 1 代</p> <p>第 2 代：[阅读详细信息](http://azure.microsoft.com/blog/2015/04/28/disaster-recovery-to-azure-enhanced-and-were-listening/) </p> |


## VMM 服务器

Site Recovery 可按如下方式，协调 System Center Virtual Machine Manager (VMM) 云中 Hyper-V 主机服务器上的虚拟机的复制：从现场 VMM 服务器复制到 Azure（使用 Hyper-V 副本）- 复制到辅助本地站点（使用 Hyper-V 副本）。我们建议在主站点和辅助站点中各一个部署 VMM 服务器。但是，如果需要，你可以为两个站点[部署单个 VMM 服务器](/documentation/articles/site-recovery-single-vmm)。- 复制到辅助本地站点（使用 SAN）。每个站点需要一个包含 VMM 服务器的主要和辅助数据中心。如果你想要使用 Site Recovery 部署 VMM，则需要设置你的 VMM 基础结构。如果你没有 VMM 服务器，请在[此处](/documentation/articles/site-recovery-hyper-v-site-to-azure)阅读详细信息。


### 验证 VMM 版本

**支持的 VMM 版本** | **支持的 Site Recovery 方案** | **支持的 Hyper-V 主机（源/目标）** | **支持的提供程序/代理**
---|---|---|---
至少装有 Update Rollup 5 的 System Center 2012 R2 上的 VMM（仅限群集中的虚拟机） | 使用 SAN 复制提供本地到本地的保护 | 至少为装有最新更新的 Windows Server 2012 | 最新版本的 Azure Site Recovery 提供程序（安装在 VMM 服务器上） |
System Center 2012 R2 上的 VMM（建议）（群集或独立计算机） | <p>使用 SAN 复制提供本地到本地的保护</p> <p>使用 Hyper-V 复制提供本地到 Azure 的保护</p> | <p>至少为装有最新更新的 Windows Server 2012</p><p>Windows Server 2012 R2（源，不适用于目标）</p> | <p>最新版本的 Azure Site Recovery 提供程序（安装在 VMM 服务器上）</p> <p>最新版本的 Azure 恢复服务代理（只能安装在要复制到 Azure 的 Hyper-V 主机服务器上）</p>
装有最新累积更新的 System Center 2012 SP1 上的 VMM（群集或独立计算机） | 本地到本地的保护 | 装有最新更新的 Windows Server 2012 | 最新版本的 Azure Site Recovery 提供程序（安装在 VMM 服务器上） |

### 设置 VMM 云基础结构

无论你想要使用 Site Recovery 部署哪种 VMM 方案，都需要在 VMM 中至少创建两个私有云（一个在源 VMM 服务器上，一个在目标上），以便在使用 VMM 控制台定义、管理和访问云及其基础资源时可以利用私有云的优势。VMM 云将收集并提供一组聚合资源，这些资源的使用受云容量设置和用户角色配额的限制。自助服务用户可以管理和使用云的资源，而无需完全了解云的底层基础结构。可以轻松添加资源，以缩小或增大云的弹性。如果你需要设置云，请使用以下资源：


- [VMM 2012 和云](http://www.server-log.com/blog/2011/8/26/vmm-2012-and-the-clouds.html)。
- [配置 VMM 云结构](https://msdn.microsoft.com/zh-cn/library/azure/dn469075.aspx#BKMK_Fabric)
- [在 VMM 中创建私有云](https://technet.microsoft.com/zh-cn/library/jj860425.aspx)
- [演练：使用 System Center 2012 SP1 VMM 创建私有云](http://blogs.technet.com/b/keithmayer/archive/2013/04/18/walkthrough-creating-private-clouds-with-system-center-2012-sp1-virtual-machine-manager-build-your-private-cloud-in-a-month.aspx)




## 提供程序和代理

提供程序和代理安装在本地服务器上，因此可以连接到 Azure 中的 Site Recovery。使用加密的 HTTPS 连接通过 Internet 进行连接。

- **提供程序要求**：在部署过程中，你将在本地服务器上安装几个组件：
	- **在 VMM 服务器上**：在要注册到保管库中的 VMM 服务器上安装 Azure Site Recovery 提供程序。
	- **在 VMM 云中的 Hyper-V 主机服务器上**：安装 Azure 恢复服务代理。
	- **在没有 VMM 服务器的 Hyper-V 主机服务器上**：在每个 Hyper-V 主机服务器或群集节点上安装 Azure Site Recovery 提供程序和 Azure 恢复服务代理。
-  **提供程序安装**：在安装提供程序和代理时，请注意：
	-  应在 Site Recovery 保管库中的所有服务器上安装相同版本的提供程序。不支持在单个保管库中使用不同的版本。
	-  如果你要从位于 VMM 云中的 Hyper-V 服务器复制到 Azure，则 VMM 服务器应运行 System Center 2012 R2。如果你要复制到辅助数据中心，VMM 应运行 System Center 2012 SP1 或 R2。
- **代理**：无需添加防火墙例外或创建特定的代理。如果你确实想要使用自定义代理建立提供程序连接，则在开始部署之前请做好以下准备工作：

	- 在安装提供程序之前设置自定义代理服务器。
	- 允许这些 URL 通过防火墙：
		- hypervrecoverymanager.windowsazure.cn
		- *.accesscontrol.chinacloudapi.cn
- *.backup.windowsazure.cn
- *.blob.core.chinacloudapi.cn
- *.store.core.chinacloudapi.cn

	- 如果你要部署包含 VMM 的 Site Recovery 并使用自定义代理，系统会使用你在 Site Recovery 门户的自定义代理设置中指定的代理凭据自动创建一个 VMM RunAs 帐户 (DRAProxyAccount)。你需要设置代理服务器，以便此帐户能够成功完成身份验证。



## 本地连接

- **提供程序连接**：提供程序和代理安装在本地服务器上，因此可以连接到 Site Recovery。它们使用加密的 HTTPS 连接通过 Internet 连接到 Site Recovery。你无需添加防火墙例外或创建特定的代理。
- **Internet 连接**：需要按如下所述设置服务器 Internet 连接：
	- 如果你要保护 VMM 云中 Hyper-V 主机服务器上的虚拟机，则只有 VMM 服务器才需要连接到 Internet。
	- 如果你要保护不包含 VMM 服务器的 Hyper-V 主机服务器上的虚拟机，则只有这些 Hyper-V 主机服务器才需要连接到 Internet。
	- 你不需要从想要保护的虚拟机建立任何 Internet 连接。
- **VPN 站点到站点连接**：不需要建立 VPN 站点到站点连接来连接到 Site Recovery。但是，如果建立了站点到站点连接，则你可以像在故障转移之前一样，访问已故障转移的虚拟机。请注意，在复制到 Azure 时，你需要在本地站点和特定 Azure 网络之间设置 VPN 站点到站点网络。该网络不会用于加密的复制流量。这些流量将通过 Internet 发往订阅中的 Azure 存储帐户。
- **故障转移后的连接**：为了确保故障转移到 Azure 之后用户可以连接到虚拟机，请执行以下操作：
	- 在故障转移之前为虚拟机启用 RDP。
	- 在故障转移后使用站点到站点连接以便可以像以前一样连接到这些虚拟机，或者为计算机启用 RDP 终结点。

## 存储

- **Azure 存储帐户**：如果要复制到 Azure，则你需要一个 Azure 存储帐户。该帐户应已启用地域复制。它应该位于 Azure Site Recovery 保管库所在的区域中，并与相同订阅关联。若要了解详细信息，请阅读 [Windows Azure 存储空间简介](/documentation/articles/storage-introduction)。
- **存储映射**：如果要复制本地 VMM 服务器上的虚拟机，你可以设置存储映射，以确保虚拟机在故障转移后以最佳方式连接到存储。在两个本地 VMM 站点之间复制时，默认情况下，副本虚拟机将存储在目标 Hyper-V 主机服务器上的指定位置。可以配置源和目标 VMM 服务器上的 VMM 存储分类之间的映射。如果你要使用此功能，请确保在开始部署之前设置存储分类。[详细了解](/documentation/articles/site-recovery-storage-mapping)有关存储映射的信息。
- **SAN**：如果你要使用 SAN 复制在两个在本地站点之间复制，请注意：
	- 使用 SAN 复制只能将 Hyper-V 虚拟机复制到辅助数据中心，而不能复制到 Azure。
	- 可以使用现有的 SAN 环境。无需在 SAN 阵列上进行任何更改。
	- 需要检查你的 SAN 阵列是否[受支持](http://social.technet.microsoft.com/wiki/contents/articles/28317.deploying-azure-site-recovery-with-vmm-and-san-supported-storage-arrays.aspx)。
	- 对于 SAN 部署，需要在源和目标站点中提供两个 VMM 服务器。
	- 如果你要部署 SAN 复制，请确保 Hyper-V 主机群集正在运行 Windows Server 2012 或 2012 R2。[详细了解](https://technet.microsoft.com/zh-cn/library/cc794868%28v=ws.10%29.aspx)不同版本的 Hyper-V 支持的来宾操作系统。
	- 你需要发现并分类 VMM 中的 SAN 存储。
	- 如果你尚未开始复制，则在发现之后，需要在 VMM 控制台中创建 LUN 并分配存储。如果你已开始复制，则可以跳过此步骤。
	- 此[文章](/documentation/articles/site-recovery-vmm-san)提供了完整说明。


## 网络

- **了解网络注意事项**：[详细了解](http://azure.microsoft.com/blog/2014/09/04/networking-infrastructure-setup-for-microsoft-azure-as-a-disaster-recovery-site/)有关网络注意事项和最佳实践的信息。

- **设置网络映射**：网络映射是部署 VMM 和 Site Recovery 时的要素。网络映射可将复制的虚拟机放在最佳的目标 Hyper-V 主机服务器上，并确保复制的虚拟机在故障转移后可连接到适当的网络。你可以在复制到 Azure 或辅助数据中心时配置网络映射：
	- **到 Azure 的网络映射**：如果你要复制到 Azure，网络映射可确保同一网络上执行故障转移的所有虚拟机都能彼此互连，与这些虚拟机所属的恢复计划无关。此外，如果在目标 Azure 网络上配置了网络网关，则虚拟机可以连接到其他本地虚拟机。如果不设置网络映射，则只有同一个恢复计划中故障转移的计算机才能进行连接。
	- **到辅助站点的网络映射**：如果要复制到辅助 VMM 站点，网络映射可确保虚拟机在故障转移后连接到适当的网络，确保以最佳方式将副本虚拟机放置在 Hyper-V 主机服务器上。如果不配置网络映射，则复制的计算机将不会连接到任何 VM 网络。
	- [详细了解](/documentation/articles/site-recovery-network-mapping)有关网络映射的信息。
- **设置 VM 网络**：
	- 在 VMM 中正确配置逻辑和 VM 网络。阅读有关[逻辑网络](http://blogs.technet.com/b/scvmm/archive/2013/02/14/networking-in-vmm-2012-sp1-logical-networks-part-i.aspx)和 [VM 网络](https://technet.microsoft.com/zh-cn/library/jj721575.aspx)的信息。
	- 确保源 VMM 服务器上的所有虚拟机已连接到 VM 网络。
	- 检查 VM 网络是否已链接到与云相关联的逻辑网络。
	- 如果要复制到 Azure，请在 Azure 中创建虚拟网络。注意，可以将多个 VM 网络映射到单个 Azure 网络。阅读[虚拟网络配置任务](/documentation/articles/vpn-gateway-site-to-site-create)。

## 优化性能

- **操作系统卷大小**：将虚拟机复制到 Azure 时，操作系统卷必须小于 127 GB。如果你的卷容量超过此值，可以在开始部署之前，手动将卷容量转移到另一个磁盘。
- **数据磁盘大小**：如果要复制到 Azure，一个虚拟机中最多可以包含 32 个数据磁盘，每个磁盘的最大大小为 1 TB。可以有效地复制和故障转移约 32 TB 的虚拟机。
- **恢复计划限制**：Site Recovery 可以扩展到数千个虚拟机。恢复计划旨在用作应一起故障转移的应用程序的模型，因此，我们可以将一个恢复计划中的计算机数目限制为 50。
- **Azure 服务限制**：每个 Azure 订阅在核心、云服务等方面附带了一组默认限制。我们建议你运行测试故障转移，以验证你的订阅中资源的可用性。可以通过 Azure 支持人员修改这些限制。
- **容量规划**：要获得指导，请使用 Hyper-V 副本容量规划器。
- **复制带宽**：如果你的复制带宽不足，请注意：
	- **ExpressRoute**：可以配合 Azure ExpressRoute 和 WAN 优化器（如 Riverbed）来使用 Site Recovery。[详细了解](http://blogs.technet.com/b/virtualization/archive/2014/07/20/expressroute-and-azure-site-recovery.aspx)有关 ExpressRoute 的信息。
	- **复制流量**：Site Recovery 用户只能使用数据块（而不是整个 VHD）执行智能初始复制。在复制过程中，只会复制更改。
	- **网络流量**：你可以通过使用基于目标 IP 地址和端口的策略设置 [Windows QoS](https://technet.microsoft.com/zh-cn/library/hh967468.aspx)，来控制用于复制的网络流量。此外，如果要使用 Azure 备份代理复制到 Azure Site Recovery，你可以配置该代理的限制。<!--[了解详细信息](https://msdn.microsoft.com/zh-cn/library/azure/dn168844.aspx)。-->
- **RTO**：如果你想要度量使用 Site Recovery 时预期的恢复时间目标 (RTO)，我们建议你运行测试故障转移并查看 Site Recovery 作业，以分析完成操作所花费的时间。如果你要故障转移到 Azure，为实现最佳 RTO，我们建议你通过与 Azure 自动化和恢复计划集成来自动化所有手动操作。
- **RPO**：当你复制到 Azure 时，Site Recovery 支持近乎同步的恢复点目标 (RPO)。这种效果假设数据中心和 Azure 之间有足够的带宽。

## 故障转移
- **主站点中断**：如果你正在从一个本地数据中心复制到另一个本地数据中心，并且两个数据中心的主站点发生中断，请从 Site Recovery 门户运行非计划的故障转移。运行故障转移不需要从主数据中心建立连接。
- **故障转移到辅助站点后保留 IP 地址**：如果在源虚拟机故障转移到辅助数据中心后你要保留它的 IP 地址，请遵照[此处](http://blogs.technet.com/b/scvmm/archive/2014/04/04/retaining-ip-address-after-failover-using-hyper-v-recovery-manager.aspx)所述的步骤。
- **故障转移到 Azure 后保留 IP 地址**：可以在虚拟机的“配置”选项卡中指定要分配给已故障转移 VM 的 IP。有关详细信息，请查看[如何配置虚拟机的网络属性](/documentation/articles/site-recovery-vmm-to-azure#step-8-enable-protection-for-virtual-machines)
- **保留公共 IP 地址**：如果你要在故障转移到辅助站点后保留公共 IP 地址，Site Recovery 不会阻止你执行此操作（如果你的 ISP 支持这种做法）。故障转移到 Azure 后无法保留公共 IP 地址。
- **保留 Azure 中的非 RFC 内部地址**：你可以在故障转移到 Azure 后保留非 RFC 1918 地址空间。
- **部分故障转移到辅助数据中心**：如果你要将部分站点故障转移到辅助数据中心并想要连接回到主站点，你可以使用站点到站点 VPN 将辅助站点上已故障转移的应用程序连接到主站点上运行的基础结构组件。请注意，如果故障转移了整个子网，则你可以保留虚拟机 IP 地址。如果故障转移了部分子网，则你无法保留虚拟机 IP 地址，因为无法在站点之间拆分子网。
- **部分故障转移到 Azure**：如果你要将部分站点故障转移到 Azure 并想要连接回到主站点，你可以使用站点到站点 VPN 将 Azure 中已故障转移的应用程序连接到主站点上运行的基础结构组件。请注意，如果故障转移了整个子网，则你可以保留虚拟机 IP 地址。如果故障转移了部分子网，则你无法保留虚拟机 IP 地址，因为无法在站点之间拆分子网。
- **保留驱动器号**：如果你要在故障转移后保留虚拟机上的驱动器号，可将虚拟机的 SAN 策略设置为“打开”。[了解详细信息](https://technet.microsoft.com/zh-cn/library/gg252636.aspx)。
- **故障转移后将客户端请求路由到 Azure**：Site Recovery 可与 Azure 流量管理器配合工作，以便在故障转移后将客户端请求路由到你的应用程序。可以使用恢复计划中的脚本（配合 Azure 自动化）来执行 DNS 更新。

## 集成

- **与其他 BCDR 技术集成**：Site Recovery 与其他业务连续性和灾难恢复 (BCDR) 技术相集成。对于基于应用程序的复制，我们建议使用 SQL Sever AlwaysOn 来复制运行数据库的虚拟机，并对前端虚拟机使用 Hyper-V 副本。AlwaysOn 要求主站点和辅助站点中有 SQL Server Enterprise 许可证，并在订阅中运行 Azure 虚拟机。如果应用程序可以处理一段时间的停机，则 Site Recovery 内置应用程序可以发挥作用，在这种情况下，我们建议使用 Site Recovery 来协调数据库、应用程序和前端虚拟机的复制。使用这种方法可以节省购买所需 SQL Server 版本的费用、许可证数量和在 Azure 中随时运行虚拟机的成本。

## 后续步骤

在了解这些最佳实践后，你可以开始部署 Site Recovery：

- [设置本地 VMM 站点与 Azure 之间的保护](/documentation/articles/site-recovery-vmm-to-azure)
- [在本地 Hyper-V 站点与 Azure 之间设置保护](/documentation/articles/site-recovery-hyper-v-site-to-azure)
- [设置两个本地 VMM 站点之间的保护](/documentation/articles/site-recovery-vmm-to-vmm)
- [使用 SAN 在两个本地 VMM 站点之间设置保护](/documentation/articles/site-recovery-vmm-san)
- [使用单个 VMM 服务器设置保护](/documentation/articles/site-recovery-single-vmm)

<!---HONumber=71-->