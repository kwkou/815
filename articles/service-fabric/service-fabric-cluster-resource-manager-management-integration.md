<properties
    pageTitle="Service Fabric 群集资源管理器 - 管理集成 | Azure"
    description="概述群集资源管理器与 Service Fabric 管理之间的集成点。"
    services="service-fabric"
    documentationcenter=".net"
    author="masnider"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="956cd0b8-b6e3-4436-a224-8766320e8cd7"
    ms.service="Service-Fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="01/05/2017"
    wacn.date="02/20/2017"
    ms.author="masnider" />  


# 群集资源管理器与 Service Fabric 群集管理的集成
Service Fabric 群集资源管理器不是负责处理管理操作（如应用程序升级）的主要 Service Fabric 组件，但会参与此类操作。群集资源管理器帮助进行管理的第一种方式是跟踪群集及其中服务的所需状态。无法将群集放入所需配置时，群集资源管理器会发出运行状况报告。例如，当容量不足，或者有关服务放置位置的规则发生冲突时，它会发出此类报告。集成的另一个部分与升级的工作方式有关。在升级期间，群集资源管理器会稍微改变其行为。下面将讨论这两个集成要点。

## 运行状况集成
群集资源管理器不断跟踪针对服务定义的规则，以及节点和群集中可用的容量。如果不满足这些规则或者容量不足，群集资源管理器将发出运行状况警告和错误。例如，如果某个节点超出容量，资源管理器将尝试通过移动服务来修复这种问题。如果无法纠正，群集资源管理器将发出运行状况警告，指出哪个节点超出容量，以及警告是针对哪些指标。

资源管理器发出运行状况警告的另一个示例是发生了放置约束违规情况。例如，如果用户定义了放置约束（如 `“NodeColor == Blue”`），而资源管理器检测到违反该约束的情况，则它会发出运行状况警告。这一点适用于自定义约束和默认约束（例如容错域和升级域约束）。

下面是此类运行状况报告的示例。在这种情况下，运行状况报告适用于系统服务的分区之一。运行状况消息指出，该分区的副本临时打包到了过少的升级域。

	PS C:\Users\User > Get-WindowsFabricPartitionHealth -PartitionId '00000000-0000-0000-0000-000000000001'


	PartitionId           : 00000000-0000-0000-0000-000000000001
	AggregatedHealthState : Warning
	UnhealthyEvaluations  :
                        	Unhealthy event: SourceId='System.PLB', Property='ReplicaConstraintViolation_UpgradeDomain', HealthState='Warning', ConsiderWarningAsError=false.

	ReplicaHealthStates   :
                        	ReplicaId             : 130766528804733380
                        	AggregatedHealthState : Ok

                        	ReplicaId             : 130766528804577821
                        	AggregatedHealthState : Ok

                        	ReplicaId             : 130766528854889931
                        	AggregatedHealthState : Ok

                        	ReplicaId             : 130766528804577822
                        	AggregatedHealthState : Ok

                        	ReplicaId             : 130837073190680024
                        	AggregatedHealthState : Ok

	HealthEvents          :
                        	SourceId              : System.PLB
                        	Property              : ReplicaConstraintViolation_UpgradeDomain
                        	HealthState           : Warning
                        	SequenceNumber        : 130837100116930204
                        	SentAt                : 8/10/2015 7:53:31 PM
                        	ReceivedAt            : 8/10/2015 7:53:33 PM
                        	TTL                   : 00:01:05
                        	Description           : The Load Balancer has detected a Constraint Violation for this Replica: fabric:/System/FailoverManagerService Secondary Partition 00000000-0000-0000-0000-000000000001 is
                        	violating the Constraint: UpgradeDomain Details: Node -- 3d1a4a68b2592f55125328cd0f8ed477  Policy -- Packing
                        	RemoveWhenExpired     : True
                        	IsExpired             : False
                        	Transitions           : Ok->Warning = 8/10/2015 7:13:02 PM, LastError = 1/1/0001 12:00:00 AM


下面是此运行状况消息所告知的信息：

1. 所有副本本身运行状况正常（这是 Service Fabric 的第一要务）
2. 当前违反了升级域分发约束（意味着特定升级域拥有此分区的过多副本）
3. 哪个节点包含造成违规的副本（具有以下 ID 的节点：3d1a4a68b2592f55125328cd0f8ed477）
4. 报告发生的时间 \(8/10/2015 7:13:02 PM\)

此类信息丰富了生产环境中触发的警报，可让用户知道某个地方出错。在此情况下，我们可以调查资源管理器为何必须将副本打包到升级域。例如，这可能是因为其他升级域中的节点已关闭。

假设你要创建一个服务，或者资源管理器正尝试查找一个用于放置某些服务的位置，但没有任何可行的解决方案。其中的原因很多，但通常不外乎是以下两种情况之一：

1. 某个暂时性情况导致无法正确放置此服务实例或副本
2. 服务的要求配置不当，以致无法满足其要求。

在每种情况下，都会看见群集资源管理器所提供的运行状况报告，其中包含可帮助判断当前状况以及为何无法放置服务的信息。我们将此过程称为“约束消除序列”。在此过程中，系统将逐步了解配置的约束如何影响服务，并记录约束消除的因素。这样，当无法放置服务时，便可以看到哪些节点已被消除及其原因。

## 约束类型
现在，我们讨论可在这些运行状况报告中看到的各种约束。大多数的情况下，这些约束不会消除节点，因为约束默认位于软式或优化级别。但是，可以看到与约束有关的运行状况消息，了解它们是否是设置为硬式约束，或它们是否在极少见的情况下确实造成节点被消除。

* **ReplicaExclusionStatic** 和 **ReplicaExclusionDynamic**：此约束指出同一分区中的两个有状态副本或无状态实例必须放置在同一节点上（不允许这样做）。ReplicaExclusionStatic 和 ReplicaExclusionDynamic 遵循几乎相同的规则。ReplicaExclusionDynamic 约束表示“我们无法将此副本放在此处，因为唯一提议的解决方案已在此处放置一个副本”。这与 ReplicaExclusionStatic 约束不同，后者指出不是提议的冲突，而是实际的冲突。在这种情况下，节点上已有一个副本。让人困惑吗？ 是的。这是否很重要？ 不会。如果看到的约束消除序列包含 ReplicaExclusionStatic 或 ReplicaExclusionDynamic 约束，群集资源管理器就会认为没有足够的节点。其他约束通常告诉我们，节点太少会造成哪种结果。
* **PlacementConstraint**：如果看到此消息，表示已消除了一些节点，因为它们不符合服务的放置约束。我们在此消息中描绘当前配置的放置约束。如果定义了放置约束，则这种情况是正常的。但是，如果放置约束中存在 bug，导致消除了过多的节点，则会在此处看到这种结果。
* **NodeCapacity**：此约束表示群集资源管理器无法将副本放在指定的节点上，因为这会导致节点超出容量。
* **Affinity**：此约束表示无法将副本放在受影响的节点上，因为这会导致违反相关性约束。[此文](/documentation/articles/service-fabric-cluster-resource-manager-advanced-placement-rules-affinity/)介绍了有关相关性的详细信息。
* **FaultDomain** 和 **UpgradeDomain**：如果将副本放在指定的节点上会导致副本打包在特定的容错域或升级域中，此约束将消除节点。[容错域与升级域约束及最终行为](/documentation/articles/service-fabric-cluster-resource-manager-cluster-description/)中的主题提供了几个介绍此约束的示例
* **PreferredLocation**：通常我们看不到这个会导致节点从解决方案中删除的约束，因为该约束默认情况下是在优化级别设置的。此外，首选的位置约束通常只出现在升级期间。在升级期间，该约束用于将副本移回到开始升级时的位置。

###<a name="constraint-priorities"></a> 约束优先级
在所有约束中，你可能觉得：“嘿 – 我认为放置约束在系统中是最重要的。如果能够确保不违反放置约束，我愿意违反其他约束，甚至包括相关性和容量的约束。”

其实我们的确可以这样做！ 可以使用几个不同级别的强制措施来配置约束，但这些措施都可归结为“硬”\(0\)、“软”\(1\)、“优化”\(2\) 和“关闭”\(-1\)。默认情况下，我们会将大多数约束定义为硬约束。举例来说，大部分用户通常不会将容量视为可以放宽松的要素，而大多数约束不是硬约束就是软约束。

不同的约束优先级也是为何某些约束违规警告出现的次数多过其他警告的原因：因为我们愿意暂时让某些约束保持宽松（违规）。这些级别并不是真正表示会违反给定的约束，只是指定了偏好的强制实施顺序。因此，当无法满足所有约束时，群集资源管理器可以做出适当的取舍。

在某些高级场合中，可以更改约束优先级。例如，假如你希望违反相关性可以确保解决节点容量问题。为此，可将相关性约束的优先级设置为“软”\(1\)，将容量约束保持设置为“硬”\(0\)。

配置文件中指定了不同约束的默认优先级值：

ClusterManifest.xml


        <Section Name="PlacementAndLoadBalancing">
            <Parameter Name="PlacementConstraintPriority" Value="0" />
            <Parameter Name="CapacityConstraintPriority" Value="0" />
            <Parameter Name="AffinityConstraintPriority" Value="0" />
            <Parameter Name="FaultDomainConstraintPriority" Value="0" />
            <Parameter Name="UpgradeDomainConstraintPriority" Value="1" />
            <Parameter Name="PreferredLocationConstraintPriority" Value="2" />
        </Section>

通过 ClusterConfig.json 进行独立部署或将 Template.json 用于 Azure 托管群集：


	"fabricSettings": [
	  {
	    "name": "PlacementAndLoadBalancing",
	    "parameters": [
	      {
	          "name": "PlacementConstraintPriority",
	          "value": "0"
	      },
	      {
	          "name": "CapacityConstraintPriority",
	          "value": "0"
	      },
	      {
	          "name": "AffinityConstraintPriority",
	          "value": "0"
	      },
	      {
	          "name": "FaultDomainConstraintPriority",
	          "value": "0"
	      },
	      {
	          "name": "UpgradeDomainConstraintPriority",
	          "value": "1"
	      },
	      {
	          "name": "PreferredLocationConstraintPriority",
	          "value": "2"
	      }
	    ]
	  }
	]


## 容错域和升级域约束
若要将服务分散在容错域与升级域之间作为资源管理器引擎内的约束，群集资源管理器可为这种想法建模。有关具体用法的详细信息，请参阅有关[群集配置](/documentation/articles/service-fabric-cluster-resource-manager-cluster-description/)的文章。

在过去的某些时候，我们需要严格定义容错域与升级域，防止发生某种错误。而在某些情况下，我们又需要完全忽略这些定义（虽然是暂时性的）。一般而言，约束优化级基础结构的弹性可以应对各种问题，但我们并不经常需要利用这种弹性。在大部分时间内，每个组成部分使用其默认优先级就能正常运作。升级域保持软约束。群集资源管理器可能需要将一些副本打包到升级域，以处理升级、故障或约束违规。通常，仅当系统中的多种故障或其他扰乱因素阻止正确放置时，才发生这种情况。如果已正确配置环境，则即使在升级期间，也会完全遵循所有约束（包括容错和升级约束）。关键的一点是，群集资源管理器会自动监视约束，在检测到违规时立即发出报告。

## 首选位置约束
PreferredLocation 约束稍有不同，因此它是唯一设置为“Optimization”的约束。我们将在升级过程中使用此约束，以便将服务放回到升级之前它们所在的位置。在实践中，有许多不同的原因会导致这种做法不可行，但优化的效果仍然不错。

## 升级
在应用程序和群集升级期间，群集资源管理器也会提供帮助，在此过程中它会执行两个作业：

* 确保规则和群集性能不受到影响
* 尝试帮助升级顺利完成

### 保持强制实施规则
规则是需要注意的重点 – 在升级期间仍强制实施一些严格控制（如放置约束）。放置约束确保工作负载仅在受允许的情况下才能运行，即使在升级期间也是如此。如果环境高度受约束，升级可能花费很长时间。这是因为，如果服务（或它所在的节点）需要关闭进行更新，则服务只有少数几个选项可用。

### 智能替换
开始升级时，资源管理器将创建当前群集排列方式的快照。完成每个升级域后，它会尝试将节点恢复为原始排列方式。因此，在升级过程中最终只会出现两次转换（将受影响的节点移出，然后移回）。将群集恢复到升级前的状态还可确保升级不会对群集的布局造成影响。如果在升级之前已排列好群集，则升级之后也会排列妥当，或者至少不会让事情变得更糕。

### 降低流动
升级期间还会发生另一种情况，那就是群集资源管理器对正在升级的实体关闭平衡。这意味着，如果有两个不同的应用程序实例并在其中一个实例上升级，则该应用程序实例的暂时失衡，但另一个不会。阻止均衡可防止对升级本身做出不必要的反应，例如，为了升级而将服务移入空节点。如果有问题的升级是群集升级，则整个群集在升级期间将会失衡。约束检查 – 确保强制实施规则 – 保持活动状态，只禁用重新均衡。

### 宽松规则
通常，即使群集受到约束或整体已填满，我们还是希望升级能够完成。在升级期间，管理群集容量的能力至关重要。这是因为，升级是在整个群集中进行的，每次通常有 5% 到 20% 的容量下降。工作负荷通常必须转到某个位置。这就是[缓冲容量](/documentation/articles/service-fabric-cluster-resource-manager-cluster-description/#buffered-capacity)概念真正派上用场的地方。尽管在正常操作期间遵守缓冲容量（留出一些可用空间），但群集资源管理器将在升级期间填满总容量（占用缓冲区）。

## 后续步骤
- 参阅 [Service Fabric 群集资源管理器简介](/documentation/articles/service-fabric-cluster-resource-manager-introduction/)，帮助自己入门

<!---HONumber=Mooncake_0213_2017-->
<!--Update_Description: enrich introduction for "约束优先级"; add Template.json script for "Azure 托管群集"-->