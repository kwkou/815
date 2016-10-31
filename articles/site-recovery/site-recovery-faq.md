<properties 
	pageTitle="Site Recovery：常见问题 | Azure" 
	description="本文讨论了有关 Azure Site Recovery 的常见问题。" 
	services="site-recovery" 
	documentationCenter=""
	authors="rayne-wiselman"
	manager="jwhit"
	editor=""/>

<tags
	ms.service="site-recovery"
	ms.date="07/12/2016" 
	wacn.date="08/01/2016"/>


# Azure Site Recovery：常见问题 (FAQ)
## 关于本文

本文包含有关 Azure Site Recovery 的常见问题。如果在阅读本文后有任何问题，请在 [Azure 恢复服务论坛](https://social.msdn.microsoft.com/Forums/zh-cn/home?forum=hypervrecovmgr)上发布你的问题。


## 常规

###站点恢复的功能是什么？

站点恢复可通过协调和自动运行本地虚拟机与物理服务器到 Azure 或辅助数据中心的复制，来帮助实现业务连续性与灾难恢复 (BCDR)。[了解详细信息](/documentation/articles/site-recovery-overview/)。


### 站点恢复可以保护哪些计算机？

- **Hyper-V 虚拟机**：站点恢复可以保护 Hyper-V VM 上运行的任何工作负荷。
- **物理服务器**：站点恢复可以保护运行 Windows 或 Linux 的物理服务器。


### 在 Hyper-V 中我需要什么？

对于 Hyper-V 主机服务器，你的所需取决于部署方案。在以下内容中查看 Hyper-V 先决条件：

- [将 Hyper-V VM 复制到 Azure（不使用 VMM）](/documentation/articles/site-recovery-hyper-v-site-to-azure/#before-you-start)
- [将 Hyper-V VM 复制到 Azure（使用 VMM）](/documentation/articles/site-recovery-vmm-to-azure/#before-you-start)
- [将 Hyper-V VM 复制到辅助数据中心](/documentation/articles/site-recovery-vmm-to-vmm/#before-you-start)

- 如果要复制到辅助数据中心，请阅读 [Hyper-V 虚拟机的受支持的来宾操作系统](https://technet.microsoft.com/zh-cn/library/mt126277.aspx)。
- 如果你要复制到 Azure，Site Recovery 支持 [Azure 支持的](https://technet.microsoft.com/zh-cn/library/cc794868%28v=ws.10%29.aspx)所有来宾操作系统。

### 当 Hyper-V 在客户端操作系统上运行时，我可以保护 VM 吗？

不可以。VM 必须位于在受支持的 Windows 服务器计算机上运行的 Hyper-V 主机服务器上。


### 我可以使用站点恢复来保护哪些工作负荷？

可以使用站点恢复来保护在支持的 VM 或物理服务器上运行的大多数工作负荷。Site Recovery 为应用程序感知型复制提供支持，因此，应用可以恢复为智能状态。它除了与 Microsoft 应用程序（例如 SharePoint、Exchange、Dynamics、SQL Server 及 Active Directory）集成之外，还能与行业领先的供应商（包括 Oracle、SAP、IBM 及 Red Hat）紧密配合。[详细了解](/documentation/articles/site-recovery-workload/)工作负荷保护。


### Hyper-V 主机是否需要位于 VMM 云中？ 

如果要复制到辅助数据中心，那么 Hyper-V VM 就必须位于 VMM 云中的 Hyper-V 主机服务器上。如果想要复制到 Azure，那么可以复制 Hyper-V 主机服务器（无论是否位于 VMM 云中）上的 VM。[了解详细信息](/documentation/articles/site-recovery-hyper-v-site-to-azure/)。

### 如果我只有一个 VMM 服务器，可以部署站点恢复来配合 VMM 吗？ 

是的。你可以将 VMM 云中 Hyper-V 服务器上的 VM 复制到 Azure，或者在同一台服务器上的 VMM 云之间进行复制。对于本地到本地复制，我们建议在主站点与辅助站点中都部署一个 VMM 服务器。[了解详细信息](/documentation/articles/site-recovery-single-vmm/)




### 我可以使用站点恢复来管理分支机构的灾难恢复吗？

是的。当你使用站点恢复来协调分支机构的复制与故障转移时，可以在一个中心位置获得所有分支机构工作负载的统一视图。不需要前往分支机构，就可以从总部轻松对所有分支机构运行故障转移和管理灾难恢复。

## “安全”

### 复制数据是否会发送到 Site Recovery 服务？

不会。Site Recovery 不拦截复制的数据，也不拥有虚拟机或物理服务器上运行哪些项目的任何相关信息。
复制数据在本地 Hyper-V 主机、物理服务器和 Azure 存储空间或辅助站点之间交换。站点恢复并不具有拦截该数据的能力。只有协调复制与故障转移所需的元数据将发送到站点恢复服务。

站点恢复已通过 ISO 27001:2013、27018、HIPAA、DPA 认证，目前正在接受 SOC2 和 FedRAMP JAB 评估。


### 为了遵从法规，即使是来自本地环境的元数据，也必须保留在同一个地理区域。站点恢复可以帮助我们吗？

是的。当你在某个区域中创建站点恢复保管库时，我们确保启用和协调复制与故障转移时所需的一切元数据都保留在该区域的地理边界范围内。

### 站点恢复是否将复制数据加密？

在本地站点之间复制虚拟机和物理服务器时，支持传输中加密。将虚拟机和物理服务器复制到 Azure 时，同时支持传输中加密和静态加密（Azure 中）。


## 复制


### 将虚拟机复制到 Azure 需要满足任何先决条件吗？

要复制到 Azure 的虚拟机应符合 [Azure 要求](/documentation/articles/site-recovery-best-practices/#azure-virtual-machine-requirements)。

### 我可以将 Hyper-V 第 2 代虚拟机复制到 Azure 吗？

是的。站点恢复在故障转移过程中将从第 2 代转换成第 1 代。在故障回复时，计算机将转换回到第 2 代。[了解详细信息](https://azure.microsoft.com/blog/2015/04/28/disaster-recovery-to-azure-enhanced-and-were-listening/)。

### 如果复制到 Azure，我要支付哪些 Azure VM 费用？ 

在常规复制期间，数据将复制到异地冗余的 Azure 存储空间，你不需要支付任何 Azure IaaS 虚拟机费用（一个明显的优势）。当你故障转移到 Azure 时，站点恢复将自动创建 Azure IaaS 虚拟机，此后，你需要为你在 Azure 中使用的计算资源付费。


### 我是否可以使用某个 SDK 来自动化 ASR 工作流？

是的。可以使用 Rest API、PowerShell 或 Azure SDK 将站点恢复工作流自动化。当前支持的使用 PowerShell 部署站点恢复的方案：

- [将 VMM 云中的 Hyper-V VM 复制到 Azure PowerShell 经典](/documentation/articles/site-recovery-deploy-with-powershell/)
- [将 Hyper-V VM（无 VMM）复制到 Azure PowerShell 经典](/documentation/articles/site-recovery-hyper-v-site-to-azure-classic/)


### 如果要复制到 Azure，我需要哪种存储帐户？

**Azure 经典管理门户**：如果要在 Azure 经典管理门户中部署站点恢复，则你需要[标准异地冗余存储帐户](/documentation/articles/storage-redundancy/#geo-redundant-storage)。当前不支持高级存储。该帐户必须位于与站点恢复保管库相同的区域中。

### 我可以多久复制数据一次？

- **Hyper-V：**可以每隔 30 秒、5 分钟或 15 分钟复制 Hyper-V VM 一次。如果你已设置 SAN 复制，则复制将是同步的。

### 我可以将复制从现有的恢复站点扩展到其他站点吗？
不支持扩展扩展或链式复制。


### 在首次复制到 Azure 时可以进行脱机复制吗？ 

不支持此操作。


### 可以从复制中排除特定的磁盘吗？

不支持此操作。

### 可以使用动态磁盘来复制虚拟机吗？

复制 Hyper-V 虚拟机时，支持使用动态磁盘。

### 可以限制针对 Hyper-V 复制流量分配的带宽吗？

是的。你可以从以下部署文章中阅读更多有关限制带宽的信息：

- [Capacity planning for replicating Hyper-V VMs in VMM clouds（复制 VMM 云中的 Hyper-V VM 的容量规划）](/documentation/articles/site-recovery-vmm-to-azure/#step-5-capacity-planning)
- [Capacity planning for replicating Hyper-V VMs without VMM（复制无 VMM 的 Hyper-V VM 的容量规划）](/documentation/articles/site-recovery-hyper-v-site-to-azure/#step-5-capacity-planning)

## 故障转移


### 在故障转移到 Azure 之后，如何访问 Azure 虚拟机？ 

可以通过安全的 Internet 连接或者站点到站点 VPN 或 Azure ExpressRoute 访问 Azure VM。在连接之前你需要做许多准备。请从以下文章中阅读更多信息：


- [Connect to Azure VMs after failover of Hyper-V VMs in VMM clouds（故障转移 VMM 云中的 Hyper-V VM 后连接到 Azure VM）](/documentation/articles/site-recovery-vmm-to-azure/#step-7-test-your-deployment)
- [Connect to Azure VMs after failover of Hyper-V VMs without VMM（故障转移无 VMM 的 Hyper-V VM 后连接到 Azure VM）](/documentation/articles/site-recovery-hyper-v-site-to-azure/#step-7-test-the-deployment)


### 如果我故障转移到 Azure，Azure 如何确保我的数据可恢复？

Azure 具有复原能力。站点恢复已经能够根据需要故障转移到符合 Azure SLA 的辅助 Azure 数据中心。发生此情况时，我们确保你的元数据和保管库都保留在你为保管库选择的相同地理区域。

### 如果我在两个数据中心之间进行复制，当我的主数据中心发生意外的服务中断时，会出现什么情况？

可以从辅助站点触发非计划的故障转移。站点恢复不需要来自主站点的连接即可执行故障转移。

### 故障转移是自动发生的吗？

故障转移不是自动的。你可以在门户中单击一下来启动故障转移，或者使用 [Site Recovery PowerShell cmdlet](https://msdn.microsoft.com/zh-cn/library/dn850420.aspx) 来触发故障转移。故障回复也是 Site Recovery 门户中的一个简单操作。

若要自动化，你可以使用本地 Orchestrator 或 Operations Manager 来检测虚拟机故障，然后使用 SDK 来触发故障转移。

- [详细了解](/documentation/articles/site-recovery-create-recovery-plans/)恢复计划。
- 在[此处](/documentation/articles/site-recovery-failover/)阅读更多有关故障转移的信息。


## 服务提供商


### 我是服务提供商。站点恢复适用于专用或共享的基础结构模型吗？
是的，站点恢复同时支持专用与共享的基础结构模型。

### 对于服务提供商而言，我的租户标识是否与 Site Recovery 服务共享？
不是。租户标识是匿名的。租户不需要访问 Site Recovery 门户。只有服务提供商管理员才能与门户交互。


### 我的租户应用程序数据是否会发往 Azure？
在服务提供商拥有的站点之间进行复制时，永远不会将应用程序数据发送到 Azure。数据进行传输中加密并直接在服务提供商站点之间复制。

如果是复制到 Azure，应用程序数据将发送到 Azure 存储空间而不是站点恢复服务。数据进行传输中加密并在 Azure 中保持加密状态。


### 我的租户会收到来自 Azure 服务的帐单吗？

不会。Azure 直接与服务提供商保持计费关系。服务提供商责任为其租户生成特定的帐单。

### 如果我复制到 Azure，需要在 Azure 中随时运行虚拟机吗？

不需要。数据会复制到你订阅中的 Azure 存储帐户。当你执行测试故障转移（DR 演练）或实际故障转移时，站点恢复将在你的订阅中自动创建虚拟机。

### 当我复制到 Azure 时，你们确保提供租户级的隔离吗？

是的。

### 目前支持哪些平台？

我们支持 Azure Pack、云平台系统和基于 System Center 的（2012 和更高版本）的部署。[了解更多](https://technet.microsoft.com/zh-cn/library/dn850370.aspx)有关 Azure Pack 和 Site Recovery 集成的信息。

### 你们是否支持单一 Azure Pack 和单一 VMM 服务器部署？

是，你可以复制 Hyper-V 虚拟机到 Azure，或者在服务提供商站点之间复制。请注意，如果在服务提供商站点之间复制，将无法使用 Azure Runbook 集成。


## 后续步骤

- 阅读 [Site Recovery 概述](/documentation/articles/site-recovery-overview/)
- 了解有关 [Site Recovery 体系结构](/documentation/articles/site-recovery-components/)的信息

 

<!---HONumber=Mooncake_0725_2016-->