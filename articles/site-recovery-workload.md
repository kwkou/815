<properties
	pageTitle="Site Recovery 工作负荷指南 | Windows Azure" 
	description="Azure Site Recovery 可以协调位于本地的虚拟机和物理服务器到 Azure 或辅助本地站点的复制、故障转移和恢复。" 
	services="site-recovery" 
	documentationCenter="" 
	authors="prateek9us" 
	manager="abhiag" 
	editor=""/>

<tags 
	ms.service="site-recovery" 
	ms.date="09/21/2015" 
	wacn.date="10/22/2015"/>

# Site Recovery 工作负荷指南

Azure Site Recovery 让客户能够部署应用程序感知可用性解决方案。它是基于 Windows Server 或 Linux 的应用程序、Microsoft 第一方企业应用程序或其他供应商的产品，你可以使用 Azure Site Recovery，让灾难恢复、按需开发/测试环境或云迁移成为可能。

Microsoft 在开发同类最佳企业应用程序（例如 SharePoint、Exchange、Dynamics、SQL Server 和 Active Directory）方面具有深厚的专业知识和经验。Microsoft 还与领先供应商密切合作，包括 Oracle、SAP、IBM 和 Red Hat，以确保他们的应用程序和服务能在 Microsoft 平台（包括 Azure 和 Hyper-V）上顺畅运行。我们已经充分利用这些特有的优势来加强对每个应用程序的可用性要求的深入理解，以让你能够部署可以为每个应用程序量身订做的同类最佳灾难恢复和可用性解决方案。


##主要功能
Azure Site Recovery 功能在设计时牢记应用程序级别的保护/恢复：

- 几乎同步的复制，RPO 低至 30 秒，满足大多数关键应用程序的需要。
- 针对单层或 N 层应用程序的应用程序一致快照
- 集成应用程序级复制。充分利用同类最佳应用程序级产品，包括 AD 复制、SQL Always On、Exchange Database Availability Groups 和 Oracle Data Guard
- 灵活恢复计划让一次单击即可恢复整个应用程序堆栈成为可能，包括执行外部脚本甚至手动操作。 
- ASR 和 Azure 中的高级网络管理让你的应用程序特有的所有网络配置都实现自动化：保留 IP 地址、配置负载平衡器，或使用 Microsoft 的流量管理器以实现低 RTO 网络切换。
- 丰富自动化库 (Rich Automation Library)，提供为生产做好准备的应用程序特定脚本。下载它们并集成到基于 ASR 的解决方案。


##工作负荷指南摘要

ASR 复制技术与虚拟机中运行的任何应用程序兼容。我们与应用程序产品团队合作展开了更多测试，以进一步支持每个应用程序。

**工作负荷** | <p>**复制 Hyper-V 虚拟机**</p> <p>**（到辅助站点）**</p> | <p>**复制 Hyper-V 虚拟机**</p> <p>**（到 Azure）**</p> | <p>**复制 VMware 虚拟机**</p> <p>**（到辅助站点）**</p> | <p>**复制 VMware 虚拟机**</p><p>**（到 Azure）**</p> 
---|---|---|---|---
Active Directory、DNS | Y | Y | Y | Y 
Web 应用程序（IIS、SQL） | Y | Y | Y | Y 
SCOM | Y | Y | Y | Y 
Sharepoint | Y | Y | Y | Y 
<p>SAP</p><p>将非群集 SAP 站点复制到 Azure</p> | Y（Microsoft 已测试）| Y（Microsoft 已测试）| Y（Microsoft 已测试）| Y（Microsoft 已测试）
Exchange（非 DAG）| Y | 即将支持 | Y | Y
远程桌面/VDI | Y | Y | Y | N/A 
<p>Linux</p> <p>（操作系统和应用程序）</p> | Y（Microsoft 已测试）| Y（Microsoft 已测试）| Y（Microsoft 已测试）| Y（Microsoft 已测试）
Dynamics AX | Y | Y | Y | Y
Dynamics CRM | Y | 即将支持 | Y | 即将支持
Oracle | Y（Microsoft 已测试）| Y（Microsoft 已测试）| Y（Microsoft 已测试）| Y（Microsoft 已测试）
Windows 文件服务器 | Y | Y | Y | Y

##Active Directory 和 DNS

所有企业应用程序，例如 SharePoint, Dynamics AX 和 SAP，都依赖于 AD 和 DNS 基础结构才能正常工作。在为任何此类应用程序创建灾难恢复 (DR) 解决方案时，如果出现中断，在恢复应用程序的其他组件之前保护和恢复 AD 非常重要。

使用 Azure Site Recovery，你可以为你的 AD 创建一个完整的自动化灾难恢复计划。如果你在主站点中有一个用于多个应用程序（例如 SharePoint 和 SAP）的 AD，并且你决定对整个站点进行故障转移，则可以首先使用 AD 恢复计划对 AD 进行故障转移，然后使用应用程序特定恢复计划对其他应用程序进行故障转移。

请参阅链接的文档以获取有关[部署面向 AD 的 Azure Site Recovery](/documentation/articles/site-recovery-active-directory) 的详细指南

##SQL Server
Microsoft SQL Server 是很多企业级第一方、第三方和自定义业务线应用程序的基础，这些应用程序在客户的本地数据中心内运行。SharePoint、Dynamics 和 SAP 等应用程序使用 SQL Server 提供数据服务。Azure Site Recovery 和 SQL Server HA/DR 技术是补充，可用于为多层企业应用程序提供端到端保护。Azure Site Recovery 为 SQL Server 环境提供以下优势：

- 轻松地将独立或群集 SQL 服务器保护到 Azure 或辅助站点。扩展面向任何版本的 SQL Server 的、符合成本效益的简单灾难恢复解决方案。
- 集成同类最佳灾难恢复解决方案 SQL Always On Availability Groups，并使用 ASR 恢复计划管理故障转移/故障回复操作。
- 为企业应用程序堆栈（包括 SQL 数据库）创建端到端恢复计划。
- 在高峰负荷期间使用 Azure Site Recovery，通过在 Azure 中将它们“爆发”到更大的 IaaS 虚拟机大小来扩大 SQL Server。
- 使用 Azure Site Recovery，在 Azure 或辅助站点中创建 SQL Server 的测试副本。使用此复本进行分析和数据合规性检查，而不影响生产环境。

请参阅链接的文档以获取有关[部署面向 SQL 的 Azure Site Recovery](/documentation/articles/site-recovery-sql) 的详细指南

##SharePoint
SharePoint 是一款流行的应用程序，让众多组织能够通过提供 Intranet 门户、文档和文件管理、社交网络、网站和企业搜索能力来进行合作。它也是一个应用程序平台，用于轻松地部署自定义应用程序和工作流程。通过使用面向 SharePoint 的 Azure Site Recovery，你可以：

- 消除对用于灾难恢复的备用场的需要以及相关的基础结构成本。使用 ASR，你可以启用整个场（Web、应用程序和数据库层）到 Azure 或到辅助站点的保护。
- 简化应用程序的部署和可管理性。部署到主站点的更新会被自动复制到辅助站点并在场恢复时变得可用。这消除了让备用场保持最新的管理复杂性，并降低了总拥有成本 (TCO)。
- 通过按需创建一个与生产类似的副本来进行测试和调试，让 SharePoint 应用程序的开发与测试变得更加容易。
- 通过使用 ASR 将 SharePoint 部署迁移到 Azure 来简化部署到云的过程。

请参阅链接的文档了获取有关[部署面向 Sharepoint 的 Azure Site Recovery](http://aka.ms/asr-sharepoint) 的详细指南


##Dynamics AX
Microsoft Dynamics AX 是一款领先的企业资源规划 (ERP) 解决方案，易学易用，让你能够更快的交付价值，利用商机，在整个组织内推动用户参与和创新。

- 使用 ASR，你能够启用整个 Dynamics Ax（Web 和 AOS 层、数据库层、SharePoint）到 Azure 或到辅助站点的保护。
- 通过使用 ASR 将 Dynamics Ax 部署迁移到 Azure 来简化部署到云的过程。
- 通过按需创建一个与生产类似的副本来进行测试和调试，让 Dynamics 应用程序的开发与测试变得更加容易。

请参阅链接的文档以获取有关[部署面向 Dynamics AX 的 Azure Site Recovery](http://aka.ms/asr-dynamics) 的详细指南

## RDS 
Windows Server 远程桌面服务 (RDS) 加快并扩展桌面和应用程序到任何设备的部署，提高远程工作者的效率，同时有助于保持关键知识产权的安全，简化监管合规。远程桌面服务允许虚拟桌面基础结构 (VDI)、基于会话的桌面和应用程序，让用户能够在任何地方工作。

使用 ASR，你可以启用托管或非托管池虚拟桌面到辅助站点以及远程应用程序和会话到辅助站点或 Azure 的保护。

请参阅链接的文档以获取有关[部署面向 RDS/VDI 的 Azure Site Recovery](http://aka.ms/asr-rds) 的详细指南


## Exchange
Microsoft Exchange 是企业用来承载其消息和电子邮件服务的首选软件。Exchange 确保跨 PC、手机或浏览器的通信，同时提供无与伦比的可靠性、可管理性和数据保护。

Microsoft Exchange 本身支持企业级高可用性和灾难恢复解决方案。Exchange Database Availability Groups 和 Azure Site Recovery 技术是补充。

- Exchange DAG 是推荐的部署选项，让 Exchange 部署的灾难恢复成为同类最佳。ASR 恢复计划可包含 DAG 以跨站点安排 DAG 故障转移。
- 对于小型部署，例如单服务器或非群集服务器，Azure Site Recovery 可保护服务器并将单个服务器向 Azure 或向辅助站点进行故障转移。

请参阅链接的文档以获取有关[部署面向 Exchange 的 Azure Site Recovery](http://aka.ms/asr-exchange) 的详细指南

## SAP

SAP 是一款领先的企业资源规划 (ERP) 软件，世界各地的很多组织都在使用。SAP 通常被视为企业中最关键的应用程序之一。这些应用程序的体验结构和操作极为复杂，并且确保你满足 BCDR 的要求非常重要。

- 使用 ASR，你可以启用整个 SAP 部署（具有不同的层）到 Azure 或到辅助站点的保护。
- 通过使用 ASR 将 SAP 部署迁移到 Azure 来简化部署到云的过程。
- 通过按需创建一个与生产类似的副本来进行测试和调试，让 SAP 应用程序的开发与测试变得更加容易。

请参阅链接的文档以获取有关[部署面向 SAP NetWeaver 的 Azure Site Recovery](http://aka.ms/asr-sap) 的详细指南

<!---HONumber=74-->