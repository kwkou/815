<properties
    pageTitle="Reliable Services 的高级用法 | Azure"
    description="了解 Service Fabric Reliable Services 的高级用法，以便在服务中提高灵活性。"
    services="Service-Fabric"
    documentationcenter=".net"
    author="vturecek"
    manager="timlt"
    editor="masnider" />
<tags
    ms.assetid="f2942871-863d-47c3-b14a-7cdad9a742c7"
    ms.service="Service-Fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="02/10/2017"
    wacn.date="03/03/2017"
    ms.author="vturecek" />  

# Reliable Services 编程模型的高级用法
Azure Service Fabric 可简化可靠的无状态服务和有状态服务的编写与管理。本指南讨论 Reliable Services 的高级用法，以便针对服务获得更多控制和灵活性。阅读本指南之前，你自己应熟悉 [Reliable Services 编程模型](/documentation/articles/service-fabric-reliable-services-introduction/)。

有状态服务和无状态服务针对用户代码有两个主要入口点：

 - `RunAsync` 是服务代码的常规用途入口点。
 - `CreateServiceReplicaListeners` 和 `CreateServiceInstanceListeners` 用于针对客户端请求打开通信侦听器。
 
对于大部分服务而言，这两个入口点已足够。在少数情况下，需要更好地控制服务的生命周期，这时可以使用其他生命周期事件。

## 无状态服务实例生命周期
无状态服务的生命周期非常简单。无状态服务只能打开、关闭或中止。无状态服务中的 `RunAsync` 会在服务实例打开时执行，在服务实例关闭或中止时取消。

虽然 `RunAsync` 应该足以应付几乎所有情况，但也可以使用无状态服务中的打开、关闭和中止事件：

- 要使用无状态服务实例时，可调用 `Task OnOpenAsync(IStatelessServicePartition, CancellationToken)` OnOpenAsync。此时可以启动扩展的服务初始化任务。

- `Task OnCloseAsync(CancellationToken)` 要正常关闭无状态服务实例时，可调用 OnCloseAsync。升级服务代码、由于负载均衡而移动服务实例或是检测到暂时性故障时，可能会出现这种情况。OnCloseAsync 可以用于安全地关闭任何资源、停止任何后台处理、完成外部状态保存或关闭现有连接。

- `void OnAbort()` 要强制关闭无状态服务实例时，可调用 OnAbort。当在节点上检测到永久性故障时，或者当 Service Fabric 由于内部错误而无法可靠地管理服务实例的生命周期时，通常会调用此方法。

## 有状态服务副本生命周期
有状态服务副本的生命周期比无状态服务实例复杂得多。除了打开、关闭和中止事件，有状态服务副本还会在其生存期内经历角色更改。有状态服务副本更改角色时，会触发 `OnChangeRoleAsync` 事件：

- `Task OnChangeRoleAsync(ReplicaRole, CancellationToken)` 有状态服务副本更改角色（例如，更改为主要副本或次要副本）时，可调用 OnChangeRoleAsync。主要副本将指定为写入状态（允许创建和写入可靠集合）。次要副本将指定为读取状态（只能从现有的可靠集合读取）。有状态服务中的大部分工作在主要副本执行。次要副本可执行只读验证、报表生成、数据挖掘或其他只读作业。

在有状态服务中，只有主要副本才对状态具有写入访问权限，因此当服务执行实际工作时通常为主要副本。只有在有状态服务副本为主要副本时，才会执行有状态服务中的 `RunAsync` 方法。主要副本的角色变成非主要副本时以及关闭和中止事件期间，会取消 `RunAsync` 方法。

使用 `OnChangeRoleAsync` 事件，可以根据副本角色执行工作以及响应角色更改。

有状态服务还提供与无状态服务相同的四个生命周期事件，并具有相同的语义和用例：

- `Task OnOpenAsync(IStatefulServicePartition, CancellationToken)`
- `Task OnCloseAsync(CancellationToken)`
- `void OnAbort()`



## 后续步骤
有关与 Service Fabric 相关的更高级的主题，请参阅以下文章：

- [配置有状态 Reliable Services](/documentation/articles/service-fabric-reliable-services-configuration/)

- [Service Fabric 运行状况简介](/documentation/articles/service-fabric-health-introduction/)

- [使用系统运行状况报告进行故障排除](/documentation/articles/service-fabric-understand-and-troubleshoot-with-system-health-reports/)

- [使用 Service Fabric 群集资源管理器配置服务](/documentation/articles/service-fabric-cluster-resource-manager-configure-services/)

<!---HONumber=Mooncake_0227_2017-->