<properties
    pageTitle="发生影响 Azure 云服务的 Azure 服务中断时该怎么办 | Azure"
    description="了解发生影响 Azure 云服务的 Azure 服务中断时该怎么办。"
    services="cloud-services"
    documentationcenter=""
    author="mmccrory"
    manager="drewm"
    editor="" />
<tags
    ms.assetid="e52634ab-003d-4f1e-85fa-794f6cd12ce4"
    ms.service="cloud-services"
    ms.workload="cloud-services"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="04/04/2017"
    wacn.date="05/22/2017"
    ms.author="mmccrory"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="8fd60f0e1095add1bff99de28a0b65a8662ce661"
    ms.openlocfilehash="d83395ca8b64679ec0d220fd8b186ba27c0238e7"
    ms.lasthandoff="05/12/2017" />

# <a name="what-to-do-in-the-event-of-an-azure-service-disruption-that-impacts-azure-cloud-services"></a>发生影响 Azure 云服务的 Azure 服务中断时该怎么办
Azure 的同仁兢兢业业，只为确保在任何时候都能提供你需要的服务。但有时候会因为不可抗力的影响，造成服务意外中断。

Azure 为其服务提供服务级别协议 (SLA)，作为运行时间和连接承诺。可以在 [Azure 服务级别协议](/support/legal/sla/)中找到各种 Azure 服务的 SLA。

Azure 已在平台中内置多种功能，用于支持高度可用的应用程序。 有关这些服务的详细信息，请参阅 [Azure 应用程序的灾难恢复和高可用性](/documentation/articles/resiliency-disaster-recovery-high-availability-azure-applications/)。

本文介绍了当整个区域因重大自然灾难或大规模服务中断而发生中断时的真实灾难恢复方案。 这些都是极其罕见的情况，但你还是必须对整个区域发生中断的可能性有所准备。 如果整个区域的服务中断，会暂时无法使用数据的本地冗余副本。 如果启用了异地复制，则会在其他区域额外存储 Azure 存储 blob 和表的三个副本。 如果发生全面性区域中断或发生主要区域无法恢复的灾难，Azure 会将所有 DNS 条目重新映射到异地复制区域。

> [AZURE.NOTE]
> 注意，你对此过程无任何控制权，并且此过程仅适用于数据中心范围的服务中断。 因此，还必须依靠应用程序特有的其他备份方法才能达到最高级别的可用性。 有关详细信息，请参阅[构建在 Microsoft Azure 之上的应用程序灾难恢复和高可用性](/documentation/articles/resiliency-disaster-recovery-high-availability-azure-applications/)。 如果要能够影响自己的故障转移，则可能需要考虑使用[读取访问异地冗余存储 (RA-GRS)](/documentation/articles/storage-redundancy/#read-access-geo-redundant-storage)，这会在其他区域中创建数据的只读副本。
>
>


## <a name="option-1-use-a-backup-deployment-through-azure-traffic-manager"></a>选项 1：通过 Azure 流量管理器使用备份部署
最可靠的灾难恢复解决方案涉及在不同区域维护应用程序的多个部署，然后使用 [Azure 流量管理器](/documentation/articles/traffic-manager-overview/)引导它们之间的流量。 Azure 流量管理器提供多个[路由方法](/documentation/articles/traffic-manager-routing-methods/)，因此可选择使用主/备份模型管理部署或拆分它们之间的流量。

![使用 Azure 流量管理器跨区域平衡 Azure 云服务](./media/cloud-services-disaster-recovery-guidance/using-azure-traffic-manager.png)

若要实现对区域丢失作出最快响应，配置流量管理器的[终结点监视](/documentation/articles/traffic-manager-monitoring/)非常重要。

## <a name="option-2-deploy-your-application-to-a-new-region"></a>选项 2：将应用程序部署到新区域
上一选项中所述的维持多个活动部署会持续产生额外成本。 如果恢复时间目标 (RTO) 足够灵活且你具有原始代码或已编译云服务包，可在另一区域中创建一个应用程序新实例，并更新 DNS 记录以指向新部署。

有关如何创建和部署云服务应用程序的详细信息，请参阅[如何创建和部署云服务](/documentation/articles/cloud-services-how-to-create-deploy-portal/)。

根据应用程序数据源，你可能需要检查应用程序数据源的恢复过程。
  * 有关 Azure 存储数据源，请参阅 [Azure 存储复制](/documentation/articles/storage-redundancy/#read-access-geo-redundant-storage)以了解基于所选复制模型而可用于应用程序的选项。
  * 对于 SQL 数据库源，请阅读[概述：云业务连续性与使用 SQL 数据库进行数据库灾难恢复](/documentation/articles/sql-database-business-continuity/)以了解基于所选复制模型而可用于应用程序的选项。


## <a name="option-3-wait-for-recovery"></a>选项 3：等待恢复
这种情况下，你无需进行任何操作，但是在区域还原前服务不可用。

##<a name="next-steps"></a>后续步骤

若要详细了解如何实现灾难恢复和高可用性策略，请参阅 [Azure 应用程序的灾难恢复和高可用性](/documentation/articles/resiliency-disaster-recovery-high-availability-azure-applications/)。

若要掌握有关云平台功能的详细技术知识，请参阅 [Azure 复原技术指南](/documentation/articles/resiliency-technical-guidance/)。