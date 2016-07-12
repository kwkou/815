<properties
   pageTitle="Azure Service Fabric Reliable Services 配置概述 | Azure"
   description="了解如何在 Azure Service Fabric 中配置有状态 Reliable Services。"
   services="Service-Fabric"
   documentationCenter=".net"
   authors="sumukhs"
   manager="anuragg"
   editor=""/>

<tags
   ms.service="Service-Fabric"
   ms.date="03/30/2016"
   wacn.date="07/04/2016"/>

有两组配置设置可供 Reliable Services 使用。一组适用于群集中的所有 Reliable Services，而另一组特定于特定的 Reliable Services。

## 全局配置

全局可靠服务配置在群集的群集清单中的 KtlLogger 节下面指定。它可配置共享日志位置和大小，以及记录器所使用的全局内存限制。群集清单是单个 XML 文件，可保留适用于群集中所有节点和服务的设置与配置。此文件通常称为 ClusterManifest.xml。你可以使用 Get-ServiceFabricClusterManifest powershell 命令查看群集的群集清单。

### 配置名称

|Name|计价单位|默认值|备注|
|----|----|-------------|-------|
|WriteBufferMemoryPoolMinimumInKB|千字节|8388608|以内核模式分配给记录器写入缓冲区内存池的最小 KB 数。此内存池用于在将状态信息写入磁盘之前缓存这些信息。|
|WriteBufferMemoryPoolMaximumInKB|千字节|无限制|记录器写入缓冲区内存池可以增长到的大小上限。|
|SharedLogId|GUID|""|指定用来标识默认共享日志文件的唯一 GUID，该文件用于群集中所有节点上的所有 Reliable Services（不会在其服务特定配置中指定 SharedLogId）。如果指定了 SharedLogId，则也必须指定 SharedLogPath。|
|SharedLogPath|完全限定的路径名|""|指定完全限定的路径，该路径中的共享日志文件用于群集中所有节点上的所有 Reliable Services（不会在其服务特定配置中指定 SharedLogPath）。但是如果指定了 SharedLogPath，则也必须指定 SharedLogId。|
|SharedLogSizeInMB|兆字节|8192|指定以静态方式分配给共享日志的磁盘空间 MB 数。此值必须为 2048 或更大。|

### 群集清单节示例
```xml
   <Section Name="KtlLogger">
     <Parameter Name="WriteBufferMemoryPoolMinimumInKB" Value="8192" />
     <Parameter Name="WriteBufferMemoryPoolMaximumInKB" Value="8192" />
     <Parameter Name="SharedLogId" Value="{7668BB54-FE9C-48ed-81AC-FF89E60ED2EF}"/>
     <Parameter Name="SharedLogPath" Value="f:\SharedLog.Log"/>
     <Parameter Name="SharedLogSizeInMB" Value="16383"/>
   </Section>
```

### 备注
记录器具有一个从未分页的内核内存分配的内存全局池，节点上的所有 Reliable Services 都可以使用该池在将状态数据写入与可靠服务副本关联的专用日志之前缓存这些数据。池大小由 WriteBufferMemoryPoolMinimumInKB 和 WriteBufferMemoryPoolMaximumInKB 设置控制。WriteBufferMemoryPoolMinimumInKB 指定此内存池的初始大小，以及内存池可以缩小到的大小下限。WriteBufferMemoryPoolMaximumInKB 是内存池可以增长到的大小上限。每个打开的可靠服务副本都可能会增加内存池的大小，增加幅度从系统决定的数量到 WriteBufferMemoryPoolMaximumInKB。如果内存池的内存需求大于可用的内存，则会延迟内存请求，直到有可用的内存。因此，如果写入缓冲区内存池对特定配置而言太小，则性能可能会受到影响。

SharedLogId 和 SharedLogPath 设置始终一起使用，用于定义群集中所有节点的默认共享日志的 GUID 和位置。默认共享日志可用于不在特定服务 settings.xml 中指定设置的所有 Reliable Services。为了获得最佳性能，共享日志文件应置于仅用于共享日志文件的磁盘上，以便减少争用。

SharedLogSizeInMB 指定要预先分配给所有节点上的默认共享日志的磁盘空间数量。若要指定 SharedLogSizeInMB，不需要指定 SharedLogId 和 SharedLogPath。


## 服务特定配置
可以通过使用配置包（配置）或服务实现（代码）来修改有状态 Reliable Services 的默认配置。

+ **配置** — 通过为应用程序中的每个服务更改 Microsoft Visual Studio 包根目录下的 Config 文件夹中生成的 Settings.xml 文件，可以使用配置包来完成配置。
+ **代码** — 通过使用 ReliableStateManagerConfiguration 对象和相应的选项集创建 ReliableStateManager，可以使用代码来完成配置。

默认情况下，Azure Service Fabric 运行时在 Settings.xml 文件中查找预定义的节名称，并在创建基础运行时组件时使用这些配置值。

>[AZURE.NOTE] 请**勿**删除 Visual Studio 解决方案中生成的 Settings.xml 文件中的以下配置的节名称，除非你打算通过代码配置你的服务。
配置 ReliableStateManager 时，重命名配置包名称或节名称需要进行代码更改。


### 复制器安全配置
复制器安全配置用于保护在复制过程中使用的通信通道的安全。这意味着服务将无法看到对方的复制流量，从而确保高度可用的数据也处于安全状态。默认情况下，空的安全配置节会影响复制安全。

### 默认节名称
ReplicatorSecurityConfig

>[AZURE.NOTE] 若要更改此节名称，请在创建此服务的 ReliableStateManager 时，将 replicatorSecuritySectionName 参数重写为 ReliableStateManagerConfiguration 构造函数。


### 复制器配置
复制器配置用于配置通过在本地复制和保持状态，负责使有状态 Reliable Service 的状态高度可靠的复制器。
默认配置由 Visual Studio 模板生成，并应已足够。本部分介绍了可用于调整复制器的其他配置。

### 默认节名称
ReplicatorConfig

>[AZURE.NOTE] 若要更改此节名称，请在创建此服务的 ReliableStateManager 时，将 replicatorSettingsSectionName 参数重写为 ReliableStateManagerConfiguration 构造函数。


### 配置名称
|Name|计价单位|默认值|备注|
|----|----|-------------|-------|
|BatchAcknowledgementInterval|秒|0\.05|收到操作后，在向主要复制器送回确认之前，辅助复制器等待的时间段。为在此间隔内处理的操作发送的任何其他确认都作为响应发送。|
|ReplicatorEndpoint|不适用|无默认值--必选参数|主要/辅助复制器用于与副本集中其他复制器通信的 IP 地址和端口。这应该引用服务清单中的 TCP 资源终结点。若要了解有关在服务清单中定义终结点资源的详细信息，请参阅[服务清单资源](/documentation/articles/service-fabric-service-manifest-resources/)。 |
|MaxPrimaryReplicationQueueSize|操作的数量|8192|主要队列中的操作的最大数目。主复制器接收到来自所有辅助复制器的确认之后，将释放一个操作。此值必须大于 64 和 2 的幂。|
|MaxSecondaryReplicationQueueSize|操作的数量|16384|辅助队列中的操作的最大数目。将在使操作的状态在暂留期间高度可用后释放该操作。此值必须大于 64 和 2 的幂。|
|CheckpointThresholdInMB|MB|50|创建状态检查点后的日志文件空间量。|
|MaxRecordSizeInKB|KB|1024|复制器可以在日志中写入的最大记录大小。此值必须是 4 的倍数，且大于 16。|
|SharedLogId|GUID|""|指定要用于标识与此副本一起使用的共享日志文件的唯一 GUID。通常情况下，服务不应使用此设置。但是如果指定了 SharedLogId，则也必须指定 SharedLogPath。|
|SharedLogPath|完全限定的路径名|""|指定将在其中创建此副本共享日志文件的完全限定路径。通常情况下，服务不应使用此设置。但是如果指定了 SharedLogPath，则也必须指定 SharedLogId。|


### 通过代码进行配置的示例
```csharp
class Program
{
    /// <summary>
    /// This is the entry point of the service host process.
    /// </summary>
    static void Main()
    {
        ServiceRuntime.RegisterServiceAsync("HelloWorldStatefulType",
            context => new HelloWorldStateful(context, 
                new ReliableStateManager(context, 
        new ReliableStateManagerConfiguration(
                        new ReliableStateManagerReplicatorSettings()
            {
                RetryInterval = TimeSpan.FromSeconds(3)
                        }
            )))).GetAwaiter().GetResult();
    }
}    
```
```csharp
class MyStatefulService : StatefulService
{
    public MyStatefulService(StatefulServiceContext context, IReliableStateManagerReplica stateManager)
        : base(context, stateManager)
    { }
    ...
}
```


### 示例配置文件
```xml
<?xml version="1.0" encoding="utf-8"?>
<Settings xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/2011/01/fabric">
   <Section Name="ReplicatorConfig">
      <Parameter Name="ReplicatorEndpoint" Value="ReplicatorEndpoint" />
      <Parameter Name="BatchAcknowledgementInterval" Value="0.05"/>
      <Parameter Name="CheckpointThresholdInMB" Value="512" />
   </Section>
   <Section Name="ReplicatorSecurityConfig">
      <Parameter Name="CredentialType" Value="X509" />
      <Parameter Name="FindType" Value="FindByThumbprint" />
      <Parameter Name="FindValue" Value="9d c9 06 b1 69 dc 4f af fd 16 97 ac 78 1e 80 67 90 74 9d 2f" />
      <Parameter Name="StoreLocation" Value="LocalMachine" />
      <Parameter Name="StoreName" Value="My" />
      <Parameter Name="ProtectionLevel" Value="EncryptAndSign" />
      <Parameter Name="AllowedCommonNames" Value="My-Test-SAN1-Alice,My-Test-SAN1-Bob" />
   </Section>
</Settings>
```


### 备注
BatchAcknowledgementInterval 控制复制延迟。“0”值导致可能的最低延迟，但代价是牺牲吞吐量（因为必须发送和处理更多的确认消息，每个包含较少的确认）。
BatchAcknowledgementInterval 的值越大，整体复制吞吐量就越高，但代价是导致更高的操作延迟。这直接转换为事务提交的延迟。

CheckpointThresholdInMB 的值控制复制器可以用于将状态信息存储在副本的专用日志文件中的磁盘空间量。将此值提高到大于默认值可以在将副本添加到集时缩短重新配置的时间。这是因为日志中会提供更多的操作历史记录，从而发生部分状态传输。在崩溃后，这可能会延长副本恢复时间。

MaxRecordSizeInKB 设置用于定义可由复制器写入日志文件的记录的最大大小。大多数情况下，默认的 1024-KB 记录大小是最佳的。但是，如果服务使更大数据项成为状态信息的一部分，则可能需要增加此值。使 MaxRecordSizeInKB 小于 1024 几乎没什么好处，因为记录仅使用较小记录所需的空间。我们预期此值只在极少数情况下需要更改。

SharedLogId 和 SharedLogPath 设置始终一起使用，使服务可以使用与节点的默认共享日志不同的共享日志。为获得最佳效率，应让尽可能多的服务指定相同共享日志。共享日志文件应置于仅用于共享日志文件的磁盘上，以便减少磁头运动争用。我们预期此值只在极少数情况下需要更改。

## 后续步骤
 - [在 Visual Studio 中调试 Service Fabric 应用程序](/documentation/articles/service-fabric-debugging-your-application/)
 - [Reliable Services 的开发人员参考](https://msdn.microsoft.com/zh-cn/library/azure/dn706529.aspx)



<!---HONumber=Mooncake_0503_2016-->