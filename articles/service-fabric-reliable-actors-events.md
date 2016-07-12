<properties
   pageTitle="Reliable Actors 事件 | Azure"
   description="Service Fabric Reliable Actors 的事件简介。"
   services="service-fabric"
   documentationCenter=".net"
   authors="jessebenson"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.date="06/13/2016"
   wacn.date="07/04/2016"/>


# 执行组件事件
执行组件事件提供了一种尽最大努力将通知从执行组件发送到客户端的方法。执行组件事件设计用于从执行组件到客户端的通信，而不应用于从执行组件到执行组件的通信。

以下代码段演示如何在你的应用程序中使用执行组件事件。

定义说明由执行组件发布的事件的接口。此接口必须派生自 `IActorEvents` 接口。方法的参数必须为[数据协定可序列化](/documentation/articles/service-fabric-reliable-actors-notes-on-actor-type-serialization/)。当事件通知是单向且为最佳效果时，方法必须返回 void。

```csharp
public interface IGameEvents : IActorEvents
{
    void GameScoreUpdated(Guid gameId, string currentScore);
}
```

声明由执行组件在执行组件界面中发布的事件。

```csharp
public interface IGameActor : IActor, IActorEventPublisher<IGameEvents>
{
    Task UpdateGameStatus(GameStatus status);

    Task<string> GetGameScore();
}
```

在客户端上，实现事件处理程序。

```csharp
class GameEventsHandler : IGameEvents
{
    public void GameScoreUpdated(Guid gameId, string currentScore)
    {
        Console.WriteLine(@"Updates: Game: {0}, Score: {1}", gameId, currentScore);
    }
}
```

在客户端上，为发布事件并订阅其事件的执行组件创建代理。

```csharp
var proxy = ActorProxy.Create<IGameActor>(
                    new ActorId(Guid.Parse(arg)), ApplicationName);

await proxy.SubscribeAsync<IGameEvents>(new GameEventsHandler());
```

如果发生故障转移，执行组件可能会故障转移到不同的进程或节点。执行组件代理管理活动的订阅，并自动重新订阅它们。你可以通过 `ActorProxyEventExtensions.SubscribeAsync<TEvent>` API 控制重新订阅的间隔。若要取消订阅，请使用 `ActorProxyEventExtensions.UnsubscribeAsync<TEvent>` API。

在执行组件上，只需在事件发生时发布事件。如果有订阅此事件的用户，那么执行组件运行时会向他们发送通知。

```csharp
var ev = GetEvent<IGameEvents>();
ev.GameScoreUpdated(Id.GetGuidId(), score);
```

## 后续步骤
 - [执行组件可重入性](/documentation/articles/service-fabric-reliable-actors-reentrancy/)
 - [执行组件诊断和性能监视](/documentation/articles/service-fabric-reliable-actors-diagnostics/)
 - [执行组件 API 参考文档](https://msdn.microsoft.com/zh-cn/library/azure/dn971626.aspx)
 - [代码示例](https://github.com/Azure/servicefabric-samples)

<!---HONumber=Mooncake_0627_2016-->