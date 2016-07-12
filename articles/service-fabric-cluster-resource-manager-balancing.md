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
   ms.date="05/20/2016"
   wacn.date="07/04/2016"/>

# 平衡 Service Fabric 群集
Service Fabric 群集资源管理器可用于报告动态负载、对群集中的更改做出反应，以及生成平衡计划，但何时执行这些操作呢？ 如果服务在创建时根据其默认负载值放置，什么因素在群集中真正触发重新平衡呢？ 有多个与此相关的控件。

第一组控件控制一组计时器的平衡，而这些计时器控制群集资源管理器针对需要处理的事项来检查群集状态的频率。这些计时器与其一直执行的操作的不同工作阶段相关。这些位置包括：

1.	放置 - 此阶段负责放置任何遗漏的有状态副本或无状态实例。这包括两项新服务，可处理已失败且需要重新创建的副本或实例。删除和丢弃副本或实例的操作也在此处处理。
2.	约束检查 – 此阶段检查并更正系统中不同放置约束（规则）的违规情况。规则示例包括确保节点不超出容量，以及符合服务的放置约束（稍后详细说明）。
3.	平衡 - 此阶段根据针对不同指标配置的所需平衡级别查看是否需要主动式重新平衡，如果需要，则尝试查找群集中更平衡的排列方式。

## 配置群集资源管理器的步骤和计时器
上述每个阶段由控管频率的不同计时器控制。例如，如果你只想要处理每小时在群集中放置新的服务工作负荷（以进行批处理），但同时想要每隔几秒钟定期平衡检查，则你可以强制这样做。当每个计时器激发时，将设置一个标志，表示我们需要处理资源管理器该部分的职责，并在下一个整体扫描时通过状态机进行选择（这就是这些配置定义为“最小间隔”的原因）。默认情况下，资源管理器每隔 1/10 秒扫描其状态并应用更新，每隔一秒设置放置和约束检查标志，每隔 5 秒设置平衡标志。

ClusterManifest.xml：

``` xml
        <Section Name="PlacementAndLoadBalancing">
            <Parameter Name="PLBRefreshGap" Value="0.1" />
            <Parameter Name="MinPlacementInterval" Value="1.0" />
            <Parameter Name="MinConstraintCheckInterval" Value="1.0" />
            <Parameter Name="MinLoadBalancingInterval" Value="5.0" />
        </Section>
```

今天我们一次只依序执行其中一项操作。例如，我们已响应任何请求，已在继续平衡群集之前创建新副本。如你所见，我们可以根据指定的默认时间间隔扫描并检查我们需要非常频繁执行的任何操作，这意味着我们在轮次末尾所做的更改集通常比较小：我们不是扫描群集中数小时的更改并尝试一次更正全部，并尝试在事情发生时进行处理，而是在同时有许多事情发生时进行某种批处理。这使得 Service Fabric 资源管理器对于群集中发生的事情保持极高的响应度。

从根本上讲，群集资源管理器还需要知道何时考虑不平衡的群集，以及应该移动哪些副本才能修复问题。为此，我们设置了其他两项主要配置：平衡阈值和活动阈值。

## 平衡阈值
平衡阈值是触发主动式重新平衡的主要控件。平衡阈值定义特定指标的群集有多么不平衡，Resource Manager 才将它视为不平衡并触发平衡。平衡阈值根据每个指标定义为群集定义的一部分：

ClusterManifest.xml

``` xml
    <Section Name="MetricBalancingThresholds">
      <Parameter Name="MetricName1" Value="2"/>
      <Parameter Name="MetricName2" Value="3.5"/>
    </Section>
```

指标的平衡阈值是一个比率。如果负载最重的节点的负载量除以负载最轻的节点的负载量超过此比率，此群集将被视为不平衡，并在下一次运行资源管理器的状态节点期间触发平衡。

![平衡阈值示例][Image1]

在此简单示例中，每个服务只使用一个单位的指标。在最上面的示例中，节点的负载上限为 5，下限为 2。假设此指标的平衡阈值为 3。因此，在最上面的示例中，群集被视为平衡，并且不触发平衡（因为群集中的比率为 5/2 = 2.5，这小于指定的平衡阈值 3）。

在最下面的示例中，节点的负载上限为 10，下限为 2（因此比率为 5），这超过了设计的平衡阈值 3。这样，全局重新平衡将在下一次激发计时器时发生，且负载几乎肯定会分散到 Node3。请注意，我们没使用其他人也可能在 Node2 上实施的穷尽方法，因为这会导致节点之间的整体差异缩减到最小，但我们预期多数负载会流向 Node3。

![平衡阈值示例操作][Image2]

请注意，获取以下平衡阈值不是明确的目标，平衡阈值只是触发器，告诉 Service Fabric 群集资源管理器应该查看群集，以确定可以做出哪些改进。

## 活动阈值
有时候，虽然节点相当不平衡，但群集中的负载总量很低。这可能只是因为当天时间关系，或是群集是新的并且刚刚开始引导。在任一情况下，你可能不想花费时间进行平衡，因为实际的收获很少 – 只是消耗网络和计算资源来移动项目。资源管理器中有另一个称为活动阈值的控件可让你指定活动的绝对下限 – 如果没有任何节点有至少这么多的负载，则即使达到平衡阈值，也不触发平衡。例如，假设在我们的报告中，这些节点的消耗量总计如下所示。同时假设我们让平衡阈值保持 3，但现在还有活动阈值 1536。在第一种情况下，根据平衡阈值，群集为不平衡状态，没有任何节点符合活动阈值下限，因此我们保持现状。在最下面的示例中，Node1 超过活动阈值，因此将执行平衡。

![活动阈值示例][Image3]

如同平衡阈值，活动阈值通过群集定义根据每个指标进行定义：

ClusterManifest.xml

``` xml
    <Section Name="MetricActivityThresholds">
      <Parameter Name="Memory" Value="1536"/>
    </Section>
```

## 一起平衡服务
值得一提的是，群集是否不平衡是群集范围的决策，但解决这种情况的方法是移动单个服务副本和实例。这种说法很合理，是吗？ 如果内存堆积在某一个节点上，则可能是由多个副本或实例造成的，因此需要移动所有使用受影响、不平衡指标的副本或实例。

有时，客户会呼叫我们或建立支持票证，指出已移动不平衡的服务。在有其他不平衡情况的时候，如果该服务的所有指标都已平衡（已经非常完美了），为何需要移动服务呢？ 让我们看一看！

以 S1、S2、S3 和 S4 这四个服务为例。S1 报告指标 M1 和 M2，S2 报告 M2 和 M3，S3 报告 M3 和 M4，S4 报告指标 M99。你可以看到此处的运行情况。这里形成了一个链条！ 从资源管理器的立场来讲，我们的确没有 4 个独立的服务，但我们有许多相关的服务（S1、 S2 和 S3）以及一个独立的服务。

![一起平衡服务][Image4]

因此，Metric1 不失衡可能会导致属于 Service3 的副本或实例四处移动。这种移动通常很有限，但是根据 Metric1 究竟有多么不平衡以及需要在群集中进行哪些更改才能更正问题，移动幅度可能变大。我们还可以肯定地说，指标 1、2 或 3 不平衡绝不会导致 Service4 移动 – 没有必要，因为移动属于 Service4 的副本或实例完全不影响指标 1、2 或 3 的平衡。

资源管理器在每次运行时自动找出相关的服务，因为可能添加、删除了服务，或者服务的指标配置发生了更改 – 例如，在两个平衡轮次之间，Service2 可能已重新配置为删除 Metric2。这会中断 Service1 和 Service2 之间的链接。现在你有三个服务组，而不是两个：

![一起平衡服务][Image5]

## 后续步骤
- 指标是 Service Fabric 群集资源管理器在群集中管理消耗和容量的方式。若要详细了解指标及其配置方式，请查看[此文](/documentation/articles/service-fabric-cluster-resource-manager-metrics/)
- 移动成本是向群集资源管理器发出信号，表示移动某些服务比移动其他服务会产生更高成本的方式之一。若要了解有关移动成本的详细信息，请参阅[此文](/documentation/articles/service-fabric-cluster-resource-manager-movement-cost/)
- 群集资源管理器提供多个限制机制，你可以配置这些限制机制，以减慢群集中的流动。这些限制通常不是必要的，但如果需要，你可以在[此处](/documentation/articles/service-fabric-cluster-resource-manager-advanced-throttling/)了解其相关信息


[Image1]: ./media/service-fabric-cluster-resource-manager-balancing/cluster-resrouce-manager-balancing-thresholds.png
[Image2]: ./media/service-fabric-cluster-resource-manager-balancing/cluster-resource-manager-balancing-threshold-triggered-results.png
[Image3]: ./media/service-fabric-cluster-resource-manager-balancing/cluster-resource-manager-activity-thresholds.png
[Image4]: ./media/service-fabric-cluster-resource-manager-balancing/cluster-resource-manager-balancing-services-together1.png
[Image5]: ./media/service-fabric-cluster-resource-manager-balancing/cluster-resource-manager-balancing-services-together2.png

<!---HONumber=Mooncake_0627_2016-->