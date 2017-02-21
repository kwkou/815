<properties
    pageTitle="使用 Azure Service Fabric 群集资源管理器管理指标 | Azure"
    description="了解如何在 Service Fabric 中配置和使用指标。"
    services="service-fabric"
    documentationcenter=".net"
    author="masnider"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="0d622ea6-a7c7-4bef-886b-06e6b85a97fb"
    ms.service="Service-Fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="01/05/2017"
    wacn.date="02/20/2017"
    ms.author="masnider" />  


# 在 Service Fabric 中使用指标管理资源消耗和负载
在 Service Fabric 中，指标是与服务关切的、由群集中的节点提供的资源相关的一般术语。一般而言，指标是要管理的、用于处理服务性能的任何信息。

内存、磁盘、CPU 使用率等信息都是指标的示例。这些是物理指标，即对应于节点上需要管理的物理资源的资源。指标也可以是（并且通常是）逻辑指标。逻辑指标是类似于“MyWorkQueueDepth”、“MessagesToProcess”或“TotalRecords”的信息。逻辑指标由应用程序定义，对应于某种资源消耗量，但应用程序并不真正了解如何测量它。逻辑指标很常见。我们看到用户使用的大多数指标是逻辑指标。主要原因是当今的许多服务是使用托管代码编写的。使用托管代码意味着从主机进程内部看，很难测量和报告单个服务对象的物理资源消耗量。测量和报告自身指标的复杂性也是我们要提供一些现成默认指标的原因。

## 默认指标
假设你想要开始体验，但并不知道要使用哪些资源，或者哪些资源对自己重要。此时，可以实施自己的想法，在不指定任何指标的情况下创建服务。没有任何问题！ Service Fabric 群集资源管理器将为你选择一些简单的指标。在未指定任何指标的情况下，我们使用的默认指标称为 PrimaryCount、ReplicaCount 和 Count。下表显示了其中每个指标有多少个负载与每个服务对象相关联：

| 指标 | 无状态实例负载 | 有状态辅助负载 | 有状态主要负载 |
| --- | --- | --- | --- |
| PrimaryCount |0 |0 |1 |
| ReplicaCount |0 |1 |1 |
| 计数 |1 |1 |1 |

好了，使用这些默认指标会得到什么结果？ 结果是基本工作负荷获得相当不错的工作分布。以下示例演示了创建两个服务时会发生什么情况。第一个服务是有状态服务，它包含三个分区，目标副本集大小为 3。第二个服务是无状态服务，它包含一个分区，实例计数为 3：

<center> 
![包含默认指标的群集布局][Image1] 
</center>

结果如下：

* 有状态服务的主要副本并未堆积在单个节点上
* 同一分区的副本不在同一节点上
* 主副本与辅助副本的总数在群集中分布
* 服务对象的总数平均分配在每个节点上

很好！

在开始运行大量的实际工作负荷之前，一切都很合适：选择的分区方案导致所有分区利用率完全平均的可能性有多高？ 给定服务的负载在一段时间内保持恒定甚至与目前完全相同的可能性有多高？

实际上，使用默认指标就能运行得很好，但这样做通常意味着群集利用率低于预期。这是因为默认指标报告不是自适应的，它假设所有条件都是均等的。在最糟的情况下，只使用默认指标还可能导致过度安排节点，从而造成性能问题。使用自定义指标和动态负载报告，我们可以处理得比较好，接下来将探讨这一点。对于任何繁忙的工作负荷，所有服务始终保持条件均等的几率较低。如果想要最充分地利用群集并避免性能问题，建议考虑使用自定义指标。

## 自定义指标
创建服务时，可根据命名服务实例配置指标。

任何指标都包含一些描述自身的属性：名称、默认负载和权重。

* 指标名称：指标的名称。从资源管理器的角度看，指标名称是群集中指标的唯一标识符。
* 默认负载：默认负载根据服务是无状态还是有状态服务以不同的方式表示。
  * 对于无状态服务，每个指标包含名为 DefaultLoad 的单个属性
  * 对于有状态服务，可以定义：
    * PrimaryDefaultLoad：此服务充当主副本时消耗此指标的默认数量
    * SecondaryDefaultLoad：此服务充当辅助副本时消耗此指标的默认数量
* Weight：指标权重定义此指标对于此服务的重要程度（相对于其他指标）。

如果在定义自定义指标的同时想要使用默认指标，需要显式将重新添加这些默认指标。这是因为，默认指标与自定义指标之间的关系必须明确。例如，也许你对内存消耗或 WorkQueueDepth 的关注程度高于主副本分布。

### 为服务定义指标 - 示例
假设你要配置一个服务用于报告名为“MemoryInMb”的指标，同时，还要使用默认指标。另外，假设你已完成一些测量，知道该服务的主副本通常占用 20 个单位的“MemoryInMb”，而辅助副本则占用 5 个单位。就管理此特定服务的性能而论，你知道内存是最重要的指标，但仍希望主副本达到均衡。均衡主副本是个不错的想法，可以避免某个节点或容错域的丢失对大多数主副本造成影响。除了这些调节以外，你还想要使用默认指标。

可以编写以下代码来创建包含该指标配置的服务：

代码：


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


Powershell：


	New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceTypeName –Stateful -MinReplicaSetSize 2 -TargetReplicaSetSize 3 -PartitionSchemeSingleton –Metric @("Memory,High,20,5”,"PrimaryCount,Medium,1,0”,"ReplicaCount,Low,1,1”,"Count,Low,1,1”)


（提醒：如果只想要使用默认指标，则完全不必要处理指标集合，或者在创建服务时执行任何特殊操作。）

完成简介并分析一个示例后，接下来将更详细地介绍其中的每项设置并探讨其行为。

## 加载
定义指标的整个要点就是表示某种负载。“负载”指给定节点上某个服务实例或副本对给定指标的消耗量。可在创建服务时、创建服务后更新服务时、根据服务对象报告服务时或者执行所有此类操作时配置预期负载。

## 默认负载
默认负载是此服务的每个服务对象（实例或副本）默认消耗的负载量。对于较简单的服务，默认负载是永远不会更新的静态定义，因此将在服务的整个生存期内使用。默认负载非常适合用于简单的容量规划方案，其中需要将特定数量的资源专门用于不同的工作负荷。

群集资源管理器允许有状态服务为其主副本和辅助副本指定不同的默认负载，而无状态服务只能指定一个值。对于有状态服务，主副本和辅助副本的默认负载通常不同，因为副本在每个角色中执行不同类型的工作。例如，主副本通常为读取和写入操作提供服务（承受大部分计算负担），而辅助副本则不是。主副本的默认负载预期高于辅助副本，但实际负载量取决你自己的测量值。

## 动态负载
假设我们已运行服务一段时间。使用某种监视功能后，我们发现：

1. 给定服务的某些分区或实例消耗的资源比其他分区或实例要多
2. 某些服务上的负载会随时间变化。

有很多因素可能会导致出现此类负载波动。服务或分区可能与特定的客户关联，或者它们对应的工作负荷在一天中的不同阶段会发生变化。无论原因是什么，可用于默认负载的数字都不是一定的。为默认负载选择的任何值有时是错误的。这是一个问题，因为错误的默认负载会导致群集资源管理器为服务过度分配资源，或者分配的资源不足。因此，某些节点的利用率过高或过低，而群集资源管理器却认为群集中的负载是均衡的。默认负载仍有作用，因为它们会提供一些信息，但是，在大多数情况下，它们不能完整地反映工作负荷。正因如此，群集资源管理器允许每个服务对象在运行时更新其自身的负载。这称为动态负载报告。

动态负载报告可让副本或实例在其生存期中调整其指标分配/报告的指标负载。未执行任何工作的空闲服务副本或实例通常会报告它使用的给定指标较少。繁忙的副本或实例则报告它们使用的指标较多。

根据副本或实例进行报告可让群集资源管理器重新组织群集中的各个服务对象，确保服务获取所需的资源。繁忙的服务能够有效地从当前空闲或执行较少工作的其他副本或实例“回收”资源。

在 Reliable Service 中动态报告负载的代码如下所示：

代码：


	this.ServicePartition.ReportLoad(new List<LoadMetric> { new LoadMetric("MemoryInMb", 1234), new LoadMetric("metric1", 42) });


服务副本或实例只能针对创建它们时配置使用的指标报告负载。服务可报告的指标列表与创建服务时指定的指标集相同。与服务关联的指标列表也可动态更新。如果服务副本或实例尝试针对当前未配置使用的指标报告负载，Service Fabric 将记录但会忽略该报告。如果同一 API 调用中报告的其他指标有效，将接受并使用这些报告。这是理想的结果，因为可以改善体验。代码可以测量并报告它了解的所有指标，操作员无需更改代码，就能指定和更新该服务的指标配置。例如，管理员或操作员团队可以禁用包含特定服务的错误报告的指标、根据行为重新配置指标的权重，或者仅在通过其他机制部署和验证代码之后才启用新指标。

## 混用默认负载值和动态负载报告
如果默认负载不足够，而系统建议使用动态报告负载，那么，可以同时使用此两者吗？ 能！ 事实上，我们建议使用这种配置。如果在设置默认负载的同时使用动态负载报告，在显示动态报告之前，可将默认负载用作估计值。这是一种很好的配置，因为可让群集资源管理器采取一些措施。创建服务对象时，默认负载可让群集资源管理器将其放在合适的位置。如果未提供默认负载信息，服务放置位置是随机的。稍后显示负载报告时，群集资源管理器几乎始终需要四处移动服务。

让我们沿用前一示例，了解在添加一些自定义指标和动态负载报告时会发生什么情况。在此示例中，我们使用“Memory”作为示例指标。假设最初使用以下命令创建了有状态服务：

Powershell：


	New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceTypeName –Stateful -MinReplicaSetSize 2 -TargetReplicaSetSize 3 -PartitionSchemeSingleton –Metric @("MemoryInMb,High,21,11”,"PrimaryCount,Medium,1,0”,"ReplicaCount,Low,1,1”,"Count,Low,1,1”)


提醒一下，此语法为 \("MetricName, MetricWeight, PrimaryDefaultLoad, SecondaryDefaultLoad"\)。

让我们看一下可能的群集布局情况：

<center> 
![使用默认和自定义指标均衡的群集][Image2] 
</center>

有几个问题值得注意：

* 由于副本或实例在报告自身的负载前使用服务的默认负载，因此我们知道有状态服务的分区 1 中不会动态报告负载
* 分区内的次要副本可以有自己的负载
* 指标总体看上去是均衡的。对于内存，最大负载与最小负载之间的比为 1.75（包含最大负载的节点为 N3，包含最小负载的节点为 N2，28/16 = 1.75）。

仍需解释一些事项：

* 何者确定 1.75 的比率是否合理？ 群集资源管理器如何知道此值是否够好，或者还需要再做些什么？
* 何时发生平衡？
* 内存的权重“很高”是什么意思？

## 指标权重
能够跨不同的服务跟踪同一指标非常重要。群集资源管理器可以通过该视图跟踪群集中的消耗，跨节点均衡消耗，确保节点不会超过容量。但是，服务可以提供不同的视图来反映同一指标的重要性。此外，在包含许多指标和服务的群集中，并非所有指标都有完全均衡的解决方案。群集资源管理器如何应对这种情况？

在没有完美的解答时，群集资源管理器可以使用指标权重来决定如何均衡群集。群集资源管理器还可使用指标权重以不同的方式来均衡特定的服务。指标可以有四个不同的权重级别：零、低、中和高。在考虑是否均衡时，权重为零的指标没有任何意义，但其负载仍会影响到容量等因素。

群集中不同指标权重的实际影响是导致群集资源管理器生成不同的解决方案。指标权重告知群集资源管理器要确保某些指标比其他指标更加重要。在没有完美解决方案的情况下，群集资源管理器可以优先选择能够更有效均衡较高权重指标的解决方案。如果一个服务认为某个指标不太重要，它可能会认为该指标的使用不均衡。这样，另一个服务便可以获得重要的均匀分布。

让我们分析一个负载报告示例，以及不同的指标权重如何在群集中造成不同的分配。在此示例中可以看到，切换指标的相对权重会导致群集资源管理器优先选择不同的解决方案并创建不同的服务排列方式。

<center> 
![指标权重的示例及其对均衡解决方案的影响][Image3] 
</center>

在此示例中，四个不同的服务全都报告两个不同指标 A 和 B 的不同值。在其中一个案例中，所有服务定义 MetricA 最为重要 \(Weight = High\)，而 MetricB 则相对不重要 \(Weight = Low\)。在此案例中，我们看到在群集资源管理器放置这些服务后，MetricA 比 MetricB 更加均衡（标准偏差更小）。在第二个案例中，我们反转指标权重。因此，群集资源管理器可能会交换服务 A 与 B，产生 MetricB 比 MetricA 更加均衡的分配。

### 全局指标权重
如果 ServiceA 将 MetricA 定义为最重要，并且 ServiceB 完全不在意 MetricA，最终使用的实际权重是什么？

实际上，我们会跟踪每个指标的多个权重。第一组权重是每个服务针对指标定义的权重。另一个权重是全局权重，它是根据报告该指标的所有服务计算得出的平均值。群集资源管理器在计算解决方案的评分时，将使用这两个权重。这是因为，必须根据服务自身的优先级来均衡服务，但同时群集在整体上应正确分配。

如果群集资源管理器不在乎全局和局部均衡，会发生什么情况？ 构造全局均衡的解决方案虽然微不足道，但这会导致单个服务的资源分配极为不佳。以下示例显示了用于配置有状态服务的默认指标，以及只考虑全局均衡时会发生什么情况：

<center> 
![全局唯一解决方案的影响][Image4] 
</center>

在最上面的示例中，我们只探讨了全局均衡，群集整体上确实实现了均衡。所有节点具有相同的主副本数和相同的副本总数。不过，如果查看此分配的实际影响，就不是那么理想：丢失任何节点对特定工作负荷带来不成比例的影响，因为主副本被排除在外。例如，如果第一个节点发生故障，圆形服务的三个不同分区的三个主副本全都会丢失。相比之下，另外两个服务（三角形和六边形）的分区丢失一个副本，这不会导致中断（除了必须恢复已关闭的副本）。

在最下面的示例中，群集资源管理器已根据全局均衡和基于服务的均衡来分布副本。在计算解决方案的评分时，群集资源管理器会将大多数权重分配给全局解决方案，将（可配置的）一部分权重分配给单个服务。可以根据针对每个服务配置的指标权重平均值计算全局均衡。可以根据服务自身的定义指标权重均衡每个服务。这可确保服务尽量根据自身的需要在内部实现均衡。因此，如果上述第一个节点发生故障，主副本（和辅助副本）的损失将分布在所有服务的所有分区之间。这种影响对于每个分区都是相同的。

## 后续步骤
- 有关可用于配置服务的其他选项的详细信息，请查看 [Learn about configuring Services](/documentation/articles/service-fabric-cluster-resource-manager-configure-services/)（了解如何配置服务）中提供的其他群集资源管理器配置的相关主题
- 定义碎片整理指标是合并（而不是分散）节点上负载的一种方式。若要了解如何配置重整，请参阅[此文](/documentation/articles/service-fabric-cluster-resource-manager-defragmentation-metrics/)
- 若要了解群集资源管理器如何管理和均衡群集中的负载，请查看有关[均衡负载](/documentation/articles/service-fabric-cluster-resource-manager-balancing/)的文章
- 参阅 [Service Fabric 群集资源管理器简介](/documentation/articles/service-fabric-cluster-resource-manager-introduction/)，帮助自己入门
- 移动成本是向群集资源管理器发出信号，表示移动某些服务比移动其他服务会产生更高成本的方式之一。若要了解有关移动成本的详细信息，请参阅[此文](/documentation/articles/service-fabric-cluster-resource-manager-movement-cost/)

[Image1]: ./media/service-fabric-cluster-resource-manager-metrics/cluster-resource-manager-cluster-layout-with-default-metrics.png
[Image2]: ./media/service-fabric-cluster-resource-manager-metrics/Service-Fabric-Resource-Manager-Dynamic-Load-Reports.png
[Image3]: ./media/service-fabric-cluster-resource-manager-metrics/cluster-resource-manager-metric-weights-impact.png
[Image4]: ./media/service-fabric-cluster-resource-manager-metrics/cluster-resource-manager-global-vs-local-balancing.png

<!---HONumber=Mooncake_0213_2017-->
<!--Update_Description: wording refine-->