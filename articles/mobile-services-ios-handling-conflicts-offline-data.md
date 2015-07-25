<properties 
	pageTitle="使用移动服务中的脱机数据处理冲突 (iOS) | 移动开发人员中心" 
	description="了解在 iOS 应用程序中同步脱机数据时如何使用 Azure 移动服务处理冲突" 
	documentationCenter="ios" 
	authors="krisragh" 
	manager="dwrede" 
	editor="" 
	services="mobile-services"/>

<tags 
	ms.service="mobile-services" 
	ms.date="04/16/2015" 
	wacn.date="07/25/2015"/>


#  使用移动服务中的脱机数据处理冲突

[AZURE.INCLUDE [mobile-services-selector-offline-conflicts](../includes/mobile-services-selector-offline-conflicts.md)]

本主题演示在使用 Azure 移动服务的脱机功能时如何同步数据和处理冲突。本教程以前一教程[脱机数据处理入门]中的步骤和示例应用程序为基础。在开始本教程之前，必须先完成[脱机数据处理入门]。

>[AZURE.NOTE]若要完成本教程，你需要一个 Azure 帐户。如果你没有帐户，可以创建一个试用帐户，只需几分钟即可完成。有关详细信息，请参阅 <a href="http://www.windowsazure.cn/pricing/1rmb-trial">Azure 试用</a>。

本教程将指导你完成以下基本步骤：

1. [更新应用程序项目以允许编辑]
2. [更新 Todo 列表视图控制器]
3. [添加 Todo 项视图控制器]
4. [将 Todo 项视图控制器和 Segue 添加到情节提要]
5. [将项详细信息添加到 Todo 项视图控制器]
6. [添加对保存编辑的支持]
7. [冲突处理问题]
8. [更新 QSTodoService 以支持冲突处理]
9. [添加 UI 警报视图帮助器以支持冲突处理]
10. [将冲突处理程序添加到 Todo 列表视图控制器]
11. [测试应用程序]

##  完成“脱机入门”教程

按照[脱机数据处理入门]教程中的说明进行操作，并完成该项目。我们将使用该教程中完成的项目作为本教程的起点。

##  <a name="update-app"></a>更新应用程序项目以允许编辑

让我们更新[脱机数据处理入门]中完成的项目以允许编辑项。目前，如果你在两个手机上运行此同一应用程序，在这两个手机上本地更改同一项，并将更改推送回服务器，则会因冲突而失败。

使用 SDK 中的脱机同步功能，你可以通过代码处理此类冲突，并且你可以动态决定如何处理冲突项。更改快速入门项目，可以让我们体验此功能。

###  <a name="update-list-view"></a>更新 Todo 列表视图控制器

1. 在 Xcode 项目导航器中选择 **MainStoryboard_iPhone.storyboard**，然后选择“Todo 列表视图控制器”。选择表视图单元格，然后将其附件模式设为“泄露指示器”。泄漏指示器向用户指示，如果用户点击关联的表视图控制器，则将显示一个新视图。泄露指示器不会生成任何事件。

      ![][update-todo-list-view-controller-2]

2. 在 **TodoListViewController.m** 中，将以下操作及其内容一起删除。我们不需要以下内容：

        -(NSString *)tableView:(UITableView *)tableView titleForDeleteConfirmationButtonForRowAtIndexPath:(NSIndexPath *)indexPath

        -(UITableViewCellEditingStyle)tableView:(UITableView *)tableView editingStyleForRowAtIndexPath:(NSIndexPath *)indexPath

        -(void)tableView:(UITableView *)tableView commitEditingStyle:(UITableViewCellEditingStyle)editingStyle
         forRowAtIndexPath:(NSIndexPath *)indexPath

###  <a name="add-view-controller"></a>添加 Todo 项视图控制器

1. 创建派生自 **UIViewController**、名为 **QSItemViewController** 的新 Cocoa Touch 类。

2. 在 **QSItemViewController.h** 中添加以下类型定义：

        typedef void (^ItemEditCompletionBlock) (NSDictionary *editedItem);

3. 在 **QSItemViewController.h** 中，添加一个属性用于保存要修改的项，并为用户在详细信息视图中按下返回按钮后调用的回调添加一个属性：

        @property (nonatomic, weak) NSMutableDictionary *item;
        @property (nonatomic, strong) ItemEditCompletionBlock editCompleteBlock;

4. 在 **QSItemViewController.m** 中，为我们将编辑的 ToDo 项的两个字段添加两个私有属性（完成状态以及 ToDo 项本身的文本）：

        @interface QSItemViewController ()

        @property (nonatomic, strong) IBOutlet UITextField *itemText;
        @property (nonatomic, strong) IBOutlet UISegmentedControl *itemComplete;

        @end

5. 在 **QSItemViewController.m** 中，将 **viewDidLoad** 的存根实现更新为以下代码：

        - (void)viewDidLoad
        {
            [super viewDidLoad];

            UINavigationItem *nav = [self navigationItem];
            [nav setTitle:@"Todo Item"];

            NSDictionary *theItem = [self item];
            [self.itemText setText:[theItem objectForKey:@"text"]];

            BOOL isComplete = [[theItem objectForKey:@"complete"] boolValue];
            [self.itemComplete setSelectedSegmentIndex:(isComplete ? 0 : 1)];

            [self.itemComplete addTarget:self
                                  action:@selector(completedValueChanged:)
                        forControlEvents:UIControlEventValueChanged];
        }

6. 在 **QSItemViewController.m** 中，添加四个附加方法，用于处理已编辑的控制事件：

        - (BOOL)textFieldShouldEndEditing:(UITextField *)textField {
            [textField resignFirstResponder];
            return YES;
        }

        - (BOOL)textFieldShouldReturn:(UITextField *)textField {
            [textField resignFirstResponder];
            return YES;
        }


        - (void)completedValueChanged:(id)sender {
            [[self view] endEditing:YES];
        }

7. 在 **QSItemViewController** 中，还要添加以下方法，当用户在导航栏中按下“返回”按钮时，将调用这些方法。可对其他事件调用该方法，因此我们要先检查父视图。如果项发生了任何更改，则会修改 **self.item** 并调用 **editCompleteBlock** 回调：

        - (void)didMoveToParentViewController:(UIViewController *)parent
        {
            if (![parent isEqual:self.parentViewController]) {
                NSNumber *completeValue = [NSNumber numberWithBool:self.itemComplete.selectedSegmentIndex == 0];
                
                Boolean changed =
                    [self.item valueForKey:@"text"] != [self.itemText text] ||
                    [self.item valueForKey:@"complete"] != completeValue;
                
                if (changed) {
            [self.item setValue:[self.itemText text] forKey:@"text"];
                    [self.item setValue:completeValue forKey:@"complete"];
                    
                    self.editCompleteBlock(self.item);
                }
            }
        }

###  <a name="add-segue"></a>将 Todo 项视图控制器和 Segue 添加到情节提要

1. 使用项目导航器返回到 **MainStoryboard_iPhone.storyboard** 文件。

2. 将 Todo 项的新视图控制器添加到现有“Todo 列表视图控制器”右侧的情节提要。将这个新视图控制器的自定义类设为 **QSItemViewController**。若要了解详细信息，请参阅[将场景添加到情节提要]。

3. 从“Todo 列表视图控制器”将“显示”Segue 添加到“Todo 项视图控制器”。然后，在属性检查器中，将 Segue 标识符设置为 **detailSegue**。

    不要从源视图控制器中的任何单元格或按钮创建此 Segue，而是从情节提要界面中“Todo 列表视图控制器”上方的视图控制器图标，按 CTRL 的同时拖动到“Todo 项视图控制器”：

    ![][todo-list-view-controller-add-segue]

    如果你意外地从单元格创建 Segue，则在运行该应用程序时将触发 Segue 两次，从而导致此错误：

        Nested push animation can result in corrupted navigation bar

    若要了解有关 Segue 的详细信息，请参阅[在情节提要中的场景之间添加 Segue]。

4. 也使用标签将项文本的文本字段以及完成状态的分段控件添加到新的“Todo 项视图控制器”。在分段控件中，将“段 0”的标题设为“是”，将“段 1”的标题设为“否”。将这些新字段连接到代码中的 Outlet。若要了解详细信息，请参阅[构建用户界面]和[分段控件]。

      ![][add-todo-item-view-controller-3]

5. 将这些新字段连接到你已添加到 **QSItemViewController.m** 的相应 Outlet。将项文本字段连接到到 **itemText** Outlet，将完成状态分段控件连接到 **itemComplete** Outlet。若要了解详细信息，请参阅[创建 Outlet 连接]。

6. 将文本字段的委托设置为视图控制器。从该文本字段按 CTRL 并拖动到情节提要界面中“Todo 项视图控制器”下的视图控制器图标，然后选择委托 Outlet；这将指示情节提要，该文本字段的委托是此视图控制器。

7. 确保该应用程序已应用到目前为止所做的所有更改。现在，在模拟器中运行该应用程序。将项添加到 Todo 列表，然后单击这些项。你将看到（当前为空的）项视图控制器。

      ![][add-todo-item-view-controller-4] ![][add-todo-item-view-controller-5]

###  <a name="add-item-details"></a>将项详细信息添加到 Todo 项视图控制器

1. 我们将从 **QSTodoListViewController.m** 引用 **QSItemViewController**。因此，在 **QSTodoListViewController.m** 中，让我们添加行以便导入 **QSItemViewController.h**。

        #import "QSItemViewController.h"

2. 向 **QSTodoListViewController.m** 中的 **QSTodoListViewController** 接口添加一个新属性，以便存储所编辑的项：

        @property (strong, nonatomic)   NSDictionary *editingItem;

3. 在 **QSTodoListViewController.m** 中实现 **tableView:didSelectRowAtIndexPath:**以便保存所编辑的项，然后调用 Segue 以显示详细信息视图。

          - (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath {
            NSManagedObject *item = [self.fetchedResultsController objectAtIndexPath:indexPath];
            self.editingItem = [MSCoreDataStore tableItemFromManagedObject:item]; // map from managed object to dictionary

              [self performSegueWithIdentifier:@"detailSegue" sender:self];
          }

4. 在 **QSTodoListViewController.m** 中实现**prepareForSegue:sender:** 以将项传递到“Todo 项视图控制器”，并指定用户退出详细信息视图时的回调：

        - (void)prepareForSegue:(UIStoryboardSegue *)segue sender:(id)sender {
            if ([[segue identifier] isEqualToString:@"detailSegue"]) {
                QSItemViewController *ivc = (QSItemViewController *) [segue destinationViewController];
                ivc.item = [self.editingItem mutableCopy];
                
                ivc.editCompleteBlock = ^(NSDictionary *editedValue) {
                    [self.todoService updateItem:editedValue completion:^(NSUInteger index) {
                        self.editingItem = nil;
                    }];
                };
            }
        }

5. 确保该应用程序已应用到目前为止所做的所有更改。现在，在模拟器中运行该应用程序。将项添加到 Todo 列表，然后单击这些项。你将看到项视图控制器不再为空，它显示 Todo 项的详细信息。

      ![][add-todo-item-view-controller-6]

###  <a name="saving-edits"></a>添加对保存编辑的支持

1. 当你单击导航视图中的“上一步”按钮时，所做的编辑都将丢失。我们已将数据发送到详细信息视图，但数据未发送回母版视图。由于我们已传递项副本的指针，我们可以使用该指针来检索对项所做的更新的列表并在服务器上更新该项。在开始之前，首先在 **QSTodoService.m** 中更新 **QSTodoService** 的服务器包装类，具体方法是删除 **completeItem** 操作并添加新的 **updateItem** 操作。这是因为 **completeItem** 仅将项标记为完成；而 **updateItem** 将更新项。

        - (void)updateItem:(NSDictionary *)item completion:(QSCompletionBlock)completion
        {
            // Set the item to be complete (we need a mutable copy)
            NSMutableDictionary *mutable = [item mutableCopy];

            // Update the item in the TodoItem table and remove from the items array when we mark an item as complete
            [self.syncTable update:mutable completion:^(NSError *error) {
                [self logErrorIfNotNil:error];

                if (completion != nil) {
                    dispatch_async(dispatch_get_main_queue(), completion);
                    }
            }];
        }

2. 从 **QSTodoService.h** 中删除 **completeItem** 的声明，并为 **updateItem** 添加此声明：

        - (void)updateItem:(NSDictionary *)item completion:(QSCompletionBlock)completion;

3. 现在让我们来测试该应用程序。确保该应用程序已应用到目前为止所做的所有更改。现在，在模拟器中运行该应用程序。将项添加到 Todo 列表，然后单击这些项。尝试编辑某个项，然后返回。验证该项的说明是否在应用程序的母版视图中更新。使用向下拖动手势刷新应用程序，然后验证编辑是否反映到你的远程服务中。

###  <a name="conflict-handling-problem"></a>冲突处理问题

1. 让我们来看一下，当两个不同的客户端尝试同时修改相同的数据片段时，会发生什么情况。在下面列出的示例中，有一个项“Mobile Services is Cool!”， 让我们在一个设备上将此项更改为“I love Mobile Services!”，并在另一个设备将此项更改为“I love Azure!”。

      ![][conflict-handling-problem-1]

2. 在两个位置启动该应用程序：在两个 iOS 设备上，或者在模拟器和 iOS 设备上。如果你没有物理设备可用于测试，请在模拟器中启动一个实例，并使用 REST 客户端将 PATCH 请求发送到移动服务。PATCH 请求的 URL 反映了移动服务的名称、你正在编辑的 Todo 项表的名称及 ID，而 x-zumo-application 标头是应用程序密钥：

        PATCH https://donnam-tutorials.azure-mobile.net/tables/todoitem/D265929E-B17B-42D1-8FAB-D0ADF26486FA?__systemproperties=__version
        Content-Type: application/json
        x-zumo-application: xuAdWVDcLuCNfkTvOfaqzCCSBVHqoy96

        {
            "id": "CBBF4464-E08A-47C9-B6FB-6DCB30ACCE7E",
            "text": "I love Azure!"
        }

3. 现在，刷新应用程序的两个实例中的项。你将看到在 Xcode 的输出日志中输出错误：

        TodoList[1575:4a2f] ERROR Error Domain=com.Microsoft.WindowsAzureMobileServices.ErrorDomain Code=-1170 "Not all operations completed successfully" UserInfo=0x8dd6310 {com.Microsoft.WindowsAzureMobileServices.ErrorPushResultKey=(
            "The server's version did not match the passed version"
        ), NSLocalizedDescription=Not all operations completed successfully}

  这是因为在完成块上调用 **pullWithQuery:completion:** 时，错误参数将为非零，这将导致错误通过 **NSLog** 输出到输出。

###  <a name="service-add-conflict-handling"></a>更新 QSTodoService 以支持冲突处理

1. 让我们通过在客户端中处理冲突来让用户决定如何处理冲突。为此，让我们实现 **MSSyncContextDelegate** 协议。在 **QSTodoService.h** 和 **QSTodoService.m** 中，将 **(QSTodoService *)defaultService;** 工厂方法声明更改为下面的语句，以便接收作为参数的同步上下文委托：

        + (QSTodoService *)defaultServiceWithDelegate:(id)delegate;

2. 在 **QSTodoService.m** 中，按如下所示更改 **init** 行，同样接收作为参数的同步上下文委托：

        -(QSTodoService *)initWithDelegate:(id)syncDelegate

3. 在 **QSTodoService.m** 中，将 **defaultServiceWithDelegate** 中的 **init** 调用更改为 **initWithDelegate**：

        service = [[QSTodoService alloc] initWithDelegate:delegate];

4. 返回到 **QSTodoService.m** 中，更改 **self.client.syncContext** 的初始化，以传入 **syncDelegate** 而不是委托的 **nil**：

        self.client.syncContext = [[MSSyncContext alloc] initWithDelegate:syncDelegate dataSource:store callback:nil];

###  <a name="add-alert-view"></a>添加 UI 警报视图帮助器以支持冲突处理

1. 如果出现冲突，让我们允许用户选择要保留哪个版本：
  * 保留客户端版本（这会覆盖服务器上的版本），
  * 保留服务器版本（这会更新客户端本地表），或者
  * 两个版本都不保留（取消推送，让操作挂起）。

  由于我们在显示提示时也可能会发生另一个更新，因此我们将继续显示选项，直到服务器停止返回失败响应为止。在代码中，让我们使用帮助器类，该类显示警报视图，并创建委托以便在显示警报视图时调用。让我们首先定义帮助器类 **QSUIAlertViewWithBlock**。

2. 使用 Xcode 添加此新类 **QSUIAlertViewWithBlock**，并使用以下内容覆盖 **QSUIAlertViewWithBlock.h**：

        #import <Foundation/Foundation.h>

        typedef void (^QSUIAlertViewBlock) (NSInteger index);

        @interface QSUIAlertViewWithBlock : NSObject <UIAlertViewDelegate>

        - (id) initWithCallback:(QSUIAlertViewBlock)callback;
        - (void) showAlertWithTitle:(NSString *)title message:(NSString *)message cancelButtonTitle:(NSString *)cancelButtonTitle otherButtonTitles:(NSArray *)otherButtonTitles;

        @end

3. 接下来，使用以下文件覆盖 **QSUIAlertViewWithBlock.m**：

        #import "QSUIAlertViewWithBlock.h"
        #import <objc/runtime.h>

        @interface QSUIAlertViewWithBlock()

        @property (nonatomic, copy) QSUIAlertViewBlock callback;

        @end

        @implementation QSUIAlertViewWithBlock

        static const char *key;

        @synthesize callback = _callback;

        - (id) initWithCallback:(QSUIAlertViewBlock)callback
        {
            self = [super init];
            if (self) {
                _callback = [callback copy];
            }
            return self;
        }

        - (void) showAlertWithTitle:(NSString *)title message:(NSString *)message cancelButtonTitle:(NSString *)cancelButtonTitle otherButtonTitles:(NSArray *)otherButtonTitles {
            UIAlertView *alert = [[UIAlertView alloc] initWithTitle:title
                                                            message:message
                                                           delegate:self
                                                  cancelButtonTitle:cancelButtonTitle
                                                  otherButtonTitles:nil];

            if (otherButtonTitles) {
                for (NSString *buttonTitle in otherButtonTitles) {
                    [alert addButtonWithTitle:buttonTitle];
                }
            }

            [alert show];

            objc_setAssociatedObject(alert, &key, self, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
        }

        - (void) alertView:(UIAlertView *)alertView didDismissWithButtonIndex:(NSInteger)buttonIndex
        {
            if (self.callback) {
                self.callback(buttonIndex);
            }
        }

        @end

###  <a name="add-conflict-handling"></a>将冲突处理程序添加到 Todo 列表视图控制器

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

###  <a name="test-app"></a>测试应用程序

让我们通过冲突测试应用程序！ 在同时运行的应用程序的两个不同实例中编辑同一项，或者使用应用程序和 REST 客户端进行编辑。

通过从顶部拖动在应用程序实例中执行刷新手势。现在，你将看到协调冲突的提示：

![][conflict-ui]


###  摘要

为了设置[脱机数据处理入门]中完成的项目以检测冲突，你首先需要添加编辑和更新 Todo 项的功能。

为此，你添加了新的项详细信息视图控制器，连接了主视图控制器和详细信息视图控制器，最后添加了保存编辑并将编辑推送到云的功能。

接下来，你发现在出现冲突时会发生的情况。你通过实现 **MSSyncContextDelegate** 协议添加了对冲突处理程序的支持。你还通过服务器接口类 **QSTodoService**、**QSTodoListViewController** 和支持类启用了对同步上下文委托的支持。

在此过程中，你添加了 **QSUIAlertViewWithBlock** 帮助器类以向用户显示警报，最后，通过将代码添加到 **QSTodoListViewController** 来提示用户以三种方式之一协调冲突。

<!-- URLs. -->

[更新应用程序项目以允许编辑]: #update-app
[更新 Todo 列表视图控制器]: #update-list-view
[添加 Todo 项视图控制器]: #add-view-controller
[将 Todo 项视图控制器和 Segue 添加到情节提要]: #add-segue
[将项详细信息添加到 Todo 项视图控制器]: #add-item-details
[添加对保存编辑的支持]: #saving-edits
[冲突处理问题]: #conflict-handling-problem
[更新 QSTodoService 以支持冲突处理]: #service-add-conflict-handling
[添加 UI 警报视图帮助器以支持冲突处理]: #add-alert-view
[将冲突处理程序添加到 Todo 列表视图控制器]: #add-conflict-handling
[测试应用程序]: #test-app


[add-todo-item-view-controller-3]: ./media/mobile-services-ios-handling-conflicts-offline-data/add-todo-item-view-controller-3.png
[add-todo-item-view-controller-4]: ./media/mobile-services-ios-handling-conflicts-offline-data/add-todo-item-view-controller-4.png
[add-todo-item-view-controller-5]: ./media/mobile-services-ios-handling-conflicts-offline-data/add-todo-item-view-controller-5.png
[add-todo-item-view-controller-6]: ./media/mobile-services-ios-handling-conflicts-offline-data/add-todo-item-view-controller-6.png
[todo-list-view-controller-add-segue]: ./media/mobile-services-ios-handling-conflicts-offline-data/todo-list-view-controller-add-segue.png
[update-todo-list-view-controller-2]: ./media/mobile-services-ios-handling-conflicts-offline-data/update-todo-list-view-controller-2.png
[conflict-handling-problem-1]: ./media/mobile-services-ios-handling-conflicts-offline-data/conflict-handling-problem-1.png
[conflict-ui]: ./media/mobile-services-ios-handling-conflicts-offline-data/conflict-ui.png


[分段控件]: https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/UIKitUICatalog/UISegmentedControl.html
[Core Data Model Editor Help]: https://developer.apple.com/library/mac/recipes/xcode_help-core_data_modeling_tool/Articles/about_cd_modeling_tool.html
[创建 Outlet 连接]: https://developer.apple.com/library/mac/recipes/xcode_help-interface_builder/articles-connections_bindings/CreatingOutlet.html
[构建用户界面]: https://developer.apple.com/library/mac/documentation/ToolsLanguages/Conceptual/Xcode_Overview/Edit_User_Interfaces/edit_user_interface.html
[在情节提要中的场景之间添加 Segue]: https://developer.apple.com/library/ios/recipes/xcode_help-IB_storyboard/chapters/StoryboardSegue.html#//apple_ref/doc/uid/TP40014225-CH25-SW1
[将场景添加到情节提要]: https://developer.apple.com/library/ios/recipes/xcode_help-IB_storyboard/chapters/StoryboardScene.html
[Core Data]: https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/CoreData/cdProgrammingGuide.html
[Download the preview SDK here]: http://aka.ms/Gc6fex
[How to use the Mobile Services client library for iOS]: mobile-services-ios-how-to-use-client-library
[Getting Started Offline iOS Sample]: https://github.com/Azure/mobile-services-samples/tree/master/TodoOffline/iOS/blog20140611
[脱机数据处理入门]: mobile-services-ios-get-started-offline-data
[Get started with Mobile Services]: mobile-services-ios-get-started
[Get started with data]: mobile-services-ios-get-started-data
 

<!---HONumber=HO63-->