<properties 
	pageTitle="站点恢复：常见问题解答 | Azure" 
	description="本文解答了有关 Azure Site Recovery 的常见问题。"
	services="site-recovery"
	documentationCenter=""
	authors="csilauraa"
	manager="jwhit"
	editor="tysonn"/>

<tags
	ms.service="site-recovery"
	ms.date="02/14/2016"
	wacn.date="04/05/2016"/>


# Azure Site Recovery：常见问题 (FAQ)

本文包含有关站点恢复的常见问题。

如果在阅读本文后还有问题，可以在本文底部或 [Azure 恢复服务论坛](https://social.msdn.microsoft.com/Forums/zh-cn/home?forum=hypervrecovmgr)上发布你的问题。


## 概述

Site Recovery 有助于业务连续性和灾难恢复 (BCDR) 策略，因为它可以帮助确保工作负荷和应用在发生中断时保持可用。Site Recovery 可安排和自动执行从本地虚拟机和物理服务器到 Azure 或辅助数据中心的复制。[了解详细信息](/documentation/articles/site-recovery-overview)。


### 站点恢复可以保护哪些计算机？

- **Hyper-V 虚拟机**：站点恢复可以保护 Hyper-V VM 上运行的任何工作负荷。
- **物理服务器**：站点恢复可以保护 Windows/Linux 物理服务器上运行的工作负荷。
- **VMware 虚拟机**：站点恢复可以保护 VMware VM 上运行的任何工作负荷。


### Hyper-V 有哪些要求？

Hyper-V 主机服务器的要求取决于部署方案。在以下路径下查看先决条件：

- [将 Hyper-V VM 复制到 Azure（不使用 VMM）](/documentation/articles/site-recovery-hyper-v-site-to-azure#before-you-start)
- [将 Hyper-V VM 复制到 Azure（使用 VMM）](/documentation/articles/site-recovery-vmm-to-azure#before-you-start)
- [将 Hyper-V VM 复制到辅助数据中心](/documentation/articles/site-recovery-vmm-to-vmm#before-you-start)

在 Hyper-V 主机服务器上运行的来宾的要求：

- 如果你要复制到辅助站点，请在 Hyper-V 主机服务器上运行的 [Hyper-V VM 的受支持来宾操作系统](https://technet.microsoft.com/zh-cn/library/mt126277.aspx)中阅读 VM 的受支持来宾操作系统的相关信息。
- 如果你要复制到 Azure，Site Recovery 支持 [Azure 支持的](https://technet.microsoft.com/zh-cn/library/cc794868%28v=ws.10%29.aspx)所有来宾操作系统。


### 当 Hyper-V 在客户端操作系统上运行时，我可以保护 VM 吗？

不可以。VM 必须位于运行受支持 Windows 服务器计算机的 Hyper-V 主机服务器上。如果需要保护客户端计算机，可以将其作为物理计算机复制到 [Azure](/documentation/articles/site-recovery-vmware-to-azure-classic) 或[辅助数据中心](/documentation/articles/site-recovery-vmware-to-vmware)。


### 我可以使用站点恢复来保护哪些工作负荷？

Site Recovery 可以复制受支持 VM 或物理服务器上运行的大多数工作负荷。Site Recovery 还为应用程序感知型复制提供支持，因此，应用可以恢复为智能状态。它除了与 Microsoft 应用程序（包括 SharePoint、Exchange、Dynamics、SQL Server 及 Active Directory）集成之外，还能与行业领先的供应商（包括 Oracle、SAP、IBM 及 Red Hat）紧密配合。[了解](/documentation/articles/site-recovery-workload)有关工作负荷保护的详细信息。


### 一定需要 System Center VMM 服务器才能保护 Hyper-V 虚拟机吗？ 

如果要复制到辅助数据中心，那么 Hyper-V VM 就必须位于 VMM 云中的 Hyper-V 主机服务器上。但是，如果想要复制到 Azure，那么可以复制 Hyper-V 主机服务器上的 VM（无论是否使用 VMM 云）。[了解详细信息](/documentation/articles/site-recovery-hyper-v-site-to-azure)。

### 如果我只有一个 VMM 服务器，可以部署站点恢复来配合 VMM 吗？ 

是的。你可以将云中 VMM 服务器上的 Hyper-V VM 复制到 Azure，或者在同一台服务器上的 VMM 云之间进行复制。对于本地到本地复制，我们建议在主站点与辅助站点中都部署一个 VMM 服务器。[了解详细信息](/documentation/articles/site-recovery-single-vmm)

### 我可以保护哪些物理服务器？

可以将运行 Windows 或 Linux 的物理服务器复制到 Azure 或辅助站点来进行保护。[了解](/documentation/articles/site-recovery-vmware-to-azure-classic#before-you-start-deployment)有关操作系统要求的信息。无论是将物理服务器复制到 Azure 还是辅助站点，都存在相同的限制。

请注意，如果本地服务器关闭，物理服务器将在 Azure 中以 VM 的形式运行。当前不支持故障回复到本地物理服务器。仅可以故障回复到 VMware 上运行的虚拟机。

### 我可以保护哪些 VMware VM？

若要保护 VMware VM，则需要 vSphere 虚拟机监控程序和运行 VMware 工具的虚拟机。我们还建议你使用 VMware vCenter 服务器托管虚拟机监控程序。[了解有关](/documentation/articles/site-recovery-vmware-to-azure-classic#before-you-start-deployment) VMware 服务器和 VM 的确切要求的详细信息。无论是将 VMware VM 复制到 Azure 还是辅助站点，都存在相同的限制。

### 将虚拟机复制到 Azure 需要满足任何先决条件吗？

要复制到 Azure 的虚拟机应符合 [Azure 要求](/documentation/articles/site-recovery-best-practices#azure-virtual-machine-requirements)。

### 我可以将 Hyper-V 第 2 代虚拟机复制到 Azure 吗？

是的。Site Recovery 在故障转移期间将 VM 从第 2 代转换成第 1 代。在故障回复时，VM 转换回第 2 代。[了解详细信息](https://azure.microsoft.com/blog/2015/04/28/disaster-recovery-to-azure-enhanced-and-were-listening/)。

### 如果复制到 Azure，我要支付哪些 Azure VM 费用？ 

在常规复制期间，数据将复制到异地冗余的 Azure 存储空间，你不需要支付任何 Azure IaaS 虚拟机费用（一个明显的优势）。当你故障转移到 Azure 时，站点恢复将自动创建 Azure IaaS 虚拟机，此后，你需要为你在 Azure 中使用的计算资源付费。

### 我可以使用站点恢复来管理分支机构的灾难恢复吗？

是的。当你使用站点恢复来协调分支机构的复制与故障转移时，可以在一个中心位置获得所有分支机构工作负载的统一视图。不需要前往分支机构，就可以从总部轻松对所有分支机构运行故障转移和管理灾难恢复。

### 你们是否提供可以用来将站点恢复工作流自动化的 SDK？

是的。可以使用 Rest API、PowerShell 或 Azure SDK 将站点恢复工作流自动化。了解有关[使用 PowerShell 部署 Site Recovery](/documentation/articles/site-recovery-deploy-with-powershell) 的详细信息。


## 安全性和遵从性

### 复制数据是否会发送到 Site Recovery 服务？

不会。Site Recovery 不拦截复制的数据，也不拥有虚拟机或物理服务器上运行哪些项目的任何相关信息。

复制数据在本地 Hyper-V 主机、VMware 虚拟机监控程序或物理服务器和 Azure 存储空间或辅助站点之间交换。站点恢复并不具有拦截该数据的能力。只有协调复制与故障转移所需的元数据将发送到站点恢复服务。

站点恢复已通过 ISO 27001:2005 认证，目前正在接受 HIPAA、DPA 和 FedRAMP JAB 评估。

### 为了遵从法规，即使是来自本地环境的元数据，也必须保留在同一个地理区域。Site Recovery 可以执行此操作吗？

是的。当你在某个区域中创建站点恢复保管库时，我们确保启用和协调复制与故障转移时所需的一切元数据都保留在该区域的地理边界范围内。

### 站点恢复是否将复制数据加密？
在本地站点之间复制虚拟机和物理服务器时，支持传输中加密。将虚拟机和物理服务器复制到 Azure 时，同时支持传输中加密和静态加密（Azure 中）。




## 复制和故障转移

### 如果要复制到 Azure，我需要哪种存储帐户？

你需要具有[标准异地冗余存储](/documentation/articles/storage-introduction#replication-for-durability-and-high-availability)的存储帐户。当前不支持高级存储。

### 我可以多久复制数据一次？
- **Hyper-V：**可以每隔 30 秒、5 分钟或 15 分钟复制一次在 Windows Server 2012 R2 上运行的 Hyper-V VM。如果你已设置 SAN 复制，则复制将是同步的。
- **VMware 和物理服务器：**此处的复制频率无关紧要。复制将是连续的。 

### 我可以将复制从现有的恢复站点扩展到其他恢复站点吗？
不支持扩展扩展或链式复制。


### 在首次复制到 Azure 时可以进行脱机复制吗？ 

不支持此操作。


### 可以从复制中排除特定的磁盘吗？

不支持此操作。

### 可以使用动态磁盘来复制虚拟机吗？

复制 Hyper-V 虚拟机时，支持使用动态磁盘。如果使用[增强版部署](/documentation/articles/site-recovery-vmware-to-azure-classic)，复制 VMware VM 和物理计算机时也支持使用动态磁盘。请注意，OS 磁盘必须是基本磁盘。


### 如何在故障转移到 Azure 后访问 Azure 虚拟机？ 

可以通过安全的 Internet 连接或者站点到站点 VPN 或 Azure ExpressRoute 访问 Azure VM。基于 VPN 连接的通信将发往 VM 所在的 Azure 网络上的内部端口。基于 Internet 的通信将映射到 Azure 云服务上 VM 的公共终结点。

### 如果故障转移到 Azure，Azure 如何确保我的数据可复原？

Azure 具有复原能力。站点恢复已经能够根据需要故障转移到符合 Azure SLA 的辅助 Azure 数据中心。发生此情况时，我们确保你的元数据和保管库都保留在你为保管库选择的相同地理区域。

### 如果在两个站点之间进行复制，当主站点发生意外中断时，会出现什么情况？

可以从辅助站点触发非计划的故障转移。站点恢复不需要来自主站点的连接即可执行故障转移。


### 故障转移是自动发生的吗？

故障转移不是自动的。你可以在门户中单击一下来启动故障转移，或者使用[ Site Recovery PowerShell cmdlet](https://msdn.microsoft.com/zh-cn/library/dn850420.aspx) 来触发故障转移。故障回复是 Site Recovery 门户中的一系列操作。

若要自动化故障转移，可以使用本地 Orchestrator 或 Operations Manager 来检测虚拟机故障，然后使用 SDK 来触发故障转移。

### 可以限制针对 Hyper-V 复制流量分配的带宽吗？
- 如果在两个本地站点上的 Hyper-V VM 之间进行复制，你可以使用 Windows QoS。下面是一个示例脚本： 

    	New-NetQosPolicy -Name ASRReplication -IPDstPortMatchCondition 8084 -ThrottleRate (2048*1024)
    	gpupdate.exe /force

- 如果将 Hyper-V VM 复制到 Azure，可以使用以下示例 PowerShell cmdlet 来设置限制：

    	Set-OBMachineSetting -WorkDay $mon, $tue -StartWorkHour "9:00:00" -EndWorkHour "18:00:00" -WorkHourBandwidth (512*1024) -NonWorkHourBandwidth (2048*1024)

- [了解](https://support.microsoft.com/zh-cn/kb/3056159)有关限制 Hyper-V 流量的详细信息。
- [了解](/documentation/articles/site-recovery-vmware-to-azure-classic#capacity-planning)有关限制 VMware 复制到 Azure 的详细信息。



## 服务提供商部署

### 站点恢复适用于专用或共享的基础结构模型吗？
是的，站点恢复同时支持专用与共享的基础结构模型。

### 我的租户标识是否与站点恢复服务共享？
不是。事实上，租户不需要访问 Site Recovery 门户。只有服务提供商管理员才能与门户交互。


### 我的租户应用程序数据是否会发往 Azure？
在服务提供商拥有的站点之间进行复制时，永远不会将应用程序数据发送到 Azure。数据进行传输中加密并直接在服务提供商站点之间复制。

如果是复制到 Azure，应用程序数据将发送到 Azure 存储空间而不是站点恢复服务。数据进行传输中加密并在 Azure 中保持加密状态。

### 我的租户会收到来自 Azure 服务的帐单吗？

不会。Azure 直接与服务提供商保持计费关系。服务提供商责任为其租户生成特定的帐单。

### 如果我复制到 Azure，需要在 Azure 中随时运行虚拟机吗？

不需要。数据会复制到你订阅中的异地冗余 Azure 存储帐户。当你执行测试故障转移（DR 演练）或实际故障转移时，站点恢复将在你的订阅中自动创建虚拟机。

### 当我复制到 Azure 时，你们确保提供租户级的隔离吗？

是的。

### 目前支持哪些平台？

我们支持 Azure Pack、云平台系统和 System Center（2012 和更高版本）部署。请在[为虚拟机配置保护](https://technet.microsoft.com/zh-cn/library/dn850370.aspx)中了解有关 Azure Pack 集成的详细信息。

### 你们是否支持单一 Azure Pack 和单一 VMM 服务器部署？

是，你可以复制 Hyper-V 虚拟机与 Azure，或者在服务提供商站点之间复制。请注意，如果在服务提供商站点之间复制，将无法使用 Azure Runbook 集成。


## 后续步骤

- 阅读 [Site Recovery 概述](/documentation/articles/site-recovery-overview)
- 了解有关 [Site Recovery 体系结构](/documentation/articles/site-recovery-components)的信息  

 

<!---HONumber=Mooncake_0328_2016-->