<properties
	pageTitle="移动服务中的脱机数据同步入门 (iOS) | Azure"
	description="了解如何在 iOS 应用程序中使用 Azure 移动服务缓存和同步脱机数据"
	documentationCenter="ios"
	authors="krisragh"
	manager="erikre"
	editor=""
	services="mobile-services"/>

<tags
	ms.service="mobile-services"
	ms.date="03/09/2016"
	wacn.date="04/11/2016"/>

#  移动服务中的脱机数据同步入门

[AZURE.INCLUDE [mobile-services-selector-offline](../includes/mobile-services-selector-offline.md)]

借助脱机同步，即使在没有网络连接的情况下，你也可以查看、添加或修改移动应用中的数据。在本程中，你将了解应用如何在本地脱机数据库中自动存储更改，并在重新联机时同步这些更改。

脱机同步具有几个优点：

* 通过在设备上本地缓存服务器数据来提高应用响应能力
* 使应用可灵活应对间歇性网络连接
* 即使连接状态很差或者根本没有连接，也能让你创建和修改数据
* 跨多个设备同步数据
* 在两个设备修改同一条记录时检测冲突

> [AZURE.NOTE] 若要完成本教程，你需要一个 Azure 帐户。如果你没有帐户，可以注册 Azure 试用版并获取[免费的移动服务，即使在试用期结束之后仍可继续使用这些服务](/home/features/mobile-services/pricing/)。有关详细信息，请参阅 [Azure 试用](http://www.azure.cn/pricing/1rmb-trial/)。

本教程是在[移动服务快速入门教程]的基础之上制作的，所以必须先完成该教程。首先，让我们回顾“快速入门”中与脱机同步相关的代码。

##  <a name="review-sync"></a>回顾移动服务同步代码

Azure 移动服务脱机同步允许最终用户在无法访问网络时与本地数据库交互。若要在应用中使用这些功能，你可以初始化 `MSClient` 的同步上下文，并引用本机存储。然后，通过 `MSSyncTable` 接口引用你的表。

* 在 **QSTodoService.m** 中，请注意成员 `syncTable` 的类型是 `MSSyncTable`。脱机同步使用此类型而不是 `MSTable`。使用同步表时，所有操作将会转到本地存储，而且只会与具有显式推送和提取操作的远程服务同步。

```
		@property (nonatomic, strong)   MSSyncTable *syncTable;
```

若要获取对同步表的引用，请使用 `syncTableWithName` 方法。若要删除脱机同步功能，请改用 `tableWithName`。

* 在 **QSTodoService.m** 中，执行表操作之前，本地存储将在 `QSTodoService.init` 中初始化：

```
		MSCoreDataStore *store = [[MSCoreDataStore alloc] initWithManagedObjectContext:context];
		self.client.syncContext = [[MSSyncContext alloc] initWithDelegate:nil dataSource:store callback:nil];
```

这会使用 `MSCoreDataStore` 接口创建本地存储。你可以通过实现 `MSSyncContextDataSource` 协议来提供不同的本地存储。

`initWithDelegate` 的第一个参数指定冲突处理程序，但由于我们已传递 `nil`，因此默认的冲突处理程序将在发生任何冲突时失败。有关如何实现自定义冲突处理程序的详细信息，请参阅[使用移动服务脱机支持处理冲突]。

* 在 **QSTodoService.m** 中，`syncData` 先推送新的更改，然后调用 `pullData` 从远程服务获取数据。在 `syncData` 中，我们先对同步上下文调用 `pushWithCompletion`。此方法是 `MSSyncContext` 的成员，而不是异步表本身，因为它会将更改推送到所有表。只有以某种方式在本地上修改过的记录（通过创建、更新或删除操作）才会发送到服务器。在 `syncData` 结束时，调用帮助器 `pullData`。

在此示例中，推送操作并非绝对必要。如果同步上下文中正在进行推送操作的表存在待定的更改，则提取始终会先发出推送。但是，如果你有多个同步表，请显式调用推送，使表之间保持一致。

```
      -(void)syncData:(QSCompletionBlock)completion
        {
          // push all changes in the sync context, then pull new data
          [self.client.syncContext pushWithCompletion:^(NSError *error) {
              [self logErrorIfNotNil:error];
              [self pullData:completion];
          }];
        }

```

* 接下来，在 **QSTodoService.m** 中，`pullData` 将获取与查询匹配的新数据。`pullData` 将调用 `MSSyncTable.pullWithQuery` 以检索远程数据，并将数据存储在本地。`pullWithQuery` 也允许你指定查询以筛选你要检索的记录。在此示例中，查询只会检索远程 `TodoItem` 表中的所有记录。

`pullWithQuery` 的第二个参数是增量同步的查询 ID。增量同步只会使用记录的 `UpdatedAt` 时间戳（在本地存储中称为 `ms_updatedAt`）检索自上次同步以来修改的记录。对于应用中的每个逻辑查询而言，查询 ID 是唯一的描述性字符串。若选择不要增量同步，请传递 `nil` 作为查询 ID。这会降低效率，因为它会检索每个推送操作的所有记录。

```
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
```


>[AZURE.NOTE] 若要从设备本地存储区中删除已在移动设备数据库中删除的记录，应启用“[软删除]”。否则，你的应用程序应定期调用 `MSSyncTable.purgeWithQuery` 以清除本地存储。


* 在 **QSTodoService.m** 中，`addItem` 和 `completeItem` 方法会在修改数据后调用 `syncData`。在 **QSTodoListViewController.m** 中，`refresh` 方法也会调用 `syncData`，使 UI 在每次刷新和启动时（**init** 调用 `refresh`）显示最新数据。

因为每当你修改数据时，应用就会调用 `syncData`，所以无论你何时在应用中编辑数据，应用都会假设你已联机。

##  <a name="review-core-data"></a>了解核心数据模型

在使用核心数据脱机存储时，你需要在数据模型中定义特定的表和字段。示例应用已包含具有正确格式的数据模型。在本部分中，我们会逐步介绍这些表及其用法。

- 打开 **QSDataModel.xcdatamodeld**。已定义了四个表，其中三个由 SDK 使用，一个供 todo 项本身使用：

      * MS\_TableOperations：用于跟踪要与服务器同步的项
      * MS\_TableOperationErrors：用于跟踪脱机同步期间发生的错误
      * MS\_TableConfig：用于跟踪所有提取操作最后一次同步操作的上次更新时间
      * TodoItem：用于储存 todo 项。系统列 **ms\_createdAt**、**ms\_updatedAt** 和 **ms\_version** 是可选的系统属性。

>[AZURE.NOTE]移动服务 SDK 会保留以“**`ms_`**”开头的列名称。请不要在系统列以外的项中使用此前缀。否则，列名称会在使用远程服务时被修改。

- 使用脱机同步功能时，必须先定义系统表，如下所示。

    ### 系统表

    #### MS\_TableOperations

    | 属性 | 类型 |
    |-------------- |   ------    |
    | ID（必需） | Integer 64 |
    | itemId | String |
    | properties | 二进制数据 |
    | table | String |
    | tableKind | 16 位整数 |

    #### MS\_TableOperationErrors

    | 属性 | 类型 |
    |-------------- | ----------  |
    | ID（必需） | 字符串 |
    | operationId | Integer 64 |
    | 属性 | 二进制数据 |
    | tableKind | 16 位整数 |

    #### MS\_TableConfig


    | 属性 | 类型 |
    |-------------- | ----------  |
    | ID（必需） | String |
    | key | String |
    | keyType | 64 位整数 |
    | table | String |
    | value | String |

    ### 数据表

    #### TodoItem

    | 属性 | 类型 | 注意 |
    |-------------- |  ------ | -------------------------------------------------------|
    | ID（必需） | String | 远程存储中的主键（必需） |
    | complete | 布尔 | todo 项字段 |
    | text | String | todo 项字段 |
    | ms\_createdAt | 日期 | （可选）映射到 \_\_createdAt 系统属性 |
    | ms\_updatedAt | 日期 |（可选）映射到 \_\_updatedAt 系统属性 |
    | ms\_version | 字符串 |（可选）用于检测冲突，映射到 \_\_version |



## <a name="setup-sync"></a>更改应用的同步行为

在本部分中，你将要修改应用，使其不会在应用启动时或插入及更新项时同步，而只会在执行刷新手势时同步。

* 在 **QSTodoListViewController.m** 中，更改 `viewDidLoad` 以删除方法末尾的 `[self refresh]` 调用。现在，数据不会在应用启动时与服务器进行同步，而只会储存在本地。

* 在 **QSTodoService.m** 中修改 `addItem`，使其不会在插入项后同步。删除 `self syncData` 块并将它替换为以下内容：

```
        if (completion != nil) {
            dispatch_async(dispatch_get_main_queue(), completion);
        }
```

* 同样地，再次在 **QSTodoService.m** 中的 `completeItem` 内，删除 `self syncData` 的块并将它替换为以下内容：

```
        if (completion != nil) {
            dispatch_async(dispatch_get_main_queue(), completion);
                }
```

##  <a name="test-app"></a>测试应用程序

在本部分中，你将在模拟器中关闭 Wi-Fi 以创建脱机方案。在添加数据项时，数据项将保存在本地核心数据存储中，而不同步到移动服务。

1. 在 Mac 上关闭 Internet 连接。只是在 iOS 模拟器中关闭 WiFi 可能不起作用，因为模拟器仍可以使用 Mac 主机的 Internet 连接，因此只是关闭了计算机本身的 Internet。这会模拟脱机方案。

2. 添加一些 todo 项或完成某些项。退出模拟器（或强行关闭应用），然后重新启动。验证你的更改是否已保存。请注意，数据项仍会显示，因为它们都保留在本地核心数据存储中。

3. 查看远程 TodoItem 表的内容。验证新项是否未同步到服务器。

   - 对于 JavaScript 后端，请转到 [Azure 管理门户](http://manage.windowsazure.cn)，然后单击“数据”选项卡查看 `TodoItem` 表的内容。
   - 对于 .NET 后端，请使用 SQL 工具（如 SQL Server Management Studio）或 REST 客户端（如 Fiddler 或 Postman）查看表内容。

4. 在 iOS 模拟器中打开 Wi-Fi。接下来，通过拉下项列表来执行刷新手势。你将看到旋转进度条和“正在同步...”文字。

5. 再次查看 TodoItem 数据。新的和更改的 TodoItem 现在应会出现。

##  摘要

为了支持移动服务的脱机功能，你使用了 `MSSyncTable` 接口，并使用本地存储初始化了 `MSClient.syncContext`。在这种情况下，本地存储是基于核心数据的数据库。

使用核心数据本地存储时，你使用[正确的系统属性][Review the Core Data model]定义了多个表。移动服务的一般操作在进行时如同应用仍处于连接状态，但所有的操作都针对本地存储执行。

为了与服务器同步本地存储，你使用了 `MSSyncTable.pullWithQuery` 和 `MSClient.syncContext.pushWithCompletion`：

		* 为了将更改推送到服务器，你调用了 `pushWithCompletion`。此方法在 `MSSyncContext` 中而不是在同步表中，因为它将在所有表上推送更改。只有以某种方式在本地修改（通过 CUD 操作）的记录才会发送到服务器。

		* 为了将数据从服务器上的表拉取到应用，你调用了 `MSSyncTable.pullWithQuery`。拉取时始终先发出推送操作。这是为了确保本地存储中的所有表以及关系都保持一致。可以通过自定义 `query` 参数，使用 `pullWithQuery` 筛选客户端上存储的数据。

##  后续步骤

* [使用移动服务脱机支持处理冲突]

* [使用移动服务中的软删除][Soft Delete]

##  其他资源

* [云覆盖：Azure 移动服务中的脱机同步]


<!-- URLs. -->

[Get the sample app]: #get-app
[Review the Core Data model]: #review-core-data
[Review the Mobile Services sync code]: #review-sync
[Change the sync behavior of the app]: #setup-sync
[Test the app]: #test-app

[core-data-1]: ./media/mobile-services-ios-get-started-offline-data/core-data-1.png
[core-data-2]: ./media/mobile-services-ios-get-started-offline-data/core-data-2.png
[core-data-3]: ./media/mobile-services-ios-get-started-offline-data/core-data-3.png
[defining-core-data-main-screen]: ./media/mobile-services-ios-get-started-offline-data/defining-core-data-main-screen.png
[defining-core-data-model-editor]: ./media/mobile-services-ios-get-started-offline-data/defining-core-data-model-editor.png
[defining-core-data-tableoperationerrors-entity]: ./media/mobile-services-ios-get-started-offline-data/defining-core-data-tableoperationerrors-entity.png
[defining-core-data-tableoperations-entity]: ./media/mobile-services-ios-get-started-offline-data/defining-core-data-tableoperations-entity.png
[defining-core-data-tableconfig-entity]: ./media/mobile-services-ios-get-started-offline-data/defining-core-data-tableconfig-entity.png
[defining-core-data-todoitem-entity]: ./media/mobile-services-ios-get-started-offline-data/defining-core-data-todoitem-entity.png
[update-framework-1]: ./media/mobile-services-ios-get-started-offline-data/update-framework-1.png
[update-framework-2]: ./media/mobile-services-ios-get-started-offline-data/update-framework-2.png

[Core Data Model Editor Help]: https://developer.apple.com/library/mac/recipes/xcode_help-core_data_modeling_tool/Articles/about_cd_modeling_tool.html
[Creating an Outlet Connection]: https://developer.apple.com/library/mac/recipes/xcode_help-interface_builder/articles-connections_bindings/CreatingOutlet.html
[Build a User Interface]: https://developer.apple.com/library/mac/documentation/ToolsLanguages/Conceptual/Xcode_Overview/Edit_User_Interfaces/edit_user_interface.html
[Adding a Segue Between Scenes in a Storyboard]: https://developer.apple.com/library/ios/recipes/xcode_help-IB_storyboard/chapters/StoryboardSegue.html#//apple_ref/doc/uid/TP40014225-CH25-SW1
[Adding a Scene to a Storyboard]: https://developer.apple.com/library/ios/recipes/xcode_help-IB_storyboard/chapters/StoryboardScene.html

[Core Data]: https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/CoreData/cdProgrammingGuide.html
[Download the preview SDK here]: http://aka.ms/Gc6fex
[How to use the Mobile Services client library for iOS]: /documentation/articles/mobile-services-ios-how-to-use-client-library/
[Offline iOS Sample]: https://github.com/Azure/mobile-services-samples/tree/master/TodoOffline/iOS/blog20140611
[Mobile Services sample repository on GitHub]: https://github.com/Azure/mobile-services-samples


[Get started with Mobile Services]: /documentation/articles/mobile-services-ios-get-started/
[使用移动服务脱机支持处理冲突]: /documentation/articles/mobile-services-ios-handling-conflicts-offline-data/
[Soft Delete]: /documentation/articles/mobile-services-using-soft-delete/
[软删除]: /documentation/articles/mobile-services-using-soft-delete/

[云覆盖：Azure 移动服务中的脱机同步]: http://channel9.msdn.com/Shows/Cloud+Cover/Episode-155-Offline-Storage-with-Donna-Malayeri
[Aazure Friday：Azure 移动服务中支持脱机的应用]: http://azure.microsoft.com/documentation/videos/azure-mobile-services-offline-enabled-apps-with-donna-malayeri/

[移动服务快速入门教程]: /documentation/articles/mobile-services-ios-get-started/
 

<!---HONumber=Mooncake_0215_2016-->