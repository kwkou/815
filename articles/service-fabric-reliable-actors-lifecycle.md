<properties
   pageTitle="Reliable Actors 生命周期 | Azure"
   description="介绍 Service Fabric Reliable Actor 生命周期、垃圾回收和如何手动删除执行组件及其状态"
   services="service-fabric"
   documentationCenter=".net"
   authors="jessebenson"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.date="06/13/2016"
   wacn.date="07/04/2016"/>


# 执行组件生命周期、自动垃圾回收和手动删除
当第一次调用执行组件的任何方法时即可激活该执行组件。如果在可配置的一段时间内未使用执行组件，则此执行组件将停用（执行组件运行时对其进行垃圾回收）。还可以在任何时候手动删除执行组件及其状态。

## 执行组件激活

当激活了某个执行组件，将出现以下情况：

- 当调用执行组件时，如果执行组件尚未处于活动状态，则将创建新的执行组件。
- 如果执行组件为维护状态，则加载此执行组件的状态。
- 将调用 `OnActivateAsync` 方法（该方法可以在执行组件实现中被覆盖）。
- 现在该执行组件被视为处于活动状态。

## 执行组件停用

当停用了某个执行组件，将出现以下情况：

- 如果在一段时间内未使用执行组件，则会从“活动执行组件”表中将其删除。
- 将调用 `OnDeactivateAsync` 方法（该方法可以在执行组件实现中被覆盖）。这将清除此执行组件的所有计时器。

> [AZURE.TIP] Fabric 执行组件运行时发出一些[与执行组件激活和停用相关的事件](/documentation/articles/service-fabric-reliable-actors-diagnostics/#actor-activation-and-deactivation-events)。它们可用于进行诊断和性能监视。

### 执行组件垃圾回收
停用某个执行组件后，将释放对此执行组件对象的引用，并且通常使用公共语言运行时 (CLR) 垃圾回收器对其进行回收。垃圾回收只清除执行组件对象；它**不会**删除存储在执行组件的状态管理器中的状态。当下次激活此执行组件时，将创建一个新的执行组件对象，并还原其状态。

就停用和垃圾回收而言，什么样的执行组件可视为“正在使用”？

- 正在接收调用
- 正在调用的 `IRemindable.ReceiveReminderAsync` 方法（仅当执行组件使用提醒时该方法才可用）。

> [AZURE.NOTE] 如果该执行组件使用计时器并且调用了其计时器回调，则**不**将其视为“正在使用”。

在了解停用的详细信息之前，定义以下术语非常重要：

- 扫描时间间隔。这是执行组件运行时为可以停用和进行垃圾回收的执行组件扫描其活动执行组件表的时间间隔。此状态的默认值为 1 分钟。
- 空闲超时。这是在停用执行组件并对其进行垃圾回收之前，该执行组件需要保持未使用（空闲）状态的时间。此状态的默认值为 60 分钟。

通常不需要更改这些默认值。但是，如果有必要，可在注册[执行组件服务](/documentation/articles/service-fabric-reliable-actors-platform/)时通过 `ActorServiceSettings` 更改这些时间间隔。

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        ActorRuntime.RegisterActorAsync<MyActor>((context, actorType) =>
                new ActorService(context, actorType,
                    settings:
                        new ActorServiceSettings()
                        {
                            ActorGarbageCollectionSettings =
                                new ActorGarbageCollectionSettings(10, 2)
                        }))
            .GetAwaiter()
            .GetResult();
    }
}
```

对于每个活动的执行组件，执行组件运行时将持续跟踪其处于空闲状态（即未使用）的时间。执行组件运行时每 `ScanIntervalInSeconds` 检查每个执行组件，以查看是否可以对它执行垃圾回收，并且如果它已空闲 `IdleTimeoutInSeconds`，则对其予以回收。

任何时候只要使用了执行组件，其空闲时间都会重置为 0。此后，仅在此执行组件再次空闲 `IdleTimeoutInSeconds`，才会对其执行垃圾回收。如果执行执行组件接口方法或执行组件提醒回调，则想到执行组件被视为已使用。如果执行其计时器回调，则执行组件**不**被视为已使用。

下图通过单个执行组件的生命周期来说明这些概念。

![空闲时间示例][1]

此示例显示了执行组件方法调用、提醒和计时器对此执行组件的生存期的影响。值得一提的是有关示例的以下几点：

- ScanInterval 和 IdleTimeout 分别设置为 5 和 10。（此处的单位无关紧要，因为我们的目的只是为了说明这一概念。）
- 按照扫描间隔时间为 5 的定义，对要执行垃圾回收的执行组件进行的扫描发生在 T = 0、5、10、15、20、25。
- T = 4、8、12、16、20、24 时启动定期计时器，并执行其回调。它不会影响执行组件的空闲时间。
- 在 T = 7 时调用的执行组件方法将空闲时间重置为 0，并延迟此执行组件的垃圾回收。
- 执行组件提醒回调在 T = 14 时执行，并进一步延迟执行组件的垃圾回收。
- 在 T = 25 时的垃圾回收扫描期间，执行组件的空闲时间最终超过空闲超时时间 10，将对此执行组件进行垃圾回收。

当执行组件正在执行它的一个方法时，无论执行该方法用了多少时间，始终不会对此执行组件进行垃圾回收。前面曾提到，执行执行组件接口方法和提醒回调可以防止通过将执行组件的空闲时间重置为 0 进行垃圾回收。执行计时器回调不会将空闲时间重置为 0。但是，执行组件的垃圾回收会推迟到计时器回调执行完毕。

## 删除执行组件及其状态

对已停用的执行组件进行垃圾回收只会清除该执行组件对象，但是存储在执行组件的状态管理器中的数据不会被删除。重新激活执行组件后，可通过状态管理器再次使用其数据。如果执行组件将数据存储在状态管理器，并且已停用且始终不激活该执行组件，那么可能需要清理其数据。

[执行组件服务](/documentation/articles/service-fabric-reliable-actors-platform/)提供了一个函数，用于从远程调用方删除执行组件：

```csharp
ActorId actorToDelete = new ActorId(id);

IActorService myActorServiceProxy = ActorServiceProxy.Create(
    new Uri("fabric:/MyApp/MyService"), actorToDelete);
            
await myActorServiceProxy.DeleteActorAsync(actorToDelete, cancellationToken)
```

删除一个执行组件将出现以下结果，具体取决于当前执行组件是否处于活动状态：

- **活动执行组件**
 - 执行组件从活动的执行组件列表中删除并停用。
 - 将永久删除该执行组件的状态。
- **非活动执行组件**
 - 将永久删除该执行组件的状态。

请注意，执行组件无法使用其任何一个执行组件方法对其自身调用删除，因为在执行组件调用时执行删除将无法删除该执行组件，在此过程中，运行时已获取该执行组件的锁，用于强制实施单线程访问。

## 后续步骤
 - [执行组件计时器和提醒](/documentation/articles/service-fabric-reliable-actors-timers-reminders/)
 - [执行组件事件](/documentation/articles/service-fabric-reliable-actors-events/)
 - [执行组件可重入性](/documentation/articles/service-fabric-reliable-actors-reentrancy/)
 - [执行组件诊断和性能监视](/documentation/articles/service-fabric-reliable-actors-diagnostics/)
 - [执行组件 API 参考文档](https://msdn.microsoft.com/zh-cn/library/azure/dn971626.aspx)
 - [代码示例](https://github.com/Azure/servicefabric-samples)


<!--Image references-->
[1]: ./media/service-fabric-reliable-actors-lifecycle/garbage-collection.png

<!---HONumber=Mooncake_0627_2016-->