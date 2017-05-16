<properties
    pageTitle="如何查看 Azure Service Fabric 实体的聚合运行状况 | Azure"
    description="说明如何通过运行状况查询和常规查询，查询、查看和评估 Azure Service Fabric 实体的聚合运行状况。"
    services="service-fabric"
    documentationcenter=".net"
    author="oanapl"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="fa34c52d-3a74-4b90-b045-ad67afa43fe5"
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
    ms.openlocfilehash="ea398eb6bbc7fcad9fabfa84782acfa3838eb556"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="view-service-fabric-health-reports"></a>查看 Service Fabric 运行状况报告
Azure Service Fabric 引入了一种由运行状况实体组成的[运行状况模型](/documentation/articles/service-fabric-health-introduction/)，系统组件和监视器可以在其上报告它们监视的本地状况。 [运行状况存储](/documentation/articles/service-fabric-health-introduction/#health-store)聚合所有运行状况数据以确定实体是否正常运行。

根据现有设定，群集中将填充系统组件发送的运行状况报告。从[使用系统运行状况报告进行故障排除](/documentation/articles/service-fabric-understand-and-troubleshoot-with-system-health-reports/)了解更多信息。

Service Fabric 提供多种方式来获取实体聚合运行状况：

* [Service Fabric Explorer](/documentation/articles/service-fabric-visualizing-your-cluster/) 或其他可视化工具
* 运行状况查询（通过 PowerShell、API 或 REST）
* 常规查询，返回将运行状况作为属性之一的实体的列表（通过 PowerShell、API 或 REST）

为了演示这些选项，让我们使用一个具有五个节点的本地群集。 **fabric:/System** 应用程序（原本即已存在）旁边，已部署其他一些应用程序。 其中一个应用程序是 **fabric:/WordCount**。 该应用程序包含一个配置有七个副本的有状态服务。 由于只有五个节点，因此系统组件显示分区低于目标计数的警告。


	<Service Name="WordCountService">
	    <StatefulService ServiceTypeName="WordCountServiceType" TargetReplicaSetSize="7" MinReplicaSetSize="2">
	      <UniformInt64Partition PartitionCount="1" LowKey="1" HighKey="26" />
	    </StatefulService>
	</Service>

## <a name="health-in-service-fabric-explorer"></a>Service Fabric Explorer 中的运行状况
通过 Service Fabric Explorer，可直观查看群集。 在下图中，你可以看到：

* 应用程序 **fabric:/WordCount** 为红色（出错），因为 **MyWatchdog** 报告“**可用性**”属性有一个错误事件。
* 其服务之一 **fabric:/WordCount/WordCountService** 为黄色（警告）。 由于服务中配置了七个副本，这些副本无法全部放置（因为只有五个节点）。 尽管此处未显示，因为系统报告，服务分区为黄色。 黄色分区触发黄色服务。
* 由于应用程序为红色，因此群集为红色。

评估使用群集清单和应用程序清单的默认策略。它们是严格的策略，不容许任何失败。

使用 Service Fabric Explorer 查看群集：

![使用 Service Fabric Explorer 查看群集。][1]

[1]: ./media/service-fabric-view-entities-aggregated-health/servicefabric-explorer-cluster-health.png


> [AZURE.NOTE]
> 了解有关 [Service Fabric Explorer](/documentation/articles/service-fabric-visualizing-your-cluster/) 的更多信息。
>
>

## <a name="health-queries"></a>运行状况查询
Service Fabric 为每个支持的[实体类型](/documentation/articles/service-fabric-health-introduction/#health-entities-and-hierarchy)提供运行状况查询。 可以通过 API（可在 **FabricClient.HealthManager** 中找到的方法）、PowerShell cmdlet 和 REST 访问它们。 这些查询返回有关实体的完整运行状况信息：聚合运行状况、实体运行状况事件、子运行状况（在适用时）以及实体不正常时的不正常评估。

> [AZURE.NOTE]
> 填满运行状况存储时，将返回运行状况实体。 实体必须处于活动状态（未删除），并且具有系统报告。 层次结构链上其父实体还必须有系统报告。 如果不满足上述任意条件，则运行状况查询返回一个异常，并显示未返回实体的原因。
>
>

运行状况查询必须传递实体标识符，具体取决于实体类型。 这些查询接受可选的运行状况策略参数。 如果未指定运行状况策略，则使用来自群集或应用程序清单的[运行状况策略](/documentation/articles/service-fabric-health-introduction/#health-policies)进行评估。 这些查询还接受筛选器，以仅返回与指定筛选器有关的部分子项或事件。

> [AZURE.NOTE]
> 在服务器端应用输出筛选器，因此减小了消息回复大小。 我们建议使用输出筛选器限制返回的数据，而不是在客户端上应用筛选器。
>
>

实体的运行状况包含：

* 实体的聚合运行状况状态。 由运行状况存储依据实体运行状况报告、子项运行状况（在适用时）和运行状况策略计算。 了解有关[实体运行状况评估](/documentation/articles/service-fabric-health-introduction/#health-evaluation)的详细信息。  
* 实体上的运行状况事件。
* 对于能够拥有子项的实体，为所有子项的运行状况集合。 运行状况状态包含实体标识符和聚合的运行状况状态。 若要获取某个子项的完整运行状况，请调用子实体类型的查询运行状况，并传递子标识符。
* 如果实体不正常，指向触发实体状态的报告的不正常评估。

## <a name="get-cluster-health"></a>获取群集运行状况
返回群集实体的运行状况，并包含应用程序和节点（群集的子项）的运行状况。 输入：

* [可选] 用于评估节点和群集事件的群集运行状况策略。
* [可选] 应用程序运行状况策略与用于取代应用程序清单策略的运行状况策略进行映射。
* [可选] 事件、节点和应用程序的筛选器，指定有哪些相关项目，并且应该在结果中返回项目（例如，仅错误或警告和错误）。 所有事件、节点及应用程序都用于评估实体聚合运行状况，无论筛选器为何。

### <a name="api"></a>API
若要获取群集运行状况，请创建 `FabricClient` 并在其 **HealthManager** 上调用 [GetClusterHealthAsync](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.healthclient.getclusterhealthasync) 方法。

以下调用将获取群集运行状况：


	ClusterHealth clusterHealth = await fabricClient.HealthManager.GetClusterHealthAsync();

以下代码使用针对节点和应用程序的自定义群集运行状况策略和筛选器获取群集运行状况。 它将创建包含输入信息的 [ClusterHealthQueryDescription](https://docs.microsoft.com/dotnet/api/system.fabric.description.clusterhealthquerydescription)。

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

### <a name="powershell"></a>PowerShell
用于获取群集运行状况的 cmdlet 为 [Get-ServiceFabricClusterHealth](https://docs.microsoft.com/zh-cn/powershell/module/servicefabric/get-servicefabricclusterhealth)。 首先使用 [Connect-ServiceFabricCluster](https://docs.microsoft.com/zh-cn/powershell/module/servicefabric/connect-servicefabriccluster) cmdlet 连接到群集。

群集的状态：有五个节点、系统应用程序和如前所述配置的 fabric:/WordCount。

以下 cmdlet 使用默认运行状况策略获取群集运行状况。聚合的运行状况状态为警告，因为 fabric:/WordCount 应用程序处于警告状态。请注意不正常评估如何提供触发聚合运行状况的详细条件。


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


	PS c:\> $appHealthPolicy = New-Object -TypeName System.Fabric.Health.ApplicationHealthPolicy
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


### <a name="rest"></a>REST
可以使用 [GET 请求](https://docs.microsoft.com/zh-cn/rest/api/servicefabric/get-the-health-of-a-cluster)或 [POST 请求](https://docs.microsoft.com/zh-cn/rest/api/servicefabric/get-the-health-of-a-cluster-by-using-a-health-policy)获取群集运行状况，其中包括正文中所述的运行状况策略。

## <a name="get-node-health"></a>获取节点运行状况
返回节点实体的运行状况，并包含针对该节点报告的运行状况事件。 输入：

* [必需] 标识该节点的节点名称。
* [可选 ] 用于评估运行状况的群集运行状况策略设置。
* [可选] 事件的筛选器，指定有哪些相关项目，并且应该在结果中返回项目（例如，仅错误或警告和错误）。 所有事件都用于评估实体聚合运行状况，无论筛选器为何。

### <a name="api"></a>API
若要通过 API 获取节点运行状况，请创建 `FabricClient` 并在其 HealthManager 上调用 [GetNodeHealthAsync](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.healthclient.getnodehealthasync) 方法。

以下代码获取指定节点名称的节点运行状况：


	NodeHealth nodeHealth = await fabricClient.HealthManager.GetNodeHealthAsync(nodeName);

以下代码获取指定节点名称的节点运行状况，并通过 [NodeHealthQueryDescription](https://docs.microsoft.com/dotnet/api/system.fabric.description.nodehealthquerydescription) 传入事件筛选器和自定义策略：


	var queryDescription = new NodeHealthQueryDescription(nodeName)
	{
	    HealthPolicy = new ClusterHealthPolicy() {  ConsiderWarningAsError = true },
	    EventsFilter = new HealthEventsFilter() { HealthStateFilterValue = HealthStateFilter.Warning },
	};
	
	NodeHealth nodeHealth = await fabricClient.HealthManager.GetNodeHealthAsync(queryDescription);

### <a name="powershell"></a>PowerShell
用于获取节点运行状况的 cmdlet 为 [Get-ServiceFabricNodeHealth](https://docs.microsoft.com/zh-cn/powershell/module/servicefabric/get-servicefabricnodehealth)。 首先使用 [Connect-ServiceFabricCluster](https://docs.microsoft.com/zh-cn/powershell/module/servicefabric/connect-servicefabriccluster) cmdlet 连接到群集。
以下 cmdlet 使用默认运行状况策略获取节点运行状况：

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


以下 cmdlet 获取群集中所有节点的运行状况：


	PS C:\> Get-ServiceFabricNode | Get-ServiceFabricNodeHealth | select NodeName, AggregatedHealthState | ft -AutoSize
	
	NodeName AggregatedHealthState
	-------- ---------------------
	_Node_2                     Ok
	_Node_0                     Ok
	_Node_1                     Ok
	_Node_3                     Ok
	_Node_4                     Ok

### <a name="rest"></a>REST
可以使用 [GET 请求](https://docs.microsoft.com/zh-cn/rest/api/servicefabric/get-the-health-of-a-node)或 [POST 请求](https://docs.microsoft.com/zh-cn/rest/api/servicefabric/get-the-health-of-a-node-by-using-a-health-policy)获取节点运行状况，其中包括正文中所述的运行状况策略。

## <a name="get-application-health"></a>获取应用程序运行状况
返回一个应用程序实体的运行状况。 包含已部署应用程序和服务子项的运行状况状态。 输入：

* [必需] 标识应用程序的应用程序名称 (URI)。
* [可选] 用于取代应用程序清单策略的应用程序运行状况策略。
* [可选] 事件、服务和已部署应用程序的筛选器，指定有哪些相关项目，并且应该在结果中返回项目（例如，仅错误或警告和错误）。 所有事件、服务和已部署应用程序都用于评估实体聚合运行状况，无论筛选器为何。

### <a name="api"></a>API
若要获取应用程序运行状况，请创建 `FabricClient` 并在其 HealthManager 上调用 [GetApplicationHealthAsync](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.healthclient.getapplicationhealthasync) 方法。

以下代码获取具有指定应用程序名称 (URI) 的应用程序的运行状况：


	ApplicationHealth applicationHealth = await fabricClient.HealthManager.GetApplicationHealthAsync(applicationName);

以下代码使用通过 [ApplicationHealthQueryDescription](https://docs.microsoft.com/dotnet/api/system.fabric.description.applicationhealthquerydescription) 指定的筛选器和自定义策略，获取指定应用程序名称 (URI) 的应用程序运行状况。

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

### <a name="powershell"></a>PowerShell
用于获取应用程序运行状况的 cmdlet 为 [Get-ServiceFabricApplicationHealth](https://docs.microsoft.com/zh-cn/powershell/module/servicefabric/get-servicefabricapplicationhealth)。 首先使用 [Connect-ServiceFabricCluster](https://docs.microsoft.com/zh-cn/powershell/module/servicefabric/connect-servicefabriccluster) cmdlet 连接到群集。

以下 cmdlet 返回 **fabric:/WordCount** 应用程序的运行状况：


	PS c:\>
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

### <a name="rest"></a>REST
可以使用 [GET 请求](https://docs.microsoft.com/zh-cn/rest/api/servicefabric/get-the-health-of-an-application)或 [POST 请求](https://docs.microsoft.com/zh-cn/rest/api/servicefabric/get-the-health-of-an-application-by-using-an-application-health-policy)获取应用程序运行状况，其中包括正文中所述的运行状况策略。

## <a name="get-service-health"></a>获取服务运行状况
返回一个服务实体的运行状况。 包含分区运行状况状态。 输入：

* [必需] 标识服务的服务名称 (URI)。
* [可选] 用于取代应用程序清单策略的应用程序运行状况策略。
* [可选] 事件和分区的筛选器，指定有哪些相关项目，并且应该在结果中返回项目（例如，仅错误或警告和错误）。 所有事件和分区都用于评估实体聚合运行状况，无论筛选器为何。

### <a name="api"></a>API
若要通过 API 获取服务运行状况，请创建 `FabricClient` 并在其 HealthManager 上调用 [GetServiceHealthAsync](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.healthclient.getservicehealthasync) 方法。

以下示例获取具有指定服务名称 (URI) 的服务的运行状况：


	ServiceHealth serviceHealth = await fabricClient.HealthManager.GetServiceHealthAsync(serviceName);

以下代码通过 [ServiceHealthQueryDescription](https://docs.microsoft.com/dotnet/api/system.fabric.description.servicehealthquerydescription) 指定筛选器和自定义策略，从而获取指定服务名称 (URI) 的服务运行状况：


	var queryDescription = new ServiceHealthQueryDescription(serviceName)
	{
	    EventsFilter = new HealthEventsFilter() { HealthStateFilterValue = HealthStateFilter.All },
	    PartitionsFilter = new PartitionHealthStatesFilter() { HealthStateFilterValue = HealthStateFilter.Error },
	};
	
	ServiceHealth serviceHealth = await fabricClient.HealthManager.GetServiceHealthAsync(queryDescription);

### <a name="powershell"></a>PowerShell
用于获取服务运行状况的 cmdlet 为 [Get-ServiceFabricServiceHealth](https://docs.microsoft.com/zh-cn/powershell/module/servicefabric/get-servicefabricservicehealth)。 首先使用 [Connect-ServiceFabricCluster](https://docs.microsoft.com/zh-cn/powershell/module/servicefabric/connect-servicefabriccluster) cmdlet 连接到群集。

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

### <a name="rest"></a>REST
可以使用 [GET 请求](https://docs.microsoft.com/zh-cn/rest/api/servicefabric/get-the-health-of-a-service)或 [POST 请求](https://docs.microsoft.com/zh-cn/rest/api/servicefabric/get-the-health-of-a-service-by-using-a-health-policy)获取服务运行状况，其中包括正文中所述的运行状况策略。

## <a name="get-partition-health"></a>获取分区运行状况
返回一个分区实体的运行状况。 包含副本运行状况状态。 输入：

* [必需] 标识分区的分区 ID (GUID)。
* [可选] 用于取代应用程序清单策略的应用程序运行状况策略。
* [可选] 事件和副本的筛选器，指定有哪些相关项目，并且应该在结果中返回项目（例如，仅错误或警告和错误）。 所有事件和副本都用于评估实体聚合运行状况，无论筛选器为何。

### <a name="api"></a>API
若要通过 API 获取分区运行状况，请创建 `FabricClient` 并在其 HealthManager 上调用 [GetPartitionHealthAsync](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.healthclient.getpartitionhealthasync) 方法。 若要指定可选参数，请创建 [PartitionHealthQueryDescription](https://docs.microsoft.com/dotnet/api/system.fabric.description.partitionhealthquerydescription)。


	PartitionHealth partitionHealth = await fabricClient.HealthManager.GetPartitionHealthAsync(partitionId);

### <a name="powershell"></a>PowerShell
用于获取分区运行状况的 cmdlet 为 [Get-ServiceFabricPartitionHealth](https://docs.microsoft.com/zh-cn/powershell/module/servicefabric/get-servicefabricpartitionhealth)。 首先使用 [Connect-ServiceFabricCluster](https://docs.microsoft.com/zh-cn/powershell/module/servicefabric/connect-servicefabriccluster) cmdlet 连接到群集。

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

### <a name="rest"></a>REST
可以使用 [GET 请求](https://docs.microsoft.com/zh-cn/rest/api/servicefabric/get-the-health-of-a-partition)或 [POST 请求](https://docs.microsoft.com/zh-cn/rest/api/servicefabric/get-the-health-of-a-partition-by-using-a-health-policy)获取分区运行状况，其中包括正文中所述的运行状况策略。

## <a name="get-replica-health"></a>获取副本运行状况
返回有状态服务副本或无状态服务实例的运行状况。 输入：

* [必需] 分区 ID (GUID) 和用于标识副本的副本 ID。
* [可选] 用于取代应用程序清单策略的应用程序运行状况策略参数。
* [可选] 事件的筛选器，指定有哪些相关项目，并且应该在结果中返回项目（例如，仅错误或警告和错误）。 所有事件都用于评估实体聚合运行状况，无论筛选器为何。

### <a name="api"></a>API
若要通过 API 获取副本运行状况，请创建 `FabricClient` 并在其 HealthManager 上调用 [GetReplicaHealthAsync](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.healthclient.getreplicahealthasync) 方法。 若要指定高级参数，请使用 [ReplicaHealthQueryDescription](https://docs.microsoft.com/dotnet/api/system.fabric.description.replicahealthquerydescription)。


	ReplicaHealth replicaHealth = await fabricClient.HealthManager.GetReplicaHealthAsync(partitionId, replicaId);

### <a name="powershell"></a>PowerShell
用于获取副本运行状况的 cmdlet 为 [Get-ServiceFabricReplicaHealth](https://docs.microsoft.com/zh-cn/powershell/module/servicefabric/get-servicefabricreplicahealth)。 首先使用 [Connect-ServiceFabricCluster](https://docs.microsoft.com/zh-cn/powershell/module/servicefabric/connect-servicefabriccluster) cmdlet 连接到群集。

以下 cmdlet 获取服务的所有分区的主要副本运行状况：


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

### <a name="rest"></a>REST
可以使用 [GET 请求](https://docs.microsoft.com/zh-cn/rest/api/servicefabric/get-the-health-of-a-replica)或 [POST 请求](https://docs.microsoft.com/zh-cn/rest/api/servicefabric/get-the-health-of-a-replica-by-using-a-health-policy)获取副本运行状况，其中包括正文中所述的运行状况策略。

## <a name="get-deployed-application-health"></a>获取已部署应用程序的运行状况
返回部署在节点实体上的一个应用程序的运行状况。 包含已部署服务包运行状况状态。 输入：

* [必需] 标识已部署应用程序的应用程序名称 (URI) 和节点名称（字符串）。
* [可选] 用于取代应用程序清单策略的应用程序运行状况策略。
* [可选] 事件和已部署服务包的筛选器，指定有哪些相关项目，并且应该在结果中返回项目（例如，仅错误或警告和错误）。 所有事件和已部署服务包都用于评估实体聚合运行状况，无论筛选器为何。

### <a name="api"></a>API
若要通过 API 获取部署在节点上的一个应用程序的运行状况，请创建 `FabricClient` 并在其 HealthManager 上调用 [GetDeployedApplicationHealthAsync](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.healthclient.getdeployedapplicationhealthasync)方法。 若要指定可选参数，请使用 [DeployedApplicationHealthQueryDescription](https://docs.microsoft.com/dotnet/api/system.fabric.description.deployedapplicationhealthquerydescription)。


	DeployedApplicationHealth health = await fabricClient.HealthManager.GetDeployedApplicationHealthAsync(
    	new DeployedApplicationHealthQueryDescription(applicationName, nodeName));

### <a name="powershell"></a>PowerShell
用于获取已部署应用程序的运行状况的 cmdlet 为 [Get-ServiceFabricDeployedApplicationHealth](https://docs.microsoft.com/zh-cn/powershell/module/servicefabric/get-servicefabricdeployedapplicationhealth)。 首先使用 [Connect-ServiceFabricCluster](https://docs.microsoft.com/zh-cn/powershell/module/servicefabric/connect-servicefabriccluster) cmdlet 连接到群集。 若要了解应用程序的部署位置，请运行 [Get-ServiceFabricApplicationHealth](https://docs.microsoft.com/zh-cn/powershell/module/servicefabric/get-servicefabricapplicationhealth) 并查看已部署应用程序子项。

以下 cmdlet 获取部署在 **_Node_2** 上的 **fabric:/WordCount** 应用程序的运行状况。


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

### <a name="rest"></a>REST
可以使用 [GET 请求](https://docs.microsoft.com/zh-cn/rest/api/servicefabric/get-the-health-of-a-deployed-application)或 [POST 请求](https://docs.microsoft.com/zh-cn/rest/api/servicefabric/get-the-health-of-a-deployed-application-by-using-a-health-policy)获取部署的应用程序运行状况，其中包括正文中所述的运行状况策略。

## <a name="get-deployed-service-package-health"></a>获取已部署服务包的运行状况
返回一个已部署服务包实体的运行状况。 输入：

* [必需] 标识已部署服务包的应用程序名称 (URI)、节点名称（字符串）和服务清单名称（字符串）。
* [可选] 用于取代应用程序清单策略的应用程序运行状况策略。
* [可选] 事件的筛选器，指定有哪些相关项目，并且应该在结果中返回项目（例如，仅错误或警告和错误）。 所有事件都用于评估实体聚合运行状况，无论筛选器为何。

### <a name="api"></a>API
若要通过 API 获取一个已部署服务包的运行状况，请创建 `FabricClient` 并在其 HealthManager 上调用 [GetDeployedServicePackageHealthAsync](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.healthclient.getdeployedservicepackagehealthasync) 方法。 若要指定可选参数，请使用 [DeployedServicePackageHealthQueryDescription](https://docs.microsoft.com/dotnet/api/system.fabric.description.deployedservicepackagehealthquerydescription)。


	DeployedServicePackageHealth health = await fabricClient.HealthManager.GetDeployedServicePackageHealthAsync(
	    new DeployedServicePackageHealthQueryDescription(applicationName, nodeName, serviceManifestName));

### <a name="powershell"></a>PowerShell
用于获取已部署服务包的运行状况的 cmdlet 为 [Get-ServiceFabricDeployedServicePackageHealth](https://docs.microsoft.com/zh-cn/powershell/module/servicefabric/get-servicefabricdeployedservicepackagehealth)。 首先使用 [Connect-ServiceFabricCluster](https://docs.microsoft.com/zh-cn/powershell/module/servicefabric/connect-servicefabriccluster) cmdlet 连接到群集。 若要查看应用程序的部署位置，请运行 [Get-ServiceFabricApplicationHealth](https://docs.microsoft.com/zh-cn/powershell/module/servicefabric/get-servicefabricapplicationhealth) 并查看已部署应用程序。 若要查看一个应用程序中有哪些服务包，请在 [Get-ServiceFabricDeployedApplicationHealth](https://docs.microsoft.com/zh-cn/powershell/module/servicefabric/get-servicefabricdeployedapplicationhealth) 输出中查看已部署的服务包子项。

以下 cmdlet 获取部署在 **_Node_2** 上的 **fabric:/WordCount** 应用程序的 **WordCountServicePkg** 服务包的运行状况。此实体的 **System.Hosting** 报告包含成功的服务包和入口点激活以及成功的服务类型注册。


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

### <a name="rest"></a>REST
可以使用 [GET 请求](https://docs.microsoft.com/zh-cn/rest/api/servicefabric/get-the-health-of-a-service-package)或 [POST 请求](https://docs.microsoft.com/zh-cn/rest/api/servicefabric/get-the-health-of-a-service-package-by-using-a-health-policy)获取部署的服务包运行状况，其中包括正文中所述的运行状况策略。

## <a name="health-chunk-queries"></a>运行状况区块查询
运行状况区块查询可以根据输入筛选器返回多级群集子项（以递归方式）。 它支持高级筛选器，利用这些筛选器，可以非常灵活地表示要返回的特定子项（由其唯一标识符或其他组标识符和/或运行状况状态标识）。 与始终包含第一级子项的运行状况命令不同的是，它在默认情况下不包含任何子项。

[运行状况查询](/documentation/articles/service-fabric-view-entities-aggregated-health/#health-queries)根据必要筛选器仅返回指定实体的第一级子项。若要获取子项的子项，必须调用每个相关实体的附加运行状况 API。同样，若要获取特定实体的运行状况，必须调用每个所需实体的一个运行状况 API。使用区块查询高级筛选可在一个查询中请求多个相关项目，将消息大小和消息数目降至最低。

使用区块查询的值可在一个调用中获取多个群集实体（可能是从必要的根开始的所有群集实体）的运行状况。你可以如下表示复杂的运行状况查询：

* 仅返回状态为错误的应用程序，并且针对这些应用程序，包含所有状态为警告|错误的服务。 针对返回的服务，包含所有分区。
* 仅返回 4 个应用程序的运行状况，按其名称指定。
* 仅返回所需应用程序类型的应用程序运行状况。
* 返回某个节点上所有已部署实体。 返回所有应用程序、指定节点上所有已部署应用程序，以及该节点上所有已部署服务包。
* 返回状态为错误的所有副本。 返回所有应用程序、服务、分区，以及仅返回状态为错误的副本。
* 返回所有应用程序。 针对指定服务，包含所有分区。

运行状况区块查询目前仅对群集实体公开。它会返回群集运行状况区块，其中包含：

* 群集聚合的运行状况状态。
* 采用输入筛选器的节点的运行状况状态区块列表。
* 采用输入筛选器的应用程序的运行状况状态区块列表。 每个应用程序运行状况状态区块都包含下列两个区块列表：包含所有采用输入筛选器的服务的区块列表，以及包含所有采用筛选器的已部署应用程序的区块列表。 对于服务和已部署应用程序的子项亦然。 这样，群集中的所有实体都有可能在请求时以分层方式返回。

### <a name="cluster-health-chunk-query"></a>群集运行状况区块查询
返回群集实体的运行状况，并包含必要子项的分层运行状况状态区块。 输入：

* [可选] 用于评估节点和群集事件的群集运行状况策略。
* [可选] 应用程序运行状况策略与用于取代应用程序清单策略的运行状况策略进行映射。
* [可选] 节点和应用程序的筛选器，用于指定有哪些相关项目，并且应该在结果中返回项目。 筛选器特定于实体/实体组，或适用于该级别的所有实体。 筛选器列表可包含一个常规筛选器和/或由查询返回的精细实体的特定标识符筛选器。 如果筛选器列表为空，默认情况下不会返回任何子项。
  有关筛选器的详细信息，请参阅 [NodeHealthStateFilter](https://docs.microsoft.com/dotnet/api/system.fabric.health.nodehealthstatefilter) 和 [ApplicationHealthStateFilter](https://docs.microsoft.com/dotnet/api/system.fabric.health.applicationhealthstatefilter)。 应用程序筛选器可采用递归方式为子项指定高级筛选器。

区块结果包含采用筛选器的子项。

区块查询目前不会返回不正常的评估或实体事件。可以使用现有的群集运行状况查询获取这些附加信息。

### <a name="api"></a>API
若要获取群集运行状况，请创建 `FabricClient` 并在其 **HealthManager** 上调用 [GetClusterHealthChunkAsync](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.healthclient.getclusterhealthchunkasync) 方法。 可以传入 [ClusterHealthQueryDescription](https://docs.microsoft.com/dotnet/api/system.fabric.description.clusterhealthchunkquerydescription) 来描述运行状况策略和高级筛选器。

以下代码使用高级筛选器获取群集运行状况区块。


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

### <a name="powershell"></a>PowerShell
用于获取群集运行状况的 cmdlet 为 [Get-ServiceFabricClusterChunkHealth](https://docs.microsoft.com/zh-cn/powershell/module/servicefabric/get-servicefabricclusterhealthchunk)。 首先使用 [Connect-ServiceFabricCluster](https://docs.microsoft.com/zh-cn/powershell/module/servicefabric/connect-servicefabriccluster) cmdlet 连接到群集。

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


以下 cmdlet 使用应用程序筛选器获取群集区块。


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


以下 cmdlet 返回某个节点上的所有已部署实体。


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

### <a name="rest"></a>REST
可以使用[GET 请求](https://docs.microsoft.com/zh-cn/rest/api/servicefabric/get-the-health-of-a-cluster-using-health-chunks)或 [POST 请求](https://docs.microsoft.com/zh-cn/rest/api/servicefabric/health-of-cluster)获取群集运行状况区块，其中包括正文中所述的运行状况策略和高级筛选器。

## <a name="general-queries"></a>常规查询
常规查询返回指定类型的 Service Fabric 实体的列表。 这些查询通过 API（通过 **FabricClient.QueryManager** 上的方法）、PowerShell cmdlet 和 REST 来公开。 这些查询聚合了来自多个组件的子查询。 其中一个组件是[运行状况存储](/documentation/articles/service-fabric-health-introduction/#health-store)，该组件填充每个查询结果的聚合运行状况。  

> [AZURE.NOTE]
> 常规查询返回实体的聚合运行状况状态，不包含丰富的运行状况数据。 如果一个实体不正常，你可以通过运行状况查询跟进，以获得所有运行状况信息，包括事件、子项运行状况状态和不正常评估。
>
>

如果常规查询返回实体的未知运行状况状态，则可能表示运行状况存储中不存在有关该实体的完整数据。此外，也有可能对运行状况存储的子查询未成功（例如，发生通信错误，或运行状况存储已受限制）。通过对实体进行运行状况查询跟进。如果子查询发生暂时性错误，例如网络问题，此跟进查询可能成功。它还可以从运行状况存储提供关于为何实体未公开的详细信息。

包含实体的 **HealthState** 的查询为：

* 节点列表：返回群集中节点的列表（已分页）。
  * API：[FabricClient.QueryClient.GetNodeListAsync](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.queryclient.getnodelistasync)
  * PowerShell：Get-ServiceFabricNode
* 应用程序列表：返回群集中应用程序的列表（已分页）。
  * API：[FabricClient.QueryClient.GetApplicationListAsync](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.queryclient.getapplicationlistasync)
  * PowerShell：Get-ServiceFabricApplication
* 服务列表：返回应用程序中服务的列表（已分页）。
  * API：[FabricClient.QueryClient.GetServiceListAsync](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.queryclient.getservicelistasync)
  * PowerShell：Get-ServiceFabricService
* 分区列表：返回服务中分区的列表（已分页）。
  * API：[FabricClient.QueryClient.GetPartitionListAsync](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.queryclient.getpartitionlistasync)
  * PowerShell：Get-ServiceFabricPartition
* 副本列表：返回分区中副本的列表（已分页）。
  * API：[FabricClient.QueryClient.GetReplicaListAsync](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.queryclient.getreplicalistasync)
  * PowerShell：Get-ServiceFabricReplica
* 已部署应用程序列表：返回节点上已部署应用程序的列表。
  * API：[FabricClient.QueryClient.GetDeployedApplicationListAsync](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.queryclient.getdeployedapplicationlistasync)
  * PowerShell：Get-ServiceFabricDeployedApplication
* 已部署服务包列表：返回已部署应用程序中服务包的列表。
  * API：[FabricClient.QueryClient.GetDeployedServicePackageListAsync](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.queryclient.getdeployedservicepackagelistasync)
  * PowerShell：Get-ServiceFabricDeployedApplication

> [AZURE.NOTE]
> 有些查询会返回已分页的结果。 这些查询的返回结果是派生自 [PagedList<T>](https://docs.microsoft.com/dotnet/api/system.fabric.query.pagedlist-1) 的列表。 如果一条消息无法容纳这些结果，则仅返回一页，以及一个用于跟踪枚举停止位置的 ContinuationToken。 应该继续调用相同的查询，并从先前的查询传入继续标记以获取后续结果。
>
>

### <a name="examples"></a>示例
以下代码获取群集中不正常的应用程序：


	var applications = fabricClient.QueryManager.GetApplicationListAsync().Result.Where(
	  app => app.HealthState == HealthState.Error);


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


	PS C:\> Get-ServiceFabricApplication | Get-ServiceFabricService | where {$_.HealthState -eq "Warning"}
	
	
	ServiceName            : fabric:/WordCount/WordCountService
	ServiceKind            : Stateful
	ServiceTypeName        : WordCountServiceType
	IsServiceGroup         : False
	ServiceManifestVersion : 1.0.0
	HasPersistedState      : True
	ServiceStatus          : Active
	HealthState            : Warning

## <a name="cluster-and-application-upgrades"></a>群集和应用程序升级
在群集与应用程序的受监视升级期间，Service Fabric 将检查运行状况，以确保一切都能维持在运行状况良好的状态。 如果实体通过使用已配置的运行状况策略评估为不正常，升级过程将应用升级特定的策略来确定后续措施。 升级可能会暂停，以允许用户交互（例如修复错误条件或更改策略），或是它自动回滚到以前的正常版本。

在*群集*升级期间，可以获取群集升级状态。升级状态包括状况不正常的评估，指向群集中状况不正常的项目。如果升级因运行状况问题而回滚，则升级状态将记住最后的不正常原因。此信息可帮助管理员调查升级回滚或停止后发生的问题。

同样，在*应用程序*升级期间，应用程序升级状态也会包含任何不正常的评估。

以下代码显示修改后的 fabric:/WordCount 应用程序的应用程序升级状态。监视器在其中一个副本上报告一个错误。因为运行状况检查不合格，升级回滚。


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

## <a name="use-health-evaluations-to-troubleshoot"></a>使用运行状况评估进行故障排除
如果群集或应用程序出现问题，请立即查看群集或应用程序运行状况以找出错误。 不正常评估将提供是什么触发了当前不正常状态的详细信息。 如果需要，你可以向下钻取到状况不正常的子实体，以识别根本原因。

> [AZURE.NOTE]
> 不正常评估将显示实体评估为当前运行状况状态的第一个原因。 可能有其他多个事件触发此状态，但是评估中不会反映这些事件。 若要获取更多信息，请向下钻取到运行状况实体，找出群集中的所有不正常报告。
>
>

## <a name="next-steps"></a>后续步骤
[使用系统运行状况报告进行故障排除](/documentation/articles/service-fabric-understand-and-troubleshoot-with-system-health-reports/)

[添加自定义 Service Fabric 运行状况报告](/documentation/articles/service-fabric-report-health/)

[如何报告和检查服务运行状况](/documentation/articles/service-fabric-diagnostics-how-to-report-and-check-service-health/)

[在本地监视和诊断服务](/documentation/articles/service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally/)

[Service Fabric 应用程序升级](/documentation/articles/service-fabric-application-upgrade/)
<!--Update_Description: wording update;add anchors to subtitles-->