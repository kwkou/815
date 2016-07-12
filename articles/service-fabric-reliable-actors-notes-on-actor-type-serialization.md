<properties
   pageTitle="有关执行组件类型序列化的 Reliable Actors 说明 | Azure"
   description="讨论了定义可序列化类的基本要求，这些类可用于定义 Service Fabric Reliable Actors 的状态和接口"
   services="service-fabric"
   documentationCenter=".net"
   authors="clca"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.date="03/25/2015"
   wacn.date="07/04/2016"/>

# 有关 Service Fabric Reliable Actors 类型序列化的说明


所有方法的参数、执行组件接口中每个方法返回的任务的结果类型和执行组件的状态管理器中存储的对象都必须是[数据协定可序列化](https://msdn.microsoft.com/zh-cn/library/ms731923.aspx)..这同样适用于[执行组件事件接口](/documentation/articles/service-fabric-reliable-actors-events/#actor-events)中定义的方法的参数。（执行组件事件接口方法始终返回 void。）

## 自定义数据类型

在此示例中，以下执行组件接口定义了一个返回自定义数据类型（名为 `VoicemailBox`）的方法。

```csharp
public interface IVoiceMailBoxActor : IActor
{
    Task<VoicemailBox> GetMailBoxAsync();
}
```

此接口由使用状态管理器来存储 `VoicemailBox` 对象的执行组件实现：

```csharp
[StatePersistence(StatePersistence.Persisted)]
public class VoiceMailBoxActor : Actor, IVoicemailBoxActor
{
    public Task<VoicemailBox> GetMailboxAsync()
    {
        return this.StateManager.GetStateAsync<VoicemailBox>("Mailbox");
    }
}

```

在此示例中，如果 `VoicemailBox` 对象出现以下情况，将序列化该对象：
 - 在执行组件实例和调用方之间传输该对象。
 - 该对象保存在状态管理器中，即持久保存在磁盘中，并且已复制到其他节点。
 
Reliable Actor 框架使用 DataContract 序列化。因此，自定义数据对象及其成员必须分别使用 **DataContract** 和 **DataMember** 属性进行批注

```csharp
[DataContract]
public class Voicemail
{
    [DataMember]
    public Guid Id { get; set; }

    [DataMember]
    public string Message { get; set; }

    [DataMember]
    public DateTime ReceivedAt { get; set; }
}
```

```csharp
[DataContract]
public class VoicemailBox
{
    public VoicemailBox()
    {
        this.MessageList = new List<Voicemail>();
    }

    [DataMember]
    public List<Voicemail> MessageList { get; set; }

    [DataMember]
    public string Greeting { get; set; }
}
```

## 后续步骤
 - [执行组件生命周期和垃圾回收](/documentation/articles/service-fabric-reliable-actors-lifecycle/)
 - [执行组件计时器和提醒](/documentation/articles/service-fabric-reliable-actors-timers-reminders/)
 - [执行组件事件](/documentation/articles/service-fabric-reliable-actors-events/)
 - [执行组件可重入性](/documentation/articles/service-fabric-reliable-actors-reentrancy/)
 - [执行组件多态性和面向对象的设计模式](/documentation/articles/service-fabric-reliable-actors-polymorphism/)
 - [执行组件诊断和性能监视](/documentation/articles/service-fabric-reliable-actors-diagnostics/)


<!---HONumber=Mooncake_0503_2016-->