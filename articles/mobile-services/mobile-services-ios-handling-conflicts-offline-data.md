<properties 
	pageTitle="使用移动服务中的脱机数据处理冲突 (iOS) | 移动开发人员中心" 
	description="了解在 iOS 应用程序中同步脱机数据时如何使用 Azure 移动服务处理冲突" 
	documentationCenter="ios" 
	authors="krisragh" 
	manager="erikre"
	editor="" 
	services="mobile-services"/>

<tags 
	ms.service="mobile-services" 
	ms.date="03/09/2016"
	wacn.date="04/11/2016"/>


#  使用移动服务中的脱机数据处理冲突

[AZURE.INCLUDE [mobile-services-selector-offline-conflicts](../includes/mobile-services-selector-offline-conflicts.md)]

本主题演示在使用 Azure 移动服务的脱机功能时如何同步数据和处理冲突。本教程基于[脱机数据入门]教程编写。

>[AZURE.NOTE]若要完成本教程，你需要一个 Azure 帐户。如果你没有帐户，可以创建一个试用帐户，只需几分钟即可完成。有关详细信息，请参阅 <a href="/pricing/1rmb-trial/?WT.mc_id=AE564AB28" target="_blank">Azure 试用</a>。


## 下载 iOS 项目

对于本教程，从 Github 下载[已更新的 Xcode 项目](https://github.com/Azure/mobile-services-samples/tree/master/TodoOffline/iOS)。我们已使用[脱机数据入门]教程末尾提供的 Xcode 项目作为起点，然后对其进行更新以允许编辑项目。我们还添加了支持的类和方法，以便我们可以在下一节中添加冲突处理程序。

在本教程结尾处，如果你在两个手机上运行此应用，在这两个手机上本地更改相同项目，并将更改推送回服务器，则会允许每个手机上的用户选择要保留的版本：
  * 保留客户端版本（这会覆盖服务器上的版本），
  * 保留服务器版本（这会更新客户端本地表），或者
  * 两个版本都不保留（取消推送，让操作挂起）。

现在，让我们添加冲突处理程序以启用此功能。

## <a name="add-conflict-handling"></a>将冲突处理程序添加到 Todo 列表视图控制器

1. 在 **QSTodoListViewController.m** 中，编辑 **viewDidLoad**。将对 **defaultService** 的调用替换为对 **defaultServiceWithDelegate** 的调用：

        self.todoService = [QSTodoService defaultServiceWithDelegate:self];

2. 在 **QSTodoListViewController.h** 中，将 **&lt;MSSyncContextDelegate&gt;** 添加到接口声明，以便实现 **MSSyncContextDelegate** 协议。

        @interface QSTodoListViewController : UITableViewController<MSSyncContextDelegate, NSFetchedResultsControllerDelegate>

3. 在 **QSTodoListViewController.m** 的顶部添加以下 import 语句：

        #import "QSUIAlertViewWithBlock.h"

4. 最后，让我们将以下两项操作添加到 **QSTodoListViewController.m**，以便使用此帮助器类，并提示用户以三种方式之一协调冲突。

        - (void)tableOperation:(MSTableOperation *)operation onComplete:(MSSyncItemBlock)completion
        {
            [self doOperation:operation complete:completion];
        }

        -(void)doOperation:(MSTableOperation *)operation complete:(MSSyncItemBlock)completion
        {
            [operation executeWithCompletion:^(NSDictionary *item, NSError *error) {

                NSDictionary *serverItem = [error.userInfo objectForKey:MSErrorServerItemKey];

                if (error.code == MSErrorPreconditionFailed) {
                    QSUIAlertViewWithBlock *alert = [[QSUIAlertViewWithBlock alloc] initWithCallback:^(NSInteger buttonIndex) {
                        if (buttonIndex == 1) { // Client
                            NSMutableDictionary *adjustedItem = [operation.item mutableCopy];

                            [adjustedItem setValue:[serverItem objectForKey:MSSystemColumnVersion] forKey:MSSystemColumnVersion];
                            operation.item = adjustedItem;

                            [self doOperation:operation complete:completion];
                            return;

                        } else if (buttonIndex == 2) { // Server
                            NSDictionary *serverItem = [error.userInfo objectForKey:MSErrorServerItemKey];
                            completion(serverItem, nil);
                        } else { // Cancel
                            [operation cancelPush];
                            completion(nil, error);
                        }
                    }];

                    NSString *message = [NSString stringWithFormat:@"Client value: %@\nServer value: %@", operation.item[@"text"], serverItem[@"text"]];

                    [alert showAlertWithTitle:@"Server Conflict"
                                      message:message
                            cancelButtonTitle:@"Cancel"
                            otherButtonTitles:[NSArray arrayWithObjects:@"Use Client", @"Use Server", nil]];
                } else {
                    completion(item, error);
                }
            }];
        }

## <a name="test-app"></a>测试应用程序

让我们通过冲突测试应用程序！ 在同时运行的应用程序的两个不同实例中编辑同一项，或者使用应用程序和 REST 客户端进行编辑。

通过从顶部拖动在应用程序实例中执行刷新手势。现在，你将看到协调冲突的提示：

![][conflict-ui]

<!-- URLs. -->

[Update the App Project to Allow Editing]: #update-app
[Update Todo List View Controller]: #update-list-view
[Add Todo Item View Controller]: #add-view-controller
[Add Todo Item View Controller and Segue to Storyboard]: #add-segue
[Add Item Details to Todo Item View Controller]: #add-item-details
[Add Support for Saving Edits]: #saving-edits
[Conflict Handling Problem]: #conflict-handling-problem
[Update QSTodoService to Support Conflict Handling]: #service-add-conflict-handling
[Add UI Alert View Helper to Support Conflict Handling]: #add-alert-view
[Add Conflict Handler to Todo List View Controller]: #add-conflict-handling
[Test the App]: #test-app


[add-todo-item-view-controller-3]: ./media/mobile-services-ios-handling-conflicts-offline-data/add-todo-item-view-controller-3.png
[add-todo-item-view-controller-4]: ./media/mobile-services-ios-handling-conflicts-offline-data/add-todo-item-view-controller-4.png
[add-todo-item-view-controller-5]: ./media/mobile-services-ios-handling-conflicts-offline-data/add-todo-item-view-controller-5.png
[add-todo-item-view-controller-6]: ./media/mobile-services-ios-handling-conflicts-offline-data/add-todo-item-view-controller-6.png
[todo-list-view-controller-add-segue]: ./media/mobile-services-ios-handling-conflicts-offline-data/todo-list-view-controller-add-segue.png
[update-todo-list-view-controller-2]: ./media/mobile-services-ios-handling-conflicts-offline-data/update-todo-list-view-controller-2.png
[conflict-handling-problem-1]: ./media/mobile-services-ios-handling-conflicts-offline-data/conflict-handling-problem-1.png
[conflict-ui]: ./media/mobile-services-ios-handling-conflicts-offline-data/conflict-ui.png


[Segmented Controls]: https://developer.apple.com/zh-cn/library/ios/documentation/UserExperience/Conceptual/UIKitUICatalog/UISegmentedControl.html
[Core Data Model Editor Help]: https://developer.apple.com/zh-cn/library/mac/recipes/xcode_help-core_data_modeling_tool/Articles/about_cd_modeling_tool.html
[Creating an Outlet Connection]: https://developer.apple.com/zh-cn/library/mac/recipes/xcode_help-interface_builder/articles-connections_bindings/CreatingOutlet.html
[Build a User Interface]: https://developer.apple.com/zh-cn/library/mac/documentation/ToolsLanguages/Conceptual/Xcode_Overview/Edit_User_Interfaces/edit_user_interface.html
[Adding a Segue Between Scenes in a Storyboard]: https://developer.apple.com/zh-cn/library/ios/recipes/xcode_help-IB_storyboard/chapters/StoryboardSegue.html#//apple_ref/doc/uid/TP40014225-CH25-SW1
[Adding a Scene to a Storyboard]: https://developer.apple.com/zh-cn/library/ios/recipes/xcode_help-IB_storyboard/chapters/StoryboardScene.html
[Core Data]: https://developer.apple.com/zh-cn/library/ios/documentation/Cocoa/Conceptual/CoreData/cdProgrammingGuide.html
[Download the preview SDK here]: http://aka.ms/Gc6fex
[How to use the Mobile Services client library for iOS]: /documentation/articles/mobile-services-ios-how-to-use-client-library/
[Getting Started Offline iOS Sample]: https://github.com/Azure/mobile-services-samples/tree/master/TodoOffline/iOS/blog20140611
[脱机数据入门]: /documentation/articles/mobile-services-ios-get-started-offline-data/
[Get started with Mobile Services]: /documentation/articles/mobile-services-ios-get-started/
 

<!---HONumber=Mooncake_0118_2016-->