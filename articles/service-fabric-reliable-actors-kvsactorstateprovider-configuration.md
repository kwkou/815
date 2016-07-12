<properties
   pageTitle="Azure Service Fabric Reliable Actors KVSActorStateProvider 配置概述 | Azure"
   description="了解有关配置类型为 KVSActorStateProvider 的 Azure Service Fabric 有状态执行组件的信息"
   services="Service-Fabric"
   documentationCenter=".net"
   authors="sumukhs"
   manager="anuragg"
   editor=""/>

<tags
   ms.service="Service-Fabric"
   ms.date="04/12/2016"
   wacn.date="07/04/2016"/>

# 配置 Reliable Actors - KVSActorStateProvider
通过更改 Microsoft Visual Studio 程序包根目录下的指定执行组件的 Config 文件夹中生成的 settings.xml 文件，可以修改 KVSActorStateProvider 的默认配置。

Azure Service Fabric 运行时在 settings.xml 文件中查找预定义的节名称，并在创建基础运行时组件时使用这些配置值。

>[AZURE.NOTE] 请**勿**删除或修改 Visual Studio 解决方案中生成的 settings.xml 文件中的以下配置的节名称。

## 复制器安全配置
复制器安全配置用于保护在复制过程中使用的通信通道的安全。这意味着服务将无法看到对方的复制流量，从而确保高度可用的数据也处于安全状态。
默认情况下，空的安全配置节会影响复制安全。

### 节名称
&lt;ActorName&gt;ServiceReplicatorSecurityConfig

## 复制器配置
复制器配置用于配置负责使执行组件状态提供程序状态高度可靠的复制器。
默认配置由 Visual Studio 模板生成，并应已足够。本部分介绍了可用于调整复制器的其他配置。

### 节名称
&lt;ActorName&gt;ServiceReplicatorConfig

### 配置名称

|Name|计价单位|默认值|备注|
|----|----|-------------|-------|
|BatchAcknowledgementInterval|秒|0\.05|收到操作后，在向主要复制器送回确认之前，辅助复制器等待的时间段。为在此间隔内处理的操作发送的任何其他确认都作为响应发送。|
|ReplicatorEndpoint|不适用|无默认值--必选参数|主要/辅助复制器用于与副本集中其他复制器通信的 IP 地址和端口。这应该引用服务清单中的 TCP 资源终结点。有关在服务清单中定义终结点资源的详细信息，请参阅[服务清单资源](/documentation/articles/service-fabric-service-manifest-resources/)。 |
|RetryInterval|秒|5|如果复制器未收到操作的确认信息而进行重新传输之后的时间段。|
|MaxReplicationMessageSize|字节|50 MB|可以在单个消息中传输的复制数据的最大大小。|
|MaxPrimaryReplicationQueueSize|操作的数量|1024|主要队列中的操作的最大数目。主复制器接收到来自所有辅助复制器的确认之后，将释放一个操作。此值必须大于 64 和 2 的幂。|
|MaxSecondaryReplicationQueueSize|操作的数量|2048|辅助队列中的操作的最大数目。将在使操作的状态在暂留期间高度可用后释放该操作。此值必须大于 64 和 2 的幂。|

## 存储配置
存储配置用于配置本地存储，该存储用于保留正在复制的状态。
默认配置由 Visual Studio 模板生成，并应已足够。本部分介绍了可用于调整本地存储的其他配置。

### 节名称
&lt;ActorName&gt;ServiceLocalStoreConfig

### 配置名称

|Name|计价单位|默认值|备注|
|----|----|-------------|-------|
|MaxAsyncCommitDelayInMilliseconds|毫秒|200|设置持久的本地存储提交的最大批处理间隔。|
|MaxVerPages|页数|16384|本地存储数据库中的最大版本页数。它确定未完成事务的最大数目。|

## 示例配置文件

```xml
<?xml version="1.0" encoding="utf-8"?>
<Settings xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/2011/01/fabric">
   <Section Name="MyActorServiceReplicatorConfig">
      <Parameter Name="ReplicatorEndpoint" Value="MyActorServiceReplicatorEndpoint" />
      <Parameter Name="BatchAcknowledgementInterval" Value="0.05"/>
   </Section>
   <Section Name="MyActorServiceLocalStoreConfig">
      <Parameter Name="MaxVerPages" Value="8192" />
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
```
## 备注

BatchAcknowledgementInterval 参数用于控制复制延迟。“0”值导致可能的最低延迟，但代价是牺牲吞吐量（因为必须发送和处理更多的确认消息，每个包含较少的确认）。
BatchAcknowledgementInterval 的值越大，整体复制吞吐量就越高，但代价是导致更高的操作延迟。这直接转换为事务提交的延迟。
 
<!---HONumber=Mooncake_0503_2016-->