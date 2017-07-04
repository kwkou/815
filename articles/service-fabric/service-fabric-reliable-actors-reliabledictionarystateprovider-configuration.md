<properties
    pageTitle="更改 Azure 微服务中的 ReliableDictionaryActorStateProvider 设置 | Azure"
    description="了解如何配置 ReliableDictionaryActorStateProvider 类型的 Azure Service Fabric 有状态执行组件。"
    services="Service-Fabric"
    documentationcenter=".net"
    author="sumukhs"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="79b48ffa-2474-4f1c-a857-3471f9590ded"
    ms.service="Service-Fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="02/8/2017"
    wacn.date="03/03/2017"
    ms.author="sumukhs" />  

# 配置 Reliable Actors - ReliableDictionaryActorStateProvider
通过更改 Visual Studio 包根目录下的指定执行组件的 Config 文件夹中生成的 settings.xml 文件，可以修改 ReliableDictionaryActorStateProvider 的默认配置。

Azure Service Fabric 运行时在 settings.xml 文件中查找预定义的节名称，并在创建基础运行时组件时使用这些配置值。

>[AZURE.NOTE] 请**勿**删除或修改 Visual Studio 解决方案中生成的 settings.xml 文件中的以下配置的节名称。

也有一些全局设置会影响 ReliableDictionaryActorStateProvider 的配置。

## 全局配置

全局配置在群集的群集清单中的 KtlLogger 节下面指定。它可配置共享日志位置和大小，以及记录器所使用的全局内存限制。请注意，群集清单中的更改会影响使用 ReliableDictionaryActorStateProvider 的所有服务以及可靠有状态服务。

群集清单是单个 XML 文件，可保留适用于群集中所有节点和服务的设置与配置。此文件通常称为 ClusterManifest.xml。你可以使用 Get-ServiceFabricClusterManifest powershell 命令查看群集的群集清单。

### 配置名称

|名称|计价单位|默认值|备注|
|----|----|-------------|-------|
|WriteBufferMemoryPoolMinimumInKB|千字节|8388608|以内核模式分配给记录器写入缓冲区内存池的最小 KB 数。此内存池用于在将状态信息写入磁盘之前缓存这些信息。|
|WriteBufferMemoryPoolMaximumInKB|千字节|无限制|记录器写入缓冲区内存池可以增长到的大小上限。|
|SharedLogId|GUID|""|指定用来标识默认共享日志文件的唯一 GUID，该文件用于群集中所有节点上的所有 Reliable Services（不会在其服务特定配置中指定 SharedLogId）。如果指定了 SharedLogId，还必须指定 SharedLogPath。|
|SharedLogPath|完全限定的路径名|""|指定完全限定的路径，该路径中的共享日志文件用于群集中所有节点上的所有 Reliable Services（不会在其服务特定配置中指定 SharedLogPath）。但是如果指定了 SharedLogPath，还必须指定 SharedLogId。|
|SharedLogSizeInMB|兆字节|8192|指定以静态方式分配给共享日志的磁盘空间 MB 数。此值必须为 2048 或更大。|

### 群集清单节示例

	   <Section Name="KtlLogger">
	     <Parameter Name="WriteBufferMemoryPoolMinimumInKB" Value="8192" />
	     <Parameter Name="WriteBufferMemoryPoolMaximumInKB" Value="8192" />
	     <Parameter Name="SharedLogId" Value="{7668BB54-FE9C-48ed-81AC-FF89E60ED2EF}"/>
	     <Parameter Name="SharedLogPath" Value="f:\SharedLog.Log"/>
	     <Parameter Name="SharedLogSizeInMB" Value="16383"/>
	   </Section>


### 备注
记录器具有一个从未分页的内核内存分配的内存全局池，节点上的所有 Reliable Services 都可以使用该池在将状态数据写入与 Reliable Service 副本关联的专用日志之前缓存这些数据。池大小由 WriteBufferMemoryPoolMinimumInKB 和 WriteBufferMemoryPoolMaximumInKB 设置控制。WriteBufferMemoryPoolMinimumInKB 指定此内存池的初始大小，以及内存池可以缩小到的大小下限。WriteBufferMemoryPoolMaximumInKB 是内存池可以增长到的大小上限。每个打开的 Reliable Service 副本都可能会增加内存池的大小，增加幅度从系统决定的数量到 WriteBufferMemoryPoolMaximumInKB。如果内存池的内存需求大于可用的内存，则会延迟内存请求，直到有可用的内存。因此，如果写入缓冲区内存池对特定配置而言太小，则性能可能会受到影响。

SharedLogId 和 SharedLogPath 设置始终一起使用，用于定义群集中所有节点的默认共享日志的 GUID 和位置。默认共享日志可用于不在特定服务的 settings.xml 中指定设置的所有 Reliable Services。为了获得最佳性能，共享日志文件应置于仅用于共享日志文件的磁盘上，以便减少争用。

SharedLogSizeInMB 指定要预先分配给所有节点上的默认共享日志的磁盘空间数量。若要指定 SharedLogSizeInMB，不需要指定 SharedLogId 和 SharedLogPath。

## 复制器安全配置
复制器安全配置用于保护在复制过程中使用的通信通道的安全。这意味着服务将无法看到对方的复制流量，从而确保高度可用的数据也处于安全状态。
默认情况下，空的安全配置节会影响复制安全。

### 节名称
&lt;ActorName&gt;ServiceReplicatorSecurityConfig

## 复制器配置
复制器配置用于配置通过在本地复制和保持状态，负责使执行组件状态提供程序的状态高度可靠的复制器。
默认配置由 Visual Studio 模板生成，并应已足够。本部分介绍了可用于调整复制器的其他配置。

### 节名称
&lt;ActorName&gt;ServiceReplicatorConfig

### 配置名称

|名称|计价单位|默认值|备注|
|----|----|-------------|-------|
|BatchAcknowledgementInterval|秒|0\.05|收到操作后，在向主要复制器送回确认之前，辅助复制器等待的时间段。为在此间隔内处理的操作发送的任何其他确认都作为响应发送。||
|ReplicatorEndpoint|不适用|无默认值--必选参数|主要/辅助复制器用于与副本集中其他复制器通信的 IP 地址和端口。这应该引用服务清单中的 TCP 资源终结点。若要了解有关在服务清单中定义终结点资源的详细信息，请参阅[服务清单资源](/documentation/articles/service-fabric-service-manifest-resources/)。 |
|MaxReplicationMessageSize|字节|50 MB|可以在单个消息中传输的复制数据的最大大小。|
|MaxPrimaryReplicationQueueSize|操作的数量|8192|主要队列中的操作的最大数目。主复制器接收到来自所有辅助复制器的确认之后，将释放一个操作。此值必须大于 64 和 2 的幂。|
|MaxSecondaryReplicationQueueSize|操作的数量|16384|辅助队列中的操作的最大数目。将在使操作的状态在暂留期间高度可用后释放该操作。此值必须大于 64 和 2 的幂。|
|CheckpointThresholdInMB|MB|200|创建状态检查点后的日志文件空间量。|
|MaxRecordSizeInKB|KB|1024|复制器可以在日志中写入的最大记录大小。此值必须是 4 的倍数，且大于 16。|
|OptimizeLogForLowerDiskUsage|布尔|true|为 true 时会配置日志，以便使用 NTFS 稀疏文件创建副本的专用日志文件。这会降低文件的实际磁盘空间使用率。为 false 时，会使用固定分配创建文件，这可提供最佳写入性能。|
|SharedLogId|guid|""|指定要用于标识与此副本一起使用的共享日志文件的唯一 guid。通常情况下，服务不应使用此设置。但是如果指定了 SharedLogId，还必须指定 SharedLogPath。|
|SharedLogPath|完全限定的路径名|""|指定将在其中创建此副本共享日志文件的完全限定路径。通常情况下，服务不应使用此设置。但是如果指定了 SharedLogPath，还必须指定 SharedLogId。|

## 示例配置文件


	<?xml version="1.0" encoding="utf-8"?>
	<Settings xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/2011/01/fabric">
	   <Section Name="MyActorServiceReplicatorConfig">
	      <Parameter Name="ReplicatorEndpoint" Value="MyActorServiceReplicatorEndpoint" />
	      <Parameter Name="BatchAcknowledgementInterval" Value="0.05"/>
	      <Parameter Name="CheckpointThresholdInMB" Value="180" />
	   </Section>
	   <Section Name="MyActorServiceReplicatorSecurityConfig">
	      <Parameter Name="CredentialType" Value="X509" />
	      <Parameter Name="FindType" Value="FindByThumbprint" />
	      <Parameter Name="FindValue" Value="9d c9 06 b1 69 dc 4f af fd 16 97 ac 78 1e 80 67 90 74 9d 2f" />
	      <Parameter Name="StoreLocation" Value="LocalMachine" />
	      <Parameter Name="StoreName" Value="My" />
	      <Parameter Name="ProtectionLevel" Value="EncryptAndSign" />
	      <Parameter Name="AllowedCommonNames" Value="My-Test-SAN1-Alice,My-Test-SAN1-Bob" />
	   </Section>
	</Settings>


## 备注
BatchAcknowledgementInterval 参数用于控制复制延迟。“0”值导致可能的最低延迟，但代价是牺牲吞吐量（因为必须发送和处理更多的确认消息，每个包含较少的确认）。
BatchAcknowledgementInterval 的值越大，整体复制吞吐量就越高，但代价是导致更高的操作延迟。这直接转换为事务提交的延迟。

CheckpointThresholdInMB 参数控制复制器可以用于将状态信息存储在副本的专用日志文件中的磁盘空间量。将此值提高到大于默认值可以在将副本添加到集时缩短重新配置的时间。这是因为日志中会提供更多的操作历史记录，从而发生部分状态传输。在崩溃后，这可能会延长副本恢复时间。

如果将 OptimizeForLowerDiskUsage 设置为 true，则会过度预配日志文件空间，以便活动副本可以将更多状态信息存储在其日志文件中，而非活动副本使用较少的磁盘空间。这样可以在节点上托管多个副本。如果将 OptimizeForLowerDiskUsage 设置为 false，状态信息将更快地写入日志文件。

MaxRecordSizeInKB 设置用于定义可由复制器写入日志文件的记录的最大大小。大多数情况下，默认的 1024-KB 记录大小是最佳大小。但是，如果服务使更大数据项成为状态信息的一部分，则可能需要增加此值。使 MaxRecordSizeInKB 小于 1024 几乎没什么好处，因为较小记录仅使用较小记录所需的空间。我们预期此值只在极少数情况下需要更改。

SharedLogId 和 SharedLogPath 设置始终一起使用，使服务可以使用与节点的默认共享日志不同的共享日志。为获得最佳效率，应让尽可能多的服务指定相同共享日志。共享日志文件应置于仅用于共享日志文件的磁盘上，以便减少磁头运动争用。我们预期这些值只在极少数情况下需要更改。

<!---HONumber=Mooncake_0227_2017-->