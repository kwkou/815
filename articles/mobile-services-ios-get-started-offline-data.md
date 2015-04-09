<properties urlDisplayName="使用脱机数据" pageTitle="使用移动服务中的脱机数据同步 (iOS) | 移动开发人员中心" metaKeywords="" description="了解如何在 iOS 应用程序中使用 Azure 移动服务缓存和同步脱机数据" metaCanonical="" disqusComments="1" umbracoNaviHide="1" documentationCenter="Mobile" title="Using Offline data sync in Mobile Services" authors="krisragh" manager="dwrede" />

<tags ms.service="mobile-services" ms.workload="mobile" ms.tgt_pltfrm="mobile-ios" ms.devlang="objective-c" ms.topic="article" ms.date="10/10/2014" ms.author="krisragh" />

# 移动服务中的脱机数据同步入门


[WACOM.INCLUDE [mobile-services-selector-offline](../includes/mobile-services-selector-offline.md)]


本教程介绍 iOS 上移动服务的脱机同步功能，该功能允许开发人员编写即使最终用户无法访问网络也能使用的应用程序。

脱机同步有几种可能的用途：

* 通过在设备上本地缓存服务器数据来提高应用程序响应能力
* 使应用程序可灵活应对间歇性网络连接
* 即使在无法访问网络时也允许最终用户创建和修改数据，支持连接性很差或没有连接的方案
* 在多个设备之间同步数据并在两个设备修改同一条记录时检测冲突

本教程将演示如何更新[移动服务入门]教程中的应用程序，以支持 Azure 移动服务的脱机功能。随后，你将在断开连接的脱机情况下添加数据，将这些项目同步到联机数据库，然后登录到 Azure 管理门户，查看在运行应用程序时对数据所做的更改。

>[AZURE.NOTE] 若要完成本教程，你需要一个 Azure 帐户。如果你没有帐户，可以注册 Azure 试用。有关详细信息，请参阅 <a href="/zh-cn/pricing/1rmb-trial/?WT.mc_id=AE564AB28" target="_blank">Azure 免费试用</a>。

本教程旨在帮助你更好地了解如何使用移动服务通过 Azure 在 Windows 应用商店应用程序中存储和检索数据。因此，本主题指导你完成的许多步骤已在移动服务快速入门中代你完成。如果这是你第一次体验移动服务，请考虑首先完成[移动服务入门]教程。

>[AZURE.NOTE] 你可以跳过这些节，并跳转到下载"入门"项目版本的部分，该版本已具有脱机支持和本主题中所述的所有内容。若要下载启用了脱机支持的项目，请参阅[脱机 iOS 示例入门]。


本教程将指导你完成以下基本步骤：

1. [获取示例快速入门应用程序]
2. [下载预览版 SDK 并更新框架]
3. [设置核心数据]
4. [定义核心数据模型]
5. [初始化和使用同步表和同步上下文]
6. [测试应用程序]

## <a name="get-app"></a>获取示例快速入门应用程序

按照[移动服务入门]中的说明进行操作并下载快速入门项目。

## <a name="update-app"></a>下载预览版 SDK 并更新框架

1. 若要将脱机支持添加到我们的应用程序，让我们获取支持脱机同步的移动服务 iOS SDK 版本。由于我们将它作为预览版功能推出，因此它未包含在正式可下载的 SDK 中。[在此处下载预览版 SDK]。

2. 然后，从 Xcode 的项目中删除现有 **WindowsAzureMobileServices.framework** 引用，方法是选择该引用，单击**"编辑"**菜单，选择"移到垃圾桶"以真正删除这些文件。

      ![][update-framework-1]

3. 将新的预览版 SDK 的内容解压缩并替代旧版 SDK，将其拖放到新的 **WindowsAzureMobileServices.framework** SDK 上。确保选择了"将项目复制到目标组的文件夹(如果需要)"。

      ![][update-framework-2]


## <a name="setup-core-data"></a>设置核心数据

1. iOS 移动服务 SDK 允许你使用任何持久存储，前提是它符合 **MSSyncContextDataSource** 协议。该 SDK 中包含的是基于[核心数据]实现此协议的数据源。

2. 由于应用程序使用核心数据，请导航到"目标"-->"生成阶段"，并在"将二进制文件链接到库"下添加 **CoreData.framework**。

      ![][core-data-1]

      ![][core-data-2]

3. 我们要将核心数据添加到 Xcode 中尚不支持核心数据的现有项目中。因此，我们需要将附加的样板代码添加到项目的各个部分中。首先在 **QSAppDelegate.h** 中添加以下代码：

        #import <UIKit/UIKit.h> 
        #import <CoreData/CoreData.h> 

        @interface QSAppDelegate : UIResponder <UIApplicationDelegate> 

        @property (strong, nonatomic) UIWindow *window; 

        @property (readonly, strong, nonatomic) NSManagedObjectContext *managedObjectContext; 
        @property (readonly, strong, nonatomic) NSManagedObjectModel *managedObjectModel; 
        @property (readonly, strong, nonatomic) NSPersistentStoreCoordinator *persistentStoreCoordinator; 

        - (void)saveContext; 
        - (NSURL *)applicationDocumentsDirectory; 

        @end

4. 接下来，将 **QSAppDelegate.m** 的内容替换为以下代码。此代码几乎与在 Xcode 中创建新的应用程序并选中"使用核心数据"复选框时获得的代码相同，不同的是，在初始化 **_managedObjectContext** 时，你使用的是私有队列并发类型。进行此更改后，你已基本准备好使用核心数据了，但你还未使用它执行任何操作。

        #import "QSAppDelegate.h"

        @implementation QSAppDelegate

        @synthesize managedObjectContext = _managedObjectContext;
        @synthesize managedObjectModel = _managedObjectModel;
        @synthesize persistentStoreCoordinator = _persistentStoreCoordinator;

        - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
        {
            return YES;
        }

        - (void)saveContext
        {
            NSError *error = nil;
            NSManagedObjectContext *managedObjectContext = self.managedObjectContext;
            if (managedObjectContext != nil) {
                if ([managedObjectContext hasChanges] && ![managedObjectContext save:&error]) {
                    // Replace this implementation with code to handle the error appropriately.
                    // abort() causes the application to generate a crash log and terminate. You should not use this function in a shipping application, although it may be useful during development.
                    NSLog(@"Unresolved error %@, %@", error, [error userInfo]);
                    abort();
                }
            }
        }

        #pragma mark - Core Data stack

        // Returns the managed object context for the application.
        // If the context doesn't already exist, it is created and bound to the persistent store coordinator for the application.
        - (NSManagedObjectContext *)managedObjectContext
        {
            if (_managedObjectContext != nil) {
                return _managedObjectContext;
            }

            NSPersistentStoreCoordinator *coordinator = [self persistentStoreCoordinator];
            if (coordinator != nil) {
                _managedObjectContext = [[NSManagedObjectContext alloc] initWithConcurrencyType:NSPrivateQueueConcurrencyType];
                [_managedObjectContext setPersistentStoreCoordinator:coordinator];
            }
            return _managedObjectContext;
        }

        // Returns the managed object model for the application.
        // If the model doesn't already exist, it is created from the application's model.
        - (NSManagedObjectModel *)managedObjectModel
        {
            if (_managedObjectModel != nil) {
                return _managedObjectModel;
            }
            NSURL *modelURL = [[NSBundle mainBundle] URLForResource:@"QSTodoDataModel" withExtension:@"momd"];
            _managedObjectModel = [[NSManagedObjectModel alloc] initWithContentsOfURL:modelURL];
            return _managedObjectModel;
        }

        // Returns the persistent store coordinator for the application.
        // If the coordinator doesn't already exist, it is created and the application's store added to it.
        - (NSPersistentStoreCoordinator *)persistentStoreCoordinator
        {
            if (_persistentStoreCoordinator != nil) {
                return _persistentStoreCoordinator;
            }

            NSURL *storeURL = [[self applicationDocumentsDirectory] URLByAppendingPathComponent:@"qstodoitem.sqlite"];

            NSError *error = nil;
            _persistentStoreCoordinator = [[NSPersistentStoreCoordinator alloc] initWithManagedObjectModel:[self managedObjectModel]];
            if (![_persistentStoreCoordinator addPersistentStoreWithType:NSSQLiteStoreType configuration:nil URL:storeURL options:nil error:&error]) {
                /*
                 Replace this implementation with code to handle the error appropriately.

                 abort() causes the application to generate a crash log and terminate. You should not use this function in a shipping application, although it may be useful during development.

                 Typical reasons for an error here include:
                 * The persistent store is not accessible;
                 * The schema for the persistent store is incompatible with current managed object model.
                 Check the error message to determine what the actual problem was.

                 If the persistent store is not accessible, there is typically something wrong with the file path. Often, a file URL is pointing into the application's resources directory instead of a writeable directory.

                 If you encounter schema incompatibility errors during development, you can reduce their frequency by:
                 * Simply deleting the existing store:
                 [[NSFileManager defaultManager] removeItemAtURL:storeURL error:nil]

                 * Performing automatic lightweight migration by passing the following dictionary as the options parameter:
                 @{NSMigratePersistentStoresAutomaticallyOption:@YES, NSInferMappingModelAutomaticallyOption:@YES}

                 Lightweight migration will only work for a limited set of schema changes; consult "Core Data Model Versioning and Data Migration Programming Guide" for details.

                 */

                NSLog(@"Unresolved error %@, %@", error, [error userInfo]);
                abort();
            }

            return _persistentStoreCoordinator;
        }

        #pragma mark - Application's Documents directory

        // Returns the URL to the application's Documents directory.
        - (NSURL *)applicationDocumentsDirectory
        {
            return [[[NSFileManager defaultManager] URLsForDirectory:NSDocumentDirectory inDomains:NSUserDomainMask] lastObject];
        }

        @end

## <a name="defining-core-data"></a>定义核心数据模型

1. 让我们继续通过定义数据模型使用核心数据设置应用程序。我们不会开始直接使用此数据模型。首先，让我们定义核心数据模型或架构。若要开始，请单击"文件"->"新建文件"，然后在"核心数据"部分中选择"数据模型"。当系统提示你输入文件名时，使用 **QSTodoDataModel.xcdatamodeld**。

      ![][defining-core-data-main-screen]

2. 接下来，让我们定义所需的实际实体（表）。我们将使用核心数据模型编辑器创建三个表（实体）。若要了解详细信息，请参阅[核心数据模型编辑器帮助]。

  * TodoItem：用于存储项目本身
  * MS_TableOperations：用于跟踪需要与服务器同步的项目（要使脱机功能正常工作，此项是必需的）
  * MS_TableOperationErrors：用于跟踪在脱机同步过程中发生的任何错误（要使脱机功能正常工作，此项是必需的）

      ![][defining-core-data-model-editor]

3. 定义三个实体，如下所示。保存模型，并生成项目以确保一切正常。现在我们已完成将应用程序设置为使用核心数据，但应用程序尚未使用它。

      ![][defining-core-data-todoitem-entity]

      ![][defining-core-data-tableoperations-entity]

      ![][defining-core-data-tableoperationerrors-entity]


    **TodoItem**

    | 属性  |  类型   |
    |----------- |  ------ |
    | ID         | String  |
    | complete   | Boolean |
    | text       | String  |
    | ms_version | String  |

    **MS_TableOperations**

    | 属性  |    类型     |
    |----------- |   ------    |
    | ID         | Integer 64  |
    | properties | Binary Data |
    | itemId     | String      |
    | table      | String      |

    **MS_TableOperationErrors**

    | 属性  |    类型     |
    |----------- |   ------    |
    | ID         | String      |
    | properties | Binary Data |

## <a name="setup-sync"></a> 初始化和使用同步表和同步上下文

1. 若要开始缓存脱机数据，让我们将 **MSTable** 的用法替换为 **MSSyncTable** 以访问移动服务。与常规的 **MSTable** 不同，同步表就像是本地表，其中添加了将本地进行的更改推送到远程表，并在本地提取这些更改的功能。 

    在 **QSTodoService.m** 中，删除 **table** 属性的定义：

        @property (nonatomic, strong)   MSTable *table;

    添加新行以定义 **syncTable** 属性：

        @property (nonatomic, strong)   MSSyncTable *syncTable;

2. 在 **QSTodoService.m** 的顶部添加以下 import 语句：

        #import "QSAppDelegate.h"

3. 在 **QSTodoService.m** 中，删除 **init** 中的以下两行：

        // Create an MSTable instance to allow us to work with the TodoItem table
        self.table = [_client tableWithName:@"TodoItem"];

    Instead, add these two new lines in its place:

        // Create an MSSyncTable instance to allow us to work with the TodoItem table
        self.syncTable = [self.client syncTableWithName:@"TodoItem"];

4. 接下来，再次在 **QSTodoService.m** 中，让我们使用上述基于核心数据的数据存储实现来初始化 **MSClient** 中的同步上下文。该上下文负责跟踪哪些项已在本地发生更改，并在启动推送操作时将这些项发送到服务器。若要初始化上下文，我们需要数据源（协议的 **MSCoreDataStore** 实现）和可选的 **MSSyncContextDelegate** 实现。在你在上面插入的两行的正上方插入这些行：

        QSAppDelegate *delegate = (QSAppDelegate *)[[UIApplication sharedApplication] delegate];
        NSManagedObjectContext *context = delegate.managedObjectContext;
        MSCoreDataStore *store = [[MSCoreDataStore alloc] initWithManagedObjectContext:context];

        self.client.syncContext = [[MSSyncContext alloc] initWithDelegate:nil dataSource:store callback:nil];

5. 接下来，让我们更新 **QSTodoService.m** 中的操作以使用同步表而不是常规表。首先，将 **refreshDataOnSuccess** 替换为以下实现。此代码从服务中检索数据，因此让我们将其更新为使用同步表，让同步表只提取匹配条件的项目，并开始从本地同步表将数据加载到服务的 **items** 属性中。使用此代码，**refreshDataOnSuccess** 将从远程表将数据提取到本地（同步）表中。我们通常应只提取该表的子集，以确保不会使用可能不需要的信息让客户端超载。

    对于此操作和更后面的剩余操作，我们将对 **dispatch_async** 调用中 completion 块的调用包装到主线程中。当我们初始化同步上下文时，未传递回调参数，因此框架会创建默认串行队列，将所有 syncTable 操作的结果调度到后台线程中。在修改 UI 组件时，我们需要将代码调度回 UI 线程。

          -(void) refreshDataOnSuccess:(QSCompletionBlock)completion
          {
              NSPredicate * predicate = [NSPredicate predicateWithFormat:@"complete == NO"];
              MSQuery *query = [self.syncTable queryWithPredicate:predicate];

              [query orderByAscending:@"text"];
              [query readWithCompletion:^(MSQueryResult *result, NSError *error) {
                  [self logErrorIfNotNil:error];

                  self.items = [result.items mutableCopy];

                  // Let the caller know that we finished
                  dispatch_async(dispatch_get_main_queue(), ^{
                      completion();
                  });
              }];
          }

6. 接下来，按如下所示替换 **QSTodoService.m** 中的 **addItem**。进行此更改后，你将对操作排队，以便将更改推送到远程服务并使该更改对每个人可见：

        -(void)addItem:(NSDictionary *)item completion:(QSCompletionWithIndexBlock)completion
        {
            // Insert the item into the TodoItem table and add to the items array on completion
            [self.syncTable insert:item completion:^(NSDictionary *result, NSError *error)
             {
                 [self logErrorIfNotNil:error];

                 NSUInteger index = [items count];
                 [(NSMutableArray *)items insertObject:result atIndex:index];

                 // Let the caller know that we finished
                 dispatch_async(dispatch_get_main_queue(), ^{
                     completion(index);
                 });
             }];
        }

7. 按如下所示更新 **QSTodoService.m** 中的 **completeItem**。与 **MSTable** 不同，**MSSyncTable** 的"更新"操作的 completion 块没有已更新项。使用 **MSTable**，服务器将修改要更新项，并将该修改反映到客户端。使用 **MSSyncTable**，将不会修改已更新项，并且 completion 块没有参数。

        -(void) completeItem:(NSDictionary *)item completion:(QSCompletionWithIndexBlock)completion
        {
            // Cast the public items property to the mutable type (it was created as mutable)
            NSMutableArray *mutableItems = (NSMutableArray *) items;

            // Set the item to be complete (we need a mutable copy)
            NSMutableDictionary *mutable = [item mutableCopy];
            [mutable setObject:@YES forKey:@"complete"];

            // Replace the original in the items array
            NSUInteger index = [items indexOfObjectIdenticalTo:item];
            [mutableItems replaceObjectAtIndex:index withObject:item];

            // Update the item in the TodoItem table and remove from the items array on completion
            [self.syncTable update:mutable completion:^(NSError *error) {

                [self logErrorIfNotNil:error];

                NSUInteger index = [items indexOfObjectIdenticalTo:mutable];
                if (index != NSNotFound)
                {
                    [mutableItems removeObjectAtIndex:index];
                }

                // Let the caller know that we have finished
                dispatch_async(dispatch_get_main_queue(), ^{
                    completion(index);
                });

            }];
        }

8. 将 **syncData** 的以下操作声明添加到 **QSTodoService.h**：

        - (void)syncData:(QSCompletionBlock)completion;

     我们将添加此操作，以便使用远程更改更新同步表。请注意，我们需要提取 *all* Todo 项（不只是未完成的项），因为应用程序可能在本地具有已标记为完成的项。如果我们仅筛选未完成的项，则可能会在 UI 中存在实际上已在服务器上设为已完成的 Todo 项。

     将 **syncData** 的相应实现添加到 **QSTodoService.m**：

            -(void)syncData:(QSCompletionBlock)completion
            {
                MSQuery *query = [self.syncTable query];

                // Pulls data from the remote server into the local table.
                // We're pulling all items and filtering in refreshDataOnSuccess
                [self.syncTable pullWithQuery:query completion:^(NSError *error) {
                    [self logErrorIfNotNil:error];
                    [self refreshDataOnSuccess:completion];
                }];
            }

9. 返回到 **QSTodoListViewController.m** 中，将 **refresh** 的实现更改为调用 **syncData** 而不是 **refreshDataOnSuccess**：

        -(void) refresh
        {
            [self.refreshControl beginRefreshing];
            [self.todoService syncData:^
             {
                  [self.refreshControl endRefreshing];
                  [self.tableView reloadData];
             }];
        }

10. 再次在 **QSTodoListViewController.m** 中，将 **viewDidLoad** 操作末尾对 **[self refresh]** 的调用替换为以下代码：

        // load the local data, but don't pull from server
        [self.todoService refreshDataOnSuccess:^
         {
             [self.refreshControl endRefreshing];
             [self.tableView reloadData];
         }];

11. 现在，让我们实际在脱机情况下测试应用程序。将几个项添加到应用程序，然后访问 Azure 管理门户并查看应用程序的"数据"选项卡。你将看到尚未添加任何项。

12. 接下来，通过从顶部拖动对应用程序执行刷新笔势。然后，再次访问 Azure 管理门户并再次查看"数据"选项卡。现在，你将看到保存在云中的数据。你还可以在添加项或编辑项（如果应用程序已启用编辑项的功能）后关闭应用程序。当重新启动应用程序时，应用程序将与服务器同步，并保存所做的更改。

13. 当客户端在本地对项目执行一些更改时，这些更改将存储在要发送到服务器的同步上下文中。 *push*操作会将跟踪的更改发送到远程服务器，但在这里，我们没有对服务进行任何推送调用。但是， *before a pull is executed, any pending operations are generally pushed to the server*，因此推送仍会自动发生以避免发生冲突。这就是为什么没有在此应用程序中显式调用  *push* 的原因。

## <a name="test-app"></a>测试应用程序

最后，让我们在脱机情况下测试应用程序。在应用程序中添加几个项。然后转到门户并浏览数据（或者使用 PostMan 或 Fiddler 之类的网络工具直接查询表）。

你将看到这些项尚未添加到该服务。现在，通过从顶部拖动在应用程序中执行刷新笔势。你将看到数据现已保存在云中。你甚至可以在添加一些项后关闭该应用程序。当你重新启动该应用程序时，该应用程序将与服务器同步，并将保存你的更改。

## 后续步骤

* [使用移动服务脱机支持处理冲突]

<!-- URLs. -->

[获取示例快速入门应用程序]: #get-app
[下载预览版 SDK 并更新框架]: #update-app
[设置核心数据]: #setup-core-data
[定义核心数据模型]: #defining-core-data
[初始化和使用同步表和同步上下文]: #setup-sync
[测试应用程序]: #test-app

[core-data-1]: ./media/mobile-services-ios-get-started-offline-data/core-data-1.png
[core-data-2]: ./media/mobile-services-ios-get-started-offline-data/core-data-2.png
[core-data-3]: ./media/mobile-services-ios-get-started-offline-data/core-data-3.png
[defining-core-data-main-screen]: ./media/mobile-services-ios-get-started-offline-data/defining-core-data-main-screen.png
[defining-core-data-model-editor]: ./media/mobile-services-ios-get-started-offline-data/defining-core-data-model-editor.png
[defining-core-data-tableoperationerrors-entity]: ./media/mobile-services-ios-get-started-offline-data/defining-core-data-tableoperationerrors-entity.png
[defining-core-data-tableoperations-entity]: ./media/mobile-services-ios-get-started-offline-data/defining-core-data-tableoperations-entity.png
[defining-core-data-todoitem-entity]: ./media/mobile-services-ios-get-started-offline-data/defining-core-data-todoitem-entity.png
[update-framework-1]: ./media/mobile-services-ios-get-started-offline-data/update-framework-1.png
[update-framework-2]: ./media/mobile-services-ios-get-started-offline-data/update-framework-2.png




[核心数据模型编辑器帮助]: https://developer.apple.com/library/mac/recipes/xcode_help-core_data_modeling_tool/Articles/about_cd_modeling_tool.html
[创建 Outlet 连接]: https://developer.apple.com/library/mac/recipes/xcode_help-interface_builder/articles-connections_bindings/CreatingOutlet.html
[构建用户界面]: https://developer.apple.com/library/mac/documentation/ToolsLanguages/Conceptual/Xcode_Overview/Edit_User_Interfaces/edit_user_interface.html
[在情节提要中的场景之间添加 Segue]: https://developer.apple.com/library/ios/recipes/xcode_help-IB_storyboard/chapters/StoryboardSegue.html#//apple_ref/doc/uid/TP40014225-CH25-SW1
[将场景添加到情节提要]: https://developer.apple.com/library/ios/recipes/xcode_help-IB_storyboard/chapters/StoryboardScene.html

[核心数据]: https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/CoreData/cdProgrammingGuide.html
[在此处下载预览版 SDK]: http://aka.ms/Gc6fex
[如何使用适用于 iOS 的移动服务客户端库]: /zh-cn/documentation/articles/mobile-services-ios-how-to-use-client-library/
[脱机 iOS 示例入门]: https://github.com/Azure/mobile-services-samples/tree/master/TodoOffline/iOS/blog20140611


[移动服务入门]: /zh-cn/documentation/articles/mobile-services-ios-get-started/
[数据处理入门]: /zh-cn/documentation/articles/mobile-services-ios-get-started-data/
[使用移动服务脱机支持处理冲突]: /zh-cn/documentation/articles/mobile-services-ios-handling-conflicts-offline-data/
