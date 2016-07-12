<properties
   pageTitle="Reliable Actors 可重入性 | Azure"
   description="Service Fabric Reliable Actors 的可重入性简介"
   services="service-fabric"
   documentationCenter=".net"
   authors="jessebenson"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.date="03/25/2016"
   wacn.date="07/04/2016"/>


# Reliable Actors 可重入性
默认情况下，Reliable Actors 运行时允许基于逻辑调用上下文的可重入性。这使执行组件在处于相同调用上下文链中时可重入。例如，如果执行组件 A 将消息发送给执行组件 B，而后者将消息发送给执行组件 C。在处理消息的过程中，如果执行组件 C 调用执行组件 A，则允许消息可重入。属于不同调用上下文的任何其他消息会在执行组件 A 上受阻，直到它完成处理。

要禁止逻辑调用基于上下文的可重入性的执行组件可以通过使用 `ReentrantAttribute(ReentrancyMode.Disallowed)` 修饰执行组件类来禁用它。

```csharp
public enum ReentrancyMode
{
    LogicalCallContext = 1,
    Disallowed = 2
}
```

以下代码演示了将重入模式设置为 `ReentrancyMode.Disallowed` 的执行组件类。在这种情况下，如果执行组件向另一个执行组件发送可重入消息，则会引发类型为 `FabricException` 的异常。

```csharp
[Reentrant(ReentrancyMode.Disallowed)]
class MyActor : Actor, IMyActor
{
    ...
}
```

## 后续步骤
 - [执行组件诊断和性能监视](/documentation/articles/service-fabric-reliable-actors-diagnostics/)
 - [执行组件 API 参考文档](https://msdn.microsoft.com/zh-cn/library/azure/dn971626.aspx)
 - [代码示例](https://github.com/Azure/servicefabric-samples)


<!---HONumber=Mooncake_0503_2016-->