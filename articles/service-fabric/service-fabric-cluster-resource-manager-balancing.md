<properties
    pageTitle="使用 Azure Service Fabric 群集资源管理器平衡群集 | Azure"
    description="介绍如何使用 Azure Service Fabric 群集资源管理器平衡群集。"
    services="service-fabric"
    documentationcenter=".net"
    author="masnider"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="030b1465-6616-4c0b-8bc7-24ed47d054c0"
    ms.service="Service-Fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="01/05/2017"
    wacn.date="02/20/2017"
    ms.author="masnider" />  


# 均衡 Service Fabric 群集
Service Fabric 群集资源管理器支持动态负载更改，对节点或服务的添加或删除作出反应，纠正约束冲突，以及重新均衡群集。但是，执行这些操作的频率是什么，以及如何触发这些操作呢？

与均衡相关的第一组控件是一组计时器。这些计时器控制群集资源管理器针对需要处理的事项检查群集状态的频率。共有三种不同的工作类别，每一种都有对应的计时器。它们具有以下特点：

1. 放置 - 此阶段负责放置任何遗漏的有状态副本或无状态实例。这包括两项新服务，可处理已失败的有状态副本或无状态实例。可在此进行删除和丢弃副本或实例。
2. 约束检查 – 此阶段检查并更正系统中不同放置约束（规则）的违规情况。规则示例包括确保节点不超出容量，以及符合服务的放置约束。
3. 均衡 - 此阶段根据针对不同指标配置的所需均衡级别查看是否需要主动式重新均衡。如果需要，则尝试查找群集中更均衡的排列方式。

## 配置群集资源管理器的步骤和计时器
群集资源管理器可以进行的每种不同类型的修复都由不同的计时器控制，控管修复频率。激发每个计时器时，将会计划任务。默认情况下，资源管理器：

* 每隔 1/10 秒扫描其状态并应用更新（例如记录节点处于关闭状态）
* 每隔 1 秒设置放置和约束检查标记
* 每隔 5 秒设置均衡标记。

以下配置信息中对此有所反映：

ClusterManifest.xml：


        <Section Name="PlacementAndLoadBalancing">
            <Parameter Name="PLBRefreshGap" Value="0.1" />
            <Parameter Name="MinPlacementInterval" Value="1.0" />
            <Parameter Name="MinConstraintCheckInterval" Value="1.0" />
            <Parameter Name="MinLoadBalancingInterval" Value="5.0" />
        </Section>

通过用于独立部署的 ClusterConfig.json 或用于 Azure 托管群集的 Template.json：


	"fabricSettings": [
	  {
	    "name": "PlacementAndLoadBalancing",
	    "parameters": [
	      {
	          "name": "PLBRefreshGap",
	          "value": "0.10"
	      },
	      {
	          "name": "MinPlacementInterval",
	          "value": "1.0"
	      },
	      {
	          "name": "MinLoadBalancingInterval",
	          "value": "5.0"
	      }
	    ]
	  }
	]


今天群集资源管理器按顺序一次只执行这些操作中的一个（这就是为什么我们将这些计时器称为“最小间隔”）。例如，群集资源管理器处理挂起的请求，以在均衡群集之前创建服务。根据指定的默认时间间隔所示，群集资源管理器会扫描并检查需要频繁执行的任何操作，因此在每个步骤结束时所做的更改集通常比较小。频繁进行微小更改让群集资源管理器能够对群集中发生的事情快速响应。许多相同类型的事件往往同时发生，因此默认计时器可进行某种批处理。默认情况下，群集资源管理器不扫描群集中数小时内进行的更改或尝试一次处理所有更改。这样会导致大量改动。

群集资源管理器还需要一些其他信息来确定群集是否不均衡。为此，我们设置了其他两项配置：*均衡阈值*和*活动阈值*。

## 均衡阈值
平衡阈值是触发主动式重新平衡的主要控件。MinLoadBalancingInterval 计时器只反应群集资源管理器应检查的频率，并不代表发生了什么情况。均衡阈值以某个特定指标来定义群集的不均衡程度，使资源管理器能够考虑它是否不均衡并触发均衡操作。

均衡阈值根据每个指标定义为群集定义的一部分。有关指标的详细信息，请参阅[此文](/documentation/articles/service-fabric-cluster-resource-manager-metrics/)。

ClusterManifest.xml


    <Section Name="MetricBalancingThresholds">
      <Parameter Name="MetricName1" Value="2"/>
      <Parameter Name="MetricName2" Value="3.5"/>
    </Section>


通过用于独立部署的 ClusterConfig.json 或用于 Azure 托管群集的 Template.json：


	"fabricSettings": [
	  {
	    "name": "MetricBalancingThresholds",
	    "parameters": [
	      {
	          "name": "MetricName1",
	          "value": "2"
	      },
	      {
	          "name": "MetricName2",
	          "value": "3.5"
	      }
	    ]
	  }
	]

指标的平衡阈值是一个比率。如果负载最重的节点的负载量除以负载最轻的节点的负载量超过此数，此群集将被视为不均衡。因此群集资源管理器进行下一次检查时将触发均衡。

<center> 
![均衡阈值示例][Image1]
</center>

在此示例中，每个服务使用一个单位的指标。在最上面的示例中，节点的负载上限为 5，下限为 2。假设此指标的均衡阈值为 3。群集中的比率为 5/2 = 2.5，这小于指定的均衡阈值 3，因此群集被视为均衡。群集资源管理器进行检查时将不会触发均衡。

在最下面的示例中，节点的负载上限为 10，下限为 2（因此比率为 5）。5 大于该指标的指定均衡阈值 3。因此，下一次引发均衡计时器时，将计划运行重新均衡。在此类情况下，系统几乎一定会将某些负载分散到 Node3。Service Fabric 群集资源管理器不使用贪婪法则，因此有些负载也可能被分散到 Node2。这样节点之间的整体差异将缩减到最小，这也是群集资源管理器的目标之一。

<center> 
![均衡阈值示例操作][Image2]
</center>

低于均衡阈值不是直接目标。均衡阈值只是触发机制，告诉群集资源管理器应该查看群集，确定可以做出哪些改进（如果有）。事实上，只是进行均衡搜索不代表任何内容的改动 - 有时群集不均衡，但此情形也无法进行改进。

## 活动阈值
有时，虽然节点相当不均衡，但群集中的负载*总量*很低。负载缺乏可能是暂时性的下降，或是因为群集是新的并且刚刚开始引导。不管是哪种情况，建议不要花费时间来均衡群集，因为实际的收获很少。要使群集得到均衡，将消耗网络和计算资源来进行变动，但不会产生任何绝对差异。为避免此问题，可以使用另一个被称为活动阈值的控制方法。活动阈值可以指定活动的绝对下限。如果没有节点高于此阈值，即使达到均衡阈值，也不触发均衡。

请参阅下图示例。同时假设我们让此指标的均衡阈值保持 3，但现在还有活动阈值 1536。在第一种情况下，根据均衡阈值，群集为不均衡状态，但没有节点符合活动阈值，因此保持现状。在最下面的示例中，Node1 已远远超过活动阈值。因为已超过指标的均衡阈值和活动阈值，因此将计划执行均衡。

<center> 
![活动阈值示例][Image3]
</center>

如同平衡阈值，活动阈值通过群集定义根据每个指标进行定义：

ClusterManifest.xml


    <Section Name="MetricActivityThresholds">
      <Parameter Name="Memory" Value="1536"/>
    </Section>

通过用于独立部署的 ClusterConfig.json 或用于 Azure 托管群集的 Template.json：


	"fabricSettings": [
	  {
	    "name": "MetricActivityThresholds",
	    "parameters": [
	      {
	          "name": "Memory",
	          "value": "1536"
	      }
	    ]
	  }
	]


均衡和活动阈值都绑定到具体指标，只有在同一个指标的均衡阈值和活动阈值都超过时才触发均衡。

## 一起平衡服务
值得一提的是，群集是否不均衡是整个群集范围的决定。但解决这种情况的方法是移动单个服务副本和实例。这种说法很合理，是吗？ 如果内存堆积在某一个节点上，可能是由多个副本或实例造成的。修复不均衡需要移动所有使用不均衡指标的有状态副本或无状态实例。

偶尔也会移动均衡的服务。在有其他不均衡情况的时候，如果该服务的所有指标都已均衡（已经非常完美了），为何还会发生服务四处移动的情况？ 让我们看一看！

假设有四个服务：Service1、Service2、Service3 及 Service4。Service1 针对 Metric1 和 Metric2 指标进行报告、Service2 针对 Metric2 和 Metric3 指标进行报告、Service3 针对 Metric3 和 Metric4 指标进行报告，Service4 针对 Metric99 指标进行报告。你可以看到此处的运行情况。这里形成了一个链条！ 我们的确没有 4 个独立的服务，但我们有许多相关的服务（Service1、Service2 和 Service3）以及一个独立的服务。

<center> 
![一起均衡服务][Image4]
</center>

因此，Metric1 不均衡可能会导致属于 Service3 且不报告 Metric1 的副本或实例四处移动。这种移动通常有限，但是根据 Metric1 究竟有多么不平衡以及需要在群集中进行哪些更改才能更正问题，移动幅度可能变大。可以肯定的是，指标 1、2 或 3 不均衡不会导致 Service4 中的移动。这没有任何意义，因为移动属于 Service4 的副本或实例完全不影响指标 1、2 或 3 的均衡。

群集资源管理器自动找出相关的服务，因为可能添加、删除了服务，或者服务的指标配置发生了更改。例如，在两个均衡轮次之间，Service2 可能已重新配置为删除 Metric2。此更改会中断 Service1 和 Service2 之间的链接。现在有三个相关服务组，而不是两个：

<center> 
![一起均衡服务][Image5]
</center>

## 后续步骤
- 指标是 Service Fabric 群集资源管理器在群集中管理消耗和容量的方式。若要详细了解指标及其配置方式，请查看[此文](/documentation/articles/service-fabric-cluster-resource-manager-metrics/)
- 移动成本是向群集资源管理器发出信号，表示移动某些服务比移动其他服务会产生更高成本的方式之一。有关移动成本的详细信息，请参阅[此文](/documentation/articles/service-fabric-cluster-resource-manager-movement-cost/)
- 群集资源管理器提供多个限制机制，你可以配置这些限制机制，以减慢群集中的流动。这些限制通常不是必要的，但如果需要，可以在[此处](/documentation/articles/service-fabric-cluster-resource-manager-advanced-throttling/)了解其相关信息

[Image1]: ./media/service-fabric-cluster-resource-manager-balancing/cluster-resrouce-manager-balancing-thresholds.png
[Image2]: ./media/service-fabric-cluster-resource-manager-balancing/cluster-resource-manager-balancing-threshold-triggered-results.png
[Image3]: ./media/service-fabric-cluster-resource-manager-balancing/cluster-resource-manager-activity-thresholds.png
[Image4]: ./media/service-fabric-cluster-resource-manager-balancing/cluster-resource-manager-balancing-services-together1.png
[Image5]: ./media/service-fabric-cluster-resource-manager-balancing/cluster-resource-manager-balancing-services-together2.png

<!---HONumber=Mooncake_0213_2017-->
<!--Update_Description: add ClusterConfig.json sample script; wording update -->