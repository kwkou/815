<properties
    pageTitle="Azure Service Fabric 中的指标碎片整理 | Azure"
    description="概述如何对 Service Fabric 中的指标使用碎片整理或打包作为策略"
    services="service-fabric"
    documentationcenter=".net"
    author="masnider"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="e5ebfae5-c8f7-4d6c-9173-3e22a9730552"
    ms.service="Service-Fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="01/05/2017"
    wacn.date="02/20/2017"
    ms.author="masnider" />  


# Service Fabric 中指标和负载的碎片整理
Service Fabric 群集资源管理器主要与负载分配方面的均衡有关 - 确保均衡使用群集中的节点。若要幸免于故障，让工作负荷分散通常是最安全的布局，因为这可以确保故障不会导致给定的工作负荷大部分失效。Service Fabric 群集资源管理器还支持另一种策略 - 碎片整理。碎片整理通常意味着我们应实际尝试合并指标，而不是尝试将指标的使用分布到群集中。合并是我们一般策略的逆向运用 – 群集资源管理器的目的是增加偏差，而不是将指标负载的平均标准偏差最小化。但为什么要使用此策略？

如果已在群集中的节点之间平均分散负载，则表明已消耗了节点必须提供的某些资源。但是，某些工作负荷创建的服务相当大，占用了节点的大部分资源。在此类情况下，可能会出现一个节点 75% 到 95% 的资源最终都由单个服务对象专用的情况。工作负荷大不是问题。群集资源管理器在创建服务时就会确定需要重新组织群集，为该大型工作负荷腾出空间。但同时，该工作负荷也必须等待，等待系统在群集中对其进行计划。

如果要移动的服务和状态很多，则要将大型工作负荷放置到群集中，可能需要等待很长时间。如果群集中的其他工作负荷很大，因此需花费较长时间来移动，则可能发生这种情况。Service Fabric 团队通过模拟此情形测量了创建时间。我们发现，如果服务很大，且群集处于高度利用状态，则创建这些大型服务会很慢。为了处理这种情形，我们引入了重整作为均衡策略。我们发现，对于大型工作负荷（尤其是创建时间很重要的）来说，重整确实有助于系统在群集中对那些新的工作负荷进行计划。

可以对重整指标进行配置，允许群集资源管理器主动尝试将服务负载压缩到较少的节点中。这可确保始终为大型服务提供空间（大多数情况下）。这样可以根据需要快速创建此类服务。

大多数人无需执行重整操作。服务通常会较小，因此不难为其在群集中找到空间。但是，如果服务为大型服务且需快速创建（而且你愿意接受其缺点），则可使用重整策略。

那么，其缺点是什么呢？ 主要缺点在于，重整可能会加大故障的影响（因为在发生故障的节点上运行的服务数量更多）。此外，重整必定会导致群集中的某些资源在等待系统对工作负荷进行计划时处于未使用状态。

下图提供了两个不同群集的视觉表示形式，其中一个已经过重整，另一个则没有经过重整。在均衡的群集示例中，考虑一下，放置其中一个最大的服务对象需要经过多少次移动。将上述结果与重整的群集进行比较，如果是后者，则可立即将大型工作负荷放置到节点 4 或 5 上面。

<center> 
![均衡的群集与重整的群集对比][Image1] 
</center>

## 碎片整理的优点和缺点
从概念上讲，碎片整理有其他哪些利弊呢？ 我们建议先彻底度量工作负荷，然后再启用碎片整理指标。下面是要注意的事项一览表：

| 碎片整理的优点 | 碎片整理的缺点 |
| --- | --- |
| 能够更快地创建大型服务 |将负载集中到更少的节点，增大资源争用 |
| 在创建期间实现较少的数据移动 |故障可能影响更多服务，并导致更多的服务流动 |
| 能够丰富描述要求和空间的回收 |更复杂的整体资源管理配置 |

可以在同一群集中混合使用重整的指标和正常指标。群集资源管理器会尝试尽可能合并重整指标，分散其他指标。如果没有服务共享上述指标，则结果可能会很好。确切结果取决于均衡指标数与重整指标数的相对比例、其重叠情况、其权重、当前负载，以及其他因素。需要通过试验来确定具体的必需配置。

## 配置碎片整理指标
配置碎片整理指标是群集中的全局决策，可以选择单个指标进行碎片整理：

ClusterManifest.xml：


	<Section Name="DefragmentationMetrics">
	    <Parameter Name="Disk" Value="true" />
	    <Parameter Name="CPU" Value="false" />
	</Section>

通过用于独立部署的 ClusterConfig.json 或用于 Azure 托管群集的 Template.json：


	"fabricSettings": [
	  {
	    "name": "DefragmentationMetrics",
	    "parameters": [
	      {
	          "name": "Disk",
	          "value": "true"
	      },
	      {
	          "name": "CPU",
	          "value": "false"
	      }
	    ]
	  }
	]


## 后续步骤
- 群集资源管理器提供许多用于描述群集的选项。若要详细了解这些选项，请查看这篇有关[描述 Service Fabric 群集](/documentation/articles/service-fabric-cluster-resource-manager-cluster-description/)的文章
- 指标是 Service Fabric 群集资源管理器在群集中管理消耗和容量的方式。若要详细了解指标及其配置方式，请查看[此文](/documentation/articles/service-fabric-cluster-resource-manager-metrics/)

[Image1]: ./media/service-fabric-cluster-resource-manager-defragmentation-metrics/balancing-defrag-compared.png

<!---HONumber=Mooncake_0213_2017-->
<!--Update_Description: enrich introduction for "指标碎片整理"; add ClusterConfig.json script-->