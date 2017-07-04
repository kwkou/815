<properties
    pageTitle="什么是 Site Recovery？ | Azure"
    description="概述了 Azure Site Recovery 服务并汇总了各种部署方案。"
    documentationcenter=""
    author="rayne-wiselman"
    manager="cfreeman"
    editor="" />
<tags
    ms.assetid="e9b97b00-0c92-4970-ae92-5166a4d43b68"
    ms.service="site-recovery"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="na"
    ms.workload="storage-backup-recovery"
    ms.date="03/14/2017"
    wacn.date="03/31/2017"
    ms.author="raynew" />

# 什么是 Site Recovery？

欢迎使用 Azure Site Recovery 服务！ 本文提供了该服务的快速概述。

自然事件和操作失败会导致服务中断。你的组织需要制定业务连续性和灾难恢复 (BCDR) 策略，以确保在计划内和计划外停机期间使数据保持安全、使应用保持可用且使业务尽快恢复到正常运行状况。

Azure 恢复服务是你的 BCDR 策略的一部分。[Azure 备份](/home/features/back-up/)服务确保你的数据安全且可恢复。Site Recovery 将对工作负荷进行复制、故障转移和恢复，以便它们在发生故障时保持可用。

## Site Recovery 的功能是什么？

- **云中的灾难恢复** - 你可以将在 VM 和物理服务器上运行的工作负荷复制到 Azure 而非复制到辅助站点。这就消除了维护辅助数据中心时面临的成本和复杂性。
- **针对混合环境的灵活复制** - 你可以复制在受支持的本地 Hyper-V VM、Windows/Linux 物理服务器上运行的任何工作负荷。
- **迁移** - 你可以使用 Site Recovery 将本地 AWS 实例迁移到 Azure VM，或者在 Azure 区域之间迁移 Azure VM。
- **简化的 BCDR** - 你可以从 Azure 门户中的单个位置部署复制。你可以针对单台计算机和多台计算机运行简单的故障转移和故障回复。
- **恢复能力** - Site Recovery 在不截获应用程序数据的情况下安排复制和故障转移。复制的数据可以存储在 Azure 存储中，可充分利用其提供的恢复能力。发生故障转移时，将基于复制的数据创建 Azure VM。
- **复制性能** - Site Recovery 对 Hyper-V 的复制频率最低可为 30 秒。你可以设置恢复点目标 (RPO) 阈值来控制以何频率创建数据恢复点，并且你可以通过 Site Recovery 的自动化恢复流程以及与 [Azure 流量管理器](https://azure.microsoft.com/zh-cn/blog/reduce-rto-by-using-azure-traffic-manager-with-azure-site-recovery/)的集成降低恢复时间目标 (RTO)。
- **应用程序一致性** - 使用应用程序一致快照进行计算机复制。除了捕获磁盘数据以外，应用程序一致的快照还捕获内存中的所有数据以及流程中的所有事务。
- **无中断地进行测试** - 可以轻松运行测试故障转移来支持灾难恢复演练，且不会影响生产环境。
- **灵活的故障转移和恢复** - 可以运行计划的故障转移，因为是预期的中断，所以不会丢失任何数据；或是运行非计划的故障转移，以在发生非预期的灾难时将数据损失减到最少（取决于复制频率）。当主站点重新可用时，可以轻松地故障回复到主站点。
- **自定义恢复计划** - 使用恢复计划可以创建模型，自定义故障转移和恢复分散在多个 VM 中的多层应用程序。你可以对计划内的组进行排序，以及添加脚本和手动操作。恢复计划可以与 Azure 自动化 runbook 进行集成。
- **多层应用** - 你可以创建恢复计划来针对多层应用进行有序的故障转移和恢复。你可以在恢复计划中将计算机分组到不同的层中（例如数据库、Web、应用），并自定义每个组的故障转移和启动方式。
* **与现有 BCDR 技术的集成** - Site Recovery 与其他 BCDR 技术集成。例如，可以使用 Site Recovery 保护企业工作负荷的 SQL Server 后端，包括本机支持 SQL Server AlwaysOn 以便管理可用性组的故障转移。
* **与自动化库的集成** - 丰富的 Azure 自动化库，提供特定于应用程序的生产就绪型脚本，可以下载并与 Site Recovery 集成。
* **简单的网络管理** — Site Recovery 和 Azure 中的高级网络管理简化了应用程序网络要求，包括保留 IP 地址、配置负载均衡器，以及集成 Azure 流量管理器来提高网络切换的效率。


## 支持的功能

**支持** | **详细信息**
--- | ---
**Site Recovery 支持哪些区域？** | [支持的区域](/support/service-dashboard/) |
**我可以复制哪些内容？** | 本地 Hyper-V VM、Windows 和 Linux 物理服务器。
**复制的计算机需要什么操作系统？** | 对于 Hyper-V VM，Azure 和 Hyper-V 支持的任何[来宾 OS](https://technet.microsoft.com/zh-cn/windows-server-docs/compute/hyper-v/supported-windows-guest-operating-systems-for-hyper-v-on-windows)都受支持。<br/><br/> 适用于物理服务器的[操作系统](/documentation/articles/site-recovery-support-matrix-to-azure/#support-for-replicated-machine-os-versions)
**可以复制到何处？** | 可以复制到 Azure 存储，或复制到辅助数据中心<br/><br/> 对于 Hyper-V，只有 System Center VMM 云中托管的 Hyper-V 主机上的 VM 可以复制到辅助数据中心。
**我可以复制哪些工作负荷** | 你可以复制在受支持的复制计算机上运行的任何工作负荷。此外，Site Recovery 团队还针对[许多应用](/documentation/articles/site-recovery-workload/#workload-summary)执行了特定于应用的测试。


## 哪一种 Azure 门户？

* Site Recovery 可以部署在 [Azure 经典管理门户](https://manage.windowsazure.cn/)中。
* 在 Azure 经典管理门户中，可以同时支持 Site Recovery 和经典服务管理模型。
* 经典管理门户仅应当用来维护现有的 Site Recovery 部署。无法在经典管理门户中创建新的保管库。

## 后续步骤
* 阅读有关[工作负荷支持](/documentation/articles/site-recovery-workload/)的详细信息
* 详细了解 [Site Recovery 体系结构和组件](/documentation/articles/site-recovery-components/)

<!---HONumber=Mooncake_0327_2017-->
<!--Update_Description: content structure simplify-->