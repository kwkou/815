<properties
    pageTitle="有关 Azure Service Fabric 的常见问题 | Azure"
    description="有关 Service Fabric 的常见问题及其回答"
    services="service-fabric"
    documentationcenter=".net"
    author="seanmck"
    manager="timlt"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="5a179703-ff0c-4b8e-98cd-377253295d12"
    ms.service="service-fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="03/08/2017"
    wacn.date="04/24/2017"
    ms.author="seanmck"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="9a254cde6944c0d6754280debfc9d41d4780108e"
    ms.lasthandoff="04/14/2017" />

# <a name="commonly-asked-service-fabric-questions"></a>Service Fabric 常见问题

有许多关于 Service Fabric 可以做些什么以及应该如何使用的常见问题。 本文档介绍其中许多常见问题及其答案。

## <a name="cluster-setup-and-management"></a>群集设置和管理

### <a name="can-i-create-a-cluster-that-spans-multiple-azure-regions"></a>是否可以创建跨多个 Azure 区域的群集？

目前不可以，但这是常见请求，我们会继续考察。

核心 Service Fabric 群集技术与 Azure 区域无关，只要在世界各地运行的计算机通过网络互相连接，就可通过 Service Fabric 组合使用这些计算机。 但是，Azure 中的 Service Fabric 群集资源具有区域性，在其中生成群集的虚拟机规模集也是如此。 此外，在相距较远的计算机之间提供具有强一致性的数据复制本身就是一个挑战。 我们想要先确保性能可预测且可接受，然后再支持跨区域群集。

### <a name="do-service-fabric-nodes-automatically-receive-os-updates"></a>Service Fabric 节点是否会自动接收操作系统更新？

目前不会，但这也是常见请求，我们打算满足这一请求。

操作系统更新的挑战在于，它们通常需要重启计算机，而这会导致暂时丢失可用性。 就其自身而言，这不是问题，因为 Service Fabric 会自动将这些服务的流量重定向到其他节点。 但是，如果不在群集间协调操作系统更新，可能会有多个节点同时停机。 此类同时重启可能导致某个服务完全失去可用性，或至少让特定分区（对于有状态服务）失去可用性。

将来，我们将会支持在更新域之间完全自动化的、经过协调的 OS 更新策略，确保即使重新启动或发生其他意外故障，也仍能保持可用性。

在此期间，我们[提供脚本](https://blogs.msdn.microsoft.com/azureservicefabric/2017/01/09/os-patching-for-vms-running-service-fabric/)，群集管理器可使用该脚本，以安全的方式手动启动每个节点的修补程序。

### <a name="can-i-use-large-virtual-scale-sets-in-my-sf-cluster"></a>是否可以在我的 SF 群集中使用大型虚拟规模集？ 

**简短解答** - 否。 

**详细解答** - 尽管大型虚拟规模集 (VMSS) 允许你将 VMSS 扩展到最多 1000 个 VM 实例，但它是通过使用放置组 (PG) 实现的。 容错域 (FD) 和升级域 (UD) 仅在使用 FD 和 UD 来为你的服务副本/服务实例做出放置决策的放置组 Service Fabric 中保持一致。 因为 FD 和 UD 仅在放置组中可比较，因此 SF 无法使用它。 例如，如果 PG1 中的 VM1 具有一个 FD=0 的拓扑，并且 PG2 中的 VM9 具有一个 FD=4 的拓扑，这并不意味着 VM1 和 VM2 在两个不同的硬件机架上，因此在这种情况下 SF 无法使用 FD 值做出放置决策。

当前，大型 VMSS 还有其他问题，例如缺少 level-4 负载均衡支持。



### <a name="what-is-the-minimum-size-of-a-service-fabric-cluster-why-cant-it-be-smaller"></a>Service Fabric 群集的最小大小如何？ 为什么不能更小？

运行生产工作负荷的 Service Fabric 群集支持的最小大小是五个节点。 对于开发/测试方案，我们支持三节点群集。

存在这些最小值的原因在于，Service Fabric 群集运行一组有状态系统服务，其中包括命名服务和故障转移管理器。 这些服务跟踪哪些服务已部署到群集及其当前的托管位置，取决于非常一致性。 而这种非常一致性又取决于能否获取*仲裁*来更新这些服务的状态，其中，仲裁表示给定服务在严格意义上的大多数副本 (N/2 + 1)。

在了解这种背景的前提下，让我们探讨一些可能的群集配置：

**单节点**：此选项无法提供高可用性，因为出于任何原因丢失单个节点就意味着丢失整个群集。

**双节点**：跨两个节点 (N = 2) 部署的服务的仲裁为 2 (2/2 + 1 = 2)。 丢失单个副本时，无法创建仲裁。 由于执行服务升级要求暂时关闭副本，因此这不是一个有用的配置。

**三节点**：包含三个节点 (N = 3)，创建仲裁仍然要求使用两个节点 (3/2 + 1 = 2)。 这意味着，可以丢失单个节点，在这种情况下，仍可保留仲裁。

开发/测试支持三节点群集配置，因为只要不同时发生，就可以安全地执行升级以及从单独的节点故障中恢复。 对于生产工作负荷，必须具有应对此类同时发生的故障的复原能力，因此需要五个节点。

### <a name="can-i-turn-off-my-cluster-at-nightweekends-to-save-costs"></a>是否可以在夜间/周末关闭群集以节约成本？

一般而言，不可以。 Service Fabric 在本地、临时磁盘上存储状态，这意味着如果虚拟机移到其他主机，数据不会随之移动。 在正常操作中，这不是问题，因为新节点会通过其他节点保持最新状态。 但是，如果停止所有节点然后重启，很可能发生的情况是，大部分节点在新主机上启动，导致系统无法恢复。

如果在部署应用程序之前想要创建群集来测试应用程序，我们建议将这些群集动态创建为[持续集成/持续部署管道](/documentation/articles/service-fabric-set-up-continuous-integration/)的一部分。

## <a name="application-design"></a>应用程序设计

### <a name="whats-the-best-way-to-query-data-across-partitions-of-a-reliable-collection"></a>跨可靠集合的分区查询数据的最佳方法是什么？

Reliable Collections 通常已[分区](/documentation/articles/service-fabric-concepts-partitioning/)以支持扩展并提高性能和吞吐量。 这意味着给定服务的状态可能跨数十台或数百台计算机分布。 若要对这个完整的数据集执行操作，有以下几个选项：

- 创建查询其他服务的所有分区的服务，拉入所需的数据。
- 创建可从其他服务的所有分区接收数据的服务。
- 定期从每个服务将数据推送到外部存储。 此方法仅适用于要执行的查询不属于核心业务逻辑的情况。


### <a name="whats-the-best-way-to-query-data-across-my-actors"></a>跨执行组件查询数据的最佳方法是什么？

执行组件设计为独立的状态和计算单元，因此，不建议在运行时对执行组件状态执行广泛查询。 如果需要跨执行组件状态的完整集进行查询，应考虑以下方法之一：

- 将执行组件服务替换为有状态可靠服务，以便所有数据的网络请求数量、执行组件数量和服务中的分区数量。
- 将执行组件设计为定期将状态推送到外部存储，方便查询。 如上所述，此方法仅在运行时行为不需要所执行的查询时才可行。

### <a name="how-much-data-can-i-store-in-a-reliable-collection"></a>可以在可靠集合中存储多少数据？

可靠服务通常已分区，因此，存储量仅受限于群集中的计算机数量以及这些计算机的可用内存量。

例如，假设某个服务中的可靠集合拥有 100 个分区和 3 个副本，存储对象的平均大小为 1 KB。 现在假设群集中有 10 台计算机，每台计算机的内存为 16 GB。 为简单和保守起见，假设操作系统和系统服务、Service Fabric 运行时和服务消耗其中 6 GB 的空间，因此，每台计算机的可用空间为 10 GB，群集的可用空间为 100 GB。

请记住，每个对象必须存储三次（一个主要副本和两个次要副本），满负荷运行时，集合可提供足够的内存来存储约 3.5 亿个对象。 但是，我们建议弹性应对故障域和升级域同时丢失的情况，这需要约 1/3 的容量，因此，上述数字减至约 2.3 亿个。

请注意，此计算还假设：

- 数据的跨分区分布大致均匀，或者你会向群集资源管理器报告负载指标。 默认情况下，Service Fabric 根据副本计数进行负载均衡。 在上述示例中，此操作会将 10 个主要副本和 20 个次要副本放在群集中的每个节点上。 这也适用于跨分区均匀分布的负载。 如果负载不均匀，则必须报告负载，以便资源管理器将较小的副本打包在一起，让较大的副本在单个节点上使用更多内存。

- 讨论的可靠服务是群集中存储状态的唯一服务。 因为已将多个服务部署到群集，所以需要注意每个服务运行和管理其状态所需的资源。

- 群集本身不会变大或缩小。 如果添加更多计算机，Service Fabric 会重新平衡副本以利用新增容量，直到计算机数量超过服务中的分区数量，因为单独的副本不能跨计算机。 相反，如果通过删除计算机来缩小群集的大小，副本会打包得更加紧凑，且总容量也会变小。

### <a name="how-much-data-can-i-store-in-an-actor"></a>可以在执行组件中存储多少数据？

和可靠服务一样，可以在执行组件服务中存储的数据量仅受限于群集中各个节点的总磁盘空间和可用内存。 但是，单独的执行组件在用于封装少量状态和关联的业务逻辑时效率最高。 一般而言，单独的执行组件应具有以千字节为单位的状态。

## <a name="other-questions"></a>其他问题

### <a name="how-does-service-fabric-relate-to-containers"></a>Service Fabric 如何与容器关联？

容器提供打包服务及其依赖项的简单方法，以便它们能够在所有环境中一致地运行并且可在单台计算机上以隔离方式运行。 使用 Service Fabric 可以部署和管理服务，包括[在容器中打包的服务](/documentation/articles/service-fabric-containers-overview/)。

### <a name="are-you-planning-to-open-source-service-fabric"></a>是否打算开放 Service Fabric 的源码？

我们打算在 GitHub 上开放 Reliable Services 和 Reliable Actors 框架的源码，并将接受社区向这些项目的贡献。 有关详细信息，请在这些项目发布后关注 [Service Fabric 博客](https://blogs.msdn.microsoft.com/azureservicefabric/) 。

目前没有计划开放 Service Fabric 运行时的源码。

## <a name="next-steps"></a>后续步骤

- [了解核心 Service Fabric 概念和最佳做法](https://mva.microsoft.com/en-us/training-courses/building-microservices-applications-on-azure-service-fabric-16747?l=tbuZM46yC_5206218965)
<!--Update_Description:add two new Q&A; add anchors to sub titles-->