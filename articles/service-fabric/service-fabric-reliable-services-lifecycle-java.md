<properties
    pageTitle="Azure Service Fabric Reliable Services 生命周期概述 | Azure"
    description="了解 Service Fabric Reliable Services 中的不同生命周期事件"
    services="Service-Fabric"
    documentationcenter="java"
    author="PavanKunapareddyMSFT"
    manager="timlt" />
<tags
    ms.assetid=""
    ms.service="Service-Fabric"
    ms.devlang="java"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="03/09/2017"
    wacn.date="05/15/2017"
    ms.author="pakunapa;"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="076df5590f0c992c0cc44e4f6a4147c64a401654"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="reliable-services-lifecycle-overview"></a>Reliable Services 生命周期概述
> [AZURE.SELECTOR]
- [Windows 上的 C#](/documentation/articles/service-fabric-reliable-services-lifecycle/)
- [Linux 上的 Java](/documentation/articles/service-fabric-reliable-services-lifecycle-java/)

在考虑要 Reliable Services 的生命周期时，具备生命周期的基础知识这一点极为重要。 一般而言：

* 在启动期间
  * 构造服务
  * 服务可能会构造并返回零个或多个侦听器
  * 打开返回的所有侦听器，以便与服务通信
  * 调用服务的 runAsync 方法，使服务能够执行长时间运行的工作或后台工作
* 在关闭期间
  * 取消传递给 runAsync 的取消令牌，同时关闭侦听器
  * 完成该过程后，销毁服务对象本身

有相应的文档详细介绍了有关这些事件的确切顺序。 具体而言，事件的顺序可能会根据 Reliable Service 是无状态服务还是有状态服务而略有变化。 此外，对于有状态服务，必须处理主副本交换方案。 在执行此序列期间，主副本的角色将转移到另一个副本（或者转移回来），而无需关闭服务。 最后，必须考虑到错误或失败状态。

## <a name="stateless-service-startup"></a>无状态服务
启动无状态服务的生命周期非常直接了当。 下面是事件的顺序：

1. 构造服务

2. 然后，并行发生两个事件：
    - 调用 `StatelessService.createServiceInstanceListeners()`，打开返回的所有侦听器（针对每个侦听器调用 `CommunicationListener.openAsync()`）
    - 调用服务的 runAsync 方法 (`StatelessService.runAsync()`)

3. 调用服务的 onOpenAsync 方法（如果存在）（具体而言，将调用 `StatelessService.onOpenAsync()`。 这是一种不常见的重写，但这种调用是可行的）。

必须注意，用于创建和打开侦听器与 runAsync 的调用之间没有一定的顺序。 可在启动 runAsync 之前打开侦听器。 同样，在通信侦听器打开甚至构造之前，即可结束调用 runAsync。 如果需要进行任何同步，这是实现器的任务。 常见解决方法：

* 有时，在创建其他某些信息或者完成工作之前，侦听器无法正常运行。 对于无状态服务，这种工作通常在服务的构造函数中、在 `createServiceInstanceListeners()` 调用期间或者在构造侦听器本身的过程中完成。
* 有时，在侦听器打开之前，runAsync 中的代码不希望启动。 在这种情况下，必须进行更多的协调。 一种常见的解决方法是在侦听器中使用某个标志来指示工作已完成，系统会先在 runAsync 中检查该标志，然后继续执行实际工作。

## <a name="stateless-service-shutdown"></a>无状态服务关闭
关闭无状态服务，将遵循相同的模式，只不过遵循的顺序相反：

1. 并行
    - 关闭所有已打开的侦听器（针对每个侦听器调用 `CommunicationListener.closeAsync()`）
    - 取消传递给 `runAsync()` 的取消标记（检查取消标记的 `isCancelled` 属性是否返回 true，如果调用标记的 `throwIfCancellationRequested` 方法，则会返回 `CancellationException`）

2. 针对每个侦听器完成 `closeAsync()` 并且完成 `runAsync()` 后，将调用服务的 `StatelessService.onCloseAsync()` 方法（如果存在）（这也是一种不常见的重写）。

3. 完成 `StatelessService.onCloseAsync()` 后，销毁服务对象

## <a name="notes-on-service-lifecycle"></a>有关服务生命周期的说明
* `runAsync()` 方法和 `createServiceInstanceListeners` 调用都是可选的。 服务可能使用其中的一个或两个，或者都不使用。 例如，如果服务执行的所有工作都只是为了响应用户调用，则无需实现 `runAsync()`。 只需提供通信侦听器及其关联的代码。 同样，创建和返回通信侦听器的操作也是可选的，因为服务可能只需执行后台工作，在这种情况下，只需实现 `runAsync()`
* 服务成功完成 `runAsync()` 并从中返回即可。 这不被视为一种错误状态，只是表示正在完成服务的后台工作。 对于有状态 Reliable Services，如果服务已从主副本降级，然后重新升级到主副本，则会再次调用 `runAsync()`。
* 如果服务通过引发某种意外的异常而从 `runAsync()` 退出，则就属于一种失败状态，在此情况下，将关闭服务对象并报告运行状况错误。
* 虽然从这些方法返回没有时间限制，但会立即丧失写入的能力，并因此无法完成任何实际工作。 建议尽快在收到取消请求后返回。 如果服务在合理的时间内未响应这些 API 调用，Service Fabric 可能会强行终止服务。 通常，只有在应用程序升级期间或删除服务时，才发生这种情况。 此超时默认为 15 分钟。
* `onCloseAsync()` 路径中的失败会导致调用 `onAbort()`，服务可以凭借这最后一个机会，尽最大努力清理并释放它们所占用的资源。

> [AZURE.NOTE]
> Java 目前不支持有状态的 Reliable Services。
>
>

## <a name="next-steps"></a>后续步骤
* [Reliable Services 简介](/documentation/articles/service-fabric-reliable-services-introduction/)
* [Reliable Services 快速启动](/documentation/articles/service-fabric-reliable-services-quick-start/)
* [Reliable Services 高级用法](/documentation/articles/service-fabric-reliable-services-advanced-usage/)