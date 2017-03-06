<properties
    pageTitle="应用程序方案和设计 | Azure"
    description="Service Fabric 中云应用程序的类别概述。介绍使用有状态服务和无状态服务的应用程序设计。"
    services="service-fabric"
    documentationcenter=".net"
    author="msfussell"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="3a8ca6ea-b8e9-4bc3-9e20-262437d2528e"
    ms.service="service-fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="2/17/2017"
    wacn.date="03/03/2017"
    ms.author="mfussell" />  


# Service Fabric 应用程序方案
Azure Service Fabric 提供了一个可靠而灵活的平台，可用于编写和运行多种类型的商业应用程序和服务。这些应用程序和微服务可以无状态也可以有状态，它们在各虚拟机间资源平衡，可最大限度提高工作效率。通过 Service Fabric 的独特体系结构，可以在应用程序中执行准实时数据分析、内存中计算、并行事务和事件处理。可根据不断变化的资源需求轻松扩展或缩减（实为放大或缩小）应用程序。

Azure 中的 Service Fabric 平台非常适合以下类别的应用程序：

* **高度可用的服务**：Service Fabric 服务通过创建多个辅助服务副本提供快速的故障转移。节点、进程或单独的服务因硬件或其他故障而不可用时，其中一个辅助副本会提升为主副本，将服务的损失降到最低。
* **可缩放的服务**：可对单个服务进行分区，以允许在群集范围内扩大状态。此外，还可动态创建和删除各项服务。可根据资源需求简便快捷地缩放服务，从几个节点上的几个实例增加到多个节点上的数千个实例，然后再次减少。可以使用 Service Fabric 生成这些服务并管理其整个生命周期。
* **非静态数据计算**：Service Fabric 使用户能够生成数据、输入/输出并计算密集型有状态应用程序。Service Fabric 允许在应用程序中共置处理（计算）和数据。当应用程序需要访问数据时，通常会存在与外部数据缓存或存储层关联的网络延迟。使用 Service Fabric 有状态服务可消除这种延迟，从而提高读取和写入性能。例如，假设有应用程序为客户执行准实时建议选择，且要求往返时间小于 100 毫秒。标准实现模型必须从远程存储提取所需数据，相比之下，Service Fabric 服务（其中建议选择计算与数据和规则共置）的延迟和性能特征可以提供快速响应的用户体验。
* **基于会话的交互式应用程序**：在应用程序（例如在线游戏或即时消息）需要低延迟读取和写入时，Service Fabric 非常有用。通过 Service Fabric，可生成这些交互式有状态应用程序，而无需创建无状态应用所需的单独存储或缓存。（这会增加延迟时间并可能产生一致性问题）。
* **分布式图形处理**：社交网络的增长大大增加了并行分析大型图形的需求。快速缩放和并行加载处理使 Service Fabric 成为用于处理大型图形的适宜平台。通过 Service Fabric，可以针对各个群组（如社交网络、商业智能和科学研究）生成高度可缩放的服务。
* **数据分析和工作流**：Service Fabric 的快速读取和写入使应用程序能够可靠处理事件或数据流。Service Fabric 还可让应用程序描述处理管道，其中的结果必须能够可靠地传递到下一个处理阶段而不会丢失。这包括交易和财务系统，其中的数据一致性和计算保证至关重要。
* **数据收集、处理和 IoT**：由于 Service Fabric 处理大规模数据并通过其有状态服务实现低延迟，因此它非常适合于设备的数据和计算共存的数百万台设备上的数据处理。我们已看到多个客户使用 Service Fabric 生成 IoT 系统，包括 [BMW](https://blogs.msdn.microsoft.com/azureservicefabric/2016/08/24/service-fabric-customer-profile-bmw-technology-corporation/)、[Schneider Electric](https://blogs.msdn.microsoft.com/azureservicefabric/2016/08/05/service-fabric-customer-profile-schneider-electric/) 和 [Mesh Systems](https://blogs.msdn.microsoft.com/azureservicefabric/2016/06/20/service-fabric-customer-profile-mesh-systems/)。

## 应用程序设计案例研究
大量说明如何使用 Service Fabric 设计应用程序的案例研究已发布到 [Service Fabric 团队博客](https://blogs.msdn.microsoft.com/azureservicefabric/tag/customer-profile/)和[微服务解决方案站点](https://azure.microsoft.com/solutions/microservice-applications/)

## 设计由无状态和有状态微服务组成的应用程序
具有 Azure 云服务辅助角色的生成应用程序是无状态服务的一个示例。相比之下，有状态微服务不仅维护请求及其响应，还维护其权威状态。在副本支持下提供交易保证的简单 API，从而提供状态的高可用性和一致性。Service Fabric 的有状态服务使高可用性变得大众化，将其引入所有类型的应用程序，而不仅仅是数据库和其他数据存储。这是一种自然的进步。为实现高可用性，应用程序已从使用单纯的关系数据库发展到 NoSQL 数据库。现在，应用程序可在自身内部管理其“热”状态和数据，以便进一步提高性能，而无需损失可靠性、一致性或可用性。

生成包含微服务的应用程序时，通常有一个无状态 Web 应用（ASP.NET 和 Node.js 等）组合调用应用于无状态和有状态业务中间层服务，这些服务使用 Service Fabric 部署命令部署到同一 Service Fabric 群集中。在规模、可靠性和资源使用情况方面，其中每项服务都是独立的，因而大大提高了开发和生命周期管理的灵活性。

有状态微服务将简化应用程序设计，因为它们不再需要传统上需要用来处理纯无状态应用程序的可用性和延迟需求的附加队列和缓存。由于有状态服务原本就具有高可用性和低延迟，这意味着在应用程序中要作为一个整体进行管理的移动部件更少。下图说明了设计有状态应用程序与无状态应用程序之间的差异。通过利用 [Reliable Services](/documentation/articles/service-fabric-reliable-services-introduction/) 和 [Reliable Actors](/documentation/articles/service-fabric-reliable-actors-introduction/) 编程模型，有状态服务降低了应用程序的复杂性，同时实现了高吞吐量和低延迟。

## 使用无状态服务生成的应用程序
![使用无状态服务的应用程序][Image1]  


## 使用有状态服务生成的应用程序
![使用无状态服务的应用程序][Image2]  


<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->
## 后续步骤
* 使用 Service Fabric [Reliable Services](/documentation/articles/service-fabric-reliable-services-quick-start/) 和 [Reliable Actors](/documentation/articles/service-fabric-reliable-actors-get-started/) 编程模型，开始生成无状态和有状态服务。
* 详细了解[模式和方案](/documentation/articles/service-fabric-patterns-and-scenarios/)。
* 了解[客户案例研究](https://mva.microsoft.com/en-us/training-courses/building-microservices-applications-on-azure-service-fabric-16747?l=qDJnf86yC_5206218965)。
* 此外，请参阅以下主题：
    * [介绍微服务](/documentation/articles/service-fabric-overview-microservices/)
    * [定义和管理服务状态](/documentation/articles/service-fabric-concepts-state/)
    * [Service Fabric 服务的可用性](/documentation/articles/service-fabric-availability-services/)
    * [缩放 Service Fabric 服务](/documentation/articles/service-fabric-concepts-scalability/)
    * [Service Fabric 服务分区](/documentation/articles/service-fabric-concepts-partitioning/)

[Image1]: ./media/service-fabric-application-scenarios/AppwithStatelessServices.jpg
[Image2]: ./media/service-fabric-application-scenarios/AppwithStatefulServices.jpg

<!---HONumber=Mooncake_0227_2017-->
<!--Update_Description: add IoT solution cases; add one case study link-->