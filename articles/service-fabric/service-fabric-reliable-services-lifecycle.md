<properties
    pageTitle="Azure Service Fabric Reliable Services 生命周期概述 | Azure"
    description="了解 Service Fabric Reliable Services 中的不同生命周期事件"
    services="Service-Fabric"
    documentationcenter=".net"
    author="masnider"
    manager="timlt"
    editor="vturecek;" />
<tags
    ms.service="service-fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="01/05/2017"
    wacn.date="02/20/2017"
    ms.author="masnider;" />  


# Reliable Services 生命周期概述
在考虑要 Reliable Services 的生命周期时，具备生命周期的基础知识这一点极为重要。一般而言：

* 在启动期间
  * 构造服务
  * 服务可能会构造并返回零个或多个侦听器
  * 打开返回的所有侦听器，以便与服务通信
  * 调用服务的 RunAsync 方法，使服务能够执行长时间运行的工作或后台工作
* 在关闭期间
  * 取消传递给 RunAsync 的取消标记，同时关闭侦听器
  * 完成该过程后，销毁服务对象本身

有相应的文档详细介绍了有关这些事件的确切顺序。具体而言，事件的顺序可能会根据 Reliable Service 是无状态服务还是有状态服务而略有变化。此外，对于有状态服务，必须处理主副本交换方案。在执行此序列期间，主副本的角色将转移到另一个副本（或者转移回来），而无需关闭服务。最后，必须考虑到错误或失败状态。

## 无状态服务
启动无状态服务的生命周期非常直接了当。下面是事件的顺序：

1. 构造服务

2. 然后，并行发生两个事件：
    - 调用 `StatelessService.CreateServiceInstanceListeners()`，打开返回的所有侦听器（针对每个侦听器调用 `ICommunicationListener.OpenAsync()`）
    - 调用服务的 RunAsync 方法 \(`StatelessService.RunAsync()`\)

3. 调用服务的 OnOpenAsync 方法（如果存在）（具体而言，将调用 `StatelessService.OnOpenAsync()`。这是一种不常见的重写，但这种调用是可行的）。

必须注意，在创建和打开侦听器与 RunAsync 时执行的调用之间没有一定的顺序。可在启动 RunAsync 之前打开侦听器。同样，在通信侦听器打开甚至构造之前，即可结束调用 RunAsync。如果需要进行任何同步，这是实现器的任务。常见解决方法：

* 有时，在创建其他某些信息或者完成工作之前，侦听器无法正常运行。对于无状态服务，这种工作通常在服务的构造函数中、在 `CreateServiceInstanceListeners()` 调用期间或者在构造侦听器本身的过程中完成。
* 有时，在侦听器打开之前，RunAsync 中的代码不希望启动。在这种情况下，必须进行更多的协调。一种常见的解决方法是在侦听器中使用某个标志来指示工作已完成，系统会先在 RunAsync 中检查该标志，然后继续执行实际工作。

## 无状态服务关闭
关闭无状态服务，将遵循相同的模式，只不过遵循的顺序相反：

1. 并行
    - 关闭所有已打开的侦听器（针对每个侦听器调用 `ICommunicationListener.CloseAsync()`）
    - 取消传递给 `RunAsync()` 的取消标记（检查取消标记的 `IsCancellationRequested` 属性是否返回 true，如果调用标记的 `ThrowIfCancellationRequested` 方法，则会返回 `OperationCanceledException`）

2. 针对每个侦听器完成 `CloseAsync()` 并且完成 `RunAsync()` 后，将调用服务的 `StatelessService.OnCloseAsync()` 方法（如果存在）（这也是一种不常见的重写）。

3. 完成 `StatelessService.OnCloseAsync()` 后，销毁服务对象

## 有状态服务启动
有状态服务的模式与无状态服务类似，只是稍有不同。启动有状态服务时，事件的顺序如下所述：

1. 构造服务

2. 调用 `StatefulServiceBase.OnOpenAsync()`。（这是服务中不常见的重写。）

3. 如果有问题的服务副本是主副本，则会并行发生以下事件，否则，服务将跳到步骤 4
    - 调用 `StatefulServiceBase.CreateServiceReplicaListeners()`，打开返回的所有侦听器（针对每个侦听器调用 `ICommunicationListener.OpenAsync()`）
    - 调用服务的 RunAsync 方法 \(`StatefulServiceBase.RunAsync()`\)

4. 完成副本侦听器的所有 `OpenAsync()` 调用并且已启动 `RunAsync()` 后（或者由于此副本当前是辅助节点而跳过了这些步骤时），将调用 `StatefulServiceBase.OnChangeRoleAsync()`。（这是服务中不常见的重写。）

类似于无状态服务，创建和打开侦听器，以及调用 RunAsync 的顺序不会经过协调。解决方法大致相同，我们另举一例：假设抵达通信侦听器的调用需要在某个 [Reliable Collections](/documentation/articles/service-fabric-reliable-services-reliable-collections/) 中保存信息才能正常工作。由于通信侦听器可能在 Reliable Collections 可读或可写之前打开，因此，在 RunAsync 可以启动之前，必须经过一定的附加协调。最简单且最常见的解决方法是让通信侦听器返回某个错误代码，告知客户端重试请求。

## 有状态服务关闭
类似于无状态服务，关闭期间的生命周期事件与启动期间是相同的，但顺序相反。关闭有状态服务时，将发生以下事件：

1. 并行
    - 关闭所有已打开的侦听器（针对每个侦听器调用 `ICommunicationListener.CloseAsync()`）
    - 取消传递给 `RunAsync()` 的取消标记（检查取消标记的 `IsCancellationRequested` 属性是否返回 true，如果调用标记的 `ThrowIfCancellationRequested` 方法，则会返回 `OperationCanceledException`）

2. 针对每个侦听器完成 `CloseAsync()` 并且完成 `RunAsync()`（仅当此服务副本是主副本时，才需要完成此调用）后，将调用服务的 `StatefulServiceBase.OnChangeRoleAsync()`。（这是服务中不常见的重写。）

3. 完成 `StatefulServiceBase.OnChangeRoleAsync()` 方法后，将调用 `StatefulServiceBase.OnCloseAsync()` 方法（这也是一种不常见的重写，但这种调用是可行的）。

3. 完成 `StatefulServiceBase.OnCloseAsync()` 后，销毁服务对象。

## 有状态服务主副本交换
运行有状态服务时，只有该有状态服务的主副本打开其通信侦听器并调用其 RunAsync 方法。将会构造辅助副本，但不会对其执行进一步的调用。不过，在运行有状态服务时当前用作主副本的副本可能会更改。从副本看到的生命周期事件角度看，这意味着什么呢？ 有状态副本看到的行为取决于在交换期间该副本是已降级还是已升级。

### 对于降级的主副本
Service Fabric 需要使用此副本来停止处理消息，退出正在执行的任何后台工作。因此，此步骤类似于关闭服务时的情况。一个差别在于，在此情况下不会销毁或关闭服务，因为它保留为辅助副本。将调用以下 API：

1. 并行
    - 关闭所有已打开的侦听器（针对每个侦听器调用 `ICommunicationListener.CloseAsync()`）
    - 取消传递给 `RunAsync()` 的取消标记（检查取消标记的 `IsCancellationRequested` 属性是否返回 true，如果调用标记的 `ThrowIfCancellationRequested` 方法，则会返回 `OperationCanceledException`）

2. 针对每个侦听器完成 `CloseAsync()` 并且完成 `RunAsync()` 后，将调用服务的 `StatefulServiceBase.OnChangeRoleAsync()`。（这是服务中不常见的重写。）

### 对于升级的辅助副本
同样，Service Fabric 需要使用此副本来开始侦听网络上的消息（如果要这样做的话），启动它所关注的任何后台任务。因此，此过程类似于创建服务时的情况，只不过副本本身已存在。将调用以下 API：

1. 并行
    - 调用 `StatefulServiceBase.CreateServiceReplicaListeners()`，打开返回的所有侦听器（针对每个侦听器调用 `ICommunicationListener.OpenAsync()`）
    - 调用服务的 RunAsync 方法 \(`StatefulServiceBase.RunAsync()`\)
    
4. 完成副本侦听器的所有 `OpenAsync()` 调用并且已启动 `RunAsync()` 后（或者由于此副本是辅助节点而跳过了这些步骤时），将调用 `StatefulServiceBase.OnChangeRoleAsync()`。（这是服务中不常见的重写。）

## 有关服务生命周期的说明
* `RunAsync()` 方法和 `CreateServiceReplicaListeners/CreateServiceInstanceListeners` 调用都是可选的。服务可能使用其中的一个或两个，或者都不使用。例如，如果服务执行的所有工作都只是为了响应用户调用，则无需实现 `RunAsync()`。只需提供通信侦听器及其关联的代码。同样，创建和返回通信侦听器的操作也是可选的，因为服务可能只需执行后台工作，在这种情况下，只需实现 `RunAsync()`
* 服务成功完成 `RunAsync()` 并从中返回即可。这不被视为一种错误状态，只是表示正在完成服务的后台工作。对于有状态 Reliable Services，如果服务已从主副本降级，然后重新升级到主副本，则会再次调用 `RunAsync()`。
* 如果服务通过引发某种意外的异常而从 `RunAsync()` 退出，则就属于一种失败状态，在此情况下，将关闭服务对象并报告运行状况错误。
* 虽然从这些方法返回没有时间限制，但会立即丧失写入到可靠集合的能力，并因此无法完成任何实际工作。建议收到取消请求后尽快返回。如果服务在合理的时间内未响应这些 API 调用，Service Fabric 可能会强行终止服务。通常，只有在应用程序升级期间或删除服务时，才发生这种情况。此超时默认为 15 分钟。
* 有状态服务针对 ServiceReplicaListeners 提供了一个附加选项，使这些侦听器能够在辅助副本上启动。这种情况不太常见，但是，生命周期的唯一变化是会调用 `CreateServiceReplicaListeners()`（生成的侦听器将会打开），即使副本是辅助副本，也是如此。同样，如果随后将副本转换为主副本，则在切换到主副本的过程中，会关闭并销毁侦听器，然后创建并打开新的侦听器。
* `OnCloseAsync()` 路径中的失败会导致调用 `OnAbort()`，服务可以凭借这最后一个机会，尽最大努力清理并释放它们所占用的资源。

## 后续步骤
* [Reliable Services 简介](/documentation/articles/service-fabric-reliable-services-introduction/)
* [Reliable Services 快速入门](/documentation/articles/service-fabric-reliable-services-quick-start/)
* [Reliable Services 高级用法](/documentation/articles/service-fabric-reliable-services-advanced-usage/)

<!---HONumber=Mooncake_0213_2017-->