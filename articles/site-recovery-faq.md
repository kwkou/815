<properties
	pageTitle="Azure Site Recovery：常见问题"
	description="本文讨论了有关使用 Azure Site Recovery 的常见问题。"
	services="site-recovery"
	documentationCenter=""
	authors="csilauraa"
	manager="jwhit"
	editor="tysonn"/>

<tags
	ms.service="site-recovery"
	ms.date="05/12/2015"
	wacn.date="10/03/2015"/>


# Azure Site Recovery：常见问题
## 关于本文

本文包含有关 Azure Site Recovery 的常见问题。有关 Site Recovery 和相关部署方案的简介，请阅读 [Site Recovery 概述](/documentation/articles/hyper-v-recovery-manager-overview)。

如果在阅读本文后还有其他问题，可以在 [Azure 恢复服务论坛](https://social.msdn.microsoft.com/Forums/zh-CN/home?forum=hypervrecovmgr)上发布你的问题。


## 在企业与 Hyper-V 站点之间复制：Hyper-V 副本
### Azure Site Recovery 安全吗？ 你们会将哪些数据发送到 Azure？

是的，Azure Site Recovery (ASR) 非常安全。ASR 既不截获你的应用程序数据，也不获取有关你的虚拟机中正在运行哪些程序的任何信息。不需要从 Hyper-V 主机或虚拟机建立 Internet 连接。

由于复制是在你自己的企业与服务提供商站点之间进行的，因此，不会将任何应用程序数据发送到 Azure。只有协调故障转移所需的元数据（例如 VM 或云名称）才会发送到 Azure。ASR 无法拦截你的应用程序数据，这些数据始终保留在本地。

Azure Site Recovery 已通过 ISO 27001:2005 认证，目前正在接受 HIPAA、DPA 和 FedRAMP JAB 评估。

### 法规要求，即使是来自本地环境的元数据，也要保留在同一个地理区域。Azure Site Recovery 能够确保我们符合这一要求吗？

是的。当你在选择的区域中创建 ASR 保管库时，我们确保需要用来启用和协调灾难恢复设置的所有元数据都保留在该区域的地理边界范围内。

### 如果 Azure 服务中断怎么办？ 可以从 SCVMM 触发故障转移吗？
尽管 Azure 在设计上提供服务弹性，但我们也要对最坏的情况做好准备。ASR 已经能够根据 Azure SLA 故障转移到辅助 Azure 数据中心。我们还确保，即使发生 Azure 服务中断，你的元数据和 ASR 保管库也仍会保留在你为保管库选择的同一个地理区域内。


### 如果在发生灾难时，主要数据中心的 ISP 也发生了服务中断怎么办？ 如果辅助数据中心的 ISP 也发生了服务中断怎么办？
你可以使用 ASR 门户执行非计划的故障转移，单击一下鼠标即可。执行故障转移不需要从主要数据中心建立连接。

故障转移后，要在辅助数据中心运行的应用程序可能需要某种形式的 Internet 连接。ASR 假设即使灾难期间主站点和 ISP 同时受到影响，辅助站点的 SCVMM 服务器仍然连接到 ASR。


### 我是否可以使用某个 SDK 来自动化 ASR 工作流？
是的。你可以使用 Rest API、PowerShell 或 Azure SDK 来自动化 ASR 工作流。可以在标题为 [Azure Site Recovery 的 PowerShell 支持简介](http://azure.microsoft.com/blog/2014/11/05/introducing-powershell-support-for-azure-site-recovery/)的博客文章中找到更多详细信息。

### 你们提供哪些解决方案让我监视已设置的以及正在进行的复制？
你可以直接从 ASR 服务订阅电子邮件警报，这样便可以接收有关需要关注的任何问题的通知。此外，你也可以使用 SCOM 监视进行中的复制的运行状况。

### 支持哪些版本的 Windows Server 主机和群集？
在选择 Hyper-V 副本以便在 Hyper-V 站点之间启用复制和保护时，可以使用 Windows Server 2012 和 Windows Server 2012 R2。


### 支持哪些版本的来宾操作系统？
标题为[关于虚拟机和来宾操作系统](https://technet.microsoft.com/zh-cn/library/cc794868%28v=ws.10%29.aspx)的主题中提供了受支持来宾操作系统的最新列表。


## 在企业与 Hyper-V 站点之间复制：基于 SAN 存储阵列的复制
### 如果我已设置了基于 SAN 的复制，但我不想对其进行更改，这样可以吗？
没问题。ASR 支持已设置应用程序的方案，在这种情况下，你可以利用 ASR 来协调以虚拟机为中心的灾难恢复、配置灾难恢复网络、计划应用程序感知的恢复、演练灾难恢复，以及制定合规性和审核报告要求。


### 我是否需要 SCVMM？ 是否需要在两端使用 SCVMM？
是的。我们需要使用特定于阵列的 SMI-S 提供程序，使 SAN 阵列受到 SVMM 的管理。

我们支持基于阵列类型的单一 SCVMM HA 部署，不过，建议的配置是使用独立的 SCVMM 服务器来管理站点。


### 如果我不确定存储管理员是谁会怎么样？
我们会处理你的存储管理员设置的现有复制，这意味着，存储管理员不需要对其阵列进行任何更改。但是，想要通过 SCVMM 自动化存储管理的组织也可以使用 ASR 和 SCVMM 设置存储。

### 我是否可以支持同步复制？ 来宾群集呢？ 共享存储呢？
都可以！

### 网络间复制：我需要在本地安装哪些程序以实现此目的？
使用 (SAN) 基于阵列的复制在 Hyper-V 站点之间启用复制和保护时，你只需在 SCVMM 服务器上安装 ASR DR 提供程序。请记住，这也是需要连接到 Internet 的*唯一*主机。


SCVMM 还需要相关存储供应商提供的更新 SMI-S 提供程序发现你的阵列。

## 在服务提供商站点之间：

### 此解决方案是否适用于专用或共享的基础结构模型？
是的，ASR 支持专用和共享的基础结构模型。

### 我的租户标识是否与 Azure 共享？
不是。事实上，你的租户不需要访问 ASR 门户。只有服务提供商管理员才会在 Azure 中的 ASR 门户上执行操作。

### 我的租户是否会收到来自 Azure 的帐单？
不会。Windows Azure 直接与服务提供商保持计费关系。服务提供商责任为其租户生成特定的帐单。

### 我的租户应用程序数据是否会发往 Azure？
在使用服务提供商拥有的站点进行灾难恢复时，应用程序数据永远不会传到 Azure。数据将会加密，并直接在服务提供商站点之间复制。将 Azure 用作 DR 站点时，应用程序数据将发送到 Azure – 静态数据和传输中的数据始终处于加密状态。你也可以使用 VPN、ExpressRoute 或公共 Internet 来与 Azure 建立连接。

### 使用 Azure 作为 DR 站点时，我们是否随时都要在 Azure 中运行虚拟机？
不用，ASR 是一流的公有云 DRaaS 解决方案。数据将复制到你自己的订阅中的地域冗余 Azure 存储帐户。当你执行 DR 演练或实际故障转移时，ASR 将在你的订阅中自动创建虚拟机。

### 当我选择使用 Azure 作为 DR 站点时，你们是否确保租户级的隔离？
是的。

### 目前支持哪些平台？
我们支持 Azure Pack、云平台系统和基于 System Center 的 Hyper 2012 及更高版本的部署。若要了解有关 ASR 和 Azure Pack 集成的详细信息，请参阅[为虚拟机配置保护](https://technet.microsoft.com/zh-cn/library/dn850370.aspx)。

### 你们是否还支持单一 Azure Pack 和单一 SCVMM 部署？
是的，我们针对以下任一方案支持单一 SCVMM 部署：服务提供商站点之间的灾难恢复，以及服务提供商站点与 Azure 之间的灾难恢复。

如果在服务提供商站点之间启用了灾难恢复，则还支持单一 Azure Pack 部署，但是，ASR Runbook 集成不适用于此拓扑。

## Hyper-V 站点与 Azure 之间（SCVMM 是可选的）

### ASR 是否支持故障回复？
是的。通过 ASR 门户单击一个鼠标，就能从 Azure 故障回复到本地站点。

### ASR 是否要求启用站点到站点 VPN？
没有这种强制要求；ASR 也可以通过公共 Internet 运行。但是，如果配置了站点到站点 VPN，则你可以像以往一样访问故障转移的虚拟机。有关在启用到 Azure 的灾难恢复时的网络注意事项详细信息，请参阅[用作灾难恢复站点的 Windows Azure 的网络基础结构设置](http://azure.microsoft.com/blog/2014/09/04/networking-infrastructure-setup-for-microsoft-azure-as-a-disaster-recovery-site/)博客文章。

### 除了 ASR 保护费用费用以外，我是否还要支付其他任何费用？
在稳定状态下，我们会将更改复制到地域冗余的 Azure 存储空间，此时，你不需要支付任何 Azure IaaS 虚拟机费用（一个明显的优势）。当你故障转移到 Azure 时，ASR 会自动创建 Azure IaaS 虚拟机，此后，你需要为你在 Azure 中使用的计算资源付费。

### 我的分支机构未安装 SCVMM。我是否可以为分支机构中的工作负载启用 Azure 保护？ 使用 ASR 提供这种保护有哪些主要优势？
是的，你可以使用 ASR 来保护不受 SCVMM 管理的 Hyper-V 主机上运行的虚拟机。

当你使用 ASR 来管理所有分支机构的灾难恢复需求时，可以在一个中心位置获得所有分支机构工作负载的统一视图。此外，你还可以从总部实现对所有分支机构的单键式故障转移和灾难恢复管理，而无需前往分支机构。目前，不支持使用网络映射和静态加密来实现分支机构灾难恢复。

### 在分支机构中，可以保护的虚拟机数量是否有限制？
没有。支持的数量与企业方案相同。

### 支持哪些版本的 Windows Server 主机和群集？
在选择 Hyper-V 副本以便在 Hyper-V 站点之间启用复制和保护时，可以使用 Windows Server 2012 和 Windows Server 2012 R2。


### 支持哪些版本的来宾操作系统？
标题为[关于虚拟机和来宾操作系统](https://technet.microsoft.com/zh-cn/library/cc794868%28v=ws.10%29.aspx)的文章中提供了受支持来宾操作系统的最新列表。

## 后续步骤

开始部署 ASR：

- [设置本地 VMM 站点与 Azure 之间的保护](/documentation/articles/site-recovery-vmm-to-azure)
- [在本地 Hyper-V 站点与 Azure 之间设置保护](/documentation/articles/site-recovery-hyper-v-site-to-azure)
- [设置两个本地 VMM 站点之间的保护](/documentation/articles/site-recovery-vmm-to-vmm)
- [使用 SAN 在两个本地 VMM 站点之间设置保护](/documentation/articles/site-recovery-vmm-san)
- [使用单个 VMM 服务器设置保护](/documentation/articles/site-recovery-single-vmm)

<!---HONumber=71-->