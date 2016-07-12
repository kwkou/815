
<properties
	pageTitle="Azure Site Recovery：复制单个 VMM 服务器上的 Hyper-V 虚拟机 | Azure"
	description="本文介绍如何只有单个 VMM 服务器时如何复制 Hyper-V 虚拟机。"
	services="site-recovery"
	documentationCenter=""
	authors="rayne-wiselman"
	manager="jwhit"
	editor=""/>

<tags
	ms.service="site-recovery"
	ms.date="03/30/2016"
	wacn.date="05/16/2016"/>

#  复制单个 VMM 服务器上的 Hyper-V 虚拟机

阅读本文了解当部署中只有单个 VMM 服务器时，如何复制位于 VMM 云中 Hyper-V 主机服务器上的 Hyper-V 虚拟机。

如果在阅读本文后有任何问题，请在本文底部或 [Azure 恢复服务论坛](https://social.msdn.microsoft.com/Forums/zh-cn/home?forum=hypervrecovmgr)上发布你的问题。

## 概述

可通过多种方式复制位于 VMM 云中 Hyper-V 主机上的 Hyper-V VM：

- 复制到 Azure。 
- 复制到辅助 VMM 站点

如果你想要复制到辅助 VMM 位置，但部署中只有单个 VMM 服务器时该怎么办？

这种情况下，你有多种选择：

- 将 VMM 云中的 Hyper-V VM 复制到 Azure。[详细了解](/documentation/articles/site-recovery-vmm-to-azure/)此方案。
- 在单个 VMM 服务器上的云之间复制。

我们建议你采用第一种选择，因为在第二种选择中，故障转移和恢复不是无缝的，需要执行许多手动步骤。如果要在站点之间复制而不是复制到 Azure，你可以使用多个选项。


## 通过单个独立的 VMM 服务器在多个站点之间复制

在此方案中，若要在站点之间复制，你需要在主站点中部署单个 VMM 服务器作为虚拟机，然后使用站点恢复和 Hyper-V 副本将此 VM 复制到辅助站点。为此，请按以下步骤操作：

1. 在 Hyper-V VM 上设置 VMM。为此，我们建议你同时考虑将 VMM 所使用的 SQL Server 实例归置到同一 VM 上。这可以节省时间，因为只需实例化一个 VM。如果你希望使用 SQL Server 的远程实例，而此时发生了中断，则需先恢复该实例，然后再恢复 VMM。
2. 确保该 VMM 服务器至少配置了两个云。一个云将包含你要复制的 VM，另一个云将充当辅助位置。包含要保护的 VM 的云应该有：

	- 一个或多个 VMM 主机组，其中每个主机组包含一个或多个 Hyper-V 主机服务器。
	- 每个 Hyper-V 主机服务器上至少有一个 Hyper-V 虚拟机。
3. 创建一个 Site Recovery 保管库，生成和下载保管库密钥，然后注册保管库中的 VMM 服务器。在注册期间，将在 VMM 服务器上安装 Azure Site Recovery 提供程序。
4. 在 VMM VM 上设置一个或多个云，然后将包含你要保护的 VM 的 Hyper-V 主机添加到这些云中。
3. 在 Azure Site Recovery 中配置云的保护设置。在“源和目标位置”中，指定单个 VMM 服务器的名称。如果配置网络映射，则需将一个云（内含你所要保护的 VM ）的 VM 网络映射到另一个云（你需要向其复制内容）的 VM 网络。
4. 使用“通过网络”作为复制方法以启用你所要保护的 VM 的复制功能，因为两个云都位于同一服务器上。
4. 在 Hyper-V 管理器控制台中，在包含 VMM VM 的 Hyper-V 主机上启用 Hyper-V 副本，在 VM 上启用复制。请确保不将 VMM 虚拟机添加到受 Site Recovery 保护的云中，使得 Hyper-V 副本设置不会被 Site Recovery 重写。
5. 如果需要创建恢复计划，则请为源和目标指定同一 VMM 服务器。

按照[此文](/documentation/articles/site-recovery-vmm-to-vmm/)中的说明创建保管库、获取密钥、注册服务器并设置保护。

### 中断后

发生中断时，请恢复 Hyper-V VM 上的工作负荷，如下所示：

1. 使用 Hyper-V 管理器通过计划的故障转移将 VMM VM 手动故障转移到辅助站点。 
2. 恢复 VMM VM 以后，你可以从辅助站点登录到 Hyper-V 恢复管理器，然后运行非计划的故障转移将 VM 从辅助站点故障转移到主站点。注意，在能够进行工作负荷 VM 的故障转移之前，必须手动进行从 VMM VM 到辅助站点的故障转移。
3. 完成非计划的故障转移之后，可再次通过主站点访问所有资源。


![独立虚拟 VMM 服务器](./media/site-recovery-single-vmm/single-vmm-standalone.png)

## 在外延式群集中，通过单个独立的 VMM 服务器在多个站点之间复制

你不必将单独的 VMM 服务器部署为将内容复制到辅助站点的虚拟机，而只需将其部署为 Windows 故障转移群集中的 VM 来提高 VMM 的可用性。这可以为工作负荷提供弹性并防止硬件故障。若要通过站点恢复进行部署，应将 VMM VM 部署在一个外延式群集中，该群集在地理位置上跨多个单独的站点。为此，请按以下步骤操作：

1. 将 VMM 安装在 Windows 故障转移群集的虚拟机上，在安装过程中请选择合适的选项，确保该服务器在运行的时候可用性高。
2. 应该使用 SQL Server AlwaysOn 可用性组来复制 VMM 所用的 SQL Server 实例，使得辅助站点中存在数据库的副本。

请注意，设置站点恢复时，必须将群集中的每个 VMM 服务器注册到站点恢复保管库。需要将提供程序安装在活动节点上，并完成安装以便在保管库中注册该 VMM 服务器。然后在其他节点上安装提供程序。
 
### 中断后 

发生中断时，VMM 服务器及其相应的 SQL Server 数据库将故障转移，并可从辅助站点访问。

![群集虚拟 VMM 服务器](./media/site-recovery-single-vmm/single-vmm-cluster.png)

## 后续步骤

[详细了解](/documentation/articles/site-recovery-vmm-to-vmm/)有关 VMM 到 VMM 复制的站点恢复部署信息。




 

<!---HONumber=Mooncake_0509_2016-->