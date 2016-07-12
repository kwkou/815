<properties
   pageTitle="Reliable Services 入门 | Azure"
   description="介绍如何创建具有无状态服务和有状态服务的 Microsoft Azure Service Fabric 应用程序。"
   services="service-fabric"
   documentationCenter=".net"
   authors="vturecek"
   manager="timlt"
   editor="jessebenson"/>

<tags
   ms.service="service-fabric"
   ms.date="03/25/2016"
   wacn.date="07/04/2016"/>

# Service Fabric Reliable Services 入门

Azure Service Fabric 应用程序包含一个或多个运行你的代码的服务。本指南说明如何使用 [Reliable Services](/documentation/articles/service-fabric-reliable-services-introduction/) 同时创建无状态与有状态的 Service Fabric 应用程序。

## 创建无状态服务

无状态服务是目前在云应用程序中作为基准的服务类型。该服务之所以被视为无状态，是因为它本身不包含需要可靠存储或高度可用的数据。如果无状态服务的实例关闭，其所有内部状态都会丢失。在这种类型的服务中，必须将状态保存到外部存储（如 Azure 表或 SQL 数据库），才能实现高可用性和可靠性。

以管理员身份启动 Visual Studio 2015 RC，并新建一个名为 HelloWorld 的 Service Fabric 应用程序项目：

![使用“新建项目”对话框新建 Service Fabric 应用程序](./media/service-fabric-reliable-services-quick-start/hello-stateless-NewProject.png)

然后，创建一个名为 HelloWorldStateless 的无状态服务项目：

![在第二个对话框中，创建无状态服务项目](./media/service-fabric-reliable-services-quick-start/hello-stateless-NewProject2.png)

解决方案现在包含两个项目：

 - HelloWorld。这是包含服务的应用程序项目。它还包含应用程序清单，用于描述该应用程序以及一些帮助你部署应用程序的 PowerShell 脚本。
 - HelloWorldStateless。这是服务项目。其中包含无状态服务实现。


## 实现服务

打开服务项目中的 HelloWorldStateless.cs 文件。在 Service Fabric 中，服务可以运行任一业务逻辑。服务 API 为你的代码提供两个入口点：

 - 名为 RunAsync 的开放式入口点方法，可在其中开始执行任何工作负荷，包括长时间运行的计算工作负荷。


		protected override async Task RunAsync(CancellationToken cancellationToken)
		{
    		...
		}


 - 一个通信入口点，可在其中插入选择的通信堆栈，例如 ASP.NET Web API。这就是你可以开始接收来自用户和其他服务请求的位置。


		protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
		{
    		...
		}


在本教程中，我们将重点放在 `RunAsync()` 入口点方法上。这是你可以立即开始运行代码的位置。
项目模板包括 `RunAsync()` 的示例实现，该实现递增滚动计数。

> [AZURE.NOTE] 有关如何使用通信堆栈的详细信息，请参阅 [Service Fabric Web API 服务与 OWIN 自托管](/documentation/articles/service-fabric-reliable-services-communication-webapi/)


### RunAsync

```csharp
protected override async Task RunAsync(CancellationToken cancellationToken)
{
    // TODO: Replace the following sample code with your own logic 
    //       or remove this RunAsync override if it's not needed in your service.

    long iterations = 0;

    while (true)
    {
        cancellationToken.ThrowIfCancellationRequested();

        ServiceEventSource.Current.ServiceMessage(this, "Working-{0}", ++iterations);

        await Task.Delay(TimeSpan.FromSeconds(1), cancellationToken);
    }
}
```

当服务实例已放置并且可以执行时，平台将调用此方法。对于无状态服务，这就意味着打开服务实例。需要关闭服务实例时，将提供取消标记进行协调。在 Service Fabric 中，服务实例的此打开-关闭循环可能会在服务的整个生存期内出现多次。发生这种情况的原因多种多样，包括：

- 系统可能会移动服务实例以实现资源平衡。
- 代码中发生错误。
- 应用程序或系统升级。
- 基础硬件遇到中断。

系统将管理此业务流程，以便保持服务的高度可用和适当平衡。

`RunAsync()` 在其自身的任务中执行。请注意，在上述的代码段中，我们直接跳到了 while 循环。不需要为工作负荷计划独立的任务。取消工作负荷是一项由所提供的取消标记协调的协同操作。系统会等你的任务结束后（成功完成、取消或出现故障）再执行下一步操作。当系统请求取消时，请务必接受取消标记，完成所有任务，然后尽快退出 `RunAsync()`。

在此无状态服务示例中，计数存储在本地变量中。不过，由于这是无状态服务，因此，所存储的值仅在其所在服务实例的当前生命周期中存在。当服务移动或重新启动时，值就会丢失。

## 创建有状态服务

Service Fabric 引入了一种新的有状态服务。有状态服务能够可靠地在服务本身内部保持状态，并与使用它的代码共置。Service Fabric 无需将状态保存到外部存储，便可实现状态的高可用性。

若要将计数器值从无状态转换为即使在服务移动或重新启动时仍高度可用并持久存在，你需要有状态服务。

在同一个 HelloWorld 应用程序中，通过右键单击应用程序项目中的服务引用并选择“添加”->“新建 Service Fabric 服务”，可以添加一个新的服务。

![向 Service Fabric 应用程序添加服务](./media/service-fabric-reliable-services-quick-start/hello-stateful-NewService.png)

选择“有状态服务”并将其命名为 HelloWorldStateful。单击**“确定”**。

![使用“新建项目”对话框新建 Service Fabric 有状态服务](./media/service-fabric-reliable-services-quick-start/hello-stateful-NewProject.png)

你的应用程序现在应有两个服务：无状态服务 HelloWorldStateless 和有状态服务 HelloWorldStateful。

有状态服务具有与无状态服务相同的入口点。主要差异在于可以可靠地存储状态的状态提供程序的可用性。Service Fabric 附带一个称为[可靠集合](/documentation/articles/service-fabric-reliable-services-reliable-collections/)的状态提供程序实现，它可让你通过可靠状态管理器创建复制的数据结构。有状态可靠服务默认使用此状态提供程序。

打开 HelloWorldStateful 中的 **HelloWorldStateful.cs**，该文件包含以下 RunAsync 方法：

```csharp
protected override async Task RunAsync(CancellationToken cancellationToken)
{
    // TODO: Replace the following sample code with your own logic 
    //       or remove this RunAsync override if it's not needed in your service.

    var myDictionary = await this.StateManager.GetOrAddAsync<IReliableDictionary<string, long>>("myDictionary");

    while (true)
    {
        cancellationToken.ThrowIfCancellationRequested();

        using (var tx = this.StateManager.CreateTransaction())
        {
            var result = await myDictionary.TryGetValueAsync(tx, "Counter");

            ServiceEventSource.Current.ServiceMessage(this, "Current Counter Value: {0}",
                result.HasValue ? result.Value.ToString() : "Value does not exist.");

            await myDictionary.AddOrUpdateAsync(tx, "Counter", 0, (key, value) => ++value);

            // If an exception is thrown before calling CommitAsync, the transaction aborts, all changes are 
            // discarded, and nothing is saved to the secondary replicas.
            await tx.CommitAsync();
        }

        await Task.Delay(TimeSpan.FromSeconds(1), cancellationToken);
    }
```

### RunAsync

`RunAsync()` 在有状态服务和无状态服务中的运行方式类似。只不过在有状态服务中，平台将先代表你执行额外的工作，然后再执行 `RunAsync()`。这项工作可能包括确保可靠状态管理器和可靠集合随时可供使用。

### 可靠集合与可靠状态管理器

```csharp
var myDictionary = await this.StateManager.GetOrAddAsync<IReliableDictionary<string, long>>("myDictionary");
```

IReliableDictionary 是一种字典实现，可用于将状态可靠地存储在服务中。利用 Service Fabric 和可靠集合，你可以将数据直接存储在服务中而无需外部持久性存储。可靠集合可让你的数据具备高可用性。Service Fabric 通过为你创建和管理服务的多个副本来实现此目的。它还提供一个抽象 API，消除了管理这些副本及其状态转换所存在的复杂性。

可靠集合可以存储任何 .NET 类型（包括自定义类型），但需要注意以下几点：

 - Service Fabric 通过跨节点复制状态，使状态具备高可用性；而可靠集合会将你的数据存储到每个副本上的本地磁盘中。这意味着可靠集合中存储的所有内容都必须可序列化。默认情况下，可靠集合使用 [DataContract](https://msdn.microsoft.com/zh-cn/library/system.runtime.serialization.datacontractattribute%28v=vs.110%29.aspx) 进行序列化，因此，在使用默认序列化程序时，务必确保你的类型[受数据协定序列化程序的支持](https://msdn.microsoft.com/zh-cn/library/ms731923%28v=vs.110%29.aspx)。

 - 当你在可靠集合上提交事务时，将复制对象以实现高可用性。存储在可靠集合中的对象保留在服务的本地内存中。这意味着你有对象的本地引用。

    切勿转变这些对象的本地实例而不在事务中的可靠集合上执行更新操作。这是因为对对象的本地实例的更改将不会自动复制。你必须将对象重新插回字典中，或在字典上使用其中一个更新方法。

可靠状态管理器为你管理可靠集合。无论何时何地，都可以根据名称向可靠状态管理器请求服务中的某个可靠集合。可靠状态管理器可确保你能取回引用。不建议将可靠集合实例的引用存储在类成员变量或属性中。请特别小心，确保在服务生命周期中随时将引用设置为某个实例。可靠状态管理器将为你处理此工作，并已针对重复访问进行优化。

### 事务和异步操作

```C#
using (ITransaction tx = this.StateManager.CreateTransaction())
{
    var result = await myDictionary.TryGetValueAsync(tx, "Counter-1");

    await myDictionary.AddOrUpdateAsync(tx, "Counter-1", 0, (k, v) => ++v);

    await tx.CommitAsync();
}
```

可靠集合具有许多与其 `System.Collections.Generic` 和 `System.Collections.Concurrent` 对应项相同的操作，LINQ 除外。可靠集合上的操作是异步的。这是因为可靠集合的写入操作执行 I/O 操作，以将数据复制并保存到磁盘。

可靠集合操作是事务性的，因此你可以跨多个可靠集合和操作保持状态的一致。例如，你可以在单个事务中，将工作项从 Reliable Queue 取消排队、对其执行操作并将结果保存在 Reliable Dictionary 中。这被视为原子操作，它可以保证整个操作要么成功，要么回滚。如果将项取消排队之后、保存结果之前发生错误，则会回滚整个事务，并且项将保留在队列中以供处理。

## 运行应用程序

现在，我们返回到 HelloWorld 应用程序。现在，你可以生成并部署你的服务。按 **F5**，即可生成应用程序并部署到本地群集。

服务开始运行之后，可以在“诊断事件”窗口中查看生成的 Windows 事件跟踪 (ETW) 事件。请注意，应用程序中会同时显示无状态服务和有状态服务的事件。可以通过单击“暂停”按钮来暂停流。然后，可以通过展开该消息来检查消息的详细信息。

>[AZURE.NOTE] 在运行应用程序之前，请确保正在运行本地开发群集。有关设置本地环境的信息，请查看[入门指南](/documentation/articles/service-fabric-get-started/)。

![在 Visual Studio 中查看诊断事件](./media/service-fabric-reliable-services-quick-start/hello-stateful-Output.png)


## 后续步骤

[在 Visual Studio 中调试 Service Fabric 应用程序](/documentation/articles/service-fabric-debugging-your-application/)

[入门：Service Fabric Web API 服务与 OWIN 自托管 | Azure](/documentation/articles/service-fabric-reliable-services-communication-webapi/)

[深入了解 Reliable Collections](/documentation/articles/service-fabric-reliable-services-reliable-collections/)

[部署应用程序](/documentation/articles/service-fabric-deploy-remove-applications/)

[应用程序升级](/documentation/articles/service-fabric-application-upgrade/)

[Reliable Services 的开发人员参考](https://msdn.microsoft.com/zh-cn/library/azure/dn706529.aspx)

<!---HONumber=Mooncake_0503_2016-->