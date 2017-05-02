<properties
    pageTitle="设计用于灾难恢复的网络基础结构 | Azure"
    description="本文讨论 Azure Site Recovery 的网络设计注意事项"
    services="site-recovery"
    documentationcenter=""
    author="prateek9us"
    manager="jwhit"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="ce787731-d930-4f00-a309-e2fbc2e1f53b"
    ms.service="site-recovery"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="storage-backup-recovery"
    ms.date="03/27/2017"
    wacn.date="05/02/2017"
    ms.author="pratshar"
    ms.sourcegitcommit="78da854d58905bc82228bcbff1de0fcfbc12d5ac"
    ms.openlocfilehash="dd94e4ee0d1e88087e2ec2985ad9c1f12d118ade"
    ms.lasthandoff="04/22/2017" />

# <a name="designing-your-network-for-disaster-recovery"></a>设计用于灾难恢复的网络
本文面向 IT 专业人员，他们负责构建、实施和支持业务连续性和灾难恢复 (BCDR) 基础结构，而且希望利用 Azure Site Recovery (ASR) 来支持并增强其 BCDR 服务。 本白皮书将讨论 System Center Virtual Machine Manager 服务器部署的实际注意事项、外延式子网与子网故障转移的优缺点比较，以及如何构建 Azure 中虚拟站点的灾难恢复。

## <a name="overview"></a>概述
[Azure Site Recovery (ASR)](/home/features/site-recovery/) 是一项 Azure 服务，可协调虚拟化应用程序的保护和恢复，以达到业务连续性和灾难恢复 (BCDR) 的目的。 本文档旨在引导读者完成网络设计的过程，着重于使用 Site Recovery 复制虚拟机 (VM) 时如何构建灾难恢复站点上的 IP 范围和子网。

此外，本文还将演示 Site Recovery 如何构建及实施多站点的虚拟数据中心，以在测试或发生灾难时支持 BCDR 服务。

在每个人都期待 24/7 全天候连接不中断的世界，保持基础结构和应用程序启动并运行的重要性更甚于以往。 业务连续性和灾难恢复 (BCDR) 的目的是要还原失败的组件，让组织可以快速恢复正常运营。 制定灾难恢复策略来应对不太可能发生的严重破坏事件非常具有挑战性。 这是因为对未来进行预测本来就有难度，尤其是它与不太可能发生的事件有关，并且提供适当措施来防止未知灾难的成本也很高。

BCDR 规划的关键在于必须为灾难恢复计划定义恢复时间目标 (RTO) 和恢复点目标 (RPO)。 当使用 Azure Site Recovery 的客户数据中心发生灾害时，客户可以在数据丢失最少（低 RPO）的情况下，迅速（低 RTO）让其位于辅助数据中心或 Azure 的复制虚拟机联机。 

ASR 让故障转移变为可能，第一步是将指定的虚拟机从主要数据中心复制到辅助数据中心或 Azure（视方案而定），然后定期刷新副本。 规划基础结构时，应将网络设计视为可能会使得无法达成公司 RTO 和 RPO 目标的潜在瓶颈。  

当管理员计划部署灾难恢复解决方案时，脑海中浮现的一个重要问题是，如何在故障转移完成后使虚拟机可供访问。 ASR 允许管理员选择在故障转移之后虚拟机连接的网络。 如果主站点由 VMM 服务器管理，则可以使用网络映射来实现。 有关更多详细信息，请参阅[准备网络映射](/documentation/articles/site-recovery-vmm-to-vmm/#prepare-for-network-mapping)。

设计恢复站点的网络时，管理员有两种选择：

* 对恢复站点的网络使用不同的 IP 地址范围。 在这种情况下，虚拟机在故障转移之后会获取新的 IP 地址，管理员必须进行 DNS 更新。 在[此处](/documentation/articles/site-recovery-test-failover-vmm-to-vmm/#preparing-infrastructure-for-test-failover)了解详细信息
* 对恢复站点的网络使用相同的 IP 地址范围。 在某些情况下，即使在故障转移之后，管理员也想在主站点上保留他们的 IP 地址。 在正常情况下，管理员必须更新路由以指示 IP 地址的新位置。 但是，对于在主站点和恢复站点之间部署了延伸 VLAN 的情况，保留虚拟机的 IP 地址会变成一个不错的选择。 保留相同 IP 地址可省去故障转移后的所有网络相关步骤，从而简化了恢复过程。

当管理员计划部署灾难恢复解决方案时，脑海中浮现的一个重要问题是，如何在故障转移完成后使应用程序可供访问。 现代应用程序在一定程度上几乎都依赖网络，因此，以物理方式将服务从一个站点移动到另一个站点会带来网络挑战。 在灾难恢复解决方案中，解决这个问题有两种主要方法。 第一种方法是保持固定的 IP 地址。 尽管移动的服务和宿主服务器位于不同的物理位置，应用程序将 IP 地址配置带到新的位置。 第二种方法需要在转换到恢复站点的过程中完全更改 IP 地址。 每种方法都有多种不同的实施方式，下面进行了汇总。

设计恢复站点的网络时，管理员有两种选择：

## <a name="option-1-retain-ip-addresses"></a>选项 1：保留 IP 地址
从灾难恢复过程的角度来看，使用固定的 IP 地址似乎是最简单的实施方法，但它有许多潜在的挑战，实际上反而是最不常用的方法。 Azure Site Recovery 在所有方案中提供保留 IP 地址的能力。 决定要保留 IP 之前，应适当考虑它对故障转移功能施加的限制。 让我们看一下可帮助你决定是否保留 IP 地址的因素。 可以通过两种方式来实现：使用外延式子网，或执行完整的子网故障转移。

### <a name="stretched-subnet"></a>外延式子网
若采取此做法，可在主要位置和 DR 位置同时使用子网。 简单而言，这意味着可以将一个服务器及其 IP（第 3 层）配置移动到辅助站点，并且网络会自动将流量路由到新的位置。 从服务器的角度来看，这处理十分常见，但是也有若干挑战：

* 从第 2 层（数据链路层）的角度来看，它需要能够管理外延式 VLAN 的网络设备，不过这不成问题，因为现在可以方便地获得这种设备。 第二个且更加困难的问题在于通过延伸 VLAN，潜在容错域也同时延伸到两个站点，本质上变为一个单点故障。 虽然不太可能，但过去曾经发生过无法隔离的广播风暴。 关于这个问题我们得到过各种意见，看过许多成功的实施，也遇到过“我们永远不会实施这种技术”的人。
* 如果使用 Azure 作为 DR 站点，就不可能实施外延式子网。

### <a name="subnet-failover"></a>子网故障转移
可以实施子网故障转移，以获得上述外延式子网解决方案的优点，而无需真的跨多个站点延伸子网。 若采取此做法，任何给定的子网会出现在站点 1 或站点 2 中，但永远不会同时出现在这两个站点中。 为了在故障转移时保留 IP 地址空间，可以通过编程方式安排路由器基础结构将子网从一个站点移到另一个站点。 在故障转移方案中，子网随关联的受保护 VM 一起移动。 此方法的主要缺点在于，当出现故障时，你必须移动整个子网，这可能没问题，但是会影响故障转移粒度注意事项。

让我们看看虚构企业 Contoso 如何能够在将其 VM 复制到某个恢复位置的同时，对整个子网进行故障转移。 首先，我们看看 Contoso 如何能够在管理其子网的同时，在两个本地位置之间复制 VM，接着，我们将讨论在 [将 Azure 用作灾难恢复站点](#failover-to-azure)时，子网故障转移是如何工作的。

#### <a name="failover-to-a-secondary-on-premises-site"></a>故障转移到辅助本地站点
让我们看看这种情况：我们想保留每个 VM 的 IP，并一起故障转移整个子网。 主站点具有在子网 192.168.1.0/24 中运行的应用程序。 发生故障转移时，属于此子网的所有虚拟机将向恢复站点进行故障转移，并且保留它们的 IP 地址。 必须适当地修改路由，以反映所有属于子网 192.168.1.0/24 的虚拟机现在都已移至恢复站点的事实。

下图中，主站点与恢复站点、第三个站点与主站点以及第三个站点与恢复站点之间的路由都必须适当地修改。

下图显示故障转移之前的子网。 子网 192.168.0.1/24 在故障转移之前在主站点上处于活动状态，在故障转移之后在恢复站点上变为活动状态

![在故障转移之前](./media/site-recovery-network-design/network-design2.png)

在故障转移之前

下图显示故障转移之后的网络和子网。

![在故障转移之后](./media/site-recovery-network-design/network-design3.png)

在故障转移之后

如果你的辅助站点是本地站点，并且你使用 VMM 服务器管理它，那么，当对特定虚拟机启用保护时，ASR 会根据以下工作流分配网络资源：

* ASR 从每个 System Center VMM 实例的相应网络所定义的静态 IP 地址池中为虚拟机上的每个网络接口分配一个 IP 地址。
* 如果管理员为恢复站点上的网络定义的 IP 地址池，与主站点上网络的 IP 地址池相同，则 ASR 在分配副本虚拟机的 IP 地址时，会分配与主要虚拟机相同的 IP 地址。  IP 保留在 VMM 中，但不会设置为故障转移 IP。 故障转移 IP 会在故障转移之前设置。

![保留 IP 地址](./media/site-recovery-network-design/network-design4.png)


上图显示副本虚拟机的故障转移 TCP/IP 设置（在 Hyper-V 控制台上）。 系统会在虚拟机启动之前、故障转移之后填充这些设置

如果找不到相同的 IP，ASR 会分配已定义的 IP 地址池中一些其他可用的 IP 地址。

在为 VM 启用保护之后，可以使用以下脚本示例验证已经分配给虚拟机的 IP。 ASR 会将相同的 IP 设置为故障转移 IP，并在故障转移时分配到 VM：

        $vm = Get-SCVirtualMachine -Name <VM_NAME>
        $na = $vm[0].VirtualNetworkAdapters>
        $ip = Get-SCIPAddress -GrantToObjectID $na[0].id
        $ip.address  

> [AZURE.NOTE]
> 在虚拟机使用 DHCP 的方案中，IP 地址的管理完全在 ASR 控制范围之外。 管理员必须确保提供恢复站点上 IP 地址的 DHCP 服务器可以从与主站点相同的范围中提供地址。
>
>

#### <a name="failover-to-azure"></a>故障转移到 Azure
Azure Site Recovery (ASR) 能让 Azure 作为虚拟机的灾难恢复站点。 在此情况下，必须多处理一个限制。 

让我们看看这个案例，虚构公司 Woodgrove Bank 的本地基础结构承载了业务线应用程序，其移动应用程序则承载于 Azure 上。 Azure 中的 Woodgrove Bank VM 以及本地服务器之间的连接由站点到站点 (S2S) 虚拟专用网 (VPN) 提供。 S2S VPN 使得 Woodgrove Bank 在 Azure 中的虚拟网络可被视为 Woodgrove Bank 本地网络的扩展。 此通信由 Woodgrove Bank 边缘和 Azure 虚拟网络之间的 S2S VPN 实现。 现在，Woodgrove 想要使用 ASR 将其数据中心内运行的工作负荷复制到 Azure。 此做法符合 Woodgrove 的需求，既拥有经济实惠的 DR 选项，又能够将数据存储在公有云环境中。 Woodgrove 必须处理依赖于硬编码 IP 地址的应用程序和配置，因此他们需要在向 Azure 进行故障转移之后为他们的应用程序保留 IP 地址。

Woodgrove 决定将来自 IP 地址范围（172.16.1.0/24、172.16.2.0/24）的 IP 地址分配给在 Azure 中运行的资源。

为了让 Woodgrove 能够在将其虚拟机复制到 Azure 的同时保留 IP 地址，需要创建 Azure 虚拟网络。 此网络应该是本地网络的扩展，这样应用程序才可以从本地站点无缝地故障转移到 Azure。 Azure 允许将站点到站点以及点到站点 VPN 连接能力添加到在 Azure 中创建的虚拟网络。 设置站点到站点连接时，只有在 IP 地址范围与本地 IP 地址范围不同时，Azure 网络才允许将流量路由到本地位置（Azure 称其为本地网络），因为 Azure 不支持外延式子网。  这意味着如果在本地有一个子网 192.168.1.0/24，则不能在 Azure 网络中添加一个本地网络 192.168.1.0/24。 这是意料之中的，因为 Azure 不知道子网中没有活动的虚拟机，也不知道正在创建的子网仅用于灾难恢复。 为了能够将网络流量从一个 Azure 网络中正确路由出去，该网络中的子网和本地网络中的子网不能有冲突。

![在子网故障转移之前](./media/site-recovery-network-design/network-design7.png)

在故障转移之前

为了帮助 Woodgrove 满足他们的业务需求，我们必须实施下列工作流：

* 创建额外的网络，我们叫它恢复网络，故障转移的虚拟机将在这里创建。
* 为了确保 VM 的 IP 会在故障转移之后保留，请转到 VM 属性下的“配置”选项卡，指定 VM 在本地使用的相同 IP，然后单击“保存”。 当 VM 故障转移时，Azure Site Recovery 将为虚拟机分配所提供的 IP。

![网络属性](./media/site-recovery-network-design/network-design8.png)

一旦触发故障转移，并在恢复网络中以所需的 IP 地址创建虚拟机后，就可使用 [Vnet 到 Vnet 连接](/documentation/articles/virtual-networks-configure-vnet-to-vnet-connection/)建立与此网络的连接。 如果需要，此操作可以编写脚本。  如我们在关于子网故障转移的上一节所讨论的那样，如果故障转移到 Azure，路由也必须适当地修改，以反映 192.168.1.0/24 现在已移到 Azure。

![在子网故障转移之后](./media/site-recovery-network-design/network-design9.png)

在故障转移之后

如果你没有如上图所示的“Azure 网络”。 你可以在故障转移之后，在“主站点”与“恢复网络”之间创建站点到站点 VPN 连接。  

## <a name="option-2-changing-ip-addresses"></a>选项 2：更改 IP 地址
据我们所见，这种方法似乎最普遍。 其采取的做法是更改故障转移中每个 VM 的 IP 地址。 此方法的缺点是传入网络需要“学习”原本在 IPx 的应用程序现在在 IPy。 即使 IPx 和 IPy 是逻辑名称，通常也需要在整个网络中更改或刷新 DNS 条目，而且必须更新或刷新网络表中的缓存条目，因此，停机时间取决于 DNS 基础结构如何设置。 对于 Intranet 应用程序，可以使用低 TTL 值；对于基于 Internet 的应用程序，可以使用 [带有 ASR 的 Azure 流量管理器](https://azure.microsoft.com/zh-cn/blog/2015/03/03/reduce-rto-by-using-azure-traffic-manager-with-azure-site-recovery/) 来缓解这些问题

### <a name="changing-the-ip-addresses---illustration"></a>更改 IP 地址 — 插图
让我们看一下在主站点和恢复站点使用不同 IP 的方案。 在以下示例中，我们有第三个站点，可以从该站点访问主站点或恢复站点上承载的应用程序。

![不同的 IP — 在故障转移之前](./media/site-recovery-network-design/network-design10.png)


在上图中，某些应用程序托管于主站点上的子网 192.168.1.0/24 中，它们被配置为在故障转移之后来到恢复站点上的子网 172.16.1.0/24 中。 已经正确配置 VPN 连接/网络路由，使所有三个站点都能相互访问。

如下图所示，在对一个或多个应用程序进行故障转移之后，它们将在恢复子网中还原。 在此情况下，我们不受同时故障转移整个子网的限制。 不需要进行任何更改来重新配置 VPN 或网络路由。 故障转移和某些 DNS 更新会确保应用程序仍然可供访问。 如果 DNS 配置为允许动态更新，则虚拟机会在故障转移之后的启动时使用新的 IP 自行注册。

![不同的 IP — 在故障转移之后](./media/site-recovery-network-design/network-design11.png)


在故障转移之后，副本虚拟机的 IP 地址可能与主虚拟机的 IP 地址不同。 虚拟机在启动后将更新它们使用的 DNS 服务器。 DNS 条目通常必须改变或在整个网络中刷新，并且网络表中的缓存条目必须更新或刷新，因此在这些状态改变发生时面临停机并不是少见的事情。 可以通过以下方式来缓解该问题：

* 对于 Intranet 应用程序，使用低 TTL 值。
* 对于基于 Internet 的应用程序，使用带有 ASR 的 Azure 流量管理器。
* 在你的恢复计划中使用以下脚本来更新 DNS 服务器，以确保及时更新（如果配置了动态 DNS 注册，则不需要此脚本）

		string]$Zone,
		[string]$name,
		[string]$IP
		)
		$Record = Get-DnsServerResourceRecord -ZoneName $zone -Name $name
		$newrecord = $record.clone()
		$newrecord.RecordData[0].IPv4Address  =  $IP
		Set-DnsServerResourceRecord -zonename $zone -OldInputObject $record -NewInputObject $Newrecord

### <a name="changing-the-ip-addresses--dr-to-azure"></a>更改 IP 地址 — 灾难恢复到 Azure

[为作为灾难恢复站点的 Azure 设置网络基础结构](https://azure.microsoft.com/zh-cn/blog/2014/09/04/networking-infrastructure-setup-for-microsoft-azure-as-a-disaster-recovery-site/) 这篇博客文章解释了在不需要保留 IP 地址时如何设置所需的 Azure 网络基础结构。 文章一开始描述应用程序，接着探讨如何在本地及 Azure 设置网络，最后说明如何执行测试故障转移和计划的故障转移。

## <a name="next-steps"></a>后续步骤
[了解](/documentation/articles/site-recovery-vmm-to-vmm/#prepare-for-network-mapping)当 VMM 服务器用于管理主站点时，Site Recovery 如何映射源和目标网络。
<!--Update_Description:update three link references;add anchors to sub titles-->