<properties
    pageTitle="复原技术指南索引 | Azure"
    description="有关了解和设计具有复原能力、高可用性、容错能力的应用程序，以及针对灾难恢复和业务连续性进行规划的技术文章索引"
    services=""
    documentationcenter="na"
    author="adamglick"
    manager="saladki"
    editor="" />
<tags
    ms.assetid="0eafb464-810a-4539-905e-8c91e5f3c09e"
    ms.service="resiliency"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="08/18/2016"
    wacn.date="02/20/2017"
    ms.author="aglick" />  

# Azure 复原技术指南

## 介绍

满足高可用性和灾难恢复要求需要掌握两种类型的知识：

* 掌握有关云平台功能的详细技术知识
* 掌握如何正确构建分布式服务的知识

本系列文章涵盖了前一方面：Azure 平台关于复原（有时称为业务连续性）的功能和限制。如果对后者感兴趣，请参阅侧重于 [Azure 应用程序的灾难恢复和高可用性](/documentation/articles/resiliency-disaster-recovery-high-availability-azure-applications/)的系列文章。尽管本系列文章提及了体系结构和设计模式，但并未重点介绍。有关设计指南，可以参阅[其他资源](#additional-resources)部分中的资料。

以下文章中涵盖的信息包括：

* [从本地故障中恢复](/documentation/articles/resiliency-technical-guidance-recovery-local-failures/)。
物理硬件（例如，驱动器、服务器和网络设备）可能失败。负载达到高峰时可能会耗尽资源。本文介绍了 Azure 提供在这些情况下可保持高可用性的功能。

* [从 Azure 区域范围的服务中断中恢复](/documentation/articles/resiliency-technical-guidance-recovery-loss-azure-region/)。
尽管大范围故障很少见，但从理论上而言可能出现这种情况。由于网络故障，整个区域可能会被隔离，或者可能在自然灾难中发生物理损坏。本文介绍了如何使用 Azure 创建可跨越不同地理区域的应用程序。

* [从本地恢复到 Azure](/documentation/articles/resiliency-technical-guidance-recovery-on-premises-azure/)。
云显著改善了灾难恢复的经济效益，使组织能够利用 Azure 来建立用于恢复的辅助站点。你只需牺牲构建和维护辅助数据中心的一小部分成本就可执行此操作。本文介绍了 Azure 提供用于将本地数据中心扩展到云的功能。

* [从数据损坏或意外删除中恢复](/documentation/articles/resiliency-technical-guidance-recovery-data-corruption/)。
应用程序可能含有会损坏数据的 bug。操作者可能错误地删除重要数据。本文介绍了 Azure 提供的用于备份数据并将其还原到之前的时间点的功能。

<a id="additional-resources"></a>
## 其他资源

* [构建在 Azure 基础之上的应用程序灾难恢复和高可用性](/documentation/articles/resiliency-disaster-recovery-high-availability-azure-applications/)。
本文详细概述了可用性和灾难恢复。介绍了手动复制引用和事务数据的难题。文章最后提供了不同类型的灾难恢复拓扑摘要，该拓扑可跨越 Azure 区域实现最高级别的可用性。

* [高可用性清单](/documentation/articles/resiliency-high-availability-checklist/)。
本文列出了所有可帮助你提高应用程序复原能力和可用性的功能、服务和设计。

* [Azure 服务复原指南](/documentation/articles/resiliency-service-guidance-index/)。
本文是 Azure 服务的索引并提供了灾难恢复指南和设计指南的链接。

* [概述：云业务连续性与使用 SQL 数据库进行数据库灾难恢复](/documentation/articles/sql-database-business-continuity/)。
本文提供了有关可用性的 Azure SQL 数据库技术。着重介绍了备份和还原策略。如果在云服务中使用 Azure SQL 数据库，则应查看本文及其相关资源。

* [Azure 虚拟机中 SQL Server 的高可用性和灾难恢复](/documentation/articles/virtual-machines-windows-sql-high-availability-dr/)。
本文介绍了使用服务架构 (IaaS) 托管数据库服务时你可浏览的可用选项。还介绍了 AlwaysOn 可用性组、数据库镜像、日志传送和备份/还原。多个教程演示了使用这些技术的方法。

* [在 Azure 云服务上设计大规模服务的最佳做法](https://azure.microsoft.com//blog/best-practices-for-designing-large-scale-services-on-windows-azure/)。
本文重点介绍了如何开发高度可缩放的云体系结构。许多用于提高可伸缩性的技术还能提高可用性。此外，如果应用程序无法在负载增加的情况下进行缩放，可伸缩性便会成为可用性问题。

* [Azure 虚拟机中 SQL Server 的备份和还原](/documentation/articles/virtual-machines-windows-sql-backup-recovery/)。
本文提供了有关如何备份和还原在 Azure 虚拟机上运行的 Microsoft SQL Server 的技术指南。

<!-- [Failsafe: Guidance for resilient cloud architectures（故障防护：弹性云体系结构指南）。](https://channel9.msdn.com/Series/FailSafe)
本文提供了有关构建弹性云体系结构的指南、利用 Microsoft 技术实现这些体系结构的指南以及实现特定方案的体系结构方法。-->

* [Technical case study: Using cloud technologies to improve disaster recovery（技术案例研究：使用云技术改进灾难恢复）。](https://www.microsoft.com/itshowcase/Article/Content/737/Using-cloud-technologies-to-improve-disaster-recovery)
本案例研究演示了 Microsoft IT 人员如何使用 Azure 改进灾难恢复。

## 后续步骤

本文是侧重于 Azure 复原指南的系列教程的一部分。如果想要阅读本系列教程中的其他文章，可以首先查看[从本地故障中恢复](/documentation/articles/resiliency-technical-guidance-recovery-local-failures/)。

<!---HONumber=Mooncake_0213_2017-->
<!--Update_Description: update meta properties; wroding update; -->