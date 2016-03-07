<properties
	pageTitle="可以通过 Azure Site Recovery 保护哪些工作负荷？" 
	description="Azure Site Recovery 可以协调本地虚拟机和物理服务器到 Azure 或辅助本地站点的复制、故障转移和恢复，从而保护工作负荷和应用程序" 
	services="site-recovery" 
	documentationCenter="" 
	authors="rayne-wiselman" 
	manager="jwhit" 
	editor=""/>

<tags 
	ms.service="site-recovery" 
	ms.date="12/01/2015" 
	wacn.date="01/21/2016"/>

# 可以通过 Azure Site Recovery 保护哪些工作负荷？

Azure Site Recovery 可以让客户部署应用程序感知型可用性解决方案，此类解决方案有助于实现组织的业务连续性和灾难恢复 (BCDR) 策略。不管它是基于 Windows Server 或 Linux 的应用、Microsoft 企业应用程序还是其他供应商的产品，你都可以通过 Azure Site Recovery 来启用灾难恢复、按需开发和测试环境以及云迁移。

Site Recovery 集成 Microsoft 应用程序，其中包括 SharePoint、Exchange、Dynamics、SQL Server 和 Active Directory。Microsoft 还与领先供应商（包括 Oracle、SAP、IBM 和 Red Hat）密切合作，以确保这些供应商的应用程序和服务能在 Microsoft 平台（例如 Azure 和 Hyper-V）上顺畅运行。你可以针对每个特定的应用程序自定义灾难恢复解决方案。


##主要功能
有助于实现应用程序级保护和恢复策略的 Azure Site Recovery 功能包括：

- 几乎同步的复制，RPO 低至 30 秒，满足大多数关键应用程序的需要。
- 针对单层或 N 层应用程序的应用一致性快照。
- 集成 SQL Server AlwaysOn，纳入了其他应用程序级复制技术，其中包括 AD 复制、SQL AlwaysOn、Exchange 数据库可用性组 (DAG) 和 Oracle 数据防护。
- 灵活的恢复计划，一次单击即可恢复整个应用程序堆栈，包括外部脚本或手动操作。 
- Site Recovery 和 Azure 中的高级网络管理可以简化应用的网络要求，包括保留 IP 地址、配置负载平衡器或集成 Azure 流量管理器以降低 RTO 网络切换数。
-  丰富的自动化库，提供特定于应用程序的生产就绪型脚本，可以下载并与 Site Recovery 集成。


##<a id="workload-guidance-summary"></a>工作负荷指南摘要

Site Recovery 复制技术与虚拟机中运行的任何应用程序兼容。另外，我们还与应用程序产品团队合作展开更多测试，以进一步支持每个应用。

**工作负荷** | <p>**复制 Hyper-V 虚拟机**</p> <p>**（到辅助站点）**</p> | <p>**复制 Hyper-V 虚拟机**</p> <p>**（到 Azure）**</p> | <p>**复制 VMware 虚拟机**</p> <p>**（到辅助站点）**</p> | <p>**复制 VMware 虚拟机**</p><p>**（到 Azure）**</p> 
---|---|---|---|---
Active Directory、DNS | Y | Y | Y | Y 
网站（IIS、SQL） | Y | Y | Y | Y 
SCOM | Y | Y | Y | Y
Sharepoint | Y | Y | Y | Y
<p>SAP</p><p>将非群集 SAP 站点复制到 Azure</p> | Y（Microsoft 已测试）| Y（Microsoft 已测试）| Y（Microsoft 已测试）| Y（Microsoft 已测试）
Exchange（非 DAG）| Y | 即将支持 | Y | Y
远程桌面/VDI | Y | Y | Y | N/A
<p>Linux</p> <p>（操作系统和应用）</p> | Y（Microsoft 已测试）| Y（Microsoft 已测试）| Y（Microsoft 已测试）| Y（Microsoft 已测试）
Dynamics AX | Y | Y | Y | Y
Dynamics CRM | Y | 即将支持 | Y | 即将支持
Oracle | Y（Microsoft 已测试）| Y（Microsoft 已测试）| Y（Microsoft 已测试）| Y（Microsoft 已测试）
Windows 文件服务器 | Y | Y | Y | Y

##保护 Active Directory 和 DNS

所有企业应用程序，例如 SharePoint, Dynamics AX 和 SAP，都依赖于 Active Directory 和 DNS 基础结构。在恢复工作负荷和应用之前，你将需要保护和恢复这些作为 BCDR 解决方案一部分的基础结构组件。

使用 Site Recovery，你可以为 Active Directory 和 DNS 创建一个完整的自动化灾难恢复计划。例如，如果你要将 Active Directory 用于主站点中的多个应用程序（如 SharePoint 和 SAP），而且希望故障转移整个站点，则可先使用恢复计划故障转移 Active Directory，然后再使用特定于应用程序的恢复计划故障转移依赖于 Active Directory 的应用程序。

[了解详细信息](/documentation/articles/site-recovery-active-directory)

##保护 SQL Server

SQL Server 是本地数据中心许多业务应用程序的数据服务基础。Site Recovery 和 SQL Server HA/DR 技术互为补充，可以一起用于为多层企业应用程序提供端到端保护。Site Recovery 为 SQL Server 环境提供以下优势：

- 轻松地对独立或群集的 SQL 服务器进行保护并将其复制到 Azure 或辅助站点。为多个 SQL Server 版本提供简单且经济高效的灾难恢复解决方案。
- 集成 SQL AlwaysOn 可用性组，在 Azure Site Recovery 中通过恢复计划管理故障转移和故障回复流程。
- 适用于整个应用程序堆栈（包括 SQL Server 数据库）的端到端恢复计划。
- 在出现高峰负载时使用 Azure Site Recovery 向上扩展 SQL Server，让这些负载“迸发”到 Azure 中更大型的 IaaS 虚拟机中。
- 使用 Azure Site Recovery，在 Azure 或辅助站点中测试 SQL Server。可以使用测试性故障转移进行分析和数据合规性检查，而不影响生产环境。

[了解详细信息](/documentation/articles/site-recovery-sql)

##保护 SharePoint

Azure Site Recovery 可帮助你保护 SharePoint 部署。使用 Site Recovery 可以：

- 消除对用于灾难恢复的备用场的需要以及相关的基础结构成本。使用 Site Recovery，你可以启用整个场（Web 层、应用层和数据库层）到 Azure 或到辅助站点的保护。
- 简化应用程序的部署和可管理性。部署到主站点的更新会被自动复制到辅助站点并且会在场恢复时变得可用。这消除了让备用场保持最新的管理复杂性，降低了成本。
- 按照副本环境的需要创建一个与生产类似的副本来进行测试和调试，从而简化 SharePoint 应用程序的开发与测试。
- 使用 Site Recovery 将 SharePoint 部署迁移到 Azure，从而简化从本地到云的过渡过程。

[了解详细信息](https://gallery.technet.microsoft.com/SharePoint-DR-Solution-f6b4aeae/)



##<a id="dynamics-ax"></a>保护 Dynamics AX
Azure Site Recovery 可以帮助你保护 Dynamics AX ERP 解决方案：

- 允许你通过 Site Recovery 启用整个 Dynamics Ax（Web 和 AOS 层、数据库层、SharePoint）到 Azure 或到辅助站点的保护。
- 使用 Site Recovery 将 Dynamics AX 部署迁移到 Azure，从而简化从本地到云的迁移过程。
- 通过按需创建一个与生产类似的副本来进行测试和调试，简化 Dynamics AX 应用程序的开发与测试。

[了解详细信息](https://gallery.technet.microsoft.com/Dynamics-AX-DR-Solution-b2a76281)

##<a id="rds"></a>保护 RDS 
远程桌面服务允许虚拟桌面基础结构 (VDI)、基于会话的桌面和应用程序，让用户能够在任何地方工作。使用 Site Recovery，你可以启用托管或非托管池虚拟桌面到辅助站点以及远程应用程序和会话到辅助站点或 Azure 的保护。

[了解详细信息](https://gallery.technet.microsoft.com/Remote-Desktop-DR-Solution-bdf6ddcb)



##<a id="exchange"></a>保护 Exchange

Microsoft Exchange 包括内置的对高可用性和灾难恢复的支持。Exchange DAG 和 Azure Site Recovery 可以协同工作。

- 在企业中进行 Exchange 灾难恢复时，Exchange DAG 是建议的解决方案。Site Recovery 中的恢复计划可以包含 DAG，以便跨站点协调 DAG 故障转移。
- 对于小型部署，例如单服务器或非群集服务器，Site Recovery 可以实施保护并进行目标为 Azure 或辅助站点的故障转移。

[了解详细信息](https://gallery.technet.microsoft.com/Exchange-DR-Solution-using-11a7dcb6)


##<a id="sap"></a>保护 SAP
使用 Site Recovery 来保护 SAP 部署：

- 启用整个 SAP 部署（具有不同的层）到 Azure 或到辅助站点的保护。
- 使用 Site Recovery 将 SAP 部署迁移到 Azure，从而简化云迁移。
- 按需创建一个与生产类似的副本来进行应用程序的测试和调试，从而简化 SAP 的开发与测试。

[了解详细信息](http://aka.ms/asr-sap)

<!---HONumber=Mooncake_0104_2016-->