
<properties
    pageTitle="Azure Site Recovery：复制单个 VMM 服务器上的 Hyper-V 虚拟机 | Azure"
    description="本文介绍如何只有单个 VMM 服务器时如何复制 Hyper-V 虚拟机。"
    services="site-recovery"
    documentationcenter=""
    author="rayne-wiselman"
    manager="jwhit"
    editor="" />  

<tags
    ms.assetid="1c0c4b99-2a6a-4fa2-89fa-d19c39869be6"
    ms.service="site-recovery"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="backup-recovery"
    ms.date="11/14/2016"
    wacn.date="12/12/2016"
    ms.author="raynew" />

# 复制单个 VMM 服务器上的 Hyper-V 虚拟机

阅读本文了解当部署中只有单个 VMM 服务器时，如何复制 VMM 云中 Hyper-V 主机服务器上的 Hyper-V 虚拟机。

如果在阅读本文后有任何问题，请在 [Azure 恢复服务论坛](https://social.msdn.microsoft.com/Forums/zh-cn/home?forum=hypervrecovmgr)上发布你的问题。


如果想要复制位于 VMM 中的 Hyper-V 主机上的 Hyper-V VM，并只有一个单一 VMM 服务器，则可以[复制到 Azure](/documentation/articles/site-recovery-vmm-to-azure/) 或在单个 VMM 服务器上的云之间复制。

建议复制到 Azure，因为在云之间复制时，故障转移和恢复并不是无缝的，并且需要大量的手动操作。如果确实想仅使用 VMM 服务器复制，则可以执行以下操作：

* 通过单个独立的 VMM 服务器复制
* 使用在已扩展的 Windows 群集中部署的单个 VMM 服务器复制

## 通过单个独立的 VMM 服务器在多个站点之间复制

![独立虚拟 VMM 服务器](./media/site-recovery-single-vmm/single-vmm-standalone.png)


在此方案中，需要在主站点中部署单个 VMM 服务器作为虚拟机，然后使用 Site Recovery 和 Hyper-V 副本将此 VM 复制到辅助站点。

1. 在 Hyper-V VM 上设置 VMM。为节约时间，建议将 VMM 所使用的 SQL Server 实例安装到同一 VM 上。如果希望使用 SQL Server 的远程实例，而此时发生了中断，则需先恢复该实例，然后再恢复 VMM。
2. 确保该 VMM 服务器至少配置了两个云。一个云包含要复制的 VM，另一个云充当辅助位置。包含要保护的 VM 的云应该有：

	- 一个或多个 VMM 主机组，其中每个主机组包含一个或多个 Hyper-V 主机服务器。
	- 每个 Hyper-V 主机服务器上至少有一个 Hyper-V 虚拟机。

3. 创建一个恢复服务保管库，生成和下载保管库密钥，然后注册保管库中的 VMM 服务器。注册期间，在 VMM 服务器上安装 Azure Site Recovery 提供程序。
4. 在 VMM VM 上设置一个或多个云，然后将 Hyper-V 主机添加到这些云中。
5. 配置云的保护设置。将这单个 VMM 服务器的名称指定为源和目标位置。若要配置网络映射，则需将一个云（内含要保护的 VM ）的 VM 网络映射到复制云的 VM 网络。
6. 可以通过网络启用 VM 的初始复制，因为两个云位于同一台服务器上。
7. 在 Hyper-V 管理器控制台中，在包含 VMM VM 的 Hyper-V 主机上启用 Hyper-V 副本，在 VM 上启用复制。请确保不要将 VMM VM 添加到任何受 Site Recovery 保护的云。这可确保 Site Recovery 不会重写 Hyper-V 副本设置。
8. 如果需要创建恢复计划，则请为源和目标指定同一 VMM 服务器。

按照[本文](/documentation/articles/site-recovery-vmm-to-vmm/)中的说明创建保管库、注册服务器并设置保护。

### 中断时的操作
如果发生完全中断，且需要从辅助站点进行操作，请执行以下操作：

1.	在辅助站点中的 Hyper-V 管理器控制台中，运行未计划的故障转移可将 VMM VM 从主站点故障转移到辅助站点。
2.	验证 VMM VM 已启动且正在辅助站点中运行。
3.	在恢复服务保管库中，运行未计划的故障转移以将工作负荷 VM 从主云故障转移到辅助云。若要完成未计划的 VM 故障转移，请提交故障转移，或根据需要选择不同的恢复点。
4.	未计划的故障转移完成后，用户可以访问辅助站点中的工作负荷资源。

主站点再次正常运行时，请执行以下操作：

1.	在 Hyper-V 管理器控制台中，启用 VMM VM 的反向复制，开始将其从辅助站点复制到主站点。
2.	在 Hyper-V 管理器控制台中，运行计划的故障转移将 VMM VM 故障回复到主站点。提交故障转移即可完成。然后，启用反向复制开始将 VMM 从主站点复制到辅助站点。
3.	在恢复服务保管库中，启用工作负荷 VM 的反向复制，开始将它们从辅助站点复制到主站点。
4.	在恢复服务保管库中，运行计划的故障转移将工作负荷 VM 故障回复到主站点。提交故障转移即可完成。然后，启用反向复制开始将工作负荷 VMM 从主站点复制到辅助站点。

## 在外延式群集中，通过单个独立的 VMM 服务器在多个站点之间复制
![群集虚拟 VMM 服务器](./media/site-recovery-single-vmm/single-vmm-cluster.png)

不必将单独的 VMM 服务器部署为将内容复制到辅助站点的 VM，而只需将其部署为 Windows 故障转移群集中的 VM 来提高 VMM 的可用性。这可以为工作负荷提供弹性并防止硬件故障。若要通过站点恢复进行部署，应将 VMM VM 部署在一个外延式群集中，该群集在地理位置上跨多个单独的站点。为此，请按以下步骤操作：

1. 将 VMM 安装在 Windows 故障转移群集的虚拟机上，在安装过程中请选择合适的选项，确保该服务器在运行的时候可用性高。
2. 应该使用 SQL Server AlwaysOn 可用性组来复制 VMM 所用的 SQL Server 实例，使得辅助站点中存在数据库的副本。
3. 按照[本文](/documentation/articles/site-recovery-vmm-to-vmm/)中的说明创建保管库、注册服务器并设置保护。需要在恢复服务保管库中注册群集中的每个 VMM 服务器。若要执行此操作，请在活动节点上安装提供程序并注册 VMM 服务器。然后在其他节点上安装提供程序。

### 中断时的操作
发生中断时，VMM 服务器及其相应的 SQL Server 数据库将故障转移，并可从辅助站点访问。

## 后续步骤

[详细了解](/documentation/articles/site-recovery-vmm-to-vmm/)有关 VMM 到 VMM 复制的站点恢复部署信息。

<!---HONumber=Mooncake_1205_2016-->