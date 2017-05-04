<properties
    pageTitle="了解 Azure Active Directory 体系结构 | Azure"
    description="介绍什么是 Azure AD 租户，以及如何通过 Azure Active Directory 管理 Azure。"
    services="active-directory"
    documentationcenter=""
    author="markvi"
    writer="v-lorisc"
    manager="femila"
    translationtype="Human Translation" />
<tags
    ms.assetid=""
    ms.service="active-directory"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.date="03/01/2017"
    wacn.date="05/02/2017"
    ms.author="markvi"
    ms.sourcegitcommit="78da854d58905bc82228bcbff1de0fcfbc12d5ac"
    ms.openlocfilehash="eac5f2a1374cd98f955d863bd042251cbd66ba0b"
    ms.lasthandoff="04/22/2017" />

# <a name="understand-azure-active-directory-architecture"></a>了解 Azure Active Directory 体系结构
使用 Azure Active Directory (Azure AD) 可以安全地管理用户对 Azure 服务和资源的访问。 Azure AD 随附了整套标识管理功能。 有关 Azure AD 功能的信息，请参阅[什么是 Azure Active Directory？](/documentation/articles/active-directory-whatis/)

在 Azure AD 中可以创建及管理用户和组，并使用权限来允许和拒绝对企业资源的访问。 有关标识管理的信息，请参阅 [Azure 标识管理基础知识](/documentation/articles/fundamentals-identity/)。

## <a name="azure-ad-architecture"></a>Azure AD 体系结构
Azure AD 的地理分布式体系结构整合了全面监视、自动重新路由、故障转移和恢复功能，使我们能够为客户提供企业级的可用性与性能。

本文介绍以下体系结构要素：
 -    服务体系结构设计
 -    可伸缩性 
 -    连续可用性
 -    数据中心

### <a name="service-architecture-design"></a>服务体系结构设计
构建可缩放、高度可用且数据丰富的系统的最常见方法是使用 Azure AD 数据层的独立构建基块或缩放单位。缩放单位称为*分区*。 

数据层包含多个可提供读写功能的前端服务。 下图显示了单目录分区的组件在整个地理分布式数据中心内的分布方式。 

  ![单目录分区](./media/active-directory-architecture/active-directory-architecture.png)

Azure AD 体系结构的组件包括主副本和辅助副本。

**主副本**

*主副本*接收它所属的分区的所有*写入*操作。 在向调用方返回成功消息之前，任何写入操作将立即复制到不同数据中心内的辅助副本，从而确保写入操作具有异地冗余的持久性。

**辅助副本**

所有目录*读取*操作将通过物理分散在不同地理区域的数据中心内的*辅助副本*提供服务。 由于数据是以异步方式复制的，因此存在许多辅助副本。 目录读取操作（例如身份验证请求）将通过靠近客户的数据中心提供服务。 辅助副本负责提供读取可伸缩性。

### <a name="scalability"></a>可伸缩性

可伸缩性是指服务根据不断提高的性能需求进行扩展的能力。 将数据分区可实现写入可伸缩性。 若要实现读取可伸缩性，可将数据从一个分区复制到分发在世界各地的多个辅助副本。

来自目录应用程序的请求通常会路由到它们在物理上最靠近的数据中心。 写入操作将以透明方式重定向到主副本，以提供读写一致性。 由于在大多数情况下目录通常为读取操作提供服务，因此辅助副本可以大幅扩展分区的规模。

目录应用程序连接到最靠近的数据中心。 这可以改善性能，因此可实现扩展。 由于一个目录分区可以包含许多辅助副本，因此，可将辅助副本放置在比较靠近目录客户端的位置。 只有写入密集型的内部目录服务组件才直接以活动的主副本为目标。

### <a name="continuous-availability"></a>连续可用性

可用性（或运行时间）是指系统无中断运行的能力。 Azure AD 提供高可用性的重要原因是，我们的服务可跨多个地理分散的数据中心快速转移流量。 数据中心彼此独立，因此可以实现互不相干的故障模式。

与企业 AD 设计相比，Azure AD 的分区设计更精简，这个特点对于系统扩展至关重要。 我们采用了单一主控设计，其中融入了精心协调的确定性主副本故障转移流程。

**容错**

如果系统能够承受硬件、网络和软件故障，则可用性更高。 目录的每个分区中有一个高度可用的主控副本：主副本。 此副本中只执行针对分区的写入。 此副本持续受到密切的监视，一旦检测到故障，可立即将写入操作转移到其他副本（该副本将变成新的主副本）。 故障转移期间，通常会出现 1-2 分钟的写入可用性损失。 读取可用性在此期间不受影响。

读取操作（比写入操作要多出许多个量级）只会转到辅助副本。 由于辅助副本是幂等的，因此，通过将读取操作定向到其他副本（通常在同一数据中心内），即可轻松补偿给定分区中发生的任一副本丢失。

**数据持久性**

在确认某个写入操作之前，会持续将该操作提交到至少两个数据中心。 首先会将写入操作提交到主副本，然后立即将写入操作复制到其他至少一个数据中心。 这可以确保托管主副本的数据中心发生灾难性损失时不会导致数据丢失。

对于令牌颁发和目录读取，Azure AD 维持零[恢复时间目标 (RTO)](https://en.wikipedia.org/wiki/Recovery_time_objective)；对于目录写入，则维持分钟级的 RTO（大约 5 分钟）。 我们还维持零[恢复点目标 (RPO)](https://en.wikipedia.org/wiki/Recovery_point_objective)，确保故障转移期间不会丢失数据。

### <a name="data-centers"></a>数据中心

Azure AD 的副本存储在分布于世界各地的数据中心内。 有关详细信息，请参阅 [Azure 数据中心](https://azure.microsoft.com/en-us/overview/datacenters)。

Azure AD 可跨数据中心运行，其特征如下：

 - 身份验证、Graph 其他 AD 服务驻留在网关服务的后面。 网关管理这些服务的负载均衡。 如果使用事务运行状况探测检测到任何不正常的服务器，网关将自动故障转移。 网关根据这些运行状况探测，将流量动态路由到正常的数据中心。
 - 对于*读取*操作，目录提供辅助副本以及在多个数据中心运行的、采用主动-主动配置的相应前端服务。 当整个数据中心发生故障时，流量将自动路由到其他数据中心。
 -    对于*写入*操作，目录将通过计划的（将新的主副本同步到旧的主副本）或紧急故障转移过程，跨数据中心故障转移主（主控）副本。 通过将所有提交项复制到至少两个数据中心来实现数据持久性。

**数据一致性**

目录模型具备最终一致性。 分布式异步复制系统的一个典型问题是，从“特定”副本返回的数据可能不是最新的。 

Azure AD 为面向辅助副本的应用程序提供读写一致性，为此，它会将写入操作路由到主副本，然后以异步方式将这些写入操作拉回到辅助副本。

使用 Azure AD 图形 API 的应用程序写入操作经过抽象化，可与目录副本保持相关性，实现读写一致性。 Azure AD Graph 服务维护一个逻辑会话，该会话与用于读取的辅助副本相关；相关性在 Graph 服务使用分布式缓存缓存的“副本令牌”中捕获。 然后，此令牌可用于同一个逻辑会话中的后续操作。 

 >[AZURE.NOTE]
 >写入操作立即复制到逻辑会话读取操作所颁发到的辅助副本。
 >

**备份保护**

目录为用户和租户实施软删除而不是硬删除，让客户在意外删除数据后轻松恢复。 如果租户管理员意外删除了用户，他们可以轻松撤消操作并还原已删除的用户。 

Azure AD 实施所有数据的每日备份，因此，在发生任何逻辑删除或损坏时，能够可靠地恢复数据。 数据层采用纠错代码，可以检查错误并自动更正特定类型的磁盘错误。

**指标和监视器**

运行高可用性服务需要一流的指标和监视功能。 Azure AD 会持续分析和报告其每个服务的关键服务运行状况指标与成功条件。 我们将不断开发并优化指标，针对每个 Azure AD 服务和所有服务中的每个情景进行监视并发出警报。

如果有任何 Azure AD 服务不按预期工作，我们将立即采取措施，尽快还原功能。 Azure AD 跟踪的最重要指标是我们能够以多快的速度检测和缓解客户或实时站点的问题。 我们在监视和警报功能方面投入了大量资金，力求将检测时间缩到最短（TTD 目标：小于 5 分钟）；在操作就绪性方面也同样如此，力求将缓解时间缩到最短（TTM 目标：小于 30 分钟）。

**安全操作**

我们针对任一操作采用多重身份验证 (MFA) 等操作控制，并针对所有操作实施审核。 此外，我们使用适时提升系统，授予必要的临时访问权限让客户完成任何日常的按需操作任务。 有关详细信息，请参阅 [The Trusted Cloud](https://azure.microsoft.com/en-us/support/trust-center)（受信任的云）。

## <a name="next-steps"></a>后续步骤
[Azure Active Directory 开发人员指南](/documentation/articles/active-directory-developers-guide/)

