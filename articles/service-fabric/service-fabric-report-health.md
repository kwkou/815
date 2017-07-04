<properties
    pageTitle="添加自定义 Service Fabric 运行状况报告 | Azure"
    description="介绍如何将自定义运行状况报告发送至 Azure Service Fabric 运行状况实体。 为设计和实现优质运行状况报告提供建议。"
    services="service-fabric"
    documentationcenter=".net"
    author="oanapl"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="0a00a7d2-510e-47d0-8aa8-24c851ea847f"
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
    ms.openlocfilehash="b1f01663ab4c29a36b3794e427191469a3763dc0"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="add-custom-service-fabric-health-reports"></a>添加自定义 Service Fabric 运行状况报告
Azure Service Fabric 引入了[运行状况模型](/documentation/articles/service-fabric-health-introduction/)，用于在特定实体上标记不正常的群集和应用程序状态。 运行状况模型使用**运行状况报告器**（系统组件和监视器）。 其目标是实现轻松快捷的诊断和修复。 服务编写人员必须预先考虑运行状况。 应报告任何可能会影响运行状况的条件，尤其是如果它有助于标记出接近根源的问题。 在云（私有云或 Azure）中大规模启动并运行服务后，运行状况信息可以大幅减少调试和调查工作所需的时间和精力。

Service Fabric 报告器可监视感兴趣的已标识条件。 它们会根据其本地视图报告这些条件。 [运行状况存储](/documentation/articles/service-fabric-health-introduction/#health-store)可汇总所有报告器发送的运行状况数据，从而确定实体的全局运行状况是否正常。 该模型应具有功能丰富、灵活且易于使用的特点。 运行状况报告的质量决定了群集运行状况视图的准确度。 错误显示不正常问题的误报会对升级或其他使用运行状况数据的服务产生负面影响。 修复服务和警报机制就是此类服务的例子。 因此，提供报表时需多加考量，才能让其以尽可能最佳的方式捕获感兴趣的条件。

若要设计和实施运行状况报告，监视器和系统组件必须：

* 定义它们感兴趣的条件、受监视的方式以及对群集或应用程序功能的影响。 根据此信息确定运行状况报告属性和运行状态。
* 确定应用报表的[实体](/documentation/articles/service-fabric-health-introduction/#health-entities-and-hierarchy)。
* 确定是从服务内部、内部监视器还是外部监视器完成报表。
* 定义用于标识报告器的源。
* 选择报告策略，是定期报告还是在转换时报告。 建议定期发送报告，因为它需要的代码较简单，不容易发生错误。
* 确定不正常条件的报告可在运行状况存储中保留多久，以及应该如何清除。 使用此信息确定报告的生存时间和到期删除行为。

如上所述，可以基于以下内容完成报告：

* 受监视的 Service Fabric 服务副本。
* 部署为 Service Fabric 服务的内部监视器（例如，监视状态和问题报告的 Service Fabric 无状态服务）。 可以在所有节点上部署监视器，也可以将监视器与受监视的服务关联。
* 在 Service Fabric 节点上运行，但*未*以 Service Fabric 服务形式实现的内部监视器。
* 从 Service Fabric 群集*外*探测资源的外部监视器（例如，诸如 Gomez 之类的监视服务）。

> [AZURE.NOTE]
> 根据现有设定，群集会被系统组件发送的运行状况报告填充。 有关详细信息，请参阅[使用系统运行状况报告进行故障排除](/documentation/articles/service-fabric-understand-and-troubleshoot-with-system-health-reports/)。 必须针对系统已创建的[运行状况实体](/documentation/articles/service-fabric-health-introduction/#health-entities-and-hierarchy)发送用户报告。
> 
> 

只要运行状况报告的设计清晰明了，发送运行状况报告就十分容易。 如果群集不[安全](/documentation/articles/service-fabric-cluster-security/)或 Fabric 客户端拥有管理员权限，可以使用 [FabricClient](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient) 报告运行状况。 可以使用 [FabricClient.HealthManager.ReportHealth](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.healthclient.reporthealth) 通过 API 来完成，或者通过 PowerShell 或 REST 来完成。 配置旋钮 Batch 报告可提升性能。

> [AZURE.NOTE]
> 报告运行状况会同步处理，并且只代表客户端上的验证工作。 运行状况客户端或者 `Partition` 或 `CodePackageActivationContext` 对象接受报告的事实并不表示该报告已应用于存储中。 它以异步方式发送并可能与其他报告一起进行批处理。 服务器上的处理仍可能失败：序号可能已过时、必须应用报告的实体已被删除，等等。
> 
> 

## <a name="health-client"></a>运行状况客户端
运行状况报告通过存在于结构客户端内的运行状况客户端发送至运行状况存储。 可以使用下列设置来配置运行状况客户端：

* **HealthReportSendInterval**：报告添加至该客户端与其发送至运行状况存储之间的时间延迟。 用于将报告 Batch 为单条消息，而不会为每个报告发送一条消息。 批处理可以提升性能。 默认值：30 秒。
* **HealthReportRetrySendInterval**：运行状况客户端将累积的运行状况报告重新发送至运行状况存储的间隔时间。 默认值：30 秒。
* **HealthOperationTimeout**：报告消息发送到运行状况存储的超时期限。 如果消息超时，运行状况客户端就会不断重试，直到运行状况存储确认报告已处理为止。 默认值：2 分钟。

> [AZURE.NOTE]
> 批量处理报告时，结构客户端必须至少保持 HealthReportSendInterval 的活动状态，以确保报告发送完毕。 如果消息丢失或运行状况存储因为暂时性错误而无法应用它们，结构客户端必须保持更长时间的活动状态，让其有重试的机会。
> 
> 

客户端上的缓冲会将报告的唯一性纳入考虑范围。 例如，如果特定的错误报告器针对相同实体的相同属性每秒报告 100 个报告，则会以最后一个版本取代所有报告。 客户端队列中最多存在一个这样的报告。 如果配置了批处理，则发送到运行状况存储的报告数目仅为每个发送间隔发送一份报告。 这是最后添加的报告，可反映实体的最新状态。
通过传递 [FabricClientSettings](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclientsettings)（包含与运行状况相关的实体的相应值）创建 `FabricClient` 时，可以指定所有配置参数。

以下命令将创建结构客户端，并指定在添加报告后尽快发送。 可重试的错误或超时发生时，每 40 秒重试一次。


	var clientSettings = new FabricClientSettings()
	{
	    HealthOperationTimeout = TimeSpan.FromSeconds(120),
	    HealthReportSendInterval = TimeSpan.FromSeconds(0),
	    HealthReportRetrySendInterval = TimeSpan.FromSeconds(40),
	};
	var fabricClient = new FabricClient(clientSettings);

通过 PowerShell 创建与群集的连接时，可以指定相同的参数。 以下命令将启动与本地群集的连接：

	PS C:\> Connect-ServiceFabricCluster -HealthOperationTimeoutInSec 120 -HealthReportSendIntervalInSec 0 -HealthReportRetrySendIntervalInSec 40
	True

	ConnectionEndpoint   :
	FabricClientSettings : {
                       		ClientFriendlyName                   : PowerShell-1944858a-4c6d-465f-89c7-9021c12ac0bb
                       		PartitionLocationCacheLimit          : 100000
                       		PartitionLocationCacheBucketCount    : 1024
                       		ServiceChangePollInterval            : 00:02:00
                       		ConnectionInitializationTimeout      : 00:00:02
                       		KeepAliveInterval                    : 00:00:20
                       		HealthOperationTimeout               : 00:02:00
                       		HealthReportSendInterval             : 00:00:00
                       		HealthReportRetrySendInterval        : 00:00:40
                       		NotificationGatewayConnectionTimeout : 00:00:00
                       		NotificationCacheUpdateTimeout       : 00:00:00
                       		}
	GatewayInformation   : {
                       		NodeAddress                          : localhost:19000
                       		NodeId                               : 1880ec88a3187766a6da323399721f53
                       		NodeInstanceId                       : 130729063464981219
                       		NodeName                             : Node.1
                       		}

> [AZURE.NOTE]
> 若要确保未授权的服务无法针对群集中的实体报告运行状况，可将服务器配置为只接受来自受保护客户端的请求。 用于报告的 `FabricClient` 必须启用安全性才能与群集通信（例如使用 Kerberos 或证书身份验证）。 详细了解[群集安全性](/documentation/articles/service-fabric-cluster-security/)。
> 
> 

## <a name="report-from-within-low-privilege-services"></a>在低特权的服务内进行报告
在对群集不具有管理员访问权限的 Service Fabric 服务内，你可以通过 `Partition` 或 `CodePackageActivationContext`，报告来自当前上下文的实体的运行状况。

* 对于无状态服务，使用 [IStatelessServicePartition.ReportInstanceHealth](https://docs.microsoft.com/dotnet/api/system.fabric.istatelessservicepartition.reportinstancehealth) 报告当前服务实例的运行状况。
* 对于有状态服务，使用 [IStatefulServicePartition.ReportReplicaHealth](https://docs.microsoft.com/dotnet/api/system.fabric.istatefulservicepartition.reportreplicahealth) 报告当前副本的运行状况。
* 使用 [IServicePartition.ReportPartitionHealth](https://docs.microsoft.com/dotnet/api/system.fabric.iservicepartition.reportpartitionhealth) 报告当前分区实体的运行状况。
* 使用 [CodePackageActivationContext.ReportApplicationHealth](https://docs.microsoft.com/dotnet/api/system.fabric.codepackageactivationcontext.reportapplicationhealth) 报告当前应用程序的运行状况。
* 使用 [CodePackageActivationContext.ReportDeployedApplicationHealth](https://docs.microsoft.com/dotnet/api/system.fabric.codepackageactivationcontext.reportdeployedapplicationhealth) 报告当前节点上部署的当前应用程序的运行状况。
* 使用 [CodePackageActivationContext.ReportDeployedServicePackageHealth](https://docs.microsoft.com/dotnet/api/system.fabric.codepackageactivationcontext.reportdeployedservicepackagehealth) 报告当前节点上部署的当前应用程序的服务包的运行状况。

> [AZURE.NOTE]
> 就内部而言，`Partition` 和 `CodePackageActivationContext` 会保留使用默认设置配置的运行状况客户端。 注意事项对[运行状况客户端](/documentation/articles/service-fabric-report-health/#health-client)同样适用 - 由于报告是通过计时器进行批处理和发送，因此对象应保持可用状态，才有机会发送报告。
> 
> 

## <a name="design-health-reporting"></a>设计运行状况报告
生成高质量报告的第一步是识别可能影响服务运行状况的条件。 在条件启动甚至发生之前，任何有助于在服务或群集中标记问题的条件，都有可能节约数十亿元的费用。 优点包括故障时间变少，晚上花在调查和修复问题上的时间变少，客户满意度自然也会更高。

识别条件后，监视器编写人员需要找出最佳的监视方式，以实现开销和实用性的平衡。例如，设想某个服务使用某个共享上的一些临时文件进行复杂计算。监视器可以监视该共享，以确保有足够空间可用。它可以侦听文件或目录更改通知。它可以在达到预先阈值时报告警告，在共享已满时报告错误。报告警告时，修复系统可以开始清理共享上较旧的文件。报告错误时，修复系统可将服务副本移至另一个节点。请注意依据运行状况描述条件状态的方式：何种条件状态可视为正常，何种条件状态可视为不正常（警告或错误）。

设定好监视的详细信息之后，监视器编写人员需要了解如何实现监视器。如果条件可在服务内确定，则监视器可以成为受监视服务本身的一部分。例如，服务代码可以在本地结构客户端每次尝试写入文件时，检查并报告共享使用量。这个方法的优点是报告变得轻而易举。为避免监视器的 bug 影响到服务功能，必须多加留意。

并非一定要在受监视的服务内报告。服务中的监视程序可能无法检测状态。可能没有逻辑或数据可供做出判断。监视状态的开销可能很高。状态也可能不特定于某项服务，而会影响服务之间的交互。也可以选择在群集中将监视器作为独立进程。监视器只监视条件和报告，不会以任何方式影响主要服务。例如，这些监视器可在相同应用程序中以无状态服务的形式实现，或在所有节点或相同节点上作为服务部署。

有时，也并非一定要在群集中运行监视器。如果受监视的条件是用户所见的服务可用性或功能，监视器最好能与用户客户端位于相同的位置。这样，就可以采用与用户调用操作相同的方式来测试操作。例如，监视器可以存留于群集外部并对服务发出请求，然后检查结果的延迟和正确性。（例如在计算器服务中，2+2 是否在合理的时间内返回 4？）

确定监视器详细信息后，应该确定可唯一标识它的源 ID。 如果多个相同类型的监视器存留于群集中，它们必须报告不同的实体，如果它们报告相同的实体，请确保源 ID 或属性不相同。 这样报告才能共存。 运行状况报告的属性应捕获受监视的条件。 （对于上述示例，属性可以是 **ShareSize**。）如果多个报告应用于同一条件，该属性应包含一些动态信息，才可让报告共存。 例如，如果需要监视多个共享，属性名称可以是 **ShareSize-sharename**。

> [AZURE.NOTE]
> 运行状况存储*不得*用于保存状态信息。 只有与运行状况相关的信息才应作为运行状况进行报告，即影响实体运行状况评估的信息。 运行状况存储并非设计作为一般用途的存储。 它使用运行状况评估逻辑将所有数据聚合到运行状况中。 发送与运行状况无关的信息（例如，报告运行状况为“正常”的状态）不会影响聚合的运行状态，但可能对运行状况存储的性能造成负面影响。
> 
> 

下一个决策点就是需要报告的实体。大多数情况下，显然是依据条件而定。应该选择具有最佳粒度的实体。如果条件影响到某个分区中的所有副本，则报告该分区，而非服务。以下是需要仔细考虑的极端案例。如果条件影响到实体（例如副本），但需要将条件标记为超过副本生存期，则应报告分区。否则，删除副本时，与其相关的所有报告都会从存储中清除。这意味着监视器编写人员也必须将实体和报告的生存期纳入考虑范围。必须清楚说明应从存储中清除报告的时间点（例如，针对实体报告的错误不再适用时）。

让我们以一个例子解释上述要点。假设在所有节点上部署一个 Service Fabric 应用程序，该应用程序由一个主要的有状态持久性服务和多个次要的无状态服务组成（每种任务类型具有一种次要服务类型）。主服务有一个处理队列，队列中包含次要服务需要执行的命令。次要服务执行传入请求，并发回确认信号。可以监视的条件之一是主要服务的处理队列长度。如果主服务队列长度达到阈值，则报告警告。该警告指出辅助服务无法处理负载。如果队列达到长度上限，而且命令已删除，则会因为服务无法恢复而报告错误。可以针对属性 **QueueStatus** 发送报告。监视器位于服务内部，它会在主要服务的主要副本上定期发送报告。生存时间为 2 分钟，将每隔 30 秒定期发送一次报告。如果主要副本发生故障，报告会自动从存储中清除。如果服务副本已启用，但发生死锁或有其他问题，该报告将在运行状况存储中过期。在这种情况下，将会错误地评估实体。

另一个可监视的条件是任务执行时间。主服务会根据任务类型将任务分发给次要服务。根据设计，主服务可以轮询次要服务以获取任务状态。它也可以等待次要服务在完成时发回确认信号。在第二种情况中，必须注意检测次要服务停止运行或消息丢失的情况。一种方法是主服务向同一个次要服务发送 Ping 请求，然后次要服务发回其状态。如果未收到状态，主服务将此视为失败并重新安排任务。此行为假设任务采用幂等模式。

如果任务未在特定时间（**t1**，例如 10 分钟）内完成，监视的条件可翻译为警告。 如果任务未按时（**t2**，例如 20 分钟）完成，监视的条件可翻译为错误。 此报告可以多种方式完成：

* 主服务的主要副本定期报告自身情况。 针对队列中的所有挂起任务可以有一个属性。 如果至少有一个任务耗时较长，则 **PendingTasks** 属性的报告状态为警告或错误（视情况而定）。 如果没有挂起的任务或所有任务已开始执行，报告状态为“正常”。 任务是持久性的。 如果主要副本发生故障，新升级的主要副本可继续适当地进行报告。
* 云或外部的另一个监视器进程检查任务（根据所需的任务结果从外部检查），以查看它们是否已完成。 如果它们不采用阈值，则发送有关主服务的报告。 此外还会发送有关每个任务的报告，其中包含任务标识符（例如 **PendingTask+taskId**）。 只有在状况不正常时才应发送报告。 将生存时间设置为几分钟，并将报告标记为到期时删除，以确保进行清理。
* 如果任务运行时间超出预期，执行任务的次要服务将发送报告。 它会报告 **PendingTasks**属性上的服务实例。 报告将指出有问题的服务实例，但它不会捕获实例停止运行的情况。 因为那时报告已清除完毕。 它可能会报告次要服务。 如果次要服务完成任务，次要实例将从存储中清除报告。 报告不会捕获确认消息丢失的情况，从主要服务的角度来看，任务并未完成。

不过，上述情况中的报告已完成，评估运行状况时，将在应用程序运行状况中捕获这些报告。

## <a name="report-periodically-vs-on-transition"></a>定期报告与转换时报告
使用运行状况报告模型，监视器可以定期发送报告，也可以在转换时发送报告。 建议定期发送监视器报告，因为代码要简单得多，不容易发生错误。 监视器必须尽可能简单，以免出现触发误报的 bug。 不正确的不正常  报告会影响运行状况评估以及基于运行状况的情况（包括升级）。 不正确的正常  报告会隐藏群集中的问题，我们不希望发生这种情况。

针对定期报告，可以使用计时器实现监视器。计时器回调时，监视器可以检查状态并根据当前状态发送报告。不需要查看先前发送的报告或在消息传送方面进行任何优化。运行状况客户端具有批处理逻辑，有助于提高性能。运行状况客户端保持活动状态时，将在内部不断重试，直到运行状况存储确认报告，或者监视器生成具有相同实体、属性和源的较新报告。

转换时报告需要注意状态处理。监视器会监视某些条件，仅当这些条件改变时才报告。此方法的优点是需要较少的报告。缺点是监视器的逻辑很复杂。监视器必须维护条件或报告，以便对其进行检查，判断状态变更。故障转移时，必须注意是否发送先前可能未发送的报告（已加入队列，但尚未发送至运行状况存储）。序列号必须不断递增。否则，报告将因为过时而被拒绝。在造成数据丢失的少数情况下，可能需要同步报告器的状态与运行状况存储的状态。

通过 `Partition` 或 `CodePackageActivationContext` 进行转换报告，对服务自行报告而言较为合理。删除本地对象（副本或已部署的服务包/已部署的应用程序）时，也会删除它的所有报告。这种自动清理会放宽在报告器和运行状况存储之间同步的需求。如果报告针对的是父分区或父应用程序，则在故障转移时必须小心谨慎，以免在运行状况存储中产生过时的报告。必须添加逻辑来维护正确的状态，并从存储中清除不再需要的报告。

## <a name="implement-health-reporting"></a>实现运行状况报告
了解实体和报告详细信息后，就可以通过 API、PowerShell 或 REST 发送运行状况报告。

### <a name="api"></a>API
若要通过 API 发送报告，必须特定于要报告的实体类型创建一个运行状况报告。 然后将报告提供给运行状况客户端。 或者，创建运行状况信息，并将其传递至 `Partition` 或 `CodePackageActivationContext` 上正确的报告方法，以报告当前实体的运行状况。

以下示例演示如何从群集内的监视器定期发送报告。该监视器会检查能否从节点内访问外部资源。应用程序内的服务清单需要该资源。如果无法访问该资源，应用程序内的其他服务仍然可以正常运行。因此，会在已部署的服务包实体上每隔 30 秒发送一次报告。


	private static Uri ApplicationName = new Uri("fabric:/WordCount");
	private static string ServiceManifestName = "WordCount.Service";
	private static string NodeName = FabricRuntime.GetNodeContext().NodeName;
	private static Timer ReportTimer = new Timer(new TimerCallback(SendReport), null, 30 * 1000, 30 * 1000);
	private static FabricClient Client = new FabricClient(new FabricClientSettings() { HealthReportSendInterval = TimeSpan.FromSeconds(0) });

	public static void SendReport(object obj)
	{
	    // Test whether the resource can be accessed from the node
	    HealthState healthState = this.TestConnectivityToExternalResource();

	    // Send report on deployed service package, as the connectivity is needed by the specific service manifest
	    // and can be different on different nodes
	    var deployedServicePackageHealthReport = new DeployedServicePackageHealthReport(
	        ApplicationName,
	        ServiceManifestName,
	        NodeName,
	        new HealthInformation("ExternalSourceWatcher", "Connectivity", healthState));

	    // TODO: handle exception. Code omitted for snippet brevity.
	    // Possible exceptions: FabricException with error codes
	    // FabricHealthStaleReport (non-retryable, the report is already queued on the health client),
	    // FabricHealthMaxReportsReached (retryable; user should retry with exponential delay until the report is accepted).
	    Client.HealthManager.ReportHealth(deployedServicePackageHealthReport);
	}

### <a name="powershell"></a>PowerShell
使用 **Send-ServiceFabric*EntityType*HealthReport** 发送运行状况报告。

以下示例演示如何定期报告某个节点上的 CPU 值。应每隔 30 秒发送一次报告，报告生存时间为 2 分钟。如果过期，就表示报告器有问题，因此会错误地评估该节点。CPU 高于阈值时，报告的运行状况为警告。CPU 保持高于阈值的配置时间时，则将其报告为错误。否则，报告器发送的运行状态为“正常”。


	PS C:\> Send-ServiceFabricNodeHealthReport -NodeName Node.1 -HealthState Warning -SourceId PowershellWatcher -HealthProperty CPU -Description "CPU is above 80% threshold" -TimeToLiveSec 120

	PS C:\> Get-ServiceFabricNodeHealth -NodeName Node.1
	NodeName              : Node.1
	AggregatedHealthState : Warning
	UnhealthyEvaluations  :
                        	Unhealthy event: SourceId='PowershellWatcher', Property='CPU', HealthState='Warning', ConsiderWarningAsError=false.

	HealthEvents          :
                        	SourceId              : System.FM
                        	Property              : State
                        	HealthState           : Ok
                        	SequenceNumber        : 5
                        	SentAt                : 4/21/2015 8:01:17 AM
                        	ReceivedAt            : 4/21/2015 8:02:12 AM
                        	TTL                   : Infinite
                        	Description           : Fabric node is up.
                        	RemoveWhenExpired     : False
                        	IsExpired             : False
                        	Transitions           : ->Ok = 4/21/2015 8:02:12 AM

                        	SourceId              : PowershellWatcher
                        	Property              : CPU
                        	HealthState           : Warning
                        	SequenceNumber        : 130741236814913394
                        	SentAt                : 4/21/2015 9:01:21 PM
                        	ReceivedAt            : 4/21/2015 9:01:21 PM
                        	TTL                   : 00:02:00
                        	Description           : CPU is above 80% threshold
                        	RemoveWhenExpired     : False
                        	IsExpired             : False
                        	Transitions           : ->Warning = 4/21/2015 9:01:21 PM

以下示例会在副本上报告暂时性警告。 它先获取分区 ID，再获取所需服务的副本 ID。 然后，从 **PowershellWatcher** 发送为属性 **ResourceDependency** 生成的报告。 此报告只需存在 2 分钟，就从存储中自动删除。

	PS C:\> $partitionId = (Get-ServiceFabricPartition -ServiceName fabric:/WordCount/WordCount.Service).PartitionId

	PS C:\> $replicaId = (Get-ServiceFabricReplica -PartitionId $partitionId | where {$_.ReplicaRole -eq "Primary"}).ReplicaId

	PS C:\> Send-ServiceFabricReplicaHealthReport -PartitionId $partitionId -ReplicaId $replicaId -HealthState Warning -SourceId PowershellWatcher -HealthProperty ResourceDependency -Description "The external resource that the primary is using has been rebooted at 4/21/2015 9:01:21 PM. Expect processing delays for a few minutes." -TimeToLiveSec 120 -RemoveWhenExpired

	PS C:\> Get-ServiceFabricReplicaHealth  -PartitionId $partitionId -ReplicaOrInstanceId $replicaId


	PartitionId           : 8f82daff-eb68-4fd9-b631-7a37629e08c0
	ReplicaId             : 130740415594605869
	AggregatedHealthState : Warning
	UnhealthyEvaluations  :
                        	Unhealthy event: SourceId='PowershellWatcher', Property='ResourceDependency', HealthState='Warning', ConsiderWarningAsError=false.

	HealthEvents          :
                        	SourceId              : System.RA
                        	Property              : State
                        	HealthState           : Ok
                        	SequenceNumber        : 130740768777734943
                        	SentAt                : 4/21/2015 8:01:17 AM
                        	ReceivedAt            : 4/21/2015 8:02:12 AM
                        	TTL                   : Infinite
                        	Description           : Replica has been created.
                        	RemoveWhenExpired     : False
                        	IsExpired             : False
                        	Transitions           : ->Ok = 4/21/2015 8:02:12 AM

                        	SourceId              : PowershellWatcher
                        	Property              : ResourceDependency
                        	HealthState           : Warning
                        	SequenceNumber        : 130741243777723555
                        	SentAt                : 4/21/2015 9:12:57 PM
                        	ReceivedAt            : 4/21/2015 9:12:57 PM
                        	TTL                   : 00:02:00
                        	Description           : The external resource that the primary is using has been rebooted 	at 4/21/2015 9:01:21 PM. Expect processing delays for a few minutes.
                        	RemoveWhenExpired     : True
                        	IsExpired             : False
                        	Transitions           : ->Warning = 4/21/2015 9:12:32 PM

### <a name="rest"></a>REST
通过 REST 使用 POST 请求发送运行状况报告，这些请求将发送到所需的实体，其正文中包含运行状况报告描述。 如需示例，请参阅有关如何发送 REST [群集运行状况报告](https://docs.microsoft.com/zh-cn/rest/api/servicefabric/report-the-health-of-a-cluster)或[服务运行状况报告](https://docs.microsoft.com/zh-cn/rest/api/servicefabric/report-the-health-of-a-service)的文档。 支持所有实体。

## <a name="next-steps"></a>后续步骤
根据运行状况数据，服务编写人员和群集/应用程序管理员可以思考使用这些信息的方法。 例如，他们可以根据运行状态设置警报，以便在出现导致服务中断的严重问题之前就将其捕获。 管理员还可以设置修复系统以便自动修复问题。

[Service Fabric 运行状况监视简介](/documentation/articles/service-fabric-health-introduction/)

[查看 Service Fabric 运行状况报告](/documentation/articles/service-fabric-view-entities-aggregated-health/)

[如何报告和检查服务运行状况](/documentation/articles/service-fabric-diagnostics-how-to-report-and-check-service-health/)

[使用系统运行状况报告进行故障排除](/documentation/articles/service-fabric-understand-and-troubleshoot-with-system-health-reports/)

[在本地监视和诊断服务](/documentation/articles/service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally/)

[Service Fabric 应用程序升级](/documentation/articles/service-fabric-application-upgrade/)
<!--Update_Description:update MSDN links to update to docs.microsoft.com links;wording update-->