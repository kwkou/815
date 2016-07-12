<properties
   pageTitle="使用 Azure Service Fabric 群集资源管理器管理指标 | Azure"
   description="了解如何在 Service Fabric 中配置和使用指标。"
   services="service-fabric"
   documentationCenter=".net"
   authors="masnider"
   manager="timlt"
   editor=""/>

<tags
   ms.service="Service-Fabric"
   ms.date="05/20/2016"
   wacn.date="07/04/2016"/>

# 在 Service Fabric 中使用指标管理资源消耗和负载
在 Service Fabric 中，指标是与服务关切的资源相关的一般术语。一般而言，指标是你想要从资源立场进行管理，以便处理服务性能的任何项目。

在上述所有示例中，我们一直在暗中提到指标；内存、磁盘、CPU 使用率等参数都是指标的示例。这些是物理指标，即对应于节点上需要管理的物理资源的资源。指标也可以是逻辑指标，例如应用程序定义的“MyWorkQueueDepth”，它对应于某个级别的资源消耗量（但应用程序并没有真正了解或不知道如何测量它）。事实上，我们看到用户使用的大多数指标是逻辑指标。这是由多种原因造成的，但是最常见的原因是现今我们的许多客户使用托管的代码编写他们的服务，很难从指定的无状态服务实例或有状态服务副本对象测量和报告实际实体资源的消耗量。报告自己的指标的复杂度也是为什么我们要提供现成的一些默认指标。

## 默认指标
假设你只是想要开始体验，但并不知道要使用哪些资源，或者哪些资源对你很重要。此时，你可以实施自己的想法，在不指定任何指标的情况下创建服务。这样就可以了！ 我们将为选择一些指标。如果你未自行指定任何指标，我们将为你使用默认的指标 PrimaryCount、ReplicaCount 和（我们知道有点模糊）Count。下表显示这些指标各有多少负载与每个服务对象关联：

| 度量值 | 无状态实例负载 |	有状态辅助负载 |	有状态主要负载 |
|--------|--------------------------|-------------------------|-----------------------|
| PrimaryCount | 0 |	0 |	1 |
| ReplicaCount | 0 | 1 | 1 |
| 计数 |	1 |	1 |	1 |

好了，使用这些默认指标会得到什么结果？ 结果就是，对于基本工作负荷，你将获得相当不错的工作分布。在以下示例中，我们看看当我们创建一个包含三个分区且目标副本集大小为 3 的有状态服务，以及一个实例计数为 3 的无状态服务时，会发生什么状况 - 你将得到类似于下面的结果！

![包含默认指标的群集布局][Image1]

在此示例中，我们看到：
-	有状态服务的主要副本并未堆积在单个节点上
-	同一分区的副本不在同一节点上
-	群集中主要副本与辅助副本的总数合理分布
-	每个节点上的服务对象（无状态与有状态）总数平均分配

非常好！

这非常适合，直到开始思考：选择的分区分配导致经过一段时间让所有分区完全平均使用的可能性？ 此外，指定服务的负载在一段时间内保持不变的几率为何，或只是现在相同？ 结果，对于任何严重的工作负荷，它实际上相当低，因此如果对于获得群集最大使用量感兴趣，可能想要开始考虑自定义指标。

实际上，你绝对可以使用默认指标，或至少配合静态自定义指标运行，但这么做通常意味着群集使用率低于期望（因为报告不是自适应的）；在最糟的情况下，还可能导致过度安排节点，因而造成性能问题。使用自定义指标和动态负载报告，我们可以处理得比较好。

我们一起来看看后续部分 – 自定义指标和动态负载。

## 自定义指标
我们已经讨论过可以有物理和逻辑指标，并且用户可以定义自己的指标。但是怎么做？ 其实很简单！ 只要在创建服务时配置指标和默认初始负载就大功告成了！ 当你创建服务时，可以为每个命名服务实例配置任何一组指标和默认值。

请注意，当你开始定义自定义指标时，如果希望我们也将默认指标用于平衡服务，则需要显式加回默认指标。这是因为我们希望能清楚了解默认指标与自定义指标之间的关系 – 或许相比于主要分布，你更看重内存消耗量或 WorkQueueDepth。

假设你要配置一个服务，该服务报告名为“内存”的指标（除了默认指标以外）。对于内存，我们假设已完成一些基本测量，并且知道该服务的主副本通常占用 20Mb 的内存，而同一服务的辅助副本则占用 5Mb。你知道就管理此特定服务的性能而论，内存是最重要的指标，但你仍希望主副本达到平衡，某个节点或容错域的丢失才不使用过多的主副本。此外，你也可以使用默认值。

以下是要执行的操作：

代码：

```csharp
StatefulServiceDescription serviceDescription = new StatefulServiceDescription();
StatefulServiceLoadMetricDescription memoryMetric = new StatefulServiceLoadMetricDescription();
memoryMetric.Name = "MemoryInMb";
memoryMetric.PrimaryDefaultLoad = 20;
memoryMetric.SecondaryDefaultLoad = 5;
memoryMetric.Weight = ServiceLoadMetricWeight.High;

StatefulServiceLoadMetricDescription primaryCountMetric = new StatefulServiceLoadMetricDescription();
primaryCountMetric.Name = "PrimaryCount";
primaryCountMetric.PrimaryDefaultLoad = 1;
primaryCountMetric.SecondaryDefaultLoad = 0;
primaryCountMetric.Weight = ServiceLoadMetricWeight.Medium;

StatefulServiceLoadMetricDescription replicaCountMetric = new StatefulServiceLoadMetricDescription();
replicaCountMetric.Name = "ReplicaCount";
replicaCountMetric.PrimaryDefaultLoad = 1;
replicaCountMetric.SecondaryDefaultLoad = 1;
replicaCountMetric.Weight = ServiceLoadMetricWeight.Low;

StatefulServiceLoadMetricDescription totalCountMetric = new StatefulServiceLoadMetricDescription();
totalCountMetric.Name = "Count";
totalCountMetric.PrimaryDefaultLoad = 1;
totalCountMetric.SecondaryDefaultLoad = 1;
totalCountMetric.Weight = ServiceLoadMetricWeight.Low;

serviceDescription.Metrics.Add(memoryMetric);
serviceDescription.Metrics.Add(primaryCountMetric);
serviceDescription.Metrics.Add(replicaCountMetric);
serviceDescription.Metrics.Add(totalCountMetric);

await fabricClient.ServiceManager.CreateServiceAsync(serviceDescription);
```

Powershell：

```posh
New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceTypeName –Stateful -MinReplicaSetSize 2 -TargetReplicaSetSize 3 -PartitionSchemeSingleton –Metric @("Memory,High,20,5”,"PrimaryCount,Medium,1,0”,"ReplicaCount,Low,1,1”,"Count,Low,1,1”)
```

（提醒一下，如果你只想要使用默认指标，则根本不需要处理指标集合。）

我们已演示如何定义自己的指标，接下来我们讨论指标具有的各种属性。我们已向你展示过这些属性，现在让我们来讨论它们的实际含义！ 目前，一个指标可有四个不同的属性：

-	指标名称：这是指标的名称。从资源管理器的立场来看，这是群集中指标的唯一标识符。
-	PrimaryDefaultLoad：此服务将为此指标（充当主要指标）使用的默认负载量
-	SecondaryDefaultLoad：此服务将为此指标（充当辅助指标）使用的默认负载量
-	权重：这是指标对于此服务的重要程度（相对于配置的其他指标）。

## 加载
负载是特定节点上某个服务实例或副本使用多少特定指标的一般概念。

## 默认负载
默认负载是群集资源管理器从实际的服务实例或副本收到任何更新之前，应假设每个服务实例或副本将使用多少负载。对于较简单的服务，这是永远不会动态更新的静态定义，因此将用于服务的生存期。这很适合用于简单的容量规划，因为这正好是我们习惯的做法 – 将某些资源投注于某些工作负荷，好处是至少我们现在是以微服务的心态运行，其中的资源实际上不静态分配到特定工作负荷，而其中的人员也不在决策圈内。

我们允许有状态服务指定其主副本和辅助副本的默认负载 – 实际上对许多服务而言，这些数字都不同，因为主副本和辅助副本运行不同的工作负荷，并且主副本通常可用于读取和写入（以及大多数的计算负荷），而主副本的默认负载大于辅助副本。

现在假设已运行服务一段时间，你已注意到服务的某些实例或副本消耗比其他实例或副本更多的资源，或其消耗量随时间改变 – 它们或许与特定客户关联，或许只是对应于一天内生成变化的工作负荷，例如消息传送流量或股票交易。无论如何，你都会注意到，如果不按照一个有意义的数量进行分隔，就没有“单一数字”可用于此负载。你还注意到，在初始估计结果中“进行分隔”导致 Service Fabric 将过多或过少的资源分配给服务，以致节点的使用量过高或过低。怎么办？ 服务可能正在报告负载！

## 动态负载
动态负载报告可让副本或实例在其生存期中调整其在群集中的分配/报告的指标使用量。空闲且未运行任何工作的服务副本或实例通常会报告它使用少量的资源，而繁忙的副本或实例则报告他们使用较多的资源。群集中的这种常见流动现象可让我们快速重新组织群集中的服务副本和实例，以确保服务和实例获取其所需的资源 – 实际上，繁忙的服务能够从当前空闲或运行较少工作的其他副本或实例回收资源。通过 ReportLoad 方法（可在 ServicePartition 上获取并可作为基本 StatefulService 的属性使用）可以快速完成负载报告。在服务中，代码如下所示：

代码：

```csharp
this.ServicePartition.ReportLoad(new List<LoadMetric> { new LoadMetric("Memory", 1234), new LoadMetric("metric1", 42) });
```

服务副本或实例只能针对它们配置要使用的指标报告负载。创建每个服务时将设置指标列表。如果服务副本或实例尝试针对当前未配置要使用的指标报告负载，Service Fabric 将记录该报告但予以忽略，这意味着我们在计算或报告群集的状态时不会使用该指标。这是理想的结果，因为可以改善体验 – 代码可以测量并报告它知道做法的所有内容，而操作员无需更改代码，就可以快速配置、调整和更新该服务的资源平衡规则。例如，这可能包括禁用有错误报告的指标、根据行为重新设置指标的权重，或仅在部署和验证代码之后启用新的指标。

## 混用默认负载值和动态负载报告
为即将以动态方式报告负载的服务指定默认负载是否合理？ 当然！ 在此情况下，默认负载可用作估计值，直到真正的报告开始从实际的服务副本或实例显示为止。这是适合，因为它可让资源管理器在创建副本或实例时进行放置 – 默认负载最后成为初始估计值，让资源管理器一开始就将服务实例或副本放在适当的位置；如果未提供任何信息，放置方式实际上是随机的，而我们几乎肯定必须在真正的负载报告开始出现时移动项目。

让我们使用前面的示例，看看当我们添加一些自定义负载时会发生什么情况，然后在创建服务之后，它是否会动态更新。在此示例中，我们将以“内存”为例，并假设我们一开始使用以下命令创建有状态服务：

Powershell：

```posh
New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceTypeName –Stateful -MinReplicaSetSize 2 -TargetReplicaSetSize 3 -PartitionSchemeSingleton –Metric @("Memory,High,21,11”,"PrimaryCount,Medium,1,0”,"ReplicaCount,Low,1,1”,"Count,Low,1,1”)
```

我们前面讨论过此语法（MetricName、MetricWeight、PrimaryDefaultLoad、SecondaryDefaultLoad），但我们稍后将详细讨论权重的特定值有何意义。

让我们看一下可能的群集布局情况：

![使用默认和自定义指标平衡的群集][Image2]

有几个问题值得注意：

-	由于副本或实例在报告自己的负载前使用服务的默认负载，我们知道有状态服务的分区 1 内的副本尚未自行报告负载
-	分区内的辅助副本可以有自己的负载
-	整体指标看起来很不错，节点上最大和最小负载之间的差异（对于内存 – 我们说过最在意的自定义指标）比率只有 1.75（具有最多内存负载的节点是 N3，最少是 N2，28/16 = 1.75）– 相当平衡！

仍需解释的一些事项

-	何者确定 1.75 的比率是否合理？ 我们如何知道这是否够好，或者还需要再做些什么？
-	何时发生平衡？
-	内存的权重“很高”是什么意思？

## 指标权重
指标权重允许两个不同的服务报告同一指标，但不能以不同方式看待平衡该指标的重要性。例如，请考虑内存中分析引擎和持续性数据库；两者可能都很在意“内存”指标，但内存中服务可能不太在意“磁盘”指标 – 它可能少量使用，但并非服务性能的关键所在。能够跨越不同的服务来跟踪同一指标真的很棒，因为此功能可让资源管理器跟踪群集中的实际消耗量，确保节点不超过容量，此外还可带来其他优势。

指标权重可让资源管理器在没有完美解答时，真正决定如何平衡群集（这需要花费很长时间）。指标可以有四个不同的权重级别：零、低、中和高。在考虑是否平衡时，权重为“零”的指标没有任何价值，但其负载对于容量测量仍有所价值。

群集中不同指标权重的实际影响是我们达到实质不同的服务排列方式，因为资源管理器被告知要确保某些指标比其他指标更加重要，因此如果这些指标与其他指标（较低优先级的指标）发生冲突，则首选使用这些指标。

让我们看一个简单的负载报告示例，以及不同的指标权重如何在群集中造成不同的分配。在此示例中，我们看到切换指标的相对权重会导致资源管理器通过创建不同的服务排列来首选某些解决方案。

![指标权重示例及其对平衡解决方案的影响][Image3]

在此示例中，有四个不同的服务全部都报告两个不同指标 A 和 B 的不同值。在所有服务定义的方案之一中，指标 A 最为重要（权重 = 高）而 MetricB 则相对不重要（权重 = 低），我们确实看到 Service Fabric 资源管理器放置这些服务，所以 MetricA 比 MetricB 更加平衡（偏差较小）。在第二个方案中，我们反转指标权重，然后将看到资源管理器可能会交换服务 A 与 B，以便产生 MetricB 比 MetricA 更加平衡的分配。

### 全局指标权重
如果 ServiceA 将 MetricA 定义为最重要，并且 ServiceB 完全不在意 MetricA，则资源管理器最终使用的实际权重为何？

我们实际为每个指标跟踪两个权重 – 服务本身定义的权重，以及所有在意该指标的服务所适用的全局平均权重。我们使用这两个权重来计算生成的解决方案的分数，因为必须要确保服务与它自己的优先级达到平衡，且群集整体上分配正确。

如果我们不在意全局和局部平衡，会发生什么状况？ 构造全局平衡的解决方案虽然微不足道，但这会导致单个服务的平衡和资源分配极为不佳。在以下示例中，请考虑用于配置有状态服务的默认指标 PrimaryCount、ReplicaCount 和 TotalCount，并了解当我们只考虑全局平衡时会发生什么情况：

![全局唯一解决方案的影响][Image4]

在最上面的示例中，我们只探讨了全局平衡，群集整体上的确达到平衡 – 所有节点的主副本和总副本的计数都相同，太棒了！ 各项都达到了平衡。不过，如果查看此分配的实际影响，就不是那么理想：丢失任何节点对特定工作负荷带来不成比例的影响，因为这除取其所有的主副本。举例来说，如果我们丢失第一个节点。如果发生这种情形，圆形服务的三个不同分区的三个主副本全部同时丢失。相比之下，另外两个服务（三角形和六边形）的分区丢失一个副本，这不会导致中断（除了必须恢复已关闭的副本）。

在最下面的示例中，我们已根据全局平衡和按服务的平衡来分发副本。在计算解决方案的分数时，我们将大多数的权重提供给全局解决方案，但是（可配置）的部分确保服务本身尽可能平衡。因此，如果我们同样丢失第一个节点，将会看到丢失的主副本（和辅助副本）分布于所有服务的所有分区之间，这对每个分区的影响都是相同的。

将指标权重纳入考虑，可以根据指标权重的平均值计算全局平衡。我们已根据服务自身的定义指标权重平衡了服务。

## 后续步骤
- 有关可用于配置服务的其他选项的详细信息，请查看 [Learn about configuring Services](/documentation/articles/service-fabric-cluster-resource-manager-configure-services/)（了解如何配置服务）中提供的其他群集资源管理器配置的相关主题
- 定义重整指标是合并（而不是分散）节点上负载的一种方式。若要了解如何配置重整，请参阅[此文](/documentation/articles/service-fabric-cluster-resource-manager-defragmentation-metrics/)
- 若要了解群集资源管理器如何管理和平衡群集中的负载，请查看关于[平衡负载](/documentation/articles/service-fabric-cluster-resource-manager-balancing/)的文章
- [获取有关 Service Fabric 群集资源管理器的简介](/documentation/articles/service-fabric-cluster-resource-manager-introduction/)以帮助自己入门
- 移动成本是向群集资源管理器发出信号，表示移动某些服务比移动其他服务会产生更高成本的方式之一。若要了解有关移动成本的详细信息，请参阅[此文](/documentation/articles/service-fabric-cluster-resource-manager-movement-cost/)

[Image1]: ./media/service-fabric-cluster-resource-manager-metrics/cluster-resource-manager-cluster-layout-with-default-metrics.png
[Image2]: ./media/service-fabric-cluster-resource-manager-metrics/Service-Fabric-Resource-Manager-Dynamic-Load-Reports.png
[Image3]: ./media/service-fabric-cluster-resource-manager-metrics/cluster-resource-manager-metric-weights-impact.png
[Image4]: ./media/service-fabric-cluster-resource-manager-metrics/cluster-resource-manager-global-vs-local-balancing.png

<!---HONumber=Mooncake_0627_2016-->