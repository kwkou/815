<properties
    pageTitle="使用可靠集合 | Azure"
    description="了解有关使用可靠集合的最佳实践。"
    services="service-fabric"
    documentationCenter=".net"
    authors="JeffreyRichter"
    manager="timlt"
    editor="" />

<tags
    ms.service="multiple"
    wacn.date="07/04/2016"/>

# 使用可靠集合

Service Fabric 通过可靠集合向 .NET 开发人员提供有状态的编程模型。具体而言，Service Fabric 提供可靠字典和可靠队列类。当你使用这些类时，状态是分区的（实现可缩放性）、复制的（实现可用性），并在分区内进行事务处理（实现 ACID 语义）。让我们看一下可靠字典对象的典型用法，并看一看它究竟做些什么。

~~~
retry:
try {
   // Create a new Transaction object for this partition
   using (ITransaction tx = base.StateManager.CreateTransaction()) {
      // AddAsync takes key's write lock; if >4 secs, TimeoutException
      // Key & value put in temp dictionary (read your own writes),
      // serialized, redo/undo record is logged & sent to  
      // secondary replicas
      await m_dic.AddAsync(tx, key, value, cancellationToken);

      // CommitAsync sends Commit record to log & secondary replicas
      // After quorum responds, all locks released
      await tx.CommitAsync();
   }
   // If CommitAsync not called, Dispose sends Abort
   // record to log & all locks released
}
catch (TimeoutException) { 
   await Task.Delay(100, cancellationToken); goto retry; 
}
~~~

可靠字典对象上的所有操作（无法恢复的 ClearAsync 除外）都需要一个 ITransaction 对象。此对象与在单个分区中对任何可靠字典和/或可靠队列对象尝试进行的任何及所有更改具有关联性。可通过调用分区的 StateManager 的 CreateTransaction 方法获取 ITransaction 对象。
 
在上面的代码中，ITransaction 对象传递到可靠字典的 AddAsync 方法。在内部，接受键的字典方法采用与键关联的读取器/写入器锁。如果此方法修改键的值，将在键上使用写入锁；如果方法只读取键的值，将在键上使用读取锁。由于 AddAsync 将键值修改成新的传入值，因此使用键的写入锁。因此，如果有 2（或更多个）线程尝试在同一时间添加相同的键值，则一个线程将获取写入锁，另一个线程将会阻塞。默认情况下，方法最多阻塞 4 秒以获取锁，4 秒后方法将引发 TimeoutException。方法重载可让你根据需要传递显式超时值。
 
通常，编写代码响应 TimeoutException 的方式是捕获它，然后重试整个操作（如以上代码中所示）。在我的简单代码中，我只调用了每次传递 100 毫秒的 Task.Delay。但实际上，最好改用某种形式的指数退让延迟。
 
获取锁后，AddAsync 将在与 ITransaction 对象关联的内部临时字典中添加键和值对象引用。这就完成了读取自己编写的语义。也就是说，在调用 AddAsync 之后，稍后对 TryGetValueAsync 的调用（使用相同的 ITransaction 对象）将返回值，即使尚未提交事务。接下来，AddAsync 将键和值对象序列化为字节数组，并将这些字节数组附加到本地节点的日志文件。最后，AddAsync 将字节数组发送给所有辅助副本，使其具有相同的键/值信息。即使键/值信息已写入日志文件，在提交其关联的事务之前，这些信息不被视为字典的一部分。

在上述代码中，调用 CommitAsync 会提交所有事务操作。具体而言，它将提交信息附加到本地节点的日志文件，同时将提交记录发送给所有辅助副本。回复副本的仲裁（多数）后，所有数据更改将被视为永久性，并释放通过 ITransaction 对象操作的任何键关联锁，使其他线程/事务可以操作相同的键及其值。

如果未调用 CommitAsync（通常是因为引发了异常），则会释放 ITransaction 对象。在释放未提交的 ITransaction 对象时，Service Fabric 会中止将信息附加到本地节点的日志文件，且不需要将任何信息发送到任何辅助副本。然后将释放通过事务操作的任何与键关联的锁。

## 常见陷阱及其规避方法
现在你已了解可靠集合在内部的工作原理，让我们了解一些常见的误用。参阅以下代码：

~~~
using (ITransaction tx = StateManager.CreateTransaction()) {
   // AddAsync serializes the name/user, logs the bytes, 
   // & sends the bytes to the secondary replicas.
   await m_dic.AddAsync(tx, name, user);

   // The line below updates the property’s value in memory only; the
   // new value is NOT serialized, logged, & sent to secondary replicas.
   user.LastLogin = DateTime.UtcNow;  // Corruption!

   await tx.CommitAsync();
}
~~~

使用常规 .NET 字典时，可以在字典中添加键/值，然后更改属性的值（例如 LastLogin）。不过，此代码无法对可靠字典正常运行。我们前面讨论过：调用 AddAsync 将键/值对象序列化成字节数组，然后将数组存储到本地文件，并将它们发送到辅助副本。稍后如果更改属性，只会更改内存中的属性值，而不影响本地文件或发送到副本的数据。如果进程崩溃，内存中的内容将全部丢失。启动新的进程或另一个副本变成主副本时，旧属性值是可用的值。

再次强调，上面这种错误是很容易发生的。你只有在进程崩溃时才能发现错误。编写代码的正确方式是只需反转两行：

~~~
using (ITransaction tx = StateManager.CreateTransaction()) {
   user.LastLogin = DateTime.UtcNow;  // Do this BEFORE calling AddAsync
   await m_dic.AddAsync(tx, name, user);
   await tx.CommitAsync(); 
}
~~~

这是另一个常见的错误：

~~~
using (ITransaction tx = StateManager.CreateTransaction()) {
   // Use the user’s name to look up their data
   ConditionalValue<User> user = 
      await m_dic.TryGetValueAsync(tx, name);

   // The user exists in the dictionary, update one of their properties.
   if (user.HasValue) {
      // The line below updates the property’s value in memory only; the
      // new value is NOT serialized, logged, & sent to secondary replicas.
      user.Value.LastLogin = DateTime.UtcNow; // Corruption!
      await tx.CommitAsync(); 
   }
}
~~~

同样地，使用常规 .NET 字典时，以上代码以常见的模式正常运行：开发人员使用键查询值。如果值存在，开发人员将更改属性的值。不过，使用可靠集合时，此代码将出现前面所述的相同问题：__将对象分配给可靠集合后，你不得修改该对象__。
 
在可靠集合中更新值的正确方式是获取对现有值的引用，并将此引用所引用的对象视为不可变。然后创建新的对象，即原始对象的完全相同副本。现在，可以修改此新对象的状态，将新对象写入集合，以便将它序列化为字节数组、附加到本地文件并发送到副本。提交更改之后，内存中的对象、本地文件和所有副本都处于完全一致的状态。大功告成！

以下代码演示在可靠集合中更新值的正确方式：

~~~
using (ITransaction tx = StateManager.CreateTransaction()) {
   // Use the user’s name to look up their data
   ConditionalValue<User> currentUser = 
      await m_dic.TryGetValueAsync(tx, name);

   // The user exists in the dictionary, update one of their properties.
   if (currentUser.HasValue) {
      // Create new user object with the same state as the current user object.
      // NOTE: This must be a deep copy; not a shallow copy. Specifically, only
      // immutable state can be shared by currentUser & updatedUser object graphs.
      User updatedUser = new User(currentUser);

      // In the new object, modify any properties you desire 
      updatedUser.LastLogin = DateTime.UtcNow;

      // Update the key’s value to the updateUser info
      await m_dic.SetValue(tx, name, updatedUser);

      await tx.CommitAsync(); 
   }
}
~~~

## 定义不可变的数据类型以防止编程器错误

理想情况下，我们希望编译器能够在你意外生成改变对象状态的代码、而此对象又不该改变时报告错误。但是 C# 编译器做不到这一点。因此，为了避免潜在的编程器错误，我们强烈建议将可靠集合使用的类型定义为不可变类型。具体而言，这意味着你要坚持使用核心值类型（例如数字 [Int32、UInt64 等]、DateTime、Guid、TimeSpan 等）。当然，你也可以使用字符串。最好是避免集合属性，因为将其序列化和反序列化经常会降低性能。但是，如果你想要使用集合属性，我们强烈建议使用 .NET 的不可变集合库 (System.Collections.Immutable)。可以从 http://nuget.org 下载此库。此外，我们建议尽可能地密封类，并将字段设为只读。

以下 UserInfo 类型演示如何利用上述建议定义不可变类型。

~~~
[DataContract]
// If you don’t seal, you must ensure that any derived classes are also immutable
public sealed class UserInfo {
   private static readonly IEnumerable<ItemId> NoBids = ImmutableList<ItemId>.Empty;
 
   public UserInfo(String email, IEnumerable<ItemId> itemsBidding = null) {
      Email = email;
      ItemsBidding = (itemsBidding == null) ? NoBids : itemsBidding.ToImmutableList();
   }

   [OnDeserialized]
   private void OnDeserialized(StreamingContext context) {
      // Convert the deserialized collection to an immutable collection
      ItemsBidding = ItemsBidding.ToImmutableList();
   }
 
   [DataMember]
   public readonly String Email;
 
   // Ideally, this would be a readonly field but it can't be because OnDeserialized 
   // has to set it. So instead, the getter is public and the setter is private.
   [DataMember]
   public IEnumerable<ItemId> ItemsBidding { get; private set; }

   // Since each UserInfo object is immutable, we add a new ItemId to the ItemsBidding
   // collection by creating a new immutable UserInfo object with the added ItemId.
   public UserInfo AddItemBidding(ItemId itemId) {
      return new UserInfo(Email, ((ImmutableList<ItemId>)ItemsBidding).Add(itemId));
   }
}
~~~

ItemId 类型也是不可变类型，如下所示：

~~~
[DataContract]
public struct ItemId {

   [DataMember] public readonly String Seller;
   [DataMember] public readonly String ItemName;
   public ItemId(String seller, String itemName) {
      Seller = seller;
      ItemName = itemName;
   }
}
~~~

## 架构版本控制（升级）

就内部而言，可靠集合使用 .NET 的 DataContractSerializer 序列化对象。序列化的对象保存在主副本的本地磁盘中，并发送到辅助副本。随着服务日趋成熟，你可能想要更改服务所需的数据种类（架构）。你必须十分谨慎对待数据的版本控制方法。首先但同样重要的是，你始终必须能够反序列化旧数据。具体而言，这意味着反序列化代码必须具有无限向后兼容性：服务代码的版本 333 必须能够对 5 年前放在可靠集合中第 1 版的服务代码数据运行。

此外，服务代码一次只升级一个域。因此，在升级期间，同时执行两个不同版本的服务代码。必须避免新版本的服务代码使用新的架构，因为旧版的服务代码可能无法处理新的架构。应该尽可能将每个版本的服务都设计成向前兼容 1 个版本。具体而言，这意味着 V1 的服务代码只要能够忽略它不显式处理的任何架构元素即可。但是，它必须能够保存它不显式了解的任何数据，在更新字典键或值时只需将它写回即可。

>[AZURE.WARNING] 尽管你可以修改键的架构，但必须确保键哈希代码和相等算法是稳定的。如果更改其中任一算法的工作方式，你再也无法在可靠字典中查询键。
  
或者，也可以执行通称为 2 阶段升级的功能。使用 2 阶段升级，服务可从 V1 升级成 V2：V2 包含知道如何处理新架构更改的代码，但这段代码不执行。当 V2 代码读取 V1 数据时，它在其上操作并写入 V1 数据。然后，在跨所有升级域的升级都完成之后，就可以通知运行中的 V2 实例，升级已完成。（通知方式之一是推出配置升级；这就是 2 阶段升级）。 现在，V2 实例可以读取 V1 数据，将它转换成 V2 数据、操作它，然后写出为 V2 数据。当其他实例读取 V2 数据时，不需要转换它，只要操作并写出 V2 数据即可。

## 后续步骤
若要了解如何创建向前兼容的数据约定，请参阅 [Forward-Compatible Data Contracts（向前兼容的数据约定）](https://msdn.microsoft.com/zh-cn/library/ms731083.aspx)。

若要了解版本控制数据约定的最佳实践，请参阅 [Data Contract Versioning（数据约定版本控制）](https://msdn.microsoft.com/zh-cn/library/ms731138.aspx)。

若要了解如何实现版本容错的数据约定，请参阅 [Version-Tolerant Serialization Callbacks（版本容错的序列化回调）](https://msdn.microsoft.com/zh-cn/library/ms733734.aspx)。

若要了解如何提供可跨多个版本互操作的数据结构，请参阅 [IExtensibleDataObject](https://msdn.microsoft.com/zh-cn/library/system.runtime.serialization.iextensibledataobject.aspx)。

<!---HONumber=Mooncake_0425_2016-->