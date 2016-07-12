<properties
   pageTitle="Reliable Actors 状态管理 | Azure"
   description="介绍如何管理、持久保存和复制 Reliable Actors 状态以实现高可用性。"
   services="service-fabric"
   documentationCenter=".net"
   authors="vturecek"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.date="03/25/2016"
   wacn.date="07/04/2016"/>

# Reliable Actors 状态管理

Reliable Actors 是可封装逻辑与状态的单线程对象。由于执行组件在 Reliable Services 上执行，因此，它们可以使用 Reliable Services 所用的相同持久性和复制机制可靠地保护状态。这样，执行组件就不会在发生故障之后、在内存回收后重新激活时或者由于资源平衡和升级的原因而在群集中的节点之间移动时丢失其状态。

## 状态持久性和复制

之所以认为所有 Reliable Actors 有状态，是因为每个执行组件实例将映射到唯一的 ID。这意味着，对同一个执行组件 ID 所做的重复调用将路由到同一个执行组件实例。这是相比于无状态系统的情况，无状态系统中的客户端调用不一定每次都路由到同一台服务器。出于此原因，执行组件服务永远都是有状态服务。

但是，即使执行组件被视为有状态，但也并不表示它们必须以可靠的方式存储状态。执行组件可以根据其数据存储要求来选择状态持久性和复制的级别：

 - **持久化状态：**状态持久保存在磁盘中，并复制到 3 个或更多个副本。这是最持久的状态存储选项，可通过完全群集中断来持久保留状态。
 - **易失性状态：**状态复制到 3 个或更多个副本，并且只保留在内存中。这可针对节点故障、执行组件故障，以及在升级和资源平衡过程中提供复原能力。但是，状态不会持久保留在磁盘中，因此，如果同时丢失所有副本，状态也会丢失。
 - **非持久化状态：**状态不复制，也不写入磁盘。适用于完全不需要以可靠方式维护状态的执行组件。
 
每个级别的持久性只是服务的不同状态提供程序和复制配置。是否要将状态写入磁盘取决于状态提供程序（Reliable Service 中存储状态的组件），而复制取决于要使用多少个副本来部署服务。如同 Reliable Services 一样，你可以轻松地手动设置状态提供程序和副本计数。执行组件框架提供一个属性，对执行组件使用时，该属性将自动选择默认的状态提供程序，并自动生成副本计数的设置，以实现这三种持久性设置中的一个。

### 持久化状态
```csharp
[StatePersistence(StatePersistence.Persisted)]
class MyActor : Actor, IMyActor
{
}
```  
此设置使用一个状态提供程序，该提供程序可在磁盘上存储数据，并自动将服务副本计数设置为 3。

### 易失性状态
```csharp
[StatePersistence(StatePersistence.Volatile)]
class MyActor : Actor, IMyActor
{
}
```
此设置使用仅在内存中的状态提供程序，并将副本计数设置为 3。

### 非持久化状态

```csharp
[StatePersistence(StatePersistence.None)]
class MyActor : Actor, IMyActor
{
}
```
此设置使用仅在内存中的状态提供程序，并将副本计数设置为 1。

### 默认值和生成的设置

如果使用 `StatePersistence` 属性，在执行组件服务启动时，系统会在运行时自动为你选择状态提供程序。但是，副本计数将在编译时由 Visual Studio 执行组件构建工具设置。构建工具在 ApplicationManifest.xml 中自动为执行组件服务生成默认服务。参数是针对**副本集大小下限**和**目标副本集大小**创建的。当然，你可以手动更改这些参数，但是，每当 `StatePersistence` 属性更改时，参数将设置为所选 `StatePersistence` 属性的默认副本集大小值，并覆盖所有旧值。

```xml
<ApplicationManifest xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" ApplicationTypeName="Application12Type" ApplicationTypeVersion="1.0.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">
   <Parameters>
      <Parameter Name="MyActorService_PartitionCount" DefaultValue="10" />
      <Parameter Name="MyActorService_MinReplicaSetSize" DefaultValue="2" />
      <Parameter Name="MyActorService_TargetReplicaSetSize" DefaultValue="3" />
   </Parameters>
   <ServiceManifestImport>
      <ServiceManifestRef ServiceManifestName="MyActorPkg" ServiceManifestVersion="1.0.0" />
   </ServiceManifestImport>
   <DefaultServices>
      <Service Name="MyActorService" GeneratedIdRef="77d965dc-85fb-488c-bd06-c6c1fe29d593|Persisted">
         <StatefulService ServiceTypeName="MyActorServiceType" TargetReplicaSetSize="[MyActorService_TargetReplicaSetSize]" MinReplicaSetSize="[MyActorService_MinReplicaSetSize]">
            <UniformInt64Partition PartitionCount="[MyActorService_PartitionCount]" LowKey="-9223372036854775808" HighKey="9223372036854775807" />
         </StatefulService>
      </Service>
   </DefaultServices>
</ApplicationManifest>
```

## 状态管理器

每个执行组件实例都有其自身的状态管理器：一种类似于字典的数据结构，能够可靠地存储密钥-值对。状态管理器是围绕状态提供程序的包装。它可用于存储数据，而不论使用的是哪一种持久性设置，但不保证运行中的执行组件服务可在保留数据的同时，以通过滚动升级从易失性（仅在内存中）状态设置更改保存的状态设置。但是，针对运行中的服务更改副本计数是可行的。

状态管理器键必须是字符串，而值是泛型且可以是任何类型，包括自定义类型。存储在状态管理器中的值必须可进行数据约定序列化的，因为根据执行组件的状态持久性设置，它们可能在复制期间通过网络传输到其他节点，并且可能写入磁盘。

状态管理器公开通用字典方法来管理状态，类似于在可靠字典中找到的项目。

### 访问状态

可以通过状态管理器根据键来访问状态。状态管理器方法全都是异步的，因为当执行组件具有保存的状态时，它们可能需要磁盘 I/O。首次访问时，状态对象将缓存在内存中。重复访问操作将从内存中直接访问对象并以同步方式返回，而不造成磁盘 I/O 或异步上下文切换的开销。在以下情况中，将从缓存中删除状态对象：

 - 从状态管理器中检索对象之后，执行组件方法将引发未处理的异常。
 - 执行组件在停用之后或发生故障后将重新激活。
 - 如果状态提供程序将状态分页到磁盘。此行为取决于状态提供程序实现。`Persisted` 设置的默认状态提供程序具有此行为。 

如果给定键的条目不存在，可以使用引发 `KeyNotFoundException` 的标准 Get 操作来检索状态：

```csharp
[StatePersistence(StatePersistence.Persisted)]
class MyActor : Actor, IMyActor
{
    public Task<int> GetCountAsync()
    {
        return this.StateManager.GetStateAsync<int>("MyState");
    }
}
```

如果给定键的条目不存在，也可以使用不引发异常的 TryGet 方法来检索状态：

```csharp
class MyActor : Actor, IMyActor
{
    public async Task<int> GetCountAsync()
    {
        ConditionalValue<int> result = await this.StateManager.TryGetStateAsync<int>("MyState");
        if (result.HasValue)
        {
            return result.Value;
        }

        return 0;
    }
}
```

### 保存状态

状态管理器检索方法将返回对本地内存中对象的引用。只是在本地内存中修改此对象并不会永久存储该对象。从状态管理器检索和修改对象时，必须将它重新插入状态管理器才能永久保存。

可以使用无条件的 Set（相当于 **dictionary["key"] = value** 语法）来插入状态：

```csharp
[StatePersistence(StatePersistence.Persisted)]
class MyActor : Actor, IMyActor
{
    public Task SetCountAsync(int value)
    {
        return this.StateManager.SetStateAsync<int>("MyState", value);
    }
}
```

可以使用 Add 方法来添加状态，但尝试添加已存在的键时会引发 `InvalidOperationException`：

```csharp
[StatePersistence(StatePersistence.Persisted)]
class MyActor : Actor, IMyActor
{
    public Task AddCountAsync(int value)
    {
        return this.StateManager.AddStateAsync<int>("MyState", value);
    }
}
```

还可以使用 TryAdd 方法来添加状态，但尝试添加已存在的键时不会引发异常：

```csharp
[StatePersistence(StatePersistence.Persisted)]
class MyActor : Actor, IMyActor
{
    public async Task AddCountAsync(int value)
    {
        bool result = await this.StateManager.TryAddStateAsync<int>("MyState", value);

        if (result)
        {
            // Added successfully!
        }
    }
}
```

在执行组件方法结束时，状态管理器将自动保存通过插入或更新操作添加或修改的任何值。根据所用的设置，“保存”可能包括持久保存到磁盘和复制。未修改的值不会持久保存或复制。如果未修改任何值，保存操作不起作用。如果保存失败，将丢弃修改的状态并重新加载原始状态。

也可以通过对执行组件基调用 `SaveStateAsync` 方法来手动保存状态：

```csharp
async Task IMyActor.SetCountAsync(int count)
{
    await this.StateManager.AddOrUpdateStateAsync("count", count, (key, value) => count > value ? count : value);
            
    await this.SaveStateAsync();
}
```

### 删除状态

可以通过调用 Remove 方法，从执行组件的状态管理器中永久删除状态。尝试删除不存在的键时，此方法将引发 `KeyNotFoundException`：

```csharp
[StatePersistence(StatePersistence.Persisted)]
class MyActor : Actor, IMyActor
{
    public Task RemoveCountAsync()
    {
        return this.StateManager.RemoveStateAsync("MyState");
    }
}
```

也可以使用 TryRemove 方法永久删除状态，此方法在尝试删除不存在的键时不会引发任何异常：

```csharp
[StatePersistence(StatePersistence.Persisted)]
class MyActor : Actor, IMyActor
{
    public async Task RemoveCountAsync()
    {
        bool result = await this.StateManager.TryRemoveStateAsync("MyState");

        if (result)
        {
            // State removed!
        }
    }
}
```

## 后续步骤
 - [执行组件类型序列化](/documentation/articles/service-fabric-reliable-actors-notes-on-actor-type-serialization/)
 - [执行组件多态性和面向对象的设计模式](/documentation/articles/service-fabric-reliable-actors-polymorphism/)
 - [执行组件诊断和性能监视](/documentation/articles/service-fabric-reliable-actors-diagnostics/)
 - [执行组件 API 参考文档](https://msdn.microsoft.com/zh-cn/library/azure/dn971626.aspx)
 - [代码示例](https://github.com/azure-samples/service-fabric-dotnet-getting-started)

<!---HONumber=Mooncake_0425_2016-->