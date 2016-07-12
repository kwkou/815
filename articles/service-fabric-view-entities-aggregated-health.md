<properties
   pageTitle="如何查看 Azure Service Fabric 实体的聚合运行状况 | Azure"
   description="说明如何通过运行状况查询和常规查询来查询、查看和评估 Azure Service Fabric 实体聚合运行状况。"
   services="service-fabric"
   documentationCenter=".net"
   authors="oanapl"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.date="04/25/2016"
   wacn.date="07/04/2016"/>

# 查看 Service Fabric 运行状况报告
Azure Service Fabric 引入了一种由运行状况实体组成的[运行状况模型](/documentation/articles/service-fabric-health-introduction/)，系统组件和监视器可以在其上报告它们监视的本地状况。[运行状况存储](/documentation/articles/service-fabric-health-introduction/#health-store)聚合所有运行状况数据以确定实体是否正常运行。

根据现有设定，群集会被系统组件发送的运行状况报告填充。从[使用系统运行状况报告进行故障排除](/documentation/articles/service-fabric-understand-and-troubleshoot-with-system-health-reports/)了解更多信息。

Service Fabric 提供多种方式来获取实体聚合运行状况：

- [Service Fabric 资源管理器](/documentation/articles/service-fabric-visualizing-your-cluster/)或其他可视化工具

- 运行状况查询（通过 PowerShell、API 或 REST）

- 常规查询，返回将运行状况作为属性之一的实体的列表（通过 PowerShell、API 或 REST）

为了演示这些选项，让我们使用一个具有五个节点的本地群集。**fabric:/System** 应用程序（原本即已存在）旁边，已部署其他一些应用程序。其中之一是 **fabric:/WordCount**。该应用程序包含一个配置有七个副本的有状态服务。由于只有五个节点，因此系统组件显示分区低于目标计数的警告。

```xml
<Service Name="WordCountService">
    <StatefulService ServiceTypeName="WordCountServiceType" TargetReplicaSetSize="7" MinReplicaSetSize="2">
      <UniformInt64Partition PartitionCount="1" LowKey="1" HighKey="26" />
    </StatefulService>
</Service>
```

## Service Fabric 资源管理器中的运行状况
Service Fabric 资源管理器提供群集的更直观展示。在下图中，你可以看到：

- 应用程序 fabric:/WordCount 为红色（出错），因为 MyWatchdog 报告“可用性”属性有一个错误事件。

- 其服务之一，**fabric:/WordCount/WordCountService** 为黄色（警告）。如上所述，服务配置有七个副本，这些副本无法全部放置（因为只有五个节点）。尽管此处未显示，因为系统报告，服务分区为黄色。黄色分区触发黄色服务。

- 由于应用程序为红色，因此群集为红色。

评估使用群集清单和应用程序清单的默认策略。它们是严格的策略，不容许任何失败。

使用 Service Fabric 资源管理器查看群集：

![使用 Service Fabric 资源管理器查看群集。][1]

[1]: ./media/service-fabric-view-entities-aggregated-health/servicefabric-explorer-cluster-health.png


> [AZURE.NOTE] 了解有关 [Service Fabric 资源管理器](/documentation/articles/service-fabric-visualizing-your-cluster/)的更多信息。

## 运行状况查询
Service Fabric 为每个支持的[实体类型](/documentation/articles/service-fabric-health-introduction/#health-entities-and-hierarchy)提供运行状况查询。可以通过 API（可在 FabricClient.HealthManager 中找到的方法）、PowerShell cmdlet 和 REST 访问它们。这些查询返回有关实体的完整运行状况信息，包括聚合运行状况、报告的实体上的运行状况事件、子运行状况（在适用时）以及实体不正常时的不正常评估。

> [AZURE.NOTE] 完全填充运行状况存储中时，运行状况实体返回给用户。实体必须是作用中（未删除），并且具有系统报告。层次结构链上其父实体还必须有系统报告。如果不满足上述条件之一，则运行状况查询返回一个异常，并显示未返回实体的原因。

运行状况查询必须传递取决于实体类型的实体标识符。这些查询接受可选的运行状况策略参数。如果未指定这些参数，则使用来自群集清单或应用程序清单的[运行状况策略](/documentation/articles/service-fabric-health-introduction/#health-policies)进行评估。这些查询还接受筛选器，以仅返回与指定筛选器有关的部分子项或事件。

> [AZURE.NOTE] 在服务器端应用输出筛选器，因此减小了消息回复大小。我们建议使用输出筛选器来限制返回的数据，而不是在客户端应用筛选器。

实体运行状况包含以下信息：

- 实体的聚合运行状况状态。这由运行状况存储依据实体运行状况报告、子项运行状况（在适用时）和运行状况策略计算。了解有关[实体运行状况评估](/documentation/articles/service-fabric-health-introduction/#entity-health-evaluation)的详细信息。  

- 实体上的运行状况事件。

- 对于能够拥有子项的实体，为所有子项的运行状况集合。运行状况包含实体标识符和聚合的运行状况。若要获取某个子项的完整运行状况，请调用子实体类型的运行状况查询，并传递子实体标识符。

- 如果实体不正常，指向触发实体状态的报告的不正常评估。

## 获取群集运行状况
返回群集实体的运行状况，并包含应用程序和节点（群集的子项）的运行状况。输入：

- [可选] 用于评估节点和群集事件的群集运行状况策略。

- [可选] 应用程序运行状况策略与用于取代应用程序清单策略的运行状况策略进行映射。

- [可选] 事件、节点和应用程序的筛选，指定对那些项目感到兴趣，并且应该在结果中返回（例如，仅限错误、或警告和错误）。请注意，所有事件、节点及应用程序都用于评估实体聚合运行状况，无论筛选器为何。

### API
若要获取群集运行状况，请创建 **FabricClient** 并在其 **HealthManager** 上调用 [**GetClusterHealthAsync**](https://msdn.microsoft.com/zh-cn/library/azure/system.fabric.fabricclient.healthclient.getclusterhealthasync.aspx) 方法。

以下代码获取群集运行状况：

```csharp
ClusterHealth clusterHealth = await fabricClient.HealthManager.GetClusterHealthAsync();
```

以下代码使用针对节点和应用程序的自定义运行状况策略和筛选器获取群集运行状况。请注意，它将创建包含所有输入数据的 [ClusterHealthQueryDescription](https://msdn.microsoft.com/zh-cn/library/azure/system.fabric.description.clusterhealthquerydescription.aspx)**。

```csharp
var policy = new ClusterHealthPolicy()
{
    MaxPercentUnhealthyNodes = 20
};
var nodesFilter = new NodeHealthStatesFilter()
{
    HealthStateFilterValue = HealthStateFilter.Error | HealthStateFilter.Warning
};
var applicationsFilter = new ApplicationHealthStatesFilter()
{
    HealthStateFilterValue = HealthStateFilter.Error
};
var queryDescription = new ClusterHealthQueryDescription()
{
    HealthPolicy = policy,
    ApplicationsFilter = applicationsFilter,
    NodesFilter = nodesFilter,
};

ClusterHealth clusterHealth = await fabricClient.HealthManager.GetClusterHealthAsync(queryDescription);
```

### PowerShell
用于获取群集运行状况的 cmdlet 为 [Get-ServiceFabricClusterHealth](https://msdn.microsoft.com/zh-cn/library/mt125850.aspx)。首先使用 [Connect-ServiceFabricCluster](https://msdn.microsoft.com/zh-cn/library/mt125938.aspx) cmdlet 连接到群集。

群集的状态：五个节点，系统应用程序和 fabric:/WordCount 按如上配置。

以下 cmdlet 使用默认运行状况策略获取群集运行状况。聚合的运行状况为警告，因为 fabric:/WordCount 应用程序处于警告状态。请注意不正常评估如何提供触发聚合运行状况的详细条件。

	PS C:\> Get-ServiceFabricClusterHealth

	AggregatedHealthState   : Warning
	UnhealthyEvaluations    :
                          	Unhealthy applications: 100% (1/1), MaxPercentUnhealthyApplications=0%.

                          	Unhealthy application: ApplicationName='fabric:/WordCount', AggregatedHealthState='Warning'.

                              	Unhealthy services: 100% (1/1), ServiceType='WordCountServiceType', MaxPercentUnhealthyServices=0%.

                              	Unhealthy service: ServiceName='fabric:/WordCount/WordCountService', AggregatedHealthState='Warning'.

                                  	Unhealthy event: SourceId='System.PLB',
                          	Property='ServiceReplicaUnplacedHealth_Secondary_a1f83a35-d6bf-4d39-b90d-28d15f39599b', HealthState='Warning',
                          	ConsiderWarningAsError=false.


	NodeHealthStates        :
                          	NodeName              : _Node_2
                          	AggregatedHealthState : Ok

                          	NodeName              : _Node_0
                          	AggregatedHealthState : Ok

                          	NodeName              : _Node_1
                          	AggregatedHealthState : Ok

                          	NodeName              : _Node_3
                          	AggregatedHealthState : Ok

                          	NodeName              : _Node_4
                          	AggregatedHealthState : Ok

	ApplicationHealthStates :
                          	ApplicationName       : fabric:/System
                          	AggregatedHealthState : Ok

                          	ApplicationName       : fabric:/WordCount
                          	AggregatedHealthState : Warning

	HealthEvents            : None


以下 PowerShell cmdlet 使用自定义应用程序策略获取群集的运行状况。它筛选结果以只获取有错误或警告的应用程序和节点。因此，不会返回任何节点，因为这些节点都是正常的。仅 fabric:/WordCount 应用程序符合应用程序筛选器。因为自定义策略指定对于 fabric:/WordCount 应用程序将警告视为错误，应用程序被评估为错误，从而群集也被评估为错误。


	PS c:> $appHealthPolicy = New-Object -TypeName System.Fabric.Health.ApplicationHealthPolicy
	$appHealthPolicy.ConsiderWarningAsError = $true
	$appHealthPolicyMap = New-Object -TypeName System.Fabric.Health.ApplicationHealthPolicyMap
	$appUri1 = New-Object -TypeName System.Uri -ArgumentList "fabric:/WordCount"
	$appHealthPolicyMap.Add($appUri1, $appHealthPolicy)
	Get-ServiceFabricClusterHealth -ApplicationHealthPolicyMap $appHealthPolicyMap -ApplicationsFilter "Warning,Error" -NodesFilter "Warning,Error"


	AggregatedHealthState   : Error
	UnhealthyEvaluations    :
                          		Unhealthy applications: 100% (1/1), MaxPercentUnhealthyApplications=0%.

                          		Unhealthy application: ApplicationName='fabric:/WordCount', AggregatedHealthState='Error'.

                              		Unhealthy services: 100% (1/1), ServiceType='WordCountServiceType', MaxPercentUnhealthyServices=0%.

                              		Unhealthy service: ServiceName='fabric:/WordCount/WordCountService', AggregatedHealthState='Error'.

                                  		Unhealthy event: SourceId='System.PLB',
                          		Property='ServiceReplicaUnplacedHealth_Secondary_a1f83a35-d6bf-4d39-b90d-28d15f39599b', HealthState='Warning',
                          		ConsiderWarningAsError=true.


	NodeHealthStates        : None
	ApplicationHealthStates :
                          		ApplicationName       : fabric:/WordCount
                          		AggregatedHealthState : Error

	HealthEvents            : None



## 获取节点运行状况
返回节点实体的运行状况，并包含针对该节点报告的运行状况事件。输入：

- [必需] 标识该节点的节点名称。

- [可选 ] 用于评估运行状况的群集运行状况策略设置。

- [可选] 事件的筛选，指定对那些项目感到兴趣，并且应该在结果中返回（例如，仅限错误、或警告和错误）。请注意，所有事件都用于评估实体聚合运行状况，无论筛选器为何。

### API
若要通过 API 获取节点运行状况，请创建 `FabricClient` 并在其 HealthManager 上调用 [GetNodeHealthAsync](https://msdn.microsoft.com/zh-cn/library/azure/system.fabric.fabricclient.healthclient.getnodehealthasync.aspx) 方法。

以下代码获取具有指定节点名称的节点运行状况：

```csharp
NodeHealth nodeHealth = await fabricClient.HealthManager.GetNodeHealthAsync(nodeName);
```

以下代码获取指定节点名称的节点运行状况，并通过 [NodeHealthQueryDescription](https://msdn.microsoft.com/zh-cn/library/azure/system.fabric.description.nodehealthquerydescription.aspx)** 传入事件筛选器和自定义策略：

```csharp
var queryDescription = new NodeHealthQueryDescription(nodeName)
{
    HealthPolicy = new ClusterHealthPolicy() {  ConsiderWarningAsError = true },
    EventsFilter = new HealthEventsFilter() { HealthStateFilterValue = HealthStateFilter.Warning },
};

NodeHealth nodeHealth = await fabricClient.HealthManager.GetNodeHealthAsync(queryDescription);
```

### PowerShell
用于获取节点运行状况的 cmdlet 为 [Get-ServiceFabricNodeHealth](https://msdn.microsoft.com/zh-cn/library/mt125937.aspx)。首先使用 [Connect-ServiceFabricCluster](https://msdn.microsoft.com/zh-cn/library/mt125938.aspx) cmdlet 连接到群集。以下 cmdlet 使用默认运行状况策略获取节点运行状况：

```powershell
PS C:\> Get-ServiceFabricNodeHealth _Node_1


NodeName              : _Node_1
AggregatedHealthState : Ok
HealthEvents          :
                        SourceId              : System.FM
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 6
                        SentAt                : 3/22/2016 7:47:56 PM
                        ReceivedAt            : 3/22/2016 7:48:19 PM
                        TTL                   : Infinite
                        Description           : Fabric node is up.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 3/22/2016 7:48:19 PM, LastWarning = 1/1/0001 12:00:00 AM
```

以下 cmdlet 获取群集中所有节点的运行状况：

```powershell
PS C:\> Get-ServiceFabricNode | Get-ServiceFabricNodeHealth | select NodeName, AggregatedHealthState | ft -AutoSize

NodeName AggregatedHealthState
-------- ---------------------
_Node_2                     Ok
_Node_0                     Ok
_Node_1                     Ok
_Node_3                     Ok
_Node_4                     Ok
```

## 获取应用程序运行状况
返回一个应用程序实体的运行状况。包含已部署应用程序和服务子项的运行状况。输入：

- [必需] 标识应用程序的应用程序名称 (URI)。

- [可选] 用于取代应用程序清单策略的应用程序运行状况策略。

- [可选] 事件、服务和已部署应用程序的筛选，指定对那些项目感到兴趣，并且应该在结果中返回（例如，仅限错误、或警告和错误）。请注意，所有事件、服务和已部署应用程序的都用于评估实体聚合运行状况，无论筛选器为何。

### API
若要获取应用程序运行状况，请创建 `FabricClient` 并在其 HealthManager 上调用 [GetApplicationHealthAsync](https://msdn.microsoft.com/zh-cn/library/azure/system.fabric.fabricclient.healthclient.getapplicationhealthasync.aspx) 方法。

以下代码获取具有指定应用程序名称 (URI) 的应用程序的运行状况：

```csharp
ApplicationHealth applicationHealth = await fabricClient.HealthManager.GetApplicationHealthAsync(applicationName);
```

以下代码使用通过 [ApplicationHealthQueryDescription](https://msdn.microsoft.com/zh-cn/library/azure/system.fabric.description.applicationhealthquerydescription.aspx) 指定的筛选器和自定义策略，获取指定应用程序名称 (URI) 的应用程序运行状况。

```csharp
HealthStateFilter warningAndErrors = HealthStateFilter.Error | HealthStateFilter.Warning;
var serviceTypePolicy = new ServiceTypeHealthPolicy()
{
    MaxPercentUnhealthyPartitionsPerService = 0,
    MaxPercentUnhealthyReplicasPerPartition = 5,
    MaxPercentUnhealthyServices = 0,
};
var policy = new ApplicationHealthPolicy()
{
    ConsiderWarningAsError = false,
    DefaultServiceTypeHealthPolicy = serviceTypePolicy,
    MaxPercentUnhealthyDeployedApplications = 0,
};

var queryDescription = new ApplicationHealthQueryDescription(applicationName)
{
    HealthPolicy = policy,
    EventsFilter = new HealthEventsFilter() { HealthStateFilterValue = warningAndErrors },
    ServicesFilter = new ServiceHealthStatesFilter() { HealthStateFilterValue = warningAndErrors },
    DeployedApplicationsFilter = new DeployedApplicationHealthStatesFilter() { HealthStateFilterValue = warningAndErrors },
};

ApplicationHealth applicationHealth = await fabricClient.HealthManager.GetApplicationHealthAsync(queryDescription);
```

### PowerShell
用于获取应用程序运行状况的 cmdlet 为 [Get-ServiceFabricApplicationHealth](https://msdn.microsoft.com/zh-cn/library/mt125976.aspx)。首先使用 [Connect-ServiceFabricCluster](https://msdn.microsoft.com/zh-cn/library/mt125938.aspx) cmdlet 连接到群集。

以下 cmdlet 返回 **fabric:/WordCount** 应用程序的运行状况：


	PS c:>
	PS C:\WINDOWS\system32>  Get-ServiceFabricApplicationHealth fabric:/WordCount


	ApplicationName                 : fabric:/WordCount
	AggregatedHealthState           : Warning
	UnhealthyEvaluations            :
                                  		Unhealthy services: 100% (1/1), ServiceType='WordCountServiceType', MaxPercentUnhealthyServices=0%.

                                  		Unhealthy service: ServiceName='fabric:/WordCount/WordCountService', AggregatedHealthState='Warning'.

                                      		Unhealthy event: SourceId='System.PLB',
                                  		Property='ServiceReplicaUnplacedHealth_Secondary_a1f83a35-d6bf-4d39-b90d-28d15f39599b', HealthState='Warning',
                                  		ConsiderWarningAsError=false.

	ServiceHealthStates             :
                                  		ServiceName           : fabric:/WordCount/WordCountService
                                  		AggregatedHealthState : Warning

                                  		ServiceName           : fabric:/WordCount/WordCountWebService
                                  		AggregatedHealthState : Ok

	DeployedApplicationHealthStates :
                                  		ApplicationName       : fabric:/WordCount
                                  		NodeName              : _Node_0
                                  		AggregatedHealthState : Ok

                                  		ApplicationName       : fabric:/WordCount
                                  		NodeName              : _Node_2
                                  		AggregatedHealthState : Ok

                                  		ApplicationName       : fabric:/WordCount
                                  		NodeName              : _Node_3
                                  		AggregatedHealthState : Ok

                                  		ApplicationName       : fabric:/WordCount
                                  		NodeName              : _Node_4
                                  		AggregatedHealthState : Ok

                                  		ApplicationName       : fabric:/WordCount
                                  		NodeName              : _Node_1
                                  		AggregatedHealthState : Ok

	HealthEvents                    :
                                  		SourceId              : System.CM
                                  		Property              : State
                                  		HealthState           : Ok
                                  		SequenceNumber        : 360
                                  		SentAt                : 3/22/2016 7:56:53 PM
                                  		ReceivedAt            : 3/22/2016 7:56:53 PM
                                  		TTL                   : Infinite
                                  		Description           : Application has been created.
                                  		RemoveWhenExpired     : False
                                  		IsExpired             : False
                                  		Transitions           : Error->Ok = 3/22/2016 7:56:53 PM, LastWarning = 1/1/0001 12:00:00 AM

                                  		SourceId              : MyWatchdog
                                  		Property              : Availability
                                  		HealthState           : Ok
                                  		SequenceNumber        : 131031545225930951
                                  		SentAt                : 3/22/2016 9:08:42 PM
                                  		ReceivedAt            : 3/22/2016 9:08:42 PM
                                  		TTL                   : Infinite
                                  		Description           : Availability checked successfully, latency ok
                                  		RemoveWhenExpired     : False
                                  		IsExpired             : False
                                  		Transitions           : Error->Ok = 3/22/2016 8:55:39 PM, LastWarning = 1/1/0001 12:00:00 AM


以下 PowerShell cmdlet 传入自定义策略。它还筛选子项和事件。


	PS C:\> Get-ServiceFabricApplicationHealth -ApplicationName fabric:/WordCount -ConsiderWarningAsError $true -ServicesFilter Error -EventsFilter Error -DeployedApplicationsFilter Error

	ApplicationName                 : fabric:/WordCount
	AggregatedHealthState           : Error
	UnhealthyEvaluations            :
                                  		Unhealthy services: 100% (1/1), ServiceType='WordCountServiceType', MaxPercentUnhealthyServices=0%.

                                  		Unhealthy service: ServiceName='fabric:/WordCount/WordCountService', AggregatedHealthState='Error'.

                                      		Unhealthy event: SourceId='System.PLB',
                                  		Property='ServiceReplicaUnplacedHealth_Secondary_a1f83a35-d6bf-4d39-b90d-28d15f39599b', HealthState='Warning',
                                  		ConsiderWarningAsError=true.

	ServiceHealthStates             :
                                  		ServiceName           : fabric:/WordCount/WordCountService
                                  		AggregatedHealthState : Error

	DeployedApplicationHealthStates : None
	HealthEvents                    : None


## 获取服务运行状况
返回一个服务实体的运行状况。包含分区运行状况。输入：

- [必需] 标识服务的服务名称 (URI)。

- [可选] 用于取代应用程序清单策略的应用程序运行状况策略。

- [可选] 事件和分区的筛选，指定对那些项目感到兴趣，并且应该在结果中返回（例如，仅限错误、或警告和错误）。请注意，所有事件和分区都用于评估实体聚合运行状况，无论筛选器为何。

### API
若要通过 API 获取服务运行状况，请创建 `FabricClient` 并在其 HealthManager 上调用 [GetServiceHealthAsync](https://msdn.microsoft.com/zh-cn/library/azure/system.fabric.fabricclient.healthclient.getservicehealthasync.aspx) 方法。

以下示例获取具有指定服务名称 (URI) 的服务的运行状况：


	ServiceHealth serviceHealth = await fabricClient.HealthManager.GetServiceHealthAsync(serviceName);


以下代码通过 [ServiceHealthQueryDescription](https://msdn.microsoft.com/zh-cn/library/azure/system.fabric.description.servicehealthquerydescription.aspx) 指定筛选器和自定义策略，从而获取指定服务名称 (URI) 的服务运行状况。

```csharp
var queryDescription = new ServiceHealthQueryDescription(serviceName)
{
    EventsFilter = new HealthEventsFilter() { HealthStateFilterValue = HealthStateFilter.All },
    PartitionsFilter = new PartitionHealthStatesFilter() { HealthStateFilterValue = HealthStateFilter.Error },
};

ServiceHealth serviceHealth = await fabricClient.HealthManager.GetServiceHealthAsync(queryDescription);
```

### PowerShell
用于获取服务运行状况的 cmdlet 为 [Get-ServiceFabricServiceHealth](https://msdn.microsoft.com/zh-cn/library/mt125984.aspx)。首先使用 [Connect-ServiceFabricCluster](https://msdn.microsoft.com/zh-cn/library/mt125938.aspx) cmdlet 连接到群集。

以下 cmdlet 使用默认运行状况策略获取服务运行状况：


	PS C:\> Get-ServiceFabricServiceHealth -ServiceName fabric:/WordCount/WordCountService


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
                        	SequenceNumber        : 131031547693687021
                        	SentAt                : 3/22/2016 9:12:49 PM
                        	ReceivedAt            : 3/22/2016 9:12:49 PM
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


## 获取分区运行状况
返回一个分区实体的运行状况。包含副本运行状况。输入：

- [必需] 标识分区的分区 ID (GUID)。

- [可选] 用于取代应用程序清单策略的应用程序运行状况策略。

- [可选] 事件和副本的筛选，指定对那些项目感到兴趣，并且应该在结果中返回（例如，仅限错误、或警告和错误）。请注意，所有事件和副本都用于评估实体聚合运行状况，无论筛选器为何。

### API
若要通过 API 获取分区运行状况，请创建 `FabricClient` 并在其 HealthManager 上调用 [GetPartitionHealthAsync](https://msdn.microsoft.com/zh-cn/library/azure/system.fabric.fabricclient.healthclient.getpartitionhealthasync.aspx) 方法。若要指定可选参数，请创建 [PartitionHealthQueryDescription](https://msdn.microsoft.com/zh-cn/library/azure/system.fabric.description.partitionhealthquerydescription.aspx)。

```csharp
PartitionHealth partitionHealth = await fabricClient.HealthManager.GetPartitionHealthAsync(partitionId);
```

### PowerShell
用于获取分区运行状况的 cmdlet 为 [Get-ServiceFabricPartitionHealth](https://msdn.microsoft.com/zh-cn/library/mt125869.aspx)。首先使用 [Connect-ServiceFabricCluster](https://msdn.microsoft.com/zh-cn/library/mt125938.aspx) cmdlet 连接到群集。

以下 cmdlet 获取 **fabric:/WordCount/WordCountService** 服务的所有分区的运行状况：


	PS C:\> Get-ServiceFabricPartition fabric:/WordCount/WordCountService | Get-ServiceFabricPartitionHealth


	PartitionId           : a1f83a35-d6bf-4d39-b90d-28d15f39599b
	AggregatedHealthState : Warning
	UnhealthyEvaluations  :
                        	Unhealthy event: SourceId='System.FM', Property='State', HealthState='Warning', ConsiderWarningAsError=false.

	ReplicaHealthStates   :
                        	ReplicaId             : 131031502143040223
                        	AggregatedHealthState : Ok

                        	ReplicaId             : 131031502346844060
                        	AggregatedHealthState : Ok

                        	ReplicaId             : 131031502346844059
                        	AggregatedHealthState : Ok

                        	ReplicaId             : 131031502346844061
                        	AggregatedHealthState : Ok

                        	ReplicaId             : 131031502346844058
                        	AggregatedHealthState : Ok

	HealthEvents          :
                        	SourceId              : System.FM
                        	Property              : State
                        	HealthState           : Warning
                        	SequenceNumber        : 76
                        	SentAt                : 3/22/2016 7:57:26 PM
                        	ReceivedAt            : 3/22/2016 7:57:48 PM
                        	TTL                   : Infinite
                        	Description           : Partition is below target replica or instance count.
                        	RemoveWhenExpired     : False
                        	IsExpired             : False
                        	Transitions           : Error->Warning = 3/22/2016 7:57:48 PM, LastOk = 1/1/0001 12:00:00 AM


## 获取副本运行状况
这会返回有状态服务副本或无状态服务实例的运行状况。输入：

- [必需] 分区 ID (GUID) 和标识副本的副本 ID。

- [可选] 用于取代应用程序清单策略的应用程序运行状况策略参数。

- [可选] 事件的筛选，指定对那些项目感到兴趣，并且应该在结果中返回（例如，仅限错误、或警告和错误）。请注意，所有事件都用于评估实体聚合运行状况，无论筛选器为何。

### API
若要通过 API 获取副本运行状况，请创建 `FabricClient` 并在其 HealthManager 上调用 [GetReplicaHealthAsync](https://msdn.microsoft.com/zh-cn/library/azure/system.fabric.fabricclient.healthclient.getreplicahealthasync.aspx) 方法。若要指定高级参数，请使用 [ReplicaHealthQueryDescription](https://msdn.microsoft.com/zh-cn/library/azure/system.fabric.description.replicahealthquerydescription.aspx)。

```csharp
ReplicaHealth replicaHealth = await fabricClient.HealthManager.GetReplicaHealthAsync(partitionId, replicaId);
```

### PowerShell
用于获取副本运行状况的 cmdlet 为 [Get-ServiceFabricReplicaHealth](https://msdn.microsoft.com/zh-cn/library/mt125808.aspx)。首先使用 [Connect-ServiceFabricCluster](https://msdn.microsoft.com/zh-cn/library/mt125938.aspx) cmdlet 连接到群集。

以下 cmdlet 获取服务所有分区的主副本的运行状况：

```powershell
PS C:\> Get-ServiceFabricPartition fabric:/WordCount/WordCountService | Get-ServiceFabricReplica | where {$_.ReplicaRole -eq "Primary"} | Get-ServiceFabricReplicaHealth


PartitionId           : a1f83a35-d6bf-4d39-b90d-28d15f39599b
ReplicaId             : 131031502143040223
AggregatedHealthState : Ok
HealthEvents          :
                        SourceId              : System.RA
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 131031502145556748
                        SentAt                : 3/22/2016 7:56:54 PM
                        ReceivedAt            : 3/22/2016 7:57:12 PM
                        TTL                   : Infinite
                        Description           : Replica has been created.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 3/22/2016 7:57:12 PM, LastWarning = 1/1/0001 12:00:00 AM
```

## 获取已部署应用程序的运行状况
返回部署在节点实体上的一个应用程序的运行状况。包含已部署服务包的运行状况。输入：

- [必需] 标识已部署应用程序的应用程序名称 (URI) 和节点名称（字符串）。

- [可选] 用于取代应用程序清单策略的应用程序运行状况策略。

- [可选] 事件和已部署服务包的筛选，指定对那些条目感到兴趣，并且应该在结果中返回（例如，仅限错误、或警告和错误）。请注意，所有事件和已部署服务包的都用于评估实体聚合运行状况，无论筛选器为何。

### API
若要通过 API 获取部署在节点上的一个应用程序的运行状况，请创建 `FabricClient` 并在其 HealthManager 上调用 [GetDeployedApplicationHealthAsync](https://msdn.microsoft.com/zh-cn/library/azure/system.fabric.fabricclient.healthclient.getdeployedapplicationhealthasync.aspx) 方法。若要指定可选参数，请使用 [DeployedApplicationHealthQueryDescription](https://msdn.microsoft.com/zh-cn/library/azure/system.fabric.description.deployedapplicationhealthquerydescription.aspx)。

```csharp
DeployedApplicationHealth health = await fabricClient.HealthManager.GetDeployedApplicationHealthAsync(
    new DeployedApplicationHealthQueryDescription(applicationName, nodeName));
```

### PowerShell
用于获取已部署应用程序的运行状况的 cmdlet 为 [Get-ServiceFabricDeployedApplicationHealth](https://msdn.microsoft.com/zh-cn/library/mt163523.aspx)。首先使用 [Connect-ServiceFabricCluster](https://msdn.microsoft.com/zh-cn/library/mt125938.aspx) cmdlet 连接到群集。若要了解应用程序的部署位置，请运行 [Get-ServiceFabricApplicationHealth](https://msdn.microsoft.com/zh-cn/library/mt125976.aspx) 并查看已部署应用程序子项。

以下 cmdlet 获取部署在 **\_Node\_2** 上的 **fabric:/WordCount** 应用程序的运行状况。


	PS C:\> Get-ServiceFabricDeployedApplicationHealth -ApplicationName fabric:/WordCount -NodeName _Node_2


	ApplicationName                    : fabric:/WordCount
	NodeName                           : _Node_2
	AggregatedHealthState              : Ok
	DeployedServicePackageHealthStates :
                                     	ServiceManifestName   : WordCountServicePkg
                                     	NodeName              : _Node_2
                                     	AggregatedHealthState : Ok

                                     	ServiceManifestName   : WordCountWebServicePkg
                                     	NodeName              : _Node_2
                                     	AggregatedHealthState : Ok

	HealthEvents                       :
                                     	SourceId              : System.Hosting
                                     	Property              : Activation
                                     	HealthState           : Ok
                                     	SequenceNumber        : 131031502143710698
                                     	SentAt                : 3/22/2016 7:56:54 PM
                                     	ReceivedAt            : 3/22/2016 7:57:12 PM
                                     	TTL                   : Infinite
                                     	Description           : The application was activated successfully.
                                     	RemoveWhenExpired     : False
                                     	IsExpired             : False
                                     	Transitions           : Error->Ok = 3/22/2016 7:57:12 PM, LastWarning = 1/1/0001 12:00:00 AM


## 获取已部署服务包的运行状况
返回一个已部署服务包实体的运行状况。输入：

- [必需] 标识已部署服务包的应用程序名称 (URI)、节点名称（字符串）和服务清单名称（字符串）。

- [可选] 用于取代应用程序清单策略的应用程序运行状况策略。

- [可选] 事件的筛选，指定对那些项目感到兴趣，并且应该在结果中返回（例如，仅限错误、或警告和错误）。请注意，所有事件都用于评估实体聚合运行状况，无论筛选器为何。

### API
若要通过 API 获取一个已部署服务包的运行状况，请创建 `FabricClient` 并在其 HealthManager 上调用 [GetDeployedServicePackageHealthAsync](https://msdn.microsoft.com/zh-cn/library/azure/system.fabric.fabricclient.healthclient.getdeployedservicepackagehealthasync.aspx) 方法。若要指定可选参数，请使用 [DeployedServicePackageHealthQueryDescription](https://msdn.microsoft.com/zh-cn/library/azure/system.fabric.description.deployedservicepackagehealthquerydescription.aspx)。

```csharp
DeployedServicePackageHealth health = await fabricClient.HealthManager.GetDeployedServicePackageHealthAsync(
    new DeployedServicePackageHealthQueryDescription(applicationName, nodeName, serviceManifestName));
```

### PowerShell
用于获取已部署服务包的运行状况的 cmdlet 为 [Get-ServiceFabricDeployedServicePackageHealth](https://msdn.microsoft.com/zh-cn/library/mt163525.aspx)。首先使用 [Connect-ServiceFabricCluster](https://msdn.microsoft.com/zh-cn/library/mt125938.aspx) cmdlet 连接到群集。若要查看应用程序的部署位置，请运行 [Get-ServiceFabricApplicationHealth](https://msdn.microsoft.com/zh-cn/library/mt125976.aspx) 并查看已部署应用程序。若要查看一个应用程序中有哪些服务包，请在 [Get-ServiceFabricDeployedApplicationHealth](https://msdn.microsoft.com/zh-cn/library/mt163523.aspx) 输出中查看已部署服务包子项。

以下 cmdlet 获取部署在 **\_Node\_2** 上的 **fabric:/WordCount** 应用程序的 **WordCountServicePkg** 服务包的运行状况。此实体的 **System.Hosting** 报告包含成功的服务包和入口点激活以及成功的服务类型注册。

```powershell
PS C:\> Get-ServiceFabricDeployedApplication -ApplicationName fabric:/WordCount -NodeName _Node_2 | Get-ServiceFabricDeployedServicePackageHealth -ServiceManifestName WordCountServicePkg


ApplicationName       : fabric:/WordCount
ServiceManifestName   : WordCountServicePkg
NodeName              : _Node_2
AggregatedHealthState : Ok
HealthEvents          :
                        SourceId              : System.Hosting
                        Property              : Activation
                        HealthState           : Ok
                        SequenceNumber        : 131031502301306211
                        SentAt                : 3/22/2016 7:57:10 PM
                        ReceivedAt            : 3/22/2016 7:57:12 PM
                        TTL                   : Infinite
                        Description           : The ServicePackage was activated successfully.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 3/22/2016 7:57:12 PM, LastWarning = 1/1/0001 12:00:00 AM

                        SourceId              : System.Hosting
                        Property              : CodePackageActivation:Code:EntryPoint
                        HealthState           : Ok
                        SequenceNumber        : 131031502301568982
                        SentAt                : 3/22/2016 7:57:10 PM
                        ReceivedAt            : 3/22/2016 7:57:12 PM
                        TTL                   : Infinite
                        Description           : The CodePackage was activated successfully.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 3/22/2016 7:57:12 PM, LastWarning = 1/1/0001 12:00:00 AM

                        SourceId              : System.Hosting
                        Property              : ServiceTypeRegistration:WordCountServiceType
                        HealthState           : Ok
                        SequenceNumber        : 131031502314788519
                        SentAt                : 3/22/2016 7:57:11 PM
                        ReceivedAt            : 3/22/2016 7:57:12 PM
                        TTL                   : Infinite
                        Description           : The ServiceType was registered successfully.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 3/22/2016 7:57:12 PM, LastWarning = 1/1/0001 12:00:00 AM
```

## 运行状况区块查询
运行状况区块查询可以根据输入筛选器返回多级群集子项（以递归方式）。它支持高级筛选器，利用这些筛选器，可以非常灵活地表示要返回的特定子项（由其唯一标识符或其他组标识符和/或健康状况标识）。与始终包含第一级子项的运行状况命令不同的是，它在默认情况下不包含任何子项。

[运行状况查询](/documentation/articles/service-fabric-view-entities-aggregated-health/#health-queries)根据必要筛选器仅返回指定实体的第一级子项。若要获取子项的子项，用户必须调用每个相关实体的附加运行状况 API。同样地，若要获取特定实体的运行状况，用户必须调用每个所需实体的一个运行状况 API。区块查询高级筛选可让用户在一个查询中请求多个相关项目，将消息大小和消息数目降至最低。

区块查询的值是用户可以在一个调用中获取多个群集实体（可能是从必要的根开始的所有群集实体）的健康状况。你可以如下表示复杂的运行状况查询：

- 仅返回状态为错误的应用程序，并且针对这些应用程序，包含所有状态为警告|错误的服务。针对返回的服务，包含所有分区。

- 仅返回 4 个应用程序的运行状况，由其名称指定。

- 仅返回所需应用程序类型的应用程序运行状况。

- 返回某个节点上所有已部署的实体。这样会返回所有应用程序，指定节点上所有已部署的应用程序，以及该节点上所有已部署的服务包。

- 返回所有状态为错误的副本。返回所有应用程序、服务、分区，以及仅返回状态为错误的副本。

- 返回所有应用程序。针对指定服务，包含所有分区。

运行状况区块查询目前仅对群集实体公开。它会返回群集运行状况区块，其中包含：

- 群集的聚合健康状况。

- 采用输入筛选器的节点的健康状况区块列表。

- 采用输入筛选器的应用程序的健康状况区块列表。每个应用程序健康状况区块都包含下列两个区块列表：包含所有采用输入筛选器的服务的区块列表，以及包含所有采用筛选器的已部署应用程序的区块列表。对于服务和已部署应用程序的子项亦然。如此一来，群集中的所有实体都有可能在请求时以分层方式返回。

### 群集运行状况区块查询
这会返回群集实体的运行状况，并包含必要子项的分层健康状况区块。输入：

- [可选] 用于评估节点和群集事件的群集运行状况策略。

- [可选] 应用程序运行状况策略与用于取代应用程序清单策略的运行状况策略进行映射。

- [可选] 节点和应用程序的筛选器，用于指定应该在结果中返回的相关条目。筛选器特定于实体/实体组，或适用于该级别的所有实体。筛选器列表可包含一个常规筛选器以及/或者一个由查询返回的精细实体的特定标识符筛选器。如果筛选器列表为空，默认情况下不会返回任何子项。如需详细了解筛选器，请参阅 [NodeHealthStateFilter](https://msdn.microsoft.com/zh-cn/library/azure/system.fabric.health.nodehealthstatefilter.aspx) 和 [ApplicationHealthStateFilter](https://msdn.microsoft.com/zh-cn/library/azure/system.fabric.health.applicationhealthstatefilter.aspx)。应用程序筛选器可采用递归方式为子项指定高级筛选器。

区块结果包含采用筛选器的子项。

区块查询目前不会返回不正常的评估或实体事件。这些项目可使用现有的群集运行状况查询获取。

### API
若要获取群集运行状况，请创建 `FabricClient` 并在其 **HealthManager** 上调用 [GetClusterHealthChunkAsync](https://msdn.microsoft.com/zh-cn/library/azure/system.fabric.fabricclient.healthclient.getclusterhealthchunkasync.aspx) 方法。你可以传入 [ClusterHealthQueryDescription](https://msdn.microsoft.com/zh-cn/library/azure/system.fabric.description.clusterhealthchunkquerydescription.aspx) 来描述运行状况策略和高级筛选器。

以下代码使用高级筛选器获取群集运行状况区块。

```csharp
var queryDescription = new ClusterHealthChunkQueryDescription();
queryDescription.ApplicationFilters.Add(new ApplicationHealthStateFilter()
    {
        // Return applications only if they are in error
        HealthStateFilter = HealthStateFilter.Error
    });

// Return all replicas
var wordCountServiceReplicaFilter = new ReplicaHealthStateFilter()
    {
        HealthStateFilter = HealthStateFilter.All
    };

// Return all replicas and all partitions
var wordCountServicePartitionFilter = new PartitionHealthStateFilter()
    {
        HealthStateFilter = HealthStateFilter.All
    };
wordCountServicePartitionFilter.ReplicaFilters.Add(wordCountServiceReplicaFilter);

// For specific service, return all partitions and all replicas
var wordCountServiceFilter = new ServiceHealthStateFilter()
{
    ServiceNameFilter = new Uri("fabric:/WordCount/WordCountService"),
};
wordCountServiceFilter.PartitionFilters.Add(wordCountServicePartitionFilter);

// Application filter: for specific application, return no services except the ones of interest
var wordCountApplicationFilter = new ApplicationHealthStateFilter()
    {
        // Always return fabric:/WordCount application
        ApplicationNameFilter = new Uri("fabric:/WordCount"),
    };
wordCountApplicationFilter.ServiceFilters.Add(wordCountServiceFilter);

queryDescription.ApplicationFilters.Add(wordCountApplicationFilter);

var result = await fabricClient.HealthManager.GetClusterHealthChunkAsync(queryDescription);
```

### PowerShell
用于获取群集运行状况的 cmdlet 为 [Get-ServiceFabricClusterChunkHealth](https://msdn.microsoft.com/zh-cn/library/mt644772.aspx)。首先使用 [Connect-ServiceFabricCluster](https://msdn.microsoft.com/zh-cn/library/mt125938.aspx) cmdlet 连接到群集。

以下代码仅在节点处于“错误”状态时才获取节点，只有一个特定节点例外，任何情况下都应返回该节点。


	PS C:\> $errorFilter = [System.Fabric.Health.HealthStateFilter]::Error;
	$allFilter = [System.Fabric.Health.HealthStateFilter]::All;

	$nodeFilter1 = New-Object System.Fabric.Health.NodeHealthStateFilter -Property @{HealthStateFilter=$errorFilter}
	$nodeFilter2 = New-Object System.Fabric.Health.NodeHealthStateFilter -Property @{NodeNameFilter="_Node_1";HealthStateFilter=$allFilter}
	# Create node filter list that will be passed in the cmdlet
	$nodeFilters = New-Object System.Collections.Generic.List[System.Fabric.Health.NodeHealthStateFilter]
	$nodeFilters.Add($nodeFilter1)
	$nodeFilters.Add($nodeFilter2)

	Get-ServiceFabricClusterHealthChunk -NodeFilters $nodeFilters

	HealthState                  : Error
	NodeHealthStateChunks        :
                               		TotalCount            : 1

                               		NodeName              : _Node_1
                               		HealthState           : Ok

	ApplicationHealthStateChunks : None


以下 cmdlet 利用应用程序筛选器获取群集区块。


	$errorFilter = [System.Fabric.Health.HealthStateFilter]::Error;
	$allFilter = [System.Fabric.Health.HealthStateFilter]::All;

	# All replicas
	$replicaFilter = New-Object System.Fabric.Health.ReplicaHealthStateFilter -Property @{HealthStateFilter=$allFilter}

	# All partitions
	$partitionFilter = New-Object System.Fabric.Health.PartitionHealthStateFilter -Property @{HealthStateFilter=$allFilter}
	$partitionFilter.ReplicaFilters.Add($replicaFilter)

	# For WordCountService, return all partitions and all replicas
	$svcFilter1 = New-Object System.Fabric.Health.ServiceHealthStateFilter -Property @{ServiceNameFilter="fabric:/WordCount/WordCountService"}
	$svcFilter1.PartitionFilters.Add($partitionFilter)

	$svcFilter2 = New-Object System.Fabric.Health.ServiceHealthStateFilter -Property @{HealthStateFilter=$errorFilter}

	$appFilter = New-Object System.Fabric.Health.ApplicationHealthStateFilter -Property @{ApplicationNameFilter="fabric:/WordCount"}
	$appFilter.ServiceFilters.Add($svcFilter1)
	$appFilter.ServiceFilters.Add($svcFilter2)

	$appFilters = New-Object System.Collections.Generic.List[System.Fabric.Health.ApplicationHealthStateFilter]
	$appFilters.Add($appFilter)

	Get-ServiceFabricClusterHealthChunk -ApplicationFilters $appFilters

	HealthState                  : Error
	NodeHealthStateChunks        : None
	ApplicationHealthStateChunks :
                               		TotalCount            : 1

                               		ApplicationName       : fabric:/WordCount
                               		ApplicationTypeName   : WordCount
                               		HealthState           : Error
                               		ServiceHealthStateChunks :
                                   		TotalCount            : 1

                                   	ServiceName           : fabric:/WordCount/WordCountService
                                   	HealthState           : Error
                                   	PartitionHealthStateChunks :
                                       	TotalCount            : 1

                                       	PartitionId           : a1f83a35-d6bf-4d39-b90d-28d15f39599b
                                       	HealthState           : Error
                                       	ReplicaHealthStateChunks :
                                           	TotalCount            : 5

                                           	ReplicaOrInstanceId   : 131031502143040223
                                           	HealthState           : Ok

                                           	ReplicaOrInstanceId   : 131031502346844060
                                           	HealthState           : Ok

                                           	ReplicaOrInstanceId   : 131031502346844059
                                           	HealthState           : Ok

                                           	ReplicaOrInstanceId   : 131031502346844061
                                           	HealthState           : Ok

                                           	ReplicaOrInstanceId   : 131031502346844058
                                           	HealthState           : Error


以下 cmdlet 返回某个节点上所有已部署的实体。


	$errorFilter = [System.Fabric.Health.HealthStateFilter]::Error;
	$allFilter = [System.Fabric.Health.HealthStateFilter]::All;

	$dspFilter = New-Object System.Fabric.Health.DeployedServicePackageHealthStateFilter -Property @{HealthStateFilter=$allFilter}
	$daFilter =  New-Object System.Fabric.Health.DeployedApplicationHealthStateFilter -Property @{HealthStateFilter=$allFilter;NodeNameFilter="_Node_2"}
	$daFilter.DeployedServicePackageFilters.Add($dspFilter)

	$appFilter = New-Object System.Fabric.Health.ApplicationHealthStateFilter -Property @{HealthStateFilter=$allFilter}
	$appFilter.DeployedApplicationFilters.Add($daFilter)

	$appFilters = New-Object System.Collections.Generic.List[System.Fabric.Health.ApplicationHealthStateFilter]
	$appFilters.Add($appFilter)
	Get-ServiceFabricClusterHealthChunk -ApplicationFilters $appFilters


	HealthState                  : Error
	NodeHealthStateChunks        : None
	ApplicationHealthStateChunks :
                               		TotalCount            : 2

                               		ApplicationName       : fabric:/System
                               		HealthState           : Ok
                               		DeployedApplicationHealthStateChunks :
                                   	TotalCount            : 1

                                   	NodeName              : _Node_2
                                   	HealthState           : Ok
                                   	DeployedServicePackageHealthStateChunks :
                                       	TotalCount            : 1

                                       	ServiceManifestName   : FAS
                                       	HealthState           : Ok



                               		ApplicationName       : fabric:/WordCount
                               		ApplicationTypeName   : WordCount
                               		HealthState           : Error
                               		DeployedApplicationHealthStateChunks :
                                   		TotalCount            : 1

                                   	NodeName              : _Node_2
                                   	HealthState           : Ok
                                   	DeployedServicePackageHealthStateChunks :
                                       	TotalCount            : 2

                                       	ServiceManifestName   : WordCountServicePkg
                                       	HealthState           : Ok

                                       	ServiceManifestName   : WordCountWebServicePkg
                                       	HealthState           : Ok


## 常规查询
常规查询返回指定类型的 Service Fabric 实体的列表。这些查询通过 API（通过 **FabricClient.QueryManager** 上的方法）、PowerShell cmdlet 和 REST 来公开。这些查询聚合了来自多个组件的子查询。其中一个组件是[运行状况存储](/documentation/articles/service-fabric-health-introduction/#health-store)，该组件为每个查询结果填充聚合的健康状况。

> [AZURE.NOTE] 一般查询返回实体的聚合运行状况状态，不包含丰富的运行状况数据。如果一个实体不正常，你可以通过运行状况查询来跟进，获得所有运行状况信息，包括事件、子项运行状况状态和不正常评估。

如果一般查询返回实体的未知运行状况，则可能表示运行状况存储中不存在有关该实体的完整数据。此外，也有可能对运行状况存储的子查询未成功（例如，发生通信错误，或运行状况存储已被限制）。通过对实体进行运行状况查询进行跟进。如果子查询发生暂时性错误，例如网络问题，此跟进查询可能成功。它还可以从运行状况存储提供关于为何实体未公开的详细信息。

包含实体的 **HealthState** 的查询为：

- 节点列表：返回群集中节点的列表（已分页）。
  - API：[FabricClient.QueryClient.GetNodeListAsync](https://msdn.microsoft.com/zh-cn/library/azure/system.fabric.fabricclient.queryclient.getnodelistasync.aspx)
  - PowerShell：Get-ServiceFabricNode
- 应用程序列表：返回群集中应用程序的列表（已分页）。
  - API：[FabricClient.QueryClient.GetApplicationListAsync](https://msdn.microsoft.com/zh-cn/library/azure/system.fabric.fabricclient.queryclient.getapplicationlistasync.aspx)
  - PowerShell：Get-ServiceFabricApplication
- 服务列表：返回应用程序中服务的列表（已分页）。
  - API：[FabricClient.QueryClient.GetServiceListAsync](https://msdn.microsoft.com/zh-cn/library/azure/system.fabric.fabricclient.queryclient.getservicelistasync.aspx)
  - PowerShell：Get-ServiceFabricService
- 分区列表：返回服务中分区的列表（已分页）。
  - API：[FabricClient.QueryClient.GetPartitionListAsync](https://msdn.microsoft.com/zh-cn/library/azure/system.fabric.fabricclient.queryclient.getpartitionlistasync.aspx)
  - PowerShell：Get-ServiceFabricPartition
- 副本列表：返回分区中副本的列表（已分页）。
  - API：[FabricClient.QueryClient.GetReplicaListAsync](https://msdn.microsoft.com/zh-cn/library/azure/system.fabric.fabricclient.queryclient.getreplicalistasync.aspx)
  - PowerShell：Get-ServiceFabricReplica
- 已部署应用程序列表：返回节点上已部署应用程序的列表。
  - API：[FabricClient.QueryClient.GetDeployedApplicationListAsync](https://msdn.microsoft.com/zh-cn/library/azure/system.fabric.fabricclient.queryclient.getdeployedapplicationlistasync.aspx)
  - PowerShell：Get-ServiceFabricDeployedApplication
- 已部署服务包列表：返回已部署应用程序中服务包的列表。
  - API：[FabricClient.QueryClient.GetDeployedServicePackageListAsync](https://msdn.microsoft.com/zh-cn/library/azure/system.fabric.fabricclient.queryclient.getdeployedservicepackagelistasync.aspx)
  - PowerShell：Get-ServiceFabricDeployedApplication

> [AZURE.NOTE] 有些查询会返回已分页的结果。这些查询的返回结果是派生自 [PagedList<T>](https://msdn.microsoft.com/zh-cn/library/azure/mt280056.aspx) 的列表。如果一条消息无法容纳这些结果，则仅返回一个页面，并且 ContinuationToken 设置为跟踪枚举的停止位置。用户应该继续调用相同的查询，并从先前的查询传入继续标记以获取后续结果。

### 示例

以下代码获取群集中不正常的应用程序：

```csharp
var applications = fabricClient.QueryManager.GetApplicationListAsync().Result.Where(
  app => app.HealthState == HealthState.Error);
```

以下 cmdlet 获取 fabric:/WordCount 应用程序的详细信息。请注意，运行状况状态为警告。

	PS C:\> Get-ServiceFabricApplication -ApplicationName fabric:/WordCount

	ApplicationName        : fabric:/WordCount
	ApplicationTypeName    : WordCount
	ApplicationTypeVersion : 1.0.0
	ApplicationStatus      : Ready
	HealthState            : Warning
	ApplicationParameters  : { "WordCountWebService_InstanceCount" = "1";
                         	"_WFDebugParams_" = "[{"ServiceManifestName":"WordCountWebServicePkg","CodePackageName":"Code","EntryPointType":"Main","Debug
                         	ExePath":"C:\\Program Files (x86)\\Microsoft Visual Studio
                         	14.0\\Common7\\Packages\\Debugger\\VsDebugLaunchNotify.exe","DebugArguments":" {74f7e5d5-71a9-47e2-a8cd-1878ec4734f1} -p
                         	[ProcessId] -tid [ThreadId]","EnvironmentBlock":"_NO_DEBUG_HEAP=1\u0000"},{"ServiceManifestName":"WordCountServicePkg","CodeP
                         	ackageName":"Code","EntryPointType":"Main","DebugExePath":"C:\\Program Files (x86)\\Microsoft Visual Studio
                         	14.0\\Common7\\Packages\\Debugger\\VsDebugLaunchNotify.exe","DebugArguments":" {2ab462e6-e0d1-4fda-a844-972f561fe751} -p
                         	[ProcessId] -tid [ThreadId]","EnvironmentBlock":"_NO_DEBUG_HEAP=1\u0000"}]" }


以下 cmdlet 获取运行状况状态为警告的服务：

```powershell
PS C:\> Get-ServiceFabricApplication | Get-ServiceFabricService | where {$_.HealthState -eq "Warning"}


ServiceName            : fabric:/WordCount/WordCountService
ServiceKind            : Stateful
ServiceTypeName        : WordCountServiceType
IsServiceGroup         : False
ServiceManifestVersion : 1.0.0
HasPersistedState      : True
ServiceStatus          : Active
HealthState            : Warning
```

## 群集和应用程序升级
在群集与应用程序的受监视升级期间，Service Fabric 将检查运行状况，以确保一切都能维持在运行状况良好的状态。如果实体通过使用已设置的运行状况策略评估为状况不良，升级过程将应用升级特定的策略来确定后续措施。升级可能会暂停，以允许用户交互（例如修复错误条件或更改策略），或是它自动回滚到以前的正常版本。

在“群集”升级期间，你可以获取群集升级状态。这包括任何状况不正常的评估，指向群集中状况不正常的项目。如果升级因运行状况问题而回滚，则升级状态将保留最后的不正常原因。这可以让管理员能够调查发生的问题。

同样地，在“应用程序”升级期间，应用程序升级状态也会包含任何不正常的评估。

以下代码显示修改后的 fabric:/WordCount 应用程序的升级状态。监视程序在其中一个副本上报告一个错误。因为运行状况检查不合格，升级回滚。


	PS C:\> Get-ServiceFabricApplicationUpgrade fabric:/WordCount

	ApplicationName               : fabric:/WordCount
	ApplicationTypeName           : WordCount
	TargetApplicationTypeVersion  : 1.0.0.0
	ApplicationParameters         : {}
	StartTimestampUtc             : 4/21/2015 5:23:26 PM
	FailureTimestampUtc           : 4/21/2015 5:23:37 PM
	FailureReason                 : HealthCheck
	UpgradeState                  : RollingBackInProgress
	UpgradeDuration               : 00:00:23
	CurrentUpgradeDomainDuration  : 00:00:00
	CurrentUpgradeDomainProgress  : UD1

                                	NodeName            : _Node_1
                                	UpgradePhase        : Upgrading

                                	NodeName            : _Node_2
                                	UpgradePhase        : Upgrading

                                	NodeName            : _Node_3
                                	UpgradePhase        : PreUpgradeSafetyCheck
                                	PendingSafetyChecks :
                                	EnsurePartitionQuorum - PartitionId: 30db5be6-4e20-4698-8185-4bd7ca744020
	NextUpgradeDomain             : UD2
	UpgradeDomainsStatus          : { "UD1" = "Completed";
                                	"UD2" = "Pending";
                                	"UD3" = "Pending";
                                	"UD4" = "Pending" }
	UnhealthyEvaluations          :
                                	Unhealthy services: 100% (1/1), ServiceType='WordCountServiceType', MaxPercentUnhealthyServices=0%.

                                  	Unhealthy service: ServiceName='fabric:/WordCount/WordCountService', AggregatedHealthState='Error'.

                                      	Unhealthy partitions: 100% (1/1), MaxPercentUnhealthyPartitionsPerService=0%.

                                      	Unhealthy partition: PartitionId='a1f83a35-d6bf-4d39-b90d-28d15f39599b', AggregatedHealthState='Error'.

                                          	Unhealthy replicas: 20% (1/5), MaxPercentUnhealthyReplicasPerPartition=0%.

                                          	Unhealthy replica: PartitionId='a1f83a35-d6bf-4d39-b90d-28d15f39599b',
                                  	ReplicaOrInstanceId='131031502346844058', AggregatedHealthState='Error'.

                                              	Error event: SourceId='DiskWatcher', Property='Disk'.

	UpgradeKind                   : Rolling
	RollingUpgradeMode            : UnmonitoredAuto
	ForceRestart                  : False
	UpgradeReplicaSetCheckTimeout : 00:15:00


了解有关 [Service Fabric 应用程序升级](/documentation/articles/service-fabric-application-upgrade/)的详细信息。

## 使用系统运行状况评估进行故障排除
如果群集或应用程序出现问题，请立即查看群集或应用程序的运行状况以找出错误。不正常评估将提供是什么触发了当前不正常状态的详细信息。如果需要，你可以向下钻取到状况不正常的子实体，以识别根本原因。

> [AZURE.NOTE] 不正常评估将显示实体评估为当前健康状况的第一个原因。可能有其他多个事件触发此状态，但是评估中不会反映这些事件。你必须向下钻取到运行状况实体，以找出群集中所有不正常的报告。

## 后续步骤
[使用系统运行状况报告进行故障排除](/documentation/articles/service-fabric-understand-and-troubleshoot-with-system-health-reports/)

[添加自定义 Service Fabric 运行状况报告](/documentation/articles/service-fabric-report-health/)

[在本地监视和诊断服务](/documentation/articles/service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally/)

[Service Fabric 应用程序升级](/documentation/articles/service-fabric-application-upgrade/)

<!---HONumber=Mooncake_0523_2016-->