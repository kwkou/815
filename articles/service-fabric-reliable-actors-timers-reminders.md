<properties
   pageTitle="Reliable Actors 计时程序和提醒程序 | Azure"
   description="Service Fabric Reliable Actors 的计时器和提醒简介。"
   services="service-fabric"
   documentationCenter=".net"
   authors="jessebenson"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.date="03/25/2016"
   wacn.date="07/04/2016"/>

# 执行组件计时器和提醒
执行组件可通过注册计时器或提醒来计划自身的定期工作。本文演示如何使用计时器和提醒，并说明它们之间的差异。

## 执行组件计时器
执行组件计时器围绕 .NET 计时器提供一个简单包装器，确保回叫方法采用执行组件运行时提供的基于轮次的并发保证。

执行组件可以对其基类使用 `RegisterTimer` 和 `UnregisterTimer` 方法以注册和注销其计时器。下面的示例演示了如何使用计时器 API。这些 API 非常类似于 .NET 计时器。在本示例中，当计时器运行结束时，执行组件运行时将调用 `MoveObject` 方法。可保证该方法遵循基于轮次的并发。这意味着，任何其他执行组件方法或计时器/提醒回调将一直进行，直到此回调完成执行为止。

```csharp
class VisualObjectActor : Actor, IVisualObject
{
    private IActorTimer _updateTimer;

    protected override Task OnActivateAsync()
    {
        ...

        _updateTimer = RegisterTimer(
            MoveObject,                     // Callback method
            null,                           // Parameter to pass to the callback method
            TimeSpan.FromMilliseconds(15),  // Amount of time to delay before the callback is invoked
            TimeSpan.FromMilliseconds(15)); // Time interval between invocations of the callback method

        return base.OnActivateAsync();
    }

    protected override Task OnDeactivateAsync()
    {
        if (_updateTimer != null)
        {
            UnregisterTimer(_updateTimer);
        }

        return base.OnDeactivateAsync();
    }

    private Task MoveObject(object state)
    {
        ...
        return Task.FromResult(true);
    }
}
```

计时器的下一个周期在回调完成执行之后启动。这意味着计时器会在回调执行期间停止，在回调完成时启动。

执行组件运行时会保存在回叫完成时，对执行组件的状态管理器所做的更改。如果在保存状态时发生错误，则会停用该执行组件对象并激活一个新实例。

如果在垃圾回收过程中停用了执行组件，所有计时器将会停止。此后不会调用任何计时器回调。此外，执行组件运行时不保留有关在停用之前运行的计时器的任何信息。这主要归功于执行组件可以注册在将来重新激活时需要的任何计时器。有关详细信息，请参阅有关[执行组件垃圾回收](/documentation/articles/service-fabric-reliable-actors-lifecycle/)的部分。

## 执行组件提醒
提醒是一种机制，用于在指定时间对执行组件触发持久回调。其功能类似于计时器。但与计时器不同的是，提醒会在所有情况下触发，直到执行组件显式注销提醒或显式删除执行组件。具体而言，提醒会在执行组件停用和故障转移间触发，因为 执行组件运行时会保持有关执行组件提醒的信息。

为了注册提醒，执行组件将调用基类上提供的 `RegisterReminderAsync` 方法，如以下示例中所示：

```csharp
protected override async Task OnActivateAsync()
{
    string reminderName = "Pay cell phone bill";
    int amountInDollars = 100;

    IActorReminder reminderRegistration = await this.RegisterReminderAsync(
        reminderName,
        BitConverter.GetBytes(amountInDollars),
        TimeSpan.FromDays(3),
        TimeSpan.FromDays(1));
}
```

在本例中，`"Pay cell phone bill"` 是提醒名称。这是执行组件用来唯一标识提醒的字符串。`BitConverter.GetBytes(amountInDollars)` 是与提醒关联的上下文。它会作为提醒回调的参数（即 `IRemindable.ReceiveReminderAsync`）传递回执行组件。

使用提醒的执行组件必须实现 `IRemindable` 接口，如以下示例中所示。

```csharp
public class ToDoListActor : Actor, IToDoListActor, IRemindable
{
    public Task ReceiveReminderAsync(string reminderName, byte[] context, TimeSpan dueTime, TimeSpan period)
    {
        if (reminderName.Equals("Pay cell phone bill"))
        {
            int amountToPay = BitConverter.ToInt32(context, 0);
            System.Console.WriteLine("Please pay your cell phone bill of ${0}!", amountToPay);
        }
        return Task.FromResult(true);
    }
}
```

触发提醒时，Reliable Actors 运行时将对执行组件调用 `ReceiveReminderAsync` 方法。一个执行组件可以注册多个提醒，而 `ReceiveReminderAsync` 方法将在触发其中任一提醒时调用。执行组件可以使用传入给 `ReceiveReminderAsync` 方法的提醒名称来找出触发的提醒。

执行组件运行时将在 `ReceiveReminderAsync` 调用完成时保存执行组件状态。如果在保存状态时发生错误，则会停用该执行组件对象并激活一个新实例。要指定在提醒回调完成时无需保存状态，可以在调用 `RegisterReminder` 方法来创建提醒时在 `attributes` 参数中设置 `ActorReminderAttributes.ReadOnly` 标志。

为了注销提醒，执行组件将调用 `UnregisterReminder` 方法，如以下示例中所示。

```csharp
IActorReminder reminder = GetReminder("Pay cell phone bill");
Task reminderUnregistration = UnregisterReminder(reminder);
```

如上所示，`UnregisterReminder` 方法接受 `IActorReminder` 接口。执行组件基类支持 `GetReminder` 方法，该方法可以用于通过传入提醒名称来检索 `IActorReminder` 接口。这十分方便，因为参与者无需保存从 `RegisterReminder` 方法调用返回的 `IActorReminder` 接口。

## 后续步骤
 - [执行组件事件](/documentation/articles/service-fabric-reliable-actors-events/)
 - [执行组件可重入性](/documentation/articles/service-fabric-reliable-actors-reentrancy/)
 - [执行组件诊断和性能监视](/documentation/articles/service-fabric-reliable-actors-diagnostics/)
 - [执行组件 API 参考文档](https://msdn.microsoft.com/zh-cn/library/azure/dn971626.aspx)
 - [代码示例](https://github.com/Azure/servicefabric-samples)

<!---HONumber=Mooncake_0503_2016-->