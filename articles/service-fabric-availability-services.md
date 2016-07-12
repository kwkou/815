<properties
   pageTitle="Service Fabric 服务的可用性 | Microsoft Azure"
   description="介绍服务的故障检测、故障转移和恢复"
   services="service-fabric"
   documentationCenter=".net"
   authors="appi101"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.date="01/20/2016"
   wacn.date="07/04/2016"/>

# Service Fabric 服务的可用性
Azure Service Fabric 服务可以是有状态服务，也可以是无状态服务。本文概述发生故障时 Service Fabric 服务如何保持服务的可用性。

## Service Fabric 无状态服务的可用性
无状态服务是不具备任何[本地持久状态](/documentation/articles/service-fabric-concepts-state/)的应用程序服务。

创建无状态服务需要定义实例计数，即应在群集中运行的无状态服务实例的数目。这是将在群集中进行实例化的应用程序逻辑的副本数。建议通过增加实例数来扩展无状态服务。

在无状态服务的任何实例上检测到故障时，都将在群集中其他符合条件的节点上创建一个新的实例。

## Service Fabric 有状态服务的可用性
有状态服务具备一些与之关联的状态。在 Service Fabric 中，有状态服务均建模为副本集。每个副本是具有状态副本的服务的代码实例。在一个副本（以下称为“主副本”）上执行读取和写入操作。因写入操作发生的状态更改将*复制*到其他多个副本（以下称为活动辅助副本）。主副本和活动辅助副本的这种组合就是服务的副本集。

只能有一个主副本为读取和写入请求提供服务，但可以有多个活动辅助副本。可以配置活动辅助副本的数目，而副本数越多，可容忍的并发软件和硬件故障数越多。

发生故障时（主副本不可用时），Service Fabric 将指定一个活动辅助副本成为新的主副本。此活动辅助副本具有最新的状态（通过*复制*）并且可以继续处理更多的读取和写入操作。

主副本和活动辅助副本的概念称为副本角色。

### 副本角色
副本的角色可用于管理受该副本所管理状态的生命周期。具有主副本角色的副本将为读取请求提供服务。通过更新状态并将此更改复制到副本集中的活动辅助副本，主副本也可以处理写入请求。活动辅助副本的角色是接收主副本已复制的状态更改并更新其状态视图。

>[AZURE.NOTE] 对开发人员而言，更高级别的编程模型（如 [Reliable Actors 框架](/documentation/articles/service-fabric-reliable-actors-introduction/)）可使副本角色的概念不再抽象化。

## 后续步骤

有关 Service Fabric 概念的详细信息，请参阅以下内容：

- [Service Fabric 服务的可伸缩性](/documentation/articles/service-fabric-concepts-scalability/)

- [Service Fabric 服务分区](/documentation/articles/service-fabric-concepts-partitioning/)

- [定义和管理状态](/documentation/articles/service-fabric-concepts-state/)
 

<!---HONumber=Mooncake_0307_2016-->