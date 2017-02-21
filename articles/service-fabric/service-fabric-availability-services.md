<properties
    pageTitle="Service Fabric 服务的可用性 | Azure"
    description="介绍服务的故障检测、故障转移和恢复"
    services="service-fabric"
    documentationcenter=".net"
    author="masnider"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="279ba4a4-f2ef-4e4e-b164-daefd10582e4"
    ms.service="service-fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="01/05/2017"
    wacn.date="02/20/2017"
    ms.author="masnider" />  


# Service Fabric 服务的可用性
本文概述 Service Fabric 如何保持服务的可用性。

## Service Fabric 无状态服务的可用性
Azure Service Fabric 服务可以是有状态服务，也可以是无状态服务。无状态服务是不具备任何[本地持久状态](/documentation/articles/service-fabric-concepts-state/)的应用程序服务。

创建无状态服务需要定义实例计数。实例计数定义应在群集中运行的无状态服务应用程序逻辑的实例数。建议通过增加实例数来扩大无状态服务。

在无状态服务的任何实例上检测到故障时，都会在群集中符合条件的节点上创建一个新的实例。

## Service Fabric 有状态服务的可用性
有状态服务具备一些与之关联的状态。在 Service Fabric 中，有状态服务均建模为副本集。每个副本是具有状态副本的服务的代码实例。在一个副本（以下称为“主要副本”）上执行读取和写入操作。因写入操作发生的状态更改将*复制*到其他多个副本（以下称为“活动次要副本”）。主要副本和活动次要副本的这种组合构成服务的副本集。

只能有一个主要副本为读取和写入请求提供服务，但可以有多个活动次要副本。可以配置活动次要副本的数目，而副本数越多，可承受的并发软件和硬件故障数越多。

如果主要副本不可用，Service Fabric 会指定一个活动次要副本成为新的主要副本。此活动次要副本具有最新的状态（通过*复制*）并且可以继续处理更多的读取和写入操作。

主要副本和活动次要副本的这一概念称为副本角色。

### 副本角色
副本的角色用于管理受该副本所管理状态的生命周期。具有主要副本角色的副本将为读取请求提供服务。主要副本还可通过更新其状态并复制更改来处理所有写入请求。这些更改会应用于副本集中的活动次要副本。活动次要副本的任务是接收主要副本已复制的状态更改并更新其状态视图。

> [AZURE.NOTE]
对开发人员而言，更高级别的编程模型（例如 [Reliable Actors 框架](/documentation/articles/service-fabric-reliable-actors-introduction/)和 [Reliable Services](/documentation/articles/service-fabric-reliable-services-introduction/)）可使副本角色的概念不再抽象化。在执行组件中，角色的概念是不必要的，而在服务中，它在必要的情况下是可见的，但已大幅简化。
>
>

## 后续步骤
有关 Service Fabric 概念的详细信息，请参阅以下文章：

- [Service Fabric 服务的可伸缩性](/documentation/articles/service-fabric-concepts-scalability/)

- [Service Fabric 服务分区](/documentation/articles/service-fabric-concepts-partitioning/)

- [定义和管理状态](/documentation/articles/service-fabric-concepts-state/)
- [Reliable Services](/documentation/articles/service-fabric-reliable-services-introduction/)
 

<!---HONumber=Mooncake_0213_2017-->
<!--Update_Description: add introduction to reliable services-->