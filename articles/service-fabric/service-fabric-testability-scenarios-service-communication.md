<properties
    pageTitle="可测试性：服务通信 | Azure"
    description="服务到服务通信是 Service Fabric 应用程序的关键集成点。本文讨论设计注意事项和测试技术。"
    services="service-fabric"
    documentationcenter=".net"
    author="vturecek"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="017557df-fb59-4e4a-a65d-2732f29255b8"
    ms.service="service-fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="01/04/2017"
    wacn.date="02/20/2017"
    ms.author="vturecek" />

# Service Fabric 可测试性方案：服务通信
Azure Service Fabric 中自然显露了微服务和面向服务的体系结构风格。在这些类型的分布式体系结构中，组件化微服务应用程序通常由需要相互通信的多个服务组成。即使在最简单的情况下，一般至少有一个无状态 Web 服务和一个有状态数据存储服务需要进行通信。

服务到服务通信是应用程序的关键集成点，因为每个服务向其他服务公开一个远程 API。使用一组涉及 I/O 的 API 边界通常需要非常谨慎，并且要进行大量的测试和验证。

这些服务边界在一个分布式系统中连接在一起时，需要考虑几个注意事项：

 - *传输协议*。使用 HTTP 协议以提高互操作性，还是使用一个自定义二进制协议以获得最大吞吐量？
 - *错误处理*。如何处理永久性错误和暂时性错误？ 服务转移到另一个节点时会发生什么情况？
 - *超时和延迟*。在多层应用程序中，每个服务层如何处理从堆栈到用户的延迟？

无论使用 Service Fabric 提供的内置服务通信组件还是构建自己的通信组件，测试服务之间的交互都是确保应用程序恢复能力的关键部分。

## 准备进行服务转移

服务实例可能随着时间的推移而四处移动。在采用加载度量值进行配置以实现量身定制的最佳资源平衡时尤其如此。即使在升级、故障转移、扩大和其他随着分布式系统的生命周期的演化而出现的各种情形下，Service Fabric 也会移动服务实例以最大可能地提高可用性。

因为服务在群集内四处移动，与服务通信时，客户端和其他服务应准备好处理两种方案：

- 从上次与之通信起，服务实例或分区副本已经移动。这是服务生命周期的正常部分，并且在应用程序的生命周期内应是预计会发生的。
- 服务实例或分区副本正在移动。尽管在 Service Fabric 中一个服务从一个节点移到另一个节点时发生故障转移的速度非常快，但如果服务的通信组件启动较慢，可用性仍然有可能出现延迟。

适当地处理这些方案对于系统的顺畅运行非常重要。若要实现此目的，请记住：

- 可以连接的每个服务都有一个它要侦听的地址（例如 HTTP 或 WebSockets）。服务实例或分区已经移动时，其地址终结点将改变。（它会移到另一个节点，并具有不同的 IP 地址。） 如果使用内置通信组件，它们会处理服务地址的重新解析。
- 服务实例再次启动其侦听程序时，服务延迟有可能暂时性增大。这取决于服务实例移动后，服务打开侦听程序的速度。
- 服务打开新的节点后，任何现有连接都需要关闭并重新打开。正常的节点关闭或重新启动会等待现有连接正常关闭。

### 测试：移动服务实例

使用 Service Fabric 的可测试性工具，我们可以编写一个测试方案，以不同的方式测试这些情形。

1. 移动有状态服务的主副本。

    移动有状态服务分区的主副本有无数原因。用此来指定某个特定分区的主副本，以查看服务如何以一种非常有控制力的方式对移动做出反应。

    

    	PS > Move-ServiceFabricPrimaryReplica -PartitionId 6faa4ffa-521a-44e9-8351-dfca0f7e0466 -ServiceName fabric:/MyApplication/MyService

    

2. 停止某个节点。

    节点停止时，Service Fabric 将该驻留在该节点上的所有服务实例或分区移到群集中的其他可用节点上。用此来测试一种情形，在这种情形中，群集丢失了一个节点，并且该节点上的所有服务实例和副本都必须移动。

    可以使用 PowerShell **Stop-ServiceFabricNode** cmdlet 来停止节点：

    

    	PS > Restart-ServiceFabricNode -NodeName Node_1

   

## 维持服务可用性

作为平台，Service Fabric 旨在为服务提供高可用性。但是，底层基础结构问题在极端情况下仍然可能导致服务不可用。因此也必须测试这些方案。

有状态服务使用基于仲裁的系统来复制状态以获得高可用性。这意味着副本的仲裁必须可用才能执行写入操作。极少数情况下，例如普遍出现的硬件故障，副本的仲裁可能不可用。在这种情况下，不能执行写入操作，但是仍然能够执行读取操作。

### 测试：写入操作不可用

使用 Service Fabric 中的可测试性工具，可以注入一个引发仲裁丢失的故障作为测试。尽管这种情况很少见，但是依赖于有状态服务的客户端和服务准备好处理无法向有状态服务进行写入请求的情形非常重要。有状态服务本身知晓这种可能性并且能够以得当的方式将其告知调用方也非常重要。

可以使用 Invoke-ServiceFabricPartitionQuorumLoss PowerShell cmdlet 引入仲裁丢失：



	PS > Invoke-ServiceFabricPartitionQuorumLoss -ServiceName fabric:/Myapplication/MyService -QuorumLossMode QuorumReplicas -QuorumLossDurationInSeconds 20



在本示例中，我们将 `QuorumLossMode` 设置为 `QuorumReplicas` 以指出我们希望引入仲裁丢失而不关闭所有副本。这样仍然能够进行读取操作。若要测试整个分区不可用的情形，可将此开关设置为 `AllReplicas`。

## 后续步骤

[了解有关可测试性操作的详细信息](/documentation/articles/service-fabric-testability-actions/)

[了解有关可测试性方案的详细信息](/documentation/articles/service-fabric-testability-scenarios/)

<!---HONumber=Mooncake_0213_2017-->
<!--Update_Description: wording update-->