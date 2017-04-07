<properties
    pageTitle="可以通过 Azure Site Recovery 保护哪些工作负荷？"
    description="Azure Site Recovery 可以协调本地虚拟机和物理服务器到 Azure 或辅助本地站点的复制、故障转移和恢复，从而保护工作负荷和应用程序"
    services="site-recovery"
    documentationcenter=""
    author="rayne-wiselman"
    manager="cfreeman"
    editor="" />
<tags
    ms.assetid="4953948f-26c0-4699-8fe7-59d3bfc1d3da"
    ms.service="site-recovery"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="na"
    ms.workload="storage-backup-recovery"
    ms.date="02/06/2017"
    wacn.date="03/10/2017"
    ms.author="raynew" />  


# 可以通过 Azure Site Recovery 保护哪些工作负荷？


本文介绍可以使用 Azure Site Recovery 服务复制的工作负荷和应用程序。

请将任何评论或问题发布到本文底部，或者发布到 [Azure 恢复服务论坛](https://social.msdn.microsoft.com/Forums/zh-cn/home?forum=hypervrecovmgr)。

## 概述

组织需要制定业务连续性和灾难恢复 (BCDR) 策略来确保工作负荷和数据在计划和非计划停机期间保持安全和可用，并尽快恢复正常运行。

Site Recovery 就是能够帮助实现 BCDR 策略的一个 Azure 服务。使用 Site Recovery，可将应用程序感知的复制部署到云或辅助站点中。无论应用是基于 Windows 还是 Linux，是在物理服务器还是 Hyper-V 上运行，都可以使用 Site Recovery 来协调复制，执行灾难恢复测试，以及运行故障转移和故障回复。


Site Recovery 与 Microsoft 应用程序集成，其中包括 SharePoint、Exchange、Dynamics、SQL Server 和 Active Directory。Microsoft 还与 Oracle、SAP、IBM 和 Red Hat 等领先供应商密切合作。你可以针对每个应用自定义复制解决方案。

## 为何使用 Site Recovery 复制应用程序？

站点恢复可帮助实现应用程序级的保护和恢复，如下所示：

- 不区分应用，为受支持计算机上运行的任何工作负荷提供复制。
- 几乎同步的复制，RPO 低至 30 秒，满足大多数关键业务应用的需要。
- 针对单层或多层应用程序的应用一致性快照。
- 集成 SQL Server AlwaysOn，纳入了其他应用程序级复制技术，其中包括 AD 复制、SQL AlwaysOn、Exchange 数据库可用性组 (DAG) 和 Oracle 数据防护。
- 灵活的恢复计划，一次单击即可恢复整个应用程序堆栈，包括在计划中使用外部脚本和手动操作。
- 站点恢复和 Azure 中的高级网络管理可以简化应用的网络要求，包括保留 IP 地址、配置负载均衡或集成 Azure 流量管理器以降低 RTO 网络切换数。
-  丰富的自动化库，提供特定于应用程序的生产就绪型脚本，可以下载并与恢复计划集成。


##<a id="workload-guidance-summary"></a><a name="workload-summary"></a>工作负荷摘要


站点恢复可复制受支持计算机上运行的任何应用。此外，我们已经与产品团队合作执行其他特定于应用的测试。

**工作负载** | **将 Hyper-V VM 复制到辅助站点** | **将 Hyper-V VM 复制到 Azure** 
---|---|---
Active Directory、DNS | Y | Y 
Web 应用（IIS、SQL） | Y | Y 
System Center Operations Manager | Y | Y 
Sharepoint | Y | Y 
SAP<br/><br/>将非群集 SAP 站点复制到 Azure | Y（Microsoft 已测试） | Y（Microsoft 已测试） 
Exchange（非 DAG） | Y | 即将支持 
远程桌面/VDI | Y | Y 
Linux（操作系统和应用） | Y（Microsoft 已测试） | Y（Microsoft 已测试） 
Dynamics AX | Y | Y 
Dynamics CRM | Y | 即将支持 
Oracle | Y（Microsoft 已测试） | Y（Microsoft 已测试） 
Windows 文件服务器 | Y | Y 


## 复制 Active Directory 和 DNS

Active Directory 和 DNS 基础结构对于大多数企业应用而言至关重要。在灾难恢复过程中恢复工作负荷和应用之前，需要保护和恢复这些基础结构组件。

你可以使用站点恢复，为 Active Directory 和 DNS 创建一个完整的自动化灾难恢复计划。例如，若要将 SharePoint 和 SAP 从主站点故障转移到辅助站点，可以先设置可故障转移 Active Directory 的恢复计划，然后设置额外的应用特定计划，以故障转移依赖于 Active Directory 的其他应用。

[详细了解](/documentation/articles/site-recovery-active-directory/)如何保护 Active Directory 和 DNS。

## 保护 SQL Server

SQL Server 是本地数据中心许多业务应用的数据服务基础。站点恢复可与 SQL Server HA/DR 技术一起用于保护采用 SQL Server 的多层企业应用。Site Recovery 提供：

- 为 SQL Server 提供简单且经济高效的灾难恢复解决方案。将多个版本的 SQL Server 独立服务器和群集复制到 Azure 或辅助站点。
- 集成 SQL AlwaysOn 可用性组，使用 Azure Site Recovery 恢复计划管理故障转移和故障回复。
- 适用于应用程序中所有层（包括 SQL Server 数据库）的端到端恢复计划。
- 在出现高峰负载时使用站点恢复扩展 SQL Server，让这些负载“迸发”到 Azure 中更大型的 IaaS 虚拟机中。
- 简单的 SQL Server 灾难恢复测试。可以运行测试故障转移来分析数据，并可以运行合规性检查，且不影响生产环境。

[详细了解](/documentation/articles/site-recovery-sql/)如何保护 SQL Server。

##<a name="sharepoint"></a><a name="protect-sharepoint"></a>保护 SharePoint

Azure Site Recovery 可帮助保护 SharePoint 部署，如下所示：

- 消除对用于灾难恢复的备用场的需要以及相关的基础结构成本。使用站点恢复将整个场（Web 层、应用层和数据库层）复制到 Azure 或辅助站点。
- 简化应用程序部署和管理。部署到主站点的更新将自动复制，因此可在故障转移和恢复辅助站点中的场之后使用。此外，还可降低使备用场保持最新状态的管理复杂性和相关成本。
- 按照副本环境的需要创建一个与生产类似的副本来进行测试和调试，从而简化 SharePoint 应用程序的开发与测试。
- 使用站点恢复将 SharePoint 部署迁移到 Azure，从而简化到云的过渡过程。

[详细了解](https://gallery.technet.microsoft.com/SharePoint-DR-Solution-f6b4aeae)如何保护 SharePoint。



##<a id="dynamics-ax"></a><a name="protect-dynamics-ax"></a>Dynamics AX
Azure Site Recovery 可通过以下方式帮助保护 Dynamics AX ERP 解决方案：

- 协调整个 Dynamics AX 环境（Web 和 AOS 层、数据库层、SharePoint）到 Azure 或到辅助站点的复制。
- 简化 Dynamics AX 部署到云 (Azure) 的迁移。
- 通过按需创建一个与生产类似的副本来进行测试和调试，简化 Dynamics AX 应用程序的开发与测试。

[详细了解](https://gallery.technet.microsoft.com/Dynamics-AX-DR-Solution-b2a76281)如何保护 Dynamic AX。

##<a id="rds"></a><a name="protect-rds"></a>RDS 
远程桌面服务 (RDS) 允许虚拟桌面基础结构 (VDI)、基于会话的桌面和应用程序，让用户能够在任何地方工作。使用 Azure Site Recovery 可以：

- 将托管或非托管的共用虚拟桌面复制到辅助站点，并将远程应用程序和会话复制到辅助站点或 Azure。
- 下面是可以复制的项：

**RDS** | **将 Hyper-V VM 复制到辅助站点** | **将 Hyper-V VM 复制到 Azure** | **将物理服务器复制到辅助站点** | **将物理服务器复制到 Azure**
---|---|---|---|---
**共用虚拟桌面（非托管）** | 是 | 否 | 是 | 否
**共用虚拟桌面（托管但不包含 UPD）** | 是 | 否 | 是 | 否
**远程应用程序和桌面会话（不包含 UPD）** | 是 | 是 | 是 | 是


[详细了解](https://gallery.technet.microsoft.com/Remote-Desktop-DR-Solution-bdf6ddcb)如何保护 RDS。



##<a id="exchange"></a><a name="protect-exchange"></a>Exchange
Site Recovery 可按如下所示帮助保护 Exchange：

- 对于小型 Exchange 部署，例如单一服务器或独立服务器，Site Recovery 可以复制和故障转移到 Azure 或辅助站点。
- 对于大型部署，Site Recovery 可与 Exchange DAGS 集成。
- 在企业中进行 Exchange 灾难恢复时，Exchange DAG 是建议的解决方案。Site Recovery 中的恢复计划可以包含 DAG，以便跨站点协调 DAG 故障转移。


[详细了解](https://gallery.technet.microsoft.com/Exchange-DR-Solution-using-11a7dcb6)如何保护 Exchange。


##<a id="sap"></a><a name="protect-sap"></a>SAP
按如下所示使用 Site Recovery 保护 SAP 部署：

- 通过将不同的部署层复制到 Azure 或辅助站点保护整个 SAP 部署。
- 使用 Site Recovery 将 SAP 部署迁移到 Azure，从而简化云迁移。
- 按需创建一个与生产类似的副本来进行应用程序的测试和调试，从而简化 SAP 的开发与测试。

[详细了解](http://aka.ms/asr-sap)如何保护 SAP。

## 后续步骤

[准备 Site Recovery 部署](/documentation/articles/site-recovery-best-practices/)

<!---HONumber=Mooncake_0306_2017-->
<!--Update_Description: wording update-->