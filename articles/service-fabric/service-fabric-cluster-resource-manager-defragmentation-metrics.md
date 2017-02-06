<properties
   pageTitle="Azure Service Fabric 中的指标重整 | Azure"
   description="概述如何对 Service Fabric 中的指标使用重整或打包作为策略"
   services="service-fabric"
   documentationCenter=".net"
   authors="masnider"
   manager="timlt"
   editor=""/>  


<tags
   ms.service="Service-Fabric"
   ms.devlang="dotnet"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA"
   ms.date="08/19/2016"
   wacn.date="01/25/2017"
   ms.author="masnider"/>  


# Service Fabric 中指标和负载的重整
Service Fabric 群集资源管理器主要与分布负载方面的平衡有关 - 确保均衡使用群集中的所有节点。若要幸免于故障，这通常是最安全且最明智的布局，因为它可确保任何给定的故障不会导致给定的工作负荷大部分失效。Service Fabric 群集资源管理器还支持另一种策略 - 重整。重整通常意味着我们应实际尝试合并指标，而不是尝试将指标的使用分布到群集中。这是我们一般策略的幸运反转 – 我们通过增加偏差来开始优化，而不是针对给定指标将指标负载的平均标准偏差最小化，来优化群集。但我们为什么要使用此策略？

如果你已在群集中的节点之间平均分散负载，则已消耗了节点必须提供的某些资源。通常这不会产生问题，但有时某些工作负荷会创建特别大型的服务并消耗绝大多数的节点 – 假设 75% 到 95% 的节点资源最终专用于单个服务实例或副本。这不是问题，群集资源管理器将在创建服务时检测是否需要重组群集，以便为此大型工作负荷腾出空间，并着手实现该创建操作，但同时，工作负荷必须等待安排到群集中。

既然计划新工作负荷通常至少需要在一定程度上注意到延迟问题，如果我们未采取一些不同的措施，并且需要移动大量的服务和状态，则有时可能会违反 SLA，特别是群集中的工作负荷很大，因此需要较长时间在群集中移动时。事实上，当我们在模拟场景中根据真实群集数据来测量创建时间时，发现如果服务够大且妥善利用群集，则创建这些大型服务的速度将会变慢，而我们可以引入重整指标策略来改善这一点。

举例来说，如果计算机硬盘中有大量碎片，则创建或访问文件的速度就会变慢，对磁盘进行碎片整理、腾出更大的连续块，即可加快速度。同理，可以配置重整指标，让群集资源管理器主动尝试将服务的负载压缩到较少数的节点，这样，（几乎）永远都有空间可供极大型服务使用，使它们能够快速创建。大多数人并不需要这样做，因为服务通常很小，因此不难为它们找到空间；但如果服务很大且需要快速创建它们（并且愿意接受其他代价，例如较高的失败影响度，以及某些资源在等待计划工作负荷时未被使用），则重整策略就很合适。

下图提供了两个不同群集的视觉表示，其中一个已经过重整，另一个则没有经过重整。在平衡的情况下，如果要创建一个新群集，相比于已重整的群集（可能立即放置在节点 4 或 5），请考虑是否需要通过移动来放置最大服务对象之一。

![平衡的群集与重整的群集对比][Image1]

## 重整的优点和缺点
从概念上讲，重整有其他哪些利弊呢？ 我们建议你先彻底度量工作负荷，然后再启用重整指标。下面是要注意的事项一览表：

| 重整的优点 | 重整的缺点 |
|----------------------|----------------------|
|能够更快地创建大型服务 |	将负载集中到更少的节点，增大资源争用
|在创建期间实现较少的数据移动 | 故障可能影响更多服务，并导致更多的服务流动
|能够丰富描述要求和空间的回收 |	更复杂的整体资源管理配置

你可以在同一个群集中混合重整和一般指标，资源管理器将尽力确保实现以下布局：当你尝试分散到其他位置时，尽可能合并最多数量的重整指标。获得的确切结果取决于相比于重整指标数量的均衡指标数量及其权重、当前负载等因素。

## 配置重整指标
配置重整指标是群集中的全局决策，你可以选择单个指标进行重整：

ClusterManifest.xml：


	<Section Name="DefragmentationMetrics">
	    <Parameter Name="Disk" Value="true" />
	    <Parameter Name="CPU" Value="false" />
	</Section>


## 后续步骤
- 群集资源管理器提供许多用于描述群集的选项。若要详细了解这些选项，请查看这篇有关[描述 Service Fabric 群集](/documentation/articles/service-fabric-cluster-resource-manager-cluster-description/)的文章
- 指标是 Service Fabric 群集资源管理器在群集中管理消耗和容量的方式。若要详细了解指标及其配置方式，请查看[此文](/documentation/articles/service-fabric-cluster-resource-manager-metrics/)

[Image1]: ./media/service-fabric-cluster-resource-manager-defragmentation-metrics/balancing-defrag-compared.png

<!---HONumber=Mooncake_Quality_Review_0125_2017-->