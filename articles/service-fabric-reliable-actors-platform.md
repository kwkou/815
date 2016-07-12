<properties
   pageTitle="Service Fabric 上的 Reliable Actors | Azure"
   description="介绍 Reliable Actors 如何在 Reliable Services 上进行分层以及如何使用 Service Fabric 平台的功能。"
   services="service-fabric"
   documentationCenter=".net"
   authors="jessebenson"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.date="03/25/2016"
   wacn.date="07/04/2016"/>

# Reliable Actors 如何使用 Service Fabric 平台

本文介绍了 Reliable Actors 如何使用 Service Fabric 平台。Reliable Actors 在有状态的 Reliable Service（称为执行组件服务）的实现托管的框架中运行。执行组件服务包含管理执行组件的生命周期和消息发送所需的所有组件。

 - 执行组件运行时管理生命周期、垃圾回收，并强制执行单线程访问。
 - 执行组件服务远程处理侦听器接受对执行组件的远程访问调用，并将它们发送到调度程序，以便路由到合适的执行组件实例。
 - 执行组件状态提供程序包装状态提供程序（例如 Reliable Collections 状态提供程序），并为执行组件状态管理提供一个适配器。

这些组件共同构成了 Reliable Actor 框架。

## 服务分层

因为执行组件服务本身是一种 Reliable Service，Reliable Services 的所有[应用程序模型](/documentation/articles/service-fabric-application-model/)、生命周期、[打包](/documentation/articles/service-fabric-application-model/#package-an-application)、[部署](/documentation/articles/service-fabric-deploy-remove-applications/#deploy-an-application)、升级和伸缩概念同样适用于执行组件服务。

![执行组件服务分层][1]

上图显示了 Service Fabric 应用程序框架和用户代码之间的关系。蓝色元素代表 Reliable Services 应用程序框架，橙色代表 Reliable Actor 框架，绿色则代表用户代码。


在 Reliable Services 中，你的服务会继承 `StatefulService` 类，该类本身派生自 `StatefulServiceBase`（或无状态服务的 `StatelessService`）。在 Reliable Actors 中，你所使用的执行组件服务是一种不同的 `StatefulServiceBase` 类的实现，该类可实现执行执行组件的模式。由于执行组件服务只是 `StatefulServiceBase` 的一个实现，因此你可以编写自己的派生自 `ActorService` 的服务，并且以与继承 `StatefulService` 时一样的方式实现服务级别的功能，例如：

 - 服务备份和还原。
 - 共享给所有执行组件的功能，例如断路器。
 - 对执行组件服务自身以及每个执行组件的远程过程调用。 

### 使用执行组件服务

执行组件实例可访问执行这些实例的执行组件服务。通过执行组件服务，执行组件实例可以编程方式获取服务上下文，其中包括分区 ID、服务名称、应用程序名称和其他特定于 Service Fabric 平台的信息。

```csharp
Task MyActorMethod()
{
    Guid partitionId = this.ActorService.Context.PartitionId;
    string serviceTypeName = this.ActorService.Context.ServiceTypeName;
    Uri serviceInstanceName = this.ActorService.Context.ServiceName;
    string applicationInstanceName = this.ActorService.Context.CodePackageActivationContext.ApplicationName;
}
```

与所有 Reliable Services 一样，执行组件服务必须使用 Service Fabric 运行时中的服务类型注册。为了使执行组件服务能够运行执行组件实例，还必须向执行组件服务注册你的执行组件类型。`ActorRuntime` 注册方法将为执行组件执行此操作。最简单的情况是，你只需注册执行组件类型，然后隐式使用具有默认设置的执行组件服务：

```csharp
static class Program
{
    private static void Main()
    {
        ActorRuntime.RegisterActorAsync<MyActor>().GetAwaiter().GetResult();

        Thread.Sleep(Timeout.Infinite);
    }
}
```  

或者，可以使用此注册方法提供的 lambda 自己构造执行组件服务。这种方法允许你配置执行组件服务和显式构造你的执行组件实例，你可以从中通过执行组件的构造函数向执行组件注入依赖项：

```csharp
static class Program
{
    private static void Main()
    {
        ActorRuntime.RegisterActorAsync<MyActor>(
            (context, actorType) => new ActorService(context, actorType, () => new MyActor()))
            .GetAwaiter().GetResult();

        Thread.Sleep(Timeout.Infinite);
    }
}
```

### 执行组件服务方法

执行组件服务实现了 `IActorService`，该接口又实现了 `IService`。这是 Reliable Services 远程控制所使用的接口，该远程控制允许对服务方法执行远程过程调用。它包含可以使用服务远程控制进行远程调用的服务级别方法。


#### 枚举执行组件

执行组件服务允许客户端枚举有关该服务托管的执行组件的元数据。由于执行组件服务是已分区的有状态服务，因此将按分区执行枚举。因为每个分区可能包含大量执行组件，所以枚举以一组分页结果的形式返回。将循环读取这些页面，直到读取所有页面。以下示例演示了如何创建执行组件服务的一个分区中所有活动执行组件的列表：

```csharp
IActorService actorServiceProxy = ActorServiceProxy.Create(
    new Uri("fabric:/MyApp/MyService"), partitionKey);

ContinuationToken continuationToken = null;
List<ActorInformation> activeActors = new List<ActorInformation>();

do
{
    PagedResult<ActorInformation> page = await actorServiceProxy.GetActorsAsync(continuationToken, cancellationToken);
                
    activeActors.AddRange(page.Items.Where(x => x.IsActive));

    continuationToken = page.ContinuationToken;
}
while (continuationToken != null);
```

#### 删除执行组件

执行组件服务提供了一个用于删除执行组件的函数：

```csharp
ActorId actorToDelete = new ActorId(id);

IActorService myActorServiceProxy = ActorServiceProxy.Create(
    new Uri("fabric:/MyApp/MyService"), actorToDelete);
            
await myActorServiceProxy.DeleteActorAsync(actorToDelete, cancellationToken)
```

有关删除执行组件及其状态的详细信息，请参阅[执行组件生命周期文档](/documentation/articles/service-fabric-reliable-actors-lifecycle/)。

### 自定义执行组件服务

使用执行组件注册 lambda，你还可以注册自定义的派生自 `ActorService` 的执行组件服务，你可以在其中实现自己的服务级别的功能。可通过编写继承 `ActorService` 的服务类来完成此操作。自定义执行组件服务从 `ActorService` 继承了所有执行组件运行时功能，可用来实现你自己的服务方法。

```csharp
class MyActorService : ActorService
{
    public MyActorService(StatefulServiceContext context, ActorTypeInformation typeInfo, Func<ActorBase> newActor)
        : base(context, typeInfo, newActor)
    { }
}
```

```csharp
static class Program
{
    private static void Main()
    {
        ActorRuntime.RegisterActorAsync<MyActor>(
            (context, actorType) => new MyActorService(context, actorType, () => new MyActor()))
            .GetAwaiter().GetResult();

        Thread.Sleep(Timeout.Infinite);
    }
}
```


#### 实现执行组件备份和还原

 在下面的示例中，自定义执行组件服务通过利用已存在于 `ActorService` 中的远程侦听器公开备份执行组件数据的方法：

```csharp
public interface IMyActorService : IService
{
    Task BackupActorsAsync();
}

class MyActorService : ActorService, IMyActorService
{
    public MyActorService(StatefulServiceContext context, ActorTypeInformation typeInfo, Func<ActorBase> newActor)
        : base(context, typeInfo, newActor)
    { }

    public Task BackupActorsAsync()
    {
        return this.BackupAsync(new BackupDescription(...));
    }
}
```

在本示例中，`IMyActorService` 是一个实现 `IService`，然后由 `MyActorService` 实现的远程协定。通过添加此远程协定，并使用 `ActorServiceProxy` 创建远程代理，现在 `IMyActorService` 的方法也可用于客户端：

```csharp
IMyActorService myActorServiceProxy = ActorServiceProxy.Create<IMyActorService>(
    new Uri("fabric:/MyApp/MyService"), ActorId.CreateRandom());

await myActorServiceProxy.BackupActorsAsync();
```


## 应用程序模型

执行组件服务都是 Reliable Services，因此服务的应用程序模型是相同的。但是，执行组件框架生成工具为你生成了很多应用程序模型文件。

### 服务清单
 
执行组件服务的 ServiceManifest.xml 的内容由执行组件框架生成工具自动生成。这包括：

 - 执行组件服务类型。根据执行组件项目名称生成此类型名称。根据执行组件的持久性属性，还将相应设置 HasPersistedState 标志。
 - 代码包。
 - 配置包。
 - 资源和终结点

### 应用程序清单

执行组件框架生成工具将为你的执行组件服务自动创建默认的服务定义。由生成工具填充默认的服务属性：

 - 副本集计数由执行组件的持久性属性决定。每次更改执行组件的持久性属性时，将相应地重置默认服务定义中的副本集计数。
 - 分区方案和范围设置为具有完整的 Int64 键范围的统一 Int64。

## 针对执行组件的 Service Fabric 分区概念

执行组件服务是已分区的有状态服务。执行组件服务的每个分区包含一组执行组件。服务分区在 Service Fabric 的多个节点中自动分布。因此，执行组件实例也就被分布到各个节点中。

![执行组件分区和分布][5]

可使用不同的分区方案和分区键范围创建 Reliable Services。执行组件服务使用具有完整的 Int64 键范围的 Int64 分区方案将执行组件映射到分区。

### 执行组件 ID

服务中创建的每个执行组件具有与之关联的唯一 ID，并使用 `ActorId` 类表示。`ActorId` 是一个不透明的 ID 值，通过生成随机 ID，可将此值用于在各个服务分区中统一分布执行组件：

```csharp
ActorProxy.Create<IMyActor>(ActorId.CreateRandom());
```

每个 `ActorId` 都经过哈希算法转换为 Int64 类型值，这就是执行组件服务必须使用具有完整 Int64 键范围的 Int64 分区方案的原因。不过，`ActorID` 也可以使用自定义 ID 值，包括 GUID、字符串和 Int64。

```csharp
ActorProxy.Create<IMyActor>(new ActorId(Guid.NewGuid()));
ActorProxy.Create<IMyActor>(new ActorId("myActorId"));
ActorProxy.Create<IMyActor>(new ActorId(1234));
```

使用 GUID 和字符串时，这些值将经过哈希算法转换为 Int64。但是，如果向 `ActorId` 显式提供 Int64，此 Int64 将直接映射到分区，而无需进行哈希转换。这可用来控制将执行组件放入哪些分区。

## 后续步骤
 - [执行组件状态管理](/documentation/articles/service-fabric-reliable-actors-state-management/)
 - [执行组件生命周期和垃圾回收](/documentation/articles/service-fabric-reliable-actors-lifecycle/)
 - [执行组件 API 参考文档](https://msdn.microsoft.com/zh-cn/library/azure/dn971626.aspx)
 - [代码示例](https://github.com/Azure/servicefabric-samples)

 
<!--Image references-->
[1]: ./media/service-fabric-reliable-actors-platform/actor-service.png
[2]: ./media/service-fabric-reliable-actors-platform/app-deployment-scripts.png
[3]: ./media/service-fabric-reliable-actors-platform/actor-partition-info.png
[4]: ./media/service-fabric-reliable-actors-platform/actor-replica-role.png
[5]: ./media/service-fabric-reliable-actors-introduction/distribution.png
<!---HONumber=Mooncake_0503_2016-->