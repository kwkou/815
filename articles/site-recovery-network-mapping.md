<properties
	pageTitle="Site Recovery 网络映射"
	description="Azure Site Recovery 可以协调位于本地的虚拟机和物理服务器到 Azure 或辅助本地站点的复制、故障转移和恢复。"
	services="site-recovery"
	documentationCenter=""
	authors="rayne-wiselman"
	manager="jwhit"
	editor=""/>

<tags
	ms.service="site-recovery"
	ms.date="05/08/2015"
	wacn.date="10/03/2015"/>


# Site Recovery 网络映射


Azure Site Recovery 有助于业务连续性和灾难恢复 (BCDR) 策略，因为它可以安排复制、故障转移和恢复虚拟机和物理服务器。在 [Site Recovery 概述](/documentation/articles/hyper-v-recovery-manager-overview)中了解可能的部署方案。


## 关于本文

网络映射是部署 VMM 和 Site Recovery 时的要素。网络映射可将复制的虚拟机放在最佳的目标 Hyper-V 主机服务器上，并确保复制的虚拟机在故障转移后可连接到适当的网络。本文介绍网络映射，并提供几个示例来帮助你了解网络映射的工作原理。


请在 [Azure 恢复服务论坛](https://social.msdn.microsoft.com/Forums/zh-CN/home?forum=hypervrecovmgr)上发布你的任何问题。

## 概述

设置网络映射的方式取决于 Site Recovery 的部署方案。



- **本地到本地 VMM 服务器** - 网络映射将在源 VMM 服务器上的 VM 网络与目标 VMM 服务器上的 VM 网络之间进行映射，以实现以下功能：

	- **在故障转移后连接虚拟机** - 确保在故障转移后，虚拟机会连接到适当的网络。副本虚拟机将连接到已映射到源网络的目标网络。
	- **将副本虚拟机放到主机服务器上** - 以最佳方式在 Hyper-V 主机服务器上放置副本虚拟机。副本虚拟机将放置在可访问映射 VM 网络的主机上。
	- **无网络映射** - 如果未配置网络映射，则复制的虚拟机在故障转移后将不会连接到任何 VM 网络。

- **本地 VMM 服务器到 Azure** - 网络映射将在源 VMM 服务器上的 VM 网络与目标 Azure 网络之间进行映射，以实现以下功能：
	- **在故障转移后连接虚拟机** - 在同一网络上进行故障转移的所有计算机都可以彼此连接到对方，不管它们位于哪个恢复计划中。
	- **网关** - 如果在目标 Azure 网络上设置了网络网关，则虚拟机可以连接到其他本地虚拟机。
	- **无网络映射** - 如果没有配置网络映射，则只有在同一恢复计划中进行故障转移的虚拟机能够在故障转移到 Azure 后彼此连接到对方。

## VM 网络

VMM 逻辑网络提供物理网络基础结构的抽象视图。VM 网络提供了一个网络接口，使虚拟机可以连接到逻辑网络。一个逻辑网络至少需要一个 VM 网络。将虚拟机放入云中以提供保护时，必须将该虚拟机连接到 VM 网络，此网络链接到与该云关联的逻辑网络。了解详细信息：

- [逻辑网络（第 1 部分）](http://blogs.technet.com/b/scvmm/archive/2013/02/14/networking-in-vmm-2012-sp1-logical-networks-part-i.aspx)
- [VMM 2012 SP1 中的虚拟网络](http://blogs.technet.com/b/scvmm/archive/2013/01/08/virtual-networking-in-vmm-2012-sp1.aspx)

## 示例

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

对网络映射有了更好的理解后，请开始阅读[最佳实践](/documentation/articles/site-recovery-best-practices)以做好部署准备。

<!---HONumber=71-->