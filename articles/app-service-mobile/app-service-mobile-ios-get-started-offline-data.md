<properties
	pageTitle="启用 iOS 移动应用的脱机同步功能"
	description="了解如何在 iOS 应用中使用 Azure App Service 移动应用来缓存和同步脱机数据"
	documentationCenter="ios"
	author="ysxu"
	manager="yochayk"
	editor=""
	services="app-service\mobile"/>

<tags
	ms.service="app-service-mobile"
	ms.workload="mobile"
	ms.tgt_pltfrm="mobile-ios"
	ms.devlang="objective-c"
	ms.topic="article"
	ms.date="10/01/2016"
	wacn.date="03/20/2017"
	ms.author="yuaxu"/>


# 启用 iOS 移动应用的脱机同步功能

[AZURE.INCLUDE [app-service-mobile-selector-offline](../../includes/app-service-mobile-selector-offline.md)]

## 概述
本教程介绍 Azure App Service for iOS 的移动应用功能的脱机同步。通过脱机同步功能，最终用户可在无网络连接的情况下与移动应用进行交互，以查看、添加或修改数据。在本地数据库中存储更改。设备重新联机后，这些更改会与远程后端同步。

对于首次体验 移动应用的用户，请先完成[创建 iOS 应用]教程。如果不使用下载的快速入门服务器项目，必须将数据访问扩展包添加到项目。有关服务器扩展包的详细信息，请参阅 [Work with the .NET backend server SDK for Azure Mobile Apps](/documentation/articles/app-service-mobile-dotnet-backend-how-to-use-server-sdk/)（使用适用于 Azure 移动应用的 .NET 后端服务器 SDK）。

若要了解有关脱机同步功能的详细信息，请参阅[移动应用中的脱机数据同步]。

## <a name="review-sync"></a>查看客户端同步代码
为[创建 iOS 应用]教程下载的客户端项目已包含使用基于核心数据的本地数据库支持脱机同步的代码。本节概括了教程代码中包含的内容。有关此功能的概念性概述，请参阅[移动应用中的脱机数据同步]。

使用移动应用的脱机数据同步功能，最终用户可在无法访问网络时仍能够与本地数据库进行交互。若要在应用中使用这些功能，你可以初始化 `MSClient` 的同步上下文，并引用本机存储。然后通过 **MSSyncTable** 接口引用表。

在 **QSTodoService.m** (Objective-C) 或 **ToDoTableViewController.swift** (Swift) 中，请注意成员 **syncTable** 的类型为 **MSSyncTable**。脱机同步使用此同步表接口而不是 **MSTable**。使用同步表时，所有操作将转到本地存储，而且只会与具有显式推送和提取操作的远程后端同步。

 若要获取对同步表的引用，请对 `MSClient` 使用 **syncTableWithName** 方法。若要删除脱机同步功能，请改用 **tableWithName**。

2. 表操作之前，必须初始化本地存储区。下面是相关的代码。
	
	**Objective-C**：
	
	在 `QSTodoService.init` 方法中：
	
	
	        MSCoreDataStore *store = [[MSCoreDataStore alloc] initWithManagedObjectContext:context];
	        self.client.syncContext = [[MSSyncContext alloc] initWithDelegate:nil dataSource:store callback:nil];
	
	
	**Swift**：
	
	在 `ToDoTableViewController.viewDidLoad` 方法中：
	
	
	        let client = MSClient(applicationURLString: "http:// ...") // URI of the Mobile App
	        let managedObjectContext = (UIApplication.sharedApplication().delegate as! AppDelegate).managedObjectContext!
	        self.store = MSCoreDataStore(managedObjectContext: managedObjectContext)
	        client.syncContext = MSSyncContext(delegate: nil, dataSource: self.store, callback: nil)
	

	随后将使用移动应用 SDK 中提供的接口 `MSCoreDataStore` 创建本地存储。可以改为通过实现 `MSSyncContextDataSource` 协议提供不同的本地存储。
	
	此外，`MSSyncContext` 的第一个参数用于指定冲突处理程序。由于已传递 `nil`，因此将获取默认的冲突处理程序，但该处理程序在发生任何冲突时会失败。
	
3. 现在，让我们执行实际的同步操作，从远程后端获取数据。

	**Objective-C**：
	
	`syncData` 先推送新的更改，然后调用 `pullData` 以便从远程后端获取数据。接下来，`pullData` 方法获取符合查询的新数据：
	
	
	        -(void)syncData:(QSCompletionBlock)completion
	        {
	            // push all changes in the sync context, and then pull new data
	            [self.client.syncContext pushWithCompletion:^(NSError *error) {
	                [self logErrorIfNotNil:error];
	                [self pullData:completion];
	            }];
	        }
	
	        -(void)pullData:(QSCompletionBlock)completion
	        {
	            MSQuery *query = [self.syncTable query];
	
	            // Pulls data from the remote server into the local table.
	            // We're pulling all items and filtering in the view
	            // query ID is used for incremental sync
	            [self.syncTable pullWithQuery:query queryId:@"allTodoItems" completion:^(NSError *error) {
	                [self logErrorIfNotNil:error];
	
	                // Let the caller know that we have finished
	                if (completion != nil) {
	                    dispatch_async(dispatch_get_main_queue(), completion);
	                }
	            }];
	        }
        
        
      **Swift**：
        
        
		func onRefresh(sender: UIRefreshControl!) {
		    UIApplication.sharedApplication().networkActivityIndicatorVisible = true
		    
		    self.table!.pullWithQuery(self.table?.query(), queryId: "AllRecords") {
		        (error) -> Void in
		        
		        UIApplication.sharedApplication().networkActivityIndicatorVisible = false
		        
		        if error != nil {
		            // A real application would handle various errors like network conditions,
		            // server conflicts, etc via the MSSyncContextDelegate
		            print("Error: (error!.description)")
		            
		            // We will discard our changes and keep the servers copy for simplicity
		            if let opErrors = error!.userInfo[MSErrorPushResultKey] as? Array<MSTableOperationError> {
		                for opError in opErrors {
		                    print("Attempted operation to item (opError.itemId)")
		                    if (opError.operation == .Insert || opError.operation == .Delete) {
		                        print("Insert/Delete, failed discarding changes")
		                        opError.cancelOperationAndDiscardItemWithCompletion(nil)
		                    } else {
		                        print("Update failed, reverting to server's copy")
		                        opError.cancelOperationAndUpdateItem(opError.serverItem!, completion: nil)
		                    }
		                }
		            }
		        }
		        self.refreshControl?.endRefreshing()
		    }
		} 
	
	
在 Objective-C 版本的 `syncData` 中，首先在同步上下文中调用 **pushWithCompletion**。此方法是 `MSSyncContext` 的成员（不是同步表本身），因为它将更改推送到所有表。只有已在本地以某种方式修改（通过 CUD 操作来完成）的记录才会发送到服务器。然后调用 **pullData** 帮助程序，该帮助程序调用 **MSSyncTable.pullWithQuery** 来检索远程数据并将其存储在本地数据库中。
	
在 Swift 版本中，因为推送操作不是必需的，所以没有调用 **pushWithCompletion**。如果同步上下文中正在进行推送操作的表存在任何挂起的更改，则提取始终会先发出推送。但是，如果有多个同步表，则最好是显式调用推送，以确保所有内容在相关表中保持一致。

在 Objective-C 和 Swift 版本中，均可以使用 **pullWithQuery** 方法指定查询，筛选想要检索的记录。在本示例中，查询检索远程 `TodoItem` 表中的所有记录。

**pullWithQuery** 的第二个参数是用于*增量同步*的查询 ID。增量同步只会使用记录的 `UpdatedAt` 时间戳（在本地存储中称为 `updatedAt`）检索自上次同步以来修改的记录。 查询 ID 应对于你的应用程序中的每个逻辑查询都是唯一的描述性字符串。若不进行增量同步，请将 `nil` 作为查询 ID 传递。此方法的效率可能较低，因为它检索每个提取操作的所有记录。

在修改或添加数据、用户执行刷新手势和启动时，Objective-C 应用将进行同步。

当用户执行刷新手势和启动时，Swift 应用将进行同步。

由于每当修改数据 (Objective-C) 或启动应用（Objective-C 和 Swift）时应用就会同步，因此，应用假设用户已联机。在后面的章节中，我们将更新应用，以便用户即使在脱机时也能进行编辑。

## <a name="review-core-data"></a>查看 Core Data 模型
在使用核心数据脱机存储时，必须在数据模型中定义特定的表和字段。示例应用已包含具有正确格式的数据模型。本节我们将逐步介绍这些表及其用法。

打开 **QSDataModel.xcdatamodeld**。已定义四个表，其中三个由 SDK 使用，还有一个供待办事项本身使用：
  * MS\_TableOperations：用于跟踪需要与服务器同步的项。
  * MS\_TableOperationErrors：用于跟踪脱机同步期间发生的任何错误。
  * MS\_TableConfig：用于跟踪所有提取操作最后一次同步操作的上次更新时间。
  * TodoItem：存储待办事项。系统列 **createdAt**、**updatedAt** 和 **version** 是可选的系统属性。

> [AZURE.NOTE]
> 移动应用 SDK 保留以“**``**”开头的列名称。请不要在系统列以外的其他列中使用此前缀。否则，列名称会在使用远程后端时被修改。
>
>

当使用脱机同步功能时，请定义三个系统表和一个数据表。

### 系统表

**MS\_TableOperations** 

![MS\_TableOperations 表属性][defining-core-data-tableoperations-entity]

| 属性 | 类型 |
|----------- |   ------    |
| id | Integer 64 |
| itemId | 字符串 |
| properties | 二进制数据 |
| 表 | 字符串 |
| tableKind | 16 位整数 |

<br> **MS\_TableOperationErrors** 

![MS\_TableOperationErrors 表属性][defining-core-data-tableoperationerrors-entity]  

| 属性 | 类型 |
|----------- |   ------    |
| id | 字符串 |
| operationId | 64 位整数 |
| 属性 | 二进制数据 |
| tableKind | 16 位整数 |

<br>**MS\_TableConfig**

![][defining-core-data-tableconfig-entity]

| 属性 | 类型 |
|----------- |   ------    |
| id | 字符串 |
| key | 字符串 |
| keyType | 64 位整数 |
| 表 | 字符串 |
| value | 字符串 |

### 数据表

**TodoItem**

| 属性 | 类型 | 注意 |
|-----------   |  ------ | -------------------------------------------------------|
| id | 字符串（标记为必需） | 远程存储中的主键 |
| complete | 布尔 | todo 项字段 |
| text | 字符串 | todo 项字段 |
| createdAt | 日期 | （可选）映射到 createdAt 系统属性 |
| updatedAt | 日期 | （可选）映射到 updatedAt 系统属性 |
| 版本 | 字符串 | （可选）用于检测冲突，映射到版本 |

## <a name="setup-sync"></a>更改应用的同步行为
本节将修改应用，使其在启动应用或插入和更新项时不进行同步。仅在按下刷新手势按钮时进行同步。

**Objective-C**：

1. 在 **QSTodoListViewController.m** 中更改 **viewDidLoad** 方法，删除方法末尾的 `[self refresh]` 调用。现在数据不会在启动应用时与服务器进行同步，而是与本地存储的内容进行同步。
2. 在 **QSTodoService.m** 中修改 `addItem` 的定义，使其不会在插入项后同步。删除 `self syncData` 块并将它替换为以下内容：

            if (completion != nil) {
                dispatch_async(dispatch_get_main_queue(), completion);
            }

3. 修改前面提到的 `completeItem` 的定义。删除 `self syncData` 块并将它替换为以下内容：

            if (completion != nil) {
                dispatch_async(dispatch_get_main_queue(), completion);
            }

**Swift**：

在 **ToDoTableViewController.swift** 中的 `viewDidLoad` 内，注释掉以下两行，以便在应用启动时停止同步。在编写本文时，如果有人添加或完成某个项，Swift Todo 应用不会更新服务。仅在应用启动时更新服务。

		self.refreshControl?.beginRefreshing()
		self.onRefresh(self.refreshControl)


## <a name="test-app"></a>测试应用程序
在本节中，将连接到无效的 URL，以模拟脱机场景。添加数据项时，数据项将保存在本地核心数据存储中，而不与移动应用后端进行同步。

1. 将 **QSTodoService.m** 中的移动应用 URL 更改为无效 URL，然后再次运行该应用：

	**Objective C**：在 QSTodoService.m 中：
	
        	self.client = [MSClient clientWithApplicationURLString:@"https://sitename.chinacloudsites.cn.fail"];
	
	**Swift**：在 ToDoTableViewController.swift 中：

		let client = MSClient(applicationURLString: "https://sitename.chinacloudsites.cn.fail")

2. 添加一些待办事项。退出模拟器（或强行关闭应用），然后重新启动。验证更改是否已保存。

3. 查看远程 **TodoItem** 表的内容：

    + 对于 Node.js 后端，请转到 [Azure 门户预览](https://portal.azure.cn/)，在移动应用后端中单击“简易表”>“TodoItem”。
   	+ 对于 .NET 后端，请使用 SQL 工具（如 SQL Server Management Studio）或 REST 客户端（如 Fiddler 或 Poistman）。

4. 验证新项是否*未*同步到服务器。

5. 将 **QSTodoService.m** 中的 URL 更改回正确的 URL，然后重新运行应用。

6. 通过下拉项列表来执行刷新手势。此时将显示进度盘。

7. 再次查看 **TodoItem** 数据。现在应显示新项和已更改的待办事项。

## 摘要
为了支持脱机同步功能，我们使用 `MSSyncTable` 接口，并使用本地存储初始化 `MSClient.syncContext`。在这种情况下，本地存储是基于核心数据的数据库。

使用核心数据本地存储时，必须使用[正确的系统属性](#review-core-data)定义多个表。

移动应用的普通创建、读取、更新和删除 (CRUD) 操作执行起来就像此应用仍处于连接状态一样，但所有操作都针对本地存储进行。

将本地存储与服务器进行同步时，使用 **MSSyncTable.pullWithQuery** 方法。


<!-- URLs. -->



[创建 iOS 应用]: /documentation/articles/app-service-mobile-ios-get-started/
[Offline Data Sync in Azure Mobile Apps]: /documentation/articles/app-service-mobile-offline-data-sync/

[defining-core-data-tableoperationerrors-entity]: ./media/app-service-mobile-ios-get-started-offline-data/defining-core-data-tableoperationerrors-entity.png
[defining-core-data-tableoperations-entity]: ./media/app-service-mobile-ios-get-started-offline-data/defining-core-data-tableoperations-entity.png
[defining-core-data-tableconfig-entity]: ./media/app-service-mobile-ios-get-started-offline-data/defining-core-data-tableconfig-entity.png
[defining-core-data-todoitem-entity]: ./media/app-service-mobile-ios-get-started-offline-data/defining-core-data-todoitem-entity.png

<!---HONumber=Mooncake_0313_2017-->
<!--Update_Description:update wording and link references-->
