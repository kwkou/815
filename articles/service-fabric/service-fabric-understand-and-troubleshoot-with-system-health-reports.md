<properties
    pageTitle="使用系统运行状况报告进行故障排除 | Azure"
    description="介绍 Azure Service Fabric 组件发送的运行状况报告，以及如何使用这些报告来排查群集或应用程序问题。"
    services="service-fabric"
    documentationcenter=".net"
    author="oanapl"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="52574ea7-eb37-47e0-a20a-101539177625"
    ms.service="service-fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="04/12/2017"
    wacn.date="05/15/2017"
    ms.author="oanapl"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="e245ed61f8521d14a466c59dea7267d8471a15d3"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="use-system-health-reports-to-troubleshoot"></a>使用系统运行状况报告进行故障排除
Azure Service Fabric 组件报告包含群集中的所有实体。 [运行状况存储](/documentation/articles/service-fabric-health-introduction/#health-store)根据系统报告来创建和删除实体。 它还将这些实体组织为层次结构以捕获实体交互。

> [AZURE.NOTE]
> 请阅读 [Service Fabric 运行状况模型](/documentation/articles/service-fabric-health-introduction/)以了解与运行状况相关的概念。
> 
> 

系统运行状况报告提供有关群集和应用程序功能的可见性，并且通过运行状况标记问题。对于应用程序和服务，系统运行状况报告从 Service Fabric 的角度验证实体得到实现并且正常运行。报告不对服务的业务逻辑进行任何运行状况监视，也不检测暂停的进程。用户服务可以使用其逻辑的特有信息来丰富运行状况数据。

> [AZURE.NOTE]
> 监视器运行状况报告仅在系统组件创建一个实体*之后*才可见。 在删除实体之后，运行状况存储自动删除与该实体关联的所有运行状况报告。 创建实体的新实例时的处理方式也一样（例如，创建新的服务副本实例）。 所有与旧实例关联的报告都将从存储中删除并清除。
> 
> 

按来源标识系统组件报告，并以“**System**”。 前缀开头。 监视器不能与来源使用相同的前缀，因为如果参数无效，报告将被拒绝。
让我们来看一些系统报告，并了解是什么触发了这些报告以及如何纠正报告指出的问题。

> [AZURE.NOTE] Service Fabric 不断添加感兴趣的状况报告，这些报告可以提高对群集和应用程序中正在发生的事情的可见性。


## <a name="cluster-system-health-reports"></a>群集系统运行状况报告
群集运行状况实体在运行状况存储中自动创建。 如果一切运行正常，则不提供系统报告。

### <a name="neighborhood-loss"></a>邻居丢失
**System.Federation** 在检测到邻居丢失时会报告一个错误。 报告来自于单个节点，并且在属性名称中包含节点 ID。 如果在整个 Service Fabric 环中出现一个邻居丢失，通常可预料到两个事件（间隙的两侧都会报告）。 如果有多个邻居丢失，则将有更多事件。

报告将全局租约超时指定为生存时间。 只要条件仍处于活动状态，就会在每半个 TTL 期间重新发送一次报告。 事件过期后将被自动删除。 即使报告节点已关闭，删除过期事件的行为也能确保从运行状况存储中正确清理报告。

* **SourceId**：System.Federation
* **属性**：以 **Neighborhood** 开头并包含节点信息
* **后续步骤**：调查网络上邻居丢失的原因（例如，检查群集节点之间的通信）。

## <a name="node-system-health-reports"></a>节点系统运行状况报告
**System.FM**表示故障转移管理器 (Failover Manager) 服务，是管理群集节点信息的主管服务。 每个节点应该都有一个来自 System.FM 的报告，显示其状态。 删除节点状态时，也会删除节点实体（请参阅 [RemoveNodeStateAsync](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.clustermanagementclient.removenodestateasync)）。

### <a name="node-updown"></a>节点开启/节点关闭
节点加入环时，System.FM 报告为正常（节点已启动且正在运行）。 节点离开环时，则报告错误（节点已关闭进行升级，或只是发生故障）。 运行状况存储构建的运行状况层次结构依据 System.FM 节点报告在已部署的实体上实施操作。 它将节点视为所有已部署实体的虚拟父项。 如果 System.FM 报告节点已启动并且其实例与实体关联的实例相同，则可以通过查询公开该节点上已部署的实体。 当 System.FM 报告节点关闭或已重新启动（新实例）时，运行状况存储自动清理只能在已关闭节点或该节点的上一实例中存在的已部署实体。

* **SourceId**：System.FM
* **属性**：State
* **后续步骤**：如果节点已关闭以进行升级，应该在升级后重新启动。 在这种情况下，运行状况应切换回正常。 如果节点没有重新启动或发生故障，则需要进一步调查问题。

以下示例显示 System.FM 事件，且节点正常运行时的运行状况状态为正常：

	PS C:\> Get-ServiceFabricNodeHealth -NodeName Node.1
	NodeName              : Node.1
	AggregatedHealthState : Ok
	HealthEvents          :
	                        SourceId              : System.FM
	                        Property              : State
	                        HealthState           : Ok
	                        SequenceNumber        : 2
	                        SentAt                : 4/24/2015 5:27:33 PM
	                        ReceivedAt            : 4/24/2015 5:28:50 PM
	                        TTL                   : Infinite
	                        Description           : Fabric node is up.
	                        RemoveWhenExpired     : False
	                        IsExpired             : False
	                        Transitions           : ->Ok = 4/24/2015 5:28:50 PM


### <a name="certificate-expiration"></a>证书过期日期
**System.FabricNode** 在节点使用的证书即将过期时报告警告。 每个节点有三个证书：**Certificate_cluster**、**Certificate_server** 和 **Certificate_default_client**。 如果过期时间至少超过两周，报告运行状况是正常。 如果过期时间在两周内，则报告类型为警告。 这些事件的 TTL 是无限的，节点离开群集时，它们会被删除。

* **SourceId**：System.FabricNode
* **属性**：以 **Certificate** 开头并且包含有关证书类型的详细信息
* **后续步骤**：如果证书即将过期，则更新证书。

### <a name="load-capacity-violation"></a>负载容量冲突
如果 Service Fabric 负载均衡器检测到节点负载容量冲突，则报告警告。

* **SourceId**：System.PLB
* **属性**：以 **Capacity** 开头
* **后续步骤**：检查提供的指标并查看节点上的当前容量。

## <a name="application-system-health-reports"></a>应用程序系统运行状况报告
**System.CM**表示群集管理器服务，是管理应用程序信息的主管服务。

### <a name="state"></a>状态
创建或更新应用程序时，System.CM 报告正常。 删除应用程序时，它会通知运行状况存储，以便能从存储中删除应用程序。

* **SourceId**：System.CM
* **属性**：State
* **后续步骤**：如果已创建应用程序，它就应该包含群集管理器运行状况报告。 否则，通过发出一个查询（例如 PowerShell cmdlet **Get-ServiceFabricApplication -ApplicationName *applicationName***）来检查应用程序的状态。

以下示例显示 **fabric:/WordCount** 应用程序上的状态事件：


	PS C:\> Get-ServiceFabricApplicationHealth fabric:/WordCount -ServicesFilter None -DeployedApplicationsFilter None
	
	ApplicationName                 : fabric:/WordCount
	AggregatedHealthState           : Ok
	ServiceHealthStates             : None
	DeployedApplicationHealthStates : None
	HealthEvents                    :
	                                  SourceId              : System.CM
	                                  Property              : State
	                                  HealthState           : Ok
	                                  SequenceNumber        : 82
	                                  SentAt                : 4/24/2015 6:12:51 PM
	                                  ReceivedAt            : 4/24/2015 6:12:51 PM
	                                  TTL                   : Infinite
	                                  Description           : Application has been created.
	                                  RemoveWhenExpired     : False
	                                  IsExpired             : False
	                                  Transitions           : ->Ok = 4/24/2015 6:12:51 PM

## <a name="service-system-health-reports"></a>服务系统运行状况报告
**System.FM**表示故障转移管理器服务，是管理服务信息的主管服务。

### <a name="state"></a>状态
已创建服务时，System.FM 报告正常。 已删除服务时，它从运行状况存储删除实体。

* **SourceId**：System.FM
* **属性**：State

以下示例显示服务 **fabric:/WordCount/WordCountService** 上的状态事件：


	PS C:\> Get-ServiceFabricServiceHealth fabric:/WordCount/WordCountService
	
	ServiceName           : fabric:/WordCount/WordCountService
	AggregatedHealthState : Ok
	PartitionHealthStates :
	                        PartitionId           : 875a1caa-d79f-43bd-ac9d-43ee89a9891c
	                        AggregatedHealthState : Ok
	
	HealthEvents          :
	                        SourceId              : System.FM
	                        Property              : State
	                        HealthState           : Ok
	                        SequenceNumber        : 3
	                        SentAt                : 4/24/2015 6:12:51 PM
	                        ReceivedAt            : 4/24/2015 6:13:01 PM
	                        TTL                   : Infinite
	                        Description           : Service has been created.
	                        RemoveWhenExpired     : False
	                        IsExpired             : False
	                        Transitions           : ->Ok = 4/24/2015 6:13:01 PM

### <a name="unplaced-replicas-violation"></a>未放置的副本冲突
**System.PLB** 找不到放置一或多个服务副本的位置，则报告警告。 报告过期时会被删除。

* **SourceId**：System.FM
* **属性**：State
* **后续步骤**：检查服务约束和当前放置状态。

以下示例显示配置有 7 个目标副本的服务在含有 5 个节点的群集中发生的冲突：


	PS C:\> Get-ServiceFabricServiceHealth fabric:/WordCount/WordCountService


	ServiceName           : fabric:/WordCount/WordCountService
	AggregatedHealthState : Warning
	UnhealthyEvaluations  :
                        	Unhealthy event: SourceId='System.PLB',
                        	Property='ServiceReplicaUnplacedHealth_Secondary_a1f83a35-d6bf-4d39-b90d-28d15f39599b', HealthState='Warning',
                        	ConsiderWarningAsError=false.

	PartitionHealthStates :
                        	PartitionId           : a1f83a35-d6bf-4d39-b90d-28d15f39599b
                        	AggregatedHealthState : Warning

	HealthEvents          :
                        	SourceId              : System.FM
                        	Property              : State
                        	HealthState           : Ok
                        	SequenceNumber        : 10
                        	SentAt                : 3/22/2016 7:56:53 PM
                        	ReceivedAt            : 3/22/2016 7:57:18 PM
                        	TTL                   : Infinite
                        	Description           : Service has been created.
                        	RemoveWhenExpired     : False
                        	IsExpired             : False
                        	Transitions           : Error->Ok = 3/22/2016 7:57:18 PM, LastWarning = 1/1/0001 12:00:00 AM

                        	SourceId              : System.PLB
                        	Property              : ServiceReplicaUnplacedHealth_Secondary_a1f83a35-d6bf-4d39-b90d-28d15f39599b
                        	HealthState           : Warning
                        	SequenceNumber        : 131032232425505477
                        	SentAt                : 3/23/2016 4:14:02 PM
                        	ReceivedAt            : 3/23/2016 4:14:03 PM
                        	TTL                   : 00:01:05
                        	Description           : The Load Balancer was unable to find a placement for one or more of the Service's Replicas:
                        	fabric:/WordCount/WordCountService Secondary Partition a1f83a35-d6bf-4d39-b90d-28d15f39599b could not be placed, possibly,
                        	due to the following constraints and properties:  
                        	Placement Constraint: N/A
                        	Depended Service: N/A

                        	Constraint Elimination Sequence:
                        	ReplicaExclusionStatic eliminated 4 possible node(s) for placement -- 1/5 node(s) remain.
                        	ReplicaExclusionDynamic eliminated 1 possible node(s) for placement -- 0/5 node(s) remain.

                        	Nodes Eliminated By Constraints:

                        	ReplicaExclusionStatic:
                        	FaultDomain:fd:/0 NodeName:_Node_0 NodeType:NodeType0 UpgradeDomain:0 UpgradeDomain: ud:/0 Deactivation Intent/Status:
                        	None/None
                        	FaultDomain:fd:/1 NodeName:_Node_1 NodeType:NodeType1 UpgradeDomain:1 UpgradeDomain: ud:/1 Deactivation Intent/Status:
                        	None/None
                        	FaultDomain:fd:/3 NodeName:_Node_3 NodeType:NodeType3 UpgradeDomain:3 UpgradeDomain: ud:/3 Deactivation Intent/Status:
                        	None/None
                        	FaultDomain:fd:/4 NodeName:_Node_4 NodeType:NodeType4 UpgradeDomain:4 UpgradeDomain: ud:/4 Deactivation Intent/Status:
                        	None/None

                        	ReplicaExclusionDynamic:
                        	FaultDomain:fd:/2 NodeName:_Node_2 NodeType:NodeType2 UpgradeDomain:2 UpgradeDomain: ud:/2 Deactivation Intent/Status:
                        	None/None


                        	RemoveWhenExpired     : True
                        	IsExpired             : False
                        	Transitions           : Error->Warning = 3/22/2016 7:57:48 PM, LastOk = 1/1/0001 12:00:00 AM

## <a name="partition-system-health-reports"></a>分区系统运行状况报告
**System.FM**表示故障转移管理器服务，是管理服务分区信息的主管服务。

### <a name="state"></a>状态
创建分区并且分区正常时，System.FM 报告正常。 删除分区时，它从运行状况存储删除实体。

如果分区小于最小副本计数，则它将报告错误。如果分区不小于最小副本计数，但是小于目标副本计数，则它将报告警告。如果分区处于仲裁丢失状态，System.FM 会报告错误。

其他重要的事件包括重新配置超过预期时间时以及构建时间超过预期时发出警告。构建和重新配置的预期时间可依据服务方案配置。例如，如果服务拥有 1 TB 的状态（例如 SQL 数据库），则构建的时间比小数量状态的服务要长。

* **SourceId**：System.FM
* **属性**：State
* **后续步骤**：如果运行状况不正常，则有可能某些副本没有正确创建、打开或提升为主副本或次要副本。 很多情况下，根本原因是服务在打开或更改角色实现中存在 bug。

以下示例显示了一个运行状况良好的分区：


	PS C:\> Get-ServiceFabricPartition fabric:/StatelessPiApplication/StatelessPiService | Get-ServiceFabricPartitionHealth
	PartitionId           : 29da484c-2c08-40c5-b5d9-03774af9a9bf
	AggregatedHealthState : Ok
	ReplicaHealthStates   : None
	HealthEvents          :
                        	SourceId              : System.FM
                        	Property              : State
                        	HealthState           : Ok
                        	SequenceNumber        : 38
                        	SentAt                : 4/24/2015 6:33:10 PM
                        	ReceivedAt            : 4/24/2015 6:33:31 PM
                        	TTL                   : Infinite
                        	Description           : Partition is healthy.
                        	RemoveWhenExpired     : False
                        	IsExpired             : False
                        	Transitions           : ->Ok = 4/24/2015 6:33:31 PM


以下示例显示了一个小于目标副本计数的分区的运行状况。下一步是获取显示分区配置方式的分区描述：**MinReplicaSetSize** 为 2，**TargetReplicaSetSize** 为 7。然后获得群集中的节点数：5。因此在这种情形下，有两个副本不能放置。


	PS C:\> Get-ServiceFabricPartition fabric:/WordCount/WordCountService | Get-ServiceFabricPartitionHealth -ReplicasFilter None

	PartitionId           : 875a1caa-d79f-43bd-ac9d-43ee89a9891c
	AggregatedHealthState : Warning
	UnhealthyEvaluations  :
                        	Unhealthy event: SourceId='System.FM', Property='State', HealthState='Warning', ConsiderWarningAsError=false.

	ReplicaHealthStates   : None
	HealthEvents          :
                        	SourceId              : System.FM
                        	Property              : State
                        	HealthState           : Warning
                        	SequenceNumber        : 37
                        	SentAt                : 4/24/2015 6:13:12 PM
                        	ReceivedAt            : 4/24/2015 6:13:31 PM
                        	TTL                   : Infinite
                        	Description           : Partition is below target replica or instance count.
                        	RemoveWhenExpired     : False
                        	IsExpired             : False
                        	Transitions           : Ok->Warning = 4/24/2015 6:13:31 PM

	PS C:\> Get-ServiceFabricPartition fabric:/WordCount/WordCountService

	PartitionId            : 875a1caa-d79f-43bd-ac9d-43ee89a9891c
	PartitionKind          : Int64Range
	PartitionLowKey        : 1
	PartitionHighKey       : 26
	PartitionStatus        : Ready
	LastQuorumLossDuration : 00:00:00
	MinReplicaSetSize      : 2
	TargetReplicaSetSize   : 7
	HealthState            : Warning
	DataLossNumber         : 130743727710830900
	ConfigurationNumber    : 8589934592


	PS C:\> @(Get-ServiceFabricNode).Count
	5

### <a name="replica-constraint-violation"></a>副本约束冲突
**System.PLB** 检测到副本约束冲突并且无法放置分区的副本，则报告警告。

* **SourceId**：System.PLB
* **属性**：以 **ReplicaConstraintViolation** 开头

## <a name="replica-system-health-reports"></a>副本系统运行状况报告
**System.RA**表示重新配置代理组件，是用于处理副本状态的主管组件。

### <a name="state"></a>状态
**System.RA** 报告正常。

* **SourceId**：System.RA
* **属性**：State

以下示例显示了一个运行状况良好的副本：


	PS C:\> Get-ServiceFabricPartition fabric:/WordCount/WordCountService | Get-ServiceFabricReplica | where {$_.ReplicaRole -eq "Primary"} | Get-ServiceFabricReplicaHealth
	PartitionId           : 875a1caa-d79f-43bd-ac9d-43ee89a9891c
	ReplicaId             : 130743727717237310
	AggregatedHealthState : Ok
	HealthEvents          :
                        	SourceId              : System.RA
                        	Property              : State
                        	HealthState           : Ok
                        	SequenceNumber        : 130743727718018580
                        	SentAt                : 4/24/2015 6:12:51 PM
                        	ReceivedAt            : 4/24/2015 6:13:02 PM
                        	TTL                   : Infinite
                        	Description           : Replica has been created.
                        	RemoveWhenExpired     : False
                        	IsExpired             : False
                        	Transitions           : ->Ok = 4/24/2015 6:13:02 PM

### <a name="replica-open-status"></a>副本打开状态
此运行状况报告的描述包含调用 API 时的开始时间（协调世界时）。

如果副本打开时间超过配置的时间（默认为 30 分钟），则 **System.RA** 报告警告。 如果 API 影响服务可用性，则以更快的速度发送报告（间隔时间可配置，默认为 30 秒）。 此计量时间包括复制器打开和服务打开所用的时间。 如果打开完成，则属性更改为正常。

* **SourceId**：System.RA
* **属性**上的状态事件： **ReplicaOpenStatus**
* **后续步骤**：如果运行状况不正常，则调查副本打开时间超过预期的原因。

### <a name="slow-service-api-call"></a>服务 API 调用缓慢
如果对用户服务代码的调用时间超过配置的时间，则 **System.RAP** 和 **System.Replicator** 报告警告。 当调用完成时，警告被清除。

* **SourceId**：System.RAP 或 System.Replicator
* **属性**：慢速 API 的名称。 说明提供了有关 API 挂起时间的详细信息。
* **后续步骤**：调查调用时间超过预期的原因。

以下示例显示仲裁丢失状态的分区以及用于找出原因的调查步骤。其中一个副本的运行状况状态为警告，因此需要获取其运行状况。它显示服务操作时间超过预期，且 System.RAP 报告了事件。收到此信息之后，下一步是查看服务代码并进行调查。对于这种情况，有状态服务的 **RunAsync** 实现会引发一个未经处理的异常。副本正在循环，因此可能看不到任何处于警告状态的副本。可重试获取运行状况，并找出副本 ID 中的差异。某些情况下，重试可以提供线索。


	PS C:\> Get-ServiceFabricPartition fabric:/HelloWorldStatefulApplication/HelloWorldStateful | Get-ServiceFabricPartitionHealth

	PartitionId           : 72a0fb3e-53ec-44f2-9983-2f272aca3e38
	AggregatedHealthState : Error
	UnhealthyEvaluations  :
                        	Error event: SourceId='System.FM', Property='State'.

	ReplicaHealthStates   :
                        	ReplicaId             : 130743748372546446
                        	AggregatedHealthState : Ok

                        	ReplicaId             : 130743746168084332
                        	AggregatedHealthState : Ok

                        	ReplicaId             : 130743746195428808
                        	AggregatedHealthState : Warning

                        	ReplicaId             : 130743746195428807
                        	AggregatedHealthState : Ok

	HealthEvents          :
                        	SourceId              : System.FM
                        	Property              : State
                        	HealthState           : Error
                        	SequenceNumber        : 182
                        	SentAt                : 4/24/2015 7:00:17 PM
                        	ReceivedAt            : 4/24/2015 7:00:31 PM
                        	TTL                   : Infinite
                        	Description           : Partition is in quorum loss.
                        	RemoveWhenExpired     : False
                        	IsExpired             : False
                        	Transitions           : Warning->Error = 4/24/2015 6:51:31 PM

	PS C:\> Get-ServiceFabricPartition fabric:/HelloWorldStatefulApplication/HelloWorldStateful

	PartitionId            : 72a0fb3e-53ec-44f2-9983-2f272aca3e38
	PartitionKind          : Int64Range
	PartitionLowKey        : -9223372036854775808
	PartitionHighKey       : 9223372036854775807
	PartitionStatus        : InQuorumLoss
	LastQuorumLossDuration : 00:00:13
	MinReplicaSetSize      : 2
	TargetReplicaSetSize   : 3
	HealthState            : Error
	DataLossNumber         : 130743746152927699
	ConfigurationNumber    : 227633266688

	PS C:\> Get-ServiceFabricReplica 72a0fb3e-53ec-44f2-9983-2f272aca3e38 130743746195428808

	ReplicaId           : 130743746195428808
	ReplicaAddress      : PartitionId: 72a0fb3e-53ec-44f2-9983-2f272aca3e38, ReplicaId: 130743746195428808
	ReplicaRole         : Primary
	NodeName            : Node.3
	ReplicaStatus       : Ready
	LastInBuildDuration : 00:00:01
	HealthState         : Warning

	PS C:\> Get-ServiceFabricReplicaHealth 72a0fb3e-53ec-44f2-9983-2f272aca3e38 130743746195428808

	PartitionId           : 72a0fb3e-53ec-44f2-9983-2f272aca3e38
	ReplicaId             : 130743746195428808
	AggregatedHealthState : Warning
	UnhealthyEvaluations  :
                        	Unhealthy event: SourceId='System.RAP', Property='ServiceOpenOperationDuration', HealthState='Warning', ConsiderWarningAsError=false.

	HealthEvents          :
                        	SourceId              : System.RA
                        	Property              : State
                        	HealthState           : Ok
                        	SequenceNumber        : 130743756170185892
                        	SentAt                : 4/24/2015 7:00:17 PM
                        	ReceivedAt            : 4/24/2015 7:00:33 PM
                        	TTL                   : Infinite
                        	Description           : Replica has been created.
                        	RemoveWhenExpired     : False
                        	IsExpired             : False
                        	Transitions           : ->Ok = 4/24/2015 7:00:33 PM

                        	SourceId              : System.RAP
                        	Property              : ServiceOpenOperationDuration
                        	HealthState           : Warning
                        	SequenceNumber        : 130743756399407044
                        	SentAt                : 4/24/2015 7:00:39 PM
                        	ReceivedAt            : 4/24/2015 7:00:59 PM
                        	TTL                   : Infinite
                        	Description           : Start Time (UTC): 2015-04-24 19:00:17.019
                        	RemoveWhenExpired     : False
                        	IsExpired             : False
                        	Transitions           : ->Warning = 4/24/2015 7:00:59 PM


在调试程序中启动有故障的应用程序时，诊断事件窗口显示 RunAsync 引发的异常：

![Visual Studio 2015 诊断事件：RunAsync 在 fabric:/HelloWorldStatefulApplication 中失败。][1]

Visual Studio 2015 诊断事件：RunAsync 在 **fabric:/HelloWorldStatefulApplication** 中失败。

[1]: ./media/service-fabric-understand-and-troubleshoot-with-system-health-reports/servicefabric-health-vs-runasync-exception.png


### <a name="replication-queue-full"></a>复制队列已满
**System.Replicator** 报告警告。 主副本上一个或多个辅助副本确认操作的速度较慢，通常会发生这种情况。 辅助副本上服务应用操作的速度较慢时，通常会发生这种情况。 队列不再满时，警告会被清除。

* **SourceId**：System.Replicator
* **属性**：**PrimaryReplicationQueueStatus** 或 **SecondaryReplicationQueueStatus**，视副本角色而定

### <a name="slow-naming-operations"></a>命名操作速度慢
当命名操作所花时间过长而导致无法接受时，**System.NamingService** 会报告其主副本的运行状况。 [CreateServiceAsync](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.servicemanagementclient.createserviceasync) 或 [DeleteServiceAsync](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.servicemanagementclient.deleteserviceasync) 都是命名操作的示例。 在 FabricClient 下可找到更多方法，例如，可在[服务管理方法](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.servicemanagementclient)或[属性管理方法](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.propertymanagementclient)下找到更多方法。

> [AZURE.NOTE]
> 命名服务将服务名称解析到群集中的某个位置，并允许用户管理服务名称和属性。 它是一个 Service Fabric 分区型持久服务。 其中一个分区代表“颁发机构所有者”，内含与所有 Service Fabric 名称和服务相关的元数据。 Service Fabric 名称映射到不同的分区，这些分区称为“名称所有者”分区，因此该服务是可扩展的。 阅读有关[命名服务](/documentation/articles/service-fabric-architecture/)的更多内容。
> 
> 

某个命名操作所需时间超出预期时，会在为操作提供服务的命名服务分区的主副本上使用警告报告对该操作进行标记。如果操作成功完成，警告会被清除。如果操作在完成时出现错误，则运行状况报告中会包括有关该错误的详细信息。

* **SourceId**：System.NamingService
* **属性**：以前缀 **Duration_** 开头，用于确定速度过慢的操作以及在其上应用该操作的 Service Fabric 名称。 例如，如果使用名称 fabric:/MyApp/MyService 创建服务的操作耗时过长，则属性为 Duration_AOCreateService.fabric:/MyApp/MyService。 AO 指向此名称和操作的命名分区的角色。
* **后续步骤**：查看命名操作失败的原因。 每个操作可能会有不同的根本原因。 例如，删除服务可能会在某个节点上受阻，因为应用程序主机总是在某个节点上崩溃，原因是服务代码中存在用户 Bug。

以下示例显示了创建服务操作。该操作花的时间超过配置的持续时间。AO 会重试并将工作发送到 NO。NO 在完成上一个操作时出现超时。这种情况下，同一个副本对于 AO 和 NO 角色来说都是主副本。


	PartitionId           : 00000000-0000-0000-0000-000000001000
	ReplicaId             : 131064359253133577
	AggregatedHealthState : Warning
	UnhealthyEvaluations  : 
	                        Unhealthy event: SourceId='System.NamingService', Property='Duration_AOCreateService.fabric:/MyApp/MyService', HealthState='Warning', ConsiderWarningAsError=false.
                        
	HealthEvents          : 
	                        SourceId              : System.RA
	                        Property              : State
	                        HealthState           : Ok
	                        SequenceNumber        : 131064359308715535
	                        SentAt                : 4/29/2016 8:38:50 PM
	                        ReceivedAt            : 4/29/2016 8:39:08 PM
	                        TTL                   : Infinite
	                        Description           : Replica has been created.
	                        RemoveWhenExpired     : False
	                        IsExpired             : False
	                        Transitions           : Error->Ok = 4/29/2016 8:39:08 PM, LastWarning = 1/1/0001 12:00:00 AM
                        
	                        SourceId              : System.NamingService
	                        Property              : Duration_AOCreateService.fabric:/MyApp/MyService
	                        HealthState           : Warning
	                        SequenceNumber        : 131064359526778775
	                        SentAt                : 4/29/2016 8:39:12 PM
	                        ReceivedAt            : 4/29/2016 8:39:38 PM
	                        TTL                   : 00:05:00
	                        Description           : The AOCreateService started at 2016-04-29 20:39:08.677 is taking longer than 30.000.
	                        RemoveWhenExpired     : True
	                        IsExpired             : False
	                        Transitions           : Error->Warning = 4/29/2016 8:39:38 PM, LastOk = 1/1/0001 12:00:00 AM
                        
	                        SourceId              : System.NamingService
	                        Property              : Duration_NOCreateService.fabric:/MyApp/MyService
	                        HealthState           : Warning
	                        SequenceNumber        : 131064360657607311
	                        SentAt                : 4/29/2016 8:41:05 PM
	                        ReceivedAt            : 4/29/2016 8:41:08 PM
	                        TTL                   : 00:00:15
	                        Description           : The NOCreateService started at 2016-04-29 20:39:08.689 completed with FABRIC_E_TIMEOUT in more than 30.000.
	                        RemoveWhenExpired     : True
	                        IsExpired             : False
	                        Transitions           : Error->Warning = 4/29/2016 8:39:38 PM, LastOk = 1/1/0001 12:00:00 AM

## <a name="deployedapplication-system-health-reports"></a>DeployedApplication 系统运行状况报告
**System.Hosting** 是已部署实体上的主管组件。

### <a name="activation"></a>激活
应用程序在节点上成功激活时，System.Hosting 报告正常。 否则报告错误。

* **SourceId**：System.Hosting
* **属性**：Activation，包括滚动更新版本
* **后续步骤**：如果应用程序不正常，则调查激活失败的原因。

以下示例显示成功激活：


	PS C:\> Get-ServiceFabricDeployedApplicationHealth -NodeName Node.1 -ApplicationName fabric:/WordCount

	ApplicationName                    : fabric:/WordCount
	NodeName                           : Node.1
	AggregatedHealthState              : Ok
	DeployedServicePackageHealthStates :
	                                     ServiceManifestName   : WordCountServicePkg
	                                     NodeName              : Node.1
	                                     AggregatedHealthState : Ok

	HealthEvents                       :
	                                     SourceId              : System.Hosting
	                                     Property              : Activation
	                                     HealthState           : Ok
	                                     SequenceNumber        : 130743727751144415
	                                     SentAt                : 4/24/2015 6:12:55 PM
	                                     ReceivedAt            : 4/24/2015 6:13:03 PM
	                                     TTL                   : Infinite
	                                     Description           : The application was activated successfully.
	                                     RemoveWhenExpired     : False
	                                     IsExpired             : False
	                                     Transitions           : ->Ok = 4/24/2015 6:13:03 PM

### <a name="download"></a>下载
**System.Hosting** 报告错误。

* **SourceId**：System.Hosting
* **属性**：**Download:*RolloutVersion***
* **后续步骤**：调查在节点上下载失败的原因。

## <a name="deployedservicepackage-system-health-reports"></a>DeployedServicePackage 系统运行状况报告
**System.Hosting** 是已部署实体上的主管组件。

### <a name="service-package-activation"></a>服务包激活
如果服务包在节点上成功激活，则 System.Hosting 报告正常。 否则报告错误。

* **SourceId**：System.Hosting
* **属性**：Activation
* **后续步骤**：调查激活失败的原因。

### <a name="code-package-activation"></a>代码包激活
**System.Hosting** 报告正常。 如果激活失败，则报告配置的警告。 如果 **CodePackage** 无法激活，或者由于错误数超过配置的 **CodePackageHealthErrorThreshold** 而终止，则 Hosting 报告错误。 如果服务包中有多个代码包，则为每个包生成激活报告。

* **SourceId**：System.Hosting
* **属性**：使用前缀 **CodePackageActivation**，并以 **CodePackageActivation:*CodePackageName*:*SetupEntryPoint/EntryPoint*** 的形式包含代码包的名称和入口点（例如，**CodePackageActivation:Code:SetupEntryPoint**）

### <a name="service-type-registration"></a>服务类型注册
如果服务类型注册成功，则 **System.Hosting** 报告正常。 如果未按时完成注册（时间通过 **ServiceTypeRegistrationTimeout** 配置），则报告错误。 如果因为运行时已关闭而导致服务类型从节点注销， Hosting 会报告警告。

* **SourceId**：System.Hosting
* **属性**：使用前缀 **ServiceTypeRegistration**，并包含服务类型名称（例如，**ServiceTypeRegistration:FileStoreServiceType**）

以下示例显示了一个正常的已部署服务包：


	PS C:\> Get-ServiceFabricDeployedServicePackageHealth -NodeName Node.1 -ApplicationName fabric:/WordCount -ServiceManifestName WordCountServicePkg


	ApplicationName       : fabric:/WordCount
	ServiceManifestName   : WordCountServicePkg
	NodeName              : Node.1
	AggregatedHealthState : Ok
	HealthEvents          :
                        	SourceId              : System.Hosting
                        	Property              : Activation
                        	HealthState           : Ok
                        	SequenceNumber        : 130743727751456915
                        	SentAt                : 4/24/2015 6:12:55 PM
                        	ReceivedAt            : 4/24/2015 6:13:03 PM
                        	TTL                   : Infinite
                        	Description           : The ServicePackage was activated successfully.
                        	RemoveWhenExpired     : False
                        	IsExpired             : False
                        	Transitions           : ->Ok = 4/24/2015 6:13:03 PM

                        	SourceId              : System.Hosting
                        	Property              : CodePackageActivation:Code:EntryPoint
                        	HealthState           : Ok
                        	SequenceNumber        : 130743727751613185
                        	SentAt                : 4/24/2015 6:12:55 PM
                        	ReceivedAt            : 4/24/2015 6:13:03 PM
                        	TTL                   : Infinite
                        	Description           : The CodePackage was activated successfully.
                        	RemoveWhenExpired     : False
                        	IsExpired             : False
                        	Transitions           : ->Ok = 4/24/2015 6:13:03 PM

                        	SourceId              : System.Hosting
                        	Property              : ServiceTypeRegistration:WordCountServiceType
                        	HealthState           : Ok
                        	SequenceNumber        : 130743727753644473
                        	SentAt                : 4/24/2015 6:12:55 PM
                        	ReceivedAt            : 4/24/2015 6:13:03 PM
                        	TTL                   : Infinite
                        	Description           : The ServiceType was registered successfully.
                        	RemoveWhenExpired     : False
                        	IsExpired             : False
                        	Transitions           : ->Ok = 4/24/2015 6:13:03 PM

### <a name="download"></a>下载
**System.Hosting** 报告错误。

* **SourceId**：System.Hosting
* **属性**：**Download:*RolloutVersion***
* **后续步骤**：调查在节点上下载失败的原因。

### <a name="upgrade-validation"></a>升级验证
**System.Hosting** 报告错误。

* **SourceId**：System.Hosting
* **属性**：使用前缀 **FabricUpgradeValidation**，并包含升级版本
* **说明**：指向遇到的错误

## <a name="next-steps"></a>后续步骤
[查看 Service Fabric 运行状况报告](/documentation/articles/service-fabric-view-entities-aggregated-health/)

[如何报告和检查服务运行状况](/documentation/articles/service-fabric-diagnostics-how-to-report-and-check-service-health/)

[在本地监视和诊断服务](/documentation/articles/service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally/)

[Service Fabric 应用程序升级](/documentation/articles/service-fabric-application-upgrade/)
<!--Update_Description:wording update;add anchors to subtitles-->