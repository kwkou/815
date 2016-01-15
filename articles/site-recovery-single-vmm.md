
<properties
	pageTitle="Azure Site Recovery：复制 Hyper-V 虚拟机（单个 VMM 服务器）"
	description="Azure Site Recovery 可以协调位于本地 VMM 云中的虚拟机到 Azure 或辅助 VMM 云的复制、故障转移和恢复。"
	services="site-recovery"
	documentationCenter=""
	authors="rayne-wiselman"
	manager="jwhit"
	editor=""/>

<tags
	ms.service="site-recovery"
	ms.date="12/01/2015"
	wacn.date="01/14/2016"/>

#  Azure Site Recovery：复制 Hyper-V 虚拟机（单个 VMM 服务器）

Azure Site Recovery 服务有助于构建稳健的业务连续性和灾难恢复 (BCDR) 解决方案，可以协调和自动完成从本地物理服务器和虚拟机到 Azure 或辅助本地数据中心的复制。本文介绍当部署中只有一个 VMM 服务器时，如果通过部署 Site Recovery 来保护 VMM 云中的 Hyper-V 虚拟机。如果在阅读本文后有任何问题，请在 [Azure 恢复服务论坛](https://social.msdn.microsoft.com/Forums/zh-cn/home?forum=hypervrecovmgr)上发布你的问题。

## 概述

你可以通过多种方式来复制 Hyper-V VM：

- 将不在 VMM 云中的 Hyper-V 主机上的 Hyper-V VM 复制到 Azure
- 将 VMM 云中的 Hyper-V 主机上的 Hyper-V VM 复制到 Azure
- 将 VMM 云中的 Hyper-V 主机上的 Hyper-V VM 复制到 Azure

不过，如果你想要使用 VMM，但基础结构中只有一个 VMM 服务器，则该怎么办？

这种情况下，你有多种选择：

- 将 VMM 云中的 Hyper-V VM 复制到 Azure。[详细了解](/documentation/articles/site-recovery-vmm-to-azure)此方案。
- 在单个 VMM 服务器上的云之间复制

我们建议你采用第一种选择，因为在第二种选择中，故障转移和恢复不是无缝的，需要执行许多手动步骤。


### 通过单个独立的 VMM 服务器在多个站点之间复制

在此方案中，你需要在主站点中部署一个独立的 VMM 服务器作为虚拟机，然后使用 Site Recovery 和 Hyper-V 副本将此虚拟机复制到辅助站点。为此，请按以下步骤操作：

1. 在 Hyper-V VM 上设置 VMM。考虑将 VMM 所使用的 SQL Server 实例归置到同一 VM 上。这可以节省时间，因为只需实例化一个 VM。如果你希望使用远程实例，而此时发生了中断，则需先恢复该实例，然后再恢复 VMM。
2. 确保该 VMM 服务器至少配置了两个云。一个云将包含你要复制的 VM，另一个云将充当辅助位置。你要保护的 VM 所在的云应该包含一个或多个 VMM 主机组，每个主机组中包含一个或多个 Hyper-V 主机服务器，每个主机服务器中至少包含一个 Hyper-V 虚拟机。
2. 创建一个 Site Recovery 保管库，生成和下载保管库密钥，然后注册保管库中的 VMM 服务器。
2. 在 VMM VM 上设置一个或多个云，然后将包含你要保护的 VM 的 Hyper-V 主机添加到这些云中。
3. 在 Azure Site Recovery 中配置云的保护设置。在“源和目标位置”中，指定单个 VMM 服务器的名称。如果配置网络映射，则需将一个云（内含你所要保护的 VM ）的 VM 网络映射到另一个云（你需要向其复制内容）的 VM 网络。
4. 使用“通过网络”作为复制方法以启用你所要保护的 VM 的复制功能，因为两个云都位于同一服务器上。
4. 在 Hyper-V 管理器控制台中，在包含 VMM VM 的 Hyper-V 主机上启用 Hyper-V 副本，在 VM 上启用复制。请确保不将 VMM 虚拟机添加到受 Site Recovery 保护的云中，使得 Hyper-V 副本设置不会被 Site Recovery 重写。
5. 如果需要创建恢复计划，则请为源和目标指定同一 VMM 服务器。 

发生中断时，请恢复 Hyper-V VM 上的工作负荷，如下所示：

1. 使用 Hyper-V 管理器通过计划的故障转移将副本 VMM VM 手动故障转移到辅助站点。
2. 恢复 VMM VM 以后，你可以从辅助站点登录到 Hyper-V 恢复管理器，然后通过非计划的故障转移将 VM 从主站点故障转移到辅助站点。
3. 完成非计划的故障转移之后，用户可以在主站点上访问所有资源。

注意，在能够进行工作负荷的故障转移之前，需要手动进行从 VMM 虚拟机到辅助站点的故障转移。

![独立虚拟 VMM 服务器](./media/site-recovery-single-vmm/single-vmm-standalone.png)

### 在外延式群集中，通过单个独立的 VMM 服务器在多个站点之间复制

你不必将单独的 VMM 服务器部署为将内容复制到辅助站点的虚拟机，而只需将其部署为 Windows 故障转移群集中的 VM 来提高 VMM 的可用性，从而提高工作负荷的弹性并在硬件故障时加强保护。若要通过 Windows 进行部署，则应将 VMM 部署在一个外延式群集中，该群集在地理位置上跨多个单独的站点。为此，请按以下步骤操作：

1. 将 VMM 安装在 Windows 故障转移群集的虚拟机上，在安装过程中请选择合适的选项，确保该服务器在运行的时候可用性高。
2. 应该使用 SQL Server AlwaysOn 可用性组来复制 VMM 所用的 SQL Server 实例，使得辅助站点中存在数据库的副本。 

发生中断时，VMM 服务器及其相应的 SQL Server 数据库将故障转移，并可从辅助站点访问。

![群集虚拟 VMM 服务器](./media/site-recovery-single-vmm/single-vmm-cluster.png)




 

<!---HONumber=Mooncake_0104_2016-->