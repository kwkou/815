<properties
    pageTitle="Reliable Services 通知 | Azure"
    description="Service Fabric Reliable Services 通知的概念文档"
    services="service-fabric"
    documentationcenter=".net"
    author="mcoskun"
    manager="timlt"
    editor="masnider,vturecek" />
<tags
    ms.assetid="cdc918dd-5e81-49c8-a03d-7ddcd12a9a76"
    ms.service="service-fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="10/18/2016"
    wacn.date="12/26/2016"
    ms.author="mcoskun" />

# Reliable Services 通知
通知可让客户端跟踪对它们感兴趣的对象所进行的更改。两种类型的对象支持通知：*可靠状态管理器*和*可靠字典*。

使用通知的常见原因如下：

- 生成副本状态的具体化视图（例如次要索引）或聚合筛选视图。可靠字典中所有键的排序索引便是其中一个例子。
- 发送监视数据，例如过去一小时内添加的用户数目。

通知会在应用操作的过程中触发。因此，应该尽快处理通知，且同步事件不应包含任何耗费资源的操作。

## 可靠状态管理器通知

可靠状态管理器为以下事件提供通知：

- 事务
    - 提交
- 状态管理器
    - 重新生成
    - 添加可靠状态
    - 删除可靠状态

可靠状态管理器会跟踪当前正在进行的事务。唯一会导致通知触发的事务状态更改是提交事务。

可靠状态管理器会维护可靠状态的集合，例如可靠字典和可靠队列。可靠状态管理器会在此集合更改时触发通知：添加或删除可靠状态，或者重新生成整个集合时。可靠状态管理器集合会在以下三种情况下重新生成：

- 恢复：当副本启动时，它会从磁盘恢复其先前的状态。恢复结束时，它会使用 **NotifyStateManagerChangedEventArgs** 触发事件，其中包含一组已恢复的可靠状态。
- 完整副本：必须先生成副本，它才能加入配置集。有时，这需要将主要副本中可靠状态管理器状态的完整副本应用到空闲的次要副本。次要副本上的可靠状态管理器会使用 **NotifyStateManagerChangedEventArgs** 触发事件，其中包含一组它从主要副本获取的可靠状态。
- 还原：在灾难恢复方案中，副本的状态可通过 **RestoreAsync** 从备份还原。在这种情况下，主要副本上的可靠状态管理器会使用 **NotifyStateManagerChangedEventArgs** 触发事件，其中包含一组它从备份还原的可靠状态。

若要注册事务通知和/或状态管理器通知，需要在可靠状态管理器上注册 **TransactionChanged** 或 **StateManagerChanged** 事件。注册这些事件处理程序的常见位置是有状态服务的构造函数。如果在构造函数上注册，也不会错过 **IReliableStateManager** 生存期内的更改导致的任何通知。


	public MyService(StatefulServiceContext context)
    		: base(MyService.EndpointName, context, CreateReliableStateManager(context))
	{
    		this.StateManager.TransactionChanged += this.OnTransactionChangedHandler;
    		this.StateManager.StateManagerChanged += this.OnStateManagerChangedHandler;
	}


**TransactionChanged** 事件处理程序使用 **NotifyTransactionChangedEventArgs** 来提供有关事件的详细信息。它包含用于指定更改类型的操作属性（例如，**NotifyTransactionChangedAction.Commit**）。也包含提供对已更改事务的引用的事务属性。

>[AZURE.NOTE] 现在，只有提交事务才会引发 **TransactionChanged** 事件。此操作等同于 **NotifyTransactionChangedAction.Commit**。但是在未来，可能会有其他类型的事务状态更改可以引发事件。建议你检查操作，仅在你预期的事件发生时处理事件。

以下是 **TransactionChanged** 事件处理程序示例。


	private void OnTransactionChangedHandler(object sender, NotifyTransactionChangedEventArgs e)
	{
    		if (e.Action == NotifyTransactionChangedAction.Commit)
    		{
        		this.lastCommitLsn = e.Transaction.CommitSequenceNumber;
        		this.lastTransactionId = e.Transaction.TransactionId;

        		this.lastCommittedTransactionList.Add(e.Transaction.TransactionId);
    		}
	}


**StateManagerChanged** 事件处理程序使用 **NotifyStateManagerChangedEventArgs** 来提供有关事件的详细信息。**NotifyStateManagerChangedEventArgs** 有两个子类：**NotifyStateManagerRebuildEventArgs** 和 **NotifyStateManagerSingleEntityChangedEventArgs**。使用 **NotifyStateManagerChangedEventArgs** 中的操作属性将 **NotifyStateManagerChangedEventArgs** 转换为正确的子类：

- **NotifyStateManagerChangedAction.Rebuild**：**NotifyStateManagerRebuildEventArgs**
- **NotifyStateManagerChangedAction.Add** 和 **NotifyStateManagerChangedAction.Remove**：**NotifyStateManagerSingleEntityChangedEventArgs**

以下是 **StateManagerChanged** 通知处理程序示例。


	public void OnStateManagerChangedHandler(object sender, NotifyStateManagerChangedEventArgs e)
	{
    		if (e.Action == NotifyStateManagerChangedAction.Rebuild)
    		{
        		this.ProcessStataManagerRebuildNotification(e);

        		return;
    		}

    		this.ProcessStateManagerSingleEntityNotification(e);
	}


## 可靠字典通知
可靠字典为以下事件提供通知：

- 重新生成：在 **ReliableDictionary** 从恢复或复制的本地状态或备份恢复其状态后调用。
- 清除：在通过 **ClearAsync** 方法清除 **ReliableDictionary** 的状态后调用。
- 添加：在向 **ReliableDictionary** 添加项后调用。
- 更新：在更新 **IReliableDictionary** 中的项后调用。
- 删除：在删除 **IReliableDictionary** 中的项后调用。

若要获取可靠字典通知，需要在 **IReliableDictionary** 上注册 **DictionaryChanged** 事件处理程序。通常在 **ReliableStateManager.StateManagerChanged** 添加通知中注册这些事件处理程序。在将 **IReliableDictionary** 添加到 **IReliableStateManager** 时注册，可确保不会错过任何通知。


	private void ProcessStateManagerSingleEntityNotification(NotifyStateManagerChangedEventArgs e)
	{
    		var operation = e as NotifyStateManagerSingleEntityChangedEventArgs;

    		if (operation.Action == NotifyStateManagerChangedAction.Add)
    		{
        		if (operation.ReliableState is IReliableDictionary<TKey, TValue>)
        		{
            			var dictionary = (IReliableDictionary<TKey, TValue>)operation.ReliableState;
            			dictionary.RebuildNotificationAsyncCallback = this.OnDictionaryRebuildNotificationHandlerAsync;
            			dictionary.DictionaryChanged += this.OnDictionaryChangedHandler;
            			}
        		}
    		}
	}


>[AZURE.NOTE] **ProcessStateManagerSingleEntityNotification** 是上述 **OnStateManagerChangedHandler** 示例所调用的示例方法。

上述代码会设置 **IReliableNotificationAsyncCallback** 接口以及 **DictionaryChanged**。**NotifyDictionaryRebuildEventArgs** 包含需要以异步方式枚举的 **IAsyncEnumerable** 接口，因此，会通过 **RebuildNotificationAsyncCallback**（而不是 **OnDictionaryChangedHandler**）来触发重新生成通知。


	public async Task OnDictionaryRebuildNotificationHandlerAsync(
    		IReliableDictionary<TKey, TValue> origin,
    		NotifyDictionaryRebuildEventArgs<TKey, TValue> rebuildNotification)
	{
    		this.secondaryIndex.Clear();

    		var enumerator = e.State.GetAsyncEnumerator();
    		while (await enumerator.MoveNextAsync(CancellationToken.None))
    		{
        		this.secondaryIndex.Add(enumerator.Current.Key, enumerator.Current.Value);
    		}
	}


>[AZURE.NOTE] 在上述代码中，在处理重新生成通知的过程中，会先清除所维护的聚合状态。由于正在利用新状态重新生成可靠集合，因此与以前的所有通知不相关。

**DictionaryChanged** 事件处理程序使用 **NotifyDictionaryChangedEventArgs** 来提供有关事件的详细信息。**NotifyDictionaryChangedEventArgs** 有五个子类。使用 **NotifyDictionaryChangedEventArgs** 中的操作属性将 **NotifyDictionaryChangedEventArgs** 转换为正确的子类：

- **NotifyDictionaryChangedAction.Rebuild**：**NotifyDictionaryRebuildEventArgs**
- **NotifyDictionaryChangedAction.Clear**：**NotifyDictionaryClearEventArgs**
- **NotifyDictionaryChangedAction.Add** 和 **NotifyDictionaryChangedAction.Remove**：**NotifyDictionaryItemAddedEventArgs**
- **NotifyDictionaryChangedAction.Update**：**NotifyDictionaryItemUpdatedEventArgs**
- **NotifyDictionaryChangedAction.Remove**：**NotifyDictionaryItemRemovedEventArgs**


		public void OnDictionaryChangedHandler(object sender, NotifyDictionaryChangedEventArgs<TKey, TValue> e)
		{
	    		switch (e.Action)
	    		{
	        		case NotifyDictionaryChangedAction.Clear:
	            			var clearEvent = e as NotifyDictionaryClearEventArgs<TKey, TValue>;
	            			this.ProcessClearNotification(clearEvent);
	            			return;
	
	        		case NotifyDictionaryChangedAction.Add:
	            			var addEvent = e as NotifyDictionaryItemAddedEventArgs<TKey, TValue>;
	            			this.ProcessAddNotification(addEvent);
	            			return;
	
	        		case NotifyDictionaryChangedAction.Update:
	            			var updateEvent = e as NotifyDictionaryItemUpdatedEventArgs<TKey, TValue>;
	            			this.ProcessUpdateNotification(updateEvent);
	            			return;
	
	        		case NotifyDictionaryChangedAction.Remove:
	            			var deleteEvent = e as NotifyDictionaryItemRemovedEventArgs<TKey, TValue>;
	            			this.ProcessRemoveNotification(deleteEvent);
	            			return;
	
	        		default:
	            			break;
	    		}
		}


## 建议

- *请*尽快完成通知事件。
- *请勿*在同步事件中执行任何耗费资源的操作（例如，I/O 操作）。
- *请*在处理事件前检查操作类型。未来可能会添加新的操作类型。

需谨记以下几点：

- 通知会在执行操作的过程中触发。例如，在还原操作的最后一个步骤触发还原通知。处理通知事件之前，不会完成还原。
- 由于通知会在应用操作的过程中触发，因此，客户端只会看见本地提交操作的通知。而且因为操作只保证会在本地提交（亦即记录），所以它们不一定可在未来恢复。
- 在恢复路径上，会针对每个应用的操作触发单个通知。这表示，如果事务 T1 包含 Create(X)、Delete(X) 和 Create(X)，你将依次收到一个针对 X 创建的通知，一个针对删除的通知，然后再收到一个针对创建的通知。
- 对于包含多个操作的事务，操作将按用户在主要副本上收到它们的顺序应用。
- 在处理错误进度的过程中，某些操作可能会恢复。通知会针对这类恢复操作加以触发，将副本状态回滚到稳定的时间点。恢复通知的一个重要区别，是具有重复键的事件会聚合在一起。例如，如果恢复事务 T1，你将看到一条针对 Delete(X) 的通知。

## 后续步骤
- [Reliable Collections](/documentation/articles/service-fabric-work-with-reliable-collections/)
- [Reliable Services 快速启动](/documentation/articles/service-fabric-reliable-services-quick-start/)
- [Reliable Services 备份和还原（灾难恢复）](/documentation/articles/service-fabric-reliable-services-backup-restore/)
- [Reliable Collections 的开发人员参考](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicefabric.data.collections.aspx)

<!---HONumber=Mooncake_1219_2016-->