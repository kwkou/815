
<properties
    pageTitle="更改 Azure Service Fabric 设置 | Azure"
    description="本文介绍可以自定义的结构设置和结构升级策略。"
    services="service-fabric"
    documentationcenter=".net"
    author="chackdan"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="7ced36bf-bd3f-474f-a03a-6ebdbc9677e2"
    ms.service="service-fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="02/15/2017"
    wacn.date="03/03/2017"
    ms.author="chackdan" />  


# 自定义 Service Fabric 群集设置和结构升级策略
本文档说明如何为 Service Fabric 群集自定义各种结构设置和结构升级策略。可以使用门户或 Azure Resource Manager 模板完成自定义。

## 可以自定义的结构设置
下面是可以自定义的结构设置：

### 节名称：Diagnostics
| **参数** | **允许的值** | **指导或简短说明** |
| --- | --- | --- |
| ConsumerInstances |字符串 |DCA 使用者实例列表。 |
| ProducerInstances |字符串 |DCA 生成者实例列表。 |
| AppEtwTraceDeletionAgeInDays |Int，默认值为 3 |在多少天后删除包含应用程序 ETW 跟踪的旧 ETL 文件。 |
| AppDiagnosticStoreAccessRequiresImpersonation |Bool，默认值为 true |代表应用程序访问诊断存储时是否需要模拟。 |
| MaxDiskQuotaInMB |Int，默认值为 65536 |Windows Fabric 日志文件的磁盘配额（以 MB 为单位）。 |
| DiskFullSafetySpaceInMB |Int，默认值为 1024 |要避免被 DCA 使用的剩余磁盘空间（以 MB 为单位）。 |
| ApplicationLogsFormatVersion |Int，默认值为 0 |应用程序日志格式的版本。支持的值为 0 和 1。版本 1 比版本 0 包含更多 ETW 事件记录的字段。 |
| ClusterId |String |群集的唯一 ID。在创建群集时生成。 |
| EnableTelemetry |Bool，默认值为 true |启用或禁用遥测。 |
| EnableCircularTraceSession |Bool，默认值为 false |标志指示是否应使用循环跟踪会话。 |

### 节名称：Trace/Etw
| **参数** | **允许的值** | **指导或简短说明** |
| --- | --- | --- |
| 级别 |Int，默认值为 4 |跟踪 ETW 级别可以使用值 1、2、3、4。若要受支持，必须将跟踪级别保持为 4 |

### 节名称：PerformanceCounterLocalStore
| **参数** | **允许的值** | **指导或简短说明** |
| --- | --- | --- |
| IsEnabled |Bool，默认值为 true |标志指示是否启用本地节点上的性能计数器集合。 |
| SamplingIntervalInSeconds |Int，默认值为 60 |正在收集的性能计数器的采样间隔。 |
| Counters |String |要收集的性能计数器的逗号分隔列表。 |
| MaxCounterBinaryFileSizeInMB |Int，默认值为 1 |每个性能计数器二进制文件的最大大小（以 MB 为单位）。 |
| NewCounterBinaryFileCreationIntervalInMinutes |Int，默认值为 10 |在其之后创建新的性能计数器二进制文件的最大间隔（以秒为单位）。 |

### 节名称：Setup
| **参数** | **允许的值** | **指导或简短说明** |
| --- | --- | --- |
| FabricDataRoot |字符串 |Service Fabric 数据根目录。Azure 的默认值为 d:\\svcfab |
| FabricLogRoot |字符串 |Service Fabric 日志根目录。这是 SF 日志和跟踪的放置位置。 |
| ServiceRunAsAccountName |String |运行结构主机服务的帐户名。 |
| ServiceStartupType |String |结构主机服务的启动类型。 |
| SkipFirewallConfiguration |Bool，默认值为 false |指定是否需要由系统设置防火墙设置。仅当使用 Windows 防火墙时才适用。如果使用第三方防火墙，必须打开要使用的系统和应用程序的端口 |

### 节名称：TransactionalReplicator
| **参数** | **允许的值** | **指导或简短说明** |
| --- | --- | --- |
| MaxCopyQueueSize |Uint，默认值为 16384 |这是用于定义队列初始大小的最大值，该队列用于维护复制操作。请注意，它必须是 2 的幂。如果在运行时该队列增长到此大小，将限制主复制器和辅助复制器之间的操作。 |
| BatchAcknowledgementInterval | 以秒为单位的时间，默认值为 0.015 | 指定以秒为单位的时间跨度。确定接收到操作后到发送回确认之前，复制器等待的时间。在该时间段期间接收的其他操作将通过一条消息发送回其确认 -> 减少网络流量，同时可能降低复制器的吞吐量。 |
| MaxReplicationMessageSize |Uint，默认值为 52428800 | 复制操作的最大消息大小。默认值为 50MB。 |
| ReplicatorAddress |Wstring，默认值为“localhost:0” | 采用字符串形式 -'IP:Port' 的终结点，Windows Fabric Replicator 将其用于与其他副本建立连接以发送/接收操作。 |
| InitialPrimaryReplicationQueueSize |Uint，默认值为 64 | 这是用于定义队列初始大小的值，该队列用于维护主复制器上的复制操作。请注意，它必须是 2 的幂。|
| MaxPrimaryReplicationQueueSize |Uint，默认值为 8192 |这是主复制队列中可以存在的最大操作数量。请注意，它必须是 2 的幂。 |
| MaxPrimaryReplicationQueueMemorySize |Uint，默认值为 0 |这是主复制队列的最大值（以字节为单位）。 |
| InitialSecondaryReplicationQueueSize |Uint，默认值为 64 |这是用于定义队列初始大小的值，该队列用于维护辅助复制器上的复制操作。请注意，它必须是 2 的幂。 |
| MaxSecondaryReplicationQueueSize |Uint，默认值为 16384 |这是辅助复制队列中可以存在的最大操作数量。请注意，它必须是 2 的幂。 |
| MaxSecondaryReplicationQueueMemorySize |Uint，默认值为 0 |这是辅助复制队列的最大值（以字节为单位）。 |
| SecondaryClearAcknowledgedOperations |Bool，默认值为 false |Bool，它控制向主复制器确认辅助复制器上的操作（刷新到磁盘）后，是否清除这些操作。将此值设置为 TRUE 可能会导致在故障转移后捕获副本的同时，新主复制器上出现其他磁盘读取。 |
| MaxMetadataSizeInKB |Int，默认值为 4 |日志流元数据的最大大小。 |
| MaxRecordSizeInKB |Uint，默认值为 1024 | 日志流记录的最大大小。 |
| CheckpointThresholdInMB |Int，默认值为 50 |日志使用量超过此值时，将启动检查点。 |
| MaxAccumulatedBackupLogSizeInMB |Int，默认值为 800 |给定备份日志链中备份日志的最大累积大小（以 MB 为单位）。如果增量备份会生成导致累积备份日志的备份日志，增量备份请求会失败，因为相关完整备份会大于此大小。在这种情况下，用户需要执行完整备份。 |
| MaxWriteQueueDepthInKB |Int，默认值为 0 | 最大写入队列深度的 Int，该写入队列深度指核心记录器可用于与此副本关联的日志的写入队列深度（以千字节为单位）。此值是核心记录器更新期间可以处于未完成状态的最大字节数。该值可能为 0，以便核心记录器计算适当值，或是 4 的倍数。 |
| SharedLogId |字符串 |共享日志标识符。这是一个 GUID，对于每个共享日志应该唯一。 |
| SharedLogPath |String |共享日志的路径。如果此值为空，则使用默认共享日志。 |
| SlowApiMonitoringDuration |以秒为单位的时间，默认值为 300 | 在触发运行状况事件警告前，指定 API 的持续时间。|
| MinLogSizeInMB |Int，默认值为 0 |事务日志的最小大小。不允许将日志截断为低于此设置的大小。0 表示复制器将根据其他设置确定最小日志大小。增加此值会提高执行部分复制和增量备份的可能性，因为这会降低截断相关日志记录的可能性。 |

### 节名称：FabricClient
| **参数** | **允许的值** | **指导或简短说明** |
| --- | --- | --- |
| NodeAddresses |Wstring，默认值为 "" |不同节点上的地址（连接字符串）的集合，可用于与命名服务通信。最初，客户端随机选择一个地址进行连接。如果提供了多个连接字符串且因通信或超时错误导致连接失败，客户端按顺序切换为使用下一个地址。请参阅“命名服务地址重试”部分，了解有关重试语义的详细信息。 |
| ConnectionInitializationTimeout |以秒为单位的时间，默认值为 2 |指定以秒为单位的时间跨度。每次客户端尝试打开网关连接时的连接超时间隔。 |
| PartitionLocationCacheLimit |Int，默认值为 100000 |为服务解析所缓存的分区数（设置为 0 表示无限制）。 |
| ServiceChangePollInterval |以秒为单位的时间，默认值为 120 |指定以秒为单位的时间跨度。服务的连续轮询之间的间隔从客户端更改为用于注册服务更改通知回调的网关。 |
| KeepAliveIntervalInSeconds |Int，默认值为 20 |FabricClient 传输向网关发送保持连接消息的时间间隔。值为 0，表示禁用 keepAlive。必须是正值。 |
| HealthOperationTimeout |以秒为单位的时间，默认值为 120 |指定以秒为单位的时间跨度。报告消息发送至运行状况管理器的超时时间。 |
| HealthReportSendInterval |以秒为单位的时间，默认值为 30 |指定以秒为单位的时间跨度。报告组件将累积的运行状况报告发送至运行状况管理器的时间间隔。 |
| HealthReportRetrySendInterval |以秒为单位的时间，默认值为 30 |指定以秒为单位的时间跨度。报告组件将累积的运行状况报告重新发送至运行状况管理器的时间间隔。 |
| RetryBackoffInterval |以秒为单位的时间，默认值为 3 |指定以秒为单位的时间跨度。重试操作之前的回退时间间隔。 |
| MaxFileSenderThreads |Uint，默认值为 10 |并行传输的最大文件数。 |

### 节名称：Common
| **参数** | **允许的值** | **指导或简短说明** |
| --- | --- | --- |
| PerfMonitorInterval |以秒为单位的时间，默认值为 1 |指定以秒为单位的时间跨度。性能监视时间间隔。设置为 0 或负值会禁用监视。 |

### 节名称：HealthManager
| **参数** | **允许的值** | **指导或简短说明** |
| --- | --- | --- |
| EnableApplicationTypeHealthEvaluation |Bool，默认值为 false |群集运行状况评估策略：启用按应用程序类型的运行状况评估。 |

### 节名称：FabricNode
| **参数** | **允许的值** | **指导或简短说明** |
| --- | --- | --- |
| StateTraceInterval |以秒为单位的时间，默认值为 300 |指定以秒为单位的时间跨度。FM/FMM 上每个节点及以上节点的节点状态跟踪的时间间隔。 |

### 节名称：NodeDomainIds
| **参数** | **允许的值** | **指导或简短说明** |
| --- | --- | --- |
| UpgradeDomainId |Wstring，默认值为 "" |描述节点所属的升级域。 |
| PropertyGroup |NodeFaultDomainIdCollection |描述节点所属的容错域。通过用于描述数据中心中节点所在位置的 URI 定义容错域。容错域 URI 的格式是 fd:/fd/，后接 URI 路径段。|

### 节名称：NodeProperties
| **参数** | **允许的值** | **指导或简短说明** |
| --- | --- | --- |
| PropertyGroup |NodePropertyCollectionMap |节点属性的字符串键值对的集合。 |

### 节名称：NodeCapacities
| **参数** | **允许的值** | **指导或简短说明** |
| --- | --- | --- |
| PropertyGroup |NodeCapacityCollectionMap |不同指标的节点容量集合。 |

### 节名称：FabricNode
| **参数** | **允许的值** | **指导或简短说明** |
| --- | --- | --- |
| StartApplicationPortRange |Int，默认值为 0 |由宿主子系统管理的应用程序端口的开始位置。当托管中的 EndpointFilteringEnabled 为 true 时为必需。 |
| EndApplicationPortRange |Int，默认值为 0 |由宿主子系统管理的应用程序端口的结束位置（不含）。当托管中的 EndpointFilteringEnabled 为 true 时为必需。 |
| ClusterX509StoreName |Wstring，默认值为“My” |X.509 证书存储的名称，该存储包含用于保护群集内部通信的群集证书。 |
| ClusterX509FindType |Wstring，默认值为“FindByThumbprint” |指示如何在由 ClusterX509StoreName 支持的值（“FindByThumbprint”和“FindBySubjectName”）指定的存储中搜索群集证书。使用“FindBySubjectName”时，如果有多个匹配项，使用到期时间最远的那一个。 |
| ClusterX509FindValue |Wstring，默认值为 "" |用于查找群集证书的搜索筛选器值。 |
| ClusterX509FindValueSecondary |Wstring，默认值为 "" |用于查找群集证书的搜索筛选器值。 |
| ServerAuthX509StoreName |Wstring，默认值为“My” |X.509 证书存储的名称，包含用于准入服务的服务器证书。 |
| ServerAuthX509FindType |Wstring，默认值为“FindByThumbprint” |指示如何在由 ServerAuthX509StoreName 支持的值（FindByThumbprint、FindBySubjectName）指定的存储中搜索服务器证书。 |
| ServerAuthX509FindValue |Wstring，默认值为 "" |用于查找服务器证书的搜索筛选器值。 |
| ServerAuthX509FindValueSecondary |Wstring，默认值为 "" |用于查找服务器证书的搜索筛选器值。 |
| ClientAuthX509StoreName |Wstring，默认值为“My” |X.509 证书存储的名称，包含默认管理员角色 FabricClient 的证书。 |
| ClientAuthX509FindType |Wstring，默认值为“FindByThumbprint” |指示如何在由 ClientAuthX509StoreName 支持的值（FindByThumbprint、FindBySubjectName）指定的存储中搜索证书。 |
| ClientAuthX509FindValue |Wstring，默认值为 "" | 用于查找默认管理员角色 FabricClient 的证书的搜索筛选器值。 |
| ClientAuthX509FindValueSecondary |Wstring，默认值为 "" |用于查找默认管理员角色 FabricClient 的证书的搜索筛选器值。 |
| UserRoleClientX509StoreName |Wstring，默认值为“My” |X.509 证书存储的名称，包含默认用户角色 FabricClient 的证书。 |
| UserRoleClientX509FindType |Wstring，默认值为“FindByThumbprint” |指示如何在由 UserRoleClientX509StoreName 支持的值（FindByThumbprint、FindBySubjectName）指定的存储中搜索证书。 |
| UserRoleClientX509FindValue |Wstring，默认值为 "" |用于查找默认用户角色 FabricClient 的证书的搜索筛选器值。 |
| UserRoleClientX509FindValueSecondary |Wstring，默认值为 "" |用于查找默认用户角色 FabricClient 的证书的搜索筛选器值。 |

### 节名称：Paas
| **参数** | **允许的值** | **指导或简短说明** |
| --- | --- | --- |
| ClusterId |Wstring，默认值为 "" |由结构用于配置保护的 X509 证书存储。 |

### 节名称：FabricHost
| **参数** | **允许的值** | **指导或简短说明** |
| --- | --- | --- |
| StopTimeout |以秒为单位的时间，默认值为 300 |指定以秒为单位的时间跨度。托管服务激活、停用和升级的超时时间。 |
| StartTimeout |以秒为单位的时间，默认值为 300 |指定以秒为单位的时间跨度。Fabricactivationmanager 启动的超时时间。 |
| ActivationRetryBackoffInterval |以秒为单位的时间，默认值为 5 |指定以秒为单位的时间跨度。每次激活失败的回退时间间隔；每次连续激活失败后，系统将重试激活最多 MaxActivationFailureCount 次。每次尝试的重试间隔是连续激活失败与激活退让间隔的积。 |
| ActivationMaxRetryInterval |以秒为单位的时间，默认值为 300 |指定以秒为单位的时间跨度。激活的最大重试时间间隔。每次连续失败后，重试时间间隔的计算结果为 Min（ActivationMaxRetryInterval；连续失败计数 * ActivationRetryBackoffInterval）（即取括号中的最小值）。 |
| ActivationMaxFailureCount |Int，默认值为 10 |这是系统在放弃前重试失败的激活的最大计数。 |
| EnableServiceFabricAutomaticUpdates |Bool，默认值为 false |这将通过 Windows 更新启用结构自动更新。 |
| EnableServiceFabricBaseUpgrade |Bool，默认值为 false |用于启用服务器的基本更新。 |
| EnableRestartManagement |Bool，默认值为 false |用于启用服务器重启。 |


### 节名称：FailoverManager
| **参数** | **允许的值** | **指导或简短说明** |
| --- | --- | --- |
| UserReplicaRestartWaitDuration |以秒为单位的时间，默认值为 60.0 * 30 |指定以秒为单位的时间跨度。指定以秒为单位的时间范围。当持久化副本不可用时，Windows Fabric 在创建新的替换副本（需要状态的副本）前，会等待该副本恢复正常，等待时间即为此持续时间。 |
| QuorumLossWaitDuration |以秒为单位的时间，默认值为 MaxValue |指定以秒为单位的时间跨度。这是允许分区处于仲裁丢失状态的最长持续时间。如果分区在此持续时间后仍然处于仲裁丢失状态，则通过将不可用副本视为已丢失，使分区从仲裁丢失状态中恢复。请注意，这可能会导致数据丢失。 |
| UserStandByReplicaKeepDuration |以秒为单位的时间，默认值为 3600.0 * 24 * 7 |指定以秒为单位的时间跨度。持久化副本从不可用状态恢复时，可能已被替换为另一副本。此计时器确定 FM 在放弃备用副本之前保留其多长时间。 |
| UserMaxStandByReplicaCount |Int，默认值为 1 |系统为用户服务保留的默认最大备用副本数。 |

### 节名称：NamingService
| **参数** | **允许的值** | **指导或简短说明** |
| --- | --- | --- |
| TargetReplicaSetSize |Int，默认值为 7 |命名服务存储的每个分区的副本集数量。增加副本集的数量会增加命名服务存储中信息的可靠性水平；减少此更改将导致信息由于节点故障而丢失；其代价是增加 Windows Fabric 上的负载以及对命名数据执行更新所花费的时间。|
|MinReplicaSetSize | Int，默认值为 3 | 完成更新时所需写入的最少命名服务副本数。如果系统中的活动副本数少于此数量，则可靠性系统将拒绝对命名服务存储执行的更新操作，直到副本还原为止。此值不应超过 TargetReplicaSetSize 的值。 |
|ReplicaRestartWaitDuration | 以秒为单位的时间，默认值为 (60.0 * 30)| 指定以秒为单位的时间跨度。命名服务副本不可用时，此计时器将启动。指定时间到期后，FM 将开始替换不可用的副本（尚不将其视作丢失）。 |
|QuorumLossWaitDuration | 以秒为单位的时间，默认值为 MaxValue | 指定以秒为单位的时间跨度。命名服务进入仲裁丢失状态时，此计时器将启动。指定时间到期后，FM 将不可用副本视为丢失，并尝试恢复仲裁。请注意，这可能会导致数据丢失。 |
|StandByReplicaKeepDuration | 以秒为单位的时间，默认值为 3600.0 * 2 | 指定以秒为单位的时间跨度。命名服务副本从不可用状态恢复时，可能已被替换为另一副本。此计时器确定 FM 在放弃备用副本之前保留其多长时间。 |
|PlacementConstraints | Wstring，默认值为 "" | 命名服务的放置约束。 |
|ServiceDescriptionCacheLimit | Int，默认值为 0 | 命名存储服务处的 LRU 服务说明缓存中可维持的最大条目数（设置为 0 表示无限制）。 |
|RepairInterval | 以秒为单位的时间，默认值为 5 | 指定以秒为单位的时间跨度。针对授权所有者和名称所有者之间命名不一致情况的修复操作的时间间隔。 |
|MaxNamingServiceHealthReports | Int，默认值为 10 | 命名存储服务一次所报告的运行不正常的最大慢速操作数量。如果设置为 0，则发送所有慢速操作。 |
| MaxMessageSize |Int，默认值为 4*1024*1024 |使用命名时客户端节点通信的最大消息大小。DOS 攻击缓解，默认值为 4MB。 |
| MaxFileOperationTimeout |以秒为单位的时间，默认值为 30 |指定以秒为单位的时间跨度。文件存储服务操作的最大超时时间。将拒绝指定更长超时时间的请求。 |
| MaxOperationTimeout |以秒为单位的时间，默认值为 600 |指定以秒为单位的时间跨度。客户端操作的最大允许超时时间。将拒绝指定更长超时时间的请求。 |
| MaxClientConnections |Int，默认值为 1000 |每个网关允许的最大客户端连接数。 |
| ServiceNotificationTimeout |以秒为单位的时间，默认值为 30 |指定以秒为单位的时间跨度。将服务通知传送到客户端时使用的超时时间。 |
| MaxOutstandingNotificationsPerClient |Int，默认值为 1000 |网关强行关闭客户端注册前的最大未完成通知数。 |
| MaxIndexedEmptyPartitions |Int，默认值为 1000 |将在通知缓存中保持被索引状态以同步重新连接的客户端的最大空分区数。将按查找版本的升序顺序，从索引中删除超出此数目的所有空分区。重新连接的客户端仍然可以同步并接收错过的空分区更新，但是同步协议的开销更大。 |
| GatewayServiceDescriptionCacheLimit |Int，默认值为 0 |命名网关处的 LRU 服务说明缓存中可维持的最大条目数（设置为 0 表示无限制）。 |
| PartitionCount |Int，默认值为 3 |要创建的命名服务存储的分区数。每个分区都拥有与其索引相对应的单个分区键，因此存在分区键 [0; PartitionCount)。增加命名服务分区数可减少由任何备份副本集保持的数据的平均量，从而增加命名服务可以执行的规模；其代价是增加资源的利用（因为必须维护 PartitionCount*ReplicaSetSize 服务副本）。|

### 节名称：RunAs
| **参数** | **允许的值** | **指导或简短说明** |
| --- | --- | --- |
| RunAsAccountName |Wstring，默认值为 "" |指示 RunAs 帐户名称。仅需用于“DomainUser”或“ManagedServiceAccount”帐户类型。有效值为“domain\\user”或“user@domain”。 |
|RunAsAccountType|Wstring，默认值为 "" |指示 RunAs 帐户类型。需用于任何 RunAs 部分，有效值为“DomainUser/NetworkService/ManagedServiceAccount/LocalSystem”。|
|RunAsPassword|Wstring，默认值为 "" |指示 RunAs 帐户密码。仅需用于“DomainUser”帐户类型。 |

### 节名称：RunAs\_Fabric
| **参数** | **允许的值** | **指导或简短说明** |
| --- | --- | --- |
| RunAsAccountName |Wstring，默认值为 "" |指示 RunAs 帐户名称。仅需用于“DomainUser”或“ManagedServiceAccount”帐户类型。有效值为“domain\\user”或“user@domain”。 |
|RunAsAccountType|Wstring，默认值为 "" |指示 RunAs 帐户类型。需用于任何 RunAs 部分，有效值为“LocalUser/DomainUser/NetworkService/ManagedServiceAccount/LocalSystem”。 |
|RunAsPassword|Wstring，默认值为 "" |指示 RunAs 帐户密码。仅需用于“DomainUser”帐户类型。 |

### 节名称：RunAs\_HttpGateway
| **参数** | **允许的值** | **指导或简短说明** |
| --- | --- | --- |
| RunAsAccountName |Wstring，默认值为 "" |指示 RunAs 帐户名称。仅需用于“DomainUser”或“ManagedServiceAccount”帐户类型。有效值为“domain\\user”或“user@domain”。 |
|RunAsAccountType|Wstring，默认值为 "" |指示 RunAs 帐户类型。需用于任何 RunAs 部分，有效值为“LocalUser/DomainUser/NetworkService/ManagedServiceAccount/LocalSystem”。 |
|RunAsPassword|Wstring，默认值为 "" |指示 RunAs 帐户密码。仅需用于“DomainUser”帐户类型。 |

### 节名称：RunAs\_DCA
| **参数** | **允许的值** | **指导或简短说明** |
| --- | --- | --- |
| RunAsAccountName |Wstring，默认值为 "" |指示 RunAs 帐户名称。仅需用于“DomainUser”或“ManagedServiceAccount”帐户类型。有效值为“domain\\user”或“user@domain”。 |
|RunAsAccountType|Wstring，默认值为 "" |指示 RunAs 帐户类型。需用于任何 RunAs 部分，有效值为“LocalUser/DomainUser/NetworkService/ManagedServiceAccount/LocalSystem”。 |
|RunAsPassword|Wstring，默认值为 "" |指示 RunAs 帐户密码。仅需用于“DomainUser”帐户类型。 |

### 节名称：HttpGateway
| **参数** | **允许的值** | **指导或简短说明** |
| --- | --- | --- |
|IsEnabled|Bool，默认值为 false | 启用/禁用 httpgateway。默认情况下，禁用 Httpgateway，需要设置此配置以启用 Httpgateway。 |
|ActiveListeners |Uint，默认值为 50 | 要发布到 http 服务器队列的读取数。此配置控制 HttpGateway 可以满足的并发请求数。 |
|MaxEntityBodySize |Uint，默认值为 4194304 | 提供可预期的 http 请求正文的最大大小。默认值为 4MB。如果请求的正文大小大于此值，Httpgateway 将无法满足该请求。最小读取块区大小为 4096 个字节。因此，该值必须 > = 4096。 |

### 节名称：KtlLogger
| **参数** | **允许的值** | **指导或简短说明** |
| --- | --- | --- |
|AutomaticMemoryConfiguration |Int，默认值为 1 | 该标志指示是否应自动且动态地配置内存设置。如果设置为 0，则根据系统条件直接使用内存配置设置而不进行任何更改。如果设置为 1，则自动配置内存设置，并可根据系统条件进行更改。 |
|WriteBufferMemoryPoolMinimumInKB |Int，默认值为 8388608 |最初为写入缓冲区内存池分配的 KB 数。设置为 0 表示没有限制，默认值应与以下 SharedLogSizeInMB 值保持一致。 |
|WriteBufferMemoryPoolMaximumInKB | Int，默认值为 0 |允许写入缓冲区内存池增长到的 KB 数。使用 0 表示没有限制。 |
|MaximumDestagingWriteOutstandingInKB | Int，默认值为 0 | 共享日志可位于专用日志之前的 KB 数。使用 0 表示没有限制。
|SharedLogPath |Wstring，默认值为 "" | 要放置共享日志容器的位置的路径和文件名。设置为 "" 表示使用结构数据根目录下的默认路径。 |
|SharedLogId |Wstring，默认值为 "" |共享日志容器的唯一 GUID。若要使用结构数据根目录下的默认路径，请设置为 ""。 |
|SharedLogSizeInMB |Int，默认值为 8192 | 共享日志容器中要分配的 MB 数。 |

### 节名称：ApplicationGateway/Http
| **参数** | **允许的值** | **指导或简短说明** |
| --- | --- | --- |
|IsEnabled |Bool，默认值为 false | 启用/禁用 HttpApplicationGateway。默认情况下，禁用 HttpApplicationGateway，需要设置此配置以启用 HttpApplicationGateway。 |
|NumberOfParallelOperations | Uint，默认值为 1000 | 要发布到 http 服务器队列的读取数。此配置控制 HttpGateway 可以满足的并发请求数。 |
|DefaultHttpRequestTimeout |以秒为单位的时间，默认值为 60 |指定以秒为单位的时间跨度。提供用于 http 应用网关中正在处理的 http 请求的默认请求超时时间。 |
|ResolveServiceBackoffInterval |以秒为单位的时间，默认值为 5 |指定以秒为单位的时间跨度。提供重试失败的解析服务操作之前的默认回退时间间隔。 |
|BodyChunkSize |Uint，默认值为 4096 | 提供用于读取正文的区块大小（以字节为单位）。 |
|GatewayAuthCredentialType |Wstring，默认值为“None” | 指示在 http 应用网关终结点处使用的安全凭据的类型，有效值为“None/X509”。 |
|GatewayX509CertificateStoreName |Wstring，默认值为“My” | 包含 http 应用网关证书的 X.509 证书存储的名称。 |
|GatewayX509CertificateFindType |Wstring，默认值为“FindByThumbprint” | 指示如何在由 GatewayX509CertificateStoreName 支持的值（FindByThumbprint、FindBySubjectName）指定的存储中搜索证书。 |
|GatewayX509CertificateFindValue | Wstring，默认值为 "" | 用于查找 http 应用网关证书的搜索筛选器值。此证书在 https 终结点上配置，并且如果服务需要，还可用于验证应用的标识。首先查找 FindValue，如果其不存在，再查找 FindValueSecondary。 |
|GatewayX509CertificateFindValueSecondary | Wstring，默认值为 "" |用于查找 http 应用网关证书的搜索筛选器值。此证书在 https 终结点上配置，并且如果服务需要，还可用于验证应用的标识。首先查找 FindValue，如果其不存在，再查找 FindValueSecondary。|

### 节名称：Management
| **参数** | **允许的值** | **指导或简短说明** |
| --- | --- | --- |
| ImageStoreConnectionString |SecureString | ImageStore 的根的连接字符串。 |
| ImageStoreMinimumTransferBPS | Int，默认值为 1024 |群集和 ImageStore 之间的最小传输速率。此值用于确定访问外部 ImageStore 时的超时时间。仅当群集和 ImageStore 之间的延迟较高时可更改此值，以允许群集获得更多的时间从外部 ImageStore 进行下载。 |
|AzureStorageMaxWorkerThreads | Int，默认值为 25 | 最大并行工作线程数。 |
|AzureStorageMaxConnections | Int，默认值为 5000 | 最大并发 Azure 存储连接数。 |
|AzureStorageOperationTimeout | 以秒为单位的时间，默认值为 6000 | 指定以秒为单位的时间跨度。完成 xstore 操作时的超时时间。 |
|ImageCachingEnabled | Bool，默认值为 true | 通过此配置可启用或禁用缓存。 |
|DisableChecksumValidation | Bool，默认值为 false | 通过此配置可在应用程序预配过程中启用或禁用校验和验证。 |
|DisableServerSideCopy | Bool，默认值为 false | 此配置可以在应用程序预配过程中启用或禁用 ImageStore 上应用程序包的服务器端副本。 |

### 节名称：HealthManager/ClusterHealthPolicy
| **参数** | **允许的值** | **指导或简短说明** |
| --- | --- | --- |
| ConsiderWarningAsError |Bool，默认值为 false |群集运行状况评估策略：警告视为错误。 |
| MaxPercentUnhealthyNodes | Int，默认值为 0 |群集运行状况评估策略：为确保群集运行正常，可允许的最高运行不正常节点百分比。 |
| MaxPercentUnhealthyApplications | Int，默认值为 0 |群集运行状况评估策略：为确保群集运行正常，可允许的最高运行不正常应用程序百分比。 |
|MaxPercentDeltaUnhealthyNodes | Int，默认值为 10 |群集升级运行状况评估策略：为确保群集运行正常，可允许的最高运行不正常节点增量百分比。 |
|MaxPercentUpgradeDomainDeltaUnhealthyNodes | Int，默认值为 15 |群集升级运行状况评估策略：为确保群集运行正常，升级域中可允许的最高运行不正常节点增量百分比。|

### 节名称：FaultAnalysisService
| **参数** | **允许的值** | **指导或简短说明** |
| --- | --- | --- |
| TargetReplicaSetSize |Int，默认值为 0 |NOT\_PLATFORM\_UNIX\_START，FaultAnalysisService 的 TargetReplicaSetSize。 |
| MinReplicaSetSize |Int，默认值为 0 |FaultAnalysisService 的 MinReplicaSetSize。 |
| ReplicaRestartWaitDuration |以秒为单位的时间，默认值为 60 分钟|指定以秒为单位的时间跨度。FaultAnalysisService 的 ReplicaRestartWaitDuration。 |
| QuorumLossWaitDuration | 以秒为单位的时间，默认值为 MaxValue |指定以秒为单位的时间跨度。FaultAnalysisService 的 QuorumLossWaitDuration。 |
| StandByReplicaKeepDuration| 以秒为单位的时间，默认值为 (60*24*7) 分钟 |指定以秒为单位的时间跨度。FaultAnalysisService 的 StandByReplicaKeepDuration。 |
| PlacementConstraints | Wstring，默认值为 ""| FaultAnalysisService 的 PlacementConstraints。 |
| StoredActionCleanupIntervalInSeconds | Int，默认值为 3600 |这是清理存储的频率。仅会删除处于终态和至少在 CompletedActionKeepDurationInSeconds 以前完成的操作。 |
| CompletedActionKeepDurationInSeconds | Int，默认值为 604800 | 这是处于终态的操作的大致保留时长。这也取决于 StoredActionCleanupIntervalInSeconds，因为仅在此间隔时间内执行清理工作。604800 秒等于 7 天。 |
| StoredChaosEventCleanupIntervalInSeconds | Int，默认值为 3600 |这是审核存储（以进行清理）的频率，如果事件数量超过 30000，则开始执行清理。 |

### 节名称：FileStoreService
| **参数** | **允许的值** | **指导或简短说明** |
| --- | --- | --- |
| NamingOperationTimeout |以秒为单位的时间，默认值为 60 |指定以秒为单位的时间跨度。执行命名操作时的超时时间。 |
| QueryOperationTimeout | 以秒为单位的时间，默认值为 60 |指定以秒为单位的时间跨度。执行查询操作的超时时间。 |
| MaxCopyOperationThreads | Uint，默认值为 0 | 辅助节点可从主节点复制的最大并行文件数。'0' == 核心数。 |
| MaxFileOperationThreads | Uint，默认值为 100 | 可在主节点中执行 FileOperations（复制/移动）操作的最大并行线程数。'0' == 核心数。 |
| MaxStoreOperations | Uint，默认值为 4096 |主节点上允许的最大并行存储事务操作数。'0' == 核心数。 |
| MaxRequestProcessingThreads | Uint，默认值为 200 |可在主节点中处理请求的最大并行线程数。'0' == 核心数。 |
| MaxSecondaryFileCopyFailureThreshold | Uint，默认值为 25| 放弃前，辅助节点上的最大文件副本重试次数。 |
| AnonymousAccessEnabled | Bool，默认值为 true |启用/禁用对 FileStoreService 共享的匿名访问。 |
| PrimaryAccountType | Wstring，默认值为 "" |FileStoreService 共享的 ACL 主体的主 AccountType。 |
| PrimaryAccountUserName | Wstring，默认值为 "" |FileStoreService 共享的 ACL 主体的主帐户用户名。 |
| PrimaryAccountUserPassword | SecureString，默认值为空 |FileStoreService 共享的 ACL 主体的主帐户密码。 |
| FileStoreService | PrimaryAccountNTLMPasswordSecret | SecureString，默认值为空 |
| PrimaryAccountNTLMX509StoreLocation | Wstring，默认值为“LocalMachine”| 使用 NTLM 身份验证时，用于在 PrimaryAccountNTLMPasswordSecret 上生成 HMAC 的 X509 证书的存储位置。 |
| PrimaryAccountNTLMX509StoreName | Wstring，默认值为“MY”| 使用 NTLM 身份验证时，用于在 PrimaryAccountNTLMPasswordSecret 上生成 HMAC 的 X509 证书的存储名称。 |
| PrimaryAccountNTLMX509Thumbprint | Wstring，默认值为 ""|使用 NTLM 身份验证时，用于在 PrimaryAccountNTLMPasswordSecret 上生成 HMAC 的 X509 证书的指纹。 |
| SecondaryAccountType | Wstring，默认值为 ""| FileStoreService 共享的 ACL 主体的辅助 AccountType。 |
| SecondaryAccountUserName | Wstring，默认值为 ""| FileStoreService 共享的 ACL 主体的辅助帐户用户名。 |
| SecondaryAccountUserPassword | SecureString，默认值为空 |FileStoreService 共享的 ACL 主体的辅助帐户密码。 |
| SecondaryAccountNTLMPasswordSecret | SecureString，默认值为空 | 密码机密，使用 NTLM 身份验证时用作种子来生成相同的密码。 |
| SecondaryAccountNTLMX509StoreLocation | Wstring，默认值为“LocalMachine” |使用 NTLM 身份验证时，用于在 SecondaryAccountNTLMPasswordSecret 上生成 HMAC 的 X509 证书的存储位置。 |
| SecondaryAccountNTLMX509StoreName | Wstring，默认值为“MY” |使用 NTLM 身份验证时，用于在 SecondaryAccountNTLMPasswordSecret 上生成 HMAC 的 X509 证书的存储名称。 |
| SecondaryAccountNTLMX509Thumbprint | Wstring，默认值为 ""| 使用 NTLM 身份验证时，用于在 SecondaryAccountNTLMPasswordSecret 上生成 HMAC 的 X509 证书的指纹。 |

### 节名称：ImageStoreService
| **参数** | **允许的值** | **指导或简短说明** |
| --- | --- | --- |
| Enabled |Bool，默认值为 false |ImageStoreService 的 Enabled 标志。 |
| TargetReplicaSetSize | Int，默认值为 7 |ImageStoreService 的 TargetReplicaSetSize。 |
| MinReplicaSetSize | Int，默认值为 3 |ImageStoreService 的 MinReplicaSetSize。 |
| ReplicaRestartWaitDuration | 以秒为单位的时间，默认值为 60.0 * 30 | 指定以秒为单位的时间跨度。ImageStoreService 的 ReplicaRestartWaitDuration。 |
| QuorumLossWaitDuration | 以秒为单位的时间，默认值为 MaxValue | 指定以秒为单位的时间跨度。ImageStoreService 的 QuorumLossWaitDuration。 |
| StandByReplicaKeepDuration | 以秒为单位的时间，默认值为 3600.0 * 2 | 指定以秒为单位的时间跨度。ImageStoreService 的 StandByReplicaKeepDuration。 |
| PlacementConstraints | Wstring，默认值为 "" | ImageStoreService 的 PlacementConstraints。 |
| ClientUploadTimeout | 以秒为单位的时间，默认值为 1800 |指定以秒为单位的时间跨度。对映像存储服务的顶级上载请求的超时值。 |
| ClientCopyTimeout | 以秒为单位的时间，默认值为 1800 | 指定以秒为单位的时间跨度。对映像存储服务的顶级复制请求的超时值。 |
| ClientDownloadTimeout | 以秒为单位的时间，默认值为 1800 | 指定以秒为单位的时间跨度。对映像存储服务的顶级下载请求的超时值 |
| ClientListTimeout | 以秒为单位的时间，默认值为 600 | 指定以秒为单位的时间跨度。对映像存储服务的顶级列出请求的超时值。 |
| ClientDefaultTimeout | 以秒为单位的时间，默认值为 180 | 指定以秒为单位的时间跨度。对映像存储服务的所有未上载/未下载请求（如 exists、delete）的超时值。 |

### 节名称：ImageStoreClient
| **参数** | **允许的值** | **指导或简短说明** |
| --- | --- | --- |
| ClientUploadTimeout |以秒为单位的时间，默认值为 1800 | 指定以秒为单位的时间跨度。对映像存储服务的顶级上载请求的超时值。 |
| ClientCopyTimeout | 以秒为单位的时间，默认值为 1800 | 指定以秒为单位的时间跨度。对映像存储服务的顶级复制请求的超时值。 |
|ClientDownloadTimeout | 以秒为单位的时间，默认值为 1800 | 指定以秒为单位的时间跨度。对映像存储服务的顶级下载请求的超时值。 |
|ClientListTimeout | 以秒为单位的时间，默认值为 600 |指定以秒为单位的时间跨度。对映像存储服务的顶级列出请求的超时值。 |
|ClientDefaultTimeout | 以秒为单位的时间，默认值为 180 | 指定以秒为单位的时间跨度。对映像存储服务的所有未上载/未下载请求（如 exists、delete）的超时值。 |

### 节名称：TokenValidationService
| **参数** | **允许的值** | **指导或简短说明** |
| --- | --- | --- |
| 提供程序 |Wstring，默认值为“DSTS” |要启用的令牌验证提供程序的逗号分隔列表（有效的提供程序是：DSTS、AAD）。目前只能在任何时候启用单个提供程序。 |

### 节名称：UpgradeOrchestrationService
| **参数** | **允许的值** | **指导或简短说明** |
| --- | --- | --- |
| TargetReplicaSetSize |Int，默认值为 0 |UpgradeOrchestrationService 的 TargetReplicaSetSize。 |
| MinReplicaSetSize |Int，默认值为 0 | UpgradeOrchestrationService 的 MinReplicaSetSize。
| ReplicaRestartWaitDuration | 以秒为单位的时间，默认值为 60 分钟| 指定以秒为单位的时间跨度。UpgradeOrchestrationService 的 ReplicaRestartWaitDuration。 |
| QuorumLossWaitDuration | 以秒为单位的时间，默认值为 MaxValue | 指定以秒为单位的时间跨度。UpgradeOrchestrationService 的 QuorumLossWaitDuration。 |
| StandByReplicaKeepDuration | 以秒为单位的时间，默认值为 60*24*7 分钟 | 指定以秒为单位的时间跨度。UpgradeOrchestrationService 的 StandByReplicaKeepDuration。 |
| PlacementConstraints | Wstring，默认值为 "" | UpgradeOrchestrationService 的 PlacementConstraints。 |
| AutoupgradeEnabled | Bool，默认值为 true | 基于目标状态文件的自动轮询和升级操作。 |
| UpgradeApprovalRequired | Bool，默认值为 false | 此设置可让升级代码需要管理员批准才能继续操作。 |

### 节名称：UpgradeService
| **参数** | **允许的值** | **指导或简短说明** |
| --- | --- | --- |
| PlacementConstraints |Wstring，默认值为 "" |升级服务的 PlacementConstraints。 |
| TargetReplicaSetSize | Int，默认值为 3 | UpgradeService 的 TargetReplicaSetSize。 |
| MinReplicaSetSize | Int，默认值为 2 | UpgradeService 的 MinReplicaSetSize。 |
| CoordinatorType | Wstring，默认值为“WUTest”| UpgradeService 的 CoordinatorType。 |
| BaseUrl | Wstring，默认值为 "" |UpgradeService 的 BaseUrl。 |
| ClusterId | Wstring，默认值为 "" | UpgradeService 的 ClusterId。 |
| X509StoreName | Wstring，默认值为“My”| UpgradeService 的 X509StoreName。 |
| X509StoreLocation | Wstring，默认值为 "" | UpgradeService 的 X509StoreLocation。 |
| X509FindType | Wstring，默认值为 ""| UpgradeService 的 X509FindType。 |
| X509FindValue | Wstring，默认值为 "" | UpgradeService 的 X509FindValue。 |
| X509SecondaryFindValue | Wstring，默认值为 "" | UpgradeService 的 X509SecondaryFindValue。 |
| OnlyBaseUpgrade | Bool，默认值为 false | UpgradeService 的 OnlyBaseUpgrade。 |
| TestCabFolder | Wstring，默认值为 "" | UpgradeService 的 TestCabFolder。 |

### 节名称：Security/ClientAccess
| **参数** | **允许的值** | **指导或简短说明** |
| --- | --- | --- |
| CreateName |Wstring，默认值为“Admin” |用于创建命名 URI 的安全配置。 |
| DeleteName |Wstring，默认值为“Admin” |用于删除命名 URI 的安全配置。 |
| PropertyWriteBatch |Wstring，默认值为“Admin” |用于 Naming 属性写入操作的安全配置。 |
| CreateService |Wstring，默认值为“Admin” | 用于创建服务的安全配置。 |
| CreateServiceFromTemplate |Wstring，默认值为“Admin” |用于通过模板创建服务的安全配置。 |
| UpdateService |Wstring，默认值为“Admin” |用于服务更新的安全配置。 |
| DeleteService |Wstring，默认值为“Admin” |用于删除服务的安全配置。 |
| ProvisionApplicationType |Wstring，默认值为“Admin” | 用于预配应用程序类型的安全配置。 |
| CreateApplication |Wstring，默认值为“Admin” | 用于创建应用程序的安全配置。 |
| DeleteApplication |Wstring，默认值为“Admin” | 用于删除应用程序的安全配置。 |
| UpgradeApplication |Wstring，默认值为“Admin” | 用于启动或中断应用程序升级的安全配置。 |
| RollbackApplicationUpgrade |Wstring，默认值为“Admin” | 用于回滚应用程序升级的安全配置。 |
| UnprovisionApplicationType |Wstring，默认值为“Admin” | 用于取消预配应用程序类型的安全配置。 |
| MoveNextUpgradeDomain |Wstring，默认值为“Admin” | 用于使用显式升级域恢复应用程序升级的安全配置。 |
| ReportUpgradeHealth |Wstring，默认值为“Admin” | 用于恢复应用程序升级并提供当前升级进度的安全配置。 |
| ReportHealth |Wstring，默认值为“Admin” | 用于报告运行状况的安全配置。 |
| ProvisionFabric |Wstring，默认值为“Admin” | 用于预配 MSI 和/或群集清单的安全配置。 |
| UpgradeFabric |Wstring，默认值为“Admin” | 用于启动群集升级的安全配置。 |
| RollbackFabricUpgrade |Wstring，默认值为“Admin” | 用于回滚群集升级的安全配置。 |
| UnprovisionFabric |Wstring，默认值为“Admin” | 用于取消预配 MSI 和/或群集清单的安全配置。 |
| MoveNextFabricUpgradeDomain |Wstring，默认值为“Admin” | 用于使用显式升级域恢复群集升级的安全配置。 |
| ReportFabricUpgradeHealth |Wstring，默认值为“Admin” | 用于恢复群集升级并提供当前升级进度的安全配置。 |
| StartInfrastructureTask |Wstring，默认值为“Admin” | 用于启动基础结构任务的安全配置。 |
| FinishInfrastructureTask |Wstring，默认值为“Admin” | 用于完成基础结构任务的安全配置。 |
| ActivateNode |Wstring，默认值为“Admin” | 用于激活节点的安全配置。 |
| DeactivateNode |Wstring，默认值为“Admin” | 用于停用节点的安全配置。 |
| DeactivateNodesBatch |Wstring，默认值为“Admin” | 用于停用多个节点的安全配置。 |
| RemoveNodeDeactivations |Wstring，默认值为“Admin” | 用于在多个节点上还原停用操作的安全配置。 |
| GetNodeDeactivationStatus |Wstring，默认值为“Admin” | 用于检查停用状态的安全配置。 |
| NodeStateRemoved |Wstring，默认值为“Admin” | 用于报告删的节点状态的安全配置。 |
| RecoverPartition |Wstring，默认值为“Admin” | 用于恢复分区的安全配置。 |
| RecoverPartitions |Wstring，默认值为“Admin” | 用于恢复多个分区的安全配置。 |
| RecoverServicePartitions |Wstring，默认值为“Admin” | 用于恢复服务分区的安全配置。 |
| RecoverSystemPartitions |Wstring，默认值为“Admin” | 用于恢复系统服务分区的安全配置。 |
| ReportFault |Wstring，默认值为“Admin” | 用于报告故障的安全配置。 |
| InvokeInfrastructureCommand |Wstring，默认值为“Admin” | 用于基础结构任务管理命令的安全配置。 |
| FileContent |Wstring，默认值为“Admin” | 用于传输映像存储客户端文件（群集外部）的安全配置。 |
| FileDownload |Wstring，默认值为“Admin” | 用于启动映像存储客户端文件下载（群集外部）的安全配置。 |
| InternalList |Wstring，默认值为“Admin” | 用于映像存储客户端文件列表操作（内部）的安全配置。 |
| 删除 |Wstring，默认值为“Admin” | 用于映像存储客户端删除操作的安全配置。 |
| 上传 |Wstring，默认值为“Admin” | 用于映像存储客户端上载操作的安全配置。 |
| GetStagingLocation |Wstring，默认值为“Admin” | 用于检索映像存储客户端暂存位置的安全配置。 |
| GetStoreLocation |Wstring，默认值为“Admin” | 用于检索映像存储客户端存储位置的安全配置。 |
| NodeControl |Wstring，默认值为“Admin” | 用于启动、停止和重启节点的安全配置。 |
| CodePackageControl |Wstring，默认值为“Admin” | 用于重启代码包的安全配置。 |
| UnreliableTransportControl |Wstring，默认值为“Admin” | 添加和删除行为的不可靠传输。 |
| MoveReplicaControl |Wstring，默认值为“Admin” | 移动副本。 |
| PredeployPackageToNode |Wstring，默认值为“Admin” | 预部署 API。 |
| StartPartitionDataLoss |Wstring，默认值为“Admin” | 在分区上引入数据丢失。 |
| StartPartitionQuorumLoss |Wstring，默认值为“Admin” | 在分区上引入仲裁丢失。 |
| StartPartitionRestart |Wstring，默认值为“Admin” | 同时重启分区的部分或所有副本。 |
| CancelTestCommand |Wstring，默认值为“Admin” | 取消特定的 TestCommand（如果正在运行）。 |
| StartChaos |Wstring，默认值为“Admin” | 启动混沌 - 如果尚未启动。 |
| StopChaos |Wstring，默认值为“Admin” | 停止混沌 - 如果已启动。 |
| StartNodeTransition |Wstring，默认值为“Admin” | 用于启动节点转换的安全配置。 |
| StartClusterConfigurationUpgrade |Wstring，默认值为“Admin” | 在分区上引入 StartClusterConfigurationUpgrade。 |
| GetUpgradesPendingApproval |Wstring，默认值为“Admin” | 在分区上引入 GetUpgradesPendingApproval。 |
| StartApprovedUpgrades |Wstring，默认值为“Admin” | 在分区上引入 StartApprovedUpgrades。 |
| Ping |Wstring，默认值为“Admin\|\|User” | 用于客户 ping 的安全配置。 |
| 查询 |Wstring，默认值为“Admin\|\|User” | 用于查询的安全配置。 |
| NameExists |Wstring，默认值为“Admin\|\|User” | 用于检查是否存在命名 URI 的安全配置。 |
| EnumerateSubnames |Wstring，默认值为“Admin\|\|User” | 用于枚举命名 URI 的安全配置。 |
| EnumerateProperties |Wstring，默认值为“Admin\|\|User” | 用于枚举 Naming 属性的安全配置。 |
| PropertyReadBatch |Wstring，默认值为“Admin\|\|User” | 用于 Naming 属性读取操作的安全配置。 |
| GetServiceDescription |Wstring，默认值为“Admin\|\|User” | 用于长时间轮询服务通知和读取服务说明的安全配置。 |
| ResolveService |Wstring，默认值为“Admin\|\|User” | 用于基于投诉解析服务的安全配置。 |
| ResolveNameOwner |Wstring，默认值为“Admin\|\|User” | 用于解析命名 URI 所有者的安全配置。 |
| ResolvePartition |Wstring，默认值为“Admin\|\|User” | 用于解析系统服务的安全配置。 |
| ServiceNotifications |Wstring，默认值为“Admin\|\|User” | 用于基于事件的服务通知的安全配置。 |
| PrefixResolveService |Wstring，默认值为“Admin\|\|User” | 用于基于投诉解析服务前缀的安全配置。 |
| GetUpgradeStatus |Wstring，默认值为“Admin\|\|User” | 用于轮询应用程序升级状态的安全配置。 |
| GetFabricUpgradeStatus |Wstring，默认值为“Admin\|\|User” | 用于轮询群集升级状态的安全配置。 |
| InvokeInfrastructureQuery |Wstring，默认值为“Admin\|\|User” | 用于查询基础结构任务的安全配置。 |
| 列出 |Wstring，默认值为“Admin\|\|User” | 用于映像存储客户端文件列表操作的安全配置。 |
| ResetPartitionLoad |Wstring，默认值为“Admin\|\|User” | 用于重置 failoverUnit 负载的安全配置。 |
| ToggleVerboseServicePlacementHealthReporting | Wstring，默认值为“Admin\|\|User” | 用于切换详细 ServicePlacement HealthReporting 的安全配置。 |
| GetPartitionDataLossProgress | Wstring，默认值为“Admin\|\|User” | 提取调用数据丢失 API 调用的进度。 |
| GetPartitionQuorumLossProgress | Wstring，默认值为“Admin\|\|User” | 提取调用仲裁丢失 API 调用的进度。 |
| GetPartitionRestartProgress | Wstring，默认值为“Admin\|\|User” | 提取重启分区 API 调用的进度。 |
| GetChaosReport | Wstring，默认值为“Admin\|\|User” | 提取给定时间范围内的混沌状态。 |
| GetNodeTransitionProgress | Wstring，默认值为“Admin\|\|User” | 用于获取节点转换命令进度的安全配置。 |
| GetClusterConfigurationUpgradeStatus | Wstring，默认值为“Admin\|\|User” | 在分区上引入 GetClusterConfigurationUpgradeStatus。 |
| GetClusterConfiguration | Wstring，默认值为“Admin\|\|User” | 在分区上引入 GetClusterConfiguration。 |

### 节名称：ReconfigurationAgent
| **参数** | **允许的值** | **指导或简短说明** |
| --- | --- | --- |
| ApplicationUpgradeMaxReplicaCloseDuration | 以秒为单位的时间，默认值为 900 |指定以秒为单位的时间跨度。如果服务主机具有进入关闭状态的副本，则该配置确定系统在终止这类服务主机前所等待的时间。 |
| ServiceApiHealthDuration | 以秒为单位的时间，默认值为 30 分钟 | 指定以秒为单位的时间跨度。ServiceApiHealthDuration 定义在等待多少时间后 API 仍未运行的话就报告其运行不正常。 |
| ServiceReconfigurationApiHealthDuration | 以秒为单位的时间，默认值为 30 | 指定以秒为单位的时间跨度。ServiceReconfigurationApiHealthDuration 定义等待多长时间后便报告重新配置中的服务运行不正常。 |
| PeriodicApiSlowTraceInterval | 以秒为单位的时间，默认值为 5 分钟 | 指定以秒为单位的时间跨度。PeriodicApiSlowTraceInterval 定义 API 监视器追溯慢速 API 调用的时间间隔。 |
| NodeDeactivationMaxReplicaCloseDuration | 以秒为单位的时间，默认值为 900 | 指定以秒为单位的时间跨度。在终止阻止节点停用的服务主机之前需要等待的最长时间。 |
| FabricUpgradeMaxReplicaCloseDuration | 以秒为单位的时间，默认值为 900 | 指定以秒为单位的时间跨度。在终止其副本处于未关闭状态的服务主机之前，RA 需要等待的最长时间。 |
| IsDeactivationInfoEnabled | Bool，默认值为 true | 确定 RA 是否将使用停用信息执行主改选。对于新群集，应将此配置设置为 true。对于要升级的现有群集，请参阅发行说明，了解启用该配置的方法。 |

### 节名称：PlacementAndLoadBalancing
| **参数** | **允许的值** | **指导或简短说明** |
| --- | --- | --- |
| TraceCRMReasons |Bool，默认值为 true |指定是否要寻找向操作事件通道移动（CRM 发出的移动）的原因。 |
| ValidatePlacementConstraint | Bool，默认值为 true | 指定更新服务的 ServiceDescription 时，是否验证服务的 PlacementConstraint 表达式。 |
| PlacementConstraintValidationCacheSize | Int，默认值为 10000 | 限制用于快速验证和缓存放置约束表达式的表的大小。 |
|VerboseHealthReportLimit | Int，默认值为 20 | 定义副本进入未放置状态的次数超过多少次后，便报告副本运行状况警告（如果已启用详细运行状况报告）。 |
|ConstraintViolationHealthReportLimit | Int，默认值为 50 | 定义在进行诊断并发出运行状况报告之前，违反约束的副本持续处于未解决状态的次数。 |
|DetailedConstraintViolationHealthReportLimit | Int，默认值为 200 | 定义在进行诊断并发出详细运行状况报告之前，违反约束的副本持续未固定的次数。 |
|DetailedVerboseHealthReportLimit | Int，默认值为 200 | 定义在发出详细运行状况报告之前，未放置的副本必须持续处于未放置状态的次数。 |
|ConsecutiveDroppedMovementsHealthReportLimit | Int，默认值为 20 | 定义在进行诊断并发出运行状况警告之前，删除 ResourceBalancer 发出的移动的连续次数。负值：在此情况下不发出警告。 |
|DetailedNodeListLimit | Int，默认值为 15 | 定义在未放置副本报告中每个约束在截断前要包含的节点数。 |
|DetailedPartitionListLimit | Int，默认值为 15 | 定义诊断中一个约束在截断前要包含的分区数（按诊断条目）。 |
|DetailedDiagnosticsInfoListLimit | Int，默认值为 15 | 定义诊断中每个约束在截断前要包含的诊断条目数（附带详细信息）。|
|PLBRefreshGap | 以秒为单位的时间，默认值为 1 | 指定以秒为单位的时间跨度。定义 PLB 再次刷新状态之前必须经过的最短时间。 |
|MinPlacementInterval | 以秒为单位的时间，默认值为 1 | 指定以秒为单位的时间跨度。定义在两个连续的放置循环之前必须经过的最短时间。 |
|MinConstraintCheckInterval | 以秒为单位的时间，默认值为 1 | 指定以秒为单位的时间跨度。定义在两个连续的约束检查循环之前必须经过的最短时间。 |
|MinLoadBalancingInterval | 以秒为单位的时间，默认值为 5 | 指定以秒为单位的时间跨度。定义在两个连续的约束检查循环之前必须经过的最短时间。 |
|BalancingDelayAfterNodeDown | 以秒为单位的时间，默认值为 120 |指定以秒为单位的时间跨度。发生节点关闭事件后，不在此时间段内启动均衡活动。 |
|BalancingDelayAfterNewNode | 以秒为单位的时间，默认值为 120 |指定以秒为单位的时间跨度。添加新节点后，不在此时间段内启动均衡活动。 |
|ConstraintFixPartialDelayAfterNodeDown | 以秒为单位的时间，默认值为 120 | 指定以秒为单位的时间跨度。发生节点关闭事件后，不在此时间段内修复 FaultDomain 和 UpgradeDomain 约束冲突。 |
|ConstraintFixPartialDelayAfterNewNode | 以秒为单位的时间，默认值为 120 | 指定以秒为单位的时间跨度。添加新节点后，不在此时间段内修复 FaultDomain 和 UpgradeDomain 约束冲突。 |
|GlobalMovementThrottleThreshold | Uint，默认值为 1000 | GlobalMovementThrottleCountingInterval 所指示的刚过去的时间间隔中的均衡阶段中所允许的最大移动数。 |
|GlobalMovementThrottleThresholdForPlacement | Uint，默认值为 0 | GlobalMovementThrottleCountingInterval 指示的过去的时间间隔中的放置阶段中所允许的最大移动数。0 表示没有限制。|
|GlobalMovementThrottleThresholdForBalancing | Uint，默认值为 0 | GlobalMovementThrottleCountingInterval 指示的过去的时间间隔中的均衡阶段中所允许的最大移动数。0 表示没有限制。 |
|GlobalMovementThrottleCountingInterval | 以秒为单位的时间，默认值为 600 | 指定以秒为单位的时间跨度。指示过去的用于跟踪每个域副本移动的时间间隔的长度（与 GlobalMovementThrottleThreshold 配合使用）。若要完全忽略全局限制，可以将其设置为 0。 |
|MovementPerPartitionThrottleThreshold | Uint，默认值为 50 | 如果分区副本的均衡相关移动数量在过去的、由 MovementPerPartitionThrottleCountingInterval 指示的间隔时间中已达到或超过 MovementPerFailoverUnitThrottleThreshold，则该分区不会发生均衡相关的移动。 |
|MovementPerPartitionThrottleCountingInterval | 以秒为单位的时间，默认值为 600 | 指定以秒为单位的时间跨度。指示过去的用于跟踪每个分区的副本移动的时间间隔的长度（与 MovementPerPartitionThrottleThreshold 配合使用）。 |
|PlacementSearchTimeout | 以秒为单位的时间，默认值为 0.5 | 指定以秒为单位的时间跨度。这是放置服务时，返回结果之前可搜索的最长时间。 |
|UseMoveCostReports | Bool，默认值为 false | 指示 LB 忽略评分函数的成本元素，从而可能产生大量可优化均衡放置的移动。 |
|PreventTransientOvercommit | Bool，默认值为 false | 确定 PLB 是否应该立即对将由启动的移动所释放的资源进行计数。默认情况下，PLB 可以在同一节点上发起移出和移入操作，这会造成暂时性过载。将此参数设置为 true 将防止这种过载，并将禁用按需碎片整理（也称为 placementWithMove）。 |
|InBuildThrottlingEnabled | Bool，默认值为 false | 确定是否启用内置限制。 |
|InBuildThrottlingAssociatedMetric | Wstring，默认值为 "" | 此限制的关联指标名称。 |
|InBuildThrottlingGlobalMaxValue | Int，默认值为 0 |全局范围内所允许的最大内置副本数。 |
|SwapPrimaryThrottlingEnabled | Bool，默认值为 false| 确定是否启用交换主限制。 |
|SwapPrimaryThrottlingAssociatedMetric | Wstring，默认值为 ""| 此限制的关联指标名称。 |
|SwapPrimaryThrottlingGlobalMaxValue | Int，默认值为 0 | 全局范围内所允许的最大交换主副本数。 |
|PlacementConstraintPriority | Int，默认值为 0 | 确定放置约束的优先级：0：硬；1：软；负值：忽略。 |
|PreferredLocationConstraintPriority | Int，默认值为 2| 确定首选位置约束的优先级：0：硬；1：软；2：最佳；负值：忽略 |
|CapacityConstraintPriority | Int，默认值为 0 | 确定容量约束的优先级：0：硬；1：软；负值：忽略。 |
|AffinityConstraintPriority | Int，默认值为 0 | 确定相关性约束的优先级：0：硬；1：软；负值：忽略。 |
|FaultDomainConstraintPriority | Int，默认值为 0 | 确定容错域约束的优先级：0：硬；1：软；负值：忽略。 |
|UpgradeDomainConstraintPriority | Int，默认值为 1| 确定升级域约束的优先级：0：硬；1：软；负值：忽略。 |
|ScaleoutCountConstraintPriority | Int，默认值为 0 | 确定横向扩展计数约束的优先级：0：硬；1：软；负值：忽略。 |
|ApplicationCapacityConstraintPriority | Int，默认值为 0 | 确定容量约束的优先级：0：硬；1：软；负值：忽略。 |
|MoveParentToFixAffinityViolation | Bool，默认值为 false | 该设置确定是否可通过移动父副本来修复相关性约束。|
|MoveExistingReplicaForPlacement | Bool，默认值为 true |该设置确定放置过程中是否移动现有副本。 |
|UseSeparateSecondaryLoad | Bool，默认值为 true | 该设置确定是否使用不同的辅助负载。 |
|PlaceChildWithoutParent | Bool，默认值为 true | 该设置确定如果没启用父副本，是否可以放置子服务副本。 |
|PartiallyPlaceServices | Bool，默认值为 true | 确定在给定有限的适当节点的情况下，是否“全部或完全不”放置群集中的所有服务副本。|
|InterruptBalancingForAllFailoverUnitUpdates | Bool，默认值为 false | 确定是否有任何类型的故障转移单元更新应中断快速或慢速均衡运行。如果指定为“false”，则将在 FailoverUnit 出现以下情况时中断均衡运行：被创建/删除、缺少副本、更改了主副本位置或更改了副本数量。在其他情况下不会中断均衡运行，包括 FailoverUnit 具有额外副本、更改了任何副本标志、仅更改了分区版本等任何其他情况。 |

### 节名称：Security

|**参数**|**允许的值**|**指导或简短说明**|
|-----------------------|--------------------------|--------------------------|
|ClusterProtectionLevel|None 或 EncryptAndSign| 不安全的群集为 None（默认值），安全的群集为 EncryptAndSign。 |

### 节名称：Hosting

|**参数**|**允许的值**|**指导或简短说明**|
|-----------------------|--------------------------|--------------------------|
|ServiceTypeRegistrationTimeout|以秒为单位的时间，默认值为 300| 允许在结构中注册 ServiceType 的最长时间|
|ServiceTypeDisableFailureThreshold|整数，默认值为 1| 这是失败计数的阈值，超过此值后，将通知 FailoverManager (FM) 禁用该节点上的服务类型，并尝试在另一个节点上放置对象。|
|ActivationRetryBackoffInterval|以秒为单位的时间，默认值为 5|每次激活失败的退让间隔；在每次连续激活失败后，系统将重试激活最多 MaxActivationFailureCount 次。每次尝试的重试间隔是连续激活失败与激活退让间隔的积。|
|ActivationMaxRetryInterval|以秒为单位的时间，默认值为 300| 在每次连续激活失败时，系统将重试激活最多 ActivationMaxFailureCount 次。ActivationMaxRetryInterval 指定每次激活失败之后、重试之前等待的时间间隔 |
|ActivationMaxFailureCount|整数，默认值为 10| 系统在放弃之前重试失败激活的次数 |

### 节名称：FailoverManager

|**参数**|**允许的值**|**指导或简短说明**|
|-----------------------|--------------------------|--------------------------|
|PeriodicLoadPersistInterval|以秒为单位的时间，默认值为 10| 确定 FM 检查新负载报告的频率|

### 节名称：Federation

|**参数**|**允许的值**|**指导或简短说明**|
|-----------------------|--------------------------|--------------------------|
|LeaseDuration|以秒为单位的时间，默认值为 30|节点与其邻居之间的租约持续时间。|
|LeaseDurationAcrossFaultDomain|以秒为单位的时间，默认值为 30|所有容错域中的节点与其邻居之间的租约持续时间。|

### 节名称：ClusterManager
| **参数** | **允许的值** | **指导或简短说明** |
| --- | --- | --- |
| UpgradeStatusPollInterval |以秒为单位的时间，默认值为 60 |轮询应用程序升级状态的频率。此值确定任何 GetApplicationUpgradeProgress 调用的更新速率 |
| UpgradeHealthCheckInterval |以秒为单位的时间，默认值为 60 |受监视应用程序升级期间的运行状况检查频率 |
| FabricUpgradeStatusPollInterval |以秒为单位的时间，默认值为 60 |轮询结构升级状态的频率。此值确定任何 GetFabricUpgradeProgress 调用的更新速率 |
| FabricUpgradeHealthCheckInterval |以秒为单位的时间，默认值为 60 |受监视结构升级期间的运行状况检查频率 |
|InfrastructureTaskProcessingInterval | 以秒为单位的时间，默认值为 10 |指定以秒为单位的时间跨度。基础结构任务处理状态机使用的处理时间间隔。 |
|TargetReplicaSetSize |Int，默认值为 7 |ClusterManager 的 TargetReplicaSetSize。 |
|MinReplicaSetSize |Int，默认值为 3 |ClusterManager 的 MinReplicaSetSize。 |
|ReplicaRestartWaitDuration |以秒为单位的时间，默认值为 (60.0 * 30)|指定以秒为单位的时间跨度。ClusterManager 的 ReplicaRestartWaitDuration。 |
|QuorumLossWaitDuration |以秒为单位的时间，默认值为 MaxValue | 指定以秒为单位的时间跨度。ClusterManager 的 QuorumLossWaitDuration。 |
|StandByReplicaKeepDuration | 以秒为单位的时间，默认值为 (3600.0 * 2)|指定以秒为单位的时间跨度。ClusterManager 的 StandByReplicaKeepDuration。 |
|PlacementConstraints | Wstring，默认值为 "" |ClusterManager 的 PlacementConstraints。 |
|SkipRollbackUpdateDefaultService | Bool，默认值为 false |CM 将在应用程序升级回滚过程中跳过恢复更新的默认服务。 |
|EnableDefaultServicesUpgrade | Bool，默认值为 false |在应用程序升级期间启用升级默认服务。升级后，将会覆盖默认服务说明。 |
|InfrastructureTaskHealthCheckWaitDuration |以秒为单位的时间，默认值为 0| 指定以秒为单位的时间跨度。对基础结构任务的后处理完成后，开始运行状况检查之前的等待时间量。 |
|InfrastructureTaskHealthCheckStableDuration | 以秒为单位的时间，默认值为 0| 指定以秒为单位的时间跨度。在成功完成基础结构任务的后处理之前，用于观察连续通过的运行状况检查的时间量。观察失败的运行状况检查将重置此计时器。 |
|InfrastructureTaskHealthCheckRetryTimeout | 以秒为单位的时间，默认值为 60 |指定以秒为单位的时间跨度。在对基础结构任务进行后处理时，重试失败的运行状况检查所花费的时间。观察通过的运行状况检查将重置此计时器。 |
|ImageBuilderTimeoutBuffer |以秒为单位的时间，默认值为 3 |指定以秒为单位的时间跨度。所允许的映像生成器特定超时错误用于返回到客户端的时间量。如果此缓冲区太小，客户端会先于服务器超时，并收到一个泛型超时错误。 |
|MinOperationTimeout | 以秒为单位的时间，默认值为 60 |指定以秒为单位的时间跨度。用于内部处理 ClusterManager 上的操作的最小全局超时时间。 |
|MaxOperationTimeout |以秒为单位的时间，默认值为 MaxValue | 指定以秒为单位的时间跨度。用于内部处理 ClusterManager 上的操作的最大全局超时时间。 |
|MaxTimeoutRetryBuffer | 以秒为单位的时间，默认值为 600 |指定以秒为单位的时间跨度。因超时导致内部重试时，最大操作超时时间为 <原始超时> + <MaxTimeoutRetryBuffer>。以 MinOperationTimeout 为增量添加额外超时时间。 |
|MaxCommunicationTimeout |以秒为单位的时间，默认值为 600 |指定以秒为单位的时间跨度。ClusterManager 与其他系统服务（即服务命名服务、故障转移管理器等）之间的内部通信的最大超时时间。此超时时间应小于全局 MaxOperationTimeout（因为对于每个客户端操作，系统组件之间可能有多个通信）。 |
|MaxDataMigrationTimeout |以秒为单位的时间，默认值为 600 |指定以秒为单位的时间跨度。发生结构升级后，数据迁移恢复操作的最大超时时间。 |
|MaxOperationRetryDelay |以秒为单位的时间，默认值为 5| 指定以秒为单位的时间跨度。遇到故障时，内部重试的最大延迟时间。 |
|ReplicaSetCheckTimeoutRollbackOverride |以秒为单位的时间，默认值为 1200 | 指定以秒为单位的时间跨度。如果 ReplicaSetCheckTimeout 设置为 DWORD 的最大值，则出于回滚目的，将用此配置的值对其进行重写。永远不会重写用于前滚的值。 |
|ImageBuilderJobQueueThrottle |Int，默认值为 10 |映像生成器代理作业队列对应用程序请求的线程计数限制。 |

## 后续步骤

有关群集管理的详细信息，请阅读以下文章：

[在 Azure 群集中添加、滚动更新和删除证书](/documentation/articles/service-fabric-cluster-security-update-certs-azure/)

<!---HONumber=Mooncake_0227_2017-->
<!--Update_Description: add custom fabric settings details-->