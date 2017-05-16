<properties
    pageTitle="Service Fabric 可靠服务编程模型概述 | Azure"
    description="了解 Service Fabric 的 Reliable Service 编程模型，并开始编写自己的服务。"
    services="Service-Fabric"
    documentationcenter=".net"
    author="masnider"
    manager="timlt"
    editor="vturecek; mani-ramaswamy" />
<tags
    ms.assetid="0c88a533-73f8-4ae1-a939-67d17456ac06"
    ms.service="Service-Fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="04/07/2017"
    wacn.date="05/15/2017"
    ms.author="masnider;"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="14425cf6e92cc1dcf787f4267c0851198b206570"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="reliable-services-overview"></a>Reliable Services 概述
Azure Service Fabric 可简化无状态和有状态 Reliable Services 的编写与管理。 本主题的内容：

* 无状态服务和有状态服务的 Reliable Services 编程模型。
* 编写 Reliable Services 时必须做出选择。
* 有关 Reliable Services 的使用方案及编写方式的一些方案和示例。

Reliable Services 是 Service Fabric 上可用的编程模型之一。另一个是 Reliable Actor 编程模型，它在 Reliable Services 模型的顶层提供虚拟执行组件编程模型。有关 Reliable Actors 编程模型的详细信息，请参阅 [Service Fabric Reliable Actors 简介](/documentation/articles/service-fabric-reliable-actors-introduction/)。

Service Fabric 通过 [Service Fabric 应用程序管理](/documentation/articles/service-fabric-deploy-remove-applications/)管理服务的生存期，从预配和部署一直到升级和删除。

## <a name="what-are-reliable-services"></a>什么是 Reliable Services？
Reliable Services 可提供简单且功能强大的顶级编程模型，以便帮助用户表达对其应用程序至关重要的内容。 借助 Reliable Services 编程模型有以下益处：

* 访问其余的 Service Fabric 编程 API。与建模为[来宾可执行文件](/documentation/articles/service-fabric-deploy-existing-app/)的 Service Fabric Services 不同，Reliable Services 往往直接使用其余的 Service Fabric API。这样，服务便可以：
  * 查询系统
  * 报告群集中实体的运行状况
  * 接收有关配置和代码更改的通知
  * 查找其他服务并与它们通信
  * （可选）使用 [Reliable Collections](/documentation/articles/service-fabric-reliable-services-reliable-collections/)
  * ...以及访问其他许多功能，所有这些操作都可通过使用多种编程语言编写的一流编程模型执行。
* 类似于用户所习惯编程模型的简单模型，用于运行自己的代码。 代码具有定义完善的入口点和易于管理的生命周期。
* 可插式通信模型。 使用选择的传输方式，如包含 [Web API](/documentation/articles/service-fabric-reliable-services-communication-webapi/) 的 HTTP、WebSockets、自定义 TCP 协议，等等。 Reliable Services 提供一些极佳的自带选项供选用，但也可以提供自己的选项。
* 对于有状态服务，Reliable Services 编程模型允许使用 [Reliable Collections](/documentation/articles/service-fabric-reliable-services-reliable-collections/) 直接在服务内以一致、可靠的方式存储状态。 Reliable Collections 是一组简单的高度可用、可靠集合类，用过 C# 集合的用户都对它很熟悉。 按照惯例，服务需借助外部系统来进行可靠的状态管理。 利用 Reliable Collections，可将状态存储在计算旁边，获得高可用性外部存储一样的高可用性和可靠性。 此模型还能改善延迟问题，因为可将运行此模型所需的计算资源与状态放置在一起。

请观看这段 Microsoft 虚拟大学视频，大致了解 Reliable Services：
<center> 
<a target="_blank" href="https://mva.microsoft.com/training-courses/building-microservices-applications-on-azure-service-fabric-16747?l=HhD9566yC_4106218965"> 
<img src="./media/service-fabric-reliable-services-introduction/ReliableServicesVid.png" WIDTH="360" HEIGHT="244" /> 
</a> 
</center>

## <a name="what-makes-reliable-services-different"></a>Reliable Services 有何不同之处？
Service Fabric 中的 Reliable Services 与你以前编写的服务不同。 Service Fabric 提供可靠性、可用性、一致性和可伸缩性。

* **可靠性** - 即使在不可靠的环境中（计算机可能出现故障或遇到网络问题），或者在服务本身遇到错误和崩溃或故障的情况下，服务也仍能正常运行。对于有状态服务，即使遇到网络故障或其他故障，状态仍会得到保留。
* **可用性** - 服务可供访问，保持响应能力。Service Fabric 将保留所需数目的运行副本。
* **可伸缩性** - 服务与特定硬件分离，可根据需要通过添加或删除硬件或其他资源来增大或缩小。轻松将服务分区（尤其是在有状态的情况下），确保服务能够缩放，处理部分故障。可以通过代码动态创建和删除，在必要的情况下运转多个实例，例如，对客户请求做出响应。最后，Service Fabric 鼓励轻量级服务。Service Fabric 允许在单个进程内预配数千个服务，而不要求将整个 OS 实例或进程专用于服务的单个实例。
* **一致性** - 保证此服务中存储的任何信息都是一致的。即使是在服务中的多个 Reliable Collections 之间，这一点同样适用。对服务中的集合所做的更改可通过事务上不可部分完成的方式进行。

## <a name="service-lifecycle"></a>服务生命周期
无论服务有状态还是无状态，Reliable Services 都会提供简单的生命周期，以便快速插入代码并开始执行。  你只需要实现一种或两种方法，即可启动并运行服务。

* **CreateServiceReplicaListeners/CreateServiceInstanceListeners** - 服务在此方法中定义要使用的通信堆栈。通信堆栈（如 [Web API](/documentation/articles/service-fabric-reliable-services-communication-webapi/)）可定义服务的一个或多个侦听终结点（客户端将如何访问服务）。它还定义所显示的消息如何与服务代码的其余部分交互。
* **RunAsync** - 服务在此方法中运行其业务逻辑，对于在服务生存期内一直运行的所有后台任务，服务可在此方法中启动这些任务。 所提供的取消标记是指示该操作何时应停止的信号。 例如，如果服务需要从 Reliable Queue 中提取消息并进行处理，这就是这些工作的发生位置。

如果这是你第一次学习 Reliable Services，请继续阅读！ 如果你正在寻找 Reliable Services 生命周期的详细演练，请阅读[此文](/documentation/articles/service-fabric-reliable-services-lifecycle/)。

## <a name="example-services"></a>示例服务
了解这一编程模型后，让我们来快速看看两种不同的服务，了解这些部分如何搭配运作。

### <a name="stateless-reliable-services"></a>无状态的 Reliable Services
无状态服务是指每次调用后不会在服务中保留状态的服务。 现有的任何状态完全可释放，无需同步、复制、保留或高可用性。

以没有内存的计算器为例，它会接收所有项并同时执行运算。

在这种情况下，由于服务无需处理任何后台任务，因此，服务的 `RunAsync()` (C#) 或 `runAsync()` (Java) 可为空。 创建计算器服务后，该服务将返回 `ICommunicationListener` (C#) 或 `CommunicationListener`（例如 [Web API](/documentation/articles/service-fabric-reliable-services-communication-webapi/)），用于在某个端口上打开侦听终结点。 此侦听终结点挂接到不同的计算方法（例如："Add(n1, n2)"），这些方法定义计算器的公共 API。

从客户端进行调用时，将调用相应的方法，并且计算器服务会对所提供的数据执行运算并返回结果。它不存储任何状态。

不存储任何内部状态让此示例计算器变得十分简单。不过大多数服务并不是真正的无状态。它们是将状态外部化到其他某个存储。（例如，任何依赖在备份存储或缓存中保留会话状态的 Web 应用程序便不是无状态的。）

Service Fabric 中常见的无状态服务使用示例是作为前端，它公开 Web 应用程序的面向公众的 API。 然后，该前端服务指示有状态服务完成用户的请求。 在这种情况下，来自客户端的调用将定向到无状态服务正在侦听的某个已知端口（如 80）。 此无状态服务将接收调用，并判断调用是否来自可信方以及其目标服务是哪一个。  然后，此无状态服务将调用转发到有状态服务的正确分区并等待响应。 无状态服务收到响应后，将回复原始客户端。 我们的示例 [C#](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started) / [Java](https://github.com/Azure-Samples/service-fabric-java-getting-started/tree/master/Actors/VisualObjectActor/VisualObjectWebService) 中提供了此类服务的示例。 这只是此模式的示例之一，还有其他许多的示例。

### <a name="stateful-reliable-services"></a>有状态的 Reliable Services
有状态服务是指必须存在状态的某部分并且使该部分保持一致才能正常运行的服务。 假设有一个服务不断地根据某个值收到的更新，计算它的滚动平均值。 为此，它必须具有目前需要处理的传入请求集以及目前的平均值。 检索、处理并将信息存储在外部存储（比如现在的 Azure Blob 或表存储）的任何服务都是有状态的， 只不过它将其状态保存在外部状态存储中。

现在的大多数服务将其状态存储在外部，因为外部存储可为该状态提供可靠性、可用性、可伸缩性和一致性。在 Service Fabric 中，服务无需将其状态存储在外部。Service Fabric 将为服务代码和服务状态处理这些要求。

> [AZURE.NOTE]
> Linux（适用于 C# 或 Java）不提供对有状态 Reliable Services 的支持。
>

假设我们要编写一个服务来处理映像。 为此，该服务将提取一个映像，然后针对该映像执行一系列转换。 此服务将返回一个可公开 API（如 `ConvertImage(Image i, IList<Conversion> conversions)`）的通信侦听器（假设为 Web API）。 在收到请求时，服务将请求存储在 `IReliableQueue` 中，然后将某个 ID 返回给客户端，使它能够跟踪该请求。

在此服务中，`RunAsync()` 可能更复杂。服务在其 `RunAsync()` 内部使用一个循环从 `IReliableQueue` 中提取请求并执行请求的转换。结果存储在 `IReliableDictionary` 中，以便当客户端返回时可以获取其转换后的映像。为了确保即使发生故障映像也不丢失，此 Reliable Services 将从队列提取数据、执行转换，然后将整个结果存储在事务中。在此情况下，仅当转换完成时，才会从队列中删除消息并将结果存储在结果字典中。或者，服务可从队列中提取映像，并立即将其存储在远程存储中。这可以减少服务必须管理的状态数量，但会增大复杂性，因为服务必须保留必要的元数据来管理远程存储。不管使用哪种方法，如果某个环节在中途失败，请求将保留在队列中等待处理。

对于此服务，要注意的一件事就是它听起来像是普通的 .NET 服务！ 唯一的区别是其使用的数据结构（`IReliableQueue` 和 `IReliableDictionary`）由 Service Fabric 提供，因此高度可靠、可用且一致。

## <a name="when-to-use-reliable-services-apis"></a>何时使用 Reliable Services API
如果下列任何一项描述了你的应用程序服务需求，便应该考虑使用 Reliable Services API：

* 希望服务的代码（有时还包括状态）高度可用且可靠
* 需要跨多个状态单元（例如订单和订单明细）提供事务保证。
* 应用程序状态可以自然地建模为可靠字典和可靠队列。
* 应用程序代码或状态需要高度可用，读写延迟较低。
* 应用程序需要跨一个或多个可靠集合控制事务处理操作的并发或精度。
* 要为服务管理通信或控制分区方案。
* 代码需要自由线程的运行时环境。
* 应用程序需要在运行时动态创建或销毁 Reliable Dictionaries、队列或整个服务。
* 需要以编程方式为服务状态控制 Service Fabric 提供的备份和还原功能。
* 应用程序需要维护其状态单元的更改历史记录。
* 想要开发或使用第三方开发的自定义状态提供程序。

## <a name="next-steps"></a>后续步骤
* [Reliable Services 快速启动](/documentation/articles/service-fabric-reliable-services-quick-start/)
* [Reliable Services 高级用法](/documentation/articles/service-fabric-reliable-services-advanced-usage/)
* [Reliable Actors 编程模型](/documentation/articles/service-fabric-reliable-actors-introduction/)
<!--Update_Description: add java sample link;add anchors to subtitles-->