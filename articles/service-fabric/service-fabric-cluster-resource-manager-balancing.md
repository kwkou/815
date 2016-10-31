<properties
   pageTitle="使用 Azure Service Fabric 群集资源管理器平衡群集 | Azure"
   description="介绍如何使用 Azure Service Fabric 群集资源管理器平衡群集。"
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
   wacn.date="10/24/2016"
   ms.author="masnider"/>  


# 均衡 Service Fabric 群集
Service Fabric 群集资源管理器可用于报告动态负载、对群集中的更改做出反应、更正约束违规，以及在必要时重新均衡群集。但是，执行这些操作的频率是什么，以及如何触发这些操作呢？ 有多个与此相关的控件。

与均衡相关的第一组控件是一组计时器。这些计时器控制群集资源管理器针对需要处理的事项检查群集状态的频率。共有三种不同的工作类别，每一种都有对应的计时器。它们具有以下特点：

1.	放置 - 此阶段负责放置任何遗漏的有状态副本或无状态实例。这包括两项新服务，可处理已失败且需要重新创建的有状态副本或无状态实例。删除和丢弃副本或实例的操作也在此处处理。
2.	约束检查 – 此阶段检查并更正系统中不同放置约束（规则）的违规情况。规则示例包括确保节点不超出容量，以及符合服务的放置约束（稍后详细说明）。
3.	平衡 - 此阶段根据针对不同指标配置的所需平衡级别查看是否需要主动式重新平衡，如果需要，则尝试查找群集中更平衡的排列方式。

## 配置群集资源管理器的步骤和计时器
群集资源管理器可以运行的每种不同类型的修复都由不同的计时器控制，控管修复频率。例如，如果只想要处理每小时在群集中放置新的服务工作负荷（进行批处理），但同时想要每隔几秒钟定期均衡检查，则可以配置该行为。激发每个计时器时，将会计划任务。默认情况下，资源管理器每隔 1/10 秒扫描其状态并应用更新（针对所有在上次扫描后发生的更改进行批处理，例如察觉到节点已关闭），每隔一秒设置放置和约束检查标志，每隔 5 秒设置均衡标志。

ClusterManifest.xml：


        <Section Name="PlacementAndLoadBalancing">
            <Parameter Name="PLBRefreshGap" Value="0.1" />
            <Parameter Name="MinPlacementInterval" Value="1.0" />
            <Parameter Name="MinConstraintCheckInterval" Value="1.0" />
            <Parameter Name="MinLoadBalancingInterval" Value="5.0" />
        </Section>


今天我们只按顺序一次执行这些操作中的一个（这就是为什么我们将这些配置称为“最小间隔”）。例如，我们已响应任何挂起的请求，已在继续均衡群集之前创建新副本。如此处所示，可以根据指定的默认时间间隔扫描并检查我们需要非常频繁执行的任何操作，这意味着我们在每个步骤结束时所做的更改集通常比较小：我们不是扫描群集中数小时的更改并尝试一次更正全部，并尝试在事情发生时进行处理，而是在同时有许多事情发生时进行某种批处理。这使得 Service Fabric 资源管理器对于群集中发生的事情保持极高的响应度。

尽管这些任务大多数都相当直接了当（有约束违规就修复它们，有需要创建的服务就创建它们），群集资源管理器还需要一些其他信息来确定群集是否处于不均衡的状态。为此，我们设置了其他两项配置：*均衡阈值*和*活动阈值*。

## 均衡阈值
均衡阈值是触发主动式重新均衡的主要控件（请记住，计时器只能确定群集资源管理器的检查频率，并不代表一定发生任何事情）。均衡阈值以某个特定指标来定义群集的不均衡程度，使资源管理器能够考虑它是否不均衡并触发均衡操作。

均衡阈值根据每个指标定义为群集定义的一部分。（有关指标的详细信息，请参阅[此文](/documentation/articles/service-fabric-cluster-resource-manager-metrics/)）。

ClusterManifest.xml


    <Section Name="MetricBalancingThresholds">
      <Parameter Name="MetricName1" Value="2"/>
      <Parameter Name="MetricName2" Value="3.5"/>
    </Section>


指标的平衡阈值是一个比率。如果负载最重的节点的负载量除以负载最轻的节点的负载量超过此数目，此群集将被视为不均衡，在群集资源管理器下一次检查时将触发均衡（默认情况下为每 5 秒，由 MinLoadBalancingInterval 控制，如上所示）。

![平衡阈值示例][Image1]  


在此简单示例中，每个服务使用一个单位的指标。在最上面的示例中，节点的负载上限为 5，下限为 2。假设此指标的平衡阈值为 3。因此，在最上面的示例中，群集被视为均衡，并且群集资源管理器在检查时不会触发均衡（因为群集中的比率为 5/2 = 2.5，这小于指定的均衡阈值 3）。

在最下面的示例中，节点的负载上限为 10，下限为 2（因此比率为 5）。这样，群集就超过了该指标的设计均衡阈值 3。因此，下一次引发均衡计时器时，将计划运行全局重新均衡。请注意，就算开始均衡搜索，也不代表移动任何对象 - 有时，尽管群集是处于不均衡的状态，但是无法改善 - 但在当前示例中，（至少以默认而言）系统几乎一定将某些负载分散到 Node3。请注意，请注意，我们未使用穷尽方法，因此有些负载也可能被分散到 Node2，因为这会导致节点之间的整体差异缩减到最小，但我们预期多数负载会流向 Node3。

![平衡阈值示例操作][Image2]  


请注意，获取以下均衡阈值不是明确的目标，均衡阈值只是*触发器*，告诉群集资源管理器应该查看群集，确定可以做出哪些改进（如果有）。

## 活动阈值
有时，虽然节点相当不均衡，但群集中的负载*总量*很低。这可能只是因为当天时间关系，或是群集是新的并且刚刚开始引导。在任一情况下，用户可能不想花费时间均衡群集，因为实际的收获很少 – 只是消耗网络和计算资源来移动项目，而不会带来任何绝对差异。由于我们要避免执行此操作，可以使用资源管理器中另一个称为活动阈值的控件指定活动的绝对下限 – 如果没有任何节点有至少这么多的负载，则即使达到均衡阈值，也不触发均衡。

例如，假设在我们的报告中，这些节点的消耗量总计如下所示。同时假设我们让此指标的均衡阈值保持 3，但现在还有活动阈值 1536。在第一种情况下，根据均衡阈值，群集为不均衡状态，没有任何节点符合活动阈值下限，因此我们保持现状。在最下面的示例中，Node1 已远远超过活动阈值，因此将执行均衡（因为已超过指标的均衡阈值和活动阈值）

![活动阈值示例][Image3]  


如同平衡阈值，活动阈值通过群集定义根据每个指标进行定义：

ClusterManifest.xml


    <Section Name="MetricActivityThresholds">
      <Parameter Name="Memory" Value="1536"/>
    </Section>

请注意，均衡和活动阈值都绑定到指标，并且只有在同一个指标的均衡和活动阈值都超过时才触发均衡。因此，如果超过内存的均衡阈值和 CPU 的活动阈值，只要其余阈值未超过（CPU 的均衡阈值和内存的活动阈值），就不触发均衡。

## 一起平衡服务
值得一提的是，群集是否不平衡是群集范围的决策，但解决这种情况的方法是移动单个服务副本和实例。这种说法很合理，是吗？ 如果内存堆积在某一个节点上，则可能是由多个副本或实例造成的，因此需要移动所有使用受影响、不均衡指标的有状态副本或无状态实例。

有时，客户会呼叫我们或[在线申请支持](/support/support-ticket-form/?l=zh-cn)创建工单，指出已移动不平衡的服务。在有其他不平衡情况的时候，如果该服务的所有指标都已平衡（已经非常完美了），为何需要移动服务呢？ 让我们看一看！

假设有四个服务：Service1、Service2、Service3 及 Service4。Service1 针对 Metric1 和 Metric2 指标进行报告、Service2 针对 Metric2 和 Metric3 指标进行报告、Service3 针对 Metric3 和 Metric4 指标进行报告，Service4 针对 Metric99 指标进行报告。你可以看到此处的运行情况。这里形成了一个链条！ 从群集资源管理器的立场来讲，我们的确没有 4 个独立的服务，但我们有许多相关的服务（Service1、Service2 和 Service3）以及一个独立的服务。

![一起平衡服务][Image4]  


因此，Metric1 不均衡可能会导致属于 Service3 的副本或实例四处移动。这种移动通常很有限，但是根据 Metric1 究竟有多么不平衡以及需要在群集中进行哪些更改才能更正问题，移动幅度可能变大。我们还可以肯定地说，指标 1、2 或 3 不平衡绝不会导致 Service4 移动 – 没有必要，因为移动属于 Service4 的副本或实例完全不影响指标 1、2 或 3 的平衡。

群集资源管理器自动找出相关的服务，因为可能添加、删除了服务，或者服务的指标配置发生了更改 – 例如，在两个均衡轮次之间，Service2 可能已重新配置为删除 Metric2。这会中断 Service1 和 Service2 之间的链接。现在你有三个服务组，而不是两个：

![一起平衡服务][Image5]  


## 后续步骤
- 指标是 Service Fabric 群集资源管理器在群集中管理消耗和容量的方式。若要详细了解指标及其配置方式，请查看[此文](/documentation/articles/service-fabric-cluster-resource-manager-metrics/)
- 移动成本是向群集资源管理器发出信号，表示移动某些服务比移动其他服务会产生更高成本的方式之一。若要了解有关移动成本的详细信息，请参阅[此文](/documentation/articles/service-fabric-cluster-resource-manager-movement-cost/)
- 群集资源管理器提供多个限制机制，你可以配置这些限制机制，以减慢群集中的流动。这些限制通常不是必要的，但如果需要，可以在[此处](/documentation/articles/service-fabric-cluster-resource-manager-advanced-throttling/)了解其相关信息


[Image1]: ./media/service-fabric-cluster-resource-manager-balancing/cluster-resrouce-manager-balancing-thresholds.png
[Image2]: ./media/service-fabric-cluster-resource-manager-balancing/cluster-resource-manager-balancing-threshold-triggered-results.png
[Image3]: ./media/service-fabric-cluster-resource-manager-balancing/cluster-resource-manager-activity-thresholds.png
[Image4]: ./media/service-fabric-cluster-resource-manager-balancing/cluster-resource-manager-balancing-services-together1.png
[Image5]: ./media/service-fabric-cluster-resource-manager-balancing/cluster-resource-manager-balancing-services-together2.png

<!---HONumber=Mooncake_1017_2016-->