<properties
	pageTitle="Active Directory 的 ASR 指南 | Windows Azure" 
	description="本文详细说明了如何使用 Azure Site Recovery 为 AD 创建灾难恢复解决方案，以及如何通过一键式恢复计划执行计划内/计划外/测试性故障转移，同时还详细说明了支持的配置和先决条件。" 
	services="site-recovery" 
	documentationCenter="" 
	authors="prateek9us" 
	manager="abhiag" 
	editor=""/>

<tags 
	ms.service="site-recovery"
	ms.date="10/12/2015" 
	wacn.date="12/15/2015"/>

#使用 ASR 的自动化 DR 解决方案，适用于 Active Directory 和 DNS


所有企业应用程序，例如 SharePoint, Dynamics AX 和 SAP，都依赖于 AD 和 DNS 基础结构才能正常工作。在为任何此类应用程序创建灾难恢复 (DR) 解决方案时，如果出现中断，在恢复应用程序的其他组件之前保护和恢复 AD 非常重要。

Azure Site Recovery 是一项基于 Azure 的服务，可提供灾难恢复功能，对虚拟机的复制、故障转移和恢复进行协调。Azure Site Recovery 支持一系列复制技术，可以前后一致地对虚拟机和应用程序进行复制、保护和无缝故障转移，转移目标是私有云/公用云或托管商的云。

使用 Azure Site Recovery，你可以为你的 AD 创建一个完整的自动化灾难恢复计划。在出现中断时，你可以在数秒内从任何位置启动故障转移，在数分钟内启动和运行 AD。如果你在主站点中有一个用于多个应用程序（例如 SharePoint 和 SAP）的 AD，并且你决定对整个站点进行故障转移，则可以首先使用 ASR 对 AD 进行故障转移，然后使用应用程序特定恢复计划对其他应用程序进行故障转移。

本文详细说明了如何为 AD 创建灾难恢复解决方案，以及如何通过一键式恢复计划执行计划内/计划外/测试性故障转移，同时还详细说明了支持的配置和先决条件。受众应熟悉 AD 和 Azure Site Recovery。可以采用两种建议的选项来保护 AD，具体取决于客户本地环境的复杂性。

####选项 1

如果应用程序的数目较小，且在环境中只有一个域控制器，而你需要将整个站点一起进行故障转移，则我们建议使用 ASR 复制将域控制器计算机复制到辅助站点（适用于站点到站点和站点到 Azure）。同一台复制的虚拟机也可以用于测试性故障转移。

####方法 2
如果应用程序的数目较大，而环境中有不止一个域控制器，或者你计划一次对数个应用程序进行故障转移，则我们建议，除了在域控制器虚拟机（用于测试性故障转移）上启用 ASR 复制，还可以在 DR 站点上（辅助性本地站点上或 Azure 中）设置另外一个域控制器。

>[AZURE.NOTE]即使你要选择的是选项 2，但为了进行测试性故障转移，也仍然需要为域控制器设置 ASR 复制。如需更多详细信息，请参阅[测试性故障转移注意事项](#considerations-for-test-failover)部分。


以下部分说明了如何在 ASR 中对域控制器启用保护，以及如何在 Azure 中设置域控制器。


##先决条件

使用 Azure Site Recovery 时，若要针对 AD 实现灾难恢复，需要满足以下先决条件。

- 在本地部署 Active Directory 和 DNS 服务器
- 已在 Windows Azure 订阅中创建 Azure Site Recovery 服务保管库 
- 如果 Azure 是你的恢复站点，则可在 VM 上运行 Azure 虚拟机准备情况评估工具，确保这些 VM 兼容 Azure VM 和 Azure Site Recovery 服务


##使用 ASR 对域控制器/DNS 启用保护


###保护 VM
在 ASR 中启用对域控制器/DNS VM 的保护。根据 VM 是部署在 Hyper-V 上还是 VMware 上，执行相关的 Azure Site Recovery 配置。建议的可用于配置且与崩溃一致的频率为 15 分钟。

###配置 VM 网络设置
- 对于域控制器/DNS 虚拟机，请对 ASR 中的网络设置进行配置，以便在故障转移后将 VM 连接到正确的网络。 
- 例如，将 Azure 用作 DR 站点时，你可以在“VMM 云”或“保护组”中选择 VM，以便将网络设置配置为以下快照所示。
- 
![VM 网络设置](./media/site-recovery-active-directory/VM-Network-Settings.png)

##使用 AD 复制为 AD 启用保护
###站点到站点方案

在辅助 (DR) 站点上创建域控制器，并在将服务器提升到域控制器角色的同时，为其指定在主站点上使用的同一个域的名称。你可以使用 Active Directory 站点和服务管理单元来配置要向其添加站点的站点链接对象上的设置。通过在站点链接上配置设置，你可以控制何时在两个或两个以上站点之间进行复制，以及复制的频率。有关更多详细信息，请参阅[计划站点之间的复制](https://technet.microsoft.com/zh-cn/library/cc731862.aspx)。

###站点到 Azure 方案
按以下说明操作，以便创建 [Azure 虚拟网络中的域控制器](/documentation/articles/virtual-networks-install-replica-active-directory-domain-controller)。在将服务器提升到域控制器角色的同时，为其指定在主站点上使用的同一个域的名称。

然后，你应该[重新配置虚拟网络的 DNS 服务器](/documentation/articles/virtual-networks-install-replica-active-directory-domain-controller#reconfigure-dns-server-for-the-virtual-network)，以便在 Azure 中使用 DNS 服务器
  
![Azure 网络](./media/site-recovery-active-directory/azure-network.png)

##<a id="considerations-for-test-failover"></a>测试性故障转移注意事项
测试性故障转移是在独立于生产网络的网络中完成的，因此对生产工作负荷没有影响。大多数应用程序还需要存在域控制器和 DNS 服务器才能运行。因此，在对应用程序进行故障转移之前，还需在用于测试性故障转移的独立网络中创建域控制器。这样做最简单的方法是，首先使用 ASR 在域控制器/DNS 虚拟机上启用保护，然后在触发对应用程序的恢复计划进行的测试性故障转移之前，触发对域控制器虚拟机进行的测试性故障转移。以下是执行测试性故障转移的分步指南：

1. 在域控制器/DNS 虚拟机上启用保护，就像对任何其他虚拟机启用保护一样。
2. 创建独立的网络。默认情况下，在 Azure 中创建的任何虚拟网络都是独立于其他网络的。建议将此网络的 IP 范围设置为与生产网络相同。不要在此网络上启用站点到站点连接。
3. 提供在上一步创建的网络中的 DNS IP，作为你希望 DNS VM 能够获取的 IP。如果使用 Azure 作为 DR 站点，则可为 VM 提供该 IP，用在 VM 属性的“目标 IP”中，以便进行故障转移。如果你的 DR 站点是本地的，而且你使用的是 DHCP，则请遵循此处提供的说明，以便[针对测试性故障转移设置 DNS 和 DHCP](/documentation/articles/site-recovery-failover#prepare-dhcp) 

>[AZURE.NOTE]在进行测试性故障转移时，提供给虚拟机的 IP 与该虚拟机在进行计划内或计划外故障转移时会获得的 IP 相同，前提是该 IP 可用于测试性故障转移网络。如果这个相同的 IP 在测试性故障转移网络中不可用，则虚拟机会获取可在测试性故障转移网络中使用的其他 IP。

4. 转到域控制器虚拟机，然后在独立网络中对其进行测试性故障转移。 
5. 对应用程序的恢复计划进行测试性故障转移。
6. 测试完成后，在 ASR 的“作业”选项卡中将针对域控制器虚拟机作业和恢复计划进行的故障转移标记为“完成”。 

###DNS 与域控制器不在同一虚拟机上： 
如果 DNS 与域控制器不在同一虚拟机上，则需创建一个可以进行测试性故障转移的 DNS。如果它们位于同一 VM 上，则可跳过此部分。你可以使用全新的 DNS 服务器并创建所有需要的区域。例如，如果你的 Active Directory 域是 contoso.com，则可以使用名称 contoso.com 创建区域。必须在 DNS 中更新与 Active Directory 对应的条目。请按如下所述执行此操作：

- 确保在恢复计划中的任何其他虚拟机到位之前，以下设置已准备就绪：
	- 区域必须以林根名称命名。
	- 区域必须备份文件。
	- 必须启用区域以进行安全和非安全更新。
	- 域控制器虚拟机的解析程序应指向 DNS 虚拟机的 IP 地址。
- 在域控制器虚拟机目录上运行以下命令：nltest /dsregdns

- **添加区域** — 使用以下脚本示例在 DNS 服务器上添加一个区域，允许非安全更新，并向 DNS 添加它自己的一个条目：

	    dnscmd /zoneadd contoso.com  /Primary 
	    dnscmd /recordadd contoso.com  contoso.com. SOA %computername%.contoso.com. hostmaster. 1 15 10 1 1 
	    dnscmd /recordadd contoso.com %computername%  A <IP_OF_DNS_VM> 
	    dnscmd /config contoso.com /allowupdate 1


##摘要
使用 Azure Site Recovery，你可以为你的 AD 创建一个完整的自动化灾难恢复计划。在出现中断时，你可以在数秒内从任何位置启动故障转移，在数分钟内启动和运行 AD。如果你在主站点中有一个用于多个应用程序（例如 SharePoint 和 SAP）的 AD，并且你决定对整个站点进行故障转移，则可以首先使用 ASR 对 AD 进行故障转移，然后使用应用程序特定恢复计划对其他应用程序进行故障转移。


请参阅 [Site Recovery 工作负荷指南](/documentation/articles/site-recovery-workload)，了解如何使用 ASR 保护各种工作负荷的详细信息。

<!---HONumber=79-->