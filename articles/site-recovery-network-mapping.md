<properties
	pageTitle="准备网络映射，以便在 Azure Site Recovery 中通过 VMM 进行 Hyper-V 虚拟机保护 | Azure"
	description="设置网络映射，以便进行从本地数据中心到 Azure 或者到辅助站点的 Hyper-V 虚拟机复制。"
	services="site-recovery"
	documentationCenter=""
	authors="rayne-wiselman"
	manager="jwhit"
	editor=""/>

<tags
	ms.service="site-recovery"
	ms.date="02/22/2016"
	wacn.date="04/05/2016"/>


# 准备网络映射，以便在 Azure Site Recovery 中通过 VMM 进行 Hyper-V 虚拟机保护

Azure Site Recovery 有助于业务连续性和灾难恢复 (BCDR) 策略，因为它可以协调虚拟机和物理服务器的复制、故障转移和恢复。

本文介绍网络映射，你可以通过网络映射来优化网络设置的配置，以便使用 Site Recovery 在两个本地数据中心之间或本地数据中心与 Azure 之间复制位于 VMM 云中的 Hyper-V 虚拟机。请注意，如果你是在没有 VMM 云的情况下复制 Hyper-V VM，或者是复制 VMware VM 或物理服务器，则本文与之不相关。

请将任何评论或问题发布到本文底部，或者发布到 [Azure 恢复服务论坛](https://social.msdn.microsoft.com/Forums/zh-cn/home?forum=hypervrecovmgr)。


## 概述

通过部署 Azure Site Recovery 将 Hyper-V 虚拟机复制到 Azure 或辅助数据中心时，可通过 Hyper-V 副本或 SAN 复制使用网络映射。

- **在两个本地数据中心之间复制 VMM 云中的 Hyper-V 虚拟机** - 网络映射功能可以在源 VMM 服务器上的 VM 网络和目标 VMM 服务器上的 VM 网络之间进行映射，以便执行以下操作：

	- **在故障转移后连接虚拟机** - 确保在故障转移后，虚拟机会连接到适当的网络。副本虚拟机将连接到已映射到源网络的目标网络。
	- **将副本虚拟机放到主机服务器上** - 以最佳方式在 Hyper-V 主机服务器上放置副本虚拟机。副本虚拟机将放置在可访问映射 VM 网络的主机上。
	- **无网络映射** - 如果未配置网络映射，则复制的虚拟机在故障转移后将不会连接到任何 VM 网络。

- **将本地 VMM 云中的 Hyper-V 虚拟机复制到 Azure** - 网络映射功能将在源 VMM 服务器上的 VM 网络与目标 Azure 网络之间进行映射，以便执行以下操作：
	- **在故障转移后连接虚拟机** - 在同一网络上进行故障转移的所有计算机都可以彼此连接到对方，不管它们位于哪个恢复计划中。
	- **网关** - 如果在目标 Azure 网络上设置了网络网关，则虚拟机可以连接到其他本地虚拟机。
	- **无网络映射** - 如果没有配置网络映射，则只有在同一恢复计划中进行故障转移的虚拟机能够在故障转移到 Azure 后彼此连接到对方。


## 网络映射示例

可以在两个 VMM 服务器上的 VM 网络之间配置网络映射；如果两个站点由同一个服务器管理，则在可以在单个 VMM 服务器上的两个对应 VM 网络之间配置网络映射。在正确地配置映射且启用复制后，位于主位置的虚拟机将连接到网络，其位于目标位置的副本将连接到其映射网络。

如果已经在 VMM 中正确地设置了网络，则当你在网络映射期间选择目标 VM 网络时，将显示使用源 VM 网络的 VMM 源云以及用于保护的目标云上的可用目标 VM 网络。

以下示例演示了此机制。我们来考察一个在北京和上海具有两个位置的组织。

**位置** | **VMM 服务器** | **VM 网络** | **映射到**
---|---|---|---
北京 | VMM-Beijing| VMNetwork1-Beijing | 映射到 VMNetwork1-Shanghai
 | | VMNetwork2-Beijing | 未映射
上海 | VMM-Shanghai| VMNetwork1-Shanghai | 映射到 VMNetwork1-Beijing
 | | VMNetwork1-Shanghai | 未映射

在此示例中：

- 当为任何连接到 VMNetwork1-Beijing 的虚拟机创建副本虚拟机时，副本虚拟机将连接到 VMNetwork1-Shanghai。
- 当为 VMNetwork2-Beijing 或 VMNetwork2-Shanghai 创建副本虚拟机时，副本虚拟机将不会连接到任何网络。

这是本示例组织中 VMM 云的设置方式，以及逻辑网络与云关联的方式。

### 云保护设置

**受保护的云** | **提供保护的云** | **逻辑网络（北京）**  
---|---|---
GoldCloud1 | GoldCloud2 |
SilverCloud1| SilverCloud2 |
GoldCloud2 | <p>NA</p><p></p> | <p>LogicalNetwork1-Beijing</p><p>LogicalNetwork1-Shanghai</p>
SilverCloud2 | <p>NA</p><p></p> | <p>LogicalNetwork1-Beijing</p><p>LogicalNetwork1-Shanghai</p>

### 逻辑和 VM 网络设置

**位置** | **逻辑网络** | **关联的 VM 网络**
---|---|---
北京 | LogicalNetwork1-Beijing | VMNetwork1-Beijing
上海 | LogicalNetwork1-Shanghai | VMNetwork1-Shanghai
 | LogicalNetwork2Shanghai | VMNetwork2-Shanghai

### 目标网络

根据这些设置，当你选择目标 VM 网络时，下表显示将可用的选项。

**选择** | **受保护的云** | **提供保护的云** | **可用目标网络**
---|---|---|---
VMNetwork1-Shanghai | SilverCloud1 | SilverCloud2 | 可用
 | GoldCloud1 | GoldCloud2 | 可用
VMNetwork2-Shanghai | SilverCloud1 | SilverCloud2 | 不可用
 | GoldCloud1 | GoldCloud2 | 可用



## 多个子网

如果目标网络具有多个子网，并且其中一个子网与源虚拟机所在的子网同名，则在故障转移后副本虚拟机将连接到该目标子网。如果没有具有匹配名称的目标子网，则虚拟机将连接到网络中的第一个子网。


### 故障回复

若要查看在故障恢复情况下发生的情况（反向复制），我们假设 VMNetwork1-Beijing 已映射到 VMNetwork1-Shanghai，设置如下。


**虚拟机** | **连接到 VM 网络**
---|---
VM1 | VMNetwork1-Network
VM2（VM1 的副本） | VMNetwork1-Shanghai

在采用这些设置的条件下，我们看看在一些可能的方案中会发生什么情况。

**方案** | **结果**
---|---
在故障转移后，VM-2 的网络属性没有发生任何变化。 | VM-1 保持连接到源网络。
故障转移且断开连接后，VM-2 的网络属性已更改 | VM-1 已断开连接
VM-2 的网络属性在故障转移后发生更改并连接到 VMNetwork2-Shanghai | 如果未映射 VMNetwork2-Shanghai，则 VM-1 将断开连接
VMNetwork1-Shanghai 的网络映射已更改 | VM-1 将连接到现映射到 VMNetwork1-Shanghai 的网络


## 后续步骤

现在，你已经对网络映射有了更好的了解，因此可以[开始 Site Recovery 部署](/documentation/articles/site-recovery-best-practices/)了。

<!---HONumber=Mooncake_0328_2016-->